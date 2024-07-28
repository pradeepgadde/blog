---

layout: single
title:  "Deploy to Cloud Run for Anthos"
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
# Deploy to Cloud Run for Anthos

- Create a basic JavaScript application and package it as a Docker container.
- Harness Cloud Build to create a Docker image and deploy it into Container Registry.
- Run the container in Google's fully managed Cloud Run.
- Deploy the application to Cloud Run for Anthos.

## Review your Anthos GKE cluster and install Cloud Run for Anthos

In this task, you review and prepare the pre-created Anthos GKE  cluster to execute Cloud Run for Anthos. First, you verify that the GKE cluster is registered in an Anthos Fleet. Second, you check that Anthos Service  Mesh is installed in the cluster as a pre-requisite to installing Cloud  Run for Anthos. Third, you enable and install Cloud Run for Anthos in the  cluster.

1. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**. Notice that a new GKE cluster has been created.
2. Click **Workloads**, and verify that the cluster is running the Anthos Service Mesh components **istio-ingressgateway** and **istiod-asm**.
3. On the **Navigation menu**, click **Anthos > Clusters** and verify that the cluster is registered and appears in the list of **Anthos managed clusters**.
4. In the **Cloud Platform Console**, click **Activate Cloud Shell** ![The Activate Cloud Shell icon](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D). If prompted, click **Continue**.

In Cloud Shell, set the Zone environment variable:

In Cloud Shell, initialize the environment variables:

Get the credentials for your **gke** GKE cluster:

Enable Cloud Run for Anthos on your project:

Install Cloud Run for Anthos on your cluster:

Enabling Cloud Run for Anthos takes up to 15 minute; 

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-6d1e80f21df7.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ C1_ZONE=us-central1-f
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ export PROJECT_ID=$(gcloud config get-value project)
export C1_NAME="gke"
Your active configuration is: [cloudshell-5954]
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ gcloud container clusters get-credentials $C1_NAME --zone $C1_ZONE --project $PROJECT_ID
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ gcloud container fleet cloudrun enable --project=$PROJECT_ID
Enabling service [appdevelopmentexperience.googleapis.com] on project [qwiklabs-gcp-00-6d1e80f21df7]...
Operation "operations/acat.p2-337614957402-a5139fcc-9cec-430c-b7b4-569c142bcbbe" finished successfully.
Waiting for Feature CloudRun to be created...done.                                                                                                                                 
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ gcloud container fleet cloudrun apply --gke-cluster=$C1_ZONE/$C1_NAME
kubeconfig entry generated for gke.
ERROR: (gcloud.container.fleet.cloudrun.apply) Failed to apply manifest to cluster: error: resource mapping not found for name: "cloud-run" namespace: "" from "STDIN": no matches for kind "CloudRun" in version "operator.run.cloud.google.com/v1alpha1"
ensure CRDs are installed first

student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ gcloud container fleet cloudrun apply --gke-cluster=$C1_ZONE/$C1_NAME
kubeconfig entry generated for gke.
Added CloudRun CR
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

## Create the Node.js application

1. To move Cloud Shell to a new tab, click the **Open in a new window** icon.
2. Create a new **helloworld** folder for the app, and change into it:

```sh
student_00_fe1d56995120@cloudshell:~ (qwiklabs-gcp-00-6d1e80f21df7)$ mkdir helloworld
cd helloworld
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ touch package.json
edit package.json
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ cat package.json 
{
    "name": "knative-helloworld_js",
    "version": "1.0.0",
    "description": "Simple hello world sample in Node",
    "main": "index.js",
    "scripts": {
      "start": "node index.js"
    },
    "author": "",
    "license": "Apache-2.0",
    "dependencies": {
      "express": "^4.17.1"
    }
  }student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

n Node.js, the package.json file is used to define dependencies, key files, and general application information. This package.json file:

- Defines an Apache licensed application named *knative-helloworld_js*.
- Sets the main starting file as *index.js*.
- Defines the script "start" so that it executes the *index.js* file. If you test locally, *npm start* would cause this code to execute. Node is essentially the JavaScript engine that's part of Chrome.
- Sets up a runtime dependency on express version 4.17.1 or higher, if bug fixes are available.

1. Create and edit the application's main execution file *index.js*:

```js
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ touch index.js.js
edit index.js
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ cat index.js 
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  console.log('Hello world received a request.');

  const target = process.env.TARGET || 'World';
  res.send(`Hello ${target}!`);
});

const port = process.env.PORT || 8080;
app.listen(port, () => {
  console.log('Hello world listening on port', port);
});student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

Express.js is a lightweight JavaScript based web server. This code:

- Creates an Express web server instance.
- Sets it up to listen on whatever port is configured in the PORT environment variable, or port 8080 if PORT is empty.
- Waits for requests coming into the server's root (/).

When a request comes to root, the code:

- Logs a message to the console (Cloud Operations).
- Returns the message "Hello *target*," where *target* is either what's configured in the TARGET environment variable, or 'World'.

### Test the application

1. Use the node package manager to load all the application dependencies:

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ npm install

added 64 packages, and audited 65 packages in 2s

12 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

A node_modules folder is created. All the dependencies, and the dependencies of the dependencies, are loaded into that folder.

1. Execute the code:

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ npm start

> knative-helloworld_js@1.0.0 start
> node index.js

Hello world listening on port 8080

```

To close the "Hello World" browser tab and stop Express, click into the Cloud Shell prompt and press CTRL + C.

### Turn the application into a container

1. Create and edit a *Dockerfile* to define a containerized version of the application:

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ touch Dockerfile
edit Dockerfile
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ cat Dockerfile 
FROM node:alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production
COPY . .
CMD [ "npm", "start" ]student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

The code:

- Creates a new container from the [node alpine image](https://hub.docker.com/_/node).
- Sets the *WORKDIR* to */user/scr/app*. Subsequent file commands will be relative to this folder.
- Copies the *package.json* and *package-lock.json* into the *WORKDIR*.
- Just as when you did testing, runs *npm install*, but only installs production dependencies and ignores any dev dependencies that might be listed (none in this case).
- Copies any files from the current folder into the *WORKDIR*.
- When the container is launched, executes the *CMD* *npm start*.

The npm installation generated some files you don't need copied into your container. Create a *.dockerignore* file to tell Docker which files to ignore:

Use Cloud Build to create the Docker image and push it to your project's Container Registry, under the name *helloworld*:

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ touch Dockerfile
edit Dockerfile
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ cat Dockerfile 
FROM node:alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --only=production
COPY . .
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ touch .dockerignore)$ touch .dockerignore
edit .dockerignore
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ export PROJECT=$(gcloud config list --format 'value(core.project)')
gcloud builds submit --tag gcr.io/$PROJECT/helloworld
Creating temporary archive of 519 file(s) totalling 2.1 MiB before compression.
Uploading tarball of [.] to [gs://qwiklabs-gcp-00-6d1e80f21df7_cloudbuild/source/1722200034.08747-a02724afef4a4aa5916dadeee2f73b9e.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/qwiklabs-gcp-00-6d1e80f21df7/locations/global/builds/82a86a2d-2807-4bed-8264-826c710d2bb6].
Logs are available at [ https://console.cloud.google.com/cloud-build/builds/82a86a2d-2807-4bed-8264-826c710d2bb6?project=337614957402 ].
Waiting for build to complete. Polling interval: 1 second(s).
------------------------------------------------------------------------------- REMOTE BUILD OUTPUT --------------------------------------------------------------------------------
starting build "82a86a2d-2807-4bed-8264-826c710d2bb6"

FETCHSOURCE
Fetching storage object: gs://qwiklabs-gcp-00-6d1e80f21df7_cloudbuild/source/1722200034.08747-a02724afef4a4aa5916dadeee2f73b9e.tgz#1722200036150011
Copying gs://qwiklabs-gcp-00-6d1e80f21df7_cloudbuild/source/1722200034.08747-a02724afef4a4aa5916dadeee2f73b9e.tgz#1722200036150011...
/ [1 files][667.1 KiB/667.1 KiB]                                                
Operation completed over 1 objects/667.1 KiB.
BUILD
Already have image (with digest): gcr.io/cloud-builders/docker
Sending build context to Docker daemon  37.38kB
Step 1/6 : FROM node:alpine
alpine: Pulling from library/node
Digest: sha256:39005f06b2fae765764d6fdf20ad1c4d0890f5ad3e1f39b56a18768334b8ecd6
Status: Downloaded newer image for node:alpine
 5c4cc5767575
Step 2/6 : WORKDIR /usr/src/app
 Running in 861d90f509ce
Removing intermediate container 861d90f509ce
 a8ee2ea9cb25
Step 3/6 : COPY package*.json ./
 e0577a047f62
Step 4/6 : RUN npm install --only=production
 Running in ec11d627fe22
npm warn config only Use `--omit=dev` to omit dev dependencies from the install.

added 64 packages, and audited 65 packages in 3s

12 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Removing intermediate container ec11d627fe22
 6dfb2955bfd9
Step 5/6 : COPY . .
 fdae9181fdf6
Step 6/6 : CMD [ "npm", "start" ]
 Running in 4c25e16d4da9
Removing intermediate container 4c25e16d4da9
 1e040779b6d4
Successfully built 1e040779b6d4
Successfully tagged gcr.io/qwiklabs-gcp-00-6d1e80f21df7/helloworld:latest
PUSH
Pushing gcr.io/qwiklabs-gcp-00-6d1e80f21df7/helloworld
The push refers to repository [gcr.io/qwiklabs-gcp-00-6d1e80f21df7/helloworld]
703572fc5fbf: Preparing
a107bc823cbe: Preparing
5123e6c9506e: Preparing
1656a996f7ee: Preparing
a64fa369054d: Preparing
1a944437090a: Preparing
4b76468bfe06: Preparing
78561cef0761: Preparing
1a944437090a: Waiting
4b76468bfe06: Waiting
78561cef0761: Waiting
a64fa369054d: Layer already exists
1a944437090a: Layer already exists
1656a996f7ee: Pushed
703572fc5fbf: Pushed
4b76468bfe06: Layer already exists
5123e6c9506e: Pushed
a107bc823cbe: Pushed
78561cef0761: Layer already exists
latest: digest: sha256:d368322b7bca9fadf94e69b6c9002e30dc7bd92769621cf8b450e92bd072f44b size: 1992
DONE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ID: 82a86a2d-2807-4bed-8264-826c710d2bb6
CREATE_TIME: 2024-07-28T20:53:56+00:00
DURATION: 29S
SOURCE: gs://qwiklabs-gcp-00-6d1e80f21df7_cloudbuild/source/1722200034.08747-a02724afef4a4aa5916dadeee2f73b9e.tgz
IMAGES: gcr.io/qwiklabs-gcp-00-6d1e80f21df7/helloworld (+1 more)
STATUS: SUCCESS
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

When the process completes, view the image in **Container Registry** in the helloworld folder.

## Deploy helloworld to Cloud Run

1. To enable the Cloud Run service in your project, enter the following code in Cloud Shell:
2. In the Google Cloud console, navigate to **Cloud Run**.
3. Click **Create Service** to create a new service.
4. To populate the **Container image URL** field, click **Select** and choose the latest image from your **helloworld** repository.
5. Leave the **Service name** as **helloworld**.
6. Leave **Region** as **us-central1 (Iowa)**.
7. Scroll down to **Autoscaling**, and set the **Maximum number of instances** to **4**.
8. Under **Authentication**, select **Allow unauthenticated invocations**.
9. Click **Container, Networking, Security**, and investigate the different options to manage your container workload, set up environment variables, or connect your container to a Cloud SQL instance or via a VCP Connector to your private VPC.
10. Click **Create** and wait for the service to come up.

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ gcloud services enable run.googleapis.com
Operation "operations/acf.p2-337614957402-86294438-feef-44f5-be53-9b1ef5e638e1" finished successfully.
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```



1. When an URL is displayed, click it to test the service.
2. To deploy a new revision, click **Edit & Deploy New Version**.
3. Under **Environment variables**, click **Add Variable**.
4. For `Name1` type **ENV**, and for `Value` type your name.
5. Click **Deploy**.
6. When the new service revision finishes deploying and the new version starts receiving 100% of the traffic, re-test your application. What changed?

## Deploy helloworld into Cloud Run for Anthos

1. On the **Navigation menu**, click **Kubernetes Engine > Workloads**. Notice that several workloads have been created in the **knative-serving** namespace to run your Cloud Run for Anthos services.

2. On the **Navigation menu**, click **Anthos > Clusters**, and click on your cluster to verify that Cloud Run for Anthos is enabled.

3. On the **Navigation menu**, click **Kubernetes Engine > Application**. Click **Go to list of services** and click **Create Service** to create a new service.

4. Configure these settings for **Service settings**. Note that the gke might take a few minutes to be ready before you can schedule Cloud Run workloads

5. | Property                      | Value           |
   | ----------------------------- | --------------- |
   | Available Anthos GKE clusters | *gke* selected  |
   | Namespace                     | `default`       |
   | Service name                  | `helloworld-gk` |

1. Click **Next**, and then paste your **helloworld** image URL(Example URL: gcr.io/qwiklabs-gcp-00-28785963622a/helloworld).
2. Click **Next**, and then select **External**. This invokes the service through the internet.
3. Click **Create**.
4. After the service deploys, the generated URL is set to the `nip.io` base domain. This allows you to test your applications immediately.
5. For production, you must map a custom domain, which is also required to enable HTTPS and use automatic TLS certificates.
6. Click on the URL to open your application.

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ curl http://helloworld-gke.default.kuberun.35.226.162.246.nip.io/
Hello World!student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```



Congratulations! You have created and deployed a Node.js application to fully managed Cloud Run.

## History

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ history 
    1  C1_ZONE=us-central1-f
    2  export PROJECT_ID=$(gcloud config get-value project)
    3  export C1_NAME="gke"
    4  gcloud container clusters get-credentials $C1_NAME --zone $C1_ZONE --project $PROJECT_ID
    5  gcloud container fleet cloudrun enable --project=$PROJECT_ID
    6  gcloud container fleet cloudrun apply --gke-cluster=$C1_ZONE/$C1_NAME
    7  mkdir helloworld
    8  cd helloworld
    9  touch package.json
   10  edit package.json
   11  cat package.json 
   12  touch index.js
   13  edit index.js
   14  cat index.js 
   15  Test the application
   16  npm install
   17  npm start
   18  touch Dockerfile
   19  edit Dockerfile
   20  cat Dockerfile 
   21  touch .dockerignore
   22  edit .dockerignore
   23  export PROJECT=$(gcloud config list --format 'value(core.project)')
   24  gcloud builds submit --tag gcr.io/$PROJECT/helloworld
   25  gcloud services enable run.googleapis.com
   26  curl http://helloworld-gke.default.kuberun.35.226.162.246.nip.io/
   27  history 
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

```sh
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ kubectl get nodes
NAME                               STATUS   ROLES    AGE   VERSION
gke-gke-pool-12345-39b47fcd-l530   Ready    <none>   19h   v1.29.6-gke.1038001
gke-gke-pool-12345-39b47fcd-l9wv   Ready    <none>   19h   v1.29.6-gke.1038001
gke-gke-pool-12345-39b47fcd-qdb7   Ready    <none>   19h   v1.29.6-gke.1038001
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ kubectl get all
NAME                                                      READY   STATUS    RESTARTS   AGE
pod/helloworld-gke-00001-zuz-deployment-cb4bcd774-qrgq4   3/3     Running   0          74s

NAME                                       TYPE           CLUSTER-IP       EXTERNAL-IP                                            PORT(S)                                     AGE
service/helloworld-gke                     ExternalName   <none>           knative-local-gateway.istio-system.svc.cluster.local   80/TCP                                      3m14s
service/helloworld-gke-00001-zuz           ClusterIP      34.118.233.25    <none>                                                 80/TCP,443/TCP                              3m24s
service/helloworld-gke-00001-zuz-private   ClusterIP      34.118.239.248   <none>                                                 80/TCP,443/TCP,9090/TCP,9091/TCP,8022/TCP   3m24s
service/kubernetes                         ClusterIP      34.118.224.1     <none>                                                 443/TCP                                     19h

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld-gke-00001-zuz-deployment   1/1     1            1           3m25s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-gke-00001-zuz-deployment-cb4bcd774   1         1         1       3m25s

NAME                                               READY   REASON
cloudrun.operator.run.cloud.google.com/cloud-run   True    
student_00_fe1d56995120@cloudshell:~/helloworld (qwiklabs-gcp-00-6d1e80f21df7)$ 
```

