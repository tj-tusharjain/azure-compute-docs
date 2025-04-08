# Disable MSP

To disable MSP, set the `securityProfile.proxyAgentSettings.enabled` to `false`.

## Via REST API

```http
PATCH https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01
{
  "properties": {
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": false,
      }
    },
  }
}
```

## Via Azure Portal

Portal doesn't allow disabling MSP completely at this time, but you can disable actual enforcement of security requirements to allow requests to flow normally:

![image.png](../../images/disable-msp-portal.png)
