---

layout: single
title:  "Caching Content with Cloud CDN"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Caching Content with Cloud CDN

Configure Cloud Content Delivery Network (Cloud CDN) for a backend bucket and verify caching of an image. Cloud CDN uses Google's globally  distributed edge points of presence to cache HTTP(S) load-balanced  content close to your users. Caching content at the edges of Google's  network provides faster delivery of content to your users while reducing serving costs

- Create and populate a Cloud Storage bucket
- Create an HTTP load balancer with Cloud CDN
- Verify the caching of your bucket's content

## Create and populate a Cloud Storage bucket

Cloud CDN content can originate from two types of backends:

- Google Compute Engine virtual machine (VM) instance groups
- Google Cloud Storage buckets

In this lab, you configure a Cloud Storage bucket as the backend.



### **Create a unique Cloud Storage bucket**

1. In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage** > **Buckets**.
2. Click **Create bucket**.
3. For the name, enter a globally unique value, and click **Continue**.
4. For the **Location Type**, select **Region**.
5. For the **Location**, choose a location that is either  halfway around the world from you or at least on a different continent.  (This provides a greater difference between accessing the image with and without Cloud CDN enabled.)
6. Click **Continue**, and then click **Choose how to control access to objects**.
7. Clear **Enforce public access prevention on this bucket** and select **Fine-grained** then click **Create**

Even if the ***\*Enforce public access prevention on this bucket\****checkbox is cleared, the project's organization policy can still deny public access to the bucket contents.

### **Copy an image file into your bucket**

Copy an image from a public Cloud Storage bucket to your own bucket.

https://storage.googleapis.com/pradeepgadde/cdn.png

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-441a64197b17.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ gsutil cp gs://cloud-training/gcpnet/cdn/cdn.png gs://pradeepgadde
Copying gs://cloud-training/gcpnet/cdn/cdn.png [Content-Type=image/png]...
- [1 files][300.5 KiB/300.5 KiB]                                                
Operation completed over 1 objects/300.5 KiB.                                    
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$  gsutil acl ch -u AllUsers:R gs://pradeepgadde/cdn.png
Updated ACL on gs://pradeepgadde/cdn.png
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ 
```



## Create the HTTP load balancer with Cloud CDN

HTTP(S) load balancing provides global load balancing for HTTP(S)  requests of static content to a Cloud Storage bucket (backend). When you enable Cloud CDN on your backend, your content is cached at a [location at the edge of Google's network](https://cloud.google.com/cdn/docs/locations), which is usually far closer to the user than your backend is.

### **Start the HTTP load balancer Configuration**

1. In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Network Services** > **Load balancing**.
2. Click **Create load balancer**.
3. Under **HTTP(S) Load Balancing**, click **Start configuration**.
4. For **Name**, type **cdn-lb**. Click Continue

### **Configure the backend**

1. Click  **Backend configuration**.
2. For **Backend services & backend buckets**, click **Create a backend bucket**.
3. For **Name**, type **cdn-bucket**.
4. Click **Browse** under **Cloud Storage bucket**.
5. Select your bucket, and click **Select**.
6. Select **Enable Cloud CDN**.
7. Click **Create**.

Yes, enabling Cloud CDN is as simple as selecting **Enable Cloud CDN**!

### **Configure the frontend**

The host and path rules determine how your traffic will be directed.  For example, you could direct video traffic to one backend and image  traffic to another backend. However, you are not configuring the Host  and path rules in this lab.

1. Click **Frontend configuration**.
2. Specify the following, leaving all other values with their defaults:

| Property   | Value (type value or select option as specified) |
| ---------- | ------------------------------------------------ |
| Protocol   | HTTP                                             |
| IP version | IPv4                                             |
| IP address | Ephemeral                                        |
| Port       | 80                                               |

Click **Done**.

### **Review and create the HTTP load balancer**

1. Click **Review and finalize**.
2. Review the **Backend Buckets** and **Frontend**.
3. Click **Create**.
    Wait for the load balancer to be created.
4. Click on the name of the load balancer (**cdn-lb**).
5. Note the IP address of the load balancer for the next task. It will be referred to as `[LB_IP_ADDRESS]`.

## Verify the caching of your bucket's content

Now that you have created the HTTP load balancer for your bucket and  enabled Cloud CDN, it is time to verify that the image is cached on the  edge of Google's network.

### **Time the HTTP request for the image**

One way to verify that the image is cached is to time the HTTP  request for the image. The first request should take significantly  longer, because content is only cached at an edge location after being  accessed through that location.

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ export LB_IP_ADDRESS=34.120.207.59
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ for i in {1..3};do curl -s -w "%{time_total}\n" -o /dev/null http://$LB_IP_ADDRESS/cdn.png; done
1.361731
0.837809
0.842696
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ 
tudent_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ for i in {1..3};do curl -s -w "%{time_total}\n" -o /dev/null http://$LB_IP_ADDRESS/cdn.png; done
0.007413
1.102943
0.005885
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ for i in {1..3};do curl -s -w "%{time_total}\n" -o /dev/null http://$LB_IP_ADDRESS/cdn.png; done
0.008453
0.006577
0.830567
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ for i in {1..3};do curl -s -w "%{time_total}\n" -o /dev/null http://$LB_IP_ADDRESS/cdn.png; done
0.006932
0.004497
1.333396
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ 
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-04-441a64197b17)$ 
```

In this example output, the second and third request take less than 1%  of the time of the first request. This demonstrates that the image was  cached during the first request and accessed from an edge location on  further requests. Depending on how far you placed your storage bucket  and where your closest edge location is, you will see different results.

### **Explore the Cloud CDN logs**

Another way to verify that the image got cached in the previous step, is to explore the Cloud CDN logs. These logs will contain information  on when content was cached and when the cache was accessed.

In the Cloud Console, on the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Logging** > **Logs Viewer**.

Under the **Resources** filter, select **Cloud HTTP Load Balancer > cdn-lb-forwarding-rule > cdn-lb**.

Expand the first log entry (on top).

Within the entry, expand the **httpRequest** and notice that the **cacheLookup** is *true* but there is no **cacheHit** field.

This illustrates that the cache did not contain the image on this first request.

Expand the **jsonPayload** and notice that the **statusDetails** field contains *response_sent_by_backend*.

This also illustrates that the image came from the backend bucket on this first request.

```json
{
  "insertId": "l3lj1yfize37e",
  "jsonPayload": {
    "backendTargetProjectNumber": "projects/1041497079288",
    "cacheId": "SIN-7c9ff679",
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "remoteIp": "34.87.131.93",
    "statusDetails": "response_sent_by_backend",
    "cacheDecision": [
      "RESPONSE_HAS_CACHE_CONTROL",
      "RESPONSE_CACHE_CONTROL_PUBLIC",
      "RESPONSE_HAS_ETAG",
      "RESPONSE_HAS_LAST_MODIFIED",
      "RESPONSE_HAS_EXPIRES",
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_CACHE_ALL_STATIC"
    ]
  },
  "httpRequest": {
    "requestMethod": "GET",
    "requestUrl": "http://34.120.207.59/cdn.png",
    "requestSize": "84",
    "status": 200,
    "responseSize": "308313",
    "userAgent": "curl/7.81.0",
    "remoteIp": "34.87.131.93",
    "cacheLookup": true,
    "cacheFillBytes": "308313",
    "serverIp": "2002:a05:6a10:7598::",
    "latency": "1.332074s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "target_proxy_name": "cdn-lb-target-proxy",
      "forwarding_rule_name": "cdn-lb-forwarding-rule",
      "project_id": "qwiklabs-gcp-04-441a64197b17",
      "zone": "global",
      "backend_service_name": "",
      "url_map_name": "cdn-lb"
    }
  },
  "timestamp": "2024-05-01T17:27:08.073884Z",
  "severity": "INFO",
  "logName": "projects/qwiklabs-gcp-04-441a64197b17/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-441a64197b17/traces/e3cb37a2826a3aa04e1e0fc74d0691ff",
  "receiveTimestamp": "2024-05-01T17:27:11.021058278Z",
  "spanId": "9302fd7b7cce986f"
}
```

Close the current log entry and expand a different log entry.

Within the entry, expand the **httpRequest** and notice that the **cacheHit** is *true*.

This illustrates that the cache contained the image on this request.

Expand the **jsonPayload** and notice that the **statusDetails** field contains *response_from_cache*.

This also illustrates that the cache, instead of the backend, provided the image on this request.

```json
{
  "insertId": "jzp0p2filex0m",
  "jsonPayload": {
    "@type": "type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry",
    "cacheId": "SIN-63b91979",
    "backendTargetProjectNumber": "projects/1041497079288",
    "remoteIp": "34.87.131.93",
    "statusDetails": "response_from_cache",
    "cacheDecision": [
      "RESPONSE_HAS_CACHE_CONTROL",
      "RESPONSE_CACHE_CONTROL_PUBLIC",
      "RESPONSE_HAS_ETAG",
      "RESPONSE_HAS_LAST_MODIFIED",
      "RESPONSE_HAS_EXPIRES",
      "RESPONSE_HAS_CONTENT_TYPE",
      "CACHE_MODE_CACHE_ALL_STATIC"
    ]
  },
  "httpRequest": {
    "requestMethod": "GET",
    "requestUrl": "http://34.120.207.59/cdn.png",
    "requestSize": "84",
    "status": 200,
    "responseSize": "308322",
    "userAgent": "curl/7.81.0",
    "remoteIp": "34.87.131.93",
    "cacheHit": true,
    "cacheLookup": true,
    "latency": "0.002182s"
  },
  "resource": {
    "type": "http_load_balancer",
    "labels": {
      "forwarding_rule_name": "cdn-lb-forwarding-rule",
      "project_id": "qwiklabs-gcp-04-441a64197b17",
      "target_proxy_name": "cdn-lb-target-proxy",
      "url_map_name": "cdn-lb",
      "backend_service_name": "",
      "zone": "global"
    }
  },
  "timestamp": "2024-05-01T17:27:08.059694Z",
  "severity": "INFO",
  "logName": "projects/qwiklabs-gcp-04-441a64197b17/logs/requests",
  "trace": "projects/qwiklabs-gcp-04-441a64197b17/traces/d90b3ece3b8b6ab19a63bc323165e7a2",
  "receiveTimestamp": "2024-05-01T17:27:09.124426067Z",
  "spanId": "59a92a2159c0a102"
}
```

The Cloud CDN logs clearly demonstrate that the image was provided from  the backend on the first request. This request filled the cache on the  edge location, and all future requests received the image from that  cache.

n this lab, you configured Cloud CDN for a backend bucket by configuring an HTTP load balancer and enabling Cloud CDN with a simple checkbox.  You verified the caching of the bucket's content by accessing an image  multiple times and exploring the Cloud CDN logs. The first time you  accessed the image, it took longer because the cache of the edge  location did not contain the image yet. All other requests were quicker  because the image was provided from the cache of the edge location  closest to your Cloud Shell instance.