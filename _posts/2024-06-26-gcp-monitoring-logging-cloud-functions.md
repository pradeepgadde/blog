---

layout: single
title:  "Monitoring and Logging for Cloud Functions"
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
# Monitoring and Logging for Cloud Functions

The Cloud Function details include execution times and counts, and memory usage.

- Create a Cloud Function
- Create logs-based metric for a Cloud Function
- Use Metrics Explorer to view data for your cloud function
- Create charts on the Monitoring Overview window

## Viewing Cloud Function logs & metrics in Cloud Monitoring

Before you collect logs and alerts, you need something to monitor. In this section, you create a Hello World cloud function to monitor.

1. In the Cloud console, select **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) > View All Products > **Cloud Functions**, and then **Create function**.
2. Set the following:

- **Function Name:** helloWorld
- **Region:** 
- **Trigger type:** HTTP
- **Authentication**: check the box next to **Allow unauthenticated invocations**

1. Click **Save**.
2. Expand **Runtime, build, connections and security settings**. Under *Autoscaling*, set the **Maximum number of instances** to **5**.
3. Click **Next**.
4. Click **Deploy**.

The cloud function automatically deploys and is listed on the Cloud  Function page. This takes a few minutes. When you see a green check mark next to the name, the cloud function is complete.

In Cloud Shell, run the following to get a tool called **vegeta** that will let you send some test traffic to your cloud function: Unpack the **vegeta** tool by running the following:

```sh
student_01_932d053b64d1@cloudshell:~$ curl -LO 'https://github.com/tsenart/vegeta/releases/download/v6.3.0/vegeta-v6.3.0-linux-386.tar.gz'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 2409k  100 2409k    0     0  1436k      0  0:00:01  0:00:01 --:--:-- 2237k
student_01_932d053b64d1@cloudshell:~$ tar xvzf vegeta-v6.3.0-linux-386.tar.gz
vegeta
student_01_932d053b64d1@cloudshell:~$ echo "GET https://us-central1-qwiklabs-gcp-03-c4f20893c97c.cloudfunctions.net/helloWorld" | ./vegeta attack -duration=300s > results.bin

student_01_932d053b64d1@cloudshell:~$ 
student_01_932d053b64d1@cloudshell:~$ 
```



1. Still in the Cloud Functions page, click the name of your function, and then click on the `Trigger` tab. Click the **Trigger URL** for your function.

If you see `Hello World!` in the new browser tab that opens, you're up and running!

Now send traffic to your cloud function. Run the following in Cloud Shell.

## Create logs-based metric

Now you'll create a Distribution type logs based metric using a  regular expression to extract the value of latency from the log entries `textPayload` field.

1. In the console, select **Navigation menu**  > View All Products > **Logging** > **Logs Explorer**. The Cloud Logging opens in the console.
2. To look at just the logs from your Cloud Function, in the **Resource** dropdown, select **Cloud Function** > **helloWorld** then click **Apply**. In the **Log name** dropdown, select **cloud-functions** checkbox then click **Apply**.
3. Click **Run query**.
4. Click **Create metric**.
5. In the Create log-based metric form:

- Change the Metric Type to **Distribution**.
- In Log-based metric name enter **CloudFunctionLatency-Logs**.
- Enter **textPayload** for Field name.
- Enter the following in the **Regular Expression** field:

```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-932d053b64d1@qwiklabs.net"
    },
    "requestMetadata": {
      "callerIp": "122.171.21.198",
      "callerSuppliedUserAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:127.0) Gecko/20100101 Firefox/127.0,gzip(gfe),gzip(gfe)",
      "requestAttributes": {
        "time": "2024-06-26T16:30:36.537972Z",
        "reason": "8uSywAYQGg5Db2xpc2V1bSBGbG93cw",
        "auth": {}
      },
      "destinationAttributes": {}
    },
    "serviceName": "cloudfunctions.googleapis.com",
    "methodName": "google.cloud.functions.v1.CloudFunctionsService.CreateFunction",
    "authorizationInfo": [
      {
        "resource": "projects/qwiklabs-gcp-03-c4f20893c97c/locations/us-central1/functions/helloWorld",
        "permission": "cloudfunctions.functions.create",
        "granted": true,
        "resourceAttributes": {},
        "permissionType": "ADMIN_WRITE"
      }
    ],
    "resourceName": "projects/qwiklabs-gcp-03-c4f20893c97c/locations/us-central1/functions/helloWorld",
    "request": {
      "location": "projects/qwiklabs-gcp-03-c4f20893c97c/locations/us-central1",
      "@type": "type.googleapis.com/google.cloud.functions.v1.CreateFunctionRequest",
      "function": {
        "dockerRegistry": "CONTAINER_REGISTRY",
        "ingressSettings": "ALLOW_ALL",
        "sourceUploadUrl": "https://storage.googleapis.com/uploads-669412913219.us-central1.cloudfunctions.appspot.com/3c026077-95e0-491b-8c95-8528e45e92c4.zip?GoogleAccessId=service-31616402511@gcf-admin-robot.iam.gserviceaccount.com&Expires=1719421231&Signature=RKweKzWj3%2B200yd4lvYPQt5amkUhUdOCfjlz2cq%2BUzLcz%2F8auzbK9a3Ye40gMxnvUT8cuIKBTun8pqL2lZuyP2%2FgvjdSCxHag1KLOqlurXnoyMtv7CiNdAHm4eyFuRyikvnP1x3XW9rBSWPPtkTbC6l4HGRUZ4XYRyVU1iQyVqJp2E1BX7OVTYgKfd1zpV9856NFb0SXZA4M7nOXbjm4qq%2B%2Bu58VjJt1jQGHNEFLrRRLkblRoiF6EOxdx%2B%2B9XdUD6oMBn706u%2FjL%2BkqzNIyxM5PMitWCyl8pDJkAaXxNx0OG4%2F3hIs2Yr4iiWg6hLgArDfAWQZkvfrjKjgoBCF4vSQ%3D%3D",
        "entryPoint": "helloWorld",
        "httpsTrigger": {
          "securityLevel": "SECURE_ALWAYS"
        },
        "labels": {
          "deployment-tool": "console-cloud"
        },
        "runtime": "nodejs20",
        "maxInstances": 5,
        "timeout": "60s",
        "name": "projects/qwiklabs-gcp-03-c4f20893c97c/locations/us-central1/functions/helloWorld",
        "availableMemoryMb": 256
      }
    },
    "resourceLocation": {
      "currentLocations": [
        "us-central1"
      ]
    }
  },
  "insertId": "uisktydbygm",
  "resource": {
    "type": "cloud_function",
    "labels": {
      "project_id": "qwiklabs-gcp-03-c4f20893c97c",
      "region": "us-central1",
      "function_name": "helloWorld"
    }
  },
  "timestamp": "2024-06-26T16:30:34.871665Z",
  "severity": "NOTICE",
  "logName": "projects/qwiklabs-gcp-03-c4f20893c97c/logs/cloudaudit.googleapis.com%2Factivity",
  "operation": {
    "id": "operations/cXdpa2xhYnMtZ2NwLTAzLWM0ZjIwODkzYzk3Yy91cy1jZW50cmFsMS9oZWxsb1dvcmxkL0dRVnhTOWZhUkJZ",
    "producer": "cloudfunctions.googleapis.com",
    "first": true
  },
  "receiveTimestamp": "2024-06-26T16:30:37.016781579Z"
}
```



## Metrics Explorer

Next, use Metrics Explorer to look at the data for your cloud function.

### Create a Monitoring Metrics Scope

Set up a Monitoring Metrics Scope that's tied to your Google Cloud  Project. The following steps create a new account that has a free trial  of Monitoring.

- In the Cloud Console, click **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) > View All Products >  **Monitoring**.

When the Monitoring **Overview** page opens, your metrics scope project is ready.

1. In the left menu, click **Metrics explorer**.
2. Under **Select a Metric > Metric** dropdown start typing `executions` and then select **Cloud Function > Function > Executions** from the suggested metrics and click **Apply**.
3. On the top right corner change the **widget type** to **Stacked bar chart** using the dropdown menu.
4. Explore other graph options, try a different metric. For example, click your current **Cloud function - Executions** metric to open the dropdown, select **Execution times**, and change the widget type to **Heatmap chart**.
5. Continue to explore and experiment. For example, go back to the **Executions** metric and change the Grouping function to the **95th percentile**. Select the widget type **Line chart**.

```
```

## Create charts on the Monitoring Overview window

Creating charts on the Monitoring Overview window is a great way to  track metrics that are important to you. In this section, you set up the same charts you created in the previous section, but now they'll be  saved into the Monitoring Overview window.

1. In the left menu, click **Dashboards**.
2. Click on **+ Create dashboard**.
3. Click on **+ Add widget**.
4. Under Visualization, select **Stacked bar**.
5. Under **Select a metric > Metric** dropdown select the default **VM instance > CPU > CPU utilization** metric to open the dropdown and change the metric. Click **Apply**.

**Note**: If VM Instance is not visible in the dropdown, uncheck `Active`

1. Start typing `executions` into the **Metric** dropdown, and then select **Cloud Function > Function > Executions** from the suggested metrics and click **Apply**.
2. Click **APPLY** in the upper right corner.
3. After you create the first chart, click **+ Add widget** > **Heatmap** to create the next one.
4. Under **Select a metric > Metric** dropdown select the default **VM INSTANCE > Vm_flow > RTT LATENCIES** metric to open the dropdown and change the metric. Click **Apply**.
5. Start typing `execution times` into the **Metric** dropdown, and then select **Cloud Function > Function > Execution times** from the suggested metrics and click **Apply**.
6. Click **APPLY** in the upper right corner.

By default, the charts name themselves after the metric you're using, but you can rename them.

For a quick reference, to see these charts click **Dashboards** in the left panel of the Monitoring page.



Two types of log-based metrics User-defined logs-based metrics and System logs-based metrics

Vegeta is a versatile HTTP load testing tool built out of a need to drill HTTP services with a constant request rate.

Logs-based metrics are Cloud Monitoring metrics that are based on the content of log entries.
