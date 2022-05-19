---

layout: single
title:  "Azure Functions"
date:   2022-05-19 02:59:04 +0530
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

If your application logic is event driven, In other words, for a large amount of time, your application is waiting for a particular input before it performs any processing, then to reduce your costs, you want to avoid having to pay for the time that your application is waiting for input. This is where Azure Functions help.

Serverless computing is the abstraction of servers, infrastructure, and operating systems. With serverless computing, Azure takes care of managing the server infrastructure and the allocation and deallocation of resources based on demand. 

Serverless computing includes:
- the abstraction of servers;
- an event-driven scale; and
- micro-billing.

Azure has two implementations of serverless compute:

- **Azure Functions**: Functions can execute code in almost any modern language.
- **Azure Logic Apps**: Logic apps are designed in a web-based designer and can execute logic triggered by Azure services without writing any code.

Functions are commonly used when you need to perform work in response to an event (often via a REST request), timer, or message from another Azure service, and when that work can be completed quickly, within seconds or less.

Functions can be either stateless or stateful.

Logic apps are similar to functions. Both enable you to trigger logic based on an event. Where functions execute code, logic apps execute *workflows* that are designed to automate business scenarios and are built from predefined logic blocks.

You create logic app workflows by using a visual designer on the Azure portal or in Visual Studio. The workflows are persisted as a JSON file with a known workflow schema.

With Functions, you write code to complete each step.
With Logic Apps, you use a GUI to define the actions and how they relate to one another.

Here is a screenshot of creating Azure Functions
![Azure Functions]({{ site.url }}{{ site.baseurl }}/assets/images/azure-functions.png)
