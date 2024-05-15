---

layout: single
title:  "Setting up Jenkins on Kubernetes Engine"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
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

# Setting up Jenkins on Kubernetes Engine

- Creating a Kubernetes cluster with Kubernetes Engine.
- Creating a Jenkins deployment and services.
- Connecting to Jenkins.

## Prepare the environment

First, you'll prepare your deployment environment and download a sample application. Next, provision a Kubernetes cluster using Kubernetes Engine. The extra scopes enable Jenkins to access Cloud Source Repositories and Google Container Registry.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-70469b8465ce.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-70469b8465ce)$ gcloud config set compute/zone us-east4-b
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-70469b8465ce)$ git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git
Cloning into 'continuous-deployment-on-kubernetes'...
remote: Enumerating objects: 971, done.
remote: Counting objects: 100% (147/147), done.
remote: Compressing objects: 100% (25/25), done.
remote: Total 971 (delta 133), reused 122 (delta 122), pack-reused 824
Receiving objects: 100% (971/971), 1.91 MiB | 21.94 MiB/s, done.
Resolving deltas: 100% (485/485), done.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-70469b8465ce)$ cd continuous-deployment-on-kubernetes
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster jenkins-cd in us-east4-b... Cluster is being health-checked (master is healthy)...done.                                                                           
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-70469b8465ce/zones/us-east4-b/clusters/jenkins-cd].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east4-b/jenkins-cd?project=qwiklabs-gcp-03-70469b8465ce
kubeconfig entry generated for jenkins-cd.
NAME: jenkins-cd
LOCATION: us-east4-b
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 35.188.244.2
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 2
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```



```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ gcloud container clusters list
NAME: jenkins-cd
LOCATION: us-east4-b
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 35.188.244.2
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 2
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ gcloud container clusters get-credentials jenkins-cd
Fetching cluster endpoint and auth data.
kubeconfig entry generated for jenkins-cd.
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ kubectl cluster-info
Kubernetes control plane is running at https://35.188.244.2
GLBCDefaultBackend is running at https://35.188.244.2/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
KubeDNS is running at https://35.188.244.2/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://35.188.244.2/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```



## Configure Helm

In this lab, you will use Helm to install Jenkins from the Charts  repository. Helm is a package manager that makes it easy to configure  and deploy Kubernetes applications. Your Cloud Shell will already have a recent, stable version of Helm pre-installed.

If curious, you can run `helm version` in Cloud Shell to check which version you are using and also ensure that Helm is installed.

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ helm version
version.BuildInfo{Version:"v3.9.3", GitCommit:"414ff28d4029ae8c8b05d62aa06c7fe3dee2bc58", GitTreeState:"clean", GoVersion:"go1.17.13"}
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ helm repo add jenkins https://charts.jenkins.io
"jenkins" has been added to your repositories
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$
```



```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
Update Complete. ⎈Happy Helming!⎈
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```



## Configure and install Jenkins

You will use a custom values file to add the Google Cloud specific  plugin necessary to use service account credentials to reach your Cloud  Source Repository.

1. Use the Helm CLI to deploy the chart with your configuration set:

```yaml
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ cat jenkins/values.yaml 
controller:
  installPlugins:
    - kubernetes:latest
    - workflow-job:latest
    - workflow-aggregator:latest
    - credentials-binding:latest
    - git:latest
    - google-oauth-plugin:latest
    - google-source-plugin:latest
    - google-kubernetes-engine:latest
    - google-storage-plugin:latest
  resources:
    requests:
      cpu: "50m"
      memory: "1024Mi"
    limits:
      cpu: "1"
      memory: "3500Mi"
  javaOpts: "-Xms3500m -Xmx3500m"
  serviceType: ClusterIP
agent:
  resources:
    requests:
      cpu: "500m"
      memory: "256Mi"
    limits:
      cpu: "1"
      memory: "512Mi"
persistence:
  size: 100Gi
serviceAccount:
  name: cd-jenkins
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```



```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ ls jenkins/
values.yaml
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$
```

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ helm upgrade --install -f jenkins/values.yaml myjenkins jenkins/jenkins
Release "myjenkins" does not exist. Installing it now.
NAME: myjenkins
LAST DEPLOYED: Wed May 15 15:43:10 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:
  kubectl exec --namespace default -it svc/myjenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  echo http://127.0.0.1:8080
  kubectl --namespace default port-forward svc/myjenkins 8080:8080

3. Login with the password from step 1 and the username: admin
4. Configure security realm and authorization strategy
5. Use Jenkins Configuration as Code by specifying configScripts in your values.yaml file, see documentation: http://127.0.0.1:8080/configuration-as-code and examples: https://github.com/jenkinsci/configuration-as-code-plugin/tree/master/demos

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine

For more information about Jenkins Configuration as Code, visit:
https://jenkins.io/projects/jcasc/


NOTE: Consider using a custom image with pre-installed plugins
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
myjenkins-0   2/2     Running   0          111s
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```
Run the following command to setup port forwarding to the Jenkins UI from the Cloud Shell:
```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ echo http://127.0.0.1:8080
kubectl --namespace default port-forward svc/myjenkins 8080:8080 >> /dev/null &
http://127.0.0.1:8080
[1] 2043
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ kubectl get svc
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.40.240.1    <none>        443/TCP     13m
myjenkins         ClusterIP   10.40.254.70   <none>        8080/TCP    3m16s
myjenkins-agent   ClusterIP   10.40.245.49   <none>        50000/TCP   3m16s
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```

We are using the [Kubernetes Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin) so that our builder nodes will be automatically launched as necessary  when the Jenkins master requests them. Upon completion of their work,  they will automatically be turned down and their resources added back to the clusters resource pool.

Notice that this service exposes ports `8080` and `50000` for any pods that match the `selector`. This will expose the Jenkins web UI and builder/agent registration ports within the Kubernetes cluster.

Additionally, the `jenkins-ui` service is exposed using a ClusterIP so that it is not accessible from outside the cluster.

## Connect to Jenkins

1. The Jenkins chart will automatically create an admin password for you. To retrieve it, run:

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ kubectl exec --namespace default -it svc/myjenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
MMaNj2NgEyfsEb6FvnJAB3
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```

To get to the Jenkins user interface, click on the **Web Preview** button in cloud shell, then click **Preview on port 8080**:

You should now be able to log in with the username `admin` and your auto-generated password.

You now have Jenkins set up in your Kubernetes cluster!

## Summary

```sh
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ history 
    1  gcloud spanner databases describe example-db --instance test-instance 
    2  gcloud config set compute/zone us-east4-b
    3  git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git
    4  cd continuous-deployment-on-kubernetes
    5  gcloud container clusters create jenkins-cd --num-nodes 2 --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
    6  gcloud container clusters list
    7  gcloud container clusters get-credentials jenkins-cd
    8  kubectl cluster-info
    9  helm version
   10  helm repo add jenkins https://charts.jenkins.io
  
   11  helm repo update
   12  ls
   13  cat jenkins/values.yaml 
   14  ls jenkins/
   15  helm upgrade --install -f jenkins/values.yaml myjenkins jenkins/jenkins
   16  kubectl get pods
   17  kubectl get pods
   18  kubectl get pods
   19  kubectl get pods
   20  echo http://127.0.0.1:8080
   21  kubectl --namespace default port-forward svc/myjenkins 8080:8080 >> /dev/null &
   22  kubectl get svc
   23  kubectl exec --namespace default -it svc/myjenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
   24  kubectl get pods -A
   25  history 
student_01_aa8f3a74b714@cloudshell:~/continuous-deployment-on-kubernetes (qwiklabs-gcp-03-70469b8465ce)$ 
```

