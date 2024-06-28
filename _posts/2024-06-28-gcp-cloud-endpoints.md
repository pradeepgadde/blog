---

layout: single
title:  "Cloud Endpoints: Qwik Start"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Cloud Endpoints: Qwik Start

In this lab you will deploy a sample API with [Google Cloud Endpoints](https://cloud.google.com/endpoints/docs/frameworks/about-cloud-endpoints-frameworks), which are a set of tools for generating APIs from within an App Engine application. The sample code will include:

- A REST API that you can query to find the name of an airport from its three-letter `IATA` code (for example, SFO, JFK, AMS).
- A script that uploads the API configuration to Cloud Endpoints.
- A script that deploys a Google App Engine flexible backend to host the sample API.

After you send some requests to the sample API, you can view the  Cloud Endpoints Activity Graphs and Logs. These are tools that allow you to monitor your APIs and gain insights into their usage.

## Getting the sample code

1. Enter the following command in Cloud Shell to get the sample API and scripts:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-9874a99f51e2.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_0957c56691ca@cloudshell:~ (qwiklabs-gcp-02-9874a99f51e2)$ gsutil cp gs://spls/gsp164/endpoints-quickstart.zip .
unzip endpoints-quickstart.zip
Copying gs://spls/gsp164/endpoints-quickstart.zip...
\ [1 files][  4.7 MiB/  4.7 MiB]                                                
Operation completed over 1 objects/4.7 MiB.                                      
Archive:  endpoints-quickstart.zip
   creating: endpoints-quickstart/
  inflating: endpoints-quickstart/openapi.yaml  
   creating: endpoints-quickstart/app/
  inflating: endpoints-quickstart/app/app_template.yaml  
  inflating: endpoints-quickstart/app/airports.py  
 extracting: endpoints-quickstart/app/requirements.txt  
   creating: endpoints-quickstart/app/third_party/
  inflating: endpoints-quickstart/app/third_party/airports.csv  
  inflating: endpoints-quickstart/app/main.py  
  inflating: endpoints-quickstart/README.md  
  inflating: endpoints-quickstart/travis.yml  
 extracting: endpoints-quickstart/CONTRIBUTING  
  inflating: endpoints-quickstart/travis.sh  
  inflating: endpoints-quickstart/openapi_with_ratelimit.yaml  
   creating: endpoints-quickstart/scripts/
  inflating: endpoints-quickstart/scripts/query_api.sh  
  inflating: endpoints-quickstart/scripts/query_api_with_key.sh  
  inflating: endpoints-quickstart/scripts/deploy_api.sh  
  inflating: endpoints-quickstart/scripts/e2e_test.sh  
  inflating: endpoints-quickstart/scripts/delete_project.sh  
  inflating: endpoints-quickstart/scripts/generate_traffic.sh  
  inflating: endpoints-quickstart/scripts/util.sh  
  inflating: endpoints-quickstart/scripts/generate_traffic_with_key.sh  
  inflating: endpoints-quickstart/scripts/deploy_app.sh  
   creating: endpoints-quickstart/.git/
  inflating: endpoints-quickstart/.git/description  
  inflating: endpoints-quickstart/.git/packed-refs  
   creating: endpoints-quickstart/.git/logs/
   creating: endpoints-quickstart/.git/logs/refs/
   creating: endpoints-quickstart/.git/logs/refs/heads/
  inflating: endpoints-quickstart/.git/logs/refs/heads/master  
   creating: endpoints-quickstart/.git/logs/refs/remotes/
   creating: endpoints-quickstart/.git/logs/refs/remotes/origin/
  inflating: endpoints-quickstart/.git/logs/refs/remotes/origin/HEAD  
  inflating: endpoints-quickstart/.git/logs/HEAD  
   creating: endpoints-quickstart/.git/objects/
   creating: endpoints-quickstart/.git/objects/pack/
  inflating: endpoints-quickstart/.git/objects/pack/pack-cfd102ffdf4e238f0d5f9ad3af1e7a364b8eba52.idx  
  inflating: endpoints-quickstart/.git/objects/pack/pack-cfd102ffdf4e238f0d5f9ad3af1e7a364b8eba52.pack  
   creating: endpoints-quickstart/.git/objects/info/
  inflating: endpoints-quickstart/.git/config  
   creating: endpoints-quickstart/.git/refs/
   creating: endpoints-quickstart/.git/refs/heads/
  inflating: endpoints-quickstart/.git/refs/heads/master  
   creating: endpoints-quickstart/.git/refs/remotes/
   creating: endpoints-quickstart/.git/refs/remotes/origin/
 extracting: endpoints-quickstart/.git/refs/remotes/origin/HEAD  
   creating: endpoints-quickstart/.git/refs/tags/
   creating: endpoints-quickstart/.git/branches/
 extracting: endpoints-quickstart/.git/HEAD  
   creating: endpoints-quickstart/.git/info/
  inflating: endpoints-quickstart/.git/info/exclude  
  inflating: endpoints-quickstart/.git/index  
   creating: endpoints-quickstart/.git/hooks/
  inflating: endpoints-quickstart/.git/hooks/update.sample  
  inflating: endpoints-quickstart/.git/hooks/pre-applypatch.sample  
  inflating: endpoints-quickstart/.git/hooks/pre-push.sample  
  inflating: endpoints-quickstart/.git/hooks/pre-commit.sample  
  inflating: endpoints-quickstart/.git/hooks/commit-msg.sample  
  inflating: endpoints-quickstart/.git/hooks/prepare-commit-msg.sample  
  inflating: endpoints-quickstart/.git/hooks/applypatch-msg.sample  
  inflating: endpoints-quickstart/.git/hooks/post-update.sample  
  inflating: endpoints-quickstart/.git/hooks/fsmonitor-watchman.sample  
  inflating: endpoints-quickstart/.git/hooks/pre-rebase.sample  
  inflating: endpoints-quickstart/.git/hooks/pre-receive.sample  
  inflating: endpoints-quickstart/LICENSE  
student_02_0957c56691ca@cloudshell:~ (qwiklabs-gcp-02-9874a99f51e2)$ cd endpoints-quickstart
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ ls
app  CONTRIBUTING  LICENSE  openapi_with_ratelimit.yaml  openapi.yaml  README.md  scripts  travis.sh  travis.yml
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ 
```



## Deploying the Endpoints configuration

To publish a REST API to Endpoints, an OpenAPI configuration file  that describes the API is required. The lab's sample API comes with a  pre-configured OpenAPI file called `openapi.yaml`.

```yaml
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ cat openapi.yaml 
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
---
swagger: "2.0"
info:
  title: "Airport Codes"
  description: "Get the name of an airport from its three-letter IATA code."
  version: "1.0.0"
# This field will be replaced by the deploy_api.sh script.
host: "YOUR-PROJECT-ID.appspot.com"
schemes:
  - "https"
paths:
  "/airportName":
    get:
      description: "Get the airport name for a given IATA code."
      operationId: "airportName"
      parameters:
        -
          name: iataCode
          in: query
          required: true
          type: string
      responses:
        200:
          description: "Success."
          schema:
            type: string
        400:
          description: "The IATA code is invalid or missing."
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ 
```



Endpoints uses Google `Service Management`, an  infrastructure service of Google Cloud, to create and manage APIs and  services. To use Endpoints to manage an API, you deploy the API's  OpenAPI configuration to Service Management.

To deploy the Endpoints configuration...

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ cat deploy_api.sh 
#!/bin/bash
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -euo pipefail

source util.sh

main() {
  # Get our working project, or exit if it's not set.
  local project_id=$(get_project_id)
  if [[ -z "$project_id" ]]; then
    exit 1
  fi
  local temp_file=$(mktemp)
  export TEMP_FILE="${temp_file}.yaml"
  mv "$temp_file" "$TEMP_FILE"
  # Because the included API is a template, we have to do some string
  # substitution before we can deploy it. Sed does this nicely.
  < "$API_FILE" sed -E "s/YOUR-PROJECT-ID/${project_id}/g" > "$TEMP_FILE"
  echo "Deploying $API_FILE..."
  echo "gcloud endpoints services deploy $API_FILE"
  gcloud endpoints services deploy "$TEMP_FILE"
}

cleanup() {
  rm "$TEMP_FILE"
}

# Defaults.
API_FILE="../openapi.yaml"

if [[ "$#" == 0 ]]; then
  : # Use defaults.
elif [[ "$#" == 1 ]]; then
  API_FILE="$1"
else
  echo "Wrong number of arguments specified."
  echo "Usage: deploy_api.sh [api-file]"
  exit 1
fi

# Cleanup our temporary files even if our deployment fails.
trap cleanup EXIT

main "$@"
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



Cloud Endpoints uses the `host` field in the OpenAPI configuration file to identify the service. The `deploy_api.sh` script sets the ID of your Cloud project as part of the name configured in the `host` field. (When you prepare an OpenAPI configuration file for your own service, you will need to do this manually.)

The script then deploys the OpenAPI configuration to Service Management using the command: `gcloud endpoints services deploy openapi.yaml`

As it is creating and configuring the service, Service Management  outputs some information to the console. You can safely ignore the  warnings about the paths in `openapi.yaml` not requiring an  API key. On successful completion, you see a line like the following  that displays the service configuration ID and the service name:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./deploy_api.sh
Deploying ../openapi.yaml...
gcloud endpoints services deploy ../openapi.yaml
Waiting for async operation operations/services.qwiklabs-gcp-02-9874a99f51e2.appspot.com-0 to complete...
Waiting for async operation operations/serviceConfigs.qwiklabs-gcp-02-9874a99f51e2.appspot.com:55070652-bb1f-4d6e-a517-498e116854f7 to complete...
Operation finished successfully. The following command can describe the Operation details:
 gcloud endpoints operations describe operations/serviceConfigs.qwiklabs-gcp-02-9874a99f51e2.appspot.com:55070652-bb1f-4d6e-a517-498e116854f7

WARNING: tmp.urv2vMst2q.yaml: Operation 'get' in path '/airportName': Operation does not require an API key; callers may invoke the method without specifying an associated API-consuming project. To enable API key all the SecurityRequirement Objects (https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md#security-requirement-object) inside security definition must reference at least one SecurityDefinition of type : 'apiKey'.

Waiting for async operation operations/rollouts.qwiklabs-gcp-02-9874a99f51e2.appspot.com:f36ab273-9ea7-4667-ae1c-ea31862e7232 to complete...
Operation finished successfully. The following command can describe the Operation details:
 gcloud endpoints operations describe operations/rollouts.qwiklabs-gcp-02-9874a99f51e2.appspot.com:f36ab273-9ea7-4667-ae1c-ea31862e7232

Enabling service [qwiklabs-gcp-02-9874a99f51e2.appspot.com] on project [qwiklabs-gcp-02-9874a99f51e2]...
Operation "operations/acat.p2-390054839071-ab717e72-d35c-4a4d-a758-44871b034b27" finished successfully.


Service Configuration [2024-06-28r0] uploaded for service [qwiklabs-gcp-02-9874a99f51e2.appspot.com]

To manage your API, go to: https://console.cloud.google.com/endpoints/api/qwiklabs-gcp-02-9874a99f51e2.appspot.com/overview?project=qwiklabs-gcp-02-9874a99f51e2
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



## Deploying the API backend

So far you have deployed the OpenAPI configuration to Service  Management, but you have not yet deployed the code that will serve the  API backend. The `deploy_app.sh` script included in the lab  sample creates an App Engine flexible environment to host the API  backend, and then the script deploys the API to App Engine.



```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ cat deploy_app.sh 
#!/bin/bash
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -euo pipefail

source util.sh

main() {
  # Get our working project, or exit if it's not set.
  local project_id="$(get_project_id)"
  if [[ -z "$project_id" ]]; then
    exit 1
  fi
  # Try to create an App Engine project in our selected region.
  # If it already exists, return a success ("|| true").
  # -----------------------------------------
  # b/300368738
  # -----------------------------------------
  echo "gcloud app create --region=$REGION"
  gcloud app create --region="$REGION" --quiet &> /dev/null || true
  # -----------------------------------------

  # Prepare the necessary variables for substitution in our app configuration
  # template, and create a temporary file to hold the templatized version.
  local service_name="${project_id}.appspot.com"
  export TEMP_FILE="../app/app.yaml"
  < "$APP" \
    sed -E "s/SERVICE_NAME/${service_name}/g" \
    > "$TEMP_FILE"
  echo "Deploying ${APP}..."
  echo "gcloud -q app deploy $TEMP_FILE"
  gcloud -q app deploy "$TEMP_FILE"
}

cleanup() {
  rm "$TEMP_FILE"
}

# Defaults.
APP="../app/app_template.yaml"
REGION="us-central"

if [[ "$#" == 0 ]]; then
  : # Use defaults.
elif [[ "$#" == 1 ]]; then
  APP="$1"
elif [[ "$#" == 2 ]]; then
  APP="$1"
  REGION="$2"
else
  echo "Wrong number of arguments specified."
  echo "Usage: deploy_app.sh [app-template] [region]"
  exit 1
fi

# Cleanup our temporary files even if our deployment fails.
trap cleanup EXIT

main "$@"
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```

- To deploy the API backend, make sure you are in the `endpoints-quickstart/scripts` directory. Then, run the following script:

The script runs the following command to create an App Engine flexible environment in the  region: `gcloud app create --region="$REGION"`

It takes a couple minutes to create the App Engine flexible backend.

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./deploy_app.sh ../app/app_template.yaml us-east1
gcloud app create --region=us-east1
Deploying ../app/app_template.yaml...
gcloud -q app deploy ../app/app.yaml
Services to deploy:

descriptor:                  [/home/student_02_0957c56691ca/endpoints-quickstart/app/app.yaml]
source:                      [/home/student_02_0957c56691ca/endpoints-quickstart/app]
target project:              [qwiklabs-gcp-02-9874a99f51e2]
target service:              [default]
target version:              [20240628t103234]
target url:                  [https://qwiklabs-gcp-02-9874a99f51e2.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-02-9874a99f51e2@appspot.gserviceaccount.com]


Beginning deployment of service [default]...
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
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...done.                                                                                                                 
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "39a3ea13-1f8f-42f7-8076-89a94d7a4be0"

FETCHSOURCE
BUILD
Starting Step #0 - "fetcher"
Step #0 - "fetcher": Already have image (with digest): gcr.io/cloud-builders/gcs-fetcher
Step #0 - "fetcher": Fetching manifest gs://staging.qwiklabs-gcp-02-9874a99f51e2.appspot.com/ae/94722edb-5f5e-473a-8c8a-c712bdf7ee8b/manifest.json.
Step #0 - "fetcher": Processing 7 files.
Step #0 - "fetcher": ******************************************************
Step #0 - "fetcher": Status:                      SUCCESS
Step #0 - "fetcher": Started:                     2024-06-28T10:32:52Z
Step #0 - "fetcher": Completed:                   2024-06-28T10:32:53Z
Step #0 - "fetcher": Requested workers:    200
Step #0 - "fetcher": Actual workers:         7
Step #0 - "fetcher": Total files:            7
Step #0 - "fetcher": Total retries:          0
Step #0 - "fetcher": GCS timeouts:           0
Step #0 - "fetcher": MiB downloaded:         7.39 MiB
Step #0 - "fetcher": MiB/s throughput:       9.60 MiB/s
Step #0 - "fetcher": Time for manifest:    419.17 ms
Step #0 - "fetcher": Total time:             1.20 s
Step #0 - "fetcher": ******************************************************
Finished Step #0 - "fetcher"
Starting Step #1
Step #1: Pulling image: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa: Pulling from gcp-runtimes/python/gen-dockerfile
Step #1: Digest: sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: Status: Downloaded newer image for gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Finished Step #1
Starting Step #2
Step #2: Pulling image: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd: Pulling from cloud-builders/docker
Step #2: 75f546e73d8b: Pulling fs layer
Step #2: 0f3bb76fc390: Pulling fs layer
Step #2: 3c2cba919283: Pulling fs layer
Step #2: 4d168e97939c: Pulling fs layer
Step #2: 4d168e97939c: Waiting
Step #2: 3c2cba919283: Verifying Checksum
Step #2: 3c2cba919283: Download complete
Step #2: 0f3bb76fc390: Verifying Checksum
Step #2: 0f3bb76fc390: Download complete
Step #2: 75f546e73d8b: Verifying Checksum
Step #2: 75f546e73d8b: Download complete
Step #2: 75f546e73d8b: Pull complete
Step #2: 4d168e97939c: Verifying Checksum
Step #2: 4d168e97939c: Download complete
Step #2: 0f3bb76fc390: Pull complete
Step #2: 3c2cba919283: Pull complete
Step #2: 4d168e97939c: Pull complete
Step #2: Digest: sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Status: Downloaded newer image for gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Sending build context to Docker daemon  7.763MB
Step #2: Step 1/9 : FROM gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6: Pulling from google-appengine/python
Step #2: Digest: sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: Status: Downloaded newer image for gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2:  ce284ab20159
Step #2: Step 2/9 : LABEL python_version=python3.6
Step #2:  Running in 1bd3b9ca64b7
Step #2: Removing intermediate container 1bd3b9ca64b7
Step #2:  4158ed9b3bb7
Step #2: Step 3/9 : RUN virtualenv --no-download /env -p python3.6
Step #2:  Running in db97168b44fd
Step #2: created virtual environment CPython3.6.10.final.0-64 in 1891ms
Step #2:   creator CPython3Posix(dest=/env, clear=False, global=False)
Step #2:   seeder FromAppData(download=False, pip=bundle, wheel=bundle, setuptools=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
Step #2:     added seed packages: pip==20.2.2, setuptools==49.6.0, wheel==0.35.1
Step #2:   activators PythonActivator,FishActivator,XonshActivator,CShellActivator,PowerShellActivator,BashActivator
Step #2: Removing intermediate container db97168b44fd
Step #2:  c303b8c6fad1
Step #2: Step 4/9 : ENV VIRTUAL_ENV /env
Step #2:  Running in 71c8df631b37
Step #2: Removing intermediate container 71c8df631b37
Step #2:  cce3875710d9
Step #2: Step 5/9 : ENV PATH /env/bin:$PATH
Step #2:  Running in a21b02899192
Step #2: Removing intermediate container a21b02899192
Step #2:  b4b57584187a
Step #2: Step 6/9 : ADD requirements.txt /app/
Step #2:  90f9cb027f30
Step #2: Step 7/9 : RUN pip install -r requirements.txt
Step #2:  Running in 46722b2f76be
Step #2: Collecting flask
Step #2:   Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Step #2: Collecting gunicorn
Step #2:   Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
Step #2: Collecting Werkzeug>=2.0
Step #2:   Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Step #2: Collecting Jinja2>=3.0
Step #2:   Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Step #2: Collecting click>=7.1.2
Step #2:   Downloading click-8.0.4-py3-none-any.whl (97 kB)
Step #2: Collecting itsdangerous>=2.0
Step #2:   Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Step #2: Collecting packaging
Step #2:   Downloading packaging-21.3-py3-none-any.whl (40 kB)
Step #2: Collecting importlib-metadata; python_version < "3.8"
Step #2:   Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Step #2: Collecting dataclasses; python_version < "3.7"
Step #2:   Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Step #2: Collecting MarkupSafe>=2.0
Step #2:   Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux2010_x86_64.whl (30 kB)
Step #2: Collecting pyparsing!=3.0.5,>=2.0.2
Step #2:   Downloading pyparsing-3.1.2-py3-none-any.whl (103 kB)
Step #2: Collecting zipp>=0.5
Step #2:   Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Step #2: Collecting typing-extensions>=3.6.4; python_version < "3.8"
Step #2:   Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Step #2: Installing collected packages: dataclasses, Werkzeug, MarkupSafe, Jinja2, zipp, typing-extensions, importlib-metadata, click, itsdangerous, flask, pyparsing, packaging, gunicorn
Step #2: Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 gunicorn-21.2.0 importlib-metadata-4.8.3 itsdangerous-2.0.1 packaging-21.3 pyparsing-3.1.2 typing-extensions-4.1.1 zipp-3.6.0
Step #2: WARNING: You are using pip version 20.2.2; however, version 21.3.1 is available.
Step #2: You should consider upgrading via the '/env/bin/python -m pip install --upgrade pip' command.
Step #2: Removing intermediate container 46722b2f76be
Step #2:  25fb7295c5ad
Step #2: Step 8/9 : ADD . /app/
Step #2:  b4f46fd2ba08
Step #2: Step 9/9 : CMD exec gunicorn -b :$PORT main:app
Step #2:  Running in c8c49231355d
Step #2: Removing intermediate container c8c49231355d
Step #2:  9d7d2b4376bb
Step #2: Successfully built 9d7d2b4376bb
Step #2: Successfully tagged us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t103234:latest
Finished Step #2
PUSH
Pushing us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t103234
The push refers to repository [us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t103234]
36552b524e0b: Preparing
458481a6fcae: Preparing
1f5cc5c9777d: Preparing
d2622708f0ab: Preparing
087d7553d285: Preparing
16919ab89eca: Preparing
74bcef7f7402: Preparing
bc9e931c388e: Preparing
20896f2c3dd8: Preparing
7b80c69caf34: Preparing
3bbec54fac0c: Preparing
4006ffa4c683: Preparing
844d958e8cbe: Preparing
84ff92691f90: Preparing
b49bce339f97: Preparing
dcb7197db903: Preparing
16919ab89eca: Waiting
74bcef7f7402: Waiting
bc9e931c388e: Waiting
20896f2c3dd8: Waiting
7b80c69caf34: Waiting
3bbec54fac0c: Waiting
4006ffa4c683: Waiting
844d958e8cbe: Waiting
84ff92691f90: Waiting
b49bce339f97: Waiting
dcb7197db903: Waiting
087d7553d285: Layer already exists
16919ab89eca: Layer already exists
1f5cc5c9777d: Pushed
74bcef7f7402: Layer already exists
bc9e931c388e: Layer already exists
36552b524e0b: Pushed
20896f2c3dd8: Layer already exists
7b80c69caf34: Layer already exists
3bbec54fac0c: Layer already exists
4006ffa4c683: Layer already exists
844d958e8cbe: Layer already exists
84ff92691f90: Layer already exists
b49bce339f97: Layer already exists
458481a6fcae: Pushed
dcb7197db903: Layer already exists
d2622708f0ab: Pushed
latest: digest: sha256:0789b1dec2f14848cd11935bf216c9700ce32ed6cff5ce9713ffb962f29a66cb size: 3672
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...done.                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Deployed service [default] to [https://qwiklabs-gcp-02-9874a99f51e2.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```

The script goes on to run the `gcloud app deploy` command to deploy the sample API to App Engine.



## Sending requests to the API

1. After deploying the sample API, you can send requests to it by running the following script:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ cat query_api.sh 
#!/bin/bash
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -euo pipefail

source util.sh

main() {
  # Get our working project, or exit if it's not set.
  local project_id=$(get_project_id)
  if [[ -z "$project_id" ]]; then
    exit 1
  fi
  # Because our included app uses query string parameters, we can include
  # them directly in the URL.
  QUERY="curl \"https://${project_id}.appspot.com/airportName?iataCode=${IATA_CODE}\""
  # First, (maybe) print the command so the user can see what's being executed.
  if [[ "$QUIET" == "false" ]]; then
    echo "$QUERY"
  fi
  # Then actually execute it.
  # shellcheck disable=SC2086
  eval $QUERY
  # Our API doesn't print newlines. So we do it ourselves.
  printf '\n'
}

# Defaults.
IATA_CODE="SFO"
QUIET="false"

if [[ "$#" == 0 ]]; then
  : # Use defaults.
elif [[ "$#" == 1 ]]; then
  IATA_CODE="$1"
elif [[ "$#" == 2 ]]; then
  # "Quiet mode" won't print the curl command.
  IATA_CODE="$1"
  QUIET="true"
else
  echo "Wrong number of arguments specified."
  echo "Usage: query_api.sh [iata-code] [quiet-mode]"
  exit 1
fi

main "$@"
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. The script echoes the `curl` command that it uses to send a  request to the API, and then displays the result. You'll see something  like the following in Cloud Shell:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./query_api.sh
curl "https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO"
San Francisco International Airport
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```

The API expects one query parameter, `iataCode`, that is set to a valid IATA airport code such as SEA or JFK.

1. To test, run this example in Cloud Shell:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./query_api.sh EWR
curl "https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=EWR"
Newark Liberty International Airport
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./query_api.sh BLR
curl "https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=BLR"
Kempegowda International Airport
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```

You just deployed and tested an API in Cloud Endpoints!

## Tracking API activity

With APIs deployed with Cloud Endpoints, you can monitor critical  operations metrics in the Cloud Console and gain insight into your users and usage with Cloud Logging:

1. Run this traffic generation script in Cloud Shell to populate the graphs and logs:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ cat generate_traffic.sh 
#!/bin/bash
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -euo pipefail

source util.sh

# Use this to keep track of what HTTP status codes we receive.
declare -A codes

# generate_traffic will print a status update every UPDATE_FREQUENCY messages.
UPDATE_FREQUENCY=25

main() {
  # Get our working project, or exit if it's not set.
  local project_id=$(get_project_id)
  if [[ -z "$project_id" ]]; then
    exit 1
  fi
  export url="https://${project_id}.appspot.com/airportName?iataCode=${IATA_CODE}"
  echo "This command will exit automatically in $TIMEOUT_SECONDS seconds."
  echo "Generating traffic to ${url}..."
  echo "Press Ctrl-C to stop."
  local endtime=$(($(date +%s) + $TIMEOUT_SECONDS))
  local request_count=0
  # Send queries repeatedly until TIMEOUT_SECONDS seconds have elapsed.
  while [[ $(date +%s) -lt $endtime ]]; do
    request_count=$(( request_count + 1))
    if [[ $((request_count % UPDATE_FREQUENCY)) == 0 ]]; then
      echo "Served ${request_count} requests."
    fi
    # Make the HTTP request and save its status in an associative array.
    http_status=$(curl -so /dev/null -w "%{http_code}" "$url")
    if [[ "${!codes[@]}" != *"$http_status"* ]]; then
      codes["$http_status"]="1"
    else
      codes["$http_status"]="$(( ${codes[$http_status]} + 1 ))"
    fi
  done
}

print_status() {
  echo ""
  echo "HTTP status codes received from ${url}:"
  for code in "${!codes[@]}"; do
    echo "${code}: ${codes[$code]}"
  done
}

# Defaults.
IATA_CODE="SFO"
TIMEOUT_SECONDS=$((5 * 60)) # Timeout after 5 minutes.

if [[ "$#" == 0 ]]; then
  : # Use defaults.
elif [[ "$#" == 1 ]]; then
  IATA_CODE="$1"
else
  echo "Wrong number of arguments specified."
  echo "Usage: generate_traffic.sh [iata-code]"
  exit 1
fi

# Print the received codes when we exit.
trap print_status EXIT

main "$@"
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. In the Console, go to **Navigation menu > Endpoints > Services** and click **Airport Codes** service to look at the activity graphs for your service. It may take a  few moments for the requests to be reflected in the graphs. You can do  this while you wait for data to be displayed:

- If the Permissions side panel is not open, click **Show Permissions Panel**. The Permissions panel allows you to control who has access to your API and the level of access.
- Click the **Deployment history** tab. This tab displays a history of your API deployments, including the deployment time and who deployed the change.
- Click the **Overview** tab. Here you'll see the traffic  coming in. After the traffic generation script has been running for a  minute, scroll down to see the three lines on the **Total latency** graph (50th, 95th, and 99th percentiles). This data provides a quick estimate of response times.

1. At the bottom of the Endpoints graphs, under Method, click the **View logs** link for GET/airportName. The Logs Viewer page displays the request logs for the API.
2. Enter **CTRL+C** in Cloud Shell to stop the script.

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./generate_traffic.sh
This command will exit automatically in 300 seconds.
Generating traffic to https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO...
Press Ctrl-C to stop.
Served 25 requests.
Served 50 requests.
Served 75 requests.
^C
HTTP status codes received from https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO:
200: 96

student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



## Add a quota to the API

Cloud Endpoints lets you set quotas so you can control the rate at  which applications can call your API. Quotas can be used to protect your API from excessive usage by a single client.

```yaml
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ cat openapi_with_ratelimit.yaml 
# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
---
swagger: "2.0"
info:
  title: "Airport Codes"
  description: "Get the name of an airport from its three-letter IATA code."
  version: "1.0.0"
security:
  - api_key: []
# This field will be replaced by the deploy_api.sh script.
host: "YOUR-PROJECT-ID.appspot.com"
schemes:
  - "https"
paths:
  "/airportName":
    get:
      description: "Get the airport name for a given IATA code."
      operationId: "airportName"
      x-google-quota:
        metricCosts:
          airport_requests: 1
      parameters:
        -
          name: iataCode
          in: query
          required: true
          type: string
      responses:
        200:
          description: "Success."
          schema:
            type: string
        400:
          description: "The IATA code is invalid or missing."
securityDefinitions:
  # Basic authentication with an API key.
  api_key:
    type: "apiKey"
    name: "key"
    in: "query"

x-google-management:
  metrics:
    - name: airport_requests
      valueType: INT64
      metricKind: DELTA
  quota:
    limits:
      - name: limit-on-airport-requests
        values:
          STANDARD: 5
        unit: "1/min/{project}"
        metric: airport_requests
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. Deploy the Endpoints configuration that has a quota:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./deploy_api.sh ../openapi_with_ratelimit.yaml
Deploying ../openapi_with_ratelimit.yaml...
gcloud endpoints services deploy ../openapi_with_ratelimit.yaml
Waiting for async operation operations/serviceConfigs.qwiklabs-gcp-02-9874a99f51e2.appspot.com:ffdd9ff3-cbef-4af7-b21c-123ac1622afb to complete...
Operation finished successfully. The following command can describe the Operation details:
 gcloud endpoints operations describe operations/serviceConfigs.qwiklabs-gcp-02-9874a99f51e2.appspot.com:ffdd9ff3-cbef-4af7-b21c-123ac1622afb

WARNING: toplevel: Value STANDARD in limit limit-on-airport-requests can't be set below 100.

Waiting for async operation operations/rollouts.qwiklabs-gcp-02-9874a99f51e2.appspot.com:56db6d22-c452-4672-87c4-a8746f533693 to complete...
Operation finished successfully. The following command can describe the Operation details:
 gcloud endpoints operations describe operations/rollouts.qwiklabs-gcp-02-9874a99f51e2.appspot.com:56db6d22-c452-4672-87c4-a8746f533693

Service Configuration [2024-06-28r1] uploaded for service [qwiklabs-gcp-02-9874a99f51e2.appspot.com]

To manage your API, go to: https://console.cloud.google.com/endpoints/api/qwiklabs-gcp-02-9874a99f51e2.appspot.com/overview?project=qwiklabs-gcp-02-9874a99f51e2
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. Redeploy your app to use the new Endpoints configuration (this may take a few minutes):

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./deploy_app.sh ../app/app_template.yaml us-east1
gcloud app create --region=us-east1
Deploying ../app/app_template.yaml...
gcloud -q app deploy ../app/app.yaml
Services to deploy:

descriptor:                  [/home/student_02_0957c56691ca/endpoints-quickstart/app/app.yaml]
source:                      [/home/student_02_0957c56691ca/endpoints-quickstart/app]
target project:              [qwiklabs-gcp-02-9874a99f51e2]
target service:              [default]
target version:              [20240628t105042]
target url:                  [https://qwiklabs-gcp-02-9874a99f51e2.ue.r.appspot.com]
target service account:      [qwiklabs-gcp-02-9874a99f51e2@appspot.gserviceaccount.com]


Beginning deployment of service [default]...
Uploading 0 files to Google Cloud Storage
100%
File upload done.
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...done.                                                                                                                 
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "02bea963-2098-419a-972c-04a3d2609b7a"

FETCHSOURCE
BUILD
Starting Step #0 - "fetcher"
Step #0 - "fetcher": Already have image (with digest): gcr.io/cloud-builders/gcs-fetcher
Step #0 - "fetcher": Fetching manifest gs://staging.qwiklabs-gcp-02-9874a99f51e2.appspot.com/ae/cb1a1df2-63a4-4a91-a135-d11347813b18/manifest.json.
Step #0 - "fetcher": Processing 7 files.
Step #0 - "fetcher": ******************************************************
Step #0 - "fetcher": Status:                      SUCCESS
Step #0 - "fetcher": Started:                     2024-06-28T10:50:52Z
Step #0 - "fetcher": Completed:                   2024-06-28T10:50:52Z
Step #0 - "fetcher": Requested workers:    200
Step #0 - "fetcher": Actual workers:         7
Step #0 - "fetcher": Total files:            7
Step #0 - "fetcher": Total retries:          0
Step #0 - "fetcher": GCS timeouts:           0
Step #0 - "fetcher": MiB downloaded:         7.39 MiB
Step #0 - "fetcher": MiB/s throughput:      17.92 MiB/s
Step #0 - "fetcher": Time for manifest:    155.19 ms
Step #0 - "fetcher": Total time:             0.57 s
Step #0 - "fetcher": ******************************************************
Finished Step #0 - "fetcher"
Starting Step #1
Step #1: Pulling image: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa: Pulling from gcp-runtimes/python/gen-dockerfile
Step #1: Digest: sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: Status: Downloaded newer image for gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Step #1: gcr.io/gcp-runtimes/python/gen-dockerfile@sha256:ac444fc620f70ff80c19cde48d18242dbed63056e434f0039bf939433e7464aa
Finished Step #1
Starting Step #2
Step #2: Pulling image: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd: Pulling from cloud-builders/docker
Step #2: 75f546e73d8b: Pulling fs layer
Step #2: 0f3bb76fc390: Pulling fs layer
Step #2: 3c2cba919283: Pulling fs layer
Step #2: 4d168e97939c: Pulling fs layer
Step #2: 4d168e97939c: Waiting
Step #2: 3c2cba919283: Verifying Checksum
Step #2: 3c2cba919283: Download complete
Step #2: 0f3bb76fc390: Verifying Checksum
Step #2: 0f3bb76fc390: Download complete
Step #2: 75f546e73d8b: Verifying Checksum
Step #2: 75f546e73d8b: Download complete
Step #2: 75f546e73d8b: Pull complete
Step #2: 0f3bb76fc390: Pull complete
Step #2: 4d168e97939c: Verifying Checksum
Step #2: 4d168e97939c: Download complete
Step #2: 3c2cba919283: Pull complete
Step #2: 4d168e97939c: Pull complete
Step #2: Digest: sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Status: Downloaded newer image for gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: gcr.io/cloud-builders/docker@sha256:08c5443ff4f8ba85c2114576bb9167c4de0bf658818aea536d3456e8d0e134cd
Step #2: Sending build context to Docker daemon  7.763MB
Step #2: Step 1/9 : FROM gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6: Pulling from google-appengine/python
Step #2: Digest: sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2: Status: Downloaded newer image for gcr.io/google-appengine/python@sha256:c6480acd38ca4605e0b83f5196ab6fe8a8b59a0288a7b8216c42dbc45b5de8f6
Step #2:  ce284ab20159
Step #2: Step 2/9 : LABEL python_version=python3.6
Step #2:  Running in 5989b6be161a
Step #2: Removing intermediate container 5989b6be161a
Step #2:  3f3b751b6ab5
Step #2: Step 3/9 : RUN virtualenv --no-download /env -p python3.6
Step #2:  Running in 41646c4acefc
Step #2: created virtual environment CPython3.6.10.final.0-64 in 1274ms
Step #2:   creator CPython3Posix(dest=/env, clear=False, global=False)
Step #2:   seeder FromAppData(download=False, pip=bundle, wheel=bundle, setuptools=bundle, via=copy, app_data_dir=/root/.local/share/virtualenv)
Step #2:     added seed packages: pip==20.2.2, setuptools==49.6.0, wheel==0.35.1
Step #2:   activators PythonActivator,FishActivator,XonshActivator,CShellActivator,PowerShellActivator,BashActivator
Step #2: Removing intermediate container 41646c4acefc
Step #2:  2ea781f2eb78
Step #2: Step 4/9 : ENV VIRTUAL_ENV /env
Step #2:  Running in 73e58cda7461
Step #2: Removing intermediate container 73e58cda7461
Step #2:  e64f84b80ca6
Step #2: Step 5/9 : ENV PATH /env/bin:$PATH
Step #2:  Running in 7b9db7683917
Step #2: Removing intermediate container 7b9db7683917
Step #2:  e594d6ce2c59
Step #2: Step 6/9 : ADD requirements.txt /app/
Step #2:  7088cd81dd9d
Step #2: Step 7/9 : RUN pip install -r requirements.txt
Step #2:  Running in 7737468cbbb5
Step #2: Collecting flask
Step #2:   Downloading Flask-2.0.3-py3-none-any.whl (95 kB)
Step #2: Collecting gunicorn
Step #2:   Downloading gunicorn-21.2.0-py3-none-any.whl (80 kB)
Step #2: Collecting Jinja2>=3.0
Step #2:   Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Step #2: Collecting itsdangerous>=2.0
Step #2:   Downloading itsdangerous-2.0.1-py3-none-any.whl (18 kB)
Step #2: Collecting Werkzeug>=2.0
Step #2:   Downloading Werkzeug-2.0.3-py3-none-any.whl (289 kB)
Step #2: Collecting click>=7.1.2
Step #2:   Downloading click-8.0.4-py3-none-any.whl (97 kB)
Step #2: Collecting importlib-metadata; python_version < "3.8"
Step #2:   Downloading importlib_metadata-4.8.3-py3-none-any.whl (17 kB)
Step #2: Collecting packaging
Step #2:   Downloading packaging-21.3-py3-none-any.whl (40 kB)
Step #2: Collecting MarkupSafe>=2.0
Step #2:   Downloading MarkupSafe-2.0.1-cp36-cp36m-manylinux2010_x86_64.whl (30 kB)
Step #2: Collecting dataclasses; python_version < "3.7"
Step #2:   Downloading dataclasses-0.8-py3-none-any.whl (19 kB)
Step #2: Collecting typing-extensions>=3.6.4; python_version < "3.8"
Step #2:   Downloading typing_extensions-4.1.1-py3-none-any.whl (26 kB)
Step #2: Collecting zipp>=0.5
Step #2:   Downloading zipp-3.6.0-py3-none-any.whl (5.3 kB)
Step #2: Collecting pyparsing!=3.0.5,>=2.0.2
Step #2:   Downloading pyparsing-3.1.2-py3-none-any.whl (103 kB)
Step #2: Installing collected packages: MarkupSafe, Jinja2, itsdangerous, dataclasses, Werkzeug, typing-extensions, zipp, importlib-metadata, click, flask, pyparsing, packaging, gunicorn
Step #2: Successfully installed Jinja2-3.0.3 MarkupSafe-2.0.1 Werkzeug-2.0.3 click-8.0.4 dataclasses-0.8 flask-2.0.3 gunicorn-21.2.0 importlib-metadata-4.8.3 itsdangerous-2.0.1 packaging-21.3 pyparsing-3.1.2 typing-extensions-4.1.1 zipp-3.6.0
Step #2: WARNING: You are using pip version 20.2.2; however, version 21.3.1 is available.
Step #2: You should consider upgrading via the '/env/bin/python -m pip install --upgrade pip' command.
Step #2: Removing intermediate container 7737468cbbb5
Step #2:  6c53eb0f66e5
Step #2: Step 8/9 : ADD . /app/
Step #2:  6dbcbd42bcef
Step #2: Step 9/9 : CMD exec gunicorn -b :$PORT main:app
Step #2:  Running in 9ffd235849f6
Step #2: Removing intermediate container 9ffd235849f6
Step #2:  ac394ca60924
Step #2: Successfully built ac394ca60924
Step #2: Successfully tagged us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t105042:latest
Finished Step #2
PUSH
Pushing us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t105042
The push refers to repository [us.gcr.io/qwiklabs-gcp-02-9874a99f51e2/appengine/default.20240628t105042]
7f221ced0340: Preparing
5a2d2db08c44: Preparing
b0903c2a4893: Preparing
85d78272be21: Preparing
087d7553d285: Preparing
16919ab89eca: Preparing
74bcef7f7402: Preparing
bc9e931c388e: Preparing
20896f2c3dd8: Preparing
7b80c69caf34: Preparing
3bbec54fac0c: Preparing
4006ffa4c683: Preparing
844d958e8cbe: Preparing
84ff92691f90: Preparing
b49bce339f97: Preparing
dcb7197db903: Preparing
16919ab89eca: Waiting
74bcef7f7402: Waiting
bc9e931c388e: Waiting
20896f2c3dd8: Waiting
7b80c69caf34: Waiting
3bbec54fac0c: Waiting
4006ffa4c683: Waiting
844d958e8cbe: Waiting
84ff92691f90: Waiting
b49bce339f97: Waiting
dcb7197db903: Waiting
087d7553d285: Layer already exists
16919ab89eca: Layer already exists
74bcef7f7402: Layer already exists
b0903c2a4893: Pushed
bc9e931c388e: Layer already exists
20896f2c3dd8: Layer already exists
7b80c69caf34: Layer already exists
7f221ced0340: Pushed
3bbec54fac0c: Layer already exists
4006ffa4c683: Layer already exists
84ff92691f90: Layer already exists
b49bce339f97: Layer already exists
844d958e8cbe: Layer already exists
dcb7197db903: Layer already exists
5a2d2db08c44: Pushed
85d78272be21: Pushed
latest: digest: sha256:d64b1d7e15e04f2d3a37f4d826322308463ef0aabaa52ef5f731ddd63dfcc209 size: 3672
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
WARNING: python 3.7 and earlier versions will reach end of support on 2024-07-10 for App Engine flexible environment. After 2024-07-10, you cannot deploy new or re-deploy existing applications that use runtimes after their end of support date. We recommend that you migrate to the latest supported version of python.

Updating service [default] (this may take several minutes)...done.                                                                                                                 
Setting traffic split for service [default]...done.                                                                                                                                
Stopping version [qwiklabs-gcp-02-9874a99f51e2/default/20240628t103234].
Sent request to stop version [qwiklabs-gcp-02-9874a99f51e2/default/20240628t103234]. This operation may take some time to complete. If you would like to verify that it succeeded, run:
  $ gcloud app versions describe -s default 20240628t103234
until it shows that the version has stopped.
Deployed service [default] to [https://qwiklabs-gcp-02-9874a99f51e2.ue.r.appspot.com]

You can stream logs from the command line by running:
  $ gcloud app logs tail -s default

To view your application in the web browser run:
  $ gcloud app browse
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. In the Console, navigate to **Navigation menu** > **APIs & Services** > **Credentials**.
2. Click **Create credentials** and choose **API key**. A new API key is displayed on the screen.
3. Click the **Copy to clipboard** icon to copy it to your clipboard.
4. In Cloud Shell, type the following. Replace **YOUR-API-KEY** with the API key you just created:
5. Send your API a request using the API key variable you just created:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ export API_KEY=AIzaSyAAC4Ga-Nd2-8fjRheI10ZC-RZmk2BV7I8
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./query_api_with_key.sh $API_KEY
curl -H 'x-api-key: AIzaSyAAC4Ga-Nd2-8fjRheI10ZC-RZmk2BV7I8' "https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO"
San Francisco International Airport
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. The API now has a limit of 5 requests per second. Run the following  command to send traffic to the API and trigger the quota limit:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./generate_traffic_with_key.sh $API_KEY
This command will exit automatically in 300 seconds.
Generating traffic to https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO&key=AIzaSyAAC4Ga-Nd2-8fjRheI10ZC-RZmk2BV7I8...
Press Ctrl-C to stop.
^C
HTTP status codes received from https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO&key=AIzaSyAAC4Ga-Nd2-8fjRheI10ZC-RZmk2BV7I8:
429: 2
200: 9

student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



1. After running the script for 5-10 seconds, enter CTRL+C in Cloud Shell to stop the script.
2. Send another authenticated request to the API:

```sh
student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ ./query_api_with_key.sh $API_KEY
curl -H 'x-api-key: AIzaSyAAC4Ga-Nd2-8fjRheI10ZC-RZmk2BV7I8' "https://qwiklabs-gcp-02-9874a99f51e2.appspot.com/airportName?iataCode=SFO"
{
 "code": 8,
 "message": "Quota exceeded for quota metric 'airport_requests' and limit 'limit-on-airport-requests' of service 'qwiklabs-gcp-02-9874a99f51e2.appspot.com' for consumer 'project_number:390054839071'.",
 "details": [
  {
   "@type": "type.googleapis.com/google.rpc.DebugInfo",
   "stackEntries": [],
   "detail": "internal"
  }
 ]
}

student_02_0957c56691ca@cloudshell:~/endpoints-quickstart/scripts (qwiklabs-gcp-02-9874a99f51e2)$ 
```



Congratulations! You've successfully rate-limited your API. You can also set varying limits on different API methods, create multiple kinds of  quotas, and keep track of which consumers use which APIs
