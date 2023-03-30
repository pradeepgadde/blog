---

layout: single
title:  "Google Cloud Pub/Sub"
date:   2023-03-29 13:59:04 +0530
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

# Google Cloud Pub/Sub

Google Cloud Pub/Sub is a messaging service for exchanging event data  among applications and services. A producer of data publishes messages  to a Cloud Pub/Sub topic. A consumer creates a subscription to that  topic. Subscribers either pull messages from a subscription or are  configured as webhooks for push subscriptions. Every subscriber must  acknowledge each message within a configurable window of time.



- Set up a topic to hold data.

  To use a Pub/Sub, you create a topic to hold data and a subscription to access data published to the topic.

- Subscribe to a topic to access the data.

  make a subscription to access the topic.

- Publish and then consume messages with a pull subscriber.

  Publish a message to the topic 
  
  View the Message





- Learn the basics of Pub/Sub.
- Create, delete, and list Pub/Sub topics.
- Create, delete, and list Pub/Sub subscriptions.
- Publish messages to a topic.
- Use a pull subscriber to output individual topic messages.
- Use a pull subscriber with a flag to output multiple messages.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-209b0aba3c08.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics create myTopic
Created topic [projects/qwiklabs-gcp-00-209b0aba3c08/topics/myTopic].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```





```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics create Test1
Created topic [projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test1].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics create Test2
Created topic [projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test2].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics list
---
messageStoragePolicy:
  allowedPersistenceRegions:
  - asia-east1
  - asia-northeast1
  - asia-south1
  - asia-southeast1
  - australia-southeast1
  - europe-central2
  - europe-north1
  - europe-west1
  - europe-west12
  - europe-west2
  - europe-west3
  - europe-west4
  - europe-west5
  - me-central1
  - me-west1
  - southamerica-west1
  - us-central1
  - us-central2
  - us-east1
  - us-east4
  - us-east5
  - us-east7
  - us-south1
  - us-west1
  - us-west2
  - us-west3
  - us-west4
name: projects/qwiklabs-gcp-00-209b0aba3c08/topics/myTopic
---
messageStoragePolicy:
  allowedPersistenceRegions:
  - asia-east1
  - asia-northeast1
  - asia-south1
  - asia-southeast1
  - australia-southeast1
  - europe-central2
  - europe-north1
  - europe-west1
  - europe-west12
  - europe-west2
  - europe-west3
  - europe-west4
  - europe-west5
  - me-central1
  - me-west1
  - southamerica-west1
  - us-central1
  - us-central2
  - us-east1
  - us-east4
  - us-east5
  - us-east7
  - us-south1
  - us-west1
  - us-west2
  - us-west3
  - us-west4
name: projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test2
---
messageStoragePolicy:
  allowedPersistenceRegions:
  - asia-east1
  - asia-northeast1
  - asia-south1
  - asia-southeast1
  - australia-southeast1
  - europe-central2
  - europe-north1
  - europe-west1
  - europe-west12
  - europe-west2
  - europe-west3
  - europe-west4
  - europe-west5
  - me-central1
  - me-west1
  - southamerica-west1
  - us-central1
  - us-central2
  - us-east1
  - us-east4
  - us-east5
  - us-east7
  - us-south1
  - us-west1
  - us-west2
  - us-west3
  - us-west4
name: projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test1
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics delete Test1
Deleted topic [projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test1].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics delete Test2
Deleted topic [projects/qwiklabs-gcp-00-209b0aba3c08/topics/Test2].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics list
---
messageStoragePolicy:
  allowedPersistenceRegions:
  - asia-east1
  - asia-northeast1
  - asia-south1
  - asia-southeast1
  - australia-southeast1
  - europe-central2
  - europe-north1
  - europe-west1
  - europe-west12
  - europe-west2
  - europe-west3
  - europe-west4
  - europe-west5
  - me-central1
  - me-west1
  - southamerica-west1
  - us-central1
  - us-central2
  - us-east1
  - us-east4
  - us-east5
  - us-east7
  - us-south1
  - us-west1
  - us-west2
  - us-west3
  - us-west4
name: projects/qwiklabs-gcp-00-209b0aba3c08/topics/myTopic
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions create --topic myTopic mySubscription
Created subscription [projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/mySubscription].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions create --topic myTopic Test1
Created subscription [projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test1].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions create --topic myTopic Test2
Created subscription [projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test2].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics list-subscriptions myTopic
---
  projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test1
---
  projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test2
---
  projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/mySubscription
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions delete Test1
Deleted subscription [projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test1].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions delete Test2
Deleted subscription [projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/Test2].
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics list-subscriptions myTopic
---
  projects/qwiklabs-gcp-00-209b0aba3c08/subscriptions/mySubscription
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Hello"
messageIds:
- '7352430483680377'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher's Name is Pradeep!"
messageIds:
- '7352446290325989'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher likes to eat Vegetarian food!"
messageIds:
- '7352495578003472'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is Awesome."
messageIds:
- '7352460159538059'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
Listed 0 items.
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
DATA: Hello
MESSAGE_ID: 7352430483680377
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```




> **Using the pull command without any flags will output only one message, even if you are subscribed to a topic that has more held in  it.**
> **Once an individual message has been outputted from a  particular subscription-based pull command, you cannot access that  message again with the pull command.**



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
messageIds:
- '7352478530808654'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
DATA: Publisher's Name is Pradeep!
MESSAGE_ID: 7352446290325989
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
DATA: Publisher likes to eat Vegetarian food!
MESSAGE_ID: 7352495578003472
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
DATA: Publisher thinks Pub/Sub is Awesome.
MESSAGE_ID: 7352460159538059
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
DATA: Publisher is starting to get the hang of Pub/Sub
MESSAGE_ID: 7352478530808654
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack
Listed 0 items.
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```



```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
messageIds:
- '7352494978076992'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
messageIds:
- '7352430733597767'
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```

```sh
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
DATA: Publisher wonders if all messages will be pulled
MESSAGE_ID: 7352494978076992
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$ gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
DATA: Publisher will have to test to find out
MESSAGE_ID: 7352430733597767
ORDERING_KEY:
ATTRIBUTES:
DELIVERY_ATTEMPT:
ACK_STATUS: SUCCESS
student_01_7006df727a46@cloudshell:~ (qwiklabs-gcp-00-209b0aba3c08)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-218.png)

In this lab, we created a Pub/Sub topic, published to the topic, created a  subscription, then used the subscription to pull data from the topic.

