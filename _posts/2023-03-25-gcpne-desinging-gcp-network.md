---

layout: single
title:  "Designing, planning, and prototyping a Google Cloud network"
date:   2023-03-25 05:59:04 +0530
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gpcne.png
  og_image: /assets/images/gpcne.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Designing, planning, and prototyping a Google Cloud network

## Designing an overall network architecture

- What subnet range should you use?

There are networking fundamentals that are common to all environments, cloud or otherwise. A network engineer should have familiarity with such networking fundamentals. For example, a network engineer should be able to select subnet ranges that satisfy requirements such as available IP addresses for assignment and prevent overlap.

- Which role should be assigned?

There are many predefined Compute Engine roles available in Google Cloud that provide differing levels of access to networking resources. Most roles can be bound at the Organization, Folder, or Project level, and some roles can also be bound at the Resource level. The principles of least privilege and separation of duties should be followed when designing access control policies.

- Which features of Google Cloud networking can you utilize? 

Google Cloud provides many networking features. 

Cloud CDN is a content delivery network that caches static resources at Google edge locations around the world. It serves these resources to a large user base with maximal availability and minimal latency. 

Cloud NAT provides network address translation that enables private/internal IP-only resources to make secure requests to the internet. 

Cloud Armor is a web-application firewall (WAF) providing traffic scanning and filtering.It provides configurable protection against attack traffic including distributed denial of service (DDoS). 

Google Cloud Load Balancing provides managed software load balancing in many varieties including full global anycast load balancing to enable delivery traffic to the nearest regional backends using a single IP address. 

Google Cloud Network Intelligence Center provides several features such as Network Topology, Connectivity Testing, Performance Dashboard, and Firewall Insights. These features aid in network troubleshooting and performance debugging.

- What is the minimal network topology in Google Cloud?

VPC networks are global resources in Google Cloud and contain regional subnetworks.
Auto VPC networks will have at least one subnet in each Google Cloud region. 
Custom VPC networks can have subnetworks in only desired regions. Each subnetwork will have at least one primary subnet range and none of the primary or secondary subnet ranges can overlap in a single VPC. Each VPC network has a set of default routes routing the subnet IP ranges of each subnet to that subnet and a route for all other IP addresses to the internet. Other custom routes can also be added. Firewall rules are the primary mechanism for traffic control and can be used to allow or deny traffic matching the configured rule parameters. Firewall rules are evaluated in priority order. The two implicit firewall rules in every VPC - implicit deny all ingress and allow all egress rules- have the lowest priority.



## Designing a Virtual Private Cloud (VPC)


There are 2 main approaches to connecting resources using private/internal IP communication across projects or VPC networks in Google Cloud. 
- Shared VPC provides a centralized networking model. 
- VPC peering provides a decentralized model. 

Shared VPC allows resources from multiple projects to be placed in a common VPC network owned by a host project. 
VPC peering allows VPC networks to be connected within or across projects, or even across organizations.

There are 3 main approaches to connecting resources using private/internal IP
communication between on-premises and cloud environments in Google Cloud.
- Dedicated Interconnect provides the highest bandwidth and lowest latencies, but
requires connectivity to and installing a router in a Google Cloud colocation facility.
- Partner Interconnect provides a variety of sub-10gbps bandwidth options for
customers who do not need the full 10 or 100gbps that Dedicated Interconnect
provides, and allows meeting Google Cloud Partners in many more locations around
the world. 
- Cloud VPN typically provides for lower bandwidths and higher latency, but
is relatively inexpensive and quick to set up.



Google Cloud offers 3 zones across approximately 30 regions (4 zones in us-central1 in Iowa). Though many capabilities are available across all regions, there are some differences in supported features and capabilities by region. Resources may be zonal, regional, multi-regional, or global with implications for availability, latency, and data residency. Placing replica resources across multiple zones increases availability (protecting against resource and zone failure). Placing replica resources across multiple regions can further increase availability, protect against regional outage, and reduce average latency for users close to those regions.

## Designing a hybrid or multi-cloud network

There are many configuration options available for Cloud VPN in Google Cloud. There is Classic VPN which supports both static and dynamic routing when used with the Cloud Router. Classic VPN offers lower availability (99.9%) but can connect to on-premises VPN gateways that don’t support BGP. HA VPN only supports dynamic routing and requires a Cloud Router. Cloud Routers and HA VPN can’t work with on-premises VPN gateways that don’t support BGP. However, HA VPN can offer higher availability (99.99%) and is the Google recommended approach where possible.



Partner Interconnect. Dedicated and Partner Interconnect can provide higher bandwidths and lower latencies. Dedicated Interconnect requires installing a router in a Google co-location facility and requires purchasing at least a 10 Gbps connection.

Partner Interconnect can provide lower bandwidths and will be available at more locations from various providers. It can provide connections as low as 50 Mbps, which means it would typically be more cost effective at bandwidths lower than 10 Gbps.

With respect to network routing and IP planning, there are two main approaches to deploying GKE: routes-based or VPC-native. VPC-native is the newer and recommended approach that provides several benefits. It is important to be aware of the subtle differences in how to select the correct IP ranges to use with each type, as well as the supported numbers of resources based on size of the ranges.

## Designing an IP addressing plan for Google Kubernetes Engine

Creating GKE private clusters improves security. There are 3 high level configurations available for private clusters with varying levels of security and ease of access for each:

- Public endpoint access disabled is the most secure. Connectivity to the control plane is not allowed from any client outside the cluster subnet.

- Public endpoint access enabled, authorized networks enabled is the next most secure and provides connectivity to the control plane from only specified public or private IP ranges.

- Public endpoint access enabled, authorized networks disabled provides the least secure and provides the most permissive connectivity allowing access to the public endpoint from any IP address.