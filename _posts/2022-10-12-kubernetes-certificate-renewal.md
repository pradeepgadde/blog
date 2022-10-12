---
layout: single
title:  "Kubernetes Certificate Renewal"
date:   2022-10-12 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/limits-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---

# Renewing Kubernetes Certificates
We can use the `kubeadm certs check-expiration` command to verify the expiry date of all certificates used in the K8s cluster.

```sh
Last login: Tue Oct 11 22:58:56 2022 from 172.25.11.254

[pradeep@kubernetes-cluster-1 ~]$ sudo kubeadm certs check-expiration

[sudo] password for pradeep: 

[check-expiration] Reading configuration from the cluster...

[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'



CERTIFICATE        EXPIRES         RESIDUAL TIME  CERTIFICATE AUTHORITY  EXTERNALLY MANAGED

admin.conf         Sep 01, 2023 06:05 UTC  323d                  no    

apiserver         Sep 01, 2023 06:05 UTC  323d      ca           no    

apiserver-etcd-client   Sep 01, 2023 06:05 UTC  323d      etcd-ca         no    

apiserver-kubelet-client  Sep 01, 2023 06:05 UTC  323d      ca           no    

controller-manager.conf  Sep 01, 2023 06:05 UTC  323d                  no    

etcd-healthcheck-client  Sep 01, 2023 06:05 UTC  323d      etcd-ca         no    

etcd-peer         Sep 01, 2023 06:05 UTC  323d      etcd-ca         no    

etcd-server        Sep 01, 2023 06:05 UTC  323d      etcd-ca         no    

front-proxy-client     Sep 01, 2023 06:05 UTC  323d      front-proxy-ca     no    

scheduler.conf       Sep 01, 2023 06:05 UTC  323d                  no    



CERTIFICATE AUTHORITY  EXPIRES         RESIDUAL TIME  EXTERNALLY MANAGED

ca           Aug 29, 2032 06:02 UTC  9y       no    

etcd-ca         Aug 29, 2032 06:02 UTC  9y       no    

front-proxy-ca     Aug 29, 2032 06:02 UTC  9y       no    

[pradeep@kubernetes-cluster-1 ~]$ 
```


```sh
[pradeep@desktop ~]$ ssh 172.25.11.44

Warning: Permanently added '172.25.11.44' (ECDSA) to the list of known hosts.

pradeep@172.25.11.44's password: 

Activate the web console with: systemctl enable --now cockpit.socket



Last login: Wed Oct 12 03:45:03 2022 from 172.25.11.254



[pradeep@kubernetes-cluster-2 ~]$ sudo kubeadm certs check-expiration

[sudo] password for pradeep: 

[check-expiration] Reading configuration from the cluster...

[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'



CERTIFICATE        EXPIRES         RESIDUAL TIME  CERTIFICATE AUTHORITY  EXTERNALLY MANAGED

admin.conf         Sep 01, 2023 06:24 UTC  323d                  no    

apiserver         Sep 01, 2023 06:24 UTC  323d      ca           no    

apiserver-etcd-client   Sep 01, 2023 06:24 UTC  323d      etcd-ca         no    

apiserver-kubelet-client  Sep 01, 2023 06:24 UTC  323d      ca           no    

controller-manager.conf  Sep 01, 2023 06:24 UTC  323d                  no    

etcd-healthcheck-client  Sep 01, 2023 06:24 UTC  323d      etcd-ca         no    

etcd-peer         Sep 01, 2023 06:24 UTC  323d      etcd-ca         no    

etcd-server        Sep 01, 2023 06:24 UTC  323d      etcd-ca         no    

front-proxy-client     Sep 01, 2023 06:24 UTC  323d      front-proxy-ca     no    

scheduler.conf       Sep 01, 2023 06:24 UTC  323d                  no    



CERTIFICATE AUTHORITY  EXPIRES         RESIDUAL TIME  EXTERNALLY MANAGED

ca           Sep 06, 2031 17:21 UTC  8y       no    

etcd-ca         Sep 06, 2031 17:21 UTC  8y       no    

front-proxy-ca     Sep 06, 2031 17:21 UTC  8y       no    
[pradeep@kubernetes-cluster-2 ~]$
```


We can renew the certificates using the `kubeadm certs renew` command.

```sh
[pradeep@kubernetes-cluster-2 ~]$ sudo kubeadm certs renew -h

This command is not meant to be run on its own. See list of avaipradeeple subcommands.



Usage:

 kubeadm certs renew [flags]

 kubeadm certs renew [command]



Avaipradeeple Commands:

 admin.conf        Renew the certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself

 all           Renew all avaipradeeple certificates

 apiserver        Renew the certificate for serving the Kubernetes API

 apiserver-etcd-client  Renew the certificate the apiserver uses to access etcd

 apiserver-kubelet-client Renew the certificate for the API server to connect to kubelet

 controller-manager.conf Renew the certificate embedded in the kubeconfig file for the controller manager to use

 etcd-healthcheck-client Renew the certificate for liveness probes to healthcheck etcd

 etcd-peer        Renew the certificate for etcd nodes to communicate with each other

 etcd-server       Renew the certificate for serving etcd

 front-proxy-client    Renew the certificate for the front proxy client

 scheduler.conf      Renew the certificate embedded in the kubeconfig file for the scheduler manager to use



Flags:

 -h, --help  help for renew



Global Flags:

   --add-dir-header      If true, adds the file directory to the header of the log messages

   --log-file string     If non-empty, use this log file

   --log-file-max-size uint  Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)

   --one-output        If true, only write logs to their native severity level (vs also writing to each lower severity level)

   --rootfs string      [EXPERIMENTAL] The path to the 'real' host root filesystem.

   --skip-headers       If true, avoid header prefixes in the log messages

   --skip-log-headers     If true, avoid headers when opening log files

 -v, --v Level         number for the log level verbosity



Use "kubeadm certs renew [command] --help" for more information about a command.

[pradeep@kubernetes-cluster-2 ~]$ 
```
Let us renew all certificates now

```sh
[pradeep@kubernetes-cluster-2 ~]$ sudo kubeadm certs renew all 

[renew] Reading configuration from the cluster...

[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'



certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed

certificate for serving the Kubernetes API renewed

certificate the apiserver uses to access etcd renewed

certificate for the API server to connect to kubelet renewed

certificate embedded in the kubeconfig file for the controller manager to use renewed

certificate for liveness probes to healthcheck etcd renewed

certificate for etcd nodes to communicate with each other renewed

certificate for serving etcd renewed

certificate for the front proxy client renewed

certificate embedded in the kubeconfig file for the scheduler manager to use renewed



Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.

```

```sh
[pradeep@kubernetes-cluster-2 ~]$ sudo kubeadm certs check-expiration

[check-expiration] Reading configuration from the cluster...

[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'



CERTIFICATE        EXPIRES         RESIDUAL TIME  CERTIFICATE AUTHORITY  EXTERNALLY MANAGED

admin.conf         Oct 12, 2023 11:33 UTC  364d                  no    

apiserver         Oct 12, 2023 11:33 UTC  364d      ca           no    

apiserver-etcd-client   Oct 12, 2023 11:33 UTC  364d      etcd-ca         no    

apiserver-kubelet-client  Oct 12, 2023 11:33 UTC  364d      ca           no    

controller-manager.conf  Oct 12, 2023 11:33 UTC  364d                  no    

etcd-healthcheck-client  Oct 12, 2023 11:33 UTC  364d      etcd-ca         no    

etcd-peer         Oct 12, 2023 11:33 UTC  364d      etcd-ca         no    

etcd-server        Oct 12, 2023 11:33 UTC  364d      etcd-ca         no    

front-proxy-client     Oct 12, 2023 11:33 UTC  364d      front-proxy-ca     no    

scheduler.conf       Oct 12, 2023 11:33 UTC  364d                  no    



CERTIFICATE AUTHORITY  EXPIRES         RESIDUAL TIME  EXTERNALLY MANAGED

ca           Sep 06, 2031 17:21 UTC  8y       no    

etcd-ca         Sep 06, 2031 17:21 UTC  8y       no    

front-proxy-ca     Sep 06, 2031 17:21 UTC  8y       no    

```

We can see all certificates are renewed for an year from today. The expiry date has changed from `Sep 01, 2023 06:24 UTC` to 

`Oct 12, 2023 11:33 UTC`.

