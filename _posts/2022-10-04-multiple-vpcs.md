---

layout: single
title:  "Working with multiple VPC Networks"
date:   2022-10-04 11:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcpne.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcpne.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Multiple VPC Networks

In this lab, you create several VPC networks and VM instances and test connectivity across networks. 

Create custom mode VPC networks with firewall rules

Create VM instances using Compute Engine

Explore the connectivity for VM instances across VPC networks

Create a VM instance with multiple network interfaces

```sh
gcloud compute networks create managementnet --project=qwiklabs-gcp-02-bffac160bdef --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
```

```sh
gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-02-bffac160bdef --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-central1
```


```sh
gcloud compute networks create privatenet --project=qwiklabs-gcp-02-bffac160bdef --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
```

```sh
gcloud compute networks subnets create privatesubnet-us --project=qwiklabs-gcp-02-bffac160bdef --range=172.16.0.0/24 --stack-type=IPV4_ONLY --network=privatenet --region=us-central1
```

```sh
gcloud compute networks subnets create privatesubnet-eu --project=qwiklabs-gcp-02-bffac160bdef --range=172.20.0.0/20 --stack-type=IPV4_ONLY --network=privatenet --region=europe-west3
```


```sh
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$ gcloud compute networks list
NAME: default
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

NAME: managementnet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

NAME: mynetwork
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:

NAME: privatenet
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE:
GATEWAY_IPV4:
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$
```

List all subnets

```sh
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$ gcloud compute networks subnets list --sort-by=NETWORK
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

NAME: managementsubnet-us
REGION: us-central1
NETWORK: managementnet
RANGE: 10.130.0.0/20
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

NAME: privatesubnet-us
REGION: us-central1
NETWORK: privatenet
RANGE: 172.16.0.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:

NAME: privatesubnet-eu
REGION: europe-west3
NETWORK: privatenet
RANGE: 172.20.0.0/20
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE:
INTERNAL_IPV6_PREFIX:
EXTERNAL_IPV6_PREFIX:
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$
```

Create a firewall rule for the management network

```sh
gcloud compute --project=qwiklabs-gcp-02-bffac160bdef firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

For the privatenet

```sh
gcloud compute --project=qwiklabs-gcp-02-bffac160bdef firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0
```

Verify firewall rules

```sh
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$ gcloud compute firewall-rules list
NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY:
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY:
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY:
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY:
DISABLED: False

NAME: managementnet-allow-icmp-ssh-rdp
NETWORK: managementnet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:3389,icmp
DENY:
DISABLED: False

NAME: mynetwork-allow-icmp
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp
DENY:
DISABLED: False

NAME: mynetwork-allow-rdp
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY:
DISABLED: False

NAME: mynetwork-allow-ssh
NETWORK: mynetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY:
DISABLED: False

NAME: privatenet-allow-icmp-ssh-rdp
NETWORK: privatenet
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:3389,icmp
DENY:
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$
```



Create a VM in the management network



```sh
gcloud compute instances create managementnet-us-vm --project=qwiklabs-gcp-02-bffac160bdef --zone=us-central1-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=managementsubnet-us --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=968172065011-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-bffac160bdef/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Create another VM in the privatenet

```sh
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$ gcloud compute instances create privatenet-us-vm --zone=us-central1-c --machine-type=n1-standard-1 --subnet=privatesubnet-us
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-bffac160bdef/zones/us-central1-c/instances/privatenet-us-vm].
NAME: privatenet-us-vm
ZONE: us-central1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 172.16.0.2
EXTERNAL_IP: 35.226.51.88
STATUS: RUNNING
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$
```

List all compute instances

```sh
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$ gcloud compute instances list --sort-by=ZONE
NAME: mynet-eu-vm
ZONE: europe-west1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 104.155.101.10
STATUS: RUNNING

NAME: managementnet-us-vm
ZONE: us-central1-c
MACHINE_TYPE: f1-micro
PREEMPTIBLE:
INTERNAL_IP: 10.130.0.2
EXTERNAL_IP: 34.136.43.5
STATUS: RUNNING

NAME: mynet-us-vm
ZONE: us-central1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.133.255.56
STATUS: RUNNING

NAME: privatenet-us-vm
ZONE: us-central1-c
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 172.16.0.2
EXTERNAL_IP: 35.226.51.88
STATUS: RUNNING
student_01_44f6d6297b5c@cloudshell:~ (qwiklabs-gcp-02-bffac160bdef)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-49.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-48.png)

Image Courtesy : GCP Labs/Networking in Google Cloud: Defining and Implementing Networks



There are three instances in `us-central1-c` and one instance in `europe-west1-c`. However, these instances are spread across three VPC networks (`managementnet`, `mynetwork`, and `privatene`t), with no instance in the same zone and network as another. In the next task, you explore the effect this has on internal connectivity.

>mynet-us-vm

```sh

Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-44f6d6297b5c'.
student-01-44f6d6297b5c@mynet-us-vm:~$ ping -c 3 104.155.101.10
PING 104.155.101.10 (104.155.101.10) 56(84) bytes of data.
64 bytes from 104.155.101.10: icmp_seq=1 ttl=53 time=105 ms
64 bytes from 104.155.101.10: icmp_seq=2 ttl=53 time=103 ms
64 bytes from 104.155.101.10: icmp_seq=3 ttl=53 time=103 ms

--- 104.155.101.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 103.001/103.576/104.599/0.725 ms
student-01-44f6d6297b5c@mynet-us-vm:~$ ping -c 3 34.136.43.5
PING 34.136.43.5 (34.136.43.5) 56(84) bytes of data.
64 bytes from 34.136.43.5: icmp_seq=1 ttl=61 time=2.14 ms
64 bytes from 34.136.43.5: icmp_seq=2 ttl=61 time=0.657 ms
64 bytes from 34.136.43.5: icmp_seq=3 ttl=61 time=0.587 ms

--- 34.136.43.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2007ms
rtt min/avg/max/mdev = 0.587/1.127/2.138/0.715 ms
student-01-44f6d6297b5c@mynet-us-vm:~$ ping -c 3 35.226.51.88
PING 35.226.51.88 (35.226.51.88) 56(84) bytes of data.
64 bytes from 35.226.51.88: icmp_seq=1 ttl=61 time=2.39 ms
64 bytes from 35.226.51.88: icmp_seq=2 ttl=61 time=0.586 ms
64 bytes from 35.226.51.88: icmp_seq=3 ttl=61 time=0.700 ms

--- 35.226.51.88 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2018ms
rtt min/avg/max/mdev = 0.586/1.223/2.385/0.822 ms
student-01-44f6d6297b5c@mynet-us-vm:~$ 
```

> You can ping the external IP address of all VM instances, even though they are in either a different zone or VPC network. This confirms that public access to those instances is only controlled by the **ICMP** firewall rules that you established earlier.



Internal connectivity verification

```sh
student-01-44f6d6297b5c@mynet-us-vm:~$ ping -c 3 10.132.0.2
PING 10.132.0.2 (10.132.0.2) 56(84) bytes of data.
64 bytes from 10.132.0.2: icmp_seq=1 ttl=64 time=103 ms
64 bytes from 10.132.0.2: icmp_seq=2 ttl=64 time=103 ms
64 bytes from 10.132.0.2: icmp_seq=3 ttl=64 time=103 ms

--- 10.132.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 102.558/102.686/102.914/0.161 ms
student-01-44f6d6297b5c@mynet-us-vm:~$ ping -c 3 10.130.0.2
PING 10.130.0.2 (10.130.0.2) 56(84) bytes of data.

--- 10.130.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2049ms

student-01-44f6d6297b5c@mynet-us-vm:~$ ping  -c 3 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.

--- 172.16.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2049ms

student-01-44f6d6297b5c@mynet-us-vm:~$ 
```

> You can ping the internal IP address of **mynet-eu-vm** because it is on the same VPC network as the source of the ping (**mynet-us-vm**), even though both VM instances are in separate zones, regions, and continents!



> You cannot ping the internal IP address of **managementnet-us-vm** and **privatenet-us-vm** because they are in separate VPC networks from the source of the ping (**mynet-us-vm**), even though they are all in the same zone, **us-central1**.



> VPC networks are by default isolated private networking domains. However, no internal IP address communication is allowed between networks, unless you set up mechanisms such as VPC peering or VPN.



Every instance in a VPC network has a default network interface. You can create additional network interfaces attached to your VMs. Multiple network interfaces enable you to create configurations in which an instance connects directly to several VPC networks (up to 8 interfaces, depending on the instance's type).

```sh
student-01-44f6d6297b5c@mynet-us-vm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:0a:80:00:02 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.128.0.2/32 brd 10.128.0.2 scope global dynamic ens4
       valid_lft 2051sec preferred_lft 2051sec
    inet6 fe80::4001:aff:fe80:2/64 scope link 
       valid_lft forever preferred_lft forever
student-01-44f6d6297b5c@mynet-us-vm:~$ ip route
default via 10.128.0.1 dev ens4 
10.128.0.1 dev ens4 scope link 
student-01-44f6d6297b5c@mynet-us-vm:~$ 
```



Create a VM with multiple NICs

```sh
gcloud compute instances create vm-appliance --project=qwiklabs-gcp-02-bffac160bdef --zone=us-central1-c --machine-type=n1-standard-4 --network-interface=network-tier=PREMIUM,subnet=privatesubnet-us --network-interface=network-tier=PREMIUM,subnet=managementsubnet-us --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=968172065011-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-appliance,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-bffac160bdef/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-50.png)

> vm-appliance

```sh
Linux vm-appliance 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-44f6d6297b5c'.
student-01-44f6d6297b5c@vm-appliance:~$ 
student-01-44f6d6297b5c@vm-appliance:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:ac:10:00:03 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 172.16.0.3/32 brd 172.16.0.3 scope global dynamic ens4
       valid_lft 3468sec preferred_lft 3468sec
    inet6 fe80::4001:acff:fe10:3/64 scope link 
       valid_lft forever preferred_lft forever
3: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:0a:82:00:03 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 10.130.0.3/32 brd 10.130.0.3 scope global dynamic ens5
       valid_lft 3468sec preferred_lft 3468sec
    inet6 fe80::4001:aff:fe82:3/64 scope link 
       valid_lft forever preferred_lft forever
4: ens6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:0a:80:00:03 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    inet 10.128.0.3/32 brd 10.128.0.3 scope global dynamic ens6
       valid_lft 3468sec preferred_lft 3468sec
    inet6 fe80::4001:aff:fe80:3/64 scope link 
       valid_lft forever preferred_lft forever
 
student-01-44f6d6297b5c@vm-appliance:~$ 
```

We can see the three NICs on this VM.



```sh
student-01-44f6d6297b5c@vm-appliance:~$ ping 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
64 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=1.51 ms
64 bytes from 172.16.0.2: icmp_seq=2 ttl=64 time=0.229 ms
64 bytes from 172.16.0.2: icmp_seq=3 ttl=64 time=0.292 ms
^C
--- 172.16.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2016ms
rtt min/avg/max/mdev = 0.229/0.678/1.513/0.590 ms

student-01-44f6d6297b5c@vm-appliance:~$ ping 10.130.0.2
PING 10.130.0.2 (10.130.0.2) 56(84) bytes of data.
64 bytes from 10.130.0.2: icmp_seq=1 ttl=64 time=0.236 ms
64 bytes from 10.130.0.2: icmp_seq=2 ttl=64 time=0.299 ms
64 bytes from 10.130.0.2: icmp_seq=3 ttl=64 time=0.435 ms
64 bytes from 10.130.0.2: icmp_seq=4 ttl=64 time=0.292 ms
^C
--- 10.130.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3068ms
rtt min/avg/max/mdev = 0.236/0.315/0.435/0.073 ms

student-01-44f6d6297b5c@vm-appliance:~$ ping 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 56(84) bytes of data.
64 bytes from 10.128.0.2: icmp_seq=1 ttl=64 time=1.49 ms
64 bytes from 10.128.0.2: icmp_seq=2 ttl=64 time=0.239 ms
64 bytes from 10.128.0.2: icmp_seq=3 ttl=64 time=0.222 ms
^C
--- 10.128.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2029ms
rtt min/avg/max/mdev = 0.222/0.651/1.492/0.594 ms
student-01-44f6d6297b5c@vm-appliance:~$ 
```

the **vm-appliance** instance is connected to **privatesubnet-us**, **managementsubnet-us**, and **mynetwork** by pinging VM instances on those subnets. In the my network, only US VM is reachable but not EU because we did not have this EU network on this vm-appliance.

```sh
student-01-44f6d6297b5c@vm-appliance:~$ ping 10.132.0.2
PING 10.132.0.2 (10.132.0.2) 56(84) bytes of data.
^C
--- 10.132.0.2 ping statistics ---
8 packets transmitted, 0 received, 100% packet loss, time 7144ms

student-01-44f6d6297b5c@vm-appliance:~$ 
```



In a multiple interface instance, every interface gets a route for the subnet that it is in. In addition, the instance gets a single default route that is associated with the primary interface eth0. Unless manually configured otherwise, any traffic leaving an instance for any destination other than a directly connected subnet will leave the instance via the default route on eth0.

```sh
student-01-44f6d6297b5c@vm-appliance:~$ ip route
default via 172.16.0.1 dev ens4 
10.128.0.0/20 via 10.128.0.1 dev ens6 
10.128.0.1 dev ens6 scope link 
10.130.0.0/20 via 10.130.0.1 dev ens5 
10.130.0.1 dev ens5 scope link 
172.16.0.0/24 via 172.16.0.1 dev ens4 
172.16.0.1 dev ens4 scope link 
student-01-44f6d6297b5c@vm-appliance:~$ 
```



Also,

```sh
student-01-44f6d6297b5c@vm-appliance:~$ ping -c 3 privatenet-us-vm
PING privatenet-us-vm.us-central1-c.c.qwiklabs-gcp-02-bffac160bdef.internal (172.16.0.2) 56(84) bytes of data.
64 bytes from privatenet-us-vm.us-central1-c.c.qwiklabs-gcp-02-bffac160bdef.internal (172.16.0.2): icmp_seq=1 ttl=64 time=1.59 ms
64 bytes from privatenet-us-vm.us-central1-c.c.qwiklabs-gcp-02-bffac160bdef.internal (172.16.0.2): icmp_seq=2 ttl=64 time=0.229 ms
64 bytes from privatenet-us-vm.us-central1-c.c.qwiklabs-gcp-02-bffac160bdef.internal (172.16.0.2): icmp_seq=3 ttl=64 time=0.220 ms

--- privatenet-us-vm.us-central1-c.c.qwiklabs-gcp-02-bffac160bdef.internal ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.220/0.679/1.590/0.643 ms
student-01-44f6d6297b5c@vm-appliance:~$ 
```

You can ping **privatenet-us-vm** by its name because VPC networks have an internal DNS service that allows you to address instances by their DNS names instead of their internal IP addresses. When an internal DNS query is made with the instance hostname, it resolves to the primary interface (nic0) of the instance. Therefore, this only works for **privatenet-us-vm** in this case.

```sh
student-01-44f6d6297b5c@vm-appliance:~$ ping -c 3 mynet-us-vm
ping: mynet-us-vm: Name or service not known
student-01-44f6d6297b5c@vm-appliance:~$ ping -c 3 mynet-us-vm
ping: mynet-us-vm: Name or service not known
student-01-44f6d6297b5c@vm-appliance:~$ ping -c 3 managementnet-us-vm
ping: managementnet-us-vm: Name or service not known
student-01-44f6d6297b5c@vm-appliance:~$ 
```

In this lab, we created several custom mode VPC networks, firewall rules, and VM instances using the `gcloud` command line. Then we tested the connectivity across VPC networks, which worked when pinging external IP addresses but not when pinging internal IP addresses. Thus we created a VM instance with three network interfaces and verified internal connectivity for VM instances that are on the subnets that are attached to the multiple interface VM.