---

layout: single
title:  "Troubleshooting Workloads on GKE for Site Reliability Engineers"
date:   2023-03-30 14:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
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

# Troubleshooting Workloads on GKE for Site Reliability Engineers

Learn how to take advantage of the integrated capabilities of Google  Cloud's operations suite that includes logging, monitoring, and rich,  out-of-the-box dashboards.

- Navigate resource pages of Google Kubernetes Engine (GKE)
- Leverage the GKE dashboard to quickly view operational data
- Create logs-based metrics to capture specific issues
- Create a Service Level Objective (SLO)
- Define an Alert to notify SRE staff of incidents

## Scenario

Your organization has deployed a multi-tier microservices  application. It is a web-based e-commerce application called "Hipster  Shop", where users can browse for vintage items, add them to their cart  and purchase them. Hipster Shop is composed of many microservices,  written in different languages, that communicate via gRPC and REST APIs. The architecture of the deployment is optimized for learning purposes  and includes modern technologies as part of the stack: Kubernetes,  Istio, Cloud Operations, App Engine, gRPC, OpenTelemetry, and similar  cloud-native technologies.

As a member of the Site Reliability Engineering (SRE) team, you are  contacted when end users report issues viewing products and adding them  to their cart. You will explore the various services deployed to  determine the root cause of the issue and set up a Service Level  Objective (SLO) to prevent similar incidents from occurring in the  future.



```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-74d8fea90040.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_d230509fc53b@cloudshell:~ (qwiklabs-gcp-00-74d8fea90040)$ git clone --depth 1 --branch csb_1220 https://github.com/GoogleCloudPlatform/cloud-ops-sandbox.git
Cloning into 'cloud-ops-sandbox'...
remote: Enumerating objects: 642, done.
remote: Counting objects: 100% (642/642), done.
remote: Compressing objects: 100% (511/511), done.
remote: Total 642 (delta 141), reused 364 (delta 63), pack-reused 0
Receiving objects: 100% (642/642), 17.85 MiB | 17.66 MiB/s, done.
Resolving deltas: 100% (141/141), done.
student_00_d230509fc53b@cloudshell:~ (qwiklabs-gcp-00-74d8fea90040)$ cd cloud-ops-sandbox/sre-recipes/
student_00_d230509fc53b@cloudshell:~/cloud-ops-sandbox/sre-recipes (qwiklabs-gcp-00-74d8fea90040)$ gcloud container clusters get-credentials cloud-ops-sandbox --zone us-central1-b --project qwiklabs-gcp-00-74d8fea90040
Fetching cluster endpoint and auth data.
kubeconfig entry generated for cloud-ops-sandbox.
student_00_d230509fc53b@cloudshell:~/cloud-ops-sandbox/sre-recipes (qwiklabs-gcp-00-74d8fea90040)$ ./sandboxctl sre-recipes restore "recipe3"
Restoring service back to normal...
Done. Restored broken service to working state.
student_00_d230509fc53b@cloudshell:~/cloud-ops-sandbox/sre-recipes (qwiklabs-gcp-00-74d8fea90040)$
```





After creating a logs-based metric which closely describes the user  experience, the SRE team will use it to measure user happiness, these  metrics are our SLIs and will be used to define a **Service Level Objective (SLO)** on the `recommendationservice`. You use an SLO to specify service-level objectives for performance  metrics. An SLO is a measurable goal for performance over a period of  time.



```json
{
  "displayName": "99% - Distribution Cut - Calendar month",
  "goal": 0.99,
  "calendarPeriod": "MONTH",
  "serviceLevelIndicator": {
    "requestBased": {
      "distributionCut": {
        "distributionFilter": "metric.type=\"custom.googleapis.com/opencensus/grpc.io/client/roundtrip_latency\" resource.type=\"global\"",
        "range": {
          "min": -9007199254740991,
          "max": 100
        }
      }
    }
  }
}
```

The `Error budget` fraction represents the actual  percentage of error budget remaining for the compliance period. In the  SLO defined, there is a period of one calendar month and a performance  goal of 99% or better.

As denoted by the percentage, the error preventing product pages from loading properly in this fictitious scenario severely degraded the  service-level objective defined. This may not be the case in a real  world scenario as this lab ran a load test against the Kubernetes  cluster hosting the application workload.



To proactively notify the SRE team of any violations of the SLO set, it  is a best practice to define an alert that will trigger when the SLO is  violated. The alert can invoke a notification channel of your choice,  including: Email, SMS, PagerDuty, Slack, a WebHook or a subscription to a PubSub topic.



In this lab, you explored the Cloud Operations suite, which allows Site  Reliability Engineers (SRE) to investigate and diagnose issues  experienced with workloads deployed. In order to increase the  reliability of workloads, you explored how to navigate resource pages or GKE, view operational data from GKE dashboards, create logs-based  metrics to capture specific issues and proactively respond to incidents  by setting service level objectives and alerts to proactively notify the SRE team about issues experienced before they cause outages.

![gcp-301]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-301.png)

![gcp-302]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-302.png)

![gcp-303]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-303.png)

![gcp-304]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-304.png)

![gcp-305]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-305.png)

![gcp-306]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-306.png)

![gcp-307]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-307.png)

![gcp-308]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-308.png)

![gcp-309]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-309.png)

![gcp-310]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-310.png)

![gcp-311]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-311.png)

![gcp-312]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-312.png)

![gcp-313]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-313.png)

![gcp-314]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-314.png)

![gcp-315]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-315.png)

![gcp-316]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-316.png)

![gcp-317]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-317.png)

![gcp-318]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-318.png)

![gcp-319]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-319.png)

![gcp-320]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-320.png)

![gcp-321]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-321.png)

![gcp-322]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-322.png)

![gcp-323]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-323.png)

![gcp-324]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-324.png)

![gcp-325]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-325.png)
