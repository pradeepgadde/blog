---
layout: single
title:  "Optimizing Cost with Google Cloud Storage"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Optimizing Cost with Google Cloud Storage

Use Cloud Functions and Cloud Scheduler to identify and clean up wasted cloud resources. You trigger a Cloud Function to migrate a storage bucket from a Cloud Monitoring alerting policy to a less expensive storage class.

Google Cloud provides storage object lifecycle rules that automatically moves objects to different storage classes based on a set of attributes, such as their creation date or live state. However, these rules can’t take into account whether the objects have been accessed. Sometimes, you might want to move newer objects to Nearline storage if they haven’t been accessed for a certain period of time.


Create two storage buckets, add a file to the serving-bucket, and generate traffic against it.
Create a Cloud Monitoring dashboard to visualize bucket utilization.
Deploy a Cloud Function to migrate the idle bucket to a less expensive storage class, and trigger the function by using a payload intended to mock a notification received from a Cloud alerting policy.

## Enable APIs and clone repository
In Cloud Shell, enable the Cloud Scheduler API:
```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-4b99f66c32a6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-4b99f66c32a6)$ gcloud services enable cloudscheduler.googleapis.com
Operation "operations/acf.p2-355065945769-8f364048-ead1-46a5-8f6c-f520f0b1a07a" finished successfully.
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Clone the repository:

```sh
student_01_9b51dda9fd9c@cloudshell:~ (qwiklabs-gcp-04-4b99f66c32a6)$ git clone https://github.com/GoogleCloudPlatform/gcf-automated-resource-cleanup.git && cd gcf-automated-resource-cleanup/
Cloning into 'gcf-automated-resource-cleanup'...
remote: Enumerating objects: 206, done.
remote: Counting objects: 100% (37/37), done.
remote: Compressing objects: 100% (31/31), done.
remote: Total 206 (delta 21), reused 9 (delta 6), pack-reused 169
Receiving objects: 100% (206/206), 51.60 KiB | 7.37 MiB/s, done.
Resolving deltas: 100% (103/103), done.
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Set environment variables and make the repository folder your $WORKDIR where you run all commands related to this lab:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ export PROJECT_ID=$(gcloud config list --format 'value(core.project)' 2>/dev/null)
WORKDIR=$(pwd)
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Install [Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html), an open source load-generation tool:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ sudo apt-get install apache2-utils -y
********************************************************************************
You are running apt-get inside of Cloud Shell. Note that your Cloud Shell  
machine is ephemeral and no system-wide change will persist beyond session end. 

To suppress this warning, create an empty ~/.cloudshell/no-apt-get-warning file.
The command will automatically proceed in 5 seconds or on any key. 

Visit https://cloud.google.com/shell/help for more information.                 
********************************************************************************
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libapr1 libaprutil1
The following NEW packages will be installed:
  apache2-utils libapr1 libaprutil1
0 upgraded, 3 newly installed, 0 to remove and 9 not upgraded.
Need to get 290 kB of archives.
After this operation, 993 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libapr1 amd64 1.7.0-8ubuntu0.22.04.1 [108 kB]
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 libaprutil1 amd64 1.6.1-5ubuntu4.22.04.2 [92.8 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 apache2-utils amd64 2.4.52-1ubuntu4.9 [88.7 kB]
Fetched 290 kB in 2s (151 kB/s)         
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libapr1:amd64.
(Reading database ... 121583 files and directories currently installed.)
Preparing to unpack .../libapr1_1.7.0-8ubuntu0.22.04.1_amd64.deb ...
Unpacking libapr1:amd64 (1.7.0-8ubuntu0.22.04.1) ...
Selecting previously unselected package libaprutil1:amd64.
Preparing to unpack .../libaprutil1_1.6.1-5ubuntu4.22.04.2_amd64.deb ...
Unpacking libaprutil1:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Selecting previously unselected package apache2-utils.
Preparing to unpack .../apache2-utils_2.4.52-1ubuntu4.9_amd64.deb ...
Unpacking apache2-utils (2.4.52-1ubuntu4.9) ...
Setting up libapr1:amd64 (1.7.0-8ubuntu0.22.04.1) ...
Setting up libaprutil1:amd64 (1.6.1-5ubuntu4.22.04.2) ...
Setting up apache2-utils (2.4.52-1ubuntu4.9) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.6) ...
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ 
```



## Create cloud storage buckets and add a file

In Cloud Shell, navigate to the `migrate-storage` directory:

Create `serving-bucket`, the Cloud Storage bucket. You use this later to change storage classes:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ cd $WORKDIR/migrate-storage
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$  export PROJECT_ID=$(gcloud config list --format 'value(core.project)' 2>/dev/null)
gcloud storage buckets create  gs://${PROJECT_ID}-serving-bucket -l us-west1
Creating gs://qwiklabs-gcp-04-4b99f66c32a6-serving-bucket/...
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Make the bucket public:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gsutil acl ch -u allUsers:R gs://${PROJECT_ID}-serving-bucket
Updated ACL on gs://qwiklabs-gcp-04-4b99f66c32a6-serving-bucket/
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Add a text file to the bucket:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gcloud storage cp $WORKDIR/migrate-storage/testfile.txt  gs://${PROJECT_ID}-serving-bucket
Copying file:///home/student_01_9b51dda9fd9c/gcf-automated-resource-cleanup/migrate-storage/testfile.txt to gs://qwiklabs-gcp-04-4b99f66c32a6-serving-bucket/testfile.txt
  Completed files 1/1 | 14.0B/14.0B                                                                                                                                                
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Make the file public:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gsutil acl ch -u allUsers:R gs://${PROJECT_ID}-serving-bucket/testfile.txt
Updated ACL on gs://qwiklabs-gcp-04-4b99f66c32a6-serving-bucket/testfile.txt
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Confirm that you’re able to access the file:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ curl http://storage.googleapis.com/${PROJECT_ID}-serving-bucket/testfile.txt
this is a teststudent_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Create a second bucket called idle-bucket that won’t serve any data:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gcloud storage buckets create gs://${PROJECT_ID}-idle-bucket -l us-west1
Creating gs://qwiklabs-gcp-04-4b99f66c32a6-idle-bucket/...
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Verify the storage class of each bucket

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gsutil defstorageclass get gs://$PROJECT_ID-idle-bucket
gs://qwiklabs-gcp-04-4b99f66c32a6-idle-bucket: STANDARD
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gsutil defstorageclass get gs://$PROJECT_ID-serving-bucket
gs://qwiklabs-gcp-04-4b99f66c32a6-serving-bucket: STANDARD
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```



## Create a monitoring dashboard

### Create a Monitoring Metrics Scope

Set up a Monitoring Metrics Scope that's tied to your Google Cloud  Project. The following steps create a new account that has a free trial  of Monitoring.

- In the Cloud Console, click **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) > **Monitoring**.

When the Monitoring **Overview** page opens, your metrics scope project is ready.

1. In the left panel, click **Dashboards** > **+Create Dashboard**.
2. Name the Dashboard `Bucket Usage`.
3. Click **+ADD WIDGET**.
4. Click **Line Chart**.
5. For **Widget Title**, type `Bucket Access`.
6. For **select a metric** > **Metric** type **gcs_bucket** for the resource, select **GCS Bucket > Api > Request count** metric and click **Apply**.
7. Click **+ Add Filter**.

To filter by the method name:

- For **Label**, select **method**.
- For **Comparison**, select **=(equals)**.
- For **Value**, select **ReadObject**.
- Click **Apply**.



1. To group the metrics by bucket name, in the **Group By** drop-down list, select **bucket_name** and click **ok**.

You’ve configured Cloud Monitoring to observe object access in your  buckets. There's no data in the chart because there's no traffic to the  Cloud Storage buckets.

## Generate load on the serving bucket

Now that you configured monitoring, use Apache Bench to send traffic to `serving-bucket`.

1. In Cloud Shell, send requests to the object in the serving bucket:
```sh
tudent_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ ab -n 10000 http://storage.googleapis.com/$PROJECT_ID-serving-bucket/testfile.txt
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking storage.googleapis.com (be patient)
Completed 1000 requests


Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
^C

Server Software:        UploadServer
Server Hostname:        storage.googleapis.com
Server Port:            80

Document Path:          /qwiklabs-gcp-04-4b99f66c32a6-serving-bucket/testfile.txt
Document Length:        14 bytes

Concurrency Level:      1
Time taken for tests:   1750.890 seconds
Complete requests:      4226
Failed requests:        0
Total transferred:      2847378 bytes
HTML transferred:       59164 bytes
Requests per second:    2.41 [#/sec] (mean)
Time per request:       414.314 [ms] (mean)
Time per request:       414.314 [ms] (mean, across all concurrent requests)
Transfer rate:          1.59 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   0.5      1      13
Processing:   169  413 355.1    198    1869
Waiting:      169  413 355.1    197    1868
Total:        170  414 355.1    198    1870

Percentage of the requests served within a certain time (ms)
  50%    198
  66%    225
  75%    950
  80%    965
  90%    983
  95%   1005
  98%   1138
  99%   1188
 100%   1870 (longest request)
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$
```

## Review and deploy the Cloud Function

```py
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ cat migrate-storage/main.py | grep "migrate_storage(" -A 15
def migrate_storage(request):
    # process incoming request to get the bucket to be migrated:
    request_json = request.get_json(force=True)
    # bucket names are globally unique
    bucket_name = request_json['incident']['resource_name']

    # create storage client
    storage_client = storage.Client(project)

    # get bucket
    bucket = storage_client.get_bucket(bucket_name)

    # update storage class
    bucket.storage_class = "NEARLINE"
    bucket.patch()

student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

```py
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ cat migrate-storage/main.py 
#Copyright 2019 Google LLC
#Licensed under the Apache License, Version 2.0 (the "License");
#you may not use this file except in compliance with the License.
#You may obtain a copy of the License at
#https://www.apache.org/licenses/LICENSE-2.0
#Unless required by applicable law or agreed to in writing, software
#distributed under the License is distributed on an "AS IS" BASIS,
#WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#See the License for the specific language governing permissions and
#limitations under the License.

# for execution in Cloud Functions Python 3.7.1

# modify these variables for your environment:
project = 'automating-cost-optimization'

# imports
import googleapiclient
from googleapiclient import discovery
from oauth2client.client import GoogleCredentials
from google.cloud import storage
import sys
from flask import request
from flask import Flask
from flask import escape


# initialize global
compute = googleapiclient.discovery.build('compute', 'v1')
credentials = GoogleCredentials.get_application_default()

# main function
def migrate_storage(request):
    # process incoming request to get the bucket to be migrated:
    request_json = request.get_json(force=True)
    # bucket names are globally unique
    bucket_name = request_json['incident']['resource_name']

    # create storage client
    storage_client = storage.Client(project)

    # get bucket
    bucket = storage_client.get_bucket(bucket_name)

    # update storage class
    bucket.storage_class = "NEARLINE"
    bucket.patch()


student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Notice that the Cloud Function uses the bucket name passed in the request to change it's storage class to Nearline.

Deploy the Cloud Function:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gcloud functions deploy migrate_storage --trigger-http --runtime=python37 --region us-west1
In a future Cloud SDK release, new functions will be deployed as 2nd gen  functions by default. This is equivalent to currently deploying new  with the --gen2 flag. Existing 1st gen functions will not be impacted and will continue to deploy as 1st gen functions.
You can preview this behavior in beta. Alternatively, you can disable this behavior by explicitly specifying the --no-gen2 flag or by setting the functions/gen2 config property to 'off'.
To learn more about the differences between 1st gen and 2nd gen functions, visit:
https://cloud.google.com/functions/docs/concepts/version-comparison
WARNING: Python 3.7 is no longer supported by the Python community as of 27 June, 2023. Runtime python37 is currently deprecated for Cloud Functions. We recommend you to upgrade to the latest version of Python as soon as possible.
Deploying function (may take a while - up to 2 minutes)...working..                                                                                                                
For Cloud Build Logs, visit: https://console.cloud.google.com/cloud-build/builds;region=us-west1/c0f65510-084f-4023-a655-c066d4df1f92?project=355065945769
Deploying function (may take a while - up to 2 minutes)...done.                                                                                                                    
automaticUpdatePolicy: {}
availableMemoryMb: 256
buildId: c0f65510-084f-4023-a655-c066d4df1f92
buildName: projects/355065945769/locations/us-west1/builds/c0f65510-084f-4023-a655-c066d4df1f92
dockerRegistry: ARTIFACT_REGISTRY
entryPoint: migrate_storage
httpsTrigger:
  securityLevel: SECURE_OPTIONAL
  url: https://us-west1-qwiklabs-gcp-04-4b99f66c32a6.cloudfunctions.net/migrate_storage
ingressSettings: ALLOW_ALL
labels:
  deployment-tool: cli-gcloud
maxInstances: 5
name: projects/qwiklabs-gcp-04-4b99f66c32a6/locations/us-west1/functions/migrate_storage
runtime: python37
serviceAccountEmail: qwiklabs-gcp-04-4b99f66c32a6@appspot.gserviceaccount.com
sourceUploadUrl: https://storage.googleapis.com/uploads-581549625170.us-west1.cloudfunctions.appspot.com/a06d1fd9-ea63-4636-9437-91d14d707789.zip
status: ACTIVE
timeout: 60s
updateTime: '2024-04-25T02:35:47.503Z'
versionId: '2'
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

Capture the trigger URL into an environment variable that you use in the next section:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ export FUNCTION_URL=$(gcloud functions describe migrate_storage --format=json --region us-west1 | jq -r '.httpsTrigger.url')
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```



## Test and validate alerting automation

Set the idle bucket name:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ export IDLE_BUCKET_NAME=$PROJECT_ID-idle-bucket
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$
```



Send a test notification to the Cloud Function you deployed using the `incident.json` file:

```json
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ cat incident.json 
{
    "incident": {
      "incident_id": "f2e08c333dc64cb09f75eaab355393bz",
      "resource_id": "3773729775088064755",
      "resource_name": "$IDLE_BUCKET_NAME",
      "state": "open",
      "started_at": 1385085727,
      "ended_at": null,
      "policy_name": "Unused Bucket Objects",
      "condition_name": "Bucket Usage",
      "url": "https://app.google.stackdriver.com/incidents/f333dc64z",
      "summary": "storage.googleapis.com/storage/request_count  is below a threshold of .01 for greater than 23 hours 30 minutes"
    },
    "version": 1.1
  }student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ export IDLE_BUCKET_NAME=$PROJECT_ID-idle-bucket
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ export FUNCTION_URL=$(gcloud functions describe migrate_storage --format=json --region us-west1 | jq -r '.httpsTrigger.url')
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ envsubst < $WORKDIR/migrate-storage/incident.json | curl -X POST -H "Content-Type: application/json" $FUNCTION_URL -d @-
OKstudent_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```
Confirm that the idle bucket was migrated to Nearline:

```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ gsutil defstorageclass get gs://$PROJECT_ID-idle-bucket
gs://qwiklabs-gcp-04-4b99f66c32a6-idle-bucket: NEARLINE
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```



```sh
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ history 
    1  gcloud services enable cloudscheduler.googleapis.com
    2  git clone https://github.com/GoogleCloudPlatform/gcf-automated-resource-cleanup.git && cd gcf-automated-resource-cleanup/
    3  export PROJECT_ID=$(gcloud config list --format 'value(core.project)' 2>/dev/null)
    4  WORKDIR=$(pwd)
    5  sudo apt-get install apache2-utils -y
    6  cd $WORKDIR/migrate-storage
    7   export PROJECT_ID=$(gcloud config list --format 'value(core.project)' 2>/dev/null)
    8  gcloud storage buckets create  gs://${PROJECT_ID}-serving-bucket -l us-west1
    9  gsutil acl ch -u allUsers:R gs://${PROJECT_ID}-serving-bucket
   10  gcloud storage cp $WORKDIR/migrate-storage/testfile.txt  gs://${PROJECT_ID}-serving-bucket
   11  gsutil acl ch -u allUsers:R gs://${PROJECT_ID}-serving-bucket/testfile.txt
   12  curl http://storage.googleapis.com/${PROJECT_ID}-serving-bucket/testfile.txt
   13  gcloud storage buckets create gs://${PROJECT_ID}-idle-bucket -l us-west1
   14  gsutil defstorageclass get gs://$PROJECT_ID-idle-bucket
   15  gsutil defstorageclass get gs://$PROJECT_ID-serving-bucket
   16  ab -n 10000 http://storage.googleapis.com/$PROJECT_ID-serving-bucket/testfile.txt
   17  ab -n 10000 http://storage.googleapis.com/$PROJECT_ID-serving-bucket/testfile.txt
   18  export IDLE_BUCKET_NAME=$PROJECT_ID-idle-bucket
   19  envsubst < $WORKDIR/migrate-storage/incident.json | curl -X POST -H "Content-Type: application/json" $FUNCTION_URL -d @-
   20  export FUNCTION_URL=$(gcloud functions describe migrate_storage --format=json --region us-west1 | jq -r '.httpsTrigger.url')
   21  envsubst < $WORKDIR/migrate-storage/incident.json | curl -X POST -H "Content-Type: application/json" $FUNCTION_URL -d @-
   22  gsutil defstorageclass get gs://$PROJECT_ID-idle-bucket
   23  history 
student_01_9b51dda9fd9c@cloudshell:~/gcf-automated-resource-cleanup/migrate-storage (qwiklabs-gcp-04-4b99f66c32a6)$ 
```

