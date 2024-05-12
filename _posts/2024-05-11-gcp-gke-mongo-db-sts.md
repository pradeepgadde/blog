---

layout: single
title:  "Running a MongoDB Database in Kubernetes with StatefulSets"
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

# Running a MongoDB Database in Kubernetes with StatefulSets

## Overview

[Kubernetes](http://kubernetes.io/) is an  open source container orchestration tool that handles the complexities  of running containerized applications. You can run Kubernetes  applications with `Kubernetes Engine`—a Google Cloud  computing service that offers many different customizations and  integrations. In this lab, you will get some practical experience with  Kubernetes by learning how to set up a MongoDB database with a [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/). Running a stateful application (a database) on a stateless service  (container) may sound contradictory. However, after getting hands-on  practice with this lab you will quickly see how that's not the case. In  fact, by using a few open-source tools you will see how Kubernetes and  stateless services can go hand-in-hand.

- How to deploy a Kubernetes cluster, a headless service, and a StatefulSet.
- How to connect a Kubernetes cluster to a MongoDB replica set.
- How to scale MongoDB replica set instances up and down.
- How to clean up your environment and shutdown the above services.

## Set a compute zone

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-6046cefd6d77.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ gcloud config set compute/zone us-east4-c
WARNING: Property validation for compute/zone was skipped.
Updated property [compute/zone].
student_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## Create a new cluster

Now that our zone is set, we will create a new cluster of containers.

- Run the following command to instantiate a cluster named `hello-world`:

```sh
tudent_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ gcloud container clusters create hello-world --num-nodes=2
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster hello-world in us-east4-c...working.                                                       Creating cluster hello-world in us-east4-c... Cluster is being health-checked (master is healthy)...done.                                                                 
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-6046cefd6d77/zones/us-east4-c/clusters/hello-world].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east4-c/hello-world?project=qwiklabs-gcp-03-6046cefd6d77
kubeconfig entry generated for hello-world.
NAME: hello-world
LOCATION: us-east4-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 34.86.231.0
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 2
STATUS: RUNNING
student_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## Setting up

Now that we have our cluster up and running, it's time to integrate it with MongoDB. We will be using a [replica set](https://docs.mongodb.com/manual/replication/) so that our data is highly available and redundant—a must for running production applications.

To get set up, we need to do the following:

- Download the MongoDB [replica set/sidecar](https://github.com/thesandlord/mongo-k8s-sidecar.git) (or utility container in our cluster).
- Instantiate a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/).
- Instantiate a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services).
- Instantiate a [StatefulSet](http://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/).

```sh
student_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ gsutil -m cp -r gs://spls/gsp022/mongo-k8s-sidecar .
Copying gs://spls/gsp022/mongo-k8s-sidecar/.foreverignore...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/commit-msg.sample...      
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/config...                       
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/fsmonitor-watchman.sample...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/post-update.sample...     
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-applypatch.sample...  
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/applypatch-msg.sample...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-commit.sample...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/description...                  
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/HEAD...                         
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-merge-commit.sample...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-push.sample...        
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-rebase.sample...      
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/pre-receive.sample...     
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/prepare-commit-msg.sample...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/hooks/update.sample...          
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/index...                        
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/info/exclude...                 
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/logs/HEAD...                    
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/logs/refs/heads/master...       
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/logs/refs/remotes/origin/HEAD...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/objects/pack/pack-ba695c54caf7d38d09fa01b0e54d8808cd04f6a8.idx...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/packed-refs...                  
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/objects/pack/pack-ba695c54caf7d38d09fa01b0e54d8808cd04f6a8.pack...
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/refs/heads/master...            
Copying gs://spls/gsp022/mongo-k8s-sidecar/.git/refs/remotes/origin/HEAD...     
Copying gs://spls/gsp022/mongo-k8s-sidecar/.gitignore...                        
Copying gs://spls/gsp022/mongo-k8s-sidecar/Dockerfile...                        
Copying gs://spls/gsp022/mongo-k8s-sidecar/LICENSE...                           
Copying gs://spls/gsp022/mongo-k8s-sidecar/README.md...                         
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/README.md...                 
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/README.md...     
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/Makefile...                  
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/azure_hdd.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/azure_ssd.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/googlecloud_hdd.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/googlecloud_ssd.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/StatefulSet/mongo-statefulset.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/README.md...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/config...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/config-host...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/images/figure1.png...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/images/figure2.png...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-rc-rbd-2.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-rc-rbd-1.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/images/figure3.png...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-rc-rbd-3.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-svc-1.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-svc-2.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-svc-3.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/ceph_rbd_pod_examples/mongo-svc.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/emptydir_pod_examples/mongo-rc-emptydir-1.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/emptydir_pod_examples/mongo-rc-emptydir-2.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/emptydir_pod_examples/mongo-rc-emptydir-3.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/emptydir_pod_examples/mongo-rc-emptydir-4.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/mongo-controller-flocker-template.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/mongo-controller-template.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/example/mongo-service-template.yaml...
Copying gs://spls/gsp022/mongo-k8s-sidecar/package.json...                      
Copying gs://spls/gsp022/mongo-k8s-sidecar/src/index.js...                      
Copying gs://spls/gsp022/mongo-k8s-sidecar/src/lib/config.js...                 
Copying gs://spls/gsp022/mongo-k8s-sidecar/src/lib/k8s.js...                    
Copying gs://spls/gsp022/mongo-k8s-sidecar/src/lib/mongo.js...                  
Copying gs://spls/gsp022/mongo-k8s-sidecar/src/lib/worker.js...                 
| [64/64 files][775.3 KiB/775.3 KiB] 100% Done                                  
Operation completed over 64 objects/775.3 KiB.                                   
student_01_2232ec055a17@cloudshell:~ (qwiklabs-gcp-03-6046cefd6d77)$ 
```

### Create the StorageClass

A `StorageClass` tells Kubernetes what kind of storage you want to use for database nodes. On the Google Cloud, you have a couple of [storage choices](https://cloud.google.com/compute/docs/disks/): SSDs and hard disks.

If you take a look inside the `StatefulSet` directory (you can do this by running the `ls` command), you will see SSD and HDD configuration files for both Azure and Google Cloud.



```yaml
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ cat googlecloud_ssd.yaml
#       Copyright 2017, Google, Inc.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http:#www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$
```

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get sc
NAME                     PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
premium-rwo              pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   3m9s
standard                 kubernetes.io/gce-pd    Delete          Immediate              true                   3m7s
standard-rwo (default)   pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   3m8s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

```sh
storageclass.storage.k8s.io/fast created
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get sc
NAME                     PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
fast                     kubernetes.io/gce-pd    Delete          Immediate              false                  3s
premium-rwo              pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   3m49s
standard                 kubernetes.io/gce-pd    Delete          Immediate              true                   3m47s
standard-rwo (default)   pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   3m48s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

Now that our StorageClass is configured, our StatefulSet can now request a volume that will automatically be created.

## Deploying the Headless Service and StatefulSet

### Find and inspect the files

1. Before we dive into what headless service and StatefulSets are, let's open up the configuration file (`mongo-statefulset.yaml`) where they are both housed in:

```yaml
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ cat mongo-statefulset.yaml 
#       Copyright 2016, Google, Inc.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http:#www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
---
apiVersion: v1
kind: Service
metadata:
  name: mongo
  labels:
    name: mongo
spec:
  ports:
  - port: 27017
    targetPort: 27017
  clusterIP: None
  selector:
    role: mongo
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  selector:
    matchLabels:
      role: mongo
  template:
    metadata:
      labels:
        role: mongo
        environment: test
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: mongo
          image: mongo
          command:
            - mongod
            - "--replSet"
            - rs0
            - "--smallfiles"
            - "--noprealloc"
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-persistent-storage
              mountPath: /data/db
        - name: mongo-sidecar
          image: cvallance/mongo-k8s-sidecar
          env:
            - name: MONGO_SIDECAR_POD_LABELS
              value: "role=mongo,environment=test"
  volumeClaimTemplates:
  - metadata:
      name: mongo-persistent-storage
      annotations:
        volume.beta.kubernetes.io/storage-class: "fast"
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 50Gi
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

**Remove** the following flags from the file (lines 49 and 50):

- "--smallfiles"
- "--noprealloc"



#### Headless service: overview

The first section of `mongo-statefulset.yaml` refers to a `headless service`. In Kubernetes terms, a service describes policies or rules for  accessing specific pods. In brief, a headless service is one that  doesn't prescribe load balancing. When combined with StatefulSets, this  will give us individual DNSs to access our pods, and in turn a way to  connect to all of our MongoDB nodes individually. In the `yaml` file, you can make sure that the service is headless by verifying that the `clusterIP` field is set to `None`.

#### StatefulSet: overview

The StatefulSet configuration is the second section of `mongo-statefulset.yaml.` This is the bread and butter of the application: it's the workload that runs MongoDB and what orchestrates your Kubernetes resources.  Referencing the `yaml` file, we see that the first section  describes the StatefulSet object. Then, we move into the Metadata  section, where labels and the number of replicas are specified.

Next comes the pod spec. The `terminationGracePeriodSeconds` is used to gracefully shutdown the pod when you scale down the number  of replicas. Then the configurations for the two containers are shown.  The first one runs MongoDB with command line flags that configure the  replica set name. It also mounts the persistent storage volume to `/data/db`: the location where MongoDB saves its data. The second container runs the sidecar. This [sidecar container](https://github.com/cvallance/mongo-k8s-sidecar) will configure the MongoDB replica set automatically. As mentioned  earlier, a "sidecar" is a helper container that helps the main container run its jobs and tasks.

Finally, there is the `volumeClaimTemplates`. This is what talks to the StorageClass we created before to provision the volume. It provisions a 100 GB disk for each MongoDB replica.



#### Deploy Headless Service and the StatefulSet

Now that we have a basic understanding of what a headless service and StatefulSet are, let's go ahead and deploy them.

- Since the two are packaged in `mongo-statefulset.yaml`, we can run the following command to run both of them:

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl apply -f mongo-statefulset.yaml
service/mongo created
statefulset.apps/mongo created
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## Connect to the MongoDB Replica set

Now that we have a cluster running and our replica set deployed, let's go ahead and connect to it.

### Wait for the MongoDB replica set to be fully deployed

Kubernetes StatefulSets deploys each pod sequentially. It waits for  the MongoDB replica set member to fully boot up and create the backing  disk before starting the next member.

- Run the following command to view and confirm that all three members are up:

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get statefulset
NAME    READY   AGE
mongo   3/3     91s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

### Initiating and Viewing the MongoDB replica set

At this point, you should have three pods created in your cluster.  These correspond to the three nodes in your MongoDB replica set.

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   2/2     Running   0          104s
mongo-1   2/2     Running   0          71s
mongo-2   2/2     Running   0          35s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl exec -ti mongo-0 -- mongosh
Defaulted container "mongo" out of: mongo, mongo-sidecar
Current Mongosh Log ID: 6640c19d389514e48c2202d7
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.2.5
Using MongoDB:          7.0.9
Using Mongosh:          2.2.5

For mongosh info see: https://docs.mongodb.com/mongodb-shell/


To help improve our products, anonymous usage data is collected and sent to MongoDB periodically (https://www.mongodb.com/legal/privacy-policy).
You can opt-out by running the disableTelemetry() command.

------
   The server generated these startup warnings when booting
   2024-05-12T13:16:47.186+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2024-05-12T13:16:47.880+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2024-05-12T13:16:47.880+00:00: You are running this process as the root user, which is not recommended
   2024-05-12T13:16:47.880+00:00: This server is bound to localhost. Remote systems will be unable to connect to this server. Start the server with --bind_ip <address> to specify which IP addresses it should serve responses from, or with --bind_ip_all to bind to all interfaces. If this behavior is desired, start the server with --bind_ip 127.0.0.1 to disable this warning
   2024-05-12T13:16:47.880+00:00: vm.max_map_count is too low
------

test> rs.initiate()
{
  info2: 'no configuration specified. Using a default configuration for the set',
  me: 'localhost:27017',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1715519909, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('AAAAAAAAAAAAAAAAAAAAAAAAAAA=', 0),
      keyId: Long('0')
    }
  },
  operationTime: Timestamp({ t: 1715519909, i: 1 })
}
rs0 [direct: other] test> rs.conf()
{
  _id: 'rs0',
  version: 1,
  term: 1,
  members: [
    {
      _id: 0,
      host: 'localhost:27017',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    }
  ],
  protocolVersion: Long('1'),
  writeConcernMajorityJournalDefault: true,
  settings: {
    chainingAllowed: true,
    heartbeatIntervalMillis: 2000,
    heartbeatTimeoutSecs: 10,
    electionTimeoutMillis: 10000,
    catchUpTimeoutMillis: -1,
    catchUpTakeoverDelayMillis: 30000,
    getLastErrorModes: {},
    getLastErrorDefaults: { w: 1, wtimeout: 0 },
    replicaSetId: ObjectId('6640c1a5badc2d6a5ffd0650')
  }
}
rs0 [direct: secondary] test> exit
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## caling the MongoDB replica set

A big advantage of Kubernetes and StatefulSets is that you can scale  the number of MongoDB Replicas up and down with a single command!

1. To scale up the number of replica set members from 3 to 5, run this command:

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl scale --replicas=5 statefulset mongo
statefulset.apps/mongo scaled
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get pods
NAME      READY   STATUS              RESTARTS   AGE
mongo-0   2/2     Running             0          3m10s
mongo-1   2/2     Running             0          2m37s
mongo-2   2/2     Running             0          2m1s
mongo-3   0/2     ContainerCreating   0          7s

student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   2/2     Running   0          3m38s
mongo-1   2/2     Running   0          3m5s
mongo-2   2/2     Running   0          2m29s
mongo-3   2/2     Running   0          35s
mongo-4   2/2     Running   0          22s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl scale --replicas=3 statefulset mongo
statefulset.apps/mongo scaled
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
mongo-0   2/2     Running   0          4m11s
mongo-1   2/2     Running   0          3m38s
mongo-2   2/2     Running   0          3m2s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)
```

## Using the MongoDB replica set

Each pod in a StatefulSet backed by a Headless Service will have a stable DNS name. The template follows this format: `<pod-name>.<service-name>`

This means the DNS names for the MongoDB replica set are:

```sh
tudent_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.111.240.1   <none>        443/TCP     12m
mongo        ClusterIP   None           <none>        27017/TCP   4m47s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.111.240.1   <none>        443/TCP     13m
mongo        ClusterIP   None           <none>        27017/TCP   4m52s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl get ep
NAME         ENDPOINTS                                             AGE
kubernetes   10.150.0.2:443                                        13m
mongo        10.112.0.13:27017,10.112.1.7:27017,10.112.1.8:27017   5m22s
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## Clean up

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl delete statefulset mongo
statefulset.apps "mongo" deleted
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl delete svc mongo
service "mongo" deleted
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ kubectl delete pvc -l role=mongo
persistentvolumeclaim "mongo-persistent-storage-mongo-0" deleted
persistentvolumeclaim "mongo-persistent-storage-mongo-1" deleted
persistentvolumeclaim "mongo-persistent-storage-mongo-2" deleted
persistentvolumeclaim "mongo-persistent-storage-mongo-3" deleted
persistentvolumeclaim "mongo-persistent-storage-mongo-4" deleted
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ gcloud container clusters delete "hello-world"
The following clusters will be deleted.
 - [hello-world] in [us-east4-c]

Do you want to continue (Y/n)?  y

Deleting cluster hello-world...working                                                                                                                                              
Deleting cluster hello-world...done.                                                                                                                                                
Deleted [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-6046cefd6d77/zones/us-east4-c/clusters/hello-world].
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

## Summary

```sh
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ history 
    1  gcloud config set compute/zone us-east4-c
    2  gcloud container clusters create hello-world --num-nodes=2
    3  gsutil -m cp -r gs://spls/gsp022/mongo-k8s-sidecar .
    4  cd ./mongo-k8s-sidecar/example/StatefulSet/
    5  cat googlecloud_ssd.yaml
    6  kubectl get sc
    7  kubectl apply -f googlecloud_ssd.yaml
    8  kubectl get sc
    9  cat mongo-statefulset.yaml 
   10  vi mongo-statefulset.yaml
   11  kubectl apply -f mongo-statefulset.yaml
   12  kubectl get statefulset
   13  kubectl get pods
   14  kubectl get statefulset
   15  kubectl get pods
   16  kubectl exec -ti mongo-0 -- mongosh
   17  kubectl scale --replicas=5 statefulset mongo
   18  kubectl get pods
   19  kubectl scale --replicas=3 statefulset mongo
   20  kubectl get pods
   21  kubectl get svc
   22  ping mongo-0.mongo
   23  ping mongo-1.mongo
   24  kubectl get ep
   25  kubectl delete statefulset mongo
   26  kubectl delete svc mongo
   27  kubectl delete pvc -l role=mongo
   28  gcloud container clusters delete "hello-world"
   29  history 
student_01_2232ec055a17@cloudshell:~/mongo-k8s-sidecar/example/StatefulSet (qwiklabs-gcp-03-6046cefd6d77)$ 
```

