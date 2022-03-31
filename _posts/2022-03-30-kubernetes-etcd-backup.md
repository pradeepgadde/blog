---
layout: single
title:  "Kubernetes Cluster ETCD Backup"
date:   2022-03-30 15:55:04 +0530
categories: Kubernetes
tags: kubeadm
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/etcd.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# ETCD Backup



Let us take a look at all the current running resources in our cluster.

```sh
lab@k8s1:~$ kubectl get all -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
default       pod/web-79d88c97d6-4w4nl           1/1     Running   0          12h
default       pod/web-79d88c97d6-7ktck           1/1     Running   0          12h
default       pod/web-79d88c97d6-8tvsk           1/1     Running   0          12h
default       pod/web-79d88c97d6-96sd9           1/1     Running   0          12h
default       pod/web-79d88c97d6-9tzlh           1/1     Running   0          12h
default       pod/web-79d88c97d6-brtgx           1/1     Running   0          12h
default       pod/web-79d88c97d6-kngc4           1/1     Running   0          12h
default       pod/web-79d88c97d6-p5vfg           1/1     Running   0          12h
default       pod/web-79d88c97d6-rbhpr           1/1     Running   0          12h
kube-system   pod/coredns-78fcd69978-cvpx2       1/1     Running   0          13h
kube-system   pod/coredns-78fcd69978-hf5sj       1/1     Running   0          12h
kube-system   pod/etcd-k8s1                      1/1     Running   1          17h
kube-system   pod/kube-apiserver-k8s1            1/1     Running   1          17h
kube-system   pod/kube-controller-manager-k8s1   1/1     Running   0          17h
kube-system   pod/kube-flannel-ds-lhcwb          1/1     Running   0          16h
kube-system   pod/kube-flannel-ds-ph9gg          1/1     Running   0          15h
kube-system   pod/kube-flannel-ds-xm28z          1/1     Running   0          14h
kube-system   pod/kube-proxy-brrvs               1/1     Running   0          17h
kube-system   pod/kube-proxy-cdl2t               1/1     Running   0          15h
kube-system   pod/kube-proxy-v8r74               1/1     Running   0          14h
kube-system   pod/kube-scheduler-k8s1            1/1     Running   1          17h

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  17h
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   17h

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-flannel-ds   3         3         3       3            3           <none>                   16h
kube-system   daemonset.apps/kube-proxy        3         3         3       3            3           kubernetes.io/os=linux   17h

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/web       9/9     9            9           13h
kube-system   deployment.apps/coredns   2/2     2            2           17h

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/web-79d88c97d6       9         9         9       13h
kube-system   replicaset.apps/coredns-78fcd69978   2         2         2       17h
lab@k8s1:~$
```



There are two deployments, two replicates, two daemonsets, two services, and 21 pods in total. We can take a backup of this database (etcd) and restore it if needed.



First, let us check the ETCD version that is currently in use in the cluster.

```sh
lab@k8s1:~$ kubectl -n kube-system describe pods etcd-k8s1
Name:                 etcd-k8s1
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 k8s1/10.210.40.172
Start Time:           Tue, 29 Mar 2022 11:14:08 -0700
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.100.1:2379
                      kubernetes.io/config.hash: 91e64623622aeb865a09c79cf82eb4a5
                      kubernetes.io/config.mirror: 91e64623622aeb865a09c79cf82eb4a5
                      kubernetes.io/config.seen: 2022-03-29T07:16:25.772358879-07:00
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   10.210.40.172
IPs:
  IP:           10.210.40.172
Controlled By:  Node/k8s1
Containers:
  etcd:
    Container ID:  docker://8d902a4981fbf57e35c080db6f69608ba6e8e40596409358a75ffe8f6b85e78d
    Image:         k8s.gcr.io/etcd:3.5.0-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:9ce33ba33d8e738a5b85ed50b5080ac746deceed4a7496c550927a7a19ca3b6d
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.100.1:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://192.168.100.1:2380
      --initial-cluster=k8s1=https://192.168.100.1:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.1:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.100.1:2380
      --name=k8s1
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Tue, 29 Mar 2022 07:16:17 -0700
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>
lab@k8s1:~$
```



From the Image section, we can see `k8s.gcr.io/etcd:3.5.0-0` and ETCD is listening on port `2379`.

ETCD Server certificate file is stored at `--cert-file=/etc/kubernetes/pki/etcd/server.crt` and CA Certificate is at `--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt`.



To work with ETCD, we need an utility called `etcdctl`.

```sh
lab@k8s1:~$ etcdctl -h

Command 'etcdctl' not found, but can be installed with:

sudo apt install etcd-client
lab@k8s1:~$ 
```
Let us install it now.
```sh
lab@k8s1:~$ sudo apt install etcd-client
[sudo] password for lab:
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  libfwupdplugin1
Use 'sudo apt autoremove' to remove it.
The following NEW packages will be installed:
  etcd-client
0 upgraded, 1 newly installed, 0 to remove and 7 not upgraded.
Need to get 4563 kB of archives.
After this operation, 17.2 MB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 etcd-client amd64 3.2.26+dfsg-6 [4563 kB]
Fetched 4563 kB in 1s (4735 kB/s)
Selecting previously unselected package etcd-client.
(Reading database ... 216135 files and directories currently installed.)
Preparing to unpack .../etcd-client_3.2.26+dfsg-6_amd64.deb ...
Unpacking etcd-client (3.2.26+dfsg-6) ...
Setting up etcd-client (3.2.26+dfsg-6) ...
Processing triggers for man-db (2.9.1-1) ...
lab@k8s1:~$

```

Let us take a snapshot of the ETCD database, using the builtin snapshot functionality, we need to specify a location.

```sh
lab@k8s1:~$ etcdctl -h
NAME:
   etcdctl - A simple command line client for etcd.

WARNING:
   Environment variable ETCDCTL_API is not set; defaults to etcdctl v2.
   Set environment variable ETCDCTL_API=3 to use v3 API or ETCDCTL_API=2 to use v2 API.

USAGE:
   etcdctl [global options] command [command options] [arguments...]

VERSION:
   3.2.26

COMMANDS:
   backup          backup an etcd directory
   cluster-health  check the health of the etcd cluster
   mk              make a new key with a given value
   mkdir           make a new directory
   rm              remove a key or a directory
   rmdir           removes the key if it is an empty directory or a key-value pair
   get             retrieve the value of a key
   ls              retrieve a directory
   set             set the value of a key
   setdir          create a new directory or update an existing directory TTL
   update          update an existing key with a given value
   updatedir       update an existing directory
   watch           watch a key for changes
   exec-watch      watch a key for changes and exec an executable
   member          member add, remove and list subcommands
   user            user add, grant and revoke subcommands
   role            role add, grant and revoke subcommands
   auth            overall auth controls
   help, h         Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                          output cURL commands which can be used to reproduce the request
   --no-sync                        don't synchronize cluster information before sending request
   --output simple, -o simple       output response in the given format (simple, `extended` or `json`) (default: "simple")
   --discovery-srv value, -D value  domain name to query for SRV records describing cluster endpoints
   --insecure-discovery             accept insecure SRV records describing cluster endpoints
   --peers value, -C value          DEPRECATED - "--endpoints" should be used instead
   --endpoint value                 DEPRECATED - "--endpoints" should be used instead
   --endpoints value                a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:2379,http://127.0.0.1:4001")
   --cert-file value                identify HTTPS client using this SSL certificate file
   --key-file value                 identify HTTPS client using this SSL key file
   --ca-file value                  verify certificates of HTTPS-enabled servers using this CA bundle
   --username value, -u value       provide username[:password] and prompt if password is not supplied.
   --timeout value                  connection timeout per request (default: 2s)
   --total-timeout value            timeout for the command execution (except watch) (default: 5s)
   --help, -h                       show help
   --version, -v                    print the version
lab@k8s1:~$
```
As seen in the WARNING, we need to set environment variable `ETCDCTL_API=3` while working with v3.

Lets explore the `snapshot` options

```sh
lab@k8s1:~$ ETCDCTL_API=3 etcdctl snapshot -h
NAME:
	snapshot - Manages etcd node snapshots

USAGE:
	etcdctl snapshot <subcommand> [flags]

API VERSION:
	3.2


COMMANDS:
	restore	Restores an etcd member snapshot to an etcd directory
	save	Stores an etcd node backend snapshot to a given file
	status	Gets backend snapshot status of a given file

OPTIONS:
  -h, --help[=false]	help for snapshot

GLOBAL OPTIONS:
      --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""					identify secure client using this TLS certificate file
      --command-timeout=5s			timeout for short running command (excluding dial timeout)
      --debug[=false]				enable client-side debug logging
      --dial-timeout=2s				dial timeout for client connections
      --endpoints=[127.0.0.1:2379]		gRPC endpoints
      --hex[=false]				print byte strings as hex encoded strings
      --insecure-skip-tls-verify[=false]	skip server certificate verification
      --insecure-transport[=true]		disable transport security for client connections
      --key=""					identify secure client using this TLS key file
      --user=""					username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"			set the output format (fields, json, protobuf, simple, table)

lab@k8s1:~$
```

We can take a snapshot by specifying the endpoint, certificates etc as shown below:

```shell
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
```
where `trusted-ca-file`, `cert-file` and `key-file` can be obtained from the description of the `etcd` Pod.
```sh
lab@k8s1:~$ sudo ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/snapshot-backup.db
[sudo] password for lab:
Snapshot saved at /opt/snapshot-backup.db
lab@k8s1:~$
```

Verify the snapshot status



```sh
lab@k8s1:~$ sudo ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status /opt/snapshot-backup.db
2140355a, 91968, 1163, 2.2 MB
lab@k8s1:~$
```



Same output in a table fashion, with `--write-out=table` option

```sh
lab@k8s1:~$ sudo ETCDCTL_API=3 etcdctl --endpoints=127.0.0.1:2379 --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot status /opt/snapshot-backup.db --write-out=table
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 2140355a |    91968 |       1163 |     2.2 MB |
+----------+----------+------------+------------+
lab@k8s1:~$
```



To mimic some crash of this cluster, let us delete the deployment (and all associated resources)

```sh
lab@k8s1:~$ kubectl delete deployment web
deployment.apps "web" deleted

```

```sh
lab@k8s1:~$ kubectl get pods
NAME                   READY   STATUS        RESTARTS   AGE
web-79d88c97d6-p5vfg   1/1     Terminating   0          14h
lab@k8s1:~$ kubectl get pods
No resources found in default namespace.
lab@k8s1:~$
```

So, at the moment, we have No resources found in default namespace.

How can we restore this cluster? Luckily, we have a snapshot, right? Lets use it to restore.



```sh
lab@k8s1:~$ sudo ETCDCTL_API=3 etcdctl snapshot restore --data-dir=/var/lib/etcd-old-copy /opt/snapshot-backup.db
[sudo] password for lab:
Error:  expected sha256 [121 49 82 41 127 17 108 224 104 202 28 46 0 145 49 221 234 71 99 210 185 176 191 126 38 217 143 170 228 223 79 127], got [84 157 250 180 184 168 7 99 44 43 8 22 51 253 10 140 54 78 152 222 239 28 99 93 236 125 236 175 74 88 153 54]
lab@k8s1:~$
```

Though, there is some error related to checksum (I am not sure why this error!), snapshot seems to be copied to the location given in `--data-dir`.

```sh
lab@k8s1:~$ sudo ls -la /var/lib/etcd-old-copy/member/snap/db
-rw------- 1 root root 2203648 Mar 30 02:17 /var/lib/etcd-old-copy/member/snap/db
lab@k8s1:~$
```

Just this restore option alone will not help, as seen below.

```sh
lab@k8s1:~$ kubectl get pods
No resources found in default namespace.
lab@k8s1:~$
```



We need to update the ETCD config to use this restored directory (`/var/lib/etcd-old-copy` in our case)

As we know, etcd is a static pod, we can find the manifest files in `/etc/kubernetes/manifests/` folder.

```sh
lab@k8s1:~$ sudo vi /etc/kubernetes/manifests/etcd.yaml
lab@k8s1:~$ sudo cat /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.100.1:2379
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.100.1:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.100.1:2380
    - --initial-cluster=k8s1=https://192.168.100.1:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.1:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.100.1:2380
    - --name=k8s1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.5.0-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-old-copy
      type: DirectoryOrCreate
    name: etcd-data
status: {}
lab@k8s1:~$
```

We have changed the hostPath of the  `volume` named `etch-data` with a new value of `/var/lib/etcd-old-copy`.

Just make this one change, save the manifest file.

We can see that, all the pods from the web deployment are back!



```sh
lab@k8s1:~$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP            NODE   NOMINATED NODE   READINESS GATES
web-79d88c97d6-4w4nl   1/1     Running   0          15h   10.244.2.14   k8s2   <none>           <none>
web-79d88c97d6-7ktck   1/1     Running   0          15h   10.244.2.15   k8s2   <none>           <none>
web-79d88c97d6-8tvsk   1/1     Running   0          14h   10.244.3.10   k8s3   <none>           <none>
web-79d88c97d6-96sd9   1/1     Running   0          15h   10.244.2.12   k8s2   <none>           <none>
web-79d88c97d6-9tzlh   1/1     Running   0          15h   10.244.2.11   k8s2   <none>           <none>
web-79d88c97d6-brtgx   1/1     Running   0          14h   10.244.3.9    k8s3   <none>           <none>
web-79d88c97d6-kngc4   1/1     Running   0          15h   10.244.2.10   k8s2   <none>           <none>
web-79d88c97d6-p5vfg   1/1     Running   0          15h   10.244.2.13   k8s2   <none>           <none>
web-79d88c97d6-rbhpr   1/1     Running   0          14h   10.244.3.11   k8s3   <none>           <none>
lab@k8s1:~$
```



After some time, I have verified all resources.



```sh
lab@k8s1:~$ kubectl get deploy
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    9/9     9            9           16h
lab@k8s1:~$ kubectl get all -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
default       pod/web-79d88c97d6-4w4nl           1/1     Running   0          15h
default       pod/web-79d88c97d6-7ktck           1/1     Running   0          15h
default       pod/web-79d88c97d6-8tvsk           1/1     Running   0          15h
default       pod/web-79d88c97d6-96sd9           1/1     Running   0          15h
default       pod/web-79d88c97d6-9tzlh           1/1     Running   0          15h
default       pod/web-79d88c97d6-brtgx           1/1     Running   0          15h
default       pod/web-79d88c97d6-kngc4           1/1     Running   0          15h
default       pod/web-79d88c97d6-p5vfg           1/1     Running   0          15h
default       pod/web-79d88c97d6-rbhpr           1/1     Running   0          15h
kube-system   pod/coredns-78fcd69978-cvpx2       1/1     Running   0          15h
kube-system   pod/coredns-78fcd69978-hf5sj       1/1     Running   0          15h
kube-system   pod/etcd-k8s1                      1/1     Running   1          20h
kube-system   pod/kube-apiserver-k8s1            1/1     Running   1          20h
kube-system   pod/kube-controller-manager-k8s1   1/1     Running   0          20h
kube-system   pod/kube-flannel-ds-lhcwb          1/1     Running   0          19h
kube-system   pod/kube-flannel-ds-ph9gg          1/1     Running   0          18h
kube-system   pod/kube-flannel-ds-xm28z          1/1     Running   0          17h
kube-system   pod/kube-proxy-brrvs               1/1     Running   0          20h
kube-system   pod/kube-proxy-cdl2t               1/1     Running   0          18h
kube-system   pod/kube-proxy-v8r74               1/1     Running   0          17h
kube-system   pod/kube-scheduler-k8s1            1/1     Running   1          20h

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  20h
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   20h

NAMESPACE     NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-flannel-ds   3         3         3       3            3           <none>                   19h
kube-system   daemonset.apps/kube-proxy        3         3         3       3            3           kubernetes.io/os=linux   20h

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/web       9/9     9            9           16h
kube-system   deployment.apps/coredns   2/2     2            2           20h

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/web-79d88c97d6       9         9         9       16h
kube-system   replicaset.apps/coredns-78fcd69978   2         2         2       20h
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl -n kube-system describe pods etcd-k8s1
Name:                 etcd-k8s1
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 k8s1/10.210.40.172
Start Time:           Tue, 29 Mar 2022 11:14:08 -0700
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.100.1:2379
                      kubernetes.io/config.hash: 91e64623622aeb865a09c79cf82eb4a5
                      kubernetes.io/config.mirror: 91e64623622aeb865a09c79cf82eb4a5
                      kubernetes.io/config.seen: 2022-03-29T07:16:25.772358879-07:00
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   10.210.40.172
IPs:
  IP:           10.210.40.172
Controlled By:  Node/k8s1
Containers:
  etcd:
    Container ID:  docker://8d902a4981fbf57e35c080db6f69608ba6e8e40596409358a75ffe8f6b85e78d
    Image:         k8s.gcr.io/etcd:3.5.0-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:9ce33ba33d8e738a5b85ed50b5080ac746deceed4a7496c550927a7a19ca3b6d
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.100.1:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://192.168.100.1:2380
      --initial-cluster=k8s1=https://192.168.100.1:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.1:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.100.1:2380
      --name=k8s1
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Tue, 29 Mar 2022 07:16:17 -0700
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>
lab@k8s1:~$
```



I was expecting the updated path for the `  etcd-data` in the above output, but still it shows the original path `/var/lib/etcd`, though YAML manifest file is changed.

This concludes our discussion on ETCD Backup.



