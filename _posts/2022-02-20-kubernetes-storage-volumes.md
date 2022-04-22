---
layout: single
title:  "Kubernetes Storage Volumes"
date:   2022-02-20 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/vol-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Storage Volumes


## Storage

### Volumes

#### emptyDir
There are many volume types. 
An `emptyDir` volume is first created when a Pod is assigned to a node, and exists as long as that Pod is running on that node. 
When a Pod is removed from a node for any reason, the data in the `emptyDir` is deleted permanently.

Here is an example for creating a volume of type `emptyDir` and use it in a Pod.

```yaml
pradeep@learnk8s$ more pod-with-emptydir-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-emptydir-volume
spec:
  volumes:
  - name: logs-volume
    emptyDir: {}
  containers:
  - image: nginx
    name: pod-with-emptydir-volume
    volumeMounts:
    - mountPath: /var/logs
      name: logs-volume
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-emptydir-volume.yaml
pod/pod-with-emptydir-volume created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running            0               9s
```
```shell
pradeep@learnk8s$ kubectl exec -it pod-with-emptydir-volume -- /bin/sh
# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# cd /var/logs
# ls
# pwd
/var/logs
# touch testing-volumes.txt
# ls
testing-volumes.txt
# exit 0
```


#### HostPath

```yaml
pradeep@learnk8s$ cat pod-with-hostpath-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-hostpath-volume
spec:
  containers:
  - image: nginx
    name: pod-with-hostpath-volume
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data-for-pod-with-hostpath-volume
      # this field is optional
      type: Directory
```
```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ ls
$ sudo mkdir /data-for-pod-with-hostpath-volume
$ sudo vi /data-for-pod-with-hostpath-volume/index.html
$ sudo cat /data-for-pod-with-hostpath-volume/index.html
testing Pod with hostPath volume mount!
$ exit
logout
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-hostpath-volume.yaml
pod/pod-with-hostpath-volume created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running             0                 13m
pod-with-hostpath-volume    0/1     ContainerCreating   0                 53s
```
Even after 50+ seconds, the `pod-with-hostpath-volume` seems to be not yet Running.
Let us describe it to understand the reason. Pay attention to Mounts section under Containers and Volumes section. Finally look at the events.
```shell
pradeep@learnk8s$ kubectl describe pods pod-with-hostpath-volume
Name:         pod-with-hostpath-volume
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Sun, 20 Feb 2022 18:43:42 +0530
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:
IPs:          <none>
Containers:
  pod-with-hostpath-volume:
    Container ID:
    Image:          nginx
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from test-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-f7lsg (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  test-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /data-for-pod-with-hostpath-volume
    HostPathType:  Directory
  kube-api-access-f7lsg:
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
  Type     Reason       Age                From               Message
  ----     ------       ----               ----               -------
  Normal   Scheduled    61s                default-scheduler  Successfully assigned default/pod-with-hostpath-volume to k8s-m02
  Warning  FailedMount  28s (x7 over 60s)  kubelet            MountVolume.SetUp failed for volume "test-volume" : hostPath type check failed: /data-for-pod-with-hostpath-volume is not a directory
```

From the events, we can see that the default-scheduler tried to create this pod on the node `k8s-m02` but MountVolume.SetUp failed for volume `test-volume` : hostPath type check failed: `/data-for-pod-with-hostpath-volume` is not a directory.

This is expected, right. A moment ago, we created a directory with this name on the `k8s` node.

To fix this, we can manually schedule this pod on the `k8s` node with the `nodeName` spec.
We did something like this already. Didn't we?

First, let us delete the pod which is still in creatingContainer stage.
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running             0                21m
pod-with-hostpath-volume    0/1     ContainerCreating   0                9m32s
```
```shell
pradeep@learnk8s$ kubectl delete pods pod-with-hostpath-volume
pod "pod-with-hostpath-volume" deleted
```
Modify the YAML definition file like this. Only thing different (from the previous definition) here  is the `nodeName: k8s` line.
```yaml
pradeep@learnk8s$ cat pod-with-hostpath-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-hostpath-volume
spec:
  nodeName: k8s
  containers:
  - image: nginx
    name: pod-with-hostpath-volume
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data-for-pod-with-hostpath-volume
      # this field is optional
      type: Directory
```
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running            0               26m
pod-with-hostpath-volume    1/1     Running            0               42s
```
This time it is running fine.
If we describe this Pod,
```shell
pradeep@learnk8s$ kubectl describe pods pod-with-hostpath-volume
Name:         pod-with-hostpath-volume
Namespace:    default
Priority:     0
Node:         k8s/192.168.177.29
Start Time:   Sun, 20 Feb 2022 18:56:51 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.0.18
IPs:
  IP:  10.244.0.18
Containers:
  pod-with-hostpath-volume:
    Container ID:   docker://f1e79dd787c5327766bbdde7b35cab8314de09e1c406fe274ee3f7c364cc5845
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:2834dc507516af02784808c5f48b7cbe38b8ed5d0f4837f16e78d00deb7e7767
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 20 Feb 2022 18:57:25 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from test-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7fv4m (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  test-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /data-for-pod-with-hostpath-volume
    HostPathType:  Directory
  kube-api-access-7fv4m:
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
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  50s   kubelet  Pulling image "nginx"
  Normal  Pulled   19s   kubelet  Successfully pulled image "nginx" in 30.708250752s
  Normal  Created  19s   kubelet  Created container pod-with-hostpath-volume
  Normal  Started  18s   kubelet  Started container pod-with-hostpath-volume
```
Get the IP assigned to the Pod, either from the previous command or with the `-o wide` option.

```shell
pradeep@learnk8s$ kubectl get pods -o wide | grep volume
pod-with-emptydir-volume    1/1     Running            0                38m     10.244.1.39   k8s-m02   <none>           <none>
pod-with-hostpath-volume    1/1     Running            0                13m     10.244.0.18   k8s       <none>           <none>
```

Let us connect to this Pod on the IP address assigned `10.244.0.18`.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.0.18
testing Pod with hostPath volume mount!
```

The test is working. The `nginx` container in the `pod-with-hostpath-volume` is readigng the data from the host directory.



#### Config Maps

A ConfigMap provides a way to inject configuration data into pods. The data stored in a ConfigMap can be referenced in a volume of type configMap and then consumed by containerized applications running in a pod.

https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/

Let us try this example as it is in our Minikube setup.

```yaml
pradeep@learnk8s$ cat redis-pod-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4
    command:
      - redis-server
      - "/redis-master/redis.conf"
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data
      name: data
    - mountPath: /redis-master
      name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
        - key: redis-config
          path: redis.conf

```
```shell
pradeep@learnk8s$ cat <<EOF >./example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: ""
EOF
```
```shell
pradeep@learnk8s$ kubectl apply -f example-redis-config.yaml
configmap/example-redis-config created
```
```shell
pradeep@learnk8s$ kubectl apply -f redis-pod-configmap.yaml
pod/redis created
```
```shell
pradeep@learnk8s$ kubectl get pod/redis configmap/example-redis-config -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP            NODE      NOMINATED NODE   READINESS GATES
pod/redis   1/1     Running   0          53s   10.244.1.40   k8s-m02   <none>           <none>

NAME                             DATA   AGE
configmap/example-redis-config   1      64s
```
Describe the configmap.
```shell
pradeep@learnk8s$ kubectl describe configmap/example-redis-config
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----


BinaryData
====

Events:  <none>
```
You should see an empty `redis-config` key.

Use `kubectl exec` to enter the pod and run the `redis-cli` tool to check the current configuration:

Check `maxmemory` and `maxmemory-policy`.

```shell
pradeep@learnk8s$ kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
127.0.0.1:6379>
```
Now let's add some configuration values to the `example-redis-config` ConfigMap:
```yaml
pradeep@learnk8s$ cat example-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-redis-config
data:
  redis-config: |
    maxmemory 2mb
    maxmemory-policy allkeys-lru
```
Apply the updated configmap.
```shell
pradeep@learnk8s$ kubectl apply -f example-redis-config.yaml
configmap/example-redis-config configured
```

Confirm configmap is updated.
```shell
pradeep@learnk8s$ kubectl describe configmap/example-redis-config
Name:         example-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb
maxmemory-policy allkeys-lru


BinaryData
====

Events:  <none>
```
Check the Redis Pod again using `redis-cli` via `kubectl exec` to see if the configuration was applied:

```shell
pradeep@learnk8s$ kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
127.0.0.1:6379> exit
```
Are the changes reflecting now? Nope! Not yet. 

The configuration values have not changed because the Pod needs to be restarted to grab updated values from associated ConfigMaps. Let's delete and recreate the Pod:

```shell
pradeep@learnk8s$ kubectl delete pod redis
pod "redis" deleted
```
```shell
pradeep@learnk8s$ kubectl apply -f redis-pod-configmap.yaml
pod/redis created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep redis
redis                       1/1     Running            0                 10s
```
```shell
pradeep@learnk8s$ kubectl exec -it redis -- redis-cli
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
127.0.0.1:6379> exit
```
This time we can seen updated (from ConfigMap) values inside the `redis` container.



#### Secrets

We are re-visiting secrets again, but this time not as an Envrionment variable but as a volume Mount.

```shell
pradeep@learnk8s$ kubectl create secret generic my-secret --from-literal=user=pradeep --from-literal=password=topsecret
secret/my-secret created
```
```shell
pradeep@learnk8s$ kubectl describe secrets my-secret
Name:         my-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  9 bytes
user:      7 bytes
```
Now, let us consume this secret as a volume in a Pod.

```yaml
pradeep@learnk8s$ cat pod-with-secret-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-volume
spec:
  containers:
  - name: pod-with-secret-volume
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: my-secret
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-secret-volume.yaml
pod/pod-with-secret-volume created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running            0                 81m
pod-with-hostpath-volume    1/1     Running            0                 55m
pod-with-secret-volume      1/1     Running            0                 15s
```
```shell
pradeep@learnk8s$ kubectl describe pod pod-with-secret-volume
Name:         pod-with-secret-volume
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Sun, 20 Feb 2022 19:52:10 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.42
IPs:
  IP:  10.244.1.42
Containers:
  pod-with-secret-volume:
    Container ID:   docker://a21403023c1bb4d5348f1aa1eee282072394e3c6785a8f0ef1203f67355f357b
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:0d9c9aed1eb385336db0bc9b976b6b49774aee3d2b9c2788a0d0d9e239986cb3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 20 Feb 2022 19:52:24 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/foo from foo (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wlmjs (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  foo:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  my-secret
    Optional:    false
  kube-api-access-wlmjs:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  93s   default-scheduler  Successfully assigned default/pod-with-secret-volume to k8s-m02
  Normal  Pulling    91s   kubelet            Pulling image "redis"
  Normal  Pulled     80s   kubelet            Successfully pulled image "redis" in 11.292924683s
  Normal  Created    80s   kubelet            Created container pod-with-secret-volume
  Normal  Started    79s   kubelet            Started container pod-with-secret-volume
```
```shell
pradeep@learnk8s$ kubectl exec -it pod-with-secret-volume -- sh
# ls /etc/foo
password  user
# ls -l /etc/foo
total 0
lrwxrwxrwx 1 root root 15 Feb 20 14:22 password -> ..data/password
lrwxrwxrwx 1 root root 11 Feb 20 14:22 user -> ..data/user
#
# cat /etc/foo/user
pradeep#
# cat /etc/foo/password
topsecret#
# exit
```
One problem here is that the password is in plain text.
We can create another YAML defitinition with a slight modification.

```yaml
pradeep@learnk8s$ cat pod-with-secret-volume-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-volume-2
spec:
  containers:
  - name: pod-with-secret-volume-2
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: my-secret
      items:
        - key: user
          path: my-group/my-username
          mode: 0777
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-secret-volume-2.yaml
pod/pod-with-secret-volume-2 created
```
```shell
kubectl describe pod pod-with-secret-volume-2
Name:         pod-with-secret-volume-2
Namespace:    default
Priority:     0
Node:         k8s-m02/192.168.177.30
Start Time:   Sun, 20 Feb 2022 20:10:40 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.244.1.43
IPs:
  IP:  10.244.1.43
Containers:
  pod-with-secret-volume-2:
    Container ID:   docker://56198fa123177a7c913973c16e686d25df1eeafa0787eef39fbf5306ac7e0b1d
    Image:          redis
    Image ID:       docker-pullable://redis@sha256:0d9c9aed1eb385336db0bc9b976b6b49774aee3d2b9c2788a0d0d9e239986cb3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 20 Feb 2022 20:10:47 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/foo from foo (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-fwthr (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  foo:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  my-secret
    Optional:    false
  kube-api-access-fwthr:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  38s   default-scheduler  Successfully assigned default/pod-with-secret-volume-2 to k8s-m02
  Normal  Pulling    36s   kubelet            Pulling image "redis"
  Normal  Pulled     32s   kubelet            Successfully pulled image "redis" in 4.744969468s
  Normal  Created    32s   kubelet            Created container pod-with-secret-volume-2
  Normal  Started    31s   kubelet            Started container pod-with-secret-volume-2
```
```shell
pradeep@learnk8s$ kubectl exec -it pod-with-secret-volume-2 -- sh
# ls /etc/foo
my-group
# ls -l /etc/foo
total 0
lrwxrwxrwx 1 root root 15 Feb 20 14:40 my-group -> ..data/my-group
# cat /etc/foo/my-group/my-username
pradeep# at /etc/foo/my-group/my-password
sh: 9: at: not found
```
Only username is exposed while the password is not projected becuase the key is not exposed.

```yaml
pradeep@learnk8s$ cat pod-with-secret-volume-3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-volume-3
spec:
  containers:
  - name: pod-with-secret-volume-3
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: my-secret
      items:
        - key: user
          mode: 0777
          path: my-group/my-username
        - key: password
          mode: 0777
          path: my-group/my-pass
```
```shell
pradeep@learnk8s$ kubectl create -f pod-with-secret-volume-3.yaml
pod/pod-with-secret-volume-3 created
```
```shell
pradeep@learnk8s$ kubectl get pods | grep volume
pod-with-emptydir-volume    1/1     Running            0               110m
pod-with-hostpath-volume    1/1     Running            0               84m
pod-with-secret-volume      1/1     Running            0               29m
pod-with-secret-volume-2    1/1     Running            0               3m40s
pod-with-secret-volume-3    1/1     Running            0               16s
```
```shell
pradeep@learnk8s$ kubectl exec -it pod-with-secret-volume-3 sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ls -l /etc/foo
total 0
lrwxrwxrwx 1 root root 15 Feb 20 14:51 my-group -> ..data/my-group
# cat /etc/foo/my-group/my-username
pradeep#
# cat /etc/foo/my-group/my-pass
topsecret#
# exit
```
