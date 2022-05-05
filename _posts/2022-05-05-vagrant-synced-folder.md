---
layout: single
title:  "Vagrant Synced Folder"
date:   2022-05-05 04:58:04 +0530
categories: Automation
tags: Vagrant
show_date: true
toc: true
toc_sticky: true
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
# Vagrant Synced Folder
Vagrant automatically syncs files to and from the guest machine. This way you can edit files locally and run them in your virtual development environment.

By default, Vagrant shares your project directory (the one containing the `Vagrantfile`) to the `/vagrant` directory in your guest machine.

Create and configure a guest machine, as specified by your Vagrantfile.

```sh
pradeep:~$pwd
/Users/pradeep/learn-vagrant
pradeep:~$ls
Vagrantfile
```

```sh
pradeep:~$vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'hashicorp/bionic64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'hashicorp/bionic64' version '1.0.282' is up to date...
==> default: Setting the name of the VM: learn-vagrant_default_1651726584701_35824
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
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
    default: /vagrant => /Users/pradeep/learn-vagrant
pradeep:~$
```

We can see from the last line that Vagrant is mounting the `/Users/pradeep/learn-vagrant` to `/vagrant`.

```sh
==> default: Mounting shared folders...
    default: /vagrant => /Users/pradeep/learn-vagrant
```

```sh
pradeep:~$vagrant ssh
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu May  5 04:57:58 UTC 2022

  System load:  0.17              Processes:           92
  Usage of /:   2.5% of 61.80GB   Users logged in:     0
  Memory usage: 11%               IP address for eth0: 10.0.2.15
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 packages can be updated.
0 updates are security updates.


vagrant@vagrant:~$ 
```
Let us check the current working directory and its contents. Also verify the contents of the `/vagrant` folder.

```sh
vagrant@vagrant:~$ pwd
/home/vagrant
vagrant@vagrant:~$ ls 
vagrant@vagrant:~$ ls /vagrant
Vagrantfile
vagrant@vagrant:~$ 
```
We can see the `Vagrantfile` from our host inside the VM.

To see the files sync between the guest machine and yours add a new folder in your virtual machine's vagrant directory.

```sh
vagrant@vagrant:~$ mkdir /vagrant/pradeep
vagrant@vagrant:~$ touch /vagrant/pradeep/vagrant-demo-synced-folder.txt
vagrant@vagrant:~$ vi /vagrant/pradeep/vagrant-demo-synced-folder.txt
vagrant@vagrant:~$ cat /vagrant/pradeep/vagrant-demo-synced-folder.txt
This is a demo of Vagrant Synced Folder!

vagrant@vagrant:~$ exit
logout
Connection to 127.0.0.1 closed.
pradeep:~$
```
List the contents of your local vagrant directory, and notice that the new directory you created on your virtual machine is reflected there.

```sh
pradeep:~$ls
Vagrantfile	pradeep
pradeep:~$ls -la
total 8
drwxr-xr-x   5 pradeep  staff   160 May  5 10:30 .
drwxr-xr-x+ 82 pradeep  staff  2624 May  5 10:00 ..
drwxr-xr-x   5 pradeep  staff   160 May  5 09:52 .vagrant
-rw-r--r--   1 pradeep  staff  3024 May  5 09:53 Vagrantfile
drwxr-xr-x   3 pradeep  staff    96 May  5 10:31 pradeep
pradeep:~$ls pradeep 
vagrant-demo-synced-folder.txt
pradeep:~$cat pradeep/vagrant-demo-synced-folder.txt 
This is a demo of Vagrant Synced Folder!

pradeep:~$
```

The folder `pradeep` is now on our host machine; Vagrant kept the folders in sync.