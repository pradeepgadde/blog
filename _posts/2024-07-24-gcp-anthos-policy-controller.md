---

layout: single
title:  "Enforcing Policy with Anthos Config Management Policy Controller"
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
# Enforcing Policy with Anthos Config Management Policy Controller

Policy Controller enforces your clusters' compliance with policies called constraints. In this lab, you use the [constraint library](https://cloud.google.com/anthos-config-management/docs/reference/constraint-template-library), a set of templates that can be easily configured to enforce and audit security and compliance policies. For example, you can require that namespaces have at least one label so that you use GKE Usage Metering and allocate costs to different teams. Or, you might want to enforce the same requirements as provided by PodSecurityPolicies, but with the ability to audit them before disrupting a deployment with enforcements.

In addition to working with the provided platform constraint templates, you will learn how to create your own. With constraint templates, a centralized team can define how a constraint works, and delegate defining the specifics of the constraint to an individual or group with subject-matter expertise.

Policy Controller is built from the [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) open source project and is integrated into Anthos Config Management v1.1+.

- Install Anthos Policy Controller
- Create and enforce constraints using the Template Library provided by Google
- Audit constraint violation
- Create your own Template Constraints to create the custom compliance policies that your company needs

## Complete and verify the lab setup

A GKE cluster named **gke** has been created and registered.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-fe43bbf70613.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ ZONE=us-west1-b
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ export CLUSTER_NAME=gke
export PROJECT_ID=$(gcloud config get-value project)
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")
gcloud container clusters get-credentials gke --zone $ZONE --project $PROJECT_ID
Your active configuration is: [cloudshell-5876]
Fetching cluster endpoint and auth data.
kubeconfig entry generated for gke.
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ gcloud beta container fleet config-management enable 
Enabling service [anthospolicycontroller.googleapis.com] on project [qwiklabs-gcp-01-fe43bbf70613]...
Operation "operations/acat.p2-146607874142-a5455c52-8e3c-4cd4-9f02-16de0a67c719" finished successfully.
Enabling service [anthosconfigmanagement.googleapis.com] on project [qwiklabs-gcp-01-fe43bbf70613]...
Operation "operations/acat.p2-146607874142-066718a7-4aa3-4281-b7e0-5ba2fef507d1" finished successfully.
Waiting for Feature Config Management to be created...done.                                                                                                                        
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

## Install the Anthos Policy Controller

1. Create the Anthos Policy Controller config file:
2. Install the Anthos Policy Controller:
3. Poll the Config Management service to see if the Anthos Policy Controller has been installed:

When the output updates to match what is shown below, you can type `<CTRL>+C` to exit the polling and continue. It may take 4-5 minutes (but not longer) for the controller to be installed. Sometimes the Status and/or the Hierachy_Controller stays in PENDING. Ignore that and continue with the next steps.

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat <<EOF > config-management.yaml
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
applySpecVersion: 1
spec:
  # Set to true to install and enable Policy Controller
  policyController:
    enabled: true
    # Uncomment to prevent the template library from being installed
    # templateLibraryInstalled: false
    # Uncomment to enable support for referential constraints
    # referentialRulesEnabled: true
EOF ...other fields...aces: ["namespace-name"]ailures audit interval
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ ls
config-management.yaml  README-cloudshell.txt
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat config-management.yaml 
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
applySpecVersion: 1
spec:
  # Set to true to install and enable Policy Controller
  policyController:
    enabled: true
    # Uncomment to prevent the template library from being installed
    # templateLibraryInstalled: false
    # Uncomment to enable support for referential constraints
    # referentialRulesEnabled: true
    # Uncomment to disable audit, adjust value to set audit interval
    # auditIntervalSeconds: 0
    # Uncomment to log all denies and dryrun failures
    # logDeniesEnabled: true
    # Uncomment to exempt namespaces
    # exemptableNamespaces: ["namespace-name"]
  # ...other fields...
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$
```



```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ gcloud beta container fleet config-management apply \
    --membership=${CLUSTER_NAME}-connect \
    --config=config-management.yaml \
    --project=$PROJECT_ID
Waiting for Feature Config Management to be updated...done.                                                                                                                        
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ watch gcloud beta container fleet config-management status \
    --project=$PROJECT_ID
student_01_2c1b6dd12803@cloudshell:~
Every 2.0s: gcloud beta container fleet config-management status --project=qwiklabs-gcp-01-fe43bbf70613                            cs-573077193808-default: Wed Jul 24 21:51:43 2024

Name: global/gke-connect
Status: NOT_INSTALLED
Last_Synced_Token: NA
Sync_Branch: NA
Last_Synced_Time: NA
Policy_Controller: INSTALLED
Hierarchy_Controller: NA
Version: 1.18.2
Upgrades: manual

```

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl get constrainttemplates
NAME                                        AGE
allowedserviceportname                      4m55s
asmauthzpolicydisallowedprefix              4m59s
asmauthzpolicyenforcesourceprincipals       4m57s
asmauthzpolicynormalization                 5m
asmauthzpolicysafepattern                   4m54s
asmingressgatewaylabel                      4m51s
asmpeerauthnstrictmtls                      4m58s
asmrequestauthnprohibitedoutputheaders      4m56s
asmsidecarinjection                         4m58s
destinationruletlsenabled                   4m54s
disallowedauthzprefix                       4m55s
gcpstoragelocationconstraintv1              4m56s
k8sallowedrepos                             4m55s
k8savoiduseofsystemmastersgroup             5m
k8sblockallingress                          5m1s
k8sblockcreationwithdefaultserviceaccount   4m58s
k8sblockendpointeditdefaultrole             4m56s
k8sblockloadbalancer                        4m51s
k8sblocknodeport                            4m57s
k8sblockobjectsoftype                       4m53s
k8sblockprocessnamespacesharing             4m50s
k8sblockwildcardingress                     4m59s
k8scontainerephemeralstoragelimit           4m53s
k8scontainerlimits                          5m
k8scontainerratios                          4m53s
k8scontainerrequests                        4m59s
k8scronjoballowedrepos                      4m54s
k8sdisallowanonymous                        5m1s
k8sdisallowedrepos                          4m55s
k8sdisallowedrolebindingsubjects            4m54s
k8sdisallowedtags                           4m53s
k8sdisallowinteractivetty                   4m56s
k8semptydirhassizelimit                     5m
k8senforcecloudarmorbackendconfig           4m51s
k8sexternalips                              4m54s
k8shttpsonly                                4m54s
k8simagedigests                             4m57s
k8slocalstoragerequiresafetoevict           4m53s
k8smemoryrequestequalslimit                 4m58s
k8snoenvvarsecrets                          5m
k8snoexternalservices                       4m59s
k8spodresourcesbestpractices                4m57s
k8spodsrequiresecuritycontext               4m56s
k8sprohibitrolewildcardaccess               5m
k8spspallowedusers                          4m58s
k8spspallowprivilegeescalationcontainer     4m52s
k8spspapparmor                              5m1s
k8spspautomountserviceaccounttokenpod       4m52s
k8spspcapabilities                          4m55s
k8spspflexvolumes                           4m57s
k8spspforbiddensysctls                      4m51s
k8spspfsgroup                               4m53s
k8spsphostfilesystem                        4m57s
k8spsphostnamespace                         4m54s
k8spsphostnetworkingports                   4m52s
k8spspprivilegedcontainer                   4m55s
k8spspprocmount                             4m50s
k8spspreadonlyrootfilesystem                4m51s
k8spspseccomp                               4m50s
k8spspselinuxv2                             4m59s
k8spspvolumetypes                           4m51s
k8spspwindowshostprocess                    5m
k8spssrunasnonroot                          4m56s
k8sreplicalimits                            4m55s
k8srequirecosnodeimage                      4m57s
k8srequiredannotations                      4m58s
k8srequiredlabels                           5m1s
k8srequiredprobes                           4m58s
k8srequiredresources                        4m55s
k8srequirevalidrangesfornetworks            4m55s
k8srestrictadmissioncontroller              4m55s
k8srestrictautomountserviceaccounttokens    4m52s
k8srestrictlabels                           4m57s
k8srestrictnamespaces                       4m52s
k8srestrictnfsurls                          4m54s
k8srestrictrbacsubjects                     4m59s
k8srestrictrolebindings                     4m52s
k8srestrictrolerules                        4m51s
noupdateserviceaccount                      4m54s
policystrictonly                            4m52s
restrictnetworkexclusions                   4m58s
sourcenotallauthz                           4m52s
verifydeprecatedapi                         4m57s
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

## Create a constraint from the templates in the library

Let's try one of the constraints from the library.

1. Let's specify that new namespaces must the have label `geo` before they can be created:

```yaml
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat <<EOF > ns-must-have-geo.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-geo
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels:
      - key: "geo"
EOF
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl apply -f ns-must-have-geo.yaml
k8srequiredlabels.constraints.gatekeeper.sh/ns-must-have-geo created
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Try creating a namespace without the `geo` key, and confirm that it doesn't work:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl create namespace test
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-geo] you must provide labels: {"geo"}
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Try creating the same namespace with the `geo` label:

```yaml
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat <<EOF > test-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    geo: eu-west1
EOF
kubectl apply -f test-namespace.yaml
namespace/test created
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Congratulations! You successfully configured a template from the  Constraints Library, deployed it, and verified that prevents users from  installing non compliant resources.

## Audit the constraint policies

Policy Controller constraint objects enable you to enforce policies for your Kubernetes clusters. To help test your policies, you can add an enforcement action to your constraints. You can then view violations in constraint objects and logs. There are two enforcement actions: **deny** and **dryrun**.

- **deny** is the default enforcement action. It's  automatically enabled, even if you don't add an enforcement action in  your constraint. Use deny to prevent a given cluster operation from  occurring when there's a violation.
- **dryrun** let's you monitor violations of your rules  without actively blocking transactions. You can use it to test if your  constraints are working as intended, prior to enabling active  enforcement using the deny action. Testing constraints this way can  prevent disruptions caused by an incorrectly configured constraint.

Audit the constraint violations using the following `kubectl` command:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl get K8sRequiredLabels ns-must-have-geo -o yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"constraints.gatekeeper.sh/v1beta1","kind":"K8sRequiredLabels","metadata":{"annotations":{},"name":"ns-must-have-geo"},"spec":{"match":{"kinds":[{"apiGroups":[""],"kinds":["Namespace"]}]},"parameters":{"labels":[{"key":"geo"}]}}}
  creationTimestamp: "2024-07-24T21:52:46Z"
  generation: 1
  name: ns-must-have-geo
  resourceVersion: "151147"
  uid: 0e84cf0f-df1e-4f70-af63-da8a6057a780
spec:
  match:
    kinds:
    - apiGroups:
      - ""
      kinds:
      - Namespace
  parameters:
    labels:
    - key: geo
status:
  auditTimestamp: "2024-07-24T21:57:34Z"
  byPod:
  - constraintUID: 0e84cf0f-df1e-4f70-af63-da8a6057a780
    enforced: true
    id: gatekeeper-audit-774c597f76-sfzhk
    observedGeneration: 1
    operations:
    - audit
    - status
  - constraintUID: 0e84cf0f-df1e-4f70-af63-da8a6057a780
    enforced: true
    id: gatekeeper-controller-manager-7db5b4484f-rlfpq
    observedGeneration: 1
    operations:
    - webhook
  totalViolations: 10
  violations:
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gke-managed-system
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: default
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-node-lease
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-public
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-system
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: config-management-system
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: config-management-monitoring
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gmp-public
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gatekeeper-system
    version: v1
  - enforcementAction: deny
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gmp-system
    version: v1
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Notice that the type of `enforcementAction` is **deny** as you have not changed the default behavior. Notice as well that there are a total of 9 existing violations - these are namespaces that don't meet the policy, but were created before the policy was applied.

Open the constraint in the Cloud Shell editor. Add the constraint `enforcementAction` to `dryrun`. That way you can validate policy constraints before enforcing them. Make sure that the yaml file looks as follows:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat ns-must-have-geo.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-geo
spec:
  enforcementAction: dryrun
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels:
      - key: "geo"
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl apply -f ns-must-have-geo.yaml
k8srequiredlabels.constraints.gatekeeper.sh/ns-must-have-geo configured
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$
```

Try creating another namespace without the `geo` label and observe that the command completes this time:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl create namespace ns-with-violations
namespace/ns-with-violations created
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Check the violations again, to see a new violation added of type `dryrun`:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl get K8sRequiredLabels ns-must-have-geo -o yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"constraints.gatekeeper.sh/v1beta1","kind":"K8sRequiredLabels","metadata":{"annotations":{},"name":"ns-must-have-geo"},"spec":{"enforcementAction":"dryrun","match":{"kinds":[{"apiGroups":[""],"kinds":["Namespace"]}]},"parameters":{"labels":[{"key":"geo"}]}}}
  creationTimestamp: "2024-07-24T21:52:46Z"
  generation: 2
  name: ns-must-have-geo
  resourceVersion: "152924"
  uid: 0e84cf0f-df1e-4f70-af63-da8a6057a780
spec:
  enforcementAction: dryrun
  match:
    kinds:
    - apiGroups:
      - ""
      kinds:
      - Namespace
  parameters:
    labels:
    - key: geo
status:
  auditTimestamp: "2024-07-24T22:00:34Z"
  byPod:
  - constraintUID: 0e84cf0f-df1e-4f70-af63-da8a6057a780
    enforced: true
    id: gatekeeper-audit-774c597f76-sfzhk
    observedGeneration: 2
    operations:
    - audit
    - status
  - constraintUID: 0e84cf0f-df1e-4f70-af63-da8a6057a780
    enforced: true
    id: gatekeeper-controller-manager-7db5b4484f-rlfpq
    observedGeneration: 2
    operations:
    - webhook
  totalViolations: 11
  violations:
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gke-managed-system
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: ns-with-violations
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: default
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-node-lease
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-public
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: kube-system
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: config-management-system
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: config-management-monitoring
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gmp-public
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gatekeeper-system
    version: v1
  - enforcementAction: dryrun
    group: ""
    kind: Namespace
    message: 'you must provide labels: {"geo"}'
    name: gmp-system
    version: v1
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

## Writing a constraint template

In this task, you will write a custom constraint template and use it to extend Policy Controller capabilities. This is great when you cannot find a pre-written constraint template that suits your needs for extending Policy Controller.

Policy Controller policies are described by using the [OPA Constraint Framework](https://github.com/open-policy-agent/frameworks/tree/master/constraint) and are written in [Rego](https://www.openpolicyagent.org/docs/latest/language-reference/). A policy can evaluate any field of a Kubernetes object.

Writing policies using Rego is a specialized skill. For this reason, a [library of common constraint templates](https://cloud.google.com/anthos-config-management/docs/reference/constraint-template-library) is installed by default. Most users can leverages these constraint templates when creating constraints. If you have specialized needs, you can create your own constraint templates.

Constraint templates let you separate a policy's logic from its specific requirements, and this enables reuse and delegation. You can create constraints by using constraint templates developed by third parties, such as open source projects, software vendors, or regulatory experts.

Create the policy template that denies all resources whose name matches a value provided by the creator of the constraint:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat <<EOF > k8sdenyname-constraint-template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sdenyname
spec:
  crd:
    spec:
      names:
        kind: K8sDenyName
      validation:
        openAPIV3Schema:
          properties:
            invalidName:
EOF     } msg := sprintf("The name %v is not allowed", [input.parameters.invalidName])
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl apply -f k8sdenyname-constraint-template.yaml
constrainttemplate.templates.gatekeeper.sh/k8sdenyname created
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat k8sdenyname-constraint-template.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sdenyname
spec:
  crd:
    spec:
      names:
        kind: K8sDenyName
      validation:
        openAPIV3Schema:
          properties:
            invalidName:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sdenynames
        violation[{"msg": msg}] {
          input.review.object.metadata.name == input.parameters.invalidName
          msg := sprintf("The name %v is not allowed", [input.parameters.invalidName])
        }
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Once the template is deployed, a team can reference it the same way it  would reference a template from Google's library to create a constraint:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ cat <<EOF > k8sdenyname-constraint.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sDenyName
metadata:
  name: no-policy-violation
spec:
  parameters:
    invalidName: "policy-violation"
EOF
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl apply -f k8sdenyname-constraint.yaml
k8sdenyname.constraints.gatekeeper.sh/no-policy-violation created
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Try creating any resource with the name `policy-violation` to verify that it does not work:

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ kubectl create namespace policy-violation
Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [no-policy-violation] The name policy-violation is not allowed
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

Congratulations! You successfully created and deployed your own  template. Then you created a constraint and verified that it prevents  users from installing non compliant resources.

In this lab, you reviewed ACM's Policy Controller and explored some of its useful features. You created a constraint from the provided templates as well as creating your own templates. You verified that constraints can be used to enforce policies and to audit policy violations before enforcing them in a production environment.

## History

```sh
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ history 
    1  ZONE=us-west1-b
    2  export CLUSTER_NAME=gke
    3  export PROJECT_ID=$(gcloud config get-value project)
    4  export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} \
    --format="value(projectNumber)")
    5  gcloud container clusters get-credentials gke --zone $ZONE --project $PROJECT_ID
    6  gcloud beta container fleet config-management enable 
    7  cat <<EOF > config-management.yaml
apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
applySpecVersion: 1
spec:
  # Set to true to install and enable Policy Controller
  policyController:
    enabled: true
    # Uncomment to prevent the template library from being installed
    # templateLibraryInstalled: false
    # Uncomment to enable support for referential constraints
    # referentialRulesEnabled: true
    # Uncomment to disable audit, adjust value to set audit interval
    # auditIntervalSeconds: 0
    # Uncomment to log all denies and dryrun failures
    # logDeniesEnabled: true
    # Uncomment to exempt namespaces
    # exemptableNamespaces: ["namespace-name"]
  # ...other fields...
EOF

    8  ls
    9  cat config-management.yaml 
   10  gcloud beta container fleet config-management apply     --membership=${CLUSTER_NAME}-connect     --config=config-management.yaml     --project=$PROJECT_ID
   11  watch gcloud beta container fleet config-management status     --project=$PROJECT_ID
   12  kubectl get constrainttemplates
   13  cat <<EOF > ns-must-have-geo.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-geo
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels:
      - key: "geo"
EOF

   14  kubectl apply -f ns-must-have-geo.yaml
   15  kubectl create namespace test
   16  cat <<EOF > test-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    geo: eu-west1
EOF

   17  kubectl apply -f test-namespace.yaml
   18  kubectl get K8sRequiredLabels ns-must-have-geo -o yaml
   19  apiVersion: constraints.gatekeeper.sh/v1beta1
   20  kind: K8sRequiredLabels
   21  metadata:
   22  spec:
   23  ls
   24  vi ns-must-have-geo.yaml 
   25  kubectl apply -f ns-must-have-geo.yaml
   26  cat ns-must-have-geo.yaml 
   27  kubectl create namespace ns-with-violations
   28  kubectl get K8sRequiredLabels ns-must-have-geo -o yaml
   29  cat <<EOF > k8sdenyname-constraint-template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sdenyname
spec:
  crd:
    spec:
      names:
        kind: K8sDenyName
      validation:
        openAPIV3Schema:
          properties:
            invalidName:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sdenynames
        violation[{"msg": msg}] {
          input.review.object.metadata.name == input.parameters.invalidName
          msg := sprintf("The name %v is not allowed", [input.parameters.invalidName])
        }
EOF

   30  kubectl apply -f k8sdenyname-constraint-template.yaml
   31  cat k8sdenyname-constraint-template.yaml 
   32  cat <<EOF > k8sdenyname-constraint.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sDenyName
metadata:
  name: no-policy-violation
spec:
  parameters:
    invalidName: "policy-violation"
EOF

   33  kubectl apply -f k8sdenyname-constraint.yaml
   34  kubectl create namespace policy-violation
   35  history 
student_01_2c1b6dd12803@cloudshell:~ (qwiklabs-gcp-01-fe43bbf70613)$ 
```

