---
layout: single
title:  "Kubernetes Certificates API"
date:   2022-02-17 10:55:04 +0530
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
# Kubernetes Certificates API


####  Certificates API

Let us create a new user called `pradeep` and generate a key pair. Once the keypair is generated, created a certificate signing request to be submitted to the Kubernetes Cluster CA.

```shell
$ sudo adduser -G wheel pradeep
Changing password for pradeep
New password:
Bad password: similar to username
Retype password:
passwd: password for pradeep changed by root
$ su - pradeep
Password:
$ mkdir .certs
$ cd .certs/
$ openssl genrsa -out pradeep.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
........+++++
......................................................................................................+++++
e is 65537 (0x010001)
$ openssl req -new -key pradeep.key -out pradeep.csr -subj "/CN=pradeep"
$ exit
logout
```
We need to convert this CSR file to base64 format and trim newline character, which needs to be passed in the CSR request.

```shell
$ ls
pradeep.csr  pradeep.key
$ cat pradeep.csr | base64 | tr -d "\n"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hjSEpoWkdWbGNEQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNVzRxa29NK0ZDRUxwa2oySUtiTnZjeGRNNGxnMS8xMnlXZjA1NHpPMVdrCmJEQTlrQnpFdEtRb0ptR1ZZRXVLLzZHT0l0TE90WU10Z1BscTJyblJZMHF5c1VrTTMwOUo1R0RUblhpbDJsZUUKU0pjL2tuWkYxRlB3SFVxeWxGdHg5OUZwa3N3OWI1YTNSVmQweXp6K1JyOVhGcFlTR2d5WjgrcUQ5WHIvOE9jWgo0NW1PQnpoaGpKeDRYRW9NSC94aGpIK1FPc1pkaHY2Q2wrdDdZVmgrU2dzdExpVEQ3YTZ0ZXh2NUVVdFNvMllxCkJabXpEcGUzUnVMTlZYd2Z1bnVQdlNUTXdvYTZDQ3h4VEJYK2tZdTJQa2NRcVBTeTNzZlR6Y2xrbWVNbXVKVUQKdmZZTDFxNnFCa3c0a2hFckxXVVVLcGVMckdvMi9vZ01TcXJNVXF0M0k4Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUF1aHV3NWIvV2l0cms2b2YrNDAwaWxqQXEyY3dmdjdRdDNaaWZTUVVzOVEzMVdnM2d3CnZZNUd4djFDV2xtd002cWttU0lXRC9MY0lzcTJFanNJbkNSNzFhbXBKaEhHYWFOZGFrVlIxNlcyU2lxd040NjUKcGtpYmlpVEVqQUJZQ1pOWnk1dG5KVkpKSVFLM1lUM3lNK2EwVXhOQXFTTHFkU3FIV2d3am5MVktCVVBMWFBrdQpMa1VqejBWTEFXRkI3TXN4OXR2OTJQT1NZNFVULzJSN0xPdC9yZzFYeklyTmxtTHRKUUJYMUo1YkpRb0NrU1lPCmI1eklDTXZrS2FpVlloZUZ6Qi8vWitNcnZiWHgwREdkYTBWTnNKT29QeDF5VzRpRTRTQVY1SE9pbDlpd0Q2emkKdjQyblZlb2R5WjdQNlp3M241OG5tTmRWUXk1dmVGYlI5UEZCCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=$
$
```
Create an YAML manifest for the CertificateSigningRequest API resource.
```shell
pradeep@learnk8s$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: pradeep
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hjSEpoWkdWbGNEQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNVzRxa29NK0ZDRUxwa2oySUtiTnZjeGRNNGxnMS8xMnlXZjA1NHpPMVdrCmJEQTlrQnpFdEtRb0ptR1ZZRXVLLzZHT0l0TE90WU10Z1BscTJyblJZMHF5c1VrTTMwOUo1R0RUblhpbDJsZUUKU0pjL2tuWkYxRlB3SFVxeWxGdHg5OUZwa3N3OWI1YTNSVmQweXp6K1JyOVhGcFlTR2d5WjgrcUQ5WHIvOE9jWgo0NW1PQnpoaGpKeDRYRW9NSC94aGpIK1FPc1pkaHY2Q2wrdDdZVmgrU2dzdExpVEQ3YTZ0ZXh2NUVVdFNvMllxCkJabXpEcGUzUnVMTlZYd2Z1bnVQdlNUTXdvYTZDQ3h4VEJYK2tZdTJQa2NRcVBTeTNzZlR6Y2xrbWVNbXVKVUQKdmZZTDFxNnFCa3c0a2hFckxXVVVLcGVMckdvMi9vZ01TcXJNVXF0M0k4Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUF1aHV3NWIvV2l0cms2b2YrNDAwaWxqQXEyY3dmdjdRdDNaaWZTUVVzOVEzMVdnM2d3CnZZNUd4djFDV2xtd002cWttU0lXRC9MY0lzcTJFanNJbkNSNzFhbXBKaEhHYWFOZGFrVlIxNlcyU2lxd040NjUKcGtpYmlpVEVqQUJZQ1pOWnk1dG5KVkpKSVFLM1lUM3lNK2EwVXhOQXFTTHFkU3FIV2d3am5MVktCVVBMWFBrdQpMa1VqejBWTEFXRkI3TXN4OXR2OTJQT1NZNFVULzJSN0xPdC9yZzFYeklyTmxtTHRKUUJYMUo1YkpRb0NrU1lPCmI1eklDTXZrS2FpVlloZUZ6Qi8vWitNcnZiWHgwREdkYTBWTnNKT29QeDF5VzRpRTRTQVY1SE9pbDlpd0Q2emkKdjQyblZlb2R5WjdQNlp3M241OG5tTmRWUXk1dmVGYlI5UEZCCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  expirationSeconds: 86400  # one day
  usages:
  - client auth
  signerName: kubernetes.io/kube-apiserver-client
EOF
certificatesigningrequest.certificates.k8s.io/pradeep created
```
Verify that the new request is in Pending state.

```shell
pradeep@learnk8s$ kubectl get csr
NAME      AGE   SIGNERNAME                            REQUESTOR       REQUESTEDDURATION   CONDITION
pradeep   4s    kubernetes.io/kube-apiserver-client   minikube-user   24h                 Pending
```

As we know this is a valid request, we can go ahead and approve this.
```shell
pradeep@learnk8s$ kubectl certificate approve pradeep
certificatesigningrequest.certificates.k8s.io/pradeep approved
```
```shell
pradeep@learnk8s$ kubectl get csr
NAME      AGE   SIGNERNAME                            REQUESTOR       REQUESTEDDURATION   CONDITION
pradeep   66s   kubernetes.io/kube-apiserver-client   minikube-user   24h                 Approved,Issued
```
```shell
pradeep@learnk8s$ kubectl describe csr
Name:         pradeep
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"pradeep"},"spec":{"expirationSeconds":86400,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hjSEpoWkdWbGNEQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNVzRxa29NK0ZDRUxwa2oySUtiTnZjeGRNNGxnMS8xMnlXZjA1NHpPMVdrCmJEQTlrQnpFdEtRb0ptR1ZZRXVLLzZHT0l0TE90WU10Z1BscTJyblJZMHF5c1VrTTMwOUo1R0RUblhpbDJsZUUKU0pjL2tuWkYxRlB3SFVxeWxGdHg5OUZwa3N3OWI1YTNSVmQweXp6K1JyOVhGcFlTR2d5WjgrcUQ5WHIvOE9jWgo0NW1PQnpoaGpKeDRYRW9NSC94aGpIK1FPc1pkaHY2Q2wrdDdZVmgrU2dzdExpVEQ3YTZ0ZXh2NUVVdFNvMllxCkJabXpEcGUzUnVMTlZYd2Z1bnVQdlNUTXdvYTZDQ3h4VEJYK2tZdTJQa2NRcVBTeTNzZlR6Y2xrbWVNbXVKVUQKdmZZTDFxNnFCa3c0a2hFckxXVVVLcGVMckdvMi9vZ01TcXJNVXF0M0k4Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUF1aHV3NWIvV2l0cms2b2YrNDAwaWxqQXEyY3dmdjdRdDNaaWZTUVVzOVEzMVdnM2d3CnZZNUd4djFDV2xtd002cWttU0lXRC9MY0lzcTJFanNJbkNSNzFhbXBKaEhHYWFOZGFrVlIxNlcyU2lxd040NjUKcGtpYmlpVEVqQUJZQ1pOWnk1dG5KVkpKSVFLM1lUM3lNK2EwVXhOQXFTTHFkU3FIV2d3am5MVktCVVBMWFBrdQpMa1VqejBWTEFXRkI3TXN4OXR2OTJQT1NZNFVULzJSN0xPdC9yZzFYeklyTmxtTHRKUUJYMUo1YkpRb0NrU1lPCmI1eklDTXZrS2FpVlloZUZ6Qi8vWitNcnZiWHgwREdkYTBWTnNKT29QeDF5VzRpRTRTQVY1SE9pbDlpd0Q2emkKdjQyblZlb2R5WjdQNlp3M241OG5tTmRWUXk1dmVGYlI5UEZCCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}

CreationTimestamp:   Thu, 17 Feb 2022 11:17:16 +0530
Requesting User:     minikube-user
Signer:              kubernetes.io/kube-apiserver-client
Requested Duration:  24h
Status:              Approved,Issued
Subject:
         Common Name:    pradeep
         Serial Number:
Events:  <none>
```
