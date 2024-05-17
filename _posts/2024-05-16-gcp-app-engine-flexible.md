---

layout: single
title:  "Deploying a Python Flask Web Application to App Engine Flexible"
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

# Deploying a Python Flask Web Application to App Engine Flexible

### About App Engine

Google App Engine applications are easy to create, easy to maintain,  and easy to scale as your traffic and data storage needs change. With  App Engine, there are no servers to maintain. You simply upload your  application and it's ready to go.

App Engine applications automatically scale based on incoming  traffic. Load balancing, microservices, authorization, SQL and NoSQL  databases, traffic splitting, logging, search, versioning, roll out and  roll backs, and security scanning are all supported natively and are  highly customizable.

App Engine's [Flexible Environment](https://cloud.google.com/appengine/docs/flexible) supports a host of programming languages, including Java, Python, PHP, NodeJS, Ruby, and Go. App Engine's [Standard Environment](https://cloud.google.com/appengine/docs/about-the-standard-environment) is an additional option for certain languages including Python. The two environments give users maximum flexibility in how their application  behaves since each environment has certain strengths. Read [Choosing an App Engine Environment](https://cloud.google.com/appengine/docs/the-appengine-environments) for more information.

- How to deploy a simple web application to the App Engine Flexible Environment
- How to access the Google Cloud client libraries for Vision, Storage, and Datastore
- How to use Cloud Shell

## Get the sample code

1. In Cloud Shell, run the following command to clone the Github repository:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-b31031997333.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_bf71fa739d08@cloudshell:~ (qwiklabs-gcp-02-b31031997333)$ gcloud storage cp -r gs://spls/gsp023/flex_and_vision/ .
Copying gs://spls/gsp023/flex_and_vision/README.md to file://./flex_and_vision/README.md
Copying gs://spls/gsp023/flex_and_vision/app.yaml to file://./flex_and_vision/app.yaml
Copying gs://spls/gsp023/flex_and_vision/main.py to file://./flex_and_vision/main.py                                                          
Copying gs://spls/gsp023/flex_and_vision/main_test.py to file://./flex_and_vision/main_test.py
Copying gs://spls/gsp023/flex_and_vision/requirements-test.txt to file://./flex_and_vision/requirements-test.txt
Copying gs://spls/gsp023/flex_and_vision/requirements.txt to file://./flex_and_vision/requirements.txt
Copying gs://spls/gsp023/flex_and_vision/templates/homepage.html to file://./flex_and_vision/templates/homepage.html
  Completed files 7/7 | 11.6kiB/11.6kiB                                                                                                       

Average throughput: 11.7kiB/s
student_04_bf71fa739d08@cloudshell:~ (qwiklabs-gcp-02-b31031997333)$ cd flex_and_vision
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



## Authenticate API requests

The Datastore, Storage, and Vision APIs are automatically enabled for you in this lab. In order to make requests to the APIs, you will need  service account credentials. You can generate credentials from your  project using *gcloud* in Cloud Shell. Your **Project ID** can be found on the tab where you started the lab.

```sh
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ export PROJECT_ID=$(gcloud config get-value project)
Your active configuration is: [cloudshell-25885]
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud iam service-accounts create qwiklab \
  --display-name "My Qwiklab Service Account"
Created service account [qwiklab].
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member serviceAccount:qwiklab@${PROJECT_ID}.iam.gserviceaccount.com \
--role roles/owner
Updated IAM policy for project [qwiklabs-gcp-02-b31031997333].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-02-b31031997333@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:671634776807@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-671634776807@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-671634776807@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-671634776807@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:671634776807-compute@developer.gserviceaccount.com
  - serviceAccount:671634776807@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklab@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-02-b31031997333@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com
  - user:student-04-bf71fa739d08@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-02-b31031997333@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-04-bf71fa739d08@qwiklabs.net
  role: roles/viewer
etag: BwYYqJPTcaY=
version: 1
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud iam service-accounts keys create ~/key.json \
--iam-account qwiklab@${PROJECT_ID}.iam.gserviceaccount.com
created key [b69c5916c15aa1f440cfd60644e8dcb4949c615d] of type [json] as [/home/student_04_bf71fa739d08/key.json] for [qwiklab@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com]
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ export GOOGLE_APPLICATION_CREDENTIALS="/home/${USER}/key.json"
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



```json
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat ~/key.json 
{
  "type": "service_account",
  "project_id": "qwiklabs-gcp-02-b31031997333",
  "private_key_id": "b69c5916c15aa1f440cfd60644e8dcb4949c615d",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDyCOz7X51zf99s\n6MkmB4/7QVEBoXX3OkpDDlXSBNa2Ms5OY4crYNtx5f/ytdrMJUabLa3idOz122xW\nw1T2eohxesVMTs7Jyjh/M8WSLLx/Z2fJUh7ppBfnweAKCB90qHClT8yMhgEjQM2l\nWSirIKc8UCtH6bhGet/QunVZtUE52aHsAe32uA6RGWIwvf4qhGfNyt1+r20VtQWr\nFXCP01ifCXkE+IWkWlVVX0dBOEAp0+XAOUCvYAtK20Z36vLgb6iZO9B73RdPIh1S\nyXmJw6q2TFM1+/GFayiwSktMGGoC6yCTAnACXkTNBT+vBS3YL2ir64QUa2fvyHp2\n6YLtP8I3AgMBAAECggEAPvzaq5qZD8dd/mpgesCqFFHNwpZh7Fqjm+rdo7/1nsn/\nDcByG3Rj97LLFr+D9u/Wfaj4IUCbsGoPuk6wTErcOmggc3jo8PPrGxN+ncl9rsxa\n4rY37Ebzn7FBXGr7wLDbS/JGAeYX4rRJMHhREKP5UcVtVhQ5jEILADeeNZ/pnyOG\nI6QtnaECeBznOJQLl7raPOd2/Jpz7Hv7E3lTFXkv7vjYyYHDEc1HEU9pZ6wXSRlx\nw1UmN9FlAS/8diVlnXRqIWYaD4PqgFIK1PVlyO1XUU89vwRSkl4FjQApbjweJ0XL\nOe6AIhOEalngth8Qj+xi3do6XdPuhDNAptMB86HWgQKBgQD6TnxKno15OFqQVFCN\n3btDbzk+2VFcDE32M/5JBtGgObi7dAMdbhuVw03np9WeDH+1C341gqVwAWAybVhq\nvtkQ8LHEP7p5hTBJWJh7Ic7m4UuJ0mYLaDRLUoctY0v3SFIMPUauYOOsvQJrmjXy\n20cvFUKKsjrL9Z8Rq33oF+vUgQKBgQD3ikZQtfIhRhzUKlMQWHToTcMlcM3Qc05J\nKeOQl4onSTKiaH3M2CLFoC9WWhLsmv7BpOdzDuUtVr/lZXOT3RU+dx2tcEmFhZyd\ncGmsuartkIjusxWC1CzU9d5Wkq/CMUn2cmvZwgaCFxOGo1xyk7Q92QRdnF0MVrc9\nR60/AjXatwKBgQCzrgWQ9zItU2PHeY79166moL/iOtQplHeehgJC3885ClZu0b+u\nr6zDnBhfc95nfydpih+GQAuMVKB+cnnm3qspeu7RJsIwm4hnDl8e/Mzudcno3Iz+\nIUZwz4RT85TDpTmoqZAEe27UQDXtkhyqAfieds92iqykXuRaJdXS9uEGgQKBgQCJ\n/xzZ68RyxjpWEM5Do3xw8MDkg3FJTq6K3P5O4hwTcJv4rBXNd4RS9czN7+Ly4ik5\nXKvmmZwrXVwXDyqSeMJaE1+JC7sA445+umc+8jaWv2eG4nEQgSYJBpQPYTD4KjAY\nYos7Vw33wdOR0Eo+WZc2j1/+q6e3tDPsxqOPJ7VMGwKBgCVPCtdqMJQ6L27faxpv\nESdfDBhOaIblx2M6xf4VfOPNPt3glLbdD4C61zpdDPF6bgU0lg5EfYiC1XcI04Qi\nCSPZj6WCuXeiFEQCEuQcgUnSumdsFZEYFZKio9TUDIdu8PNbYomF8KfRC3k8Yy5h\nT1wCC0uuJI0Lby1gK3HPmYL6\n-----END PRIVATE KEY-----\n",
  "client_email": "qwiklab@qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com",
  "client_id": "115156694934979817423",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/qwiklab%40qwiklabs-gcp-02-b31031997333.iam.gserviceaccount.com",
  "universe_domain": "googleapis.com"
}
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



## Testing the application locally

### Starting your virtual environment and installing dependencies

```sh
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ virtualenv -p python3 env
created virtual environment CPython3.10.12.final.0-64 in 1006ms
  creator CPython3Posix(dest=/home/student_04_bf71fa739d08/flex_and_vision/env, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/student_04_bf71fa739d08/.local/share/virtualenv)
    added seed packages: pip==24.0, setuptools==69.5.1, wheel==0.43.0
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ source env/bin/activate
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ pip install -r requirements.txt
Ignoring gunicorn: markers 'python_version < "3.0"' don't match your environment
Ignoring google-cloud-storage: markers 'python_version < "3.7"' don't match your environment
Collecting Flask==2.0.3 (from -r requirements.txt (line 1))
  Downloading Flask-2.0.3-py3-none-any.whl.metadata (3.8 kB)
Collecting gunicorn==20.1.0 (from -r requirements.txt (line 2))
  Downloading gunicorn-20.1.0-py3-none-any.whl.metadata (3.8 kB)
Collecting google-cloud-vision==2.1.0 (from -r requirements.txt (line 4))
  Downloading google_cloud_vision-2.1.0-py2.py3-none-any.whl.metadata (4.9 kB)
Collecting google-cloud-storage==2.1.0 (from -r requirements.txt (line 6))
  Downloading google_cloud_storage-2.1.0-py2.py3-none-any.whl.metadata (5.2 kB)
Collecting google-cloud-datastore==2.7.1 (from -r requirements.txt (line 7))
  Downloading google_cloud_datastore-2.7.1-py2.py3-none-any.whl.metadata (4.9 kB)
Collecting Werkzeug==2.0.3 (from -r requirements.txt (line 8))
  Downloading Werkzeug-2.0.3-py3-none-any.whl.metadata (4.5 kB)
Collecting urllib3<2.0.0 (from -r requirements.txt (line 9))
  Downloading urllib3-1.26.18-py2.py3-none-any.whl.metadata (48 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 48.9/48.9 kB 3.2 MB/s eta 0:00:00
Collecting Jinja2>=3.0 (from Flask==2.0.3->-r requirements.txt (line 1))
  Downloading jinja2-3.1.4-py3-none-any.whl.metadata (2.6 kB)
Collecting itsdangerous>=2.0 (from Flask==2.0.3->-r requirements.txt (line 1))
  Downloading itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting click>=7.1.2 (from Flask==2.0.3->-r requirements.txt (line 1))
  Downloading click-8.1.7-py3-none-any.whl.metadata (3.0 kB)
Requirement already satisfied: setuptools>=3.0 in ./env/lib/python3.10/site-packages (from gunicorn==20.1.0->-r requirements.txt (line 2)) (69.5.1)
Collecting google-api-core<2.0.0dev,>=1.22.2 (from google-api-core[grpc]<2.0.0dev,>=1.22.2->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading google_api_core-1.34.1-py3-none-any.whl.metadata (2.4 kB)
Collecting proto-plus>=1.4.0 (from google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading proto_plus-1.23.0-py3-none-any.whl.metadata (2.2 kB)
Collecting libcst>=0.2.5 (from google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading libcst-1.3.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (17 kB)
Collecting google-auth<3.0dev,>=1.25.0 (from google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading google_auth-2.29.0-py2.py3-none-any.whl.metadata (4.7 kB)
Collecting google-cloud-core<3.0dev,>=1.6.0 (from google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading google_cloud_core-2.4.1-py2.py3-none-any.whl.metadata (2.7 kB)
Collecting google-resumable-media>=1.3.0 (from google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading google_resumable_media-2.7.0-py2.py3-none-any.whl.metadata (2.2 kB)
Collecting requests<3.0.0dev,>=2.18.0 (from google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading requests-2.31.0-py3-none-any.whl.metadata (4.6 kB)
Collecting protobuf (from google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading protobuf-5.26.1-cp37-abi3-manylinux2014_x86_64.whl.metadata (592 bytes)
  Downloading protobuf-3.20.3-cp310-cp310-manylinux_2_12_x86_64.manylinux2010_x86_64.whl.metadata (679 bytes)
Collecting googleapis-common-protos<2.0dev,>=1.56.2 (from google-api-core<2.0.0dev,>=1.22.2->google-api-core[grpc]<2.0.0dev,>=1.22.2->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading googleapis_common_protos-1.63.0-py2.py3-none-any.whl.metadata (1.5 kB)
Collecting grpcio<2.0dev,>=1.33.2 (from google-api-core[grpc]<2.0.0dev,>=1.22.2->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading grpcio-1.63.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.2 kB)
Collecting grpcio-status<2.0dev,>=1.33.2 (from google-api-core[grpc]<2.0.0dev,>=1.22.2->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading grpcio_status-1.63.0-py3-none-any.whl.metadata (1.1 kB)
Collecting cachetools<6.0,>=2.0.0 (from google-auth<3.0dev,>=1.25.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading cachetools-5.3.3-py3-none-any.whl.metadata (5.3 kB)
Collecting pyasn1-modules>=0.2.1 (from google-auth<3.0dev,>=1.25.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading pyasn1_modules-0.4.0-py3-none-any.whl.metadata (3.4 kB)
Collecting rsa<5,>=3.1.4 (from google-auth<3.0dev,>=1.25.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading rsa-4.9-py3-none-any.whl.metadata (4.2 kB)
Collecting google-crc32c<2.0dev,>=1.0 (from google-resumable-media>=1.3.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading google_crc32c-1.5.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (2.3 kB)
Collecting MarkupSafe>=2.0 (from Jinja2>=3.0->Flask==2.0.3->-r requirements.txt (line 1))
  Downloading MarkupSafe-2.1.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.0 kB)
Collecting pyyaml>=5.2 (from libcst>=0.2.5->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading PyYAML-6.0.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (2.1 kB)
Collecting charset-normalizer<4,>=2 (from requests<3.0.0dev,>=2.18.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading charset_normalizer-3.3.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (33 kB)
Collecting idna<4,>=2.5 (from requests<3.0.0dev,>=2.18.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading idna-3.7-py3-none-any.whl.metadata (9.9 kB)
Collecting certifi>=2017.4.17 (from requests<3.0.0dev,>=2.18.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading certifi-2024.2.2-py3-none-any.whl.metadata (2.2 kB)
INFO: pip is looking at multiple versions of grpcio-status to determine which version is compatible with other requirements. This could take a while.
Collecting grpcio-status<2.0dev,>=1.33.2 (from google-api-core[grpc]<2.0.0dev,>=1.22.2->google-cloud-vision==2.1.0->-r requirements.txt (line 4))
  Downloading grpcio_status-1.62.2-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.62.1-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.62.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.60.1-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.60.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.59.3-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.59.2-py3-none-any.whl.metadata (1.3 kB)
INFO: pip is still looking at multiple versions of grpcio-status to determine which version is compatible with other requirements. This could take a while.
  Downloading grpcio_status-1.59.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.58.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.57.0-py3-none-any.whl.metadata (1.2 kB)
  Downloading grpcio_status-1.56.2-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.56.0-py3-none-any.whl.metadata (1.3 kB)
INFO: This is taking longer than usual. You might need to provide the dependency resolver with stricter constraints to reduce runtime. See https://pip.pypa.io/warnings/backtracking for guidance. If you want to abort this run, press Ctrl + C.
  Downloading grpcio_status-1.55.3-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.54.3-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.54.2-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.54.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.53.2-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.53.1-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.53.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.51.3-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.51.1-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.50.0-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.49.1-py3-none-any.whl.metadata (1.3 kB)
  Downloading grpcio_status-1.48.2-py3-none-any.whl.metadata (1.2 kB)
Collecting pyasn1<0.7.0,>=0.4.6 (from pyasn1-modules>=0.2.1->google-auth<3.0dev,>=1.25.0->google-cloud-storage==2.1.0->-r requirements.txt (line 6))
  Downloading pyasn1-0.6.0-py2.py3-none-any.whl.metadata (8.3 kB)
Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.6/95.6 kB 7.0 MB/s eta 0:00:00
Downloading gunicorn-20.1.0-py3-none-any.whl (79 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 79.5/79.5 kB 6.3 MB/s eta 0:00:00
Downloading google_cloud_vision-2.1.0-py2.py3-none-any.whl (458 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 458.4/458.4 kB 18.7 MB/s eta 0:00:00
Downloading google_cloud_storage-2.1.0-py2.py3-none-any.whl (106 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 106.6/106.6 kB 7.6 MB/s eta 0:00:00
Downloading google_cloud_datastore-2.7.1-py2.py3-none-any.whl (140 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 140.4/140.4 kB 8.6 MB/s eta 0:00:00
Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 289.2/289.2 kB 14.9 MB/s eta 0:00:00
Downloading urllib3-1.26.18-py2.py3-none-any.whl (143 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 143.8/143.8 kB 9.5 MB/s eta 0:00:00
Downloading click-8.1.7-py3-none-any.whl (97 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.9/97.9 kB 7.8 MB/s eta 0:00:00
Downloading google_api_core-1.34.1-py3-none-any.whl (120 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 120.4/120.4 kB 7.2 MB/s eta 0:00:00
Downloading google_auth-2.29.0-py2.py3-none-any.whl (189 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 189.2/189.2 kB 11.2 MB/s eta 0:00:00
Downloading google_cloud_core-2.4.1-py2.py3-none-any.whl (29 kB)
Downloading google_resumable_media-2.7.0-py2.py3-none-any.whl (80 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 80.6/80.6 kB 6.3 MB/s eta 0:00:00
Downloading itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Downloading jinja2-3.1.4-py3-none-any.whl (133 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.3/133.3 kB 8.3 MB/s eta 0:00:00
Downloading libcst-1.3.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.3 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.3/2.3 MB 31.5 MB/s eta 0:00:00
Downloading proto_plus-1.23.0-py3-none-any.whl (48 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 48.8/48.8 kB 3.4 MB/s eta 0:00:00
Downloading protobuf-3.20.3-cp310-cp310-manylinux_2_12_x86_64.manylinux2010_x86_64.whl (1.1 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.1/1.1 MB 41.3 MB/s eta 0:00:00
Downloading requests-2.31.0-py3-none-any.whl (62 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.6/62.6 kB 4.6 MB/s eta 0:00:00
Downloading cachetools-5.3.3-py3-none-any.whl (9.3 kB)
Downloading certifi-2024.2.2-py3-none-any.whl (163 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 163.8/163.8 kB 12.2 MB/s eta 0:00:00
Downloading charset_normalizer-3.3.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (142 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 142.1/142.1 kB 9.2 MB/s eta 0:00:00
Downloading google_crc32c-1.5.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (32 kB)
Downloading googleapis_common_protos-1.63.0-py2.py3-none-any.whl (229 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 229.1/229.1 kB 8.0 MB/s eta 0:00:00
Downloading grpcio-1.63.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (5.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.6/5.6 MB 66.7 MB/s eta 0:00:00
Downloading grpcio_status-1.48.2-py3-none-any.whl (14 kB)
Downloading idna-3.7-py3-none-any.whl (66 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 66.8/66.8 kB 4.2 MB/s eta 0:00:00
Downloading MarkupSafe-2.1.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Downloading pyasn1_modules-0.4.0-py3-none-any.whl (181 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 181.2/181.2 kB 11.8 MB/s eta 0:00:00
Downloading PyYAML-6.0.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (705 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 705.5/705.5 kB 24.2 MB/s eta 0:00:00
Downloading rsa-4.9-py3-none-any.whl (34 kB)
Downloading pyasn1-0.6.0-py2.py3-none-any.whl (85 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 85.3/85.3 kB 6.3 MB/s eta 0:00:00
Installing collected packages: Werkzeug, urllib3, pyyaml, pyasn1, protobuf, MarkupSafe, itsdangerous, idna, gunicorn, grpcio, google-crc32c, click, charset-normalizer, certifi, cachetools, rsa, requests, pyasn1-modules, proto-plus, libcst, Jinja2, googleapis-common-protos, google-resumable-media, grpcio-status, google-auth, Flask, google-api-core, google-cloud-core, google-cloud-vision, google-cloud-storage, google-cloud-datastore
Successfully installed Flask-2.0.3 Jinja2-3.1.4 MarkupSafe-2.1.5 Werkzeug-2.0.3 cachetools-5.3.3 certifi-2024.2.2 charset-normalizer-3.3.2 click-8.1.7 google-api-core-1.34.1 google-auth-2.29.0 google-cloud-core-2.4.1 google-cloud-datastore-2.7.1 google-cloud-storage-2.1.0 google-cloud-vision-2.1.0 google-crc32c-1.5.0 google-resumable-media-2.7.0 googleapis-common-protos-1.63.0 grpcio-1.63.0 grpcio-status-1.48.2 gunicorn-20.1.0 idna-3.7 itsdangerous-2.2.0 libcst-1.3.1 proto-plus-1.23.0 protobuf-3.20.3 pyasn1-0.6.0 pyasn1-modules-0.4.0 pyyaml-6.0.1 requests-2.31.0 rsa-4.9 urllib3-1.26.18
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



### Creating an App Engine app

1. First, create an environment variable with your assigned region:

```sh
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ AE_REGION=us-west1
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud app create --region=$AE_REGION
You are creating an app for project [qwiklabs-gcp-02-b31031997333].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Creating App Engine application in project [qwiklabs-gcp-02-b31031997333] and region [us-west1]....done.                                      
Success! The app is now created. Please use `gcloud app deploy` to deploy your first app.
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$
```



### Creating a storage bucket

1. First, set the environment variable *CLOUD_STORAGE_BUCKET* equal to the name of your *PROJECT_ID*. (It is generally recommended to name your bucket the same as your *PROJECT_ID* for convenience purposes):

```sh
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ export CLOUD_STORAGE_BUCKET=${PROJECT_ID}
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gsutil mb gs://${PROJECT_ID}
Creating gs://qwiklabs-gcp-02-b31031997333/...
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



### Running the Application

1. Execute the following command to start your application:

```py
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ python main.py
 * Serving Flask app 'main' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 161-988-275
127.0.0.1 - - [17/May/2024 16:14:17] "GET /?authuser=2&redirectedPreviously=true HTTP/1.1" 200 -
127.0.0.1 - - [17/May/2024 16:14:17] "GET /favicon.ico HTTP/1.1" 404 -
127.0.0.1 - - [17/May/2024 16:14:56] "POST /upload_photo HTTP/1.1" 302 -
127.0.0.1 - - [17/May/2024 16:14:57] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [17/May/2024 16:14:57] "GET /favicon.ico HTTP/1.1" 404 -
^C(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 

```

```sh
Google Cloud Platform - Face Detection Sample

This Python Flask application demonstrates App Engine Flexible, Google Cloud Storage, Datastore, and the Cloud Vision API.
```



## Exploring the code

### Sample code layout

```py
env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat main.py 
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from datetime import datetime
import logging
import os

from flask import Flask, redirect, render_template, request

from google.cloud import datastore
from google.cloud import storage
from google.cloud import vision


CLOUD_STORAGE_BUCKET = os.environ.get("CLOUD_STORAGE_BUCKET")


app = Flask(__name__)


@app.route("/")
def homepage():
    # Create a Cloud Datastore client.
    datastore_client = datastore.Client()

    # Use the Cloud Datastore client to fetch information from Datastore about
    # each photo.
    query = datastore_client.query(kind="Faces")
    image_entities = list(query.fetch())

    # Return a Jinja2 HTML template and pass in image_entities as a parameter.
    return render_template("homepage.html", image_entities=image_entities)


@app.route("/upload_photo", methods=["GET", "POST"])
def upload_photo():
    photo = request.files["file"]

    # Create a Cloud Storage client.
    storage_client = storage.Client()

    # Get the bucket that the file will be uploaded to.
    bucket = storage_client.get_bucket(CLOUD_STORAGE_BUCKET)

    # Create a new blob and upload the file's content.
    blob = bucket.blob(photo.filename)
    blob.upload_from_string(photo.read(), content_type=photo.content_type)

    # Make the blob publicly viewable.
    blob.make_public()

    # Create a Cloud Vision client.
    vision_client = vision.ImageAnnotatorClient()

    # Use the Cloud Vision client to detect a face for our image.
    source_uri = "gs://{}/{}".format(CLOUD_STORAGE_BUCKET, blob.name)
    image = vision.Image(source=vision.ImageSource(gcs_image_uri=source_uri))
    faces = vision_client.face_detection(image=image).face_annotations

    # If a face is detected, save to Datastore the likelihood that the face
    # displays 'joy,' as determined by Google's Machine Learning algorithm.
    if len(faces) > 0:
        face = faces[0]

        # Convert the likelihood string.
        likelihoods = [
            "Unknown",
            "Very Unlikely",
            "Unlikely",
            "Possible",
            "Likely",
            "Very Likely",
        ]
        face_joy = likelihoods[face.joy_likelihood]
    else:
        face_joy = "Unknown"

    # Create a Cloud Datastore client.
    datastore_client = datastore.Client()

    # Fetch the current date / time.
    current_datetime = datetime.now()

    # The kind for the new entity.
    kind = "Faces"

    # The name/ID for the new entity.
    name = blob.name

    # Create the Cloud Datastore key for the new entity.
    key = datastore_client.key(kind, name)

    # Construct the new entity using the key. Set dictionary values for entity
    # keys blob_name, storage_public_url, timestamp, and joy.
    entity = datastore.Entity(key)
    entity["blob_name"] = blob.name
    entity["image_public_url"] = blob.public_url
    entity["timestamp"] = current_datetime
    entity["joy"] = face_joy

    # Save the new entity to Datastore.
    datastore_client.put(entity)

    # Redirect to the home page.
    return redirect("/")


@app.errorhandler(500)
def server_error(e):
    logging.exception("An error occurred during a request.")
    return (
        """
    An internal error occurred: <pre>{}</pre>
    See logs for full stacktrace.
    """.format(
            e
        ),
        500,
    )


if __name__ == "__main__":
    # This is used when running locally. Gunicorn is used to run the
    # application on Google App Engine. See entrypoint in app.yaml.
    app.run(host="127.0.0.1", port=8080, debug=True)
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



```py
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat main_test.py 
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import uuid

import backoff
from google.api_core.exceptions import GatewayTimeout
import pytest
import requests
import six

import main

TEST_PHOTO_URL = (
    'https://upload.wikimedia.org/wikipedia/commons/5/5e/'
    'John_F._Kennedy%2C_White_House_photo_portrait%2C_looking_up.jpg')


@pytest.fixture
def app():
    main.app.testing = True
    client = main.app.test_client()
    return client


def test_index(app):
    r = app.get('/')
    assert r.status_code == 200


def test_upload_photo(app):
    test_photo_data = requests.get(TEST_PHOTO_URL).content
    test_photo_filename = 'flex_and_vision_{}.jpg'.format(uuid.uuid4().hex)

    @backoff.on_exception(backoff.expo, GatewayTimeout, max_time=120)
    def run_sample():
        return app.post(
            '/upload_photo',
            data={
                'file': (six.BytesIO(test_photo_data), test_photo_filename)
            }
        )

    r = run_sample()

    assert r.status_code == 302
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

```html
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat templates/homepage.html 
<!--
 Copyright 2021 Google LLC

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

<h1>Google Cloud Platform - Face Detection Sample</h1>

<p>This Python Flask application demonstrates App Engine Flexible, Google Cloud
Storage, Datastore, and the Cloud Vision API.</p>

<br>

<html>
  <body>
    <form action="upload_photo" method="POST" enctype="multipart/form-data">
      Upload File: <input type="file" name="file"><br>
      <input type="submit" name="submit" value="Submit">
    </form>
    {% for image_entity in image_entities %}
      <img src="{{image_entity['image_public_url']}}" width=200 height=200>
      <p>{{image_entity['blob_name']}} was uploaded {{image_entity['timestamp']}}.</p>
      <p>Joy Likelihood for Face: {{image_entity['joy']}}</p>
    {% endfor %}
  </body>
</html> 
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



## Deploying the App to App Engine Flexible

App Engine Flexible uses a file called `app.yaml` to  describe an application's deployment configuration. If this file is not  present, App Engine will try to guess the deployment configuration.  However, it is a good idea to provide this file.

1. Next, you will modify `app.yaml` using an editor of your choice *vim*, *nano*, or *emacs*. We will use the `nano` editor:

```yaml
env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat app.yaml 
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

runtime: python
env: flex
entrypoint: gunicorn -b :$PORT main:app

runtime_config:
  python_version: 3

env_variables:
  CLOUD_STORAGE_BUCKET: <your-cloud-storage-bucket>
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```



after modification

```yaml
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ cat app.yaml 
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

runtime: python
env: flex
entrypoint: gunicorn -b :$PORT main:app

runtime_config:
  python_version: 3.7

env_variables:
  CLOUD_STORAGE_BUCKET: qwiklabs-gcp-02-b31031997333

manual_scaling:
  instances: 1
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

This is the basic configuration needed to deploy a Python 3 App Engine  Flex application. You can learn more about configuring App Engine at [Configuring your App with app.yaml Guide](https://cloud.google.com/appengine/docs/flexible/python/configuring-your-app-with-app-yaml).

```sh
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud config set app/cloud_build_timeout 1000
Updated property [app/cloud_build_timeout].
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

Watch in Cloud Shell as the application gets built. This will take up to **10** minutes. The App Engine Flexible environment is automatically  provisioning a Compute Engine virtual machine for you behind the scenes, and then installing the application, then starting 

```sh
env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud app deploy
Services to deploy:

descriptor:                  [/home/student_04_bf71fa739d08/flex_and_vision/app.yaml]
source:                      [/home/student_04_bf71fa739d08/flex_and_vision]
target project:              [qwiklabs-gcp-02-b31031997333]
target service:              [default]
target version:              [20240517t162117]
target url:                  [https://qwiklabs-gcp-02-b31031997333.uw.r.appspot.com]
target service account:      [qwiklabs-gcp-02-b31031997333@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Uploading 2138 files to Google Cloud Storage
5%
10%
15%
20%
25%
30%
35%
40%
45%
50%
55%
60%
65%
70%
75%
80%
85%
90%
95%
100%
100%
File upload done.
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...done.                                                                                                                 
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "084b3295-33ba-41ed-b6e0-daa6f4aca42d"

FETCHSOURCE
BUILD
Starting Step #0 - "fetcher"
Step #0 - "fetcher": Already have image (with digest): gcr.io/cloud-builders/gcs-fetcher
Step #0 - "fetcher": Fetching manifest gs://staging.qwiklabs-gcp-02-b31031997333.appspot.com/ae/2de3a7d1-2d3b-4a78-bd29-f5ff16dd6635/manifest.json.
Step #0 - "fetcher": Processing 2427 files.
Step #0 - "fetcher": ******************************************************
Step #0 - "fetcher": Status:                      SUCCESS
Step #0 - "fetcher": Started:                     2024-05-17T16:25:33Z
Step #0 - "fetcher": Completed:                   2024-05-17T16:25:36Z
Step #0 - "fetcher": Requested workers:    200
Step #0 - "fetcher": Actual workers:       200
Step #0 - "fetcher": Total files:         2427
Step #0 - "fetcher": Total retries:          0
Step #0 - "fetcher": GCS timeouts:           0
Step #0 - "fetcher": MiB downloaded:        64.04 MiB
Step #0 - "fetcher": MiB/s throughput:      25.12 MiB/s
Step #0 - "fetcher": Time for manifest:    435.17 ms
Step #0 - "fetcher": Total time:             3.01 s
Step #0 - "fetcher": ******************************************************
Finished Step #0 - "fetcher"
Starting Step #1
Step #1: Pulling image: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa: Pulling from gcp-runtimes/python/gen-dockerfile
Step #1: Digest: sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: Status: Downloaded newer image for gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Finished Step #1
Starting Step #2
Step #2: Pulling image: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd: Pulling from cloud-builders/docker
Step #2: 75f546e73d8b: Pulling fs layer
Step #2: 0f3bb76fc390: Pulling fs layer
Step #2: 3c2cba919283: Pulling fs layer
Step #2: 4d168e97939c: Pulling fs layer
Step #2: 3c2cba919283: Verifying Checksum
Step #2: 3c2cba919283: Download complete
Step #2: 0f3bb76fc390: Verifying Checksum
Step #2: 0f3bb76fc390: Download complete
Step #2: 75f546e73d8b: Verifying Checksum
Step #2: 75f546e73d8b: Download complete
Step #2: 75f546e73d8b: Pull complete
Step #2: 4d168e97939c: Verifying Checksum
Step #2: 4d168e97939c: Download complete
Step #2: 0f3bb76fc390: Pull complete
Step #2: 3c2cba919283: Pull complete
Step #2: 4d168e97939c: Pull complete
Step #2: Digest: sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Status: Downloaded newer image for gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Sending build context to Docker daemon   69.2MB
Step #2: Step 1/9 : FROM gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6: Pulling from google-appengine/python
Step #2: Digest: sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: Status: Downloaded newer image for gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2:  ce284ab20159
Step #2: Step 2/9 : LABEL python_version=python3.7
Step #2:  Running in 8e415e8f7ad7
Step #2: Removing intermediate container 8e415e8f7ad7
Step #2:  0a6e28c7989c
Step #2: Step 3/9 : RUN virtualenv --no-download /env -p python3.7
Step #2:  Running in 8b005e9f22e4
Step #2: created virtual environment CPython3.7.9.final.0-64 in 1239ms
Step #2:   creator CPython3Posix(dest=/env, clear=False, global=False)
Step #2:   seeder FromAppData(download=False, pip=bundle, wheel=bundle, setuptools=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
Step #2:     added seed packages: pip==20.2.2, setuptools==49.6.0, wheel==0.35.1
Step #2:   activators PythonActivator,FishActivator,XonshActivator,CShellActivator,PowerShellActivator,BashActivator
Step #2: Removing intermediate container 8b005e9f22e4
Step #2:  baf49f600814
Step #2: Step 4/9 : ENV VIRTUAL_ENV /env
Step #2:  Running in 07e66a1cb1e7
Step #2: Removing intermediate container 07e66a1cb1e7
Step #2:  770939a79ac8
Step #2: Step 5/9 : ENV PATH /env/bin:$PATH
Step #2:  Running in f6f70eb1f517
Step #2: Removing intermediate container f6f70eb1f517
Step #2:  1e03b6cf5a18
Step #2: Step 6/9 : ADD requirements.txt /app/
Step #2:  f476fe58dd83
Step #2: Step 7/9 : RUN pip install -r requirements.txt
Step #2:  Running in 14fa680c9498
Step #2: Ignoring gunicorn: markers 'python_version < "3.0"' don't match your environment
Step #2: Ignoring google-cloud-storage: markers 'python_version < "3.7"' don't match your environment
Step #2: Collecting Flask==2.0.3
Step #2:   Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Step #2: Collecting gunicorn==20.1.0
Step #2:   Downloading gunicorn-20.1.0-py3-none-any.whl (79 kB)
Step #2: Collecting google-cloud-vision==2.1.0
Step #2:   Downloading google_cloud_vision-2.1.0-py2.py3-none-any.whl (458 kB)
Step #2: Collecting google-cloud-storage==2.1.0
Step #2:   Downloading google_cloud_storage-2.1.0-py2.py3-none-any.whl (106 kB)
Step #2: Collecting google-cloud-datastore==2.7.1
Step #2:   Downloading google_cloud_datastore-2.7.1-py2.py3-none-any.whl (140 kB)
Step #2: Collecting Werkzeug==2.0.3
Step #2:   Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Step #2: Collecting urllib3<2.0.0
Step #2:   Downloading urllib3-1.26.18-py2.py3-none-any.whl (143 kB)
Step #2: Collecting click>=7.1.2
Step #2:   Downloading click-8.1.7-py3-none-any.whl (97 kB)
Step #2: Collecting itsdangerous>=2.0
Step #2:   Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Step #2: Collecting Jinja2>=3.0
Step #2:   Downloading jinja2-3.1.4-py3-none-any.whl (133 kB)
Step #2: Requirement already satisfied: setuptools>=3.0 in /env/lib/python3.7/site-packages (from gunicorn==20.1.0->-r requirements.txt (line 2)) (49.6.0)
Step #2: Collecting proto-plus>=1.4.0
Step #2:   Downloading proto_plus-1.23.0-py3-none-any.whl (48 kB)
Step #2: Collecting libcst>=0.2.5
Step #2:   Downloading libcst-1.0.1-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.9 MB)
Step #2: Collecting google-api-core[grpc]<2.0.0dev,>=1.22.2
Step #2:   Downloading google_api_core-1.34.1-py3-none-any.whl (120 kB)
Step #2: Collecting protobuf
Step #2:   Downloading protobuf-4.24.4-cp37-abi3-manylinux2014_x86_64.whl (311 kB)
Step #2: Collecting google-cloud-core<3.0dev,>=1.6.0
Step #2:   Downloading google_cloud_core-2.4.1-py2.py3-none-any.whl (29 kB)
Step #2: Collecting requests<3.0.0dev,>=2.18.0
Step #2:   Downloading requests-2.31.0-py3-none-any.whl (62 kB)
Step #2: Collecting google-resumable-media>=1.3.0
Step #2:   Downloading google_resumable_media-2.7.0-py2.py3-none-any.whl (80 kB)
Step #2: Collecting google-auth<3.0dev,>=1.25.0
Step #2:   Downloading google_auth-2.29.0-py2.py3-none-any.whl (189 kB)
Step #2: Collecting importlib-metadata; python_version < "3.8"
Step #2:   Downloading importlib_metadata-6.7.0-py3-none-any.whl (22 kB)
Step #2: Collecting MarkupSafe>=2.0
Step #2:   Downloading MarkupSafe-2.1.5-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Step #2: Collecting typing-extensions>=3.7.4.2
Step #2:   Downloading typing_extensions-4.7.1-py3-none-any.whl (33 kB)
Step #2: Collecting typing-inspect>=0.4.0
Step #2:   Downloading typing_inspect-0.9.0-py3-none-any.whl (8.8 kB)
Step #2: Collecting pyyaml>=5.2
Step #2:   Downloading PyYAML-6.0.1-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (670 kB)
Step #2: Collecting googleapis-common-protos<2.0dev,>=1.56.2
Step #2:   Downloading googleapis_common_protos-1.63.0-py2.py3-none-any.whl (229 kB)
Step #2: Collecting grpcio-status<2.0dev,>=1.33.2; extra == "grpc"
Step #2:   Downloading grpcio_status-1.62.2-py3-none-any.whl (14 kB)
Step #2: Collecting grpcio<2.0dev,>=1.33.2; extra == "grpc"
Step #2:   Downloading grpcio-1.62.2-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (5.6 MB)
Step #2: Collecting idna<4,>=2.5
Step #2:   Downloading idna-3.7-py3-none-any.whl (66 kB)
Step #2: Collecting certifi>=2017.4.17
Step #2:   Downloading certifi-2024.2.2-py3-none-any.whl (163 kB)
Step #2: Collecting charset-normalizer<4,>=2
Step #2:   Downloading charset_normalizer-3.3.2-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (136 kB)
Step #2: Collecting google-crc32c<2.0dev,>=1.0
Step #2:   Downloading google_crc32c-1.5.0-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (32 kB)
Step #2: Collecting rsa<5,>=3.1.4
Step #2:   Downloading rsa-4.9-py3-none-any.whl (34 kB)
Step #2: Collecting pyasn1-modules>=0.2.1
Step #2:   Downloading pyasn1_modules-0.3.0-py2.py3-none-any.whl (181 kB)
Step #2: Collecting cachetools<6.0,>=2.0.0
Step #2:   Downloading cachetools-5.3.3-py3-none-any.whl (9.3 kB)
Step #2: Collecting zipp>=0.5
Step #2:   Downloading zipp-3.15.0-py3-none-any.whl (6.8 kB)
Step #2: Collecting mypy-extensions>=0.3.0
Step #2:   Downloading mypy_extensions-1.0.0-py3-none-any.whl (4.7 kB)
Step #2: Collecting pyasn1>=0.1.3
Step #2:   Downloading pyasn1-0.5.1-py2.py3-none-any.whl (84 kB)
Step #2: Installing collected packages: typing-extensions, zipp, importlib-metadata, click, itsdangerous, MarkupSafe, Jinja2, Werkzeug, Flask, gunicorn, protobuf, proto-plus, mypy-extensions, typing-inspect, pyyaml, libcst, googleapis-common-protos, idna, certifi, charset-normalizer, urllib3, requests, pyasn1, rsa, pyasn1-modules, cachetools, google-auth, grpcio, grpcio-status, google-api-core, google-cloud-vision, google-cloud-core, google-crc32c, google-resumable-media, google-cloud-storage, google-cloud-datastore
Step #2: Successfully installed Flask-2.0.3 Jinja2-3.1.4 MarkupSafe-2.1.5 Werkzeug-2.0.3 cachetools-5.3.3 certifi-2024.2.2 charset-normalizer-3.3.2 click-8.1.7 google-api-core-1.34.1 google-auth-2.29.0 google-cloud-core-2.4.1 google-cloud-datastore-2.7.1 google-cloud-storage-2.1.0 google-cloud-vision-2.1.0 google-crc32c-1.5.0 google-resumable-media-2.7.0 googleapis-common-protos-1.63.0 grpcio-1.62.2 grpcio-status-1.62.2 gunicorn-20.1.0 idna-3.7 importlib-metadata-6.7.0 itsdangerous-2.1.2 libcst-1.0.1 mypy-extensions-1.0.0 proto-plus-1.23.0 protobuf-4.24.4 pyasn1-0.5.1 pyasn1-modules-0.3.0 pyyaml-6.0.1 requests-2.31.0 rsa-4.9 typing-extensions-4.7.1 typing-inspect-0.9.0 urllib3-1.26.18 zipp-3.15.0
Step #2: ERROR: After October 2020 you may experience errors when installing or updating packages. This is because pip will change the way that it resolves dependency conflicts.
Step #2: 
Step #2: We recommend you use --use-feature=2020-resolver to test your packages with the new resolver before it becomes the default.
Step #2: 
Step #2: google-api-core 1.34.1 requires protobuf!=3.20.0,!=3.20.1,!=4.21.0,!=4.21.1,!=4.21.2,!=4.21.3,!=4.21.4,!=4.21.5,<4.0.0dev,>=3.19.5, but you'll have protobuf 4.24.4 which is incompatible.
Step #2: google-cloud-datastore 2.7.1 requires protobuf<4.0.0dev,>=3.19.0, but you'll have protobuf 4.24.4 which is incompatible.
Step #2: WARNING: You are using pip version 20.2.2; however, version 24.0 is available.
Step #2: You should consider upgrading via the '/env/bin/python -m pip install --upgrade pip' command.
Step #2: Removing intermediate container 14fa680c9498
Step #2:  6f3dff300654
Step #2: Step 8/9 : ADD . /app/
Step #2:  6385abddb99d
Step #2: Step 9/9 : CMD exec gunicorn -b :$PORT main:app
Step #2:  Running in 834e81ada53a
Step #2: Removing intermediate container 834e81ada53a
Step #2:  eb4b6d9aa67d
Step #2: Successfully built eb4b6d9aa67d
Step #2: Successfully tagged us.gcr.io/qwiklabs-gcp-02-b31031997333/appengine/default.20240517t162117:latest
Finished Step #2
PUSH
Pushing us.gcr.io/qwiklabs-gcp-02-b31031997333/appengine/default.20240517t162117
The push refers to repository [us.gcr.io/qwiklabs-gcp-02-b31031997333/appengine/default.20240517t162117]
638479c96750: Preparing
b50b732245f5: Preparing
13ba1e30fa6c: Preparing
d21eb8ed5f2f: Preparing
087d7553d285: Preparing
16919ab89eca: Preparing
74bcef7f7402: Preparing
bc9e931c388e: Preparing
20896f2c3dd8: Preparing
7b80c69caf34: Preparing
3bbec54fac0c: Preparing
4006ffa4c683: Preparing
844d958e8cbe: Preparing
84ff92691f90: Preparing
b49bce339f97: Preparing
dcb7197db903: Preparing
3bbec54fac0c: Waiting
4006ffa4c683: Waiting
844d958e8cbe: Waiting
16919ab89eca: Waiting
74bcef7f7402: Waiting
bc9e931c388e: Waiting
20896f2c3dd8: Waiting
7b80c69caf34: Waiting
b49bce339f97: Waiting
dcb7197db903: Waiting
087d7553d285: Layer already exists
16919ab89eca: Layer already exists
74bcef7f7402: Layer already exists
bc9e931c388e: Layer already exists
13ba1e30fa6c: Pushed
20896f2c3dd8: Layer already exists
7b80c69caf34: Layer already exists
3bbec54fac0c: Layer already exists
4006ffa4c683: Layer already exists
844d958e8cbe: Layer already exists
84ff92691f90: Layer already exists
dcb7197db903: Layer already exists
d21eb8ed5f2f: Pushed
638479c96750: Pushed
b50b732245f5: Pushed
latest: digest: sha256:8a6bb61bab6f8c12ca1f16d8370d83f51488284b4ba42183b8370fc483a0aba3 size: 3674
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...working.                                       Updating service [default] (this may take several minutes)...done.
Setting traffic split for service [default]...done.                                                                                         
Deployed service [default] to [https://qwiklabs-gcp-02-b31031997333.uw.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

```sh
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-02-b31031997333.uw.r.appspot.com
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

Congratulations! In this lab, you learned how to write and deploy a  Python web application to the App Engine Flexible environment!

## Summary

```sh
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ history 
    1  gcloud storage cp -r gs://spls/gsp023/flex_and_vision/ .
    2  cd flex_and_vision
    3  export PROJECT_ID=$(gcloud config get-value project)
    4  gcloud iam service-accounts create qwiklab   --display-name "My Qwiklab Service Account"
    5  gcloud projects add-iam-policy-binding ${PROJECT_ID} --member serviceAccount:qwiklab@${PROJECT_ID}.iam.gserviceaccount.com --role roles/owner
    6  gcloud iam service-accounts keys create ~/key.json --iam-account qwiklab@${PROJECT_ID}.iam.gserviceaccount.com
    7  export GOOGLE_APPLICATION_CREDENTIALS="/home/${USER}/key.json"
    8  ls
    9  cat ~/key.json 
   10  virtualenv -p python3 env
   11  source env/bin/activate
   12  pip install -r requirements.txt
   13  AE_REGION=us-west1
   14  gcloud app create --region=$AE_REGION
   15  export CLOUD_STORAGE_BUCKET=${PROJECT_ID}
   16  gsutil mb gs://${PROJECT_ID}
   17  python main.py
   18  ls
   19  cat main.py 
   20  ls
   21  cat main_test.py 
   22  ls
   23  cat app.yaml 
   24  cat templates/homepage.html 
   25  vi app.yaml 
   26  cat app.yaml 
   27  vi app.yaml 
   28  cat app.yaml 
   29  gcloud config set app/cloud_build_timeout 1000
   30  gcloud app deploy
   31  gcloud app browse
   32  history 
(env) student_04_bf71fa739d08@cloudshell:~/flex_and_vision (qwiklabs-gcp-02-b31031997333)$ 
```

