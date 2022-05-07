---
layout: single
title:  "Kubernetes + Citrix ADC"
date:   2022-05-07 05:59:04 +0530
categories: Kubernetes
tags: Citrix
show_date: true
classes: wide
header:
  teaser: /assets/images/citrix.jpeg
author:
  name     : "Citrix"
  avatar   : "/assets/images/citrix.jpeg"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Kubernetes + Citrix ADC
In this post, let us see how to load balance Ingress traffic with Citrix ADC CPX in Minikube.

```sh
pradeep:~$minikube start                                                                  
😄  minikube v1.25.2 on Darwin 12.2.1
✨  Automatically selected the docker driver. Other choices: hyperkit, virtualbox, ssh
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
pradeep:~$

```

Deploy Citrix ADC CPX as an Ingress proxy in the Minikube cluster

```yaml
pradeep:~$cat cpx.yaml 
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cpx-ingress-k8s-role
rules:
  - apiGroups: [""]
    resources: ["endpoints", "ingresses", "pods", "secrets", "nodes", "routes", "namespaces", "configmaps", "services"]
    verbs: ["get", "list", "watch"]
  # services/status is needed to update the loadbalancer IP in service status for integrating
  # service of type LoadBalancer with external-dns
  - apiGroups: [""]
    resources: ["services/status"]
    verbs: ["patch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create"]
  - apiGroups: ["extensions"]
    resources: ["ingresses", "ingresses/status"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses", "ingresses/status", "ingressclasses"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["apiextensions.k8s.io"]
    resources: ["customresourcedefinitions"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["citrix.com"]
    resources: ["rewritepolicies", "authpolicies", "ratelimits", "listeners", "httproutes", "continuousdeployments", "apigatewaypolicies", "wafs", "bots"]
    verbs: ["get", "list", "watch", "create", "delete", "patch"]
  - apiGroups: ["citrix.com"]
    resources: ["rewritepolicies/status", "continuousdeployments/status", "authpolicies/status", "ratelimits/status", "listeners/status", "httproutes/status", "wafs/status", "apigatewaypolicies/status", "bots/status"]
    verbs: ["patch"]
  - apiGroups: ["citrix.com"]
    resources: ["vips"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: ["route.openshift.io"]
    resources: ["routes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["crd.projectcalico.org"]
    resources: ["ipamblocks"]
    verbs: ["get", "list", "watch"]

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cpx-ingress-k8s-role
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cpx-ingress-k8s-role
subjects:
- kind: ServiceAccount
  name: cpx-ingress-k8s-role
  namespace: default
apiVersion: rbac.authorization.k8s.io/v1

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: cpx-ingress-k8s-role
  namespace: default

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpx-ingress
  labels:
    name: cpx-ingress
    app: cpx-ingress
spec:
  selector:
    matchLabels:
      app: cpx-ingress
  replicas: 1
  template:
    metadata:
      name: cpx-ingress
      labels:
        app: cpx-ingress
      annotations: null
    spec:
      serviceAccountName: cpx-ingress-k8s-role
      containers:
        - name: cpx-ingress
          image: quay.io/citrix/citrix-k8s-cpx-ingress:13.0-79.64
          tty: true
          securityContext:
             privileged: true
          env:
          - name: "EULA"
            value: "yes"
          - name: "KUBERNETES_TASK_ID"
            value: ""
          imagePullPolicy: Always
          volumeMounts:
            - mountPath: /var/deviceinfo
              name: shared-data
            - mountPath: /cpx/
              name: cpx-volume
        # Add cic as a sidecar
        - name: cic
          image: quay.io/citrix/citrix-k8s-ingress-controller:1.18.5
          volumeMounts:
          - mountPath: /var/deviceinfo
            name: shared-data
          args:
          - --ingress-classes
              citrix
          env:
          - name: "EULA"
            value: "yes"
          - name: "NS_IP"
            value: "127.0.0.1"
          - name: "NS_PROTOCOL"
            value: "HTTP"
          - name: "NS_PORT"
            value: "80"
          - name: "NS_DEPLOYMENT_MODE"
            value: "SIDECAR"
          - name: "NS_ENABLE_MONITORING"
            value: "YES"
          - name: "LOGLEVEL"
            value: "INFO"
          - name: POD_NAME
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.namespace
          imagePullPolicy: Always
      volumes:
      - name: shared-data
        emptyDir: {}
      - name: cpx-volume
        emptyDir: {}
        
---

apiVersion: v1
kind: Service
metadata:
  name: cpx-service
  labels:
    app: cpx-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: cpx-ingress

pradeep:~$
```

```sh
pradeep:~$kubectl create -f cpx.yaml 
clusterrole.rbac.authorization.k8s.io/cpx-ingress-k8s-role created
clusterrolebinding.rbac.authorization.k8s.io/cpx-ingress-k8s-role created
serviceaccount/cpx-ingress-k8s-role created
deployment.apps/cpx-ingress created
service/cpx-service created
pradeep:~$
```
Verify the installation using the following commands

```sh
pradeep:~$kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/cpx-ingress-64464fdb75-9wchv   2/2     Running   0          2m53s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/cpx-service   NodePort    10.99.230.163   <none>        80:32364/TCP,443:31712/TCP   2m53s
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP                      3m47s

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cpx-ingress   1/1     1            1           2m53s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/cpx-ingress-64464fdb75   1         1         1       2m53s
pradeep:~$

```



```sh
pradeep:~$kubectl get pods -l app=cpx-ingress -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
cpx-ingress-64464fdb75-9wchv   2/2     Running   0          3m11s   172.17.0.3   minikube   <none>           <none>
pradeep:~$

```

Deploy the `Guestbook` application

```yaml
pradeep:~$cat guestbook-app.yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    tier: backend
    role: master
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    tier: backend
    role: master
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: redis-master
spec:
  selector:
    matchLabels:
      app: redis
      role: master
      tier: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
      - name: master
        image: k8s.gcr.io/redis:e2e  # or just image: redis
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    tier: backend
    role: slave
spec:
  ports:
  - port: 6379
  selector:
    app: redis
    tier: backend
    role: slave
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: redis-slave
spec:
  selector:
    matchLabels:
      app: redis
      role: slave
      tier: backend
  replicas: 2
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
      - name: slave
        image: gcr.io/google_samples/gb-redisslave:v1
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access an environment variable to find the master
          # service's host, comment out the 'value: dns' line above, and
          # uncomment the line below:
          # value: env
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: guestbook
    tier: frontend
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google-samples/gb-frontend:v4
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below:
          # value: env
        ports:
        - containerPort: 80
---
```

```sh
pradeep:~$kubectl create -f guestbook-app.yaml
service/redis-master created
deployment.apps/redis-master created
service/redis-slave created
deployment.apps/redis-slave created
service/frontend created
deployment.apps/frontend created
pradeep:~$

```



```sh
pradeep:~$kubectl get all
NAME                                READY   STATUS              RESTARTS   AGE
pod/cpx-ingress-64464fdb75-9wchv    2/2     Running             0          3m43s
pod/frontend-7b988b7c4b-2vmht       0/1     ContainerCreating   0          10s
pod/frontend-7b988b7c4b-8q8t9       0/1     ContainerCreating   0          10s
pod/frontend-7b988b7c4b-xmp7f       0/1     ContainerCreating   0          10s
pod/redis-master-55db8bb568-9lnqx   0/1     ContainerCreating   0          10s
pod/redis-slave-566774f44b-mwzzn    0/1     ContainerCreating   0          10s
pod/redis-slave-566774f44b-whbj8    0/1     ContainerCreating   0          10s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/cpx-service    NodePort    10.99.230.163   <none>        80:32364/TCP,443:31712/TCP   3m43s
service/frontend       ClusterIP   10.99.43.154    <none>        80/TCP                       10s
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP                      4m37s
service/redis-master   ClusterIP   10.102.9.199    <none>        6379/TCP                     11s
service/redis-slave    ClusterIP   10.98.102.18    <none>        6379/TCP                     10s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cpx-ingress    1/1     1            1           3m43s
deployment.apps/frontend       0/3     3            0           10s
deployment.apps/redis-master   0/1     1            0           11s
deployment.apps/redis-slave    0/2     2            0           10s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/cpx-ingress-64464fdb75    1         1         1       3m43s
replicaset.apps/frontend-7b988b7c4b       3         3         0       10s
replicaset.apps/redis-master-55db8bb568   1         1         0       10s
replicaset.apps/redis-slave-566774f44b    2         2         0       10s
pradeep:~$
```
After some time

```sh
pradeep:~$kubectl get all -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod/cpx-ingress-64464fdb75-9wchv    2/2     Running   0          14m   172.17.0.3   minikube   <none>           <none>
pod/frontend-7b988b7c4b-2vmht       1/1     Running   0          11m   172.17.0.9   minikube   <none>           <none>
pod/frontend-7b988b7c4b-8q8t9       1/1     Running   0          11m   172.17.0.8   minikube   <none>           <none>
pod/frontend-7b988b7c4b-xmp7f       1/1     Running   0          11m   172.17.0.7   minikube   <none>           <none>
pod/redis-master-55db8bb568-9lnqx   1/1     Running   0          11m   172.17.0.6   minikube   <none>           <none>
pod/redis-slave-566774f44b-mwzzn    1/1     Running   0          11m   172.17.0.5   minikube   <none>           <none>
pod/redis-slave-566774f44b-whbj8    1/1     Running   0          11m   172.17.0.4   minikube   <none>           <none>

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE   SELECTOR
service/cpx-service    NodePort    10.99.230.163   <none>        80:32364/TCP,443:31712/TCP   14m   app=cpx-ingress
service/frontend       ClusterIP   10.99.43.154    <none>        80/TCP                       11m   app=guestbook,tier=frontend
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP                      15m   <none>
service/redis-master   ClusterIP   10.102.9.199    <none>        6379/TCP                     11m   app=redis,role=master,tier=backend
service/redis-slave    ClusterIP   10.98.102.18    <none>        6379/TCP                     11m   app=redis,role=slave,tier=backend

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                                                                                 SELECTOR
deployment.apps/cpx-ingress    1/1     1            1           14m   cpx-ingress,cic   quay.io/citrix/citrix-k8s-cpx-ingress:13.0-79.64,quay.io/citrix/citrix-k8s-ingress-controller:1.18.5   app=cpx-ingress
deployment.apps/frontend       3/3     3            3           11m   php-redis         gcr.io/google-samples/gb-frontend:v4                                                                   app=guestbook,tier=frontend
deployment.apps/redis-master   1/1     1            1           11m   master            k8s.gcr.io/redis:e2e                                                                                   app=redis,role=master,tier=backend
deployment.apps/redis-slave    2/2     2            2           11m   slave             gcr.io/google_samples/gb-redisslave:v1                                                                 app=redis,role=slave,tier=backend

NAME                                      DESIRED   CURRENT   READY   AGE   CONTAINERS        IMAGES                                                                                                 SELECTOR
replicaset.apps/cpx-ingress-64464fdb75    1         1         1       14m   cpx-ingress,cic   quay.io/citrix/citrix-k8s-cpx-ingress:13.0-79.64,quay.io/citrix/citrix-k8s-ingress-controller:1.18.5   app=cpx-ingress,pod-template-hash=64464fdb75
replicaset.apps/frontend-7b988b7c4b       3         3         3       11m   php-redis         gcr.io/google-samples/gb-frontend:v4                                                                   app=guestbook,pod-template-hash=7b988b7c4b,tier=frontend
replicaset.apps/redis-master-55db8bb568   1         1         1       11m   master            k8s.gcr.io/redis:e2e                                                                                   app=redis,pod-template-hash=55db8bb568,role=master,tier=backend
replicaset.apps/redis-slave-566774f44b    2         2         2       11m   slave             gcr.io/google_samples/gb-redisslave:v1                                                                 app=redis,pod-template-hash=566774f44b,role=slave,tier=backend
pradeep:~$

```


```sh
pradeep:~$kubectl get pods -l 'app in (guestbook, redis)' 
NAME                            READY   STATUS    RESTARTS   AGE
frontend-7b988b7c4b-2vmht       1/1     Running   0          11m
frontend-7b988b7c4b-8q8t9       1/1     Running   0          11m
frontend-7b988b7c4b-xmp7f       1/1     Running   0          11m
redis-master-55db8bb568-9lnqx   1/1     Running   0          11m
redis-slave-566774f44b-mwzzn    1/1     Running   0          11m
redis-slave-566774f44b-whbj8    1/1     Running   0          11m
pradeep:~$

```






Deploy an Ingress rule that sends traffic to [http://www.guestbook.com](http://www.guestbook.com/).

```yaml
pradeep:~$cat guestbook-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: citrix
spec:
  controller: citrix.com/ingress-controller

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: guestbook-ingress
spec:
  ingressClassName: citrix
  rules:
  - host:  www.guestbook.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port: 
              number: 80
---
pradeep:~$
```

```sh
pradeep:~$kubectl create -f guestbook-ingress.yaml 
ingressclass.networking.k8s.io/citrix created
ingress.networking.k8s.io/guestbook-ingress created
pradeep:~$
```

```sh
pradeep:~$kubectl get ingress
NAME                CLASS    HOSTS               ADDRESS   PORTS   AGE
guestbook-ingress   citrix   www.guestbook.com             80      22s
```

```sh
pradeep:~$kubectl describe ingress
Name:             guestbook-ingress
Namespace:        default
Address:          
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host               Path  Backends
  ----               ----  --------
  www.guestbook.com  
                     /   frontend:80 (172.17.0.7:80,172.17.0.8:80,172.17.0.9:80)
Annotations:         <none>
Events:              <none>
pradeep:~$

```

```sh
pradeep:~$kubectl get svc
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
cpx-service    NodePort    10.99.230.163   <none>        80:32364/TCP,443:31712/TCP   16m
frontend       ClusterIP   10.99.43.154    <none>        80/TCP                       12m
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP                      17m
redis-master   ClusterIP   10.102.9.199    <none>        6379/TCP                     12m
redis-slave    ClusterIP   10.98.102.18    <none>        6379/TCP                     12m
pradeep:~$

```

```sh
pradeep:~$kubectl get nodes -o wide
NAME       STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane,master   17m   v1.23.3   192.168.49.2   <none>        Ubuntu 20.04.2 LTS   5.10.104-linuxkit   docker://20.10.12
pradeep:~$
```

```sh
pradeep:~$curl -s -H "Host: www.guestbook.com" http://192.168.49.2:32364 | grep Guestbook
^C
pradeep:~$
```

```sh
pradeep:~$minikube service cpx-service --url

🏃  Starting tunnel for service cpx-service.
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.



^C✋  Stopping tunnel for service cpx-service.
pradeep:~$

```

For some reason, we are not able to access the URL.

I have edited the Service Type from NodePort to LoadBalancer in the `cpx.yaml` manifest.

Here is how it looks now.

```yaml
---

apiVersion: v1
kind: Service
metadata:
  name: cpx-service
  labels:
    app: cpx-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: cpx-ingress

pradeep:~$

```



```sh
pradeep:~$kubectl apply -f cpx.yaml         
Warning: resource clusterroles/cpx-ingress-k8s-role is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrole.rbac.authorization.k8s.io/cpx-ingress-k8s-role configured
Warning: resource clusterrolebindings/cpx-ingress-k8s-role is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
clusterrolebinding.rbac.authorization.k8s.io/cpx-ingress-k8s-role configured
Warning: resource serviceaccounts/cpx-ingress-k8s-role is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
serviceaccount/cpx-ingress-k8s-role configured
Warning: resource deployments/cpx-ingress is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/cpx-ingress configured
Warning: resource services/cpx-service is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
service/cpx-service configured
pradeep:~$
```



```sh
pradeep:~$kubectl get svc
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
cpx-service    LoadBalancer   10.99.230.163   <pending>     80:32364/TCP,443:31712/TCP   20m
frontend       ClusterIP      10.99.43.154    <none>        80/TCP                       16m
kubernetes     ClusterIP      10.96.0.1       <none>        443/TCP                      21m
redis-master   ClusterIP      10.102.9.199    <none>        6379/TCP                     16m
redis-slave    ClusterIP      10.98.102.18    <none>        6379/TCP                     16m
pradeep:~$
```

Service Type has changed, but External IP is still pending. For this, I ran `minikube tunnel` in another window

```sh
pradeep:~$minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress cpx-service requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service cpx-service.
❗  The service/ingress guestbook-ingress requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service guestbook-ingress.
Password:

```



```sh
pradeep:~$kubectl get svc
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
cpx-service    LoadBalancer   10.99.230.163   127.0.0.1     80:32364/TCP,443:31712/TCP   22m
frontend       ClusterIP      10.99.43.154    <none>        80/TCP                       18m
kubernetes     ClusterIP      10.96.0.1       <none>        443/TCP                      23m
redis-master   ClusterIP      10.102.9.199    <none>        6379/TCP                     18m
redis-slave    ClusterIP      10.98.102.18    <none>        6379/TCP                     18m

```

Now the external IP for `cpx-service` is shown as `127.0.0.1` , so I have edited my hosts table 

```sh
pradeep:~$cat /etc/hosts
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost
# Added by Docker Desktop
# To allow the same kube context to work on the host and the container:
127.0.0.1 kubernetes.docker.internal
# End of section
172.16.30.6	hello-world.info
127.0.0.1    www.guestbook.com
172.16.30.6	play.games.com
pradeep:~$
```

With this, I am able to access the `www.guestbook.com` URL and I can see our `guestbook` application. I have added few test entries.

![Guestbook Demo]({{ site.url }}{{ site.baseurl }}/assets/images/guestbook-demo.png)
