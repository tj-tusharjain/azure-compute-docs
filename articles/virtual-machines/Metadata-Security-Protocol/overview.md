---
title: Overview of metadata security protocol
description: overview of metadata security protocol
author: minnielahoti
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 04/08/2025
ms.author: minnielahoti
ms.reviewer: azmetadatadev
---

# Metadata Security Protocol (MSP)

MSP enhances the security of the [Azure Instance Metadata Service](https://aka.ms/azureimds) and [Azure Wireserver](https://aka.ms/azureWireserver) endpoints (available in Azure IaaS VM/VMSS at 169.254.169.254 and 168.63.129.16 respectively). These cloud "metadata services" are ubiquitous across the industry with all major providers offering their own varaints.

These services are used for providing metadata and bootstrapping VM credentials. As a result, they are frequently targeted by threat actors. Common vectors include confused deputy attacks (e.g. SSRF) against in guest workloads and sandbox escapes, which are of particular concern for hosted-on-behalf-of workloads where untrusted code is intentionally loaded into the VM.

With metadata services, the trust boundary is the VM itself. Any software within the guest is authorized to request secrets from IMDS + Wireserver. VM owners are responsible for carefully sandboxing any software they run inside the VM and ensuring that external actors can't exfiltrate data. This is achievable, but in practice the complexity of the problem leads to mistakes at scale which in turn lead to exploits.

While numerous defense in depth strategies exist, providing secrets over an unauthenticated HTTP API carries inherent risk. Across the industry this class of vulnerabilities has impacted hundreds of companies, millions of individuals, and caused financial losses in the hundreds of millions of dollars. MSP closes many of the most common vulnerabilities by addressing the root cause of these attacks and introducing strong Authentication (AuthN) and Authorization (AuthZ) concepts to cloud metadata services.

## Compatibility

MSP is supported on Azure IaaS VMs + VMSS running OS based on:

- Windows 10 or later (x64; ARM64)
- Windows Server 2019 (x64, ARM64)
- Windows Server 2022 (x64, ARM64)
- Windows Server 2025 (x64, ARM64)
- Mariner 2.0 (x64, ARM64)
- Azure Linux(Mariner 3.0) (x64, ARM64)
- Ubuntu 20.04+

> Once MSP is Generally Available, the list will be expanded to include:
> - Redhat 9+
> - Flatcar
> - Rocky-Linux9+
> - SUSE 15 SP4+

The following are not supported yet:
> - Ephemeral Disks 
> - Compatability with Azure Backup 
## How to Configure MSP

Examples:

- ARM Templates
- REST API 
- PowerShell
- [Azure Portal](./other-examples/portal.md)

## Enhanced Security

### Implementation

The GPA hardens against these types of attacks by:

- Limiting metadata access to a subset of the VM (applying the principle of least privileged access).
- Switching from a "default-open" to "default-closed" model. For instance, with nested virtualization a misconfigured L2
  VM that has access to the L1 VM's vNIC can communicate with a metadata service as the L1. With the GPA, a misconfigured
  L2 would no longer be able to gain access, as it would be unable to authenticate with the service.

At provisioning time the metadata service establishes a trusted delegate within the guest (the GPA). A long-lived
secret is negotiated to authenticate with the trusted delegate, and all requests to the metadata service must be
endorsed by the delegate using an [HMAC](https://en.wikipedia.org/wiki/HMAC). This establishes a point-to-point trust
relationship with strong AuthN.

The GPA leverages [eBPF](https://ebpf.io/what-is-ebpf/) to intercept HTTP requests to the metadata services. eBPF
enables the GPA to authoritatively verify the identity of the in guest software that made the request without introducing
an additional kernel module. Using this information, it compares the identity of the client against an allow list defined
as a part of the VM model in the Azure Resource Manager (ARM) and endorses requests that are authorized by transparently
adding a signature header. This means that the feature can be enabled on existing workloads without breaking changes.

- By default, the existing authorization levels are enforced: IMDS is open to all users and Wireserver is root / admin only.
  - Today this restriction is accomplished with firewall rules in the guest. This is still a default-open mechanism,
    because if that rule can be disabled or bypassed for any reason the metadata service will accept the request. The
    AuthN mechanism enabled here default-closed. Bypassing interception maliciously or by error does not grant access to
    the metadata service.
- Advanced AuthZ configuration to authorize specific in-guest processes and users to access only specific endpoints is
  supported by defining a custom allow list with RBAC semantics.

## Getting Audit Logs

In `Audit` and `Enforce` modes, audit logs will be generated on the local disk.

| OS Family | Audit Log Location |
|--|--|
| Linux | `/var/lib/azure-proxy-agent/ProxyAgent.Connection.log` |
| Windows | `C:\WindowsAzure\ProxyAgent\Logs\ProxyAgent.Connection.log` |


## Related Content

- [MSP Configuration](./configuration.md)
- [Deploy a VM/VMSS With MSP](./greenfield.md)
- [Enable MSP on Existing VM/VMSS](./brownfield.md)
