---

layout: single
title:  "Cloud SQL for MySQL: Qwik Start"
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
# Cloud SQL for MySQL: Qwik Start

In this lab, you learn how to create and connect to a Cloud SQL for  MySQL instance and perform basic SQL operations using the Cloud console  and the `mysql` client.

- Create a Cloud SQL instance
- Connect to the instance in Cloud Shell
- Create a database and upload data

## Create a Cloud SQL instance

1. From the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) click on **SQL**.
2. Click **Create Instance**.
3. Choose **MySQL** database engine.
4. Enter Instance ID as `myinstance`.
5. In the password field click on the **Generate** link and the eye icon to see the password. **Save** the password to use in the next section.
6. Select the database version as **MySQL 8**.
7. For **Choose a Cloud SQL edition**, select **Enterprise** edition.
8. For **Preset** choose **Development** (4 vCPU, 16 GB RAM, 100 GB Storage, Single zone).
9. Set **Region** as .
10. Set the **Multi zones (Highly available)** > **Primary Zone** field as .
11. Click **CREATE INSTANCE**.

It might take a few minutes for the instance to be created. Once it  is, you will see a green checkmark next to the instance name.

1. Click on the Cloud SQL instance. The **SQL Overview** page opens.



## Connect to your instance using the mysql client in Cloud Shell

1. In the Cloud Console, click the **Cloud Shell** icon in the upper right corner.

1. Click **Continue**.

2. At the Cloud Shell prompt, connect to your Cloud SQL instance by running the following:

3. Click **Authorize**.

   1. Enter your root password when prompted. **Note:** The cursor will not move.
   2. Press the **Enter** key when you're done typing.

   You should now see the `mysql` prompt.

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-b21c0b25561f)$ gcloud sql connect myinstance --user=root
WARNING: If you're connecting from an IPv6 address, or are constrained by certain organization policies (restrictPublicIP, restrictAuthorizedNetworks), consider running the beta version of this command by connecting through the Cloud SQL proxy: gcloud beta sql connect
ERROR: (gcloud.sql.connect) HTTPError 409: Operation failed because another operation was already in progress. Try your request after the current operation is complete.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-b21c0b25561f)$ gcloud sql connect myinstance --user=root
Allowlisting your IP for incoming connection for 5 minutes...done.                                                                                                                 
Connecting to database with SQL user [root].Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 40
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```



## Create a database and upload data

1. Create a SQL database called `guestbook` on your Cloud SQL instance:

```sql
mysql> CREATE DATABASE guestbook;
Query OK, 1 row affected (0.23 sec)

mysql> USE guestbook;
Database changed
mysql> CREATE TABLE entries (guestName VARCHAR(255), content VARCHAR(255),
    ->     entryID INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(entryID));
Query OK, 0 rows affected (0.25 sec)

mysql>     INSERT INTO entries (guestName, content) values ("first guest", "I got here!");
Query OK, 1 row affected (0.24 sec)

mysql> INSERT INTO entries (guestName, content) values ("second guest", "Me too!");
Query OK, 1 row affected (0.23 sec)

mysql> SELECT * FROM entries;
+--------------+-------------+---------+
| guestName    | content     | entryID |
+--------------+-------------+---------+
| first guest  | I got here! |       1 |
| second guest | Me too!     |       2 |
+--------------+-------------+---------+
2 rows in set (0.22 sec)

mysql> exit;
Bye
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-03-b21c0b25561f)$ 
```



You have created a Cloud SQL for MySQL instance and database, and then uploaded data.
