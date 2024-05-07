---

layout: single
title:  "Setting up a Private Kubernetes Cluster"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Setting up a Private Kubernetes Cluster

In Kubernetes Engine, a private cluster is a cluster that makes your  master inaccessible from the public internet. In a private cluster,  nodes do not have public IP addresses, only private addresses, so your  workloads run in an isolated environment. Nodes and masters communicate  with each other using VPC peering.

In the Kubernetes Engine API, address ranges are expressed as Classless Inter-Domain Routing (CIDR) blocks.

## Set the region and zone

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-2ec0cef35421.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud config set compute/zone europe-west1-c
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ export REGION=europe-west1
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ export ZONE=europe-west1-c
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

## Creating a private cluster

1. When you create a private cluster, you must specify a `/28` CIDR range for the VMs that run the Kubernetes master components and you need to enable IP aliases.

Next you'll create a cluster named `private-cluster`, and specify a CIDR range of `172.16.0.16/28` for the masters. When you enable IP aliases, you let Kubernetes Engine automatically create a subnetwork for you.

You'll create the private cluster by using the `--private-cluster`, `--master-ipv4-cidr`, and `--enable-ip-alias` flags.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud beta container clusters create private-cluster \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.16/28 \
    --enable-ip-alias \
    --create-subnetwork ""
Creating cluster private-cluster in europe-west1-c... Cluster is being health-checked (master is healthy)...done.                             
Created [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/clusters/private-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/europe-west1-c/private-cluster?project=qwiklabs-gcp-01-2ec0cef35421
kubeconfig entry generated for private-cluster.
NAME: private-cluster
LOCATION: europe-west1-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.79.86.52
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

## View your subnet and secondary address ranges

1. List the subnets in the default network:

2. In the output, find the name of the subnetwork that was automatically created for your cluster. For example, `gke-private-cluster-subnet-xxxxxxxx`. Save the name of the cluster, you'll use it in the next step.

   Now get information about the automatically created subnet, replacing `[SUBNET_NAME]` with your subnet by running:

The output shows you the primary address range with the name of your GKE private cluster and the secondary ranges:

In the output you can see that one secondary range is for **pods** and the other secondary range is for **services**.

Notice that `privateIPGoogleAccess` is set to `true`. This enables your cluster hosts, which have only private IP addresses, to communicate with Google APIs and services.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute networks subnets list --network default
NAME: default
REGION: us-central1
NETWORK: default
RANGE: 10.128.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west1
NETWORK: default
RANGE: 10.132.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: gke-private-cluster-subnet-4299284a
REGION: europe-west1
NETWORK: default
RANGE: 10.125.156.0/22
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-west1
NETWORK: default
RANGE: 10.138.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: asia-east1
NETWORK: default
RANGE: 10.140.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-east1
NETWORK: default
RANGE: 10.142.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: asia-northeast1
NETWORK: default
RANGE: 10.146.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: asia-southeast1
NETWORK: default
RANGE: 10.148.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-east4
NETWORK: default
RANGE: 10.150.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: australia-southeast1
NETWORK: default
RANGE: 10.152.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west2
NETWORK: default
RANGE: 10.154.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west3
NETWORK: default
RANGE: 10.156.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: asia-south1
NETWORK: default
RANGE: 10.160.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west4
NETWORK: default
RANGE: 10.164.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-north1
NETWORK: default
RANGE: 10.166.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-west2
NETWORK: default
RANGE: 10.168.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-west3
NETWORK: default
RANGE: 10.180.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-west4
NETWORK: default
RANGE: 10.182.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-central2
NETWORK: default
RANGE: 10.186.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: southamerica-west1
NETWORK: default
RANGE: 10.194.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-east7
NETWORK: default
RANGE: 10.196.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-east5
NETWORK: default
RANGE: 10.202.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-south1
NETWORK: default
RANGE: 10.206.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: me-west1
NETWORK: default
RANGE: 10.208.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west12
NETWORK: default
RANGE: 10.210.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: me-central1
NETWORK: default
RANGE: 10.212.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: europe-west10
NETWORK: default
RANGE: 10.214.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: me-central2
NETWORK: default
RANGE: 10.216.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: africa-south1
NETWORK: default
RANGE: 10.218.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: default
REGION: us-west8
NETWORK: default
RANGE: 10.220.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute networks subnets describe  gke-private-cluster-subnet-4299284a --region=$REGION
creationTimestamp: '2024-05-06T19:50:15.341-07:00'
description: auto-created subnetwork for cluster "private-cluster"
fingerprint: 5BW5aDaOndY=
gatewayAddress: 10.125.156.1
id: '4352888197258422280'
ipCidrRange: 10.125.156.0/22
kind: compute#subnetwork
name: gke-private-cluster-subnet-4299284a
network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-2ec0cef35421/global/networks/default
privateIpGoogleAccess: true
privateIpv6GoogleAccess: DISABLE_GOOGLE_ACCESS
purpose: PRIVATE
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-2ec0cef35421/regions/europe-west1
reservedInternalRange: https://networkconnectivity.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/locations/global/internalRanges/gke-private-cluster-subnet-4299284a
secondaryIpRanges:
- ipCidrRange: 10.64.0.0/14
  rangeName: gke-private-cluster-pods-4299284a
  reservedInternalRange: https://networkconnectivity.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/locations/global/internalRanges/gke-private-cluster-pods-4299284a
- ipCidrRange: 10.49.240.0/20
  rangeName: gke-private-cluster-services-4299284a
  reservedInternalRange: https://networkconnectivity.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/locations/global/internalRanges/gke-private-cluster-services-4299284a
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-2ec0cef35421/regions/europe-west1/subnetworks/gke-private-cluster-subnet-4299284a
stackType: IPV4_ONLY
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```



## Enable master authorized networks

At this point, the only IP addresses that have access to the master are the addresses in these ranges:

- The primary range of your subnetwork. This is the range used for nodes.
- The secondary range of your subnetwork that is used for pods.

To provide additional access to the master, you must authorize selected address ranges.



### Create a VM instance

1. Create a source instance which you'll use to check the connectivity to Kubernetes clusters:
2. Get the `<External_IP>` of the `source-instance` with:
3. Copy the `<nat_IP>` address and save it to use in later steps.
4. Run the following to Authorize your external address range, replacing `[MY_EXTERNAL_RANGE]` with the CIDR range of the external addresses from the previous output (your CIDR range is `natIP/32`). With CIDR range as `natIP/32`, we are allowlisting one specific IP address:
5. Now that you have access to the master from a range of external addresses, you'll install `kubectl` so you can use it to get information about your cluster. For example, you can use `kubectl` to verify that your nodes do not have external IP addresses.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute instances create source-instance --zone=$ZONE --scopes 'https://www.googleapis.com/auth/cloud-platform'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/instances/source-instance].
NAME: source-instance
ZONE: europe-west1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 34.38.106.222
STATUS: RUNNING
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
    natIP: 34.38.106.222
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud container clusters update private-cluster     --enable-master-authorized-networks     --master-authorized-networks 34.38.106.222/32
Updating private-cluster...done.                                                                                                                                                   
Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/clusters/private-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/europe-west1-c/private-cluster?project=qwiklabs-gcp-01-2ec0cef35421
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute ssh source-instance --zone=$ZONE
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_c9f851639dd7/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_c9f851639dd7/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_c9f851639dd7/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:Eq+LSoDwRSs+sNUIA4lznECdLZLbfuiTEbuRsAZfGq0 student_01_c9f851639dd7@cs-621429599476-default
The key's randomart image is:
+---[RSA 3072]----+
|O=o.+            |
|+=+B o           |
|ooB.= .          |
|+O.=o  o         |
|=.B== . S        |
| +EO . o         |
|. o * .          |
| . = . .         |
|  ..o .          |
+----[SHA256]-----+
Warning: Permanently added 'compute.9185706435190990803' (ED25519) to the list of known hosts.
Linux source-instance 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-c9f851639dd7'.
student-01-c9f851639dd7@source-instance:~$ 
```

Install `kubectl` component of Cloud-SDK:

```sh
student-01-c9f851639dd7@source-instance:~$ sudo apt-get install kubectl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 51.3 MB of archives.
After this operation, 244 MB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 kubectl amd64 1:471.0.0-0 [51.3 MB]
Fetched 51.3 MB in 1s (51.3 MB/s)  
Selecting previously unselected package kubectl.
(Reading database ... 64633 files and directories currently installed.)
Preparing to unpack .../kubectl_1%3a471.0.0-0_amd64.deb ...
Unpacking kubectl (1:471.0.0-0) ...
Setting up kubectl (1:471.0.0-0) ...
student-01-c9f851639dd7@source-instance:~$ 
```

Configure access to the Kubernetes cluster from SSH shell with:

```sh
student-01-c9f851639dd7@source-instance:~$ sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
gcloud container clusters get-credentials private-cluster --zone=$ZONE
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  google-cloud-cli-gke-gcloud-auth-plugin
The following NEW packages will be installed:
  google-cloud-cli-gke-gcloud-auth-plugin google-cloud-sdk-gke-gcloud-auth-plugin
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 3229 kB of archives.
After this operation, 11.3 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 google-cloud-cli-gke-gcloud-auth-plugin amd64 471.0.0-0 [3224 kB]
Get:2 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 google-cloud-sdk-gke-gcloud-auth-plugin all 467.0.0-0 [5018 B]
Fetched 3229 kB in 0s (29.3 MB/s)                            
Selecting previously unselected package google-cloud-cli-gke-gcloud-auth-plugin.
(Reading database ... 64646 files and directories currently installed.)
Preparing to unpack .../google-cloud-cli-gke-gcloud-auth-plugin_471.0.0-0_amd64.deb ...
Unpacking google-cloud-cli-gke-gcloud-auth-plugin (471.0.0-0) ...
Selecting previously unselected package google-cloud-sdk-gke-gcloud-auth-plugin.
Preparing to unpack .../google-cloud-sdk-gke-gcloud-auth-plugin_467.0.0-0_all.deb ...
Unpacking google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
Setting up google-cloud-cli-gke-gcloud-auth-plugin (471.0.0-0) ...
Setting up google-cloud-sdk-gke-gcloud-auth-plugin (467.0.0-0) ...
ERROR: (gcloud.container.clusters.get-credentials) One of [--location, --zone, --region] must be supplied.
student-01-c9f851639dd7@source-instance:~$ 
student-01-c9f851639dd7@source-instance:~$ export ZONE=europe-west1-c
student-01-c9f851639dd7@source-instance:~$ gcloud container clusters get-credentials private-cluster --zone=$ZONE
Fetching cluster endpoint and auth data.
kubeconfig entry generated for private-cluster.
student-01-c9f851639dd7@source-instance:~$ 
```

Verify that your cluster nodes do not have external IP addresses:

```sh
student-01-c9f851639dd7@source-instance:~$ kubectl get nodes --output yaml | grep -A4 addresses
    addresses:
    - address: 10.125.156.4
      type: InternalIP
    - address: gke-private-cluster-default-pool-2d72369e-3n4d
      type: Hostname
--
    addresses:
    - address: 10.125.156.2
      type: InternalIP
    - address: gke-private-cluster-default-pool-2d72369e-f21r
      type: Hostname
--
    addresses:
    - address: 10.125.156.3
      type: InternalIP
    - address: gke-private-cluster-default-pool-2d72369e-ml7m
      type: Hostname
student-01-c9f851639dd7@source-instance:~$ 
```

The output shows that the nodes have internal IP addresses but do not have external addresses:

Here is another command you can use to verify that your nodes do not have external IP addresses:

```sh
student-01-c9f851639dd7@source-instance:~$ kubectl get nodes --output wide
NAME                                             STATUS   ROLES    AGE   VERSION               INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-private-cluster-default-pool-2d72369e-3n4d   Ready    <none>   15m   v1.28.7-gke.1026000   10.125.156.4   <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-private-cluster-default-pool-2d72369e-f21r   Ready    <none>   15m   v1.28.7-gke.1026000   10.125.156.2   <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-private-cluster-default-pool-2d72369e-ml7m   Ready    <none>   15m   v1.28.7-gke.1026000   10.125.156.3   <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
student-01-c9f851639dd7@source-instance:~$ 
```

## Clean Up

1. Delete the Kubernetes cluster:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud container clusters delete private-cluster --zone=$ZONE
The following clusters will be deleted.
 - [private-cluster] in [europe-west1-c]

Do you want to continue (Y/n)?  y

Deleting cluster private-cluster...done.                                                                                                                                           
Deleted [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/clusters/private-cluster].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

## Create a private cluster that uses a custom subnetwork

In the previous section Kubernetes Engine automatically created a  subnetwork for you. In this section, you'll create your own custom  subnetwork, and then create a private cluster. Your subnetwork has a primary address range and two secondary address  ranges.

Create a subnetwork and secondary ranges:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute networks subnets create my-subnet \
    --network default \
    --range 10.0.4.0/22 \
    --enable-private-ip-google-access \
    --region=$REGION \
    --secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-2ec0cef35421/regions/europe-west1/subnetworks/my-subnet].
NAME: my-subnet
REGION: europe-west1
NETWORK: default
RANGE: 10.0.4.0/22
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```



Create a private cluster that uses your subnetwork:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud beta container clusters create private-cluster2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.32/28 \
    --subnetwork my-subnet \
    --services-secondary-range-name my-svc-range \
    --cluster-secondary-range-name my-pod-range \
    --zone=$ZONE
Creating cluster private-cluster2 in europe-west1-c... Cluster is being health-checked...working.           Creating cluster private-cluster2 in europe-west1-c... Cluster is being health-checked (master is healthy)...working...
Creating cluster private-cluster2 in europe-west1-c... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/clusters/private-cluster2].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/europe-west1-c/private-cluster2?project=qwiklabs-gcp-01-2ec0cef35421
kubeconfig entry generated for private-cluster2.
NAME: private-cluster2
LOCATION: europe-west1-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 35.240.19.65
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute instances describe source-instance --zone=$ZONE | grep natIP
    natIP: 34.38.106.222
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```



Run the following to Authorize your external address range, replacing `[MY_EXTERNAL_RANGE]` with the CIDR range of the external addresses from the previous output (your CIDR range is `natIP/32`). With CIDR range as `natIP/32`, we are allowlisting one specific IP address:

1. At this point, the only IP addresses that have access to the master are the addresses in these ranges:
   - The primary range of your subnetwork. This is the range used for nodes. In this example, the range for nodes is `10.0.4.0/22`.
   - The secondary range of your subnetwork that is used for pods. In this example, the range for pods is `10.4.0.0/14`.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud container clusters update private-cluster2     --enable-master-authorized-networks     --zone=$ZONE     --master-authorized-networks 34.38.106.222/32
Updating private-cluster2...done.                                                                                                           
Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-2ec0cef35421/zones/europe-west1-c/clusters/private-cluster2].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/europe-west1-c/private-cluster2?project=qwiklabs-gcp-01-2ec0cef35421
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-01-2ec0cef35421)$ gcloud compute ssh source-instance --zone=$ZONE
Linux source-instance 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May  7 03:08:13 2024 from 35.186.144.217
student-01-c9f851639dd7@source-instance:~$ 
```

```sh
student-01-c9f851639dd7@source-instance:~$ gcloud container clusters get-credentials private-cluster2 --zone=europe-west1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for private-cluster2.
student-01-c9f851639dd7@source-instance:~$ 
```

```sh
student-01-c9f851639dd7@source-instance:~$ kubectl get nodes --output yaml | grep -A4 addresses
    addresses:
    - address: 10.0.4.4
      type: InternalIP
    - address: gke-private-cluster2-default-pool-92bd966f-61cl
      type: Hostname
--
    addresses:
    - address: 10.0.4.2
      type: InternalIP
    - address: gke-private-cluster2-default-pool-92bd966f-8l8s
      type: Hostname
--
    addresses:
    - address: 10.0.4.3
      type: InternalIP
    - address: gke-private-cluster2-default-pool-92bd966f-blkl
      type: Hostname
student-01-c9f851639dd7@source-instance:~$ 
```

```sh
student-01-c9f851639dd7@source-instance:~$ kubectl get nodes -o wide
NAME                                              STATUS   ROLES    AGE     VERSION               INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-private-cluster2-default-pool-92bd966f-61cl   Ready    <none>   4m38s   v1.28.7-gke.1026000   10.0.4.4      <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-private-cluster2-default-pool-92bd966f-8l8s   Ready    <none>   4m38s   v1.28.7-gke.1026000   10.0.4.2      <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-private-cluster2-default-pool-92bd966f-blkl   Ready    <none>   4m38s   v1.28.7-gke.1026000   10.0.4.3      <none>        Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
student-01-c9f851639dd7@source-instance:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                                         READY   STATUS    RESTARTS        AGE     IP          NODE                                              NOMINATED NODE   READINESS GATES
gmp-system    alertmanager-0                                               2/2     Running   0               5m31s   10.4.2.10   gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
gmp-system    collector-6hvtn                                              2/2     Running   0               4m13s   10.4.0.3    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
gmp-system    collector-kt5bz                                              2/2     Running   0               4m14s   10.4.1.3    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
gmp-system    collector-pmpd2                                              2/2     Running   0               4m14s   10.4.2.12   gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
gmp-system    gmp-operator-7b97b84cf9-zzl2p                                1/1     Running   0               5m33s   10.4.2.3    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
gmp-system    rule-evaluator-79f4bf44b7-8kgfz                              2/2     Running   2 (3m56s ago)   4m15s   10.4.0.2    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   event-exporter-gke-7d996c57bf-9pvlw                          2/2     Running   0               5m53s   10.4.2.6    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   fluentbit-gke-ql9p4                                          2/2     Running   0               4m44s   10.0.4.4    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   fluentbit-gke-tnxxj                                          2/2     Running   0               4m43s   10.0.4.3    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
kube-system   fluentbit-gke-w7pnb                                          2/2     Running   0               4m43s   10.0.4.2    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   gke-metrics-agent-6489l                                      2/2     Running   0               4m41s   10.0.4.3    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
kube-system   gke-metrics-agent-6c9hn                                      2/2     Running   0               4m41s   10.0.4.2    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   gke-metrics-agent-6chrm                                      2/2     Running   0               4m45s   10.0.4.4    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   konnectivity-agent-6fbf5bc986-g27qc                          2/2     Running   0               5m43s   10.4.2.7    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   konnectivity-agent-6fbf5bc986-rmsmq                          2/2     Running   0               4m13s   10.4.1.4    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
kube-system   konnectivity-agent-6fbf5bc986-wjqch                          2/2     Running   0               4m13s   10.4.0.4    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   konnectivity-agent-autoscaler-5847cf65c7-6pgx5               1/1     Running   0               5m41s   10.4.2.8    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   kube-dns-6f955b858b-4tpc2                                    4/4     Running   0               4m15s   10.4.1.2    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
kube-system   kube-dns-6f955b858b-hz44l                                    4/4     Running   0               5m54s   10.4.2.4    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   kube-dns-autoscaler-755c7dfdf5-mpbgj                         1/1     Running   0               5m54s   10.4.2.5    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   kube-proxy-gke-private-cluster2-default-pool-92bd966f-61cl   1/1     Running   0               3m39s   10.0.4.4    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   kube-proxy-gke-private-cluster2-default-pool-92bd966f-8l8s   1/1     Running   0               4m34s   10.0.4.2    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   kube-proxy-gke-private-cluster2-default-pool-92bd966f-blkl   1/1     Running   0               3m20s   10.0.4.3    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
kube-system   l7-default-backend-6779bb6c8d-frjzq                          1/1     Running   0               5m38s   10.4.2.2    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   metrics-server-v0.6.3-764c8d87d9-m2hsf                       2/2     Running   0               4m3s    10.4.0.5    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   pdcsi-node-4rg2c                                             2/2     Running   0               4m44s   10.0.4.4    gke-private-cluster2-default-pool-92bd966f-61cl   <none>           <none>
kube-system   pdcsi-node-qrlc7                                             2/2     Running   0               4m43s   10.0.4.2    gke-private-cluster2-default-pool-92bd966f-8l8s   <none>           <none>
kube-system   pdcsi-node-sm8hp                                             2/2     Running   0               4m43s   10.0.4.3    gke-private-cluster2-default-pool-92bd966f-blkl   <none>           <none>
student-01-c9f851639dd7@source-instance:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.0.32.1    <none>        443/TCP   6m17s
student-01-c9f851639dd7@source-instance:~$ 
```

