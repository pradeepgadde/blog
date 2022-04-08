---
layout: single
title:  "Kubectl -o name option"
date:   2022-04-04 10:57:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---


# Kubectl -o name option

Let us take a look at the help output of `kubectl get` command.

```sh
pradeep@learnk8s$ kubectl get -h
Display one or many resources.

 Prints a table of the most important information about the specified resources. You can filter the list using a label
selector and the --selector flag. If the desired resource type is namespaced you will only see results in your current
namespace unless you pass --all-namespaces.

 By specifying the output as 'template' and providing a Go template as the value of the --template flag, you can filter
the attributes of the fetched resources.

Use "kubectl api-resources" for a complete list of supported resources.

Examples:
  # List all pods in ps output format
  kubectl get pods

  # List all pods in ps output format with more information (such as node name)
  kubectl get pods -o wide

  # List a single replication controller with specified NAME in ps output format
  kubectl get replicationcontroller web

  # List deployments in JSON output format, in the "v1" version of the "apps" API group
  kubectl get deployments.v1.apps -o json

  # List a single pod in JSON output format
  kubectl get -o json pod web-pod-13je7

  # List a pod identified by type and name specified in "pod.yaml" in JSON output format
  kubectl get -f pod.yaml -o json

  # List resources from a directory with kustomization.yaml - e.g. dir/kustomization.yaml
  kubectl get -k dir/

  # Return only the phase value of the specified pod
  kubectl get -o template pod/web-pod-13je7 --template={{.status.phase}}

  # List resource information in custom columns
  kubectl get pod test-pod -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image

  # List all replication controllers and services together in ps output format
  kubectl get rc,services

  # List one or more resources by their type and names
  kubectl get rc/web service/frontend pods/web-pod-13je7

Options:
  -A, --all-namespaces=false: If present, list the requested object(s) across all namespaces. Namespace in current
context is ignored even if specified with --namespace.
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or map key is missing in
the template. Only applies to golang and jsonpath output formats.
      --chunk-size=500: Return large lists in chunks rather than all at once. Pass 0 to disable. This flag is beta and
may change in the future.
      --field-selector='': Selector (field query) to filter on, supports '=', '==', and '!='.(e.g. --field-selector
key1=value1,key2=value2). The server only supports a limited number of field queries per type.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to get from a server.
      --ignore-not-found=false: If the requested object does not exist the command will return exit code 0.
  -k, --kustomize='': Process the kustomization directory. This flag can't be used together with -f or -R.
  -L, --label-columns=[]: Accepts a comma separated list of labels that are going to be presented as columns. Names are
case-sensitive. You can also use multiple flag options like -L label1 -L label2...
      --no-headers=false: When using the default or custom-column output format, don't print headers (default print
headers).
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns-file|custom-columns|wide
See custom columns [https://kubernetes.io/docs/reference/kubectl/overview/#custom-columns], golang template
[http://golang.org/pkg/text/template/#pkg-overview] and jsonpath template
[https://kubernetes.io/docs/reference/kubectl/jsonpath/].
      --output-watch-events=false: Output watch event objects when --watch or --watch-only is used. Existing objects are
output as initial ADDED events.
      --raw='': Raw URI to request from the server.  Uses the transport specified by the kubeconfig file.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you want to manage
related manifests organized within the same directory.
  -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2)
      --server-print=true: If true, have the server return the appropriate table output. Supports extension APIs and
CRDs.
      --show-kind=false: If present, list the resource type for the requested object(s).
      --show-labels=false: When printing, show all labels as the last column (default hide labels column)
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or YAML format.
      --sort-by='': If non-empty, sort list types using this field specification.  The field specification is expressed
as a JSONPath expression (e.g. '{.metadata.name}'). The field in the API resource specified by this JSONPath expression
must be an integer or a string.
      --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The
template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].
  -w, --watch=false: After listing/getting the requested object, watch for changes.
      --watch-only=false: Watch for changes to the requested object(s), without listing/getting first.

Usage:
  kubectl get
[(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns|custom-columns-file|wide]
(TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags] [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
```

The `-o` option supports many values.

` -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns-file|custom-columns|wide`

If we just need the list of names of resources, we can make use of the `-o name` option with the kubectl command.

Here are some examples.



```sh
pradeep@learnk8s$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
fieldref-demo           1/1     Running   0          19m
resourcefieldref-demo   1/1     Running   0          9m22s`
```

```sh
pradeep@learnk8s$ kubectl get pods -o name
pod/fieldref-demo
pod/resourcefieldref-demo
```

```sh
pradeep@learnk8s$ kubectl get nodes -o name
node/minikube
node/minikube-m02
```

```sh
pradeep@learnk8s$ kubectl get svc -o name
service/kubernetes
```

List of all API resources

```sh
pradeep@learnk8s$ kubectl api-resources -o name
bindings
componentstatuses
configmaps
endpoints
events
limitranges
namespaces
nodes
persistentvolumeclaims
persistentvolumes
pods
podtemplates
replicationcontrollers
resourcequotas
secrets
serviceaccounts
services
mutatingwebhookconfigurations.admissionregistration.k8s.io
validatingwebhookconfigurations.admissionregistration.k8s.io
customresourcedefinitions.apiextensions.k8s.io
apiservices.apiregistration.k8s.io
controllerrevisions.apps
daemonsets.apps
deployments.apps
replicasets.apps
statefulsets.apps
tokenreviews.authentication.k8s.io
localsubjectaccessreviews.authorization.k8s.io
selfsubjectaccessreviews.authorization.k8s.io
selfsubjectrulesreviews.authorization.k8s.io
subjectaccessreviews.authorization.k8s.io
horizontalpodautoscalers.autoscaling
cronjobs.batch
jobs.batch
certificatesigningrequests.certificates.k8s.io
leases.coordination.k8s.io
bgpconfigurations.crd.projectcalico.org
bgppeers.crd.projectcalico.org
blockaffinities.crd.projectcalico.org
clusterinformations.crd.projectcalico.org
felixconfigurations.crd.projectcalico.org
globalnetworkpolicies.crd.projectcalico.org
globalnetworksets.crd.projectcalico.org
hostendpoints.crd.projectcalico.org
ipamblocks.crd.projectcalico.org
ipamconfigs.crd.projectcalico.org
ipamhandles.crd.projectcalico.org
ippools.crd.projectcalico.org
kubecontrollersconfigurations.crd.projectcalico.org
networkpolicies.crd.projectcalico.org
networksets.crd.projectcalico.org
endpointslices.discovery.k8s.io
events.events.k8s.io
flowschemas.flowcontrol.apiserver.k8s.io
prioritylevelconfigurations.flowcontrol.apiserver.k8s.io
ingressclasses.networking.k8s.io
ingresses.networking.k8s.io
networkpolicies.networking.k8s.io
runtimeclasses.node.k8s.io
poddisruptionbudgets.policy
podsecuritypolicies.policy
clusterrolebindings.rbac.authorization.k8s.io
clusterroles.rbac.authorization.k8s.io
rolebindings.rbac.authorization.k8s.io
roles.rbac.authorization.k8s.io
priorityclasses.scheduling.k8s.io
csidrivers.storage.k8s.io
csinodes.storage.k8s.io
csistoragecapacities.storage.k8s.io
storageclasses.storage.k8s.io
volumeattachments.storage.k8s.i
```

