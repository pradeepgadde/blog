---
layout: single
title:  "Kubernetes Operator Example"
date:   2022-06-29 10:55:04 +0530
categories: Kubernetes
tags: minikube awx-operator
classes: wide
show_date: true
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  teaser: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
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

# Kubernetes Operator

According to Kubernetes documentation, Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components. Operators follow Kubernetes principles, notably the control loop.

Kubernetes' operator pattern concept lets you extend the cluster's behaviour without modifying the code of Kubernetes itself by linking controllers to one or more custom resources. Operators are clients of the Kubernetes API that act as controllers for a Custom Resource.

## An Example Operator
AWX Operator: This operator is meant to provide a more Kubernetes-native installation method for AWX via an AWX Custom Resource Definition (CRD).
https://github.com/ansible/awx-operator

Let us test this operator in Minukube.

```sh
(base) pradeep:~$ minikube start --cpus=4 --memory=6g --addons=ingress
😄  minikube v1.25.2 on Darwin 12.4
✨  Using the docker driver based on existing profile
❗  You cannot change the memory size for an existing minikube cluster. Please first delete the cluster.
❗  You cannot change the CPUs for an existing minikube cluster. Please first delete the cluster.
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🤷  docker "minikube" container is missing, will recreate.
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  Enabled addons: storage-provisioner, default-storageclass, ingress
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get nodes -A
NAME       STATUS   ROLES                  AGE   VERSION
minikube   Ready    control-plane,master   53d   v1.23.3
(base) pradeep:~$
```
```sh
(base) pradeep:~$kubectl get pods -A 
NAMESPACE       NAME                                       READY   STATUS      RESTARTS          AGE
default         cpx-ingress-64464fdb75-9wchv               2/2     Running     2 (2m36s ago)     53d
default         frontend-7b988b7c4b-2vmht                  1/1     Running     1 (2m36s ago)     53d
default         frontend-7b988b7c4b-8q8t9                  1/1     Running     1 (2m36s ago)     53d
default         frontend-7b988b7c4b-xmp7f                  1/1     Running     1 (2m36s ago)     53d
default         redis-master-55db8bb568-9lnqx              1/1     Running     1 (2m36s ago)     53d
default         redis-slave-566774f44b-mwzzn               1/1     Running     1 (2m36s ago)     53d
default         redis-slave-566774f44b-whbj8               1/1     Running     1 (2m36s ago)     53d
ingress-nginx   ingress-nginx-admission-create-trm4s       0/1     Completed   0                 108s
ingress-nginx   ingress-nginx-admission-patch-njpkl        0/1     Completed   0                 108s
ingress-nginx   ingress-nginx-controller-cc8496874-c2qkk   1/1     Running     0                 108s
kube-system     coredns-64897985d-tkhw6                    1/1     Running     3 (2m36s ago)     53d
kube-system     etcd-minikube                              1/1     Running     3 (2m36s ago)     53d
kube-system     kube-apiserver-minikube                    1/1     Running     110 (2m36s ago)   53d
kube-system     kube-controller-manager-minikube           1/1     Running     1 (2m36s ago)     53d
kube-system     kube-proxy-znj26                           1/1     Running     1 (2m36s ago)     53d
kube-system     kube-scheduler-minikube                    1/1     Running     1 (2m36s ago)     53d
kube-system     storage-provisioner                        1/1     Running     184 (82s ago)     53d
(base) pradeep:~$
```

Verify if any CRDs present.

```sh
(base) pradeep:~$kubectl get crd
No resources found
(base) pradeep:~$
```

Once you have a running Kubernetes cluster, you can deploy AWX Operator into your cluster using Kustomize. Follow the instructions here to install the latest version of Kustomize: https://kubectl.docs.kubernetes.io/installation/kustomize/

```sh
(base) pradeep:~$curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
{Version:kustomize/v4.5.5 GitCommit:daa3e5e2c2d3a4b8c94021a7384bfb06734bcd26 BuildDate:2022-05-20T20:25:40Z GoOs:darwin GoArch:amd64}
kustomize installed to /Users/pradeep/kustomize
(base) pradeep:~$
```

```sh
(base) pradeep:~$/Users/pradeep/kustomize version
{Version:kustomize/v4.5.5 GitCommit:daa3e5e2c2d3a4b8c94021a7384bfb06734bcd26 BuildDate:2022-05-20T20:25:40Z GoOs:darwin GoArch:amd64}
(base) pradeep:~$
```
```sh
(base) pradeep:~$/Users/pradeep/kustomize -h     

Manages declarative configuration of Kubernetes.
See https://sigs.k8s.io/kustomize

Usage:
  kustomize [command]

Available Commands:
  build                     Build a kustomization target from a directory or URL.
  cfg                       Commands for reading and writing configuration.
  completion                Generate shell completion script
  create                    Create a new kustomization in the current directory
  edit                      Edits a kustomization file
  fn                        Commands for running functions against configuration.
  help                      Help about any command
  version                   Prints the kustomize version

Flags:
  -h, --help          help for kustomize
      --stack-trace   print a stack-trace on error

Additional help topics:
  kustomize docs-fn                   [Alpha] Documentation for developing and invoking Configuration Functions.
  kustomize docs-fn-spec              [Alpha] Documentation for Configuration Functions Specification.
  kustomize docs-io-annotations       [Alpha] Documentation for annotations used by io.
  kustomize docs-merge                [Alpha] Documentation for merging Resources (2-way merge).
  kustomize docs-merge3               [Alpha] Documentation for merging Resources (3-way merge).
  kustomize tutorials-command-basics  [Alpha] Tutorials for using basic config commands.
  kustomize tutorials-function-basics [Alpha] Tutorials for using functions.

Use "kustomize [command] --help" for more information about a command.
(base) pradeep:~$

```

```yaml
(base) pradeep:~$cat kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=0.23.0

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 0.23.0

# Specify a custom namespace in which to install AWX
namespace: awx
(base) pradeep:~$
```

```sh
(base) pradeep:~$/Users/pradeep/kustomize build . | kubectl apply -f -
namespace/awx created
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com created
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com created
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com created
serviceaccount/awx-operator-controller-manager created
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role created
role.rbac.authorization.k8s.io/awx-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding created
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding created
configmap/awx-operator-awx-manager-config created
service/awx-operator-controller-manager-metrics-service created
deployment.apps/awx-operator-controller-manager created
(base) pradeep:~$

```

Wait a bit and you should have the `awx-operator` running:

```sh
(base) pradeep:~$kubectl get pods -n awx
NAME                                               READY   STATUS              RESTARTS   AGE
awx-operator-controller-manager-7594795b6b-qlmqc   0/2     ContainerCreating   0          65s
(base) pradeep:~$

```
After some time
```sh
(base) pradeep:~$kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-7594795b6b-qlmqc   2/2     Running   0          2m43s
(base) pradeep:~$

```

Next, create a file named `awx-demo.yaml` in the same folder
```yaml
(base) pradeep:~$cat awx-demo.yaml 
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  # default nodeport_port is 30080
  nodeport_port: 30080
(base) pradeep:~$
```

Look at the kind, `AWX`. Its a custom resource (CRD)

Verify if the operator has installed any CRDs.
```sh
(base) pradeep:~$kubectl get crd
NAME                          CREATED AT
awxbackups.awx.ansible.com    2022-06-29T14:44:04Z
awxrestores.awx.ansible.com   2022-06-29T14:44:04Z
awxs.awx.ansible.com          2022-06-29T14:44:04Z
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl describe crds     
Name:         awxbackups.awx.ansible.com
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-06-29T14:44:04Z
  Generation:          1
  Managed Fields:
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:acceptedNames:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:conditions:
          k:{"type":"Established"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
          k:{"type":"NamesAccepted"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
    Manager:      Go-http-client
    Operation:    Update
    Subresource:  status
    Time:         2022-06-29T14:44:04Z
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        f:conversion:
          .:
          f:strategy:
        f:group:
        f:names:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:scope:
        f:versions:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2022-06-29T14:44:04Z
  Resource Version:  28858
  UID:               e77ee16b-2552-437c-aac4-129d21d965b1
Spec:
  Conversion:
    Strategy:  None
  Group:       awx.ansible.com
  Names:
    Kind:       AWXBackup
    List Kind:  AWXBackupList
    Plural:     awxbackups
    Singular:   awxbackup
  Scope:        Namespaced
  Versions:
    Name:  v1beta1
    Schema:
      openAPIV3Schema:
        Description:  Schema validation for the AWXBackup CRD
        Properties:
          Spec:
            Properties:
              backup_pvc:
                Description:  Name of the PVC to be used for storing the backup
                Type:         string
              backup_pvc_namespace:
                Description:  Namespace the PVC is in
                Type:         string
              backup_storage_class:
                Description:  Storage class to use when creating PVC for backup
                Type:         string
              backup_storage_requirements:
                Description:  Storage requirements for the PostgreSQL container
                Type:         string
              deployment_name:
                Description:  Name of the deployment to be backed up
                Type:         string
              no_log:
                Description:  Configure no_log for no_log tasks
                Type:         string
              postgres_image:
                Description:  Registry path to the PostgreSQL container to use
                Type:         string
              postgres_image_version:
                Description:  PostgreSQL container image version to use
                Type:         string
              postgres_label_selector:
                Description:  Label selector used to identify postgres pod for backing up data
                Type:         string
            Required:
              deployment_name
            Type:  object
          Status:
            Properties:
              Backup Claim:
                Description:  Backup persistent volume claim
                Type:         string
              Backup Directory:
                Description:  Backup directory name on the specified pvc
                Type:         string
              Conditions:
                Description:  The resulting conditions when a Service Telemetry is instantiated
                Items:
                  Properties:
                    Last Transition Time:
                      Type:  string
                    Reason:
                      Type:  string
                    Status:
                      Type:  string
                    Type:
                      Type:                            string
                  Type:                                object
                Type:                                  array
            Type:                                      object
        Type:                                          object
        X - Kubernetes - Preserve - Unknown - Fields:  true
    Served:                                            true
    Storage:                                           true
    Subresources:
      Status:
Status:
  Accepted Names:
    Kind:       AWXBackup
    List Kind:  AWXBackupList
    Plural:     awxbackups
    Singular:   awxbackup
  Conditions:
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1beta1
Events:  <none>


Name:         awxrestores.awx.ansible.com
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-06-29T14:44:04Z
  Generation:          1
  Managed Fields:
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:acceptedNames:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:conditions:
          k:{"type":"Established"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
          k:{"type":"NamesAccepted"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
    Manager:      Go-http-client
    Operation:    Update
    Subresource:  status
    Time:         2022-06-29T14:44:04Z
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        f:conversion:
          .:
          f:strategy:
        f:group:
        f:names:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:scope:
        f:versions:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2022-06-29T14:44:04Z
  Resource Version:  28859
  UID:               d6c9d8e0-a067-4f7e-a853-fed09d0280ee
Spec:
  Conversion:
    Strategy:  None
  Group:       awx.ansible.com
  Names:
    Kind:       AWXRestore
    List Kind:  AWXRestoreList
    Plural:     awxrestores
    Singular:   awxrestore
  Scope:        Namespaced
  Versions:
    Name:  v1beta1
    Schema:
      openAPIV3Schema:
        Description:  Schema validation for the AWXRestore CRD
        Properties:
          Spec:
            Properties:
              backup_dir:
                Description:  Backup directory name, set as a status found on the awxbackup object (backupDirectory)
                Type:         string
              backup_name:
                Description:  AWXBackup object name
                Type:         string
              backup_pvc:
                Description:  Name of the PVC to be restored from, set as a status found on the awxbackup object (backupClaim)
                Type:         string
              backup_pvc_namespace:
                Description:  Namespace the PVC is in
                Type:         string
              backup_source:
                Description:  Backup source
                Enum:
                  CR
                  PVC
                Type:  string
              deployment_name:
                Description:  Name of the deployment to be restored to
                Type:         string
              no_log:
                Description:  Configure no_log for no_log tasks
                Type:         string
              postgres_image:
                Description:  Registry path to the PostgreSQL container to use
                Type:         string
              postgres_image_version:
                Description:  PostgreSQL container image version to use
                Type:         string
              postgres_label_selector:
                Description:  Label selector used to identify postgres pod for backing up data
                Type:         string
            Type:             object
          Status:
            Properties:
              Conditions:
                Description:  The resulting conditions when a Service Telemetry is instantiated
                Items:
                  Properties:
                    Last Transition Time:
                      Type:  string
                    Reason:
                      Type:  string
                    Status:
                      Type:  string
                    Type:
                      Type:  string
                  Type:      object
                Type:        array
              Restore Complete:
                Description:                           Restore process complete
                Type:                                  boolean
            Type:                                      object
        Type:                                          object
        X - Kubernetes - Preserve - Unknown - Fields:  true
    Served:                                            true
    Storage:                                           true
    Subresources:
      Status:
Status:
  Accepted Names:
    Kind:       AWXRestore
    List Kind:  AWXRestoreList
    Plural:     awxrestores
    Singular:   awxrestore
  Conditions:
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1beta1
Events:  <none>


Name:         awxs.awx.ansible.com
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-06-29T14:44:04Z
  Generation:          1
  Managed Fields:
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:acceptedNames:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:conditions:
          k:{"type":"Established"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
          k:{"type":"NamesAccepted"}:
            .:
            f:lastTransitionTime:
            f:message:
            f:reason:
            f:status:
            f:type:
    Manager:      Go-http-client
    Operation:    Update
    Subresource:  status
    Time:         2022-06-29T14:44:04Z
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        f:conversion:
          .:
          f:strategy:
        f:group:
        f:names:
          f:kind:
          f:listKind:
          f:plural:
          f:singular:
        f:scope:
        f:versions:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2022-06-29T14:44:04Z
  Resource Version:  28869
  UID:               c08ee52b-5796-440a-929c-fc262fcf6a81
Spec:
  Conversion:
    Strategy:  None
  Group:       awx.ansible.com
  Names:
    Kind:       AWX
    List Kind:  AWXList
    Plural:     awxs
    Singular:   awx
  Scope:        Namespaced
  Versions:
    Name:  v1beta1
    Schema:
      openAPIV3Schema:
        Description:  Schema validation for the AWX CRD
        Properties:
          Spec:
            Properties:
              admin_email:
                Description:  The admin user email
                Type:         string
              admin_password_secret:
                Description:  Secret where the admin password can be found
                Type:         string
              admin_user:
                Default:      admin
                Description:  Username to use for the admin account
                Type:         string
              Annotations:
                Description:  annotations for the pods
                Type:         string
              api_version:
                Description:  apiVersion of the deployment type
                Type:         string
              broadcast_websocket_secret:
                Description:  Secret where the broadcast websocket secret can be found
                Type:         string
              bundle_cacert_secret:
                Description:  Secret where can be found the trusted Certificate Authority Bundle
                Type:         string
              ca_trust_bundle:
                Description:  Path where the trusted CA bundle is available
                Type:         string
              control_plane_ee_image:
                Description:  Registry path to the Execution Environment container image to use on control plane pods
                Type:         string
              control_plane_priority_class:
                Description:  Assign a preexisting priority class to the control plane pods
                Type:         string
              create_preload_data:
                Default:      true
                Description:  Whether or not to preload data upon instance creation
                Type:         boolean
              csrf_cookie_secure:
                Description:  Set csrf cookie secure mode for web
                Type:         string
              deployment_type:
                Description:  Name of the deployment type
                Type:         string
              development_mode:
                Description:  If the deployment should be done in development mode
                Type:         boolean
              ee_extra_env:
                Type:  string
              ee_extra_volume_mounts:
                Description:  Specify volume mounts to be added to Execution container
                Type:         string
              ee_images:
                Description:  Registry path to the Execution Environment container to use
                Items:
                  Properties:
                    Image:
                      Type:  string
                    Name:
                      Type:  string
                  Type:      object
                Type:        array
              ee_pull_credentials_secret:
                Description:  Secret where pull credentials for registered ees can be found
                Type:         string
              ee_resource_requirements:
                Description:  Resource requirements for the ee container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
              extra_settings:
                Description:  Extra settings to specify for the API
                Items:
                  Properties:
                    Setting:
                      Type:  string
                    Value:
                      X - Kubernetes - Preserve - Unknown - Fields:  true
                  Type:                                              object
                Type:                                                array
              extra_volumes:
                Description:  Specify extra volumes to add to the application pod
                Type:         string
              garbage_collect_secrets:
                Default:      false
                Description:  Whether or not to remove secrets upon instance removal
                Type:         boolean
              Hostname:
                Description:  The hostname of the instance
                Type:         string
              Image:
                Description:  Registry path to the application container to use
                Type:         string
              image_pull_policy:
                Default:      IfNotPresent
                Description:  The image pull policy
                Enum:
                  Always
                  always
                  Never
                  never
                  IfNotPresent
                  ifnotpresent
                Type:  string
              image_pull_secret:
                Description:  (Deprecated) Image pull secret for app and database containers
                Type:         string
              image_pull_secrets:
                Description:  Image pull secrets for app and database containers
                Items:
                  Type:  string
                Type:    array
              image_version:
                Description:  Application container image version to use
                Type:         string
              ingress_annotations:
                Description:  Annotations to add to the Ingress Controller
                Type:         string
              ingress_path:
                Description:  The ingress path used to reach the deployed service
                Type:         string
              ingress_path_type:
                Description:  The ingress path type for the deployed service
                Type:         string
              ingress_tls_secret:
                Description:  Secret where the Ingress TLS secret can be found
                Type:         string
              ingress_type:
                Description:  The ingress type to use to reach the deployed instance
                Enum:
                  none
                  Ingress
                  ingress
                  Route
                  route
                Type:  string
              init_container_extra_commands:
                Description:  Extra commands for the init container
                Type:         string
              init_container_extra_volume_mounts:
                Description:  Specify volume mounts to be added to the init container
                Type:         string
              init_container_image:
                Description:  Registry path to the init container to use
                Type:         string
              init_container_image_version:
                Description:  Init container image version to use
                Type:         string
              Kind:
                Description:  Kind of the deployment type
                Type:         string
              ldap_cacert_secret:
                Description:  Secret where can be found the LDAP trusted Certificate Authority Bundle
                Type:         string
              ldap_password_secret:
                Description:  Secret where can be found the LDAP bind password
                Type:         string
              loadbalancer_port:
                Default:      80
                Description:  Port to use for the loadbalancer
                Type:         integer
              loadbalancer_protocol:
                Default:      http
                Description:  Protocol to use for the loadbalancer
                Enum:
                  http
                  https
                Type:  string
              no_log:
                Description:  Configure no_log for no_log tasks
                Type:         string
              node_selector:
                Description:  nodeSelector for the pods
                Type:         string
              nodeport_port:
                Default:      30080
                Description:  Port to use for the nodeport
                Type:         integer
              old_postgres_configuration_secret:
                Description:  Secret where the old database configuration can be found for data migration
                Type:         string
              postgres_configuration_secret:
                Description:  Secret where the database configuration can be found
                Type:         string
              postgres_data_path:
                Description:  Path where the PostgreSQL data are located
                Type:         string
              postgres_extra_args:
                Items:
                  Type:  string
                Type:    array
              postgres_image:
                Description:  Registry path to the PostgreSQL container to use
                Type:         string
              postgres_image_version:
                Description:  PostgreSQL container image version to use
                Type:         string
              postgres_init_container_resource_requirements:
                Description:  Resource requirements for the postgres init container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
              postgres_label_selector:
                Description:  Label selector used to identify postgres pod for data migration
                Type:         string
              postgres_priority_class:
                Description:  Assign a preexisting priority class to the postgres pod
                Type:         string
              postgres_resource_requirements:
                Description:  Resource requirements for the PostgreSQL container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                    Type:      object
                Type:          object
              postgres_selector:
                Description:  nodeSelector for the Postgres pods
                Type:         string
              postgres_storage_class:
                Description:  Storage class to use for the PostgreSQL PVC
                Type:         string
              postgres_storage_requirements:
                Description:  Storage requirements for the PostgreSQL container
                Properties:
                  Limits:
                    Properties:
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
              postgres_tolerations:
                Description:  node tolerations for the Postgres pods
                Type:         string
              projects_existing_claim:
                Description:  PersistentVolumeClaim to mount /var/lib/projects directory
                Type:         string
              projects_persistence:
                Default:      false
                Description:  Whether or not the /var/lib/projects directory will be persistent
                Type:         boolean
              projects_storage_access_mode:
                Default:      ReadWriteMany
                Description:  AccessMode for the /var/lib/projects PersistentVolumeClaim
                Type:         string
              projects_storage_class:
                Description:  Storage class for the /var/lib/projects PersistentVolumeClaim
                Type:         string
              projects_storage_size:
                Default:      8Gi
                Description:  Size for the /var/lib/projects PersistentVolumeClaim
                Type:         string
              projects_use_existing_claim:
                Description:  Using existing PersistentVolumeClaim
                Enum:
                  _Yes_
                  _No_
                Type:  string
              redis_capabilities:
                Description:  Redis container capabilities
                Items:
                  Type:  string
                Type:    array
              redis_image:
                Description:  Registry path to the redis container to use
                Type:         string
              redis_image_version:
                Description:  Redis container image version to use
                Type:         string
              redis_resource_requirements:
                Description:  Resource requirements for the redis container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
              Replicas:
                Default:      1
                Description:  Number of instance replicas
                Format:       int32
                Type:         integer
              route_host:
                Description:  The DNS to use to points to the instance
                Type:         string
              route_tls_secret:
                Description:  Secret where the TLS related credentials are stored
                Type:         string
              route_tls_termination_mechanism:
                Default:      Edge
                Description:  The secure TLS termination mechanism to use
                Enum:
                  Edge
                  edge
                  Passthrough
                  passthrough
                Type:  string
              secret_key_secret:
                Description:  Secret where the secret key can be found
                Type:         string
              security_context_settings:
                Description:                                   Key/values that will be set under the pod-level securityContext field
                Type:                                          object
                X - Kubernetes - Preserve - Unknown - Fields:  true
              service_account_annotations:
                Description:  ServiceAccount annotations
                Type:         string
              service_annotations:
                Description:  Annotations to add to the service
                Type:         string
              service_labels:
                Description:  Additional labels to apply to the service
                Type:         string
              service_type:
                Description:  The service type to be used on the deployed instance
                Enum:
                  LoadBalancer
                  loadbalancer
                  ClusterIP
                  clusterip
                  NodePort
                  nodeport
                Type:  string
              session_cookie_secure:
                Description:  Set session cookie secure mode for web
                Type:         string
              task_args:
                Items:
                  Type:  string
                Type:    array
              task_command:
                Items:
                  Type:  string
                Type:    array
              task_extra_env:
                Type:  string
              task_extra_volume_mounts:
                Description:  Specify volume mounts to be added to Task container
                Type:         string
              task_privileged:
                Default:      false
                Description:  If a privileged security context should be enabled
                Type:         boolean
              task_resource_requirements:
                Description:  Resource requirements for the task container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
              Tolerations:
                Description:  node tolerations for the pods
                Type:         string
              topology_spread_constraints:
                Description:  topology rule(s) for the pods
                Type:         string
              web_args:
                Items:
                  Type:  string
                Type:    array
              web_command:
                Items:
                  Type:  string
                Type:    array
              web_extra_env:
                Type:  string
              web_extra_volume_mounts:
                Description:  Specify volume mounts to be added to the Web container
                Type:         string
              web_resource_requirements:
                Description:  Resource requirements for the web container
                Properties:
                  Limits:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                  Requests:
                    Properties:
                      Cpu:
                        Type:  string
                      Memory:
                        Type:  string
                      Storage:
                        Type:  string
                    Type:      object
                Type:          object
            Type:              object
          Status:
            Properties:
              URL:
                Description:  URL to access the deployed instance
                Type:         string
              Admin Password Secret:
                Description:  Admin password secret name of the deployed instance
                Type:         string
              Admin User:
                Description:  Admin user of the deployed instance
                Type:         string
              Broadcast Websocket Secret:
                Description:  Broadcast websocket secret name of the deployed instance
                Type:         string
              Conditions:
                Description:  The resulting conditions when a Service Telemetry is instantiated
                Items:
                  Properties:
                    Last Transition Time:
                      Type:  string
                    Reason:
                      Type:  string
                    Status:
                      Type:  string
                    Type:
                      Type:  string
                  Type:      object
                Type:        array
              Image:
                Description:  URL of the image used for the deployed instance
                Type:         string
              Migrated From Secret:
                Description:  The secret used for migrating an old instance.
                Type:         string
              Postgres Configuration Secret:
                Description:  Postgres Configuration secret name of the deployed instance
                Type:         string
              Secret Key Secret:
                Description:  Secret key secret name of the deployed instance
                Type:         string
              Version:
                Description:  Version of the deployed instance
                Type:         string
            Type:             object
        Type:                 object
    Served:                   true
    Storage:                  true
    Subresources:
      Status:
Status:
  Accepted Names:
    Kind:       AWX
    List Kind:  AWXList
    Plural:     awxs
    Singular:   awx
  Conditions:
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-06-29T14:44:04Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1beta1
Events:  <none>
(base) pradeep:~$
```

Make sure to add this new file to the list of "resources" in your kustomization.yaml file:

```yaml
(base) pradeep:~$cat kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=0.23.0
  - awx-demo.yaml 
# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 0.23.0

# Specify a custom namespace in which to install AWX
namespace: awx
(base) pradeep:~$
```

Finally, run kustomize again to create the AWX instance in your cluster:

```sh
(base) pradeep:~$/Users/pradeep/kustomize build . | kubectl apply -f -
namespace/awx unchanged
customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com unchanged
customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com unchanged
serviceaccount/awx-operator-controller-manager unchanged
role.rbac.authorization.k8s.io/awx-operator-awx-manager-role configured
role.rbac.authorization.k8s.io/awx-operator-leader-election-role unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-metrics-reader unchanged
clusterrole.rbac.authorization.k8s.io/awx-operator-proxy-role unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-awx-manager-rolebinding unchanged
rolebinding.rbac.authorization.k8s.io/awx-operator-leader-election-rolebinding unchanged
clusterrolebinding.rbac.authorization.k8s.io/awx-operator-proxy-rolebinding unchanged
configmap/awx-operator-awx-manager-config unchanged
service/awx-operator-controller-manager-metrics-service unchanged
deployment.apps/awx-operator-controller-manager unchanged
awx.awx.ansible.com/awx-demo created
(base) pradeep:~$
```


After a few minutes, the new AWX instance will be deployed. You can look at the operator pod logs in order to know where the installation process is at:

```json
(base) pradeep:~$kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager -n awx
{"level":"info","ts":1656513979.0858583,"logger":"cmd","msg":"Version","Go Version":"go1.16.9","GOOS":"linux","GOARCH":"amd64","ansible-operator":"v1.12.0","commit":"d3b2761afdb78f629a7eaf4461b0fb8ae3b02860"}
{"level":"info","ts":1656513979.08625,"logger":"cmd","msg":"Watching single namespace.","Namespace":"awx"}
{"level":"info","ts":1656513979.247896,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"127.0.0.1:8080"}
{"level":"info","ts":1656513979.2497354,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWX_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656513979.2499053,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWXBACKUP_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656513979.2499378,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWXRESTORE_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656513979.2499719,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWX"}
{"level":"info","ts":1656513979.2503543,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWXBackup"}
{"level":"info","ts":1656513979.250415,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWXRestore"}
{"level":"info","ts":1656513979.254063,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
I0629 14:46:19.253894       7 leaderelection.go:243] attempting to acquire leader lease awx/awx-operator...
{"level":"info","ts":1656513979.2537081,"logger":"proxy","msg":"Starting to serve","Address":"127.0.0.1:8888"}
I0629 14:46:19.276210       7 leaderelection.go:253] successfully acquired lease awx/awx-operator
{"level":"info","ts":1656513979.2779229,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWXRestore"}
{"level":"info","ts":1656513979.278723,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting Controller"}
{"level":"info","ts":1656513979.2780023,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWXBackup"}
{"level":"info","ts":1656513979.2787871,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting Controller"}
{"level":"info","ts":1656513979.2835915,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656513979.2839265,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting Controller"}
{"level":"info","ts":1656513979.382164,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting workers","worker count":4}
{"level":"info","ts":1656513979.3835855,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting workers","worker count":4}
{"level":"info","ts":1656513979.3863482,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting workers","worker count":4}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Patching labels to AWX kind] *********************************
task path: /opt/ansible/roles/installer/tasks/main.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656514381.2612007,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Patching labels to AWX kind"}
{"level":"info","ts":1656514383.8062181,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/apis/awx.ansible.com/v1beta1/namespaces/awx/awxs/awx-demo","Verb":"get","APIPrefix":"apis","APIGroup":"awx.ansible.com","APIVersion":"v1beta1","Namespace":"awx","Resource":"awxs","Subresource":"","Name":"awx-demo","Parts":["awxs","awx-demo"]}}
{"level":"info","ts":1656514383.9843922,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Include secret key configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include secret key configuration tasks] **********************
task path: /opt/ansible/roles/installer/tasks/main.yml:20

-------------------------------------------------------------------------------
{"level":"info","ts":1656514384.0234187,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for specified secret key configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified secret key configuration] ****************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656514384.135357,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for default secret key configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default secret key configuration] ******************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656514385.684847,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656514385.9231038,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Create secret key secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create secret key secret] ************************************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:25

-------------------------------------------------------------------------------
{"level":"info","ts":1656514387.1324403,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656514387.1379874,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656514387.1429126,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514387.1436377,"logger":"proxy","msg":"Watching child resource","kind":"/v1, Kind=Secret","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656514387.143966,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: /v1, Kind=Secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read secret key secret] **************************************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:31

-------------------------------------------------------------------------------
{"level":"info","ts":1656514387.2785668,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Read secret key secret"}
{"level":"info","ts":1656514388.1862056,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-secret-key","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-secret-key","Parts":["secrets","awx-demo-secret-key"]}}
{"level":"info","ts":1656514388.5468383,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Load LDAP CAcert certificate"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load LDAP CAcert certificate] ********************************
task path: /opt/ansible/roles/installer/tasks/main.yml:23

-------------------------------------------------------------------------------
{"level":"info","ts":1656514388.6443744,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Load ldap bind password"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load ldap bind password] *************************************
task path: /opt/ansible/roles/installer/tasks/main.yml:28

-------------------------------------------------------------------------------
{"level":"info","ts":1656514388.7427282,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Load bundle certificate authority certificate"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load bundle certificate authority certificate] ***************
task path: /opt/ansible/roles/installer/tasks/main.yml:33

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include admin password configuration tasks] ******************
task path: /opt/ansible/roles/installer/tasks/main.yml:38

-------------------------------------------------------------------------------
{"level":"info","ts":1656514388.861366,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Include admin password configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified admin password configuration] ************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656514388.8959813,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for specified admin password configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default admin password configuration] **************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656514388.996421,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for default admin password configuration"}
{"level":"info","ts":1656514389.8901668,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create admin password secret] ********************************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:25

-------------------------------------------------------------------------------
{"level":"info","ts":1656514390.1513214,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Create admin password secret"}
{"level":"info","ts":1656514391.1139622,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}
{"level":"info","ts":1656514391.1187196,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}
{"level":"info","ts":1656514391.123459,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514391.2733018,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Read admin password secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read admin password secret] **********************************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:31

-------------------------------------------------------------------------------
{"level":"info","ts":1656514392.286784,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-admin-password","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-admin-password","Parts":["secrets","awx-demo-admin-password"]}}
{"level":"info","ts":1656514392.644318,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Include broadcast websocket configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include broadcast websocket configuration tasks] *************
task path: /opt/ansible/roles/installer/tasks/main.yml:41

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified broadcast websocket secret configuration] ***
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656514392.6997364,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for specified broadcast websocket secret configuration"}
{"level":"info","ts":1656514392.826467,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for default broadcast websocket secret configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default broadcast websocket secret configuration] ***
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656514394.098465,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656514394.4652138,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Create broadcast websocket secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create broadcast websocket secret] ***************************
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:26

-------------------------------------------------------------------------------
{"level":"info","ts":1656514395.7890007,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656514395.7940722,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656514395.7992058,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514395.9262986,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Read broadcast websocket secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read broadcast websocket secret] *****************************
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:32

-------------------------------------------------------------------------------
{"level":"info","ts":1656514396.9118576,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-broadcast-websocket","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-broadcast-websocket","Parts":["secrets","awx-demo-broadcast-websocket"]}}
{"level":"info","ts":1656514397.3507445,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Include set_images tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include set_images tasks] ************************************
task path: /opt/ansible/roles/installer/tasks/main.yml:44

-------------------------------------------------------------------------------
{"level":"info","ts":1656514397.7680774,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Include database configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include database configuration tasks] ************************
task path: /opt/ansible/roles/installer/tasks/main.yml:47

-------------------------------------------------------------------------------
{"level":"info","ts":1656514397.846408,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for specified PostgreSQL configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified PostgreSQL configuration] ****************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656514397.966465,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for default PostgreSQL configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default PostgreSQL configuration] ******************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656514398.9796596,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}
{"level":"info","ts":1656514399.1473835,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for specified old PostgreSQL configuration secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified old PostgreSQL configuration secret] *****
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:19

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default old PostgreSQL configuration] **************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:28

-------------------------------------------------------------------------------
{"level":"info","ts":1656514399.294304,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for default old PostgreSQL configuration"}
{"level":"info","ts":1656514400.3911288,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-old-postgres-configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create Database configuration] *******************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:71

-------------------------------------------------------------------------------
{"level":"info","ts":1656514401.3200467,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Create Database configuration"}
{"level":"info","ts":1656514402.358127,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}
{"level":"info","ts":1656514402.3636377,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}
{"level":"info","ts":1656514402.3702097,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514402.5437484,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Read Database Configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read Database Configuration] *********************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:77

-------------------------------------------------------------------------------
{"level":"info","ts":1656514403.8506935,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-postgres-configuration","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-postgres-configuration","Parts":["secrets","awx-demo-postgres-configuration"]}}
{"level":"info","ts":1656514404.5861232,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Create Database if no database is specified"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create Database if no database is specified] *****************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:96

-------------------------------------------------------------------------------
{"level":"info","ts":1656514406.302074,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=StatefulSet, awx/awx-demo-postgres"}
{"level":"info","ts":1656514406.3088312,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=StatefulSet, awx/awx-demo-postgres"}
{"level":"info","ts":1656514406.3169386,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514406.3175225,"logger":"proxy","msg":"Watching child resource","kind":"apps/v1, Kind=StatefulSet","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656514406.3175895,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: apps/v1, Kind=StatefulSet"}
{"level":"info","ts":1656514406.4489024,"logger":"proxy","msg":"Cache miss: /v1, Kind=Service, awx/awx-demo-postgres"}
{"level":"info","ts":1656514406.464284,"logger":"proxy","msg":"Cache miss: /v1, Kind=Service, awx/awx-demo-postgres"}
{"level":"info","ts":1656514406.475667,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656514406.4760535,"logger":"proxy","msg":"Watching child resource","kind":"/v1, Kind=Service","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656514406.4760835,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: /v1, Kind=Service"}
{"level":"info","ts":1656514407.0634108,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Scale down Deployment for migration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Scale down Deployment for migration] *************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:102

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for presence of Deployment] ****************************
task path: /opt/ansible/roles/installer/tasks/scale_down_deployment.yml:3

-------------------------------------------------------------------------------
{"level":"info","ts":1656514407.2420993,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Check for presence of Deployment"}
{"level":"info","ts":1656514408.9700537,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=Deployment, awx/awx-demo"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Scale down Deployment for migration] *************************
task path: /opt/ansible/roles/installer/tasks/scale_down_deployment.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656514409.4453008,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Scale down Deployment for migration"}
{"level":"info","ts":1656514409.9223673,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5480330406766847676","EventData.Name":"installer : Wait for Database to initialize if managed DB"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Wait for Database to initialize if managed DB] ***************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:145

-------------------------------------------------------------------------------
{"level":"info","ts":1656514410.862179,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514417.4362078,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514423.8269668,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514430.4474628,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514437.2217798,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514444.9185786,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514452.5875804,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656514460.0878494,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}


```
After a few seconds, you should see the operator begin to create new resources:
```sh
(base) pradeep:~$kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                  READY   STATUS     RESTARTS   AGE
awx-demo-postgres-0   0/1     Init:0/1   0          97s
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
awx-demo-postgres   ClusterIP   None         <none>        5432/TCP   2m12s
(base) pradeep:~$
```

After some more time
```sh
(base) pradeep:~$kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                       READY   STATUS              RESTARTS   AGE
awx-demo-794c579d5-ttkrc   0/4     ContainerCreating   0          40s
awx-demo-postgres-0        1/1     Running             0          4m19s
(base) pradeep:~$kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator" -n awx 
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None           <none>        5432/TCP       4m25s
awx-demo-service    NodePort    10.98.36.228   <none>        80:30080/TCP   50s
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-demo-794c579d5-ttkrc                           4/4     Running   0          14m
awx-demo-postgres-0                                1/1     Running   0          17m
awx-operator-controller-manager-7594795b6b-qlmqc   2/2     Running   0          27m
(base) pradeep:~$
```
Once deployed, the AWX instance will be accessible by running

```sh
(base) pradeep:~$minikube service awx-demo-service --url -n awx                                                    
🏃  Starting tunnel for service awx-demo-service.
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.


```
By default, the admin user is admin and the password is available in the <resourcename>-admin-password secret. To retrieve the admin password, run:

```sh
(base) pradeep:~$kubectl get -n awx secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
ORQ3pqalx9V3vlEYnQmKt9CpGJJ6odTL%                                                                                               (base) pradeep:~$
```

This completes the most basic install of an AWX instance via this operator. Congratulations!!!