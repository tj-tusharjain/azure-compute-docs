---
title: View instance mix configurations
description: How to view instance mix configurations on a scale set. 
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 1/10/2025
ms.reviewer: jushiman
---

# View instance mix configurations

This article details how to view your instance mix configuration on a virtual machine scale set, including the VM sizes and the allocation strategy.

> [!IMPORTANT]
> Instance mix for Virtual Machine Scale Sets with Flexible Orchestration Mode is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature may change prior to general availability (GA). 

## Prerequisites
Before using instance mix, complete feature registration for the `FlexVMScaleSetSkuProfileEnabled` feature flag using the [az feature register](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

It takes a few moments for the feature to register. Verify the registration status by using the [az feature show](/cli/azure/feature#az-feature-register) command:

```azurecli-interactive
az feature show --namespace "Microsoft.Compute" --name "FlexVMScaleSetSkuProfileEnabled"
```

## View the instance mix configurations
### [Azure portal](#tab/portal-1)
1. Navigate to the virtual machine scale set you want to view.
2. In the **Overview** blade, you will find the **Properties** section.
3. In the **Size** subsection of the properties section, you'll see the VM sizes that can be used in the scale set.
4. In the **Management** subsection of the properties section, you'll see the **Allocation strategy**.

### [Azure CLI](#tab/cli-1)

#### skuProfile
To view all properties in the `skuProfile`, run the following command
```azurecli-interactive
az vmss show --resource-group resourceGroupName --name scaleSetName --query "skuProfile"
```

---