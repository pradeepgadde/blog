---

layout: single
title:  "Configuring Pod Autoscaling and NodePools"
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
# Configuring Pod Autoscaling and NodePools

- Configure autoscaling and HorizontalPodAutoscaler
- Add a node pool and configure taints on the nodes for Pod anti-affinity
- Configure an exception for the node taint by adding a toleration to a Pod's manifest

## Connect to the lab GKE cluster and deploy a sample workload

In this task, you connect to the lab GKE cluster and create a deployment manifest for a set of Pods within the cluster.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-7ea9bf2c579b.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ export my_zone=us-central1-a
export my_cluster=standard-cluster-1
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ source <(kubectl completion bash)
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64994, done.
remote: Counting objects: 100% (131/131), done.
remote: Compressing objects: 100% (87/87), done.
remote: Total 64994 (delta 60), reused 99 (delta 41), pack-reused 64863
Receiving objects: 100% (64994/64994), 697.25 MiB | 14.96 MiB/s, done.
Resolving deltas: 100% (41502/41502), done.
Updating files: 100% (12864/12864), done.
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_00_715f72a34b73@cloudshell:~ (qwiklabs-gcp-04-7ea9bf2c579b)$ cd ~/ak8s/Autoscaling/
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ ls
loadgen.yaml  web-tolerations.yaml  web.yaml
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```yaml
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ cat web.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      run: web
  template:
    metadata:
      labels:
        run: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: web
        ports:
        - containerPort: 8080
          protocol: TCP  
        resources:
          # You must specify requests for CPU to autoscale
          # based on CPU utilization
          requests:
            cpu: "250m"
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl create -f web.yaml --save-config
deployment.apps/web created
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl expose deployment web --target-port=8080 --type=NodePort
service/web exposed
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get service web
NAME   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
web    NodePort   34.118.231.200   <none>        8080:31289/TCP   7s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```



## Configure autoscaling on the cluster

In this task, you configure the cluster to automatically scale the sample application that you deployed earlier.

To configure your sample application for autoscaling (and to set the  maximum number of replicas to four and the minimum to one, with a CPU  utilization target of 1%), execute the following command.

When you use `kubectl autoscale`, you specify a maximum and minimum number of replicas for your application, as well as a CPU utilization target.

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           61s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl autoscale deployment web --max 4 --min 1 --cpu-percent 1
horizontalpodautoscaler.autoscaling/web autoscaled
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           106s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

### Inspect the HorizontalPodAutoscaler object

The `kubectl autoscale` command you used in the previous task creates a `HorizontalPodAutoscaler` object that targets a specified resource, called the scale target, and scales it as needed.

The autoscaler periodically adjusts the number of replicas of the  scale target to match the average CPU utilization that you specify when  creating the autoscaler.

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get hpa
NAME   REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web    Deployment/web   0%/1%     1         4         1          56s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl describe horizontalpodautoscaler web
Name:                                                  web
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Thu, 20 Jun 2024 12:30:48 +0000
Reference:                                             Deployment/web
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (1m) / 1%
Min replicas:                                          1
Max replicas:                                          4
Deployment pods:                                       1 current / 1 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    ReadyForNewScale    recommended size matches current size
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type     Reason                   Age   From                       Message
  ----     ------                   ----  ----                       -------
  Warning  FailedGetResourceMetric  64s   horizontal-pod-autoscaler  No recommendation
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```yaml
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get horizontalpodautoscaler web -o yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: "2024-06-20T12:30:48Z"
  name: web
  namespace: default
  resourceVersion: "474829"
  uid: 3d6bbbe4-0241-4a8e-b1d7-3534415eb5c3
spec:
  maxReplicas: 4
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 1
        type: Utilization
    type: Resource
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
status:
  conditions:
  - lastTransitionTime: "2024-06-20T12:32:18Z"
    message: recent recommendations were higher than current one, applying the highest
      recent recommendation
    reason: ScaleDownStabilized
    status: "True"
    type: AbleToScale
  - lastTransitionTime: "2024-06-20T12:31:07Z"
    message: the HPA was able to successfully calculate a replica count from cpu resource
      utilization (percentage of request)
    reason: ValidMetricFound
    status: "True"
    type: ScalingActive
  - lastTransitionTime: "2024-06-20T12:31:03Z"
    message: the desired count is within the acceptable range
    reason: DesiredWithinRange
    status: "False"
    type: ScalingLimited
  currentMetrics:
  - resource:
      current:
        averageUtilization: 0
        averageValue: "0"
        value: "0"
      name: cpu
    type: Resource
  currentReplicas: 1
  desiredReplicas: 1
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

### Test the autoscale configuration

You need to create a heavy load on the web application to force it to scale out. You create a configuration file that defines a deployment of four containers that run an infinite loop of HTTP queries against the  sample application web server.

You create the load on your web application by deploying the loadgen application using the `loadgen.yaml` file that has been provided for you.

```yaml
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ cat loadgen.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadgen
spec:
  replicas: 4
  selector:
    matchLabels:
      app: loadgen
  template:
    metadata:
      labels:
        app: loadgen
    spec:
      containers:
      - name: loadgen
        image: k8s.gcr.io/busybox
        args:
        - /bin/sh
        - -c
        - while true; do wget -q -O- http://web:8080; donestudent_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl apply -f loadgen.yaml
deployment.apps/loadgen created
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get deploy
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   4/4     4            4           11s
web       1/1     1            1           4m41s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get hpa
NAME   REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web    Deployment/web   58%/1%    1         4         4          4m11s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

Once the loadgen Pod starts to generate traffic, the web deployment CPU  utilization begins to increase. In the example output, the targets are  now at 58% CPU utilization compared to the 1% CPU threshold.

To stop the load on the web application, scale the loadgen deployment to zero replicas:

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl scale deployment loadgen --replicas 0
deployment.apps/loadgen scaled
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get deploy
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   0/0     0            0           2m10s
web       4/4     4            4           6m40s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

Get the list of deployments to verify that the web application has  scaled down to the minimum value of 1 replica that you configured when  you deployed the autoscaler:



## Manage node pools

In this task, you create a new pool of nodes using preemptible  instances, and then you constrain the web deployment to run only on the  preemptible nodes.

### Add a node pool

1. To deploy a new node pool with three preemptible VM instances, execute the following command:

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ gcloud container node-pools create "temp-pool-1" \
--cluster=$my_cluster --zone=$my_zone \
--num-nodes "2" --node-labels=temp=true --preemptible
Creating node pool temp-pool-1...done.                                                                                                                                             
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-04-7ea9bf2c579b/zones/us-central1-a/clusters/standard-cluster-1/nodePools/temp-pool-1].
NAME: temp-pool-1
MACHINE_TYPE: e2-medium
DISK_SIZE_GB: 100
NODE_VERSION: 1.29.4-gke.1043002
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get nodes
NAME                                                  STATUS   ROLES    AGE   VERSION
gke-standard-cluster-1-temp-pool-1-c546e5da-d86h      Ready    <none>   73s   v1.29.4-gke.1043002
gke-standard-cluster-1-temp-pool-1-c546e5da-dd9n      Ready    <none>   74s   v1.29.4-gke.1043002
gke-standard-cluster-standard-cluster-f4dffa0b-pqsm   Ready    <none>   13h   v1.29.4-gke.1043002
gke-standard-cluster-standard-cluster-f4dffa0b-vh6n   Ready    <none>   13h   v1.29.4-gke.1043002
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get nodes -l temp=true
NAME                                               STATUS   ROLES    AGE    VERSION
gke-standard-cluster-1-temp-pool-1-c546e5da-d86h   Ready    <none>   99s    v1.29.4-gke.1043002
gke-standard-cluster-1-temp-pool-1-c546e5da-dd9n   Ready    <none>   100s   v1.29.4-gke.1043002
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

All the nodes that you added have the `temp=true` label  because you set that label when you created the node-pool. This label  makes it easier to locate and configure these nodes.

### Control scheduling with taints and tolerations

To prevent the scheduler from running a Pod on the temporary nodes,  you add a taint to each of the nodes in the temp pool. Taints are  implemented as a key-value pair with an effect (such as NoExecute) that  determines whether Pods can run on a certain node. Only nodes that are  configured to tolerate the key-value of the taint are scheduled to run  on these nodes.

1. To add a taint to each of the newly created nodes, execute the following command.

You can use the `temp=true` label to apply this change across all the new nodes simultaneously:

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl taint node -l temp=true nodetype=preemptible:NoExecute
node/gke-standard-cluster-1-temp-pool-1-c546e5da-d86h tainted
node/gke-standard-cluster-1-temp-pool-1-c546e5da-dd9n tainted
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

To allow application Pods to execute on these tainted nodes, you must add a tolerations key to the deployment configuration.

```yaml
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ cat web.yaml 


apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      run: web
  template:
    metadata:
      labels:
        run: web
    spec:
      tolerations:
      - key: "nodetype"
        operator: Equal
        value: "preemptible"
      nodeSelector:
        temp: "true"
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: web
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          # You must specify requests for CPU to autoscale
          # based on CPU utilization
          requests:
            cpu: "250m"


student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl apply -f web.yaml
deployment.apps/web configured
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-68c78798d4-2pbkw   1/1     Running   0          44s
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl describe pods -l run=web
Name:             web-68c78798d4-2pbkw
Namespace:        default
Priority:         0
Service Account:  default
Node:             gke-standard-cluster-1-temp-pool-1-c546e5da-d86h/10.10.0.6
Start Time:       Thu, 20 Jun 2024 12:47:29 +0000
Labels:           pod-template-hash=68c78798d4
                  run=web
Annotations:      <none>
Status:           Running
IP:               10.140.2.4
IPs:
  IP:           10.140.2.4
Controlled By:  ReplicaSet/web-68c78798d4
Containers:
  web:
    Container ID:   containerd://49fd98301dae5d7719b537ca139e3dafc179316adf04ad8e464d6f688a49a427
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       gcr.io/google-samples/hello-app@sha256:b1455e1c4fcc5ea1023c9e3b584cd84b64eb920e332feff690a2829696e379e7
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 20 Jun 2024 12:47:31 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qlxhj (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-qlxhj:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              temp=true
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                             nodetype=preemptible
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  54s   default-scheduler  Successfully assigned default/web-68c78798d4-2pbkw to gke-standard-cluster-1-temp-pool-1-c546e5da-d86h
  Normal  Pulling    54s   kubelet            Pulling image "gcr.io/google-samples/hello-app:1.0"
  Normal  Pulled     53s   kubelet            Successfully pulled image "gcr.io/google-samples/hello-app:1.0" in 1.326s (1.326s including waiting)
  Normal  Created    53s   kubelet            Created container web
  Normal  Started    52s   kubelet            Started container web
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

A `Tolerations` section with `nodetype=preemptible` in the list should appear near the bottom of the (truncated) output.

The output confirms that the Pods will tolerate the taint value on  the new preemptible nodes, and thus that they can be scheduled to  execute on those nodes.

To force the web application to scale out again, scale the loadgen deployment back to four replicas:

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl scale deployment loadgen --replicas 4
deployment.apps/loadgen scaled
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE                                                  NOMINATED NODE   READINESS GATES
loadgen-65c9cf7485-4889r   1/1     Running   0          21s     10.140.1.11   gke-standard-cluster-standard-cluster-f4dffa0b-pqsm   <none>           <none>
loadgen-65c9cf7485-7xzhf   1/1     Running   0          21s     10.140.1.10   gke-standard-cluster-standard-cluster-f4dffa0b-pqsm   <none>           <none>
loadgen-65c9cf7485-krbkd   1/1     Running   0          20s     10.140.0.18   gke-standard-cluster-standard-cluster-f4dffa0b-vh6n   <none>           <none>
loadgen-65c9cf7485-m8j8h   1/1     Running   0          21s     10.140.0.17   gke-standard-cluster-standard-cluster-f4dffa0b-vh6n   <none>           <none>
web-68c78798d4-2cj74       1/1     Running   0          3s      10.140.3.6    gke-standard-cluster-1-temp-pool-1-c546e5da-dd9n      <none>           <none>
web-68c78798d4-2pbkw       1/1     Running   0          2m24s   10.140.2.4    gke-standard-cluster-1-temp-pool-1-c546e5da-d86h      <none>           <none>
web-68c78798d4-flln5       1/1     Running   0          3s      10.140.2.5    gke-standard-cluster-1-temp-pool-1-c546e5da-d86h      <none>           <none>
web-68c78798d4-jfdnl       1/1     Running   0          3s      10.140.3.5    gke-standard-cluster-1-temp-pool-1-c546e5da-dd9n      <none>           <none>
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

You could scale just the web application directly but using the loadgen  app will allow you to see how the different taint,  toleration and  nodeSelector settings that apply to the web and loadgen applications  affect which nodes they are scheduled on.

This shows that the loadgen app is running only on `default-pool` nodes while the web app is running only the preemptible nodes in `temp-pool-1. `

The taint setting prevents Pods from running on the preemptible nodes so the loadgen application only runs on the default pool. The  toleration setting allows the web application to run on the preemptible  nodes and the nodeSelector forces the web application Pods to run on  those nodes.

```sh
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ history 
    1  export my_zone=us-central1-a
    2  export my_cluster=standard-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    5  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    6  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
    7  cd ~/ak8s/Autoscaling/
    8  ls
    9  cat web.yaml 
   10  kubectl create -f web.yaml --save-config
   11  kubectl expose deployment web --target-port=8080 --type=NodePort
   12  kubectl get service web
   13  kubectl get deployment
   14  kubectl autoscale deployment web --max 4 --min 1 --cpu-percent 1
   15  kubectl get deployment
   16  kubectl get hpa
   17  kubectl describe horizontalpodautoscaler web
   18  kubectl get horizontalpodautoscaler web -o yaml
   19  ls
   20  cat loadgen.yaml 
   21  kubectl apply -f loadgen.yaml
   22  kubectl get deploy
   23  kubectl get hpa
   24  kubectl scale deployment loadgen --replicas 0
   25  kubectl get deploy
   26  kubectl get hpa
   27  kubectl get deploy
   28  kubectl get hpa
   29  kubectl get deploy
   30  kubectl get hpa
   31  gcloud container node-pools create "temp-pool-1" --cluster=$my_cluster --zone=$my_zone --num-nodes "2" --node-labels=temp=true --preemptible
   32  kubectl get hpa
   33  kubectl get nodes
   34  kubectl get nodes -l temp=true
   35  kubectl taint node -l temp=true nodetype=preemptible:NoExecute
   36  nano web.yaml
   37  cat web
   38  cat web.yaml 
   39  vi web.yaml 
   40  cat web.yaml 
   41  rm web.yaml 
   42  vi web.yaml
   43  cat web.yaml 
   44  kubectl apply -f web.yaml
   45  kubectl get pods
   46  kubectl describe pods -l run=web
   47  kubectl scale deployment loadgen --replicas 4
   48  kubectl get pods -o wide
   49  history 
student_00_715f72a34b73@cloudshell:~/ak8s/Autoscaling (qwiklabs-gcp-04-7ea9bf2c579b)$ 
```

