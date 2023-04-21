---
layout: single
title:  "Error Reporting and Debugging"
date:   2023-04-21 10:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Error Reporting and Debugging

Perform the following tasks:

- Launch a simple Google App Engine application
- Introduce an error into the application
- Explore Cloud Error Reporting
- Use Cloud Debugger to identify the error in the code
- Fix the bug and monitor in Cloud Operations



## 1. Create an application

### Get and test the application

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-dc79aa408817.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_821796f8d08e@cloudshell:~ (qwiklabs-gcp-01-dc79aa408817)$ mkdir appengine-hello
cd appengine-hello
gsutil cp gs://cloud-training/archinfra/gae-hello/* .
Copying gs://cloud-training/archinfra/gae-hello/app.yaml...
Copying gs://cloud-training/archinfra/gae-hello/main.py...
Copying gs://cloud-training/archinfra/gae-hello/main.pyc...
Copying gs://cloud-training/archinfra/gae-hello/main_test.py...
- [4 files][  2.5 KiB/  2.5 KiB]
Operation completed over 4 objects/2.5 KiB.
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```



```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ dev_appserver.py $(pwd)
INFO     2023-04-21 07:54:26,913 devappserver2.py:321] Skipping SDK update check.
WARNING  2023-04-21 07:54:27,114 simple_search_stub.py:1196] Could not read search indexes from /tmp/appengine.None.student_01_821796f8d08e/search_indexes
INFO     2023-04-21 07:54:27,117 <string>:398] Starting API server at: http://localhost:39447
INFO     2023-04-21 07:54:27,127 dispatcher.py:276] Starting module "default" running at: http://localhost:8080
INFO     2023-04-21 07:54:27,128 admin_server.py:70] Starting admin server at: http://localhost:8000
INFO     2023-04-21 07:54:29,140 instance.py:294] Instance PID: 710
INFO     2023-04-21 07:54:29,140 module.py:1919] New instance for module "default" serving on:
http://localhost:8080

INFO     2023-04-21 07:54:29,762 module.py:862] default: "GET /_ah/start HTTP/1.1" 404 52

```



```sh
^CINFO     2023-04-21 07:55:37,132 shutdown.py:50] Shutting down.
INFO     2023-04-21 07:55:37,132 stub_util.py:360] Applying all pending transactions and saving the datastore
INFO     2023-04-21 07:55:37,133 stub_util.py:363] Saving search indexes
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```

### Deploy the application to App Engine

1. To deploy the application to App Engine, run the following command:

```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ gcloud app deploy app.yaml
You are creating an app for project [qwiklabs-gcp-01-dc79aa408817].
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

Creating App Engine application in project [qwiklabs-gcp-01-dc79aa408817] and region [us-east1]....done.     
Services to deploy:

descriptor:                  [/home/student_01_821796f8d08e/appengine-hello/app.yaml]
source:                      [/home/student_01_821796f8d08e/appengine-hello]
target project:              [qwiklabs-gcp-01-dc79aa408817]
target service:              [default]
target version:              [20230421t075656]
target url:                  [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]
target service account:      [App Engine default service account]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Uploading 3 files to Google Cloud Storage
33%
67%
100%
100%
File upload done.
Updating service [default]...done.     
Setting traffic split for service [default]...done.   
Deployed service [default] to [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```



```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```

```python
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ cat main.py
# Copyright 2016 Google Inc.
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

import webapp2


class MainPage(webapp2.RequestHandler):
    def get(self):
        self.response.headers['Content-Type'] = 'text/plain'
        self.response.write('Hello, World!')


app = webapp2.WSGIApplication([
    ('/', MainPage),
], debug=True)
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```

### Introduce an error to break the application

To use the sed stream editor to change the import library to the nonexistent `webapp22`, run the following command:

```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ sed -i -e 's/webapp2/webapp22/' main.py
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```

```python
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ cat main.py
# Copyright 2016 Google Inc.
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

import webapp22


class MainPage(webapp22.RequestHandler):
    def get(self):
        self.response.headers['Content-Type'] = 'text/plain'
        self.response.write('Hello, World!')


app = webapp22.WSGIApplication([
    ('/', MainPage),
], debug=True)
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```

### Re-deploy the application to App Engine

1. To re-deploy the application to App Engine, run the following command:

```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ gcloud app deploy app.yaml --quiet
Services to deploy:

descriptor:                  [/home/student_01_821796f8d08e/appengine-hello/app.yaml]
source:                      [/home/student_01_821796f8d08e/appengine-hello]
target project:              [qwiklabs-gcp-01-dc79aa408817]
target service:              [default]
target version:              [20230421t080127]
target url:                  [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]
target service account:      [App Engine default service account]


Beginning deployment of service [default]...
Uploading 1 file to Google Cloud Storage
100%
100%
File upload done.
Updating service [default]...done.     
Setting traffic split for service [default]...done.     
Stopping version [qwiklabs-gcp-01-dc79aa408817/default/20230421t075656].
Sent request to stop version [qwiklabs-gcp-01-dc79aa408817/default/20230421t075656]. This operation may take some time to complete. If you would like to verify that it succeeded, run:
  $ gcloud app versions describe -s default 20230421t075656
until it shows that the version has stopped.
Deployed service [default] to [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```



```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```



## 2. Explore Cloud Error Reporting

```sh
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ sed -i -e 's/webapp22/webapp2/' main.py
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$ gcloud app deploy app.yaml --quiet
Services to deploy:

descriptor:                  [/home/student_01_821796f8d08e/appengine-hello/app.yaml]
source:                      [/home/student_01_821796f8d08e/appengine-hello]
target project:              [qwiklabs-gcp-01-dc79aa408817]
target service:              [default]
target version:              [20230421t081232]
target url:                  [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]
target service account:      [App Engine default service account]


Beginning deployment of service [default]...
Uploading 0 files to Google Cloud Storage
100%
File upload done.
Updating service [default]...done.     
Setting traffic split for service [default]...done.     
Stopping version [qwiklabs-gcp-01-dc79aa408817/default/20230421t080127].
Sent request to stop version [qwiklabs-gcp-01-dc79aa408817/default/20230421t080127]. This operation may take some time to complete. If you would like to verify that it succeeded, run:
  $ gcloud app versions describe -s default 20230421t080127
until it shows that the version has stopped.
Deployed service [default] to [https://qwiklabs-gcp-01-dc79aa408817.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_821796f8d08e@cloudshell:~/appengine-hello (qwiklabs-gcp-01-dc79aa408817)$
```



## 3. Review

In this lab we deployed an application to App Engine. Then we  introduced a code bug and broke the application. Weused Cloud Error  Reporting to identify the issue, and then analyzed the problem, finding  the root cause using Cloud Debugger. We modified the code to fix the  problem, and then saw the results in Cloud Operations.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-error-reporting-10.png)





