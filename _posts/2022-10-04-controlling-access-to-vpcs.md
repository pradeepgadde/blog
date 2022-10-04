---

layout: single
title:  "Controlling Access to VPC Networks"
date:   2022-10-04 12:59:04 +0530
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
# Control Access to VPCs

Create an nginx web server
Create tagged firewall rules
Create a service account with IAM roles
Explore permissions for the Network Admin and Security Admin roles



## Task 1. Create the web servers

Create two web servers (**blue** and **green**) in the **default** VPC network. Then install **nginx** on the web servers and modify the welcome page to distinguish the servers.

> Blue

```sh
gcloud compute instances create blue --project=qwiklabs-gcp-00-dbb9bd775ddb --zone=us-central1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=850707783529-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=web-server --create-disk=auto-delete=yes,boot=yes,device-name=blue,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-00-dbb9bd775ddb/zones/us-central1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Network tags (`--tags=webserver`)are used by networks to identify which VM instances are subject to certain firewall rules and network routes. In the next task, you create a firewall rule to allow HTTP access for VM instances with the **web-server** tag. Alternatively, you could select the **Allow HTTP traffic** checkbox, which would tag this instance as **http-server** and create the tagged firewall rule for tcp:80 for you.

> Green

```sh
gcloud compute instances create green --project=qwiklabs-gcp-00-dbb9bd775ddb --zone=us-central1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=850707783529-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=green,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-00-dbb9bd775ddb/zones/us-central1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-51.png)

> Blue

```sh
Linux blue 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-02-9bcbe9228694@blue:~$ sudo apt-get install nginx-light -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libnginx-mod-http-echo nginx-common
Suggested packages:
  fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  libnginx-mod-http-echo nginx-common nginx-light
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 726 kB of archives.
After this operation, 1744 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 nginx-common all 1.18.0-6.1+deb11u2 [126 kB]
Get:2 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-echo amd64 1.18.0-6.1+deb11u2 [109 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 nginx-light amd64 1.18.0-6.1+deb11u2 [492 kB]
Fetched 726 kB in 0s (3443 kB/s)   
Preconfiguring packages ...
Selecting previously unselected package nginx-common.
(Reading database ... 53755 files and directories currently installed.)
Preparing to unpack .../nginx-common_1.18.0-6.1+deb11u2_all.deb ...
Unpacking nginx-common (1.18.0-6.1+deb11u2) ...
Selecting previously unselected package libnginx-mod-http-echo.
Preparing to unpack .../libnginx-mod-http-echo_1.18.0-6.1+deb11u2_amd64.deb ...
Unpacking libnginx-mod-http-echo (1.18.0-6.1+deb11u2) ...
Selecting previously unselected package nginx-light.
Preparing to unpack .../nginx-light_1.18.0-6.1+deb11u2_amd64.deb ...
Unpacking nginx-light (1.18.0-6.1+deb11u2) ...
Setting up nginx-common (1.18.0-6.1+deb11u2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
Setting up libnginx-mod-http-echo (1.18.0-6.1+deb11u2) ...
Setting up nginx-light (1.18.0-6.1+deb11u2) ...
Upgrading binary: nginx.
Processing triggers for man-db (2.9.4-2) ...
student-02-9bcbe9228694@blue:~$ sudo nano /var/www/html/index.nginx-debian.html
student-02-9bcbe9228694@blue:~$ cat /var/www/html/index.nginx-debian.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Blue Server!</title>
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
student-02-9bcbe9228694@blue:~$ 
```

> Green

```sh
Linux green 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-02-9bcbe9228694@green:~$ sudo apt-get install nginx-light -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libnginx-mod-http-echo nginx-common
Suggested packages:
  fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  libnginx-mod-http-echo nginx-common nginx-light
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 726 kB of archives.
After this operation, 1744 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 nginx-common all 1.18.0-6.1+deb11u2 [126 kB]
Get:2 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-echo amd64 1.18.0-6.1+deb11u2 [109 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 nginx-light amd64 1.18.0-6.1+deb11u2 [492 kB]
Fetched 726 kB in 0s (4223 kB/s) 
Preconfiguring packages ...
Selecting previously unselected package nginx-common.
(Reading database ... 53755 files and directories currently installed.)
Preparing to unpack .../nginx-common_1.18.0-6.1+deb11u2_all.deb ...
Unpacking nginx-common (1.18.0-6.1+deb11u2) ...
Selecting previously unselected package libnginx-mod-http-echo.
Preparing to unpack .../libnginx-mod-http-echo_1.18.0-6.1+deb11u2_amd64.deb ...
Unpacking libnginx-mod-http-echo (1.18.0-6.1+deb11u2) ...
Selecting previously unselected package nginx-light.
Preparing to unpack .../nginx-light_1.18.0-6.1+deb11u2_amd64.deb ...
Unpacking nginx-light (1.18.0-6.1+deb11u2) ...
Setting up nginx-common (1.18.0-6.1+deb11u2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
Setting up libnginx-mod-http-echo (1.18.0-6.1+deb11u2) ...
Setting up nginx-light (1.18.0-6.1+deb11u2) ...
Upgrading binary: nginx.
Processing triggers for man-db (2.9.4-2) ...
student-02-9bcbe9228694@green:~$ sudo nano /var/www/html/index.nginx-debian.html
student-02-9bcbe9228694@green:~$ cat /var/www/html/index.nginx-debian.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to the Green Server!</title>
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
student-02-9bcbe9228694@green:~$
```

## Task 2. Create the firewall rule

Create the tagged firewall rule and test HTTP connectivity.



The **default-allow-internal** firewall rule allows traffic on all protocols/ports within the **default** network. You want to create a firewall rule to allow traffic from outside this network to only the **blue** server, by using the network tag **web-server**.

```sh
gcloud compute --project=qwiklabs-gcp-00-dbb9bd775ddb firewall-rules create allow-http-web-server --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=web-server
```

### Create a test-vm

Create a **test-vm** instance using the `gcloud` command line.

```sh
student_02_9bcbe9228694@cloudshell:~ (qwiklabs-gcp-00-dbb9bd775ddb)$ gcloud compute instances create test-vm --machine-type=n1-standard-1 --subnet=default --zone=us-central1-a
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-dbb9bd775ddb/zones/us-central1-a/instances/test-vm].
NAME: test-vm
ZONE: us-central1-a
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE:
INTERNAL_IP: 10.128.0.4
EXTERNAL_IP: 35.226.243.156
STATUS: RUNNING
student_02_9bcbe9228694@cloudshell:~ (qwiklabs-gcp-00-dbb9bd775ddb)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-52.png)

### Test HTTP connectivity

From **test-vm** `curl` the internal and external IP addresses of **blue** and **green**.

```sh
Linux test-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-9bcbe9228694'.
student-02-9bcbe9228694@test-vm:~$ curl 10.128.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Blue Server!</title>
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
student-02-9bcbe9228694@test-vm:~$
```

```sh
student-02-9bcbe9228694@test-vm:~$ curl 10.128.0.3
<!DOCTYPE html>
<html>
<head>
<title>Welcome to the Green Server!</title>
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
student-02-9bcbe9228694@test-vm:~$
```

We can HTTP access both servers using their internal IP addresses. The connection on tcp:80 is allowed by the **default-allow-internal** firewall rule, because **test-vm** is on the same VPC network as the web servers (**default** network).

```sh
student-02-9bcbe9228694@test-vm:~$ curl 35.223.134.228
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Blue Server!</title>
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
student-02-9bcbe9228694@test-vm:~$
```
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-53.png)

```sh
student-02-9bcbe9228694@test-vm:~$ curl 34.122.119.161
^C
student-02-9bcbe9228694@test-vm:~$ 
```

As expected, we can only HTTP access the external IP address of the **blue** server, because the **allow-http-web-server** only applies to VM instances with the **web-server** tag.



## Task 3. Explore the Network and Security Admin roles

Cloud IAM lets you authorize who can take action on specific resources, which give you full control and visibility to manage cloud resources centrally. The following roles are used in conjunction with single-project networking to independently control administrative access to each VPC network:

- **Network Admin:** Permissions to create, modify, and delete networking resources, except for firewall rules and SSL certificates.
- **Security Admin:** Permissions to create, modify, and delete firewall rules and SSL certificates.

Explore these roles by applying them to a service account, which is a special Google account that belongs to your VM instance instead of to an individual end user. You will authorize **test-vm** to use the service account to demonstrate the permissions of the **Network Admin** and **Security Admin** roles, instead of creating a new user.



Currently, **test-vm** uses the [Compute Engine default service account](https://cloud.google.com/compute/docs/access/service-accounts#compute_engine_default_service_account), which is enabled on all instances created by the `gcloud` command-line tool and the Cloud Console.



```sh
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules list
ERROR: (gcloud.compute.firewall-rules.list) Some requests did not succeed:
 - Request had insufficient authentication scopes.

student-02-9bcbe9228694@test-vm:~$ 
```

```sh
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules delete allow-http-web-server
The following firewalls will be deleted:
 - [allow-http-web-server]

Do you want to continue (Y/n)?  y

ERROR: (gcloud.compute.firewall-rules.delete) Could not fetch resource:
 - Request had insufficient authentication scopes.

student-02-9bcbe9228694@test-vm:~$ 
```

The **Compute Engine default service account** does not have the right permissions to allow you to list or delete firewall rules. The same applies to other users who do not have the right roles.



### Create a service account

Create a service account and apply the **Network Admin** role.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-54.png)


An instance must be stopped to edit its service account.
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-55.png)

After changing the service account of the test-vm.



```sh
Linux test-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct  4 14:55:19 2022 from 35.235.244.33
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules list
NAME                    NETWORK  DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
allow-http-web-server   default  INGRESS    1000      tcp:80                              False
default-allow-icmp      default  INGRESS    65534     icmp                                False
default-allow-internal  default  INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
default-allow-rdp       default  INGRESS    65534     tcp:3389                            False
default-allow-ssh       default  INGRESS    65534     tcp:22                              False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student-02-9bcbe9228694@test-vm:~$ 
```



However,

```sh
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules delete allow-http-web-server
The following firewalls will be deleted:
 - [allow-http-web-server]

Do you want to continue (Y/n)?  y

ERROR: (gcloud.compute.firewall-rules.delete) Could not fetch resource:
 - Required 'compute.firewalls.delete' permission for 'projects/qwiklabs-gcp-00-dbb9bd775ddb/global/firewalls/allow-http-web-server'

student-02-9bcbe9228694@test-vm:~$ 
```

As expected, the **Network Admin** role has permissions to list but not modify/delete firewall rules.



### Update service account and verify permissions

After changing the role to security admin

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-56.png)


```sh
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules list
NAME                    NETWORK  DIRECTION  PRIORITY  ALLOW                         DENY  DISABLED
allow-http-web-server   default  INGRESS    1000      tcp:80                              False
default-allow-icmp      default  INGRESS    65534     icmp                                False
default-allow-internal  default  INGRESS    65534     tcp:0-65535,udp:0-65535,icmp        False
default-allow-rdp       default  INGRESS    65534     tcp:3389                            False
default-allow-ssh       default  INGRESS    65534     tcp:22                              False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student-02-9bcbe9228694@test-vm:~$ 
```



```sh
student-02-9bcbe9228694@test-vm:~$ gcloud compute firewall-rules delete allow-http-web-server
The following firewalls will be deleted:
 - [allow-http-web-server]

Do you want to continue (Y/n)?  y

Deleted [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-dbb9bd775ddb/global/firewalls/allow-http-web-server].
student-02-9bcbe9228694@test-vm:~$ 
```

This too worked, this time!

As expected, the **Security Admin** role has permissions to list and delete firewall rules.



Verify that you can no longer HTTP access the external IP of the **blue** server because you deleted the **allow-http-web-server** firewall rule.

```sh
Linux test-vm 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct  4 15:12:52 2022 from 35.235.244.32
student-02-9bcbe9228694@test-vm:~$ curl 35.223.134.228

student-02-9bcbe9228694@test-vm:~$ 
```

In this lab, we created two nginx web servers and controlled external HTTP access using a tagged firewall rule. Then we created a service account with first the **Network Admin** role and then the **Security Admin** role to explore the different permissions of these roles.

If your company has a security team that manages firewalls and SSL certificates and a networking team that manages the rest of the networking resources, grant the security team the **Security Admin** role and the networking team the **Network Admin** role.

