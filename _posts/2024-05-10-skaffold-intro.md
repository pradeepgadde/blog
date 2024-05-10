---
layout: single
title:  "Getting Started with Skaffold"
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
 
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  teaser: /assets/images/kubernetes.png
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

# Skaffold

Skaffold is a command line tool that facilitates continuous development for container based & Kubernetes applications. Skaffold handles the workflow for building, pushing, and deploying your application, and provides building blocks for creating CI/CD pipelines. This enables you to focus on iterating on your application locally while Skaffold continuously deploys to your local or remote Kubernetes cluster, local Docker environment or Cloud Run project.

Skaffold handles the workflow for building, pushing and deploying your application, allowing you  to focus on what matters most:  **writing code**.  
An open source project from Google.                        
Skaffold is client-side only. With no on-cluster component, there is no overhead or maintenance burden to your cluster.
Skaffold is the easiest way to share your project with the world: 'git clone', then 'skaffold run'.

Additionally, you can use profiles, local user config, environment variables, and flags to easily incorporate differences across environments.
Skaffold has many essential features for container & Kubernetes development, including policy-based image tagging, resource port-forwarding and logging, file syncing, and much more.

Use **skaffold init** to bootstrap your Skaffold config.

Use **skaffold dev** to automatically build and deploy your application when your code changes.

Use **skaffold build** and **skaffold test** to tag, push, and test your container images.

Use **skaffold render** and **skaffold apply** to generate and deploy Kubernetes manifests as part of a GitOps workflow



## Install Skaffold

```sh
(base) pradeep:~$curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-darwin-amd64 && \
sudo install skaffold /usr/local/bin/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 83.6M  100 83.6M    0     0  5627k      0  0:00:15  0:00:15 --:--:-- 5822k
Password:
(base) pradeep:~$
```

## Install Minikube

```sh
(base) pradeep:~$curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 93.4M  100 93.4M    0     0  7119k      0  0:00:13  0:00:13 --:--:-- 8250k
(base) pradeep:~$
```

## Install Kubectl

```sh
(base) pradeep:~$curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    454      0 --:--:-- --:--:-- --:--:--   453
100 50.1M  100 50.1M    0     0  1503k      0  0:00:34  0:00:34 --:--:-- 1703k
(base) pradeep:~$
```

This tutorial uses minikube because Skaffold knows how to build the app using the Docker daemon hosted inside minikube. This means we don’t need a registry to host the app’s container images.

### Clone the sample app

Let’s get a sample application set up to use Skaffold.

1. Clone the Skaffold repository:

```sh
(base) pradeep:~$cd Downloads 
(base) pradeep:~$git clone https://github.com/GoogleContainerTools/skaffold
Cloning into 'skaffold'...
remote: Enumerating objects: 145142, done.
remote: Counting objects: 100% (6392/6392), done.
remote: Compressing objects: 100% (3712/3712), done.
remote: Total 145142 (delta 2751), reused 5818 (delta 2299), pack-reused 138750
Receiving objects: 100% (145142/145142), 140.46 MiB | 7.93 MiB/s, done.
Resolving deltas: 100% (90066/90066), done.
Updating files: 100% (13229/13229), done.
(base) pradeep:~$cd skaffold/examples/buildpacks-node-tutorial
(base) pradeep:~$

```

## Initialize Skaffold

Your working directory is the application directory, `skaffold/examples/buildpacks-node-tutorial`. This will be our root Skaffold directory.

This sample application is written in Node, but Skaffold is language-agnostic and works with any containerized application.

### Bootstrap Skaffold configuration

Run the following command to generate a `skaffold.yaml` config file:

```sh
(base) pradeep:~$skaffold init
WARN[0000] Skipping Jib: no JVM: [java -version] failed: exit status 1  subtask=-1 task=DevLoop
? Choose the builder to build image skaffold-buildpacks-node Buildpacks (package.json)
? Which builders would you like to create kubernetes resources for? 
apiVersion: skaffold/v4beta10
kind: Config
metadata:
  name: buildpacks-node-tutorial
build:
  artifacts:
    - image: skaffold-buildpacks-node
      buildpacks:
        builder: gcr.io/buildpacks/builder:v1
manifests:
  rawYaml:
    - k8s/web.yaml

? Do you want to write this configuration to skaffold.yaml? Yes
Configuration skaffold.yaml was written
You can now run [skaffold build] to build the artifacts
or [skaffold run] to build and deploy
or [skaffold dev] to enter development mode, with auto-redeploy
(base) pradeep:~$

```
Open your new skaffold.yaml, generated at skaffold/examples/buildpacks-node-tutorial/skaffold.yaml. All of your Skaffold configuration lives in this file.
```sh
(base) pradeep:~$ls
README.md	k8s		package.json	project.toml	public		skaffold.yaml	src
(base) pradeep:~$cat skaffold.yaml 
apiVersion: skaffold/v4beta10
kind: Config
metadata:
  name: buildpacks-node-tutorial
build:
  artifacts:
    - image: skaffold-buildpacks-node
      buildpacks:
        builder: gcr.io/buildpacks/builder:v1
manifests:
  rawYaml:
    - k8s/web.yaml
(base) pradeep:~$

```

## Use Skaffold for continuous development

Skaffold speeds up your development loop by automatically building and deploying the application whenever your code changes.

### Start minikube

To see this in action, let’s start up minikube so Skaffold has a cluster to run your application.

```sh
(base) pradeep:~$minikube start --profile custom
skaffold config set --global local-cluster true
eval $(minikube -p custom docker-env)
😄  [custom] minikube v1.33.0 on Darwin 14.3
✨  Automatically selected the hyperkit driver. Other choices: virtualbox, ssh
💿  Downloading VM boot image ...
    > minikube-v1.33.0-amd64.iso....:  65 B / 65 B [---------] 100.00% ? p/s 0s
    > minikube-v1.33.0-amd64.iso:  314.16 MiB / 314.16 MiB  100.00% 9.20 MiB p/
👍  Starting "custom" primary control-plane node in "custom" cluster
💾  Downloading Kubernetes v1.30.0 preload ...
    > preloaded-images-k8s-v18-v1...:  342.90 MiB / 342.90 MiB  100.00% 4.97 Mi
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.30.0 on Docker 26.0.1 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner

❗  /usr/local/bin/kubectl is version 1.22.5, which may have incompatibilities with Kubernetes 1.30.0.
    ▪ Want kubectl v1.30.0? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "custom" cluster and "default" namespace by default
set global value local-cluster to true
(base) pradeep:~$
```

### Use `skaffold dev`

Run the following command to begin using Skaffold for continuous development:

```sh
(base) pradeep:~$skaffold dev
Generating tags...
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:v2.11.1-22-g0ffd37362-dirty
Checking cache...
 - skaffold-buildpacks-node: Not found. Building
Starting build...
Found [custom] context, using local docker daemon.
Building [skaffold-buildpacks-node]...
Target platforms: [linux/amd64]
v1: Pulling from buildpacks/builder
53e5e158da5a: Pulling fs layer
7577c9c60d3f: Pulling fs layer
3c2cba919283: Pulling fs layer
ddf16da93ef1: Pulling fs layer
6ffe2f6b5faf: Pulling fs layer
82ee75ce5f62: Pulling fs layer
c1ed5ab79d02: Pulling fs layer
4fa2f2eee7ab: Pulling fs layer
357fefdf9bc9: Pulling fs layer
63045f5a9150: Pulling fs layer
5c845929ff47: Pulling fs layer
5d89def61866: Pulling fs layer
52d83dddeb00: Pulling fs layer
1f6aa24b3537: Pulling fs layer
d9835bafd9a5: Pulling fs layer
0f100bd1c899: Pulling fs layer
cfb7c428ddd9: Pulling fs layer
488db72b4c32: Pulling fs layer
cbb1319168f1: Pulling fs layer
39a33e842774: Pulling fs layer
e413611130ac: Pulling fs layer
7d1e10233d91: Pulling fs layer
eeaa9a5b1a56: Pulling fs layer
657c99d14632: Pulling fs layer
c6859daffdc7: Pulling fs layer
5629d6e331de: Pulling fs layer
bbc780188d53: Pulling fs layer
ff0a644a8425: Pulling fs layer
21cb215ffb96: Pulling fs layer
ae241ea219c0: Pulling fs layer
c0ccc3eec588: Pulling fs layer
26a1dba98788: Pulling fs layer
8077ac75dada: Pulling fs layer
65b9baa7ff62: Pulling fs layer
b249d11b4d35: Pulling fs layer
39b066e68433: Pulling fs layer
d415f1cc81e2: Pulling fs layer
1568be7279f9: Pulling fs layer
0039a3f345d3: Pulling fs layer
b0605e0af834: Pulling fs layer
cdb6e09085d7: Pulling fs layer
eca845cf7218: Pulling fs layer
b8f0af2431d6: Pulling fs layer
66f0bc1a3d06: Pulling fs layer
3231ef0e6a1c: Pulling fs layer
a439551e53ff: Pulling fs layer
374505b19cfa: Pulling fs layer
81c9e2fc6c30: Pulling fs layer
f0ece9af7e6d: Pulling fs layer
c5de287817bb: Pulling fs layer
8f9d9735d824: Pulling fs layer
0525625392c9: Pulling fs layer
57dbba1756aa: Pulling fs layer
5d3c60fb9de7: Pulling fs layer
1e995e7fb013: Pulling fs layer
996866880cc3: Pulling fs layer
82171b7749a1: Pulling fs layer
5c166ffa20fe: Pulling fs layer
9e1199b2ba61: Pulling fs layer
53f7877c3d60: Pulling fs layer
4f4fb700ef54: Pulling fs layer
3ae844d48c6f: Pulling fs layer
f1441c52a54f: Pulling fs layer
ddf16da93ef1: Waiting
6ffe2f6b5faf: Waiting
82ee75ce5f62: Waiting
c1ed5ab79d02: Waiting
4fa2f2eee7ab: Waiting
357fefdf9bc9: Waiting
63045f5a9150: Waiting
5c845929ff47: Waiting
5d89def61866: Waiting
52d83dddeb00: Waiting
1f6aa24b3537: Waiting
d9835bafd9a5: Waiting
0f100bd1c899: Waiting
cfb7c428ddd9: Waiting
488db72b4c32: Waiting
cbb1319168f1: Waiting
39a33e842774: Waiting
e413611130ac: Waiting
7d1e10233d91: Waiting
eeaa9a5b1a56: Waiting
657c99d14632: Waiting
c6859daffdc7: Waiting
66f0bc1a3d06: Waiting
3231ef0e6a1c: Waiting
a439551e53ff: Waiting
374505b19cfa: Waiting
81c9e2fc6c30: Waiting
f0ece9af7e6d: Waiting
c5de287817bb: Waiting
8f9d9735d824: Waiting
0525625392c9: Waiting
57dbba1756aa: Waiting
5d3c60fb9de7: Waiting
1e995e7fb013: Waiting
996866880cc3: Waiting
82171b7749a1: Waiting
5c166ffa20fe: Waiting
9e1199b2ba61: Waiting
53f7877c3d60: Waiting
4f4fb700ef54: Waiting
3ae844d48c6f: Waiting
f1441c52a54f: Waiting
5629d6e331de: Waiting
bbc780188d53: Waiting
39b066e68433: Waiting
d415f1cc81e2: Waiting
1568be7279f9: Waiting
0039a3f345d3: Waiting
b0605e0af834: Waiting
cdb6e09085d7: Waiting
eca845cf7218: Waiting
b8f0af2431d6: Waiting
21cb215ffb96: Waiting
ae241ea219c0: Waiting
c0ccc3eec588: Waiting
26a1dba98788: Waiting
8077ac75dada: Waiting
65b9baa7ff62: Waiting
b249d11b4d35: Waiting
ff0a644a8425: Waiting
3c2cba919283: Verifying Checksum
3c2cba919283: Download complete
7577c9c60d3f: Verifying Checksum
7577c9c60d3f: Download complete
6ffe2f6b5faf: Verifying Checksum
6ffe2f6b5faf: Download complete
82ee75ce5f62: Verifying Checksum
82ee75ce5f62: Download complete
c1ed5ab79d02: Verifying Checksum
c1ed5ab79d02: Download complete
53e5e158da5a: Verifying Checksum
53e5e158da5a: Download complete
357fefdf9bc9: Verifying Checksum
357fefdf9bc9: Download complete
53e5e158da5a: Pull complete
7577c9c60d3f: Pull complete
3c2cba919283: Pull complete
63045f5a9150: Verifying Checksum
63045f5a9150: Download complete
ddf16da93ef1: Verifying Checksum
ddf16da93ef1: Download complete
5c845929ff47: Verifying Checksum
5c845929ff47: Download complete
52d83dddeb00: Verifying Checksum
52d83dddeb00: Download complete
5d89def61866: Verifying Checksum
5d89def61866: Download complete
ddf16da93ef1: Pull complete
6ffe2f6b5faf: Pull complete
82ee75ce5f62: Pull complete
c1ed5ab79d02: Pull complete
1f6aa24b3537: Verifying Checksum
1f6aa24b3537: Download complete
d9835bafd9a5: Verifying Checksum
d9835bafd9a5: Download complete
0f100bd1c899: Verifying Checksum
0f100bd1c899: Download complete
488db72b4c32: Verifying Checksum
488db72b4c32: Download complete
cfb7c428ddd9: Verifying Checksum
cfb7c428ddd9: Download complete
39a33e842774: Verifying Checksum
39a33e842774: Download complete
cbb1319168f1: Verifying Checksum
cbb1319168f1: Download complete
e413611130ac: Verifying Checksum
e413611130ac: Download complete
7d1e10233d91: Verifying Checksum
7d1e10233d91: Download complete
eeaa9a5b1a56: Verifying Checksum
eeaa9a5b1a56: Download complete
657c99d14632: Download complete
c6859daffdc7: Verifying Checksum
c6859daffdc7: Download complete
5629d6e331de: Verifying Checksum
5629d6e331de: Download complete
bbc780188d53: Verifying Checksum
bbc780188d53: Download complete
ff0a644a8425: Verifying Checksum
ff0a644a8425: Download complete
ae241ea219c0: Verifying Checksum
ae241ea219c0: Download complete
4fa2f2eee7ab: Verifying Checksum
4fa2f2eee7ab: Download complete
c0ccc3eec588: Verifying Checksum
c0ccc3eec588: Download complete
8077ac75dada: Verifying Checksum
8077ac75dada: Download complete
26a1dba98788: Verifying Checksum
26a1dba98788: Download complete
21cb215ffb96: Verifying Checksum
21cb215ffb96: Download complete
65b9baa7ff62: Verifying Checksum
65b9baa7ff62: Download complete
b249d11b4d35: Verifying Checksum
b249d11b4d35: Download complete
39b066e68433: Verifying Checksum
39b066e68433: Download complete
1568be7279f9: Verifying Checksum
1568be7279f9: Download complete
d415f1cc81e2: Verifying Checksum
d415f1cc81e2: Download complete
0039a3f345d3: Verifying Checksum
0039a3f345d3: Download complete
cdb6e09085d7: Verifying Checksum
cdb6e09085d7: Download complete
b0605e0af834: Verifying Checksum
b0605e0af834: Download complete
eca845cf7218: Verifying Checksum
eca845cf7218: Download complete
b8f0af2431d6: Verifying Checksum
b8f0af2431d6: Download complete
66f0bc1a3d06: Verifying Checksum
66f0bc1a3d06: Download complete
3231ef0e6a1c: Verifying Checksum
3231ef0e6a1c: Download complete
a439551e53ff: Verifying Checksum
a439551e53ff: Download complete
374505b19cfa: Verifying Checksum
374505b19cfa: Download complete
81c9e2fc6c30: Verifying Checksum
81c9e2fc6c30: Download complete
f0ece9af7e6d: Verifying Checksum
f0ece9af7e6d: Download complete
4fa2f2eee7ab: Pull complete
357fefdf9bc9: Pull complete
0525625392c9: Verifying Checksum
0525625392c9: Download complete
c5de287817bb: Verifying Checksum
c5de287817bb: Download complete
63045f5a9150: Pull complete
8f9d9735d824: Verifying Checksum
8f9d9735d824: Download complete
5c845929ff47: Pull complete
5d89def61866: Pull complete
52d83dddeb00: Pull complete
1f6aa24b3537: Pull complete
57dbba1756aa: Verifying Checksum
57dbba1756aa: Download complete
5d3c60fb9de7: Verifying Checksum
5d3c60fb9de7: Download complete
d9835bafd9a5: Pull complete
0f100bd1c899: Pull complete
cfb7c428ddd9: Pull complete
488db72b4c32: Pull complete
1e995e7fb013: Verifying Checksum
1e995e7fb013: Download complete
cbb1319168f1: Pull complete
39a33e842774: Pull complete
e413611130ac: Pull complete
996866880cc3: Verifying Checksum
996866880cc3: Download complete
7d1e10233d91: Pull complete
eeaa9a5b1a56: Pull complete
82171b7749a1: Verifying Checksum
82171b7749a1: Download complete
9e1199b2ba61: Verifying Checksum
9e1199b2ba61: Download complete
657c99d14632: Pull complete
c6859daffdc7: Pull complete
5c166ffa20fe: Verifying Checksum
5c166ffa20fe: Download complete
5629d6e331de: Pull complete
53f7877c3d60: Verifying Checksum
53f7877c3d60: Download complete
bbc780188d53: Pull complete
4f4fb700ef54: Verifying Checksum
4f4fb700ef54: Download complete
ff0a644a8425: Pull complete
21cb215ffb96: Pull complete
3ae844d48c6f: Verifying Checksum
3ae844d48c6f: Download complete
ae241ea219c0: Pull complete
c0ccc3eec588: Pull complete
f1441c52a54f: Verifying Checksum
f1441c52a54f: Download complete
26a1dba98788: Pull complete
8077ac75dada: Pull complete
65b9baa7ff62: Pull complete
b249d11b4d35: Pull complete
39b066e68433: Pull complete
d415f1cc81e2: Pull complete
1568be7279f9: Pull complete
0039a3f345d3: Pull complete
b0605e0af834: Pull complete
cdb6e09085d7: Pull complete
eca845cf7218: Pull complete
b8f0af2431d6: Pull complete
66f0bc1a3d06: Pull complete
3231ef0e6a1c: Pull complete
a439551e53ff: Pull complete
374505b19cfa: Pull complete
81c9e2fc6c30: Pull complete
f0ece9af7e6d: Pull complete
c5de287817bb: Pull complete
8f9d9735d824: Pull complete
0525625392c9: Pull complete
57dbba1756aa: Pull complete
5d3c60fb9de7: Pull complete
1e995e7fb013: Pull complete
996866880cc3: Pull complete
82171b7749a1: Pull complete
5c166ffa20fe: Pull complete
9e1199b2ba61: Pull complete
53f7877c3d60: Pull complete
4f4fb700ef54: Pull complete
3ae844d48c6f: Pull complete
f1441c52a54f: Pull complete
Digest: sha256:76bedb4c2b82d4f086a398f39bafb7dec6a2e9c0484bb951efa5832aaa9db0a4
Status: Downloaded newer image for gcr.io/buildpacks/builder:v1
v1: Pulling from buildpacks/gcp/run
53e5e158da5a: Already exists
7577c9c60d3f: Already exists
3c2cba919283: Already exists
ddf16da93ef1: Already exists
6ffe2f6b5faf: Already exists
0500131c09e4: Pulling fs layer
92543b4fb5b7: Pulling fs layer
866d8ffe09a1: Pulling fs layer
0551acb400d6: Pulling fs layer
205928769f4f: Pulling fs layer
34696c788a43: Pulling fs layer
2bc67dd145ee: Pulling fs layer
0551acb400d6: Waiting
205928769f4f: Waiting
34696c788a43: Waiting
2bc67dd145ee: Waiting
0500131c09e4: Verifying Checksum
0500131c09e4: Download complete
866d8ffe09a1: Verifying Checksum
92543b4fb5b7: Verifying Checksum
92543b4fb5b7: Download complete
866d8ffe09a1: Download complete
0500131c09e4: Pull complete
92543b4fb5b7: Pull complete
866d8ffe09a1: Pull complete
0551acb400d6: Verifying Checksum
0551acb400d6: Download complete
205928769f4f: Verifying Checksum
205928769f4f: Download complete
34696c788a43: Verifying Checksum
34696c788a43: Download complete
0551acb400d6: Pull complete
205928769f4f: Pull complete
34696c788a43: Pull complete
2bc67dd145ee: Download complete
2bc67dd145ee: Pull complete
Digest: sha256:af071306ff95d9e79dd7e4d9513add7cb7a6e162752cc96ee3344188f1bf264e
Status: Downloaded newer image for gcr.io/buildpacks/gcp/run:v1
0.17.4: Pulling from buildpacksio/lifecycle
6b16ad2aede1: Pulling fs layer
fe5ca62666f0: Pulling fs layer
be1681d2fb7c: Pulling fs layer
fcb6f6d2c998: Pulling fs layer
e8c73c638ae9: Pulling fs layer
1e3d9b7d1452: Pulling fs layer
4aa0ea1413d3: Pulling fs layer
7c881f9ab25e: Pulling fs layer
5627a970d25e: Pulling fs layer
1a5d1730211d: Pulling fs layer
1e3d9b7d1452: Waiting
4aa0ea1413d3: Waiting
7c881f9ab25e: Waiting
5627a970d25e: Waiting
1a5d1730211d: Waiting
fcb6f6d2c998: Waiting
e8c73c638ae9: Waiting
fe5ca62666f0: Verifying Checksum
fe5ca62666f0: Download complete
6b16ad2aede1: Download complete
6b16ad2aede1: Pull complete
be1681d2fb7c: Verifying Checksum
be1681d2fb7c: Download complete
fe5ca62666f0: Pull complete
fcb6f6d2c998: Verifying Checksum
fcb6f6d2c998: Download complete
e8c73c638ae9: Verifying Checksum
e8c73c638ae9: Download complete
1e3d9b7d1452: Verifying Checksum
1e3d9b7d1452: Download complete
be1681d2fb7c: Pull complete
fcb6f6d2c998: Pull complete
e8c73c638ae9: Pull complete
1e3d9b7d1452: Pull complete
4aa0ea1413d3: Verifying Checksum
4aa0ea1413d3: Download complete
4aa0ea1413d3: Pull complete
7c881f9ab25e: Verifying Checksum
7c881f9ab25e: Download complete
7c881f9ab25e: Pull complete
5627a970d25e: Verifying Checksum
5627a970d25e: Download complete
5627a970d25e: Pull complete
1a5d1730211d: Verifying Checksum
1a5d1730211d: Download complete
1a5d1730211d: Pull complete
Digest: sha256:83f2d6ff5ef9e08e90565f0a00753490c53a8eab76a27483b68e491e776fdbd8
Status: Downloaded newer image for buildpacksio/lifecycle:0.17.4
===> ANALYZING
[analyzer] Timer: Analyzer started at 2024-05-10T05:56:59Z
[analyzer] Image with name "skaffold-buildpacks-node:latest" not found
[analyzer] Timer: Analyzer ran for 130.915µs and ended at 2024-05-10T05:56:59Z
===> DETECTING
[detector] Timer: Detector started at 2024-05-10T05:57:00Z
[detector] 3 of 5 buildpacks participating
[detector] google.nodejs.runtime    1.0.0
[detector] google.nodejs.npm        1.1.0
[detector] google.utils.label-image 0.0.2
[detector] Timer: Detector ran for 337.528419ms and ended at 2024-05-10T05:57:00Z
===> RESTORING
[restorer] Timer: Restorer started at 2024-05-10T05:57:01Z
[restorer] Timer: Restorer ran for 1.31471ms and ended at 2024-05-10T05:57:01Z
===> BUILDING
[builder] Timer: Builder started at 2024-05-10T05:57:01Z
[builder] === Node.js - Runtime (google.nodejs.runtime@1.0.0) ===
[builder] Using runtime version from GOOGLE_RUNTIME_VERSION: 14.3.0
[builder] ***** CACHE MISS: "nodejs"
[builder] Installing Node.js v14.3.0.
[builder] 2024/05/10 05:57:01 [DEBUG] GET https://dl.google.com/runtimes/ubuntu1804/nodejs/nodejs-14.3.0.tar.gz
[builder] === Node.js - Npm (google.nodejs.npm@1.1.0) ===
[builder] Generating package-lock.json.
[builder] WARNING: *** Improve build performance by generating and committing package-lock.json.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --package-lock-only --quiet"
[builder] npm WARN backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] added 64 packages and audited 64 packages in 2.904s
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --package-lock-only --quiet" (4.239193553s)
[builder] ***** CACHE MISS: "npm_modules"
[builder] Installing application dependencies.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm ci --quiet (NODE_ENV=production)"
[builder] added 64 packages in 0.858s
[builder] Done "npm ci --quiet (NODE_ENV=production)" (1.983429971s)
[builder] ***** CACHE MISS: "watchexec"
[builder] Installing watchexec v1.12.0
[builder] --------------------------------------------------------------------------------
[builder] Running "bash -c curl --fail --show-error --silent --location --retry 3 https://github.com/watchexec/watchexec/releases/download/1.12.0/watchexec-1.12.0-x86_64-unknown-linux-gnu.tar.xz | tar xJ --directory /layers/google.nodejs.npm/watchexec/bin --strip-components=1 --wildcards \"*watchexec\""
[builder] Done "bash -c curl --fail --show-error --silent --location --retry..." (1.967011907s)
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] === Utils - Label Image (google.utils.label-image@0.0.2) ===
[builder] Timer: Builder ran for 13.170213198s and ended at 2024-05-10T05:57:14Z
===> EXPORTING
[exporter] Timer: Exporter started at 2024-05-10T05:57:15Z
[exporter] Adding layer 'google.nodejs.runtime:node'
[exporter] Adding layer 'google.nodejs.npm:devmode_scripts'
[exporter] Adding layer 'google.nodejs.npm:env'
[exporter] Adding layer 'google.nodejs.npm:watchexec'
[exporter] Adding layer 'buildpacksio/lifecycle:launch.sbom'
[exporter] Adding 1/1 app layer(s)
[exporter] Adding layer 'buildpacksio/lifecycle:launcher'
[exporter] Adding layer 'buildpacksio/lifecycle:config'
[exporter] Adding layer 'buildpacksio/lifecycle:process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Timer: Saving skaffold-buildpacks-node:latest... started at 2024-05-10T05:57:20Z
[exporter] *** Images (8ee880df933d):
[exporter]       skaffold-buildpacks-node:latest
[exporter] Timer: Saving skaffold-buildpacks-node:latest... ran for 7.187477258s and ended at 2024-05-10T05:57:28Z
[exporter] Timer: Exporter ran for 12.344991689s and ended at 2024-05-10T05:57:28Z
[exporter] Timer: Cache started at 2024-05-10T05:57:28Z
[exporter] Adding cache layer 'google.nodejs.runtime:node'
[exporter] Adding cache layer 'google.nodejs.npm:npm_modules'
[exporter] Adding cache layer 'google.nodejs.npm:watchexec'
[exporter] Timer: Cache ran for 1.083789381s and ended at 2024-05-10T05:57:29Z
Build [skaffold-buildpacks-node] succeeded
Tags used in deployment:
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:8ee880df933d52ae4b7c4e65c1deded58153e237d29ab5682dbc89af83ba0f72
Starting deploy...
 - service/web created
 - deployment.apps/web created
Waiting for deployments to stabilize...
 - deployment/web is ready.
Deployments stabilized in 3.189 seconds
Listing files to watch...
 - skaffold-buildpacks-node
Press Ctrl+C to exit
Watching for changes...
[web] 
[web] > backend@1.0.0 start /workspace
[web] > node src/index.js
[web] 
[web] Example app listening on port 3000!


```

Notice how Skaffold automatically builds and deploys your application.  You should see the following application output in your terminal:

To browse to the web page, open a new terminal and run:

```terminal
minikube tunnel -p custom
```

```sh
(base) pradeep:~$minikube tunnel -p custom
Password:
Status:	
	machine: custom
	pid: 43954
	route: 10.96.0.0/12 -> 172.16.30.17
	minikube: Running
	services: [web]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors



```



Now open your browser at `http://localhost:3000`. This displays the content of `public/index.html` file.

Skaffold is now watching for any file changes, and will rebuild your application automatically. Let’s see this in action

Open `skaffold/examples/buildpacks-node-tutorial/src/index.js` and change line 10 to the following:

```sh
app.listen(port, () => console.log(`Example app listening on port ${port}! This is version 2.`))
```

Notice how Skaffold automatically hot reloads your code changes to your  application running in minikube, intelligently syncing only the file you changed. Your application is now automatically deployed with the  changes you made, as it prints the following to your terminal:

```sh
Generating tags...
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:v2.11.1-22-g0ffd37362-dirty
Checking cache...
 - skaffold-buildpacks-node: Not found. Building
Starting build...
Found [custom] context, using local docker daemon.
Building [skaffold-buildpacks-node]...
Target platforms: [linux/amd64]
===> ANALYZING
[analyzer] Timer: Analyzer started at 2024-05-10T06:02:50Z
[analyzer] Restoring data for SBOM from previous image
[analyzer] Timer: Analyzer ran for 15.289293ms and ended at 2024-05-10T06:02:50Z
===> DETECTING
[detector] Timer: Detector started at 2024-05-10T06:02:54Z
[detector] 3 of 5 buildpacks participating
[detector] google.nodejs.runtime    1.0.0
[detector] google.nodejs.npm        1.1.0
[detector] google.utils.label-image 0.0.2
[detector] Timer: Detector ran for 5.474599861s and ended at 2024-05-10T06:02:59Z
===> RESTORING
[restorer] Timer: Restorer started at 2024-05-10T06:03:04Z
[restorer] Restoring metadata for "google.nodejs.runtime:node" from app image
[restorer] Restoring metadata for "google.nodejs.npm:watchexec" from app image
[restorer] Restoring metadata for "google.nodejs.npm:devmode_scripts" from app image
[restorer] Restoring metadata for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.runtime:node" from cache
[restorer] Restoring data for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.npm:watchexec" from cache
[restorer] Timer: Restorer ran for 8.613737167s and ended at 2024-05-10T06:03:13Z
===> BUILDING
[builder] Timer: Builder started at 2024-05-10T06:03:18Z
[builder] === Node.js - Runtime (google.nodejs.runtime@1.0.0) ===
[builder] Using runtime version from GOOGLE_RUNTIME_VERSION: 14.3.0
[builder] ***** CACHE HIT: "nodejs"
[builder] Node.js v14.3.0 cache hit, skipping installation.
[builder] === Node.js - Npm (google.nodejs.npm@1.1.0) ===
[builder] Generating package-lock.json.
[builder] WARNING: *** Improve build performance by generating and committing package-lock.json.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --package-lock-only --quiet"
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] added 64 packages and audited 64 packages in 11.18s
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --package-lock-only --quiet" (14.969561393s)
[builder] ***** CACHE HIT: "npm_modules"
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --quiet (NODE_ENV=production)"
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] audited 64 packages in 7.983s
[builder] 
[builder] 12 packages are looking for funding
[builder]   run `npm fund` for details
[builder] 
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --quiet (NODE_ENV=production)" (13.547369034s)
[builder] ***** CACHE HIT: "watchexec"
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Timer: Builder ran for 29.67495216s and ended at 2024-05-10T06:03:47Z
[builder] === Utils - Label Image (google.utils.label-image@0.0.2) ===
===> EXPORTING
[exporter] Timer: Exporter started at 2024-05-10T06:03:55Z
[exporter] Adding layer 'google.nodejs.runtime:node'
[exporter] Adding layer 'google.nodejs.npm:devmode_scripts'
[exporter] Adding layer 'google.nodejs.npm:env'
[exporter] Adding layer 'google.nodejs.npm:watchexec'
[exporter] Reusing layer 'buildpacksio/lifecycle:launch.sbom'
[exporter] Adding 1/1 app layer(s)
[exporter] Reusing layer 'buildpacksio/lifecycle:launcher'
[exporter] Reusing layer 'buildpacksio/lifecycle:config'
[exporter] Reusing layer 'buildpacksio/lifecycle:process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Timer: Saving skaffold-buildpacks-node:latest... started at 2024-05-10T06:04:21Z
[exporter] *** Images (6dac0eb6fb77):
[exporter]       skaffold-buildpacks-node:latest
[exporter] Timer: Saving skaffold-buildpacks-node:latest... ran for 47.23599526s and ended at 2024-05-10T06:05:08Z
[exporter] Timer: Exporter ran for 1m13.269224649s and ended at 2024-05-10T06:05:08Z
[exporter] Timer: Cache started at 2024-05-10T06:05:08Z
[exporter] Adding cache layer 'google.nodejs.runtime:node'
[exporter] Adding cache layer 'google.nodejs.npm:npm_modules'
[exporter] Adding cache layer 'google.nodejs.npm:watchexec'
[exporter] Timer: Cache ran for 5.62498726s and ended at 2024-05-10T06:05:14Z
Build [skaffold-buildpacks-node] succeeded
Tags used in deployment:
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:6dac0eb6fb770d4b206b0b75e398c74c64db9e23a4161e45775752e7f4be6c60
Starting deploy...
 - deployment.apps/web configured
Waiting for deployments to stabilize...
 - deployment/web: creating container web
    - pod/web-859456b759-9qm2x: creating container web
 - deployment/web: waiting for rollout to finish: 1 old replicas are pending termination...
 - deployment/web is ready.
Deployments stabilized in 23.002 seconds
Watching for changes...
Generating tags...
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:v2.11.1-22-g0ffd37362-dirty
Checking cache...
 - skaffold-buildpacks-node: Not found. Building
Starting build...
Found [custom] context, using local docker daemon.
Building [skaffold-buildpacks-node]...
Target platforms: [linux/amd64]
===> ANALYZING
[analyzer] Timer: Analyzer started at 2024-05-10T06:05:53Z
[analyzer] Restoring data for SBOM from previous image
[analyzer] Timer: Analyzer ran for 43.015804ms and ended at 2024-05-10T06:05:53Z
===> DETECTING
[detector] Timer: Detector started at 2024-05-10T06:05:58Z
[detector] 3 of 5 buildpacks participating
[detector] google.nodejs.runtime    1.0.0
[detector] google.nodejs.npm        1.1.0
[detector] google.utils.label-image 0.0.2
[detector] Timer: Detector ran for 4.403171167s and ended at 2024-05-10T06:06:03Z
===> RESTORING
[restorer] Timer: Restorer started at 2024-05-10T06:06:08Z
[restorer] Restoring metadata for "google.nodejs.runtime:node" from app image
[restorer] Restoring metadata for "google.nodejs.npm:devmode_scripts" from app image
[restorer] Restoring metadata for "google.nodejs.npm:watchexec" from app image
[restorer] Restoring metadata for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.runtime:node" from cache
[restorer] Restoring data for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.npm:watchexec" from cache
[restorer] Timer: Restorer ran for 5.686630984s and ended at 2024-05-10T06:06:13Z
===> BUILDING
[builder] Timer: Builder started at 2024-05-10T06:06:20Z
[builder] === Node.js - Runtime (google.nodejs.runtime@1.0.0) ===
[builder] Using runtime version from GOOGLE_RUNTIME_VERSION: 14.3.0
[builder] ***** CACHE HIT: "nodejs"
[builder] Node.js v14.3.0 cache hit, skipping installation.
[builder] === Node.js - Npm (google.nodejs.npm@1.1.0) ===
[builder] Generating package-lock.json.
[builder] WARNING: *** Improve build performance by generating and committing package-lock.json.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --package-lock-only --quiet"
[builder] npm
[builder]  WARN backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] added 64 packages and audited 64 packages in 9.11s
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --package-lock-only --quiet" (12.702616329s)
[builder] ***** CACHE HIT: "npm_modules"
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --quiet (NODE_ENV=production)"
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] audited 64 packages in 3.625s
[builder] 
[builder] 12 packages are looking for funding
[builder]   run `npm fund` for details
[builder] 
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --quiet (NODE_ENV=production)" (6.755755618s)
[builder] ***** CACHE HIT: "watchexec"
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] Warning: BOM table is deprecated in this buildpack api version, though it remains supported for backwards compatibility. Buildpack authors should write BOM information to <layer>.sbom.<ext>, launch.sbom.<ext>, or build.sbom.<ext>.
[builder] === Utils - Label Image (google.utils.label-image@0.0.2) ===
[builder] Timer: Builder ran for 20.348534652s and ended at 2024-05-10T06:06:41Z
===> EXPORTING
[exporter] Timer: Exporter started at 2024-05-10T06:06:44Z
[exporter] Reusing layer 'google.nodejs.runtime:node'
[exporter] Reusing layer 'google.nodejs.npm:devmode_scripts'
[exporter] Reusing layer 'google.nodejs.npm:env'
[exporter] Reusing layer 'google.nodejs.npm:watchexec'
[exporter] Reusing layer 'buildpacksio/lifecycle:launch.sbom'
[exporter] Adding 1/1 app layer(s)
[exporter] Reusing layer 'buildpacksio/lifecycle:launcher'
[exporter] Reusing layer 'buildpacksio/lifecycle:config'
[exporter] Reusing layer 'buildpacksio/lifecycle:process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Timer: Saving skaffold-buildpacks-node:latest... started at 2024-05-10T06:06:54Z
[exporter] *** Images (6a1d675a2599):
[exporter]       skaffold-buildpacks-node:latest
[exporter] Timer: Saving skaffold-buildpacks-node:latest... ran for 5.525890575s and ended at 2024-05-10T06:07:00Z
[exporter] Timer: Exporter ran for 16.146054355s and ended at 2024-05-10T06:07:00Z
[exporter] Timer: Cache started at 2024-05-10T06:07:00Z
[exporter] Reusing cache layer 'google.nodejs.runtime:node'
[exporter] Reusing cache layer 'google.nodejs.npm:npm_modules'
[exporter] Reusing cache layer 'google.nodejs.npm:watchexec'
[exporter] Timer: Cache ran for 979.224159ms and ended at 2024-05-10T06:07:01Z
Build [skaffold-buildpacks-node] succeeded
Tags used in deployment:
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:6a1d675a25999fad25269aa055e59092a5365170b27f412035f27677e8c8d81a
Starting deploy...
 - deployment.apps/web configured
Waiting for deployments to stabilize...
 - deployment/web: creating container web
    - pod/web-7bc77bb876-swlz4: creating container web
 - deployment/web is ready.
Deployments stabilized in 6.351 seconds
Watching for changes...
[web] 
[web] > backend@1.0.0 start /workspace
[web] > node src/index.js
[web] 
[web] Example app listening on port 3000! This is version 2.


```

### Exit dev mode

Let’s stop continuous dev mode by pressing the following keys in your terminal:

```sh
[web] Example app listening on port 3000! This is version 2.
^CCleaning up...
 - service "web" deleted
 - deployment.apps "web" deleted

Help improve Skaffold with our 2-minute anonymous survey: run 'skaffold survey'
To help improve the quality of this product, we collect anonymized usage data for details on what is tracked and how we use this data visit <https://skaffold.dev/docs/resources/telemetry/>. This data is handled in accordance with our privacy policy <https://policies.google.com/privacy>

You may choose to opt out of this collection by running the following command:
	skaffold config set --global collect-metrics false
(base) pradeep:~$
```

Skaffold will clean up all deployed artifacts and end dev mode.

## Use Skaffold for continuous integration

While Skaffold shines for continuous development, it can also be used for continuous integration (CI). Let’s use Skaffold to build and test a container image.

### Build an image

Your CI pipelines can run `skaffold build` to build, tag, and push your container images to a registry.

```sh
(base) pradeep:~$export STATE=$(git rev-list -1 HEAD --abbrev-commit)
skaffold build --file-output build-$STATE.json
Generating tags...
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:v2.11.1-22-g0ffd37362-dirty
Checking cache...
 - skaffold-buildpacks-node: Not found. Building
Starting build...
Found [custom] context, using local docker daemon.
Building [skaffold-buildpacks-node]...
v1: Pulling from buildpacks/builder
Digest: sha256:76bedb4c2b82d4f086a398f39bafb7dec6a2e9c0484bb951efa5832aaa9db0a4
Status: Image is up to date for gcr.io/buildpacks/builder:v1
v1: Pulling from buildpacks/gcp/run
Digest: sha256:af071306ff95d9e79dd7e4d9513add7cb7a6e162752cc96ee3344188f1bf264e
Status: Image is up to date for gcr.io/buildpacks/gcp/run:v1
0.17.4: Pulling from buildpacksio/lifecycle
Digest: sha256:83f2d6ff5ef9e08e90565f0a00753490c53a8eab76a27483b68e491e776fdbd8
Status: Image is up to date for buildpacksio/lifecycle:0.17.4
===> ANALYZING
[analyzer] Timer: Analyzer started at 2024-05-10T06:16:52Z
[analyzer] Restoring data for SBOM from previous image
[analyzer] Timer: Analyzer ran for 4.005618ms and ended at 2024-05-10T06:16:52Z
===> DETECTING
[detector] Timer: Detector started at 2024-05-10T06:16:53Z
[detector] 3 of 5 buildpacks participating
[detector] google.nodejs.runtime    1.0.0
[detector] google.nodejs.npm        1.1.0
[detector] google.utils.label-image 0.0.2
[detector] Timer: Detector ran for 1.624693065s and ended at 2024-05-10T06:16:54Z
===> RESTORING
[restorer] Timer: Restorer started at 2024-05-10T06:16:55Z
[restorer] Restoring metadata for "google.nodejs.runtime:node" from app image
[restorer] Restoring metadata for "google.nodejs.npm:devmode_scripts" from app image
[restorer] Restoring metadata for "google.nodejs.npm:watchexec" from app image
[restorer] Restoring metadata for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.runtime:node" from cache
[restorer] Restoring data for "google.nodejs.npm:npm_modules" from cache
[restorer] Restoring data for "google.nodejs.npm:watchexec" from cache
[restorer] Timer: Restorer ran for 1.300893037s and ended at 2024-05-10T06:16:56Z
===> BUILDING
[builder] Timer: Builder started at 2024-05-10T06:16:57Z
[builder] === Node.js - Runtime (google.nodejs.runtime@1.0.0) ===
[builder] Using runtime version from GOOGLE_RUNTIME_VERSION: 14.3.0
[builder] ***** CACHE HIT: "nodejs"
[builder] Node.js v14.3.0 cache hit, skipping installation.
[builder] === Node.js - Npm (google.nodejs.npm@1.1.0) ===
[builder] Generating package-lock.json.
[builder] WARNING: *** Improve build performance by generating and committing package-lock.json.
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --package-lock-only --quiet"
[builder] npm WARN backend@1.0.0 No repository field.
[builder] npm WARN backend@1.0.0 No license field.
[builder] 
[builder] added 64 packages and audited 64 packages in 3.516s
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --package-lock-only --quiet" (5.314596693s)
[builder] ***** CACHE HIT: "npm_modules"
[builder] --------------------------------------------------------------------------------
[builder] Running "npm install --quiet (NODE_ENV=production)"
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No repository field.
[builder] npm
[builder]  
[builder] WARN
[builder]  backend@1.0.0 No license field.
[builder] 
[builder] audited 64 packages in 1.396s
[builder] 
[builder] 12 packages are looking for funding
[builder]   run `npm fund` for details
[builder] 
[builder] found 0 vulnerabilities
[builder] 
[builder] Done "npm install --quiet (NODE_ENV=production)" (2.445691297s)
[builder] === Utils - Label Image (google.utils.label-image@0.0.2) ===
[builder] Timer: Builder ran for 8.089179499s and ended at 2024-05-10T06:17:05Z
===> EXPORTING
[exporter] Timer: Exporter started at 2024-05-10T06:17:06Z
[exporter] Reusing layer 'google.nodejs.runtime:node'
[exporter] Reusing layer 'google.nodejs.npm:env'
[exporter] Adding layer 'buildpacksio/lifecycle:launch.sbom'
[exporter] Reusing 1/1 app layer(s)
[exporter] Reusing layer 'buildpacksio/lifecycle:launcher'
[exporter] Adding layer 'buildpacksio/lifecycle:config'
[exporter] Reusing layer 'buildpacksio/lifecycle:process-types'
[exporter] Adding label 'io.buildpacks.lifecycle.metadata'
[exporter] Adding label 'io.buildpacks.build.metadata'
[exporter] Adding label 'io.buildpacks.project.metadata'
[exporter] Setting default process type 'web'
[exporter] Timer: Saving skaffold-buildpacks-node:latest... started at 2024-05-10T06:17:09Z
[exporter] *** Images (fade976f3083):
[exporter]       skaffold-buildpacks-node:latest
[exporter] Timer: Saving skaffold-buildpacks-node:latest... ran for 2.560804971s and ended at 2024-05-10T06:17:12Z
[exporter] Timer: Exporter ran for 5.639388728s and ended at 2024-05-10T06:17:12Z
[exporter] Timer: Cache started at 2024-05-10T06:17:12Z
[exporter] Reusing cache layer 'google.nodejs.runtime:node'
[exporter] Reusing cache layer 'google.nodejs.npm:npm_modules'
[exporter] Timer: Cache ran for 310.228842ms and ended at 2024-05-10T06:17:12Z
Build [skaffold-buildpacks-node] succeeded

(base) pradeep:~$

```

Skaffold writes the output of the build to a JSON file, which we’ll pass to our continuous delivery (CD) process in the next step.

### Test an image

Skaffold can also run tests against your images before deploying them.  Let’s try this out by creating a simple custom test.

1. Open your`skaffold.yaml` and add the following test configuration to the bottom, without any additional indentation:

```yaml
(base) pradeep:~$cat skaffold.yaml 
apiVersion: skaffold/v4beta10
kind: Config
metadata:
  name: buildpacks-node-tutorial
build:
  artifacts:
    - image: skaffold-buildpacks-node
      buildpacks:
        builder: gcr.io/buildpacks/builder:v1
test:
- image: skaffold-buildpacks-node
  custom:
    - command: echo This is a custom test commmand!
manifests:
  rawYaml:
    - k8s/web.yaml
(base) pradeep:~$

```

Now you have a simple custom test set up that will run a bash command and await a successful response.

Run the following command to execute this test with Skaffold:

```sh
(base) pradeep:~$skaffold test --build-artifacts build-$STATE.json
Starting test...
Testing images...
Running custom test command: "echo This is a custom test commmand!"
This is a custom test commmand!
Command finished successfully.

(base) pradeep:~$

```

## Use Skaffold for continuous delivery

Let’s learn how Skaffold can handle continuous delivery (CD).

### Deploy in a single step

For simple deployments, run `skaffold deploy`:

```sh
(base) pradeep:~$skaffold deploy -a build-$STATE.json
Tags used in deployment:
 - skaffold-buildpacks-node -> skaffold-buildpacks-node:fade976f308369f72395d2220148e08306681c48b9608071e13d5162233349be
Starting deploy...
 - service/web created
 - deployment.apps/web created
Waiting for deployments to stabilize...
 - deployment/web is ready.
Deployments stabilized in 2.158 seconds

(base) pradeep:~$

```

Skaffold hydrates your Kubernetes manifest with the image you built and tagged in the previous step, and deploys the application.

### Render and apply in separate steps

For GitOps delivery workflows, you may want to decompose your deployments  into separate render and apply phases. That way, you can commit your  hydrated Kubernetes manifests to source control before they are applied.

Run the following command to render a hydrated manifest:

```sh
(base) pradeep:~$skaffold render -a build-$STATE.json --output render.yaml --digest-source local

(base) pradeep:~$ls
README.md		k8s			project.toml		render.yaml		src
build-0ffd37362.json	package.json		public			skaffold.yaml
(base) pradeep:~$
```

```yaml
(base) pradeep:~$cat  render.yaml 
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  ports:
    - name: http
      port: 3000
  selector:
    app: web
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - image: skaffold-buildpacks-node:fade976f308369f72395d2220148e08306681c48b9608071e13d5162233349be
          name: web
          ports:
            - containerPort: 3000
(base) pradeep:~$

```

Open `skaffold/examples/buildpacks-node-tutorial/render.yaml` to check out the hydrated manifest.

Next, run the following command to apply your hydrated manifest:

```sh
(base) pradeep:~$skaffold apply render.yaml

Starting deploy...
 - service/web configured
 - deployment.apps/web configured
Waiting for deployments to stabilize...
 - deployment/web is ready.
Deployments stabilized in 2.223 seconds

(base) pradeep:~$

```

```sh
(base) pradeep:~$kubectl get nodes
NAME     STATUS   ROLES           AGE   VERSION
custom   Ready    control-plane   39m   v1.30.0
(base) pradeep:~$kubectl get pods 
NAME                   READY   STATUS    RESTARTS   AGE
web-8664dfc774-9pngg   1/1     Running   0          21s
(base) pradeep:~$
(base) pradeep:~$kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           3m2s
(base) pradeep:~$kubectl get svc   
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1      <none>         443/TCP          39m
web          LoadBalancer   10.98.123.96   10.98.123.96   3000:32071/TCP   3m6s
(base) pradeep:~$

```

You have now successfully deployed your application in two ways.

## Congratulations, you successfully deployed with Skaffold!

You have learned how to use Skaffold for continuous development, integration, and delivery.
