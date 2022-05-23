---

layout: single
title:  "Azure Blob Storage"
date:   2022-05-22 06:59:04 +0530
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

Azure Blob storage is an object storage solution.
It is unstructured, so any kind of data can be stored.
It can be reached from anywhere in the Internet.

It is not limited to common file formats.
It does not require developers to think about or manage disks;

Blob Storage is ideal for:
Serving images or documents directly to a browser.
Storing files for distributed access.
Streaming video and audio.
Storing data for backup and restore, disaster recovery, and archiving.
Storing data for analysis by an on-premises or Azure-hosted service.
Storing up to 8 TB of data for virtual machines.

You store blobs in containers, which helps you organize your blobs depending on your business needs.

![Azure Blob]({{ site.url }}{{ site.baseurl }}/assets/images/azure-blob-storage.png)

Data stored in the cloud can be different based on how it's generated, processed, and accessed over its lifetime.

Azure Storage offers different access tiers for your blob storage, helping you store object data in the most cost-effective manner. The available access tiers include:

Hot access tier: Optimized for storing data that is accessed frequently (for example, images for your website).

Cool access tier: Optimized for data that is infrequently accessed and stored for at least 30 days (for example, invoices for your customers).

Archive access tier: Appropriate for data that is rarely accessed and stored for at least 180 days, with flexible latency requirements (for example, long-term backups).

![Azure Blob]({{ site.url }}{{ site.baseurl }}/assets/images/azure-blob-access-tiers.png)
