---
title:       Configure EventHubs for Vm watch
description: Get started to configure EventHubs for VM watch
author:      ofemifowode
ms.author:   ofemifowode 
ms.service:   azure-virtual-machines
ms.topic:     get-started
ms.date:     02/05/2025
---

# Configure EventHubs for VM watch

VM watch can send signal data to a pre-configured [Event Hub](/azure/event-hubs/event-hubs-about).

This article describes how to configure an EventHub in order to access signals collected by VM watch.

### Pre-requisites
This article assumes that you are familiar with:
- [Azure Event Hubs](/azure/event-hubs/event-hubs-about)
- [VM watch Checks, Metrics and Logs](/azure/virtual-machines/azure-vm-watch)
- [Installing VM watch to Virtual Machines and Scale Sets](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2) 

### Enable EventHubs Output

#### Step 1: Prepare an EventHub for VM watch
- Deploy an [Event Hub](/azure/event-hubs/event-hubs-create)
- [Authorize access to the Azure Event Hub](/azure/event-hubs/authorize-access-event-hubs)
> [!IMPORTANT]
> VM watch supports managed identity, SAS token and connection string as authentication methods. Please be aware that if multiple authentication methods are provided, managed identity will be given the highest priority, while the connection string will be given the lowest priority

#### Step 2: Enable Event Hub output for VM watch

For each Event Hub authentication method, you will need to combine both the common and authentication specific parameter settings. Instructions are given below for each authentication scenario. To apply this to Virtual Machines and Virtual Machine Scale Sets, specify the following settings within `vmWatchSettings` in the JSON configurations. See Configure VM watch `<insert link to configure vm watch page>` for instructions on how to access `vmWatchSettings` using [ARM template](/azure/azure-resource-manager/templates/), [AZ CLI](/cli/azure/) or [Powershell](/powershell/).

##### Common parameters for authentication methods

For all authentication methods, the following parameter set applies:

|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_NAMESPACE`|Yes|Event hub name space name, without the domain name `".servicebus.windows.net"`|
|`EVENT_HUB_OUTPUT_NAME`|Yes|Event hub name within the given namespace|
|`EVENT_HUB_OUTPUT_DOMAIN_NAME`|No|Event hub domain name. Default value `"servicebus.windows.net"`|
|`EVENT_HUB_OUTPUT_CLOSE_TIMEOUT`|No|Client close timeout. Default is 30s|
|`EVENT_HUB_OUTPUT_PARTITION_ID`|No|Metric tag or field name to use for the event partition key. Default is null|
|`EVENT_HUB_OUTPUT_MAX_MESSAGE_SIZE`|No|The maximum batch message size in bytes. Setting this to 0 means using the default size from the Azure Event Hubs Client library (1000000 bytes). Default is 0|
|`SEND_INTERNAL_TELEMETRY_TO_EVENT_HUB`|No|To receive VM watch internal metrics (startup and heartbeat events), set this value to "true". Default is "false"|


#### [Managed Identity](#tab/managedidentity-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY`|No|Set this value to "true". Default is "false"|
|`EVENT_HUB_OUTPUT_MANAGED_IDENTITY_CLIENT_ID`|No|If you are using a specific managed identity to authenticate, specify this value|

For example, the following VM watch JSON configuration sets the environment variables `EVENT_HUB_OUTPUT_NAMESPACE`, `EVENT_HUB_OUTPUT_NAME`, and `EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY`. This allows the Event Hub to use managed identity as the authentication method without needing to specify a managed identity client ID.

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_USE_MANAGED_IDENTITY": "true"
    }
  }
}
```

#### [SAS Token](#tab/sastoken-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_SAS_TOKEN_BASE64`|No|If using SAS token to authenticate, specify this value|

For example, the following VM watch JSON configuration enables Event Hub as an output by using a SAS token for authentication.

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_SAS_TOKEN_BASE64": "<base 64 encoded SAS token>"
    }
  }
}
```

#### [Connection String](#tab/connectionstring-1)
|**Parameter**|**Is required**|**Description**|
| -------- | -------- | -------- |
|`EVENT_HUB_OUTPUT_CONNECTION_STRING_BASE64`|No|If using connection string token to authenticate, encode it with base64.|

For example, the following VM watch JSON configuration enables Event Hub as an output by using a connection string for authentication

```
{
  "vmWatchSettings": {
    "enabled": true,
    "parameterOverrides": {
      "EVENT_HUB_OUTPUT_NAMESPACE": "<example event hub namespace>",
      "EVENT_HUB_OUTPUT_NAME": "<example event hub name>",
      "EVENT_HUB_OUTPUT_CONNECTION_STRING_BASE64": "<base 64 encoded connection string>"
    }
  }
}
```
---


#### Step 3: Examine Events in Event Hub

Once VM watch settings have been successfully configured to use Event Hub as the output, VM watch will restart. Events will start flowing into Event Hub within a few minutes. You can use the [Azure Portal](https://portal.azure.com/) to observe the incoming messages. 
The following screenshot shows data flowing into the Event Hub

Also, you can use the [Event Hub's Data Explorer](/azure/event-hubs/event-hubs-data-explorer) feature to view incoming event and content, as shown in the screenshot below.




#### Event Hub Events Schema

Each Event Hub event has the following schema:

|**Field Name**|**Data Type**|**Description**|
| -------- | -------- | -------- |
|`DateTime`|time|The time this signal was emitted|
|`SignalType`|string|The type of this signal, which can be either "StartUp", "Heartbeat", "Check", "Metric" or "EventLog"|
|`SignalName`|string|The name of this signal|
|`SubscriptionId`|string|The VM subscription Id|
|`ResourceGroup`|string|The VM resource group name|
|`ResourceId`|string|The azure resource URI of the VM|
|`VmId`|string|The unique Id of the VM queried from IMDS endpoint within the VM|
|`Vmss`|string|The VMSS name, if applicable|
|`Offer`|string|The Azure VM offer|
|`VmSize`|string|The VM size|
|`MeasurementTarget`|string|The target that the Signal is measuring at. `Name` field and `MeasurementTarget` are used together for raw Signal aggregation|
|`SignalValue`|json|The value of this signal, where the schema depends on the SignalType.
|`Version`|string|The Event Hub output event schema version|

#### Debug Event Hub connection issues

If there are no events in Event Hub after several minutes, check the VM watch logs in the following directories on the Virtual Machine or Virtual Machine Scale Set to diagnose the issue:

#### [Linux](#tab/linux-1)
```
/var/log/azure/Microsoft.ManagedServices.ApplicationHealthLinux/vmwatch.log
```
#### [Windows](#tab/windows-1)
```
C:/WindowsAzure/Logs/Plugins/Microsoft.ManagedServices.Edp.ApplicationHealthWindows/vmwatch.log
```
---

### Next Steps

- Configure VM watch 

- VM watch Collector Suites 

- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)

- [VM watch overview](/azure/virtual-machines/azure-vm-watch)
