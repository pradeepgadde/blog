---

layout: single
title:  "GCP—Resource Monitoring"
date:   2022-10-08 03:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcpne.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcpne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Resource Monitoring
Use Cloud Monitoring to gain insight into applications that run on Google Cloud.

- Explore Cloud Monitoring

- Add charts to dashboards

- Create alerts with multiple conditions

- Create resource groups

- Create uptime checks

## Task 1. Create a Cloud Monitoring workspace
Verify resources to monitor
Three VM instances have been created for you that you will monitor.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-110.png)

Create a Monitoring workspace
You will now setup a Monitoring workspace that's tied to your Google Cloud Project. The following steps create a new account that has a free trial of Monitoring.

## Task 2. Custom dashboards
Create a dashboard

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-112.png)
Metrics Explorer
The Metrics Explorer allows you to examine resources and metrics without having to create a chart on a dashboard. Try to recreate the chart you just created using the Metrics Explorer.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-113.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-114.png)

## Task 3. Alerting policies
Create an alert and add the first condition
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-115.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-116.png)

## Task 4. Resource groups

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-117.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-118.png)

## Task 5. Uptime monitoring

## Task 6. Disable the alert
Disable the alert Alerting policies stay active for a while after a project is deleted, just in case it needs to be reinstalled. Since this is a lab, and you will not have access to this project again, remove the alerting policy you created.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-119.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-120.png)

## Task 7. Review
In this lab, we learned how to:

Monitor your projects
Create a Cloud Monitoring workspace
Create alerts with multiple conditions
Add charts to dashboards
Create resource groups
Create uptime checks for your services