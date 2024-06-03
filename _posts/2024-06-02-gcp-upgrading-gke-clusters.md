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

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-7f5a9810db45.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ gcloud beta container --project "qwiklabs-gcp-00-7f5a9810db45" clusters create "cluster-1" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.26.15-gke.1090000" --release-channel "None" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/qwiklabs-gcp-00-7f5a9810db45/global/networks/default" --subnetwork "projects/qwiklabs-gcp-00-7f5a9810db45/regions/us-central1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --security-posture=disabled --workload-vulnerability-scanning=disabled --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --binauthz-evaluation-mode=DISABLED --no-enable-managed-prometheus --enable-shielded-nodes --node-locations "us-central1-c"
Creating cluster cluster-1 in us-central1-c... Cluster is being health-checked (master is healthy)...done.                                  
Created [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-00-7f5a9810db45/zones/us-central1-c/clusters/cluster-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-c/cluster-1?project=qwiklabs-gcp-00-7f5a9810db45
kubeconfig entry generated for cluster-1.
NAME: cluster-1
LOCATION: us-central1-c
MASTER_VERSION: 1.26.15-gke.1090000
MASTER_IP: 104.154.241.19
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.26.15-gke.1090000
NUM_NODES: 3
STATUS: RUNNING
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ kubectl get nodes -o wide
NAME                                       STATUS   ROLES    AGE   VERSION                INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cluster-1-default-pool-4d839da4-jttf   Ready    <none>   11m   v1.26.15-gke.1090000   10.128.0.5    34.71.122.4      Container-Optimized OS from Google   5.15.146+        containerd://1.6.28
gke-cluster-1-default-pool-4d839da4-rq4h   Ready    <none>   11m   v1.26.15-gke.1090000   10.128.0.4    34.123.18.100    Container-Optimized OS from Google   5.15.146+        containerd://1.6.28
gke-cluster-1-default-pool-4d839da4-x19m   Ready    <none>   11m   v1.26.15-gke.1090000   10.128.0.3    104.197.240.94   Container-Optimized OS from Google   5.15.146+        containerd://1.6.28
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ 
```



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

```sh
Node pools with auto-upgrade enabled have upcoming node upgrade to 1.27.14-gke.1022000. Learn more
```

In the upgrade wizard window, choose the most recent version (at the top of the list), and then click Upgrade to continue with the upgrade.

Because this process must upgrade all nodes in your cluster, it might take several minutes to complete.

You can check the status by refreshing the web browser. A progress bar appears.

When the node pool upgrade process finishes, your cluster upgrade is complete.

```sh
Updating nodes in the node pool default-pool. For node version upgrades, it typically takes 4-5 minutes to upgrade a single node or longer (e.g., due to pod disruption budget or grace period). For updates to node metadata like kubernetes labels, taints and tags, it takes less than a minute per node and it does not recreate the nodes or cause any disruption to running workloads. Learn more
33%
33% -- 1 out of 3 complete. 
```

```sh
Updating nodes in the node pool default-pool. For node version upgrades, it typically takes 4-5 minutes to upgrade a single node or longer (e.g., due to pod disruption budget or grace period). For updates to node metadata like kubernetes labels, taints and tags, it takes less than a minute per node and it does not recreate the nodes or cause any disruption to running workloads. Learn more
67%
67% -- 2 out of 3 complete. 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ kubectl get nodes -o wide
NAME                                       STATUS   ROLES    AGE     VERSION                INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cluster-1-default-pool-4d839da4-6qp0   Ready    <none>   8m21s   v1.27.14-gke.1022000   10.128.0.8    34.123.18.100    Container-Optimized OS from Google   5.15.154+        containerd://1.7.15
gke-cluster-1-default-pool-4d839da4-edvk   Ready    <none>   11m     v1.27.14-gke.1022000   10.128.0.7    34.71.122.4      Container-Optimized OS from Google   5.15.154+        containerd://1.7.15
gke-cluster-1-default-pool-4d839da4-h8uk   Ready    <none>   15m     v1.27.14-gke.1022000   10.128.0.6    34.170.101.170   Container-Optimized OS from Google   5.15.154+        containerd://1.7.15
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-00-7f5a9810db45)$ 
```

