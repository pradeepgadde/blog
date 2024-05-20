---

layout: single
title:  "Distributed Load Testing Using Kubernetes"
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
# Distributed Load Testing Using Kubernetes

In this lab you will learn how to use Kubernetes Engine to deploy a  distributed load testing framework. The framework uses multiple  containers to create load testing traffic for a simple REST-based API.  Although this solution tests a simple web application, the same pattern  can be used to create more complex load testing scenarios such as gaming or Internet-of-Things (IoT) applications. This solution discusses the  general architecture of a container-based load testing framework.

### System under test

For this lab the system under test is a small web application  deployed to Google App Engine. The application exposes basic REST-style  endpoints to capture incoming HTTP POST requests (incoming data is not  persisted).

### Example workloads

The application that you'll deploy is modeled after the backend  service component found in many Internet-of-Things (IoT) deployments.  Devices first register with the service and then begin reporting metrics or sensor readings, while also periodically re-registering with the  service.

To model this interaction, you'll use `Locust`, a  distributed, Python-based load testing tool that is capable of  distributing requests across multiple target paths. For example, Locust  can distribute requests to the `/login` and `/metrics` target paths.

The workload is based on the interaction described above and is  modeled as a set of Tasks in Locust. To approximate real-world clients,  each Locust task is weighted. For example, registration happens once per thousand total client requests.

### Container-based computing

- The Locust container image is a Docker image that contains the Locust software.
- A `container cluster` consists of at least one cluster  master and multiple worker machines called nodes. These master and node  machines run the Kubernetes cluster orchestration system. For more information about clusters, see the [Kubernetes Engine documentation](https://cloud.google.com/kubernetes-engine/docs/)
- A `pod` is one or more containers deployed together on one host, and the smallest compute unit that can be defined, deployed, and  managed. Some pods contain only a single container. For example, in this lab, each of the Locust containers runs in its own pod.
- A `Deployment controller` provides declarative updates for Pods and ReplicaSets. This lab has two deployments: one for `locust-master` and other for `locust-worker`.

### Services

A particular pod can disappear for a variety of reasons, including  node failure or intentional node disruption for updates or maintenance.  This means that the IP address of a pod does not provide a reliable  interface for that pod. A more reliable approach would use an abstract  representation of that interface that never changes, even if the  underlying pod disappears and is replaced by a new pod with a different  IP address. A Kubernetes Engine `service` provides this type of abstract interface by defining a logical set of pods and a policy for accessing them.

In this lab there are several services that represent pods or sets of pods. For example, there is a service for the DNS server pod, another  service for the Locust master pod, and a service that represents all 10  Locust worker pods.

- Create a system under test i.e. a small web application deployed to Google App Engine. 
- Use Kubernetes Engine to deploy a distributed load testing framework.
- Create load testing traffic for a simple REST-based API.

## Set project and zone

- Define environment variables for the `project id`, `region` and `zone` you want to use for the lab.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-32db76504dc6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_7771ea220d2b@cloudshell:~ (qwiklabs-gcp-04-32db76504dc6)$ PROJECT=$(gcloud config get-value project)
REGION=us-east1
ZONE=us-east1-d
CLUSTER=gke-load-test
TARGET=${PROJECT}.appspot.com
gcloud config set compute/region $REGION
gcloud config set compute/zone $ZONE
Your active configuration is: [cloudshell-19139]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_7771ea220d2b@cloudshell:~ (qwiklabs-gcp-04-32db76504dc6)$ 
```



## Get the sample code and build a Docker image for the application

```sh
tudent_01_7771ea220d2b@cloudshell:~ (qwiklabs-gcp-04-32db76504dc6)$ gsutil -m cp -r gs://spls/gsp182/distributed-load-testing-using-kubernetes .
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/FETCH_HEAD...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/HEAD... 
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/post-update.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/commit-msg.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/fsmonitor-watchman.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/pre-applypatch.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/config...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/pre-commit.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/applypatch-msg.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/description...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/pre-push.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/pre-rebase.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/pre-receive.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/prepare-commit-msg.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/hooks/update.sample...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/index...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/info/exclude...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/logs/HEAD...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/logs/refs/heads/master...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/logs/refs/remotes/origin/HEAD...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/objects/pack/pack-7e64e9c4d6a23b9f6e649a04c0f5ba809bb803fc.idx...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/objects/pack/pack-7e64e9c4d6a23b9f6e649a04c0f5ba809bb803fc.pack...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/packed-refs...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/refs/heads/master...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.git/refs/remotes/origin/HEAD...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/.gitignore...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/Dockerfile...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/LICENSE...   
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/README.md... 
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/Dockerfile...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/Flask/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/Jinja2/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/MarkupSafe/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/Werkzeug/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/gevent/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/greenlet/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/itsdangerous/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/licenses.txt...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/locustio/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/msgpack-python/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/pyzmq/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/licenses/requests/LICENSE...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/locust-tasks/requirements.txt...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/locust-tasks/run.sh...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/docker-image/locust-tasks/tasks.py...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/flow.groovy...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/kubernetes-config/locust-master-service.yaml...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/kubernetes-config/locust-master-controller.yaml...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/kubernetes-config/locust-worker-controller.yaml...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/sample-webapp/app.yaml...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/sample-webapp/main.py...
Copying gs://spls/gsp182/distributed-load-testing-using-kubernetes/sample-webapp/requirements.txt...
- [52/52 files][118.2 KiB/118.2 KiB] 100% Done                                  
Operation completed over 52 objects/118.2 KiB.                                   
student_01_7771ea220d2b@cloudshell:~ (qwiklabs-gcp-04-32db76504dc6)$ 
```

```sh
student_01_7771ea220d2b@cloudshell:~ (qwiklabs-gcp-04-32db76504dc6)$ cd distributed-load-testing-using-kubernetes/sample-webapp/
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes/sample-webapp (qwiklabs-gcp-04-32db76504dc6)$ sed -i "s/python37/python39/g" app.yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes/sample-webapp (qwiklabs-gcp-04-32db76504dc6)$ cd ..
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$
```

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ gcloud builds submit --tag gcr.io/$PROJECT/locust-tasks:latest docker-image/.
Creating temporary archive of 16 file(s) totalling 18.3 KiB before compression.
Uploading tarball of [docker-image/.] to [gs://qwiklabs-gcp-04-32db76504dc6_cloudbuild/source/1716171339.2951-44abf0da5c564e3a9c9e52bcdf4e08cf.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-04-32db76504dc6/locations/global/builds/e9d86a1d-d33a-4395-8181-4c3d21f16d0b].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/e9d86a1d-d33a-4395-8181-4c3d21f16d0b?project=135945371566 ].
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "e9d86a1d-d33a-4395-8181-4c3d21f16d0b"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-04-32db76504dc6_cloudbuild/source/1716171339.2951-44abf0da5c564e3a9c9e52bcdf4e08cf.tgz#1716171341307083
Copying gs://qwiklabs-gcp-04-32db76504dc6_cloudbuild/source/1716171339.2951-44abf0da5c564e3a9c9e52bcdf4e08cf.tgz#1716171341307083...
/ [1 files][  4.9 KiB/  4.9 KiB]                                                
Operation completed over 1 objects/4.9 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  39.42kB
Step 1/7 : FROM python:3.7.2
3.7.2: Pulling from library/python
e79bb959ec00: Pulling fs layer
d4b7902036fe: Pulling fs layer
1b2a72d4e030: Pulling fs layer
d54db43011fd: Pulling fs layer
69d473365bb3: Pulling fs layer
7dc3a6a0e509: Pulling fs layer
a288a79001c3: Pulling fs layer
7d3cdae56021: Pulling fs layer
dbf17696f820: Pulling fs layer
d54db43011fd: Waiting
a288a79001c3: Waiting
7d3cdae56021: Waiting
dbf17696f820: Waiting
69d473365bb3: Waiting
7dc3a6a0e509: Waiting
1b2a72d4e030: Download complete
d4b7902036fe: Verifying Checksum
d4b7902036fe: Download complete
e79bb959ec00: Verifying Checksum
e79bb959ec00: Download complete
d54db43011fd: Verifying Checksum
d54db43011fd: Download complete
7dc3a6a0e509: Verifying Checksum
7dc3a6a0e509: Download complete
7d3cdae56021: Verifying Checksum
7d3cdae56021: Download complete
dbf17696f820: Download complete
a288a79001c3: Verifying Checksum
a288a79001c3: Download complete
69d473365bb3: Verifying Checksum
69d473365bb3: Download complete
e79bb959ec00: Pull complete
d4b7902036fe: Pull complete
1b2a72d4e030: Pull complete
d54db43011fd: Pull complete
69d473365bb3: Pull complete
7dc3a6a0e509: Pull complete
a288a79001c3: Pull complete
7d3cdae56021: Pull complete
dbf17696f820: Pull complete
Digest: sha256:1eecfb3b9cee73af760e00d3aa59bae76c4c0160ddb0213f14f7e0ae968e4835
Status: Downloaded newer image for python:3.7.2
 2053ca75899e
Step 2/7 : ADD licenses /licenses
 00ca4cea700f
Step 3/7 : ADD locust-tasks /locust-tasks
 212cc0b678a5
Step 4/7 : RUN pip install -r /locust-tasks/requirements.txt
 Running in 7260127c0b49
Collecting certifi==2019.3.9 (from -r /locust-tasks/requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/60/75/f692a584e85b7eaba0e03827b3d51f45f571c2e793dd731e598828d380aa/certifi-2019.3.9-py2.py3-none-any.whl (158kB)
Collecting chardet==3.0.4 (from -r /locust-tasks/requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl (133kB)
Collecting Click==7.0 (from -r /locust-tasks/requirements.txt (line 3))
  Downloading https://files.pythonhosted.org/packages/fa/37/45185cb5abbc30d7257104c434fe0b07e5a195a6847506c074527aa599ec/Click-7.0-py2.py3-none-any.whl (81kB)
Collecting Flask==1.0.2 (from -r /locust-tasks/requirements.txt (line 4))
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
Collecting gevent==1.4.0 (from -r /locust-tasks/requirements.txt (line 5))
  Downloading https://files.pythonhosted.org/packages/d4/89/57b63d6d7967d8763b913172bf6831afb01951b9ed9da127f2938a365585/gevent-1.4.0-cp37-cp37m-manylinux1_x86_64.whl (5.4MB)
Collecting greenlet==0.4.15 (from -r /locust-tasks/requirements.txt (line 6))
  Downloading https://files.pythonhosted.org/packages/9d/ef/ac10aa1293f64939e4511909c570d969566126214af5dd7ba0afd353d88b/greenlet-0.4.15-cp37-cp37m-manylinux1_x86_64.whl (42kB)
Collecting idna==2.8 (from -r /locust-tasks/requirements.txt (line 7))
  Downloading https://files.pythonhosted.org/packages/14/2c/cd551d81dbe15200be1cf41cd03869a46fe7226e7450af7a6545bfc474c9/idna-2.8-py2.py3-none-any.whl (58kB)
Collecting itsdangerous==1.1.0 (from -r /locust-tasks/requirements.txt (line 8))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting Jinja2==2.10.1 (from -r /locust-tasks/requirements.txt (line 9))
  Downloading https://files.pythonhosted.org/packages/1d/e7/fd8b501e7a6dfe492a433deb7b9d833d39ca74916fa8bc63dd1a4947a671/Jinja2-2.10.1-py2.py3-none-any.whl (124kB)
Collecting locustio==0.11.0 (from -r /locust-tasks/requirements.txt (line 10))
  Downloading https://files.pythonhosted.org/packages/98/12/60351b28a00c76d36022d79bead7dc217e9e49cddd7091cb164319196323/locustio-0.11.0-py2.py3-none-any.whl (240kB)
Collecting MarkupSafe==1.1.1 (from -r /locust-tasks/requirements.txt (line 11))
  Downloading https://files.pythonhosted.org/packages/c2/37/2e4def8ce3739a258998215df907f5815ecd1af71e62147f5eea2d12d4e8/MarkupSafe-1.1.1-cp37-cp37m-manylinux2010_x86_64.whl
Collecting msgpack==0.6.1 (from -r /locust-tasks/requirements.txt (line 12))
  Downloading https://files.pythonhosted.org/packages/a8/7b/630049fc4af9e68a625738612edc264ce7cb586c5001a2d4d2209a4f61c1/msgpack-0.6.1-cp37-cp37m-manylinux1_x86_64.whl (245kB)
Collecting msgpack-python==0.5.6 (from -r /locust-tasks/requirements.txt (line 13))
  Downloading https://files.pythonhosted.org/packages/8a/20/6eca772d1a5830336f84aca1d8198e5a3f4715cd1c7fc36d3cc7f7185091/msgpack-python-0.5.6.tar.gz (138kB)
Collecting pyzmq==18.0.1 (from -r /locust-tasks/requirements.txt (line 14))
  Downloading https://files.pythonhosted.org/packages/3c/72/1259cea3a7bc0ace82afd2244fdc448c4c2fb6b49a14e87ba832e00ee86a/pyzmq-18.0.1-cp37-cp37m-manylinux1_x86_64.whl (1.1MB)
Collecting requests==2.21.0 (from -r /locust-tasks/requirements.txt (line 15))
  Downloading https://files.pythonhosted.org/packages/7d/e3/20f3d364d6c8e5d2353c72a67778eb189176f08e873c9900e10c0287b84b/requests-2.21.0-py2.py3-none-any.whl (57kB)
Collecting six==1.12.0 (from -r /locust-tasks/requirements.txt (line 16))
  Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Collecting urllib3==1.24.2 (from -r /locust-tasks/requirements.txt (line 17))
  Downloading https://files.pythonhosted.org/packages/df/1c/59cca3abf96f991f2ec3131a4ffe72ae3d9ea1f5894abe8a9c5e3c77cfee/urllib3-1.24.2-py2.py3-none-any.whl (131kB)
Collecting Werkzeug==0.15.1 (from -r /locust-tasks/requirements.txt (line 18))
  Downloading https://files.pythonhosted.org/packages/24/4d/2fc4e872fbaaf44cc3fd5a9cd42fda7e57c031f08e28c9f35689e8b43198/Werkzeug-0.15.1-py2.py3-none-any.whl (328kB)
Building wheels for collected packages: msgpack-python
  Building wheel for msgpack-python (setup.py): started
  Building wheel for msgpack-python (setup.py): finished with status 'done'
  Stored in directory: /root/.cache/pip/wheels/d5/de/86/7fa56fda12511be47ea0808f3502bc879df4e63ab168ec0406
Successfully built msgpack-python
Installing collected packages: certifi, chardet, Click, itsdangerous, MarkupSafe, Jinja2, Werkzeug, Flask, greenlet, gevent, idna, pyzmq, msgpack, urllib3, requests, six, locustio, msgpack-python
Successfully installed Click-7.0 Flask-1.0.2 Jinja2-2.10.1 MarkupSafe-1.1.1 Werkzeug-0.15.1 certifi-2019.3.9 chardet-3.0.4 gevent-1.4.0 greenlet-0.4.15 idna-2.8 itsdangerous-1.1.0 locustio-0.11.0 msgpack-0.6.1 msgpack-python-0.5.6 pyzmq-18.0.1 requests-2.21.0 six-1.12.0 urllib3-1.24.2
You are using pip version 19.0.3, however version 24.0 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container 7260127c0b49
 d2b81d88795d
Step 5/7 : EXPOSE 5557 5558 8089
 Running in 1ba2df0b05bc
Removing intermediate container 1ba2df0b05bc
 03b63e62b968
Step 6/7 : RUN chmod 755 /locust-tasks/run.sh
 Running in 0e4cf7c7a050
Removing intermediate container 0e4cf7c7a050
 b1e8dad23df2
Step 7/7 : ENTRYPOINT ["/locust-tasks/run.sh"]
 Running in b6d70ab7999c
Removing intermediate container b6d70ab7999c
 5af6b23fb89b
Successfully built 5af6b23fb89b
Successfully tagged gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks:latest
PUSH
Pushing gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks:latest
The push refers to repository [gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks]
4ec5cb84b121: Preparing
7aa119085edf: Preparing
768238be4269: Preparing
9f640e359cea: Preparing
b24438cea2cc: Preparing
c76a8d0f7f24: Preparing
b4074dfd8a23: Preparing
9f72e27f9aa4: Preparing
0fe19df8b8f8: Preparing
b17cc31e431b: Preparing
12cb127eee44: Preparing
604829a174eb: Preparing
fbb641a8b943: Preparing
c76a8d0f7f24: Waiting
b4074dfd8a23: Waiting
9f72e27f9aa4: Waiting
0fe19df8b8f8: Waiting
b17cc31e431b: Waiting
12cb127eee44: Waiting
604829a174eb: Waiting
fbb641a8b943: Waiting
b24438cea2cc: Layer already exists
c76a8d0f7f24: Layer already exists
b4074dfd8a23: Layer already exists
9f72e27f9aa4: Layer already exists
4ec5cb84b121: Pushed
0fe19df8b8f8: Layer already exists
768238be4269: Pushed
9f640e359cea: Pushed
b17cc31e431b: Layer already exists
604829a174eb: Layer already exists
fbb641a8b943: Layer already exists
12cb127eee44: Layer already exists
7aa119085edf: Pushed
latest: digest: sha256:b5f271564f59be5614e85d6ad12c6f864e4fb25de82bfb82fc896e038404b456 size: 3053
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ID: e9d86a1d-d33a-4395-8181-4c3d21f16d0b
CREATE_TIME: 2024-05-20T02:15:41+00:00
DURATION: 1M8S
SOURCE: gs://qwiklabs-gcp-04-32db76504dc6_cloudbuild/source/1716171339.2951-44abf0da5c564e3a9c9e52bcdf4e08cf.tgz
IMAGES: gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks (+1 more)
STATUS: SUCCESS
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```



## Deploy web application

The `sample-webapp` folder contains a simple Google App Engine Python application as the "system under test".

```yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ cat sample-webapp/app.yaml 
# Copyright 2015 Google Inc. All rights reserved.
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


runtime: python39

instance_class: F2

handlers:
- url: /.*
  script: auto
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

```py
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ cat sample-webapp/main.py 
#!/usr/bin/env python

# Copyright 2019 Google Inc. All rights reserved.
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


from flask import Flask, request

app = Flask(__name__)


@app.route('/')
def root():
    return 'Welcome to the "Distributed Load Testing Using Kubernetes" sample web app\n'

@app.route('/login',  methods=['GET', 'POST'])
def login():
    deviceid = request.values.get('deviceid')
    return '/login - device: {}\n'.format(deviceid)

@app.route('/metrics',  methods=['GET', 'POST'])
def metrics():
    deviceid = request.values.get('deviceid')
    timestamp = request.values.get('timestamp')
    
    return '/metrics - device: {}, timestamp: {}\n'.format(deviceid, timestamp)


if __name__ == '__main__':
    # This is used when running locally only. When deploying to Google App
    # Engine, a webserver process such as Gunicorn will serve the app. This
    # can be configured by adding an `entrypoint` to app.yaml.
    # Flask's development server will automatically serve static files in
    # the "static" directory. See:
    # http://flask.pocoo.org/docs/1.0/quickstart/#static-files. Once deployed,
    # App Engine itself will serve those files as configured in app.yaml.
    app.run(host='127.0.0.1', port=8080, debug=True)
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```



```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ gcloud app deploy sample-webapp/app.yaml
You are creating an app for project [qwiklabs-gcp-04-32db76504dc6].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Please choose the region where you want your App Engine application located:

 [1] asia-east1    (supports standard and flexible)
 [2] asia-northeast1 (supports standard and flexible and search_api)
 [3] asia-south1   (supports standard and flexible and search_api)
 [4] asia-southeast1 (supports standard and flexible)
 [5] australia-southeast1 (supports standard and flexible and search_api)
 [6] europe-central2 (supports standard and flexible)
 [7] europe-west   (supports standard and flexible and search_api)
 [8] europe-west2  (supports standard and flexible and search_api)
 [9] europe-west3  (supports standard and flexible and search_api)
 [10] us-central    (supports standard and flexible and search_api)
 [11] us-east1      (supports standard and flexible and search_api)
 [12] us-east4      (supports standard and flexible and search_api)
 [13] us-west1      (supports standard and flexible)
 [14] us-west2      (supports standard and flexible and search_api)
 [15] us-west3      (supports standard and flexible and search_api)
 [16] us-west4      (supports standard and flexible and search_api)
 [17] cancel
Please enter your numeric choice:  11

Creating App Engine application in project [qwiklabs-gcp-04-32db76504dc6] and region [us-east1]....done.                                                                           
Services to deploy:

descriptor:                  [/home/student_01_7771ea220d2b/distributed-load-testing-using-kubernetes/sample-webapp/app.yaml]
source:                      [/home/student_01_7771ea220d2b/distributed-load-testing-using-kubernetes/sample-webapp]
target project:              [qwiklabs-gcp-04-32db76504dc6]
target service:              [default]
target version:              [20240520t021858]
target url:                  [https://qwiklabs-gcp-04-32db76504dc6.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-04-32db76504dc6@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 4 files to Google Cloud Storage
25%
50%
75%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-04-32db76504dc6.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

You will need the URL of the deployed sample web application when deploying the `locust-master` and `locust-worker` deployments which is already stored in `TARGET` variable.

## Deploy Kubernetes cluster

- Create the [Google Kubernetes Engine](http://cloud.google.com/kubernetes-engine) cluster using the `gcloud` command shown below:

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ gcloud container clusters create $CLUSTER \
  --zone $ZONE \
  --num-nodes=5
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster gke-load-test in us-east1-d... Cluster is being health-checked (master is healthy)...done.                                                                        
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-04-32db76504dc6/zones/us-east1-d/clusters/gke-load-test].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-d/gke-load-test?project=qwiklabs-gcp-04-32db76504dc6
kubeconfig entry generated for gke-load-test.
NAME: gke-load-test
LOCATION: us-east1-d
MASTER_VERSION: 1.28.8-gke.1095000
MASTER_IP: 104.196.167.99
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.8-gke.1095000
NUM_NODES: 5
STATUS: RUNNING
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE   VERSION
gke-gke-load-test-default-pool-08c73817-0v36   Ready    <none>   60s   v1.28.8-gke.1095000
gke-gke-load-test-default-pool-08c73817-8jsc   Ready    <none>   64s   v1.28.8-gke.1095000
gke-gke-load-test-default-pool-08c73817-crpb   Ready    <none>   64s   v1.28.8-gke.1095000
gke-gke-load-test-default-pool-08c73817-drg8   Ready    <none>   64s   v1.28.8-gke.1095000
gke-gke-load-test-default-pool-08c73817-s4s2   Ready    <none>   63s   v1.28.8-gke.1095000
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get pods
No resources found in default namespace.
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get ns
NAME                 STATUS   AGE
default              Active   2m31s
gke-managed-system   Active   2m6s
gmp-public           Active   114s
gmp-system           Active   114s
kube-node-lease      Active   2m31s
kube-public          Active   2m31s
kube-system          Active   2m31s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.51.240.1   <none>        443/TCP   2m32s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```



## Load testing master

The first component of the deployment is the Locust master, which is  the entry point for executing the load testing tasks described above.  The Locust master is deployed with a single replica because we need only one master.

The configuration for the master deployment specifies several  elements, including the ports that need to be exposed by the container (`8089` for web interface, `5557` and `5558` for communicating with workers). This information is later used to configure the Locust workers.



## Deploy locust-master

1. Replace `[TARGET_HOST]` and `[PROJECT_ID]` in `locust-master-controller.yaml` and `locust-worker-controller.yaml` with the deployed endpoint and project-id respectively.

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-master-controller.yaml
sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-worker-controller.yaml
sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-master-controller.yaml
sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-worker-controller.yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$
```



```yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ cat kubernetes-config/locust-master-controller.yaml
# Copyright 2015 Google Inc. All rights reserved.
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


apiVersion: "apps/v1"
kind: "Deployment"
metadata:
  name: locust-master
  labels:
    name: locust-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: locust-master
  template:
    metadata:
      labels:
        app: locust-master
    spec:
      containers:
        - name: locust-master
          image: gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks:latest
          env:
            - name: LOCUST_MODE
              value: master
            - name: TARGET_HOST
              value: https://qwiklabs-gcp-04-32db76504dc6.appspot.com
          ports:
            - name: loc-master-web
              containerPort: 8089
              protocol: TCP
            - name: loc-master-p1
              containerPort: 5557
              protocol: TCP
            - name: loc-master-p2
              containerPort: 5558
              protocol: TCP
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

```yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ cat kubernetes-config/locust-worker-controller.yaml 
# Copyright 2015 Google Inc. All rights reserved.
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

apiVersion: "apps/v1"
kind: "Deployment"
metadata:
  name: locust-worker
  labels:
    name: locust-worker
spec:
  replicas: 5
  selector:
    matchLabels:
      app: locust-worker
  template:
    metadata:
      labels:
        app: locust-worker
    spec:
      containers:
        - name: locust-worker
          image: gcr.io/qwiklabs-gcp-04-32db76504dc6/locust-tasks:latest
          env:
            - name: LOCUST_MODE
              value: worker
            - name: LOCUST_MASTER
              value: locust-master
            - name: TARGET_HOST
              value: https://qwiklabs-gcp-04-32db76504dc6.appspot.com
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

```yaml
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ cat kubernetes-config/locust-master-service.yaml 
# Copyright 2015 Google Inc. All rights reserved.
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


kind: Service
apiVersion: v1
metadata:
  name: locust-master
  labels:
    app: locust-master
spec:
  ports:
    - port: 8089
      targetPort: loc-master-web
      protocol: TCP
      name: loc-master-web
    - port: 5557
      targetPort: loc-master-p1
      protocol: TCP
      name: loc-master-p1
    - port: 5558
      targetPort: loc-master-p2
      protocol: TCP
      name: loc-master-p2
  selector:
    app: locust-master
  type: LoadBalancer
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl apply -f kubernetes-config/locust-master-controller.yaml
deployment.apps/locust-master created
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get pods -l app=locust-master
NAME                             READY   STATUS              RESTARTS   AGE
locust-master-56876895bc-fq7r6   0/1     ContainerCreating   0          9s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl apply -f kubernetes-config/locust-master-service.yaml
service/locust-master created
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get svc locust-master
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                        AGE
locust-master   LoadBalancer   10.51.251.92   <pending>     8089:31893/TCP,5557:31527/TCP,5558:31684/TCP   6s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get pods -l app=locust-master
NAME                             READY   STATUS    RESTARTS   AGE
locust-master-56876895bc-fq7r6   1/1     Running   0          48s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get svc locust-master
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                        AGE
locust-master   LoadBalancer   10.51.251.92   35.196.106.66   8089:31893/TCP,5557:31527/TCP,5558:31684/TCP   55s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```





## Load testing workers

The next component of the deployment includes the Locust workers,  which execute the load testing tasks described above. The Locust workers are deployed by a single deployment that creates multiple pods. The  pods are spread out across the Kubernetes cluster. Each pod uses  environment variables to control important configuration information  such as the hostname of the system under test and the hostname of the  Locust master.

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl apply -f kubernetes-config/locust-worker-controller.yaml
deployment.apps/locust-worker created
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get pods -l app=locust-worker
NAME                             READY   STATUS              RESTARTS   AGE
locust-worker-6bd869cb4b-72h4g   0/1     ContainerCreating   0          13s
locust-worker-6bd869cb4b-jzcp6   1/1     Running             0          13s
locust-worker-6bd869cb4b-s4hr4   0/1     ContainerCreating   0          13s
locust-worker-6bd869cb4b-wpdsz   0/1     ContainerCreating   0          13s
locust-worker-6bd869cb4b-x547b   1/1     Running             0          13s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

Scaling up the number of simulated users will require an increase in the number of Locust worker pods. To increase the number of pods deployed  by the deployment, Kubernetes offers the ability to resize deployments  without redeploying them.

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl scale deployment/locust-worker --replicas=20
deployment.apps/locust-worker scaled
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ kubectl get pods -l app=locust-worker
NAME                             READY   STATUS              RESTARTS   AGE
locust-worker-6bd869cb4b-72h4g   1/1     Running             0          83s
locust-worker-6bd869cb4b-7gxwh   1/1     Running             0          7s
locust-worker-6bd869cb4b-8f4t8   1/1     Running             0          7s
locust-worker-6bd869cb4b-9jtm7   1/1     Running             0          7s
locust-worker-6bd869cb4b-b5lzb   1/1     Running             0          7s
locust-worker-6bd869cb4b-b7tkx   1/1     Running             0          8s
locust-worker-6bd869cb4b-bdgmz   1/1     Running             0          7s
locust-worker-6bd869cb4b-bp5p9   1/1     Running             0          7s
locust-worker-6bd869cb4b-gkkcv   0/1     ContainerCreating   0          7s
locust-worker-6bd869cb4b-j759b   1/1     Running             0          7s
locust-worker-6bd869cb4b-jzcp6   1/1     Running             0          83s
locust-worker-6bd869cb4b-kttt5   1/1     Running             0          7s
locust-worker-6bd869cb4b-nr95b   1/1     Running             0          7s
locust-worker-6bd869cb4b-ntqdg   0/1     ContainerCreating   0          7s
locust-worker-6bd869cb4b-rbf8m   1/1     Running             0          7s
locust-worker-6bd869cb4b-s4hr4   1/1     Running             0          83s
locust-worker-6bd869cb4b-tn5nx   1/1     Running             0          7s
locust-worker-6bd869cb4b-wpdsz   1/1     Running             0          83s
locust-worker-6bd869cb4b-x547b   1/1     Running             0          83s
locust-worker-6bd869cb4b-xx5lf   1/1     Running             0          7s
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```



After the Locust workers are deployed, you can return to the Locust  master web interface and see that the number of slaves corresponds to  the number of deployed workers.

The following diagram shows the relationship between the Locust master and the Locust workers:

![The flow from the Locust master to the Locust worker to the application](https://cdn.qwiklabs.com/QYSigq8YwCYxOoT4X8EHZuugeOjPlOlz5kOY57CMR8I%3D)

## Execute tests

1. To execute the Locust tests, get the external IP address by following command

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ EXTERNAL_IP=$(kubectl get svc locust-master -o yaml | grep ip: | awk -F": " '{print $NF}')
echo http://$EXTERNAL_IP:8089
http://35.196.106.66:8089
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```



The Locust master web interface enables you to execute the load testing tasks against the system under test.

1. To begin, specify the total number of users to simulate and a rate at which each user should be spawned.
2. Next, click Start swarming to begin the simulation. For example you can specify **Number of users to simulate** as 300 and **Hatch rate** as 10.
3. Click **Start swarming**.

As time progresses and users are spawned, statistics aggregate for  simulation metrics, such as the number of requests and requests per  second.

1. To stop the simulation, click **Stop** and the test will terminate. The complete results can be downloaded into a spreadsheet.

## History

```sh
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ history 
    1  PROJECT=$(gcloud config get-value project)
    2  REGION=us-east1
    3  ZONE=us-east1-d
    4  CLUSTER=gke-load-test
    5  TARGET=${PROJECT}.appspot.com
    6  gcloud config set compute/region $REGION
    7  gcloud config set compute/zone $ZONE
    8  gsutil -m cp -r gs://spls/gsp182/distributed-load-testing-using-kubernetes .
    9  cd distributed-load-testing-using-kubernetes/sample-webapp/
   10  sed -i "s/python37/python39/g" app.yaml
   11  cd ..
   12  gcloud builds submit --tag gcr.io/$PROJECT/locust-tasks:latest docker-image/.
   13  gcloud app deploy sample-webapp/app.yaml
   14  cat sample-webapp/app.yaml 
   15  cat sample-webapp/main.py 
   16  gcloud container clusters create $CLUSTER   --zone $ZONE   --num-nodes=5
   17  kubectl get nodes
   18  kubectl get pods
   19  kubectl get ns
   20  kubectl get svc
   21  ls
   22  ls kubernetes-config/locust-master-
   23  ls kubernetes-config/locust-master-controller.yaml 
   24  cat kubernetes-config/locust-master-controller.yaml
   25  sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-master-controller.yaml
   26  sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-worker-controller.yaml
   27  sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-master-controller.yaml
   28  sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-worker-controller.yaml
   29  cat kubernetes-config/locust-master-controller.yaml
   30  cat kubernetes-config/locust-worker-controller.yaml 
   31  cat kubernetes-config/locust-master-service.yaml 
   32  kubectl apply -f kubernetes-config/locust-master-controller.yaml
   33  kubectl get pods -l app=locust-master
   34  kubectl apply -f kubernetes-config/locust-master-service.yaml
   35  kubectl get svc locust-master
   36  kubectl get pods -l app=locust-master
   37  kubectl get svc locust-master
   38  kubectl apply -f kubernetes-config/locust-worker-controller.yaml
   39  kubectl get pods -l app=locust-worker
   40  kubectl scale deployment/locust-worker --replicas=20
   41  kubectl get pods -l app=locust-worker
   42  EXTERNAL_IP=$(kubectl get svc locust-master -o yaml | grep ip: | awk -F": " '{print $NF}')
   43  echo http://$EXTERNAL_IP:8089
   44  history 
student_01_7771ea220d2b@cloudshell:~/distributed-load-testing-using-kubernetes (qwiklabs-gcp-04-32db76504dc6)$ 
```

