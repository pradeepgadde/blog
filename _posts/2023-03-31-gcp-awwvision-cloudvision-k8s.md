---

layout: single
title:  "Awwvision: Cloud Vision API from a Kubernetes Cluster "
date:   2023-03-31 01:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Awwvision: Cloud Vision API from a Kubernetes Cluster



The Awwvision lab uses  [Kubernetes](https://kubernetes.io) and  [Cloud Vision API](https://cloud.google.com/vision) to demonstrate how to use the Vision API to classify (label) images from Reddit's  [/r/aww](https://reddit.com/r/aww) subreddit and display the labelled results in a web app.

Awwvision has three components:

1. A simple  [Redis](http://redis.io) instance.
2. A web app that displays the labels and associated images.
3. A worker that handles scraping Reddit for images and classifying them using the Vision API.  [Cloud Pub/Sub](https://cloud.google.com/pubsub) is used to coordinate tasks between multiple worker instances.

## 1. Create a Kubernetes Engine cluster

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-158d547a0831.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ gcloud config set compute/zone us-central1-a
Updated property [compute/zone].
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ gcloud container clusters create awwvision \
    --num-nodes 2 \
    --scopes cloud-platform
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Default change: During creation of nodepools or autoscaling configuration changes for cluster versions greater than 1.24.1-gke.800 a default location policy is applied. For Spot and PVM it defaults to ANY, and for all other VM kinds a BALANCED policy is used. To change the default values use the `--location-policy` flag.
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster awwvision in us-central1-a... Cluster is being health-checked (master is healthy)...done.
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-00-158d547a0831/zones/us-central1-a/clusters/awwvision].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/awwvision?project=qwiklabs-gcp-00-158d547a0831
kubeconfig entry generated for awwvision.
NAME: awwvision
LOCATION: us-central1-a
MASTER_VERSION: 1.24.9-gke.3200
MASTER_IP: 34.135.173.60
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.24.9-gke.3200
NUM_NODES: 2
STATUS: RUNNING
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```



```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ gcloud container clusters get-credentials awwvision
Fetching cluster endpoint and auth data.
kubeconfig entry generated for awwvision.
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```



```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ kubectl cluster-info
Kubernetes control plane is running at https://34.135.173.60
GLBCDefaultBackend is running at https://34.135.173.60/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
KubeDNS is running at https://34.135.173.60/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://34.135.173.60/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```





## 2. Create a virtual environment

```sh
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ sudo apt-get install -y virtualenv
********************************************************************************
You are running apt-get inside of Cloud Shell. Note that your Cloud Shell
machine is ephemeral and no system-wide change will persist beyond session end.

To suppress this warning, create an empty ~/.cloudshell/no-apt-get-warning file.
The command will automatically proceed in 5 seconds or on any key.

Visit https://cloud.google.com/shell/help for more information.
********************************************************************************
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  python3-appdirs python3-distlib python3-filelock python3-importlib-metadata python3-more-itertools python3-six python3-virtualenv
  python3-zipp
The following NEW packages will be installed:
  python3-appdirs python3-distlib python3-filelock python3-importlib-metadata python3-more-itertools python3-six python3-virtualenv
  python3-zipp virtualenv
0 upgraded, 9 newly installed, 0 to remove and 7 not upgraded.
Need to get 331 kB of archives.
After this operation, 1,464 kB of additional disk space will be used.
Get:1 https://deb.debian.org/debian bullseye/main amd64 python3-appdirs all 1.4.4-1 [12.7 kB]
Get:2 https://deb.debian.org/debian bullseye/main amd64 python3-distlib all 0.3.2+really+0.3.1-0.1 [123 kB]
Get:3 https://deb.debian.org/debian bullseye/main amd64 python3-filelock all 3.0.12-2 [8,036 B]
Get:4 https://deb.debian.org/debian bullseye/main amd64 python3-six all 1.16.0-2 [17.5 kB]
Get:5 https://deb.debian.org/debian bullseye/main amd64 python3-more-itertools all 4.2.0-3 [42.7 kB]
Get:6 https://deb.debian.org/debian bullseye/main amd64 python3-zipp all 1.0.0-3 [6,060 B]
Get:7 https://deb.debian.org/debian bullseye/main amd64 python3-importlib-metadata all 1.6.0-2 [10.3 kB]
Get:8 https://deb.debian.org/debian bullseye/main amd64 python3-virtualenv all 20.4.0+ds-2+deb11u1 [89.1 kB]
Get:9 https://deb.debian.org/debian bullseye/main amd64 virtualenv all 20.4.0+ds-2+deb11u1 [21.4 kB]
Fetched 331 kB in 0s (968 kB/s)  
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package python3-appdirs.
(Reading database ... 142198 files and directories currently installed.)
Preparing to unpack .../0-python3-appdirs_1.4.4-1_all.deb ...
Unpacking python3-appdirs (1.4.4-1) ...
Selecting previously unselected package python3-distlib.
Preparing to unpack .../1-python3-distlib_0.3.2+really+0.3.1-0.1_all.deb ...
Unpacking python3-distlib (0.3.2+really+0.3.1-0.1) ...
Selecting previously unselected package python3-filelock.
Preparing to unpack .../2-python3-filelock_3.0.12-2_all.deb ...
Unpacking python3-filelock (3.0.12-2) ...
Selecting previously unselected package python3-six.
Preparing to unpack .../3-python3-six_1.16.0-2_all.deb ...
Unpacking python3-six (1.16.0-2) ...
Selecting previously unselected package python3-more-itertools.
Preparing to unpack .../4-python3-more-itertools_4.2.0-3_all.deb ...
Unpacking python3-more-itertools (4.2.0-3) ...
Selecting previously unselected package python3-zipp.
Preparing to unpack .../5-python3-zipp_1.0.0-3_all.deb ...
Unpacking python3-zipp (1.0.0-3) ...
Selecting previously unselected package python3-importlib-metadata.
Preparing to unpack .../6-python3-importlib-metadata_1.6.0-2_all.deb ...
Unpacking python3-importlib-metadata (1.6.0-2) ...
Selecting previously unselected package python3-virtualenv.
Preparing to unpack .../7-python3-virtualenv_20.4.0+ds-2+deb11u1_all.deb ...
Unpacking python3-virtualenv (20.4.0+ds-2+deb11u1) ...
Selecting previously unselected package virtualenv.
Preparing to unpack .../8-virtualenv_20.4.0+ds-2+deb11u1_all.deb ...
Unpacking virtualenv (20.4.0+ds-2+deb11u1) ...
Setting up python3-filelock (3.0.12-2) ...
Setting up python3-distlib (0.3.2+really+0.3.1-0.1) ...
Setting up python3-six (1.16.0-2) ...
Setting up python3-appdirs (1.4.4-1) ...
Setting up python3-more-itertools (4.2.0-3) ...
Setting up python3-zipp (1.0.0-3) ...
Setting up python3-importlib-metadata (1.6.0-2) ...
Setting up python3-virtualenv (20.4.0+ds-2+deb11u1) ...
Setting up virtualenv (20.4.0+ds-2+deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```



```python
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ python3 -m venv venv
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ source venv/bin/activate
(venv) student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```



## 3. Get the sample

```sh
venv) student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ gsutil -m cp -r gs://spls/gsp066/cloud-vision .
Copying gs://spls/gsp066/cloud-vision/.git/hooks/fsmonitor-watchman.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/post-update.sample...
Copying gs://spls/gsp066/cloud-vision/.git/HEAD...
Copying gs://spls/gsp066/cloud-vision/.git/config...
Copying gs://spls/gsp066/cloud-vision/.git/description...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/applypatch-msg.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/commit-msg.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-applypatch.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-commit.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-merge-commit.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-push.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-rebase.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/pre-receive.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/update.sample...
Copying gs://spls/gsp066/cloud-vision/.git/hooks/prepare-commit-msg.sample...
Copying gs://spls/gsp066/cloud-vision/.git/index...
Copying gs://spls/gsp066/cloud-vision/.git/info/exclude...
Copying gs://spls/gsp066/cloud-vision/.git/logs/HEAD...
Copying gs://spls/gsp066/cloud-vision/.git/logs/refs/heads/master...
Copying gs://spls/gsp066/cloud-vision/.git/logs/refs/remotes/origin/HEAD...
Copying gs://spls/gsp066/cloud-vision/.git/objects/pack/pack-5c1bb6814125262a2d4f091f1548e8000d6bb33f.idx...
Copying gs://spls/gsp066/cloud-vision/.git/objects/pack/pack-5c1bb6814125262a2d4f091f1548e8000d6bb33f.pack...
Copying gs://spls/gsp066/cloud-vision/.git/packed-refs...
Copying gs://spls/gsp066/cloud-vision/.git/refs/heads/master...
Copying gs://spls/gsp066/cloud-vision/.git/refs/remotes/origin/HEAD...
Copying gs://spls/gsp066/cloud-vision/CONTRIBUTING.md...
Copying gs://spls/gsp066/cloud-vision/.gitignore...
Copying gs://spls/gsp066/cloud-vision/LICENSE...
Copying gs://spls/gsp066/cloud-vision/README.md...
Copying gs://spls/gsp066/cloud-vision/python/README.md...
Copying gs://spls/gsp066/cloud-vision/python/.gitignore...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/README.md...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/redis/Makefile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/redis/spec.yaml...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/Makefile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/Dockerfile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/.dockerignore...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/.gitignore...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/Makefile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/spec.tmpl.yaml...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/main.py...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/static/styles.css...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/templates/crawler_started.html...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/storage.py...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/templates/index.html...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/templates/label.html...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/webapp/src/templates/template.html...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/.dockerignore...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/.gitignore...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/Makefile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/spec.tmpl.yaml...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/Dockerfile...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/src/main.py...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/src/reddit.py...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/src/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/src/storage.py...
Copying gs://spls/gsp066/cloud-vision/python/awwvision/worker/src/vision.py...
Copying gs://spls/gsp066/cloud-vision/python/landmark_detection/README.md...
Copying gs://spls/gsp066/cloud-vision/python/landmark_detection/__init__.py...
Copying gs://spls/gsp066/cloud-vision/python/landmark_detection/detect_landmark.py...
Copying gs://spls/gsp066/cloud-vision/python/landmark_detection/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/text/README.md...
Copying gs://spls/gsp066/cloud-vision/python/text/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/text/textindex.py...
Copying gs://spls/gsp066/cloud-vision/python/twilio/README.md...
Copying gs://spls/gsp066/cloud-vision/python/text/textindex_test.py...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-k8s/Dockerfile...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-k8s/README.md...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-k8s/src/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-k8s/webapp-service.yaml...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-k8s/webapp-dep.yaml...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-labels/README.md...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-labels/requirements.txt...
Copying gs://spls/gsp066/cloud-vision/python/twilio/twilio-labels/whats_that.py...
Copying gs://spls/gsp066/cloud-vision/python/utils/README.md...
Copying gs://spls/gsp066/cloud-vision/python/utils/__init__.py...
Copying gs://spls/gsp066/cloud-vision/python/utils/generatejson.py...
Copying gs://spls/gsp066/cloud-vision/python/utils/generatejson_test.py...
Copying gs://spls/gsp066/cloud-vision/python/utils/generatejson_test_fake.jpg...
| [80/80 files][  4.2 MiB/  4.2 MiB] 100% Done
Operation completed over 80 objects/4.2 MiB.
(venv) student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$
```



## 4. Deploy the sample

```sh
(venv) student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-00-158d547a0831)$ cd cloud-vision/python/awwvision/
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ ls
Makefile  README.md  redis  webapp  worker
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ cat Makefile
.PHONY: all
all: redis webapp worker

.PHONY: delete
delete: delete-redis delete-webapp delete-worker

.PHONY: redis
redis:
        $(MAKE) -C redis all

.PHONY: webapp
webapp:
        $(MAKE) -C webapp all

.PHONY: worker
worker:
        $(MAKE) -C worker all

.PHONY: delete-redis
delete-redis:
        $(MAKE) -C redis delete

.PHONY: delete-webapp
delete-webapp:
        $(MAKE) -C webapp delete

.PHONY: delete-worker
delete-worker:
        $(MAKE) -C worker delete
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$
```



```sh
venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ ls
Makefile  README.md  redis  webapp  worker
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ cat webapp/Makefile
GCLOUD_PROJECT:=$(shell gcloud config list project --format="value(core.project)")

.PHONY: all
all: deploy

.PHONY: build
build:
        docker build -t gcr.io/$(GCLOUD_PROJECT)/awwvision-webapp .

.PHONY: push
push: build
        gcloud docker -- push gcr.io/$(GCLOUD_PROJECT)/awwvision-webapp

.PHONY: template
template:
        sed "s/\$$GCLOUD_PROJECT/$(GCLOUD_PROJECT)/g" spec.tmpl.yaml > spec.yaml

.PHONY: deploy
deploy: push template
        kubectl create -f spec.yaml

.PHONY: delete
delete: template
        kubectl delete --ignore-not-found -f spec.yaml
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ cat worker/Makefile
GCLOUD_PROJECT:=$(shell gcloud config list project --format="value(core.project)")

.PHONY: all
all: deploy

.PHONY: build
build:
        docker build -t gcr.io/$(GCLOUD_PROJECT)/awwvision-worker .

.PHONY: push
push: build
        gcloud docker -- push gcr.io/$(GCLOUD_PROJECT)/awwvision-worker

.PHONY: template
template:
        sed "s/\$$GCLOUD_PROJECT/$(GCLOUD_PROJECT)/g" spec.tmpl.yaml > spec.yaml

.PHONY: deploy
deploy: push template
        kubectl create -f spec.yaml

.PHONY: delete
delete: template
        kubectl delete --ignore-not-found -f spec.yaml
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$
```



````markdown
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ cat worker/Makefile
GCLOUD_PROJECT:=$(shell gcloud config list project --format="value(core.project)")

.PHONY: all
all: deploy

.PHONY: build
build:
        docker build -t gcr.io/$(GCLOUD_PROJECT)/awwvision-worker .

.PHONY: push
push: build
        gcloud docker -- push gcr.io/$(GCLOUD_PROJECT)/awwvision-worker

.PHONY: template
template:
        sed "s/\$$GCLOUD_PROJECT/$(GCLOUD_PROJECT)/g" spec.tmpl.yaml > spec.yaml

.PHONY: deploy
deploy: push template
        kubectl create -f spec.yaml

.PHONY: delete
delete: template
        kubectl delete --ignore-not-found -f spec.yaml
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ cat README.md
# Awwvision

*Awwvision* is a [Kubernetes](https://github.com/kubernetes/kubernetes/) and [Cloud Vision API](https://cloud.google.com/vision/) sample that uses the Vision API to classify (label) images from Reddit's [/r/aww](https://reddit.com/r/aww) subreddit, and display the labelled results in a web app.

Awwvision has three components:

1. A simple [Redis](http://redis.io/) instance.
2. A webapp that displays the labels and associated images.
3. A worker that handles scraping Reddit for images and classifying them using the Vision API. [Cloud Pub/Sub](https://cloud.google.com/pubsub/) is used to coordinate tasks between multiple worker instances.

## Prerequisites

1. Create a project in the [Google Cloud Platform Console](https://console.cloud.google.com).

2. [Enable billing](https://console.cloud.google.com/project/_/settings) for your project.

3. Enable the Vision and Pub/Sub APIs. See the ["Getting Started"](https://cloud.google.com/vision/docs/getting-started) page in the Vision API documentation for more information on using the Vision API.

4. Install the [Google Cloud SDK](https://cloud.google.com/sdk):

        $ curl https://sdk.cloud.google.com | bash
        $ gcloud init

5. Install and start up [Docker](https://www.docker.com/).

If you like, you can alternately run this tutorial from your project's
[Cloud Shell](https://cloud.google.com/shell/docs/).  In that case, you don't need to do steps 4 and 5.

## Create a Container Engine cluster

This example uses [Container Engine](https://cloud.google.com/container-engine/) to set up the Kubernetes cluster.

1. Create a cluster using `gcloud`. You can specify as many nodes as you want,
   but you need at least one. The `cloud-platform` scope is used to allow
   access to the Pub/Sub and Vision APIs.
   First set your zone, e.g.:

        gcloud config set compute/zone us-central1-f

   Then start up the cluster:

        gcloud container clusters create awwvision \
            --num-nodes 2 \
            --scopes cloud-platform

2. Set up the `kubectl` command-line tool to use the container's credentials.

        gcloud container clusters get-credentials awwvision

3. Verify that everything is working:

        kubectl cluster-info

## Deploy the sample

From the `awwvision` directory, use `make all` to build and deploy everything.
Make sure Docker is running first.

        make all

As part of the process, a Docker image will be built and uploaded to the
[GCR](https://cloud.google.com/container-registry/docs/) private container
registry. In addition, `.yaml` files will be generated from templates— filled in
with information specific to your project— and used to deploy the 'redis',
'webapp', and 'worker' Kubernetes resources for the example.

### Check the Kubernetes resources on the cluster

After you've deployed, check that the Kubernetes resources are up and running.
First, list the [pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/).
You should see something like the following, though your pod names will be different.

```
$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
awwvision-webapp-vwmr1   1/1       Running   0          1m
awwvision-worker-oz6xn   1/1       Running   0          1m
awwvision-worker-qc0b0   1/1       Running   0          1m
awwvision-worker-xpe53   1/1       Running   0          1m
redis-master-rpap8       1/1       Running   0          2m
```

List the
[deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).
You can see the number of replicas specified for each, and the images used.

```
$ kubectl get deployments -o wide
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS         IMAGES                                SELECTOR
awwvision-webapp   1         1         1            1           1m        awwvision-webapp   gcr.io/your-project/awwvision-webapp   app=awwvision,role=frontend
awwvision-worker   3         3         3            3           1m        awwvision-worker   gcr.io/your-project/awwvision-worker   app=awwvision,role=worker
redis-master       1         1         1            1           1m        redis-master       redis                                 app=redis,role=master
```

Once deployed, get the external IP address of the webapp
[service](https://kubernetes.io/docs/concepts/services-networking/service/).
It may take a few minutes for the assigned external IP to be
listed in the output.  After a short wait, you should see something like the
following, though your IPs will be different.

```
$ kubectl get svc awwvision-webapp
NAME               CLUSTER_IP      EXTERNAL_IP    PORT(S)   SELECTOR                      AGE
awwvision-webapp   10.163.250.49   23.236.61.91   80/TCP    app=awwvision,role=frontend   13m
```

### Visit your new webapp and start its crawler

Visit the external IP of the `awwvision-webapp` service to open the webapp in
your browser, and click the `Start the Crawler` button.

Next, click `go back`, and you should start to see images from the
[/r/aww](https://reddit.com/r/aww) subreddit classified by the labels provided
by the Vision API. You will see some of the images classified multiple times, where multiple
labels are detected for them.
(You can reload in a bit, in case you brought up the page before the crawler was
finished).

<a href="https://storage.googleapis.com/amy-jo/images/ubiquity/awwvision.png" target="_blank"><img src="https://storage.googleapis.com/amy-jo/images/ubiquity/awwvision.png" width=500/></a>

## Cleanup

To delete your Kubernetes pods, replication controllers, and services, and to
remove your auto-generated `.yaml` files, do:

        make delete

Note: this won't delete your Container Engine cluster itself.
If you are no longer using the cluster, you may want to take it down.
You can do this through the
[Google Cloud Platform Console](https://console.cloud.google.com).
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$
````



```makefile
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ make all
make -C redis all
make[1]: Entering directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/redis'
kubectl create -f spec.yaml
service/redis-master created
deployment.apps/redis-master created
make[1]: Leaving directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/redis'
make -C webapp all
make[1]: Entering directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/webapp'
docker build -t gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-webapp .
[+] Building 44.1s (10/10) FINISHED
 => [internal] load .dockerignore                                                                                                       0.0s
 => => transferring context: 120B                                                                                                       0.0s
 => [internal] load build definition from Dockerfile                                                                                    0.1s
 => => transferring dockerfile: 414B                                                                                                    0.0s
 => [internal] load metadata for gcr.io/google_appengine/python:latest                                                                  0.4s
 => [1/5] FROM gcr.io/google_appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6                  27.2s
 => => resolve gcr.io/google_appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6                   0.0s
 => => sha256:3c2cba919283a210665e480bcbf943eaaf4ed87a83f02e81bb286b8bdead0e75 49B / 49B                                                0.1s
 => => sha256:ce284ab20159170e6b579dac1c7b3013ac0d31bb421a5db89a016ac1287d42b0 6.12kB / 6.12kB                                          0.0s
 => => sha256:6c5b97b864a653d10b2da9fe9e48b9bea155199471b64a28c40a809c39393b00 46.72MB / 46.72MB                                        0.7s
 => => sha256:8ca77d5ce16632222cc29e986c56ca5fc37250470c8ce90c42d71a7f2ecf8c12 10.51MB / 10.51MB                                        0.2s
 => => sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6 2.83kB / 2.83kB                                          0.0s
 => => sha256:ebeccf37c57f19e886c07977d70aab82a7ce41fc2f7e0659c13d6693524f12d8 564B / 564B                                              0.2s
 => => sha256:42ebf62db3f550fab9733a5d864080b1e76252ece3cf95e115adbc41e74fffc0 339B / 339B                                              0.3s
 => => sha256:c0c3f74f3f7cca6be583ed147be75101be2ff55cf084e47e7117921dad8e8285 179.45MB / 179.45MB                                      3.7s
 => => sha256:6d5fad3b20f09eb4aa04055e27d971be5b0e14bc34f932ccbb59ca860ba9bb28 7.38MB / 7.38MB                                          0.5s
 => => sha256:906c5911ea69d0ec61518fe6da40e9ac813327ce05df636fa7dcdf1f752dbae8 114.82MB / 114.82MB                                      2.8s
 => => sha256:3c18b1c18dfc0e2327493926a6c5ef901f3df70ed08e8a4c9f879c70108d9def 1.27kB / 1.27kB                                          0.8s
 => => extracting sha256:6c5b97b864a653d10b2da9fe9e48b9bea155199471b64a28c40a809c39393b00                                               4.3s
 => => sha256:0360f916794e3fbe9c8ab4462a8aee8763d1d3f32a6d48508bca1e8839daf76d 32.54MB / 32.54MB                                        1.7s
 => => sha256:e90908c49002686c5a5c1b931cd27dad70df54a34ac0e53e3cb5c9a9771aa161 103B / 103B                                              1.8s
 => => sha256:392df4c613fda0a3dedc09db4351fe5d60960639f4503db21e4ba8d30f289458 166B / 166B                                              1.9s
 => => extracting sha256:8ca77d5ce16632222cc29e986c56ca5fc37250470c8ce90c42d71a7f2ecf8c12                                               0.6s
 => => extracting sha256:3c2cba919283a210665e480bcbf943eaaf4ed87a83f02e81bb286b8bdead0e75                                               0.0s
 => => extracting sha256:ebeccf37c57f19e886c07977d70aab82a7ce41fc2f7e0659c13d6693524f12d8                                               0.0s
 => => extracting sha256:42ebf62db3f550fab9733a5d864080b1e76252ece3cf95e115adbc41e74fffc0                                               0.0s
 => => extracting sha256:c0c3f74f3f7cca6be583ed147be75101be2ff55cf084e47e7117921dad8e8285                                               8.6s
 => => extracting sha256:6d5fad3b20f09eb4aa04055e27d971be5b0e14bc34f932ccbb59ca860ba9bb28                                               0.6s
 => => extracting sha256:906c5911ea69d0ec61518fe6da40e9ac813327ce05df636fa7dcdf1f752dbae8                                               8.3s
 => => extracting sha256:3c18b1c18dfc0e2327493926a6c5ef901f3df70ed08e8a4c9f879c70108d9def                                               0.0s
 => => extracting sha256:0360f916794e3fbe9c8ab4462a8aee8763d1d3f32a6d48508bca1e8839daf76d                                               1.9s
 => => extracting sha256:e90908c49002686c5a5c1b931cd27dad70df54a34ac0e53e3cb5c9a9771aa161                                               0.0s
 => => extracting sha256:392df4c613fda0a3dedc09db4351fe5d60960639f4503db21e4ba8d30f289458                                               0.0s
 => [internal] load build context                                                                                                       0.0s
 => => transferring context: 10.94kB                                                                                                    0.0s
 => [2/5] RUN virtualenv -p python3 /env                                                                                                3.5s
 => [3/5] ADD src/requirements.txt /app/requirements.txt                                                                                0.1s
 => [4/5] RUN pip install -r /app/requirements.txt                                                                                     11.8s
 => [5/5] ADD src /app                                                                                                                  0.1s
 => exporting to image                                                                                                                  0.9s
 => => exporting layers                                                                                                                 0.9s
 => => writing image sha256:bf6373e6547b763845e19165ea709330fb41ed071db13f8d6de2943527cb69b0                                            0.0s
 => => naming to gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-webapp                                                                   0.0s
gcloud docker -- push gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-webapp
WARNING: `gcloud docker` will not be supported for Docker client versions above 18.03.

As an alternative, use `gcloud auth configure-docker` to configure `docker` to
use `gcloud` as a credential helper, then use `docker` as you would for non-GCR
registries, e.g. `docker pull gcr.io/project-id/my-image`. Add
`--verbosity=error` to silence this warning: `gcloud docker
--verbosity=error -- pull gcr.io/project-id/my-image`.

See: https://cloud.google.com/container-registry/docs/support/deprecation-notices#gcloud-docker

Using default tag: latest
The push refers to repository [gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-webapp]
36a027973b51: Preparing
d3d3fffc7f92: Preparing
75bf2fc69e52: Preparing
d3d3fffc7f92: Pushed
087d7553d285: Layer already exists
16919ab89eca: Layer already exists
74bcef7f7402: Layer already exists
bc9e931c388e: Layer already exists
20896f2c3dd8: Layer already exists
7b80c69caf34: Layer already exists
3bbec54fac0c: Layer already exists
4006ffa4c683: Layer already exists
844d958e8cbe: Layer already exists
84ff92691f90: Layer already exists
b49bce339f97: Layer already exists
dcb7197db903: Layer already exists
latest: digest: sha256:9c270bcac84b694874dd28611c3f3c5b03afd2a908c1a78b36ecc2d23cb466a5 size: 3670
sed "s/\$GCLOUD_PROJECT/qwiklabs-gcp-00-158d547a0831/g" spec.tmpl.yaml > spec.yaml
kubectl create -f spec.yaml
service/awwvision-webapp created
deployment.apps/awwvision-webapp created
make[1]: Leaving directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/webapp'
make -C worker all
make[1]: Entering directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/worker'
docker build -t gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-worker .
[+] Building 34.0s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                                                                                    0.0s
 => => transferring dockerfile: 1.00kB                                                                                                  0.0s
 => [internal] load .dockerignore                                                                                                       0.0s
 => => transferring context: 120B                                                                                                       0.0s
 => [internal] load metadata for gcr.io/google_appengine/python:latest                                                                  0.1s
 => [1/5] FROM gcr.io/google_appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6                   0.0s
 => [internal] load build context                                                                                                       0.0s
 => => transferring context: 6.89kB                                                                                                     0.0s
 => CACHED [2/5] RUN virtualenv -p python3 /env                                                                                         0.0s
 => [3/5] ADD src/requirements.txt /app/requirements.txt                                                                                0.0s
 => [4/5] RUN pip install -r /app/requirements.txt                                                                                     33.0s
 => [5/5] ADD src /app                                                                                                                  0.1s
 => exporting to image                                                                                                                  0.7s
 => => exporting layers                                                                                                                 0.7s
 => => writing image sha256:5a52eac6d3ca3cdc69fd9f732789cc23b1176b9ea98019499a0f61f6113273ea                                            0.0s
 => => naming to gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-worker                                                                   0.0s
gcloud docker -- push gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-worker
WARNING: `gcloud docker` will not be supported for Docker client versions above 18.03.

As an alternative, use `gcloud auth configure-docker` to configure `docker` to
use `gcloud` as a credential helper, then use `docker` as you would for non-GCR
registries, e.g. `docker pull gcr.io/project-id/my-image`. Add
`--verbosity=error` to silence this warning: `gcloud docker
--verbosity=error -- pull gcr.io/project-id/my-image`.

See: https://cloud.google.com/container-registry/docs/support/deprecation-notices#gcloud-docker

Using default tag: latest
The push refers to repository [gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-worker]
7691144aa444: Preparing
c2b6852a8c41: Preparing
4f18a83e98d7: Preparing
c2b6852a8c41: Pushed
087d7553d285: Layer already exists
16919ab89eca: Layer already exists
74bcef7f7402: Layer already exists
bc9e931c388e: Layer already exists
20896f2c3dd8: Layer already exists
7b80c69caf34: Layer already exists
3bbec54fac0c: Layer already exists
4006ffa4c683: Layer already exists
844d958e8cbe: Layer already exists
84ff92691f90: Layer already exists
b49bce339f97: Layer already exists
dcb7197db903: Layer already exists
latest: digest: sha256:d1d0bbf7c1af19cb5aebede2b1ec0f67bfc7f2380cd39486e5c2d1270209ffc3 size: 3670
sed "s/\$GCLOUD_PROJECT/qwiklabs-gcp-00-158d547a0831/g" spec.tmpl.yaml > spec.yaml
kubectl create -f spec.yaml
deployment.apps/awwvision-worker created
make[1]: Leaving directory '/home/student_01_c41e1c322859/cloud-vision/python/awwvision/worker'
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$
```

As part of the process, Docker images were built and uploaded to the  [Google Container Registry](https://cloud.google.com/container-registry/docs) private container registry.

In addition, `yaml` files were generated from templates, filled in with information specific to your project, and used to deploy the `redis`, `webapp`, and `worker` Kubernetes resources for the lab.

## 5. Check the Kubernetes resources on the cluster

```sh
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ kubectl get pods
NAME                                READY   STATUS    RESTARTS      AGE
awwvision-webapp-797d47bddf-cpgjk   1/1     Running   1 (72s ago)   2m
awwvision-worker-f57cc9888-4dvw6    1/1     Running   0             75s
awwvision-worker-f57cc9888-npj8t    1/1     Running   0             75s
awwvision-worker-f57cc9888-xszzt    1/1     Running   0             75s
redis-master-bcbf646cf-tn6b4        1/1     Running   0             2m57s
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ kubectl get deploy -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS         IMAGES                                                 SELECTOR
awwvision-webapp   1/1     1            1           2m11s   awwvision-webapp   gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-webapp   app=awwvision
awwvision-worker   3/3     3            3           86s     awwvision-worker   gcr.io/qwiklabs-gcp-00-158d547a0831/awwvision-worker   app=awwvision
redis-master       1/1     1            1           3m8s    redis-master       redis                                                  app=redis
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$ kubectl get svc
NAME               TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
awwvision-webapp   LoadBalancer   10.52.0.109   34.31.121.222   80:31281/TCP   2m30s
kubernetes         ClusterIP      10.52.0.1     <none>          443/TCP        14m
redis-master       ClusterIP      10.52.5.215   <none>          6379/TCP       3m27s
(venv) student_01_c41e1c322859@cloudshell:~/cloud-vision/python/awwvision (qwiklabs-gcp-00-158d547a0831)$
```



## 6. Visit your new webapp and start its crawler

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-426.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-427.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-428.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-429.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-430.png)



We used Kubernetes and Cloud Vision API to classify images from  Reddit's /r/aww subreddit and displayed the results in a web app.

