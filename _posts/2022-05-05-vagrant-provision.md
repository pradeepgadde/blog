---
layout: single
title:  "Vagrant Provision"
date:   2022-05-05 04:59:04 +0530
categories: Automation
tags: Vagrant
show_date: true
classes: wide
header:
  teaser: /assets/images/vagrant.png
author:
  name     : "Vagrant"
  avatar   : "/assets/images/vagrant.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Vagrant Provision
So far we have a virtual machine running a basic copy of Ubuntu. We can edit files locally and have them synced into the virtual machine. In this post we  will serve those files using a webserver.

First let us create a simple HTML file

```html
pradeep:~$pwd
/Users/pradeep/learn-vagrant/html
pradeep:~$ls
index.html
pradeep:~$cat index.html 
<!DOCTYPE html>
<html>
  <body>
    <h1>Getting started with Vagrant!</h1>
  </body>
</html>


pradeep:~$
```
Create the following shell script and save it as `start-script.sh` in the same directory as your `Vagrantfile`.

```sh
pradeep:~$cat start-script.sh 
#!/usr/bin/env bash

apt-get update
apt-get install -y apache2
if ! [ -L /var/www ]; then
  rm -rf /var/www
  ln -fs /vagrant /var/www
fi


pradeep:~$
```

Next, configure Vagrant to run this shell script when setting up the machine, by editing the Vagrantfile. It which should now look like this.

```sh
pradeep:~$cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.provision :shell, path: "start-script.sh"
end


pradeep:~$
```

This line `config.vm.provision :shell, path: "start-script.sh"` does the provisioning of our VM.
The file path is relative to the location of the project root (where the Vagrantfile is).

```sh
pradeep:~$vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'hashicorp/bionic64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'hashicorp/bionic64' version '1.0.282' is up to date...
==> default: Setting the name of the VM: html_default_1651728777632_64363
==> default: Fixed port collision for 22 => 2222. Now on port 2200.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2200 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2200
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 6.0.10
    default: VirtualBox Version: 6.1
==> default: Mounting shared folders...
    default: /vagrant => /Users/pradeep/learn-vagrant/html
==> default: Running provisioner: shell...
    default: Running: /var/folders/cf/vzmh318x285f0c1sbsnm14m40000gn/T/vagrant-shell20220505-1626-1dnckyj.sh
    default: Hit:1 http://archive.ubuntu.com/ubuntu bionic InRelease
    default: Get:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
    default: Get:3 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
    default: Get:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
    default: Get:5 http://security.ubuntu.com/ubuntu bionic-security/main i386 Packages [1,162 kB]
    default: Get:6 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2,545 kB]
    default: Get:7 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [2,202 kB]
    default: Get:8 http://archive.ubuntu.com/ubuntu bionic-updates/main i386 Packages [1,464 kB]
    default: Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/main Translation-en [476 kB]
    default: Get:10 http://archive.ubuntu.com/ubuntu bionic-updates/restricted i386 Packages [32.4 kB]
    default: Get:11 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [734 kB]
    default: Get:12 http://archive.ubuntu.com/ubuntu bionic-updates/restricted Translation-en [101 kB]
    default: Get:13 http://archive.ubuntu.com/ubuntu bionic-updates/universe i386 Packages [1,606 kB]
    default: Get:14 http://security.ubuntu.com/ubuntu bionic-security/main Translation-en [386 kB]
    default: Get:15 http://security.ubuntu.com/ubuntu bionic-security/restricted i386 Packages [25.7 kB]
    default: Get:16 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [709 kB]
    default: Get:17 http://security.ubuntu.com/ubuntu bionic-security/restricted Translation-en [97.1 kB]
    default: Get:18 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1,193 kB]
    default: Get:19 http://security.ubuntu.com/ubuntu bionic-security/universe i386 Packages [1,016 kB]
    default: Get:20 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1,806 kB]
    default: Get:21 http://security.ubuntu.com/ubuntu bionic-security/universe Translation-en [274 kB]
    default: Get:22 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [17.6 kB]
    default: Get:23 http://security.ubuntu.com/ubuntu bionic-security/multiverse i386 Packages [6,012 B]
    default: Get:24 http://security.ubuntu.com/ubuntu bionic-security/multiverse Translation-en [3,660 B]
    default: Get:25 http://archive.ubuntu.com/ubuntu bionic-updates/universe Translation-en [391 kB]
    default: Get:26 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [24.8 kB]
    default: Get:27 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse i386 Packages [11.2 kB]
    default: Get:28 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse Translation-en [6,012 B]
    default: Get:29 http://archive.ubuntu.com/ubuntu bionic-backports/main i386 Packages [10.8 kB]
    default: Get:30 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [10.8 kB]
    default: Get:31 http://archive.ubuntu.com/ubuntu bionic-backports/main Translation-en [5,016 B]
    default: Get:32 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [11.6 kB]
    default: Get:33 http://archive.ubuntu.com/ubuntu bionic-backports/universe i386 Packages [11.6 kB]
    default: Get:34 http://archive.ubuntu.com/ubuntu bionic-backports/universe Translation-en [5,864 B]
    default: Fetched 16.6 MB in 6s (2,743 kB/s)
    default: Reading package lists...
    default: Reading package lists...
    default: Building dependency tree...
    default: Reading state information...
    default: The following additional packages will be installed:
    default:   apache2-bin apache2-data apache2-utils libapr1 libaprutil1
    default:   libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0 ssl-cert
    default: Suggested packages:
    default:   www-browser apache2-doc apache2-suexec-pristine | apache2-suexec-custom
    default:   openssl-blacklist
    default: The following NEW packages will be installed:
    default:   apache2 apache2-bin apache2-data apache2-utils libapr1 libaprutil1
    default:   libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.2-0 ssl-cert
    default: 0 upgraded, 10 newly installed, 0 to remove and 259 not upgraded.
    default: Need to get 1,730 kB of archives.
    default: After this operation, 6,997 kB of additional disk space will be used.
    default: Get:1 http://archive.ubuntu.com/ubuntu bionic/main amd64 libapr1 amd64 1.6.3-2 [90.9 kB]
    default: Get:2 http://archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1 amd64 1.6.1-2 [84.4 kB]
    default: Get:3 http://archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.1-2 [10.6 kB]
    default: Get:4 http://archive.ubuntu.com/ubuntu bionic/main amd64 libaprutil1-ldap amd64 1.6.1-2 [8,764 B]
    default: Get:5 http://archive.ubuntu.com/ubuntu bionic/main amd64 liblua5.2-0 amd64 5.2.4-1.1build1 [108 kB]
    default: Get:6 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-bin amd64 2.4.29-1ubuntu4.22 [1,071 kB]
    default: Get:7 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-utils amd64 2.4.29-1ubuntu4.22 [84.0 kB]
    default: Get:8 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2-data all 2.4.29-1ubuntu4.22 [160 kB]
    default: Get:9 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 apache2 amd64 2.4.29-1ubuntu4.22 [95.1 kB]
    default: Get:10 http://archive.ubuntu.com/ubuntu bionic/main amd64 ssl-cert all 1.0.39 [17.0 kB]
    default: dpkg-preconfigure: unable to re-open stdin: No such file or directory
    default: Fetched 1,730 kB in 2s (1,047 kB/s)
    default: Selecting previously unselected package libapr1:amd64.
(Reading database ... 42512 files and directories currently installed.)
    default: Preparing to unpack .../0-libapr1_1.6.3-2_amd64.deb ...
    default: Unpacking libapr1:amd64 (1.6.3-2) ...
    default: Selecting previously unselected package libaprutil1:amd64.
    default: Preparing to unpack .../1-libaprutil1_1.6.1-2_amd64.deb ...
    default: Unpacking libaprutil1:amd64 (1.6.1-2) ...
    default: Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.
    default: Preparing to unpack .../2-libaprutil1-dbd-sqlite3_1.6.1-2_amd64.deb ...
    default: Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.1-2) ...
    default: Selecting previously unselected package libaprutil1-ldap:amd64.
    default: Preparing to unpack .../3-libaprutil1-ldap_1.6.1-2_amd64.deb ...
    default: Unpacking libaprutil1-ldap:amd64 (1.6.1-2) ...
    default: Selecting previously unselected package liblua5.2-0:amd64.
    default: Preparing to unpack .../4-liblua5.2-0_5.2.4-1.1build1_amd64.deb ...
    default: Unpacking liblua5.2-0:amd64 (5.2.4-1.1build1) ...
    default: Selecting previously unselected package apache2-bin.
    default: Preparing to unpack .../5-apache2-bin_2.4.29-1ubuntu4.22_amd64.deb ...
    default: Unpacking apache2-bin (2.4.29-1ubuntu4.22) ...
    default: Selecting previously unselected package apache2-utils.
    default: Preparing to unpack .../6-apache2-utils_2.4.29-1ubuntu4.22_amd64.deb ...
    default: Unpacking apache2-utils (2.4.29-1ubuntu4.22) ...
    default: Selecting previously unselected package apache2-data.
    default: Preparing to unpack .../7-apache2-data_2.4.29-1ubuntu4.22_all.deb ...
    default: Unpacking apache2-data (2.4.29-1ubuntu4.22) ...
    default: Selecting previously unselected package apache2.
    default: Preparing to unpack .../8-apache2_2.4.29-1ubuntu4.22_amd64.deb ...
    default: Unpacking apache2 (2.4.29-1ubuntu4.22) ...
    default: Selecting previously unselected package ssl-cert.
    default: Preparing to unpack .../9-ssl-cert_1.0.39_all.deb ...
    default: Unpacking ssl-cert (1.0.39) ...
    default: Setting up libapr1:amd64 (1.6.3-2) ...
    default: Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...
    default: Processing triggers for ureadahead (0.100.0-21) ...
    default: Setting up apache2-data (2.4.29-1ubuntu4.22) ...
    default: Setting up ssl-cert (1.0.39) ...
    default: Processing triggers for libc-bin (2.27-3ubuntu1) ...
    default: Setting up libaprutil1:amd64 (1.6.1-2) ...
    default: Processing triggers for systemd (237-3ubuntu10.25) ...
    default: Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
    default: Setting up liblua5.2-0:amd64 (5.2.4-1.1build1) ...
    default: Setting up libaprutil1-ldap:amd64 (1.6.1-2) ...
    default: Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.1-2) ...
    default: Setting up apache2-utils (2.4.29-1ubuntu4.22) ...
    default: Setting up apache2-bin (2.4.29-1ubuntu4.22) ...
    default: Setting up apache2 (2.4.29-1ubuntu4.22) ...
    default: Enabling module mpm_event.
    default: Enabling module authz_core.
    default: Enabling module authz_host.
    default: Enabling module authn_core.
    default: Enabling module auth_basic.
    default: Enabling module access_compat.
    default: Enabling module authn_file.
    default: Enabling module authz_user.
    default: Enabling module alias.
    default: Enabling module dir.
    default: Enabling module autoindex.
    default: Enabling module env.
    default: Enabling module mime.
    default: Enabling module negotiation.
    default: Enabling module setenvif.
    default: Enabling module filter.
    default: Enabling module deflate.
    default: Enabling module status.
    default: Enabling module reqtimeout.
    default: Enabling conf charset.
    default: Enabling conf localized-error-pages.
    default: Enabling conf other-vhosts-access-log.
    default: Enabling conf security.
    default: Enabling conf serve-cgi-bin.
    default: Enabling site 000-default.
    default: Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /lib/systemd/system/apache2.service.
    default: Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /lib/systemd/system/apache-htcacheclean.service.
    default: Processing triggers for libc-bin (2.27-3ubuntu1) ...
    default: Processing triggers for systemd (237-3ubuntu10.25) ...
    default: Processing triggers for ureadahead (0.100.0-21) ...
    default: Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...
pradeep:~$
```

In the above output, from line starting from the below line, rest all is part of our provisioning.
```sh
==> default: Running provisioner: shell...
    default: Running: 
```

It is possible to change the synced_folder path, for example

```sh
pradeep:~$cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.synced_folder "html", "/var/www"
  config.vm.provision :shell, path: "start-script.sh"
end

```
The result is that, along with the default mounting, we do see an extra mapping of `html` folder to `/var/www` inside the VM.

```sh
==> default: Mounting shared folders...
    default: /var/www => /Users/pradeep/learn-vagrant/html
    default: /vagrant => /Users/pradeep/learn-vagrant
```

