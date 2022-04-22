---
layout: single
title:  "Kubectl Proxy"
date:   2022-04-10 10:58:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
show_date: true
classes: wide
header:
  teaser: /assets/images/k-proxy-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---



# Kubectl Proxy

We can use an HTTP Proxy to access the Kubernetes API with the `kubectl proxy --port=8080` command. This  starts a proxy to the Kubernetes API server and we can explore the API using `curl`, `wget`, or a browser.



```sh
pradeep@learnk8s$ kubectl proxy -h
Creates a proxy server or application-level gateway between localhost and the
Kubernetes API server. It also allows serving static content over specified HTTP
path. All incoming data enters through one port and gets forwarded to the remote
Kubernetes API server port, except for the path matching the static content
path.

Examples:
  # To proxy all of the Kubernetes API and nothing else
  kubectl proxy --api-prefix=/
  
  # To proxy only part of the Kubernetes API and also some static files
  # You can get pods info with 'curl localhost:8001/api/v1/pods'
  kubectl proxy --www=/my/files --www-prefix=/static/ --api-prefix=/api/
  
  # To proxy the entire Kubernetes API at a different root
  # You can get pods info with 'curl localhost:8001/custom/api/v1/pods'
  kubectl proxy --api-prefix=/custom/
  
  # Run a proxy to the Kubernetes API server on port 8011, serving static
content from ./local/www/
  kubectl proxy --port=8011 --www=./local/www/
  
  # Run a proxy to the Kubernetes API server on an arbitrary local port
  # The chosen port for the server will be output to stdout
  kubectl proxy --port=0
  
  # Run a proxy to the Kubernetes API server, changing the API prefix to k8s-api
  # This makes e.g. the pods API available at localhost:8001/k8s-api/v1/pods/
  kubectl proxy --api-prefix=/k8s-api

Options:
      --accept-hosts='^localhost$,^127\.0\.0\.1$,^\[::1\]$': Regular expression
for hosts that the proxy should accept.
      --accept-paths='^.*': Regular expression for paths that the proxy should
accept.
      --address='127.0.0.1': The IP address on which to serve on.
      --api-prefix='/': Prefix to serve the proxied API under.
      --disable-filter=false: If true, disable request filtering in the proxy.
This is dangerous, and can leave you vulnerable to XSRF attacks, when used with
an accessible port.
      --keepalive=0s: keepalive specifies the keep-alive period for an active
network connection. Set to 0 to disable keepalive.
  -p, --port=8001: The port on which to run the proxy. Set to 0 to pick a random
port.
      --reject-methods='^$': Regular expression for HTTP methods that the proxy
should reject (example --reject-methods='POST,PUT,PATCH'). 
      --reject-paths='^/api/.*/pods/.*/exec,^/api/.*/pods/.*/attach': Regular
expression for paths that the proxy should reject. Paths specified here will be
rejected even accepted by --accept-paths.
  -u, --unix-socket='': Unix socket on which to run the proxy.
  -w, --www='': Also serve static files from the given directory under the
specified prefix.
  -P, --www-prefix='/static/': Prefix to serve static files under, if static
file directory is specified.

Usage:
  kubectl proxy [--port=PORT] [--www=static-dir] [--www-prefix=prefix]
[--api-prefix=prefix] [options]

Use "kubectl options" for a list of global command-line options (applies to all
commands).
pradeep@learnk8s$ 
```



For example, to get the API versions:

```
curl http://localhost:8080/api/
```

```sh
pradeep@learnk8s$ curl http://localhost:8080/api
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "172.16.30.6:8443"
    }
  ]
}
pradeep@learnk8s$
```



To get all namespaces

```sh
pradeep@learnk8s$ curl http://localhost:8080/api/v1/namespaces
{
  "kind": "NamespaceList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "206974"
  },
  "items": [
    {
      "metadata": {
        "name": "default",
        "uid": "01063dc0-349e-4817-94d7-12280522af20",
        "resourceVersion": "207",
        "creationTimestamp": "2022-03-19T18:18:40Z",
        "labels": {
          "kubernetes.io/metadata.name": "default"
        },
        "managedFields": [
          {
            "manager": "kube-apiserver",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-03-19T18:18:40Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "ingress-nginx",
        "uid": "5b548454-88a5-4d82-9201-29a2d8c6a393",
        "resourceVersion": "12203",
        "creationTimestamp": "2022-03-21T15:41:50Z",
        "labels": {
          "app.kubernetes.io/instance": "ingress-nginx",
          "app.kubernetes.io/name": "ingress-nginx",
          "kubernetes.io/metadata.name": "ingress-nginx"
        },
        "annotations": {
          "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Namespace\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/instance\":\"ingress-nginx\",\"app.kubernetes.io/name\":\"ingress-nginx\"},\"name\":\"ingress-nginx\"}}\n"
        },
        "managedFields": [
          {
            "manager": "kubectl-client-side-apply",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-03-21T15:41:50Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:kubectl.kubernetes.io/last-applied-configuration": {}
                },
                "f:labels": {
                  ".": {},
                  "f:app.kubernetes.io/instance": {},
                  "f:app.kubernetes.io/name": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "kube-node-lease",
        "uid": "0b65dc54-266e-46a8-9170-08b2b8fd0a70",
        "resourceVersion": "50",
        "creationTimestamp": "2022-03-19T18:18:36Z",
        "labels": {
          "kubernetes.io/metadata.name": "kube-node-lease"
        },
        "managedFields": [
          {
            "manager": "kube-apiserver",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-03-19T18:18:36Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "kube-public",
        "uid": "c5c1e001-9c14-468a-8636-e789ce2de8f8",
        "resourceVersion": "46",
        "creationTimestamp": "2022-03-19T18:18:36Z",
        "labels": {
          "kubernetes.io/metadata.name": "kube-public"
        },
        "managedFields": [
          {
            "manager": "kube-apiserver",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-03-19T18:18:36Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "kube-system",
        "uid": "f507ee30-54cc-485c-b8b1-2a0d3e7188c9",
        "resourceVersion": "28",
        "creationTimestamp": "2022-03-19T18:18:36Z",
        "labels": {
          "kubernetes.io/metadata.name": "kube-system"
        },
        "managedFields": [
          {
            "manager": "Go-http-client",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-03-19T18:18:36Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "pacman",
        "uid": "3ab50fe9-3065-489a-8c27-ffaa322b2e16",
        "resourceVersion": "185039",
        "creationTimestamp": "2022-04-08T18:14:39Z",
        "labels": {
          "kubernetes.io/metadata.name": "pacman"
        },
        "managedFields": [
          {
            "manager": "kubectl-create",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T18:14:39Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    },
    {
      "metadata": {
        "name": "wordsmith",
        "uid": "2abb71b7-4e2f-422d-a42d-5c770570bb54",
        "resourceVersion": "182875",
        "creationTimestamp": "2022-04-08T17:42:40Z",
        "labels": {
          "kubernetes.io/metadata.name": "wordsmith"
        },
        "managedFields": [
          {
            "manager": "kubectl-create",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T17:42:40Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:labels": {
                  ".": {},
                  "f:kubernetes.io/metadata.name": {}
                }
              }
            }
          }
        ]
      },
      "spec": {
        "finalizers": [
          "kubernetes"
        ]
      },
      "status": {
        "phase": "Active"
      }
    }
  ]
}
pradeep@learnk8s$
```



To get the list of pods in default namespace

```sh
pradeep@Pradeeps-MacBook-Air ~ % curl http://localhost:8080/api/v1/namespaces/default/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "206989"
  },
  "items": [
    {
      "metadata": {
        "name": "web-746c8679d4-kn2md",
        "generateName": "web-746c8679d4-",
        "namespace": "default",
        "uid": "69190787-0e0c-4e6d-8bb2-3a5f3a7c36e7",
        "resourceVersion": "204950",
        "creationTimestamp": "2022-04-08T17:42:01Z",
        "labels": {
          "app": "web",
          "pod-template-hash": "746c8679d4"
        },
        "annotations": {
          "cni.projectcalico.org/containerID": "b89679b04d13deea49118d2d6bb2582472bf00347fc627d5ccaaabc0efec328d",
          "cni.projectcalico.org/podIP": "10.244.205.200/32",
          "cni.projectcalico.org/podIPs": "10.244.205.200/32"
        },
        "ownerReferences": [
          {
            "apiVersion": "apps/v1",
            "kind": "ReplicaSet",
            "name": "web-746c8679d4",
            "uid": "696de3be-9e73-4d15-9d74-3e072f8eb751",
            "controller": true,
            "blockOwnerDeletion": true
          }
        ],
        "managedFields": [
          {
            "manager": "kube-controller-manager",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T17:42:01Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:generateName": {},
                "f:labels": {
                  ".": {},
                  "f:app": {},
                  "f:pod-template-hash": {}
                },
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"696de3be-9e73-4d15-9d74-3e072f8eb751\"}": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"hello-app\"}": {
                    ".": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "calico",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T17:42:07Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:cni.projectcalico.org/containerID": {},
                  "f:cni.projectcalico.org/podIP": {},
                  "f:cni.projectcalico.org/podIPs": {}
                }
              }
            },
            "subresource": "status"
          },
          {
            "manager": "Go-http-client",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-09T11:19:32Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:phase": {},
                "f:podIP": {},
                "f:podIPs": {
                  ".": {},
                  "k:{\"ip\":\"10.244.205.200\"}": {
                    ".": {},
                    "f:ip": {}
                  }
                },
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-lskkv",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "hello-app",
            "image": "gcr.io/google-samples/hello-app:1.0",
            "resources": {},
            "volumeMounts": [
              {
                "name": "kube-api-access-lskkv",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "IfNotPresent"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube-m02",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:01Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:07Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:07Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:01Z"
          }
        ],
        "hostIP": "172.16.30.7",
        "podIP": "10.244.205.200",
        "podIPs": [
          {
            "ip": "10.244.205.200"
          }
        ],
        "startTime": "2022-04-08T17:42:01Z",
        "containerStatuses": [
          {
            "name": "hello-app",
            "state": {
              "running": {
                "startedAt": "2022-04-08T17:42:07Z"
              }
            },
            "lastState": {},
            "ready": true,
            "restartCount": 0,
            "image": "gcr.io/google-samples/hello-app:1.0",
            "imageID": "docker-pullable://gcr.io/google-samples/hello-app@sha256:88b205d7995332e10e836514fbfd59ecaf8976fc15060cd66e85cdcebe7fb356",
            "containerID": "docker://54275596466a535f9ffab8316725315dbf3ea33aa8e29a413ee38e1119750752",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    },
    {
      "metadata": {
        "name": "web2-5858b4c7c5-9lzg9",
        "generateName": "web2-5858b4c7c5-",
        "namespace": "default",
        "uid": "a4a604ff-bd77-4bf0-8a92-5aa69a9951cb",
        "resourceVersion": "205003",
        "creationTimestamp": "2022-04-08T17:42:01Z",
        "labels": {
          "app": "web2",
          "pod-template-hash": "5858b4c7c5"
        },
        "annotations": {
          "cni.projectcalico.org/containerID": "37acec19f53bf37cdd5eaa2a2e67e3809d79545ba32d6e555c0ff010a9ce9f80",
          "cni.projectcalico.org/podIP": "10.244.205.199/32",
          "cni.projectcalico.org/podIPs": "10.244.205.199/32"
        },
        "ownerReferences": [
          {
            "apiVersion": "apps/v1",
            "kind": "ReplicaSet",
            "name": "web2-5858b4c7c5",
            "uid": "caf4fc40-6681-4312-9cce-9fc1644c71c8",
            "controller": true,
            "blockOwnerDeletion": true
          }
        ],
        "managedFields": [
          {
            "manager": "kube-controller-manager",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T17:42:01Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:generateName": {},
                "f:labels": {
                  ".": {},
                  "f:app": {},
                  "f:pod-template-hash": {}
                },
                "f:ownerReferences": {
                  ".": {},
                  "k:{\"uid\":\"caf4fc40-6681-4312-9cce-9fc1644c71c8\"}": {}
                }
              },
              "f:spec": {
                "f:containers": {
                  "k:{\"name\":\"hello-app\"}": {
                    ".": {},
                    "f:image": {},
                    "f:imagePullPolicy": {},
                    "f:name": {},
                    "f:resources": {},
                    "f:terminationMessagePath": {},
                    "f:terminationMessagePolicy": {}
                  }
                },
                "f:dnsPolicy": {},
                "f:enableServiceLinks": {},
                "f:restartPolicy": {},
                "f:schedulerName": {},
                "f:securityContext": {},
                "f:terminationGracePeriodSeconds": {}
              }
            }
          },
          {
            "manager": "calico",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-08T17:42:06Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:metadata": {
                "f:annotations": {
                  ".": {},
                  "f:cni.projectcalico.org/containerID": {},
                  "f:cni.projectcalico.org/podIP": {},
                  "f:cni.projectcalico.org/podIPs": {}
                }
              }
            },
            "subresource": "status"
          },
          {
            "manager": "Go-http-client",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-04-09T11:19:40Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:status": {
                "f:conditions": {
                  "k:{\"type\":\"ContainersReady\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Initialized\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  },
                  "k:{\"type\":\"Ready\"}": {
                    ".": {},
                    "f:lastProbeTime": {},
                    "f:lastTransitionTime": {},
                    "f:status": {},
                    "f:type": {}
                  }
                },
                "f:containerStatuses": {},
                "f:hostIP": {},
                "f:phase": {},
                "f:podIP": {},
                "f:podIPs": {
                  ".": {},
                  "k:{\"ip\":\"10.244.205.199\"}": {
                    ".": {},
                    "f:ip": {}
                  }
                },
                "f:startTime": {}
              }
            },
            "subresource": "status"
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "kube-api-access-s95gm",
            "projected": {
              "sources": [
                {
                  "serviceAccountToken": {
                    "expirationSeconds": 3607,
                    "path": "token"
                  }
                },
                {
                  "configMap": {
                    "name": "kube-root-ca.crt",
                    "items": [
                      {
                        "key": "ca.crt",
                        "path": "ca.crt"
                      }
                    ]
                  }
                },
                {
                  "downwardAPI": {
                    "items": [
                      {
                        "path": "namespace",
                        "fieldRef": {
                          "apiVersion": "v1",
                          "fieldPath": "metadata.namespace"
                        }
                      }
                    ]
                  }
                }
              ],
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "hello-app",
            "image": "gcr.io/google-samples/hello-app:2.0",
            "resources": {},
            "volumeMounts": [
              {
                "name": "kube-api-access-s95gm",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "IfNotPresent"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "minikube-m02",
        "securityContext": {},
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:01Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:12Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:12Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-04-08T17:42:01Z"
          }
        ],
        "hostIP": "172.16.30.7",
        "podIP": "10.244.205.199",
        "podIPs": [
          {
            "ip": "10.244.205.199"
          }
        ],
        "startTime": "2022-04-08T17:42:01Z",
        "containerStatuses": [
          {
            "name": "hello-app",
            "state": {
              "running": {
                "startedAt": "2022-04-08T17:42:11Z"
              }
            },
            "lastState": {},
            "ready": true,
            "restartCount": 0,
            "image": "gcr.io/google-samples/hello-app:2.0",
            "imageID": "docker-pullable://gcr.io/google-samples/hello-app@sha256:2b0febe1b9bd01739999853380b1a939e8102fd0dc5e2ff1fc6892c4557d52b9",
            "containerID": "docker://5290fb73214777390900501a7fc6df2136b2243a77645d81b1bcc759d85944c0",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    }
  ]
} 
pradeep@learnk8s$

```

