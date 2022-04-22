---
layout: single
title:  "Kubernetes StatefulSets"
date:   2022-04-05 12:55:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/sts-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes StatefulSets

According to Kubernetes documentation,

> StatefulSet is the workload API object used to manage stateful applications.

> Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.

> Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

> If you want to use storage volumes to provide persistence for your workload, you can use a StatefulSet as part of the solution. Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.


```yaml
lab@k8s1:~$ cat sts-demo.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi


```



```sh
lab@k8s1:~$ kubectl apply -f sts-demo.yaml
service/nginx created
statefulset.apps/web created
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get pods -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
web-0   0/1     Pending   0          17s

^Clab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-0                  0/1     Pending   0          114s
web-79d88c97d6-4w4nl   1/1     Running   0          6d8h
web-79d88c97d6-7ktck   1/1     Running   0          6d8h
web-79d88c97d6-8tvsk   1/1     Running   0          6d8h
web-79d88c97d6-96sd9   1/1     Running   0          6d8h
web-79d88c97d6-9tzlh   1/1     Running   0          6d8h
web-79d88c97d6-brtgx   1/1     Running   0          6d8h
web-79d88c97d6-kngc4   1/1     Running   0          6d8h
web-79d88c97d6-p5vfg   1/1     Running   0          6d8h
web-79d88c97d6-rbhpr   1/1     Running   0          6d8h
lab@k8s1:~$ kubectl get pods -w -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
web-0   0/1     Pending   0          119s
```

The pods are not getting created as the `pod has unbound immediate PersistentVolumeClaims.`

```yaml
lab@k8s1:~$ cat pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: www
  labels:
    type: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl create -f pv.yaml
persistentvolume/www created

```

```sh
lab@k8s1:~$ kubectl get pv,pvc
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
persistentvolume/www   1Gi        RWO            Retain           Bound    default/www-web-0                           6s

NAME                              STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-web-0   Bound    www      1Gi        RWO                           10m
lab@k8s1:~$
```

After this, we can see one pod got created, `web-0` in `Running` state while `web-1` still in `Pending` state.

```sh
lab@k8s1:~$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
web-0                  1/1     Running   0          10m
web-1                  0/1     Pending   0          14s
web-79d88c97d6-4w4nl   1/1     Running   0          6d8h
web-79d88c97d6-7ktck   1/1     Running   0          6d8h
web-79d88c97d6-8tvsk   1/1     Running   0          6d8h
web-79d88c97d6-96sd9   1/1     Running   0          6d8h
web-79d88c97d6-9tzlh   1/1     Running   0          6d8h
web-79d88c97d6-brtgx   1/1     Running   0          6d8h
web-79d88c97d6-kngc4   1/1     Running   0          6d8h
web-79d88c97d6-p5vfg   1/1     Running   0          6d8h
web-79d88c97d6-rbhpr   1/1     Running   0          6d8h
lab@k8s1:~$
```



Let us describe it to understand the reason

```sh
lab@k8s1:~$ kubectl describe pods web-1
Name:           web-1
Namespace:      default
Priority:       0
Node:           <none>
Labels:         app=nginx
                controller-revision-hash=web-b46f789c4
                statefulset.kubernetes.io/pod-name=web-1
Annotations:    <none>
Status:         Pending
IP:
IPs:            <none>
Controlled By:  StatefulSet/web
Containers:
  nginx:
    Image:        k8s.gcr.io/nginx-slim:0.8
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /usr/share/nginx/html from www (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-d65sv (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  www:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  www-web-1
    ReadOnly:   false
  kube-api-access-d65sv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  5m17s  default-scheduler  0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling  4m5s   default-scheduler  0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.
lab@k8s1:~$
```

Hmm, its again due to `FailedScheduling`. `0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.` 

```sh
lab@k8s1:~$ kubectl get pv,pvc
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
persistentvolume/www   1Gi        RWO            Retain           Bound    default/www-web-0                           7m29s

NAME                              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-web-0   Bound     www      1Gi        RWO                           17m
persistentvolumeclaim/www-web-1   Pending                                                     7m12s
lab@k8s1:~$
```

We can see `persistentvolumeclaim/www-web-1` in Pending state.

Looking back at the PersistentVolume, we have defined only 1G and that is requested by `web-0`, so nothing left for `web-1`.Let us increase the `PV` size to 2Gi.



Let us delete all the previously created STS related resources and apply again.

```sh
lab@k8s1:~$ kubectl get pv,pvc,sts
No resources found
lab@k8s1:~$
```



```yaml
lab@k8s1:~$ cat sts-final-demo.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl apply -f sts-final-demo.yaml
service/nginx unchanged
statefulset.apps/web created
persistentvolume/www created
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pv,pvc,sts
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
persistentvolume/www   2Gi        RWO            Retain           Bound    default/www-web-0                           65s

NAME                              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/www-web-0   Bound     www      2Gi        RWO                           65s
persistentvolumeclaim/www-web-1   Pending                                                     54s

NAME                   READY   AGE
statefulset.apps/web   1/2     65s
lab@k8s1:~$
```

Hmm, increasing the PV size to 2G did not help. This time, all 2G capacity is bound to `web-0`. 

Let us delete all again.

```sh
lab@k8s1:~$ kubectl get pv,pvc,sts
No resources found
lab@k8s1:~$
```

> To be continued

