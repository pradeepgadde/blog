---
layout: single
title:  "Kubernetes Operator Pattern: AWX-Operator"
date:   2022-06-30 02:55:04 +0530
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
(base) pradeep:~$minikube start --cpus=4 --memory=6g --addons=ingress --driver=hyperkit                             
😄  minikube v1.25.2 on Darwin 12.4
✨  Using the hyperkit driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating hyperkit VM (CPUs=4, Memory=6144MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
🔎  Verifying ingress addon...
🌟  Enabled addons: storage-provisioner, default-storageclass, ingress
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~

```

```sh
(base) pradeep:~$kubectl get nodes   
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   4m31s   v1.23.3
(base) pradeep:~$
```
```sh

(base) pradeep:~$kubectl get pods -A
NAMESPACE       NAME                                       READY   STATUS      RESTARTS        AGE
ingress-nginx   ingress-nginx-admission-create-5wcjh       0/1     Completed   0               4m19s
ingress-nginx   ingress-nginx-admission-patch-j6kbm        0/1     Completed   0               4m19s
ingress-nginx   ingress-nginx-controller-cc8496874-7tbj5   1/1     Running     0               4m19s
kube-system     coredns-64897985d-cqzk6                    1/1     Running     0               4m19s
kube-system     etcd-minikube                              1/1     Running     0               4m31s
kube-system     kube-apiserver-minikube                    1/1     Running     0               4m33s
kube-system     kube-controller-manager-minikube           1/1     Running     0               4m31s
kube-system     kube-proxy-9jvmb                           1/1     Running     0               4m19s
kube-system     kube-scheduler-minikube                    1/1     Running     0               4m30s
kube-system     storage-provisioner                        1/1     Running     1 (3m47s ago)   4m29s
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
awx-operator-controller-manager-7594795b6b-7t952   0/2     ContainerCreating   0          18s
(base) pradeep:~$

```
After some time
```sh
(base) pradeep:~$kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-7594795b6b-7t952   2/2     Running   0          2m3s
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
  nodeport_port: 30081
  hostname: awx-demo.example.com
(base) pradeep:~$
```

Look at the kind, `AWX`. Its a custom resource (CRD)

Verify if the operator has installed any CRDs.
```sh
(base) pradeep:~$kubectl get crd
NAME                          CREATED AT
awxbackups.awx.ansible.com    2022-06-30T01:09:31Z
awxrestores.awx.ansible.com   2022-06-30T01:09:31Z
awxs.awx.ansible.com          2022-06-30T01:09:31Z

(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl describe crd awxs.awx.ansible.com
Name:         awxs.awx.ansible.com
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-06-30T01:09:31Z
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
    Time:         2022-06-30T01:09:31Z
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
    Time:            2022-06-30T01:09:31Z
  Resource Version:  839
  UID:               57291a70-8fd3-4256-bb6c-8e48e02f7e43
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
    Last Transition Time:  2022-06-30T01:09:31Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-06-30T01:09:31Z
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
{"level":"info","ts":1656551479.8186436,"logger":"cmd","msg":"Version","Go Version":"go1.16.9","GOOS":"linux","GOARCH":"amd64","ansible-operator":"v1.12.0","commit":"d3b2761afdb78f629a7eaf4461b0fb8ae3b02860"}
{"level":"info","ts":1656551479.819281,"logger":"cmd","msg":"Watching single namespace.","Namespace":"awx"}
{"level":"info","ts":1656551479.983489,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"127.0.0.1:8080"}
{"level":"info","ts":1656551479.9867737,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWX_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656551479.987127,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWXBACKUP_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656551479.9871945,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_AWXRESTORE_AWX_ANSIBLE_COM","default":2}
{"level":"info","ts":1656551479.9875116,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWX"}
{"level":"info","ts":1656551479.9878044,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWXBackup"}
{"level":"info","ts":1656551479.9880188,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"awx.ansible.com","Options.Version":"v1beta1","Options.Kind":"AWXRestore"}
{"level":"info","ts":1656551479.9955385,"logger":"proxy","msg":"Starting to serve","Address":"127.0.0.1:8888"}
{"level":"info","ts":1656551479.9958413,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
I0630 01:11:19.995943       8 leaderelection.go:243] attempting to acquire leader lease awx/awx-operator...
I0630 01:11:20.012467       8 leaderelection.go:253] successfully acquired lease awx/awx-operator
{"level":"info","ts":1656551480.0129783,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWXRestore"}
{"level":"info","ts":1656551480.0133452,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting Controller"}
{"level":"info","ts":1656551480.013377,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWXBackup"}
{"level":"info","ts":1656551480.0135717,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting Controller"}
{"level":"info","ts":1656551480.0131257,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656551480.0141025,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting Controller"}
{"level":"info","ts":1656551480.115646,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting workers","worker count":4}
{"level":"info","ts":1656551480.1157455,"logger":"controller-runtime.manager.controller.awxbackup-controller","msg":"Starting workers","worker count":4}
{"level":"info","ts":1656551480.1168225,"logger":"controller-runtime.manager.controller.awxrestore-controller","msg":"Starting workers","worker count":4}
{"level":"info","ts":1656551651.8780146,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Patching labels to AWX kind"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Patching labels to AWX kind] *********************************
task path: /opt/ansible/roles/installer/tasks/main.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656551653.7789667,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/apis/awx.ansible.com/v1beta1/namespaces/awx/awxs/awx-demo","Verb":"get","APIPrefix":"apis","APIGroup":"awx.ansible.com","APIVersion":"v1beta1","Namespace":"awx","Resource":"awxs","Subresource":"","Name":"awx-demo","Parts":["awxs","awx-demo"]}}
{"level":"info","ts":1656551653.9396234,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Include secret key configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include secret key configuration tasks] **********************
task path: /opt/ansible/roles/installer/tasks/main.yml:20

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified secret key configuration] ****************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656551653.9770892,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for specified secret key configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default secret key configuration] ******************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656551654.0799713,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for default secret key configuration"}
{"level":"info","ts":1656551656.025587,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656551656.259788,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Create secret key secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create secret key secret] ************************************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:25

-------------------------------------------------------------------------------
{"level":"info","ts":1656551657.1330822,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656551657.1374023,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-secret-key"}
{"level":"info","ts":1656551657.1415858,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656551657.1440074,"logger":"proxy","msg":"Watching child resource","kind":"/v1, Kind=Secret","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656551657.1440873,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: /v1, Kind=Secret"}
{"level":"info","ts":1656551657.265475,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Read secret key secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read secret key secret] **************************************
task path: /opt/ansible/roles/installer/tasks/secret_key_configuration.yml:31

-------------------------------------------------------------------------------
{"level":"info","ts":1656551658.3593879,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-secret-key","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-secret-key","Parts":["secrets","awx-demo-secret-key"]}}
{"level":"info","ts":1656551658.7293444,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Load LDAP CAcert certificate"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load LDAP CAcert certificate] ********************************
task path: /opt/ansible/roles/installer/tasks/main.yml:23

-------------------------------------------------------------------------------
{"level":"info","ts":1656551658.830105,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Load ldap bind password"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load ldap bind password] *************************************
task path: /opt/ansible/roles/installer/tasks/main.yml:28

-------------------------------------------------------------------------------
{"level":"info","ts":1656551658.92726,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Load bundle certificate authority certificate"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Load bundle certificate authority certificate] ***************
task path: /opt/ansible/roles/installer/tasks/main.yml:33

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include admin password configuration tasks] ******************
task path: /opt/ansible/roles/installer/tasks/main.yml:38

-------------------------------------------------------------------------------
{"level":"info","ts":1656551659.0182931,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Include admin password configuration tasks"}
{"level":"info","ts":1656551659.057363,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for specified admin password configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified admin password configuration] ************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656551659.1481063,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for default admin password configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default admin password configuration] **************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656551660.0125716,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}
{"level":"info","ts":1656551660.2378795,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Create admin password secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create admin password secret] ********************************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:25

-------------------------------------------------------------------------------
{"level":"info","ts":1656551661.2781322,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}
{"level":"info","ts":1656551661.2860236,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-admin-password"}
{"level":"info","ts":1656551661.291983,"logger":"proxy","msg":"Injecting owner reference"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read admin password secret] **********************************
task path: /opt/ansible/roles/installer/tasks/admin_password_configuration.yml:31

-------------------------------------------------------------------------------
{"level":"info","ts":1656551661.4300885,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Read admin password secret"}
{"level":"info","ts":1656551662.2890952,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-admin-password","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-admin-password","Parts":["secrets","awx-demo-admin-password"]}}
{"level":"info","ts":1656551662.6438525,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Include broadcast websocket configuration tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include broadcast websocket configuration tasks] *************
task path: /opt/ansible/roles/installer/tasks/main.yml:41

-------------------------------------------------------------------------------
{"level":"info","ts":1656551662.693613,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for specified broadcast websocket secret configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified broadcast websocket secret configuration] ***
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656551662.8171732,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for default broadcast websocket secret configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default broadcast websocket secret configuration] ***
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656551663.8193839,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656551664.0785222,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Create broadcast websocket secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create broadcast websocket secret] ***************************
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:26

-------------------------------------------------------------------------------
{"level":"info","ts":1656551665.0578918,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656551665.063082,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-broadcast-websocket"}
{"level":"info","ts":1656551665.0685573,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656551665.206693,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Read broadcast websocket secret"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read broadcast websocket secret] *****************************
task path: /opt/ansible/roles/installer/tasks/broadcast_websocket_configuration.yml:32

-------------------------------------------------------------------------------
{"level":"info","ts":1656551666.2865722,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-broadcast-websocket","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-broadcast-websocket","Parts":["secrets","awx-demo-broadcast-websocket"]}}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include set_images tasks] ************************************
task path: /opt/ansible/roles/installer/tasks/main.yml:44

-------------------------------------------------------------------------------
{"level":"info","ts":1656551666.7777162,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Include set_images tasks"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Include database configuration tasks] ************************
task path: /opt/ansible/roles/installer/tasks/main.yml:47

-------------------------------------------------------------------------------
{"level":"info","ts":1656551667.2651138,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Include database configuration tasks"}
{"level":"info","ts":1656551667.347047,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for specified PostgreSQL configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified PostgreSQL configuration] ****************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:2

-------------------------------------------------------------------------------
{"level":"info","ts":1656551667.4916499,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for default PostgreSQL configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default PostgreSQL configuration] ******************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656551668.6158075,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for specified old PostgreSQL configuration secret] *****
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:19

-------------------------------------------------------------------------------
{"level":"info","ts":1656551668.7733934,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for specified old PostgreSQL configuration secret"}
{"level":"info","ts":1656551668.9838223,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for default old PostgreSQL configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for default old PostgreSQL configuration] **************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:28

-------------------------------------------------------------------------------
{"level":"info","ts":1656551670.0687404,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-old-postgres-configuration"}
{"level":"info","ts":1656551671.009511,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Create Database configuration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create Database configuration] *******************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:71

-------------------------------------------------------------------------------
{"level":"info","ts":1656551674.531274,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}
{"level":"info","ts":1656551674.5407627,"logger":"proxy","msg":"Cache miss: /v1, Kind=Secret, awx/awx-demo-postgres-configuration"}
{"level":"info","ts":1656551674.5501308,"logger":"proxy","msg":"Injecting owner reference"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Read Database Configuration] *********************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:77

-------------------------------------------------------------------------------
{"level":"info","ts":1656551674.8235004,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Read Database Configuration"}
{"level":"info","ts":1656551676.601051,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/secrets/awx-demo-postgres-configuration","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"secrets","Subresource":"","Name":"awx-demo-postgres-configuration","Parts":["secrets","awx-demo-postgres-configuration"]}}
{"level":"info","ts":1656551677.0890338,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Create Database if no database is specified"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Create Database if no database is specified] *****************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:96

-------------------------------------------------------------------------------
{"level":"info","ts":1656551679.3154178,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=StatefulSet, awx/awx-demo-postgres"}
{"level":"info","ts":1656551679.323545,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=StatefulSet, awx/awx-demo-postgres"}
{"level":"info","ts":1656551679.3303406,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656551679.3316474,"logger":"proxy","msg":"Watching child resource","kind":"apps/v1, Kind=StatefulSet","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656551679.331736,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: apps/v1, Kind=StatefulSet"}
{"level":"info","ts":1656551679.4991903,"logger":"proxy","msg":"Cache miss: /v1, Kind=Service, awx/awx-demo-postgres"}
{"level":"info","ts":1656551679.5520222,"logger":"proxy","msg":"Cache miss: /v1, Kind=Service, awx/awx-demo-postgres"}
{"level":"info","ts":1656551679.7807493,"logger":"proxy","msg":"Injecting owner reference"}
{"level":"info","ts":1656551679.7903244,"logger":"proxy","msg":"Watching child resource","kind":"/v1, Kind=Service","enqueue_kind":"awx.ansible.com/v1beta1, Kind=AWX"}
{"level":"info","ts":1656551679.7904308,"logger":"controller-runtime.manager.controller.awx-controller","msg":"Starting EventSource","source":"kind source: /v1, Kind=Service"}
{"level":"info","ts":1656551680.4959095,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Scale down Deployment for migration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Scale down Deployment for migration] *************************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:102

-------------------------------------------------------------------------------

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Check for presence of Deployment] ****************************
task path: /opt/ansible/roles/installer/tasks/scale_down_deployment.yml:3

-------------------------------------------------------------------------------
{"level":"info","ts":1656551680.6904542,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Check for presence of Deployment"}
{"level":"info","ts":1656551682.6884406,"logger":"proxy","msg":"Cache miss: apps/v1, Kind=Deployment, awx/awx-demo"}
{"level":"info","ts":1656551682.948929,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Scale down Deployment for migration"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Scale down Deployment for migration] *************************
task path: /opt/ansible/roles/installer/tasks/scale_down_deployment.yml:11

-------------------------------------------------------------------------------
{"level":"info","ts":1656551683.41285,"logger":"logging_event_handler","msg":"[playbook task start]","name":"awx-demo","namespace":"awx","gvk":"awx.ansible.com/v1beta1, Kind=AWX","event_type":"playbook_on_task_start","job":"5233543046885435728","EventData.Name":"installer : Wait for Database to initialize if managed DB"}

--------------------------- Ansible Task StdOut -------------------------------

TASK [installer : Wait for Database to initialize if managed DB] ***************
task path: /opt/ansible/roles/installer/tasks/database_configuration.yml:145

-------------------------------------------------------------------------------
{"level":"info","ts":1656551687.5439122,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656551694.2677295,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656551701.1495914,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}
{"level":"info","ts":1656551708.2674649,"logger":"proxy","msg":"Read object from cache","resource":{"IsResourceRequest":true,"Path":"/api/v1/namespaces/awx/pods/awx-demo-postgres-0","Verb":"get","APIPrefix":"api","APIGroup":"","APIVersion":"v1","Namespace":"awx","Resource":"pods","Subresource":"","Name":"awx-demo-postgres-0","Parts":["pods","awx-demo-postgres-0"]}}


```

```sh
(base) pradeep:~$kubectl get awx -n awx
NAME       AGE
awx-demo   5m43s
(base) pradeep:~$
```

```sh

(base) pradeep:~$kubectl describe awx -n awx
Name:         awx-demo
Namespace:    awx
Labels:       app.kubernetes.io/component=awx
              app.kubernetes.io/managed-by=awx-operator
              app.kubernetes.io/name=awx-demo
              app.kubernetes.io/operator-version=0.23.0
              app.kubernetes.io/part-of=awx-demo
Annotations:  <none>
API Version:  awx.ansible.com/v1beta1
Kind:         AWX
Metadata:
  Creation Timestamp:  2022-06-30T01:14:09Z
  Generation:          1
  Managed Fields:
    API Version:  awx.ansible.com/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:conditions:
    Manager:      ansible-operator
    Operation:    Update
    Subresource:  status
    Time:         2022-06-30T01:14:09Z
    API Version:  awx.ansible.com/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:admin_user:
        f:create_preload_data:
        f:garbage_collect_secrets:
        f:hostname:
        f:image_pull_policy:
        f:loadbalancer_port:
        f:loadbalancer_protocol:
        f:nodeport_port:
        f:projects_persistence:
        f:projects_storage_access_mode:
        f:projects_storage_size:
        f:replicas:
        f:route_tls_termination_mechanism:
        f:service_type:
        f:task_privileged:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2022-06-30T01:14:09Z
    API Version:  awx.ansible.com/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .:
          f:app.kubernetes.io/component:
          f:app.kubernetes.io/managed-by:
          f:app.kubernetes.io/name:
          f:app.kubernetes.io/operator-version:
          f:app.kubernetes.io/part-of:
    Manager:         OpenAPI-Generator
    Operation:       Update
    Time:            2022-06-30T01:14:13Z
  Resource Version:  1206
  UID:               7e61824c-55d3-406b-8767-cfacff661928
Spec:
  admin_user:                       admin
  create_preload_data:              true
  garbage_collect_secrets:          false
  Hostname:                         awx-demo.example.com
  image_pull_policy:                IfNotPresent
  loadbalancer_port:                80
  loadbalancer_protocol:            http
  nodeport_port:                    30081
  projects_persistence:             false
  projects_storage_access_mode:     ReadWriteMany
  projects_storage_size:            8Gi
  Replicas:                         1
  route_tls_termination_mechanism:  Edge
  service_type:                     nodeport
  task_privileged:                  false
Status:
  Conditions:
    Last Transition Time:  2022-06-30T01:14:09Z
    Reason:                Running
    Status:                True
    Type:                  Running
Events:                    <none>
(base) pradeep:~$

```



After a few seconds, you should see the operator begin to create new resources:

```sh
(base) pradeep:~$kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                  READY   STATUS            RESTARTS   AGE
awx-demo-postgres-0   0/1     PodInitializing   0          113s
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
awx-demo-postgres   ClusterIP   None         <none>        5432/TCP   2m18s
(base) pradeep:~$
```

After some more time
```sh
(base) pradeep:~$kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
awx-demo-postgres   ClusterIP   None            <none>        5432/TCP       8m2s
awx-demo-service    NodePort    10.109.91.153   <none>        80:30081/TCP   4m12s
(base) pradeep:~$kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx
NAME                       READY   STATUS              RESTARTS   AGE
awx-demo-794c579d5-cr89z   0/4     ContainerCreating   0          4m10s
awx-demo-postgres-0        1/1     Running             0          8m4s
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-demo-794c579d5-cr89z                           4/4     Running   0          9m16s
awx-demo-postgres-0                                1/1     Running   0          13m
awx-operator-controller-manager-7594795b6b-7t952   2/2     Running   0          18m
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl describe pods awx-demo-794c579d5-cr89z -n awx
Name:         awx-demo-794c579d5-cr89z
Namespace:    awx
Priority:     0
Node:         minikube/172.16.30.10
Start Time:   Thu, 30 Jun 2022 06:48:33 +0530
Labels:       app.kubernetes.io/component=awx
              app.kubernetes.io/managed-by=awx-operator
              app.kubernetes.io/name=awx-demo
              app.kubernetes.io/part-of=awx-demo
              app.kubernetes.io/version=21.2.0
              pod-template-hash=794c579d5
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/awx-demo-794c579d5
Containers:
  redis:
    Container ID:  docker://942a19dd9c890b49584cbcfcf59a730a2d5ad5a34cb3e29cc46c78a50134a253
    Image:         docker.io/redis:latest
    Image ID:      docker-pullable://redis@sha256:d581aded52343c461f32e4a48125879ed2596291f4ea4baa7e3af0ad1e56feed
    Port:          <none>
    Host Port:     <none>
    Args:
      redis-server
      /etc/redis.conf
    State:          Running
      Started:      Thu, 30 Jun 2022 06:48:52 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        50m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /data from awx-demo-redis-data (rw)
      /etc/redis.conf from awx-demo-redis-config (ro,path="redis.conf")
      /var/run/redis from awx-demo-redis-socket (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pm4zh (ro)
  awx-demo-web:
    Container ID:  docker://f1a35db3b415a047fc59e728a8cfa41b6f3224bdea9f24667aba5ddd7dbc297c
    Image:         quay.io/ansible/awx:21.2.0
    Image ID:      docker-pullable://quay.io/ansible/awx@sha256:121a8d962fc2d3d439a1ef92ad77411b1b42b3b9cf4027f2749a176ac9a8a4f2
    Port:          8052/TCP
    Host Port:     0/TCP
    Args:
      /usr/bin/launch_awx.sh
    State:          Running
      Started:      Thu, 30 Jun 2022 06:51:53 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:     100m
      memory:  128Mi
    Environment:
      MY_POD_NAMESPACE:  awx (v1:metadata.namespace)
      UWSGI_MOUNT_PATH:  /
    Mounts:
      /etc/nginx/nginx.conf from awx-demo-nginx-conf (ro,path="nginx.conf")
      /etc/tower/SECRET_KEY from awx-demo-secret-key (ro,path="SECRET_KEY")
      /etc/tower/conf.d/credentials.py from awx-demo-application-credentials (ro,path="credentials.py")
      /etc/tower/conf.d/execution_environments.py from awx-demo-application-credentials (ro,path="execution_environments.py")
      /etc/tower/conf.d/ldap.py from awx-demo-application-credentials (ro,path="ldap.py")
      /etc/tower/settings.py from awx-demo-settings (ro,path="settings.py")
      /var/lib/awx/projects from awx-demo-projects (rw)
      /var/lib/awx/rsyslog from rsyslog-dir (rw)
      /var/run/awx-rsyslog from rsyslog-socket (rw)
      /var/run/redis from awx-demo-redis-socket (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pm4zh (ro)
      /var/run/supervisor from supervisor-socket (rw)
  awx-demo-task:
    Container ID:  docker://cb06b3328d9f357aa94fd3d15fe8ba738e0d87b3bbd3d239dd61db6c87d5ac0d
    Image:         quay.io/ansible/awx:21.2.0
    Image ID:      docker-pullable://quay.io/ansible/awx@sha256:121a8d962fc2d3d439a1ef92ad77411b1b42b3b9cf4027f2749a176ac9a8a4f2
    Port:          <none>
    Host Port:     <none>
    Args:
      /usr/bin/launch_awx_task.sh
    State:          Running
      Started:      Thu, 30 Jun 2022 06:51:53 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:     100m
      memory:  128Mi
    Environment:
      SUPERVISOR_WEB_CONFIG_PATH:  /etc/supervisord.conf
      AWX_SKIP_MIGRATIONS:         1
      MY_POD_UID:                   (v1:metadata.uid)
      MY_POD_IP:                    (v1:status.podIP)
      MY_POD_NAMESPACE:            awx (v1:metadata.namespace)
    Mounts:
      /etc/receptor/receptor.conf from awx-demo-receptor-config (ro,path="receptor.conf")
      /etc/tower/SECRET_KEY from awx-demo-secret-key (ro,path="SECRET_KEY")
      /etc/tower/conf.d/credentials.py from awx-demo-application-credentials (ro,path="credentials.py")
      /etc/tower/conf.d/execution_environments.py from awx-demo-application-credentials (ro,path="execution_environments.py")
      /etc/tower/conf.d/ldap.py from awx-demo-application-credentials (ro,path="ldap.py")
      /etc/tower/settings.py from awx-demo-settings (ro,path="settings.py")
      /var/lib/awx/projects from awx-demo-projects (rw)
      /var/lib/awx/rsyslog from rsyslog-dir (rw)
      /var/run/awx-rsyslog from rsyslog-socket (rw)
      /var/run/receptor from receptor-socket (rw)
      /var/run/redis from awx-demo-redis-socket (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pm4zh (ro)
      /var/run/supervisor from supervisor-socket (rw)
  awx-demo-ee:
    Container ID:  docker://30f971bb4f5ffe8ac4a42799160230be4a6d081c99aa8001db5195a693192757
    Image:         quay.io/ansible/awx-ee:latest
    Image ID:      docker-pullable://quay.io/ansible/awx-ee@sha256:94a2c8b5568ae5bfdb378afe26bdb823e6fda62e1bb0bb8f59816216436a825f
    Port:          <none>
    Host Port:     <none>
    Args:
      receptor
      --config
      /etc/receptor/receptor.conf
    State:          Running
      Started:      Thu, 30 Jun 2022 06:56:33 +0530
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /etc/receptor/receptor.conf from awx-demo-receptor-config (ro,path="receptor.conf")
      /var/lib/awx/projects from awx-demo-projects (rw)
      /var/run/receptor from receptor-socket (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-pm4zh (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  awx-demo-application-credentials:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  awx-demo-app-credentials
    Optional:    false
  awx-demo-secret-key:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  awx-demo-secret-key
    Optional:    false
  awx-demo-settings:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      awx-demo-awx-configmap
    Optional:  false
  awx-demo-nginx-conf:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      awx-demo-awx-configmap
    Optional:  false
  awx-demo-redis-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      awx-demo-awx-configmap
    Optional:  false
  awx-demo-redis-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  awx-demo-redis-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  supervisor-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  rsyslog-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  receptor-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  rsyslog-dir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  awx-demo-receptor-config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      awx-demo-awx-configmap
    Optional:  false
  awx-demo-projects:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-pm4zh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  22m   default-scheduler  Successfully assigned awx/awx-demo-794c579d5-cr89z to minikube
  Normal  Pulling    22m   kubelet            Pulling image "docker.io/redis:latest"
  Normal  Pulled     22m   kubelet            Successfully pulled image "docker.io/redis:latest" in 16.962689372s
  Normal  Created    22m   kubelet            Created container redis
  Normal  Started    22m   kubelet            Started container redis
  Normal  Pulling    22m   kubelet            Pulling image "quay.io/ansible/awx:21.2.0"
  Normal  Pulled     19m   kubelet            Successfully pulled image "quay.io/ansible/awx:21.2.0" in 3m0.709836248s
  Normal  Started    19m   kubelet            Started container awx-demo-web
  Normal  Created    19m   kubelet            Created container awx-demo-web
  Normal  Pulled     19m   kubelet            Container image "quay.io/ansible/awx:21.2.0" already present on machine
  Normal  Created    19m   kubelet            Created container awx-demo-task
  Normal  Started    19m   kubelet            Started container awx-demo-task
  Normal  Pulling    19m   kubelet            Pulling image "quay.io/ansible/awx-ee:latest"
  Normal  Pulled     14m   kubelet            Successfully pulled image "quay.io/ansible/awx-ee:latest" in 4m39.602052148s
  Normal  Created    14m   kubelet            Created container awx-demo-ee
  Normal  Started    14m   kubelet            Started container awx-demo-ee
(base) pradeep:~$
```
Once deployed, the AWX instance will be accessible by running

```sh
(base) pradeep:~$minikube service awx-demo-service --url -n awx
http://172.16.30.10:30081
(base) pradeep:~$
```

Let us access the REST API of the AWX and test the `ping` by pointing the service URL followed by `/api/v2/ping`.

![AWX Rest API]({{ site.url }}{{ site.baseurl }}/assets/images/awx-rest-api-ping-1.png)

If we look at the `X-API-Node` in the header,  or `node` under AWX Instances, we can see our Pod Name `awx-demo-794c579d5-cr89z`.

By default, the admin user is `admin` and the password is available in the `<resourcename>-admin-password` secret. To retrieve the admin password, run:

```sh
(base) pradeep:~$kubectl get -n awx secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode
Y7SH2eQ4EdcREYVapiYLaw8xTRbxuWDh%                                                                                             (base) pradeep:~$
```
Login with username as `admin` and password as `Y7SH2eQ4EdcREYVapiYLaw8xTRbxuWDh`.

![AWX Login]({{ site.url }}{{ site.baseurl }}/assets/images/awx-login.png)

Explore the AWX Dashboard
![AWX Dashboard]({{ site.url }}{{ site.baseurl }}/assets/images/awx-dashboard.png)

and Projects
![AWX Project]({{ site.url }}{{ site.baseurl }}/assets/images/awx-projects.png)

This completes the most basic install of an AWX instance via the Kubernetes operator. Congratulations!!!