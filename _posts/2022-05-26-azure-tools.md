---

layout: single
title:  "Tools for managing and configuring your Azure environment"
date:   2022-05-26 07:59:04 +0530
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

Microsoft Azure provides a collection of management tools through which we can deploy dozens or hundreds of resources at a time, configure individual services programmatically, view rich reports across usage, health, costs, and more.

## Azure Portal
The Azure portal is how most users first experience Azure, and that is what we have been using so far to create new services and view them.
## Azure Powershell
Azure PowerShell is a shell with which developers and DevOps and IT professionals can execute commands called cmdlets (pronounced command-lets). 

These commands call the Azure Rest API to perform every possible management task in Azure. 

Cmdlets can be executed independently or combined into a script file and executed together to orchestrate:
The routine setup, teardown, and maintenance of a single resource or multiple connected resources.
The deployment of an entire infrastructure, which might contain dozens or hundreds of resources, from imperative code.

Azure PowerShell is available for Windows, Linux, and Mac, and you can access it in a web browser via Azure Cloud Shell.

## The Azure CLI
In many respects, the Azure CLI is almost identical to Azure PowerShell in what you can do with it. Both run on Windows, Linux, and Mac, and can be accessed in a web browser via Cloud Shell. The primary difference is the syntax you use. 

## ARM Templates
By using Azure Resource Manager templates (ARM templates), you can describe the resources you want to use in a declarative JSON format. The benefit is that the entire ARM template is verified before any code is executed to ensure that the resources will be created and connected correctly. The template then orchestrates the creation of those resources in parallel. That is, if you need 50 instances of the same resource, all 50 instances are created at the same time.

![Azure ARM]({{ site.url }}{{ site.baseurl }}/assets/images/azure-arm-templates.svg)

## The Azure Mobile App
The Azure mobile app provides iOS and Android access to your Azure resources when you're away from your computer. 

With the Mobile App, you can monitor the health and status of your Azure resources, check for alerts, quickly diagnose and fix issues, and restart a web app or virtual machine (VM) and also run the Azure CLI or Azure PowerShell commands to manage your Azure resources.

![Azure App]({{ site.url }}{{ site.baseurl }}/assets/images/azure-app.jpeg)
