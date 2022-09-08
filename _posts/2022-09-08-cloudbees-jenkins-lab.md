---
layout: single
title:  "CloudBees Jenkins Training Lab Setup"
date:   2022-09-08 010:58:04 +0530
categories: Automation
tags: Jenkins
show_date: true
classes: wide
header:
  teaser: /assets/images/jenkins.svg
author:
  name     : "Jenkins"
  avatar   : "/assets/images/jenkins.svg"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Jenkins Essentials Lab Instance Setup

```sh
(base) pradeep:~$pwd
/Users/pradeep
(base) pradeep:~$
```
```sh
(base) pradeep:~$cd Downloads/cloudbees-training-admin-fundamentals
(base) pradeep:~$ls
Vagrantfile					cb-training-admin-fundamentals-1596650931.box
(base) pradeep:~$
```
```sh
(base) pradeep:~$cat Vagrantfile 

Vagrant.configure("2") do |config|
  config.vm.box = "cb-training-admin-fundamentals-1596650931"
  config.vm.box_url = "cb-training-admin-fundamentals-1596650931.box"
end
(base) pradeep:~$
```
```sh
(base) pradeep:~$vagrant up
==> vagrant: A new version of Vagrant is available: 2.3.0 (installed version: 2.2.19)!
==> vagrant: To upgrade visit: https://www.vagrantup.com/downloads.html

Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'cb-training-admin-fundamentals-1596650931' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Box file was not detected as metadata. Adding it directly...
==> default: Adding box 'cb-training-admin-fundamentals-1596650931' (v0) for provider: virtualbox
    default: Unpacking necessary files from: file:///Users/pradeep/Downloads/cloudbees-training-admin-fundamentals/cb-training-admin-fundamentals-1596650931.box
==> default: Successfully added box 'cb-training-admin-fundamentals-1596650931' (v0) for 'virtualbox'!
==> default: Importing base box 'cb-training-admin-fundamentals-1596650931'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: cloudbees-training-admin-fundamentals_default_1662627947779_90103
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 5000 (guest) => 5000 (host) (adapter 1)
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: butler
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default: 
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
(base) pradeep:~$
```

```sh

(base) pradeep:~$vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching micromachine-3.0.0.gem
Fetching vagrant-vbguest-0.30.0.gem
Installed the plugin 'vagrant-vbguest (0.30.0)'!
(base) pradeep:~$
```

```sh
(base) pradeep:~$vagrant up                            
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
(base) pradeep:~$
```
```sh
(base) pradeep:~$vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
(base) pradeep:~$
```

![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/cb-1.png)
![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/cb-2.png)
![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/cb-3.png)
![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/cb-4.png)
![Jenkins]({{ site.url }}{{ site.baseurl }}/assets/images/cb-5.png)
