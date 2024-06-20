---

layout: single
title:  "Configuring Google Kubernetes Engine (GKE) Networking"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Configuring Google Kubernetes Engine (GKE) Networking

- Create and test a private cluster.
- Configure a cluster for authorized network control plane access.
- Configure a Cluster network policy.

## Create a private cluster

In this task, you create a private cluster, consider the options for  how private to make it, and then compare your private cluster to your  original cluster.

In a private cluster, the nodes have internal RFC 1918 IP addresses  only, which ensures that their workloads are isolated from the public  Internet. The nodes in a non-private cluster have external IP addresses, potentially allowing traffic to and from the internet.

### Set up a private cluster

1. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**.
2. Click **Create** and select **SWITCH TO STANDARD CLUSTER** for the cluster.
3. Name the cluster `private-cluster`.
4. Select  as the Zone.
5. In the left pane, in **Node Pools** click **default-pool**.
6. In **Number of nodes** type **2**.

7. In the left pane, in **Cluster** click on **Networking**.

8. Click **Private cluster** and select **Access control plane using its external IP address**.

9. Ensure **Enable control plane authorized networks** is not selected.
10. Select the checkbox to **Override control plane’s default private endpoint subnet**.
11. Under **Private endpoint subnet** select as **default**.

This setting allows you the range of addresses that can access the  cluster externally. When this checkbox is not selected, you can access `kubectl` only from within the Google Cloud network. In this lab, you will only access `kubectl` through the Google Cloud network but you will modify this setting later.

```sh
gcloud beta container --project "qwiklabs-gcp-01-e175c33ff1b1" clusters create "private-cluster" --zone "us-east1-c" --no-enable-basic-auth --cluster-version "1.29.4-gke.1043002" --release-channel "regular" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-balanced" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "2" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-private-nodes --private-endpoint-subnetwork="projects/qwiklabs-gcp-01-e175c33ff1b1/regions/us-east1/subnetworks/default" --enable-ip-alias --network "projects/qwiklabs-gcp-01-e175c33ff1b1/global/networks/default" --subnetwork "projects/qwiklabs-gcp-01-e175c33ff1b1/regions/us-east1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --security-posture=standard --workload-vulnerability-scanning=disabled --enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --binauthz-evaluation-mode=DISABLED --enable-managed-prometheus --enable-shielded-nodes --node-locations "us-east1-c"
```



```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ gcloud container clusters describe private-cluster --zone us-east1-c
addonsConfig:
  dnsCacheConfig: {}
  gcePersistentDiskCsiDriverConfig:
    enabled: true
  gcsFuseCsiDriverConfig: {}
  horizontalPodAutoscaling: {}
  httpLoadBalancing: {}
  kubernetesDashboard:
    disabled: true
  networkPolicyConfig:
    disabled: true
authenticatorGroupsConfig: {}
autopilot: {}
autoscaling:
  autoscalingProfile: BALANCED
binaryAuthorization:
  evaluationMode: DISABLED
clusterIpv4Cidr: 10.96.0.0/14
createTime: '2024-06-20T14:19:03+00:00'
currentMasterVersion: 1.29.4-gke.1043002
currentNodeCount: 2
currentNodeVersion: 1.29.4-gke.1043002
databaseEncryption:
  currentState: CURRENT_STATE_DECRYPTED
  state: DECRYPTED
defaultMaxPodsConstraint:
  maxPodsPerNode: '110'
endpoint: 35.229.93.218
enterpriseConfig:
  clusterTier: STANDARD
etag: 3ef7f9f4-926f-4871-a27e-7e64823e5676
id: c905438b563e40febae6fc0355a03a030c32736e5a894d478a7982565f097ebd
initialClusterVersion: 1.29.4-gke.1043002
instanceGroupUrls:
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/instanceGroupManagers/gke-private-cluster-default-pool-b08e2ddd-grp
ipAllocationPolicy:
  clusterIpv4Cidr: 10.96.0.0/14
  clusterIpv4CidrBlock: 10.96.0.0/14
  clusterSecondaryRangeName: gke-private-cluster-pods-c905438b
  defaultPodIpv4RangeUtilization: 0.002
  podCidrOverprovisionConfig: {}
  servicesIpv4Cidr: 34.118.224.0/20
  servicesIpv4CidrBlock: 34.118.224.0/20
  stackType: IPV4
  useIpAliases: true
labelFingerprint: a9dc16a7
legacyAbac: {}
location: us-east1-c
locations:
- us-east1-c
loggingConfig:
  componentConfig:
    enableComponents:
    - SYSTEM_COMPONENTS
    - WORKLOADS
loggingService: logging.googleapis.com/kubernetes
maintenancePolicy:
  resourceVersion: e3b0c442
masterAuth:
  clientCertificateConfig: {}
  clusterCaCertificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVMVENDQXBXZ0F3SUJBZ0lSQUxuRDVXcmQ1eEhWVW1PbVh6VUt1Um93RFFZSktvWklodmNOQVFFTEJRQXcKTHpFdE1Dc0dBMVVFQXhNa1lUazBNVGRsTURBdE5tUTVZaTAwWldVM0xXRTVZMk10Tm1JNVpUTTVPVFppTlRZNApNQ0FYRFRJME1EWXlNREV6TVRrd00xb1lEekl3TlRRd05qRXpNVFF4T1RBeldqQXZNUzB3S3dZRFZRUURFeVJoCk9UUXhOMlV3TUMwMlpEbGlMVFJsWlRjdFlUbGpZeTAyWWpsbE16azVObUkxTmpnd2dnR2lNQTBHQ1NxR1NJYjMKRFFFQkFRVUFBNElCandBd2dnR0tBb0lCZ1FDMXdVcUR4MEl1TC9Vd21WV3M3QWp0VU9ZbWZMRG15dXVwM1c0awp2elBTTHQ5aWxoT3hDR1BaWllMeG9qYitNN1FPbHNRdVpERDk2UDJoOUg2QXlBMUduSmMxQXBXNCtXRGlCWHVpCkJEeElXMzJBTjhaZXBQcmQrT2FZblh5Z1lBZkJ0bFcrdktuOHJjQ3FYMEJkVHhTRUVCNWhEQjk0NXJMVW9kczgKeWFkaFREaHl4Q2VXcEozODZBSXpRaHQyallvRXFwS1kwWVNUUWQrenprVVpaK1pQWU93SDlranVWTlhCZFFqZAo3dEgxelRNUVgrYzFSeWJ2WUVjMmNIakFSN01wdUpPcEs5Umh0UGdNQ0JtalhrK2Z5bDkxSzdRU2VVc3JzNWdxCmFOM3cvNXhNa1BFWVhXTHJJQXdBWjFTKzc2WlR6bFFIdWdUbFp0VFNpVW9HWVpiSmRZcncyaC9YY1NaaE8rSFgKa0NpU045WUdvLzhSdHdvSGtRRHhNdElpZ1hqSGtNU0Frb21qMzhuSXprU0c1VXRjUkdJdytlN0JyNXBDRUJtZQppWSt5Z1VXQ3ppNE1wNkc2aHY1cUdiRXZUY3JnQzgvaklpSGsvUVVBVUdEMVk5S2RtaEhvR2xzcVZsWGMzczNkCmxRY1ZGRWRqbnpmTmVjQ0VCeHFRbDEyelYxRUNBd0VBQWFOQ01FQXdEZ1lEVlIwUEFRSC9CQVFEQWdJRU1BOEcKQTFVZEV3RUIvd1FGTUFNQkFmOHdIUVlEVlIwT0JCWUVGT1FuRzlXS3VueW14KzVZODQ4K0xMZmRwZk42TUEwRwpDU3FHU0liM0RRRUJDd1VBQTRJQmdRQ3BOWC9ZSjFtMUtLMmJSalhmb2ErZjMway9qUDl1Y09wNkxCbUxTSXFqCjNidWxDRFVUWUpVTG1tMk1TODY1TlZOUWxYWHFhemxDcTZ2TVBnNTRhdDAzNGFLaXhHKysvOExYdDZ2U1BWcjUKNmhTdi85RVM3bU9LTy9DckhmeHlFeDBJbDd3cXU2WEZZRXlrMWYyNDc5MjhYYUZaODZQcEFGS3BWZVQ2SzN0TgpRZ3g1enJrc1VsMUhMOWRmUXpWZnV5LzF2aTVDM0dLSWF1cjBORXpBWHFnWS9pckhmUlMwTjlMd0Q0a1d4VEIrCkRDVkR2aXpYZVB2TS9WZFlRb0VBR1V1NG4wZVZLR0U3cGtqMUt1Q2p3cXpaR1I3K0NCWkpGV0pRbTBpcWQ4S2kKdlIvU1V0czdVYUk3S2xhTFV3a0tTT21abXU4elBzTm1TS284Yi9talFTdEVGMFZBeVMzaEp6SkZ5Uklvclg5RwpCeEhkeEhrVWw2eFltcWpBZ2FDNFBCSmVlM09NcjYzcnRtSU5lRWxiU01sZ25VMzZmZUdyTU1Wa3B3b3NWc29oCkM3T0RkUmhiRWtlL3ptcm8yU0V2ZjRaR3crTnhhUG05aW5iL2lsVVcrQ2tsR2dnZElrRW42ZXVMWWV4QVJ4bDgKcVYvamVOUzdpbEs2YTZaMXRkeFVXbU09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
masterAuthorizedNetworksConfig:
  enabled: true
monitoringConfig:
  advancedDatapathObservabilityConfig: {}
  componentConfig:
    enableComponents:
    - SYSTEM_COMPONENTS
  managedPrometheusConfig:
    enabled: true
monitoringService: monitoring.googleapis.com/kubernetes
name: private-cluster
network: default
networkConfig:
  datapathProvider: LEGACY_DATAPATH
  defaultSnatStatus: {}
  inTransitEncryptionConfig: IN_TRANSIT_ENCRYPTION_DISABLED
  network: projects/qwiklabs-gcp-01-e175c33ff1b1/global/networks/default
  serviceExternalIpsConfig: {}
  subnetwork: projects/qwiklabs-gcp-01-e175c33ff1b1/regions/us-east1/subnetworks/default
nodeConfig:
  advancedMachineFeatures:
    enableNestedVirtualization: false
  diskSizeGb: 100
  diskType: pd-balanced
  imageType: COS_CONTAINERD
  machineType: e2-medium
  metadata:
    disable-legacy-endpoints: 'true'
  oauthScopes:
  - https://www.googleapis.com/auth/devstorage.read_only
  - https://www.googleapis.com/auth/logging.write
  - https://www.googleapis.com/auth/monitoring
  - https://www.googleapis.com/auth/servicecontrol
  - https://www.googleapis.com/auth/service.management.readonly
  - https://www.googleapis.com/auth/trace.append
  serviceAccount: default
  shieldedInstanceConfig:
    enableIntegrityMonitoring: true
  windowsNodeConfig: {}
nodePoolDefaults:
  nodeConfigDefaults:
    loggingConfig:
      variantConfig:
        variant: DEFAULT
nodePools:
- autoscaling: {}
  config:
    advancedMachineFeatures:
      enableNestedVirtualization: false
    diskSizeGb: 100
    diskType: pd-balanced
    imageType: COS_CONTAINERD
    machineType: e2-medium
    metadata:
      disable-legacy-endpoints: 'true'
    oauthScopes:
    - https://www.googleapis.com/auth/devstorage.read_only
    - https://www.googleapis.com/auth/logging.write
    - https://www.googleapis.com/auth/monitoring
    - https://www.googleapis.com/auth/servicecontrol
    - https://www.googleapis.com/auth/service.management.readonly
    - https://www.googleapis.com/auth/trace.append
    serviceAccount: default
    shieldedInstanceConfig:
      enableIntegrityMonitoring: true
    windowsNodeConfig: {}
  etag: f34d4ba1-c44a-4075-99e1-82ea4264fc7a
  initialNodeCount: 2
  instanceGroupUrls:
  - https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/instanceGroupManagers/gke-private-cluster-default-pool-b08e2ddd-grp
  locations:
  - us-east1-c
  management:
    autoRepair: true
    autoUpgrade: true
  maxPodsConstraint:
    maxPodsPerNode: '110'
  name: default-pool
  networkConfig:
    enablePrivateNodes: true
    podIpv4CidrBlock: 10.96.0.0/14
    podIpv4RangeUtilization: 0.002
    podRange: gke-private-cluster-pods-c905438b
  podIpv4CidrSize: 24
  queuedProvisioning: {}
  selfLink: https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/clusters/private-cluster/nodePools/default-pool
  status: RUNNING
  upgradeSettings:
    maxSurge: 1
    strategy: SURGE
  version: 1.29.4-gke.1043002
notificationConfig:
  pubsub: {}
privateClusterConfig:
  enablePrivateNodes: true
  privateEndpoint: 10.142.0.2
  privateEndpointSubnetwork: projects/qwiklabs-gcp-01-e175c33ff1b1/regions/us-east1/subnetworks/default
  publicEndpoint: 35.229.93.218
releaseChannel:
  channel: REGULAR
securityPostureConfig:
  mode: BASIC
  vulnerabilityMode: VULNERABILITY_DISABLED
selfLink: https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/clusters/private-cluster
servicesIpv4Cidr: 34.118.224.0/20
shieldedNodes:
  enabled: true
status: RUNNING
subnetwork: default
zone: us-east1-c
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

The following values appear only under the private cluster:

- privateEndpoint, an internal IP address. Nodes use this internal IP address to communicate with the cluster control plane.
- publicEndpoint, an external IP address. External services and  administrators can use the external IP address to communicate with the  cluster control plane.

You have several options to lock down your cluster to varying degrees:

- The whole cluster can have external access.
- The whole cluster can be private.
- The nodes can be private while the cluster control plane is public,  and you can limit which external networks are authorized to access the  cluster control plane.

Without public IP addresses, code running on the nodes can't access  the public internet unless you configure a NAT gateway such as Cloud  NAT.

You might use private clusters to provide services such as internal  APIs that are meant only to be accessed by resources inside your  network. For example, the resources might be private tools that only  your company uses. Or they might be backend services accessed by your  frontend services, and perhaps only those frontend services are accessed directly by external customers or users. In such cases, private  clusters are a good way to reduce the surface area of attack for your  application.

## Add an authorized network for cluster control plane access

After cluster creation, you might want to issue commands to your  cluster from outside Google Cloud. For example, you might decide that  only your corporate network should issue commands to your cluster  control plane. Unfortunately, you didn't specify the authorized network  on cluster creation.

In this task, you add an authorized network for cluster control plane access.

In the Google Cloud Console **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**.

Click **private-cluster** to open the Clusters details page.

In **Details** tab, under **Networking** section, for **Control plane authorized networks** click on **Edit**(![Edit icon](https://cdn.qwiklabs.com/zxK8nW520maN72Qq6D1Lt9gCeDh7QOMGWCwhny5S8sQ%3D)).

Select **Enable Control plane authorized networks**.

Click **Add an authorized network**.

For **Name**, type the name for the network, use `Corporate`.

For **Network**, type a CIDR range that you want to grant whitelisted access to your cluster control plane. As an example, you can use `192.168.1.0/24`.

Click **Done**.

Multiple networks can be added here if necessary, but no more than 50 CIDR ranges.

## Create a cluster network policy

In this task, you create a cluster network policy to restrict  communication between the Pods. A zero-trust zone is important to  prevent lateral attacks within the cluster when an intruder compromises  one of the Pods.

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ export my_zone=us-east1-c
export my_cluster=standard-cluster-1
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ source <(kubectl completion bash)
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ gcloud container clusters create $my_cluster --num-nodes 3 --enable-ip-alias --zone $my_zone --enable-network-policy
Creating cluster standard-cluster-1 in us-east1-c... Cluster is being health-checked (master is healthy)...done.                                                                   
Created [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/clusters/standard-cluster-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-east1-c/standard-cluster-1?project=qwiklabs-gcp-01-e175c33ff1b1
kubeconfig entry generated for standard-cluster-1.
NAME: standard-cluster-1
LOCATION: us-east1-c
MASTER_VERSION: 1.29.4-gke.1043002
MASTER_IP: 34.73.14.244
MACHINE_TYPE: e2-medium
NODE_VERSION: 1.29.4-gke.1043002
NUM_NODES: 3
STATUS: RUNNING
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl run hello-web --labels app=hello \
  --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose
service/hello-web created
pod/hello-web created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

### Restrict incoming traffic to Pods

Let's create a sample NetworkPolicy manifest file called `hello-allow-from-foo.yaml` . This manifest file defines an ingress policy that allows access to Pods labeled  `app: hello` from Pods labeled `app: foo`.

1. Create and open a file called `hello-allow-from-foo.yaml` with **nano** using the following command:

```yaml
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ cat hello-allow-from-foo.yaml 
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: hello-allow-from-foo
spec:
  policyTypes:
  - Ingress
  podSelector:
    matchLabels:
      app: hello
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: foo
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl apply -f hello-allow-from-foo.yaml
networkpolicy.networking.k8s.io/hello-allow-from-foo created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl get networkpolicy
NAME                   POD-SELECTOR   AGE
hello-allow-from-foo   app=hello      5s
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl describe netpol
Name:         hello-allow-from-foo
Namespace:    default
Created on:   2024-06-20 14:39:03 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=hello
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: app=foo
  Not affecting egress traffic
  Policy Types: Ingress
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

### Validate the ingress policy

1. Run a temporary Pod called `test-1` with the label `app=foo` and get a shell in the Pod:

```sh
The kubectl switches used here in conjunction with the run command are important to note.

--stdin ( alternatively -i ) creates an interactive session attached to STDIN on the container.

--tty ( alternatively -t ) allocates a TTY for each container in the pod.

--rm instructs Kubernetes to treat this as a temporary Pod that will be removed as soon as it completes its startup task. As this is an interactive session it will be removed as soon as the user exits the session.

--label ( alternatively -l ) adds a set of labels to the pod.

--restart defines the restart policy for the Pod. 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl run test-1 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty
If you don't see a command prompt, try pressing enter.
/ # 
```

Make a request to the hello-web:8080 endpoint to verify that the incoming traffic is allowed:

```sh
/ # wget -qO- --timeout=2 http://hello-web:8080
Hello, world!
Version: 1.0.0
Hostname: hello-web
/ # 
```

1. Type **exit** and press **ENTER** to leave the shell.
2. Now you will run a different Pod using the same Pod name but using a label, `app=other`, that does not match the podSelector in the active network policy. This  Pod should not have the ability to access the hello-web application.

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl run test-1 --labels app=other --image=alpine --restart=Never --rm --stdin --tty
If you don't see a command prompt, try pressing enter.
/ # 
```

Make a request to the hello-web:8080 endpoint to verify that the incoming traffic is not allowed:

```sh
/ # wget -qO- --timeout=2 http://hello-web:8080
wget: download timed out
/ # 
```

The request times out.

1. ype **exit** and press **ENTER** to leave the shell.

### Restrict outgoing traffic from the Pods

You can restrict outgoing (egress) traffic as you do incoming traffic. However, in order to query internal hostnames (such as hello-web) or  external hostnames (such as www.example.com), you must allow DNS  resolution in your egress network policies. DNS traffic occurs on port  53, using TCP and UDP protocols.

Let's create a NetworkPolicy manifest file  `foo-allow-to-hello.yaml`. This file defines a policy that permits Pods with the label `app: foo` to communicate with Pods labeled `app: hello` on any port number, and allows the Pods labeled `app: foo` to communicate to any computer on UDP port 53, which is used for DNS  resolution. Without the DNS port open, you will not be able to resolve  the hostnames.

```yaml
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ cat foo-allow-to-hello.yaml 
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: foo-allow-to-hello
spec:
  policyTypes:
  - Egress
  podSelector:
    matchLabels:
      app: foo
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: hello
  - to:
    ports:
    - protocol: UDP
      port: 53
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl apply -f foo-allow-to-hello.yaml
networkpolicy.networking.k8s.io/foo-allow-to-hello created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl get networkpolicy
NAME                   POD-SELECTOR   AGE
foo-allow-to-hello     app=foo        5s
hello-allow-from-foo   app=hello      4m18s
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl describe netpol
Name:         foo-allow-to-hello
Namespace:    default
Created on:   2024-06-20 14:43:16 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=foo
  Not affecting ingress traffic
  Allowing egress traffic:
    To Port: <any> (traffic allowed to all ports)
    To:
      PodSelector: app=hello
    ----------
    To Port: 53/UDP
    To: <any> (traffic not restricted by destination)
  Policy Types: Egress


Name:         hello-allow-from-foo
Namespace:    default
Created on:   2024-06-20 14:39:03 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     app=hello
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: app=foo
  Not affecting egress traffic
  Policy Types: Ingress
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

### Validate the egress policy

1. Deploy a new web application called hello-web-2 and expose it internally in the cluster:

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl run hello-web-2 --labels app=hello-2 \
  --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose
service/hello-web-2 created
pod/hello-web-2 created
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

Run a temporary Pod with the `app=foo` label and get a shell prompt inside the container:

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ kubectl run test-3 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty
If you don't see a command prompt, try pressing enter.
/ # wget -qO- --timeout=2 http://hello-web:8080
Hello, world!
Version: 1.0.0
Hostname: hello-web
/ # wget -qO- --timeout=2 http://hello-web-2:8080
wget: download timed out
/ # wget -qO- --timeout=2 http://www.example.com
wget: download timed out
/ # exit
pod "test-3" deleted
pod default/test-3 terminated (Error)
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

Verify that the Pod can establish connections to hello-web:8080

Verify that the Pod cannot establish connections to hello-web-2:8080. This fails because none of the Network policies you have defined allow traffic to Pods labeled `app: hello-2`.

Verify that the Pod cannot establish connections to external websites, such as www.example.com: This fails because  the network policies do not allow external http traffic (tcp port 80).

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ history 
    1  gcloud container clusters describe private-cluster --zone us-east1-c
    2  gcloud config set project        qwiklabs-gcp-01-e175c33ff1b1 
    3  gcloud container clusters describe private-cluster --zone us-east1-c
    4  export my_zone=us-east1-c
    5  export my_cluster=standard-cluster-1
    6  source <(kubectl completion bash)
    7  gcloud container clusters create $my_cluster --num-nodes 3 --enable-ip-alias --zone $my_zone --enable-network-policy
    8  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    9  kubectl run hello-web --labels app=hello   --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose
   10  nano hello-allow-from-foo.yaml
   11  cat hello-allow-from-foo.yaml 
   12  kubectl apply -f hello-allow-from-foo.yaml
   13  kubectl get networkpolicy
   14  kubectl describe netpol
   15  kubectl run test-1 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty
   16  kubectl run test-1 --labels app=other --image=alpine --restart=Never --rm --stdin --tty
   17  nano foo-allow-to-hello.yaml
   18  cat foo-allow-to-hello.yaml 
   19  kubectl apply -f foo-allow-to-hello.yaml
   20  kubectl get networkpolicy
   21  kubectl describe netpol
   22  kubectl run hello-web-2 --labels app=hello-2   --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose
   23  kubectl run test-3 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty
   24  history 
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

```sh
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ gcloud container clusters describe standard-cluster-1 --zone us-east1-c
addonsConfig:
  gcePersistentDiskCsiDriverConfig:
    enabled: true
  kubernetesDashboard:
    disabled: true
  networkPolicyConfig: {}
autopilot: {}
autoscaling:
  autoscalingProfile: BALANCED
clusterIpv4Cidr: 10.56.0.0/14
createTime: '2024-06-20T14:29:56+00:00'
currentMasterVersion: 1.29.4-gke.1043002
currentNodeCount: 3
currentNodeVersion: 1.29.4-gke.1043002
databaseEncryption:
  currentState: CURRENT_STATE_DECRYPTED
  state: DECRYPTED
defaultMaxPodsConstraint:
  maxPodsPerNode: '110'
endpoint: 34.73.14.244
enterpriseConfig:
  clusterTier: STANDARD
etag: 69f2c684-0dc3-478d-a30a-e61dbe5d5454
id: 6440de66aa404c38bb7ec81768c8c6a40d4d8d18500249c38ec1bf14b9747d6a
initialClusterVersion: 1.29.4-gke.1043002
instanceGroupUrls:
- https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/instanceGroupManagers/gke-standard-cluster-1-default-pool-16295322-grp
ipAllocationPolicy:
  clusterIpv4Cidr: 10.56.0.0/14
  clusterIpv4CidrBlock: 10.56.0.0/14
  clusterSecondaryRangeName: gke-standard-cluster-1-pods-6440de66
  defaultPodIpv4RangeUtilization: 0.0029
  podCidrOverprovisionConfig: {}
  servicesIpv4Cidr: 34.118.224.0/20
  servicesIpv4CidrBlock: 34.118.224.0/20
  stackType: IPV4
  useIpAliases: true
labelFingerprint: a9dc16a7
legacyAbac: {}
location: us-east1-c
locations:
- us-east1-c
loggingConfig:
  componentConfig:
    enableComponents:
    - SYSTEM_COMPONENTS
    - WORKLOADS
loggingService: logging.googleapis.com/kubernetes
maintenancePolicy:
  resourceVersion: e3b0c442
masterAuth:
  clientCertificateConfig: {}
  clusterCaCertificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVMRENDQXBTZ0F3SUJBZ0lRVXU2Unh2ZGpxQVZOMkdIcVBuRXlEREFOQmdrcWhraUc5dzBCQVFzRkFEQXYKTVMwd0t3WURWUVFERXlRNU1UbGpZVEpsTlMxbFlqSXhMVFJqT1dNdE9EVXpaaTB5WXpFeE4ySTVNR1JsTkdFdwpJQmNOTWpRd05qSXdNVE15T1RVM1doZ1BNakExTkRBMk1UTXhOREk1TlRkYU1DOHhMVEFyQmdOVkJBTVRKRGt4Ck9XTmhNbVUxTFdWaU1qRXROR001WXkwNE5UTm1MVEpqTVRFM1lqa3daR1UwWVRDQ0FhSXdEUVlKS29aSWh2Y04KQVFFQkJRQURnZ0dQQURDQ0FZb0NnZ0dCQU1UenIvV29RTVowL0g0QW1uSGNSWW1aUmtXWE5QN28yYzZMZlk1bApoaGJpY0NnR1psZ0pXVnFoNXBhVG94dFJ0UDN4STA1M0ZxVzJGbm9TOEQ5T0dDMkJiTUM1bzdNM3puNEN2c2VOCmM0c3M1QXozL2ZnYzVBZENhY09MSDFGSmQrakJDK3l5Q0xWaHZqL1Y3YTI5OXM3dnZJeXBSSjFyTFdEU1VZSGIKNkRLcDNCTkRjeEZjSEdwWG5RYlRaTGx6aVVzVnRKc250Z0tVT1N5bTlaallwa0h6UDhGNGhlWFVxUFZUUWU2QwpTTXNQeUU5RlZwKzdSdFMxUytQMmkyeHFaWUdrdlBDeThzUzN4YW92SmZ2REFWSjBsZWpKeW4rMm5LaWFWdndDClBWcFpZL0g0ck4rR1Q3YUlPbTY4NnRmN2VCNGxFdVRzZUozZlIyb245ZjVFQkZLeVdrUk5oZ1E4RlBnMldKM1cKbGZGc0I0RkdXd0RZaElDWmJ6YzBtNFVBaGlNWXQ2UndKSEZaVkJxVGkvNitrRENZL2YvRWFGS25HM1M0emVlYQovaGdXTkczakxRM0Fvek1xR1ZnTDA5Q0ZWeGpxblVKN2U4VEpiK0hpZHlEeExhVDZqZ3FSekxGQjZMaUhjdjlTCjFOMklJeTlPeWVFSEFqMWJoOWZTKzZvdXdRSURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQWdRd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVVpRDcwaVhlWWJSdW5qU21vbk1uR3hYYXpDdEF3RFFZSgpLb1pJaHZjTkFRRUxCUUFEZ2dHQkFEaXlocUNlT1dsUENpSlRFdlVRbnA1TitNVGRPMFZuQ3BlTXo3ay9PbnNiClV6bVloQ1NPcXRZOXM4RjNFalRnemFTcS8wekt5eTRWYzFpM3NHZ2J6MkdIaTByN05lUzF6Qk13OWFWdEpZdDgKL05JMnVNTjF3cmRhZ0hEVzMyV2NCd1V4N09UZXVId1NxWmk2cWtka1BEZEpqNnRXNDFTWVBwdHExYUt6bkcvTQpvVGhUeTNrTjd2dThIMnFmTERhVXZXelc1QWdhNnJnSVNqd0F6aDZ6Wk9RZXF0MGxQQ0UyMEJ1TDlVcWRJTUh0CmhRUXFYcENQMmdrN21HOEE4K1Uwd0JOcDVMUkg5SnFhdHdnazhGY0hGdTJiSThSUUF2amxoZ2RmYjZOaGN2anEKemtwQnMrcS9hQjh4REpmNUJrVzA3Nkdnbm1MTjFONXpaOER2MHkwdllGbnNzcnpCZHkvVmUxS2Q3NkQ2V1F0RApCU2k3ZTVEaHhYS05rSnYxWDlpR0R4YkpLdXZTRGg2Y1dIWXUxYmcxWCtVV2hzL2g4RklDbXA0YXNFdU8veUtyCi9WTlUxN1JNcGdoVUFoQnJHek1ZMHlUZlpZNEpDL1JTQ2pQaHdhZlNKbnJTOFpUWVVnNUJKMnZzdWQyamIrcnYKTDdMN1VHSFZiVEZFcWZSRGNxaDdHZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
masterAuthorizedNetworksConfig:
  gcpPublicCidrsAccessEnabled: true
monitoringConfig:
  advancedDatapathObservabilityConfig: {}
  componentConfig:
    enableComponents:
    - SYSTEM_COMPONENTS
  managedPrometheusConfig:
    enabled: true
monitoringService: monitoring.googleapis.com/kubernetes
name: standard-cluster-1
network: default
networkConfig:
  network: projects/qwiklabs-gcp-01-e175c33ff1b1/global/networks/default
  serviceExternalIpsConfig: {}
  subnetwork: projects/qwiklabs-gcp-01-e175c33ff1b1/regions/us-east1/subnetworks/default
networkPolicy:
  enabled: true
  provider: CALICO
nodeConfig:
  diskSizeGb: 100
  diskType: pd-balanced
  imageType: COS_CONTAINERD
  machineType: e2-medium
  metadata:
    disable-legacy-endpoints: 'true'
  oauthScopes:
  - https://www.googleapis.com/auth/devstorage.read_only
  - https://www.googleapis.com/auth/logging.write
  - https://www.googleapis.com/auth/monitoring
  - https://www.googleapis.com/auth/service.management.readonly
  - https://www.googleapis.com/auth/servicecontrol
  - https://www.googleapis.com/auth/trace.append
  serviceAccount: default
  shieldedInstanceConfig:
    enableIntegrityMonitoring: true
  windowsNodeConfig: {}
nodePoolDefaults:
  nodeConfigDefaults:
    loggingConfig:
      variantConfig:
        variant: DEFAULT
nodePools:
- config:
    diskSizeGb: 100
    diskType: pd-balanced
    imageType: COS_CONTAINERD
    machineType: e2-medium
    metadata:
      disable-legacy-endpoints: 'true'
    oauthScopes:
    - https://www.googleapis.com/auth/devstorage.read_only
    - https://www.googleapis.com/auth/logging.write
    - https://www.googleapis.com/auth/monitoring
    - https://www.googleapis.com/auth/service.management.readonly
    - https://www.googleapis.com/auth/servicecontrol
    - https://www.googleapis.com/auth/trace.append
    serviceAccount: default
    shieldedInstanceConfig:
      enableIntegrityMonitoring: true
    windowsNodeConfig: {}
  etag: 3e6e9d98-0364-4c68-b5a0-2638fc9cf6de
  initialNodeCount: 3
  instanceGroupUrls:
  - https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/instanceGroupManagers/gke-standard-cluster-1-default-pool-16295322-grp
  locations:
  - us-east1-c
  management:
    autoRepair: true
    autoUpgrade: true
  maxPodsConstraint:
    maxPodsPerNode: '110'
  name: default-pool
  networkConfig:
    enablePrivateNodes: false
    podIpv4CidrBlock: 10.56.0.0/14
    podIpv4RangeUtilization: 0.0029
    podRange: gke-standard-cluster-1-pods-6440de66
  podIpv4CidrSize: 24
  selfLink: https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/clusters/standard-cluster-1/nodePools/default-pool
  status: RUNNING
  upgradeSettings:
    maxSurge: 1
    strategy: SURGE
  version: 1.29.4-gke.1043002
notificationConfig:
  pubsub: {}
privateClusterConfig:
  privateEndpoint: 10.142.0.5
  publicEndpoint: 34.73.14.244
releaseChannel:
  channel: REGULAR
securityPostureConfig:
  mode: BASIC
  vulnerabilityMode: VULNERABILITY_MODE_UNSPECIFIED
selfLink: https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-e175c33ff1b1/zones/us-east1-c/clusters/standard-cluster-1
servicesIpv4Cidr: 34.118.224.0/20
shieldedNodes:
  enabled: true
status: RUNNING
subnetwork: default
zone: us-east1-c
student_01_ea69a93ec367@cloudshell:~ (qwiklabs-gcp-01-e175c33ff1b1)$ 
```

