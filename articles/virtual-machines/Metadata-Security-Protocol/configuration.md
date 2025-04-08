# Configuration

MSP offers customization to maximally restrict metadata server access in your workload. This page introduces the fundamental concepts that can be quickly enabled on any supported VM/VMSS.

Users that are more familiar how their workload uses metadata services can harden access further by following the [Advanced Configuration guide](./advanced-configuration.md).

## Concepts

| Term | Definition |
|--|--|
| "Metadata Services" | A general term for the industry practice of offering private, non-internet anonymous HTTP APIs available at a fixed location. Typically `169.254.169.254`. While the exact features, details, and service count vary by cloud provider, they all share a similar architecture and purpose. |
| [IMDS](https://aka.ms/azureimds) | A user facing metadata service that comes with support and full API documentation. It is intended for both Azure and 3rd party integrations. It offers a mix of metadata and credentials. |
| [WireServer](https://aka.ms/azurewireserver) | A platform facing metadata service, used to implement core IaaS functionality. It isn't intended for general consumption and Azure offers no support for doing so. While the consumers of the service are Azure implementation related, the service must still be considered in security assessments. WireServer resources are treated as privileged by default. However, IMDS resources shouldn't be viewed as non-privileged. Depending on your workload, IMDS can also provide sensitive information. |
| Guest Proxy Agent (GPA) | An in-guest agent that enables MSP protections. |

## ARM Fields

All configuration is managed via new properties in the `securityProfile` section of your resource's configuration.

> The minimum API version to configure MSP is `2024-03-01`

### General Configuration

`ProxyAgentSettings` is the new property in the VM model for Azure VM and VMSS to configure MSP feature.

| `ProxyAgentSettings` property | Type | Default | Details  |
|--|--|--|--|
| `enabled`| `Bool` | `false` | Specifies whether MSP feature should be enabled on the virtual machine or virtual machine scale set. Default is `false`. This controls: <ul><li>Applying other settings</li><li>Platform latched key generation</li><li>Automatic GPA Installation (when `true`) / Uninstallation (when `false`) on Windows VM/VMSS</li></ul> |
| `imds` | HostEndpointSettings | N/A | IMDS specific configuration. See [Per-Metadata Service Configuration](#per-metadata-service-configuration). |
| `wireServer` | HostEndpointSettings | N/A | Settings for the Wireserver endpoint. See [Per-Metadata Service Configuration](#per-metadata-service-configuration). |
| `keyIncarnationId` | `Integer` | `0` | A counter for key generation. Strictly monotonically incrementing this value (other values will be ignored) instructs MSP to reset the key used for securing communication channel between guest and host. The GPA will notice that the key has been reset and automatically acquire the new one. (*This property is only for used recovery/troubleshooting purposes.*) |

### Per-Metadata Service Configuration

Protections for each metadata service can be individually configured. The schema is the same for each service.

> Inline and linked configurations are mutually exclusive on a per-service basis.

#### Inline Configuration

An inline configuration can be defined using the `mode` property of `HostEndpointSettings`. Inline configurations do not support customization.

| `mode` (Enum; expressed as `String`) | GPA Behavior | Service Behavior |
|--|--|--|
| `"Disabled"` | The GPA will not set up eBPF interception of requests to this service. Requests will go directly to the service. | Unchanged. The service will **not** require that requests are endorsed (signed) by the GPA. |
| `"Audit"` | The GPA will intercept requests to this service and determine if they are authorized by the current configuration. The request will always be forwarded to the service, but the result + caller information will be logged. | Unchanged. The service will **not** require that requests are endorsed (signed) by the GPA. |
| `"Enforce"` | The GPA will intercept requests to this service and determine if they are authorized by the current configuration. If the caller is authorized, it will sign the request to endorse it. If the caller is not authorized, the GPA will immediately respond with status code `401` or `403` as appropriate. All outcomes will be recorded in the audit log. | Requests to the service **must** be endorsed (signed) by the GPA. Unsigned requests will be rejected with a `401` status code. |

> This property cannot be used at the same time as `inVMAccessControlProfileReferenceId`!

> In WireServer `Enforce` mode, the GPA will implicitly require that a process runs as `Administrator` / `root` to access WireServer. This is equivalent to the in-guest firewall rule that exists even when MSP is disabled but with a more secure implementation.

#### Linked configuration

The `inVMAccessControlProfiles` resource type defines a per-service configuration which can be linked against VM/VMSS. These support full customization. See [Advanced Configuration](./advanced-configuration.md) for full details.

| `HostEndpointSettings` property | Type | Details |
|--|--|--|
| `inVMAccessControlProfileReferenceId` | `String` | The resource id of the configuration to apply. |

> Must be a full ARM ID, which takes the form: `/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}`

> This property cannot be used at the same time as `mode`!

#### Full Example

- VM: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01`
- VMSS: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSet-name}/virtualMachines/{instance-id}?api-version=2024-03-01`

##### Using `mode`

```json
{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "wireServer": {
          "mode": "Enforce"
        },
        "imds": {
          "mode": "Audit"
        }
      }
    },
  }
}
```

##### Using `inVMAccessControlProfileReferenceId`

```json
{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "wireServer": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
        },
        "imds": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{profile}/versions/{version}"
        }
      }
    },
  }
}
```


## Recommended Progression

Customers often aren't aware of the full extent Wireserver + IMDS usage in their workload. Your VM's usage is a combination of any custom integrations you've created and the implementation details of any other software you're running. We recommend starting with a simpler configuration, and iterating over time as you gain more information. In general, it's best to go in this order:

1. Enable MSP for Wireserver and IMDS, in `Audit` mode. This will allow you to confirm compatibility, learn more about your workload, and generates valuable telemetry for detecting threat actors.
1. Review the audit telemetry, paying particularly close attention to any flagged requests. These can be signs of an ongoing attack or a workload incompatibility.
1. Remediate any issues. Under the default setup anything being flagged is a misuse of the metadata services. Contact your security team and/or the corresponding service owners as needed.
1. Switch from `Audit` to `Enforce` mode. Flagged requests will be rejected, protecting your workload.
1. Experiment with [Advanced Configuration](./advanced-configuration.md). The default configuration mirror's pre-MSP security rules, which are often overly permissive. Defining custom configuration for your workload will enable you to further enhance your security by applying the principle of least privileged access.

*Don't make the perfect the enemy of the good*. **Any** level of MSP enablement greatly improves your security and protects against the majority of previous real world attacks.

## Next Steps

- [Deploy a VM/VMSS With MSP](./greenfield.md)
- [Enable MSP on Existing VM/VMSS](./brownfield.md)

## Related Content

- [Advanced Configuration guide](./advanced-configuration.md)
