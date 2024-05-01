---

layout: single
title:  "GKE Services"
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
# GKE Services Ingress
```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-f1b2097fd2bf.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ export my_zone=europe-west4-c
export my_cluster=standard-cluster-1
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ source <(kubectl completion bash)
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 64726, done.
remote: Counting objects: 100% (185/185), done.
remote: Compressing objects: 100% (110/110), done.
remote: Total 64726 (delta 98), reused 143 (delta 73), pack-reused 64541
Receiving objects: 100% (64726/64726), 697.99 MiB | 10.57 MiB/s, done.
Resolving deltas: 100% (41299/41299), done.
Updating files: 100% (12864/12864), done.
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_00_28c782da3114@cloudshell:~ (qwiklabs-gcp-04-f1b2097fd2bf)$ cd ~/ak8s/GKE_Services/
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ ls
dns-demo.yaml  hello-ingress.yaml  hello-lb-svc.yaml  hello-nodeport-svc.yaml  hello-svc.yaml  hello-v1.yaml  hello-v2.yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat dns-demo.yaml 
apiVersion: v1
kind: Service
metadata:
  name: dns-demo
spec:
  selector:
    name: dns-demo
  clusterIP: None
  ports:
  - name: dns-demo
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: dns-demo-1
  labels:
    name: dns-demo
spec:
  hostname: dns-demo-1
  subdomain: dns-demo
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: dns-demo-2
  labels:
    name: dns-demo
spec:
  hostname: dns-demo-2
  subdomain: dns-demo
  containers:
  - name: nginx
    image: nginx
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl apply -f dns-demo.yaml
service/dns-demo created
pod/dns-demo-1 created
pod/dns-demo-2 created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get pods
NAME         READY   STATUS              RESTARTS   AGE
dns-demo-1   0/1     ContainerCreating   0          6s
dns-demo-2   0/1     ContainerCreating   0          5s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
dns-demo-1   1/1     Running   0          9s
dns-demo-2   1/1     Running   0          8s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE                                                  NOMINATED NODE   READINESS GATES
dns-demo-1   1/1     Running   0          14s   10.140.1.6    gke-standard-cluster-standard-cluster-9c3fe34a-n2bh   <none>           <none>
dns-demo-2   1/1     Running   0          13s   10.140.0.14   gke-standard-cluster-standard-cluster-9c3fe34a-9tq6   <none>           <none>
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl exec -it dns-demo-1 -- /bin/bash
root@dns-demo-1:/# apt-get update
apt-get install -y iputils-ping
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:4 http://deb.debian.org/debian bookworm/main amd64 Packages [8786 kB]
Get:5 http://deb.debian.org/debian bookworm-updates/main amd64 Packages [13.8 kB]
Get:6 http://deb.debian.org/debian-security bookworm-security/main amd64 Packages [155 kB]
Fetched 9209 kB in 2s (5227 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libcap2-bin libpam-cap
The following NEW packages will be installed:
  iputils-ping libcap2-bin libpam-cap
0 upgraded, 3 newly installed, 0 to remove and 2 not upgraded.
Need to get 96.2 kB of archives.
After this operation, 311 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 libcap2-bin amd64 1:2.66-4 [34.7 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 iputils-ping amd64 3:20221126-1 [47.1 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 libpam-cap amd64 1:2.66-4 [14.5 kB]
Fetched 96.2 kB in 0s (1677 kB/s)       
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libcap2-bin.
(Reading database ... 7582 files and directories currently installed.)
Preparing to unpack .../libcap2-bin_1%3a2.66-4_amd64.deb ...
Unpacking libcap2-bin (1:2.66-4) ...
Selecting previously unselected package iputils-ping.
Preparing to unpack .../iputils-ping_3%3a20221126-1_amd64.deb ...
Unpacking iputils-ping (3:20221126-1) ...
Selecting previously unselected package libpam-cap:amd64.
Preparing to unpack .../libpam-cap_1%3a2.66-4_amd64.deb ...
Unpacking libpam-cap:amd64 (1:2.66-4) ...
Setting up libcap2-bin (1:2.66-4) ...
Setting up libpam-cap:amd64 (1:2.66-4) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 78.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.36.0 /usr/local/share/perl/5.36.0 /usr/lib/x86_64-linux-gnu/perl5/5.36 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.36 /usr/share/perl/5.36 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up iputils-ping (3:20221126-1) ...
root@dns-demo-1:/# ping dns-demo-2.dns-demo.default.svc.cluster.local
PING dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14) 56(84) bytes of data.
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=1 ttl=62 time=1.01 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=2 ttl=62 time=0.300 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=3 ttl=62 time=0.299 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=4 ttl=62 time=0.288 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=5 ttl=62 time=0.305 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=6 ttl=62 time=0.298 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=7 ttl=62 time=0.302 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=8 ttl=62 time=0.325 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=9 ttl=62 time=0.354 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=10 ttl=62 time=0.310 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=11 ttl=62 time=0.298 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=12 ttl=62 time=0.290 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=13 ttl=62 time=0.280 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=14 ttl=62 time=0.335 ms
64 bytes from dns-demo-2.dns-demo.default.svc.cluster.local (10.140.0.14): icmp_seq=15 ttl=62 time=0.363 ms
^C
--- dns-demo-2.dns-demo.default.svc.cluster.local ping statistics ---
15 packets transmitted, 15 received, 0% packet loss, time 14241ms
rtt min/avg/max/mdev = 0.280/0.357/1.009/0.175 ms
root@dns-demo-1:/# ping ping dns-demo.default.svc.cluster.local
ping: ping: No address associated with hostname
root@dns-demo-1:/# ping dns-demo.default.svc.cluster.local
PING dns-demo.default.svc.cluster.local (10.140.1.6) 56(84) bytes of data.
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=1 ttl=64 time=0.114 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=2 ttl=64 time=0.036 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=3 ttl=64 time=0.038 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=4 ttl=64 time=0.037 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=5 ttl=64 time=0.036 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=6 ttl=64 time=0.043 ms
64 bytes from dns-demo-1.dns-demo.default.svc.cluster.local (10.140.1.6): icmp_seq=7 ttl=64 time=0.037 ms
^C
--- dns-demo.default.svc.cluster.local ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6154ms
rtt min/avg/max/mdev = 0.036/0.048/0.114/0.026 ms
root@dns-demo-1:/# exit
exit
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
dns-demo     ClusterIP   None           <none>        1234/TCP   2m23s
kubernetes   ClusterIP   10.180.160.1   <none>        443/TCP    16h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get ep
NAME         ENDPOINTS                          AGE
dns-demo     10.140.0.14:1234,10.140.1.6:1234   2m29s
kubernetes   10.10.0.2:443                      16h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP            NODE                                                  NOMINATED NODE   READINESS GATES
dns-demo-1   1/1     Running   0          2m42s   10.140.1.6    gke-standard-cluster-standard-cluster-9c3fe34a-n2bh   <none>           <none>
dns-demo-2   1/1     Running   0          2m41s   10.140.0.14   gke-standard-cluster-standard-cluster-9c3fe34a-9tq6   <none>           <none>
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ ls
dns-demo.yaml  hello-ingress.yaml  hello-lb-svc.yaml  hello-nodeport-svc.yaml  hello-svc.yaml  hello-v1.yaml  hello-v2.yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```

```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cd ~/ak8s/GKE_Services/
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-v1.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      run: hello-v1
  template:
    metadata:
      labels:
        run: hello-v1
        name: hello-v1
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-v1
        ports:
        - containerPort: 8080
          protocol: TCP
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl create -f hello-v1.yaml
deployment.apps/hello-v1 created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
hello-v1   3/3     3            3           11s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```

```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  type: ClusterIP
  selector:
    name: hello-v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get pods --show-labels
NAME                        READY   STATUS    RESTARTS   AGE     LABELS
dns-demo-1                  1/1     Running   0          4m20s   name=dns-demo
dns-demo-2                  1/1     Running   0          4m19s   name=dns-demo
hello-v1-848b87d68c-bddhj   1/1     Running   0          59s     name=hello-v1,pod-template-hash=848b87d68c,run=hello-v1
hello-v1-848b87d68c-mq2tm   1/1     Running   0          59s     name=hello-v1,pod-template-hash=848b87d68c,run=hello-v1
hello-v1-848b87d68c-xsfqn   1/1     Running   0          59s     name=hello-v1,pod-template-hash=848b87d68c,run=hello-v1
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl apply -f ./hello-svc.yaml
service/hello-svc created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
dns-demo     ClusterIP   None             <none>        1234/TCP   4m38s
hello-svc    ClusterIP   10.180.171.157   <none>        80/TCP     5s
kubernetes   ClusterIP   10.180.160.1     <none>        443/TCP    16h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get service hello-svc
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
hello-svc   ClusterIP   10.180.171.157   <none>        80/TCP    17s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl hello-svc.default.svc.cluster.local
curl: (6) Could not resolve host: hello-svc.default.svc.cluster.local
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl hello-svc.default.svc.cluster.local
curl: (6) Could not resolve host: hello-svc.default.svc.cluster.local
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl exec -it dns-demo-1 -- /bin/bash
root@dns-demo-1:/# apt-get install -y curl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (7.88.1-10+deb12u5).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
root@dns-demo-1:/# curl hello-svc.default.svc.cluster.local
Hello, world!
Version: 1.0.0
Hostname: hello-v1-848b87d68c-xsfqn
root@dns-demo-1:/# exit
exit
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ ls
dns-demo.yaml  hello-ingress.yaml  hello-lb-svc.yaml  hello-nodeport-svc.yaml  hello-svc.yaml  hello-v1.yaml  hello-v2.yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-nodeport-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  type: NodePort
  selector:
    name: hello-v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30100
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```

```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl apply -f ./hello-nodeport-svc.yaml
service/hello-svc configured
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dns-demo     ClusterIP   None             <none>        1234/TCP       7m2s
hello-svc    NodePort    10.180.171.157   <none>        80:30100/TCP   2m29s
kubernetes   ClusterIP   10.180.160.1     <none>        443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get service hello-svc
NAME        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hello-svc   NodePort   10.180.171.157   <none>        80:30100/TCP   2m43s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl hello-svc.default.svc.cluster.local
curl: (6) Could not resolve host: hello-svc.default.svc.cluster.local
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl exec -it dns-demo-1 -- /bin/bash
root@dns-demo-1:/# curl hello-svc.default.svc.cluster.local
Hello, world!
Version: 1.0.0
Hostname: hello-v1-848b87d68c-mq2tm
root@dns-demo-1:/# exit
exit
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ ls
dns-demo.yaml  hello-ingress.yaml  hello-lb-svc.yaml  hello-nodeport-svc.yaml  hello-svc.yaml  hello-v1.yaml  hello-v2.yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-v2.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      run: hello-v2
  template:
    metadata:
      labels:
        run: hello-v2
        name: hello-v2
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        name: hello-v2
        ports:
        - containerPort: 8080
          protocol: TCP
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl create -f hello-v2.yaml
deployment.apps/hello-v2 created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
hello-v1   3/3     3            3           21m
hello-v2   3/3     3            3           7s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ export STATIC_LB=$(gcloud compute addresses describe regional-loadbalancer --region europe-west4 --format json | jq -r '.address')
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ sed -i "s/10\.10\.10\.10/$STATIC_LB/g" hello-lb-svc.yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-lb-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: hello-lb-svc
spec:
  type: LoadBalancer
  loadBalancerIP: 35.204.46.116
  selector:
    name: hello-v2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl apply -f ./hello-lb-svc.yaml
service/hello-lb-svc created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dns-demo       ClusterIP      None             <none>        1234/TCP       26m
hello-lb-svc   LoadBalancer   10.180.170.88    <pending>     80:31864/TCP   9s
hello-svc      NodePort       10.180.171.157   <none>        80:30100/TCP   21m
kubernetes     ClusterIP      10.180.160.1     <none>        443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dns-demo       ClusterIP      None             <none>        1234/TCP       26m
hello-lb-svc   LoadBalancer   10.180.170.88    <pending>     80:31864/TCP   14s
hello-svc      NodePort       10.180.171.157   <none>        80:30100/TCP   21m
kubernetes     ClusterIP      10.180.160.1     <none>        443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dns-demo       ClusterIP      None             <none>        1234/TCP       26m
hello-lb-svc   LoadBalancer   10.180.170.88    <pending>     80:31864/TCP   17s
hello-svc      NodePort       10.180.171.157   <none>        80:30100/TCP   21m
kubernetes     ClusterIP      10.180.160.1     <none>        443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
dns-demo       ClusterIP      None             <none>        1234/TCP       26m
hello-lb-svc   LoadBalancer   10.180.170.88    <pending>     80:31864/TCP   39s
hello-svc      NodePort       10.180.171.157   <none>        80:30100/TCP   22m
kubernetes     ClusterIP      10.180.160.1     <none>        443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get services
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
dns-demo       ClusterIP      None             <none>          1234/TCP       27m
hello-lb-svc   LoadBalancer   10.180.170.88    35.204.46.116   80:31864/TCP   56s
hello-svc      NodePort       10.180.171.157   <none>          80:30100/TCP   22m
kubernetes     ClusterIP      10.180.160.1     <none>          443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ 
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl hello-lb-svc.default.svc.cluster.local
curl: (6) Could not resolve host: hello-lb-svc.default.svc.cluster.local
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl 35.204.46.116
Hello, world!
Version: 2.0.0
Hostname: hello-v2-bbb49f8b9-fkfz6
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl exec -it dns-demo-1 -- /bin/bash
root@dns-demo-1:/# curl hello-lb-svc.default.svc.cluster.local
Hello, world!
Version: 2.0.0
Hostname: hello-v2-bbb49f8b9-2bf4d
root@dns-demo-1:/# exit
exit
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl exec -it dns-demo-1 -- /bin/bash
root@dns-demo-1:/# curl 35.204.46.116
Hello, world!
Version: 2.0.0
Hostname: hello-v2-bbb49f8b9-2gt5j
root@dns-demo-1:/# exit
exit
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
dns-demo       ClusterIP      None             <none>          1234/TCP       29m
hello-lb-svc   LoadBalancer   10.180.170.88    35.204.46.116   80:31864/TCP   3m31s
hello-svc      NodePort       10.180.171.157   <none>          80:30100/TCP   25m
kubernetes     ClusterIP      10.180.160.1     <none>          443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
hello-v1   3/3     3            3           26m
hello-v2   3/3     3            3           4m49s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```
```yaml
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ cat hello-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.global-static-ip-name: "global-ingress"
spec:
  rules:
  - http:
      paths:
      - path: /v1
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-svc
            port: 
              number: 80
      - path: /v2
        pathType: ImplementationSpecific
        backend:
          service:
            name: hello-lb-svc
            port: 
              number: 80
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```

```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ 
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl apply -f hello-ingress.yaml
ingress.networking.k8s.io/hello-ingress created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
dns-demo       ClusterIP      None             <none>          1234/TCP       31m
hello-lb-svc   LoadBalancer   10.180.170.88    35.204.46.116   80:31864/TCP   4m57s
hello-svc      NodePort       10.180.171.157   <none>          80:30100/TCP   26m
kubernetes     ClusterIP      10.180.160.1     <none>          443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get ingress
NAME            CLASS    HOSTS   ADDRESS   PORTS   AGE
hello-ingress   <none>   *                 80      56s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl describe ingress
Name:             hello-ingress
Labels:           <none>
Namespace:        default
Address:          
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /v1   hello-svc:80 (10.140.0.15:8080,10.140.1.7:8080,10.140.1.8:8080)
              /v2   hello-lb-svc:80 (10.140.0.16:8080,10.140.1.10:8080,10.140.1.9:8080)
Annotations:  kubernetes.io/ingress.global-static-ip-name: global-ingress
              nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age                    From                     Message
  ----    ------  ----                   ----                     -------
  Normal  Sync    2m36s (x2 over 2m36s)  loadbalancer-controller  Scheduled for sync
  Normal  Sync    7s                     loadbalancer-controller  UrlMap "k8s2-um-iju7yyd7-default-hello-ingress-t5vhxuju" created
  Normal  Sync    4s                     loadbalancer-controller  TargetProxy "k8s2-tp-iju7yyd7-default-hello-ingress-t5vhxuju" created
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get svc                                       
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
dns-demo       ClusterIP      None             <none>          1234/TCP       33m
hello-lb-svc   LoadBalancer   10.180.170.88    35.204.46.116   80:31864/TCP   6m58s
hello-svc      NodePort       10.180.171.157   <none>          80:30100/TCP   28m
kubernetes     ClusterIP      10.180.160.1     <none>          443/TCP        17h
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl get ingress                                   
NAME            CLASS    HOSTS   ADDRESS        PORTS   AGE
hello-ingress   <none>   *       34.120.38.19   80      2m59s
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ 
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ kubectl describe ingress hello-ingress
Name:             hello-ingress
Labels:           <none>
Namespace:        default
Address:          34.120.38.19
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /v1   hello-svc:80 (10.140.0.15:8080,10.140.1.7:8080,10.140.1.8:8080)
              /v2   hello-lb-svc:80 (10.140.0.16:8080,10.140.1.10:8080,10.140.1.9:8080)
Annotations:  ingress.kubernetes.io/backends:
                {"k8s1-bc63e6a2-default-hello-lb-svc-80-7c51ad2e":"HEALTHY","k8s1-bc63e6a2-default-hello-svc-80-535daa55":"HEALTHY","k8s1-bc63e6a2-kube-sy...
              ingress.kubernetes.io/forwarding-rule: k8s2-fr-iju7yyd7-default-hello-ingress-t5vhxuju
              ingress.kubernetes.io/target-proxy: k8s2-tp-iju7yyd7-default-hello-ingress-t5vhxuju
              ingress.kubernetes.io/url-map: k8s2-um-iju7yyd7-default-hello-ingress-t5vhxuju
              kubernetes.io/ingress.global-static-ip-name: global-ingress
              nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason     Age                  From                     Message
  ----    ------     ----                 ----                     -------
  Normal  Sync       87s                  loadbalancer-controller  UrlMap "k8s2-um-iju7yyd7-default-hello-ingress-t5vhxuju" created
  Normal  Sync       84s                  loadbalancer-controller  TargetProxy "k8s2-tp-iju7yyd7-default-hello-ingress-t5vhxuju" created
  Normal  Sync       66s                  loadbalancer-controller  ForwardingRule "k8s2-fr-iju7yyd7-default-hello-ingress-t5vhxuju" created
  Normal  IPChanged  66s                  loadbalancer-controller  IP is now 34.120.38.19
  Normal  Sync       48s (x5 over 3m56s)  loadbalancer-controller  Scheduled for sync
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v1
curl: (52) Empty reply from server
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v1
curl: (56) Recv failure: Connection reset by peer
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v1
curl: (52) Empty reply from server
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v2
curl: (52) Empty reply from server
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v2
curl: (56) Recv failure: Connection reset by peer
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v1
curl: (52) Empty reply from server
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v1
Hello, world!
Version: 1.0.0
Hostname: hello-v1-848b87d68c-xsfqn
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ curl http://34.120.38.19/v2
Hello, world!
Version: 2.0.0
Hostname: hello-v2-bbb49f8b9-fkfz6
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$
```

```sh
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ history 
    1  export my_zone=europe-west4-c
    2  export my_cluster=standard-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    5  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    6  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
    7  cd ~/ak8s/GKE_Services/
    8  ls
    9  cat dns-demo.yaml 
   10  kubectl apply -f dns-demo.yaml
   11  kubectl get pods
   12  kubectl get pods -o wide
   13  kubectl exec -it dns-demo-1 -- /bin/bash
   14  kubectl get svc
   15  kubectl get ep
   16  kubectl get pods -o wide
   17  ls
   18  cd ~/ak8s/GKE_Services/
   19  cat hello-v1.yaml 
   20  kubectl create -f hello-v1.yaml
   21  kubectl get deployments
   22  cat hello-svc.yaml 
   23  kubectl get pods -l
   24  kubectl get pods -L
   25  kubectl get pods --show-labels
   26  kubectl apply -f ./hello-svc.yaml
   27  kubectl get svc
   28  kubectl get service hello-svc
   29  curl hello-svc.default.svc.cluster.local
   30  kubectl exec -it dns-demo-1 -- /bin/bash
   31  ls
   32  cat hello-nodeport-svc.yaml 
   33  kubectl apply -f ./hello-nodeport-svc.yaml
   34  kubectl get svc
   35  kubectl get service hello-svc
   36  curl hello-svc.default.svc.cluster.local
   37  kubectl exec -it dns-demo-1 -- /bin/bash
   38  ls
   39  cat hello-v2.yaml 
   40  kubectl create -f hello-v2.yaml
   41  kubectl get deployments
   42  export STATIC_LB=$(gcloud compute addresses describe regional-loadbalancer --region europe-west4 --format json | jq -r '.address')
   43  sed -i "s/10\.10\.10\.10/$STATIC_LB/g" hello-lb-svc.yaml
   44  cat hello-lb-svc.yaml 
   45  kubectl apply -f ./hello-lb-svc.yaml
   46  kubectl get services
   47  curl hello-lb-svc.default.svc.cluster.local
   48  curl 35.204.46.116
   49  kubectl exec -it dns-demo-1 -- /bin/bash
   50  kubectl get svc
   51  kubectl get deploy
   52  cat hello-ingress.yaml 
   53  kubectl apply -f hello-ingress.yaml
   54  kubectl get svc
   55  kubectl get ingress
   56  kubectl describe ingress
   57  kubectl get svc
   58  kubectl get ingress
   59  kubectl describe ingress hello-ingress
   60  curl http://34.120.38.19/v1
   61  curl http://34.120.38.19/v2
   62  curl http://34.120.38.19/v1
   63  curl http://34.120.38.19/v2
   64  history 
student_00_28c782da3114@cloudshell:~/ak8s/GKE_Services (qwiklabs-gcp-04-f1b2097fd2bf)$ 
```

