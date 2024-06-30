---

layout: single
title:  "Vertex AI: Qwik Start"
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
# Vertex AI: Qwik Start

In this lab, you use BigQuery for data processing and exploratory data analysis and the Vertex AI platform to train and deploy a custom TensorFlow Regressor model to predict customer lifetime value. The goal of the lab is to introduce to Vertex AI through a high value real world use case - predictive CLV. You start with a local BigQuery and TensorFlow workflow that you may already be familiar with and progress toward training and deploying your model in the cloud with Vertex AI.

Vertex AI is Google Cloud's next generation, unified platform for  machine learning development and the successor to AI Platform announced  at Google I/O in May 2021. By developing machine learning solutions on  Vertex AI, you can leverage the latest ML pre-built components and  AutoML to significantly enhance development productivity, the ability to scale your workflow and decision making with your data, and accelerate  time to value.



- Train a TensorFlow model locally in a hosted [Vertex Notebook](https://cloud.google.com/vertex-ai/docs/general/notebooks?hl=sv).
- Create a [managed Tabular dataset](https://cloud.google.com/vertex-ai/docs/training/using-managed-datasets?hl=sv) artifact for experiment tracking.
- Containerize your training code with [Cloud Build](https://cloud.google.com/build) and push it to [Google Cloud Artifact Registry](https://cloud.google.com/artifact-registry).
- Run a [Vertex AI custom training job](https://cloud.google.com/vertex-ai/docs/training/custom-training) with your custom model container.
- Use [Vertex TensorBoard](https://cloud.google.com/vertex-ai/docs/experiments/tensorboard-overview) to visualize model performance.
- Deploy your trained model to a [Vertex Online Prediction Endpoint](https://cloud.google.com/vertex-ai/docs/predictions/getting-predictions) for serving predictions.
- Request an online prediction and explanation and see the response.



## Enable Google Cloud services

- In Cloud Shell, use `gcloud` to enable the services used in the lab:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-aff506dc3787.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ gcloud services enable \
  compute.googleapis.com \
  iam.googleapis.com \
  iamcredentials.googleapis.com \
  monitoring.googleapis.com \
  logging.googleapis.com \
  notebooks.googleapis.com \
  aiplatform.googleapis.com \
  bigquery.googleapis.com \
  artifactregistry.googleapis.com \
  cloudbuild.googleapis.com \
  container.googleapis.com
Operation "operations/acat.p2-137820269452-727bc02e-7c3f-4415-963b-ba8e59c9d80d" finished successfully.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ 
```



## Create Vertex AI custom service account for Vertex Tensorboard integration

1. Create custom service account and Grant it access to Cloud Storage for writing and retrieving Tensorboard logs:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ SERVICE_ACCOUNT_ID=vertex-custom-training-sa
gcloud iam service-accounts create $SERVICE_ACCOUNT_ID  \
    --description="A custom service account for Vertex custom training with Tensorboard" \
    --display-name="Vertex AI Custom Training"
Created service account [vertex-custom-training-sa].
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ 
```

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ PROJECT_ID=$(gcloud config get-value core/project)
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
    --role="roles/storage.admin"
Your active configuration is: [cloudshell-21982]
Updated IAM policy for project [qwiklabs-gcp-02-aff506dc3787].
bindings:
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/aiplatform.admin
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/bigquery.admin
- members:
  - serviceAccount:137820269452@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-137820269452@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-137820269452@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-137820269452@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:137820269452-compute@developer.gserviceaccount.com
  - serviceAccount:137820269452@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/iam.serviceAccountUser
- members:
  - serviceAccount:service-137820269452@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/storage.admin
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-137820269452@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
etag: BwYcG_K1BAw=
version: 1
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ 
```

Grant it access to your BigQuery data source to read data into your TensorFlow model:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
    --role="roles/bigquery.admin"
Updated IAM policy for project [qwiklabs-gcp-02-aff506dc3787].
bindings:
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/aiplatform.admin
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/bigquery.admin
- members:
  - serviceAccount:137820269452@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-137820269452@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-137820269452@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-137820269452@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:137820269452-compute@developer.gserviceaccount.com
  - serviceAccount:137820269452@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/iam.serviceAccountUser
- members:
  - serviceAccount:service-137820269452@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/storage.admin
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-137820269452@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
etag: BwYcG_VQ70k=
version: 1
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ 
```

Grant it access to Vertex AI for running model training, deployment, and explanation jobs:

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$SERVICE_ACCOUNT_ID@$PROJECT_ID.iam.gserviceaccount.com \
    --role="roles/aiplatform.user"
Updated IAM policy for project [qwiklabs-gcp-02-aff506dc3787].
bindings:
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/aiplatform.admin
- members:
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  role: roles/aiplatform.user
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/bigquery.admin
- members:
  - serviceAccount:137820269452@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-137820269452@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-137820269452@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-137820269452@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:137820269452-compute@developer.gserviceaccount.com
  - serviceAccount:137820269452@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/iam.serviceAccountUser
- members:
  - serviceAccount:service-137820269452@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-aff506dc3787@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - serviceAccount:vertex-custom-training-sa@qwiklabs-gcp-02-aff506dc3787.iam.gserviceaccount.com
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/storage.admin
- members:
  - user:student-01-932d053b64d1@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-137820269452@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
etag: BwYcG_cZt5I=
version: 1
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-02-aff506dc3787)$ 
```



## Launch Vertex AI Workbench notebook

To create and launch a Vertex AI Workbench notebook:

1. In the **Navigation Menu** ![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D), click **Vertex AI** > **Workbench**.
2. On the **Workbench** page, click **Enable Notebooks API** (if it isn't enabled yet).
3. Click on **User-Managed Notebooks** tab then, click **Create New**.
4. Name the notebook.



## Clone the lab repository

Next you'll clone the `training-data-analyst` repo to your JupyterLab instance.

To clone the `training-data-analyst` repository in your JupyterLab instance:

1. In JupyterLab, click the **Terminal** icon to open a new terminal.

## Install lab dependencies

- Run the following to go to the `training-data-analyst/self-paced-labs/vertex-ai/vertex-ai-qwikstart` folder, then `pip3 install` `requirements.txt` to install lab dependencies:

```sh
(base) jupyter@instance-20240630-193809:~$ cd training-data-analyst/self-paced-labs/vertex-ai/vertex-ai-qwikstart
pip3 install --user -r requirements.txt
sudo apt -y install python3-pandas
pip uninstall openpyxl
pip install openpyxl
Requirement already satisfied: tensorflow>=2.11.0 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 1)) (2.11.0)
Collecting pyarrow==7.0.0 (from -r requirements.txt (line 2))
  Downloading pyarrow-7.0.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (2.9 kB)
Requirement already satisfied: httplib2>=0.20.4 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 3)) (0.21.0)
Requirement already satisfied: grpcio-status>=1.38.1 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 4)) (1.48.0)
Requirement already satisfied: google-api-python-client>=1.8.0 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 5)) (1.8.0)
Requirement already satisfied: apache-beam>=2.28.0 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 6)) (2.46.0)
Requirement already satisfied: google-cloud-aiplatform>=1.8.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (1.55.0)
Requirement already satisfied: six==1.16.0 in /opt/conda/lib/python3.10/site-packages (from -r requirements.txt (line 8)) (1.16.0)
Collecting wget==3.2 (from -r requirements.txt (line 9))
  Downloading wget-3.2.zip (10 kB)
  Preparing metadata (setup.py) ... done
Collecting xlrd==2.0.1 (from -r requirements.txt (line 10))
  Downloading xlrd-2.0.1-py2.py3-none-any.whl.metadata (3.4 kB)
Collecting openpyxl==3.0.10 (from -r requirements.txt (line 11))
  Downloading openpyxl-3.0.10-py2.py3-none-any.whl.metadata (2.4 kB)
Requirement already satisfied: numpy>=1.16.6 in /opt/conda/lib/python3.10/site-packages (from pyarrow==7.0.0->-r requirements.txt (line 2)) (1.24.4)
Collecting et-xmlfile (from openpyxl==3.0.10->-r requirements.txt (line 11))
  Downloading et_xmlfile-1.1.0-py3-none-any.whl.metadata (1.8 kB)
Requirement already satisfied: absl-py>=1.0.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (1.4.0)
Requirement already satisfied: astunparse>=1.6.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (1.6.3)
Requirement already satisfied: flatbuffers>=2.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (24.3.25)
Requirement already satisfied: gast<=0.4.0,>=0.2.1 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.4.0)
Requirement already satisfied: google-pasta>=0.1.1 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.2.0)
Requirement already satisfied: grpcio<2.0,>=1.24.3 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (1.48.0)
Requirement already satisfied: h5py>=2.9.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (3.11.0)
Requirement already satisfied: keras<2.12,>=2.11.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (2.11.0)
Requirement already satisfied: libclang>=13.0.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (18.1.1)
Requirement already satisfied: opt-einsum>=2.3.2 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (3.3.0)
Requirement already satisfied: packaging in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (24.1)
Requirement already satisfied: protobuf<3.20,>=3.9.2 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (3.19.6)
Requirement already satisfied: setuptools in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (70.0.0)
Requirement already satisfied: tensorboard<2.12,>=2.11 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (2.11.2)
Requirement already satisfied: tensorflow-estimator<2.12,>=2.11.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (2.11.0)
Requirement already satisfied: termcolor>=1.1.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (2.4.0)
Requirement already satisfied: typing-extensions>=3.6.6 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (4.12.2)
Requirement already satisfied: wrapt>=1.11.0 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (1.16.0)
Requirement already satisfied: tensorflow-io-gcs-filesystem>=0.23.1 in /opt/conda/lib/python3.10/site-packages (from tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.29.0)
Requirement already satisfied: pyparsing!=3.0.0,!=3.0.1,!=3.0.2,!=3.0.3,<4,>=2.4.2 in /opt/conda/lib/python3.10/site-packages (from httplib2>=0.20.4->-r requirements.txt (line 3)) (3.1.2)
Requirement already satisfied: googleapis-common-protos>=1.5.5 in /opt/conda/lib/python3.10/site-packages (from grpcio-status>=1.38.1->-r requirements.txt (line 4)) (1.63.1)
Requirement already satisfied: google-auth>=1.4.1 in /opt/conda/lib/python3.10/site-packages (from google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (2.30.0)
Requirement already satisfied: google-auth-httplib2>=0.0.3 in /opt/conda/lib/python3.10/site-packages (from google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (0.1.1)
Requirement already satisfied: google-api-core<2dev,>=1.13.0 in /opt/conda/lib/python3.10/site-packages (from google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (1.34.1)
Requirement already satisfied: uritemplate<4dev,>=3.0.0 in /opt/conda/lib/python3.10/site-packages (from google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (3.0.1)
Requirement already satisfied: crcmod<2.0,>=1.7 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (1.7)
Requirement already satisfied: orjson<4.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (3.10.5)
Requirement already satisfied: dill<0.3.2,>=0.3.1.1 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (0.3.1.1)
Requirement already satisfied: cloudpickle~=2.2.1 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2.2.1)
Requirement already satisfied: fastavro<2,>=0.23.6 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (1.9.4)
Requirement already satisfied: fasteners<1.0,>=0.3 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (0.19)
Collecting grpcio<2.0,>=1.24.3 (from tensorflow>=2.11.0->-r requirements.txt (line 1))
  Downloading grpcio-1.64.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.3 kB)
Requirement already satisfied: hdfs<3.0.0,>=2.1.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2.7.3)
Requirement already satisfied: objsize<0.7.0,>=0.6.1 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (0.6.1)
Requirement already satisfied: pymongo<4.0.0,>=3.8.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (3.13.0)
Requirement already satisfied: proto-plus<2,>=1.7.1 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (1.23.0)
Requirement already satisfied: pydot<2,>=1.2.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (1.4.2)
Requirement already satisfied: python-dateutil<3,>=2.8.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2.9.0)
Requirement already satisfied: pytz>=2018.3 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2024.1)
Requirement already satisfied: regex>=2020.6.8 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2024.5.15)
Requirement already satisfied: requests<3.0.0,>=2.24.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (2.32.3)
Requirement already satisfied: zstandard<1,>=0.18.0 in /opt/conda/lib/python3.10/site-packages (from apache-beam>=2.28.0->-r requirements.txt (line 6)) (0.22.0)
Requirement already satisfied: google-cloud-storage<3.0.0dev,>=1.32.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (2.14.0)
Requirement already satisfied: google-cloud-bigquery!=3.20.0,<4.0.0dev,>=1.15.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (3.24.0)
Requirement already satisfied: google-cloud-resource-manager<3.0.0dev,>=1.3.3 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (1.12.3)
Requirement already satisfied: shapely<3.0.0dev in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (2.0.4)
Requirement already satisfied: pydantic<3 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (1.10.16)
Requirement already satisfied: docstring-parser<1 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (0.16)
Requirement already satisfied: tensorboard-plugin-profile<3.0.0dev,>=2.4.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (2.15.1)
Collecting werkzeug<2.1.0dev,>=2.0.0 (from google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7))
  Downloading Werkzeug-2.0.3-py3-none-any.whl.metadata (4.5 kB)
Requirement already satisfied: wheel<1.0,>=0.23.0 in /opt/conda/lib/python3.10/site-packages (from astunparse>=1.6.0->tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.43.0)
Requirement already satisfied: cachetools<6.0,>=2.0.0 in /opt/conda/lib/python3.10/site-packages (from google-auth>=1.4.1->google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (4.2.4)
Requirement already satisfied: pyasn1-modules>=0.2.1 in /opt/conda/lib/python3.10/site-packages (from google-auth>=1.4.1->google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (0.4.0)
Requirement already satisfied: rsa<5,>=3.1.4 in /opt/conda/lib/python3.10/site-packages (from google-auth>=1.4.1->google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (4.9)
Requirement already satisfied: google-cloud-core<3.0.0dev,>=1.6.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-bigquery!=3.20.0,<4.0.0dev,>=1.15.0->google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (2.4.1)
Requirement already satisfied: google-resumable-media<3.0dev,>=0.6.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-bigquery!=3.20.0,<4.0.0dev,>=1.15.0->google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (2.7.1)
Requirement already satisfied: grpc-google-iam-v1<1.0.0dev,>=0.12.4 in /opt/conda/lib/python3.10/site-packages (from google-cloud-resource-manager<3.0.0dev,>=1.3.3->google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (0.12.7)
Requirement already satisfied: google-crc32c<2.0dev,>=1.0 in /opt/conda/lib/python3.10/site-packages (from google-cloud-storage<3.0.0dev,>=1.32.0->google-cloud-aiplatform>=1.8.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (1.5.0)
Requirement already satisfied: docopt in /opt/conda/lib/python3.10/site-packages (from hdfs<3.0.0,>=2.1.0->apache-beam>=2.28.0->-r requirements.txt (line 6)) (0.6.2)
Requirement already satisfied: charset-normalizer<4,>=2 in /opt/conda/lib/python3.10/site-packages (from requests<3.0.0,>=2.24.0->apache-beam>=2.28.0->-r requirements.txt (line 6)) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /opt/conda/lib/python3.10/site-packages (from requests<3.0.0,>=2.24.0->apache-beam>=2.28.0->-r requirements.txt (line 6)) (3.7)
Requirement already satisfied: urllib3<3,>=1.21.1 in /opt/conda/lib/python3.10/site-packages (from requests<3.0.0,>=2.24.0->apache-beam>=2.28.0->-r requirements.txt (line 6)) (1.26.18)
Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/lib/python3.10/site-packages (from requests<3.0.0,>=2.24.0->apache-beam>=2.28.0->-r requirements.txt (line 6)) (2024.6.2)
Requirement already satisfied: google-auth-oauthlib<0.5,>=0.4.1 in /opt/conda/lib/python3.10/site-packages (from tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.4.6)
Requirement already satisfied: markdown>=2.6.8 in /opt/conda/lib/python3.10/site-packages (from tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (3.6)
Requirement already satisfied: tensorboard-data-server<0.7.0,>=0.6.0 in /opt/conda/lib/python3.10/site-packages (from tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (0.6.1)
Requirement already satisfied: tensorboard-plugin-wit>=1.6.0 in /opt/conda/lib/python3.10/site-packages (from tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (1.8.1)
Requirement already satisfied: gviz-api>=1.9.0 in /opt/conda/lib/python3.10/site-packages (from tensorboard-plugin-profile<3.0.0dev,>=2.4.0->google-cloud-aiplatform[tensorboard]>=1.8.0->-r requirements.txt (line 7)) (1.10.0)
Requirement already satisfied: requests-oauthlib>=0.7.0 in /opt/conda/lib/python3.10/site-packages (from google-auth-oauthlib<0.5,>=0.4.1->tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (2.0.0)
Requirement already satisfied: pyasn1<0.7.0,>=0.4.6 in /opt/conda/lib/python3.10/site-packages (from pyasn1-modules>=0.2.1->google-auth>=1.4.1->google-api-python-client>=1.8.0->-r requirements.txt (line 5)) (0.6.0)
Requirement already satisfied: oauthlib>=3.0.0 in /opt/conda/lib/python3.10/site-packages (from requests-oauthlib>=0.7.0->google-auth-oauthlib<0.5,>=0.4.1->tensorboard<2.12,>=2.11->tensorflow>=2.11.0->-r requirements.txt (line 1)) (3.2.2)
Downloading pyarrow-7.0.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26.7 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 26.7/26.7 MB 46.8 MB/s eta 0:00:00
Downloading xlrd-2.0.1-py2.py3-none-any.whl (96 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.5/96.5 kB 13.7 MB/s eta 0:00:00
Downloading openpyxl-3.0.10-py2.py3-none-any.whl (242 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 242.1/242.1 kB 29.7 MB/s eta 0:00:00
Downloading grpcio-1.64.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (5.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.6/5.6 MB 56.3 MB/s eta 0:00:00
Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 289.2/289.2 kB 35.0 MB/s eta 0:00:00
Downloading et_xmlfile-1.1.0-py3-none-any.whl (4.7 kB)
Building wheels for collected packages: wget
  Building wheel for wget (setup.py) ... done
  Created wheel for wget: filename=wget-3.2-py3-none-any.whl size=9656 sha256=a143f9ba84737015027950b6a871d00acad7d67e2a4d2df8b31c43ced476154d
  Stored in directory: /home/jupyter/.cache/pip/wheels/8b/f1/7f/5c94f0a7a505ca1c81cd1d9208ae2064675d97582078e6c769
Successfully built wget
Installing collected packages: wget, xlrd, werkzeug, pyarrow, grpcio, et-xmlfile, openpyxl
  WARNING: The script plasma_store is installed in '/home/jupyter/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed et-xmlfile-1.1.0 grpcio-1.64.1 openpyxl-3.0.10 pyarrow-7.0.0 werkzeug-2.0.3 wget-3.2 xlrd-2.0.1
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  binfmt-support fonts-font-awesome fonts-lyx libblosc1 libclang-cpp9 libffi-dev libimagequant0 libjs-jquery-ui libjs-sphinxdoc libjs-underscore liblbfgsb0 libllvm9 liblzo2-2
  libncurses-dev libpfm4 libtbb2 libtinfo-dev libwebpdemux2 libwebpmux3 libxslt1.1 libz3-dev llvm-9 llvm-9-dev llvm-9-runtime llvm-9-tools mailcap mime-support numba-doc
  python-matplotlib-data python-odf-doc python-odf-tools python-tables-data python3-attr python3-bottleneck python3-bs4 python3-cycler python3-dateutil python3-decorator
  python3-defusedxml python3-et-xmlfile python3-html5lib python3-importlib-metadata python3-iniconfig python3-jdcal python3-jinja2 python3-kiwisolver python3-llvmlite
  python3-lxml python3-markupsafe python3-matplotlib python3-more-itertools python3-numba python3-numexpr python3-numpy python3-odf python3-olefile python3-openpyxl
  python3-packaging python3-pandas-lib python3-pil python3-pluggy python3-py python3-pygments python3-pyparsing python3-pytest python3-scipy python3-soupsieve python3-tables
  python3-tables-lib python3-toml python3-tz python3-webencodings python3-xlwt python3-yaml python3-zipp sphinx-rtd-theme-common ttf-bitstream-vera
Suggested packages:
  libjs-jquery-ui-docs ncurses-doc llvm-9-doc python-attr-doc python-bottleneck-doc python-cycler-doc python3-genshi python-jinja2-doc llvmlite-doc python3-lxml-dbg
  python-lxml-doc dvipng ffmpeg ghostscript gir1.2-gtk-3.0 inkscape ipython3 librsvg2-common python-matplotlib-doc python3-cairocffi python3-gi-cairo python3-gobject
  python3-nose python3-pyqt5 python3-sip python3-tornado texlive-extra-utils ttf-staypuft nvidia-cuda-toolkit gfortran python-numpy-doc python3-dev python3-numpy-dbg
  python-pandas-doc python3-statsmodels python-pil-doc python3-pil-dbg subversion python-pygments-doc python-pyparsing-doc python-scipy-doc python3-netcdf4 python-tables-doc
  vitables python3-xlrd python-xlrt-doc
The following NEW packages will be installed:
  binfmt-support fonts-font-awesome fonts-lyx libblosc1 libclang-cpp9 libffi-dev libimagequant0 libjs-jquery-ui libjs-sphinxdoc libjs-underscore liblbfgsb0 libllvm9 liblzo2-2
  libncurses-dev libpfm4 libtbb2 libtinfo-dev libwebpdemux2 libwebpmux3 libxslt1.1 libz3-dev llvm-9 llvm-9-dev llvm-9-runtime llvm-9-tools mailcap mime-support numba-doc
  python-matplotlib-data python-odf-doc python-odf-tools python-tables-data python3-attr python3-bottleneck python3-bs4 python3-cycler python3-dateutil python3-decorator
  python3-defusedxml python3-et-xmlfile python3-html5lib python3-importlib-metadata python3-iniconfig python3-jdcal python3-jinja2 python3-kiwisolver python3-llvmlite
  python3-lxml python3-markupsafe python3-matplotlib python3-more-itertools python3-numba python3-numexpr python3-numpy python3-odf python3-olefile python3-openpyxl
  python3-packaging python3-pandas python3-pandas-lib python3-pil python3-pluggy python3-py python3-pygments python3-pyparsing python3-pytest python3-scipy python3-soupsieve
  python3-tables python3-tables-lib python3-toml python3-tz python3-webencodings python3-xlwt python3-yaml python3-zipp sphinx-rtd-theme-common ttf-bitstream-vera
0 upgraded, 78 newly installed, 0 to remove and 2 not upgraded.
Need to get 93.5 MB of archives.
After this operation, 504 MB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 mailcap all 3.69 [31.7 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 mime-support all 3.66 [10.9 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 binfmt-support amd64 2.2.1-1+deb11u1 [66.8 kB]
Get:4 https://deb.debian.org/debian bullseye/main amd64 fonts-font-awesome all 5.0.10+really4.7.0~dfsg-4.1 [517 kB]
Get:5 https://deb.debian.org/debian bullseye/main amd64 fonts-lyx all 2.3.6-1 [205 kB]
Get:6 https://deb.debian.org/debian bullseye/main amd64 libblosc1 amd64 1.20.1+ds1-2 [48.7 kB]
Get:7 https://deb.debian.org/debian bullseye/main amd64 libllvm9 amd64 1:9.0.1-16.1 [14.9 MB]
Get:8 https://deb.debian.org/debian bullseye/main amd64 libclang-cpp9 amd64 1:9.0.1-16.1 [8421 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 libffi-dev amd64 3.3-6 [56.5 kB]
Get:10 https://deb.debian.org/debian bullseye/main amd64 libimagequant0 amd64 2.12.2-1.1 [32.5 kB]
Get:11 https://deb.debian.org/debian bullseye/main amd64 libjs-jquery-ui all 1.12.1+dfsg-8+deb11u2 [232 kB]
Get:12 https://deb.debian.org/debian bullseye/main amd64 libjs-underscore all 1.9.1~dfsg-3 [100 kB]
Get:13 https://deb.debian.org/debian bullseye/main amd64 libjs-sphinxdoc all 3.4.3-2 [127 kB]
Get:14 https://deb.debian.org/debian bullseye/main amd64 liblbfgsb0 amd64 3.0+dfsg.3-9 [28.5 kB]
Get:15 https://deb.debian.org/debian bullseye/main amd64 liblzo2-2 amd64 2.10-2 [56.9 kB]
Get:16 https://deb.debian.org/debian bullseye/main amd64 libncurses-dev amd64 6.2+20201114-2+deb11u2 [344 kB]
Get:17 https://deb.debian.org/debian bullseye/main amd64 libpfm4 amd64 4.11.1+git32-gd0b85fb-1 [286 kB]
Get:18 https://deb.debian.org/debian bullseye/main amd64 libtbb2 amd64 2020.3-1 [161 kB]
Get:19 https://deb.debian.org/debian bullseye/main amd64 libtinfo-dev amd64 6.2+20201114-2+deb11u2 [940 B]
Get:20 https://deb.debian.org/debian bullseye/main amd64 libwebpdemux2 amd64 0.6.1-2.1+deb11u2 [87.8 kB]
Get:21 https://deb.debian.org/debian bullseye/main amd64 libwebpmux3 amd64 0.6.1-2.1+deb11u2 [97.7 kB]
Get:22 https://deb.debian.org/debian bullseye/main amd64 libxslt1.1 amd64 1.1.34-4+deb11u1 [240 kB]
Get:23 https://deb.debian.org/debian bullseye/main amd64 libz3-dev amd64 4.8.10-1 [90.8 kB]
Get:24 https://deb.debian.org/debian bullseye/main amd64 llvm-9-runtime amd64 1:9.0.1-16.1 [213 kB]
Get:25 https://deb.debian.org/debian bullseye/main amd64 llvm-9 amd64 1:9.0.1-16.1 [4851 kB]
Get:26 https://deb.debian.org/debian bullseye/main amd64 python3-pygments all 2.7.1+dfsg-2.1 [657 kB]
Get:27 https://deb.debian.org/debian bullseye/main amd64 python3-yaml amd64 5.3.1-5 [138 kB]
Get:28 https://deb.debian.org/debian bullseye/main amd64 llvm-9-tools amd64 1:9.0.1-16.1 [330 kB]
Get:29 https://deb.debian.org/debian bullseye/main amd64 llvm-9-dev amd64 1:9.0.1-16.1 [24.4 MB]
Get:30 https://deb.debian.org/debian bullseye/main amd64 sphinx-rtd-theme-common all 0.5.1+dfsg-1 [995 kB]
Get:31 https://deb.debian.org/debian bullseye/main amd64 numba-doc all 0.52.0-4 [756 kB]
Get:32 https://deb.debian.org/debian bullseye/main amd64 ttf-bitstream-vera all 1.10-8.1 [223 kB]
Get:33 https://deb.debian.org/debian bullseye/main amd64 python-matplotlib-data all 3.3.4-1 [4153 kB]
Get:34 https://deb.debian.org/debian bullseye/main amd64 python-odf-doc all 1.4.1-1 [244 kB]
Get:35 https://deb.debian.org/debian bullseye/main amd64 python3-defusedxml all 0.6.0-2 [38.2 kB]
Get:36 https://deb.debian.org/debian bullseye/main amd64 python3-odf all 1.4.1-1 [79.7 kB]
Get:37 https://deb.debian.org/debian bullseye/main amd64 python-odf-tools all 1.4.1-1 [29.8 kB]
Get:38 https://deb.debian.org/debian bullseye/main amd64 python-tables-data all 3.6.1-3 [53.1 kB]
Get:39 https://deb.debian.org/debian bullseye/main amd64 python3-attr all 20.3.0-1 [52.9 kB]
Get:40 https://deb.debian.org/debian bullseye/main amd64 python3-numpy amd64 1:1.19.5-1 [2693 kB]
Get:41 https://deb.debian.org/debian bullseye/main amd64 python3-bottleneck amd64 1.2.1+ds1-2+b4 [77.3 kB]
Get:42 https://deb.debian.org/debian bullseye/main amd64 python3-soupsieve all 2.2.1-1 [34.7 kB]
Get:43 https://deb.debian.org/debian bullseye/main amd64 python3-bs4 all 4.9.3-1 [112 kB]
Get:44 https://deb.debian.org/debian bullseye/main amd64 python3-cycler all 0.10.0-3 [8084 B]
Get:45 https://deb.debian.org/debian bullseye/main amd64 python3-dateutil all 2.8.1-6 [79.2 kB]
Get:46 https://deb.debian.org/debian bullseye/main amd64 python3-decorator all 4.4.2-2 [15.8 kB]
Get:47 https://deb.debian.org/debian bullseye/main amd64 python3-et-xmlfile all 1.0.1-2.1 [9180 B]
Get:48 https://deb.debian.org/debian bullseye/main amd64 python3-webencodings all 0.5.1-2 [11.0 kB]
Get:49 https://deb.debian.org/debian bullseye/main amd64 python3-html5lib all 1.1-3 [93.0 kB]
Get:50 https://deb.debian.org/debian bullseye/main amd64 python3-more-itertools all 4.2.0-3 [42.7 kB]
Get:51 https://deb.debian.org/debian bullseye/main amd64 python3-zipp all 1.0.0-3 [6060 B]
Get:52 https://deb.debian.org/debian bullseye/main amd64 python3-importlib-metadata all 1.6.0-2 [10.3 kB]
Get:53 https://deb.debian.org/debian bullseye/main amd64 python3-iniconfig all 1.1.1-1 [6308 B]
Get:54 https://deb.debian.org/debian bullseye/main amd64 python3-jdcal all 1.0-1.3 [7904 B]
Get:55 https://deb.debian.org/debian bullseye/main amd64 python3-markupsafe amd64 1.1.1-1+b3 [15.2 kB]
Get:56 https://deb.debian.org/debian bullseye/main amd64 python3-jinja2 all 2.11.3-1 [114 kB]
Get:57 https://deb.debian.org/debian bullseye/main amd64 python3-kiwisolver amd64 1.3.1-1+b1 [55.7 kB]
Get:58 https://deb.debian.org/debian bullseye/main amd64 python3-llvmlite amd64 0.35.0-3 [120 kB]
Get:59 https://deb.debian.org/debian bullseye/main amd64 python3-lxml amd64 4.6.3+dfsg-0.1+deb11u1 [1093 kB]
Get:60 https://deb.debian.org/debian bullseye/main amd64 python3-pyparsing all 2.4.7-1 [109 kB]
Get:61 https://deb.debian.org/debian-security bullseye-security/main amd64 python3-pil amd64 8.1.2+dfsg-0.3+deb11u2 [446 kB]
Get:62 https://deb.debian.org/debian bullseye/main amd64 python3-matplotlib amd64 3.3.4-1 [4163 kB]
Get:63 https://deb.debian.org/debian bullseye/main amd64 python3-scipy amd64 1.6.0-2 [12.3 MB]
Get:64 https://deb.debian.org/debian bullseye/main amd64 python3-numba amd64 0.52.0-4 [1515 kB]
Get:65 https://deb.debian.org/debian bullseye/main amd64 python3-numexpr amd64 2.7.2-2 [144 kB]
Get:66 https://deb.debian.org/debian bullseye/main amd64 python3-olefile all 0.46-3 [36.1 kB]
Get:67 https://deb.debian.org/debian bullseye/main amd64 python3-openpyxl all 3.0.3-1 [159 kB]
Get:68 https://deb.debian.org/debian bullseye/main amd64 python3-packaging all 20.9-2 [33.5 kB]
Get:69 https://deb.debian.org/debian bullseye/main amd64 python3-tz all 2021.1-1 [34.8 kB]
Get:70 https://deb.debian.org/debian bullseye/main amd64 python3-pandas-lib amd64 1.1.5+dfsg-2 [3287 kB]
Get:71 https://deb.debian.org/debian bullseye/main amd64 python3-pandas all 1.1.5+dfsg-2 [2096 kB]
Get:72 https://deb.debian.org/debian bullseye/main amd64 python3-pluggy all 0.13.0-6 [22.3 kB]
Get:73 https://deb.debian.org/debian bullseye/main amd64 python3-py all 1.10.0-1 [94.2 kB]
Get:74 https://deb.debian.org/debian bullseye/main amd64 python3-toml all 0.10.1-1 [15.9 kB]
Get:75 https://deb.debian.org/debian bullseye/main amd64 python3-pytest all 6.0.2-2 [211 kB]
Get:76 https://deb.debian.org/debian bullseye/main amd64 python3-tables-lib amd64 3.6.1-3 [394 kB]
Get:77 https://deb.debian.org/debian bullseye/main amd64 python3-tables all 3.6.1-3 [342 kB]
Get:78 https://deb.debian.org/debian bullseye/main amd64 python3-xlwt all 1.3.0-3 [86.1 kB]
Fetched 93.5 MB in 2s (58.0 MB/s)       
Extracting templates from packages: 100%
Selecting previously unselected package mailcap.
(Reading database ... 137209 files and directories currently installed.)
Preparing to unpack .../00-mailcap_3.69_all.deb ...
Unpacking mailcap (3.69) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../01-mime-support_3.66_all.deb ...
Unpacking mime-support (3.66) ...
Selecting previously unselected package binfmt-support.
Preparing to unpack .../02-binfmt-support_2.2.1-1+deb11u1_amd64.deb ...
Unpacking binfmt-support (2.2.1-1+deb11u1) ...
Selecting previously unselected package fonts-font-awesome.
Preparing to unpack .../03-fonts-font-awesome_5.0.10+really4.7.0~dfsg-4.1_all.deb ...
Unpacking fonts-font-awesome (5.0.10+really4.7.0~dfsg-4.1) ...
Selecting previously unselected package fonts-lyx.
Preparing to unpack .../04-fonts-lyx_2.3.6-1_all.deb ...
Unpacking fonts-lyx (2.3.6-1) ...
Selecting previously unselected package libblosc1.
Preparing to unpack .../05-libblosc1_1.20.1+ds1-2_amd64.deb ...
Unpacking libblosc1 (1.20.1+ds1-2) ...
Selecting previously unselected package libllvm9:amd64.
Preparing to unpack .../06-libllvm9_1%3a9.0.1-16.1_amd64.deb ...
Unpacking libllvm9:amd64 (1:9.0.1-16.1) ...
Selecting previously unselected package libclang-cpp9.
Preparing to unpack .../07-libclang-cpp9_1%3a9.0.1-16.1_amd64.deb ...
Unpacking libclang-cpp9 (1:9.0.1-16.1) ...
Selecting previously unselected package libffi-dev:amd64.
Preparing to unpack .../08-libffi-dev_3.3-6_amd64.deb ...
Unpacking libffi-dev:amd64 (3.3-6) ...
Selecting previously unselected package libimagequant0:amd64.
Preparing to unpack .../09-libimagequant0_2.12.2-1.1_amd64.deb ...
Unpacking libimagequant0:amd64 (2.12.2-1.1) ...
Selecting previously unselected package libjs-jquery-ui.
Preparing to unpack .../10-libjs-jquery-ui_1.12.1+dfsg-8+deb11u2_all.deb ...
Unpacking libjs-jquery-ui (1.12.1+dfsg-8+deb11u2) ...
Selecting previously unselected package libjs-underscore.
Preparing to unpack .../11-libjs-underscore_1.9.1~dfsg-3_all.deb ...
Unpacking libjs-underscore (1.9.1~dfsg-3) ...
Selecting previously unselected package libjs-sphinxdoc.
Preparing to unpack .../12-libjs-sphinxdoc_3.4.3-2_all.deb ...
Unpacking libjs-sphinxdoc (3.4.3-2) ...
Selecting previously unselected package liblbfgsb0:amd64.
Preparing to unpack .../13-liblbfgsb0_3.0+dfsg.3-9_amd64.deb ...
Unpacking liblbfgsb0:amd64 (3.0+dfsg.3-9) ...
Selecting previously unselected package liblzo2-2:amd64.
Preparing to unpack .../14-liblzo2-2_2.10-2_amd64.deb ...
Unpacking liblzo2-2:amd64 (2.10-2) ...
Selecting previously unselected package libncurses-dev:amd64.
Preparing to unpack .../15-libncurses-dev_6.2+20201114-2+deb11u2_amd64.deb ...
Unpacking libncurses-dev:amd64 (6.2+20201114-2+deb11u2) ...
Selecting previously unselected package libpfm4:amd64.
Preparing to unpack .../16-libpfm4_4.11.1+git32-gd0b85fb-1_amd64.deb ...
Unpacking libpfm4:amd64 (4.11.1+git32-gd0b85fb-1) ...
Selecting previously unselected package libtbb2:amd64.
Preparing to unpack .../17-libtbb2_2020.3-1_amd64.deb ...
Unpacking libtbb2:amd64 (2020.3-1) ...
Selecting previously unselected package libtinfo-dev:amd64.
Preparing to unpack .../18-libtinfo-dev_6.2+20201114-2+deb11u2_amd64.deb ...
Unpacking libtinfo-dev:amd64 (6.2+20201114-2+deb11u2) ...
Selecting previously unselected package libwebpdemux2:amd64.
Preparing to unpack .../19-libwebpdemux2_0.6.1-2.1+deb11u2_amd64.deb ...
Unpacking libwebpdemux2:amd64 (0.6.1-2.1+deb11u2) ...
Selecting previously unselected package libwebpmux3:amd64.
Preparing to unpack .../20-libwebpmux3_0.6.1-2.1+deb11u2_amd64.deb ...
Unpacking libwebpmux3:amd64 (0.6.1-2.1+deb11u2) ...
Selecting previously unselected package libxslt1.1:amd64.
Preparing to unpack .../21-libxslt1.1_1.1.34-4+deb11u1_amd64.deb ...
Unpacking libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Selecting previously unselected package libz3-dev:amd64.
Preparing to unpack .../22-libz3-dev_4.8.10-1_amd64.deb ...
Unpacking libz3-dev:amd64 (4.8.10-1) ...
Selecting previously unselected package llvm-9-runtime.
Preparing to unpack .../23-llvm-9-runtime_1%3a9.0.1-16.1_amd64.deb ...
Unpacking llvm-9-runtime (1:9.0.1-16.1) ...
Selecting previously unselected package llvm-9.
Preparing to unpack .../24-llvm-9_1%3a9.0.1-16.1_amd64.deb ...
Unpacking llvm-9 (1:9.0.1-16.1) ...
Selecting previously unselected package python3-pygments.
Preparing to unpack .../25-python3-pygments_2.7.1+dfsg-2.1_all.deb ...
Unpacking python3-pygments (2.7.1+dfsg-2.1) ...
Selecting previously unselected package python3-yaml.
Preparing to unpack .../26-python3-yaml_5.3.1-5_amd64.deb ...
Unpacking python3-yaml (5.3.1-5) ...
Selecting previously unselected package llvm-9-tools.
Preparing to unpack .../27-llvm-9-tools_1%3a9.0.1-16.1_amd64.deb ...
Unpacking llvm-9-tools (1:9.0.1-16.1) ...
Selecting previously unselected package llvm-9-dev.
Preparing to unpack .../28-llvm-9-dev_1%3a9.0.1-16.1_amd64.deb ...
Unpacking llvm-9-dev (1:9.0.1-16.1) ...

Selecting previously unselected package sphinx-rtd-theme-common................................................................................
Preparing to unpack .../29-sphinx-rtd-theme-common_0.5.1+dfsg-1_all.deb ...
Unpacking sphinx-rtd-theme-common (0.5.1+dfsg-1) ...
Selecting previously unselected package numba-doc.
Preparing to unpack .../30-numba-doc_0.52.0-4_all.deb ...
Unpacking numba-doc (0.52.0-4) ...
Selecting previously unselected package ttf-bitstream-vera.
Preparing to unpack .../31-ttf-bitstream-vera_1.10-8.1_all.deb ...
Unpacking ttf-bitstream-vera (1.10-8.1) ...
Selecting previously unselected package python-matplotlib-data.
Preparing to unpack .../32-python-matplotlib-data_3.3.4-1_all.deb ...
Unpacking python-matplotlib-data (3.3.4-1) ...
Selecting previously unselected package python-odf-doc.
Preparing to unpack .../33-python-odf-doc_1.4.1-1_all.deb ...
Unpacking python-odf-doc (1.4.1-1) ...
Selecting previously unselected package python3-defusedxml.
Preparing to unpack .../34-python3-defusedxml_0.6.0-2_all.deb ...
Unpacking python3-defusedxml (0.6.0-2) ...
Selecting previously unselected package python3-odf.
Preparing to unpack .../35-python3-odf_1.4.1-1_all.deb ...
Unpacking python3-odf (1.4.1-1) ...
Selecting previously unselected package python-odf-tools.
Preparing to unpack .../36-python-odf-tools_1.4.1-1_all.deb ...
Unpacking python-odf-tools (1.4.1-1) ...
Selecting previously unselected package python-tables-data.
Preparing to unpack .../37-python-tables-data_3.6.1-3_all.deb ...
Unpacking python-tables-data (3.6.1-3) ...
Selecting previously unselected package python3-attr.
Preparing to unpack .../38-python3-attr_20.3.0-1_all.deb ...
Unpacking python3-attr (20.3.0-1) ...
Selecting previously unselected package python3-numpy.
Preparing to unpack .../39-python3-numpy_1%3a1.19.5-1_amd64.deb ...
Unpacking python3-numpy (1:1.19.5-1) ...
Selecting previously unselected package python3-bottleneck.
Preparing to unpack .../40-python3-bottleneck_1.2.1+ds1-2+b4_amd64.deb ...
Unpacking python3-bottleneck (1.2.1+ds1-2+b4) ...
Selecting previously unselected package python3-soupsieve.
Preparing to unpack .../41-python3-soupsieve_2.2.1-1_all.deb ...
Unpacking python3-soupsieve (2.2.1-1) ...
Selecting previously unselected package python3-bs4.
Preparing to unpack .../42-python3-bs4_4.9.3-1_all.deb ...
Unpacking python3-bs4 (4.9.3-1) ...
Selecting previously unselected package python3-cycler.
Preparing to unpack .../43-python3-cycler_0.10.0-3_all.deb ...
Unpacking python3-cycler (0.10.0-3) ...
Selecting previously unselected package python3-dateutil.
Preparing to unpack .../44-python3-dateutil_2.8.1-6_all.deb ...
Unpacking python3-dateutil (2.8.1-6) ...
Selecting previously unselected package python3-decorator.
Preparing to unpack .../45-python3-decorator_4.4.2-2_all.deb ...
Unpacking python3-decorator (4.4.2-2) ...
Selecting previously unselected package python3-et-xmlfile.
Preparing to unpack .../46-python3-et-xmlfile_1.0.1-2.1_all.deb ...
Unpacking python3-et-xmlfile (1.0.1-2.1) ...
Selecting previously unselected package python3-webencodings.
Preparing to unpack .../47-python3-webencodings_0.5.1-2_all.deb ...
Unpacking python3-webencodings (0.5.1-2) ...
Selecting previously unselected package python3-html5lib.
Preparing to unpack .../48-python3-html5lib_1.1-3_all.deb ...
Unpacking python3-html5lib (1.1-3) ...
Selecting previously unselected package python3-more-itertools.
Preparing to unpack .../49-python3-more-itertools_4.2.0-3_all.deb ...
Unpacking python3-more-itertools (4.2.0-3) ...
Selecting previously unselected package python3-zipp.
Preparing to unpack .../50-python3-zipp_1.0.0-3_all.deb ...
Unpacking python3-zipp (1.0.0-3) ...
Selecting previously unselected package python3-importlib-metadata.
Preparing to unpack .../51-python3-importlib-metadata_1.6.0-2_all.deb ...
Unpacking python3-importlib-metadata (1.6.0-2) ...
Selecting previously unselected package python3-iniconfig.
Preparing to unpack .../52-python3-iniconfig_1.1.1-1_all.deb ...
Unpacking python3-iniconfig (1.1.1-1) ...
Selecting previously unselected package python3-jdcal.
Preparing to unpack .../53-python3-jdcal_1.0-1.3_all.deb ...
Unpacking python3-jdcal (1.0-1.3) ...
Selecting previously unselected package python3-markupsafe.
Preparing to unpack .../54-python3-markupsafe_1.1.1-1+b3_amd64.deb ...
Unpacking python3-markupsafe (1.1.1-1+b3) ...
Selecting previously unselected package python3-jinja2.
Preparing to unpack .../55-python3-jinja2_2.11.3-1_all.deb ...
Unpacking python3-jinja2 (2.11.3-1) ...
Selecting previously unselected package python3-kiwisolver.
Preparing to unpack .../56-python3-kiwisolver_1.3.1-1+b1_amd64.deb ...
Unpacking python3-kiwisolver (1.3.1-1+b1) ...
Selecting previously unselected package python3-llvmlite.
Preparing to unpack .../57-python3-llvmlite_0.35.0-3_amd64.deb ...
Unpacking python3-llvmlite (0.35.0-3) ...
Selecting previously unselected package python3-lxml:amd64.
Preparing to unpack .../58-python3-lxml_4.6.3+dfsg-0.1+deb11u1_amd64.deb ...
Unpacking python3-lxml:amd64 (4.6.3+dfsg-0.1+deb11u1) ...
Selecting previously unselected package python3-pyparsing.
Preparing to unpack .../59-python3-pyparsing_2.4.7-1_all.deb ...
Unpacking python3-pyparsing (2.4.7-1) ...
Selecting previously unselected package python3-pil:amd64.
Preparing to unpack .../60-python3-pil_8.1.2+dfsg-0.3+deb11u2_amd64.deb ...
Unpacking python3-pil:amd64 (8.1.2+dfsg-0.3+deb11u2) ...
Selecting previously unselected package python3-matplotlib.
Preparing to unpack .../61-python3-matplotlib_3.3.4-1_amd64.deb ...
Unpacking python3-matplotlib (3.3.4-1) ...
Selecting previously unselected package python3-scipy.
Preparing to unpack .../62-python3-scipy_1.6.0-2_amd64.deb ...
Unpacking python3-scipy (1.6.0-2) ...
Selecting previously unselected package python3-numba.
Preparing to unpack .../63-python3-numba_0.52.0-4_amd64.deb ...
Unpacking python3-numba (0.52.0-4) ...
Selecting previously unselected package python3-numexpr.
Preparing to unpack .../64-python3-numexpr_2.7.2-2_amd64.deb ...
Unpacking python3-numexpr (2.7.2-2) ...
Selecting previously unselected package python3-olefile.
Preparing to unpack .../65-python3-olefile_0.46-3_all.deb ...
Unpacking python3-olefile (0.46-3) ...
Selecting previously unselected package python3-openpyxl.
Preparing to unpack .../66-python3-openpyxl_3.0.3-1_all.deb ...
Unpacking python3-openpyxl (3.0.3-1) ...
Selecting previously unselected package python3-packaging.
Preparing to unpack .../67-python3-packaging_20.9-2_all.deb ...
Unpacking python3-packaging (20.9-2) ...
Selecting previously unselected package python3-tz.
Preparing to unpack .../68-python3-tz_2021.1-1_all.deb ...
Unpacking python3-tz (2021.1-1) ...
Selecting previously unselected package python3-pandas-lib:amd64.
Preparing to unpack .../69-python3-pandas-lib_1.1.5+dfsg-2_amd64.deb ...
Unpacking python3-pandas-lib:amd64 (1.1.5+dfsg-2) ...
Selecting previously unselected package python3-pandas.
Preparing to unpack .../70-python3-pandas_1.1.5+dfsg-2_all.deb ...
Unpacking python3-pandas (1.1.5+dfsg-2) ...
Selecting previously unselected package python3-pluggy.
Preparing to unpack .../71-python3-pluggy_0.13.0-6_all.deb ...
Unpacking python3-pluggy (0.13.0-6) ...
Selecting previously unselected package python3-py.
Preparing to unpack .../72-python3-py_1.10.0-1_all.deb ...
Unpacking python3-py (1.10.0-1) ...
Selecting previously unselected package python3-toml.
Preparing to unpack .../73-python3-toml_0.10.1-1_all.deb ...
Unpacking python3-toml (0.10.1-1) ...
Selecting previously unselected package python3-pytest.
Preparing to unpack .../74-python3-pytest_6.0.2-2_all.deb ...
Unpacking python3-pytest (6.0.2-2) ...
Selecting previously unselected package python3-tables-lib.
Preparing to unpack .../75-python3-tables-lib_3.6.1-3_amd64.deb ...
Unpacking python3-tables-lib (3.6.1-3) ...
Selecting previously unselected package python3-tables.
Preparing to unpack .../76-python3-tables_3.6.1-3_all.deb ...
Unpacking python3-tables (3.6.1-3) ...
Selecting previously unselected package python3-xlwt.
Preparing to unpack .../77-python3-xlwt_1.3.0-3_all.deb ...
Unpacking python3-xlwt (1.3.0-3) ...
Setting up python3-more-itertools (4.2.0-3) ...
Setting up python3-iniconfig (1.1.1-1) ...
Setting up python3-attr (20.3.0-1) ...
Setting up libz3-dev:amd64 (4.8.10-1) ...
Setting up libncurses-dev:amd64 (6.2+20201114-2+deb11u2) ...
Setting up ttf-bitstream-vera (1.10-8.1) ...
Setting up python3-py (1.10.0-1) ...
Setting up python3-jdcal (1.0-1.3) ...
Setting up python3-defusedxml (0.6.0-2) ...
Setting up fonts-lyx (2.3.6-1) ...
Setting up python3-olefile (0.46-3) ...
Setting up libwebpdemux2:amd64 (0.6.1-2.1+deb11u2) ...
Setting up libtbb2:amd64 (2020.3-1) ...
Setting up python3-yaml (5.3.1-5) ...
Setting up liblzo2-2:amd64 (2.10-2) ...
Setting up python3-zipp (1.0.0-3) ...
Setting up libffi-dev:amd64 (3.3-6) ...
Setting up python3-markupsafe (1.1.1-1+b3) ...
Setting up python3-webencodings (0.5.1-2) ...
Setting up python3-tz (2021.1-1) ...
Setting up python3-decorator (4.4.2-2) ...
Setting up python3-jinja2 (2.11.3-1) ...
Setting up python3-pygments (2.7.1+dfsg-2.1) ...
Setting up libpfm4:amd64 (4.11.1+git32-gd0b85fb-1) ...
Setting up libjs-jquery-ui (1.12.1+dfsg-8+deb11u2) ...
Setting up python3-pyparsing (2.4.7-1) ...
Setting up python3-cycler (0.10.0-3) ...
Setting up python-odf-doc (1.4.1-1) ...
Setting up libimagequant0:amd64 (2.12.2-1.1) ...
Setting up python3-kiwisolver (1.3.1-1+b1) ...
Setting up python3-xlwt (1.3.0-3) ...
Setting up binfmt-support (2.2.1-1+deb11u1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/binfmt-support.service → /lib/systemd/system/binfmt-support.service.
Setting up python3-html5lib (1.1-3) ...
Setting up python3-numpy (1:1.19.5-1) ...
Setting up python3-toml (0.10.1-1) ...
Setting up libxslt1.1:amd64 (1.1.34-4+deb11u1) ...
Setting up libblosc1 (1.20.1+ds1-2) ...
Setting up python3-et-xmlfile (1.0.1-2.1) ...
Setting up python3-dateutil (2.8.1-6) ...
Setting up python-matplotlib-data (3.3.4-1) ...
Setting up libwebpmux3:amd64 (0.6.1-2.1+deb11u2) ...
Setting up mailcap (3.69) ...
Setting up python3-soupsieve (2.2.1-1) ...
Setting up python-tables-data (3.6.1-3) ...
Setting up fonts-font-awesome (5.0.10+really4.7.0~dfsg-4.1) ...
Setting up sphinx-rtd-theme-common (0.5.1+dfsg-1) ...
Setting up libllvm9:amd64 (1:9.0.1-16.1) ...
Setting up libjs-underscore (1.9.1~dfsg-3) ...
Setting up liblbfgsb0:amd64 (3.0+dfsg.3-9) ...
Setting up python3-odf (1.4.1-1) ...
Setting up libtinfo-dev:amd64 (6.2+20201114-2+deb11u2) ...
Setting up python3-scipy (1.6.0-2) ...
Setting up mime-support (3.66) ...
Setting up python3-importlib-metadata (1.6.0-2) ...
Setting up llvm-9-tools (1:9.0.1-16.1) ...
Setting up python3-tables-lib (3.6.1-3) ...
Setting up libclang-cpp9 (1:9.0.1-16.1) ...
Setting up python3-pandas-lib:amd64 (1.1.5+dfsg-2) ...
Setting up python-odf-tools (1.4.1-1) ...
Setting up python3-bs4 (4.9.3-1) ...
Setting up python3-pil:amd64 (8.1.2+dfsg-0.3+deb11u2) ...
Setting up python3-packaging (20.9-2) ...
Setting up python3-pandas (1.1.5+dfsg-2) ...
Setting up python3-bottleneck (1.2.1+ds1-2+b4) ...
Setting up python3-numexpr (2.7.2-2) ...
Setting up llvm-9-runtime (1:9.0.1-16.1) ...
Setting up libjs-sphinxdoc (3.4.3-2) ...
Setting up python3-pluggy (0.13.0-6) ...
Setting up python3-lxml:amd64 (4.6.3+dfsg-0.1+deb11u1) ...
Setting up numba-doc (0.52.0-4) ...
Setting up python3-matplotlib (3.3.4-1) ...
Setting up python3-pytest (6.0.2-2) ...
Setting up python3-tables (3.6.1-3) ...
Setting up llvm-9 (1:9.0.1-16.1) ...
Setting up python3-openpyxl (3.0.3-1) ...
Setting up llvm-9-dev (1:9.0.1-16.1) ...
Setting up python3-llvmlite (0.35.0-3) ...
Setting up python3-numba (0.52.0-4) ...
update-alternatives: using /usr/share/python3-numba/numba to provide /usr/bin/numba (numba) in auto mode
Processing triggers for fontconfig (2.13.1-4.2) ...
Processing triggers for libc-bin (2.31-13+deb11u10) ...
ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_ops_train.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_cnn_infer.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_adv_train.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_adv_infer.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_ops_infer.so.8 is not a symbolic link

ldconfig: /usr/local/cuda-11.3/targets/x86_64-linux/lib/libcudnn_cnn_train.so.8 is not a symbolic link

ldconfig: /lib/libnvonnxparser.so.8 is not a symbolic link

ldconfig: /lib/libnvinfer.so.8 is not a symbolic link

ldconfig: /lib/libnvinfer_plugin.so.8 is not a symbolic link

ldconfig: /lib/libnvparsers.so.8 is not a symbolic link

Processing triggers for man-db (2.9.4-2) ...
Processing triggers for install-info (6.7.0.dfsg.2-6) ...
Found existing installation: openpyxl 3.0.10
Uninstalling openpyxl-3.0.10:
  Would remove:
    /home/jupyter/.local/lib/python3.10/site-packages/openpyxl-3.0.10.dist-info/*
    /home/jupyter/.local/lib/python3.10/site-packages/openpyxl/*
Proceed (Y/n)? Y
  Successfully uninstalled openpyxl-3.0.10
Collecting openpyxl
  Downloading openpyxl-3.1.5-py2.py3-none-any.whl.metadata (2.5 kB)
Requirement already satisfied: et-xmlfile in /home/jupyter/.local/lib/python3.10/site-packages (from openpyxl) (1.1.0)
Downloading openpyxl-3.1.5-py2.py3-none-any.whl (250 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 250.9/250.9 kB 2.1 MB/s eta 0:00:00
Installing collected packages: openpyxl
Successfully installed openpyxl-3.1.5
(base) jupyter@instance-20240630-193809:~/training-data-analyst/self-paced-labs/vertex-ai/vertex-ai-qwikstart$ history 
    1  cd training-data-analyst/self-paced-labs/vertex-ai/vertex-ai-qwikstart
    2  pip3 install --user -r requirements.txt
    3  sudo apt -y install python3-pandas
    4  pip uninstall openpyxl
    5  pip install openpyxl
    6  history 
(base) jupyter@instance-20240630-193809:~/training-data-analyst/self-paced-labs/vertex-ai/vertex-ai-qwikstart$ 
```



In this lab, you ran a machine learning experimentation workflow using  Google Cloud BigQuery for data storage and analysis and Vertex AI  machine learning services to train and deploy a TensorFlow model to  predict customer lifetime value. You progressed from training a  TensorFlow model locally to training on the cloud with Vertex AI and  leveraged several new unified platform capabilities such as Vertex  TensorBoard and prediction feature attributions.
