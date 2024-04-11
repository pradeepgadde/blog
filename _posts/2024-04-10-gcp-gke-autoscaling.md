---
layout: single
title:  "GCP GKE Auto Scaling"
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
## GCP GKE Auto Scaling


- Decrease number of replicas for a Deployment with Horizontal Pod Autoscaler
- Decrease CPU request of a Deployment with Vertical Pod Autoscaler
- Decrease number of nodes used in cluster with Cluster Autoscaler
- Automatically create an optimized node pool for workload with Node Auto Provisioning
- Test the autoscaling behavior against a spike in demand
- Overprovision your cluster with Pause Pods

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-696b4c0e98b8.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud config set compute/zone us-west1-a
Updated property [compute/zone].
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud container clusters create scaling-demo --num-nodes=3 --enable-vertical-pod-autoscaling
Default change: VPC-native is the default mode during cluster creation for versions greater than 1.21.0-gke.1500. To create advanced routes based clusters, please pass the `--no-enable-ip-alias` flag
Note: Your Pod address range (`--cluster-ipv4-cidr`) can accommodate at most 1008 node(s).
Creating cluster scaling-demo in us-west1-a... Cluster is being health-checked (master is healthy)...done.                                    
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-696b4c0e98b8/zones/us-west1-a/clusters/scaling-demo].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/scaling-demo?project=qwiklabs-gcp-01-696b4c0e98b8
kubeconfig entry generated for scaling-demo.
NAME: scaling-demo
LOCATION: us-west1-a
MASTER_VERSION: 1.27.8-gke.1067004
MASTER_IP: 35.203.170.65
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.27.8-gke.1067004
NUM_NODES: 3
STATUS: RUNNING
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



```yaml
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ cat php-apache.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 3
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl apply -f php-apache.yaml
deployment.apps/php-apache created
service/php-apache created
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

## Scale pods with Horizontal Pod Autoscaling

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   3/3     3            3           72s
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/php-apache autoscaled
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

This `autoscale` command will configure a Horizontal Pod Autoscaler that will maintain between 1 and 10 replicas of the pods controlled by the `php-apache` deployment. The `cpu-percent` flag specifies 50% as the target average CPU utilization of requested  CPU over all the pods. HPA will adjust the number of replicas (via the  deployment) to maintain an average CPU utilization of 50% across all  pods.



```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   1%/50%    1         10        3          114s
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$
```

Under the **Targets** column you see **1%/50%**.

This means that the pods within your deployment are currently at 1%  of their target average CPU utilization. This is to be expected as the `php-apache` app is receiving no traffic right now.

Also, take note of the **Replicas** column. To start with, the value will be **3**. This number will be changed by the autoscaler as the number of required pods changes.

In this case, the autoscaler will scale the deployment down to the minimum number of pods indicated when you run the `autoscale` command. Horizontal Pod Autoscaling takes 5-10 minutes and will require shutting down or starting new pods depending on which way it's scaling.



## Scale size of pods with Vertical Pod Autoscaling

Vertical Pod Autoscaling frees you from having to think about what  values to specify for a container's CPU and memory requests. The  autoscaler can recommend values for CPU and memory requests and limits,  or it can automatically update the values.

> Vertical Pod Autoscaling should not be used alongside Horizontal Pod  Autoscaling on CPU or memory. Both autoscalers will try to respond to  changes in demand on the same metrics and conflict. However, VPA on CPU  or memory can be used with HPA on custom metrics to avoid overlap.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud container clusters describe scaling-demo | grep ^verticalPodAutoscaling -A 1
verticalPodAutoscaling:
  enabled: true
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

> Vertical Pod Autoscaling can be enabled on an existing cluster with `gcloud container clusters update scaling-demo --enable-vertical-pod-autoscaling`



```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/hello-server created
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment hello-server
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-server   1/1     1            1           9s
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

Assign a CPU resource request of 450m to the deployment:

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl set resources deployment hello-server --requests=cpu=450m
deployment.apps/hello-server resource requirements updated
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"
Containers:
  hello-app:
    Container ID:   containerd://b1acb5451e2d957019664b6025aff416b3a7d7e4dc6d28ff6d5cb5a1bbeb38d6
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       gcr.io/google-samples/hello-app@sha256:b1455e1c4fcc5ea1023c9e3b584cd84b64eb920e332feff690a2829696e379e7
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 11 Apr 2024 04:09:17 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        450m
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-g4v99 (ro)
Conditions:
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

create a manifest for you **Vertical Pod Autoscaler**:

```yaml
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ cat hello-vpa.yaml 
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: hello-server-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       hello-server
  updatePolicy:
    updateMode: "Off"
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

The above generates a manifest for a Vertical Pod Autoscaler targeting the `hello-server` deployment with an Update Policy of **Off**. A VPA can have one of three different update policies which can be useful depending on your application:

- **Off:** this policy means VPA will generate recommendations based on historical data which you can manually apply.
- **Initial**: VPA recommendations will be used to create new pods once and then won't change the pod size after.
- **Auto:** pods will regularly be deleted and recreated to match the size of the recommendations.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl apply -f hello-vpa.yaml
verticalpodautoscaler.autoscaling.k8s.io/hello-server-vpa created
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get vpa
NAME               MODE   CPU   MEM   PROVIDED   AGE
hello-server-vpa   Off                           6s
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl describe vpa hello-server-vpa
Name:         hello-server-vpa
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  autoscaling.k8s.io/v1
Kind:         VerticalPodAutoscaler
Metadata:
  Creation Timestamp:  2024-04-11T04:11:32Z
  Generation:          1
  Resource Version:    7499
  UID:                 56e63a5e-f38c-477b-b48a-002fa7504cea
Spec:
  Target Ref:
    API Version:  apps/v1
    Kind:         Deployment
    Name:         hello-server
  Update Policy:
    Update Mode:  Off
Events:           <none>
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

After some time

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl describe vpa hello-server-vpa
Name:         hello-server-vpa
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  autoscaling.k8s.io/v1
Kind:         VerticalPodAutoscaler
Metadata:
  Creation Timestamp:  2024-04-11T04:11:32Z
  Generation:          2
  Resource Version:    7862
  UID:                 56e63a5e-f38c-477b-b48a-002fa7504cea
Spec:
  Target Ref:
    API Version:  apps/v1
    Kind:         Deployment
    Name:         hello-server
  Update Policy:
    Update Mode:  Off
Status:
  Conditions:
    Last Transition Time:  2024-04-11T04:12:12Z
    Message:               Some containers have a small number of samples
    Reason:                hello-app
    Status:                True
    Type:                  LowConfidence
    Last Transition Time:  2024-04-11T04:12:12Z
    Status:                True
    Type:                  RecommendationProvided
  Recommendation:
    Container Recommendations:
      Container Name:  hello-app
      Lower Bound:
        Cpu:     1m
        Memory:  2097152
      Target:
        Cpu:     2m
        Memory:  3145728
      Uncapped Target:
        Cpu:     2m
        Memory:  3145728
      Upper Bound:
        Cpu:     1180m
        Memory:  1595932672
Events:          <none>
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

- **Lower Bound**: this is the lower bound number VPA looks  at for triggering a resize. If your pod utilization goes below this, VPA will delete the pod and scale it down.
- **Target:** this is the value VPA will use when resizing the pod.
- **Uncapped Target**: if no minimum or maximum capacity is assigned to the VPA, this will be the target utilization for VPA.
- **Upper Bound**: this is the upper bound number VPA looks  at for triggering a resize. If your pod utilization goes above this, VPA will delete the pod and scale it up.

You'll notice VPA is recommending the CPU request for this container be set to **25m** instead of the previous **100m** as well as giving you a suggested number for how much memory should be  requested. At this point, these recommendations can be manually applied  to the `hello-server` deployment.



> Vertical Pod Autoscaling bases its recommendations on historical data  from the container. In practice, it's recommended to wait at least 24  hours to collect recommendation data before applying any changes.

Update the manifest to set the policy to **Auto** and apply the configuration:

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ sed -i 's/Off/Auto/g' hello-vpa.yaml
kubectl apply -f hello-vpa.yaml
verticalpodautoscaler.autoscaling.k8s.io/hello-server-vpa configured
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$
```

```yaml
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ cat hello-vpa.yaml 
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: hello-server-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       hello-server
  updatePolicy:
    updateMode: "Auto"
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

In order to resize a pod, Vertical Pod Autoscaler will need to delete  that pod and recreate it with the new size. By default, to avoid  downtime, VPA will not delete and resize the last active pod. Because of this, you will need at least 2 replicas to see VPA make any changes.



Scale `hello-server` deployment to 2 replicas:

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl scale deployment hello-server --replicas=2
deployment.apps/hello-server scaled
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-server   2/2     2            2           11m
php-apache     1/1     1            1           17m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get vpa
NAME               MODE   CPU   MEM       PROVIDED   AGE
hello-server-vpa   Auto   2m    3145728   True       8m17s
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get pods -w
NAME                            READY   STATUS    RESTARTS   AGE
hello-server-7c7f99c596-n9mj5   1/1     Running   0          11m
hello-server-7c7f99c596-r7wvg   1/1     Running   0          70s
php-apache-69f9bc5fd5-2d6h6     1/1     Running   0          18m
hello-server-7c7f99c596-n9mj5   1/1     Running   0          11m
hello-server-7c7f99c596-n9mj5   1/1     Terminating   0          11m
hello-server-7c7f99c596-n9mj5   1/1     Terminating   0          11m
hello-server-7c7f99c596-cvkmx   0/1     Pending       0          0s
hello-server-7c7f99c596-cvkmx   0/1     Pending       0          0s
hello-server-7c7f99c596-cvkmx   0/1     ContainerCreating   0          0s
hello-server-7c7f99c596-n9mj5   0/1     Terminating         0          11m
hello-server-7c7f99c596-n9mj5   0/1     Terminating         0          11m
hello-server-7c7f99c596-n9mj5   0/1     Terminating         0          11m
hello-server-7c7f99c596-n9mj5   0/1     Terminating         0          11m
hello-server-7c7f99c596-cvkmx   1/1     Running             0          2s

```

This is a sign that your VPA is deleting and resizing your pods. 

## HPA results

By this point, your **Horizontal Pod Autoscaler** will have most likely scaled your `php-apache` deployment down.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   1%/50%    1         10        1          19m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

You'll see that your `php-apache` deployment has been scaled down to 1 pod.



## VPA results

Now, the VPA should have resized your pods in the `hello-server` deployment.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"
Containers:
  hello-app:
    Container ID:   containerd://15adff5ee96b6b39332baa723b1653fe9dc0eccafa86156d7bd10868749e3ce3
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       gcr.io/google-samples/hello-app@sha256:b1455e1c4fcc5ea1023c9e3b584cd84b64eb920e332feff690a2829696e379e7
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 11 Apr 2024 04:21:03 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        2m
      memory:     3145728
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2f7zw (ro)
Conditions:
Containers:
  hello-app:
    Container ID:   containerd://95b0eb44f3c159a4df4a96fc80d016c36b66fc90f54db068497412905182a130
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       gcr.io/google-samples/hello-app@sha256:b1455e1c4fcc5ea1023c9e3b584cd84b64eb920e332feff690a2829696e379e7
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 11 Apr 2024 04:19:30 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        2m
      memory:     3145728
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-q449t (ro)
Conditions:
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



## Cluster autoscaler

The **Cluster Autoscaler** is designed to add or remove  nodes based on demand. When demand is high, cluster autoscaler will add  nodes to the node pool to accommodate that demand. When demand is low,  cluster autoscaler will scale your cluster back down by removing nodes.  This allows you to maintain high availability of your cluster while  minimizing superfluous costs associated with additional machines.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud beta container clusters update scaling-demo --enable-autoscaling --min-nodes 1 --max-nodes 5
Updating scaling-demo...done.                                                                                                                                                      
Updated [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-01-696b4c0e98b8/zones/us-west1-a/clusters/scaling-demo].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/scaling-demo?project=qwiklabs-gcp-01-696b4c0e98b8
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

When scaling a cluster, the decision of when to remove a node is a  trade-off between optimizing for utilization or the availability of  resources. Removing underutilized nodes improves cluster utilization,  but new workloads might have to wait for resources to be provisioned  again before they can run.

You can specify which autoscaling profile to use when making such decisions. The currently available profiles are:

- **Balanced**: The default profile.

- **Optimize-utilization**: Prioritize optimizing utilization over keeping spare resources in the cluster. When selected, the cluster autoscaler scales down the cluster more aggressively. It can remove  more nodes, and remove nodes faster. This profile has been optimized for use with batch workloads that are not sensitive to start-up latency.

  Switch to the `optimize-utilization` autoscaling profile so that the full effects of scaling can be observed:

  ```sh
  tudent_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud beta container clusters update scaling-demo \
  --autoscaling-profile optimize-utilization
  Updating scaling-demo...working                                                                                                                                                    
  Updating scaling-demo...done.                                                                                                                                                      
  Updated [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-01-696b4c0e98b8/zones/us-west1-a/clusters/scaling-demo].
  To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/scaling-demo?project=qwiklabs-gcp-01-696b4c0e98b8
  student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
  student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
  ```

  

By default, most of the system pods from these deployments will  prevent cluster autoscaler from taking them completely offline to  reschedule them. Generally, this is desired because many of these pods  collect data used in other deployments and services. For example, *metrics-agent* being temporarily down would cause a gap in data collected for VPA and HPA, or the *fluentd* pod being down could create a gap in your cloud logs.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment -n kube-system
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
event-exporter-gke              1/1     1            1           37m
konnectivity-agent              3/3     3            3           37m
konnectivity-agent-autoscaler   1/1     1            1           37m
kube-dns                        2/2     2            2           37m
kube-dns-autoscaler             1/1     1            1           37m
l7-default-backend              1/1     1            1           37m
metrics-server-v0.5.2           1/1     1            1           37m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



For the purpose of this lab, you will apply Pod Disruption Budgets to your `kube-system` pods which will allow cluster autoscaler to safely reschedule them on  another node. This will give enough room to scale your cluster down.



**Pod Disruption Budgets (PDB)** define how Kubernetes  should handle disruptions like upgrades, pod removals, running out of  resources, etc. In PDBs, you can specify the `max-unavailable` and/or the `min-available` number of pods a deployment should have.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl create poddisruptionbudget kube-dns-pdb --namespace=kube-system --selector k8s-app=kube-dns --max-unavailable 1
kubectl create poddisruptionbudget prometheus-pdb --namespace=kube-system --selector k8s-app=prometheus-to-sd --max-unavailable 1
kubectl create poddisruptionbudget kube-proxy-pdb --namespace=kube-system --selector component=kube-proxy --max-unavailable 1
kubectl create poddisruptionbudget metrics-agent-pdb --namespace=kube-system --selector k8s-app=gke-metrics-agent --max-unavailable 1
kubectl create poddisruptionbudget metrics-server-pdb --namespace=kube-system --selector k8s-app=metrics-server --max-unavailable 1
kubectl create poddisruptionbudget fluentd-pdb --namespace=kube-system --selector k8s-app=fluentd-gke --max-unavailable 1
kubectl create poddisruptionbudget backend-pdb --namespace=kube-system --selector k8s-app=glbc --max-unavailable 1
kubectl create poddisruptionbudget kube-dns-autoscaler-pdb --namespace=kube-system --selector k8s-app=kube-dns-autoscaler --max-unavailable 1
kubectl create poddisruptionbudget stackdriver-pdb --namespace=kube-system --selector app=stackdriver-metadata-agent --max-unavailable 1
kubectl create poddisruptionbudget event-pdb --namespace=kube-system --selector k8s-app=event-exporter --max-unavailable 1
poddisruptionbudget.policy/kube-dns-pdb created
poddisruptionbudget.policy/prometheus-pdb created
poddisruptionbudget.policy/kube-proxy-pdb created
poddisruptionbudget.policy/metrics-agent-pdb created
poddisruptionbudget.policy/metrics-server-pdb created
poddisruptionbudget.policy/fluentd-pdb created
poddisruptionbudget.policy/backend-pdb created
poddisruptionbudget.policy/kube-dns-autoscaler-pdb created
poddisruptionbudget.policy/stackdriver-pdb created
poddisruptionbudget.policy/event-pdb created
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

In each of these commands, you are selecting a different **kube-system** deployment pod based on a label defined in its creation and specifying  that there can be 1 unavailable pod for each of these deployments. This  will allow the autoscaler to reschedule the system pods.

With the PDBs in place, your cluster should scale down from three nodes to two nodes in a minute or two.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get nodes
NAME                                          STATUS     ROLES    AGE   VERSION
gke-scaling-demo-default-pool-0e73e0ba-3s7q   NotReady   <none>   42m   v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-hd9j   Ready      <none>   42m   v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-q83h   Ready      <none>   42m   v1.27.8-gke.1067004
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get nodes
NAME                                          STATUS     ROLES    AGE   VERSION
gke-scaling-demo-default-pool-0e73e0ba-3s7q   NotReady   <none>   42m   v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-hd9j   Ready      <none>   42m   v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-q83h   Ready      <none>   42m   v1.27.8-gke.1067004
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



You set up automation that scaled your cluster down from three nodes to two nodes!



Thinking about the costs, as a result of scaling down your nodepool,  you will be billed for less machines during periods of low demand on  your cluster. This scaling could be even more dramatic if you were  fluctuating from high demand to low demand periods during the day.

It’s important to note that, while **Cluster Autoscaler** removed an unnecessary node, **Vertical Pod Autoscaling** and **Horizontal Pod Autoscaling** helped reduce enough CPU demand so that the node was no longer needed.  Combining these tools is a great way to optimize your overall costs and  resource usage.

So, the cluster autoscaler helps add and remove nodes in response to  pods needing to be scheduled. However, GKE specifically has another  feature to scale vertically, called **node auto-provisioning**.

## Node Auto Provisioning

**Node Auto Provisioning** (NAP) actually adds new node  pools that are sized to meet demand. Without node auto provisioning, the cluster autoscaler will only be creating new nodes in the node pools  you've specified, meaning the new nodes will be the same machine type as the other nodes in that pool. This is perfect for helping optimize  resource usage for batch workloads and other apps that don't need  extreme scaling, since creating a node pool that is specifically  optimized for your use case might take more time than just adding more  nodes to an existing pool.

- Enable Node Auto Provisioning:

  ```sh
  student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ gcloud container clusters update scaling-demo \
      --enable-autoprovisioning \
      --min-cpu 1 \
      --min-memory 2 \
      --max-cpu 45 \
      --max-memory 160
  Updating scaling-demo...done.                                                                                                                                                      
  Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-696b4c0e98b8/zones/us-west1-a/clusters/scaling-demo].
  To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/scaling-demo?project=qwiklabs-gcp-01-696b4c0e98b8
  student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
  ```

  

In the command, you specify a minimum and maximum number for your CPU and memory resources. This is for the entire cluster.

**NAP** can take a little bit of time and it's also highly likely it won't create a new node pool for the **scaling-demo** cluster at its current state.

## Test with larger demand

So far, you've analyzed how HPA, VPA, and cluster autoscaler can help save resources and costs while your application has low demand. Now,  you'll look at how these tools handle availability for increased demand.

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   1%/50%    1         10        1          48m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get hpa
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   173%/50%   1         10        1          48m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

Wait and rerun the command until you see your target above 100%.

Now, monitor how your cluster handles the increased load by periodically running this command:

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment php-apache
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   2/4     4            2           50m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

After a few minutes, you will see a few things happen.

- First, your `php-apache` deployment will automatically be scaled up by HPA to handle the increased load.
- Then, **cluster autoscaler** will need to provision new nodes to handle the increased demand.
- Finally, **node auto provisioning** will create a node  pool optimized for the CPU and memory requests of your cluster’s  workloads. In this case, it should be a high cpu, low memory node pool  because the load test is pushing the cpu limits.

Wait until your `php-apache` deployment is scaled up to 7 replicas and your nodes tab looks similar to this:

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get deployment 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-server   2/2     2            2           45m
php-apache     6/6     6            6           52m
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl get nodes
NAME                                          STATUS   ROLES    AGE    VERSION
gke-scaling-demo-default-pool-0e73e0ba-ctx4   Ready    <none>   116s   v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-hd9j   Ready    <none>   53m    v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-pwnr   Ready    <none>   54s    v1.27.8-gke.1067004
gke-scaling-demo-default-pool-0e73e0ba-q83h   Ready    <none>   53m    v1.27.8-gke.1067004
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```



Your cluster efficiently scaled up to meet a higher demand! However,  take note of the amount of time it took to handle this spike in demand.  For many applications, losing availability while provisioning new  resources can be an issue.

## Optimize larger loads

When scaling up for larger loads, horizontal pod autoscaling will add new pods while vertical pod autoscaling will resize them depending on  your settings. If there's room on an existing node, it might be able to  skip pulling the image and immediately start running the application on a new pod. If you're working with a node that hasn't deployed your  application before, a bit of time might be added if it needs to download the container images before running it.

So, if you don't have enough room on your existing nodes and you're  using the cluster autoscaler, it could take even longer. Now it needs to provision a new node, set it up, then download the image and start up  pods. If the node auto-provisioner is going to create a new node pool  like it did in your cluster, there will be even more time as you  provision the new node pool first, and then go through all the same  steps for the new node.

In order to handle these different latencies for autoscaling, you'll probably want to **over-provision** a little bit so there's less pressure on your apps when autoscaling-up. This is really important for cost-optimization, because you don't want  to pay for more resources than you need, but you also don't want your  apps' performance to suffer.



An efficient strategy to overprovision a cluster with Cluster Autoscaling is to use Pause Pods.

**Pause Pods** are low priority deployments which are  able to be removed and replaced by high priority deployments. This means you can create low priority pods which don't actually do anything  except reserve buffer space. When the higher-priority pod needs room,  the pause pods will be removed and rescheduled to another node, or a new node, and the higher-priority pod has the room it needs to be scheduled quickly.

Create a manifest for a pause pod:

```yaml
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ cat pause-pod.yaml 
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class used by overprovisioning."
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: 1
            memory: 4Gi
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ kubectl apply -f pause-pod.yaml
priorityclass.scheduling.k8s.io/overprovisioning created
deployment.apps/overprovisioning created
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

Observe how a new node is created, most likely in a new node pool, to  fit your newly created pause pod. Now, if you were to run the load test  again, when you needed an extra node for your `php-apache`  deployment, it could be scheduled on the node with your pause pod while  your pause pod is instead put on a new node. This is excellent because  your dummy pause pods allow your cluster to provision a new node in  advance so that your actual application can scale up much faster. If you were expecting higher amounts of traffic, you could add more pause  pods, but it's considered best practice to not add more than one pause  pod per node.



In this lab, you configured a cluster to automatically and  efficiently scale up or down based on its demand. Horizontal Pod  Autoscaling and Vertical Pod Autoscaling provided solutions for  automatically scaling your cluster's deployments while Cluster  Autoscaler and Node Auto Provisioning provided solutions for  automatically scaling your cluster's infrastructure.

As always, knowing which of these tools to use will depend on your  workload. Careful use of these autoscalers can mean maximizing  availability when you need it while only paying for what you need during times of low demand. When thinking about costs, this means you are  optimizing your resource usage and saving money.

### ## Summary

```sh
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ history 
    1  gcloud config set compute/zone us-west1-a
    2  gcloud container clusters create scaling-demo --num-nodes=3 --enable-vertical-pod-autoscaling
    3  cat << EOF > php-apache.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  replicas: 3
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
EOF

    4  cat php-apache.yaml 
    5  kubectl apply -f php-apache.yaml
    6  kubectl get deployment
    7  kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
    8  kubectl get deployment
    9  kubectl get deployment
   10  kubectl get hpa
   11  kubectl get hpa
   12  kubectl get deployment
   13  gcloud container clusters describe scaling-demo | grep ^verticalPodAutoscaling -A 1
   14  kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
   15  kubectl get deployment hello-server
   16  kubectl set resources deployment hello-server --requests=cpu=450m
   17  kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"
   18  cat << EOF > hello-vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: hello-server-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       hello-server
  updatePolicy:
    updateMode: "Off"
EOF

   19  cat hello-vpa.yaml 
   20  kubectl apply -f hello-vpa.yaml
   21  kubectl get vpa
   22  kubectl describe vpa hello-server-vpa
   23  kubectl describe vpa hello-server-vpa
   24  sed -i 's/Off/Auto/g' hello-vpa.yaml
   25  kubectl apply -f hello-vpa.yaml
   26  cat hello-vpa.yaml 
   27  kubectl scale deployment hello-server --replicas=2
   28  kubectl get deployment
   29  kubectl get pva
   30  kubectl get vpa
   31  kubectl get pods -w
   32  kubectl get hpa
   33  kubectl describe pod hello-server | sed -n "/Containers:$/,/Conditions:/p"
   34  gcloud beta container clusters update scaling-demo --enable-autoscaling --min-nodes 1 --max-nodes 5
   35  gcloud beta container clusters update scaling-demo --autoscaling-profile optimize-utilization
   36  kubectl get deployment -n kube-system
   37  kubectl create poddisruptionbudget kube-dns-pdb --namespace=kube-system --selector k8s-app=kube-dns --max-unavailable 1
   38  kubectl create poddisruptionbudget prometheus-pdb --namespace=kube-system --selector k8s-app=prometheus-to-sd --max-unavailable 1
   39  kubectl create poddisruptionbudget kube-proxy-pdb --namespace=kube-system --selector component=kube-proxy --max-unavailable 1
   40  kubectl create poddisruptionbudget metrics-agent-pdb --namespace=kube-system --selector k8s-app=gke-metrics-agent --max-unavailable 1
   41  kubectl create poddisruptionbudget metrics-server-pdb --namespace=kube-system --selector k8s-app=metrics-server --max-unavailable 1
   42  kubectl create poddisruptionbudget fluentd-pdb --namespace=kube-system --selector k8s-app=fluentd-gke --max-unavailable 1
   43  kubectl create poddisruptionbudget backend-pdb --namespace=kube-system --selector k8s-app=glbc --max-unavailable 1
   44  kubectl create poddisruptionbudget kube-dns-autoscaler-pdb --namespace=kube-system --selector k8s-app=kube-dns-autoscaler --max-unavailable 1
   45  kubectl create poddisruptionbudget stackdriver-pdb --namespace=kube-system --selector app=stackdriver-metadata-agent --max-unavailable 1
   46  kubectl create poddisruptionbudget event-pdb --namespace=kube-system --selector k8s-app=event-exporter --max-unavailable 1
   47  kubectl get nodes
   48  kubectl get nodes -w
   49  kubectl get nodes
   50  kubectl get nodes
   51  kubectl get nodes
   52  kubectl get nodes
   53  kubectl get nodes
   54  kubectl get nodes
   55  kubectl get nodes
   56  kubectl get nodes
   57  kubectl get nodes
   58  kubectl get nodes
   59  kubectl get nodes
   60  kubectl get nodes
   61  kubectl get nodes
   62  kubectl get nodes
   63  kubectl get nodes
   64  kubectl get nodes
   65  gcloud container clusters update scaling-demo     --enable-autoprovisioning     --min-cpu 1     --min-memory 2     --max-cpu 45     --max-memory 160
   66  kubectl get hpa
   67  kubectl get hpa
   68  kubectl get deployment php-apache
   69  kubectl get deployment php-apache
   70  kubectl get deployment 
   71  kubectl get nodes
   72  kubectl get deployment 
   73  kubectl get deployment 
   74  kubectl get deployment 
   75  kubectl get nodes
   76  cat << EOF > pause-pod.yaml
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
description: "Priority class used by overprovisioning."
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: 1
            memory: 4Gi
EOF

   77  cat pause-pod.yaml 
   78  kubectl apply -f pause-pod.yaml
   79  kubectl get nodes
   80  history 
student_01_fa9e4f85d2dc@cloudshell:~ (qwiklabs-gcp-01-696b4c0e98b8)$ 
```

