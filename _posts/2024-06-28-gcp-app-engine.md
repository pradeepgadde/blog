---

layout: single
title:  "App Engine: Qwik Start - Python"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# App Engine: Qwik Start - Python

[App Engine](https://cloud.google.com/appengine) allows developers to focus on doing what they do best, writing code,  and not what it runs on. Developers upload their apps to App Engine, and Google Cloud takes care of the rest. The notion of servers, virtual  machines, and instances have been abstracted away, with App Engine  providing all the compute necessary. Developers don't have to worry  about operating systems, web servers, logging, monitoring,  load-balancing, system administration, or scaling, as App Engine takes  care of all that. Developers only need to focus on building solutions  for their organizations or their users.

The App Engine standard environment provides application-hosting  services supporting the following languages: Python, Java, PHP, Go,  Node.js, and Ruby). The App Engine flexible environment provides even  more flexibility by supporting custom runtimes, however it is  out-of-scope for this lab.

App Engine is Google Cloud's original serverless runtime, and since its original launch in 2008, has been joined by:

- [Cloud Functions](https://cloud.google.com/functions), great for situations where you don't have an entire app, have broken up a larger, monolithic app into multiple microservices, or have short  event-driven tasks that execute based on user activity.
- [Cloud Run](http://cloud.run), the serverless container-hosting service similar to App Engine but more accurately  reflects the state of software development today.

In this lab, you'll learn how to deploy a basic app to App Engine,  but we invite you to also explore Cloud Functions and Cloud Run. App  Engine makes it easy to build and deploy an application that runs  reliably even under heavy load and with large amounts of data. (Cloud  Functions and Cloud Run do the same.)

App Engine apps can access numerous additional Cloud or other Google services for use in their applications:

- **NoSQL database:** [Cloud Datastore](https://cloud.google.com/datastore), [Cloud Firestore](https://cloud.google.com/firestore), [Cloud BigTable](https://cloud.google.com/bigtable)
- **Relational database:** [Cloud SQL](https://cloud.google.com/sql) or [Cloud AlloyDB](https://cloud.google.com/alloydb), [Cloud Spanner](https://cloud.google.com/spanner)
- **File/object storage:** [Cloud Storage](https://cloud.google.com/storage), [Cloud Filestore](https://cloud.google.com/filestore), [Google Drive](https://developers.google.com/drive)
- **Caching:** [Cloud Memorystore](https://cloud.google.com/memorystore) (Redis or `memcached`)
- **Task execution:** [Cloud Tasks](https://cloud.google.com/tasks), [Cloud Pub/Sub](https://cloud.google.com/pubsub), [Cloud Scheduler](https://cloud.google.com/scheduler), [Cloud Workflows](https://cloud.google.com/workflows)
- **User authentication:** [Cloud Identity Platform](https://cloud.google.com/identity-platform), [Firebase Auth](https://firebase.google.com/docs/auth), [Google Identity Services](https://developers.google.com/identity)

Applications run in a secure, sandboxed environment, allowing App  Engine standard environment to distribute requests across multiple  servers, and scaling servers to meet traffic demands. Your application  runs within its own secure, reliable environment that is independent of  the hardware, operating system, or physical location of the server.

This hands-on lab shows you how to create a small App Engine application that displays a short message.

In this lab you'll do the following with a Python app:

- Clone/download
- Test
- Update
- Test
- Deploy

## Enable Google App Engine Admin API

The App Engine Admin API enables developers to provision and manage their App Engine Applications.

1. In the left **Navigation menu**, click **APIs & Services** > **Library**.
2. Type "App Engine Admin API" in the search box.
3. Click  the **App Engine Admin API** card.
4. Click **Enable**. If there is no prompt to enable the API, then it is already enabled and no action is needed.

## Download the Hello World app

There is a simple Hello World app for Python you can use to quickly  get a feel for deploying an app to Google Cloud. Follow these steps to  download Hello World to your Google Cloud instance.

1. Enter the following command to copy the Hello World sample app repository to your Google Cloud instance:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-479dcfea3fbd.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_8e06b6e74720@cloudshell:~ (qwiklabs-gcp-03-479dcfea3fbd)$ git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
Cloning into 'python-docs-samples'...
remote: Enumerating objects: 113408, done.
remote: Counting objects: 100% (552/552), done.
remote: Compressing objects: 100% (312/312), done.
remote: Total 113408 (delta 280), reused 446 (delta 217), pack-reused 112856
Receiving objects: 100% (113408/113408), 241.67 MiB | 12.90 MiB/s, done.
Resolving deltas: 100% (66855/66855), done.
Updating files: 100% (5064/5064), done.
student_00_8e06b6e74720@cloudshell:~ (qwiklabs-gcp-03-479dcfea3fbd)$ cd python-docs-samples/appengine/standard_python3/hello_world
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ ls
app.yaml  main.py  main_test.py  requirements-test.txt  requirements.txt
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat main.py 
# Copyright 2018 Google LLC
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

# [START gae_python38_app]
# [START gae_python3_app]
from flask import Flask


# If `entrypoint` is not defined in app.yaml, App Engine will look for an app
# called `app` in `main.py`.
app = Flask(__name__)


@app.route("/")
def hello():
    """Return a friendly HTTP greeting.

    Returns:
        A string with the words 'Hello World!'.
    """
    return "Hello World!"


if __name__ == "__main__":
    # This is used when running locally only. When deploying to Google App
    # Engine, a webserver process such as Gunicorn will serve the app. You
    # can configure startup instructions by adding `entrypoint` to app.yaml.
    app.run(host="127.0.0.1", port=8080, debug=True)
# [END gae_python3_app]
# [END gae_python38_app]
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat requirements.txt 
Flask==3.0.0
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat app.yaml 
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

runtime: python39
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ ls
app.yaml  main.py  main_test.py  requirements-test.txt  requirements.txt
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat main_test.py 
# Copyright 2018 Google Inc. All Rights Reserved.
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

import main


def test_index():
    main.app.testing = True
    client = main.app.test_client()

    r = client.get("/")
    assert r.status_code == 200
    assert "Hello World" in r.data.decode("utf-8")
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat requirements-test.txt 
pytest==7.0.1
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ 
```

## Test the application

Test the application using the Google Cloud development server (`dev_appserver.py`), which is included with the preinstalled App Engine SDK.

1. From within your helloworld directory where the app's  [app.yaml](https://cloud.google.com/appengine/docs/standard/python/config/appref) configuration file is located, start the Google Cloud development server with the following command:

```sh
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ dev_appserver.py app.yaml
/usr/lib/google-cloud-sdk/platform/google_appengine/google/protobuf/internal/api_implementation.py:100: UserWarning: Selected implementation upb is not available. Falling back to the python implementation.
  warnings.warn('Selected implementation upb is not available. '
INFO     2024-06-28 09:26:43,463 <string>:316] Skipping SDK update check.
WARNING  2024-06-28 09:26:43,463 <string>:325] The default encoding of your local Python interpreter is set to 'utf-8' while App Engine's production environment uses 'ascii'; as a result your code may behave differently when deployed.
WARNING  2024-06-28 09:26:43,504 simple_search_stub.py:1206] Could not read search indexes from /tmp/appengine.None.student_00_8e06b6e74720/search_indexes
INFO     2024-06-28 09:26:43,506 <string>:391] Starting API server at: http://localhost:43817
INFO     2024-06-28 09:26:43,510 instance_factory.py:127] Detected python version "b'Python 3.10.12\n'" for runtime "python39" at "/usr/bin/python".
INFO     2024-06-28 09:26:47,838 instance_factory.py:280] Using pip to install dependency libraries; pip stdout is redirected to /tmp/tmpe7seyb8n
INFO     2024-06-28 09:26:47,839 instance_factory.py:310] Running /tmp/tmpet3ox86r/bin/pip install --upgrade pip
                                                                                            
INFO     2024-06-28 09:26:50,797 instance_factory.py:310] Running /tmp/tmpet3ox86r/bin/pip install -r /tmp/tmp69qpfjrm
                                                                                
INFO     2024-06-28 09:26:53,477 dispatcher.py:267] Starting module "default" running at: http://localhost:8080
INFO     2024-06-28 09:26:53,478 admin_server.py:67] Starting admin server at: http://localhost:8000
INFO     2024-06-28 09:26:54,479 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-03-479dcfea3fbd in environment variables
[2024-06-28 09:26:54 +0000] [857] [INFO] Starting gunicorn 22.0.0
[2024-06-28 09:26:54 +0000] [857] [INFO] Listening at: http://0.0.0.0:39653 (857)
[2024-06-28 09:26:54 +0000] [857] [INFO] Using worker: sync
[2024-06-28 09:26:54 +0000] [859] [INFO] Booting worker with pid: 859
INFO     2024-06-28 09:26:55,486 instance.py:293] Instance PID: 857
INFO     2024-06-28 09:26:55,588 module.py:413] [default] Detected file changes:
  /home/student_00_8e06b6e74720/python-docs-samples/appengine/standard_python3/hello_world/__pycache__
INFO     2024-06-28 09:27:09,369 module.py:830] default: "GET /?authuser=2&redirectedPreviously=true HTTP/1.1" 200 12
INFO     2024-06-28 09:27:09,487 module.py:830] default: "GET /favicon.ico HTTP/1.1" 404 207
INFO     2024-06-28 09:27:09,605 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-03-479dcfea3fbd in environment variables
[2024-06-28 09:27:09 +0000] [861] [INFO] Starting gunicorn 22.0.0
[2024-06-28 09:27:09 +0000] [861] [INFO] Listening at: http://0.0.0.0:40705 (861)
[2024-06-28 09:27:09 +0000] [861] [INFO] Using worker: sync
[2024-06-28 09:27:09 +0000] [863] [INFO] Booting worker with pid: 863
INFO     2024-06-28 09:27:10,610 instance.py:293] Instance PID: 861

```

## Make a change

You can leave the development server running while you develop your  application. The development server watches for changes in your source  files and reloads them if necessary.

Let's try it. Leave the development server running. We'll open another command line window, then edit `main.py` to change "Hello World!" to "Hello, Cruel World!".

1. Click the (**+**) next to your Cloud Shell tab to open a new command line session.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-479dcfea3fbd.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_8e06b6e74720@cloudshell:~ (qwiklabs-gcp-03-479dcfea3fbd)$ cd python-docs-samples/appengine/standard_python3/hello_world
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ vi main.py 
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ cat main.py 
# Copyright 2018 Google LLC
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

# [START gae_python38_app]
# [START gae_python3_app]
from flask import Flask


# If `entrypoint` is not defined in app.yaml, App Engine will look for an app
# called `app` in `main.py`.
app = Flask(__name__)


@app.route("/")
def hello():
    """Return a friendly HTTP greeting.

    Returns:
        A string with the words 'Hello World!'.
    """
    return "Hello Cruel World!"


if __name__ == "__main__":
    # This is used when running locally only. When deploying to Google App
    # Engine, a webserver process such as Gunicorn will serve the app. You
    # can configure startup instructions by adding `entrypoint` to app.yaml.
    app.run(host="127.0.0.1", port=8080, debug=True)
# [END gae_python3_app]
# [END gae_python38_app]
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ 
```

```sh
INFO     2024-06-28 09:29:07,739 module.py:413] [default] Detected file changes:
  /home/student_00_8e06b6e74720/python-docs-samples/appengine/standard_python3/hello_world/__pycache__/main.cpython-310.pyc.136404621274512
INFO     2024-06-28 09:29:12,741 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-03-479dcfea3fbd in environment variables
[2024-06-28 09:29:12 +0000] [1258] [INFO] Starting gunicorn 22.0.0
[2024-06-28 09:29:12 +0000] [1258] [INFO] Listening at: http://0.0.0.0:42751 (1258)
[2024-06-28 09:29:12 +0000] [1258] [INFO] Using worker: sync
[2024-06-28 09:29:12 +0000] [1260] [INFO] Booting worker with pid: 1260
INFO     2024-06-28 09:29:13,747 instance.py:293] Instance PID: 1258
INFO     2024-06-28 09:29:16,444 module.py:830] default: "GET /?authuser=2 HTTP/1.1" 200 18
[2024-06-28 09:29:20 +0000] [863] [INFO] Parent changed, shutting down: <Worker 863>
[2024-06-28 09:29:20 +0000] [863] [INFO] Worker exiting (pid: 863)

```

## Deploy your app

1. To deploy your app to App Engine, run the following command from  within the root directory of your application where the app.yaml file is located:
2. Enter the number that represents your region: 

1. The App Engine application will then be created.
2. Enter **Y** when prompted to confirm the details and begin the deployment of service.

```sh
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ gcloud app deploy
You are creating an app for project [qwiklabs-gcp-03-479dcfea3fbd].
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

Creating App Engine application in project [qwiklabs-gcp-03-479dcfea3fbd] and region [us-east1]....done.                                                                           
Services to deploy:

descriptor:                  [/home/student_00_8e06b6e74720/python-docs-samples/appengine/standard_python3/hello_world/app.yaml]
source:                      [/home/student_00_8e06b6e74720/python-docs-samples/appengine/standard_python3/hello_world]
target project:              [qwiklabs-gcp-03-479dcfea3fbd]
target service:              [default]
target version:              [20240628t093046]
target url:                  [https://qwiklabs-gcp-03-479dcfea3fbd.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-03-479dcfea3fbd@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 6 files to Google Cloud Storage
17%
33%
50%
67%
83%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-03-479dcfea3fbd.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ 
```

## View your application

- To launch your browser enter the following command, then click on the link it provides:

```sh
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-03-479dcfea3fbd.ue.r.appspot.com
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ 
```

Your application is deployed and you can read the short message in your browser.

## History

```sh
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ history 
    1  git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
    2  cd python-docs-samples/appengine/standard_python3/hello_world
    3  ls
    4  cat main
    5  cat main,
    6  cat main.py 
    7  cat requirements.txt 
    8  cat app.yaml 
    9  ls
   10  cat main_test.py 
   11  cat requirements-test.txt 
   12  cd python-docs-samples/appengine/standard_python3/hello_world
   13  vi main.py 
   14  cat main.py 
   15  gcloud app deploy
   16  gcloud app browse
   17  history 
student_00_8e06b6e74720@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-03-479dcfea3fbd)$ 
```

