# Troubleshooting Guide

## New VM/VMSS deployment failures

### Windows

In Windows, the necessary components (GPA and eBPF for Windows) will be automatically installed by the platform. If this fails:

1. Ensure you are using a [supported OS version]() <- link to compatibility matrix here.
1. Ensure the failure code is [MSP related]() <- Link to possible errors
1. Retry the deployment. Transient failures are a part of cloud computing.
1. If the issue persists, please contact support.

### Linux

Ensure that you are using a [valid image]()

- The GPA must be baked into your image in order to deploy with MSP enabled.
- Your cloud-init version must be a newer, MSP aware version. If it is not, a race condition will occur between it and the GPA.

The exact failure will depend on if/how your image is misconfigured or if a platform failure occured:

| GPA Installed | Cloud-init Version | Expected Failure | Cause | 
|--|--|--|--|
| No| `< 24.3`| `Linux.OSProvisioningInternalError`<br>Linux.cloud-init successfully reported ready for provisioning but the platform failed to record success | VM may fail to provision even though cloud-init reports ready. |  
| No | `>= 24.3`| Linux.Cloud-Init failed due to azure-proxy-agent not found | Cloud-init reports failure to platform after detecting GPA not installed. |  
| Yes | `< 24.3`| DID NOT LATCH ERROR HERE | Cloud-init may report ready before GPA is configured as it is GPA-unaware.  May fail up to 100% of the time depending on scenario. | 
| Yes | `>= 24.3`| Cloud-init reports GPA is unhealthy | Any of: eBPF setup failure, Cgroups v2 not enabled, generic startup failure, failure to acquire key |  

## MSP enabled, but not applied in existing VM/VMSS

To avoid service disruptions when enabling MSP on an existing VM/VMSS, protections will not be applied until the VM has indicated it has successfully set up and acquired the long-lived key. This means the VM model can show that MSP is enabled, but the GPA service may still be unhealthy and indicate that protections are not activated.

### Confirming the issue still exists

Sometimes the delay is higher than anticipated. To begin, check the status report of the GPA service:

#### 1. Get status.json file: 

Windows VM:  
ProxyAgent logs from inside the VM as they have more detailed logs, the logs folder is `C:\WindowsAzure\ProxyAgent\Logs`

Status.json
ProxyAgent service captured its overall status into `C:\WindowsAzure\ProxyAgent\Logs\status.json`

Linux VM:
ProxyAgent logs from inside the VMs as they have more details, the logs folder is `/var/log/azure-proxy-agent/`

status.json
azure-proxy-agent service captures its overall status into `/var/log/azure-proxy-agent/status.json`

#### 2. Check `proxyAgentStatus` in  the json file and ensure it's "SUCCESS". If it's not "SUCCESS" check these other statuses:

|Status| Expected value|
|--|--|
|`keyLatchStatus`| "RUNNING" |
| `ebpfProgramStatus`| "RUNNING"|
|`proxyListenerStatus`|"RUNNING"|

### GPA service missing or not running
#### Windows VM
For Windows VMs, CRP implicitly installs an extension "Microsoft.CPlat.ProxyAgent.ProxyAgentWindows" that installs Windows Guest Proxy Agent (GPA)to the VM for MSP feature. 

When enabling MSP, it installs 3 Windows Services to the VM:
- Windows Guest Proxy Agent (GPA) service (**GuestProxyAgent**)
- ebpf-for-windows kernel drivers: **eBPFCore**, **NetEbpfExt**

#### Linux VM
For Linux VMs, Linux Guest Proxy Agent (GPA) must be baked into its base image, or the user explicitly add GPA VM Extension `Microsoft.CPlat.ProxyAgent.ProxyAgentLinux` before enable MSP feature.
#Prerequisites
- Linux Kernel 5.15 or later, which has all our required eBPF features;
- cgroup2 mounted by default, GPA service hooks up cgroup/connect4 eBPF event.

When enabling MSP, it installs one service `azure-proxy-agent` to the VM.
```
# systemctl status azure-proxy-agent.service 
● azure-proxy-agent.service - Microsoft Azure GuestProxyAgent
     Loaded: loaded (/lib/systemd/system/azure-proxy-agent.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-09-13 18:52:00 UTC; 1 week 5 days ago
   Main PID: 4040671 (azure-proxy-age)
      Tasks: 10 (limit: 19169)
     Memory: 491.2M
        CPU: 1d 8h 13min 2.321s
     CGroup: /system.slice/azure-proxy-agent.service
             └─4040671 /usr/sbin/azure-proxy-agent
```
### GPA service is running, but unable to set up

#### eBPF setup failed

Windows VM

eBPFCore and NetEbpfExt service must in started (RUNNING) state, or <ask windows eBPF team for details>,
```
c:\>sc query eBPFCore

SERVICE_NAME: eBPFCore
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```
```
c:\>sc query NetEbpfExt

SERVICE_NAME: NetEbpfExt
        TYPE               : 1  KERNEL_DRIVER
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Linux VM:

#### Failed to load eBPF program

GPA uses some eBPF features which requires Linux kernel 5.15 or later. If the customer is trying to enable the MSP feature on an unsupported Linux distro version, you may see the message similar to the one below in status.json:

```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to load program 'connect4' with error: the BPF_PROG_LOAD syscall failed. Verifier output: ; int connect4(struct bpf_sock_addr *ctx)\n0: (bf) r6 = r1\n; __u64 cookie = bpf_get_socket_cookie(ctx);\n1: (85) call bpf_get_socket_cookie#46\n2: (b7) r1 = 0\n; destination_entry entry = {0};\n3: (63) *(u32 *)(r10 -44) = r1\nlast_idx 3 first_idx 0\nregs=2 stack=0 before 2: (b7) r1 = 0\n4: (63) *(u32 *)(r10 -48) = r1\n5: (63) *(u32 *)(r10 -52) = r1\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n6: (61) r1 = *(u32 *)(r6 +4)\n; entry.destination_ip.ipv4 = ctx->user_ip4;\n7: (63) *(u32 *)(r10 -56) = r1\n; entry.destination_port = ctx->user_port;\n8: (61) r1 = *(u32 *)(r6 +24)\n; entry.destination_port = ctx->user_port;\n9: (63) *(u32 *)(r10 -40) = r1\n; entry.protocol = ctx->protocol;\n10: (61) r1 = *(u32 *)(r6 +36)\n; entry.protocol = ctx->protocol;\n11: (63) *(u32 *)(r10 -36) = r1\n12: (bf) r2 = r10\n; \n13: (07) r2 += -56\n; destination_entry *policy = bpf_map_lookup_elem(&policy_map, &entry);\n14: (18) r1 = 0xffff8baa17403400\n16: (85) cal..."
    },
```
Try get the kernel version using `uname -r`, if it display lower than 5.15, please opt-out MSP feature, upgrade your OS image to latest version and enable MSP feature.

#### Failed to attach cgroup program for redirection

Guest ProxyAgent requires cgroup2 mounted by default to hooks up cgroup/connect4 eBPF event. If the customer trying to enable the MSP feature on an unsupported Linux distro version, you may see the message like:
- status.json
```
    "ebpfProgramStatus": {
      "status": "STOPPED",
      "message": "Failed to attach program 'connect4' with error: `bpf_link_create` failed"
    },
```
- Check if current Linux VM supports CGroup2 using `grep cgroup /proc/filesystems`,
```
grep cgroup /proc/filesystems
``` 
If `nodev cgroup2` is listed, it means cgroup v2 is supported by this OS; if not listed, please opt-out MSP feature, update to this latest OS version and then enable MSP feature.
- Check if CGroup2 is mounted by default using `mount | grep cgroup2`
```
mount | grep cgroup2
```
If there is no record displayed, ask the VM owner to setup it with command below and then reboot the VM.
```
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

#### Failed to acquire key / key rejected

The GPA was not able to acquire a key, because the platform is no longer offering it. This can be caused by actions like migrating a disk to a new VM; or deleting the VMs OS disk and replacing it.

Follow the Key recovery / key reset instructions. Resetting the key will allow the platform to offer a new one. The GPA periodically attempts to recover. Once the reset is completed, the GPA will automatically acquire a new key and go back to a healthy state.

## Key recovery / key reset

If your VM's long-lived key is lost it will no longer be able to communicate with IMDS nor Wireserver. Without the key, a new one cannot be safely issued to the VM automatically. Additionally, if the key is compromised in some way it must be reset to ensure security is maintained.

### Resetting a Key

1. Refer to <configuration link here> for complete documentation on the `KeyIncarnationId` filed of the `ProxyAgentSettings` section of the VM model.
1. Select a new `KeyIncarnationId` value, and apply it to the VM model with a `PUT` operation

> Note: If sending multiple requests or otherwise using automation you must ensure that the values you send are strictly monotonically increasing or else no change may be applied. For more details see <link to general concept if one indeed exists>

#### Example

```text
POST <URL here>
<headers here?>
<body here>
```
