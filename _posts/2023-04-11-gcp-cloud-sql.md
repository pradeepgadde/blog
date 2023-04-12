---


layout: single
title:  "Implementing Cloud SQL"
date:   2023-04-11 09:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Implementing Cloud SQL

In this lab, you configure a Cloud SQL server and learn how to  connect an application to it via a proxy over an external connection.  You also configure a connection over a Private IP link that offers  performance and security benefits. The app we chose to demonstrate in  this lab is Wordpress, but the information and best practices are  applicable to any application that needs SQL Server.

By the end of this lab, you will have 2 working instances of the  Wordpress frontend connected over 2 different connection types to their  SQL instance backend, as shown in this diagram:

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-1.png)

In this lab, you learn how to perform the following tasks:

- Create a Cloud SQL database
- Configure a virtual machine to run a proxy
- Create a connection between an application and Cloud SQL
- Connect an application to Cloud SQL using Private IP address

## Create a Cloud SQL database

In this task, you configure a SQL server according to Google Cloud best practices and create a Private IP connection.

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-11.png)





## Configure a proxy on a virtual machine

When your application does not reside in the same VPC connected  network and region as your Cloud SQL instance, use a proxy to secure its external connection.

In order to configure the proxy, you need the Cloud SQL instance connection name.

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-12.png)

```sh
Linux wordpress-proxy 5.10.0-21-cloud-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-62c2f98ca037'.
student-02-62c2f98ca037@wordpress-proxy:~$ wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy && chmod +x cloud_sql_proxy
--2023-04-12 03:00:04--  https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64
Resolving dl.google.com (dl.google.com)... 74.125.124.136, 74.125.124.93, 74.125.124.190, ...
Connecting to dl.google.com (dl.google.com)|74.125.124.136|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 17991205 (17M) [application/octet-stream]
Saving to: ‘cloud_sql_proxy’

cloud_sql_proxy             100%[==========================================>]  17.16M  --.-KB/s    in 0.06s   

2023-04-12 03:00:04 (280 MB/s) - ‘cloud_sql_proxy’ saved [17991205/17991205]

student-02-62c2f98ca037@wordpress-proxy:~$ 
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-16.png)

```sh
student-02-62c2f98ca037@wordpress-proxy:~$ export SQL_CONNECTION=qwiklabs-gcp-02-874ed12f5c8c:us-central1:wordpress-db
student-02-62c2f98ca037@wordpress-proxy:~$ echo $SQL_CONNECTION
qwiklabs-gcp-02-874ed12f5c8c:us-central1:wordpress-db
student-02-62c2f98ca037@wordpress-proxy:~$ 

```

To activate the proxy connection to your Cloud SQL database and send the process to the background, run the following command:

```sh
student-02-62c2f98ca037@wordpress-proxy:~$ ./cloud_sql_proxy -instances=$SQL_CONNECTION=tcp:3306 &
[1] 14823
student-02-62c2f98ca037@wordpress-proxy:~$ 2023/04/12 03:05:03 current FDs rlimit set to 1048576, wanted limit is 8500. Nothing to do here.
2023/04/12 03:05:07 Listening on 127.0.0.1:3306 for qwiklabs-gcp-02-874ed12f5c8c:us-central1:wordpress-db
2023/04/12 03:05:07 Ready for new connections
2023/04/12 03:05:07 Generated RSA key in 311.428622ms

student-02-62c2f98ca037@wordpress-proxy:~$ 
```



## Connect an application to the Cloud SQL instance

In this task, you will connect a sample application to the Cloud SQL instance.

1. Configure the Wordpress application. To find the external IP address of your virtual machine, query its metadata

```sh
student-02-62c2f98ca037@wordpress-proxy:~$ curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/network-interfaces/0/access-configs/0/external-ip && echo
34.69.150.226
student-02-62c2f98ca037@wordpress-proxy:~$ 
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-17.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-20.png)

## Connect to Cloud SQL via internal IP

If you can host your application in the same region and VPC connected network as your Cloud SQL, you can leverage a more secure and  performant configuration using Private IP.

By using Private IP, you will increase performance by reducing  latency and minimize the attack surface of your Cloud SQL instance  because you can communicate with it exclusively over internal IPs.

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-23.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-24.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/sql-25.png)



In this lab, you created a Cloud SQL database and configured it to use  both an external connection over a secure proxy and a Private IP  address, which is more secure and performant. Remember that you can only connect via Private IP if the application and the Cloud SQL server are  collocated in the same region and are part of the same VPC network. If  your application is hosted in another region, VPC, or even project, use a proxy to secure its connection over the external connection.







