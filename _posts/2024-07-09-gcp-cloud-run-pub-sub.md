---

layout: single
title:  "Using Cloud PubSub with Cloud Run"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Using Cloud PubSub with Cloud Run 

Pub/Sub enables applications to take advantage of efficient message queues. The service is compatible with a range of Google Cloud services, and in this lab, you learn how to integrate it with Cloud Run.

This lab is based on resolving a customer use case by using serverless infrastructure. 

- Enable the Cloud Run API.
- Deploy microservices to Cloud Run.
- Create a Pub/Sub topic.
- Invoke a Cloud Run service from a Pub/Sub subscription.

In this lab, you will help the development team at Critter Junction investigate the use of Pub/Sub for their requirements. The team would like to explore how to perform efficient queue processing within their applications.

The team at Critter Junction has a public web application and several microservices built on Google Cloud. Communication between the microservices is critical and needs a  resilient form of messaging to be established between each application  component.

The development team's previous attempts were unsuccessful due to the microservices needing to know a lot about each other ( [High Coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming))). In addition, if a service was temporarily unavailable, messages would be lost.

The team needs a solution that includes a level of resilience without introducing additional service dependencies (Low Coupling) into their  systems. Now that you know a bit more about Critter Junction and the  issues they face, try to prioritize the key criteria for a solution.

After considering the requirements, the development team chooses Pub/Sub because they only require a push based distribution pattern. The  following high level architecture diagram summarizes the minimal viable  product (MVP) that they need to investigate.

![The MVP architecture diagram](https://cdn.qwiklabs.com/91ZhY5Zo%2F7HuSfGIMzAIRPynsyJHs7YsaYLWAd5sWOA%3D)

In the proposed solution, Pub/Sub will be used to handle asynchronous messages between services.

## Ensure that the Pub/Sub API is successfully enabled

To ensure access to the necessary API, re-enable the **Pub/Sub** API.

1. In the Google Cloud console **Navigation menu** (![navmenu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), under **APIs & Services**, click **Library**.

2. In the **Search** box, type **Pub/Sub**

3. Click the result for **Cloud Pub/Sub API**.

4. Click **Manage**.

5. Click **Disable API**. If asked to confirm, click **Disable**.

6. Again, when prompted `Do you want to disable Cloud Pub/Sub API and its dependent APIs?`, Click **Confirm**.

7. To re-enable the API, click **Enable**.

   When the API has been re-enabled, the page displays information about the API.

## Developing a minimal viable product (MVP)

Critter Junction has multiple Cloud Run services that they would like integrated with Pub/Sub. To build an MVP, the following tasks are required:

- Deploy a producer service
- Deploy a consumer service
- Create a service account
- Create a Pub/Sub topic

### Deploy a producer service

Critter Junction specifies that the externally facing *store* service should be configured as a public endpoint, indicating these requirements:

| **Type**          | **Permission**          | **Description**                                             |
| ----------------- | ----------------------- | ----------------------------------------------------------- |
| URL Access        | --allow-unauthenticated | Make the service PUBLIC (Unauthenticated users can see it). |
| Invoke Permission | allUsers                | Allow the service be invoked/triggered by anyone.           |

The producer *store* service accepts public internet based connections for *purchase orders*. To do this, the service must not require authentication and must be able to be triggered by anyone.

Information collected by this service will be passed to the backend consumer services.

Configure and deploy the **store** service on Cloud Run. Execute the following commands in Cloud Shell.

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”

student_01_e488ef31cb8a@cloudshell:~$ gcloud config set project qwiklabs-gcp-01-e4390abfab41 
Updated property [core/project].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ gcloud services enable run.googleapis.com
Operation "operations/acf.p2-1067452512535-2e8c055f-5cd6-43b3-b554-5be2cf48f410" finished successfully.
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  LOCATION=europe-west1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ gcloud config set compute/region $LOCATION
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud run deploy store-service \
  --image gcr.io/qwiklabs-resources/gsp724-store-service \
  --region $LOCATION \
  --allow-unauthenticated
Deploying container to Cloud Run service [store-service] in project [qwiklabs-gcp-01-e4390abfab41] region [europe-west1]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
  OK Setting IAM Policy...                                                                                                                                                         
Done.                                                                                                                                                                              
Service [store-service] revision [store-service-00001-9nl] has been deployed and is serving 100 percent of traffic.
Service URL: https://store-service-kmr4fqo4aa-ew.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



Once the store service is deployed, the store service is publicly accessible over the internet.

### Deploy a consumer service

The development team also needs to configure the *order* service that can be accessed at a private endpoint. Unlike the **store** service, the **order**  service is not meant to be publicly accessible over the internet, and  should only be invoked by an account with the appropriate permissions.

For Cloud Run based services, this can be achieved by using the following settings:

| **Type**               | **Permission**             | **Description**                                              |
| ---------------------- | -------------------------- | ------------------------------------------------------------ |
| URL Access             | --no-allow-unauthenticated | Make the service PRIVATE (Only authenticated users can see it). |
| Invoke Role/Permission | Cloud Run Invoker          | Only allow the service to be invoked by an account with the Cloud Run Invoker role. |

Configure and deploy the **order service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud run deploy order-service \
  --image gcr.io/qwiklabs-resources/gsp724-order-service \
  --region $LOCATION \
  --no-allow-unauthenticated
Deploying container to Cloud Run service [order-service] in project [qwiklabs-gcp-01-e4390abfab41] region [europe-west1]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
Done.                                                                                                                                                                              
Service [order-service] revision [order-service-00001-964] has been deployed and is serving 100 percent of traffic.
Service URL: https://order-service-kmr4fqo4aa-ew.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



Now only authenticated accounts can access and invoke the service.

## Pub/Sub overview

Pub/Sub is an asynchronous messaging service that decouples services  that produce events from services that consume and process events.

Pub/Sub core concepts:

- Topic
- Subscription
- Message
- Message attribute

Pub/Sub requires a couple of options to be completed prior to successful deployment. In the Google Cloud console, Pub/Sub can be accessed under the Big Data menu option.

## Deploying Pub/Sub

Now that the producer (`store service`) and consumer (`order service`) services have been successfully deployed, you can focus on the main  features of Pub/Sub. Using Pub/Sub requires two activities:

- Create a Topic
- Create a Subscription

### Create a Topic

When an asynchronous (push) event is created on a topic, applications that subscribe to the topic will be able to process the associated  messages. [Push event processing with Pub/Sub](http://cloud.google.com/run/docs/events/pubsub-push) provides a scalable way to handle messaging on Google Cloud.

The new Pub/Sub Topic will have following values.

| **Field**  | **Value**          |
| ---------- | ------------------ |
| Name       | ORDER_PLACED       |
| Encryption | Google-managed key |

1. Create a Topic in Pub/Sub:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ gcloud pubsub topics create ORDER_PLACED
Created topic [projects/qwiklabs-gcp-01-e4390abfab41/topics/ORDER_PLACED].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



1. By creating the Pub/Sub Topic, messages can now be independently stored and delivered in a resilient manner.

   Messages that are sent using Pub/Sub are encoded as base64 on transmission, and need to be decoded on receipt.

   You'll create a subscription in a subsequent task.

   

## Creating a service account

To deliver a Pub/Sub message to a Cloud Run service, you need a  Pub/Sub subscription. The subscription must be able to invoke the  service using a service account with the appropriate permissions. In  this lab, the consumer **order** service will be invoked by a subscription using the service account.

To achieve this functionality, the following activities are required:

- Create a Service Account
- Bind the Invoker Role permissions to the service account

### Service account creation

Create a new service account that will provide authenticated access.

Create a new service account called **Order Initiator**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud iam service-accounts create pubsub-cloud-run-invoker \
  --display-name "Order Initiator"
Created service account [pubsub-cloud-run-invoker].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```
Confirm that the service account has been created:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud iam service-accounts list --filter="Order Initiator"
DISPLAY NAME: Order Initiator
EMAIL: pubsub-cloud-run-invoker@qwiklabs-gcp-01-e4390abfab41.iam.gserviceaccount.com
DISABLED: False
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



At this point, the **Order Initiator** service account is available. However, it does not have a role or permissions assigned. To assign it IAM permissions, you need to apply or bind role permissions to the service account.

### Bind role permissions

To bind permissions to an account that is used to invoke a service on Cloud Run, you need the following information:

| **Category** | **Description**                                              |
| ------------ | ------------------------------------------------------------ |
| Service Name | The name of the deployed service to be invoked.              |
| Member       | The account to bestow the role permissions.                  |
| Region       | The region in which the service is deployed.                 |
| Platform     | The platform type (Cloud Run Managed, Cloud Run for Anthos, or Cloud Run for VMWare) |

Bind the service account with the role `Cloud Run Invoker` on the **order service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud run services add-iam-policy-binding order-service --region $LOCATION \
  --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker --platform managed
Updated IAM policy for service [order-service].
bindings:
- members:
  - serviceAccount:pubsub-cloud-run-invoker@qwiklabs-gcp-01-e4390abfab41.iam.gserviceaccount.com
  role: roles/run.invoker
etag: BwYc06ORVjE=
version: 1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

The new service account has now been given permissions to invoke a Cloud Run service.

Create an environment variable to store the project number:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  PROJECT_NUMBER=$(gcloud projects list \
  --filter="qwiklabs-gcp" \
  --format='value(PROJECT_NUMBER)')
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

Enable the project service account to create tokens:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
   --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
   --role=roles/iam.serviceAccountTokenCreator
Updated IAM policy for project [qwiklabs-gcp-01-e4390abfab41].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-01-e4390abfab41@qwiklabs-gcp-01-e4390abfab41.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:1067452512535@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-1067452512535@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-1067452512535@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-1067452512535@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:service-1067452512535@containerregistry.iam.gserviceaccount.com
  role: roles/containerregistry.ServiceAgent
- members:
  - serviceAccount:1067452512535-compute@developer.gserviceaccount.com
  - serviceAccount:1067452512535@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-1067452512535@gcp-sa-pubsub.iam.gserviceaccount.com
  role: roles/iam.serviceAccountTokenCreator
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-01-e4390abfab41@qwiklabs-gcp-01-e4390abfab41.iam.gserviceaccount.com
  - user:student-01-e488ef31cb8a@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:service-1067452512535@gcp-sa-pubsub.iam.gserviceaccount.com
  role: roles/pubsub.serviceAgent
- members:
  - serviceAccount:service-1067452512535@serverless-robot-prod.iam.gserviceaccount.com
  role: roles/run.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-01-e4390abfab41@qwiklabs-gcp-01-e4390abfab41.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-e488ef31cb8a@qwiklabs.net
  role: roles/viewer
etag: BwYc06ibZ20=
version: 1
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



## Create a Pub/Sub subscription

In this task, you create the Pub/Sub subscription and configure it to use the new service account.

1. Create an environment variable to store the endpoint of the **order service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  ORDER_SERVICE_URL=$(gcloud run services describe order-service \
   --region $LOCATION \
   --format="value(status.address.url)")
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ echo $ORDER_SERVICE_URL 
https://order-service-kmr4fqo4aa-ew.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

Create a subscription and bind it to the **order service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  gcloud pubsub subscriptions create order-service-sub \
   --topic ORDER_PLACED \
   --push-endpoint=$ORDER_SERVICE_URL \
   --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
Created subscription [projects/qwiklabs-gcp-01-e4390abfab41/subscriptions/order-service-sub].
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



## Testing the application

To test the application, send a sample JSON payload to the store service.

1. Create a file called `test.json` with the following content. You can use your choice of editor such as `nano`, `vi`, or the Cloud Shell editor.

```json
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ cat test.json 
{
 "billing_address": {
   "name": "Kylie Scull",
   "address": "6471 Front Street",
   "city": "Mountain View",
   "state_province": "CA",
   "postal_code": "94043",
   "country": "US"
 },
 "shipping_address": {
   "name": "Kylie Scull",
   "address": "9902 Cambridge Grove",
   "city": "Martinville",
   "state_province": "BC",
   "postal_code": "V1A",
   "country": "Canada"
 },
 "items": [
   {
     "id": "RW134",
     "quantity": 1,
     "sub-total": 12.95
   },
   {
     "id": "IB541",
     "quantity": 2,
     "sub-total": 24.5
   }
 ]
}
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

Create an environment variable to store the endpoint of the **store service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  STORE_SERVICE_URL=$(gcloud run services describe store-service \
   --region $LOCATION \
   --format="value(status.address.url)")
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ echo $STORE_SERVICE_URL 
https://store-service-kmr4fqo4aa-ew.a.run.app
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

To test communication between the microservices and generate an order ID, post a message to the **store service**:

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$  curl -X POST -H "Content-Type: application/json" -d @test.json $STORE_SERVICE_URL
{"status":"success","order_id":"q5w7kk6"}student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```



### Store service

The *store service* (public endpoint) uses Pub/Sub to transmit information to the **order service** (private endpoint).

1. In the Google Cloud console **Navigation menu** (![navmenu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Run**.
2. Click the link to the **store-service**.
3. To view the service logs, click **Logs**. Check the store service logs to view the order ID that was generated.
4. Add the log filter `ORDER ID` to see the ID generated by the *store service*.

```json
{
  "textPayload": "ORDER ID: q5w7kk6",
  "insertId": "668d70dd0009c97ecfc33e82",
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "revision_name": "store-service-00001-9nl",
      "configuration_name": "store-service",
      "location": "europe-west1",
      "service_name": "store-service",
      "project_id": "qwiklabs-gcp-01-e4390abfab41"
    }
  },
  "timestamp": "2024-07-09T17:18:21.641406Z",
  "labels": {
    "instanceId": "0087244a806e3c39e1a66ba96040897d763951f2f98ac98001ce68e47befc0ad61d29b24ba4e3779f7a9dbf7f65341685e1ef377f9edcfedc90cdde84aff24"
  },
  "logName": "projects/qwiklabs-gcp-01-e4390abfab41/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-09T17:18:21.925730456Z"
}
```



### Order service

The **order service** receives a message from the **store service** passed with Pub/Sub.

1. Check the order service logs to confirm that the JSON data was successfully transferred.
2. Add the log filter `Order Placed` to see the generated order ID that was passed to the **order service**.

```json
{
  "textPayload": "Order Placed: q5w7kk6",
  "insertId": "668d70e0000870bbf5784cbb",
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "project_id": "qwiklabs-gcp-01-e4390abfab41",
      "location": "europe-west1",
      "configuration_name": "order-service",
      "service_name": "order-service",
      "revision_name": "order-service-00001-964"
    }
  },
  "timestamp": "2024-07-09T17:18:24.553147Z",
  "labels": {
    "instanceId": "0087244a80ee86ecde2e0786b26029577ad88fa7eceb1fc9b69869773e6241f3b73d16ee076ae2ee391b6e4ca891d9dfa35026e9744e95e328472962892b5e091e"
  },
  "logName": "projects/qwiklabs-gcp-01-e4390abfab41/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-09T17:18:24.709505211Z"
}
```



Critter Junction have now updated their solution to take advantage of Pub/Sub. The following high level architecture diagram summaries the solution deployed.

![Architecture diagram](https://cdn.qwiklabs.com/Hcr1aEF3D4QfghnerR6TTSS0djo7UpGyzSsidGSp9vA%3D)

You have successfully deployed Pub/Sub on Google Cloud to asynchronously communicate between Cloud Run services.

## History

```sh
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ history 
    1  gcloud config set project HQGP8EvCRXcm
    2  gcloud config set project qwiklabs-gcp-01-e4390abfab41 
    3  gcloud services enable run.googleapis.com
    4   LOCATION=europe-west1
    5  gcloud config set compute/region $LOCATION
    6   gcloud run deploy store-service   --image gcr.io/qwiklabs-resources/gsp724-store-service   --region $LOCATION   --allow-unauthenticated
    7   gcloud run deploy order-service   --image gcr.io/qwiklabs-resources/gsp724-order-service   --region $LOCATION   --no-allow-unauthenticated
    8  gcloud pubsub topics create ORDER_PLACED
    9   gcloud iam service-accounts create pubsub-cloud-run-invoker   --display-name "Order Initiator"
   10   gcloud iam service-accounts list --filter="Order Initiator"
   11   gcloud run services add-iam-policy-binding order-service --region $LOCATION   --member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com   --role=roles/run.invoker --platform managed
   12   PROJECT_NUMBER=$(gcloud projects list \
  --filter="qwiklabs-gcp" \
  --format='value(PROJECT_NUMBER)')
   13   gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT    --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com    --role=roles/iam.serviceAccountTokenCreator
   14   ORDER_SERVICE_URL=$(gcloud run services describe order-service \
   --region $LOCATION \
   --format="value(status.address.url)")
   15  cat $ORDER_SERVICE_URL 
   16  echo $ORDER_SERVICE_URL 
   17   gcloud pubsub subscriptions create order-service-sub    --topic ORDER_PLACED    --push-endpoint=$ORDER_SERVICE_URL    --push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   18  vi test.json
   19  cat test.json 
   20   STORE_SERVICE_URL=$(gcloud run services describe store-service \
   --region $LOCATION \
   --format="value(status.address.url)")
   21  echo $STORE_SERVICE_URL 
   22   curl -X POST -H "Content-Type: application/json" -d @test.json $STORE_SERVICE_URL
   23  history 
student_01_e488ef31cb8a@cloudshell:~ (qwiklabs-gcp-01-e4390abfab41)$ 
```

