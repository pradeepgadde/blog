---

layout: single
title:  "Configuring Clusters with Anthos Config Management"
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
# Configuring Clusters with Anthos Config Management

Kubernetes clusters are configured using manifests, or **configs**, written in YAML or JSON. These configurations include important Kubernetes objects such as Namespaces, ClusterRoles, ClusterRoleBindings, Roles, RoleBindings, PodSecurityPolicy, NetworkPolicy, and ResourceQuotas, etc.

These declarative configs can be applied by hand or with automated tooling.  The preferred method is to use an automated process to establish and maintain a consistently managed environment from the beginning.

**Anthos Config Management** is a solution to help manage these resources in a *configuration-as-code* like manner. Anthos Config Management utilizes a version-controlled **Git repository** (repo) for configuration storage along with **configuration operators** which apply configs to selected clusters.

Anthos Config Management allows you to easily manage the configuration of many clusters. At the heart of this process are the Git repositories that store the configurations to be applied on the clusters.

- Install the Config Management Operator and the nomos command-line tool
- Set up your config repo in Cloud Source Repositories
- Connect your GKE clusters to the config repo
- Examine the configs in your clusters and repo
- Filter application of configs by Namespace
- Review automated drift management
- Update a config in the repo

## Complete and verify the lab setup

- A GKE cluster named **gke** has been created and  registered. Anthos Service Mesh has been installed, as has the  [   Online Boutique demo application.](https://github.com/GoogleCloudPlatform/microservices-demo)
- An open source Kubernetes cluster named **onprem-connect**   has been created. Istio has been installed, as has the Online Boutique   application.

Set up the Cloud Shell environment for command-line access to your clusters:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-076ef386825e.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ export PROJECT_ID=$(gcloud config get-value project)
export SHELL_IP=$(curl -s api.ipify.org)
export KUBECONFIG=~/.kube/config
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")

gcloud compute firewall-rules create shell-to-onprem \
    --network=onprem-k8s-local \
    --allow tcp \
    --source-ranges $SHELL_IP

gsutil cp gs://$PROJECT_ID-kops-onprem/config \
    ~/.kube/config
Your active configuration is: [cloudshell-8425]
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-076ef386825e/global/firewalls/shell-to-onprem].                               
Creating firewall...done.                                                                                                                                                          
NAME: shell-to-onprem
NETWORK: onprem-k8s-local
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp
DENY: 
DISABLED: False
Copying gs://qwiklabs-gcp-01-076ef386825e-kops-onprem/config...
/ [1 files][  5.5 KiB/  5.5 KiB]                                                
Operation completed over 1 objects/5.5 KiB.                                      
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ 
```



Set `kubectl` to use the context for the **onprem** cluster:

```sh
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ kubectx onprem.k8s.local
Switched to context "onprem.k8s.local".
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ 
```



Create a JSON Web Token (JWTs) for the `remote-admin-sa` secret that represents the Kubernetes Service Account that will be used for GKE Connect:

```sh
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ kubectl create token remote-admin-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6Ik9EOXBoTlE0T3NHNHdwM3NkYnRzTk1CaTNnYzJoWDlBOVBxRG5razRyZHcifQ.eyJhdWQiOlsia3ViZXJuZXRlcy5zdmMuZGVmYXVsdCJdLCJleHAiOjE3MjE4NDM5NTMsImlhdCI6MTcyMTg0MDM1MywiaXNzIjoiaHR0cHM6Ly9hcGkuaW50ZXJuYWwub25wcmVtLms4cy5sb2NhbCIsImt1YmVybmV0ZXMuaW8iOnsibmFtZXNwYWNlIjoiZGVmYXVsdCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJyZW1vdGUtYWRtaW4tc2EiLCJ1aWQiOiIwNzY4YmQ3MC1jZTlkLTQxM2MtYjU0Yi0wMWFlZjJiNGY0ZmEifX0sIm5iZiI6MTcyMTg0MDM1Mywic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6cmVtb3RlLWFkbWluLXNhIn0.iHvoL18yghJEuG60liy35NjO9L-aiQSpLfazcMwtKA56bAwZNx7QgJvYwqVCgdLk-_opUtK5WCwhHPR7pnb07E_NSfKrOLClBae70n82533d5AH1sAF7sTcny3naGNorml9tCXI51tOC1sp_esqnJOMYqc3zbC30oiRIYh4ImZWadGgWkoJcg1Hody7-suoJd6DLxcRe8fHkR3RGbodw0XTkkeHSqsgsMFs52tVSMd5Gt5OuR6mjTs1TMofo5DC3I7yLAO_UDMVYIU9tf3sBojErr1IgDvhyo0jzOqWtZVjBNsPOtyvIbmjLbQ3xjFnrONjoXxIetyldMoUjoRcaKw
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ 
```



Select the token contents in the Cloud Shell (this will automatically copy the contents).

Go to **Navigation > Kubernetes Engine > Clusters**, scroll to the right, click on the 3 dots to open the dropdown menu of the **the onprem-connect** cluster row, and click on the **Log in** option.

When prompted, select **Token** as the authentication type, and paste the previously copied token, then click **Login**.

You should now see two clusters listed with green checkmarks which indicates both clusters are registered successfully.

Visit the **Gateways, Services & Ingress** page, select **Services** tab and find the **frontend-external** service address for each cluster.

- Remove the filters(if any) to see the frontend-external service addresses.
- Visit those addresses in new browser tabs and verify that separate, independent applications are up and running in each cluster.

## Install the Config Management Operator and the nomos command-line tool

The Config Management **Operator** is a Kubernetes controller that manages Anthos Config Management in a Kubernetes cluster. In this task, you install the Operator as a system workload on both clusters. You also install the `nomos` command-line tool which helps you to understand the state of Anthos Config Management in your clusters.

### Install the Config Management Operator on the gke cluster

```sh
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ ZONE=us-west1-a
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ gcloud container clusters get-credentials gke --zone $ZONE --project $PROJECT_ID
kubectx gke=gke_${PROJECT_ID}_${ZONE}_gke
kubectx gke
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
Context "gke_qwiklabs-gcp-01-076ef386825e_us-west1-a_gke" renamed to "gke".
Switched to context "gke".
student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ export LAB_DIR=$HOME/acm-lab
mkdir $LAB_DIR
cd $LAB_DIR
gsutil cp gs://config-management-release/released/latest/config-management-operator.yaml config-management-operator.yaml
Copying gs://config-management-release/released/latest/config-management-operator.yaml...
/ [1 files][ 22.0 KiB/ 22.0 KiB]                                                
Operation completed over 1 objects/22.0 KiB.                                     
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```yaml
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ cat config-management-operator.yaml 
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.5.0
  creationTimestamp: null
  name: configmanagements.configmanagement.gke.io
spec:
  group: configmanagement.gke.io
  names:
    kind: ConfigManagement
    listKind: ConfigManagementList
    plural: configmanagements
    singular: configmanagement
  scope: Cluster
  versions:
  - name: v1
    schema:
      openAPIV3Schema:
        description: ConfigManagement is the Schema for the ConfigManagement API.
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            properties:
              name:
                pattern: config-management
                type: string
            type: object
          spec:
            description: ConfigManagementSpec defines the desired state of ConfigManagement.
            properties:
              ConfigSyncDisableFSWatcher:
                description: ConfigSyncDisableFSWatcher provides the ability to disable
                  the fs-watcher process.  This field is intentionally left hidden/undocumented
                  since it is only meant to be used by customers who have very large
                  repositories. Optional.
                type: boolean
              ConfigSyncLogLevel:
                description: ConfigSyncLogLevel overrides the logging verbosity for
                  all ConfigSync pods. This field is intentionally left hidden/undocumented
                  since it is really only used to gather extra logs for support cases.
                type: integer
              binauthz:
                description: 'Deprecated: Does nothing. binauthz can no longer be
                  enabled/disabled with the ConfigManagement resource; the software
                  is available as a standalone: https://cloud.google.com/binary-authorization'
                properties:
                  enabled:
                    description: 'Enable or disable BinAuthz.  Default: false.'
                    type: boolean
                  policyRef:
                    description: PolicyRef is a reference to the BinAuthz policy which
                      will be evaluated. Required if BinAuthz is enabled.
                    properties:
                      gkeCluster:
                        description: BinAuthz policy associated with this GKE-on-GCP
                          cluster.
                        properties:
                          location:
                            description: Location of this cluster
                            type: string
                          name:
                            description: The name of this cluster according to GKE.
                              This is not necessarily the same as the hub membership
                              name.
                            type: string
                          project:
                            description: The name of the GCP project containing this
                              cluster
                            type: string
                        type: object
                    type: object
                type: object
              channel:
                description: 'Channel specifies a channel that can be used to resolve
                  a specific addon, eg: stable It will be ignored if Version is specified'
                type: string
              clusterName:
                description: ClusterName, if defined, sets the name for this cluster.  If
                  unset, the cluster is considered to be unnamed, and cannot use ClusterSelectors.
                type: string
              configConnector:
                description: 'Deprecated: Does nothing.  ConfigConnector can no longer
                  be enabled/disabled with the ConfigManagement resource; the software
                  is available as a standalone: https://cloud.google.com/config-connector'
                properties:
                  enabled:
                    description: 'Enable or disable the Config Connector.  Default:
                      false.'
                    type: boolean
                type: object
              enableLegacyFields:
                description: EnableLegacyFields instructs the operator to use spec.git
                  for generating a RootSync resource in MultiRepo mode. Note that
                  this should only be set to true if spec.enableMultiRepo is set to
                  true.
                type: boolean
              enableMultiRepo:
                description: EnableMultiRepo instructs the operator to enable Multi
                  Repo mode for Config Sync.
                type: boolean
              git:
                description: Git contains configuration specific to importing policies
                  from a Git repo.
                properties:
                  gcpServiceAccountEmail:
                    description: 'GCPServiceAccountEmail specifies the GCP service
                      account used to annotate the Config Sync Kubernetes Service
                      Account. Note: The field is used when secretType: gcpServiceAccount.'
                    type: string
                  policyDir:
                    description: 'PolicyDir is the absolute path of the directory
                      that contains the local policy.  Default: the root directory
                      of the repo.'
                    type: string
                  proxy:
                    description: Proxy is a struct that contains options for configuring
                      access to the Git repo via a proxy. Only has an effect when
                      secretType is one of ("cookiefile", "none").  Optional.
                    properties:
                      httpProxy:
                        description: HTTPProxy defines a HTTP_PROXY env variable used
                          to access the Git repo.  If both HTTPProxy and HTTPSProxy
                          are specified, HTTPProxy will be ignored. Optional.
                        type: string
                      httpsProxy:
                        description: HTTPSProxy defines a HTTPS_PROXY env variable
                          used to access the Git repo.  If both HTTPProxy and HTTPSProxy
                          are specified, HTTPProxy will be ignored. Optional.
                        type: string
                    type: object
                  secretType:
                    description: SecretType is the type of secret configured for access
                      to the Git repo. Must be one of ssh, cookiefile, gcenode, token,
                      gcpserviceaccount or none. Required. The validation of this
                      is case-sensitive.
                    pattern: ^(ssh|cookiefile|gcenode|gcpserviceaccount|token|none)$
                    type: string
                  syncBranch:
                    description: 'SyncBranch is the branch to sync from.  Default:
                      "master".'
                    type: string
                  syncRepo:
                    pattern: ^(((https?|git|ssh):\/\/)|git@)
                    type: string
                  syncRev:
                    description: 'SyncRev is the git revision (tag or hash) to check
                      out. Default: HEAD.'
                    type: string
                  syncWait:
                    description: 'SyncWaitSeconds is the time duration in seconds
                      between consecutive syncs.  Default: 15 seconds. Note that SyncWaitSecs
                      is not a time.Duration on purpose. This provides a reminder
                      to developers that customers specify this value using using
                      integers like "3" in their ConfigManagement YAML. However, time.Duration
                      is at a nanosecond granularity, and it''s easy to introduce
                      a bug where it looks like the code is dealing with seconds but
                      its actually nanoseconds (or vice versa).'
                    type: integer
                type: object
              hierarchyController:
                description: Hierarchy Controller enables HierarchyController components
                  as recognized by the "hierarchycontroller.configmanagement.gke.io"
                  label set to "true".
                properties:
                  enableHierarchicalResourceQuota:
                    description: 'HierarchicalResourceQuota enforces resource quota
                      in a hierarchical fashion: a resource quota set for one namespace
                      provides constraints that limit aggregate resource consumption
                      for that namespace and all its descendants. Disabling this will
                      not delete user created hrq CRs, but will delete all the intermediate
                      resources created by HRQ (specifically the resource quota singletons),
                      which are labeled with hierarchycontroller.configmanagement.gke.io/hrq
                      for easier cleanup.'
                    type: boolean
                  enablePodTreeLabels:
                    description: PodTreeLabels copies the tree labels from namespaces
                      to pods, allowing any system that uses pod logs (such as Stackdriver
                      logging) to inspect the hierarchy.
                    type: boolean
                  enabled:
                    description: 'Enable or disable the Hierarchy Controller.  Default:
                      false.'
                    type: boolean
                type: object
              importer:
                description: Importer allows one to override the existing resource
                  requirements for the importer pod
                properties:
                  limits:
                    additionalProperties:
                      anyOf:
                      - type: integer
                      - type: string
                      pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                      x-kubernetes-int-or-string: true
                    description: 'Limits describes the maximum amount of compute resources
                      allowed. More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/'
                    type: object
                  requests:
                    additionalProperties:
                      anyOf:
                      - type: integer
                      - type: string
                      pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                      x-kubernetes-int-or-string: true
                    description: 'Requests describes the minimum amount of compute
                      resources required. If Requests is omitted for a container,
                      it defaults to Limits if that is explicitly specified, otherwise
                      to an implementation-defined value. More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/'
                    type: object
                type: object
              metricsGCPServiceAccountEmail:
                description: MetricsGCPServiceAccountEmail specifies a Google Service
                  Account (GSA) Email with the Monitoring Metric Writer (roles/monitoring.metricWriter)
                  IAM role. This GSA Email is used to annotate the `default` Kubernetes
                  ServiceAccount under the `config-management-monitoring` namespace,
                  which allows Config Sync to export metrics to Cloud Monitoring.
                type: string
              patches:
                items:
                  type: object
                type: array
                x-kubernetes-preserve-unknown-fields: true
              policyController:
                description: 'Deprecated: Does nothing. Policy Controller can no longer
                  be enabled/disabled with the ConfigManagement resource; see https://cloud.google.com/anthos-config-management/docs/how-to/installing-policy-controller
                  for supported ways of enabling Policy Controller.'
                properties:
                  auditIntervalSeconds:
                    description: AuditIntervalSeconds. The number of seconds between
                      audit runs. Defaults to 60 seconds. To disable audit, set this
                      to 0.
                    format: int64
                    type: integer
                  enabled:
                    description: 'Enable or disable the Policy Controller.  Default:
                      false.'
                    type: boolean
                  exemptableNamespaces:
                    description: ExemptableNamespaces. The namespaces in this list
                      are able to have the admission.gatekeeper.sh/ignore label set.
                      When the label is set, Policy Controller will not be called
                      for that namespace or any resources contained in it. `gatekeeper-system`
                      is always exempted.
                    items:
                      type: string
                    type: array
                  logDeniesEnabled:
                    description: 'LogDeniesEnabled.  If true, Policy Controller will
                      log all denies and dryrun failures.  No effect unless policyController
                      is enabled.  Default: false.'
                    type: boolean
                  monitoring:
                    description: Monitoring specifies the configuration of monitoring.
                    properties:
                      backends:
                        items:
                          type: string
                        type: array
                    type: object
                  mutation:
                    description: Mutation specifies the configuration of mutation.
                      This is a preview feature and may change before becoming generally
                      available.
                    properties:
                      enabled:
                        description: 'Enable or disable mutation in policy controller.
                          If true, mutation CRDs, webhook and controller will be deployed
                          to the cluster. Default: false.'
                        type: boolean
                    type: object
                  referentialRulesEnabled:
                    description: 'ReferentialRulesEnabled.  If true, Policy Controller
                      will allow `data.inventory` references in the contents of ConstraintTemplate
                      Rego.  No effect unless policyController is enabled.  Default:
                      false.'
                    type: boolean
                  templateLibraryInstalled:
                    description: 'TemplateLibraryInstalled.  If true, a set of default
                      ConstraintTemplates will be deployed to the cluster. ConstraintTemplates
                      will not be deployed if this is explicitly set to false or if
                      policyController is not enabled. Default: true.'
                    type: boolean
                type: object
              preventDrift:
                description: 'preventDrift, if set to `true`, enables the Config Sync
                  admission webhook to prevent drifts. If set to `false`, disables
                  the Config Sync admission webhook and does not prevent drifts. Default:
                  false. Config Sync always corrects drifts no matter the value of
                  preventDrift.'
                type: boolean
              sourceFormat:
                description: "SourceFormat specifies how the repository is formatted.
                  See documentation for specifics of what these options do. \n Must
                  be one of hierarchy, unstructured. Optional. Set to hierarchy if
                  not specified. \n The validation of this is case-sensitive."
                pattern: ^(hierarchy|unstructured|)$
                type: string
              syncer:
                description: Syncer allows one to override the existing resource requirements
                  for the syncer pod
                properties:
                  limits:
                    additionalProperties:
                      anyOf:
                      - type: integer
                      - type: string
                      pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                      x-kubernetes-int-or-string: true
                    description: 'Limits describes the maximum amount of compute resources
                      allowed. More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/'
                    type: object
                  requests:
                    additionalProperties:
                      anyOf:
                      - type: integer
                      - type: string
                      pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                      x-kubernetes-int-or-string: true
                    description: 'Requests describes the minimum amount of compute
                      resources required. If Requests is omitted for a container,
                      it defaults to Limits if that is explicitly specified, otherwise
                      to an implementation-defined value. More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/'
                    type: object
                type: object
              version:
                description: Version specifies the exact addon version to be deployed,
                  eg 1.2.3 It should not be specified if Channel is specified
                type: string
            type: object
          status:
            description: ConfigManagementStatus defines the observed state of ConfigManagement.
            properties:
              configManagementVersion:
                description: ConfigManagementVersion is the semantic version number
                  of the config management system enforced by the currently running
                  config management operator.
                type: string
              errors:
                items:
                  type: string
                type: array
              healthy:
                type: boolean
              phase:
                type: string
            required:
            - healthy
            type: object
        required:
        - metadata
        - spec
        type: object
    served: true
    storage: true
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
---
apiVersion: v1
kind: Namespace
metadata:
  name: config-management-system
  labels:
    configmanagement.gke.io/system: "true"
---
apiVersion: v1
kind: Namespace
metadata:
  name: config-management-monitoring
  labels:
    configmanagement.gke.io/system: "true"
---
# The Nomos system creates RBAC rules, so it requires
# full cluster-admin access. Thus, the operator needs
# to be able to grant tha permission to the installed
# Nomos components.
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: config-management-operator
  name: config-management-operator
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: config-management-operator
  name: config-management-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: config-management-operator
subjects:
- kind: ServiceAccount
  name: config-management-operator
  namespace: config-management-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: config-management-operator
  name: config-management-operator
  namespace: config-management-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-management-operator
  namespace: config-management-system
  labels:
    k8s-app: config-management-operator
spec:
  # specifying replicas explicitly would help to enforce the intended
  # value when the file is applied.
  replicas: 1
  strategy:
    type: Recreate
    # must be null due to 3-way merge, as
    # rollingUpdate added to the resource by default by the APIServer
    rollingUpdate: null
  selector:
    matchLabels:
      k8s-app: config-management-operator
      component: config-management-operator
  template:
    metadata:
      labels:
        k8s-app: config-management-operator
        component: config-management-operator
    spec:
      containers:
      - command:
        - /manager
        - --private-registry=
        name: manager
        image: gcr.io/config-management-release/config-management-operator:v1.18.2-rc.1
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        envFrom:
        - configMapRef:
            name: operator-environment-options
            optional: true
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
      serviceAccount: config-management-operator
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```



```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl apply -f config-management-operator.yaml
customresourcedefinition.apiextensions.k8s.io/configmanagements.configmanagement.gke.io created
namespace/config-management-system created
namespace/config-management-monitoring created
clusterrole.rbac.authorization.k8s.io/config-management-operator created
clusterrolebinding.rbac.authorization.k8s.io/config-management-operator created
serviceaccount/config-management-operator created
deployment.apps/config-management-operator created
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

Use the GCP Console to verify that a system workload called **config-management-operator** has been created. Visit **Navigation > Kubernetes Engine > Workloads**.

### Install the Config Management Operator on the onprem cluster

1. Switch contexts, and apply the configuration file to the **onprem** cluster:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx onprem.k8s.local
kubectl apply -f config-management-operator.yaml
Switched to context "onprem.k8s.local".
customresourcedefinition.apiextensions.k8s.io/configmanagements.configmanagement.gke.io created
namespace/config-management-system created
namespace/config-management-monitoring created
clusterrole.rbac.authorization.k8s.io/config-management-operator created
clusterrolebinding.rbac.authorization.k8s.io/config-management-operator created
serviceaccount/config-management-operator created
deployment.apps/config-management-operator created
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

Using the console, or the `kubectl` command, verify that the Config Management Operator has been deployed to the **onprem** cluster.

### Install the nomos command-line tool in Cloud Shell

1. In Cloud Shell, download the `nomos` command-line tool:
2. Use `nomos status` to check if Anthos Config Management is properly installed and configured:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ cd $LAB_DIR
gsutil cp gs://config-management-release/released/latest/linux_amd64/nomos nomos
chmod +x ./nomos
Copying gs://config-management-release/released/latest/linux_amd64/nomos...
\ [1 files][ 70.9 MiB/ 70.9 MiB]                                                
Operation completed over 1 objects/70.9 MiB.                                     
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ ./nomos status
Connecting to clusters...

gke
  --------------------
     Failed to get the RootSync CRD: customresourcedefinitions.apiextensions.k8s.io "rootsyncs.configsync.gke.io" not found

*onprem.k8s.local
  --------------------
     Failed to get the RootSync CRD: customresourcedefinitions.apiextensions.k8s.io "rootsyncs.configsync.gke.io" not found
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

In this case, config management is installed but not yet configured for your clusters.

When `nomos status` reports an error, it also shows any additional error text available to help diagnose the problem under `Config Management Errors`.

You will correct the issues you see here in later steps.



## Set up your Anthos Config Management repository

Anthos Config Management requires you to store your configurations in a Git repository. In this task, you set up that repository.

Anthos Config Management supports any Git repo including GitHub and Google Cloud Source Repositories. In this lab, you will use **Cloud Source Repositories**.

### Create a new local config repo

1. Set the username and email address for your Git activities:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ export GCLOUD_EMAIL=$(gcloud config get-value account)
echo $GCLOUD_EMAIL
echo $USER

git config --global user.email "$GCLOUD_EMAIL"
git config --global user.name "$USER"
Your active configuration is: [cloudshell-8425]
student-00-ac340a727c48@qwiklabs.net
student_00_ac340a727c48
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

Get sample config files for the lab:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
cd ./training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 65054, done.
remote: Total 65054 (delta 0), reused 0 (delta 0), pack-reused 65054
Receiving objects: 100% (65054/65054), 697.18 MiB | 24.47 MiB/s, done.
Resolving deltas: 100% (41575/41575), done.
Updating files: 100% (12864/12864), done.
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
```

Take a moment to review the structure of the **config** directory. Click the **Open Editor** button in Cloud Shell, then in the explorer section of the editor, drill down into **acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config**.

Take a minute to review the subdirectories and the contents of the config files you find.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ ls
cluster  namespaces  README.md  system
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ ls cluster/
clusterrolebinding-namespace-readers.yaml  clusterrole-namespace-readers.yaml
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ ls 
cluster  namespaces  README.md  system
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ ls namespaces/
dev  prod  rolebinding-sre.yaml  selector-sre-support.yaml
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ ls 
system/
README.md  repo.yaml
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ cat system/repo.yaml 
apiVersion: configmanagement.gke.io/v1
kind: Repo
metadata:
  name: repo
spec:
  version: 1.0.0
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
```

initialize the config directory as a new local Git repo:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ git init
git add .
git commit -m "Initial config repo commit"
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Initialized empty Git repository in /home/student_00_ac340a727c48/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config/.git/
[master (root-commit) 2b3bb35] Initial config repo commit
 9 files changed, 67 insertions(+)
 create mode 100644 README.md
 create mode 100644 cluster/clusterrole-namespace-readers.yaml
 create mode 100644 cluster/clusterrolebinding-namespace-readers.yaml
 create mode 100644 namespaces/dev/namespace.yaml
 create mode 100644 namespaces/prod/namespace.yaml
 create mode 100644 namespaces/rolebinding-sre.yaml
 create mode 100644 namespaces/selector-sre-support.yaml
 create mode 100644 system/README.md
 create mode 100644 system/repo.yaml
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
```

### Create a Cloud Source Repositories repo and configure it as a remote for the local repo

1. Create a Cloud Source Repositories repo named `anthos_config` to host your code:

   ```sh
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ gcloud source repos create anthos_config
   Created [anthos_config].
   WARNING: You may be billed for this repository. See https://cloud.google.com/source-repositories/docs/pricing for details.
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   Set `gcloud.sh` to supply credentials for Git access:

   ```sh
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ git config credential.helper gcloud.sh
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   Add your newly created config repository as a Git remote:

   ```sh
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ git remote add origin https://source.developers.google.com/p/$PROJECT_ID/r/anthos_config
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   Push your code to the new repository's master branch:

   ```sh
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ git push origin master
   Enumerating objects: 16, done.
   Counting objects: 100% (16/16), done.
   Delta compression using up to 2 threads
   Compressing objects: 100% (14/14), done.
   Writing objects: 100% (16/16), 1.82 KiB | 372.00 KiB/s, done.
   Total 16 (delta 0), reused 0 (delta 0), pack-reused 0
   remote: Waiting for private key checker: 9/9 objects left
   To https://source.developers.google.com/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config
    * [new branch]      master -> master
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   Verify your repo and source code were created in **Cloud Source Repositories**. Select **Navigation > VIEW ALL PRODUCTS > Source Repositories**. Then select the **anthos_config** repository.

### Generate keys, and create secrets on your clusters

The Anthos Config Management Config Operator, when running on your clusters, needs read-only access to your Git repo, so it can read the latest committed configs, then check and/or apply them to your clusters. The credentials for this read-only access to your Git repo are stored in the `git-creds` secret on each enrolled cluster.

When using Cloud Source Repositories, an SSH keypair is the recommended approach to authorize access to your repo.

Using Cloud Shell, generate an SSH keypair, Save the private key to a secret on each cluster.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ export GCLOUD_EMAIL=$(gcloud config get-value account)
cd $LAB_DIR
ssh-keygen -t rsa -b 4096 \
  -C "$GCLOUD_EMAIL" \
  -N '' \
  -f $HOME/.ssh/id_rsa.acm
Your active configuration is: [cloudshell-8425]
Generating public/private rsa key pair.
Created directory '/home/student_00_ac340a727c48/.ssh'.
Your identification has been saved in /home/student_00_ac340a727c48/.ssh/id_rsa.acm
Your public key has been saved in /home/student_00_ac340a727c48/.ssh/id_rsa.acm.pub
The key fingerprint is:
SHA256:Czoq5A61jI23IXXRmPQayNl5APs4moDFnkvZISK91Ng student-00-ac340a727c48@qwiklabs.net
The key's randomart image is:
+---[RSA 4096]----+
|  ..o            |
| + X B           |
|o % E +          |
|o= O *           |
|o @ = . S        |
|.% = . . .       |
|X B o   .        |
|o+ + .           |
|.oo              |
+----[SHA256]-----+
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx gke
kubectl create secret generic git-creds \
    --namespace=config-management-system \
    --from-file=ssh=$HOME/.ssh/id_rsa.acm
kubectx onprem.k8s.local
kubectl create secret generic git-creds \
    --namespace=config-management-system \
    --from-file=ssh=$HOME/.ssh/id_rsa.acm
Switched to context "gke".
secret/git-creds created
Switched to context "onprem.k8s.local".
secret/git-creds created
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

### Manage keys in Cloud Source Repositories

The SSH public key portion of your generated SSH keypair needs to be  registered with Cloud Source Repositories. The Config Operators on your  clusters can then use the SSH private key, just stored as a cluster  secret, to access your config repository.

In the Cloud Source Repositories console, click the three dots ![More icon](https://cdn.qwiklabs.com/2ufrDePg5inKfodUoT2Kib4oE7II7emYn%2BypCC85FjQ%3D) in the top-right toolbar, then click [Manage SSH Keys](https://source.cloud.google.com/user/ssh_keys).

Click **Register SSH Key**.

You may be prompted to enter your Qwiklabs user `password`.

Enter `config demo key` in the **Key Name** field. You can choose a different key name if needed.

From Cloud Shell, copy the key value from the output of this command:

Return to **Cloud Source Repositories**, and paste the copied key from your public key file into the **Key** field.

Click **Register**.

You will now see your registered key on the Manage SSH Keys page.



## Define and deploy Config Management Operators

### Create your ConfigManagement YAML files

To configure the Config Management Operators to read from your repo, you will create configuration files for the ConfigManagement CustomResources and apply them to your clusters.

You have been provided configuration files for your two clusters. You will need to modify each to point to your hosted repo.

Using the Cloud Shell Code Editor, open the **gke** configuration file for editing:

```sh
edit ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
```

Replace the `[qwiklabs-user-email]` placeholder with the email address for your Qwiklabs user, as shown in the upper left corner of the Qwiklabs window.

Replace the `[qwiklabs-project]` placeholder with GCP Project ID for your project shown in the upper left corner of the Qwiklabs window.

Notice also that a variety of options can be included to configure how the resource interacts with your repo. For example, `secretType` is set to **ssh** indicating ConfigManagement should use the keys stored previously.

Repeat the process for the **onprem** configuration file:

```sh
edit ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yaml
```

```yaml
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ cat ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
# config-management.yaml
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
  namespace: config-management-system
spec:
  clusterName: gke
  git:
    syncRepo: ssh://student-00-ac340a727c48@qwiklabs.net@source.developers.google.com:2022/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config
    syncBranch: master
    secretType: ssh
    policyDir: "."
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 076e~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yamlmanagement.yaml
# config-management.yaml
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
  namespace: config-management-system
spec:
  clusterName: onprem.k8s.local
  git:
    syncRepo: ssh://student-00-ac340a727c48@qwiklabs.net@source.developers.google.com:2022/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config
    syncBranch: master
    secretType: ssh
    policyDir: "."
    syncWait: 2student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

### Check the current state of your clusters

1. Back in Cloud Shell, switch contexts to your **gke** cluster and list Namespaces:

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx gkee)$ kubectx gke
kubectl get namespace
Switched to context "gke".
NAME                           STATUS   AGE
asm-system                     Active   12h
config-management-monitoring   Active   29m
config-management-system       Active   29m
default                        Active   12h
gke-managed-system             Active   12h
gmp-public                     Active   12h
gmp-system                     Active   12h
istio-system                   Active   12h
kube-node-lease                Active   12h
kube-public                    Active   12h
kube-system                    Active   12h
prod                           Active   12h
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl describe namespace prod
Name:         prod
Labels:       istio.io/rev=asm-1157-23
              kubernetes.io/metadata.name=prod
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterroles
NAME                                                                   CREATED AT
admin                                                                  2024-07-24T05:06:50Z
ca-cr-actor                                                            2024-07-24T05:07:25Z
ca-pr-beta-actor                                                       2024-07-24T05:07:26Z
canonical-service-manager-role                                         2024-07-24T05:19:24Z
canonical-service-metrics-reader                                       2024-07-24T05:19:24Z
cloud-provider                                                         2024-07-24T05:08:15Z
cluster-admin                                                          2024-07-24T05:06:49Z
cluster-autoscaler                                                     2024-07-24T05:07:24Z
config-management-operator                                             2024-07-24T17:05:31Z
edit                                                                   2024-07-24T05:06:50Z
external-metrics-reader                                                2024-07-24T05:08:11Z
fluentbit-gke-pod-label-reader                                         2024-07-24T05:07:29Z
gce:beta:kubelet-certificate-bootstrap                                 2024-07-24T05:08:17Z
gce:beta:kubelet-certificate-rotation                                  2024-07-24T05:08:18Z
gce:cloud-provider                                                     2024-07-24T05:08:15Z
gce:gke-metadata-server-reader                                         2024-07-24T05:07:33Z
gke-connect-controllers-impersonation                                  2024-07-24T05:16:27Z
gke-metrics-agent                                                      2024-07-24T05:07:36Z
gmp-system:collector                                                   2024-07-24T05:07:51Z
gmp-system:operator                                                    2024-07-24T05:07:51Z
istio-reader-clusterrole-asm-1157-23-istio-system                      2024-07-24T05:19:06Z
istio-reader-istio-system                                              2024-07-24T05:19:04Z
istiod-clusterrole-asm-1157-23-istio-system                            2024-07-24T05:19:06Z
istiod-gateway-controller-asm-1157-23-istio-system                     2024-07-24T05:19:06Z
istiod-istio-system                                                    2024-07-24T05:19:04Z
konnectivity-agent-cpha                                                2024-07-24T05:07:42Z
kubelet-api-admin                                                      2024-07-24T05:08:17Z
maintenance-handler                                                    2024-07-24T05:07:40Z
membership-reader                                                      2024-07-24T05:16:26Z
metering                                                               2024-07-24T05:16:27Z
netd                                                                   2024-07-24T05:08:01Z
pdcsi-attacher-role                                                    2024-07-24T05:08:02Z
pdcsi-provisioner-role                                                 2024-07-24T05:08:02Z
pdcsi-resizer-role                                                     2024-07-24T05:08:03Z
pdcsi-snapshotter-role                                                 2024-07-24T05:08:03Z
read-updateinfo                                                        2024-07-24T05:08:10Z
snapshot-controller-runner                                             2024-07-24T05:08:07Z
system:aggregate-to-admin                                              2024-07-24T05:06:50Z
system:aggregate-to-edit                                               2024-07-24T05:06:50Z
system:aggregate-to-view                                               2024-07-24T05:06:50Z
system:auth-delegator                                                  2024-07-24T05:06:50Z
system:basic-user                                                      2024-07-24T05:06:49Z
system:certificates.k8s.io:certificatesigningrequests:nodeclient       2024-07-24T05:06:50Z
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   2024-07-24T05:06:50Z
system:certificates.k8s.io:kube-apiserver-client-approver              2024-07-24T05:06:50Z
system:certificates.k8s.io:kube-apiserver-client-kubelet-approver      2024-07-24T05:06:50Z
system:certificates.k8s.io:kubelet-serving-approver                    2024-07-24T05:06:50Z
system:certificates.k8s.io:legacy-unknown-approver                     2024-07-24T05:06:50Z
system:cloud-controller-manager                                        2024-07-24T05:07:22Z
system:clustermetrics                                                  2024-07-24T05:07:26Z
system:controller:attachdetach-controller                              2024-07-24T05:06:50Z
system:controller:certificate-controller                               2024-07-24T05:06:51Z
system:controller:cloud-node-controller                                2024-07-24T05:07:23Z
system:controller:clusterrole-aggregation-controller                   2024-07-24T05:06:50Z
system:controller:cronjob-controller                                   2024-07-24T05:06:50Z
system:controller:daemon-set-controller                                2024-07-24T05:06:50Z
system:controller:deployment-controller                                2024-07-24T05:06:50Z
system:controller:disruption-controller                                2024-07-24T05:06:50Z
system:controller:endpoint-controller                                  2024-07-24T05:06:50Z
system:controller:endpointslice-controller                             2024-07-24T05:06:50Z
system:controller:endpointslicemirroring-controller                    2024-07-24T05:06:50Z
system:controller:ephemeral-volume-controller                          2024-07-24T05:06:51Z
system:controller:expand-controller                                    2024-07-24T05:06:50Z
system:controller:generic-garbage-collector                            2024-07-24T05:06:51Z
system:controller:glbc                                                 2024-07-24T05:07:46Z
system:controller:horizontal-pod-autoscaler                            2024-07-24T05:06:51Z
system:controller:job-controller                                       2024-07-24T05:06:51Z
system:controller:legacy-service-account-token-cleaner                 2024-07-24T05:06:51Z
system:controller:namespace-controller                                 2024-07-24T05:06:51Z
system:controller:node-controller                                      2024-07-24T05:06:51Z
system:controller:persistent-volume-binder                             2024-07-24T05:06:51Z
system:controller:pod-garbage-collector                                2024-07-24T05:06:51Z
system:controller:pv-protection-controller                             2024-07-24T05:06:51Z
system:controller:pvc-protection-controller                            2024-07-24T05:06:51Z
system:controller:replicaset-controller                                2024-07-24T05:06:51Z
system:controller:replication-controller                               2024-07-24T05:06:51Z
system:controller:resourcequota-controller                             2024-07-24T05:06:51Z
system:controller:root-ca-cert-publisher                               2024-07-24T05:06:51Z
system:controller:route-controller                                     2024-07-24T05:06:51Z
system:controller:service-account-controller                           2024-07-24T05:06:51Z
system:controller:service-controller                                   2024-07-24T05:06:51Z
system:controller:statefulset-controller                               2024-07-24T05:06:51Z
system:controller:ttl-after-finished-controller                        2024-07-24T05:06:51Z
system:controller:ttl-controller                                       2024-07-24T05:06:51Z
system:discovery                                                       2024-07-24T05:06:49Z
system:gcp-controller-manager                                          2024-07-24T05:07:30Z
system:gke-common-webhooks                                             2024-07-24T05:07:31Z
system:gke-controller                                                  2024-07-24T05:08:10Z
system:gke-hpa-actor                                                   2024-07-24T05:08:11Z
system:gke-hpa-service-reader                                          2024-07-24T05:08:12Z
system:gke-master-resourcequota                                        2024-07-24T05:08:06Z
system:gke-uas-collection-reader                                       2024-07-24T05:08:12Z
system:gke-uas-metrics-reader                                          2024-07-24T05:08:12Z
system:glbc-status                                                     2024-07-24T05:07:46Z
system:heapster                                                        2024-07-24T05:06:50Z
system:kube-aggregator                                                 2024-07-24T05:06:50Z
system:kube-controller-manager                                         2024-07-24T05:06:50Z
system:kube-dns                                                        2024-07-24T05:06:50Z
system:kube-dns-autoscaler                                             2024-07-24T05:07:19Z
system:kube-scheduler                                                  2024-07-24T05:06:50Z
system:kubelet-api-admin                                               2024-07-24T05:06:50Z
system:kubestore-collector                                             2024-07-24T05:07:45Z
system:maintenance-controller-cluster-role                             2024-07-24T05:07:47Z
system:managed-certificate-controller                                  2024-07-24T05:07:56Z
system:master-monitoring-role                                          2024-07-24T05:07:57Z
system:metrics-server                                                  2024-07-24T05:07:59Z
system:monitoring                                                      2024-07-24T05:06:49Z
system:node                                                            2024-07-24T05:06:50Z
system:node-bootstrapper                                               2024-07-24T05:06:50Z
system:node-problem-detector                                           2024-07-24T05:06:50Z
system:node-proxier                                                    2024-07-24T05:06:50Z
system:persistent-volume-provisioner                                   2024-07-24T05:06:50Z
system:public-info-viewer                                              2024-07-24T05:06:50Z
system:resource-tracker                                                2024-07-24T05:08:06Z
system:service-account-issuer-discovery                                2024-07-24T05:06:50Z
system:slo-monitor                                                     2024-07-24T05:08:07Z
system:volume-scheduler                                                2024-07-24T05:06:50Z
view                                                                   2024-07-24T05:06:50Z
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterrolebindings
NAME                                                     ROLE                                                                 AGE
ca-cr                                                    ClusterRole/ca-cr-actor                                              12h
ca-pr-beta                                               ClusterRole/ca-pr-beta-actor                                         12h
canonical-service-manager-rolebinding                    ClusterRole/canonical-service-manager-role                           12h
canonical-service-proxy-rolebinding                      ClusterRole/canonical-service-proxy-role                             12h
cluster-admin                                            ClusterRole/cluster-admin                                            12h
cluster-autoscaler                                       ClusterRole/cluster-autoscaler                                       12h
cluster-autoscaler-updateinfo                            ClusterRole/read-updateinfo                                          12h
config-management-operator                               ClusterRole/config-management-operator                               31m
event-exporter-rb                                        ClusterRole/view                                                     12h
feature-authorizer                                       ClusterRole/cluster-admin                                            12h
fluentbit-gke-pod-label-reader                           ClusterRole/fluentbit-gke-pod-label-reader                           12h
gce:beta:kubelet-certificate-bootstrap                   ClusterRole/gce:beta:kubelet-certificate-bootstrap                   12h
gce:beta:kubelet-certificate-rotation                    ClusterRole/gce:beta:kubelet-certificate-rotation                    12h
gce:cloud-provider                                       ClusterRole/gce:cloud-provider                                       12h
gce:gke-metadata-server-reader                           ClusterRole/gce:gke-metadata-server-reader                           12h
gke-connect-controllers-impersonation                    ClusterRole/gke-connect-controllers-impersonation                    12h
gke-metrics-agent                                        ClusterRole/gke-metrics-agent                                        12h
gmp-system:collector                                     ClusterRole/gmp-system:collector                                     12h
gmp-system:operator                                      ClusterRole/gmp-system:operator                                      12h
istio-reader-clusterrole-asm-1157-23-istio-system        ClusterRole/istio-reader-clusterrole-asm-1157-23-istio-system        12h
istio-reader-istio-system                                ClusterRole/istio-reader-istio-system                                12h
istiod-clusterrole-asm-1157-23-istio-system              ClusterRole/istiod-clusterrole-asm-1157-23-istio-system              12h
istiod-gateway-controller-asm-1157-23-istio-system       ClusterRole/istiod-gateway-controller-asm-1157-23-istio-system       12h
istiod-istio-system                                      ClusterRole/istiod-istio-system                                      12h
konnectivity-agent-cpha                                  ClusterRole/konnectivity-agent-cpha                                  12h
kube-apiserver-kubelet-api-admin                         ClusterRole/kubelet-api-admin                                        12h
kubelet-bootstrap                                        ClusterRole/system:node-bootstrapper                                 12h
kubelet-bootstrap-certificate-bootstrap                  ClusterRole/gce:beta:kubelet-certificate-bootstrap                   12h
kubelet-bootstrap-node-bootstrapper                      ClusterRole/system:node-bootstrapper                                 12h
kubelet-cluster-admin                                    ClusterRole/system:node                                              12h
kubelet-user-npd-binding                                 ClusterRole/system:node-problem-detector                             12h
maintenance-controller                                   ClusterRole/system:maintenance-controller-cluster-role               12h
maintenance-handler                                      ClusterRole/maintenance-handler                                      12h
master-monitoring-role-binding                           ClusterRole/system:master-monitoring-role                            12h
membership-reader                                        ClusterRole/membership-reader                                        12h
metering                                                 ClusterRole/metering                                                 12h
metrics-server:system:auth-delegator                     ClusterRole/system:auth-delegator                                    12h
netd                                                     ClusterRole/netd                                                     12h
npd-binding                                              ClusterRole/system:node-problem-detector                             12h
pdcsi-controller-attacher-binding                        ClusterRole/pdcsi-attacher-role                                      12h
pdcsi-controller-provisioner-binding                     ClusterRole/pdcsi-provisioner-role                                   12h
pdcsi-controller-resizer-binding                         ClusterRole/pdcsi-resizer-role                                       12h
pdcsi-snapshotter-binding                                ClusterRole/pdcsi-snapshotter-role                                   12h
qwiklabs-gcp-01-076ef386825e-cluster-admin-binding       ClusterRole/cluster-admin                                            12h
snapshot-controller-role                                 ClusterRole/snapshot-controller-runner                               12h
system:basic-user                                        ClusterRole/system:basic-user                                        12h
system:cloud-controller-manager                          ClusterRole/system:cloud-controller-manager                          12h
system:clustermetrics                                    ClusterRole/system:clustermetrics                                    12h
system:controller:attachdetach-controller                ClusterRole/system:controller:attachdetach-controller                12h
system:controller:certificate-controller                 ClusterRole/system:controller:certificate-controller                 12h
system:controller:clusterrole-aggregation-controller     ClusterRole/system:controller:clusterrole-aggregation-controller     12h
system:controller:cronjob-controller                     ClusterRole/system:controller:cronjob-controller                     12h
system:controller:daemon-set-controller                  ClusterRole/system:controller:daemon-set-controller                  12h
system:controller:deployment-controller                  ClusterRole/system:controller:deployment-controller                  12h
system:controller:disruption-controller                  ClusterRole/system:controller:disruption-controller                  12h
system:controller:endpoint-controller                    ClusterRole/system:controller:endpoint-controller                    12h
system:controller:endpointslice-controller               ClusterRole/system:controller:endpointslice-controller               12h
system:controller:endpointslicemirroring-controller      ClusterRole/system:controller:endpointslicemirroring-controller      12h
system:controller:ephemeral-volume-controller            ClusterRole/system:controller:ephemeral-volume-controller            12h
system:controller:expand-controller                      ClusterRole/system:controller:expand-controller                      12h
system:controller:generic-garbage-collector              ClusterRole/system:controller:generic-garbage-collector              12h
system:controller:horizontal-pod-autoscaler              ClusterRole/system:controller:horizontal-pod-autoscaler              12h
system:controller:job-controller                         ClusterRole/system:controller:job-controller                         12h
system:controller:legacy-service-account-token-cleaner   ClusterRole/system:controller:legacy-service-account-token-cleaner   12h
system:controller:namespace-controller                   ClusterRole/system:controller:namespace-controller                   12h
system:controller:node-controller                        ClusterRole/system:controller:node-controller                        12h
system:controller:persistent-volume-binder               ClusterRole/system:controller:persistent-volume-binder               12h
system:controller:pod-garbage-collector                  ClusterRole/system:controller:pod-garbage-collector                  12h
system:controller:pv-protection-controller               ClusterRole/system:controller:pv-protection-controller               12h
system:controller:pvc-protection-controller              ClusterRole/system:controller:pvc-protection-controller              12h
system:controller:replicaset-controller                  ClusterRole/system:controller:replicaset-controller                  12h
system:controller:replication-controller                 ClusterRole/system:controller:replication-controller                 12h
system:controller:resourcequota-controller               ClusterRole/system:controller:resourcequota-controller               12h
system:controller:root-ca-cert-publisher                 ClusterRole/system:controller:root-ca-cert-publisher                 12h
system:controller:route-controller                       ClusterRole/system:controller:route-controller                       12h
system:controller:service-account-controller             ClusterRole/system:controller:service-account-controller             12h
system:controller:service-controller                     ClusterRole/system:controller:service-controller                     12h
system:controller:statefulset-controller                 ClusterRole/system:controller:statefulset-controller                 12h
system:controller:ttl-after-finished-controller          ClusterRole/system:controller:ttl-after-finished-controller          12h
system:controller:ttl-controller                         ClusterRole/system:controller:ttl-controller                         12h
system:discovery                                         ClusterRole/system:discovery                                         12h
system:gcp-controller-manager                            ClusterRole/system:gcp-controller-manager                            12h
system:gke-common-webhooks                               ClusterRole/system:gke-common-webhooks                               12h
system:gke-controller                                    ClusterRole/system:gke-controller                                    12h
system:gke-hpa-actor                                     ClusterRole/system:gke-hpa-actor                                     12h
system:gke-hpa-service-reader                            ClusterRole/system:gke-hpa-service-reader                            12h
system:gke-master-resourcequota                          ClusterRole/system:gke-master-resourcequota                          12h
system:gke-uas-collection-reader                         ClusterRole/system:gke-uas-collection-reader                         12h
system:gke-uas-hpa-controller                            ClusterRole/system:controller:horizontal-pod-autoscaler              12h
system:gke-uas-metrics-reader                            ClusterRole/system:gke-uas-metrics-reader                            12h
system:glbc-status                                       ClusterRole/system:glbc-status                                       12h
system:konnectivity-server                               ClusterRole/system:auth-delegator                                    12h
system:kube-controller-manager                           ClusterRole/system:kube-controller-manager                           12h
system:kube-dns                                          ClusterRole/system:kube-dns                                          12h
system:kube-dns-autoscaler                               ClusterRole/system:kube-dns-autoscaler                               12h
system:kube-proxy                                        ClusterRole/system:node-proxier                                      12h
system:kube-scheduler                                    ClusterRole/system:kube-scheduler                                    12h
system:kubestore-collector                               ClusterRole/system:kubestore-collector                               12h
system:managed-certificate-controller                    ClusterRole/system:managed-certificate-controller                    12h
system:metrics-server                                    ClusterRole/system:metrics-server                                    12h
system:monitoring                                        ClusterRole/system:monitoring                                        12h
system:node                                              ClusterRole/system:node                                              12h
system:node-proxier                                      ClusterRole/system:node-proxier                                      12h
system:public-info-viewer                                ClusterRole/system:public-info-viewer                                12h
system:resource-tracker                                  ClusterRole/system:resource-tracker                                  12h
system:service-account-issuer-discovery                  ClusterRole/system:service-account-issuer-discovery                  12h
system:slo-monitor                                       ClusterRole/system:slo-monitor                                       12h
system:volume-scheduler                                  ClusterRole/system:volume-scheduler                                  12h
uas-hpa-external-metrics-reader                          ClusterRole/external-metrics-reader                                  12h
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get rolebindings -n prod
NAME                   ROLE                        AGE
istio-ingressgateway   Role/istio-ingressgateway   12h
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

At this point, both your clusters have a **prod** Namespace, but no **dev** Namespace. There are no **namespace-readers** ClusterRoles or bindings, nor are there any RoleBindings in the **prod** Namespace for the **sre** group. This will all change when config management is enabled.  



### Review the configurations stored in your repo

1. In the Cloud Shell editor, navigate to **acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config**. Note the folder structure:
   - The **cluster** folder has configurations that apply to clusters being managed
   - The **namespaces** folder has configurations that apply to namespaces on clusters being managed.
2. In the **cluster** folder, open and review the  configuration files you find. One defines a ClusterRole you wish to add  to each cluster, and the second defines a ClusterRoleBinding you wish to add to each cluster.
3. In the **namespaces** folder, open the **dev** folder and then the **namespace.yaml** file inside. This file defines a Namespace you wish to have created on every cluster.
4. In the **namespaces** folder, open the **prod** folder and then the **namespace.yaml** file inside. This file defines a Namespace you wish to have created on every cluster. Note the **env** label.
5. In the **namespaces** folder, open the **selector-sre-support.yaml** file. Note that the NamespaceSelector will select only Namespaces that have a given label. In this case, the label is **env:prod** - so only the **prod** Namespace will be affected by configurations that use this selector.
6. In the **namespaces** folder, open the **rolebinding-sre.yaml** file. Note the annotations which indicate that this config should be applied using a selector.

​     When these configurations are applied, you should end up with the     following in place: 



- ​         A ClusterRole named **namespace-readers**
- ​         A ClusterRoleBinding for **Cheryl**
- ​         A **dev** Namespace     
- ​         A **prod** Namespace with **env** and         **istio-injection** labels     
- ​         A RoleBinding in the **prod** Namespace for         **sre@foo-corp.com**

### Deploy the Config Management Operator

1. In Cloud Shell, apply the configuration to the **gke** cluster.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx gke
kubectl apply -f ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
Switched to context "gke".
configmanagement.configmanagement.gke.io/config-management created
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

2. Apply the configuration to the **onprem** cluster.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx onprem.k8s.local
kubectl apply -f ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yaml
Switched to context "onprem.k8s.local".
configmanagement.configmanagement.gke.io/config-management created
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

Wait 30 seconds, then use `nomos status` to see if Anthos  Config Management is properly installed and configured. If the clusters  aren't both synched, wait another 30 seconds and try again. They should  be synched at this point.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ nomos status
Connecting to clusters...
Notice: The following clusters are still running in the legacy mode:
- onprem.k8s.local
- gke
Run `nomos migrate` to enable multi-repo mode. It provides you with additional features and gives you the flexibility to sync to a single repository, or multiple repositories.

gke
  --------------------
  <root>:                                  ssh://student-00-ac340a727c48@qwiklabs.net@source.developers.google.com:2022/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config@master   
  SYNCED @ 0001-01-01 00:00:00 +0000 UTC   2b3bb353                                                                                                                             

*onprem.k8s.local
  --------------------
  <root>:                                  ssh://student-00-ac340a727c48@qwiklabs.net@source.developers.google.com:2022/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config@master   
  SYNCED @ 0001-01-01 00:00:00 +0000 UTC   2b3bb353                                                                                                                             
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

## Verify that the configurations have been applied to your clusters

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectx gke
kubectl get namespaces
Switched to context "gke".
NAME                           STATUS   AGE
asm-system                     Active   12h
config-management-monitoring   Active   36m
config-management-system       Active   36m
default                        Active   12h
dev                            Active   108s
gke-managed-system             Active   12h
gmp-public                     Active   12h
gmp-system                     Active   12h
istio-system                   Active   12h
kube-node-lease                Active   12h
kube-public                    Active   12h
kube-system                    Active   12h
prod                           Active   12h
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterroles | grep readers
namespace-readers                                                      2024-07-24T17:40:33Z
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterrolebindings | grep readers
namespace-readers                                        ClusterRole/namespace-reader                                         2m46s
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl describe clusterrolebinding namespace-readers
Name:         namespace-readers
Labels:       app.kubernetes.io/managed-by=configmanagement.gke.io
              configsync.gke.io/declared-version=v1
Annotations:  configmanagement.gke.io/cluster-name: gke
              configmanagement.gke.io/managed: enabled
              configmanagement.gke.io/source-path: cluster/clusterrolebinding-namespace-readers.yaml
              configmanagement.gke.io/token: 2b3bb353f269d2e354c70c31e66fae5b15722c6e
              configsync.gke.io/declared-fields: {"f:metadata":{"f:annotations":{},"f:labels":{}},"f:roleRef":{},"f:subjects":{}}
              configsync.gke.io/resource-id: rbac.authorization.k8s.io_clusterrolebinding_namespace-readers
Role:
  Kind:  ClusterRole
  Name:  namespace-reader
Subjects:
  Kind  Name                    Namespace
  ----  ----                    ---------
  User  cheryl@anthos-labs.com  
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get rolebindings -n dev
No resources found in dev namespace.
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get rolebindings -n prod
NAME                   ROLE                        AGE
istio-ingressgateway   Role/istio-ingressgateway   12h
sre-admin              ClusterRole/admin           4m3s
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
```

- Note that the Namespace selector limited application of this configuration to only the **prod** Namespace.

Your configurations, stored in your Cloud Source Repository, have been applied to the **gke** cluster. Now, check to see if they have been applied to the **onprem** cluster.

Repeat the steps that you performed against the **gke** cluster. Verify that the changes have applied to the **onprem** cluster as well.

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterroles | grep readers
namespace-readers                                                      2024-07-24T17:41:10Z
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ kubectl get clusterrolebindings | grep readers
namespace-readers                                      ClusterRole/namespace-reader                                       4m46s
student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$
```

## Review automated drift management

In this task, you verify that Anthos Config Management keeps objects  in sync with the configs in your repo, even if someone makes manual  changes.

### Set up tmux panes in Cloud Shell

You are going to configure three Cloud Shell panes so that you can  issue commands in one pane and watch the effects on the two clusters in  the other panes.

In this lab, you configured Anthos Config Management and explored somSplit the session screen with the `tmux` utility built-into Cloud Shell by typing `<Ctrl>+b`, then `%`. You should see 2 panes in the Cloud Shell.

Any time you interact with `tmux`, you'll start with the `<Ctrl>+b` combination, which signals a command to `tmux`.

Switch to the left-hand pane by typing:

- `<Ctrl>+b`
- `<left-arrow>`

Resize the left-hand pane by doing the following:

- Type `<Ctrl>+b` to begin interaction with tmux
- Type `:` to get a tmux command prompt
- Type `resize-pane -L 35` to make the left-hand pane narrower

Switch to the right-hand pane by typing:

- `<Ctrl>+b`
- `<right-arrow>`

In the right-hand pane, split the pane by typing:

- `<Ctrl>+b`
- `%`

You should now have 3 panes that are roughly the same width.



### Try deleting an object managed by Anthos Config Management

1. Switch the the left-hand pane (`<Ctrl>+b`, `<right-arrow>`), set the `kubectl` context, and have `kubectl` watch for changes to the ClusterRoleBinding for **namespace-readers** on the **gke** cluster:
2. Switch the the middle pane (`<Ctrl>+b`, `<right-arrow>`), set the `kubectl` context and have `kubectl` watch for changes to the ClusterRoleBinding for **namespace-readers** on the **onprem** cluster:
3. Switch the the right-hand pane (`<Ctrl>+b`, `<right-arrow>`), and delete the ClusterRoleBinding on both clusters:
4. You should see two updates display in each of the panes where you are watching for object changes. One indicating the deletion of the object, and one showing the creation of the object to have the cluster comply with the defined config.
5. In the right-hand pane, confirm that the ClusterRoleBinding has been recreated on the **gke** cluster:
6. Repeat the process for the **onprem** cluster.

### Try updating an object managed by Anthos Config Management

1. Switch to the left-hand pane  (`<Ctrl>+b`, `<right-arrow>`), and cancel the `kubectl` watch command by typing `<Ctrl>+c`.

2. Start a new watch command, this time observing changes to the **prod** Namespace labels on the **gke** cluster, and display the Namespace config in detail:

3. Switch to the middle pane (`<Ctrl>+b`, `<right-arrow>`), and cancel the `kubectl` watch command by typing `<Ctrl>+c`.

   Start a new watch command, this time observing changes to the **prod** Namespace labels on the **onprem** cluster, and display the Namespace config in detail:

   Switch to the right-hand pane (`<Ctrl>+b`, `<right-arrow>`), and remove the **env: prod** label from the prod Namespaces on both clusters:

   You should see two messages in each of the panes where you are watching for object changes. The first shows the labels on the namespace after the **env:prod** label is removed. The second shows the labels after it's been re-added.

## Update a config in the repo

In this task, you verify that Anthos Config Management **updates** managed objects when the configs in your repo change.

Switch to the left-hand pane  (`<Ctrl>+b`, `<rigt-arrow>`), and cancel the `kubectl` watch command by typing `<Ctrl>+c`.

In the left pane, review the `namespace-readers` ClusterRoleBinding on the **gke** cluster:

Configure `kubectl` to watch for changes to the subjects in this ClusterRoleBinding on the **gke** cluster:

Switch to the middle pane (`<Ctrl>+b`, `<right-arrow>`), and cancel the `kubectl` watch command by typing `<Ctrl>+c`.

In the middle pane, review the `namespace-readers` ClusterRoleBinding on the **oprem** cluster:

Configure `kubectl` to watch for changes to the subjects in this ClusterRoleBinding on the **onprem** cluster:

Switch to the right-hand pane (`<Ctrl>+b`, `<right-arrow>`), and clear the pane:

### Update your cluster configuration repo

1. Using the Cloud Shell Code Editor, edit the configuration file for the ClusterRoleBinding:

2. Add a new **User** block to the `subjects` field for `jane@anthos_labs.com`. You can copy the entire `cheryl@anthos_labs.com` **User**, to a new **User**, and replace the **name** with `jane@anthos_labs.com`

   ```sh
   subjects:
   - kind: User
     name: cheryl@anthos_labs.com
     apiGroup: rbac.authorization.k8s.io
   - kind: User
     name: jane@anthos_labs.com
     apiGroup: rbac.authorization.k8s.io
   ```

   1. Save your changes.

   ### Push the change to your config repo

   1. In the **right pane**, check your config changes are syntactically valid:

   ```sh
   student_00_ac340a727c48@cloudshell:~ (qwiklabs-gcp-01-076ef386825e)$ export LAB_DIR=$HOME/acm-lab
   cd $LAB_DIR
   
   ./nomos vet --path=training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
   ✅ No validation issues found.
   student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   No errors are printed, so the configuration is valid.

   In the **right pane**, create a commit, and push the change to your repo:

   ```sh
   student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$ cd ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
   
   git add .
   git commit -m "Add Jane to namespace-reader."
   git push origin master
   [master 51dc433] Add Jane to namespace-reader.
    1 file changed, 3 insertions(+)
   Enumerating objects: 7, done.
   Counting objects: 100% (7/7), done.
   Delta compression using up to 2 threads
   Compressing objects: 100% (4/4), done.
   Writing objects: 100% (4/4), 468 bytes | 234.00 KiB/s, done.
   Total 4 (delta 2), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (2/2)
   remote: Waiting for private key checker: 1/1 objects left
   To https://source.developers.google.com/p/qwiklabs-gcp-01-076ef386825e/r/anthos_config
      2b3bb35..51dc433  master -> master
   student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
   ```

   Within a few seconds of the push being completed, you should see a message in each of the panes where you are watching for object changes. They should show that there are now entries for both **Cheryl** and **Jane**.

### Review the current configuration and set up watches

In this lab, you configured Anthos Config Management and explored some of its useful features. You connected a Git repository for configuration-as-code change-management.  You set up a Config Operator to manage your clusters, and you verified that the operator maintains state in your clusters to match your repository.

## History

```sh
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ history 
    1  export PROJECT_ID=$(gcloud config get-value project)
    2  export SHELL_IP=$(curl -s api.ipify.org)
    3  export KUBECONFIG=~/.kube/config
    4  export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    5      --format="value(projectNumber)")
    6  gcloud compute firewall-rules create shell-to-onprem     --network=onprem-k8s-local     --allow tcp     --source-ranges $SHELL_IP
    7  gsutil cp gs://$PROJECT_ID-kops-onprem/config     ~/.kube/config
    8  kubectx onprem.k8s.local
    9  kubectl create token remote-admin-sa
   10  ZONE=us-west1-a
   11  gcloud container clusters get-credentials gke --zone $ZONE --project $PROJECT_ID
   12  kubectx gke=gke_${PROJECT_ID}_${ZONE}_gke
   13  kubectx gke
   14  export LAB_DIR=$HOME/acm-lab
   15  mkdir $LAB_DIR
   16  cd $LAB_DIR
   17  gsutil cp gs://config-management-release/released/latest/config-management-operator.yaml config-management-operator.yaml
   18  cat config-management-operator.yaml 
   19  kubectl apply -f config-management-operator.yaml
   20  kubectx onprem.k8s.local
   21  kubectl apply -f config-management-operator.yaml
   22  cd $LAB_DIR
   23  gsutil cp gs://config-management-release/released/latest/linux_amd64/nomos nomos
   24  chmod +x ./nomos
   25  ./nomos status
   26  export GCLOUD_EMAIL=$(gcloud config get-value account)
   27  echo $GCLOUD_EMAIL
   28  echo $USER
   29  git config --global user.email "$GCLOUD_EMAIL"
   30  git config --global user.name "$USER"
   31  gcloud config get value account
   32  gcloud config get-value account
   33  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
   34  cd ./training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
   35  ls
   36  cat cluster/
   37  ls cluster/
   38  ls 
   39  ls namespaces/
   40  ls system/
   41  cat system/repo.yaml 
   42  git init
   43  git add .
   44  git commit -m "Initial config repo commit"
   45  gcloud source repos create anthos_config
   46  git config credential.helper gcloud.sh
   47  git remote add origin https://source.developers.google.com/p/$PROJECT_ID/r/anthos_config
   48  git push origin master
   49  export GCLOUD_EMAIL=$(gcloud config get-value account)
   50  cd $LAB_DIR
   51  ssh-keygen -t rsa -b 4096   -C "$GCLOUD_EMAIL"   -N ''   -f $HOME/.ssh/id_rsa.acm
   52  kubectx gke
   53  kubectl create secret generic git-creds     --namespace=config-management-system     --from-file=ssh=$HOME/.ssh/id_rsa.acm
   54  kubectx onprem.k8s.local
   55  kubectl create secret generic git-creds     --namespace=config-management-system     --from-file=ssh=$HOME/.ssh/id_rsa.acm
   56  cat $HOME/.ssh/id_rsa.acm.pub
   57  edit ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
   58  edit ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yaml
   59  cat ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
   60  cat ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yaml
   61  kubectx gke
   62  kubectl get namespace
   63  kubectl describe namespace prod
   64  kubectl get clusterroles
   65  kubectl get clusterrolebindings
   66  kubectl get rolebindings -n prod
   67  kubectx gke
   68  kubectl apply -f ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/gke-config-management.yaml
   69  kubectx onprem.k8s.local
   70  kubectl apply -f ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config-management/onprem-config-management.yaml
   71  nomos status
   72  kubectx gke
   73  kubectl get namespaces
   74  kubectl get clusterroles | match readers
   75  kubectl get clusterroles | grep readers
   76  kubectl get clusterrolebinding | grep foo
   77  kubectl get clusterrolebindings | grep foo
   78  kubectl get clusterrolebindings | grep readers
   79  kubectl describe clusterrolebinding namespace-readers
   80  kubectl get rolebindings -n dev
   81  kubectl get rolebindings -n prod
   82  kubectx onprem.k8s.local
   83  kubectl get clusterroles | grep readers
   84  kubectl get clusterrolebindings | grep readers
   85  %
   86  clear
   87  kubectx gke
   88  kubectl delete clusterrolebinding namespace-readers
   89  kubectx onprem.k8s.local
   90  kubectl delete clusterrolebinding namespace-readers
   91  kubectx gke
   92  kubectl describe ClusterRoleBinding namespace-readers
   93  clear
   94  kubectx gke
   95  kubectl get namespace prod -o custom-columns=NAME:.metadata.labels     --watch-only
   96  clear
   97  kubectx gke
   98  kubectl get clusterrolebinding namespace-readers --watch-only
   99  edit ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config/cluster/clusterrolebinding-namespace-readers.yaml
  100  export LAB_DIR=$HOME/acm-lab
  101  cd $LAB_DIR
  102  ./nomos vet --path=training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
  103  v1.0/AHYBRID071/config
  104  student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$student_00_ac3│                                            │✅ No validation issues found.
  105  student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-student_00_ac340a727c48@clou│                                            │student_00_ac340a727c48@cloudshell:~/acm-lab
  106  student_00_ac340a727c48@cloudshell:~/acm-lab (qwiklabs-gcp-01-076ef386825e)$              │                                            │ (qwiklabs-gcp-01-076ef386825e)$
  107  export LAB_DIR=$HOME/acm-lab
  108  cd $LAB_DIR
  109  ./nomos vet --path=training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
  110  cd ~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config
  111  git add .
  112  git commit -m "Add Jane to namespace-reader."
  113  git push origin master
  114  history 
student_00_ac340a727c48@cloudshell:~/acm-lab/training-data-analyst/courses/ahybrid/v1.0/AHYBRID071/config (qwiklabs-gcp-01-076ef386825e)$ 
```

