---

layout: single
title:  "Using Prometheus for Monitoring on Google Cloud: Qwik Start"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"
sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Using Prometheus for Monitoring on Google Cloud: Qwik Start

Set up a Google Kubernetes Engine cluster, then deploy the Managed  Service for Prometheus to ingest metrics from a simple application.

[Managed Service for Prometheus](https://cloud.google.com/stackdriver/docs/managed-prometheus) is Google Cloud's fully managed storage and query service for  Prometheus metrics. This service is built on top of Monarch, the same  globally scalable data store as Cloud Monitoring.

A thin fork of Prometheus replaces existing Prometheus deployments  and sends data to the managed service with no user intervention. This  data can then be queried by using PromQL through the Prometheus Query  API supported by the managed service and by using the existing Cloud  Monitoring query mechanisms.

- Deploy the Managed Service for Prometheus to a GKE cluster
- Deploy a Python application to monitor
- Create a Cloud Monitoring dashboard to view metrics collected

## Setup a Google Kubernetes Engine cluster

1. Run the following command to deploy a standard GKE cluster, which will prompt you to authorize and enable the GKE API:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-b442466199d8.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ gcloud beta container clusters create gmp-cluster --num-nodes=1 --zone us-central1-c --enable-managed-prometheus
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster gmp-cluster in us-central1-c... Cluster is being health-checked (master is healthy)...done.                                                                       
Created [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-02-b442466199d8/zones/us-central1-c/clusters/gmp-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-c/gmp-cluster?project=qwiklabs-gcp-02-b442466199d8
kubeconfig entry generated for gmp-cluster.
NAME: gmp-cluster
LOCATION: us-central1-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.172.96.144
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 1
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ gcloud container clusters get-credentials gmp-cluster --zone us-central1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gmp-cluster.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```



## Deploy the Prometheus service

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl create ns gmp-test
namespace/gmp-test created
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl get ns
NAME                 STATUS   AGE
default              Active   2m40s
gke-managed-system   Active   119s
gmp-public           Active   107s
gmp-system           Active   107s
gmp-test             Active   5s
kube-node-lease      Active   2m41s
kube-public          Active   2m41s
kube-system          Active   2m41s
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```



## Deploy the application

1. Deploy a simple application which emits metrics at the `/metrics` endpoint:

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_deployment.yaml
deployment.apps/helloworld-gke created
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_service.yaml
service/hello created
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl get pods -n gmp-test
NAME                              READY   STATUS    RESTARTS   AGE
helloworld-gke-5df949bf69-6mgbp   1/1     Running   0          69s
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl get svc -n gmp-test
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
hello   LoadBalancer   10.106.133.141   35.232.222.96   80:31658/TCP   48s
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ curl $url/metrics
curl: (3) URL using bad/illegal format or missing URL
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ url=$(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ curl $url/metrics


curl: (28) Failed to connect to 35.232.222.96 port 80 after 130101 ms: Connection timed out
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ curl $url/metrics
# HELP flask_exporter_info Multiprocess metric
# TYPE flask_exporter_info gauge
flask_exporter_info{version="0.18.5"} 1.0
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/prom_deploy.yaml
podmonitoring.monitoring.googleapis.com/prom-example created
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/prom_deploy.yaml
podmonitoring.monitoring.googleapis.com/prom-example created
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl get pods -n gmp-test
NAME                              READY   STATUS    RESTARTS   AGE
helloworld-gke-5df949bf69-6mgbp   1/1     Running   0          4m57s
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ kubectl get svc -n gmp-test
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
hello   LoadBalancer   10.106.133.141   35.232.222.96   80:31658/TCP   4m36s
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ timeout 120 bash -c -- 'while true; do curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'); sleep $((RANDOM % 4)) ; done'
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:03 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:07 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:12 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:13 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:17 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:19 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:22 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:26 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:29 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:31 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:34 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:37 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:42 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:45 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:47 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:51 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:54 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:02:59 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:04 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:09 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:13 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:16 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:21 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:22 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:25 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:29 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:33 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:38 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:40 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:42 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:47 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:49 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:51 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:55 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:57 GMT"}
{"message":"Hello World!","severity":"info","timestamp":"Tue, 14 May 2024 03:03:59 GMT"}
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```



## Observing the app via metrics

In this last section, quickly use `gcloud` to deploy a custom monitoring dashboard that shows the metrics from this application in a line chart.

```sh
tudent_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ gcloud monitoring dashboards create --config='''
{
  "category": "CUSTOM",
  "displayName": "Prometheus Dashboard Example",
  "mosaicLayout": {
    "columns": 12,
    "tiles": [
      {
        "height": 4,
        "widget": {
          "title": "prometheus/flask_http_request_total/counter [MEAN]",
          "xyChart": {
            "chartOptions": {
              "mode": "COLOR"
''' ] } "yPos": 0,,e": "LINEAR", "0s",r": "ALIGN_MEAN"AN",ogleapis.com/flask_http_request_total/counter\" resource.type=\"prometheus_target\"",
Created [159cade0-4570-472f-8d48-c53ca09334d7].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ history 
    1  gcloud beta container clusters create gmp-cluster --num-nodes=1 --zone us-central1-c --enable-managed-prometheus
    2  gcloud container clusters get-credentials gmp-cluster --zone us-central1-c
    3  kubectl create ns gmp-test
    4  kubectl get ns
    5  kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_deployment.yaml
    6  kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/flask_service.yaml
    7  url=$(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')
    8  curl $url/metrics
    9  curl $url/metrics
   10  kubectl get pods -n gmp-test
   11  kubectl get svc -n gmp-test
   12  curl $url/metrics
   13  url=$(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')
   14  curl $url/metrics
   15  curl $url/metrics
   16  kubectl -n gmp-test apply -f https://raw.githubusercontent.com/kyleabenson/flask_telemetry/master/gmp_prom_setup/prom_deploy.yaml
   17  kubectl get pods -n gmp-test
   18  kubectl get svc -n gmp-test
   19  timeout 120 bash -c -- 'while true; do curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'); sleep $((RANDOM % 4)) ; done'
   20  gcloud monitoring dashboards create --config='''
{
  "category": "CUSTOM",
  "displayName": "Prometheus Dashboard Example",
  "mosaicLayout": {
    "columns": 12,
    "tiles": [
      {
        "height": 4,
        "widget": {
          "title": "prometheus/flask_http_request_total/counter [MEAN]",
          "xyChart": {
            "chartOptions": {
              "mode": "COLOR"
            },
            "dataSets": [
              {
                "minAlignmentPeriod": "60s",
                "plotType": "LINE",
                "targetAxis": "Y1",
                "timeSeriesQuery": {
                  "apiSource": "DEFAULT_CLOUD",
                  "timeSeriesFilter": {
                    "aggregation": {
                      "alignmentPeriod": "60s",
                      "crossSeriesReducer": "REDUCE_NONE",
                      "perSeriesAligner": "ALIGN_RATE"
                    },
                    "filter": "metric.type=\"prometheus.googleapis.com/flask_http_request_total/counter\" resource.type=\"prometheus_target\"",
                    "secondaryAggregation": {
                      "alignmentPeriod": "60s",
                      "crossSeriesReducer": "REDUCE_MEAN",
                      "groupByFields": [
                        "metric.label.\"status\""
                      ],
                      "perSeriesAligner": "ALIGN_MEAN"
                    }
                  }
                }
              }
            ],
            "thresholds": [],
            "timeshiftDuration": "0s",
            "yAxis": {
              "label": "y1Axis",
              "scale": "LINEAR"
            }
          }
        },
        "width": 6,
        "xPos": 0,
        "yPos": 0
      }
    ]
  }
}
'''
   21  history 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-b442466199d8)$ 
```

