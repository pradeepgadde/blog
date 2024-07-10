---

layout: single
title:  "Eventarc for Cloud Run"
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
# Eventarc for Cloud Run

In this lab, you will learn how to utilize events for Cloud Run to manage communication between producer and consumers. Producers (i.e. Event Sources) provide the originating data. The data produced is sent to a Consumer (i.e. Event Sinks) that use the information passed.

The unifying delivery mechanism between producers and consumers is Eventarc for Cloud Run. In the above example, Cloud Pub/Sub facilitates event delivery for their project events generated.

At the end of this lab, you will be able to deliver events from various sources to Google Cloud sinks and Custom sinks.

What you'll learn:

- Eventarc for Cloud Run
- Create a Cloud Run sink
- Create an Event trigger for Cloud Pub/Sub
- Create an Event trigger for Audit Logs

## Set up your environment

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-13287e2e8172.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud config set project qwiklabs-gcp-03-13287e2e8172
Updated property [core/project].
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud config set run/region us-east4
Updated property [run/region].
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud config set run/platform managed
Updated property [run/platform].
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud config set eventarc/location us-east4
Updated property [eventarc/location].
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



## Enable service account

Next, configure a couple of service accounts needed for the Audit Log trigger.

Store the Project Number in an environment variable

Grant the `eventarc.admin` role to the default Compute Engine service account

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ export PROJECT_NUMBER="$(gcloud projects list \
  --filter=$(gcloud config get-value project) \
  --format='value(PROJECT_NUMBER)')"
Your active configuration is: [cloudshell-10653]
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
  --member=serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com \
  --role='roles/eventarc.admin'
Your active configuration is: [cloudshell-10653]
Updated IAM policy for project [qwiklabs-gcp-03-13287e2e8172].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-13287e2e8172@qwiklabs-gcp-03-13287e2e8172.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:653530056673@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-653530056673@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-653530056673@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-653530056673@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:653530056673-compute@developer.gserviceaccount.com
  - serviceAccount:653530056673@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:653530056673-compute@developer.gserviceaccount.com
  role: roles/eventarc.admin
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-13287e2e8172@qwiklabs-gcp-03-13287e2e8172.iam.gserviceaccount.com
  - user:student-03-e82a56afaf2e@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:service-653530056673@serverless-robot-prod.iam.gserviceaccount.com
  role: roles/run.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-03-13287e2e8172@qwiklabs-gcp-03-13287e2e8172.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-03-e82a56afaf2e@qwiklabs.net
  role: roles/viewer
etag: BwYc4x91bcw=
version: 1
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



## Event discovery

Registered sources and the types of events can be discovered using the command line.

1. To see the list of different types of events, run the following:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc providers list
NAME: vmmigration.googleapis.com
LOCATION: us-east4

NAME: cloudaudit.googleapis.com
LOCATION: us-east4

NAME: pubsub.googleapis.com
LOCATION: us-east4

NAME: metastore.googleapis.com
LOCATION: us-east4

NAME: datafusion.googleapis.com
LOCATION: us-east4

NAME: workflows.googleapis.com
LOCATION: us-east4

NAME: apigateway.googleapis.com
LOCATION: us-east4

NAME: gkebackup.googleapis.com
LOCATION: us-east4

NAME: datamigration.googleapis.com
LOCATION: us-east4

NAME: transcoder.googleapis.com
LOCATION: us-east4

NAME: memcache.googleapis.com
LOCATION: us-east4

NAME: apigeeregistry.googleapis.com
LOCATION: us-east4

NAME: dataplex.googleapis.com
LOCATION: us-east4

NAME: batch.googleapis.com
LOCATION: us-east4

NAME: datastream.googleapis.com
LOCATION: us-east4

NAME: alloydb.googleapis.com
LOCATION: us-east4

NAME: networkservices.googleapis.com
LOCATION: us-east4

NAME: eventarc.googleapis.com
LOCATION: us-east4

NAME: clouddeploy.googleapis.com
LOCATION: us-east4

NAME: cloudfunctions.googleapis.com
LOCATION: us-east4

NAME: firestore.googleapis.com
LOCATION: us-east4

NAME: storage.googleapis.com
LOCATION: us-east4

NAME: servicedirectory.googleapis.com
LOCATION: us-east4

NAME: networkconnectivity.googleapis.com
LOCATION: us-east4

NAME: gkehub.googleapis.com
LOCATION: us-east4

NAME: dataflow.googleapis.com
LOCATION: us-east4

NAME: redis.googleapis.com
LOCATION: us-east4
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc providers describe \
  pubsub.googleapis.com
displayName: Cloud Pub/Sub
eventTypes:
- description: A message is published to the specified Pub/Sub topic.
  filteringAttributes:
  - attribute: type
    required: true
  type: google.cloud.pubsub.topic.v1.messagePublished
name: projects/qwiklabs-gcp-03-13287e2e8172/locations/us-east4/providers/pubsub.googleapis.com
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



## Create a Cloud Run sink

1. Set up an environment variable for the service:
2. Set up an environment variable for the image:
3. Deploy your containerized application to Cloud Run:

On successful deployment, the command line displays the service URL. At this point the service is up and running.

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ export SERVICE_NAME=event-display
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ export IMAGE_NAME="gcr.io/cloudrun/hello"
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud run deploy ${SERVICE_NAME} \
  --image ${IMAGE_NAME} \
  --allow-unauthenticated \
  --max-instances=3
Deploying container to Cloud Run service [event-display] in project [qwiklabs-gcp-03-13287e2e8172] region [us-east4]
OK Deploying new service... Done.                                                                                                                                                  
  OK Creating Revision...                                                                                                                                                          
  OK Routing traffic...                                                                                                                                                            
  OK Setting IAM Policy...                                                                                                                                                         
Done.                                                                                                                                                                              
Service [event-display] revision [event-display-00001-ls7] has been deployed and is serving 100 percent of traffic.
Service URL: https://event-display-jbguyq5g3q-uk.a.run.app
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



You can now visit your deployed container by opening the service URL in any browser window.

## Create a Cloud Pub/Sub event trigger

One way of receiving events is through Cloud Pub/Sub. Custom  applications can publish messages to Cloud Pub/Sub and these messages  can be delivered to Google Cloud Run sinks via Eventarc for Cloud Run.

### **Create a trigger**

1. First, get more details on the parameters you'll need to construct a trigger for events from Cloud Pub/Sub:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc providers describe \
  pubsub.googleapis.com
displayName: Cloud Pub/Sub
eventTypes:
- description: A message is published to the specified Pub/Sub topic.
  filteringAttributes:
  - attribute: type
    required: true
  type: google.cloud.pubsub.topic.v1.messagePublished
name: projects/qwiklabs-gcp-03-13287e2e8172/locations/us-east4/providers/pubsub.googleapis.com
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

2. Create a trigger to filter events published to the Cloud Pub/Sub topic to your deployed Cloud Run service:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers create trigger-pubsub \
  --destination-run-service=${SERVICE_NAME} \
  --event-filters="type=google.cloud.pubsub.topic.v1.messagePublished"
Creating trigger [trigger-pubsub] in project [qwiklabs-gcp-03-13287e2e8172], location [us-east4]...done.                                                                           
Created Pub/Sub topic [projects/qwiklabs-gcp-03-13287e2e8172/topics/eventarc-us-east4-trigger-pubsub-137].
Publish to this topic to receive events in Cloud Run service [event-display].
WARNING: It may take up to 2 minutes for the new trigger to become active.
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

### **Find the topic**

The Pub/Sub trigger creates a Pub/Sub topic behind the scenes. Find it, and assign it to an environment variable:

Export the TOPIC ID

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ export TOPIC_ID=$(gcloud eventarc triggers describe trigger-pubsub \
  --format='value(transport.pubsub.topic)')
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ echo ${TOPIC_ID}
projects/qwiklabs-gcp-03-13287e2e8172/topics/eventarc-us-east4-trigger-pubsub-137
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

### **Test the trigger**

You can check that the trigger is created by listing all triggers:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers list
NAME: trigger-pubsub
TYPE: google.cloud.pubsub.topic.v1.messagePublished
DESTINATION: Cloud Run service: event-display
ACTIVE: Yes
LOCATION: us-east4
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

In order to simulate a custom application sending message, you can use a `gcloud` command to to fire an event:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud pubsub topics publish ${TOPIC_ID} --message="Hello there"
messageIds:
- '10228977938220025'
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

1. In the **Navigation menu** > **Serverless** > **Cloud Run**, click on **event display**.
2. Click on **Logs**.

The Cloud Run sink you created logs the body of the incoming message. You can view this in the Logs section of your Cloud Run instance:

```json
{
  "insertId": "668e7586000b05d178a1abc8",
  "jsonPayload": {
    "message": "Received event of type google.cloud.pubsub.topic.v1.messagePublished. Event data: Hello there",
    "eventType": "google.cloud.pubsub.topic.v1.messagePublished",
    "event": {
      "id": "10228977938220025",
      "type": "google.cloud.pubsub.topic.v1.messagePublished",
      "source": "//pubsub.googleapis.com/projects/qwiklabs-gcp-03-13287e2e8172/topics/eventarc-us-east4-trigger-pubsub-137",
      "datacontenttype": "application/json",
      "time": "2024-07-10T11:50:29.182Z",
      "data": {
        "message": {
          "data": "SGVsbG8gdGhlcmU=",
          "publish_time": "2024-07-10T11:50:29.182Z",
          "messageId": "10228977938220025",
          "publishTime": "2024-07-10T11:50:29.182Z",
          "message_id": "10228977938220025"
        },
        "subscription": "projects/qwiklabs-gcp-03-13287e2e8172/subscriptions/eventarc-us-east4-trigger-pubsub-sub-615"
      },
      "specversion": "1.0"
    }
  },
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "project_id": "qwiklabs-gcp-03-13287e2e8172",
      "service_name": "event-display",
      "configuration_name": "event-display",
      "revision_name": "event-display-00001-ls7",
      "location": "us-east4"
    }
  },
  "timestamp": "2024-07-10T11:50:30.722385Z",
  "severity": "INFO",
  "labels": {
    "instanceId": "0087244a806e36d0ccbdbc292a55e373de6061afbde0f75c54aac3214a0091387ee792fcc73880988c714b2895c2b09f9cc1445875a492401e909e36a4a181"
  },
  "logName": "projects/qwiklabs-gcp-03-13287e2e8172/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-10T11:50:30.727612232Z"
}
```

### **Delete the trigger**

You can delete the trigger once you're done testing

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers delete trigger-pubsub
Deleting trigger [trigger-pubsub] in project [qwiklabs-gcp-03-13287e2e8172], location [us-east4]...done.                                                                           
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



## Create a Audit Logs event trigger

Next, set up a trigger to listen for events from Audit Logs. You will listen for Cloud Storage events in Audit Logs.

### **Create a bucket**

1. Create an environment variable for your bucket:
2. Create a Cloud Storage bucket in the same region as the deployed Cloud Run service:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ export BUCKET_NAME=$(gcloud config get-value project)-cr-bucket
Your active configuration is: [cloudshell-10653]
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gsutil mb -p $(gcloud config get-value project) \
  -l $(gcloud config get-value run/region) \
  gs://${BUCKET_NAME}/
Your active configuration is: [cloudshell-10653]
Your active configuration is: [cloudshell-10653]
Creating gs://qwiklabs-gcp-03-13287e2e8172-cr-bucket/...
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



### **Enable Audit Logs**

In order to receive events from a service, you need to enable audit logs.

1. From the **Navigation menu**, select **IAM & Admin**  > **Audit Logs**.
2. In the list of services, check the box for `Google Cloud Storage`.
3. On the right hand side, click the **LOG TYPE** tab. **Admin Write** is selected by default, make sure you also select **Admin Read**, **Data Read**, **Data Write**   and then click **Save**.

### **Test audit logs**

To learn how to identify the parameters you'll need to set up an actual trigger, and perform an actual operation.

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ echo "Hello World" > random.txt
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gsutil cp random.txt gs://${BUCKET_NAME}/random.txt
Copying file://random.txt [Content-Type=text/plain]...
- [1 files][   12.0 B/   12.0 B]                                                
Operation completed over 1 objects/12.0 B.                                       
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```



Now, see what kind of audit log this update generated.

1. In the Cloud Console, go to **Navigation menu** > **Logging** > **Logs Explorer**.
2. Under **Resource**, choose **GCS Bucket > [Bucket Name] > Location** then choose your bucket and its location. Click **Apply**.

```sh
resource.type="gcs_bucket"
resource.labels.bucket_name="qwiklabs-gcp-03-13287e2e8172-cr-bucket"
resource.labels.location="us-east4"
```

Once you run the query, you'll see logs for the storage bucket. One of those should be `storage.buckets.create`.

Note the `serviceName`, `methodName` and `resourceName`. You will use these in creating the trigger.

```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "status": {},
    "authenticationInfo": {
      "principalEmail": "student-03-e82a56afaf2e@qwiklabs.net"
    },
    "requestMetadata": {
      "callerIp": "34.143.200.31",
      "callerSuppliedUserAgent": "apitools Python/3.11.8 gsutil/5.30 (linux) analytics/enabled interactive/True command/mb google-cloud-sdk/483.0.0,gzip(gfe)",
      "requestAttributes": {
        "time": "2024-07-10T11:54:01.107028299Z",
        "auth": {}
      },
      "destinationAttributes": {}
    },
    "serviceName": "storage.googleapis.com",
    "methodName": "storage.buckets.create",
    "authorizationInfo": [
      {
        "resource": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket",
        "permission": "storage.buckets.create",
        "granted": true,
        "resourceAttributes": {}
      }
    ],
    "resourceName": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket",
    "serviceData": {
      "@type": "type.googleapis.com/google.iam.v1.logging.AuditData",
      "policyDelta": {
        "bindingDeltas": [
          {
            "action": "ADD",
            "role": "roles/storage.legacyBucketOwner",
            "member": "projectEditor:qwiklabs-gcp-03-13287e2e8172"
          },
          {
            "action": "ADD",
            "role": "roles/storage.legacyBucketOwner",
            "member": "projectOwner:qwiklabs-gcp-03-13287e2e8172"
          },
          {
            "action": "ADD",
            "role": "roles/storage.legacyBucketReader",
            "member": "projectViewer:qwiklabs-gcp-03-13287e2e8172"
          }
        ]
      }
    },
    "request": {
      "defaultObjectAcl": {
        "bindings": [
          {
            "members": [
              "projectViewer:qwiklabs-gcp-03-13287e2e8172"
            ],
            "role": "roles/storage.legacyObjectReader"
          },
          {
            "members": [
              "projectOwner:qwiklabs-gcp-03-13287e2e8172",
              "projectEditor:qwiklabs-gcp-03-13287e2e8172"
            ],
            "role": "roles/storage.legacyObjectOwner"
          }
        ],
        "@type": "type.googleapis.com/google.iam.v1.Policy"
      }
    },
    "resourceLocation": {
      "currentLocations": [
        "us-east4"
      ]
    }
  },
  "insertId": "8buas7e2ed25",
  "resource": {
    "type": "gcs_bucket",
    "labels": {
      "project_id": "qwiklabs-gcp-03-13287e2e8172",
      "location": "us-east4",
      "bucket_name": "qwiklabs-gcp-03-13287e2e8172-cr-bucket"
    }
  },
  "timestamp": "2024-07-10T11:54:01.098466250Z",
  "severity": "NOTICE",
  "logName": "projects/qwiklabs-gcp-03-13287e2e8172/logs/cloudaudit.googleapis.com%2Factivity",
  "receiveTimestamp": "2024-07-10T11:54:02.426292955Z"
}
```

### **Create a trigger**

You are now ready to create an event trigger for Audit Logs.

1. Get more details on the parameters you'll need to construct the trigger:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc providers describe cloudaudit.googleapis.com
displayName: Cloud Audit Logs
eventTypes:
- description: An audit log is created that matches the trigger's filter criteria.
  filteringAttributes:
  - attribute: methodName
    description: The identifier of the service's operation.
    required: true
  - attribute: resourceName
    description: The complete path to a resource. Used to filter events for a specific
      resource.
    pathPatternSupported: true
  - attribute: serviceName
    description: The identifier of the Google Cloud service.
    required: true
  - attribute: type
    required: true
  type: google.cloud.audit.log.v1.written
name: projects/qwiklabs-gcp-03-13287e2e8172/locations/us-east4/providers/cloudaudit.googleapis.com
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

Create the trigger with the right filters:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers create trigger-auditlog \
  --destination-run-service=${SERVICE_NAME} \
  --event-filters="type=google.cloud.audit.log.v1.written" \
  --event-filters="serviceName=storage.googleapis.com" \
  --event-filters="methodName=storage.objects.create" \
  --service-account=${PROJECT_NUMBER}-compute@developer.gserviceaccount.com
Creating trigger [trigger-auditlog] in project [qwiklabs-gcp-03-13287e2e8172], location [us-east4]...done.                                  
WARNING: It may take up to 2 minutes for the new trigger to become active.
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

### **Test the trigger**

1. List all triggers to confirm that the trigger was successfully created:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers list
NAME: trigger-auditlog
TYPE: google.cloud.audit.log.v1.written
DESTINATION: Cloud Run service: event-display
ACTIVE: By 12:03:50
LOCATION: us-east4
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

Wait for up to 10 minutes for the trigger creation to be propagated and for it to begin filtering events.

Once ready, it will filter create events and send them to the service.  You're now ready to fire an event.

Upload the same file to the Cloud Storage bucket as you did earlier:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gsutil cp random.txt gs://${BUCKET_NAME}/random.txt
Copying file://random.txt [Content-Type=text/plain]...
- [1 files][   12.0 B/   12.0 B]                                                
Operation completed over 1 objects/12.0 B.                                       
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

Navigate to **Navigation menu** > **Cloud Run** to check the logs of the Cloud Run service, you should see the received event.

```json
{
  "insertId": "668e7891000ec2597f01cb3f",
  "jsonPayload": {
    "event": {
      "data": {
        "severity": "INFO",
        "timestamp": "2024-07-10T12:03:23.801000699Z",
        "insertId": "1hu3wp7e1gx26",
        "@type": "type.googleapis.com/google.events.cloud.audit.v1.LogEntryData",
        "receiveTimestamp": "2024-07-10T12:03:24.825317964Z",
        "resource": {
          "type": "gcs_bucket",
          "labels": {
            "project_id": "qwiklabs-gcp-03-13287e2e8172",
            "location": "us-east4",
            "bucket_name": "qwiklabs-gcp-03-13287e2e8172-cr-bucket"
          }
        },
        "logName": "projects/qwiklabs-gcp-03-13287e2e8172/logs/cloudaudit.googleapis.com%2Fdata_access",
        "protoPayload": {
          "status": {},
          "resourceName": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt",
          "serviceName": "storage.googleapis.com",
          "resourceLocation": {
            "currentLocations": [
              "us-east4"
            ]
          },
          "authenticationInfo": {
            "principalEmail": "student-03-e82a56afaf2e@qwiklabs.net"
          },
          "authorizationInfo": [
            {
              "permission": "storage.objects.delete",
              "resource": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt",
              "resourceAttributes": {},
              "granted": true
            },
            {
              "resource": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt",
              "granted": true,
              "resourceAttributes": {},
              "permission": "storage.objects.create"
            }
          ],
          "requestMetadata": {
            "destinationAttributes": {},
            "callerSuppliedUserAgent": "apitools Python/3.11.8 gsutil/5.30 (linux) analytics/enabled interactive/True command/cp google-cloud-sdk/483.0.0,gzip(gfe)",
            "callerIp": "34.143.200.31",
            "requestAttributes": {
              "time": "2024-07-10T12:03:23.807743748Z",
              "auth": {}
            }
          },
          "methodName": "storage.objects.create",
          "serviceData": {
            "policyDelta": {
              "bindingDeltas": [
                {
                  "action": "ADD",
                  "role": "roles/storage.legacyObjectOwner",
                  "member": "user:student-03-e82a56afaf2e@qwiklabs.net"
                },
                {
                  "member": "projectViewer:qwiklabs-gcp-03-13287e2e8172",
                  "action": "ADD",
                  "role": "roles/storage.legacyObjectReader"
                },
                {
                  "role": "roles/storage.legacyObjectOwner",
                  "action": "ADD",
                  "member": "projectEditor:qwiklabs-gcp-03-13287e2e8172"
                },
                {
                  "role": "roles/storage.legacyObjectOwner",
                  "action": "ADD",
                  "member": "projectOwner:qwiklabs-gcp-03-13287e2e8172"
                }
              ]
            },
            "@type": "type.googleapis.com/google.iam.v1.logging.AuditData"
          }
        }
      },
      "source": "//cloudaudit.googleapis.com/projects/qwiklabs-gcp-03-13287e2e8172/logs/data_access",
      "specversion": "1.0",
      "time": "2024-07-10T12:03:24.825317964Z",
      "datacontenttype": "application/json; charset=utf-8",
      "recordedtime": "2024-07-10T12:03:23.801000699Z",
      "type": "google.cloud.audit.log.v1.written",
      "servicename": "storage.googleapis.com",
      "dataschema": "https://googleapis.github.io/google-cloudevents/jsonschema/google/events/cloud/audit/v1/LogEntryData.json",
      "id": "projects/qwiklabs-gcp-03-13287e2e8172/logs/cloudaudit.googleapis.com%2Fdata_access1hu3wp7e1gx261720613003801000",
      "resourcename": "projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt",
      "methodname": "storage.objects.create",
      "subject": "storage.googleapis.com/projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt"
    },
    "eventType": "google.cloud.audit.log.v1.written",
    "message": "Received event of type google.cloud.audit.log.v1.written. Event data: {\"@type\":\"type.googleapis.com/google.events.cloud.audit.v1.LogEntryData\",\"protoPayload\":{\"status\":{},\"authenticationInfo\":{\"principalEmail\":\"student-03-e82a56afaf2e@qwiklabs.net\"},\"requestMetadata\":{\"callerIp\":\"34.143.200.31\",\"callerSuppliedUserAgent\":\"apitools Python/3.11.8 gsutil/5.30 (linux) analytics/enabled interactive/True command/cp google-cloud-sdk/483.0.0,gzip(gfe)\",\"requestAttributes\":{\"time\":\"2024-07-10T12:03:23.807743748Z\",\"auth\":{}},\"destinationAttributes\":{}},\"serviceName\":\"storage.googleapis.com\",\"methodName\":\"storage.objects.create\",\"authorizationInfo\":[{\"resource\":\"projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt\",\"permission\":\"storage.objects.delete\",\"granted\":true,\"resourceAttributes\":{}},{\"resource\":\"projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt\",\"permission\":\"storage.objects.create\",\"granted\":true,\"resourceAttributes\":{}}],\"resourceName\":\"projects/_/buckets/qwiklabs-gcp-03-13287e2e8172-cr-bucket/objects/random.txt\",\"serviceData\":{\"policyDelta\":{\"bindingDeltas\":[{\"action\":\"ADD\",\"role\":\"roles/storage.legacyObjectOwner\",\"member\":\"user:student-03-e82a56afaf2e@qwiklabs.net\"},{\"member\":\"projectViewer:qwiklabs-gcp-03-13287e2e8172\",\"action\":\"ADD\",\"role\":\"roles/storage.legacyObjectReader\"},{\"action\":\"ADD\",\"role\":\"roles/storage.legacyObjectOwner\",\"member\":\"projectEditor:qwiklabs-gcp-03-13287e2e8172\"},{\"role\":\"roles/storage.legacyObjectOwner\",\"member\":\"projectOwner:qwiklabs-gcp-03-13287e2e8172\",\"action\":\"ADD\"}]},\"@type\":\"type.googleapis.com/google.iam.v1.logging.AuditData\"},\"resourceLocation\":{\"currentLocations\":[\"us-east4\"]}},\"insertId\":\"1hu3wp7e1gx26\",\"resource\":{\"type\":\"gcs_bucket\",\"labels\":{\"project_id\":\"qwiklabs-gcp-03-13287e2e8172\",\"bucket_name\":\"qwiklabs-gcp-03-13287e2e8172-cr-bucket\",\"location\":\"us-east4\"}},\"timestamp\":\"2024-07-10T12:03:23.801000699Z\",\"severity\":\"INFO\",\"logName\":\"projects/qwiklabs-gcp-03-13287e2e8172/logs/cloudaudit.googleapis.com%2Fdata_access\",\"receiveTimestamp\":\"2024-07-10T12:03:24.825317964Z\"}"
  },
  "resource": {
    "type": "cloud_run_revision",
    "labels": {
      "configuration_name": "event-display",
      "service_name": "event-display",
      "project_id": "qwiklabs-gcp-03-13287e2e8172",
      "revision_name": "event-display-00001-ls7",
      "location": "us-east4"
    }
  },
  "timestamp": "2024-07-10T12:03:29.967257Z",
  "severity": "INFO",
  "labels": {
    "instanceId": "0087244a8052b61d4eda7e4faa996765260bda62ffb31c44a6e1137008466f955b97a9221f147304d3e22d1cf980518cde9f5279a7e97539c5660275a772ff"
  },
  "logName": "projects/qwiklabs-gcp-03-13287e2e8172/logs/run.googleapis.com%2Fstdout",
  "receiveTimestamp": "2024-07-10T12:03:29.975001826Z"
}
```

### **Delete the trigger**

You can delete the trigger once done testing:

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ gcloud eventarc triggers delete trigger-auditlog
Deleting trigger [trigger-auditlog] in project [qwiklabs-gcp-03-13287e2e8172], location [us-east4]...done.                                                                         
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

## History

```sh
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ history 
    1  gcloud config set project qwiklabs-gcp-03-13287e2e8172
    2  gcloud config set run/region us-east4
    3  gcloud config set run/platform managed
    4  gcloud config set eventarc/location us-east4
    5  export PROJECT_NUMBER="$(gcloud projects list \
  --filter=$(gcloud config get-value project) \
  --format='value(PROJECT_NUMBER)')"
    6  gcloud projects add-iam-policy-binding $(gcloud config get-value project)   --member=serviceAccount:${PROJECT_NUMBER}-compute@developer.gserviceaccount.com   --role='roles/eventarc.admin'
    7  gcloud eventarc providers list
    8  gcloud eventarc providers describe   pubsub.googleapis.com
    9  export SERVICE_NAME=event-display
   10  export IMAGE_NAME="gcr.io/cloudrun/hello"
   11  gcloud run deploy ${SERVICE_NAME}   --image ${IMAGE_NAME}   --allow-unauthenticated   --max-instances=3
   12  gcloud eventarc providers describe   pubsub.googleapis.com
   13  gcloud eventarc triggers create trigger-pubsub   --destination-run-service=${SERVICE_NAME}   --event-filters="type=google.cloud.pubsub.topic.v1.messagePublished"
   14  export TOPIC_ID=$(gcloud eventarc triggers describe trigger-pubsub \
  --format='value(transport.pubsub.topic)')
   15  echo ${TOPIC_ID}
   16  gcloud eventarc triggers list
   17  gcloud pubsub topics publish ${TOPIC_ID} --message="Hello there"
   18  gcloud eventarc triggers delete trigger-pubsub
   19  export BUCKET_NAME=$(gcloud config get-value project)-cr-bucket
   20  gsutil mb -p $(gcloud config get-value project)   -l $(gcloud config get-value run/region)   gs://${BUCKET_NAME}/
   21  echo "Hello World" > random.txt
   22  gsutil cp random.txt gs://${BUCKET_NAME}/random.txt
   23  gcloud eventarc providers describe cloudaudit.googleapis.com
   24  gcloud eventarc triggers create trigger-auditlog   --destination-run-service=${SERVICE_NAME}   --event-filters="type=google.cloud.audit.log.v1.written"   --event-filters="serviceName=storage.googleapis.com"   --event-filters="methodName=storage.objects.create"   --service-account=${PROJECT_NUMBER}-compute@developer.gserviceaccount.com
   25  gcloud eventarc triggers list
   26  gsutil cp random.txt gs://${BUCKET_NAME}/random.txt
   27  gcloud eventarc triggers delete trigger-auditlog
   28  history 
student_03_e82a56afaf2e@cloudshell:~ (qwiklabs-gcp-03-13287e2e8172)$ 
```

You have successfully learned about Events for Cloud Run on Google  Cloud infrastructure. Over the course of this lab, you have performed  the following tasks:

- Events for Cloud Run
- Create a Cloud Run sink
- Create an Event trigger for Cloud Pub/Sub
- Create an Event trigger for Audit Logs
