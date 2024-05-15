---

layout: single
title:  "Dataproc: Qwik Start"
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

# Dataproc: Qwik Start 

Dataproc is a fast, easy-to-use, fully-managed cloud service for running  [Apache Spark](http://spark.Apache.org/) and  [Apache Hadoop](http://hadoop.Apache.org/) clusters in a simpler, more cost-efficient way. Operations that used to take hours or days take seconds or minutes instead. Create Dataproc  clusters quickly and resize them at any time, so you don't have to worry about your data pipelines outgrowing your clusters.

- Create a Dataproc cluster using the command line
- Run a simple Apache Spark job
- Modify the number of workers in the cluster

## Create a cluster

Dataproc creates staging and temp buckets that are shared among clusters in the same region. Since we're not specifying an account for Dataproc  to use, it will use the Compute Engine default service account, which  doesn't have storage bucket permissions by default.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-c826dbd8eef5.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud config set dataproc/region us-east1
Updated property [dataproc/region].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ PROJECT_ID=$(gcloud config get-value project) && \
gcloud config set project $PROJECT_ID

PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
Your active configuration is: [cloudshell-5691]
Updated property [core/project].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
  --role=roles/storage.admin
Updated IAM policy for project [qwiklabs-gcp-03-c826dbd8eef5].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-c826dbd8eef5@qwiklabs-gcp-03-c826dbd8eef5.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:68615778665@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-68615778665@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-68615778665@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-68615778665@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:service-68615778665@dataproc-accounts.iam.gserviceaccount.com
  role: roles/dataproc.serviceAgent
- members:
  - serviceAccount:68615778665-compute@developer.gserviceaccount.com
  - serviceAccount:68615778665@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-c826dbd8eef5@qwiklabs-gcp-03-c826dbd8eef5.iam.gserviceaccount.com
  - user:student-01-aa8f3a74b714@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:68615778665-compute@developer.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-c826dbd8eef5@qwiklabs-gcp-03-c826dbd8eef5.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-aa8f3a74b714@qwiklabs.net
  role: roles/viewer
etag: BwYYdHjRmsQ=
version: 1
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud dataproc clusters create example-cluster --worker-boot-disk-size 500 --worker-machine-type=e2-standard-4 --master-machine-type=e2-standard-4
Waiting on operation [projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/operations/95d1e7af-0b74-3a9f-b03f-49cba0b8cffa].
Waiting for cluster creation operation...working.                                                                                                                                  
WARNING: No image specified. Using the default image version. It is recommended to select a specific image version in production, as the default image version may change at any time.
WARNING: For PD-Standard without local SSDs, we strongly recommend provisioning 1TB or larger to ensure consistently high I/O performance. See https://cloud.google.com/compute/docs/disks/performance for information on disk I/O performance.
WARNING: The firewall rules for specified network or subnetwork would allow ingress traffic from 0.0.0.0/0, which could be a security risk.
Waiting for cluster creation operation...done.                                                                                                                                     
Created [https://dataproc.googleapis.com/v1/projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/clusters/example-cluster] Cluster placed in zone [us-east1-d].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ 
```



## Submit a job

- Run this command to submit a sample Spark job that calculates a rough value for pi:

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud dataproc jobs submit spark --cluster example-cluster \
  --class org.apache.spark.examples.SparkPi \
  --jars file:///usr/lib/spark/examples/jars/spark-examples.jar -- 1000
Job [55b14246481644648d805a0076d080b2] submitted.
Waiting for job output...
24/05/15 02:03:21 INFO org.apache.spark.SparkEnv: Registering MapOutputTracker
24/05/15 02:03:21 INFO org.apache.spark.SparkEnv: Registering BlockManagerMaster
24/05/15 02:03:21 INFO org.apache.spark.SparkEnv: Registering BlockManagerMasterHeartbeat
24/05/15 02:03:22 INFO org.apache.spark.SparkEnv: Registering OutputCommitCoordinator
24/05/15 02:03:22 INFO org.sparkproject.jetty.util.log: Logging initialized @4788ms to org.sparkproject.jetty.util.log.Slf4jLog
24/05/15 02:03:22 INFO org.sparkproject.jetty.server.Server: jetty-9.4.40.v20210413; built: 2021-04-13T20:42:42.668Z; git: b881a572662e1943a14ae12e7e1207989f218b74; jvm 1.8.0_412-b08
24/05/15 02:03:22 INFO org.sparkproject.jetty.server.Server: Started @4906ms
24/05/15 02:03:22 INFO org.sparkproject.jetty.server.AbstractConnector: Started ServerConnector@30d25c03{HTTP/1.1, (http/1.1)}{0.0.0.0:39419}
24/05/15 02:03:23 INFO org.apache.hadoop.yarn.client.RMProxy: Connecting to ResourceManager at example-cluster-m/10.142.0.3:8032
24/05/15 02:03:23 INFO org.apache.hadoop.yarn.client.AHSProxy: Connecting to Application History server at example-cluster-m/10.142.0.3:10200
24/05/15 02:03:24 INFO org.apache.hadoop.conf.Configuration: resource-types.xml not found
24/05/15 02:03:24 INFO org.apache.hadoop.yarn.util.resource.ResourceUtils: Unable to find 'resource-types.xml'.
24/05/15 02:03:26 INFO org.apache.hadoop.yarn.client.api.impl.YarnClientImpl: Submitted application application_1715738475083_0001
24/05/15 02:03:27 INFO org.apache.hadoop.yarn.client.RMProxy: Connecting to ResourceManager at example-cluster-m/10.142.0.3:8030
24/05/15 02:03:29 INFO com.google.cloud.hadoop.fs.gcs.GhfsStorageStatistics: Detected potential high latency for operation op_get_file_status. latencyMs=308; previousMaxLatencyMs=0; operationCount=1; context=gs://dataproc-temp-us-east1-68615778665-jlffzgeo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/spark-job-history
24/05/15 02:03:29 INFO com.google.cloud.hadoop.repackaged.gcs.com.google.cloud.hadoop.gcsio.GoogleCloudStorageImpl: Ignoring exception of type GoogleJsonResponseException; verified object already exists with desired state.
24/05/15 02:03:29 INFO com.google.cloud.hadoop.fs.gcs.GhfsStorageStatistics: Detected potential high latency for operation op_mkdirs. latencyMs=199; previousMaxLatencyMs=0; operationCount=1; context=gs://dataproc-temp-us-east1-68615778665-jlffzgeo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/spark-job-history
Pi is roughly 3.1413946714139467
24/05/15 02:03:44 INFO org.sparkproject.jetty.server.AbstractConnector: Stopped Spark@30d25c03{HTTP/1.1, (http/1.1)}{0.0.0.0:0}
24/05/15 02:03:44 INFO com.google.cloud.hadoop.fs.gcs.GhfsStorageStatistics: Detected potential high latency for operation stream_write_close_operations. latencyMs=163; previousMaxLatencyMs=0; operationCount=1; context=gs://dataproc-temp-us-east1-68615778665-jlffzgeo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/spark-job-history/application_1715738475083_0001.inprogress
24/05/15 02:03:45 INFO com.google.cloud.hadoop.fs.gcs.GhfsStorageStatistics: Detected potential high latency for operation op_rename. latencyMs=176; previousMaxLatencyMs=0; operationCount=1; context=rename(gs://dataproc-temp-us-east1-68615778665-jlffzgeo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/spark-job-history/application_1715738475083_0001.inprogress -> gs://dataproc-temp-us-east1-68615778665-jlffzgeo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/spark-job-history/application_1715738475083_0001)
Job [55b14246481644648d805a0076d080b2] finished successfully.
done: true
driverControlFilesUri: gs://dataproc-staging-us-east1-68615778665-cqnm9mcc/google-cloud-dataproc-metainfo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/jobs/55b14246481644648d805a0076d080b2/
driverOutputResourceUri: gs://dataproc-staging-us-east1-68615778665-cqnm9mcc/google-cloud-dataproc-metainfo/c95a4167-c1a0-4fc0-8fb3-a3e0241882e8/jobs/55b14246481644648d805a0076d080b2/driveroutput
jobUuid: 501986b0-3dc5-3f66-926a-b6fb6d9d527c
placement:
  clusterName: example-cluster
  clusterUuid: c95a4167-c1a0-4fc0-8fb3-a3e0241882e8
reference:
  jobId: 55b14246481644648d805a0076d080b2
  projectId: qwiklabs-gcp-03-c826dbd8eef5
sparkJob:
  args:
  - '1000'
  jarFileUris:
  - file:///usr/lib/spark/examples/jars/spark-examples.jar
  mainClass: org.apache.spark.examples.SparkPi
status:
  state: DONE
  stateStartTime: '2024-05-15T02:03:45.734622Z'
statusHistory:
- state: PENDING
  stateStartTime: '2024-05-15T02:03:15.770291Z'
- state: SETUP_DONE
  stateStartTime: '2024-05-15T02:03:15.799282Z'
- details: Agent reported job success
  state: RUNNING
  stateStartTime: '2024-05-15T02:03:16.126665Z'
yarnApplications:
- name: Spark Pi
  progress: 1.0
  state: FINISHED
  trackingUrl: http://example-cluster-m:8088/proxy/application_1715738475083_0001/
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ 
```

The command specifies:

- That you want to run a  [spark](https://cloud.google.com/sdk/gcloud/reference/dataproc/jobs/submit/spark) job on the `example-cluster` cluster
- The `class` containing the main method for the job's pi-calculating application
- The location of the jar file containing your job's code
- The parameters you want to pass to the job—in this case, the number of tasks, which is `1000`

## Update a cluster

1. To change the number of workers in the cluster to four, run the following command:

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud dataproc clusters update example-cluster --num-workers 4
Waiting on operation [projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/operations/d6a32aee-61c2-3743-b27f-ede208c7944f].
Waiting for cluster update operation...done.                                                                                                                                       
Updated [https://dataproc.googleapis.com/v1/projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/clusters/example-cluster].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ 
```



```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ gcloud dataproc clusters update example-cluster --num-workers 2
Waiting on operation [projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/operations/6e5827e0-3808-353c-8eca-7a30a166d2c8].
Waiting for cluster update operation...working..                                                             Waiting for cluster update operation...done.                                                                                                                                       
Updated [https://dataproc.googleapis.com/v1/projects/qwiklabs-gcp-03-c826dbd8eef5/regions/us-east1/clusters/example-cluster].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ 
```



## Summary

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ history 
    1  gcloud config set dataproc/region us-east1
    2  PROJECT_ID=$(gcloud config get-value project) && gcloud config set project $PROJECT_ID
    3  PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
    4  gcloud projects add-iam-policy-binding $PROJECT_ID   --member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com   --role=roles/storage.admin
    5  gcloud dataproc clusters create example-cluster --worker-boot-disk-size 500 --worker-machine-type=e2-standard-4 --master-machine-type=e2-standard-4
    6  gcloud dataproc jobs submit spark --cluster example-cluster   --class org.apache.spark.examples.SparkPi   --jars file:///usr/lib/spark/examples/jars/spark-examples.jar -- 1000
    7  gcloud dataproc clusters update example-cluster --num-workers 4
    8  gcloud dataproc clusters update example-cluster --num-workers 2
    9  history 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-c826dbd8eef5)$ 
```

