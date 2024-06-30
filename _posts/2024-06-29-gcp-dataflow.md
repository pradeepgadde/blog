---

layout: single
title:  "Dataflow: Qwik Start - Templates"
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
# Dataflow: Qwik Start - Templates

In this lab, you learn how to create a streaming pipeline using one of [Google's Dataflow templates](https://cloud.google.com/dataflow/docs/templates/provided-templates). More specifically, you use the Pub/Sub to BigQuery template, which  reads messages written in JSON from a Pub/Sub topic and pushes them to a BigQuery table. You can find the documentation for this template in the [Get started with Google-provided templates Guide](https://cloud.google.com/dataflow/docs/templates/provided-templates#cloudpubsubtobigquery).

- Create a BigQuery dataset and table
- Create a Cloud Storage bucket
- Create a streaming pipeline using the Pub/Sub to BigQuery Dataflow template

## Ensure that the Dataflow API is successfully re-enabled

To ensure access to the necessary API, restart the connection to the Dataflow API.

1. In the Cloud Console, enter "Dataflow API" in the top search bar. Click on the result for **Dataflow API**.
2. Click **Manage**.
3. Click **Disable API**.

If asked to confirm, click **Disable**.

1. Click **Enable**.

When the API has been enabled again, the page will show the option to disable.



## Create a BigQuery dataset, BigQuery table, and Cloud Storage bucket using Cloud Shell

Let's first create a BigQuery dataset and table.

Run the following command to create a dataset called `taxirides`

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-3029984c9df7.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ bq mk taxirides
Dataset 'qwiklabs-gcp-02-3029984c9df7:taxirides' successfully created.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$
```

Now that you have your dataset created, you'll use it in the following step to instantiate a BigQuery table.

1. Run the following command to do so:

```sh

student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t taxirides.realtime
Table 'qwiklabs-gcp-02-3029984c9df7:taxirides.realtime' successfully created.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ 
```



On its face, the `bq mk` command looks a bit complicated. However, with some assistance from the [BigQuery command-line documentation](https://cloud.google.com/bigquery/docs/reference/bq-cli-reference), we can break down what's going on here. For example, the documentation tells us a little bit more about **schema**:

- Either the path to a local JSON schema file or a comma-separated list of column definitions in the form `[FIELD]`:`[DATA_TYPE]`, `[FIELD]`:`[DATA_TYPE]`.

In this case, we are using the latter—a comma-separated list.

### Create a Cloud Storage bucket using Cloud Shell

Now that we have our table instantiated, let's create a bucket.

Use the Project ID as the bucket name to ensure a globally unique name: 

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ export BUCKET_NAME=qwiklabs-gcp-02-3029984c9df7
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ gsutil mb gs://$BUCKET_NAME/
Creating gs://qwiklabs-gcp-02-3029984c9df7/...
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ 
```



## Run the pipeline

Deploy the Dataflow Template:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ gcloud dataflow jobs run iotflow \
    --gcs-location gs://dataflow-templates-us-west1/latest/PubSub_to_BigQuery \
    --region us-west1 \
    --worker-machine-type e2-medium \
    --staging-location gs://qwiklabs-gcp-02-3029984c9df7/temp \
    --parameters inputTopic=projects/pubsub-public-data/topics/taxirides-realtime,outputTableSpec=qwiklabs-gcp-02-3029984c9df7:taxirides.realtime
createTime: '2024-06-30T09:58:08.882178Z'
currentStateTime: '1970-01-01T00:00:00Z'
id: 2024-06-30_02_58_07-9097816481133158380
location: us-west1
name: iotflow
projectId: qwiklabs-gcp-02-3029984c9df7
startTime: '2024-06-30T09:58:08.882178Z'
type: JOB_TYPE_STREAMING
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ 
```

In the **Google Cloud Console**, on the **Navigation menu**, click **Dataflow > Jobs**, and you will see your dataflow job.

Please refer the [document](https://cloud.google.com/sdk/gcloud/reference/dataflow/jobs/run) for more information.

 You'll watch your resources build and become ready for use.

Now, let's go view the data written to BigQuery by clicking on **BigQuery** found in the Navigation menu.

- When the BigQuery UI opens, you'll see the **taxirides** dataset added under your project name and **realtime** table underneath that.



## Submit a query

You can submit queries using standard SQL.

1. In the BigQuery **Editor**, add the following to query the data in your project:

```sh
SELECT * FROM `qwiklabs-gcp-02-3029984c9df7.taxirides.realtime` LIMIT 1000
```



Great work! You just pulled 1000 taxi rides from a Pub/Sub topic and  pushed them to a BigQuery table. As you saw firsthand, templates are a  practical, easy-to-use way to run Dataflow jobs. 

Here is a sample of the 10 records

```json
[{
  "ride_id": "b7a027d0-ff51-4246-88e1-3d9d6409ab72",
  "point_idx": "651",
  "latitude": "40.704780000000007",
  "longitude": "-74.00507",
  "timestamp": "2024-06-30 09:58:28.117960 UTC",
  "meter_reading": "15.845455",
  "meter_increment": "0.024340177",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "a9844c4c-ae4b-471c-9fbb-d890433277dc",
  "point_idx": "53",
  "latitude": "40.74559",
  "longitude": "-73.99457000000001",
  "timestamp": "2024-06-30 09:58:33.234110 UTC",
  "meter_reading": "1.8110553",
  "meter_increment": "0.034170855",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "7445c505-e073-42a2-81b1-016762a83d49",
  "point_idx": "343",
  "latitude": "40.65466",
  "longitude": "-73.807210000000012",
  "timestamp": "2024-06-30 09:58:34.719850 UTC",
  "meter_reading": "7.841809",
  "meter_increment": "0.022862416",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "1b914424-ac0f-423d-9ce8-eb69f09eada4",
  "point_idx": "250",
  "latitude": "40.735490000000006",
  "longitude": "-73.91652",
  "timestamp": "2024-06-30 09:58:33.731600 UTC",
  "meter_reading": "5.298473",
  "meter_increment": "0.021193892",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "4254ba48-e0fa-4e50-8bfa-3d7a871f2f21",
  "point_idx": "133",
  "latitude": "40.65073",
  "longitude": "-73.785630000000012",
  "timestamp": "2024-06-30 09:58:28.753720 UTC",
  "meter_reading": "1.7530973",
  "meter_increment": "0.013181183",
  "ride_status": "enroute",
  "passenger_count": "6"
}, {
  "ride_id": "ca304808-31d8-44e3-8979-ecc77fe592b9",
  "point_idx": "2269",
  "latitude": "40.711650000000006",
  "longitude": "-74.164930000000012",
  "timestamp": "2024-06-30 09:58:34.359620 UTC",
  "meter_reading": "51.31192",
  "meter_increment": "0.022614332",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "89be6820-ff08-48cf-9529-ca64f43e7191",
  "point_idx": "499",
  "latitude": "40.74078",
  "longitude": "-73.94543",
  "timestamp": "2024-06-30 09:58:34.611600 UTC",
  "meter_reading": "10.899726",
  "meter_increment": "0.021843137",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "beb8ed12-438d-440a-8315-b1860cc4b30d",
  "point_idx": "206",
  "latitude": "40.73311",
  "longitude": "-73.987320000000011",
  "timestamp": "2024-06-30 09:58:28.656480 UTC",
  "meter_reading": "8.462441",
  "meter_increment": "0.04107981",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "ac8876a6-1051-491e-aee7-39f03a51daca",
  "point_idx": "1445",
  "latitude": "40.73734",
  "longitude": "-73.85221",
  "timestamp": "2024-06-30 09:58:33.244580 UTC",
  "meter_reading": "26.319237",
  "meter_increment": "0.018214004",
  "ride_status": "enroute",
  "passenger_count": "1"
}, {
  "ride_id": "e962316d-27e2-446b-a157-3c17b59b981d",
  "point_idx": "6",
  "latitude": "40.774730000000005",
  "longitude": "-73.954050000000009",
  "timestamp": "2024-06-30 09:58:33.880690 UTC",
  "meter_reading": "0.12327273",
  "meter_increment": "0.020545455",
  "ride_status": "enroute",
  "passenger_count": "1"
}]
```



You created a streaming pipeline using the Pub/Sub to BigQuery Dataflow  template, which reads messages written in JSON from a Pub/Sub topic and  pushes them to a BigQuery table.

## History

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ history 
    1  bq mk taxirides
    2  bq mk --time_partitioning_field timestamp --schema ride_id:string,point_idx:integer,latitude:float,longitude:float,timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,passenger_count:integer -t taxirides.realtime
    3  export BUCKET_NAME=qwiklabs-gcp-02-3029984c9df7
    4  gsutil mb gs://$BUCKET_NAME/
    5  gcloud dataflow jobs run iotflow     --gcs-location gs://dataflow-templates-us-west1/latest/PubSub_to_BigQuery     --region us-west1     --worker-machine-type e2-medium     --staging-location gs://qwiklabs-gcp-02-3029984c9df7/temp     --parameters inputTopic=projects/pubsub-public-data/topics/taxirides-realtime,outputTableSpec=qwiklabs-gcp-02-3029984c9df7:taxirides.realtime
    6  history 
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-3029984c9df7)$ 
```

