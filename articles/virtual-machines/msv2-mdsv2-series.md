---
title: Msv2/Mdsv2 Medium Memory Series - Azure Virtual Machines
description: Specifications for the Msv2-series VMs.
author: ju-shim
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 12/20/2022
ms.author: jushiman
---

# Msv2 and Mdsv2-series Medium Memory

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

The Msv2 and Mdsv2 Medium Memory VM Series features Intel® Xeon® Platinum 8280 (Cascade Lake) processor with an all core base frequency of 2.7 GHz and 4.0 GHz single core turbo frequency. With these VMs, customers achieve increased flexibility with local disk and diskless options. Customers also have access to a set of new isolated VM sizes with more CPU and memory that go up to 192 vCPU with 4 TiB of memory. 

> [!NOTE]
> Msv2 and Mdsv2 Medium Memory VMs are generation 2 only. For more information on generation 2 virtual machines, see [Support for generation 2 VMs on Azure](./generation-2.md). Standard_M192is_v2, Standard_M192ims_v2, Standard_M192ids_v2, and Standard_M192idms_v2 will be retired on March 31, 2027.



[Premium Storage](premium-storage-performance.md): Supported<br>
[Premium Storage caching](premium-storage-performance.md): Supported<br>
[Live Migration](maintenance-and-updates.md): Restricted Support<br>
[Memory Preserving Updates](maintenance-and-updates.md): Not Supported<br>
[VM Generation Support](generation-2.md): Generation 2<br>
[Write Accelerator](./how-to-enable-write-accelerator.md): Supported<br>
[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported<br>
[Ephemeral OS Disks](ephemeral-os-disks.md): Supported for Mdsv2 <br>
[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Not Supported <br>
<br>
 
## Msv2 Medium Memory Diskless

### [Basics](#tab/msv2basic)

vCPUs (Qty.) and Memory for each size

| Size | vCPU | Memory: GiB | 
|---|---|---|
| Standard_M32ms_v2   | 32  | 875  | 
| Standard_M64s_v2    | 64  | 1024 | 
| Standard_M64ms_v2   | 64  | 1792 |
| Standard_M128s_v2   | 128 | 2048 |
| Standard_M128ms_v2  | 128 | 3892 | 
| Standard_M192is_v2  | 192 | 2048 |
| Standard_M192ims_v2 | 192 | 4096 |

### [Local Storage](#tab/msv2local)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote Storage](#tab/msv2storageremote)

Remote (uncached) storage info for each size

| Size | Max Remote Storage Disks (Qty) | Max uncached disk trhoughput: IOPs/MBps | Burst uncached disk throughput: IOPS/MBps<sup>1</sup> |
| ---- | ------------------------------ | --------------------------------------- | ----------------------------------------------------- |
| Standard_M32ms_v2   | 32 | 20000/500  | 40000/1000 | 
| Standard_M64s_v2    | 64 | 40000/1000 | 80000/2000 | 
| Standard_M64ms_v2   | 64 | 40000/1000 | 80000/2000 |
| Standard_M128s_v2   | 64 | 80000/2000 | 80000/4000 |
| Standard_M128ms_v2  | 64 | 80000/2000 | 80000/4000 |
| Standard_M192is_v2<sup>2</sup> | 64 | 80000/2000 | 80000/4000 |
| Standard_M192ims_v2 | 64 | 80000/2000 | 80000/4000 |

<sup>1</sup> Msv2 and Mdsv2 medium memory VMs can [burst](./disk-bursting.md) their disk performance for up to 30 minutes at a time.

<sup>2</sup> Attaching Ultra Disk or Premium SSDs V2 to **Standard_M192is_v2** results in higher IOPs and MBps than standard premium disks:
- Max uncached Ultra Disk and Premium SSD V2 throughput (IOPS/ MBps): 120000/2000 
- Max burst uncached Ultra Disk and Premium SSD V2 disk throughput (IOPS/ MBps): 120000/4000

### [Network](#tab/msv2network)

Network interface info for each size

| Size | Max NICs | Expected network egress bandwidth (Mbps) | 
|---|---|---|
| Standard_M32ms_v2   | 8 | 8000  | 
| Standard_M64s_v2    | 8 | 16000 | 
| Standard_M64ms_v2   | 8 | 16000 | 
| Standard_M128s_v2   | 8 | 30000 | 
| Standard_M128ms_v2  | 8 | 30000 | 
| Standard_M192is_v2  | 8 | 30000 | 
| Standard_M192ims_v2 | 8 | 30000 |

---
## Mdsv2 Medium Memory with Disk  

### [Basics](#tab/mdsv2basics)

vCPUs (Qty.) and Memory for each size

| Size | vCPU | Memory: GiB |
|---|---|---|
| Standard_M32dms_v2   | 32  | 875  |
| Standard_M64ds_v2    | 64  | 1024 |
| Standard_M64dms_v2   | 64  | 1792 |
| Standard_M128ds_v2   | 128 | 2048 |
| Standard_M128dms_v2  | 128 | 3892 |
| Standard_M192ids_v2  | 192 | 2048 |
| Standard_M192idms_v2 | 192 | 4096 |

### [Local Storage](#tab/mdsv2storagelocal)

Local (temp and cached) storage info for each size

| Size | Temp storage (SSD) GiB | Max cached and temp storage throughput: IOPS / MBps | Burst cached and temp storage throughput: IOPS/MBps<sup>1</sup> |
|---|---|---|---|
| Standard_M32dms_v2   | 1024 | 40000/400 | 40000/1000 |
| Standard_M64ds_v2    | 2048 | 80000/800   | 80000/2000 |
| Standard_M64dms_v2   | 2048 | 80000/800   | 80000/2000 |
| Standard_M128ds_v2   | 4096 | 160000/1600 | 250000/4000 |
| Standard_M128dms_v2  | 4096 | 160000/1600 | 250000/4000 |
| Standard_M192ids_v2  | 4096 | 160000/1600 | 250000/4000 |
| Standard_M192idms_v2 | 4096 | 160000/1600 | 250000/4000 |

<sup>1</sup> Msv2 and Mdsv2 medium memory VMs can [burst](./disk-bursting.md) their disk performance for up to 30 minutes at a time.

### [Remote Storage](#tab/mdsv2storageremote)

Remote (uncached) storage info for each size

| Size | Max data disk | Max uncached disk throughput: IOPS/MBps | Burst uncached disk throughput: IOPS/MBps<sup>1</sup> |
|---|---|---|---|---|---|
| Standard_M32dms_v2   | 32 | 20000/500  | 40000/1000 |
| Standard_M64ds_v2    | 64 | 40000/1000 | 80000/2000 |
| Standard_M64dms_v2   | 64 | 40000/1000 | 80000/2000 |
| Standard_M128ds_v2   | 64 | 80000/2000 | 80000/4000 |
| Standard_M128dms_v2  | 64 | 80000/2000 | 80000/4000 |
| Standard_M192ids_v2  | 64 | 80000/2000 | 80000/4000 |
| Standard_M192idms_v2 | 64 | 80000/2000 | 80000/4000 |

<sup>1</sup> Msv2 and Mdsv2 medium memory VMs can [burst](./disk-bursting.md) their disk performance for up to 30 minutes at a time. 

### [Network](#tab/mdsv2network)

Network interface info for each size

| Size | Max NICs | Expected network egress bandwidth (Mbps) | 
|---|---|---|
| Standard_M32dms_v2   | 8 | 8000  | 
| Standard_M64ds_v2    | 8 | 16000 | 
| Standard_M64dms_v2   | 8 | 16000 | 
| Standard_M128ds_v2   | 8 | 30000 | 
| Standard_M128dms_v2  | 8 | 30000 | 
| Standard_M192ids_v2  | 8 | 30000 | 
| Standard_M192idms_v2 | 8 | 30000 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](./includes/virtual-machines-common-sizes-table-defs.md)]

## Other sizes and information

- [General purpose](sizes-general.md)
- [Memory optimized](sizes-memory.md)
- [Storage optimized](sizes-storage.md)
- [GPU optimized](sizes-gpu.md)
- [High performance compute](sizes-hpc.md)
- [Previous generations](sizes-previous-gen.md)

Pricing Calculator: [Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

## Next steps

Learn more about how [Azure compute units (ACU)](acu.md) can help you compare compute performance across Azure SKUs.
