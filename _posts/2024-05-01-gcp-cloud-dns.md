---

layout: single
title:  "Cloud DNS - Traffic Steering using Geolocation Policy"
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

# Cloud DNS - Traffic Steering using Geolocation Policy

Cloud DNS routing policies enable users to configure DNS based  traffic steering. A user can either create a Weighted Round Robin (WRR)  routing policy or a Geolocation (GEO) routing policy.  You can configure routing policies by creating special ResourceRecordSets with special  routing policy values.

Use WRR to specify different weights per ResourceRecordSet for the  resolution of domain names. Cloud DNS routing policies help ensure that  traffic is distributed across multiple IP addresses by resolving DNS  requests according to the configured weights.

In this lab, you will configure and test the Geolocation routing  policy. Use GEO to specify source geolocations and to provide DNS  answers corresponding to those geographies. The geolocation routing  policy applies the nearest match for the source location when the  traffic source location doesn't match any policy items exactly

Use the default VPC network to create all the virtual machines (VM) and  launch client VMs in 3 Google Cloud locations: one in the United States, another in Europe, and another in Asia. To demonstrate the Geolocation  routing policy behavior, you will create the server VMs only in two of  those location - in the United States and in Europe. 

You will use Cloud DNS routing policies and create `ResourceRecordSets` for geo.example.com and configure the Geolocation policy to help ensure that a client request is routed to a server in the client's closest  region.

## Enable APIs

Ensure that the Compute and the Cloud DNS APIs are enabled. In this section, you will enable the APIs manually, using `gcloud` commands.

### Enable Compute Engine API

- Run the `gcloud services enable` command to enable the Compute Engine API:

### Enable Cloud DNS API

- Run the `gcloud services enable` command to enable the Cloud DNS API:

```sh
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-6940d13456e9.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud services enable compute.googleapis.com
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud services enable dns.googleapis.com
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud services list | grep -E 'compute|dns'
NAME: compute.googleapis.com
NAME: dns.googleapis.com
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Configure the firewall

Before you create the client VMs and the  web servers, you need to create two firewall rules.

To be able to SSH into the client VMs, run the following to create a  firewall rule to allow SSH traffic from Identity Aware Proxies (IAP):

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute firewall-rules create fw-default-iapproxy \
--direction=INGRESS \
--priority=1000 \
--network=default \
--action=ALLOW \
--rules=tcp:22,icmp \
--source-ranges=35.235.240.0/20
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/global/firewalls/fw-default-iapproxy].                           
Creating firewall...done.                                                                                                                                                          
NAME: fw-default-iapproxy
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,icmp
DENY: 
DISABLED: False
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

To allow HTTP traffic on the web servers, each web server will have a  "http-server" tag associated with it. You will use this tag to apply the firewall rule only to your web servers:

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute firewall-rules create allow-http-traffic --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/global/firewalls/allow-http-traffic].                            
Creating firewall...done.                                                                                                                                                          
NAME: allow-http-traffic
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Launch client VMs

Now that the APIs are enabled, and the firewall rules are in place,  the next step is to set up the environment. In this section, you will  create 3 client VMs, one in each region.

### Launch a client in the United States

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute instances create us-client-vm --machine-type=e2-micro --zone us-west1-c
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/us-west1-c/instances/us-client-vm].
NAME: us-client-vm
ZONE: us-west1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.138.0.2
EXTERNAL_IP: 34.82.141.9
STATUS: RUNNING
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

### Launch a client in Europe

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute instances create europe-client-vm --machine-type=e2-micro --zone "europe-west1-b"
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/europe-west1-b/instances/europe-client-vm].
NAME: europe-client-vm
ZONE: europe-west1-b
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 35.195.177.0
STATUS: RUNNING
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

### Launch a client in Asia

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute instances create asia-client-vm --machine-type=e2-micro --zone asia-south1-a
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/asia-south1-a/instances/asia-client-vm].
NAME: asia-client-vm
ZONE: asia-south1-a
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.160.0.2
EXTERNAL_IP: 34.93.99.203
STATUS: RUNNING
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Launch Server VMs

Now that the client VM's are up and running, the next step is to  create the server VMs. You will use a startup script to configure and  set up the web servers.  As mentioned earlier, you will create the  server VMs only in 2 regions: us-east1 and europe-west2.

### Launch server in the United States

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute instances create us-web-vm \
--machine-type=e2-micro \
--zone us-west1-c \
--network=default \
--subnet=default \
--tags=http-server \
--metadata=startup-script='#! /bin/bash
 apt-get update
 apt-get install apache2 -y
 echo "Page served from: us-west1" | \
 tee /var/www/html/index.html
 systemctl restart apache2'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/us-west1-c/instances/us-web-vm].
NAME: us-web-vm
ZONE: us-west1-c
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.138.0.3
EXTERNAL_IP: 35.247.60.194
STATUS: RUNNING
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

### Launch server in Europe

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute instances create europe-web-vm \
--machine-type=e2-micro \
--zone=europe-west1-b \
--network=default \
--subnet=default \
--tags=http-server \
--metadata=startup-script='#! /bin/bash
 apt-get update
 apt-get install apache2 -y
 echo "Page served from: europe-west1" | \
 tee /var/www/html/index.html
 systemctl restart apache2'
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/europe-west1-b/instances/europe-web-vm].
NAME: europe-web-vm
ZONE: europe-west1-b
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.3
EXTERNAL_IP: 35.205.206.78
STATUS: RUNNING
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Setting up environment variables

Before you configure Cloud DNS, note the Internal IP addresses of the web servers. You need these IPs to create the routing policy. In this  section, you will use the `gcloud compute instances describe` command to save the internal IP addresses as environment variables.

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ export US_WEB_IP=$(gcloud compute instances describe us-web-vm --zone=us-west1-c --format="value(networkInterfaces.networkIP)")
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ export EUROPE_WEB_IP=$(gcloud compute instances describe europe-web-vm --zone=europe-west1-b --format="value(networkInterfaces.networkIP)")
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ echo $US_WEB_IP
10.138.0.3
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ echo $EUROPE_WEB_IP
10.132.0.3
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Create the private zone

Now that your client and server VMs are running, it's time to  configure the DNS settings. Before creating the A records for the web  servers, you need to create the Cloud DNS Private Zone.

For this lab, use the `example.com` domain name for the Cloud DNS zone.

- Use the `gcloud dns managed-zones create` command to create the zone:

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud dns managed-zones create example --description=test --dns-name=example.com --networks=default --visibility=private
Created [https://dns.googleapis.com/dns/v1/projects/qwiklabs-gcp-04-6940d13456e9/managedZones/example].
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

## Create Cloud DNS Routing Policy

In this section, configure the Cloud DNS Geolocation Routing Policy. You will create a record set in the `example.com` zone that you created in the previous section.

### Create

- Use the `gcloud dns record-sets create` command to create the geo.example.com recordset:

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud dns record-sets create geo.example.com \
--ttl=5 --type=A --zone=example \
--routing-policy-type=GEO \
--routing-policy-data="us-west1=$US_WEB_IP;europe-west1=$EUROPE_WEB_IP"
NAME: geo.example.com.
TYPE: A
TTL: 5
DATA: us-west1: 10.138.0.3; europe-west1: 10.132.0.3
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

You are creating an A record with a Time to Live (TTL) of 5 seconds. The policy type is GEO, and the `routing_policy_data` field accepts a semicolon-delimited list of the format `${region}:${rrdata},${rrdata}`.

### Verify

- Use the `dns record-sets list` command to verify that the `geo.example.com` DNS record is configured as expected:

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud dns record-sets list --zone=example
NAME: example.com.
TYPE: NS
TTL: 21600
DATA: ns-gcp-private.googledomains.com.

NAME: example.com.
TYPE: SOA
TTL: 21600
DATA: ns-gcp-private.googledomains.com. cloud-dns-hostmaster.google.com. 1 21600 3600 259200 300

NAME: geo.example.com.
TYPE: A
TTL: 5
DATA: us-west1: 10.138.0.3; europe-west1: 10.132.0.3
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

The output shows that an A record with a TTL of 5 is created for `geo.example.com`, and the data matches our server set up in each region.

## Testing

It's time to test the configuration. In this section, you will SSH  into all the client VMs. Since all of the web server VMs are behind the `geo.example.com` domain, you will use `CURL` command to access this endpoint.

Since you are using a Geolocation policy, the expected result is that:

- The client in the US should always get a response from the us-west1-c region.
- The client in Europe should always get a response from the  europe-west1-b region.

### Testing from the client VM in Europe

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute ssh europe-client-vm --zone europe-west1-b --tunnel-through-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_00_1f149e03ac6d/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_00_1f149e03ac6d/.ssh/google_compute_engine
Your public key has been saved in /home/student_00_1f149e03ac6d/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:u/atHlKp4XB3BNDWle9gWWIKah8JFtICDwAJtTZCMZc student_00_1f149e03ac6d@cs-361375827713-default
The key's randomart image is:
+---[RSA 3072]----+
|+B+o+...o+.. ... |
|o oE o..+ +.. + .|
|. +   .o + o.o = |
| o .    o oo. + .|
|      ..S.+... o |
|       + *..    .|
|        = .      |
|        .o o     |
|       ..o+..    |
+----[SHA256]-----+
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.1285740987626245118' (ED25519) to the list of known hosts.
Linux europe-client-vm 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-00-1f149e03ac6d'.
student-00-1f149e03ac6d@europe-client-vm:~$ 
student-00-1f149e03ac6d@europe-client-vm:~$ for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done
1
Page served from: europe-west1
2
Page served from: europe-west1
3
Page served from: europe-west1
4
Page served from: europe-west1
5
Page served from: europe-west1
6
Page served from: europe-west1
7
Page served from: europe-west1
8
Page served from: europe-west1
9
Page served from: europe-west1
10
Page served from: europe-west1
student-00-1f149e03ac6d@europe-client-vm:~$ 
```

Since the TTL on the DNS record is set to 5 seconds, a sleep timer of 6  seconds has been added. The sleep timer will make sure that you get an  uncached DNS response for each cURL request. Analyze the output to see which server is responding to the request. The client should always receive a response from a server in the client's  region.

### Testing from the client VM in us-east1

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute ssh us-client-vm --zone us-west1-c --tunnel-through-iap
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.8916973303598369291' (ED25519) to the list of known hosts.
Linux us-client-vm 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-00-1f149e03ac6d'.
student-00-1f149e03ac6d@us-client-vm:~$ for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done
1
Page served from: us-west1
2
Page served from: us-west1
3
Page served from: us-west1
4
Page served from: us-west1
5
Page served from: us-west1
6
Page served from: us-west1
7
Page served from: us-west1
8
Page served from: us-west1
9
Page served from: us-west1
10
Page served from: us-west1
student-00-1f149e03ac6d@us-client-vm:~$ 
```

Now analyze the output to see which server is responding to the request. The client should always receive a response from a server in the  client's region. The expected output is "Page served from: us-west1".

### Testing from  the client VM in Asia

So far you have tested the setup from the United States and Europe.  You have servers running in both the regions and have matching record  sets for both the regions in Cloud DNS routing policy. There is no  matching policy item for the region within Asia (selected earlier) in  the Cloud DNS routing policy.

The Geolocation policy will apply a "nearest" match for source  location when the source of the traffic doesn't match any policy items  exactly. This means that the Asia client should be directed to the  nearest web server.

In this section, you will resolve the `geo.example.com` domain from the client VM in Asia and will analyze the response.

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ gcloud compute ssh asia-client-vm --zone asia-south1-a --tunnel-through-iap
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.3259574679737682849' (ED25519) to the list of known hosts.
Linux asia-client-vm 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-00-1f149e03ac6d'.
student-00-1f149e03ac6d@asia-client-vm:~$ for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done
1
Page served from: us-west1
2
Page served from: us-west1
3
Page served from: us-west1
4
Page served from: us-west1
5
Page served from: us-west1
6
Page served from: us-west1
7
Page served from: us-west1
8
Page served from: us-west1
9
Page served from: us-west1
10
Page served from: us-west1
student-00-1f149e03ac6d@asia-client-vm:~$ 
```

Analyze the output to see which server is responding to the request.  Since there is no policy item for any of the Asia regions, Cloud DNS  will direct the client to the nearest server.

## Delete lab resources

```sh
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ #delete VMS
gcloud compute instances delete -q us-client-vm --zone us-west1-c

gcloud compute instances delete -q us-web-vm --zone us-west1-c

gcloud compute instances delete -q europe-client-vm --zone europe-west1-b

gcloud compute instances delete -q europe-web-vm --zone europe-west1-b

gcloud compute instances delete -q asia-client-vm --zone asia-south1-a

#delete FW rules
gcloud compute firewall-rules delete -q allow-http-traffic

gcloud dns managed-zones delete examplele.com --type=A --zone=example
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/us-west1-c/instances/us-client-vm].
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/us-west1-c/instances/us-web-vm].
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/europe-west1-b/instances/europe-client-vm].
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/europe-west1-b/instances/europe-web-vm].
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/zones/asia-south1-a/instances/asia-client-vm].
Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/global/firewalls/allow-http-traffic].
The following firewalls will be deleted:
 - [fw-default-iapproxy]

Do you want to continue (Y/n)?  y

Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-04-6940d13456e9/global/firewalls/fw-default-iapproxy].
Deleted [https://dns.googleapis.com/dns/v1/projects/qwiklabs-gcp-04-6940d13456e9/managedZones/example/rrsets/geo.example.com/A].
Deleted [https://dns.googleapis.com/dns/v1/projects/qwiklabs-gcp-04-6940d13456e9/managedZones/example].
student_00_1f149e03ac6d@cloudshell:~ (qwiklabs-gcp-04-6940d13456e9)$ 
```

In this lab, you configured and used Cloud DNS routing policies with  Geolocation routing policy. You also verified the configuration and  behavior of the Cloud DNS routing policy by observing the HTTP response  when accessing the web servers.