---
title: Key reset for MSP
description: Learn more about key reset
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Reset Latched Key

If the VM loses its copy of the latched key, a disk is migrated to a new VM, or any other key mismatch occurs, the VM will be unable to access Wireserver + IMDS. Reseting the key will bring the VM back to a healthy state if the key is lost or unmatched between Host and Guest.

> The VM owner must request the key reset. Metadata services can't distinguish between an attacker or the GPA requesting a reset when the key is lost, so resets can not be issued from within the VM.

## Reset a VM's Key

The platform will always ensure the `keyIncarnationId` in the VM model matches the actual key in storage. By incrementing this value, a key reset will be triggered. See [Configuration](../configuration.md) for more details.

```http
PATCH https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine_Name}?api-version=2024-03-01
{
  "properties": {
    "securityProfile": {
      "proxyAgentSettings": {
        "keyIncarnationId": 10
      }
    }
  }
}
```

### Confirm Reset Occurred
Check the `AzureProxyAgentExtension` portion of the VM Instance view to confirm the new key has been generated. You will see your new `keyIncarnationId` once the change has propagated end to end.

```json
"keyLatchStatus":{
    "status":"RUNNING",
    "message":"Found key details from local and ready to use. - 122",
    "states":{
        "imdsRuleId":"/SUBSCRIPTIONS/{subscription_id}/RESOURCEGROUPS/{resource_group}/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSIMDS/VERSIONS/{data_version}",
        "secureChannelState":"WireServer Enforce - IMDS Enforce",
        "keyIncarnationId":"10",
        "keyGuid":"e3882f98-da8d-4410-8394-06c23462781c",
        "wireServerRuleId":"/SUBSCRIPTIONS/{subscription_id}/RESOURCEGROUPS/{resource_group}/PROVIDERS/MICROSOFT.COMPUTE/GALLERIES/GALLERYXX/INVMACCESSCONTROLPROFILES/WINDOWSWIRESERVER/VERSIONS/{data_version}"
        }
    }
```

> [!NOTE]
> These requests are idempotent. However, if multiple requests are made with multiple `keyIncarnationId` there is no guarantee on the number and order of `keyIncarnationId` you will be able to observe. The final state will reflect whichever request used the largest value.

## Reset a VMSS' Key

Key data is unique for each instance in a Scale Set. The key must be reset on a per-instance basis.

We only support reset key for particular VmScaleSet VM Instance. We cannot reset key for all the VMSS instances in a single API.

See [Reset a VM's Key](#reset-a-vms-key), and substitute in your VMSS instance's resource ID instead. Example:

`https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSet_name}/virtualMachines/{instance_id}?api-version=2024-03-01`
