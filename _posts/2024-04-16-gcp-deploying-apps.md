---
layout: single
title:  "Deploying Apps to GCP"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## Deploying Apps to GCP
```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-bb788c0cc1a1.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_5f3e8b93b23f@cloudshell:~ (qwiklabs-gcp-01-bb788c0cc1a1)$ mkdir gcp-course
student_01_5f3e8b93b23f@cloudshell:~ (qwiklabs-gcp-01-bb788c0cc1a1)$ cd gcp-course
student_01_5f3e8b93b23f@cloudshell:~/gcp-course (qwiklabs-gcp-01-bb788c0cc1a1)$ git clone https://GitHub.com/GoogleCloudPlatform/training-data-analyst.git
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64692, done.
remote: Counting objects: 100% (116/116), done.
remote: Compressing objects: 100% (68/68), done.
remote: Total 64692 (delta 60), reused 92 (delta 46), pack-reused 64576
Receiving objects: 100% (64692/64692), 698.01 MiB | 16.24 MiB/s, done.
Resolving deltas: 100% (41316/41316), done.
Updating files: 100% (12864/12864), done.
student_01_5f3e8b93b23f@cloudshell:~/gcp-course (qwiklabs-gcp-01-bb788c0cc1a1)$ cd training-data-analyst/courses/design-process/deploying-apps-to-gcp
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ ls
Dockerfile  main.py  requirements.txt  templates
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat Dockerfile 
FROM python:3.7
WORKDIR /app
COPY . .
RUN pip install gunicorn
RUN pip install -r requirements.txt
ENV PORT=8080
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ docker build -t test-python .
[+] Building 33.5s (10/10) FINISHED                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                      0.1s
 => => transferring dockerfile: 217B                                                                                                      0.0s
 => [internal] load metadata for docker.io/library/python:3.7                                                                             0.5s
 => [internal] load .dockerignore                                                                                                         0.0s
 => => transferring context: 2B                                                                                                           0.0s
 => [1/5] FROM docker.io/library/python:3.7@sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0                      25.0s
 => => resolve docker.io/library/python:3.7@sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0                       0.0s
 => => sha256:167b8a53ca4504bc6aa3182e336fa96f4ef76875d158c1933d3e2fa19c57e0c3 49.56MB / 49.56MB                                          0.9s
 => => sha256:debce5f9f3a9709885f7f2ad3cf41f036a3b57b406b27ba3a883928315787042 64.11MB / 64.11MB                                          1.3s
 => => sha256:2011a37d2a08fe83dd9ff923e0f83bfd7290152e2e6afe359bde1453170d9bdc 2.01kB / 2.01kB                                            0.0s
 => => sha256:16d93ae3411be3db255b6b52fdfc155a0dff0f697c2e4e3d862caf8d978830b2 8.13kB / 8.13kB                                            0.0s
 => => sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0 1.86kB / 1.86kB                                            0.0s
 => => sha256:b47a222d28fa95680198398973d0a29b82a968f03e7ef361cc8ded562e4d84a3 24.03MB / 24.03MB                                          0.6s
 => => sha256:1d7ca7cd2e066ae77ac6284a9d027f72a31a02a18bfc2a249ef2e7b01074338b 211.04MB / 211.04MB                                        4.2s
 => => sha256:ff3119008f58beef8f336fa833707b0fe914db94ca6b7bb55abe3e1bf2b1ad56 6.39MB / 6.39MB                                            1.3s
 => => extracting sha256:167b8a53ca4504bc6aa3182e336fa96f4ef76875d158c1933d3e2fa19c57e0c3                                                 4.3s
 => => sha256:c2423a76a32b7ffb2ee7bb6d1e0c74bb1811237eddcb3200594daf7a52d4f378 14.70MB / 14.70MB                                          1.8s
 => => sha256:e1c98ca4926a91839805ce76d76a70225e303007331ee60f45dfabbbf55fd8c8 244B / 244B                                                1.4s
 => => sha256:3b62c8e1d79b6554a8bffcf196ff5dd822858c179f1f8dc6f0c74a288859a6fb 2.85MB / 2.85MB                                            1.7s
 => => extracting sha256:b47a222d28fa95680198398973d0a29b82a968f03e7ef361cc8ded562e4d84a3                                                 1.0s
 => => extracting sha256:debce5f9f3a9709885f7f2ad3cf41f036a3b57b406b27ba3a883928315787042                                                 4.3s
 => => extracting sha256:1d7ca7cd2e066ae77ac6284a9d027f72a31a02a18bfc2a249ef2e7b01074338b                                                11.4s
 => => extracting sha256:ff3119008f58beef8f336fa833707b0fe914db94ca6b7bb55abe3e1bf2b1ad56                                                 0.4s
 => => extracting sha256:c2423a76a32b7ffb2ee7bb6d1e0c74bb1811237eddcb3200594daf7a52d4f378                                                 0.8s
 => => extracting sha256:e1c98ca4926a91839805ce76d76a70225e303007331ee60f45dfabbbf55fd8c8                                                 0.0s
 => => extracting sha256:3b62c8e1d79b6554a8bffcf196ff5dd822858c179f1f8dc6f0c74a288859a6fb                                                 0.4s
 => [internal] load build context                                                                                                         0.0s
 => => transferring context: 1.30kB                                                                                                       0.0s
 => [2/5] WORKDIR /app                                                                                                                    1.2s
 => [3/5] COPY . .                                                                                                                        0.0s
 => [4/5] RUN pip install gunicorn                                                                                                        3.9s
 => [5/5] RUN pip install -r requirements.txt                                                                                             2.5s
 => exporting to image                                                                                                                    0.3s
 => => exporting layers                                                                                                                   0.2s
 => => writing image sha256:53fe95a8af26f5c079184976d58c8475203f8451843d41f505feb14ab7580a85                                              0.0s
 => => naming to docker.io/library/test-python                                                                                            0.0s
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ docker build -t test-python .
[+] Building 0.1s (10/10) FINISHED                                                                                              docker:default
 => [internal] load build definition from Dockerfile                                                                                      0.0s
 => => transferring dockerfile: 217B                                                                                                      0.0s
 => [internal] load metadata for docker.io/library/python:3.7                                                                             0.1s
 => [internal] load .dockerignore                                                                                                         0.0s
 => => transferring context: 2B                                                                                                           0.0s
 => [1/5] FROM docker.io/library/python:3.7@sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0                       0.0s
 => [internal] load build context                                                                                                         0.0s
 => => transferring context: 204B                                                                                                         0.0s
 => CACHED [2/5] WORKDIR /app                                                                                                             0.0s
 => CACHED [3/5] COPY . .                                                                                                                 0.0s
 => CACHED [4/5] RUN pip install gunicorn                                                                                                 0.0s
 => CACHED [5/5] RUN pip install -r requirements.txt                                                                                      0.0s
 => exporting to image                                                                                                                    0.0s
 => => exporting layers                                                                                                                   0.0s
 => => writing image sha256:53fe95a8af26f5c079184976d58c8475203f8451843d41f505feb14ab7580a85                                              0.0s
 => => naming to docker.io/library/test-python                                                                                            0.0s
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ docker run --rm -p 8080:8080 test-python
[2024-04-16 13:47:16 +0000] [1] [INFO] Starting gunicorn 21.2.0
[2024-04-16 13:47:16 +0000] [1] [INFO] Listening at: http://0.0.0.0:8080 (1)
[2024-04-16 13:47:16 +0000] [1] [INFO] Using worker: gthread
[2024-04-16 13:47:16 +0000] [9] [INFO] Booting worker with pid: 9

```

## Deploy to App Engine
```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ ls
app.yaml  Dockerfile  main.py  requirements.txt  templates
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat app.yaml 
runtime: python39student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat app.yaml 
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud app create --region=us-east1--region=us-east1
You are creating an app for project [qwiklabs-gcp-01-bb788c0cc1a1].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Creating App Engine application in project [qwiklabs-gcp-01-bb788c0cc1a1] and region [us-east1]....done.                                                                           
Success! The app is now created. Please use `gcloud app deploy` to deploy your first app.
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud app deploy --version=one --quiet
Services to deploy:

descriptor:                  [/home/student_01_5f3e8b93b23f/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp/app.yaml]
source:                      [/home/student_01_5f3e8b93b23f/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp]
target project:              [qwiklabs-gcp-01-bb788c0cc1a1]
target service:              [default]
target version:              [one]
target url:                  [https://qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-01-bb788c0cc1a1@appspot.gserviceaccount.com]


Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 7 files to Google Cloud Storage
14%
29%
43%
57%
71%
86%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```html
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ curl https://qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com/
<!doctype html>
<html lang="en">
<head>
    <title>Hello GCP.</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">

 
</head>
<body>
    <div class="container">

        
<div class="jumbotron">
    <div class="container">
        <h1>Hello GCP.</h1>
    </div>
</div>


        <footer></footer>
    </div>
</body>
</html>student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



Modify the contents

```py
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat main.py 
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route("/")
def main():
    model = {"title": "Hello App Engine"}
    return render_template('index.html', model=model)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080, debug=True, threaded=True)student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

Deploy the modified version

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud app deploy --version=two --no-promote --quiet
Services to deploy:

descriptor:                  [/home/student_01_5f3e8b93b23f/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp/app.yaml]
source:                      [/home/student_01_5f3e8b93b23f/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp]
target project:              [qwiklabs-gcp-01-bb788c0cc1a1]
target service:              [default]
target version:              [two]
target url:                  [https://two-dot-qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-01-bb788c0cc1a1@appspot.gserviceaccount.com]


     (add --promote if you also want to make this service available from
     [https://qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com])

Beginning deployment of service [default]...
Uploading 1 file to Google Cloud Storage
100%
100%
File upload done.
Updating service [default]...done.                                                                                                          
Deployed service [default] to [https://two-dot-qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



> **Note:** The `--no-promote` parameter tells App Engine to continue serving requests with the old version. This allows  you to test the new version before putting it into production.



```html
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ curl https://two-dot-qwiklabs-gcp-01-bb788c0cc1a1.ue.r.appspot.com/
<!doctype html>
<html lang="en">
<head>
    <title>Hello App Engine</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">

 
</head>
<body>
    <div class="container">

        
<div class="jumbotron">
    <div class="container">
        <h1>Hello App Engine</h1>
    </div>
</div>


        <footer></footer>
    </div>
</body>
</html>student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

To migrate production traffic to version two, click  **Split Traffic** at the top. Change the version to two, and click **Save**.

You can split incoming traffic to different versions of your app.  Traffic splitting is useful for slowly rolling out new versions or A/B  testing different designs and features.



## Deploy to Kubernetes Engine with Cloud Build and Artifact Registry

Standard

```sh
gcloud beta container --project "qwiklabs-gcp-01-bb788c0cc1a1" clusters create "cluster-1" --no-enable-basic-auth --cluster-version "1.27.8-gke.1067004" --release-channel "regular" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/qwiklabs-gcp-01-bb788c0cc1a1/global/networks/default" --subnetwork "projects/qwiklabs-gcp-01-bb788c0cc1a1/regions/us-east1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --security-posture=standard --workload-vulnerability-scanning=disabled --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --binauthz-evaluation-mode=DISABLED --enable-managed-prometheus --enable-shielded-nodes
```

Autopilot

```sh
gcloud beta container --project "qwiklabs-gcp-01-bb788c0cc1a1" clusters create-auto "autopilot-cluster-1" --region "us-east1" --release-channel "regular" --network "projects/qwiklabs-gcp-01-bb788c0cc1a1/global/networks/default" --subnetwork "projects/qwiklabs-gcp-01-bb788c0cc1a1/regions/us-east1/subnetworks/default" --cluster-ipv4-cidr "/17" --binauthz-evaluation-mode=DISABLED
```



```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud container clusters get-credentials autopilot-cluster-1 --region us-east1 --project qwiklabs-gcp-01-bb788c0cc1a1
Fetching cluster endpoint and auth data.
kubeconfig entry generated for autopilot-cluster-1.
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ kubectl get nodes
E0416 14:31:37.129806    2246 memcache.go:287] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:31:37.798779    2246 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:31:38.020964    2246 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:31:38.242783    2246 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
NAME                                                 STATUS   ROLES    AGE    VERSION
gk3-autopilot-cluster-1-default-pool-763b25dd-fm15   Ready    <none>   3m5s   v1.27.8-gke.1067004
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



```py
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat main.py 
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route("/")
def main():
    model = {"title": "Hello Kubernetes Engine"}
    return render_template('index.html', model=model)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080, debug=True, threaded=True)student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



Enter the following command to create an Artifact Registry repository named devops-demo:



```
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud artifacts repositories create devops-demo \
    --repository-format=docker \
    --location=us-east1
Create request issued for: [devops-demo]
Waiting for operation [projects/qwiklabs-gcp-01-bb788c0cc1a1/locations/us-east1/operations/c9c1c5a3-e3d7-4bf7-80c4-07da60d7886a] to complete
...done.                                                                                                                                    
Created repository [devops-demo].
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ gcloud auth configure-docker us-east1-docker.pkg.dev
WARNING: Your config file at [/home/student_01_5f3e8b93b23f/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "africa-south1-docker.pkg.dev": "gcloud",
    "asia-docker.pkg.dev": "gcloud",
    "asia-east1-docker.pkg.dev": "gcloud",
    "asia-east2-docker.pkg.dev": "gcloud",
    "asia-northeast1-docker.pkg.dev": "gcloud",
    "asia-northeast2-docker.pkg.dev": "gcloud",
    "asia-northeast3-docker.pkg.dev": "gcloud",
    "asia-south1-docker.pkg.dev": "gcloud",
    "asia-south2-docker.pkg.dev": "gcloud",
    "asia-southeast1-docker.pkg.dev": "gcloud",
    "asia-southeast2-docker.pkg.dev": "gcloud",
    "australia-southeast1-docker.pkg.dev": "gcloud",
    "australia-southeast2-docker.pkg.dev": "gcloud",
    "europe-docker.pkg.dev": "gcloud",
    "europe-central2-docker.pkg.dev": "gcloud",
    "europe-north1-docker.pkg.dev": "gcloud",
    "europe-southwest1-docker.pkg.dev": "gcloud",
    "europe-west1-docker.pkg.dev": "gcloud",
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud"
  }
}
Adding credentials for: us-east1-docker.pkg.dev
gcloud credential helpers already registered correctly.
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

To use Kubernetes Engine, you need to build a Docker image. Enter the  following commands to use Cloud Build to create the image and store it  in Artifact Registry:

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cd ~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp
gcloud builds submit --tag us-east1-docker.pkg.dev/$DEVSHELL_PROJECT_ID/devops-demo/devops-image:v0.2 .
Creating temporary tarball archive of 7 file(s) totalling 1.7 KiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278104.542225-012afe3ee1bd4e849319138d19d2d19c.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-bb788c0cc1a1/locations/global/builds/19db1348-ffaf-4a07-a1aa-4689cc4d07b6].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/19db1348-ffaf-4a07-a1aa-4689cc4d07b6?project=1040999101836 ].
------------------------------------------------------------ REMOTE BUILD OUTPUT ------------------------------------------------------------
starting build "19db1348-ffaf-4a07-a1aa-4689cc4d07b6"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278104.542225-012afe3ee1bd4e849319138d19d2d19c.tgz#1713278105482769
Copying gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278104.542225-012afe3ee1bd4e849319138d19d2d19c.tgz#1713278105482769...
/ [1 files][  1.3 KiB/  1.3 KiB]                                                
Operation completed over 1 objects/1.3 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  9.216kB
Step 1/7 : FROM python:3.7
3.7: Pulling from library/python
167b8a53ca45: Pulling fs layer
b47a222d28fa: Pulling fs layer
debce5f9f3a9: Pulling fs layer
1d7ca7cd2e06: Pulling fs layer
ff3119008f58: Pulling fs layer
c2423a76a32b: Pulling fs layer
e1c98ca4926a: Pulling fs layer
3b62c8e1d79b: Pulling fs layer
1d7ca7cd2e06: Waiting
ff3119008f58: Waiting
c2423a76a32b: Waiting
e1c98ca4926a: Waiting
3b62c8e1d79b: Waiting
b47a222d28fa: Verifying Checksum
b47a222d28fa: Download complete
167b8a53ca45: Verifying Checksum
167b8a53ca45: Download complete
debce5f9f3a9: Verifying Checksum
debce5f9f3a9: Download complete
ff3119008f58: Verifying Checksum
ff3119008f58: Download complete
c2423a76a32b: Verifying Checksum
c2423a76a32b: Download complete
e1c98ca4926a: Verifying Checksum
e1c98ca4926a: Download complete
3b62c8e1d79b: Verifying Checksum
3b62c8e1d79b: Download complete
1d7ca7cd2e06: Verifying Checksum
1d7ca7cd2e06: Download complete
167b8a53ca45: Pull complete
b47a222d28fa: Pull complete
debce5f9f3a9: Pull complete
1d7ca7cd2e06: Pull complete
ff3119008f58: Pull complete
c2423a76a32b: Pull complete
e1c98ca4926a: Pull complete
3b62c8e1d79b: Pull complete
Digest: sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0
Status: Downloaded newer image for python:3.7
 16d93ae3411b
Step 2/7 : WORKDIR /app
 Running in ac2f451b892b
Removing intermediate container ac2f451b892b
 3d0749d5b443
Step 3/7 : COPY . .
 712fef20c8b9
Step 4/7 : RUN pip install gunicorn
 Running in e6e949a3fd5e
Collecting gunicorn
  Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 80.2/80.2 kB 3.4 MB/s eta 0:00:00
Collecting importlib-metadata
  Downloading importlib_metadata-6.7.0-py3-none-any.whl (22 kB)
Collecting packaging
  Downloading packaging-24.0-py3-none-any.whl (53 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 53.5/53.5 kB 8.1 MB/s eta 0:00:00
Collecting zipp>=0.5
  Downloading zipp-3.15.0-py3-none-any.whl (6.8 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.7.1-py3-none-any.whl (33 kB)
Installing collected packages: zipp, typing-extensions, packaging, importlib-metadata, gunicorn
Successfully installed gunicorn-21.2.0 importlib-metadata-6.7.0 packaging-24.0 typing-extensions-4.7.1 zipp-3.15.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container e6e949a3fd5e
 0d3ca5c022cc
Step 5/7 : RUN pip install -r requirements.txt
 Running in afe894fb7538
Collecting Flask==2.0.3
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.6/95.6 kB 3.4 MB/s eta 0:00:00
Collecting itsdangerous==2.0.1
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2==3.0.3
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.6/133.6 kB 14.0 MB/s eta 0:00:00
Collecting werkzeug==2.2.2
  Downloading Werkzeug-2.2.2-py3-none-any.whl (232 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 232.7/232.7 kB 24.1 MB/s eta 0:00:00
Collecting click>=7.1.2
  Downloading click-8.1.7-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.9/97.9 kB 14.5 MB/s eta 0:00:00
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.5-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Requirement already satisfied: importlib-metadata in /usr/local/lib/python3.7/site-packages (from click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (6.7.0)
Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.7/site-packages (from importlib-metadata->click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (3.15.0)
Requirement already satisfied: typing-extensions>=3.6.4 in /usr/local/lib/python3.7/site-packages (from importlib-metadata->click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (4.7.1)
Installing collected packages: MarkupSafe, itsdangerous, werkzeug, Jinja2, click, Flask
Successfully installed Flask-2.0.3 Jinja2-3.0.3 MarkupSafe-2.1.5 click-8.1.7 itsdangerous-2.0.1 werkzeug-2.2.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container afe894fb7538
 6417bc6e2aea
Step 6/7 : ENV PORT=8080
 Running in 066e651a7b8e
Removing intermediate container 066e651a7b8e
 be6f8eeb253d
Step 7/7 : CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
 Running in e0623f111c2a
Removing intermediate container e0623f111c2a
 8d401de749dd
Successfully built 8d401de749dd
Successfully tagged us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/devops-image:v0.2
PUSH
Pushing us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/devops-image:v0.2
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/devops-image]
ce76e1f5a3dd: Preparing
822d7d481a7e: Preparing
6989955c91c4: Preparing
fd9e2cf25b63: Preparing
45c430b35dba: Preparing
8e23f007f16f: Preparing
aef22e07d5d7: Preparing
c26432533a6a: Preparing
01d6cdeac539: Preparing
a981dddd4c65: Preparing
f6589095d5b5: Preparing
7c85cfa30cb1: Preparing
8e23f007f16f: Waiting
aef22e07d5d7: Waiting
c26432533a6a: Waiting
01d6cdeac539: Waiting
7c85cfa30cb1: Waiting
a981dddd4c65: Waiting
f6589095d5b5: Waiting
6989955c91c4: Pushed
fd9e2cf25b63: Pushed
ce76e1f5a3dd: Pushed
822d7d481a7e: Pushed
8e23f007f16f: Pushed
45c430b35dba: Pushed
c26432533a6a: Pushed
aef22e07d5d7: Pushed
f6589095d5b5: Pushed
a981dddd4c65: Pushed
7c85cfa30cb1: Pushed
01d6cdeac539: Pushed
v0.2: digest: sha256:1c9c50f18fc1425d56b91fbba5621739f68344b91d5bdaa01ea09ab8eae7154b size: 2845
DONE
---------------------------------------------------------------------------------------------------------------------------------------------
ID: 19db1348-ffaf-4a07-a1aa-4689cc4d07b6
CREATE_TIME: 2024-04-16T14:35:05+00:00
DURATION: 1M32S
SOURCE: gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278104.542225-012afe3ee1bd4e849319138d19d2d19c.tgz
IMAGES: us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/devops-image:v0.2
STATUS: SUCCESS
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```yaml
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat kubernetes-config.yaml 
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-deployment
  labels:
    app: devops
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devops
      tier: frontend
  template:
    metadata:
      labels:
        app: devops
        tier: frontend
    spec:
      containers:
      - name: devops-demo
        image: us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/devops-image:v0.2
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: devops-deployment-lb
  labels:
    app: devops
    tier: frontend-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: devops
    tier: frontendstudent_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ kubectl apply -f kubernetes-config.yaml
E0416 14:38:30.408666    2523 memcache.go:287] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:38:30.933552    2523 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
Warning: autopilot-default-resources-mutator:Autopilot updated Deployment default/devops-deployment: defaulted unspecified resources for containers [devops-demo] (see http://g.co/gke/autopilot-defaults)
deployment.apps/devops-deployment created
service/devops-deployment-lb created
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ kubectl get pods
E0416 14:40:57.619465    2724 memcache.go:287] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:40:58.064156    2724 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:40:58.287184    2724 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:40:58.519194    2724 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
NAME                                 READY   STATUS    RESTARTS   AGE
devops-deployment-576b47ff4c-4kh9m   1/1     Running   0          2m26s
devops-deployment-576b47ff4c-6zj8q   1/1     Running   0          2m26s
devops-deployment-576b47ff4c-d4n75   1/1     Running   0          2m26s
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ kubectl get svc
E0416 14:41:04.558286    2744 memcache.go:287] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:41:05.227340    2744 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:41:05.449725    2744 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
E0416 14:41:05.676756    2744 memcache.go:121] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
devops-deployment-lb   LoadBalancer   34.118.232.36   35.231.192.251   80:30890/TCP   2m33s
kubernetes             ClusterIP      34.118.224.1    <none>           443/TCP        14m
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ curl 35.231.192.251
<!doctype html>
<html lang="en">
<head>
    <title>Hello Kubernetes Engine</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">

 
</head>
<body>
    <div class="container">

        
<div class="jumbotron">
    <div class="container">
        <h1>Hello Kubernetes Engine</h1>
    </div>
</div>


        <footer></footer>
    </div>
</body>
</html>student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```



## Deploy to Cloud Run

```py
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cat main.py 
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route("/")
def main():
    model = {"title": "Hello Cloud Run"}
    return render_template('index.html', model=model)


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080, debug=True, threaded=True)student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```sh
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ cd ~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp
gcloud builds submit --tag us-east1-docker.pkg.dev/$DEVSHELL_PROJECT_ID/devops-demo/cloud-run-image:v0.1 .
Creating temporary tarball archive of 7 file(s) totalling 1.8 KiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278591.429462-f94af23c35f24742bee9c35169442d3d.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-bb788c0cc1a1/locations/global/builds/ae67cd13-16ff-4f69-b15f-c8857e7eebd3].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/ae67cd13-16ff-4f69-b15f-c8857e7eebd3?project=1040999101836 ].
------------------------------------------------------------ REMOTE BUILD OUTPUT ------------------------------------------------------------
starting build "ae67cd13-16ff-4f69-b15f-c8857e7eebd3"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278591.429462-f94af23c35f24742bee9c35169442d3d.tgz#1713278592448427
Copying gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278591.429462-f94af23c35f24742bee9c35169442d3d.tgz#1713278592448427...
/ [1 files][  1.3 KiB/  1.3 KiB]                                                
Operation completed over 1 objects/1.3 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  9.216kB
Step 1/7 : FROM python:3.7
3.7: Pulling from library/python
167b8a53ca45: Pulling fs layer
b47a222d28fa: Pulling fs layer
debce5f9f3a9: Pulling fs layer
1d7ca7cd2e06: Pulling fs layer
ff3119008f58: Pulling fs layer
c2423a76a32b: Pulling fs layer
e1c98ca4926a: Pulling fs layer
3b62c8e1d79b: Pulling fs layer
ff3119008f58: Waiting
c2423a76a32b: Waiting
e1c98ca4926a: Waiting
3b62c8e1d79b: Waiting
1d7ca7cd2e06: Waiting
b47a222d28fa: Verifying Checksum
b47a222d28fa: Download complete
167b8a53ca45: Verifying Checksum
167b8a53ca45: Download complete
debce5f9f3a9: Verifying Checksum
debce5f9f3a9: Download complete
ff3119008f58: Verifying Checksum
ff3119008f58: Download complete
e1c98ca4926a: Verifying Checksum
e1c98ca4926a: Download complete
c2423a76a32b: Verifying Checksum
c2423a76a32b: Download complete
3b62c8e1d79b: Verifying Checksum
3b62c8e1d79b: Download complete
1d7ca7cd2e06: Verifying Checksum
1d7ca7cd2e06: Download complete
167b8a53ca45: Pull complete
b47a222d28fa: Pull complete
debce5f9f3a9: Pull complete
1d7ca7cd2e06: Pull complete
ff3119008f58: Pull complete
c2423a76a32b: Pull complete
e1c98ca4926a: Pull complete
3b62c8e1d79b: Pull complete
Digest: sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0
Status: Downloaded newer image for python:3.7
 16d93ae3411b
Step 2/7 : WORKDIR /app
 Running in 12f82b619210
Removing intermediate container 12f82b619210
 e60c45d6874b
Step 3/7 : COPY . .
 73f2c19d0edf
Step 4/7 : RUN pip install gunicorn
 Running in 235244886a18
Collecting gunicorn
  Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 80.2/80.2 kB 1.5 MB/s eta 0:00:00
Collecting importlib-metadata
  Downloading importlib_metadata-6.7.0-py3-none-any.whl (22 kB)
Collecting packaging
  Downloading packaging-24.0-py3-none-any.whl (53 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 53.5/53.5 kB 5.2 MB/s eta 0:00:00
Collecting zipp>=0.5
  Downloading zipp-3.15.0-py3-none-any.whl (6.8 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.7.1-py3-none-any.whl (33 kB)
Installing collected packages: zipp, typing-extensions, packaging, importlib-metadata, gunicorn
Successfully installed gunicorn-21.2.0 importlib-metadata-6.7.0 packaging-24.0 typing-extensions-4.7.1 zipp-3.15.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container 235244886a18
 401d8480ce76
Step 5/7 : RUN pip install -r requirements.txt
 Running in a938b93cdfa1
Collecting Flask==2.0.3
  Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.6/95.6 kB 1.4 MB/s eta 0:00:00
Collecting itsdangerous==2.0.1
  Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Collecting Jinja2==3.0.3
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.6/133.6 kB 6.1 MB/s eta 0:00:00
Collecting werkzeug==2.2.2
  Downloading Werkzeug-2.2.2-py3-none-any.whl (232 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 232.7/232.7 kB 12.9 MB/s eta 0:00:00
Collecting click>=7.1.2
  Downloading click-8.1.7-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.9/97.9 kB 15.8 MB/s eta 0:00:00
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.5-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Requirement already satisfied: importlib-metadata in /usr/local/lib/python3.7/site-packages (from click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (6.7.0)
Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.7/site-packages (from importlib-metadata->click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (3.15.0)
Requirement already satisfied: typing-extensions>=3.6.4 in /usr/local/lib/python3.7/site-packages (from importlib-metadata->click>=7.1.2->Flask==2.0.3->-r requirements.txt (line 1)) (4.7.1)
Installing collected packages: MarkupSafe, itsdangerous, werkzeug, Jinja2, click, Flask
Successfully installed Flask-2.0.3 Jinja2-3.0.3 MarkupSafe-2.1.5 click-8.1.7 itsdangerous-2.0.1 werkzeug-2.2.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container a938b93cdfa1
 cdcbb3dbeb07
Step 6/7 : ENV PORT=8080
 Running in 0524ee49e264
Removing intermediate container 0524ee49e264
 65a204001d24
Step 7/7 : CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
 Running in d522fe518370
Removing intermediate container d522fe518370
 b080850be631
Successfully built b080850be631
Successfully tagged us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/cloud-run-image:v0.1
PUSH
Pushing us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/cloud-run-image:v0.1
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/cloud-run-image]
a64a25e9902e: Preparing
18f863bdfa3b: Preparing
659109150e1e: Preparing
7f892e733fd1: Preparing
45c430b35dba: Preparing
8e23f007f16f: Preparing
aef22e07d5d7: Preparing
c26432533a6a: Preparing
01d6cdeac539: Preparing
a981dddd4c65: Preparing
f6589095d5b5: Preparing
7c85cfa30cb1: Preparing
c26432533a6a: Waiting
01d6cdeac539: Waiting
a981dddd4c65: Waiting
f6589095d5b5: Waiting
7c85cfa30cb1: Waiting
8e23f007f16f: Waiting
aef22e07d5d7: Waiting
45c430b35dba: Layer already exists
8e23f007f16f: Layer already exists
aef22e07d5d7: Layer already exists
c26432533a6a: Layer already exists
7f892e733fd1: Pushed
659109150e1e: Pushed
01d6cdeac539: Layer already exists
a64a25e9902e: Pushed
f6589095d5b5: Layer already exists
a981dddd4c65: Layer already exists
7c85cfa30cb1: Layer already exists
18f863bdfa3b: Pushed
v0.1: digest: sha256:a079f1658a68a67440a450c90a509b499a36c1b329a6bf29c86a79ead118c676 size: 2845
DONE
---------------------------------------------------------------------------------------------------------------------------------------------
ID: ae67cd13-16ff-4f69-b15f-c8857e7eebd3
CREATE_TIME: 2024-04-16T14:43:12+00:00
DURATION: 44S
SOURCE: gs://qwiklabs-gcp-01-bb788c0cc1a1_cloudbuild/source/1713278591.429462-f94af23c35f24742bee9c35169442d3d.tgz
IMAGES: us-east1-docker.pkg.dev/qwiklabs-gcp-01-bb788c0cc1a1/devops-demo/cloud-run-image:v0.1
STATUS: SUCCESS
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

```html
student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ curl https://hello-cloud-run-3zjcty2beq-ue.a.run.app/
<!doctype html>
<html lang="en">
<head>
    <title>Hello Cloud Run</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">

 
</head>
<body>
    <div class="container">

        
<div class="jumbotron">
    <div class="container">
        <h1>Hello Cloud Run</h1>
    </div>
</div>


        <footer></footer>
    </div>
</body>
</html>student_01_5f3e8b93b23f@cloudshell:~/gcp-course/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-01-bb788c0cc1a1)$ 
```

