---

layout: single
title:  "Azure Disk Storage"
date:   2022-05-21 07:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
Disk Storage provides disks for Azure virtual machines. Applications and other services can access and use these disks as needed, similar to how they would in on-premises scenarios. Disk Storage allows data to be persistently stored and accessed from an attached virtual hard disk.

There are several disk types:
Traditional HDDs (Hard Disk Drives)
SSDs (Solid-state Drives)
Premium SSDs
Ultra Disks


Azure virtual machine can use separate disks to store different data like OS Disk and Data Disk.

Azure VMs have one operating system disk and a temporary disk for short-term storage. You can attach additional data disks. The size of the VM determines the type of storage you can use and the number of data disks allowed.

Earlier we have seen how to create compute resources like virtual machines, but we havent focussed on the Storage part then.

Here is revisiting that with focus on Disks.

![Azure Disks]({{ site.url }}{{ site.baseurl }}/assets/images/azure-disks-1.png)


![Azure Disks]({{ site.url }}{{ site.baseurl }}/assets/images/azure-disks-2.png)