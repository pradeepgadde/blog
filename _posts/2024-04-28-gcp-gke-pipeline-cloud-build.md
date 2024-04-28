---

layout: single
title:  "Google Kubernetes Engine Pipeline using Cloud Build"
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

# Google Kubernetes Engine Pipeline using Cloud Build

Create a CI/CD pipeline that automatically builds a container image from committed code, stores the image in Artifact Registry, updates a  Kubernetes manifest in a Git repository, and deploys the application to  Google Kubernetes Engine using that manifest.

create 2 Git repositories:

- app repository: contains the source code of the application itself
- env repository: contains the manifests for the Kubernetes Deployment

When you push a change to the **app repository**, the Cloud Build pipeline runs tests, builds a container image, and pushes it to  Artifact Registry. After pushing the image, Cloud Build updates the  Deployment manifest and pushes it to the **env repository**. This triggers another Cloud Build pipeline that applies the manifest to the GKE cluster and, if successful, stores the manifest in another  branch of the **env repository**.



The app and env repositories are kept separate because they have different lifecycles and uses. The main users of the **app repository** are actual humans and this repository is dedicated to a specific application. The main users of the **env repository** are automated systems (such as Cloud Build), and this repository might be shared by several applications. The **env repository** can have several branches that each map to a specific environment (you  only use production in this lab) and reference a specific container  image, whereas the **app repository** does not.

When you finish this lab, you have a system where you can easily:

- Distinguish between failed and successful deployments by looking at the Cloud Build history.
- Access the manifest currently used by looking at the production branch of the **env repository**.
- Rollback to any previous version by re-executing the corresponding Cloud Build build.



- Create Kubernetes Engine clusters
- Create Cloud Source Repositories
- Trigger Cloud Build from Cloud Source Repositories
- Automate tests and publish a deployable container image via Cloud Build
- Manage resources deployed in a Kubernetes Engine cluster via Cloud Build

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-c78491702415.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud config list project
[core]
project = qwiklabs-gcp-01-c78491702415

Your active configuration is: [cloudshell-22042]
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-01-3c1bb184aaad@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
export REGION=us-east4
gcloud config set compute/region $REGION
Your active configuration is: [cloudshell-22042]
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud services enable container.googleapis.com \
    cloudbuild.googleapis.com \
    sourcerepo.googleapis.com \
    containeranalysis.googleapis.com
Operation "operations/acat.p2-359830206908-88f33205-b028-4c0f-9a2f-ddd5ccceb31a" finished successfully.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud artifacts repositories create my-repository \
  --repository-format=docker \
  --location=$REGION
Create request issued for: [my-repository]
Waiting for operation [projects/qwiklabs-gcp-01-c78491702415/locations/us-east4/operations/f32967e1-a8bf-406f-8263-491ce0470dd0] to complete..
.done.                                                                                                                                        
Created repository [my-repository].
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$   gcloud container clusters create hello-cloudbuild --num-nodes 1 --region $REGION
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster hello-cloudbuild in us-east4... Cluster is being health-checked (master is healthy)...done.                                  
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-c78491702415/zones/us-east4/clusters/hello-cloudbuild].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east4/hello-cloudbuild?project=qwiklabs-gcp-01-c78491702415
kubeconfig entry generated for hello-cloudbuild.
NAME: hello-cloudbuild
LOCATION: us-east4
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.145.241.254
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ git config --global user.email "pradeep@example.com"  
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ git config --global user.name "Pradeep Gadde"
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ 
```

In this task, you create the two Git repositories (**hello-cloudbuild-app** and **hello-cloudbuild-env**) and initialize **hello-cloudbuild-app** with some sample code.

The code you just cloned contains a simple "Hello World" application.

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud source repos create hello-cloudbuild-app
Created [hello-cloudbuild-app].
WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud source repos create hello-cloudbuild-env
Created [hello-cloudbuild-env].
WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ cd ~
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ git clone https://github.com/GoogleCloudPlatform/gke-gitops-tutorial-cloudbuild hello-cloudbuild-app
Cloning into 'hello-cloudbuild-app'...
remote: Enumerating objects: 28, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 28 (delta 3), reused 7 (delta 0), pack-reused 12
Receiving objects: 100% (28/28), 16.90 KiB | 1.69 MiB/s, done.
Resolving deltas: 100% (6/6), done.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ export REGION=us-east4
sed -i "s/us-central1/$REGION/g" cloudbuild.yaml
sed -i "s/us-central1/$REGION/g" cloudbuild-delivery.yaml
sed -i "s/us-central1/$REGION/g" cloudbuild-trigger-cd.yaml
sed -i "s/us-central1/$REGION/g" kubernetes.yaml.tpl
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ PROJECT_ID=$(gcloud config get-value project)
Your active configuration is: [cloudshell-22042]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git remote add google "https://source.developers.google.com/p/${PROJECT_ID}/r/hello-cloudbuild-app"
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

```py
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ ls
app.py                    cloudbuild-trigger-cd.yaml  CONTRIBUTING.md  kubernetes.yaml.tpl  README.md
cloudbuild-delivery.yaml  cloudbuild.yaml             Dockerfile       LICENSE              test_app.py
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat app.py 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START hello-app]
from flask import Flask
app = Flask('hello-cloudbuild')

@app.route('/')
def hello():
  return "Hello World!\n"

if __name__ == '__main__':
  app.run(host = '0.0.0.0', port = 8080)
# [END hello-app]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```



```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat Dockerfile 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START dockerfile]
FROM python:3.7-slim
RUN pip install flask
WORKDIR /app
COPY app.py /app/app.py
ENTRYPOINT ["python"]
CMD ["/app/app.py"]
# [END dockerfile]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

## Create a container image with Cloud Build

The code you cloned already contains the following Dockerfile. With this Dockerfile, you can create a container image with Cloud Build and store it in Artifact Registry.

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ COMMIT_ID="$(git rev-parse --short=7 HEAD)"
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ gcloud builds submit --tag="${REGION}-docker.pkg.dev/${PROJECT_ID}/my-repository/hello-cloudbuild:${COMMIT_ID}" .
Creating temporary archive of 11 file(s) totalling 23.7 KiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-c78491702415_cloudbuild/source/1714311544.498301-c321f645e2854cb993675e3523337faa.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-c78491702415/locations/global/builds/79e5f9f3-921b-42d0-b10f-182ec1c09ae4].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/79e5f9f3-921b-42d0-b10f-182ec1c09ae4?project=359830206908 ].
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------- REMOTE BUILD OUTPUT -------------------------------------------------------------
starting build "79e5f9f3-921b-42d0-b10f-182ec1c09ae4"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-c78491702415_cloudbuild/source/1714311544.498301-c321f645e2854cb993675e3523337faa.tgz#1714311546870198
Copying gs://qwiklabs-gcp-01-c78491702415_cloudbuild/source/1714311544.498301-c321f645e2854cb993675e3523337faa.tgz#1714311546870198...
/ [1 files][  7.7 KiB/  7.7 KiB]                                                
Operation completed over 1 objects/7.7 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  35.33kB
Step 1/6 : FROM python:3.7-slim
3.7-slim: Pulling from library/python
a803e7c4b030: Pulling fs layer
bf3336e84c8e: Pulling fs layer
8973eb85275f: Pulling fs layer
f9afc3cc0135: Pulling fs layer
39312d8b4ab7: Pulling fs layer
f9afc3cc0135: Waiting
39312d8b4ab7: Waiting
bf3336e84c8e: Verifying Checksum
bf3336e84c8e: Download complete
8973eb85275f: Verifying Checksum
8973eb85275f: Download complete
a803e7c4b030: Verifying Checksum
a803e7c4b030: Download complete
39312d8b4ab7: Verifying Checksum
39312d8b4ab7: Download complete
f9afc3cc0135: Verifying Checksum
f9afc3cc0135: Download complete
a803e7c4b030: Pull complete
bf3336e84c8e: Pull complete
8973eb85275f: Pull complete
f9afc3cc0135: Pull complete
39312d8b4ab7: Pull complete
Digest: sha256:b53f496ca43e5af6994f8e316cf03af31050bf7944e0e4a308ad86c001cf028b
Status: Downloaded newer image for python:3.7-slim
 a255ffcb469f
Step 2/6 : RUN pip install flask
 Running in 210e58bf4b4b
Collecting flask
  Downloading Flask-2.2.5-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 kB 3.8 MB/s eta 0:00:00
Collecting click>=8.0
  Downloading click-8.1.7-py3-none-any.whl (97 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 97.9/97.9 kB 11.3 MB/s eta 0:00:00
Collecting importlib-metadata>=3.6.0
  Downloading importlib_metadata-6.7.0-py3-none-any.whl (22 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.1.3-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.2/133.2 kB 13.4 MB/s eta 0:00:00
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 kB 15.2 MB/s eta 0:00:00
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.7.1-py3-none-any.whl (33 kB)
Collecting zipp>=0.5
  Downloading zipp-3.15.0-py3-none-any.whl (6.8 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.5-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Installing collected packages: zipp, typing-extensions, MarkupSafe, itsdangerous, Werkzeug, Jinja2, importlib-metadata, click, flask
Successfully installed Jinja2-3.1.3 MarkupSafe-2.1.5 Werkzeug-2.2.3 click-8.1.7 flask-2.2.5 importlib-metadata-6.7.0 itsdangerous-2.1.2 typing-extensions-4.7.1 zipp-3.15.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip is available: 23.0.1 -> 24.0
[notice] To update, run: pip install --upgrade pip
Removing intermediate container 210e58bf4b4b
 1ee15db17b49
Step 3/6 : WORKDIR /app
 Running in 4c2744b94833
Removing intermediate container 4c2744b94833
 8b20bc8f4c1f
Step 4/6 : COPY app.py /app/app.py
 178d793ac0fc
Step 5/6 : ENTRYPOINT ["python"]
 Running in fc79cf6f2641
Removing intermediate container fc79cf6f2641
 24ab8c26c056
Step 6/6 : CMD ["/app/app.py"]
 Running in e2241055ca3a
Removing intermediate container e2241055ca3a
 a1073c99be2a
Successfully built a1073c99be2a
Successfully tagged us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:d4080a7
PUSH
Pushing us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:d4080a7
The push refers to repository [us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild]
3166d3a8aea1: Preparing
b5c68e519e4b: Preparing
e7743b0d8089: Preparing
b8594deafbe5: Preparing
8a55150afecc: Preparing
ad34ffec41dd: Preparing
f19cb1e4112d: Preparing
d310e774110a: Preparing
ad34ffec41dd: Waiting
f19cb1e4112d: Waiting
d310e774110a: Waiting
3166d3a8aea1: Pushed
b5c68e519e4b: Pushed
8a55150afecc: Pushed
e7743b0d8089: Pushed
b8594deafbe5: Pushed
f19cb1e4112d: Pushed
ad34ffec41dd: Pushed
d310e774110a: Pushed
d4080a7: digest: sha256:29ca28d4eb5b826a2dffcb259e549ebd2ee3bda383ccdd75fa1c587ec3154a0e size: 1995
DONE
-----------------------------------------------------------------------------------------------------------------------------------------------
ID: 79e5f9f3-921b-42d0-b10f-182ec1c09ae4
CREATE_TIME: 2024-04-28T13:39:07+00:00
DURATION: 34S
SOURCE: gs://qwiklabs-gcp-01-c78491702415_cloudbuild/source/1714311544.498301-c321f645e2854cb993675e3523337faa.tgz
IMAGES: us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:d4080a7
STATUS: SUCCESS
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

Cloud Build streams the logs generated by the creation of the container image to your terminal when you execute this command.

1. After the build finishes, in the Cloud console go to **Artifact Registry > Repositories** to verify that your new container image is indeed available in Artifact Registry. Click **my-repository**.

## Create the Continuous Integration (CI) pipeline

In this task, you will configure Cloud Build to automatically run a  small unit test, build the container image, and then push it to Artifact Registry. Pushing a new commit to Cloud Source Repositories triggers  this pipeline automatically. The **cloudbuild.yaml** file already included in the code is the pipeline's configuration.
```yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat cloudbuild.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START cloudbuild]
steps:
# This step runs the unit tests on the app
- name: 'python:3.7-slim'
  id: Test
  entrypoint: /bin/sh
  args:
  - -c
  - 'pip install flask && python test_app.py -v'

# This step builds the container image.
- name: 'gcr.io/cloud-builders/docker'
  id: Build
  args:
  - 'build'
  - '-t'
  - 'us-east4-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:$SHORT_SHA'
  - '.'

# This step pushes the image to Artifact Registry
# The PROJECT_ID and SHORT_SHA variables are automatically
# replaced by Cloud Build.
- name: 'gcr.io/cloud-builders/docker'
  id: Push
  args:
  - 'push'
  - 'us-east4-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:$SHORT_SHA'
# [END cloudbuild]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

1. In the Cloud console, go to **Cloud Build > Triggers**.
2. Click **Create Trigger**
3. In the Name field, type `hello-cloudbuild`.
4. Under **Event**, select **Push to a branch**.
5. Under **Source**, select **hello-cloudbuild-app** as your **Repository** and `.* (any branch)` as your **Branch**.
6. Under **Build configuration**, select **Cloud Build configuration file**.
7. In the **Cloud Build configuration file location** field, type `cloudbuild.yaml` after the /.
8. Click **Create**.

When the trigger is created, return to the Cloud Shell. You now need  to push the application code to Cloud Source Repositories to trigger the CI pipeline in Cloud Build.

1. To start this trigger, run the following command:

   ```sh
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git add .
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git commit -m "Type Any Commit Message here"
   [master b394925] Type Any Commit Message here
    4 files changed, 7 insertions(+), 7 deletions(-)
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git push google master
   Enumerating objects: 34, done.
   Counting objects: 100% (34/34), done.
   Delta compression using up to 2 threads
   Compressing objects: 100% (26/26), done.
   Writing objects: 100% (34/34), 15.97 KiB | 15.97 MiB/s, done.
   Total 34 (delta 14), reused 24 (delta 6), pack-reused 0
   remote: Resolving deltas: 100% (14/14)
   remote: Waiting for private key checker: 20/20 objects left
   To https://source.developers.google.com/p/qwiklabs-gcp-01-c78491702415/r/hello-cloudbuild-app
    * [new branch]      master -> master
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
   ```

   

## Create the Test Environment and CD pipeline

Cloud Build is also used for the continuous delivery pipeline. The  pipeline runs each time a commit is pushed to the candidate branch of  the **hello-cloudbuild-env** repository. The pipeline  applies the new version of the manifest to the Kubernetes cluster and,  if successful, copies the manifest over to the production branch. This  process has the following properties:

- The candidate branch is a history of the deployment attempts.
- The production branch is a history of the successful deployments.
- You have a view of successful and failed deployments in Cloud Build.
- You can rollback to any previous deployment by re-executing the  corresponding build in Cloud Build. A rollback also updates the  production branch to truthfully reflect the history of deployments.

Next you will modify the continuous integration pipeline to update the candidate branch of the **hello-cloudbuild-env** repository, triggering the continuous delivery pipeline.

```yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat cloudbuild-delivery.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START cloudbuild-delivery]
steps:
# This step deploys the new version of our container image
# in the hello-cloudbuild Kubernetes Engine cluster.
- name: 'gcr.io/cloud-builders/kubectl'
  id: Deploy
  args:
  - 'apply'
  - '-f'
  - 'kubernetes.yaml'
  env:
  - 'CLOUDSDK_COMPUTE_REGION=us-east4'
  - 'CLOUDSDK_CONTAINER_CLUSTER=hello-cloudbuild'

# This step copies the applied manifest to the production branch
# The COMMIT_SHA variable is automatically
# replaced by Cloud Build.
- name: 'gcr.io/cloud-builders/git'
  id: Copy to production branch
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    set -x && \
    # Configure Git to create commits with Cloud Build's service account
    git config user.email $(gcloud auth list --filter=status:ACTIVE --format='value(account)') && \
    # Switch to the production branch and copy the kubernetes.yaml file from the candidate branch
    git fetch origin production && git checkout production && \
    git checkout $COMMIT_SHA kubernetes.yaml && \
    # Commit the kubernetes.yaml file with a descriptive commit message
    git commit -m "Manifest from commit $COMMIT_SHA
    $(git log --format=%B -n 1 $COMMIT_SHA)" && \
    # Push the changes back to Cloud Source Repository
    git push origin production
# [END cloudbuild-delivery]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```
```yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat cloudbuild-trigger-cd.yaml 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# [START cloudbuild]
steps:
# This step runs the unit tests on the app
- name: 'python:3.7-slim'
  id: Test
  entrypoint: /bin/sh
  args:
  - -c
  - 'pip install flask && python test_app.py -v'

# This step builds the container image.
- name: 'gcr.io/cloud-builders/docker'
  id: Build
  args:
  - 'build'
  - '-t'
  - 'us-east4-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:$SHORT_SHA'
  - '.'

# This step pushes the image to Artifact Registry
# The PROJECT_ID and SHORT_SHA variables are automatically
# replaced by Cloud Build.
- name: 'gcr.io/cloud-builders/docker'
  id: Push
  args:
  - 'push'
  - 'us-east4-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:$SHORT_SHA'
# [END cloudbuild]

# [START cloudbuild-trigger-cd]
# This step clones the hello-cloudbuild-env repository
- name: 'gcr.io/cloud-builders/gcloud'
  id: Clone env repository
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    gcloud source repos clone hello-cloudbuild-env && \
    cd hello-cloudbuild-env && \
    git checkout candidate && \
    git config user.email $(gcloud auth list --filter=status:ACTIVE --format='value(account)')

# This step generates the new manifest
- name: 'gcr.io/cloud-builders/gcloud'
  id: Generate manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
     sed "s/GOOGLE_CLOUD_PROJECT/${PROJECT_ID}/g" kubernetes.yaml.tpl | \
     sed "s/COMMIT_SHA/${SHORT_SHA}/g" > hello-cloudbuild-env/kubernetes.yaml

# This step pushes the manifest back to hello-cloudbuild-env
- name: 'gcr.io/cloud-builders/gcloud'
  id: Push manifest
  entrypoint: /bin/sh
  args:
  - '-c'
  - |
    set -x && \
    cd hello-cloudbuild-env && \
    git add kubernetes.yaml && \
    git commit -m "Deploying image us-east4-docker.pkg.dev/$PROJECT_ID/my-repository/hello-cloudbuild:${SHORT_SHA}
    Built from commit ${COMMIT_SHA} of repository hello-cloudbuild-app
    Author: $(git log --format='%an <%ae>' -n 1 HEAD)" && \
    git push origin candidate

# [END cloudbuild-trigger-cd]
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```



### Grant Cloud Build access to GKE

To deploy the application in your Kubernetes cluster, Cloud Build  needs the Kubernetes Engine Developer Identity and Access Management  role.

1. In Cloud Shell execute the following command:

   ```sh
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ gcloud projects add-iam-policy-binding ${PROJECT_NUMBER} \
   --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com \
   --role=roles/container.developer
   Updated IAM policy for project [359830206908].
   bindings:
   - members:
     - serviceAccount:service-359830206908@gcp-sa-artifactregistry.iam.gserviceaccount.com
     role: roles/artifactregistry.serviceAgent
   - members:
     - serviceAccount:qwiklabs-gcp-01-c78491702415@qwiklabs-gcp-01-c78491702415.iam.gserviceaccount.com
     role: roles/bigquery.admin
   - members:
     - serviceAccount:359830206908@cloudbuild.gserviceaccount.com
     role: roles/cloudbuild.builds.builder
   - members:
     - serviceAccount:service-359830206908@gcp-sa-cloudbuild.iam.gserviceaccount.com
     role: roles/cloudbuild.serviceAgent
   - members:
     - serviceAccount:service-359830206908@compute-system.iam.gserviceaccount.com
     role: roles/compute.serviceAgent
   - members:
     - serviceAccount:359830206908@cloudbuild.gserviceaccount.com
     role: roles/container.developer
   - members:
     - serviceAccount:service-359830206908@container-engine-robot.iam.gserviceaccount.com
     role: roles/container.serviceAgent
   - members:
     - serviceAccount:service-359830206908@container-analysis.iam.gserviceaccount.com
     role: roles/containeranalysis.ServiceAgent
   - members:
     - serviceAccount:service-359830206908@gcp-sa-containerscanning.iam.gserviceaccount.com
     role: roles/containerscanning.ServiceAgent
   - members:
     - serviceAccount:359830206908-compute@developer.gserviceaccount.com
     - serviceAccount:359830206908@cloudservices.gserviceaccount.com
     role: roles/editor
   - members:
     - serviceAccount:service-359830206908@gcp-sa-networkconnectivity.iam.gserviceaccount.com
     role: roles/networkconnectivity.serviceAgent
   - members:
     - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
     - serviceAccount:qwiklabs-gcp-01-c78491702415@qwiklabs-gcp-01-c78491702415.iam.gserviceaccount.com
     - user:student-01-3c1bb184aaad@qwiklabs.net
     role: roles/owner
   - members:
     - serviceAccount:qwiklabs-gcp-01-c78491702415@qwiklabs-gcp-01-c78491702415.iam.gserviceaccount.com
     role: roles/storage.admin
   - members:
     - user:student-01-3c1bb184aaad@qwiklabs.net
     role: roles/viewer
   etag: BwYXKG2cqoY=
   version: 1
   student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
   ```

   

You need to initialize the **hello-cloudbuild-env** repository with two branches (production and candidate) and a Cloud Build configuration file describing the deployment process.

The first step is to clone the **hello-cloudbuild-env** repository and create the production branch. It is still empty.

You need to copy the **cloudbuild-delivery.yaml** file available in the **hello-cloudbuild-app** repository and commit the change:

The `cloudbuild-delivery.yaml` file describes the deployment process to be run in Cloud Build. It has two steps:

- Cloud Build applies the manifest on the GKE cluster.
- If successful, Cloud Build copies the manifest on the production branch.

```sh
tudent_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cd ~
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ gcloud source repos clone hello-cloudbuild-env
Cloning into '/home/student_01_3c1bb184aaad/hello-cloudbuild-env'...
warning: You appear to have cloned an empty repository.
Project [qwiklabs-gcp-01-c78491702415] repository [hello-cloudbuild-env] was cloned to [/home/student_01_3c1bb184aaad/hello-cloudbuild-env].
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-env
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git checkout -b production
Switched to a new branch 'production'
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-env
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ cp ~/hello-cloudbuild-app/cloudbuild-delivery.yaml ~/hello-cloudbuild-env/cloudbuild.yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git add .
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git commit -m "Create cloudbuild.yaml for deployment"
[production (root-commit) 5dea9e6] Create cloudbuild.yaml for deployment
 1 file changed, 49 insertions(+)
 create mode 100644 cloudbuild.yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ 
```

Create a candidate branch and push both branches for them to be available in Cloud Source Repositories:

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ 
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git checkout -b candidate
Switched to a new branch 'candidate'
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git push origin production
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 1.17 KiB | 1.17 MiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote: Waiting for private key checker: 1/1 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-01-c78491702415/r/hello-cloudbuild-env
 * [new branch]      production -> production
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ git push origin candidate
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://source.developers.google.com/p/qwiklabs-gcp-01-c78491702415/r/hello-cloudbuild-env
 * [new branch]      candidate -> candidate
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ 
```

Grant the Source Repository Writer IAM role to the Cloud Build service account for the **hello-cloudbuild-env** repository:

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} \
--format='get(projectNumber)')"
cat >/tmp/hello-cloudbuild-env-policy.yaml <<EOF
bindings:
- members:
  - serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
  role: roles/source.writer
EOF
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ 
```

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ gcloud source repos set-iam-policy \
hello-cloudbuild-env /tmp/hello-cloudbuild-env-policy.yaml
Updated IAM policy for repo [hello-cloudbuild-env].
bindings:
- members:
  - serviceAccount:359830206908@cloudbuild.gserviceaccount.com
  role: roles/source.writer
etag: BwYXKH9usBw=
version: 1
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ 
```

### Create the trigger for the continuous delivery pipeline

1. In the Cloud console, go to **Cloud Build > Triggers**.
2. Click **Create Trigger**.
3. In the Name field, type `hello-cloudbuild-deploy`.
4. Under **Event**, select **Push to a branch**.
5. Under **Source**, select **hello-cloudbuild-env** as your **Repository** and `^candidate$` as your **Branch**.
6. Under **Build configuration**, select **Cloud Build configuration file**.
7. In the **Cloud Build configuration file location** field, type `cloudbuild.yaml` after the /.
8. Click **Create**.

### Modify the continuous integration pipeline to trigger the continuous delivery pipeline.

Next, add some steps to the continuous integration pipeline that will generate a new version of the Kubernetes manifest and push it to the **hello-cloudbuild-env** repository to trigger the continuous delivery pipeline.

1. Copy the extended version of the **cloudbuild.yaml** file for the **app repository**:

The **cloudbuild-trigger-cd.yaml** is an extended version of the **cloudbuild.yaml** file. It adds the steps below: they generate the new Kubernetes manifest and trigger the continuous delivery pipeline.



```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-env (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cp cloudbuild-trigger-cd.yaml cloudbuild.yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

This pipeline uses a simple `sed` to render the manifest  template. In reality, you will benefit from using a dedicated tool such  as kustomize or skaffold. They allow for more control over the rendering of the manifest templates.



Commit the modifications and push them to Cloud Source Repositories:

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git add cloudbuild.yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git commit -m "Trigger CD pipeline"
[master 589731b] Trigger CD pipeline
 1 file changed, 40 insertions(+)
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git push google master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 237 bytes | 237.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1)
To https://source.developers.google.com/p/qwiklabs-gcp-01-c78491702415/r/hello-cloudbuild-app
   b394925..589731b  master -> master
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

## Review Cloud Build Pipeline

1. In the Cloud console, go to **Cloud Build > Dashboard**.
2. Click into the **hello-cloudbuild-app** trigger to follow its execution and examine its logs. The last step of this pipeline pushes the new manifest to the **hello-cloudbuild-env** repository, which triggers the continuous delivery pipeline.

```yaml
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat kubernetes.yaml.tpl 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-cloudbuild
  labels:
    app: hello-cloudbuild
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-cloudbuild
  template:
    metadata:
      labels:
        app: hello-cloudbuild
    spec:
      containers:
      - name: hello-cloudbuild
        image: us-east4-docker.pkg.dev/GOOGLE_CLOUD_PROJECT/my-repository/hello-cloudbuild:COMMIT_SHA
        ports:
        - containerPort: 8080
---
kind: Service
apiVersion: v1
metadata:
  name: hello-cloudbuild
spec:
  selector:
    app: hello-cloudbuild
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get nodes
NAME                                              STATUS   ROLES    AGE   VERSION
gke-hello-cloudbuild-default-pool-7f8f1841-8bsv   Ready    <none>   33m   v1.28.7-gke.1026000
gke-hello-cloudbuild-default-pool-ba456a74-t9xx   Ready    <none>   33m   v1.28.7-gke.1026000
gke-hello-cloudbuild-default-pool-e74332d5-1v68   Ready    <none>   33m   v1.28.7-gke.1026000
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get services
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
hello-cloudbuild   LoadBalancer   10.101.220.243   35.245.55.249   80:31095/TCP   110s
kubernetes         ClusterIP      10.101.208.1     <none>          443/TCP        34m
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
hello-cloudbuild   1/1     1            1           115s
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
hello-cloudbuild-594cc8f88-tznnk   1/1     Running   0          119s
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

The complete CI/CD pipeline is now configured. Test it from end to end.

1. In the Cloud console, go to **Kubernetes Engine > Gateways,Services & Ingress**.

There should be a single service called **hello-cloudbuild** in the list. It has been created by the continuous delivery build that just ran.

1. Click on the endpoint for the **hello-cloudbuild**  service. You should see "Hello World!". If there is no endpoint, or if  you see a load balancer error, you may have to wait a few minutes for  the load balancer to be completely initialized. Click **Refresh** to update the page if needed.

In Cloud Shell, replace "Hello World" with "Hello Cloud Build", both in the application and in the unit test:

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cd ~/hello-cloudbuild-app
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ sed -i 's/Hello World/Hello Cloud Build/g' app.py
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ sed -i 's/Hello World/Hello Cloud Build/g' test_app.py
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git add app.py test_app.py
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ cat test_app.py 
# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import unittest
from app import hello

class TestHelloApp(unittest.TestCase):

  def test_hello(self):
    self.assertEqual(hello(), "Hello Cloud Build!\n")

if __name__ == '__main__':
  unittest.main()
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git commit -m "Hello Cloud Build"
[master e1cbcdf] Hello Cloud Build
 2 files changed, 2 insertions(+), 2 deletions(-)
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ git push google master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 2 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 378 bytes | 189.00 KiB/s, done.
Total 4 (delta 3), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (3/3)
remote: Waiting for private key checker: 2/2 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-01-c78491702415/r/hello-cloudbuild-app
   589731b..e1cbcdf  master -> master
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

This triggers the full CI/CD pipeline.

After a few minutes, reload the application in your browser. You should now see "Hello Cloud Build!".

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl describe pods| grep Image
    Image:          us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:e1cbcdf
    Image ID:       us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild@sha256:265ee03d85788f16f352923ac30445adfc8a864c254c9e3f8a887527886d0f33
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```



## Test the rollback

In this task, you rollback to the version of the application that said "Hello World!".

1. In the Cloud console, go to **Cloud Build > Dashboard**.
2. Click on *View all* link under **Build History** for the **hello-cloudbuild-env** repository.
3. Click on the second most recent build available.
4. Click **Rebuild**.

When the build is finished, reload the application in your browser. You should now see "Hello World!" again.

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl describe pods| grep Image
    Image:          us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:589731b
    Image ID:       us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild@sha256:0dc0b75a7b20b0f86c52dc571fc1c003d38a13138969799972168f3db3cfc882
    Image:          us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:e1cbcdf
    Image ID:       us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild@sha256:265ee03d85788f16f352923ac30445adfc8a864c254c9e3f8a887527886d0f33
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
hello-cloudbuild-594cc8f88-jfl94   1/1     Running   0          38s
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
hello-cloudbuild-594cc8f88-jfl94   1/1     Running   0          44s
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ kubectl describe pods| grep Image
    Image:          us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild:589731b
    Image ID:       us-east4-docker.pkg.dev/qwiklabs-gcp-01-c78491702415/my-repository/hello-cloudbuild@sha256:0dc0b75a7b20b0f86c52dc571fc1c003d38a13138969799972168f3db3cfc882
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

Now you can use Cloud Build to create and rollback continuous integration pipelines with GKE on Google Cloud!

```sh
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ history 
    1  gcloud config list project
    2  gcloud auth list
    3  export PROJECT_ID=$(gcloud config get-value project)
    4  export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
    5  export REGION=us-east4
    6  gcloud config set compute/region $REGION
    7  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     sourcerepo.googleapis.com     containeranalysis.googleapis.com
    8  gcloud artifacts repositories create my-repository   --repository-format=docker   --location=$REGION
    9  git config --global user.email "pradeep@example.com"  
   10  git config --global user.name "Pradeep Gadde"
   11  gcloud source repos create hello-cloudbuild-app
   12  gcloud source repos create hello-cloudbuild-env
   13  cd ~
   14  git clone https://github.com/GoogleCloudPlatform/gke-gitops-tutorial-cloudbuild hello-cloudbuild-app
   15  cd ~/hello-cloudbuild-app
   16  export REGION=us-east4
   17  sed -i "s/us-central1/$REGION/g" cloudbuild.yaml
   18  sed -i "s/us-central1/$REGION/g" cloudbuild-delivery.yaml
   19  sed -i "s/us-central1/$REGION/g" cloudbuild-trigger-cd.yaml
   20  sed -i "s/us-central1/$REGION/g" kubernetes.yaml.tpl
   21  PROJECT_ID=$(gcloud config get-value project)
   22  git remote add google "https://source.developers.google.com/p/${PROJECT_ID}/r/hello-cloudbuild-app"
   23  ls
   24  cat app.py 
   25  ls
   26  cat cloudbuild.yaml 
   27  ls
   28  cat Dockerfile 
   29  ls
   30  cat cloudbuild-trigger-cd.yaml 
   31  ls
   32  cat cloudbuild-delivery.yaml 
   33  cd ~/hello-cloudbuild-app
   34  COMMIT_ID="$(git rev-parse --short=7 HEAD)"
   35  gcloud builds submit --tag="${REGION}-docker.pkg.dev/${PROJECT_ID}/my-repository/hello-cloudbuild:${COMMIT_ID}" .
   36  git rev-parse --short=7 HEAD
   37  cd ~/hello-cloudbuild-app
   38  git add .
   39  git commit -m "Type Any Commit Message here"
   40  git push google master
   41  PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} --format='get(projectNumber)')"
   42  gcloud projects add-iam-policy-binding ${PROJECT_NUMBER} --member=serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com --role=roles/container.developer
   43  cd ~
   44  gcloud source repos clone hello-cloudbuild-env
   45  cd ~/hello-cloudbuild-env
   46  git checkout -b production
   47  cd ~/hello-cloudbuild-env
   48  cp ~/hello-cloudbuild-app/cloudbuild-delivery.yaml ~/hello-cloudbuild-env/cloudbuild.yaml
   49  git add .
   50  git commit -m "Create cloudbuild.yaml for deployment"
   51  git checkout -b candidate
   52  git push origin production
   53  git push origin candidate
   54  PROJECT_NUMBER="$(gcloud projects describe ${PROJECT_ID} \
--format='get(projectNumber)')"
   55  cat >/tmp/hello-cloudbuild-env-policy.yaml <<EOF
bindings:
- members:
  - serviceAccount:${PROJECT_NUMBER}@cloudbuild.gserviceaccount.com
  role: roles/source.writer
EOF

   56  gcloud source repos set-iam-policy hello-cloudbuild-env /tmp/hello-cloudbuild-env-policy.yaml
   57  cd ~/hello-cloudbuild-app
   58  cp cloudbuild-trigger-cd.yaml cloudbuild.yaml
   59  cd ~/hello-cloudbuild-app
   60  git add cloudbuild.yaml
   61  git commit -m "Trigger CD pipeline"
   62  git push google master
   63  ls
   64  cat kubernetes.yaml.tpl 
   65  kubectl get nodes
   66  kubectl get services
   67  kubectl get deploy
   68  kubectl get pods
   69  cd ~/hello-cloudbuild-app
   70  sed -i 's/Hello World/Hello Cloud Build/g' app.py
   71  sed -i 's/Hello World/Hello Cloud Build/g' test_app.py
   72  git add app.py test_app.py
   73  cat test_app.py 
   74  git commit -m "Hello Cloud Build"
   75  git push google master
   76  kubectl get pods
   77  kubectl describe pods
   78  kubectl describe pods| grep Image
   79  kubectl get pods
   80  kubectl describe pods| grep Image
   81  history 
student_01_3c1bb184aaad@cloudshell:~/hello-cloudbuild-app (qwiklabs-gcp-01-c78491702415)$ 
```

