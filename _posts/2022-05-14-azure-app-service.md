---
layout: single
title:  "Azure App Service"
date:   2022-05-14 10:59:04 +0530
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
# Azure App Service
App Service enables you to build and host web apps, background jobs, mobile back-ends, and RESTful APIs in the programming language of your choice without managing infrastructure.

It offers automatic scaling and high availability. 

App Service supports Windows and Linux and enables automated deployments from GitHub, Azure DevOps, or any Git repo to support a continuous deployment model.

This platform as a service (PaaS) environment allows you to focus on the website and API logic while Azure handles the infrastructure to run and scale your web applications.

## Web apps
App Service includes full support for hosting web apps by using ASP.NET, ASP.NET Core, Java, Ruby, Node.js, PHP, or Python. You can choose either Windows or Linux as the host operating system.

## API apps
Much like hosting a website, you can build REST-based web APIs by using your choice of language and framework. You get full Swagger support and the ability to package and publish your API in Azure Marketplace. The produced apps can be consumed from any HTTP- or HTTPS-based client.

## WebJobs
You can use the WebJobs feature to run a program (.exe, Java, PHP, Python, or Node.js) or script (.cmd, .bat, PowerShell, or Bash) in the same context as a web app, API app, or mobile app. They can be scheduled or run by a trigger. WebJobs are often used to run background tasks as part of your application logic.

## Mobile apps
Use the Mobile Apps feature of App Service to quickly build a back end for iOS and Android apps. 

With just a few clicks in the Azure portal, you can:
Store mobile app data in a cloud-based SQL database.
Authenticate customers against common social providers, such as MSA, Google, Twitter, and Facebook.
Send push notifications.
Execute custom back-end logic in C# or Node.js.

On the mobile app side, there's SDK support for native iOS and Android, Xamarin, and React native apps.

Here is a screenshot of creating a Web App.

![Azure WebApp]({{ site.url }}{{ site.baseurl }}/assets/images/azure-webapp.png)