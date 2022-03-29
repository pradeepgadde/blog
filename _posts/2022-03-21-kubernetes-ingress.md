---
layout: single
title:  "Kubernetes Ingress Controller"
date:   2022-03-21 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/nginx-ingress.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes Ingress Controller 

## Enable Ingress Addon

```sh
pradeep@learnk8s$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | third-party (ambassador)       |
| auto-pause                  | minikube | disabled     | google                         |
| csi-hostpath-driver         | minikube | disabled     | kubernetes                     |
| dashboard                   | minikube | disabled     | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| efk                         | minikube | disabled     | third-party (elastic)          |
| freshpod                    | minikube | disabled     | google                         |
| gcp-auth                    | minikube | disabled     | google                         |
| gvisor                      | minikube | disabled     | google                         |
| helm-tiller                 | minikube | disabled     | third-party (helm)             |
| ingress                     | minikube | disabled     | unknown (third-party)          |
| ingress-dns                 | minikube | disabled     | google                         |
| istio                       | minikube | disabled     | third-party (istio)            |
| istio-provisioner           | minikube | disabled     | third-party (istio)            |
| kong                        | minikube | disabled     | third-party (Kong HQ)          |
| kubevirt                    | minikube | disabled     | third-party (kubevirt)         |
| logviewer                   | minikube | disabled     | unknown (third-party)          |
| metallb                     | minikube | disabled     | third-party (metallb)          |
| metrics-server              | minikube | disabled     | kubernetes                     |
| nvidia-driver-installer     | minikube | disabled     | google                         |
| nvidia-gpu-device-plugin    | minikube | disabled     | third-party (nvidia)           |
| olm                         | minikube | disabled     | third-party (operator          |
|                             |          |              | framework)                     |
| pod-security-policy         | minikube | disabled     | unknown (third-party)          |
| portainer                   | minikube | disabled     | portainer.io                   |
| registry                    | minikube | disabled     | google                         |
| registry-aliases            | minikube | disabled     | unknown (third-party)          |
| registry-creds              | minikube | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | minikube | enabled ✅   | google                         |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party)          |
| volumesnapshots             | minikube | disabled     | kubernetes                     |
|-----------------------------|----------|--------------|--------------------------------|
pradeep@learnk8s$
```
Enable the `ingress` addon which is currently disabled in this environment.

```sh
pradeep@learnk8s$ minikube addons enable ingress
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
pradeep@learnk8s$
```
Verify the addons list again, and confirm that the `ingress` is enabled.

```sh
pradeep@learnk8s$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | third-party (ambassador)       |
| auto-pause                  | minikube | disabled     | google                         |
| csi-hostpath-driver         | minikube | disabled     | kubernetes                     |
| dashboard                   | minikube | disabled     | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| efk                         | minikube | disabled     | third-party (elastic)          |
| freshpod                    | minikube | disabled     | google                         |
| gcp-auth                    | minikube | disabled     | google                         |
| gvisor                      | minikube | disabled     | google                         |
| helm-tiller                 | minikube | disabled     | third-party (helm)             |
| ingress                     | minikube | enabled ✅   | unknown (third-party)          |
| ingress-dns                 | minikube | disabled     | google                         |
| istio                       | minikube | disabled     | third-party (istio)            |
| istio-provisioner           | minikube | disabled     | third-party (istio)            |
| kong                        | minikube | disabled     | third-party (Kong HQ)          |
| kubevirt                    | minikube | disabled     | third-party (kubevirt)         |
| logviewer                   | minikube | disabled     | unknown (third-party)          |
| metallb                     | minikube | disabled     | third-party (metallb)          |
| metrics-server              | minikube | disabled     | kubernetes                     |
| nvidia-driver-installer     | minikube | disabled     | google                         |
| nvidia-gpu-device-plugin    | minikube | disabled     | third-party (nvidia)           |
| olm                         | minikube | disabled     | third-party (operator          |
|                             |          |              | framework)                     |
| pod-security-policy         | minikube | disabled     | unknown (third-party)          |
| portainer                   | minikube | disabled     | portainer.io                   |
| registry                    | minikube | disabled     | google                         |
| registry-aliases            | minikube | disabled     | unknown (third-party)          |
| registry-creds              | minikube | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | minikube | enabled ✅   | google                         |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party)          |
| volumesnapshots             | minikube | disabled     | kubernetes                     |
|-----------------------------|----------|--------------|--------------------------------|
pradeep@learnk8s$
```
Verify all resources that got created as part of this addon installation.

```sh
pradeep@learnk8s$ kubectl get pods -n ingress-nginx
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-cvwv7       0/1     Completed   0          65s
ingress-nginx-admission-patch-zsrq4        0/1     Completed   1          65s
ingress-nginx-controller-cc8496874-6zr4h   1/1     Running     0          65s
pradeep@learnk8s$ kubectl get all -n ingress-nginx
NAME                                           READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-cvwv7       0/1     Completed   0          83s
pod/ingress-nginx-admission-patch-zsrq4        0/1     Completed   1          83s
pod/ingress-nginx-controller-cc8496874-6zr4h   1/1     Running     0          83s

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.101.85.4      <none>        80:30419/TCP,443:30336/TCP   83s
service/ingress-nginx-controller-admission   ClusterIP   10.104.163.136   <none>        443/TCP                      83s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           83s

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-cc8496874   1         1         1       83s

NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           10s        83s
job.batch/ingress-nginx-admission-patch    1/1           11s        83s
pradeep@learnk8s$ 
```
## Deploy a Hello, World App!
```sh
pradeep@learnk8s$ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/web created
pradeep@learnk8s$
```

Expose the deployment as  a NodePort Service on port number 8080.
```sh
pradeep@learnk8s$ kubectl expose deployment web --type=NodePort -
-port=8080
service/web exposed
pradeep@learnk8s$
```
Verify that the service is created properly.
```sh
pradeep@learnk8s$ kubectl get service web
NAME   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.105.47.136   <none>        8080:31270/TCP   11s
pradeep@learnk8s$
```
Get the URL for this newly created service.

```sh
pradeep@learnk8s$ minikube service web --url
http://172.16.30.6:31270
pradeep@learnk8s$
```
Access the URL
```sh
pradeep@learnk8s$ curl http://172.16.30.6:31270
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-nfkcw
pradeep@learnk8s$
```

## Create an Ingress
```yaml
pradeep@learnk8s$ cat example-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
pradeep@learnk8s$
```
Create the Ingress using the above definition.

```sh
pradeep@learnk8s$kubectl apply -f example-ingress.yaml
ingress.networking.k8s.io/example-ingress created
pradeep@learnk8s$
```

Verify that the Ingress is created.
```sh
pradeep@learnk8s$ kubectl get ingress
NAME              CLASS   HOSTS              ADDRESS       PORTS   AGE
example-ingress   nginx   hello-world.info   172.16.30.6   80      87s
pradeep@learnk8s$
```

Add the `172.16.30.6	hello-world.info` to the bottom of the /etc/hosts file on your computer (you will need administrator access):

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
172.16.30.6	hello-world.info
pradeep@learnk8s$
```

```sh
pradeep@learnk8s$ curl hello-world.info
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-nfkcw
pradeep@learnk8s$
```

## Create a second Deployment
```sh
pradeep@learnk8s$ kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
deployment.apps/web2 created
pradeep@learnk8s$
```
Expose the deployment.
```sh
pradeep@learnk8s$ kubectl expose deployment web2 --port=8080 --type=NodePort
service/web2 exposed
pradeep@learnk8s$
```

Edit the existing `example-ingress.yaml` manifest, and add the following lines at the end:

```yaml
pradeep@learnk8s$ cat example-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
          - path: /v2
            pathType: Prefix
            backend:
              service:
                name: web2
                port:
                  number: 8080
pradeep@learnk8s$
```
Apply the changes.
```sh
pradeep@learnk8s$ kubectl apply -f example-ingress.yaml
ingress.networking.k8s.io/example-ingress configured
pradeep@learnk8s$
```

## Test your Ingress
```sh
pradeep@learnk8s$ curl hello-world.info
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-nfkcw
pradeep@learnk8s$
```
Access the 2nd version of the Hello World app.

```sh
pradeep@learnk8s$ curl hello-world.info/v2
Hello, world!
Version: 2.0.0
Hostname: web2-5858b4c7c5-krt4l
pradeep@learnk8s$
```

```
pradeep@learnk8s$ kubectl get ingress
NAME              CLASS   HOSTS              ADDRESS       PORTS   AGE
example-ingress   nginx   hello-world.info   172.16.30.6   80      11m
pradeep@learnk8s$
```
Let us describe the ingress to see how it looks like.

```sh
pradeep@learnk8s$ kubectl describe ingress
Name:             example-ingress
Namespace:        default
Address:          172.16.30.6
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  hello-world.info
                    /     web:8080 (10.244.205.195:8080)
                    /v2   web2:8080 (10.244.151.3:8080)
Annotations:        nginx.ingress.kubernetes.io/rewrite-target: /$1
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    12m (x3 over 20m)  nginx-ingress-controller  Scheduled for sync
pradeep@learnk8s$
```

We can see one host (`hello-world.info`) and two paths (`/` and `/v2`), two backends (`web:8080` and `web2:8080`). These two services point to the respective Pods.

```sh
pradeep@learnk8s$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
web-746c8679d4-nfkcw    1/1     Running   0          75m   10.244.205.195   minikube-m02   <none>           <none>
web2-5858b4c7c5-krt4l   1/1     Running   0          66m   10.244.151.3     minikube-m03   <none>           <none>
pradeep@learnk8s$
```

What happens to the requests sent to a path different from both of these (`/` and `/v2`)?
Those requests are handled by the default backend.

From the above description, we can see there is a default backend `default-http-backend:80` which is not defined yet, so there is an error.
Let us test few undefined paths like (`/v3`, `/v5` etc)

```sh
pradeep@learnk8s$ curl hello-world.info/v3
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-nfkcw
pradeep@learnk8s$ curl hello-world.info/v5
Hello, world!
Version: 1.0.0
Hostname: web-746c8679d4-nfkcw
```

It looks like all these undefined paths are handled by the `web:8080` backend only. According to Kubernetes documentation, If `defaultBackend` is not set, the handling of requests that do not match any of the rules will be up to the ingress controller (consult the documentation for your ingress controller to find out how it handles this case).

If none of the hosts or paths match the HTTP request in the Ingress objects, the traffic is routed to your default backend.

Let us edit the ingress, to add a defaultBackend, in this case , change it to `web2`.

```sh
pradeep@learnk8s$ kubectl edit ingress example-ingress
ingress.networking.k8s.io/example-ingress edited
```
Describe it again, to see the changes.
```sh
pradeep@learnk8s$ kubectl describe ingress
Name:             example-ingress
Namespace:        default
Address:          172.16.30.6
Default backend:  web2:8080 (10.244.151.3:8080)
Rules:
  Host              Path  Backends
  ----              ----  --------
  hello-world.info
                    /     web:8080 (10.244.205.195:8080)
                    /v2   web2:8080 (10.244.151.3:8080)
Annotations:        nginx.ingress.kubernetes.io/rewrite-target: /$1
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    11s (x4 over 78m)  nginx-ingress-controller  Scheduled for sync
pradeep@learnk8s$
```

This line `Default backend:  web2:8080 (10.244.151.3:8080)` confirms our change.
List out all the endpoints.
```sh
pradeep@learnk8s$ kubectl get ep
NAME         ENDPOINTS             AGE
kubernetes   172.16.30.6:8443      47h
web          10.244.205.195:8080   87m
web2         10.244.151.3:8080     78m
pradeep@learnk8s$
```
Let us look at the ingress controller logs.
```sh
pradeep@learnk8s$ kubectl -n ingress-nginx logs ingress-nginx-controller-cc8496874-6zr4h
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.1.1
  Build:         a17181e43ec85534a6fea968d95d019c5a4bc8cf
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.9

-------------------------------------------------------------------------------

W0321 15:42:29.028539       9 client_config.go:615] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0321 15:42:29.028983       9 main.go:223] "Creating API client" host="https://10.96.0.1:443"
I0321 15:42:29.057278       9 main.go:267] "Running in Kubernetes cluster" major="1" minor="23" git="v1.23.3" state="clean" commit="816c97ab8cff8a1c72eccca1026f7820e93e0d25" platform="linux/amd64"
I0321 15:42:29.317187       9 main.go:104] "SSL fake certificate created" file="/etc/ingress-controller/ssl/default-fake-certificate.pem"
I0321 15:42:29.374201       9 ssl.go:531] "loading tls certificate" path="/usr/local/certificates/cert" key="/usr/local/certificates/key"
I0321 15:42:29.419869       9 nginx.go:255] "Starting NGINX Ingress controller"
I0321 15:42:29.437178       9 event.go:282] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"ingress-nginx-controller", UID:"d14f8541-76a0-4357-b9ad-38c7ce8f6586", APIVersion:"v1", ResourceVersion:"12223", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/ingress-nginx-controller
I0321 15:42:29.442938       9 event.go:282] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"tcp-services", UID:"9cffa0f1-6754-4bfd-bf78-6cae4febc393", APIVersion:"v1", ResourceVersion:"12224", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/tcp-services
I0321 15:42:29.443183       9 event.go:282] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"udp-services", UID:"3ac9b32a-17b9-4824-bc0b-51a609648e9c", APIVersion:"v1", ResourceVersion:"12225", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/udp-services
I0321 15:42:30.622870       9 nginx.go:297] "Starting NGINX process"
I0321 15:42:30.623750       9 leaderelection.go:248] attempting to acquire leader lease ingress-nginx/ingress-controller-leader...
I0321 15:42:30.624864       9 controller.go:155] "Configuration changes detected, backend reload required"
I0321 15:42:30.623967       9 nginx.go:317] "Starting validation webhook" address=":8443" certPath="/usr/local/certificates/cert" keyPath="/usr/local/certificates/key"
I0321 15:42:30.666454       9 leaderelection.go:258] successfully acquired lease ingress-nginx/ingress-controller-leader
I0321 15:42:30.667675       9 status.go:84] "New leader elected" identity="ingress-nginx-controller-cc8496874-6zr4h"
I0321 15:42:30.700891       9 status.go:214] "POD is not ready" pod="ingress-nginx/ingress-nginx-controller-cc8496874-6zr4h" node="minikube"
I0321 15:42:30.884747       9 controller.go:172] "Backend successfully reloaded"
I0321 15:42:30.885032       9 controller.go:183] "Initial sync, sleeping for 1 second"
I0321 15:42:30.885722       9 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-cc8496874-6zr4h", UID:"de6796ef-94a6-499c-b9b5-418c39cc62c0", APIVersion:"v1", ResourceVersion:"12337", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
I0321 15:56:40.753346       9 admission.go:149] processed ingress via admission controller {testedIngressLength:1 testedIngressTime:0.412s renderingIngressLength:1 renderingIngressTime:0.009s admissionTime:18.0kBs testedConfigurationSize:0.421}
I0321 15:56:40.755944       9 main.go:101] "successfully validated configuration, accepting" ingress="default/example-ingress"
I0321 15:56:40.794396       9 store.go:424] "Found valid IngressClass" ingress="default/example-ingress" ingressclass="nginx"
I0321 15:56:40.796827       9 event.go:282] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"example-ingress", UID:"51471ff6-1982-47da-85ee-a9356165df9f", APIVersion:"networking.k8s.io/v1", ResourceVersion:"13112", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
I0321 15:56:40.802072       9 controller.go:155] "Configuration changes detected, backend reload required"
I0321 15:56:40.979390       9 controller.go:172] "Backend successfully reloaded"
I0321 15:56:40.988286       9 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-cc8496874-6zr4h", UID:"de6796ef-94a6-499c-b9b5-418c39cc62c0", APIVersion:"v1", ResourceVersion:"12337", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
I0321 15:57:28.297662       9 status.go:299] "updating Ingress status" namespace="default" ingress="example-ingress" currentValue=[] newValue=[{IP:172.16.30.6 Hostname: Ports:[]}]
I0321 15:57:28.322904       9 event.go:282] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"example-ingress", UID:"51471ff6-1982-47da-85ee-a9356165df9f", APIVersion:"networking.k8s.io/v1", ResourceVersion:"13164", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
172.16.30.1 - - [21/Mar/2022:16:00:09 +0000] "GET / HTTP/1.1" 200 60 "-" "curl/7.77.0" 80 0.009 [default-web-8080] [] 10.244.205.195:8080 60 0.009 200 766ea45be48fc8dcb53c425eb3800ab2
I0321 16:04:49.720933       9 admission.go:149] processed ingress via admission controller {testedIngressLength:1 testedIngressTime:0.161s renderingIngressLength:1 renderingIngressTime:0s admissionTime:21.7kBs testedConfigurationSize:0.161}
I0321 16:04:49.721038       9 main.go:101] "successfully validated configuration, accepting" ingress="default/example-ingress"
I0321 16:04:49.733433       9 controller.go:155] "Configuration changes detected, backend reload required"
I0321 16:04:49.734529       9 event.go:282] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"example-ingress", UID:"51471ff6-1982-47da-85ee-a9356165df9f", APIVersion:"networking.k8s.io/v1", ResourceVersion:"13649", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
I0321 16:04:49.911585       9 controller.go:172] "Backend successfully reloaded"
I0321 16:04:49.914092       9 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-cc8496874-6zr4h", UID:"de6796ef-94a6-499c-b9b5-418c39cc62c0", APIVersion:"v1", ResourceVersion:"12337", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
172.16.30.1 - - [21/Mar/2022:16:05:30 +0000] "GET / HTTP/1.1" 200 60 "-" "curl/7.77.0" 80 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.004 200 c3d48047b7ef7ac70326098b25f1e6c1
172.16.30.1 - - [21/Mar/2022:16:05:33 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.013 [default-web2-8080] [] 10.244.151.3:8080 61 0.013 200 e90cf46a8385a64ac01c834734768488
172.16.30.1 - - [21/Mar/2022:16:32:44 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.114 [default-web2-8080] [] 10.244.151.3:8080 61 0.114 200 cb87a3a8abd12099f7db64a2ec83c681
172.16.30.1 - - [21/Mar/2022:17:03:17 +0000] "GET /v3 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.040 [default-web-8080] [] 10.244.205.195:8080 60 0.041 200 f794e2a783715ac73ff7177f60e9dd68
172.16.30.1 - - [21/Mar/2022:17:03:21 +0000] "GET /v5 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.016 [default-web-8080] [] 10.244.205.195:8080 60 0.017 200 095cbb53ca04bdb4359fa7b745e5a809
172.16.30.1 - - [21/Mar/2022:17:03:29 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.002 [default-web2-8080] [] 10.244.151.3:8080 61 0.002 200 cbc0c654a89c5f98f18a5102ddf61b0e
172.16.30.1 - - [21/Mar/2022:17:03:33 +0000] "GET /1 HTTP/1.1" 200 60 "-" "curl/7.77.0" 81 0.008 [default-web-8080] [] 10.244.205.195:8080 60 0.008 200 767955b2b4eed7791912b2a843363b28
172.16.30.1 - - [21/Mar/2022:17:05:26 +0000] "GET /1 HTTP/1.1" 200 60 "-" "curl/7.77.0" 81 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.005 200 0ec584df780661bccfa10731bcc70524
172.16.30.1 - - [21/Mar/2022:17:05:29 +0000] "GET / HTTP/1.1" 200 60 "-" "curl/7.77.0" 80 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 5ecb91527d77741daf16c17a8d79a462
172.16.30.1 - - [21/Mar/2022:17:05:31 +0000] "GET /as HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.006 [default-web-8080] [] 10.244.205.195:8080 60 0.005 200 034643529f3104fef19915593168b200
I0321 17:14:53.079003       9 admission.go:149] processed ingress via admission controller {testedIngressLength:1 testedIngressTime:0.41s renderingIngressLength:1 renderingIngressTime:0.007s admissionTime:21.7kBs testedConfigurationSize:0.417}
I0321 17:14:53.079746       9 main.go:101] "successfully validated configuration, accepting" ingress="default/example-ingress"
I0321 17:14:53.099157       9 controller.go:155] "Configuration changes detected, backend reload required"
I0321 17:14:53.099689       9 event.go:282] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"example-ingress", UID:"51471ff6-1982-47da-85ee-a9356165df9f", APIVersion:"networking.k8s.io/v1", ResourceVersion:"15050", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
I0321 17:14:53.225350       9 controller.go:172] "Backend successfully reloaded"
I0321 17:14:53.227604       9 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-cc8496874-6zr4h", UID:"de6796ef-94a6-499c-b9b5-418c39cc62c0", APIVersion:"v1", ResourceVersion:"12337", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
172.16.30.1 - - [21/Mar/2022:17:16:53 +0000] "GET /v5 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.009 [default-web-8080] [] 10.244.205.195:8080 60 0.009 200 2e8ee25ad012ea3d5ac6fb4646f37d61
172.16.30.1 - - [21/Mar/2022:17:16:57 +0000] "GET /v3 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.005 200 70d9d27891c2b97a6bcda5e315fa3691
172.16.30.1 - - [21/Mar/2022:17:17:01 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.010 [default-web2-8080] [] 10.244.151.3:8080 61 0.009 200 297effa7825bef53d93e6dce4e5d9d62
172.16.30.1 - - [21/Mar/2022:17:17:02 +0000] "GET /v1 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 421c9cb655e07183d9a7d1a89562db97
172.16.30.1 - - [21/Mar/2022:17:17:29 +0000] "GET /v1 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 3bff305b8c1b782d08acf2de386076d9
172.16.30.1 - - [21/Mar/2022:17:17:31 +0000] "GET /v3 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.001 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 e3dca522b7470345325f5dcda8adfc8e
172.16.30.1 - - [21/Mar/2022:17:17:34 +0000] "GET /v5 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 207b1c9f4b52683061bb0807857c32f6
172.16.30.1 - - [21/Mar/2022:17:17:36 +0000] "GET /v6 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 f2d0ae4264c26168246099ee90cc9f27
172.16.30.1 - - [21/Mar/2022:17:17:39 +0000] "GET /vdfs HTTP/1.1" 200 60 "-" "curl/7.77.0" 84 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.001 200 b08773ef6c074b4b10fee5626457df47
172.16.30.1 - - [21/Mar/2022:17:17:40 +0000] "GET /vdfs HTTP/1.1" 200 60 "-" "curl/7.77.0" 84 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 cd5a39f9d32b212bf97b41c57f7b429b
172.16.30.1 - - [21/Mar/2022:17:18:58 +0000] "GET /vdfs HTTP/1.1" 200 60 "-" "curl/7.77.0" 84 0.009 [default-web-8080] [] 10.244.205.195:8080 60 0.008 200 f413dd82a31c72df36114843c5054aad
172.16.30.1 - - [21/Mar/2022:17:18:59 +0000] "GET /vdfs HTTP/1.1" 200 60 "-" "curl/7.77.0" 84 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.004 200 09a92db2773acffcd23b31c856673a9b
172.16.30.1 - - [21/Mar/2022:17:19:12 +0000] "GET /v1 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.001 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 c21e9af32944af9d9d70b39eaedbe11c
172.16.30.1 - - [21/Mar/2022:17:19:14 +0000] "GET /v HTTP/1.1" 200 60 "-" "curl/7.77.0" 81 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 5daf84fe1f8992ff8d9d0cdec4b751c1
172.16.30.1 - - [21/Mar/2022:17:19:17 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.007 [default-web2-8080] [] 10.244.151.3:8080 61 0.008 200 52df2d3780d65d11e34a545583958a17
172.16.30.1 - - [21/Mar/2022:17:20:32 +0000] "GET /v2 HTTP/1.1" 200 61 "-" "curl/7.77.0" 82 0.003 [default-web2-8080] [] 10.244.151.3:8080 61 0.003 200 d05940e016c2b36ce79996c04065b5a8
172.16.30.1 - - [21/Mar/2022:17:20:34 +0000] "GET /v2f HTTP/1.1" 200 61 "-" "curl/7.77.0" 83 0.003 [default-web2-8080] [] 10.244.151.3:8080 61 0.002 200 c59de52e577935709573bf0515e999e2
172.16.30.1 - - [21/Mar/2022:17:20:36 +0000] "GET /v3 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 3354dc569894182a6ee5a5f69892ca66
172.16.30.1 - - [21/Mar/2022:17:20:40 +0000] "GET /v35 HTTP/1.1" 200 60 "-" "curl/7.77.0" 83 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.004 200 75cccfe798f3a6f2ee072c9233c8e3b7
172.16.30.1 - - [21/Mar/2022:17:20:44 +0000] "GET /v2a HTTP/1.1" 200 61 "-" "curl/7.77.0" 83 0.004 [default-web2-8080] [] 10.244.151.3:8080 61 0.004 200 8ae6170f4a95446d0615bfc5d4a16403
172.16.30.1 - - [21/Mar/2022:17:20:48 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 c4cea1304f930170d7a7148fc2ac04a7
172.16.30.1 - - [21/Mar/2022:17:21:49 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 b860ca473d1b622514dd4ebf194311f2
172.16.30.1 - - [21/Mar/2022:17:21:50 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 1f8bf150a942fb5272d01d9a82c6fd77
172.16.30.1 - - [21/Mar/2022:17:21:51 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 2a2c3eec3ebe85dd546ea35cf92f1357
172.16.30.1 - - [21/Mar/2022:17:21:52 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.005 [default-web-8080] [] 10.244.205.195:8080 60 0.005 200 fca07770b107dfb25126f60a1f4dabcc
172.16.30.1 - - [21/Mar/2022:17:21:52 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 421ddaffd339aafd3ac912c195035b7e
172.16.30.1 - - [21/Mar/2022:17:22:41 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 58a2dc6300d892b640bd26ab007daba8
172.16.30.1 - - [21/Mar/2022:17:22:42 +0000] "GET /dg HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 402a0a5fd8037182ea8671d3d9347ee7
172.16.30.1 - - [21/Mar/2022:17:22:44 +0000] "GET /df HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.004 200 4cb6f78cb734373510f98f669cdf253d
172.16.30.1 - - [21/Mar/2022:17:22:46 +0000] "GET /dfdg HTTP/1.1" 200 60 "-" "curl/7.77.0" 84 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.001 200 2fd8c843e0fac657bc80e2a6b23dcaf6
172.16.30.1 - - [21/Mar/2022:17:23:52 +0000] "GET /v7 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 5aaabd3963132e95499b5be8b57368e6
172.16.30.1 - - [21/Mar/2022:17:23:54 +0000] "GET /v9 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.002 [default-web-8080] [] 10.244.205.195:8080 60 0.002 200 452ce52c0ab19f2fdc37152f925887f7
172.16.30.1 - - [21/Mar/2022:17:24:28 +0000] "GET /v9 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.004 200 faf0e7f8be1b66b38f6747c7df815525
172.16.30.1 - - [21/Mar/2022:17:24:30 +0000] "GET /v9 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.004 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 c4526f5d5a9fab7245b1d77becf24bb7
172.16.30.1 - - [21/Mar/2022:17:24:31 +0000] "GET /v9 HTTP/1.1" 200 60 "-" "curl/7.77.0" 82 0.003 [default-web-8080] [] 10.244.205.195:8080 60 0.003 200 47fd2dd3ed81e5fa3762687343ed06c0
pradeep@learnk8s$
```
