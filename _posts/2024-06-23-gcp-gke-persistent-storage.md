---

layout: single
title:  "Configuring Persistent Storage for Google Kubernetes Engine"
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
# Configuring Persistent Storage for Google Kubernetes Engine

PersistentVolumes are storage that is available to a Kubernetes cluster. PersistentVolumeClaims enable Pods to access PersistentVolumes. Without PersistentVolumeClaims Pods are mostly ephemeral, so you should use  PersistentVolumeClaims for any data that you expect to survive Pod  scaling, updating, or migrating.

- Create manifests for PersistentVolumes (PVs) and  PersistentVolumeClaims (PVCs) for Google Cloud persistent disks  (dynamically created or existing)
- Mount Google Cloud persistent disk PVCs as volumes in Pods
- Use manifests to create StatefulSets
- Mount Google Cloud persistent disk PVCs as volumes in StatefulSets
- Verify the connection of Pods in StatefulSets to particular PVs as the Pods are stopped and restarted

## Create PVs and PVCs

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-41684334779c.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ export my_region=us-east1
export my_cluster=autopilot-cluster-1
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ source <(kubectl completion bash)
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ gcloud container clusters get-credentials $my_cluster --region $my_region
Fetching cluster endpoint and auth data.
kubeconfig entry generated for autopilot-cluster-1.
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

### Create and apply a manifest with a PVC

Most of the time, you don't need to directly configure PV objects or  create Compute Engine persistent disks. Instead, you can create a PVC,  and Kubernetes automatically provisions a persistent disk for you.

Let's creates a 30 gigabyte PVC called `hello-web-disk`  that can be mounted as a read-write volume on a single node at a time.

```yaml
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ cat pvc-demo.yaml 


apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hello-web-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi

student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get persistentvolumeclaim
No resources found in default namespace.
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl apply -f pvc-demo.yaml
persistentvolumeclaim/hello-web-disk created
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get persistentvolumeclaim
NAME             STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
hello-web-disk   Pending                                      standard-rwo   <unset>                 4s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

> The status will remain pending until after the next step.

## Mount and verify Google Cloud persistent disk PVCs in Pods

In this task, you attach your persistent disk PVC to a Pod. You mount the PVC as a volume as part of the manifest for the Pod.

### Mount the PVC to a Pod

Create a  manifest file `pod-volume-demo.yaml` to deploy an nginx container, attach the `pvc-demo-volume` to the Pod and mount that volume to the path `/var/www/html` inside the nginx container.  Files saved to this directory inside the  container will be saved to the persistent volume and persist even if the Pod and the container are shutdown and recreated.

```yaml
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ cat pod-volume-demo.yaml 


kind: Pod
apiVersion: v1
metadata:
  name: pvc-demo-pod
spec:
  containers:
    - name: frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: pvc-demo-volume
  volumes:
    - name: pvc-demo-volume
      persistentVolumeClaim:
        claimName: hello-web-disk

student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

To create the Pod with the volume, execute the following command

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   0/1     Pending   0          6s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   0/1     Pending   0          11s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   0/1     Pending   0          32s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Pending                                      standard-rwo   <unset>                 3m28s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Pending                                      standard-rwo   <unset>                 3m35s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Pending                                      standard-rwo   <unset>                 3m42s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Pending                                      standard-rwo   <unset>                 3m57s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Pending                                      standard-rwo   <unset>                 4m
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   0/1     Pending   0          75s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl describe pods pvc-demo-pod
Name:             pvc-demo-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           <none>
Annotations:      autopilot.gke.io/resource-adjustment:
                    {"input":{"containers":[{"name":"frontend"}]},"output":{"containers":[{"limits":{"cpu":"500m","ephemeral-storage":"1Gi","memory":"2Gi"},"r...
                  autopilot.gke.io/warden-version: 2.9.37
Status:           Pending
SeccompProfile:   RuntimeDefault
IP:               
IPs:              <none>
Containers:
  frontend:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qbvwz (ro)
      /var/www/html from pvc-demo-volume (rw)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  pvc-demo-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hello-web-disk
    ReadOnly:   false
  kube-api-access-qbvwz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 kubernetes.io/arch=amd64:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                From                                   Message
  ----     ------            ----               ----                                   -------
  Warning  FailedScheduling  90s                gke.io/optimize-utilization-scheduler  no nodes available to schedule pods
  Warning  FailedScheduling  86s (x2 over 88s)  gke.io/optimize-utilization-scheduler  no nodes available to schedule pods
  Normal   TriggeredScaleUp  49s                cluster-autoscaler                     pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-41684334779c/zones/us-east1-b/instanceGroups/gk3-autopilot-cluster-1-nap-l8h3ctae-3077ca01-grp 0->1 (max: 1000)}]
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

If you do this quickly after creating the Pod, you will see the status  listed as "ContainerCreating" while the volume is mounted before the  status changes to "Running".

after sometime,

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   1/1     Running   0          6m35s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            Delete           Bound    default/hello-web-disk   standard-rwo   <unset>                          4m33s

NAME                                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Bound    pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            standard-rwo   <unset>                 9m28s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl describe pod pvc-demo-pod
Name:             pvc-demo-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             gk3-autopilot-cluster-1-nap-l8h3ctae-3077ca01-l9jp/10.142.0.4
Start Time:       Sun, 23 Jun 2024 13:33:37 +0000
Labels:           <none>
Annotations:      autopilot.gke.io/resource-adjustment:
                    {"input":{"containers":[{"name":"frontend"}]},"output":{"containers":[{"limits":{"cpu":"500m","ephemeral-storage":"1Gi","memory":"2Gi"},"r...
                  autopilot.gke.io/warden-version: 2.9.37
                  cloud.google.com/cluster_autoscaler_unhelpable_since: 2024-06-23T13:33:33+0000
                  cloud.google.com/cluster_autoscaler_unhelpable_until: Inf
Status:           Running
SeccompProfile:   RuntimeDefault
IP:               10.113.0.14
IPs:
  IP:  10.113.0.14
Containers:
  frontend:
    Container ID:   containerd://42ef705f72099ef1d88a81df97e75cb78e86ed55e76f226cdb4ceeb88258e45a
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:9c367186df9a6b18c6735357b8eb7f407347e84aea09beb184961cb83543d46e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 23 Jun 2024 13:34:14 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qbvwz (ro)
      /var/www/html from pvc-demo-volume (rw)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  pvc-demo-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  hello-web-disk
    ReadOnly:   false
  kube-api-access-qbvwz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 kubernetes.io/arch=amd64:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age                    From                                   Message
  ----     ------                  ----                   ----                                   -------
  Warning  FailedScheduling        7m8s                   gke.io/optimize-utilization-scheduler  no nodes available to schedule pods
  Warning  FailedScheduling        5m43s (x11 over 7m7s)  gke.io/optimize-utilization-scheduler  no nodes available to schedule pods
  Normal   Scheduled               5m                     gke.io/optimize-utilization-scheduler  Successfully assigned default/pvc-demo-pod to gk3-autopilot-cluster-1-nap-l8h3ctae-3077ca01-l9jp
  Normal   TriggeredScaleUp        6m28s                  cluster-autoscaler                     pod triggered scale-up: [{https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-41684334779c/zones/us-east1-b/instanceGroups/gk3-autopilot-cluster-1-nap-l8h3ctae-3077ca01-grp 0->1 (max: 1000)}]
  Normal   NotTriggerScaleUp       5m5s                   cluster-autoscaler                     pod didn't trigger scale-up (it wouldn't fit if a new node is added): 3 Insufficient memory, 1 node(s) had untolerated taint {cloud.google.com/gke-quick-remove: true}, 18 node(s) didn't find available persistent volumes to bind, 3 Insufficient cpu
  Normal   SuccessfulAttachVolume  4m53s                  attachdetach-controller                AttachVolume.Attach succeeded for volume "pvc-efe382e6-a099-4055-a17d-742273e9ec65"
  Normal   Pulling                 4m45s                  kubelet                                Pulling image "nginx"
  Normal   Pulled                  4m24s                  kubelet                                Successfully pulled image "nginx" in 15.328s (21.091s including waiting)
  Normal   Created                 4m24s                  kubelet                                Created container frontend
  Normal   Started                 4m24s                  kubelet                                Started container frontend
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl exec -it pvc-demo-pod -- sh
# echo Test webpage in a persistent volume!>/var/www/html/index.html
chmod +x /var/www/html/index.html# 
# cat /var/www/html/index.html
Test webpage in a persistent volume!
# exit
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

You will now delete the Pod from the cluster, confirm that the PV still  exists, then redeploy the Pod and verify the contents of the PV remain  intact.



### Test the persistence of the PV

You will now delete the Pod from the cluster, confirm that the PV  still exists, then redeploy the Pod and verify the contents of the PV  remain intact.

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl delete pod pvc-demo-pod
pod "pvc-demo-pod" deleted
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
No resources found in default namespace.
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

There should be no Pods on the cluster. Your PVC still exists, and was not deleted when the Pod was deleted.

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get persistentvolumeclaim
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
hello-web-disk   Bound    pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            standard-rwo   <unset>                 12m
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
persistentvolume/pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            Delete           Bound    default/hello-web-disk   standard-rwo   <unset>                          8m13s

NAME                                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/hello-web-disk   Bound    pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            standard-rwo   <unset>                 13m
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

Redeploy the pvc-demo-pod:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl apply -f pod-volume-demo.yaml
Warning: autopilot-default-resources-mutator:Autopilot updated Pod default/pvc-demo-pod: defaulted unspecified 'cpu' resource for containers [frontend] (see http://g.co/gke/autopilot-defaults).
pod/pvc-demo-pod created
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS              RESTARTS   AGE
pvc-demo-pod   0/1     ContainerCreating   0          5s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS              RESTARTS   AGE
pvc-demo-pod   0/1     ContainerCreating   0          8s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pvc-demo-pod   1/1     Running   0          15s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

The Pod will deploy and the status will change to "Running" faster  this time because the PV already exists and does not need to be created.

1. To verify the PVC is still accessible within the Pod, you must gain  shell access to your Pod. To start the shell session, execute the  following command:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl exec -it pvc-demo-pod -- sh
# cat /var/www/html/index.html
Test webpage in a persistent volume!
# exit
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

The contents of the persistent volume were not removed, even though the Pod was deleted from the cluster and recreated.

## Create StatefulSets with PVCs

In this task, you use your PVC in a StatefulSet. A StatefulSet is  like a Deployment, except that the Pods are given unique identifiers.

Release the PVC

Before you can use the PVC with the statefulset, you must delete the Pod that is currently using it. Execute the following command to delete the Pod:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl delete pod pvc-demo-pod
pod "pvc-demo-pod" deleted
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
No resources found in default namespace.
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```



### Create a StatefulSet

Let's create a manifest file `statefulset-demo.yaml` that  creates a StatefulSet that includes a LoadBalancer service and three  replicas of a Pod containing an nginx container and a  volumeClaimTemplate for 30 gigabyte PVCs with the name `hello-web-disk`. The nginx containers mount the PVC called `hello-web-disk` at `/var/www/html` as in the previous task.

```yaml
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ cat statefulset-demo.yaml 


kind: Service
apiVersion: v1
metadata:
  name: statefulset-demo-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  type: LoadBalancer
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: statefulset-demo
spec:
  selector:
    matchLabels:
      app: MyApp
  serviceName: statefulset-demo-service
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: stateful-set-container
        image: nginx
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: hello-web-disk
          mountPath: "/var/www/html"
  volumeClaimTemplates:
  - metadata:
      name: hello-web-disk
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 30Gi

student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

To create the StatefulSet with the volume, execute the following command:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl apply -f statefulset-demo.yaml
service/statefulset-demo-service created
Warning: autopilot-default-resources-mutator:Autopilot updated StatefulSet default/statefulset-demo: defaulted unspecified 'cpu' resource for containers [stateful-set-container] (see http://g.co/gke/autopilot-defaults).
statefulset.apps/statefulset-demo created
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```



### Verify the connection of Pods in StatefulSets

1. Use "kubectl describe" to view the details of the StatefulSet:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl describe statefulset statefulset-demo
Name:               statefulset-demo
Namespace:          default
CreationTimestamp:  Sun, 23 Jun 2024 13:49:57 +0000
Selector:           app=MyApp
Labels:             <none>
Annotations:        autopilot.gke.io/resource-adjustment:
                      {"input":{"containers":[{"name":"stateful-set-container"}]},"output":{"containers":[{"limits":{"cpu":"500m","ephemeral-storage":"1Gi","mem...
                    autopilot.gke.io/warden-version: 2.9.37
Replicas:           3 desired | 2 total
Update Strategy:    RollingUpdate
Pods Status:        1 Running / 1 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=MyApp
  Containers:
   stateful-set-container:
    Image:      nginx
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/www/html from hello-web-disk (rw)
  Volumes:  <none>
Volume Claims:
  Name:          hello-web-disk
  StorageClass:  
  Labels:        <none>
  Annotations:   <none>
  Capacity:      30Gi
  Access Modes:  [ReadWriteOnce]
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  20s   statefulset-controller  create Claim hello-web-disk-statefulset-demo-0 Pod statefulset-demo-0 in StatefulSet statefulset-demo success
  Normal  SuccessfulCreate  19s   statefulset-controller  create Pod statefulset-demo-0 in StatefulSet statefulset-demo successful
  Normal  SuccessfulCreate  5s    statefulset-controller  create Claim hello-web-disk-statefulset-demo-1 Pod statefulset-demo-1 in StatefulSet statefulset-demo success
  Normal  SuccessfulCreate  4s    statefulset-controller  create Pod statefulset-demo-1 in StatefulSet statefulset-demo successful
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

Note the event status at the end of the output. The service and statefulset created successfully.

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
statefulset-demo-0   1/1     Running   0          80s
statefulset-demo-1   1/1     Running   0          65s
statefulset-demo-2   0/1     Pending   0          51s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pvc
NAME                                STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
hello-web-disk                      Bound     pvc-efe382e6-a099-4055-a17d-742273e9ec65   30Gi       RWO            standard-rwo   <unset>                 23m
hello-web-disk-statefulset-demo-0   Bound     pvc-2c552959-5f21-4434-b99f-6b0c74d3a4d6   30Gi       RWO            standard-rwo   <unset>                 2m1s
hello-web-disk-statefulset-demo-1   Bound     pvc-bf6c9e65-1be5-41c1-82d7-8b9cc6f03303   30Gi       RWO            standard-rwo   <unset>                 106s
hello-web-disk-statefulset-demo-2   Pending                                                                        standard-rwo   <unset>                 91s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

The original hello-web-disk is still there and you can now see the  individual PVCs that were created for each Pod in the new statefulset  Pod.

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl describe pvc hello-web-disk-statefulset-demo-0
Name:          hello-web-disk-statefulset-demo-0
Namespace:     default
StorageClass:  standard-rwo
Status:        Bound
Volume:        pvc-2c552959-5f21-4434-b99f-6b0c74d3a4d6
Labels:        app=MyApp
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: pd.csi.storage.gke.io
               volume.kubernetes.io/selected-node: gk3-autopilot-cluster-1-nap-l8h3ctae-3077ca01-l9jp
               volume.kubernetes.io/storage-provisioner: pd.csi.storage.gke.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      30Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       statefulset-demo-0
Events:
  Type    Reason                 Age    From                                                                                              Message
  ----    ------                 ----   ----                                                                                              -------
  Normal  WaitForFirstConsumer   2m40s  persistentvolume-controller                                                                       waiting for first consumer to be created before binding
  Normal  Provisioning           2m39s  pd.csi.storage.gke.io_gke-108836dd682049699d1e-bde5-8604-vm_07f99d8a-4f7c-4e8d-bb52-f7671552978c  External provisioner is provisioning volume for claim "default/hello-web-disk-statefulset-demo-0"
  Normal  ExternalProvisioning   2m39s  persistentvolume-controller                                                                       Waiting for a volume to be created either by the external provisioner 'pd.csi.storage.gke.io' or manually by the system administrator. If volume creation is delayed, please verify that the provisioner is running and correctly registered.
  Normal  ProvisioningSucceeded  2m36s  pd.csi.storage.gke.io_gke-108836dd682049699d1e-bde5-8604-vm_07f99d8a-4f7c-4e8d-bb52-f7671552978c  Successfully provisioned volume pvc-2c552959-5f21-4434-b99f-6b0c74d3a4d6
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```



## Verify the persistence of Persistent Volume connections to Pods managed by StatefulSets

In this task, you verify the connection of Pods in StatefulSets to particular PVs as the Pods are stopped and restarted.

To verify that the PVC is accessible within the Pod, you must gain shell access to your Pod. To start the shell session, execute the following  command: Verify that there is no `index.html` text file in the `/var/www/html` directory: To create a simple text message as a web page in the Pod enter the following commands: Verify the text file contains your message:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl exec -it statefulset-demo-0 -- sh
# cat /var/www/html/index.html
cat: /var/www/html/index.html: No such file or directory
# echo Test webpage in a persistent volume!>/var/www/html/index.html
chmod +x /var/www/html/index.html# 
# cat /var/www/html/index.html
Test webpage in a persistent volume!
# exit
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

Delete the Pod where you updated the file on the PVC:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl delete pod statefulset-demo-0
pod "statefulset-demo-0" deleted
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
statefulset-demo-0   1/1     Running   0          28s
statefulset-demo-1   1/1     Running   0          6m22s
statefulset-demo-2   1/1     Running   0          7m19s
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

You will see that the StatefulSet is automatically restarting the `statefulset-demo-0` Pod.

The StatefulSet restarts the Pod and reconnects the existing dedicated  PVC to the new Pod, ensuring that the data for that Pod is preserved. Verify that the text file still contains your message:

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ kubectl exec -it statefulset-demo-0 -- sh
# cat /var/www/html/index.html
Test webpage in a persistent volume!
# exit
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

## History

```sh
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ history 
    1  export my_region=us-east1
    2  export my_cluster=autopilot-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud container clusters get-credentials $my_cluster --region $my_region
    5  nano pvc-demo.yaml
    6  cat pvc-demo.yaml 
    7  kubectl get persistentvolumeclaim
    8  kubectl apply -f pvc-demo.yaml
    9  kubectl get persistentvolumeclaim
   10  nano pod-volume-demo.yaml
   11  cat pod-volume-demo.yaml 
   12  kubectl apply -f pod-volume-demo.yaml
   13  kubectl get pods
   14  kubectl get pv,pvc
   15  kubectl get pods
   16  kubectl describe pods pvc-demo-pod
   17  kubectl get pods
   18  kubectl get pv,pvc
   19  kubectl describe pod pvc-demo-pod
   20  kubectl exec -it pvc-demo-pod -- sh
   21  kubectl get pods
   22  kubectl get pods -o wide
   23  curl 10.113.0.14
   24  kubectl delete pod pvc-demo-pod
   25  kubectl get pods
   26  kubectl get persistentvolumeclaim
   27  kubectl get pv,pvc
   28  kubectl apply -f pod-volume-demo.yaml
   29  kubectl get pods
   30  kubectl exec -it pvc-demo-pod -- sh
   31  kubectl delete pod pvc-demo-pod
   32  kubectl get pods
   33  nano statefulset-demo.yaml
   34  cat statefulset-demo.yaml 
   35  kubectl apply -f statefulset-demo.yaml
   36  kubectl describe statefulset statefulset-demo
   37  kubectl get pods
   38  kubectl get pvc
   39  kubectl describe pvc hello-web-disk-statefulset-demo-0
   40  kubectl exec -it statefulset-demo-0 -- sh
   41  kubectl delete pod statefulset-demo-0
   42  kubectl get pods
   43  kubectl exec -it statefulset-demo-0 -- sh
   44  history 
student_03_995ecaa6bfd9@cloudshell:~ (qwiklabs-gcp-01-41684334779c)$ 
```

