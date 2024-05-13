---

layout: single
title:  "Cloud Filestore: Qwik Start"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"
sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Cloud Filestore: Qwik Start

Cloud Filestore is a managed file storage service for applications that  require a file system interface and a shared filesystem for data.  Filestore gives users a simple, native experience for standing up  managed Network Attached Storage (NAS) with their Compute Engine and  Google Kubernetes Engine instances. The ability to fine-tune Filestore’s performance and capacity independently leads to predictably fast  performance for your file-based workloads.

- Create a Cloud Filestore instance.
- Mount the fileshare from that instance on a client VM instance.
- Create a file on the mounted fileshare.

## Cloud Filestore architecture

Cloud Filestore architectural choices that affect your Cloud Filestore instances:

### Permissions

A Cloud Filestore instance consists of a single NFS fileshare with fixed export settings and default Unix permissions.

### Networking

1. You must create a Cloud Filestore instance in the same Google Cloud project and [VPC network](https://cloud.google.com/compute/docs/vpc/) as any clients that connect to it. All [internal IP addresses](https://www.arin.net/knowledge/address_filters.html) in the selected VPC network can connect to the Cloud Filestore instance.
2. If you are using a VPC network other than the default network, you  might need to create firewall rules to enable communication with Cloud  Filestore instances.
3. You can't use a [legacy network](https://cloud.google.com/compute/docs/vpc/legacy) with Cloud Filestore instances.

### IP address range

Each Cloud Filestore instance must have an IP address range  associated with it. The IP address range must be from within the  internal IP address ranges (`10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`) and have a block size of 29. Examples of valid Cloud Filestore instance IP address ranges are `10.0.3.0/29` and `172.31.0.0/29`.

You can assign the IP address range if there's a specific one you  want to use, otherwise Cloud Filestore picks a random range to use from  within the internal IP address ranges. If the range is already in use,  the service tries again until it finds one that is free. If you assign  an IP address range, make sure it doesn't overlap with any existing  subnets in the VPC network that the Cloud Filestore instance uses, or  with the IP address ranges assigned to any other existing Cloud  Filestore instances in that network.

### Cloud Filestore network peering

The first time you create a Cloud Filestore instance, Cloud Filestore also creates a [peered network](https://cloud.google.com/vpc/docs/vpc-peering) to enable network connectivity between clients in your project and the  Cloud Filestore instance. The peered network has a machine-generated  name similar to r-1abc2d3e-45fg-6789-hf12-3456i78j9k1-0000000a-peer, and appears in the VPC Network Peering page.

Don't delete the peered network, because this will cause you to lose  connectivity with your Cloud Filestore instances. If you accidentally  delete the peered network, the easiest thing to do to recreate it is to  create another Cloud Filestore instance. Cloud Filestore will recognize  that there is no connectivity between your project and the new instance, and will re-create the peered network. you can delete the new Cloud  Filestore instance after that if you don't need it for anything else.

### Storage size units

Cloud Filestore defines 1 gigabyte (`GB`) as 1024^3 bytes, also known as a gibibyte (`GiB`). Cloud Filestore also defines 1 terabyte (`TB`) as 1024^4 bytes, also known as a tebibyte (`TiB`).



## Create a Compute Engine instance

1. Navigate to **VM Instances** by going to **Navigation Menu** > **Compute Engine** > **VM Instances** and click **+CREATE INSTANCE**.



## Create a Cloud Filestore instance

An instance is a fully managed network-attached storage system you can use with your Google        Compute Engine and Kubernetes Engine instances.       

1. Go to **Navigation menu** > **APIs and Services** > **Library**.
2. Search for `Cloud Filestore API` and click **Enable** if it is not already enabled.
3. Navigate to **Navigation Menu** > **Filestore**.

If you get an error message be sure you navigated to **Filestore** and not **Firestore**.

1. Click **+CREATE INSTANCE** at the top of the page.



## Mount the Cloud Filestore fileshare on a Compute Engine VM

1. Navigate  **Navigation Menu** > **Compute Engine** > **VM Instances**.
2. In the list of VM instances, click the **SSH** button for **nfs-client** to open a terminal window connected to that instance.
3. In SSH shell install NFS by running the following commands:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-d4dadc3eb474.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$ gcloud compute ssh --zone "us-east4-c" "nfs-client" --project "qwiklabs-gcp-02-d4dadc3eb474"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_aa8f3a74b714/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_aa8f3a74b714/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_aa8f3a74b714/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:gTShtc1MEyFvecZ/V3RSY/0Slmx3Yz9Obz6ahXaZCgw student_01_aa8f3a74b714@cs-334013342407-default
The key's randomart image is:
+---[RSA 3072]----+
|      *.=o   ..==|
|     + @ +    B=*|
|    . o O +  o.o*|
|       . + .  .o+|
|        SE  . +.+|
|          o  ..o=|
|           o o B |
|            o =..|
|             +. .|
+----[SHA256]-----+
Warning: Permanently added 'compute.8358127393296928845' (ED25519) to the list of known hosts.
Linux nfs-client 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-aa8f3a74b714'.
student-01-aa8f3a74b714@nfs-client:~$ 
```

```sh
student-01-aa8f3a74b714@nfs-client:~$ sudo apt-get -y update &&
sudo apt-get -y install nfs-common
Hit:1 https://deb.debian.org/debian bullseye InRelease
Get:2 https://deb.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Get:3 https://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
Get:4 https://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:5 https://packages.cloud.google.com/apt google-compute-engine-bullseye-stable InRelease [1321 B]
Get:6 https://packages.cloud.google.com/apt cloud-sdk-bullseye InRelease [1602 B]
Get:7 https://deb.debian.org/debian-security bullseye-security/main Sources [174 kB]
Get:8 https://deb.debian.org/debian-security bullseye-security/main amd64 Packages [273 kB]
Get:9 https://deb.debian.org/debian-security bullseye-security/main Translation-en [176 kB]
Get:10 https://packages.cloud.google.com/apt google-compute-engine-bullseye-stable/main all Packages [2830 B]
Get:11 https://packages.cloud.google.com/apt google-compute-engine-bullseye-stable/main amd64 Packages [3127 B]
Get:12 https://deb.debian.org/debian bullseye-backports/main Sources.diff/Index [63.3 kB]
Get:13 https://deb.debian.org/debian bullseye-backports/main amd64 Packages.diff/Index [63.3 kB]
Get:14 https://deb.debian.org/debian bullseye-backports/main Sources T-2024-05-03-1405.15-F-2024-04-24-1420.52.pdiff [3092 B]
Get:14 https://deb.debian.org/debian bullseye-backports/main Sources T-2024-05-03-1405.15-F-2024-04-24-1420.52.pdiff [3092 B]
Get:15 https://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2024-05-03-1405.15-F-2024-04-24-2006.45.pdiff [2690 B]
Get:15 https://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2024-05-03-1405.15-F-2024-04-24-2006.45.pdiff [2690 B]
Get:16 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 Packages [2993 kB]
Get:17 https://packages.cloud.google.com/apt cloud-sdk-bullseye/main all Packages [1453 kB]
Fetched 5352 kB in 1s (4937 kB/s)   
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap2 rpcbind
Suggested packages:
  open-iscsi watchdog
The following NEW packages will be installed:
  keyutils libnfsidmap2 nfs-common rpcbind
0 upgraded, 4 newly installed, 0 to remove and 8 not upgraded.
Need to get 369 kB of archives.
After this operation, 1215 kB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 rpcbind amd64 1.2.5-9 [51.4 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 keyutils amd64 1.6.1-2 [52.8 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 libnfsidmap2 amd64 0.25-6 [32.6 kB]
Get:4 https://deb.debian.org/debian bullseye/main amd64 nfs-common amd64 1:1.3.4-6 [232 kB]
Fetched 369 kB in 0s (6479 kB/s)
Selecting previously unselected package rpcbind.
(Reading database ... 64633 files and directories currently installed.)
Preparing to unpack .../rpcbind_1.2.5-9_amd64.deb ...
Unpacking rpcbind (1.2.5-9) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../keyutils_1.6.1-2_amd64.deb ...
Unpacking keyutils (1.6.1-2) ...
Selecting previously unselected package libnfsidmap2:amd64.
Preparing to unpack .../libnfsidmap2_0.25-6_amd64.deb ...
Unpacking libnfsidmap2:amd64 (0.25-6) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../nfs-common_1%3a1.3.4-6_amd64.deb ...
Unpacking nfs-common (1:1.3.4-6) ...
Setting up rpcbind (1.2.5-9) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /lib/systemd/system/rpcbind.socket.
Setting up keyutils (1.6.1-2) ...
Setting up libnfsidmap2:amd64 (0.25-6) ...
Setting up nfs-common (1:1.3.4-6) ...

Creating config file /etc/idmapd.conf with new version
Adding system user `statd' (UID 110) ...
Adding new user `statd' (UID 110) with group `nogroup' ...
Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
nfs-utils.service is a disabled or a static unit, not starting it.
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+deb11u8) ...
student-01-aa8f3a74b714@nfs-client:~$ 
```
Mount the fileshare by running the mount command and specifying the Cloud Filestore instance IP address and fileshare name:

```sh
student-01-aa8f3a74b714@nfs-client:~$ sudo mkdir /mnt/test
student-01-aa8f3a74b714@nfs-client:~$ sudo mount 10.218.49.210:/vol1 /mnt/test
student-01-aa8f3a74b714@nfs-client:~$ sudo chmod go+rw /mnt/test
student-01-aa8f3a74b714@nfs-client:~$ 
```

 NFS mount point 

 Used to mount this file share on a linux client VM. Run the mount  command with the following remote target on the VM's local directory. 

```sh
10.218.49.210:/vol1
```



## Create a file on the fileshare

1. In the terminal window that is connected to the nfs-client instance, run the following to create a file named testfile:

```sh
student-01-aa8f3a74b714@nfs-client:~$ echo 'This is a test' > /mnt/test/testfile
student-01-aa8f3a74b714@nfs-client:~$ ls /mnt/test
lost+found  testfile
student-01-aa8f3a74b71``4@nfs-client:~$ cat /mnt/test/testfile
This is a test
student-01-aa8f3a74b714@nfs-client:~$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$ gcloud filestore instances list
INSTANCE_NAME: nfs-server
LOCATION: us-east4-c
TIER: BASIC_HDD
CAPACITY_GB: 1024
FILE_SHARE_NAME: vol1
IP_ADDRESS: 10.218.49.210
STATE: READY
CREATE_TIME: 2024-05-13T01:40:18
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$ gcloud filestore instances describe nfs-server --location us-east4-c
createTime: '2024-05-13T01:40:18.517135316Z'
fileShares:
- capacityGb: '1024'
  name: vol1
name: projects/qwiklabs-gcp-02-d4dadc3eb474/locations/us-east4-c/instances/nfs-server
networks:
- connectMode: DIRECT_PEERING
  ipAddresses:
  - 10.218.49.210
  modes:
  - MODE_IPV4
  network: default
  reservedIpRange: 10.218.49.208/29
satisfiesPzi: true
state: READY
tier: BASIC_HDD
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$ gcloud compute networks peerings list
NAME: filestore-peer-919245376566
NETWORK: default
PEER_PROJECT: nf3432c9f7fc1618bp-tp
PEER_NETWORK: default-2309458234058
STACK_TYPE: IPV4_ONLY
PEER_MTU: 
IMPORT_CUSTOM_ROUTES: True
EXPORT_CUSTOM_ROUTES: True
STATE: ACTIVE
STATE_DETAILS: [2024-05-12T18:42:09.716-07:00]: Connected.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-d4dadc3eb474)$
```

