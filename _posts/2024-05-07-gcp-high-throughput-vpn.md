---

layout: single
title:  "Building a High-throughput VPN"
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

# Building a High-throughput VPN

Secure communication between Google Cloud and other clouds or  on-premises systems is a common, critical need. Fortunately, Google  Cloud makes it easy for you to create a secure Internet Protocol  security (IPsec) virtual private networks (VPNs) to achieve this goal.  If a single tunnel does not provide necessary throughput, Google Cloud  can smoothly distribute traffic across multiple tunnels to provide  additional bandwidth.

## Create VPN

- Create a Virtual Private Cloud (VPC) named `cloud` to simulate your Google Cloud network, and a VPC named `on-prem` (on-premises) to simulate an external network.
- Create VPN gateways, forwarding rules, and addresses for the `cloud` VPC.
- Form a tunnel for the new VPN, and route traffic through it.
- Repeat the VPN creation process for the `on-prem` VPC, creating a second VPN.

## Test VPNs

- Create a virtual machine (VM) using  [Compute Engine](https://cloud.google.com/compute/) for throughput load testing.
- Test throughput speed of a single VPN using `iperf`.

## Creating the cloud VPC

In this section, you will:

- Create a VPC to simulate your cloud production network.
- Allow common types of traffic to flow through the VPC.
- Create a subnet for deploying hosts.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-1572bc1b26ce.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute networks create cloud --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/networks/cloud].
NAME: cloud
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network cloud --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network cloud --allow tcp:22,tcp:3389,icmp

student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute firewall-rules create cloud-fw --network cloud --allow tcp:22,tcp:5001,udp:5001,icmp
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/firewalls/cloud-fw].                                      
Creating firewall...done.                                                                                                                                                          
NAME: cloud-fw
NETWORK: cloud
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:5001,udp:5001,icmp
DENY: 
DISABLED: False
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute networks subnets create cloud-east --network cloud \
    --range 10.0.1.0/24 --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/subnetworks/cloud-east].
NAME: cloud-east
REGION: europe-west4
NETWORK: cloud
RANGE: 10.0.1.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```



## Creating the on-prem VPC

In this section create a simulation of your `on-prem` VPC, or any network you want to connect to `cloud`. In practice you'll already have resources here, but for the purpose of  creating tunnels and validating configurations, follow these steps.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute networks create on-prem --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/networks/on-prem].
NAME: on-prem
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network on-prem --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network on-prem --allow tcp:22,tcp:3389,icmp

student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute firewall-rules create on-prem-fw --network on-prem --allow tcp:22,tcp:5001,udp:5001,icmp
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/firewalls/on-prem-fw].                                    
Creating firewall...done.                                                                                                                                                          
NAME: on-prem-fw
NETWORK: on-prem
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:5001,udp:5001,icmp
DENY: 
DISABLED: False
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute networks subnets create on-prem-central \
    --network on-prem --range 192.168.1.0/24 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/subnetworks/on-prem-central].
NAME: on-prem-central
REGION: us-west1
NETWORK: on-prem
RANGE: 192.168.1.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```



## Creating VPN gateways

Each environment requires VPN gateways for secure external  communication. Follow these steps to create the initial gateways for  your cloud and `on-prem` VPCs:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute target-vpn-gateways create on-prem-gw1 --network on-prem --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/targetVpnGateways/on-prem-gw1].
NAME: on-prem-gw1
NETWORK: on-prem
REGION: us-west1
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute target-vpn-gateways create cloud-gw1 --network cloud --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/targetVpnGateways/cloud-gw1].
NAME: cloud-gw1
NETWORK: cloud
REGION: europe-west4
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```



## Creating a route-based VPN tunnel between local and Google Cloud networks

The VPN gateways each need a static, external IP address so that  systems outside the VPC can communicate with them. Now you'll create IP  addresses and routes on the cloud and `on-prem` VPCs.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute addresses create cloud-gw1 --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/addresses/cloud-gw1].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute addresses create on-prem-gw1 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/addresses/on-prem-gw1].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ cloud_gw1_ip=$(gcloud compute addresses describe cloud-gw1 \
    --region europe-west4 --format='value(address)')
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ on_prem_gw_ip=$(gcloud compute addresses describe on-prem-gw1 \
    --region us-west1 --format='value(address)')
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

create forwarding rules for IPsec on the `cloud` VPC. You'll need to create forwarding rules in both directions.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create cloud-1-fr-esp --ip-protocol ESP \
    --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/forwardingRules/cloud-1-fr-esp].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create cloud-1-fr-udp500 --ip-protocol UDP \
    --ports 500 --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/forwardingRules/cloud-1-fr-udp500].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create cloud-fr-1-udp4500 --ip-protocol UDP \
    --ports 4500 --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/forwardingRules/cloud-fr-1-udp4500].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Use the same method to create firewall forwarding rules for the IPsec tunnel on the `on-prem` VPC. This step allows the IPsec tunnel to exit your firewalls:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create on-prem-fr-esp --ip-protocol ESP \
    --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/forwardingRules/on-prem-fr-esp].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create on-prem-fr-udp500 --ip-protocol UDP --ports 500 \
    --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/forwardingRules/on-prem-fr-udp500].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute forwarding-rules create on-prem-fr-udp4500 --ip-protocol UDP --ports 4500 \
    --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/forwardingRules/on-prem-fr-udp4500].
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Ordinarily you would need to go generate a secret for the next step, where you create and validate the tunnels `on-prem-tunnel1` and `cloud-tunnel1`. For details about how to create and securely store secrets, view the  [Secret Manager conceptual overview guide](https://cloud.google.com/kms/docs/secret-management).  For now just use the string "sharedsecret".

Create a tunnel for the local network `on-prem-tunnel1`, and for the cloud-based network `cloud-tunnel1`. Each network must have a VPN gateway, and the secrets must match. In  the following two commands, where you would, in a production scenario,  replace `[MY_SECRET]` with the secret you generated, replace it with "sharedsecret"

Create the VPN tunnel from `on-prem` to `cloud`:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute vpn-tunnels create on-prem-tunnel1 --peer-address $cloud_gw1_ip \
    --target-vpn-gateway on-prem-gw1 --ike-version 2 --local-traffic-selector 0.0.0.0/0 \
    --remote-traffic-selector 0.0.0.0/0 --shared-secret=[MY_SECRET] --region us-west1
Creating VPN tunnel...done.                                                                                                                                                        
NAME: on-prem-tunnel1
REGION: us-west1
GATEWAY: on-prem-gw1
VPN_INTERFACE: 
PEER_ADDRESS: 34.91.185.118
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Create the VPN tunnel from cloud to on-prem:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute vpn-tunnels create cloud-tunnel1 --peer-address $on_prem_gw_ip \
    --target-vpn-gateway cloud-gw1 --ike-version 2 --local-traffic-selector 0.0.0.0/0 \
    --remote-traffic-selector 0.0.0.0/0 --shared-secret=[MY_SECRET] --region europe-west4
Creating VPN tunnel...done.                                                                                                                                                        
NAME: cloud-tunnel1
REGION: europe-west4
GATEWAY: cloud-gw1
VPN_INTERFACE: 
PEER_ADDRESS: 35.233.179.69
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Now that you've created the gateways and built the tunnels, you need to add routes from the subnets through the two tunnels.

1. Route traffic from the `on-prem` VPC to the `cloud 10.0.1.0/24` range into the tunnel:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute routes create on-prem-route1 --destination-range 10.0.1.0/24 \
    --network on-prem --next-hop-vpn-tunnel on-prem-tunnel1 \
    --next-hop-vpn-tunnel-region us-west1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/routes/on-prem-route1].
NAME: on-prem-route1
NETWORK: on-prem
DEST_RANGE: 10.0.1.0/24
NEXT_HOP: us-west1/vpnTunnels/on-prem-tunnel1
PRIORITY: 1000
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Route traffic from the `cloud` VPC to the `on-prem 192.168.1.0/24` range into the tunnel:

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute routes create cloud-route1 --destination-range 192.168.1.0/24 \
    --network cloud --next-hop-vpn-tunnel cloud-tunnel1 --next-hop-vpn-tunnel-region europe-west4
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/global/routes/cloud-route1].
NAME: cloud-route1
NETWORK: cloud
DEST_RANGE: 192.168.1.0/24
NEXT_HOP: europe-west4/vpnTunnels/cloud-tunnel1
PRIORITY: 1000
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```



## Testing throughput over VPN

At this point, you've established a secure path between the on-prem and cloud VPCs. To test throughput use  [iperf](https://iperf.fr/), an open-source tool for network load testing. To test, you'll need a VM in each environment, one to send traffic and the other to receive it,  and you'll create them next.

### Single VPN load testing

Now you'll create a virtual machine for the cloud VPC named is `cloud-loadtest`. This example uses a Debian Linux image for the OS.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute instances create "cloud-loadtest" --zone europe-west4-a \
    --machine-type "e2-standard-4" --subnet "cloud-east" \
    --image-family "debian-11" --image-project "debian-cloud" --boot-disk-size "10" \
    --boot-disk-type "pd-standard" --boot-disk-device-name "cloud-loadtest"
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/zones/europe-west4-a/instances/cloud-loadtest].
NAME: cloud-loadtest
ZONE: europe-west4-a
MACHINE_TYPE: e2-standard-4
PREEMPTIBLE: 
INTERNAL_IP: 10.0.1.2
EXTERNAL_IP: 34.34.68.43
STATUS: RUNNING
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

Create a virtual machine for the `on-prem` VPC named `on-prem-loadtest`. This example uses the same Debian image as in the cloud VPC. Omit this step if you have existing resources.

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute instances create "on-prem-loadtest" --zone us-west1-c \
    --machine-type "e2-standard-4" --subnet "on-prem-central" \
    --image-family "debian-11" --image-project "debian-cloud" --boot-disk-size "10" \
    --boot-disk-type "pd-standard" --boot-disk-device-name "on-prem-loadtest"
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/zones/us-west1-c/instances/on-prem-loadtest].
NAME: on-prem-loadtest
ZONE: us-west1-c
MACHINE_TYPE: e2-standard-4
PREEMPTIBLE: 
INTERNAL_IP: 192.168.1.2
EXTERNAL_IP: 35.230.0.16
STATUS: RUNNING
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute ssh --zone "europe-west4-a" "cloud-loadtest" --project "qwiklabs-gcp-00-1572bc1b26ce"
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
SHA256:fD1ToonWOtwWsRVDMQ/h2v5CubR+RWfjvySDO4oqrqE student_01_c9f851639dd7@cs-621429599476-default
The key's randomart image is:
+---[RSA 3072]----+
|           .Oo   |
|           . *   |
|          . + o  |
|       . o X o .+|
|        S O =..oo|
|       o + o=o ..|
| .      + o+.= o.|
|. ..    .o .=.= .|
|E.o..... ..ooo...|
+----[SHA256]-----+
Warning: Permanently added 'compute.1877535288571976263' (ED25519) to the list of known hosts.
Linux cloud-loadtest 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-c9f851639dd7'.
student-01-c9f851639dd7@cloud-loadtest:~$ sudo apt-get install iperf
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  iperf
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 102 kB of archives.
After this operation, 269 kB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 iperf amd64 2.0.14a+dfsg1-1 [102 kB]
Fetched 102 kB in 0s (1381 kB/s)
Selecting previously unselected package iperf.
(Reading database ... 64633 files and directories currently installed.)
Preparing to unpack .../iperf_2.0.14a+dfsg1-1_amd64.deb ...
Unpacking iperf (2.0.14a+dfsg1-1) ...
Setting up iperf (2.0.14a+dfsg1-1) ...
Processing triggers for man-db (2.9.4-2) ...
student-01-c9f851639dd7@cloud-loadtest:~$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute ssh --zone "us-west1-c" "on-prem-loadtest" --project "qwiklabs-gcp-00-1572bc1b26ce"
Warning: Permanently added 'compute.7124077388141943351' (ED25519) to the list of known hosts.
Linux on-prem-loadtest 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-c9f851639dd7'.
student-01-c9f851639dd7@on-prem-loadtest:~$ sudo apt-get install iperf
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  iperf
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 102 kB of archives.
After this operation, 269 kB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 iperf amd64 2.0.14a+dfsg1-1 [102 kB]
Fetched 102 kB in 0s (1185 kB/s)
Selecting previously unselected package iperf.
(Reading database ... 64633 files and directories currently installed.)
Preparing to unpack .../iperf_2.0.14a+dfsg1-1_amd64.deb ...
Unpacking iperf (2.0.14a+dfsg1-1) ...
Setting up iperf (2.0.14a+dfsg1-1) ...
Processing triggers for man-db (2.9.4-2) ...
student-01-c9f851639dd7@on-prem-loadtest:~$ 
```

On the `on-prem-loadtest` VM, run this command:

You have created an iperf server on the VM that reports its status every 5 seconds.

On the `cloud-loadtest` VM, run this command:

This creates an iperf client with twenty streams, which will report values after 10 seconds of testing.

```sh
student-01-c9f851639dd7@cloud-loadtest:~$ iperf -c 192.168.1.2 -P 20 -x C
------------------------------------------------------------
Client connecting to 192.168.1.2, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[ ID] Interval       Transfer     Bandwidth
[ 15] 0.0000-10.0128 sec   147 MBytes   123 Mbits/sec
[ 17] 0.0000-10.0151 sec   163 MBytes   137 Mbits/sec
[ 20] 0.0000-10.0187 sec   148 MBytes   124 Mbits/sec
[ 10] 0.0000-10.0312 sec   167 MBytes   139 Mbits/sec
[ 14] 0.0000-10.0293 sec   163 MBytes   136 Mbits/sec
[  9] 0.0000-10.0142 sec   178 MBytes   149 Mbits/sec
[  6] 0.0000-10.0845 sec   177 MBytes   147 Mbits/sec
[ 18] 0.0000-10.0057 sec  98.9 MBytes  82.9 Mbits/sec
[  5] 0.0000-10.0224 sec   100 MBytes  83.9 Mbits/sec
[ 11] 0.0000-10.1083 sec   108 MBytes  89.2 Mbits/sec
[ 12] 0.0000-10.0049 sec  90.0 MBytes  75.5 Mbits/sec
[ 19] 0.0000-10.0120 sec  86.9 MBytes  72.8 Mbits/sec
[  4] 0.0000-10.1152 sec  98.6 MBytes  81.8 Mbits/sec
[  3] 0.0000-10.1058 sec  97.3 MBytes  80.7 Mbits/sec
[ 21] 0.0000-10.1142 sec   102 MBytes  84.3 Mbits/sec
[ 16] 0.0000-10.0817 sec  78.9 MBytes  65.6 Mbits/sec
[ 22] 0.0000-10.0874 sec  69.5 MBytes  57.8 Mbits/sec
[  7] 0.0000-10.1166 sec  66.8 MBytes  55.3 Mbits/sec
[  8] 0.0000-10.1208 sec  55.1 MBytes  45.7 Mbits/sec
[ 13] 0.0000-10.2255 sec  57.3 MBytes  47.0 Mbits/sec
[SUM] 0.0000-10.2269 sec  2.20 GBytes  1.85 Gbits/sec
student-01-c9f851639dd7@cloud-loadtest:~$ 
```

```sh
student-01-c9f851639dd7@on-prem-loadtest:~$ iperf -s -i 5
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37664
[  6] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37652
[  7] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37532
[  8] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37688
[  9] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37582
[ 10] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37616
[ 12] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37552
[ 13] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37516
[ 14] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37644
[ 15] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37672
[ 16] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37624
[ 17] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37556
[ 18] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37700
[ 19] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37514
[ 20] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37596
[ 21] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37690
[ 22] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37630
[ 23] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37546
[  5] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37566
[ 11] local 192.168.1.2 port 5001 connected with 10.0.1.2 port 37612
[ ID] Interval       Transfer     Bandwidth
[  7] 0.0000-5.0000 sec  41.8 MBytes  70.1 Mbits/sec
[  8] 0.0000-5.0000 sec  59.7 MBytes   100 Mbits/sec
[ 22] 0.0000-5.0000 sec  33.5 MBytes  56.3 Mbits/sec
[ 23] 0.0000-5.0000 sec  28.5 MBytes  47.8 Mbits/sec
[  4] 0.0000-5.0000 sec  37.5 MBytes  62.9 Mbits/sec
[  5] 0.0000-5.0000 sec  67.2 MBytes   113 Mbits/sec
[  6] 0.0000-5.0000 sec  66.0 MBytes   111 Mbits/sec
[  9] 0.0000-5.0000 sec  72.6 MBytes   122 Mbits/sec
[ 10] 0.0000-5.0000 sec  65.4 MBytes   110 Mbits/sec
[ 11] 0.0000-5.0000 sec  39.1 MBytes  65.6 Mbits/sec
[ 13] 0.0000-5.0000 sec  41.9 MBytes  70.3 Mbits/sec
[ 14] 0.0000-5.0000 sec  59.6 MBytes  99.9 Mbits/sec
[ 15] 0.0000-5.0000 sec  42.8 MBytes  71.8 Mbits/sec
[ 17] 0.0000-5.0000 sec  23.8 MBytes  39.9 Mbits/sec
[ 12] 0.0000-5.0000 sec  71.6 MBytes   120 Mbits/sec
[ 19] 0.0000-5.0000 sec  43.0 MBytes  72.2 Mbits/sec
[ 20] 0.0000-5.0000 sec  45.8 MBytes  76.9 Mbits/sec
[ 16] 0.0000-5.0000 sec  24.2 MBytes  40.6 Mbits/sec
[ 18] 0.0000-5.0000 sec  29.4 MBytes  49.3 Mbits/sec
[ 21] 0.0000-5.0000 sec  43.0 MBytes  72.2 Mbits/sec
[SUM] 0.0000-5.0000 sec   936 MBytes  1.57 Gbits/sec
[ 22] 5.0000-10.0000 sec  43.6 MBytes  73.1 Mbits/sec
[ 23] 5.0000-10.0000 sec  36.3 MBytes  60.9 Mbits/sec
[  4] 5.0000-10.0000 sec  47.8 MBytes  80.2 Mbits/sec
[  5] 5.0000-10.0000 sec  98.6 MBytes   165 Mbits/sec
[  6] 5.0000-10.0000 sec  96.8 MBytes   162 Mbits/sec
[  7] 5.0000-10.0000 sec  53.3 MBytes  89.4 Mbits/sec
[  8] 5.0000-10.0000 sec  87.6 MBytes   147 Mbits/sec
[  9] 5.0000-10.0000 sec   105 MBytes   176 Mbits/sec
[ 10] 5.0000-10.0000 sec  96.2 MBytes   161 Mbits/sec
[ 11] 5.0000-10.0000 sec  49.6 MBytes  83.2 Mbits/sec
[ 13] 5.0000-10.0000 sec  55.1 MBytes  92.4 Mbits/sec
[ 14] 5.0000-10.0000 sec  87.3 MBytes   147 Mbits/sec
[ 15] 5.0000-10.0000 sec  54.9 MBytes  92.1 Mbits/sec
[ 19] 5.0000-10.0000 sec  56.0 MBytes  93.9 Mbits/sec
[  5] 10.0000-10.0290 sec   782 KBytes   221 Mbits/sec
[  5] 0.0000-10.0290 sec   167 MBytes   139 Mbits/sec
[  8] 10.0000-10.0212 sec  1.00 MBytes   396 Mbits/sec
[  8] 0.0000-10.0212 sec   148 MBytes   124 Mbits/sec
[ 10] 10.0000-10.0274 sec   984 KBytes   294 Mbits/sec
[ 10] 0.0000-10.0274 sec   163 MBytes   136 Mbits/sec
[ 14] 10.0000-10.0169 sec   222 KBytes   107 Mbits/sec
[ 14] 0.0000-10.0169 sec   147 MBytes   123 Mbits/sec
[ 16] 5.0000-10.0000 sec  30.4 MBytes  51.0 Mbits/sec
[ 20] 5.0000-10.0000 sec  60.5 MBytes   101 Mbits/sec
[SUM] 5.0000-10.0000 sec  1.04 GBytes  1.78 Gbits/sec
[ 21] 5.0000-10.0000 sec  56.5 MBytes  94.9 Mbits/sec
[  6] 10.0000-10.0133 sec   170 KBytes   105 Mbits/sec
[  6] 0.0000-10.0133 sec   163 MBytes   137 Mbits/sec
[  9] 10.0000-10.0119 sec   397 KBytes   273 Mbits/sec
[  9] 0.0000-10.0119 sec   178 MBytes   149 Mbits/sec
[ 17] 5.0000-10.0000 sec  28.8 MBytes  48.4 Mbits/sec
[ 18] 5.0000-10.0000 sec  38.5 MBytes  64.6 Mbits/sec
[ 12] 5.0000-10.0000 sec   105 MBytes   176 Mbits/sec
[ 12] 10.0000-10.0814 sec   572 KBytes  57.6 Mbits/sec
[ 12] 0.0000-10.0814 sec   177 MBytes   147 Mbits/sec
[ 15] 10.0000-10.1056 sec  1.21 MBytes  96.5 Mbits/sec
[ 15] 0.0000-10.1056 sec  98.9 MBytes  82.1 Mbits/sec
[ 19] 10.0000-10.1070 sec  1.25 MBytes  97.8 Mbits/sec
[ 19] 0.0000-10.1070 sec   100 MBytes  83.2 Mbits/sec
[ 11] 10.0000-10.1256 sec  1.30 MBytes  86.9 Mbits/sec
[ 11] 0.0000-10.1256 sec  90.0 MBytes  74.6 Mbits/sec
[ 20] 10.0000-10.1214 sec  1.19 MBytes  82.1 Mbits/sec
[ 20] 0.0000-10.1214 sec   108 MBytes  89.1 Mbits/sec
[  4] 10.0000-10.1442 sec  1.55 MBytes  90.4 Mbits/sec
[  4] 0.0000-10.1442 sec  86.9 MBytes  71.8 Mbits/sec
[ 13] 10.0000-10.1983 sec  1.64 MBytes  69.5 Mbits/sec
[ 13] 0.0000-10.1983 sec  98.6 MBytes  81.1 Mbits/sec
[  7] 10.0000-10.2245 sec  2.20 MBytes  82.4 Mbits/sec
[  7] 0.0000-10.2245 sec  97.3 MBytes  79.8 Mbits/sec
[ 18] 10.0000-10.2373 sec  1.67 MBytes  58.9 Mbits/sec
[ 18] 0.0000-10.2373 sec  69.5 MBytes  56.9 Mbits/sec
[ 22] 10.0000-10.2311 sec  1.76 MBytes  63.8 Mbits/sec
[ 22] 0.0000-10.2311 sec  78.9 MBytes  64.7 Mbits/sec
[ 21] 10.0000-10.2330 sec  2.04 MBytes  73.6 Mbits/sec
[ 21] 0.0000-10.2330 sec   102 MBytes  83.3 Mbits/sec
[ 23] 10.0000-10.2624 sec  1.94 MBytes  62.1 Mbits/sec
[ 23] 0.0000-10.2624 sec  66.8 MBytes  54.6 Mbits/sec
[ 16] 10.0000-10.4931 sec  2.65 MBytes  45.1 Mbits/sec
[ 16] 0.0000-10.4931 sec  57.3 MBytes  45.8 Mbits/sec
[ 17] 10.0000-10.4841 sec  2.48 MBytes  43.0 Mbits/sec
[ 17] 0.0000-10.4841 sec  55.1 MBytes  44.1 Mbits/sec
[SUM] 10.0000-10.4895 sec   253 MBytes  4.33 Gbits/sec
[SUM] 0.0000-10.4895 sec  2.20 GBytes  1.80 Gbits/sec

```



## Summary

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ history 
    1  gcloud compute networks create cloud --subnet-mode custom
    2  gcloud compute firewall-rules create cloud-fw --network cloud --allow tcp:22,tcp:5001,udp:5001,icmp
    3  gcloud compute networks subnets create cloud-east --network cloud     --range 10.0.1.0/24 --region europe-west4
    4  gcloud compute networks create on-prem --subnet-mode custom
    5  gcloud compute firewall-rules create on-prem-fw --network on-prem --allow tcp:22,tcp:5001,udp:5001,icmp
    6  gcloud compute networks subnets create on-prem-central     --network on-prem --range 192.168.1.0/24 --region us-west1
    7  gcloud compute target-vpn-gateways create on-prem-gw1 --network on-prem --region us-west1
    8  gcloud compute target-vpn-gateways create cloud-gw1 --network cloud --region europe-west4
    9  gcloud compute addresses create cloud-gw1 --region europe-west4
   10  gcloud compute addresses create on-prem-gw1 --region us-west1
   11  cloud_gw1_ip=$(gcloud compute addresses describe cloud-gw1 \
    --region europe-west4 --format='value(address)')
   12  on_prem_gw_ip=$(gcloud compute addresses describe on-prem-gw1 \
    --region us-west1 --format='value(address)')
   13  gcloud compute forwarding-rules create cloud-1-fr-esp --ip-protocol ESP     --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
   14  gcloud compute forwarding-rules create cloud-1-fr-udp500 --ip-protocol UDP     --ports 500 --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
   15  gcloud compute forwarding-rules create cloud-fr-1-udp4500 --ip-protocol UDP     --ports 4500 --address $cloud_gw1_ip --target-vpn-gateway cloud-gw1 --region europe-west4
   16  gcloud compute forwarding-rules create on-prem-fr-esp --ip-protocol ESP     --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
   17  gcloud compute forwarding-rules create on-prem-fr-udp500 --ip-protocol UDP --ports 500     --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
   18  gcloud compute forwarding-rules create on-prem-fr-udp4500 --ip-protocol UDP --ports 4500     --address $on_prem_gw_ip --target-vpn-gateway on-prem-gw1 --region us-west1
   19  gcloud compute vpn-tunnels create on-prem-tunnel1 --peer-address $cloud_gw1_ip     --target-vpn-gateway on-prem-gw1 --ike-version 2 --local-traffic-selector 0.0.0.0/0     --remote-traffic-selector 0.0.0.0/0 --shared-secret=[MY_SECRET] --region us-west1
   20  gcloud compute vpn-tunnels create cloud-tunnel1 --peer-address $on_prem_gw_ip     --target-vpn-gateway cloud-gw1 --ike-version 2 --local-traffic-selector 0.0.0.0/0     --remote-traffic-selector 0.0.0.0/0 --shared-secret=[MY_SECRET] --region europe-west4
   21  gcloud compute routes create on-prem-route1 --destination-range 10.0.1.0/24     --network on-prem --next-hop-vpn-tunnel on-prem-tunnel1     --next-hop-vpn-tunnel-region us-west1
   22  gcloud compute routes create cloud-route1 --destination-range 192.168.1.0/24     --network cloud --next-hop-vpn-tunnel cloud-tunnel1 --next-hop-vpn-tunnel-region europe-west4
   23  gcloud compute instances create "cloud-loadtest" --zone europe-west4-a     --machine-type "e2-standard-4" --subnet "cloud-east"     --image-family "debian-11" --image-project "debian-cloud" --boot-disk-size "10"     --boot-disk-type "pd-standard" --boot-disk-device-name "cloud-loadtest"
   24  gcloud compute instances create "on-prem-loadtest" --zone us-west1-c     --machine-type "e2-standard-4" --subnet "on-prem-central"     --image-family "debian-11" --image-project "debian-cloud" --boot-disk-size "10"     --boot-disk-type "pd-standard" --boot-disk-device-name "on-prem-loadtest"
   25  gcloud compute ssh --zone "europe-west4-a" "cloud-loadtest" --project "qwiklabs-gcp-00-1572bc1b26ce"
   26  history 
   27  gcloud compute ssh --zone "us-west1-c" "on-prem-loadtest" --project "qwiklabs-gcp-00-1572bc1b26ce"
   28  history 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute vpn-tunnels describe cloud-tunnel1 --region europe-west4
creationTimestamp: '2024-05-07T10:25:32.385-07:00'
description: ''
detailedStatus: Tunnel is up and running.
id: '8824705603509672675'
ikeVersion: 2
kind: compute#vpnTunnel
labelFingerprint: 42WmSpB8rSM=
localTrafficSelector:
- 0.0.0.0/0
name: cloud-tunnel1
peerIp: 35.233.179.69
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4
remoteTrafficSelector:
- 0.0.0.0/0
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/vpnTunnels/cloud-tunnel1
sharedSecret: '*************'
sharedSecretHash: x1XAl8AN1vrs5dAjQWU-B3Y1kBqx
status: ESTABLISHED
targetVpnGateway: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/europe-west4/targetVpnGateways/cloud-gw1
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

```sh
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ gcloud compute vpn-tunnels describe on-prem-tunnel1 --region us-west1
creationTimestamp: '2024-05-07T10:24:34.149-07:00'
description: ''
detailedStatus: Tunnel is up and running.
id: '7247586766240603453'
ikeVersion: 2
kind: compute#vpnTunnel
labelFingerprint: 42WmSpB8rSM=
localTrafficSelector:
- 0.0.0.0/0
name: on-prem-tunnel1
peerIp: 34.91.185.118
region: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1
remoteTrafficSelector:
- 0.0.0.0/0
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/vpnTunnels/on-prem-tunnel1
sharedSecret: '*************'
sharedSecretHash: zMN_icRCu3bjOTyPTNZEJ3LtPt7G
status: ESTABLISHED
targetVpnGateway: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-1572bc1b26ce/regions/us-west1/targetVpnGateways/on-prem-gw1
student_01_c9f851639dd7@cloudshell:~ (qwiklabs-gcp-00-1572bc1b26ce)$ 
```

