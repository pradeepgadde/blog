---

layout: single
title:  "Deploying workloads on Anthos clusters on bare metal"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Deploying workloads on Anthos clusters on bare metal

This lab is the second in a series of labs, each of which is intended to build skills related to the setup and operation of Anthos clusters on bare metal. In this lab, you start with the admin workstation and admin cluster in place; you then build the user cluster. After the user cluster is running, you deploy stateless and stateful workloads and expose the workloads using LoadBalancer services and Ingresses.

- Configure and create your Anthos on bare metal user cluster.
- Launch workloads on your user cluster.
- Expose L4 and L7 services on your created user cluster using the bundled MetalLB load balancer.
- Install a CSI driver and deploy stateful workloads.

## Prepare your environment and connect to the admin cluster

To reflect real-world best practices, your project has been configured as follows:    

-            The **Default** network has been deleted.        
-             A customer subnet network has been created.        
-             Several firewall rules have been created:            
  - **abm-allow-cp**: allows traffic to the control plane servers.                
  - **abm-allow-worker**: allows inbound traffic  to the worker nodes.                
  - **abm-allow-lb / abm-allow-gfe-to-lb**: allows inbound traffic to the load balancer nodes. In our case, the load balancer is hosted in the same node as the admin cluster control plane node.                
  - **abm-allow-multi**: allows multicluster traffic. This allows the communication between the admin and the user cluster.                
  - **iap**: allows traffic from Identity Aware Proxy (IAP), so you can SSH internal VMs without opening port 22 to the internet.                
  - **vxlan**: allow vxlan networking, a network virtualization technology that encapsulates L2 Ethernet frames on an underlying L3 network.                
-             Your admin workstation has been created.        
- Your admin cluster has been created.        

![Lab Diagram](https://cdn.qwiklabs.com/nXEJJSXWcP0E3uPZFr2ciNXPnzjI16PliiCk1peGwIg%3D)

1. Set the Zone environment variable
