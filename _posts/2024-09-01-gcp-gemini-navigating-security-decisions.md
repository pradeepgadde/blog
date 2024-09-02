---

layout: single
title:  "Navigate Security Decisions with Gemini"
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

# Navigate Security Decisions with Gemini

Learn how to identify and remediate security misconfigurations in your Google Cloud environment using Security Command Center's Gemini features.


- Enable Gemini in a Google Cloud project
- Deploy example workloads into an environment in Google Cloud
- Identify security misconfigurations with Gemini
- Remediate security misconfigurations with Gemini

You are a security engineer at an ecommerce company where managed Kubernetes clusters are regularly deployed. You need a way to see if there are any misconfigurations, and you want quick instructions to help fix those issues in your cloud environment.

## Enable Gemini

You will first enable Gemini in your Google Cloud project and configure the necessary permissions for your Google Cloud Qwiklabs user account.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-d5cb75ca7975.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ PROJECT_ID=$(gcloud config get-value project)
REGION=us-east4
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
Your active configuration is: [cloudshell-22973]
PROJECT_ID=qwiklabs-gcp-03-d5cb75ca7975
REGION=us-east4
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ USER=$(gcloud config get-value account 2> /dev/null)
echo "USER=${USER}"
USER=student-01-ad3293e8fd4f@qwiklabs.net
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
Operation "operations/acat.p2-454355847231-3ea7899a-428d-4344-84d7-25c85519579e" finished successfully.
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
Updated IAM policy for project [qwiklabs-gcp-03-d5cb75ca7975].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:454355847231@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-454355847231@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.orgFirewallPolicyAdmin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.orgSecurityPolicyAdmin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.publicIpAdmin
- members:
  - serviceAccount:service-454355847231@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-454355847231@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:454355847231-compute@developer.gserviceaccount.com
  - serviceAccount:454355847231@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-454355847231@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-454355847231@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/workstations.networkAdmin
etag: BwYhKnZEFHQ=
version: 1
Updated IAM policy for project [qwiklabs-gcp-03-d5cb75ca7975].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/cloudaicompanion.user
- members:
  - serviceAccount:454355847231@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-454355847231@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.orgFirewallPolicyAdmin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.orgSecurityPolicyAdmin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/compute.publicIpAdmin
- members:
  - serviceAccount:service-454355847231@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-454355847231@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:454355847231-compute@developer.gserviceaccount.com
  - serviceAccount:454355847231@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-454355847231@gcp-sa-notebooks.iam.gserviceaccount.com
  role: roles/notebooks.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/owner
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/serviceusage.serviceUsageViewer
- members:
  - serviceAccount:qwiklabs-gcp-03-d5cb75ca7975@qwiklabs-gcp-03-d5cb75ca7975.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/viewer
- members:
  - serviceAccount:service-454355847231@gcp-sa-websecurityscanner.iam.gserviceaccount.com
  role: roles/websecurityscanner.serviceAgent
- members:
  - user:student-01-ad3293e8fd4f@qwiklabs.net
  role: roles/workstations.networkAdmin
etag: BwYhKnZlZhY=
version: 1
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ 
```

## Create a GKE cluster and deploy a web app

You will now be creating a Google Kubernetes Engine (GKE) cluster running a handful of microservices.

```sh
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ gcloud container clusters create test --region=us-east4 --num-nodes=1
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: The Kubelet readonly port (10255) is now deprecated. Please update your workloads to use the recommended alternatives. See https://cloud.google.com/kubernetes-engine/docs/how-to/disable-kubelet-readonly-port for ways to check usage and for migration instructions.
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster test in us-east4... Cluster is being health-checked (master is healthy)...done.                                                                                   
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-d5cb75ca7975/zones/us-east4/clusters/test].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east4/test?project=qwiklabs-gcp-03-d5cb75ca7975
kubeconfig entry generated for test.
NAME: test
LOCATION: us-east4
MASTER_VERSION: 1.29.7-gke.1104000
MASTER_IP: 34.48.147.154
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.29.7-gke.1104000
NUM_NODES: 3
STATUS: RUNNING
student_01_ad3293e8fd4f@cloudshell:~ (qwiklabs-gcp-03-d5cb75ca7975)$ git clone https://github.com/GoogleCloudPlatform/microservices-demo && cd microservices-demo
Cloning into 'microservices-demo'...
remote: Enumerating objects: 17691, done.
remote: Counting objects: 100% (221/221), done.
remote: Compressing objects: 100% (136/136), done.
remote: Total 17691 (delta 138), reused 156 (delta 81), pack-reused 17470 (from 1)
Receiving objects: 100% (17691/17691), 34.13 MiB | 31.54 MiB/s, done.
Resolving deltas: 100% (13566/13566), done.
student_01_ad3293e8fd4f@cloudshell:~/microservices-demo (qwiklabs-gcp-03-d5cb75ca7975)$ kubectl apply -f ./release/kubernetes-manifests.yaml
deployment.apps/currencyservice created
service/currencyservice created
serviceaccount/currencyservice created
deployment.apps/loadgenerator created
serviceaccount/loadgenerator created
deployment.apps/productcatalogservice created
service/productcatalogservice created
serviceaccount/productcatalogservice created
deployment.apps/checkoutservice created
service/checkoutservice created
serviceaccount/checkoutservice created
deployment.apps/shippingservice created
service/shippingservice created
serviceaccount/shippingservice created
deployment.apps/cartservice created
service/cartservice created
serviceaccount/cartservice created
deployment.apps/redis-cart created
service/redis-cart created
deployment.apps/emailservice created
service/emailservice created
serviceaccount/emailservice created
deployment.apps/paymentservice created
service/paymentservice created
serviceaccount/paymentservice created
deployment.apps/frontend created
service/frontend created
service/frontend-external created
serviceaccount/frontend created
deployment.apps/recommendationservice created
service/recommendationservice created
serviceaccount/recommendationservice created
deployment.apps/adservice created
service/adservice created
serviceaccount/adservice created
student_01_ad3293e8fd4f@cloudshell:~/microservices-demo (qwiklabs-gcp-03-d5cb75ca7975)$ kubectl get service frontend-external | awk '{print $4}'
EXTERNAL-IP
<pending>
student_01_ad3293e8fd4f@cloudshell:~/microservices-demo (qwiklabs-gcp-03-d5cb75ca7975)$ kubectl get service frontend-external | awk '{print $4}'
EXTERNAL-IP
<pending>
student_01_ad3293e8fd4f@cloudshell:~/microservices-demo (qwiklabs-gcp-03-d5cb75ca7975)$ kubectl get service frontend-external | awk '{print $4}'
EXTERNAL-IP
34.48.104.145
student_01_ad3293e8fd4f@cloudshell:~/microservices-demo (qwiklabs-gcp-03-d5cb75ca7975)$ 
```

## Identify security misconfigurations with Gemini

Now that you have an existing GKE cluster running an ecommerce app,  you will identify areas where you can improve your security posture with Gemini.

1. Return to your tab with the Google Cloud console.
2. **Refresh the Google Cloud console page.**
3. Minimize the Cloud Shell pane.
4. Click on the Gemini icon (![Gemini icon](https://cdn.qwiklabs.com/8enFLY%2FtuyohEbT5f1NbBaC%2Fa%2Be7cuSzbbq1l19r%2B3Q%3D)) in the top-right corner of the Google Cloud console toolbar.
5. Click **Start Chatting**.
6. Enter the following prompt:

```sh
What services in Google Cloud can help me identify areas to improve security for a set of microservices running in a GKE cluster?
```

In this scenario, you decide that Security Command Center sounds like the right place to start.

1. Open the Navigation menu and select **Security > Risk Overview**.

With multiple visualizations discussing vulnerabilities, you want to  ask Gemini to help you quickly understand what is classified as a  vulnerability in Security Command Center.

1. Enter the following prompt

```sh
How does Security Command Center define a vulnerability?
```

1. Click **Findings** from the sidebar of the Google Cloud console.
2. To see the findings for your GKE cluster, find the "Quick Filters" section and select **Google container cluster** under Resource Type.

You should see a number of Medium severity findings.

1. Click on the **Cluster secrets encryption disabled** finding.

At the top of this finding panel, you should see a section titled "Description" with a summary similar to the following:

1. Exit out of this information panel.
2. Next, click on the **Over privileged account** finding and view the summary in the description which is similar to the following:
3. Exit out of this information panel.
4. Finally, click on the **Master authorized networks disabled** finding. You should see a description similar to the following:

Now that you have an understanding of some of your environment's  security misconfigurations, you will leverage Gemini's recommendations  to remediate them.



## Remediate a security misconfiguration

Now that you have had the opportunity to review multiple areas to  begin improving the security of your GKE cluster, you will remediate the "Master authorized networks disabled" finding using the instructions  provided by Gemini's summary in Security Command Center.

1. Open the Navigation menu and select **Kubernetes Engine > Clusters**.
2. Click the name of the cluster you created earlier ("test").
3. Scroll down to the **Networking** section.
4. Click on the pencil icon next to **Control plane authorized networks**.
5. Check the box next to **Enable control plane authorized networks**.
6. Check the box underneath **Allow access through Google Cloud public IP addresses**.
7. Click **Save changes**.
8. Click **Clusters** in the sidebar.

You will see cluster being updated, shown by a spinning wheel or half green circle icon in the Status section.

1. While the network policy updates for the control plane take place, open Gemini from the Google Cloud console toolbar.
2. Enter the following prompt:

```sh
How do control plane authorized networks work in GKE?
```

*Control plane authorized networks allow you to restrict access to  the control plane of your GKE cluster. By default, the control plane is  accessible from any address. However, you can add one or more authorized networks to restrict access to only those networks.* 

 *To add an authorized network, go to the Google Kubernetes Engine  page in the Google Cloud console. Click the name of the cluster you want to modify. Under Networking, in the Control plane authorized networks  field, click edit Edit control plane authorized networks...*

Applying the updated control plane policy may take a few minutes to take effect.

Once you see a green checkmark in the **Status** column next to your cluster, you have successfully updated your cluster configuration.