---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: Configure VM watch
description: Getting started guide on how to configure VM watch on virtual machines
author:      ofemifowode # GitHub alias
ms.author:   ofemifowode # Microsoft alias
ms.service: azure-virtual-machines
ms.topic: get-started
ms.date:     10/28/2024
---

# Configure VM watch


VM watch signals can be categorized into two groups: Core and Optional <link to plugin page>. By default, only core group signals are enabled with default configurations <link to plugin page>. However, these default settings can be easily overwritten from the `vmWatchSettings` using either [ARM template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/), [AZ CLI](https://learn.microsoft.com/en-us/cli/azure/) or [Powershell](https://learn.microsoft.com/en-us/powershell/).

### Pre-requisites
This article assumes that you are familiar with:
- VM watch [Checks, Metrics and Logs](/azure/virtual-machines/azure-vm-watch) and their descriptions
- [Installing VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2) to Virtual Machines and Scale Sets

- VM Watch Collector Suite

### Accessing vmWatchSettings on Azure Virtual Machines

> [!IMPORTANT]
> The code segment is identical for both Windows and Linux except for the value of the parameter `<your extension name>` passed into the Extension name. Please replace `<your extension name>` with `ApplicationHealthLinux` for Linux and `ApplicationHealthWindows` for Windows installations.  

#### [ARM Template](#tab/ARM-template-1)
1. Navigate back to the Overview Page on [Azure portal](https://portal.azure.com/) and click on the JSON view for the VM to find the code segment below.
2. Copy the code segment to an IDE such as [Visual Studio Code](https://learn.microsoft.com/en-us/visualstudio/ide/?view=vs-2022) and make the necessary customizations

```
{
   "settings": {
      "vmWatchSettings": {
         "enabled": true
      }
   }
}
```

#### [CLI](#tab/cli-1)
```
az vm extension show -g <your resource group name> --vm-name <your vm name> -n <your extension name>
```
#### [PowerShell](#tab/powershell-1)
```
Get-AzVMExtension -ResourceGroupName "<your resource group name>" -VMName "<your vm name>" -Name "<your extension name>" 
```
---

### Customizing VM watch configurations 
VM watch tests, metrics and logs can be customized by configuring the `vmWatchSettings` properties to meet specific requirements. Below is a list of `vmWatchSettings` properties.

#### vmWatchSettings Properties 
| **Name** | **Type** | **Description** | **Is Required**
|---|---|---|---|
| **enabled** | `bool` | This shows whether VM watch is enabled or not | true
| **signalFilters** | `object` | This filters the enabled / disabled signals, either by tag or signal name. | false
| **parameterOverrides** | `object` | This specifies the parameters to overwrite for each signal execution. The default parameters can be found in the VM watch plugin collection page <link to plugin page>. | false
| **environmentAttributes** | `object` | This specifies any environment attributes to help decide if a test is eligible to execute or not. | false

> [!IMPORTANT]
> see VM watch Plugin Collection <insert link to plug in page> for a full list of VM watch signals, associated tags, overwritable parameters and environment attributes.
>
 
#### Switching On / Off VM watch 

VM watch can be switched on / off by configuring the `enabled` property, as shown in the code segment below. 

```
{
   "vmWatchSettings": {
      "enabled": true
   }
}
```
> [!NOTE]
> | **Name**  | **Description** |
> |---|---|
> | **true**  | This setting will enable VM watch |
> | **false** | This setting will disable VM watch |
> 

#### Enabling / disabling signal execution 

By default, only the core group signals <link> are enabled. However, the `signalFilters` property can be used to control and configure the signals to be executed. This property includes the following subfields.

| **Subfields** | **Description** |
|---|------|
| **enabledTags** | This will enable the signals in the optional group specified with these tags |  
| **disabledTags** | This will disable the signals in the core and optional groups specified with these tags | 
| **enabledOptionalSignals** | This will enable signals specified in optional group. Provide signal name(s) as parameter | 
| **disabledSignals** | This will disable the signals specified in the core and optional groups. Provide signal name(s) as parameter | 


For instance, to enable signals in the optional group containing `Network` tag and disable signals containing `Disk` tag, specify such tags under the `enabledTags` and `disabledTags` as shown below:
```
{
   "vmWatchSettings": {
      "enabled": true,
      "signalFilters": {
         "enabledTags": [
            "Network"
         ],
         "disabledTags": [
            "Disk"
         ]
      }
   }
}
```

Similarly, to enable an optional group signal with name `hardware`, and disable signals with name `process` and `dns`, specify such names under the `enabledOptionalSignals` and `disabledSignals` as shown below:  

```
{
   "vmWatchSettings": {
      "enabled": true,
      "signalFilters": {
         "enabledOptionalSignals": [
            "hardware"
         ],
         "disabledSignals": [
            "process",
            "dns"
         ]
      }
   }
}
```

#### Configure signal execution frequency 

The signal execution frequency can be customized by adjusting the `parameterOverrides` property. 
For instance, to set the `outbound connectivity` test execution frequency to 60 seconds, specify the following configuration below:
```
{
   "vmWatchSettings": {
      "enabled": true,
      "parameterOverrides ": {
         "OUTBOUND_CONNECTIVITY_INTERVAL": "60s"
      }
   }
}
```
 

#### Override default signal execution parameters 

Signal execution parameters can be overwritten by setting the `parameterOverrides` property. For instance, to set `disk_io` signal mount point to `/mnt`, the following configuration can be used: 
```
{
   "vmWatchSettings": {
      "enabled": true,
      "parameterOverrides ": {
         "DISK_IO_MOUNT_POINTS": "/mnt"
      }
   }
}

``` 

#### Environment attribute enrichments 

In addition to tags, VM watch also checks the eligibility <insert link to plugin page> of the signals before execution. The environment attributes can be specified to help VM watch determine the eligibility of each signal for execution. 
For instance, if outbound traffic has been disabled on a VM, this information can be provided to VM watch. This will ensure that any outbound network-related signal execution will be marked as ineligible.
```
{
   "vmWatchSettings": {
      "enabled": true,
      "environmentAttributes ": {
         "OutboundConnectivityDisabled": true
      }
   }
}
```

### Next Steps

- Link to VM watch Plugin Collection Page

- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)

- [VM watch overview](/azure/virtual-machines/azure-vm-watch)

