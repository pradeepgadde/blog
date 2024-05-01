---

layout: single
title:  "Provision Cloud Infrastructure with Gemini"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/Google_Gemini_logo.png
  og_image: /assets/images/Google_Gemini_logo.png
  teaser: /assets/images/gemini.webp
author:
  name     : "Gemini"
  avatar   : "/assets/images/gemini.webp"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Provision Cloud Infrastructure with Gemini

Gemini for Google Cloud is an always-on AI collaborator that provides help to users of all skill levels where they need it.

Enable Gemini
Explore various Google services that you can use to deploy an app to GKE by asking Gemini context-based questions.
Prompt Gemini to provide commands that you can use to deploy a basic app to a GKE cluster.
Create, explore, and modify the GKE cluster by using Gemini to explain and generate the shell commands.

## Enable Gemini
You will first enable Gemini in your Google Cloud project and configure the necessary permissions for your Google Cloud Qwiklabs user account.
Enable the Cloud AI Companion API for Gemini:
To use Gemini, grant the necessary IAM roles to your Google Cloud Qwiklabs user account:
```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-03762c2d44fe.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ PROJECT_ID=$(gcloud config get-value project)
REGION=us-west1
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
Your active configuration is: [cloudshell-4771]
PROJECT_ID=qwiklabs-gcp-01-03762c2d44fe
REGION=us-west1
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ USER=$(gcloud config get-value account 2> /dev/null)
echo "USER=${USER}"
USER=student-01-3c1bb184aaad@qwiklabs.net
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
Operation "operations/acat.p2-730940802334-37539881-5066-4e72-9bce-6dbd703949e3" finished successfully.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
Updated IAM policy for project [qwiklabs-gcp-01-03762c2d44fe].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:730940802334@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-730940802334@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.orgFirewallPolicyAdmin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.orgSecurityPolicyAdmin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.publicIpAdmin
- members:
  - serviceAccount:service-730940802334@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-730940802334@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:730940802334-compute@developer.gserviceaccount.com
  - serviceAccount:730940802334@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-730940802334@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-730940802334@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/workstations.networkAdmin
etag: BwYXXxHk6t0=
version: 1
Updated IAM policy for project [qwiklabs-gcp-01-03762c2d44fe].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:730940802334@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-730940802334@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.orgFirewallPolicyAdmin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.orgSecurityPolicyAdmin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/compute.publicIpAdmin
- members:
  - serviceAccount:service-730940802334@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-730940802334@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:730940802334-compute@developer.gserviceaccount.com
  - serviceAccount:730940802334@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-730940802334@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/owner
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/serviceusage.serviceUsageViewer
- members:
  - serviceAccount:qwiklabs-gcp-01-03762c2d44fe@qwiklabs-gcp-01-03762c2d44fe.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-730940802334@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - user:student-01-3c1bb184aaad@qwiklabs.net
  role: roles/workstations.networkAdmin
etag: BwYXXxILSjk=
version: 1
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

Adding these roles lets the user use Gemini assistance.

## Deploy GKE clusters

In this task, you use the Google Cloud console and Cloud Shell to deploy GKE clusters.

For the following example, consider that you are an infrastructure  admin who is responsible for setting up infrastructure for a team  developing a web application. The organization at large has standardized on containers and Kubernetes, so the team wants to understand how to  run their web application on Kubernetes in Google Cloud. The team also  wants to understand how to do so with as little infrastructure  management overhead as possible.

In this case, you have the existing GKE cluster running a set of  microservices that make up an ecommerce web app. With Gemini assistance, you will now identify areas to improve.



##  Welcome to Gemini for Cloud console 

 Get help where you need it with a conversational assistant that answers your questions about Google Cloud, reviews code snippets, and  troubleshoots issues quickly. 

```sh
 Gemini is an AI-powered collaborator to help you get more done faster. Get answers to your questions about how to get started with a Cloud solution, strategies for optimizing resources, or using the gcloud CLI to manage Google Cloud.

In addition to general knowledge about Google Cloud, it also has some awareness of your context, like your project and console page. 
```

Q: How can I run Kubernetes on Google Cloud without having to own management of nodes and the control plane?

A:

To run Kubernetes on Google Cloud without having to own management of  nodes and the control plane, you can use Google Kubernetes Engine (GKE)  Autopilot. GKE Autopilot is a managed Kubernetes service that takes care of all the operational tasks associated with running Kubernetes,  including node and control plane management, upgrades, and security  patching. This allows you to focus on developing and deploying your  applications without having to worry about the underlying  infrastructure. 

 To create a GKE Autopilot cluster, you can use the Google Cloud  console, the gcloud command-line tool, or the Kubernetes Engine API.  Once your cluster is created, you can deploy your applications to it  using the same tools and techniques that you would use with any other  Kubernetes cluster. 

 Here are the steps on how to create a GKE Autopilot cluster using the Google Cloud console: 

 Go to the  [ Kubernetes Engine page ](https://console.cloud.google.com/kubernetes/clusters?chat=true&authuser=1&cloudshell=true&project=qwiklabs-gcp-01-03762c2d44fe)  in the Google Cloud console.  Click the   **Create cluster**   button.  In the   **Cluster name**   field, enter a name for your cluster.  In the   **Location**   field, select the region where you want to create your cluster.  In the   **Node pool**   section, select the   **Autopilot**   option.  Click the   **Create**   button. 

 Once your cluster is created, you can deploy your applications to it using the  kubectl  command-line tool. For example, to deploy a simple Nginx application, you can run the following command: 

```sh
kubectl create deployment nginx --image=nginx

```

 Generated code may be subject to license restrictions not shown here. Use code with care. [ Learn more ](https://cloud.google.com/duet-ai/docs/discover/code-generation-source-citation?authuser=1#how_helps_with_coding)

 This will create a deployment object that will deploy an Nginx container to your cluster. You can then use the  kubectl get pods  command to check the status of your deployment. 

 For more information on GKE Autopilot, please refer to the  [GKE Autopilot documentation ](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview?hl=en_US)

Source:

1. [https://cloud.google.com/duet-ai/docs/use-cases/infrastructure-admin-duet-ai ](https://cloud.google.com/duet-ai/docs/use-cases/infrastructure-admin-duet-ai?hl=en_US)

## Deploy a GKE Autopilot cluster

In this task, you prompt Gemini on how to deploy a GKE Autopliot cluster.

1. Enter the following prompt:

   Q: How do I create a GKE Autopilot cluster?

   A:  To create a GKE Autopilot cluster, you can use the Google Cloud  console, the gcloud command-line tool, or the Kubernetes Engine API. 

    **Using the Google Cloud console** 

    Go to the  [ Kubernetes Engine page ](https://console.cloud.google.com/kubernetes/clusters?chat=true&authuser=1&cloudshell=true&project=qwiklabs-gcp-01-03762c2d44fe)  in the Google Cloud console.  Click the   **Create cluster**   button.  In the   **Cluster name**   field, enter a name for your cluster.  In the   **Location**   field, select the region where you want to create your cluster.  In the   **Node pool**   section, select the   **Autopilot**   option.  Click the   **Create**   button. 

    **Using the gcloud command-line tool** 

   ```sh
   gcloud container clusters create CLUSTER_NAME \
   --node-pool=default-pool \
   --num-nodes=3 \
   --enable-autopilot \
   --location=europe-central2-b
   
   ```

    Generated code may be subject to license restrictions not shown here. Use code with care. [ Learn more ](https://cloud.google.com/duet-ai/docs/discover/code-generation-source-citation?authuser=1#how_helps_with_coding)

    **Using the Kubernetes Engine API** 

   ```yaml
   apiVersion: v1
   kind: Cluster
   metadata:
     name: my-cluster
     location: europe-central2-b
   nodePools:
   - name: default-pool
     config:
       autopilot: {}
   ```

    Generated code may be subject to license restrictions not shown here. Use code with care. [ Learn more ](https://cloud.google.com/duet-ai/docs/discover/code-generation-source-citation?authuser=1#how_helps_with_coding)

    Once your cluster is created, you can deploy your applications to it  using the same tools and techniques that you would use with any other  Kubernetes cluster. 

    For more information on GKE Autopilot, please refer to the  [GKE Autopilot documentation ](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview?hl=en_US)

Q: What is the command to create a GKE Autopilot cluster in my current project and in the us-west1 region named gemini-demo using the gcloud CLI?

A: The command to create a GKE Autopilot cluster in your current project and in the  us-west1  region named  gemini-demo  using the  gcloud  CLI is: 

```sh
gcloud container clusters create gemini-demo \
--node-pool=default-pool \
--num-nodes=3 \
--enable-autopilot \
--location=us-west1

```

 This command will create a cluster with the following configuration: 

 Name:  gemini-demo Region:  us-west1 Node pool:  default-pool  with 3 nodes  Autopilot: enabled 

 Once the cluster is created, you can deploy your applications to it  using the same tools and techniques that you would use with any other  Kubernetes cluster. 

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ gcloud container clusters create gemini-demo \

--node-pool=default-pool \
--num-nodes=3 \
--enable-autopilot \
--location=us-west1
ERROR: (gcloud.container.clusters.create) unrecognized arguments:
--node-pool=default-pool (did you mean '--max-nodes-per-pool'?)
--enable-autopilot (did you mean '--enable-autorepair'?)
To search the help text of gcloud commands, run:
gcloud help -- SEARCH_TERMS
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 

```
> This seems to be wrong command. 



Here is the correct method of creating autopilot GKE cluster

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ gcloud container clusters create-auto gemini-demo --region us-west1
Creating cluster gemini-demo in us-west1... Cluster is being health-checked (master is healthy)...working...                    
Creating cluster gemini-demo in us-west1... Cluster is being health-checked (master is healthy)...done.                         
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-03762c2d44fe/zones/us-west1/clusters/gemini-demo].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1/gemini-demo?project=qwiklabs-gcp-01-03762c2d44fe
kubeconfig entry generated for gemini-demo.
NAME: gemini-demo
LOCATION: us-west1
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.83.24.75
MACHINE_TYPE: e2-small
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

## Deploy a sample web application

Now that you have created a GKE cluster in Autopilot mode, you now  decide you would like to test running a sample web application similar  to the web application your team will run on this infrastructure.  Internally, you see that you have the following container image from  Google *us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0* bookmarked for testing a containerized web application:

Q: What is the kubectl command to create a deployment called hello-server for the image us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0?

A: 
```sh
kubectl create deployment hello-server --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
```

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ kubectl create deployment hello-server --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
deployment.apps/hello-server created
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

Q: What is the kubectl command to expose this deployment on port 80 and target port 8080 with a load balancer?

A:

```sh
kubectl expose deployment hello-server --port 80 --target-port 8080 --type LoadBalancer
```

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ kubectl expose deployment hello-server --port 80 --target-port 8080 --type LoadBalancer
service/hello-server exposed
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

Q: What is the kubectl command to see the external IP address associated with this load balancer?

A: 

```sh
kubectl get service hello-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ kubectl get service hello-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
34.127.34.64student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ curl 34.127.34.64
Hello, world!
Version: 1.0.0
Hostname: hello-server-545d5cf56d-s9bmb
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

## Summary

```sh
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ history 
    1  PROJECT_ID=$(gcloud config get-value project)
    2  REGION=us-west1
    3  echo "PROJECT_ID=${PROJECT_ID}"
    4  echo "REGION=${REGION}"
    5  USER=$(gcloud config get-value account 2> /dev/null)
    6  echo "USER=${USER}"
    7  gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
    8  gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
    9  gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
   10  gcloud container clusters create gemini-demo --node-pool=default-pool --num-nodes=3 --enable-autopilot --location=us-west1
   11  gcloud container clusters create-auto gemini-demo --region us-west1
   12  kubectl create deployment hello-server --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
   13  kubectl expose deployment hello-server --port 80 --target-port 8080 --type LoadBalancer
   14  kubectl get service hello-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
   15  curl 34.127.34.64
   16  history 
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-01-03762c2d44fe)$ 
```

 The following is a summary of all  gcloud  commands suggested so far: 

```sh
gcloud container clusters create gemini-demo \ --node-pool=default-pool \  --num-nodes=3 \ --enable-autopilot \ --location=us-west1 

This command creates a GKE Autopilot cluster named  gemini-demo  in the  us-west1  region with 3 nodes. 

kubectl create deployment hello-server --image=us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0 

This command creates a deployment called  hello-server  that runs the image  us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0 . 

kubectl expose deployment hello-server --port 80 --target-port 8080 --type LoadBalancer 

This command creates a service called  hello-server  that exposes the deployment  hello-server  on port 80 with a load balancer.

kubectl get service hello-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}' 

This command prints the external IP address of the load balancer associated with the service  hello-server . 
```



