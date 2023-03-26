---

layout: single
title:  "Implementing a Virtual Private Cloud (VPC)"
date:   2023-03-25 06:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
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

# Implementing a Virtual Private Cloud (VPC)

## Configuring VPCs

When configuring VPC networks and their subnetworks, you can expand the primary range of a subnet (to max /16 size in auto networks or to the size supported by the IP block in custom networks) as long as the expansion does not introduce overlap to other existing subnets. When doing such expansion, any firewall rules depending on the older range should be updated.

Secondary ranges cannot be expanded or changed and must be deleted and recreated. Secondary ranges are used by an alias IP allowing secondary internal IP addresses to be assigned to the VM, typically for container or pod networking.

Deleting a subnet first requires deleting all VMs in that subnet.

Many traffic control scenarios can be accomplished leaving the default routes and adding, removing, or changing only the firewall rules. You can set multiple rules to be applied in priority order - but remember to account for the implicit deny all ingress and allow all egress rules at lowest priority.

Shared VPC is a centralized networking model (for billing, administration, and access control) whereas VPC peering is a decentralized approach. VPC peering can work across organizations whereas Shared VPC can only work within organizations. VPC peering does not support transitive peering and requires any two connected VPCs to be directly peered.

## Configuring routing

HA VPN only supports dynamic routing with BGP and can’t be used if the peer on-premises VPN gateway does not support BGP. Classic VPN supports static routing, either route-based or policy-based. Use Classic VPN when the on-premises VPN gateway does not support BGP. HA VPN provides 99.99% availability; Classic VPN only supports 99.9% availability. HA VPN is the recommended approach whenever the on-premises VPN gateway supports BGP.

Dynamic routing requires a Cloud Router. Dynamic routing is simpler to configure and maintain than static routing because route changes in either connected network can be discovered and advertised automatically. The Dynamic routing mode of the VPC determines whether Cloud Routers will use regional or global dynamic routing or dynamic routing. A single VPN connection can provide connectivity across all subnets and regions within a VPC when the VPC is in global dynamic routing mode. However, the average latency will not be as low as if there are separate Cloud VPN gateways and Cloud Routers per region.

Most standard network routing can be accomplished using the default created routes. In some scenarios, traffic must be forwarded through a NAT or Proxy instance, or routed through an intermediary different than the destination IP address. This can be accomplished by creating custom routes. When creating a custom route, you can specify a destination IP address or range and the next hop instance, load balancer, IP address, internet gateway, or VPN gateway. You can limit which VMs would use a custom route by adding a tag to the custom route that matches a tag on the appropriate VMs.



## Configuring and maintaining Google Kubernetes Engine clusters

For VPC-native clusters, secondary subnet ranges are used for pod and service IP ranges. You can either pre-create the secondary ranges or simply specify them when creating the cluster. Each node will support up to 110 pods. The primary IP range needs to be large enough that when the number of nodes is multiplied by 110, the supported number of pods will be sufficient. The size of the pod range also needs to be large enough to provide one IP address per pod for the maximum number of pods. The service range should be large enough to provide one IP address for the maximum number of services



When running GKE clusters in a Shared VPC, some extra IAM role bindings must be granted to the Kubernetes Engine service agent to let the GKE cluster create the necessary networking resources. It requires the Compute Network User role for the Shared VPC subnet of the GKE cluster, as well as the Host Service Agent User for the Shared VPC host project.

## Configuring and managing firewall rules

Firewall rules can be assigned to operate on all instances in a VPC. You can also assign them to specific VMs using source or destination IP addresses or IP ranges. You can also assign firewall rules using source and target service accounts or tags. In general using source and target service accounts is the recommended approach and provides the best security.



Firewall Logging and Firewall Insights can be useful tools for troubleshooting firewall configuration problems. They must be enabled and properly configured before usage. Firewall Insights provides high-level details for quick analysis and detection of problems, such as shadowed rules, overly permissive rules, and active deny rules. It doesn’t directly provide detailed information about the firewall rule activity.

Firewall Insights can be combined with filtering on matching firewall logs to get more of the details. Firewall logs capture all of the detailed information about firewall activity but can produce large logs. These logs may require filtering to effectively find activity related to the traffic flows of interest. When troubleshooting, it’s also important to remember that logs and insights are not captured for the implicit deny all ingress and allow all egress firewall rules.

## Implementing VPC Service Controls and Access Contexts

VPC Service controls provide another option for access control and restriction along with Cloud IAM and Firewall rules. They allow for the creation of service perimeters around one or more projects that can allow or restrict access to Google Cloud APIs in those projects from inside (VPCs within the project) and outside those projects (VPCs in other projects or the internet). Service perimeter access control can be tailored with access levels, ingress and egress rules, and service perimeter bridges allowing for flexibility in how access is enabled or disabled.



Access Levels, Service perimeter bridges, and Ingress and Egress rules can all be used to adjust the access control provided by VPC service control service perimeters.

Access levels allow external access across service perimeters based on the attributes of the request (such as IP address, identity, geolocation, and device type). 

Service perimeter bridges provide bi-directional access to all the projects of both service perimeters on either side of the service perimeter bridge.

 Ingress and Egress rules allow for more granular control of access across service perimeter boundaries as compared to service perimeter bridges. Ingress and Egress rules can also include and adjust the access provided by access levels.