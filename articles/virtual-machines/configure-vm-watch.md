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



### Customize VM Watch configuration 

VM watch signals can be categorized into two groups: Core and Optional. By default, only core group <link to plugin page> signals are enabled with default configurations <link to plugin page>. Each signal is executed at an interval of every three minutes. However, customers can easily overwrite these default settings from the "vmWatchSettings" using either the [ARM template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/) or [AZ CLI](https://learn.microsoft.com/en-us/cli/azure/).


To access "vmWatchSettings" for ARM Template, navigate back to the Overview Page on [Azure portal](https://portal.azure.com/) and click on the JSON view for the VM to find the code segment below. 

```
"settings": { 
"vmWatchSettings": { 
"enabled": true 
   } 
} 
```

For AZ CLI, customers can run the following command to access "vmWatchSettings".

```
az vm extension show -g <your resource group name> --vm-name <your vm name> -n <application health extension type>
```
> [!IMPORTANT]
> The code segment is identical for both Windows and Linux except for the value of the parameter <application health extension type> passed into the Extension Type.
> 
> Please replace <application health extension type> with "ApplicationHealthLinux" for Linux and "ApplicationHealthWindows" for Windows installations.  


### vmWatchSettings Fields 

| **Name** | **Type** | **Description** |
|---|---|---|
| **enabled** | bool | This shows whether VM watch is enabled or not |
| **signalFilters** | object | This filters the enabled / disabled signals, either by tag or signal name. |
| **parameterOverrides** | object | This specifies the parameters to overwrite for each signal execution. The default parameters can be found in the VM watch plugin collection page <link to plugin page>. |
| **environmentAttributes** | object | This specifies any environment attributes to help decide if a test is eligible to execute or not. This setting is optional |

 

#### Switch On / Off VM watch 

VM watch can be switched on / off by setting the "enabled" configuration.  

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

#### Enable / disable test execution 

Customers can control the signals to be executed with "signalFilter" field. This field contains the following subfields: 

| **Subfields** | **Description** |
|---|---|
| **enabledTags** | This will enable the signals in the optional group specified with these tags | 
| **disabledTags** | This will disable the signals in the core and optional groups specified with these tags | 
| **enabledOptionalSignals** | This will enable signals specified in optional group. Provide signal name(s) as parameter | 
| **disabledSignals** | This will disable the signals specified in the core and optional groups. Provide signal name(s) as parameter | 


For instance, to add signals in optional group containing "Network" tag to the execution, remove signals containing "Disk" tag, customers can provide the configuration below: 
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

Subsequently, to enable an optional group signal with name "hardware", and disable signals with name "process" and "dns", customers can provide the configuration below:   

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

> [!NOTE]
> Please find below the existing tags and corresponding VM watch signals
> | **Existing tags**  | **Signals** |
> |---|---|
> | **Network**  | outbound_connectivity, dns, tcp_stats |
> | **Clock** | clockskew |
> | **Disk** | disk_io, disk_iops |
> | **IMDS** | imds |
> | **Process** | process, process_cpu, proces_monitor |
> | **AZBlob** | az_storage_blob |
> | **Hardware** | hardware_health_monitor |
>  

#### Configure signal execution frequency 

The signal execution frequency can be customized by adjusting the "parameterOverrides" field. To configure the frequency, customers should append "_INTERVAL" to the signal name and specify the desired time duration. 
For instance, to set the outbound connectivity test execution frequency to 60 seconds, the following configuration can be used.
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

Signal execution parameters can be overwritten by setting the "parameterOverrides" field as well. For instance, to set "disk_io" signal mount point to "/mnt", customers can specify the configuration below: 
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

In addition to tags, VM watch also checks the eligibility <insert link to plugin page> of the signal before execution. Customers can specify environment attributes to help VM watch determine the eligibility of each signal for execution. 
For instance, if a customer knows that the VM has outbound traffic disabled, they can provide this information to VM Watch. This will ensure that any outbound network-related signal execution is marked as ineligible.
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

