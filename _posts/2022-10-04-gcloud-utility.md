---

layout: single
title:  "Using gcloud utility"
date:   2022-10-04 10:59:04 +0530
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
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---



# Creating GCP resources using gcloud utility

# Without any firewall rules

Creating a VPC without selecting any of the default firewall rules

```sh
gcloud compute networks create mynetwork --project=qwiklabs-gcp-02-24d285a2c4db --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional
```

Creating a VM in the US region

```sh
gcloud compute instances create mynet-us-vm --project=qwiklabs-gcp-02-24d285a2c4db --zone=us-central1-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=172382504241-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-24d285a2c4db/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Creating another VM in Europe region

```sh
gcloud compute instances create mynet-eu-vm --project=qwiklabs-gcp-02-24d285a2c4db --zone=europe-west4-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=172382504241-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-eu-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-24d285a2c4db/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

This will create all resources but we will not be able to SSH also to verify connectivity.
We need those firewall rules (`gcloud compute firewall-rules`) to enable management traffic and other ingress/egress traffic out of these VMs.

## With Firewall Rules

All in one line separated by `&&`

```sh
gcloud compute networks create mynetwork --project=qwiklabs-gcp-02-24d285a2c4db --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional && gcloud compute firewall-rules create mynetwork-allow-custom --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all && gcloud compute firewall-rules create mynetwork-allow-icmp --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp && gcloud compute firewall-rules create mynetwork-allow-rdp --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ RDP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 3389. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389 && gcloud compute firewall-rules create mynetwork-allow-ssh --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
```

Or separately

```sh
gcloud compute networks create mynetwork --project=qwiklabs-gcp-02-24d285a2c4db --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional
```

```sh
gcloud compute firewall-rules create mynetwork-allow-custom --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all
```

```sh
gcloud compute firewall-rules create mynetwork-allow-icmp --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ ICMP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=icmp
```

```sh
gcloud compute firewall-rules create mynetwork-allow-rdp --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ RDP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 3389. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:3389
```

```sh
gcloud compute firewall-rules create mynetwork-allow-ssh --project=qwiklabs-gcp-02-24d285a2c4db --network=projects/qwiklabs-gcp-02-24d285a2c4db/global/networks/mynetwork --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22
```

```sh
gcloud compute instances create mynet-us-vm --project=qwiklabs-gcp-02-24d285a2c4db --zone=us-central1-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=172382504241-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-us-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-24d285a2c4db/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

```sh
gcloud compute instances create mynet-eu-vm --project=qwiklabs-gcp-02-24d285a2c4db --zone=europe-west4-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=172382504241-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=mynet-eu-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-24d285a2c4db/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```



## Verification
>mynet-us-vm

```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-da96e1e7e410'.
student-01-da96e1e7e410@mynet-us-vm:~$ ping 35.204.119.221
PING 35.204.119.221 (35.204.119.221) 56(84) bytes of data.
64 bytes from 35.204.119.221: icmp_seq=1 ttl=51 time=104 ms
64 bytes from 35.204.119.221: icmp_seq=2 ttl=51 time=102 ms
^C
--- 35.204.119.221 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 102.038/102.900/103.763/0.862 ms
student-01-da96e1e7e410@mynet-us-vm:~$ ping 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.
64 bytes from 10.164.0.2: icmp_seq=1 ttl=64 time=104 ms
64 bytes from 10.164.0.2: icmp_seq=2 ttl=64 time=102 ms
64 bytes from 10.164.0.2: icmp_seq=3 ttl=64 time=103 ms
^C
--- 10.164.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 102.373/102.931/103.857/0.659 ms
student-01-da96e1e7e410@mynet-us-vm:~$ ip a
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
       valid_lft 3501sec preferred_lft 3501sec
    inet6 fe80::4001:aff:fe80:2/64 scope link 
       valid_lft forever preferred_lft forever
student-01-da96e1e7e410@mynet-us-vm:~$ ip route
default via 10.128.0.1 dev ens4 
10.128.0.1 dev ens4 scope link 
student-01-da96e1e7e410@mynet-us-vm:~$ 
```

>mynet-eu-vm

```sh
Linux mynet-eu-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-da96e1e7e410'.
student-01-da96e1e7e410@mynet-eu-vm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 42:01:0a:a4:00:02 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.164.0.2/32 brd 10.164.0.2 scope global dynamic ens4
       valid_lft 3482sec preferred_lft 3482sec
    inet6 fe80::4001:aff:fea4:2/64 scope link 
       valid_lft forever preferred_lft forever
student-01-da96e1e7e410@mynet-eu-vm:~$ ping 35.232.104.120
PING 35.232.104.120 (35.232.104.120) 56(84) bytes of data.
64 bytes from 35.232.104.120: icmp_seq=1 ttl=51 time=105 ms
64 bytes from 35.232.104.120: icmp_seq=2 ttl=51 time=102 ms
64 bytes from 35.232.104.120: icmp_seq=3 ttl=51 time=102 ms
^C
--- 35.232.104.120 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 102.112/103.054/104.869/1.283 ms
student-01-da96e1e7e410@mynet-eu-vm:~$ ping 10.128.0.2
PING 10.128.0.2 (10.128.0.2) 56(84) bytes of data.
64 bytes from 10.128.0.2: icmp_seq=1 ttl=64 time=111 ms
64 bytes from 10.128.0.2: icmp_seq=2 ttl=64 time=109 ms
64 bytes from 10.128.0.2: icmp_seq=3 ttl=64 time=109 ms
^C
--- 10.128.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 109.284/110.037/111.494/1.030 ms
student-01-da96e1e7e410@mynet-eu-vm:~$
```

