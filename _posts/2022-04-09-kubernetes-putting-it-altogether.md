---
layout: single
title:  "Kubernetes Putting Together All Resources"
date:   2022-04-09 10:58:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
toc: true
toc_sticky: true
show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Putting Together All Resources

So far we have seen multiple Kubernetes resources but mostly independently. I would like to summarize the concepts discussed so far by deploying one or two simple applications covering most of the Kubernetes API resources.

So, I was searching for some cool demo apps and found these two game apps which I thought are very cool!

First one is from the [Official Docker Samples](https://github.com/dockersamples/k8s-wordsmith-demo), It is called the `wordsmith`, which was presented at DockerCon EU 2017 and 2018.

Other application is the  Pac-Man (`pacman`) which is  the classic arcade game from 90's  or early 2000's. I found this from [VMware {code}](https://developer.vmware.com/samples/7611/running-pac-man-on-tanzu-kubernetes-grid-clusters).

First we will deploy them independently in their respective namespaces and finally join them together with an Ingress resource.

##  Standard Kubernetes Application

As shown in [Kubernetes GitHub repository](https://github.com/kubernetes/community/blob/master/icons/examples/schemas/std-app.png), to deploy a standard application in Kubernetes, we might have to deploy a lot of these API resources.

![kubernetes-std-app]({{ site.url }}{{ site.baseurl }}/assets/images/kubernetes-std-app.png)

Reference [https://kubernetes.io/docs](https://kubernetes.io/docs) 

## Pac-Man Kubernetes App
As shown at the VMware website, here is the topology of the Pac-Man Kubernetes application representing the corresponding resources.

![kubernetes-pacman-app]({{ site.url }}{{ site.baseurl }}/assets/images/pacman-k8s.png)

The application is made up of the following components:

- Namespace

- Deployment
- MongoDB Pod
- DB Authentication configured
   - Attached to a PVC
   - Pac-Man Pod
- Node JS web front end that connects back to the MongoDB Pod by looking for the Pod DNS address internally
- RBAC Configuration for Pod Security and Service Account
- Secret which holds the data for the MongoDB Usernames and Passwords to be configured
- Service
   - Type: LoadBalancer Used to balance traffic to the Pac-Man Pods

### Namespace pacman

```sh
pradeep@learnk8s$ kubectl create namespace pacman
namespace/pacman created
```
### RBAC pacman

If we take a look at the `rbac.yaml` file under the `security` folder, we can see the YAML definitions for

-  `PodSecurityPolicy`
-  `ClusterRole`
- `RoleBinding`
-  `ClusterRoleBinding`.

```yaml
pradeep@learnk8s$ cat security/rbac.yaml 
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: pacman
spec:
  privileged: true
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pacman-clusterrole
rules:
- apiGroups:
  - policy
  resources:
  - podsecuritypolicies
  verbs:
  - use
  resourceNames:
  - pacman
- apiGroups: [""]
  resources: ["pods", "nodes"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pacman-clusterrole
  namespace: pacman
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pacman-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
- kind: ServiceAccount
  name: default
  namespace: pacman
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pacman-clusterrole
  namespace: pacman
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pacman-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
- kind: ServiceAccount
  name: default
  namespace: pacman
---
pradeep@learnk8s$ 

```
### Secrets pacman
In the `secret.yaml` file, we see five secrets are defined.

```yaml
pradeep@learnk8s$ cat security/secret.yaml 
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-users-secret
  namespace: pacman
type: Opaque 
data:
  database-admin-name: Y2x5ZGU=
  database-admin-password: Y2x5ZGU=
  database-name: cGFjbWFu
  database-password: cGlua3k=
  database-user: Ymxpbmt5
pradeep@learnk8s$ 

```

- database-admin-name: `clyde`
- database-admin-password: `clyde`
- database-name: `pacman`
- database-password: `pinky`
- database-user: `blinky`

```sh
pradeep@learnk8s$ echo Y2x5ZGU= | base64 -d
clyde%                                                                          pradeep@learnk8s$ echo Y2x5ZGU= | base64 -d
clyde%                                                                          pradeep@learnk8s$ echo cGFjbWFu | base64 -d
pacman%                                                                         pradeep@learnk8s$ echo cGlua3k= | base64 -d
pinky%                                                                          pradeep@learnk8s$ echo Ymxpbmt5 | base64 -d
blinky%                                                                         pradeep@learnk8s$ 
```

### Persistent Volume Claim pacman
Take a look at the definition of the `mongo-pvc.yaml` manifest in `persistentvolumeclaim` directory.

```yaml
pradeep@learnk8s$ cat persistentvolumeclaim/mongo-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-storage
  namespace: pacman
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
pradeep@learnk8s$ 
```
### Deployment mongo pacman
The `mongo-deployment.yaml` manifest contents are like this:

```yaml
pradeep@learnk8s$ cat deployments/mongo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: mongo
  name: mongo
  annotations:
    source: "https://github.com/saintdle/pacman-tanzu"
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mongo
  template:
    metadata:
      labels:
        name: mongo
    spec:
      initContainers:
      - args:
        - |
          mkdir -p /bitnami/mongodb
          chown -R "1001:1001" "/bitnami/mongodb"
        command:
        - /bin/bash
        - -ec
        image: docker.io/bitnami/bitnami-shell:10-debian-10-r158
        imagePullPolicy: Always
        name: volume-permissions
        resources: {}
        securityContext:
          runAsUser: 0
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /bitnami/mongodb
          name: mongo-db
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1001
      serviceAccountName: default
      terminationGracePeriodSeconds: 30
      volumes:
      - name: mongo-db
        persistentVolumeClaim:
          claimName: mongo-storage
      containers:
      - image: bitnami/mongodb:4.4.8
        name: mongo
        env:
        - name: MONGODB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-admin-password
              name: mongodb-users-secret
        - name: MONGODB_DATABASE
          valueFrom:
            secretKeyRef:
              key: database-name
              name: mongodb-users-secret
        - name: MONGODB_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: mongodb-users-secret
        - name: MONGODB_USERNAME
          valueFrom:
            secretKeyRef:
              key: database-user
              name: mongodb-users-secret
        readinessProbe:
          exec:
           command:
            - /bin/sh
            - -i
            - -c
            - mongo 127.0.0.1:27017/$MONGODB_DATABASE -u $MONGODB_USERNAME -p $MONGODB_PASSWORD
              --eval="quit()"
        ports:
        - name: mongo
          containerPort: 27017
        volumeMounts:
          - name: mongo-db
            mountPath: /bitnami/mongodb/                                
pradeep@learnk8s$ 
```
We can see this deployment is making use of many kubernetes resources previously defined: `secrets`, `persistentvolumeclaims`. Also, we can see `Init Containers`, `Readiness Probes`, `Labels`, `Security Contexts`, `Commands` and `arguments`, `env` variables, and  `serive accounts` are being utilized here. Also, look at the `annotations`.

### Service mongo pacman

Definition of the `mongo-service.yaml`

```yaml
pradeep@learnk8s$ cat services/mongo-service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mongo
  name: mongo
spec:
  type: ClusterIP
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    name: mongo
pradeep@learnk8s$
```

Mongo DB pods are exposed as a `ClusterIP` service on port number `27017` (the default port used for MongoDB)

### Deployment pacman

There is one more deployment called `pacman` for this application which is based on the `quay.io/ifont/pacman-nodejs-app:latest` image. In this deployment, we can see the utilization of 

- annotations
- labels
- liveness probes
- readiness probes
- secrets
- environment variables
- jsonpath (`spec.nodeName`)

```yaml
pradeep@learnk8s$ cat deployments/pacman-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: pacman
  name: pacman
  annotations:
    source: "https://github.com/saintdle/pacman-tanzu"
spec:
  replicas: 1
  selector:
    matchLabels:
      name: pacman
  template:
    metadata:
      labels:
        name: pacman
    spec:
      containers:
      - image: quay.io/ifont/pacman-nodejs-app:latest
        name: pacman
        ports:
        - containerPort: 8080
          name: http-server
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /
            port: 8080
        readinessProbe:
          httpGet:
            path: /
            port: 8080
        env:
        - name: MONGO_SERVICE_HOST
          value: mongo
        - name: MONGO_AUTH_USER
          valueFrom:
            secretKeyRef:
              key: database-user
              name: mongodb-users-secret
        - name: MONGO_AUTH_PWD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: mongodb-users-secret
        - name: MONGO_DATABASE
          value: pacman
        - name: MY_MONGO_PORT
          value: "27017"
        - name: MONGO_USE_SSL
          value: "false"
        - name: MONGO_VALIDATE_SSL
          value: "false"
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: spec.nodeName
pradeep@learnk8s$ 

```

### Service pacman

Let us look at the `pacman-service.yaml`

```yaml
pradeep@learnk8s$ cat services/pacman-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: pacman
  labels:
    name: pacman
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    name: pacman
pradeep@learnk8s$
```

We can see that a `LoadBalancer` type service called `pacman` is defined here to make the Pod port (`targetPort: 8080`) available at ClusterIP:port 80 (`port: 80`).

### Applying all pacman resources together



There is a `shell` script provided in the github repository to apply all these resources in one go.

```sh
pradeep@learnk8s$ cat pacman-install.sh 
#!/bin/sh

kubectl create namespace pacman
kubectl create -n pacman -f security/rbac.yaml
kubectl create -n pacman -f security/secret.yaml
kubectl create -n pacman -f persistentvolumeclaim/mongo-pvc.yaml
kubectl create -n pacman -f deployments/mongo-deployment.yaml
while [ "$(kubectl get pods -l=name='mongo' -n pacman -o jsonpath='{.items[*].status.containerStatuses[0].ready}')" != "true" ]; do
   sleep 5
   echo "Waiting for mongo pod to change to running status"
done
kubectl create -n pacman -f deployments/pacman-deployment.yaml
kubectl create -n pacman -f services/mongo-service.yaml
kubectl create -n pacman -f services/pacman-service.yaml
pradeep@learnk8s$
```

We can either run this shell script or run each command manually.

```sh
pradeep@learnk8s$ kubectl create -n pacman -f security/rbac.yaml
kubectl create -n pacman -f security/secret.yaml
kubectl create -n pacman -f persistentvolumeclaim/mongo-pvc.yaml
kubectl create -n pacman -f deployments/mongo-deployment.yaml
while [ "$(kubectl get pods -l=name='mongo' -n pacman -o jsonpath='{.items[*].status.containerStatuses[0].ready}')" != "true" ]; do
   sleep 5
   echo "Waiting for mongo pod to change to running status"
done
kubectl create -n pacman -f deployments/pacman-deployment.yaml
kubectl create -n pacman -f services/mongo-service.yaml
kubectl create -n pacman -f services/pacman-service.yaml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/pacman created
clusterrole.rbac.authorization.k8s.io/pacman-clusterrole created
rolebinding.rbac.authorization.k8s.io/pacman-clusterrole created
clusterrolebinding.rbac.authorization.k8s.io/pacman-clusterrole created
secret/mongodb-users-secret created
persistentvolumeclaim/mongo-storage created
deployment.apps/mongo created
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
Waiting for mongo pod to change to running status
deployment.apps/pacman created
service/mongo created
service/pacman created
pradeep@learnk8s$
```



### Verifying pacman resources

```sh
pradeep@learnk8s$ kubectl get all -n pacman 
NAME                         READY   STATUS    RESTARTS   AGE
pod/mongo-7955ccdfb4-sv227   1/1     Running   0          3m49s
pod/pacman-d44c74699-kwmf5   1/1     Running   0          2m34s

NAME             TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
service/mongo    ClusterIP      10.106.16.246    <none>           27017/TCP      2m33s
service/pacman   LoadBalancer   10.111.159.169   10.111.159.169   80:30634/TCP   2m32s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo    1/1     1            1           3m49s
deployment.apps/pacman   1/1     1            1           2m34s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/mongo-7955ccdfb4   1         1         1       3m49s
replicaset.apps/pacman-d44c74699   1         1         1       2m34s
pradeep@learnk8s$
```

We can see that there are 

- two pods

- two replicasets

- two deployments and 

- two services

  

  > For the `LoadBalancer` service to show `EXTERNAL-IP`, in our minikube envrironment, we have to open another terminal window and issue the  `minikube tunnel` command.

```sh
pradeep@learnk8s$ minikube tunnel
Password:
Status:	
	machine: minikube
	pid: 3776
	route: 10.96.0.0/12 -> 172.16.30.6
	minikube: Running
	services: [pacman]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors
```

Use the `minikube service` command with `--url`option to get the URL for this application

```sh
pradeep@learnk8s$ minikube service pacman -n pacman --url
http://172.16.30.6:30634
pradeep@learnk8s$ 
```



Let us open a browser and point to this URL

![pacman-service-url]({{ site.url }}{{ site.baseurl }}/assets/images/pacman-service-url.jpeg)

Start playing by clicking on this window.

![pacman-game]({{ site.url }}{{ site.baseurl }}/assets/images/pacman-game.jpeg)

Don't forget to turn the music! It is disabled by default, just click on the Speaker icon.

## Wordsmith Kubernetes App

According to the [Docker Samples GitHub repository](https://github.com/dockersamples/k8s-wordsmith-demo), Wordsmith is the demo project shown at DockerCon EU 2017 and 2018.

This demo app runs across three containers:

- [db](https://github.com/dockersamples/k8s-wordsmith-demo/blob/master/db/Dockerfile) - a Postgres database which stores words
- [words](https://github.com/dockersamples/k8s-wordsmith-demo/blob/master/words/Dockerfile) - a Java REST API which serves words read from the database
- [web](https://github.com/dockersamples/k8s-wordsmith-demo/blob/master/web/Dockerfile) - a Go web application which calls the API and builds words into sentences:

Let us deploy these resources as our second app.

### Namespace wordsmith

```sh
pradeep@learnk8s$ kubectl create ns wordsmith
namespace/wordsmith created
pradeep@learnk8s$
```

### Deployments and Services wordsmith

In the [Docker Samples GitHub repository](https://github.com/dockersamples/k8s-wordsmith-demo), there is a single manifest file which defines three deployments and corresponding three services.

Deployments: 

- words-db 
- words-api
- words-web

Services:

- db
- words
- Web

```yaml
pradeep@learnk8s$ cat wordsmith.yaml 
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: words-db
spec:
  ports:
    - port: 5432
      targetPort: 5432
      name: db
  selector:
    app: words-db
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  labels:
    app: words-db
spec:
  selector:
    matchLabels:
      app: words-db
  template:
    metadata:
      labels:
        app: words-db
    spec:
      containers:
      - name: db
        image: dockersamples/k8s-wordsmith-db
        ports:
        - containerPort: 5432
          name: db
---
apiVersion: v1
kind: Service
metadata:
  name: words
  labels:
    app: words-api
spec:
  ports:
    - port: 8080
      targetPort: 8080
      name: api
  selector:
    app: words-api
  clusterIP: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: words
  labels:
    app: words-api
spec:
  replicas: 5
  selector:
    matchLabels:
      app: words-api
  template:
    metadata:
      labels:
        app: words-api
    spec:
      containers:
      - name: words
        image: dockersamples/k8s-wordsmith-api
        ports:
        - containerPort: 8080
          name: api
---
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    app: words-web
spec:
  ports:
    - port: 8081
      targetPort: 80
      name: web
  selector:
    app: words-web
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: words-web
spec:
  selector:
    matchLabels:
      app: words-web
  template:
    metadata:
      labels:
        app: words-web
    spec:
      containers:
      - name: web
        image: dockersamples/k8s-wordsmith-web
        ports:
        - containerPort: 80
          name: words-web
pradeep@learnk8s$

```

### Applying resources wordsmith

```sh
pradeep@learnk8s$ kubectl create -f wordsmith.yaml -n wordsmith      
service/db created
deployment.apps/db created
service/words created
deployment.apps/words created
service/web created
deployment.apps/web created
pradeep@learnk8s$
```

As expected, we can see that three deployments and three services got created in the `wordsmith` namespace.

### Verifying resources wordsmith

```sh
pradeep@learnk8s$ kubectl get all -n wordsmith
NAME                         READY   STATUS    RESTARTS   AGE
pod/db-85c89687b-r2vnb       1/1     Running   0          52s
pod/web-78684f4b46-xrz5v     1/1     Running   0          52s
pod/words-6fc5cf68df-56vtl   1/1     Running   0          52s
pod/words-6fc5cf68df-7fk94   1/1     Running   0          52s
pod/words-6fc5cf68df-kbxvc   1/1     Running   0          52s
pod/words-6fc5cf68df-rg6jn   1/1     Running   0          52s
pod/words-6fc5cf68df-v4tc2   1/1     Running   0          52s

NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/db      ClusterIP      None            <none>        5432/TCP         53s
service/web     LoadBalancer   10.101.238.36   <pending>     8081:30246/TCP   52s
service/words   ClusterIP      None            <none>        8080/TCP         53s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db      1/1     1            1           53s
deployment.apps/web     1/1     1            1           52s
deployment.apps/words   5/5     5            5           53s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/db-85c89687b       1         1         1       53s
replicaset.apps/web-78684f4b46     1         1         1       52s
replicaset.apps/words-6fc5cf68df   5         5         5       53s
pradeep@learnk8s$
```

A total of seven pods, three replicasets, three deployments, and three services. One deployment `words` has five replicas.

> As seen earlier, for the `LoadBalancer` type service to obtain `EXTERNAL-IP`, in our minikube environment, we have to issue the `minikube tunnel` command.



```sh
pradeep@learnk8s$ minikube tunnel
Password:
Status:	
	machine: minikube
	pid: 3776
	route: 10.96.0.0/12 -> 172.16.30.6
	minikube: Running
	services: [web]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors
```

After this, we can see that EXTERNAL-IP is changed from `<pending>` to `10.101.238.36`.

```sh
pradeep@learnk8s$ kubectl get all -n wordsmith
NAME                         READY   STATUS    RESTARTS   AGE
pod/db-85c89687b-r2vnb       1/1     Running   0          3m35s
pod/web-78684f4b46-xrz5v     1/1     Running   0          3m35s
pod/words-6fc5cf68df-56vtl   1/1     Running   0          3m35s
pod/words-6fc5cf68df-7fk94   1/1     Running   0          3m35s
pod/words-6fc5cf68df-kbxvc   1/1     Running   0          3m35s
pod/words-6fc5cf68df-rg6jn   1/1     Running   0          3m35s
pod/words-6fc5cf68df-v4tc2   1/1     Running   0          3m35s

NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
service/db      ClusterIP      None            <none>          5432/TCP         3m36s
service/web     LoadBalancer   10.101.238.36   10.101.238.36   8081:30246/TCP   3m35s
service/words   ClusterIP      None            <none>          8080/TCP         3m36s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db      1/1     1            1           3m36s
deployment.apps/web     1/1     1            1           3m35s
deployment.apps/words   5/5     5            5           3m36s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/db-85c89687b       1         1         1       3m36s
replicaset.apps/web-78684f4b46     1         1         1       3m35s
replicaset.apps/words-6fc5cf68df   5         5         5       3m36s
pradeep@learnk8s$ 
```

Let us look at all the pods in the `wordsmith` namespace with the `-o wide` option 

```sh
pradeep@Pradeeps-MacBook-Air ~ % kubectl get pods -o wide -n wordsmith
NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
db-85c89687b-r2vnb       1/1     Running   0          6m38s   10.244.151.8     minikube-m03   <none>           <none>
web-78684f4b46-xrz5v     1/1     Running   0          6m38s   10.244.151.6     minikube-m03   <none>           <none>
words-6fc5cf68df-56vtl   1/1     Running   0          6m38s   10.244.151.9     minikube-m03   <none>           <none>
words-6fc5cf68df-7fk94   1/1     Running   0          6m38s   10.244.205.202   minikube-m02   <none>           <none>
words-6fc5cf68df-kbxvc   1/1     Running   0          6m38s   10.244.120.68    minikube       <none>           <none>
words-6fc5cf68df-rg6jn   1/1     Running   0          6m38s   10.244.151.7     minikube-m03   <none>           <none>
words-6fc5cf68df-v4tc2   1/1     Running   0          6m38s   10.244.205.201   minikube-m02   <none>           <none>
pradeep@Pradeeps-MacBook-Air ~ %
```



Use the `minikube service` command with `--url`option to get the URL for this application

```sh
pradeep@learnk8s$ minikube service web -n wordsmith --url
http://172.16.30.6:30246
pradeep@learnk8s$ 
```

Let us open a browser and point to this URL

![wordsmith-service-url]({{ site.url }}{{ site.baseurl }}/assets/images/wordsmith-service-url.jpeg)

Click refresh to see another set of words.

![wordsmith-game]({{ site.url }}{{ site.baseurl }}/assets/images/wordsmith-game.jpeg)

## Launch our game store (play.games.com)!

So far we have deployed two games and they are working fine. Users can access 

`wordsmith` game at [http://172.16.30.6:30246](http://172.16.30.6:30246)

and `pacman` game at [http://172.16.30.6:30634](http://172.16.30.6:30634) 

Look at those random  port numbers?! who remembers these random numbers? Wouldn't it be nice, if we have URLs like [http://play.games.com/pacman](http://play.games.com/pacman) and [http://play.games.com/wordsmith](http://play.games.com/wordsmith) ?

Thats where Kubernetes Ingress Controllers help us!

In an earlier post on [Kubernetes Ingress Controllers](https://pradeepgadde.com/blog/kubernetes/2022/03/21/kubernetes-ingress.html) we looked at the basics. Let us apply the same here and make these two apps available on a common URL with friendly names (rather than some obscure port numbers)

But here is a catch! Remember our two apps are living in their dedicated namespaces. So we have to create two ingress controller definitions, one in each namespace.

### Ingress pacman

```yaml
pradeep@learnk8s$ cat pacman-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pacman-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  namespace: pacman 
spec:
  rules:
    - host: play.games.com
      http:
        paths:
          - path: /pacman(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: pacman
                port:
                  number: 80 
pradeep@learnk8s$ 

```

We can see that namespace `pacman` specified in the `metadata` and the host is `play.games.com`. In our local lab setup, we have to make sure that this name `play.games.com` resolves to the IP obtained for the Ingress. For the backend, we are referring the `pacman` service on port number `80`.

>  Look at the annotation `nginx.ingress.kubernetes.io/rewrite-target: /$2` and `path: /pacman(/|$)(.*)` which are slightly different from the previous post on this topic [Kubernetes Ingress Controllers](https://pradeepgadde.com/blog/kubernetes/2022/03/21/kubernetes-ingress.html) . Without this change, the Ingress rules are working but static assets like images are not getting rendered (404 errors).



### Ingress wordsmith

```yaml
pradeep@learnk8s$ cat wordsmith-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordsmith-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  namespace: wordsmith  
spec:
  rules:
    - host: play.games.com
      http:
        paths:
          - path: /wordsmith(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8081
pradeep@learnk8s$ 

```

Similar to the `pacman` Ingress, this `wordsmith` ingress also defined in a namespace and using the same annotations with similar path settings and host information. For the backend, we are referring the `web` service on port number `8081`.



### Apply both Ingress rules pacman and wordsmith

```sh
pradeep@learnk8s$ kubectl create -f pacman-ingress.yaml 
ingress.networking.k8s.io/pacman-ingress created
pradeep@learnk8s$ 
```

```sh
pradeep@learnk8s$ kubectl create -f wordsmith-ingress.yaml
ingress.networking.k8s.io/wordsmith-ingress created
pradeep@learnk8s$
```

### Verifying Ingress rules 

Issue the `kubectl get ingress -A` command to list all ingress resources from all namespaces.

```sh
pradeep@learnk8s$ kubectl get ingress -A

NAMESPACE  NAME        CLASS  HOSTS      ADDRESS    PORTS  AGE

pacman   pacman-ingress   nginx  play.games.com  172.16.30.6  80   11m

wordsmith  wordsmith-ingress  nginx  play.games.com  172.16.30.6  80   2m21s

pradeep@learnk8s$
```

We have our two ingress resources: `pacman-ingress` and `wordsmith-ingress` of class `nginx`. Both are using the same host `play.games.com ` and ADDRESS of `172.16.30.6`.



Describe these Ingress resources

```sh
pradeep@learnk8s$ kubectl describe ingress -A
Name:             pacman-ingress
Namespace:        pacman
Address:          172.16.30.6
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host            Path  Backends
  ----            ----  --------
  play.games.com  
                  /pacman(/|$)(.*)   pacman:80 (10.244.205.203:8080)
Annotations:      nginx.ingress.kubernetes.io/rewrite-target: /$2
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    11m (x2 over 11m)  nginx-ingress-controller  Scheduled for sync


Name:             wordsmith-ingress
Namespace:        wordsmith
Address:          172.16.30.6
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host            Path  Backends
  ----            ----  --------
  play.games.com  
                  /wordsmith(/|$)(.*)   web:8081 (10.244.151.6:80)
Annotations:      nginx.ingress.kubernetes.io/rewrite-target: /$2
Events:
  Type    Reason  Age                   From                      Message
  ----    ------  ----                  ----                      -------
  Normal  Sync    2m2s (x2 over 2m29s)  nginx-ingress-controller  Scheduled for sync
pradeep@learnk8s$ 
```



Let us modify our `/etc/hosts` file to make this DNS resolution from `play.games.com` to `172.16.30.6` work in our local lab setup.

```sh
pradeep@learnk8s$ cat /etc/hosts
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
172.16.30.6	play.games.com
pradeep@learnk8s$

```

Let us open a browser and open these URLs.

Wordsmith URL

![wordsmith-ingress-url]({{ site.url }}{{ site.baseurl }}/assets/images/wordsmith-ingress-url.jpeg)

Pac-Man URL

![pacman-ingress-url]({{ site.url }}{{ site.baseurl }}/assets/images/pacman-ingress-url.jpeg)

> Note that the `wordsmith` app still has some issues with loading the images (missing information) and also notice the `/` at the end, without which the rendering is still bad. I guess these are related to the way actual applications are defined and URL rewriting. But we get the idea (of using Ingress) to meet our final goal.





With this we have achieved our goal and our two application kubernetes deployment looks like this and is working fine.

![pacman-wordsmith]({{ site.url }}{{ site.baseurl }}/assets/images/pacman-wordsmith.png)



I hope this summary is helpful to review many of the kubernetes topics.

Thank you for reading my notes.



Be Curious and Keep learning.

Best regards,

Pradeep
