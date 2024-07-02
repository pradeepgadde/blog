---

layout: single
title:  "Build a Secure Google Cloud Network: Challenge Lab"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Build a Secure Google Cloud Network: Challenge Lab

## Challenge scenario

You are a security consultant brought in by Jeff, who owns a small  local company, to help him with his very successful website (juiceshop). Jeff is new to Google Cloud and had his neighbour's son set up the  initial site. The neighbour's son has since had to leave for college,  but before leaving, he made sure the site was running.

You need to create the appropriate security configuration for Jeff's  site. Your first challenge is to set up firewall rules and virtual  machine tags.  You also need to ensure that SSH is only available to the bastion via IAP.

For the firewall rules, make sure that:

- The bastion host does not have a public IP address.

- You can only SSH to the bastion and only via IAP.

- You can only SSH to `juice-shop` via the bastion.

- Only HTTP is open to the world for `juice-shop`.

  

![The Google Cloud environment to configure](https://cdn.qwiklabs.com/BgxgsuLyqMkhxmO3jDlkHE7yGLIR%2B3rrUabKimlgrbo%3D)

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-f5adeb762d87.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute instances list
NAME: bastion
ZONE: europe-west1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 192.168.10.2
EXTERNAL_IP: 
STATUS: TERMINATED

NAME: juice-shop
ZONE: europe-west1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 192.168.11.2
EXTERNAL_IP: 34.76.141.117
STATUS: RUNNING
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute firewall-rules list
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

NAME: open-access
NETWORK: acme-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



Check the firewall rules.  Remove the overly permissive rules.

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute firewall-rules delete open-access 
The following firewalls will be deleted:
 - [open-access]

Do you want to continue (Y/n)?  y

Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/global/firewalls/open-access].
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



Navigate to Compute Engine in the Cloud console and identify the bastion host. The instance should be stopped. Start the instance.

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute instances describe bastion --zone europe-west1-c
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-07-02T09:27:16.625-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/disks/bastion
  type: PERSISTENT
fingerprint: s6QEDzsH2n8=
id: '4758679786869068939'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-07-02T10:10:58.723-07:00'
lastStopTimestamp: '2024-07-02T09:29:07.719-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/machineTypes/e2-micro
metadata:
  fingerprint: 2tnXmpe-C8I=
  items:
  - key: startup-script
    value: |
      #! /bin/bash
      if [ -e "/root/foo" ]
      then
        echo its ok
      else
        apt update
        curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
        bash install-logging-agent.sh
        gsutil cp gs://cloud-training/gsp506/auth.conf /etc/google-fluentd/config.d/auth.conf
        service google-fluentd restart
        touch /root/foo
        poweroff
      fi
  kind: compute#metadata
name: bastion
networkInterfaces:
- fingerprint: INHQ11R2Gv4=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/global/networks/acme-vpc
  networkIP: 192.168.10.2
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/regions/europe-west1/subnetworks/acme-mgmt-subnet
satisfiesPzi: true
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/instances/bastion
serviceAccounts:
- email: 1058778023706-compute@developer.gserviceaccount.com
  scopes:
  - https://www.googleapis.com/auth/cloud-platform
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: SFX9LNAOdG8=
  items:
  - allow-ssh
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



The bastion host is the one machine authorized to receive external SSH traffic. Create a firewall rule that allows [SSH (tcp/22) from the IAP service](https://cloud.google.com/iap/docs/using-tcp-forwarding). The firewall rule must be enabled for the bastion host instance using a network tag of

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ export BASTION_TAG=allow-ssh-iap-ingress-ql-752
export SSH_INTERNAL_TAG=allow-ssh-internal-ingress-ql-752
export HTTP_EXTERNAL_TAG=allow-http-ingress-ql-752
export ZONE=europe-west1-c
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



Create a firewall rule that allows SSH (tcp/22) from the IAP service and add network tag on bastion

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute firewall-rules create iap-ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags $BASTION_TAG --network acme-vpc
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/global/firewalls/iap-ssh-ingress].                               
Creating firewall...done.                                                                                                                                                          
NAME: iap-ssh-ingress
NETWORK: acme-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$
```

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute instances add-tags bastion --tags=$BASTION_TAG --zone=$ZONE
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/instances/bastion].
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



The `juice-shop` server serves HTTP traffic. Create a  firewall rule that allows traffic on HTTP (tcp/80) to any address. The  firewall rule must be enabled for the juice-shop instance using a  network tag of .

Create a firewall rule that allows traffic on HTTP (tcp/80) to any address    and add network tag on juice-shop

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute firewall-rules create http-external --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags $HTTP_EXTERNAL_TAG --network acme-vpc
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/global/firewalls/http-external].                                 
Creating firewall...done.                                                                                                                                                          
NAME: http-external
NETWORK: acme-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$
```

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute instances add-tags juice-shop --tags=$HTTP_EXTERNAL_TAG --zone=$ZONE
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/instances/juice-shop].
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



You need to connect to `juice-shop` from the bastion using SSH. Create a firewall rule that allows traffic on SSH (tcp/22) from `acme-mgmt-subnet` network address. The firewall rule must be enabled for the `juice-shop` instance using a network tag of

Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute firewall-rules create ssh-internal --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags $SSH_INTERNAL_TAG --network acme-vpc
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/global/firewalls/ssh-internal].                                  
Creating firewall...done.                                                                                                                                                          
NAME: ssh-internal
NETWORK: acme-vpc
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute instances add-tags juice-shop --tags=$SSH_INTERNAL_TAG --zone=$ZONE
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-f5adeb762d87/zones/europe-west1-c/instances/juice-shop].
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```



In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to `juice-shop`.

SSH to bastion host via IAP and juice-shop via bastion

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ gcloud compute ssh --zone "europe-west1-c" "bastion" --tunnel-through-iap --project "qwiklabs-gcp-02-f5adeb762d87"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_03_9717f3e77292/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_03_9717f3e77292/.ssh/google_compute_engine
Your public key has been saved in /home/student_03_9717f3e77292/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:sCc5EC3ycBJP1Bih8l0YQB6F242sb3wlpIpCBSC3ens student_03_9717f3e77292@cs-1044452471987-default
The key's randomart image is:
+---[RSA 3072]----+
|+.**O*           |
|.+=*++o          |
|. =X++o          |
| +.o=oo+         |
|. +..o= S        |
| o... .+.        |
|...+E  o         |
|o ..+ .          |
|.  . .           |
+----[SHA256]-----+
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.4758679786869068939' (ED25519) to the list of known hosts.
Linux bastion 5.10.0-30-cloud-amd64 #1 SMP Debian 5.10.218-1 (2024-06-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-03-9717f3e77292'.
student-03-9717f3e77292@bastion:~$ 
```

```sh
student-03-9717f3e77292@bastion:~$ gcloud compute ssh juice-shop --internal-ip
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student-03-9717f3e77292/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student-03-9717f3e77292/.ssh/google_compute_engine
Your public key has been saved in /home/student-03-9717f3e77292/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:N8IcNd9Og0BefrOR83XQtymEihK8LJD2lW+cfvr/pHA student-03-9717f3e77292@bastion
The key's randomart image is:
+---[RSA 3072]----+
|  . . .  .= o .. |
| +   =   o B + oo|
|. o o = + o = X *|
|   o + O o   = Xo|
|    . + S o   + .|
|       . + .     |
|        o. E .   |
|       .  o o    |
|        ...o..   |
+----[SHA256]-----+
Did you mean zone [europe-west1-c] for instance: [juice-shop] (Y/n)?  y

Linux juice-shop 5.10.0-30-cloud-amd64 #1 SMP Debian 5.10.218-1 (2024-06-01) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
sa_112040232431885774406@juice-shop:~$ 
```



You've completed the challenge lab and helped Jeff tighten security.

## History

```sh
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ history 
    1  gcloud compute instances list
    2  gcloud compute firewall-rules list
    3  gcloud compute firewall-rules delete open-access 
    4  gcloud compute instances describe bastion
    5  gcloud compute instances describe bastion --zone europe-west1-c
    {snip}
   11  export BASTION_TAG=allow-ssh-iap-ingress-ql-752
   12  export SSH_INTERNAL_TAG=allow-ssh-internal-ingress-ql-752
   13  export HTTP_EXTERNAL_TAG=allow-http-ingress-ql-752
   14  export ZONE=europe-west1-c
   15  gcloud compute firewall-rules create iap-ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags $BASTION_TAG --network acme-vpc
   16  gcloud compute instances add-tags bastion --tags=$BASTION_TAG --zone=$ZONE
   17  gcloud compute firewall-rules create http-external --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags $HTTP_EXTERNAL_TAG --network acme-vpc
   18  gcloud compute instances add-tags juice-shop --tags=$HTTP_EXTERNAL_TAG --zone=$ZONE
   19  gcloud compute firewall-rules create ssh-internal --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags $SSH_INTERNAL_TAG --network acme-vpc
{snip}
   21  gcloud compute instances add-tags juice-shop --tags=$SSH_INTERNAL_TAG --zone=$ZONE
   22  gcloud compute ssh --zone "europe-west1-c" "bastion" --tunnel-through-iap --project "qwiklabs-gcp-02-f5adeb762d87"
   23  history 
student_03_9717f3e77292@cloudshell:~ (qwiklabs-gcp-02-f5adeb762d87)$ 
```

