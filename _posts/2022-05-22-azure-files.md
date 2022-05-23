---

layout: single
title:  "Azure Files"
date:   2022-05-22 07:59:04 +0530
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

Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block and Network File System (preview) protocols.

Azure file shares can be mounted concurrently by cloud or on-premises deployments of Windows, Linux, and macOS. 

We can use Azure Files for the following situations:

Azure Files makes it easier to migrate the on-prem applications that share data to Azure. If you mount the Azure file share to the same drive letter that the on-premises application uses, the part of your application that accesses the file share should work with minimal changes, if any.

Tools and utilities used by multiple developers in a group can be stored on a file share, ensuring that everybody can find them, and that they use the same version.

You might want to write diagnostic logs, metrics, and crash dumps to a file share, and process or analyze the data later.

Azure Files ensures the data is encrypted at rest, and the SMB protocol ensures the data is encrypted in transit.

There is a difference between  Azure Files and files on a corporate file share. You can access the Azure Files  from anywhere in the world, by using a URL that points to the file.

You can also use Shared Access Signature (SAS) tokens to allow access to a private asset for a specific amount of time.

![Azure File Share]({{ site.url }}{{ site.baseurl }}/assets/images/azure-file-share.png)



