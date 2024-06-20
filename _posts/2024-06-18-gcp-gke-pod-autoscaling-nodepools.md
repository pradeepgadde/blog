---

layout: single
title:  "Configuring Pod Autoscaling and NodePools"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Configuring Pod Autoscaling and NodePools

- Configure autoscaling and HorizontalPodAutoscaler
- Add a node pool and configure taints on the nodes for Pod anti-affinity
- Configure an exception for the node taint by adding a toleration to a Pod's manifest

## Connect to the lab GKE cluster and deploy a sample workload

In this task, you connect to the lab GKE cluster and create a deployment manifest for a set of Pods within the cluster.





## Configure autoscaling on the cluster

In this task, you configure the cluster to automatically scale the sample application that you deployed earlier.



## Manage node pools

In this task, you create a new pool of nodes using preemptible  instances, and then you constrain the web deployment to run only on the  preemptible nodes.
