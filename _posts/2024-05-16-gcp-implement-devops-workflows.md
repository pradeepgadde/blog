---

layout: single
title:  "Implement DevOps Workflows in Google Cloud"
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

# Implement DevOps Workflows in Google Cloud

Creating a GKE cluster based on a set of configurations provided.
Creating a Google Source Repository to host your Go application code.
Creating Cloud Build Triggers that deploy a production and development application.
Pushing updates to the app and creating new builds.
Rolling back the production application to a previous version.

Overall, you are creating a simple CI/CD pipeline using Cloud Source Repositories, Artifact Registry, and Cloud Build.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-3b729f45e4ec.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-3b729f45e4ec)$ history 
    1  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com
    2  gcloud config set project qwiklabs-gcp-03-3b729f45e4ec 
    3  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com
    4  export PROJECT_ID=$(gcloud config get-value project)
    5  gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$(gcloud projects describe $PROJECT_ID \
    6  --format="value(projectNumber)")@cloudbuild.gserviceaccount.com --role="roles/container.developer"
    7  git config --global user.email student-01-aa8f3a74b714@qwiklabs.net
    8  git config --global --global user.name Pradeep
    9  gcloud artifacts repositories create my-repository --repository-format=docker   --location=europe-west1   --description="Docker repository for Container Dev Workshop"
   10  gcloud container clusters create hello-cluster --zone=europe-west1-d -h
   11  gcloud container clusters create hello-cluster --zone=europe-west1-d --enable-autoscaling --max-nodes 6 --min-nodes 2 --num-nodes 3 --release-channel regular --cluster-version 1.29.1-gke.1589020
   12  gcloud container clusters get-credentials hello-cluster
   13  gcloud container clusters get-credentials hello-cluster --location europe-west1-d
   14  kubectl get nodes
   15  kubectl get ns
   16  kubectl create ns prod
   17  kubectl create ns dev
   18  kubectl get ns
   19  history 
   20  gcloud source repos create sample-app
   21  ls
   22  cd ~
   23  gsutil cp -r gs://spls/gsp330/sample-app/* sample-app
   24  gcloud source repos clone sample-app
   25  ls
   26  cd ~
   27  gsutil cp -r gs://spls/gsp330/sample-app/* sample-app
   28  ls 
   29  ls sample-app/
   30  export REGION="europe-west1"
   31  export ZONE="europe-west1-d"
   32  for file in sample-app/cloudbuild-dev.yaml sample-app/cloudbuild.yaml; do     sed -i "s/<your-region>/${REGION}/g" "$file";     sed -i "s/<your-zone>/${ZONE}/g" "$file"; done
   33  git add .
   34  cd sample-app/
   35  git status
   36  git add .
   37  git commit -m "first commit"
   38  git push origin master
   39  git branch
   40  git checkout dev
   41  git branch
   42  git checkout -b dev
   43  git branch
   44  git status
   45  ls
   46  git push origin dev
   47  git push 
   48  git push --set-upstream origin dev
   49  git push
   50  ls
   51  cat cloudbuild-dev.yaml 
   52  git branch
   53  ls
   54  cat cloudbuild-dev.yaml 
   55  cat dev/deployment.yaml 
   56  git status
   57  git add .
   58  git commit -m "first job"
   59  git push
   60  kubectl get pods -n dev
   61  kubectl get pods -n prod
   62  kubectl get deploy -n dev
   63  kubectl expose deployment development-deployment --name dev-deployment-service     --type LoadBalancer --port 8080 --target-port 8080
   64  kubectl expose deployment -n dev development-deployment --name dev-deployment-service     --type LoadBalancer --port 8080 --target-port 8080 
   65  kubectl get svc -n dev
   66  kubectl get svc -n dev
   67  kubectl get svc -n dev
   68  kubectl get svc -n dev
   69  kubectl get svc -n dev
   70  kubectl get svc -n dev
   71  kubectl get svc -n dev
   72  kubectl get svc -n dev
   73  kubectl get svc -n dev
   74  kubectl get svc -n dev
   75  kubectl get svc -n dev
   76  kubectl get svc -n dev
   77  git branch
   78  git checkout master
   79  git branch
 * You may obtain a copy of the License at
   80  ;s
steps:
   81  ls
apiVersion: apps/v1
   82  vi cloudbuild.yaml 
   83  vi cloudbuild.yaml 
   84  cat cloudbuild.yaml 
   85  vi prod/deployment.yaml 
   86  cat prod/deployment.yaml 
   87  git status
   88  git add .
   89  git commit -m "master build"
   90  git push
   91  kubectl get pods -n prod
   92  kubectl get deploy -m prod
   93  kubectl get deploy -n prod
   94  ls
   95  cat Dockerfile 
   96  kubectl expose deployment -n prod production-deployment --name prod-deployment-service     --type LoadBalancer --port 8080 --target-port 8080
   97  kubectl get svc -n prod
   98  kubectl get svc -n prod
   99  kubectl get svc -n prod
  100  kubectl get svc -n prod
  101  kubectl get svc -n prod
  102  kubectl get svc -n prod
  103  kubectl get svc -n prod
  104  kubectl get svc -n prod
  105  kubectl get svc -n prod
  106  git branch
  107  git checkout dev
  108  history 
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-3b729f45e4ec)$ git branch
fatal: not a git repository (or any parent up to mount point /)
Stopping at filesystem boundary (GIT_DISCOVERY_ACROSS_FILESYSTEM not set).
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-3b729f45e4ec)$ cd sample-app/
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git branch
* dev
  master
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ ls
cloudbuild-dev.yaml  cloudbuild.yaml  dev  Dockerfile  main.go  prod
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi main.go
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi cloudbuild-dev.yaml 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi dev/deployment.yaml 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git status
On branch dev
Your branch is up to date with 'origin/dev'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   cloudbuild-dev.yaml
        modified:   dev/deployment.yaml
        modified:   main.go

no changes added to commit (use "git add" and/or "git commit -a")
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git add .
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git commit -m "dev commit 2"
[dev dfad173] dev commit 2
 3 files changed, 12 insertions(+), 5 deletions(-)
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git push
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 2 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 644 bytes | 322.00 KiB/s, done.
Total 6 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3)
remote: Waiting for private key checker: 3/3 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-03-3b729f45e4ec/r/sample-app
   7f6d615..dfad173  dev -> dev
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n dev
NAME                                      READY   STATUS              RESTARTS   AGE
development-deployment-6f6c4766d9-kl4j2   1/1     Running             0          20m
development-deployment-6f6c4766d9-n2b8q   1/1     Running             0          20m
development-deployment-6f6c4766d9-qvfll   1/1     Running             0          20m
development-deployment-79db946d8b-zmvfr   0/1     ContainerCreating   0          2s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n dev
NAME                                      READY   STATUS    RESTARTS   AGE
development-deployment-79db946d8b-4rhcc   1/1     Running   0          4s
development-deployment-79db946d8b-l8dfh   1/1     Running   0          7s
development-deployment-79db946d8b-zmvfr   1/1     Running   0          10s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get deploy -n dev
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
development-deployment   3/3     3            3           21m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get svc -n dev
NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
dev-deployment-service   LoadBalancer   10.66.154.131   35.187.62.52   8080:31800/TCP   17m
/**
/**
 * Copyright 2023 Google Inc.
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

        "image/draw"
NAME                                      READY   STATUS              RESTARTS   AGE
steps:
development-deployment-6f6c4766d9-kl4j2   1/1     Running             0          20m
apiVersion: apps/v1
development-deployment-6f6c4766d9-n2b8q   1/1     Running             0          20m
development-deployment-6f6c4766d9-qvfll   1/1     Running             0          20m
development-deployment-79db946d8b-zmvfr   0/1     ContainerCreating   0          2s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n dev
NAME                                      READY   STATUS    RESTARTS   AGE
development-deployment-79db946d8b-4rhcc   1/1     Running   0          4s
development-deployment-79db946d8b-l8dfh   1/1     Running   0          7s
development-deployment-79db946d8b-zmvfr   1/1     Running   0          10s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get deploy -n dev
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
development-deployment   3/3     3            3           21m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get svc -n dev
NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
dev-deployment-service   LoadBalancer   10.66.154.131   35.187.62.52   8080:31800/TCP   17m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git branch
  dev
* master
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi main.go 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi cloudbuild.yaml 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ vi prod/deployment.yaml 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   cloudbuild.yaml
        modified:   main.go
        modified:   prod/deployment.yaml

no changes added to commit (use "git add" and/or "git commit -a")
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git add .
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git commit -m "second commit"
[master 4e8acdc] second commit
 3 files changed, 11 insertions(+), 4 deletions(-)
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ git push
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 2 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 544 bytes | 272.00 KiB/s, done.
Total 6 (delta 4), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (4/4)
remote: Waiting for private key checker: 3/3 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-03-3b729f45e4ec/r/sample-app
   2f88d7a..4e8acdc  master -> master
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n prod
NAME                                     READY   STATUS    RESTARTS   AGE
production-deployment-6dd46c8c5f-4x9b4   1/1     Running   0          13m
production-deployment-6dd46c8c5f-5nd2z   1/1     Running   0          13m
production-deployment-6dd46c8c5f-nhqpl   1/1     Running   0          13m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n prod
NAME                                     READY   STATUS    RESTARTS   AGE
production-deployment-6dd46c8c5f-4x9b4   1/1     Running   0          14m
production-deployment-6dd46c8c5f-5nd2z   1/1     Running   0          14m
production-deployment-6dd46c8c5f-nhqpl   1/1     Running   0          14m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n prod
NAME                                     READY   STATUS              RESTARTS   AGE
production-deployment-6dd46c8c5f-4x9b4   1/1     Running             0          14m
production-deployment-6dd46c8c5f-5nd2z   1/1     Running             0          14m
production-deployment-79f4f5fbcb-4t4hv   0/1     ContainerCreating   0          2s
production-deployment-79f4f5fbcb-c2jf8   1/1     Running             0          4s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get pods -n prod
NAME                                     READY   STATUS    RESTARTS   AGE
production-deployment-79f4f5fbcb-4t4hv   1/1     Running   0          6s
production-deployment-79f4f5fbcb-c2jf8   1/1     Running   0          8s
production-deployment-79f4f5fbcb-jppsc   1/1     Running   0          3s
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get deploy -n prod
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
production-deployment   3/3     3            3           14m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ kubectl get svc -n prod
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
prod-deployment-service   LoadBalancer   10.66.153.111   35.205.248.197   8080:32129/TCP   13m
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ history 
    1  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com
    2  gcloud config set project qwiklabs-gcp-03-3b729f45e4ec 
    3  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com
    4  export PROJECT_ID=$(gcloud config get-value project)
    5  gcloud projects add-iam-policy-binding $PROJECT_ID --member=serviceAccount:$(gcloud projects describe $PROJECT_ID \
    6  --format="value(projectNumber)")@cloudbuild.gserviceaccount.com --role="roles/container.developer"
    7  git config --global user.email student-01-aa8f3a74b714@qwiklabs.net
    8  git config --global --global user.name Pradeep
    9  gcloud artifacts repositories create my-repository --repository-format=docker   --location=europe-west1   --description="Docker repository for Container Dev Workshop"
   10  gcloud container clusters create hello-cluster --zone=europe-west1-d -h
   11  gcloud container clusters create hello-cluster --zone=europe-west1-d --enable-autoscaling --max-nodes 6 --min-nodes 2 --num-nodes 3 --release-channel regular --cluster-version 1.29.1-gke.1589020
   12  gcloud container clusters get-credentials hello-cluster
   13  gcloud container clusters get-credentials hello-cluster --location europe-west1-d
   14  kubectl get nodes
   15  kubectl get ns
   16  kubectl create ns prod
   17  kubectl create ns dev
   18  kubectl get ns
   19  history 
   20  gcloud source repos create sample-app
   21  ls
   22  cd ~
   23  gsutil cp -r gs://spls/gsp330/sample-app/* sample-app
   24  gcloud source repos clone sample-app
   25  ls
   26  cd ~
   27  gsutil cp -r gs://spls/gsp330/sample-app/* sample-app
   28  ls 
   29  ls sample-app/
   30  export REGION="europe-west1"
   31  export ZONE="europe-west1-d"
   32  for file in sample-app/cloudbuild-dev.yaml sample-app/cloudbuild.yaml; do     sed -i "s/<your-region>/${REGION}/g" "$file";     sed -i "s/<your-zone>/${ZONE}/g" "$file"; done
   33  git add .
   34  cd sample-app/
   35  git status
   36  git add .
   37  git commit -m "first commit"
   38  git push origin master
   39  git branch
   40  git checkout dev
   41  git branch
   42  git checkout -b dev
   43  git branch
   44  git status
   45  ls
   46  git push origin dev
   47  git push 
   48  git push --set-upstream origin dev
   49  git push
   50  ls
   51  cat cloudbuild-dev.yaml 
   52  git branch
   53  ls
   54  cat cloudbuild-dev.yaml 
   55  cat dev/deployment.yaml 
   56  git status
   57  git add .
   58  git commit -m "first job"
   59  git push
   60  kubectl get pods -n dev
   61  kubectl get pods -n prod
   62  kubectl get deploy -n dev
   63  kubectl expose deployment development-deployment --name dev-deployment-service     --type LoadBalancer --port 8080 --target-port 8080
   64  kubectl expose deployment -n dev development-deployment --name dev-deployment-service     --type LoadBalancer --port 8080 --target-port 8080 
   65  kubectl get svc -n dev
   66  kubectl get svc -n dev
   67  kubectl get svc -n dev
   68  kubectl get svc -n dev
   69  kubectl get svc -n dev
   70  kubectl get svc -n dev
   71  kubectl get svc -n dev
   72  kubectl get svc -n dev
   73  kubectl get svc -n dev
   74  kubectl get svc -n dev
   75  kubectl get svc -n dev
   76  kubectl get svc -n dev
   77  git branch
   78  git checkout master
   79  git branch
   80  ;s
   81  ls
   82  vi cloudbuild.yaml 
   83  vi cloudbuild.yaml 
   84  cat cloudbuild.yaml 
   85  vi prod/deployment.yaml 
   86  cat prod/deployment.yaml 
   87  git status
   88  git add .
   89  git commit -m "master build"
   90  git push
   91  kubectl get pods -n prod
   92  kubectl get deploy -m prod
   93  kubectl get deploy -n prod
   94  ls
   95  cat Dockerfile 
   96  kubectl expose deployment -n prod production-deployment --name prod-deployment-service     --type LoadBalancer --port 8080 --target-port 8080
   97  kubectl get svc -n prod
   98  kubectl get svc -n prod
   99  kubectl get svc -n prod
  100  kubectl get svc -n prod
  101  kubectl get svc -n prod
  102  kubectl get svc -n prod
  103  kubectl get svc -n prod
  104  kubectl get svc -n prod
  105  kubectl get svc -n prod
  106  git branch
  107  git checkout dev
  108  history 
  109  git branch
  110  cd sample-app/
  111  git branch
  112  ls
  113  vi main.go
  114  vi cloudbuild-dev.yaml 
  115  vi dev/deployment.yaml 
  116  git status
  117  git add .
  118  git commit -m "dev commit 2"
  119  git push
  120  kubectl get pods -n dev
  121  kubectl get pods -n dev
  122  kubectl get deploy -n dev
  123  kubectl get svc -n dev
  124  git checkout master
  125  git branch
  126  vi main.go 
  127  vi cloudbuild.yaml 
  128  vi prod/deployment.yaml 
  129  git status
  130  git add .
  131  git commit -m "second commit"
  132  git push
  133  kubectl get pods -n prod
  134  kubectl get pods -n prod
  135  kubectl get pods -n prod
  136  kubectl get pods -n prod
  137  kubectl get deploy -n prod
  138  kubectl get svc -n prod
  139  history 
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat cloudbuild.yaml 
steps:
  # Step 1: Compile the Go Application
  - name: 'gcr.io/cloud-builders/go'
    id: 'Compile application'
    env: ['GOPATH=/gopath']
    args: ['build', '-o', 'main', 'main.go']

  # Step 2: Build the Docker image for the Go application
  - name: 'gcr.io/cloud-builders/docker'
    id: 'Build Docker image'
    args: ['build', '-t', 'europe-west1-docker.pkg.dev/qwiklabs-gcp-03-3b729f45e4ec/my-repository/hello-cloudbuild:v2.0', '.']

  # Step 3: Push the Docker image to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    id: 'Push Docker image'
    args: ['push', 'europe-west1-docker.pkg.dev/qwiklabs-gcp-03-3b729f45e4ec/my-repository/hello-cloudbuild:v2.0']

  # Step 4: Apply the production deployment YAML file to the production namespace
  - name: 'gcr.io/cloud-builders/kubectl'
    id: 'Deploy'
    args: ['-n', 'prod', 'apply', '-f', 'prod/deployment.yaml']
    env:
    - 'CLOUDSDK_COMPUTE_REGION=europe-west1-d'
    - 'CLOUDSDK_CONTAINER_CLUSTER=hello-cluster'
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat cloudbuild-dev.yaml 
steps:
  # Step 1: Compile the Go Application
  - name: 'gcr.io/cloud-builders/go'
    env: ['GOPATH=/gopath']
    args: ['build', '-o', 'main', 'main.go']

  # Step 2: Build the Docker image for the Go application
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'europe-west1-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild-dev:<version>', '.']

  # Step 3: Push the Docker image to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'europe-west1-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild-dev:<version>']

  # Step 4: Apply the production deployment YAML file to the production namespace
  - name: 'gcr.io/cloud-builders/kubectl'
    id: 'Deploy'
    args: ['-n', 'dev', 'apply', '-f', 'dev/deployment.yaml']
    env:
    - 'CLOUDSDK_COMPUTE_REGION=europe-west1-d'
    - 'CLOUDSDK_CONTAINER_CLUSTER=hello-cluster'
  student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat Dockerfile 
# Copyright 2023 Google LLC
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

FROM golang:1.19.2 as builder
WORKDIR /app
RUN go mod init hello-app
COPY *.go ./
RUN CGO_ENABLED=0 GOOS=linux go build -o /hello-app

FROM gcr.io/distroless/base-debian11
WORKDIR /
COPY --from=builder /hello-app /hello-app
ENV PORT 8080
USER nonroot:nonroot
CMD ["/hello-app"]student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat main.go 
/**
 * Copyright 2023 Google Inc.
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

package main

import (
        "image"
        "image/color"
        "image/draw"
        "image/png"
        "net/http"
)

func main() {
        http.HandleFunc("/blue", blueHandler)
        http.HandleFunc("/red", redHandler)
        http.ListenAndServe(":8080", nil)
}

func blueHandler(w http.ResponseWriter, r *http.Request) {
        img := image.NewRGBA(image.Rect(0, 0, 100, 100))
        draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{0, 0, 255, 255}}, image.ZP, draw.Src)
        w.Header().Set("Content-Type", "image/png")
        png.Encode(w, img)
}
func redHandler(w http.ResponseWriter, r *http.Request) {
        img := image.NewRGBA(image.Rect(0, 0, 100, 100))
        draw.Draw(img, img.Bounds(), &image.Uniform{color.RGBA{255, 0, 0, 255}}, image.ZP, draw.Src)
        w.Header().Set("Content-Type", "image/png")
        png.Encode(w, img)
}
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ ls
cloudbuild-dev.yaml  cloudbuild.yaml  dev  Dockerfile  main.go  prod
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat dev/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: development-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dev-app
  template:
    metadata:
      labels:
        app: dev-app
    spec:
      containers:
      - name: dev-container
        image: <todo>
        ports:
        - containerPort: 8080
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ cat prod/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: production-app
  template:
    metadata:
      labels:
        app: production-app
    spec:
      containers:
      - name: production-container
        image: europe-west1-docker.pkg.dev/qwiklabs-gcp-03-3b729f45e4ec/my-repository/hello-cloudbuild:v2.0
        ports:
        - containerPort: 8080
student_01_aa8f3a74b714@cloudshell:~/sample-app (qwiklabs-gcp-03-3b729f45e4ec)$ 
```

