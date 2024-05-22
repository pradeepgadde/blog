---

layout: single
title:  "A Tour of Cloud Networking"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# A Tour of Cloud Networking

Google has a planet-scale, advanced, fiber-optic software-defined  network with presence in over 200 countries and territories. This  network provides services such as Search, Maps, YouTube, Google Cloud  and more to billions of users and customers.

There are six Google Cloud building blocks of cloud networking. By  grouping the network functions into six building blocks (Network  connectivity, Network security, Service Networking, Service security,  Content delivery, Observability) we can conceptualize the Google Cloud  networking services that help us achieve the requirements we are trying  to address.

- Virtual Private Cloud (VPC) network
- Network services
- Network connectivity
- Networking security
- Network intelligence (i.e. Observability)
- Network Service Tiers

### Understanding Regions and Zones

Certain Compute Engine resources live in regions or zones. A region  is a specific geographical location where you can run your resources.  Each region has one or more zones. For example, the us-central1 region  denotes a region in the Central United States that has zones `us-central1-a`, `us-central1-b`, `us-central1-c`, and `us-central1-f`.

Resources that live in a zone are referred to as zonal resources.  Virtual machine Instances and persistent disks live in a zone. To attach a persistent disk to a virtual machine instance, both resources must be in the same zone. Similarly, if you want to assign a static IP address  to an instance, the instance must be in the same region as the static  IP.

## Networking Overview

Google Cloud networking is a comprehensive suite of networking  services to enable businesses to build, scale, and manage secure and  scalable network infrastructure in Google Cloud.

**The network consists of:**

- [Region](https://cloud.google.com/compute/docs/regions-zones#identifying_a_region_or_zone) - Geographical location.
- [Zones](https://cloud.google.com/compute/docs/regions-zones#identifying_a_region_or_zone) - Interconnected deployment centers within a region. Currently a region comprises a minimum of three zones.
- [Point of presence](https://peering.google.com/#/infrastructure) (PoP) - Connects public internet to Google Cloud. Provides services like CDN, Media CDN, Interconnects.

Google Cloud provides a wide range of products and services that  address all aspects of networking, from basic connectivity to advanced  traffic management and security.



## VPC network

Google Cloud VPC network is a foundational component of Google  Cloud's networking infrastructure. It allows you to create a logically  isolated virtual network within the Google Cloud, providing a private  and secure environment for your cloud resources. You can define your own IP address space, subnetworks, and routing policies, giving you  complete control over your network connectivity.

**Key features of Google Cloud VPC network:**

- **Private IP address space**: Define your own private IP address range, ensuring no overlap with other networks.
- **Subnetwork**: Divide your VPC into multiple subnets to organize and segment your network resources.
- **Customizable routing**: Control how traffic flows within your VPC and between VPCs.
- **Firewall rules**: Define firewall rules to filter incoming and outgoing traffic, enhancing network security.

**Example use cases of Google Cloud VPC network:**

- **Hosting web applications and services**: Create a VPC to  isolate your web applications from other resources and the public  internet, enhancing security and performance.
- **Deploying microservices-based architectures**: Utilize VPCs to segment microservices and manage traffic flow between them, enabling scalability and flexibility.
- **Connecting on-premises networks**: Establish secure  connections between your on-premises network and Google Cloud resources  via Cloud VPN, or Cloud Interconnect enabling hybrid cloud deployments.
- **Creating a secure cloud environment for sensitive data**: Leverage VPCs to isolate and protect sensitive data from unauthorized access, ensuring data privacy and compliance.

Google Cloud VPC network provides a powerful and flexible foundation  for building and managing secure, scalable, and performant network  infrastructure in the cloud.



## Network services

Google Cloud network offers a suite of network services that empower  users to effectively control and optimize their network infrastructure.  Some of these include:

- [Load Balancing](https://cloud.google.com/load-balancing?hl=en): Distribute incoming traffic across multiple instances of an application or service, ensuring high availability and scalability.
- [Cloud DNS](https://cloud.google.com/dns?hl=en): Translate domain names into IP addresses, enabling users to access websites and services seamlessly.
- [Cloud CDN](https://cloud.google.com/cdn?hl=en#section-1): Accelerate content delivery to users worldwide by caching content in edge locations close to their devices.
- [Cloud NAT](https://cloud.google.com/nat):  Enable instances within a private network to access the internet without requiring public IP addresses, enhancing security and simplifying  network management.

These tools empower businesses to optimize network performance,  improve user experience, and enhance overall network security within the Google Cloud.



## Network connectivity

Google Cloud network connectivity solutions enable seamless  connections between on-premises networks, cloud resources, and other  cloud providers. These solutions include:

- [Cloud VPN](https://cloud.google.com/network-connectivity/docs/vpn/concepts/overview): Establish secure encrypted connections between on-premises networks and VPCs, enabling hybrid cloud deployments.
- [Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/overview): Provide high-bandwidth, low-latency connectivity between on-premises  networks and VPCs, ideal for mission-critical applications.
- [Cross-Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect/concepts/cci-overview): Provides direct, high-bandwidth, low-latency connectivity between Google Cloud and other cloud providers.
- [Network Connectivity Center](https://cloud.google.com/network-connectivity-center?hl=en): Centralized logical hub for managing and monitoring connection. With support for [hybrid spokes](https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview#hybrid_spokes) and [VPC spokes.](https://cloud.google.com/network-connectivity/docs/network-connectivity-center/concepts/overview#vpc-spokes)

These connectivity solutions empower businesses to extend their  existing networks to the cloud, achieve high-performance data transfers, and build complex hybrid and multi cloud architectures.



## Network security

Google Cloud network security solutions provide comprehensive  protection against network threats and vulnerabilities. These solutions  include:

- [Cloud Armor](https://cloud.google.com/security/products/armor?hl=en): Safeguard applications and websites against denial-of-service (DoS) attacks, OWASP top 10 and other malicious traffic.
- [Cloud IDS](https://cloud.google.com/intrusion-detection-system?hl=en) (Intrusion Detection System): Continuously monitor network traffic for  suspicious activity, enabling early detection of potential threats.
- [Cloud Firewall](https://cloud.google.com/firewall?hl=en): Define firewall rules to control incoming and outgoing traffic,  preventing unauthorized access and protecting against cyberattacks.  These also provide advanced capabilities such as [Intrusion Prevention System](https://cloud.google.com/firewall/docs/about-intrusion-prevention) (IPS) for Cloud Firewall Plus editions.

These security solutions empower businesses to enhance network  security, protect sensitive data, and ensure compliance with industry  standards.



## Network Intelligence

Google Cloud [Network Intelligence Center](https://cloud.google.com/network-intelligence-center?hl=en) provides a comprehensive suite of tools for monitoring,  troubleshooting, and optimizing your network performance. These tools  include:

- [Network Topology](https://cloud.google.com/network-intelligence-center/docs/network-topology/concepts/overview): Visualize the topology of your Virtual Private Cloud (VPC) networks and their associated metrics, enabling you to identify and resolve  connectivity issues.
- [Connectivity Tests](https://cloud.google.com/network-intelligence-center/docs/connectivity-tests/concepts/overview): Test network connectivity to and from your VPC network, ensuring that  your network is functioning properly and that your resources are  accessible.
- [Performance Dashboard](https://cloud.google.com/network-intelligence-center/docs/performance-dashboard/concepts/overview): Monitor and visualize the performance of your Google Cloud network and resources.
- [Firewall Insights](https://cloud.google.com/network-intelligence-center/docs/firewall-insights/concepts/overview): Gain insights into firewall rules usage, identify misconfigurations,  and optimize your firewall rules to improve security and performance.
- [Network Analyzer](https://cloud.google.com/network-intelligence-center/docs/network-analyzer/overview): Monitor network traffic and identify potential issues, such as high latency, packet loss, and routing problems.

These network intelligence tools empower businesses to proactively  identify and resolve network issues, maintain network performance, and  enhance overall network health.



## Network Service Tiers

Google Cloud network offers two [service tiers](https://cloud.google.com/network-tiers/docs/overview), Premium Tier and Standard Tier, catering to different performance, availability, and cost requirements.

[Premium Tier](https://cloud.google.com/network-tiers#tab1):

- Global network with low latency: Leverage Google's high-performance global network for global reach and consistent performance.
- High availability and scalability: Ensure continuous availability and seamless scaling for mission-critical applications.
- Ideal for production workloads and demanding applications.

[Standard Tier](https://cloud.google.com/network-tiers#tab2):

- Regional network with cost-effectiveness: Utilize a regional network with lower costs for less demanding workloads.
- Suitable for development, testing, and non-production environments.
- Choose Standard Tier for cost-sensitive scenarios.
