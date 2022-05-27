---

layout: single
title:  "Exploring Microsoft Graph"
date:   2022-05-26 09:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/ms_build_cloud_skills.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

In this post, let us learn how Microsoft Graph facilitates the access and flow of data and how to form queries through REST and code.

Microsoft Graph is the gateway to data and intelligence in Microsoft 365. It provides a unified programmability model that you can use to access the tremendous amount of data in Microsoft 365, Windows 10, and Enterprise Mobility + Security.

Here is the official diagram representation of it, from Microsoft.
![MS Graph]({{ site.url }}{{ site.baseurl }}/assets/images/microsoft-graph.png)

The Microsoft Graph API offers a single endpoint, https://graph.microsoft.com. 

Microsoft Graph connectors work in the incoming direction, delivering data external to the Microsoft cloud into Microsoft Graph services and applications, to enhance Microsoft 365 experiences such as Microsoft Search. Connectors exist for many commonly used data sources such as Box, Google Drive, Jira, and Salesforce.

Microsoft Graph Data Connect provides a set of tools to streamline secure and scalable delivery of Microsoft Graph data to popular Azure data stores. The cached data serves as data sources for Azure development tools that you can use to build intelligent applications.


Microsoft Graph is a RESTful web API that enables you to access Microsoft Cloud service resources. After you register your app and get authentication tokens for a user or service, you can make requests to the Microsoft Graph API.

To read from or write to a resource such as a user or an email message, you construct a request that looks like the following:

```html
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```
Microsoft Graph currently supports two versions: v1.0 and beta.
our URL will include the resource you are interacting with in the request, such as me, user, group, drive, and site. 

For example, adding the following filter parameter restricts the messages returned to only those with the emailAddress property of jon@contoso.com.

```html
GET https://graph.microsoft.com/v1.0/me/messages?filter=emailAddress eq 'jon@contoso.com'
```

To access the data in Microsoft Graph, your application will need to acquire an OAuth 2.0 access token, and present it to Microsoft Graph in either of the following:
The HTTP Authorization request header, as a Bearer token
The graph client constructor, when using a Microsoft Graph client library

Microsoft is providing a Graph Explorer at [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer) 

![MS Graph]({{ site.url }}{{ site.baseurl }}/assets/images/ms-graph-explorer.png)

This is with our specific login
![MS Graph]({{ site.url }}{{ site.baseurl }}/assets/images/ms-graph-explorer-pradeep.png)