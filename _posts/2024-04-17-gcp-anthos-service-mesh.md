---
layout: single
title:  "GCP Anthos Service Mesh"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
## Anthos Service Mesh 

To provide resiliency, you must be able to recover from infrastructure  failure such as losing a cluster where a service might be running. To do this, You can run a service in multiple clusters with the so-called *distributed services*. A distributed service is a set of Kubernetes services that acts as a  single logical service. Distributed services are more resilient than  Kubernetes services because they run on multiple Kubernetes clusters in  the same namespace.

A distributed service provides multi-regional availability and  remains up even if one or more GKE clusters are down, but only if the  healthy clusters can serve the desired load. Running distributed  services makes cluster lifecycle management easier because you can bring clusters down for maintenance or upgrades while other clusters service  traffic. In order to create a distributed service, service mesh  functionality provided by Anthos Service Mesh is used to link services  running on multiple clusters together to act as a single logical  service.

Two GKE clusters called `gke-central` and `gke-west` have been provisioned in `us-central1` and `us-west2`. Anthos Service Mesh has been configured across these clusters to  provide cross-cluster service discoverability and secure routing so that a microservice pod running on `gke-central` can seamlessly communicate with a pod on `gke-west`. Additionally, the Bank of Anthos application has been deployed across these two clusters as shown in the following diagram.



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-4d63da4fb9a6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ gcloud container clusters list
NAME: gke-central
LOCATION: us-central1-c
MASTER_VERSION: 1.28.7-gke.1026000
MASTER_IP: 104.197.22.203
MACHINE_TYPE: e2-standard-4
NODE_VERSION: 1.28.7-gke.1026000
NUM_NODES: 2
STATUS: RUNNING

NAME: gke-west
LOCATION: us-west2-c
MASTER_VERSION: 1.27.8-gke.1067004
MASTER_IP: 35.236.64.223
MACHINE_TYPE: e2-standard-4
NODE_VERSION: 1.27.8-gke.1067004
NUM_NODES: 2
STATUS: RUNNING
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get pods -A
NAMESPACE        NAME                                                      READY   STATUS    RESTARTS        AGE
asm-gateways     istio-ingressgateway-59c87f6556-smx2w                     1/1     Running   0               4h3m
asm-gateways     istio-ingressgateway-59c87f6556-szkj6                     1/1     Running   0               4h3m
asm-gateways     istio-ingressgateway-59c87f6556-wk5hs                     1/1     Running   0               4h3m
asm-system       canonical-service-controller-manager-74c6dc6698-mjmmz     2/2     Running   0               4h5m
bank-of-anthos   balancereader-79ccbc7699-hlvl7                            2/2     Running   0               4h3m
bank-of-anthos   contacts-79f89bb6fd-h75nh                                 2/2     Running   0               4h3m
bank-of-anthos   frontend-78bbd8db59-5982n                                 2/2     Running   0               4h3m
bank-of-anthos   ledgerwriter-58d4579664-drpfg                             2/2     Running   0               4h3m
bank-of-anthos   loadgenerator-7bd8fdc955-zfmh7                            2/2     Running   0               4h3m
bank-of-anthos   transactionhistory-c9879784d-g6tdl                        2/2     Running   0               4h3m
bank-of-anthos   userservice-9757b7456-kk574                               2/2     Running   0               4h3m
gmp-system       alertmanager-0                                            2/2     Running   0               4h22m
gmp-system       collector-868lt                                           2/2     Running   0               4h17m
gmp-system       collector-t5trf                                           2/2     Running   0               4h17m
gmp-system       gmp-operator-6d895cd767-9j9gx                             1/1     Running   0               4h22m
gmp-system       rule-evaluator-6b9c45d5b8-pvlbg                           2/2     Running   1 (4h16m ago)   4h22m
istio-system     istiod-asm-1178-20-5b945c7c4b-hczf6                       1/1     Running   0               4h5m
istio-system     istiod-asm-1178-20-5b945c7c4b-xnrct                       1/1     Running   0               4h5m
kube-system      event-exporter-gke-7d996c57bf-ghvtf                       2/2     Running   0               4h22m
kube-system      fluentbit-gke-g7rbp                                       2/2     Running   0               4h17m
kube-system      fluentbit-gke-nptrx                                       2/2     Running   0               4h17m
kube-system      gke-metadata-server-r948n                                 1/1     Running   0               4h17m
kube-system      gke-metadata-server-t27fg                                 1/1     Running   0               4h17m
kube-system      gke-metrics-agent-5fqfr                                   2/2     Running   0               4h17m
kube-system      gke-metrics-agent-srmkx                                   2/2     Running   0               4h17m
kube-system      konnectivity-agent-575f97f5ff-28cf5                       2/2     Running   0               4h16m
kube-system      konnectivity-agent-575f97f5ff-dmt97                       2/2     Running   0               4h22m
kube-system      konnectivity-agent-autoscaler-5847cf65c7-q2flb            1/1     Running   0               4h22m
kube-system      kube-dns-6f955b858b-w2dgs                                 4/4     Running   0               4h16m
kube-system      kube-dns-6f955b858b-zzzb7                                 4/4     Running   0               4h22m
kube-system      kube-dns-autoscaler-755c7dfdf5-2fgxg                      1/1     Running   0               4h22m
kube-system      kube-proxy-gke-gke-central-gke-central-np-fceb55e8-f1xz   1/1     Running   0               4h17m
kube-system      kube-proxy-gke-gke-central-gke-central-np-fceb55e8-rjg4   1/1     Running   0               4h17m
kube-system      l7-default-backend-6779bb6c8d-dj42v                       1/1     Running   0               4h22m
kube-system      metrics-server-v0.6.3-764c8d87d9-zhxx7                    2/2     Running   0               4h16m
kube-system      netd-7lwsz                                                2/2     Running   0               4h17m
kube-system      netd-n7m9t                                                2/2     Running   0               4h17m
kube-system      pdcsi-node-9fkvb                                          2/2     Running   0               4h17m
kube-system      pdcsi-node-r6xqk                                          2/2     Running   0               4h17m
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```



```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ gcloud container clusters get-credentials gke-central --region us-central1-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-central.
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
gke-gke-central-gke-central-np-fceb55e8-f1xz   Ready    <none>   4h14m   v1.28.7-gke.1026000
gke-gke-central-gke-central-np-fceb55e8-rjg4   Ready    <none>   4h14m   v1.28.7-gke.1026000
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```



```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ gcloud container clusters get-credentials gke-west --region us-west2-c
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke-west.
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get nodes
NAME                                     STATUS   ROLES    AGE     VERSION
gke-gke-west-gke-west-np-f258e1af-q8ld   Ready    <none>   4h15m   v1.27.8-gke.1067004
gke-gke-west-gke-west-np-f258e1af-qq2m   Ready    <none>   4h15m   v1.27.8-gke.1067004
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get pods -A
NAMESPACE        NAME                                                    READY   STATUS    RESTARTS        AGE
asm-gateways     istio-ingressgateway-f655c454-bkmj5                     1/1     Running   0               4h2m
asm-gateways     istio-ingressgateway-f655c454-cpfk8                     1/1     Running   0               4h2m
asm-gateways     istio-ingressgateway-f655c454-l9wtr                     1/1     Running   0               4h2m
asm-system       canonical-service-controller-manager-748cb7dd6b-zzjzl   2/2     Running   0               4h9m
bank-of-anthos   accounts-db-0                                           2/2     Running   0               4h2m
bank-of-anthos   balancereader-55c8d45f57-2rzsl                          2/2     Running   0               4h2m
bank-of-anthos   contacts-567df45b9b-nsgxm                               2/2     Running   0               4h2m
bank-of-anthos   frontend-645c7d6bdf-9542f                               2/2     Running   0               4h2m
bank-of-anthos   ledger-db-0                                             2/2     Running   0               4h2m
bank-of-anthos   ledgerwriter-757d786f4-r4qb5                            2/2     Running   0               4h2m
bank-of-anthos   loadgenerator-695d7d9469-6456s                          2/2     Running   0               4h2m
bank-of-anthos   transactionhistory-768d477874-t5t7d                     2/2     Running   0               4h2m
bank-of-anthos   userservice-79654db886-mstvz                            2/2     Running   0               4h2m
gmp-system       alertmanager-0                                          2/2     Running   0               4h19m
gmp-system       collector-chrzw                                         2/2     Running   0               4h15m
gmp-system       collector-p2dcd                                         2/2     Running   0               4h15m
gmp-system       gmp-operator-b964d6967-5q96s                            1/1     Running   0               4h21m
gmp-system       rule-evaluator-57c86c9bb7-97fxm                         2/2     Running   1 (4h16m ago)   4h16m
istio-system     istiod-asm-1178-20-55bbfc664c-9ww5j                     1/1     Running   0               4h9m
istio-system     istiod-asm-1178-20-55bbfc664c-wsvnc                     1/1     Running   0               4h9m
kube-system      event-exporter-gke-5b8bcb44f7-cpnwg                     2/2     Running   0               4h21m
kube-system      fluentbit-gke-8lc88                                     2/2     Running   0               4h16m
kube-system      fluentbit-gke-zd7bb                                     2/2     Running   0               4h16m
kube-system      gke-metadata-server-vtblh                               1/1     Running   0               4h16m
kube-system      gke-metadata-server-z2849                               1/1     Running   0               4h16m
kube-system      gke-metrics-agent-gjpcr                                 2/2     Running   0               4h16m
kube-system      gke-metrics-agent-hdgb5                                 2/2     Running   0               4h16m
kube-system      konnectivity-agent-5ddbcdbc97-pl6ct                     1/1     Running   0               4h16m
kube-system      konnectivity-agent-5ddbcdbc97-tvmch                     1/1     Running   0               4h21m
kube-system      konnectivity-agent-autoscaler-5d9dbcc6d8-9swqj          1/1     Running   0               4h21m
kube-system      kube-dns-6f9b8847ff-7pcqb                               4/4     Running   0               4h21m
kube-system      kube-dns-6f9b8847ff-ncgb6                               4/4     Running   0               4h16m
kube-system      kube-dns-autoscaler-84b8db4dc7-zs9n7                    1/1     Running   0               4h21m
kube-system      kube-proxy-gke-gke-west-gke-west-np-f258e1af-q8ld       1/1     Running   0               4h16m
kube-system      kube-proxy-gke-gke-west-gke-west-np-f258e1af-qq2m       1/1     Running   0               4h16m
kube-system      l7-default-backend-cf7cdc6f6-r6hk5                      1/1     Running   0               4h21m
kube-system      metrics-server-v0.5.2-8fb865474-j6xc5                   2/2     Running   0               4h16m
kube-system      netd-695lh                                              1/1     Running   0               4h16m
kube-system      netd-mbfq2                                              1/1     Running   0               4h16m
kube-system      pdcsi-node-sr9vg                                        2/2     Running   0               4h16m
kube-system      pdcsi-node-zmqwg                                        2/2     Running   0               4h16m
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

Verify that every microservice is deployed on both GKE clusters except from the two databases, **accounts-db** and **ledger-db**, which are only running in **gke-west**.



 Verify that every microservice is deployed on both GKE clusters, including the two databases, **accounts-db** and **ledger-db**, which are needed for traffic routing.
 **istio-ingressgateway** is also in the list and should be available in both clusters.

```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get svc -A
NAMESPACE        NAME                                                   TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                                      AGE
asm-gateways     istio-ingressgateway                                   LoadBalancer   10.9.85.191   34.170.58.250   15021:31178/TCP,80:32391/TCP,443:30715/TCP   4h4m
asm-system       canonical-service-controller-manager-metrics-service   ClusterIP      10.9.83.176   <none>          8443/TCP                                     4h6m
bank-of-anthos   accounts-db                                            ClusterIP      10.9.82.204   <none>          5432/TCP                                     4h4m
bank-of-anthos   balancereader                                          ClusterIP      10.9.81.28    <none>          8080/TCP                                     4h4m
bank-of-anthos   contacts                                               ClusterIP      10.9.86.193   <none>          8080/TCP                                     4h4m
bank-of-anthos   frontend                                               LoadBalancer   10.9.92.72    34.71.170.213   80:32139/TCP                                 4h4m
bank-of-anthos   ledger-db                                              ClusterIP      10.9.86.157   <none>          5432/TCP                                     4h4m
bank-of-anthos   ledgerwriter                                           ClusterIP      10.9.90.250   <none>          8080/TCP                                     4h4m
bank-of-anthos   transactionhistory                                     ClusterIP      10.9.90.203   <none>          8080/TCP                                     4h4m
bank-of-anthos   userservice                                            ClusterIP      10.9.85.67    <none>          8080/TCP                                     4h4m
default          kubernetes                                             ClusterIP      10.9.80.1     <none>          443/TCP                                      4h25m
gmp-system       alertmanager                                           ClusterIP      None          <none>          9093/TCP                                     4h25m
gmp-system       gmp-operator                                           ClusterIP      10.9.93.175   <none>          8443/TCP,443/TCP                             4h25m
istio-system     istiod-asm-1178-20                                     ClusterIP      10.9.84.52    <none>          15010/TCP,15012/TCP,443/TCP,15014/TCP        4h7m
kube-system      default-http-backend                                   NodePort       10.9.86.7     <none>          80:30638/TCP                                 4h25m
kube-system      kube-dns                                               ClusterIP      10.9.80.10    <none>          53/UDP,53/TCP                                4h25m
kube-system      metrics-server                                         ClusterIP      10.9.94.40    <none>          443/TCP                                      4h24m
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get svc -A
NAMESPACE        NAME                                                   TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                                      AGE
asm-gateways     istio-ingressgateway                                   LoadBalancer   10.45.251.169   35.236.59.233    15021:30447/TCP,80:32507/TCP,443:31031/TCP   4h6m
asm-system       canonical-service-controller-manager-metrics-service   ClusterIP      10.45.243.103   <none>           8443/TCP                                     4h12m
bank-of-anthos   accounts-db                                            ClusterIP      10.45.248.160   <none>           5432/TCP                                     4h5m
bank-of-anthos   balancereader                                          ClusterIP      10.45.246.188   <none>           8080/TCP                                     4h5m
bank-of-anthos   contacts                                               ClusterIP      10.45.254.189   <none>           8080/TCP                                     4h5m
bank-of-anthos   frontend                                               LoadBalancer   10.45.249.197   35.235.121.171   80:32632/TCP                                 4h5m
bank-of-anthos   ledger-db                                              ClusterIP      10.45.248.145   <none>           5432/TCP                                     4h5m
bank-of-anthos   ledgerwriter                                           ClusterIP      10.45.244.67    <none>           8080/TCP                                     4h5m
bank-of-anthos   transactionhistory                                     ClusterIP      10.45.246.204   <none>           8080/TCP                                     4h5m
bank-of-anthos   userservice                                            ClusterIP      10.45.241.255   <none>           8080/TCP                                     4h5m
default          kubernetes                                             ClusterIP      10.45.240.1     <none>           443/TCP                                      4h26m
gmp-system       alertmanager                                           ClusterIP      None            <none>           9093/TCP                                     4h25m
gmp-system       gmp-operator                                           ClusterIP      10.45.247.50    <none>           8443/TCP,443/TCP                             4h26m
istio-system     istiod-asm-1178-20                                     ClusterIP      10.45.254.175   <none>           15010/TCP,15012/TCP,443/TCP,15014/TCP        4h12m
kube-system      default-http-backend                                   NodePort       10.45.242.199   <none>           80:30998/TCP                                 4h26m
kube-system      kube-dns                                               ClusterIP      10.45.240.10    <none>           53/UDP,53/TCP                                4h26m
kube-system      metrics-server                                         ClusterIP      10.45.244.213   <none>           443/TCP                                      4h25m
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

This component provides a public IP address, creates a Cloud Load Balancer, and enables access to the application from outside the cluster.

To open websites, click on the IP addresses for each of the instances of **istio-ingressgateway**.
 This will open the sign-in page for each website.

1. To open websites, click on the IP addresses for each of the instances of **istio-ingressgateway**.
    This will open the sign-in page for each website.
2. Click **Sign In** to sign in with an existing test user.
3. Deposit funds or send a payment, which will create a new transaction on the shared database.
4. Refresh the pages and confirm that both **Transaction History** and **Current Balance** are the same across clusters.

> **Note:** Notice that even though all services are replicated across clusters, they are both using the same database because the **ledger-db** is only deployed on one cluster. Anthos Service Mesh routes requests to the available pods regardless of the cluster you are ingressing from. This is called east-west routing

## Force cross-cluster traffic routing

In this task, you delete all frontend pods in one cluster to force the traffic to be routed to the other cluster from the ingress gateway. That simulates a scenario where an application might be unavailable due to an upgrade or a spike in traffic.

1. In the Google Cloud Console, return to **Kubernetes Engine > Workloads**.
2. To open the dashboard, click on the **frontend** deployment in the **gke-central** cluster.
3. On the **Actions** dropdown, select **Scale** **>** **Edit replicas**.
4. Enter **0** replicas, and click **Scale**.
5. Return to the **istio-ingressgateways** IP addresses that you opened before.
    The application should continue to work.

```sh
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl scale deployment frontend --replicas 0 -n bank-of-anthos
deployment.apps/frontend scaled
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get deploy frontend -n bank-of-anthos
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   0/0     0            0           4h16m
student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
```

Congratulations! You successfully tested your application resiliency. You removed all pods from one cluster, and the application continued to work without experiencing any downtime.

1. Return to your **frontend** deployment and scale it back to 1 replica.

   ```sh
   student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl scale deployment frontend --replicas 1 -n bank-of-anthos
   deployment.apps/frontend scaled
   
   student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ kubectl get deploy frontend -n bank-of-anthos
   NAME       READY   UP-TO-DATE   AVAILABLE   AGE
   frontend   1/1     1            1           4h17m
   student_00_f8c6491462da@cloudshell:~ (qwiklabs-gcp-04-4d63da4fb9a6)$ 
   ```

   SLO Preview

```json
{
  "displayName": "99.5% - Latency - Calendar day",
  "goal": 0.995,
  "calendarPeriod": "DAY",
  "serviceLevelIndicator": {
    "basicSli": {
      "latency": {
        "threshold": "0.35s"
      }
    }
  }
}
```

Notice that the created SLO is already out-of-budget. You could also set up an alert so that when this happens you can investigate any possible problems. This helps you understand and tune your services to provide a better user experience.

Congratulations!You learned to view the topology of your service mesh, viewed service metrics, and set up SLOs.



From the **frontend** service ASM Dashboard, open the **Security (BETA)** tab. The following diagram is displayed:

Notice that all service to service communication has a green lock. That's because all the communication is encrypted over mutual TLS. Also, notice that an unknown source has an open red lock. That means that an unauthenticated agent is accessing the frontend service that is communicating over plain text. This unauthenticated agent is the browser.

**Note:** Anthos Service Mesh automatically enables mutual TLS (mTLS) across services, so that all communications are authenticated and encrypted.

The table at the bottom of the page also shows the HTTP operations that clients are allowed to perform on the frontend service. In this case, the ingressgateway and the loadgenerator are allowed to perform HTTP GET and HTTP Post requests, but unauthenticated actors are only able to perform HTTP GET requests.

Anthos Service Mesh provides L7 networking policies that can be used in combination with Kubernetes L4 networking polices to provide in-depth security.

In this lab, you explored Anthos clusters in the Google Cloud Console  and learned about the benefits of using Anthos Service Mesh to create  distributed services. You observed distributed services, viewed metrics, set up SLOs, investigated network topology, and verified security and  encryption configuration using the Anthos Service Mesh Dashboards.
