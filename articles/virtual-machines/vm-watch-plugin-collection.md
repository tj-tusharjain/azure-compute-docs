---
# Required metadata
# For more information, see https://review.learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata?branch=main
# For valid values of ms.service, ms.prod, and ms.topic, see https://review.learn.microsoft.com/en-us/help/platform/metadata-taxonomies?branch=main

title: VM watch Plugin Collection
description: Getting started guide on VM watch Signals and Plugins
author:      ofemifowode # GitHub alias
ms.author:   ofemifowode # Microsoft alias
ms.service: azure-virtual-machines
ms.topic: get-started
ms.date:     11/05/2024
ms.subservice: monitoring
---

# VM watch Collector Suite 

VM watch collectors are designed to measure VM health in specific areas and emit signals through checks, metrics, and logs. The collector suite can help identify issues, monitor performance trends, and optimize resources to enhance the overall user experience. Below is a summary of all the available collectors in VM watch, along with the corresponding checks, metrics, logs, and their parameter configurations. Please refer to the [VM watch Signals](/azure/virtual-machines/azure-vm-watch) page for the description of each check, metric and log.

### Pre-requisites
This article assumes that you are familiar with:
- VM watch [Signals](/azure/virtual-machines/azure-vm-watch) and their descriptions
- [Installing VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2) to Virtual Machines and Scale Sets

> [!NOTE]
> | **Name**  | **Description** |
> |---|---|
> | **Collector** | This is a logical grouping of similar "" where you can collect checks, metrics and logs to determine the health of a particular resource |
> | **Group** | This indicates whether the collectors are part of the core or optional group. Core group collectors are enabled by default, while Optional group collectors can be enabled or disabled based on the user's requirements |
> | **Tags** | This is used to categorize and filter checks, metrics and logs |
> | **Overwritable Parameters** | These are associated parameters that can be customized to override the default configuration |
> | **Eligibility** | This determines whether a collector is eligible to be executed based on the environment attributes specified by the user |
> | **Default Behavior** | This is the standard setting and action that would be followed if no custom configurations are provided. |


### Groups, Tags and Corresponding Checks, Metrics and Event Logs

| Collector Name | Group | Tags | Checks | Metrics | Event Logs |
|---|---|---|---|---|---|
| outbound_connectivity| Core|Network| <ul><li>outbound_connectivity</li></ul>|||
| dns| Core|Network| <ul><li>dns</li> </ul>|||
| tcp_stats | Core|Network| | <ul><li>SegmentsRetransimitted</li> <li>NormalizedSegmentsRetransimitted</li>  <li>ConnectionResets</li> <li>NormalizedConnectionResets</li> <li>FailedConnectionAttempts</li> <li> NormalizedFailedConnectionAttempts </li><li> ActiveConnectionOpenings </li><li> PassiveConnectionOpenings </li><li> CurrentConnections </li><li> SegmentsReceived </li><li> SegmentsSent </li> </ul>||
| clock_skew | Core|Clock| <ul><li>clockskew</li> </ul>|||
| disk_io | Core|Disk|<ul><li>disk_io</li> </ul>| <ul><li>UsedSpaceInBytes</li>  <li>FreeSpaceInBytes</li>  <li>CapacityInBytes</li>  <li>UsedPercent</li> </ul>||
| disk_iops  | Core|Disk||<ul> <li>WriteOps</li> <li>ReadOps</li> </ul>||
| imds | Core|IMDS| <ul> <li>imds</li> </ul>|||
| process  | Core|Process| <ul> <li>process</li> </ul>|||
| process_cpu   | Core|Process| |<ul> <li>ProcessCoreUsage</li> <li>ProcessMachineUsage</li> <li>MachineTotalCpuUsage</li></ul>||
| process_monitor   | Optional|Process|<ul> <li>process_monitor</li> </ul>|<ul> <li>UpTime</li> </ul>||
| az_storage_blob   | Optional|AzBlob| <ul> <li>az_storage_blob</li> </ul>|||
| hardware_health_monitor   | Optional|Hardware| | | <ul> <li>hardware_health_monitor</li> </ul>|



### Eligibility, Default Behavior and Overwritable Parameters
| Collector Name | Eligibility | Default Behavior | Overwritable Parameters |
|---|---|---|---|
| outbound_connectivity| Eligible if EnvironmentAttribute "OutboundConnectivityDisabled" is not set or set to "false" |This collector is executed every 60s. In each execution, it sends an http GET request to http://www.msftconnecttest.com/connecttest.txt with a timeout of 5s. If the request fails, it retries at most 2 more times with and internval of 10s. The verification is marked as "Failed" if all the retries fail.                                                                                                                                                               | <ul> <li>OUTBOUND_CONNECTIVITY_INTERVAL: the execution interval of the Collector. Default: 60s</li> <li>OUTBOUND_CONNECTIVITY_URLS: the URLs that this Collector sends http GET requests to. URLs are provided as a string using `,` as separator. Default: http://www.msftconnecttest.com/connecttest.txt</li> <li>OUTBOUND_CONNECTIVITY_TIMEOUT_IN_MILLISECONDS: the http GET request timeout in milliseconds. Default: 5000</li> <li>OUTBOUND_CONNECTIVITY_TOTAL_ATTEMPTS: the total number of attempts to send http request if the previous one fails. Default: 3</li> <li>OUTBOUND_CONNECTIVITY_RETRY_INTERVAL_IN_SECONDS: the retry interval in seconds if the previous http request fails. Default: 10</li> </ul> |
| dns| Eligible if EnvironmentAttribute "OutboundConnectivityDisabled" is not set or set to "false" |This Collector is executed every 180s. In each execution, it tries to resolve the DNS name `www.msftconnecttest.com` . The verification is marked as "Failed" if the DNS name cannot be resolved.                                                                                                                                                                                                                                                                            | <ul> <li>DNS_INTERVAL: the execution interval of the Collector. Default: 180s</li>  <li>DNS_NAMES: the DNS names to be resolved separated by `,`. Default: www.msftconnecttest.com</li> </ul>|
| tcp_stats| Always eligible  |This Collector is executed every 180s. In each execution, it collects the TCP statistics of the last 180s.                                                                                                                                                                                                                                                                                                                                                                   | <ul> <li>TCP_STATS_INTERVAL: the execution interval of the Collector. Default: 180s</li>  </ul> |
| clock_skew| Eligible if EnvironmentAttribute "OutboundConnectivityDisabled" is not set or set to "false"|This Collector is executed every 180s. In each execution, it retrieves the clock offset between the remote NTP server `time.windows.com` and the VM. The verification is marked as "Failed" if the clock skew is larger than 5.0 seconds. In Windows VM, if connecting to remote NTP server fails, it fallbacks to check Windows Time Service with w32tm command. The verification is marked as "Failed" if the w32tm command returns "Leap Indicator: 3(not synchronized)". | <ul> <li>CLOCK_SKEW_INTERVAL: the execution interval of the Collector. Default: 180s</li> <li>CLOCK_SKEW_NTP_SERVER: the remote NTP server used to calculate clock skew. Default: time.windows.com</li> <li>CLOCK_SKEW_TIME_SKEW_THRESHOLD_IN_SECONDS: the threshold in seconds of clock offset to mark the verification as "Failed". Default: 5.0</li> </ul> |
| disk_io| Always eligible if mount points are not specified. If mount points are explicitly specified, only eligible when data disks are attached to the VM |This Collector is executed every 180s. In each execution, it verifies the disk io availability in each available mount point by creating a folder, creating a file, writing bytes to it, deleting it and delete the folder. Then it collects the disk usage info indcluding used space, free space, total capacity and used percentage from each mount point.                                                                                                                | <ul> <li>DISK_IO_INTERVAL: the execution interval of the Collector. Default: 180s</li> <li>DISK_IO_MOUNT_POINTS: the mount points separated by `,`. No default value</li> <li>DISK_IO_IGNORE_FS_LIST: the file system list that should be ignored separated by `,`. Default: tmpfs,devtmpfs,devfs,iso9660,overlay,aufs,squashfs,autofs</li> <li>DISK_IO_FILENAME: the name of the file used to verify the file read/write. Default: vmwatch-{timestamp}.txt </li> </ul>   |
| disk_iops| Always eligible |This Collector is executed every 180s. In each execution, it collects the disk read and write operations per second metrics from each available disk device.                                                                                                                                                                                                                                                                                                                 | <ul> <li>DISK_IOPS_INTERVAL: the execution interval of the Collector. Default: 180s</li> <li>DISK_IOPS_DEVICES: the device names separated by `,`. No default value</li> <li>DISK_IOPS_IGNORE_DEVICE_REGEX: the regex of the device name that should be ignored. Default: loop</li> </ul>  |
| imds| Always eligible|This Collector is executed every 180s. In each execution, it queries the IMDS endpoint http://169.254.169.254/metadata/instance/compute and verifies the response body contains the information (SubscriptionId, ResourceGroup, VMId, ResourceId) of the VM. The query timeout is 10s. If the query fails, it will retry at most another 3 more times with an interval of 15s, 30s and 45s.                                                                                   | <ul> <li>IMDS_INTERVAL: the execution interval of the Collector. Default: 180s</li> <li>IMDS_ENDPOINT: the URL of the IMDS endpoint. Default:http://169.254.169.254/metadata/instance/compute</li> <li>IMDS_TIMEOUT_IN_SECONDS: the timeout in seconds of each query. Default: 10</li> <li>IMDS_QUERY_TOTAL_ATTEMPTS: the total number of attempts to send http request if the previous one fails. Default: 4</li> <li>IMDS_RETRY_INTERVAL_IN_SEONDS: the retry interval in seconds if the previous http request fails. Default: 15, 30, 45</li> </ul> |
| process| Always eligible|This Collector is executed every 180s. In each execution, it creates and executes command `${SYTEM_DIR}\system32\cmd.exe /c echo hello` in Windows machine and `/bin/sh -c echo hello` in Linux machine. The timeout of process execution is 10s.                                                                                                                                                                                                                            | <ul> <li>PROCESS_INTERVAL: the execution interval of the Collector. Default: 180s</li>  <li>PROCESS_TIMEOUT: the timeout of process execution. Default: 10s</li> </ul>|
| process_cpu| Always eligible|This Collector is executed every 180s. In each execution, it selects the top 3 processes with the most CPU usage and reports the ProcessCoreUsage, ProcessMachineUsage and MachineTotalCpuUsage.                                                                                                                                                                                                                                                                             | <ul> <li>PROCESS_CPU_INTERVAL: the execution interval of the Collector. Default: 180s</li>  </ul>|
| process_monitor| Always eligible|Not executed.                                                                                                                                                                                                                                                                                                                                                                                                                                                             | <ul> <li>PROCESS_MONITOR_INTERVAL: the execution interval of the Collector. Default: 180s</li>  <li>PROCESS_MONITOR_PROCESS_NAMES: the process names to be monitored separated by `,`. No default value</li> </ul>     |
| az_storage_blob| Eligible if EnvironmentAttribute "OutboundConnectivityDisabled" is not set or set to "false"  |Not executed.                                                                                                                                                                                                                                                                                                                                                                                                                                                             | <ul> <li>AZ_STORAGE_BLOB_INTERVAL: the execution interval of the Collector. Default: 180s</li> <li>AZ_STORAGE_ACCOUNT_NAME: the Azure Storage account name. No default value</li> <li>AZ_STORAGE_CONTAINER_NAME: the Azure Storage Container name. No default value</li> <li>AZ_STORAGE_BLOB_NAME: the Azure Storage Blob name. No default value</li> <li>AZ_STORAGE_BLOB_DOMAIN_NAME: the Azure Storage domain name. No default value</li> <li>AZ_STORAGE_SAS_TOKEN_BASE64: the Base64 encoded Azure Storage SAS token. No default value</li> <li>AZ_STORAGE_USE_MANAGED_IDENTITY: if the managed identity will be used for authentication. Default: false</li> <li>AZ_STORAGE_MANAGED_IDENTITY_CLIENT_ID: the managed identity client id for authentication. No default value</li> </ul>|
| hardware_health_monitor| Eligible in Windows machine|Not executed.                                                                                                                                                                                                                                                                                                                                                                                                                                                             | <ul> <li>HARDWARE_HEALTH_MONITOR_INTERVAL: the execution interval of the Collector. Default: 180s</li>  </ul> |


### Next Steps

- Link to Configure VM watch Page

- [Install VM watch](/azure/virtual-machines/install-vm-watch?tabs=ARM-template-1%2Ccli-2)

- [VM watch overview](/azure/virtual-machines/azure-vm-watch)