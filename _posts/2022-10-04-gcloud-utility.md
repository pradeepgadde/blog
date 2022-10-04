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

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-dbb9bd775ddb.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_9bcbe9228694@cloudshell:~ (qwiklabs-gcp-00-dbb9bd775ddb)$ gcloud compute zones list
NAME: us-east1-b
REGION: us-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east1-c
REGION: us-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east1-d
REGION: us-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east4-c
REGION: us-east4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east4-b
REGION: us-east4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east4-a
REGION: us-east4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-central1-c
REGION: us-central1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-central1-a
REGION: us-central1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-central1-f
REGION: us-central1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-central1-b
REGION: us-central1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west1-b
REGION: us-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west1-c
REGION: us-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west1-a
REGION: us-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west4-a
REGION: europe-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west4-b
REGION: europe-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west4-c
REGION: europe-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west1-b
REGION: europe-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west1-d
REGION: europe-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west1-c
REGION: europe-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west3-c
REGION: europe-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west3-a
REGION: europe-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west3-b
REGION: europe-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west2-c
REGION: europe-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west2-b
REGION: europe-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west2-a
REGION: europe-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east1-b
REGION: asia-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east1-a
REGION: asia-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east1-c
REGION: asia-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast1-b
REGION: asia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast1-a
REGION: asia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast1-c
REGION: asia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast1-b
REGION: asia-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast1-c
REGION: asia-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast1-a
REGION: asia-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south1-c
REGION: asia-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south1-b
REGION: asia-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south1-a
REGION: asia-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast1-b
REGION: australia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast1-c
REGION: australia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast1-a
REGION: australia-southeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-east1-b
REGION: southamerica-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-east1-c
REGION: southamerica-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-east1-a
REGION: southamerica-east1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east2-a
REGION: asia-east2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east2-b
REGION: asia-east2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-east2-c
REGION: asia-east2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast2-a
REGION: asia-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast2-b
REGION: asia-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast2-c
REGION: asia-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast3-a
REGION: asia-northeast3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast3-b
REGION: asia-northeast3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-northeast3-c
REGION: asia-northeast3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south2-a
REGION: asia-south2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south2-b
REGION: asia-south2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-south2-c
REGION: asia-south2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast2-a
REGION: asia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast2-b
REGION: asia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: asia-southeast2-c
REGION: asia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast2-a
REGION: australia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast2-b
REGION: australia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: australia-southeast2-c
REGION: australia-southeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-central2-a
REGION: europe-central2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-central2-b
REGION: europe-central2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-central2-c
REGION: europe-central2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-north1-a
REGION: europe-north1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-north1-b
REGION: europe-north1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-north1-c
REGION: europe-north1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-southwest1-a
REGION: europe-southwest1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-southwest1-b
REGION: europe-southwest1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-southwest1-c
REGION: europe-southwest1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west6-a
REGION: europe-west6
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west6-b
REGION: europe-west6
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west6-c
REGION: europe-west6
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west8-a
REGION: europe-west8
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west8-b
REGION: europe-west8
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west8-c
REGION: europe-west8
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west9-a
REGION: europe-west9
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west9-b
REGION: europe-west9
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: europe-west9-c
REGION: europe-west9
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: me-west1-a
REGION: me-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: me-west1-b
REGION: me-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: me-west1-c
REGION: me-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast1-a
REGION: northamerica-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast1-b
REGION: northamerica-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast1-c
REGION: northamerica-northeast1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast2-a
REGION: northamerica-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast2-b
REGION: northamerica-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: northamerica-northeast2-c
REGION: northamerica-northeast2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-west1-a
REGION: southamerica-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-west1-b
REGION: southamerica-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: southamerica-west1-c
REGION: southamerica-west1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east5-a
REGION: us-east5
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east5-b
REGION: us-east5
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-east5-c
REGION: us-east5
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-south1-a
REGION: us-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-south1-b
REGION: us-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-south1-c
REGION: us-south1
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west2-a
REGION: us-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west2-b
REGION: us-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west2-c
REGION: us-west2
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west3-a
REGION: us-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west3-b
REGION: us-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west3-c
REGION: us-west3
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west4-a
REGION: us-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west4-b
REGION: us-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:

NAME: us-west4-c
REGION: us-west4
STATUS: UP
NEXT_MAINTENANCE:
TURNDOWN_DATE:
student_02_9bcbe9228694@cloudshell:~ (qwiklabs-gcp-00-dbb9bd775ddb)$
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

