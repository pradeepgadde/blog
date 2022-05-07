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

Deploy Citrix ADC CPX as an Ingress proxy in the Minikube cluster using the manifest file available at [Citrix GitHub Repository](https://github.com/citrix/cloud-native-getting-started/blob/master/beginners-guide/cpx-in-minikube.md).

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

Let us look at the logs of our `cpx-ingress` pod

```sh
pradeep:~$kubectl logs cpx-ingress-64464fdb75-9wchv
error: a container name must be specified for pod cpx-ingress-64464fdb75-9wchv, choose one of: [cpx-ingress cic]

```

There are two containers inside this Pod, so let us look at the logs of individual container.

```sh
pradeep:~$kubectl logs cpx-ingress-64464fdb75-9wchv cpx-ingress

 User has accepted EULA. Starting CPX 

/var/netscaler/bins/docker_startup.sh: line 457: [: too many arguments
/var/netscaler/bins/docker_startup.sh: line 460: [: too many arguments
ignoring ['#', 'Kubernetes-managed', 'hosts', 'file.']
ignoring ['::1', 'localhost', 'ip6-localhost', 'ip6-loopback']
ignoring ['fe00::0', 'ip6-localnet']
ignoring ['fe00::0', 'ip6-mcastprefix']
ignoring ['fe00::1', 'ip6-allnodes']
ignoring ['fe00::2', 'ip6-allrouters']
cp: cannot stat '/etc/monit.d/*': No such file or directory
RTNETLINK answers: Permission denied
1500
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv rsyslogd: [origin software="rsyslogd" swVersion="8.16.0" x-pid="160" x-info="http://www.rsyslog.com"] start
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  763.258313] device eth0 left promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  763.268031] device ns1 left promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  764.937479] br-d040a08c262e: port 1(vetha3afe06) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  764.937572] veth1c3fee2: renamed from eth0
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  764.963480] br-d040a08c262e: port 1(vetha3afe06) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  764.967141] device vetha3afe06 left promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  764.967150] br-d040a08c262e: port 1(vetha3afe06) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  784.723203] docker0: port 1(veth651ef49) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  784.723206] docker0: port 1(veth651ef49) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  784.723329] device veth651ef49 entered promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  784.814341] IPVS: ftp: loaded support on port[0] = 21
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.118627] eth0: renamed from vetha2a2d6a
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.126414] IPv6: ADDRCONF(NETDEV_CHANGE): veth651ef49: link becomes ready
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.126520] docker0: port 1(veth651ef49) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.126523] docker0: port 1(veth651ef49) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.126618] IPv6: ADDRCONF(NETDEV_CHANGE): docker0: link becomes ready
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.270379] vetha2a2d6a: renamed from eth0
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.283761] docker0: port 1(veth651ef49) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.287905] docker0: port 1(veth651ef49) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.289121] device veth651ef49 left promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.289148] docker0: port 1(veth651ef49) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.559124] docker0: port 1(veth816d305) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.559132] docker0: port 1(veth816d305) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.559214] device veth816d305 entered promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.559363] docker0: port 1(veth816d305) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.559366] docker0: port 1(veth816d305) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.665023] IPVS: ftp: loaded support on port[0] = 21
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.724457] docker0: port 1(veth816d305) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.892701] eth0: renamed from veth83c8cbb
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.901416] IPv6: ADDRCONF(NETDEV_CHANGE): veth816d305: link becomes ready
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.901538] docker0: port 1(veth816d305) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  785.901541] docker0: port 1(veth816d305) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  799.524278] veth83c8cbb: renamed from eth0
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  799.647665] docker0: port 1(veth816d305) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  799.651233] docker0: port 1(veth816d305) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  799.651927] device veth816d305 left promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  799.651933] docker0: port 1(veth816d305) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.073123] br-262a9c2faaf0: port 1(vetha1d0438) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.073126] br-262a9c2faaf0: port 1(vetha1d0438) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.073205] device vetha1d0438 entered promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.300626] IPVS: ftp: loaded support on port[0] = 21
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.610528] eth0: renamed from veth1fbc6b8
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.618593] IPv6: ADDRCONF(NETDEV_CHANGE): vetha1d0438: link becomes ready
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.618713] br-262a9c2faaf0: port 1(vetha1d0438) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.618716] br-262a9c2faaf0: port 1(vetha1d0438) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  802.618772] IPv6: ADDRCONF(NETDEV_CHANGE): br-262a9c2faaf0: link becomes ready
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.070485] docker0: port 1(veth22d4ecf) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.070490] docker0: port 1(veth22d4ecf) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.070625] device veth22d4ecf entered promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.408490] IPVS: ftp: loaded support on port[0] = 21
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.687396] eth0: renamed from vethfece6c1
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.698757] docker0: port 1(veth22d4ecf) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  853.698766] docker0: port 1(veth22d4ecf) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  892.902692] docker0: port 2(vethbef4eac) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  892.902696] docker0: port 2(vethbef4eac) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  892.902752] device vethbef4eac entered promiscuous mode
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  892.988796] IPVS: ftp: loaded support on port[0] = 21
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  893.170749] eth0: renamed from veth544f918
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  893.176343] docker0: port 2(vethbef4eac) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [  893.176346] docker0: port 2(vethbef4eac) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [ 1009.083137] docker0: port 2(vethbef4eac) entered disabled state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [ 1009.088205] docker0: port 2(vethbef4eac) entered blocking state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv kernel: [ 1009.088209] docker0: port 2(vethbef4eac) entered forwarding state
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv rsyslogd-2039: Could not open output pipe '/dev/xconsole':: No such file or directory [v8.16.0 try http://www.rsyslog.com/e/2039 ]
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv rsyslogd-2007: action 'action 10' suspended, next retry is Sat May  7 12:21:55 2022 [v8.16.0 try http://www.rsyslog.com/e/2007 ]
('Generated UUID for CPX: %s\n', 'f54be1f3-9d4b-4727-b35c-245866c5737f')
sysctl netscaler.additional_mgmt_cpu: No such file or directory
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: PPE-0 : Lower PE :Debug Info 1: 0x6d5a56da 0x255b0ec2 0x4167253d 0x43a38fb0 0xd0ca2bcb 0xae7b30b4 0x77cb2da3 0x8030f20c 0x4167253d 0x43a38fb0#012 
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: ns_setup_sc:  mtu is[65535] for interface[ns1]
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv kernel: [ 1009.844368] device ns1 entered promiscuous mode
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: vmpe_intf_linux_attach: setting interface 0 MTU as 65535 in PE ID=0
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: ns_setup_sc:  mtu is[1500] for interface[eth0]
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv kernel: [ 1009.860250] device eth0 entered promiscuous mode
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: vmpe_intf_linux_attach: setting interface 1 MTU as 1500 in PE ID=0












RTNETLINK answers: Permission denied
May  7 12:21:26 cpx-ingress-64464fdb75-9wchv nsppe: IPv6 address fe80:0:0:0:3048:11ff:fe95:4c2d/64 modification on interface ns2 failed
May  7 12:21:25 cpx-ingress-64464fdb75-9wchv rsyslogd-2222: command 'KLogPermitNonKernelFacility' is currently not permitted - did you already set it via a RainerScript command (v6+ config)? [v8.16.0 try http://www.rsyslog.com/e/2222 ]
cat: /root/.blx/environ: No such file or directory
/netscaler/nslog.sh: 55: /netscaler/nslog.sh: [[: not found
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv cron[257]: (CRON) INFO (pidfile fd = 3)
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv cron[258]: (CRON) STARTUP (fork ok)
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv cron[258]: (CRON) INFO (Running @reboot jobs)
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: nsaggregator: system command 'echo nslog: cannot access /var/nslog/nslog.nextfile due to error=2, use 0 >> /var/nslog/ns.log' failed ret -1
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: nsaggregator: system command 'echo nslog: `date`: renaming /var/nslog/newnslog to /var/nslog/newnslog.0 >> /var/nslog/ns.log' failed ret -1
nsaggregatord  pid file not written properly !!! Retrying !!!
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: nsaggregator: system command 'mkdir /var/nslog/newnslog.0' failed ret -1
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: nsaggregator: system command 'mv /var/nslog/newnslog/newnslog.ppe.* /var/nslog/newnslog.0' failed ret -1
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: nsaggregator: system command 'echo nslog: next backup file number will be 1>>/var/nslog/ns.log' failed ret -1
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsapimgr: nsnet_tcpipconnect: connect() failed; returned -1 errno=2
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: System command nice -15 /netscaler/nslog.sh checkspace failed ret -1
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nsaggregatord: not enough space left on /var partition!#012Failed to free disk space for NetScaler system log:#012No more file available for deletion#012Free some disk space on /var partition
nsaggregatord  pid file written !!! Successfully !!!
mallopt(): setting M_MMAP_THRESHOLD failed
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nssetup: zebos_config_migrate(): /nsconfig/ZebOS.conf will be backed up to /nsconfig/ZebOS.conf.
cp: cannot stat '/nsconfig/ZebOS.conf': No such file or directory
Command failed: cp /nsconfig/ZebOS.conf /nsconfig/ZebOS.conf.migrate
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nssetup: zebos_config_migrate(): Command failed: cp /nsconfig/ZebOS.conf /nsconfig/ZebOS.conf.migrate
cp: cannot stat '/nsconfig/ZebOS.conf': No such file or directory
Command failed: cp /nsconfig/ZebOS.conf /nsconfig/ZebOS.conf.
initial config source: NONE
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nssetup: zebos_config_migrate(): Command failed: cp /nsconfig/ZebOS.conf /nsconfig/ZebOS.conf.
Warning: NetScaler IP not found, using defaults
peconfig: {ip=192.168.100.1, netmask=255.255.0.0, num_pes=1, num_sslchips=-1,
May  7 12:21:28 cpx-ingress-64464fdb75-9wchv nssetup: _config(): Warning: NetScaler IP not found, using defaults
           platform_lic=0x0, timezone=0, dstflag=0,
			rss_key_type=0, rss_key=NULL}
mallopt(): setting M_MMAP_THRESHOLD failed
May  7 12:21:33 cpx-ingress-64464fdb75-9wchv nsnetsvc: check_and_establish_connections(): successfully connected to all packet engines
May  7 12:21:33 cpx-ingress-64464fdb75-9wchv nsnetsvc: nsnetsvc sent command NSAPI_ENABLELICENSE to PEs, ErrorCode=0x0
May  7 12:21:33 cpx-ingress-64464fdb75-9wchv nsnetsvc: nsnetsvc sent command NSAPI_PLCY_ADD_DEF to PEs, ErrorCode=0x80000C09
May  7 12:21:33 cpx-ingress-64464fdb75-9wchv nsnetsvc: process_dhparam NSAPI_EXCHANGE_DHPARAM completed.
May  7 12:21:33 cpx-ingress-64464fdb75-9wchv nsmap[333]: Started nsmap daemon
mallopt(): setting M_MMAP_THRESHOLD failed
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: ns_init_global_partition_id(): Creating SHM for storing partition id
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: main(): Warm Reboot - unsetting partition ids in shared mmy
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_prime(): Cluster is not enabled
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: check_and_establish_connections(): successfully connected to all packet engines
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): Establishing built-in entities
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset vpn_cache_dirs", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_vpn_dfa_client_cookies", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_vpn_client_useragents", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_bypass_domains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_inet_domains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_js_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_css_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_xcmp_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_xml_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_client_cookies", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_rel_urls_set", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_custom_content_types", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_default_custom_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_owa_js_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_owa_xml_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_owa_client_cookies", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp_js_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp_res_pat", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp_body_decode_pat", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cr_dynamic_ext", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cr_dynamic_path", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sharepoint_hostnames", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp2013_hostnames", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp2013_js_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_sp2013_custom_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_js_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_css_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_xcmp_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_xml_urls_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_custom_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_fast_regex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_fast_regex_light_ver", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_vsvr_fqdn", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_cvpn_v2_application_content_type_end", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset vpn_cache_dirs "/epa/" -index 1 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset vpn_cache_dirs "/vpn/" -index 2 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset vpn_cache_dirs "/vpns/" -index 3 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset vpn_cache_dirs "/logon/" -index 4 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_bypass_domains schemas. -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_bypass_domains .w3.org -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_bypass_domains help.outlook.com -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_inet_domains 127.0.0. -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_dfa_client_cookies CtxsAMStorage -index 1 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_client_useragents AGEE -index 1 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_client_useragents CitrixReceiver -index 2 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_client_useragents AGMacClient -index 3 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_client_useragents "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:18.0) Gecko/20100101 Firefox/18.0" -index 4 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:35 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_vpn_client_useragents "Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:22.0) Gecko/20100101 Firefox/22.0" -index 5 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDialogUrl|eeDlgUrl)(?:,|src\s!=|=|return)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDailogUrl|eeDlgUrl)(?:stImagesPath\s\+=)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q~(?<!replace|dexOf|charAt|if\s|split|peForSync|a.push|b.push|a.append|eval|.startsWith)(?:\()\s*(?:'|"|\&#39;|&quot;)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?))(?:'|"|\&#39;|&quot;|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?:\.location\.replace\s*\(\s*)(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q~(?<!\+|strDlgReturnValue\[3\]\s)(?:,|=|\(|return)\s*(?:'|"|\\u0022|\\u0027|\&#39;)((https?:\\u002f)?\\u002f[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?)(?:EditForm\.aspx|'|"|\\u0022|\\u0027|\&#39;)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q~(?<!\+)(?:=)\s*((https?(%3a%2f|%253a%252f))?(%2f|%252f)[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?)(?:'|"|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?:=|\||:')(https?://[\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*?)(?:'|"|\&|\?|\|)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?:\\\\u0027)((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:\\\\u0027)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?:: ")((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?::"|\?"|'"|\"|\'|\\"|\\')((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:"|\"|\'|\\"|\\')> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_js_urls_regex q<(?:Source=)((https?\\u00253A)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:\\u00253FRootFolder)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_fast_regex q<(?:"|'|\||=|%[25]*22|%[25]*27|\\+u00[25]*22|\\+u00[25]*27|1|0)(https?\\*(:|%[25]*3[aA]|\\+u00[25]*3[aA])(\\*/|(%[25]*2[fF]|\\+u00[25]*2[fF])){2}([\\/a-zA-Z0-9:;%#@! ,\.&_\|\?\=\[\]\(\){}~-]+))(?:"|'|\||\?|\&|%[25]*22|%[25]*27|\\+u00[25]*22|\\+u00[25]*27)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_fast_regex q<(?:"|'|\||=|1|0)(https?\\*(:|\\x[25]*3[aA]|&#x[25]*3[aA];?)(\\*/|(\\x[25]*2[fF]|&#x[25]*2[fF];?)){2}([\\/a-zA-Z0-9:;%#@! ,\.&_\|\?\=\[\]\(\){}~-]+))(?:"|'|\||\?|\&)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_fast_regex_light_ver q<(?:"|'|\||=|%[25]*2[27]|\\+u00[25]*2[27]|[01])(https?\\*(:|%[25]*3[aA]|\\+u00[25]*3[aA]|\\x[25]*3[aA]|&#x[25]*3[aA];?)([\\/a-zA-Z0-9:;%#@! ,\.&_\|\?\=\[\]\(\){}~-]+))(?:"|'|\||\?|\&|%[25]*2[27]|\\+u00[25]*2[27])> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_css_urls_regex "url\\((?:\'|\")?((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_css_urls_regex "@import\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xcmp_urls_regex "(?:src|href)\\s*=\\s*(?:\'|\")?((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xcmp_urls_regex q~(?<!replace|dexOf|charAt|if\s)(?:\()\s*(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|"|\?|\&)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xml_urls_regex "(?<!xsd|xsi)\\s*(?:=)\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9 :/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\+\\|\\^\\`{};-]+?)(?:\'|\"|\\?|\\&)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xml_urls_regex "(?:href|url|link|true\"|Result|icon|url|p2|p1)>((https?:/)?/[a-zA-Z0-9 :/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\+\\|\\^\\`{};-]+)</" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xml_urls_regex "src\\s*=\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9:/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\|\\^\\`{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xml_urls_regex "<imagepath>(/exchweb/img/)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_xml_urls_regex "<xsl:attribute name=\"src\">(/exchweb/img/)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDialogUrl|eeDlgUrl)(?:,|src\s!=|=|return)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDailogUrl|eeDlgUrl)(?:stImagesPath\s\+=)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q~(?<!replace|dexOf|charAt|if\s|split|peForSync|a.push|b.push|a.append|eval|.startsWith)(?:\()\s*(?:'|"|\&#39;|&quot;)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?))(?:'|"|\&#39;|&quot;|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?:\.location\.replace\s*\(\s*)(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q~(?<!\+|strDlgReturnValue\[3\]\s)(?:,|=|\(|return)\s*(?:'|"|\\u0022|\\u0027|\&#39;)((https?:\\u002f)?\\u002f[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?)(?:EditForm\.aspx|'|"|\\u0022|\\u0027|\&#39;)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q~(?<!\+)(?:=)\s*((https?(%3a%2f|%253a%252f))?(%2f|%252f)[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?)(?:'|"|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?:=|\||:')(https?://[\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*?)(?:'|"|\&|\?|\|)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?:\\\\u0027)((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:\\\\u0027)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?:: ")((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?::"|\?"|'"|\"|\'|\\"|\\')((https?)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:"|\"|\'|\\"|\\')> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_custom_regex q<(?:Source=)((https?\\u00253A)([\\/a-zA-Z0-9:;%#@!, \.&_\|\?\=\[\]\(\){}~-]*))(?:\\u00253FRootFolder)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_js_urls_regex q~(?<!\+|chDateSep\s)(?:,|=|:|return)\s*(?:"|'|\\"|\\')((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&\+{};-]*?))(?:'|"|\?\&)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_js_urls_regex q~(?<!replace|dexOf|charAt)(?:\()\s*(?:"|'|\\"|\\')((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&\+{};-]*?))(?:'|"|\?\&)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_js_urls_regex q<(?:\.location\.replace\s*\(\s*)(?:"|'|\\"|\\')((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_js_urls_regex q<\.location\.host\s*\+\s*(?:"|'|\\"|\\')(/[a-zA-Z0-9:\._%/\!,\*\(\)\]\[@\#$\?\=\&{};-]+)(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_js_urls_regex q<(?:path\s*=\s*)(/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_css_urls_regex "url\\((?:\'|\")?((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_css_urls_regex "@import\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_xcmp_urls_regex "(?:src|href)\\s*=\\s*(?:\'|\")?((https?:/)?/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_xcmp_urls_regex q~(?<!replace|dexOf|charAt|if\s)(?:\()\s*(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_xml_urls_regex "(?<!xsd|xsi)\\s*(?:=)\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9 :/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\+\\|\\^\\`{};-]+?)(?:\'|\")" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_xml_urls_regex "(?:href|url|link|true\"|Result)>((https?:/)?/[a-zA-Z0-9 :/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\+\\|\\^\\`{};-]+)</" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_xml_urls_regex "(?:>webUrl=|>service_name=)((https?:/)?/[a-zA-Z0-9 :/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\+\\|\\^\\`{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_client_cookies WIClientInfo -index 1 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_client_cookies AGSSOHasOccurred -index 2 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_client_cookies CsrfToken -index 3 -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_rel_urls_set "/" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_rel_urls_set "\\" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_rel_urls_set "\\/" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_rel_urls_set "%2f" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_default_rel_urls_set "\\u002f" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_custom_content_types "text/plain" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_custom_content_types "application/json" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_custom_content_types "application/javascript" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end "+xml"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end "+json"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end javascript", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end http", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end node", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_v2_application_content_type_end x-www-form-urlencoded", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?::"|:')((\\u002f_layouts|\\u002fsites|https?)([\\/a-zA-Z0-9:% \._\?\={}&-~]*))(?:"|')> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?:\\u0027)((https?|\\u002fsites)([\\/a-zA-Z0-9:% \._\?\={}&-~]*))(?:\\u0027)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?::")(\\\\?\\?(/sites|/_layouts)([\\/a-zA-Z0-9:% \._\?\={}&~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?:: "|,"|,\s"|{"|:"|,\n"|,\r\n")((/sites|/_layouts)([\\/a-zA-Z0-9:% \._\?\={}&~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?:: ")((\\u002fsites)([\\/a-zA-Z0-9:% \._\?\={}&-~]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_custom_regex q<(?:\(\\")((/_layouts)([\\/a-zA-Z0-9:% \._\?\={}&~-]*))(?:\\")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex q~(?<!a_sId\s|_id|a_sCK\s)(?:=|szServer\s\+)\s*(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&{};-]+))(?:'|")~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex q~(?<!replace|selectNodes|ngleNode)(?:\()\s*(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\?\=\&{};-]+))(?:'|")~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex "\\.location\\.host\\s*\\+\\s*(?:\'|\")(/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)(?:\'|\")" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex ",\\s*(?:\'|\")(/exchweb/[a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)(?:\'|\")" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex q<(?:'|")(/owa/[\.\*\(\)\[\]\{\}\$\-\?\+\|\^\\/a-zA-Z0-9:_%!,@#=$;]*)(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_js_urls_regex q<(?:'|")((/|\\/)ecp[\.\*\(\)\[\]\{\}\$\-\?\+\|\^\\/a-zA-Z0-9:_%!,@#=$;]*)(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_xml_urls_regex "(?:href|icon|url|p2|p1)>((https?:/)?/[a-zA-Z0-9:/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\|\\^\\`{};-]+)</" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_xml_urls_regex "src\\s*=\\s*(?:\'|\")((https?:/)?/[a-zA-Z0-9:/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\|\\^\\`{};-]+)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_xml_urls_regex "<imagepath>(/exchweb/img/)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_xml_urls_regex "<xsl:attribute name=\"src\">(/exchweb/img/)" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_client_cookies msExchEcpCanary -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_client_cookies UserContext -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_client_cookies PBack -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_owa_client_cookies X-OWA-CANARY -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|RootFolder\s|RootFolder|L_newaspx_Text\s|inalImg|g_RteDialogUrl)(?:,|src\s!=|=|return)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|RootFolder\s|RootFolder|L_newaspx_Text\s|inalImg|g_RteDailogUrl)(?:stImagesPath\s\+=)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q~(?<!replace|dexOf|charAt|if\s|split|peForSync)(?:\()\s*(?:'|"|\&#39;|&quot;)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?))(?:'|"|\&#39;|&quot;|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex "\\.location\\.host\\s*\\+\\s*(?:\'|\")(/[a-zA-Z0-9: \\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)(?:\'|\")" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q<(?:\.location\.replace\s*\(\s*)(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q~(?<!\+)(?:,|=|\(|return)\s*(?:'|"|\\u0022|\\u0027|\&#39;)((https?:\\u002f)?\\u002f[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?)(?:'|"|\\u0022|\\u0027|\&#39;)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q~(?<!\+)(?:=)\s*((https?(%3a%2f|%253a%252f))?(%2f|%252f)[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?)(?:'|"|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_js_urls_regex q<(?:\?|\?=)(/_layouts([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\$\?\=\&{};-]*))(?:#)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sharepoint_hostnames sharepoint -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDialogUrl|eeDlgUrl)(?:,|src\s!=|=|return)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q~(?<!\+|chDateSep\s|chSep\s|\!|=|otFolder\s|otFolder|waspx_Text\s|alImg|eDailogUrl|eeDlgUrl)(?:stImagesPath\s\+=)\s*(?:'|"|&quot;|\\u0022)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?))(?:\?|\&|'|"|&quot;|\\u0022)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q~(?<!replace|dexOf|charAt|if\s|split|peForSync|a.push|b.push|a.append|eval|.startsWith)(?:\()\s*(?:'|"|\&#39;|&quot;)((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?))(?:'|"|\&#39;|&quot;|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex "\\.location\\.host\\s*\\+\\s*(?:\'|\")(/[a-zA-Z0-9: \\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&{};-]+)(?:\'|\")" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:\.location\.replace\s*\(\s*)(?:'|")((https?:(\\)?/)?(\\)?/([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*))(?:'|")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q~(?<!\+|strDlgReturnValue\[3\]\s)(?:,|=|\(|return)\s*(?:'|"|\\u0022|\\u0027|\&#39;)((https?:\\u002f)?\\u002f[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\?\=\&{};-]*?)(?:EditForm\.aspx|'|"|\\u0022|\\u0027|\&#39;)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q~(?<!\+)(?:=)\s*((https?(%3a%2f|%253a%252f))?(%2f|%252f)[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?)(?:'|"|\&|\?)~ -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:\?|\?=)(/_layouts([\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\$\?\=\&{};-]*))(?:#)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:=|\|)(https?://[\\/a-zA-Z0-9: \._%\!,\*\(\)\]\[@\#$\={};-]*?)(?:'|"|\&|\?|\|)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:0\|)(/[\\/a-zA-Z0-9:\._%\!,\*\(\)\]\[@\#$\={};-]*?)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?::"|:')((\\u002f_layouts|\\u002fsites|/my)([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:"|')> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:\\\\u0027)((\\u002fShared Documents|\\/sites|\\u002fsites|https?|\\\\\\\\u002fsites|\\\\u002fsites|\\\\u002fmy|\\u002fmy|\\\\\\\\u002fmy)([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:\\\\u0027)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:: "|:")((/sites|\\u002fsites|\\u002fIT|\\u002fwebsites|\\u002fNews Release|\\u002fSiteAssets|\\u002fmy|/my|https?)([\\/a-zA-Z0-9:% \._\?\=(){}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:: "|:")((/PublishingImages|\\u002fPublishingImages|/SitePages|\\u002fSitePages|\\u002fIT|\\u002fShared|/Shared)([\\/a-zA-Z0-9:% \._\?\=(){}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:;|\|)((/sites)([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:&|\|)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?::")(\\\\?\\?/sites([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?::"|\?"|'")((/_layouts|/sites|https?)([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:\\')((https?)([\\/a-zA-Z0-9:% \._\?\={}~-]*))(?:\\')> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:Source=)((https?\\u00253A)([\\/a-zA-Z0-9% \._\?\={}~-]*))(?:\\u00253FRootFolder)> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_js_urls_regex q<(?:Url":\s"|Url.desc":\s")((\\u002f_layouts)([\\/a-zA-Z0-9:%\ ._\?\={}~-]*))(?:")> -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp2013_hostnames sharepoint -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_res_pat KpiData -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_res_pat window.location -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_res_pat "/_layouts/1033/images" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_res_pat "/_layouts/images" -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_body_decode_pat Upload.aspx -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cvpn_sp_body_decode_pat owssvr.dll -charset ASCII", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .cgi", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .asp", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .exe", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .cfm", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .ex", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .shtml", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_ext .htx", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_path "/cgi-bin"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_path "/exec"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_cr_dynamic_path "/bin"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression is_vpn_url "((HTTP.REQ.HOSTNAME.STARTSWITH(\"cvpn\") && !HTTP.REQ.HOSTNAME.EQUALS_ANY(\"ns_cvpn_vsvr_fqdn\")) || http.req.url.path.eq(\"/\") || http.req.url.set_text_mode(IGNORECASE).startswith_any(\"aaa_url\") || (http.req.url.path.set_text_mode(IGNORECASE).startswith(\"/\") && (http.req.url.path.get(1).set_text_mode(IGNORECASE).equals_any(\"aaa_path\") || (http.req.url.path.get(1).set_text_mode(IGNORECASE).eq(\"p\") && (http.req.url.path.get(2).set_text_mode(IGNORECASE).eq(\"u\") || http.req.url.path.get(2).set_text_mode(IGNORECASE).eq(\"a\"))) || (http.req.url.path.get(1).set_text_mode(IGNORECASE).eq(\"BASE\") && http.req.url.path.get(2).eq(\"%08X%08X\")))) || http.REQ.HEADER(\"upgrade\").VALUE(0).SET_TEXT_MODE(IGNORECASE).CONTAINS(\"websocket\")) || http.req.url.path.contains(\"/metadata/samlidp/\") || http.req.url.path.contains(\"/metadata/samlsp/\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression is_aoservice "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"AO-Service\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_non_get "METHOD != GET" -comment "Http request method is not get"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_non_get_adv "HTTP.REQ.METHOD.EQ(GET).NOT" -comment "Http request method is not get"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_cachecontrol_nostore "HEADER Cache-Control CONTAINS \'no-store\'" -comment "Http request header Cache-Control contains no-store"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_cachecontrol_nostore_adv "HTTP.REQ.HEADER(\"Cache-Control\").CONTAINS(\"no-store\")" -comment "Http request header Cache-Control contains no-store"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_cachecontrol_nocache "HEADER Cache-Control CONTAINS \'no-cache\'" -comment "Http request header Cache-Control contains no-cache"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_cachecontrol_nocache_adv "HTTP.REQ.HEADER(\"Cache-Control\").CONTAINS(\"no-cache\")" -comment "Http request header Cache-Control contains no-cache"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_header_pragma "HEADER Pragma CONTAINS \'no-cache\'" -comment "Http request header Pragma contains no-cache"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_header_pragma_adv "HTTP.REQ.HEADER(\"Pragma\").CONTAINS(\"no-cache\")" -comment "Http request header Pragma contains no-cache"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_header_cookie "HEADER Cookie EXISTS" -comment "Http request header Cookie exists"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_header_cookie_adv "HTTP.REQ.HEADER(\"Cookie\").EXISTS" -comment "Http request header Cookie exists"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_cgi "URL == \'/*.cgi\'" -comment "File extension is cgi in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_cgi_adv "HTTP.REQ.URL.SUFFIX.EQ(\"cgi\")" -comment "File extension is cgi in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_asp "URL == \'/*.asp\'" -comment "File extension is asp in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_asp_adv "HTTP.REQ.URL.SUFFIX.EQ(\"asp\")" -comment "File extension is asp in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_exe "URL == \'/*.exe\'" -comment "File extension is exe in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_exe_adv "HTTP.REQ.URL.SUFFIX.EQ(\"exe\")" -comment "File extension is exe in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_cfm "URL == \'/*.cfm\'" -comment "File extension is cfm in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_cfm_adv "HTTP.REQ.URL.SUFFIX.EQ(\"cfm\")" -comment "File extension is cfm in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_ex "URL == \'/*.ex\'" -comment "File extension is ex in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_ex_adv "HTTP.REQ.URL.SUFFIX.EQ(\"ex\")" -comment "File extension is ex in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_shtml "URL == \'/*.shtml\'" -comment "File extension is stml in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_shtml_adv "HTTP.REQ.URL.SUFFIX.EQ(\"shtml\")" -comment "File extension is stml in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_htx "URL == \'/*.htx\'" -comment "File extension is htx in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_ext_htx_adv "HTTP.REQ.URL.SUFFIX.EQ(\"htx\")" -comment "File extension is htx in request URL"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add contentinspection action NOINSPECTION -type NOINSPECTION", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_cert_deviceid -authenticationSchema "LoginSchema/DeviceID_Cert.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_single_factor_deviceid -authenticationSchema "LoginSchema/SingleAuthDeviceID.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_dual_factor_deviceid -authenticationSchema "LoginSchema/DualAuthDeviceID.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_cert_single_factor_deviceid -authenticationSchema "LoginSchema/ClientCertSingleAuthDeviceID.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_cert_dual_factor_deviceid -authenticationSchema "LoginSchema/ClientCertDualAuthDeviceID.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_adal -authenticationSchema "LoginSchema/OnlyOAuthToken.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_cert_deviceid -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"NAC/1.0\")" -action lschema_cert_deviceid"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_single_factor_deviceid -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"NAC/1.0\")" -action lschema_single_factor_deviceid"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_dual_factor_deviceid -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"NAC/1.0\")" -action lschema_dual_factor_deviceid"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_cert_single_factor_deviceid -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"NAC/1.0\")" -action lschema_cert_single_factor_deviceid"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_cert_dual_factor_deviceid -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"NAC/1.0\")" -action lschema_cert_dual_factor_deviceid"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_adal -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"OAuth/2.0\")" -action lschema_adal"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Invalid password
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add ssl certKey ns-sftrust-certificate -cert ns-sftrust.cert -key ns-sftrust.key"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_dual_factor_builtin -authenticationSchema "LoginSchema/DualAuth.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such file
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchema lschema_dual_factor_flipped_builtin -authenticationSchema "LoginSchema/DualAuth_Flipped.xml""
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_dual_factor_builtin -rule true -action lschema_dual_factor_builtin"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Action does not exist
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add authentication loginSchemaPolicy lschema_dual_factor_flipped_builtin -rule true -action lschema_dual_factor_flipped_builtin"
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn sessionAction SETVPNPARAMS_ACT -defaultAuthorizationAction DENY -SSO OFF", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add aaa preauthenticationaction SET_PREAUTHPARAMS_ACT ALLOW", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_cmp_content_type -rule ns_content_type -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_cmp_msapp -rule "(ns_msie&&(ns_msword||ns_msexcel||ns_msppt))" -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_cmp_mscss -rule "(ns_msie&&ns_css)" -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_nocmp_mozilla_47 -rule "ns_mozilla_47&&ns_css" -resAction NOCOMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_nocmp_xml_ie -rule "(ns_msie&&ns_xmldata)" -resAction NOCOMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn sessionPolicy SETVPNPARAMS_POL ns_true SETVPNPARAMS_ACT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn sessionPolicy SETVPNPARAMS_ADV_POL true SETVPNPARAMS_ACT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add aaa preauthenticationpolicy SETPREAUTHPARAMS_POL ns_true SET_PREAUTHPARAMS_ACT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-non-get -rule NS_NON_GET -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-cache-control -rule "(NS_CACHECONTROL_NOSTORE || NS_CACHECONTROL_NOCACHE  || NS_HEADER_PRAGMA)" -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-dynamic-url -rule "(NS_EXT_CGI || NS_EXT_ASP || NS_EXT_EXE ||NS_EXT_CFM  || NS_EXT_EX ||NS_EXT_SHTML  ||NS_EXT_HTX) ||( NS_URL_PATH_CGIBIN|| NS_URL_PATH_EXEC ||NS_URL_PATH_BIN)" -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-urltokens -rule NS_URL_TOKENS -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-cookie -rule "(NS_HEADER_COOKIE && NS_EXT_NOT_GIF &&  NS_EXT_NOT_JPEG)" -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-non-get-adv -rule "HTTP.REQ.METHOD.EQ(GET).NOT" -action ORIGIN -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-cache-control-adv -rule "((HTTP.REQ.CACHE_CONTROL.IS_NO_STORE) || (HTTP.REQ.CACHE_CONTROL.IS_NO_CACHE) || (HTTP.REQ.HEADER(\"Pragma\").CONTAINS(\"no-cache\")))" -action ORIGIN -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-dynamic-url-adv -rule "(HTTP.REQ.URL.ENDSWITH_ANY(\"ns_cr_dynamic_ext\") || (HTTP.REQ.URL.PATH.STARTSWITH_ANY(\"ns_cr_dynamic_path\")))" -action ORIGIN -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-urltokens-adv -rule "HTTP.REQ.URL.REGEX_MATCH(re/[?!=]/)" -action ORIGIN -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cr policy bypass-cookie-adv -rule "((HTTP.REQ.HEADER(\"Cookie\").EXISTS) && (!(HTTP.REQ.URL.ENDSWITH(\".gif\"))) && (!(HTTP.REQ.URL.ENDSWITH(\".jpeg\"))))" -action ORIGIN -builtin IMMUTABLE -feature CR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind cmp global ns_cmp_content_type -priority 10000 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind cmp global ns_cmp_msapp -priority 9000 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind cmp global ns_cmp_mscss -priority 8900 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind cmp global ns_nocmp_mozilla_47 -priority 8800 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind cmp global ns_nocmp_xml_ie -priority 8700 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind tunnel global ns_tunnel_nocmp -priority 0 -state ENABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName SETVPNPARAMS_POL -priority 65534", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName SETVPNPARAMS_ADV_POL -priority 65534", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:36 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind aaa global -policy SETPREAUTHPARAMS_POL -priority 65534", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set sc parameter -sessionLife 300 -vsr DEFAULT ", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set cmp parameter -cmpLevel optimal -quantumSize 57344 -serverCmp ON -minResSize 0 -cmpBypassPct 100 -cmpOnPush DISABLED -policyType CLASSIC -addVaryHeader DISABLED -externalCache NO", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add bot profile BOT_BYPASS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add bot policy NOPOLICY -rule true -profileName BOT_BYPASS -builtin IMMUTABLE", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add bot policy NOPOLICY-BOT -rule true -profileName BOT_BYPASS -builtin IMMUTABLE", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set bot settings -defaultProfile BOT_BYPASS -javaScriptName client.ns.js -sessionTimeout 900 -sessionCookieName citrix_bot_id -dfpRequestLimit 1000 -signatureAutoUpdate OFF -signatureUrl "https://nsbotsignatures.s3.amazonaws.com/BotSignatureMapping.json" -proxyPort 8080 -builtin MODIFIABLE -trapURLAutoGenerate OFF -trapURLInterval 3600 -trapURLLength 32", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set sc parameter -sessionLife 300 -vsr DEFAULT ", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Specified parameters are not applicable for this type of SSL profile.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add ssl profile ns_default_ssl_profile_backend -sslProfileType BackEnd -eRSA DISABLED -sessReuse ENABLED -sessTimeout 300"
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Invalid rule.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add cache policy _cacheVPNStaticObjects -rule "HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH_ANY(\"vpn_cache_dirs\") && !HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH(\"/vpns/portal/\") && !HTTP.REQ.URL.CONTAINS(\"/vpn/pluginlist.xml\")" -action CACHE -storeInGroup loginstaticobjects"
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Invalid rule.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add cache policy _cacheTCVPNStaticObjects -rule "CLIENT.SSLVPN.MODE.EQ(\"CVPN_TRANSPARENT\")&&HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH_ANY(\"vpn_cache_dirs\") && !HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH(\"/vpns/portal/\")" -action CACHE -storeInGroup TCVPNloginstaticobjects"
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Invalid rule.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "add cache policy _cacheOCVPNStaticObjects -rule "CLIENT.SSLVPN.MODE.EQ(\"CVPN_OPAQUE\")&&HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH_ANY(\"vpn_cache_dirs\") && !HTTP.REQ.URL.PATH_AND_QUERY.STARTSWITH(\"/vpns/portal/\")" -action CACHE -storeInGroup OCVPNloginstaticobjects"
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): No such resource
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch():   Failing command: "set aaa preauthenticationparameter -preauthenticationaction ALLOW -rule ns_true"
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn parameter -splitDns BOTH -sessTimeout 30 -clientSecurityLog OFF -splitTunnel OFF -localLanAccess OFF -rfc1918 OFF -killConnections OFF -transparentInterception OFF -defaultAuthorizationAction DENY -proxyLocalBypass DISABLED -clientCleanupPrompt ON -forceCleanup none -clientConfiguration all -SSO OFF -ssoCredential PRIMARY -windowsAutoLogon OFF -useMIP NS -useIIP NOSPILLOVER -icaProxy OFF -ClientChoices OFF -clientlessVpnMode OFF -clientlessModeUrlEncoding OPAQUE -clientlessPersistentCookie DENY -encryptCsecExp ENABLED -appTokenTimeout 100 -mdxTokenTimeout 10 -SecureBrowse ENABLED -WindowsPluginUpgrade Always -MacPluginUpgrade Always -LinuxPluginUpgrade Always -iconWithReceiver OFF -AdvancedClientlessVpnMode DISABLED -backendServerSni DISABLED -backendCertValidation DISABLED", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_adv_cmp_content_type -rule "HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"text\")" -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_adv_cmp_msapp -rule "ns_msie_adv && (HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"application/msword\") || HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"application/vnd.ms-excel\") || HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"application/vnd.ms-powerpoint\"))" -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_adv_cmp_mscss -rule "ns_msie_adv && HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"text/css\")" -resAction COMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_adv_nocmp_mozilla_47 -rule "HTTP.REQ.HEADER(\"User-Agent\").CONTAINS(\"Mozilla/4.7\") && HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"text/css\")" -resAction NOCOMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add cmp policy ns_adv_nocmp_xml_ie -rule "ns_msie_adv && HTTP.RES.HEADER(\"Content-Type\").CONTAINS(\"text/xml\")" -resAction NOCOMPRESS", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add ipsec profile ns_ipsec_default_profile -psk 1ae7561a324325381b46f810b2992ec93ccee8810c6c132bbfcae60b8b071669 -encrypted -encryptmethod ENCMTHD_3 -kek -suffix 2022_05_07_12_21_35", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn parameter -clientversions refresh", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_default_url_encode_act clientless_vpn_encode url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_src_qurl_encode_act clientless_vpn_encode "url.query.value(\"Source\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_default_url_decode_act clientless_vpn_decode_all "http.req.body(4096)" -pattern "re~(?:=)((https?%3(a|A)%2(f|F))?%2(f|F)([a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\={}-]+))~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_body_dest_rw_act clientless_vpn_decode "http.req.body(4096).skip(12)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_desthdr_rw_act clientless_vpn_decode "http.req.header(\"destination\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_body_href_rw_act clientless_vpn_decode_all "http.req.body(4096)" -pattern "re~(?:href[ a-zA-Z0-9:\\=\"]*>)((https?:/)?/[a-zA-Z0-9:/\\._%\\!,\\*\\(\\)\\]\\[@\\#$\\?\\=\\&\\|\\^\\`{};-]+)(:?</)~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_body_range_rw_act clientless_vpn_decode q{http.req.body(4096).after_str("<range type=\"url\" rows=\"25\">").before_str("</range>")}", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_body_val_rw_act clientless_vpn_decode_all "http.req.body(10240)" -pattern q<re~(?:=\")((https?:/)?/[a-zA-Z0-9:/\._%\!,\*\(\)\]\[@\#$\?\=\&\|\^\`{};-]+)(:?\")~>", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_js_cac_rw_act delete_all TEXT -pattern "re~document\\.execCommand\\(\"ClearAuthenticationCache\",\\s*\"false\"\\)~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_ct_rw_act replace "http.res.header(\"Content-Type\").regex_select(re$[\\w/]+$)" "\"text/javascript\""", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_q_iisadmpwd_act clientless_vpn_decode "http.req.url.after_str(\"?\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_js_EcpUgeVD_rw_act replace_all TEXT "\"if($1[2]==\'http\' || $1[2]==\'https\'){$2+=$1[1]+\'/\'+$1[2]+\'/\'+$1[3]+\'/\'+$1[4]+\'/\'+EcpUrl.get_$0();}else{$2+=$1[1]+\'/\'+$1[2]+\'/\'+$1[3]+\'/\'+EcpUrl.get_$0();}\"" -pattern "re~\\$2\\s*\\+\\s*=\\s*\\$1\\[1\\]\\s*\\+\\s*\'/\'\\s*\\+\\s*EcpUrl\\.get_\\$0\\(\\);~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_js_EcpUgeVD_rw_act1 replace_all TEXT "\"if($1[2]==\'http\' || $1[2]==\'https\'){$2+=$1[1]+\'/\'+$1[2]+\'/\'+$1[3]+\'/\'+$1[4]+\'/\'+EcpUrl.get_$1();}else{$2+=$1[1]+\'/\'+$1[2]+\'/\'+$1[3]+\'/\'+EcpUrl.get_$1();}\"" -pattern "re~\\$2\\s*\\+\\s*=\\s*\\$1\\[1\\]\\s*\\+\\s*\'/\'\\s*\\+\\s*EcpUrl\\.get_\\$1\\(\\);~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_js_EcpUgeVD_rw_act2 replace_all TEXT q<"{if(n[2].indexOf(\'http\')!=-1){t+=n[1]+\'/\'+n[2]+\'/\'+n[3]+\'/\'+n[4]+\'/\'+EcpUrl.get_$89()}else{t+=n[1]+\'/\'+n[2]+\'/\'+n[3]+\'/\'+EcpUrl.get_$89()}}"> -pattern q{re~t\+=n\[1\]\+\"/\"\+EcpUrl\.get_\$89\(\);~}", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_owa_js_boot.0.mouse.act replace_all TEXT "\"this.$3lB=n;if(n.startsWith(\'/cvpn/http\')){n=n.substr(n.split(\'/\',4).join(\'/\').length);}else if(n.startsWith(\'/cvpn\')){n=n.substr(n.split(\'/\',3).join(\'/\').length);}\"" -pattern "re~this\\.\\$3lB=n;~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_src_rw_act clientless_vpn_decode "http.req.url.query.value(\"Source\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_fn_rw_act clientless_vpn_decode "http.req.url.query.value(\"FileName\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_rf_rw_act clientless_vpn_decode "http.req.url.query.value(\"RootFolder\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_su_rw_act clientless_vpn_decode "http.req.url.query.value(\"SourceUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_nu_rw_act clientless_vpn_decode "http.req.url.query.value(\"NextUsing\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_ofr_rw_act clientless_vpn_decode "http.req.url.query.value(\"owsfileref\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_ru_rw_act clientless_vpn_decode "http.req.url.query.value(\"ReturnUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_lvu_rw_act clientless_vpn_decode "http.req.url.query.value(\"ListViewURL\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_loc_rw_act clientless_vpn_decode "http.req.url.query.value(\"location\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_dlg_rw_act clientless_vpn_decode "http.req.url.query.value(\"dlg\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_url_rw_act clientless_vpn_decode "http.req.url.query.value(\"URL\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_url2_rw_act clientless_vpn_decode "http.req.url.query.value(\"url\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_url1_rw_act clientless_vpn_decode "http.req.url.query.value(\"Url\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_masterUrl_rw_act clientless_vpn_decode "http.req.url.query.value(\"masterUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_themeUrl_rw_act clientless_vpn_decode "http.req.url.query.value(\"themeUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_imageUrl_rw_act clientless_vpn_decode "http.req.url.query.value(\"imageUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_q_url_fontSchemeUrl_rw_act clientless_vpn_decode "http.req.url.query.value(\"fontSchemeUrl\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_req_query_rw_0_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(0)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ct_rw_act replace "http.res.header(\"Content-Type\").regex_select(re$[\\w/]+$)" "\"text/javascript\""", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_body_decode_act clientless_vpn_decode_all "http.req.body(20480)" -pattern "re~(?:=|\\+|%5b|>|%C2%A7|%c2%a7|\"|%3E)((https?(:/|%3a%2f|%3A%2F))?(/|%2f|%2F)([a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\={}-]+))~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_body_decode_act1 clientless_vpn_decode_all "http.req.body(10240)" -pattern "re~(?:name=|name%3d|%5b|initialUrl=)(cvpn(/|%2f|%2F)([a-zA-Z0-9:\\._%/\\!,\\*\\(\\)\\]\\[@\\#$\\?\\={}-]+))~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_body_decode_form clientless_vpn_decode_all "http.req.body(http.req.content_length)" -pattern q<re~(?:"\r\n\r\n|=)((https?(:/|%3A%2F|%3a%2f))?(/|%2f|%2F)([a-zA-Z0-9: \._%/\!,\*\(\)\]\[@\#$\={}-]+))(?:\?|\&|\r\n|\b)~>", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_cac_rw_act delete_all TEXT -pattern "re~document\\.execCommand\\(\"ClearAuthenticationCache\"\\)~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_rurl_rw_act insert_after_all TEXT q<"var _ns_rurl=location.href.substr(location.href.indexOf('/cvpn'));\r\nif(_ns_rurl.substr(6,4) == 'http'){_ns_rurl=_ns_rurl.substr(0, _ns_rurl.indexOf('/', _ns_rurl.indexOf('/',6)+1))}else{_ns_rurl=_ns_rurl.substr(0, _ns_rurl.indexOf('/',6))}\r\n"> -pattern q/re~var\sbrowseris\s?=\s?new\sBrowseris\(\);\r\n~/", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_source_rw_act insert_after_all TEXT q|"if(b&&b.startsWith(\'http\')&&(b.indexOf(\'cvpn/\')<0)){z=document.location.href.split(\'/\');c=\'/\'+b.split(\'/\').splice(3).join(\'/\');x=5;if(z[4].startsWith(\'http\'))x++;b=z.splice(0,x).join(\'/\')+c;}"| -pattern q/re~b=GetUrlKeyValue\(\"Source\"\);~/", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_source_rw_act2 insert_before_all TEXT q<"if(!d.startsWith(\'/cvpn\')&&!d.startsWith(\'http\')){w=window.location.pathname;x=w.split(\'/\');y=\'/\'+x[1]+\'/\'+x[2];if(x[2].startsWith(\'http\'))y=y+\'/\'+x[3];d=y+d;}"> -pattern "re~STSNavigate\\(d\\)~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_slwp_rw_act replace_all TEXT "\"=(slwp_webUrl || _ns_rurl)+\"" -pattern "re~=slwp_webUrl\\+~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_xcmp_vti_act replace "http.res.header(\"Content-Type\").regex_select(re$[\\w/-]+$)" "\"text/xml\""", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_du_rw_act replace_all TEXT "\"=this.CurrentWebBaseUrl+\'/_layouts/\'+dialogName\"" -pattern "re~=this.CurrentWebBaseUrl\\+dialogUrl~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_checkout_rw_act insert_after_all TEXT q{"\'/\'+window.location.pathname.split(\'/\')[1]+\'/\'+window.location.pathname.split(\'/\')[2]+\'/\'+window.location.pathname.split(\'/\')[3]+"} -pattern q{re~a.substr\(0,3\).toLowerCase\(\)==\"%2f\"\)a=window.location.protocol\+\"//\"\+window.location.host\+~}", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_guk_rw_act insert_after_all TEXT "\"var k=keyValue;if(k.indexOf(\'cvpn/\')<0&&k.indexOf(\'cvpn%2F\')<0&&k.indexOf(\'cvpn%2f\')<0){if(k.substr(0,4)==\'http\'){k=location.protocol+\'/\'+\'/\'+location.host+_ns_rurl+k.substr(k.indexOf(\'/\',k.indexOf(\':/\')+3));}else if(k!=\'\'){k=_ns_rurl+k;}}keyValue=k;\n\"" -pattern q/re~keyValue\s?\=\s?url\.substring\(ndx\s?\+\s?keyName\.length\s?\+\s?2,\sndx2\);\r\n\s*}\r\n~/", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_hdr_sa_rw_act clientless_vpn_decode "http.req.header(\"soapaction\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ui_dia_1_rw_act replace_all TEXT "\"if(i.indexOf(\'http\')==-1){\"+\"i\"+\"=\"+\"\'/\'\"+\"+\"+\"window.location.pathname.split(\'/\')[1]\"+\"+\"+\"\'/\'\"+\"+\"+\"window.location.pathname.split(\'/\')[2]\"+\"+\"+\"\'/\'\"+\"+\"+\"window.location.pathname.split(\'/\')[3]\"+\"+\"+\"i;} else{ if((i.indexOf(\'https://\')!=-1)&&(i.indexOf(\'/cvpn/\' )==-1)){i=\'/cvpn/\'+i.replace(\'https://\',\'https/\');}} i=SP.UI.Dialog.$1h(i);\"" -search "regex(re~(i[ ]*)(=)[ ]*(SP.UI.Dialog.\\$1h\\(i\\)\\;)~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ui_dia_2_rw_act insert_before_all TEXT "\"var p=0,len=0,j=0;if(i.startsWith(\'/cvpn\')){p=3;if(i.substr(5).startsWith(\'/http\'))p=4;}if(p!=0){for(;j<i.length&&len<p;j++)if(i.charAt(j)==\'/\')++len;s=i.substr(0,j);i=s.substr(0,s.length-1)+i.split(s).join(\'/\');}\"" -search "regex(re~(g.className[ ]*=)([ ]*\"ms-dlgFrame\"[ ]*\\;)~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ui_dia_3_act insert_before_all TEXT q<"if(i.lastIndexOf(\'/cvpn/\')!=i.indexOf(\'/cvpn/\')){i=i.substr(i.lastIndexOf(\'/cvpn/\'));}"> -search "regex(re~(g.)(setAttribute\\(\"src\"[ ]*,[ ]*i\\);)~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ribbon_js_rw_act replace_all TEXT "\"df=SP.Utilities.HttpUtility.urlPathEncode(GetSource());df=decodeURI(df); d+=df;\"" -search "regex(re~(d[ ]*\\+=)([ ]*SP.Utilities.HttpUtility.urlPathEncode\\(GetSource\\([ ]*\\)\\)\\;)~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_runtime_rw_act replace_all TEXT "\"if(this.$T_0.indexOf(\'/cvpn/\')==-1){this.$T_0 =\'/cvpn\'+\'/\'+ window.location.pathname.split(\'/\')[2]+\'/\'+window.location.pathname.split(\'/\')[3]+this.$T_0;}this.$T_0=this.$0_0.getRequestUrl(this.$T_0)\"" -search "regex(re~(this\\.\\$T_0[ ]*)=([ ]*this\\.\\$0_0\\.getRequestUrl\\(this\\.\\$T_0\\))~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_ribbon_js_rw_act_1 replace_all TEXT "\"df=SP.Utilities.HttpUtility.urlPathEncode(GetSource());df=decodeURI(df); c+=df;\"" -search "regex(re~(c[ ]*\\+=)([ ]*SP.Utilities.HttpUtility.urlPathEncode\\(GetSource\\([ ]*\\)\\)\\;)~)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_contentType_act replace "http.res.header(\"Content-Type\").regex_select(re$[\\w-/]+$)" "\"text/javascript\""", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_0_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(0)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_1_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(1)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_2_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(2)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_3_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(3)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_4_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(4)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_5_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(5)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_query_rw_6_act clientless_vpn_decode "HTTP.REQ.URL.QUERY.VALUE(6)"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_v2_req_body_decode_act clientless_vpn_decode_all "http.req.body(4096)" -pattern "re~(?:=|=\"|\\+|%5b|>|%C2%A7|%c2%a7|\"|%3E|%22|%27|\\?|href[ a-zA-Z0-9:\\=\"]*>)((https?(:/|%3a%2f|%3A%2F))?(/|%2f|%2F)([a-zA-Z0-9:\\._%]+))~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_0_pol "HTTP.REQ.URL.QUERY.VALUE(0).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_0_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_1_pol "HTTP.REQ.URL.QUERY.VALUE(1).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_1_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_2_pol "HTTP.REQ.URL.QUERY.VALUE(2).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_2_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_3_pol "HTTP.REQ.URL.QUERY.VALUE(3).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_3_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_4_pol "HTTP.REQ.URL.QUERY.VALUE(4).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_4_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_5_pol "HTTP.REQ.URL.QUERY.VALUE(5).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_5_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_query_rw_6_pol "HTTP.REQ.URL.QUERY.VALUE(6).STARTSWITH(\"http\")" ns_cvpn_v2_req_query_rw_6_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_req_body_decode_pol "http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0) && http.req.header(\"Content-Type\").exists && (HTTP.REQ.HEADER(\"Content-Type\").REGEX_MATCH(re$text/[0-9a-zA-z+-]+$) || HTTP.REQ.HEADER(\"Content-Type\").REGEX_SELECT(re$application/[0-9a-zA-z+-]+$).ENDSWITH_ANY(\"ns_cvpn_v2_application_content_type_end\"))" ns_cvpn_v2_req_body_decode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_bypass_url_pol "(text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || text.set_text_mode(ignorecase).startswith(\"http\")) && !text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_bypass_domains\")" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_v2_inet_url_pol "(text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || text.set_text_mode(ignorecase).startswith(\"http\")) && text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_inet_domains\")" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite action ns_cvpn_sp_js_localStorage_act insert_after_all TEXT q/"if(!IsNullOrUndefined(window.localStorage))window.localStorage.removeItem(\'SPSuiteLinksJson\');"/ -pattern "re~function OpenSuiteLinksJson\\(\\){~"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_localStorage_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"init.js\")" ns_cvpn_sp_js_localStorage_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_contentType_pol "HTTP.REQ.URL.CONTAINS(\"jsgrid.js\") || HTTP.REQ.URL.CONTAINS(\"aspx?AjaxDelta\") || HTTP.REQ.URL.PATH.ENDSWITH(\"clienttemplates.js\") || HTTP.REQ.URL.PATH.ENDSWITH(\"init.js\") || http.req.url.path.endswith(\"clientforms.js\") || HTTP.REQ.URL.CONTAINS(\"userdisp.aspx?ID\") || HTTP.REQ.URL.CONTAINS(\"listform.aspx?PageType\") || HTTP.REQ.URL.CONTAINS(\"listform.aspx?ListId\") || ((HTTP.REQ.URL.CONTAINS(\"settings.aspx?Source\") || HTTP.REQ.URL.CONTAINS(\"Workflow.aspx\") || HTTP.REQ.URL.CONTAINS(\"osssearchresults.aspx\") || HTTP.REQ.URL.CONTAINS(\"MySite.aspx\") || HTTP.REQ.URL.CONTAINS(\"BackLinks.aspx\") || HTTP.REQ.URL.CONTAINS(\"listedit.aspx\")|| HTTP.REQ.URL.CONTAINS(\"ListEdit.aspx\") || HTTP.REQ.URL.CONTAINS(\"designbuilder.aspx?masterUrl\") || HTTP.REQ.URL.CONTAINS(\"people.aspx?MembershipGroupId\") || HTTP.REQ.URL.CONTAINS(\"ViewType.aspx\") || HTTP.REQ.URL.CONTAINS(\"ManageContentType.aspx?ctype\") || HTTP.REQ.URL.CONTAINS(\"ConfigureResultType.aspx?ID\") || HTTP.REQ.URL.CONTAINS(\"wrksetng.aspx\")) && HTTP.REQ.URL.CONTAINS(\"AjaxDelta\"))" ns_cvpn_sp_contentType_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_mobile_pol_1 "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.ribbon.js\")" ns_cvpn_sp_ribbon_js_rw_act_1", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_ui_dia_3_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.ui.dialog.js\")" ns_cvpn_sp_ui_dia_3_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_mobile_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.ribbon.js\")" ns_cvpn_sp_ribbon_js_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_ui_dia_1_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.ui.dialog.js\")" ns_cvpn_sp_ui_dia_1_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_ui_dia_2_rw_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.ui.dialog.js\")" ns_cvpn_sp_ui_dia_2_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_default_bypass_url_pol "text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || (text.set_text_mode(ignorecase).startswith(\"http\") && !text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_bypass_domains\"))" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_default_inet_url_pol "text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || (text.set_text_mode(ignorecase).startswith(\"http\") && text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_inet_domains\"))" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_bypass_url_pol "text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || (text.set_text_mode(ignorecase).startswith(\"http\") && !text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_bypass_domains\"))" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_inet_url_pol "text.set_text_mode(ignorecase).startswith_any(\"ns_cvpn_default_rel_urls_set\") || (text.set_text_mode(ignorecase).startswith(\"http\") && text.set_text_mode(backslash_encoded).set_text_mode(urlencoded).prefix(1024).decode_using_text_mode.typecast_http_url_t.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_default_inet_domains\"))" ns_cvpn_default_url_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_src_qurl_pol1 "url.query.value(\"Source\").length.gt(0)" ns_cvpn_sp_src_qurl_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_src_qurl_pol2 "url.query.value(\"Source\").length.gt(0)" ns_cvpn_sp_src_qurl_encode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_q_body_dest_rw_pol "http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0)" ns_cvpn_default_url_decode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_body_dest_rw_pol "http.req.method.eq(POST) && http.req.body(1024).startswith(\"destination=\")" ns_cvpn_owa_q_body_dest_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_desthdr_rw_pol "http.req.header(\"destination\").exists" ns_cvpn_owa_q_desthdr_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_body_href_rw_pol "http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0)" ns_cvpn_owa_q_body_href_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_body_range_rw_pol "http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0)" ns_cvpn_owa_q_body_range_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_body_val_rw_pol "http.req.method.eq(POST) && http.req.body(1024).startswith(\"<params>\")" ns_cvpn_owa_q_body_val_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_s_body_ev_owa_pol "http.req.url.path.endswith(\"ev.owa\")&&!http.req.url.path_and_query.endswith(\"oeh=1&ns=CalendarView&ev=Move\")&&!http.req.url.path_and_query.endswith(\"oeh=1&ns=CalendarView&ev=Delete\")&&!http.req.url.query.startswith(\"UA=0&oeh=1&ns=PendingRequest&ev=PendingNotificationRequest\")&&!http.req.url.path_and_query.endswith(\"oeh=1&ns=MsgVLV2&ev=LoadFresh\")&&!http.req.url.path_and_query.endswith(\"oeh=1&ns=MsgVLV2&ev=LoadNext\")&&!http.req.url.path_and_query.endswith(\"oeh=1&ns=MsgVLV2&ev=LoadPrevious\")" ns_cvpn_owa_ct_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_js_cac_rw_pol TRUE ns_cvpn_owa_js_cac_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_ct_rw_pol " http.req.url.endswith(\"mouse.js\") || http.req.url.path.endswith(\"client.core.framework.js\") || http.req.url.path.endswith(\"client.boot.viewmodels.js\") || http.req.url.path.endswith(\"uglobal.js\") " ns_cvpn_owa_ct_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_q_iisadmpwd_pol "http.req.url.path.contains(\"iisadmpwd\")" ns_cvpn_owa_q_iisadmpwd_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_js_EcpUgeVD_rw_pol "http.req.url.path.endswith(\"common.js\")" ns_cvpn_owa_js_EcpUgeVD_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_js_EcpUgeVD_rw_pol1 "http.req.url.path.endswith(\"common.js\")" ns_cvpn_owa_js_EcpUgeVD_rw_act1", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_js_EcpUgeVD_rw_pol2 "http.req.url.path.endswith(\"common.js\")" ns_cvpn_owa_js_EcpUgeVD_rw_act2", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_owa_js_boot.0.mouse.pol "http.req.url.path.endswith(\"boot.0.mouse.js\")" ns_cvpn_owa_js_boot.0.mouse.act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_url2_rw_pol "http.req.url.query.value(\"url\").length.gt(0)" ns_cvpn_sp_q_url_url2_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_src_rw_pol "http.req.url.query.value(\"Source\").length.gt(0)" ns_cvpn_sp_q_url_src_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_fn_rw_pol "http.req.url.query.value(\"FileName\").length.gt(0)" ns_cvpn_sp_q_url_fn_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_rf_rw_pol "http.req.url.query.value(\"RootFolder\").length.gt(0)" ns_cvpn_sp_q_url_rf_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_su_rw_pol "http.req.url.query.value(\"SourceUrl\").length.gt(0)" ns_cvpn_sp_q_url_su_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_nu_rw_pol "http.req.url.query.value(\"NextUsing\").length.gt(0)" ns_cvpn_sp_q_url_nu_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_ofr_rw_pol "http.req.url.query.value(\"owsfileref\").length.gt(0)" ns_cvpn_sp_q_url_ofr_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_ru_rw_pol "http.req.url.query.value(\"ReturnUrl\").length.gt(0)" ns_cvpn_sp_q_url_ru_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_lvu_rw_pol "http.req.url.query.value(\"ListViewURL\").length.gt(0)" ns_cvpn_sp_q_url_lvu_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_loc_rw_pol "http.req.url.query.value(\"location\").length.gt(0)" ns_cvpn_sp_q_url_loc_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_dlg_rw_pol "http.req.url.query.value(\"dlg\").length.gt(0)" ns_cvpn_sp_q_url_dlg_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_url_rw_pol "http.req.url.query.value(\"URL\").length.gt(0)" ns_cvpn_sp_q_url_url_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_url1_rw_pol "http.req.url.query.value(\"Url\").length.gt(0)" ns_cvpn_sp_q_url_url1_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_masterUrl_rw_pol "http.req.url.query.value(\"masterUrl\").length.gt(0)" ns_cvpn_sp_q_url_masterUrl_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_themeUrl_rw_pol "http.req.url.query.value(\"themeUrl\").length.gt(0)" ns_cvpn_sp_q_url_themeUrl_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_imageUrl_rw_pol "http.req.url.query.value(\"imageUrl\").length.gt(0)" ns_cvpn_sp_q_url_imageUrl_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_url_fontSchemeUrl_rw_pol "http.req.url.query.value(\"fontSchemeUrl\").length.gt(0)" ns_cvpn_sp_q_url_fontSchemeUrl_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_req_query_rw_0_pol "HTTP.REQ.URL.QUERY.VALUE(0).STARTSWITH(\"http\")" ns_cvpn_req_query_rw_0_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_s_post_body_pol "http.req.method.eq(POST) && http.res.header(\"Content-Length\").exists && http.res.header(\"Content-Length\").value(0).typecast_num_t(decimal).lt(2500) && http.res.body(1024).contains_any(\"ns_cvpn_sp_res_pat\")" ns_cvpn_sp_ct_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_body_dest_rw_pol "http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0)" ns_cvpn_sp_body_decode_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_body_dest_rw_pol1 "http.req.url.path.endswith(\"author.dll\") && http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0)" ns_cvpn_sp_body_decode_act1", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_source_rw_pol "http.req.url.path.endswith(\"clientforms.js\")" ns_cvpn_sp_js_source_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_source_rw_pol2 "http.req.url.path.endswith(\"clientforms.js\")" ns_cvpn_sp_js_source_rw_act2", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_q_body_up_rw_pol "http.req.method.eq(POST) && http.req.header(\"Content-Length\").exists && http.req.header(\"Content-Length\").value(0).typecast_num_t(decimal).gt(0) && http.req.url.path.endswith_any(\"ns_cvpn_sp_body_decode_pat\")" ns_cvpn_sp_body_decode_form", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_cac_rw_pol TRUE ns_cvpn_sp_js_cac_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_du_rw_pol "http.req.url.path.endswith(\"AssetPickers.js\")" ns_cvpn_sp_js_du_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_checkout_rw_pol "http.req.url.path.endswith(\"core.js\")" ns_cvpn_sp_js_checkout_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_rurl_rw_pol "http.req.url.path.endswith(\"owsbrows.js\") || http.req.url.path.endswith(\"init.js\")" ns_cvpn_sp_js_rurl_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_slwp_rw_pol "http.req.url.path.endswith(\"cmssummarylinks.js\")" ns_cvpn_sp_js_slwp_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_xcmp_vti_pol "http.req.url.path.endswith(\"_vti_rpc\") || http.req.url.path.endswith(\"author.dll\")" ns_cvpn_sp_xcmp_vti_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_vgp_pol "http.req.url.path.endswith(\"ViewGroupPermissions.aspx\") && http.req.method.eq(POST) && http.res.body(10).contains(\"0|/\")" ns_cvpn_sp_ct_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_guk_rw_pol "http.req.url.path.endswith(\"ows.js\") || http.req.url.path.endswith(\"init.js\")" ns_cvpn_sp_js_guk_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_hdr_sa_rw_pol "http.req.header(\"soapaction\").exists" ns_cvpn_sp_hdr_sa_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policy ns_cvpn_sp_js_runtime_rw_pol "HTTP.REQ.URL.PATH.ENDSWITH(\"sp.runtime.js\")" ns_cvpn_sp_js_runtime_rw_act", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_default_url_label url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_default_inet_url_label url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_sp_url_label url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_sp_inet_url_label url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_default_req_rw_label clientless_vpn_req", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_owa_req_rw_label clientless_vpn_req", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_owa_res_rw_label clientless_vpn_res", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_owa_js_rw_label text", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_sp_req_rw_label clientless_vpn_req", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_sp_res_rw_label clientless_vpn_res", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_sp_js_rw_label text", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_v2_url_label url", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_v2_js_rw_label text", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_v2_req_rw_label clientless_vpn_req", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add rewrite policylabel ns_cvpn_v2_res_rw_label clientless_vpn_res", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_default_url_label ns_cvpn_default_bypass_url_pol 200000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_default_inet_url_label ns_cvpn_default_inet_url_pol 200000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_url_label ns_cvpn_sp_bypass_url_pol 200000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_url_label ns_cvpn_sp_src_qurl_pol1 210000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_inet_url_label ns_cvpn_sp_inet_url_pol 200000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_inet_url_label ns_cvpn_sp_src_qurl_pol2 210000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_default_req_rw_label ns_cvpn_q_body_dest_rw_pol 300000 END", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_desthdr_rw_pol 10000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_body_href_rw_pol 20000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_body_range_rw_pol 30000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_body_dest_rw_pol 40000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_body_val_rw_pol 50000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_req_rw_label ns_cvpn_owa_q_iisadmpwd_pol 60000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_res_rw_label ns_cvpn_owa_s_body_ev_owa_pol 10000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_res_rw_label ns_cvpn_owa_ct_rw_pol 30000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_js_rw_label ns_cvpn_owa_js_cac_rw_pol 10000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_js_rw_label ns_cvpn_owa_js_EcpUgeVD_rw_pol 20000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_js_rw_label ns_cvpn_owa_js_EcpUgeVD_rw_pol1 30000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_js_rw_label ns_cvpn_owa_js_EcpUgeVD_rw_pol2 40000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_owa_js_rw_label ns_cvpn_owa_js_boot.0.mouse.pol 50000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_src_rw_pol 250000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_fn_rw_pol 260000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_rf_rw_pol 270000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_su_rw_pol 280000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_nu_rw_pol 290000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_body_dest_rw_pol 300000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_body_up_rw_pol 310000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_ofr_rw_pol 320000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_ru_rw_pol 330000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_lvu_rw_pol 340000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_body_dest_rw_pol1 350000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_loc_rw_pol 360000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_dlg_rw_pol 370000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_url_rw_pol 380000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_hdr_sa_rw_pol 390000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_url2_rw_pol 395000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_url1_rw_pol 400000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_masterUrl_rw_pol 410000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_themeUrl_rw_pol 420000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_imageUrl_rw_pol 430000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_sp_q_url_fontSchemeUrl_rw_pol 440000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_req_rw_label ns_cvpn_req_query_rw_0_pol 450000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_res_rw_label ns_cvpn_sp_s_post_body_pol 240000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_res_rw_label ns_cvpn_sp_xcmp_vti_pol 250000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_res_rw_label ns_cvpn_sp_js_vgp_pol 260000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_res_rw_label ns_cvpn_sp_contentType_pol 270000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_mobile_pol_1 24050 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_cac_rw_pol 250000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_du_rw_pol 260000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_rurl_rw_pol 270000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_slwp_rw_pol 280000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_guk_rw_pol 290000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_source_rw_pol 300000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_source_rw_pol2 305000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_checkout_rw_pol 310000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_ui_dia_1_pol 24000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_ui_dia_2_rw_pol 320000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_runtime_rw_pol 320050 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_ui_dia_3_pol 320060 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_mobile_pol 24001 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_0_pol 20000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_1_pol 21000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_2_pol 22000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_3_pol 23000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_4_pol 24000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_5_pol 25000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_query_rw_6_pol 26000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_req_rw_label ns_cvpn_v2_req_body_decode_pol 27000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_v2_url_label ns_cvpn_v2_bypass_url_pol 20000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind rewrite policylabel ns_cvpn_sp_js_rw_label ns_cvpn_sp_js_localStorage_pol 330000 NEXT", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessProfile ns_cvpn_default_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessProfile ns_cvpn_owa_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessProfile ns_cvpn_sp_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessProfile ns_cvpn_sp2013_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessProfile ns_cvpn_v2_default_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn clientlessAccessProfile ns_cvpn_default_profile -URLRewritePolicyLabel ns_cvpn_default_url_label -ReqHdrRewritePolicyLabel ns_cvpn_default_req_rw_label -RegexForFindingURLinJavaScript ns_cvpn_default_js_urls_regex -RegexForFindingURLinCSS ns_cvpn_default_css_urls_regex -RegexForFindingURLinXComponent ns_cvpn_default_xcmp_urls_regex -RegexForFindingURLinXML ns_cvpn_default_xml_urls_regex -RegexForFindingCustomURLs ns_cvpn_default_custom_regex -ClientConsumedCookies ns_cvpn_default_client_cookies", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn clientlessAccessProfile ns_cvpn_owa_profile -URLRewritePolicyLabel ns_cvpn_default_url_label -JavaScriptRewritePolicyLabel ns_cvpn_owa_js_rw_label -ReqHdrRewritePolicyLabel ns_cvpn_owa_req_rw_label -ResHdrRewritePolicyLabel ns_cvpn_owa_res_rw_label -RegexForFindingURLinJavaScript ns_cvpn_owa_js_urls_regex -RegexForFindingURLinCSS ns_cvpn_default_css_urls_regex -RegexForFindingURLinXComponent ns_cvpn_default_xcmp_urls_regex -RegexForFindingURLinXML ns_cvpn_owa_xml_urls_regex -RegexForFindingCustomURLs ns_cvpn_default_custom_regex -ClientConsumedCookies ns_cvpn_owa_client_cookies -requirePersistentCookie OFF", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn clientlessAccessProfile ns_cvpn_sp_profile -URLRewritePolicyLabel ns_cvpn_sp_url_label -JavaScriptRewritePolicyLabel ns_cvpn_sp_js_rw_label -ReqHdrRewritePolicyLabel ns_cvpn_sp_req_rw_label -ResHdrRewritePolicyLabel ns_cvpn_sp_res_rw_label -RegexForFindingURLinJavaScript ns_cvpn_sp_js_urls_regex -RegexForFindingURLinCSS ns_cvpn_default_css_urls_regex -RegexForFindingURLinXComponent ns_cvpn_default_xcmp_urls_regex -RegexForFindingURLinXML ns_cvpn_default_xml_urls_regex -RegexForFindingCustomURLs ns_cvpn_default_custom_regex -requirePersistentCookie ON", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn clientlessAccessProfile ns_cvpn_sp2013_profile -URLRewritePolicyLabel ns_cvpn_sp_url_label -JavaScriptRewritePolicyLabel ns_cvpn_sp_js_rw_label -ReqHdrRewritePolicyLabel ns_cvpn_sp_req_rw_label -ResHdrRewritePolicyLabel ns_cvpn_sp_res_rw_label -RegexForFindingURLinJavaScript ns_cvpn_sp2013_js_urls_regex -RegexForFindingURLinCSS ns_cvpn_default_css_urls_regex -RegexForFindingURLinXComponent ns_cvpn_default_xcmp_urls_regex -RegexForFindingURLinXML ns_cvpn_default_xml_urls_regex -RegexForFindingCustomURLs ns_cvpn_sp2013_custom_regex -requirePersistentCookie ON", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "set vpn clientlessAccessProfile ns_cvpn_v2_default_profile -URLRewritePolicyLabel ns_cvpn_v2_url_label -JavaScriptRewritePolicyLabel ns_cvpn_v2_js_rw_label -ReqHdrRewritePolicyLabel ns_cvpn_v2_req_rw_label -ResHdrRewritePolicyLabel ns_cvpn_v2_res_rw_label -RegexForFindingURLinJavaScript ns_cvpn_v2_fast_regex -RegexForFindingURLinCSS ns_cvpn_v2_css_urls_regex -RegexForFindingURLinXComponent ns_cvpn_v2_xcmp_urls_regex -RegexForFindingURLinXML ns_cvpn_v2_xml_urls_regex -RegexForFindingCustomURLs ns_cvpn_v2_fast_regex -requirePersistentCookie ON", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessPolicy ns_cvpn_default_policy true ns_cvpn_default_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessPolicy ns_cvpn_owa_policy "http.req.url.path.get(1).set_text_mode(ignorecase).eq(\"exchange\") || http.req.url.path.get(1).set_text_mode(ignorecase).eq(\"exchweb\") || http.req.url.path.get(1).set_text_mode(ignorecase).eq(\"owa\") || http.req.url.path.get(1).set_text_mode(ignorecase).eq(\"public\") || http.req.url.path.get(1).set_text_mode(ignorecase).eq(\"ecp\")" ns_cvpn_owa_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessPolicy ns_cvpn_sp_policy "http.req.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_sharepoint_hostnames\")" ns_cvpn_sp_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn clientlessAccessPolicy ns_cvpn_sp2013_policy "http.req.hostname.set_text_mode(ignorecase).contains_any(\"ns_cvpn_sp2013_hostnames\")" ns_cvpn_sp2013_profile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName ns_cvpn_default_policy -priority 100000", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName ns_cvpn_owa_policy -priority 95000", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName ns_cvpn_sp_policy -priority 96000", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -policyName ns_cvpn_sp2013_policy -priority 97000", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add ica accessprofile default_ica_accessprofile", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn portaltheme Default -basetheme Default", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn portaltheme Greenbubble -basetheme Greenbubble", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn portaltheme X1 -basetheme X1", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add vpn portaltheme RfWebUI -basetheme RfWebUI", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:37 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind vpn global -portaltheme RfWebUI", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_quic_abr_sni_whitelist", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_quic_abr_sni_whitelist googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_quic_abr_sni_whitelist c.youtube.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_quic_abr_sni_blacklist", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_quic_abr_sni_blacklist manifest.googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_quic_abr_sni_blacklist redirector.googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionaction DETECT_CLEARTEXT_PD -type clear_text_pd", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionaction DETECT_CLEARTEXT_ABR -type clear_text_abr", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionaction DETECT_ENCRYPTED_ABR -type encrypted_abr", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionaction TRIGGER_ENC_ABR_DETECTION -type trigger_enc_abr", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionaction TRIGGER_CT_ABR_BODY_DETECTION -type trigger_body_detection", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_body_detection -rule TRUE -action TRIGGER_CT_ABR_BODY_DETECTION", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_ismvExtension", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_ismvExtension ismv", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_ismvExtension isma", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_abr_netflix -rule "( HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH(\"application/octet-stream\") && HTTP.REQ.URL.SUFFIX.TO_LOWER.EQUALS_ANY(\"ns_videoopt_ismvExtension\"))" -action DETECT_CLEARTEXT_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_netflix_abr_host_ends", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_ends nflxvideo.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_netflix_abr_host_starts", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 37.77.184.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 108.175.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 198.45.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 198.38.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 185.2.223.146", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 209.91.216.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 37.77.186.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 2.127.239.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 74.85.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_netflix_abr_host_starts 24.139.136.", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_abr_netflix2 -rule "(HTTP.REQ.URL.QUERY.AFTER_STR(\"o=\").AFTER_STR(\"&v=\").CONTAINS(\"&e=\") || HTTP.REQ.HOSTNAME.SERVER.ENDSWITH_ANY(\"ns_videoopt_netflix_abr_host_ends\") || HTTP.REQ.HOSTNAME.SERVER.STARTSWITH_ANY(\"ns_videoopt_netflix_abr_host_starts\") )" -action DETECT_CLEARTEXT_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_abr_content_type", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "application/octet-stream"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "video/mp4"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "video/webm"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "audio/mp4"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "audio/webm"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_abr_content_type "video/flv"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_domain_eq", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_domain_eq docs.google.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_domain_eq c.docs.google.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_domain_eq c.youtube.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_domain_eq googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_abr_path_contains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/232/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/231/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/234/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/230/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/229/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/269/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/270/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/233/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/93/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/94/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/95/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/92/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/91/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/151/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/132/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/0/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/96/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/127/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/128/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/138/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/142/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/143/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/148/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/149/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/150/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/161/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/222/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/223/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/241/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/249/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/264/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/266/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/271/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/298/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/299/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/302/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/303/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/308/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/313/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_path_contains "/itag/315/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_abr_queryitag_eq", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 140", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 134", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 133", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 135", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 160", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 136", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 242", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 243", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 244", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 137", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 139", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 250", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 251", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 171", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 278", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 247", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 141", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 248", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 245", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 246", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 96", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 127", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 128", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 138", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 142", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 143", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 148", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 149", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 150", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 161", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 222", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 223", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 241", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 249", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 264", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 266", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 271", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 298", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 299", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 302", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 303", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 308", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 313", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_abr_queryitag_eq 315", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression rqd_is_yt_domain "HTTP.REQ.HOSTNAME.EQUALS_ANY(\"ns_videoopt_yt_domain_eq\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression rqd_is_yt_abr "(HTTP.REQ.URL.PATH.TO_LOWER.CONTAINS_ANY(\"ns_videoopt_yt_abr_path_contains\") || HTTP.REQ.URL.QUERY.VALUE(\"itag\").EQUALS_ANY(\"ns_videoopt_yt_abr_queryitag_eq\") || ((HTTP.REQ.URL.QUERY.VALUE(\"range\") ALT \"undefined\") != \"undefined\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_abr_youtube -rule "rqd_is_yt_domain && rqd_is_yt_abr && HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH_ANY(\"ns_videoopt_abr_content_type\")" -action DETECT_CLEARTEXT_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_otherpd_path_contains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/114/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/61/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/62/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/63/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/100/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/194/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/195/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/220/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_path_contains "/itag/221/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_otherpd_queryitag_eq", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 43", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 62", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 63", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 100", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 194", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 195", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 220", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_otherpd_queryitag_eq 221", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression rqd_is_yt_otherpd "(HTTP.REQ.URL.PATH.TO_LOWER.CONTAINS_ANY(\"ns_videoopt_yt_otherpd_path_contains\") || HTTP.REQ.URL.QUERY.VALUE(\"itag\").EQUALS_ANY(\"ns_videoopt_yt_otherpd_queryitag_eq\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_pd_youtube -rule "HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH(\"video/webm\") && rqd_is_yt_domain && rqd_is_yt_otherpd " -action DETECT_CLEARTEXT_PD", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_flash_content_type", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_flash_content_type "application/octet-stream"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_flash_content_type "video/3gpp"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_flash_content_type "video/mp4"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_flash_content_type "video/x-flv"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_pd_queryitag_eq", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 18", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 36", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 22", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 17", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 13", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 5", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 59", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 78", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 52", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 60", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 82", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 83", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 84", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_queryitag_eq 85", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_pd_path_contains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/18/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/36/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/22/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/17/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/5/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/59/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/52/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/60/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/82/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/83/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/84/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_pd_path_contains "/itag/85/"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression rqd_is_yt_pd_1 "(HTTP.REQ.URL.PATH.TO_LOWER.CONTAINS_ANY(\"ns_videoopt_yt_pd_path_contains\") || HTTP.REQ.URL.QUERY.VALUE(\"itag\").EQUALS_ANY(\"ns_videoopt_yt_pd_queryitag_eq\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_pd_youtube2 -rule "rqd_is_yt_domain && rqd_is_yt_pd_1 &&  HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH_ANY(\"ns_videoopt_yt_flash_content_type\")" -action DETECT_CLEARTEXT_PD", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_yt_media_url_pattern", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_media_url_pattern "youtube.com/videoplayback"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_yt_media_url_pattern "googlevideo.com/videoplayback"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_pd_youtube3 -rule "(HTTP.REQ.HOSTNAME.APPEND(HTTP.REQ.URL).CONTAINS_ANY(\"ns_videoopt_yt_media_url_pattern\")) && (!HTTP.REQ.HOSTNAME.APPEND(HTTP.REQ.URL).CONTAINS(\"redirector\"))" -action DETECT_CLEARTEXT_PD", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_netflix_abr_ssl "(CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_netflix_abr_host_ends\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_https_abr_netflix -rule ns_videoopt_netflix_abr_ssl -action DETECT_ENCRYPTED_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_enc_video_detection_domain_list", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_enc_video_detection_domain_list c.youtube.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_enc_video_detection_domain_list googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_non_video_yt_match", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_non_video_yt_match manifest.googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_non_video_yt_match redirector.googlevideo.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_pd_abr_detection "((CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_enc_video_detection_domain_list\")) && (!CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_non_video_yt_match\")))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_https_abr_youtube -rule ns_videoopt_pd_abr_detection -action TRIGGER_ENC_ABR_DETECTION", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_content_type_mp2t "HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH(\"video/mp2t\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_extension_ts_and_not_text "( ! HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH(\"text/html\") && HTTP.REQ.URL.SUFFIX.TO_LOWER.EQ(\"ts\") )"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_content_type_m4s "( HTTP.REQ.URL.SUFFIX.TO_LOWER.EQ(\"m4s\") && HTTP.RES.HEADER(\"Content-Type\").TO_LOWER.STARTSWITH_ANY(\"ns_videoopt_abr_content_type\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_http_abr_generic -rule "ns_videoopt_genabr_content_type_mp2t || ns_videoopt_genabr_extension_ts_and_not_text || ns_videoopt_genabr_content_type_m4s" -action DETECT_CLEARTEXT_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_ssl_cdn_domains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .llnwd.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .akamaized.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .akamaihd.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .aiv-cdn.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .vimeocdn.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .ttvnw.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .xvideos.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_domains .xvideos-cdn.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_ssl_cdn_slugs", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs hls", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs live", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs hds", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs "-vh."", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs "-lh."", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs vod", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs dash", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs sls3", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs streaming-video", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs smooth", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs amzn", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs amazon", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs skyfire", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_cdn_slugs vid-egc", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_ssl_cdn_hostnames "( CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_genabr_ssl_cdn_domains\") && CLIENT.SSL.DETECTED_DOMAIN.CONTAINS_ANY(\"ns_videoopt_genabr_ssl_cdn_slugs\")  && ! CLIENT.SSL.DETECTED_DOMAIN.CONTAINS(\"avodassets\") )"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_cloufdront_hosts", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts d1zq994zlcon8z.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts d2y4getct1maot.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts d6vqxr8nbo8pq.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts d2dl5stlh6f4nv.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts dbck1u9egdq5w.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_cloufdront_hosts d2lkq7nlcrdi7q.cloudfront.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_ssl_cloudfront "CLIENT.SSL.DETECTED_DOMAIN.EQUALS_ANY(\"ns_videoopt_genabr_cloufdront_hosts\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_ssl_xvideos "( CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH(\"-cdn.xvideos.com\") &&  CLIENT.SSL.DETECTED_DOMAIN.STARTSWITH(\"cdn\") )"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_ssl_domains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_domains .cdn.showmax.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_domains .dc3.dailymotion.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_domains skyfire-h2.vimeocdn.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_domains skyfire.vimeocdn.com", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_ssl_domains_expr " CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_genabr_ssl_domains\")"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_ssl_fb_domains", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_fb_domains fbcdn.net", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy patset ns_videoopt_genabr_ssl_fb_slugs", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "bind policy patset ns_videoopt_genabr_ssl_fb_slugs video", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add policy expression ns_videoopt_genabr_ssl_fb_hostnames "( CLIENT.SSL.DETECTED_DOMAIN.ENDSWITH_ANY(\"ns_videoopt_genabr_ssl_fb_domains\") && CLIENT.SSL.DETECTED_DOMAIN.CONTAINS_ANY(\"ns_videoopt_genabr_ssl_fb_slugs\"))"", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: _dispatch(): Failing command: "add videooptimization detectionpolicy ns_videoopt_https_abr_generic -rule "ns_videoopt_genabr_ssl_cdn_hostnames || ns_videoopt_genabr_ssl_cloudfront || ns_videoopt_genabr_ssl_xvideos || ns_videoopt_genabr_ssl_domains_expr || ns_videoopt_genabr_ssl_fb_hostnames" -action DETECT_ENCRYPTED_ABR", Error: cmd not applied as Feature(s) not licensed.
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsnetsvc: nsnetsvc sent command NSAPI_POST_STARTUP to PEs, ErrorCode=0x0
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 22 built-ins failed
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 9 built-ins exempted
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 44 built-ins immune
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 742 built-ins not sourced due to license absent
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 800 built-ins sourced for license SYSTEM
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 1 built-ins sourced for license WL
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 3 built-ins sourced for license LB
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 1 built-ins sourced for license CS
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 5 built-ins sourced for license SSL
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 2 built-ins sourced for license GSLB
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 24 built-ins sourced for license CF
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 1 built-ins sourced for license IC
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 80 built-ins sourced for license AAA
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 4 built-ins sourced for license REWRITE
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 14 built-ins sourced for license AppFw
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 9 built-ins sourced for license RESPONDER
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): 1 built-ins sourced for license AppFlow
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): Loading initial configuration
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_init_config(): Loading Partition configuration
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsnetsvc: nsnetsvc sent command NSAPI_NSCONF_READ_END to PEs, ErrorCode=0x0
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsnetsvc: nsnetsvc sent command NSAPI_INIT_DYNMEMPOOLS to PEs, ErrorCode=0x0
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_ch_config(): Skipping command (set callhome -mode CSP -hbcustomInterval 1#012) for non-CSP depoyments
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: Failed to open file:/nsconfig/.callhome.conf, No such file or directory
May  7 12:21:38 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_ch_config(): get_set_callhome_conf failed, error code: -1
May  7 12:21:43 cpx-ingress-64464fdb75-9wchv nsconfigd: cfd_start(): starting
May  7 12:21:43 cpx-ingress-64464fdb75-9wchv nscollect: ns_copyfile(): Not able to get info of file /var/log/db/default/nsdevmap.txt : No such file or directory
May  7 12:21:43 cpx-ingress-64464fdb75-9wchv aslearn[389]: Starting aslearn, debug_mode = false, restart_mode = false
#AppFW learning engine configuration file /var/netscaler/conf/aslearn.linux.conf is correct
task: name: Aslearn_HA_Primary_Task, isRtc: false, periodicity: 5, nextExecutionAt: 5, overRunCount: 0, maxOverRuns: 24, 
task: name: Aslearn_WAL_Cleanup_Task, isRtc: false, periodicity: 60, nextExecutionAt: 5, overRunCount: 0, maxOverRuns: 2, 
task: name: Aslearn_Packet_Loop_Task, isRtc: true, periodicity: 4294967295, nextExecutionAt: 5, overRunCount: 0, maxOverRuns: 2, 
mallopt(): setting M_MMAP_THRESHOLD failed
May  7 12:21:43 cpx-ingress-64464fdb75-9wchv nssync:  NSSYNC: SYNC started ....
Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully
Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully
ERROR: cpx/conf directory is not present
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:26 GMT  0-PPE-0 : default ICA Message 1 0 :  "Initializing c2c and nnm for session reliability"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:37 GMT  0-PPE-0 : default APPFW Message 3 0 :  "Found more than one fastmatch pattern in signature rule with id : 999522, using it as normal pattern"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:37 GMT  0-PPE-0 : default EVENT DEVICEUP 14 0 :  Device "server_svc_NSSVC_SYSLOG_UDP_192.0.0.2:514(internal)" - State UP
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:37 GMT  0-PPE-0 : default SSLVPN Message 15 0 :  "ns_addrm_homepagedbs_svc : Returning because homepagelen 0 nsconf_read_end 0 "
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:37 GMT  0-PPE-0 : default SSLVPN Message 16 0 :  "ns_addrm_homepagedbs_svc : Returning because homepagelen 0 nsconf_read_end 0 "
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:38 GMT  0-PPE-0 : default SSLVPN Message 17 0 :  "ns_vpn_dfa_client_cookies_pent default patset absent"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:38 GMT  0-PPE-0 : default SSLVPN Message 18 0 :  "ns_vpn_clients_ua_pent default patset absent"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:38 GMT  0-PPE-0 : default SSLVPN Message 19 0 :  "ns_cvpn_vsvr_fqdn_pent default patset absent"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:38 GMT  0-PPE-0 : default EVENT DEVICEUP 20 0 :  Device "server_svc_NSSVC_HTTP_127.0.0.2:8788(internal)" - State UP
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:38 GMT  0-PPE-0 : default EVENT CONFIGEND 21 0 :  CONFIG END
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:43 GMT  0-PPE-0 : default ROUTING Message 0 0 :  "IMI started"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:43 GMT  0-PPE-0 : default EVENT DEVICEUP 23 0 :  Device "server_svc_NSSVC_SNMPD_192.0.0.2:3335(internal)" - State UP
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:43 GMT  0-PPE-0 : default CLI CMD_EXECUTED 24 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 25 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "set system user nsroot -password "********" -externalAuth ENABLED -logging ENABLED" - Status "Success"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 26 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 27 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add ns ip 172.17.0.3 255.255.0.0 -type SNIP -arp ENABLED -icmp ENABLED -vServer ENABLED -telnet ENABLED -ftp ENABLED -gui ENABLED -ssh ENABLED -snmp ENABLED -mgmtAccess DISABLED -restrictAccess DISABLED -dynamicRouting DISABLED -decrementTTL DISABLED -advertiseOnDefaultPartition DISABLED -networkRoute DISABLED -state ENABLED -icmpResponse NONE -ownerNode 255 -arpResponse NONE -ownerDownResponse YES -arpOwner 255 -mptcpAdvertise NO -devno 16482304" - Status "ERROR: Adding NSIP not allowed"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 28 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 29 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add route 0.0.0.0 0.0.0.0 172.17.0.1 -distance 1 -cost 0 -weight 1 -protocol OSPF ISIS RIP BGP -msr DISABLED -routeType STATIC -dflags 0 -ownerGroup DEFAULT_NG -devno 16515072" - Status "Success"
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default EVENT ROUTEUP 30 0 :  Route route(0.0.0.0_0.0.0.0_172.17.0.1) - State UP
May  7 12:21:44 192.0.0.1  05/07/2022:12:21:44 GMT  0-PPE-0 : default CLI CMD_EXECUTED 31 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 32 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 33 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 cpx-ingress-64464fdb75-9wchv nsnetsvc: nslocal_fileRead(): Data secured for:ns-server-certificate
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 34 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add ssl certKey ns-server-certificate -cert ns-server.cert -key ns-server.key -inform PEM -expiryMonitor ENABLED -notificationPeriod 30 -bundle NO" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 35 0 :  Device "server_svc_internal_NSSVC_SIP_TLS_172.17.0.3:5061(nsrnatsip-127.0.0.1-5061)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 36 0 :  Device "server_svc_internal_NSSVC_RPCSVRS_172.17.0.3:3009(nskrpcs-127.0.0.1-3009)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 37 0 :  Device "server_svc_internal_NSSVC_SSL_172.17.0.3:9443(nshttps-127.0.0.1-9443)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 38 0 :  Device "server_svc_internal_NSSVC_SSL_TCP_172.17.0.3:3008(nsrpcs-127.0.0.1-3008)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 39 0 :  Device "server_svc_internal_NSSVC_SSL_fe80::3048:11ff:fe95:4c2d:9443(nshttps-::1l-9443)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 40 0 :  Device "server_svc_internal_NSSVC_SSL_TCP_fe80::3048:11ff:fe95:4c2d:3008(nsrpcs-::1l-3008)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default EVENT DEVICEUP 41 0 :  Device "server_svc_internal_NSSVC_RPCSVRS_192.0.0.1:3009(nskrpcs-192.0.0.1-3009)" - State UP
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 42 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 43 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "set ns tcpProfile nstcp_default_profile -WS ENABLED -SACK ENABLED -WSVal 8 -nagle DISABLED -ackOnPush ENABLED -mss 1460 -maxBurst 6 -initialCwnd 10 -delayedAck 100 -oooQSize 300 -maxPktPerMss 0 -pktPerRetx 1 -minRTO 1000 -slowStartIncr 2 -bufferSize 131072 -synCookie ENABLED -KAprobeUpdateLastactivity ENABLED -flavor BIC -dynamicReceiveBuffering DISABLED -KA DISABLED -KAconnIdleTime 900 -KAmaxProbes 3 -KAprobeInterval 75 -sendBuffsize 131072 -mptcp DISABLED -EstablishClientConn AUTOMATIC -tcpSegOffload AUTOMATIC -rstWindowAttenuate DISABLED -rstMaxAck ENABLED -spoofSynDrop DISABLED -ecn DISABLED -mptcpDropDataOnPreEstSF DISABLED -mptcpFastOpen DISABLED -mptcpSessionTimeout 0 -TimeStamp DISABLED -dsack ENABLED -ackAggregation DISABLED -frto ENABLED -maxcwnd 524288 -fack ENABLED -tcpmode TRANSPARENT -tcpFastO" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 44 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 45 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "set ns tcpProfile nstcp_internal_apps -WS ENABLED -SACK ENABLED -WSVal 8 -nagle DISABLED -ackOnPush ENABLED -mss 1460 -maxBurst 6 -initialCwnd 10 -delayedAck 100 -oooQSize 300 -maxPktPerMss 0 -pktPerRetx 1 -minRTO 1000 -slowStartIncr 2 -bufferSize 131072 -synCookie ENABLED -KAprobeUpdateLastactivity ENABLED -flavor BIC -dynamicReceiveBuffering DISABLED -KA DISABLED -KAconnIdleTime 900 -KAmaxProbes 3 -KAprobeInterval 75 -sendBuffsize 131072 -mptcp DISABLED -EstablishClientConn AUTOMATIC -tcpSegOffload AUTOMATIC -rstWindowAttenuate DISABLED -rstMaxAck ENABLED -spoofSynDrop DISABLED -ecn DISABLED -mptcpDropDataOnPreEstSF DISABLED -mptcpFastOpen DISABLED -mptcpSessionTimeout 0 -TimeStamp DISABLED -dsack ENABLED -ackAggregation DISABLED -frto ENABLED -maxcwnd 524288 -fack ENABLED -tcpmode TRANSPARENT -tcpFastOpe" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT  0-PPE-0 : default CLI CMD_EXECUTED 46 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:45 192.0.0.1  05/07/2022:12:21:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 47 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "set ns hostName cpx-ingress-64464fdb75-9wchv" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 48 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 49 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add appfw policy cpx_import_bypadd "client.ip.src.eq(192.0.0.2)" APPFW_BYPASS -devno 16547840" - Status "ERROR: Feature(s) not enabled"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 50 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 51 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind appfw global cpx_import_bypadd 1 -state ENABLED END -type REQ_OVERRIDE -devno 16580608" - Status "ERROR: Feature(s) not enabled"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 52 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 53 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add serviceGroup cpx_default_dns_servicegroup DNS -td 0 -cacheable NO -pathMonitor NO -pathMonitorIndv NO -sp OFF -rtspSessionidRemap OFF -maxBandwidth 0 -state ENABLED -downStateFlush ENABLED -appflowLog ENABLED -memberPort 0 -autoDisablegraceful NO -noDefaultBindings NO -devno 16613376" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 54 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:46 192.0.0.1  05/07/2022:12:21:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 55 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add serviceGroup cpx_default_dns_tcp_servicegroup DNS_TCP -td 0 -cacheable NO -pathMonitor NO -pathMonitorIndv NO -sp OFF -rtspSessionidRemap OFF -maxBandwidth 0 -state ENABLED -downStateFlush ENABLED -appflowLog ENABLED -memberPort 0 -autoDisablegraceful NO -noDefaultBindings NO -devno 16646144" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 56 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 57 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add lb monitor cpx_default_dns_tcp_monitor TCP -resptimeoutThresh 0 -retries 3 -failureRetries 0 -alertRetries 0 -successRetries 1 -IPMapping 0.0.0.0 -state ENABLED -reverse NO -transparent NO -ipTunnel NO -tos NO -secure NO -devno 16678912" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 58 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 59 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind serviceGroup cpx_default_dns_servicegroup -monitorName cpx_default_dns_tcp_monitor -monState ENABLED -devno 16711680" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 60 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 61 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind serviceGroup cpx_default_dns_tcp_servicegroup -monitorName cpx_default_dns_tcp_monitor -monState ENABLED -devno 16744448" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 62 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 63 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add lb vserver cpx_default_dns_vserver DNS -IPPattern 0.0.0.0 -IPMask * 0 -range 1 -timeout 2 -backupPersistenceTimeout 2 -lbMethod LEASTCONNECTION -rule none -Listenpolicy NONE -resRule none -persistMask 255.255.255.255 -v6persistmasklen 128 -m IP -sessionless DISABLED -trofsPersistence ENABLED -state ENABLED -connfailover DISABLED -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -healthThreshold 0 -redirectPortRewrite DISABLED -downStateFlush ENABLED -IPMapping 0.0.0.0 -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -minAutoscaleMembers 0 -maxAutoscaleMembers 0 -skippersistency None -td 0 -macmodeRetainvlan DISABLED -dns64 DISABLED -bypassAAAA NO -processLocal DISABLED -retainConnectionsOnCluster NO -noDefault" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 64 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:47 192.0.0.1  05/07/2022:12:21:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 65 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add lb vserver cpx_default_dns_tcp_vserver DNS_TCP -IPPattern 0.0.0.0 -IPMask * 0 -range 1 -timeout 2 -backupPersistenceTimeout 2 -lbMethod LEASTCONNECTION -rule none -Listenpolicy NONE -resRule none -persistMask 255.255.255.255 -v6persistmasklen 128 -m IP -sessionless DISABLED -trofsPersistence ENABLED -state ENABLED -connfailover DISABLED -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -healthThreshold 0 -redirectPortRewrite DISABLED -downStateFlush ENABLED -IPMapping 0.0.0.0 -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -minAutoscaleMembers 0 -maxAutoscaleMembers 0 -skippersistency None -td 0 -macmodeRetainvlan DISABLED -dns64 DISABLED -bypassAAAA NO -processLocal DISABLED -retainConnectionsOnCluster NO -n" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 66 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 67 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind lb vserver cpx_default_dns_vserver cpx_default_dns_servicegroup -weight 1 -devno 16842752" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 68 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 69 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind lb vserver cpx_default_dns_tcp_vserver cpx_default_dns_tcp_servicegroup -weight 1 -devno 16875520" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 70 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 71 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind serviceGroup cpx_default_dns_servicegroup 10.96.0.10 53 -CustomServerID None -state ENABLED -devno 16908288" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT MONITORUP 72 0 :  Monitor MonServiceBinding_10.96.0.10:53_(cpx_default_dns_tcp_monitor)(cpx_default_dns_servicegroup?10.96.0.10?53) - State UP
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 73 0 :  Device "server_serviceGroup_NSSVC_DNS_10.96.0.10:53(cpx_default_dns_servicegroup?10.96.0.10?53)" - State UP
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 74 0 :  Device "server_vip_NSSVC_DNS_0.0.0.0:0(cpx_default_dns_vserver)" - State UP
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 75 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 76 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "bind serviceGroup cpx_default_dns_tcp_servicegroup 10.96.0.10 53 -CustomServerID None -state ENABLED -devno 16941056" - Status "Success"
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT MONITORUP 77 0 :  Monitor MonServiceBinding_10.96.0.10:53_(cpx_default_dns_tcp_monitor)(cpx_default_dns_tcp_servicegroup?10.96.0.10?53) - State U
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 78 0 :  Device "server_serviceGroup_NSSVC_DNS_TCP_10.96.0.10:53(cpx_default_dns_tcp_servicegroup?10.96.0.10?53)" - State UP
May  7 12:21:48 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 79 0 :  Device "server_vip_NSSVC_DNS_TCP_0.0.0.0:0(cpx_default_dns_tcp_vserver)" - State UP
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 80 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 81 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add dns nameServer cpx_default_dns_vserver -state ENABLED -devno 16973824" - Status "Success"
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 82 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 83 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "add dns nameServer cpx_default_dns_tcp_vserver -state ENABLED -type TCP -devno 17006592" - Status "Success"
f54be1f3-9d4b-4727-b35c-245866c5737f
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 84 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "login nsroot "********"" - Status "Success"
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default CLI CMD_EXECUTED 85 0 :  User nsroot - Remote_ip 58.0.0.0 - Command "set ns config -IPAddress 172.17.0.3 -netmask 255.255.0.0 -tagged YES" - Status "Success"
exec: set ns config -deviceid f54be1f3-9d4b-4727-b35c-245866c5737f
Done
May  7 12:21:49 cpx-ingress-64464fdb75-9wchv nsaaad: (0-2) process_kernel_socket: AAA_INIT_PASS_KEY size is 384
May  7 12:21:49 192.0.0.1  05/07/2022:12:21:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 86 0 :  Device "server_svc_NSSVC_AAA_192.0.0.2:8766(internal)" - State UP
/etc/monit/monitrc:328: Include failed -- Success '/etc/monit/conf.d/*'
CPX started successfully. For logs please refer to /var/log/boot.log
May  7 12:22:00 cpx-ingress-64464fdb75-9wchv cron[258]: (*system*) RELOAD (/etc/crontab)
May  7 12:22:00 cpx-ingress-64464fdb75-9wchv CRON[746]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 87 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 88 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s_def_trans_server" - Status "ERROR: No such resource"
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 89 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add lb vserver k8s_def_trans_server LOGSTREAM -IPPattern 0.0.0.0 -IPMask * 0 -range 1 -timeout 2 -backupPersistenceTimeout 2 -lbMethod LEASTCONNECTION -rule none -Listenpolicy NONE -resRule none -persistMask 255.255.255.255 -v6persistmasklen 128 -m IP -sessionless DISABLED -trofsPersistence ENABLED -state ENABLED -connfailover DISABLED -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -healthThreshold 0 -redirectPortRewrite DISABLED -downStateFlush ENABLED -IPMapping 0.0.0.0 -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -minAutoscaleMembers 0 -maxAutoscaleMembers 0 -skippersistency None -td 0 -macmodeRetainvlan DISABLED -dns64 DISABLED -bypassAAAA NO -processLocal DISABLED -retainConnectionsOnCluster NO -noDefa" - Status "Success"
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 90 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "set ssl parameter -quantumSize 8192 -crlMemorySizeMB 256 -strictCAChecks NO -sslTriggerTimeout 100 -sendCloseNotify YES -encryptTriggerPktCount 45 -denySSLReneg ALL -insertionEncoding Unicode -ocspCacheSize 10 -pushFlag 0 -dropReqWithNoHostHeader NO -SNIHTTPHostMatch CERT -pushEncTriggerTimeout 1 -cryptodevDisableLimit 0 -undefActionControl CLIENTAUTH -undefActionData NOOP -defaultProfile ENABLED -svctls1112disable NO -montls1112disable NO -softwareCryptoThreshold 0 -hybridFIPSMode DISABLED -sigDigestType ALL -ssliErrorCache DISABLED -ssliMaxErrorCacheMem 0 -insertCertSpace YES -ndcppComplianceCertCheck NO -heterogeneousSSLHW DISABLED -operationQueueLimit 150" - Status "Success"
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 91 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns version" - Status "Success"
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 92 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns feature" - Status "Success"
May  7 12:22:24 cpx-ingress-64464fdb75-9wchv [344]: Failed to open file:/nsconfig/.callhome.conf, No such file or directory
May  7 12:22:24 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 93 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature LB CS SSL REWRITE RESPONDER AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 94 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature LB SSL REWRITE RESPONDER AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 95 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature SSL REWRITE RESPONDER AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 96 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature REWRITE RESPONDER AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 97 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature REWRITE AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 98 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature AppFlow APIGateway" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 99 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "enable ns feature APIGateway" - Status "ERROR: Feature(s) not licensed"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 100 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns httpProfile" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 101 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl profile" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 102 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns tcpProfile" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 103 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show analytics profile" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 104 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show server" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 105 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs policy" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 106 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs action" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 107 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s-dummy_csvs_to_test_edit.deleteme" - Status "ERROR: No such resource"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 108 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add lb vserver k8s-dummy_csvs_to_test_edit.deleteme HTTP 0.0.0.0 0 -range 1 -timeout 2 -backupPersistenceTimeout 2 -lbMethod LEASTCONNECTION -rule none -Listenpolicy NONE -resRule none -persistMask 255.255.255.255 -v6persistmasklen 128 -m IP -sessionless DISABLED -trofsPersistence ENABLED -state ENABLED -connfailover DISABLED -cacheable NO -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -healthThreshold 0 -redirectPortRewrite DISABLED -downStateFlush ENABLED -IPMapping 0.0.0.0 -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -push DISABLED -pushLabel none -pushMultiClients NO -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -minAutoscaleMembers 0 -maxAutoscaleMembers 0 -skippersistency None -td 0 -macmodeRetainvlan DISABLED -dns64 DISABLED -bypassAAAA NO -proc" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 109 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "rm lb vserver k8s-dummy_csvs_to_test_edit.deleteme" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 110 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "set ns timeout -zombie 120 -httpClient 0 -httpServer 0 -tcpClient 0 -tcpServer 0 -anyClient 0 -anyServer 0 -anyTcpClient 18000 -anyTcpServer 18000 -halfclose 10 -nontcpZombie 60 -ReducedFinTimeOut 30 -ReducedRstTimeOut 0 -NewConnIdleTimeOut 4" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 111 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns ip" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 112 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 113 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 114 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 115 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 116 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:25 192.0.0.1  05/07/2022:12:22:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 117 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show responder policy" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 118 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 119 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 120 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 121 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 122 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 123 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy stringmap CICINFO_default_k8s_HXGRQNQA3N2EDC3QCPBFP5B" - Status "ERROR: No such resource"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 124 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 125 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add policy stringmap CICINFO_default_k8s_HXGRQNQA3N2EDC3QCPBFP5B -comment "Prefix: k8s; Version: 1.18.5; Namespace: default; info used by CIC, do not delete;" -devno 17104896" - Status "Success"
May  7 12:22:25 cpx-ingress-64464fdb75-9wchv [344]: Failed to open file:/nsconfig/.callhome.conf, No such file or directory
May  7 12:22:26 cpx-ingress-64464fdb75-9wchv IMI[380]: informational IMI: Command "write memory" mode(4)
May  7 12:22:26 192.0.0.1  05/07/2022:12:22:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT STARTSAVECONFIG 126 0 :  SAVECONFIG start
May  7 12:22:32 cpx-ingress-64464fdb75-9wchv nsnetsvc: ns_copyfile(): Not able to get info of file /nsconfig/ns.conf : No such file or directory
May  7 12:22:32 192.0.0.1  05/07/2022:12:22:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT STOPSAVECONFIG 127 0 :  SAVECONFIG completed
May  7 12:22:33 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 128 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "save ns config" - Status "Success"
May  7 12:22:33 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 129 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:33 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 130 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns ip" - Status "Success"
May  7 12:22:33 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 131 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ipset" - Status "Success"
May  7 12:22:33 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 132 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy stringmap" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 133 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy patset" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 134 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy dataset" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 135 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns tcpProfile" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 136 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns httpProfile" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 137 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl profile" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 138 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy httpCallout" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 139 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show stream selector" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 140 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ns limitIdentifier" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 141 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show server" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 142 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show audit messageaction" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 143 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 144 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication ldapAction" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 145 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication OAuthAction" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 146 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication samlAction" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 147 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication loginSchema" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 148 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication loginSchemaPolicy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 149 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authorization policy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 150 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show analytics profile" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 151 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 152 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication vserver" - Status "ERROR: Feature(s) not licensed"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 153 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 154 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs action" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 155 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs policy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 156 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show rewrite action" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 157 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show rewrite policy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 158 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show responder action" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 159 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show responder policy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 160 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication Policy" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 161 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl cipher" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 162 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 163 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s_def_trans_server" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 164 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 165 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s_def_trans_server" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 166 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver" - Status "Success"
May  7 12:22:34 192.0.0.1  05/07/2022:12:22:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 167 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl cipher" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 168 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication vserver" - Status "ERROR: Feature(s) not licensed"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 169 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 170 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s_def_trans_server" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 171 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl vserver" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 172 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 173 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s_def_trans_server" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 174 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ssl profile" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 175 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 176 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 177 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show authentication vserver" - Status "ERROR: Feature(s) not licensed"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 178 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show ipset" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 179 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy patset" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 180 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy dataset" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 181 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show policy stringmap" - Status "Success"
May  7 12:22:35 192.0.0.1  05/07/2022:12:22:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 182 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup" - Status "Success"
May  7 12:22:36 192.0.0.1  05/07/2022:12:22:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 183 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:36 192.0.0.1  05/07/2022:12:22:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 184 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:36 192.0.0.1  05/07/2022:12:22:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 185 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:22:46 192.0.0.1  05/07/2022:12:22:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 186 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:46 192.0.0.1  05/07/2022:12:22:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 187 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:46 192.0.0.1  05/07/2022:12:22:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 188 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:22:56 192.0.0.1  05/07/2022:12:22:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 189 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:22:56 192.0.0.1  05/07/2022:12:22:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 190 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:22:56 192.0.0.1  05/07/2022:12:22:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 191 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:01 cpx-ingress-64464fdb75-9wchv CRON[756]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.377576] docker0: port 3(vethec1d40f) entered blocking state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.377648] docker0: port 3(vethec1d40f) entered disabled state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.377750] device vethec1d40f entered promiscuous mode
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.445826] docker0: port 4(veth3c3318d) entered blocking state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.445830] docker0: port 4(veth3c3318d) entered disabled state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.446989] device veth3c3318d entered promiscuous mode
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.447124] docker0: port 4(veth3c3318d) entered blocking state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.447127] docker0: port 4(veth3c3318d) entered forwarding state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.481032] docker0: port 5(vethf4a2a57) entered blocking state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.481037] docker0: port 5(vethf4a2a57) entered disabled state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.481185] device vethf4a2a57 entered promiscuous mode
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.481522] docker0: port 5(vethf4a2a57) entered blocking state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.481525] docker0: port 5(vethf4a2a57) entered forwarding state
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.620968] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:03 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.623002] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.685183] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.854612] docker0: port 6(vethd9aa6ab) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.854616] docker0: port 6(vethd9aa6ab) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.854677] device vethd9aa6ab entered promiscuous mode
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.854779] docker0: port 6(vethd9aa6ab) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1108.854782] docker0: port 6(vethd9aa6ab) entered forwarding state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.090646] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.111596] docker0: port 7(veth9974825) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.111600] docker0: port 7(veth9974825) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.111919] device veth9974825 entered promiscuous mode
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.141739] docker0: port 7(veth9974825) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.141743] docker0: port 7(veth9974825) entered forwarding state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.190781] eth0: renamed from veth9e0bd65
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.248596] docker0: port 5(vethf4a2a57) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.251884] docker0: port 6(vethd9aa6ab) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.252639] docker0: port 7(veth9974825) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.353546] docker0: port 8(vetha2661d1) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.353551] docker0: port 8(vetha2661d1) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.355946] device vetha2661d1 entered promiscuous mode
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.359872] docker0: port 8(vetha2661d1) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.359875] docker0: port 8(vetha2661d1) entered forwarding state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.380471] docker0: port 8(vetha2661d1) entered disabled state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.444423] eth0: renamed from veth5beb329
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.450574] docker0: port 3(vethec1d40f) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.450585] docker0: port 3(vethec1d40f) entered forwarding state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.490826] eth0: renamed from vethe45e242
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.494722] docker0: port 5(vethf4a2a57) entered blocking state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.494727] docker0: port 5(vethf4a2a57) entered forwarding state
May  7 12:23:04 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.646178] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1109.944758] IPVS: ftp: loaded support on port[0] = 21
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.010887] eth0: renamed from vethbc3c578
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.041522] docker0: port 6(vethd9aa6ab) entered blocking state
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.041546] docker0: port 6(vethd9aa6ab) entered forwarding state
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.287183] eth0: renamed from veth8e74853
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.296648] docker0: port 7(veth9974825) entered blocking state
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.296665] docker0: port 7(veth9974825) entered forwarding state
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.386842] eth0: renamed from vethd6504d0
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.392631] docker0: port 8(vetha2661d1) entered blocking state
May  7 12:23:05 cpx-ingress-64464fdb75-9wchv kernel: [ 1110.392635] docker0: port 8(vetha2661d1) entered forwarding state
May  7 12:23:06 192.0.0.1  05/07/2022:12:23:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 192 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:06 192.0.0.1  05/07/2022:12:23:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 193 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:06 192.0.0.1  05/07/2022:12:23:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 194 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:16 192.0.0.1  05/07/2022:12:23:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 195 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:16 192.0.0.1  05/07/2022:12:23:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 196 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:16 192.0.0.1  05/07/2022:12:23:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 197 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:26 192.0.0.1  05/07/2022:12:23:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 198 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:27 192.0.0.1  05/07/2022:12:23:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 199 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:27 192.0.0.1  05/07/2022:12:23:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 200 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:36 192.0.0.1  05/07/2022:12:23:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 201 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:36 192.0.0.1  05/07/2022:12:23:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 202 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:36 192.0.0.1  05/07/2022:12:23:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 203 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:47 192.0.0.1  05/07/2022:12:23:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 204 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:47 192.0.0.1  05/07/2022:12:23:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 205 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:47 192.0.0.1  05/07/2022:12:23:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 206 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:23:57 192.0.0.1  05/07/2022:12:23:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 207 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:23:57 192.0.0.1  05/07/2022:12:23:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 208 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:23:57 192.0.0.1  05/07/2022:12:23:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 209 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:00 cpx-ingress-64464fdb75-9wchv CRON[762]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:24:07 192.0.0.1  05/07/2022:12:24:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 210 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:07 192.0.0.1  05/07/2022:12:24:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 211 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:07 192.0.0.1  05/07/2022:12:24:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 212 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:17 192.0.0.1  05/07/2022:12:24:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 213 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:17 192.0.0.1  05/07/2022:12:24:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 214 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:17 192.0.0.1  05/07/2022:12:24:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 215 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:27 192.0.0.1  05/07/2022:12:24:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 216 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:27 192.0.0.1  05/07/2022:12:24:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 217 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:27 192.0.0.1  05/07/2022:12:24:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 218 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:37 192.0.0.1  05/07/2022:12:24:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 219 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:37 192.0.0.1  05/07/2022:12:24:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 220 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:37 192.0.0.1  05/07/2022:12:24:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 221 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:48 192.0.0.1  05/07/2022:12:24:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 222 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:48 192.0.0.1  05/07/2022:12:24:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 223 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:48 192.0.0.1  05/07/2022:12:24:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 224 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:24:58 192.0.0.1  05/07/2022:12:24:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 225 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:24:58 192.0.0.1  05/07/2022:12:24:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 226 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:24:58 192.0.0.1  05/07/2022:12:24:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 227 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:00 cpx-ingress-64464fdb75-9wchv CRON[768]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:25:08 192.0.0.1  05/07/2022:12:25:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 228 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:08 192.0.0.1  05/07/2022:12:25:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 229 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:08 192.0.0.1  05/07/2022:12:25:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 230 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:19 192.0.0.1  05/07/2022:12:25:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 231 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:19 192.0.0.1  05/07/2022:12:25:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 232 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:19 192.0.0.1  05/07/2022:12:25:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 233 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:29 192.0.0.1  05/07/2022:12:25:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 234 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:29 192.0.0.1  05/07/2022:12:25:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 235 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:29 192.0.0.1  05/07/2022:12:25:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 236 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:39 192.0.0.1  05/07/2022:12:25:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 237 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:39 192.0.0.1  05/07/2022:12:25:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 238 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:39 192.0.0.1  05/07/2022:12:25:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 239 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:49 192.0.0.1  05/07/2022:12:25:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 240 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:49 192.0.0.1  05/07/2022:12:25:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 241 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:49 192.0.0.1  05/07/2022:12:25:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 242 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:25:59 192.0.0.1  05/07/2022:12:25:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 243 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:25:59 192.0.0.1  05/07/2022:12:25:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 244 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:25:59 192.0.0.1  05/07/2022:12:25:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 245 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:26:01 cpx-ingress-64464fdb75-9wchv CRON[774]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:26:09 192.0.0.1  05/07/2022:12:26:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 246 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:26:09 192.0.0.1  05/07/2022:12:26:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 247 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:26:09 192.0.0.1  05/07/2022:12:26:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 248 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:26:19 192.0.0.1  05/07/2022:12:26:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 249 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:26:19 192.0.0.1  05/07/2022:12:26:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 250 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:26:19 192.0.0.1  05/07/2022:12:26:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 251 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:34:00 cpx-ingress-64464fdb75-9wchv kernel: [ 1770.963099] Hangcheck: hangcheck value past margin!
May  7 12:34:04 cpx-ingress-64464fdb75-9wchv CRON[780]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:34:10 192.0.0.1  05/07/2022:12:34:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 252 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:34:10 192.0.0.1  05/07/2022:12:34:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 253 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:34:10 192.0.0.1  05/07/2022:12:34:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 254 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:34:20 192.0.0.1  05/07/2022:12:34:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 255 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:34:20 192.0.0.1  05/07/2022:12:34:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 256 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:34:20 192.0.0.1  05/07/2022:12:34:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 257 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:34:30 192.0.0.1  05/07/2022:12:34:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 258 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:34:30 192.0.0.1  05/07/2022:12:34:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 259 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:34:30 192.0.0.1  05/07/2022:12:34:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 260 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:34:40 192.0.0.1  05/07/2022:12:34:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 261 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:34:40 192.0.0.1  05/07/2022:12:34:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 262 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:34:40 192.0.0.1  05/07/2022:12:34:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 263 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:34:50 192.0.0.1  05/07/2022:12:34:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 264 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:34:50 192.0.0.1  05/07/2022:12:34:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 265 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:34:50 192.0.0.1  05/07/2022:12:34:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 266 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:00 192.0.0.1  05/07/2022:12:35:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 267 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:00 192.0.0.1  05/07/2022:12:35:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 268 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:00 192.0.0.1  05/07/2022:12:35:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 269 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:00 cpx-ingress-64464fdb75-9wchv CRON[786]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 270 0 :  User <unknown> - Remote_ip 127.0.0.1 - Command "login "<unknown>" "********"" - Status "ERROR: Session expired or killed. Please login again"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 271 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 272 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver k8s-172.17.0.3_80_http" - Status "ERROR: No such resource"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 273 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add cs vserver k8s-172.17.0.3_80_http -td 0 HTTP 172.17.0.3 -range 1 80 -state ENABLED -stateupdate DISABLED -cacheable NO -precedence RULE -caseSensitive ON -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -redirectPortRewrite DISABLED -downStateFlush ENABLED -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -Listenpolicy NONE -push DISABLED -pushLabel none -pushMultiClients NO -comment uid=EAB4T5T3QI6B4P7335ZR3TGW4YWGYU5OR4WD6QPJWQVXQD236FKQ==== -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -dtls OFF -noDefaultBindings NO -persistMask 255.255.255.255 -v6persistmasklen 128 -timeout 2 -backupPersistenceTimeout 2 -devno 17137664" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 274 0 :  Device "server_vip_NSSVC_HTTP_172.17.0.3:80(k8s-172.17.0.3_80_http)" - State UP
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 275 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "ERROR: No such resource"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 276 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g HTTP 0.0.0.0 0 -range 1 -timeout 2 -backupPersistenceTimeout 2 -lbMethod LEASTCONNECTION -rule none -Listenpolicy NONE -resRule none -persistMask 255.255.255.255 -v6persistmasklen 128 -m IP -sessionless DISABLED -trofsPersistence ENABLED -state ENABLED -connfailover DISABLED -cacheable NO -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -healthThreshold 0 -redirectPortRewrite DISABLED -downStateFlush ENABLED -IPMapping 0.0.0.0 -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -push DISABLED -pushLabel none -pushMultiClients NO -comment "rv:1158,ing:guestbook-ingress,ingport:80,ns:default,svc:frontend,svcport:80" -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -minAutoscaleMembers 0 -maxAutoscal" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 277 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs action k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "ERROR: Action does not exist"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 278 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add cs action k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g -targetLBVserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g -devno 17203200" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 279 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "ERROR: No such policy exists"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 280 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add cs policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g -rule "HTTP.REQ.HOSTNAME.SERVER.EQ(\"www.guestbook.com\") && HTTP.REQ.URL.PATH.SET_TEXT_MODE(IGNORECASE).STARTSWITH(\"/\")" -action k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g -devno 17235968" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 281 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver k8s-172.17.0.3_80_http" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 282 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver k8s-172.17.0.3_80_http" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 283 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "bind cs vserver k8s-172.17.0.3_80_http -policyName k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g -priority 200000008 -devno 17268736" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 284 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "ERROR: No such resource"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 285 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "add serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g HTTP -td 0 -cacheable NO -pathMonitor NO -pathMonitorIndv NO -sp OFF -rtspSessionidRemap OFF -maxBandwidth 0 -state ENABLED -downStateFlush ENABLED -comment "ing:guestbook-ingress,ingport:80,ns:default,svc:frontend,svcport:80" -appflowLog ENABLED -memberPort 0 -autoDisablegraceful NO -noDefaultBindings NO -devno 17301504" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 286 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 287 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "bind lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g -weight 1 -devno 17334272" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 288 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 289 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "bind serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g 172.17.0.8 80 -CustomServerID None -state ENABLED -devno 17367040" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT MONITORUP 290 0 :  Monitor MonServiceBinding_172.17.0.8:80_(tcp-default)(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.8?80) - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 291 0 :  Device "server_serviceGroup_NSSVC_HTTP_172.17.0.8:80(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.8?80)" - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 292 0 :  Device "server_vip_NSSVC_HTTP_0.0.0.0:0(k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g)" - State UP
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 293 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "bind serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g 172.17.0.7 80 -CustomServerID None -state ENABLED -devno 17399808" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT MONITORUP 294 0 :  Monitor MonServiceBinding_172.17.0.7:80_(tcp-default)(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.7?80) - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 295 0 :  Device "server_serviceGroup_NSSVC_HTTP_172.17.0.7:80(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.7?80)" - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 296 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "bind serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g 172.17.0.9 80 -CustomServerID None -state ENABLED -devno 17432576" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT MONITORUP 297 0 :  Monitor MonServiceBinding_172.17.0.9:80_(tcp-default)(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.9?80) - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default EVENT DEVICEUP 298 0 :  Device "server_serviceGroup_NSSVC_HTTP_172.17.0.9:80(k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g?172.17.0.9?80)" - Sta
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 299 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver k8s-172.17.0.3_80_http" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 300 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "set cs vserver k8s-172.17.0.3_80_http -IPAddress 172.17.0.3 -IPPattern 0.0.0.0 -IPMask * -stateupdate DISABLED -soMethod NONE -soPersistence DISABLED -soPersistenceTimeOut 2 -redirectPortRewrite DISABLED -downStateFlush ENABLED -disablePrimaryOnDown DISABLED -insertVserverIPPort OFF -rtspNat OFF -Listenpolicy NONE -push DISABLED -pushLabel none -comment uid=EAB4T5T3QI6B4P7335ZR3TGW4YWGYU5OR4WD6QPJWQVXQD236FKQ==== -l2Conn OFF -appflowLog ENABLED -icmpVsrResponse PASSIVE -RHIstate PASSIVE -dnsRecordType A -dtls OFF -persistMask 255.255.255.255 -v6persistmasklen 128 -timeout 2" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 301 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 302 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs action k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 303 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 304 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show cs vserver k8s-172.17.0.3_80_http" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 305 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 306 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show lb vserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:06 192.0.0.1  05/07/2022:12:35:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 307 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "show serviceGroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g" - Status "Success"
May  7 12:35:10 192.0.0.1  05/07/2022:12:35:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 308 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:10 192.0.0.1  05/07/2022:12:35:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 309 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:10 192.0.0.1  05/07/2022:12:35:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 310 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:21 192.0.0.1  05/07/2022:12:35:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 311 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:21 192.0.0.1  05/07/2022:12:35:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 312 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:21 192.0.0.1  05/07/2022:12:35:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 313 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:30 192.0.0.1  05/07/2022:12:35:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 314 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:30 192.0.0.1  05/07/2022:12:35:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 315 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:30 192.0.0.1  05/07/2022:12:35:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 316 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:41 192.0.0.1  05/07/2022:12:35:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 317 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:41 192.0.0.1  05/07/2022:12:35:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 318 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:41 192.0.0.1  05/07/2022:12:35:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 319 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:35:51 192.0.0.1  05/07/2022:12:35:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 320 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:35:51 192.0.0.1  05/07/2022:12:35:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 321 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:35:51 192.0.0.1  05/07/2022:12:35:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 322 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:00 cpx-ingress-64464fdb75-9wchv CRON[792]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:36:01 192.0.0.1  05/07/2022:12:36:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 323 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:01 192.0.0.1  05/07/2022:12:36:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 324 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:01 192.0.0.1  05/07/2022:12:36:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 325 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:11 192.0.0.1  05/07/2022:12:36:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 326 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:11 192.0.0.1  05/07/2022:12:36:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 327 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:11 192.0.0.1  05/07/2022:12:36:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 328 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:21 192.0.0.1  05/07/2022:12:36:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 329 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:21 192.0.0.1  05/07/2022:12:36:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 330 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:21 192.0.0.1  05/07/2022:12:36:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 331 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:31 192.0.0.1  05/07/2022:12:36:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 332 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:31 192.0.0.1  05/07/2022:12:36:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 333 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:31 192.0.0.1  05/07/2022:12:36:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 334 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:41 192.0.0.1  05/07/2022:12:36:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 335 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:41 192.0.0.1  05/07/2022:12:36:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 336 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:41 192.0.0.1  05/07/2022:12:36:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 337 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:36:51 192.0.0.1  05/07/2022:12:36:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 338 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:36:51 192.0.0.1  05/07/2022:12:36:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 339 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:36:51 192.0.0.1  05/07/2022:12:36:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 340 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:01 cpx-ingress-64464fdb75-9wchv CRON[798]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:37:01 192.0.0.1  05/07/2022:12:37:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 341 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:01 192.0.0.1  05/07/2022:12:37:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 342 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:01 192.0.0.1  05/07/2022:12:37:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 343 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:11 192.0.0.1  05/07/2022:12:37:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 344 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:11 192.0.0.1  05/07/2022:12:37:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 345 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:11 192.0.0.1  05/07/2022:12:37:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 346 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:21 192.0.0.1  05/07/2022:12:37:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 347 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:21 192.0.0.1  05/07/2022:12:37:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 348 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:21 192.0.0.1  05/07/2022:12:37:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 349 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:31 192.0.0.1  05/07/2022:12:37:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 350 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:31 192.0.0.1  05/07/2022:12:37:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 351 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:31 192.0.0.1  05/07/2022:12:37:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 352 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:42 192.0.0.1  05/07/2022:12:37:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 353 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:42 192.0.0.1  05/07/2022:12:37:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 354 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:42 192.0.0.1  05/07/2022:12:37:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 355 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:37:52 192.0.0.1  05/07/2022:12:37:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 356 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:37:52 192.0.0.1  05/07/2022:12:37:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 357 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:37:52 192.0.0.1  05/07/2022:12:37:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 358 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:00 cpx-ingress-64464fdb75-9wchv CRON[804]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:38:02 192.0.0.1  05/07/2022:12:38:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 359 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:02 192.0.0.1  05/07/2022:12:38:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 360 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:02 192.0.0.1  05/07/2022:12:38:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 361 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:12 192.0.0.1  05/07/2022:12:38:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 362 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:12 192.0.0.1  05/07/2022:12:38:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 363 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:12 192.0.0.1  05/07/2022:12:38:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 364 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:22 192.0.0.1  05/07/2022:12:38:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 365 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:22 192.0.0.1  05/07/2022:12:38:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 366 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:22 192.0.0.1  05/07/2022:12:38:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 367 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:32 192.0.0.1  05/07/2022:12:38:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 368 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:32 192.0.0.1  05/07/2022:12:38:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 369 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:32 192.0.0.1  05/07/2022:12:38:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 370 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:42 192.0.0.1  05/07/2022:12:38:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 371 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:42 192.0.0.1  05/07/2022:12:38:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 372 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:42 192.0.0.1  05/07/2022:12:38:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 373 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:38:52 192.0.0.1  05/07/2022:12:38:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 374 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:38:52 192.0.0.1  05/07/2022:12:38:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 375 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:38:52 192.0.0.1  05/07/2022:12:38:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 376 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:00 cpx-ingress-64464fdb75-9wchv CRON[810]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:39:02 192.0.0.1  05/07/2022:12:39:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 377 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:02 192.0.0.1  05/07/2022:12:39:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 378 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:02 192.0.0.1  05/07/2022:12:39:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 379 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:12 192.0.0.1  05/07/2022:12:39:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 380 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:12 192.0.0.1  05/07/2022:12:39:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 381 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:12 192.0.0.1  05/07/2022:12:39:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 382 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:22 192.0.0.1  05/07/2022:12:39:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 383 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:22 192.0.0.1  05/07/2022:12:39:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 384 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:22 192.0.0.1  05/07/2022:12:39:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 385 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:32 192.0.0.1  05/07/2022:12:39:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 386 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:32 192.0.0.1  05/07/2022:12:39:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 387 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:32 192.0.0.1  05/07/2022:12:39:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 388 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:42 192.0.0.1  05/07/2022:12:39:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 389 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:42 192.0.0.1  05/07/2022:12:39:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 390 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:42 192.0.0.1  05/07/2022:12:39:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 391 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:39:53 192.0.0.1  05/07/2022:12:39:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 392 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:39:53 192.0.0.1  05/07/2022:12:39:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 393 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:39:53 192.0.0.1  05/07/2022:12:39:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 394 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:01 cpx-ingress-64464fdb75-9wchv CRON[816]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:40:02 192.0.0.1  05/07/2022:12:40:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 395 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:02 192.0.0.1  05/07/2022:12:40:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 396 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:02 192.0.0.1  05/07/2022:12:40:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 397 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:13 192.0.0.1  05/07/2022:12:40:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 398 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:13 192.0.0.1  05/07/2022:12:40:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 399 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:13 192.0.0.1  05/07/2022:12:40:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 400 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:23 192.0.0.1  05/07/2022:12:40:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 401 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:23 192.0.0.1  05/07/2022:12:40:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 402 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:23 192.0.0.1  05/07/2022:12:40:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 403 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:33 192.0.0.1  05/07/2022:12:40:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 404 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:33 192.0.0.1  05/07/2022:12:40:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 405 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:33 192.0.0.1  05/07/2022:12:40:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 406 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:43 192.0.0.1  05/07/2022:12:40:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 407 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:43 192.0.0.1  05/07/2022:12:40:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 408 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:43 192.0.0.1  05/07/2022:12:40:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 409 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:40:53 192.0.0.1  05/07/2022:12:40:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 410 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:40:53 192.0.0.1  05/07/2022:12:40:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 411 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:40:53 192.0.0.1  05/07/2022:12:40:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 412 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:00 cpx-ingress-64464fdb75-9wchv CRON[822]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:41:03 192.0.0.1  05/07/2022:12:41:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 413 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:03 192.0.0.1  05/07/2022:12:41:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 414 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:03 192.0.0.1  05/07/2022:12:41:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 415 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:13 192.0.0.1  05/07/2022:12:41:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 416 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:13 192.0.0.1  05/07/2022:12:41:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 417 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:13 192.0.0.1  05/07/2022:12:41:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 418 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:23 192.0.0.1  05/07/2022:12:41:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 419 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:23 192.0.0.1  05/07/2022:12:41:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 420 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:23 192.0.0.1  05/07/2022:12:41:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 421 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:33 192.0.0.1  05/07/2022:12:41:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 422 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:33 192.0.0.1  05/07/2022:12:41:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 423 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:33 192.0.0.1  05/07/2022:12:41:33 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 424 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:43 192.0.0.1  05/07/2022:12:41:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 425 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:43 192.0.0.1  05/07/2022:12:41:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 426 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:43 192.0.0.1  05/07/2022:12:41:43 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 427 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:41:53 192.0.0.1  05/07/2022:12:41:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 428 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:41:53 192.0.0.1  05/07/2022:12:41:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 429 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:41:53 192.0.0.1  05/07/2022:12:41:53 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 430 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:00 cpx-ingress-64464fdb75-9wchv CRON[828]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:42:03 192.0.0.1  05/07/2022:12:42:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 431 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:03 192.0.0.1  05/07/2022:12:42:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 432 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:03 192.0.0.1  05/07/2022:12:42:03 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 433 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:14 192.0.0.1  05/07/2022:12:42:13 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 434 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:14 192.0.0.1  05/07/2022:12:42:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 435 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:14 192.0.0.1  05/07/2022:12:42:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 436 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:23 192.0.0.1  05/07/2022:12:42:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 437 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:23 192.0.0.1  05/07/2022:12:42:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 438 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:23 192.0.0.1  05/07/2022:12:42:23 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 439 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:34 192.0.0.1  05/07/2022:12:42:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 440 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:34 192.0.0.1  05/07/2022:12:42:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 441 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:34 192.0.0.1  05/07/2022:12:42:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 442 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:44 192.0.0.1  05/07/2022:12:42:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 443 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:44 192.0.0.1  05/07/2022:12:42:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 444 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:44 192.0.0.1  05/07/2022:12:42:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 445 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:42:54 192.0.0.1  05/07/2022:12:42:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 446 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:42:54 192.0.0.1  05/07/2022:12:42:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 447 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:42:54 192.0.0.1  05/07/2022:12:42:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 448 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:01 cpx-ingress-64464fdb75-9wchv CRON[834]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:43:04 192.0.0.1  05/07/2022:12:43:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 449 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:04 192.0.0.1  05/07/2022:12:43:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 450 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:04 192.0.0.1  05/07/2022:12:43:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 451 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:14 192.0.0.1  05/07/2022:12:43:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 452 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:14 192.0.0.1  05/07/2022:12:43:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 453 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:14 192.0.0.1  05/07/2022:12:43:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 454 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:24 192.0.0.1  05/07/2022:12:43:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 455 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:24 192.0.0.1  05/07/2022:12:43:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 456 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:24 192.0.0.1  05/07/2022:12:43:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 457 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:34 192.0.0.1  05/07/2022:12:43:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 458 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:34 192.0.0.1  05/07/2022:12:43:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 459 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:34 192.0.0.1  05/07/2022:12:43:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 460 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:44 192.0.0.1  05/07/2022:12:43:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 461 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:44 192.0.0.1  05/07/2022:12:43:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 462 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:44 192.0.0.1  05/07/2022:12:43:44 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 463 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:43:54 192.0.0.1  05/07/2022:12:43:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 464 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:43:54 192.0.0.1  05/07/2022:12:43:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 465 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:43:54 192.0.0.1  05/07/2022:12:43:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 466 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:00 cpx-ingress-64464fdb75-9wchv CRON[840]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:44:04 192.0.0.1  05/07/2022:12:44:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 467 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:04 192.0.0.1  05/07/2022:12:44:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 468 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:04 192.0.0.1  05/07/2022:12:44:04 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 469 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:14 192.0.0.1  05/07/2022:12:44:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 470 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:14 192.0.0.1  05/07/2022:12:44:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 471 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:14 192.0.0.1  05/07/2022:12:44:14 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 472 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:24 192.0.0.1  05/07/2022:12:44:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 473 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:24 192.0.0.1  05/07/2022:12:44:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 474 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:24 192.0.0.1  05/07/2022:12:44:24 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 475 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:34 192.0.0.1  05/07/2022:12:44:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 476 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:34 192.0.0.1  05/07/2022:12:44:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 477 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:34 192.0.0.1  05/07/2022:12:44:34 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 478 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:45 192.0.0.1  05/07/2022:12:44:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 479 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:45 192.0.0.1  05/07/2022:12:44:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 480 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:45 192.0.0.1  05/07/2022:12:44:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 481 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:44:54 192.0.0.1  05/07/2022:12:44:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 482 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:44:54 192.0.0.1  05/07/2022:12:44:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 483 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:44:54 192.0.0.1  05/07/2022:12:44:54 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 484 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:00 cpx-ingress-64464fdb75-9wchv CRON[846]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:45:05 192.0.0.1  05/07/2022:12:45:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 485 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:05 192.0.0.1  05/07/2022:12:45:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 486 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:05 192.0.0.1  05/07/2022:12:45:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 487 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:15 192.0.0.1  05/07/2022:12:45:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 488 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:15 192.0.0.1  05/07/2022:12:45:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 489 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:15 192.0.0.1  05/07/2022:12:45:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 490 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:25 192.0.0.1  05/07/2022:12:45:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 491 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:25 192.0.0.1  05/07/2022:12:45:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 492 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:25 192.0.0.1  05/07/2022:12:45:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 493 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:35 192.0.0.1  05/07/2022:12:45:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 494 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:35 192.0.0.1  05/07/2022:12:45:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 495 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:35 192.0.0.1  05/07/2022:12:45:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 496 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:45 192.0.0.1  05/07/2022:12:45:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 497 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:45 192.0.0.1  05/07/2022:12:45:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 498 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:45 192.0.0.1  05/07/2022:12:45:45 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 499 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:45:55 192.0.0.1  05/07/2022:12:45:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 500 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:45:55 192.0.0.1  05/07/2022:12:45:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 501 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:45:55 192.0.0.1  05/07/2022:12:45:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 502 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:00 cpx-ingress-64464fdb75-9wchv CRON[852]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:46:05 192.0.0.1  05/07/2022:12:46:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 503 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:05 192.0.0.1  05/07/2022:12:46:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 504 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:05 192.0.0.1  05/07/2022:12:46:05 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 505 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:15 192.0.0.1  05/07/2022:12:46:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 506 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:15 192.0.0.1  05/07/2022:12:46:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 507 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:15 192.0.0.1  05/07/2022:12:46:15 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 508 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:25 192.0.0.1  05/07/2022:12:46:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 509 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:25 192.0.0.1  05/07/2022:12:46:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 510 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:25 192.0.0.1  05/07/2022:12:46:25 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 511 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:35 192.0.0.1  05/07/2022:12:46:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 512 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:35 192.0.0.1  05/07/2022:12:46:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 513 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:35 192.0.0.1  05/07/2022:12:46:35 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 514 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:46 192.0.0.1  05/07/2022:12:46:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 515 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:46 192.0.0.1  05/07/2022:12:46:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 516 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:46 192.0.0.1  05/07/2022:12:46:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 517 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:46:55 192.0.0.1  05/07/2022:12:46:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 518 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:46:55 192.0.0.1  05/07/2022:12:46:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 519 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:46:55 192.0.0.1  05/07/2022:12:46:55 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 520 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:01 cpx-ingress-64464fdb75-9wchv CRON[858]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:47:06 192.0.0.1  05/07/2022:12:47:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 521 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:06 192.0.0.1  05/07/2022:12:47:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 522 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:06 192.0.0.1  05/07/2022:12:47:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 523 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:16 192.0.0.1  05/07/2022:12:47:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 524 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:16 192.0.0.1  05/07/2022:12:47:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 525 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:16 192.0.0.1  05/07/2022:12:47:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 526 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:26 192.0.0.1  05/07/2022:12:47:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 527 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:26 192.0.0.1  05/07/2022:12:47:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 528 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:26 192.0.0.1  05/07/2022:12:47:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 529 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:36 192.0.0.1  05/07/2022:12:47:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 530 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:36 192.0.0.1  05/07/2022:12:47:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 531 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:36 192.0.0.1  05/07/2022:12:47:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 532 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:46 192.0.0.1  05/07/2022:12:47:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 533 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:46 192.0.0.1  05/07/2022:12:47:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 534 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:46 192.0.0.1  05/07/2022:12:47:46 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 535 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:47:56 192.0.0.1  05/07/2022:12:47:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 536 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:47:56 192.0.0.1  05/07/2022:12:47:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 537 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:47:56 192.0.0.1  05/07/2022:12:47:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 538 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:00 cpx-ingress-64464fdb75-9wchv CRON[865]: (root) CMD (   /var/netscaler/bins/nslog.sh dozip)
May  7 12:48:00 cpx-ingress-64464fdb75-9wchv CRON[866]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:48:06 192.0.0.1  05/07/2022:12:48:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 539 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:06 192.0.0.1  05/07/2022:12:48:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 540 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:06 192.0.0.1  05/07/2022:12:48:06 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 541 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:16 192.0.0.1  05/07/2022:12:48:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 542 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:16 192.0.0.1  05/07/2022:12:48:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 543 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:16 192.0.0.1  05/07/2022:12:48:16 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 544 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:26 192.0.0.1  05/07/2022:12:48:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 545 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:26 192.0.0.1  05/07/2022:12:48:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 546 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:26 192.0.0.1  05/07/2022:12:48:26 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 547 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:36 192.0.0.1  05/07/2022:12:48:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 548 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:36 192.0.0.1  05/07/2022:12:48:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 549 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:36 192.0.0.1  05/07/2022:12:48:36 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 550 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:47 192.0.0.1  05/07/2022:12:48:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 551 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:47 192.0.0.1  05/07/2022:12:48:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 552 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:47 192.0.0.1  05/07/2022:12:48:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 553 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:56 192.0.0.1  05/07/2022:12:48:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 554 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:48:56 192.0.0.1  05/07/2022:12:48:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 555 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:48:56 192.0.0.1  05/07/2022:12:48:56 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 556 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:48:59 cpx-ingress-64464fdb75-9wchv CRON[863]: (CRON) info (No MTA installed, discarding output)
May  7 12:49:01 cpx-ingress-64464fdb75-9wchv CRON[890]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:49:07 192.0.0.1  05/07/2022:12:49:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 557 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:07 192.0.0.1  05/07/2022:12:49:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 558 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:07 192.0.0.1  05/07/2022:12:49:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 559 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:49:17 192.0.0.1  05/07/2022:12:49:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 560 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:17 192.0.0.1  05/07/2022:12:49:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 561 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:17 192.0.0.1  05/07/2022:12:49:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 562 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:49:27 192.0.0.1  05/07/2022:12:49:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 563 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:27 192.0.0.1  05/07/2022:12:49:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 564 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:27 192.0.0.1  05/07/2022:12:49:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 565 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:49:37 192.0.0.1  05/07/2022:12:49:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 566 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:37 192.0.0.1  05/07/2022:12:49:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 567 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:37 192.0.0.1  05/07/2022:12:49:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 568 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:49:47 192.0.0.1  05/07/2022:12:49:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 569 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:47 192.0.0.1  05/07/2022:12:49:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 570 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:47 192.0.0.1  05/07/2022:12:49:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 571 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:49:57 192.0.0.1  05/07/2022:12:49:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 572 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:49:57 192.0.0.1  05/07/2022:12:49:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 573 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:49:57 192.0.0.1  05/07/2022:12:49:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 574 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:01 cpx-ingress-64464fdb75-9wchv CRON[896]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:50:07 192.0.0.1  05/07/2022:12:50:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 575 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:07 192.0.0.1  05/07/2022:12:50:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 576 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:07 192.0.0.1  05/07/2022:12:50:07 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 577 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:17 192.0.0.1  05/07/2022:12:50:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 578 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:17 192.0.0.1  05/07/2022:12:50:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 579 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:17 192.0.0.1  05/07/2022:12:50:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 580 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:27 192.0.0.1  05/07/2022:12:50:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 581 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:27 192.0.0.1  05/07/2022:12:50:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 582 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:27 192.0.0.1  05/07/2022:12:50:27 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 583 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:37 192.0.0.1  05/07/2022:12:50:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 584 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:37 192.0.0.1  05/07/2022:12:50:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 585 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:37 192.0.0.1  05/07/2022:12:50:37 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 586 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:47 192.0.0.1  05/07/2022:12:50:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 587 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:47 192.0.0.1  05/07/2022:12:50:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 588 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:47 192.0.0.1  05/07/2022:12:50:47 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 589 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:50:57 192.0.0.1  05/07/2022:12:50:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 590 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:50:57 192.0.0.1  05/07/2022:12:50:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 591 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:50:57 192.0.0.1  05/07/2022:12:50:57 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 592 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:00 cpx-ingress-64464fdb75-9wchv CRON[902]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:51:08 192.0.0.1  05/07/2022:12:51:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 593 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:08 192.0.0.1  05/07/2022:12:51:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 594 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:08 192.0.0.1  05/07/2022:12:51:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 595 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:17 192.0.0.1  05/07/2022:12:51:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 596 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:17 192.0.0.1  05/07/2022:12:51:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 597 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:17 192.0.0.1  05/07/2022:12:51:17 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 598 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:28 192.0.0.1  05/07/2022:12:51:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 599 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:28 192.0.0.1  05/07/2022:12:51:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 600 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:28 192.0.0.1  05/07/2022:12:51:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 601 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:38 192.0.0.1  05/07/2022:12:51:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 602 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:38 192.0.0.1  05/07/2022:12:51:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 603 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:38 192.0.0.1  05/07/2022:12:51:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 604 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:48 192.0.0.1  05/07/2022:12:51:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 605 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:48 192.0.0.1  05/07/2022:12:51:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 606 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:48 192.0.0.1  05/07/2022:12:51:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 607 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:51:58 192.0.0.1  05/07/2022:12:51:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 608 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:51:58 192.0.0.1  05/07/2022:12:51:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 609 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:51:58 192.0.0.1  05/07/2022:12:51:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 610 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:00 cpx-ingress-64464fdb75-9wchv CRON[908]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:52:08 192.0.0.1  05/07/2022:12:52:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 611 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:08 192.0.0.1  05/07/2022:12:52:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 612 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:08 192.0.0.1  05/07/2022:12:52:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 613 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:18 192.0.0.1  05/07/2022:12:52:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 614 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:18 192.0.0.1  05/07/2022:12:52:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 615 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:18 192.0.0.1  05/07/2022:12:52:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 616 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:28 192.0.0.1  05/07/2022:12:52:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 617 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:28 192.0.0.1  05/07/2022:12:52:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 618 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:28 192.0.0.1  05/07/2022:12:52:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 619 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:38 192.0.0.1  05/07/2022:12:52:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 620 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:38 192.0.0.1  05/07/2022:12:52:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 621 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:38 192.0.0.1  05/07/2022:12:52:38 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 622 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:48 192.0.0.1  05/07/2022:12:52:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 623 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:48 192.0.0.1  05/07/2022:12:52:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 624 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:48 192.0.0.1  05/07/2022:12:52:48 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 625 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:52:58 192.0.0.1  05/07/2022:12:52:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 626 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:52:58 192.0.0.1  05/07/2022:12:52:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 627 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:52:58 192.0.0.1  05/07/2022:12:52:58 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 628 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:01 cpx-ingress-64464fdb75-9wchv CRON[914]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:53:08 192.0.0.1  05/07/2022:12:53:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 629 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:08 192.0.0.1  05/07/2022:12:53:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 630 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:08 192.0.0.1  05/07/2022:12:53:08 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 631 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:18 192.0.0.1  05/07/2022:12:53:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 632 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:18 192.0.0.1  05/07/2022:12:53:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 633 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:18 192.0.0.1  05/07/2022:12:53:18 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 634 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:28 192.0.0.1  05/07/2022:12:53:28 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 635 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:29 192.0.0.1  05/07/2022:12:53:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 636 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:29 192.0.0.1  05/07/2022:12:53:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 637 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:39 192.0.0.1  05/07/2022:12:53:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 638 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:39 192.0.0.1  05/07/2022:12:53:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 639 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:39 192.0.0.1  05/07/2022:12:53:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 640 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:49 192.0.0.1  05/07/2022:12:53:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 641 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:49 192.0.0.1  05/07/2022:12:53:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 642 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:49 192.0.0.1  05/07/2022:12:53:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 643 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:53:59 192.0.0.1  05/07/2022:12:53:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 644 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:53:59 192.0.0.1  05/07/2022:12:53:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 645 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:53:59 192.0.0.1  05/07/2022:12:53:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 646 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:00 cpx-ingress-64464fdb75-9wchv CRON[920]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:54:09 192.0.0.1  05/07/2022:12:54:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 647 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:09 192.0.0.1  05/07/2022:12:54:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 648 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:09 192.0.0.1  05/07/2022:12:54:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 649 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:19 192.0.0.1  05/07/2022:12:54:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 650 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:19 192.0.0.1  05/07/2022:12:54:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 651 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:19 192.0.0.1  05/07/2022:12:54:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 652 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:29 192.0.0.1  05/07/2022:12:54:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 653 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:29 192.0.0.1  05/07/2022:12:54:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 654 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:29 192.0.0.1  05/07/2022:12:54:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 655 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:39 192.0.0.1  05/07/2022:12:54:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 656 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:39 192.0.0.1  05/07/2022:12:54:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 657 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:39 192.0.0.1  05/07/2022:12:54:39 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 658 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:49 192.0.0.1  05/07/2022:12:54:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 659 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:49 192.0.0.1  05/07/2022:12:54:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 660 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:49 192.0.0.1  05/07/2022:12:54:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 661 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:54:59 192.0.0.1  05/07/2022:12:54:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 662 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:54:59 192.0.0.1  05/07/2022:12:54:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 663 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:54:59 192.0.0.1  05/07/2022:12:54:59 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 664 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:55:00 cpx-ingress-64464fdb75-9wchv CRON[926]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:55:09 192.0.0.1  05/07/2022:12:55:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 665 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:55:09 192.0.0.1  05/07/2022:12:55:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 666 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:55:09 192.0.0.1  05/07/2022:12:55:09 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 667 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:55:19 192.0.0.1  05/07/2022:12:55:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 668 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:55:19 192.0.0.1  05/07/2022:12:55:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 669 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:55:19 192.0.0.1  05/07/2022:12:55:19 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 670 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:55:29 192.0.0.1  05/07/2022:12:55:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 671 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:55:29 192.0.0.1  05/07/2022:12:55:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 672 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:55:29 192.0.0.1  05/07/2022:12:55:29 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 673 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:55:40 192.0.0.1  05/07/2022:12:55:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 674 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:55:40 192.0.0.1  05/07/2022:12:55:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 675 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:55:40 192.0.0.1  05/07/2022:12:55:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 676 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:55:49 192.0.0.1  05/07/2022:12:55:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 677 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:55:49 192.0.0.1  05/07/2022:12:55:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 678 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:55:49 192.0.0.1  05/07/2022:12:55:49 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 679 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:00 192.0.0.1  05/07/2022:12:56:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 680 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:00 192.0.0.1  05/07/2022:12:56:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 681 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:00 192.0.0.1  05/07/2022:12:56:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 682 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:00 cpx-ingress-64464fdb75-9wchv CRON[932]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:56:10 192.0.0.1  05/07/2022:12:56:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 683 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:10 192.0.0.1  05/07/2022:12:56:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 684 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:10 192.0.0.1  05/07/2022:12:56:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 685 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:20 192.0.0.1  05/07/2022:12:56:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 686 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:20 192.0.0.1  05/07/2022:12:56:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 687 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:20 192.0.0.1  05/07/2022:12:56:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 688 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:30 192.0.0.1  05/07/2022:12:56:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 689 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:30 192.0.0.1  05/07/2022:12:56:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 690 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:30 192.0.0.1  05/07/2022:12:56:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 691 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:40 192.0.0.1  05/07/2022:12:56:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 692 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:40 192.0.0.1  05/07/2022:12:56:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 693 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:40 192.0.0.1  05/07/2022:12:56:40 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 694 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:56:50 192.0.0.1  05/07/2022:12:56:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 695 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:56:50 192.0.0.1  05/07/2022:12:56:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 696 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:56:50 192.0.0.1  05/07/2022:12:56:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 697 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:00 192.0.0.1  05/07/2022:12:57:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 698 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:00 192.0.0.1  05/07/2022:12:57:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 699 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:00 192.0.0.1  05/07/2022:12:57:00 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 700 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:01 cpx-ingress-64464fdb75-9wchv CRON[938]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:57:10 192.0.0.1  05/07/2022:12:57:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 701 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:10 192.0.0.1  05/07/2022:12:57:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 702 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:10 192.0.0.1  05/07/2022:12:57:10 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 703 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:20 192.0.0.1  05/07/2022:12:57:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 704 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:20 192.0.0.1  05/07/2022:12:57:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 705 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:20 192.0.0.1  05/07/2022:12:57:20 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 706 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:30 192.0.0.1  05/07/2022:12:57:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 707 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:30 192.0.0.1  05/07/2022:12:57:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 708 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:30 192.0.0.1  05/07/2022:12:57:30 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 709 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:41 192.0.0.1  05/07/2022:12:57:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 710 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:41 192.0.0.1  05/07/2022:12:57:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 711 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:41 192.0.0.1  05/07/2022:12:57:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 712 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:57:50 192.0.0.1  05/07/2022:12:57:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 713 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:57:50 192.0.0.1  05/07/2022:12:57:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 714 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:57:50 192.0.0.1  05/07/2022:12:57:50 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 715 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:00 cpx-ingress-64464fdb75-9wchv CRON[944]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:58:01 192.0.0.1  05/07/2022:12:58:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 716 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:01 192.0.0.1  05/07/2022:12:58:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 717 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:01 192.0.0.1  05/07/2022:12:58:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 718 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:11 192.0.0.1  05/07/2022:12:58:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 719 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:11 192.0.0.1  05/07/2022:12:58:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 720 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:11 192.0.0.1  05/07/2022:12:58:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 721 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:21 192.0.0.1  05/07/2022:12:58:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 722 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:21 192.0.0.1  05/07/2022:12:58:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 723 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:21 192.0.0.1  05/07/2022:12:58:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 724 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:31 192.0.0.1  05/07/2022:12:58:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 725 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:31 192.0.0.1  05/07/2022:12:58:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 726 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:31 192.0.0.1  05/07/2022:12:58:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 727 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:41 192.0.0.1  05/07/2022:12:58:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 728 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:41 192.0.0.1  05/07/2022:12:58:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 729 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:41 192.0.0.1  05/07/2022:12:58:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 730 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:58:51 192.0.0.1  05/07/2022:12:58:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 731 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:58:51 192.0.0.1  05/07/2022:12:58:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 732 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:58:51 192.0.0.1  05/07/2022:12:58:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 733 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:00 cpx-ingress-64464fdb75-9wchv CRON[950]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 12:59:01 192.0.0.1  05/07/2022:12:59:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 734 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:01 192.0.0.1  05/07/2022:12:59:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 735 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:01 192.0.0.1  05/07/2022:12:59:01 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 736 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:11 192.0.0.1  05/07/2022:12:59:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 737 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:11 192.0.0.1  05/07/2022:12:59:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 738 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:11 192.0.0.1  05/07/2022:12:59:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 739 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:21 192.0.0.1  05/07/2022:12:59:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 740 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:21 192.0.0.1  05/07/2022:12:59:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 741 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:21 192.0.0.1  05/07/2022:12:59:21 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 742 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:31 192.0.0.1  05/07/2022:12:59:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 743 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:31 192.0.0.1  05/07/2022:12:59:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 744 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:31 192.0.0.1  05/07/2022:12:59:31 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 745 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:41 192.0.0.1  05/07/2022:12:59:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 746 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:41 192.0.0.1  05/07/2022:12:59:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 747 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:41 192.0.0.1  05/07/2022:12:59:41 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 748 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 12:59:51 192.0.0.1  05/07/2022:12:59:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 749 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 12:59:51 192.0.0.1  05/07/2022:12:59:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 750 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 12:59:51 192.0.0.1  05/07/2022:12:59:51 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 751 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:01 cpx-ingress-64464fdb75-9wchv CRON[956]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 13:00:02 192.0.0.1  05/07/2022:13:00:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 752 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:02 192.0.0.1  05/07/2022:13:00:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 753 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:02 192.0.0.1  05/07/2022:13:00:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 754 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:11 192.0.0.1  05/07/2022:13:00:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 755 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:11 192.0.0.1  05/07/2022:13:00:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 756 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:11 192.0.0.1  05/07/2022:13:00:11 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 757 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:22 192.0.0.1  05/07/2022:13:00:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 758 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:22 192.0.0.1  05/07/2022:13:00:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 759 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:22 192.0.0.1  05/07/2022:13:00:22 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 760 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:32 192.0.0.1  05/07/2022:13:00:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 761 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:32 192.0.0.1  05/07/2022:13:00:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 762 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:32 192.0.0.1  05/07/2022:13:00:32 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 763 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:42 192.0.0.1  05/07/2022:13:00:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 764 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:42 192.0.0.1  05/07/2022:13:00:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 765 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:42 192.0.0.1  05/07/2022:13:00:42 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 766 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:00:52 192.0.0.1  05/07/2022:13:00:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 767 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:00:52 192.0.0.1  05/07/2022:13:00:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 768 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:00:52 192.0.0.1  05/07/2022:13:00:52 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 769 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:01:00 cpx-ingress-64464fdb75-9wchv CRON[962]: (root) CMD (/var/netscaler/bins/nsfsyncd -p)
May  7 13:01:02 192.0.0.1  05/07/2022:13:01:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 770 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:01:02 192.0.0.1  05/07/2022:13:01:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 771 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:01:02 192.0.0.1  05/07/2022:13:01:02 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 772 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"
May  7 13:01:12 192.0.0.1  05/07/2022:13:01:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 773 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "login nsroot "********"" - Status "Success"
May  7 13:01:12 192.0.0.1  05/07/2022:13:01:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 774 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "stat ns globalcntr -counters sys_cur_duration_sincestart" - Status "Success"
May  7 13:01:12 192.0.0.1  05/07/2022:13:01:12 GMT cpx-ingress-64464fdb75-9wchv 0-PPE-0 : default API CMD_EXECUTED 775 0 :  User nsroot - Remote_ip 127.0.0.1 - Command "logout" - Status "Success"

```



Logs of the other container `cic` which is nothing but the Citrix Ingress Controller.

```sh
pradeep:~$kubectl logs cpx-ingress-64464fdb75-9wchv cic        

 User has accepted EULA. Starting Triton 

Citrix Ingress Controller version: 1.18.5, build: Mon 20 Sep 09:50:55 UTC 2021 B:0dbea01c7e94406f6756bb24d4ff3eb23716093b
2022-05-07 12:22:19,242  - INFO - [nstriton.py:main:534] (MainThread) No config file is present
2022-05-07 12:22:19,243  - INFO - [nstriton.py:read_env_from_file:206] (MainThread) Env file /etc/citrix/.env doesn't exist: Skipping reading env file
2022-05-07 12:22:19,244  - INFO - [nstriton.py:initalize_k8s_client:279] (MainThread) Initialize the k8s client interface and k8sclienthelper
2022-05-07 12:22:19,245  - INFO - [client.py:__init__:220] (MainThread) Kubernetes parameters:
2022-05-07 12:22:19,245  - INFO - [client.py:__init__:221] (MainThread) 	Cluster:
2022-05-07 12:22:19,245  - INFO - [client.py:__init__:222] (MainThread) 		Kube API Server URL: https://10.96.0.1:443
2022-05-07 12:22:19,248  - INFO - [client.py:__init__:227] (MainThread) Setting Connect_timeout to 5 sec and Read_timeout to 10 sec for all requests methods.
2022-05-07 12:22:19,248  - INFO - [nstriton.py:main:674] (MainThread) ==============> Starting Citrix Ingress Controller <================
2022-05-07 12:22:19,248  - INFO - [nstriton.py:main:756] (MainThread) configured  NS_APPS_NAME_PREFIX as "k8s-"
2022-05-07 12:22:19,248  - INFO - [nstriton.py:main:796] (MainThread) NS is running as sidecar
2022-05-07 12:22:19,248  - INFO - [nstriton.py:main:813] (MainThread) Default VIP is $NS-VIP$. NS IP is 127.0.0.1
2022-05-07 12:22:19,248  - INFO - [nstriton.py:get_cpx_credentials:513] (MainThread) SIDECAR Mode: Trying to get credentials for CPX
2022-05-07 12:22:19,249  - INFO - [nstriton.py:read_cpx_credentials:504] (MainThread) SIDECAR Mode: Successfully read crendetials for CPX
2022-05-07 12:22:19,642  - INFO - [nstriton.py:main:982] (MainThread) Let's give some time to NSPPE to be ready.
2022-05-07 12:22:24,646  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for LOGLEVEL config variable value: info
2022-05-07 12:22:24,658  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for NS_PROTOCOL config variable value: http
2022-05-07 12:22:24,661  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for NS_PORT config variable value: 80
2022-05-07 12:22:24,662  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for IGNORE_NODE_EXTERNAL_IP config variable value: false
2022-05-07 12:22:24,664  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for POD_IPS_FOR_SERVICEGROUP_MEMBERS config variable value: false
2022-05-07 12:22:24,666  - INFO - [baseconfigmapvar.py:_load_data:60] (MainThread) Data loaded successful for NS_CONFIG_DNS_REC config variable value: false
2022-05-07 12:22:24,667  - INFO - [nitrointerface.py:register_ns_cfg_vars:791] (MainThread) register_ns_cfg_vars: CIC session variables updated successfully
2022-05-07 12:22:24,669  - INFO - [nitrointerface.py:apply_session_vars:765] (MainThread) CIC session variables ns_protocol:http ns_port:80 updated successfully
2022-05-07 12:22:24,669  - INFO - [nitrointerface.py:__init__:680] (MainThread) Citrix ADC ip address 127.0.0.1, NS vip = $NS-VIP$ , Lbrole = server User nsroot Password *******
2022-05-07 12:22:24,670  - INFO - [config_dispatcher.py:run:163] (Dispatcher) Starting Dispatcher thread
2022-05-07 12:22:24,880  - INFO - [kubeeventwriter.py:write:116] (MainThread) CIC Event: CONNECTED: Citrix ADC:127.0.0.1:80
2022-05-07 12:22:24,926  - INFO - [nitrointerface.py:create_transactional_lbvserver:4739] (MainThread) Successfully created transaction lbvserver for COE configuration: k8s_def_trans_server
2022-05-07 12:22:24,926  - INFO - [nitrointerface.py:__init__:731] (MainThread) Enabling the default profile for ssl parameter for sidecar CPX
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:enable_default_sslprofile:6181] (MainThread) Succesfully enabled default profile in ssl parameter object
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:743] (MainThread) Citrix ADC parameters:
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:744] (MainThread) 	ADC URL: http://127.0.0.1:80
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:745] (MainThread) 	ADC Entity Prefix is set: k8s-
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:746] (MainThread) 	Embedded ADC: True
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:747] (MainThread) 	Enable Certificate Validation: False
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:748] (MainThread) 	Monitor Citrix ADC: yes
2022-05-07 12:22:24,945  - INFO - [nitrointerface.py:log_ns_info:749] (MainThread) 	Monitor ADC probe frequency: 10
2022-05-07 12:22:24,959  - INFO - [nitrointerface.py:enable_ns_features:4885] (MainThread) ADC version major:13, minor:0, mr_major:79, mr_minor:64.
2022-05-07 12:22:25,033  - INFO - [nitrointerface.py:enable_ns_features:4902] (MainThread) Successfully enabled {'APIGateway', 'RESPONDER', 'AppFlow', 'CS', 'REWRITE'} features in Citrix ADC
2022-05-07 12:22:25,034  - INFO - [kubeeventwriter.py:write:116] (MainThread) CIC Event: SUCCESS: ENABLING INIT features on Citrix ADC:
2022-05-07 12:22:25,041  - INFO - [upgrade_entity_names.py:rename_on_upgrade:134] (MainThread) UPGRADE: This module will upgrade entity names from old naming convention to new naming convention.
2022-05-07 12:22:25,221  - INFO - [upgrade_entity_names.py:rename_on_upgrade:197] (MainThread) UPGRADE: No CSActions found with old naming format. No need to upgrade names.
2022-05-07 12:22:25,221  - INFO - [NSAppInterfaceInitHandler.py:operation:115] (MainThread) Configuration upgrade is successful
2022-05-07 12:22:25,222  - INFO - [nitrointerface.py:_test_user_edit_permission:4944] (MainThread) Processing test user permission to edit configuration
2022-05-07 12:22:25,222  - INFO - [nitrointerface.py:_test_user_edit_permission:4946] (MainThread) In this process, CIC will try to create a dummy lbvserver with name k8s-dummy_csvs_to_test_edit.deleteme
2022-05-07 12:22:25,258  - INFO - [nitrointerface.py:_test_user_edit_permission:4971] (MainThread) Successfully created test LB k8s-dummy_csvs_to_test_edit.deleteme  in NetScaler
2022-05-07 12:22:25,276  - INFO - [nitrointerface.py:_test_user_edit_permission:4976] (MainThread) Finished processing test user permission to edit configuration
2022-05-07 12:22:25,277  - INFO - [kubeeventwriter.py:write:116] (MainThread) CIC Event: SUCCESS: Test LB Vserver Creation on Citrix ADC:
2022-05-07 12:22:25,309  - INFO - [NSAppInterfaceInitHandler.py:operation:25] (MainThread) Successfully done adjustment of timeout parameters in CPX
2022-05-07 12:22:25,334  - INFO - [NSAppInterfaceInitHandler.py:operation:47] (MainThread) Default VIP is configured as 172.17.0.3
2022-05-07 12:22:25,335  - INFO - [kubeeventwriter.py:write:116] (MainThread) CIC Event: SUCCESS: GET Default VIP from Citrix ADC:
2022-05-07 12:22:25,513  - INFO - [nitrointerface.py:_perform_post_configure_operation:814] (MainThread) NetScaler UPTime is recorded as 56
2022-05-07 12:22:25,514  - INFO - [kubernetes.py:__init__:125] (MainThread) Ingress classes allowed: ['citrix']
2022-05-07 12:22:25,518  - INFO - [clienthelper.py:get:49] (MainThread) Resource not found:  namespace None
2022-05-07 12:22:25,521  - INFO - [clienthelper.py:get:49] (MainThread) Resource not found: /anthos.gke.io namespace None
2022-05-07 12:22:25,524  - INFO - [clienthelper.py:get:49] (MainThread) Resource not found: /ipamblocks/ namespace None
2022-05-07 12:22:25,524  - INFO - [nodeWatch.py:__init__:35] (MainThread) CNI found: default
2022-05-07 12:22:25,527  - INFO - [kubernetes.py:get_kubernetes_version:5235] (MainThread) Kubernetes server version is discovered as: v1.23.3
2022-05-07 12:22:25,528  - INFO - [client.py:update_ingress_version_to_v1:60] (MainThread) Ingress V1 is supported. Setting the prefix to /apis/networking.k8s.io/v1 to fetch ingresses
2022-05-07 12:22:25,536  - INFO - [referencemanager.py:__init__:128] (MainThread) Initializing Reference manager singleton
2022-05-07 12:22:25,849  - INFO - [clienthelper.py:get:49] (MainThread) Resource not found: /customresourcedefinitions/vips.citrix.com namespace None
2022-05-07 12:22:25,861  - INFO - [kubernetes.py:configure_cpx_for_all_apps:4197] (MainThread) ADC-SYNC:STARTED
2022-05-07 12:22:25,867  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.cpx-service
2022-05-07 12:22:25,877  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD kube-system.lbvserver.kube-dns
2022-05-07 12:22:25,886  - INFO - [nitrointerface.py:sync_ns_cs_apps:5286] (MainThread) Starting Syncing existing Citrix ADC configurations 
2022-05-07 12:22:25,904  - INFO - [nitrointerface.py:_handle_cs_vserver_sync:5071] (MainThread) No csvserver found in NetScaler
2022-05-07 12:22:25,939  - INFO - [nitrointerface.py:_handle_lb_vserver_sync:5121] (MainThread) No lbvserver found in NetScaler
2022-05-07 12:22:25,975  - INFO - [nitrointerface.py:_handle_responderpolicy_sync:5166] (MainThread) No responder policies found in NetScaler
2022-05-07 12:22:25,975  - INFO - [nitrointerface.py:sync_ns_cs_apps:5291] (MainThread) Completed Syncing existing Citrix ADC configurations
2022-05-07 12:22:25,975  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification cpx-service.default
2022-05-07 12:22:25,975  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification kubernetes.default
2022-05-07 12:22:25,976  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification kube-dns.kube-system
2022-05-07 12:22:25,976  - INFO - [nitrointerface.py:_initialize_binding_priority_pool:1299] (MainThread) Skipping priority pool adjustment as no csvserver found in NetScaler
2022-05-07 12:22:26,140  - INFO - [nitrointerface.py:handle_ns_monitoring:950] (MainThread) Starting Monitor thread that watches NetScaler state
2022-05-07 12:22:26,143  - INFO - [nitrointerface.py:configure_apps_during_sync:4077] (MainThread) ADC-SYNC: Ingress configuration completed. No downtime observed.
2022-05-07 12:22:26,143  - INFO - [nitrointerface.py:configure_apps_during_sync:4087] (MainThread) ADC-SYNC: Loadbalancer type service configuration completed. No downtime observed.
2022-05-07 12:22:26,150  - INFO - [kubernetes.py:configure_cpx_for_all_apps:4260] (MainThread) ADC-SYNC: Completed. Time Taken:0:00:00.288012.
2022-05-07 12:22:26,150  - INFO - [kubernetes.py:configure_cpx_for_all_apps:4261] (MainThread) ADC-SYNC: Nitro-Stats: defaultdict(<class 'int'>, {'get_filtered': 3})
2022-05-07 12:22:26,150  - INFO - [nitrointerface.py:add_cic_policystrmap:6189] (MainThread) Adding CIC stringmap in ADC
2022-05-07 12:22:26,334  - INFO - [nitrointerface.py:add_cic_policystrmap:6203] (MainThread) Succesfully added stringmap policy object
2022-05-07 12:22:33,715  - INFO - [nitrointerface.py:apply_session_vars:765] (MainThread) CIC session variables ns_protocol:http ns_port:80 updated successfully
2022-05-07 12:22:33,750  - INFO - [kubernetes.py:run:356] (SecretListener) Starting thread that watches /secrets...
2022-05-07 12:22:33,753  - INFO - [kubernetes.py:run:356] (DeploymentListener) Starting thread that watches /deployments...
2022-05-07 12:22:33,754  - INFO - [kubernetes.py:main_thread:687] (MainThread) Starting watch threads...
2022-05-07 12:22:33,756  - INFO - [kubernetes.py:run:356] (EndpointsListener) Starting thread that watches /endpoints...
2022-05-07 12:22:33,758  - INFO - [kubernetes.py:run:356] (IngressListener) Starting thread that watches /ingresses...
2022-05-07 12:22:33,759  - INFO - [kubernetes.py:run:356] (ServiceListener) Starting thread that watches /services...
2022-05-07 12:22:33,761  - INFO - [kubernetes.py:run:356] (CRDListener) Starting thread that watches /customresourcedefinitions...
2022-05-07 12:22:33,766  - INFO - [kubernetes.py:run:356] (NodesListener) Starting thread that watches /nodes...
2022-05-07 12:22:33,767  - INFO - [kubernetes.py:run:356] (IngressClassListener) Starting thread that watches /ingressclasses...
2022-05-07 12:22:33,915  - INFO - [config_dispatcher.py:_pull_crd_config_from_ns:481] (Dispatcher) Pulling CRD configuration from NetScaler started
2022-05-07 12:22:35,407  - INFO - [config_dispatcher.py:_pull_crd_config_from_ns:531] (Dispatcher) Pulling CRD configuration from NetScaler Completed
2022-05-07 12:22:35,407  - INFO - [config_dispatcher.py:_synchronize_config:198] (Dispatcher) Config Synchronization started
2022-05-07 12:22:35,407  - INFO - [config_dispatcher.py:__dispatch_config_pack:298] (Dispatcher) Processing of ConfigPack '{ID:NetScaler Configuration_diff_delete+__synchronize_config___diff_add ConfigObjects(0)[]}' started
2022-05-07 12:22:35,408  - INFO - [config_dispatcher.py:__dispatch_config_pack:352] (Dispatcher) Processing of ConfigPack 'NetScaler Configuration_diff_delete+__synchronize_config___diff_add' is successful
2022-05-07 12:22:35,408  - INFO - [config_dispatcher.py:_synchronize_config:221] (Dispatcher) Config Synchronization ended
2022-05-07 12:23:02,993  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification redis-master.default
2022-05-07 12:23:03,197  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.redis-master
2022-05-07 12:23:03,388  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification redis-slave.default
2022-05-07 12:23:03,495  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.redis-slave
2022-05-07 12:23:03,496  - INFO - [kubernetes.py:kubernetes_service_to_nsapps:2867] (MainThread) Handling Service creation/Modification frontend.default
2022-05-07 12:23:03,580  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.frontend
2022-05-07 12:35:06,077  - INFO - [referencemanager.py:register_crd_instance:392] (MainThread) Adding new instance for CRD ANY-NS.IngressClass.citrix
2022-05-07 12:35:06,078  - INFO - [ingressclass.py:_create_instance:101] (MainThread) Created instance for citrix  IngressClass event_type:ADDED
2022-05-07 12:35:06,117  - INFO - [kubernetes.py:recalibrate_ingresses:4187] (MainThread) Processing to configure Ingress default.guestbook-ingress
2022-05-07 12:35:06,118  - INFO - [referencemanager.py:register_crd_instance:392] (MainThread) Adding new instance for CRD Ingress.default.guestbook-ingress
2022-05-07 12:35:06,118  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3271] (MainThread) Configuring Ingress resource name:guestbook-ingress namespace:default
2022-05-07 12:35:06,118  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3281] (MainThread) Configuring ingress guestbook-ingress default  Port:80 app port parameters: {'app_port': 80, 'protocol': 'http', 'port_allowed': True, 'tls_service': False, 'stylebook_params': {}, 'dns_mapping': {'www.guestbook.com'}}
2022-05-07 12:35:06,119  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3421] (MainThread) Policies of the app k8s-guestbook-ingress_80_default are
2022-05-07 12:35:06,120  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3430] (MainThread) policy name:k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g Rule:HTTP.REQ.HOSTNAME.SERVER.EQ("www.guestbook.com") && HTTP.REQ.URL.PATH.SET_TEXT_MODE(IGNORECASE).STARTSWITH("/") target LB App Name:k8s-frontend_80_default Service: frontend ServiceNameSpace:default Secret:None 
2022-05-07 12:35:06,120  - INFO - [nitrointerface.py:configure_ns_cs_app:4307] (MainThread) Configuring csvserver: k8s-172.17.0.3_80_http and associated services
2022-05-07 12:35:06,348  - INFO - [nitrointerface.py:_create_nsapp_cs_vserver:3089] (MainThread) csvserver k8s-172.17.0.3_80_http is created successfully
2022-05-07 12:35:06,348  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: ANY-NS.csvserver.172.17.0.3 - k8s-172.17.0.3_80_http
2022-05-07 12:35:06,349  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: default.csvserver_ingress.guestbook-ingress - k8s-172.17.0.3_80_http
2022-05-07 12:35:06,350  - INFO - [nitrointerface.py:create_entities_for_policy:2016] (MainThread) Processing lbvserver:k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g for csvserver:k8s-guestbook-ingress_80_default service type for lbvserver: http service type for servicegroup:http
2022-05-07 12:35:06,390  - INFO - [nitrointerface.py:_create_nsapp_vserver:1413] (MainThread) lbvserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g is created successfully
2022-05-07 12:35:06,425  - INFO - [nitrointerface.py:_create_cs_action:3378] (MainThread) cs action k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g create successful
2022-05-07 12:35:06,459  - INFO - [nitrointerface.py:_create_cs_policy:3570] (MainThread) CS policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g create successful
2022-05-07 12:35:06,505  - INFO - [policypriority.py:get_priority:677] (MainThread) Priority fetched 200000008 for VS: k8s-172.17.0.3_80_http and rule: HTTP.REQ.HOSTNAME.SERVER.EQ("www.guestbook.com") && HTTP.REQ.URL.PATH.SET_TEXT_MODE(IGNORECASE).STARTSWITH("/") 
2022-05-07 12:35:06,522  - INFO - [nitrointerface.py:_bind_cs_vserver_cs_policy:3668] (MainThread) csvserver k8s-172.17.0.3_80_http binding to CS policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g with priority:200000008 is successful
2022-05-07 12:35:06,556  - INFO - [nitrointerface.py:_create_nsapp_service_group:1631] (MainThread) Servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is created successfully
2022-05-07 12:35:06,597  - INFO - [nitrointerface.py:_bind_service_group_lb:1716] (MainThread) servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g bind to lbvserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,633  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.8:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,650  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.7:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,667  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.9:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,668  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: default.lbvserver.frontend - k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g
2022-05-07 12:35:06,668  - INFO - [nitrointerface.py:configure_ns_cs_app:4355] (MainThread) Finished processing instruction to configure k8s-guestbook-ingress_80_default app associated with k8s-172.17.0.3_80_http csvserver
2022-05-07 12:35:06,668  - INFO - [ingress.py:_handle_ingress_add_events:121] (MainThread) Processing of Ingress ADDED event guestbook-ingress.default is completed
2022-05-07 12:35:06,669  - INFO - [referencemanager.py:activate_crd:971] (MainThread) Activating CRD default.Ingress.guestbook-ingress
2022-05-07 12:35:06,669  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.Ingress.guestbook-ingress
2022-05-07 12:35:06,669  - INFO - [referencemanager.py:resolve:475] (MainThread) Resolving node default.Ingress.guestbook-ingress
2022-05-07 12:35:06,669  - INFO - [referencemanager.py:activate_crd:971] (MainThread) Activating CRD ANY-NS.IngressClass.citrix
2022-05-07 12:35:06,670  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD ANY-NS.IngressClass.citrix
2022-05-07 12:35:06,670  - INFO - [referencemanager.py:resolve:475] (MainThread) Resolving node ANY-NS.IngressClass.citrix
2022-05-07 12:35:06,670  - INFO - [referencemanager.py:register_crd_instance:392] (MainThread) Adding new instance for CRD Ingress.default.guestbook-ingress
2022-05-07 12:35:06,670  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3271] (MainThread) Configuring Ingress resource name:guestbook-ingress namespace:default
2022-05-07 12:35:06,670  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3281] (MainThread) Configuring ingress guestbook-ingress default  Port:80 app port parameters: {'app_port': 80, 'protocol': 'http', 'port_allowed': True, 'tls_service': False, 'stylebook_params': {}, 'dns_mapping': {'www.guestbook.com'}}
2022-05-07 12:35:06,671  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3421] (MainThread) Policies of the app k8s-guestbook-ingress_80_default are
2022-05-07 12:35:06,671  - INFO - [kubernetes.py:kubernetes_ingress_to_nsapps:3430] (MainThread) policy name:k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g Rule:HTTP.REQ.HOSTNAME.SERVER.EQ("www.guestbook.com") && HTTP.REQ.URL.PATH.SET_TEXT_MODE(IGNORECASE).STARTSWITH("/") target LB App Name:k8s-frontend_80_default Service: frontend ServiceNameSpace:default Secret:None 
2022-05-07 12:35:06,689  - INFO - [nitrointerface.py:is_csvserver_exists:3049] (MainThread) Ignoring as csvserver k8s-172.17.0.3_80_http already exists with same VIP and port
2022-05-07 12:35:06,689  - INFO - [nitrointerface.py:configure_ns_cs_app:4314] (MainThread) Updating k8s-guestbook-ingress_80_default App to k8s-172.17.0.3_80_http csvserver as csvserver is already configured
2022-05-07 12:35:06,711  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: ANY-NS.csvserver.172.17.0.3 - k8s-172.17.0.3_80_http
2022-05-07 12:35:06,711  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: default.csvserver_ingress.guestbook-ingress - k8s-172.17.0.3_80_http
2022-05-07 12:35:06,711  - INFO - [nitrointerface.py:create_entities_for_policy:2016] (MainThread) Processing lbvserver:k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g for csvserver:k8s-guestbook-ingress_80_default service type for lbvserver: http service type for servicegroup:http
2022-05-07 12:35:06,738  - INFO - [nitrointerface.py:_create_nsapp_vserver:1392] (MainThread) lbvserver k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g already exists
2022-05-07 12:35:06,770  - INFO - [nitrointerface.py:_create_cs_policy:3551] (MainThread) Ignoring creation of CS policy as CS policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g already exists with action k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g, rule HTTP.REQ.HOSTNAME.SERVER.EQ("www.guestbook.com") && HTTP.REQ.URL.PATH.SET_TEXT_MODE(IGNORECASE).STARTSWITH("/") 
2022-05-07 12:35:06,798  - INFO - [nitrointerface.py:_bind_cs_vserver_cs_policy:3653] (MainThread) csvserver k8s-172.17.0.3_80_http already binds policy k8s-frontend_80_csp_wpm4rauzaggggb4q4sc4ejbful6cun7g with priority 200000008
2022-05-07 12:35:06,848  - INFO - [nitrointerface.py:_bind_service_group_lb:1706] (MainThread) LB k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g is already bound to servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g
2022-05-07 12:35:06,891  - INFO - [NSAppInterfacePriorityPoolManager.py:createPriorityPool:383] (MainThread) NSPriorityPoolManagerPerCSVS: Skipping since k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g CS VS already has pool
2022-05-07 12:35:06,892  - INFO - [referencemanager.py:process_unmanaged_add_event:1033] (MainThread) Adding unmanaged entity: default.lbvserver.frontend - k8s-frontend_80_lbv_wpm4rauzaggggb4q4sc4ejbful6cun7g
2022-05-07 12:35:06,892  - INFO - [nitrointerface.py:configure_ns_cs_app:4355] (MainThread) Finished processing instruction to configure k8s-guestbook-ingress_80_default app associated with k8s-172.17.0.3_80_http csvserver
2022-05-07 12:35:06,892  - INFO - [ingress.py:_handle_ingress_add_events:121] (MainThread) Processing of Ingress ADDED event guestbook-ingress.default is completed
2022-05-07 12:35:06,892  - INFO - [referencemanager.py:activate_crd:971] (MainThread) Activating CRD default.Ingress.guestbook-ingress
2022-05-07 12:35:06,892  - WARNING - [referencemanager.py:activate_crd:977] (MainThread) CRD Modifcation in progress : Skipping activation
2022-05-07 12:35:06,892  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.Ingress.guestbook-ingress
2022-05-07 12:35:06,892  - INFO - [referencemanager.py:resolve_references:546] (MainThread) CRD Modification in progress: skipping resolve reference and processing as modification
2022-05-07 12:35:06,892  - INFO - [referencemanager.py:modify_references:914] (MainThread) Modify reference for CRD default.Ingress.guestbook-ingress
2022-05-07 12:39:32,190  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.cpx-service
2022-05-07 12:39:48,519  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.cpx-service
2022-05-07 12:44:53,253  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.cpx-service
2022-05-07 12:45:32,693  - INFO - [referencemanager.py:resolve_references:539] (MainThread) Resolve reference for CRD default.lbvserver.cpx-service
pradeep:~$

```

From the above logs, we can see that all the endpoints of `frontend` service are binded.

```sh
2022-05-07 12:35:06,633  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.8:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,650  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.7:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
2022-05-07 12:35:06,667  - INFO - [nitrointerface.py:_configure_services_nondesired:1916] (MainThread) Binding 172.17.0.9:80 from servicegroup k8s-frontend_80_sgp_wpm4rauzaggggb4q4sc4ejbful6cun7g is successful
```

```sh
pradeep:~$kubectl get ep
NAME           ENDPOINTS                                   AGE
cpx-service    172.17.0.3:443,172.17.0.3:80                48m
frontend       172.17.0.7:80,172.17.0.8:80,172.17.0.9:80   45m
kubernetes     192.168.49.2:8443                           49m
redis-master   172.17.0.6:6379                             45m
redis-slave    172.17.0.4:6379,172.17.0.5:6379             45m
pradeep:~$
```

This concludes our initial testing of Citrix Ingress Controller on Kubernetes.

