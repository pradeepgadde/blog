---
layout: single
title:  "Kubernetes with Calico"
categories: Kubernetes
tags: Calico
author: "Pradeep Gadde"
classes: wide

show_date: true
 
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  teaser: /assets/images/kubernetes.png
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Kubernetes with Calico 

# Cluster deployment

Do you know how to build a cluster? Would you like to learn to automate it? then this workshop is for you!

There are a million ways to deploy a cluster, this workshop gives you a  walkthrough of couple of these methods that are usually used in a  enterprise setup to create a cluster.

## Deploy a k3s cluster
```
root@control:~# curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=172.16.0.0/16 --disable-network-policy --disable=traefik,local-storage,metrics-server" sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.32.3+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.32.3+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.32.3+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, command exists in PATH at /snap/bin/kubectl
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
root@control:~# 
```

## Copy the cluster key file

```
root@control:~# cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
root@control:~# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
root@control:~# 
```

```
root@control:~# kubectl get nodes
NAME      STATUS     ROLES                  AGE     VERSION
control   NotReady   control-plane,master   4m35s   v1.32.3+k3s1
root@control:~# kubectl get pods -A
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-ff8999cc5-xcnm2   0/1     Pending   0          4m33s
root@control:~# 
```



## Install Calico
The recommended way to install Calico is through the tigera-operator. The best part about an operator-based installation is its integration with Kubernetes because it can group Pods, Deployments, Configmaps, or services that are required for your cloud-native application and give you a single interface to manage and deploy them. With the help of the operator SDK, the tigera-operator can configure, maintain or upgrade your Calico installation.

Use the following command to install tigera-operator:

```
root@control:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
root@control:~# 
```

```
root@control:~# kubectl get crds
NAME                                                  CREATED AT
addons.k3s.cattle.io                                  2025-04-11T06:09:20Z
apiservers.operator.tigera.io                         2025-04-11T06:14:30Z
bgpconfigurations.crd.projectcalico.org               2025-04-11T06:14:30Z
bgpfilters.crd.projectcalico.org                      2025-04-11T06:14:30Z
bgppeers.crd.projectcalico.org                        2025-04-11T06:14:30Z
blockaffinities.crd.projectcalico.org                 2025-04-11T06:14:30Z
caliconodestatuses.crd.projectcalico.org              2025-04-11T06:14:30Z
clusterinformations.crd.projectcalico.org             2025-04-11T06:14:30Z
etcdsnapshotfiles.k3s.cattle.io                       2025-04-11T06:09:20Z
felixconfigurations.crd.projectcalico.org             2025-04-11T06:14:30Z
globalnetworkpolicies.crd.projectcalico.org           2025-04-11T06:14:30Z
globalnetworksets.crd.projectcalico.org               2025-04-11T06:14:30Z
helmchartconfigs.helm.cattle.io                       2025-04-11T06:09:20Z
helmcharts.helm.cattle.io                             2025-04-11T06:09:20Z
hostendpoints.crd.projectcalico.org                   2025-04-11T06:14:30Z
imagesets.operator.tigera.io                          2025-04-11T06:14:30Z
installations.operator.tigera.io                      2025-04-11T06:14:30Z
ipamblocks.crd.projectcalico.org                      2025-04-11T06:14:30Z
ipamconfigs.crd.projectcalico.org                     2025-04-11T06:14:30Z
ipamhandles.crd.projectcalico.org                     2025-04-11T06:14:30Z
ippools.crd.projectcalico.org                         2025-04-11T06:14:30Z
ipreservations.crd.projectcalico.org                  2025-04-11T06:14:30Z
kubecontrollersconfigurations.crd.projectcalico.org   2025-04-11T06:14:30Z
networkpolicies.crd.projectcalico.org                 2025-04-11T06:14:30Z
networksets.crd.projectcalico.org                     2025-04-11T06:14:30Z
tigerastatuses.operator.tigera.io                     2025-04-11T06:14:30Z
root@control:~# 
```

Use the following Installation resource to install Calico:

```
root@control:~# kubectl create -f - <<EOF
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  registry: quay.io/
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 172.16.0.0/16
      encapsulation: VXLAN
      natOutgoing: Enabled
      nodeSelector: all()
EOF
installation.operator.tigera.io/default created
root@control:~# 
```

After an installation resource is created, the operator will  initialize its deployment process to set up networking and policy engine by installing Calico, ensuring that essential settings are in place for your cluster's operation.

Use the following command to check your installation status:

```
root@control:~# kubectl get tigerastatus
NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   False       True          False      36s
root@control:~# kubectl get tigerastatus
NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   False       True          False      49s
root@control:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE    VERSION
control   Ready    control-plane,master   8m3s   v1.32.3+k3s1
root@control:~# kubectl get pods -A
NAMESPACE         NAME                                       READY   STATUS    RESTARTS   AGE
calico-system     calico-kube-controllers-5b7b868fc6-zwjbr   1/1     Running   0          68s
calico-system     calico-node-m8rxq                          1/1     Running   0          57s
calico-system     calico-typha-66ccd7f588-x56ph              1/1     Running   0          68s
calico-system     csi-node-driver-fdbgb                      2/2     Running   0          68s
kube-system       coredns-ff8999cc5-xcnm2                    1/1     Running   0          8m
tigera-operator   tigera-operator-5bc947f575-tgx4k           1/1     Running   0          2m56s
root@control:~# kubectl get tigerastatus
NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   True        False         False      10s
root@control:~# 
```

## Cluster API

Cluster API is a Kubernetes sub-project focused on providing  declarative APIs and tooling to simplify provisioning, upgrading, and  operating multiple Kubernetes clusters.

Started by the Kubernetes Special Interest Group (SIG) Cluster  Lifecycle, the Cluster API project uses Kubernetes-style APIs and  patterns to automate cluster lifecycle management for platform  operators. The supporting infrastructure, like virtual machines,  networks, load balancers, and VPCs, as well as the Kubernetes cluster  configuration are all defined in the same way that application  developers operate deploying and managing their workloads. This enables  consistent and repeatable cluster deployments across a wide variety of  infrastructure environments.

## Enabling Feature Gates

Feature gates can be enabled by exporting environment variables  before executing clusterctl init. For example, the ClusterTopology  feature, which is required to enable support for managed topologies and  ClusterClass, can be enabled 

via:



```
root@control:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
control   Ready    control-plane,master   11m   v1.32.3+k3s1
root@control:~# kubectl get cluster
error: the server doesn't have a resource type "cluster"
root@control:~# kubectl get pods -A
NAMESPACE         NAME                                       READY   STATUS    RESTARTS   AGE
calico-system     calico-kube-controllers-5b7b868fc6-zwjbr   1/1     Running   0          4m46s
calico-system     calico-node-m8rxq                          1/1     Running   0          4m35s
calico-system     calico-typha-66ccd7f588-x56ph              1/1     Running   0          4m46s
calico-system     csi-node-driver-fdbgb                      2/2     Running   0          4m46s
kube-system       coredns-ff8999cc5-xcnm2                    1/1     Running   0          11m
tigera-operator   tigera-operator-5bc947f575-tgx4k           1/1     Running   0          6m34s
root@control:~# 

```

```
root@control:~# export CLUSTER_TOPOLOGY=true
root@control:~# clusterctl init --infrastructure virtink
Fetching providers
Installing cert-manager Version="v1.14.2"
Waiting for cert-manager to be available...
Installing Provider="cluster-api" Version="v1.9.6" TargetNamespace="capi-system"
Installing Provider="bootstrap-kubeadm" Version="v1.9.6" TargetNamespace="capi-kubeadm-bootstrap-system"
Installing Provider="control-plane-kubeadm" Version="v1.9.6" TargetNamespace="capi-kubeadm-control-plane-system"
Installing Provider="infrastructure-virtink" Version="v0.7.0" TargetNamespace="capch-system"

Your management cluster has been initialized successfully!

You can now create your first workload cluster by running the following:

  clusterctl generate cluster [name] --kubernetes-version [version] | kubectl apply -f -

root@control:~# 
```

Virtink is a Kubernetes add-on for running Cloud Hypervisor virtual  machines. By using Cloud Hypervisor as the underlying hypervisor,  Virtink enables a lightweight and secure way to run fully virtualized  workloads in a canonical Kubernetes cluster.

```
root@control:~# kubectl apply -f virtink.yaml
namespace/virtink-system created
customresourcedefinition.apiextensions.k8s.io/virtualmachinemigrations.virt.virtink.smartx.com created
customresourcedefinition.apiextensions.k8s.io/virtualmachines.virt.virtink.smartx.com created
serviceaccount/virt-controller created
serviceaccount/virt-daemon created
clusterrole.rbac.authorization.k8s.io/virt-controller created
clusterrole.rbac.authorization.k8s.io/virt-daemon created
clusterrolebinding.rbac.authorization.k8s.io/virt-controller created
clusterrolebinding.rbac.authorization.k8s.io/virt-daemon created
service/virt-controller created
deployment.apps/virt-controller created
daemonset.apps/virt-daemon created
certificate.cert-manager.io/virt-controller-cert created
certificate.cert-manager.io/virt-daemon-cert created
issuer.cert-manager.io/virt-controller-cert-issuer created
issuer.cert-manager.io/virt-daemon-cert-issuer created
mutatingwebhookconfiguration.admissionregistration.k8s.io/virtink-mutating-webhook-configuration created
validatingwebhookconfiguration.admissionregistration.k8s.io/virtink-validating-webhook-configuration created
root@control:~# 
```

Default Virtink images are created for Kubernetes v1.24.0, and since we  are going to use customized images to deploy the latest version of  Kubernetes. (v1.29.0)

```
root@control:~# export VIRTINK_CONTROL_PLANE_MACHINE_KERNEL_IMAGE=us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-kernel-5.15.12
export VIRTINK_CONTROL_PLANE_MACHINE_ROOTFS_IMAGE=us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-rootfs-1.29.0
export VIRTINK_WORKER_MACHINE_KERNEL_IMAGE=us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-kernel-5.15.12
export VIRTINK_WORKER_MACHINE_ROOTFS_IMAGE=us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-rootfs-1.29.0
root@control:~# 
```



```
root@control:~# clusterctl generate cluster capi-quickstart --infrastructure virtink --flavor internal \
  --kubernetes-version v1.29.0 \
  --control-plane-machine-count=1 \
  --worker-machine-count=1 \
  > capi-quickstart.yaml
root@control:~#
```



```yaml
root@control:~# cat capi-quickstart.yaml 
apiVersion: cluster.x-k8s.io/v1beta1
kind: Cluster
metadata:
  name: capi-quickstart
  namespace: default
spec:
  clusterNetwork:
    pods:
      cidrBlocks:
      - 192.168.0.0/16
    services:
      cidrBlocks:
      - 10.96.0.0/12
  controlPlaneRef:
    apiVersion: controlplane.cluster.x-k8s.io/v1beta1
    kind: KubeadmControlPlane
    name: capi-quickstart-cp
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
    kind: VirtinkCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: VirtinkCluster
metadata:
  name: capi-quickstart
  namespace: default
spec:
  controlPlaneServiceTemplate:
    metadata:
      namespace: default
    type: NodePort
---
apiVersion: controlplane.cluster.x-k8s.io/v1beta1
kind: KubeadmControlPlane
metadata:
  name: capi-quickstart-cp
  namespace: default
spec:
  kubeadmConfigSpec:
    initConfiguration:
      nodeRegistration:
        ignorePreflightErrors:
        - SystemVerification
        kubeletExtraArgs:
          provider-id: virtink://{{ ds.meta_data.instance_id }}
    joinConfiguration:
      nodeRegistration:
        ignorePreflightErrors:
        - SystemVerification
        kubeletExtraArgs:
          provider-id: virtink://{{ ds.meta_data.instance_id }}
    postKubeadmCommands:
    - rm -rf /usr/share/capch/images/*.tar
    preKubeadmCommands:
    - for image in $(find /usr/share/capch/images -name '*.tar'); do ctr -n k8s.io
      images import $image; done
  machineTemplate:
    infrastructureRef:
      apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
      kind: VirtinkMachineTemplate
      name: capi-quickstart-cp
  replicas: 1
  version: v1.29.0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: VirtinkMachineTemplate
metadata:
  name: capi-quickstart-cp
  namespace: default
spec:
  template:
    spec:
      virtualMachineTemplate:
        metadata:
          namespace: default
        spec:
          affinity:
            podAntiAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
              - podAffinityTerm:
                  labelSelector:
                    matchExpressions:
                    - key: cluster.x-k8s.io/control-plane
                      operator: Exists
                  topologyKey: kubernetes.io/hostname
                weight: 100
          instance:
            cpu:
              coresPerSocket: 2
              sockets: 1
            disks:
            - name: rootfs
            interfaces:
            - name: pod
            kernel:
              cmdline: console=ttyS0 root=/dev/vda rw
              image: us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-kernel-5.15.12
            memory:
              size: 4Gi
          networks:
          - name: pod
            pod: {}
          readinessProbe:
            httpGet:
              path: /readyz
              port: 6443
              scheme: HTTPS
          runPolicy: Once
          volumes:
          - containerRootfs:
              image: us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-rootfs-1.29.0
              size: 4Gi
            name: rootfs
---
apiVersion: cluster.x-k8s.io/v1beta1
kind: MachineHealthCheck
metadata:
  name: capi-quickstart-cp-unhealthy-5m
  namespace: default
spec:
  clusterName: capi-quickstart
  selector:
    matchLabels:
      cluster.x-k8s.io/control-plane: ""
  unhealthyConditions:
  - status: Unknown
    timeout: 300s
    type: Ready
  - status: "False"
    timeout: 300s
    type: Ready
---
apiVersion: cluster.x-k8s.io/v1beta1
kind: MachineDeployment
metadata:
  name: capi-quickstart-md-0
  namespace: default
spec:
  clusterName: capi-quickstart
  replicas: 1
  selector:
    matchLabels: {}
  template:
    spec:
      bootstrap:
        configRef:
          apiVersion: bootstrap.cluster.x-k8s.io/v1beta1
          kind: KubeadmConfigTemplate
          name: capi-quickstart-md-0
      clusterName: capi-quickstart
      infrastructureRef:
        apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
        kind: VirtinkMachineTemplate
        name: capi-quickstart-md-0
      version: v1.29.0
---
apiVersion: bootstrap.cluster.x-k8s.io/v1beta1
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-md-0
  namespace: default
spec:
  template:
    spec:
      joinConfiguration:
        nodeRegistration:
          ignorePreflightErrors:
          - SystemVerification
          kubeletExtraArgs:
            provider-id: virtink://{{ ds.meta_data.instance_id }}
      postKubeadmCommands:
      - rm -rf /usr/share/capch/images/*.tar
      preKubeadmCommands:
      - for image in $(find /usr/share/capch/images -name '*.tar'); do ctr -n k8s.io
        images import $image; done
---
apiVersion: infrastructure.cluster.x-k8s.io/v1beta1
kind: VirtinkMachineTemplate
metadata:
  name: capi-quickstart-md-0
  namespace: default
spec:
  template:
    spec:
      virtualMachineTemplate:
        metadata:
          namespace: default
        spec:
          affinity:
            podAntiAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
              - podAffinityTerm:
                  labelSelector:
                    matchExpressions:
                    - key: cluster.x-k8s.io/deployment-name
                      operator: In
                      values:
                      - capi-quickstart-md-0
                  topologyKey: kubernetes.io/hostname
                weight: 100
          instance:
            cpu:
              coresPerSocket: 2
              sockets: 1
            disks:
            - name: rootfs
            interfaces:
            - name: pod
            kernel:
              cmdline: console=ttyS0 root=/dev/vda rw
              image: us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-kernel-5.15.12
            memory:
              size: 4Gi
          networks:
          - name: pod
            pod: {}
          runPolicy: Once
          volumes:
          - containerRootfs:
              image: us-west1-docker.pkg.dev/tigera-marketing/tigera-instruqt/capch-rootfs-1.29.0
              size: 4Gi
            name: rootfs
---
apiVersion: cluster.x-k8s.io/v1beta1
kind: MachineHealthCheck
metadata:
  name: capi-quickstart-md-0-unhealthy-5m
  namespace: default
spec:
  clusterName: capi-quickstart
  selector:
    matchLabels:
      cluster.x-k8s.io/deployment-name: capi-quickstart-md-0
  unhealthyConditions:
  - status: Unknown
    timeout: 300s
    type: Ready
  - status: "False"
    timeout: 300s
    type: Ready
root@control:~# 
```



```
root@control:~# kubectl apply -f capi-quickstart.yaml
cluster.cluster.x-k8s.io/capi-quickstart created
virtinkcluster.infrastructure.cluster.x-k8s.io/capi-quickstart created
kubeadmcontrolplane.controlplane.cluster.x-k8s.io/capi-quickstart-cp created
virtinkmachinetemplate.infrastructure.cluster.x-k8s.io/capi-quickstart-cp created
machinehealthcheck.cluster.x-k8s.io/capi-quickstart-cp-unhealthy-5m created
machinedeployment.cluster.x-k8s.io/capi-quickstart-md-0 created
kubeadmconfigtemplate.bootstrap.cluster.x-k8s.io/capi-quickstart-md-0 created
virtinkmachinetemplate.infrastructure.cluster.x-k8s.io/capi-quickstart-md-0 created
machinehealthcheck.cluster.x-k8s.io/capi-quickstart-md-0-unhealthy-5m created
root@control:~# 
```

```
root@control:~# kubectl get cluster
NAME              CLUSTERCLASS   PHASE         AGE   VERSION
capi-quickstart                  Provisioned   26s   
root@control:~# clusterctl describe cluster capi-quickstart
NAME                                                                             READY  SEVERITY  REASON                           SINCE  MESSAGE                                                       
Cluster/capi-quickstart                                                          False  Warning   ScalingUp                        33s    Scaling up control plane to 1 replicas (actual 0)              
├─ClusterInfrastructure - VirtinkCluster/capi-quickstart                                                                                                                                                 
├─ControlPlane - KubeadmControlPlane/capi-quickstart-cp                          False  Warning   ScalingUp                        33s    Scaling up control plane to 1 replicas (actual 0)              
│ └─Machine/capi-quickstart-cp-f48mj                                             False  Info      WaitingForInfrastructure         32s    1 of 2 completed                                               
│   └─MachineInfrastructure - VirtinkMachine/capi-quickstart-cp-f48mj                                                                                                                                    
└─Workers                                                                                                                                                                                                
  └─MachineDeployment/capi-quickstart-md-0                                       False  Warning   WaitingForAvailableMachines      33s    Minimum availability requires 1 replicas, current 0 available  
    └─Machine/capi-quickstart-md-0-k9pwq-r7jjc                                   False  Info      WaitingForInfrastructure         18s    0 of 2 completed                                               
      ├─BootstrapConfig - KubeadmConfig/capi-quickstart-md-0-k9pwq-r7jjc         False  Info      WaitingForControlPlaneAvailable  18s                                                                   
      └─MachineInfrastructure - VirtinkMachine/capi-quickstart-md-0-k9pwq-r7jjc                                                                                                                          
root@control:~# 
```



```
root@control:~# kubectl get kubeadmcontrolplane
NAME                 CLUSTER           INITIALIZED   API SERVER AVAILABLE   REPLICAS   READY   UPDATED   UNAVAILABLE   AGE   VERSION
capi-quickstart-cp   capi-quickstart                                        1                  1         1             74s   v1.29.0
root@control:~# clusterctl get kubeconfig capi-quickstart > capi-quickstart.kubeconfig
root@control:~# export KUBECONFIG=capi-quickstart.kubeconfig
root@control:~# 
```

Now that you have exported the config file, kubectl can be used to query the newly provisioned cluster. Use your favorite kubectl commands to examine the cluster.



```
root@control:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
control   Ready    control-plane,master   18m   v1.32.3+k3s1
root@control:~# kubectl get pods -A
NAMESPACE                           NAME                                                             READY   STATUS    RESTARTS   AGE
calico-system                       calico-kube-controllers-5b7b868fc6-zwjbr                         1/1     Running   0          11m
calico-system                       calico-node-m8rxq                                                1/1     Running   0          11m
calico-system                       calico-typha-66ccd7f588-x56ph                                    1/1     Running   0          11m
calico-system                       csi-node-driver-fdbgb                                            2/2     Running   0          11m
capch-system                        capch-controller-manager-bbc7cd8c-4mbz7                          2/2     Running   0          6m17s
capi-kubeadm-bootstrap-system       capi-kubeadm-bootstrap-controller-manager-5b959f764c-wtdtk       1/1     Running   0          6m19s
capi-kubeadm-control-plane-system   capi-kubeadm-control-plane-controller-manager-787bd68f69-wrsvf   1/1     Running   0          6m18s
capi-system                         capi-controller-manager-79c8c7bc9d-zl5sq                         1/1     Running   0          6m20s
cert-manager                        cert-manager-8df9d88c7-mhtff                                     1/1     Running   0          6m34s
cert-manager                        cert-manager-cainjector-866445547b-lnvrg                         1/1     Running   0          6m34s
cert-manager                        cert-manager-webhook-5d655548c8-s2ccl                            1/1     Running   0          6m34s
default                             vm-capi-quickstart-cp-f48mj-4924z                                1/1     Running   0          3m24s
default                             vm-capi-quickstart-md-0-k9pwq-r7jjc-2gmgg                        1/1     Running   0          95s
kube-system                         coredns-ff8999cc5-xcnm2                                          1/1     Running   0          18m
tigera-operator                     tigera-operator-5bc947f575-tgx4k                                 1/1     Running   0          13m
virtink-system                      virt-controller-d99cd5587-mqh2z                                  1/1     Running   0          5m8s
virtink-system                      virt-daemon-nml9l                                                1/1     Running   0          5m8s
root@control:~# 
```



GitOps is a methodology for managing and automating the deployment,  monitoring, and management of infrastructure and applications in a  cloud-native environment. It leverages Git as the single source of truth for declarative infrastructure and application code, allowing teams to  use Git workflows and pull requests to manage changes to their  infrastructure and application configurations.

```
root@control:~# kubectl create namespace argocd
namespace/argocd created
root@control:~# kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-applicationset-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-notifications-controller created
serviceaccount/argocd-redis created
serviceaccount/argocd-repo-server created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-applicationset-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-notifications-controller created
role.rbac.authorization.k8s.io/argocd-redis created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-notifications-controller created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-applicationset-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-notifications-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-notifications-secret created
secret/argocd-secret created
service/argocd-applicationset-controller created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-notifications-controller-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-applicationset-controller created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-notifications-controller created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-applicationset-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-notifications-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
root@control:~# 
```

```
root@control:~# kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
service/argocd-server patched
root@control:~# kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
PHAAZtiTMSeWVR0Y
root@control:~# 
```

```sh
root@control:~# argocd login 127.0.0.1
WARNING: server certificate had error: tls: failed to verify certificate: x509: cannot validate certificate for 127.0.0.1 because it doesn't contain any IP SANs. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context '127.0.0.1' updated
root@control:~# 
```

Calico perfectly fits with ArgoCD deployment pipeline. This is  because Calico resources are manifests and can easily be managed by  Argo.

Create an Argo app that deploys a default deny policy:

```
root@control:~# argocd app create defaultdeny --repo https://github.com/frozenprocess/calico-argocd-demo  --dest-server https://kubernetes.default.svc --path default-deny
application 'defaultdeny' created
root@control:~# 

```

