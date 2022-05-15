---
layout: single
title:  "Introduction to Azure"
date:   2022-05-13 06:59:04 +0530
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

# Introduction to Azure
In this post, let us take a look at how does Azure work, and Azure Portal, Azure Marketplace.

## How Does Azure Work?
First of all what is Azure? Azure is a Cloud Computing Platform which facilitates build, deploy and management of compute, network, and storage resources.

Cloud is not a nebulous magic! 
Cloud is nothing but someone else's computer resources, in this case Microsoft's. The underlying technology for any Cloud platform is Virtualization. 

Virtualization makes use of hypervisor to decouple hardware and software. On a single machine, multiple virtual machines can be run with different compatible operating systems.

Azure or any cloud platform takes the virtualization to a global level, by deploying multiple racks of physical servers with hypervisors. 

As per the Mircosoft Azure learn portal, there is a special piece of component called `Fabric Controller` running on a server in each rack.

All of these multiple Fabric controllers will be talking to another component called `Orchestraor`.

As a consumer of the cloud, we will be interacting with this Orchestrator either via the web console or programmatic calls (APIs). Depending on the request, Orchestrator will pick one of the Fabric Controllers, and the resources get instantiated on a selected physical server.

Here is a graphical representation of this as described by Microsoft.

![Azure Working]({{ site.url }}{{ site.baseurl }}/assets/images/azure-working.jpeg)

## Azure Portal
Azure portal is the unified web interface, through which we can build, deploy, and manage virtual resources in the cloud.

Azure portal is designed for continuous availability and has its presence in every Azure Data Center.

The home page of  [Azure Portal](https://portal.azure.com/#home) looks like this, however this might change regularly depending on the availability of new features, updates etc.

![Azure Portal]({{ site.url }}{{ site.baseurl }}/assets/images/azure-portal.png)

## Azure Marketplace
Azure Marketplace which can be accssed at [https://azuremarketplace.microsoft.com/en-IN/](https://azuremarketplace.microsoft.com/en-IN/) contains solutions from Microsoft Partners, other vendors, which are optimized to run on Azure.

Here is a screenshot of Azure compatible solutions offered by Juniper Networks, for example.

![Azure Marketplace]({{ site.url }}{{ site.baseurl }}/assets/images/azure-mp-jnpr.png)

The catalog spans sever categories like containers, virtual machine images, databases etc.

That's a brief intro on Azure.

