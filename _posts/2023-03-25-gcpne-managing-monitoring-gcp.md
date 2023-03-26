---

layout: single
title:  "Managing, monitoring, and optimizing network operations"
date:   2023-03-25 08:59:04 +0530
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

# Managing, monitoring, and optimizing network operations

## Logging and monitoring with Google Cloud’s operations suite

As with most networking services in Google Cloud there are logs collected related to the functionality of the HTTP(S) Load Balancer. These logs also include logs related to functionality of the Cloud CDN and Cloud Armor which are tightly integrated with the HTTP(S) LB. Backend buckets will provide logs automatically, but backend services require configuration to enable logs and set the logs sampling rate.

Most networking features and functions have associated metrics automatically collected by Cloud Monitoring that can be visually tracked over time via charts/dashboards, or alerted on when crossing safe thresholds. For Cloud VPN tunnel capacity, the limitations in bytes and packets per second can be monitored via the associated bytes and packets counts metrics.

## Managing and maintaining security

Firewall Insights can provide high-level details about the behavior of firewall rules and help diagnose problems. Firewall Insights requires you to enable the associated API, as well as enabling firewall rules logging for all rules to be included in the insights.

Several insights, such as the Deny rules with hits, also require configuring an observation period over which the insight will be calculated.



Policy Intelligence provides several tools for analyzing, simulating, troubleshooting, and recommending Cloud IAM policies. The Policy Analyzer would be the best tool in this or similar scenarios where the goal is to to determine which identities have what sort of access to which resources.

## Maintaining and troubleshooting connectivity issues

VPC flow logs can be enabled and configured on a subnet level and capture a maximum of 10% of traffic. That amount can be further reduced by sampling and/or filtering and is aggregated over a configurable period which defaults to 5 seconds.

Log entries are created for the aggregate traffic from the period and information about the source and destination IP addresses and ports, the protocol, and other useful information is provided in the log entry.

Cloud VPN troubleshooting will depend on the type of Cloud VPN tunnel. Classic VPN is primarily used in static routing configurations to create route-based or policy-based tunnels. When using policy-based tunnels, the local and remote traffic selector configuration may result in only partial connectivity between resources on opposite sides of the tunnel.



For Partner Interconnect, all the Cloud Routers must have a local ASN of 16550.

Dedicated Interconnect and Cloud VPN do not have this requirement. ASN misconfiguration in the Cloud Router or on-premises router is a common cause of failure to establish a BGP session in the Cloud Router.

## Monitoring, maintaining, and troubleshooting latency and traffic flow

Routing problems can be caused by misconfiguring VPC routes, firewall rules, firewall software, VPNs, interconnect links, or Cloud Routers. Routing problems can also be caused by malfunctioning or misconfigured software, or transient Google Cloud networking issues. The Network Intelligence Center includes these tools: Network Topology Connectivity Tests, Performance Dashboard, and Firewall Insights. Use these tools to help identify transient Google Cloud networking issues, as well as VPC or hybrid link configuration problems.

**Network Intelligence Center** tools provide simple network monitoring and debugging capabilities to help identify problems, as well as monitor performance of your VPC networks and connectivity to on-premises networks. The Network Topology tool provides a topological visualization of your VPC networks and the infrastructure within, as well as links to on-premises networks and the internet. The tool also provides metrics on traffic and latency between resources and across links. The Performance Dashboard tool provides metrics on packet loss and latency between zones within and across regions. The Connectivity Tests tool provides static and dynamic connectivity tests to verify if the network configuration is blocking connectivity. The Firewall Insights tool provides details and metrics about the behavior of the firewall configuration.
