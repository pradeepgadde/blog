---

layout: single
title:  "Creating a Virtual Machine"
date:   2023-03-29 10:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gpcne.png
  og_image: /assets/images/gpcne.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Creating a Virtual Machine

- Create a virtual machine with the Cloud Console.
- Create a virtual machine with the `gcloud` command line.
- Deploy a web server and connect it to a virtual machine.

Certain Compute Engine resources live in regions or zones. A region is a specific geographical location where you can run your resources. Each  region has one or more zones. For example, the us-central1 region  denotes a region in the Central United States that has zones `us-central1-a`, `us-central1-b`, `us-central1-c`, and `us-central1-f`.

Resources that live in a zone are referred to as zonal resources.  Virtual machine Instances and persistent disks live in a zone. To attach a persistent disk to a virtual machine instance, both resources must be in the same zone. Similarly, if you want to assign a static IP address  to an instance, the instance must be in the same region as the static  IP.



```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-514c99c3d06e.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$ gcloud config list project
[core]
project = qwiklabs-gcp-03-514c99c3d06e

Your active configuration is: [cloudshell-25191]
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$
```





```sh
Linux gcelab 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-04-60c907fc0120@gcelab:~$  sudo apt-get update
Hit:1 http://deb.debian.org/debian bullseye InRelease
Get:2 http://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
Get:3 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]         
Get:4 http://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
Get:5 http://security.debian.org/debian-security bullseye-security/main Sources [188 kB]
Get:6 http://security.debian.org/debian-security bullseye-security/main amd64 Packages [236 kB]
Get:7 http://security.debian.org/debian-security bullseye-security/main Translation-en [155 kB]
Get:8 http://deb.debian.org/debian bullseye-updates/main Sources.diff/Index [17.3 kB]
Get:9 http://deb.debian.org/debian bullseye-updates/main amd64 Packages.diff/Index [17.3 kB]
Get:10 http://deb.debian.org/debian bullseye-updates/main Sources T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [391 B]
Get:10 http://deb.debian.org/debian bullseye-updates/main Sources T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [391 B]
Get:11 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [288 B]
Get:11 http://deb.debian.org/debian bullseye-updates/main amd64 Packages T-2023-03-25-2025.40-F-2023-03-25-2025.40.pdiff [288 B]
Get:12 http://deb.debian.org/debian bullseye-backports/main Sources.diff/Index [63.3 kB]
Get:13 http://deb.debian.org/debian bullseye-backports/main amd64 Packages.diff/Index [63.3 kB]
Get:14 http://deb.debian.org/debian bullseye-backports/main Translation-en.diff/Index [63.3 kB]
Get:15 http://deb.debian.org/debian bullseye-backports/main Sources T-2023-03-28-0207.50-F-2023-03-06-2006.07.pdiff [42.6 kB]
Get:16 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2023-03-28-0207.50-F-2023-03-06-2006.07.pdiff [46.9 kB]
Get:15 http://deb.debian.org/debian bullseye-backports/main Sources T-2023-03-28-0207.50-F-2023-03-06-2006.07.pdiff [42.6 kB]
Get:16 http://deb.debian.org/debian bullseye-backports/main amd64 Packages T-2023-03-28-0207.50-F-2023-03-06-2006.07.pdiff [46.9 kB]
Get:17 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2023-03-26-1414.40-F-2023-03-07-2009.30.pdiff [24.7 kB]
Get:17 http://deb.debian.org/debian bullseye-backports/main Translation-en T-2023-03-26-1414.40-F-2023-03-07-2009.30.pdiff [24.7 kB]
Get:18 http://packages.cloud.google.com/apt google-compute-engine-bullseye-stable InRelease [5146 B]    
Get:19 http://packages.cloud.google.com/apt cloud-sdk-bullseye InRelease [6400 B]
Get:20 http://packages.cloud.google.com/apt google-compute-engine-bullseye-stable/main amd64 Packages [1903 B]
Get:21 http://packages.cloud.google.com/apt cloud-sdk-bullseye/main amd64 Packages [266 kB]
Fetched 1339 kB in 2s (541 kB/s)  
Reading package lists... Done
student-04-60c907fc0120@gcelab:~$  sudo apt-get install -y nginx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0 libfontconfig1 libgd3 libgeoip1 libicu67
  libjbig0 libjpeg62-turbo libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip libtiff5
  libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 nginx-common nginx-core
Suggested packages:
  libgd-tools geoip-bin fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database libdeflate0 libfontconfig1 libgd3 libgeoip1 libicu67
  libjbig0 libjpeg62-turbo libnginx-mod-http-geoip libnginx-mod-http-image-filter
  libnginx-mod-http-xslt-filter libnginx-mod-mail libnginx-mod-stream libnginx-mod-stream-geoip libtiff5
  libwebp6 libx11-6 libx11-data libxau6 libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 nginx nginx-common
  nginx-core
0 upgraded, 29 newly installed, 0 to remove and 5 not upgraded.
Need to get 18.0 MB of archives.
After this operation, 60.1 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bullseye/main amd64 fonts-dejavu-core all 2.37-2 [1069 kB]
Get:2 http://security.debian.org/debian-security bullseye-security/main amd64 libtiff5 amd64 4.2.0-1+deb11u4 [290 kB]
Get:3 http://deb.debian.org/debian bullseye/main amd64 fontconfig-config all 2.13.1-4.2 [281 kB]
Get:4 http://deb.debian.org/debian bullseye/main amd64 geoip-database all 20191224-3 [3032 kB]
Get:5 http://deb.debian.org/debian bullseye/main amd64 libdeflate0 amd64 1.7-1 [53.1 kB]
Get:6 http://deb.debian.org/debian bullseye/main amd64 libfontconfig1 amd64 2.13.1-4.2 [347 kB]
Get:7 http://deb.debian.org/debian bullseye/main amd64 libjpeg62-turbo amd64 1:2.0.6-4 [151 kB]
Get:8 http://deb.debian.org/debian bullseye/main amd64 libjbig0 amd64 2.1-3.1+b2 [31.0 kB]
Get:9 http://deb.debian.org/debian bullseye/main amd64 libwebp6 amd64 0.6.1-2.1 [258 kB]
Get:10 http://deb.debian.org/debian bullseye/main amd64 libxau6 amd64 1:1.0.9-1 [19.7 kB]
Get:11 http://deb.debian.org/debian bullseye/main amd64 libxdmcp6 amd64 1:1.1.2-3 [26.3 kB]
Get:12 http://deb.debian.org/debian bullseye/main amd64 libxcb1 amd64 1.14-3 [140 kB]
Get:13 http://deb.debian.org/debian bullseye/main amd64 libx11-data all 2:1.7.2-1 [311 kB]
Get:14 http://deb.debian.org/debian bullseye/main amd64 libx11-6 amd64 2:1.7.2-1 [772 kB]
Get:15 http://deb.debian.org/debian bullseye/main amd64 libxpm4 amd64 1:3.5.12-1 [49.1 kB]
Get:16 http://deb.debian.org/debian bullseye/main amd64 libgd3 amd64 2.3.0-2 [137 kB]
Get:17 http://deb.debian.org/debian bullseye/main amd64 libgeoip1 amd64 1.6.12-7 [92.5 kB]
Get:18 http://deb.debian.org/debian bullseye/main amd64 libicu67 amd64 67.1-7 [8622 kB]
Get:19 http://deb.debian.org/debian bullseye/main amd64 nginx-common all 1.18.0-6.1+deb11u3 [126 kB]
Get:20 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-geoip amd64 1.18.0-6.1+deb11u3 [98.4 kB]
Get:21 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-image-filter amd64 1.18.0-6.1+deb11u3 [102 kB]
Get:22 http://deb.debian.org/debian bullseye/main amd64 libxml2 amd64 2.9.10+dfsg-6.7+deb11u3 [693 kB]
Get:23 http://deb.debian.org/debian bullseye/main amd64 libxslt1.1 amd64 1.1.34-4+deb11u1 [240 kB]
Get:24 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-http-xslt-filter amd64 1.18.0-6.1+deb11u3 [100 kB]
Get:25 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-mail amd64 1.18.0-6.1+deb11u3 [129 kB]
Get:26 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-stream amd64 1.18.0-6.1+deb11u3 [154 kB]
Get:27 http://deb.debian.org/debian bullseye/main amd64 libnginx-mod-stream-geoip amd64 1.18.0-6.1+deb11u3 [97.7 kB]
Get:28 http://deb.debian.org/debian bullseye/main amd64 nginx-core amd64 1.18.0-6.1+deb11u3 [515 kB]
Get:29 http://deb.debian.org/debian bullseye/main amd64 nginx all 1.18.0-6.1+deb11u3 [92.9 kB]
Fetched 18.0 MB in 0s (56.0 MB/s)
Preconfiguring packages ...
Selecting previously unselected package fonts-dejavu-core.
(Reading database ... 55808 files and directories currently installed.)
Preparing to unpack .../00-fonts-dejavu-core_2.37-2_all.deb ...
Unpacking fonts-dejavu-core (2.37-2) ...
Selecting previously unselected package fontconfig-config.
Preparing to unpack .../01-fontconfig-config_2.13.1-4.2_all.deb ...
Unpacking fontconfig-config (2.13.1-4.2) ...
Selecting previously unselected package geoip-database.
Preparing to unpack .../02-geoip-database_20191224-3_all.deb ...
Unpacking geoip-database (20191224-3) ...
Selecting previously unselected package libdeflate0:amd64.
Preparing to unpack .../03-libdeflate0_1.7-1_amd64.deb ...
Unpacking libdeflate0:amd64 (1.7-1) ...
Selecting previously unselected package libfontconfig1:amd64.
Preparing to unpack .../04-libfontconfig1_2.13.1-4.2_amd64.deb ...
Unpacking libfontconfig1:amd64 (2.13.1-4.2) ...
Selecting previously unselected package libjpeg62-turbo:amd64.
Preparing to unpack .../05-libjpeg62-turbo_1%3a2.0.6-4_amd64.deb ...
Unpacking libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Selecting previously unselected package libjbig0:amd64.
Preparing to unpack .../06-libjbig0_2.1-3.1+b2_amd64.deb ...
Unpacking libjbig0:amd64 (2.1-3.1+b2) ...
Selecting previously unselected package libwebp6:amd64.
Preparing to unpack .../07-libwebp6_0.6.1-2.1_amd64.deb ...
Unpacking libwebp6:amd64 (0.6.1-2.1) ...
Selecting previously unselected package libtiff5:amd64.
Preparing to unpack .../08-libtiff5_4.2.0-1+deb11u4_amd64.deb ...
Unpacking libtiff5:amd64 (4.2.0-1+deb11u4) ...
Selecting previously unselected package libxau6:amd64.
Preparing to unpack .../09-libxau6_1%3a1.0.9-1_amd64.deb ...
Unpacking libxau6:amd64 (1:1.0.9-1) ...
Selecting previously unselected package libxdmcp6:amd64.
Preparing to unpack .../10-libxdmcp6_1%3a1.1.2-3_amd64.deb ...
Unpacking libxdmcp6:amd64 (1:1.1.2-3) ...
Selecting previously unselected package libxcb1:amd64.
Preparing to unpack .../11-libxcb1_1.14-3_amd64.deb ...
Unpacking libxcb1:amd64 (1.14-3) ...
Selecting previously unselected package libx11-data.
Preparing to unpack .../12-libx11-data_2%3a1.7.2-1_all.deb ...
Unpacking libx11-data (2:1.7.2-1) ...
Selecting previously unselected package libx11-6:amd64.
Preparing to unpack .../13-libx11-6_2%3a1.7.2-1_amd64.deb ...
Unpacking libx11-6:amd64 (2:1.7.2-1) ...
Selecting previously unselected package libxpm4:amd64.
Preparing to unpack .../14-libxpm4_1%3a3.5.12-1_amd64.deb ...
Unpacking libxpm4:amd64 (1:3.5.12-1) ...
Selecting previously unselected package libgd3:amd64.
Preparing to unpack .../15-libgd3_2.3.0-2_amd64.deb ...
Unpacking libgd3:amd64 (2.3.0-2) ...
Selecting previously unselected package libgeoip1:amd64.
Preparing to unpack .../16-libgeoip1_1.6.12-7_amd64.deb ...
Unpacking libgeoip1:amd64 (1.6.12-7) ...
Selecting previously unselected package libicu67:amd64.
Preparing to unpack .../17-libicu67_67.1-7_amd64.deb ...
Unpacking libicu67:amd64 (67.1-7) ...
Selecting previously unselected package nginx-common.
Preparing to unpack .../18-nginx-common_1.18.0-6.1+deb11u3_all.deb ...
Unpacking nginx-common (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-http-geoip.
Preparing to unpack .../19-libnginx-mod-http-geoip_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-geoip (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-http-image-filter.
Preparing to unpack .../20-libnginx-mod-http-image-filter_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-image-filter (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../21-libxml2_2.9.10+dfsg-6.7+deb11u3_amd64.deb ...
Unpacking libxml2:amd64 (2.9.10+dfsg-6.7+deb11u3) ...
Selecting previously unselected package libxslt1.1:amd64.
Preparing to unpack .../22-libxslt1.1_1.1.34-4+deb11u1_amd64.deb ...
Unpacking libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Selecting previously unselected package libnginx-mod-http-xslt-filter.
Preparing to unpack .../23-libnginx-mod-http-xslt-filter_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-http-xslt-filter (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-mail.
Preparing to unpack .../24-libnginx-mod-mail_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-mail (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-stream.
Preparing to unpack .../25-libnginx-mod-stream_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-stream (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package libnginx-mod-stream-geoip.
Preparing to unpack .../26-libnginx-mod-stream-geoip_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking libnginx-mod-stream-geoip (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package nginx-core.
Preparing to unpack .../27-nginx-core_1.18.0-6.1+deb11u3_amd64.deb ...
Unpacking nginx-core (1.18.0-6.1+deb11u3) ...
Selecting previously unselected package nginx.
Preparing to unpack .../28-nginx_1.18.0-6.1+deb11u3_all.deb ...
Unpacking nginx (1.18.0-6.1+deb11u3) ...
Setting up libxau6:amd64 (1:1.0.9-1) ...
Setting up libxdmcp6:amd64 (1:1.1.2-3) ...
Setting up libxcb1:amd64 (1.14-3) ...
Setting up libicu67:amd64 (67.1-7) ...
Setting up libdeflate0:amd64 (1.7-1) ...
Setting up nginx-common (1.18.0-6.1+deb11u3) ...
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /lib/systemd/system/nginx.service.
Setting up libjbig0:amd64 (2.1-3.1+b2) ...
Setting up libjpeg62-turbo:amd64 (1:2.0.6-4) ...
Setting up libx11-data (2:1.7.2-1) ...
Setting up libwebp6:amd64 (0.6.1-2.1) ...
Setting up fonts-dejavu-core (2.37-2) ...
Setting up libgeoip1:amd64 (1.6.12-7) ...
Setting up libx11-6:amd64 (2:1.7.2-1) ...
Setting up libtiff5:amd64 (4.2.0-1+deb11u4) ...
Setting up geoip-database (20191224-3) ...
Setting up libxml2:amd64 (2.9.10+dfsg-6.7+deb11u3) ...
Setting up libnginx-mod-mail (1.18.0-6.1+deb11u3) ...
Setting up libxpm4:amd64 (1:3.5.12-1) ...
Setting up fontconfig-config (2.13.1-4.2) ...
Setting up libnginx-mod-stream (1.18.0-6.1+deb11u3) ...
Setting up libnginx-mod-stream-geoip (1.18.0-6.1+deb11u3) ...
Setting up libnginx-mod-http-geoip (1.18.0-6.1+deb11u3) ...
Setting up libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Setting up libfontconfig1:amd64 (2.13.1-4.2) ...
Setting up libnginx-mod-http-xslt-filter (1.18.0-6.1+deb11u3) ...
Setting up libgd3:amd64 (2.3.0-2) ...
Setting up libnginx-mod-http-image-filter (1.18.0-6.1+deb11u3) ...
Setting up nginx-core (1.18.0-6.1+deb11u3) ...
Upgrading binary: nginx.
Setting up nginx (1.18.0-6.1+deb11u3) ...
Processing triggers for man-db (2.9.4-2) ...
Processing triggers for libc-bin (2.31-13+deb11u5) ...
student-04-60c907fc0120@gcelab:~$  ps auwx | grep nginx
root        1762  0.0  0.2  58424 11452 ?        S    17:28   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
www-data    1764  0.0  0.2  70104 10576 ?        S    17:28   0:00 nginx: worker process
www-data    1765  0.0  0.2  70104 10724 ?        S    17:28   0:00 nginx: worker process
student+    1792  0.0  0.0   5136   704 pts/0    S+   17:29   0:00 grep nginx
student-04-60c907fc0120@gcelab:~$ 
```



```sh
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$      gcloud compute instances create gcelab2 --machine-type e2-medium --zone us-east1-c
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-514c99c3d06e/zones/us-east1-c/instances/gcelab2].
NAME: gcelab2
ZONE: us-east1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.142.0.3
EXTERNAL_IP: 34.23.72.13
STATUS: RUNNING
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$
```

At this point, there should be two VMs

```sh
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$ gcloud compute instances list
NAME: gcelab
ZONE: us-east1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.142.0.2
EXTERNAL_IP: 34.73.79.28
STATUS: RUNNING

NAME: gcelab2
ZONE: us-east1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE:
INTERNAL_IP: 10.142.0.3
EXTERNAL_IP: 34.23.72.13
STATUS: RUNNING
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$
```



```sh
student_04_60c907fc0120@cloudshell:~ (qwiklabs-gcp-03-514c99c3d06e)$      gcloud compute ssh gcelab2 --zone us-east1-c 
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_04_60c907fc0120/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  Y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student_04_60c907fc0120/.ssh/google_compute_engine
Your public key has been saved in /home/student_04_60c907fc0120/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:22RjvJknjplyBt/JtqjGb5Pe6GuAP8STd3qVoz+h0bg student_04_60c907fc0120@cs-737665234318-default
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|                 |
|         .       |
|     o .S *o .   |
|    . B .*+=*    |
|     + *.**B.o   |
|      * XOEo.    |
|     ..%@=+o..   |
+----[SHA256]-----+
Warning: Permanently added 'compute.1993197709913869097' (ECDSA) to the list of known hosts.
Linux gcelab2 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-04-60c907fc0120'.
student-04-60c907fc0120@gcelab2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:0a:8e:00:03 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.142.0.3/32 brd 10.142.0.3 scope global dynamic ens4
       valid_lft 3492sec preferred_lft 3492sec
    inet6 fe80::4001:aff:fe8e:3/64 scope link
       valid_lft forever preferred_lft forever
student-04-60c907fc0120@gcelab2:~$ ping 10.142.0.2
PING 10.142.0.2 (10.142.0.2) 56(84) bytes of data.
64 bytes from 10.142.0.2: icmp_seq=1 ttl=64 time=2.82 ms
64 bytes from 10.142.0.2: icmp_seq=2 ttl=64 time=0.321 ms
64 bytes from 10.142.0.2: icmp_seq=3 ttl=64 time=0.326 ms
^C
--- 10.142.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2024ms
rtt min/avg/max/mdev = 0.321/1.156/2.821/1.177 ms
student-04-60c907fc0120@gcelab2:~$
```

We can ping each other.
