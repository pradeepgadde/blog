---
layout: single
title:  "Getting Started with Helm: Kubernetes Package Manager"
date:   2022-06-24 10:55:04 +0530
categories: Kubernetes
tags: minikube
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  teaser: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"

author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---

# Getting Started with Helm
Helm is a package manager for Kubernetes.

Using Helm, we can find, share, and use software built for Kubernetes.

```sh
$ kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   80s   v1.23.4
node01         Ready    <none>                 56s   v1.23.4
$ 
```

```sh
$ kubectl version --short
Client Version: v1.23.4
Server Version: v1.23.4
$ kubectl cluster-info 
Kubernetes control plane is running at https://172.23.92.5:6443
CoreDNS is running at https://172.23.92.5:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ 
```

## Install Helm

Helm is a cluster administration tool that manages *charts* on Kubernetes.

Helm relies on a packaging format called *charts*.

Charts define a composition of related Kubernetes resources and values that make up a deployment solution.

The Helm CLI tool deploys charts to Kubernetes.

```sh
$ helm version 
version.BuildInfo{Version:"v3.8.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.5"}
$ helm version --short
v3.8.0+gd141386
$ 
```

Install the latest version using the `curl` script.

```sh
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
Helm v3.9.0 is available. Changing from version v3.8.0.
Downloading https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
$ 
```

After install,

## Helm Version

```sh
$ helm version --short
v3.9.0+g7ceeda6
$ 
```
Use `helm env` to print all environment variables.

## Helm Env

```sh
$ helm env
HELM_BIN="helm"
HELM_CACHE_HOME="/root/.cache/helm"
HELM_CONFIG_HOME="/root/.config/helm"
HELM_DATA_HOME="/root/.local/share/helm"
HELM_DEBUG="false"
HELM_KUBEAPISERVER=""
HELM_KUBEASGROUPS=""
HELM_KUBEASUSER=""
HELM_KUBECAFILE=""
HELM_KUBECONTEXT=""
HELM_KUBETOKEN=""
HELM_MAX_HISTORY="10"
HELM_NAMESPACE="default"
HELM_PLUGINS="/root/.local/share/helm/plugins"
HELM_REGISTRY_CONFIG="/root/.config/helm/registry/config.json"
HELM_REPOSITORY_CACHE="/root/.cache/helm/repository"
HELM_REPOSITORY_CONFIG="/root/.config/helm/repositories.yaml"
$ 
```

## Artifact Hub

Now, the canonical source for cloud native artifacts, and specifically Helm charts, is [Artifact Hub](https://artifacthub.io/), an aggregator for distributed chart repos. Artifact Hub is the replacement for Helm Hub.

## Helm Search

```sh
$ helm search hub | wc -l
8446
$ 
```

Search the Hub for a specific chart, for example, search for `Clair`.

```sh
$ helm search hub clair
URL                                                     CHART VERSION   APP VERSION     DESCRIPTION                                       
https://artifacthub.io/packages/helm/devtron-la...      0.1.5           3.0.0-pre       Clair is an open source project for the static ...
https://artifacthub.io/packages/helm/wiremind/c...      0.2.10-dev0     2.1.6           Clair is an open source project for the static ...
https://artifacthub.io/packages/helm/devtron/clair      0.1.5           3.0.0-pre       Clair is an open source project for the static ...
https://artifacthub.io/packages/helm/banzaiclou...      0.1.5           v2.0.9          DEPRECATED: Clair is an open source project for...
$ 
```

Another example, search for `Grafana`

```sh
$ helm search hub grafana | wc -l
124
$ 
```

One more, search for `Redis`, there seems to be 66 packages.

```sh
$ helm search hub redis
URL                                                     CHART VERSION   APP VERSION             DESCRIPTION                                       
https://artifacthub.io/packages/helm/redis/redis        0.1.1           6.0.8.9                 A Helm chart for Redis.                           
https://artifacthub.io/packages/helm/wener/redis        16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/pascaliske...      0.0.3           6.2.6                   A Helm chart for Redis                            
https://artifacthub.io/packages/helm/choerodon/...      16.4.1          6.2.6                   Redis(TM) is an open source, advanced key-value...
https://artifacthub.io/packages/helm/bitnami-ak...      16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/bitnami/redis      16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/wenerme/redis      16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/mmontes/redis      0.3.0           1.16.0                  Redis with metrics compatible with ARM            
https://artifacthub.io/packages/helm/conduction...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/cloudnativ...      8.0.1           5.0.5                   Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/drycc-cana...      1.0.0                                   A Redis database for use inside a Kubernetes cl...
https://artifacthub.io/packages/helm/truecharts...      3.0.17          7.0.2                   Open source, advanced key-value store.            
https://artifacthub.io/packages/helm/riftbit/redis      15.4.0          6.2.5                   Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/notificati...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/agendaserv...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/authorizat...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/contacten-...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/taalhuizen...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/jfrog/redis        12.10.1         6.0.12                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/proto-appl...      12.7.7          6.0.11                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/openstack-...      0.1.1           v4.0.1                  OpenStack-Helm Redis                              
https://artifacthub.io/packages/helm/kubesphere...      0.3.5           6.0.9                   Redis is an open source (BSD licensed), in-memo...
https://artifacthub.io/packages/helm/appuio/redis       1.3.5           6.2.1                   Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/groundhog2...      0.5.2           7.0.2                   A Helm chart for Redis on Kubernetes              
https://artifacthub.io/packages/helm/drycc/redis        1.2.0                                   A Redis database for use inside a Kubernetes cl...
https://artifacthub.io/packages/helm/kubegemsap...      12.8.3          6.0.12                  Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/redis-char...      0.1.0           v0.2.0                  A Helm chart for redis-cart service               
https://artifacthub.io/packages/helm/cloudnativ...      0.4.1           4.0.12-alpine           A pure in-memory redis cache, using statefulset...
https://artifacthub.io/packages/helm/bitnami/re...      7.6.3           6.2.7                   Redis(R) is an open source, scalable, distribut...
https://artifacthub.io/packages/helm/bitnami-ak...      7.6.3           6.2.7                   Redis(R) is an open source, scalable, distribut...
https://artifacthub.io/packages/helm/inspur/red...      0.0.2           5.0.6                   Highly available Kubernetes implementation of R...
https://artifacthub.io/packages/helm/kubesphere...      3.4.6           1.3.4                   Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/dandydev-c...      4.16.0          6.2.5                   This Helm chart provides a highly available Red...
https://artifacthub.io/packages/helm/cloudnativ...      3.4.2           5.0.3                   Highly available Kubernetes implementation of R...
https://artifacthub.io/packages/helm/hkube/redi...      3.6.1005        5.0.5                   Highly available Kubernetes implementation of R...
https://artifacthub.io/packages/helm/kfirfer/re...      4.12.9          6.0.11                  This Helm chart provides a highly available Red...
https://artifacthub.io/packages/helm/aigisuk/re...      0.1.9           1.7.4.2-r0              A lightweight redis proxy deployment/daemonset ...
https://artifacthub.io/packages/helm/ygqygq2/re...      1.0.0           v1.5.2-alpine           A Helm chart servicemonitor for redis exporter    
https://artifacthub.io/packages/helm/softonic/r...      0.3.0           6.0.6                   A Helm chart for sharded redis                    
https://artifacthub.io/packages/helm/riftbit/re...      6.3.7           6.2.5                   Open source, advanced key-value store. It is of...
https://artifacthub.io/packages/helm/kfirfer/re...      0.1.3           latest                  A Helm chart for redis-commander                  
https://artifacthub.io/packages/helm/openshift/...      1.0.1           1.0.1                   A Helm chart for Redis Service Endpoint Definit...
https://artifacthub.io/packages/helm/tyk-helm/s...      0.1.1                                   A Simple Helm chart for running redis for CI      
https://artifacthub.io/packages/helm/wenerme/pr...      4.8.0           1.27.0                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/cloudnativ...      1.0.2           0.28.0                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/prometheus...      4.8.0           1.27.0                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/prometheus...      4.0.0           1.11.1                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/wener/prom...      4.8.0           1.27.0                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/kubegemsap...      4.6.0           1.27.0                  Prometheus exporter for Redis metrics             
https://artifacthub.io/packages/helm/hmdmph/red...      1.0.2           1.0.0                   Labelling redis pods as master/slave periodical...
https://artifacthub.io/packages/helm/wyrihaximu...      1.0.5           v1.0.1                  Redis Database Assignment Operator                
https://artifacthub.io/packages/helm/appscode/k...      2022.5.11       v0.5.0                  Kubeform Provider Azurerm Redis Custom Resource...
https://artifacthub.io/packages/helm/appscode/k...      2022.5.11       v0.5.0                  Kubeform Provider Google Redis Custom Resource ...
https://artifacthub.io/packages/helm/devops-cha...      0.1.0           1.16.0                  A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/riftbit/re...      0.1.0           v1.0.0                  RedisInsight - The GUI for Redis                  
https://artifacthub.io/packages/helm/pozetron/k...      0.5.3           v6.0.16                 A Helm chart for multimaster KeyDB optionally w...
https://artifacthub.io/packages/helm/enapter/keydb      0.39.0          6.3.1                   A Helm chart for KeyDB multimaster setup          
https://artifacthub.io/packages/helm/riftbit/lo...      2.5.1           v2.1.0                  Loki: like Prometheus, but for logs.              
https://artifacthub.io/packages/helm/ananace-ch...      4.2.0           3.1.0                   An IP address management (IPAM) and data center...
https://artifacthub.io/packages/helm/oauth2-pro...      6.2.1           7.2.1                   A reverse proxy that provides authentication wi...
https://artifacthub.io/packages/helm/sossickd/p...      0.1.0           5.0.0                   A Helm chart for the php guest book.              
https://artifacthub.io/packages/helm/ygqygq2/ra...      1.0.0           v1.0.0-RC6.1            A Helm chart servicemonitor for rabbitmq exporter 
https://artifacthub.io/packages/helm/camptocamp...      3.0.1                                   A Helm chart for the Spotahome Redis Operator     
https://artifacthub.io/packages/helm/ectobit/rs...      0.8.18          3.2-alpine3.16.0        Rspamd Helm chart for Kubernetes                  
https://artifacthub.io/packages/helm/spring-res...      0.1.0           1.16.0                  A SpringBoot Helm chart for Kubernetes            
$ helm search hub redis | wc -l
66
$ 
```

```sh
$ helm search hub redis | grep bitnami
https://artifacthub.io/packages/helm/bitnami-ak...      16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/bitnami/redis      16.12.3         6.2.7                   Redis(R) is an open source, advanced key-value ...
https://artifacthub.io/packages/helm/bitnami/re...      7.6.3           6.2.7                   Redis(R) is an open source, scalable, distribut...
https://artifacthub.io/packages/helm/bitnami-ak...      7.6.3           6.2.7                   Redis(R) is an open source, scalable, distribut...
$ 
```



## Repos

While the chart is listed in Artifact Hub, the Bitnami organization has a public repo of all its charts. In each Hub chart page a repo is listed  for you to add for access the chart.

## Helm Repo

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
$ 
```

```sh
$ helm repo list
NAME                    URL                                    
kubernetes-dashboard    https://kubernetes.github.io/dashboard/
bitnami                 https://charts.bitnami.com/bitnami     
$ 
```

Instead of searching the Hub for charts you can also search the Bitnami repo:

```sh
$ helm search repo bitnami/redis
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/redis           16.12.3         6.2.7           Redis(R) is an open source, advanced key-value ...
bitnami/redis-cluster   7.6.3           6.2.7           Redis(R) is an open source, scalable, distribut...
$ 
```

## Helm Show Chart

```sh
$ helm show chart bitnami/redis
annotations:
  category: Database
apiVersion: v2
appVersion: 6.2.7
dependencies:
- name: common
  repository: https://charts.bitnami.com/bitnami
  tags:
  - bitnami-common
  version: 1.x.x
description: Redis(R) is an open source, advanced key-value store. It is often referred
  to as a data structure server since keys can contain strings, hashes, lists, sets
  and sorted sets.
home: https://github.com/bitnami/charts/tree/master/bitnami/redis
icon: https://bitnami.com/assets/stacks/redis/img/redis-stack-220x234.png
keywords:
- redis
- keyvalue
- database
maintainers:
- name: Bitnami
  url: https://github.com/bitnami/charts
- email: cedric@desaintmartin.fr
  name: desaintmartin
name: redis
sources:
- https://github.com/bitnami/bitnami-docker-redis
version: 16.12.3

$ 
```

## Helm Show Readme

````sh
$ helm show readme bitnami/redis | less
<!--- app-name: Redis&reg; -->

# Bitnami package for Redis(R)

Redis(R) is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.

[Overview of Redis&reg;](http://redis.io)

Disclaimer: Redis is a registered trademark of Redis Ltd. Any rights therein are reserved to Redis Ltd. Any use by Bitnami is for referential purposes only and does not indicate any sponsorship, endorsement, or affiliation between Redis Ltd.
                           
## TL;DR

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm install my-release bitnami/redis
```

## Introduction

This chart bootstraps a [Redis&reg;](https://github.com/bitnami/bitnami-docker-redis) deployment on a [Kubernetes](https://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

Bitnami charts can be used with [Kubeapps](https://kubeapps.dev/) for deployment and management of Helm Charts in clusters.

### Choose between Redis&reg; Helm Chart and Redis&reg; Cluster Helm Chart

You can choose any of the two Redis&reg; Helm charts for deploying a Redis&reg; cluster.

1. [Redis&reg; Helm Chart](https://github.com/bitnami/charts/tree/master/bitnami/redis) will deploy a master-slave cluster, with the [option](https://github.com/bitnami/charts/tree/master/bitnami/redis#redis-sentinel-configuration-parameters) of enabling using Redis&reg; Sentinel.
2. [Redis&reg; Cluster Helm Chart](https://github.com/bitnami/charts/tree/master/bitnami/redis-cluster) will deploy a Redis&reg; Cluster topology with sharding.

The main features of each chart are the following:

{trimmed}

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install my-release \
  --set auth.password=secretpassword \
    bitnami/redis
```

The above command sets the Redis&reg; server password to `secretpassword`.

> NOTE: Once this chart is deployed, it is not possible to change the application's access credentials, such as usernames or passwords, using Helm. To change these application credentials after deployment, delete any persistent volumes (PVs) used by the chart and re-deploy it, or use the application's built-in administrative tools if available.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install my-release -f values.yaml bitnami/redis
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Configuration and installation details

{Trimmed}

In order to upgrade, delete the Redis&reg; StatefulSet before upgrading:

```bash
kubectl delete statefulsets.apps --cascade=false my-release-redis-master
```

And edit the Redis&reg; slave (and metrics if enabled) deployment:

```bash
kubectl patch deployments my-release-redis-slave --type=json -p='[{"op": "remove", "path": "/spec/selector/matchLabels/chart"}]'
kubectl patch deployments my-release-redis-metrics --type=json -p='[{"op": "remove", "path": "/spec/selector/matchLabels/chart"}]'
```

## License

Copyright &copy; 2022 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
````
## Helm Show Values
```sh
$ helm show values bitnami/redis | less
## @section Global parameters
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry, imagePullSecrets and storageClass
##

## @param global.imageRegistry Global Docker image registry
## @param global.imagePullSecrets Global Docker registry secret names as an array
## @param global.storageClass Global StorageClass for Persistent Volume(s)
## @param global.redis.password Global Redis&reg; password (overrides `auth.password`)
##
global:
  imageRegistry: ""
  ## E.g.
  ## imagePullSecrets:
  ##   - myRegistryKeySecretName
  ##
  imagePullSecrets: []
  storageClass: ""
  redis:
    password: ""

## @section Common parameters
##

## @param kubeVersion Override Kubernetes version
##
kubeVersion: ""
## @param nameOverride String to partially override common.names.fullname
##
nameOverride: ""
## @param fullnameOverride String to fully override common.names.fullname
##
fullnameOverride: ""
## @param commonLabels Labels to add to all deployed objects
##
```



## Another Example

```sh
$ 
$ helm search repo fabric8
No results found
$ helm repo add fabric8 https://fabric8.io/helm
"fabric8" has been added to your repositories
$ helm repo list
NAME                    URL                                    
kubernetes-dashboard    https://kubernetes.github.io/dashboard/
bitnami                 https://charts.bitnami.com/bitnami     
fabric8                 https://fabric8.io/helm                
$ 
```

```sh
$ helm search repo fabric8
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                       
fabric8/fabric8-camel                   2.2.168                         Sonatype helps open source projects to set up M...
fabric8/fabric8-console                 2.2.199                         Sonatype helps open source projects to set up M...
fabric8/fabric8-docker-registry         2.2.327                         [Docker Registry](https://github.com/docker/dis...
fabric8/fabric8-dsaas                   1.0.54                          The Fabric8 Online Platform                       
fabric8/fabric8-forge                   2.3.90                          Fabric8 :: Forge                                  
fabric8/fabric8-online                  1.0.20                          The Fabric8 Online Platform                       
fabric8/fabric8-online-team             1.0.54                          The Fabric8 Microservices Online :: Team          
fabric8/fabric8-planner                 1.0.54                          Sonatype helps open source projects to set up M...
fabric8/fabric8-platform                2.4.24                          The Fabric8 Microservices Platform                
fabric8/fabric8-runtime-console         1.0.12                          Sonatype helps open source projects to set up M...
fabric8/fabric8-team                    2.4.24                          The Fabric8 Microservices Platform :: Team        
fabric8/fabric8-ui                      1.0.15                          Sonatype helps open source projects to set up M...
fabric8/apiman                          2.2.168                         Sonatype helps open source projects to set up M...
fabric8/apiman-gateway                  2.2.168                         Apiman Gateway deployed on Jetty                  
fabric8/app-catalog                     2.4.24                          Sonatype helps open source projects to set up M...
fabric8/artifactory                     2.2.327                         [Artifactory](http://www.jfrog.com/open-source/...
fabric8/bayesian-link                   1.0.54                          Sonatype helps open source projects to set up M...
fabric8/brackets                        2.2.327                         [Brackets](http://brackets.io/) editor for work...
fabric8/camel-amq                       2.2.168                         Sonatype helps open source projects to set up M...
fabric8/camel-master                    2.2.168                         Sonatype helps open source projects to set up M...
fabric8/cassandra                       2.2.168                         [Apache Cassandra](http://cassandra.apache.org/...
fabric8/cd-pipeline                     2.4.24                          Creates the Continuous Delivery Pipeline via: g...
fabric8/chaos-monkey                    2.2.327                         Kills random pods for chaos fun!                  
fabric8/chat-irc                        2.4.24                          Sonatype helps open source projects to set up M...
fabric8/chat-letschat                   2.4.24                          Sonatype helps open source projects to set up M...
fabric8/chat-slack                      2.4.24                          Sonatype helps open source projects to set up M...
fabric8/che                             1.0.54                          Sonatype helps open source projects to set up M...
fabric8/configmapcontroller             2.2.327                         Performs Rolling Upgrades to Deployments when C...
fabric8/console                         2.4.24                          The Fabric8 Microservices Console                 
fabric8/content-repository              2.2.327                         A Repository storing static HTML reports and co...
fabric8/elasticsearch                   2.2.327                         [elasticsearch](http://elasticsearch.com/) prov...
fabric8/elasticsearch-v1                2.2.168                         [elasticsearch](http://elasticsearch.com/) prov...
fabric8/example-message-consumer        2.2.168                         Fabric8 Messaging Example Message Consumer        
fabric8/example-message-producer        2.2.168                         Fabric8 Messaging Example Message Producer        
fabric8/exposecontroller                2.2.327                         Exposes services externally based on Kubernetes...
fabric8/fluentd                         2.2.327                         [Fluentd](http://www.fluentd.org/) is an open s...
fabric8/funktion                        2.4.24                          Sonatype helps open source projects to set up M...
fabric8/funktion-operator               2.4.24                          Sonatype helps open source projects to set up M...
fabric8/funktion-platform               2.4.24                          Sonatype helps open source projects to set up M...
fabric8/gerrit                          2.2.327                         [Gerrit](https://code.google.com/p/gerrit/) Web...
fabric8/git-collector                   2.2.327                         Collects events from git repositories from proj...
fabric8/gitlab                          2.2.327                         [Gitlab](http://gitlab.com/) - A self-hosted Gi...
fabric8/gogs                            2.2.327                         [Gogs](http://gogs/) - A self-hosted Git servic...
fabric8/grafana                         2.2.327                         [Grafana](http://grafana.org/) console for view...
fabric8/hello-hystrix                   1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/hello-ribbon                    1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/hello-ribbon-spring-boot        1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/hubot-irc                       2.2.327                         [Hubot](http://hubot.github.com) IRC chat bot     
fabric8/hubot-letschat                  2.2.327                         [Hubot](http://hubot.github.com) chat bot for L...
fabric8/hubot-notifier                  2.2.327                         Watches the OpenShift environment and notifies ...
fabric8/hubot-slack                     2.2.327                         [Hubot](https://hubot.github.com/) adapter to u...
fabric8/hystrix-dashboard               1.0.28                          Dashboard for visualization of Hystrix streams    
fabric8/image-linker                    2.2.327                         Provides embedded chat friendly links to images   
fabric8/ingress-nginx                   2.2.327                         Runs an nginx ingress load balancer               
fabric8/ipaas-console                   0.0.9                           iPaaS Console                                     
fabric8/ipaas-platform                  1.0.32                          The Fabric8 Microservices Platform                
fabric8/jenkins                         2.2.327                         [Jenkins](http://jenkins-ci.org/) extendable op...
fabric8/jenkins-openshift               2.2.327                         [Jenkins](http://jenkins-ci.org/) extendable op...
fabric8/jnatsd                          2.2.168                         Java broker for the NATS protocol                 
fabric8/kafka                           2.2.168                         Sonatype helps open source projects to set up M...
fabric8/kafka-manager                   2.2.168                         [Kafka Manager](https://github.com/yahoo/kafka-...
fabric8/keycloak                        2.2.327                         Keycloack (security) - Integrated SSO and IDM f...
fabric8/kibana                          2.2.327                         Awesome front-end for Elasticsearch               
fabric8/kiwiirc                         2.2.327                         [Kiwi IRC](https://kiwiirc.com/) KiwiIRC makes ...
fabric8/kubeflix                        1.0.28                          Turbine Server and Hystrix Dashboard              
fabric8/kubernetes-reload-example       0.1.6                           Example demostrating how to use the configurati...
fabric8/letschat                        2.2.327                         [Lets Chat](http://sdelements.github.io/lets-ch...
fabric8/loanbroker-bank                 1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/loanbroker-broker               1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/loanbroker-credit-bureau        1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/loanbroker-generator            1.0.28                          Kubernetes integration of Netflix OSS components  
fabric8/logging                         2.4.24                          Sonatype helps open source projects to set up M...
fabric8/manageiq                        2.2.327                         ManageIQ Cloud Management Platform http://manag...
fabric8/management                      2.4.24                          Sonatype helps open source projects to set up M...
fabric8/maven-shell                     2.2.327                         Maven Shell is an app that contains java build ...
fabric8/message-broker                  2.2.168                         Enterprise Message Broker based on Apache Activ...
fabric8/message-gateway                 2.2.168                         An Enterprise Messaging Gateway that can apply ...
fabric8/messaging                       2.2.168                         Sonatype helps open source projects to set up M...
fabric8/metrics                         2.4.24                          Sonatype helps open source projects to set up M...
fabric8/mq-client                       2.2.168                         Sonatype helps open source projects to set up M...
fabric8/nexus                           2.2.327                         [Nexus](http://www.sonatype.org/nexus/) - A mav...
fabric8/nexus3                          2.2.327                         [Nexus](http://www.sonatype.org/nexus/) - A mav...
fabric8/orion                           2.2.327                         [Orion](http://eclipse.org/orion/) is a web bas...
fabric8/prometheus                      2.2.327                         [Prometheus](http://prometheus.io/) - an open-s...
fabric8/prometheus-blackbox-exporter    2.2.327                         [Prometheus Blackbox Exporter](https://github.c...
fabric8/prometheus-node-exporter        2.2.327                         [Prometheus](http://prometheus.io/) [Node Expor...
fabric8/social                          2.4.24                          Sonatype helps open source projects to set up M...
fabric8/sonarqube                       2.2.327                         [SonarQube](http://www.sonarqube.org/) is an op...
fabric8/taiga                           2.2.327                         [Taiga](https://github.com/taigaio) Your Agile,...
fabric8/turbine-server                  1.0.28                          Turbine server with the Kubernetes discovery mo...
fabric8/upgrade-job                     2.2.327                         Checks for a new release and update running app...
fabric8/validator-job                   2.2.327                         Validates a fabric8 install using system tests ...
fabric8/zipkin                          0.1.8                           Zipkin Package                                    
fabric8/zipkin-collector                0.1.3                           Zipkin collector service                          
fabric8/zipkin-hello-world              0.1.8                           Zipkin Hello World example                        
fabric8/zipkin-mysql                    0.1.8                           Zipkin MySQL storage                              
fabric8/zipkin-query                    0.1.3                           Zipkin query service                              
fabric8/zipkin-starter                  0.1.8                           Starter module for installing Zipkin into Kuber...
fabric8/zipkin-starter-minimal          0.1.8                           Starter module for installing Zipkin (minimal) ...
fabric8/zipkin-starter-mysql            0.1.8                           Starter module for installing Zipkin into Kuber...
fabric8/zookeeper                       2.2.168                         Sonatype helps open source projects to set up M...
fabric8/zookeeper-ensemble              2.2.168                         Sonatype helps open source projects to set up M...
$ 
```

So far we have seen how to find and list public charts. 



```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami && helm repo list
"bitnami" already exists with the same configuration, skipping
NAME                    URL                                    
kubernetes-dashboard    https://kubernetes.github.io/dashboard/
bitnami                 https://charts.bitnami.com/bitnami     
fabric8                 https://fabric8.io/helm                
$ VERSION=16.8.10
$ helm install my-redis bitnami/redis \
>   --version $VERSION \
>   --namespace redis \
>   --create-namespace \
>   --values redis-values.yaml
NAME: my-redis
LAST DEPLOYED: Fri Jun 24 14:49:50 2022
NAMESPACE: redis
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.8.10
APP VERSION: 6.2.7

** Please be patient while the chart is being deployed **

Redis&trade; can be accessed on the following DNS names from within your cluster:

    my-redis-master.redis.svc.cluster.local for read/write operations (port 6379)
    my-redis-replicas.redis.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace redis my-redis -o jsonpath="{.data.redis-password}" | base64 --decode)

To connect to your Redis&trade; server:

1. Run a Redis&trade; pod that you can use as a client:

   kubectl run --namespace redis redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.7-debian-10-r0 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace redis -- bash

2. Connect using the Redis&trade; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h my-redis-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h my-redis-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace redis svc/my-redis-master : &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p
$ 
```

This will name a new install called *my-redis* and install a specific chart name and version into the *redis* namespace. The redis-values file override the chart's default values to ensure there are just 2 replicas and some file permission configuration is performed at startup. With the install command Helm launches the  required Deployments, ReplicaSets, Pods, Services, ConfigMaps, or any  other Kubernetes resource the chart defines.



```sh
$ cat redis-values.yaml 
replica:
   replicaCount: 2

volumePermissions:
  enabled: true

securityContext:
  enabled: true
  fsGroup: 1001
  runAsUser: 1001
$ 
```

```sh
$ helm list --all-namespaces
NAME                    NAMESPACE               REVISION        UPDATED                                     STATUS          CHART                           APP VERSION
kubernetes-dashboard    kubernetes-dashboard    1               2022-06-24 13:58:01.784829321 +0000 UTC     deployed        kubernetes-dashboard-5.4.1      2.5.1      
my-redis                redis                   1               2022-06-24 14:49:50.888943355 +0000 UTC     deployed        redis-16.8.10                   6.2.7      
$ helm ls -n redis
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS     CHART            APP VERSION
my-redis        redis           1               2022-06-24 14:49:50.888943355 +0000 UTC deployed   redis-16.8.10    6.2.7      
$ 
```

```sh
$ kubectl get secrets --all-namespaces | grep sh.helm
kubernetes-dashboard   sh.helm.release.v1.kubernetes-dashboard.v1        helm.sh/release.v1                    1      53m
redis                  sh.helm.release.v1.my-redis.v1                    helm.sh/release.v1                    1      97s
$ helm list -A
NAME                    NAMESPACE               REVISION        UPDATED                                     STATUS          CHART                           APP VERSION
kubernetes-dashboard    kubernetes-dashboard    1               2022-06-24 13:58:01.784829321 +0000 UTC     deployed        kubernetes-dashboard-5.4.1      2.5.1      
my-redis                redis                   1               2022-06-24 14:49:50.888943355 +0000 UTC     deployed        redis-16.8.10                   6.2.7      
$ kubectl get secrets --all-namespaces --selector owner=helm
NAMESPACE              NAME                                         TYPE                 DATA   AGE
kubernetes-dashboard   sh.helm.release.v1.kubernetes-dashboard.v1   helm.sh/release.v1   1      53m
redis                  sh.helm.release.v1.my-redis.v1               helm.sh/release.v1   1      107s
$ kubectl get secrets --all-namespaces --selector owner=helm
NAMESPACE              NAME                                         TYPE                 DATA   AGE
kubernetes-dashboard   sh.helm.release.v1.kubernetes-dashboard.v1   helm.sh/release.v1   1      53m
redis                  sh.helm.release.v1.my-redis.v1               helm.sh/release.v1   1      109s
$ kubectl --namespace redis describe secret sh.helm.release.v1.my-redis.v1
Name:         sh.helm.release.v1.my-redis.v1
Namespace:    redis
Labels:       modifiedAt=1656082191
              name=my-redis
              owner=helm
              status=deployed
              version=1
Annotations:  <none>

Type:  helm.sh/release.v1

Data
====
release:  132232 bytes
$ 
```

```sh
$ watch kubectl get statefulsets,pods,services -n redis
Every 2.0s: kubectl get statefulsets,pods,services -n redis   controlplane: Fri Jun 24 14:52:52 2022

NAME                                 READY   AGE
statefulset.apps/my-redis-master     0/1     3m1s
statefulset.apps/my-redis-replicas   0/2     3m1s

NAME                      READY   STATUS    RESTARTS   AGE
pod/my-redis-master-0     0/1     Pending   0          3m1s
pod/my-redis-replicas-0   0/1     Pending   0          3m1s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-redis-headless   ClusterIP   None             <none>        6379/TCP   3m1s
service/my-redis-master     ClusterIP   10.96.42.13      <none>        6379/TCP   3m1s
service/my-redis-replicas   ClusterIP   10.109.110.111   <none>        6379/TCP   3m1s

```



```sh
$ cat pv.yaml 
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume1
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data1"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume2
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data2"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume3
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data3"$ 
```



```sh
$ kubectl apply -f pv.yaml
persistentvolume/pv-volume1 created
persistentvolume/pv-volume2 created
persistentvolume/pv-volume3 created
$ 
```



```sh
$ mkdir /mnt/data1 /mnt/data2 /mnt/data3 --mode=777
$ watch kubectl get statefulsets,pods,services -n redis

Every 2.0s: kubectl get statefulsets,pods,services -n redis   controlplane: Fri Jun 24 14:53:45 2022

NAME                                 READY   AGE
statefulset.apps/my-redis-master     0/1     3m54s
statefulset.apps/my-redis-replicas   0/2     3m54s

NAME                      READY   STATUS    RESTARTS   AGE
pod/my-redis-master-0     0/1     Running   0          3m54s
pod/my-redis-replicas-0   0/1     Running   0          3m54s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-redis-headless   ClusterIP   None             <none>        6379/TCP   3m54s
service/my-redis-master     ClusterIP   10.96.42.13      <none>        6379/TCP   3m54s
service/my-redis-replicas   ClusterIP   10.109.110.111   <none>        6379/TCP   3m54s
```

```sh
$ kubectl get statefulsets,pods,services -n redis
NAME                                 READY   AGE
statefulset.apps/my-redis-master     1/1     4m32s
statefulset.apps/my-redis-replicas   1/2     4m32s

NAME                      READY   STATUS    RESTARTS   AGE
pod/my-redis-master-0     1/1     Running   0          4m32s
pod/my-redis-replicas-0   1/1     Running   0          4m32s
pod/my-redis-replicas-1   0/1     Running   0          8s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/my-redis-headless   ClusterIP   None             <none>        6379/TCP   4m32s
service/my-redis-master     ClusterIP   10.96.42.13      <none>        6379/TCP   4m32s
service/my-redis-replicas   ClusterIP   10.109.110.111   <none>        6379/TCP   4m32s
$ 
```

## Connect to Your Redis Server

To get your password query the Redis Secret:

Expose the Redis service:

```sh
$ export REDIS_PASSWORD=$(kubectl get secret --namespace redis my-redis -o jsonpath="{.data.redis-password}" | base64 --decode)
$ kubectl port-forward --namespace redis service/my-redis-master 6379:6379 > /dev/null &
[1] 22287
$ 
```

```sh
$ redis-cli -h 127.0.0.1 -p 6379 -a $REDIS_PASSWORD ping | grep PONG
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
PONG
$ 
```

If you see `PONG` as the response you have connected successfully to the Redis application installed by the Helm chart. Nice work!

## Helm Delete
```sh
$ helm delete my-redis -n redis
release "my-redis" uninstalled
$ kubectl delete namespace redis
namespace "redis" deleted

```

```sh
$ echo "The number of charts on Artifact Hub is: $(helm search hub | wc -l)."
The number of charts on Artifact Hub is: 8445.
$ 
```



## Create Helm Chart

```sh
$ helm create app-chart
Creating app-chart
$ tree app-chart
app-chart
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files
$ cat app-chart/templates/deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "app-chart.fullname" . }}
  labels:
    {{- include "app-chart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "app-chart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "app-chart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "app-chart.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
$ 
```

```sh
$ cat app-chart/templates/deployment.yaml | grep 'kind:' -n -B1 -A5
1-apiVersion: apps/v1
2:kind: Deployment
3-metadata:
4-  name: {{ include "app-chart.fullname" . }}
5-  labels:
6-    {{- include "app-chart.labels" . | nindent 4 }}
7-spec:
$ cat app-chart/templates/deployment.yaml | grep 'image:' -n -C3
31-        - name: {{ .Chart.Name }}
32-          securityContext:
33-            {{- toYaml .Values.securityContext | nindent 12 }}
34:          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
35-          imagePullPolicy: {{ .Values.image.pullPolicy }}
36-          ports:
37-            - name: http
$ cat app-chart/values.yaml | grep 'repository' -n -C3
5-replicaCount: 1
6-
7-image:
8:  repository: nginx
9-  pullPolicy: IfNotPresent
10-  # Overrides the image tag whose default is the chart appVersion.
11-  tag: ""
$ 
```

```sh
$ helm install my-app ./app-chart --dry-run --debug | grep 'image: "' -n -C3
install.go:178: [debug] Original chart version: ""
install.go:195: [debug] CHART PATH: /root/app-chart

133-        - name: app-chart
134-          securityContext:
135-            {}
136:          image: "nginx:1.16.0"
137-          imagePullPolicy: IfNotPresent
138-          ports:
139-            - name: http
$ helm install my-app ./app-chart --dry-run --debug --set image.pullPolicy=Always | grep 'image: "' -n -C3
install.go:178: [debug] Original chart version: ""
install.go:195: [debug] CHART PATH: /root/app-chart

134-        - name: app-chart
135-          securityContext:
136-            {}
137:          image: "nginx:1.16.0"
138-          imagePullPolicy: Always
139-          ports:
140-            - name: http
$ helm install my-app ./app-chart --set image.pullPolicy=Always
NAME: my-app
LAST DEPLOYED: Fri Jun 24 14:58:38 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=app-chart,app.kubernetes.io/instance=my-app" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
$ 
```

```sh
$ kubectl get deployments,service
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-app-app-chart   1/1     1            1           30s

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP   61m
service/my-app-app-chart   ClusterIP   10.106.231.228   <none>        80/TCP    30s
$ 
```

```sh
$ helm search hub chartmuseum
URL                                                     CHART VERSION   APP VERSION     DESCRIPTION                                       
https://artifacthub.io/packages/helm/chartmuseu...      3.8.0           0.14.0          Host your own Helm Chart Repository               
https://artifacthub.io/packages/helm/mike7515/c...      1.0.0           1.0.0           Host your own Helm Chart Repository               
https://artifacthub.io/packages/helm/phntom/cha...      4.0.12          0.14.0          chartmuseum fork with local storage + s3 sync     
https://artifacthub.io/packages/helm/choerodon/...      3.6.3           0.14.0          Host your own Helm Chart Repository               
https://artifacthub.io/packages/helm/cloudnativ...      2.3.1           0.8.2           Host your own Helm Chart Repository               
https://artifacthub.io/packages/helm/jenkins-x/...      1.1.7           0.7.1           Helm Chart Repository with support for Amazon S...
https://artifacthub.io/packages/helm/stakater/c...      1.8.0           0.8.0           Helm Chart Repository with support for Amazon S...
https://artifacthub.io/packages/helm/cloudposse...      0.2.0           0.2.7           Helm Chart Repository with support for Amazon S...
https://artifacthub.io/packages/helm/stakater/c...      1.0.12                          Chart that deploys PV and PVC for use with Char...
$ 
```

```sh
$ helm upgrade my-app ./app-chart --install --reuse-values --set service.type=NodePort
Release "my-app" has been upgraded. Happy Helming!
NAME: my-app
LAST DEPLOYED: Fri Jun 24 14:59:55 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-app-app-chart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
$ 
```

## Further commands

There are a few more helpful commands that you can discover:

```
helm --help
```

The `helm lint` command will check your charts for errors. Something that commonly happens when editing YAML files and  experimenting with the Go templating syntax in the template files.

The `helm test` command can be used to bake testing into your chart usage in CI/CD pipelines.

The [helm plugin](https://helm.sh/docs/topics/plugins/) opens Helm for many extension possibilities. Here is a [curated list](https://helm.sh/docs/community/related/) of helpful extensions for Helm.



Happy Helming!
