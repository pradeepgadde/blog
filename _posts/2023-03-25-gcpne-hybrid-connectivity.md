---

layout: single
title:  "Implementing Hybrid Connectivity"
date:   2023-03-25 07:59:04 +0530
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

# Implementing Hybrid Connectivity

## Configuring Google Cloud Interconnect

Partner Interconnect provides private connectivity between on-premises networks and Google Cloud through a partner network. The two main classes of Partner Interconnect are Layer 2 and Layer 3 Partner Interconnect. Both approaches have similar setup, but for Layer 2 BGP configuration must be done for the on-premises routers, because the BGP session is established between them and the Cloud Routers in Google Cloud. For Layer 3, the BGP configuration is done in the partner’s routers.

Dedicated Interconnect provides the highest capacity connectivity between Google Cloud and on-premises networks and supports 1-8 10 Gbps or 1-2 100 Gbps circuits per connection.

Multiple VLAN attachments are necessary to provide for high availability configurations for both Dedicated and Partner Interconnect. Dedicated Interconnect also requires multiple connections for high availability. Multiple VLAN attachments may also be required to provide for the capacity requirements of connections as they have a maximum capacity of 50 Gbps whereas connections can have capacities up to 200 Gbps.

To share an Interconnect connection to on-premises infrastructure across multiple projects you can use Shared VPC or VPC peering. For projects with their own VPC networks, you can create separate VLAN attachments and Cloud Routers per project.

Shared VPC is the recommended approach as the configuration is simpler and the solution is easier to scale compared to VPC peering and cheaper than having separate VLAN attachments per project. In Shared VPC, the Interconnect and associated resources should all be created in the Shared VPC host project.

## Configuring a site-to-site IPsec VPN

Google Cloud has HA and Classic VPN gateways. HA VPN gateways only support dynamic routing with BGP while Classic VPN can be used with dynamic or static routing. Classic VPN with static routing supports route-based or policy-based tunnels.

Classic VPN route-based tunnels use 0.0.0.0/0 as local and remote traffic selectors and policy-based tunnels allow configuration of both. The Classic VPN gateway local and remote selectors should match the peer VPN gateway remote and local selectors.



When creating VPN gateways with the gcloud tool, the routes directing appropriate traffic to the gateway are not created automatically and require creation with separate gcloud commands.



HA VPN gateways provide 2 interfaces, and both must have tunnels to peer VPN gateway interfaces with BGP sessions to provide 99.99% availability. When configuring HA VPN gateways, an external VPN gateway resource must be created that matches the number of interfaces available on the peer VPN gateway.

## Configuring Cloud Router

When creating BGP sessions in Cloud Routers for VPN tunnels or Interconnect VLAN attachments, the base advertised route priority can be configured for the BGP session. That value is sent as a multi-exit discriminator (MED) attribute. That particular tunnel or attachment is typically preferred, because lower values are preferred to higher values with all else being equal. Two BGP sessions with equal advertised priority would be equally preferred (active/active) and with different values, one would be prioritized (active/passive).



Cloud Routers have default and custom route advertisement modes that can be set for the router as a whole or separately for each BGP session. The default advertisement mode will advertise all subnets in the same region when the VPC in the Cloud Router is set to regional dynamic routing mode, or all subnets in all regions when the VPC is set to global dynamic routing mode. When in custom route advertisement mode, the Cloud Router can be configured to advertise a specified set of IP ranges. In addition, Cloud Router can also be configured to advertise all subnets in the region or across all regions, based on the VPC’s dynamic routing mode.



For 99.99% availability for Dedicated and Partner Interconnect, 2 Cloud Routers in distinct regions are required. To achieve this same 99.99% availability with HA VPN, only a single Cloud Router in a single region is necessary.
