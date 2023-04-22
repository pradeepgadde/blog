---
layout: single
title:  "VPC Networking in Google Cloud"
date:   2023-04-21 14:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# VPC Networking

## Overview

Google Cloud Virtual Private Cloud (VPC) provides networking  functionality to Compute Engine virtual machine (VM) instances,  Kubernetes Engine containers, and the App Engine flexible environment.  In other words, without a VPC network, you cannot create VM instances,  containers, or App Engine applications. Therefore, each Google Cloud  project has a **default** network to get you started.

You can think of a VPC network as similar to a physical network,  except that it is virtualized within Google Cloud. A VPC network is a  global resource that consists of a list of regional virtual subnetworks  (subnets) in data centers, all connected by a global wide area network  (WAN). VPC networks are logically isolated from each other in Google  Cloud.

In this lab, you create an auto mode VPC network with firewall rules  and two VM instances. Then, you convert the auto mode network to a  custom mode network and create other custom mode networks as shown in  the example network diagram below. You also test connectivity across  networks.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-1.png)

Perform the following tasks:

- Explore the default VPC network
- Create an auto mode network with firewall rules
- Convert an auto mode network to a custom mode network
- Create custom mode VPC networks with firewall rules
- Create VM instances using Compute Engine
- Explore the connectivity for VM instances across VPC networks



## 1. Explore the default network

Each Google Cloud project has a **default** network with subnets, routes, and firewall rules.

### **View the subnets**

### **View the routes**

Routes tell VM instances and the VPC network how to send traffic from an instance to a destination, either inside the network or outside   Google Cloud. Each VPC network comes with some default routes to route  traffic among its subnets and send traffic from eligible instances to  the internet.

### **View the firewall rules**

Each VPC network implements a distributed virtual firewall that you  can configure. Firewall rules allow you to control which packets are  allowed to travel to which destinations. Every VPC network has two  implied firewall rules that block all incoming connections and allow all outgoing connections.

### **Delete the Firewall rules**

1. In the left pane, click **Firewall**.
2. Select all default network firewall rules.
3. Click **Delete**.
4. Click **Delete** to confirm the deletion of the firewall rules.

### **Delete the default network**

1. In the left pane, click  **VPC networks**.

2. Select the **default** network.

3. Click **Delete VPC network**.

4. Click **Delete** to confirm the deletion of the **default** network

5. Wait for the network to be deleted before continuing.

   1. In the left pane, click  **Routes**.

   Notice that there are no routes.

   1. In the left pane, click  **Firewall**.

   Notice that there are no firewall rules.

### **Try to create a VM instance**

Verify that you cannot create a VM instance without a VPC network.

## 2. Create an auto mode network

You have been tasked to create an auto mode network with two VM  instances. Auto mode networks are easy to set up and use because they  automatically create subnets in each region. However, you don't have  complete control over the subnets created in your VPC network, including regions and IP address ranges used.

### **Create an auto mode VPC network with firewall rules**

1. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VPC network** > **VPC networks**.
2. Click **Create VPC network**.
3. For **Name**, type **mynetwork**
4. For **Subnet creation mode**, click **Automatic**.

Auto mode networks create subnets in each region automatically.

1. For **Firewall rules**, select all available rules.

These are the same standard firewall rules that the default network had. The **deny-all-ingress** and **allow-all-egress** rules are also displayed, but you cannot select or disable them because they are implied. These two rules have a lower **Priority** (higher integers indicate lower priorities) so that the allow ICMP, custom, RDP, and SSH rules are considered first.

1. Click **Create**.
2. When the new network is ready, click **mynetwork**.

Create a VM instance in the  region. Selecting a region and zone determines the subnet and assigns  the internal IP address from the subnet's IP address range.

```sh
gcloud compute networks create mynetwork --project=qwiklabs-gcp-01-a2c8eac5980c --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional && gcloud compute firewall-rules create mynetwork-allow-custom --project=qwiklabs-gcp-01-a2c8eac5980c --network=projects/qwiklabs-gcp-01-a2c8eac5980c/global/networks/mynetwork --description=Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all && gcloud compute firewall-rules create mynetwork-allow-icmp --project=qwiklabs-gcp-01-a2c8eac5980c --network=projects/qwiklabs-gcp-01-a2c8eac5980c/global/networks/mynetwork --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp && gcloud compute firewall-rules create mynetwork-allow-rdp --project=qwiklabs-gcp-01-a2c8eac5980c --network=projects/qwiklabs-gcp-01-a2c8eac5980c/global/networks/mynetwork --description=Allows\ RDP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 3389. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389 && gcloud compute firewall-rules create mynetwork-allow-ssh --project=qwiklabs-gcp-01-a2c8eac5980c --network=projects/qwiklabs-gcp-01-a2c8eac5980c/global/networks/mynetwork --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
```



```sh
gcloud compute instances create mynet-us-vm --project=qwiklabs-gcp-01-a2c8eac5980c --zone=us-west3-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=834483548885-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230411,mode=rw,size=10,type=projects/qwiklabs-gcp-01-a2c8eac5980c/zones/us-west3-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```



```sh
gcloud compute instances create mynet-eu-vm --project=qwiklabs-gcp-01-a2c8eac5980c --zone=europe-west1-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=834483548885-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-eu-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230411,mode=rw,size=10,type=projects/qwiklabs-gcp-01-a2c8eac5980c/zones/us-west3-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```



### **Verify connectivity for the VM instances**

The firewall rules that you created with **mynetwork** allow ingress SSH and ICMP traffic from within **mynetwork** (internal IP) and outside that network (external IP).

```sh
Linux mynet-us-vm 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-b9934b2a0bae'.
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 10.132.0.2
PING 10.132.0.2 (10.132.0.2) 56(84) bytes of data.
64 bytes from 10.132.0.2: icmp_seq=1 ttl=64 time=124 ms
64 bytes from 10.132.0.2: icmp_seq=2 ttl=64 time=120 ms
64 bytes from 10.132.0.2: icmp_seq=3 ttl=64 time=120 ms

--- 10.132.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 119.565/121.166/124.291/2.209 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 34.77.44.243
PING 34.77.44.243 (34.77.44.243) 56(84) bytes of data.
64 bytes from 34.77.44.243: icmp_seq=1 ttl=50 time=122 ms
64 bytes from 34.77.44.243: icmp_seq=2 ttl=50 time=120 ms
64 bytes from 34.77.44.243: icmp_seq=3 ttl=50 time=121 ms

--- 34.77.44.243 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 120.189/120.871/121.817/0.690 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:0a:b4:00:02 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.180.0.2/32 brd 10.180.0.2 scope global dynamic ens4
       valid_lft 3380sec preferred_lft 3380sec
    inet6 fe80::4001:aff:feb4:2/64 scope link 
       valid_lft forever preferred_lft forever
student-01-b9934b2a0bae@mynet-us-vm:~$ 
```



### **Convert the network to a custom mode network**

The auto mode network worked great so far, but you have been asked to convert it to a custom mode network so that new subnets aren't  automatically created as new regions become available. This could result in overlap with IP addresses used by manually created subnets or static routes, or could interfere with your overall network planning.

## 3.Create custom mode networks

You have been tasked to create two additional custom networks, **managementnet** and **privatenet**, along with firewall rules to allow **SSH**, **ICMP**, and **RDP** ingress traffic and VM instances as shown in this example diagram (with the exception of vm-appliance):



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-2.png)



Note that the IP CIDR ranges of these networks do not overlap. This  allows you to set up mechanisms such as VPC peering between the  networks. If you specify IP CIDR ranges that are different from your  on-premises network, you could even configure hybrid connectivity using  VPN or Cloud Interconnect.

```sh
gcloud compute networks create managementnet --project=qwiklabs-gcp-01-a2c8eac5980c --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional && gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-01-a2c8eac5980c --range=10.240.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-west3
```

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-a2c8eac5980c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute networks create privatenet --subnet-mode=custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-a2c8eac5980c/global/networks/privatenet].
NAME: privatenet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network privatenet --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network privatenet --allow tcp:22,tcp:3389,icmp

student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-a2c8eac5980c/regions/europe-west1/subnetworks/privatesubnet-eu].
NAME: privatesubnet-eu
REGION: europe-west1
NETWORK: privatenet
RANGE: 172.20.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-west3 --range=172.16.0.0/24
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-a2c8eac5980c/regions/us-west3/subnetworks/privatesubnet-us].
NAME: privatesubnet-us
REGION: us-west3
NETWORK: privatenet
RANGE: 172.16.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute networks list
NAME: managementnet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

NAME: mynetwork
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

NAME: privatenet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute networks subnets list --sort-by=NETWORK
NAME: managementsubnet-us
REGION: us-west3
NETWORK: managementnet
RANGE: 10.240.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-central1
NETWORK: mynetwork
RANGE: 10.128.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-west1
NETWORK: mynetwork
RANGE: 10.132.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-west1
NETWORK: mynetwork
RANGE: 10.138.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: asia-east1
NETWORK: mynetwork
RANGE: 10.140.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-east1
NETWORK: mynetwork
RANGE: 10.142.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: asia-northeast1
NETWORK: mynetwork
RANGE: 10.146.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: asia-southeast1
NETWORK: mynetwork
RANGE: 10.148.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-east4
NETWORK: mynetwork
RANGE: 10.150.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: australia-southeast1
NETWORK: mynetwork
RANGE: 10.152.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-west2
NETWORK: mynetwork
RANGE: 10.154.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-west3
NETWORK: mynetwork
RANGE: 10.156.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: asia-south1
NETWORK: mynetwork
RANGE: 10.160.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-west4
NETWORK: mynetwork
RANGE: 10.164.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-north1
NETWORK: mynetwork
RANGE: 10.166.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-west2
NETWORK: mynetwork
RANGE: 10.168.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-west3
NETWORK: mynetwork
RANGE: 10.180.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-west4
NETWORK: mynetwork
RANGE: 10.182.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-central2
NETWORK: mynetwork
RANGE: 10.186.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: southamerica-west1
NETWORK: mynetwork
RANGE: 10.194.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-east5
NETWORK: mynetwork
RANGE: 10.202.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: us-south1
NETWORK: mynetwork
RANGE: 10.206.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: me-west1
NETWORK: mynetwork
RANGE: 10.208.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: europe-west12
NETWORK: mynetwork
RANGE: 10.210.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: mynetwork
REGION: me-central1
NETWORK: mynetwork
RANGE: 10.212.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: privatesubnet-eu
REGION: europe-west1
NETWORK: privatenet
RANGE: 172.20.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: privatesubnet-us
REGION: us-west3
NETWORK: privatenet
RANGE: 172.16.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

```sh
gcloud compute --project=qwiklabs-gcp-01-a2c8eac5980c firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-a2c8eac5980c/global/firewalls/privatenet-allow-icmp-ssh-rdp].
Creating firewall...done.
NAME: privatenet-allow-icmp-ssh-rdp
NETWORK: privatenet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:22,tcp:3389
DENY:
DISABLED: False
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute firewall-rules list --sort-by=NETWORK
NAME: managementnet-allow-icmp-ssh-rdp
NETWORK: managementnet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:3389,icmp
DENY:
DISABLED: False

NAME: mynetwork-allow-custom
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: all
DENY:
DISABLED: False

NAME: mynetwork-allow-icmp
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY:
DISABLED: False

NAME: mynetwork-allow-rdp
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY:
DISABLED: False

NAME: mynetwork-allow-ssh
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY:
DISABLED: False

NAME: privatenet-allow-icmp-ssh-rdp
NETWORK: privatenet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:22,tcp:3389
DENY:
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

The firewall rules for **mynetwork** network have been created for you. You can define multiple protocols and ports in one firewall rule (**privatenet** and **managementnet**) or spread them across multiple rules (**default** and **mynetwork**).

```sh
gcloud compute instances create managementnet-us-vm --project=qwiklabs-gcp-01-a2c8eac5980c --zone=us-west3-c --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=managementsubnet-us --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=834483548885-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230411,mode=rw,size=10,type=projects/qwiklabs-gcp-01-a2c8eac5980c/zones/us-west3-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```

```sh
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute instances create privatenet-us-vm --zone=us-west3-c --machine-type=e2-micro --subnet=privatesubnet-us --image-family=debian-11 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=privatenet-us-vm
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-a2c8eac5980c/zones/us-west3-c/instances/privatenet-us-vm].
NAME: privatenet-us-vm
ZONE: us-west3-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE:
INTERNAL_IP: 172.16.0.2
EXTERNAL_IP: 34.106.2.168
STATUS: RUNNING
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```

```sh
tudent_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$ gcloud compute instances list --sort-by=ZONE
NAME: mynet-eu-vm
ZONE: europe-west1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE:
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 34.77.44.243
STATUS: RUNNING

NAME: managementnet-us-vm
ZONE: us-west3-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE:
INTERNAL_IP: 10.240.0.2
EXTERNAL_IP: 34.106.115.233
STATUS: RUNNING

NAME: mynet-us-vm
ZONE: us-west3-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE:
INTERNAL_IP: 10.180.0.2
EXTERNAL_IP: 34.106.50.185
STATUS: RUNNING

NAME: privatenet-us-vm
ZONE: us-west3-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE:
INTERNAL_IP: 172.16.0.2
EXTERNAL_IP: 34.106.2.168
STATUS: RUNNING
student_01_b9934b2a0bae@cloudshell:~ (qwiklabs-gcp-01-a2c8eac5980c)$
```



## 4. Explore the connectivity across networks

Explore the connectivity between the VM instances. Specifically,  determine the effect of having VM instances in the same zone versus  having instances in the same VPC network.

### **Ping the external IP addresses**

Ping the external IP addresses of the VM instances to determine whether you can reach the instances from the public internet

```sh
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 34.106.115.233
PING 34.106.115.233 (34.106.115.233) 56(84) bytes of data.
64 bytes from 34.106.115.233: icmp_seq=1 ttl=61 time=1.82 ms
64 bytes from 34.106.115.233: icmp_seq=2 ttl=61 time=0.581 ms
64 bytes from 34.106.115.233: icmp_seq=3 ttl=61 time=0.633 ms

--- 34.106.115.233 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 0.581/1.010/1.818/0.571 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 34.106.2.168
PING 34.106.2.168 (34.106.2.168) 56(84) bytes of data.
64 bytes from 34.106.2.168: icmp_seq=1 ttl=61 time=1.71 ms
64 bytes from 34.106.2.168: icmp_seq=2 ttl=61 time=0.477 ms
64 bytes from 34.106.2.168: icmp_seq=3 ttl=61 time=0.651 ms

--- 34.106.2.168 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2015ms
rtt min/avg/max/mdev = 0.477/0.945/1.707/0.543 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 34.77.44.243
PING 34.77.44.243 (34.77.44.243) 56(84) bytes of data.
64 bytes from 34.77.44.243: icmp_seq=1 ttl=51 time=123 ms
64 bytes from 34.77.44.243: icmp_seq=2 ttl=51 time=122 ms
64 bytes from 34.77.44.243: icmp_seq=3 ttl=51 time=122 ms

--- 34.77.44.243 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 121.654/122.036/122.793/0.535 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ 
```

### **Ping the internal IP addresses**

Ping the internal IP addresses of the VM instances to determine whether you can reach the instances from within a VPC network.

```sh
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 10.132.0.2
PING 10.132.0.2 (10.132.0.2) 56(84) bytes of data.
64 bytes from 10.132.0.2: icmp_seq=1 ttl=64 time=123 ms
64 bytes from 10.132.0.2: icmp_seq=2 ttl=64 time=122 ms
64 bytes from 10.132.0.2: icmp_seq=3 ttl=64 time=122 ms

--- 10.132.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 121.985/122.427/123.303/0.619 ms
student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 10.240.0.2
PING 10.240.0.2 (10.240.0.2) 56(84) bytes of data.

--- 10.240.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2036ms

student-01-b9934b2a0bae@mynet-us-vm:~$ ping -c 3 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.

--- 172.16.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2040ms

student-01-b9934b2a0bae@mynet-us-vm:~$ 
```

> This should not work either, as indicated by a 100% packet loss! You cannot ping the internal IP address of <b>managementnet-us-vm</b> and <b>privatenet-us-vm</b> because they are in separate VPC networks from the source of the ping (<b>mynet-us-vm</b>), even though they are all in the same zone.





![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-12.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-17.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-20.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-23.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-24.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-25.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-26.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-27.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-28.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vpc-29.png)

## 5. Review

In this lab, you explored the default network and determined that you cannot create VM instances without a VPC network. Thus, you created a  new auto mode VPC network with subnets, routes, firewall rules, and two  VM instances and tested the connectivity for the VM instances. Because  auto mode networks aren't recommended for production, you converted the  auto mode network to a custom mode network.

Next, you created two more custom mode VPC networks with firewall  rules and VM instances using the Cloud Console and the gcloud command  line. Then you tested the connectivity across VPC networks, which worked when pinging external IP addresses but not when pinging internal IP  addresses.

VPC networks are by default isolated private networking domains.  Therefore, no internal IP address communication is allowed between  networks, unless you set up mechanisms such as VPC peering or VPN.



