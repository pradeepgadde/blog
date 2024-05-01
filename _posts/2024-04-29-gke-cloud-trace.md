---

layout: single
title:  "View application latency with Cloud Trace"
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
# View application latency with Cloud Trace 

## View application latency with Cloud Trace 

- Deploy a sample application to a Google Kubernetes Engine (GKE) cluster.
- Create a trace by sending an HTTP request to the sample application.
- Use the Cloud Trace interface to view the latency information of the trace you created.

## Download and deploy your application

The script setup.sh configures three services of the application using a pre-built image. The workloads are named cloud-trace-demo-a,  cloud-trace-demo-b, and cloud-trace-demo-c. The setup script waits for  all resources to be provisioned, so the configuration might take several minutes to complete.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-2956453c7dee.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ git clone https://github.com/GoogleCloudPlatform/python-docs-samples.git
Cloning into 'python-docs-samples'...
remote: Enumerating objects: 111002, done.
remote: Counting objects: 100% (655/655), done.
remote: Compressing objects: 100% (370/370), done.
remote: Total 111002 (delta 330), reused 521 (delta 263), pack-reused 110347
Receiving objects: 100% (111002/111002), 240.00 MiB | 13.41 MiB/s, done.
Resolving deltas: 100% (65338/65338), done.
Updating files: 100% (5027/5027), done.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ gcloud services enable container.googleapis.com
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ ZONE=us-east1-d
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ gcloud container clusters create cloud-trace-demo \
   --zone $ZONE
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster cloud-trace-demo in us-east1-d... Cluster is being health-checked (master is healthy)...done.                                
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-2956453c7dee/zones/us-east1-d/clusters/cloud-trace-demo].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-d/cloud-trace-demo?project=qwiklabs-gcp-03-2956453c7dee
kubeconfig entry generated for cloud-trace-demo.
NAME: cloud-trace-demo
LOCATION: us-east1-d
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.148.81.109
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 3
STATUS: RUNNING
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ gcloud container clusters get-credentials cloud-trace-demo --zone $ZONE
Fetching cluster endpoint and auth data.
kubeconfig entry generated for cloud-trace-demo.
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ kubectl get nodes
NAME                                              STATUS   ROLES    AGE   VERSION
gke-cloud-trace-demo-default-pool-5f7c16a3-lj05   Ready    <none>   88s   v1.28.7-gke.1026000
gke-cloud-trace-demo-default-pool-5f7c16a3-lzvg   Ready    <none>   88s   v1.28.7-gke.1026000
gke-cloud-trace-demo-default-pool-5f7c16a3-qm7h   Ready    <none>   88s   v1.28.7-gke.1026000
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ kubectl get nodes -o wide
NAME                                              STATUS   ROLES    AGE   VERSION               INTERNAL-IP   EXTERNAL-IP     OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-cloud-trace-demo-default-pool-5f7c16a3-lj05   Ready    <none>   95s   v1.28.7-gke.1026000   10.142.0.3    35.237.53.38    Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-cloud-trace-demo-default-pool-5f7c16a3-lzvg   Ready    <none>   95s   v1.28.7-gke.1026000   10.142.0.5    35.229.92.193   Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
gke-cloud-trace-demo-default-pool-5f7c16a3-qm7h   Ready    <none>   95s   v1.28.7-gke.1026000   10.142.0.4    35.185.81.252   Container-Optimized OS from Google   6.1.58+          containerd://1.7.10
student_01_3c1bb184aaad@cloudshell:~ (qwiklabs-gcp-03-2956453c7dee)$ cd python-docs-samples/trace/cloud-trace-demo-app-opentelemetry && ./setup.sh

deployment.apps/cloud-trace-demo-a created
service/cloud-trace-demo-a created
deployment.apps/cloud-trace-demo-b created
service/cloud-trace-demo-b created
deployment.apps/cloud-trace-demo-c created
service/cloud-trace-demo-c created

Wait for load balancer initialization complete.......
Completed. You can access the demo at http://34.138.96.57/
~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry
student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ cat setup.sh 
#!/bin/bash
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -o errexit  # exit on error
SCRIPT_DIR=$(realpath $(dirname "$0"))
pushd $SCRIPT_DIR > /dev/null

echo ################## Set up cloud trace demo application ###########################
kubectl apply -f app/cloud-trace-demo.yaml

echo ""
echo -n "Wait for load balancer initialization complete."
for run in {1..20}
do
  sleep 5
  endpoint=`kubectl get svc cloud-trace-demo-a -ojsonpath='{.status.loadBalancer.ingress[0].ip}'`
  if [[ "$endpoint" != "" ]]; then
    break
  fi
  echo -n "."
done

echo ""
if [ -n "$endpoint" ]; then
  echo "Completed. You can access the demo at http://${endpoint}/"
else
  echo "There is a problem with the setup. Cannot determine the endpoint."
fi
popd
student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ 
```

## Create a trace

1. To create a trace by sending a curl request to cloud-trace-demo-a, use the following command:

   ```sh
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ curl $(kubectl get svc -o=jsonpath='{.items[?(@.metadata.name=="cloud-trace-demo-a")].status.loadBalancer.ingress[0].ip}')
   
   Hello, I am service A
   And I am service B
   Hello, I am service Cstudent_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ 
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ 
   ```

   ```py
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ cat app/app.py 
   # Copyright 2020 Google LLC
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
   
   """
   A sample app demonstrating CloudTraceSpanExporter
   """
   
   import os
   import random
   import time
   
   import flask
   
   # [START trace_demo_imports]
   from opentelemetry import trace
   from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
   from opentelemetry.instrumentation.flask import FlaskInstrumentor
   from opentelemetry.instrumentation.requests import RequestsInstrumentor
   from opentelemetry.propagate import set_global_textmap
   from opentelemetry.propagators.cloud_trace_propagator import CloudTraceFormatPropagator
   from opentelemetry.sdk.trace import TracerProvider
   from opentelemetry.sdk.trace.export import BatchSpanProcessor
   
   # [END trace_demo_imports]
   import requests
   
   
   # [START trace_demo_create_exporter]
   def configure_exporter(exporter):
       """Configures OpenTelemetry context propagation to use Cloud Trace context
   
       Args:
           exporter: exporter instance to be configured in the OpenTelemetry tracer provider
       """
       set_global_textmap(CloudTraceFormatPropagator())
       tracer_provider = TracerProvider()
       tracer_provider.add_span_processor(BatchSpanProcessor(exporter))
       trace.set_tracer_provider(tracer_provider)
   
   
   configure_exporter(CloudTraceSpanExporter())
   tracer = trace.get_tracer(__name__)
   # [END trace_demo_create_exporter]
   
   
   # [START trace_demo_middleware]
   app = flask.Flask(__name__)
   FlaskInstrumentor().instrument_app(app)
   RequestsInstrumentor().instrument()
   # [END trace_demo_middleware]
   
   
   @app.route("/")
   def template_test() -> str or (str, int):
       # Sleep for a random time to imitate a random processing time
       time.sleep(random.uniform(0, 0.5))
   
       # If there is an endpoint, send keyword to next service.
       # Return received input with the keyword
       keyword = os.getenv("KEYWORD")
       endpoint = os.getenv("ENDPOINT")
       # [START trace_context_header]
       if endpoint is not None and endpoint != "":
           data = {"body": keyword}
           response = requests.get(
               endpoint,
               params=data,
           )
           return keyword + "\n" + response.text
       else:
           return keyword, 200
   
   
   # [END trace_context_header]
   
   
   if __name__ == "__main__":
       port = os.getenv("PORT")
       app.run(debug=True, host="0.0.0.0", port=port)
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ ls
   app  example-trace.png  README.md  setup.sh
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ 
   ```

   ```yaml
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ cat app/cloud-trace-demo.yaml 
   # Copyright 2022 Google LLC
   #
   # Licensed under the Apache License, Version 2.0 (the "License");
   # you may not use this file except in compliance with the License.
   # You may obtain a copy of the License at
   #
   #      https://www.apache.org/licenses/LICENSE-2.0
   #
   # Unless required by applicable law or agreed to in writing, software
   # distributed under the License is distributed on an "AS IS" BASIS,
   # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   # See the License for the specific language governing permissions and
   # limitations under the License.
   
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cloud-trace-demo-a
     labels:
       app: cloud-trace-demo-app-a
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: cloud-trace-demo-app-a
     template:
       metadata:
         name: cloud-trace-demo-a
         labels:
           app: cloud-trace-demo-app-a
       spec:
         containers:
         - name: cloud-trace-demo-container
           image: gcr.io/google_samples/cloud-trace-demo-opentelemetry:latest
           imagePullPolicy: "Always"
           command:
           - python
           args:
           - app.py
           ports:
           - containerPort: 8080
           env:
           - name: PORT
             value: "8080"
           - name: KEYWORD
             value: "Hello, I am service A"
           - name: ENDPOINT
             value: "http://cloud-trace-demo-b:8090"
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: cloud-trace-demo-a
   spec:
     selector:
       app: cloud-trace-demo-app-a
     ports:
     - protocol: TCP
       port: 80
       targetPort: 8080
     type: LoadBalancer
   
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cloud-trace-demo-b
     labels:
       app: cloud-trace-demo-app-b
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: cloud-trace-demo-app-b
     template:
       metadata:
         name: cloud-trace-demo-b
         labels:
           app: cloud-trace-demo-app-b
       spec:
         containers:
         - name: cloud-trace-demo-container
           image: gcr.io/google_samples/cloud-trace-demo-opentelemetry:latest
           imagePullPolicy: "Always"
           command:
           - python
           args:
           - app.py
           ports:
           - containerPort: 8090
           env:
           - name: PORT
             value: "8090"
           - name: KEYWORD
             value: "And I am service B"
           - name: ENDPOINT
             value: "http://cloud-trace-demo-c:8090"
   
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: cloud-trace-demo-b
   spec:
     selector:
       app: cloud-trace-demo-app-b
     ports:
     - protocol: TCP
       port: 8090
       targetPort: 8090
     type: ClusterIP
   
   ---
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: cloud-trace-demo-c
     labels:
       app: cloud-trace-demo-app-c
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: cloud-trace-demo-app-c
     template:
       metadata:
         name: cloud-trace-demo-c
         labels:
           app: cloud-trace-demo-app-c
       spec:
         containers:
         - name: cloud-trace-demo-container
           image: gcr.io/google_samples/cloud-trace-demo-opentelemetry:latest
           imagePullPolicy: "Always"
           command:
           - python
           args:
           - app.py
           ports:
           - containerPort: 8090
           env:
           - name: PORT
             value: "8090"
           - name: KEYWORD
             value: "Hello, I am service C"
   
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: cloud-trace-demo-c
   spec:
     selector:
       app: cloud-trace-demo-app-c
     ports:
     - protocol: TCP
       port: 8090
       targetPort: 8090
     type: ClusterIP
   student_01_3c1bb184aaad@cloudshell:~/python-docs-samples/trace/cloud-trace-demo-app-opentelemetry (qwiklabs-gcp-03-2956453c7dee)$ 
   ```

   

### View the trace data

1. In the Google Cloud console, select Cloud Trace.

The Overview window is the default view in Trace. This window  displays latency data and summary information, including an analysis  report. If you created a new project, the most interesting pane of the  Overview window is the pane labeled Recent traces:

Recent traces pane that displays the most recent traces and their  latency. This pane lists the most recent traces and their latency. To  view the details of a trace, click its link.

1. In the Trace navigation pane, click list Trace explorer:

Trace explorer window for the quickstart.