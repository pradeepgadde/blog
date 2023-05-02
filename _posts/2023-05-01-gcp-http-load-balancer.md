---
layout: single
title:  "Configuring an HTTP Load Balancer with Autoscaling"
date:   2023-05-01 12:59:05 +0530
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

# Configuring an HTTP Load Balancer with Autoscaling

## Overview

Google Cloud HTTP(S) load balancing is implemented at the edge of  Google's network in Google's points of presence (POP) around the world.  User traffic directed to an HTTP(S) load balancer enters the POP closest to the user and is then load-balanced over Google's global network to  the closest backend that has sufficient available capacity.

In this lab, you configure an HTTP load balancer as shown in the  diagram below. Then, you stress test the load balancer to demonstrate  global load balancing and autoscaling.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-lb-1.png)

- Create a health check firewall rule
- Create a NAT configuration using Cloud Router
- Create a custom image for a web server
- Create an instance template based on the custom image
- Create two managed instance groups
- Configure an HTTP load balancer with IPv4 and IPv6
- Stress test an HTTP load balancer

## Task 1. Configure a health check firewall rule

Health checks determine which instances of a load balancer can  receive new connections. For HTTP load balancing, the health check  probes to your load-balanced instances come from addresses in the ranges **130.211.0.0/22** and **35.191.0.0/16**. Your firewall rules must allow these connections.



```sh
gcloud compute --project=qwiklabs-gcp-04-1059d35c6e27 firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-checks
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-lb-2.png)

## Task 2. Create a NAT configuration using Cloud Router

The Google Cloud VM backend instances that you set up in Task 3 will not be configured with external IP addresses.

Instead, you will set up the Cloud NAT service to allow these VM  instances to send outbound traffic only through the Cloud NAT, and  receive inbound traffic through the load balancer.

### Create the Cloud Router instance

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-lb-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-lb-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-lb-5.png)

## Task 3. Create a custom image for a web server

Create a custom web server image for the backend of the load balancer.

Equivalent Terraform configuration 

```sh
# This code is compatible with Terraform 4.25.0 and versions that are backwards compatible to 4.25.0.
# For information about validating this Terraform code, see https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/google-cloud-platform-build#format-and-validate-the-configuration

resource "google_compute_instance" "webserver" {
  boot_disk {
    auto_delete = false
    device_name = "webserver"

    initialize_params {
      image = "projects/debian-cloud/global/images/debian-10-buster-v20230411"
      size  = 10
      type  = "pd-balanced"
    }

    mode = "READ_WRITE"
  }

  can_ip_forward      = false
  deletion_protection = false
  enable_display      = false

  labels = {
    ec-src = "vm_add-tf"
  }

  machine_type = "e2-medium"

  metadata = {
    enable-oslogin = "true"
  }

  name = "webserver"

  network_interface {
    subnetwork = "projects/qwiklabs-gcp-04-1059d35c6e27/regions/us-central1/subnetworks/default"
  }

  scheduling {
    automatic_restart   = true
    on_host_maintenance = "MIGRATE"
    preemptible         = false
    provisioning_model  = "STANDARD"
  }

  service_account {
    email  = "425659184878-compute@developer.gserviceaccount.com"
    scopes = ["https://www.googleapis.com/auth/devstorage.read_only", "https://www.googleapis.com/auth/logging.write", "https://www.googleapis.com/auth/monitoring.write", "https://www.googleapis.com/auth/service.management.readonly", "https://www.googleapis.com/auth/servicecontrol", "https://www.googleapis.com/auth/trace.append"]
  }

  shielded_instance_config {
    enable_integrity_monitoring = true
    enable_secure_boot          = false
    enable_vtpm                 = true
  }

  tags = ["allow-health-checks"]
  zone = "us-central1-a"
}

```

On the Webserver VM

```sh
Linux webserver 4.19.0-23-cloud-amd64 #1 SMP Debian 4.19.269-1 (2022-12-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-01-a0075c49c713'.
student-01-a0075c49c713@webserver:~$ sudo apt-get update
Get:1 http://packages.cloud.google.com/apt google-compute-engine-buster-stable InRelease [5136 B]
Hit:2 http://deb.debian.org/debian buster InRelease                            
Get:3 http://packages.cloud.google.com/apt cloud-sdk-buster InRelease [6396 B]
Get:4 http://security.debian.org/debian-security buster/updates InRelease [34.8 kB]
Get:5 http://deb.debian.org/debian buster-updates InRelease [56.6 kB]
Get:6 http://deb.debian.org/debian buster-backports InRelease [51.4 kB]
Get:7 http://packages.cloud.google.com/apt google-compute-engine-buster-stable/main amd64 Packages [2239 B]
Get:8 http://packages.cloud.google.com/apt cloud-sdk-buster/main amd64 Packages [429 kB]
Get:9 http://security.debian.org/debian-security buster/updates/main Sources [330 kB]
Get:10 http://security.debian.org/debian-security buster/updates/main amd64 Packages [484 kB]
Get:11 http://security.debian.org/debian-security buster/updates/main Translation-en [263 kB]
Fetched 1662 kB in 1s (1716 kB/s)                         
Reading package lists... Done
student-01-a0075c49c713@webserver:~$ sudo apt-get install -y apache2
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
  libbrotli1 libgdbm-compat4 libicu63 libjansson4 liblua5.2-0 libperl5.28 libxml2 perl perl-modules-5.28
  ssl-cert
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser perl-doc libterm-readline-gnu-perl
  | libterm-readline-perl-perl make libb-debug-perl liblocale-codes-perl openssl-blacklist
The following NEW packages will be installed:
  apache2 apache2-bin apache2-data apache2-utils libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap
  libbrotli1 libgdbm-compat4 libicu63 libjansson4 liblua5.2-0 libperl5.28 libxml2 perl perl-modules-5.28
  ssl-cert
0 upgraded, 18 newly installed, 0 to remove and 5 not upgraded.
Need to get 18.6 MB of archives.
After this operation, 90.2 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian buster/main amd64 perl-modules-5.28 all 5.28.1-6+deb10u1 [2873 kB]
Get:2 http://security.debian.org/debian-security buster/updates/main amd64 libaprutil1 amd64 1.6.1-4+deb10u1 [91.9 kB]
Get:3 http://security.debian.org/debian-security buster/updates/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-4+deb10u1 [18.9 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main amd64 libaprutil1-ldap amd64 1.6.1-4+deb10u1 [17.0 kB]
Get:5 http://security.debian.org/debian-security buster/updates/main amd64 libxml2 amd64 2.9.4+dfsg1-7+deb10u6 [690 kB]
Get:6 http://deb.debian.org/debian buster/main amd64 libgdbm-compat4 amd64 1.18.1-4 [44.1 kB]
Get:7 http://deb.debian.org/debian buster/main amd64 libperl5.28 amd64 5.28.1-6+deb10u1 [3894 kB]
Get:8 http://security.debian.org/debian-security buster/updates/main amd64 apache2-bin amd64 2.4.38-3+deb10u10 [1310 kB]
Get:9 http://deb.debian.org/debian buster/main amd64 perl amd64 5.28.1-6+deb10u1 [204 kB]
Get:10 http://deb.debian.org/debian buster/main amd64 libapr1 amd64 1.6.5-1+b1 [102 kB]
Get:11 http://deb.debian.org/debian buster/main amd64 libbrotli1 amd64 1.0.7-2+deb10u1 [269 kB]
Get:12 http://deb.debian.org/debian buster/main amd64 libjansson4 amd64 2.12-1 [38.0 kB]
Get:13 http://deb.debian.org/debian buster/main amd64 liblua5.2-0 amd64 5.2.4-1.1+b2 [110 kB]
Get:14 http://deb.debian.org/debian buster/main amd64 libicu63 amd64 63.1-6+deb10u3 [8293 kB]
Get:15 http://security.debian.org/debian-security buster/updates/main amd64 apache2-data all 2.4.38-3+deb10u10 [165 kB]
Get:16 http://security.debian.org/debian-security buster/updates/main amd64 apache2-utils amd64 2.4.38-3+deb10u10 [237 kB]
Get:17 http://security.debian.org/debian-security buster/updates/main amd64 apache2 amd64 2.4.38-3+deb10u10 [252 kB]
Get:18 http://deb.debian.org/debian buster/main amd64 ssl-cert all 1.0.39 [20.8 kB]
Fetched 18.6 MB in 0s (44.3 MB/s)
Preconfiguring packages ...
Selecting previously unselected package perl-modules-5.28.
(Reading database ... 55537 files and directories currently installed.)
Preparing to unpack .../00-perl-modules-5.28_5.28.1-6+deb10u1_all.deb ...
Unpacking perl-modules-5.28 (5.28.1-6+deb10u1) ...
Selecting previously unselected package libgdbm-compat4:amd64.
Preparing to unpack .../01-libgdbm-compat4_1.18.1-4_amd64.deb ...
Unpacking libgdbm-compat4:amd64 (1.18.1-4) ...
Selecting previously unselected package libperl5.28:amd64.
Preparing to unpack .../02-libperl5.28_5.28.1-6+deb10u1_amd64.deb ...
Unpacking libperl5.28:amd64 (5.28.1-6+deb10u1) ...
Selecting previously unselected package perl.
Preparing to unpack .../03-perl_5.28.1-6+deb10u1_amd64.deb ...
Unpacking perl (5.28.1-6+deb10u1) ...
Selecting previously unselected package libapr1:amd64.
Preparing to unpack .../04-libapr1_1.6.5-1+b1_amd64.deb ...
Unpacking libapr1:amd64 (1.6.5-1+b1) ...
Selecting previously unselected package libaprutil1:amd64.
Preparing to unpack .../05-libaprutil1_1.6.1-4+deb10u1_amd64.deb ...
Unpacking libaprutil1:amd64 (1.6.1-4+deb10u1) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
Preparing to unpack .../06-libaprutil1-dbd-sqlite3_1.6.1-4+deb10u1_amd64.deb ...
Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-4+deb10u1) ...
Selecting previously unselected package libaprutil1-ldap:amd64.
Preparing to unpack .../07-libaprutil1-ldap_1.6.1-4+deb10u1_amd64.deb ...
Unpacking libaprutil1-ldap:amd64 (1.6.1-4+deb10u1) ...
Selecting previously unselected package libbrotli1:amd64.
Preparing to unpack .../08-libbrotli1_1.0.7-2+deb10u1_amd64.deb ...
Unpacking libbrotli1:amd64 (1.0.7-2+deb10u1) ...
Selecting previously unselected package libjansson4:amd64.
Preparing to unpack .../09-libjansson4_2.12-1_amd64.deb ...
Unpacking libjansson4:amd64 (2.12-1) ...
Selecting previously unselected package liblua5.2-0:amd64.
Preparing to unpack .../10-liblua5.2-0_5.2.4-1.1+b2_amd64.deb ...
Unpacking liblua5.2-0:amd64 (5.2.4-1.1+b2) ...
Selecting previously unselected package libicu63:amd64.
Preparing to unpack .../11-libicu63_63.1-6+deb10u3_amd64.deb ...
Unpacking libicu63:amd64 (63.1-6+deb10u3) ...
Selecting previously unselected package libxml2:amd64.
Preparing to unpack .../12-libxml2_2.9.4+dfsg1-7+deb10u6_amd64.deb ...
Unpacking libxml2:amd64 (2.9.4+dfsg1-7+deb10u6) ...
Selecting previously unselected package apache2-bin.
Preparing to unpack .../13-apache2-bin_2.4.38-3+deb10u10_amd64.deb ...
Unpacking apache2-bin (2.4.38-3+deb10u10) ...
Selecting previously unselected package apache2-data.
Preparing to unpack .../14-apache2-data_2.4.38-3+deb10u10_all.deb ...
Unpacking apache2-data (2.4.38-3+deb10u10) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../15-apache2-utils_2.4.38-3+deb10u10_amd64.deb ...
Unpacking apache2-utils (2.4.38-3+deb10u10) ...
Selecting previously unselected package apache2.
Preparing to unpack .../16-apache2_2.4.38-3+deb10u10_amd64.deb ...
Unpacking apache2 (2.4.38-3+deb10u10) ...
Selecting previously unselected package ssl-cert.
Preparing to unpack .../17-ssl-cert_1.0.39_all.deb ...
Unpacking ssl-cert (1.0.39) ...
Setting up perl-modules-5.28 (5.28.1-6+deb10u1) ...
Setting up libbrotli1:amd64 (1.0.7-2+deb10u1) ...
Setting up libapr1:amd64 (1.6.5-1+b1) ...
Setting up libicu63:amd64 (63.1-6+deb10u3) ...
Setting up libjansson4:amd64 (2.12-1) ...
Setting up ssl-cert (1.0.39) ...
Setting up libgdbm-compat4:amd64 (1.18.1-4) ...
Setting up libperl5.28:amd64 (5.28.1-6+deb10u1) ...
Setting up liblua5.2-0:amd64 (5.2.4-1.1+b2) ...
Setting up apache2-data (2.4.38-3+deb10u10) ...
Setting up libxml2:amd64 (2.9.4+dfsg1-7+deb10u6) ...
Setting up libaprutil1:amd64 (1.6.1-4+deb10u1) ...
Setting up libaprutil1-ldap:amd64 (1.6.1-4+deb10u1) ...
Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-4+deb10u1) ...
Setting up perl (5.28.1-6+deb10u1) ...
Setting up apache2-utils (2.4.38-3+deb10u10) ...
Setting up apache2-bin (2.4.38-3+deb10u10) ...
Setting up apache2 (2.4.38-3+deb10u10) ...
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
Processing triggers for systemd (241-7~deb10u9) ...
Processing triggers for man-db (2.8.5-2) ...
Processing triggers for libc-bin (2.28-10+deb10u2) ...
student-01-a0075c49c713@webserver:~$ 
```

```html
student-01-a0075c49c713@webserver:~$ sudo service apache2 start
student-01-a0075c49c713@webserver:~$ curl localhost

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Debian Default Page: It works</title>
    <style type="text/css" media="screen">
  * {
    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
  }

  body, html {
    padding: 3px 3px 3px 3px;

    background-color: #D8DBE2;

    font-family: Verdana, sans-serif;
    font-size: 11pt;
    text-align: center;
  }

  div.main_page {
    position: relative;
    display: table;

    width: 800px;

    margin-bottom: 3px;
    margin-left: auto;
    margin-right: auto;
    padding: 0px 0px 0px 0px;

    border-width: 2px;
    border-color: #212738;
    border-style: solid;

    background-color: #FFFFFF;

    text-align: center;
  }

  div.page_header {
    height: 99px;
    width: 100%;

    background-color: #F5F6F7;
  }

  div.page_header span {
    margin: 15px 0px 0px 50px;

    font-size: 180%;
    font-weight: bold;
  }

  div.page_header img {
    margin: 3px 0px 0px 40px;

    border: 0px 0px 0px;
  }

  div.table_of_contents {
    clear: left;

    min-width: 200px;

    margin: 3px 3px 3px 3px;

    background-color: #FFFFFF;

    text-align: left;
  }

  div.table_of_contents_item {
    clear: left;

    width: 100%;

    margin: 4px 0px 0px 0px;

    background-color: #FFFFFF;

    color: #000000;
    text-align: left;
  }

  div.table_of_contents_item a {
    margin: 6px 0px 0px 6px;
  }

  div.content_section {
    margin: 3px 3px 3px 3px;

    background-color: #FFFFFF;

    text-align: left;
  }

  div.content_section_text {
    padding: 4px 8px 4px 8px;

    color: #000000;
    font-size: 100%;
  }

  div.content_section_text pre {
    margin: 8px 0px 8px 0px;
    padding: 8px 8px 8px 8px;

    border-width: 1px;
    border-style: dotted;
    border-color: #000000;

    background-color: #F5F6F7;

    font-style: italic;
  }

  div.content_section_text p {
    margin-bottom: 6px;
  }

  div.content_section_text ul, div.content_section_text li {
    padding: 4px 8px 4px 16px;
  }

  div.section_header {
    padding: 3px 6px 3px 6px;

    background-color: #8E9CB2;

    color: #FFFFFF;
    font-weight: bold;
    font-size: 112%;
    text-align: center;
  }

  div.section_header_red {
    background-color: #CD214F;
  }

  div.section_header_grey {
    background-color: #9F9386;
  }

  .floating_element {
    position: relative;
    float: left;
  }

  div.table_of_contents_item a,
  div.content_section_text a {
    text-decoration: none;
    font-weight: bold;
  }

  div.table_of_contents_item a:link,
  div.table_of_contents_item a:visited,
  div.table_of_contents_item a:active {
    color: #000000;
  }

  div.table_of_contents_item a:hover {
    background-color: #000000;

    color: #FFFFFF;
  }

  div.content_section_text a:link,
  div.content_section_text a:visited,
   div.content_section_text a:active {
    background-color: #DCDFE6;

    color: #000000;
  }

  div.content_section_text a:hover {
    background-color: #000000;

    color: #DCDFE6;
  }

  div.validator {
  }
    </style>
  </head>
  <body>
    <div class="main_page">
      <div class="page_header floating_element">
        <img src="/icons/openlogo-75.png" alt="Debian Logo" class="floating_element"/>
        <span class="floating_element">
          Apache2 Debian Default Page
        </span>
      </div>
<!--      <div class="table_of_contents floating_element">
        <div class="section_header section_header_grey">
          TABLE OF CONTENTS
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#about">About</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#changes">Changes</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#scope">Scope</a>
        </div>
        <div class="table_of_contents_item floating_element">
          <a href="#files">Config files</a>
        </div>
      </div>
-->
      <div class="content_section floating_element">


        <div class="section_header section_header_red">
          <div id="about"></div>
          It works!
        </div>
        <div class="content_section_text">
          <p>
                This is the default welcome page used to test the correct 
                operation of the Apache2 server after installation on Debian systems.
                If you can read this page, it means that the Apache HTTP server installed at
                this site is working properly. You should <b>replace this file</b> (located at
                <tt>/var/www/html/index.html</tt>) before continuing to operate your HTTP server.
          </p>


          <p>
                If you are a normal user of this web site and don't know what this page is
                about, this probably means that the site is currently unavailable due to
                maintenance.
                If the problem persists, please contact the site's administrator.
          </p>

        </div>
        <div class="section_header">
          <div id="changes"></div>
                Configuration Overview
        </div>
        <div class="content_section_text">
          <p>
                Debian's Apache2 default configuration is different from the
                upstream default configuration, and split into several files optimized for
                interaction with Debian tools. The configuration system is
                <b>fully documented in
                /usr/share/doc/apache2/README.Debian.gz</b>. Refer to this for the full
                documentation. Documentation for the web server itself can be
                found by accessing the <a href="/manual">manual</a> if the <tt>apache2-doc</tt>
                package was installed on this server.

          </p>
          <p>
                The configuration layout for an Apache2 web server installation on Debian systems is as follows:
          </p>
          <pre>
/etc/apache2/
|-- apache2.conf
|       `--  ports.conf
|-- mods-enabled
|       |-- *.load
|       `-- *.conf
|-- conf-enabled
|       `-- *.conf
|-- sites-enabled
|       `-- *.conf
          </pre>
          <ul>
                        <li>
                           <tt>apache2.conf</tt> is the main configuration
                           file. It puts the pieces together by including all remaining configuration
                           files when starting up the web server.
                        </li>

                        <li>
                           <tt>ports.conf</tt> is always included from the
                           main configuration file. It is used to determine the listening ports for
                           incoming connections, and this file can be customized anytime.
                        </li>

                        <li>
                           Configuration files in the <tt>mods-enabled/</tt>,
                           <tt>conf-enabled/</tt> and <tt>sites-enabled/</tt> directories contain
                           particular configuration snippets which manage modules, global configuration
                           fragments, or virtual host configurations, respectively.
                        </li>

                        <li>
                           They are activated by symlinking available
                           configuration files from their respective
                           *-available/ counterparts. These should be managed
                           by using our helpers
                           <tt>
                                a2enmod,
                                a2dismod,
                           </tt>
                           <tt>
                                a2ensite,
                                a2dissite,
                            </tt>
                                and
                           <tt>
                                a2enconf,
                                a2disconf
                           </tt>. See their respective man pages for detailed information.
                        </li>

                        <li>
                           The binary is called apache2. Due to the use of
                           environment variables, in the default configuration, apache2 needs to be
                           started/stopped with <tt>/etc/init.d/apache2</tt> or <tt>apache2ctl</tt>.
                           <b>Calling <tt>/usr/bin/apache2</tt> directly will not work</b> with the
                           default configuration.
                        </li>
          </ul>
        </div>

        <div class="section_header">
            <div id="docroot"></div>
                Document Roots
        </div>

        <div class="content_section_text">
            <p>
                By default, Debian does not allow access through the web browser to
                <em>any</em> file apart of those located in <tt>/var/www</tt>,
                <a href="http://httpd.apache.org/docs/2.4/mod/mod_userdir.html" rel="nofollow">public_html</a>
                directories (when enabled) and <tt>/usr/share</tt> (for web
                applications). If your site is using a web document root
                located elsewhere (such as in <tt>/srv</tt>) you may need to whitelist your
                document root directory in <tt>/etc/apache2/apache2.conf</tt>.
            </p>
            <p>
                The default Debian document root is <tt>/var/www/html</tt>. You
                can make your own virtual hosts under /var/www. This is different
                to previous releases which provides better security out of the box.
            </p>
        </div>

        <div class="section_header">
          <div id="bugs"></div>
                Reporting Problems
        </div>
        <div class="content_section_text">
          <p>
                Please use the <tt>reportbug</tt> tool to report bugs in the
                Apache2 package with Debian. However, check <a
                href="http://bugs.debian.org/cgi-bin/pkgreport.cgi?ordering=normal;archive=0;src=apache2;repeatmerged=0"
                rel="nofollow">existing bug reports</a> before reporting a new bug.
          </p>
          <p>
                Please report bugs specific to modules (such as PHP and others)
                to respective packages, not to the web server itself.
          </p>
        </div>




      </div>
    </div>
    <div class="validator">
    </div>
  </body>
</html>

student-01-a0075c49c713@webserver:~$ 
```

```sh
student-01-a0075c49c713@webserver:~$ sudo update-rc.d apache2 enable
student-01-a0075c49c713@webserver:~$ 
```

After resetting the VM

```sh
Linux webserver 4.19.0-23-cloud-amd64 #1 SMP Debian 4.19.269-1 (2022-12-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May  2 02:36:27 2023 from 35.235.241.19
student-01-a0075c49c713@webserver:~$ sudo service apache2 status
● apache2.service - The Apache HTTP Server
   Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2023-05-02 02:38:59 UTC; 27s ago
     Docs: https://httpd.apache.org/docs/2.4/
  Process: 414 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
 Main PID: 459 (apache2)
    Tasks: 55 (limit: 4663)
   Memory: 13.6M
   CGroup: /system.slice/apache2.service
           ├─459 /usr/sbin/apache2 -k start
           ├─460 /usr/sbin/apache2 -k start
           └─461 /usr/sbin/apache2 -k start

May 02 02:38:58 webserver systemd[1]: Starting The Apache HTTP Server...
May 02 02:38:59 webserver systemd[1]: Started The Apache HTTP Server.
student-01-a0075c49c713@webserver:~$ 
```

### Prepare the disk to create a custom image

Verify that the boot disk will not be deleted when the instance is deleted.

1. On the VM instances page, click **webserver** to view the VM instance details.
2. Under **Storage** > **Boot disk**, verify that **When deleting instance** is set to **Keep disk**.
3. Return to the VM instances page, select **webserver**, and then click **More actions** (![More actions icon](https://cdn.qwiklabs.com/3TiyfyCT%2F0FPozvdrtFUVIIx4mnoMgoPTov6GlwTwzs%3D)) .
4. Click **Delete.**
5. In the confirmation dialog, click **Delete**.
6. In the left pane, click **Disks** and verify that the **webserver** disk exists.



### Create the custom image

1. In the left pane, click **Images**.
2. Click **Create image**.
3. Specify the following, and leave the remaining settings as their defaults:

```sh
gcloud compute images create mywebserver --project=qwiklabs-gcp-04-1059d35c6e27 --source-disk=webserver --source-disk-zone=us-central1-a --storage-location=us
```



## Task 4. Configure an instance template and create instance groups

A managed instance group uses an instance template to create a group  of identical instances. Use these to create the backends of the HTTP  load balancer.

### Configure the instance template

An instance template is an API resource that you can use to create VM instances and managed instance groups. Instance templates define the  machine type, boot disk image, subnet, labels, and other instance  properties.

1. In the Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Compute Engine** > **Instance templates**.

```sh
gcloud compute instance-templates create mywebserver-template --project=qwiklabs-gcp-04-1059d35c6e27 --machine-type=f1-micro --network-interface=network=default,no-address --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=425659184878-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=allow-health-checks --create-disk=auto-delete=yes,boot=yes,device-name=mywebserver-template,image=projects/qwiklabs-gcp-04-1059d35c6e27/global/images/mywebserver,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
```

### **Create the health check for managed instance groups**

1. On the **Navigation menu**, click **Compute Engine** > **Health checks**.
2. Click **Create health check**.
3. Specify the following, and leave the remaining settings as their defaults:

```sh
gcloud beta compute health-checks create tcp http-health-check --project=qwiklabs-gcp-04-1059d35c6e27 --port=80 --proxy-header=NONE --no-enable-logging --check-interval=5 --timeout=5 --unhealthy-threshold=2 --healthy-threshold=2
```

### Create the managed instance groups

Create a managed instance group in **us-central1** and one in **europe-west1**.

1. On the **Navigation menu**, click **Compute Engine** > **Instance groups**.
2. Click **Create Instance Group**.



### Verify the backends

Verify that VM instances are being created in both regions.

- On the **Navigation menu**, click **Compute Engine** > **VM instances**.
   Notice the instances that start with *us-central1-mig* and *europe-west1-mig*. These instances are part of the managed instance groups.

## Task 5. Configure the HTTP load balancer

Configure the HTTP load balancer to balance traffic between the two backends (**us-central1-mig** in us-central1 and **europe-west1-mig** in europe-west1) as illustrated in the network diagram:



### Start the configuration

1. On the **Navigation menu**, click **Network Services** > **Load balancing**.
2. Click **Create Load Balancer**.
3. Under **HTTP(S) Load Balancing**, click **Start configuration**.
4. Under **Internet facing or internal only**, select **From Internet to my VMs or serverless services**.
5. Under **Global or Regional**, select **Global HTTP(S) Load Balancer (classic)**.
6. Click **Continue**.
7. For **Name**, type **http-lb**.





## Task 6. Stress test the HTTP load balancer

Now that you have created the HTTP load balancer for your backends,  it is time to verify that traffic is forwarded to the backend service.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-1059d35c6e27.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-04-1059d35c6e27)$ LB_IP=34.111.70.232
while [ -z "$RESULT" ] ;
do
  echo "Waiting for Load Balancer";
  sleep 5;
  RESULT=$(curl -m1 -s $LB_IP | grep Apache);
done
Waiting for Load Balancer
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-04-1059d35c6e27)$
```

### Stress test the HTTP load balancer

Create a new VM to simulate a load on the HTTP load balancer. Then  determine whether traffic is balanced across both backends when the load is high.

1. In the Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Compute Engine** > **VM instances**.
2. Click **Create instance**.

```sh
Linux stress-test 4.19.0-23-cloud-amd64 #1 SMP Debian 4.19.269-1 (2022-12-20) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May  2 02:39:23 2023 from 35.235.241.16
student-01-a0075c49c713@stress-test:~$ export LB_IP=http://34.111.70.232/
student-01-a0075c49c713@stress-test:~$ export LB_IP=34.111.70.232
student-01-a0075c49c713@stress-test:~$ echo $LB_IP
34.111.70.232
student-01-a0075c49c713@stress-test:~$ ab -n 500000 -c 1000 http://$LB_IP/
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 34.111.70.232 (be patient)
Completed 50000 requests
Completed 100000 requests



```



## Task 7. Review

In this lab, you configured an HTTP load balancer with backends in  us-central1 and europe-west1. Then you stress-tested the load balancer  with a VM to demonstrate global load balancing and autoscaling.
