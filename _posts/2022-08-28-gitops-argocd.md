---
layout: single
title:  "Getting Started with ArgoCD "
date:   2022-08-28 11:55:04 +0530
categories: Kubernetes
tags: ArgoCD
classes: wide
show_date: true
header:
  overlay_image: /assets/images/argo.png
  og_image: /assets/images/argo.png
  teaser: /assets/images/argo.png
  caption: "Photo credit: [**Argo**](https://argo-cd.readthedocs.io/en/stable/)"
  actions:
    - label: "Learn more"
      url: "https://argo-cd.readthedocs.io/en/stable/"

author:
  name     : "ArgoCD"
  avatar   : "/assets/images/argo.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

# Getting Started with ArgoCD - GitOps

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.

In this post, let us install ArgoCD in a minikube Kubernetes cluster and explore its user interface.

CodeFresh is offering an interactive training (theory + hands-on) on GitOps fundamentals.

For more info check out [https://learning.codefresh.io/](https://learning.codefresh.io/) .

First let us spin a new cluster using minikube.

```sh
(base) pradeep:~$minikube start 
😄  minikube v1.25.2 on Darwin 12.4
🎉  minikube 1.26.1 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.26.1
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

✨  Automatically selected the hyperkit driver. Other choices: virtualbox, ssh
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~$

```

Create a new namespace for ArgoCD.

```sh
(base) pradeep:~$kubectl create namespace argocd
namespace/argocd created
(base) pradeep:~$
```

Create all ArgoCD resources using the manifest file.

```sh
(base) pradeep:~$kubectl apply -n argocd -f https://raw.githubusercontent.com/codefresh-contrib/gitops-certification-examples/main/argocd-noauth/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-redis created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-secret created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
(base) pradeep:~$

```



Verify the resources created 

```sh
(base) pradeep:~$kubectl get all -n argocd
NAME                                      READY   STATUS              RESTARTS   AGE
pod/argocd-application-controller-0       0/1     ContainerCreating   0          114s
pod/argocd-dex-server-6bcd85b666-m7qjl    0/1     Init:0/1            0          114s
pod/argocd-redis-84558bbb99-bhd4d         1/1     Running             0          114s
pod/argocd-repo-server-6dc7b6f6b9-l6kl7   0/1     ContainerCreating   0          114s
pod/argocd-server-6fcf5d68db-c6lg2        0/1     ContainerCreating   0          114s

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/argocd-dex-server       ClusterIP   10.105.157.71   <none>        5556/TCP,5557/TCP,5558/TCP   114s
service/argocd-metrics          ClusterIP   10.98.118.244   <none>        8082/TCP                     114s
service/argocd-redis            ClusterIP   10.108.73.63    <none>        6379/TCP                     114s
service/argocd-repo-server      ClusterIP   10.102.43.4     <none>        8081/TCP,8084/TCP            114s
service/argocd-server           ClusterIP   10.96.199.251   <none>        80/TCP,443/TCP               114s
service/argocd-server-metrics   ClusterIP   10.97.45.79     <none>        8083/TCP                     114s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-dex-server    0/1     1            0           114s
deployment.apps/argocd-redis         1/1     1            1           114s
deployment.apps/argocd-repo-server   0/1     1            0           114s
deployment.apps/argocd-server        0/1     1            0           114s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-dex-server-6bcd85b666    1         1         0       114s
replicaset.apps/argocd-redis-84558bbb99         1         1         1       114s
replicaset.apps/argocd-repo-server-6dc7b6f6b9   1         1         0       114s
replicaset.apps/argocd-server-6fcf5d68db        1         1         0       114s

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   0/1     114s
(base) pradeep:~$

```

Expose the ArgoCD UI using a NodePort Service.

```yaml
(base) pradeep:~$cat argocd-ui-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    app: argocd-server
  managedFields:
  name: argocd-server-nodeport
  namespace: argocd
spec:
  ports:
  - nodePort: 30443
    port: 8080
    protocol: TCP
  selector:
    app.kubernetes.io/name: argocd-server
  type: NodePort

(base) pradeep:~$

```



```sh
(base) pradeep:~$kubectl apply -f argocd-ui-service.yaml 
service/argocd-server-nodeport created
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get service -n argocd
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-dex-server        ClusterIP   10.105.157.71    <none>        5556/TCP,5557/TCP,5558/TCP   5m44s
argocd-metrics           ClusterIP   10.98.118.244    <none>        8082/TCP                     5m44s
argocd-redis             ClusterIP   10.108.73.63     <none>        6379/TCP                     5m44s
argocd-repo-server       ClusterIP   10.102.43.4      <none>        8081/TCP,8084/TCP            5m44s
argocd-server            ClusterIP   10.96.199.251    <none>        80/TCP,443/TCP               5m44s
argocd-server-metrics    ClusterIP   10.97.45.79      <none>        8083/TCP                     5m44s
argocd-server-nodeport   NodePort    10.102.152.203   <none>        8080:30443/TCP               30s
(base) pradeep:~$minikube ip
172.16.30.11
(base) pradeep:~$
```

Lets open a web browser and open the url 172.16.30.11:30443

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-2.png)



```sh
(base) pradeep:~$kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > admin-pass.txt
(base) pradeep:~$cat admin-pass.txt 
HnHrq5degJR3l4PX%                                                         (base) pradeep:~$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-3.png)

```sh
(base) pradeep:~$curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.1.5/argocd-linux-amd64
(base) pradeep:~$
```

```sh
(base) pradeep:~$chmod +x /usr/local/bin/argocd
(base) pradeep:~$
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-8.png)

```sh
(base) pradeep:~$kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
simple-deployment-848675b7d-h2h8n   1/1     Running   0          2m36s
```

```sh
(base) pradeep:~$kubectl describe pods
Name:         simple-deployment-848675b7d-h2h8n
Namespace:    default
Priority:     0
Node:         minikube/172.16.30.11
Start Time:   Sun, 28 Aug 2022 16:30:59 +0530
Labels:       app=trivial-go-web-app
              pod-template-hash=848675b7d
Annotations:  <none>
Status:       Running
IP:           172.17.0.8
IPs:
  IP:           172.17.0.8
Controlled By:  ReplicaSet/simple-deployment-848675b7d
Containers:
  webserver-simple:
    Container ID:   docker://a80e8922e574b1b7860d0ee8f8d05af66158ecf231f8327b2564a939ec25ca02
    Image:          docker.io/kostiscodefresh/gitops-simple-app:v1.0
    Image ID:       docker-pullable://kostiscodefresh/gitops-simple-app@sha256:babc3a814fb27ddf736e34d6f20bde9d3d9191d4bc13e760d624f2143abde646
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 28 Aug 2022 16:31:09 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tn8cq (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-tn8cq:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m44s  default-scheduler  Successfully assigned default/simple-deployment-848675b7d-h2h8n to minikube
  Normal  Pulling    2m42s  kubelet            Pulling image "docker.io/kostiscodefresh/gitops-simple-app:v1.0"
  Normal  Pulled     2m34s  kubelet            Successfully pulled image "docker.io/kostiscodefresh/gitops-simple-app:v1.0" in 7.494980849s
  Normal  Created    2m34s  kubelet            Created container webserver-simple
  Normal  Started    2m34s  kubelet            Started container webserver-simple
(base) pradeep:~$
```



```sh
(base) pradeep:~$kubectl get svc
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP          29m
simple-service   NodePort    10.104.91.157   <none>        8080:31000/TCP   3m37s
(base) pradeep:~$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-9.png)
![]({{ site.url }}{{ site.baseurl }}/assets/images/argocd-10.png)

