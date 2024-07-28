---

layout: single
title:  "Creating CI/CD pipelines for Anthos clusters "
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/anthos.png
  og_image: /assets/images/anthos.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Creating CI/CD pipelines for Anthos clusters

In this exercise, you create a modern serverless continuous integration/continuous delivery (CI/CD) pipeline to automate application delivery to GKE and Anthos clusters.

The pipeline starts with a "hello world" Node.js code stored in Cloud Source Repositories in a private Git repository. Commits that are made to the master branch trigger a Cloud Build workflow, which builds a new container image, pushes it to Container Registry, and triggers Google Cloud Deploy as the CD part.

Google Cloud Deploy delivers the application to the dev cluster, which, after testing, allows you to promote it to a staging and  production cluster. Additionally, you learn to implement approvals to ensure that code only reaches production when the right stakeholders approve the release.

![Lab architecture diagram](https://cdn.qwiklabs.com/7mWXsBQVFSuCxnbU%2BDDoQGFV%2FDzdDAtCpBqpTglle38%3D)

- Create and use a repository in Cloud Source Repositories.
- Set up continous integration with Cloud Build workflows.
- Configure continous delivery pipelines with Google Cloud Deploy.

## Create a basic Node.js application from the Google Cloud Console

1. Activate Cloud Shell.

2. If the tab under the Cloud Shell header does not list your project ID *qwiklabs-gcp-[hex]*, press the down arrow to create a new Cloud Shell tab associated with the correct project.

3. Enable the required Google Cloud APIs:

4. Export the current Qwiklabs project ID into a PROJECT_ID environmental variable:

5. Set the Zone environment variable:

6. Authenticate to the Container Registry:

7. Clone a copy of a hello world example into Cloud Shell, and change into the HelloWorldNodeJs folder that the clone operation creates

8. You can explore the code in the Cloud Shell Editor.

   Use the Docker CLI to build the code into a container and push it to Container Registry:

9. The *docker build* command looks in the specified path (.) for a Dockerfile. If it finds one, it follows the instructions to build a Docker image (in this case adding the specified tag `-t`). *docker push* then uploads a copy of the image to Container Registry.

   In the Google Cloud Console, navigate to Container Registry and confirm that the image was created and uploaded as expected.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-7d4f733ccbf1.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_b9ddf527878d@cloudshell:~ (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud services enable container.googleapis.com \
    cloudbuild.googleapis.com \
    artifactregistry.googleapis.com \
    clouddeploy.googleapis.com \
    cloudresourcemanager.googleapis.com
Operation "operations/acat.p2-301498661488-2f608e6c-f5d3-46ac-b7ab-49b40864d627" finished successfully.
student_00_b9ddf527878d@cloudshell:~ (qwiklabs-gcp-04-7d4f733ccbf1)$ export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
student_00_b9ddf527878d@cloudshell:~ (qwiklabs-gcp-04-7d4f733ccbf1)$ CLUSTER_ZONE=us-central1-b
student_00_b9ddf527878d@cloudshell:~ (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud auth configure-docker
WARNING: Your config file at [/home/student_00_b9ddf527878d/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud",
    "africa-south1-docker.pkg.dev": "gcloud",
    "asia-docker.pkg.dev": "gcloud",
    "asia-east1-docker.pkg.dev": "gcloud",
    "asia-east2-docker.pkg.dev": "gcloud",
    "asia-northeast1-docker.pkg.dev": "gcloud",
    "asia-northeast2-docker.pkg.dev": "gcloud",
    "asia-northeast3-docker.pkg.dev": "gcloud",
    "asia-south1-docker.pkg.dev": "gcloud",
    "asia-south2-docker.pkg.dev": "gcloud",
    "asia-southeast1-docker.pkg.dev": "gcloud",
    "asia-southeast2-docker.pkg.dev": "gcloud",
    "australia-southeast1-docker.pkg.dev": "gcloud",
    "australia-southeast2-docker.pkg.dev": "gcloud",
    "europe-docker.pkg.dev": "gcloud",
    "europe-central2-docker.pkg.dev": "gcloud",
    "europe-north1-docker.pkg.dev": "gcloud",
    "europe-southwest1-docker.pkg.dev": "gcloud",
    "europe-west1-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "docker.me-central2.rep.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "southamerica-west1-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud",
    "us-west8-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud"
  }
}
Adding credentials for all GCR repositories.
WARNING: A long list of credential helpers may cause delays running 'docker build'. We recommend passing the registry name to configure only the registry you are using.
gcloud credential helpers already registered correctly.
student_00_b9ddf527878d@cloudshell:~ (qwiklabs-gcp-04-7d4f733ccbf1)$ git clone https://github.com/haggman/HelloWorldNodeJs.git
cd HelloWorldNodeJs/
Cloning into 'HelloWorldNodeJs'...
remote: Enumerating objects: 115, done.
remote: Counting objects: 100% (69/69), done.
remote: Compressing objects: 100% (53/53), done.
remote: Total 115 (delta 33), reused 30 (delta 12), pack-reused 46
Receiving objects: 100% (115/115), 24.17 KiB | 2.42 MiB/s, done.
Resolving deltas: 100% (48/48), done.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ ls
app.yaml  buildContainer.sh  Dockerfile  index.js  k8sapp.yaml  k8s-autoscaler.yaml  loadgenerator  package.json  readme.md
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ cat Dockerfile 
FROM node:16-bookworm-slim

WORKDIR /usr/src/app

COPY package.json ./

RUN npm install --only=production

COPY . .

student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ docker build -t gcr.io/$PROJECT_ID/helloworld .ROJECT_ID/helloworld .
docker push gcr.io/$PROJECT_ID/helloworld
[+] Building 12.0s (10/10) FINISHED                                                                                                                                  docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 178B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/node:16-bookworm-slim                                                                                                       1.9s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 152B                                                                                                                                              0.0s
 => [1/5] FROM docker.io/library/node:16-bookworm-slim@sha256:89061680359f5d9103d24f0066e2c96a8018c24d5c4fe8eb8d59d16dd4ab64ba                                                 5.5s
 => => resolve docker.io/library/node:16-bookworm-slim@sha256:89061680359f5d9103d24f0066e2c96a8018c24d5c4fe8eb8d59d16dd4ab64ba                                                 0.0s
 => => sha256:89061680359f5d9103d24f0066e2c96a8018c24d5c4fe8eb8d59d16dd4ab64ba 1.21kB / 1.21kB                                                                                 0.0s
 => => sha256:b7455f5272e7397f3879a8b3bc7263d18dfb95e75d74ed56cf5506b5d8bc493f 1.37kB / 1.37kB                                                                                 0.0s
 => => sha256:79726f7d8307dfae41a122dd51af60f1597719654c2998158032896f5fe924a2 7.02kB / 7.02kB                                                                                 0.0s
 => => sha256:360eba32fa65016e0d558c6af176db31a202e9a6071666f9b629cb8ba6ccedf0 29.12MB / 29.12MB                                                                               1.0s
 => => sha256:24f632c8bcc8f838302a408474f63c842f77e774d05328594fb0e195569cd7aa 3.36kB / 3.36kB                                                                                 0.4s
 => => sha256:075fefbc2ffbf19b76e590482e1f82654df9dda0613b5bd552b9dfb012ec4d32 35.25MB / 35.25MB                                                                               0.8s
 => => sha256:d694c153631ff9210d858b771a82fb2b47ea3a708ffb30bf5f9e7c3caa74866e 2.73MB / 2.73MB                                                                                 1.0s
 => => sha256:45bad16176ca6bae5557955e5121a0400a10b46c7b1e38a0b47e790e02cad3f5 450B / 450B                                                                                     1.2s
 => => extracting sha256:360eba32fa65016e0d558c6af176db31a202e9a6071666f9b629cb8ba6ccedf0                                                                                      1.9s
 => => extracting sha256:24f632c8bcc8f838302a408474f63c842f77e774d05328594fb0e195569cd7aa                                                                                      0.0s
 => => extracting sha256:075fefbc2ffbf19b76e590482e1f82654df9dda0613b5bd552b9dfb012ec4d32                                                                                      2.0s
 => => extracting sha256:d694c153631ff9210d858b771a82fb2b47ea3a708ffb30bf5f9e7c3caa74866e                                                                                      0.1s
 => => extracting sha256:45bad16176ca6bae5557955e5121a0400a10b46c7b1e38a0b47e790e02cad3f5                                                                                      0.0s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 61.34kB                                                                                                                                           0.0s
 => [2/5] WORKDIR /usr/src/app                                                                                                                                                 1.1s
 => [3/5] COPY package.json ./                                                                                                                                                 0.0s
 => [4/5] RUN npm install --only=production                                                                                                                                    3.1s
 => [5/5] COPY . .                                                                                                                                                             0.0s
 => exporting to image                                                                                                                                                         0.2s
 => => exporting layers                                                                                                                                                        0.2s
 => => writing image sha256:6b3795cdd898637e8e29e611003a829b3e8e28aa4495293f063a92849d81cb30                                                                                   0.0s
 => => naming to gcr.io/qwiklabs-gcp-04-7d4f733ccbf1/helloworld                                                                                                                0.0s
Using default tag: latest
The push refers to repository [gcr.io/qwiklabs-gcp-04-7d4f733ccbf1/helloworld]
d6727d446513: Pushed 
aaedf546e919: Pushed 
bf6111039f60: Pushed 
557c16889edc: Pushed 
dbf72bc23fb4: Layer already exists 
0e1d2e3ff122: Layer already exists 
20244d1e9a38: Layer already exists 
9a2a56d458ec: Layer already exists 
a2d7501dfb35: Layer already exists 
latest: digest: sha256:d9903a5ba9131c734bf7a45a4ed9f25520c3ebab455e8538b0d5df1d5c9feb43 size: 2201
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ history 
    1  gcloud services enable container.googleapis.com     cloudbuild.googleapis.com     artifactregistry.googleapis.com     clouddeploy.googleapis.com     cloudresourcemanager.googleapis.com
    2  export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
    3  CLUSTER_ZONE=us-central1-b
    4  gcloud auth configure-docker
    5  git clone https://github.com/haggman/HelloWorldNodeJs.git
    6  cd HelloWorldNodeJs/
    7  ls
    8  cat Dockerfile 
    9  docker build -t gcr.io/$PROJECT_ID/helloworld .
   10  docker push gcr.io/$PROJECT_ID/helloworld
   11  history 
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```



## Configure your CI/CD pipeline with Cloud Build and Google Cloud Deploy

In the previous task, you took code from a Git repository, used Docker to build an image, and pushed it to Container Registry. Cloud Build allows you to create workflows of build steps similar to the kind you executed manually. Google and the Cloud Build community have created several builders, each usable by name.

In this task, you create a `cloudbuild.yaml` file to  define the build workflow: build the image, push it to Container  Registry, and finally push it to Google Cloud Deploy. From there, Google Cloud Deploy manages continous delivery. You then test for correctness.

![Cloud Build workflow diagram](https://cdn.qwiklabs.com/J1TMRA1wySpsd9wdKzpoWc%2Bw6beVk0o7a9s8hoDq1mM%3D)

Create a `cloudbuild.yaml` file, which executes the same steps you just performed manually:

The file name `cloudbuild.yaml` is the expected default, but you can give it a custom name. At its most basic, the `cloudbuild.yaml` file lays out the steps the Cloud Build worker should complete. The file can then be manually passed to Cloud Build or automated as part of a trigger.

1. Create a `clouddeploy.yaml` file to specify the delivery pipeline containing the target platforms where the `helloworld` file should be delivered; in this case, the three existing GKE clusters: dev-cluster, staging-cluster, and prod-cluster:

2. Create the `scaffold.yaml` file to specify the Kubernetes YAML files to be deployed by Google Cloud Deploy:

3. Create the Kubernetes files to be deployed to the clusters:

4. Deploy the `clouddeploy.yaml` file to Google Cloud Deploy to configure the release pipeline:

5. In the Google Cloud Console, navigate to Google Cloud Deploy and confirm that the pipeline was created.

   Click on **helloworld-pipeline** to open it and view the pipeline.

   Use Cloud Build to execute the workflow:

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ cat <<EOF > cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/helloworld', '.']
  # Push the container image to Artifact Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/helloworld']
  # Create release in Google Cloud Deploy
  - name: gcr.io/google.com/cloudsdktool/cloud-sdk
    entrypoint: gcloud
    args: 
      [
        "beta", "deploy", "releases", "create", "rel-\${SHORT_SHA}a",
        "--delivery-pipeline", "helloworld-pipeline",
EOF   ] "--images", "helloworld=gcr.io/$PROJECT_ID/helloworld"
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ cat <<EOF > clouddeploy.yaml
apiVersion: deploy.cloud.google.com/v1beta1
kind: DeliveryPipeline
metadata:
  name: helloworld-pipeline
description: helloworld application delivery pipeline
serialPipeline:
 stages:
 - targetId: dev
   profiles: []
 - targetId: staging
   profiles: []
 - targetId: prod
   profiles: []
EOFluster: projects/$PROJECT_ID/locations/$CLUSTER_ZONE/clusters/prod-clusterter
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ cat <<EOF > skaffold.yaml
apiVersion: skaffold/v2beta16
kind: Config
deploy:
  kubectl:
    manifests:
      - kubernetes-app.yaml
EOF
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ cat <<EOF > kubernetes-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-demo
  template:
    metadata:
      labels:
        app: hello-world-demo
EOF targetPort: 8080-demo8080CT_ID}/helloworld
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$
```

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud beta deploy apply --file clouddeploy.yaml \
--region=us-central1 --project=$PROJECT_ID
Waiting for the operation on resource projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/deliveryPipelines/helloworld-pipeline...done.                                    
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/deliveryPipelines/helloworld-pipeline.
Waiting for the operation on resource projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/dev...done.                                                              
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/dev.
Waiting for the operation on resource projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/staging...done.                                                          
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/staging.
Waiting for the operation on resource projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/prod...done.                                                             
Created Cloud Deploy resource: projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/targets/prod.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud builds submit --config cloudbuild.yaml
Creating temporary archive of 9 file(s) totalling 3.4 KiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-04-7d4f733ccbf1_cloudbuild/source/1722193674.833454-a0bd08d447ed4e238d8788a8b7612b7f.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/global/builds/06c7a9de-8ef2-484f-a185-19952bcd9413].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/06c7a9de-8ef2-484f-a185-19952bcd9413?project=301498661488 ].
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "06c7a9de-8ef2-484f-a185-19952bcd9413"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-04-7d4f733ccbf1_cloudbuild/source/1722193674.833454-a0bd08d447ed4e238d8788a8b7612b7f.tgz#1722193676408740
Copying gs://qwiklabs-gcp-04-7d4f733ccbf1_cloudbuild/source/1722193674.833454-a0bd08d447ed4e238d8788a8b7612b7f.tgz#1722193676408740...
/ [1 files][  1.9 KiB/  1.9 KiB]                                                
Operation completed over 1 objects/1.9 KiB.
BUILD
Starting Step #0
Step #0: Already have image (with digest): gcr.io/cloud-builders/docker
Step #0: Sending build context to Docker daemon  11.78kB
Step #0: Step 1/6 : FROM node:16-bookworm-slim
Step #0: 16-bookworm-slim: Pulling from library/node
Step #0: 360eba32fa65: Pulling fs layer
Step #0: 24f632c8bcc8: Pulling fs layer
Step #0: 075fefbc2ffb: Pulling fs layer
Step #0: d694c153631f: Pulling fs layer
Step #0: 45bad16176ca: Pulling fs layer
Step #0: d694c153631f: Waiting
Step #0: 45bad16176ca: Waiting
Step #0: 24f632c8bcc8: Verifying Checksum
Step #0: 24f632c8bcc8: Download complete
Step #0: 360eba32fa65: Verifying Checksum
Step #0: 360eba32fa65: Download complete
Step #0: d694c153631f: Verifying Checksum
Step #0: d694c153631f: Download complete
Step #0: 45bad16176ca: Verifying Checksum
Step #0: 45bad16176ca: Download complete
Step #0: 075fefbc2ffb: Verifying Checksum
Step #0: 075fefbc2ffb: Download complete
Step #0: 360eba32fa65: Pull complete
Step #0: 24f632c8bcc8: Pull complete
Step #0: 075fefbc2ffb: Pull complete
Step #0: d694c153631f: Pull complete
Step #0: 45bad16176ca: Pull complete
Step #0: Digest: sha256:89061680359f5d9103d24f0066e2c96a8018c24d5c4fe8eb8d59d16dd4ab64ba
Step #0: Status: Downloaded newer image for node:16-bookworm-slim
Step #0:  79726f7d8307
Step #0: Step 2/6 : WORKDIR /usr/src/app
Step #0:  Running in dffae745a9f0
Step #0: Removing intermediate container dffae745a9f0
Step #0:  4a0c79766075
Step #0: Step 3/6 : COPY package.json ./
Step #0:  de4e7cc29edc
Step #0: Step 4/6 : RUN npm install --only=production
Step #0:  Running in a4efbe9ea9f3
Step #0: npm WARN config only Use `--omit=dev` to omit dev dependencies from the install.
Step #0: 
Step #0: added 64 packages, and audited 65 packages in 2s
Step #0: 
Step #0: 12 packages are looking for funding
Step #0:   run `npm fund` for details
Step #0: 
Step #0: found 0 vulnerabilities
Step #0: npm notice 
Step #0: npm notice New major version of npm available! 8.19.4 -> 10.8.2
Step #0: npm notice Changelog: <https://github.com/npm/cli/releases/tag/v10.8.2>
Step #0: npm notice Run `npm install -g npm@10.8.2` to update!
Step #0: npm notice
Step #0: Removing intermediate container a4efbe9ea9f3
Step #0:  88f04955802f
Step #0: Step 5/6 : COPY . .
Step #0:  217595c1b2ee
Step #0: Step 6/6 : CMD [ "npm", "start" ]
Step #0:  Running in f7aec89f30ad
Step #0: Removing intermediate container f7aec89f30ad
Step #0:  fd0d640c49c7
Step #0: Successfully built fd0d640c49c7
Step #0: Successfully tagged gcr.io/qwiklabs-gcp-04-7d4f733ccbf1/helloworld:latest
Finished Step #0
Starting Step #1
Step #1: Already have image (with digest): gcr.io/cloud-builders/docker
Step #1: Using default tag: latest
Step #1: The push refers to repository [gcr.io/qwiklabs-gcp-04-7d4f733ccbf1/helloworld]
Step #1: ea61edfdaa2f: Preparing
Step #1: 36b356c7bb9c: Preparing
Step #1: 553b8a718b9e: Preparing
Step #1: 65076f9d22d8: Preparing
Step #1: dbf72bc23fb4: Preparing
Step #1: 0e1d2e3ff122: Preparing
Step #1: 20244d1e9a38: Preparing
Step #1: 9a2a56d458ec: Preparing
Step #1: a2d7501dfb35: Preparing
Step #1: 0e1d2e3ff122: Waiting
Step #1: 20244d1e9a38: Waiting
Step #1: 9a2a56d458ec: Waiting
Step #1: a2d7501dfb35: Waiting
Step #1: dbf72bc23fb4: Layer already exists
Step #1: 0e1d2e3ff122: Layer already exists
Step #1: ea61edfdaa2f: Pushed
Step #1: 65076f9d22d8: Pushed
Step #1: 553b8a718b9e: Pushed
Step #1: 20244d1e9a38: Layer already exists
Step #1: 36b356c7bb9c: Pushed
Step #1: 9a2a56d458ec: Layer already exists
Step #1: a2d7501dfb35: Layer already exists
Step #1: latest: digest: sha256:323a4f6230be771926bdfab7c50613b7bc0068cb0ded4fd4f4c04d7ee046e566 size: 2200
Finished Step #1
Starting Step #2
Step #2: Pulling image: gcr.io/google.com/cloudsdktool/cloud-sdk
Step #2: Using default tag: latest
Step #2: latest: Pulling from google.com/cloudsdktool/cloud-sdk
Step #2: 73226dab8db5: Pulling fs layer
Step #2: ce7fac1a5795: Pulling fs layer
Step #2: edb29c6924c5: Pulling fs layer
Step #2: 339b381494cc: Pulling fs layer
Step #2: b5e7bd0d8cfb: Pulling fs layer
Step #2: dde0a77d18d7: Pulling fs layer
Step #2: 216efcc6f646: Pulling fs layer
Step #2: 339b381494cc: Waiting
Step #2: b5e7bd0d8cfb: Waiting
Step #2: dde0a77d18d7: Waiting
Step #2: 216efcc6f646: Waiting
Step #2: edb29c6924c5: Verifying Checksum
Step #2: edb29c6924c5: Download complete
Step #2: ce7fac1a5795: Verifying Checksum
Step #2: ce7fac1a5795: Download complete
Step #2: 339b381494cc: Verifying Checksum
Step #2: 339b381494cc: Download complete
Step #2: 73226dab8db5: Verifying Checksum
Step #2: 73226dab8db5: Download complete
Step #2: 216efcc6f646: Verifying Checksum
Step #2: 216efcc6f646: Download complete
Step #2: dde0a77d18d7: Verifying Checksum
Step #2: dde0a77d18d7: Download complete
Step #2: 73226dab8db5: Pull complete
Step #2: ce7fac1a5795: Pull complete
Step #2: edb29c6924c5: Pull complete
Step #2: 339b381494cc: Pull complete
Step #2: b5e7bd0d8cfb: Verifying Checksum
Step #2: b5e7bd0d8cfb: Download complete
Step #2: b5e7bd0d8cfb: Pull complete
Step #2: dde0a77d18d7: Pull complete
Step #2: 216efcc6f646: Pull complete
Step #2: Digest: sha256:99861235b6984e7756cf5574009a706be7e83ae6f510ff14204f4e6fae515efe
Step #2: Status: Downloaded newer image for gcr.io/google.com/cloudsdktool/cloud-sdk:latest
Step #2: gcr.io/google.com/cloudsdktool/cloud-sdk:latest
Step #2: Creating temporary archive of 9 file(s) totalling 3.4 KiB before compression.
Step #2: Uploading tarball of [.] to [gs://c03e0955b8fc4968a1a4b6ac57e6a0c1_clouddeploy/source/1722193760.003354-e4f55c40c058443c90c31295994259a3.tgz]
Step #2: Waiting for operation [operation-1722193761350-61e537b613490-379769fd-dbd16e72]...
Step #2: ......done.
Step #2: Created Cloud Deploy release rel-a.
Step #2: Creating rollout projects/qwiklabs-gcp-04-7d4f733ccbf1/locations/us-central1/deliveryPipelines/helloworld-pipeline/releases/rel-a/rollouts/rel-a-to-dev-0001 in target dev
Step #2: Waiting for rollout creation operation to complete...
Step #2: ......done.
Finished Step #2
PUSH
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ID: 06c7a9de-8ef2-484f-a185-19952bcd9413
CREATE_TIME: 2024-07-28T19:07:56+00:00
DURATION: 1M28S
SOURCE: gs://qwiklabs-gcp-04-7d4f733ccbf1_cloudbuild/source/1722193674.833454-a0bd08d447ed4e238d8788a8b7612b7f.tgz
IMAGES: -
STATUS: SUCCESS
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```



What's the error?

The problem is that the service account that Cloud Build uses doesn't by default have any access to Google Cloud Deploy.

In the Google Cloud Console, navigate to **IAM & admin > IAM**. Enable the option **Include Google-provided role grants** and locate the service account with the suffix *@cloudbuild.gserviceaccount.com*.

Click **Edit** (image), and add the **Cloud Deploy > Cloud Deploy Releaser** and **Service Accounts > Service Account User** roles.

Return to Cloud Shell and retry the Cloud Build command:

In the Google Cloud Console, navigate to **Cloud Deploy**, open the pipeline you created earlier, and wait for the release to be deployed to dev-cluster.

In the Google Cloud Console, navigate to **Kubernetes Engine > Services & Ingress**, locate the hello-world-demo service, and click on the link with the public IP address to see the deployed application running on dev-cluster.



## Automate continuous integration using a Cloud Build trigger

For this task, you use a very simplified development process. You commit changes to the Git repository local to Cloud Shell and push the changes to the remote repository. When changes are pushed to Cloud Source Reposies master branch, a Cloud Build trigger based on the cloudbuild.yaml file created earlier is excuted, which builds the image, pushes it to Container Registry, and schedules a Google Cloud Deploy release.

1. In Cloud Shell, initialize and configure Git for use:

2. Stage the code for commit in the local Cloud Shell Git repository, and make the initial commit:

3. Create a new *helloworld* Git repository in your Cloud repository:

4. Git allows for the creation of remote team repositories for sharing code between developers.

   In Cloud Shell, create a remote link to the Git repository created in the previous step, and name it `to-google`:

5. Push the code from your local Cloud Shell repository to the Google repository helloworld master branch:

6. In the Google Cloud Console, navigate to Cloud Source Repositories and investigate your new repo.

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ git init
git config --global user.email "you@example.com"
git config --global user.name "Student"
Reinitialized existing Git repository in /home/student_00_b9ddf527878d/HelloWorldNodeJs/.git/
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ git add .
git commit -m "Initial commit"
[master 928e0bf] Initial commit
 4 files changed, 95 insertions(+)
 create mode 100644 cloudbuild.yaml
 create mode 100644 clouddeploy.yaml
 create mode 100644 kubernetes-app.yaml
 create mode 100644 skaffold.yaml
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud source repos create helloworld
Created [helloworld].
WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ git remote add to-google https://source.developers.google.com/p/$PROJECT_ID/r/helloworld
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ git push to-google master
Enumerating objects: 109, done.
Counting objects: 100% (109/109), done.
Delta compression using up to 2 threads
Compressing objects: 100% (62/62), done.
Writing objects: 100% (109/109), 21.98 KiB | 7.33 MiB/s, done.
Total 109 (delta 41), reused 100 (delta 39), pack-reused 0
remote: Resolving deltas: 100% (41/41)
remote: Waiting for private key checker: 59/59 objects left
To https://source.developers.google.com/p/qwiklabs-gcp-04-7d4f733ccbf1/r/helloworld
 * [new branch]      master -> master
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```



The code is written locally and stored in a central repository, the build steps are defined in a `cloudbuild.yaml` file, and the deployment steps are specified in the `clouddeploy.yaml` file. The last step is to create a trigger to put it all together.

In the Google Cloud Console, navigate to **Cloud Build > Triggers**.

Click on the **CREATE TRIGGER** button.

Enter the name `HelloWorldJS`.

Under **Source**, enter select the repository to be `helloworld (Cloud Source Repositories)` and the branch to match `^master$`.

Under **Configuration**, select `Cloud Build configuration file`, and enter `/cloudbuild.yaml` as the Cloud Build configuration file location.

Click on **CREATE** to create the trigger.

When the trigger is ready, to test the trigger's logic, click **RUN**. If a window pops up to select a branch, enter `master` and press **RUN TRIGGER**.

Navigate to **Cloud Build > History** and wait for the build workflow to complete.

Navigate to Google Cloud Deploy and confirm that the new pipeline has been triggered.

### Test the process

1. Return to Cloud Shell and edit the services code:

2. Make a change to the services returned message; for example, change `World` to `World 2`.

   Stage and commit the change to your local developers Git repository, and then push the change to the repository:

   Return to the Google Cloud Console and navigate to **Cloud Build > History**.

   When the trigger is finished executing, go to Google Cloud Deploy and wait for the deployment to occur.

   Re-load the application and confirm that the new message is displayed.

## Promote and approve releases

You have been working out of dev-cluster, which is great for developing and testing new features of your application. However, after those features have been developed, you might want to expose them to an internal audience in a staging environment for further testing and eventually deploy them to a production environment.

In this task, you learn how to promote your application to other environments and use approvals for stakeholders to approve a release.

1. In the Google Cloud Console, navigate to **Cloud Deploy** and open your pipeline.
2. In the pipeline visualization area, click on the 3-dot menu in the dev target box. Notice the following actions:
   - Promote release: Launches your application to the next target; in this case, staging-cluster.
   - Re-depoy release: If an error occurs, you can try to redeploy the current version to dev-cluster to fix any transient issues.
   - Roll back release: Discard the latest release and bring the previous version of your application to the target cluster.
3. To deploy your application to staging, click **Promote release**. Investigate the options in the pop-up window, and check the **Manifest Diff** tab to check the differences with the previous version.
4. Click **Promote**.
5. After the application is deployed, to see the deployed application running on staging-cluster, navigate to **Kubernetes Engine > Services & Ingress**.
6. Locate the hello-world-demo service and click on the link with the public IP address.
7. To promote the release from staging to production, return to **Cloud Deploy**, open your pipeline, click **Promote** in the staging box, and then in the dialog, click **Promote** again. Notice that this time the promotion has been blocked by a review action that you specified earlier in the `clouddeploy.yaml`.
8. In the visualization area between staging and production, click **Review**.
9. After reviewing the changes, click **Approve**. Usually, approvals would be done by a different person or team.
10. Return to your Cloud Deploy pipeline visualization, and click on the release that is deployed to prod-cluster.
11. Click again on the rollout to open a details window and open the **Render logs** and **Deployment logs**. Here you can debug if there are any issues in the deployment. Notice that the actual deployment also runs on a Cloud Build runner.
12. After the rollout has finished, to see the deployed application running on prod-cluster, navigate to **Kubernetes Engine > Services & Ingress**, locate the hello-world-demo service, and click on the link with the public IP address.

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud container clusters get-credentials prod-cluster --zone us-central1-b 
Fetching cluster endpoint and auth data.
kubeconfig entry generated for prod-cluster.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ kubectl get nodes
NAME                                        STATUS   ROLES    AGE   VERSION
gke-prod-cluster-pool-12345-6ba84e81-hnv5   Ready    <none>   18h   v1.29.6-gke.1038001
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-world-demo-869565858d-kpsxz   1/1     Running   0          2m45s
pod/hello-world-demo-869565858d-l68sv   1/1     Running   0          2m45s
pod/hello-world-demo-869565858d-vmnn2   1/1     Running   0          2m46s

NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
service/hello-world-demo   LoadBalancer   34.118.226.7   34.70.190.105   80:32087/TCP   2m47s
service/kubernetes         ClusterIP      34.118.224.1   <none>          443/TCP        18h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-world-demo   3/3     3            3           2m47s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-world-demo-869565858d   3         3         3       2m47s
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud container clusters get-credentials staging-clust
er --zone us-central1-b 
Fetching cluster endpoint and auth data.
kubeconfig entry generated for staging-cluster.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ kubectl get all
NAME                                   READY   STATUS    RESTARTS   AGE
pod/hello-world-demo-8d8c747b5-hwtxj   1/1     Running   0          5m42s
pod/hello-world-demo-8d8c747b5-j9mxd   1/1     Running   0          5m42s
pod/hello-world-demo-8d8c747b5-nbz7t   1/1     Running   0          5m42s

NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
service/hello-world-demo   LoadBalancer   34.118.225.126   34.44.235.54   80:30277/TCP   5m43s
service/kubernetes         ClusterIP      34.118.224.1     <none>         443/TCP        18h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-world-demo   3/3     3            3           5m44s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-world-demo-8d8c747b5   3         3         3       5m43s
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```

```sh
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ gcloud container clusters get-credentials dev-cluster -
-zone us-central1-b 
Fetching cluster endpoint and auth data.
kubeconfig entry generated for dev-cluster.
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/hello-world-demo-77574f6944-5rf4k   1/1     Running   0          12m
pod/hello-world-demo-77574f6944-92m2w   1/1     Running   0          12m
pod/hello-world-demo-77574f6944-gjjcp   1/1     Running   0          12m

NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
service/hello-world-demo   LoadBalancer   34.118.237.233   104.154.205.83   80:31701/TCP   57m
service/kubernetes         ClusterIP      34.118.224.1     <none>           443/TCP        18h

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-world-demo   3/3     3            3           57m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-world-demo-5f887ff49c   0         0         0       23m
replicaset.apps/hello-world-demo-77574f6944   3         3         3       12m
replicaset.apps/hello-world-demo-d8948b957    0         0         0       57m
student_00_b9ddf527878d@cloudshell:~/HelloWorldNodeJs (qwiklabs-gcp-04-7d4f733ccbf1)$ 
```



You have used Cloud Source Repositories, Cloud Build, and Google Cloud  Deploy to automate the application delivery to GKE and Anthos clusters.  You also learned how to promote and approve releases  to staging and  production clusters.
