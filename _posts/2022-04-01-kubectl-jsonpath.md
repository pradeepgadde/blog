---
layout: single
title:  "Kubectl JsonPath"
date:   2022-04-01 10:55:04 +0530
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
# Kubectl JsonPath

Let us display the result of `kubectl get nodes` in the `JSON` format.

First, in plain text



```sh
lab@k8s1:~$ kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
k8s1   Ready    control-plane,master   2d12h   v1.22.8
k8s2   Ready    <none>                 2d11h   v1.22.8
k8s3   Ready    <none>                 2d9h    v1.22.8
lab@k8s1:~$
```



```json
lab@k8s1:~$ kubectl get nodes -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "flannel.alpha.coreos.com/backend-data": "{\"VNI\":1,\"VtepMAC\":\"52:1d:12:10:50:8f\"}",
                    "flannel.alpha.coreos.com/backend-type": "vxlan",
                    "flannel.alpha.coreos.com/kube-subnet-manager": "true",
                    "flannel.alpha.coreos.com/public-ip": "10.210.40.172",
                    "kubeadm.alpha.kubernetes.io/cri-socket": "/var/run/dockershim.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2022-03-29T14:16:22Z",
                "labels": {
                    "beta.kubernetes.io/arch": "amd64",
                    "beta.kubernetes.io/os": "linux",
                    "kubernetes.io/arch": "amd64",
                    "kubernetes.io/hostname": "k8s1",
                    "kubernetes.io/os": "linux",
                    "node-role.kubernetes.io/control-plane": "",
                    "node-role.kubernetes.io/master": "",
                    "node.kubernetes.io/exclude-from-external-load-balancers": ""
                },
                "name": "k8s1",
                "resourceVersion": "221514",
                "uid": "97295130-f079-420c-a18a-1782471530ea"
            },
            "spec": {
                "podCIDR": "10.244.0.0/24",
                "podCIDRs": [
                    "10.244.0.0/24"
                ],
                "taints": [
                    {
                        "effect": "NoSchedule",
                        "key": "node-role.kubernetes.io/master"
                    }
                ]
            },
            "status": {
                "addresses": [
                    {
                        "address": "10.210.40.172",
                        "type": "InternalIP"
                    },
                    {
                        "address": "k8s1",
                        "type": "Hostname"
                    }
                ],
                "allocatable": {
                    "cpu": "2",
                    "ephemeral-storage": "59278082360",
                    "hugepages-2Mi": "0",
                    "memory": "1922804Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "2",
                    "ephemeral-storage": "64320836Ki",
                    "hugepages-2Mi": "0",
                    "memory": "2025204Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2022-03-29T14:20:58Z",
                        "lastTransitionTime": "2022-03-29T14:20:58Z",
                        "message": "Flannel is running on this node",
                        "reason": "FlannelIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T14:16:19Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T14:16:19Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T14:16:19Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T18:14:07Z",
                        "message": "kubelet is posting ready status. AppArmor enabled",
                        "reason": "KubeletReady",
                        "status": "True",
                        "type": "Ready"
                    }
                ],
                "daemonEndpoints": {
                    "kubeletEndpoint": {
                        "Port": 10250
                    }
                },
                "images": [
                    {
                        "names": [
                            "k8s.gcr.io/etcd@sha256:9ce33ba33d8e738a5b85ed50b5080ac746deceed4a7496c550927a7a19ca3b6d",
                            "k8s.gcr.io/etcd:3.5.0-0"
                        ],
                        "sizeBytes": 294536887
                    },
                    {
                        "names": [
                            "k8s.gcr.io/kube-apiserver@sha256:c2235616f1fbb21e13876cffb72d94241c560b1829fc3820d9a9e5ffb2cfa8e8",
                            "k8s.gcr.io/kube-apiserver:v1.22.8"
                        ],
                        "sizeBytes": 128364093
                    },
                    {
                        "names": [
                            "k8s.gcr.io/kube-controller-manager@sha256:7be1045265f305ce7c827af613411986eb1c437ed370939800f29ed6d6b1c941",
                            "k8s.gcr.io/kube-controller-manager:v1.22.8"
                        ],
                        "sizeBytes": 122052768
                    },
                    {
                        "names": [
                            "k8s.gcr.io/kube-proxy@sha256:46c852ee61a7ea0cdc020ccb46028a6783336548074f86ab22e26945ffe31f98",
                            "k8s.gcr.io/kube-proxy:v1.22.8"
                        ],
                        "sizeBytes": 103669604
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel@sha256:4bf659e449be809763b04f894f53a3d8610e00cf2cd979bb4fffc9470eb40d1b",
                            "rancher/mirrored-flannelcni-flannel:v0.17.0"
                        ],
                        "sizeBytes": 59803434
                    },
                    {
                        "names": [
                            "k8s.gcr.io/kube-scheduler@sha256:ae27af87fe15a0cb36c10409eef35363d68243ab3c6e888fc9adf7a0ed858f1e",
                            "k8s.gcr.io/kube-scheduler:v1.22.8"
                        ],
                        "sizeBytes": 52707184
                    },
                    {
                        "names": [
                            "k8s.gcr.io/coredns/coredns@sha256:6e5a02c21641597998b4be7cb5eb1e7b02c0d8d23cce4dd09f4682d463798890",
                            "k8s.gcr.io/coredns/coredns:v1.8.4"
                        ],
                        "sizeBytes": 47554275
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel-cni-plugin@sha256:5dd61f95e28fa7ef897ff2fa402ce283e5078d334401d2f62d00a568f779f2d5",
                            "rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1"
                        ],
                        "sizeBytes": 8098691
                    },
                    {
                        "names": [
                            "k8s.gcr.io/pause@sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07",
                            "k8s.gcr.io/pause:3.5"
                        ],
                        "sizeBytes": 682696
                    },
                    {
                        "names": [
                            "hello-world@sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38",
                            "hello-world:latest"
                        ],
                        "sizeBytes": 13256
                    }
                ],
                "nodeInfo": {
                    "architecture": "amd64",
                    "bootID": "79681989-c2a6-478c-bb51-7c46b1596fba",
                    "containerRuntimeVersion": "docker://20.10.14",
                    "kernelVersion": "5.13.0-37-generic",
                    "kubeProxyVersion": "v1.22.8",
                    "kubeletVersion": "v1.22.8",
                    "machineID": "596c7f47034440028cd05f4d0fa9c753",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.4 LTS",
                    "systemUUID": "18962942-410c-c225-65ca-e0a7de38d7cd"
                }
            }
        },
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "flannel.alpha.coreos.com/backend-data": "{\"VNI\":1,\"VtepMAC\":\"92:01:5e:01:fe:01\"}",
                    "flannel.alpha.coreos.com/backend-type": "vxlan",
                    "flannel.alpha.coreos.com/kube-subnet-manager": "true",
                    "flannel.alpha.coreos.com/public-ip": "10.210.40.173",
                    "kubeadm.alpha.kubernetes.io/cri-socket": "/var/run/dockershim.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2022-03-29T15:29:34Z",
                "labels": {
                    "beta.kubernetes.io/arch": "amd64",
                    "beta.kubernetes.io/os": "linux",
                    "kubernetes.io/arch": "amd64",
                    "kubernetes.io/hostname": "k8s2",
                    "kubernetes.io/os": "linux"
                },
                "name": "k8s2",
                "resourceVersion": "221513",
                "uid": "f5c6831f-f3a5-4a67-b472-5f1dc70f017e"
            },
            "spec": {
                "podCIDR": "10.244.2.0/24",
                "podCIDRs": [
                    "10.244.2.0/24"
                ]
            },
            "status": {
                "addresses": [
                    {
                        "address": "10.210.40.173",
                        "type": "InternalIP"
                    },
                    {
                        "address": "k8s2",
                        "type": "Hostname"
                    }
                ],
                "allocatable": {
                    "cpu": "2",
                    "ephemeral-storage": "59278082360",
                    "hugepages-2Mi": "0",
                    "memory": "1922804Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "2",
                    "ephemeral-storage": "64320836Ki",
                    "hugepages-2Mi": "0",
                    "memory": "2025204Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2022-03-29T15:29:40Z",
                        "lastTransitionTime": "2022-03-29T15:29:40Z",
                        "message": "Flannel is running on this node",
                        "reason": "FlannelIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T15:29:34Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T15:29:34Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T15:29:34Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T18:22:47Z",
                        "message": "kubelet is posting ready status. AppArmor enabled",
                        "reason": "KubeletReady",
                        "status": "True",
                        "type": "Ready"
                    }
                ],
                "daemonEndpoints": {
                    "kubeletEndpoint": {
                        "Port": 10250
                    }
                },
                "images": [
                    {
                        "names": [
                            "k8s.gcr.io/kube-proxy@sha256:46c852ee61a7ea0cdc020ccb46028a6783336548074f86ab22e26945ffe31f98",
                            "k8s.gcr.io/kube-proxy:v1.22.8"
                        ],
                        "sizeBytes": 103669604
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel@sha256:4bf659e449be809763b04f894f53a3d8610e00cf2cd979bb4fffc9470eb40d1b",
                            "rancher/mirrored-flannelcni-flannel:v0.17.0"
                        ],
                        "sizeBytes": 59803434
                    },
                    {
                        "names": [
                            "k8s.gcr.io/coredns/coredns@sha256:6e5a02c21641597998b4be7cb5eb1e7b02c0d8d23cce4dd09f4682d463798890",
                            "k8s.gcr.io/coredns/coredns:v1.8.4"
                        ],
                        "sizeBytes": 47554275
                    },
                    {
                        "names": [
                            "gcr.io/google-samples/hello-app@sha256:88b205d7995332e10e836514fbfd59ecaf8976fc15060cd66e85cdcebe7fb356",
                            "gcr.io/google-samples/hello-app:1.0"
                        ],
                        "sizeBytes": 11480274
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel-cni-plugin@sha256:5dd61f95e28fa7ef897ff2fa402ce283e5078d334401d2f62d00a568f779f2d5",
                            "rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1"
                        ],
                        "sizeBytes": 8098691
                    },
                    {
                        "names": [
                            "k8s.gcr.io/pause@sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07",
                            "k8s.gcr.io/pause:3.5"
                        ],
                        "sizeBytes": 682696
                    },
                    {
                        "names": [
                            "hello-world@sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38",
                            "hello-world:latest"
                        ],
                        "sizeBytes": 13256
                    }
                ],
                "nodeInfo": {
                    "architecture": "amd64",
                    "bootID": "ed911925-bdd7-4d98-aa4e-eaa3a689bad2",
                    "containerRuntimeVersion": "docker://20.10.14",
                    "kernelVersion": "5.13.0-37-generic",
                    "kubeProxyVersion": "v1.22.8",
                    "kubeletVersion": "v1.22.8",
                    "machineID": "596c7f47034440028cd05f4d0fa9c753",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.4 LTS",
                    "systemUUID": "02cd2942-ef22-71c9-90d0-54187982487f"
                }
            }
        },
        {
            "apiVersion": "v1",
            "kind": "Node",
            "metadata": {
                "annotations": {
                    "flannel.alpha.coreos.com/backend-data": "{\"VNI\":1,\"VtepMAC\":\"46:57:f7:c6:05:51\"}",
                    "flannel.alpha.coreos.com/backend-type": "vxlan",
                    "flannel.alpha.coreos.com/kube-subnet-manager": "true",
                    "flannel.alpha.coreos.com/public-ip": "10.210.40.175",
                    "kubeadm.alpha.kubernetes.io/cri-socket": "/var/run/dockershim.sock",
                    "node.alpha.kubernetes.io/ttl": "0",
                    "volumes.kubernetes.io/controller-managed-attach-detach": "true"
                },
                "creationTimestamp": "2022-03-29T17:00:06Z",
                "labels": {
                    "beta.kubernetes.io/arch": "amd64",
                    "beta.kubernetes.io/os": "linux",
                    "kubernetes.io/arch": "amd64",
                    "kubernetes.io/hostname": "k8s3",
                    "kubernetes.io/os": "linux"
                },
                "name": "k8s3",
                "resourceVersion": "221512",
                "uid": "9e27cee3-6769-4bbc-bfd3-d3d5d2067ab7"
            },
            "spec": {
                "podCIDR": "10.244.3.0/24",
                "podCIDRs": [
                    "10.244.3.0/24"
                ]
            },
            "status": {
                "addresses": [
                    {
                        "address": "10.210.40.175",
                        "type": "InternalIP"
                    },
                    {
                        "address": "k8s3",
                        "type": "Hostname"
                    }
                ],
                "allocatable": {
                    "cpu": "2",
                    "ephemeral-storage": "59278082360",
                    "hugepages-2Mi": "0",
                    "memory": "1922804Ki",
                    "pods": "110"
                },
                "capacity": {
                    "cpu": "2",
                    "ephemeral-storage": "64320836Ki",
                    "hugepages-2Mi": "0",
                    "memory": "2025204Ki",
                    "pods": "110"
                },
                "conditions": [
                    {
                        "lastHeartbeatTime": "2022-03-29T17:00:29Z",
                        "lastTransitionTime": "2022-03-29T17:00:29Z",
                        "message": "Flannel is running on this node",
                        "reason": "FlannelIsUp",
                        "status": "False",
                        "type": "NetworkUnavailable"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T17:00:06Z",
                        "message": "kubelet has sufficient memory available",
                        "reason": "KubeletHasSufficientMemory",
                        "status": "False",
                        "type": "MemoryPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T17:00:06Z",
                        "message": "kubelet has no disk pressure",
                        "reason": "KubeletHasNoDiskPressure",
                        "status": "False",
                        "type": "DiskPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T17:00:06Z",
                        "message": "kubelet has sufficient PID available",
                        "reason": "KubeletHasSufficientPID",
                        "status": "False",
                        "type": "PIDPressure"
                    },
                    {
                        "lastHeartbeatTime": "2022-04-01T02:29:11Z",
                        "lastTransitionTime": "2022-03-29T18:32:13Z",
                        "message": "kubelet is posting ready status. AppArmor enabled",
                        "reason": "KubeletReady",
                        "status": "True",
                        "type": "Ready"
                    }
                ],
                "daemonEndpoints": {
                    "kubeletEndpoint": {
                        "Port": 10250
                    }
                },
                "images": [
                    {
                        "names": [
                            "k8s.gcr.io/kube-proxy@sha256:46c852ee61a7ea0cdc020ccb46028a6783336548074f86ab22e26945ffe31f98",
                            "k8s.gcr.io/kube-proxy:v1.22.8"
                        ],
                        "sizeBytes": 103669604
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel@sha256:4bf659e449be809763b04f894f53a3d8610e00cf2cd979bb4fffc9470eb40d1b",
                            "rancher/mirrored-flannelcni-flannel:v0.17.0"
                        ],
                        "sizeBytes": 59803434
                    },
                    {
                        "names": [
                            "k8s.gcr.io/coredns/coredns@sha256:6e5a02c21641597998b4be7cb5eb1e7b02c0d8d23cce4dd09f4682d463798890",
                            "k8s.gcr.io/coredns/coredns:v1.8.4"
                        ],
                        "sizeBytes": 47554275
                    },
                    {
                        "names": [
                            "gcr.io/google-samples/hello-app@sha256:88b205d7995332e10e836514fbfd59ecaf8976fc15060cd66e85cdcebe7fb356",
                            "gcr.io/google-samples/hello-app:1.0"
                        ],
                        "sizeBytes": 11480274
                    },
                    {
                        "names": [
                            "rancher/mirrored-flannelcni-flannel-cni-plugin@sha256:5dd61f95e28fa7ef897ff2fa402ce283e5078d334401d2f62d00a568f779f2d5",
                            "rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1"
                        ],
                        "sizeBytes": 8098691
                    },
                    {
                        "names": [
                            "k8s.gcr.io/pause@sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07",
                            "k8s.gcr.io/pause:3.5"
                        ],
                        "sizeBytes": 682696
                    },
                    {
                        "names": [
                            "hello-world@sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38",
                            "hello-world:latest"
                        ],
                        "sizeBytes": 13256
                    }
                ],
                "nodeInfo": {
                    "architecture": "amd64",
                    "bootID": "3c77369e-54ee-471e-979c-7c730ee356d8",
                    "containerRuntimeVersion": "docker://20.10.14",
                    "kernelVersion": "5.13.0-37-generic",
                    "kubeProxyVersion": "v1.22.8",
                    "kubeletVersion": "v1.22.8",
                    "machineID": "596c7f47034440028cd05f4d0fa9c753",
                    "operatingSystem": "linux",
                    "osImage": "Ubuntu 20.04.4 LTS",
                    "systemUUID": "e6ee2942-fc1b-488a-6add-25e5744a15ee"
                }
            }
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": "",
        "selfLink": ""
    }
}
lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get nodes -o jsonpath="{$.items[*].metadata.name}"
k8s1 k8s2 k8s3lab@k8s1:~$
```



```sh
lab@k8s1:~$ kubectl get nodes -o jsonpath="{$.items[0].metadata.name}"
k8s1lab@k8s1:~$ kubectl get nodes -o jsonpath="{$.items[1].metadata.name}"
k8s2lab@k8s1:~$ kubectl get nodes -o jsonpath="{$.items[2].metadata.name}"
k8s3lab@k8s1:~$
```



Another example

```sh
lab@k8s1:~$ kubectl get nodes -o jsonpath="{$.items[*].spec.podCIDR}"
10.244.0.0/24 10.244.2.0/24 10.244.3.0/24lab@k8s1:~$
```

Let us format it a bit, so that we get each of them in a new line,



```sh
lab@k8s1:~$ kubectl get nodes -o jsonpath='{range.items[*]} {.spec.podCIDR} {"\n"}'
 10.244.0.0/24
 10.244.2.0/24
 10.244.3.0/24
 lab@k8s1:~$
```



Looping through all items showing multiple columns


```sh
lab@k8s1:~$ kubectl get nodes -o jsonpath'{range.items[*]} {.metadata.name} {"\t"} {.status.capacity.cpu} {"\n"} {end}'
 k8s1 	 2
  k8s2 	 2
  k8s3 	 2
```



## Custom-Columns

Make use of the `-o=custom-columns` option to change the default output of kubectl commands.

Here is a simple example

```sh
b@k8s1:~$ kubectl get nodes -o=custom-columns=PodCIDR:.spec.podCIDR
PodCIDR
10.244.0.0/24
10.244.2.0/24
10.244.3.0/24
```

Another one, with two columns


```sh
lab@k8s1:~$ kubectl get nodes -o=custom-columns=PodCIDR:.spec.podCIDR,CPU:.status.capacity.cpu
PodCIDR         CPU
10.244.0.0/24   2
10.244.2.0/24   2
10.244.3.0/24   2
```

One more,
```sh
lab@k8s1:~$ kubectl get nodes -o=custom-columns=PodCIDR:.spec.podCIDR,CPU:.status.capacity.cpu,NAME:.metadata.name
PodCIDR         CPU   NAME
10.244.0.0/24   2     k8s1
10.244.2.0/24   2     k8s2
10.244.3.0/24   2     k8s3
lab@k8s1:~$
```



> One thing to note, is that , with multiple columns, ensure that there is no extra space after each column followed by a comma `(,)`.

```sh
lab@k8s1:~$ kubectl get nodes -o=custom-columns=PodCIDR:.spec.podCIDR, CPU:.status.capacity.cpu,NAME:.metadata.name
Error from server (NotFound): nodes "CPU:.status.capacity.cpu,NAME:.metadata.name" not found
lab@k8s1:~$
```



We have an extra space before CPU column name, hence the error.



Let us conclude with one more example, by modifying the `kubectl get pods` output, by just showing the name of the Pod and namespace.

```sh
lab@k8s1:~$ kubectl get pods -o=custom-columns=NAME:.metadata.name,NS:.metadata.namespace
NAME                   NS
web-79d88c97d6-4w4nl   default
web-79d88c97d6-7ktck   default
web-79d88c97d6-8tvsk   default
web-79d88c97d6-96sd9   default
web-79d88c97d6-9tzlh   default
web-79d88c97d6-brtgx   default
web-79d88c97d6-kngc4   default
web-79d88c97d6-p5vfg   default
web-79d88c97d6-rbhpr   default
lab@k8s1:~$
```







