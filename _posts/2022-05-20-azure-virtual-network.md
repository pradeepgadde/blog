---

layout: single
title:  "Azure Virtual Network Fundamentals"
date:   2022-05-20 03:59:04 +0530
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

In this post, let us look into Azure Virtual Network Fundamentals.

You have an on-premises datacenter that you plan to keep, but you want to use Azure to offload peak traffic by using virtual machines (VMs) hosted in Azure. You want to keep your existing IP addressing scheme and network appliances while ensuring that any data transfer is secure.

Using Azure Virtual Network for your virtual networking can help you reach your goals.


Azure virtual networks enable Azure resources, such as VMs, web apps, and databases, to communicate with each other, with users on the internet, and with your on-premises client computers.

Azure virtual network allows you to create multiple isolated virtual networks. When you set up a virtual network, you define a private IP address space by using either public or private IP address ranges.

A VM in Azure can connect to the internet by default. You can enable incoming connections from the internet by assigning a public IP address to the VM or by putting the VM behind a public load balancer. For VM management, you can connect via the Azure CLI, Remote Desktop Protocol, or Secure Shell.

You can enable Azure resources to communicate securely with each other either by using 

-  Virtual Networks or 
- Service Endpoints.

You can create a network that spans both your local and cloud environments in three ways:

- Point-to-site virtual private networks
- Site-to-site virtual private networks
- Azure ExpressRoute

ExpressRoute provides a dedicated private connectivity to Azure that doesn't travel over the internet.



By default, Azure routes traffic between subnets on any connected virtual networks, on-premises networks, and the internet.

You can create custom route tables that control how packets are routed between subnets.

Border Gateway Protocol (BGP) works with Azure VPN gateways, Azure Route Server, or ExpressRoute to propagate on-premises BGP routes to Azure virtual networks.

Azure virtual networks enable you to filter traffic between subnets by using the following approaches:

- Network Security Groups
- Network Virtual Appliances

You can link virtual networks together by using virtual network *peering*. Peering enables resources in each virtual network to communicate with each other. These virtual networks can be in separate regions, which allows you to create a global interconnected network through Azure.

User-defined routes (UDR) are a significant update to Azure’s Virtual Networks that allows for greater control over network traffic flow. This method allows network administrators to control the routing tables between subnets within a VNet, as well as between VNets.



![Azure VN]({{ site.url }}{{ site.baseurl }}/assets/images/azure-vn.png)
