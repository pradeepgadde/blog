---

layout: single
title:  "Anthos: North-south routing with Multi-Cluster Gateways"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Anthos: North-south routing with Multi-Cluster Gateways

- Register GKE clusters to an Anthos Fleet
- Enable and configure Multi-cluster Services (MCS)
- Enable and configure Multi-clusster Gateways (MCG)
- Deploy a distributed application and balance traffic accross clusters

## Configure access for kubectl and verify the cluster

3 Google Kubernetes Engine (GKE) clusters have already been created. You are going to setup access to these clusters.

There are three clusters named, **gke-west-1** , **gke-west-2**, **gke-east-1** .



1. In Cloud Shell, set environment variables for use in scripts
2. Set the zone environment variable for the west1 cluster:
3. Set the zone environment variable for the west2 cluster:
4. Set the zone environment variable for the east1 cluster:
5. Configure `kubectl` to manage your GKE clusters:
6. Rename the cluster contexts so they are easier to reference later
7. Enable the multi-cluster gateway API. (Please note this will take a few minutes.)

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-be9df1e09e70.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe "$PROJECT_ID" \
  --format "value(projectNumber)")
Your active configuration is: [cloudshell-17251]
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ WEST1_LOCATION=us-west2-a
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ WEST2_LOCATION=us-west2-a
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ EAST1_LOCATION=us-central1-a
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container clusters get-credentials gke-west-2 --zone=${WEST2_LOCATION} --project=${PROJECT_ID}
gcloud container clusters get-credentials gke-east-1 --zone=${EAST1_LOCATION}  --project=${PROJECT_ID}
gcloud container clusters get-credentials gke-west-1 --zone=${WEST1_LOCATION} --project=${PROJECT_ID}
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-west-2.
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-east-1.
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-west-1.
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl config rename-context gke_${PROJECT_ID}_${WEST2_LOCATION}_gke-west-2 gke-west-2
kubectl config rename-context gke_${PROJECT_ID}_${EAST1_LOCATION}_gke-east-1 gke-east-1
kubectl config rename-context gke_${PROJECT_ID}_${WEST1_LOCATION}_gke-west-1 gke-west-1
Context "gke_qwiklabs-gcp-01-be9df1e09e70_us-west2-a_gke-west-2" renamed to "gke-west-2".
Context "gke_qwiklabs-gcp-01-be9df1e09e70_us-central1-a_gke-east-1" renamed to "gke-east-1".
Context "gke_qwiklabs-gcp-01-be9df1e09e70_us-west2-a_gke-west-1" renamed to "gke-west-1".
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container clusters update gke-west-1  --gateway-api=standard --region=${WEST1_LOCATION}
Updating gke-west-1...done.                                                                                                                                                        
Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-be9df1e09e70/zones/us-west2-a/clusters/gke-west-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west2-a/gke-west-1?project=qwiklabs-gcp-01-be9df1e09e70
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

## Register clusters in an Anthos Fleet

1. Register these clusters in an Anthos Fleet
2. Confirm that the clusters have successfully registered with an Anthos Fleet:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet memberships register gke-west-1 \
 --gke-cluster ${WEST1_LOCATION}/gke-west-1 \
 --enable-workload-identity \
 --project=${PROJECT_ID}

gcloud container fleet memberships register gke-west-2 \
    --gke-cluster ${WEST2_LOCATION}/gke-west-2 \
    --enable-workload-identity \
    --project=${PROJECT_ID}

gcloud container fleet memberships register gke-east-1 \
    --gke-cluster ${EAST1_LOCATION}/gke-east-1 \
    --enable-workload-identity \
    --project=${PROJECT_ID}
Waiting for membership to be created...done.                                                                                                                                       
Finished registering to the Fleet.
Waiting for membership to be created...done.                                                                                                                                       
Finished registering to the Fleet.
Waiting for membership to be created...done.                                                                                                                                       
Finished registering to the Fleet.
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet memberships list --project=${PROJECT_ID}
NAME: gke-east-1
UNIQUE_ID: 5c550e69-4b99-4278-a870-61941223beff
LOCATION: us-central1

NAME: gke-west-2
UNIQUE_ID: 95721504-2bb1-4fed-bb85-c83c6dbebc03
LOCATION: us-west2

NAME: gke-west-1
UNIQUE_ID: 3dc7534c-b3bf-49ff-980b-852e4c2cf98a
LOCATION: us-west2
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```



## Enable Multi-cluster Services (MCS)

In this task, you enable Multi-cluster Services (MCS) in your fleet for the registered clusters. MCS controller listens for import/export Services, so that Kubernetes Services are routable across clusters and traffic can be distributed across them.

1. Enable multi-cluster Services in your fleet for the registered clusters:
2. Grant the required Identity and Access Management (IAM) permissions required for MCS:
3. Confirm that MCS is enabled for the registered clusters. You will see the memberships for the three registered clusters. It may take several minutes for all of the clusters to show. **Wait and retry until you see a similar output.**

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet multi-cluster-services enable \
--project ${PROJECT_ID}
Waiting for Feature Multi-cluster Services to be created...done.                                                                                                                   
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$  gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --member "serviceAccount:${PROJECT_ID}.svc.id.goog[gke-mcs/gke-mcs-importer]" \
 --role "roles/compute.networkViewer" \
 --project=${PROJECT_ID}
Updated IAM policy for project [qwiklabs-gcp-01-be9df1e09e70].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:272087841988@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-272087841988@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70.svc.id.goog[gke-mcs/gke-mcs-importer]
  role: roles/compute.networkViewer
- members:
  - serviceAccount:service-272087841988@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-272087841988@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:272087841988-compute@developer.gserviceaccount.com
  - serviceAccount:272087841988@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-272087841988@gcp-sa-gkehub.iam.gserviceaccount.com
  role: roles/gkehub.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-mcmetering.iam.gserviceaccount.com
  role: roles/multiclustermetering.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-mcsd.iam.gserviceaccount.com
  role: roles/multiclusterservicediscovery.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-networkconnectivity.iam.gserviceaccount.com
  role: roles/networkconnectivity.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/storage.objectViewer
- members:
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/viewer
etag: BwYd_8_F_cs=
version: 1
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet multi-cluster-services describe --project=${PROJECT_ID}
createTime: '2024-07-24T15:19:23.501923812Z'
name: projects/qwiklabs-gcp-01-be9df1e09e70/locations/global/features/multiclusterservicediscovery
resourceState:
  state: ACTIVE
spec: {}
updateTime: '2024-07-24T15:19:26.059093873Z'
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet multi-cluster-services describe --project=${PROJECT_ID}
createTime: '2024-07-24T15:19:23.501923812Z'
membershipStates:
  projects/272087841988/locations/us-central1/memberships/gke-east-1:
    state:
      code: OK
      description: Firewall successfully updated
      updateTime: '2024-07-24T15:23:37.300509476Z'
  projects/272087841988/locations/us-west2/memberships/gke-west-1:
    state:
      code: OK
      description: Firewall successfully updated
      updateTime: '2024-07-24T15:23:36.425508341Z'
  projects/272087841988/locations/us-west2/memberships/gke-west-2:
    state:
      code: OK
      description: Firewall successfully updated
      updateTime: '2024-07-24T15:23:36.861579482Z'
name: projects/qwiklabs-gcp-01-be9df1e09e70/locations/global/features/multiclusterservicediscovery
resourceState:
  state: ACTIVE
spec: {}
updateTime: '2024-07-24T15:19:26.059093873Z'
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```



## Install Gateway API CRDs and enable the Multi-cluster Gateway (MCG) controller

Before using Gateway resources in GKE, you must install the [Gateway API CustomResource Definitions (CRDs)](https://gateway-api.sigs.k8s.io) in your config cluster and enable the Multi-cluster Gateway (MCG) controller. The config cluster is the GKE cluster in which your Gateway and Route resources are deployed. It is a central place that controls routing across your clusters. You will use **gke-west-1** as your config cluster.

Deploy Gateway resources into the `gke-west-1` cluster:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.0" \
| kubectl apply -f - --context=gke-west-1
Warning: resource customresourcedefinitions/gatewayclasses.gateway.networking.k8s.io is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io configured
Warning: resource customresourcedefinitions/gateways.gateway.networking.k8s.io is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io configured
Warning: resource customresourcedefinitions/httproutes.gateway.networking.k8s.io is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io configured
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Enable the Multi-cluster Gateway controller for the `gke-west-1` cluster:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet ingress enable \
  --config-membership=gke-west-1 \
  --project=${PROJECT_ID} \
  --location=us-west2
Waiting for Feature Ingress to be created...done.                                                                                                                                  
Waiting for controller to start......done.                                                                                                                                         
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Confirm that the global Gateway controller is enabled for the registered clusters:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud container fleet ingress describe --project=${PROJECT_ID}
createTime: '2024-07-24T15:23:13.122782619Z'
membershipStates:
  projects/272087841988/locations/us-central1/memberships/gke-east-1:
    state:
      code: OK
      updateTime: '2024-07-24T15:24:13.833390790Z'
  projects/272087841988/locations/us-west2/memberships/gke-west-1:
    state:
      code: OK
      updateTime: '2024-07-24T15:24:13.833389327Z'
  projects/272087841988/locations/us-west2/memberships/gke-west-2:
    state:
      code: OK
      updateTime: '2024-07-24T15:24:13.833392020Z'
name: projects/qwiklabs-gcp-01-be9df1e09e70/locations/global/features/multiclusteringress
resourceState:
  state: ACTIVE
spec:
  multiclusteringress:
    configMembership: projects/qwiklabs-gcp-01-be9df1e09e70/locations/us-west2/memberships/gke-west-1
state:
  state:
    code: OK
    description: Ready to use
    updateTime: '2024-07-24T15:24:13.274032222Z'
updateTime: '2024-07-24T15:24:25.827956580Z'
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Grant Identity and Access Management (IAM) permissions required by the Gateway controller:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member "serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-multiclusteringress.iam.gserviceaccount.com" \
  --role "roles/container.admin" \
  --project=${PROJECT_ID}
Updated IAM policy for project [qwiklabs-gcp-01-be9df1e09e70].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  role: roles/bigquery.admin
- members:
  - serviceAccount:272087841988@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-272087841988@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70.svc.id.goog[gke-mcs/gke-mcs-importer]
  role: roles/compute.networkViewer
- members:
  - serviceAccount:service-272087841988@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-multiclusteringress.iam.gserviceaccount.com
  role: roles/container.admin
- members:
  - serviceAccount:service-272087841988@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:272087841988-compute@developer.gserviceaccount.com
  - serviceAccount:272087841988@cloudservices.gserviceaccount.com
  role: roles/editor
- members:
  - serviceAccount:service-272087841988@gcp-sa-gkehub.iam.gserviceaccount.com
  role: roles/gkehub.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-multiclusteringress.iam.gserviceaccount.com
  role: roles/multiclusteringress.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-mcmetering.iam.gserviceaccount.com
  role: roles/multiclustermetering.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-mcsd.iam.gserviceaccount.com
  role: roles/multiclusterservicediscovery.serviceAgent
- members:
  - serviceAccount:service-272087841988@gcp-sa-networkconnectivity.iam.gserviceaccount.com
  role: roles/networkconnectivity.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-01-be9df1e09e70@qwiklabs-gcp-01-be9df1e09e70.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/storage.objectViewer
- members:
  - user:student-02-c9fcafe3ea80@qwiklabs.net
  role: roles/viewer
etag: BwYd_-LvecY=
version: 1
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

List the GatewayClasses:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl get gatewayclasses --context=gke-west-1
Error from server (NotFound): Unable to list "gateway.networking.k8s.io/v1, Resource=gatewayclasses": the server could not find the requested resource (get gatewayclasses.gateway.networking.k8s.io)
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

after a few minutes	

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl get gatewayclasses --context=gke-west-1
NAME                                  CONTROLLER                  ACCEPTED   AGE
gke-l7-global-external-managed        networking.gke.io/gateway   True       46m
gke-l7-global-external-managed-mc     networking.gke.io/gateway   True       38m
gke-l7-gxlb                           networking.gke.io/gateway   True       46m
gke-l7-gxlb-mc                        networking.gke.io/gateway   True       38m
gke-l7-regional-external-managed      networking.gke.io/gateway   True       46m
gke-l7-regional-external-managed-mc   networking.gke.io/gateway   True       38m
gke-l7-rilb                           networking.gke.io/gateway   True       46m
gke-l7-rilb-mc                        networking.gke.io/gateway   True       38m
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```



Four Gateway classes have been installed. The gke-l7-gxlb-mc for external multi-cluster Gateways and gke-l7-rilb-mc for internal multi-cluster Gateways. The other two are used for single cluster deployments.

Congratulations! You can now create multi-cluster Gateways using these Gateway classes.

## Deploy the demo application

1. Create the `store` Deployment and Namespace in the `gke-east-1` and `gke-west-2`. The config cluster can also host workloads, but in this lab, you only run the Gateway controllers and configuration on it:

```yaml
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ cat <<EOF > store-deployment.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: store
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store
  namespace: store
spec:
  replicas: 2
  selector:
kubectl apply -f store-deployment.yaml --context=gke-east-1
namespace/store created
deployment.apps/store created
namespace/store created
deployment.apps/store created
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Create the Service and ServiceExports for the `gke-west-2` cluster:

```yaml
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ cat <<EOF > store-west-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: store
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
kubectl apply -f store-west-service.yaml --context=gke-west-2
service/store created
serviceexport.net.gke.io/store created
service/store-west-2 created
serviceexport.net.gke.io/store-west-2 created
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Create the Service and ServiceExports for the `gke-east-1` cluster:

```yaml
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ cat <<EOF > store-east-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: store
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
kubectl apply -f store-east-service.yaml --context=gke-east-1
service/store created
serviceexport.net.gke.io/store created
service/store-east-1 created
serviceexport.net.gke.io/store-east-1 created
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Make sure that the service exports have been successfully created:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl get serviceexports --context gke-west-2 --namespace store
kubectl get serviceexports --context gke-east-1 --namespace store
NAME           AGE
store          62s
store-west-2   61s
NAME           AGE
store          30s
store-east-1   30s
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

## Deploy the Gateway and HTTPRoutes

Gateway and HTTPRoutes are resources deployed in the Config cluster, which in this case is the `gke-west-1` cluster.

Platform administrators manage and deploy **Gateways** to centralize security policies such as TLS.

Service Owners in different teams deploy **HTTPRoutes** in their own namespace so that they can independently control their routing logic.

Deploy the Gateway in the `gke-west-1` config cluster

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ cat <<EOF > external-http-gateway.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: store
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: external-http
  namespace: store
spec:
  gatewayClassName: gke-l7-gxlb-mc
  listeners:
kubectl apply -f external-http-gateway.yaml --context=gke-west-1
namespace/store created
gateway.gateway.networking.k8s.io/external-http created
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Deploy the HTTPRoute in the `gke-west-1` config cluster:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ cat <<EOF > public-store-route.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: public-store-route
  namespace: store
  labels:
    gateway: external-http
spec:
  hostnames:
  - "store.example.com"
  parentRefs:
  - name: external-http
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /west
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-west-2
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /east
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-east-1
      port: 8080
  - backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store
      port: 8080
EOF
kubectl apply -f public-store-route.yaml --context=gke-west-1
httproute.gateway.networking.k8s.io/public-store-route created
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Notice that we are sending the default requests to the closest backend defined by the default rule. In case that the path `/west` is in the request, the request is routed to the service in `gke-west-2`. If the request's path matches `/east`, the request is routed to the `gke-east-1` cluster.

View the status of the Gateway that you just created in `gke-west-1`:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl describe gateway external-http --context gke-west-1 --namespace store
Name:         external-http
Namespace:    store
Labels:       <none>
Annotations:  networking.gke.io/addresses: /projects/272087841988/global/addresses/gkemcg1-store-external-http-laup24msshu4
              networking.gke.io/backend-services:
                /projects/272087841988/global/backendServices/gkemcg1-kube-system-gw-serve404-80-7cq0brelgzex, /projects/272087841988/global/backendServic...
              networking.gke.io/firewalls: /projects/272087841988/global/firewalls/gkemcg1-l7-default-global
              networking.gke.io/forwarding-rules: /projects/272087841988/global/forwardingRules/gkemcg1-store-external-http-a5et3e3itxsv
              networking.gke.io/health-checks:
                /projects/272087841988/global/healthChecks/gkemcg1-kube-system-gw-serve404-80-7cq0brelgzex, /projects/272087841988/global/healthChecks/gke...
              networking.gke.io/last-reconcile-time: 2024-07-24T16:02:08Z
              networking.gke.io/ssl-certificates: 
              networking.gke.io/target-http-proxies: /projects/272087841988/global/targetHttpProxies/gkemcg1-store-external-http-94oqhkftu5yz
              networking.gke.io/target-https-proxies: 
              networking.gke.io/url-maps: /projects/272087841988/global/urlMaps/gkemcg1-store-external-http-94oqhkftu5yz
API Version:  gateway.networking.k8s.io/v1beta1
Kind:         Gateway
Metadata:
  Creation Timestamp:  2024-07-24T15:36:49Z
  Finalizers:
    gateway.finalizer.networking.gke.io
  Generation:        1
  Resource Version:  305193
  UID:               aa6b37e6-4492-406b-8d7f-0964c3c32934
Spec:
  Gateway Class Name:  gke-l7-gxlb-mc
  Listeners:
    Allowed Routes:
      Kinds:
        Group:  gateway.networking.k8s.io
        Kind:   HTTPRoute
      Namespaces:
        From:  Same
    Name:      http
    Port:      80
    Protocol:  HTTP
Status:
  Addresses:
    Type:   IPAddress
    Value:  34.144.212.124
  Conditions:
    Last Transition Time:  2024-07-24T15:38:46Z
    Message:               The OSS Gateway API has deprecated this condition, do not depend on it.
    Observed Generation:   1
    Reason:                Scheduled
    Status:                True
    Type:                  Scheduled
    Last Transition Time:  2024-07-24T15:38:46Z
    Message:               
    Observed Generation:   1
    Reason:                Accepted
    Status:                True
    Type:                  Accepted
    Last Transition Time:  2024-07-24T15:38:46Z
    Message:               
    Observed Generation:   1
    Reason:                Programmed
    Status:                True
    Type:                  Programmed
    Last Transition Time:  2024-07-24T15:38:46Z
    Message:               The OSS Gateway API has altered the "Ready" condition semantics and reserved it for future use.  GKE Gateway will stop emitting it in a future update, use "Programmed" instead.
    Observed Generation:   1
    Reason:                Ready
    Status:                True
    Type:                  Ready
    Last Transition Time:  2024-07-24T15:38:46Z
    Message:               
    Observed Generation:   1
    Reason:                Healthy
    Status:                True
    Type:                  networking.gke.io/GatewayHealthy
  Listeners:
    Attached Routes:  1
    Conditions:
      Last Transition Time:  2024-07-24T15:38:46Z
      Message:               
      Observed Generation:   1
      Reason:                ResolvedRefs
      Status:                True
      Type:                  ResolvedRefs
      Last Transition Time:  2024-07-24T15:38:46Z
      Message:               
      Observed Generation:   1
      Reason:                Accepted
      Status:                True
      Type:                  Accepted
      Last Transition Time:  2024-07-24T15:38:46Z
      Message:               
      Observed Generation:   1
      Reason:                Programmed
      Status:                True
      Type:                  Programmed
      Last Transition Time:  2024-07-24T15:38:46Z
      Message:               The OSS Gateway API has altered the "Ready" condition semantics and reserved it for future use.  GKE Gateway will stop emitting it in a future update, use "Programmed" instead.
      Observed Generation:   1
      Reason:                Ready
      Status:                True
      Type:                  Ready
    Name:                    http
    Supported Kinds:
      Group:  gateway.networking.k8s.io
      Kind:   HTTPRoute
Events:
  Type    Reason  Age                 From                   Message
  ----    ------  ----                ----                   -------
  Normal  ADD     26m                 mc-gateway-controller  store/external-http
  Normal  UPDATE  22m (x4 over 26m)   mc-gateway-controller  store/external-http
  Normal  SYNC    22m (x11 over 25m)  mc-gateway-controller  store/external-http
  Normal  SYNC    67s (x10 over 24m)  mc-gateway-controller  SYNC on store/external-http was a success
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Sometimes there are transient errors shown in the Events section. Wait till it shows the follwoing message: "SYNC on store/external-http was a success". It might take up to 10 minutes to sync.

It takes some time for the external IP to be created. To ensure that it is, run this command until you see the external IP.

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ kubectl get gateway external-http -o=jsonpath="{.status.addresses[0].value}" --context gke-west-1 --namespace store | xargs echo -e
34.144.212.124
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

If it doesn't return an external IP, wait for a few minutes and run it again.

Get the external IP created by the Gateway:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ EXTERNAL_IP=$(kubectl get gateway external-http -o=jsonpath="{.status.addresses[0].value}" --context gke-west-1 --namespace store)
echo $EXTERNAL_IP
34.144.212.124
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Make sure that the IP is not empty.

Access the default application. This returns the cluster closest to you:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ curl -H "host: store.example.com" http://${EXTERNAL_IP}
{
  "cluster_name": "gke-east-1", 
  "host_header": "store.example.com", 
  "node_name": "gke-gke-east-1-pool-12345-7d235a06-jh63.us-central1-a.c.qwiklabs-gcp-01-be9df1e09e70.internal", 
  "pod_name": "store-c9f5dbdd7-kgbxs", 
  "pod_name_emoji": "🧡", 
  "project_id": "qwiklabs-gcp-01-be9df1e09e70", 
  "timestamp": "2024-07-24T16:05:47", 
  "zone": "us-central1-a"
}
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

If you see the following message: "default backend - 404" or "curl: (52) Empty reply from server", the Gateway is not ready yet. Wait a couple of minutes and try again.

Access the application located in the `gke-west-2` cluster:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ curl -H "host: store.example.com" http://${EXTERNAL_IP}/west
{
  "cluster_name": "gke-west-2", 
  "host_header": "store.example.com", 
  "node_name": "gke-gke-west-2-pool-12345-c0204061-d2mz.us-west2-a.c.qwiklabs-gcp-01-be9df1e09e70.internal", 
  "pod_name": "store-c9f5dbdd7-t4kg4", 
  "pod_name_emoji": "🤵‍♀️", 
  "project_id": "qwiklabs-gcp-01-be9df1e09e70", 
  "timestamp": "2024-07-24T16:06:35", 
  "zone": "us-west2-a"
}
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

Access the application located in the `gke-east-1` cluster:

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ curl -H "host: store.example.com" http://${EXTERNAL_IP}/east
{
  "cluster_name": "gke-east-1", 
  "host_header": "store.example.com", 
  "node_name": "gke-gke-east-1-pool-12345-7d235a06-jh63.us-central1-a.c.qwiklabs-gcp-01-be9df1e09e70.internal", 
  "pod_name": "store-c9f5dbdd7-jbbxw", 
  "pod_name_emoji": "🦸🏾‍♂️", 
  "project_id": "qwiklabs-gcp-01-be9df1e09e70", 
  "timestamp": "2024-07-24T16:07:11", 
  "zone": "us-central1-a"
}
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

In this lab, you registered the pre-created GKE clusters to an Anthos Fleet, enabled and configured the Multi-cluster Services (MCS) and Multi-cluster Gateways (MCG) controllers, deployed Gateways and HTTPRoutes in the config cluster, and run a distributed application across multiple clusters with a unique Load Balancer routing the traffic to the right pods.

## History

```sh
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project)
    2  export PROJECT_NUMBER=$(gcloud projects describe "$PROJECT_ID" \
  --format "value(projectNumber)")
    3  WEST1_LOCATION=us-west2-a
    4  WEST2_LOCATION=us-west2-a
    5  EAST1_LOCATION=us-central1-a
    6  gcloud container clusters get-credentials gke-west-2 --zone=${WEST2_LOCATION} --project=${PROJECT_ID}
    7  gcloud container clusters get-credentials gke-east-1 --zone=${EAST1_LOCATION}  --project=${PROJECT_ID}
    8  gcloud container clusters get-credentials gke-west-1 --zone=${WEST1_LOCATION} --project=${PROJECT_ID}
    9  kubectl config rename-context gke_${PROJECT_ID}_${WEST2_LOCATION}_gke-west-2 gke-west-2
   10  kubectl config rename-context gke_${PROJECT_ID}_${EAST1_LOCATION}_gke-east-1 gke-east-1
   11  kubectl config rename-context gke_${PROJECT_ID}_${WEST1_LOCATION}_gke-west-1 gke-west-1
   12  gcloud container clusters update gke-west-1  --gateway-api=standard --region=${WEST1_LOCATION}
   13  gcloud container fleet memberships register gke-west-1  --gke-cluster ${WEST1_LOCATION}/gke-west-1  --enable-workload-identity  --project=${PROJECT_ID}
   14  gcloud container fleet memberships register gke-west-2     --gke-cluster ${WEST2_LOCATION}/gke-west-2     --enable-workload-identity     --project=${PROJECT_ID}
   15  gcloud container fleet memberships register gke-east-1     --gke-cluster ${EAST1_LOCATION}/gke-east-1     --enable-workload-identity     --project=${PROJECT_ID}
   16  gcloud container fleet memberships list --project=${PROJECT_ID}
   17  gcloud container fleet multi-cluster-services enable --project ${PROJECT_ID}
   18  gcloud container fleet multi-cluster-services describe --project=${PROJECT_ID}
   19  kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.0" | kubectl apply -f - --context=gke-west-1
   20  gcloud container fleet ingress enable   --config-membership=gke-west-1   --project=${PROJECT_ID}   --location=us-west2
   21  gcloud container fleet ingress describe --project=${PROJECT_ID}
   22  gcloud projects add-iam-policy-binding ${PROJECT_ID}   --member "serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-multiclusteringress.iam.gserviceaccount.com"   --role "roles/container.admin"   --project=${PROJECT_ID}
   23  kubectl get gatewayclasses --context=gke-west-1
   24  gcloud projects add-iam-policy-binding ${PROJECT_ID}   --member "serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-multiclusteringress.iam.gserviceaccount.com"   --role "roles/container.admin"   --project=${PROJECT_ID}
   25  gcloud container fleet ingress describe --project=${PROJECT_ID}
   26  kubectl get gatewayclasses --context=gke-west-1
   27  kubectl get gatewayclasses --context=gke-west-2
   28  kubectl get gatewayclasses --context=gke-west-1
   29  kubectl get gatewayclasses 
   30  kubectl get crd
   31  kubectl get gateway
   32  kubectl get gateway --context=gke-west-1
   33  kubectl get gatewayclasses --context=gke-west-1
   34  gcloud container fleet ingress enable   --config-membership=gke-west-1   --project=${PROJECT_ID}   --location=us-west2
   35  kubectl get gatewayclasses --context=gke-west-1
   36  gcloud container fleet multi-cluster-services describe --project=${PROJECT_ID}
   37  kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.0" | kubectl apply -f - --context=gke-west-1
   38  kubectl get gatewayclasses --context=gke-west-1
   39  kubectl get gatewayclass --context=gke-west-1
   40  kubectl get gatewayclasses --context=gke-west-1
   41  cat <<EOF > store-deployment.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: store
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store
  namespace: store
spec:
  replicas: 2
  selector:
    matchLabels:
      app: store
      version: v1
  template:
    metadata:
      labels:
        app: store
        version: v1
    spec:
      containers:
      - name: whereami
        image: gcr.io/google-samples/whereami:v1.2.1
        ports:
          - containerPort: 8080
EOF

   42  kubectl apply -f store-deployment.yaml --context=gke-west-2
   43  kubectl apply -f store-deployment.yaml --context=gke-east-1
   44  cat <<EOF > store-west-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: store
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
apiVersion: net.gke.io/v1
metadata:
  name: store
  namespace: store
---
apiVersion: v1
kind: Service
metadata:
  name: store-west-2
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
apiVersion: net.gke.io/v1
metadata:
  name: store-west-2
  namespace: store
EOF

   45  kubectl apply -f store-west-service.yaml --context=gke-west-2
   46  cat <<EOF > store-east-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: store
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
apiVersion: net.gke.io/v1
metadata:
  name: store
  namespace: store
---
apiVersion: v1
kind: Service
metadata:
  name: store-east-1
  namespace: store
spec:
  selector:
    app: store
  ports:
  - port: 8080
    targetPort: 8080
---
kind: ServiceExport
apiVersion: net.gke.io/v1
metadata:
  name: store-east-1
  namespace: store
EOF

   47  kubectl apply -f store-east-service.yaml --context=gke-east-1
   48  kubectl get serviceexports --context gke-west-2 --namespace store
   49  kubectl get serviceexports --context gke-east-1 --namespace store
   50  cat <<EOF > external-http-gateway.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: store
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: external-http
  namespace: store
spec:
  gatewayClassName: gke-l7-gxlb-mc
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      kinds:
      - kind: HTTPRoute
EOF

   51  kubectl apply -f external-http-gateway.yaml --context=gke-west-1
   52  cat <<EOF > public-store-route.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: public-store-route
  namespace: store
  labels:
    gateway: external-http
spec:
  hostnames:
  - "store.example.com"
  parentRefs:
  - name: external-http
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /west
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-west-2
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /east
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-east-1
      port: 8080
  - backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store
      port: 8080
EOF

   53  kubectl apply -f public-store-route.yaml --context=gke-west-1
   54  cat <<EOF > public-store-route.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: public-store-route
  namespace: store
  labels:
    gateway: external-http
spec:
  hostnames:
  - "store.example.com"
  parentRefs:
  - name: external-http
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /west
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-west-2
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /east
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-east-1
      port: 8080
  - backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store
      port: 8080
EOF

   55  kubectl apply -f public-store-route.yaml --context=gke-west-1
   56  cat <<EOF > public-store-route.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: public-store-route
  namespace: store
  labels:
    gateway: external-http
spec:
  hostnames:
  - "store.example.com"
  parentRefs:
  - name: external-http
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /west
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-west-2
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /east
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-east-1
      port: 8080
  - backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store
      port: 8080
EOF

   57  kubectl apply -f public-store-route.yaml --context=gke-west-1
   58  kubectl describe gateway external-http --context gke-west-1 --namespace store
   59  kubectl get gatewayclasses --context=gke-west-1
   60  gcloud container fleet multi-cluster-services enable --project ${PROJECT_ID}
   61  kubectl get gatewayclasses --context=gke-west-1
   62  kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.5.0" | kubectl apply -f - --context=gke-west-1
   63  gcloud container fleet ingress enable   --config-membership=gke-west-1   --project=${PROJECT_ID}   --location=us-west2
   64  gcloud container fleet ingress describe --project=${PROJECT_ID}
   65  gcloud projects add-iam-policy-binding ${PROJECT_ID}   --member "serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-multiclusteringress.iam.gserviceaccount.com"   --role "roles/container.admin"   --project=${PROJECT_ID}
   66  kubectl get gatewayclasses --context=gke-west-1
   67  kubectl describe gateway external-http --context gke-west-1 --namespace store
   68  kubectl get crd | grep gateways
   69  cat <<EOF > external-http-gateway.yaml
kind: Namespace
apiVersion: v1
metadata:
  name: store
---
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: external-http
  namespace: store
spec:
  gatewayClassName: gke-l7-gxlb-mc
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      kinds:
      - kind: HTTPRoute
EOF

   70  kubectl apply -f external-http-gateway.yaml --context=gke-west-1
   71  cat <<EOF > public-store-route.yaml
kind: HTTPRoute
apiVersion: gateway.networking.k8s.io/v1beta1
metadata:
  name: public-store-route
  namespace: store
  labels:
    gateway: external-http
spec:
  hostnames:
  - "store.example.com"
  parentRefs:
  - name: external-http
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /west
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-west-2
      port: 8080
  - matches:
    - path:
        type: PathPrefix
        value: /east
    backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store-east-1
      port: 8080
  - backendRefs:
    - group: net.gke.io
      kind: ServiceImport
      name: store
      port: 8080
EOF

   72  kubectl apply -f public-store-route.yaml --context=gke-west-1
   73  kubectl describe gateway external-http --context gke-west-1 --namespace store
   74  ls
   75  vi public-store-route.yaml 
   76  kubectl apply -f public-store-route.yaml --context=gke-west-1
   77  cat external-http-gateway.yaml 
   78  vi external-http-gateway.yaml 
   79  kubectl apply -f external-http-gateway.yaml --context=gke-west-1
   80  kubectl get crd
   81  kubectl describe crd gatewayclasses.gateway.networking.k8s.io
   82  kubectl describe gateway external-http --context gke-west-1 --namespace store
   83  kubectl get gatewayclasses --context=gke-west-1
   84  kubectl describe gateway external-http --context gke-west-1 --namespace store
   85  kubectl get gateway external-http -o=jsonpath="{.status.addresses[0].value}" --context gke-west-1 --namespace store | xargs echo -e
   86  EXTERNAL_IP=$(kubectl get gateway external-http -o=jsonpath="{.status.addresses[0].value}" --context gke-west-1 --namespace store)
   87  echo $EXTERNAL_IP
   88  curl -H "host: store.example.com" http://${EXTERNAL_IP}
   89  curl -H "host: store.example.com" http://${EXTERNAL_IP}/west
   90  curl -H "host: store.example.com" http://${EXTERNAL_IP}/east
   91  history 
student_02_c9fcafe3ea80@cloudshell:~ (qwiklabs-gcp-01-be9df1e09e70)$ 
```

