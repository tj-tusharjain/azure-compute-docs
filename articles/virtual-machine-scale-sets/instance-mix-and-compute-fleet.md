---
title: Instance Mix and Compute Fleet
description: The functional similarities and differences between Azure Compute Fleet and Virtual Machine Scale Sets, as well as use case scenarios.
author: brittanyrowe 
ms.author: brittanyrowe
ms.topic: conceptual
ms.service: azure-virtual-machine-scale-sets
ms.date: 2/13/2025
ms.reviewer: jushiman
---

# Understanding Scale Sets and Azure Compute Fleet

## What are Azure Compute Fleet and Instance Mix?
Azure Compute Fleet and instance mix on virtual machine scale sets are two means of deploying virtual machines (VMs) on Azure that cater to different needs and use cases. While both are designed to optimize the deployment and management of VMs, they offer distinct functionalities and benefits.

## Functional similarities
Both Instance Mix and Azure Compute Fleet are designed to optimize the deployment and management of virtual machines (VMs) within Azure, sharing several functional similarities:
* **Allocation Strategies**: Both Instance Mix and Compute Fleet utilize allocation strategies to optimize your compute deployments. Instance Mix adopts a subset of these strategies, while Compute Fleet offers a more extensive range. These strategies help in optimizing cost and performance based on user requirements. You can learn more about the allocation strategies in the instance mix and Compute Fleet documentation.
* **Built on scale sets**: Both features are built on Virtual Machine Scale Sets, leveraging its capabilities for scaling and managing large numbers of VMs. This foundation ensures that both instance mix and Compute Fleet can provide robust and scalable solutions for managing virtual machine deployments.
* **Utilize Spot and Standard VMs**: Both instance mix and Compute Fleet support the use of Spot and Standard VMs. This flexibility allows users to balance cost and availability, choosing the type of VMs that best suit their needs and budget constraints.
* **Near-Real-Time capacity signals**: A new component for decision-making that utilizes near-real-time capacity signals is used by both instance mix and Compute Fleet. This component improves the efficiency of resource allocation, ensuring that VMs are deployed in an optimal manner based on current capacity and demand signals.
* **Free to use**: Both Compute Fleet and Virtual Machine Scale sets are free services which deploy and manage VMs for you. VMs are billed for their associated compute, storage, and networking costs. 

These shared functionalities enable both Instance Mix and Azure Compute Fleet to provide efficient, cost-effective, and scalable solutions for managing virtual machine deployments in Azure.


## Functional differences
### Azure Compute Fleet
* **Ultra large scale**: Azure Compute Fleet allows for large-scale deployments, supporting up to 10,000 VMs (or 1 million cores. This feature is ideal for scenarios requiring extensive compute resources.
* **Multi-region deployments**: Compute Fleet supports multi-region deployments, providing high availability and disaster recovery capabilities.
* **Attribute-based VM selection**: Compute Fleet can utilize attribute-based VM selection to match available SKUs to customer requirements.

### Instance Mix
* **Easy to adopt**: Instance mix is available for Virtual Machine Scale Sets with Flexible Orchestration Mode. Existing scale sets using Flexible Orchestration Mode can make a simple update to start using instance mix.
* **Flexibility in VM size**: You can specify up to five different VM sizes with instance mix. By being flexible with VM sizes, instance mix helps navigate capacity constraints and ensures consistent application performance.
* **Utilize scale set orchestrations**: Instance mix integrates with autoscale, upgrade policies, health monitoring, and more to allow the scale set to manage the lifecycle management of your VMs.


## Use Cases
### Azure Compute Fleet
* **Large-scale deployments**: Ideal for organizations requiring large-scale compute resources across multiple regions and VMSS.
* **Cost optimization**: Suitable for scenarios where cost optimization is crucial, leveraging Spot VMs and various pricing models.
* **High availability**: Best for applications needing high availability and disaster recovery through multi-region deployments.

### Instance Mix
* **Capacity constraints**: Perfect for users facing capacity constraints, as it allows tapping into a larger pool of resources by specifying multiple VM sizes.
Simplified management: Beneficial for users who want to simplify the management of diverse VM sizes within a single VMSS.
* **VM lifecycle orchestrations**: Ideal for scenarios where customers don't want to have to manage various elements of the VM lifecycle, such as updates and upgrades.
* **Dynamic scaling**: Integrating with autoscale, instance mix enabled scale sets can dynamically scale based on scale set usage or a set schedule.


## Quick comparison table

|   | Virtual Machine | Virtual Machine Scale Set | Compute Fleet (Preview) |
|---------|----------------|---------------------------|-------------------------|
| **Description** | Create and manually configure individual VMs. Best for small/unique workloads or apps, with no automation features to manage a group of VMs. | Load-balanced VM groups with workload optimization across multiple availability zones, plus automatic scaling. | Manage and automate provisioning of thousands of mixed-size VMs to maximize performance and high availability. |
| **Product Differences** | - Deploy and manage each VM individually. Fine-tune custom applications—no group management, optimization, or scaling.  <br> - Each VM can be assigned to one availability zone at a time.  <br> - Schedule automatic backups or perform them manually. | - Deploy and manage up to 2,000 mixed-size VMs as a group.  <br> - Scale automatically.  <br> - Optimization across multiple zones/fault domains for high availability.  <br> - Integrate Azure Spot VMs to cut costs.  <br> - Optimized for stateless or stateful workloads. | - Deploy and manage up to 10,000 mixed-size VMs as a group, and add/remove VM sizes anytime.  <br> - Optimization across multiple zones/fault domains for high availability.  <br> - Hyper-scale with demand and maintain capacity with Spot VMs to cut costs.  <br> - Configure fleet to allocate strategically, optimizing price, capacity, or both. |
| **Best For** | Small, simple jobs or specialized apps where creating/managing individual VMs is feasible. Use to test configurations, experiment with Azure, or when you don’t need to optimize resiliency or capacity. | More than one VM, or if you plan to scale. Platform engineering or infrastructure as code implementations; moving on-premises apps to the cloud; specialized workloads for high-performance computing; open-source databases; or batch processing. Built-in optimizations maintain performance while controlling costs—only pay for capacity used. Mix VM sizes for more flexibility. | Large-scale compute-intensive workloads or batch jobs. Choose this option if you want flexibility with VM sizes or want to combine discounted pricing models in addition to optimizing costs on a large scale with Azure Spot. |

## Detailed comparison table
|  | Virtual Machine | Virtual Machine Scale Set | Compute Fleet (Preview) |
|---------|----------------|---------------------------|-------------------------|
| **Cost** | Billed as an individual VM instance. Cost-effective for consistent load and traffic. | **Optimize costs with VMSS:** <br> - VMSS automatically uses less capacity when load/traffic is low and optimizes cost while accommodating dips and spikes of cyclical, intermittent, and growing workloads. <br> - Use 100% Spot VMs, or mix with standard ones, to cut cost. | **Optimize costs with Compute Fleet:** <br> - Combine discounts (like reserved instances and savings plans) with Spot VM savings; just rank the top VM sizes you want Fleet to prioritize. <br> - Your fleet will maintain capacity with discounted Spot VMs to optimize cost. |
| **Scaling** | Change VM size manually (vertical scaling). | Horizontal - Manually add/remove instances, and autoscale based on: <br> - Demand <br> - Schedule <br> - AI-predicted usage patterns | Horizontal - Manually managed through modifying the target capacity. |
| **Instances** | Single instances deployed individually. | 2 to 2,000 instances. | Up to 10,000 instances. |
| **Management** | Deploy and manage each VM individually. | - Deploy and manage VMs as a group, or opt to manage them individually. <br> - Orchestrate batch operations (start, stop, restart) and updates (configuration, app, security, and patching) for safer rollouts. <br> - Attach and manage one universal add-on for services like data disk and public IP at the VMSS level. | - Deploy and manage up to 10,000 VMs as an automated fleet-managed group. <br> - Enhance the flexibility, efficiency, and control of your VMs, attaching one universal add-on for services like data disks and Public IPs. |
| **Availability + Zonal Migration** | Manually deploy each standalone VM into a single availability zone. To boost uptime, place one or more VMs into each eligible availability zone. | - Automatically deploy and spread VMs into one, or across multiple, availability zones (zonal or zone-spanning) to boost resilience and uptime. <br> - Remove VM instances from the VMSS anytime. | Fleet uses availability zones to automatically deploy and distribute VMs across multiple zones. Choose specific zones, all eligible zones, or all eligible zones within a given region. |