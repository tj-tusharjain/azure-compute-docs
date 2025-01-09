---
title: Create a Virtual Machine Scale Set with Instance Mix
description: How to create a virtual machine scale set using Instance Mix on different platforms. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 1/7/2025
ms.reviewer: jushiman
---

# Update Instance Mix settings on an existing scale set
The article walks through how to update the Instance Mix settings on a scale set.

> [!IMPORTANT]
> Instance Mix for Virtual Machine Scale Sets with Flexible Orchestration Mode is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

## Prerequisites
Before using Instance Mix, complete feature registration for the `FlexVMScaleSetSkuProfileEnabled` feature flag using the [az feature register](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

It takes a few moments for the feature to register. Verify the registration status by using the [az feature show](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

## Update the Instance Mix settings on an existing scale set

### [Azure CLI](#tab/cli-1)

#### Change the Allocation Strategy
You can use the following basic command to update the allocation strategy. In this case, we're updating the scale set to use the `CapacityOptimized` allocation strategy:

```azurecli-interactive
az vmss update \
    --resource-group {resourceGroupName}
    --name {scaleSetName}
    --set skuProfile.allocationStrategy=CapacityOptimized
```
#### Change the VM Sizes
You can use the following command to update the VM sizes specified in the `skuProfile`

### [Azure PowerShell](#tab/powershell-1)

#### Change the Allocation Strategy
You can use the following basic command to update the allocation strategy:
 
```azurepowershell-interactive
# Set variable values
$resourceGroupName = "resourceGroupName"
$vmssName = "scaleSetName"
$allocationStrategy = "CapacityOptimized";

# Get the scale set information
$vmss = Get-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName;

# Update the allocation strategy
$vmss.Sku.AllocationStrategy = $allocationStrategy

#Update the scale set
Update-AzVmss `
-ResourceGroupName $resourceGroupName `
-VMScaleSetName $vmssName `
```
#### Change the VM Sizes

### [REST API](#tab/arm-1)
To update the Instance Mix settings through REST API, use a `PATCH` call to the VMSS resource. Be sure to use an API version on or after `2023-09-01`:
```json
PUT https://management.azure.com/subscriptions/{YourSubscriptionId}/resourceGroups/{YourResourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{youScaleSetName}?api-version=2023-09-01
```

The following subsections will walk through what to use if you want to change the allocation strategy or VM sizes through REST APIs.

#### Change the Allocation Strategy
Be sure to include the properties you'd like to change in the request body. In this example, we're changing the allocation strategy:
```json

```

#### Change the VM Sizes
In this example, we're changing the VM sizes specified in the scale set:
```json

```

---

## Update an existing scale set to use Instance Mix
Existing scale sets that do not have Instance Mix can enable Instance Mix by specifying the `skuProfile` properties in the scale set. This can be specified through REST, CLI, and PowerShell. The following sections have sample code snippets to demonstrate enabling Instance Mix on existing scale sets.

### [Azure portal](#tab/portal-2)
### [Azure CLI](#tab/cli-2)
```azurecli-interactive

```
### [Azure PowerShell](#tab/powershell-2)
```azurepowershell-interactive
```

### [REST API](#tab/arm-2)
```json
```


## Next steps
Learn how to [troubleshoot](instance-mix-faq-troubleshooting.md) your Instance Mix enabled scale set.