---

layout: single
title:  "Web Security Scanner: Qwik Start"
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
# Web Security Scanner: Qwik Start

The Web Security Scanner, one of [Security Command Center's](https://cloud.google.com/security-command-center) built-in services, identifies security vulnerabilities in your Google  App Engine, Google Kubernetes Engine (GKE), and Compute Engine web  applications. It crawls your application, following all links within the scope of your starting URLs, and attempts to exercise as many user  inputs and event handlers as possible.

The scanner is designed to complement your existing secure design and development processes. To avoid distracting developers with false  positives, the scanner errs on the side of under reporting and will not  display low confidence alerts. It does not replace a manual security  review, and it does not guarantee that your application is free from  security flaws.

## Before you begin, you need an app to scan

In this lab, you will deploy a sample Hello World application to run Security Scanner on.

1. Run the following command in Cloud Shell to clone the [Hello World sample app repository](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/main/appengine/standard_python3/hello_world):

The package `itsdangerous==2.0.1` is added in *requirements.txt* file to safely pass data to untrusted environments and get it back safe.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-43a9448d7501.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-04-43a9448d7501)$ gsutil -m cp -r gs://spls/gsp067/python-docs-samples .
Copying gs://spls/gsp067/python-docs-samples/appengine/standard_python3/hello_world/app.yaml...
Copying gs://spls/gsp067/python-docs-samples/appengine/standard_python3/hello_world/main.py...
Copying gs://spls/gsp067/python-docs-samples/appengine/standard_python3/hello_world/requirements.txt...
Copying gs://spls/gsp067/python-docs-samples/appengine/standard_python3/hello_world/main_test.py...
Copying gs://spls/gsp067/python-docs-samples/appengine/standard_python3/hello_world/requirements-test.txt...
student_01_932d053b64d1@cloudshell:~ (qwiklabs-gcp-04-43a9448d7501)$ cd python-docs-samples/appengine/standard_python3/hello_world
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ sed -i "s/python37/python39/g" app.yaml
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ vi requirements.txt
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ cat requirements.txt 


Flask==1.1.2
itsdangerous==2.0.1
Jinja2==3.0.3
werkzeug==2.0.1
Flask==1.1.2
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ 
```



## Test app

1. From within the `hello_world` directory where the app's [app.yaml](https://cloud.google.com/appengine/docs/standard/python/config/appref) configuration file is located, start the local development server with the following command:

```sh
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ cat app.yaml 
runtime: python39
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ dev_appserver.py app.yaml
/usr/lib/google-cloud-sdk/platform/google_appengine/google/protobuf/internal/api_implementation.py:100: UserWarning: Selected implementation upb is not available. Falling back to the python implementation.
  warnings.warn('Selected implementation upb is not available. '
INFO     2024-06-28 07:11:22,631 <string>:316] Skipping SDK update check.
WARNING  2024-06-28 07:11:22,631 <string>:325] The default encoding of your local Python interpreter is set to 'utf-8' while App Engine's production environment uses 'ascii'; as a result your code may behave differently when deployed.
WARNING  2024-06-28 07:11:22,673 simple_search_stub.py:1206] Could not read search indexes from /tmp/appengine.None.student_01_932d053b64d1/search_indexes
INFO     2024-06-28 07:11:22,676 <string>:391] Starting API server at: http://localhost:42833
INFO     2024-06-28 07:11:22,680 instance_factory.py:127] Detected python version "b'Python 3.10.12\n'" for runtime "python39" at "/usr/bin/python".
INFO     2024-06-28 07:11:27,047 instance_factory.py:280] Using pip to install dependency libraries; pip stdout is redirected to /tmp/tmpavwde2yl
INFO     2024-06-28 07:11:27,048 instance_factory.py:310] Running /tmp/tmp3205epht/bin/pip install --upgrade pip
                                                                                            
INFO     2024-06-28 07:11:30,060 instance_factory.py:310] Running /tmp/tmp3205epht/bin/pip install -r /tmp/tmpby96j1fi
                                                                  
INFO     2024-06-28 07:11:32,601 dispatcher.py:267] Starting module "default" running at: http://localhost:8080
INFO     2024-06-28 07:11:32,603 admin_server.py:67] Starting admin server at: http://localhost:8000
INFO     2024-06-28 07:11:33,603 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-04-43a9448d7501 in environment variables
[2024-06-28 07:11:33 +0000] [2078] [INFO] Starting gunicorn 22.0.0
[2024-06-28 07:11:33 +0000] [2078] [INFO] Listening at: http://0.0.0.0:41295 (2078)
[2024-06-28 07:11:33 +0000] [2078] [INFO] Using worker: sync
[2024-06-28 07:11:33 +0000] [2080] [INFO] Booting worker with pid: 2080
INFO     2024-06-28 07:11:34,610 instance.py:293] Instance PID: 2078
INFO     2024-06-28 07:11:34,655 module.py:830] default: "GET /?authuser=1&redirectedPreviously=true HTTP/1.1" 200 12
INFO     2024-06-28 07:11:34,712 module.py:413] [default] Detected file changes:
  /home/student_01_932d053b64d1/python-docs-samples/appengine/standard_python3/hello_world/__pycache__
INFO     2024-06-28 07:11:34,947 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-04-43a9448d7501 in environment variables
[2024-06-28 07:11:35 +0000] [2082] [INFO] Starting gunicorn 22.0.0
[2024-06-28 07:11:35 +0000] [2082] [INFO] Listening at: http://0.0.0.0:43433 (2082)
[2024-06-28 07:11:35 +0000] [2082] [INFO] Using worker: sync
[2024-06-28 07:11:35 +0000] [2084] [INFO] Booting worker with pid: 2084
INFO     2024-06-28 07:11:35,953 instance.py:293] Instance PID: 2082
INFO     2024-06-28 07:11:35,956 module.py:830] default: "GET /favicon.ico HTTP/1.1" 404 232
INFO     2024-06-28 07:11:39,713 instance.py:555] Detected GOOGLE_CLOUD_PROJECT=qwiklabs-gcp-04-43a9448d7501 in environment variables
[2024-06-28 07:11:39 +0000] [2085] [INFO] Starting gunicorn 22.0.0
[2024-06-28 07:11:39 +0000] [2085] [INFO] Listening at: http://0.0.0.0:40159 (2085)
[2024-06-28 07:11:39 +0000] [2085] [INFO] Using worker: sync
[2024-06-28 07:11:39 +0000] [2087] [INFO] Booting worker with pid: 2087
INFO     2024-06-28 07:11:40,719 instance.py:293] Instance PID: 2085
^CINFO     2024-06-28 07:11:45,911 shutdown.py:48] Shutting down.
[2024-06-28 07:11:45 +0000] [2085] [INFO] Handling signal: int
[2024-06-28 07:11:45 +0000] [2082] [INFO] Handling signal: int
[2024-06-28 07:11:46 +0000] [2080] [INFO] Worker exiting (pid: 2080)
[2024-06-28 07:11:46 +0000] [2084] [INFO] Worker exiting (pid: 2084)
[2024-06-28 07:11:46 +0000] [2085] [INFO] Shutting down: Master
[2024-06-28 07:11:46 +0000] [2082] [INFO] Shutting down: Master
INFO     2024-06-28 07:11:46,620 stub_util.py:321] Applying all pending transactions and saving the datastore
INFO     2024-06-28 07:11:46,620 stub_util.py:324] Saving search indexes
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ ls
app.yaml  main.py  main_test.py  __pycache__  requirements-test.txt  requirements.txt
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ 
```

The local development server is now running and listening for requests on port 8080. Click on the **web preview** button in Cloud Shell, and select **Preview on port 8080** to see it:

Press **Ctrl+c** to stop the local app and return to the command line.

## Deploy app

In this lab use  as the App Engine region.

1. Deploy your app to App Engine by running the following command from within the root directory of your application (`hello_world`):

```sh
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ gcloud app deploy
You are creating an app for project [qwiklabs-gcp-04-43a9448d7501].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Please choose the region where you want your App Engine application located:

 [1] europe-west   (supports standard and flexible and search_api)
 [2] us-central    (supports standard and flexible and search_api)
 [3] us-east1      (supports standard and flexible and search_api)
 [4] cancel
Please enter your numeric choice:  3

Creating App Engine application in project [qwiklabs-gcp-04-43a9448d7501] and region [us-east1]....done.                                                                           
Services to deploy:

descriptor:                  [/home/student_01_932d053b64d1/python-docs-samples/appengine/standard_python3/hello_world/app.yaml]
source:                      [/home/student_01_932d053b64d1/python-docs-samples/appengine/standard_python3/hello_world]
target project:              [qwiklabs-gcp-04-43a9448d7501]
target service:              [default]
target version:              [20240628t071308]
target url:                  [https://qwiklabs-gcp-04-43a9448d7501.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-04-43a9448d7501@appspot.gserviceaccount.com]


Do you want to continue (Y/n)?  y

Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 5 files to Google Cloud Storage
20%
40%
60%
80%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-04-43a9448d7501.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ 
```



## View app

- To launch the app in your browser, run the following command:

```sh
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ gcloud app browse
Did not detect your browser. Go to this link to view your app:
https://qwiklabs-gcp-04-43a9448d7501.ue.r.appspot.com
student_01_932d053b64d1@cloudshell:~/python-docs-samples/appengine/standard_python3/hello_world (qwiklabs-gcp-04-43a9448d7501)$ 
```

There will be a link in Cloud Shell that you can use, or view the app at `http://[YOUR_PROJECT_ID].uc.r.appspot.com`. This is the URL you'll scan for vulnerabilities and it will be added to your scan parameters in the next step.

## Run the scan

The scan does not run immediately, but is queued for later execution; it can take hours before the scan executes, depending on current load.  For more information about these form settings, refer to  [Using Web Security Scanner](https://cloud.google.com/security-command-center/docs/how-to-use-web-security-scanner).

1. Go to **Navigation menu** > **APIs & Services** > **Library**.

2. In Search for APIs & Services type **Web Security Scanner**.

3. Select the **Web Security Scanner API** then Click **Enable** to enable the API.

4. From the **Navigation menu** select **Security** > **Web Security Scanner**.

5. Click **New Scan**.

6. Under `Starting URL 1`, enter the URL of the application you want to scan.

7. Click **Save** to create the scan.

8. Click **Run** to start scanning:

9. The scan will be queued, and you can watch the status bar progress as it scans. The scan overview page displays a results section when the scan  completes. The following image shows example scan results when no  vulnerabilities are detected:

   It will take 4-5 minutes for the scan to complete. Try refreshing the page if you aren't seeing any updates.

   ```sh
    No vulnerabilities found.
   
   Cloud Web Security Scanner is continuously updated, so check back and re-run this scan to look for new vulnerabilities. 
   ```

   

   Nice job! You just completed a scan using Web Security Scanner. You will see a warning to let you know that only scanning 1 URL isn't ideal.  This lab is just to demonstrate a simple example. Your production  environment will have plenty of URLs to scan.
