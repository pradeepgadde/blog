---

layout: single
title:  "Bigtable: Qwik Start"
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
# Bigtable: Qwik Start

[Cloud Bigtable](https://cloud.google.com/bigtable/) is Google's NoSQL Big Data database service. It's the same database  that powers many core Google services, including Search, Analytics,  Maps, and Gmail. Bigtable is designed to handle massive workloads at  consistent low latency and high throughput, so it's a great choice for  both operational and analytical applications, including IoT, user  analytics, and financial data analysis.

## Create a Cloud Bigtable instance

1. In the Cloud Console, go to **Navigation menu (![Navigation_menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D))**, click on **Bigtable** in the Databases section, then click **Create instance**.

Bigtable is a fully managed, wide-column NoSQL database that offers                        low latency and replication for high availability. To use Bigtable,                        create an instance and then set up your development environment to access                        Bigtable so that you can add data and monitor performance. 

A Bigtable instance is a container for your clusters.

A cluster handles application requests for an instance. It contains  nodes which determine your cluster's performance and storage limit. 

 Additional clusters can be added at any time. 

 Nodes are  compute resources that Bigtable uses to manage your data and perform  maintenance tasks. Adding nodes helps a cluster handle larger workloads. 

 Scaling mode and configurations can be changed at any time. 



```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ gcloud bigtable instances list
NAME: quickstart-instance
DISPLAY_NAME: quickstart-instance
STATE: READY
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```



## Connect to your instance

- In Cloud Shell, configure `cbt` to use your project and instance by modifying the `.cbtrc` file:

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cat ~/.cbtrc 
project = qwiklabs-gcp-03-96e0dc631273
instance = quickstart-instance
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```



## Read and write data

Cloud Bigtable stores data in *tables*, which contain *rows*. Each row is identified by a *row key*.

Data in a row is organized into *column families*, or groups of columns. A *column qualifier* identifies a single column within a column family.

A *cell* is the intersection of a row and a column. Each cell can contain multiple *versions* of a value.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt createtable my-table
2024/05/22 01:51:55 -creds flag unset, will use gcloud credential
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt ls
2024/05/22 01:52:11 -creds flag unset, will use gcloud credential
my-table
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt createfamily my-table cf1
2024/05/22 01:52:32 -creds flag unset, will use gcloud credential
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt ls my-table
2024/05/22 01:52:41 -creds flag unset, will use gcloud credential
Family Name     GC Policy
-----------     ---------
cf1             <never>
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```

```sh
tudent_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt set my-table r1 cf1:c1=test-value
2024/05/22 01:53:13 -creds flag unset, will use gcloud credential
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt read my-table
2024/05/22 01:53:21 -creds flag unset, will use gcloud credential
----------------------------------------
r1
  cf1:c1                                   @ 2024/05/22-01:53:13.807000
    "test-value"

student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt deletetable my-table
2024/05/22 01:53:51 -creds flag unset, will use gcloud credential
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ cbt ls
2024/05/22 01:53:55 -creds flag unset, will use gcloud credential
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ history 
    1  echo project = `gcloud config get-value project` > ~/.cbtrc
    2  echo instance = quickstart-instance >> ~/.cbtrc
    3  gcloud bigtable instances list
    4  cat ~/.cbtrc 
    5  cbt createtable my-table
    6  cbt ls
    7  cbt createfamily my-table cf1
    8  cbt ls my-table
    9  cbt set my-table r1 cf1:c1=test-value
   10  cbt read my-table
   11  cbt deletetable my-table
   12  cbt ls
   13  history 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-96e0dc631273)$ 
```

