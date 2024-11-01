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



## Customize VM Watch configuration 

By default, only all tests inside [Core Group](https://dev.azure.com/msazure/One/_git/VMWatch?version=GBmain&_a=contents&path=/infra/components/metadata.json) are enabled with [default configurations](https://dev.azure.com/msazure/One/_git/VMWatch?version=GBmain&_a=contents&path=/vmwatch.conf). Each test is executed at an interval of every three minutes. However, these default settings can be easily overridden through the "vmWatchSettings" using either the [ARM template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/) or [AZ CLI](https://learn.microsoft.com/en-us/cli/azure/) as shown below.

```
"settings": { 
"vmWatchSettings": { 
"enabled": true 
   } 
} 
```

### vmWatchSettings Fields 

| **Name** | **Type** | **Description** |
|---|---|---|
| **enabled** | bool | This shows whether VM watch is enabled or not |
| **signalFilters** | object | This filters the enabled / disabled signals, either by tag or signal name. |
| **parameterOverrides** | object | This specifies the parameters to overwrite for each signal execution. The default parameters can be found in each signal's description page. |
| **environmentAttributes** | object | This specifies any environment attributes to help decide if a test is eligible to execute or not. This is an optional setting |

 

### Switch On / Off VM watch 

VM watch can be switched on / off by setting the "enabled" configuration.  

```
{ 

"vmWatchSettings": { 

"enabled": true 

} 

} 
```
| **Name** | **Type** | **Description** |
|---|---|---|
| **True** | bool | This setting enables VM watch | 
| **False** | bool | This setting disables VM watch | 


### Enable/disable test execution 

Customers can control the signals to be executed with "signalFilter" field. This field contains the following subfields: 

| **Subfield** | **Description** |
|---|---|---|
| **enabledTags** | Use this to specify the signals with these tags in Optional Group to be enabled | 
| **disabledTags** | This specifies the signals with these tags in Core and Optional Groups to be disabled | 
| **enabledOptionalSignals** | This specifies signal names in Optional Group to be enabled | 
| **disabledSignals** | This specifies the signal names in Core and Optional Groups to be disabled | 


"enabledTags": specify the signals with these tags in Optional Group to be enabled 

"disabledTags": specify the signals with these tags in Core and Optional Groups to be disabled 

"enabledOptionalSignals": specify signal names in Optional Group to be enabled 

"disabledSignals": specify the signal names in Core and Optional Groups to be disabled 

The signals applied by "signalFilter" is called Candidate signals: 

**Candidate Signals = all Core group signals + signals in Optional group with tag enabled - signals with tag disabled + explicitly enabled signals in Optional group - explicitly disabled signals** 

For example, to add signals in Optional group containing "Availability" tag to the execution, remove signals containing "Disk" tag, enable an Optional group signal with name "simple", and disable signals with name "process" and "dns", customer can provide the below configuration: 

{ 

"vmWatchSettings": { 

"enabled": true, 

"signalFilters": { 

"enabledTags": [ 

"Availability" 

], 

"disabledTags": [ 

"Disk" 

], 

"enabledOptionalSignals": [ 

"simple" 

], 

"disabledSignals": [ 

"process", 

"dns" 

] 

} 

} 

} 

 

### Configure test execution frequency 

The test execution frequency can be customized by adjusting the 'parameterOverrides' field. Using the "_interval" fter the name of the test and specify the new time duration. For instance, to set the outbound connectivity test execution frequency to 60 seconds, the following configuration can be used.
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
 

Override Default signal execution parameters 

Signal execution parameters can be overwritten by setting the "parameterOverrides" field as well. For example, to set disk io signal mount point to "/mnt", customer can specify the below configuration: 
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

Override Default signal name 

When the customer override the default signal parameters, it is suggested to override the signal name as well by providing a suffix. This will help the downstream to aggregate the signals based on different parameters. For example, besides setting disk io signal mount point to "/mnt", customer can also annotate this signal with name "disk_io_mnt" with the below configuration: 

{ 

"vmWatchSettings": { 

"enabled": true, 

"parameterOverrides ": { 

"DISK_IO_MOUNT_POINTS": "/mnt", 

"DISK_IO_SUFFIX_NAME": "\_mnt" 

} 

} 

} 

 

 

Environment Attribute Enrichments 

Customers can specify environment attributes to assist VM watch in determining the eligibility of each test for execution. For instance, if a customer knows that the VM has outbound traffic disabled, this information can be provided to VM watch. This will help VM watch mark any outbound network-related test execution as ineligible."
```
{ 

"vmWatchSettings": { 

   "enabled": true, 

   "environmentAttributes ": { 

       "OutboundConnectivityEnabled": false 

       } 

   } 

} 
```

