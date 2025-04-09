---
title: audit log for metadata security protocol
description: Learn more about deploying a VM with MSP
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Deploy a VM/VMSS With MSP

## Prerequisites

1. Ensure your image of choice is [compatible](./overview.md#compatibility).
1. Familiarize yourself with the [basic configuration](./configuration.md#configuration) options.

## Deploy VM with MSP

ARM templates are also supported. Here are examples for some of the deployment options:

See [examples](./other-examples/portal.md).

### With REST API

Deploy a VM as you normally would, but with the MSP configuration also applied. Example referencing a linked [Advanced Configuration](./advanced-configuration.md):

```http
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{virtualMachine-Name}?api-version=2024-03-01

{
  "name": "GPAWinVM",
  "location": "centraluseuap",
  "properties": {
    ...
    ...
    "securityProfile": {
      "proxyAgentSettings": {
        "enabled": true,
        "keyIncarnationId": 0,
        "wireServer": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{wireServerProfileName}/versions/{version}"
        },
        "imds": {
          "inVMAccessControlProfileReferenceId": "/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/inVMAccessControlProfiles/{imdsProfileName}/versions/{version}"
        }
      }
    },
  }
}
```

#### Validating the linked rules were applied to your VM

Once a VM is deployed, these steps are the same as for an [existing VM](./brownfield.md#validating-the-linked-rules-were-applied-to-your-vm).

## Deploy VMSS with MSP

Applying the same steps to the VMSS model will apply MSP to every VM in the Scale Set.
