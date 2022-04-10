---
layout: single
title:  "Kubernetes Custom Resource Definitions"
date:   2022-04-10 10:59:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
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



# Kubernetes Custom Resource Definitions

We can extend the Kubernetes API with CustomResourceDefinitions (CRDs).

In our setup, let us check if we have any CRDs.



```sh
pradeep@learnk8s$ kubectl get crd
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2022-03-19T18:18:46Z
bgppeers.crd.projectcalico.org                        2022-03-19T18:18:46Z
blockaffinities.crd.projectcalico.org                 2022-03-19T18:18:46Z
clusterinformations.crd.projectcalico.org             2022-03-19T18:18:46Z
felixconfigurations.crd.projectcalico.org             2022-03-19T18:18:46Z
globalnetworkpolicies.crd.projectcalico.org           2022-03-19T18:18:47Z
globalnetworksets.crd.projectcalico.org               2022-03-19T18:18:47Z
hostendpoints.crd.projectcalico.org                   2022-03-19T18:18:47Z
ipamblocks.crd.projectcalico.org                      2022-03-19T18:18:47Z
ipamconfigs.crd.projectcalico.org                     2022-03-19T18:18:47Z
ipamhandles.crd.projectcalico.org                     2022-03-19T18:18:47Z
ippools.crd.projectcalico.org                         2022-03-19T18:18:48Z
kubecontrollersconfigurations.crd.projectcalico.org   2022-03-19T18:18:48Z
networkpolicies.crd.projectcalico.org                 2022-03-19T18:18:48Z
networksets.crd.projectcalico.org                     2022-03-19T18:18:48Z

```

Let us describe one of these.

```sh
pradeep@learnk8s$ kubectl describe crd bgpconfigurations.crd.projectcalico.org
Name:         bgpconfigurations.crd.projectcalico.org
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-03-19T18:18:46Z
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
    Time:         2022-03-19T18:18:46Z
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
    Time:            2022-03-19T18:18:46Z
  Resource Version:  368
  UID:               4ad43247-6b43-4393-82ba-5852ec281f74
Spec:
  Conversion:
    Strategy:  None
  Group:       crd.projectcalico.org
  Names:
    Kind:       BGPConfiguration
    List Kind:  BGPConfigurationList
    Plural:     bgpconfigurations
    Singular:   bgpconfiguration
  Scope:        Cluster
  Versions:
    Name:  v1
    Schema:
      openAPIV3Schema:
        Description:  BGPConfiguration contains the configuration for any BGP routing.
        Properties:
          API Version:
            Description:  APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
            Type:         string
          Kind:
            Description:  Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
            Type:         string
          Metadata:
            Type:  object
          Spec:
            Description:  BGPConfigurationSpec contains the values of the BGP configuration.
            Properties:
              As Number:
                Description:  ASNumber is the default AS number used by a node. [Default: 64512]
                Format:       int32
                Type:         integer
              Communities:
                Description:  Communities is a list of BGP community values and their arbitrary names for tagging routes.
                Items:
                  Description:  Community contains standard or large community value and its name.
                  Properties:
                    Name:
                      Description:  Name given to community value.
                      Type:         string
                    Value:
                      Description:  Value must be of format `aa:nn` or `aa:nn:mm`. For standard community use `aa:nn` format, where `aa` and `nn` are 16 bit number. For large community use `aa:nn:mm` format, where `aa`, `nn` and `mm` are 32 bit number. Where, `aa` is an AS Number, `nn` and `mm` are per-AS identifier.
                      Pattern:      ^(\d+):(\d+)$|^(\d+):(\d+):(\d+)$
                      Type:         string
                  Type:             object
                Type:               array
              Listen Port:
                Description:  ListenPort is the port where BGP protocol should listen. Defaults to 179
                Maximum:      65535
                Minimum:      1
                Type:         integer
              Log Severity Screen:
                Description:  LogSeverityScreen is the log severity above which logs are sent to the stdout. [Default: INFO]
                Type:         string
              Node To Node Mesh Enabled:
                Description:  NodeToNodeMeshEnabled sets whether full node to node BGP mesh is enabled. [Default: true]
                Type:         boolean
              Prefix Advertisements:
                Description:  PrefixAdvertisements contains per-prefix advertisement configuration.
                Items:
                  Description:  PrefixAdvertisement configures advertisement properties for the specified CIDR.
                  Properties:
                    Cidr:
                      Description:  CIDR for which properties should be advertised.
                      Type:         string
                    Communities:
                      Description:  Communities can be list of either community names already defined in `Specs.Communities` or community value of format `aa:nn` or `aa:nn:mm`. For standard community use `aa:nn` format, where `aa` and `nn` are 16 bit number. For large community use `aa:nn:mm` format, where `aa`, `nn` and `mm` are 32 bit number. Where,`aa` is an AS Number, `nn` and `mm` are per-AS identifier.
                      Items:
                        Type:  string
                      Type:    array
                  Type:        object
                Type:          array
              Service Cluster I Ps:
                Description:  ServiceClusterIPs are the CIDR blocks from which service cluster IPs are allocated. If specified, Calico will advertise these blocks, as well as any cluster IPs within them.
                Items:
                  Description:  ServiceClusterIPBlock represents a single allowed ClusterIP CIDR block.
                  Properties:
                    Cidr:
                      Type:  string
                  Type:      object
                Type:        array
              Service External I Ps:
                Description:  ServiceExternalIPs are the CIDR blocks for Kubernetes Service External IPs. Kubernetes Service ExternalIPs will only be advertised if they are within one of these blocks.
                Items:
                  Description:  ServiceExternalIPBlock represents a single allowed External IP CIDR block.
                  Properties:
                    Cidr:
                      Type:  string
                  Type:      object
                Type:        array
              Service Load Balancer I Ps:
                Description:  ServiceLoadBalancerIPs are the CIDR blocks for Kubernetes Service LoadBalancer IPs. Kubernetes Service status.LoadBalancer.Ingress IPs will only be advertised if they are within one of these blocks.
                Items:
                  Description:  ServiceLoadBalancerIPBlock represents a single allowed LoadBalancer IP CIDR block.
                  Properties:
                    Cidr:
                      Type:  string
                  Type:      object
                Type:        array
            Type:            object
        Type:                object
    Served:                  true
    Storage:                 true
Status:
  Accepted Names:
    Kind:       BGPConfiguration
    List Kind:  BGPConfigurationList
    Plural:     bgpconfigurations
    Singular:   bgpconfiguration
  Conditions:
    Last Transition Time:  2022-03-19T18:18:46Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-03-19T18:18:46Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1
Events:  <none> 

```



Now, let us create our own custom resource definition

```yaml
pradeep@learnk8s$ cat democrd-def.yaml 
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: posts.blog.pradeepgadde.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: blog.pradeepgadde.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                postSpec:
                  type: string
                category:
                  type: string
                
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: posts
    # singular name to be used as an alias on the CLI and for display
    singular: post
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: Post
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ps

```



```sh
pradeep@learnk8s$ kubectl create -f democrd-def.yaml 
customresourcedefinition.apiextensions.k8s.io/posts.blog.pradeepgadde.com created
pradeep@learnk8s$

```
List all CRDs again and confirm our newly created one is shown
```sh
pradeep@learnk8s$ kubectl get crd
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2022-03-19T18:18:46Z
bgppeers.crd.projectcalico.org                        2022-03-19T18:18:46Z
blockaffinities.crd.projectcalico.org                 2022-03-19T18:18:46Z
clusterinformations.crd.projectcalico.org             2022-03-19T18:18:46Z
felixconfigurations.crd.projectcalico.org             2022-03-19T18:18:46Z
globalnetworkpolicies.crd.projectcalico.org           2022-03-19T18:18:47Z
globalnetworksets.crd.projectcalico.org               2022-03-19T18:18:47Z
hostendpoints.crd.projectcalico.org                   2022-03-19T18:18:47Z
ipamblocks.crd.projectcalico.org                      2022-03-19T18:18:47Z
ipamconfigs.crd.projectcalico.org                     2022-03-19T18:18:47Z
ipamhandles.crd.projectcalico.org                     2022-03-19T18:18:47Z
ippools.crd.projectcalico.org                         2022-03-19T18:18:48Z
kubecontrollersconfigurations.crd.projectcalico.org   2022-03-19T18:18:48Z
networkpolicies.crd.projectcalico.org                 2022-03-19T18:18:48Z
networksets.crd.projectcalico.org                     2022-03-19T18:18:48Z
posts.blog.pradeepgadde.com                           2022-04-10T15:02:25Z

```
Describe the new CRD
```sh
pradeep@learnk8s$ kubectl describe crd posts.blog.pradeepgadde.com
Name:         posts.blog.pradeepgadde.com
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  apiextensions.k8s.io/v1
Kind:         CustomResourceDefinition
Metadata:
  Creation Timestamp:  2022-04-10T15:02:25Z
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
          f:shortNames:
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
    Time:         2022-04-10T15:02:25Z
    API Version:  apiextensions.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:conversion:
          .:
          f:strategy:
        f:group:
        f:names:
          f:kind:
          f:listKind:
          f:plural:
          f:shortNames:
          f:singular:
        f:scope:
        f:versions:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2022-04-10T15:02:25Z
  Resource Version:  208536
  UID:               3e19dbef-2f54-4af0-81da-b97136f829b9
Spec:
  Conversion:
    Strategy:  None
  Group:       blog.pradeepgadde.com
  Names:
    Kind:       Post
    List Kind:  PostList
    Plural:     posts
    Short Names:
      ps
    Singular:  post
  Scope:       Namespaced
  Versions:
    Name:  v1
    Schema:
      openAPIV3Schema:
        Properties:
          Spec:
            Properties:
              Category:
                Type:  string
              Post Spec:
                Type:  string
            Type:      object
        Type:          object
    Served:            true
    Storage:           true
Status:
  Accepted Names:
    Kind:       Post
    List Kind:  PostList
    Plural:     posts
    Short Names:
      ps
    Singular:  post
  Conditions:
    Last Transition Time:  2022-04-10T15:02:25Z
    Message:               no conflicts found
    Reason:                NoConflicts
    Status:                True
    Type:                  NamesAccepted
    Last Transition Time:  2022-04-10T15:02:25Z
    Message:               the initial names have been accepted
    Reason:                InitialNamesAccepted
    Status:                True
    Type:                  Established
  Stored Versions:
    v1
Events:  <none>
```

The same is available when we list all api-resources. 

```sh
pradeep@learnk8s$ kubectl api-resources | grep pradeep
posts                             ps           blog.pradeepgadde.com/v1               true         Post
```

Note the shortform `ps` as we mentioned in the definition. You can then manage your `Post` objects using `kubectl`. For example:

Let us list all resources of this type `ps`.

```sh
pradeep@learnk8s$ kubectl get ps
No resources found in default namespace.
```

After the CustomResourceDefinition object has been created, you can create custom objects. Custom objects can contain custom fields. These fields can contain arbitrary JSON. In the following example, the `postSpec` and `category` custom fields are set in a custom object of kind `Post`. The kind `Post` comes from the spec of the CustomResourceDefinition object you created above.

```yaml
pradeep@learnk8s$ cat my-post.yaml 
apiVersion: "blog.pradeepgadde.com/v1"
kind: Post
metadata:
  name: my-new-post-object
spec:
  postSpec: "my-new-shiny-post"
  category: learning

```

Create this object 

```sh
pradeep@learnk8s$ kubectl create -f my-post.yaml 
post.blog.pradeepgadde.com/my-new-post-object created

```

Get the list of objects, there should be only one of this type `ps`.

```sh
pradeep@learnk8s$ kubectl get ps                
NAME                 AGE
my-new-post-object   4s

```

Describe it

```sh
pradeep@learnk8s$ kubectl describe ps
Name:         my-new-post-object
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  blog.pradeepgadde.com/v1
Kind:         Post
Metadata:
  Creation Timestamp:  2022-04-10T15:11:49Z
  Generation:          1
  Managed Fields:
    API Version:  blog.pradeepgadde.com/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:category:
        f:postSpec:
    Manager:         kubectl-create
    Operation:       Update
    Time:            2022-04-10T15:11:49Z
  Resource Version:  209127
  UID:               9f04adbe-63a7-4911-8a87-d7caa12b83c5
Spec:
  Category:   learning
  Post Spec:  my-new-shiny-post
Events:       <none>
 

```

Delete the CRD definition

```sh
pradeep@learnk8s$ kubectl delete -f democrd-def.yaml 
customresourcedefinition.apiextensions.k8s.io "posts.blog.pradeepgadde.com" deleted
```

After deleting the CRD definition, attempt to list all the `ps` resources again!

```sh
pradeep@learnk8s$ kubectl get ps
Error from server (NotFound): Unable to list "blog.pradeepgadde.com/v1, Resource=posts": the server could not find the requested resource (get posts.blog.pradeepgadde.com)

```
If you later recreate the same CustomResourceDefinition, it will start out empty.
```sh
pradeep@learnk8s$ kubectl create -f democrd-def.yaml
customresourcedefinition.apiextensions.k8s.io/posts.blog.pradeepgadde.com created

```
```sh
pradeep@learnk8s$ kubectl get ps                    
No resources found in default namespace.
```
As expected, the list is empty.

This concludes our discussion on basics of Kubernetes Custom Resource Definitions (CRDs).



