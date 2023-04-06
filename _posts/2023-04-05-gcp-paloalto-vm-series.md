---


layout: single
title:  "Scaling VM-Series to Secure Google Cloud Networks"
date:   2023-04-05 07:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Scaling VM-Series to Secure Google Cloud Networks

This lab shows how to deploy and scale Palo Alto Networks VM-Series Next Generation Firewall to secure a hub and spoke architecture in Google  Cloud.  The VM-Series enables enterprises to secure their applications,  users, and data deployed across Google Cloud and other virtualization  environments.

The VM-Series prevention capabilities include:

1. Prevent inbound threats from the internet to resources deployed in VPC networks, on-premises, and in other cloud environments.

2. Stop outbound connections from VPC networks to suspicious  destinations, such as command-and-control servers, or malicious code  repositories.

3. Prevent threats from moving laterally between workload VPC networks and stealing data.

4. Secure traffic between remote networks connected through Cloud Interconnect, Network Connectivity Center, or Cloud VPN.

5. Extend security to remote users and mobile devices to provide granular application access to Google Cloud resources.

   

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-601.png)



### Objectives

- Review hub-and-spoke network topology secured by VM-Series
- Access the VM-Series Firewall
- Safely enable applications with App-ID
- Autoscale the VM-Series



Topology

Below is a diagram of the lab environment.  VM-Series firewalls are  deployed within a regional managed instance group to secure north/south  and east/west traffic for two spoke VPC networks.

The VM-Series inspects traffic as follows:

1. Traffic from the internet to applications in the spoke networks are  distributed by the External TCP/UDP Load Balancer to the VM-Series  untrust interfaces (NIC0). The VM-Series inspects the traffic and  forwards permissible traffic through its trust interface (NIC2) to the  application in the spoke network.
2. Traffic from the spoke networks destined to the internet is routed to the Internal TCP/UDP Load Balancer in the hub VPC. The VM-Series  inspects the traffic and forwards permissible traffic through its  untrust interface (NIC0) to the internet.
3. Traffic between spoke networks is routed to the Internal TCP/UDP Load Balancer in the hub VPC. The VM-Series inspects and forwards the  traffic through the trust interface (NIC2) into the hub network which  routes permissible traffic to the destination spoke network.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-602.png)



## 1. Access the VM-Series firewall



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-509.png)



## 2. Secure internet inbound traffic

Internet traffic is distributed by an external TCP/UDP load balancer  to the VM-Series untrust interfaces. The VM-Series inspects and  translates the traffic to `VM A` in the `spoke 1` network. `VM A`  runs a generic web service and Jenkins.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-510.png)

## 3. Autoscale the VM-Series

This lab uses a regional managed instance group to deploy and scale  VM-Series firewalls across zones within a region.  Autoscaling enables  you to scale the security protecting your cloud assets while providing  high availability through cross-zone redundancy.
