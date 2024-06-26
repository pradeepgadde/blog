---

layout: single
title:  "Monitoring Multiple Projects with Cloud Monitoring"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Monitoring Multiple Projects with Cloud Monitoring

Cloud Monitoring provides dashboards and alerts so you can review  performance metrics for cloud services, virtual machines, and common  open source servers such as MongoDB, Apache, Nginx, Elasticsearch, and  more. You configure Cloud Monitoring in the Console.

In this hands-on lab you will have 2 projects to monitor in Cloud  Monitoring. You'll add them both to a Cloud Monitoring account and  monitor the metrics the virtual machines in the projects provide.

- Create a Cloud Monitoring account that has two Google Cloud projects.
- Monitor across both projects from the single Cloud Monitoring account.

Project 1 already has a virtual machine (and you can look at it by going to **Compute Engine > VM instances**). You will create a virtual machine in Project 2, and then monitor both projects in Cloud Monitoring.

## Create Project 2's virtual machine

1. At the top of the screen, click on the dropdown arrow next to Project 1's name.
2. Make sure that you're on the **All** tab, then click on the name of **Project 2** to go into it. 
3. Select **Navigation menu > Compute Engine** to open the VM instances window.
4. Click **+Create instance** to create a new instance.
5. Name this instance **instance2**.
6. Select `Region`  and `Zone` .

Leave all of the options at the default settings.

1. Click **Create**.

Now you have resources to monitor in both of your projects.

### Create a Monitoring Metrics Scope

Set up a Monitoring Metrics Scope that's tied to your Google Cloud  Project. The following steps create a new account that has a free trial  of Monitoring.

- In the Cloud Console, click **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) > View All Products >  **Monitoring**.

When the Monitoring **Overview** page opens, your metrics scope project is ready.

Now add both projects to Monitoring.

1. In the left panel, click **Monitoring Settings** and then in the **Settings** window, click **+Add GCP PROJECTS** in the GCP Projects section.
2. Click **Select Projects**.
3. Check Project ID 1 and click **Select**.
4. Click **Add projects**.

## Monitoring Overview

Click on **Overview** in the left menu. You'll be adding a lot of good information here as the lab goes along. First, you'll create a [Cloud Monitoring Group](https://cloud.google.com/monitoring/groups/) for visibility across both projects.

### About Cloud Monitoring groups

Cloud Monitoring lets you define and monitor groups of resources,  such as VM instances, databases, and load balancers. Groups can be based on names, tags, regions, applications, and other criteria. You can also create subgroups, up to six levels deep, within groups.

### Create a Cloud Monitoring group

1. In the left menu, click **Groups**, then click **+Create group**.
2. Name your group **DemoGroup**.

The **Criteria** is a set of rules that will dynamically evaluate which resources should be part of this group.

Cloud Monitoring dynamically determines which resources belong to your group based on the filter criteria that you set up.

- In the first dropdown field (Type), **Name** is selected by default.
- In the second dropdown (Operator), **Contains** is selected by default.
- In the third field (Value), type in "instance" since both of the instance names in both of your projects start with the word `instance`.
- Click **Done**, then click **Create**.

## Uptime check for your group

Uptime checks let you quickly verify the health of any web page,  instance, or group of resources. Each configured check is regularly  contacted from a variety of locations around the world. Uptime checks  can be used as conditions in alerting policy definitions.

1. In the left menu, click **Uptime checks**, then click **+Create uptime check**.

2. Create your uptime check with the following information:

   **Protocol:** TCP

   **Resource Type:** Instance

   **Applies To:** Group, and then select **DemoGroup**.

   **Port:** 22

   **Check frequency:** 1 minute, then click **Continue**.

   Click **Continue** again.

   Leave the slider **ON** state for **Create an alert** option in **Alert & notification** section, then click **Continue**.

   For **Title:** enter `DemoGroup uptime check`.

   Click **TEST** to verify that your uptime check can connect to the resource.

   When you see a green check mark everything can connect, click **Create**

## Alerting policy for the group

Use Cloud Monitoring to create one or more alerting policies.

1. In the left menu, click **Uptime checks**.

2. Click the three dots ![More menu icon](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D) at the far right of your Display Name and click **Add alert policy**.

3. Click **+Add alert condition**.

4. Select the previously created **Uptime health check on DemoGroup** condition from the left section and click **Delete alert condition**.

5. In your **New condition**, click **Select a metric**.

   Uncheck the **Active**.

   In the **Select a metric** field, search `check_passed` and click **VM Instance > Uptime_check > Check passed**. Click **Apply**.

   Click **Add a filter**, set the `Filter` to **check_id** and select **demogroup-uptime-check-id** as the `Value`. Click **Done**.

   n left panel, click on the arrow button next to **VM Instance-Check passed**, then click on **Configure trigger**.

   Select **Metric absence** as Condition type and click **Next**.

   Turn off **Configure notifications**.

   In the **Alert policy name** field, enter the **Name** as **Uptime Check Policy**. Click **Next**.

   Click **Create policy**.

## Custom dashboard for your group

Create a custom dashboard so you can monitor your group easily.

1. In the left menu, click **Dashboards**, then click **+Create dashboard**.
2. Name your dashboard.
3. Click **+Add Widget** and select **Line** option in **Visualization**.
4. In the **Metric** field, Uncheck the **Active**.
5. Search **uptime** (compute.googleapis.com/instance/uptime) and click **VM Instance > Instance > Uptime**. Click **Apply**.

## Remove one instance to cause a problem

1. In the console, select **Navigation menu > Compute Engine**.
2. Check the box next to **instance2**, then click on the 3 vertical dots ![More icon](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D) at the top of the page and click **Stop**. Click **Stop** again to turn off the machine.
3. Wait a minute or 2 for the instance to stop and violate the uptime  check you just set up. After a couple of minutes, turn your machine back on by clicking **Start/Resume**, then **Start**.
4. Click **Navigation menu > Monitoring > Alerting**  and refresh your browser. It may take a few more minutes to show that  you have issues in the Summary section. Refresh until you see an  Incident similar to this:

### Incidents

When the alerting policy conditions are violated, an "incident" is created and displayed in the Incident section.

Responders can acknowledge receipt of the notification and can close the incident when it has been taken care of.

1. In the **Incidents** section, click on the name of the alerting policy that was violated to go into it.

You've already **fixed** your problem by turning the VM back on, so the incident was cleared and you no longer see an incident in the Incidents section.

1. To see the cleared incident, scroll down and click on the **Show closed incidents** link.

2. Your incident should have a **Closed** status. You can read through the incident details.

   1. You can also click on the **Uptime Check Policy** link to explore the metrics it gives you.

   In several more minutes the Monitoring Overview page will all go back to green when the instance in Project 2 passes the Uptime Check.

## Remove your alerting policy

If you set up an email alert as part of your alerting policy, there  is a chance that you will receive a few emails about your resources even after the lab is completed.

To avoid this, remove the alerting policy before you complete your lab.
