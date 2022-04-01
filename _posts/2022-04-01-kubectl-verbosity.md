---
layout: single
title:  "Kubectl Verbosity"
date:   2022-04-01 12:55:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
classes: wide
toc: true
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

# Kubectl Verbosity

Kubectl verbosity is controlled with the -v or --v flags followed by an integer (0 to 9) representing the log level.
For example, 
--v=9 	Display HTTP request contents without truncation of contents.

Sample output for `--v=8 Display HTTP request contents`
```sh

lab@k8s1:~$ kubectl get nodes --v=8
I0401 01:08:21.132227 2015543 loader.go:372] Config loaded from file:  /home/lab/.kube/config
I0401 01:08:21.140214 2015543 round_trippers.go:432] GET https://192.168.100.1:6443/api/v1/nodes?limit=500
I0401 01:08:21.140238 2015543 round_trippers.go:438] Request Headers:
I0401 01:08:21.140248 2015543 round_trippers.go:442]     Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
I0401 01:08:21.140257 2015543 round_trippers.go:442]     User-Agent: kubectl/v1.22.0 (linux/amd64) kubernetes/c2b5237
I0401 01:08:21.152861 2015543 round_trippers.go:457] Response Status: 200 OK in 12 milliseconds
I0401 01:08:21.153037 2015543 round_trippers.go:460] Response Headers:
I0401 01:08:21.153140 2015543 round_trippers.go:463]     X-Kubernetes-Pf-Flowschema-Uid: 10ca7bc4-0deb-4382-af07-13786e908ab6
I0401 01:08:21.153254 2015543 round_trippers.go:463]     X-Kubernetes-Pf-Prioritylevel-Uid: e1a275c1-a809-4c6b-8e09-6a96b976ceb6
I0401 01:08:21.153328 2015543 round_trippers.go:463]     Date: Fri, 01 Apr 2022 08:08:21 GMT
I0401 01:08:21.153390 2015543 round_trippers.go:463]     Audit-Id: 72ed6b06-94fb-4b92-b4b5-17d0c5d33cd1
I0401 01:08:21.153442 2015543 round_trippers.go:463]     Cache-Control: no-cache, private
I0401 01:08:21.153512 2015543 round_trippers.go:463]     Content-Type: application/json
I0401 01:08:21.153859 2015543 request.go:1181] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1","metadata":{"resourceVersion":"250015"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Status","type":"string","format":"","description":"The status of the node","priority":0},{"name":"Roles","type":"string","format":"","description":"The roles of the node","priority":0},{"name":"Age","type":"string","format":"","description":"CreationTimestamp is a timestamp representing the server time when this object was created. It is not guaranteed to be set in happens-before order across separate operations. Clients may not set this value. It is represented in RFC3339 fo [truncated 11196 chars]
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   2d17h   v1.22.8
k8s2   Ready    <none>                 2d16h   v1.22.8
k8s3   Ready    <none>                 2d15h   v1.22.8
lab@k8s1:~$
```

```sh
lab@k8s1:~$ kubectl get pods --v=8
I0401 01:09:53.596462 2016120 loader.go:372] Config loaded from file:  /home/lab/.kube/config
I0401 01:09:53.605561 2016120 round_trippers.go:432] GET https://192.168.100.1:6443/api/v1/namespaces/default/pods?limit=500
I0401 01:09:53.605604 2016120 round_trippers.go:438] Request Headers:
I0401 01:09:53.605618 2016120 round_trippers.go:442]     Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json
I0401 01:09:53.605629 2016120 round_trippers.go:442]     User-Agent: kubectl/v1.22.0 (linux/amd64) kubernetes/c2b5237
I0401 01:09:53.618308 2016120 round_trippers.go:457] Response Status: 200 OK in 12 milliseconds
I0401 01:09:53.618334 2016120 round_trippers.go:460] Response Headers:
I0401 01:09:53.618342 2016120 round_trippers.go:463]     Date: Fri, 01 Apr 2022 08:09:53 GMT
I0401 01:09:53.618347 2016120 round_trippers.go:463]     Audit-Id: 05eaeecf-6184-4406-b9ae-75f549f6009a
I0401 01:09:53.618355 2016120 round_trippers.go:463]     Cache-Control: no-cache, private
I0401 01:09:53.618365 2016120 round_trippers.go:463]     Content-Type: application/json
I0401 01:09:53.618375 2016120 round_trippers.go:463]     X-Kubernetes-Pf-Flowschema-Uid: 10ca7bc4-0deb-4382-af07-13786e908ab6
I0401 01:09:53.618386 2016120 round_trippers.go:463]     X-Kubernetes-Pf-Prioritylevel-Uid: e1a275c1-a809-4c6b-8e09-6a96b976ceb6
I0401 01:09:53.620382 2016120 request.go:1181] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1","metadata":{"resourceVersion":"250143"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Ready","type":"string","format":"","description":"The aggregate readiness state of this pod for accepting traffic.","priority":0},{"name":"Status","type":"string","format":"","description":"The aggregate status of the containers in this pod.","priority":0},{"name":"Restarts","type":"string","format":"","description":"The number of times the containers in this pod have been restarted and when the last container in this pod has restarted.","priority":0},{"name":"Age","type":"st [truncated 19784 chars]
NAME                   READY   STATUS    RESTARTS   AGE
web-79d88c97d6-4w4nl   1/1     Running   0          2d13h
web-79d88c97d6-7ktck   1/1     Running   0          2d13h
web-79d88c97d6-8tvsk   1/1     Running   0          2d13h
web-79d88c97d6-96sd9   1/1     Running   0          2d13h
web-79d88c97d6-9tzlh   1/1     Running   0          2d13h
web-79d88c97d6-brtgx   1/1     Running   0          2d13h
web-79d88c97d6-kngc4   1/1     Running   0          2d13h
web-79d88c97d6-p5vfg   1/1     Running   0          2d13h
web-79d88c97d6-rbhpr   1/1     Running   0          2d13h
lab@k8s1:~$
```

Sample output for `--v=6 	Display requested resources`

```sh
lab@k8s1:~$ kubectl get pods --v=6
I0401 01:10:17.042637 2016316 loader.go:372] Config loaded from file:  /home/lab/.kube/config
I0401 01:10:17.063058 2016316 round_trippers.go:454] GET https://192.168.100.1:6443/api/v1/namespaces/default/pods?limit=500 200 OK in 10 milliseconds
NAME                   READY   STATUS    RESTARTS   AGE
web-79d88c97d6-4w4nl   1/1     Running   0          2d13h
web-79d88c97d6-7ktck   1/1     Running   0          2d13h
web-79d88c97d6-8tvsk   1/1     Running   0          2d13h
web-79d88c97d6-96sd9   1/1     Running   0          2d13h
web-79d88c97d6-9tzlh   1/1     Running   0          2d13h
web-79d88c97d6-brtgx   1/1     Running   0          2d13h
web-79d88c97d6-kngc4   1/1     Running   0          2d13h
web-79d88c97d6-p5vfg   1/1     Running   0          2d13h
web-79d88c97d6-rbhpr   1/1     Running   0          2d13h
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get pods --v=9
I0401 01:13:30.893071 2017538 loader.go:372] Config loaded from file:  /home/lab/.kube/config
I0401 01:13:30.903470 2017538 round_trippers.go:435] curl -v -XGET  -H "Accept: application/json;as=Table;v=v1;g=meta.k8s.io,application/json;as=Table;v=v1beta1;g=meta.k8s.io,application/json" -H "User-Agent: kubectl/v1.22.0 (linux/amd64) kubernetes/c2b5237" 'https://192.168.100.1:6443/api/v1/namespaces/default/pods?limit=500'
I0401 01:13:30.914694 2017538 round_trippers.go:454] GET https://192.168.100.1:6443/api/v1/namespaces/default/pods?limit=500 200 OK in 11 milliseconds
I0401 01:13:30.914870 2017538 round_trippers.go:460] Response Headers:
I0401 01:13:30.914987 2017538 round_trippers.go:463]     Cache-Control: no-cache, private
I0401 01:13:30.915107 2017538 round_trippers.go:463]     Content-Type: application/json
I0401 01:13:30.915169 2017538 round_trippers.go:463]     X-Kubernetes-Pf-Flowschema-Uid: 10ca7bc4-0deb-4382-af07-13786e908ab6
I0401 01:13:30.915223 2017538 round_trippers.go:463]     X-Kubernetes-Pf-Prioritylevel-Uid: e1a275c1-a809-4c6b-8e09-6a96b976ceb6
I0401 01:13:30.915276 2017538 round_trippers.go:463]     Date: Fri, 01 Apr 2022 08:13:30 GMT
I0401 01:13:30.915343 2017538 round_trippers.go:463]     Audit-Id: 97dbe3b8-ea5a-4483-bf18-b02c44567001
I0401 01:13:30.915983 2017538 request.go:1181] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1","metadata":{"resourceVersion":"250450"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to request the generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Ready","type":"string","format":"","description":"The aggregate readiness state of this pod for accepting traffic.","priority":0},{"name":"Status","type":"string","format":"","description":"The aggregate status of the containers in this pod.","priority":0},{"name":"Restarts","type":"string","format":"","description":"The number of times the containers in this pod have been restarted and when the last container in this pod has restarted.","priority":0},{"name":"Age","type":"string","format":"","description":"CreationTimestamp is a timestamp representing the server time when this object was created. It is not guaranteed to be set in happens-before order across separate operations. Clients may not set this value. It is represented in RFC3339 form and is in UTC.\n\nPopulated by the system. Read-only. Null for lists. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata","priority":0},{"name":"IP","type":"string","format":"","description":"IP address allocated to the pod. Routable at least within the cluster. Empty if not yet allocated.","priority":1},{"name":"Node","type":"string","format":"","description":"NodeName is a request to schedule this pod onto a specific node. If it is non-empty, the scheduler simply schedules this pod onto that node, assuming that it fits resource requirements.","priority":1},{"name":"Nominated Node","type":"string","format":"","description":"nominatedNodeName is set only when this pod preempts other pods on the node, but it cannot be scheduled right away as preemption victims receive their graceful termination periods. This field does not guarantee that the pod will be scheduled on this node. Scheduler may decide to place the pod elsewhere if other nodes become available sooner. Scheduler may also decide to give the resources on this node to a higher priority pod that is created after preemption. As a result, this field may be different than PodSpec.nodeName when the pod is scheduled.","priority":1},{"name":"Readiness Gates","type":"string","format":"","description":"If specified, all readiness gates will be evaluated for pod readiness. A pod is ready when all its containers are ready AND all conditions specified in the readiness gates have status equal to \"True\" More info: https://git.k8s.io/enhancements/keps/sig-network/580-pod-readiness-gates","priority":1}],"rows":[{"cells":["web-79d88c97d6-4w4nl","1/1","Running","0","2d13h","10.244.2.18","k8s2","\u003cnone\u003e","\u003cnone\u003e"],"object":{"kind":"PartialObjectMetadata","apiVersion":"meta.k8s.io/v1","metadata":{"name":"web-79d88c97d6-4w4nl","generateName":"web-79d88c97d6-","namespace":"default","uid":"ca77b1cb-9adf-4ae0-be9d-15836a452733","resourceVersion":"91985","creationTimestamp":"2022-03-29T18:27:05Z","labels":{"app":"web","pod-template-hash":"79d88c97d6"},"ownerReferences":[{"apiVersion":"apps/v1","kind":"ReplicaSet","name":"web-79d88c97d6","uid":"710786df-5695-495e-836f-6e6b19235157","controller":true,"blockOwnerDeletion":true}],"managedFields":[{"manager":"kube-controller-manager","operation":"Update","apiVersion":"v1","time":"2022-03-29T18:27:05Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:generateName":{},"f:labels":{".":{},"f:app":{},"f:pod-template-hash":{}},"f:ownerReferences":{".":{},"k:{\"uid\":\"710786df-5695-495e-836f-6e6b19235157\"}":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"hello-app\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2022-03-31T00:48:13Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"10.244.2.18\"}":{".":{},"f:ip":{}}},"f:startTime":{}}},"subresource":"status"}]}}},{"cells":["web-79d88c97d6-7ktck","1/1","Running","0","2d13h","10.244.2.20","k8s2","\u003cnone\u003e","\u003cnone\u003e"],"object":{"kind":"PartialObjectMetadata","apiVersion":"meta.k8s.io/v1","metadata":{"name":"web-79d88c97d6-7ktck","generateName":"web-79d88c97d6-","namespace":"default","uid":"070fe0df-a60d-4d7d-9e6f-ce9b8f93f1bd","resourceVersion":"91987","creationTimestamp":"2022-03-29T18:27:05Z","labels":{"app":"web","pod-template-hash":"79d88c97d6"},"ownerReferences":[{"apiVersion":"apps/v1","kind":"ReplicaSet","name":"web-79d88c97d6","uid":"710786df-5695-495e-836f-6e6b19235157","controller":true,"blockOwnerDeletion":true}],"managedFields":[{"manager":"kube-controller-manager","operation":"Update","apiVersion":"v1","time":"2022-03-29T18:27:05Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:generateName":{},"f:labels":{".":{},"f:app":{},"f:pod-template-hash":{}},"f:ownerReferences":{".":{},"k:{\"uid\":\"710786df-5695-495e-836f-6e6b19235157\"}":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"hello-app\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2022-03-31T00:48:13Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"10.244.2.20\"}":{".":{},"f:ip":{}}},"f:startTime":{}}},"subresource":"status"}]}}},{"cells":["web-79d88c97d6-8tvsk","1/1","Running","0","2d13h","10.244.3.13","k8s3","\u003cnone\u003e","\u003cnone\u003e"],"object":{"kind":"PartialObjectMetadata","apiVersion":"meta.k8s.io/v1","metadata":{"name":"web-79d88c97d6-8tvsk","generateName":"web-79d88c97d6-","namespace":"default","uid":"72a93db6-3e4a-46f2-ae70-c06092630599","resourceVersion":"92002","creationTimestamp":"2022-03-29T18:34:45Z","labels":{"app":"web","pod-template-hash":"79d88c97d6"},"ownerReferences":[{"apiVersion":"apps/v1","kind":"ReplicaSet","name":"web-79d88c97d6","uid":"710786df-5695-495e-836f-6e6b19235157","controller":true,"blockOwnerDeletion":true}],"managedFields":[{"manager":"kube-controller-manager","operation":"Update","apiVersion":"v1","time":"2022-03-29T18:34:45Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:generateName":{},"f:labels":{".":{},"f:app":{},"f:pod-template-hash":{}},"f:ownerReferences":{".":{},"k:{\"uid\":\"710786df-5695-495e-836f-6e6b19235157\"}":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"hello-app\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2022-03-31T00:48:16Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"10.244.3.13\"}":{".":{},"f:ip":{}}},"f:startTime":{}}},"subresource":"status"}]}}},{"cells":["web-79d88c97d6-96sd9","1/1","Running","0","2d13h","10.244.2.19","k8s2","\u003cnone\u003e","\u003cnone\u003e"],"object":{"kind":"PartialObjectMetadata","apiVersion":"meta.k8s.io/v1","metadata":{"name":"web-79d88c97d6-96sd9","generateName":"web-79d88c97d6-","namespace":"default","uid":"87c900af-8b55-4877-a2f3-667928655178","resourceVersion":"91983","creationTimestamp":"2022-03-29T18:27:05Z","labels":{"app":"web","pod-template-hash":"79d88c97d6"},"ownerReferences":[{"apiVersion":"apps/v1","kind":"ReplicaSet","name":"web-79d88c97d6","uid":"710786df-5695-495e-836f-6e6b19235157","controller":true,"blockOwnerDeletion":true}],"managedFields":[{"manager":"kube-controller-manager","operation":"Update","apiVersion":"v1","time":"2022-03-29T18:27:05Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:generateName":{},"f:labels":{".":{},"f:app":{},"f:pod-template-hash":{}},"f:ownerReferences":{".":{},"k:{\"uid\":\"710786df-5695-495e-836f-6e6b19235157\"}":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"hello-app\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"kubelet","operation":"Up [truncated 10568 chars]
NAME                   READY   STATUS    RESTARTS   AGE
web-79d88c97d6-4w4nl   1/1     Running   0          2d13h
web-79d88c97d6-7ktck   1/1     Running   0          2d13h
web-79d88c97d6-8tvsk   1/1     Running   0          2d13h
web-79d88c97d6-96sd9   1/1     Running   0          2d13h
web-79d88c97d6-9tzlh   1/1     Running   0          2d13h
web-79d88c97d6-brtgx   1/1     Running   0          2d13h
web-79d88c97d6-kngc4   1/1     Running   0          2d13h
web-79d88c97d6-p5vfg   1/1     Running   0          2d13h
web-79d88c97d6-rbhpr   1/1     Running   0          2d13h
lab@k8s1:~$
```

