---

layout: single
title:  "Continuous Delivery with Google Cloud Deploy"
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

# Continuous Delivery with Google Cloud Deploy

Google Cloud Deploy is a managed service that automates delivery of  your applications to a series of target environments in a defined  promotion sequence. When you want to deploy your updated application,  you create a release, whose lifecycle is managed by a delivery pipeline.

In this lab, you will create a delivery pipeline using Google Cloud  Deploy. You will then create a release for a basic application and  promote the application through a series of Google Kubernetes Engine  (GKE) targets.

The sample application is a simple web app that listens to a port,  provides an HTTP response code and adds a log entry. This lab is derived from a tutorial published by Google:  https://cloud.google.com/deploy/docs/tutorials.



- Deploy a container image to Google Cloud Artifact Registry using Skaffold
- Create a Google Cloud Deploy delivery pipeline
- Create a release for the delivery pipeline
- Promote the application through the targets in the delivery pipeline

## Set variables

- Declare the environment variables that will be used by various commands:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-aec191b9afa6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-east1
gcloud config set compute/region $REGION
Your active configuration is: [cloudshell-5846]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ 
```

## Create three GKE clusters

In this task you will create the three GKE clusters that will be targets for the delivery pipeline.

Three GKE clusters will be created, denoting the three targets for the delivery pipeline:

- **test**
- **staging**
- **prod**

```sh
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ gcloud services enable \
container.googleapis.com \
clouddeploy.googleapis.com
Operation "operations/acat.p2-613884732946-4478cc43-f3bf-46c1-9272-df6554c2a70f" finished successfully.
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ 
gcloud container clusters create test --node-locations=us-east1-c --num-nodes=1  --async
gcloud container clusters create staging --node-locations=us-east1-c --num-nodes=1  --async
gcloud container clusters create prod --node-locations=us-east1-c --num-nodes=1  --async
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
NAME: test
TYPE: 
LOCATION: us-east1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
NAME: staging
TYPE: 
LOCATION: us-east1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
NAME: prod
TYPE: 
LOCATION: us-east1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ 
```

```sh
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,PROVISIONING
staging,PROVISIONING
test,PROVISIONING
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ 
```

## Prepare the web application container image

In this task you'll create a repository in Artifact Registry to hold the web application's container images.

1. Enable the Artifact Registry API:

```sh
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ gcloud services enable artifactregistry.googleapis.com
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ gcloud artifacts repositories create web-app \
--description="Image registry for tutorial web app" \
--repository-format=docker \
--location=$REGION
Create request issued for: [web-app]
Waiting for operation [projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/operations/d54df320-b04c-4b81-b204-d20008607b95] to complete...done.                               
Created repository [web-app].
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ 
```

## Build and deploy the container images to the Artifact Registry

In this task you will clone the git repository containing the web  application and deploy the application's container images to Artifact  Registry.

### Prepare the application configuration

1. Clone the repository for the lab into your home directory:

```sh
student_01_c867285f72bb@cloudshell:~ (qwiklabs-gcp-04-aec191b9afa6)$ cd ~/
git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
cd cloud-deploy-tutorials
git checkout c3cae80 --quiet
cd tutorials/base
Cloning into 'cloud-deploy-tutorials'...
remote: Enumerating objects: 1453, done.
remote: Counting objects: 100% (1453/1453), done.
remote: Compressing objects: 100% (803/803), done.
remote: Total 1453 (delta 675), reused 1333 (delta 555), pack-reused 0
Receiving objects: 100% (1453/1453), 591.13 KiB | 6.64 MiB/s, done.
Resolving deltas: 100% (675/675), done.
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
cat web/skaffold.yaml
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

apiVersion: skaffold/v2beta7
kind: Config
build:
  artifacts:
    - image: leeroy-web
      context: leeroy-web
    - image: leeroy-app
      context: leeroy-app
  googleCloudBuild:
    projectId: qwiklabs-gcp-04-aec191b9afa6
deploy:
  kubectl:
    manifests:
      - leeroy-web/kubernetes/*
      - leeroy-app/kubernetes/*
portForward:
  - resourceType: deployment
    resourceName: leeroy-web
    port: 8080
    localPort: 9000
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

The web directory now contains the `skaffold.yaml`  configuration file, which provides instructions for Skaffold to build a  container image for your application. This configuration describes the  following items.

The build section configures:

- The two container images that will be built (artifacts)
- The Google Cloud Build project used to build the images

The `deploy` section configures the Kubernetes manifests needed in deploying the workload to a cluster.

The `portForward` configuration is used to define the Kubernetes service for the deployment.

### Build the web application

The skaffold tool will handle submission of the codebase to Cloud Build.

1. Enable the Cloud Build API:
2. Run the skaffold command to build the application and deploy the  container image to the Artifact Registry repository previously created:
3. Once the skaffold build has completed, check for the container images in Artifact Registry:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud services enable cloudbuild.googleapis.com
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cd web
skaffold build --interactive=false \
--default-repo $REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--file-output artifacts.json
cd ..
Generating tags...
 - leeroy-web -> us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web:c3cae80
 - leeroy-app -> us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app:c3cae80
Checking cache...
 - leeroy-web: Not found. Building
 - leeroy-app: Not found. Building
Starting build...
Building 2 artifacts in parallel
Building [leeroy-web]...
Pushing code to gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-72c5a4b9-2266-4ac5-b014-4e34cb3e7614.tar.gz
Logs are available at 
https://storage.cloud.google.com/qwiklabs-gcp-04-aec191b9afa6_cloudbuild/log-c30ab29b-292a-4c73-8575-77738313bc73.txt
starting build "c30ab29b-292a-4c73-8575-77738313bc73"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-72c5a4b9-2266-4ac5-b014-4e34cb3e7614.tar.gz#1715314327079200
Copying gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-72c5a4b9-2266-4ac5-b014-4e34cb3e7614.tar.gz#1715314327079200...
/ [1 files][  986.0 B/  986.0 B]                                                
Operation completed over 1 objects/986.0 B.                                      
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM golang:1.12.9-alpine3.10 as builder
1.12.9-alpine3.10: Pulling from library/golang
9d48c3bd43c5: Pulling fs layer
7f94eaf8af20: Pulling fs layer
9fe9984849c1: Pulling fs layer
cf0db633a67d: Pulling fs layer
0f7136d71739: Pulling fs layer
cf0db633a67d: Waiting
0f7136d71739: Waiting
7f94eaf8af20: Verifying Checksum
7f94eaf8af20: Download complete
9fe9984849c1: Verifying Checksum
9fe9984849c1: Download complete
9d48c3bd43c5: Verifying Checksum
9d48c3bd43c5: Download complete
0f7136d71739: Verifying Checksum
0f7136d71739: Download complete
9d48c3bd43c5: Pull complete
7f94eaf8af20: Pull complete
9fe9984849c1: Pull complete
cf0db633a67d: Verifying Checksum
cf0db633a67d: Download complete
cf0db633a67d: Pull complete
0f7136d71739: Pull complete
Digest: sha256:e0660b4f1e68e0d408420acb874b396fc6dd25e7c1d03ad36e7d6d1155a4dff6
Status: Downloaded newer image for golang:1.12.9-alpine3.10
 ---> e0d646523991
Step 2/7 : COPY web.go .
 ---> 8a3eb9213bd1
Step 3/7 : RUN go build -o /web .
 ---> Running in 3324c099265a
Removing intermediate container 3324c099265a
 ---> 920a845a5ac4
Step 4/7 : FROM alpine:3.10
3.10: Pulling from library/alpine
396c31837116: Pulling fs layer
396c31837116: Download complete
396c31837116: Pull complete
Digest: sha256:451eee8bedcb2f029756dc3e9d73bab0e7943c1ac55cff3a4861c52a0fdd3e98
Status: Downloaded newer image for alpine:3.10
 ---> e7b300aee9f9
Step 5/7 : ENV GOTRACEBACK=single
 ---> Running in 709b9702ff86
Removing intermediate container 709b9702ff86
 ---> 8dd2ae5b3353
Step 6/7 : CMD ["./web"]
 ---> Running in 876bd8d18c3d
Removing intermediate container 876bd8d18c3d
 ---> 28ea7f47e1d3
Step 7/7 : COPY --from=builder /web .
 ---> ce22ba92e346
Successfully built ce22ba92e346
Successfully tagged us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web:c3cae80
PUSH
Pushing us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web:c3cae80
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web]
d0645710ba66: Preparing
9fb3aa2f8b80: Preparing
d0645710ba66: Pushed
9fb3aa2f8b80: Pushed
c3cae80: digest: sha256:c5562084e518d40cce8375be86b919aacab5d61f39e66fb01c4e3cef6a412fcb size: 739
DONE
Build [leeroy-web] succeeded

Building [leeroy-app]...
Pushing code to gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-8b23ef31-6117-4be8-a8e3-ccad7ae6e627.tar.gz
Logs are available at 
https://storage.cloud.google.com/qwiklabs-gcp-04-aec191b9afa6_cloudbuild/log-66a3471b-4f67-4c38-9849-6392aadb67bc.txt
starting build "66a3471b-4f67-4c38-9849-6392aadb67bc"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-8b23ef31-6117-4be8-a8e3-ccad7ae6e627.tar.gz#1715314327177833
Copying gs://qwiklabs-gcp-04-aec191b9afa6_cloudbuild/source/qwiklabs-gcp-04-aec191b9afa6-8b23ef31-6117-4be8-a8e3-ccad7ae6e627.tar.gz#1715314327177833...
/ [1 files][  921.0 B/  921.0 B]                                                
Operation completed over 1 objects/921.0 B.                                      
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM golang:1.12.9-alpine3.10 as builder
1.12.9-alpine3.10: Pulling from library/golang
9d48c3bd43c5: Pulling fs layer
7f94eaf8af20: Pulling fs layer
9fe9984849c1: Pulling fs layer
cf0db633a67d: Pulling fs layer
0f7136d71739: Pulling fs layer
cf0db633a67d: Waiting
0f7136d71739: Waiting
9fe9984849c1: Verifying Checksum
9fe9984849c1: Download complete
7f94eaf8af20: Verifying Checksum
7f94eaf8af20: Download complete
9d48c3bd43c5: Verifying Checksum
9d48c3bd43c5: Download complete
9d48c3bd43c5: Pull complete
0f7136d71739: Verifying Checksum
0f7136d71739: Download complete
7f94eaf8af20: Pull complete
9fe9984849c1: Pull complete
cf0db633a67d: Verifying Checksum
cf0db633a67d: Download complete
cf0db633a67d: Pull complete
0f7136d71739: Pull complete
Digest: sha256:e0660b4f1e68e0d408420acb874b396fc6dd25e7c1d03ad36e7d6d1155a4dff6
Status: Downloaded newer image for golang:1.12.9-alpine3.10
 ---> e0d646523991
Step 2/7 : COPY app.go .
 ---> ddddbe2a75e2
Step 3/7 : RUN go build -o /app .
 ---> Running in 5a3c61aebdbf
Removing intermediate container 5a3c61aebdbf
 ---> eb61fffd72c1
Step 4/7 : FROM alpine:3.10
3.10: Pulling from library/alpine
396c31837116: Pulling fs layer
396c31837116: Download complete
396c31837116: Pull complete
Digest: sha256:451eee8bedcb2f029756dc3e9d73bab0e7943c1ac55cff3a4861c52a0fdd3e98
Status: Downloaded newer image for alpine:3.10
 ---> e7b300aee9f9
Step 5/7 : ENV GOTRACEBACK=single
 ---> Running in d67ad66dc7d0
Removing intermediate container d67ad66dc7d0
 ---> 6eafbe23b231
Step 6/7 : CMD ["./app"]
 ---> Running in cf03794e5dee
Removing intermediate container cf03794e5dee
 ---> bda9f7167bcf
Step 7/7 : COPY --from=builder /app .
 ---> 0a42f2d61cf8
Successfully built 0a42f2d61cf8
Successfully tagged us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app:c3cae80
PUSH
Pushing us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app:c3cae80
The push refers to repository [us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app]
60a497cbd306: Preparing
9fb3aa2f8b80: Preparing
9fb3aa2f8b80: Layer already exists
60a497cbd306: Pushed
c3cae80: digest: sha256:0547b0792391a3a450404eaca02c50a6e7711447b1efd36c467f75e7afb1a157 size: 739
DONE
Build [leeroy-app] succeeded

student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud artifacts docker images list \
$REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--include-tags \
--format yaml
Listing items under project qwiklabs-gcp-04-aec191b9afa6, location us-east1, repository web-app.

---
createTime: '2024-05-10T04:12:31.120927Z'
metadata:
  buildTime: '2024-05-10T04:12:28.532708148Z'
  imageSizeBytes: '6576466'
  mediaType: application/vnd.docker.distribution.manifest.v2+json
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/repositories/web-app/dockerImages/leeroy-app@sha256:0547b0792391a3a450404eaca02c50a6e7711447b1efd36c467f75e7afb1a157
package: us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app
tags: c3cae80
updateTime: '2024-05-10T04:12:31.120927Z'
version: sha256:0547b0792391a3a450404eaca02c50a6e7711447b1efd36c467f75e7afb1a157
---
createTime: '2024-05-10T04:12:30.574687Z'
metadata:
  buildTime: '2024-05-10T04:12:25.775018140Z'
  imageSizeBytes: '6636949'
  mediaType: application/vnd.docker.distribution.manifest.v2+json
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/repositories/web-app/dockerImages/leeroy-web@sha256:c5562084e518d40cce8375be86b919aacab5d61f39e66fb01c4e3cef6a412fcb
package: us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web
tags: c3cae80
updateTime: '2024-05-10T04:12:30.574687Z'
version: sha256:c5562084e518d40cce8375be86b919aacab5d61f39e66fb01c4e3cef6a412fcb
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

By default, Skaffold sets the tag for an image to its related git tag if one is available. Similar information can be found in the `artifacts.json` file that was created by the skaffold command.

Skaffold generates the `web/artifacts.json` file with details of the deployed images:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat web/artifacts.json | jq
{
  "builds": [
    {
      "imageName": "leeroy-web",
      "tag": "us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-web:c3cae80@sha256:c5562084e518d40cce8375be86b919aacab5d61f39e66fb01c4e3cef6a412fcb"
    },
    {
      "imageName": "leeroy-app",
      "tag": "us-east1-docker.pkg.dev/qwiklabs-gcp-04-aec191b9afa6/web-app/leeroy-app:c3cae80@sha256:0547b0792391a3a450404eaca02c50a6e7711447b1efd36c467f75e7afb1a157"
    }
  ]
}
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$
```

## Create the delivery pipeline

In this task you will set up the delivery pipeline.

1. Enable the Google Cloud Deploy API:
2. Create the delivery-pipeline resource using the `delivery-pipeline.yaml` file:
3. Verify the delivery pipeline was created:

```yaml
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat clouddeploy-config/delivery-pipeline.yaml
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

apiVersion: deploy.cloud.google.com/v1beta1
kind: DeliveryPipeline
metadata:
  name: web-app
description: web-app delivery pipeline
serialPipeline:
 stages:
 - targetId: test
 - targetId: staging
 - targetId: prod
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```



```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud services enable clouddeploy.googleapis.com
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud config set deploy/region $REGION
cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
gcloud beta deploy apply --file=clouddeploy-config/delivery-pipeline.yaml
Updated property [deploy/region].
Waiting for the operation on resource projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app...done.                                                   
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app.
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy delivery-pipelines describe web-app
Unable to get target test
Unable to get target staging
Unable to get target prod
Delivery Pipeline:
  condition:
    pipelineReadyCondition: {}
    targetsPresentCondition:
      missingTargets:
      - projects/613884732946/locations/us-east1/targets/test
      - projects/613884732946/locations/us-east1/targets/staging
      - projects/613884732946/locations/us-east1/targets/prod
    targetsTypeCondition:
      status: true
  createTime: '2024-05-10T04:15:41.428778234Z'
  description: web-app delivery pipeline
  etag: d5a82aad55c9ea12
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app
  serialPipeline:
    stages:
    - targetId: test
    - targetId: staging
    - targetId: prod
  uid: 4ac03178b7f94ef9974a328c93656717
  updateTime: '2024-05-10T04:15:42.207019475Z'
Targets: []
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$
```

Notice the first three lines of the output. The delivery pipeline  currently references three target environments that haven't been created yet. In the next task you will create those targets.

## Configure the deployment targets

Three delivery pipeline targets will be created - one for each of the GKE clusters.

### Ensure that the clusters are ready

The three GKE clusters should now be running, but it's useful to verify this.

- Run the following to get the status of the clusters:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,RUNNING
staging,RUNNING
test,RUNNING
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

All three clusters should be in the RUNNING state, as indicated in the  output below. If they are not yet marked as RUNNING, retry the command  above until their status has changed to RUNNING.

### Create a context for each cluster

Use the commands below to get the credentials for each cluster and create an easy-to-use `kubectl` context for referencing the clusters later:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ CONTEXTS=("test" "staging" "prod")
for CONTEXT in ${CONTEXTS[@]}
do
    gcloud container clusters get-credentials ${CONTEXT} --region ${REGION}
    kubectl config rename-context gke_${PROJECT_ID}_${REGION}_${CONTEXT} ${CONTEXT}
done
Fetching cluster endpoint and auth data.
kubeconfig entry generated for test.
Context "gke_qwiklabs-gcp-04-aec191b9afa6_us-east1_test" renamed to "test".
Fetching cluster endpoint and auth data.
kubeconfig entry generated for staging.
Context "gke_qwiklabs-gcp-04-aec191b9afa6_us-east1_staging" renamed to "staging".
Fetching cluster endpoint and auth data.
kubeconfig entry generated for prod.
Context "gke_qwiklabs-gcp-04-aec191b9afa6_us-east1_prod" renamed to "prod".
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

### Create a namespace in each cluster

Use the commands below to create a Kubernetes namespace (web-app) in each of the three clusters:

```yaml
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat kubernetes-config/web-app-namespace.yaml 
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

apiVersion: v1
kind: Namespace
metadata:
  name: web-app
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ for CONTEXT in ${CONTEXTS[@]}
do
    kubectl --context ${CONTEXT} apply -f kubernetes-config/web-app-namespace.yaml
done
namespace/web-app created
namespace/web-app created
namespace/web-app created
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$
```

The application will be deployed to the (web-app) namespace.

### Create the delivery pipeline targets

1. Submit a target definition for each of the targets:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ for CONTEXT in ${CONTEXTS[@]}
do
    envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml
    gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml
done
Waiting for the operation on resource projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/test...done.                                                                
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/test.
Waiting for the operation on resource projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/staging...done.                                                             
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/staging.
Waiting for the operation on resource projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/prod...done.                                                                
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/prod.
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

The targets are described in a yaml file. Each target configures the  relevant cluster information for the target. The test and staging target configurations are mostly the same.

1. Display the details for the test Target:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat clouddeploy-config/target-test.yaml
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

apiVersion: deploy.cloud.google.com/v1beta1
kind: Target
metadata:
  name: test
description: test cluster
gke:
  cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/test
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

```yaml
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat clouddeploy-config/target-staging.yaml
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

apiVersion: deploy.cloud.google.com/v1beta1
kind: Target
metadata:
  name: staging
description: staging cluster
gke:
  cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/staging
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```



The prod target is slightly different as it requires approval (see the `requireApproval` setting in the output) before a release can be promoted to the cluster.

```yaml
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ cat clouddeploy-config/target-prod.yaml
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

apiVersion: deploy.cloud.google.com/v1beta1
kind: Target
metadata:
  name: prod
description: prod cluster
requireApproval: true
gke:
  cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/prod
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

Verify the three targets (test, staging, prod) have been created:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy targets list
targets:
- createTime: '2024-05-10T04:33:03.452188742Z'
  description: staging cluster
  etag: dfb6a7fcc69f53a0
  executionConfigs:
  - artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
    defaultPool:
      artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
      serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/staging
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/staging
  targetId: staging
  uid: b0b3a604df0342338d06d6b1ac8f7c6d
  updateTime: '2024-05-10T04:33:04.023165285Z'
- createTime: '2024-05-10T04:32:57.880231678Z'
  description: test cluster
  etag: 6ca89343761bec8a
  executionConfigs:
  - artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
    defaultPool:
      artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
      serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/test
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/test
  targetId: test
  uid: 6e90d3351ab74cf8ae65f99231c4fc08
  updateTime: '2024-05-10T04:32:58.466060003Z'
- createTime: '2024-05-10T04:33:09.108108536Z'
  description: prod cluster
  etag: b407fb3771b73d5a
  executionConfigs:
  - artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
    defaultPool:
      artifactStorage: gs://us-east1.deploy-artifacts.qwiklabs-gcp-04-aec191b9afa6.appspot.com
      serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 613884732946-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/clusters/prod
  name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/targets/prod
  requireApproval: true
  targetId: prod
  uid: d5a05fd8e6744d3685246c1b2f1266ec
  updateTime: '2024-05-10T04:33:09.189581594Z'
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

All Google Cloud Deploy targets for the delivery pipeline have now been created.

## Create a release

In this task you create a release of the application.

A Google Cloud Deploy release is a specific version of one or more  container images associated with a specific delivery pipeline. Once a  release is created, it can be promoted through multiple targets (the  promotion sequence). Additionally, creating a release renders your  application using skaffold and saves the output as a point-in-time  reference that's used for the duration of that release.

Since this is the first release of your application, you'll name it `web-app-001`.

1. Run the following command to create the release:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ ls
cleanup.sh  clouddeploy-config  kubernetes-config  setup.sh  terraform-config  web
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ ls web/
artifacts.json  leeroy-app  leeroy-web  README.md  skaffold.yaml
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```



```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy releases create web-app-001 \
--delivery-pipeline web-app \
--build-artifacts web/artifacts.json \
--source web/
Creating temporary archive of 9 file(s) totalling 9.2 KiB before compression.
Uploading tarball of [web/] to [gs://4ac03178b7f94ef9974a328c93656717_clouddeploy/source/1715315852.417426-9d51d765bdc1421caab47f3670676eef.tgz]
Waiting for operation [operation-1715315857233-618121859165e-cd8327b1-d4f73b6d]...done.                                                                                            
Created Cloud Deploy release web-app-001.
Creating rollout projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001 in target test
Waiting for rollout creation operation to complete...done.                                                                                                                         
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

The `--build-artifacts` parameter references the `artifacts.json` file created by skaffold earlier. The `--source parameter` references the application source directory where skaffold.yaml can be found.

When a release is created, it will also be automatically rolled out  to the first target in the pipeline (unless approval is required, which  will be covered in a later step of this lab).

To confirm the test target has your application deployed, run the following command:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:37:49.722425Z'
deployEndTime: '2024-05-10T04:38:08.695447Z'
deployStartTime: '2024-05-10T04:37:53.219466650Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/370f99d0-66f8-4112-aba5-625c51339a44
enqueueTime: '2024-05-10T04:37:52.726052Z'
etag: 686ce1df68cd13cc
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/e036061a-a7af-4aed-a8a0-ae3ff0a05cd4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 284504b7604a479591d01aa7bdeb2dd3
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

The first rollout of a release will take several minutes because  Google Cloud Deploy renders the manifests for all targets when the  release is created. The GKE cluster may also take a few minutes to  provide the resources required by the deployment.

If you do not see `state: SUCCESS` in the output from the previous command, please wait and periodically re-run the command until the rollout completes.

Confirm your application was deployed to the test GKE cluster by running the following commands:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ kubectx test
kubectl get all -n web-app
Switched to context "test".
NAME                              READY   STATUS    RESTARTS   AGE
pod/leeroy-app-6f6554984b-t5hsn   1/1     Running   0          3m
pod/leeroy-web-8576f64bdf-mtrzg   1/1     Running   0          3m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
service/leeroy-app   ClusterIP   None         <none>        50051/TCP   3m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/leeroy-app   1/1     1            1           3m1s
deployment.apps/leeroy-web   1/1     1            1           3m1s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/leeroy-app-6f6554984b   1         1         1       3m1s
replicaset.apps/leeroy-web-8576f64bdf   1         1         1       3m1s
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

## Promote the application to staging

In this task you will promote the application from test and into the staging target.

1. Promote the application to the staging target:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001

Promoting release web-app-001 to target staging.

Do you want to continue (Y/n)?  y

Creating rollout projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001 in target staging
Waiting for rollout creation operation to complete...done.                                                                                                                         
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

To confirm the staging Target has your application deployed, run the following command:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:43:51.352255Z'
deployEndTime: '2024-05-10T04:44:08.041774Z'
deployStartTime: '2024-05-10T04:43:52.050738987Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/ccc3a44b-020f-478e-bfb6-0a48d7d99826
enqueueTime: '2024-05-10T04:43:51.352255Z'
etag: 19759b424fe3470c
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001/jobRuns/b94e7578-9a8f-45df-bf99-288017a1941c
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: staging
uid: 7f8b52b733e042d6bb9daafbacdefcee
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:37:49.722425Z'
deployEndTime: '2024-05-10T04:38:08.695447Z'
deployStartTime: '2024-05-10T04:37:53.219466650Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/370f99d0-66f8-4112-aba5-625c51339a44
enqueueTime: '2024-05-10T04:37:52.726052Z'
etag: 686ce1df68cd13cc
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/e036061a-a7af-4aed-a8a0-ae3ff0a05cd4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 284504b7604a479591d01aa7bdeb2dd3
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

### Review the output

Look for the section marked `targetId: staging`. As before, if you do not see `state: SUCCEEDED` in the output from the previous command, wait and periodically re-run the command until the rollout completes.

## Promote the application to prod

In this task you will again promote the application but will also provide approval.

1. Promote the application to the prod target:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001

Promoting release web-app-001 to target prod.

Do you want to continue (Y/n)?  y

Creating rollout projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001 in target prod
Waiting for rollout creation operation to complete...done.                                                                                                                         
The rollout is pending approval.
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

To review the status of the prod target, run the following command:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
---
approvalState: NEEDS_APPROVAL
createTime: '2024-05-10T04:45:56.418572Z'
etag: 6c68c09f58e7b2c2
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      state: PENDING
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: PENDING
state: PENDING_APPROVAL
targetId: prod
uid: edbe657612e14cb6a647909a6f47daae
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:43:51.352255Z'
deployEndTime: '2024-05-10T04:44:08.041774Z'
deployStartTime: '2024-05-10T04:43:52.050738987Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/ccc3a44b-020f-478e-bfb6-0a48d7d99826
enqueueTime: '2024-05-10T04:43:51.352255Z'
etag: 19759b424fe3470c
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001/jobRuns/b94e7578-9a8f-45df-bf99-288017a1941c
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: staging
uid: 7f8b52b733e042d6bb9daafbacdefcee
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:37:49.722425Z'
deployEndTime: '2024-05-10T04:38:08.695447Z'
deployStartTime: '2024-05-10T04:37:53.219466650Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/370f99d0-66f8-4112-aba5-625c51339a44
enqueueTime: '2024-05-10T04:37:52.726052Z'
etag: 686ce1df68cd13cc
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/e036061a-a7af-4aed-a8a0-ae3ff0a05cd4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 284504b7604a479591d01aa7bdeb2dd3
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

In the output, note that the `approvalState` is `NEEDS_APPROVAL` and the state is `PENDING_APPROVAL`.

Approve the rollout with the following:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy rollouts approve web-app-001-to-prod-0001 \
--delivery-pipeline web-app \
--release web-app-001
Approving rollout web-app-001-to-prod-0001 from web-app-001 to target prod.


Do you want to continue (Y/n)?  y

student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

To confirm the prod target has your application deployed, run the following command:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
---
approvalState: APPROVED
approveTime: '2024-05-10T04:48:08.897700Z'
createTime: '2024-05-10T04:45:56.418572Z'
deployEndTime: '2024-05-10T04:48:24.174643Z'
deployStartTime: '2024-05-10T04:48:09.421718955Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/cfac83d0-1ded-4ea4-b875-9e0551ba103d
enqueueTime: '2024-05-10T04:48:08.897700Z'
etag: c9b6a3e7ffff7869
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001/jobRuns/14668f0f-3e09-408d-a70f-0e7d7de5e23b
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: prod
uid: edbe657612e14cb6a647909a6f47daae
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:43:51.352255Z'
deployEndTime: '2024-05-10T04:44:08.041774Z'
deployStartTime: '2024-05-10T04:43:52.050738987Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/ccc3a44b-020f-478e-bfb6-0a48d7d99826
enqueueTime: '2024-05-10T04:43:51.352255Z'
etag: 19759b424fe3470c
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001/jobRuns/b94e7578-9a8f-45df-bf99-288017a1941c
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: staging
uid: 7f8b52b733e042d6bb9daafbacdefcee
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-05-10T04:37:49.722425Z'
deployEndTime: '2024-05-10T04:38:08.695447Z'
deployStartTime: '2024-05-10T04:37:53.219466650Z'
deployingBuild: projects/613884732946/locations/us-east1/builds/370f99d0-66f8-4112-aba5-625c51339a44
enqueueTime: '2024-05-10T04:37:52.726052Z'
etag: 686ce1df68cd13cc
name: projects/qwiklabs-gcp-04-aec191b9afa6/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/613884732946/locations/us-east1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/e036061a-a7af-4aed-a8a0-ae3ff0a05cd4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 284504b7604a479591d01aa7bdeb2dd3
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

As for previous rollouts, locate the entry for the target (`targetId: prod`) and check that the rollout has completed (`state: SUCCEEDED`). Periodically re-run the command until the rollout completes.

Use `kubectl` to check on the status of the deployed application:

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ kubectx prod
kubectl get all -n web-app
Switched to context "prod".
NAME                              READY   STATUS    RESTARTS   AGE
pod/leeroy-app-655f88fb77-dj6n2   1/1     Running   0          87s
pod/leeroy-web-54d44dd8d4-kdh8k   1/1     Running   0          87s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
service/leeroy-app   ClusterIP   None         <none>        50051/TCP   88s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/leeroy-app   1/1     1            1           89s
deployment.apps/leeroy-web   1/1     1            1           89s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/leeroy-app-655f88fb77   1         1         1       89s
replicaset.apps/leeroy-web-54d44dd8d4   1         1         1       89s
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

You learned how to create a delivery pipeline using Google Cloud Deploy. You created a release for a basic application and promoted the  application through a series of Google Kubernetes Engine (GKE) targets.  You first deployed the application to the test target, then promoted it  to the staging target, and finally to the prod target. Now you can use  Cloud Deploy to create continuous delivery pipelines!

## Summary

```sh
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project)
    2  export REGION=us-east1
    3  gcloud config set compute/region $REGION
    4  gcloud services enable container.googleapis.com clouddeploy.googleapis.com
    5  gcloud container clusters create test --node-locations=us-east1-c --num-nodes=1  --async
    6  gcloud container clusters create staging --node-locations=us-east1-c --num-nodes=1  --async
    7  gcloud container clusters create prod --node-locations=us-east1-c --num-nodes=1  --async
    8  gcloud container clusters list --format="csv(name,status)"
    9  gcloud services enable artifactregistry.googleapis.com
   10  gcloud artifacts repositories create web-app --description="Image registry for tutorial web app" --repository-format=docker --location=$REGION
   11  cd ~/
   12  git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
   13  cd cloud-deploy-tutorials
   14  git checkout c3cae80 --quiet
   15  cd tutorials/base
   16  envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
   17  cat web/skaffold.yaml
   18  gcloud services enable cloudbuild.googleapis.com
   19  cd web
   20  skaffold build --interactive=false --default-repo $REGION-docker.pkg.dev/$PROJECT_ID/web-app --file-output artifacts.json
   21  cd ..
   22  gcloud artifacts docker images list $REGION-docker.pkg.dev/$PROJECT_ID/web-app --include-tags --format yaml
   23  cat web/artifacts.json | jq
   24  gcloud services enable clouddeploy.googleapis.com
   25  gcloud config set deploy/region $REGION
   26  cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
   27  gcloud beta deploy apply --file=clouddeploy-config/delivery-pipeline.yaml
   28  gcloud beta deploy delivery-pipelines describe web-app
   29  cat clouddeploy-config/delivery-pipeline.yaml
   30  gcloud container clusters list --format="csv(name,status)"
   31  CONTEXTS=("test" "staging" "prod")
   32  for CONTEXT in ${CONTEXTS[@]}; do     gcloud container clusters get-credentials ${CONTEXT} --region ${REGION};     kubectl config rename-context gke_${PROJECT_ID}_${REGION}_${CONTEXT} ${CONTEXT}; done
   33  for CONTEXT in ${CONTEXTS[@]}; do     kubectl --context ${CONTEXT} apply -f kubernetes-config/web-app-namespace.yaml; done
   34  cat kubernetes-config/web-app-namespace.yaml 
   35  for CONTEXT in ${CONTEXTS[@]}; do     envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml;     gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml; done
   36  cat clouddeploy-config/target-test.yaml
   37  cat clouddeploy-config/target-prod.yaml
   38  cat clouddeploy-config/target-staging.yaml
   39  gcloud beta deploy targets list
   40  gcloud beta deploy releases create web-app-001 --delivery-pipeline web-app --build-artifacts web/artifacts.json --source web/
   41  ls
   42  ls web/
   43  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   44  kubectx test
   45  kubectl get all -n web-app
   46  gcloud beta deploy releases promote --delivery-pipeline web-app --release web-app-001
   47  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   48  gcloud beta deploy releases promote --delivery-pipeline web-app --release web-app-001
   49  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   50  gcloud beta deploy rollouts approve web-app-001-to-prod-0001 --delivery-pipeline web-app --release web-app-001
   51  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   52  kubectx prod
   53  kubectl get all -n web-app
   54  history 
student_01_c867285f72bb@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-04-aec191b9afa6)$ 
```

