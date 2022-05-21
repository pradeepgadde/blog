---

layout: single
title:  "Azure ExpressRoute"
date:   2022-05-21 05:59:04 +0530
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
With ExpressRoute we can extend our on-premises networks into the Microsoft cloud over a private connection with the help of a connectivity provider.

BGP can be used as the dynamic routing protocol between your network and Microsoft.

ExpressRoute enables direct access to the several Azure services like Microsoft Office 365, Azure compute services, Azure cloud services in all regions.

You can enable ExpressRoute Global Reach to exchange data across your on-premises sites by connecting your ExpressRoute circuits. Your cross-datacenter traffic will travel through the Microsoft network.

There are four ExpressRoute Connectivity models, which are further categorized as 
- Service Provider Model
- Direct Model

Cloud Exchange Colocation, Point-to-Point Ethernet connection, Any-to-Any Connection fall under the Service Provider model.

In Any-to-any model, Azure integrates with your WAN connection to provide a connection like you would have between your datacenter and any branch offices.

If you already use Multiprotocol Label Switching (MPLS) to connect to your branch offices or other sites in your organization, an ExpressRoute connection to Microsoft behaves like any other location on your private WAN.

The other option is Direct model, where You can connect directly into the Microsoft's global network at a peering location strategically distributed across the world. ExpressRoute Direct provides dual 100 Gbps or 10-Gbps connectivity, which supports Active/Active connectivity at scale.

One thing to note from security point of view is that, even if you have an ExpressRoute connection, DNS queries, certificate revocation list checking, and Azure Content Delivery Network requests are still sent over the public internet.

This reference architecture shows how to connect an on-premises network to virtual networks on Azure, using Azure ExpressRoute.

![Azure ExpressRoute]({{ site.url }}{{ site.baseurl }}/assets/images/azure-expressroute.png)

