---

layout: single
title:  "Configuring an HTTP Load Balancer with Google Cloud Armor"
date:   2022-10-05 05:59:04 +0530
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


# Configuring an HTTP Load Balancer with Google Cloud Armor

Google Cloud HTTP(S) load balancing is implemented at the edge of Google's network in Google's points of presence (POP) around the world. User traffic directed to an HTTP(S) load balancer enters the POP closest to the user and is then load-balanced over Google's global network to the closest backend that has sufficient capacity available.

Google Cloud Armor IP deny/allow rules enable you to restrict or allow access to your HTTP(S) load balancer at the edge of the Google Cloud, as close as possible to the user and to malicious traffic. This prevents malicious users or traffic from consuming resources or entering your virtual private cloud (VPC) networks.



- Create a health check firewall rule
- Create two regional NAT configurations using Cloud Router
- Configure two instance templates
- Create two managed instance groups
- Configure an HTTP load balancer with IPv4 and IPv6
- Stress test an HTTP load balancer
- Deny an IP address to restrict access to an HTTP load balancer

## Task 1. Configure a health check firewall rule

Health checks determine which instances of a load balancer can receive new connections. For HTTP load balancing, the health check probes to your load-balanced instances come from addresses in the ranges **130.211.0.0/22** and **35.191.0.0/16**. Your firewall rules must allow these connections.



## Task 2: Create two NAT configurations using Cloud Router

The Google Cloud VM backend instances that we setup in Task 3 will not be configured with external IP addresses.

Instead, we will setup the Cloud NAT service to allow these VM instances to make outbound requests in order to install Apache Web server and PHP when they are launched. We create a Cloud Router for each managed instance group, one in **us-central1** and one in the **europe-west1** region, which we configure in the next task.



### Create the Cloud Router instance

Cloud NAT lets your Compute Engine instances and Kubernetes Engine container pods communicate with the internet using a shared, public IP address. Cloud NAT uses a Cloud NAT gateway to connect your subnets to a Cloud Router, a virtual router that connects to the internet.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-63.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-64.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-65.png)



## Task 3. Configure an instance template and create instance groups

A managed instance group uses an instance template to create a group of identical instances. Use these to create the backends of the HTTP load balancer.

### **Configure the instance template**

An instance template is an API resource that you can use to create VM instances and managed instance groups. Instance templates define the machine type, boot disk image, subnet, labels, and other instance properties.



Managed instance groups offer **autoscaling** capabilities that allow you to automatically add or remove instances from a managed instance group based on increases or decreases in load. Autoscaling helps your applications gracefully handle increases in traffic and reduces cost when the need for resources is lower. You just define the autoscaling policy, and the autoscaler performs automatic scaling based on the measured load.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-66.png)

### **Verify the backends**

Verify that VM instances are being created in both regions and access their HTTP sites.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-67.png)



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-347e3ad27384.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-03-347e3ad27384)$ gcloud compute ssh europe-west1-mig-ndf8 --zone europe-west1-b
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_01_da96e1e7e410/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student_01_da96e1e7e410/.ssh/google_compute_engine
Your public key has been saved in /home/student_01_da96e1e7e410/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:5cb+dV8sFr79QyQefkmZPvVdhEo3rM2W8hGK/F01sRM student_01_da96e1e7e410@cs-993619767812-default
The key's randomart image is:
+---[RSA 3072]----+
|             . E.|
|            . *.=|
|         ..o B B=|
|         +o =o*==|
|        S +.o=Bo*|
|         o  .+oOo|
|          .   B =|
|           . o *o|
|            . . *|
+----[SHA256]-----+
External IP address was not found; defaulting to using IAP tunneling.
WARNING:

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.4098487783932359400' (ECDSA) to the list of known hosts.
Linux europe-west1-mig-ndf8 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-da96e1e7e410'.
student-01-da96e1e7e410@europe-west1-mig-ndf8:~$
```



```sh
student-01-da96e1e7e410@europe-west1-mig-ndf8:~$ curl localhost
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : ::1<h2>Hostname</h2>Server Hostname: europe-west1-mig-ndf8<h2>Server Location</h2>Region and Zone: europe-west1-bstudent-01-da96e1e7e410@europe-west1-mig-ndf8:~$
```

```sh
student-01-da96e1e7e410@europe-west1-mig-ndf8:~$ curl 10.128.0.2
<h1>HTTP Load Balancing Lab</h1><h2>Client IP</h2>Your IP address : 10.132.0.2<h2>Hostname</h2>Server Hostname: us-central1-mig-fsz6<h2>Server Location</h2>Region and Zone: us-central1-fstudent-01-da96e1e7e410@europe-west1-mig-ndf8:~$
```

## Task 4. Configure the HTTP load balancer

Configure the HTTP load balancer to balance traffic between the two backends (**us-central1-mig** in us-central1 and **europe-west1-mig** in europe-west1).

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-68.png)



## Task 5. Test the HTTP load balancer

Now that we have created the HTTP load balancer for our backends, it is time to verify that traffic is forwarded to the backend service.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-69.png)

### **Access the HTTP load balancer**

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-70.png)

Depending on our proximity to **us-central1** and **europe-west1**, our traffic is either forwarded to a **us-central1-mig** or **europe-west1-mig** instance.



### **Stress test the HTTP load balancer**

Create a new VM to simulate a load on the HTTP load balancer. Then determine whether traffic is balanced across both backends when the load is high.



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-347e3ad27384.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-03-347e3ad27384)$ gcloud compute ssh siege-vm --zone us-west1-c
Warning: Permanently added 'compute.1251821788431707640' (ECDSA) to the list of known hosts.
Linux siege-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-01-da96e1e7e410@siege-vm:~$ sudo apt-get -y install siege
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  siege
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 105 kB of archives.
After this operation, 286 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 siege amd64 4.0.7-1 [105 kB]
Fetched 105 kB in 0s (1822 kB/s)
Selecting previously unselected package siege.
(Reading database ... 53755 files and directories currently installed.)
Preparing to unpack .../siege_4.0.7-1_amd64.deb ...
Unpacking siege (4.0.7-1) ...
Setting up siege (4.0.7-1) ...
Processing triggers for man-db (2.9.4-2) ...
student-01-da96e1e7e410@siege-vm:~$ export LB_IP=34.160.243.189
student-01-da96e1e7e410@siege-vm:~$ echo $LB_IP
34.160.243.189
student-01-da96e1e7e410@siege-vm:~$ siege -c 250 http://$LB_IP
New configuration template added to /home/student-01-da96e1e7e410/.siege
Run siege -C to view the current settings in that file
^C
{       "transactions":                         3896,
        "availability":                       100.00,
        "elapsed_time":                       108.18,
        "data_transferred":                     0.60,
        "response_time":                        6.64,
        "transaction_rate":                    36.01,
        "throughput":                           0.01,
        "concurrency":                        238.96,
        "successful_transactions":              3896,
        "failed_transactions":                     0,
        "longest_transaction":                 13.20,
        "shortest_transaction":                 0.03
}
student-01-da96e1e7e410@siege-vm:~$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-71.png)



```sh
student-01-da96e1e7e410@siege-vm:~$ curl http://$LB_IP
<!doctype html><meta charset="utf-8"><meta name=viewport content="width=device-width, initial-scale=1"><title>403</title>403 Forbiddenstudent-01-da96e1e7e410@siege-vm:~$
```



We can access the HTTP load balancer from our browser because of the default rule to **allow** traffic; however, we cannot access it from the **siege-vm** because of the **deny** rule that we implemented.

Google Cloud Armor security policies create logs that can be explored to determine when traffic is denied and when it is allowed, along with the source of the traffic.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-72.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-73.png)

In this lab, we configured an HTTP load balancer with backends in us-central1 and europe-west1. Then we stress-tested the load balancer with a VM and denied the IP address of that VM with Google Cloud Armor. We were able to explore the security policy logs to identify why the traffic was blocked.