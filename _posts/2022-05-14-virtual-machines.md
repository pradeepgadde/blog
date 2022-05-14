---
layout: single
title:  "Azure Virtual Machines"
date:   2022-05-14 03:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
# Azure Virtual Machines
Azure Virtual Machines provide infrastructure as a service (IaaS) in the form of a virtualized server. Just like a physical computer, you can customize all of the software running on the VM. 

Azure Virtual Machines are an ideal choice when you need:
- Total control over the operating system (OS).
- The ability to run custom software.
- To use custom hosting configurations.

These are some scenarios where we might want to use Azure Virtual Machines:
- During testing and development
- When running applications in the cloud
- When extending your datacenter to the cloud
- During disaster recovery

## Lift and Shift
VMs are also an excellent choice when you move from a physical server to the cloud (also known as lift and shift). You can create an image of the physical server and host it within a VM with little or no changes. Just like a physical on-premises server, you must maintain the VM. You update the installed OS and the software it runs.

## Scaling VMs in Azure
You can run single VMs for testing, development, or minor tasks. 
Or you can group VMs together to provide high availability, scalability, and redundancy. 
We can make use of these features:
- Virtual machine scale sets: Virtual machine scale sets let you create and manage a group of identical, load-balanced VMs. The number of VM instances can automatically increase or decrease in response to demand or a defined schedule. Without virtual machine scale sets, lot of manual work would be needed.
- Azure Batch: Azure Batch enables large-scale parallel and high-performance computing (HPC) batch jobs with the ability to scale to tens, hundreds, or thousands of VMs. Batch starts a pool of compute VMs, installs applications and staging data, runs jobs with as many tasks as you have, scales down the pool as work completes.

Here is a screenshot of creating Azure Virtual Machines

![Azure VM]({{ site.url }}{{ site.baseurl }}/assets/images/azure-create-vm.png)

We can see as part of virtual machine creation, we are asked to provide some mandatory inputs like:

- Subscription
- Resource Group (either select from already available resource groups, or create new)
- Virtual machine name
- Region
- Image

There seems to be 1500+ options available in the Marketplace for selecting a compute Image.
![Azure VM]({{ site.url }}{{ site.baseurl }}/assets/images/azure-create-vm-3.png)

- Size (There seems to be 500+, to be specific 591 VM sizes available as on today)

Here is a screenshot of the most used sizes by users in Azure

![Azure VM]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vm-size.png)

Some more mandatory inputs while creating a VM are 
- Username 
- Key pair name
- Inbound port Rules

![Azure VM]({{ site.url }}{{ site.baseurl }}/assets/images/azure-create-vm-2.png)

These are just from the `Basic` tab, let us go through the other tabs like `Disks`, `Networking` in other posts.

