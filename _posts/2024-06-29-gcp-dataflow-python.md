---

layout: single
title:  "Dataflow: Qwik Start - Python"
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
# Dataflow: Qwik Start - Python

The Apache Beam SDK is an open source programming model for data  pipelines. In Google Cloud, you can define a pipeline with an Apache  Beam program and then use Dataflow to run your pipeline.

In this lab, you set up your Python development environment for  Dataflow (using the Apache Beam SDK for Python) and run an example  Dataflow pipeline.

- Create a Cloud Storage bucket to store results of a Dataflow pipeline
- Install the Apache Beam SDK for Python
- Run a Dataflow pipeline remotely

### Ensure that the Dataflow API is successfully enabled

To ensure access to the necessary API, restart the connection to the Dataflow API.

1. In the Cloud Console, enter "Dataflow API" in the top search bar. Click on the result for **Dataflow API**.
2. Click **Manage**.
3. Click **Disable API**.

If asked to confirm, click **Disable**.

1. Click **Enable**.

When the API has been enabled again, the page will show the option to disable.

## Create a Cloud Storage bucket

When you run a pipeline using Dataflow, your results are stored in a  Cloud Storage bucket. In this task, you create a Cloud Storage bucket  for the results of the pipeline that you run in a later task.

1. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Cloud Storage** > **Buckets**.
2. Click **Create bucket**.
3. In the **Create bucket** dialog, specify the following attributes:

- ***Name***: To ensure a unique bucket name, use the following name: **-bucket**. Note that this name does not include sensitive information in the  bucket name, as the bucket namespace is global and publicly visible.
- ***Location type***: Multi-region
- ***Location***: `us`
- A location where bucket data will be stored.

1. Click **Create**.
2. If Prompted Public access will be prevented, click **Confirm**.

## Install the Apache Beam SDK for Python

1. To ensure that you use a supported Python version, begin by running the `Python3.9` Docker Image:

   ```sh
   student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-01-618fd45678b4)$ docker run -it -e DEVSHELL_PROJECT_ID=$DEVSHELL_PROJECT_ID python:3.9 /bin/bash
   Unable to find image 'python:3.9' locally
   3.9: Pulling from library/python
   fea1432adf09: Pull complete 
   5651b5803b18: Pull complete 
   3873416e6a33: Pull complete 
   8a142b8b0e69: Pull complete 
   1cf9c810f5bd: Pull complete 
   af54864fd69d: Pull complete 
   092662c12da2: Pull complete 
   00113f908027: Pull complete 
   Digest: sha256:e298e2e898691a938073f670dac8ef1a551c83344b67b5d8e32d1fbc8e0b57f8
   Status: Downloaded newer image for python:3.9
   root@7c2c11083d7b:/# 
   ```

   

2. This command pulls a Docker container with the latest stable version  of Python 3.9 and then opens up a command shell for you to run the  following commands inside your container.

   1. After the container is running, install the latest version of the  Apache Beam SDK for Python by running the following command from a  virtual environment:

   ```sh
   root@7c2c11083d7b:/# pip install 'apache-beam[gcp]'==2.42.0
   Collecting apache-beam[gcp]==2.42.0
     Downloading apache_beam-2.42.0-cp39-cp39-manylinux2010_x86_64.whl (12.1 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.1/12.1 MB 63.2 MB/s eta 0:00:00
   Collecting cloudpickle~=2.1.0
     Downloading cloudpickle-2.1.0-py3-none-any.whl (25 kB)
   Collecting zstandard<1,>=0.18.0
     Downloading zstandard-0.22.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (5.4 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.4/5.4 MB 70.7 MB/s eta 0:00:00
   Collecting hdfs<3.0.0,>=2.1.0
     Downloading hdfs-2.7.3.tar.gz (43 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 43.5/43.5 kB 5.9 MB/s eta 0:00:00
     Preparing metadata (setup.py) ... done
   Collecting pymongo<4.0.0,>=3.8.0
     Downloading pymongo-3.13.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (515 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 515.5/515.5 kB 40.1 MB/s eta 0:00:00
   Collecting fastavro<2,>=0.23.6
     Downloading fastavro-1.9.4-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (3.1 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.1/3.1 MB 75.6 MB/s eta 0:00:00
   Collecting typing-extensions>=3.7.0
     Downloading typing_extensions-4.12.2-py3-none-any.whl (37 kB)
   Collecting pydot<2,>=1.2.0
     Downloading pydot-1.4.2-py2.py3-none-any.whl (21 kB)
   Collecting dill<0.3.2,>=0.3.1.1
     Downloading dill-0.3.1.1.tar.gz (151 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 152.0/152.0 kB 21.4 MB/s eta 0:00:00
     Preparing metadata (setup.py) ... done
   Collecting regex>=2020.6.8
     Downloading regex-2024.5.15-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (774 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 774.6/774.6 kB 53.1 MB/s eta 0:00:00
   Collecting numpy<1.23.0,>=1.14.3
     Downloading numpy-1.22.4-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (16.8 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 16.8/16.8 MB 48.0 MB/s eta 0:00:00
   Collecting grpcio!=1.48.0,<2,>=1.33.1
     Downloading grpcio-1.64.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (5.6 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.6/5.6 MB 78.4 MB/s eta 0:00:00
   Collecting protobuf<4,>=3.12.2
     Downloading protobuf-3.20.3-cp39-cp39-manylinux_2_5_x86_64.manylinux1_x86_64.whl (1.0 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.0/1.0 MB 58.0 MB/s eta 0:00:00
   Collecting pyarrow<8.0.0,>=0.15.1
     Downloading pyarrow-7.0.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26.7 MB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 26.7/26.7 MB 39.0 MB/s eta 0:00:00
   Collecting orjson<4.0
     Downloading orjson-3.10.5-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (144 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 144.8/144.8 kB 20.4 MB/s eta 0:00:00
   Collecting httplib2<0.21.0,>=0.8
     Downloading httplib2-0.20.4-py3-none-any.whl (96 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 13.6 MB/s eta 0:00:00
   Collecting python-dateutil<3,>=2.8.0
     Downloading python_dateutil-2.9.0.post0-py2.py3-none-any.whl (229 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 229.9/229.9 kB 28.1 MB/s eta 0:00:00
   Collecting requests<3.0.0,>=2.24.0
     Downloading requests-2.32.3-py3-none-any.whl (64 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 64.9/64.9 kB 9.5 MB/s eta 0:00:00
   Collecting pytz>=2018.3
     Downloading pytz-2024.1-py2.py3-none-any.whl (505 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 505.5/505.5 kB 44.0 MB/s eta 0:00:00
   Collecting proto-plus<2,>=1.7.1
     Downloading proto_plus-1.24.0-py3-none-any.whl (50 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 50.1/50.1 kB 7.7 MB/s eta 0:00:00
   Collecting crcmod<2.0,>=1.7
     Downloading crcmod-1.7.tar.gz (89 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 89.7/89.7 kB 13.2 MB/s eta 0:00:00
     Preparing metadata (setup.py) ... done
   Collecting grpcio-gcp<1,>=0.2.2
     Downloading grpcio_gcp-0.2.2-py2.py3-none-any.whl (9.4 kB)
   Collecting cachetools<5,>=3.1.0
     Downloading cachetools-4.2.4-py3-none-any.whl (10 kB)
   Collecting google-cloud-dlp<4,>=3.0.0
     Downloading google_cloud_dlp-3.18.0-py2.py3-none-any.whl (180 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 180.3/180.3 kB 18.7 MB/s eta 0:00:00
   Collecting google-auth<3,>=1.18.0
     Downloading google_auth-2.30.0-py2.py3-none-any.whl (193 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 193.7/193.7 kB 25.8 MB/s eta 0:00:00
   Collecting google-auth-httplib2<0.2.0,>=0.1.0
     Downloading google_auth_httplib2-0.1.1-py2.py3-none-any.whl (9.3 kB)
   Collecting google-api-core!=2.8.2,<3
     Downloading google_api_core-2.19.1-py3-none-any.whl (139 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 139.4/139.4 kB 19.4 MB/s eta 0:00:00
   Collecting google-cloud-vision<2,>=0.38.0
     Downloading google_cloud_vision-1.0.2-py2.py3-none-any.whl (435 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 435.1/435.1 kB 40.8 MB/s eta 0:00:00
   Collecting google-cloud-pubsub<3,>=2.1.0
     Downloading google_cloud_pubsub-2.21.5-py2.py3-none-any.whl (273 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 273.2/273.2 kB 31.5 MB/s eta 0:00:00
   Collecting google-cloud-datastore<2,>=1.8.0
     Downloading google_cloud_datastore-1.15.5-py2.py3-none-any.whl (134 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 134.2/134.2 kB 15.9 MB/s eta 0:00:00
   Collecting google-cloud-pubsublite<2,>=1.2.0
     Downloading google_cloud_pubsublite-1.10.0-py2.py3-none-any.whl (299 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 299.2/299.2 kB 31.4 MB/s eta 0:00:00
   Collecting google-cloud-bigtable<2,>=0.31.1
     Downloading google_cloud_bigtable-1.7.3-py2.py3-none-any.whl (268 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 268.7/268.7 kB 30.7 MB/s eta 0:00:00
   Collecting google-cloud-language<2,>=1.3.0
     Downloading google_cloud_language-1.3.2-py2.py3-none-any.whl (83 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 83.6/83.6 kB 12.0 MB/s eta 0:00:00
   Collecting google-cloud-spanner<2,>=1.13.0
     Downloading google_cloud_spanner-1.19.3-py2.py3-none-any.whl (255 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 255.6/255.6 kB 27.7 MB/s eta 0:00:00
   Collecting google-cloud-bigquery-storage<2.14,>=2.6.3
     Downloading google_cloud_bigquery_storage-2.13.2-py2.py3-none-any.whl (180 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 180.2/180.2 kB 17.8 MB/s eta 0:00:00
   Collecting google-cloud-bigquery<3,>=1.6.0
     Downloading google_cloud_bigquery-2.34.4-py2.py3-none-any.whl (206 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 206.6/206.6 kB 25.5 MB/s eta 0:00:00
   Collecting google-cloud-core<3,>=0.28.1
     Downloading google_cloud_core-2.4.1-py2.py3-none-any.whl (29 kB)
   Collecting google-cloud-videointelligence<2,>=1.8.0
     Downloading google_cloud_videointelligence-1.16.3-py2.py3-none-any.whl (183 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 183.9/183.9 kB 23.4 MB/s eta 0:00:00
   Collecting google-cloud-recommendations-ai<0.8.0,>=0.1.0
     Downloading google_cloud_recommendations_ai-0.7.1-py2.py3-none-any.whl (148 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 148.2/148.2 kB 17.8 MB/s eta 0:00:00
   Collecting google-apitools<0.5.32,>=0.5.31
     Downloading google-apitools-0.5.31.tar.gz (173 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 173.5/173.5 kB 19.5 MB/s eta 0:00:00
     Preparing metadata (setup.py) ... done
   Collecting googleapis-common-protos<2.0.dev0,>=1.56.2
     Downloading googleapis_common_protos-1.63.2-py2.py3-none-any.whl (220 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 220.0/220.0 kB 26.3 MB/s eta 0:00:00
   Collecting fasteners>=0.14
     Downloading fasteners-0.19-py3-none-any.whl (18 kB)
   Collecting oauth2client>=1.4.12
     Downloading oauth2client-4.1.3-py2.py3-none-any.whl (98 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 98.2/98.2 kB 14.5 MB/s eta 0:00:00
   Collecting six>=1.12.0
     Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
   Collecting rsa<5,>=3.1.4
     Downloading rsa-4.9-py3-none-any.whl (34 kB)
   Collecting pyasn1-modules>=0.2.1
     Downloading pyasn1_modules-0.4.0-py3-none-any.whl (181 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 181.2/181.2 kB 22.9 MB/s eta 0:00:00
   Collecting google-resumable-media<3.0dev,>=0.6.0
     Downloading google_resumable_media-2.7.1-py2.py3-none-any.whl (81 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 81.2/81.2 kB 12.4 MB/s eta 0:00:00
   Collecting packaging<22.0dev,>=14.3
     Downloading packaging-21.3-py3-none-any.whl (40 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 40.8/40.8 kB 4.3 MB/s eta 0:00:00
   Collecting grpc-google-iam-v1<0.13dev,>=0.12.3
     Downloading grpc_google_iam_v1-0.12.7-py2.py3-none-any.whl (26 kB)
   Collecting grpcio-status>=1.33.2
     Downloading grpcio_status-1.64.1-py3-none-any.whl (14 kB)
   Collecting overrides<8.0.0,>=6.0.1
     Downloading overrides-7.7.0-py3-none-any.whl (17 kB)
   Collecting docopt
     Downloading docopt-0.6.2.tar.gz (25 kB)
     Preparing metadata (setup.py) ... done
   Collecting pyparsing!=3.0.0,!=3.0.1,!=3.0.2,!=3.0.3,<4,>=2.4.2
     Downloading pyparsing-3.1.2-py3-none-any.whl (103 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 103.2/103.2 kB 14.1 MB/s eta 0:00:00
   Collecting urllib3<3,>=1.21.1
     Downloading urllib3-2.2.2-py3-none-any.whl (121 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 121.4/121.4 kB 12.9 MB/s eta 0:00:00
   Collecting charset-normalizer<4,>=2
     Downloading charset_normalizer-3.3.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (142 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 142.3/142.3 kB 19.9 MB/s eta 0:00:00
   Collecting certifi>=2017.4.17
     Downloading certifi-2024.6.2-py3-none-any.whl (164 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 164.4/164.4 kB 21.5 MB/s eta 0:00:00
   Collecting idna<4,>=2.5
     Downloading idna-3.7-py3-none-any.whl (66 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 66.8/66.8 kB 9.6 MB/s eta 0:00:00
   Collecting google-crc32c<2.0dev,>=1.0
     Downloading google_crc32c-1.5.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (32 kB)
   Collecting grpcio-status>=1.33.2
     Downloading grpcio_status-1.64.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.63.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.62.2-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.62.1-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.62.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.60.1-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.60.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.59.3-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.59.2-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.59.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.58.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.57.0-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.56.2-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.56.0-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.55.3-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.54.3-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.54.2-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.54.0-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.53.2-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.53.1-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.53.0-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.51.3-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.51.1-py3-none-any.whl (5.1 kB)
     Downloading grpcio_status-1.50.0-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.49.1-py3-none-any.whl (14 kB)
     Downloading grpcio_status-1.48.2-py3-none-any.whl (14 kB)
   Collecting pyasn1>=0.1.7
     Downloading pyasn1-0.6.0-py2.py3-none-any.whl (85 kB)
        ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 85.3/85.3 kB 12.4 MB/s eta 0:00:00
   Building wheels for collected packages: crcmod, dill, google-apitools, hdfs, docopt
     Building wheel for crcmod (setup.py) ... done
     Created wheel for crcmod: filename=crcmod-1.7-cp39-cp39-linux_x86_64.whl size=30719 sha256=dbe7aa364526e3cc4735f660ea990d8d3e2782e1f2c5398a49f9651c31e588bf
     Stored in directory: /root/.cache/pip/wheels/4a/6c/a6/ffdd136310039bf226f2707a9a8e6857be7d70a3fc061f6b36
     Building wheel for dill (setup.py) ... done
     Created wheel for dill: filename=dill-0.3.1.1-py3-none-any.whl size=78542 sha256=926a861f3902f88b9c9e4e5fd9f3813ea8fed5307043cb0eb886148d25624520
     Stored in directory: /root/.cache/pip/wheels/4f/0b/ce/75d96dd714b15e51cb66db631183ea3844e0c4a6d19741a149
     Building wheel for google-apitools (setup.py) ... done
     Created wheel for google-apitools: filename=google_apitools-0.5.31-py3-none-any.whl size=131031 sha256=d66fb7289a8a10bb3b5ba082011a6bf377e706396dde27a51b7ca3a4c74f6ab9
     Stored in directory: /root/.cache/pip/wheels/6c/f8/60/b9e91899dbaf25b6314047d3daee379bdd8d61b1dc3fd5ec7f
     Building wheel for hdfs (setup.py) ... done
     Created wheel for hdfs: filename=hdfs-2.7.3-py3-none-any.whl size=34342 sha256=5d5f53599e41443325f042456ac2ede461dcedce2071a3521edf616815115bcb
     Stored in directory: /root/.cache/pip/wheels/05/6f/21/aa8d233f90da3017b4ef7c61829589dc267402d376dd3efcf5
     Building wheel for docopt (setup.py) ... done
     Created wheel for docopt: filename=docopt-0.6.2-py2.py3-none-any.whl size=13722 sha256=bd97b2bf546b5b5b5d3cb251a7e0820bb26c12d8fe73f42864dbe747389581ff
     Stored in directory: /root/.cache/pip/wheels/70/4a/46/1309fc853b8d395e60bafaf1b6df7845bdd82c95fd59dd8d2b
   Successfully built crcmod dill google-apitools hdfs docopt
   Installing collected packages: pytz, docopt, crcmod, zstandard, urllib3, typing-extensions, six, regex, pyparsing, pymongo, pyasn1, protobuf, overrides, orjson, numpy, idna, grpcio, google-crc32c, fasteners, fastavro, dill, cloudpickle, charset-normalizer, certifi, cachetools, rsa, requests, python-dateutil, pydot, pyasn1-modules, pyarrow, proto-plus, packaging, httplib2, grpcio-gcp, googleapis-common-protos, google-resumable-media, oauth2client, hdfs, grpcio-status, google-auth, grpc-google-iam-v1, google-auth-httplib2, google-apitools, google-api-core, apache-beam, google-cloud-core, google-cloud-vision, google-cloud-videointelligence, google-cloud-spanner, google-cloud-recommendations-ai, google-cloud-pubsub, google-cloud-language, google-cloud-dlp, google-cloud-datastore, google-cloud-bigtable, google-cloud-bigquery-storage, google-cloud-bigquery, google-cloud-pubsublite
   Successfully installed apache-beam-2.42.0 cachetools-4.2.4 certifi-2024.6.2 charset-normalizer-3.3.2 cloudpickle-2.1.0 crcmod-1.7 dill-0.3.1.1 docopt-0.6.2 fastavro-1.9.4 fasteners-0.19 google-api-core-2.19.1 google-apitools-0.5.31 google-auth-2.30.0 google-auth-httplib2-0.1.1 google-cloud-bigquery-2.34.4 google-cloud-bigquery-storage-2.13.2 google-cloud-bigtable-1.7.3 google-cloud-core-2.4.1 google-cloud-datastore-1.15.5 google-cloud-dlp-3.18.0 google-cloud-language-1.3.2 google-cloud-pubsub-2.21.5 google-cloud-pubsublite-1.10.0 google-cloud-recommendations-ai-0.7.1 google-cloud-spanner-1.19.3 google-cloud-videointelligence-1.16.3 google-cloud-vision-1.0.2 google-crc32c-1.5.0 google-resumable-media-2.7.1 googleapis-common-protos-1.63.2 grpc-google-iam-v1-0.12.7 grpcio-1.64.1 grpcio-gcp-0.2.2 grpcio-status-1.48.2 hdfs-2.7.3 httplib2-0.20.4 idna-3.7 numpy-1.22.4 oauth2client-4.1.3 orjson-3.10.5 overrides-7.7.0 packaging-21.3 proto-plus-1.24.0 protobuf-3.20.3 pyarrow-7.0.0 pyasn1-0.6.0 pyasn1-modules-0.4.0 pydot-1.4.2 pymongo-3.13.0 pyparsing-3.1.2 python-dateutil-2.9.0.post0 pytz-2024.1 regex-2024.5.15 requests-2.32.3 rsa-4.9 six-1.16.0 typing-extensions-4.12.2 urllib3-2.2.2 zstandard-0.22.0
   WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
   
   [notice] A new release of pip is available: 23.0.1 -> 24.1.1
   [notice] To update, run: pip install --upgrade pip
   root@7c2c11083d7b:/# 
   ```

   You will see some warnings returned that are related to dependencies. It is safe to ignore them for this lab.

   1. Run the `wordcount.py` example locally by running the following command:

   ```sh
   root@7c2c11083d7b:/# python -m apache_beam.examples.wordcount --output OUTPUT_FILE
   INFO:root:Missing pipeline option (runner). Executing pipeline using the default runner: DirectRunner.
   INFO:apache_beam.internal.gcp.auth:Setting socket default timeout to 60 seconds.
   INFO:apache_beam.internal.gcp.auth:socket default timeout is 60.0 seconds.
   WARNING:google.auth._default:No project ID could be determined. Consider running `gcloud config set project` or setting the GOOGLE_CLOUD_PROJECT environment variable
   INFO:root:Default Python SDK image for environment is apache/beam_python3.9_sdk:2.42.0
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function annotate_downstream_side_inputs at 0x7b7522a3cdc0> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function fix_side_input_pcoll_coders at 0x7b7522a3cee0> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function pack_combiners at 0x7b7522a3d430> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function lift_combiners at 0x7b7522a3d4c0> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function expand_sdf at 0x7b7522a3d670> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function expand_gbk at 0x7b7522a3d700> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function sink_flattens at 0x7b7522a3d820> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function greedily_fuse at 0x7b7522a3d8b0> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function read_to_impulse at 0x7b7522a3d940> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function impulse_to_input at 0x7b7522a3d9d0> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function sort_stages at 0x7b7522a3dc10> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function add_impulse_to_dangling_transforms at 0x7b7522a3dd30> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function setup_timer_mapping at 0x7b7522a3db80> ====================
   INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function populate_data_channel_coders at 0x7b7522a3dca0> ====================
   INFO:apache_beam.runners.worker.statecache:Creating state cache with size 100
   INFO:apache_beam.runners.portability.fn_api_runner.worker_handlers:Created Worker handler <apache_beam.runners.portability.fn_api_runner.worker_handlers.EmbeddedWorkerHandler object at 0x7b752299d820> for environment ref_Environment_default_environment_1 (beam:env:embedded_python:v1, b'')
   INFO:apache_beam.io.filebasedsink:Starting finalize_write threads with num_shards: 1 (skipped: 0), batches: 1, num_threads: 1
   INFO:apache_beam.io.filebasedsink:Renamed 1 shards in 0.01 seconds.
   root@7c2c11083d7b:/# 
   ```

   You can now list the files that are on your local cloud environment to get the name of the `OUTPUT_FILE`:

   ```sh
   root@7c2c11083d7b:/# ls
   OUTPUT_FILE-00000-of-00001  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
   root@7c2c11083d7b:/# 
   ```

   Your results show each word in the file and how many times it appears.

   ```sh
   root@7c2c11083d7b:/# cat OUTPUT_FILE-00000-of-00001 
   {snip}
   ghost: 1
   rack: 1
   Stretch: 1
   usurp'd: 1
   Friends: 1
   Rule: 1
   gored: 1
   journey: 1
   weight: 1
   ought: 1
   oldest: 1
   root@7c2c11083d7b:/# 
   ```

   

## Run an example Dataflow pipeline remotely

1. Set the BUCKET environment variable to the bucket you created earlier:
2. Now you'll run the `wordcount.py` example remotely:

```sh
root@7c2c11083d7b:/# BUCKET=gs://qwiklabs-gcp-01-618fd45678b4-bucket
root@7c2c11083d7b:/# python -m apache_beam.examples.wordcount --project $DEVSHELL_PROJECT_ID \
  --runner DataflowRunner \
  --staging_location $BUCKET/staging \
  --temp_location $BUCKET/temp \
  --output $BUCKET/results/output \
  --region us-east4
INFO:apache_beam.internal.gcp.auth:Setting socket default timeout to 60 seconds.
INFO:apache_beam.internal.gcp.auth:socket default timeout is 60.0 seconds.
WARNING:google.auth._default:No project ID could be determined. Consider running `gcloud config set project` or setting the GOOGLE_CLOUD_PROJECT environment variable
INFO:apache_beam.runners.portability.stager:Downloading source distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/usr/local/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmpmaf9raoj', 'apache-beam==2.42.0', '--no-deps', '--no-binary', ':all:']

[notice] A new release of pip is available: 23.0.1 -> 24.1.1
[notice] To update, run: pip install --upgrade pip
INFO:apache_beam.runners.portability.stager:Staging SDK sources from PyPI: dataflow_python_sdk.tar
INFO:apache_beam.runners.portability.stager:Downloading binary distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/usr/local/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmpmaf9raoj', 'apache-beam==2.42.0', '--no-deps', '--only-binary', ':all:', '--python-version', '39', '--implementation', 'cp', '--abi', 'cp39', '--platform', 'manylinux1_x86_64']

[notice] A new release of pip is available: 23.0.1 -> 24.1.1
[notice] To update, run: pip install --upgrade pip
INFO:apache_beam.runners.portability.stager:Staging binary distribution of the SDK from PyPI: apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl
INFO:root:Default Python SDK image for environment is apache/beam_python3.9_sdk:2.42.0
INFO:root:Using provided Python SDK container image: gcr.io/cloud-dataflow/v1beta3/python39:2.42.0
INFO:root:Python SDK container image set to "gcr.io/cloud-dataflow/v1beta3/python39:2.42.0" for Docker environment
INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function pack_combiners at 0x783473775ca0> ====================
INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function sort_stages at 0x7834737764c0> ====================
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/pickled_main_session...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/pickled_main_session in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/dataflow_python_sdk.tar...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/dataflow_python_sdk.tar in 1 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl in 17 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/pipeline.pb...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102022-747304-i8vre7kn.1719742822.747553/pipeline.pb in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Create job: <Job
 clientRequestId: '20240630102022748643-8657'
 createTime: '2024-06-30T10:20:47.456875Z'
 currentStateTime: '1970-01-01T00:00:00Z'
 id: '2024-06-30_03_20_45-328318508161872831'
 location: 'us-east4'
 name: 'beamapp-root-0630102022-747304-i8vre7kn'
 projectId: 'qwiklabs-gcp-01-618fd45678b4'
 stageStates: []
 startTime: '2024-06-30T10:20:47.456875Z'
 steps: []
 tempFiles: []
 type: TypeValueValuesEnum(JOB_TYPE_BATCH, 1)>
INFO:apache_beam.runners.dataflow.internal.apiclient:Created job with id: [2024-06-30_03_20_45-328318508161872831]
INFO:apache_beam.runners.dataflow.internal.apiclient:Submitted job: 2024-06-30_03_20_45-328318508161872831
INFO:apache_beam.runners.dataflow.internal.apiclient:To access the Dataflow monitoring console, please navigate to https://console.cloud.google.com/dataflow/jobs/us-east4/2024-06-30_03_20_45-328318508161872831?project=qwiklabs-gcp-01-618fd45678b4
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:20:47.850Z: JOB_MESSAGE_ERROR: The Dataflow service agent cannot access the worker service account. Ensure that the Dataflow API is enabled for your project. In addition, verify that for the worker service account, the Dataflow service agent principal has the 'Cloud Dataflow Service Agent' role. To grant the role, see https://cloud.google.com/iam/docs/manage-access-service-accounts#view-access. On the Service Accounts page, select the worker service account, open the Permissions tab, and select 'Include Google-provided role grants' to verify roles. To learn more about service accounts, see https://cloud.google.com/dataflow/docs/concepts/security-and-permissions#permissions
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2024-06-30_03_20_45-328318508161872831 is in state JOB_STATE_FAILED
ERROR:apache_beam.runners.dataflow.dataflow_runner:Console URL: https://console.cloud.google.com/dataflow/jobs/<RegionId>/2024-06-30_03_20_45-328318508161872831?project=<ProjectId>
Traceback (most recent call last):
  File "/usr/local/lib/python3.9/runpy.py", line 197, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/local/lib/python3.9/runpy.py", line 87, in _run_code
    exec(code, run_globals)
  File "/usr/local/lib/python3.9/site-packages/apache_beam/examples/wordcount.py", line 105, in <module>
    run()
  File "/usr/local/lib/python3.9/site-packages/apache_beam/examples/wordcount.py", line 100, in run
    output | 'Write' >> WriteToText(known_args.output)
  File "/usr/local/lib/python3.9/site-packages/apache_beam/pipeline.py", line 598, in __exit__
    self.result.wait_until_finish()
  File "/usr/local/lib/python3.9/site-packages/apache_beam/runners/dataflow/dataflow_runner.py", line 1673, in wait_until_finish
    raise DataflowRuntimeException(
apache_beam.runners.dataflow.dataflow_runner.DataflowRuntimeException: Dataflow pipeline failed. State: FAILED, Error:
The Dataflow service agent cannot access the worker service account. Ensure that the Dataflow API is enabled for your project. In addition, verify that for the worker service account, the Dataflow service agent principal has the 'Cloud Dataflow Service Agent' role. To grant the role, see https://cloud.google.com/iam/docs/manage-access-service-accounts#view-access. On the Service Accounts page, select the worker service account, open the Permissions tab, and select 'Include Google-provided role grants' to verify roles. To learn more about service accounts, see https://cloud.google.com/dataflow/docs/concepts/security-and-permissions#permissions
root@7c2c11083d7b:/# 
```

Re-run the same command again

```sh
root@7c2c11083d7b:/# python -m apache_beam.examples.wordcount --project $DEVSHELL_PROJECT_ID   --runner DataflowRunner   --staging_location $BUCKET/staging   --temp_location $BUCKET/temp   --output $BUCKET/results/output   --region us-east4
INFO:apache_beam.internal.gcp.auth:Setting socket default timeout to 60 seconds.
INFO:apache_beam.internal.gcp.auth:socket default timeout is 60.0 seconds.
WARNING:google.auth._default:No project ID could be determined. Consider running `gcloud config set project` or setting the GOOGLE_CLOUD_PROJECT environment variable
INFO:apache_beam.runners.portability.stager:Downloading source distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/usr/local/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmpc0k4upu4', 'apache-beam==2.42.0', '--no-deps', '--no-binary', ':all:']

[notice] A new release of pip is available: 23.0.1 -> 24.1.1
[notice] To update, run: pip install --upgrade pip
INFO:apache_beam.runners.portability.stager:Staging SDK sources from PyPI: dataflow_python_sdk.tar
INFO:apache_beam.runners.portability.stager:Downloading binary distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/usr/local/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmpc0k4upu4', 'apache-beam==2.42.0', '--no-deps', '--only-binary', ':all:', '--python-version', '39', '--implementation', 'cp', '--abi', 'cp39', '--platform', 'manylinux1_x86_64']

[notice] A new release of pip is available: 23.0.1 -> 24.1.1
[notice] To update, run: pip install --upgrade pip
INFO:apache_beam.runners.portability.stager:Staging binary distribution of the SDK from PyPI: apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl
INFO:root:Default Python SDK image for environment is apache/beam_python3.9_sdk:2.42.0
INFO:root:Using provided Python SDK container image: gcr.io/cloud-dataflow/v1beta3/python39:2.42.0
INFO:root:Python SDK container image set to "gcr.io/cloud-dataflow/v1beta3/python39:2.42.0" for Docker environment
INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function pack_combiners at 0x78a2698a8ca0> ====================
INFO:apache_beam.runners.portability.fn_api_runner.translations:==================== <function sort_stages at 0x78a2698ab4c0> ====================
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/pickled_main_session...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/pickled_main_session in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/dataflow_python_sdk.tar...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/dataflow_python_sdk.tar in 1 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/apache_beam-2.42.0-cp39-cp39-manylinux1_x86_64.whl in 17 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/pipeline.pb...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/beamapp-root-0630102152-176732-gol8mvj5.1719742912.176964/pipeline.pb in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Create job: <Job
 clientRequestId: '20240630102152177971-5806'
 createTime: '2024-06-30T10:22:15.036916Z'
 currentStateTime: '1970-01-01T00:00:00Z'
 id: '2024-06-30_03_22_14-4559699916825121977'
 location: 'us-east4'
 name: 'beamapp-root-0630102152-176732-gol8mvj5'
 projectId: 'qwiklabs-gcp-01-618fd45678b4'
 stageStates: []
 startTime: '2024-06-30T10:22:15.036916Z'
 steps: []
 tempFiles: []
 type: TypeValueValuesEnum(JOB_TYPE_BATCH, 1)>
INFO:apache_beam.runners.dataflow.internal.apiclient:Created job with id: [2024-06-30_03_22_14-4559699916825121977]
INFO:apache_beam.runners.dataflow.internal.apiclient:Submitted job: 2024-06-30_03_22_14-4559699916825121977
INFO:apache_beam.runners.dataflow.internal.apiclient:To access the Dataflow monitoring console, please navigate to https://console.cloud.google.com/dataflow/jobs/us-east4/2024-06-30_03_22_14-4559699916825121977?project=qwiklabs-gcp-01-618fd45678b4
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2024-06-30_03_22_14-4559699916825121977 is in state JOB_STATE_PENDING
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:15.984Z: JOB_MESSAGE_DETAILED: Autoscaling is enabled for job 2024-06-30_03_22_14-4559699916825121977. The number of workers will be between 1 and 4000.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:16.056Z: JOB_MESSAGE_DETAILED: Autoscaling was automatically enabled for job 2024-06-30_03_22_14-4559699916825121977.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:17.930Z: JOB_MESSAGE_BASIC: Worker configuration: n1-standard-1 in us-east4-b.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.701Z: JOB_MESSAGE_DETAILED: Expanding SplittableParDo operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.724Z: JOB_MESSAGE_DETAILED: Expanding CollectionToSingleton operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.760Z: JOB_MESSAGE_DETAILED: Expanding CoGroupByKey operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.779Z: JOB_MESSAGE_DEBUG: Combiner lifting skipped for step Write/Write/WriteImpl/GroupByKey: GroupByKey not followed by a combiner.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.815Z: JOB_MESSAGE_DETAILED: Expanding GroupByKey operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.834Z: JOB_MESSAGE_DEBUG: Annotating graph with Autotuner information.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.863Z: JOB_MESSAGE_DETAILED: Fusing adjacent ParDo, Read, Write, and Flatten operations
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.884Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/InitializeWrite into Write/Write/WriteImpl/DoOnce/Map(decode)
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.900Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/DoOnce/FlatMap(<lambda at core.py:3481>) into Write/Write/WriteImpl/DoOnce/Impulse
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.918Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/DoOnce/Map(decode) into Write/Write/WriteImpl/DoOnce/FlatMap(<lambda at core.py:3481>)
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.937Z: JOB_MESSAGE_DETAILED: Fusing consumer Read/Read/Map(<lambda at iobase.py:908>) into Read/Read/Impulse
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.965Z: JOB_MESSAGE_DETAILED: Fusing consumer ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/PairWithRestriction into Read/Read/Map(<lambda at iobase.py:908>)
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:18.983Z: JOB_MESSAGE_DETAILED: Fusing consumer ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/SplitWithSizing into ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/PairWithRestriction
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19Z: JOB_MESSAGE_DETAILED: Fusing consumer Split into ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/ProcessElementAndRestrictionWithSizing
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.016Z: JOB_MESSAGE_DETAILED: Fusing consumer PairWithOne into Split
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.034Z: JOB_MESSAGE_DETAILED: Fusing consumer GroupAndSum/GroupByKey+GroupAndSum/Combine/Partial into PairWithOne
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.055Z: JOB_MESSAGE_DETAILED: Fusing consumer GroupAndSum/GroupByKey/Write into GroupAndSum/GroupByKey+GroupAndSum/Combine/Partial
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.072Z: JOB_MESSAGE_DETAILED: Fusing consumer GroupAndSum/Combine into GroupAndSum/GroupByKey/Read
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.094Z: JOB_MESSAGE_DETAILED: Fusing consumer GroupAndSum/Combine/Extract into GroupAndSum/Combine
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.114Z: JOB_MESSAGE_DETAILED: Fusing consumer Format into GroupAndSum/Combine/Extract
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.136Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/WindowInto(WindowIntoFn) into Format
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.157Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/WriteBundles into Write/Write/WriteImpl/WindowInto(WindowIntoFn)
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.172Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/Pair into Write/Write/WriteImpl/WriteBundles
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.190Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/GroupByKey/Write into Write/Write/WriteImpl/Pair
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.207Z: JOB_MESSAGE_DETAILED: Fusing consumer Write/Write/WriteImpl/Extract into Write/Write/WriteImpl/GroupByKey/Read
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.243Z: JOB_MESSAGE_DEBUG: Workflow config is missing a default resource spec.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.262Z: JOB_MESSAGE_DEBUG: Adding StepResource setup and teardown to workflow graph.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.281Z: JOB_MESSAGE_DEBUG: Adding workflow start and stop steps.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.300Z: JOB_MESSAGE_DEBUG: Assigning stage ids.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.391Z: JOB_MESSAGE_DEBUG: Executing wait step start34
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.429Z: JOB_MESSAGE_BASIC: Executing operation Read/Read/Impulse+Read/Read/Map(<lambda at iobase.py:908>)+ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/PairWithRestriction+ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/SplitWithSizing
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.449Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/DoOnce/Impulse+Write/Write/WriteImpl/DoOnce/FlatMap(<lambda at core.py:3481>)+Write/Write/WriteImpl/DoOnce/Map(decode)+Write/Write/WriteImpl/InitializeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.461Z: JOB_MESSAGE_DEBUG: Starting worker pool setup.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:22:19.479Z: JOB_MESSAGE_BASIC: Starting 1 workers in us-east4...
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2024-06-30_03_22_14-4559699916825121977 is in state JOB_STATE_RUNNING
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:23:24.022Z: JOB_MESSAGE_DETAILED: Autoscaling: Raised the number of workers to 1 based on the rate of progress in the currently running stage(s).
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:24:00.171Z: JOB_MESSAGE_DETAILED: Workers have started successfully.

```



## Check that your Dataflow job succeeded

1. Open the Navigation menu and click **Dataflow** from the list of services.

You should see your **wordcount** job with a **status** of **Running** at first.

1. Click on the name to watch the process. When all the boxes are checked off, you can continue watching the logs in Cloud Shell.

The process is complete when the status is **Succeeded**.

```sh
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:51.770Z: JOB_MESSAGE_BASIC: All workers have finished the startup processes and began to receive work requests.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.218Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/DoOnce/Impulse+Write/Write/WriteImpl/DoOnce/FlatMap(<lambda at core.py:3481>)+Write/Write/WriteImpl/DoOnce/Map(decode)+Write/Write/WriteImpl/InitializeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.257Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/DoOnce/Map(decode).None" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.278Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/InitializeWrite.None" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.318Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/WriteBundles/View-python_side_input0-Write/Write/WriteImpl/WriteBundles
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.338Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input0-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.358Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/PreFinalize/View-python_side_input0-Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.362Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/WriteBundles/View-python_side_input0-Write/Write/WriteImpl/WriteBundles
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.384Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input0-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.398Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/WriteBundles/View-python_side_input0-Write/Write/WriteImpl/WriteBundles.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.402Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/PreFinalize/View-python_side_input0-Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.422Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/FinalizeWrite/View-python_side_input0-Write/Write/WriteImpl/FinalizeWrite.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:54.439Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/PreFinalize/View-python_side_input0-Write/Write/WriteImpl/PreFinalize.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:55.686Z: JOB_MESSAGE_BASIC: Finished operation Read/Read/Impulse+Read/Read/Map(<lambda at iobase.py:908>)+ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/PairWithRestriction+ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/SplitWithSizing
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:55.727Z: JOB_MESSAGE_DEBUG: Value "ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7-split-with-sizing-out3" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:55.767Z: JOB_MESSAGE_BASIC: Executing operation GroupAndSum/GroupByKey/Create
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:56.235Z: JOB_MESSAGE_BASIC: Finished operation GroupAndSum/GroupByKey/Create
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:56.266Z: JOB_MESSAGE_DEBUG: Value "GroupAndSum/GroupByKey/Session" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:56.308Z: JOB_MESSAGE_BASIC: Executing operation ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/ProcessElementAndRestrictionWithSizing+Split+PairWithOne+GroupAndSum/GroupByKey+GroupAndSum/Combine/Partial+GroupAndSum/GroupByKey/Write
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:57.961Z: JOB_MESSAGE_BASIC: Finished operation ref_AppliedPTransform_Read-Read-SDFBoundedSourceReader-ParDo-SDFBoundedSourceDoFn-_7/ProcessElementAndRestrictionWithSizing+Split+PairWithOne+GroupAndSum/GroupByKey+GroupAndSum/Combine/Partial+GroupAndSum/GroupByKey/Write
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.004Z: JOB_MESSAGE_BASIC: Executing operation GroupAndSum/GroupByKey/Close
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.433Z: JOB_MESSAGE_BASIC: Finished operation GroupAndSum/GroupByKey/Close
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.469Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/GroupByKey/Create
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.576Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/GroupByKey/Create
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.618Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/GroupByKey/Session" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:26:58.658Z: JOB_MESSAGE_BASIC: Executing operation GroupAndSum/GroupByKey/Read+GroupAndSum/Combine+GroupAndSum/Combine/Extract+Format+Write/Write/WriteImpl/WindowInto(WindowIntoFn)+Write/Write/WriteImpl/WriteBundles+Write/Write/WriteImpl/Pair+Write/Write/WriteImpl/GroupByKey/Write
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:00.219Z: JOB_MESSAGE_BASIC: Finished operation GroupAndSum/GroupByKey/Read+GroupAndSum/Combine+GroupAndSum/Combine/Extract+Format+Write/Write/WriteImpl/WindowInto(WindowIntoFn)+Write/Write/WriteImpl/WriteBundles+Write/Write/WriteImpl/Pair+Write/Write/WriteImpl/GroupByKey/Write
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:00.259Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/GroupByKey/Close
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:00.474Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/GroupByKey/Close
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:00.514Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/GroupByKey/Read+Write/Write/WriteImpl/Extract
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.066Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/GroupByKey/Read+Write/Write/WriteImpl/Extract
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.104Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/Extract.None" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.145Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input1-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.164Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/PreFinalize/View-python_side_input1-Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.190Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input1-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.206Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/PreFinalize/View-python_side_input1-Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.219Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/FinalizeWrite/View-python_side_input1-Write/Write/WriteImpl/FinalizeWrite.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.244Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/PreFinalize/View-python_side_input1-Write/Write/WriteImpl/PreFinalize.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:02.285Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.193Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/PreFinalize
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.228Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/PreFinalize.None" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.268Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input2-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.308Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/FinalizeWrite/View-python_side_input2-Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.345Z: JOB_MESSAGE_DEBUG: Value "Write/Write/WriteImpl/FinalizeWrite/View-python_side_input2-Write/Write/WriteImpl/FinalizeWrite.out" materialized.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:04.379Z: JOB_MESSAGE_BASIC: Executing operation Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:06.113Z: JOB_MESSAGE_BASIC: Finished operation Write/Write/WriteImpl/FinalizeWrite
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:06.151Z: JOB_MESSAGE_DEBUG: Executing success step success32
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:06.197Z: JOB_MESSAGE_DETAILED: Cleaning up.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:06.406Z: JOB_MESSAGE_DEBUG: Starting worker pool teardown.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:06.423Z: JOB_MESSAGE_BASIC: Stopping worker pool...
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:48.072Z: JOB_MESSAGE_DETAILED: Autoscaling: Resized worker pool from 1 to 0.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:48.101Z: JOB_MESSAGE_BASIC: Worker pool stopped.
INFO:apache_beam.runners.dataflow.dataflow_runner:2024-06-30T10:27:48.125Z: JOB_MESSAGE_DEBUG: Tearing down pending resources...
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2024-06-30_03_22_14-4559699916825121977 is in state JOB_STATE_DONE
root@7c2c11083d7b:/# 
```



1. Click **Navigation menu** > **Cloud Storage** in the Cloud Console.
2. Click on the name of your bucket. In your bucket, you should see the **results** and **staging** directories.
3. Click on the **results** folder and you should see the output files that your job created:
4. Click on a file to see the word counts it contains.

```sh
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-01-618fd45678b4)$ gsutil ls gs://qwiklabs-gcp-01-618fd45678b4-bucket
gs://qwiklabs-gcp-01-618fd45678b4-bucket/results/
gs://qwiklabs-gcp-01-618fd45678b4-bucket/staging/
gs://qwiklabs-gcp-01-618fd45678b4-bucket/temp/
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-01-618fd45678b4)$ gsutil ls gs://qwiklabs-gcp-01-618fd45678b4-bucket/results
gs://qwiklabs-gcp-01-618fd45678b4-bucket/results/output-00000-of-00001
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-01-618fd45678b4)$ 
```



You learned how to set up your Python development environment for  Dataflow (using the Apache Beam SDK for Python) and ran an example  Dataflow pipeline.
