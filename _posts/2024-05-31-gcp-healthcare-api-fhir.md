---

layout: single
title:  "Ingesting FHIR Data with the Healthcare API"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Ingesting FHIR Data with the Healthcare API

Cloud Healthcare API provides a managed solution for storing and accessing healthcare data in Google Cloud, providing a critical bridge between existing care systems and applications hosted on Google Cloud. Using the API, you can unlock significant new capabilities for data analysis, machine learning and application development, and use these capabilities to build the next generation of healthcare solutions.

In this lab you will discover and use the basic functionality of Cloud Healthcare API using Fast Healthcare Interoperability Resources (FHIR) data model, how to export data to BigQuery, and how to access data in BigQuery via SQL.


- Gain a general understanding of Cloud Healthcare API and its role in managing healthcare data.
- Learn how to create Cloud Healthcare API datasets and stores.
- Import and export FHIR data using the Cloud Healthcare API.
- Export data from Cloud healthcare API to BigQuery
- Access data in BigQuery via SQL

##  Define the variables needed


```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-dc76fc006b40.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export PROJECT_NUMBER=$(gcloud projects list --filter=projectId:$PROJECT_ID \
  --format="value(projectNumber)")
export LOCATION=us-east1
export DATASET_ID=dataset1
export FHIR_STORE_ID=fhirstore1
export TOPIC=fhir-topic
export HL7_STORE_ID=hl7v2store1
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$
```
## Create BigQuery datasets
```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ bq --location=us-east1 mk --dataset --description HCAPI-dataset $PROJECT_ID:$DATASET_ID
Dataset 'qwiklabs-gcp-02-dc76fc006b40:dataset1' successfully created.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ bq --location=us-east1 mk --dataset --description HCAPI-dataset-de-id $PROJECT_ID:de_id
Dataset 'qwiklabs-gcp-02-dc76fc006b40:de_id' successfully created.
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-healthcare.iam.gserviceaccount.com \
--role=roles/bigquery.dataEditor
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-healthcare.iam.gserviceaccount.com \
--role=roles/bigquery.jobUser
Updated IAM policy for project [qwiklabs-gcp-02-dc76fc006b40].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:service-119798440602@gcp-sa-healthcare.iam.gserviceaccount.com
  role: roles/bigquery.dataEditor
- members:
  - serviceAccount:119798440602@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-119798440602@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-119798440602@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-119798440602@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:119798440602-compute@developer.gserviceaccount.com
  - serviceAccount:119798440602@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-119798440602@gcp-sa-healthcare.iam.gserviceaccount.com
  role: roles/healthcare.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/viewer
etag: BwYZlLFxM0M=
version: 1
Updated IAM policy for project [qwiklabs-gcp-02-dc76fc006b40].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:service-119798440602@gcp-sa-healthcare.iam.gserviceaccount.com
  role: roles/bigquery.dataEditor
- members:
  - serviceAccount:service-119798440602@gcp-sa-healthcare.iam.gserviceaccount.com
  role: roles/bigquery.jobUser
- members:
  - serviceAccount:119798440602@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-119798440602@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-119798440602@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-119798440602@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:119798440602-compute@developer.gserviceaccount.com
  - serviceAccount:119798440602@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-119798440602@gcp-sa-healthcare.iam.gserviceaccount.com
  role: roles/healthcare.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-dc76fc006b40@qwiklabs-gcp-02-dc76fc006b40.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-41d0c1af2373@qwiklabs.net
  role: roles/viewer
etag: BwYZlLGhprE=
version: 1
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```
## Healthcare API setup
Create a dataset for the healthcare API datastores to be organized under:

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ gcloud healthcare datasets create $DATASET_ID \
--location=$LOCATION
Create request issued for: [dataset1]
Waiting for operation [projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/operations/16361766510697381889] to complete...done.                             
Created dataset [dataset1].
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```
```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ gcloud healthcare fhir-stores import gcs $FHIR_STORE_ID \
--dataset=$DATASET_ID \
--location=$LOCATION \
--gcs-uri=gs://spls/gsp457/fhir_devdays_gcp/fhir1/* \
--content-structure=BUNDLE_PRETTY
Request issued for: [fhirstore1]
Waiting for operation [projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/operations/8430364616944517121] to complete...done.                              
complexDataTypeReferenceParsing: ENABLED
name: projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/fhirStores/fhirstore1
notificationConfigs:
- pubsubTopic: projects/qwiklabs-gcp-02-dc76fc006b40/topics/fhir-topic
validationConfig: {}
version: R4
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ gcloud healthcare fhir-stores export bq $FHIR_STORE_ID \
--dataset=$DATASET_ID \
--location=$LOCATION \
--bq-dataset=bq://$PROJECT_ID.$DATASET_ID \
--schema-type=analytics
Request issued for: [fhirstore1]
Waiting for operation [projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/operations/1278578039935991809] to complete...done.                              
complexDataTypeReferenceParsing: ENABLED
name: projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/fhirStores/fhirstore1
notificationConfigs:
- pubsubTopic: projects/qwiklabs-gcp-02-dc76fc006b40/topics/fhir-topic
validationConfig: {}
version: R4
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```
```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ gcloud healthcare fhir-stores export bq de_id \
--dataset=$DATASET_ID \
--location=$LOCATION \
--bq-dataset=bq://$PROJECT_ID.de_id \
--schema-type=analytics
Request issued for: [de_id]
Waiting for operation [projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/operations/16676877747124961281] to complete...done.                             
complexDataTypeReferenceParsing: ENABLED
name: projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/fhirStores/de_id
validationConfig: {}
version: R4
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```

```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ curl -X PATCH \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    --data "{ 'streamConfigs': [ { 'bigqueryDestination': { 'datasetUri':
'bq://$PROJECT_ID.$DATASET_ID', 'schemaConfig': { 'schemaType': 'ANALYTICS' } }
} ] }" \
"https://healthcare.googleapis.com/v1/projects/$PROJECT_ID/locations/$LOCATION/datasets/$DATASET_ID/fhirStores/$FHIR_STORE_ID?updateMask=streamConfigs"
{
  "name": "projects/qwiklabs-gcp-02-dc76fc006b40/locations/us-east1/datasets/dataset1/fhirStores/fhirstore1",
  "version": "R4",
  "streamConfigs": [
    {
      "bigqueryDestination": {
        "datasetUri": "bq://qwiklabs-gcp-02-dc76fc006b40.dataset1",
        "schemaConfig": {
          "schemaType": "ANALYTICS"
        }
      }
    }
  ],
  "validationConfig": {},
  "complexDataTypeReferenceParsing": "ENABLED",
  "notificationConfigs": [
    {
      "pubsubTopic": "projects/qwiklabs-gcp-02-dc76fc006b40/topics/fhir-topic"
    }
  ]
}
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ curl -X POST \
    -H "Authorization: Bearer $(gcloud auth application-default print-access-token)" \
    -H "Content-Type: application/fhir+json; charset=utf-8" \
    --data "{
      \"name\": [
        {
          \"use\": \"official\",
          \"family\": \"Smith\",
          \"given\": [
            \"Darcy\"
          ]
        }
      ],
      \"gender\": \"female\",
      \"birthDate\": \"1970-01-01\",
      \"resourceType\": \"Patient\"
    }" \
    "https://healthcare.googleapis.com/v1/projects/$PROJECT_ID/locations/$LOCATION/datasets/$DATASET_ID/fhirStores/$FHIR_STORE_ID/fhir/Patient"
{
  "birthDate": "1970-01-01",
  "gender": "female",
  "id": "2e43320c-7294-443e-bd38-314a3f4b277a",
  "meta": {
    "lastUpdated": "2024-05-29T10:08:34.563630+00:00",
    "versionId": "MTcxNjk3NzMxNDU2MzYzMDAwMA"
  },
  "name": [
    {
      "family": "Smith",
      "given": [
        "Darcy"
      ],
      "use": "official"
    }
  ],
  "resourceType": "Patient"
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```
## History
```sh
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ history 
    1  export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
    2  export PROJECT_NUMBER=$(gcloud projects list --filter=projectId:$PROJECT_ID \
  --format="value(projectNumber)")
    3  export LOCATION=us-east1
    4  export DATASET_ID=dataset1
    5  export FHIR_STORE_ID=fhirstore1
    6  export TOPIC=fhir-topic
    7  export HL7_STORE_ID=hl7v2store1
    8  bq --location=us-east1 mk --dataset --description HCAPI-dataset $PROJECT_ID:$DATASET_ID
    9  bq --location=us-east1 mk --dataset --description HCAPI-dataset-de-id $PROJECT_ID:de_id
   10  gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-healthcare.iam.gserviceaccount.com --role=roles/bigquery.dataEditor
   11  gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-healthcare.iam.gserviceaccount.com --role=roles/bigquery.jobUser
   12  gcloud healthcare datasets create $DATASET_ID --location=$LOCATION
   13  gcloud healthcare fhir-stores import gcs $FHIR_STORE_ID --dataset=$DATASET_ID --location=$LOCATION --gcs-uri=gs://spls/gsp457/fhir_devdays_gcp/fhir1/* --content-structure=BUNDLE_PRETTY
   14  gcloud healthcare fhir-stores export bq $FHIR_STORE_ID --dataset=$DATASET_ID --location=$LOCATION --bq-dataset=bq://$PROJECT_ID.$DATASET_ID --schema-type=analytics
   15  gcloud healthcare fhir-stores export bq de_id --dataset=$DATASET_ID --location=$LOCATION --bq-dataset=bq://$PROJECT_ID.de_id --schema-type=analytics
   16  curl -X PATCH     -H "Authorization: Bearer $(gcloud auth application-default print-access-token)"     -H "Content-Type: application/json; charset=utf-8"     --data "{ 'streamConfigs': [ { 'bigqueryDestination': { 'datasetUri':
'bq://$PROJECT_ID.$DATASET_ID', 'schemaConfig': { 'schemaType': 'ANALYTICS' } }
} ] }" "https://healthcare.googleapis.com/v1/projects/$PROJECT_ID/locations/$LOCATION/datasets/$DATASET_ID/fhirStores/$FHIR_STORE_ID?updateMask=streamConfigs"
   17  curl -X POST     -H "Authorization: Bearer $(gcloud auth application-default print-access-token)"     -H "Content-Type: application/fhir+json; charset=utf-8"     --data "{
      \"name\": [
        {
          \"use\": \"official\",
          \"family\": \"Smith\",
          \"given\": [
            \"Darcy\"
          ]
        }
      ],
      \"gender\": \"female\",
      \"birthDate\": \"1970-01-01\",
      \"resourceType\": \"Patient\"
    }"     "https://healthcare.googleapis.com/v1/projects/$PROJECT_ID/locations/$LOCATION/datasets/$DATASET_ID/fhirStores/$FHIR_STORE_ID/fhir/Patient"
   18  history 
student_01_41d0c1af2373@cloudshell:~ (qwiklabs-gcp-02-dc76fc006b40)$ 
```