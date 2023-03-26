---

layout: single
title:  "Google Cloud Certified Professional Cloud Network Engineer"
date:   2023-03-25 04:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Google Cloud Certified Professional Cloud Network Engineer

Topics to be leant:
- Designing, planning, and prototyping a Google Cloud network
- Implementing a Virtual Private Cloud (VPC)
- Configuring network service
- Implementing hybrid interconnectivity
- Managing, monitoring, and optimizing network operations

## Designing, planning, and prototyping a Google Cloud network

- Designing an overall network architecture
- Designing a Virtual Private Cloud (VPC)
- Designing a hybrid or multi-cloud network
- Designing an IP addressing plan for Google Kubernetes Engine

### Designing an overall network architecture
Considerations include:
- High availability, failover, and disaster recovery strategy
- DNS strategy (e.g., on-premises, Cloud DNS)
- Security and data exfiltration requirements
- Load balancing
- Applying quotas per project and per VPC
- Hybrid connectivity (e.g., Google private access for hybrid connectivity)
- Container networking
- IAM roles
- SaaS, PaaS, and IaaS services
- Micro-segmentation for security purposes (e.g., using metadata, tags, service accounts)

### Designing a Virtual Private Cloud (VPC)
Considerations include:
- IP Address Management
- Standalone vs. Shared VPC
- Multiple vs. single
- Regional vs. multi-regional
- VPC Network Peering
- Firewalls (e.g., service account–based, tag-based)
- Custom routes

### Designing a hybrid or multi-cloud network
Considerations include:
- Dedicated vs. Partner Interconnect
- Multi-cloud connectivity
- Direct Peering
- IPsec VPN
- Failover and disaster recovery strategy
- Regional vs. global VPC routing mode
- Accessing multiple VPCs from on-premises locations (e.g., Shared VPC, multi-VPC peering topologies)
- Bandwidth and constraints provided by hybrid connectivity solutions
- Accessing Google Services/APIs privately from on-premises locations
- IP Address Management across on-premises locations and cloud
- DNS peering and forwarding

### Designing an IP addressing plan for Google Kubernetes Engine
Considerations include:
- Public and private cluster nodes
- Control plane public vs. private endpoints
- Subnets and alias IPs
- RFC 1918, non-RFC 1918, and privately used public IP (PUPI) address options

## Implementing a Virtual Private Cloud (VPC)

- Configuring VPCs
- Configuring routing
- Configuring and maintaining Google Kubernetes Engine clusters
- Configuring and managing firewall rules
- Implementing VPC Service Controls and Access Contexts

### Configuring VPCs
Considerations include:
- Google Cloud VPC resources (e.g., networks, subnets, firewall rules)
- VPC Network Peering
- Creating a Shared VPC network and sharing subnets with other projects
- Configuring API access to Google services (e.g., Private Google Access,
public interfaces)
- Expanding VPC subnet ranges after creation

### Configuring routing
Considerations include:
- Static versus dynamic routing
- Global versus regional dynamic routing
- Routing policies using tags and priority
- Internal load balancer as a next hop
- Custom route import/export over VPC Peering

### Configuring and maintaining Google Kubernetes Engine clusters
Considerations include:
- VPC-native clusters using alias IP addresses
- Clusters with Shared VPC
- Creating Kubernetes Network Policies
- Private clusters and private control plane endpoints
- Adding authorized networks for cluster control plane endpoints

### Configuring and managing firewall rules
Considerations include:
- Target network tags and service accounts
- Rule priority
- Network protocols
- Ingress and egress rules
- Firewall rule logging
- Firewall Insights
- Hierarchical firewalls

### Implementing VPC Service Controls and Access Contexts
Considerations include:
- VPC Service Control Perimeters
- Creating and configuring Access Contexts and attaching to a Perimeter
- VPC accessible services
- Perimeter Bridges
- Audit logging
- Dry run mode

## Configuring network service
- Configuring load balancing
- Configuring Google Cloud Armor policies
- Configuring Cloud CDN
- Configuring and maintaining Cloud DNS
- Configuring Cloud NAT
- Configuring network packet inspection

### Configuring network packet inspection
Considerations include:
- Backend services and network endpoint groups (NEGs)
- Firewall rules to allow traffic and health checks to backend services
- Health checks for backend services and target instance groups
- Configuring backends and backend services with balancing method (e.g., RPS, CPU,
Custom), session affinity, and capacity scaling/scaler
- TCP and SSL proxy load balancers
- Load balancers (e.g., External TCP/UDP Network Load Balancing, Internal TCP/UDP
Load Balancing, External HTTP(S) Load Balancing, Internal HTTP(S) Load Balancing)
- Protocol Forwarding
- Accommodating workload increases using autoscaling versus manual scaling

### Configuring Google Cloud Armor policies
Considerations include:
- Security policies
- Web application firewall (WAF) rules (e.g., SQL injection, cross-site scripting, remote file inclusion)
- Attaching security policies to load balancer backends

### Configuring Cloud CDN
Considerations include:
- Enabling and disabling Cloud CDN
- Invalidating cached objects
- Signed URLs
- Custom origins

### Configuring and maintaining Cloud DNS
Considerations include:
- Managing zones and records
- Migrating to Cloud DNS
- DNS security (DNSSEC)
- Forwarding and DNS server policies
- Integrating on-premises DNS with Google Cloud
- Split-horizon DNS
- DNS peering
- Private DNS logging

### Configuring Cloud NAT
Considerations include:
- Addressing
- Port allocations
- Customizing timeouts
- Logging and monitoring
- Restrictions per organization policy constraints

### Configuring network packet inspection
Considerations include:
- Packet Mirroring in single and multi-VPC topologies
- Capturing relevant traffic using Packet Mirroring source and traffic filters
- Routing and inspecting inter-VPC traffic using multi-NIC VMs (e.g., next-generation firewall
appliances)
- Configuring an internal load balancer as a next hop for highly available multi-NIC VM routing

## Implementing hybrid interconnectivity
- Configuring Google Cloud Interconnect
- Configuring a site-to-site IPsec VPN
- Configuring Cloud Router

### Configuring Google Cloud Interconnect
Considerations include:
- Dedicated Interconnects and Dedicated
- Interconnect VLAN attachments
-  Partner Interconnect VLAN attachments

### Configuring a site-to-site IPsec VPN
Considerations include:
- High availability VPN (dynamic routing)
- Classic VPN (e.g., route-based routing, policy-based routing)

### Configuring Cloud Router
Considerations include:
- Border Gateway Protocol (BGP) attributes
(e.g., ASN, route priority/MED, link-local addresses)
- Custom route advertisements via BGP
- Deploying reliable and redundant Cloud Routers

## Managing, monitoring, and optimizing network operations
- Logging and monitoring with Google Cloud’s operations suite
- Managing and maintaining security
- Maintaining and troubleshooting connectivity issues
- Monitoring, maintaining, and troubleshooting latency and traffic flow

### Logging and monitoring with Google Cloud’s operations suite
Considerations include:
- Reviewing logs for networking components (e.g., VPN, Cloud
Router, VPC Service Controls)
- Monitoring networking components (e.g., VPN, Cloud
Interconnects and interconnect attachments, Cloud Router, load balancers, Google Cloud Armor, Cloud NAT)

### Managing and maintaining security
Considerations include:
- Firewalls (e.g., cloud-based, private)
- Diagnosing and resolving IAM issues (e.g., Shared VPC,
security/network admin)

### Maintaining and troubleshooting connectivity issues
Considerations include:
- Draining and redirecting traffic flows with HTTP(S) Load Balancing
- Monitoring ingress and egress traffic using VPC flow logs
- Monitoring firewall logs and Firewall Insights
- Managing and troubleshooting VPNs
- Troubleshooting Cloud Router BGP peering issues

### Monitoring, maintaining, and troubleshooting latency and traffic flow
Considerations include:
- Testing network throughput and latency
- Diagnosing routing issues
- Using Network Intelligence Center to visualize topology, test connectivity, and monitor performance