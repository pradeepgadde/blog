---

layout: single
title:  "Continuous Delivery with Google Cloud Deploy "
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-gke.png
  og_image: /assets/images/gcp-gke.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Continuous Delivery with Google Cloud Deploy

Create three GKE clusters

In this task you will create the three GKE clusters that will be targets for the delivery pipeline.

Three GKE clusters will be created, denoting the three targets for the delivery pipeline:

    test
    staging
    prod

Prepare the web application container image

In this task you'll create a repository in Artifact Registry to hold the web application's container images.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-7def284f141a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ export PROJECT_ID=$(gcloud config get-value project)
export REGION=us-central1
gcloud config set compute/region $REGION
Your active configuration is: [cloudshell-27957]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud services enable \
container.googleapis.com \
clouddeploy.googleapis.com

Operation "operations/acat.p2-222293978205-f420e054-c64d-47b0-b721-5e9eaf9ea780" finished successfully.
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ 
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ source <(kubectl completion bash)
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud container clusters create-auto test --async
gcloud container clusters create-auto staging --async
gcloud container clusters create-auto prod --async
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
NAME: test
TYPE: 
LOCATION: us-central1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
NAME: staging
TYPE: 
LOCATION: us-central1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
NAME: prod
TYPE: 
LOCATION: us-central1
TARGET: 
STATUS_MESSAGE: 
STATUS: PROVISIONING
START_TIME: 
END_TIME: 
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,PROVISIONING
staging,PROVISIONING
test,PROVISIONING
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,PROVISIONING
staging,PROVISIONING
test,PROVISIONING
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud services enable artifactregistry.googleapis.com
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ gcloud artifacts repositories create web-app \
--description="Image registry for sample web app" \
--repository-format=docker \
--location=$REGION
Create request issued for: [web-app]
Waiting for operation [projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/operations/cd52a2c2-7a24-4629-b494-f876e805853e] to complete...done.                            
Created repository [web-app].
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-01-7def284f141a)$ cd ~/
git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
cd cloud-deploy-tutorials
git checkout c3cae80 --quiet
cd tutorials/base
Cloning into 'cloud-deploy-tutorials'...
remote: Enumerating objects: 1617, done.
remote: Counting objects: 100% (166/166), done.
remote: Compressing objects: 100% (103/103), done.
remote: Total 1617 (delta 65), reused 122 (delta 63), pack-reused 1451 (from 1)
Receiving objects: 100% (1617/1617), 593.96 KiB | 10.42 MiB/s, done.
Resolving deltas: 100% (762/762), done.
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
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
    projectId: qwiklabs-gcp-01-7def284f141a
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
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud services enable cloudbuild.googleapis.com
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ cd web
skaffold build --interactive=false \
--default-repo $REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--file-output artifacts.json
cd ..
Generating tags...
 - leeroy-web -> us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web:c3cae80
 - leeroy-app -> us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app:c3cae80
Checking cache...
 - leeroy-web: Not found. Building
 - leeroy-app: Not found. Building
Starting build...
Building 2 artifacts in parallel
Building [leeroy-web]...
Pushing code to gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-74f3162c-24aa-4292-b2c7-57d9e7839f6b.tar.gz
Logs are available at 
https://storage.cloud.google.com/qwiklabs-gcp-01-7def284f141a_cloudbuild/log-5cf43c6b-8cfb-4c09-8a59-8db4c494fcef.txt
starting build "5cf43c6b-8cfb-4c09-8a59-8db4c494fcef"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-74f3162c-24aa-4292-b2c7-57d9e7839f6b.tar.gz#1723656152139572
Copying gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-74f3162c-24aa-4292-b2c7-57d9e7839f6b.tar.gz#1723656152139572...
/ [1 files][  982.0 B/  982.0 B]                                                
Operation completed over 1 objects/982.0 B.                                      
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
9d48c3bd43c5: Verifying Checksum
9fe9984849c1: Verifying Checksum
9fe9984849c1: Download complete
9d48c3bd43c5: Pull complete
7f94eaf8af20: Pull complete
9fe9984849c1: Pull complete
0f7136d71739: Verifying Checksum
0f7136d71739: Download complete
cf0db633a67d: Verifying Checksum
cf0db633a67d: Download complete
cf0db633a67d: Pull complete
0f7136d71739: Pull complete
Digest: sha256:e0660b4f1e68e0d408420acb874b396fc6dd25e7c1d03ad36e7d6d1155a4dff6
Status: Downloaded newer image for golang:1.12.9-alpine3.10
 ---> e0d646523991
Step 2/7 : COPY web.go .
 ---> 16615d7f1123
Step 3/7 : RUN go build -o /web .
 ---> Running in 3fe33dc8f269
Removing intermediate container 3fe33dc8f269
 ---> 47a30d7bf3dd
Step 4/7 : FROM alpine:3.10
3.10: Pulling from library/alpine
396c31837116: Pulling fs layer
396c31837116: Verifying Checksum
396c31837116: Download complete
396c31837116: Pull complete
Digest: sha256:451eee8bedcb2f029756dc3e9d73bab0e7943c1ac55cff3a4861c52a0fdd3e98
Status: Downloaded newer image for alpine:3.10
 ---> e7b300aee9f9
Step 5/7 : ENV GOTRACEBACK=single
 ---> Running in 139c27ed517a
Removing intermediate container 139c27ed517a
 ---> 8b97f1d426d1
Step 6/7 : CMD ["./web"]
 ---> Running in 6f0588af6639
Removing intermediate container 6f0588af6639
 ---> e886d9b6251a
Step 7/7 : COPY --from=builder /web .
 ---> f966e12ae4a6
Successfully built f966e12ae4a6
Successfully tagged us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web:c3cae80
PUSH
Pushing us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web:c3cae80
The push refers to repository [us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web]
ac1b84cace0d: Preparing
9fb3aa2f8b80: Preparing
9fb3aa2f8b80: Layer already exists
ac1b84cace0d: Pushed
c3cae80: digest: sha256:d91224739e3c01f361b471644950bd8eb8fe57dd134556684f7d21172af0e1a3 size: 739
DONE
Build [leeroy-web] succeeded

Building [leeroy-app]...
Pushing code to gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-71cfe8e6-1f0c-4dd2-8d8a-464bf3e59517.tar.gz
Logs are available at 
https://storage.cloud.google.com/qwiklabs-gcp-01-7def284f141a_cloudbuild/log-bd471bab-5ea1-45b8-b9ab-21618dfe6870.txt
starting build "bd471bab-5ea1-45b8-b9ab-21618dfe6870"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-71cfe8e6-1f0c-4dd2-8d8a-464bf3e59517.tar.gz#1723656152141389
Copying gs://qwiklabs-gcp-01-7def284f141a_cloudbuild/source/qwiklabs-gcp-01-7def284f141a-71cfe8e6-1f0c-4dd2-8d8a-464bf3e59517.tar.gz#1723656152141389...
/ [1 files][  918.0 B/  918.0 B]                                                
Operation completed over 1 objects/918.0 B.                                      
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
0f7136d71739: Verifying Checksum
0f7136d71739: Download complete
9d48c3bd43c5: Verifying Checksum
9d48c3bd43c5: Download complete
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
Step 2/7 : COPY app.go .
 ---> b8c5822ae64b
Step 3/7 : RUN go build -o /app .
 ---> Running in f9ae1d44378e
Removing intermediate container f9ae1d44378e
 ---> 0572365e039d
Step 4/7 : FROM alpine:3.10
3.10: Pulling from library/alpine
396c31837116: Pulling fs layer
396c31837116: Verifying Checksum
396c31837116: Download complete
396c31837116: Pull complete
Digest: sha256:451eee8bedcb2f029756dc3e9d73bab0e7943c1ac55cff3a4861c52a0fdd3e98
Status: Downloaded newer image for alpine:3.10
 ---> e7b300aee9f9
Step 5/7 : ENV GOTRACEBACK=single
 ---> Running in 37ffb2b8e007
Removing intermediate container 37ffb2b8e007
 ---> bc2c35c44810
Step 6/7 : CMD ["./app"]
 ---> Running in 4676ec26f574
Removing intermediate container 4676ec26f574
 ---> cc2101fea04b
Step 7/7 : COPY --from=builder /app .
 ---> 4d53a1cb8a72
Successfully built 4d53a1cb8a72
Successfully tagged us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app:c3cae80
PUSH
Pushing us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app:c3cae80
The push refers to repository [us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app]
a3e394ede89c: Preparing
9fb3aa2f8b80: Preparing
a3e394ede89c: Pushed
9fb3aa2f8b80: Pushed
c3cae80: digest: sha256:15c8fe520911357abbd483f8ea7da1b7d728be543475c26e2b9a5087174753d4 size: 739
DONE
Build [leeroy-app] succeeded

student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud artifacts docker images list \
$REGION-docker.pkg.dev/$PROJECT_ID/web-app \
--include-tags \
--format yaml
Listing items under project qwiklabs-gcp-01-7def284f141a, location us-central1, repository web-app.

---
createTime: '2024-08-14T17:22:57.170415Z'
metadata:
  buildTime: '2024-08-14T17:22:52.570870278Z'
  imageSizeBytes: '6576467'
  mediaType: application/vnd.docker.distribution.manifest.v2+json
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/repositories/web-app/dockerImages/leeroy-app@sha256:15c8fe520911357abbd483f8ea7da1b7d728be543475c26e2b9a5087174753d4
package: us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app
tags:
- c3cae80
updateTime: '2024-08-14T17:22:57.170415Z'
version: sha256:15c8fe520911357abbd483f8ea7da1b7d728be543475c26e2b9a5087174753d4
---
createTime: '2024-08-14T17:22:58.033823Z'
metadata:
  buildTime: '2024-08-14T17:22:54.100651554Z'
  imageSizeBytes: '6636952'
  mediaType: application/vnd.docker.distribution.manifest.v2+json
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/repositories/web-app/dockerImages/leeroy-web@sha256:d91224739e3c01f361b471644950bd8eb8fe57dd134556684f7d21172af0e1a3
package: us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web
tags:
- c3cae80
updateTime: '2024-08-14T17:22:58.033823Z'
version: sha256:d91224739e3c01f361b471644950bd8eb8fe57dd134556684f7d21172af0e1a3
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```



```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ cat web/artifacts.json | jq
{
  "builds": [
    {
      "imageName": "leeroy-web",
      "tag": "us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-web:c3cae80@sha256:d91224739e3c01f361b471644950bd8eb8fe57dd134556684f7d21172af0e1a3"
    },
    {
      "imageName": "leeroy-app",
      "tag": "us-central1-docker.pkg.dev/qwiklabs-gcp-01-7def284f141a/web-app/leeroy-app:c3cae80@sha256:15c8fe520911357abbd483f8ea7da1b7d728be543475c26e2b9a5087174753d4"
    }
  ]
}
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```



```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud services enable clouddeploy.googleapis.com
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud config set deploy/region $REGION
cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
gcloud beta deploy apply --file=clouddeploy-config/delivery-pipeline.yaml
Updated property [deploy/region].
Waiting for the operation on resource projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app...done.                                                
Created Cloud Deploy resource: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app.
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy delivery-pipelines describe web-app
Unable to get target test
Unable to get target staging
Unable to get target prod
Delivery Pipeline:
  condition:
    pipelineReadyCondition: {}
    targetsPresentCondition:
      missingTargets:
      - projects/222293978205/locations/us-central1/targets/prod
      - projects/222293978205/locations/us-central1/targets/test
      - projects/222293978205/locations/us-central1/targets/staging
    targetsTypeCondition:
      status: true
  createTime: '2024-08-14T17:28:45.210141015Z'
  description: web-app delivery pipeline
  etag: 214e521dcd71072d
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app
  serialPipeline:
    stages:
    - targetId: test
    - targetId: staging
    - targetId: prod
  uid: 37f6407f33ae494aab3c6c2d22275857
  updateTime: '2024-08-14T17:28:45.544999560Z'
Targets: []
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,RUNNING
staging,RUNNING
test,RUNNING
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud container clusters list --format="csv(name,status)"
name,status
prod,RUNNING
staging,RUNNING
test,RUNNING
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ CONTEXTS=("test" "staging" "prod")
for CONTEXT in ${CONTEXTS[@]}
do
    gcloud container clusters get-credentials ${CONTEXT} --region ${REGION}
    kubectl config rename-context gke_${PROJECT_ID}_${REGION}_${CONTEXT} ${CONTEXT}
done
Fetching cluster endpoint and auth data.
kubeconfig entry generated for test.
Context "gke_qwiklabs-gcp-01-7def284f141a_us-central1_test" renamed to "test".
Fetching cluster endpoint and auth data.
kubeconfig entry generated for staging.
Context "gke_qwiklabs-gcp-01-7def284f141a_us-central1_staging" renamed to "staging".
Fetching cluster endpoint and auth data.
kubeconfig entry generated for prod.
Context "gke_qwiklabs-gcp-01-7def284f141a_us-central1_prod" renamed to "prod".
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ for CONTEXT in ${CONTEXTS[@]}
do
    kubectl --context ${CONTEXT} apply -f kubernetes-config/web-app-namespace.yaml
done
namespace/web-app created
namespace/web-app created
namespace/web-app created
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ for CONTEXT in ${CONTEXTS[@]}
do
    envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml
    gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml
done
Waiting for the operation on resource projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/test...done.                                                             
Created Cloud Deploy resource: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/test.
Waiting for the operation on resource projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/staging...done.                                                          
Created Cloud Deploy resource: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/staging.
Waiting for the operation on resource projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/prod...done.                                                             
Created Cloud Deploy resource: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/prod.
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```



```yaml
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ cat clouddeploy-config/target-test.yaml
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
  cluster: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/clusters/test
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```yaml
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ cat clouddeploy-config/target-prod.yaml
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
  cluster: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/clusters/prod
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```



```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy targets list
targets:
- createTime: '2024-08-14T17:33:43.850736610Z'
  description: staging cluster
  etag: 25fa56f0f4f38c91
  executionConfigs:
  - artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
    defaultPool:
      artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
      serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/clusters/staging
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/staging
  targetId: staging
  uid: 5633666dabfe4cb2a122b44739c1bf16
  updateTime: '2024-08-14T17:33:44.210419463Z'
- createTime: '2024-08-14T17:33:40.211291790Z'
  description: test cluster
  etag: 541407344e4819e4
  executionConfigs:
  - artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
    defaultPool:
      artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
      serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/clusters/test
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/test
  targetId: test
  uid: 6a46416787394d8399f74cb5268739b3
  updateTime: '2024-08-14T17:33:40.934730930Z'
- createTime: '2024-08-14T17:33:47.445180239Z'
  description: prod cluster
  etag: fe62c23405cb400c
  executionConfigs:
  - artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
    defaultPool:
      artifactStorage: gs://us-central1.deploy-artifacts.qwiklabs-gcp-01-7def284f141a.appspot.com
      serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    executionTimeout: 3600s
    serviceAccount: 222293978205-compute@developer.gserviceaccount.com
    usages:
    - RENDER
    - DEPLOY
    - VERIFY
    - PREDEPLOY
    - POSTDEPLOY
  gke:
    cluster: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/clusters/prod
  name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/targets/prod
  requireApproval: true
  targetId: prod
  uid: c08fd8e26947486eb61a1a4599432ee8
  updateTime: '2024-08-14T17:33:47.553153473Z'
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

## Create a release

In this task you create a release of the application.

A Google Cloud Deploy release is a specific version of one or more  container images associated with a specific delivery pipeline. Once a  release is created, it can be promoted through multiple targets (the  promotion sequence). Additionally, creating a release renders your  application using skaffold and saves the output as a point-in-time  reference that's used for the duration of that release.

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy releases create web-app-001 \
--delivery-pipeline web-app \
--build-artifacts web/artifacts.json \
--source web/
Creating temporary archive of 9 file(s) totalling 9.2 KiB before compression.
Uploading tarball of [web/] to [gs://37f6407f33ae494aab3c6c2d22275857_clouddeploy/source/1723657112.908885-320ecaa966d147ceaeb87f754e93954f.tgz]
Waiting for operation [operation-1723657114250-61fa832026bc0-c8063d73-7dfff245]...done.                                                                                            
Created Cloud Deploy release web-app-001.
Creating rollout projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001 in target test
Waiting for rollout creation operation to complete...done.                                                                                                                         
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:38:39.321721Z'
deployStartTime: '2024-08-14T17:38:50.270650118Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/5be0a695-94aa-4f04-ad53-377803181474
enqueueTime: '2024-08-14T17:38:49.483961Z'
etag: 5203c488b1f05621
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/87fff6cb-9c46-476a-8b4f-1c3eb70244c4
      state: IN_PROGRESS
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: IN_PROGRESS
state: IN_PROGRESS
targetId: test
uid: 4cc9cd10167b40a3892ff10efe24392b
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:38:39.321721Z'
deployEndTime: '2024-08-14T17:42:21.176409Z'
deployStartTime: '2024-08-14T17:38:50.270650118Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/5be0a695-94aa-4f04-ad53-377803181474
enqueueTime: '2024-08-14T17:38:49.483961Z'
etag: 1c9d3eb226fda535
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/87fff6cb-9c46-476a-8b4f-1c3eb70244c4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 4cc9cd10167b40a3892ff10efe24392b
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ kubectx test
kubectl get all -n web-app
Switched to context "test".
NAME                              READY   STATUS    RESTARTS   AGE
pod/leeroy-app-585bd46679-sshh5   1/1     Running   0          6m19s
pod/leeroy-web-6499bcbc75-rvbtz   1/1     Running   0          6m19s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
service/leeroy-app   ClusterIP   None         <none>        50051/TCP   6m19s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/leeroy-app   1/1     1            1           6m19s
deployment.apps/leeroy-web   1/1     1            1           6m19s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/leeroy-app-585bd46679   1         1         1       6m19s
replicaset.apps/leeroy-web-6499bcbc75   1         1         1       6m19s
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

Promote the application to staging

In this task you will promote the application from test and into the staging target.

    Promote the application to the staging target:



```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001

Promoting release web-app-001 to target staging.

Do you want to continue (Y/n)?  y

Creating rollout projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001 in target staging
Waiting for rollout creation operation to complete...done.                                                                                                                         
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:46:16.651860Z'
deployEndTime: '2024-08-14T17:49:15.671470Z'
deployStartTime: '2024-08-14T17:46:17.246735184Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/abe66ad7-fb71-40ca-8d73-b8d996bd7828
enqueueTime: '2024-08-14T17:46:16.651860Z'
etag: 4b0d84bd0b151109
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001/jobRuns/7bea6428-98e7-4fe9-8696-90c937f1757b
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: staging
uid: cfb1b89646994a338bc60fae16cc6cfe
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:38:39.321721Z'
deployEndTime: '2024-08-14T17:42:21.176409Z'
deployStartTime: '2024-08-14T17:38:50.270650118Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/5be0a695-94aa-4f04-ad53-377803181474
enqueueTime: '2024-08-14T17:38:49.483961Z'
etag: 1c9d3eb226fda535
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/87fff6cb-9c46-476a-8b4f-1c3eb70244c4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 4cc9cd10167b40a3892ff10efe24392b
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

## Promote the application to prod

In this task you will again promote the application but will also provide approval.

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy releases promote \
--delivery-pipeline web-app \
--release web-app-001

Promoting release web-app-001 to target prod.

Do you want to continue (Y/n)?  y

Creating rollout projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001 in target prod
Waiting for rollout creation operation to complete...done.                                                                                                                         
The rollout is pending approval.
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy rollouts list \
--delivery-pipeline web-app \
--release web-app-001
---
approvalState: NEEDS_APPROVAL
createTime: '2024-08-14T17:51:46.740070Z'
etag: 6c68c09f58e7b2c2
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-prod-0001
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
uid: a79e24ae81c34829bf407607ab16400c
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:46:16.651860Z'
deployEndTime: '2024-08-14T17:49:15.671470Z'
deployStartTime: '2024-08-14T17:46:17.246735184Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/abe66ad7-fb71-40ca-8d73-b8d996bd7828
enqueueTime: '2024-08-14T17:46:16.651860Z'
etag: 4b0d84bd0b151109
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-staging-0001/jobRuns/7bea6428-98e7-4fe9-8696-90c937f1757b
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: staging
uid: cfb1b89646994a338bc60fae16cc6cfe
---
approvalState: DOES_NOT_NEED_APPROVAL
createTime: '2024-08-14T17:38:39.321721Z'
deployEndTime: '2024-08-14T17:42:21.176409Z'
deployStartTime: '2024-08-14T17:38:50.270650118Z'
deployingBuild: projects/222293978205/locations/us-central1/builds/5be0a695-94aa-4f04-ad53-377803181474
enqueueTime: '2024-08-14T17:38:49.483961Z'
etag: 1c9d3eb226fda535
name: projects/qwiklabs-gcp-01-7def284f141a/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001
phases:
- deploymentJobs:
    deployJob:
      deployJob: {}
      id: deploy
      jobRun: projects/222293978205/locations/us-central1/deliveryPipelines/web-app/releases/web-app-001/rollouts/web-app-001-to-test-0001/jobRuns/87fff6cb-9c46-476a-8b4f-1c3eb70244c4
      state: SUCCEEDED
    verifyJob:
      id: verify
      state: DISABLED
      verifyJob: {}
  id: stable
  state: SUCCEEDED
state: SUCCEEDED
targetId: test
uid: 4cc9cd10167b40a3892ff10efe24392b
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ gcloud beta deploy rollouts approve web-app-001-to-prod-0001 \
--delivery-pipeline web-app \
--release web-app-001
Approving rollout web-app-001-to-prod-0001 from web-app-001 to target prod.


Do you want to continue (Y/n)?  y

student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```





```sh
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project)
    2  export REGION=us-central1
    3  gcloud config set compute/region $REGION
    4  gcloud services enable container.googleapis.com clouddeploy.googleapis.com
    5  source <(kubectl completion bash)
    6  gcloud container clusters create-auto test --async
    7  gcloud container clusters create-auto staging --async
    8  gcloud container clusters create-auto prod --async
    9  gcloud container clusters list --format="csv(name,status)"
   10  gcloud services enable artifactregistry.googleapis.com
   11  gcloud artifacts repositories create web-app --description="Image registry for sample web app" --repository-format=docker --location=$REGION
   12  cd ~/
   13  git clone https://github.com/GoogleCloudPlatform/cloud-deploy-tutorials.git
   14  cd cloud-deploy-tutorials
   15  git checkout c3cae80 --quiet
   16  cd tutorials/base
   17  envsubst < clouddeploy-config/skaffold.yaml.template > web/skaffold.yaml
   18  cat web/skaffold.yaml
   19  gcloud services enable cloudbuild.googleapis.com
   20  cd web
   21  skaffold build --interactive=false --default-repo $REGION-docker.pkg.dev/$PROJECT_ID/web-app --file-output artifacts.json
   22  cd ..
   23  gcloud artifacts docker images list $REGION-docker.pkg.dev/$PROJECT_ID/web-app --include-tags --format yaml
   24  cat web/artifacts.json | jq
   25  gcloud services enable clouddeploy.googleapis.com
   26  gcloud config set deploy/region $REGION
   27  cp clouddeploy-config/delivery-pipeline.yaml.template clouddeploy-config/delivery-pipeline.yaml
   28  gcloud beta deploy apply --file=clouddeploy-config/delivery-pipeline.yaml
   29  gcloud beta deploy delivery-pipelines describe web-app
   30  gcloud container clusters list --format="csv(name,status)"
   31  CONTEXTS=("test" "staging" "prod")
   32  for CONTEXT in ${CONTEXTS[@]}; do     gcloud container clusters get-credentials ${CONTEXT} --region ${REGION};     kubectl config rename-context gke_${PROJECT_ID}_${REGION}_${CONTEXT} ${CONTEXT}; done
   33  for CONTEXT in ${CONTEXTS[@]}; do     kubectl --context ${CONTEXT} apply -f kubernetes-config/web-app-namespace.yaml; done
   34  for CONTEXT in ${CONTEXTS[@]}; do     envsubst < clouddeploy-config/target-$CONTEXT.yaml.template > clouddeploy-config/target-$CONTEXT.yaml;     gcloud beta deploy apply --file clouddeploy-config/target-$CONTEXT.yaml; done
   35  cat clouddeploy-config/target-test.yaml
   36  cat clouddeploy-config/target-prod.yaml
   37  gcloud beta deploy targets list
   38  gcloud container clusters list
   39  gcloud beta deploy releases create web-app-001 --delivery-pipeline web-app --build-artifacts web/artifacts.json --source web/
   40  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   41  kubectx test
   42  kubectl get all -n web-app
   43  gcloud beta deploy releases promote --delivery-pipeline web-app --release web-app-001
   44  gcloud beta deploy releases promote --delivery-pipeline web-app --release web-app-001
   45  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   46  gcloud beta deploy releases promote --delivery-pipeline web-app --release web-app-001
   47  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   48  gcloud beta deploy rollouts approve web-app-001-to-prod-0001 --delivery-pipeline web-app --release web-app-001
   49  gcloud beta deploy rollouts list --delivery-pipeline web-app --release web-app-001
   50  kubectx prod
   51  kubectl get all -n web-app
   52  kubectl get nodes
   53  kubectl get all -n web-app
   
   59  history 
student_01_ceb2542405cf@cloudshell:~/cloud-deploy-tutorials/tutorials/base (qwiklabs-gcp-01-7def284f141a)$ 
```

