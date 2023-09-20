---
layout: single
title:  "Testing of Webhooks in Juniper Mist"
date:   2023-09-20 08:58:04 +0530
categories: Automation
tags: Mist
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/mist.png
author:
  name     : "Mist"
  avatar   : "/assets/images/mist.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Mist Webhooks

## What’s a Webhook?

Reference: https://simonfredsted.com/1583

“The Webhook is the Web's way to integrate completely different systems in semi-real time.”

There are many public and free services like `Webhook.site` are available to test Webhooks.

You need to generate an unique URL and use it to `POST` the updates to this URL.

![]({{ site.url }}{{ site.baseurl }}/assets/images/Webhook_Mist_5.png)



In this post, let’s enable Webhooks in Juniper Mist Cloud and test!

First we need to Add a Webhook under `Organization` settings.

![]({{ site.url }}{{ site.baseurl }}/assets/images/Webhook_Mist_1.png)

Use your unique URL, in this case we used  `https://webhook.site/82842d5e-82ea-4af9-9815-7383c418d7a3`.

Select all available topics.

![]({{ site.url }}{{ site.baseurl }}/assets/images/Webhook_Mist_2.png) 

In `Advanced settings` , we have security settings and custom headers.

![]({{ site.url }}{{ site.baseurl }}/assets/images/Webhook_Mist_3.png) 

Once you save the Webhook configuration on Mist, it starts sending the HTTP POST messages to the URL configured.

The screenshot below shows that there are some 40 requests received so far from the  `54.193.71.17` Host, which is the AWS IP used by Mist Cloud. All the requests are posted to the same URL `https://webhook.site/#!/82842d5e-82ea-4af9-9815-7383c418d7a3/`. Each request has a unit ID. In the screenshot, the ID is `6b3f881c-98dc-45d9-8787-7667d5db8697`.

Here the complete URL.

`https://webhook.site/#!/82842d5e-82ea-4af9-9815-7383c418d7a3/6b3f881c-98dc-45d9-8787-7667d5db8697/1`

![]({{ site.url }}{{ site.baseurl }}/assets/images/Webhook_Mist_4.png)

Here is the raw content for some of the Topics/events.

## Add Webhook

```json
{
  "topic": "audits",
  "events": [
    {
      "admin_name": "Pradeep Gadde gaddepradeep@gmail.com",
      "id": "f0cd0bbc-d60f-4f5b-a8fe-e6627d9bcdb4",
      "message": "Add Webhook \"webhook_site\"",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "src_ip": "x.x.x.x",
      "timestamp": 1695169967.62159,
      "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/117.0",
      "webhook_id": "58c5f734-3d22-436e-8151-6a0725fa2645"
    }
  ]
}
```

## Device Events

```json
{
  "topic": "device-events",
  "events": [
    {
      "ap": "xx2316ed6axx",
      "ap_name": "Pradeep AP 01",
      "band": "6",
      "bandwidth": 80,
      "channel": 197,
      "device_name": "Pradeep AP 01",
      "device_type": "ap",
      "ev_type": "NOTICE",
      "mac": "xx2316ed6axx",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "power": 16,
      "pre_bandwidth": 80,
      "pre_channel": 197,
      "pre_power": 0,
      "pre_usage": 6,
      "reason": "auto-triggered-ACS",
      "site_id": "xxxxxx0a4-2770-47b6-93b1-87114bxxxx",
      "site_name": "Primary Site",
      "timestamp": 1695170180,
      "type": "AP_RRM_ACTION",
      "usage": 6
    },
    {
      "ap": "xx2316ed6axx",
      "ap_name": "Pradeep AP 01",
      "audit_id": "6175746f-0000-0000-3157-000000000000",
      "device_name": "Pradeep AP 01",
      "device_type": "ap",
      "ev_type": "NOTICE",
      "mac": "xx2316ed6axx",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "site_id": "xxxxxx0a4-2770-47b6-93b1-87114bxxxx",
      "site_name": "Primary Site",
      "timestamp": 1695170180,
      "type": "AP_CONFIG_CHANGED_BY_RRM"
    },
    {
      "ap": "xx2316ed6axx",
      "ap_name": "Pradeep AP 01",
      "device_name": "Pradeep AP 01",
      "device_type": "ap",
      "ev_type": "NOTICE",
      "mac": "xx2316ed6axx",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "site_id": "xxxxxx0a4-2770-47b6-93b1-87114bxxxx",
      "site_name": "Primary Site",
      "timestamp": 1695170181,
      "type": "AP_CONFIGURED"
    }
  ]
}
```

## Client Join

```json
{
  "topic": "client-join",
  "events": [
    {
      "ap": "xx2316ed6axx",
      "ap_name": "Pradeep AP 01",
      "band": "5",
      "bssid": "xxxx16f918xx",
      "connect": 1695172665,
      "connect_float": 1695172665.082,
      "mac": "xxdd8e7508xx",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "random_mac": false,
      "rssi": -27,
      "site_id": "xxxxxx0a4-2770-47b6-93b1-87114bxxxx",
      "site_name": "Primary Site",
      "ssid": "Pradeep_Guest",
      "timestamp": 1695172665,
      "version": 2,
      "wlan_id": "77ed81d2-4fab-4df7-807a-0b679900d8f7"
    }
  ]
}
```

## Client Sessions

```json
{
  "topic": "client-sessions",
  "events": [
    {
      "ap": "xx2316ed6axx",
      "ap_name": "Pradeep AP 01",
      "band": "5",
      "bssid": "xxxx16f918xx",
      "client_family": "",
      "client_manufacture": "Intel Corporate",
      "client_model": "",
      "client_os": "Windows 10",
      "connect": 1695172665,
      "connect_float": 1695172665.083,
      "disconnect": 1695189374,
      "disconnect_float": 1695189374.476,
      "duration": 16709.393960068,
      "mac": "xxdd8e7508xx",
      "next_ap": "000000000000",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "random_mac": false,
      "rssi": -23,
      "site_id": "xxxxxx0a4-2770-47b6-93b1-87114bxxxx",
      "site_name": "Primary Site",
      "ssid": "Pradeep_Guest",
      "termination_reason": 1,
      "timestamp": 1695189374,
      "version": 2,
      "wlan_id": "77ed81d2-4fab-4df7-807a-0b679900d8f7"
    }
  ]
}
```

## Audits

```json
{
  "topic": "audits",
  "events": [
    {
      "admin_name": "Pradeep Gadde gaddepradeep@gmail.com",
      "id": "198cf177-a73e-43c8-8dde-c490e0c033d8",
      "message": "Accessed Org \"gaddepradeep@gmail.com\"",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "src_ip": "x.x.x.x",
      "timestamp": 1695171740.805999,
      "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/117.0"
    }
  ]
}
```

## Mist Edge

```json
{
  "topic": "mxedge-events",
  "events": [
    {
      "audit_id": "e8025445-8f64-42ee-81a3-8dc276f0c2b9",
      "mxedge_id": "00000000-0000-0000-1000-020000335690",
      "mxedge_name": "HQ-Edge-1",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "timestamp": "1695194041.635342",
      "type": "ME_CONFIG_CHANGED_BY_USER"
    },
    {
      "audit_id": "e8025445-8f64-42ee-81a3-8dc276f0c2b9",
      "mxedge_id": "00000000-0000-0000-1000-020000a00997",
      "mxedge_name": "Site-Edge",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "timestamp": "1695194041.638763",
      "type": "ME_CONFIG_CHANGED_BY_USER"
    },
    {
      "audit_id": "e8025445-8f64-42ee-81a3-8dc276f0c2b9",
      "mxedge_id": "00000000-0000-0000-1000-020000c357d8",
      "mxedge_name": "HQ-Edge-2",
      "org_id": "xxxx0047-ccbb-46b8-a04e-d8240ddxxxxx",
      "timestamp": "1695194041.642188",
      "type": "ME_CONFIG_CHANGED_BY_USER"
    }
  ]
}

```

That’s it. A brief introduction and testing of Webhooks with Juniper Mist.

