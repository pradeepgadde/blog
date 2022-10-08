---

layout: single
title:  "GCP—Analyzing network traffic with VPC Flow Logs"
date:   2022-10-08 04:59:04 +0530
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
  - title: "Topics"
    nav: my-sidebar

---

# Analyzing network traffic with VPC Flow Logs

In this lab, you configure a network to record traffic to and from an Apache web server using VPC Flow Logs. You then export the logs to BigQuery to analyze them.

- Configure a custom network with VPC Flow Logs

- Create an Apache web server

- Verify that network traffic is logged

- Export the network traffic to BigQuery to further analyze the logs

- Setup VPC flow log aggregation



## Task 1. Configure a custom network with VPC Flow Logs
Create the custom network
```sh
gcloud compute networks create vpc-net --project=qwiklabs-gcp-02-91a00c91652c --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional
```

```sh
gcloud compute networks subnets create vpc-subnet --project=qwiklabs-gcp-02-91a00c91652c --range=10.1.3.0/24 --stack-type=IPV4_ONLY --network=vpc-net --region=us-central1 --enable-flow-logs --logging-aggregation-interval=interval-5-sec --logging-flow-sampling=0.5 --logging-metadata=include-all
```

Turning on VPC flow logs doesn't affect performance, but some systems generate a large number of logs, which can increase costs. If you click on Configure logs you'll notice that you can modify the aggregation interval and sample rate. This allows you to trade off longer interval updates for lower data volume generation which lowers logging costs. 

Create the firewall rule
In order to serve HTTP and SSH traffic on the network, you need to create a firewall rule.


```sh
gcloud compute --project=qwiklabs-gcp-02-91a00c91652c firewall-rules create allow-http-ssh --direction=INGRESS --priority=1000 --network=vpc-net --action=ALLOW --rules=tcp:22,tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```

## Task 2. Create an Apache web server
Create the web server

```sh
gcloud compute instances create web-server --project=qwiklabs-gcp-02-91a00c91652c --zone=us-central1-c --machine-type=f1-micro --network-interface=network-tier=PREMIUM,subnet=vpc-subnet --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=361062432930-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=web-server,image=projects/debian-cloud/global/images/debian-11-bullseye-v20220920,mode=rw,size=10,type=projects/qwiklabs-gcp-02-91a00c91652c/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

Install Apache
Configure the VM instance that you created as an Apache web server, and overwrite the default web page

```sh
Linux web-server 5.10.0-18-cloud-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-01-e3c2b9ab4419@web-server:~$ sudo apt-get update
Get:1 http://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Hit:2 http://deb.debian.org/debian bullseye InRelease                                                         
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]           
Get:4 http://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:5 http://security.debian.org/debian-security bullseye-security/main Sources [160 kB]
Get:6 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [189 kB]
Get:7 http://security.debian.org/debian-security bullseye-security/main Translation-en [119 kB]
Get:8 http://packages.cloud.google.com/apt cloud-sdk-bullseye InRelease [6781 B]    
Get:9 http://deb.debian.org/debian bullseye-updates/main Sources.diff/Index [11.7 kB]
Get:10 http://deb.debian.org/debian bullseye-updates/main amd64 Packages.diff/Index [11.7 kB]
Get:11 http://deb.debian.org/debian bullseye-updates/main Translation-en.diff/Index [4995 B]
Get:12 http://deb.debian.org/debian bullseye-updates/main Sources T-2022-10-02-2032.44-F-2022-09-22-1635.40.pdiff [2486 B]
Get:13 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2022-10-02-2032.44-F-2022-09-22-1635.40.pdiff [4713 B]
Get:12 http://deb.debian.org/debian bullseye-updates/main Sources T-2022-10-02-2032.44-F-2022-09-22-1635.40.pdiff [2486 B]
Get:13 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2022-10-02-2032.44-F-2022-09-22-1635.40.pdiff [4713 B]
Get:14 http://deb.debian.org/debian bullseye-backports/main Sources.diff/Index [63.3 kB]
Get:15 http://deb.debian.org/debian bullseye-backports/main amd64 Packages.diff/Index [63.3 kB]
Get:16 http://deb.debian.org/debian bullseye-backports/main Translation-en.diff/Index [63.3 kB]
Get:17 http://deb.debian.org/debian bullseye-updates/main Translation-en T-2022-09-22-1635.40-F-2022-09-22-1635.40.pdiff [3855 B]
Get:17 http://deb.debian.org/debian bullseye-updates/main Translation-en T-2022-09-22-1635.40-F-2022-09-22-1635.40.pdiff [3855 B]
Get:18 http://deb.debian.org/debian bullseye-backports/main Sources T-2022-10-08-0805.07-F-2022-09-22-0435.24.pdiff [61.0 kB]
Get:19 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2022-10-08-0805.07-F-2022-09-22-1115.32.pdiff [67.6 kB]
Get:18 http://deb.debian.org/debian bullseye-backports/main Sources T-2022-10-08-0805.07-F-2022-09-22-0435.24.pdiff [61.0 kB]
Get:19 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2022-10-08-0805.07-F-2022-09-22-1115.32.pdiff [67.6 kB]
Get:20 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2022-10-07-1403.51-F-2022-09-22-1115.32.pdiff [28.2 kB]
Get:20 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2022-10-07-1403.51-F-2022-09-22-1115.32.pdiff [28.2 kB]
Get:21 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-bullseye InRelease [5557 B]
Get:22 http://packages.cloud.google.com/apt google-compute-engine-bullseye-stable InRelease [5533 B]
Get:23 http://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 Packages [185 kB]
Get:24 http://packages.cloud.google.com/apt google-cloud-packages-archive-keyring-bullseye/main amd64 Packages [387 B]
Fetched 1198 kB in 1s (843 kB/s)                     
Reading package lists... Done
student-01-e3c2b9ab4419@web-server:~$ sudo apt-get install apache2 -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils bzip2 file libapr1 libaprutil1 libaprutil1-dbd-sqlite3
  libaprutil1-ldap libgdbm-compat4 libicu67 libjansson4 liblua5.3-0 libmagic-mgc libmagic1 libperl5.32
  libxml2 mailcap mime-support perl perl-modules-5.32 ssl-cert
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser bzip2-doc perl-doc
  libterm-readline-gnu-perl | libterm-readline-perl-perl make libtap-harness-archive-perl
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils bzip2 file libapr1 libaprutil1 libaprutil1-dbd-sqlite3
  libaprutil1-ldap libgdbm-compat4 libicu67 libjansson4 liblua5.3-0 libmagic-mgc libmagic1 libperl5.32
  libxml2 mailcap mime-support perl perl-modules-5.32 ssl-cert
0 upgraded, 23 newly installed, 0 to remove and 12 not upgraded.
Need to get 19.7 MB of archives.
After this operation, 98.9 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 perl-modules-5.32 all 5.32.1-4+deb11u2 [2823 kB]
Get:2 http://deb.debian.org/debian bullseye/main amd64 libgdbm-compat4 amd64 1.19-2 [44.7 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 libperl5.32 amd64 5.32.1-4+deb11u2 [4106 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 perl amd64 5.32.1-4+deb11u2 [293 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 libapr1 amd64 1.7.0-6+deb11u1 [106 kB]
Get:6 http://deb.debian.org/debian bullseye/main amd64 libaprutil1 amd64 1.6.1-5 [92.1 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-5 [18.8 kB]
Get:8 http://deb.debian.org/debian bullseye/main amd64 libaprutil1-ldap amd64 1.6.1-5 [17.0 kB]
Get:9 http://deb.debian.org/debian bullseye/main amd64 libjansson4 amd64 2.13.1-1.1 [39.7 kB]
Get:10 http://deb.debian.org/debian bullseye/main amd64 liblua5.3-0 amd64 5.3.3-1.1+b1 [120 kB]
Get:11 http://deb.debian.org/debian bullseye/main amd64 libicu67 amd64 67.1-7 [8622 kB]
Get:12 http://deb.debian.org/debian bullseye/main amd64 libxml2 amd64 2.9.10+dfsg-6.7+deb11u2 [692 kB]
Get:13 http://deb.debian.org/debian bullseye/main amd64 apache2-bin amd64 2.4.54-1~deb11u1 [1425 kB]
Get:14 http://deb.debian.org/debian bullseye/main amd64 apache2-data all 2.4.54-1~deb11u1 [160 kB]
Get:15 http://deb.debian.org/debian bullseye/main amd64 apache2-utils amd64 2.4.54-1~deb11u1 [260 kB]
Get:16 http://deb.debian.org/debian bullseye/main amd64 mailcap all 3.69 [31.7 kB]
Get:17 http://deb.debian.org/debian bullseye/main amd64 mime-support all 3.66 [10.9 kB]
Get:18 http://deb.debian.org/debian bullseye/main amd64 apache2 amd64 2.4.54-1~deb11u1 [275 kB]
Get:19 http://deb.debian.org/debian bullseye/main amd64 bzip2 amd64 1.0.8-4 [49.3 kB]
Get:20 http://deb.debian.org/debian bullseye/main amd64 libmagic-mgc amd64 1:5.39-3 [273 kB]
Get:21 http://deb.debian.org/debian bullseye/main amd64 libmagic1 amd64 1:5.39-3 [126 kB]
Get:22 http://deb.debian.org/debian bullseye/main amd64 file amd64 1:5.39-3 [69.1 kB]
Get:23 http://deb.debian.org/debian bullseye/main amd64 ssl-cert all 1.1.0+nmu1 [21.0 kB]
Fetched 19.7 MB in 1s (26.2 MB/s)
Preconfiguring packages ...
Selecting previously unselected package perl-modules-5.32.
(Reading database ... 53755 files and directories currently installed.)
Preparing to unpack .../00-perl-modules-5.32_5.32.1-4+deb11u2_all.deb ...
Unpacking perl-modules-5.32 (5.32.1-4+deb11u2) ...
Selecting previously unselected package libgdbm-compat4:amd64.
Preparing to unpack .../01-libgdbm-compat4_1.19-2_amd64.deb ...
Unpacking libgdbm-compat4:amd64 (1.19-2) ...
Selecting previously unselected package libperl5.32:amd64.
Preparing to unpack .../02-libperl5.32_5.32.1-4+deb11u2_amd64.deb ...
Unpacking libperl5.32:amd64 (5.32.1-4+deb11u2) ...
Selecting previously unselected package perl.
Preparing to unpack .../03-perl_5.32.1-4+deb11u2_amd64.deb ...
Unpacking perl (5.32.1-4+deb11u2) ...
Selecting previously unselected package libapr1:amd64.
Preparing to unpack .../04-libapr1_1.7.0-6+deb11u1_amd64.deb ...
Unpacking libapr1:amd64 (1.7.0-6+deb11u1) ...
Selecting previously unselected package libaprutil1:amd64.
Preparing to unpack .../05-libaprutil1_1.6.1-5_amd64.deb ...
Unpacking libaprutil1:amd64 (1.6.1-5) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
Preparing to unpack .../06-libaprutil1-dbd-sqlite3_1.6.1-5_amd64.deb ...
Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-5) ...
Selecting previously unselected package libaprutil1-ldap:amd64.
Preparing to unpack .../07-libaprutil1-ldap_1.6.1-5_amd64.deb ...
Unpacking libaprutil1-ldap:amd64 (1.6.1-5) ...
Selecting previously unselected package libjansson4:amd64.
Preparing to unpack .../08-libjansson4_2.13.1-1.1_amd64.deb ...
Unpacking libjansson4:amd64 (2.13.1-1.1) ...
Selecting previously unselected package liblua5.3-0:amd64.
Preparing to unpack .../09-liblua5.3-0_5.3.3-1.1+b1_amd64.deb ...
Unpacking liblua5.3-0:amd64 (5.3.3-1.1+b1) ...
Selecting previously unselected package libicu67:amd64.
Preparing to unpack .../10-libicu67_67.1-7_amd64.deb ...
Unpacking libicu67:amd64 (67.1-7) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../11-libxml2_2.9.10+dfsg-6.7+deb11u2_amd64.deb ...
Unpacking libxml2:amd64 (2.9.10+dfsg-6.7+deb11u2) ...
Selecting previously unselected package apache2-bin.
Preparing to unpack .../12-apache2-bin_2.4.54-1~deb11u1_amd64.deb ...
Unpacking apache2-bin (2.4.54-1~deb11u1) ...
Selecting previously unselected package apache2-data.
Preparing to unpack .../13-apache2-data_2.4.54-1~deb11u1_all.deb ...
Unpacking apache2-data (2.4.54-1~deb11u1) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../14-apache2-utils_2.4.54-1~deb11u1_amd64.deb ...
Unpacking apache2-utils (2.4.54-1~deb11u1) ...
Selecting previously unselected package mailcap.
Preparing to unpack .../15-mailcap_3.69_all.deb ...
Unpacking mailcap (3.69) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../16-mime-support_3.66_all.deb ...
Unpacking mime-support (3.66) ...
Selecting previously unselected package apache2.
Preparing to unpack .../17-apache2_2.4.54-1~deb11u1_amd64.deb ...
Unpacking apache2 (2.4.54-1~deb11u1) ...
Selecting previously unselected package bzip2.
Preparing to unpack .../18-bzip2_1.0.8-4_amd64.deb ...
Unpacking bzip2 (1.0.8-4) ...
Selecting previously unselected package libmagic-mgc.
Preparing to unpack .../19-libmagic-mgc_1%3a5.39-3_amd64.deb ...
Unpacking libmagic-mgc (1:5.39-3) ...
Selecting previously unselected package libmagic1:amd64.
Preparing to unpack .../20-libmagic1_1%3a5.39-3_amd64.deb ...
Unpacking libmagic1:amd64 (1:5.39-3) ...
Selecting previously unselected package file.
Preparing to unpack .../21-file_1%3a5.39-3_amd64.deb ...
Unpacking file (1:5.39-3) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../22-ssl-cert_1.1.0+nmu1_all.deb ...
Unpacking ssl-cert (1.1.0+nmu1) ...
Setting up libicu67:amd64 (67.1-7) ...
Setting up libmagic-mgc (1:5.39-3) ...
Setting up perl-modules-5.32 (5.32.1-4+deb11u2) ...
Setting up libmagic1:amd64 (1:5.39-3) ...
Setting up libapr1:amd64 (1.7.0-6+deb11u1) ...
Setting up file (1:5.39-3) ...
Setting up bzip2 (1.0.8-4) ...
Setting up libjansson4:amd64 (2.13.1-1.1) ...
Setting up ssl-cert (1.1.0+nmu1) ...
Setting up libgdbm-compat4:amd64 (1.19-2) ...
Setting up libperl5.32:amd64 (5.32.1-4+deb11u2) ...
Setting up liblua5.3-0:amd64 (5.3.3-1.1+b1) ...
Setting up apache2-data (2.4.54-1~deb11u1) ...
Setting up libxml2:amd64 (2.9.10+dfsg-6.7+deb11u2) ...
Setting up libaprutil1:amd64 (1.6.1-5) ...
Setting up libaprutil1-ldap:amd64 (1.6.1-5) ...
Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-5) ...
Setting up perl (5.32.1-4+deb11u2) ...
Setting up mailcap (3.69) ...
Setting up apache2-utils (2.4.54-1~deb11u1) ...
Setting up mime-support (3.66) ...
Setting up apache2-bin (2.4.54-1~deb11u1) ...
Setting up apache2 (2.4.54-1~deb11u1) ...
Enabling module mpm_event.
Enabling module authz_core.
Enabling module authz_host.
Enabling module authn_core.
Enabling module auth_basic.
Enabling module access_compat.
Enabling module authn_file.
Enabling module authz_user.
Enabling module alias.
Enabling module dir.
Enabling module autoindex.
Enabling module env.
Enabling module mime.
Enabling module negotiation.
Enabling module setenvif.
Enabling module filter.
Enabling module deflate.
Enabling module status.
Enabling module reqtimeout.
Enabling conf charset.
Enabling conf localized-error-pages.
Enabling conf other-vhosts-access-log.
Enabling conf security.
Enabling conf serve-cgi-bin.
Enabling site 000-default.
Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /lib/systemd/system/apache2.service.
Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /lib/systemd/system/apache-htcacheclean.service.
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+deb11u4) ...
student-01-e3c2b9ab4419@web-server:~$ echo '<!doctype html><html><body><h1>Hello World!</h1></body></html>' | sudo tee /var/www/html/index.html
<!doctype html><html><body><h1>Hello World!</h1></body></html>
student-01-e3c2b9ab4419@web-server:~$
```

## Task 3. Verify that network traffic is logged

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-121.png)

Access the VPC Flow Logs
In the Cloud Console, go to Navigation menu > Logging > Logs Explorer.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-122.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-123.png)

## Task 4. Export the network traffic to BigQuery to further analyze the logs
Create an export sink
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-124.png)

Generate log traffic for BigQuery
Now that the network traffic logs are being exported to BigQuery, generate more traffic by accessing the web-server several times. Using Cloud Shell, you can curl the IP address of the web-server several times.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-91a00c91652c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-02-91a00c91652c)$ export MY_SERVER=34.172.87.166
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-02-91a00c91652c)$ for ((i=1;i<=50;i++)); do curl $MY_SERVER; done
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
<!doctype html><html><body><h1>Hello World!</h1></body></html>
student_01_e3c2b9ab4419@cloudshell:~ (qwiklabs-gcp-02-91a00c91652c)$
```
Visualize the VPC Flow Logs in BigQuery
In the Cloud Console, on the Navigation menu, click BigQuery.


## Task 5. Add VPC Flow Log aggregation
In this task, you will now explore a new release of VPC flow log volume reduction. Not every packet is captured into its own log record. However, even with sampling, log record captures can be quite large.

You can balance your traffic visibility and storage cost needs by adjusting specific aspects of logs collection, which you will explore in this section.

Review
You configured a VPC network, enabled VPC Flow Logs, and created a web server in that network. Then, you generated HTTP traffic to the web server, viewed the traffic logs in the Cloud Console, and analyzed the traffic logs in BigQuery. Finally, you used VPC Flow Log aggregation for balancing your traffic visibility and storage cost.

There are multiple use cases for VPC Flow Logs. For example, you might use VPC Flow Logs to determine where your applications are being accessed from in order to optimize network traffic expense, to create HTTP load balancers to balance traffic globally, or to deny unwanted IP addresses with Cloud Armor.
