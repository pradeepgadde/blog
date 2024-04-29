---

layout: single
title:  "Monitoring a Compute Engine by using Ops Agent"
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

# Monitoring a Compute Engine by using Ops Agent

- Create a Compute Engine VM instance.
- Install an Apache Web Server.
- Install and configure the Ops Agent for the Apache Web Server.
- Generate traffic and view metrics on the predefined Apache dashboard.
- Create an alerting policy.

## Create a Compute Engine VM instance

1. In Google Cloud console, go to **Compute** and then select **Compute Engine**.
2. To create a VM instance, click **Create instance**.
3. Fill in the fields for your instance as follows:

- In the **Name** field, enter `quickstart-vm`.
- In the **Machine type** field, select **e2-small**.
- Ensure the **Boot disk** is configured for **Debian GNU/Linux**.
- In the **Firewall** field, select both **Allow HTTP traffic** and **Allow HTTPS traffic**.

Leave the rest of the fields at their default values.

1. Click **Create**. When your VM is ready, it appears in the list of instances in the Instances tab.

## Install an Apache Web Server

To deploy an Apache Web Server on your Compute Engine VM instance, do the following:

1. To open a terminal to your instance, in the **Connect** column, click **SSH**.

2. To update the package lists on your instance, run the following command:

   ```sh
   Welcome to Cloud Shell! Type "help" to get started.
   To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
   student_01_3c1bb184aaad@cloudshell:~$ gcloud compute ssh --zone "us-east4-c" "quickstart-vm" --project "qwiklabs-gcp-01-e534366a5a4c"
   WARNING: The private SSH key file for gcloud does not exist.
   WARNING: The public SSH key file for gcloud does not exist.
   WARNING: You do not have an SSH key for gcloud.
   WARNING: SSH keygen will be executed to generate a key.
   This tool needs to create the directory [/home/student_01_3c1bb184aaad/.ssh] before being able to generate SSH keys.
   
   Do you want to continue (Y/n)?  y
   
   Generating public/private rsa key pair.
   Enter passphrase (empty for no passphrase): 
   Enter same passphrase again: 
   Your identification has been saved in /home/student_01_3c1bb184aaad/.ssh/google_compute_engine
   Your public key has been saved in /home/student_01_3c1bb184aaad/.ssh/google_compute_engine.pub
   The key fingerprint is:
   SHA256:hjWINavTTZy+n1Qvo3/RccV4gvYqqJyGMTLKJ8aMnS8 student_01_3c1bb184aaad@cs-344282503593-default
   The key's randomart image is:
   +---[RSA 3072]----+
   |      o      . o |
   |     o = .  o o +|
   |    . o *  . . o.|
   |     o * .    ...|
   |    o o S.  .. .o|
   |  o o. ........ .|
   |=o + = o. ..o .. |
   |o*E.. =  o o o.  |
   |. oo..    +...   |
   +----[SHA256]-----+
   Warning: Permanently added 'compute.8333915369857259538' (ED25519) to the list of known hosts.
   Linux quickstart-vm.us-east4-c.c.qwiklabs-gcp-01-e534366a5a4c.internal 6.1.0-20-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.85-1 (2024-04-11) x86_64
   
   The programs included with the Debian GNU/Linux system are free software;
   the exact distribution terms for each program are described in the
   individual files in /usr/share/doc/*/copyright.
   
   Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
   permitted by applicable law.
   Creating directory '/home/student-01-3c1bb184aaad'.
   student-01-3c1bb184aaad@quickstart-vm:~$ 
   student-01-3c1bb184aaad@quickstart-vm:~$ sudo apt-get update
   Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
   Get:5 file:/etc/apt/mirrors/debian-security.list Mirrorlist [39 B]
   Get:2 https://deb.debian.org/debian bookworm InRelease [151 kB]
   Get:7 https://packages.cloud.google.com/apt google-compute-engine-bookworm-stable InRelease [5146 B]
   Get:3 https://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
   Get:4 https://deb.debian.org/debian bookworm-backports InRelease [56.5 kB]
   Get:6 https://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
   Get:8 https://packages.cloud.google.com/apt cloud-sdk-bookworm InRelease [6406 B]
   Get:9 https://packages.cloud.google.com/apt google-compute-engine-bookworm-stable/main amd64 Packages [1916 B]
   Get:10 https://deb.debian.org/debian bookworm-updates/main Sources.diff/Index [10.6 kB]
   Get:11 https://deb.debian.org/debian bookworm-updates/main amd64 Packages.diff/Index [10.6 kB]
   Get:12 https://deb.debian.org/debian bookworm-updates/main Translation-en.diff/Index [10.6 kB]
   Get:13 https://deb.debian.org/debian bookworm-updates/main Sources T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [831 B]
   Get:14 https://deb.debian.org/debian bookworm-updates/main amd64 Packages T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [1595 B]
   Get:13 https://deb.debian.org/debian bookworm-updates/main Sources T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [831 B]
   Get:14 https://deb.debian.org/debian bookworm-updates/main amd64 Packages T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [1595 B]
   Get:18 https://deb.debian.org/debian bookworm-updates/main Translation-en T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [2563 B]
   Get:18 https://deb.debian.org/debian bookworm-updates/main Translation-en T-2024-04-23-2036.10-F-2024-04-23-2036.10.pdiff [2563 B]
   Get:15 https://deb.debian.org/debian bookworm-backports/main Sources.diff/Index [63.3 kB]
   Get:16 https://deb.debian.org/debian bookworm-backports/main amd64 Packages.diff/Index [63.3 kB]
   Get:17 https://deb.debian.org/debian bookworm-backports/main Translation-en.diff/Index [63.3 kB]
   Get:22 https://deb.debian.org/debian bookworm-backports/main Sources T-2024-04-28-0805.24-F-2024-04-15-2018.42.pdiff [11.5 kB]
   Get:22 https://deb.debian.org/debian bookworm-backports/main Sources T-2024-04-28-0805.24-F-2024-04-15-2018.42.pdiff [11.5 kB]
   Get:23 https://packages.cloud.google.com/apt cloud-sdk-bookworm/main amd64 Packages [480 kB]
   Get:24 https://deb.debian.org/debian bookworm-backports/main amd64 Packages T-2024-04-28-1408.42-F-2024-04-15-2018.42.pdiff [9682 B]
   Get:24 https://deb.debian.org/debian bookworm-backports/main amd64 Packages T-2024-04-28-1408.42-F-2024-04-15-2018.42.pdiff [9682 B]
   Get:25 https://deb.debian.org/debian bookworm-backports/main Translation-en T-2024-04-25-1406.51-F-2024-04-19-2009.49.pdiff [7601 B]
   Get:25 https://deb.debian.org/debian bookworm-backports/main Translation-en T-2024-04-25-1406.51-F-2024-04-19-2009.49.pdiff [7601 B]
   Get:19 https://deb.debian.org/debian-security bookworm-security/main Sources [91.6 kB]
   Get:20 https://deb.debian.org/debian-security bookworm-security/main amd64 Packages [155 kB]
   Get:21 https://deb.debian.org/debian-security bookworm-security/main Translation-en [94.3 kB]
   Fetched 1400 kB in 1s (1112 kB/s)                         
   Reading package lists... Done
   student-01-3c1bb184aaad@quickstart-vm:~$ sudo apt-get install apache2 php7.0
   Reading package lists... Done
   Building dependency tree... Done
   Reading state information... Done
   Note, selecting 'php7.0-thrift' for regex 'php7.0'
   Note, selecting 'php7.0-common' for regex 'php7.0'
   Note, selecting 'php7.0-curl' for regex 'php7.0'
   The following additional packages will be installed:
     apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libjansson4 liblua5.3-0 ssl-cert
   Suggested packages:
     apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser
   The following NEW packages will be installed:
     apache2 apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap libjansson4 liblua5.3-0
     ssl-cert
   0 upgraded, 11 newly installed, 0 to remove and 4 not upgraded.
   Need to get 2367 kB of archives.
   After this operation, 8454 kB of additional disk space will be used.
   Do you want to continue? [Y/n] y
   Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
   Get:2 file:/etc/apt/mirrors/debian-security.list Mirrorlist [39 B]
   Get:3 https://deb.debian.org/debian bookworm/main amd64 libapr1 amd64 1.7.2-3 [102 kB]
   Get:4 https://deb.debian.org/debian bookworm/main amd64 libaprutil1 amd64 1.6.3-1 [87.8 kB]
   Get:5 https://deb.debian.org/debian bookworm/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.3-1 [13.6 kB]
   Get:6 https://deb.debian.org/debian bookworm/main amd64 libaprutil1-ldap amd64 1.6.3-1 [11.8 kB]
   Get:7 https://deb.debian.org/debian bookworm/main amd64 libjansson4 amd64 2.14-2 [40.8 kB]
   Get:8 https://deb.debian.org/debian bookworm/main amd64 liblua5.3-0 amd64 5.3.6-2 [123 kB]
   Get:9 https://deb.debian.org/debian-security bookworm-security/main amd64 apache2-bin amd64 2.4.59-1~deb12u1 [1380 kB]
   Get:10 https://deb.debian.org/debian-security bookworm-security/main amd64 apache2-data all 2.4.59-1~deb12u1 [160 kB]
   Get:11 https://deb.debian.org/debian-security bookworm-security/main amd64 apache2-utils amd64 2.4.59-1~deb12u1 [207 kB]
   Get:12 https://deb.debian.org/debian-security bookworm-security/main amd64 apache2 amd64 2.4.59-1~deb12u1 [220 kB]
   Get:13 https://deb.debian.org/debian bookworm/main amd64 ssl-cert all 1.1.2 [21.1 kB]
   Fetched 2367 kB in 0s (20.6 MB/s)    
   Preconfiguring packages ...
   Selecting previously unselected package libapr1:amd64.
   (Reading database ... 67169 files and directories currently installed.)
   Preparing to unpack .../00-libapr1_1.7.2-3_amd64.deb ...
   Unpacking libapr1:amd64 (1.7.2-3) ...
   Selecting previously unselected package libaprutil1:amd64.
   Preparing to unpack .../01-libaprutil1_1.6.3-1_amd64.deb ...
   Unpacking libaprutil1:amd64 (1.6.3-1) ...
   Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
   Preparing to unpack .../02-libaprutil1-dbd-sqlite3_1.6.3-1_amd64.deb ...
   Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.3-1) ...
   Selecting previously unselected package libaprutil1-ldap:amd64.
   Preparing to unpack .../03-libaprutil1-ldap_1.6.3-1_amd64.deb ...
   Unpacking libaprutil1-ldap:amd64 (1.6.3-1) ...
   Selecting previously unselected package libjansson4:amd64.
   Preparing to unpack .../04-libjansson4_2.14-2_amd64.deb ...
   Unpacking libjansson4:amd64 (2.14-2) ...
   Selecting previously unselected package liblua5.3-0:amd64.
   Preparing to unpack .../05-liblua5.3-0_5.3.6-2_amd64.deb ...
   Unpacking liblua5.3-0:amd64 (5.3.6-2) ...
   Selecting previously unselected package apache2-bin.
   Preparing to unpack .../06-apache2-bin_2.4.59-1~deb12u1_amd64.deb ...
   Unpacking apache2-bin (2.4.59-1~deb12u1) ...
   Selecting previously unselected package apache2-data.
   Preparing to unpack .../07-apache2-data_2.4.59-1~deb12u1_all.deb ...
   Unpacking apache2-data (2.4.59-1~deb12u1) ...
   Selecting previously unselected package apache2-utils.
   Preparing to unpack .../08-apache2-utils_2.4.59-1~deb12u1_amd64.deb ...
   Unpacking apache2-utils (2.4.59-1~deb12u1) ...
   Selecting previously unselected package apache2.
   Preparing to unpack .../09-apache2_2.4.59-1~deb12u1_amd64.deb ...
   Unpacking apache2 (2.4.59-1~deb12u1) ...
   Selecting previously unselected package ssl-cert.
   Preparing to unpack .../10-ssl-cert_1.1.2_all.deb ...
   Unpacking ssl-cert (1.1.2) ...
   Setting up libapr1:amd64 (1.7.2-3) ...
   Setting up libjansson4:amd64 (2.14-2) ...
   Setting up ssl-cert (1.1.2) ...
   Setting up liblua5.3-0:amd64 (5.3.6-2) ...
   Setting up apache2-data (2.4.59-1~deb12u1) ...
   Setting up libaprutil1:amd64 (1.6.3-1) ...
   Setting up libaprutil1-ldap:amd64 (1.6.3-1) ...
   Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.3-1) ...
   Setting up apache2-utils (2.4.59-1~deb12u1) ...
   Setting up apache2-bin (2.4.59-1~deb12u1) ...
   Setting up apache2 (2.4.59-1~deb12u1) ...
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
   Processing triggers for man-db (2.11.2-2) ...
   Processing triggers for libc-bin (2.36-9+deb12u4) ...
   student-01-3c1bb184aaad@quickstart-vm:~$ 
   ```

   

Open your browser and connect to your Apache2 HTTP server by using the URL `http://EXTERNAL_IP`, where `EXTERNAL_IP` is the external IP address of your VM. You can find this address in the **External IP** column of your VM instance.

## Install and configure the Ops Agent

To collect logs and metrics from your Apache Web Server, install the [Ops Agent](https://cloud.google.com/logging/docs/agent/ops-agent) by using the terminal:

```sh
student-01-3c1bb184aaad@quickstart-vm:~$ curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:5 file:/etc/apt/mirrors/debian-security.list Mirrorlist [39 B]                      
Hit:2 https://deb.debian.org/debian bookworm InRelease       
Hit:7 https://packages.cloud.google.com/apt google-compute-engine-bookworm-stable InRelease
Hit:3 https://deb.debian.org/debian bookworm-updates InRelease
Hit:4 https://deb.debian.org/debian bookworm-backports InRelease
Hit:6 https://deb.debian.org/debian-security bookworm-security InRelease
Hit:8 https://packages.cloud.google.com/apt cloud-sdk-bookworm InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 4 not upgraded.
Need to get 25.2 kB of archives.
After this operation, 35.8 kB of additional disk space will be used.
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:2 https://deb.debian.org/debian bookworm/main amd64 apt-transport-https all 2.6.1 [25.2 kB]
Fetched 25.2 kB in 0s (371 kB/s)          
Selecting previously unselected package apt-transport-https.
(Reading database ... 67904 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.6.1_all.deb ...
Unpacking apt-transport-https (2.6.1) ...
Setting up apt-transport-https (2.6.1) ...
Adding agent repository for debian.
deb https://packages.cloud.google.com/apt google-cloud-ops-agent-bookworm-all main
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
0 upgraded, 0 newly installed, 0 to remove and 4 not upgraded.
Get:1 file:/etc/apt/mirrors/debian.list Mirrorlist [30 B]
Get:5 file:/etc/apt/mirrors/debian-security.list Mirrorlist [39 B]                      
Hit:2 https://deb.debian.org/debian bookworm InRelease       
Hit:3 https://deb.debian.org/debian bookworm-updates InRelease
Hit:4 https://deb.debian.org/debian bookworm-backports InRelease
Hit:6 https://deb.debian.org/debian-security bookworm-security InRelease
Get:7 https://packages.cloud.google.com/apt google-cloud-ops-agent-bookworm-all InRelease [5112 B]
Hit:8 https://packages.cloud.google.com/apt google-compute-engine-bookworm-stable InRelease
Hit:9 https://packages.cloud.google.com/apt cloud-sdk-bookworm InRelease
Get:10 https://packages.cloud.google.com/apt google-cloud-ops-agent-bookworm-all/main amd64 Packages [2798 B]
Fetched 7910 B in 1s (7437 B/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  google-cloud-ops-agent
0 upgraded, 1 newly installed, 0 to remove and 4 not upgraded.
Need to get 94.7 MB of archives.
After this operation, 415 MB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt google-cloud-ops-agent-bookworm-all/main amd64 google-cloud-ops-agent amd64 2.46.1~debian12 [94.7 MB]
Fetched 94.7 MB in 3s (33.3 MB/s)                  
Selecting previously unselected package google-cloud-ops-agent.
(Reading database ... 67908 files and directories currently installed.)
Preparing to unpack .../google-cloud-ops-agent_2.46.1~debian12_amd64.deb ...
Unpacking google-cloud-ops-agent (2.46.1~debian12) ...
Setting up google-cloud-ops-agent (2.46.1~debian12) ...
Created symlink /etc/systemd/system/multi-user.target.wants/google-cloud-ops-agent.service → /lib/systemd/system/google-cloud-ops-agent.service.
google-cloud-ops-agent  installation succeeded.
student-01-3c1bb184aaad@quickstart-vm:~$ 
```

```sh
student-01-3c1bb184aaad@quickstart-vm:~$ # Configures Ops Agent to collect telemetry from the app and restart Ops Agent.

set -e

# Create a back up of the existing file so existing configurations are not lost.
sudo cp /etc/google-cloud-ops-agent/config.yaml /etc/google-cloud-ops-agent/config.yaml.bak

# Configure the Ops Agent.
sudo tee /etc/google-cloud-ops-agent/config.yaml > /dev/null << EOF
metrics:
  receivers:
    apache:
      type: apache
  service:
sleep 60vice google-cloud-ops-agent restart

student-01-3c1bb184aaad@quickstart-vm:~$ 
student-01-3c1bb184aaad@quickstart-vm:~$ 
```

```yaml
student-01-3c1bb184aaad@quickstart-vm:~$ cat /etc/google-cloud-ops-agent/config.yaml 
metrics:
  receivers:
    apache:
      type: apache
  service:
    pipelines:
      apache:
        receivers:
          - apache
logging:
  receivers:
    apache_access:
      type: apache_access
    apache_error:
      type: apache_error
  service:
    pipelines:
      apache:
        receivers:
          - apache_access
          - apache_error
student-01-3c1bb184aaad@quickstart-vm:~$ 
```

The previous command creates the configuration to collect and ingest logs and metrics from the Apache Web Server. 

## Generate traffic and view metrics

Monitoring dashboards let you view and analyze metrics related to  your services. In this quickstart, you generate metrics on your Apache  Web Server and view metric data on the automatically created **Apache GCE Overview** dashboard.

To generate metrics on your Apache Web Server, do the following:

1. In the Google Cloud console, go to **Compute Engine**.

2. In the **Connect** column, click **SSH** to open a terminal to your VM instance.

3. To generate traffic on your Apache Web Server, run the following command:

   ```sh
   timeout 120 bash -c -- 'while true; do curl localhost; sleep $((RANDOM % 4)) ; done'
   ```

   The previous command generates traffic by making a request to the Apache Web Server every four seconds.

   To view the **Apache GCE Overview** dashboard, do the following:

   1. In the Google Cloud console, search for **Monitoring** in the top search bar and navigate to the **Monitoring** service.
   2. In the navigation pane, select **Dashboards**.
   3. In **All Dashboards**, select the **Apache Overview** dashboard. The dashboard opens.

   In the dashboard, there are several charts that contain information about your Apache and Compute Engine integration:

   ## Create an alerting policy

   1. To set up an email notification channel, do the following:

   - In the Google Cloud console > **Monitoring** select **Alerting** and then click **Edit notification channels**.
   - In the **Email section**, click Add new and enter your desired Email Address.
   - **Name the Email Channel**: `An email address you have access to`

   To create an alerting policy that monitors a metric and sends an  email notification when the traffic rate on your Apache Web Server  exceeds 4 KiB/s, do the following:

1. In the Google Cloud console > **Monitoring** select **Alerting** and then click **Create policy**.
2. Select the time series to be monitored:

- Click **Select a metric** and enter **VM instance** into the filter bar.
- In the **Active metric categories** list, select **Apache**.
- In the **Active metrics** list, select **workload/apache.traffic**.
- Click **Apply**.

The chart for Apache traffic is shown.

1. n the **Transform data** section, select the following values and click **Next**:

- **Rolling window**: `1 min`
- **Rolling window function**: `rate`

1. In the **Configure alert trigger** section, select the following values and click **Next**:

- **Alert trigger**: `Any time series violates`
- **Threshold position**: `Above threshold`
- **Threshold value**: `4000`

1. In the **Configure notifications and finalize alert** section, select the following values:
2. **Notification channels**: `An email address you have access to`
3. **Incident autoclose duration**: `30 min`
4. **Name the alert policy**: `Apache traffic above threshold`

1. Click **Create policy**. Your alerting policy is now active.

## Test the alerting policy

To test the alerting policy you just created, do the following:

1. Navigate to Cloud Console > **Compute Engine**.

2. In the **Connect** column, click **SSH** to open a terminal to your VM instance.

3. In the terminal, enter the following command:

   ```sh
   timeout 120 bash -c -- 'while true; do curl localhost; sleep $((RANDOM % 4)) ; done'
   ```

   The previous command generates traffic in your Apache Web Server.

   After the traffic rate threshold value of 4 KiB/s is exceeded in your Apache Web Server, an email notification is sent. It might take several minutes for this process to complete.

   

   ## Condition

   VM Instance - workload/apache.traffic

   ### Description

    Violates when: Any workload.googleapis.com/apache.traffic stream is above a threshold of 4000  

   ## Message

   workload/apache.traffic for qwiklabs-gcp-01-e534366a5a4c quickstart-vm with metric labels  {instrumentation_source=agent.googleapis.com/apache,  instrumentation_version=1.0, server_name=127.0.0.1} is above the  threshold of 4000.000 with a value of 4843.262.

   ## Documentation

   No documentation is configured.

   In this lab, you learned how to install Ops Agent on a VM and use it to  set an alerting policy to notify a recipient of potential issues with  the instance.
