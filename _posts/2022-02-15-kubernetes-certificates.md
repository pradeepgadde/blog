---
layout: single
title:  "Kubernetes Certificates"
date:   2022-02-15 10:55:04 +0530
categories: Kubernetes
tags: minikube
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
# Kubernetes Certificates


## Security

###  Certificates

Take a look at the description of the `kube-apiserver` pod in the `kube-system` namespace. Pay attention to the `kube-apiserver` command arguments. It is a long list!!

Primarily, there are many pairs of certfile and keyfile for apiserver, etcd, kubectl etc. In Minikube setup, most of these are located in `/var/lib/minikube/certs/`. It would be different when the cluster is setup with `kubeadm` which we have not discussed yet.

```shell
pradeep@learnk8s$ kubectl describe -n kube-system pods kube-apiserver-k8s
Name:                 kube-apiserver-k8s
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 k8s/192.168.177.29
Start Time:           Tue, 15 Feb 2022 12:28:03 +0530
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.177.29:8443
                      kubernetes.io/config.hash: cca804e910f3a6e748c66a6963d63fdd
                      kubernetes.io/config.mirror: cca804e910f3a6e748c66a6963d63fdd
                      kubernetes.io/config.seen: 2022-02-15T06:58:02.427574419Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   192.168.177.29
IPs:
  IP:           192.168.177.29
Controlled By:  Node/k8s
Containers:
  kube-apiserver:
    Container ID:  docker://0ec9bc91aa13e593b1518fac7a4f9f39c7e16a0e478c2362336b8c050f9c085c
    Image:         k8s.gcr.io/kube-apiserver:v1.23.1
    Image ID:      docker-pullable://k8s.gcr.io/kube-apiserver@sha256:f54681a71cce62cbc1b13ebb3dbf1d880f849112789811f98b6aebd2caa2f255
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=192.168.177.29
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/var/lib/minikube/certs/ca.crt
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/var/lib/minikube/certs/etcd/ca.crt
      --etcd-certfile=/var/lib/minikube/certs/apiserver-etcd-client.crt
      --etcd-keyfile=/var/lib/minikube/certs/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --kubelet-client-certificate=/var/lib/minikube/certs/apiserver-kubelet-client.crt
      --kubelet-client-key=/var/lib/minikube/certs/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/var/lib/minikube/certs/front-proxy-client.crt
      --proxy-client-key-file=/var/lib/minikube/certs/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/var/lib/minikube/certs/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=8443
      --service-account-issuer=https://kubernetes.default.svc.cluster.local
      --service-account-key-file=/var/lib/minikube/certs/sa.pub
      --service-account-signing-key-file=/var/lib/minikube/certs/sa.key
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/var/lib/minikube/certs/apiserver.crt
      --tls-private-key-file=/var/lib/minikube/certs/apiserver.key
    State:          Running
      Started:      Tue, 15 Feb 2022 12:27:50 +0530
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:        250m
    Liveness:     http-get https://192.168.177.29:8443/livez delay=10s timeout=15s period=10s #success=1 #failure=8
    Readiness:    http-get https://192.168.177.29:8443/readyz delay=0s timeout=15s period=1s #success=1 #failure=3
    Startup:      http-get https://192.168.177.29:8443/livez delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/ssl/certs from ca-certs (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
      /var/lib/minikube/certs from k8s-certs (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/certs
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason     Age                  From     Message
  ----     ------     ----                 ----     -------
  Warning  Unhealthy  178m (x5 over 30h)   kubelet  Liveness probe failed: Get "https://192.168.177.29:8443/livez": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  177m (x17 over 30h)  kubelet  Readiness probe failed: Get "https://192.168.177.29:8443/readyz": net/http: TLS handshake timeout
  Warning  Unhealthy  170m (x7 over 27h)   kubelet  Readiness probe failed: Get "https://192.168.177.29:8443/readyz": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  170m (x6 over 27h)   kubelet  Liveness probe failed: Get "https://192.168.177.29:8443/livez": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  Unhealthy  170m (x8 over 30h)   kubelet  Readiness probe failed: Get "https://192.168.177.29:8443/readyz": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```
We can login to the minikube node and use the `openssl x509` command to view the actual certificate. Here is an example. Pass the certificate path as `-in` argument and use `-text` to display the certificate in plain-text.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ openssl x509 -in /var/lib/minikube/certs/apiserver.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2 (0x2)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = minikubeCA
        Validity
            Not Before: Feb 14 06:54:36 2022 GMT
            Not After : Feb 14 06:54:36 2025 GMT
        Subject: O = system:masters, CN = minikube
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:c7:e1:19:9b:17:43:df:ff:31:4e:fe:66:7c:4b:
                    c4:1a:63:e6:5d:a6:a2:85:1c:11:b9:5a:72:42:50:
                    8b:25:a4:f0:89:eb:24:bf:3b:a6:e6:79:26:b8:18:
                    ed:9b:7b:76:01:68:a4:b1:1b:39:d8:b8:36:14:21:
                    44:b5:26:57:a3:a6:d3:55:e2:8c:32:5b:55:71:1e:
                    47:3e:56:b6:e8:92:86:af:aa:90:d1:4a:5a:36:ac:
                    a7:4f:a4:6c:09:a6:16:3b:e7:76:bc:41:18:89:7e:
                    be:87:df:c7:a9:ee:b7:da:34:43:ae:9f:37:cd:5d:
                    8d:e2:71:5c:e6:4c:e4:60:46:8c:b1:ef:7d:90:4b:
                    51:c3:e3:7f:a7:84:fe:06:28:1a:28:18:fd:9a:00:
                    b0:a7:d9:c9:b1:61:c9:d7:81:2d:c1:5d:5b:d2:f3:
                    f6:13:e4:d8:7f:d6:5c:c0:39:56:b1:14:04:f6:b7:
                    ea:9b:50:d7:aa:4d:f2:20:89:8b:8b:bc:81:b0:91:
                    4f:9b:f2:b9:69:b5:ce:80:67:a4:9e:f3:ba:17:03:
                    f6:89:ee:22:0e:8d:65:61:ef:16:96:67:dc:d7:4f:
                    ea:aa:36:7a:0c:59:53:2d:2a:fd:01:0f:93:15:fa:
                    8d:42:94:da:f1:0d:c8:8e:6b:15:b3:8b:4f:de:1d:
                    03:2b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:8B:FA:31:A7:8B:76:DE:C6:F5:3D:C0:BF:25:05:1D:05:78:B9:82:40

            X509v3 Subject Alternative Name:
                DNS:minikubeCA, DNS:control-plane.minikube.internal, DNS:kubernetes.default.svc.cluster.local, DNS:kubernetes.default.svc, DNS:kubernetes.default, DNS:kubernetes, DNS:localhost, IP Address:192.168.177.29, IP Address:10.96.0.1, IP Address:127.0.0.1, IP Address:10.0.0.1
    Signature Algorithm: sha256WithRSAEncryption
         8e:94:63:81:ad:57:80:84:2d:89:8b:3c:af:7c:13:d1:6c:49:
         53:53:61:cc:cb:bc:9d:63:93:9b:4e:b2:0e:a0:e3:9d:22:e4:
         4e:a9:de:75:88:05:23:46:bb:75:4c:be:ff:ba:68:e3:19:d0:
         15:b2:6a:01:5d:5b:ea:d0:a2:2d:53:80:99:25:e9:4f:f0:1a:
         65:47:c3:e4:8e:06:6c:db:23:55:57:64:f3:0d:5a:4a:e8:63:
         b2:b6:57:00:13:85:29:fe:e0:de:06:d6:e3:ec:f3:96:1d:5c:
         e7:03:8f:46:d9:bf:6b:f5:dd:1a:41:db:15:23:14:36:42:c3:
         c7:34:28:2e:a3:c4:e8:99:29:6c:28:9b:40:35:aa:58:0e:4a:
         b4:fd:0b:b4:11:a6:c5:f4:10:97:9b:c8:1c:ec:ea:a0:77:7c:
         c2:b1:70:c6:7b:85:34:8a:36:b0:ca:35:6a:7c:1c:e9:4e:08:
         9c:f9:be:de:41:ce:84:5e:51:60:52:e0:63:89:a7:18:1f:23:
         3e:f2:8e:0c:d6:9d:d2:38:04:cd:cc:2c:2e:70:c8:57:99:2b:
         3e:ba:08:1f:86:f4:0f:39:63:55:71:33:bc:49:ac:44:cf:e6:
         4f:27:dd:78:45:88:13:a7:57:d1:a3:09:76:cb:06:00:4b:84:
         df:ac:cb:0e
-----BEGIN CERTIFICATE-----
MIID3DCCAsSgAwIBAgIBAjANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
a3ViZUNBMB4XDTIyMDIxNDA2NTQzNloXDTI1MDIxNDA2NTQzNlowLDEXMBUGA1UE
ChMOc3lzdGVtOm1hc3RlcnMxETAPBgNVBAMTCG1pbmlrdWJlMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEAx+EZmxdD3/8xTv5mfEvEGmPmXaaihRwRuVpy
QlCLJaTwieskvzum5nkmuBjtm3t2AWigsRs52Lg2FCFEtSZXo6bTVeKMMltVcR5H
Pla26JKGr6qQ0UpaNqynT6RsCaYWO+d2vEEYiX6+h9/Hqe632jRDrp83zV2N4nFc
5kzkYEaMse99kEtRw+N/p4T+BigaKBj9mgCwp9nJsWHJ14EtwV1b0vP2E+TYf9Zc
wDlWsRQE9rfqm1DXqk3yIImLi7yBsJFPm/K5abXOgGeknvO6FwP2ie4iDo1lYe8W
lmfc10/qqjZ6DFlTLSr9AQ+TFfqNQpTa8Q3IjmsVs4tP3h0DKwIDAQABo4IBHjCC
ARowDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcD
AjAMBgNVHRMBAf8EAjAAMB8GA1UdIwQYMBaAFIv6MadLdt7G9T3AvyUFHQV4uYJA
MIG5BgNVHREEgbEwgb6CCm1pbmlrdWJlQ0GCH2NvbnRyb2wtcGxhbmUubWluaWt1
YmUuaW50ZXJuYWyCJGt1YmVybmV0ZXMuZGVmYXVsdC5zdmMuY2x1c3Rlci5sb2Nh
bIIWa3ViZXJuZXRlcy5kZWZhdWx0LnN2Y4ISa3ViZXJuZXRlcy5kZWZhdWx0ggpr
dWJlcm5ldGVzgglsb2NhbGhvc3SHBMCosR2HBApgAAGHBH8AAAGHBAoAAAEwDQYJ
KoZIhvcNAQELBQADggEBAI6UY4GtV4CELYmLPK98E9FsSVNTYczLvJ1jk5tOsg6g
450i5E6p3nWIBSNGu3VMvv+6aOMZ0BWyagFdW+rQoi1TgJkl6U/wGmVHw+SOBmzb
I1VXZPMNWkroY7K2VwAThSn+4N4G1uPs85YdXOcDj0bZv2v13Roh2xUjFDZCw8c0
KC6jxOiZKWwom0A1qlgOSrT9C7QRpsX0EJtbyBzs6qB3fMKxcMZ7hTSKNrDKNWp8
HOlOCJz5vt5BzoReUWBS4GOJpxgfIz7yjgzWndI4BM3MLC5wyFeZKz66CB+G9A85
Y1VxM7xJrETP5k8n3XhFiBOnV9GjCXbLBgBLhN+syw4=
-----END CERTIFICATE-----
$
```

We can see that the  `Issuer: CN = minikubeCA` ,  `Subject: O = system:masters, CN = minikube` and `Subject Alternative Name: DNS:minikubeCA, DNS:control-plane.minikube.internal, DNS:kubernetes.default.svc.cluster.local, DNS:kubernetes.default.svc, DNS:kubernetes.default, DNS:kubernetes, DNS:localhost, IP Address:192.168.177.29, IP Address:10.96.0.1, IP Address:127.0.0.1, IP Address:10.0.0.1`.

Similarly, let us take a look at the description of the `etcd` pod in the `kube-system` namespace and pay attention to the `etcd` command arguments. Again a long list, but for now, just look for cert and key files.

```shell
pradeep@learnk8s$ kubectl describe -n kube-system pods etcd-k8s
Name:                 etcd-k8s
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 k8s/192.168.177.29
Start Time:           Tue, 15 Feb 2022 12:28:03 +0530
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.177.29:2379
                      kubernetes.io/config.hash: 553a1d887eba16384375f475194d677c
                      kubernetes.io/config.mirror: 553a1d887eba16384375f475194d677c
                      kubernetes.io/config.seen: 2022-02-15T06:58:02.427571167Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   192.168.177.29
IPs:
  IP:           192.168.177.29
Controlled By:  Node/k8s
Containers:
  etcd:
    Container ID:  docker://7ad2e9b1c5fb22610382e026ecea82c4def63655f4771b398416b6dfd7b88374
    Image:         k8s.gcr.io/etcd:3.5.1-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:64b9ea357325d5db9f8a723dcf503b5a449177b17ac87d69481e126bb724c263
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.177.29:2379
      --cert-file=/var/lib/minikube/certs/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/minikube/etcd
      --initial-advertise-peer-urls=https://192.168.177.29:2380
      --initial-cluster=k8s=https://192.168.177.29:2380
      --key-file=/var/lib/minikube/certs/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.177.29:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.177.29:2380
      --name=k8s
      --peer-cert-file=/var/lib/minikube/certs/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/var/lib/minikube/certs/etcd/peer.key
      --peer-trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
      --proxy-refresh-interval=70000
      --snapshot-count=10000
      --trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
    State:          Running
      Started:      Tue, 15 Feb 2022 12:27:49 +0530
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /var/lib/minikube/certs/etcd from etcd-certs (rw)
      /var/lib/minikube/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/certs/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason     Age                   From     Message
  ----     ------     ----                  ----     -------
  Warning  Unhealthy  3h18m (x12 over 28h)  kubelet  Liveness probe failed: Get "http://127.0.0.1:2381/health": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

ETCD can have its own Certificate Authority (CA). Let us verify if both api-server and etcd are using the same CA certificate or different.

```shell
pradeep@learnk8s$ minikube ssh -p k8s
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ openssl x509 -in /var/lib/minikube/certs/etcd/ca.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = etcd-ca
        Validity
            Not Before: Feb 15 06:54:39 2022 GMT
            Not After : Feb 13 06:54:39 2032 GMT
        Subject: CN = etcd-ca
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:c2:8c:1f:12:73:c7:89:bc:f9:8d:14:c2:5e:61:
                    2f:ae:d4:f0:48:89:f7:cc:bc:be:a6:19:9d:a8:4d:
                    85:99:59:d6:96:57:35:9e:6c:62:39:31:12:e6:ee:
                    32:f9:a4:aa:ce:91:8b:5f:1d:7d:99:cc:b5:fc:c7:
                    14:75:f0:a2:43:4f:fd:e6:e7:b3:ab:11:c2:5f:db:
                    a8:72:b8:80:2f:66:5f:98:21:3b:06:af:b9:09:69:
                    94:1a:06:33:96:2f:1c:c7:f8:b2:ca:bd:87:d9:13:
                    36:c5:f6:de:aa:6a:81:c2:d4:94:2b:9e:63:dc:56:
                    27:b4:32:31:1d:49:ab:69:0e:dc:d1:14:d3:bb:f1:
                    43:80:19:31:73:29:7e:7b:d4:3b:2d:cf:14:7f:3b:
                    3c:84:4b:21:a4:2d:a6:59:79:bb:6f:1d:dc:5c:d5:
                    44:fd:3f:bd:34:b4:33:38:0d:cc:76:48:e1:de:53:
                    02:2d:54:79:44:22:64:f5:a6:39:1e:87:24:6c:91:
                    3c:5f:eb:7f:1f:84:38:e0:96:19:1b:46:9d:fc:e6:
                    79:98:c7:2e:6e:2a:2e:63:f0:28:42:57:16:14:45:
                    f3:de:bf:32:8e:d9:49:e5:ab:a5:06:6e:0d:d2:9c:
                    8b:65:40:46:17:43:3e:d2:46:53:bf:22:89:72:d0:
                    6f:49
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier:
                E3:7B:E5:78:30:93:55:DB:08:E4:EB:D0:2E:0C:30:6C:DB:17:40:40
            X509v3 Subject Alternative Name:
                DNS:etcd-ca
    Signature Algorithm: sha256WithRSAEncryption
         1a:29:48:3a:dc:24:57:35:20:b7:21:31:ed:c5:12:88:d5:79:
         7e:42:ca:21:12:92:64:49:5e:eb:2e:60:ce:16:71:5f:43:76:
         09:97:3d:6c:19:15:16:b0:6d:fc:c9:84:14:e7:5b:c2:c8:d4:
         24:77:bf:fd:87:3d:e4:c9:e4:39:49:ba:f6:41:bb:a0:9c:97:
         ef:71:b0:46:a1:86:dc:00:6b:19:26:39:32:26:c4:0c:70:4e:
         bf:1b:6b:93:55:54:d7:97:89:07:5e:e9:3b:63:a5:49:4e:6c:
         21:aa:75:8b:b1:a9:94:68:bf:1c:2e:cf:84:09:b0:52:03:62:
         72:54:b0:e2:c8:63:88:31:c0:1e:de:38:89:39:25:92:df:b9:
         1d:56:fb:c5:3b:71:fa:4e:70:e7:ec:1b:c5:fc:39:bb:71:90:
         ab:d3:36:c1:80:c5:30:6f:4d:8b:7c:8a:ee:24:15:f5:fc:5c:
         63:47:51:a0:9f:eb:30:ee:4e:95:a7:10:41:10:44:37:1e:19:
         0d:37:65:f5:94:66:4a:93:5e:fb:df:f3:24:28:17:4e:7e:7f:
         4f:d0:97:3a:24:b2:95:27:42:6f:42:0d:32:c7:b6:a6:a2:0f:
         66:df:91:e9:af:c7:66:a9:eb:01:d4:74:ae:2c:1f:72:b8:40:
         5e:15:d6:bc
-----BEGIN CERTIFICATE-----
MIIC9TCCAd2gAwIBAgIBADANBgkqhkiG9w0BAQsFADASMRAwDgYDVQQDEwdldGNk
LWNhMB4XDTIyMDIxNTA2NTQzOVoXDTMyMDIxMzA2NTQzOVowEjEQMA4GA1UEAxMH
ZXRjZC1jYTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMKMHxJzx4m8
+Y0Uwl5hL67U8EiJ98y8vqYZnahNhZlZ1pZXNZ5sYjkxEubuMvmkqs6Ri18dfZnM
tfzHFHXwokNP/ebns6sRwl/bqHK4gC9mX5ghOwabuQlplBoGM5YvHMf4ssq9h9kT
NsX23qpqgcLUlCueY9xWJ7QyMR1Jq2kO3NEU07vxQ4AZMXMpfnvUOy3PFH87PIRL
IaQtpll5u28d3FzVRP0/vTS0MzgNzHZI4d5TAi1UeUQiZPWmOR6HJGyRPF/rfx+E
OOCWGRtGnfzmeZjHLm4qLmPwKEJXFhRF896/Mo7ZSeWrpQZuDdKci2VARhdDPtJG
U78iiXLQb0kCAwEAAaNWMFQwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB/wQFMAMB
Af8wHQYDVR0OBBYEFON75Xgwk1XbCOTr0C4MMGzbF0BAMBIGA1UdEQQLMAmCB2V0
Y2QtY2EwDQYJKoZIhvcNAQELBQADggEBABopSDrfJFc1ILchMe3FEojVeX5CyiES
kmRJXusuYM4WcV9DdgmXPWwZFRawbfzJhBTnW8LI1CR3v/2HPeTJ5DlJuvZBu6Cc
l+9xsEahhtwAaxkmOTImxAxwTr8ba5NVVNeXiQde6TtjpUlObCGqdYuxqZRovxwu
z4QJsFIDYnJUsOLIY4gxwB7eOIk5JZLfuR1W+8U7cfpOcOfsG8X8ObtxkKvTNsGA
xTBvTYt8iu4kFfX8XGNHUaCf6zDuTpWnEEEQRDceGQ03ZfWUZkqTXvvf8yQoF05+
f0/QlzokspUnQm9CDTLHtqaiD2bfkemvx2ap6wHUdK4sH3K4QF4V1rw=
-----END CERTIFICATE-----
$
```
We can see it is a different Issuer: CN = etcd-ca, and Subject: CN = etcd-ca, X509v3 Subject Alternative Name: DNS:etcd-ca.

If we look at the ETCD server certificate subject details, we can see the CN as `k8s` and subject alternative names of `k8s`, `localhost` and the node IP address `192.168.177.29`.

```shell
$ openssl x509 -in /var/lib/minikube/certs/etcd/server.crt -text | grep -e Subject -e DNS
        Subject: CN = k8s
        Subject Public Key Info:
            X509v3 Subject Alternative Name:
                DNS:k8s, DNS:localhost, IP Address:192.168.177.29, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1
$
```
Also, we can see that this certificate is valid for one year.
```shell
openssl x509 -in /var/lib/minikube/certs/etcd/server.crt -text | grep -A 2 Validity
        Validity
            Not Before: Feb 15 06:54:39 2022 GMT
            Not After : Feb 15 06:54:39 2023 GMT
$
```
Where as the `minikubeCA` certificate is valid for ten years.
```shell
$ openssl x509 -in /var/lib/minikube/certs/ca.crt -text | grep -A 2 Validity
        Validity
            Not Before: Jun 28 04:53:56 2021 GMT
            Not After : Jun 27 04:53:56 2031 GMT
```
