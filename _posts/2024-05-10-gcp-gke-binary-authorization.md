---

layout: single
title:  "Google Kubernetes Engine Security: Binary Authorization "
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

# Google Kubernetes Engine Security: Binary Authorization

One of the key security concerns for running Kubernetes clusters is  knowing what container images are running inside each pod and being able to account for their origin.  Establishing "container provenance" means having the ability to trace the source of a container to a trusted  point of origin and ensuring your organization follows the desired  processes during artifact (container) creation.

Some of the key concerns are:

- **Safe Origin** - How do you ensure that all container images running in the cluster come from an approved source?
- **Consistency and Validation** - How do you ensure that all desired validation steps were completed successfully for every  container build and every deployment?
- **Integrity** - How do you ensure that containers were not modified before running after their provenance was proven?

From a security standpoint, not enforcing where images originate from presents several risks:

- A malicious actor that has compromised a container may be able to  obtain sufficient cluster privileges to launch other containers from an  unknown source without enforcement.
- An authorized user with the permissions to create pods may be able  to accidentally or maliciously run an undesired container directly  inside a cluster.
- An authorized user may accidentally or maliciously overwrite a  docker image tag with a functional container that has undesired code  silently added to it, and Kubernetes will pull and deploy that container as a part of a deployment automatically.

To help system operators address these concerns, Google Cloud offers a capability called [Binary Authorization](https://cloud.google.com/binary-authorization/). Binary Authorization is a Google Cloud managed service that works  closely with GKE to enforce deploy-time security controls to ensure that only trusted container images are deployed. With Binary Authorization  you can allowlist container registries, require images to be signed by  trusted authorities, and centrally enforce those policies. By enforcing  this policy, you can gain tighter control over your container  environment by ensuring only approved and/or verified images are  integrated into the build-and-release process.

This lab deploys a [Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) Cluster with the Binary Authorization feature enabled demonstrates how  to allowlist approved container registries, and walks you through the  process of creating and running a signed container.

## Architecture

The Binary Authorization and Container Analysis APIs are based upon the open-source projects Grafeas and Kritis.

- Grafeas defines an API spec for managing metadata about software  resources, such as container images, Virtual Machine (VM) images, JAR  files, and scripts. You can use Grafeas to define and aggregate  information about your project’s components.
- Kritis defines an API for ensuring a deployment is prevented unless  the artifact (container image) is conformant to central policy and  optionally has the necessary attestations present.

he container goes through at least 4 steps:

1. The source code for creating the container is stored in source control.
2. Upon committing a change to source control, the container is built and tested.
3. If the build and test steps are completed, the container image  artifact is then placed in a central container registry, ready for  deployment.
4. When a deployment of that container version is submitted to the  Kubernetes API, the container runtime will pull that container image  from the container registry and run it as a pod.

In a container build pipeline, there are opportunities to inject  additional processes to signify or "attest" that each step was completed successfully. Examples include running unit tests, source control  analysis checks, licensing verification, vulnerability analysis, and  more.  Each step could be given the power or "attestation authority" to  sign for that step being completed.  An "attestation authority" is a  human or system with the correct PGP key and the ability to register  that "attestation" with the Container Analysis API.

By using separate PGP keys for each step, each attestation step could be performed by different humans, systems, or build steps in the  pipeline (a).  Each PGP key is associated with an "attestation note"  which is stored in the Container Analysis API.  When a build step  "signs" an image, a snippet of JSON metadata about that image is signed  via PGP and that signed snippet is submitted to the API as a "note  occurrence".

Once the container image has been built and the necessary  attestations have been stored centrally, they are available for being  queried as a part of a policy decision process.  In this case, a [Kubernetes Admission Controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/), upon receiving an API request to `create` or `update` a Pod:

1. Send a WebHook to the Binary Authorization API for a policy decision.
2. The Binary Authorization policy is then consulted.
3. If necessary, the Container Analysis API is also queried for the necessary attestation occurrences.
4. If the container image conforms to the policy, it is allowed to run.
5. If the container image fails to meet the policy, an error is  presented to the API client with a message describing why it was  prevented.



## Copy resources

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-e98db2f05087.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_03aeec6f7a0c@cloudshell:~ (qwiklabs-gcp-04-e98db2f05087)$ gsutil -m cp -r gs://spls/gke-binary-auth/* .
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/.gitignore...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/Makefile...              
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/OWNERS...                
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/CONTRIBUTING.md...       
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/Jenkinsfile...           
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/LICENSE...               
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/create.sh...             
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/README.md...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/common.sh...             
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/README-QWIKLABS.md...    
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/delete.sh...             
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/Stackdriver_break-glass-filter.png...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/attest.png...     
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/attest1.png...    
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/attest2.png...    
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/binauthz1.png...  
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/blockallpolicy.png...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/editpolicy.png... 
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/enforce.png...    
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/initialpolicy.png...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/sdlc.png...       
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/images/whitelist.png...  
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.BUILD.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.Dockerfile.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.Makefile.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.WORKSPACE.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.bazel.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.bzl.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.css.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.go.preamble...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.go.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.html.preamble...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.html.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.java.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.js.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.py.preamble...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.py.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.scss.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.sh.preamble...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.sh.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.tf.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.ts.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.xml.preamble...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.yaml.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/boilerplate/boilerplate.xml.txt...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/verify_boilerplate.py...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/test/make.sh...
Copying gs://spls/gke-binary-auth/gke-binary-auth-demo/validate.sh...           
student_01_03aeec6f7a0c@cloudshell:~ (qwiklabs-gcp-04-e98db2f05087)$ cd gke-binary-auth-demo
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
WARNING: Property validation for compute/region was skipped.
Updated property [compute/region].
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ chmod +x create.sh
chmod +x delete.sh
chmod 777 validate.sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```
## Set default cluster version

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ sed -i 's/validMasterVersions\[0\]/defaultClusterVersion/g' ./create.sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ ./create.sh -c my-cluster-1
Fetching server config for us-central1-a
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Enabling the API compute.googleapis.com
Enabling the API container.googleapis.com
Enabling the API containerregistry.googleapis.com
Enabling the API containeranalysis.googleapis.com
Enabling the API binaryauthorization.googleapis.com
Operation "operations/acf.p2-186605527204-0014e151-04c4-4e24-a892-f79effdd071e" finished successfully.
Creating cluster
WARNING: The `--enable-binauthz` flag is deprecated. Please use `--binauthz-evaluation-mode` instead. 
Creating cluster my-cluster-1 in us-central1-a... Cluster is being health-checked (master is healthy)...done.                                                                      
Created [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-04-e98db2f05087/zones/us-central1-a/clusters/my-cluster-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/my-cluster-1?project=qwiklabs-gcp-04-e98db2f05087
kubeconfig entry generated for my-cluster-1.
NAME: my-cluster-1
LOCATION: us-central1-a
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.30.69.170
MACHINE_TYPE: n1-standard-1
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 2
STATUS: RUNNING
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster-1.
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

## Deployment steps

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat create.sh 
#!/usr/bin/env bash

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Creates a GKE Cluster                                -"
# "-                                                       -"
# "---------------------------------------------------------"

set -o errexit
set -o nounset
set -o pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
CLUSTER_NAME=""
ZONE=""
GKE_VERSION=$(gcloud container get-server-config \
  --format="value(defaultClusterVersion)")

# shellcheck source=./common.sh
source "$ROOT/common.sh"

# Ensure the required APIs are enabled
enable_project_api "${PROJECT}" "compute.googleapis.com"
enable_project_api "${PROJECT}" "container.googleapis.com"
enable_project_api "${PROJECT}" "containerregistry.googleapis.com"
enable_project_api "${PROJECT}" "containeranalysis.googleapis.com"
enable_project_api "${PROJECT}" "binaryauthorization.googleapis.com"

# Create a 2-node zonal GKE cluster
# Requires the Beta API to enable binary authorization support
echo "Creating cluster"
gcloud beta container clusters create "$CLUSTER_NAME" \
  --zone "$ZONE" \
  --cluster-version "$GKE_VERSION" \
  --machine-type "n1-standard-1" \
  --num-nodes=2 \
  --no-enable-basic-auth \
  --no-issue-client-certificate \
  --enable-ip-alias \
  --metadata disable-legacy-endpoints=true \
  --enable-binauthz

# Get the kubectl credentials for the GKE cluster.
gcloud container clusters get-credentials "$CLUSTER_NAME" --zone "$ZONE"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```
## Validation

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat validate.sh 
#!/usr/bin/env bash

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Validation script checks if the GKE cluster and the  -"
# "-  necessary APIs were deployed successfully.           -"
# "-                                                       -"
# "---------------------------------------------------------"

# Do no set exit on error, since the rollout status command may fail
set -o nounset
set -o pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
PROJECT=""
CLUSTER_NAME=""
ZONE=""

# shellcheck source=./common.sh
source "$ROOT/common.sh"

# Get credentials for the k8s cluster
gcloud container clusters get-credentials "$CLUSTER_NAME" --zone "$ZONE"

# Verify BinAuthZ policy is available/enabled
if gcloud beta container binauthz policy export | grep "defaultAdmissionRule" > /dev/null; then
  echo "Validation Passed: a working BinAuthZ policy was available"
else
  echo "Validation Failed: a working BinAuthZ policy was NOT available"
  exit 1
fi

# Verify Container Analysis API is available/enabled
if curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT}/notes/" > /dev/null; then
  echo "Validation Passed: the Container Analysis API was available"
else
  echo "Validation Failed: the Container Analysis API was NOT available"
  exit 1
fi
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ ./validate.sh -c my-cluster-1
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster-1.
Validation Passed: a working BinAuthZ policy was available
Validation Passed: the Container Analysis API was available
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```
## Using Binary Authorization

```yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz policy export > policy.yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat policy.yaml 
clusterAdmissionRules:
  us-central1-a.my-cluster-1:
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    evaluationMode: ALWAYS_ALLOW
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: ALWAYS_DENY
etag: '"MTHbw9ppQqhK"'
globalPolicyEvaluationMode: ENABLE
name: projects/qwiklabs-gcp-04-e98db2f05087/policy
updateTime: '2024-05-12T05:06:28.965039Z'
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

## Creating a private GCR image

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ docker pull gcr.io/google-containers/nginx:latest
latest: Pulling from google-containers/nginx
a3ed95caeb02: Pull complete 
e30706b9b4ff: Pull complete 
82286846aa71: Pull complete 
d433c42f6001: Pull complete 
74d2508996ef: Pull complete 
4fceae443db7: Pull complete 
577897efe0e1: Pull complete 
69d79999108f: Pull complete 
091f65bae2b8: Pull complete 
Digest: sha256:f49a843c290594dcf4d193535d1f4ba8af7d56cea2cf79d1e9554f077f1e7aaa
Status: Downloaded newer image for gcr.io/google-containers/nginx:latest
gcr.io/google-containers/nginx:latest
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud auth configure-docker
WARNING: Your config file at [/home/student_01_03aeec6f7a0c/.docker/config.json] contains these credential helper entries:

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
After update, the following will be written to your Docker config file located at [/home/student_01_03aeec6f7a0c/.docker/config.json]:
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
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ PROJECT_ID="$(gcloud config get-value project)"
Your active configuration is: [cloudshell-26215]
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ docker tag gcr.io/google-containers/nginx "gcr.io/${PROJECT_ID}/nginx:latest"
docker push "gcr.io/${PROJECT_ID}/nginx:latest"
The push refers to repository [gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx]
5f70bf18a086: Layer already exists 
42569d4ac115: Mounted from google-containers/nginx 
018e5230ea9b: Mounted from google-containers/nginx 
5aac04a72bda: Mounted from google-containers/nginx 
adb177e98c68: Mounted from google-containers/nginx 
0d9dceed9901: Mounted from google-containers/nginx 
e8061ac24ae3: Mounted from google-containers/nginx 
357b5eff542e: Mounted from google-containers/nginx 
4781101e0522: Mounted from google-containers/nginx 
latest: digest: sha256:02115bc7409dc044f700a729cacc1648dc4e445572e5765fbcc77029b0adba67 size: 4051
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud container images list-tags "gcr.io/${PROJECT_ID}/nginx"
DIGEST: 02115bc7409d
TAGS: latest
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF
pod/nginx created
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          3s
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          16s
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl delete pod nginx
pod "nginx" deleted
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl get pods
No resources found in default namespace.
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

## Denying all images

```yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz policy export > policy_modified.yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat policy_modified.yaml 
clusterAdmissionRules:
  us-central1-a.my-cluster-1:
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    evaluationMode: ALWAYS_DENY
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: ALWAYS_DENY
etag: '"Gt8iumzkM0IC"'
globalPolicyEvaluationMode: ENABLE
name: projects/qwiklabs-gcp-04-e98db2f05087/policy
updateTime: '2024-05-12T05:12:08.150271Z'
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF
Error from server (VIOLATES_POLICY): error when creating "STDIN": admission webhook "imagepolicywebhook.image-policy.k8s.io" denied the request: Image gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest denied by Binary Authorization cluster admission rule for us-central1-a.my-cluster-1. Denied by always_deny admission rule
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-03aeec6f7a0c@qwiklabs.net"
    },
    "authorizationInfo": [
      {
        "granted": true,
        "permission": "io.k8s.core.v1.pods.create",
        "resource": "core/v1/namespaces/default/pods/nginx"
      }
    ],
    "methodName": "io.k8s.core.v1.pods.create",
    "request": {
      "@type": "core.k8s.io/v1.Pod",
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "creationTimestamp": null,
        "name": "nginx",
        "namespace": "default"
      },
      "spec": {
        "containers": [
          {
            "image": "gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest",
            "imagePullPolicy": "Always",
            "name": "nginx",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File"
          }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "terminationGracePeriodSeconds": 30
      },
      "status": {}
    },
    "requestMetadata": {
      "callerIp": "34.143.180.117",
      "callerSuppliedUserAgent": "kubectl/v1.28.9 (linux/amd64) kubernetes/587f5fe"
    },
    "resourceName": "core/v1/namespaces/default/pods/nginx",
    "response": {
      "@type": "core.k8s.io/v1.Status",
      "apiVersion": "v1",
      "code": 400,
      "details": {
        "causes": [
          {
            "reason": "'gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest' : Denied by an ALWAYS_DENY admission rule\n"
          }
        ],
        "kind": "Pod"
      },
      "kind": "Status",
      "message": "admission webhook \"imagepolicywebhook.image-policy.k8s.io\" denied the request: Image gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest denied by Binary Authorization cluster admission rule for us-central1-a.my-cluster-1. Denied by always_deny admission rule",
      "metadata": {},
      "reason": "VIOLATES_POLICY",
      "status": "Failure"
    },
    "serviceName": "k8s.io",
    "status": {
      "code": 3,
      "message": "admission webhook \"imagepolicywebhook.image-policy.k8s.io\" denied the request: Image gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest denied by Binary Authorization cluster admission rule for us-central1-a.my-cluster-1. Denied by always_deny admission rule"
    }
  },
  "insertId": "5a3a3498-e69a-47f5-a02d-67bcc8f9efae",
  "resource": {
    "type": "k8s_cluster",
    "labels": {
      "cluster_name": "my-cluster-1",
      "location": "us-central1-a",
      "project_id": "qwiklabs-gcp-04-e98db2f05087"
    }
  },
  "timestamp": "2024-05-12T05:13:08.760303Z",
  "labels": {
    "authorization.k8s.io/reason": "access granted by IAM permissions.",
    "mutation.webhook.admission.k8s.io/round_0_index_4": "{\"configuration\":\"warden-mutating.config.common-webhooks.networking.gke.io\",\"webhook\":\"warden-mutating.common-webhooks.networking.gke.io\",\"mutated\":false}",
    "pod-security.kubernetes.io/enforce-policy": "privileged:latest",
    "authorization.k8s.io/decision": "allow",
    "mutation.webhook.admission.k8s.io/round_0_index_3": "{\"configuration\":\"pod-ready.config.common-webhooks.networking.gke.io\",\"webhook\":\"pod-ready.common-webhooks.networking.gke.io\",\"mutated\":false}"
  },
  "logName": "projects/qwiklabs-gcp-04-e98db2f05087/logs/cloudaudit.googleapis.com%2Factivity",
  "operation": {
    "id": "5a3a3498-e69a-47f5-a02d-67bcc8f9efae",
    "producer": "k8s.io",
    "first": true,
    "last": true
  },
  "receiveTimestamp": "2024-05-12T05:13:17.693033521Z"
}
```

## Denying images except from allowlisted container registries

```yaml
                                                                                          gcloud beta container binauthz policy export > policy_modified_2.yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat policy_modified_2.yaml 
admissionWhitelistPatterns:
- namePattern: gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx*
clusterAdmissionRules:
  us-central1-a.my-cluster-1:
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    evaluationMode: ALWAYS_DENY
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: ALWAYS_DENY
etag: '"Mna+BCqXznjH"'
globalPolicyEvaluationMode: ENABLE
name: projects/qwiklabs-gcp-04-e98db2f05087/policy
updateTime: '2024-05-12T05:17:42.444479Z'
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
EOF - containerPort: 80
pod/nginx created
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          24s
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl delete pod nginx
pod "nginx" deleted
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

## Enforcing attestations

Allowlisting container image registries is a great first step in  preventing undesired container images from being run inside a cluster,  but there is more you can do to ensure the container was built  correctly.

You want to cryptographically verify that a given container image was approved for deployment. This is done by an "attestation authority"  which states or *attests* to the fact that a certain step was completed.  The attestation authority does this by using a PGP key to *sign* a snippet of metadata describing the SHA256 hash of a container image  and submitting it to a central metadata repository--the Container  Analysis API.

Later, when the Admission Controller goes to validate if a container  image is allowed to run by consulting a Binary Authorization policy that requires attestations to be present on an image, it will check to see  if the Container Analysis API holds the signed snippet(s) of metadata  saying which steps were completed.  With that information, the Admission Controller will know whether to allow or deny that pod from running.

Next, perform a manual attestation of a container image.  You will  take on the role of a human attestation authority and will perform all  the steps to sign a container image, create a policy to require that  attestation to be present on images running inside your cluster, and  then successfully run that image in a pod.

## "Signing" a container image

The preceeding steps only need to be performed once.  From this point on, this step is the only step that needs repeating for every new  container image.



## Running an image with attestation enforcement enabled



## Handling emergency situations

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ ATTESTOR="manually-verified" # No spaces allowed
ATTESTOR_NAME="Manual Attestor"
ATTESTOR_EMAIL="$(gcloud config get-value core/account)" # This uses your current user/email
Your active configuration is: [cloudshell-26215]
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ NOTE_ID="Human-Attestor-Note" # No spaces
NOTE_DESC="Human Attestation Note Demo"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ NOTE_PAYLOAD_PATH="note_payload.json"
IAM_REQUEST_JSON="iam_request.json"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```json
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat > ${NOTE_PAYLOAD_PATH} << EOF
{
  "name": "projects/${PROJECT_ID}/notes/${NOTE_ID}",
  "attestation_authority": {
    "hint": {
      "human_readable_name": "${NOTE_DESC}"
    }
  }
}
EOF
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ curl -X POST \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $(gcloud auth print-access-token)"  \
    --data-binary @${NOTE_PAYLOAD_PATH}  \
    "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/?noteId=${NOTE_ID}"
{
  "name": "projects/qwiklabs-gcp-04-e98db2f05087/notes/Human-Attestor-Note",
  "kind": "ATTESTATION",
  "createTime": "2024-05-12T05:21:29.838942Z",
  "updateTime": "2024-05-12T05:21:29.838942Z",
  "attestationAuthority": {
    "hint": {
      "humanReadableName": "Human Attestation Note Demo"
    }
  }
}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ curl -H "Authorization: Bearer $(gcloud auth print-access-token)"  \
    "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"
{
  "name": "projects/qwiklabs-gcp-04-e98db2f05087/notes/Human-Attestor-Note",
  "kind": "ATTESTATION",
  "createTime": "2024-05-12T05:21:29.838942Z",
  "updateTime": "2024-05-12T05:21:29.838942Z",
  "attestationAuthority": {
    "hint": {
      "humanReadableName": "Human Attestation Note Demo"
    }
  }
}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ PGP_PUB_KEY="generated-key.pgp"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ sudo apt-get install rng-tools
********************************************************************************
You are running apt-get inside of Cloud Shell. Note that your Cloud Shell  
machine is ephemeral and no system-wide change will persist beyond session end. 

To suppress this warning, create an empty ~/.cloudshell/no-apt-get-warning file.
The command will automatically proceed in 5 seconds or on any key. 

Visit https://cloud.google.com/shell/help for more information.                 
********************************************************************************
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'rng-tools-debian' instead of 'rng-tools'
The following NEW packages will be installed:
  rng-tools-debian
0 upgraded, 1 newly installed, 0 to remove and 9 not upgraded.
Need to get 43.3 kB of archives.
After this operation, 131 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/universe amd64 rng-tools-debian amd64 2.3 [43.3 kB]
Fetched 43.3 kB in 1s (46.1 kB/s)           
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package rng-tools-debian.
(Reading database ... 121851 files and directories currently installed.)
Preparing to unpack .../rng-tools-debian_2.3_amd64.deb ...
Unpacking rng-tools-debian (2.3) ...
Setting up rng-tools-debian (2.3) ...
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.
Processing triggers for man-db (2.10.2-1) ...
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ sudo rngd -r /dev/urandom
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gpg --quick-generate-key --yes ${ATTESTOR_EMAIL}
gpg: directory '/home/student_01_03aeec6f7a0c/.gnupg' created
gpg: keybox '/home/student_01_03aeec6f7a0c/.gnupg/pubring.kbx' created
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/student_01_03aeec6f7a0c/.gnupg/trustdb.gpg: trustdb created
gpg: key 4B6C87B093542ADB marked as ultimately trusted
gpg: directory '/home/student_01_03aeec6f7a0c/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/student_01_03aeec6f7a0c/.gnupg/openpgp-revocs.d/3D87DC2675483B282B0202C44B6C87B093542ADB.rev'
public and secret key created and signed.

pub   rsa3072 2024-05-12 [SC] [expires: 2026-05-12]
      3D87DC2675483B282B0202C44B6C87B093542ADB
uid                      student-01-03aeec6f7a0c@qwiklabs.net
sub   rsa3072 2024-05-12 [E]

student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gpg --armor --export "${ATTESTOR_EMAIL}" > ${PGP_PUB_KEY}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud --project="${PROJECT_ID}" \
    beta container binauthz attestors create "${ATTESTOR}" \
    --attestation-authority-note="${NOTE_ID}" \
    --attestation-authority-note-project="${PROJECT_ID}"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud --project="${PROJECT_ID}" \
    beta container binauthz attestors public-keys add \
    --attestor="${ATTESTOR}" \
    --pgp-public-key-file="${PGP_PUB_KEY}"
asciiArmoredPgpPublicKey: |
  -----BEGIN PGP PUBLIC KEY BLOCK-----

  mQGNBGZAUj8BDADFmFbkEK65GA7vIneClFRISjyJtxXv63HftILeCBNtlP/7CO9P
  MV9Qi2yYh6KARMk3oJKYOw8wPM0QWOOTHXGKL1QP89vVz/0YZPOcfsro9O3Ghn6u
  nWKpO8mPlNCVmSpNXcWD0NWS/w48xpfVqvjr9DbNazoQcWrjVdYurNFFF3WjReA6
  LIClaySfnzoD7123FBDTQbfecMJTeVWJ8zSBXqEQqI5IwGbHdZGiZYJsqjwgEoVh
  klfWsCLSDb+42Ik0+NNAtwWVF9aZ+V/ztg/HRn1G73OAz3/ng/EEfuPfc/OPEYWJ
  RK0ICXO4ASZarPjweaAgDCt/Q9SoqClhmXqyk1OrBqsDuYnIcgoiYweZ3JrM4PwY
  8QYGUfmbRwUxDbu/XjtggbDE5oUp/9Z7ChY2nPben/LcMyd0OxXwMk7PHcNpDPnl
  pci4yMVg8vGvngnlx/nUGcJj2rIGudu0BUHab/rrcYWFDIqSNPALpZYLw3pWo4gM
  iGA7MaKWddDpss0AEQEAAbQkc3R1ZGVudC0wMS0wM2FlZWM2ZjdhMGNAcXdpa2xh
  YnMubmV0iQHUBBMBCgA+FiEEPYfcJnVIOygrAgLES2yHsJNUKtsFAmZAUj8CGwMF
  CQPCZwAFCwkIBwIGFQoJCAsCBBYCAwECHgECF4AACgkQS2yHsJNUKtun+AwAlrct
  swlHqn45gkUgwGzQQewybUhHOx6gNy0+t2V+ZvOrde1L4gavNkLQnsYEzHaGfn/v
  74+UdHad98dm11+us4zMfxpjEqp6lvgdjFpiOQANcxNdpsk55Ujbrwyy9oDalqEp
  Bxkf+Fviyt8DII++pcwqdu13FiGNNWihiM71pkikRn81QrqRMMT6WFPz3yflHUdN
  d5IzlZSjpKTkWrN26oWWWbLAoFwiXLC86+3uYQSDi9KBu51pkoJrzBfVdjZsFnmk
  n1Ui7ET6Dh8da0QBY1vgOmgybcI5WP5osBSL07kuZGwI8dVoCURdPgTUPqKg+gUE
  CSxM7N5Zo6U36xNMWh14QwFXFcZeoJ/w4KchEGCrX1+MlEZjjMV8P2sMRcP5i65a
  rlJRsUK6fC2SkADmlZ+3YQtpYKBAQ29SqSYbUrRvsvqaMGw6IFqRXuD+OwKBeCIi
  T0s3zK36MFdqbeqqQPtliqqeGEMxJd3PAzRTxKCMyXhVK4boCwadpbi7jqtguQGN
  BGZAUj8BDADFx4nAM1Rw+T8RuqnToIFdUoFbnUUdwXUuVjcBWZgAfeY42c9/zJJc
  wzUe6dupK/uGK8MAddzzp9cPrUBWpjLDCgTCyWQRjBb47yLpXh3if2rQVJ7QN/ds
  JWoB3vW0Zt2+6R8+gcQex0VvIcdOpx+sp2o0eUlpCCvQ0ElzTgWqNNeMEglN7HjA
  xeGNnTgYRbpm4ymNbnKCMk63l7s4Ybig1iuXGBYF9/R/KQCTOyClnoUxETNO2dRq
  Dzao3fumZDpkDdftXPMDGNcuPmGkKk9kcytIynpmrtRI4x9aD4FpFAOyJrv/iYin
  zq67m82rJuXlylyLhla5qYI+oaoDdeSNI3B5aHVOzRyvTaevelzNq7lWVkG740Jc
  atyDDLssJqLL4tkhCoxF268+l7ffFt+SBCBtvTQyupwLUFDMFp5zPOPCUNLBka0D
  EsPIATHFGlEfrJGdRwfnzOT25Yn6wXRKPfwxnxZ8jlfad1Ki041/Ev9AjqLUK30Y
  iNDNMFw4idkAEQEAAYkBtgQYAQoAIBYhBD2H3CZ1SDsoKwICxEtsh7CTVCrbBQJm
  QFI/AhsMAAoJEEtsh7CTVCrbL3kL/1t3li0PUK9RXE+VGA3iIjSZ1kscrBBbbJnW
  5TmVA0zaxov9iHIm0pdxhZPpWFnLSsEq4Jxxv+lnqxvlN+GWjDsuek4yKWdik2fy
  so2oLUPw0tHeiEA516LHoScm+CU17sR1BxlpuMzxrx9KVeQ6p9oDuRpaqJSfN24L
  AhXfndHFQMyTcIJafGz/J9gfVHKotMjF9cheyPoCPYJos8NLz9H3qL9UZVlMTYIy
  QrIvhD01Pl+wKqAXyKjOU/0a+axhmTTg0YiFYe82clWY6QSg1eVs+dI4HhoI86jJ
  vfow2TQv0/uYY5F6HFH4DAk0mzy0xtUab6TcYpYXJ0wQ/2jWNjUnFCPZvgGCU38c
  wIMWfNXkAF1LCWUECUcnHR8fZOslsGoGR/Ti1b+RA1plODTvOl5jmGYhs51NT/Tr
  T9qdG9btisHw/4AD1DfJEuvFd4czJEE2ZDeEM7KkWYKU5Al3pfHKJ4HF00BboP9o
  M7nO5cdXJAD8Oq1PrgaoZBfJuLm4wA==
  =pc9D
  -----END PGP PUBLIC KEY BLOCK-----
id: 3D87DC2675483B282B0202C44B6C87B093542ADB
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud --project="${PROJECT_ID}" \
    beta container binauthz attestors list
NAME: manually-verified
NOTE: projects/qwiklabs-gcp-04-e98db2f05087/notes/Human-Attestor-Note
NUM_PUBLIC_KEYS: 1
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ GENERATED_PAYLOAD="generated_payload.json"
GENERATED_SIGNATURE="generated_signature.pgp"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ PGP_FINGERPRINT="$(gpg --list-keys ${ATTESTOR_EMAIL} | head -2 | tail -1 | awk '{print $1}')"
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2026-05-12
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"
IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz create-signature-payload \
    --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}" > ${GENERATED_PAYLOAD}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat "${GENERATED_PAYLOAD}"
{
  "critical": {
    "identity": {
      "docker-reference": "gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx"
    },
    "image": {
      "docker-manifest-digest": "sha256:02115bc7409dc044f700a729cacc1648dc4e445572e5765fbcc77029b0adba67"
    },
    "type": "Google cloud binauthz container signature"
  }
}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gpg --local-user "${ATTESTOR_EMAIL}" \
    --armor \
    --output ${GENERATED_SIGNATURE} \
    --sign ${GENERATED_PAYLOAD}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat "${GENERATED_SIGNATURE}"
-----BEGIN PGP MESSAGE-----

owGbwMvMwMXondO+YXKI1m3GtYzeSWLpqXmpRYklqSnxBYmVOfmJKXpZxfl5aQ4h
YtVcCgpKyUWZJZnJiTlKVgogPlAkMyU1rySzpBIuAhRLyU/OTi3SLUpNSy1KzUtO
BcoppScX6WXm6xeWZ2bnJCYV66YnF+gamOimWlqkJBmlGZgaWJjr56Vn5lUogU2p
1YEan5uYnorF7NzEvMy01OIS3ZTMdCAFsqE4I9HI1MzKwMjQ0DQp2dzEwDIl2cDE
JM3cwCDR3MgyOTE52dDMxCIl2STVxMTU1Nwo1dTczDQtKTnZ3NzAyDLJIDElKdHM
HNX+ksoCsPPd8/PTc1IVknPyS1MUkjLzEktLMqoUkvPzShIzgSGmUJyZnpdYUlqU
CtJey1XL1cl4k4WBkYvBWUyRxbb9jlqph7WGNhPTEVhwszKBAlVVprikFBSCugaG
ugbGiampyWZp5okGyQ6wgNLLSy1h4OIUgOkTDOdhaN7Zvnla1KXkdyrHF3onr9x9
lmvLz1Kmt8fsu68LH5ZfY6ZybHUHj9fjKx05J6WPTjafefRonNvxN4LvLRib1hzf
ODFgluu/Dq2lirf716e8TBGYnee0fOHRXHGFky21TK2PtjxlZ6tbZr24atH7c4K/
75epvfmxKTxbdLWmd5fMevHT17dxFn5csHR3sJNeCoMTe/PCV9pZKcKfjlwXmaHu
M/n26TU7Km6siHm8aIbe2daZR44Lsl01W3rF7nDTiiAbhiliAXqRNUFufbfU9uzc
6Hlcdn0Ly0f+ppvH+eTTTi9/lNp7sCqwLKW1O1H0uP83dvYSQW/VbLUdZhxZRz9P
Nz5u6P8z76zP68qYZt7ExLW7/907o97zYPEyo8rZEw4o2d+5J7VI1adV0q1CoexA
gULP9Te7jk9fu7G2tXle+7zKBHMD49BdKdwNxaIGAnnnDrn+1LmXKBSc2+R2dqad
TcxVn97+Y/fvfijfEVTQGSWc9Ke24U3dRWX5far/d2Q9nW37aZ9f8ftbWxe+27T2
1t35+gA=
=e5CO
-----END PGP MESSAGE-----
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz attestations create \
    --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}" \
    --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}" \
    --signature-file=${GENERATED_SIGNATURE} \
    --public-key-id="${PGP_FINGERPRINT}"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz attestations list \
    --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}"
---
attestation:
  serializedPayload: ewogICJjcml0aWNhbCI6IHsKICAgICJpZGVudGl0eSI6IHsKICAgICAgImRvY2tlci1yZWZlcmVuY2UiOiAiZ2NyLmlvL3F3aWtsYWJzLWdjcC0wNC1lOThkYjJmMDUwODcvbmdpbngiCiAgICB9LAogICAgImltYWdlIjogewogICAgICAiZG9ja2VyLW1hbmlmZXN0LWRpZ2VzdCI6ICJzaGEyNTY6MDIxMTViYzc0MDlkYzA0NGY3MDBhNzI5Y2FjYzE2NDhkYzRlNDQ1NTcyZTU3NjVmYmNjNzcwMjliMGFkYmE2NyIKICAgIH0sCiAgICAidHlwZSI6ICJHb29nbGUgY2xvdWQgYmluYXV0aHogY29udGFpbmVyIHNpZ25hdHVyZSIKICB9Cn0K
  signatures:
  - publicKeyId: 3D87DC2675483B282B0202C44B6C87B093542ADB
    signature: LS0tLS1CRUdJTiBQR1AgTUVTU0FHRS0tLS0tCgpvd0did012TXdNWG9uZE8rWVhLSTFtM0d0WXplU1dMcHFYbXBSWWtscVNueEJZbVZPZm1KS1hwWnhmbDVhUTRoCll0VmNDZ3BLeVVXWkpabkppVGxLVmdvZ1BsQWtNeVUxcnlTenBCSXVBaFJMeVUvT1RpM1NMVXBOU3kxS3pVdE8KQmNvcHBTY1g2V1htNnhlV1oyYm5KQ1lWNjZZbkYrZ2FtT2ltV2xxa0pCbWxHWmdhV0pqcjU2Vm41bFVvZ1UycAoxWUVhbjV1WW5vckY3TnpFdk15MDFPSVMzWlRNZENBRnNxRTRJOUhJMU16S3dNalEwRFFwMmR6RXdESWwyY0RFCkpNM2N3Q0RSM01neU9URTUyZERNeENJbDJTVFZ4TVRVMU53bzFkVGN6RFF0S1RuWjNOekF5RExKSURFbEtkSE0KSE5YK2tzb0NzUFBkOC9QVGMxSVZrblB5UzFNVWtqTHpFa3RMTXFvVWt2UHpTaEl6Z1NHbVVKeVpucGRZVWxxVQpDdEpleTFYTDFjbDRrNFdCa1l2QldVeVJ4YmI5amxxcGg3V0dOaFBURVZod3N6S0JBbFZWcHJpa0ZCU0N1Z2FHCnVnYkdpYW1weVdacDVva0d5UTZ3Z05MTFN5MWg0T0lVZ09rVERPZGhhTjdadm5sYTFLWGtkeXJIRjNvbnI5eDkKbG12THoxS210OGZzdTY4TEg1WmZZNlp5YkhVSGo5ZmpLeDA1SjZXUFRqYWZlZlJvbk52eE40THZMUmliMWh6ZgpPREZnbHV1L0RxMmxpcmY3MTZlOFRCR1luZWUwZk9IUlhIR0ZreTIxVEsyUHRqeGxaNnRiWnIyNGF0SDdjNEsvCjc1ZXB2Zm14S1R4YmRMV21kNWZNZXZIVDE3ZHhGbjVjc0hSM3NKTmVDb01UZS9QQ1Y5cFpLY0tmamx3WG1hSHUKTS9uMjZUVTdLbTZzaUhtOGFJYmUyZGFaUjQ0THNsMDFXM3JGN25EVGlpQWJoaWxpQVhxUk5VRnVmYmZVOXV6Ywo2SGxjZG4wTHkwZitwcHZIK2VUVFRpOS9sTnA3c0Nxd0xLVzFPMUgwdVA4M2R2WVNRVy9WYkxVZFpoeFpSejlQCk56NXU2UDh6NzZ6UDY4cVladDdFeExXNy85MDdvOTd6WVBFeW84clpFdzRvMmQrNUo3VkkxYWRWMHExQ29leEEKZ1VMUDlUZTdqazlmdTdHMnRYbGUrN3pLQkhNRDQ5QmRLZHdOeGFJR0Fubm5Ecm4rMUxtWEtCU2MyK1IyZHFhZApUY3hWbjk3K1kvZnZmaWpmRVZUUUdTV2M5S2UyNFUzZFJXWDVmYXIvZDJROW5XMzdhWjlmOGZ0Yld4ZSsyN1QyCjF0MzUrZ0E9Cj1lNUNPCi0tLS0tRU5EIFBHUCBNRVNTQUdFLS0tLS0K
createTime: '2024-05-12T05:31:37.647232Z'
kind: ATTESTATION
name: projects/qwiklabs-gcp-04-e98db2f05087/occurrences/fc538d77-4f3f-4418-91d8-425bc42916d4
noteName: projects/qwiklabs-gcp-04-e98db2f05087/notes/Human-Attestor-Note
resourceUri: gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx@sha256:02115bc7409dc044f700a729cacc1648dc4e445572e5765fbcc77029b0adba67
updateTime: '2024-05-12T05:31:37.647232Z'
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ echo "projects/${PROJECT_ID}/attestors/${ATTESTOR}" # Copy this output to your copy/paste buffer
projects/qwiklabs-gcp-04-e98db2f05087/attestors/manually-verified
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud beta container binauthz policy export > policy_modified_3.yaml
cstudent_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat policy_modified_3.yaml 
admissionWhitelistPatterns:
- namePattern: gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx*
clusterAdmissionRules:
  us-central1-a.my-cluster-1:
    enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
    evaluationMode: REQUIRE_ATTESTATION
    requireAttestationsBy:
    - projects/qwiklabs-gcp-04-e98db2f05087/attestors/manually-verified
defaultAdmissionRule:
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  evaluationMode: ALWAYS_DENY
etag: '"GDXq/WrZN7Kp"'
globalPolicyEvaluationMode: ENABLE
name: projects/qwiklabs-gcp-04-e98db2f05087/policy
updateTime: '2024-05-12T05:34:29.325967Z'
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"
IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "${IMAGE_PATH}@${IMAGE_DIGEST}"
    ports:
EOF - containerPort: 80
pod/nginx created
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```yaml
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-alpha
  annotations:
    alpha.image-policy.k8s.io/break-glass: "true"
spec:
  containers:
  - name: nginx
EOF - containerPort: 80t"
pod/nginx-alpha created
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx         1/1     Running   0          67s
nginx-alpha   1/1     Running   0          21s
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
resource.type="k8s_cluster" protoPayload.request.metadata.annotations."alpha.image-policy.k8s.io/break-glass"="true"
```



```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-03aeec6f7a0c@qwiklabs.net"
    },
    "authorizationInfo": [
      {
        "granted": true,
        "permission": "io.k8s.core.v1.pods.create",
        "resource": "core/v1/namespaces/default/pods/nginx-alpha"
      }
    ],
    "methodName": "io.k8s.core.v1.pods.create",
    "request": {
      "@type": "core.k8s.io/v1.Pod",
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "annotations": {
          "alpha.image-policy.k8s.io/break-glass": "true"
        },
        "creationTimestamp": null,
        "name": "nginx-alpha",
        "namespace": "default"
      },
      "spec": {
        "containers": [
          {
            "image": "nginx:latest",
            "imagePullPolicy": "Always",
            "name": "nginx",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File"
          }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "terminationGracePeriodSeconds": 30
      },
      "status": {}
    },
    "requestMetadata": {
      "callerIp": "34.143.180.117",
      "callerSuppliedUserAgent": "kubectl/v1.28.9 (linux/amd64) kubernetes/587f5fe"
    },
    "resourceName": "core/v1/namespaces/default/pods/nginx-alpha",
    "response": {
      "@type": "core.k8s.io/v1.Pod",
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "annotations": {
          "alpha.image-policy.k8s.io/break-glass": "true"
        },
        "creationTimestamp": "2024-05-12T05:37:32Z",
        "managedFields": [
          {
            "apiVersion": "v1",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:alpha.image-policy.k8s.io/break-glass": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"nginx\"}": {
                    ".": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:ports": {
                      ".": {},
                      "k:{\"containerPort\":80,\"protocol\":\"TCP\"}": {
                        ".": {},
                        "f:containerPort": {},
                        "f:protocol": {}
                      }
                    },
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            },
            "manager": "kubectl-create",
            "operation": "Update",
            "time": "2024-05-12T05:37:32Z"
          }
        ],
        "name": "nginx-alpha",
        "namespace": "default",
        "resourceVersion": "22786",
        "uid": "3367719c-88ea-454c-ba14-701ef8489c9e"
      },
      "spec": {
        "containers": [
          {
            "image": "nginx:latest",
            "imagePullPolicy": "Always",
            "name": "nginx",
            "ports": [
              {
                "containerPort": 80,
                "protocol": "TCP"
              }
            ],
            "resources": {},
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "volumeMounts": [
              {
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                "name": "kube-api-access-j7qzr",
                "readOnly": true
              }
            ]
          }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority",
        "priority": 0,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "serviceAccount": "default",
        "serviceAccountName": "default",
        "terminationGracePeriodSeconds": 30,
        "tolerations": [
          {
            "effect": "NoExecute",
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "tolerationSeconds": 300
          },
          {
            "effect": "NoExecute",
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "tolerationSeconds": 300
          }
        ],
        "volumes": [
          {
            "name": "kube-api-access-j7qzr",
            "projected": {
              "defaultMode": 420,
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ],
                    "name": "kube-root-ca.crt"
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        },
                        "path": "namespace"
                      }
                    ]
                  }
                }
              ]
            }
          }
        ]
      },
      "status": {
        "phase": "Pending",
        "qosClass": "BestEffort"
      }
    },
    "serviceName": "k8s.io",
    "status": {
      "code": 0
    }
  },
  "insertId": "0e496604-1743-4232-80f8-34c2456bad07",
  "resource": {
    "type": "k8s_cluster",
    "labels": {
      "cluster_name": "my-cluster-1",
      "location": "us-central1-a",
      "project_id": "qwiklabs-gcp-04-e98db2f05087"
    }
  },
  "timestamp": "2024-05-12T05:37:32.702702Z",
  "labels": {
    "pod-security.kubernetes.io/enforce-policy": "privileged:latest",
    "mutation.webhook.admission.k8s.io/round_0_index_3": "{\"configuration\":\"pod-ready.config.common-webhooks.networking.gke.io\",\"webhook\":\"pod-ready.common-webhooks.networking.gke.io\",\"mutated\":false}",
    "authorization.k8s.io/reason": "access granted by IAM permissions.",
    "mutation.webhook.admission.k8s.io/round_0_index_4": "{\"configuration\":\"warden-mutating.config.common-webhooks.networking.gke.io\",\"webhook\":\"warden-mutating.common-webhooks.networking.gke.io\",\"mutated\":false}",
    "imagepolicywebhook.image-policy.k8s.io/break-glass": "true",
    "imagepolicywebhook.image-policy.k8s.io/overridden-verification-result": "'nginx:latest' : Image nginx:latest denied by attestor projects/qwiklabs-gcp-04-e98db2f05087/attestors/manually-verified: Expected digest with sha256 scheme, but got tag or malformed digest\n",
    "authorization.k8s.io/decision": "allow"
  },
  "logName": "projects/qwiklabs-gcp-04-e98db2f05087/logs/cloudaudit.googleapis.com%2Factivity",
  "operation": {
    "id": "0e496604-1743-4232-80f8-34c2456bad07",
    "producer": "k8s.io",
    "first": true,
    "last": true
  },
  "receiveTimestamp": "2024-05-12T05:37:33.211963825Z"
}
```

## Tear down

Qwiklabs will remove all resources you created for this lab, but it's good to know how to clean up your own environment.


```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ cat delete.sh 
#!/usr/bin/env bash

# Copyright 2018 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# "---------------------------------------------------------"
# "-                                                       -"
# "-  Delete deletes the GKE Cluster                       -"
# "-                                                       -"
# "---------------------------------------------------------"

# Do not set errexit as it makes partial deletes impossible
set -o nounset
set -o pipefail

ROOT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
CLUSTER_NAME=""
ZONE=""

# shellcheck source=./common.sh
source "$ROOT/common.sh"

# Get credentials for the k8s cluster
gcloud container clusters get-credentials "$CLUSTER_NAME" --zone "$ZONE"

# Cleanup the cluster
echo "Deleting cluster"
gcloud container clusters delete "$CLUSTER_NAME" --zone "$ZONE" --async --quiet
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```



```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ ./delete.sh -c my-cluster-1
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Your active configuration is: [cloudshell-26215]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for my-cluster-1.
Deleting cluster
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud container images delete "${IMAGE_PATH}@${IMAGE_DIGEST}" --force-delete-tags
Digests:
- gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx@sha256:02115bc7409dc044f700a729cacc1648dc4e445572e5765fbcc77029b0adba67
  Associated tags:
 - latest
This operation will delete the tags and images identified by the digests above.

Do you want to continue (Y/n)?  y

Deleted [gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx:latest].
Deleted [gcr.io/qwiklabs-gcp-04-e98db2f05087/nginx@sha256:02115bc7409dc044f700a729cacc1648dc4e445572e5765fbcc77029b0adba67].
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ gcloud --project="${PROJECT_ID}" \
    beta container binauthz attestors delete "${ATTESTOR}"
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)
```

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ curl -X DELETE \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"
{}
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

## Summary

```sh
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ history 
    1  gsutil -m cp -r gs://spls/gke-binary-auth/* .
    2  cd gke-binary-auth-demo
    3  gcloud config set compute/region us-central1
    4  gcloud config set compute/zone us-central1-a
    5  chmod +x create.sh
    6  chmod +x delete.sh
    7  chmod 777 validate.sh
    8  sed -i 's/validMasterVersions\[0\]/defaultClusterVersion/g' ./create.sh
    9  ./create.sh -c my-cluster-1
   10  cat create.sh 
   11  cat validate.sh 
   12  ./validate.sh -c my-cluster-1
   13  gcloud beta container binauthz policy export > policy.yaml
   14  cat policy.yaml 
   15  docker pull gcr.io/google-containers/nginx:latest
   16  gcloud auth configure-docker
   17  PROJECT_ID="$(gcloud config get-value project)"
   18  docker tag gcr.io/google-containers/nginx "gcr.io/${PROJECT_ID}/nginx:latest"
   19  docker push "gcr.io/${PROJECT_ID}/nginx:latest"
   20  gcloud container images list-tags "gcr.io/${PROJECT_ID}/nginx"
   21  cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF

   22  kubectl get pods
   23  kubectl delete pod nginx
   24  kubectl get pods
   25  gcloud beta container binauthz policy export > policy_modified.yaml
   26  cat policy_modified.yaml 
   27  cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF

   28  echo "gcr.io/${PROJECT_ID}/nginx*"
   29  gcloud beta container binauthz policy export > policy_modified_2.yaml
   30  cat policy_modified_2.yaml 
   31  cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "gcr.io/${PROJECT_ID}/nginx:latest"
    ports:
    - containerPort: 80
EOF

   32  kubectl get pods
   33  kubectl delete pod nginx
   34  ATTESTOR="manually-verified" # No spaces allowed
   35  ATTESTOR_NAME="Manual Attestor"
   36  ATTESTOR_EMAIL="$(gcloud config get-value core/account)" # This uses your current user/email
   37  NOTE_ID="Human-Attestor-Note" # No spaces
   38  NOTE_DESC="Human Attestation Note Demo"
   39  NOTE_PAYLOAD_PATH="note_payload.json"
   40  IAM_REQUEST_JSON="iam_request.json"
   41  cat > ${NOTE_PAYLOAD_PATH} << EOF
{
  "name": "projects/${PROJECT_ID}/notes/${NOTE_ID}",
  "attestation_authority": {
    "hint": {
      "human_readable_name": "${NOTE_DESC}"
    }
  }
}
EOF

   42  curl -X POST     -H "Content-Type: application/json"     -H "Authorization: Bearer $(gcloud auth print-access-token)"      --data-binary @${NOTE_PAYLOAD_PATH}      "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/?noteId=${NOTE_ID}"
   43  curl -H "Authorization: Bearer $(gcloud auth print-access-token)"      "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"
   44  PGP_PUB_KEY="generated-key.pgp"
   45  sudo apt-get install rng-tools
   46  sudo rngd -r /dev/urandom
   47  gpg --quick-generate-key --yes ${ATTESTOR_EMAIL}
   48  gpg --armor --export "${ATTESTOR_EMAIL}" > ${PGP_PUB_KEY}
   49  gcloud --project="${PROJECT_ID}"     beta container binauthz attestors create "${ATTESTOR}"     --attestation-authority-note="${NOTE_ID}"     --attestation-authority-note-project="${PROJECT_ID}"
   50  gcloud --project="${PROJECT_ID}"     beta container binauthz attestors public-keys add     --attestor="${ATTESTOR}"     --pgp-public-key-file="${PGP_PUB_KEY}"
   51  gcloud --project="${PROJECT_ID}"     beta container binauthz attestors list
   52  GENERATED_PAYLOAD="generated_payload.json"
   53  GENERATED_SIGNATURE="generated_signature.pgp"
   54  PGP_FINGERPRINT="$(gpg --list-keys ${ATTESTOR_EMAIL} | head -2 | tail -1 | awk '{print $1}')"
   55  IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"
   56  IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"
   57  gcloud beta container binauthz create-signature-payload     --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}" > ${GENERATED_PAYLOAD}
   58  cat "${GENERATED_PAYLOAD}"
   59  gpg --local-user "${ATTESTOR_EMAIL}"     --armor     --output ${GENERATED_SIGNATURE}     --sign ${GENERATED_PAYLOAD}
   60  cat "${GENERATED_SIGNATURE}"
   61  gcloud beta container binauthz attestations create     --artifact-url="${IMAGE_PATH}@${IMAGE_DIGEST}"     --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}"     --signature-file=${GENERATED_SIGNATURE}     --public-key-id="${PGP_FINGERPRINT}"
   62  gcloud beta container binauthz attestations list     --attestor="projects/${PROJECT_ID}/attestors/${ATTESTOR}"
   63  echo "projects/${PROJECT_ID}/attestors/${ATTESTOR}" # Copy this output to your copy/paste buffer
   64  gcloud beta container binauthz policy export > policy_modified_3.yaml
   65  cat policy_modified_3.yaml 
   66  IMAGE_PATH="gcr.io/${PROJECT_ID}/nginx"
   67  IMAGE_DIGEST="$(gcloud container images list-tags --format='get(digest)' $IMAGE_PATH | head -1)"
   68  cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: "${IMAGE_PATH}@${IMAGE_DIGEST}"
    ports:
    - containerPort: 80
EOF

   69  cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-alpha
  annotations:
    alpha.image-policy.k8s.io/break-glass: "true"
spec:
  containers:
  - name: nginx
    image: "nginx:latest"
    ports:
    - containerPort: 80
EOF

   70  kubectl get pods
   71  ./delete.sh -c my-cluster-1
   72  cat delete.sh 
   73  gcloud container images delete "${IMAGE_PATH}@${IMAGE_DIGEST}" --force-delete-tags
   74  gcloud --project="${PROJECT_ID}"     beta container binauthz attestors delete "${ATTESTOR}"
   75  curl -X DELETE     -H "Authorization: Bearer $(gcloud auth print-access-token)"     "https://containeranalysis.googleapis.com/v1beta1/projects/${PROJECT_ID}/notes/${NOTE_ID}"
   76  history 
student_01_03aeec6f7a0c@cloudshell:~/gke-binary-auth-demo (qwiklabs-gcp-04-e98db2f05087)$ 
```

