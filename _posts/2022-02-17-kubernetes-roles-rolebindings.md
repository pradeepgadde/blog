---
layout: single
title:  "Kubernetes Roles and RoleBindings"
date:   2022-02-17 10:55:04 +0530
categories: Kubernetes
tags: minikube
author: "Pradeep Gadde"
classes: wide

show_date: true
header:
  teaser: /assets/images/role-128.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
  
sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---
# Kubernetes Roles and RoleBindings



We can see that the current user is `k8s`, how can we change this to some other user, like `pradeep` user?

For that we need the signed certificate for that user first. We can retrieve the signed/issued certificate using this.
Pay attention to the Status section, there is the certificate.

```shell
pradeep@learnk8s$ kubectl get csr/pradeep -o yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"pradeep"},"spec":{"expirationSeconds":86400,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hjSEpoWkdWbGNEQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNVzRxa29NK0ZDRUxwa2oySUtiTnZjeGRNNGxnMS8xMnlXZjA1NHpPMVdrCmJEQTlrQnpFdEtRb0ptR1ZZRXVLLzZHT0l0TE90WU10Z1BscTJyblJZMHF5c1VrTTMwOUo1R0RUblhpbDJsZUUKU0pjL2tuWkYxRlB3SFVxeWxGdHg5OUZwa3N3OWI1YTNSVmQweXp6K1JyOVhGcFlTR2d5WjgrcUQ5WHIvOE9jWgo0NW1PQnpoaGpKeDRYRW9NSC94aGpIK1FPc1pkaHY2Q2wrdDdZVmgrU2dzdExpVEQ3YTZ0ZXh2NUVVdFNvMllxCkJabXpEcGUzUnVMTlZYd2Z1bnVQdlNUTXdvYTZDQ3h4VEJYK2tZdTJQa2NRcVBTeTNzZlR6Y2xrbWVNbXVKVUQKdmZZTDFxNnFCa3c0a2hFckxXVVVLcGVMckdvMi9vZ01TcXJNVXF0M0k4Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUF1aHV3NWIvV2l0cms2b2YrNDAwaWxqQXEyY3dmdjdRdDNaaWZTUVVzOVEzMVdnM2d3CnZZNUd4djFDV2xtd002cWttU0lXRC9MY0lzcTJFanNJbkNSNzFhbXBKaEhHYWFOZGFrVlIxNlcyU2lxd040NjUKcGtpYmlpVEVqQUJZQ1pOWnk1dG5KVkpKSVFLM1lUM3lNK2EwVXhOQXFTTHFkU3FIV2d3am5MVktCVVBMWFBrdQpMa1VqejBWTEFXRkI3TXN4OXR2OTJQT1NZNFVULzJSN0xPdC9yZzFYeklyTmxtTHRKUUJYMUo1YkpRb0NrU1lPCmI1eklDTXZrS2FpVlloZUZ6Qi8vWitNcnZiWHgwREdkYTBWTnNKT29QeDF5VzRpRTRTQVY1SE9pbDlpd0Q2emkKdjQyblZlb2R5WjdQNlp3M241OG5tTmRWUXk1dmVGYlI5UEZCCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}
  creationTimestamp: "2022-02-17T05:47:16Z"
  name: pradeep
  resourceVersion: "63056"
  uid: 585f7670-134c-4062-a649-442054314407
spec:
  expirationSeconds: 86400
  groups:
  - system:masters
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hjSEpoWkdWbGNEQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNVzRxa29NK0ZDRUxwa2oySUtiTnZjeGRNNGxnMS8xMnlXZjA1NHpPMVdrCmJEQTlrQnpFdEtRb0ptR1ZZRXVLLzZHT0l0TE90WU10Z1BscTJyblJZMHF5c1VrTTMwOUo1R0RUblhpbDJsZUUKU0pjL2tuWkYxRlB3SFVxeWxGdHg5OUZwa3N3OWI1YTNSVmQweXp6K1JyOVhGcFlTR2d5WjgrcUQ5WHIvOE9jWgo0NW1PQnpoaGpKeDRYRW9NSC94aGpIK1FPc1pkaHY2Q2wrdDdZVmgrU2dzdExpVEQ3YTZ0ZXh2NUVVdFNvMllxCkJabXpEcGUzUnVMTlZYd2Z1bnVQdlNUTXdvYTZDQ3h4VEJYK2tZdTJQa2NRcVBTeTNzZlR6Y2xrbWVNbXVKVUQKdmZZTDFxNnFCa3c0a2hFckxXVVVLcGVMckdvMi9vZ01TcXJNVXF0M0k4Y0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUF1aHV3NWIvV2l0cms2b2YrNDAwaWxqQXEyY3dmdjdRdDNaaWZTUVVzOVEzMVdnM2d3CnZZNUd4djFDV2xtd002cWttU0lXRC9MY0lzcTJFanNJbkNSNzFhbXBKaEhHYWFOZGFrVlIxNlcyU2lxd040NjUKcGtpYmlpVEVqQUJZQ1pOWnk1dG5KVkpKSVFLM1lUM3lNK2EwVXhOQXFTTHFkU3FIV2d3am5MVktCVVBMWFBrdQpMa1VqejBWTEFXRkI3TXN4OXR2OTJQT1NZNFVULzJSN0xPdC9yZzFYeklyTmxtTHRKUUJYMUo1YkpRb0NrU1lPCmI1eklDTXZrS2FpVlloZUZ6Qi8vWitNcnZiWHgwREdkYTBWTnNKT29QeDF5VzRpRTRTQVY1SE9pbDlpd0Q2emkKdjQyblZlb2R5WjdQNlp3M241OG5tTmRWUXk1dmVGYlI5UEZCCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
  username: minikube-user
status:
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMrRENDQWVDZ0F3SUJBZ0lSQUlHNEdtUk5tdmczZENXZHhFcTBIUEV3RFFZSktvWklodmNOQVFFTEJRQXcKRlRFVE1CRUdBMVVFQXhNS2JXbHVhV3QxWW1WRFFUQWVGdzB5TWpBeU1UY3dOVFF6TWpCYUZ3MHlNakF5TVRndwpOVFF6TWpCYU1CSXhFREFPQmdOVkJBTVRCM0J5WVdSbFpYQXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCCkR3QXdnZ0VLQW9JQkFRREZ1S3BLRFBoUWhDNlpJOWlDbXpiM01YVE9KWU5mOWRzbG45T2VNenRWcEd3d1BaQWMKeExTa0tDWmhsV0JMaXYraGppTFN6cldETFlENWF0cTUwV05Lc3JGSkROOVBTZVJnMDUxNHBkcFhoRWlYUDVKMgpSZFJUOEIxS3NwUmJjZmZSYVpMTVBXK1d0MFZYZE1zOC9rYS9WeGFXRWhvTW1mUHFnL1Y2Ly9EbkdlT1pqZ2M0CllZeWNlRnhLREIvOFlZeC9rRHJHWFliK2dwZnJlMkZZZmtvTExTNGt3KzJ1clhzYitSRkxVcU5tS2dXWnN3NlgKdDBiaXpWVjhIN3A3ajcwa3pNS0d1Z2dzY1V3Vi9wR0x0ajVIRUtqMHN0N0gwODNKWkpuakpyaVZBNzMyQzlhdQpxZ1pNT0pJUkt5MWxGQ3FYaTZ4cU52NklERXFxekZLcmR5UEhBZ01CQUFHalJqQkVNQk1HQTFVZEpRUU1NQW9HCkNDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGpCQmd3Rm9BVWkvb3hwNHQyM3NiMVBjQy8KSlFVZEJYaTVna0F3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUp3R1dVR1IzRVd3bnc2cVdoUVJRTHY1aUFuOQpESmVhb3o1M25VVmhuV0tzRWdETXhzcTgvcHRVK0NJalgvR3NXS2hSaE1nU3Q0ZHR1cEswMHROVnB3cGRQTExQCm5kWHRzZmhmQmRPV1hNWDVBSmNXNTQ2Sk02eVhkQ1VXbnBmSE5LTTJ5bnRqb0FPSXp4azNhUGcxeDNuckdCZVkKQzZ6UmhRdWRYTitwdFFNK0lRa0pYVzF4TWhSeVRoNHU2bXI3c2RTTGdQZ0dsaXZGNGE2Ukx3YWttOWdoNWJtOApIUkxYZWpuTVo0TFVUOExYWmgyR0tRenRrR1QwMkgxanNDMTZpYnJpbmNFSFpxUXVvd0Z3TUMxSUdhS1l0d1BkCnMwVFQxcFZBR3N1VzRlYUpPT3B2MmtpcVVITkExLzBrb1IwYnpscFhObFZIZzVySlY1dFBtWXp4YjRjPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  conditions:
  - lastTransitionTime: "2022-02-17T05:48:20Z"
    lastUpdateTime: "2022-02-17T05:48:20Z"
    message: This CSR was approved by kubectl certificate approve.
    reason: KubectlApprove
    status: "True"
    type: Approved
```

The certificate value is in Base64-encoded format under status.certificate.

Let us export the issued certificate from the CertificateSigningRequest.

```shell
pradeep@learnk8s$ echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMrRENDQWVDZ0F3SUJBZ0lSQUlHNEdtUk5tdmczZENXZHhFcTBIUEV3RFFZSktvWklodmNOQVFFTEJRQXcKRlRFVE1CRUdBMVVFQXhNS2JXbHVhV3QxWW1WRFFUQWVGdzB5TWpBeU1UY3dOVFF6TWpCYUZ3MHlNakF5TVRndwpOVFF6TWpCYU1CSXhFREFPQmdOVkJBTVRCM0J5WVdSbFpYQXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCCkR3QXdnZ0VLQW9JQkFRREZ1S3BLRFBoUWhDNlpJOWlDbXpiM01YVE9KWU5mOWRzbG45T2VNenRWcEd3d1BaQWMKeExTa0tDWmhsV0JMaXYraGppTFN6cldETFlENWF0cTUwV05Lc3JGSkROOVBTZVJnMDUxNHBkcFhoRWlYUDVKMgpSZFJUOEIxS3NwUmJjZmZSYVpMTVBXK1d0MFZYZE1zOC9rYS9WeGFXRWhvTW1mUHFnL1Y2Ly9EbkdlT1pqZ2M0CllZeWNlRnhLREIvOFlZeC9rRHJHWFliK2dwZnJlMkZZZmtvTExTNGt3KzJ1clhzYitSRkxVcU5tS2dXWnN3NlgKdDBiaXpWVjhIN3A3ajcwa3pNS0d1Z2dzY1V3Vi9wR0x0ajVIRUtqMHN0N0gwODNKWkpuakpyaVZBNzMyQzlhdQpxZ1pNT0pJUkt5MWxGQ3FYaTZ4cU52NklERXFxekZLcmR5UEhBZ01CQUFHalJqQkVNQk1HQTFVZEpRUU1NQW9HCkNDc0dBUVVGQndNQ01Bd0dBMVVkRXdFQi93UUNNQUF3SHdZRFZSMGpCQmd3Rm9BVWkvb3hwNHQyM3NiMVBjQy8KSlFVZEJYaTVna0F3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUp3R1dVR1IzRVd3bnc2cVdoUVJRTHY1aUFuOQpESmVhb3o1M25VVmhuV0tzRWdETXhzcTgvcHRVK0NJalgvR3NXS2hSaE1nU3Q0ZHR1cEswMHROVnB3cGRQTExQCm5kWHRzZmhmQmRPV1hNWDVBSmNXNTQ2Sk02eVhkQ1VXbnBmSE5LTTJ5bnRqb0FPSXp4azNhUGcxeDNuckdCZVkKQzZ6UmhRdWRYTitwdFFNK0lRa0pYVzF4TWhSeVRoNHU2bXI3c2RTTGdQZ0dsaXZGNGE2Ukx3YWttOWdoNWJtOApIUkxYZWpuTVo0TFVUOExYWmgyR0tRenRrR1QwMkgxanNDMTZpYnJpbmNFSFpxUXVvd0Z3TUMxSUdhS1l0d1BkCnMwVFQxcFZBR3N1VzRlYUpPT3B2MmtpcVVITkExLzBrb1IwYnpscFhObFZIZzVySlY1dFBtWXp4YjRjPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==" | base64 -d > pradeep.crt
```
Just to confirm, let us view the certificate.

```shell
pradeep@learnk8s$ cat pradeep.crt
-----BEGIN CERTIFICATE-----
MIIC+DCCAeCgAwIBAgIRAIG4GmRNmvg3dCWdxEq0HPEwDQYJKoZIhvcNAQELBQAw
FTETMBEGA1UEAxMKbWluaWt1YmVDQTAeFw0yMjAyMTcwNTQzMjBaFw0yMjAyMTgw
NTQzMjBaMBIxEDAOBgNVBAMTB3ByYWRlZXAwggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQDFuKpKDPhQhC6ZI9iCmzb3MXTOJYNf9dsln9OeMztVpGwwPZAc
xLSkKCZhlWBLiv+hjiLSzrWDLYD5atq50WNKsrFJDN9PSeRg0514pdpXhEiXP5J2
RdRT8B1KspRbcffRaZLMPW+Wt0VXdMs8/ka/VxaWEhoMmfPqg/V6//DnGeOZjgc4
YYyceFxKDB/8YYx/kDrGXYb+gpfre2FYfkoLLS4kw+2urXsb+RFLUqNmKgWZsw6X
t0bizVV8H7p7j70kzMKGuggscUwV/pGLtj5HEKj0st7H083JZJnjJriVA732C9au
qgZMOJIRKy1lFCqXi6xqNv6IDEqqzFKrdyPHAgMBAAGjRjBEMBMGA1UdJQQMMAoG
CCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHwYDVR0jBBgwFoAUi/oxp4t23sb1PcC/
JQUdBXi5gkAwDQYJKoZIhvcNAQELBQADggEBAJwGWUGR3EWwnw6qWhQRQLv5iAn9
DJeaoz53nUVhnWKsEgDMxsq8/ptU+CIjX/GsWKhRhMgSt4dtupK00tNVpwpdPLLP
ndXtsfhfBdOWXMX5AJcW546JM6yXdCUWnpfHNKM2yntjoAOIzxk3aPg1x3nrGBeY
C6zRhQudXN+ptQM+IQkJXW1xMhRyTh4u6mr7sdSLgPgGlivF4a6RLwakm9gh5bm8
HRLXejnMZ4LUT8LXZh2GKQztkGT02H1jsC16ibrincEHZqQuowFwMC1IGaKYtwPd
s0TT1pVAGsuW4eaJOOpv2kiqUHNA1/0koR0bzlpXNlVHg5rJV5tPmYzxb4c=
-----END CERTIFICATE-----
```


### Roles and RoleBindings

With the certificate created it is time to define the Role and RoleBinding for this user `pradeep` to access Kubernetes cluster resources.

First let us create a role called `developer` who can create, get, list and update all pods.

```shell
pradeep@learnk8s$ kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
role.rbac.authorization.k8s.io/developer created
```
Next, we have to bind this role to the user, by creating a role binding and giving it a name, in this case `developer-pradeep`.
```shell
pradeep@learnk8s$ kubectl create rolebinding developer-pradeep --role=developer --user=pradeep
rolebinding.rbac.authorization.k8s.io/developer-pradeep created
```
To verify the role:

```shell
pradeep@learnk8s$ kubectl get roles
NAME        CREATED AT
developer   2022-02-17T06:25:08Z
```
Rolebinding:

```shell
pradeep@learnk8s$ kubectl get rolebindings
NAME                ROLE             AGE
developer-pradeep   Role/developer   26s
```

The last step is to add this user `pradeep` into the kubeconfig file.

For this, first, you need to add new credentials:
```shell
pradeep@learnk8s$ kubectl config set-credentials pradeep --client-key=pradeep.key --client-certificate=pradeep.crt --embed-certs=true
User "pradeep" set.
```

Then, you need to add the context:
```shell
pradeep@learnk8s$ kubectl config set-context pradeep --cluster=k8s --user=pradeep
Context "pradeep" created.
```
Let us check the available contexts first.
```shell
pradeep@learnk8s$ kubectl config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         k8s       k8s       k8s        default
          pradeep   k8s       pradeep
```
We can see there are two, `k8s` the old (default) one and the `pradeep` context which is new. Looking at the first column `CURRENT`, the `*` mark is present for the `k8s` context, indicating that it is the active context.

With this change, let us take another look at the `kubectl config` file.

```yaml
pradeep@learnk8s$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/pradeep/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://192.168.177.29:8443
  name: k8s
contexts:
- context:
    cluster: k8s
    extensions:
    - extension:
        last-update: Tue, 15 Feb 2022 12:28:03 IST
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: k8s
  name: k8s
- context:
    cluster: k8s
    user: pradeep
  name: pradeep
current-context: pradeep
kind: Config
preferences: {}
users:
- name: k8s
  user:
    client-certificate: /Users/pradeep/.minikube/profiles/k8s/client.crt
    client-key: /Users/pradeep/.minikube/profiles/k8s/client.key
- name: pradeep
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

We can see that , there is still one cluster, but two contexts and two users.

It is time to test all the work done in the last half an hour by switching the context.

```shell
pradeep@learnk8s$ kubectl config use-context pradeep
Switched to context "pradeep".
```
Verify it one more way.
```shell
pradeep@learnk8s$ kubectl config current-context
pradeep
```

Now that we are sure of the current-context, let us get the pods as this new user `pradeep`.
```shell
pradeep@learnk8s$ kubectl get pods
NAME                        READY   STATUS             RESTARTS        AGE
kodekloud-8477b7849-blzrs   1/1     Running            0               46h
kodekloud-8477b7849-jkfmr   1/1     Running            0               46h
kodekloud-8477b7849-m65m8   1/1     Running            0               46h
kodekloud-8477b7849-rgbmn   1/1     Running            0               46h
kodekloud-change-color      1/1     Running            0               43h
kodekloud-cm                1/1     Running            0               39h
kodekloud-env-color         1/1     Running            0               40h
kodekloud-env-color-2       1/1     Running            0               40h
kodekloud-secret            1/1     Running            0               29h
multi-containers            1/2     NotReady           0               28h
pod-with-init-container     1/1     Running            14 (12m ago)    28h
```
Yay! :clap: We could get the pods as user `pradeep`.

Let us try to get the nodes this time.
```shell
pradeep@learnk8s$ kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "pradeep" cannot list resource "nodes" in API group "" at the cluster scope
```

Try accessing deployments.
```shell
pradeep@learnk8s$ kubectl get deployments
Error from server (Forbidden): deployments.apps is forbidden: User "pradeep" cannot list resource "deployments" in API group "apps" in the namespace "default"
```

What happened? It is Forbidden. But, this is expected, right?!  We only gave permission for Pods. Isn't it?

We can't verify that (if we don't remember what we have given for this user/role) from the current context. 

```shell
pradeep@learnk8s$ kubectl describe roles developer
Error from server (Forbidden): roles.rbac.authorization.k8s.io "developer" is forbidden: User "pradeep" cannot get resource "roles" in API group "rbac.authorization.k8s.io" in the namespace "default"
```

So, we have to switch the context back to `k8s`.
```shell
pradeep@learnk8s$ kubectl config use-context k8s
Switched to context "k8s".
```
Describe the `developer` role and check what all Resources this role has access to.
```shell
pradeep@learnk8s$ kubectl describe roles developer
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [create get list update delete]
```
This confirms that, any user who is given this role of `developer` can only work with `pods`.

Here is the same in YAML form.

```yaml
pradeep@learnk8s$ kubectl get roles developer -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2022-02-17T06:25:08Z"
  name: developer
  namespace: default
  resourceVersion: "64839"
  uid: 9eae4036-0858-4c09-8c06-7532d74ca1aa
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - get
  - list
  - update
  - delete

```
How can we give access to deployments for this developer? We just need to add additional resource of deployments.

Let us edit the `developer` role.
```shell
pradeep@learnk8s$ kubectl edit role developer
role.rbac.authorization.k8s.io/developer edited
```
Describe the role after editing it, just to confirm we have additional access to deployments.
We have modified few Verbs as well (removed some access!)

```shell
pradeep@learnk8s$ kubectl describe role developer
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources    Non-Resource URLs  Resource Names  Verbs
  ---------    -----------------  --------------  -----
  deployments  []                 []              [get list]
  pods         []                 []              [get list]
```
This is modified YAML version of the `developer` role.

```yaml
pradeep@learnk8s$ kubectl get role developer -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2022-02-17T06:25:08Z"
  name: developer
  namespace: default
  resourceVersion: "66960"
  uid: 9eae4036-0858-4c09-8c06-7532d74ca1aa
rules:
- apiGroups:
  - ""
  resources:
  - deployments
  - pods
  verbs:
  - get
  - list
```
Now switch back to the `pradeep` context and verify if you can get the nodes.
```shell
pradeep@learnk8s$ kubectl config use-context pradeep
Switched to context "pradeep".
```
Let us see if `pradeep` can get deployments.

```shell
pradeep@learnk8s$ kubectl get deployments
Error from server (Forbidden): deployments.apps is forbidden: User "pradeep" cannot list resource "deployments" in API group "apps" in the namespace "default"
```
Still Forbidden.  It is do with the `apiGroups` settings. Currently we did not specify anything, it was left blank.

We can edit again, change the `apiGroups` to any with the `*`. Though in production environment, we might want to be more specific and only configured limited groups as needed.
```shell
pradeep@learnk8s$ kubectl config use-context k8s
Switched to context "k8s".
```
```shell
pradeep@learnk8s$ kubectl edit role developer
role.rbac.authorization.k8s.io/developer edited
```
Look at the `apiGroups` value of `*`.
```yaml
pradeep@learnk8s$ kubectl get role developer -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2022-02-17T06:25:08Z"
  name: developer
  namespace: default
  resourceVersion: "67224"
  uid: 9eae4036-0858-4c09-8c06-7532d74ca1aa
rules:
- apiGroups:
  - '*'
  resources:
  - deployments
  - pods
  verbs:
  - get
  - list
```
To test if this is working or not for `pradeep` user, there is another away, without switching context. We can make use of the `--as` option (impersonation).

Before modifying the `apiGroup` setting, it was like this, for the same test.
```shell
pradeep@learnk8s$ kubectl get deployment --as pradeep
Error from server (Forbidden): deployments.apps is forbidden: User "pradeep" cannot list resource "deployments" in API group "apps" in the namespace "default"
```
Now that we modified, 
```shell
pradeep@learnk8s$ kubectl get deployment --as pradeep
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           2d
```
Though this test confirms that `pradeep` can get the deployments, let us switch context and verify one more time.
```shell
pradeep@learnk8s$ kubectl config use-context pradeep
Switched to context "pradeep".
```
```shell
pradeep@learnk8s$ kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
kodekloud   4/4     4            4           2d
```
Congratulations!
