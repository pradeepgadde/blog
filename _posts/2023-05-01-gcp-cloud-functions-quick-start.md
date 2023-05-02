---
layout: single
title:  "Cloud Functions: Qwik Start"
date:   2023-05-01 02:59:05 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# GCP Cloud Functions: Qwik Start

## Overview

A cloud function is a piece of code that runs in response to an  event, such as an HTTP request, a message from a messaging service, or a file upload. Cloud events are *things* that happen in your cloud environment. These might be things like changes to data in a database,  files added to a storage system, or a new virtual machine instance being created.

Since cloud functions are event-driven, they only run when something  happens. This makes them a good choice for tasks that need to be done  quickly or that don't need to be running all the time.

For example, you can use a cloud function to:

- automatically generate thumbnails for images that are uploaded to Cloud Storage.
- send a notification to a user's phone when a new message is received in Cloud Pub/Sub.
- process data from a Cloud Firestore database and generate a report.

You can write your code in any language that supports Node.js, and  you can deploy your code to the cloud with a few clicks. Once your cloud function is deployed, it will automatically start running in response  to events.

- Create a simple cloud function
- Deploy and test the function
- View logs

## Task 1. Create a function

First, you're going to create a simple function named helloWorld.  This function writes a message to the Cloud Functions logs. It is  triggered by cloud function events and accepts a callback function used  to signal completion of the function.

For this lab the cloud function event is a cloud pub/sub topic event. A pub/sub is a messaging service where the senders of messages are  decoupled from the receivers of messages. When a message is sent or  posted, a subscription is required for a receiver to be alerted and  receive the message. To learn more about pub/subs, in Cloud Pub/Sub  Guides, see [Google Cloud Pub/Sub: A Google-Scale Messaging Service](https://cloud.google.com/pubsub/architecture).

To learn more about the event parameter and the callback parameter, in Cloud Functions Documentation, see [Background Functions](https://cloud.google.com/functions/docs/writing/background).

To create a cloud function:

1. In Cloud Shell, run the following command to set the default region:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-6b5fdf471c94.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-00-6b5fdf471c94)$ gcloud config set compute/region us-west1
Updated property [compute/region].
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-00-6b5fdf471c94)$ mkdir gcf_hello_world
student_01_a0075c49c713@cloudshell:~ (qwiklabs-gcp-00-6b5fdf471c94)$ cd gcf_hello_world
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ nano index.js

```

```js
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ cat index.js
/**
* Background Cloud Function to be triggered by Pub/Sub.
* This function is exported by index.js, and executed when
* the trigger topic receives a message.
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";
console.log(`My Cloud Function: ${name}`);
};
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

## Task 2. Create a cloud storage bucket

- Use the following command to create a new cloud storage bucket for your function:

```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gsutil mb -p qwiklabs-gcp-00-6b5fdf471c94 gs://pradeepgadde
Creating gs://pradeepgadde/...
ServiceException: 409 A Cloud Storage bucket named 'pradeepgadde' already exists. Try another name. Bucket names must be globally unique across all Google Cloud projects, including those outside of your organization.
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gsutil mb -p qwiklabs-gcp-00-6b5fdf471c94 gs://gaddepradeep
Creating gs://gaddepradeep/...
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

## Task 3. Deploy your function

When deploying a new function, you must specify `--trigger-topic`, `--trigger-bucket`, or `--trigger-http`. When deploying an update to an existing function, the function keeps the existing trigger unless otherwise specified.

For this lab, you'll set the `--trigger-topic` as `hello_world`.



Deploy the function to a pub/sub topic named **hello_world**, replacing `[BUCKET_NAME]` with the name of your bucket:

```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gcloud functions deploy helloWorld \
  --stage-bucket gaddepradeep \
  --trigger-topic hello_world \
  --runtime nodejs8
WARNING: The nodejs8 runtime is deprecated on Cloud Functions. Please migrate to a newer Node.js version (--runtime=nodejs12). See https://cloud.google.com/functions/docs/migrating/nodejs-runtimes
Deploying function (may take a while - up to 2 minutes)...working   
For Cloud Build Logs, visit: https://console.cloud.google.com/cloud-build/builds;region=us-central1/703981fa-8dea-460a-86a3-e9f7f32093f6?project=146430729049
Deploying function (may take a while - up to 2 minutes)...done.     
availableMemoryMb: 256
buildId: 703981fa-8dea-460a-86a3-e9f7f32093f6
buildName: projects/146430729049/locations/us-central1/builds/703981fa-8dea-460a-86a3-e9f7f32093f6
dockerRegistry: CONTAINER_REGISTRY
entryPoint: helloWorld
eventTrigger:
  eventType: google.pubsub.topic.publish
  failurePolicy: {}
  resource: projects/qwiklabs-gcp-00-6b5fdf471c94/topics/hello_world
  service: pubsub.googleapis.com
ingressSettings: ALLOW_ALL
labels:
  deployment-tool: cli-gcloud
maxInstances: 5
name: projects/qwiklabs-gcp-00-6b5fdf471c94/locations/us-central1/functions/helloWorld
runtime: nodejs8
serviceAccountEmail: qwiklabs-gcp-00-6b5fdf471c94@appspot.gserviceaccount.com
sourceArchiveUrl: gs://gaddepradeep/us-central1-projects/qwiklabs-gcp-00-6b5fdf471c94/locations/us-central1/functions/helloWorld-jxpzibtzrojh.zip
status: ACTIVE
timeout: 60s
updateTime: '2023-05-02T01:59:51.340Z'
versionId: '1'
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```



```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gcloud functions describe helloWorld
availableMemoryMb: 256
buildId: 703981fa-8dea-460a-86a3-e9f7f32093f6
buildName: projects/146430729049/locations/us-central1/builds/703981fa-8dea-460a-86a3-e9f7f32093f6
dockerRegistry: CONTAINER_REGISTRY
entryPoint: helloWorld
eventTrigger:
  eventType: google.pubsub.topic.publish
  failurePolicy: {}
  resource: projects/qwiklabs-gcp-00-6b5fdf471c94/topics/hello_world
  service: pubsub.googleapis.com
ingressSettings: ALLOW_ALL
labels:
  deployment-tool: cli-gcloud
maxInstances: 5
name: projects/qwiklabs-gcp-00-6b5fdf471c94/locations/us-central1/functions/helloWorld
runtime: nodejs8
serviceAccountEmail: qwiklabs-gcp-00-6b5fdf471c94@appspot.gserviceaccount.com
sourceArchiveUrl: gs://gaddepradeep/us-central1-projects/qwiklabs-gcp-00-6b5fdf471c94/locations/us-central1/functions/helloWorld-jxpzibtzrojh.zip
status: ACTIVE
timeout: 60s
updateTime: '2023-05-02T01:59:51.340Z'
versionId: '1'
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

Every message published in the topic triggers function execution, the message contents are passed as input data.

## Task 4. Test the function

After you deploy the function and know that it's active, test that  the function writes a message to the cloud log after detecting an event.

- Enter this command to create a message test of the function:

```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
executionId: z8gu6up9lfet
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

The cloud tool returns the execution ID for the function, which means a message has been written in the log.

View logs to confirm that there are log messages with that execution ID.

## Task 5. View logs

- Check the logs to see your messages in the log history:

```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gcloud functions logs read helloWorld
LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:18.020
LOG: Function execution took 872 ms, finished with status: 'ok'

LEVEL: I
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:18.007
LOG: My Cloud Function: Hello World!

LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:17.148
LOG: Function execution started
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

Your application is deployed, tested, and you can view the logs.

```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ DATA=$(printf 'Google Cloud is Awesome!!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
executionId: z8gucjoti2f8
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```



```sh
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$ gcloud functions logs read helloWorld
LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gucjoti2f8
TIME_UTC: 2023-05-02 02:05:16.068
LOG: Function execution took 12 ms, finished with status: 'ok'

LEVEL: I
NAME: helloWorld
EXECUTION_ID: z8gucjoti2f8
TIME_UTC: 2023-05-02 02:05:16.063
LOG: My Cloud Function: Google Cloud is Awesome!!

LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gucjoti2f8
TIME_UTC: 2023-05-02 02:05:16.056
LOG: Function execution started

LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:18.020
LOG: Function execution took 872 ms, finished with status: 'ok'

LEVEL: I
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:18.007
LOG: My Cloud Function: Hello World!

LEVEL: D
NAME: helloWorld
EXECUTION_ID: z8gu6up9lfet
TIME_UTC: 2023-05-02 02:02:17.148
LOG: Function execution started
student_01_a0075c49c713@cloudshell:~/gcf_hello_world (qwiklabs-gcp-00-6b5fdf471c94)$
```

 The logs will take around 10 mins to appear. Also, the alternative way to view the logs is, go to **Logging** > **Logs Explorer**

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-cloud-functions-1.png)
