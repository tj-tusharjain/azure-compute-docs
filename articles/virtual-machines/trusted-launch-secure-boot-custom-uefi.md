---
title: Secure Boot UEFI keys 
description: Customers can replace, append secure boot UEFI keys (pk/kek/db/dbx).
author: prasadmsft
ms.author: reprasa
ms.service: azure-virtual-machines
ms.subservice: trusted-launch
ms.topic: conceptual
ms.date: 02/24/2025
ms.custom: template-concept, devx-track-azurecli
---

# Secure Boot UEFI keys

For certain advanced scenarios, it's necessary for customizing secure boot keys. The secure boot UEFI keys customization allows you to modify Unified extensible firmware interface (UEFI) keys – PK, KEK, DB, DBX – in your image for secure boot capable Azure virtual machine (VM) (Trusted Launch and Confidential VM). With this feature, UEFI keys can be fully replaced or appended to default key databases.

When a secure boot Azure VM is deployed, during the boot process, signatures of all the boot components such as UEFI, shim/bootloader, kernel, and kernel modules/drivers are verified. Verification fails if boot component signature doesn't match with a key in the trusted key databases and the VM fails to boot. This failure can occur if a component is: signed by a key not found in the trusted key databases, or the signing key is listed in the revoked key databases, or is unsigned. Through custom secure boot UEFI keys, necessary can be supplied for successful VM boot.

UEFI key customization is offered through [Azure compute gallery](https://learn.microsoft.com/azure/virtual-machines/azure-compute-gallery) resources. If you currently use a Marketplace image to create a Trusted Launch virtual machine, you must first create Azure compute gallery resources using the Marketplace image to modify UEFI keys.

> [!NOTE]
> Customizing UEFI keys is for advanced customers intimately familiar with inner workings of secure boot technology.


## UEFI key customization using Azure compute gallery image

The Azure compute gallery resources necessary for customizing UEFI keys are: [gallery](https://learn.microsoft.com/azure/templates/microsoft.compute/galleries), [image definition](https://learn.microsoft.com/azure/templates/microsoft.compute/galleries/images), and [image version](https://learn.microsoft.com/azure/templates/microsoft.compute/galleries/images/versions).


Currently, UEFI key customization is available only through REST API.

**Pre-requisite:** 
You must attach a "VM image version" resource to a "VM image definition" resource with security-type set to "*TrustedLaunchSupported*" or "*TrustedLaunchAndConfidentialVmSupported*." Detailed instructions on how to create an image definition resource can be found [here](https://learn.microsoft.com/azure/virtual-machines/image-version). To create a VM image definition resource, you must also create a gallery resource. You can choose Azure portal, PowerShell, CLI, or REST API to create gallery and image definition resources.

**REST API**
A generic ARM template is provided as a sample for creating an image version resource. For customizing UEFI keys, you also need to include ‘security-profile’ block in the image version resource. Details on what goes inside the *securityProfile* block for each of the UEFI key modification scenarios is provided in the scenario sections.

###### Generic ARM template
```json
{
    "apiVersion": "2022-03-03",
    "type": "Microsoft.Compute/galleries/images/versions",
    "dependsOn": [
        "[concat('Microsoft.Compute/galleries/', parameters('galleryName'), '/images/', parameters('imageDefinitionName'))]"
    ],
    "name": "[concat(parameters('galleryName'), '/', parameters('imageDefinitionName'), '/', parameters('versionName'))]",
    "location": "[parameters('location')]",
    "properties": {
        "publishingProfile": {
            "replicaCount": "[parameters('defaultReplicaCount')]",
            "targetRegions": "[parameters('regionReplications')]",
            "excludeFromLatest": "[parameters('excludedFromLatest')]",
            "replicationMode": "[parameters('replicationMode')]"
        },
        "storageProfile": {
            "source": {
                "id": "[parameters('sourceVmId')]"
            }
        },
        "safetyProfile": {
            "allowDeletionOfReplicatedLocations": "[parameters('allowDeletionOfReplicatedLocations')]"
        },
        "securityProfile": {}
    },
    "tags": {}
```

## Scenarios
Instructions on how to customize UEFI keys for common scenarios are described in subsequent sections.

**Scenario 1: Replace all UEFI keys - PK, KEK, DB, and DBX**
In this scenario, the instructions here provide information on how to fully replace default keys available in Trusted Launch virtual machine.

- Create certificates and/or keys for each or PK, KEK, DB, and DBX. Detailed instructions for various operating systems through links in the ‘Useful Links’ section.
- Create a new image version. Use the following securityProfile ARM template block with the image version creation [Generic ARM template](#generic-arm-template). (Replace "Base64 formatted certificate" and "Base64 formatted sha256 hash in the JSON text with the real values.)

###### Replace all UEFI keys
```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "NoSignatureTemplate"
        ],
        "additionalSignatures": {
            "pk": {
                "type": "x509",
                "value": [
                    "Base64 formatted certificate"
                ]
            },
            "kek": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "db": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "dbx": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                },
                {
                    "type": "sha256",
                    "value": [
                        "Base64 formatted sha256 hash",
                        "Base64 formatted sha256 hash"
                    ]
                }
            ]
        }
    }
}
```

- Create a Trusted Launch VM using Portal, PowerShell, CLI, or REST API with the source image as the one you created using Azure compute gallery resources.
- Once Trusted Launch VM is in the ‘running’ state, verify that custom UEFI keys are in the UEFI key databases. Follow instructions in the section [Verifying keys in UEFI databases](#verifying-keys-in-uefi-databases).

**Scenario 2: Append keys to DB and DBX to a signature template**
In this scenario, the instructions here provide information on how to append DB and DBX keys to the default values available in Trusted Launch virtual machine.

- Create certificates and/or keys for each or DB and DBX. Detailed instructions for various operating systems through links in the [Useful Links](#useful-links) section.
- Create new image version. Use the following securityProfile ARM template block with the image version creation [Generic ARM template](#generic-arm-template). (Replace "Base64 formatted certificate" and "Base64 formatted sha256 hash in the JSON text with the real values.)

###### Append UEFI keys
```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "MicrosoftUefiCertificateAuthorityTemplate"
        ],
        "additionalSignatures": {
            "db": [
                {
                    "type": "x509",
                    "value": [
                        "Base64 formatted certificate",
                        "Base64 formatted certificate"
                    ]
                }
            ],
            "dbx": [
                {
                    "type": "sha256",
                    "value": [
                        "Base64 formatted sha256 hash",
                        "Base64 formatted sha256 hash"
                    ]
                }
            ]
        }
    }
}
```
- Create a Trusted Launch VM using Portal, PowerShell, CLI, or REST API with the source image as the one you created using Azure compute gallery resources.
- Once Trusted Launch VM is in the ‘running’ state, verify that custom UEFI keys are in the UEFI key databases. Follow instructions in the section [Verifying keys in UEFI databases](#verifying-keys-in-uefi-databases).

**Scenario 3: Apply multiple signature templates (UEFI, Windows)**
In this scenario, the instructions here provide information on how to replace default signature templates and inject multiple signature templates into a Trusted Launch virtual machine.

- Create a new image version. Use the following securityProfile ARM template block with the image version creation [Generic ARM template](#generic-arm-template) block.

###### Multiple signature templates
```json
"securityProfile": {
    "uefiSettings": {
        "signatureTemplateNames": [
            "MicrosoftUefiCertificateAuthorityTemplate",
            "MicrosoftWindowsTemplate"
        ]
    }
}
```

- Create a Trusted Launch VM using Portal, PowerShell, CLI, or REST API with the source image as the one you created using Azure compute gallery resources.
- Once Trusted Launch VM is in the ‘running’ state, verify that custom UEFI keys are in the UEFI key databases. Follow instructions in the section [Verifying keys in UEFI databases](#verifying-keys-in-uefi-databases).


## Verifying keys in UEFI databases
To verify the keys stored in UEFI databases, use the following commands.

- Enter command line mode (SSH, etc.) for the desired virtual machine
- For Linux OS:
```bash
// For key(s) added to db
$ mokutil --db

// For keys(s) added to dbx
$ mokutil --dbx
```
- Verify displayed keys are the ones supplied with image version.

# Clouds and regions supported
- Public cloud: All regions are supported except the following regions: Austria East, Belgium Central, Chile Central, Indonesia Central, Israel Northwest, Jio India Central, Jio India West, Malaysia South, Malaysia West, Mexico Central, Qatar Central, Southcentral US 2, Southeast US,Southeast US 3, Switzerland West, Taiwan North, Taiwan Northwest, UK West, and West India.
- USGov and China clouds: Coming soon.

# Useful links
- How to convert Base64 certificates: [Base64 certificate encoding](https://www.base64encode.org/enc/certificate/)
- How to convert X.509 certificate to Base64: [X.509 Certificate Public Key in Base64](https://stackoverflow.com/questions/24492981/x-509-certificate-public-key-in-base64)
- How to sign things for secure boot: [Ubuntu](https://ubuntu.com/blog/how-to-sign-things-for-secure-boot), [Redhat](https://access.redhat.com/documentation/red_hat_enterprise_linux/8/html/managing_monitoring_and_updating_the_kernel/signing-a-kernel-and-modules-for-secure-boot_managing-monitoring-and-updating-the-kernel), [Debian](https://wiki.debian.org/SecureBoot), [Oracle](https://docs.oracle.com/en/operating-systems/oracle-linux/secure-boot/sboot-SigningKernelModulesforUseWithSecureBoot.html#sb-mod-sign-kernel-uek6-req)