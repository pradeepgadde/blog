---

layout: single
title:  "Upgrading Google Kubernetes Engine Clusters"
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
# Upgrading Google Kubernetes Engine Clusters 

Use the Google Cloud Console to upgrade your GKE cluster.

## Deploy a GKE cluster

In this task, you use Google Cloud Console to deploy a GKE cluster running a Kubernetes version that is not the most recent release. You will upgrade this cluster to a more recent release in a later task.

In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click Kubernetes Engine > Clusters.
    
Click Create to begin creating a GKE cluster.
    
Click Switch to Standard Cluster in the top right of the screen to switch operation modes.
    
Click Switch to Standard Cluster to confirm choice

In the Release Channel section, choose No channel (not recommended) as the release channel, and then click on the version dropdown to expand the list of available Kubernetes versions.


Select the lowest version number in the list.

Leave all of the other settings at the defaults and click Create to begin creating a GKE cluster.

## Upgrade your GKE cluster

In this task, you upgrade your GKE cluster and all of the nodes.
Upgrade your GKE cluster

In the Google Cloud Console, on the Navigation menu (Navigation menu icon), in the Kubernetes Engine section, click Clusters.

Click the name of your cluster to open its properties.

To the right of the Release channel version, click the Upgrade available link to start the upgrade wizard.

Select the most recent (highest) build available.

Click the checkbox for Acknowledgement and then click Save Change.

> Version numbers are presented in the following format major.minor.patch.For Example in the version 1.14.10, 1.14.10 is the major version, 1.14.10 is the minor version, and 1.14.10 is the patch version.

> If the version you are upgrading to is more than 1 minor version away from the current version. You may have to do this step in stages.

> For Example: I will upgrade from 1.14.10 to 1.15.11 first, then I will upgrade from 1.15.11 to my most recent version (1.16.9). 

You need to wait 2 or 3 minutes for the Control Plane upgrade to complete.
When the control plane upgrade has completed, the control plane version number changes to the version that you selected in the upgrade wizard.

Upgrade the node pool in your cluster

You must now upgrade the nodes of your cluster to the same version as the control plane.

Refresh your web browser to display the prompt, and then click Upgrade oldest node pool.

In the upgrade wizard window, choose the most recent version (at the top of the list), and then click Upgrade to continue with the upgrade.

Because this process must upgrade all nodes in your cluster, it might take several minutes to complete.

You can check the status by refreshing the web browser. A progress bar appears.

When the node pool upgrade process finishes, your cluster upgrade is complete.
