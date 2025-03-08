---
layout: single
title:  "Getting Started with Linkerd Service Mesh"
categories: Kubernetes
tags: Linkerd
classes: wide
show_date: true
header:
  overlay_image: /assets/images/linkerd.png
  og_image: /assets/images/linkerd.png
  teaser: /assets/images/linkerd.png
  caption: "Photo credit: [**Linkerd**](https://linkerd.io)"
  actions:
    - label: "Learn more"
      url: "https://linkerd.io"

author:
  name     : "Linkerd"
  avatar   : "/assets/images/linkerd.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

# Getting Started with Linkerd Service Mesh

```sh
(base) pradeep:~$minikube status
🤷  Profile "minikube" not found. Run "minikube profile list" to view all profiles.
👉  To start a cluster, run: "minikube start"
(base) pradeep:~$minikube start --container-runtime=containerd
😄  minikube v1.33.0 on Darwin 14.3
🎉  minikube 1.34.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.34.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

✨  Automatically selected the hyperkit driver. Other choices: virtualbox, ssh
👍  Starting "minikube" primary control-plane node in "minikube" cluster
💾  Downloading Kubernetes v1.30.0 preload ...
    > preloaded-images-k8s-v18-v1...:  375.69 MiB / 375.69 MiB  100.00% 20.38 M
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🔥  Deleting "minikube" in hyperkit ...
🤦  StartHost failed, but will try again: creating host: create: Error creating machine: Error in driver during machine creation: IP address never found in dhcp leases file Temporary error: could not find an IP address for 32:40:c4:ba:39:2
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
📦  Preparing Kubernetes v1.30.0 on containerd 1.7.15 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/local/bin/kubectl is version 1.22.5, which may have incompatibilities with Kubernetes 1.30.0.
    ▪ Want kubectl v1.30.0? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~$kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   70s   v1.30.0
(base) pradeep:~$kubectl get pods -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-7db6d8ff4d-l7hhz           1/1     Running   0             53s
kube-system   etcd-minikube                      1/1     Running   0             69s
kube-system   kube-apiserver-minikube            1/1     Running   0             69s
kube-system   kube-controller-manager-minikube   1/1     Running   0             69s
kube-system   kube-proxy-r9rg5                   1/1     Running   0             53s
kube-system   kube-scheduler-minikube            1/1     Running   0             75s
kube-system   storage-provisioner                1/1     Running   1 (21s ago)   64s
(base) pradeep:~$
```


```sh
(base) pradeep:~$kubectl version
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.5", GitCommit:"5c99e2ac2ff9a3c549d9ca665e7bc05a3e18f07e", GitTreeState:"clean", BuildDate:"2021-12-16T08:38:33Z", GoVersion:"go1.16.12", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.0", GitCommit:"7c48c2bd72b9bf5c44d21d7338cc7bea77d0ad2a", GitTreeState:"clean", BuildDate:"2024-04-17T17:27:03Z", GoVersion:"go1.22.2", Compiler:"gc", Platform:"linux/amd64"}
WARNING: version difference between client (1.22) and server (1.30) exceeds the supported minor version skew of +/-1
(base) pradeep:~$
```

## Install Linkerd CLI

```sh
(base) pradeep:~$curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install-edge | sh

Downloading linkerd2-cli-edge-24.11.8-darwin...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 61.5M  100 61.5M    0     0  14.5M      0  0:00:04  0:00:04 --:--:-- 21.3M
Download complete!

Validating checksum...
Checksum valid.

Linkerd edge-24.11.8 was successfully installed 🎉


Add the linkerd CLI to your path with:

  export PATH=$PATH:/Users/pradeep/.linkerd2/bin

Now run:

  linkerd check --pre                         # validate that Linkerd can be installed
  linkerd install --crds | kubectl apply -f - # install the Linkerd CRDs
  linkerd install | kubectl apply -f -        # install the control plane into the 'linkerd' namespace
  linkerd check                               # validate everything worked!

You can also obtain observability features by installing the viz extension:

  linkerd viz install | kubectl apply -f -  # install the viz extension into the 'linkerd-viz' namespace
  linkerd viz check                         # validate the extension works!
  linkerd viz dashboard                     # launch the dashboard

Looking for more? Visit https://linkerd.io/2/tasks

(base) pradeep:~$export PATH=$PATH:/Users/pradeep/.linkerd2/bin
(base) pradeep:~$
```

Verify version
```sh
(base) pradeep:~$linkerd version
Client version: edge-24.11.8
Server version: unavailable
(base) pradeep:~$
```

You should see the CLI version, and also Server version: unavailable. This is because you haven’t installed the control plane on your cluster

## Validate your Kubernetes cluster
Kubernetes clusters can be configured in many different ways. Before we can install the Linkerd control plane, we need to check and validate that everything is configured correctly. To check that your cluster is ready to install Linkerd, run:

```sh
(base) pradeep:~$linkerd check --pre

kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

pre-kubernetes-setup
--------------------
√ control plane namespace does not already exist
√ can create non-namespaced resources
√ can create ServiceAccounts
√ can create Services
√ can create Deployments
√ can create CronJobs
√ can create ConfigMaps
√ can create Secrets
√ can read Secrets
√ can read extension-apiserver-authentication configmap
√ no clock skew detected

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

Status check results are √
(base) pradeep:~$
```

## Install Linkerd onto your cluster

Now that you have the CLI running locally and a cluster that is ready to go, it’s time to install Linkerd on your Kubernetes cluster. To do this, run:

```sh
(base) pradeep:~$linkerd install --crds | kubectl apply -f -

Rendering Linkerd CRDs...
Next, run `linkerd install | kubectl apply -f -` to install the control plane.

customresourcedefinition.apiextensions.k8s.io/authorizationpolicies.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/egressnetworks.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/httplocalratelimitpolicies.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/meshtlsauthentications.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/networkauthentications.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/serverauthorizations.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/servers.policy.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/serviceprofiles.linkerd.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tlsroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/tcproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/externalworkloads.workload.linkerd.io created
(base) pradeep:~$
```


```sh
(base) pradeep:~$linkerd install | kubectl apply -f -

namespace/linkerd created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-identity created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-identity created
serviceaccount/linkerd-identity created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-destination created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-destination created
serviceaccount/linkerd-destination created
secret/linkerd-sp-validator-k8s-tls created
validatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-sp-validator-webhook-config created
secret/linkerd-policy-validator-k8s-tls created
validatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-policy-validator-webhook-config created
clusterrole.rbac.authorization.k8s.io/linkerd-policy created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-destination-policy created
role.rbac.authorization.k8s.io/remote-discovery created
rolebinding.rbac.authorization.k8s.io/linkerd-destination-remote-discovery created
role.rbac.authorization.k8s.io/linkerd-heartbeat created
rolebinding.rbac.authorization.k8s.io/linkerd-heartbeat created
clusterrole.rbac.authorization.k8s.io/linkerd-heartbeat created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-heartbeat created
serviceaccount/linkerd-heartbeat created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-proxy-injector created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-proxy-injector created
serviceaccount/linkerd-proxy-injector created
secret/linkerd-proxy-injector-k8s-tls created
mutatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-proxy-injector-webhook-config created
configmap/linkerd-config created
role.rbac.authorization.k8s.io/ext-namespace-metadata-linkerd-config created
secret/linkerd-identity-issuer created
configmap/linkerd-identity-trust-roots created
service/linkerd-identity created
service/linkerd-identity-headless created
deployment.apps/linkerd-identity created
service/linkerd-dst created
service/linkerd-dst-headless created
service/linkerd-sp-validator created
service/linkerd-policy created
service/linkerd-policy-validator created
deployment.apps/linkerd-destination created
cronjob.batch/linkerd-heartbeat created
deployment.apps/linkerd-proxy-injector created
service/linkerd-proxy-injector created
secret/linkerd-config-overrides created
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get all -n linkerd
NAME                                          READY   STATUS            RESTARTS   AGE
pod/linkerd-destination-5f77fd8cd7-q4629      0/4     PodInitializing   0          65s
pod/linkerd-identity-59457ffdf8-82lmt         2/2     Running           0          67s
pod/linkerd-proxy-injector-779b8d5b6c-r7zwg   0/2     PodInitializing   0          63s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/linkerd-dst                 ClusterIP   10.110.149.162   <none>        8086/TCP   68s
service/linkerd-dst-headless        ClusterIP   None             <none>        8086/TCP   67s
service/linkerd-identity            ClusterIP   10.104.20.117    <none>        8080/TCP   69s
service/linkerd-identity-headless   ClusterIP   None             <none>        8080/TCP   69s
service/linkerd-policy              ClusterIP   None             <none>        8090/TCP   66s
service/linkerd-policy-validator    ClusterIP   10.110.1.191     <none>        443/TCP    66s
service/linkerd-proxy-injector      ClusterIP   10.100.186.102   <none>        443/TCP    63s
service/linkerd-sp-validator        ClusterIP   10.97.138.232    <none>        443/TCP    67s

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/linkerd-destination      0/1     1            0           65s
deployment.apps/linkerd-identity         1/1     1            1           69s
deployment.apps/linkerd-proxy-injector   0/1     1            0           64s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/linkerd-destination-5f77fd8cd7      1         1         0       65s
replicaset.apps/linkerd-identity-59457ffdf8         1         1         1       68s
replicaset.apps/linkerd-proxy-injector-779b8d5b6c   1         1         0       63s

NAME                              SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/linkerd-heartbeat   40 15 * * *   <none>     False     0        <none>          65s
(base) pradeep:~$
```

Depending on the speed of your cluster’s Internet connection, it may take a minute or two for the control plane to finish installing. Wait for the control plane to be ready (and verify your installation) by running:
```sh
(base) pradeep:~$linkerd check
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ control plane pods are ready
√ cluster networks contains all node podCIDRs
√ cluster networks contains all pods
√ cluster networks contains all services

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ proxy-init container runs as root user if docker container runtime is used

linkerd-identity
----------------
√ certificate config is valid
√ trust anchors are using supported crypto algorithm
√ trust anchors are within their validity period
√ trust anchors are valid for at least 60 days
√ issuer cert is using supported crypto algorithm
√ issuer cert is within its validity period
√ issuer cert is valid for at least 60 days
√ issuer cert is issued by the trust anchor

linkerd-webhooks-and-apisvc-tls
-------------------------------
√ proxy-injector webhook has valid cert
√ proxy-injector cert is valid for at least 60 days
√ sp-validator webhook has valid cert
√ sp-validator cert is valid for at least 60 days
√ policy-validator webhook has valid cert
√ policy-validator cert is valid for at least 60 days

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ can retrieve the control plane version
√ control plane is up-to-date
√ control plane and cli versions match

linkerd-control-plane-proxy
---------------------------
√ control plane proxies are healthy
√ control plane proxies are up-to-date
√ control plane proxies and cli versions match

linkerd-extension-checks
------------------------
√ namespace configuration for extensions

Status check results are √
(base) pradeep:~$
```

Let’s install a demo application called Emojivoto. Emojivoto is a simple standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the user to vote on their favorite emojis.

Install Emojivoto into the emojivoto namespace by running:

```sh
(base) pradeep:~$curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -

namespace/emojivoto created
serviceaccount/emoji created
serviceaccount/voting created
serviceaccount/web created
service/emoji-svc created
service/voting-svc created
service/web-svc created
deployment.apps/emoji created
deployment.apps/vote-bot created
deployment.apps/voting created
deployment.apps/web created
(base) pradeep:~$    
```

```sh
(base) pradeep:~$kubectl get all -n emojivoto
NAME                            READY   STATUS              RESTARTS   AGE
pod/emoji-97f8b9d7b-9mmxz       0/1     ContainerCreating   0          35s
pod/vote-bot-6988f549dc-46jdk   0/1     ContainerCreating   0          35s
pod/voting-589cdb687-tg6ss      0/1     ContainerCreating   0          35s
pod/web-6dcb9b6479-9fxvk        0/1     ContainerCreating   0          35s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/emoji-svc    ClusterIP   10.106.217.229   <none>        8080/TCP,8801/TCP   37s
service/voting-svc   ClusterIP   10.98.247.254    <none>        8080/TCP,8801/TCP   36s
service/web-svc      ClusterIP   10.105.158.156   <none>        80/TCP              36s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/emoji      0/1     1            0           39s
deployment.apps/vote-bot   0/1     1            0           39s
deployment.apps/voting     0/1     1            0           39s
deployment.apps/web        0/1     1            0           39s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/emoji-97f8b9d7b       1         1         0       39s
replicaset.apps/vote-bot-6988f549dc   1         1         0       39s
replicaset.apps/voting-589cdb687      1         1         0       39s
replicaset.apps/web-6dcb9b6479        1         1         0       39s
(base) pradeep:~$
```

This command installs Emojivoto onto your cluster, but Linkerd hasn’t been activated on it yet—we’ll need to “mesh” the application before Linkerd can work its magic.

Before we mesh it, let’s take a look at Emojivoto in its natural state. We’ll do this by forwarding traffic to its web-svc service so that we can point our browser to it. Forward web-svc locally to port 8080 by running:

```sh
(base) pradeep:~$kubectl get all -n emojivoto                         
NAME                            READY   STATUS    RESTARTS   AGE
pod/emoji-97f8b9d7b-9mmxz       1/1     Running   0          2m50s
pod/vote-bot-6988f549dc-46jdk   1/1     Running   0          2m50s
pod/voting-589cdb687-tg6ss      1/1     Running   0          2m50s
pod/web-6dcb9b6479-9fxvk        1/1     Running   0          2m50s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/emoji-svc    ClusterIP   10.106.217.229   <none>        8080/TCP,8801/TCP   2m51s
service/voting-svc   ClusterIP   10.98.247.254    <none>        8080/TCP,8801/TCP   2m50s
service/web-svc      ClusterIP   10.105.158.156   <none>        80/TCP              2m50s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/emoji      1/1     1            1           2m51s
deployment.apps/vote-bot   1/1     1            1           2m51s
deployment.apps/voting     1/1     1            1           2m51s
deployment.apps/web        1/1     1            1           2m51s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/emoji-97f8b9d7b       1         1         1       2m51s
replicaset.apps/vote-bot-6988f549dc   1         1         1       2m51s
replicaset.apps/voting-589cdb687      1         1         1       2m51s
replicaset.apps/web-6dcb9b6479        1         1         1       2m51s
(base) pradeep:~$kubectl -n emojivoto port-forward svc/web-svc 8181:80

Forwarding from 127.0.0.1:8181 -> 8080
Forwarding from [::1]:8181 -> 8080
Handling connection for 8181
Handling connection for 8181


```
With Emoji installed and running, we’re ready to mesh it—that is, to add Linkerd’s data plane proxies to it. We can do this on a live application without downtime, thanks to Kubernetes’s rolling deploys. Mesh your Emojivoto application by running:

## Mesh your Application

```sh
(base) pradeep:~$kubectl get -n emojivoto deploy -o yaml \      
  | linkerd inject - \
  | kubectl apply -f -


deployment "emoji" injected
deployment "vote-bot" injected
deployment "voting" injected
deployment "web" injected

deployment.apps/emoji configured
deployment.apps/vote-bot configured
deployment.apps/voting configured
deployment.apps/web configured
(base) pradeep:~$
```
This command retrieves all of the deployments running in the emojivoto namespace, runs their manifests through linkerd inject, and then reapplies it to the cluster. (The linkerd inject command simply adds annotations to the pod spec that instruct Linkerd to inject the proxy into the pods when they are created.)

As with install, inject is a pure text operation, meaning that you can inspect the input and output before you use it. Once piped into kubectl apply, Kubernetes will execute a rolling deploy and update each pod with the data plane’s proxies.

Congratulations! You’ve now added Linkerd to an application! Just as with the control plane, it’s possible to verify that everything is working the way it should on the data plane side. Check your data plane with:

## Verify Data Plane

```sh
(base) pradeep:~$linkerd -n emojivoto check --proxy

kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ control plane pods are ready
√ cluster networks contains all node podCIDRs
√ cluster networks contains all pods
√ cluster networks contains all services

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ proxy-init container runs as root user if docker container runtime is used

linkerd-identity
----------------
√ certificate config is valid
√ trust anchors are using supported crypto algorithm
√ trust anchors are within their validity period
√ trust anchors are valid for at least 60 days
√ issuer cert is using supported crypto algorithm
√ issuer cert is within its validity period
√ issuer cert is valid for at least 60 days
√ issuer cert is issued by the trust anchor

linkerd-webhooks-and-apisvc-tls
-------------------------------
√ proxy-injector webhook has valid cert
√ proxy-injector cert is valid for at least 60 days
√ sp-validator webhook has valid cert
√ sp-validator cert is valid for at least 60 days
√ policy-validator webhook has valid cert
√ policy-validator cert is valid for at least 60 days

linkerd-identity-data-plane
---------------------------
√ data plane proxies certificate match CA

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

linkerd-control-plane-proxy
---------------------------
√ control plane proxies are healthy
√ control plane proxies are up-to-date
√ control plane proxies and cli versions match

linkerd-data-plane
------------------
√ data plane namespace exists
√ data plane proxies are ready
√ data plane is up-to-date
√ data plane and cli versions match
√ data plane pod labels are configured correctly
√ data plane service labels are configured correctly
√ data plane service annotations are configured correctly
√ opaque ports are properly annotated

Status check results are √
(base) pradeep:~$
```

Let’s install the viz extension, which will install an on-cluster metric stack and dashboard.

To install the viz extension, run:

```sh
(base) pradeep:~$linkerd viz install | kubectl apply -f -
namespace/linkerd-viz created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-metrics-api created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-metrics-api created
serviceaccount/metrics-api created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-prometheus created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-prometheus created
serviceaccount/prometheus created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-tap created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-tap-admin created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-tap created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-tap-auth-delegator created
serviceaccount/tap created
rolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-tap-auth-reader created
secret/tap-k8s-tls created
apiservice.apiregistration.k8s.io/v1alpha1.tap.linkerd.io created
role.rbac.authorization.k8s.io/web created
rolebinding.rbac.authorization.k8s.io/web created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-web-check created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-web-check created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-web-admin created
clusterrole.rbac.authorization.k8s.io/linkerd-linkerd-viz-web-api created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-linkerd-viz-web-api created
serviceaccount/web created
service/metrics-api created
deployment.apps/metrics-api created
server.policy.linkerd.io/metrics-api created
authorizationpolicy.policy.linkerd.io/metrics-api created
meshtlsauthentication.policy.linkerd.io/metrics-api-web created
networkauthentication.policy.linkerd.io/kubelet created
configmap/prometheus-config created
service/prometheus created
deployment.apps/prometheus created
server.policy.linkerd.io/prometheus-admin created
authorizationpolicy.policy.linkerd.io/prometheus-admin created
service/tap created
deployment.apps/tap created
server.policy.linkerd.io/tap-api created
authorizationpolicy.policy.linkerd.io/tap created
clusterrole.rbac.authorization.k8s.io/linkerd-tap-injector created
clusterrolebinding.rbac.authorization.k8s.io/linkerd-tap-injector created
serviceaccount/tap-injector created
secret/tap-injector-k8s-tls created
mutatingwebhookconfiguration.admissionregistration.k8s.io/linkerd-tap-injector-webhook-config created
service/tap-injector created
deployment.apps/tap-injector created
server.policy.linkerd.io/tap-injector-webhook created
authorizationpolicy.policy.linkerd.io/tap-injector created
networkauthentication.policy.linkerd.io/kube-api-server created
service/web created
deployment.apps/web created
serviceprofile.linkerd.io/metrics-api.linkerd-viz.svc.cluster.local created
serviceprofile.linkerd.io/prometheus.linkerd-viz.svc.cluster.local created
(base) pradeep:~$

```

```sh
(base) pradeep:~$linkerd check
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ control plane pods are ready
√ cluster networks contains all node podCIDRs
√ cluster networks contains all pods
√ cluster networks contains all services

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ proxy-init container runs as root user if docker container runtime is used

linkerd-identity
----------------
√ certificate config is valid
√ trust anchors are using supported crypto algorithm
√ trust anchors are within their validity period
√ trust anchors are valid for at least 60 days
√ issuer cert is using supported crypto algorithm
√ issuer cert is within its validity period
√ issuer cert is valid for at least 60 days
√ issuer cert is issued by the trust anchor

linkerd-webhooks-and-apisvc-tls
-------------------------------
√ proxy-injector webhook has valid cert
√ proxy-injector cert is valid for at least 60 days
√ sp-validator webhook has valid cert
√ sp-validator cert is valid for at least 60 days
√ policy-validator webhook has valid cert
√ policy-validator cert is valid for at least 60 days

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ can retrieve the control plane version
√ control plane is up-to-date
√ control plane and cli versions match

linkerd-control-plane-proxy
---------------------------
√ control plane proxies are healthy
√ control plane proxies are up-to-date
√ control plane proxies and cli versions match

linkerd-extension-checks
------------------------
√ namespace configuration for extensions

linkerd-viz
-----------
√ linkerd-viz Namespace exists
√ can initialize the client
√ linkerd-viz ClusterRoles exist
√ linkerd-viz ClusterRoleBindings exist
√ tap API server has valid cert
√ tap API server cert is valid for at least 60 days
√ tap API service is running
√ linkerd-viz pods are injected
√ viz extension pods are running
√ viz extension proxies are healthy
√ viz extension proxies are up-to-date
√ viz extension proxies and cli versions match
√ prometheus is installed and configured correctly
√ viz extension self-check

Status check results are √
(base) pradeep:~$

```

With the control plane and extensions installed and running, we’re now ready to explore Linkerd! Access the dashboard with:

```sh
(base) pradeep:~$Linkerd dashboard available at:
http://localhost:50750
Grafana dashboard available at:
http://localhost:50750/grafana
Opening Linkerd dashboard in the default browser

```

That's it. A quick intro on the Linkerd Service Mesh.

