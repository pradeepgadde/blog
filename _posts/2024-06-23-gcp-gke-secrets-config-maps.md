---

layout: single
title:  "Working with Google Kubernetes Engine Secrets and ConfigMaps"
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
# Working with Google Kubernetes Engine Secrets and ConfigMaps

In this lab, you set up configuration information, both encrypted and unencrypted. Encrypted configuration information is stored as secrets.  Unencrypted configuration information is stored as ConfigMaps.

This approach avoids hard coding such information into code bases.  Credentials (like API keys) that belong in secrets should never travel  inside code repositories like GitHub (unless they are encrypted before  going in, but even then it is better to keep them separate).

- Create secrets by using the `kubectl` command and manifest files.
- Create ConfigMaps by using the `kubectl` command and manifest files.
- Consume secrets in containers by using environment variables or mounted volumes.
- Consume ConfigMaps in containers by using environment variables or mounted volumes.

## Work with secrets

In this task, you authenticate containers with Google Cloud in order  to access Google Cloud services. You set up a Cloud Pub/Sub topic and  subscription, try to access the Cloud Pub/Sub topic from a container  running in GKE, and see that the access request fails.

To properly access the pub/sub topic, you create a service account  with credentials, and pass those credentials through Kubernetes Secrets.

### Prepare a service account with no permissions

1. In the Google Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **IAM & Admin** > **Service accounts**.
2. Click **+ Create Service Account**.
3. In the **Service Account Name** text box, enter `no-permissions`.
4. Click **Create and Continue**.
5. Click **Continue** and then click **Done**.
6. Find the **no-permissions** service account in the list, and copy the email address associated with it to a text file for later use

### Create a GKE cluster

When you create this cluster you will specify the service account you created earlier. To illustrate the need for service accounts with  proper permissions, that service account has no permissions to any other Google Cloud services, and therefore will be unable to connect to the  Cloud Pub/Sub test application you will later deploy. You will fix this  later in the lab.

1. In Cloud Shell, type the following command to create environment  variables for the Google Cloud zone and cluster name that will be used  to create the cluster for this lab:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-205a82dafe3a.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ export my_zone=us-east1-d
export my_cluster=standard-cluster-1
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ source <(kubectl completion bash)
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ export my_service_account=no-permissions@qwiklabs-gcp-03-205a82dafe3a.iam.gserviceaccount.com
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ gcloud container clusters create $my_cluster \
  --num-nodes 2 --zone $my_zone \
  --service-account=$my_service_account
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster standard-cluster-1 in us-east1-d... Cluster is being health-checked (master is healthy)...done.                            
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-205a82dafe3a/zones/us-east1-d/clusters/standard-cluster-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-d/standard-cluster-1?project=qwiklabs-gcp-03-205a82dafe3a
kubeconfig entry generated for standard-cluster-1.
NAME: standard-cluster-1
LOCATION: us-east1-d
MASTER_VERSION: 1.29.4-gke.1043002
MASTER_IP: 35.237.196.180
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.29.4-gke.1043002
NUM_NODES: 2
STATUS: RUNNING
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ 
```



### Set up Cloud Pub/Sub and deploy an application to read from the topic

1. In Cloud Shell, type the following command to set the environment variables for the Pub/Sub components:

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ export my_pubsub_topic=echo
export my_pubsub_subscription=echo-read
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ gcloud pubsub topics create $my_pubsub_topic
gcloud pubsub subscriptions create $my_pubsub_subscription \
 --topic=$my_pubsub_topic
Created topic [projects/qwiklabs-gcp-03-205a82dafe3a/topics/echo].
Created subscription [projects/qwiklabs-gcp-03-205a82dafe3a/subscriptions/echo-read].
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ 
```



### **Deploy an application to read from Cloud Pub/Sub topics**

You create a deployment with a container that can read from Cloud  Pub/Sub topics. Since specific permissions are required to subscribe to, and read from, Cloud Pub/Sub topics this container needs to be provided with credentials in order to successfully connect to Cloud Pub/Sub.

The Deployment manifest file called  `pubsub.yaml` is provided for you.

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64994, done.
remote: Total 64994 (delta 0), reused 0 (delta 0), pack-reused 64994
Receiving objects: 100% (64994/64994), 697.09 MiB | 12.67 MiB/s, done.
Resolving deltas: 100% (41534/41534), done.
Updating files: 100% (12864/12864), done.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-03-205a82dafe3a)$ cd ~/ak8s/Secrets/
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```yaml
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat pubsub.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pubsub
spec:
  selector:
    matchLabels:
      app: pubsub
  template:
    metadata:
      labels:
        app: pubsub
    spec:
      containers:
      - name: subscriber
        image: gcr.io/google-samples/pubsub-sample:v1
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl apply -f pubsub.yaml
deployment.apps/pubsub created
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods -l app=pubsub
NAME                      READY   STATUS              RESTARTS   AGE
pubsub-69fdc4b8f8-bw2sh   0/1     ContainerCreating   0          5s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods -l app=pubsub
NAME                      READY   STATUS   RESTARTS   AGE
pubsub-69fdc4b8f8-bw2sh   0/1     Error    0          10s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods -l app=pubsub
NAME                      READY   STATUS   RESTARTS     AGE
pubsub-69fdc4b8f8-bw2sh   0/1     Error    1 (6s ago)   13s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

Notice the status of the Pod. It has an error and has restarted several times.

1. To inspect the logs from the Pod, execute the following command:

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl logs -l app=pubsub
    return api_call(*args)
  File "/usr/local/lib/python3.8/site-packages/google/gax/api_callable.py", line 376, in inner
    return a_func(*args, **kwargs)
  File "/usr/local/lib/python3.8/site-packages/google/gax/retry.py", line 125, in inner
    raise errors.RetryError(
google.gax.errors.RetryError: RetryError(Exception occurred in retry method that was not classified as transient, caused by <_InactiveRpcError of RPC that terminated with:
        status = StatusCode.PERMISSION_DENIED
        details = "User not authorized to perform this action."
        debug_error_string = "UNKNOWN:Error received from peer ipv4:142.251.107.95:443 {created_time:"2024-06-23T15:55:06.588265144+00:00", grpc_status:7, grpc_message:"User not authorized to perform this action."}"
>)
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

The error message displayed at the end of the log indicates that the  application doesn't have permissions to query the Cloud Pub/Sub service.

### Create service account credentials

You will now create a new service account and grant it access to the  pub/sub subscription that the test application is attempting to use.  Instead of changing the service account of the GKE cluster nodes, you  will generate a JSON key for the service account, and then securely pass the JSON key to the Pod via Kubernetes Secrets.

1. In the Google Cloud Console, on the **Navigation menu**, click **IAM & Admin** > **Service Accounts**.
2. Click **+ Create Service Account**.
3. In the **Service Account Name** text box, enter `pubsub-app` and then click **Create and Continue**.
4. In the **Select a role** drop-down list, select **Pub/Sub > Pub/Sub Subscriber**.
5. Confirm the role is listed, and then click **Continue** and then click **Done**.
6. In the Service Account overview screen, click the three dots on the right hand side of the `pubsub-app` service account, then select **Manage Keys**.
7. In the drop down, click **Add Key**, and then select **Create new key**.
8. Select **JSON** as the key type, and then click **Create**.
9. A JSON key file containing the credentials of the service account  will download to your computer. You can see the file in the download bar at the bottom of your screen. You will use this key file to configure  the sample application to authenticate to Cloud Pub/Sub API.
10. Click **Close**.
11. On your hard drive, locate the JSON key that you just downloaded and rename the file to `credentials.json`.

### Import credentials as a secret

1. In Cloud Shell, click the three dots (![more icon](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D)) in the Cloud Shell toolbar to display further options.
2. Click **Upload** and upload the `credentials.json` file from your local machine to the Cloud Shell VM and the click **Upload**.
3. In Cloud Shell, enter the following command to confirm that the file was uploaded:

You should see the credentials file that has been uploaded along with the lab files directory you cloned earlier.

1. To save the `credentials.json` key file to a Kubernetes Secret named `pubsub-key`  execute the following command:

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl create secret generic pubsub-key \
 --from-file=key.json=$HOME/credentials.json
secret/pubsub-key created
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ rm -rf ~/credentials.json
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

This command creates a secret named `pubsub-key` that has a `key.json` value containing the contents of the private key that you downloaded from the Google Cloud Console.

### Configure the application with the secret

You now update the deployment to include the following changes:

- Add a volume to the Pod specification. This volume contains the secret.
- The secrets volume is mounted in the application container.
- The `GOOGLE_APPLICATION_CREDENTIALS` environment variable is set to point to the key file in the secret volume mount.

The  `GOOGLE_APPLICATION_CREDENTIALS` environment variable is automatically recognized by Cloud Client Libraries, in this case, the Cloud Pub/Sub client for Python.

The updated Deployment file called `pubsub-secret.yaml` has been provided for you.

```yaml
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat pubsub-secret.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pubsub
spec:
  selector:
    matchLabels:
      app: pubsub
  template:
    metadata:
      labels:
        app: pubsub
    spec:
      volumes:
      - name: google-cloud-key
        secret:
          secretName: pubsub-key
      containers:
      - name: subscriber
        image: gcr.io/google-samples/pubsub-sample:v1
        volumeMounts:
        - name: google-cloud-key
          mountPath: /var/secrets/google
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/key.json
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl apply -f pubsub-secret.yaml
deployment.apps/pubsub configured
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods -l app=pubsub
NAME                      READY   STATUS    RESTARTS   AGE
pubsub-7bdb99d574-7pz65   1/1     Running   0          8s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

The Pod status should be *Running*. The status change might take several seconds to appear.

### Test receiving Cloud Pub/Sub messages

1. Now that you configured the application, publish a message to the Cloud Pub/Sub topic you created earlier in the lab:
2. Within a few seconds, the message should be picked up by the application and printed to the output stream.
   1. To inspect the logs from the deployed Pod, execute the following command:

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ gcloud pubsub topics publish $my_pubsub_topic --message="Hello, world!"
messageIds:
- '11577591262533931'
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl logs -l app=pubsub
Pulling messages from Pub/Sub subscription...
[2024-06-23 16:03:18.905453] Received message: ID=11577591262533931 Data=b'Hello, world!'
[2024-06-23 16:03:18.905556] Processing: 11577591262533931
[2024-06-23 16:03:21.908712] Processed: 11577591262533931
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```



## Work with ConfigMaps

ConfigMaps bind configuration files, command-line arguments,  environment variables, port numbers, and other configuration artifacts  to your Pods' containers and system components at runtime.

ConfigMaps enable you to separate your configurations from your Pods  and components. But ConfigMaps aren't encrypted, making them  inappropriate for credentials. This is the difference between secrets  and ConfigMaps: secrets are encrypted and are therefore better suited  for confidential or sensitive information such as credentials.

ConfigMaps are better suited for general configuration information such as port numbers.



### Use the kubectl command to create ConfigMaps

You use `kubectl` to create ConfigMaps by following the pattern `kubectl create configmap [NAME] [DATA]` and adding a flag for file (`--from-file`) or literal (`--from-literal`).

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl create configmap sample --from-literal=message=hello
configmap/sample created
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl describe configmaps sample
Name:         sample
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
message:
----
hello

BinaryData
====

Events:  <none>
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat sample2.properties 
message2=world
foo=bar
meaningOfLife=42
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl create configmap sample2 --from-file=sample2.properties
configmap/sample2 created
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl describe configmaps sample2
Name:         sample2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
sample2.properties:
----
message2=world
foo=bar
meaningOfLife=42


BinaryData
====

Events:  <none>
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```





### Use manifest files to create ConfigMaps

You can also use a YAML configuration file to create a ConfigMap. The `config-map-3.yaml` file contains a ConfigMap definition called `sample3`. You will use this ConfigMap later to demonstrate two different ways to expose the data inside a container.

```yaml
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat config-map-3.yaml 
apiVersion: v1
data:
  airspeed: africanOrEuropean
  meme: testAllTheThings
kind: ConfigMap
metadata:
  name: sample3
  namespace: default
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl apply -f config-map-3.yaml
configmap/sample3 created
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl describe configmaps sample3
Name:         sample3
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
airspeed:
----
africanOrEuropean
meme:
----
testAllTheThings

BinaryData
====

Events:  <none>
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

Now you have some non secret, unencrypted, configuration information  properly separated from your application and available to your cluster.  You've performed this task using ConfigMaps in three different ways to  demonstrate the various options, but in practice you typically pick one  method, most likely the YAML configuration file approach. Configuration  files provide a record of the values that you've stored so that you can  easily repeat the process in the future.

### Use environment variables to consume ConfigMaps in containers

In order to access ConfigMaps from inside Containers using  environment variables the Pod definition must be updated to include one  or more `configMapKeyRefs`.

The file `pubsub-configmap.yaml` is an updated version of the Cloud Pub/Sub demo Deployment that includes the following additional `env:` setting at the end of the file to import environmental variables from the ConfigMap into the container.

```yaml
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat pubsub-configmap.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pubsub
spec:
  selector:
    matchLabels:
      app: pubsub
  template:
    metadata:
      labels:
        app: pubsub
    spec:
      volumes:
      - name: google-cloud-key
        secret:
          secretName: pubsub-key
      containers:
      - name: subscriber
        image: gcr.io/google-samples/pubsub-sample:v1
        volumeMounts:
        - name: google-cloud-key
          mountPath: /var/secrets/google
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/key.json
        - name: INSIGHTS
          valueFrom:
            configMapKeyRef:
              name: sample3
              key: meme

student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl apply -f pubsub-configmap.yaml
deployment.apps/pubsub configured
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

Now your application has access to an environment variable called `INSIGHTS`, which has a value of `testAllTheThings`.

1. To verify that the environment variable has the correct value, you  must gain shell access to your Pod, which means that you need the name  of your Pod. To get the name of the Pod, execute the following the  command:

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
pubsub-78b7f66cb8-lhnl9   1/1     Running   0          33s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl exec -it pubsub-78b7f66cb8-lhnl9 -- sh
# printenv
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://34.118.224.1:443
HOSTNAME=pubsub-78b7f66cb8-lhnl9
PYTHON_PIP_VERSION=22.0.4
HOME=/root
GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568
GOOGLE_APPLICATION_CREDENTIALS=/var/secrets/google/key.json
PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/0d8570dc44796f4369b652222cf176b3db6ac70e/public/get-pip.py
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=34.118.224.1
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
LANG=C.UTF-8
PYTHON_VERSION=3.8.16
PYTHON_SETUPTOOLS_VERSION=57.5.0
INSIGHTS=testAllTheThings
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://34.118.224.1:443
KUBERNETES_SERVICE_HOST=34.118.224.1
PWD=/
PYTHON_GET_PIP_SHA256=96461deced5c2a487ddc65207ec5a9cffeca0d34e7af7ea1afc470ff0d746207
# exit
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```



### Use mounted volumes to consume ConfigMaps in containers

You can populate a volume with the ConfigMap data instead of (or in addition to) storing it in an environment variable.

In this Deployment the ConfigMap named `sample-3` that you created earlier in this task is also added as a volume called `config-3` in the Pod spec. The `config-3` volume is then mounted inside the container on the path `/etc/config`. The original method using Environment variables to import ConfigMaps is also configured.

The updated Deployment file called `pubsub-configmap2.yaml` has been provided for you.

```yaml
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ cat pubsub-configmap2.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pubsub
spec:
  selector:
    matchLabels:
      app: pubsub
  template:
    metadata:
      labels:
        app: pubsub
    spec:
      volumes:
      - name: google-cloud-key
        secret:
          secretName: pubsub-key
      - name: config-3
        configMap:
          name: sample3
      containers:
      - name: subscriber
        image: gcr.io/google-samples/pubsub-sample:v1
        volumeMounts:
        - name: google-cloud-key
          mountPath: /var/secrets/google
        - name: config-3
          mountPath: /etc/config
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/key.json
        - name: INSIGHTS
          valueFrom:
            configMapKeyRef:
              name: sample3
              key: meme
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl get pods
NAME                      READY   STATUS        RESTARTS   AGE
pubsub-78b7f66cb8-lhnl9   1/1     Terminating   0          2m34s
pubsub-86459c465-889lr    1/1     Running       0          8s
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ kubectl exec -it pubsub-86459c465-889lr -- sh
# cd /etc/config
# ls
airspeed  meme
# cat airspeed
africanOrEuropean# cat meme
testAllTheThings# exit
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

## History

```sh
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ history 
    1  export my_zone=us-east1-d
    2  export my_cluster=standard-cluster-1
    3  source <(kubectl completion bash)
    4  export my_service_account=no-permissions@qwiklabs-gcp-03-205a82dafe3a.iam.gserviceaccount.com
    5  gcloud container clusters create $my_cluster   --num-nodes 2 --zone $my_zone   --service-account=$my_service_account
    6  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    7  export my_pubsub_topic=echo
    8  export my_pubsub_subscription=echo-read
    9  gcloud pubsub topics create $my_pubsub_topic
   10  gcloud pubsub subscriptions create $my_pubsub_subscription  --topic=$my_pubsub_topic
   11  ls
   12  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
   13  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
   14  cd ~/ak8s/Secrets/
   15  cat pubsub.yaml 
   16  kubectl apply -f pubsub.yaml
   17  kubectl get pods -l app=pubsub
   18  kubectl logs -l app=pubsub
   19  ls ~/
   20  kubectl create secret generic pubsub-key  --from-file=key.json=$HOME/credentials.json
   21  rm -rf ~/credentials.json
   22  cat pubsub-secret.yaml 
   23  kubectl apply -f pubsub-secret.yaml
   24  kubectl get pods -l app=pubsub
   25  gcloud pubsub topics publish $my_pubsub_topic --message="Hello, world!"
   26  kubectl logs -l app=pubsub
   27  kubectl create configmap sample --from-literal=message=hello
   28  kubectl describe configmaps sample
   29  cat sample2.properties 
   30  kubectl create configmap sample2 --from-file=sample2.properties
   31  kubectl describe configmaps sample2
   32  cat config-map-3.yaml 
   33  kubectl apply -f config-map-3.yaml
   34  kubectl describe configmaps sample3
   35  cat pubsub-configmap
   36  cat pubsub-configmap.yaml 
   37  kubectl apply -f pubsub-configmap.yaml
   38  kubectl get pods
   39  kubectl exec -it pubsub-78b7f66cb8-lhnl9 -- sh
   40  cat pubsub-configmap2.yaml 
   41  kubectl apply -f pubsub-configmap2.yaml
   42  kubectl get pods
   43  kubectl exec -it pubsub-86459c465-889lr -- sh
   44  history 
student_01_4f5493923909@cloudshell:~/ak8s/Secrets (qwiklabs-gcp-03-205a82dafe3a)$ 
```

