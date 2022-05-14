---
layout: single
title:  "Azure Compute Services"
date:   2022-05-14 02:59:04 +0530
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
# Azure Compute Services
In this post, let us explore core Azure Compute services like
-  Azure Virtual Machines
-  Azure App Service
-  Azure Container Instances 
-  Azure Kubernetes Service
-  Azure Functions
-  Azure Virtual Desktop

Azure compute is an on-demand computing service for running cloud-based applications. It provides computing resources such as disks, processors, memory, networking, and operating systems. 

Some of the most prominent services are:
Azure Virtual Machines
Azure Container Instances
Azure App Service
Azure Functions (or serverless computing)

## Azure Virtual Machines
Virtual machines are software emulations of physical computers. They include a virtual processor, memory, storage, and networking resources. VMs host an operating system, and you can install and run software just like a physical computer. When using a remote desktop client, you can use and control the VM as if you were sitting in front of it.

With Azure Virtual Machines, you can create and use VMs in the cloud. Virtual Machines provides infrastructure as a service (IaaS) and can be used in different ways. 

## Azure Virtual Machine Scalesets
Virtual machine scale sets are an Azure compute resource that you can use to deploy and manage a set of identical VMs (replicas in Kubernetes!). With all VMs configured the same, virtual machine scale sets are designed to support true autoscale. No pre-provisioning of VMs is required.

## Containers and Kubernetes
Container Instances and Azure Kubernetes Service are Azure compute resources that you can use to deploy and manage containers. Containers are lightweight, virtualized application environments. They're designed to be quickly created, scaled out, and stopped dynamically. You can run multiple instances of a containerized application on a single host machine.

## Azure App Service
With Azure App Service, you can quickly build, deploy, and scale enterprise-grade web, mobile, and API apps running on any platform. 
App Service is a platform as a service (PaaS) offering.

## Azure Functions (Serverless)
Functions are ideal when you're concerned only about the code running your service and not the underlying platform or infrastructure. They're commonly used when you need to perform work in response to an event (often via a REST request), timer, or message from another Azure service, and when that work can be completed quickly, within seconds or less.

Here is a screenshot of available compute services from the Azure Portal.
![Azure Compute]({{ site.url }}{{ site.baseurl }}/assets/images/azure-compute-services.png)

One nice thing that I noticed in this poratl is that, just next to each resource, there is a link to appropriate learning material.

Also, seen is a listing of various resources from the Marketplace.
