---

layout: single
title:  "Scale Out and Update a Containerized Application on a Kubernetes Cluster"
categories: Cloud
tags: GCP
classes: wide
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
# Scale Out and Update a Containerized Application on a Kubernetes Cluster

## Challenge scenario

You are taking over ownership of a test environment and have been given  an updated version of a containerized test application to deploy. Your  systems' architecture team has started adopting a containerized  microservice architecture. You are responsible for managing the  containerized test web applications. You will first deploy the initial  version of a test application, called `echo-app` to a Kubernetes cluster called `echo-cluster` in a deployment called `echo-web`. The cluster will be deployed in the  zone.

## Your challenge

You need to update the running `echo-app` application in the `echo-web` deployment from the v1 to the v2 code you have been provided. You must  also scale out the application to 2 instances and confirm that they are  all running.



## Build and deploy the updated application with a new tag

The updated sample application, including the Dockerfile and the application context files, are contained in an archive called `echo-web-v2.tar.gz`. The archive has been copied to a Cloud Storage bucket in your lab project called . V2 of the application adds a version number to the output of the  application. In this task, you will download the archive, build the  Docker image, and tag it with the `v2` tag.



## Push the image to the Container Registry

Your organization uses the Container Registry to host Docker images for deployments, and uses the `gcr.io` Container Registry hostname for all projects. You must push the updated image to the Container Registry before deploying it.



## Deploy the updated application to the Kubernetes cluster

In this task, you will deploy the updated application to the Kubernetes cluster. The deployment should be named `echo-web` and the application should be exposed on port 80. The application should be accessible from outside the cluster.



## Scale out the application

In this task, you will need to scale out the application to 2 replicas.



## Confirm the application is running

In this task, you will need to confirm that the application is  running and responding correctly. You can use the external IP address of the application to test it.


```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-a7cad0fd7fcb.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ gcloud container clusters get-credentials echo-cluster --zone=us-central1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for echo-cluster.
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1
deployment.apps/echo-web created
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
service/echo-web exposed
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-5bc676cd5b-jbxfv   1/1     Running   0          19s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get deploy,svc
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-web   1/1     1            1           25s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/echo-web     LoadBalancer   10.107.246.222   <pending>     80:31289/TCP   16s
service/kubernetes   ClusterIP      10.107.240.1     <none>        443/TCP        49m
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get deploy,svc
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-web   1/1     1            1           42s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/echo-web     LoadBalancer   10.107.246.222   <pending>     80:31289/TCP   34s
service/kubernetes   ClusterIP      10.107.240.1     <none>        443/TCP        49m
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get deploy,svc
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/echo-web   1/1     1            1           89s

NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/echo-web     LoadBalancer   10.107.246.222   34.72.7.13    80:31289/TCP   80s
service/kubernetes   ClusterIP      10.107.240.1     <none>        443/TCP        50m
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ gsutil cp gs://qwiklabs-gcp-00-a7cad0fd7fcb/echo-web-v2.tar.gz .
Copying gs://qwiklabs-gcp-00-a7cad0fd7fcb/echo-web-v2.tar.gz...
/ [1 files][  2.0 KiB/  2.0 KiB]                                                
Operation completed over 1 objects/2.0 KiB.                                      
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ ls
echo-web-v2.tar.gz  README-cloudshell.txt
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ tar -xvf echo-web-v2.tar.gz 
./
./manifests/
./manifests/echoweb-ingress-static-ip.yaml
./manifests/echoweb-deployment.yaml
./manifests/echoweb-service-static-ip.yaml
./README.md
./Dockerfile
./main.go
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ ls
Dockerfile  echo-web-v2.tar.gz  main.go  manifests  README-cloudshell.txt  README.md
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker build -t gcr.io/qwiklabs-resources/echo-app:v2
ERROR: "docker buildx build" requires exactly 1 argument.
See 'docker buildx build --help'.

Usage:  docker buildx build [OPTIONS] PATH | URL | -

Start a build
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker build -t gcr.io/qwiklabs-resources/echo-app:v2 .
[+] Building 15.7s (11/11) FINISHED                                                                                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 195B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                                               2.8s
 => [internal] load metadata for docker.io/library/golang:1.8-alpine                                                                                                           4.8s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load build context                                                                                                                                              0.1s
 => => transferring context: 113.67kB                                                                                                                                          0.0s
 => [stage-1 1/2] FROM docker.io/library/alpine:latest@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b                                                 0.9s
 => => resolve docker.io/library/alpine:latest@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b                                                         0.0s
 => => sha256:6457d53fb065d6f250e1504b9bc42d5b6c65941d57532c072d929dd0628977d0 528B / 528B                                                                                     0.0s
 => => sha256:05455a08881ea9cf0e752bc48e61bbd71a34c029bb13df01e40e3e70e0d007bd 1.47kB / 1.47kB                                                                                 0.0s
 => => sha256:4abcf20661432fb2d719aaf90656f55c287f8ca915dc1c92ec14ff61e67fbaf8 3.41MB / 3.41MB                                                                                 0.6s
 => => sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b 1.64kB / 1.64kB                                                                                 0.0s
 => => extracting sha256:4abcf20661432fb2d719aaf90656f55c287f8ca915dc1c92ec14ff61e67fbaf8                                                                                      0.1s
 => [stage-0 1/3] FROM docker.io/library/golang:1.8-alpine@sha256:693568f2ab0dae1e19f44b41628d2aea148fac65974cfd18f83cb9863ab1a177                                             8.1s
 => => resolve docker.io/library/golang:1.8-alpine@sha256:693568f2ab0dae1e19f44b41628d2aea148fac65974cfd18f83cb9863ab1a177                                                     0.0s
 => => sha256:4cb86d3661bfe0bca825a3a509fa29a8a291a72a8661cdf3a65898bb79b406ff 4.20kB / 4.20kB                                                                                 0.0s
 => => sha256:693568f2ab0dae1e19f44b41628d2aea148fac65974cfd18f83cb9863ab1a177 434B / 434B                                                                                     0.0s
 => => sha256:8edab665c397766a23aa484ef057ac6897dc0e7538afb2f165eda0190f1b3084 1.57kB / 1.57kB                                                                                 0.0s
 => => sha256:550fe1bea624a5c62551cf09f3aa10886eed133794844af1dfb775118309387e 1.97MB / 1.97MB                                                                                 0.9s
 => => sha256:cbc8da23026a6f8a7d1254ad40c91449a96ca7ee3263195b70dec76020fab46a 0B / 350.67kB                                                                                  10.8s
 => => sha256:9b35aaa06d7a456c2bccf1c8fd8d51384d549806dfe2b50e4f8da11a64e940c4 487B / 487B                                                                                     1.3s
 => => sha256:46ca6ce0ffd195964343d5d6afa214c30ee0bca12cacf7dbe64ef06fd9f0a4d9 75.65MB / 75.65MB                                                                               2.3s
 => => extracting sha256:550fe1bea624a5c62551cf09f3aa10886eed133794844af1dfb775118309387e                                                                                      0.1s
 => => sha256:7a270aebe80a722432503e5c547c8159489d2c0a55387e976ec53cfc52138c24 126B / 126B                                                                                     1.6s
 => => extracting sha256:cbc8da23026a6f8a7d1254ad40c91449a96ca7ee3263195b70dec76020fab46a                                                                                      0.1s
 => => extracting sha256:9b35aaa06d7a456c2bccf1c8fd8d51384d549806dfe2b50e4f8da11a64e940c4                                                                                      0.0s
 => => sha256:8695117c367ea4ccf3098c70d1cdbdc9a483cccef9e56320534e65b717695535 1.36kB / 1.36kB                                                                                 1.8s
 => => extracting sha256:46ca6ce0ffd195964343d5d6afa214c30ee0bca12cacf7dbe64ef06fd9f0a4d9                                                                                      5.3s
 => => extracting sha256:7a270aebe80a722432503e5c547c8159489d2c0a55387e976ec53cfc52138c24                                                                                      0.0s
 => => extracting sha256:8695117c367ea4ccf3098c70d1cdbdc9a483cccef9e56320534e65b717695535                                                                                      0.0s
 => [stage-0 2/3] ADD . /go/src/echo-app                                                                                                                                       1.6s
 => [stage-0 3/3] RUN go install echo-app                                                                                                                                      0.9s
 => [stage-1 2/2] COPY --from=0 /go/bin/echo-app .                                                                                                                             0.0s
 => exporting to image                                                                                                                                                         0.1s
 => => exporting layers                                                                                                                                                        0.1s
 => => writing image sha256:0d5b69ecbd7b98a7ea84ec6b6363b600029908dfb813994581bfc0be140efcdc                                                                                   0.0s
 => => naming to gcr.io/qwiklabs-resources/echo-app:v2                                                                                                                         0.0s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker push  gcr.io/qwiklabs-resources/echo-app:v2 
The push refers to repository [gcr.io/qwiklabs-resources/echo-app]
94c3aaf4477b: Preparing 
d4fc045c9e3a: Preparing 
unauthorized: You don't have the needed permissions to perform this operation, and you may have invalid credentials. To authenticate your request, follow the steps in: https://cloud.google.com/container-registry/docs/advanced-authentication
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ gcloud auth configure-docker
WARNING: Your config file at [/home/student_00_ce66bacea858/.docker/config.json] contains these credential helper entries:

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
Adding credentials for all GCR repositories.
WARNING: A long list of credential helpers may cause delays running 'docker build'. We recommend passing the registry name to configure only the registry you are using.
After update, the following will be written to your Docker config file located at [/home/student_00_ce66bacea858/.docker/config.json]:
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
    "us-west4-docker.pkg.dev": "gcloud",
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  y

Docker configuration file updated.
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker push  gcr.io/qwiklabs-resources/echo-app:v2 
The push refers to repository [gcr.io/qwiklabs-resources/echo-app]
94c3aaf4477b: Preparing 
d4fc045c9e3a: Preparing 
denied: Token exchange failed for project 'qwiklabs-resources'. Caller does not have permission or the resource may not exist 'storage.buckets.get'. To configure permissions, follow instructions at: https://cloud.google.com/container-registry/docs/access-control
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker push  gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2 
The push refers to repository [gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app]
An image does not exist locally with the tag: gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker images
REPOSITORY                           TAG       IMAGE ID       CREATED              SIZE
gcr.io/qwiklabs-resources/echo-app   v2        0d5b69ecbd7b   About a minute ago   13.3MB
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker rm 0d5b69ecbd7b
Error response from daemon: No such container: 0d5b69ecbd7b
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker rmi 0d5b69ecbd7b
Untagged: gcr.io/qwiklabs-resources/echo-app:v2
Deleted: sha256:0d5b69ecbd7b98a7ea84ec6b6363b600029908dfb813994581bfc0be140efcdc
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker build -t gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2 .
[+] Building 2.5s (11/11) FINISHED                                                                                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 195B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                                               1.1s
 => [internal] load metadata for docker.io/library/golang:1.8-alpine                                                                                                           1.0s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 14.50kB                                                                                                                                           0.0s
 => [stage-1 1/2] FROM docker.io/library/alpine:latest@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b                                                 0.0s
 => CACHED [stage-0 1/3] FROM docker.io/library/golang:1.8-alpine@sha256:693568f2ab0dae1e19f44b41628d2aea148fac65974cfd18f83cb9863ab1a177                                      0.0s
 => [stage-0 2/3] ADD . /go/src/echo-app                                                                                                                                       0.1s
 => [stage-0 3/3] RUN go install echo-app                                                                                                                                      1.0s
 => CACHED [stage-1 2/2] COPY --from=0 /go/bin/echo-app .                                                                                                                      0.0s
 => exporting to image                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:0d5b69ecbd7b98a7ea84ec6b6363b600029908dfb813994581bfc0be140efcdc                                                                                   0.0s
 => => naming to gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2                                                                                                               0.0s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker iamges
docker: 'iamges' is not a docker command.
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-05-21T11:25:25Z"
  generation: 1
  labels:
    app: echo-web
  name: echo-web
  namespace: default
  resourceVersion: "27931"
  uid: 34bcd45b-f004-4301-8b49-1c7629433adb
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: echo-web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
See 'docker --help'
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker images                                                                                                  
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-05-21T11:25:25Z"
  generation: 1
  labels:
    app: echo-web
  name: echo-web
  namespace: default
  resourceVersion: "27931"
  uid: 34bcd45b-f004-4301-8b49-1c7629433adb
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: echo-web
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
REPOSITORY                                     TAG       IMAGE ID       CREATED         SIZE
gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app   v2        0d5b69ecbd7b   2 minutes ago   13.3MB
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ docker push  gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2                                                   
The push refers to repository [gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app]
94c3aaf4477b: Pushed 
d4fc045c9e3a: Pushed 
v2: digest: sha256:8d748aad55743a872ee7a771ad70b2cddec38e1ffdd640ea1bd8f8de98a32857 size: 739
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
echo-web   1/1     1            1           8m4s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl edit deploy echo-web
Edit cancelled, no changes made.
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl edit deploy echo-web
deployment.apps/echo-web edited
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-66956f7cd6-27ns9   1/1     Running   0          5s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
echo-web   1/1     1            1           9m12s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-66956f7cd6-27ns9   1/1     Running   0          17s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.107.246.222   34.72.7.13    80:31289/TCP   9m14s
kubernetes   ClusterIP      10.107.240.1     <none>        443/TCP        58m
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl scale deployment echo-web --replicas=2
deployment.apps/echo-web scaled
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-66956f7cd6-27ns9   1/1     Running   0          86s
echo-web-66956f7cd6-v9vkp   1/1     Running   0          4s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-66956f7cd6-27ns9   1/1     Running   0          3m11s
echo-web-66956f7cd6-v9vkp   1/1     Running   0          109s
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ ls
Dockerfile  echo-web-v2.tar.gz  main.go  manifests  README-cloudshell.txt  README.md
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ cat Dockerfile 
FROM golang:1.8-alpine
ADD . /go/src/echo-app
RUN go install echo-app

FROM alpine:latest
COPY --from=0 /go/bin/echo-app .
ENV PORT 8000
CMD ["./echo-app"]
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ cat main.go 
/**
 * Copyright 2017 Google Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *   http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

// [START all]
package main

import (
        "fmt"
        "log"
        "net/http"
        "net"
        "strings"
        "os"
)

func main() {
        // use PORT environment variable, or default to 8000
        port := "8000"
        if fromEnv := os.Getenv("PORT"); fromEnv != "" {
                port = fromEnv
        }

        // register hello function to handle all requests
        server := http.NewServeMux()
        server.HandleFunc("/", echo)

        // start the web server on port and accept requests
        log.Printf("Server listening on port %s", port)
        err := http.ListenAndServe(":"+port, server)
        log.Fatal(err)
}

// echo responds to the request with a plain-text "Serving request" message 
// followed by some meta-data baout the environment where it is running
func echo(w http.ResponseWriter, r *http.Request) {
        log.Printf("Serving request: %s", r.URL.Path)
        host, _ := os.Hostname()
        addrs, err := net.LookupHost(host)
        ipaddresses := ""

        if err == nil {
                ipaddresses = strings.Join(addrs, " ")
        }

        fmt.Fprintf(w, "Echo Test\n")
        fmt.Fprintf(w, "Version: 2.0.0\n")
        fmt.Fprintf(w, "Hostname: %s\n", host)
        fmt.Fprintf(w, "Host ip-address(es): %s\n", ipaddresses)
}
// [END all]
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ ls
Dockerfile  echo-web-v2.tar.gz  main.go  manifests  README-cloudshell.txt  README.md
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ history 
    1  gcloud container clusters get-credentials echo-cluster --zone=us-central1-c
    2  kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1
    3  kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
    4  kubectl get pods
    5  kubectl get deploy,svc
    6  gsutil cp gs://qwiklabs-gcp-00-a7cad0fd7fcb/echo-web-v2.tar.gz .
    7  ls
    8  tar -xvf echo-web-v2.tar.gz 
    9  ls
   10  docker build -t gcr.io/qwiklabs-resources/echo-app:v2
   11  docker build -t gcr.io/qwiklabs-resources/echo-app:v2 .
   12  docker push  gcr.io/qwiklabs-resources/echo-app:v2 
   13  gcloud auth configure-docker
   14  docker push  gcr.io/qwiklabs-resources/echo-app:v2 
   15  docker push  gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2 
   16  docker images
   17  docker rm 0d5b69ecbd7b
   18  docker rmi 0d5b69ecbd7b
   19  docker build -t gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2 .
   20  docker iamges
   21  docker images
   22  docker push  gcr.io/qwiklabs-gcp-00-a7cad0fd7fcb/echo-app:v2 
   23  kubectl get deploy
   24  kubectl edit deploy echo-web
   25  kubectl get pods
   26  kubectl get deploy
   27  kubectl get pods
   28  kubectl get svc
   29  kubectl scale deployment echo-web --replicas=2
   30  kubectl get pods
   31  ls
   32  cat Dockerfile 
   33  cat main.go 
   34  ls
   35  history 
student_00_ce66bacea858@cloudshell:~ (qwiklabs-gcp-00-a7cad0fd7fcb)$ 
```