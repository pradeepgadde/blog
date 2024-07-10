---

layout: single
title:  "Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab"
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
# Implement Cloud Security Fundamentals on Google Cloud: Challenge Lab

-  Create a custom security role.
-  Create a service account.
-  Bind IAM security roles to a service account.
-  Create a private Kubernetes Engine cluster in a custom subnet. 
-  Deploy an application to a private Kubernetes Engine cluster 

## Challenge scenario

You have started a new role as a junior member of the security team  for the Orca team in Jooli Inc. Your team is responsible for ensuring  the security of the Cloud infrastructure and services that the company's applications depend on.

You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides to be provided.

### Your challenge

You have been asked to deploy, configure, and test a new Kubernetes  Engine cluster that will be used for application development and  pipeline testing by the Orca development team.

As per the organization's security standards you must ensure that the new Kubernetes Engine cluster is built according to the organization's  most recent security standards and thereby must comply with the  following:

- The cluster must be deployed using a dedicated service account configured with the least privileges required.
- The cluster must be deployed as a Kubernetes Engine private cluster, with the public endpoint disabled, and the master authorized network  set to include only the ip-address of the Orca group's management  jumphost.
- The Kubernetes Engine private cluster must be deployed to the `orca-build-subnet` in the Orca Build VPC.

From a previous project you know that the minimum permissions  required by the service account that is specified for a Kubernetes  Engine cluster is covered by these three built in roles:

- `roles/monitoring.viewer`
- `roles/monitoring.metricWriter`
- `roles/logging.logWriter`

These roles are specified in the Google Kubernetes Engine (GKE)'s Harden your cluster's security guide in the [Use least privilege Google service accounts](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster#use_least_privilege_sa) section.

You must bind the above roles to the service account used by the  cluster as well as a custom role that you must create in order to  provide access to any other services specified by the development team.  Initially you have been told that the development team requires that the service account used by the cluster should have the permissions  necessary to add and update objects in Google Cloud Storage buckets. To  do this you will have to create a new custom IAM role that will provide  the following permissions:

- `storage.buckets.get`
- `storage.objects.get`
- `storage.objects.list`
- `storage.objects.update`
- `storage.objects.create`

Once you have created the new private cluster you must test that it  is correctly configured by connecting to it from the jumphost, `orca-jumphost`, in the management subnet `orca-mgmt-subnet`. As this compute instance is not in the same subnet as the private  cluster you must make sure that the master authorized networks for the  cluster includes the internal ip-address for the instance, and you must  specify the `--internal-ip` flag when retrieving cluster credentials using the `gcloud container clusters get-credentials` command.

All new cloud objects and services that you create should include the "orca-" prefix.

Your final task is to validate that the cluster is working correctly  by deploying a simple application to the cluster to test that management access to the cluster using the `kubectl` tool is working from the `orca-jumphost` compute instance.

## Create a custom security role

Your first task is to create a new custom IAM security role called  that will provide the Google Cloud storage bucket and object  permissions required to be able to create and update storage objects.

## Create a service account

Your second task is to create the dedicated service account that will be used as the service account for your new private cluster. You must  name this account .

## Bind a custom security role to a service account

You must now bind the Cloud Operations logging and monitoring roles  that are required for Kubernetes Engine Cluster service accounts as well as the custom IAM role you created for storage permissions to the  Service Account you created earlier.

```sh
student_02_6781a7ab3969@cloudshell:~ (qwiklabs-gcp-03-25fec291e65f)$ history 
    1  gcloud projects add-iam-policy-binding qwiklabs-gcp-03-25fec291e65f     --member=serviceAccount:orca-private-cluster-932-sa@qwiklabs-gcp-03-25fec291e65f.iam.gserviceaccount.com     --role=roles/monitoring.viewer
    2  gcloud projects add-iam-policy-binding qwiklabs-gcp-03-25fec291e65f     --member=serviceAccount:orca-private-cluster-932-sa@qwiklabs-gcp-03-25fec291e65f.iam.gserviceaccount.com     --role=roles/monitoring.metricWriter
    3  gcloud projects add-iam-policy-binding qwiklabs-gcp-03-25fec291e65f     --member=serviceAccount:orca-private-cluster-932-sa@qwiklabs-gcp-03-25fec291e65f.iam.gserviceaccount.com     --role=roles/logging.logWriter 
    4  gcloud projects add-iam-policy-binding qwiklabs-gcp-03-25fec291e65f     --member=serviceAccount:orca-private-cluster-932-sa@qwiklabs-gcp-03-25fec291e65f.iam.gserviceaccount.com     --role=projects/qwiklabs-gcp-03-25fec291e65f/roles/orca_storage_editor_934
    5  gcloud compute ssh --zone "europe-west4-b" "orca-jumphost" --project "qwiklabs-gcp-03-25fec291e65f"
    6  history 
student_02_6781a7ab3969@cloudshell:~ (qwiklabs-gcp-03-25fec291e65f)$ 
```



## Create and configure a new Kubernetes Engine private cluster

You must now use the service account you have configured when  creating a new Kubernetes Engine private cluster. The new cluster  configuration must include the following:

- The cluster must be called 
- The cluster must be deployed to the subnet `orca-build-subnet`
- The cluster must be configured to use the  service account.
- The private cluster options `enable-master-authorized-networks`, `enable-ip-alias`, `enable-private-nodes`, and `enable-private-endpoint` must be enabled.

Once the cluster is configured you must add the internal ip-address of the `orca-jumphost` compute instance to the master authorized network list.



## Deploy an application to a private Kubernetes Engine cluster

You have a simple test application that can be deployed to any  cluster to quickly test that basic container deployment functionality is working and that basic services can be created and accessed. You must  configure the environment so that you can deploy this simple demo to the new cluster using the jumphost `orca-jumphost`.

Make sure to properly install the `gke-gcloud-auth-plugin` before running any `kubectl` commands.

```sh
student-02-6781a7ab3969@orca-jumphost:~$ history 
    1  sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin
    2  echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc
    3  source ~/.bashrc
    4  gcloud container clusters get-credentials orca-cluster-186  --internal-ip --project=qwiklabs-gcp-03-25fec291e65f --zone=europe-west4-b
    5  kubectl get nodes
    6  kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
    7  kubectl get deploy
    8  kubectl get pods
    9  history 
student-02-6781a7ab3969@orca-jumphost:~$ exit
```

