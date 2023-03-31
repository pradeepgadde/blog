---

layout: single
title:  "Managing Deployments Using Kubernetes Engine"
date:   2023-03-30 15:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Managing Deployments Using Kubernetes Engine

Dev Ops practices will regularly make use of multiple deployments to  manage application deployment scenarios such as "Continuous deployment", "Blue-Green deployments", "Canary deployments" and more. This lab is to provide practice in scaling and managing containers so you can  accomplish these common scenarios where multiple heterogeneous  deployments are being used.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-3c5a2417b6e4.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-03-3c5a2417b6e4)$ gcloud config set compute/zone us-east1-b
Updated property [compute/zone].
student_01_c41e1c322859@cloudshell:~ (qwiklabs-gcp-03-3c5a2417b6e4)$ gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/fsmonitor-watchman.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/post-update.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/HEAD...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/config...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/applypatch-msg.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/pre-applypatch.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/description...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/commit-msg.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/pre-commit.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/pre-push.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/pre-rebase.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/pre-receive.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/prepare-commit-msg.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/hooks/update.sample...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/info/exclude...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/logs/HEAD...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/logs/refs/heads/master...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/02/1045ed5654fe659c68ff1e227d097e96218ad9...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/03/6a7cccf5377d5da982d3570b686dd46dc46ecd...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/logs/refs/remotes/origin/HEAD...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/00/3cec4f1b60af35c100072af6fb3c08661b7c1a...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/index...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/06/f7eccdd621cad86710d2f11d36823e68324600...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/03/69e9021e870d8e0a3f901ad12430c313e64170...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/09/2b7773dbc7c3a41355270b196cd8fc0c19e7f2...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/09/80232e617219a67ad57da7472fc643aff6def6...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/0b/83cec4e65254cb6315c772f97e1fd94d3ab214...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/12/516ddb42a309650e7aee43adc9e8f491470918...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/16/6fcd6095cbf19c57198bc9fc9ce0fbaffb43f9...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/0c/287d4607de60e405d354d7201b25d2a3f492c8...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/19/228bd6bf90d004de67c09a2bb86fbd92d9e9cc...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/1f/e779bcf156c392627afccc44922d506c06f315...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/20/4bb4e9651f407dfd81a58f715e95d49b84cfba...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/21/89a47794fd5b592aaeac939a66a85917483a31...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/24/998705e44deed51c09a7c55e218ff074b6c34a...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/22/36a0da90acdf6182fcbb6ac076bd2da32ff000...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/25/535ff347ee434f2da7851e7501af50bdba0a2f...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/26/8167d020044ae3f91c90aa1da611b81bc311aa...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/27/2d355afb62ef37c1f4c7508e8216bc804666d5...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/2d/69c3d3d3aeceb9ca4c47ff28d64d8aad176889...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/32/da1368260cd9f24e6ea41364d8e2a934d51361...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/2e/34b7d353738c0cce66343dd5cc0670746fc3e8...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/46/499de05a9e3ce017e7e2ca96e2f255a38e0108...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/47/9308e5f5254f64cdd1f1b7609e8d5768b80aec...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/37/23ed58d9f123bed49053d66a8123b005416c6d...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/34/04c8d5d0789596d5eaf3a7f97f45028e27f2e0...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/48/19aa58ec6e80bdbb91923cfdfe1ecba32a6137...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/4e/4bc74c6e3ecaa71ef688b72ef4a003453e106e...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/5c/63d3230677e7fe2b09f9c0ac7ef6265fd209e1...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/59/6b70ca0fd7318ed0f374d8c33cb50222328dda...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/5c/df68701c0c6b97f537c592e3f2df9b808f9db3...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/5c/d15de4dfe4cf2836afc2f3902d566518119cc6...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/5d/11d54064a7d153ad33babc565afc3714664f30...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/61/03835f632ef57b9eb08d65c5d3d6207d02b425...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/61/f21ac1117df93cbc2650ce9e2345aab815d13c...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/63/0bb766ee2c1812147f785ffdca3c94bf56f5ac...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/63/6133b0f8382be8742b3b18dcc78c43d49e1132...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/67/c4bebfb5f340d55dcebdec6c7c7743462695d2...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/67/ffa615c795003ba4b3a1d693ce4c7ce9f31a27...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/6b/5ff4a34dbdb1dc1f678475483209b2545a81f6...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/6e/9c67d5633eff9ca749bca2257e1e1fe2548698...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/6f/5f3d1bf673c822bdccf253c2bf7588c0ff2ed8...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/71/deb4e418f3f3223790548670bdfd37534bf666...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/7b/d2bd68f4d3d75aa9e185fa9f97f151fda85340...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/85/895faefec94252ce297ea50feb2e9505e5fe69...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/7f/8a88f59f9adb6db90db2245fe96a6b7e4d4b7c...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/85/ee013323592fbeafb837ceb9270172647f2b8c...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/86/64c23a158798578e5d42440016a68ac3b04a07...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/86/f4a67742560f14fdcececce20a058b75148402...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/89/b3769bbb3b5fb565178ccd66d03e69e41038ee...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/8c/47471efb13225d4249da0e7379b993c9689d54...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/8c/6e788b4148e07f27083afbed6a12126379644e...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/8d/ada3edaf50dbc082c9a125058f25def75e625a...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/8e/fcd3235d9185a67cec49232cccb14cdc8384e2...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/8f/770d40965afdb77c27d86e4fbfd2d70853932f...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/99/4e01454cc6cd676b18afca07337b7bf9b7b7c2...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/92/a517b2445fc96c43b8cf86097648d187149d7d...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/9e/b85a2e9ebcef0fb388b8f7c50602d9b3ef3419...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/a2/a903fd8de5d98b83765bcceeb1c50ca358f1ac...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/a6/b224cde2907c2bab5764b654c543c848d44c27...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/af/96521650dd9834e3341b5e6cebc821d85a1716...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/af/4e6d1c65ea37cd37a641b08cfcd3022f69f0c9...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/b0/b70547b933cedf26075050c2e35df22284cbb0...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/b0/cb83da75e86c1b39c710f68be55ffa569e0413...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/b0/f0dd5287d00b32b10c2bb99e3eaee147705af9...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/b7/6d51b4ac67b6fcb44b498487da6c83751699bb...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/be/0c8183fe7e154564b349b09cafb34c2a06f75c...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/be/6df2a25a8edf6a30dfe763ea3aec7339f465ce...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/c3/19306fa1a460c032eae80c12f3be1ad86c4af3...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/c3/3e8151f1b9c3737499778e80348cb3fa2f6d30...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/cc/51815c55c945a27c34bfb87c180adaa1565bcd...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/cc/080e5c11d167b0f16b5bb30ced56c062e5c5b7...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/ce/9c4053ec356ec27e99542c46438b3f65215f4e...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/ce/aa6726c2c86dcd7fa9d0b579c2a68ceb59976a...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/c5/13f4a612a10f66ba5c3c98b3eb1275288f398b...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/d1/7e743fecc5d6625a31f76811fafdfbd698e3f0...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/da/746e8bbc11364dc5b0eb4a51c3b147eb0a531f...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/db/bebb09e66e26b1e16a1aa65c34f2f495896c56...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/e3/315a105758fb7407a5a834f0dfb06b4f95ebe4...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/e7/81ac28725ce17c35f39bc0b6b49f1a7f3c57e8...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/e9/f25feb4bf41f6e54a5b9137bd34952f8f2f145...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/eb/2246c215774a74d18bb2b31a70591d8c6c86b3...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/ed/0a2e71cfb2fce07f8afb6420b88a639c87630a...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/ef/4190ef2189a7e354afefeb7598cabbe39e50a3...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/f1/f2a4c615439d9d3135787ba4972f70fd68d9fd...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/f2/3eb16fdf193af369aa637db31324d8ae648eb6...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/f6/7797c823fe5726d65f96b83f03783d1001a5b4...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/f8/751d062d4c0fa4361d53c7c2301cfa081c0c56...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/objects/fd/c29c8470849e75061af1264622dbf12a2e3f1f...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/packed-refs...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/refs/heads/master...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/.git/refs/remotes/origin/HEAD...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/CONTRIBUTING.md...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/LICENSE...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/README.md...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/cleanup.sh...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/cleanup.sh...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/deployments/frontend.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/deployments/auth.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/deployments/hello-canary.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/deployments/hello-green.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/deployments/hello.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/nginx/proxy.conf...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/nginx/frontend.conf...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/pods/healthy-monolith.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/pods/monolith.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/pods/secure-monolith.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/auth.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/frontend.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/hello-blue.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/hello-green.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/monolith.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/tls/ca-key.pem...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/services/hello.yaml...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/tls/ca.pem...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/tls/cert.pem...
Copying gs://spls/gsp053/orchestrate-with-kubernetes/kubernetes/tls/key.pem...
- [137/137 files][172.8 KiB/172.8 KiB] 100% Done
Operation completed over 137 objects/172.8 KiB.
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ gcloud container clusters create bootcamp \
  --machine-type e2-small \
  --num-nodes 3 \
  --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Default change: During creation of nodepools or autoscaling configuration changes for cluster versions greater than 1.24.1-gke.800 a default location policy is applied. For Spot and PVM it defaults to ANY, and for all other VM kinds a BALANCED policy is used. To change the default values use the `--location-policy` flag.
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster bootcamp in us-east1-b... Cluster is being health-checked (master is healthy)...done.     
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-3c5a2417b6e4/zones/us-east1-b/clusters/bootcamp].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-b/bootcamp?project=qwiklabs-gcp-03-3c5a2417b6e4
kubeconfig entry generated for bootcamp.
NAME: bootcamp
LOCATION: us-east1-b
MASTER_VERSION: 1.24.9-gke.3200
MASTER_IP: 34.23.126.117
MACHINE_TYPE: e2-small
NODE_VERSION: 1.24.9-gke.3200
NUM_NODES: 3
STATUS: RUNNING
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl explain deployment.metadata.name
KIND:     Deployment
VERSION:  apps/v1

FIELD:    name <string>

DESCRIPTION:
     Name must be unique within a namespace. Is required when creating
     resources, although some resources may allow a client to request the
     generation of an appropriate name automatically. Name is primarily intended
     for creation idempotence and configuration definition. Cannot be updated.
     More info: http://kubernetes.io/docs/user-guide/identifiers#names
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat deployments/auth.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
        track: stable
    spec:
      containers:
        - name: auth
          image: "kelseyhightower/auth:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f deployments/auth.yaml
deployment.apps/auth created

student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
auth   0/1     1            0           8s
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/auth-5c65b6d58-chjdr   1/1     Running   0          23s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.120.0.1   <none>        443/TCP   11m

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auth   1/1     1            1           23s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/auth-5c65b6d58   1         1         1       23s
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```




```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat deployments/hello.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 1.0.0
    spec:
      containers:
        - name: hello
          image: "kelseyhightower/hello:1.0.0"
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: "10Mi"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat services/hello.yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  selector:
    app: "hello"
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```


```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f services/auth.yaml
service/auth created
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f deployments/hello.yaml
deployment.apps/hello created
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f services/hello.yaml
service/hello created
```

```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat deployments/frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        track: stable
    spec:
      containers:
        - name: nginx
          image: "nginx:1.9.14"
          lifecycle:
            preStop:
              exec:
                command: ["/usr/sbin/nginx","-s","quit"]
          volumeMounts:
            - name: "nginx-frontend-conf"
              mountPath: "/etc/nginx/conf.d"
            - name: "tls-certs"
              mountPath: "/etc/tls"
      volumes:
        - name: "tls-certs"
          secret:
            secretName: "tls-certs"
        - name: "nginx-frontend-conf"
          configMap:
            name: "nginx-frontend-conf"
            items:
              - key: "frontend.conf"
                path: "frontend.conf"
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat services/frontend.yaml
kind: Service
apiVersion: v1
metadata:
  name: "frontend"
spec:
  selector:
    app: "frontend"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
  type: LoadBalancer
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```

```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
secret/tls-certs created
configmap/nginx-frontend-conf created
deployment.apps/frontend created
service/frontend created
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get services frontend
NAME       TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)         AGE
frontend   LoadBalancer   10.120.6.59   35.196.141.70   443:31046/TCP   71s
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://35.196.141.70
{"message":"Hello"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
{"message":"Hello"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl explain deployment.spec.replicas
KIND:     Deployment
VERSION:  apps/v1

FIELD:    replicas <integer>

DESCRIPTION:
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl scale deployment hello --replicas=5
deployment.apps/hello scaled
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get pods | grep hello- | wc -l
5
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl scale deployment hello --replicas=3
deployment.apps/hello scaled
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get pods | grep hello- | wc -l
3
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl edit deploy hello
deployment.apps/hello edited
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get rs
NAME                 DESIRED   CURRENT   READY   AGE
auth-5c65b6d58       1         1         1       7m34s
frontend-886c96b4d   1         1         1       4m41s
hello-7c575694fc     3         3         3       6m20s
hello-8654fb85d      1         1         0       5s
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout pause deployment/hello
deployment.apps/hello paused
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout status deployment/hello
deployment "hello" successfully rolled out
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
auth-5c65b6d58-chjdr            kelseyhightower/auth:1.0.0
frontend-886c96b4d-xzl7g                nginx:1.9.14
hello-8654fb85d-4p5j9           kelseyhightower/hello:2.0.0
hello-8654fb85d-l25hx           kelseyhightower/hello:2.0.0
hello-8654fb85d-qx56r           kelseyhightower/hello:2.0.0
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout resume deployment/hello
deployment.apps/hello resumed
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout status deployment/hello
deployment "hello" successfully rolled out
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout undo deployment/hello
deployment.apps/hello rolled back
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl rollout history deployment/hello
deployment.apps/hello
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
auth-5c65b6d58-chjdr            kelseyhightower/auth:1.0.0
frontend-886c96b4d-xzl7g                nginx:1.9.14
hello-7c575694fc-7jsx5          kelseyhightower/hello:1.0.0
hello-7c575694fc-g746w          kelseyhightower/hello:1.0.0
hello-7c575694fc-tltm4          kelseyhightower/hello:1.0.0
hello-8654fb85d-l25hx           kelseyhightower/hello:2.0.0
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat deployments/hello-canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: canary
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f deployments/hello-canary.yaml
deployment.apps/hello-canary created

student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
auth           1/1     1            1           12m
frontend       1/1     1            1           9m40s
hello          3/3     3            3           11m
hello-canary   1/1     1            1           11s
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"2.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat services/hello-blue.yaml
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  selector:
    app: "hello"
    version: 1.0.0
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl apply -f services/hello-blue.yaml
Warning: resource services/hello is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
service/hello configured
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```

```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat deployments/hello-green.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: stable
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
          resources:
            limits:
              cpu: 0.2
              memory: 10Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /readiness
              port: 81
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl create -f deployments/hello-green.yaml
deployment.apps/hello-green created
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```yaml
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ cat services/hello-green.yaml
kind: Service
apiVersion: v1
metadata:
  name: hello
spec:
  selector:
    app: hello
    version: 2.0.0
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl apply -f services/hello-green.yaml
service/hello configured
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"2.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"2.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



```sh
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ kubectl apply -f services/hello-blue.yaml
service/hello configured
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$ curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
{"version":"1.0.0"}
student_01_c41e1c322859@cloudshell:~/orchestrate-with-kubernetes/kubernetes (qwiklabs-gcp-03-3c5a2417b6e4)$
```



