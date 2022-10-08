---

layout: single
title:  "GCP—Implement Private Google Access and Cloud NAT"
date:   2022-10-07 05:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcp.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Implement Private Google Access and Cloud NAT
In this lab, we implement Private Google Access and Cloud NAT for a VM instance that doesn't have an external IP address. Then, we verify access to public IP addresses of Google APIs and services and other connections to the internet.

VM instances without external IP addresses are isolated from external networks. Using Cloud NAT, these instances can access the internet for updates and patches, and in some cases, for bootstrapping. As a managed service, Cloud NAT provides high availability without user management and intervention.

Configure a VM instance that doesn't have an external IP address
Connect to a VM instance using an Identity-Aware Proxy (IAP) tunnel
Enable Private Google Access on a subnet
Configure a Cloud NAT gateway
Verify access to public IP addresses of Google APIs and services and other connections to the internet

## Task 1. Create the VM instance
Create a VPC network with some firewall rules and a VM instance that has no external IP address, and connect to the instance using an IAP tunnel.

Create a VPC network and firewall rules
First, create a VPC network for the VM instance and a firewall rule to allow SSH access.

```sh
gcloud compute networks create privatenet --project=qwiklabs-gcp-01-9d92c04e98fa --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
```

```sh
gcloud compute networks subnets create privatenet-us --project=qwiklabs-gcp-01-9d92c04e98fa --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=privatenet --region=us-central1
```

```sh
gcloud compute --project=qwiklabs-gcp-01-9d92c04e98fa firewall-rules create privatenet-allow-ssh --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20
```
In order to connect to your private instance using SSH, you need to open an appropriate port on the firewall. IAP connections come from a specific set of IP addresses (35.235.240.0/20). Therefore, you can limit the rule to this CIDR range.

Create the VM instance with no public IP address

```sh
gcloud compute instances create vm-internal --project=qwiklabs-gcp-01-9d92c04e98fa --zone=us-central1-c --machine-type=n1-standard-1 --network-interface=subnet=privatenet-us,no-address --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=44827933619-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-internal,image=projects/debian-cloud/global/images/debian-10-buster-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-01-9d92c04e98fa/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

>The default setting for a VM instance is to have an ephemeral external IP address. This behavior can be changed with a policy constraint at the organization or project level. 


![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-103.png)


### SSH to vm-internal to test the IAP tunnel



```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-9d92c04e98fa.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$ gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_e3c2b9ab4419/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student_01_e3c2b9ab4419/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_e3c2b9ab4419/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:32tpogjwdzo/tMUdxvqX6P1FyJWucp4xmNFQ0Zqy05o student_01_e3c2b9ab4419@cs-25715951773-default
The key's randomart image is:
+---[RSA 3072]----+
|             oo  |
|            .  ..|
|           o  o..|
|           .*+.o |
|  .     S. =+oo..|
|   o    ..++=... |
|    o ...o.==B ..|
|     o.+o .EXo* .|
|      oooo =o+...|
+----[SHA256]-----+

WARNING:

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.6865677423966281884' (ECDSA) to the list of known hosts.
Linux vm-internal 4.19.0-21-cloud-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-e3c2b9ab4419'.
student-01-e3c2b9ab4419@vm-internal:~$
student-01-e3c2b9ab4419@vm-internal:~$
```





```sh
student-01-e3c2b9ab4419@vm-internal:~$ ping -c 2 www.google.com
PING www.google.com (172.217.212.99) 56(84) bytes of data.

--- www.google.com ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 26ms

student-01-e3c2b9ab4419@vm-internal:~$
```



This should not work because **vm-internal** has no external IP address!

When instances do not have external IP addresses, they can only be reached by other instances on the network via a managed VPN gateway or via a Cloud IAP tunnel. Cloud IAP enables context-aware access to VMs via SSH and RDP without bastion hosts.

## Task 2. Enable Private Google Access

VM instances that have no external IP addresses can use Private Google Access to reach external IP addresses of Google APIs and services. By default, Private Google Access is disabled on a VPC network.



### Create a Cloud Storage bucket

Create a Cloud Storage bucket to test access to Google APIs and services.

### Copy an image file into your bucket

Copy an image from a public Cloud Storage bucket to your own bucket.

```sh
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$ gsutil cp gs://cloud-training/gcpnet/private/access.svg gs://$MY_BUCKET
Copying gs://cloud-training/gcpnet/private/access.svg [Content-Type=image/svg+xml]...
- [1 files][ 24.8 KiB/ 24.8 KiB]
Operation completed over 1 objects/24.8 KiB.
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$
```



```sh
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$ gsutil cp gs://$MY_BUCKET/*.svg .
Copying gs://gaddepradeep/access.svg...
/ [1 files][ 24.8 KiB/ 24.8 KiB]
Operation completed over 1 objects/24.8 KiB.
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$
```



```sh
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$ gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
WARNING:

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Linux vm-internal 4.19.0-21-cloud-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Oct  7 17:17:00 2022 from 35.235.244.32
student-01-e3c2b9ab4419@vm-internal:~$ export MY_BUCKET=gaddepradeep
student-01-e3c2b9ab4419@vm-internal:~$ echo $MY_BUCKET
gaddepradeep
student-01-e3c2b9ab4419@vm-internal:~$
```

```sh
student-01-e3c2b9ab4419@vm-internal:~$ gsutil cp gs://$MY_BUCKET/*.svg .
^Z
[1]+  Stopped                 gsutil cp gs://$MY_BUCKET/*.svg .
student-01-e3c2b9ab4419@vm-internal:~$
```



This should not work: **vm-internal** can only send traffic within the VPC network because Private Google Access is disabled (by default).

Press **Ctrl+Z** to stop the request.


![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-104.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-105.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-106.png)

### Enable Private Google Access

Private Google Access is enabled at the subnet level. When it is enabled, instances in the subnet that only have private IP addresses can send traffic to Google APIs and services through the default route (0.0.0.0/0) with a next hop to the default internet gateway.

In **Cloud Shell** for **vm-internal**,  try to copy the image to **vm-internal**, 
```sh
student-01-e3c2b9ab4419@vm-internal:~$ gsutil cp gs://$MY_BUCKET/*.svg .
Copying gs://gaddepradeep/access.svg...
/ [1 files][ 24.8 KiB/ 24.8 KiB]
Operation completed over 1 objects/24.8 KiB.
student-01-e3c2b9ab4419@vm-internal:~$
```
This should work because vm-internal's subnet has Private Google Access enabled!
```sh
student-01-e3c2b9ab4419@vm-internal:~$ exit
logout
There are stopped jobs.
student-01-e3c2b9ab4419@vm-internal:~$
```

## Task 3. Configure a Cloud NAT gateway
Although vm-internal can now access certain Google APIs and services without an external IP address, the instance cannot access the internet for updates and patches. Configure a Cloud NAT gateway, which allows vm-internal to reach the internet.

```sh
student-01-e3c2b9ab4419@vm-internal:~$ sudo apt-get update
Get:1 http://packages.cloud.google.com/apt cloud-sdk-buster InRelease [6786 B]
Get:2 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-buster InRelease [5553 B]                                 
Get:3 http://packages.cloud.google.com/apt google-compute-engine-buster-stable InRelease [5526 B]                                          
Get:4 http://packages.cloud.google.com/apt cloud-sdk-buster/main amd64 Packages [333 kB]                             
Get:5 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-buster/main amd64 Packages [387 B]                                
0% [Connecting to debian.map.fastlydns.net (146.75.78.132)] [Connecting to debian.map.fastlydns.net (199.232.30.132)]^C                                                            
student-01-e3c2b9ab4419@vm-internal:~$
```

Configure a Cloud NAT gateway
Cloud NAT is a regional resource. You can configure it to allow traffic from all ranges of all subnets in a region, from specific subnets in the region only, or from specific primary and secondary CIDR ranges only.



The NAT mapping section allows you to choose the subnets to map to the NAT gateway. You can also manually assign static IP addresses that should be used when performing NAT. Do not change the NAT mapping configuration in this lab.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-107.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-108.png)


Verify the Cloud NAT gateway
It may take up to 3 minutes for the NAT configuration to propagate to the VM, so wait at least a minute before trying to access the internet again.
```sh
student-01-e3c2b9ab4419@vm-internal:~$ sudo apt-get update
Hit:1 http://deb.debian.org/debian buster InRelease
Get:2 http://security.debian.org/debian-security buster/updates InRelease [34.8 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [56.6 kB]           
Get:4 http://packages.cloud.google.com/apt cloud-sdk-buster InRelease [6786 B]
Get:5 http://deb.debian.org/debian buster-backports InRelease [51.4 kB]
Get:6 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-buster InRelease [5553 B]
Get:7 http://security.debian.org/debian-security buster/updates/main Sources [271 kB]
Get:8 http://packages.cloud.google.com/apt google-compute-engine-buster-stable InRelease [5526 B]
Get:9 http://security.debian.org/debian-security buster/updates/main amd64 Packages [366 kB]
Get:10 http://security.debian.org/debian-security buster/updates/main Translation-en [198 kB]
Get:11 http://packages.cloud.google.com/apt cloud-sdk-buster/main amd64 Packages [333 kB]
Get:12 http://deb.debian.org/debian buster-backports/main Sources.diff/Index [27.8 kB]
Get:13 http://deb.debian.org/debian buster-backports/main amd64 Packages.diff/Index [27.8 kB]
Get:14 http://deb.debian.org/debian buster-backports/main Translation-en.diff/Index [27.8 kB]
Get:15 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-buster/main amd64 Packages [387 B]
Get:16 http://deb.debian.org/debian buster-backports/main Sources 2022-09-22-1115.32.pdiff [68 B]
Get:17 http://deb.debian.org/debian buster-backports/main amd64 Packages 2022-09-22-1115.32.pdiff [85 B]
Get:16 http://deb.debian.org/debian buster-backports/main Sources 2022-09-22-1115.32.pdiff [68 B]
Get:17 http://deb.debian.org/debian buster-backports/main amd64 Packages 2022-09-22-1115.32.pdiff [85 B]
Get:18 http://deb.debian.org/debian buster-backports/main Translation-en 2022-09-22-1115.32.pdiff [80 B]
Get:18 http://deb.debian.org/debian buster-backports/main Translation-en 2022-09-22-1115.32.pdiff [80 B]
Fetched 1079 kB in 1s (910 kB/s)
Reading package lists... Done
student-01-e3c2b9ab4419@vm-internal:~$
```
This should work because vm-internal is using the NAT gateway!

The Cloud NAT gateway implements outbound NAT, but not inbound NAT. In other words, hosts outside of your VPC network can only respond to connections initiated by your instances; they cannot initiate their own, new connections to your instances via NAT.

## Task 4. Configure and view logs with Cloud NAT Logging

Cloud NAT logging allows you to log NAT connections and errors. When Cloud NAT logging is enabled, one log entry can be generated for each of the following scenarios:

When a network connection using NAT is created.
When a packet is dropped because no port was available for NAT.
You can opt to log both kinds of events, or just one or the other. Created logs are sent to Cloud Logging.

Enabling logging
If logging is enabled, all collected logs are sent to Cloud Logging by default. You can filter these so that only certain logs are sent.

You can also specify these values when you create a NAT gateway or by editing one after it has been created. The following directions show how to enable logging for an existing NAT gateway.

```sh
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-01-9d92c04e98fa)$ gcloud compute ssh vm-internal --zone us-central1-c --tunnel-through-iap
WARNING:

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Linux vm-internal 4.19.0-21-cloud-amd64 #1 SMP Debian 4.19.249-2 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Oct  7 17:24:19 2022 from 35.235.244.34
student-01-e3c2b9ab4419@vm-internal:~$ sudo apt-get update
Hit:1 http://packages.cloud.google.com/apt cloud-sdk-buster InRelease
Hit:2 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-buster InRelease                                           
Hit:3 http://packages.cloud.google.com/apt google-compute-engine-buster-stable InRelease
Hit:4 http://deb.debian.org/debian buster InRelease
Hit:5 http://security.debian.org/debian-security buster/updates InRelease
Hit:6 http://deb.debian.org/debian buster-updates InRelease
Hit:7 http://deb.debian.org/debian buster-backports InRelease
Reading package lists... Done
student-01-e3c2b9ab4419@vm-internal:~$
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-109.png)
## Task 5. Review
You created vm-internal, an instance with no external IP address, and connected to it securely using an IAP tunnel. Then you enabled Private Google Access, configured a NAT gateway, and verified that vm-internal can access Google APIs and services and other public IP addresses.

VM instances without external IP addresses are isolated from external networks. Using Cloud NAT, these instances can access the internet for updates and patches, and in some cases, for bootstrapping. As a managed service, Cloud NAT provides high availability without user management and intervention.

IAP uses your existing project roles and permissions when you connect to VM instances. By default, instance owners are the only users that have the IAP Secured Tunnel User role.