---

layout: single
title:  "Migrate a MySQL Database to Google Cloud SQL"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-gke.png
  og_image: /assets/images/gcp-gke.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Migrate a MySQL Database to Google Cloud SQL

## Challenge scenario

Your WordPress blog is running on a server that is no longer  suitable. As the first part of a complete migration exercise, you are  migrating the locally hosted database used by the blog to Cloud SQL.

The existing WordPress installation is installed in the `/var/www/html/wordpress` directory in the instance called `blog` that is already running in the lab. You can access the blog by opening a web browser and pointing to the external IP address of the blog  instance.

The existing database for the blog is provided by MySQL running on the same server. The existing MySQL database is called `wordpress` and the user called **blogadmin** with password **Password1\***, which provides full access to that database.

### Your challenge

1. You need to create a new Cloud SQL instance to host the migrated database.
2. Once you have created the new database and configured it, you can  then create a database dump of the existing database and import it into  Cloud SQL.
3. When the data has been migrated, you will then reconfigure the blog software to use the migrated database.

For this lab, the WordPress site configuration file is located here: `/var/www/html/wordpress/wp-config.php`.

To sum it all up, your challenge is to migrate the database to Cloud  SQL and then reconfigure the application so that it no longer relies on  the local MySQL database.



```sh

student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"                      
Linux blog 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 28 05:37:42 2024 from 34.87.149.242
student-03-2a004b33a597@blog:~$ ls
wordpress.sql
student-03-2a004b33a597@blog:~$ pwd
/home/student-03-2a004b33a597
student-03-2a004b33a597@blog:~$ exit
logout
Connection to 34.66.17.32 closed.
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ 

student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ gcloud compute scp --zone "us-central1-c" blog:/home/student-03-2a004b33a597/wordpress.sql .
wordpress.sql                                                                                              100%   48KB  77.1KB/s   00:00    
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ ls
README-cloudshell.txt  wordpress.sql
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ gsutil cp ~/wordpress.sql gs://${PROJECT_ID}
Copying file:///home/student_03_2a004b33a597/wordpress.sql [Content-Type=application/sql]...
- [1 files][ 47.6 KiB/ 47.6 KiB]                                                
Operation completed over 1 objects/47.6 KiB.                                     
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ gcloud compute instances list
NAME: blog
ZONE: us-central1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.3
EXTERNAL_IP: 34.66.17.32
STATUS: RUNNING

NAME: lab-monitor
ZONE: us-central1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.133.202.209
STATUS: RUNNING
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"
Linux blog 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 28 05:48:08 2024 from 34.87.149.242
student-03-2a004b33a597@blog:~$ sudo service mysql status
● mariadb.service - MariaDB 10.5.23 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-05-28 04:57:10 UTC; 1h 0min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 3063 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 8 (limit: 4691)
     Memory: 79.1M
        CPU: 1.699s
     CGroup: /system.slice/mariadb.service
             └─3063 /usr/sbin/mariadbd

student-03-2a004b33a597@blog:~$ sudo service mysql stop
student-03-2a004b33a597@blog:~$ cd /var/www/html/wordpress
student-03-2a004b33a597@blog:/var/www/html/wordpress$ ls
index.php    wp-activate.php     wp-comments-post.php  wp-content   wp-links-opml.php  wp-mail.php      wp-trackback.php
license.txt  wp-admin            wp-config-sample.php  wp-cron.php  wp-load.php        wp-settings.php  xmlrpc.php
readme.html  wp-blog-header.php  wp-config.php         wp-includes  wp-login.php       wp-signup.php
student-03-2a004b33a597@blog:/var/www/html/wordpress$ sudo nano wp-config.php
student-03-2a004b33a597@blog:/var/www/html/wordpress$ sudo service apache2 restart
student-03-2a004b33a597@blog:/var/www/html/wordpress$ sudo service apache2 status
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-05-28 05:58:39 UTC; 8s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 13580 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 13586 (apache2)
      Tasks: 6 (limit: 4691)
     Memory: 15.5M
        CPU: 187ms
     CGroup: /system.slice/apache2.service
             ├─13586 /usr/sbin/apache2 -k start
             ├─13587 /usr/sbin/apache2 -k start
             ├─13588 /usr/sbin/apache2 -k start
student-03-2a004b33a597@blog:/var/www/html/wordpress$ history 
    1  mysqldump --databases wordpress -h localhost -u blogadmin -p --hex-blob --skip-triggers --single-transaction --default-character-set=utf8mb4 > wordpress.sql
    2  ls
    3  export PROJECT_ID=$(gcloud info --format='value(config.project)')
    4  gsutil mb gs://${PROJECT_ID}
    5  gsutil cp ~/wordpress.sql gs://${PROJECT_ID}
    6  gsutil mb gs://${PROJECT_ID}
    7  exit
    8  sutil cp ~/wordpress.sql gs://${PROJECT_ID}
    9  gsutil cp ~/wordpress.sql gs://${PROJECT_ID}
   10  ls
   11  pwd
   12  exit
   13  ls
   14  exit
   15  sudo service mysql status
   16  sudo service mysql stop
   17  cd /var/www/html/wordpress
   18  ls
   19  sudo nano wp-config.php
   20  sudo service apache2 restart
   21  sudo service apache2 status
   22  history 
student-03-2a004b33a597@blog:/var/www/html/wordpress$ exit
logout
Connection to 34.66.17.32 closed.
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ history 
    1  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"
    2  export PROJECT_ID=$(gcloud info --format='value(config.project)')
    3  gsutil mb gs://${PROJECT_ID}
    4  gsutil cp ~/wordpress.sql gs://${PROJECT_ID}
    5  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"
    6  scp 34.66.17.32:/home/student-03-2a004b33a597/wordpress.sql .
    7  ls
    8  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2" scp /home/student-03-2a004b33a597/wordpress.sql .
    9  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2" 
   10  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"
   11  gcloud compute scp --recurse blog:/home/student-03-2a004b33a597/wordpress.sql .
   12  gcloud compute scp --zone "us-central1-c" blog:/home/student-03-2a004b33a597/wordpress.sql .
   13  ls
   14  gsutil cp ~/wordpress.sql gs://${PROJECT_ID}
   15  gcloud compute instances list
   16  gcloud compute ssh --zone "us-central1-c" "blog" --project "qwiklabs-gcp-04-2d8ba71b6ea2"
   17  history 
student_03_2a004b33a597@cloudshell:~ (qwiklabs-gcp-04-2d8ba71b6ea2)$ 
```

