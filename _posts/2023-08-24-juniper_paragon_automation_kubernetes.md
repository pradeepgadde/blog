---
layout: single
title:  "Juniper Paragon Automation Platform"
date:   2023-08-24 1:55:04 +0530
categories: Kubernetes
tags: paragon
author: "Pradeep Gadde"
classes: wide

show_date: true
 
header:
  teaser: /assets/images/kubernetes.png
  
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Topics"
    nav: my-sidebar
---



## Paragon Automation Cluster Nodes

```
pradeep@paragon:~$ kubectl get nodes
NAME            STATUS   ROLES                  AGE   VERSION
172.16.100.100   Ready    control-plane,master   29d   v1.21.14
172.16.100.101   Ready    <none>                 29d   v1.21.14
172.16.100.102   Ready    <none>                 29d   v1.21.14
172.16.100.103   Ready    <none>                 29d   v1.21.14
```
## Namespaces

```
pradeep@paragon:~$ kubectl get ns
NAME                   STATUS   AGE
ambassador             Active   29d
auditlog               Active   29d
common                 Active   29d
default                Active   29d
ems                    Active   29d
fault                  Active   29d
hb-ems-dmon            Active   29d
hb-northstar           Active   29d
hb-tm                  Active   29d
healthbot              Active   29d
kube-node-lease        Active   29d
kube-public            Active   29d
kube-system            Active   29d
kubernetes-dashboard   Active   29d
metallb-system         Active   29d
northstar              Active   29d
paragonui              Active   29d
report                 Active   29d
rook-ceph              Active   29d
```
## Northstar

```
pradeep@paragon:~$ kubectl get all -n northstar
NAME                                      READY   STATUS    RESTARTS   AGE
pod/bmp-77f4788c94-g9zdg                  3/3     Running   0          3d22h
pod/dcscheduler-5bfd584489-j4tvv          2/2     Running   0          3d22h
pod/license-util-799cdfb5c5-smbm9         1/1     Running   6          29d
pod/ns-anuta-proxy-656c6dcbfc-nmd8h       2/2     Running   0          3d22h
pod/ns-anycastgroup-66f44879f7-wvtq7      1/1     Running   0          3d22h
pod/ns-celeryscheduler-8568497d69-8vzl6   1/1     Running   0          3d22h
pod/ns-celeryworker-0                     1/1     Running   0          3d22h
pod/ns-celeryworker-1                     1/1     Running   0          3d22h
pod/ns-cmgd-78d4dd9486-srgxp              3/3     Running   0          3d22h
pod/ns-configserver-654584fd79-d5mf7      2/2     Running   0          3d22h
pod/ns-dbutils-6f6f59c9c7-k6c44           1/1     Running   0          3d22h
pod/ns-dpadapter-76dd857f9b-gw9xp         2/2     Running   0          3d22h
pod/ns-epe-planner-c9944d57-wftnf         1/1     Running   0          3d22h
pod/ns-espublisher-f477dcb87-2v6kz        1/1     Running   0          3d22h
pod/ns-filter-6df5cc55f8-w4pdc            2/2     Running   0          3d22h
pod/ns-ipe-769b4f9f56-tvp6d               1/1     Running   0          3d22h
pod/ns-lsp-intent-5c44bb8b4d-zvvj2        1/1     Running   0          3d22h
pod/ns-lsp-policy-68b9999d4-tg9x2         1/1     Running   0          3d22h
pod/ns-lsp-profile-584c987d55-r8fl7       1/1     Running   0          3d22h
pod/ns-mladapter-54996b6dcd-ztb7x         2/2     Running   0          3d22h
pod/ns-ncc-557bd4494-p5mls                2/2     Running   0          3d22h
pod/ns-netconfd-678b64656-z7v2g           2/2     Running   0          3d22h
pod/ns-pceserver-c66f99c88-fvxww          2/2     Running   0          3d22h
pod/ns-pcserver-556565f84b-scmb2          2/2     Running   0          3d22h
pod/ns-pcviewer-6f55c865c7-9czwp          2/2     Running   0          3d22h
pod/ns-prpdclient-5cfdcf4f44-kvnj6        2/2     Running   0          3d22h
pod/ns-ptalkserver-6c6ff9f7b4-tm4qc       1/1     Running   1          29d
pod/ns-slice-6f458f7c9b-5v8sx             1/1     Running   0          3d22h
pod/ns-slice-selector-64df54fd86-m6jkx    1/1     Running   0          3d22h
pod/ns-srt-569b6d97fd-7z9z2               1/1     Running   0          3d22h
pod/ns-toposerver-7749bd66dc-sxt55        2/2     Running   1          3d22h
pod/ns-web-78f4754d6d-fwzff               2/2     Running   0          3d22h
pod/ns-web-planner-9db6d78d-zpnx9         2/2     Running   0          3d22h
pod/rabbitmq-0                            1/1     Running   2          3d23h
pod/rabbitmq-1                            1/1     Running   2          29d
pod/rabbitmq-2                            1/1     Running   1          29d
pod/redis-master-7f79b6789d-rfdft         1/1     Running   1          29d
pod/restconf-759974d9c9-g7h96             2/2     Running   0          3d22h

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                 AGE
service/bgp                     ClusterIP      10.107.57.33     <none>          179/TCP                                 29d
service/bmp-grpc                ClusterIP      10.101.18.224    <none>          10002/TCP,10001/TCP                     29d
service/cmgd                    ClusterIP      10.97.169.169    <none>          2222/TCP,3000/TCP                       29d
service/crpd                    ClusterIP      10.96.241.77     <none>          40051/TCP,830/TCP                       29d
service/epe-planner             ClusterIP      10.109.86.47     <none>          8081/TCP                                29d
service/ns-celeryworker         ClusterIP      None             <none>          <none>                                  29d
service/ns-filter               ClusterIP      10.98.80.60      <none>          10004/TCP,8082/TCP                      29d
service/ns-ipe                  ClusterIP      10.100.110.150   <none>          3543/TCP                                29d
service/ns-lsp-policy-svc       ClusterIP      10.106.64.194    <none>          3400/TCP                                29d
service/ns-lsp-profile-svc      ClusterIP      10.101.123.175   <none>          3400/TCP                                29d
service/ns-ncc                  ClusterIP      10.109.24.44     <none>          9092/TCP                                29d
service/ns-netconfd             ClusterIP      10.96.8.122      <none>          9091/TCP                                29d
service/ns-pceserver            LoadBalancer   10.100.82.123    192.168.1.115   4189:32726/TCP                          29d
service/ns-pcviewer             ClusterIP      10.99.123.31     <none>          7000/TCP                                29d
service/ns-slice-selector-svc   ClusterIP      10.97.250.233    <none>          3400/TCP                                29d
service/ns-slice-svc            ClusterIP      10.101.79.177    <none>          3400/TCP                                29d
service/ns-web                  ClusterIP      10.107.76.169    <none>          3301/TCP                                29d
service/ns-web-planner          ClusterIP      10.99.199.139    <none>          3600/TCP                                29d
service/ptalk                   ClusterIP      10.98.46.162     <none>          80/TCP                                  29d
service/rabbitmq                ClusterIP      10.97.127.103    <none>          4369/TCP,5672/TCP,25672/TCP,15672/TCP   29d
service/rabbitmq-headless       ClusterIP      None             <none>          4369/TCP,5672/TCP,25672/TCP,15672/TCP   29d
service/redis-master            ClusterIP      10.109.228.80    <none>          6379/TCP                                29d
service/restconf                ClusterIP      10.106.218.1     <none>          3510/TCP                                29d

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bmp                  1/1     1            1           29d
deployment.apps/dcscheduler          1/1     1            1           29d
deployment.apps/license-util         1/1     1            1           29d
deployment.apps/ns-anuta-proxy       1/1     1            1           29d
deployment.apps/ns-anycastgroup      1/1     1            1           29d
deployment.apps/ns-celeryscheduler   1/1     1            1           29d
deployment.apps/ns-cmgd              1/1     1            1           29d
deployment.apps/ns-configserver      1/1     1            1           29d
deployment.apps/ns-dbutils           1/1     1            1           29d
deployment.apps/ns-dpadapter         1/1     1            1           29d
deployment.apps/ns-epe-planner       1/1     1            1           29d
deployment.apps/ns-espublisher       1/1     1            1           29d
deployment.apps/ns-filter            1/1     1            1           29d
deployment.apps/ns-ipe               1/1     1            1           29d
deployment.apps/ns-lsp-intent        1/1     1            1           29d
deployment.apps/ns-lsp-policy        1/1     1            1           29d
deployment.apps/ns-lsp-profile       1/1     1            1           29d
deployment.apps/ns-mladapter         1/1     1            1           29d
deployment.apps/ns-ncc               1/1     1            1           29d
deployment.apps/ns-netconfd          1/1     1            1           29d
deployment.apps/ns-pceserver         1/1     1            1           29d
deployment.apps/ns-pcserver          1/1     1            1           29d
deployment.apps/ns-pcviewer          1/1     1            1           29d
deployment.apps/ns-prpdclient        1/1     1            1           29d
deployment.apps/ns-ptalkserver       1/1     1            1           29d
deployment.apps/ns-slice             1/1     1            1           29d
deployment.apps/ns-slice-selector    1/1     1            1           29d
deployment.apps/ns-srt               1/1     1            1           29d
deployment.apps/ns-toposerver        1/1     1            1           29d
deployment.apps/ns-web               1/1     1            1           29d
deployment.apps/ns-web-planner       1/1     1            1           29d
deployment.apps/redis-master         1/1     1            1           29d
deployment.apps/restconf             1/1     1            1           29d

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/bmp-77f4788c94                  1         1         1       3d22h
replicaset.apps/bmp-7878d9b57                   0         0         0       29d
replicaset.apps/dcscheduler-5bfd584489          1         1         1       3d22h
replicaset.apps/dcscheduler-68745d96c5          0         0         0       3d22h
replicaset.apps/dcscheduler-865598f9dc          0         0         0       29d
replicaset.apps/dcscheduler-b5bfc9587           0         0         0       29d
replicaset.apps/dcscheduler-fc4bd84c6           0         0         0       29d
replicaset.apps/license-util-799cdfb5c5         1         1         1       29d
replicaset.apps/ns-anuta-proxy-54fcb86447       0         0         0       29d
replicaset.apps/ns-anuta-proxy-656c6dcbfc       1         1         1       3d22h
replicaset.apps/ns-anuta-proxy-ccc7b6649        0         0         0       29d
replicaset.apps/ns-anycastgroup-5667b5fcb4      0         0         0       29d
replicaset.apps/ns-anycastgroup-66f44879f7      1         1         1       3d22h
replicaset.apps/ns-anycastgroup-6f5bb44fd4      0         0         0       3d22h
replicaset.apps/ns-anycastgroup-d95fb855f       0         0         0       29d
replicaset.apps/ns-celeryscheduler-56466458f8   0         0         0       29d
replicaset.apps/ns-celeryscheduler-5f5869df5    0         0         0       3d22h
replicaset.apps/ns-celeryscheduler-676b55b8bf   0         0         0       29d
replicaset.apps/ns-celeryscheduler-855c87548c   0         0         0       29d
replicaset.apps/ns-celeryscheduler-8568497d69   1         1         1       3d22h
replicaset.apps/ns-cmgd-5c789ddf79              0         0         0       29d
replicaset.apps/ns-cmgd-78d4dd9486              1         1         1       3d22h
replicaset.apps/ns-cmgd-7fd89f9c99              0         0         0       29d
replicaset.apps/ns-configserver-587b66d945      0         0         0       29d
replicaset.apps/ns-configserver-654584fd79      1         1         1       3d22h
replicaset.apps/ns-configserver-6d4cfd69bb      0         0         0       29d
replicaset.apps/ns-configserver-84bfbcb4c9      0         0         0       3d22h
replicaset.apps/ns-configserver-85764c89d7      0         0         0       29d
replicaset.apps/ns-dbutils-659c59fb8f           0         0         0       29d
replicaset.apps/ns-dbutils-6f6f59c9c7           1         1         1       3d22h
replicaset.apps/ns-dpadapter-6dd9c5f48d         0         0         0       29d
replicaset.apps/ns-dpadapter-76dd857f9b         1         1         1       3d22h
replicaset.apps/ns-dpadapter-fc878945f          0         0         0       29d
replicaset.apps/ns-epe-planner-669d56cff7       0         0         0       29d
replicaset.apps/ns-epe-planner-84bcf49788       0         0         0       29d
replicaset.apps/ns-epe-planner-c9944d57         1         1         1       3d22h
replicaset.apps/ns-espublisher-56d5d86d57       0         0         0       29d
replicaset.apps/ns-espublisher-658f5849f7       0         0         0       3d22h
replicaset.apps/ns-espublisher-76bdd59bc6       0         0         0       29d
replicaset.apps/ns-espublisher-7b56c65f5c       0         0         0       29d
replicaset.apps/ns-espublisher-f477dcb87        1         1         1       3d22h
replicaset.apps/ns-filter-5755d6795d            0         0         0       29d
replicaset.apps/ns-filter-6b974cbc7b            0         0         0       29d
replicaset.apps/ns-filter-6df5cc55f8            1         1         1       3d22h
replicaset.apps/ns-filter-7b967cfc77            0         0         0       29d
replicaset.apps/ns-ipe-5f8795d544               0         0         0       29d
replicaset.apps/ns-ipe-769b4f9f56               1         1         1       3d22h
replicaset.apps/ns-ipe-7c76b68bd9               0         0         0       29d
replicaset.apps/ns-ipe-dcc875c5b                0         0         0       29d
replicaset.apps/ns-lsp-intent-56c97dfc79        0         0         0       29d
replicaset.apps/ns-lsp-intent-5c44bb8b4d        1         1         1       3d22h
replicaset.apps/ns-lsp-intent-668f5c9886        0         0         0       29d
replicaset.apps/ns-lsp-intent-68845664f5        0         0         0       3d22h
replicaset.apps/ns-lsp-policy-68b9999d4         1         1         1       3d22h
replicaset.apps/ns-lsp-policy-694c78c444        0         0         0       29d
replicaset.apps/ns-lsp-policy-7784fbbf99        0         0         0       29d
replicaset.apps/ns-lsp-policy-84fb65d4c         0         0         0       3d22h
replicaset.apps/ns-lsp-profile-584c987d55       1         1         1       3d22h
replicaset.apps/ns-lsp-profile-669f86969d       0         0         0       3d22h
replicaset.apps/ns-lsp-profile-6cd767d65c       0         0         0       29d
replicaset.apps/ns-lsp-profile-86bb94656b       0         0         0       29d
replicaset.apps/ns-mladapter-54996b6dcd         1         1         1       3d22h
replicaset.apps/ns-mladapter-6c7489c696         0         0         0       29d
replicaset.apps/ns-mladapter-6fcb7cdb6f         0         0         0       29d
replicaset.apps/ns-mladapter-747f75c64          0         0         0       3d22h
replicaset.apps/ns-mladapter-78957944c8         0         0         0       29d
replicaset.apps/ns-ncc-557bd4494                1         1         1       3d22h
replicaset.apps/ns-ncc-57ffdf5d49               0         0         0       3d22h
replicaset.apps/ns-ncc-598f74bbbf               0         0         0       29d
replicaset.apps/ns-ncc-5bdbcbd67f               0         0         0       3d22h
replicaset.apps/ns-ncc-8565bc7b78               0         0         0       29d
replicaset.apps/ns-ncc-ccd686469                0         0         0       29d
replicaset.apps/ns-netconfd-57778b6786          0         0         0       29d
replicaset.apps/ns-netconfd-58975f74c8          0         0         0       3d22h
replicaset.apps/ns-netconfd-678b64656           1         1         1       3d22h
replicaset.apps/ns-netconfd-755d65dcdb          0         0         0       29d
replicaset.apps/ns-netconfd-7dc7d5bd8b          0         0         0       3d22h
replicaset.apps/ns-netconfd-8b8977597           0         0         0       29d
replicaset.apps/ns-pceserver-5464c4dfdc         0         0         0       29d
replicaset.apps/ns-pceserver-57948d4d48         0         0         0       29d
replicaset.apps/ns-pceserver-c66f99c88          1         1         1       3d22h
replicaset.apps/ns-pcserver-556565f84b          1         1         1       3d22h
replicaset.apps/ns-pcserver-84fbccb64d          0         0         0       29d
replicaset.apps/ns-pcserver-85cd9ff56b          0         0         0       29d
replicaset.apps/ns-pcviewer-54b99768c8          0         0         0       29d
replicaset.apps/ns-pcviewer-6699f57f59          0         0         0       3d22h
replicaset.apps/ns-pcviewer-6f55c865c7          1         1         1       3d22h
replicaset.apps/ns-pcviewer-6f5757b5d7          0         0         0       29d
replicaset.apps/ns-pcviewer-7cf87f479           0         0         0       3d22h
replicaset.apps/ns-pcviewer-8dd6576b9           0         0         0       29d
replicaset.apps/ns-prpdclient-5cfdcf4f44        1         1         1       3d22h
replicaset.apps/ns-prpdclient-6c99c9c4b8        0         0         0       29d
replicaset.apps/ns-prpdclient-6f4fd9cdd7        0         0         0       29d
replicaset.apps/ns-ptalkserver-6c6ff9f7b4       1         1         1       29d
replicaset.apps/ns-slice-578dff58f5             0         0         0       29d
replicaset.apps/ns-slice-6c8bb758b6             0         0         0       29d
replicaset.apps/ns-slice-6f458f7c9b             1         1         1       3d22h
replicaset.apps/ns-slice-9b9d5d787              0         0         0       3d22h
replicaset.apps/ns-slice-selector-64df54fd86    1         1         1       3d22h
replicaset.apps/ns-slice-selector-6d56c6d76d    0         0         0       29d
replicaset.apps/ns-slice-selector-7756f4f957    0         0         0       29d
replicaset.apps/ns-srt-569b6d97fd               1         1         1       3d22h
replicaset.apps/ns-srt-78b5886bbf               0         0         0       29d
replicaset.apps/ns-srt-c745c8d6b                0         0         0       29d
replicaset.apps/ns-toposerver-5b8b546f77        0         0         0       29d
replicaset.apps/ns-toposerver-6466c49d54        0         0         0       29d
replicaset.apps/ns-toposerver-6564fd6d6c        0         0         0       29d
replicaset.apps/ns-toposerver-7749bd66dc        1         1         1       3d22h
replicaset.apps/ns-web-5f9c7d6c7d               0         0         0       29d
replicaset.apps/ns-web-695b4bc577               0         0         0       29d
replicaset.apps/ns-web-6bc45d6f8d               0         0         0       3d22h
replicaset.apps/ns-web-78f4754d6d               1         1         1       3d22h
replicaset.apps/ns-web-8564f7fc6d               0         0         0       29d
replicaset.apps/ns-web-planner-7fff6bfb48       0         0         0       3d22h
replicaset.apps/ns-web-planner-9db6d78d         1         1         1       3d22h
replicaset.apps/ns-web-planner-b6fb75c6d        0         0         0       29d
replicaset.apps/ns-web-planner-c8bf65ccc        0         0         0       29d
replicaset.apps/ns-web-planner-f65c6c484        0         0         0       29d
replicaset.apps/redis-master-7f79b6789d         1         1         1       29d
replicaset.apps/restconf-546fc47d8f             0         0         0       29d
replicaset.apps/restconf-5564f6fc94             0         0         0       29d
replicaset.apps/restconf-68685cb95              0         0         0       29d
replicaset.apps/restconf-759974d9c9             1         1         1       3d22h
replicaset.apps/restconf-77d876448c             0         0         0       3d22h

NAME                               READY   AGE
statefulset.apps/ns-celeryworker   2/2     29d
statefulset.apps/rabbitmq          3/3     29d

NAME                  COMPLETIONS   DURATION   AGE
job.batch/ns-common   1/1           32s        29d
job.batch/ns-dbinit   1/1           21s        29d
job.batch/ns-hbinit   1/1           87s        29d
```

## Paragon UI 

```
pradeep@paragon:~$ kubectl get all -n paragonui
NAME                              READY   STATUS    RESTARTS   AGE
pod/paragon-ui-669fb7f46c-jn64k   1/1     Running   0          3d23h

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/paragon-ui   ClusterIP   10.108.205.213   <none>        8080/TCP   29d

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/paragon-ui   1/1     1            1           29d

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/paragon-ui-669fb7f46c   1         1         1       29d
```
## Healthbot

```
pradeep@paragon:~$ kubectl get all -n healthbot
NAME                                           READY   STATUS    RESTARTS   AGE
pod/alerta-599dc46cd6-hhsd6                    1/1     Running   1          29d
pod/api-server-68bc646745-z7lrs                1/1     Running   0          3d22h
pod/argo-server-7764598c5-sqxz6                1/1     Running   0          3d23h
pod/config-server-8574845bc-676pv              1/1     Running   0          3d22h
pod/configmanager-4xrv5                        1/1     Running   7          29d
pod/configmanager-7ljf2                        1/1     Running   6          29d
pod/configmanager-v4c4j                        1/1     Running   8          29d
pod/configmanager-xl9l8                        1/1     Running   8          29d
pod/debugger-54bb6c5548-ls5cr                  1/1     Running   0          26d
pod/grafana-86cb864bfd-4cktl                   1/1     Running   1          29d
pod/hb-proxy-syslog-5c56dc556d-7wklw           1/1     Running   0          26d
pod/hbmon-5b89df8966-lv4j7                     1/1     Running   1          29d
pod/inference-engine-5868b8b899-5sbzw          1/1     Running   1          29d
pod/influxdb-192-168-1-130-6cd9458968-xld79    1/1     Running   1          29d
pod/ingest-jti-native-proxy-84fb99b458-j7s2j   1/1     Running   1          29d
pod/ingest-snmp-proxy-8xc4z                    1/1     Running   0          3d22h
pod/ingest-snmp-proxy-czk4x                    1/1     Running   0          3d22h
pod/ingest-snmp-proxy-kh2k5                    1/1     Running   0          3d22h
pod/ingest-snmp-proxy-nnrnt                    1/1     Running   0          3d22h
pod/kube-state-metrics-87477b87c-72fts         1/1     Running   0          3d23h
pod/license-client-8f684646d-j4rz5             1/1     Running   0          26d
pod/mgd-b7bb4d5c9-jllc2                        1/1     Running   1          29d
pod/node-exporter-5xp2c                        1/1     Running   1          29d
pod/node-exporter-drm2d                        1/1     Running   1          29d
pod/node-exporter-p4lzt                        1/1     Running   1          29d
pod/node-exporter-tzld2                        1/1     Running   1          29d
pod/rca-enrichment-564c7c967d-mwnmb            1/1     Running   1          29d
pod/redisoperator-87cf4c786-gkmbt              1/1     Running   0          26d
pod/reports-5b8b77d56-p8hps                    1/1     Running   0          3d22h
pod/rfr-redis-0                                1/1     Running   0          26d
pod/rfr-redis-1                                1/1     Running   1          29d
pod/rfr-redis-2                                1/1     Running   0          26d
pod/rfs-redis-746bfcbbc7-m2h59                 1/1     Running   1          29d
pod/rfs-redis-746bfcbbc7-pt4sh                 1/1     Running   0          26d
pod/rfs-redis-746bfcbbc7-s6mvs                 1/1     Running   0          26d
pod/tsdb-shim-66f597d846-bczzv                 1/1     Running   1          29d
pod/udf-farm-6pkbn                             1/1     Running   1          29d
pod/udf-farm-8r9gv                             1/1     Running   1          29d
pod/udf-farm-9nldf                             1/1     Running   1          29d
pod/udf-farm-tftgx                             1/1     Running   1          29d
pod/workflow-controller-cd9bdc675-rv4hc        1/1     Running   2          29d

NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                    AGE
service/alerta                        ClusterIP      10.109.252.182   <none>          9090/TCP                   29d
service/api-server                    ClusterIP      10.98.225.15     <none>          9000/TCP                   29d
service/argo-server                   ClusterIP      10.111.121.212   <none>          2746/TCP                   29d
service/config-server                 ClusterIP      10.108.83.49     <none>          9000/TCP                   29d
service/debugger                      ClusterIP      10.99.139.244    <none>          22/TCP,50001/TCP           29d
service/grafana                       ClusterIP      10.97.163.51     <none>          3000/TCP                   29d
service/hb-proxy-syslog               ClusterIP      10.109.218.32    <none>          8070/TCP                   29d
service/hb-proxy-syslog-udp           LoadBalancer   10.104.58.112    192.168.1.110   514:30090/UDP              29d
service/inference-engine              ClusterIP      10.104.38.240    <none>          8080/TCP                   29d
service/influxdb                      ClusterIP      10.105.46.165    <none>          8086/TCP                   29d
service/influxdb-192-168-1-130        ClusterIP      10.101.144.209   <none>          8086/TCP,8088/TCP          29d
service/ingest-jti-native-proxy       ClusterIP      10.103.212.139   <none>          8070/TCP                   29d
service/ingest-jti-native-proxy-udp   LoadBalancer   10.100.195.243   192.168.1.110   4000:32289/UDP             29d
service/ingest-snmp-proxy             ClusterIP      10.99.107.150    <none>          8070/TCP                   3d22h
service/ingest-snmp-proxy-udp         LoadBalancer   10.110.33.220    192.168.1.110   162:31508/UDP              3d22h
service/kube-state-metrics            ClusterIP      10.104.131.21    <none>          8080/TCP,8081/TCP          29d
service/license-client                ClusterIP      10.100.232.22    <none>          50052/TCP                  29d
service/mgd                           ClusterIP      10.110.112.40    <none>          22/TCP,6500/TCP,8082/TCP   29d
service/rca-enrichment                ClusterIP      10.107.238.232   <none>          8080/TCP,50051/TCP         29d
service/reports                       ClusterIP      10.101.168.76    <none>          8080/TCP                   29d
service/rfs-redis                     ClusterIP      10.102.155.80    <none>          26379/TCP                  29d
service/tsdb                          ClusterIP      10.108.174.131   <none>          8086/TCP                   29d
service/workflow-controller-metrics   ClusterIP      10.108.48.250    <none>          9090/TCP                   29d

NAME                               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/configmanager       4         4         4       4            4           <none>          29d
daemonset.apps/ingest-snmp-proxy   4         4         4       4            4           <none>          3d22h
daemonset.apps/node-exporter       4         4         4       4            4           <none>          29d
daemonset.apps/udf-farm            4         4         4       4            4           <none>          29d

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/alerta                    1/1     1            1           29d
deployment.apps/api-server                1/1     1            1           29d
deployment.apps/argo-server               1/1     1            1           29d
deployment.apps/config-server             1/1     1            1           29d
deployment.apps/debugger                  1/1     1            1           29d
deployment.apps/grafana                   1/1     1            1           29d
deployment.apps/hb-proxy-syslog           1/1     1            1           29d
deployment.apps/hbmon                     1/1     1            1           29d
deployment.apps/inference-engine          1/1     1            1           29d
deployment.apps/influxdb-192-168-1-130    1/1     1            1           29d
deployment.apps/ingest-jti-native-proxy   1/1     1            1           29d
deployment.apps/kube-state-metrics        1/1     1            1           29d
deployment.apps/license-client            1/1     1            1           29d
deployment.apps/mgd                       1/1     1            1           29d
deployment.apps/rca-enrichment            1/1     1            1           29d
deployment.apps/redisoperator             1/1     1            1           29d
deployment.apps/reports                   1/1     1            1           29d
deployment.apps/rfs-redis                 3/3     3            3           29d
deployment.apps/tsdb-shim                 1/1     1            1           29d
deployment.apps/workflow-controller       1/1     1            1           29d

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/alerta-599dc46cd6                    1         1         1       29d
replicaset.apps/api-server-54cfb4d88f                0         0         0       29d
replicaset.apps/api-server-59ffc6546f                0         0         0       29d
replicaset.apps/api-server-68bc646745                1         1         1       3d22h
replicaset.apps/argo-server-7764598c5                1         1         1       29d
replicaset.apps/config-server-6c6794dc9c             0         0         0       29d
replicaset.apps/config-server-8574845bc              1         1         1       3d22h
replicaset.apps/config-server-f5789ffd               0         0         0       29d
replicaset.apps/debugger-54bb6c5548                  1         1         1       29d
replicaset.apps/grafana-86cb864bfd                   1         1         1       29d
replicaset.apps/hb-proxy-syslog-5c56dc556d           1         1         1       29d
replicaset.apps/hbmon-5b89df8966                     1         1         1       29d
replicaset.apps/inference-engine-5868b8b899          1         1         1       29d
replicaset.apps/influxdb-192-168-1-130-6cd9458968    1         1         1       29d
replicaset.apps/ingest-jti-native-proxy-84fb99b458   1         1         1       29d
replicaset.apps/kube-state-metrics-87477b87c         1         1         1       29d
replicaset.apps/license-client-8f684646d             1         1         1       29d
replicaset.apps/mgd-b7bb4d5c9                        1         1         1       29d
replicaset.apps/rca-enrichment-564c7c967d            1         1         1       29d
replicaset.apps/redisoperator-87cf4c786              1         1         1       29d
replicaset.apps/reports-5b8b77d56                    1         1         1       3d22h
replicaset.apps/reports-cc8c8fc44                    0         0         0       29d
replicaset.apps/reports-ff6b9595d                    0         0         0       29d
replicaset.apps/rfs-redis-746bfcbbc7                 3         3         3       29d
replicaset.apps/tsdb-shim-66f597d846                 1         1         1       29d
replicaset.apps/workflow-controller-cd9bdc675        1         1         1       29d

NAME                         READY   AGE
statefulset.apps/rfr-redis   3/3     29d

NAME                                          AGE
redisfailover.databases.spotahome.com/redis   29d
```
## Ambassador
```
pradeep@paragon:~$ kubectl get all -n ambassador
NAME                                    READY   STATUS    RESTARTS   AGE
pod/ambassador-7fc6cb695f-fjkxg         1/1     Running   0          26d
pod/ambassador-7fc6cb695f-r8wr5         1/1     Running   1          29d
pod/ambassador-7fc6cb695f-zbr4f         1/1     Running   1          29d
pod/ambassador-agent-679579885c-44v42   1/1     Running   1          26d

NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                    AGE
service/ambassador         LoadBalancer   10.108.134.158   192.168.1.100   80:31974/TCP,443:32101/TCP,7804:32737/TCP,7000:32523/TCP   29d
service/ambassador-admin   ClusterIP      10.100.88.30     <none>          8877/TCP,8005/TCP                                          29d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ambassador         3/3     3            3           29d
deployment.apps/ambassador-agent   1/1     1            1           29d

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/ambassador-7c56c8749d         0         0         0       29d
replicaset.apps/ambassador-7fc6cb695f         3         3         3       29d
replicaset.apps/ambassador-agent-679579885c   1         1         1       29d
````
## Common

```
pradeep@paragon:~$ kubectl get all -n common
NAME                                          READY   STATUS    RESTARTS   AGE
pod/ambassador-auth-jwt-8f7995477-zfvsb       1/1     Running   0          3d22h
pod/atom-db-0                                 1/1     Running   1          29d
pod/atom-db-1                                 1/1     Running   0          3d23h
pod/atom-db-2                                 1/1     Running   1          29d
pod/cert-manager-85c745f8b8-kxdkc             1/1     Running   1          29d
pod/cert-manager-cainjector-7c96fd94c-55tcv   1/1     Running   0          3d23h
pod/cert-manager-webhook-8b857c49d-4bt6c      1/1     Running   1          29d
pod/iam-79b5668f97-rjtmp                      1/1     Running   0          3d22h
pod/kafka-0                                   2/2     Running   0          3d23h
pod/kafka-1                                   2/2     Running   5          29d
pod/kafka-2                                   2/2     Running   0          3d23h
pod/local-volume-provisioner-7wr44            1/1     Running   1          29d
pod/local-volume-provisioner-95jl5            1/1     Running   1          29d
pod/local-volume-provisioner-cths6            1/1     Running   1          29d
pod/local-volume-provisioner-ztsst            1/1     Running   1          29d
pod/postgres-operator-65d76f55c6-4f7z2        1/1     Running   1          29d
pod/zookeeper-0                               1/1     Running   0          3d23h
pod/zookeeper-1                               1/1     Running   1          29d
pod/zookeeper-2                               1/1     Running   1          29d

NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ambassador-auth-jwt    ClusterIP   10.104.204.240   <none>        80/TCP                       29d
service/atom-db                ClusterIP   10.96.60.161     <none>        5432/TCP                     29d
service/atom-db-config         ClusterIP   None             <none>        <none>                       29d
service/atom-db-repl           ClusterIP   10.96.170.54     <none>        5432/TCP                     29d
service/cert-manager           ClusterIP   10.108.123.53    <none>        9402/TCP                     29d
service/cert-manager-webhook   ClusterIP   10.105.74.142    <none>        443/TCP                      29d
service/iam                    ClusterIP   None             <none>        9091/TCP                     29d
service/kafka                  ClusterIP   10.108.51.197    <none>        9092/TCP,5556/TCP            29d
service/kafka-headless         ClusterIP   None             <none>        9092/TCP                     29d
service/postgres-operator      ClusterIP   10.99.57.149     <none>        8080/TCP                     29d
service/zookeeper              ClusterIP   10.105.218.88    <none>        2181/TCP,2888/TCP,3888/TCP   29d
service/zookeeper-headless     ClusterIP   None             <none>        2181/TCP,2888/TCP,3888/TCP   29d

NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/local-volume-provisioner   4         4         4       4            4           <none>          29d

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ambassador-auth-jwt       1/1     1            1           29d
deployment.apps/cert-manager              1/1     1            1           29d
deployment.apps/cert-manager-cainjector   1/1     1            1           29d
deployment.apps/cert-manager-webhook      1/1     1            1           29d
deployment.apps/iam                       1/1     1            1           29d
deployment.apps/postgres-operator         1/1     1            1           29d

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/ambassador-auth-jwt-5b85ccd67f      0         0         0       29d
replicaset.apps/ambassador-auth-jwt-6f499c749f      0         0         0       29d
replicaset.apps/ambassador-auth-jwt-8f7995477       1         1         1       3d22h
replicaset.apps/ambassador-auth-jwt-f4b55c7b7       0         0         0       29d
replicaset.apps/cert-manager-85c745f8b8             1         1         1       29d
replicaset.apps/cert-manager-cainjector-7c96fd94c   1         1         1       29d
replicaset.apps/cert-manager-webhook-8b857c49d      1         1         1       29d
replicaset.apps/iam-646576bd66                      0         0         0       29d
replicaset.apps/iam-769b56769f                      0         0         0       29d
replicaset.apps/iam-79b5668f97                      1         1         1       3d22h
replicaset.apps/iam-d8957dd96                       0         0         0       3d22h
replicaset.apps/postgres-operator-65d76f55c6        1         1         1       29d

NAME                         READY   AGE
statefulset.apps/atom-db     3/3     29d
statefulset.apps/kafka       3/3     29d
statefulset.apps/zookeeper   3/3     29d

NAME                                            COMPLETIONS   DURATION   AGE
job.batch/iam-db-contract-0.0.2-gedc5e02263c    1/1           44s        29d
job.batch/iam-db-create-0.0.2-gedc5e02263c      1/1           12s        29d
job.batch/iam-db-init-0.0.2-gedc5e02263c        1/1           19s        29d
job.batch/iam-db-post-init-0.0.2-gedc5e02263c   1/1           25s        29d
job.batch/iam-populate-0.0.2-gedc5e02263c       1/1           31s        29d

NAME                               TEAM   VERSION   PODS   VOLUME   CPU-REQUEST   MEMORY-REQUEST   AGE   STATUS
postgresql.acid.zalan.do/atom-db   atom   12        3      5Gi      100m          100Mi            29d   Running
```
## Audit Log

```
pradeep@paragon:~$ kubectl get all -n auditlog
NAME                                     READY   STATUS      RESTARTS   AGE
pod/auditlog-594fc64fd5-knllc            1/1     Running     0          3d22h
pod/auditlog-purge-cron-28213920-nb7qw   0/1     Completed   0          4h3m

NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/auditlog   ClusterIP   10.102.192.69   <none>        9091/TCP   29d

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auditlog   1/1     1            1           29d

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/auditlog-594fc64fd5   1         1         1       3d22h
replicaset.apps/auditlog-598448df74   0         0         0       29d
replicaset.apps/auditlog-868d67c46f   0         0         0       29d

NAME                                SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/auditlog-purge-cron   0 0 * * *   False     0        4h3m            29d

NAME                                                COMPLETIONS   DURATION   AGE
job.batch/auditlog-db-contract-1.0.0-g7e8ebb2f10    1/1           13s        29d
job.batch/auditlog-db-create-1.0.0-g7e8ebb2f10      1/1           4s         29d
job.batch/auditlog-db-init-1.0.0-g7e8ebb2f10        1/1           17s        29d
job.batch/auditlog-db-partition-1.0.0-g7e8ebb2f10   1/1           27s        29d
job.batch/auditlog-purge-cron-28211040              1/1           10s        2d4h
job.batch/auditlog-purge-cron-28212480              1/1           12s        28h
job.batch/auditlog-purge-cron-28213920              1/1           12s        4h3m
pradeep@paragon:~$ kubectl get all -n ems
NAME                                        READY   STATUS      RESTARTS   AGE
pod/dcs-647d9b7cd9-82lqz                    1/1     Running     0          3d22h
pod/devicemanager-64d56bd5b6-t7r6g          1/1     Running     0          3d22h
pod/devicemodel-75b899fb74-74csh            1/1     Running     0          3d22h
pod/devicemodel-connector-f5ffbfd5f-kbr6p   1/1     Running     0          3d22h
pod/dmonproxy-775c74cf57-ck9kr              1/1     Running     0          3d22h
pod/dpm-5c5b95f9cb-mbnvx                    1/1     Running     0          3d22h
pod/jobmanager-5f58cbd6fd-7vsgm             1/1     Running     0          3d22h
pod/jobmanager-purge-cron-28213920-q5jtc    0/1     Completed   0          4h3m
pod/jobstore-6dd4fc9fd4-q6sqv               1/1     Running     0          3d22h
pod/ztpservice-fccc7675-8br6d               1/1     Running     0          3d22h

NAME                            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)             AGE
service/dcs                     ClusterIP      None           <none>          9091/TCP,7804/TCP   29d
service/devicemanager           ClusterIP      None           <none>          9091/TCP            29d
service/devicemodel             ClusterIP      None           <none>          9091/TCP            29d
service/devicemodel-connector   ClusterIP      None           <none>          9091/TCP            29d
service/dmonproxy               ClusterIP      None           <none>          9091/TCP            29d
service/dpm                     ClusterIP      None           <none>          9091/TCP            29d
service/jobmanager              ClusterIP      None           <none>          8090/TCP,9091/TCP   29d
service/jobstore                ClusterIP      None           <none>          9091/TCP            29d
service/ztpservice              ClusterIP      None           <none>          9090/TCP            29d
service/ztpservicedhcp          LoadBalancer   10.102.2.147   192.168.1.110   67:31566/UDP        29d

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dcs                     1/1     1            1           29d
deployment.apps/devicemanager           1/1     1            1           29d
deployment.apps/devicemodel             1/1     1            1           29d
deployment.apps/devicemodel-connector   1/1     1            1           29d
deployment.apps/dmonproxy               1/1     1            1           29d
deployment.apps/dpm                     1/1     1            1           29d
deployment.apps/jobmanager              1/1     1            1           29d
deployment.apps/jobstore                1/1     1            1           29d
deployment.apps/ztpservice              1/1     1            1           29d

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/dcs-647d9b7cd9                     1         1         1       3d22h
replicaset.apps/dcs-8484bcff57                     0         0         0       29d
replicaset.apps/dcs-84c845fd75                     0         0         0       29d
replicaset.apps/devicemanager-5776f4995c           0         0         0       29d
replicaset.apps/devicemanager-64d56bd5b6           1         1         1       3d22h
replicaset.apps/devicemanager-c4988c5bf            0         0         0       29d
replicaset.apps/devicemodel-5b574f8858             0         0         0       29d
replicaset.apps/devicemodel-694565ccf9             0         0         0       29d
replicaset.apps/devicemodel-75b899fb74             1         1         1       3d22h
replicaset.apps/devicemodel-connector-55f96b9cf6   0         0         0       29d
replicaset.apps/devicemodel-connector-6b9c966897   0         0         0       29d
replicaset.apps/devicemodel-connector-f5ffbfd5f    1         1         1       3d22h
replicaset.apps/dmonproxy-575fb47786               0         0         0       29d
replicaset.apps/dmonproxy-6fd76875d8               0         0         0       29d
replicaset.apps/dmonproxy-775c74cf57               1         1         1       3d22h
replicaset.apps/dpm-5c5b95f9cb                     1         1         1       3d22h
replicaset.apps/dpm-7588b685d6                     0         0         0       29d
replicaset.apps/dpm-7bdd8967c5                     0         0         0       29d
replicaset.apps/jobmanager-57797575d7              0         0         0       29d
replicaset.apps/jobmanager-5f58cbd6fd              1         1         1       3d22h
replicaset.apps/jobmanager-6b5fc4cfd8              0         0         0       29d
replicaset.apps/jobstore-6dd4fc9fd4                1         1         1       3d22h
replicaset.apps/jobstore-74bc5c86df                0         0         0       29d
replicaset.apps/jobstore-86956b4d95                0         0         0       29d
replicaset.apps/ztpservice-57f6846855              0         0         0       29d
replicaset.apps/ztpservice-86b7697586              0         0         0       29d
replicaset.apps/ztpservice-fccc7675                1         1         1       3d22h

NAME                                  SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/jobmanager-purge-cron   0 0 * * *   False     0        4h3m            29d

NAME                                                                 COMPLETIONS   DURATION   AGE
job.batch/devicemanager-copyiamsecret-0.0.2-gedc5e02263c             1/1           8s         29d
job.batch/devicemanager-copystoragesecret-0.0.2-gedc5e02263c         1/1           4s         29d
job.batch/devicemanager-db-contract-0.0.2-gedc5e02263c               1/1           45s        29d
job.batch/devicemanager-db-create-0.0.2-gedc5e02263c                 1/1           6s         29d
job.batch/devicemanager-db-init-0.0.2-gedc5e02263c                   1/1           23s        29d
job.batch/devicemanager-load-metadata-0.0.2-gedc5e02263c             1/1           66s        29d
job.batch/devicemanager-register-jobplugins-0.0.2-gedc5e02263c       1/1           20s        29d
job.batch/devicemodel-connector-copy-iam-secret-0.0.1-gedc5e02263c   1/1           6s         29d
job.batch/devicemodel-db-contract-0.0.3-gedc5e02263c                 1/1           41s        29d
job.batch/devicemodel-db-create-0.0.3-gedc5e02263c                   1/1           7s         29d
job.batch/devicemodel-db-init-0.0.3-gedc5e02263c                     1/1           17s        29d
job.batch/devicemodel-db-post-init-0.0.3-gedc5e02263c                1/1           23s        29d
job.batch/dmonproxy-db-contract-0.0.1-gedc5e02263c                   1/1           34s        29d
job.batch/dmonproxy-db-create-0.0.1-gedc5e02263c                     1/1           6s         29d
job.batch/dmonproxy-db-init-0.0.1-gedc5e02263c                       1/1           15s        29d
job.batch/dpm-copystoragesecret-0.0.2-gedc5e02263c                   1/1           5s         29d
job.batch/dpm-db-contract-0.0.2-gedc5e02263c                         1/1           33s        29d
job.batch/dpm-db-create-0.0.2-gedc5e02263c                           1/1           5s         29d
job.batch/dpm-db-init-0.0.2-gedc5e02263c                             1/1           17s        29d
job.batch/dpm-register-jobplugins-0.0.2-gedc5e02263c                 1/1           10s        29d
job.batch/jobmanager-copyiamsecret-0.0.4-gedc5e02263c                1/1           10s        29d
job.batch/jobmanager-db-create-0.0.4-gedc5e02263c                    1/1           7s         29d
job.batch/jobmanager-purge-cron-28211040                             1/1           10s        2d4h
job.batch/jobmanager-purge-cron-28212480                             1/1           9s         28h
job.batch/jobmanager-purge-cron-28213920                             1/1           11s        4h3m
job.batch/jobstore-copyiamsecret-0.0.2-gedc5e02263c                  1/1           6s         29d
job.batch/jobstore-db-contract-0.0.2-gedc5e02263c                    1/1           33s        29d
job.batch/jobstore-db-create-0.0.2-gedc5e02263c                      1/1           4s         29d
job.batch/jobstore-db-init-0.0.2-gedc5e02263c                        1/1           17s        29d
job.batch/jobstore-load-metadata-0.0.2-gedc5e02263c                  1/1           33s        29d
job.batch/ztpservice-copystoragesecret-0.0.5-gedc5e02263c            1/1           7s         29d
```
## Fault

```
pradeep@paragon:~$ kubectl get all -n fault
NAME                                                          READY   STATUS      RESTARTS   AGE
pod/alarmmanager-85d8684b87-hjxfv                             1/1     Running     2          3d22h
pod/alarmmanager-purge-cron-28213920-ss9j6                    0/1     Completed   0          4h3m
pod/telemetrymanager-8595db6884-2q59k                         1/1     Running     2          3d22h
pod/telemetrymanager-resync-devices-cron-job-28214105-dhhff   0/1     Completed   0          58m
pod/telemetrymanager-resync-faults-cron-job-28214160-7dww9    0/1     Completed   0          3m50s

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/alarmmanager       ClusterIP   10.107.128.71   <none>        9091/TCP   29d
service/telemetrymanager   ClusterIP   10.104.153.69   <none>        9091/TCP   29d

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/alarmmanager       1/1     1            1           29d
deployment.apps/telemetrymanager   1/1     1            1           29d

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/alarmmanager-5fdcdb59db       0         0         0       29d
replicaset.apps/alarmmanager-77c8c6bbbb       0         0         0       29d
replicaset.apps/alarmmanager-85d8684b87       1         1         1       3d22h
replicaset.apps/telemetrymanager-76685cfc96   0         0         0       29d
replicaset.apps/telemetrymanager-8595db6884   1         1         1       3d22h
replicaset.apps/telemetrymanager-86c8765856   0         0         0       29d

NAME                                                     SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/alarmmanager-purge-cron                    0 0 * * *   False     0        4h3m            29d
cronjob.batch/telemetrymanager-resync-devices-cron-job   5 * * * *   False     0        58m             29d
cronjob.batch/telemetrymanager-resync-faults-cron-job    0 * * * *   False     0        3m50s           29d

NAME                                                          COMPLETIONS   DURATION   AGE
job.batch/alarmmanager-db-contract-0.0.6-g10e37e0897          1/1           23s        29d
job.batch/alarmmanager-db-create-0.0.6-g10e37e0897            1/1           5s         29d
job.batch/alarmmanager-db-init-0.0.6-g10e37e0897              1/1           12s        29d
job.batch/alarmmanager-purge-cron-28213920                    1/1           12s        4h3m
job.batch/telemetrymanager-db-create-0.0.2-g10e37e0897        1/1           8s         29d
job.batch/telemetrymanager-db-init-0.0.2-g10e37e0897          1/1           14s        29d
job.batch/telemetrymanager-init-metadata-0.0.2-g10e37e0897    1/1           38s        29d
job.batch/telemetrymanager-resync-devices-cron-job-28214105   1/1           6s         58m
job.batch/telemetrymanager-resync-faults-cron-job-28214160    1/1           10s        3m50s
job.batch/telemetrymanager-sync-devices-0.0.2-g10e37e0897     1/1           105s       29d
job.batch/telemetrymanager-upload-rules-0.0.2-g10e37e0897     1/1           96s        29d
pradeep@paragon:~$ kubectl get all -n hb-ems-dmon
No resources found in hb-ems-dmon namespace.
pradeep@paragon:~$ kubectl get all -n hb-northstar
No resources found in hb-northstar namespace.
pradeep@paragon:~$ kubectl get all -n hb-tm
No resources found in hb-tm namespace.
pradeep@paragon:~$ kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-76c64f9bd7-2t52n   1/1     Running   0          3d23h
pod/kubernetes-dashboard-c9f7dd5d7-k4d2w         1/1     Running   0          26d

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.100.223.204   <none>        8000/TCP   29d
service/kubernetes-dashboard        ClusterIP   10.103.136.55    <none>        443/TCP    29d

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           29d
deployment.apps/kubernetes-dashboard        1/1     1            1           29d

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-76c64f9bd7   1         1         1       29d
replicaset.apps/kubernetes-dashboard-c9f7dd5d7         1         1         1       29d
```

## MetalLB System

```
pradeep@paragon:~$ kubectl get all -n metallb-system
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-7cccff9cfc-xt7f5   1/1     Running   0          26d
pod/speaker-fw9gx                 1/1     Running   1          29d
pod/speaker-trpbs                 1/1     Running   2          29d
pod/speaker-v2tdd                 1/1     Running   1          29d
pod/speaker-wgshc                 1/1     Running   1          29d

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
daemonset.apps/speaker   4         4         4       4            4           beta.kubernetes.io/os=linux   29d

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           29d

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-7cccff9cfc   1         1         1       29d
```

## Report

```
pradeep@paragon:~$ kubectl get all -n report
NAME                                  READY   STATUS    RESTARTS   AGE
pod/report-service-548d5cf85b-4xj6b   1/1     Running   0          3d23h

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/report-service   ClusterIP   10.101.160.162   <none>        9091/TCP   29d

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/report-service   1/1     1            1           29d

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/report-service-548d5cf85b   1         1         1       3d23h
replicaset.apps/report-service-6d9bfbb4cf   0         0         0       29d
replicaset.apps/report-service-86dc576c44   0         0         0       29d
```

## Rook CEPH

```
pradeep@paragon:~$ kubectl get all -n rook-ceph
NAME                                                          READY   STATUS      RESTARTS   AGE
pod/csi-cephfsplugin-jrxkx                                    3/3     Running     3          29d
pod/csi-cephfsplugin-mb5t2                                    3/3     Running     3          29d
pod/csi-cephfsplugin-provisioner-5bc5b556f5-dbtmf             6/6     Running     0          26d
pod/csi-cephfsplugin-provisioner-5bc5b556f5-w69qb             6/6     Running     6          29d
pod/csi-cephfsplugin-q7gk5                                    3/3     Running     3          29d
pod/csi-cephfsplugin-r7v2z                                    3/3     Running     3          29d
pod/csi-rbdplugin-bwrg5                                       3/3     Running     3          29d
pod/csi-rbdplugin-g7j2g                                       3/3     Running     3          29d
pod/csi-rbdplugin-htchk                                       3/3     Running     3          29d
pod/csi-rbdplugin-provisioner-77ddc5bc76-krwgh                6/6     Running     0          3d23h
pod/csi-rbdplugin-provisioner-77ddc5bc76-vwpd8                6/6     Running     6          29d
pod/csi-rbdplugin-qjmb9                                       3/3     Running     3          29d
pod/rook-ceph-crashcollector-172.16.100.100-6895dbc64b-ms8dx   1/1     Running     0          3d23h
pod/rook-ceph-crashcollector-172.16.100.101-65ffddc586-42rfc   1/1     Running     0          3d23h
pod/rook-ceph-crashcollector-172.16.100.102-7649df5799-tpsbb   1/1     Running     0          3d23h
pod/rook-ceph-crashcollector-172.16.100.103-794f5f4cd6-f6g52   1/1     Running     0          3d23h
pod/rook-ceph-mds-cephfs-a-6d65f75854-ptvbb                   1/1     Running     7          26d
pod/rook-ceph-mds-cephfs-b-76b646b956-86b8p                   1/1     Running     10         26d
pod/rook-ceph-mgr-a-bd75f4f9-c2j2x                            1/1     Running     6          26d
pod/rook-ceph-mon-a-7fdd7c5cf6-h96v2                          1/1     Running     0          3d23h
pod/rook-ceph-mon-b-f48fffd99-kdqkr                           1/1     Running     0          26d
pod/rook-ceph-mon-c-5f9fb7c79f-4pkfc                          1/1     Running     1          29d
pod/rook-ceph-operator-674974dcf4-shkk2                       1/1     Running     0          3d23h
pod/rook-ceph-osd-0-694849b848-xjbtz                          1/1     Running     0          3d23h
pod/rook-ceph-osd-1-55c7c8dd6d-wlqqs                          1/1     Running     0          26d
pod/rook-ceph-osd-2-6fdd46bd75-cmwgk                          1/1     Running     1          28d
pod/rook-ceph-osd-3-b8cd457f5-sr47m                           1/1     Running     1          28d
pod/rook-ceph-osd-prepare-172.16.100.102-f22hd                 0/1     Completed   0          8h
pod/rook-ceph-osd-prepare-172.16.100.103-76qtv                 0/1     Completed   0          8h
pod/rook-ceph-rgw-object-store-a-77dfb8bf89-jp7pg             1/1     Running     10         26d
pod/rook-ceph-tools-974586b47-2qz76                           1/1     Running     1          26d

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/csi-cephfsplugin-metrics     ClusterIP   10.111.78.179    <none>        8080/TCP,8081/TCP   29d
service/csi-rbdplugin-metrics        ClusterIP   10.105.20.244    <none>        8080/TCP,8081/TCP   29d
service/rook-ceph-mgr                ClusterIP   10.107.168.134   <none>        9283/TCP            29d
service/rook-ceph-mon-a              ClusterIP   10.102.144.175   <none>        6789/TCP,3300/TCP   29d
service/rook-ceph-mon-b              ClusterIP   10.110.130.128   <none>        6789/TCP,3300/TCP   29d
service/rook-ceph-mon-c              ClusterIP   10.110.236.112   <none>        6789/TCP,3300/TCP   29d
service/rook-ceph-rgw-object-store   ClusterIP   10.98.197.73     <none>        80/TCP              29d

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/csi-cephfsplugin   4         4         4       4            4           <none>          29d
daemonset.apps/csi-rbdplugin      4         4         4       4            4           <none>          29d

NAME                                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/csi-cephfsplugin-provisioner             2/2     2            2           29d
deployment.apps/csi-rbdplugin-provisioner                2/2     2            2           29d
deployment.apps/rook-ceph-crashcollector-172.16.100.100   1/1     1            1           29d
deployment.apps/rook-ceph-crashcollector-172.16.100.101   1/1     1            1           28d
deployment.apps/rook-ceph-crashcollector-172.16.100.102   1/1     1            1           29d
deployment.apps/rook-ceph-crashcollector-172.16.100.103   1/1     1            1           29d
deployment.apps/rook-ceph-mds-cephfs-a                   1/1     1            1           29d
deployment.apps/rook-ceph-mds-cephfs-b                   1/1     1            1           29d
deployment.apps/rook-ceph-mgr-a                          1/1     1            1           29d
deployment.apps/rook-ceph-mon-a                          1/1     1            1           29d
deployment.apps/rook-ceph-mon-b                          1/1     1            1           29d
deployment.apps/rook-ceph-mon-c                          1/1     1            1           29d
deployment.apps/rook-ceph-operator                       1/1     1            1           29d
deployment.apps/rook-ceph-osd-0                          1/1     1            1           29d
deployment.apps/rook-ceph-osd-1                          1/1     1            1           29d
deployment.apps/rook-ceph-osd-2                          1/1     1            1           29d
deployment.apps/rook-ceph-osd-3                          1/1     1            1           29d
deployment.apps/rook-ceph-rgw-object-store-a             1/1     1            1           29d
deployment.apps/rook-ceph-tools                          1/1     1            1           29d

NAME                                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/csi-cephfsplugin-provisioner-5bc5b556f5             2         2         2       29d
replicaset.apps/csi-rbdplugin-provisioner-77ddc5bc76                2         2         2       29d
replicaset.apps/rook-ceph-crashcollector-172.16.100.100-56b47f447b   0         0         0       29d
replicaset.apps/rook-ceph-crashcollector-172.16.100.100-6895dbc64b   1         1         1       3d23h
replicaset.apps/rook-ceph-crashcollector-172.16.100.101-65ffddc586   1         1         1       3d23h
replicaset.apps/rook-ceph-crashcollector-172.16.100.101-785749cb96   0         0         0       28d
replicaset.apps/rook-ceph-crashcollector-172.16.100.102-7649df5799   1         1         1       29d
replicaset.apps/rook-ceph-crashcollector-172.16.100.102-799df8dfbf   0         0         0       29d
replicaset.apps/rook-ceph-crashcollector-172.16.100.103-794f5f4cd6   1         1         1       29d
replicaset.apps/rook-ceph-crashcollector-172.16.100.103-7c4867c9fb   0         0         0       29d
replicaset.apps/rook-ceph-mds-cephfs-a-6d65f75854                   1         1         1       29d
replicaset.apps/rook-ceph-mds-cephfs-b-76b646b956                   1         1         1       29d
replicaset.apps/rook-ceph-mgr-a-bd75f4f9                            1         1         1       29d
replicaset.apps/rook-ceph-mon-a-7fdd7c5cf6                          1         1         1       29d
replicaset.apps/rook-ceph-mon-b-f48fffd99                           1         1         1       29d
replicaset.apps/rook-ceph-mon-c-5f9fb7c79f                          1         1         1       29d
replicaset.apps/rook-ceph-operator-674974dcf4                       1         1         1       29d
replicaset.apps/rook-ceph-osd-0-56868cfcb6                          0         0         0       29d
replicaset.apps/rook-ceph-osd-0-694849b848                          1         1         1       28d
replicaset.apps/rook-ceph-osd-1-55c7c8dd6d                          1         1         1       28d
replicaset.apps/rook-ceph-osd-1-778c958b5b                          0         0         0       29d
replicaset.apps/rook-ceph-osd-2-6fdd46bd75                          1         1         1       28d
replicaset.apps/rook-ceph-osd-2-78cc9f9fcc                          0         0         0       29d
replicaset.apps/rook-ceph-osd-3-675d4c79b9                          0         0         0       29d
replicaset.apps/rook-ceph-osd-3-b8cd457f5                           1         1         1       28d
replicaset.apps/rook-ceph-rgw-object-store-a-77dfb8bf89             1         1         1       29d
replicaset.apps/rook-ceph-tools-974586b47                           1         1         1       29d

NAME                                            COMPLETIONS   DURATION   AGE
job.batch/rook-ceph-osd-prepare-172.16.100.100   1/1           7s         8h
job.batch/rook-ceph-osd-prepare-172.16.100.101   1/1           7s         8h
job.batch/rook-ceph-osd-prepare-172.16.100.102   1/1           6s         8h
job.batch/rook-ceph-osd-prepare-172.16.100.103   1/1           7s         8h
pradeep@paragon:~$ 
```
## PV, PVC

```
pradeep@paragon:~$ kubectl get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                         STORAGECLASS    REASON   AGE
persistentvolume/local-pv-3b76bfaa                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-44e2fe7f                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-4ae50a2b                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-513edcc0                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-0       local-storage            29d
persistentvolume/local-pv-518c6a7e                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-0      local-storage            29d
persistentvolume/local-pv-5baf3f65                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-2       local-storage            29d
persistentvolume/local-pv-6a394d4f                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-6a75740e                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-78c29479                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-9e8c69d1                          244Gi      RWO            Delete           Bound       common/data-zookeeper-1       local-storage            29d
persistentvolume/local-pv-a6e2743b                          244Gi      RWO            Delete           Bound       common/data-zookeeper-2       local-storage            29d
persistentvolume/local-pv-ad89470c                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-bb27e05                           244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c0b7e581                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c41003b3                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c70d0a38                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-2      local-storage            29d
persistentvolume/local-pv-cc64f612                          244Gi      RWO            Delete           Bound       common/data-zookeeper-0       local-storage            29d
persistentvolume/local-pv-dc022675                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-1      local-storage            29d
persistentvolume/local-pv-df16b8d0                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-f371f5a4                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-1       local-storage            29d
persistentvolume/pvc-1acb3f6b-5cd4-462f-9a5b-0401696d1c1d   25Gi       RWX            Delete           Bound       kube-system/docker-registry   rook-cephfs              29d
```

```
pradeep@paragon:~$ kubectl get pv,pvc -A
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                         STORAGECLASS    REASON   AGE
persistentvolume/local-pv-3b76bfaa                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-44e2fe7f                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-4ae50a2b                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-513edcc0                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-0       local-storage            29d
persistentvolume/local-pv-518c6a7e                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-0      local-storage            29d
persistentvolume/local-pv-5baf3f65                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-2       local-storage            29d
persistentvolume/local-pv-6a394d4f                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-6a75740e                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-78c29479                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-9e8c69d1                          244Gi      RWO            Delete           Bound       common/data-zookeeper-1       local-storage            29d
persistentvolume/local-pv-a6e2743b                          244Gi      RWO            Delete           Bound       common/data-zookeeper-2       local-storage            29d
persistentvolume/local-pv-ad89470c                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-bb27e05                           244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c0b7e581                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c41003b3                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-c70d0a38                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-2      local-storage            29d
persistentvolume/local-pv-cc64f612                          244Gi      RWO            Delete           Bound       common/data-zookeeper-0       local-storage            29d
persistentvolume/local-pv-dc022675                          244Gi      RWO            Delete           Bound       common/datadir-0-kafka-1      local-storage            29d
persistentvolume/local-pv-df16b8d0                          244Gi      RWO            Delete           Available                                 local-storage            29d
persistentvolume/local-pv-f371f5a4                          244Gi      RWO            Delete           Bound       common/pgdata-atom-db-1       local-storage            29d
persistentvolume/pvc-1acb3f6b-5cd4-462f-9a5b-0401696d1c1d   25Gi       RWX            Delete           Bound       kube-system/docker-registry   rook-cephfs              29d

NAMESPACE     NAME                                      STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
common        persistentvolumeclaim/data-zookeeper-0    Bound     local-pv-cc64f612                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/data-zookeeper-1    Bound     local-pv-9e8c69d1                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/data-zookeeper-2    Bound     local-pv-a6e2743b                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/datadir-0-kafka-0   Bound     local-pv-518c6a7e                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/datadir-0-kafka-1   Bound     local-pv-dc022675                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/datadir-0-kafka-2   Bound     local-pv-c70d0a38                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/db-backup-pvc       Pending                                                                        local-storage   29d
common        persistentvolumeclaim/pgdata-atom-db-0    Bound     local-pv-513edcc0                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/pgdata-atom-db-1    Bound     local-pv-f371f5a4                          244Gi      RWO            local-storage   29d
common        persistentvolumeclaim/pgdata-atom-db-2    Bound     local-pv-5baf3f65                          244Gi      RWO            local-storage   29d
kube-system   persistentvolumeclaim/docker-registry     Bound     pvc-1acb3f6b-5cd4-462f-9a5b-0401696d1c1d   25Gi       RWX            rook-cephfs     29d
```
```
pradeep@paragon:~$ kubectl get sc -A
NAME                      PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-storage (default)   kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  29d
local-storage-hb          kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  29d
rook-ceph-block           rook-ceph.rbd.csi.ceph.com      Delete          Immediate              true                   29d
rook-ceph-bucket          rook-ceph.ceph.rook.io/bucket   Delete          Immediate              false                  29d
rook-cephfs               rook-ceph.cephfs.csi.ceph.com   Delete          Immediate              true                   29d
```
## CRDs

```
pradeep@paragon:~$ kubectl get crd -A
NAME                                                        CREATED AT
authservices.getambassador.io                               2023-07-25T23:35:54Z
bgpconfigurations.crd.projectcalico.org                     2023-07-25T23:16:32Z
bgppeers.crd.projectcalico.org                              2023-07-25T23:16:32Z
blockaffinities.crd.projectcalico.org                       2023-07-25T23:16:32Z
capabilitymappings.rbac.juniper.net                         2023-07-25T23:37:26Z
cephblockpools.ceph.rook.io                                 2023-07-25T23:18:50Z
cephclients.ceph.rook.io                                    2023-07-25T23:18:50Z
cephclusters.ceph.rook.io                                   2023-07-25T23:18:50Z
cephfilesystemmirrors.ceph.rook.io                          2023-07-25T23:18:50Z
cephfilesystems.ceph.rook.io                                2023-07-25T23:18:50Z
cephnfses.ceph.rook.io                                      2023-07-25T23:18:50Z
cephobjectrealms.ceph.rook.io                               2023-07-25T23:18:50Z
cephobjectstores.ceph.rook.io                               2023-07-25T23:18:50Z
cephobjectstoreusers.ceph.rook.io                           2023-07-25T23:18:50Z
cephobjectzonegroups.ceph.rook.io                           2023-07-25T23:18:50Z
cephobjectzones.ceph.rook.io                                2023-07-25T23:18:50Z
cephrbdmirrors.ceph.rook.io                                 2023-07-25T23:18:50Z
certificaterequests.cert-manager.io                         2023-07-25T23:35:35Z
certificates.cert-manager.io                                2023-07-25T23:35:35Z
challenges.acme.cert-manager.io                             2023-07-25T23:35:35Z
clusterinformations.crd.projectcalico.org                   2023-07-25T23:16:32Z
clusterissuers.cert-manager.io                              2023-07-25T23:35:35Z
clusterworkflowtemplates.argoproj.io                        2023-07-25T23:41:54Z
consulresolvers.getambassador.io                            2023-07-25T23:35:54Z
cronworkflows.argoproj.io                                   2023-07-25T23:41:54Z
devportals.getambassador.io                                 2023-07-25T23:35:54Z
felixconfigurations.crd.projectcalico.org                   2023-07-25T23:16:32Z
filterpolicies.getambassador.io                             2023-07-25T23:35:54Z
filters.getambassador.io                                    2023-07-25T23:35:54Z
globalnetworkpolicies.crd.projectcalico.org                 2023-07-25T23:16:32Z
globalnetworksets.crd.projectcalico.org                     2023-07-25T23:16:32Z
hostendpoints.crd.projectcalico.org                         2023-07-25T23:16:32Z
hosts.getambassador.io                                      2023-07-25T23:35:54Z
ipamblocks.crd.projectcalico.org                            2023-07-25T23:16:32Z
ipamconfigs.crd.projectcalico.org                           2023-07-25T23:16:32Z
ipamhandles.crd.projectcalico.org                           2023-07-25T23:16:32Z
ippools.crd.projectcalico.org                               2023-07-25T23:16:32Z
issuers.cert-manager.io                                     2023-07-25T23:35:35Z
kubecontrollersconfigurations.crd.projectcalico.org         2023-07-25T23:16:32Z
kubernetesendpointresolvers.getambassador.io                2023-07-25T23:35:54Z
kubernetesserviceresolvers.getambassador.io                 2023-07-25T23:35:54Z
logservices.getambassador.io                                2023-07-25T23:35:54Z
mappings.getambassador.io                                   2023-07-25T23:35:54Z
modules.getambassador.io                                    2023-07-25T23:35:54Z
networkpolicies.crd.projectcalico.org                       2023-07-25T23:16:32Z
networksets.crd.projectcalico.org                           2023-07-25T23:16:32Z
objectbucketclaims.objectbucket.io                          2023-07-25T23:18:51Z
objectbuckets.objectbucket.io                               2023-07-25T23:18:51Z
operatorconfigurations.acid.zalan.do                        2023-07-25T23:36:38Z
orders.acme.cert-manager.io                                 2023-07-25T23:35:35Z
postgresqls.acid.zalan.do                                   2023-07-25T23:36:38Z
postgresteams.acid.zalan.do                                 2023-07-25T23:36:38Z
projectcontrollers.getambassador.io                         2023-07-25T23:35:54Z
projectrevisions.getambassador.io                           2023-07-25T23:35:54Z
projects.getambassador.io                                   2023-07-25T23:35:54Z
ratelimits.getambassador.io                                 2023-07-25T23:35:54Z
ratelimitservices.getambassador.io                          2023-07-25T23:35:54Z
redisfailovers.databases.spotahome.com                      2023-07-25T23:39:07Z
rolebindings.rbac.juniper.net                               2023-07-25T23:37:26Z
roles.rbac.juniper.net                                      2023-07-25T23:37:26Z
tcpmappings.getambassador.io                                2023-07-25T23:35:54Z
tlscontexts.getambassador.io                                2023-07-25T23:35:54Z
tracingservices.getambassador.io                            2023-07-25T23:35:54Z
volumereplicationclasses.replication.storage.openshift.io   2023-07-25T23:18:51Z
volumereplications.replication.storage.openshift.io         2023-07-25T23:18:51Z
volumes.rook.io                                             2023-07-25T23:18:51Z
workflowartifactgctasks.argoproj.io                         2023-07-25T23:41:54Z
workflowdefinitions.jobmgr.juniper.net                      2023-07-25T23:38:33Z
workfloweventbindings.argoproj.io                           2023-07-25T23:41:54Z
workflowplugins.jobmgr.juniper.net                          2023-07-25T23:38:33Z
workflows.argoproj.io                                       2023-07-25T23:41:54Z
workflowtaskresults.argoproj.io                             2023-07-25T23:41:54Z
workflowtasksets.argoproj.io                                2023-07-25T23:41:55Z
workflowtemplates.argoproj.io                               2023-07-25T23:41:55Z
```

## CNI

```
pradeep@paragon:~$ cat /etc/cni/net.d/
10-calico.conflist  calico-kubeconfig   
pradeep@paragon:~$ cat /etc/cni/net.d/10-calico.conflist 
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "log_file_path": "/var/log/calico/cni/cni.log",
      "datastore_type": "kubernetes",
      "nodename": "172.16.100.100",
      "mtu": 0,
      "ipam": {
          "type": "calico-ipam",
          "assign_ipv4": "true",
          "assign_ipv6": "false"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    }
  ]
}
```

```
pradeep@paragon:~$ sudo cat /etc/cni/net.d/calico-kubeconfig 
[sudo] password for lab: 
# Kubeconfig file for Calico CNI plugin.
apiVersion: v1
kind: Config
clusters:
- name: local
  cluster:
    server: https://[10.96.0.1]:443
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EWXdOakl6TWpreU9Gb1hEVE16TURZd016SXpNamt5T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTW45CkprZENILy9wMVRYb1JNNnYrSTlYVGdYbVJBNDhJeUVTelhqb2dZdEo4ZmFvNjdRU2Y4Z1FVcWMxcHZkb1BEbE0KK0hJVWpsQlpTc25RYmhueEhWYzRaTlZkOTZuNmU1bnU0L3lRR09SMGU3czgxYkVzVHdrMEx1SCtmeUo0cVVrcApMUlppMkZVQk9JSWROQklRYWhQUXhHK3pUTkltMkN5cEJhVXVlZEQ4L3kvMDM3S0NkNGhMQlRSenJDR1VBMlNTCkVhSlBjY1lON2V0YzZRaFo1Qm9HNFRDVWRmYmNuczR3elVLS1ZJd2xnSTBEa1FnbUtqQnA3czR5QUVvVFpEeVQKbjlJa0QxdzVib3J3RGRoT0J1YTRMcXBqUVVINldMdzFEU3l2bm9sQTNaNllhblBhVzJWalFmZUhFV0dKcHRxMwpUMFJQbd5ejFPVnh4cGp2NW5FQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFbmd1SUtMMkI5Q0MyRWE1Wk8xaHU3QWtuNkhNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBa1NLam1tWGNlL1Fyb24xMmJzUDJMZ0pQTzZEblEza0t5VEZCQ3JvTXhrN3lHQXhxTQpkNjgyWEZkS3VuTW5uVmtmSkp5d1M0SmFWMUt5dzI0ZWlkMDYyQmlvR0tEYlovSCtSR1gxVkhmcUxWbjh5clJICmJtbTFYT2Jra2NSY0JaSjBkU1dJMERVSUliaEt4NHRdDI0akFRQWNQS200aEMzS2doV0dQYk9tcXExWEcrOGMKQlVsaFh3YmtyYnZMclphV3d0V3VsakF4bW9XWWhxYWJ4NkQyWWdRTTk4OUJ6enIwVncreldKaWgvYThIbnVTRwpMNzRtU2xIaDEvVm1NVnBGYk0rWktLbndzUm4xS0NCdVl1MGxOZm5UTWtwYjVNbjd6WlpMMVNOTEZsL0J2b1JICnJNMTRWbXQ2cVRGU2l4V1BsSlpWNjJxeXdvTVIvbWRlRDNuSQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  users:
- name: calico
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6Ijg1RTluc1NFUzZuS2N3YktJRS04dTduTkVtX2l4cldCTzZ6NkRWbHVQRTgifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzI0MDQyODA3LCJpYXQiOjE2OTI1MDY4MDcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInBvZCI6eyJuYW1lIjoiY2FsaWNvLW5vZGUtanE0eHgiLCJ1aWQiOiJkODM1ZGZlNy0wNjc5LTQ2OWUtYWZjZi0xZGjN2ZjZmQzMmYifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImNhbGljby1ub2RlIiwidWlkIjoiNjQ2MGE2MjItOWRiNi00MTE0LTlmYjYtNjU0YzNlOTk0NDdmIn0sIndhcm5hZnRlciI6MTY5MjUxMDQxNH0sIm5iZiI6MTY5MjUwNjgwNywic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmNhbGljby1ub2RlIn0.CgLpBlmDpkRnyAkIYjWuASVjF0pQMkVpp4ht0fpQQgIO1LeWbqzk_O42CANyUcn6MWJgs9v3lQGpy_zJoUtcgqLupVG2qKBZHbDC6jNS7q_kkB6xeyQK7tbcvPCjiyFNORawLVp7-Cb7AxhsNzzh5oaXsG7EZtAWSgwWc6NCrgDXXcEwjJZ-osAXsm2U0MXa2kIDMCgUSTPqVzh99MkHRlyyoOcMOF9c1Ga33f40R65IteYVMH5tHwgTjFUrcdizUOuMwi5vxycHw5fKKn4n3dc0P1Zq3lC7zIA5JpZtf3-JrhfQuqPKhehXJZkjc8FPKcZq8AZ7qOANj6zX4FK1w
  contexts:
- name: calico-context
  context:
    cluster: local
    user: calico
  current-context: calico-contextpradeep@paragon:~$ ls /opt/cni/bin/
  bandwidth    calico       dhcp         firewall     host-device  install      loopback     portmap      sbr          tuning       vrf          
  bridge       calico-ipam  dummy        flannel      host-local   ipvlan       macvlan      ptp          static       vlan         
```

## Pods

```
pradeep@paragon:~$ kubectl get pods -A
NAMESPACE              NAME                                                      READY   STATUS      RESTARTS   AGE
ambassador             ambassador-7fc6cb695f-fjkxg                               1/1     Running     0          26d
ambassador             ambassador-7fc6cb695f-r8wr5                               1/1     Running     1          29d
ambassador             ambassador-7fc6cb695f-zbr4f                               1/1     Running     1          29d
ambassador             ambassador-agent-679579885c-44v42                         1/1     Running     1          26d
auditlog               auditlog-594fc64fd5-knllc                                 1/1     Running     0          3d23h
auditlog               auditlog-purge-cron-28213920-nb7qw                        0/1     Completed   0          4h12m
common                 ambassador-auth-jwt-8f7995477-zfvsb                       1/1     Running     0          3d23h
common                 atom-db-0                                                 1/1     Running     1          29d
common                 atom-db-1                                                 1/1     Running     0          3d23h
common                 atom-db-2                                                 1/1     Running     1          29d
common                 cert-manager-85c745f8b8-kxdkc                             1/1     Running     1          29d
common                 cert-manager-cainjector-7c96fd94c-55tcv                   1/1     Running     0          3d23h
common                 cert-manager-webhook-8b857c49d-4bt6c                      1/1     Running     1          29d
common                 iam-79b5668f97-rjtmp                                      1/1     Running     0          3d23h
common                 kafka-0                                                   2/2     Running     0          3d23h
common                 kafka-1                                                   2/2     Running     5          29d
common                 kafka-2                                                   2/2     Running     0          3d23h
common                 local-volume-provisioner-7wr44                            1/1     Running     1          29d
common                 local-volume-provisioner-95jl5                            1/1     Running     1          29d
common                 local-volume-provisioner-cths6                            1/1     Running     1          29d
common                 local-volume-provisioner-ztsst                            1/1     Running     1          29d
common                 postgres-operator-65d76f55c6-4f7z2                        1/1     Running     1          29d
common                 zookeeper-0                                               1/1     Running     0          3d23h
common                 zookeeper-1                                               1/1     Running     1          29d
common                 zookeeper-2                                               1/1     Running     1          29d
ems                    dcs-647d9b7cd9-82lqz                                      1/1     Running     0          3d23h
ems                    devicemanager-64d56bd5b6-t7r6g                            1/1     Running     0          3d23h
ems                    devicemodel-75b899fb74-74csh                              1/1     Running     0          3d23h
ems                    devicemodel-connector-f5ffbfd5f-kbr6p                     1/1     Running     0          3d23h
ems                    dmonproxy-775c74cf57-ck9kr                                1/1     Running     0          3d23h
ems                    dpm-5c5b95f9cb-mbnvx                                      1/1     Running     0          3d23h
ems                    jobmanager-5f58cbd6fd-7vsgm                               1/1     Running     0          3d23h
ems                    jobmanager-purge-cron-28213920-q5jtc                      0/1     Completed   0          4h12m
ems                    jobstore-6dd4fc9fd4-q6sqv                                 1/1     Running     0          3d23h
ems                    ztpservice-fccc7675-8br6d                                 1/1     Running     0          3d23h
fault                  alarmmanager-85d8684b87-hjxfv                             1/1     Running     2          3d23h
fault                  alarmmanager-purge-cron-28213920-ss9j6                    0/1     Completed   0          4h12m
fault                  telemetrymanager-8595db6884-2q59k                         1/1     Running     2          3d23h
fault                  telemetrymanager-resync-devices-cron-job-28214165-vrk92   0/1     Completed   0          7m54s
fault                  telemetrymanager-resync-faults-cron-job-28214160-7dww9    0/1     Completed   0          12m
healthbot              alerta-599dc46cd6-hhsd6                                   1/1     Running     1          29d
healthbot              api-server-68bc646745-z7lrs                               1/1     Running     0          3d23h
healthbot              argo-server-7764598c5-sqxz6                               1/1     Running     0          3d23h
healthbot              config-server-8574845bc-676pv                             1/1     Running     0          3d23h
healthbot              configmanager-4xrv5                                       1/1     Running     7          29d
healthbot              configmanager-7ljf2                                       1/1     Running     6          29d
healthbot              configmanager-v4c4j                                       1/1     Running     8          29d
healthbot              configmanager-xl9l8                                       1/1     Running     8          29d
healthbot              debugger-54bb6c5548-ls5cr                                 1/1     Running     0          26d
healthbot              grafana-86cb864bfd-4cktl                                  1/1     Running     1          29d
healthbot              hb-proxy-syslog-5c56dc556d-7wklw                          1/1     Running     0          26d
healthbot              hbmon-5b89df8966-lv4j7                                    1/1     Running     1          29d
healthbot              inference-engine-5868b8b899-5sbzw                         1/1     Running     1          29d
healthbot              influxdb-192-168-1-130-6cd9458968-xld79                   1/1     Running     1          29d
healthbot              ingest-jti-native-proxy-84fb99b458-j7s2j                  1/1     Running     1          29d
healthbot              ingest-snmp-proxy-8xc4z                                   1/1     Running     0          3d22h
healthbot              ingest-snmp-proxy-czk4x                                   1/1     Running     0          3d22h
healthbot              ingest-snmp-proxy-kh2k5                                   1/1     Running     0          3d22h
healthbot              ingest-snmp-proxy-nnrnt                                   1/1     Running     0          3d22h
healthbot              kube-state-metrics-87477b87c-72fts                        1/1     Running     0          3d23h
healthbot              license-client-8f684646d-j4rz5                            1/1     Running     0          26d
healthbot              mgd-b7bb4d5c9-jllc2                                       1/1     Running     1          29d
healthbot              node-exporter-5xp2c                                       1/1     Running     1          29d
healthbot              node-exporter-drm2d                                       1/1     Running     1          29d
healthbot              node-exporter-p4lzt                                       1/1     Running     1          29d
healthbot              node-exporter-tzld2                                       1/1     Running     1          29d
healthbot              rca-enrichment-564c7c967d-mwnmb                           1/1     Running     1          29d
healthbot              redisoperator-87cf4c786-gkmbt                             1/1     Running     0          26d
healthbot              reports-5b8b77d56-p8hps                                   1/1     Running     0          3d23h
healthbot              rfr-redis-0                                               1/1     Running     0          26d
healthbot              rfr-redis-1                                               1/1     Running     1          29d
healthbot              rfr-redis-2                                               1/1     Running     0          26d
healthbot              rfs-redis-746bfcbbc7-m2h59                                1/1     Running     1          29d
healthbot              rfs-redis-746bfcbbc7-pt4sh                                1/1     Running     0          26d
healthbot              rfs-redis-746bfcbbc7-s6mvs                                1/1     Running     0          26d
healthbot              tsdb-shim-66f597d846-bczzv                                1/1     Running     1          29d
healthbot              udf-farm-6pkbn                                            1/1     Running     1          29d
healthbot              udf-farm-8r9gv                                            1/1     Running     1          29d
healthbot              udf-farm-9nldf                                            1/1     Running     1          29d
healthbot              udf-farm-tftgx                                            1/1     Running     1          29d
healthbot              workflow-controller-cd9bdc675-rv4hc                       1/1     Running     2          29d
kube-system            backup-28214160-wx5rz                                     0/1     Completed   0          12m
kube-system            backup-28214165-z5z9l                                     0/1     Completed   0          7m54s
kube-system            backup-28214170-hnp8c                                     0/1     Completed   0          2m54s
kube-system            calico-kube-controllers-866b8bd7cb-dqkg7                  1/1     Running     1          29d
kube-system            calico-node-5t4xq                                         1/1     Running     1          29d
kube-system            calico-node-798s8                                         1/1     Running     1          29d
kube-system            calico-node-jq4xx                                         1/1     Running     1          29d
kube-system            calico-node-vcfzv                                         1/1     Running     1          29d
kube-system            coredns-8dbdd69bc-8r6xz                                   1/1     Running     1          29d
kube-system            coredns-8dbdd69bc-nxtds                                   1/1     Running     1          29d
kube-system            docker-registry-docker-registry-679f7f774b-tsjvv          1/1     Running     1          29d
kube-system            docker-registry-docker-registry-679f7f774b-xr5sj          1/1     Running     0          26d
kube-system            docker-registry-docker-registry-proxy-8dmr7               1/1     Running     1          29d
kube-system            docker-registry-docker-registry-proxy-hjqkd               1/1     Running     1          29d
kube-system            docker-registry-docker-registry-proxy-jptzm               1/1     Running     1          29d
kube-system            docker-registry-docker-registry-proxy-rbqh4               1/1     Running     1          29d
kube-system            etcd-172.16.100.100                                        1/1     Running     2          29d
kube-system            kube-apiserver-172.16.100.100                              1/1     Running     1          29d
kube-system            kube-controller-manager-172.16.100.100                     1/1     Running     1          29d
kube-system            kube-proxy-9fcdr                                          1/1     Running     1          29d
kube-system            kube-proxy-ffff4                                          1/1     Running     1          29d
kube-system            kube-proxy-hstsr                                          1/1     Running     1          29d
kube-system            kube-proxy-rkb94                                          1/1     Running     1          29d
kube-system            kube-scheduler-172.16.100.100                              1/1     Running     1          29d
kube-system            kubelet-rubber-stamp-6df55984d6-62lqp                     1/1     Running     1          29d
kube-system            metrics-server-658b97c6c6-lcfzr                           1/1     Running     0          3d23h
kube-system            reloader-reloader-6cd6899bd8-qhv77                        1/1     Running     0          3d23h
kubernetes-dashboard   dashboard-metrics-scraper-76c64f9bd7-2t52n                1/1     Running     0          3d23h
kubernetes-dashboard   kubernetes-dashboard-c9f7dd5d7-k4d2w                      1/1     Running     0          26d
metallb-system         controller-7cccff9cfc-xt7f5                               1/1     Running     0          26d
metallb-system         speaker-fw9gx                                             1/1     Running     1          29d
metallb-system         speaker-trpbs                                             1/1     Running     2          29d
metallb-system         speaker-v2tdd                                             1/1     Running     1          29d
metallb-system         speaker-wgshc                                             1/1     Running     1          29d
northstar              bmp-77f4788c94-g9zdg                                      3/3     Running     0          3d23h
northstar              dcscheduler-5bfd584489-j4tvv                              2/2     Running     0          3d23h
northstar              license-util-799cdfb5c5-smbm9                             1/1     Running     6          29d
northstar              ns-anuta-proxy-656c6dcbfc-nmd8h                           2/2     Running     0          3d23h
northstar              ns-anycastgroup-66f44879f7-wvtq7                          1/1     Running     0          3d23h
northstar              ns-celeryscheduler-8568497d69-8vzl6                       1/1     Running     0          3d23h
northstar              ns-celeryworker-0                                         1/1     Running     0          3d23h
northstar              ns-celeryworker-1                                         1/1     Running     0          3d23h
northstar              ns-cmgd-78d4dd9486-srgxp                                  3/3     Running     0          3d23h
northstar              ns-configserver-654584fd79-d5mf7                          2/2     Running     0          3d23h
northstar              ns-dbutils-6f6f59c9c7-k6c44                               1/1     Running     0          3d23h
northstar              ns-dpadapter-76dd857f9b-gw9xp                             2/2     Running     0          3d23h
northstar              ns-epe-planner-c9944d57-wftnf                             1/1     Running     0          3d23h
northstar              ns-espublisher-f477dcb87-2v6kz                            1/1     Running     0          3d23h
northstar              ns-filter-6df5cc55f8-w4pdc                                2/2     Running     0          3d23h
northstar              ns-ipe-769b4f9f56-tvp6d                                   1/1     Running     0          3d23h
northstar              ns-lsp-intent-5c44bb8b4d-zvvj2                            1/1     Running     0          3d23h
northstar              ns-lsp-policy-68b9999d4-tg9x2                             1/1     Running     0          3d23h
northstar              ns-lsp-profile-584c987d55-r8fl7                           1/1     Running     0          3d23h
northstar              ns-mladapter-54996b6dcd-ztb7x                             2/2     Running     0          3d23h
northstar              ns-ncc-557bd4494-p5mls                                    2/2     Running     0          3d23h
northstar              ns-netconfd-678b64656-z7v2g                               2/2     Running     0          3d23h
northstar              ns-pceserver-c66f99c88-fvxww                              2/2     Running     0          3d23h
northstar              ns-pcserver-556565f84b-scmb2                              2/2     Running     0          3d23h
northstar              ns-pcviewer-6f55c865c7-9czwp                              2/2     Running     0          3d23h
northstar              ns-prpdclient-5cfdcf4f44-kvnj6                            2/2     Running     0          3d23h
northstar              ns-ptalkserver-6c6ff9f7b4-tm4qc                           1/1     Running     1          29d
northstar              ns-slice-6f458f7c9b-5v8sx                                 1/1     Running     0          3d23h
northstar              ns-slice-selector-64df54fd86-m6jkx                        1/1     Running     0          3d23h
northstar              ns-srt-569b6d97fd-7z9z2                                   1/1     Running     0          3d23h
northstar              ns-toposerver-7749bd66dc-sxt55                            2/2     Running     1          3d23h
northstar              ns-web-78f4754d6d-fwzff                                   2/2     Running     0          3d23h
northstar              ns-web-planner-9db6d78d-zpnx9                             2/2     Running     0          3d23h
northstar              rabbitmq-0                                                1/1     Running     2          3d23h
northstar              rabbitmq-1                                                1/1     Running     2          29d
northstar              rabbitmq-2                                                1/1     Running     1          29d
northstar              redis-master-7f79b6789d-rfdft                             1/1     Running     1          29d
northstar              restconf-759974d9c9-g7h96                                 2/2     Running     0          3d23h
paragonui              paragon-ui-669fb7f46c-jn64k                               1/1     Running     0          3d23h
report                 report-service-548d5cf85b-4xj6b                           1/1     Running     0          3d23h
rook-ceph              csi-cephfsplugin-jrxkx                                    3/3     Running     3          29d
rook-ceph              csi-cephfsplugin-mb5t2                                    3/3     Running     3          29d
rook-ceph              csi-cephfsplugin-provisioner-5bc5b556f5-dbtmf             6/6     Running     0          26d
rook-ceph              csi-cephfsplugin-provisioner-5bc5b556f5-w69qb             6/6     Running     6          29d
rook-ceph              csi-cephfsplugin-q7gk5                                    3/3     Running     3          29d
rook-ceph              csi-cephfsplugin-r7v2z                                    3/3     Running     3          29d
rook-ceph              csi-rbdplugin-bwrg5                                       3/3     Running     3          29d
rook-ceph              csi-rbdplugin-g7j2g                                       3/3     Running     3          29d
rook-ceph              csi-rbdplugin-htchk                                       3/3     Running     3          29d
rook-ceph              csi-rbdplugin-provisioner-77ddc5bc76-krwgh                6/6     Running     0          3d23h
rook-ceph              csi-rbdplugin-provisioner-77ddc5bc76-vwpd8                6/6     Running     6          29d
rook-ceph              csi-rbdplugin-qjmb9                                       3/3     Running     3          29d
rook-ceph              rook-ceph-crashcollector-172.16.100.100-6895dbc64b-ms8dx   1/1     Running     0          3d23h
rook-ceph              rook-ceph-crashcollector-172.16.100.101-65ffddc586-42rfc   1/1     Running     0          3d23h
rook-ceph              rook-ceph-crashcollector-172.16.100.102-7649df5799-tpsbb   1/1     Running     0          3d23h
rook-ceph              rook-ceph-crashcollector-172.16.100.103-794f5f4cd6-f6g52   1/1     Running     0          3d23h
rook-ceph              rook-ceph-mds-cephfs-a-6d65f75854-ptvbb                   1/1     Running     7          26d
rook-ceph              rook-ceph-mds-cephfs-b-76b646b956-86b8p                   1/1     Running     10         26d
rook-ceph              rook-ceph-mgr-a-bd75f4f9-c2j2x                            1/1     Running     6          26d
rook-ceph              rook-ceph-mon-a-7fdd7c5cf6-h96v2                          1/1     Running     0          3d23h
rook-ceph              rook-ceph-mon-b-f48fffd99-kdqkr                           1/1     Running     0          26d
rook-ceph              rook-ceph-mon-c-5f9fb7c79f-4pkfc                          1/1     Running     1          29d
rook-ceph              rook-ceph-operator-674974dcf4-shkk2                       1/1     Running     0          3d23h
rook-ceph              rook-ceph-osd-0-694849b848-xjbtz                          1/1     Running     0          3d23h
rook-ceph              rook-ceph-osd-1-55c7c8dd6d-wlqqs                          1/1     Running     0          26d
rook-ceph              rook-ceph-osd-2-6fdd46bd75-cmwgk                          1/1     Running     1          28d
rook-ceph              rook-ceph-osd-3-b8cd457f5-sr47m                           1/1     Running     1          28d
rook-ceph              rook-ceph-osd-prepare-172.16.100.102-f22hd                 0/1     Completed   0          8h
rook-ceph              rook-ceph-osd-prepare-172.16.100.103-76qtv                 0/1     Completed   0          8h
rook-ceph              rook-ceph-rgw-object-store-a-77dfb8bf89-jp7pg             1/1     Running     10         26d
rook-ceph              rook-ceph-tools-974586b47-2qz76                           1/1     Running     1          26d
pradeep@paragon:~$ kubectl get pods -A | wc -l
187
```
## Deployments

```
pradeep@paragon:~$ kubectl get deploy -A 
NAMESPACE              NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
ambassador             ambassador                               3/3     3            3           29d
ambassador             ambassador-agent                         1/1     1            1           29d
auditlog               auditlog                                 1/1     1            1           29d
common                 ambassador-auth-jwt                      1/1     1            1           29d
common                 cert-manager                             1/1     1            1           29d
common                 cert-manager-cainjector                  1/1     1            1           29d
common                 cert-manager-webhook                     1/1     1            1           29d
common                 iam                                      1/1     1            1           29d
common                 postgres-operator                        1/1     1            1           29d
ems                    dcs                                      1/1     1            1           29d
ems                    devicemanager                            1/1     1            1           29d
ems                    devicemodel                              1/1     1            1           29d
ems                    devicemodel-connector                    1/1     1            1           29d
ems                    dmonproxy                                1/1     1            1           29d
ems                    dpm                                      1/1     1            1           29d
ems                    jobmanager                               1/1     1            1           29d
ems                    jobstore                                 1/1     1            1           29d
ems                    ztpservice                               1/1     1            1           29d
fault                  alarmmanager                             1/1     1            1           29d
fault                  telemetrymanager                         1/1     1            1           29d
healthbot              alerta                                   1/1     1            1           29d
healthbot              api-server                               1/1     1            1           29d
healthbot              argo-server                              1/1     1            1           29d
healthbot              config-server                            1/1     1            1           29d
healthbot              debugger                                 1/1     1            1           29d
healthbot              grafana                                  1/1     1            1           29d
healthbot              hb-proxy-syslog                          1/1     1            1           29d
healthbot              hbmon                                    1/1     1            1           29d
healthbot              inference-engine                         1/1     1            1           29d
healthbot              influxdb-192-168-1-130                   1/1     1            1           29d
healthbot              ingest-jti-native-proxy                  1/1     1            1           29d
healthbot              kube-state-metrics                       1/1     1            1           29d
healthbot              license-client                           1/1     1            1           29d
healthbot              mgd                                      1/1     1            1           29d
healthbot              rca-enrichment                           1/1     1            1           29d
healthbot              redisoperator                            1/1     1            1           29d
healthbot              reports                                  1/1     1            1           29d
healthbot              rfs-redis                                3/3     3            3           29d
healthbot              tsdb-shim                                1/1     1            1           29d
healthbot              workflow-controller                      1/1     1            1           29d
kube-system            calico-kube-controllers                  1/1     1            1           29d
kube-system            coredns                                  2/2     2            2           29d
kube-system            docker-registry-docker-registry          2/2     2            2           29d
kube-system            kubelet-rubber-stamp                     1/1     1            1           29d
kube-system            metrics-server                           1/1     1            1           29d
kube-system            reloader-reloader                        1/1     1            1           29d
kubernetes-dashboard   dashboard-metrics-scraper                1/1     1            1           29d
kubernetes-dashboard   kubernetes-dashboard                     1/1     1            1           29d
metallb-system         controller                               1/1     1            1           29d
northstar              bmp                                      1/1     1            1           29d
northstar              dcscheduler                              1/1     1            1           29d
northstar              license-util                             1/1     1            1           29d
northstar              ns-anuta-proxy                           1/1     1            1           29d
northstar              ns-anycastgroup                          1/1     1            1           29d
northstar              ns-celeryscheduler                       1/1     1            1           29d
northstar              ns-cmgd                                  1/1     1            1           29d
northstar              ns-configserver                          1/1     1            1           29d
northstar              ns-dbutils                               1/1     1            1           29d
northstar              ns-dpadapter                             1/1     1            1           29d
northstar              ns-epe-planner                           1/1     1            1           29d
northstar              ns-espublisher                           1/1     1            1           29d
northstar              ns-filter                                1/1     1            1           29d
northstar              ns-ipe                                   1/1     1            1           29d
northstar              ns-lsp-intent                            1/1     1            1           29d
northstar              ns-lsp-policy                            1/1     1            1           29d
northstar              ns-lsp-profile                           1/1     1            1           29d
northstar              ns-mladapter                             1/1     1            1           29d
northstar              ns-ncc                                   1/1     1            1           29d
northstar              ns-netconfd                              1/1     1            1           29d
northstar              ns-pceserver                             1/1     1            1           29d
northstar              ns-pcserver                              1/1     1            1           29d
northstar              ns-pcviewer                              1/1     1            1           29d
northstar              ns-prpdclient                            1/1     1            1           29d
northstar              ns-ptalkserver                           1/1     1            1           29d
northstar              ns-slice                                 1/1     1            1           29d
northstar              ns-slice-selector                        1/1     1            1           29d
northstar              ns-srt                                   1/1     1            1           29d
northstar              ns-toposerver                            1/1     1            1           29d
northstar              ns-web                                   1/1     1            1           29d
northstar              ns-web-planner                           1/1     1            1           29d
northstar              redis-master                             1/1     1            1           29d
northstar              restconf                                 1/1     1            1           29d
paragonui              paragon-ui                               1/1     1            1           29d
report                 report-service                           1/1     1            1           29d
rook-ceph              csi-cephfsplugin-provisioner             2/2     2            2           29d
rook-ceph              csi-rbdplugin-provisioner                2/2     2            2           29d
rook-ceph              rook-ceph-crashcollector-172.16.100.100   1/1     1            1           29d
rook-ceph              rook-ceph-crashcollector-172.16.100.101   1/1     1            1           28d
rook-ceph              rook-ceph-crashcollector-172.16.100.102   1/1     1            1           29d
rook-ceph              rook-ceph-crashcollector-172.16.100.103   1/1     1            1           29d
rook-ceph              rook-ceph-mds-cephfs-a                   1/1     1            1           29d
rook-ceph              rook-ceph-mds-cephfs-b                   1/1     1            1           29d
rook-ceph              rook-ceph-mgr-a                          1/1     1            1           29d
rook-ceph              rook-ceph-mon-a                          1/1     1            1           29d
rook-ceph              rook-ceph-mon-b                          1/1     1            1           29d
rook-ceph              rook-ceph-mon-c                          1/1     1            1           29d
rook-ceph              rook-ceph-operator                       1/1     1            1           29d
rook-ceph              rook-ceph-osd-0                          1/1     1            1           29d
rook-ceph              rook-ceph-osd-1                          1/1     1            1           29d
rook-ceph              rook-ceph-osd-2                          1/1     1            1           29d
rook-ceph              rook-ceph-osd-3                          1/1     1            1           29d
rook-ceph              rook-ceph-rgw-object-store-a             1/1     1            1           29d
rook-ceph              rook-ceph-tools                          1/1     1            1           29d
```
```
pradeep@paragon:~$ kubectl get deploy -A | wc -l
104
```
## Services

```
pradeep@paragon:~$ kubectl get service -A 
NAMESPACE              NAME                              TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                    AGE
ambassador             ambassador                        LoadBalancer   10.108.134.158   192.168.1.100   80:31974/TCP,443:32101/TCP,7804:32737/TCP,7000:32523/TCP   29d
ambassador             ambassador-admin                  ClusterIP      10.100.88.30     <none>          8877/TCP,8005/TCP                                          29d
auditlog               auditlog                          ClusterIP      10.102.192.69    <none>          9091/TCP                                                   29d
common                 ambassador-auth-jwt               ClusterIP      10.104.204.240   <none>          80/TCP                                                     29d
common                 atom-db                           ClusterIP      10.96.60.161     <none>          5432/TCP                                                   29d
common                 atom-db-config                    ClusterIP      None             <none>          <none>                                                     29d
common                 atom-db-repl                      ClusterIP      10.96.170.54     <none>          5432/TCP                                                   29d
common                 cert-manager                      ClusterIP      10.108.123.53    <none>          9402/TCP                                                   29d
common                 cert-manager-webhook              ClusterIP      10.105.74.142    <none>          443/TCP                                                    29d
common                 iam                               ClusterIP      None             <none>          9091/TCP                                                   29d
common                 kafka                             ClusterIP      10.108.51.197    <none>          9092/TCP,5556/TCP                                          29d
common                 kafka-headless                    ClusterIP      None             <none>          9092/TCP                                                   29d
common                 postgres-operator                 ClusterIP      10.99.57.149     <none>          8080/TCP                                                   29d
common                 zookeeper                         ClusterIP      10.105.218.88    <none>          2181/TCP,2888/TCP,3888/TCP                                 29d
common                 zookeeper-headless                ClusterIP      None             <none>          2181/TCP,2888/TCP,3888/TCP                                 29d
default                kubernetes                        ClusterIP      10.96.0.1        <none>          443/TCP                                                    29d
ems                    dcs                               ClusterIP      None             <none>          9091/TCP,7804/TCP                                          29d
ems                    devicemanager                     ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    devicemodel                       ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    devicemodel-connector             ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    dmonproxy                         ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    dpm                               ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    jobmanager                        ClusterIP      None             <none>          8090/TCP,9091/TCP                                          29d
ems                    jobstore                          ClusterIP      None             <none>          9091/TCP                                                   29d
ems                    ztpservice                        ClusterIP      None             <none>          9090/TCP                                                   29d
ems                    ztpservicedhcp                    LoadBalancer   10.102.2.147     192.168.1.110   67:31566/UDP                                               29d
fault                  alarmmanager                      ClusterIP      10.107.128.71    <none>          9091/TCP                                                   29d
fault                  telemetrymanager                  ClusterIP      10.104.153.69    <none>          9091/TCP                                                   29d
healthbot              alerta                            ClusterIP      10.109.252.182   <none>          9090/TCP                                                   29d
healthbot              api-server                        ClusterIP      10.98.225.15     <none>          9000/TCP                                                   29d
healthbot              argo-server                       ClusterIP      10.111.121.212   <none>          2746/TCP                                                   29d
healthbot              config-server                     ClusterIP      10.108.83.49     <none>          9000/TCP                                                   29d
healthbot              debugger                          ClusterIP      10.99.139.244    <none>          22/TCP,50001/TCP                                           29d
healthbot              grafana                           ClusterIP      10.97.163.51     <none>          3000/TCP                                                   29d
healthbot              hb-proxy-syslog                   ClusterIP      10.109.218.32    <none>          8070/TCP                                                   29d
healthbot              hb-proxy-syslog-udp               LoadBalancer   10.104.58.112    192.168.1.110   514:30090/UDP                                              29d
healthbot              inference-engine                  ClusterIP      10.104.38.240    <none>          8080/TCP                                                   29d
healthbot              influxdb                          ClusterIP      10.105.46.165    <none>          8086/TCP                                                   29d
healthbot              influxdb-192-168-1-130            ClusterIP      10.101.144.209   <none>          8086/TCP,8088/TCP                                          29d
healthbot              ingest-jti-native-proxy           ClusterIP      10.103.212.139   <none>          8070/TCP                                                   29d
healthbot              ingest-jti-native-proxy-udp       LoadBalancer   10.100.195.243   192.168.1.110   4000:32289/UDP                                             29d
healthbot              ingest-snmp-proxy                 ClusterIP      10.99.107.150    <none>          8070/TCP                                                   3d22h
healthbot              ingest-snmp-proxy-udp             LoadBalancer   10.110.33.220    192.168.1.110   162:31508/UDP                                              3d22h
healthbot              kube-state-metrics                ClusterIP      10.104.131.21    <none>          8080/TCP,8081/TCP                                          29d
healthbot              license-client                    ClusterIP      10.100.232.22    <none>          50052/TCP                                                  29d
healthbot              mgd                               ClusterIP      10.110.112.40    <none>          22/TCP,6500/TCP,8082/TCP                                   29d
healthbot              rca-enrichment                    ClusterIP      10.107.238.232   <none>          8080/TCP,50051/TCP                                         29d
healthbot              reports                           ClusterIP      10.101.168.76    <none>          8080/TCP                                                   29d
healthbot              rfs-redis                         ClusterIP      10.102.155.80    <none>          26379/TCP                                                  29d
healthbot              tsdb                              ClusterIP      10.108.174.131   <none>          8086/TCP                                                   29d
healthbot              workflow-controller-metrics       ClusterIP      10.108.48.250    <none>          9090/TCP                                                   29d
kube-system            docker-registry-docker-registry   ClusterIP      10.106.54.87     <none>          5000/TCP                                                   29d
kube-system            kube-dns                          ClusterIP      10.96.0.10       <none>          53/UDP,53/TCP,9153/TCP                                     29d
kube-system            metrics-server                    ClusterIP      10.111.156.117   <none>          443/TCP                                                    29d
kubernetes-dashboard   dashboard-metrics-scraper         ClusterIP      10.100.223.204   <none>          8000/TCP                                                   29d
kubernetes-dashboard   kubernetes-dashboard              ClusterIP      10.103.136.55    <none>          443/TCP                                                    29d
northstar              bgp                               ClusterIP      10.107.57.33     <none>          179/TCP                                                    29d
northstar              bmp-grpc                          ClusterIP      10.101.18.224    <none>          10002/TCP,10001/TCP                                        29d
northstar              cmgd                              ClusterIP      10.97.169.169    <none>          2222/TCP,3000/TCP                                          29d
northstar              crpd                              ClusterIP      10.96.241.77     <none>          40051/TCP,830/TCP                                          29d
northstar              epe-planner                       ClusterIP      10.109.86.47     <none>          8081/TCP                                                   29d
northstar              ns-celeryworker                   ClusterIP      None             <none>          <none>                                                     29d
northstar              ns-filter                         ClusterIP      10.98.80.60      <none>          10004/TCP,8082/TCP                                         29d
northstar              ns-ipe                            ClusterIP      10.100.110.150   <none>          3543/TCP                                                   29d
northstar              ns-lsp-policy-svc                 ClusterIP      10.106.64.194    <none>          3400/TCP                                                   29d
northstar              ns-lsp-profile-svc                ClusterIP      10.101.123.175   <none>          3400/TCP                                                   29d
northstar              ns-ncc                            ClusterIP      10.109.24.44     <none>          9092/TCP                                                   29d
northstar              ns-netconfd                       ClusterIP      10.96.8.122      <none>          9091/TCP                                                   29d
northstar              ns-pceserver                      LoadBalancer   10.100.82.123    192.168.1.115   4189:32726/TCP                                             29d
northstar              ns-pcviewer                       ClusterIP      10.99.123.31     <none>          7000/TCP                                                   29d
northstar              ns-slice-selector-svc             ClusterIP      10.97.250.233    <none>          3400/TCP                                                   29d
northstar              ns-slice-svc                      ClusterIP      10.101.79.177    <none>          3400/TCP                                                   29d
northstar              ns-web                            ClusterIP      10.107.76.169    <none>          3301/TCP                                                   29d
northstar              ns-web-planner                    ClusterIP      10.99.199.139    <none>          3600/TCP                                                   29d
northstar              ptalk                             ClusterIP      10.98.46.162     <none>          80/TCP                                                     29d
northstar              rabbitmq                          ClusterIP      10.97.127.103    <none>          4369/TCP,5672/TCP,25672/TCP,15672/TCP                      29d
northstar              rabbitmq-headless                 ClusterIP      None             <none>          4369/TCP,5672/TCP,25672/TCP,15672/TCP                      29d
northstar              redis-master                      ClusterIP      10.109.228.80    <none>          6379/TCP                                                   29d
northstar              restconf                          ClusterIP      10.106.218.1     <none>          3510/TCP                                                   29d
paragonui              paragon-ui                        ClusterIP      10.108.205.213   <none>          8080/TCP                                                   29d
report                 report-service                    ClusterIP      10.101.160.162   <none>          9091/TCP                                                   29d
rook-ceph              csi-cephfsplugin-metrics          ClusterIP      10.111.78.179    <none>          8080/TCP,8081/TCP                                          29d
rook-ceph              csi-rbdplugin-metrics             ClusterIP      10.105.20.244    <none>          8080/TCP,8081/TCP                                          29d
rook-ceph              rook-ceph-mgr                     ClusterIP      10.107.168.134   <none>          9283/TCP                                                   29d
rook-ceph              rook-ceph-mon-a                   ClusterIP      10.102.144.175   <none>          6789/TCP,3300/TCP                                          29d
rook-ceph              rook-ceph-mon-b                   ClusterIP      10.110.130.128   <none>          6789/TCP,3300/TCP                                          29d
rook-ceph              rook-ceph-mon-c                   ClusterIP      10.110.236.112   <none>          6789/TCP,3300/TCP                                          29d
rook-ceph              rook-ceph-rgw-object-store        ClusterIP      10.98.197.73     <none>          80/TCP                                                     29d
```
```
pradeep@paragon:~$ kubectl get service -A | wc -l
89
```
## BMP

```
pradeep@paragon:~$ kubectl describe pod bmp-77f4788c94-g9zdg -n northstar
Name:         bmp-77f4788c94-g9zdg
Namespace:    northstar
Priority:     0
Node:         172.16.100.103/172.16.100.103
Start Time:   Sat, 19 Aug 2023 22:04:23 -0700
Labels:       app=bmp
              northstar=bmp
              pod-template-hash=77f4788c94
Annotations:  cni.projectcalico.org/podIP: 10.244.254.137/32
              cni.projectcalico.org/podIPs: 10.244.254.137/32
Status:       Running
IP:           10.244.254.137
IPs:
  IP:           10.244.254.137
Controlled By:  ReplicaSet/bmp-77f4788c94
Containers:
  bmp:
    Container ID:  docker://6ced3c5dfe92fe7f9531d0b7bf8aacdf40031aa2d841637a77c34593a9f46feb
    Image:         localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/bmp:release-23-1-ge572e4b914
    Image ID:      docker-pullable://localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/bmp@sha256:900d40f79ff43720deb53a4b20ced9cffc1a9d01fa09314a55cd56a500ccbd33
    Ports:         10001/TCP, 10002/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /bin/sh
    Args:
      -c
      bmpMonitor -p 10001 -g 10002 -r 127.0.0.1:10003
    State:          Running
      Started:      Sat, 19 Aug 2023 22:04:32 -0700
    Ready:          True
    Restart Count:  0
    Liveness:       tcp-socket :10002 delay=5s timeout=1s period=15s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j7g5p (ro)
  syslog:
    Container ID:   docker://295059225f88613ea915d2affca6cc0dac1146eb1271ccc53e1966aa623d4c9b
    Image:          localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/syslog:v7.0.0
    Image ID:       docker-pullable://localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/syslog@sha256:d885aeb694f00b192b01f521fd2b49a41574ad14dc1acf75466495f181448857
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 19 Aug 2023 22:04:32 -0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/rsyslog/dev from syslog (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j7g5p (ro)
  crpd:
    Container ID:  docker://3eca1e146303d9bcb123d8b09190d984e4436e6259b40b67b4f47c8e56bef416
    Image:         localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/ns-crpd:21.4R2.2
    Image ID:      docker-pullable://localhost:5000/eng-registry.juniper.net/northstar-scm/northstar-containers/ns-crpd@sha256:adc9a8046bdd4f17e8ba507f46872045480af82ecdd46bf2a7678001104db928
    Ports:         179/TCP, 10003/TCP
    Host Ports:    0/TCP, 0/TCP
    Command:
      /bin/sh
    Args:
      -c
      if [ -e /var/run/secrets/crpd-license ]; then
        cp -f /var/run/secrets/crpd-license/license /opt/init/license.txt;
      fi && rm -f /dev/log /etc/runit/runsvdir/default/rsyslog && ln -s /var/run/rsyslog/dev/log /dev && sed -i 's#^\(.*/usr/sbin/license-check .*\)$#\1 | grep -v "^[0-9] "#' /etc/sv/license-check/run && exec /sbin/runit-init 0
      
    State:          Running
      Started:      Sat, 19 Aug 2023 22:04:55 -0700
    Ready:          True
    Restart Count:  0
    Liveness:       tcp-socket :10003 delay=0s timeout=1s period=15s #success=1 #failure=3
    Startup:        tcp-socket :10003 delay=60s timeout=1s period=15s #success=1 #failure=20
    Environment:
      AS:                              64512
      STAKATER_CRPD_CONFIG_CONFIGMAP:  1f7947ad4307346c002f3c60671b5d417d6a938b
    Mounts:
      /lib/modules from modules (rw)
      /run/config from config (rw)
      /var/run/rsyslog/dev from syslog (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j7g5p (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  modules:
    Type:          HostPath (bare host directory volume)
    Path:          /lib/modules
    HostPathType:  
  config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      crpd-config
    Optional:  false
  syslog:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-j7g5p:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

```
pradeep@paragon:~$ sudo -i
root@paragon:~# pf
pf2afm         pfbtopfa       pf-cmgd        pf-crpd        pf-debugutils  pf-redis       pftp           
```

## CRPD

```
root@paragon:~# pf-crpd 
root@bmp-77f4788c94-g9zdg> show configuration 
## Last commit: 2023-08-23 06:00:30 UTC by root
version 20220309.081619_builder.r1244936;
groups {
    extra {
        protocols {
            bgp {
                group northstar {
                    neighbor 192.168.1.2;
                }
            }
        }
    }
}
apply-groups extra;
routing-options {
    autonomous-system 64512;
    bmp {
        local-address 127.0.0.1;
        local-port 10003;
        connection-mode passive;
        monitor enable;
        station localhost {
            station-address 127.0.0.1;
        }
    }
    static {
        route 0.0.0.0/0 next-hop 169.254.1.1;
    }
}
protocols {
    bgp {
        group northstar {
            type internal;
            family traffic-engineering {
                unicast;
            }
            allow 0.0.0.0/0;
        }
    }
}

root@bmp-77f4788c94-g9zdg> show route  

inet.0: 1 destinations, 2 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.244.254.137/32  *[Direct/0] 3d 23:11:11
                    >  via eth0
                    [Local/0] 3d 23:11:11
                       Local via eth0

inet6.0: 11 destinations, 12 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

::/96              *[Direct/0] 3d 23:11:11
                    >  via sit0
                    [Direct/0] 3d 23:11:11
                    >  via sit0
::10.244.254.137/128
                   *[Local/0] 3d 23:11:11
                       Local via sit0
::127.0.0.1/128    *[Local/0] 3d 23:11:11
                       Local via sit0
fe80::1/128        *[Direct/0] 3d 23:11:11
                    >  via lo.0
fe80::af4:fe89/128 *[Local/0] 3d 23:11:11
                       Local via gre0
fe80::7f00:1/128   *[Local/0] 3d 23:11:11
                       Local via gre0
fe80::4c1e:39ff:fe63:24c6/128
                   *[Local/0] 3d 23:11:11
                       Local via ip6tnl0
fe80::60e8:19ff:fed4:d30e/128
                   *[Local/0] 3d 23:11:10
                       Local via irb
fe80::d8cd:91ff:fe83:d0dd/128
                   *[Local/0] 3d 23:11:11
                       Local via lsi
fe80::f8a5:1aff:fe98:2e91/128
                   *[Local/0] 3d 23:11:11
                       Local via eth0
ff02::2/128        *[INET6/0] 3d 23:11:11
                       MultiRecv

lsdist.0: 26 destinations, 26 routes (0 active, 0 holddown, 26 hidden)

root@bmp-77f4788c94-g9zdg> show route hidden 

inet.0: 1 destinations, 2 routes (1 active, 0 holddown, 0 hidden)

inet6.0: 11 destinations, 12 routes (11 active, 0 holddown, 0 hidden)

lsdist.0: 26 destinations, 26 routes (0 active, 0 holddown, 26 hidden)
+ = Active Route, - = Last Active, * = Both

NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.1 OSPF:0 }/1536              
                    [BGP/170] 15:11:11, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.2 OSPF:0 }/1536              
                    [BGP/170] 15:11:11, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 OSPF:0 }/1536              
                    [BGP/170] 15:11:11, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 OSPF:0 }/1536              
                    [BGP/170] 15:11:11, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 OSPF:0 }/1536              
                    [BGP/170] 15:11:13, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
NODE { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 OSPF:0 }/1536              
                    [BGP/170] 15:11:12, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.1 }.{ IPv4:172.22.101.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.101.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:08, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.1 }.{ IPv4:172.22.102.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.102.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:35, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.2 }.{ IPv4:172.22.110.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.110.2 } OSPF:0 }/1536              
                    [BGP/170] 14:55:46, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.2 }.{ IPv4:172.22.111.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.111.2 } OSPF:0 }/1536              
                    [BGP/170] 02:12:29, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.101.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.1 }.{ IPv4:172.22.101.1 } OSPF:0 }/1536              
                    [BGP/170] 02:11:31, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.103.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.103.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:26, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.105.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.105.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:11, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.107.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.107.2 } OSPF:0 }/1536              
                    [BGP/170] 02:12:23, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.102.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.1 }.{ IPv4:172.22.102.1 } OSPF:0 }/1536              
                    [BGP/170] 02:11:35, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.103.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.103.1 } OSPF:0 }/1536              
                    [BGP/170] 02:13:03, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.106.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.106.2 } OSPF:0 }/1536              
                    [BGP/170] 02:12:20, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.108.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.108.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:25, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.105.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.105.1 } OSPF:0 }/1536              
                    [BGP/170] 02:13:07, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.108.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.108.1 } OSPF:0 }/1536              
                    [BGP/170] 02:13:51, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.109.1 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.109.2 } OSPF:0 }/1536              
                    [BGP/170] 02:11:29, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.110.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.2 }.{ IPv4:172.22.110.1 } OSPF:0 }/1536              
                    [BGP/170] 02:11:09, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.106.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.4 }.{ IPv4:172.22.106.1 } OSPF:0 }/1536              
                    [BGP/170] 02:12:18, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.107.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.3 }.{ IPv4:172.22.107.1 } OSPF:0 }/1536              
                    [BGP/170] 02:12:24, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.109.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.5 }.{ IPv4:172.22.109.1 } OSPF:0 }/1536              
                    [BGP/170] 14:46:55, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable
LINK { Local { AS:64512 Area:0.0.0.0 IPv4:172.20.20.6 }.{ IPv4:172.22.111.2 } Remote { AS:64512 Area:0.0.0.0 IPv4:172.20.20.2 }.{ IPv4:172.22.111.1 } OSPF:0 }/1536              
                    [BGP/170] 02:11:27, localpref 100, from 192.168.1.2
                      AS path: I, validation-state: unverified
                       Unusable

root@bmp-77f4788c94-g9zdg> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
lsdist.0             
                      26          0          0          0          0          0
inet.0               
                       0          0          0          0          0          0
inet6.0              
                       0          0          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
192.168.1.2           64512       2714       2556       0       0    19:09:12 Establ
  lsdist.0: 0/26/26/0

root@bmp-77f4788c94-g9zdg> exit 

```

## API Resources

```
root@paragon:~# kubectl api-resources 
NAME                              SHORTNAMES           APIVERSION                                  NAMESPACED   KIND
bindings                                               v1                                          true         Binding
componentstatuses                 cs                   v1                                          false        ComponentStatus
configmaps                        cm                   v1                                          true         ConfigMap
endpoints                         ep                   v1                                          true         Endpoints
events                            ev                   v1                                          true         Event
limitranges                       limits               v1                                          true         LimitRange
namespaces                        ns                   v1                                          false        Namespace
nodes                             no                   v1                                          false        Node
persistentvolumeclaims            pvc                  v1                                          true         PersistentVolumeClaim
persistentvolumes                 pv                   v1                                          false        PersistentVolume
pods                              po                   v1                                          true         Pod
podtemplates                                           v1                                          true         PodTemplate
replicationcontrollers            rc                   v1                                          true         ReplicationController
resourcequotas                    quota                v1                                          true         ResourceQuota
secrets                                                v1                                          true         Secret
serviceaccounts                   sa                   v1                                          true         ServiceAccount
services                          svc                  v1                                          true         Service
operatorconfigurations            opconfig             acid.zalan.do/v1                            true         OperatorConfiguration
postgresqls                       pg                   acid.zalan.do/v1                            true         postgresql
postgresteams                     pgteam               acid.zalan.do/v1                            true         PostgresTeam
challenges                                             acme.cert-manager.io/v1                     true         Challenge
orders                                                 acme.cert-manager.io/v1                     true         Order
mutatingwebhookconfigurations                          admissionregistration.k8s.io/v1             false        MutatingWebhookConfiguration
validatingwebhookconfigurations                        admissionregistration.k8s.io/v1             false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds             apiextensions.k8s.io/v1                     false        CustomResourceDefinition
apiservices                                            apiregistration.k8s.io/v1                   false        APIService
controllerrevisions                                    apps/v1                                     true         ControllerRevision
daemonsets                        ds                   apps/v1                                     true         DaemonSet
deployments                       deploy               apps/v1                                     true         Deployment
replicasets                       rs                   apps/v1                                     true         ReplicaSet
statefulsets                      sts                  apps/v1                                     true         StatefulSet
clusterworkflowtemplates          clusterwftmpl,cwft   argoproj.io/v1alpha1                        false        ClusterWorkflowTemplate
cronworkflows                     cwf,cronwf           argoproj.io/v1alpha1                        true         CronWorkflow
workflowartifactgctasks           wfat                 argoproj.io/v1alpha1                        true         WorkflowArtifactGCTask
workfloweventbindings             wfeb                 argoproj.io/v1alpha1                        true         WorkflowEventBinding
workflows                         wf                   argoproj.io/v1alpha1                        true         Workflow
workflowtaskresults                                    argoproj.io/v1alpha1                        true         WorkflowTaskResult
workflowtasksets                  wfts                 argoproj.io/v1alpha1                        true         WorkflowTaskSet
workflowtemplates                 wftmpl               argoproj.io/v1alpha1                        true         WorkflowTemplate
tokenreviews                                           authentication.k8s.io/v1                    false        TokenReview
localsubjectaccessreviews                              authorization.k8s.io/v1                     true         LocalSubjectAccessReview
selfsubjectaccessreviews                               authorization.k8s.io/v1                     false        SelfSubjectAccessReview
selfsubjectrulesreviews                                authorization.k8s.io/v1                     false        SelfSubjectRulesReview
subjectaccessreviews                                   authorization.k8s.io/v1                     false        SubjectAccessReview
horizontalpodautoscalers          hpa                  autoscaling/v1                              true         HorizontalPodAutoscaler
cronjobs                          cj                   batch/v1                                    true         CronJob
jobs                                                   batch/v1                                    true         Job
cephblockpools                                         ceph.rook.io/v1                             true         CephBlockPool
cephclients                                            ceph.rook.io/v1                             true         CephClient
cephclusters                                           ceph.rook.io/v1                             true         CephCluster
cephfilesystemmirrors                                  ceph.rook.io/v1                             true         CephFilesystemMirror
cephfilesystems                                        ceph.rook.io/v1                             true         CephFilesystem
cephnfses                         nfs                  ceph.rook.io/v1                             true         CephNFS
cephobjectrealms                                       ceph.rook.io/v1                             true         CephObjectRealm
cephobjectstores                                       ceph.rook.io/v1                             true         CephObjectStore
cephobjectstoreusers              rcou,objectuser      ceph.rook.io/v1                             true         CephObjectStoreUser
cephobjectzonegroups                                   ceph.rook.io/v1                             true         CephObjectZoneGroup
cephobjectzones                                        ceph.rook.io/v1                             true         CephObjectZone
cephrbdmirrors                                         ceph.rook.io/v1                             true         CephRBDMirror
certificaterequests               cr,crs               cert-manager.io/v1                          true         CertificateRequest
certificates                      cert,certs           cert-manager.io/v1                          true         Certificate
clusterissuers                                         cert-manager.io/v1                          false        ClusterIssuer
issuers                                                cert-manager.io/v1                          true         Issuer
certificatesigningrequests        csr                  certificates.k8s.io/v1                      false        CertificateSigningRequest
leases                                                 coordination.k8s.io/v1                      true         Lease
bgpconfigurations                                      crd.projectcalico.org/v1                    false        BGPConfiguration
bgppeers                                               crd.projectcalico.org/v1                    false        BGPPeer
blockaffinities                                        crd.projectcalico.org/v1                    false        BlockAffinity
clusterinformations                                    crd.projectcalico.org/v1                    false        ClusterInformation
felixconfigurations                                    crd.projectcalico.org/v1                    false        FelixConfiguration
globalnetworkpolicies                                  crd.projectcalico.org/v1                    false        GlobalNetworkPolicy
globalnetworksets                                      crd.projectcalico.org/v1                    false        GlobalNetworkSet
hostendpoints                                          crd.projectcalico.org/v1                    false        HostEndpoint
ipamblocks                                             crd.projectcalico.org/v1                    false        IPAMBlock
ipamconfigs                                            crd.projectcalico.org/v1                    false        IPAMConfig
ipamhandles                                            crd.projectcalico.org/v1                    false        IPAMHandle
ippools                                                crd.projectcalico.org/v1                    false        IPPool
kubecontrollersconfigurations                          crd.projectcalico.org/v1                    false        KubeControllersConfiguration
networkpolicies                                        crd.projectcalico.org/v1                    true         NetworkPolicy
networksets                                            crd.projectcalico.org/v1                    true         NetworkSet
redisfailovers                                         databases.spotahome.com/v1                  true         RedisFailover
endpointslices                                         discovery.k8s.io/v1                         true         EndpointSlice
events                            ev                   events.k8s.io/v1                            true         Event
ingresses                         ing                  extensions/v1beta1                          true         Ingress
flowschemas                                            flowcontrol.apiserver.k8s.io/v1beta1        false        FlowSchema
prioritylevelconfigurations                            flowcontrol.apiserver.k8s.io/v1beta1        false        PriorityLevelConfiguration
authservices                                           getambassador.io/v2                         true         AuthService
consulresolvers                                        getambassador.io/v2                         true         ConsulResolver
devportals                                             getambassador.io/v2                         true         DevPortal
filterpolicies                    fp                   getambassador.io/v2                         true         FilterPolicy
filters                           fil                  getambassador.io/v2                         true         Filter
hosts                                                  getambassador.io/v2                         true         Host
kubernetesendpointresolvers                            getambassador.io/v2                         true         KubernetesEndpointResolver
kubernetesserviceresolvers                             getambassador.io/v2                         true         KubernetesServiceResolver
logservices                                            getambassador.io/v2                         true         LogService
mappings                                               getambassador.io/v2                         true         Mapping
modules                                                getambassador.io/v2                         true         Module
projectcontrollers                                     getambassador.io/v2                         true         ProjectController
projectrevisions                                       getambassador.io/v2                         true         ProjectRevision
projects                                               getambassador.io/v2                         true         Project
ratelimits                        rl                   getambassador.io/v2                         true         RateLimit
ratelimitservices                                      getambassador.io/v2                         true         RateLimitService
tcpmappings                                            getambassador.io/v2                         true         TCPMapping
tlscontexts                                            getambassador.io/v2                         true         TLSContext
tracingservices                                        getambassador.io/v2                         true         TracingService
workflowdefinitions               jwd                  jobmgr.juniper.net/v1                       false        WorkflowDefinition
workflowplugins                   jwp                  jobmgr.juniper.net/v1                       false        WorkflowPlugin
nodes                                                  metrics.k8s.io/v1beta1                      false        NodeMetrics
pods                                                   metrics.k8s.io/v1beta1                      true         PodMetrics
ingressclasses                                         networking.k8s.io/v1                        false        IngressClass
ingresses                         ing                  networking.k8s.io/v1                        true         Ingress
networkpolicies                   netpol               networking.k8s.io/v1                        true         NetworkPolicy
runtimeclasses                                         node.k8s.io/v1                              false        RuntimeClass
objectbucketclaims                obc,obcs             objectbucket.io/v1alpha1                    true         ObjectBucketClaim
objectbuckets                     ob,obs               objectbucket.io/v1alpha1                    false        ObjectBucket
poddisruptionbudgets              pdb                  policy/v1                                   true         PodDisruptionBudget
podsecuritypolicies               psp                  policy/v1beta1                              false        PodSecurityPolicy
clusterrolebindings                                    rbac.authorization.k8s.io/v1                false        ClusterRoleBinding
clusterroles                                           rbac.authorization.k8s.io/v1                false        ClusterRole
rolebindings                                           rbac.authorization.k8s.io/v1                true         RoleBinding
roles                                                  rbac.authorization.k8s.io/v1                true         Role
capabilitymappings                jcm                  rbac.juniper.net/v1                         false        CapabilityMapping
rolebindings                      jrb                  rbac.juniper.net/v1                         false        RoleBinding
roles                             jr                   rbac.juniper.net/v1                         false        Role
volumereplicationclasses          vrc                  replication.storage.openshift.io/v1alpha1   false        VolumeReplicationClass
volumereplications                vr                   replication.storage.openshift.io/v1alpha1   true         VolumeReplication
volumes                           rv                   rook.io/v1alpha2                            true         Volume
priorityclasses                   pc                   scheduling.k8s.io/v1                        false        PriorityClass
csidrivers                                             storage.k8s.io/v1                           false        CSIDriver
csinodes                                               storage.k8s.io/v1                           false        CSINode
csistoragecapacities                                   storage.k8s.io/v1beta1                      true         CSIStorageCapacity
storageclasses                    sc                   storage.k8s.io/v1                           false        StorageClass
volumeattachments                                      storage.k8s.io/v1                           false        VolumeAttachment
root@paragon:~# 
```

