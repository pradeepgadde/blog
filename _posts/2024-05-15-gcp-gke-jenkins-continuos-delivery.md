---

layout: single
title:  "Continuous Delivery with Jenkins in Kubernetes Engine"
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

# Continuous Delivery with Jenkins in Kubernetes Engine

Set up a continuous delivery pipeline with `Jenkins` on  Kubernetes engine. Jenkins is the go-to automation server used by  developers who frequently integrate their code in a shared repository. 

- Provision a Jenkins application into a Kubernetes Engine Cluster
- Set up your Jenkins application using Helm Package Manager
- Explore the features of a Jenkins application
- Create and exercise a Jenkins pipeline

### What is Kubernetes Engine?

Kubernetes Engine is Google Cloud's hosted version of `Kubernetes` - a powerful cluster manager and orchestration system for containers.  Kubernetes is an open source project that can run on many different  environments—from laptops to high-availability multi-node clusters; from virtual machines to bare metal. As mentioned before, Kubernetes apps  are built on `containers` - these are lightweight  applications bundled with all the necessary dependencies and libraries  to run them. This underlying structure makes Kubernetes applications  highly available, secure, and quick to deploy—an ideal framework for  cloud developers.

[Jenkins](https://jenkins.io/) is an  open-source automation server that lets you flexibly orchestrate your  build, test, and deployment pipelines. Jenkins allows developers to  iterate quickly on projects without worrying about overhead issues that  can stem from continuous delivery.

### What is Continuous Delivery / Continuous Deployment?

When you need to set up a continuous delivery (CD) pipeline,  deploying Jenkins on Kubernetes Engine provides important benefits over a standard VM-based deployment.

When your build process uses containers, one virtual host can run  jobs on multiple operating systems. Kubernetes Engine provides `ephemeral build executors`—these are only utilized when builds are actively running, which leaves  resources for other cluster tasks such as batch processing jobs. Another benefit of ephemeral build executors is *speed*—they launch in a matter of seconds.

Kubernetes Engine also comes pre-equipped with Google's global load  balancer, which you can use to automate web traffic routing to your  instance(s). The load balancer handles SSL termination and utilizes a  global IP address that's configured with Google's backbone  network—coupled with your web front, this load balancer will always set  your users on the fastest possible path to an application instance.

Now that you've learned a little bit about Kubernetes, Jenkins, and  how the two interact in a CD pipeline, it's time to go build one.

## Download the source code

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-4d196b79a33b.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-4d196b79a33b)$ gcloud config set compute/zone us-west1-b
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-4d196b79a33b)$ gsutil cp gs://spls/gsp051/continuous-deployment-on-kubernetes.zip .
Copying gs://spls/gsp051/continuous-deployment-on-kubernetes.zip...
\ [1 files][  3.3 MiB/  3.3 MiB]                                                
Operation completed over 1 objects/3.3 MiB.                                      
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-4d196b79a33b)$ unzip continuous-deployment-on-kubernetes.zip
Archive:  continuous-deployment-on-kubernetes.zip
   creating: continuous-deployment-on-kubernetes/
   creating: continuous-deployment-on-kubernetes/sample-app/
  inflating: continuous-deployment-on-kubernetes/sample-app/Gopkg.toml  
  inflating: continuous-deployment-on-kubernetes/sample-app/Dockerfile  
   creating: continuous-deployment-on-kubernetes/sample-app/k8s/
   creating: continuous-deployment-on-kubernetes/sample-app/k8s/canary/
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/canary/frontend-canary.yaml  
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/canary/backend-canary.yaml  
   creating: continuous-deployment-on-kubernetes/sample-app/k8s/production/
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/production/backend-production.yaml  
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/production/frontend-production.yaml  
   creating: continuous-deployment-on-kubernetes/sample-app/k8s/dev/
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/dev/backend-dev.yaml  
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/dev/default.yml  
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/dev/frontend-dev.yaml  
   creating: continuous-deployment-on-kubernetes/sample-app/k8s/services/
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/services/frontend.yaml  
  inflating: continuous-deployment-on-kubernetes/sample-app/k8s/services/backend.yaml  
  inflating: continuous-deployment-on-kubernetes/sample-app/html.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/Gopkg.lock  
  inflating: continuous-deployment-on-kubernetes/sample-app/Jenkinsfile  
  inflating: continuous-deployment-on-kubernetes/sample-app/main.go  
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/LICENSE  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/AUTHORS  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/CONTRIBUTORS  
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/compute/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/compute/metadata/
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/cloud.google.com/go/compute/metadata/metadata.go  
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/pre_go17.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/go19.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/pre_go19.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/go17.go  
   creating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/ctxhttp/
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/ctxhttp/ctxhttp.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/ctxhttp/ctxhttp_pre17.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/context/context.go  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/LICENSE  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/PATENTS  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/AUTHORS  
  inflating: continuous-deployment-on-kubernetes/sample-app/vendor/golang.org/x/net/CONTRIBUTORS  
  inflating: continuous-deployment-on-kubernetes/sample-app/main_test.go  
  inflating: continuous-deployment-on-kubernetes/LICENSE  
   creating: continuous-deployment-on-kubernetes/tests/
   creating: continuous-deployment-on-kubernetes/tests/tasks/
  inflating: continuous-deployment-on-kubernetes/tests/tasks/deploy-sample-app.yaml  
  inflating: continuous-deployment-on-kubernetes/tests/tasks/build-sample-app.yaml  
  inflating: continuous-deployment-on-kubernetes/tests/tasks/install-jenkins.yaml  
   creating: continuous-deployment-on-kubernetes/tests/pipelines/
  inflating: continuous-deployment-on-kubernetes/tests/pipelines/cd-on-k8s-prs.yaml  
  inflating: continuous-deployment-on-kubernetes/tests/pipelines/cd-on-k8s-regression.yaml  
   creating: continuous-deployment-on-kubernetes/tests/scripts/
  inflating: continuous-deployment-on-kubernetes/tests/scripts/deploy-sample-app.sh  
  inflating: continuous-deployment-on-kubernetes/tests/scripts/install-jenkins.sh  
  inflating: continuous-deployment-on-kubernetes/tests/scripts/tutorial_setup.sh  
  inflating: continuous-deployment-on-kubernetes/tests/scripts/cleanup.sh  
   creating: continuous-deployment-on-kubernetes/docs/
   creating: continuous-deployment-on-kubernetes/docs/img/
  inflating: continuous-deployment-on-kubernetes/docs/img/new_feature_job.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/master_build_executor.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/sample-app.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/approve.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/jenkins-login.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/download_file.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/cloud-shell-prompt.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/cloud-shell.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/info_card.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/blue_gceme.png  
 extracting: continuous-deployment-on-kubernetes/docs/img/web-preview.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/git-credentials.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/jenkins-credentials.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/first-build.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/jenkins_sa_iam.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/sample_app_master_canary.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/jenkins.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/clone_url.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/jenkins_creds_safromkey.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/enable-gke.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/preview-8080.png  
  inflating: continuous-deployment-on-kubernetes/docs/img/master_two_pipeline.png  
  inflating: continuous-deployment-on-kubernetes/README.md  
 extracting: continuous-deployment-on-kubernetes/.gitignore  
  inflating: continuous-deployment-on-kubernetes/CONTRIBUTING.md  
   creating: continuous-deployment-on-kubernetes/jenkins/
  inflating: continuous-deployment-on-kubernetes/jenkins/values.yaml  
   creating: continuous-deployment-on-kubernetes/.git/
  inflating: continuous-deployment-on-kubernetes/.git/config  
   creating: continuous-deployment-on-kubernetes/.git/objects/
   creating: continuous-deployment-on-kubernetes/.git/objects/03/
 extracting: continuous-deployment-on-kubernetes/.git/objects/03/2422532baf943ed8a512f6f4a4c517c3b81bbf  
   creating: continuous-deployment-on-kubernetes/.git/objects/6a/
 extracting: continuous-deployment-on-kubernetes/.git/objects/6a/203528bfc88b8561d72bfcd70372d2349dbcf6  
   creating: continuous-deployment-on-kubernetes/.git/objects/b5/
 extracting: continuous-deployment-on-kubernetes/.git/objects/b5/06008de1ee0db127e65bb7b7d099356d5d195b  
   creating: continuous-deployment-on-kubernetes/.git/objects/d9/
 extracting: continuous-deployment-on-kubernetes/.git/objects/d9/6090487c7c881747fd76362273ae37a25aa455  
   creating: continuous-deployment-on-kubernetes/.git/objects/e5/
 extracting: continuous-deployment-on-kubernetes/.git/objects/e5/a00958b5e96fd15030a9b0fd0525e9f2862250  
   creating: continuous-deployment-on-kubernetes/.git/objects/f5/
 extracting: continuous-deployment-on-kubernetes/.git/objects/f5/f77a593c36986244ee2fe0a27b843a959dfbaf  
   creating: continuous-deployment-on-kubernetes/.git/objects/pack/
  inflating: continuous-deployment-on-kubernetes/.git/objects/pack/pack-478b0d11cd8afe3060645aeae70ce8882a7ef712.idx  
  inflating: continuous-deployment-on-kubernetes/.git/objects/pack/pack-478b0d11cd8afe3060645aeae70ce8882a7ef712.pack  
   creating: continuous-deployment-on-kubernetes/.git/objects/16/
 extracting: continuous-deployment-on-kubernetes/.git/objects/16/31efbbb375b65fc03e3a86067801a7ebf3d4e8  
   creating: continuous-deployment-on-kubernetes/.git/objects/4c/
 extracting: continuous-deployment-on-kubernetes/.git/objects/4c/b16af6b4a060b7cc3d04bdbccdbbb4184eb57d  
   creating: continuous-deployment-on-kubernetes/.git/objects/43/
 extracting: continuous-deployment-on-kubernetes/.git/objects/43/f009883b6ee0520ba9c3649a6b9d836a179643  
   creating: continuous-deployment-on-kubernetes/.git/objects/9f/
 extracting: continuous-deployment-on-kubernetes/.git/objects/9f/e0325b305bb9e521829306d02d4948dda6d306  
   creating: continuous-deployment-on-kubernetes/.git/objects/00/
 extracting: continuous-deployment-on-kubernetes/.git/objects/00/77ead85a611e04757915d1c1992a929b402764  
   creating: continuous-deployment-on-kubernetes/.git/objects/info/
   creating: continuous-deployment-on-kubernetes/.git/objects/37/
 extracting: continuous-deployment-on-kubernetes/.git/objects/37/fc146d6c2fe591e39b4ccef96c574dd08c6fa2  
   creating: continuous-deployment-on-kubernetes/.git/objects/39/
 extracting: continuous-deployment-on-kubernetes/.git/objects/39/b6a067d38df6129d2e31cae3cf3f13a2737ea1  
   creating: continuous-deployment-on-kubernetes/.git/objects/52/
 extracting: continuous-deployment-on-kubernetes/.git/objects/52/3bb807f80f954bc5d02ad2eb2534336bcb2025  
   creating: continuous-deployment-on-kubernetes/.git/objects/55/
 extracting: continuous-deployment-on-kubernetes/.git/objects/55/90105984de334570897ca26d8236e16b45c688  
   creating: continuous-deployment-on-kubernetes/.git/objects/0f/
 extracting: continuous-deployment-on-kubernetes/.git/objects/0f/abbf295d9e64c959b103e05d1d98fb20d5756c  
   creating: continuous-deployment-on-kubernetes/.git/objects/d4/
 extracting: continuous-deployment-on-kubernetes/.git/objects/d4/03996ba1f3f9781f063665de0d1f515a51faea  
   creating: continuous-deployment-on-kubernetes/.git/objects/a6/
 extracting: continuous-deployment-on-kubernetes/.git/objects/a6/1b8e0ec954c852298bcbcc1861a130b29c442b  
   creating: continuous-deployment-on-kubernetes/.git/objects/e0/
 extracting: continuous-deployment-on-kubernetes/.git/objects/e0/935af9300f5300b5542b2bb25e79712149a33d  
   creating: continuous-deployment-on-kubernetes/.git/objects/2d/
 extracting: continuous-deployment-on-kubernetes/.git/objects/2d/266b6ee3b11dcfce2c54ec6a6949276c3882d3  
   creating: continuous-deployment-on-kubernetes/.git/objects/77/
 extracting: continuous-deployment-on-kubernetes/.git/objects/77/fcbc0a7ed334d2f49841ca7c72f8c07ea2fa5c  
   creating: continuous-deployment-on-kubernetes/.git/objects/1e/
 extracting: continuous-deployment-on-kubernetes/.git/objects/1e/7feaf56eb88533c8a42c58af8bf25225cbb2eb  
   creating: continuous-deployment-on-kubernetes/.git/objects/23/
 extracting: continuous-deployment-on-kubernetes/.git/objects/23/08e9138ca42cff23d1ff74d22e5774e9751d35  
 extracting: continuous-deployment-on-kubernetes/.git/HEAD  
   creating: continuous-deployment-on-kubernetes/.git/info/
  inflating: continuous-deployment-on-kubernetes/.git/info/exclude  
   creating: continuous-deployment-on-kubernetes/.git/logs/
  inflating: continuous-deployment-on-kubernetes/.git/logs/HEAD  
   creating: continuous-deployment-on-kubernetes/.git/logs/refs/
   creating: continuous-deployment-on-kubernetes/.git/logs/refs/heads/
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/heads/master  
   creating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/
   creating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/origin/
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/origin/HEAD  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/origin/master  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/origin/patch-1  
   creating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/v1  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/jenkins-2.67  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/region-tags-yaml  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/link-to-cgc  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/docker-image-readme  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/update-pipeline-plugin  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/enable-agent  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/link-typo  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/increase-agent-resources  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/add-tests  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/revert-177-master  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/v2  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/update-to-jenkins-2  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/test-pr-check  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/dm-templates  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/master  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/test-pr-check-2  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/e2e-test  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/test-branch  
  inflating: continuous-deployment-on-kubernetes/.git/logs/refs/remotes/upstream/helm  
  inflating: continuous-deployment-on-kubernetes/.git/description  
   creating: continuous-deployment-on-kubernetes/.git/hooks/
  inflating: continuous-deployment-on-kubernetes/.git/hooks/commit-msg.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-rebase.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-commit.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/applypatch-msg.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/fsmonitor-watchman.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-receive.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/prepare-commit-msg.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/post-update.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-merge-commit.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-applypatch.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/pre-push.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/update.sample  
  inflating: continuous-deployment-on-kubernetes/.git/hooks/push-to-checkout.sample  
   creating: continuous-deployment-on-kubernetes/.git/refs/
   creating: continuous-deployment-on-kubernetes/.git/refs/heads/
  inflating: continuous-deployment-on-kubernetes/.git/refs/heads/master  
   creating: continuous-deployment-on-kubernetes/.git/refs/tags/
   creating: continuous-deployment-on-kubernetes/.git/refs/remotes/
   creating: continuous-deployment-on-kubernetes/.git/refs/remotes/origin/
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/origin/HEAD  
  inflating: continuous-deployment-on-kubernetes/.git/refs/remotes/origin/master  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/origin/patch-1  
   creating: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/v1  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/jenkins-2.67  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/region-tags-yaml  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/link-to-cgc  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/docker-image-readme  
  inflating: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/update-pipeline-plugin  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/enable-agent  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/link-typo  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/increase-agent-resources  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/add-tests  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/revert-177-master  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/v2  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/update-to-jenkins-2  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/test-pr-check  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/dm-templates  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/master  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/test-pr-check-2  
  inflating: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/e2e-test  
 extracting: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/test-branch  
  inflating: continuous-deployment-on-kubernetes/.git/refs/remotes/upstream/helm  
  inflating: continuous-deployment-on-kubernetes/.git/index  
   creating: continuous-deployment-on-kubernetes/.git/branches/
  inflating: continuous-deployment-on-kubernetes/.git/packed-refs  
 extracting: continuous-deployment-on-kubernetes/.git/COMMIT_EDITMSG  
  inflating: continuous-deployment-on-kubernetes/.git/FETCH_HEAD  
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-02-4d196b79a33b)$ cd continuous-deployment-on-kubernetes
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-02-4d196b79a33b)$ 
```



## Provisioning Jenkins



### **Creating a Kubernetes cluster**

1. Now, run the following command to provision a Kubernetes cluster:
