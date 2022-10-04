---

layout: single
title:  "Getting Started with Google Cloud"
date:   2022-10-02 07:59:04 +0530
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

# Google Cloud Fundamentals
## Introduction to Google Cloud
- Cloud Computing Overview
- IaaS and PaaS
- Google Cloud Network
- Secuirty
- Environmental Impact
- Open Source Ecosystems
- Pricing and Billing
## Resources and Access in the Cloud
- Google Cloud Resource Hierarchy
- IAM
- Service Accounts
- Cloud Identity
- Interacting with Google Cloud
- Cloud Marketplace:  Use Cloud Marketplace to quickly and easily deploy a LAMP stack on a Compute Engine instance. The Bitnami LAMP Stack provides a complete web development environment for Linux that can be launched in one click.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-1.png)

```sh
Linux lampstack-1-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
       ___ _ _                   _
      | _ |_) |_ _ _  __ _ _ __ (_)
      | _ \ |  _| ' \/ _` | '  \| |
      |___/_|\__|_|_|\__,_|_|_|_|_|
  
  *** Welcome to the LAMP packaged by Bitnami 8.0.23-1                     ***
  *** Documentation:  https://docs.bitnami.com/google/infrastructure/lamp/ ***
  ***                 https://docs.bitnami.com/google/                     ***
  *** Bitnami Forums: https://github.com/bitnami/vms/                      ***
Creating directory '/home/student-04-1c4e6cf5917c'.
student-04-1c4e6cf5917c@lampstack-1-vm:~$ cd /opt/bitnami/
student-04-1c4e6cf5917c@lampstack-1-vm:/opt/bitnami$ sudo sh -c 'echo "<?php phpinfo(); ?>" > apache2/htdocs/phpinfo.php'
student-04-1c4e6cf5917c@lampstack-1-vm:/opt/bitnami$ 
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-3.png)

## Virtual Machines and Networks in the Cloud

- Virtual Private Cloud (VPC) Networking
- Compute Engine
- Scaling Virtual Machines
- Important VPC Capabilities
- Cloud Load Balancing
- Cloud DNS 
- Cloud CDN
- Connecting networks to Google VPC

Google Cloud Virtual Private Cloud (VPC) provides networking functionality to Compute Engine virtual machine (VM) instances, Kubernetes Engine containers, and App Engine flexible environment. In other words, without a VPC network you cannot create VM instances, containers, or App Engine applications. Therefore, each Google Cloud project has a **default** network to get you started.

You can think of a VPC network as similar to a physical network, except that it is virtualized within Google Cloud. A VPC network is a global resource that consists of a list of regional virtual subnetworks (subnets) in data centers, all connected by a global wide area network (WAN). VPC networks are logically isolated from each other in Google Cloud.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-4.png)

Routes tell VM instances and the VPC network how to send traffic from an instance to a destination, either inside the network or outside Google Cloud. Each VPC network comes with some default routes to route traffic among its subnets and send traffic from eligible instances to the internet.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-6.png)

Each VPC network implements a distributed virtual firewall that you can configure. Firewall rules allow you to control which packets are allowed to travel to which destinations. Every VPC network has two implied firewall rules that block all incoming connections and allow all outgoing connections.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-7.png)

If we delete the virtual network, we cant create any VMs

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-8.png)



Create a new network

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-9.png)

Create a new VM and Verify that the **Internal IP** for the new instance was assigned from the IP address range for the subnet in **europe-west4** (10.164.0.0/20).



The **Internal IP** should be 10.164.0.2 because 10.164.0.1 is reserved for the gateway and you have not configured any other instances in that subnet.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-10.png)

The **External IP addresses** for VM instances are ephemeral. If an instance is stopped, any ephemeral external IP addresses assigned to the instance are released back into the general Compute Engine pool and become available for use by other projects. When a stopped instance is started again, a new ephemeral external IP address is assigned to the instance.



Alternatively, you can reserve a static external IP address, which assigns the address to your project indefinitely until you explicitly release it.

If we try to delete the virtual network while it is in use, we will get an error.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-11.png)



Deleted the VPC and VM and created again this time with a new name.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-12.png)

Total of two VMs in each region

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-13.png)

Explore the connectivity for the VM instances. Specifically, try to SSH to your VM instances using tcp:22, and ping both the internal and external IP addresses of your VM instances using ICMP. Then explore the effects of the firewall rules on connectivity by removing the firewall rules individually.

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.
64 bytes from 10.164.0.2: icmp_seq=1 ttl=64 time=103 ms
64 bytes from 10.164.0.2: icmp_seq=2 ttl=64 time=101 ms
64 bytes from 10.164.0.2: icmp_seq=3 ttl=64 time=101 ms

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 101.444/101.876/102.690/0.575 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

You can ping **mynet-eu-vm**'s internal IP because of the **allow-custom** firewall rule.

```sh
Linux mynet-us-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-d8fa2820dfbd'.
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.
64 bytes from 34.90.55.19: icmp_seq=1 ttl=51 time=104 ms
64 bytes from 34.90.55.19: icmp_seq=2 ttl=51 time=103 ms
64 bytes from 34.90.55.19: icmp_seq=3 ttl=51 time=103 ms

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 102.774/103.242/104.166/0.653 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$
```

**mynetwork-allow-icmp** is allowing the external connectivity.



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-14.png)

Delete the Firewall rule and verify connectivity

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.
64 bytes from 10.164.0.2: icmp_seq=1 ttl=64 time=111 ms
64 bytes from 10.164.0.2: icmp_seq=2 ttl=64 time=109 ms
64 bytes from 10.164.0.2: icmp_seq=3 ttl=64 time=109 ms

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 109.194/109.671/110.562/0.630 ms
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2042ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-15.png)

After deleting one more rule

```sh
student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 10.164.0.2
PING 10.164.0.2 (10.164.0.2) 56(84) bytes of data.

--- 10.164.0.2 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2037ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ ping -c 3 34.90.55.19
PING 34.90.55.19 (34.90.55.19) 56(84) bytes of data.

--- 34.90.55.19 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2054ms

student-01-d8fa2820dfbd@mynet-us-vm:~$ 
```

The **100% packet loss** indicates that you cannot ping **mynet-eu-vm**'s internal IP. This is expected because you deleted the **allow-custom** firewall rule!



Delete SSH rule also

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-17.png)

In this lab, we explored the default network along with its subnets, routes, and firewall rules. We deleted the default network and determined that we cannot create any VM instances without a VPC network. Thus, we created a new auto mode VPC network with subnets, routes, firewall rules, and two VM instances. Then we tested the connectivity for the VM instances and explored the effects of the firewall rules on connectivity.



## Storage in the Cloud

- Google Cloud storage options

- Cloud Storage

- Storae classes and data transfer

- Cloud SQL

- Cloud Spanner

- Firestore

- Cloud Bigtable

- Compare storage options 

- Getting started with Cloud Storage and Cloud SQL Lab

In this lab, 

- Create a Cloud Storage bucket and place an image into it.

- Create a Cloud SQL instance and configure it.

- Connect to the Cloud SQL instance from a web server.

- Use the image in the Cloud Storage bucket on a web page.

  

Task 2 Deploy a web server VM instance
In the GCP Console, on the Navigation menu (Navigation menu icon), click Compute Engine > VM instances.

Click Create Instance.

On the Create an Instance page, for Name, type bloghost

For Region and Zone, select the region and zone assigned by Qwiklabs.

For Machine type, accept the default.

For Boot disk, if the Image shown is not Debian GNU/Linux 11 (bullseye), click Change and select Debian GNU/Linux 11 (bullseye).

Leave the defaults for Identity and API access unmodified.

For Firewall, click Allow HTTP traffic.

Click Networking, disks, security, management, sole tenancy to open that section of the dialog.

Click Management to open that section of the dialog.

Scroll down to the Automation section, and enter the following script as the value for Startup script:

apt-get update
apt-get install apache2 php php-mysql -y
service apache2 restart
Copied!
Note: Be sure to supply that script as the value of the Startup script field. If you accidentally put it into another field, it won't be executed when the VM instance starts.
Leave the remaining settings as their defaults, and click Create.

Task 3. Create a Cloud Storage bucket using the gsutil command line
All Cloud Storage bucket names must be globally unique. To ensure that your bucket name is unique, these instructions will guide you to give your bucket the same name as your Cloud Platform project ID, which is also globally unique.

Cloud Storage buckets can be associated with either a region or a multi-region location: US, EU, or ASIA. In this activity, you associate your bucket with the multi-region closest to the region and zone that Qwiklabs or your instructor assigned you to.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-5cb6e22598de.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$ export LOCATION=ASIA
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$ gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
Creating gs://qwiklabs-gcp-03-5cb6e22598de/...
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$ gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
Copying gs://cloud-training/gcpfci/my-excellent-blog.png...
/ [1 files][  8.2 KiB/  8.2 KiB]
Operation completed over 1 objects/8.2 KiB.
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$ gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
Copying file://my-excellent-blog.png [Content-Type=image/png]...
/ [1 files][  8.2 KiB/  8.2 KiB]
Operation completed over 1 objects/8.2 KiB.
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$ gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
Updated ACL on gs://qwiklabs-gcp-03-5cb6e22598de/my-excellent-blog.png
student_03_1a8f2ec3df60@cloudshell:~ (qwiklabs-gcp-03-5cb6e22598de)$
```

`mb` Enter this command to make a bucket 

1. Retrieved a banner image from a publicly accessible Cloud Storage location
2. Copied the banner image to our newly created Cloud Storage bucket
3. Modified the Access Control List of the object we just created so that it is readable by everyone

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-20.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-23.png)



```sh

individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-03-1a8f2ec3df60'.
student-03-1a8f2ec3df60@bloghost:~$ cd /var/www/html
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano index..php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano inx.php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano index.php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo service apache2 restart
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano index.php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo service apache2 restart
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano index.php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo service apache2 restart
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo nano index.php
student-03-1a8f2ec3df60@bloghost:/var/www/html$ sudo service apache2 restart
student-03-1a8f2ec3df60@bloghost:/var/www/html$ cat index.php 
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<img src='https://storage.googleapis.com/qwiklabs-gcp-03-5cb6e22598de/my-excellent-blog.png'>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver ="34.170.7.1";
$dbuser = "bloguser";
$dbpassword = "GCP2022";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.
$conn = new mysqli($dbserver, $dbuser, $dbpassword);
if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>
student-03-1a8f2ec3df60@bloghost:/var/www/html$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-24.png)

In this lab, we configured a Cloud SQL instance and connected an application in a Compute Engine instance to it. We also worked with a Cloud Storage bucket.

## Containers in the Cloud

- Introduction to containers
- Kubernetes
- Google Kubernetes Engine
- Hybrid and multi-cloud
- Anthos
- Getting Started with GKE Lab

## Applications in the Cloud

- App Engine
- App Engine evrionments
- Google Cloud API management tool
- Cloud Run
- Hello Cloud Run Lab

## Developing and Deploying the Cloud

- Development in the cloud
- Infrastructure as Code
- Terraform Lab

## Logging and Monitoring in the Cloud

- Importance of monitoring

- Measuring performance and reliability

- SLI, SLO, and SLA

- Integrated observability tools

- Monitoring tools

- Logging tools

- Error reporting and debugging tools

  