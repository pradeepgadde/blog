---
layout: single
title:  "Azure Architectural Components"
date:   2022-05-13 10:59:04 +0530
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
# Azure Architectural Components
In this post let us look at some of the architectural components of Azure Cloud.
In an earlier post on [Azure Accounts]() we looked into subscriptions and resource groups.

- **Resources**: Resources are instances of services that you create, like virtual machines, storage, or SQL databases.
- **Resource groups**: Resources are combined into resource groups, which act as a logical container into which Azure resources like web apps, databases, and storage accounts are deployed and managed.
- **Subscriptions**: A subscription groups together user accounts and the resources that have been created by those user accounts. For each subscription, there are limits or quotas on the amount of resources that you can create and use. Organizations can use subscriptions to manage costs and the resources that are created by users, teams, or projects.
- **Management groups**: These groups help you manage access, policy, and compliance for multiple subscriptions. All subscriptions in a management group automatically inherit the conditions applied to the management group.

## Azure Regions 

Resources are created in regions, which are different geographical locations around the globe that contain Azure datacenters. When you deploy a resource in Azure, you'll often need to choose the region where you want your resource deployed. Regions are what you use to identify the location for your resources. 

We can look into the Azure Global Infrastructure map here [https://infrastructuremap.microsoft.com/]https://infrastructuremap.microsoft.com/

Here is a screenshot of the same.

![Azure Infra]({{ site.url }}{{ site.baseurl }}/assets/images/azure-infra.png)

There are some special Azure Regions like

- **US DoD Central**
- **China East**

for compliance or legal purposes.

## Azure Availability Zones

Availability zones are physically separate datacenters within an Azure region. Each availability zone is made up of one or more datacenters equipped with independent power, cooling, and networking. An availability zone is set up to be an *isolation boundary*. If one zone goes down, the other continues working. Availability zones are connected through high-speed, private fiber-optic networks.

This is how Microsoft described the Availability Zones

![Azure Infra]({{ site.url }}{{ site.baseurl }}/assets/images/availability-zones.png)

Availability zones are primarily for VMs, managed disks, load balancers, and SQL databases. 

## Region Pairs

Availability zones are created by using one or more datacenters. There's a minimum of three zones within a single region. It's possible that a large disaster could cause an outage big enough to affect even two datacenters. That's why Azure also creates *region pairs*.

Each Azure region is always paired with another region within the same geography (such as US, Europe, or Asia) at least 300 miles away.  If a region in a pair was affected by a natural disaster, for instance, services would automatically failover to the other region in its region pair.

Examples of region pairs in Azure are West US paired with East US and SouthEast Asia paired with East Asia.

![Azure Region Pairs]({{ site.url }}{{ site.baseurl }}/assets/images/azure-region-pairs.png)

## Azure Resources and Azure Resource Manager

- **Resource**: A manageable item that's available through Azure. Virtual machines (VMs), storage accounts, web apps, databases, and virtual networks are examples of resources.
- **Resource group**: A container that holds related resources for an Azure solution. The resource group includes resources that you want to manage as a group. You decide which resources belong in a resource group based on what makes the most sense for your organization.

> All resources must be in a resource group, and a resource can only be a member of a single resource group. 
>
> Resource groups can't be nested. 
>
> Before any resource can be provisioned, you need a resource group for it to be placed in.

By placing resources of similar usage, type, or location in a resource group, you can provide order and organization to resources you create in Azure. 

If you delete a resource group, all resources contained within it are also deleted. 

Resource groups are also a scope for applying role-based access control (RBAC) permissions. 

**Azure Resource Manager** is the deployment and management service for Azure. It provides a management layer that enables you to create, update, and delete resources in your Azure account. You use management features like access control, locks, and tags to secure and organize your resources after deployment.

When a user sends a request from any of the Azure tools, APIs, or SDKs, Resource Manager receives the request. It authenticates and authorizes the request. Resource Manager sends the request to the Azure service, which takes the requested action. Because all requests are handled through the same API, you see consistent results and capabilities in all the different tools.

![Azure Resource Manager]({{ site.url }}{{ site.baseurl }}/assets/images/azure-resource-manager.png)

## Azure Subscriptions
Using Azure requires an Azure subscription. A subscription provides you with authenticated and authorized access to Azure products and services. It also allows you to provision resources. 

An Azure subscription is a logical unit of Azure services that links to an Azure account, which is an identity in Azure Active Directory (Azure AD) or in a directory that Azure AD trusts.

An account can have one subscription or multiple subscriptions. You might want to create additional subscriptions for resource or billing management purposes. If you have multiple subscriptions, you can organize them into invoice sections.

## Azure Management Groups
If your organization has many subscriptions, you might need a way to efficiently manage access, policies, and compliance for those subscriptions. Azure management groups provide a level of scope above subscriptions. You organize subscriptions into containers called management groups and apply your governance conditions to the management groups. All subscriptions within a management group automatically inherit the conditions applied to the management group. 

You can build a flexible structure of management groups and subscriptions to organize your resources into a hierarchy for unified policy and access management.  For example, you can apply policies to a management group that limits the regions available for VM creation. This policy would be applied to all management groups, subscriptions, and resources under that management group by only allowing VMs to be created in that region.
