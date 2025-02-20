---
title: Configure VM watch
description: Getting started guide on how to configure VM watch on virtual machines
author:      ofemifowode 
ms.author:   ofemifowode 
ms.service: azure-virtual-machines
ms.topic: get-started
ms.date:     10/28/2024
---

# Configure VM watch

VM watch signals are logically grouped into Collector Suites, which can be categorized into two groups: core and optional`<link to collector page>`. By default, only core group collectors are enabled with default configurations. However, these default settings can be easily overwritten from `vmWatchSettings` using either [ARM template](/azure/azure-resource-manager/templates/), [AZ CLI](/cli/azure/), or [Powershell](/powershell/).

This article describes how to configure VM watch to suit specific requirements.

### Prerequisites
This article assumes that you're familiar with:
- [VM watch Checks, Metrics, and Logs](/azure/virtual-machines/azure-vm-watch)
- [Installing VM watch to Virtual Machines and Scale Sets](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)
- VM watch Collector Suites `<insert link to collector page here>`

### Access `vmWatchSettings` on Azure Virtual Machines

> [!IMPORTANT]
> The code segment is identical for both Windows and Linux except for the value of the parameter `<application health extension type>` passed into the Extension Type. Replace `<application health extension type>` with `ApplicationHealthLinux` for Linux and `ApplicationHealthWindows` for Windows installations.  

#### [ARM Template](#tab/ARM-template-1)
1. Navigate back to the Overview Page on [Azure portal](https://portal.azure.com/) and click on the JSON view for the VM to find the code segment below.
2. Copy the code segment to an IDE such as Visual Studio Code and make the necessary customizations

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
az vm extension show -g <your resource group name> --vm-name <your vm name> -n <application health extension type>
```
#### [PowerShell](#tab/powershell-1)
```
Get-AzVMExtension -ResourceGroupName "<your resource group name>" -VMName "<your vm name>" -Name "<application health extension type>" 
```
---

### Customize VM watch configurations 
VM watch signals can be customized by configuring the `vmWatchSettings` properties to meet specific requirements. The following table lists the properties for `vmWatchSettings`.

#### vmWatchSettings Properties 
| **Name** | **Type** | **Description** | **Is Required**
|---|---|---|---|
| **enabled** | `bool` | This allows you to enable or disable VM watch | true
| **signalFilters** | `object` | This filters the enabled / disabled signals, either by tag or collector suite name. | false
| **parameterOverrides** | `object` | This specifies the parameters that can be overwritten for each signal execution. The full list of overwritable parameters can be found in the VM watch Collector Suites page `<link to collector page>`. | false
| **environmentAttributes** | `object` | This specifies any environment attributes that help decide if a test is eligible to execute or not. | false

> [!IMPORTANT]
> For the full list of collectors, associated signals, tags, overwritable parameters, and environment attributes, visit VM watch Collector Suites page `<insert link to collector page>` 
>
 
#### Switch On/Off VM watch 

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
> | **true**  | This setting enables VM watch |
> | **false** | This setting disables VM watch |
> 

#### Enable/Disable signal execution 

By default, only the core group signals `<link>` are enabled. However, the `signalFilters` property can be used to control and configure the signals to be executed. This property includes the following subfields.

| **Subfields** | **Description** |
|---|------|
| **enabledTags** | This enables the signals in the optional group specified with tags provided by the user |  
| **disabledTags** | This disables the signals in the core and optional groups specified with tags provided by the user | 
| **enabledOptionalSignals** | This enables signals specified in optional group. Provide collector suite name(s) as parameter | 
| **disabledSignals** | This disables the signals specified in the core and optional groups. Provide collector suite name(s) as parameter | 


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

Similarly, to enable an optional group signal with name `hardware_health_monitor`, and disable signals with name `process` and `dns`, specify such names under the `enabledOptionalSignals` and `disabledSignals` as shown below:  

```
{
   "vmWatchSettings": {
      "enabled": true,
      "signalFilters": {
         "enabledOptionalSignals": [
            "hardware_health_monitor"
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
For instance, to set the `outbound connectivity` test execution frequency to 120 seconds, specify the following configuration below:
```
{
   "vmWatchSettings": {
      "enabled": true,
      "parameterOverrides": {
         "OUTBOUND_CONNECTIVITY_INTERVAL": "120s"
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
      "parameterOverrides": {
         "DISK_IO_MOUNT_POINTS": "/mnt"
      }
   }
}

``` 

#### Environment attribute enrichments 

In addition to tags, VM watch also checks the eligibility `<insert link to collector page>` of the signals before execution. The `environmentattributes` can be specified to help VM watch determine the eligibility of each signal for execution. 
For instance, if outbound traffic has been disabled on a VM, this information can be provided to VM watch. This ensures that any outbound network-related signal execution will be marked as ineligible.
```
{
   "vmWatchSettings": {
      "enabled": true,
      "environmentAttributes": {
         "OutboundConnectivityDisabled": true
      }
   }
}
```

### Next Steps

- Link to VM watch Collector Suites Page

- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)

- [VM watch overview](/azure/virtual-machines/azure-vm-watch)

