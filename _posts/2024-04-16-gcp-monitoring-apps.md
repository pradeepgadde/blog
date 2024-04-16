---
layout: single
title:  "Monitoring Apps in GCP"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## Monitoring Apps in GCP

Deploy an application to Google Cloud and then use the tools provided by Google Cloud to monitor it. 
- Cloud Logging, 
- Trace,
-  Profiler, and
-  Dashboards 
-  Create uptime checks and alerting policies.

> **Note:** Profiler allows you to monitor the resources your applications use.
>
> **Note:** This code simply turns Profiler on. Once on, Profiler starts reporting application metrics to Google Cloud.

```py
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ cat main.py 
from flask import Flask, render_template, request
import googlecloudprofiler
app = Flask(__name__)


@app.route("/")
def main():
    model = {"title": "Hello GCP."}
    return render_template('index.html', model=model)
try:
    googlecloudprofiler.start(verbose=3)
except (ValueError, NotImplementedError) as exc:
    print(exc)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080, debug=True, threaded=True)student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```

```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ cat requirements.txt 
Flask==2.0.3
itsdangerous==2.0.1
Jinja2==3.0.3
werkzeug==2.2.2
google-cloud-profiler==3.0.6
protobuf==3.20.1student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```



Profiler has to be enabled in the project. In Cloud Shell, enter the following command:

```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ gcloud services enable cloudprofiler.googleapis.com
Operation "operations/acat.p2-436242463909-59953dd5-af9d-49c3-87c5-56aeef67548c" finished successfully.
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```

To test the program, enter the following command to build a Docker container of the image:

```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ docker build -t test-python .
[+] Building 47.4s (10/10) FINISHED                                                                                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 217B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/python:3.7                                                                                                                  0.6s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [1/5] FROM docker.io/library/python:3.7@sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0                                                           24.4s
 => => resolve docker.io/library/python:3.7@sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0                                                            0.0s
 => => sha256:eedf63967cdb57d8214db38ce21f105003ed4e4d0358f02bedc057341bcf92a0 1.86kB / 1.86kB                                                                                 0.0s
 => => sha256:167b8a53ca4504bc6aa3182e336fa96f4ef76875d158c1933d3e2fa19c57e0c3 49.56MB / 49.56MB                                                                               0.8s
 => => sha256:2011a37d2a08fe83dd9ff923e0f83bfd7290152e2e6afe359bde1453170d9bdc 2.01kB / 2.01kB                                                                                 0.0s
 => => sha256:16d93ae3411be3db255b6b52fdfc155a0dff0f697c2e4e3d862caf8d978830b2 8.13kB / 8.13kB                                                                                 0.0s
 => => sha256:b47a222d28fa95680198398973d0a29b82a968f03e7ef361cc8ded562e4d84a3 24.03MB / 24.03MB                                                                               0.6s
 => => sha256:debce5f9f3a9709885f7f2ad3cf41f036a3b57b406b27ba3a883928315787042 64.11MB / 64.11MB                                                                               1.1s
 => => sha256:1d7ca7cd2e066ae77ac6284a9d027f72a31a02a18bfc2a249ef2e7b01074338b 211.04MB / 211.04MB                                                                             3.7s
 => => sha256:ff3119008f58beef8f336fa833707b0fe914db94ca6b7bb55abe3e1bf2b1ad56 6.39MB / 6.39MB                                                                                 1.4s
 => => sha256:c2423a76a32b7ffb2ee7bb6d1e0c74bb1811237eddcb3200594daf7a52d4f378 14.70MB / 14.70MB                                                                               1.5s
 => => extracting sha256:167b8a53ca4504bc6aa3182e336fa96f4ef76875d158c1933d3e2fa19c57e0c3                                                                                      3.9s
 => => sha256:e1c98ca4926a91839805ce76d76a70225e303007331ee60f45dfabbbf55fd8c8 244B / 244B                                                                                     1.5s
 => => sha256:3b62c8e1d79b6554a8bffcf196ff5dd822858c179f1f8dc6f0c74a288859a6fb 2.85MB / 2.85MB                                                                                 1.7s
 => => extracting sha256:b47a222d28fa95680198398973d0a29b82a968f03e7ef361cc8ded562e4d84a3                                                                                      0.9s
 => => extracting sha256:debce5f9f3a9709885f7f2ad3cf41f036a3b57b406b27ba3a883928315787042                                                                                      4.2s
 => => extracting sha256:1d7ca7cd2e066ae77ac6284a9d027f72a31a02a18bfc2a249ef2e7b01074338b                                                                                     11.5s
 => => extracting sha256:ff3119008f58beef8f336fa833707b0fe914db94ca6b7bb55abe3e1bf2b1ad56                                                                                      0.4s
 => => extracting sha256:c2423a76a32b7ffb2ee7bb6d1e0c74bb1811237eddcb3200594daf7a52d4f378                                                                                      0.9s
 => => extracting sha256:e1c98ca4926a91839805ce76d76a70225e303007331ee60f45dfabbbf55fd8c8                                                                                      0.0s
 => => extracting sha256:3b62c8e1d79b6554a8bffcf196ff5dd822858c179f1f8dc6f0c74a288859a6fb                                                                                      0.4s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 1.48kB                                                                                                                                            0.0s
 => [2/5] WORKDIR /app                                                                                                                                                         1.7s
 => [3/5] COPY . .                                                                                                                                                             0.0s
 => [4/5] RUN pip install gunicorn                                                                                                                                             3.9s
 => [5/5] RUN pip install -r requirements.txt                                                                                                                                 15.4s 
 => exporting to image                                                                                                                                                         1.1s 
 => => exporting layers                                                                                                                                                        1.1s 
 => => writing image sha256:c3f1fbff6b5b05ba0e2711e56d8a9050bcdd32ee9b14c698c240aac46416a04a                                                                                   0.0s 
 => => naming to docker.io/library/test-python                                                                                                                                 0.0s 
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$                                
```



```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ docker run --rm -p 8080:8080 test-python
[2024-04-16 15:46:59 +0000] [1] [INFO] Starting gunicorn 21.2.0
[2024-04-16 15:46:59 +0000] [1] [INFO] Listening at: http://0.0.0.0:8080 (1)
[2024-04-16 15:46:59 +0000] [1] [INFO] Using worker: gthread
[2024-04-16 15:46:59 +0000] [9] [INFO] Booting worker with pid: 9
WARNING:google.auth._default:No project ID could be determined. Consider running `gcloud config set project` or setting the GOOGLE_CLOUD_PROJECT environment variable


^C[2024-04-16 15:47:23 +0000] [1] [INFO] Handling signal: int
Unable to determine the project ID from the environment. project ID mush be provided if running outside of GCP.
[2024-04-16 15:47:23 +0000] [9] [INFO] Worker exiting (pid: 9)
[2024-04-16 15:47:24 +0000] [1] [INFO] Shutting down: Master
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```



## Deploy an application to App Engine and examine the Cloud logs

Now you will deploy the program to App Engine and use Google Cloud tools to monitor it.

On the **File** menu, click **New File**, and then name the file **app.yaml**.

```yaml
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ cat app.yaml 
runtime: python39student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```

In a project, an App Engine application has to be created. This is done just once using the `gcloud app create` command and specifying the region where you want the app to be created. Enter the following command:

```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ gcloud app create --region=us-east4
You are creating an app for project [qwiklabs-gcp-03-57e29b122480].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Creating App Engine application in project [qwiklabs-gcp-03-57e29b122480] and region [us-east4]....done.                                                                           
Success! The app is now created. Please use `gcloud app deploy` to deploy your first app.
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```



```sh
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ gcloud app deploy --version=one --quiet
Services to deploy:

descriptor:                  [/home/student_03_b88891f8e948/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp/app.yaml]
source:                      [/home/student_03_b88891f8e948/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp]
target project:              [qwiklabs-gcp-03-57e29b122480]
target service:              [default]
target version:              [one]
target url:                  [https://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com]
target service account:      [qwiklabs-gcp-03-57e29b122480@appspot.gserviceaccount.com]


Beginning deployment of service [default]...
Created .gcloudignore file. See `gcloud topic gcloudignore` for details.
Uploading 7 files to Google Cloud Storage
14%
29%
43%
57%
71%
86%
100%
100%
File upload done.
Updating service [default]...done.                                                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```



```html
student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ curl https://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com/
<!doctype html>
<html lang="en">
<head>
    <title>Hello GCP.</title>
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css">

 
</head>
<body>
    <div class="container">

        
<div class="jumbotron">
    <div class="container">
        <h1>Hello GCP.</h1>
    </div>
</div>


        <footer></footer>
    </div>
</body>
</html>student_03_b88891f8e948@cloudshell:~/gcp-logging/training-data-analyst/courses/design-process/deploying-apps-to-gcp (qwiklabs-gcp-03-57e29b122480)$ 
```

```json
[
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
      "status": {},
      "authenticationInfo": {
        "principalEmail": "student-03-b88891f8e948@qwiklabs.net"
      },
      "requestMetadata": {
        "callerIp": "34.124.235.65",
        "requestAttributes": {
          "time": "2024-04-16T15:50:35.737256Z",
          "auth": {}
        },
        "destinationAttributes": {}
      },
      "serviceName": "appengine.googleapis.com",
      "methodName": "google.appengine.v1.Versions.CreateVersion",
      "authorizationInfo": [
        {
          "resource": "apps/qwiklabs-gcp-03-57e29b122480/services/default/versions/one",
          "permission": "appengine.versions.create",
          "granted": true,
          "resourceAttributes": {},
          "permissionType": "ADMIN_WRITE"
        }
      ],
      "resourceName": "apps/qwiklabs-gcp-03-57e29b122480/services/default/versions/one",
      "serviceData": {
        "@type": "type.googleapis.com/google.appengine.v1.AuditData",
        "createVersion": {
          "request": {
            "parent": "apps/qwiklabs-gcp-03-57e29b122480/services/default",
            "version": {
              "id": "one",
              "runtime": "python39",
              "entrypoint": {
                "shell": ""
              }
            }
          }
        }
      },
      "resourceLocation": {
        "currentLocations": [
          "us-east4"
        ]
      }
    },
    "insertId": "-6xner6d12iq",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:50:35.559401Z",
    "severity": "NOTICE",
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/cloudaudit.googleapis.com%2Factivity",
    "operation": {
      "id": "ad8bdc29-b8d5-45d8-98c1-fcd8a8b01df7",
      "producer": "appengine.googleapis.com",
      "first": true
    },
    "receiveTimestamp": "2024-04-16T15:50:35.961165649Z"
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
      "authenticationInfo": {
        "principalEmail": "student-03-b88891f8e948@qwiklabs.net"
      },
      "requestMetadata": {
        "requestAttributes": {},
        "destinationAttributes": {}
      },
      "serviceName": "appengine.googleapis.com",
      "methodName": "google.appengine.v1.Versions.CreateVersion",
      "resourceName": "apps/qwiklabs-gcp-03-57e29b122480/services/default/versions/one",
      "resourceLocation": {
        "currentLocations": [
          "us-east4"
        ]
      }
    },
    "insertId": "-oyos0qcmdm",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:52:04.365253Z",
    "severity": "NOTICE",
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/cloudaudit.googleapis.com%2Factivity",
    "operation": {
      "id": "ad8bdc29-b8d5-45d8-98c1-fcd8a8b01df7",
      "producer": "appengine.googleapis.com",
      "last": true
    },
    "receiveTimestamp": "2024-04-16T15:52:05.176559114Z"
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.appengine.logging.v1.RequestLog",
      "appId": "d~qwiklabs-gcp-03-57e29b122480",
      "versionId": "one",
      "requestId": "661e9edd00ff00ffbd8fe770f95a0001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "ip": "122.171.22.62",
      "startTime": "2024-04-16T15:53:02.387974Z",
      "endTime": "2024-04-16T15:53:04.608886Z",
      "latency": "2.220912s",
      "megaCycles": "6377",
      "method": "GET",
      "resource": "/",
      "httpVersion": "HTTP/1.1",
      "status": 200,
      "responseSize": "506",
      "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:124.0) Gecko/20100101 Firefox/124.0",
      "urlMapEntry": "auto",
      "host": "qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com",
      "wasLoadingRequest": true,
      "instanceIndex": -1,
      "finished": true,
      "instanceId": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a",
      "line": [
        {
          "time": "2024-04-16T15:53:04.608669Z",
          "severity": "INFO",
          "logMessage": "This request caused a new process to be started for your application, and thus caused your application code to be loaded for the first time. This request may thus take longer and use more CPU than a typical request for your application."
        }
      ],
      "appEngineRelease": "1.9.71",
      "traceId": "5a394110b7cf7d519e782771f2476e4a",
      "first": true,
      "traceSampled": true,
      "spanId": "2703176114609722884"
    },
    "insertId": "661e9ee000094d9f68b9c208",
    "httpRequest": {
      "status": 200
    },
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:02.387974Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/appengine.googleapis.com%2Frequest_log",
    "operation": {
      "id": "661e9edd00ff00ffbd8fe770f95a0001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "producer": "appengine.googleapis.com/request_id",
      "first": true,
      "last": true
    },
    "trace": "projects/qwiklabs-gcp-03-57e29b122480/traces/5a394110b7cf7d519e782771f2476e4a",
    "receiveTimestamp": "2024-04-16T15:53:04.614347811Z",
    "spanId": "2703176114609722884",
    "traceSampled": true
  },
  {
    "textPayload": "[pid1] started [session:N60B72P]",
    "insertId": "661e9ede000a5a766e2dcda8",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:02.678518Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:02.681806105Z"
  },
  {
    "textPayload": "[pid1-app] Using app start info from /srv/.googleconfig/app_start.json: &main.appStart{Entrypoint:struct { Type string \"json:\\\"type\\\"\"; UnparsedValue string \"json:\\\"unparsed_value\\\"\"; Command string \"json:\\\"command\\\"\"; WorkDir string \"json:\\\"workdir\\\"\" }{Type:\"Default\", UnparsedValue:\"\", Command:\"/serve\", WorkDir:\"\"}, EntrypointFromAppYAML:\"\", EntrypointContents:\"\", Runtime:\"python39\"} [session:N60B72P]",
    "insertId": "661e9ede000a651c85ce8a22",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:02.681244Z",
    "severity": "DEBUG",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[pid1] Starting processes [app nginx] [session:N60B72P]",
    "insertId": "661e9ede000a6677ff722db5",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:02.681591Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[pid1-app] app has no prerequisites, starting immediately [session:N60B72P]",
    "insertId": "661e9ede000a69df3b050a56",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:02.682463Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[pid1-nginx] nginx waiting for any of 4 prerequisite(s): [portbind:tcp:127.0.0.1:8081 portbind:tcp:localhost:8080 portbind:tcp:localhost:8081 portbind:unix:/tmp/google-config/app.sock] [session:N60B72P]",
    "insertId": "661e9ede000a6bd12ec6cc94",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:02.682961Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[pid1-app] Starting app (pid 11): /serve [session:N60B72P]",
    "insertId": "661e9ede000aa3cb8082fe5e",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "zone": "us-east4-3",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:02.697291Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[serve] Serve started.",
    "insertId": "661e9ede000aea1f79f19513",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "module_id": "default",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:02.715295Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[serve] Args: {runtimeLanguage:python runtimeName:python39 memoryMB:384 positional:[]}",
    "insertId": "661e9ede000aebafa6f64056",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:02.715695Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[serve] Running /bin/sh -c exec gunicorn main:app --workers 4 -c /config/gunicorn.py",
    "insertId": "661e9ede000aebb6829a9b29",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "version_id": "one",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:02.715702Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.015158Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [11] [INFO] Starting gunicorn 21.2.0",
    "insertId": "661e9edf00036842e14a25f0",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "version_id": "one",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:03.223298Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.226270785Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [11] [INFO] Listening at: http://0.0.0.0:8081 (11)",
    "insertId": "661e9edf00036d40d9663ad7",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:03.224576Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [11] [INFO] Using worker: gthread",
    "insertId": "661e9edf00036d7e46eb2d2a",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.224638Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "[pid1-nginx] Successfully connected to localhost:8081 after 546.044133ms [session:N60B72P]",
    "insertId": "661e9edf00038114274cb237",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:03.229652Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.347867892Z"
  },
  {
    "textPayload": "[pid1-nginx] Successfully connected to 127.0.0.1:8081 after 545.505977ms [session:N60B72P]",
    "insertId": "661e9edf0003812423f2a632",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "module_id": "default",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.229668Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.347867892Z"
  },
  {
    "textPayload": "[pid1-nginx] Creating config at /tmp/nginxconf-60496491/nginx.conf [session:N60B72P]",
    "insertId": "661e9edf0003847dc776cd0f",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "version_id": "one",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.230525Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.347867892Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [19] [INFO] Booting worker with pid: 19",
    "insertId": "661e9edf0003b32ff2ff2bdc",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.242479Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "[pid1-nginx] Starting nginx (pid 20): /usr/sbin/nginx -c /tmp/nginxconf-60496491/nginx.conf [session:N60B72P]",
    "insertId": "661e9edf0003b7dffef1c2ae",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.243679Z",
    "severity": "INFO",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/%2Fvar%2Flog%2Fgoogle_init.log",
    "receiveTimestamp": "2024-04-16T15:53:03.347867892Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [22] [INFO] Booting worker with pid: 22",
    "insertId": "661e9edf0004a5594093e2b2",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.304473Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [23] [INFO] Booting worker with pid: 23",
    "insertId": "661e9edf00061cf280442374",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:03.400626Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "[2024-04-16 15:53:03 +0000] [24] [INFO] Booting worker with pid: 24",
    "insertId": "661e9edf0006cca46cc6f5c9",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:03.445604Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:03.563709722Z"
  },
  {
    "textPayload": "INFO:googlecloudprofiler:Google Cloud Profiler Python agent version: 3.0.6",
    "insertId": "661e9ee00008ea87a758c366",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:04.584327Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "INFO:googlecloudprofiler:Google Cloud Profiler Python agent version: 3.0.6",
    "insertId": "661e9ee00008ea936b9848b1",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:04.584339Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "INFO:googlecloudprofiler:Google Cloud Profiler Python agent version: 3.0.6",
    "insertId": "661e9ee00008ec7741277698",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "version_id": "one",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:04.584823Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Profiler has started",
    "insertId": "661e9ee00008ecfbb2350fe4",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "module_id": "default",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:04.584955Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Profiler has started",
    "insertId": "661e9ee00008ecfff00e79de",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one",
        "zone": "us-east4-3",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:04.584959Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Profiler has started",
    "insertId": "661e9ee00008ee1cc5b9ceaa",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:04.585244Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "INFO:googlecloudprofiler:Google Cloud Profiler Python agent version: 3.0.6",
    "insertId": "661e9ee00008ff278e0850c8",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:04.589607Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Profiler has started",
    "insertId": "661e9ee000090253a1d3e4e1",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:04.590419Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9ee0000bf4f1c38c2cbb",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:04.783601Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9ee0000c371ccdb5c8a7",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:04.800540Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9ee0000c3729944d7323",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:04.800553Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9ee0000cbaecbd119bc0",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:04.834284Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:53:04.892323203Z"
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.appengine.logging.v1.RequestLog",
      "appId": "d~qwiklabs-gcp-03-57e29b122480",
      "versionId": "one",
      "requestId": "661e9ee000ff0e4abd028dfd9b0001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "ip": "122.171.22.62",
      "startTime": "2024-04-16T15:53:05.060251Z",
      "endTime": "2024-04-16T15:53:05.065740Z",
      "latency": "0.005489s",
      "megaCycles": "520",
      "method": "GET",
      "resource": "/favicon.ico",
      "httpVersion": "HTTP/1.1",
      "status": 404,
      "responseSize": "370",
      "referrer": "https://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com/",
      "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:124.0) Gecko/20100101 Firefox/124.0",
      "urlMapEntry": "auto",
      "host": "qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com",
      "instanceIndex": -1,
      "finished": true,
      "instanceId": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a",
      "appEngineRelease": "1.9.71",
      "traceId": "6355340815f739bb0f6cdc8fc086cc7f",
      "first": true,
      "spanId": "15735226852420926203"
    },
    "insertId": "661e9ee10001011bd94c0b11",
    "httpRequest": {
      "status": 404
    },
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:53:05.060251Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/appengine.googleapis.com%2Frequest_log",
    "operation": {
      "id": "661e9ee000ff0e4abd028dfd9b0001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "producer": "appengine.googleapis.com/request_id",
      "first": true,
      "last": true
    },
    "trace": "projects/qwiklabs-gcp-03-57e29b122480/traces/6355340815f739bb0f6cdc8fc086cc7f",
    "receiveTimestamp": "2024-04-16T15:53:05.278557168Z",
    "spanId": "15735226852420926203"
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.appengine.logging.v1.RequestLog",
      "appId": "d~qwiklabs-gcp-03-57e29b122480",
      "versionId": "one",
      "requestId": "661e9ef700ff04209a52d6bf670001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "ip": "34.124.235.65",
      "startTime": "2024-04-16T15:53:28.432055Z",
      "endTime": "2024-04-16T15:53:28.446256Z",
      "latency": "0.014201s",
      "megaCycles": "214",
      "method": "GET",
      "resource": "/",
      "httpVersion": "HTTP/1.1",
      "status": 200,
      "responseSize": "573",
      "userAgent": "curl/7.74.0",
      "urlMapEntry": "auto",
      "host": "qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com",
      "instanceIndex": -1,
      "finished": true,
      "instanceId": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a",
      "appEngineRelease": "1.9.71",
      "traceId": "a5047ca8d2ff79762ca516531ec530ae",
      "first": true,
      "traceSampled": true,
      "spanId": "10506101927102067846"
    },
    "insertId": "661e9ef80006cf83a95726e1",
    "httpRequest": {
      "status": 200
    },
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:53:28.432055Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/appengine.googleapis.com%2Frequest_log",
    "operation": {
      "id": "661e9ef700ff04209a52d6bf670001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "producer": "appengine.googleapis.com/request_id",
      "first": true,
      "last": true
    },
    "trace": "projects/qwiklabs-gcp-03-57e29b122480/traces/a5047ca8d2ff79762ca516531ec530ae",
    "receiveTimestamp": "2024-04-16T15:53:28.450584262Z",
    "spanId": "10506101927102067846",
    "traceSampled": true
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.appengine.logging.v1.RequestLog",
      "appId": "d~qwiklabs-gcp-03-57e29b122480",
      "versionId": "one",
      "requestId": "661e9f0d00ff087aa3a4162b070001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "ip": "122.171.22.62",
      "startTime": "2024-04-16T15:53:49.679896Z",
      "endTime": "2024-04-16T15:53:49.685372Z",
      "latency": "0.005476s",
      "megaCycles": "161",
      "method": "GET",
      "resource": "/",
      "httpVersion": "HTTP/1.1",
      "status": 200,
      "responseSize": "449",
      "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:124.0) Gecko/20100101 Firefox/124.0",
      "urlMapEntry": "auto",
      "host": "qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com",
      "instanceIndex": -1,
      "finished": true,
      "instanceId": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a",
      "appEngineRelease": "1.9.71",
      "traceId": "8f12f22660444d9fa417beb32405fc5e",
      "first": true,
      "traceSampled": true,
      "spanId": "134974917338682405"
    },
    "insertId": "661e9f0d000a75b39788d31b",
    "httpRequest": {
      "status": 200
    },
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:53:49.679896Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/appengine.googleapis.com%2Frequest_log",
    "operation": {
      "id": "661e9f0d00ff087aa3a4162b070001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "producer": "appengine.googleapis.com/request_id",
      "first": true,
      "last": true
    },
    "trace": "projects/qwiklabs-gcp-03-57e29b122480/traces/8f12f22660444d9fa417beb32405fc5e",
    "receiveTimestamp": "2024-04-16T15:53:49.688244541Z",
    "spanId": "134974917338682405",
    "traceSampled": true
  },
  {
    "protoPayload": {
      "@type": "type.googleapis.com/google.appengine.logging.v1.RequestLog",
      "appId": "d~qwiklabs-gcp-03-57e29b122480",
      "versionId": "one",
      "requestId": "661e9f0e00ff0be21481a704b90001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "ip": "122.171.22.62",
      "startTime": "2024-04-16T15:53:50.902778Z",
      "endTime": "2024-04-16T15:53:50.907785Z",
      "latency": "0.005007s",
      "megaCycles": "16",
      "method": "GET",
      "resource": "/",
      "httpVersion": "HTTP/1.1",
      "status": 200,
      "responseSize": "449",
      "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:124.0) Gecko/20100101 Firefox/124.0",
      "urlMapEntry": "auto",
      "host": "qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com",
      "instanceIndex": -1,
      "finished": true,
      "instanceId": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a",
      "appEngineRelease": "1.9.71",
      "traceId": "12988bc4e4418d1efe0ac83c608b5c75",
      "first": true,
      "spanId": "11434164468470765071"
    },
    "insertId": "661e9f0e000dda691e1b4370",
    "httpRequest": {
      "status": 200
    },
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:53:50.902778Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/appengine.googleapis.com%2Frequest_log",
    "operation": {
      "id": "661e9f0e00ff0be21481a704b90001647e7177696b6c6162732d6763702d30332d35376532396231323234383000016f6e65000100",
      "producer": "appengine.googleapis.com/request_id",
      "first": true,
      "last": true
    },
    "trace": "projects/qwiklabs-gcp-03-57e29b122480/traces/12988bc4e4418d1efe0ac83c608b5c75",
    "receiveTimestamp": "2024-04-16T15:53:51.020637697Z",
    "spanId": "11434164468470765071"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Successfully created a WALL profile",
    "insertId": "661e9f18000c5a1df8c4c633",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:54:00.809501Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:00.811912283Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Successfully created a CPU profile",
    "insertId": "661e9f1c000626fa14140c68",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:54:04.403194Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:04.480470103Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to upload profile",
    "insertId": "661e9f22000c64cc3a7369cf",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:54:10.812236Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:11.135140399Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9f230003bcaa457df6f2",
    "resource": {
      "type": "gae_app",
      "labels": {
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3",
        "module_id": "default",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:54:11.244906Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:11.471598082Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to upload profile",
    "insertId": "661e9f260007c98d1557beb1",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:54:14.510349Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:14.801459301Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9f26000dd47ee540eea7",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "version_id": "one"
      }
    },
    "timestamp": "2024-04-16T15:54:14.906366Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:15.136645062Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Successfully created a WALL profile",
    "insertId": "661e9f480008531d15c51542",
    "resource": {
      "type": "gae_app",
      "labels": {
        "module_id": "default",
        "version_id": "one",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "zone": "us-east4-3"
      }
    },
    "timestamp": "2024-04-16T15:54:48.545565Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:48.550752473Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to upload profile",
    "insertId": "661e9f5200085cff02a70951",
    "resource": {
      "type": "gae_app",
      "labels": {
        "version_id": "one",
        "zone": "us-east4-3",
        "project_id": "qwiklabs-gcp-03-57e29b122480",
        "module_id": "default"
      }
    },
    "timestamp": "2024-04-16T15:54:58.548095Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:58.872460235Z"
  },
  {
    "textPayload": "DEBUG:googlecloudprofiler.client:Starting to create profile",
    "insertId": "661e9f52000d627e23ec8881",
    "resource": {
      "type": "gae_app",
      "labels": {
        "zone": "us-east4-3",
        "version_id": "one",
        "module_id": "default",
        "project_id": "qwiklabs-gcp-03-57e29b122480"
      }
    },
    "timestamp": "2024-04-16T15:54:58.877182Z",
    "labels": {
      "clone_id": "00a22404dc9c5c0e1c2eb0a65f35d7b249dfe0fdcccde96f1a8b890aa264fc1cbe9b09fd400f6cb016a92340ddd77974ccb20b94c448a47053cce44ade801a"
    },
    "logName": "projects/qwiklabs-gcp-03-57e29b122480/logs/stderr",
    "receiveTimestamp": "2024-04-16T15:54:59.207465698Z"
  }
]
```

## View Profiler information

In the Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Profiler**.

**Note:** The gray bar at the top represents the total  amount of CPU time used by the program. The bars below represent the  amount of CPU time used by the program's functions relative to the  total. At this point, there is no traffic, so the chart is not very  interesting. Throw some load at the application.



```sh
student-03-b88891f8e948@instance-20240416-155756:~$ ab -n 1000 -c 10 https://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com/

{snip}
Completed 1000 requests
Finished 1000 requests


Server Software:        Google
Server Hostname:        qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.3,TLS_AES_256_GCM_SHA384,256,256
Server Temp Key:        X25519 253 bits
TLS Server Name:        qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com

Document Path:          /
Document Length:        0 bytes

Concurrency Level:      10
Time taken for tests:   10.984 seconds
Complete requests:      1000
Failed requests:        1000
   (Connect: 0, Receive: 0, Length: 1000, Exceptions: 0)
Total transferred:      635008 bytes
HTML transferred:       413000 bytes
Requests per second:    91.04 [#/sec] (mean)
Time per request:       109.838 [ms] (mean)
Time per request:       10.984 [ms] (mean, across all concurrent requests)
Transfer rate:          56.46 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        3    5   1.7      4      15
Processing:    22  104  29.3     98     208
Waiting:       22  103  29.3     97     207
Total:         26  109  29.2    103     212

Percentage of the requests served within a certain time (ms)
  50%    103
  66%    107
  75%    112
  80%    116
  90%    155
  95%    177
  98%    192
  99%    198
 100%    212 (longest request)
student-03-b88891f8e948@instance-20240416-155756:~$
```

 Each bar represents a function. The width of the bars represents how much CPU time each function consumed.

The Profiler is a way developers can track down parts of a program that are consuming too many resources.

## Explore Cloud Trace

1. Every request to your application is added to the **Trace** list. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Trace**.

The overview screen shows recent requests and allows you to create  reports to analyze traffic. Because your program is new and has only one page, it's not very interesting, but in a real app there would be lots  of useful information.

1. Click **Trace Explorer**.

This shows a history of requests and their latency. Again, it's not  very exciting because the application hasn't been running for very long. The chart in the upper-left plots requests and how long they took. The  table to the right shows a list of requests. If you select a request,  more detail will be displayed at the bottom of the screen.



## Monitor resources using Dashboards

Cloud Monitoring analyzes the resources used in your projects and  generates some default dashboards for you.

## Create uptime checks and alerts

​          ![error icon](https://ci3.googleusercontent.com/meips/ADKq_NYUObvm0FtynsZdLxh0nvZIjvj1zfw8M8zdgv_hIfgj3MCfUm3J_mQqgCHcz4wnbGFbC9_OL33PINqaA984aQL3RqghE3oHqhLnqU0Gizro=s0-d-e1-ft#https://www.gstatic.com/stackdriver/notification/error.png)          Alert firing                  No severity      

### Condition type Uptime Monitoring fired    

| **Start time**           April 16, 2024 at 4:17PM UTC (less than 1 sec ago) |      | **Policy**           [           Uptime Check Alert](https://console.cloud.google.com/monitoring/alerting/policies/5544587932660822857?project=qwiklabs-gcp-03-57e29b122480) |
| ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| **Project**           [           qwiklabs-gcp-03-57e29b122480](https://console.cloud.google.com/?project=qwiklabs-gcp-03-57e29b122480) |      | **Condition**           Uptime Health Check on App Engine Uptime Check |

​        metric : monitoring.googleapis.com/uptime_check/check_passed      

​      host : [qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com](http://qwiklabs-gcp-03-57e29b122480.uk.r.appspot.com)    

​      project_id : qwiklabs-gcp-03-57e29b122480    