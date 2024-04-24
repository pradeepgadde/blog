---
layout: single
title:  "Migrate to Cloud SQL for PostgreSQL using Database Migration Service "
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Migrate to Cloud SQL for PostgreSQL using Database Migration Service 
Database Migration Service provides options for one-time and continuous jobs to migrate data to Cloud SQL using different connectivity options, including IP allowlists, VPC peering, and reverse SSH tunnels (see documentation on connectivity options at https://cloud.google.com/database-migration/docs/postgresql/configure-connectivity).

In this lab, you migrate a stand-alone PostgreSQL database (running on a virtual machine) to Cloud SQL for PostgreSQL using a continuous Database Migration Service job and VPC peering for connectivity.

Migrating a database via Database Migration Service requires some preparation of the source database, including creating a dedicated user with replication rights, adding the pglogical database extension to the source database and granting rights to the schemata and tables in the database to be migrated, as well as the postgres database, to that user.

After you create and run the migration job, you confirm that an initial copy of your database has been successfully migrated to your Cloud SQL for PostgreSQL instance. You also explore how continuous migration jobs apply data updates from your source database to your Cloud SQL instance. To conclude the migration job, you promote the Cloud SQL instance to be a stand-alone database for reading and writing data.

Verify that the Database Migration API is enabled

In the Google Cloud console, enter Database Migration API in the top search bar. Click on the result for Database Migration API.

Verify that the Service Networking API is enabled

The Service Networking API is required in order to be able to configure Cloud SQL to support VPC Peering and connections over a private ip-address.

In this task you will add supporting features to the source database which are required in order for Database Migration Service to perform a migration. These are:

Installing and configuring the pglogical database extension.
Configuring the stand-alone PostgreSQL database to allow access from Cloud Shell and Cloud SQL.
Adding the pglogicaldatabase extension to the postgres, orders and gmemegen_db databases on the stand-alone server.
Creating a migration_admin user (with Replication permissions) for database migration and granting the required permissions to schemata and relations to that user.


```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-a78ba407065a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_5f77cf67d70a@cloudshell:~ (qwiklabs-gcp-01-a78ba407065a)$ gcloud compute ssh --zone "us-east1-d" "postgresql-vm" --project "qwiklabs-gcp-01-a78ba407065a"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_03_5f77cf67d70a/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_03_5f77cf67d70a/.ssh/google_compute_engine
Your public key has been saved in /home/student_03_5f77cf67d70a/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:I64N+grol4XUXtrG2IADu1exniAw+gjVXtk70hMW4XM student_03_5f77cf67d70a@cs-1081786469575-default
The key's randomart image is:
+---[RSA 3072]----+
|   .   ooo       |
|o.. ..o.+        |
|ooo.o.ooooE      |
|oo =.=..=o       |
|.o+ B @.So       |
|o..o O * .       |
|o . + o          |
|.. + +           |
| .+oo .          |
+----[SHA256]-----+
Warning: Permanently added 'compute.2221867867572640089' (ED25519) to the list of known hosts.
Linux postgresql-vm 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
student-03-5f77cf67d70a@postgresql-vm:~$ sudo apt install postgresql-13-pglogical
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  postgresql-13-pglogical
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 435 kB of archives.
After this operation, 1260 kB of additional disk space will be used.
Get:1 http://apt.postgresql.org/pub/repos/apt bullseye-pgdg/main amd64 postgresql-13-pglogical amd64 2.4.4-1.pgdg110+1 [435 kB]
Fetched 435 kB in 1s (544 kB/s)                  
Selecting previously unselected package postgresql-13-pglogical.
(Reading database ... 69608 files and directories currently installed.)
Preparing to unpack .../postgresql-13-pglogical_2.4.4-1.pgdg110+1_amd64.deb ...
Unpacking postgresql-13-pglogical (2.4.4-1.pgdg110+1) ...
Setting up postgresql-13-pglogical (2.4.4-1.pgdg110+1) ...
Processing triggers for postgresql-common (259.pgdg110+1) ...
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
Removing obsolete dictionary files:
student-03-5f77cf67d70a@postgresql-vm:~$ 
student-03-5f77cf67d70a@postgresql-vm:~$ sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"

sudo systemctl restart postgresql@13-main
Copying gs://cloud-training/gsp918/pg_hba_append.conf...
/ [1 files][   68.0 B/   68.0 B]                                                
Operation completed over 1 objects/68.0 B.                                       
Copying gs://cloud-training/gsp918/postgresql_append.conf...
/ [1 files][  543.0 B/  543.0 B]                                                
Operation completed over 1 objects/543.0 B.                                      
student-03-5f77cf67d70a@postgresql-vm:~$ sudo su - postgres
postgres@postgresql-vm:~$ psql
psql (13.14 (Debian 13.14-1.pgdg110+2))
Type "help" for help.

postgres=# \c postgres;
You are now connected to database "postgres" as user "postgres".
postgres=# CREATE EXTENSION pglogical;
CREATE EXTENSION
postgres=# \c orders;
You are now connected to database "orders" as user "postgres".
orders=# CREATE EXTENSION pglogical;
CREATE EXTENSION
orders=# \c gmemegen_db;
You are now connected to database "gmemegen_db" as user "postgres".
gmemegen_db=# CREATE EXTENSION pglogical;
CREATE EXTENSION
gmemegen_db=# \l
                               List of databases
    Name     |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-------------+----------+----------+---------+---------+-----------------------
 gmemegen_db | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 orders      | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres    | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
 template1   | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
             |          |          |         |         | postgres=CTc/postgres
(5 rows)

gmemegen_db=# 
gmemegen_db=# CREATE USER migration_admin PASSWORD 'DMS_1s_cool!';
ALTER DATABASE orders OWNER TO migration_admin;
ALTER ROLE migration_admin WITH REPLICATION;
CREATE ROLE
ALTER DATABASE
ALTER ROLE
gmemegen_db=# 
gmemegen_db=# 
gmemegen_db=# \c postgres;
You are now connected to database "postgres" as user "postgres".
postgres=# GRANT USAGE ON SCHEMA pglogical TO migration_admin;
           GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
postgres=# 
postgres=# 
postgres=# 
postgres=# \c orders;
You are now connected to database "orders" as user "postgres".
orders=# GRANT USAGE ON SCHEMA pglogical TO migration_admin;
         GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
orders=# GRANT USAGE ON SCHEMA public TO migration_admin;
GRANT ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON public.distribution_centers TO migration_admin;
GRANT SELECT ON public.inventory_items TO migration_admin;
GRANT SELECT ON public.order_items TO migration_admin;
GRANT SELECT ON public.products TO migration_admin;
GRANT SELECT ON public.users TO migration_admin;
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
orders=# \c gmemegen_db;
You are now connected to database "gmemegen_db" as user "postgres".
gmemegen_db=# GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
GRANT
gmemegen_db=# GRANT USAGE ON SCHEMA public TO migration_admin;
GRANT ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON public.meme TO migration_admin;
GRANT
GRANT
GRANT
gmemegen_db=# \c orders;
\dt
You are now connected to database "orders" as user "postgres".
                List of relations
 Schema |         Name         | Type  |  Owner   
--------+----------------------+-------+----------
 public | distribution_centers | table | postgres
 public | inventory_items      | table | postgres
 public | order_items          | table | postgres
 public | products             | table | postgres
 public | users                | table | postgres
(5 rows)

orders=# ALTER TABLE public.distribution_centers OWNER TO migration_admin;
ALTER TABLE public.inventory_items OWNER TO migration_admin;
ALTER TABLE public.order_items OWNER TO migration_admin;
ALTER TABLE public.products OWNER TO migration_admin;
ALTER TABLE public.users OWNER TO migration_admin;
\dt
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
                    List of relations
 Schema |         Name         | Type  |      Owner      
--------+----------------------+-------+-----------------
 public | distribution_centers | table | migration_admin
 public | inventory_items      | table | migration_admin
 public | order_items          | table | migration_admin
 public | products             | table | migration_admin
 public | users                | table | migration_admin
(5 rows)

orders=# \q
postgres@postgresql-vm:~$ exit
logout
student-03-5f77cf67d70a@postgresql-vm:~$ 
```

Create a new connection profile for the PostgreSQL source instance

A connection profile stores information about the source database instance (e.g., stand-alone PosgreSQL) and is used by the Database Migration Service to migrate data from the source to your destination Cloud SQL database instance. After you create a connection profile, it can be reused across migration jobs.

In this step you will create a new connection profile for the PostgreSQL source instance.

In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click Database Migration > Connection profiles.
    
Click + Create Profile.
    
For Database engine, select PostgreSQL.
    
For Connection profile name, enter postgres-vm.
    
For Hostname or IP address, enter the internal IP for the PostgreSQL source instance that you copied in the previous task (e.g., 10.128.0.2)
    
For Port, enter 5432.
    
For Username, enter migration_admin.
    
For Password, enter DMS_1s_cool! .
    
For Region select .
    
For all other values leave the defaults.
    
Click Create.



A new connection profile named postgres-vm will appear in the Connections profile list.

Create and start a continuous migration job

When you create a new migration job, you first define the source database instance using a previously created connection profile. Then you create a new destination database instance and configure connectivity between the source and destination instances.

In this task, you use the migration job interface to create a new Cloud SQL for PostgreSQL database instance and set it as the destination for the continuous migration job from the PostgreSQL source instance.

Create a new continuous migration job

In this step you will create a new continuous migration job.

In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click Database Migration > Migration jobs.
    
Click + Create Migration Job.
    
For Migration job name, enter vm-to-cloudsql.
    
For Source database engine, select PostgreSQL.
    
For Destination region, select .
    
For Destination database engine, select Cloud SQL for PostgreSQL.
    
For Migration job type, select Continuous.

Leave the defaults for the other settings.

Click Save & Continue.

Define the source instance

In this step, you will define the source instance for the migration.

For Source connection profile, select postgres-vm.

Leave the defaults for the other settings.

Click Save & Continue.

After you select the source connection profile, you can see its configuration details, including source hostname or IP address, port, username, and encryption type.
Create the destination instance

In this step, you will create the destination instance for the migration.

For Destination Instance ID, enter postgresql-cloudsql.
    
For Password, enter supersecret!.
    
For Choose a Cloud SQL edition, select Enterprise edition.
    
For Database version, select Cloud SQL for PostgreSQL 13.
    
In Choose region and zone section, select Single zone and select as primary zone.
    
For Instance connectivity, select Private IP and Public IP.
    
Select Use an automatically allocated IP range.

Leave the defaults for the other settings.

Click Allocate & Connect.

Leave the default option selected to use an automatically allocated IP range.

Note: This step may take a few minutes. If asked to retry the request, click the Retry button to refresh the Service Networking API.

When this step is complete, an updated message notifies you that the instance will use the existing managed service connection.

You will need to edit the pg_hba.conf file on the VM instance to allow access to the IP range that is automatically generated in point 5 of the previous step. You will do this in a later step before testing the migration configuration at the end of this task.

The updated message says that the instance will use the existing managed service connection.

Enter the additional information needed to create the destination instance on Cloud SQL.

For Machine shapes. check 1 vCPU, 3.75 GB
For Storage type, select SSD
For Storage capacity, select 10 GB
Click Create & Continue.

If prompted to confirm, click Create Destination & Continue. A message will state that your destination database instance is being created. Continue to the next step while you wait.

Define the connectivity method

In this step, you will define the connectivity method for the migration.


For Connectivity method, select VPC peering.
For VPC, select default.
VPC peering is configured by Database Migration Service using the information provided for the VPC network (the default network in this example).
When you see an updated message that the destination instance was created, proceed to the next step.

Click Configure & Continue.

Allow access to the posgresql-vm instance from automatically allocated IP range

In this step you will edit the pg_hba.conf PostgreSQL configuration file to allow the Database Migration Service to access the stand-alone PostgreSQL database.

Get the allocated IP address range. In the Google Cloud Console on the Navigation menu (Navigation menu icon), right-click VPC network > VPC network peering and open it in a new tab.
Click on the servicenetworking-googleapis-com entry.
In the Imported routes tab, select and copy the Destination IP range (e.g. 10.107.176.0/24).
In the Terminal session on the VM instance, edit the pg_hba.conf file as follows:
Save and exit the nano editor with Ctrl-O, Enter, Ctrl-X
Restart the PostgreSQL service to make the changes take effect. In the VM instance Terminal session:

Test and start the continuous migration job
In this step, you will test and start the migration job.

In the Database Migration Service tab you open earlier, review the details of the migration job.
Click Test Job.
After a successful test, click Create & Start Job.

```sh
student-03-5f77cf67d70a@postgresql-vm:~$ sudo nano /etc/postgresql/13/main/pg_hba.conf
student-03-5f77cf67d70a@postgresql-vm:~$ cat /etc/postgresql/13/main/pg_hba.conf
cat: /etc/postgresql/13/main/pg_hba.conf: Permission denied
student-03-5f77cf67d70a@postgresql-vm:~$ sudo cat /etc/postgresql/13/main/pg_hba.conf
# PostgreSQL Client Authentication Configuration File
# ===================================================
#
# Refer to the "Client Authentication" section in the PostgreSQL
# documentation for a complete description of this file.  A short
# synopsis follows.
#
# This file controls: which hosts are allowed to connect, how clients
# are authenticated, which PostgreSQL user names they can use, which
# databases they can access.  Records take one of these forms:
#
# local         DATABASE  USER  METHOD  [OPTIONS]
# host          DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostssl       DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnossl     DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostgssenc    DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnogssenc  DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
#
# (The uppercase items must be replaced by actual values.)
#
# The first field is the connection type: "local" is a Unix-domain
# socket, "host" is either a plain or SSL-encrypted TCP/IP socket,
# "hostssl" is an SSL-encrypted TCP/IP socket, and "hostnossl" is a
# non-SSL TCP/IP socket.  Similarly, "hostgssenc" uses a
# GSSAPI-encrypted TCP/IP socket, while "hostnogssenc" uses a
# non-GSSAPI socket.
#
# DATABASE can be "all", "sameuser", "samerole", "replication", a
# database name, or a comma-separated list thereof. The "all"
# keyword does not match "replication". Access to replication
# must be enabled in a separate record (see example below).
#
# USER can be "all", a user name, a group name prefixed with "+", or a
# comma-separated list thereof.  In both the DATABASE and USER fields
# you can also write a file name prefixed with "@" to include names
# from a separate file.
#
# ADDRESS specifies the set of hosts the record matches.  It can be a
# host name, or it is made up of an IP address and a CIDR mask that is
# an integer (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that
# specifies the number of significant bits in the mask.  A host name
# that starts with a dot (.) matches a suffix of the actual host name.
# Alternatively, you can write an IP address and netmask in separate
# columns to specify the set of hosts.  Instead of a CIDR-address, you
# can write "samehost" to match any of the server's own IP addresses,
# or "samenet" to match any address in any subnet that the server is
# directly connected to.
#
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
# "gss", "sspi", "ident", "peer", "pam", "ldap", "radius" or "cert".
# Note that "password" sends passwords in clear text; "md5" or
# "scram-sha-256" are preferred since they send encrypted passwords.
#
# OPTIONS are a set of options for the authentication in the format
# NAME=VALUE.  The available options depend on the different
# authentication methods -- refer to the "Client Authentication"
# section in the documentation for a list of which options are
# available for which authentication methods.
#
# Database and user names containing spaces, commas, quotes and other
# special characters must be quoted.  Quoting one of the keywords
# "all", "sameuser", "samerole" or "replication" makes the name lose
# its special character, and just match a database or username with
# that name.
#
# This file is read on server startup and when the server receives a
# SIGHUP signal.  If you edit the file on a running system, you have to
# SIGHUP the server for the changes to take effect, run "pg_ctl reload",
# or execute "SELECT pg_reload_conf()".
#
# Put your actual configuration here
# ----------------------------------
#
# If you want to allow non-local connections, you need to add more
# "host" records.  In that case you will also need to make PostgreSQL
# listen on a non-local interface via the listen_addresses
# configuration parameter, or via the -i or -h command line switches.




# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
#GSP918 - allow access to all hosts
host    all all 10.98.240.0/24    md5
student-03-5f77cf67d70a@postgresql-vm:~$ sudo systemctl start postgresql@13-main
student-03-5f77cf67d70a@postgresql-vm:~$ 
```
Review the status of the continuous migration job
In this step, you will confirm that the continuous migration job is running.
In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click Database Migration > Migration jobs.
Click the migration job vm-to-cloudsql to see the details page.
Review the migration job status.
 If you have not started the job, the status will show as Not started. You can choose to start or delete the job.
After the job has started, the status will show as Starting and then transition to Running Full dump in progress to indicate that the initial database dump is in progress.
After the initial database dump has been completed, the status will transition to Running CDC in progress to indicate that continuous migration is active.
When the job status changes to Running CDC in progress, proceed to the next task.

```sh
student-03-5f77cf67d70a@postgresql-vm:~$ gcloud sql connect postgresql-cloudsql --user=postgres --quiet
Allowlisting your IP for incoming connection for 5 minutes...done.                                                                                                                 
Connecting to database with SQL user [postgres].Password: 
psql (13.14 (Debian 13.14-1.pgdg110+2), server 13.13)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=> \c orders;
Password: 
psql (13.14 (Debian 13.14-1.pgdg110+2), server 13.13)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "orders" as user "postgres".
orders=> select * from distribution_centers;
 longitude | latitude |                    name                     | id 
-----------+----------+---------------------------------------------+----
 -89.9711  | 35.1174  | Memphis TN                                  |  1
 -87.6847  | 41.8369  | Chicago IL                                  |  2
 -95.3698  | 29.7604  | Houston TX                                  |  3
 -118.25   | 34.05    | Los Angeles CA                              |  4
 -90.0667  | 29.95    | New Orleans LA                              |  5
 -73.7834  | 40.634   | Port Authority of New York/New Jersey NY/NJ |  6
 -75.1667  | 39.95    | Philadelphia PA                             |  7
 -88.0431  | 30.6944  | Mobile AL                                   |  8
 -79.9333  | 32.7833  | Charleston SC                               |  9
 -81.1167  | 32.0167  | Savannah GA                                 | 10
(10 rows)

orders=> \q
student-03-5f77cf67d70a@postgresql-vm:~$ exit
logout
Connection to 34.148.178.231 closed.
student_03_5f77cf67d70a@cloudshell:~ (qwiklabs-gcp-01-a78ba407065a)$ export VM_NAME=postgresql-vm
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export POSTGRESQL_IP=$(gcloud compute instances describe ${VM_NAME} \
  --zone=us-east1-d --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
echo $POSTGRESQL_IP
psql -h $POSTGRESQL_IP -p 5432 -d orders -U migration_admin
34.148.178.231
Password for user migration_admin: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), server 13.14 (Debian 13.14-1.pgdg110+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

orders=> \c orders;
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), server 13.14 (Debian 13.14-1.pgdg110+2))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "orders" as user "migration_admin".
orders=> insert into distribution_centers values(-80.1918,25.7617,'Miami FL',11);
INSERT 0 1
orders=> \q
student_03_5f77cf67d70a@cloudshell:~ (qwiklabs-gcp-01-a78ba407065a)$ gcloud sql connect postgresql-cloudsql --user=postgres --quiet
Allowlisting your IP for incoming connection for 5 minutes...done.                                                                                                                 
Connecting to database with SQL user [postgres].Password: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), server 13.13)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=> \c orders;
Password: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), server 13.13)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
You are now connected to database "orders" as user "postgres".
orders=> select * from distribution_centers;
 longitude | latitude |                    name                     | id 
-----------+----------+---------------------------------------------+----
 -89.9711  | 35.1174  | Memphis TN                                  |  1
 -87.6847  | 41.8369  | Chicago IL                                  |  2
 -95.3698  | 29.7604  | Houston TX                                  |  3
 -118.25   | 34.05    | Los Angeles CA                              |  4
 -90.0667  | 29.95    | New Orleans LA                              |  5
 -73.7834  | 40.634   | Port Authority of New York/New Jersey NY/NJ |  6
 -75.1667  | 39.95    | Philadelphia PA                             |  7
 -88.0431  | 30.6944  | Mobile AL                                   |  8
 -79.9333  | 32.7833  | Charleston SC                               |  9
 -81.1167  | 32.0167  | Savannah GA                                 | 10
 -80.1918  | 25.7617  | Miami FL                                    | 11
(11 rows)

orders=> \q
student_03_5f77cf67d70a@cloudshell:~ (qwiklabs-gcp-01-a78ba407065a)$ 
```
Promote Cloud SQL to be a stand-alone instance for reading and writing data

In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click Database Migration > Migration jobs.
Click the migration job name vm-to-cloudsql to see the details page.
Click Promote.

If prompted to confirm, click Promote.

When the promotion is complete, the status of the job will update to Completed.
