---

layout: single
title:  "Creating and Alerting on Logs-based Metrics"
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
# Creating and Alerting on Logs-based Metrics

[Log-based metrics](https://cloud.google.com/logging/docs/logs-based-metrics) are [Cloud Monitoring](https://cloud.google.com/monitoring/docs/) metrics that are based on the content of log entries. These metrics can help you identify trends, extract numeric values out of the logs, and  set up an alert when a certain log entry occurs by creating a metric for that event. You can use both system and user-defined log-based metrics  in Cloud Monitoring to create charts and alerting policies.google

The log-based metrics interface is divided into two metric-type panes: System metrics and User-defined metrics.

**[System-defined log-based metrics](https://cloud.google.com/logging/docs/logs-based-metrics#system_log-based_metrics)** are provided by Cloud Logging for use by all Google Cloud projects.They calculated only from logs that have been ingested by Logging. If a log  has been explicitly [excluded](https://cloud.google.com/logging/docs/routing/overview#exclusions) from ingestion, it isn't included in these metrics.

**[User-defined log-based metrics](https://cloud.google.com/logging/docs/logs-based-metrics#user-metrics)** are created by you to track things in your Google Cloud project. For  example, you might create a log-based metric to count the number of log  entries that match a given filter.

Creating an alert from a metric lets you create an alerting policy based on the log-based metric.

- Create a log-based alert
- Create a system-defined log-based metric
- Create a user-defined log-based metric
- Create an alert for the user-defined log-based metric

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-c1d509bcd624.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ gcloud config set compute/zone us-east1-c
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ export PROJECT_ID=$(gcloud info --format='value(config.project)')
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ gcloud container clusters create gmp-cluster --num-nodes=1 --zone us-east1-c
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster gmp-cluster in us-east1-c... Cluster is being health-checked (master is healthy)...done.                                                                          
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-c1d509bcd624/zones/us-east1-c/clusters/gmp-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-c/gmp-cluster?project=qwiklabs-gcp-03-c1d509bcd624
kubeconfig entry generated for gmp-cluster.
NAME: gmp-cluster
LOCATION: us-east1-c
MASTER_VERSION: 1.29.4-gke.1043002
MASTER_IP: 34.73.253.183
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.29.4-gke.1043002
NUM_NODES: 1
STATUS: RUNNING
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ 
```

## Log-based alert

Log-based alerts notify you whenever a specific message appears in  your logs. Try it out by setting up a log-based alert to tell you when a VM stops running.

1. From Cloud Console, in the Search bar, type in “logs explorer”, then click on the **Logs Explorer** result.
2. Click the **Show Query** slide bar.
3. Enter the following parameters to create Log Based Alert:

```sh
resource.type="gce_instance" protoPayload.methodName="v1.compute.instances.stop"
```

1. Click **Create alert** link.
2. Add the following parameters, click **Next** to move to the next parameter.

- **Alert name:** stopped vm
- **Choose logs to include in the alert:** will auto-fill with the query you entered
- **Set notification frequency and autoclose duration:** Time between notifications is `5 min` and Incident autoclose duration is `1 hr`. Click **Next**.

**Who should be notified (optional):**

- Click on the dropdown arrow next to **Notification Channels**, then click on **Manage Notification Channels**.
- A Notification channels page will open in the new tab.
- Scroll down the page and click on **ADD NEW** for **Email**.
- Enter your personal email in the **Email Address** field and a **Display name**.
- Click **Save**.
- When done, return to the Logs Explorer tab you were in previously.
- Refresh the Notification Channels, then select the channel you just created. Click **OK**.

1. Click **Save**.

You will now cause your VM to stop.

1. Go to the 2nd Cloud Console tab, and navigate to **Navigation menu** > **Compute Engine** > **VM instances**.
2. Check the box next to **instance1**, then click **Stop** at the top of the page, then click **Stop** again in the pop-up window. The green check mark will turn to a gray circle when the instance has been stopped.
3. In the Search bar, type "monitoring", then choose the **Monitoring** option.
4. Click on the **Alerting** tab. You'll see that your alert has registered. Under Alert Policies click the **See all policies** link and you'll see the log-based alert you created listed.

## Log-based metric

Using log-based metrics you can define a metric that tracks errors in the logs to proactively respond to similar problems and symptoms before they are noticed by end users.

1. At the beginning of the lab you deployed a standard GKE cluster. Run the following command to ensure that the cluster named `gmp-cluster` has been created:

```sh
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ gcloud container clusters list
NAME: gmp-cluster
LOCATION: us-east1-c
MASTER_VERSION: 1.29.4-gke.1043002
MASTER_IP: 34.73.253.183
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.29.4-gke.1043002
NUM_NODES: 1
STATUS: RUNNING
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ gcloud container clusters get-credentials gmp-cluster
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gmp-cluster.
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl create ns gmp-test
namespace/gmp-test created
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl -n gmp-test apply -f https://storage.googleapis.com/spls/gsp091/gmp_flask_deployment.yaml
deployment.apps/helloworld-gke created
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl -n gmp-test apply -f https://storage.googleapis.com/spls/gsp091/gmp_flask_service.yaml
service/hello created
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ 
```

```sh
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl get services -n gmp-test
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello   LoadBalancer   34.118.234.35   <pending>     80:30132/TCP   20s
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl get services -n gmp-test
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello   LoadBalancer   34.118.234.35   <pending>     80:30132/TCP   26s
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ kubectl get services -n gmp-test
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
hello   LoadBalancer   34.118.234.35   34.148.54.121   80:30132/TCP   41s
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$
```

```sh
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')/metrics
# HELP flask_exporter_info Multiprocess metric
# TYPE flask_exporter_info gauge
flask_exporter_info{version="0.18.5"} 1.0
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ 
```

## Create a log-based metric

1. Return to **Logs Explorer**.
2. Click **Create metric** link.
3. On the Create metric page, input the following:

- **Metric type:** leave the default setting, Counter
- **Log based metric name:** hello-app-error
- **Filter selection:** update the following into the Build filter:

```sh
severity=ERROR
resource.labels.container_name="hello-app"
textPayload: "ERROR: 404 Error page not found"
```

Click **Create metric**.



## Create a metrics-based alert

1. In the left pane of **Logging** window select **Log-based Metrics**. Then in user-defined metrics click on **3 vertical dots** next to metrics and select **Create alert from metric**.
2. Under **Select a Metric**, the metric parameters will automatically fill in.

- Update the Rolling window to **2 min**.
- Accept the other default settings
- Click **Next**.

1. You will need to set Notifications. Feel free to re-use the channel you created earlier in the lab.
2. Name the alert policy `log based metric alert`.

## Generate some errors

Next you'll generate some errors to match the log-based metric you created and trigger the metric-based alert.

1. In Cloud Shell, run the following to generate some errors:

1. Return to the **Logs Explorer** page, and go to the Severity section on the lower left side. Click on the **Error** severity. Now you can search for the `404 Error page not found` error. View more information by expanding one of the 404 Error messages.
2. Return to the **Monitoring** page, and click on **Alerting**. You will see the 2 policies you created.
3. Click on the **Alert policies** link, and you should see both alerts in the Incidents section. Click on an incident to see details.

```sh
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ timeout 120 bash -c -- 'while true; do curl $(kubectl get services -n gmp-test -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}')/error; sleep $((RANDOM % 4)) ; done'
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:26:54 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:26:56 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:01 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:06 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:11 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:14 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:18 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:23 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:27 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:29 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:34 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:39 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:41 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:46 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:51 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:53 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:55 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:27:57 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:01 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:04 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:09 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:14 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:18 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:23 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:28 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:32 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:34 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:37 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:39 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:42 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:44 GMT"}
{"message":"404 Error page not found","request_time":"Wed, 26 Jun 2024 17:28:49 GMT"}
student_03_d5d699a9681e@cloudshell:~ (qwiklabs-gcp-03-c1d509bcd624)$ 
```

```json
{
  "textPayload": "[2024-06-26 17:28:01,848] 10.32.0.1 requested http://34.148.54.121/error ERROR: 404 Error page not found http://34.148.54.121/error",
  "insertId": "lmg0tgmo6ji4i9j1",
  "resource": {
    "type": "k8s_container",
    "labels": {
      "cluster_name": "gmp-cluster",
      "location": "us-east1-c",
      "namespace_name": "gmp-test",
      "container_name": "hello-app",
      "pod_name": "helloworld-gke-5df949bf69-l5zjh",
      "project_id": "qwiklabs-gcp-03-c1d509bcd624"
    }
  },
  "timestamp": "2024-06-26T17:28:01.849340722Z",
  "severity": "ERROR",
  "labels": {
    "compute.googleapis.com/resource_name": "gke-gmp-cluster-default-pool-11cc53c6-m2wn",
    "k8s-pod/app": "hello",
    "k8s-pod/pod-template-hash": "5df949bf69"
  },
  "logName": "projects/qwiklabs-gcp-03-c1d509bcd624/logs/stderr",
  "receiveTimestamp": "2024-06-26T17:28:04.042778157Z"
}
```
