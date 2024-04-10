---
layout: single
title:  "GCP GKE Multi-tenant Cluster with Namespaces"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/pca-gcp.png
  og_image: /assets/images/pca-gcp.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## GCP GKE Multi-tenant Cluster with Namespaces



- Create multiple namespaces in a GKE cluster.
- Configure role-based access control for namespace access.
- Configure Kubernetes resource quotas for fair sharing resources across multiple namespaces.
- View and configure monitoring dashboards to view resource usage by namespace.
- Generate a GKE metering report in Looker Studio for fine grained metrics on resource utilization by namespace.

```sh

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-77265de0bb34.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ gsutil -m cp -r gs://spls/gsp766/gke-qwiklab ~
Copying gs://spls/gsp766/gke-qwiklab/cpu-mem-demo-pod.yaml...
Copying gs://spls/gsp766/gke-qwiklab/cpu-mem-quota.yaml...                      
Copying gs://spls/gsp766/gke-qwiklab/developer-role.yaml...
Copying gs://spls/gsp766/gke-qwiklab/usage_metering_query_template_request_and_consumption.sql...
Copying gs://spls/gsp766/gke-qwiklab/test-quota.yaml...                         
Copying gs://spls/gsp766/gke-qwiklab/usage_metering_query_template.sql...       
/ [6/6 files][ 21.2 KiB/ 21.2 KiB] 100% Done                                    
Operation completed over 6 objects/21.2 KiB.                                     
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ cd ~/gke-qwiklab
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export ZONE=us-central1-f
gcloud config set compute/zone ${ZONE} && gcloud container clusters get-credentials multi-tenant-cluster
Updated property [compute/zone].
Fetching cluster endpoint and auth data.
kubeconfig entry generated for multi-tenant-cluster.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get namespace
NAME              STATUS   AGE
default           Active   4h35m
gmp-public        Active   4h34m
gmp-system        Active   4h34m
kube-node-lease   Active   4h35m
kube-public       Active   4h35m
kube-system       Active   4h35m
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl api-resources --namespaced=true
NAME                           SHORTNAMES   APIVERSION                       NAMESPACED   KIND
bindings                                    v1                               true         Binding
configmaps                     cm           v1                               true         ConfigMap
endpoints                      ep           v1                               true         Endpoints
events                         ev           v1                               true         Event
limitranges                    limits       v1                               true         LimitRange
persistentvolumeclaims         pvc          v1                               true         PersistentVolumeClaim
pods                           po           v1                               true         Pod
podtemplates                                v1                               true         PodTemplate
replicationcontrollers         rc           v1                               true         ReplicationController
resourcequotas                 quota        v1                               true         ResourceQuota
secrets                                     v1                               true         Secret
serviceaccounts                sa           v1                               true         ServiceAccount
services                       svc          v1                               true         Service
controllerrevisions                         apps/v1                          true         ControllerRevision
daemonsets                     ds           apps/v1                          true         DaemonSet
deployments                    deploy       apps/v1                          true         Deployment
replicasets                    rs           apps/v1                          true         ReplicaSet
statefulsets                   sts          apps/v1                          true         StatefulSet
localsubjectaccessreviews                   authorization.k8s.io/v1          true         LocalSubjectAccessReview
horizontalpodautoscalers       hpa          autoscaling/v2                   true         HorizontalPodAutoscaler
cronjobs                       cj           batch/v1                         true         CronJob
jobs                                        batch/v1                         true         Job
backendconfigs                 bc           cloud.google.com/v1              true         BackendConfig
leases                                      coordination.k8s.io/v1           true         Lease
endpointslices                              discovery.k8s.io/v1              true         EndpointSlice
events                         ev           events.k8s.io/v1                 true         Event
capacityrequests               capreq       internal.autoscaling.gke.io/v1   true         CapacityRequest
pods                                        metrics.k8s.io/v1beta1           true         PodMetrics
operatorconfigs                             monitoring.googleapis.com/v1     true         OperatorConfig
podmonitorings                              monitoring.googleapis.com/v1     true         PodMonitoring
rules                                       monitoring.googleapis.com/v1     true         Rules
frontendconfigs                             networking.gke.io/v1beta1        true         FrontendConfig
managedcertificates            mcrt         networking.gke.io/v1             true         ManagedCertificate
serviceattachments                          networking.gke.io/v1             true         ServiceAttachment
servicenetworkendpointgroups   svcneg       networking.gke.io/v1beta1        true         ServiceNetworkEndpointGroup
ingresses                      ing          networking.k8s.io/v1             true         Ingress
networkpolicies                netpol       networking.k8s.io/v1             true         NetworkPolicy
updateinfos                    updinf       nodemanagement.gke.io/v1alpha1   true         UpdateInfo
poddisruptionbudgets           pdb          policy/v1                        true         PodDisruptionBudget
rolebindings                                rbac.authorization.k8s.io/v1     true         RoleBinding
roles                                       rbac.authorization.k8s.io/v1     true         Role
volumesnapshots                vs           snapshot.storage.k8s.io/v1       true         VolumeSnapshot
csistoragecapacities                        storage.k8s.io/v1                true         CSIStorageCapacity
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get nodes
NAME                                                  STATUS   ROLES    AGE     VERSION
gke-multi-tenant-clu-multi-tenant-clu-adad92c0-5n9z   Ready    <none>   4h28m   v1.27.8-gke.1067004
gke-multi-tenant-clu-multi-tenant-clu-adad92c0-6hm6   Ready    <none>   4h28m   v1.27.8-gke.1067004
gke-multi-tenant-clu-multi-tenant-clu-adad92c0-cswg   Ready    <none>   4h28m   v1.27.8-gke.1067004
gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr   Ready    <none>   4h28m   v1.27.8-gke.1067004
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get services --namespace=kube-system
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default-http-backend   NodePort    10.13.233.197   <none>        80:31849/TCP    4h35m
kube-dns               ClusterIP   10.13.224.10    <none>        53/UDP,53/TCP   4h35m
metrics-server         ClusterIP   10.13.238.240   <none>        443/TCP         4h35m
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create namespace team-a && \
kubectl create namespace team-b
namespace/team-a created
namespace/team-b created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get ns
NAME              STATUS   AGE
default           Active   4h36m
gmp-public        Active   4h35m
gmp-system        Active   4h35m
kube-node-lease   Active   4h36m
kube-public       Active   4h36m
kube-system       Active   4h36m
team-a            Active   6s
team-b            Active   5s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl run app-server --image=centos --namespace=team-a -- sleep infinity && \
kubectl run app-server --image=centos --namespace=team-b -- sleep infinity
pod/app-server created
pod/app-server created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods
No resources found in default namespace.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods -n team-a
NAME         READY   STATUS    RESTARTS   AGE
app-server   1/1     Running   0          13s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods -n team-b
NAME         READY   STATUS    RESTARTS   AGE
app-server   1/1     Running   0          15s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods -A
NAMESPACE     NAME                                                             READY   STATUS    RESTARTS        AGE
gmp-system    alertmanager-0                                                   2/2     Running   0               4h33m
gmp-system    collector-bz6l5                                                  2/2     Running   0               4h29m
gmp-system    collector-nm98c                                                  2/2     Running   0               4h29m
gmp-system    collector-zlg8f                                                  2/2     Running   0               4h29m
gmp-system    collector-zwz55                                                  2/2     Running   0               4h29m
gmp-system    gmp-operator-569d6c8979-mv2th                                    1/1     Running   0               4h35m
gmp-system    rule-evaluator-5f9c59c975-5xhh5                                  2/2     Running   1 (4h29m ago)   4h29m
kube-system   event-exporter-gke-5b8bcb44f7-s8q2q                              2/2     Running   0               4h35m
kube-system   fluentbit-gke-4p5vp                                              2/2     Running   0               4h30m
kube-system   fluentbit-gke-7kktb                                              2/2     Running   0               4h29m
kube-system   fluentbit-gke-8j2lk                                              2/2     Running   0               4h29m
kube-system   fluentbit-gke-w9z28                                              2/2     Running   0               4h29m
kube-system   gke-metrics-agent-2rf7b                                          2/2     Running   0               4h29m
kube-system   gke-metrics-agent-8rppn                                          2/2     Running   0               4h29m
kube-system   gke-metrics-agent-p5j47                                          2/2     Running   0               4h30m
kube-system   gke-metrics-agent-wt4vs                                          2/2     Running   0               4h29m
kube-system   konnectivity-agent-84d9d5d9d7-546m9                              1/1     Running   0               4h29m
kube-system   konnectivity-agent-84d9d5d9d7-fgmgc                              1/1     Running   0               4h35m
kube-system   konnectivity-agent-84d9d5d9d7-kkftr                              1/1     Running   0               4h29m
kube-system   konnectivity-agent-84d9d5d9d7-mvk2q                              1/1     Running   0               4h29m
kube-system   konnectivity-agent-autoscaler-5d9dbcc6d8-7wlvv                   1/1     Running   0               4h35m
kube-system   kube-dns-6f9b8847ff-k8lzz                                        4/4     Running   0               4h35m
kube-system   kube-dns-6f9b8847ff-vj4hs                                        4/4     Running   0               4h29m
kube-system   kube-dns-autoscaler-84b8db4dc7-p4pmb                             1/1     Running   0               4h35m
kube-system   kube-proxy-gke-multi-tenant-clu-multi-tenant-clu-adad92c0-5n9z   1/1     Running   0               4h30m
kube-system   kube-proxy-gke-multi-tenant-clu-multi-tenant-clu-adad92c0-6hm6   1/1     Running   0               4h29m
kube-system   kube-proxy-gke-multi-tenant-clu-multi-tenant-clu-adad92c0-cswg   1/1     Running   0               4h29m
kube-system   kube-proxy-gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr   1/1     Running   0               4h29m
kube-system   l7-default-backend-cf7cdc6f6-pv799                               1/1     Running   0               4h35m
kube-system   metrics-server-v0.5.2-8fb865474-d555d                            2/2     Running   0               4h29m
kube-system   pdcsi-node-fxfr2                                                 2/2     Running   0               4h29m
kube-system   pdcsi-node-lwzht                                                 2/2     Running   0               4h29m
kube-system   pdcsi-node-ncxs7                                                 2/2     Running   0               4h29m
kube-system   pdcsi-node-xc4f7                                                 2/2     Running   0               4h30m
team-a        app-server                                                       1/1     Running   0               27s
team-b        app-server                                                       1/1     Running   0               25s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl describe pod app-server --namespace=team-a
Name:             app-server
Namespace:        team-a
Priority:         0
Service Account:  default
Node:             gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr/10.10.0.6
Start Time:       Wed, 10 Apr 2024 02:35:34 +0000
Labels:           run=app-server
Annotations:      <none>
Status:           Running
IP:               10.112.2.6
IPs:
  IP:  10.112.2.6
Containers:
  app-server:
    Container ID:  containerd://259f32fb26d0f9180b5fa752b561d26c2ba4217ec82723b512139473579af21f
    Image:         centos
    Image ID:      docker.io/library/centos@sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      infinity
    State:          Running
      Started:      Wed, 10 Apr 2024 02:35:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kcb48 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-kcb48:
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
  Normal  Scheduled  47s   default-scheduler  Successfully assigned team-a/app-server to gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr
  Normal  Pulling    46s   kubelet            Pulling image "centos"
  Normal  Pulled     40s   kubelet            Successfully pulled image "centos" in 6.163184669s (6.163203768s including waiting)
  Normal  Created    40s   kubelet            Created container app-server
  Normal  Started    40s   kubelet            Started container app-server
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl config set-context --current --namespace=team-a
Context "gke_qwiklabs-gcp-03-77265de0bb34_us-central1-f_multi-tenant-cluster" modified.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl describe pod app-server
Name:             app-server
Namespace:        team-a
Priority:         0
Service Account:  default
Node:             gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr/10.10.0.6
Start Time:       Wed, 10 Apr 2024 02:35:34 +0000
Labels:           run=app-server
Annotations:      <none>
Status:           Running
IP:               10.112.2.6
IPs:
  IP:  10.112.2.6
Containers:
  app-server:
    Container ID:  containerd://259f32fb26d0f9180b5fa752b561d26c2ba4217ec82723b512139473579af21f
    Image:         centos
    Image ID:      docker.io/library/centos@sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
    Port:          <none>
    Host Port:     <none>
    Args:
      sleep
      infinity
    State:          Running
      Started:      Wed, 10 Apr 2024 02:35:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kcb48 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-kcb48:
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
  Normal  Scheduled  74s   default-scheduler  Successfully assigned team-a/app-server to gke-multi-tenant-clu-multi-tenant-clu-adad92c0-g7pr
  Normal  Pulling    73s   kubelet            Pulling image "centos"
  Normal  Pulled     67s   kubelet            Successfully pulled image "centos" in 6.163184669s (6.163203768s including waiting)
  Normal  Created    67s   kubelet            Created container app-server
  Normal  Started    67s   kubelet            Started container app-server
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ gcloud projects add-iam-policy-binding ${GOOGLE_CLOUD_PROJECT} \
--member=serviceAccount:team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com  \
--role=roles/container.clusterViewer
Updated IAM policy for project [qwiklabs-gcp-03-77265de0bb34].
bindings:
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/appengine.appAdmin
- members:
  - serviceAccount:qwiklabs-gcp-03-77265de0bb34@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/bigquery.admin
- members:
  - serviceAccount:535722686914@cloudbuild.gserviceaccount.com
  role: roles/cloudbuild.builds.builder
- members:
  - serviceAccount:service-535722686914@gcp-sa-cloudbuild.iam.gserviceaccount.com
  role: roles/cloudbuild.serviceAgent
- members:
  - serviceAccount:service-535722686914@compute-system.iam.gserviceaccount.com
  role: roles/compute.serviceAgent
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/container.admin
- members:
  - serviceAccount:team-a-dev@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com
  role: roles/container.clusterViewer
- members:
  - serviceAccount:service-535722686914@container-engine-robot.iam.gserviceaccount.com
  role: roles/container.serviceAgent
- members:
  - serviceAccount:535722686914-compute@developer.gserviceaccount.com
  - serviceAccount:535722686914@cloudservices.gserviceaccount.com
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/editor
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/iam.securityAdmin
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/iam.serviceAccountKeyAdmin
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/iam.serviceAccountTokenCreator
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/monitoring.admin
- members:
  - serviceAccount:service-535722686914@gcp-sa-networkconnectivity.iam.gserviceaccount.com
  role: roles/networkconnectivity.serviceAgent
- members:
  - serviceAccount:admiral@qwiklabs-services-prod.iam.gserviceaccount.com
  - serviceAccount:qwiklabs-gcp-03-77265de0bb34@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com
  role: roles/owner
- members:
  - serviceAccount:qwiklabs-gcp-03-77265de0bb34@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com
  role: roles/storage.admin
- members:
  - user:student-01-db0d0c7211fb@qwiklabs.net
  role: roles/viewer
etag: BwYVtOxKS_Q=
version: 1
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create role pod-reader \
--resource=pods --verb=watch --verb=get --verb=list
role.rbac.authorization.k8s.io/pod-reader created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ cat developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: team-a
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods", "services", "serviceaccounts"]
  verbs: ["update", "create", "delete", "get", "watch", "list"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["update", "create", "delete", "get", "watch", "list"]
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create -f developer-role.yaml
role.rbac.authorization.k8s.io/developer created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get roles
NAME         CREATED AT
developer    2024-04-10T02:38:52Z
pod-reader   2024-04-10T02:38:15Z
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create rolebinding team-a-developers \
--role=developer --user=team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
rolebinding.rbac.authorization.k8s.io/team-a-developers created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get rolebinding
NAME                ROLE             AGE
team-a-developers   Role/developer   8s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ gcloud iam service-accounts keys create /tmp/key.json --iam-account team-a-dev@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com
created key [1669fd8dfa9ce2c2d9a27004c637cdaafd080190] of type [json] as [/tmp/key.json] for [team-a-dev@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com]
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export ZONE=us-central1-f
gcloud container clusters get-credentials multi-tenant-cluster --zone ${ZONE} --project ${GOOGLE_CLOUD_PROJECT}
Fetching cluster endpoint and auth data.
kubeconfig entry generated for multi-tenant-cluster.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create quota test-quota \
--hard=count/pods=2,count/services.loadbalancers=1 --namespace=team-a
resourcequota/test-quota created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get quota
NAME                  AGE     REQUEST                                                                                                                              LIMIT
gke-resource-quotas   4h38m   count/ingresses.extensions: 0/100, count/ingresses.networking.k8s.io: 0/100, count/jobs.batch: 0/5k, pods: 0/1500, services: 1/500   
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl run app-server-2 --image=centos --namespace=team-a -- sleep infinity
pod/app-server-2 created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods
No resources found in default namespace.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods -n team-a
NAME           READY   STATUS    RESTARTS   AGE
app-server     1/1     Running   0          6m31s
app-server-2   1/1     Running   0          13s
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl run app-server-3 --image=centos --namespace=team-a -- sleep infinity

Error from server (Forbidden): pods "app-server-3" is forbidden: exceeded quota: test-quota, requested: count/pods=1, used: count/pods=2, limited: count/pods=2
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl describe quota test-quota --namespace=team-a
Name:                         test-quota
Namespace:                    team-a
Resource                      Used  Hard
--------                      ----  ----
count/pods                    2     2
count/services.loadbalancers  0     1
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export KUBE_EDITOR="nano"
kubectl edit quota test-quota --namespace=team-a
resourcequota/test-quota edited
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl describe quota test-quota --namespace=team-a
Name:                         test-quota
Namespace:                    team-a
Resource                      Used  Hard
--------                      ----  ----
count/pods                    2     6
count/services.loadbalancers  0     1
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ cat cpu-mem-
cat: cpu-mem-: No such file or directory
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ cat cpu-mem-
cpu-mem-demo-pod.yaml  cpu-mem-quota.yaml     
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ cat cpu-mem-quota.yaml 
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cpu-mem-quota
  namespace: team-a
spec:
  hard:
    limits.cpu: "4"
    limits.memory: "12Gi"
    requests.cpu: "2"
    requests.memory: "8Gi"
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create -f cpu-mem-quota.yaml
resourcequota/cpu-mem-quota created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get quota
NAME                  AGE     REQUEST                                                                                                                              LIMIT
gke-resource-quotas   4h41m   count/ingresses.extensions: 0/100, count/ingresses.networking.k8s.io: 0/100, count/jobs.batch: 0/5k, pods: 0/1500, services: 1/500   
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl get quota -n team-a
NAME                  AGE     REQUEST                                                                                                                              LIMIT
cpu-mem-quota         24s     requests.cpu: 0/2, requests.memory: 0/8Gi                                                                                            limits.cpu: 0/4, limits.memory: 0/12Gi
gke-resource-quotas   9m33s   count/ingresses.extensions: 0/100, count/ingresses.networking.k8s.io: 0/100, count/jobs.batch: 0/5k, pods: 2/1500, services: 0/500   
test-quota            3m11s   count/pods: 2/6, count/services.loadbalancers: 0/1                                                                                   
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ cat cpu-mem-demo-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: cpu-mem-demo
spec:
  containers:
  - name: cpu-mem-demo-ctr
    image: nginx
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits: 
        cpu: "400m"
        memory: "512Mi"
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl create -f cpu-mem-demo-pod.yaml --namespace=team-a
pod/cpu-mem-demo created
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ kubectl describe quota cpu-mem-quota --namespace=team-a
Name:            cpu-mem-quota
Namespace:       team-a
Resource         Used   Hard
--------         ----   ----
limits.cpu       400m   4
limits.memory    512Mi  12Gi
requests.cpu     100m   2
requests.memory  128Mi  8Gi
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export ZONE=us-central1-f
gcloud container clusters \
  update multi-tenant-cluster --zone ${ZONE} \
  --resource-usage-bigquery-dataset cluster_dataset
Updating multi-tenant-cluster...working..                                                                                                                                          
Updating multi-tenant-cluster...done.                                                                                                                                              
Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-03-77265de0bb34/zones/us-central1-f/clusters/multi-tenant-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-f/multi-tenant-cluster?project=qwiklabs-gcp-03-77265de0bb34
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export GCP_BILLING_EXPORT_TABLE_FULL_PATH=${GOOGLE_CLOUD_PROJECT}.billing_dataset.gcp_billing_export_v1_xxxx
export USAGE_METERING_DATASET_ID=cluster_dataset
export COST_BREAKDOWN_TABLE_ID=usage_metering_cost_breakdown
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ export USAGE_METERING_QUERY_TEMPLATE=~/gke-qwiklab/usage_metering_query_template.sql
export USAGE_METERING_QUERY=cost_breakdown_query.sql
export USAGE_METERING_START_DATE=2020-10-26
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ sed \
-e "s/\${fullGCPBillingExportTableID}/$GCP_BILLING_EXPORT_TABLE_FULL_PATH/" \
-e "s/\${projectID}/$GOOGLE_CLOUD_PROJECT/" \
-e "s/\${datasetID}/$USAGE_METERING_DATASET_ID/" \
-e "s/\${startDate}/$USAGE_METERING_START_DATE/" \
"$USAGE_METERING_QUERY_TEMPLATE" \
> "$USAGE_METERING_QUERY"
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ bq query \
--project_id=$GOOGLE_CLOUD_PROJECT \
--use_legacy_sql=false \
--destination_table=$USAGE_METERING_DATASET_ID.$COST_BREAKDOWN_TABLE_ID \
--schedule='every 24 hours' \
--display_name="GKE Usage Metering Cost Breakdown Scheduled Query" \
--replace=true \
"$(cat $USAGE_METERING_QUERY)"
/usr/lib/google-cloud-sdk/platform/bq/third_party/requests/__init__.py:103: RequestsDependencyWarning: urllib3 (2.0.7) or chardet (None)/charset_normalizer (2.0.7) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({})/charset_normalizer ({}) doesn't match a supported "

https://www.gstatic.com/bigquerydatatransfer/oauthz/auth?client_id=433065040935-hav5fqnc9p9cht3rqneus9115ias2kn1.apps.googleusercontent.com&scope=https://www.googleapis.com/auth/bigquery%20https://www.googleapis.com/auth/drive&redirect_uri=urn:ietf:wg:oauth:2.0:oob&response_type=version_info
Please copy and paste the above URL into your web browser and follow the instructions to retrieve a version_info.
Enter your version_info here: CmxfU1ZJX0VOVzk0LWpUdG9VREdCQWlQMDFCUlVSSVpsOXhiR1IwWVdGMmRUZHRZbWxxUVY4eGFqaDVjWFpYVkd0MGQwRTBkSEZpTkVSbk1IQnBWekJpT0d0ck5GRTBWemRHVGw5VVZHSm9WUV8
Transfer configuration 'projects/535722686914/locations/us/transferConfigs/66276a77-0000-2f0e-95ea-582429aa9754' successfully created.
student_01_db0d0c7211fb@cloudshell:~/gke-qwiklab (qwiklabs-gcp-03-77265de0bb34)$ 
```



```sh
elcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-77265de0bb34.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ gcloud auth activate-service-account  --key-file=/tmp/key.json
Activated service account credentials for: [team-a-dev@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com]
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ export ZONE=us-central1-f
gcloud container clusters get-credentials multi-tenant-cluster --zone ${ZONE} --project ${GOOGLE_CLOUD_PROJECT}
Fetching cluster endpoint and auth data.
kubeconfig entry generated for multi-tenant-cluster.
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods --namespace=team-a
NAME         READY   STATUS    RESTARTS   AGE
app-server   1/1     Running   0          4m59s
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ kubectl get pods --namespace=team-b
Error from server (Forbidden): pods is forbidden: User "team-a-dev@qwiklabs-gcp-03-77265de0bb34.iam.gserviceaccount.com" cannot list resource "pods" in API group "" in the namespace "team-b": requires one of ["container.pods.list"] permission(s).
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ export GCP_BILLING_EXPORT_TABLE_FULL_PATH=${GOOGLE_CLOUD_PROJECT}.billing_dataset.gcp_billing_export_v1_xxxx
export USAGE_METERING_DATASET_ID=cluster_dataset
export COST_BREAKDOWN_TABLE_ID=usage_metering_cost_breakdown
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ export USAGE_METERING_QUERY_TEMPLATE=~/gke-qwiklab/usage_metering_query_template.sql
export USAGE_METERING_QUERY=cost_breakdown_query.sql
export USAGE_METERING_START_DATE=2020-10-26
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ sed \
-e "s/\${fullGCPBillingExportTableID}/$GCP_BILLING_EXPORT_TABLE_FULL_PATH/" \
-e "s/\${projectID}/$GOOGLE_CLOUD_PROJECT/" \
-e "s/\${datasetID}/$USAGE_METERING_DATASET_ID/" \
-e "s/\${startDate}/$USAGE_METERING_START_DATE/" \
"$USAGE_METERING_QUERY_TEMPLATE" \
> "$USAGE_METERING_QUERY"
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ bq query \
--project_id=$GOOGLE_CLOUD_PROJECT \
--use_legacy_sql=false \
--destination_table=$USAGE_METERING_DATASET_ID.$COST_BREAKDOWN_TABLE_ID \
--schedule='every 24 hours' \
--display_name="GKE Usage Metering Cost Breakdown Scheduled Query" \
--replace=true \
"$(cat $USAGE_METERING_QUERY)"
/usr/lib/google-cloud-sdk/platform/bq/third_party/requests/__init__.py:103: RequestsDependencyWarning: urllib3 (2.0.7) or chardet (None)/charset_normalizer (2.0.7) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({})/charset_normalizer ({}) doesn't match a supported "
BQ CLI will soon require all users to log in using `gcloud auth login`. `bq
init` will no longer handle authentication after January 1, 2024.

Welcome to BigQuery! This script will walk you through the 
process of initializing your .bigqueryrc configuration file.

First, we need to set up your credentials if they do not 
already exist.

Setting project_id qwiklabs-gcp-03-77265de0bb34 as the default.

BigQuery configuration complete! Type "bq" to get started.

BigQuery error in query operation: Scheduled queries are not enabled on this project. Please enable them at https://console.cloud.google.com/bigquery/scheduled-queries
student_01_db0d0c7211fb@cloudshell:~ (qwiklabs-gcp-03-77265de0bb34)$ 
```

