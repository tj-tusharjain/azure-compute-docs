---
title: Trusted Launch for Azure VMs
description: Learn about Trusted Launch for Azure virtual machines.
author: AjKundnani
ms.author: ajkundna
ms.service: azure-virtual-machines
ms.subservice: trusted-launch
ms.topic: conceptual
ms.date: 11/06/2023
ms.reviewer: jushiman
ms.custom: template-concept; references_regions
---

# Trusted Launch for Azure virtual machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure offers Trusted Launch as a seamless way to improve the security of [Generation 2](generation-2.md) virtual machines (VM). Trusted Launch protects against advanced and persistent attack techniques. Trusted Launch is composed of several coordinated infrastructure technologies that can be enabled independently. Each technology provides another layer of defense against sophisticated threats.

> [!IMPORTANT]
>
> - Trusted Launch is selected as the default state for newly created Azure VM. If your new VM requires features that aren't supported with Trusted launch, see the [Trusted Launch FAQs](trusted-launch-faq.md).
> - [Existing VM](overview.md) can have Trusted Launch enabled after being created. For more information, see [Enable Trusted Launch on existing VMs](trusted-launch-existing-vm.md).
> - Existing [virtual machine scale set](../virtual-machine-scale-sets/overview.md) can have Trusted Launch enabled after being created. For more information, see [Enable Trusted Launch on existing scale set](trusted-launch-existing-vmss.md).

## Benefits

- Securely deploy VMs with verified boot loaders, operating system (OS) kernels, and drivers.
- Securely protect keys, certificates, and secrets in the VMs.
- Gain insights and confidence of the entire boot chain's integrity.
- Ensure that workloads are trusted and verifiable.

## Virtual machines sizes

| Type | Supported size families | Currently not supported size families | Not supported size families
|:--- |:--- |:--- |:--- |
| [General purpose](./sizes/overview.md#general-purpose) | [B-family](./sizes/general-purpose/b-family.md), [D-family](./sizes/general-purpose/d-family.md), [Dpsv5-series](./sizes/general-purpose/dpsv5-series.md), [Dpdsv5-series](./sizes/general-purpose/dpdsv5-series.md), [Dplsv5-series](./sizes/general-purpose/dplsv5-series.md), [Dpldsv5-series](./sizes/general-purpose/dpldsv5-series.md), [A-family](./sizes/general-purpose/a-family.md), [Dv2-series](./sizes/general-purpose/dv2-series.md), [Dv3-series](./sizes/general-purpose/dv3-series.md), [DC-Confidential-family](./sizes/general-purpose/dc-family.md) | All sizes supported. |  | 
[Compute optimized](./sizes/overview.md#compute-optimized) | [F-family](./sizes/compute-optimized/f-family.md), [Fx-family](./sizes/compute-optimized/fx-family.md) | All sizes supported. | 
| [Memory optimized](./sizes/overview.md#memory-optimized) | [E-family](./sizes/memory-optimized/e-family.md), [Eb-family](./sizes/memory-optimized/eb-family.md), [M-family](./sizes/memory-optimized/m-family.md), [EC-Confidential-family](./sizes/memory-optimized/ec-family.md)  |  [ECas_cc_v5, ECads_cc_v5 series](./sizes/memory-optimized/ec-family.md)  |
| [Storage optimized](./sizes/overview.md#storage-optimized) | [L-family](./sizes/storage-optimized/l-family.md) | All sizes supported. | 
| [GPU](./sizes/overview.md#gpu-accelerated) | [NC-family](./sizes/gpu-accelerated/nc-family.md), [ND-family](./sizes/gpu-accelerated/nv-family.md), [NV-family](./sizes/gpu-accelerated/nv-family.md) | [NDasrA100_v4-series](nda100-v4-series.md), [NDm_A100_v4-series](ndm-a100-v4-series.md) | [NC-series](nc-series.md), [NV-series](nv-series.md), [NP-series](np-series.md)
| [High Performance Compute](./sizes/overview.md#high-performance-compute) |[HBv2-series](./hbv2-series-overview.md), [HBv3-series](./hbv3-series-overview.md), [HBv4-series](./hbv4-series-overview.md), [HC-series](./hc-series-overview.md), [HX-series](./hx-series-overview.md) | All sizes supported. | 

> [!NOTE]
>
> - Installation of the *CUDA & GRID drivers on Secure Boot-enabled Windows VMs* doesn't require any extra steps.
> - Installation of the *CUDA driver on Secure Boot-enabled Ubuntu VMs* requires extra steps. For more information, see [Install NVIDIA GPU drivers on N-series VMs running Linux](./linux/n-series-driver-setup.md#install-cuda-drivers-on-n-series-vms). Secure Boot should be disabled for installing CUDA drivers on other Linux VMs.
> - Installation of the *GRID driver* requires Secure Boot to be disabled for Linux VMs.
> - *Not supported* size families don't support [Generation 2](generation-2.md) VMs. Change the VM size to equivalent *supported size families* for enabling Trusted Launch.

## Operating systems supported

| OS | Version |
|:--- |:--- |
| Alma Linux | 8.7, 8.8, 9.0 |
| Azure Linux | 1.0, 2.0 |
| Debian |11, 12 |
| Oracle Linux |8.3, 8.4, 8.5, 8.6, 8.7, 8.8 LVM, 9.0, 9.1 LVM |
| RedHat Enterprise Linux | 8.4, 8.5, 8.6, 8.7, 8.8, 8.9, 8.10, 9.0, 9.1, 9.2, 9.3, 9.4, 9.5 |
| SUSE Enterprise Linux |15SP3, 15SP4, 15SP5 |
| Ubuntu Server |18.04 LTS, 20.04 LTS, 22.04 LTS, 23.04, 23.10 |
| Windows 10 |Pro, Enterprise, Enterprise Multi-Session &#42; |
| Windows 11 |Pro, Enterprise, Enterprise Multi-Session &#42; |
| Windows Server |2016, 2019, 2022, 2022-Azure-Edition, 2025, 2025-Azure-Edition &#42; |

&#42; Variations of this OS are supported.

## More information

**Regions**:

- All public regions
- All Azure Government regions
- All Azure China regions

**Pricing**:
Trusted Launch doesn't increase existing VM pricing costs.

## Unsupported features

Currently, the following VM features aren't supported with Trusted Launch:

- [Azure Site Recovery](/azure/site-recovery/concepts-trusted-vm) (*Generally available for Windows, Public preview for Linux*).
- [Managed Image](capture-image-resource.yml) (customers are encouraged to use [Azure Compute Gallery](trusted-launch-portal.md#trusted-launch-vm-supported-images)).
- [Linux VM Hibernation](./linux/hibernate-resume-linux.md)

## Secure Boot

At the root of Trusted Launch is Secure Boot for your VM. Secure Boot, which is implemented in platform firmware, protects against the installation of malware-based rootkits and boot kits. Secure Boot works to ensure that only signed operating systems and drivers can boot. It establishes a "root of trust" for the software stack on your VM.

With Secure Boot enabled, all OS boot components (boot loader, kernel, kernel drivers) require trusted publishers signing. Both Windows and select Linux distributions support Secure Boot. If Secure Boot fails to authenticate that the image is signed with a trusted publisher, the VM fails to boot. For more information, see [Secure Boot](/windows-hardware/design/device-experiences/oem-secure-boot).

## vTPM

Trusted Launch also introduces virtual Trusted Platform Module (vTPM) for Azure VMs. This virtualized version of a hardware [Trusted Platform Module](/windows/security/information-protection/tpm/trusted-platform-module-overview) is compliant with the TPM2.0 spec. It serves as a dedicated secure vault for keys and measurements.

Trusted Launch provides your VM with its own dedicated TPM instance that runs in a secure environment outside the reach of any VM. The vTPM enables [attestation](/windows/security/information-protection/tpm/tpm-fundamentals#measured-boot-with-support-for-attestation) by measuring the entire boot chain of your VM (UEFI, OS, system, and drivers).

Trusted Launch uses the vTPM to perform remote attestation through the cloud. Attestations enable platform health checks and are used for making trust-based decisions. As a health check, Trusted Launch can cryptographically certify that your VM booted correctly.

If the process fails, possibly because your VM is running an unauthorized component, Microsoft Defender for Cloud issues integrity alerts. The alerts include details on which components failed to pass integrity checks.

## Virtualization-based security

[Virtualization-based security](/windows-hardware/design/device-experiences/oem-vbs) (VBS) uses the hypervisor to create a secure and isolated region of memory. Windows uses these regions to run various security solutions with increased protection against vulnerabilities and malicious exploits. Trusted Launch lets you enable hypervisor code integrity (HVCI) and Windows Defender Credential Guard.

HVCI is a powerful system mitigation that protects Windows kernel-mode processes against injection and execution of malicious or unverified code. It checks kernel mode drivers and binaries before they run, preventing unsigned files from loading into memory. Checks ensure that executable code can't be modified after it's allowed by HVCI to load. For more information about VBS and HVCI, see [Virtualization-based security and hypervisor-enforced code integrity](https://techcommunity.microsoft.com/t5/windows-insider-program/virtualization-based-security-vbs-and-hypervisor-enforced-code/m-p/240571).

With Trusted Launch and VBS, you can enable Windows Defender Credential Guard. Credential Guard isolates and protects secrets so that only privileged system software can access them. It helps prevent unauthorized access to secrets and credential theft attacks, like Pass-the-Hash attacks. For more information, see [Credential Guard](/windows/security/identity-protection/credential-guard/credential-guard).

## Microsoft Defender for Cloud integration

Trusted Launch is integrated with Defender for Cloud to ensure that your VMs are properly configured. Defender for Cloud continually assesses compatible VMs and issues relevant recommendations:

- **Recommendation to enable Secure Boot**: The Secure Boot recommendation only applies for VMs that support Trusted Launch. Defender for Cloud identifies VMs that have Secure boot disabled. It issues a low-severity recommendation to enable it.
- **Recommendation to enable vTPM**: If vTPM is enabled for VM, Defender for Cloud can use it to perform guest attestation and identify advanced threat patterns. If Defender for Cloud identifies VMs that support Trusted Launch with vTPM disabled, it issues a low-severity recommendation to enable it.
- **Recommendation to install guest attestation extension**: If your VM has Secure Boot and vTPM enabled but it doesn't have the Guest Attestation extension installed, Defender for Cloud issues low-severity recommendations to install the Guest Attestation extension on it. This extension allows Defender for Cloud to proactively attest and monitor the boot integrity of your VMs. Boot integrity is attested via remote attestation.
- **Attestation health assessment or boot integrity monitoring**: If your VM has Secure Boot and vTPM enabled and the Attestation extension installed, Defender for Cloud can remotely validate that your VM booted in a healthy way. This practice is known as boot integrity monitoring. Defender for Cloud issues an assessment that indicates the status of remote attestation.

   If your VMs are properly set up with Trusted Launch, Defender for Cloud can detect and alert you of VM health problems.

- **Alert for VM attestation failure**: Defender for Cloud periodically performs attestation on your VMs. The attestation also happens after your VM boots. If the attestation fails, it triggers a medium-severity alert.
    VM attestation can fail for the following reasons:
  - The attested information, which includes a boot log, deviates from a trusted baseline. Any deviation can indicate that untrusted modules are loaded, and the OS could be compromised.
  - The attestation quote couldn't be verified to originate from the vTPM of the attested VM. An unverified origin can indicate that malware is present and could be intercepting traffic to the vTPM.

    > [!NOTE]
    > Alerts are available for VMs with vTPM enabled and the Attestation extension installed. Secure Boot must be enabled for attestation to pass. Attestation fails if Secure Boot is disabled. If you must disable Secure Boot, you can suppress this alert to avoid false positives.

- **Alert for untrusted Linux kernel module**: For Trusted Launch with Secure Boot enabled, it's possible for a VM to boot even if a kernel driver fails validation and is prohibited from loading. If kernel driver validation failure happens, Defender for Cloud issues low-severity alerts. While there's no immediate threat, because the untrusted driver didn't load, these events should be investigated. Ask yourself:

  - Which kernel driver failed? Am I familiar with the failed kernel driver and do I expect it to load?
  - Is the exact version of the driver same as expected? Are the driver binaries intact? If failed driver is a partner driver, did the partner pass the OS compliance tests to get it signed?

## Related content

Deploy a [Trusted Launch VM](trusted-launch-portal.md).
