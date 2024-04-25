---
layout: single
title:  "VPC Network Peering"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# VPC Network Peering

Google Cloud Virtual Private Cloud (VPC) Network Peering allows private  connectivity across two VPC networks regardless of whether or not they  belong to the same project or the same organization.

- Create a custom network in two projects
- Set up a VPC network peering session

Within the same organization node, a network could be hosting  services that need to be accessible from other VPC networks in the same  or different projects.

Alternatively, one organization may want to access services a third-party service is offering.

Project names are unique across all of Google Cloud, so you do not  need to specify the organization when setting up peering. Google Cloud  knows the organization based on the project name.

In this lab, you have been provisioned 2 projects, the first project is *project-A* and second is *project-B*.

#### project-A:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-76c4fb5fb450.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud config set project qwiklabs-gcp-01-76c4fb5fb450
Updated property [core/project].
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud compute networks create network-a --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-76c4fb5fb450/global/networks/network-a].
NAME: network-a
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network network-a --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network network-a --allow tcp:22,tcp:3389,icmp

student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud compute networks subnets create network-a-subnet --network network-a \
    --range 10.0.0.0/16 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-76c4fb5fb450/regions/us-west1/subnetworks/network-a-subnet].
NAME: network-a-subnet
REGION: us-west1
NETWORK: network-a
RANGE: 10.0.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud compute instances create vm-a --zone us-west1-b --network network-a --subnet network-a-subnet --machine-type e2-small
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-76c4fb5fb450/zones/us-west1-b/instances/vm-a].
NAME: vm-a
ZONE: us-west1-b
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.0.0.2
EXTERNAL_IP: 34.83.43.24
STATUS: RUNNING
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud compute firewall-rules create network-a-fw --network network-a --allow tcp:22,icmp
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-76c4fb5fb450/global/firewalls/network-a-fw].                                  
Creating firewall...done.                                                                                                                                                          
NAME: network-a-fw
NETWORK: network-a
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,icmp
DENY: 
DISABLED: False
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ 
```

#### project-B:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-d3754ada080a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud config set project qwiklabs-gcp-02-d3754ada080a
Updated property [core/project].
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud compute networks create network-b --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-d3754ada080a/global/networks/network-b].
NAME: network-b
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network network-b --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network network-b --allow tcp:22,tcp:3389,icmp

student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud compute networks subnets create network-b-subnet --network network-b \
    --range 10.8.0.0/16 --region us-east4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-d3754ada080a/regions/us-east4/subnetworks/network-b-subnet].
NAME: network-b-subnet
REGION: us-east4
NETWORK: network-b
RANGE: 10.8.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud compute instances create vm-b --zone us-east4-c --network network-b --subnet network-b-subnet --machine-type e2-small
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-d3754ada080a/zones/us-east4-c/instances/vm-b].
NAME: vm-b
ZONE: us-east4-c
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.8.0.2
EXTERNAL_IP: 35.245.182.141
STATUS: RUNNING
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud compute firewall-rules create network-b-fw --network network-b --allow tcp:22,icmp
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-d3754ada080a/global/firewalls/network-b-fw].                                  
Creating firewall...done.                                                                                                                                                          
NAME: network-b-fw
NETWORK: network-b
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,icmp
DENY: 
DISABLED: False
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ 
```

**project-A**

Go to the `VPC Network Peering` in the Cloud Console by navigating to the Networking section and clicking **VPC Network** > **VPC network peering** in the left menu. Once you're there:

1. Click **Create connection**.
2. Click **Continue**.
3. Type "peer-ab" as the **Name** for this side of the connection.
4. Under **Your VPC network**, select the network you want to peer (network-a).
5. Set the **Peered VPC network** radio buttons to **In another project**.
6. Paste in the **Project ID** of the second project.
7. Type in the **VPC network name** of the other network (network-b).
8. Click **Create**.

At this point, the peering state remains INACTIVE because there is no matching configuration in network-b in project-B. You should see the  Status message, `Waiting for peer network to connect`.

**project-B**

1. Click **Create connection**.
2. Click **Continue**.
3. Type "peer-ba" as the **Name** for this side of the connection.
4. Under **Your VPC network**, select the network you want to peer (network-b).
5. Set the **Peering VPC network** radio buttons to **In another project**, unless you wish to peer within the same project.
6. Specify the **Project ID** of the first project.
7. Specify **VPC network name** of the other network (network-a).
8. Click **Create**.

In the VPC network peering, you should now see `peer-ba` listed in the property list.

VPC Network Peering becomes ACTIVE and routes are exchanged As soon as the peering moves to an ACTIVE state, traffic flows are set up:

- Between VM instances in the peered networks: Full mesh connectivity.
- From VM instances in one network to Internal Load Balancing endpoints in the peered network.

The routes to peered network CIDR prefixes are now visible across the  VPC network peers. These routes are implicit routes generated for active peerings. They don't have corresponding route resources. The following  command lists routes for all VPC networks for project-A.

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ gcloud compute routes list --project qwiklabs-gcp-01-76c4fb5fb450
NAME: default-route-0839676dcbef999d
NETWORK: default
DEST_RANGE: 10.218.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-0d6f1565beabac1d
NETWORK: default
DEST_RANGE: 10.146.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-22ed38d316bd0d90
NETWORK: default
DEST_RANGE: 10.168.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-25e43c1e90f463f5
NETWORK: default
DEST_RANGE: 10.210.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-2fd36911c82e8178
NETWORK: default
DEST_RANGE: 0.0.0.0/0
NEXT_HOP: default-internet-gateway
PRIORITY: 1000

NAME: default-route-315091028b1fd043
NETWORK: default
DEST_RANGE: 10.140.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-3818a234521afdbd
NETWORK: default
DEST_RANGE: 10.128.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-3a88fde491d909fa
NETWORK: default
DEST_RANGE: 10.142.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-407b4327a6d525f0
NETWORK: default
DEST_RANGE: 10.180.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-40c63482bbef6565
NETWORK: default
DEST_RANGE: 10.220.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-4cbfad6a4779570f
NETWORK: default
DEST_RANGE: 10.152.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-4e4e6e3771f28591
NETWORK: default
DEST_RANGE: 10.208.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-5638b69a82700e05
NETWORK: default
DEST_RANGE: 10.154.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-5ea3530ce60c1ae7
NETWORK: network-a
DEST_RANGE: 10.0.0.0/16
NEXT_HOP: network-a
PRIORITY: 0

NAME: default-route-6141597d2144d712
NETWORK: default
DEST_RANGE: 10.150.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-6cc998688a5d27db
NETWORK: default
DEST_RANGE: 10.214.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-76031fc06943d283
NETWORK: default
DEST_RANGE: 10.138.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-773a500cf90c47c5
NETWORK: default
DEST_RANGE: 10.156.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-86406da7c121aeb6
NETWORK: default
DEST_RANGE: 10.166.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-9a78451e6bfe5c11
NETWORK: default
DEST_RANGE: 10.182.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-a21c7f45ae0e7b1a
NETWORK: default
DEST_RANGE: 10.216.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-a2a0e0a94cc25244
NETWORK: default
DEST_RANGE: 10.196.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-a415755e130c9f2b
NETWORK: default
DEST_RANGE: 10.212.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-a48eb441f2a98724
NETWORK: default
DEST_RANGE: 10.194.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-b1e45cb8bff87981
NETWORK: default
DEST_RANGE: 10.132.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-b70a500c517128fb
NETWORK: default
DEST_RANGE: 10.186.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-bbfcee1c49a84690
NETWORK: default
DEST_RANGE: 10.206.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-d0478897b8c2db7e
NETWORK: default
DEST_RANGE: 10.160.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-da3fa6d1834a6884
NETWORK: network-a
DEST_RANGE: 0.0.0.0/0
NEXT_HOP: default-internet-gateway
PRIORITY: 1000

NAME: default-route-de8ff4d5a3deecb6
NETWORK: default
DEST_RANGE: 10.164.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-e7586e275c44e34f
NETWORK: default
DEST_RANGE: 10.202.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-f5e23142cc4d26f5
NETWORK: default
DEST_RANGE: 10.148.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: peering-route-7ed182eb6d8594bb
NETWORK: network-a
DEST_RANGE: 10.8.0.0/16
NEXT_HOP: peer-ab
PRIORITY: 0
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-01-76c4fb5fb450)$ 
```

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$  gcloud compute routes list
NAME: default-route-002440725a27d687
NETWORK: default
DEST_RANGE: 10.128.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-0227d2b82ccd3042
NETWORK: default
DEST_RANGE: 10.214.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-243cef98666777c6
NETWORK: default
DEST_RANGE: 10.196.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-2e045e34d71b6b27
NETWORK: default
DEST_RANGE: 10.220.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-32a1cf936b0dfd5c
NETWORK: default
DEST_RANGE: 10.194.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-32decde20a8c4745
NETWORK: default
DEST_RANGE: 10.146.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-336066dc443b26dc
NETWORK: default
DEST_RANGE: 0.0.0.0/0
NEXT_HOP: default-internet-gateway
PRIORITY: 1000

NAME: default-route-3760ab57ddc72503
NETWORK: default
DEST_RANGE: 10.148.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-3e0051affc9232ce
NETWORK: default
DEST_RANGE: 10.164.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-4a3db4296c82d12c
NETWORK: default
DEST_RANGE: 10.150.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-5221a2987fd16bcc
NETWORK: default
DEST_RANGE: 10.182.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-66d17f6e59dd98e3
NETWORK: default
DEST_RANGE: 10.180.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-69baead11c6e82c6
NETWORK: default
DEST_RANGE: 10.156.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-714b4953f3a935c9
NETWORK: default
DEST_RANGE: 10.160.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-7223ada7913866c2
NETWORK: default
DEST_RANGE: 10.152.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-9acda2d4c068b79d
NETWORK: default
DEST_RANGE: 10.140.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-9b6f8d60b643db44
NETWORK: default
DEST_RANGE: 10.216.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-a90116b5e7ce1a22
NETWORK: default
DEST_RANGE: 10.168.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-abea4ceccda7b242
NETWORK: default
DEST_RANGE: 10.212.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-bedfe06f83144217
NETWORK: default
DEST_RANGE: 10.186.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-cc9731b494d21722
NETWORK: default
DEST_RANGE: 10.218.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-cecca5247fc85e04
NETWORK: default
DEST_RANGE: 10.210.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-d07c8af9c5d74db2
NETWORK: default
DEST_RANGE: 10.208.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-d9e7b25923034cf5
NETWORK: default
DEST_RANGE: 10.142.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-e76950183f62ba5c
NETWORK: network-b
DEST_RANGE: 10.8.0.0/16
NEXT_HOP: network-b
PRIORITY: 0

NAME: default-route-e7e4ddf6c27cc292
NETWORK: default
DEST_RANGE: 10.206.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-ea102881616661f0
NETWORK: default
DEST_RANGE: 10.202.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-f29657bb2cedda91
NETWORK: default
DEST_RANGE: 10.138.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-f546c386ca0214fd
NETWORK: default
DEST_RANGE: 10.166.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-f6ae6b3cd8e2b31a
NETWORK: default
DEST_RANGE: 10.154.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: default-route-f7eaa771446c27e4
NETWORK: network-b
DEST_RANGE: 0.0.0.0/0
NEXT_HOP: default-internet-gateway
PRIORITY: 1000

NAME: default-route-fcb6595a5aef4ad5
NETWORK: default
DEST_RANGE: 10.132.0.0/20
NEXT_HOP: default
PRIORITY: 0

NAME: peering-route-ab3d2457cf9658dc
NETWORK: network-b
DEST_RANGE: 10.0.0.0/16
NEXT_HOP: peer-ba
PRIORITY: 0
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ 
```



```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-02-d3754ada080a)$ gcloud compute ssh --zone "us-east4-c" "vm-b" --project "qwiklabs-gcp-02-d3754ada080a"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_9b51dda9fd9c/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Passphrases do not match.  Try again.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_9b51dda9fd9c/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_9b51dda9fd9c/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:jyldszdx8qKLH3IfsEA7sk8uiL5T42+NUIGIpvZYubo student_01_9b51dda9fd9c@cs-22481000744-default
The key's randomart image is:
+---[RSA 3072]----+
| . . .           |
|... . .          |
|o   .  ..        |
|.. o  .. .       |
|. + ... S + o .  |
| . o+  + B = =   |
|  .+ +oo* * = .  |
| .o o +=.+ = +   |
| E+o o..+.+..    |
+----[SHA256]-----+
Warning: Permanently added 'compute.195608168263601807' (ED25519) to the list of known hosts.
Linux vm-b 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-9b51dda9fd9c'.
student-01-9b51dda9fd9c@vm-b:~$ 
```

```sh
student-01-9b51dda9fd9c@vm-b:~$ ping 10.0.0.2 -c 5
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=56.7 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=54.7 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=54.6 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=54.7 ms
64 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=54.8 ms

--- 10.0.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 54.584/55.089/56.656/0.786 ms
student-01-9b51dda9fd9c@vm-b:~$ 
```

![gcp-vpc-peering]({{ site.url }}{{ site.baseurl }}/assets/images/vpc-peering.png)
