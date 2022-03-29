---
layout: single
title:  "OpenShift"
date:   2022-02-23 10:55:04 +0530e
categories: OpenShift  Automation
tags: redhat  codereadycontainers
author: "Pradeep Gadde"
classes: wide
toc: true
show_date: true
header:
  teaser: /assets/images/openshift.png
author:
  name     : "OpenShift"
  avatar   : "/assets/images/openshift.png"

sidebar:
  - title: "Blog"
    
    text: "Checkout other topics"
    nav: my-sidebar
---

![](/assets/images/RedHatOpenShift.png)
## RedHat OpenShift
Getting Started with OpenShift.



### Learning Resources

https://github.com/sandervanvugt/openshift

OCP

Kubernetes Distribution with additional features: Source Code Integration, CI/CD

Service Mesh like Istio Integration

CodeReady Containers https://developers.redhat.com/products/codeready-containers/overview

Offerings

RHOCP : RedHat OpenShift Container Platform: managed by customers (on-prem)

RedHat OpenShift Dedicated: RedHat managed cluster in AWS/GCP/Azure/IBM Clouds

RedHat OpenShift Online: shared across multiple customers, RedHat managed

RedHat OpenShift Kubernetes Engine : Just kubernetes

RedHat CodeReady Containers : a minimal installation

OKD: Open source upstream (OpenShift Kubernetes Distribution)



OpenShift is more expensive than Rancher, which is another Kubernetes distribution.



## Understanding Containers 

Docker Container engine:  common 

CRI-o , native in RHEL8 

In OpenShift, containers are managed in Pods. A pod consists of one or more containers 

OpenShift adds features on top of Kubernetes, but uses core k8s infrastructure

OpenShift adds resource types to K8s environment

Most OpenShift services are implemented in containers



## Setting up a Lab

RedHat Certified Specialist in Containers and Kubernetes EX180

Chapter#6

OpenShift 3.x MiniShift (not Recommended)

OpenShift 4.x CRC  (Recommended), CRC is awesome!

Alternative option: 30-day trial version of Developer Sandbox



CodeReady Containers (CRC) : all-in-one RedHat licensed OpenShift 4.x installation

Min 4vCPUs, 16GB RAM minimum

```sh
pradeep@learnOpenShift$ crc
No command given
CodeReady Containers is a tool that manages a local OpenShift 4.x cluster optimized for testing and development purposes

Usage:
  crc [flags]
  crc [command]

Available Commands:
  bundle      Manage CRC bundles
  cleanup     Undo config changes
  completion  generate the autocompletion script for the specified shell
  config      Modify crc configuration
  console     Open the OpenShift Web Console in the default browser
  delete      Delete the OpenShift cluster
  help        Help about any command
  ip          Get IP address of the running OpenShift cluster
  oc-env      Add the 'oc' executable to PATH
  podman-env  Setup podman environment
  setup       Set up prerequisites for the OpenShift cluster
  start       Start the OpenShift cluster
  status      Display status of the OpenShift cluster
  stop        Stop the OpenShift cluster
  version     Print version information

Flags:
  -h, --help               help for crc
      --log-level string   log level (e.g. "debug | info | warn | error") (default "info")

Use "crc [command] --help" for more information about a command.
```

```sh
pradeep@learnOpenShift$ crc status
CRC VM:          Stopped
OpenShift:       Stopped (v4.9.12)
Disk Usage:      0B of 0B (Inside the CRC VM)
Cache Usage:     12.78GB
Cache Directory: /Users/pradeep/.crc/cache
```

```sh
pradeep@learnOpenShift$ crc start
WARN A new version (1.39.0) has been published on https://developers.redhat.com/content-gateway/file/pub/openshift-v4/clients/crc/1.39.0/crc-macos-amd64.pkg
INFO Checking if running as non-root
INFO Checking if crc-admin-helper executable is cached
INFO Checking for obsolete admin-helper executable
INFO Checking if running on a supported CPU architecture
INFO Checking minimum RAM requirements
INFO Checking if running emulated on a M1 CPU
INFO Checking if HyperKit is installed
INFO Checking if qcow-tool is installed
INFO Checking if crc-driver-hyperkit is installed
INFO Starting CodeReady Containers VM for OpenShift 4.9.12...
INFO CodeReady Containers instance is running with IP 127.0.0.1
INFO CodeReady Containers VM is running
INFO Check internal and public DNS query...
INFO Check DNS query from host...
INFO Verifying validity of the kubelet certificates...
INFO Starting OpenShift kubelet service
INFO Kubelet client certificate has expired, renewing it... [will take up to 8 minutes]


INFO Kubelet serving certificate has expired, waiting for automatic renewal... [will take up to 8 minutes]
INFO Waiting for kube-apiserver availability... [takes around 2min]
INFO Waiting for user's pull secret part of instance disk...

INFO Starting OpenShift cluster... [waiting for the cluster to stabilize]
INFO Operator operator-lifecycle-manager-packageserver is progressing
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager

INFO 2 operators are progressing: kube-apiserver, openshift-controller-manager
INFO Operator openshift-controller-manager is progressing
INFO Operator openshift-controller-manager is progressing
INFO Operator openshift-controller-manager is progressing
INFO Operator openshift-controller-manager is progressing
INFO Operator openshift-controller-manager is progressing
INFO Operator openshift-controller-manager is progressing
INFO All operators are available. Ensuring stability...
INFO Operator authentication is not yet available
ERRO Cluster is not ready: cluster operators are still not stable after 10m44.701469756s
INFO Adding crc-admin and crc-developer contexts to kubeconfig...
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: ZyxGy-rKUxa-kE4Pp-EJLFc

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443
```

```sh
pradeep@learnOpenShift$ crc status
CRC VM:          Running
OpenShift:       Starting (v4.9.12)
Disk Usage:      13.14GB of 32.74GB (Inside the CRC VM)
Cache Usage:     12.78GB
Cache Directory: /Users/pradeep/.crc/cache
```

```sh
pradeep@learnOpenShift$ eval $(crc oc-env)
```

```sh
pradeep@learnOpenShift$ crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p ZyxGy-rKUxa-kE4Pp-EJLFc https://api.crc.testing:6443'
```

```sh
pradeep@learnOpenShift$ oc login -u developer https://api.crc.testing:6443
Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>
```



## Setting up an Application in OpenShift   

### Using Console 




![/assets/images/OS-1](/assets/images/OS-1.png) 

![/assets/images/OS-2](/assets/images/OS-2.png)

![/assets/images/OS-3](/assets/images/OS-3.png) 

![/assets/images/OS-4](/assets/images/OS-4.png) 


![/assets/images/OS-1](/assets/images/OS-5.png) 

![/assets/images/OS-2](/assets/images/OS-6.png)

![/assets/images/OS-3](/assets/images/OS-7.png) 

![/assets/images/OS-4](/assets/images/OS-8.png) 


![/assets/images/OS-1](/assets/images/OS-9.png) 

![/assets/images/OS-2](/assets/images/OS-10.png)

![/assets/images/OS-3](/assets/images/OS-11.png) 

![/assets/images/OS-4](/assets/images/OS-12.png) 


![/assets/images/OS-1](/assets/images/OS-13.png) 

![/assets/images/OS-2](/assets/images/OS-14.png)

![/assets/images/OS-3](/assets/images/OS-15.png) 

![/assets/images/OS-4](/assets/images/OS-16.png)  


```sh
pradeep@learnOpenShift$ oc login --token=sha256~pWLPko6xkJ71k00adi-k3pyXCVr4TJrIs5l8muLOUnk --server=https://api.crc.testing:6443
Logged into "https://api.crc.testing:6443" as "developer" using the token provided.

You have one project on this server: "pradeep-os-demo"

Using project "pradeep-os-demo".
```

```sh
pradeep@learnOpenShift$ oc get all
NAME                                READY   STATUS      RESTARTS   AGE
pod/pradeep-cake-1-build            0/1     Completed   0          40m
pod/pradeep-cake-64796f74fd-mmh6p   1/1     Running     0          9m3s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/pradeep-cake   ClusterIP   10.217.5.152   <none>        8080/TCP,8443/TCP   40m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pradeep-cake   1/1     1            1           40m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/pradeep-cake-64796f74fd   1         1         1       9m3s
replicaset.apps/pradeep-cake-95f4d7758    0         0         0       40m

NAME                                          TYPE     FROM   LATEST
buildconfig.build.openshift.io/pradeep-cake   Source   Git    2

NAME                                      TYPE     FROM          STATUS     STARTED          DURATION
build.build.openshift.io/pradeep-cake-1   Source   Git@ba4b1cd   Complete   40 minutes ago   31m38s

NAME                                          IMAGE REPOSITORY                                                                       TAGS     UPDATED
imagestream.image.openshift.io/pradeep-cake   default-route-openshift-image-registry.apps-crc.testing/pradeep-os-demo/pradeep-cake   latest   9 minutes ago

NAME                                    HOST/PORT                                       PATH   SERVICES       PORT       TERMINATION   WILDCARD
route.route.openshift.io/pradeep-cake   pradeep-cake-pradeep-os-demo.apps-crc.testing          pradeep-cake   8080-tcp                 None
```



## Switching to Administrator Persona

![](/assets/images/OS-17.png) 

![/assets/images/OS-2](/assets/images/OS-18.png)

![/assets/images/OS-3](/assets/images/OS-19.png) 

![/assets/images/OS-4](/assets/images/OS-20.png) 

![/assets/images/OS-4](/assets/images/OS-21.png) 

After deleting the pod from failed build 

```sh
pradeep@learnOpenShift$ oc get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/pradeep-cake-64796f74fd-mmh6p   1/1     Running   0          13m

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/pradeep-cake   ClusterIP   10.217.5.152   <none>        8080/TCP,8443/TCP   45m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pradeep-cake   1/1     1            1           45m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/pradeep-cake-64796f74fd   1         1         1       13m
replicaset.apps/pradeep-cake-95f4d7758    0         0         0       45m

NAME                                          TYPE     FROM   LATEST
buildconfig.build.openshift.io/pradeep-cake   Source   Git    2

NAME                                      TYPE     FROM          STATUS     STARTED          DURATION
build.build.openshift.io/pradeep-cake-1   Source   Git@ba4b1cd   Complete   45 minutes ago   31m38s

NAME                                          IMAGE REPOSITORY                                                                       TAGS     UPDATED
imagestream.image.openshift.io/pradeep-cake   default-route-openshift-image-registry.apps-crc.testing/pradeep-os-demo/pradeep-cake   latest   13 minutes ago

NAME                                    HOST/PORT                                       PATH   SERVICES       PORT       TERMINATION   WILDCARD
route.route.openshift.io/pradeep-cake   pradeep-cake-pradeep-os-demo.apps-crc.testing          pradeep-cake   8080-tcp                 None
```

```sh
pradeep@learnOpenShift$ oc projects
You have one project on this server: "pradeep-os-demo".

Using project "pradeep-os-demo" on server "https://api.crc.testing:6443".
```

```sh
pradeep@learnOpenShift$ oc get pods
NAME                            READY   STATUS    RESTARTS   AGE
pradeep-cake-64796f74fd-mmh6p   1/1     Running   0          18m
```

```sh
pradeep@learnOpenShift$ oc get nodes
Error from server (Forbidden): nodes is forbidden: User "developer" cannot list resource "nodes" in API group "" at the cluster scope
```

```sh
pradeep@learnOpenShift$ oc get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
pradeep-cake   1/1     1            1           50m
```

```sh
pradeep@learnOpenShift$ oc get svc
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
pradeep-cake   ClusterIP   10.217.5.152   <none>        8080/TCP,8443/TCP   50m
```

```sh
pradeep@learnOpenShift$ oc describe pod pradeep-cake-64796f74fd-mmh6p
Name:         pradeep-cake-64796f74fd-mmh6p
Namespace:    pradeep-os-demo
Priority:     0
Node:         crc-xxcfw-master-0/192.168.126.11
Start Time:   Tue, 22 Feb 2022 20:54:31 +0530
Labels:       app=pradeep-cake
              deploymentconfig=pradeep-cake
              pod-template-hash=64796f74fd
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "openshift-sdn",
                    "interface": "eth0",
                    "ips": [
                        "10.217.0.68"
                    ],
                    "default": true,
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks-status:
                [{
                    "name": "openshift-sdn",
                    "interface": "eth0",
                    "ips": [
                        "10.217.0.68"
                    ],
                    "default": true,
                    "dns": {}
                }]
              openshift.io/scc: restricted
Status:       Running
IP:           10.217.0.68
IPs:
  IP:           10.217.0.68
Controlled By:  ReplicaSet/pradeep-cake-64796f74fd
Containers:
  pradeep-cake:
    Container ID:   cri-o://1801d269a5ba5c12a1228678fe09452e670f94944dc91003d3d0fe38f3d9f4d8
    Image:          image-registry.openshift-image-registry.svc:5000/pradeep-os-demo/pradeep-cake@sha256:66900b89f25f97553f3fe81f91354d4fd1746d0265ad135b34acffbf211b62ef
    Image ID:       image-registry.openshift-image-registry.svc:5000/pradeep-os-demo/pradeep-cake@sha256:66900b89f25f97553f3fe81f91354d4fd1746d0265ad135b34acffbf211b62ef
    Ports:          8080/TCP, 8443/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Tue, 22 Feb 2022 20:56:37 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-789n7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-789n7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
    ConfigMapName:           openshift-service-ca.crt
    ConfigMapOptional:       <nil>
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason          Age   From               Message
  ----    ------          ----  ----               -------
  Normal  Scheduled       19m   default-scheduler  Successfully assigned pradeep-os-demo/pradeep-cake-64796f74fd-mmh6p to crc-xxcfw-master-0
  Normal  AddedInterface  19m   multus             Add eth0 [10.217.0.68/23] from openshift-sdn
  Normal  Pulling         19m   kubelet            Pulling image "image-registry.openshift-image-registry.svc:5000/pradeep-os-demo/pradeep-cake@sha256:66900b89f25f97553f3fe81f91354d4fd1746d0265ad135b34acffbf211b62ef"
  Normal  Pulled          17m   kubelet            Successfully pulled image "image-registry.openshift-image-registry.svc:5000/pradeep-os-demo/pradeep-cake@sha256:66900b89f25f97553f3fe81f91354d4fd1746d0265ad135b34acffbf211b62ef" in 2m1.585839677s
  Normal  Created         17m   kubelet            Created container pradeep-cake
  Normal  Started         17m   kubelet            Started container pradeep-cake
```

```sh
pradeep@learnOpenShift$ oc api-resources
NAME                                  SHORTNAMES       APIVERSION                                    NAMESPACED   KIND
bindings                                               v1                                            true         Binding
componentstatuses                     cs               v1                                            false        ComponentStatus
configmaps                            cm               v1                                            true         ConfigMap
endpoints                             ep               v1                                            true         Endpoints
events                                ev               v1                                            true         Event
limitranges                           limits           v1                                            true         LimitRange
namespaces                            ns               v1                                            false        Namespace
nodes                                 no               v1                                            false        Node
persistentvolumeclaims                pvc              v1                                            true         PersistentVolumeClaim
persistentvolumes                     pv               v1                                            false        PersistentVolume
pods                                  po               v1                                            true         Pod
podtemplates                                           v1                                            true         PodTemplate
replicationcontrollers                rc               v1                                            true         ReplicationController
resourcequotas                        quota            v1                                            true         ResourceQuota
secrets                                                v1                                            true         Secret
serviceaccounts                       sa               v1                                            true         ServiceAccount
services                              svc              v1                                            true         Service
mutatingwebhookconfigurations                          admissionregistration.k8s.io/v1               false        MutatingWebhookConfiguration
validatingwebhookconfigurations                        admissionregistration.k8s.io/v1               false        ValidatingWebhookConfiguration
customresourcedefinitions             crd,crds         apiextensions.k8s.io/v1                       false        CustomResourceDefinition
apiservices                                            apiregistration.k8s.io/v1                     false        APIService
apirequestcounts                                       apiserver.openshift.io/v1                     false        APIRequestCount
controllerrevisions                                    apps/v1                                       true         ControllerRevision
daemonsets                            ds               apps/v1                                       true         DaemonSet
deployments                           deploy           apps/v1                                       true         Deployment
replicasets                           rs               apps/v1                                       true         ReplicaSet
statefulsets                          sts              apps/v1                                       true         StatefulSet
deploymentconfigs                     dc               apps.openshift.io/v1                          true         DeploymentConfig
tokenreviews                                           authentication.k8s.io/v1                      false        TokenReview
localsubjectaccessreviews                              authorization.k8s.io/v1                       true         LocalSubjectAccessReview
selfsubjectaccessreviews                               authorization.k8s.io/v1                       false        SelfSubjectAccessReview
selfsubjectrulesreviews                                authorization.k8s.io/v1                       false        SelfSubjectRulesReview
subjectaccessreviews                                   authorization.k8s.io/v1                       false        SubjectAccessReview
clusterrolebindings                                    authorization.openshift.io/v1                 false        ClusterRoleBinding
clusterroles                                           authorization.openshift.io/v1                 false        ClusterRole
localresourceaccessreviews                             authorization.openshift.io/v1                 true         LocalResourceAccessReview
localsubjectaccessreviews                              authorization.openshift.io/v1                 true         LocalSubjectAccessReview
resourceaccessreviews                                  authorization.openshift.io/v1                 false        ResourceAccessReview
rolebindingrestrictions                                authorization.openshift.io/v1                 true         RoleBindingRestriction
rolebindings                                           authorization.openshift.io/v1                 true         RoleBinding
roles                                                  authorization.openshift.io/v1                 true         Role
selfsubjectrulesreviews                                authorization.openshift.io/v1                 true         SelfSubjectRulesReview
subjectaccessreviews                                   authorization.openshift.io/v1                 false        SubjectAccessReview
subjectrulesreviews                                    authorization.openshift.io/v1                 true         SubjectRulesReview
horizontalpodautoscalers              hpa              autoscaling/v1                                true         HorizontalPodAutoscaler
clusterautoscalers                    ca               autoscaling.openshift.io/v1                   false        ClusterAutoscaler
machineautoscalers                    ma               autoscaling.openshift.io/v1beta1              true         MachineAutoscaler
cronjobs                              cj               batch/v1                                      true         CronJob
jobs                                                   batch/v1                                      true         Job
buildconfigs                          bc               build.openshift.io/v1                         true         BuildConfig
builds                                                 build.openshift.io/v1                         true         Build
certificatesigningrequests            csr              certificates.k8s.io/v1                        false        CertificateSigningRequest
credentialsrequests                                    cloudcredential.openshift.io/v1               true         CredentialsRequest
apiservers                                             config.openshift.io/v1                        false        APIServer
authentications                                        config.openshift.io/v1                        false        Authentication
builds                                                 config.openshift.io/v1                        false        Build
clusteroperators                      co               config.openshift.io/v1                        false        ClusterOperator
clusterversions                                        config.openshift.io/v1                        false        ClusterVersion
consoles                                               config.openshift.io/v1                        false        Console
dnses                                                  config.openshift.io/v1                        false        DNS
featuregates                                           config.openshift.io/v1                        false        FeatureGate
images                                                 config.openshift.io/v1                        false        Image
infrastructures                                        config.openshift.io/v1                        false        Infrastructure
ingresses                                              config.openshift.io/v1                        false        Ingress
networks                                               config.openshift.io/v1                        false        Network
oauths                                                 config.openshift.io/v1                        false        OAuth
operatorhubs                                           config.openshift.io/v1                        false        OperatorHub
projects                                               config.openshift.io/v1                        false        Project
proxies                                                config.openshift.io/v1                        false        Proxy
schedulers                                             config.openshift.io/v1                        false        Scheduler
consoleclidownloads                                    console.openshift.io/v1                       false        ConsoleCLIDownload
consoleexternalloglinks                                console.openshift.io/v1                       false        ConsoleExternalLogLink
consolelinks                                           console.openshift.io/v1                       false        ConsoleLink
consolenotifications                                   console.openshift.io/v1                       false        ConsoleNotification
consoleplugins                                         console.openshift.io/v1alpha1                 false        ConsolePlugin
consolequickstarts                                     console.openshift.io/v1                       false        ConsoleQuickStart
consoleyamlsamples                                     console.openshift.io/v1                       false        ConsoleYAMLSample
podnetworkconnectivitychecks                           controlplane.operator.openshift.io/v1alpha1   true         PodNetworkConnectivityCheck
leases                                                 coordination.k8s.io/v1                        true         Lease
endpointslices                                         discovery.k8s.io/v1                           true         EndpointSlice
events                                ev               events.k8s.io/v1                              true         Event
flowschemas                                            flowcontrol.apiserver.k8s.io/v1beta1          false        FlowSchema
prioritylevelconfigurations                            flowcontrol.apiserver.k8s.io/v1beta1          false        PriorityLevelConfiguration
helmchartrepositories                                  helm.openshift.io/v1beta1                     false        HelmChartRepository
images                                                 image.openshift.io/v1                         false        Image
imagesignatures                                        image.openshift.io/v1                         false        ImageSignature
imagestreamimages                     isimage          image.openshift.io/v1                         true         ImageStreamImage
imagestreamimports                                     image.openshift.io/v1                         true         ImageStreamImport
imagestreammappings                                    image.openshift.io/v1                         true         ImageStreamMapping
imagestreams                          is               image.openshift.io/v1                         true         ImageStream
imagestreamtags                       istag            image.openshift.io/v1                         true         ImageStreamTag
imagetags                             itag             image.openshift.io/v1                         true         ImageTag
configs                                                imageregistry.operator.openshift.io/v1        false        Config
imagepruners                                           imageregistry.operator.openshift.io/v1        false        ImagePruner
dnsrecords                                             ingress.operator.openshift.io/v1              true         DNSRecord
network-attachment-definitions        net-attach-def   k8s.cni.cncf.io/v1                            true         NetworkAttachmentDefinition
machinehealthchecks                   mhc,mhcs         machine.openshift.io/v1beta1                  true         MachineHealthCheck
machines                                               machine.openshift.io/v1beta1                  true         Machine
machinesets                                            machine.openshift.io/v1beta1                  true         MachineSet
containerruntimeconfigs               ctrcfg           machineconfiguration.openshift.io/v1          false        ContainerRuntimeConfig
controllerconfigs                                      machineconfiguration.openshift.io/v1          false        ControllerConfig
kubeletconfigs                                         machineconfiguration.openshift.io/v1          false        KubeletConfig
machineconfigpools                    mcp              machineconfiguration.openshift.io/v1          false        MachineConfigPool
machineconfigs                        mc               machineconfiguration.openshift.io/v1          false        MachineConfig
baremetalhosts                        bmh,bmhost       metal3.io/v1alpha1                            true         BareMetalHost
provisionings                                          metal3.io/v1alpha1                            false        Provisioning
storagestates                                          migration.k8s.io/v1alpha1                     false        StorageState
storageversionmigrations                               migration.k8s.io/v1alpha1                     false        StorageVersionMigration
alertmanagerconfigs                                    monitoring.coreos.com/v1alpha1                true         AlertmanagerConfig
alertmanagers                                          monitoring.coreos.com/v1                      true         Alertmanager
podmonitors                                            monitoring.coreos.com/v1                      true         PodMonitor
probes                                                 monitoring.coreos.com/v1                      true         Probe
prometheuses                                           monitoring.coreos.com/v1                      true         Prometheus
prometheusrules                                        monitoring.coreos.com/v1                      true         PrometheusRule
servicemonitors                                        monitoring.coreos.com/v1                      true         ServiceMonitor
thanosrulers                                           monitoring.coreos.com/v1                      true         ThanosRuler
clusternetworks                                        network.openshift.io/v1                       false        ClusterNetwork
egressnetworkpolicies                                  network.openshift.io/v1                       true         EgressNetworkPolicy
hostsubnets                                            network.openshift.io/v1                       false        HostSubnet
netnamespaces                                          network.openshift.io/v1                       false        NetNamespace
egressrouters                                          network.operator.openshift.io/v1              true         EgressRouter
operatorpkis                                           network.operator.openshift.io/v1              true         OperatorPKI
ingressclasses                                         networking.k8s.io/v1                          false        IngressClass
ingresses                             ing              networking.k8s.io/v1                          true         Ingress
networkpolicies                       netpol           networking.k8s.io/v1                          true         NetworkPolicy
runtimeclasses                                         node.k8s.io/v1                                false        RuntimeClass
oauthaccesstokens                                      oauth.openshift.io/v1                         false        OAuthAccessToken
oauthauthorizetokens                                   oauth.openshift.io/v1                         false        OAuthAuthorizeToken
oauthclientauthorizations                              oauth.openshift.io/v1                         false        OAuthClientAuthorization
oauthclients                                           oauth.openshift.io/v1                         false        OAuthClient
tokenreviews                                           oauth.openshift.io/v1                         false        TokenReview
useroauthaccesstokens                                  oauth.openshift.io/v1                         false        UserOAuthAccessToken
authentications                                        operator.openshift.io/v1                      false        Authentication
cloudcredentials                                       operator.openshift.io/v1                      false        CloudCredential
clustercsidrivers                                      operator.openshift.io/v1                      false        ClusterCSIDriver
configs                                                operator.openshift.io/v1                      false        Config
consoles                                               operator.openshift.io/v1                      false        Console
csisnapshotcontrollers                                 operator.openshift.io/v1                      false        CSISnapshotController
dnses                                                  operator.openshift.io/v1                      false        DNS
etcds                                                  operator.openshift.io/v1                      false        Etcd
imagecontentsourcepolicies                             operator.openshift.io/v1alpha1                false        ImageContentSourcePolicy
ingresscontrollers                                     operator.openshift.io/v1                      true         IngressController
kubeapiservers                                         operator.openshift.io/v1                      false        KubeAPIServer
kubecontrollermanagers                                 operator.openshift.io/v1                      false        KubeControllerManager
kubeschedulers                                         operator.openshift.io/v1                      false        KubeScheduler
kubestorageversionmigrators                            operator.openshift.io/v1                      false        KubeStorageVersionMigrator
networks                                               operator.openshift.io/v1                      false        Network
openshiftapiservers                                    operator.openshift.io/v1                      false        OpenShiftAPIServer
openshiftcontrollermanagers                            operator.openshift.io/v1                      false        OpenShiftControllerManager
servicecas                                             operator.openshift.io/v1                      false        ServiceCA
storages                                               operator.openshift.io/v1                      false        Storage
catalogsources                        catsrc           operators.coreos.com/v1alpha1                 true         CatalogSource
clusterserviceversions                csv,csvs         operators.coreos.com/v1alpha1                 true         ClusterServiceVersion
installplans                          ip               operators.coreos.com/v1alpha1                 true         InstallPlan
operatorconditions                    condition        operators.coreos.com/v2                       true         OperatorCondition
operatorgroups                        og               operators.coreos.com/v1                       true         OperatorGroup
operators                                              operators.coreos.com/v1                       false        Operator
subscriptions                         sub,subs         operators.coreos.com/v1alpha1                 true         Subscription
packagemanifests                                       packages.operators.coreos.com/v1              true         PackageManifest
poddisruptionbudgets                  pdb              policy/v1                                     true         PodDisruptionBudget
podsecuritypolicies                   psp              policy/v1beta1                                false        PodSecurityPolicy
projectrequests                                        project.openshift.io/v1                       false        ProjectRequest
projects                                               project.openshift.io/v1                       false        Project
appliedclusterresourcequotas                           quota.openshift.io/v1                         true         AppliedClusterResourceQuota
clusterresourcequotas                 clusterquota     quota.openshift.io/v1                         false        ClusterResourceQuota
clusterrolebindings                                    rbac.authorization.k8s.io/v1                  false        ClusterRoleBinding
clusterroles                                           rbac.authorization.k8s.io/v1                  false        ClusterRole
rolebindings                                           rbac.authorization.k8s.io/v1                  true         RoleBinding
roles                                                  rbac.authorization.k8s.io/v1                  true         Role
routes                                                 route.openshift.io/v1                         true         Route
configs                                                samples.operator.openshift.io/v1              false        Config
priorityclasses                       pc               scheduling.k8s.io/v1                          false        PriorityClass
rangeallocations                                       security.internal.openshift.io/v1             false        RangeAllocation
podsecuritypolicyreviews                               security.openshift.io/v1                      true         PodSecurityPolicyReview
podsecuritypolicyselfsubjectreviews                    security.openshift.io/v1                      true         PodSecurityPolicySelfSubjectReview
podsecuritypolicysubjectreviews                        security.openshift.io/v1                      true         PodSecurityPolicySubjectReview
rangeallocations                                       security.openshift.io/v1                      false        RangeAllocation
securitycontextconstraints            scc              security.openshift.io/v1                      false        SecurityContextConstraints
csidrivers                                             storage.k8s.io/v1                             false        CSIDriver
csinodes                                               storage.k8s.io/v1                             false        CSINode
csistoragecapacities                                   storage.k8s.io/v1beta1                        true         CSIStorageCapacity
storageclasses                        sc               storage.k8s.io/v1                             false        StorageClass
volumeattachments                                      storage.k8s.io/v1                             false        VolumeAttachment
brokertemplateinstances                                template.openshift.io/v1                      false        BrokerTemplateInstance
processedtemplates                                     template.openshift.io/v1                      true         Template
templateinstances                                      template.openshift.io/v1                      true         TemplateInstance
templates                                              template.openshift.io/v1                      true         Template
profiles                                               tuned.openshift.io/v1                         true         Profile
tuneds                                                 tuned.openshift.io/v1                         true         Tuned
groups                                                 user.openshift.io/v1                          false        Group
identities                                             user.openshift.io/v1                          false        Identity
useridentitymappings                                   user.openshift.io/v1                          false        UserIdentityMapping
users                                                  user.openshift.io/v1                          false        User
ippools                                                whereabouts.cni.cncf.io/v1alpha1              true         IPPool
overlappingrangeipreservations                         whereabouts.cni.cncf.io/v1alpha1              true         OverlappingRangeIPReservation
```



## Setting up an Application using oc new-app

```sh
pradeep@learnOpenShift$ oc whoami
developer
```

```sh
pradeep@learnOpenShift$ oc new-project mysql
Now using project "mysql" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname

```



```sh
pradeep@learnOpenShift$ oc new-app --docker-image=mysql:latest --name=mysql-openshift -e MYSQL_USER=myuser -e MYSQL_PASSWORD=password -e
MYSQL_DATABASE=mydb -e MYSQL_ROOT_PASSWORD=password
Flag --docker-image has been deprecated, Deprecated flag use --image
--> Found container image 17b062d (5 days old) from Docker Hub for "mysql:latest"

    * An image stream tag will be created as "mysql-openshift:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "mysql-openshift" created
    deployment.apps "mysql-openshift" created
    service "mysql-openshift" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/mysql-openshift'
    Run 'oc status' to view your app.
```



```sh
pradeep@learnOpenShift$ oc status
In project mysql on server https://api.crc.testing:6443

svc/mysql-openshift - 10.217.4.70 ports 3306, 33060
  deployment/mysql-openshift deploys istag/mysql-openshift:latest
    deployment #2 running for about a minute - 0/1 pods
    deployment #1 deployed 2 minutes ago - 0/1 pods growing to 1


1 info identified, use 'oc status --suggest' to see details.
```

```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running   0          8m43s

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70   <none>        3306/TCP,33060/TCP   8m46s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           8m46s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       8m46s
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       8m43s

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   8 minutes ago
```

```sh
pradeep@learnOpenShift$ oc get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
mysql-openshift-f5b68568f-shwz7   1/1     Running   0          9m19s   10.217.0.64   crc-xxcfw-master-0   <none>           <none>
```

Log in to the web console and see the new application in Project: mysql > Overview



![/assets/images/OS-22](/assets/images/OS-22.png)





![](/assets/images/OS-23.png)



```sh
pradeep@learnOpenShift$ oc get events
LAST SEEN   TYPE      REASON              OBJECT                                  MESSAGE
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-n8wtz" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-8kv89" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-fl2b8" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-wskrp" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-kgncj" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-4c8dz" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-r5ksg" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-pc94d" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   Error creating: Pod "mysql-openshift-68996c4c8c-n2gkp" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
11m         Warning   FailedCreate        replicaset/mysql-openshift-68996c4c8c   (combined from similar events): Error creating: Pod "mysql-openshift-68996c4c8c-5pkcx" is invalid: spec.containers[0].image: Invalid value: " ": must not have leading or trailing whitespace
12m         Normal    Scheduled           pod/mysql-openshift-f5b68568f-shwz7     Successfully assigned mysql/mysql-openshift-f5b68568f-shwz7 to crc-xxcfw-master-0
12m         Normal    AddedInterface      pod/mysql-openshift-f5b68568f-shwz7     Add eth0 [10.217.0.64/23] from openshift-sdn
10m         Normal    Pulling             pod/mysql-openshift-f5b68568f-shwz7     Pulling image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f"
11m         Warning   Failed              pod/mysql-openshift-f5b68568f-shwz7     Failed to pull image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f": rpc error: code = Unknown desc = reading blob sha256:5d726bac08eac44e8c39b1afbe20d14e2288e6846dd17299ae5725de86124bc7: Get "https://registry-1.docker.io/v2/library/mysql/blobs/sha256:5d726bac08eac44e8c39b1afbe20d14e2288e6846dd17299ae5725de86124bc7": net/http: TLS handshake timeout
11m         Warning   Failed              pod/mysql-openshift-f5b68568f-shwz7     Error: ErrImagePull
11m         Normal    BackOff             pod/mysql-openshift-f5b68568f-shwz7     Back-off pulling image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f"
11m         Warning   Failed              pod/mysql-openshift-f5b68568f-shwz7     Error: ImagePullBackOff
9m31s       Normal    Pulled              pod/mysql-openshift-f5b68568f-shwz7     Successfully pulled image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f" in 1m17.121697748s
9m31s       Normal    Created             pod/mysql-openshift-f5b68568f-shwz7     Created container mysql-openshift
9m31s       Normal    Started             pod/mysql-openshift-f5b68568f-shwz7     Started container mysql-openshift
12m         Normal    SuccessfulCreate    replicaset/mysql-openshift-f5b68568f    Created pod: mysql-openshift-f5b68568f-shwz7
12m         Normal    ScalingReplicaSet   deployment/mysql-openshift              Scaled up replica set mysql-openshift-68996c4c8c to 1
12m         Normal    ScalingReplicaSet   deployment/mysql-openshift              Scaled up replica set mysql-openshift-f5b68568f to 1
9m30s       Normal    ScalingReplicaSet   deployment/mysql-openshift              Scaled down replica set mysql-openshift-68996c4c8c to 0
```



```sh
pradeep@learnOpenShift$ oc logs pod/mysql-openshift-f5b68568f-shwz7
2022-02-23 04:10:46+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.28-1debian10 started.
2022-02-23 04:10:47+00:00 [Note] [Entrypoint]: Initializing database files
2022-02-23T04:10:47.109360Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.28) initializing of server in progress as process 23
2022-02-23T04:10:47.119625Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-02-23T04:10:49.742825Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-02-23T04:10:51.821227Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2022-02-23 04:10:56+00:00 [Note] [Entrypoint]: Database files initialized
2022-02-23 04:10:56+00:00 [Note] [Entrypoint]: Starting temporary server
2022-02-23T04:10:58.380929Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.28) starting as process 72
2022-02-23T04:10:58.577032Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-02-23T04:10:59.882536Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-02-23T04:11:00.781991Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2022-02-23T04:11:00.782167Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2022-02-23T04:11:00.784296Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2022-02-23T04:11:00.851759Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.28'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
2022-02-23T04:11:00.855887Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
2022-02-23 04:11:00+00:00 [Note] [Entrypoint]: Temporary server started.
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
2022-02-23 04:11:07+00:00 [Note] [Entrypoint]: Creating database mydb
2022-02-23 04:11:07+00:00 [Note] [Entrypoint]: Creating user myuser
2022-02-23 04:11:07+00:00 [Note] [Entrypoint]: Giving user myuser access to schema mydb

2022-02-23 04:11:07+00:00 [Note] [Entrypoint]: Stopping temporary server
2022-02-23T04:11:07.421551Z 13 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.28).
2022-02-23T04:11:09.892006Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.28)  MySQL Community Server - GPL.
2022-02-23 04:11:10+00:00 [Note] [Entrypoint]: Temporary server stopped

2022-02-23 04:11:10+00:00 [Note] [Entrypoint]: MySQL init process done. Ready for start up.

2022-02-23T04:11:10.813524Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.28) starting as process 1
2022-02-23T04:11:10.841138Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-02-23T04:11:11.181491Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-02-23T04:11:11.468439Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2022-02-23T04:11:11.468601Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2022-02-23T04:11:11.478843Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2022-02-23T04:11:11.506637Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2022-02-23T04:11:11.506839Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.28'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```



```sh
pradeep@learnOpenShift$ oc describe pod mysql-openshift-f5b68568f-shwz
Name:         mysql-openshift-f5b68568f-shwz7
Namespace:    mysql
Priority:     0
Node:         crc-xxcfw-master-0/192.168.126.11
Start Time:   Wed, 23 Feb 2022 09:37:25 +0530
Labels:       deployment=mysql-openshift
              pod-template-hash=f5b68568f
Annotations:  k8s.v1.cni.cncf.io/network-status:
                [{
                    "name": "openshift-sdn",
                    "interface": "eth0",
                    "ips": [
                        "10.217.0.64"
                    ],
                    "default": true,
                    "dns": {}
                }]
              k8s.v1.cni.cncf.io/networks-status:
                [{
                    "name": "openshift-sdn",
                    "interface": "eth0",
                    "ips": [
                        "10.217.0.64"
                    ],
                    "default": true,
                    "dns": {}
                }]
              openshift.io/generated-by: OpenShiftNewApp
              openshift.io/scc: restricted
Status:       Running
IP:           10.217.0.64
IPs:
  IP:           10.217.0.64
Controlled By:  ReplicaSet/mysql-openshift-f5b68568f
Containers:
  mysql-openshift:
    Container ID:   cri-o://bd324439c67da1c3efd52709d4718b54f6e0619c0540c7ca1f8d59a61984dceb
    Image:          mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f
    Image ID:       docker.io/library/mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f
    Ports:          3306/TCP, 33060/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Wed, 23 Feb 2022 09:40:46 +0530
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_DATABASE:       mydb
      MYSQL_PASSWORD:       password
      MYSQL_ROOT_PASSWORD:  password
      MYSQL_USER:           myuser
    Mounts:
      /var/lib/mysql from mysql-openshift-volume-1 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mqqkz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  mysql-openshift-volume-1:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-mqqkz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
    ConfigMapName:           openshift-service-ca.crt
    ConfigMapOptional:       <nil>
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason          Age                From               Message
  ----     ------          ----               ----               -------
  Normal   Scheduled       17m                default-scheduler  Successfully assigned mysql/mysql-openshift-f5b68568f-shwz7 to crc-xxcfw-master-0
  Normal   AddedInterface  17m                multus             Add eth0 [10.217.0.64/23] from openshift-sdn
  Warning  Failed          16m                kubelet            Failed to pull image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f": rpc error: code = Unknown desc = reading blob sha256:5d726bac08eac44e8c39b1afbe20d14e2288e6846dd17299ae5725de86124bc7: Get "https://registry-1.docker.io/v2/library/mysql/blobs/sha256:5d726bac08eac44e8c39b1afbe20d14e2288e6846dd17299ae5725de86124bc7": net/http: TLS handshake timeout
  Warning  Failed          16m                kubelet            Error: ErrImagePull
  Normal   BackOff         16m                kubelet            Back-off pulling image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f"
  Warning  Failed          16m                kubelet            Error: ImagePullBackOff
  Normal   Pulling         15m (x2 over 17m)  kubelet            Pulling image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f"
  Normal   Pulled          14m                kubelet            Successfully pulled image "mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f" in 1m17.121697748s
  Normal   Created         14m                kubelet            Created container mysql-openshift
  Normal   Started         14m                kubelet            Started container mysql-openshift
```



```sh
pradeep@learnOpenShift$ oc projects
You have access to the following projects and can switch between them with ' project <projectname>':

  * mysql
    pradeep-os-demo

Using project "mysql" on server "https://api.crc.testing:6443".
```

Let us go back to our mysql application and expose it, to create a route.

```sh
pradeep@learnOpenShift$ oc expose service/mysql-openshift
route.route.openshift.io/mysql-openshift exposed
```

```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running   0          20m

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70   <none>        3306/TCP,33060/TCP   20m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           20m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       20m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       20m

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   20 minutes ago

NAME                                       HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
```



![/assets/images/OS-24](/assets/images/OS-24.png) 

Click on Open URL

![/assets/images/OS-25](/assets/images/OS-25.png) 





## OpenShift Building Blocks

A project is a Kubernetes  namespace that contains all resources running  in 
the OpenShift  application.

```sh
pradeep@learnOpenShift$ oc config get-contexts
CURRENT   NAME                                             CLUSTER                AUTHINFO                         NAMESPACE
          /api-crc-testing:6443/developer                  api-crc-testing:6443   developer/api-crc-testing:6443
          crc-admin                                        api-crc-testing:6443   kubeadmin                        default
          crc-developer                                    api-crc-testing:6443   developer                        default
          k8s                                              k8s                    k8s                              default
*         mysql/api-crc-testing:6443/developer             api-crc-testing:6443   developer/api-crc-testing:6443   mysql
          pradeep                                          k8s                    pradeep
          pradeep-os-demo/api-crc-testing:6443/developer   api-crc-testing:6443   developer/api-crc-testing:6443   pradeep-os-demo
```

```sh
pradeep@learnOpenShift$ oc get pods --show-labels
NAME                              READY   STATUS    RESTARTS   AGE   LABELS
mysql-openshift-f5b68568f-shwz7   1/1     Running   0          32m   deployment=mysql-openshift,pod-template-hash=f5b68568f
```
```sh
pradeep@learnOpenShift$ oc describe rs mysql-openshift-f5b68568f
Name:           mysql-openshift-f5b68568f
Namespace:      mysql
Selector:       deployment=mysql-openshift,pod-template-hash=f5b68568f
Labels:         deployment=mysql-openshift
                pod-template-hash=f5b68568f
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 2
                image.openshift.io/triggers:
                  [{"from":{"kind":"ImageStreamTag","name":"mysql-openshift:latest"},"fieldPath":"spec.template.spec.containers[?(@.name==\"mysql-openshift\...
                openshift.io/generated-by: OpenShiftNewApp
Controlled By:  Deployment/mysql-openshift
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       deployment=mysql-openshift
                pod-template-hash=f5b68568f
  Annotations:  openshift.io/generated-by: OpenShiftNewApp
  Containers:
   mysql-openshift:
    Image:       mysql@sha256:e9c9e3680bbadd5230a62c5548793bd8e59cbcc868032781e48bd53e888bd82f
    Ports:       3306/TCP, 33060/TCP
    Host Ports:  0/TCP, 0/TCP
    Environment:
      MYSQL_DATABASE:       mydb
      MYSQL_PASSWORD:       password
      MYSQL_ROOT_PASSWORD:  password
      MYSQL_USER:           myuser
    Mounts:
      /var/lib/mysql from mysql-openshift-volume-1 (rw)
  Volumes:
   mysql-openshift-volume-1:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  33m   replicaset-controller  Created pod: mysql-openshift-f5b68568f-shwz7
```



```sh
pradeep@learnOpenShift$ oc get all --selector deployment=mysql-openshift
NAME                                  READY   STATUS    RESTARTS   AGE
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running   0          35m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       35m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       35m
```



```sh
pradeep@learnOpenShift$ oc explain pods
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```



## Creating a Pod from YAML files

```sh
pradeep@learnOpenShift$ oc run bnginx --image=bitnami/nginx --dry-run=client -o yaml > bnginx.yaml
```



```yaml
pradeep@learnOpenShift$ cat bnginx.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bnginx
  name: bnginx
spec:
  containers:
  - image: bitnami/nginx
    name: bnginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```sh
pradeep@learnOpenShift$ oc create -f bnginx.yaml
pod/bnginx created
```

```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/bnginx                            1/1     Running   0          79s
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running   0          40m

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70   <none>        3306/TCP,33060/TCP   40m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           40m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       40m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       40m

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   40 minutes ago

NAME                                       HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
```



```sh
pradeep@learnOpenShift$ oc -it exec bnginx -- sh
$ uname -a
Linux bnginx 4.18.0-305.30.1.el8_4.x86_64 #1 SMP Tue Nov 30 13:13:11 EST 2021 x86_64 GNU/Linux
$ exit 0
```



## Using Source-to-Image to Create Applications



To create an Image, a Dockerfile could be used.
 Source 2 Image (S2I) takes application source code from a source control 
repository  (such as Git) and builds  a base container based on that to run 
the application. Also while doing so, the image is pushed to the OpenShift registry.



### Images and Image Streams

An Image is a runtime  template that contains all data that is needed  to run  a container

An Image Stream is a collection of  different versions that exist of an image

Builder Image provide  specific programming  language information  that is 
required  to create an application

Please note `is` is a short name for `image stream`.

```sh
pradeep@learnOpenShift$ oc get is -n openshift
NAME                                                  IMAGE REPOSITORY                                                                                                        TAGS                                                     UPDATED
apicast-gateway                                       default-route-openshift-image-registry.apps-crc.testing/openshift/apicast-gateway                                       2.1.0.GA,2.10.0.GA,2.2.0.GA,2.3.0.GA + 7 more...         6 weeks ago
apicurito-ui                                          default-route-openshift-image-registry.apps-crc.testing/openshift/apicurito-ui                                          1.2,1.3,1.4,1.5,1.6,1.7,1.8                              6 weeks ago
cli                                                   default-route-openshift-image-registry.apps-crc.testing/openshift/cli                                                   latest                                                   6 weeks ago
cli-artifacts                                         default-route-openshift-image-registry.apps-crc.testing/openshift/cli-artifacts                                         latest                                                   6 weeks ago
dotnet                                                default-route-openshift-image-registry.apps-crc.testing/openshift/dotnet                                                2.1,2.1-el7,2.1-ubi8,3.1,3.1-el7 + 4 more...             6 weeks ago
dotnet-runtime                                        default-route-openshift-image-registry.apps-crc.testing/openshift/dotnet-runtime                                        2.1,2.1-el7,2.1-ubi8,3.1,3.1-el7 + 4 more...             6 weeks ago
driver-toolkit                                        default-route-openshift-image-registry.apps-crc.testing/openshift/driver-toolkit                                        49.84.202112162103-0,latest                              6 weeks ago
eap-cd-openshift                                      default-route-openshift-image-registry.apps-crc.testing/openshift/eap-cd-openshift                                      12,12.0,13,13.0,14,14.0,15,15.0,16,16.0 + 9 more...      6 weeks ago
eap-cd-runtime-openshift                              default-route-openshift-image-registry.apps-crc.testing/openshift/eap-cd-runtime-openshift                              18,18.0,19,19.0,20,20.0,latest                           6 weeks ago
fis-java-openshift                                    default-route-openshift-image-registry.apps-crc.testing/openshift/fis-java-openshift                                    1.0,2.0                                                  6 weeks ago
fis-karaf-openshift                                   default-route-openshift-image-registry.apps-crc.testing/openshift/fis-karaf-openshift                                   1.0,2.0                                                  6 weeks ago
fuse-apicurito-generator                              default-route-openshift-image-registry.apps-crc.testing/openshift/fuse-apicurito-generator                              1.2,1.3,1.4,1.5,1.6,1.7,1.8                              6 weeks ago
fuse7-console                                         default-route-openshift-image-registry.apps-crc.testing/openshift/fuse7-console                                         1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8                      6 weeks ago
fuse7-eap-openshift                                   default-route-openshift-image-registry.apps-crc.testing/openshift/fuse7-eap-openshift                                   1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8                      6 weeks ago
fuse7-java-openshift                                  default-route-openshift-image-registry.apps-crc.testing/openshift/fuse7-java-openshift                                  1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8                      6 weeks ago
fuse7-karaf-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/fuse7-karaf-openshift                                 1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8                      6 weeks ago
golang                                                default-route-openshift-image-registry.apps-crc.testing/openshift/golang                                                1.13.4-ubi7,1.14.7-ubi8,latest                           6 weeks ago
httpd                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/httpd                                                 2.4,2.4-el7,2.4-el8,latest                               6 weeks ago
installer                                             default-route-openshift-image-registry.apps-crc.testing/openshift/installer                                             latest                                                   6 weeks ago
installer-artifacts                                   default-route-openshift-image-registry.apps-crc.testing/openshift/installer-artifacts                                   latest                                                   6 weeks ago
java                                                  default-route-openshift-image-registry.apps-crc.testing/openshift/java                                                  11,8,latest,openjdk-11-el7,openjdk-11-ubi8 + 2 more...   6 weeks ago
jboss-amq-62                                          default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-amq-62                                          1.1,1.2,1.3,1.4,1.5,1.6,1.7                              6 weeks ago
jboss-amq-63                                          default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-amq-63                                          1.0,1.1,1.2,1.3,1.4                                      6 weeks ago
jboss-datagrid65-client-openshift                     default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid65-client-openshift                     1.0,1.1                                                  6 weeks ago
jboss-datagrid65-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid65-openshift                            1.2,1.3,1.4,1.5,1.6                                      6 weeks ago
jboss-datagrid71-client-openshift                     default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid71-client-openshift                     1.0                                                      6 weeks ago
jboss-datagrid71-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid71-openshift                            1.0,1.1,1.2,1.3                                          6 weeks ago
jboss-datagrid72-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid72-openshift                            1.0,1.1,1.2                                              6 weeks ago
jboss-datagrid73-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datagrid73-openshift                            1.0,1.1,1.2,1.3,1.4                                      6 weeks ago
jboss-datavirt64-driver-openshift                     default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datavirt64-driver-openshift                     1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7                          6 weeks ago
jboss-datavirt64-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-datavirt64-openshift                            1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7                          6 weeks ago
jboss-decisionserver64-openshift                      default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-decisionserver64-openshift                      1.0,1.1,1.2,1.3,1.4,1.5,1.6                              6 weeks ago
jboss-eap64-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap64-openshift                                 1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8,1.9 + 1 more...          6 weeks ago
jboss-eap70-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap70-openshift                                 1.3,1.4,1.5,1.6,1.7                                      6 weeks ago
jboss-eap71-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap71-openshift                                 1.1,1.2,1.3,1.4,latest                                   6 weeks ago
jboss-eap72-openjdk11-openshift-rhel8                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap72-openjdk11-openshift-rhel8                 1.0,1.1,latest                                           6 weeks ago
jboss-eap72-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap72-openshift                                 1.0,1.1,1.2,latest                                       6 weeks ago
jboss-eap73-openjdk11-openshift                       default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap73-openjdk11-openshift                       7.3,7.3.0,latest                                         6 weeks ago
jboss-eap73-openjdk11-runtime-openshift               default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap73-openjdk11-runtime-openshift               7.3,7.3.0,latest                                         6 weeks ago
jboss-eap73-openshift                                 default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap73-openshift                                 7.3,7.3.0,latest                                         6 weeks ago
jboss-eap73-runtime-openshift                         default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap73-runtime-openshift                         7.3,7.3.0,latest                                         6 weeks ago
jboss-eap74-openjdk11-openshift                       default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap74-openjdk11-openshift                       7.4.0,latest                                             6 weeks ago
jboss-eap74-openjdk11-runtime-openshift               default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap74-openjdk11-runtime-openshift               7.4.0,latest                                             6 weeks ago
jboss-eap74-openjdk8-openshift                        default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap74-openjdk8-openshift                        7.4.0,latest                                             6 weeks ago
jboss-eap74-openjdk8-runtime-openshift                default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-eap74-openjdk8-runtime-openshift                7.4.0,latest                                             6 weeks ago
jboss-fuse70-console                                  default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-fuse70-console                                  1.0                                                      6 weeks ago
jboss-fuse70-eap-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-fuse70-eap-openshift                            1.0                                                      6 weeks ago
jboss-fuse70-java-openshift                           default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-fuse70-java-openshift                           1.0                                                      6 weeks ago
jboss-fuse70-karaf-openshift                          default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-fuse70-karaf-openshift                          1.0                                                      6 weeks ago
jboss-processserver64-openshift                       default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-processserver64-openshift                       1.0,1.1,1.2,1.3,1.4,1.5,1.6                              6 weeks ago
jboss-webserver31-tomcat7-openshift                   default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver31-tomcat7-openshift                   1.0,1.1,1.2,1.3,1.4                                      6 weeks ago
jboss-webserver31-tomcat8-openshift                   default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver31-tomcat8-openshift                   1.0,1.1,1.2,1.3,1.4                                      6 weeks ago
jboss-webserver54-openjdk11-tomcat9-openshift-rhel7   default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver54-openjdk11-tomcat9-openshift-rhel7   1.0,latest                                               6 weeks ago
jboss-webserver54-openjdk11-tomcat9-openshift-ubi8    default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver54-openjdk11-tomcat9-openshift-ubi8    1.0,latest                                               6 weeks ago
jboss-webserver54-openjdk8-tomcat9-openshift-rhel7    default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver54-openjdk8-tomcat9-openshift-rhel7    1.0,latest                                               6 weeks ago
jboss-webserver54-openjdk8-tomcat9-openshift-ubi8     default-route-openshift-image-registry.apps-crc.testing/openshift/jboss-webserver54-openjdk8-tomcat9-openshift-ubi8     1.0,latest                                               6 weeks ago
jenkins                                               default-route-openshift-image-registry.apps-crc.testing/openshift/jenkins                                               2,latest                                                 6 weeks ago
jenkins-agent-base                                    default-route-openshift-image-registry.apps-crc.testing/openshift/jenkins-agent-base                                    latest                                                   6 weeks ago
jenkins-agent-maven                                   default-route-openshift-image-registry.apps-crc.testing/openshift/jenkins-agent-maven                                   latest,v4.0                                              6 weeks ago
jenkins-agent-nodejs                                  default-route-openshift-image-registry.apps-crc.testing/openshift/jenkins-agent-nodejs                                  latest,v4.0                                              6 weeks ago
mariadb                                               default-route-openshift-image-registry.apps-crc.testing/openshift/mariadb                                               10.3,10.3-el7,10.3-el8,10.5-el7 + 2 more...              6 weeks ago
must-gather                                           default-route-openshift-image-registry.apps-crc.testing/openshift/must-gather                                           latest                                                   6 weeks ago
mysql                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/mysql                                                 8.0,8.0-el7,8.0-el8,latest                               6 weeks ago
nginx                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/nginx                                                 1.16,1.16-el7,1.16-el8,1.18-ubi7 + 2 more...             6 weeks ago
nodejs                                                default-route-openshift-image-registry.apps-crc.testing/openshift/nodejs                                                12,12-ubi7,12-ubi8,14-ubi7,14-ubi8 + 2 more...           6 weeks ago
oauth-proxy                                           default-route-openshift-image-registry.apps-crc.testing/openshift/oauth-proxy                                           v4.4                                                     6 weeks ago
openjdk-11-rhel7                                      default-route-openshift-image-registry.apps-crc.testing/openshift/openjdk-11-rhel7                                      1.0,1.1                                                  6 weeks ago
openjdk-11-rhel8                                      default-route-openshift-image-registry.apps-crc.testing/openshift/openjdk-11-rhel8                                      1.0                                                      6 weeks ago
perl                                                  default-route-openshift-image-registry.apps-crc.testing/openshift/perl                                                  5.26-ubi8,5.30,5.30-el7,5.30-ubi8 + 1 more...            6 weeks ago
php                                                   default-route-openshift-image-registry.apps-crc.testing/openshift/php                                                   7.3,7.3-ubi7,7.3-ubi8,7.4-ubi8,latest                    6 weeks ago
postgresql                                            default-route-openshift-image-registry.apps-crc.testing/openshift/postgresql                                            10,10-el7,10-el8,12,12-el7,12-el8 + 4 more...            6 weeks ago
python                                                default-route-openshift-image-registry.apps-crc.testing/openshift/python                                                2.7,2.7-ubi7,2.7-ubi8,3.6-ubi8,3.8 + 4 more...           6 weeks ago
redhat-openjdk18-openshift                            default-route-openshift-image-registry.apps-crc.testing/openshift/redhat-openjdk18-openshift                            1.0,1.1,1.2,1.3,1.4,1.5,1.6,1.7,1.8                      6 weeks ago
redhat-sso70-openshift                                default-route-openshift-image-registry.apps-crc.testing/openshift/redhat-sso70-openshift                                1.3,1.4                                                  6 weeks ago
redhat-sso71-openshift                                default-route-openshift-image-registry.apps-crc.testing/openshift/redhat-sso71-openshift                                1.0,1.1,1.2,1.3                                          6 weeks ago
redhat-sso72-openshift                                default-route-openshift-image-registry.apps-crc.testing/openshift/redhat-sso72-openshift                                1.0,1.1,1.2,1.3,1.4                                      6 weeks ago
redhat-sso73-openshift                                default-route-openshift-image-registry.apps-crc.testing/openshift/redhat-sso73-openshift                                1.0,latest                                               6 weeks ago
redis                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/redis                                                 5,5-el7,5-el8,latest                                     6 weeks ago
rhdm-decisioncentral-rhel8                            default-route-openshift-image-registry.apps-crc.testing/openshift/rhdm-decisioncentral-rhel8                            7.11.0                                                   6 weeks ago
rhdm-kieserver-rhel8                                  default-route-openshift-image-registry.apps-crc.testing/openshift/rhdm-kieserver-rhel8                                  7.11.0                                                   6 weeks ago
rhpam-businesscentral-monitoring-rhel8                default-route-openshift-image-registry.apps-crc.testing/openshift/rhpam-businesscentral-monitoring-rhel8                7.11.0                                                   6 weeks ago
rhpam-businesscentral-rhel8                           default-route-openshift-image-registry.apps-crc.testing/openshift/rhpam-businesscentral-rhel8                           7.11.0                                                   6 weeks ago
rhpam-kieserver-rhel8                                 default-route-openshift-image-registry.apps-crc.testing/openshift/rhpam-kieserver-rhel8                                 7.11.0                                                   6 weeks ago
rhpam-smartrouter-rhel8                               default-route-openshift-image-registry.apps-crc.testing/openshift/rhpam-smartrouter-rhel8                               7.11.0                                                   6 weeks ago
ruby                                                  default-route-openshift-image-registry.apps-crc.testing/openshift/ruby                                                  2.5-ubi8,2.6,2.6-ubi7,2.6-ubi8,2.7 + 4 more...           6 weeks ago
sso74-openshift-rhel8                                 default-route-openshift-image-registry.apps-crc.testing/openshift/sso74-openshift-rhel8                                 7.4,latest                                               6 weeks ago
tests                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/tests                                                 latest                                                   6 weeks ago
tools                                                 default-route-openshift-image-registry.apps-crc.testing/openshift/tools                                                 latest                                                   6 weeks ago
ubi8-openjdk-11                                       default-route-openshift-image-registry.apps-crc.testing/openshift/ubi8-openjdk-11                                       1.3                                                      6 weeks ago
ubi8-openjdk-8                                        default-route-openshift-image-registry.apps-crc.testing/openshift/ubi8-openjdk-8                                        1.3                                                      6 weeks ag
```

The resulting  custom runtime image is used to run  your own application
Custom images are stored in the OpenShift internal registry, and are accessible  from the current project



```sh
pradeep@learnOpenShift$ oc -o yaml new-app php~https://github.com/pradeepgadde/simple-openshift-app --name=simple1 >
 s2i.yaml
```

```yaml
pradeep@learnOpenShift$ cat s2i.yaml
apiVersion: v1
items:
- apiVersion: image.openshift.io/v1
  kind: ImageStream
  metadata:
    annotations:
      openshift.io/generated-by: OpenShiftNewApp
    creationTimestamp: null
    labels:
      app: simple1
      app.kubernetes.io/component: simple1
      app.kubernetes.io/instance: simple1
    name: simple1
  spec:
    lookupPolicy:
      local: false
  status:
    dockerImageRepository: ""
- apiVersion: build.openshift.io/v1
  kind: BuildConfig
  metadata:
    annotations:
      openshift.io/generated-by: OpenShiftNewApp
    creationTimestamp: null
    labels:
      app: simple1
      app.kubernetes.io/component: simple1
      app.kubernetes.io/instance: simple1
    name: simple1
  spec:
    nodeSelector: null
    output:
      to:
        kind: ImageStreamTag
        name: simple1:latest
    postCommit: {}
    resources: {}
    source:
      git:
        uri: https://github.com/pradeepgadde/simple-openshift-app
      type: Git
    strategy:
      sourceStrategy:
        from:
          kind: ImageStreamTag
          name: php:7.4-ubi8
          namespace: openshift
      type: Source
    triggers:
    - github:
        secret: 2eqGHBsvG0TW29dPD4xB
      type: GitHub
    - generic:
        secret: GIZUWKlkGO4IDlvLLQP_
      type: Generic
    - type: ConfigChange
    - imageChange: {}
      type: ImageChange
  status:
    lastVersion: 0
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      image.openshift.io/triggers: '[{"from":{"kind":"ImageStreamTag","name":"simple1:latest"},"fieldPath":"spec.template.spec.containers[?(@.name==\"simple1\")].image"}]'
      openshift.io/generated-by: OpenShiftNewApp
    creationTimestamp: null
    labels:
      app: simple1
      app.kubernetes.io/component: simple1
      app.kubernetes.io/instance: simple1
    name: simple1
  spec:
    replicas: 1
    selector:
      matchLabels:
        deployment: simple1
    strategy: {}
    template:
      metadata:
        annotations:
          openshift.io/generated-by: OpenShiftNewApp
        creationTimestamp: null
        labels:
          deployment: simple1
      spec:
        containers:
        - image: ' '
          name: simple1
          ports:
          - containerPort: 8080
            protocol: TCP
          - containerPort: 8443
            protocol: TCP
          resources: {}
  status: {}
- apiVersion: v1
  kind: Service
  metadata:
    annotations:
      openshift.io/generated-by: OpenShiftNewApp
    creationTimestamp: null
    labels:
      app: simple1
      app.kubernetes.io/component: simple1
      app.kubernetes.io/instance: simple1
    name: simple1
  spec:
    ports:
    - name: 8080-tcp
      port: 8080
      protocol: TCP
      targetPort: 8080
    - name: 8443-tcp
      port: 8443
      protocol: TCP
      targetPort: 8443
    selector:
      deployment: simple1
  status:
    loadBalancer: {}
kind: List
metadata: {}
```



```sh
pradeep@learnOpenShift$ oc apply -f s2i.yaml
imagestream.image.openshift.io/simple1 created
buildconfig.build.openshift.io/simple1 created
deployment.apps/simple1 created
service/simple1 created
```



```sh
pradeep@learnOpenShift$ oc status
In project mysql on server https://api.crc.testing:6443

http://mysql-openshift-mysql.apps-crc.testing to pod port 3306-tcp (svc/mysql-openshift)
  deployment/mysql-openshift deploys istag/mysql-openshift:latest
    deployment #2 running for about an hour - 1 pod
    deployment #1 deployed about an hour ago

svc/simple1 - 10.217.5.217 ports 8080, 8443
  deployment/simple1 deploys istag/simple1:latest <-
    bc/simple1 source builds https://github.com/pradeepgadde/simple-openshift-app on openshift/php:7.4-ubi8
      build #1 running for 51 seconds - 7695b44: Getting Started with OpenShift by sandervanvugt (Pradeep Gadde <63095349+pradeepgadde@users.noreply.github.com>)
    deployment #1 running for 53 seconds - 0/1 pods growing to 1

pod/bnginx runs bitnami/nginx


2 infos identified, use 'oc status --suggest' to see details.
```



```sh
pradeep@learnOpenShift$ oc get builds
NAME        TYPE     FROM          STATUS    STARTED         DURATION
simple1-1   Source   Git@7695b44   Running   2 minutes ago
```

```sh
pradeep@learnOpenShift$ oc get pods
NAME                              READY   STATUS    RESTARTS   AGE
bnginx                            1/1     Running   0          33m
mysql-openshift-f5b68568f-shwz7   1/1     Running   0          73m
simple1-1-build                   1/1     Running   0          2m41s
```



```sh
pradeep@learnOpenShift$ oc get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
bnginx                            1/1     Running   0          33m     10.217.0.83   crc-xxcfw-master-0   <none>           <none>
mysql-openshift-f5b68568f-shwz7   1/1     Running   0          73m     10.217.0.64   crc-xxcfw-master-0   <none>           <none>
simple1-1-build                   1/1     Running   0          2m48s   10.217.0.98   crc-xxcfw-master-0   <none>           <none>
```



```sh
pradeep@learnOpenShift$ oc get deploy
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
mysql-openshift   1/1     1            1           74m
simple1           0/1     0            0           3m52s
```



```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/bnginx                            1/1     Running   0          36m
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running   0          75m
pod/simple1-1-build                   1/1     Running   0          5m

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70    <none>        3306/TCP,33060/TCP   75m
service/simple1           ClusterIP   10.217.5.217   <none>        8080/TCP,8443/TCP    5m2s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           75m
deployment.apps/simple1           0/1     0            0           5m3s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       75m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       75m
replicaset.apps/simple1-59c9b6d884           1         0         0       5m3s

NAME                                     TYPE     FROM   LATEST
buildconfig.build.openshift.io/simple1   Source   Git    1

NAME                                 TYPE     FROM          STATUS    STARTED         DURATION
build.build.openshift.io/simple1-1   Source   Git@7695b44   Running   5 minutes ago

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   About an hour ago
imagestream.image.openshift.io/simple1           default-route-openshift-image-registry.apps-crc.testing/mysql/simple1

NAME                                       HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
```

```sh
pradeep@learnOpenShift$ oc describe build simple1-1
Name:		simple1-1
Namespace:	mysql
Created:	6 minutes ago
Labels:		app=simple1
		app.kubernetes.io/component=simple1
		app.kubernetes.io/instance=simple1
		buildconfig=simple1
		openshift.io/build-config.name=simple1
		openshift.io/build.start-policy=Serial
Annotations:	openshift.io/build-config.name=simple1
		openshift.io/build.number=1
		openshift.io/build.pod-name=simple1-1-build

Status:		Complete
Started:	Wed, 23 Feb 2022 10:47:46 IST
Duration:	5m44s
  FetchInputs:	  2s
  PullImages:	  4m50s
  Build:	  20s
  PushImage:	  6s

Build Config:	simple1
Build Pod:	simple1-1-build
Image Digest:	sha256:953bec8b4ee57661ae61df0ab8ff7d1ad568d3dead2a1f0f785d1e2cd25c71e0

Strategy:		Source
URL:			https://github.com/pradeepgadde/simple-openshift-app
Commit:			7695b44 (Getting Started with OpenShift by sandervanvugt)
Author/Committer:	Pradeep Gadde / GitHub
From Image:		DockerImage image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:34f2d80adac5a8aeb65c045e3a8c1caea56d53704d8d5cddec588c9008ceb5ea
Pull Secret Name:	builder-dockercfg-n8v7b
Volumes:		<none>
Output to:		ImageStreamTag simple1:latest
Push Secret:		builder-dockercfg-n8v7b

Build trigger cause:	Image change
Image ID:		image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:34f2d80adac5a8aeb65c045e3a8c1caea56d53704d8d5cddec588c9008ceb5ea
Image Name/Kind:	php:7.4-ubi8 / ImageStreamTag

Events:
  Type		Reason		Age			From			Message
  ----		------		----			----			-------
  Normal	Scheduled	6m33s			default-scheduler	Successfully assigned mysql/simple1-1-build to crc-xxcfw-master-0
  Normal	AddedInterface	6m28s			multus			Add eth0 [10.217.0.98/23] from openshift-sdn
  Normal	Pulled		6m27s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Created		6m27s			kubelet			Created container git-clone
  Normal	Started		6m26s			kubelet			Started container git-clone
  Normal	BuildStarted	6m26s			build-controller	Build mysql/simple1-1 is now running
  Normal	Pulled		6m22s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Created		6m13s			kubelet			Created container manage-dockerfile
  Normal	Started		6m9s			kubelet			Started container manage-dockerfile
  Normal	Pulled		6m8s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Created		6m8s			kubelet			Created container sti-build
  Normal	Started		6m7s			kubelet			Started container sti-build
  Normal	BuildCompleted	48s			build-controller	Build mysql/simple1-1 completed successfully
  Warning	FailedMount	47s (x3 over 49s)	kubelet			MountVolume.SetUp failed for volume "build-proxy-ca-bundles" : object "mysql"/"simple1-1-global-ca" not registered
  Warning	FailedMount	47s (x3 over 49s)	kubelet			MountVolume.SetUp failed for volume "build-system-configs" : object "mysql"/"simple1-1-sys-config" not registered
  Warning	FailedMount	47s (x3 over 49s)	kubelet			MountVolume.SetUp failed for volume "build-ca-bundles" : object "mysql"/"simple1-1-ca" not registered
  Warning	FailedMount	47s (x3 over 49s)	kubelet			MountVolume.SetUp failed for volume "builder-dockercfg-n8v7b-pull" : object "mysql"/"builder-dockercfg-n8v7b" not registered
  Warning	FailedMount	47s (x3 over 49s)	kubelet			MountVolume.SetUp failed for volume "builder-dockercfg-n8v7b-push" : object "mysql"/"builder-dockercfg-n8v7b" not registered
```

```sh
pradeep@learnOpenShift$ oc explain buildconfig.spec.triggers
KIND:     BuildConfig
VERSION:  build.openshift.io/v1

RESOURCE: triggers <[]Object>

DESCRIPTION:
     triggers determine how new Builds can be launched from a BuildConfig. If no
     triggers are defined, a new build can only occur as a result of an explicit
     client build creation.

     BuildTriggerPolicy describes a policy for a single trigger that results in
     a new Build.

FIELDS:
   bitbucket	<Object>
     BitbucketWebHook contains the parameters for a Bitbucket webhook type of
     trigger

   generic	<Object>
     generic contains the parameters for a Generic webhook type of trigger

   github	<Object>
     github contains the parameters for a GitHub webhook type of trigger

   gitlab	<Object>
     GitLabWebHook contains the parameters for a GitLab webhook type of trigger

   imageChange	<Object>
     imageChange contains parameters for an ImageChange type of trigger

   type	<string> -required-
     type is the type of build trigger. Valid values:
     - GitHub GitHubWebHookBuildTriggerType represents a trigger that launches
     builds on GitHub webhook invocations

     - Generic GenericWebHookBuildTriggerType represents a trigger that launches
     builds on generic webhook invocations

     - GitLab GitLabWebHookBuildTriggerType represents a trigger that launches
     builds on GitLab webhook invocations

     - Bitbucket BitbucketWebHookBuildTriggerType represents a trigger that
     launches builds on Bitbucket webhook invocations

     - ImageChange ImageChangeBuildTriggerType represents a trigger that
     launches builds on availability of a new version of an image

     - ConfigChange ConfigChangeBuildTriggerType will trigger a build on an
     initial build config creation WARNING: In the future the behavior will
     change to trigger a build on any config change
```

Let us switch to the web console and look at the `mysql` project and applications running currently inside it.

![/assets/images/OS-26](/assets/images/OS-26.png)

```sh
pradeep@learnOpenShift$ oc expose service simple1
route.route.openshift.io/simple1 exposed
```

```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS      RESTARTS   AGE
pod/bnginx                            1/1     Running     2          88m
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running     2          127m
pod/simple1-1-build                   0/1     Completed   0          57m
pod/simple1-7678fbccc9-dnzbd          1/1     Running     2          51m

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70    <none>        3306/TCP,33060/TCP   127m
service/simple1           ClusterIP   10.217.5.217   <none>        8080/TCP,8443/TCP    57m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           127m
deployment.apps/simple1           1/1     1            1           57m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       127m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       127m
replicaset.apps/simple1-59c9b6d884           0         0         0       57m
replicaset.apps/simple1-7678fbccc9           1         1         1       51m

NAME                                     TYPE     FROM   LATEST
buildconfig.build.openshift.io/simple1   Source   Git    1

NAME                                 TYPE     FROM          STATUS     STARTED          DURATION
build.build.openshift.io/simple1-1   Source   Git@7695b44   Complete   57 minutes ago   5m44s

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   2 hours ago
imagestream.image.openshift.io/simple1           default-route-openshift-image-registry.apps-crc.testing/mysql/simple1           latest   51 minutes ago

NAME                                       HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
route.route.openshift.io/simple1           simple1-mysql.apps-crc.testing                  simple1           8080-tcp                 None
```

![](/assets/images/OS-27.png)



![](/assets/images/OS-28.png) 

![](/assets/images/OS-29.png)



## Manually Triggering Updates

Edit the `index.php` in the simple-app  git repository. Remove the PHP Test info section and modify the body to show greetings in English!

Use `oc start-build simple1` to trigger the build procedure again

```sh
pradeep@learnOpenShift$ oc start-build simple1
build.build.openshift.io/simple1-2 started
```
```sh
pradeep@learnOpenShift$ oc get build
NAME        TYPE     FROM          STATUS     STARTED             DURATION
simple1-1   Source   Git@7695b44   Complete   About an hour ago   5m44s
simple1-2   Source   Git@854c5cb   Running    18 seconds ago
```

```sh
pradeep@learnOpenShift$ oc describe build simple1-2
Name:		simple1-2
Namespace:	mysql
Created:	About a minute ago
Labels:		app=simple1
		app.kubernetes.io/component=simple1
		app.kubernetes.io/instance=simple1
		buildconfig=simple1
		openshift.io/build-config.name=simple1
		openshift.io/build.start-policy=Serial
Annotations:	openshift.io/build-config.name=simple1
		openshift.io/build.number=2
		openshift.io/build.pod-name=simple1-2-build

Status:		Complete
Started:	Wed, 23 Feb 2022 11:57:06 IST
Duration:	1m39s
  FetchInputs:	  1s
  PullImages:	  1m18s
  Build:	  8s
  PushImage:	  1s

Build Config:	simple1
Build Pod:	simple1-2-build
Image Digest:	sha256:d4cce6a7d47b33f3cb8d071c210829a71b24f8a2e3c263fc65fdd884eeebbacb

Strategy:		Source
URL:			https://github.com/pradeepgadde/simple-openshift-app
Commit:			854c5cb (To test manual trigger of  OpenShift Build)
Author/Committer:	Pradeep Gadde / GitHub
From Image:		DockerImage image-registry.openshift-image-registry.svc:5000/openshift/php@sha256:34f2d80adac5a8aeb65c045e3a8c1caea56d53704d8d5cddec588c9008ceb5ea
Pull Secret Name:	builder-dockercfg-n8v7b
Volumes:		<none>
Output to:		ImageStreamTag simple1:latest
Push Secret:		builder-dockercfg-n8v7b

Build trigger cause:	Manually triggered

Events:
  Type		Reason		Age			From			Message
  ----		------		----			----			-------
  Normal	Scheduled	113s			default-scheduler	Successfully assigned mysql/simple1-2-build to crc-xxcfw-master-0
  Normal	Started		111s			kubelet			Started container git-clone
  Normal	AddedInterface	111s			multus			Add eth0 [10.217.0.59/23] from openshift-sdn
  Normal	Pulled		111s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Created		111s			kubelet			Created container git-clone
  Normal	BuildStarted	110s			build-controller	Build mysql/simple1-2 is now running
  Normal	Created		108s			kubelet			Created container manage-dockerfile
  Normal	Pulled		108s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Started		107s			kubelet			Started container manage-dockerfile
  Normal	Pulled		106s			kubelet			Container image "quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:961b30e0ab62cf04d000c1eb8330e62ff2daf4545ee094d3c9acd4c443e3c5d2" already present on machine
  Normal	Created		106s			kubelet			Created container sti-build
  Normal	Started		106s			kubelet			Started container sti-build
  Normal	BuildCompleted	15s			build-controller	Build mysql/simple1-2 completed successfully
  Warning	FailedMount	15s (x2 over 15s)	kubelet			MountVolume.SetUp failed for volume "build-ca-bundles" : object "mysql"/"simple1-2-ca" not registered
  Warning	FailedMount	15s (x2 over 15s)	kubelet			MountVolume.SetUp failed for volume "build-system-configs" : object "mysql"/"simple1-2-sys-config" not registered
  Warning	FailedMount	15s (x2 over 15s)	kubelet			MountVolume.SetUp failed for volume "build-proxy-ca-bundles" : object "mysql"/"simple1-2-global-ca" not registered
  Warning	FailedMount	15s (x2 over 15s)	kubelet			MountVolume.SetUp failed for volume "builder-dockercfg-n8v7b-push" : object "mysql"/"builder-dockercfg-n8v7b" not registered
  Warning	FailedMount	15s (x2 over 15s)	kubelet			MountVolume.SetUp failed for volume "builder-dockercfg-n8v7b-pull" : object "mysql"/"builder-dockercfg-n8v7b" not registere
```

![](/assets/images/OS-30.png) 



```sh
pradeep@learnOpenShift$ oc get all
NAME                                  READY   STATUS      RESTARTS   AGE
pod/bnginx                            1/1     Running     2          103m
pod/mysql-openshift-f5b68568f-shwz7   1/1     Running     2          143m
pod/simple1-1-build                   0/1     Completed   0          72m
pod/simple1-2-build                   0/1     Completed   0          3m32s
pod/simple1-56465c7f74-xp9m8          1/1     Running     0          115s

NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/mysql-openshift   ClusterIP   10.217.4.70    <none>        3306/TCP,33060/TCP   143m
service/simple1           ClusterIP   10.217.5.217   <none>        8080/TCP,8443/TCP    72m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-openshift   1/1     1            1           143m
deployment.apps/simple1           1/1     1            1           72m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-openshift-68996c4c8c   0         0         0       143m
replicaset.apps/mysql-openshift-f5b68568f    1         1         1       143m
replicaset.apps/simple1-56465c7f74           1         1         1       115s
replicaset.apps/simple1-59c9b6d884           0         0         0       72m
replicaset.apps/simple1-7678fbccc9           0         0         0       67m

NAME                                     TYPE     FROM   LATEST
buildconfig.build.openshift.io/simple1   Source   Git    2

NAME                                 TYPE     FROM          STATUS     STARTED             DURATION
build.build.openshift.io/simple1-1   Source   Git@7695b44   Complete   About an hour ago   5m44s
build.build.openshift.io/simple1-2   Source   Git@854c5cb   Complete   3 minutes ago       1m39s

NAME                                             IMAGE REPOSITORY                                                                TAGS     UPDATED
imagestream.image.openshift.io/mysql-openshift   default-route-openshift-image-registry.apps-crc.testing/mysql/mysql-openshift   latest   2 hours ago
imagestream.image.openshift.io/simple1           default-route-openshift-image-registry.apps-crc.testing/mysql/simple1           latest   About a minute ago

NAME                                       HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
route.route.openshift.io/simple1           simple1-mysql.apps-crc.testing                  simple1           8080-tcp                 None
```

```sh
pradeep@learnOpenShift$ curl simple1-mysql.apps-crc.testing
<html>
	<head>
	<title>PHP Test</title>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1"/>
	</head>

<body>
	<h1>PHP Test</h1>
		<p><b>An Example of PHP in Action</b></p>
		The Current Date and Time is: <br />6:30 AM Wednesday, February 23 2022. </p>

        <h2>Greetings from Pradeep Gadde!</h2>
	</body>
</html>
```

These tests confirm the changes.



### Understanding Routes

Routes are an alternative to Kubernetes Ingress

Routes are OpenShift resources that allow for network access to pods from users and applications outside the OpenShift instance

A route connects a public IP address and DNS name to the internal service IP address

OpenShift uses a HAProxy based shared router service to implement the route.

When exposing a service, the DNS name is auto-generated as `routename-projectname-defaultDNSdomain`

```sh
pradeep@learnOpenShift$ oc get route
NAME              HOST/PORT                                PATH   SERVICES          PORT       TERMINATION   WILDCARD
mysql-openshift   mysql-openshift-mysql.apps-crc.testing          mysql-openshift   3306-tcp                 None
simple1           simple1-mysql.apps-crc.testing                  simple1           8080-tcp                 None
```



This completes my initial testing of RedHat OpenShift on CloudReady Container platform.  There is one more major topic to basic tests that is storage, which I haven't explored yet. 

I will keep updating this post as I get a chance to test and learn.

Keep learning :rocket:



Best regards,

Pradeep




![](/assets/images/RedHatOpenShift.png)  