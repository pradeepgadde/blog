---

layout: single
title:  "Build and Deploy a Docker Image to a Kubernetes Cluster"
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
# Build and Deploy a Docker Image to a Kubernetes Cluster

## Challenge scenario

Your development team is interested in adopting a containerized  microservices approach to application architecture. You need to test a  sample application they have provided for you to make sure that it can  be deployed to a Google Kubernetes container. The development group  provided a simple Go application called `echo-web` with a Dockerfile and the associated context that allows you to build a Docker image immediately.

## Your challenge

To test the deployment, you need to download the sample application,  then build the Docker container image using a tag that allows it to be  stored on the Container Registry. Once the image has been built, you'll  push it out to the Container Registry before you can deploy it.

With the image prepared you can then create a Kubernetes cluster, then deploy the sample application to the cluster.

## Create a Kubernetes cluster

1. Your test environment is limited in capacity, so you should limit the test Kubernetes cluster you are creating to just two `e2-standard-2` instances. You must call your cluster `echo-cluster`.

## Build a tagged Docker image

The sample application, including the Dockerfile and the application context files, are contained in an archive called `echo-web.tar.gz`. The archive has been copied to a Cloud Storage bucket belonging to your lab project called `gs://[PROJECT_ID].`

- You must deploy this with a tag called `v1. `

## Push the image to the Google Container Registry

- Your organization has decided that it will always use the `gcr.io` Container Registry hostname for all projects. The sample application is a simple web application that reports some data describing the  configuration of the system where the application is running. It is  configured to use TCP port 8000 by default.

## Deploy the application to the Kubernetes cluster

- Even though the application is configured to respond to HTTP  requests on port 8000, you must configure the service to respond to  normal web requests on port 80. When configuring the cluster for your  sample application, call your deployment `echo-web`.


```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-02-b72e20704fdb.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ gcloud container clusters create echo-cluster --zone=us-east4-b --num-nodes 2 --machine-type=e2-standard-2
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster echo-cluster in us-east4-b... Cluster is being health-checked (master is healthy)...done.                                    
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-02-b72e20704fdb/zones/us-east4-b/clusters/echo-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east4-b/echo-cluster?project=qwiklabs-gcp-02-b72e20704fdb
kubeconfig entry generated for echo-cluster.
NAME: echo-cluster
LOCATION: us-east4-b
MASTER_VERSION: 1.28.8-gke.1095000
MASTER_IP: 34.85.218.181
MACHINE_TYPE: e2-standard-2
NODE_VERSION: 1.28.8-gke.1095000
NUM_NODES: 2
STATUS: RUNNING
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ gsutil cp gs://qwiklabs-gcp-02-b72e20704fdb/echo-web.tar.gz .
Copying gs://qwiklabs-gcp-02-b72e20704fdb/echo-web.tar.gz...
/ [1 files][  2.0 KiB/  2.0 KiB]                                                
Operation completed over 1 objects/2.0 KiB.                                      
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ ls
echo-web.tar.gz  README-cloudshell.txt
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ tar -xvf echo-web.tar.gz 
./
./README.md
./main.go
./manifests/
./manifests/echoweb-deployment.yaml
./manifests/echoweb-service-static-ip.yaml
./manifests/echoweb-ingress-static-ip.yaml
./Dockerfile
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ ls
Dockerfile  echo-web.tar.gz  main.go  manifests  README-cloudshell.txt  README.md
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ cat Dockerfile 
FROM golang:1.8-alpine
ADD . /go/src/echo-app
RUN go install echo-app

FROM alpine:latest
COPY --from=0 /go/bin/echo-app .
ENV PORT 8000
CMD ["./echo-app"]
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ 
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ gcloud auth configure-docker 
WARNING: Your config file at [/home/student_02_0bc8a0c336eb/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
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
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud"
  }
}
Adding credentials for all GCR repositories.
WARNING: A long list of credential helpers may cause delays running 'docker build'. We recommend passing the registry name to configure only the registry you are using.
After update, the following will be written to your Docker config file located at [/home/student_02_0bc8a0c336eb/.docker/config.json]:
 {
  "credHelpers": {
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
    "europe-west10-docker.pkg.dev": "gcloud",
    "europe-west12-docker.pkg.dev": "gcloud",
    "europe-west2-docker.pkg.dev": "gcloud",
    "europe-west3-docker.pkg.dev": "gcloud",
    "europe-west4-docker.pkg.dev": "gcloud",
    "europe-west6-docker.pkg.dev": "gcloud",
    "europe-west8-docker.pkg.dev": "gcloud",
    "europe-west9-docker.pkg.dev": "gcloud",
    "me-central1-docker.pkg.dev": "gcloud",
    "me-central2-docker.pkg.dev": "gcloud",
    "me-west1-docker.pkg.dev": "gcloud",
    "northamerica-northeast1-docker.pkg.dev": "gcloud",
    "northamerica-northeast2-docker.pkg.dev": "gcloud",
    "southamerica-east1-docker.pkg.dev": "gcloud",
    "us-docker.pkg.dev": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud",
    "us-central2-docker.pkg.dev": "gcloud",
    "us-east1-docker.pkg.dev": "gcloud",
    "us-east4-docker.pkg.dev": "gcloud",
    "us-east5-docker.pkg.dev": "gcloud",
    "us-east7-docker.pkg.dev": "gcloud",
    "us-south1-docker.pkg.dev": "gcloud",
    "us-west1-docker.pkg.dev": "gcloud",
    "us-west2-docker.pkg.dev": "gcloud",
    "us-west3-docker.pkg.dev": "gcloud",
    "us-west4-docker.pkg.dev": "gcloud",
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}

Do you want to continue (Y/n)?  y

Docker configuration file updated.
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ docker build -t gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1 .
[+] Building 1.8s (11/11) FINISHED                                                                                                                                   docker:default
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 195B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/golang:1.8-alpine                                                                                                           0.7s
 => [internal] load metadata for docker.io/library/alpine:latest                                                                                                               0.7s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 2.94kB                                                                                                                                            0.0s
 => [stage-1 1/2] FROM docker.io/library/alpine:latest@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b                                                 0.0s
 => CACHED [stage-0 1/3] FROM docker.io/library/golang:1.8-alpine@sha256:693568f2ab0dae1e19f44b41628d2aea148fac65974cfd18f83cb9863ab1a177                                      0.0s
 => [stage-0 2/3] ADD . /go/src/echo-app                                                                                                                                       0.0s
 => [stage-0 3/3] RUN go install echo-app                                                                                                                                      0.8s
 => CACHED [stage-1 2/2] COPY --from=0 /go/bin/echo-app .                                                                                                                      0.0s
 => exporting to image                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:d106b3841d829ea2adb605a84a243a7f68cbc4c4e737f2d52a11ed65d6627935                                                                                   0.0s
 => => naming to gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1                                                                                                               0.0s
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ docker push gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
The push refers to repository [gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app]
41e88e995997: Layer already exists 
d4fc045c9e3a: Layer already exists 
v1: digest: sha256:c8a231ae40d1fd98ef21353cd1e637c8ef2805bff87a272357d95e810ce93f75 size: 739
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get nodes
NAME                                          STATUS   ROLES    AGE   VERSION
gke-echo-cluster-default-pool-15cbea64-l757   Ready    <none>   24m   v1.28.8-gke.1095000
gke-echo-cluster-default-pool-15cbea64-ldmp   Ready    <none>   24m   v1.28.8-gke.1095000
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get deploy
No resources found in default namespace.
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl create deploy -h
Create a deployment with the specified name.

Aliases:
deployment, deploy

Examples:
  # Create a deployment named my-dep that runs the busybox image
  kubectl create deployment my-dep --image=busybox
  
  # Create a deployment with a command
  kubectl create deployment my-dep --image=busybox -- date
  
  # Create a deployment named my-dep that runs the nginx image with 3 replicas
  kubectl create deployment my-dep --image=nginx --replicas=3
  
  # Create a deployment named my-dep that runs the busybox image and expose port 5701
  kubectl create deployment my-dep --image=busybox --port=5701

Options:
    --allow-missing-template-keys=true:
        If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
        golang and jsonpath output formats.

    --dry-run='none':
        Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
        sending it. If server strategy, submit server-side request without persisting the resource.

    --field-manager='kubectl-create':
        Name of the manager used to track field ownership.

    --image=[]:
        Image names to run.

    -o, --output='':
        Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
        jsonpath-as-json, jsonpath-file).

    --port=-1:
        The port that this container exposes.

    -r, --replicas=1:
        Number of replicas to create. Default is 1.

    --save-config=false:
        If true, the configuration of current object will be saved in its annotation. Otherwise, the annotation will
        be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.

    --show-managed-fields=false:
        If true, keep the managedFields when printing objects in JSON or YAML format.

    --template='':
        Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
        is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    --validate='strict':
        Must be one of: strict (or true), warn, ignore (or false).              "true" or "strict" will use a schema to validate
        the input and fail the request if invalid. It will perform server side validation if ServerSideFieldValidation
        is enabled on the api-server, but will fall back to less reliable client-side validation if not.                "warn" will
        warn about unknown or duplicate fields without blocking the request if server-side field validation is enabled
        on the API server, and behave as "ignore" otherwise.            "false" or "ignore" will not perform any schema
        validation, silently dropping any unknown or duplicate fields.

Usage:
  kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl create deployment echo-web --image=gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
deployment.apps/echo-web created
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-5cd58676b4-jsjcd   1/1     Running   0          5s
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE                                          NOMINATED NODE   READINESS GATES
echo-web-5cd58676b4-jsjcd   1/1     Running   0          11s   10.88.1.11   gke-echo-cluster-default-pool-15cbea64-l757   <none>           <none>
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ 
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ docker run -p 8000:8000 --name echo-app -d gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
dd4ea2901da065fe381c72c35e0e10125e6f828dc7059887822f3da797ace872
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ 
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl exec -it echo-web-5cd58676b4-jsjcd -- ash
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1460 qdisc noqueue state UP 
    link/ether 86:ac:c0:3a:74:b8 brd ff:ff:ff:ff:ff:ff
    inet 10.88.1.11/24 brd 10.88.1.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # curl localhost:8000
ash: curl: not found
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 ./echo-app
   17 root      0:00 ash
   24 root      0:00 ps
/ # apt-get install curl
ash: apt-get: not found
/ # yum install curl
ash: yum: not found
/ # apt install curl
ash: apt: not found
/ # dnf curl
ash: dnf: not found
/ # exit
command terminated with exit code 127
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
echo-web   1/1     1            1           3m57s
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl expose deploy echo-web -h
Expose a resource as a new Kubernetes service.

 Looks up a deployment, service, replica set, replication controller or pod by name and uses the selector for that
resource as the selector for a new service on the specified port. A deployment or replica set will be exposed as a
service only if its selector is convertible to a selector that service supports, i.e. when the selector contains only
the matchLabels component. Note that if no port is specified via --port and the exposed resource has multiple ports, all
will be re-used by the new service. Also if no labels are specified, the new service will re-use the labels from the
resource it exposes.

 Possible resources include (case insensitive):

 pod (po), service (svc), replicationcontroller (rc), deployment (deploy), replicaset (rs)

Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000
  
  # Create a service for a replication controller identified by type and name specified in "nginx-controller.yaml",
which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000
  
  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend
  
  # Create a second service based on the above service, exposing the container port 8443 as port 443 with the name
"nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https
  
  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream
  
  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects to the containers on
port 8000
  kubectl expose rs nginx --port=80 --target-port=8000
  
  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000

Options:
    --allow-missing-template-keys=true:
        If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
        golang and jsonpath output formats.

    --cluster-ip='':
        ClusterIP to be assigned to the service. Leave empty to auto-allocate, or set to 'None' to create a headless
        service.

    --dry-run='none':
        Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
        sending it. If server strategy, submit server-side request without persisting the resource.

    --external-ip='':
        Additional external IP address (not managed by Kubernetes) to accept for the service. If this IP is routed to
        a node, the service can be accessed by this IP in addition to its generated service IP.

    --field-manager='kubectl-expose':
        Name of the manager used to track field ownership.

    -f, --filename=[]:
        Filename, directory, or URL to files identifying the resource to expose a service

    -k, --kustomize='':
        Process the kustomization directory. This flag can't be used together with -f or -R.

    -l, --labels='':
        Labels to apply to the service created by this call.

    --load-balancer-ip='':
        IP to assign to the LoadBalancer. If empty, an ephemeral IP will be created and used (cloud-provider
        specific).

    --name='':
        The name for the newly created object.

    -o, --output='':
        Output format. One of: (json, yaml, name, go-template, go-template-file, template, templatefile, jsonpath,
        jsonpath-as-json, jsonpath-file).

    --override-type='merge':
        The method used to override the generated object: json, merge, or strategic.

    --overrides='':
        An inline JSON override for the generated object. If this is non-empty, it is used to override the generated
        object. Requires that the object supply a valid apiVersion field.

    --port='':
        The port that the service should serve on. Copied from the resource being exposed, if unspecified

    --protocol='':
        The network protocol for the service to be created. Default is 'TCP'.

    -R, --recursive=false:
        Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests
        organized within the same directory.

    --save-config=false:
        If true, the configuration of current object will be saved in its annotation. Otherwise, the annotation will
        be unchanged. This flag is useful when you want to perform kubectl apply on this object in the future.

    --selector='':
        A label selector to use for this service. Only equality-based selector requirements are supported. If empty
        (the default) infer the selector from the replication controller or replica set.)

    --session-affinity='':
        If non-empty, set the session affinity for the service to this; legal values: 'None', 'ClientIP'

    --show-managed-fields=false:
        If true, keep the managedFields when printing objects in JSON or YAML format.

    --target-port='':
        Name or number for the port on the container that the service should direct traffic to. Optional.

    --template='':
        Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format
        is golang templates [http://golang.org/pkg/text/template/#pkg-overview].

    --type='':
        Type for this service: ClusterIP, NodePort, LoadBalancer, or ExternalName. Default is 'ClusterIP'.

Usage:
  kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP|SCTP] [--target-port=number-or-name]
[--name=name] [--external-ip=external-ip-of-service] [--type=type] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get nodes -o wide
NAME                                          STATUS   ROLES    AGE   VERSION               INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-echo-cluster-default-pool-15cbea64-l757   Ready    <none>   31m   v1.28.8-gke.1095000   10.150.0.5    34.145.144.92   Container-Optimized OS from Google   6.1.75+          containerd://1.7.13
gke-echo-cluster-default-pool-15cbea64-ldmp   Ready    <none>   31m   v1.28.8-gke.1095000   10.150.0.4    34.85.137.235   Container-Optimized OS from Google   6.1.75+          containerd://1.7.13
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
echo-web-5cd58676b4-jsjcd   1/1     Running   0          6m40s
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl describe pods echo-web-5cd58676b4-jsjcd
Name:             echo-web-5cd58676b4-jsjcd
Namespace:        default
Priority:         0
Service Account:  default
Node:             gke-echo-cluster-default-pool-15cbea64-l757/10.150.0.5
Start Time:       Tue, 21 May 2024 08:48:10 +0000
Labels:           app=echo-web
                  pod-template-hash=5cd58676b4
Annotations:      <none>
Status:           Running
IP:               10.88.1.11
IPs:
  IP:           10.88.1.11
Controlled By:  ReplicaSet/echo-web-5cd58676b4
Containers:
  echo-app:
    Container ID:   containerd://0da1200b4edb7e68a5e167fe662147006789d110b507dc9bcb9cf120901eadf3
    Image:          gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
    Image ID:       gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app@sha256:c8a231ae40d1fd98ef21353cd1e637c8ef2805bff87a272357d95e810ce93f75
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 21 May 2024 08:48:12 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rmrgp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-rmrgp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  6m51s  default-scheduler  Successfully assigned default/echo-web-5cd58676b4-jsjcd to gke-echo-cluster-default-pool-15cbea64-l757
  Normal  Pulling    6m50s  kubelet            Pulling image "gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1"
  Normal  Pulled     6m49s  kubelet            Successfully pulled image "gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1" in 1.575s (1.575s including waiting)
  Normal  Created    6m49s  kubelet            Created container echo-app
  Normal  Started    6m49s  kubelet            Started container echo-app
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl logs echo-web-5cd58676b4-jsjcd
2024/05/21 08:48:12 Server listening on port 8000
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ curl 0.88.1.11:8000
^C
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl logs echo-web-5cd58676b4-jsjcd
2024/05/21 08:48:12 Server listening on port 8000
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl logs echo-web-5cd58676b4-jsjcd
2024/05/21 08:48:12 Server listening on port 8000
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
echo-web     ClusterIP   10.84.142.174   <none>        80/TCP    4m37s
kubernetes   ClusterIP   10.84.128.1     <none>        443/TCP   35m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ 
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl expose deploy echo-web --port=80 --target-port=8000 --type=LoadBalancer
service/echo-web exposed
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   9s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   21s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   23s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   26s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   30s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   33s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   36s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   <pending>     80:32152/TCP   39s
kubernetes   ClusterIP      10.84.128.1    <none>        443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   34.48.102.246   80:32152/TCP   42s
kubernetes   ClusterIP      10.84.128.1    <none>          443/TCP        37m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ kubectl get svc
cuNAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
echo-web     LoadBalancer   10.84.138.89   34.48.102.246   80:32152/TCP   71s
kubernetes   ClusterIP      10.84.128.1    <none>          443/TCP        38m
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ curl 34.48.102.246
Echo Test
Version: 1.0.0
Hostname: echo-web-5cd58676b4-jsjcd
Host ip-address(es): 10.88.1.11
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ history 
    1  gcloud container clusters create echo-cluster --zone=us-east4-b --num-nodes 2 --machine-type=e2-standard-2
    2  gsutil cp gs://qwiklabs-gcp-02-b72e20704fdb/echo-web.tar.gz .
    3  ls
    4  tar -xvf echo-web.tar.gz 
    5  ls
    6  cat Dockerfile 
    7  gcloud auth configure-docker us-east4-docker.pkg.dev
    8  gcloud auth configure-docker 
    {snip}
   27  docker images
   28  docker build -t gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1 .
   29  docker push gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
   30  kubectl get nodes
   31  kubectl get deploy
   32  kubectl create deploy -h
   33  kubectl create deployment echo-web --image=gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
   34  kubectl get pods
   35  kubectl get pods -o wide
   36  curl 10.88.1.11:8080
   37  curl 10.88.1.11:8000
   38  docker run -p 8000:8000 --name echo-app -d gcr.io/qwiklabs-gcp-02-b72e20704fdb/echo-app:v1
   {snip}
   43  kubectl get deploy
   44  kubectl expose deploy echo-web -h
   
   {snip}
   58  kubectl expose deploy echo-web --port=80 --target-port=8000 --type=LoadBalancer
   59  kubectl get svc
   60  curl 34.48.102.246
   61  history 
student_02_0bc8a0c336eb@cloudshell:~ (qwiklabs-gcp-02-b72e20704fdb)$ 
```