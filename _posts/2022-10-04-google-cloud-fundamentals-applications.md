---

layout: single
title:  "Getting Started with Google Cloud—Applications"
date:   2022-10-04 08:59:04 +0530
categories: Cloud
tags: GCP
show_date: true
classes: wide
header:
  teaser: /assets/images/gcp.png
author:
  name     : "Google Cloud Platform"
  avatar   : "/assets/images/gcp.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---


## Applications in the Cloud

- App Engine
- App Engine evrionments
- Google Cloud API management tool
- Cloud Run
- Hello Cloud Run Lab

Cloud Run is a managed compute platform that enables you to run stateless containers that are invocable via HTTP requests. Cloud Run is serverless: it abstracts away all infrastructure management, so you can focus on what matters most — building great applications.

Cloud Run is built from Knative, letting you choose to run your containers either fully managed with Cloud Run, or in your Google Kubernetes Engine cluster with Cloud Run on GKE.

The goal of this lab is for you to build a simple containerized application image and deploy it to Cloud Run.

- Enable the Cloud Run API.

- Create a simple Node.js application that can be deployed as a serverless, stateless container.

- Containerize your application and upload to Container Registry (now called "Artifact Registry.")

- Deploy a containerized application on Cloud Run.

- Delete unneeded images to avoid incurring extra storage charges.


## Activate Google Cloud Shell

Google Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud.

Google Cloud Shell provides command-line access to your Google Cloud resources.

In Cloud console, on the top right toolbar, click the Open Cloud Shell button.
Click Continue.

It takes a few moments to provision and connect to the environment. When you are connected, you are already authenticated, and the project is set to your PROJECT_ID. 

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-41.png)



`gcloud` is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

You can list the active account name with this command:

```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-01-da96e1e7e410@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$
```

You can list the project ID with this command:

```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ gcloud config list project
[core]
project = qwiklabs-gcp-01-fe91867477e4

Your active configuration is: [cloudshell-24308]
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$
```

From Cloud Shell, enable the Cloud Run API :

```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ gcloud services enable run.googleapis.com
Operation "operations/acf.p2-1093784407584-bea23088-1531-4300-9496-8a9265591c53" finished successfully.
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$
```

Set the compute region:
```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ gcloud config set compute/region us-central1
Updated property [compute/region].
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$
```
Create a LOCATION environment variable:
```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ LOCATION="us-central1"
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$
```

build a simple express-based NodeJS application which responds to HTTP requests.

In Cloud Shell create a new directory named `helloworld`, then move your view into that directory:

```sh
student_01_da96e1e7e410@cloudshell:~ (qwiklabs-gcp-01-fe91867477e4)$ mkdir helloworld && cd helloworld
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ nano package.json
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ cat package.json
{
  "name": "helloworld",
  "description": "Simple hello world sample in Node",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "Google LLC",
  "license": "Apache-2.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```



```js
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ cat index.js
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;
app.get('/', (req, res) => {
  const name = process.env.NAME || 'World';
  res.send(`Hello ${name}!`);
});
app.listen(port, () => {
  console.log(`helloworld: listening on port ${port}`);
});
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

This code creates a basic web server that listens on the port defined by the `PORT` environment variable. Your app is now finished and ready to be containerized and uploaded to Container Registry.



To containerize the sample app, create a new file named `Dockerfile` in the same directory as the source files, and add the following content:

```dockerfile
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ cat Dockerfile
# Use the official lightweight Node.js 12 image.
# https://hub.docker.com/_/node
FROM node:12-slim
# Create and change to the app directory.
WORKDIR /usr/src/app
# Copy application dependency manifests to the container image.
# A wildcard is used to ensure copying both package.json AND package-lock.json (when available).
# Copying this first prevents re-running npm install on every code change.
COPY package*.json ./
# Install production dependencies.
# If you add a package-lock.json, speed your build by switching to 'npm ci'.
# RUN npm ci --only=production
RUN npm install --only=production
# Copy local code to the container image.
COPY . ./
# Run the web service on container startup.
CMD [ "npm", "start" ]
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

build your container image using Cloud Build by running the following command from the directory containing the `Dockerfile.` (Note the $GOOGLE_CLOUD_PROJECT environmental variable in the command, which contains your lab's Project ID):

```sh

student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
Creating temporary tarball archive of 3 file(s) totalling 1.3 KiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-01-fe91867477e4_cloudbuild/source/1664861676.724426-71e9b0e996f841a6828dc40489a1cc2d.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-01-fe91867477e4/locations/global/builds/fecbfc09-320e-47e6-9b0c-3c6488043f3d].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/fecbfc09-320e-47e6-9b0c-3c6488043f3d?project=1093784407584 ].
------------------------------------------------------------- REMOTE BUILD OUTPUT -------------------------------------------------------------
starting build "fecbfc09-320e-47e6-9b0c-3c6488043f3d"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-01-fe91867477e4_cloudbuild/source/1664861676.724426-71e9b0e996f841a6828dc40489a1cc2d.tgz#1664861678606918
Copying gs://qwiklabs-gcp-01-fe91867477e4_cloudbuild/source/1664861676.724426-71e9b0e996f841a6828dc40489a1cc2d.tgz#1664861678606918...
/ [1 files][  988.0 B/  988.0 B]
Operation completed over 1 objects/988.0 B.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  4.608kB
Step 1/6 : FROM node:12-slim
12-slim: Pulling from library/node
8bd3f5a20b90: Pulling fs layer
3a665e454db5: Pulling fs layer
b5b0e172ab63: Pulling fs layer
45013ee4878f: Pulling fs layer
42a448233138: Pulling fs layer
45013ee4878f: Waiting
42a448233138: Waiting
3a665e454db5: Verifying Checksum
3a665e454db5: Download complete
45013ee4878f: Verifying Checksum
45013ee4878f: Download complete
42a448233138: Verifying Checksum
42a448233138: Download complete
8bd3f5a20b90: Verifying Checksum
8bd3f5a20b90: Download complete
b5b0e172ab63: Verifying Checksum
b5b0e172ab63: Download complete
8bd3f5a20b90: Pull complete
3a665e454db5: Pull complete
b5b0e172ab63: Pull complete
45013ee4878f: Pull complete
42a448233138: Pull complete
Digest: sha256:c28ca4124ac84ff1e72ee68abfe70aa4cc86f499c1ef20408dc2d35af1058ac1
Status: Downloaded newer image for node:12-slim
 9b792ac6810e
Step 2/6 : WORKDIR /usr/src/app
 Running in d6b981447f60
Removing intermediate container d6b981447f60
 cdb616d7bf67
Step 3/6 : COPY package*.json ./
 b6e62ec12ac9
Step 4/6 : RUN npm install --only=production
 Running in 3a345391e3fc
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

added 57 packages from 42 contributors and audited 57 packages in 2.791s

7 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Removing intermediate container 3a345391e3fc
 bf8960147cfb
Step 5/6 : COPY . ./
 0a9f3280e7cb
Step 6/6 : CMD [ "npm", "start" ]
 Running in 78ea8e03b248
Removing intermediate container 78ea8e03b248
 398d13b5a756
Successfully built 398d13b5a756
Successfully tagged gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld:latest
PUSH
Pushing gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld
The push refers to repository [gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld]
c7b2e6b1fc3f: Preparing
d57d117657e5: Preparing
910cd5b1e051: Preparing
25c90b372d60: Preparing
a2300a3fbe99: Preparing
7722feb318fa: Preparing
56ff4c5413ca: Preparing
3d1236fa695c: Preparing
dc8ea258c7d2: Preparing
7722feb318fa: Waiting
56ff4c5413ca: Waiting
3d1236fa695c: Waiting
dc8ea258c7d2: Waiting
a2300a3fbe99: Layer already exists
7722feb318fa: Layer already exists
56ff4c5413ca: Layer already exists
3d1236fa695c: Layer already exists
25c90b372d60: Pushed
910cd5b1e051: Pushed
dc8ea258c7d2: Layer already exists
c7b2e6b1fc3f: Pushed
d57d117657e5: Pushed
latest: digest: sha256:f33be54c5e79816f86ae4653371bbc38407f31e123f04631359d3f08c9e4435a size: 2199
DONE
-----------------------------------------------------------------------------------------------------------------------------------------------
ID: fecbfc09-320e-47e6-9b0c-3c6488043f3d
CREATE_TIME: 2022-10-04T05:34:39+00:00
DURATION: 30S
SOURCE: gs://qwiklabs-gcp-01-fe91867477e4_cloudbuild/source/1664861676.724426-71e9b0e996f841a6828dc40489a1cc2d.tgz
IMAGES: gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld (+1 more)
STATUS: SUCCESS
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

Cloud Build is a service that executes your builds on GCP. It executes a series of build steps, where each build step is run in a Docker container to produce your application container (or other artifacts) and push it to Cloud Registry, all in one command.

Once pushed to the registry, you will see a SUCCESS message containing the image name (`gcr.io/[PROJECT-ID]/helloworld`). The image is stored in Artifact Registry and can be re-used if desired.

List all the container images associated with your current project using this command:

```sh
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud container images list
NAME: gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld
Only listing images in gcr.io/qwiklabs-gcp-01-fe91867477e4. Use --repository to list images in other repositories.
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

To run and test the application locally from Cloud Shell, start it using this standard `docker` command:

```sh
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ docker run -d -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
Unable to find image 'gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld:latest' locally
latest: Pulling from qwiklabs-gcp-01-fe91867477e4/helloworld
8bd3f5a20b90: Pull complete
3a665e454db5: Pull complete
b5b0e172ab63: Pull complete
45013ee4878f: Pull complete
42a448233138: Pull complete
45d5b24f7d56: Pull complete
2b7391e0b31b: Pull complete
8dcfcd126bc9: Pull complete
7c46bd99a445: Pull complete
Digest: sha256:f33be54c5e79816f86ae4653371bbc38407f31e123f04631359d3f08c9e4435a
Status: Downloaded newer image for gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld:latest
3ac30bf957f1a5545b27bc06982b5d42644cd32128dd20dccc0e58d6aeeaa23d
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

In the Cloud Shell window, click on **Web preview** and select **Preview on port 8080**.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-42.png)





This should open a browser window showing the "Hello World!" message. You could also simply use `curl localhost:8080`.





Deploying your containerized application to Cloud Run is done using the following command adding your Project-ID:

```sh
   student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud run deploy --image gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld --allow-unauthenticated --region=$LOCATION
   Service name (helloworld):
   Deploying container to Cloud Run service [helloworld] in project [qwiklabs-gcp-01-fe91867477e4] region [us-central1]
   /  Deploying new service... Initializing project for the current region.
     /  Creating Revision...
     .  Routing traffic...
     OK Setting IAM Policy...
   
```

```sh
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud run deploy --image gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld --allow-unauthenticated --region=$LOCATION
Service name (helloworld):
Deploying container to Cloud Run service [helloworld] in project [qwiklabs-gcp-01-fe91867477e4] region [us-central1]
OK Deploying new service... Done.                                                        
  OK Creating Revision... Deploying Revision.
  OK Routing traffic...
  OK Setting IAM Policy...
Done.
Service [helloworld] revision [helloworld-00001-wof] has been deployed and is serving 100 percent of traffic.
Service URL: https://helloworld-fblfbzsa7q-uc.a.run.app
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

You can now visit your deployed container by opening the service URL in any browser window.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-43.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-44.png)

**Congratulations!** You have just deployed an application packaged in a container image to Cloud Run. Cloud Run automatically and horizontally scales your container image to handle the received requests, then scales down when demand decreases. In your own environment, you only pay for the CPU, memory, and networking consumed during request handling.

While Cloud Run does not charge when the service is not in use, you might still be charged for storing the built container image.

You can either decide to delete your GCP project to avoid incurring charges, which will stop billing for all the resources used within that project, or simply delete your `helloworld` image using this command :

```sh
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud container images delete gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
WARNING: Implicit ":latest" tag specified: gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld
WARNING: Successfully resolved tag to sha256, but it is recommended to use sha256 directly.
Digests:
- gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld@sha256:f33be54c5e79816f86ae4653371bbc38407f31e123f04631359d3f08c9e4435a
  Associated tags:
 - latest
Tags:
- gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld:latest
This operation will delete the tags and images identified by the digests above.

Do you want to continue (Y/n)?  y

Deleted [gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld:latest].
Deleted [gcr.io/qwiklabs-gcp-01-fe91867477e4/helloworld@sha256:f33be54c5e79816f86ae4653371bbc38407f31e123f04631359d3f08c9e4435a].
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

To delete the Cloud Run service, use this command :



```sh
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$ gcloud run services delete helloworld --region=us-central1
Service [helloworld] will be deleted.

Do you want to continue (Y/n)?  y

Deleting [helloworld]...done.                              
Deleted service [helloworld].
student_01_da96e1e7e410@cloudshell:~/helloworld (qwiklabs-gcp-01-fe91867477e4)$
```

