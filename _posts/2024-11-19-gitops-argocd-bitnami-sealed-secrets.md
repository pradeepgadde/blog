---
layout: single
title:  "GitOps: ArgoCD Bitnami Sealed Secrets"
categories: Kubernetes
tags: ArgoCD
classes: wide
show_date: true
header:
  overlay_image: /assets/images/argo.png
  og_image: /assets/images/argo.png
  teaser: /assets/images/argo.png
  caption: "Photo credit: [**Argo**](https://argo-cd.readthedocs.io/en/stable/)"
  actions:
    - label: "Learn more"
      url: "https://argo-cd.readthedocs.io/en/stable/"

author:
  name     : "ArgoCD"
  avatar   : "/assets/images/argo.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

# GitOps: ArgoCD Bitnami Sealed Secrets

```sh

myk8scluster ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
myk8scluster   Ready    control-plane   3h35m   v1.30.0

myk8scluster ~ ➜  wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-0.18.0-linux-amd64.tar.gz
tar -xvzf kubeseal-0.18.0-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
--2024-11-19 20:54:04--  https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-0.18.0-linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.112.3
Connecting to github.com (github.com)|140.82.112.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/92702519/c48b366d-822a-44f3-ab3e-4e201bbd43d1?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20241119%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241119T205404Z&X-Amz-Expires=300&X-Amz-Signature=511b18be658012adfdce70726c4a177bd01e7675e8acacd20cafea77a79d9799&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dkubeseal-0.18.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2024-11-19 20:54:04--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/92702519/c48b366d-822a-44f3-ab3e-4e201bbd43d1?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20241119%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20241119T205404Z&X-Amz-Expires=300&X-Amz-Signature=511b18be658012adfdce70726c4a177bd01e7675e8acacd20cafea77a79d9799&X-Amz-SignedHeaders=host&response-content-disposition=attachment%3B%20filename%3Dkubeseal-0.18.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.108.133, 185.199.111.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18116498 (17M) [application/octet-stream]
Saving to: ‘kubeseal-0.18.0-linux-amd64.tar.gz’

kubeseal-0.18.0-linu 100%[===================>]  17.28M  --.-KB/s    in 0.1s    

2024-11-19 20:54:05 (119 MB/s) - ‘kubeseal-0.18.0-linux-amd64.tar.gz’ saved [18116498/18116498]

kubeseal

myk8scluster ~ ➜  kubectl -n kube-system get secret
NAME                      TYPE                            DATA   AGE
bootstrap-token-tzbawj    bootstrap.kubernetes.io/token   7      3h37m
sealed-secrets-keybhhp7   kubernetes.io/tls               2      3m2s

myk8scluster ~ ➜  kubectl -n kube-system get secret sealed-secrets-keybhhp7 -o js
on | jq -r .data'."tls.crt"' | base64 -d > /root/sealedSecret-publicCert.crt

myk8scluster ~ ➜  kubectl create secret generic app-crds --from-literal=apikey=zaCELgL-0imfnc8mVLWwsAawjYr4Rx-Af50DDqtlx --from-literal=username=admin-dev-group --from-literal=password=paSsw0rD-1erT-diS -o yaml --dry-run=client > mysql-password_k8s-secret.yaml

myk8scluster ~ ➜  kubeseal -o yaml --scope cluster-wide --cert sealedSecret-publicCert.crt < mysql-password_k8s-secret.yaml > mysql-password_sealed-secret.yaml

myk8scluster ~ ➜  cat mysql-password_k8s-secret.yaml 
apiVersion: v1
data:
  apikey: emFDRUxnTC0waW1mbmM4bVZMV3dzQWF3allyNFJ4LUFmNTBERHF0bHg=
  password: cGFTc3cwckQtMWVyVC1kaVM=
  username: YWRtaW4tZGV2LWdyb3Vw
kind: Secret
metadata:
  creationTimestamp: null
  name: app-crds

myk8scluster ~ ➜  cat mysql-password_sealed-secret.yaml 
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  annotations:
    sealedsecrets.bitnami.com/cluster-wide: "true"
  creationTimestamp: null
  name: app-crds
spec:
  encryptedData:
    apikey: AgC6oFhEB00GaXodLCTgg8cp82Wa8Zo8fBYKT8hilgsmvKBUsf97V+8kKyG7syLWpBWDmUWS5//PFnZbz5D0QqapJYDCkLw0q3kJn2pGzV9rV78e0pk5w+emcqCYk6VD4YCA40FcY1KuXe1ifRkLp2ldyhqmUyMo7XN49L3ic6VcWUhaGakLRKbJ5XEKeOHOxMQCI59XeOpdz0T/JTd8/FURNAT+Cd7uFrOT3ScySeD8CqfzwF+/Jc9nEiCoto54ulyH6Qgh4TR8YN52PvlT6h1K6xHf6EUY7rVHiS1HL4z2JmsWE6NJFCg9iBNOitJKvNhslgrToQw4QAjMW8RZ2eVes1jTD8IqK/czuQGUE5YXOTraPXN1feIzKXOYRrau4kKphKJQNB0wAMX4Wu4Em4bm82e8qSB+kl5cVEPbdhbc9g6v6+xfThhl/T4R9NXYYQCez9I3dSdyC739ya9NeUN4BPD9SK0q+zWbfO4LNyIhOxkrh/T3RFlQDYjiefkLKw6GvGmRpw9Jgs2WYXFYIwp3NKL5qASElsuktq0+l1VQyb6Gkfu+/EhCr3Kfowr3nQJiMGF6gSxXmYwcu5KKibh6WGzTfjoxkv/hjwASjvGZ8y8nk+icVPCN53Fjf8P0l5JMEseT9TymxiWJVD2x2ZUoBQLBrhA5ZvtRWk7svLOOq7UPcaFH6pGOO8rVHrIYFylMTD+Y+UyXmO2DGBk7NvHJxZxXjLhjbvMhc+9dMGsgaRe+bwVyZtM6Hg==
    password: AgDAiPU5CmcGRHWJr4briTkNo7oly9WhFDLAS+t8YFCuYbwgSzLfT+5XYGXO+Lp4WAehpprEhIMWLZJWH9OSl+3kv2t1dy104AufwfMUWR8tSwZNDTOU6R1LkmLOt+zicpmi5gfTNylYaJbqBmth/qgGHc5ybj3xWoJEDTpEQGqolmEd+wxNL1Y3ZMrabJ0Rv8E03E8SI69TNx4+vx9EOrJ36iLOifTJbHpQ8CqF0HmOxmeGKmNrcZZuLKkQdlhLYE1S9XS3s6m5Pv0a0Y6Cb93gWa0FDvjaeKiWxWlBdwiFOBGKLQhZDpaeN3M6egVSu51bSjKVFI3LU8ki4FUk6dHEIbxDEsx51V0FuqKxNmdPEeXoqrDvnswk+8JeCQj2VyFGKZleNd7+CiKvQfko25P2yRPWQIVts9PMJNUU6qfX6VkfEo867q+wYL0tmJ8Xnx4/3pXm8vaMNLqgs/uPclkBQxPLcJ7sBa0RedmwSoDgzzqfxA430L5mCu0LzX3WLBKvld2PtjdxWS4zRj59Wrn5LcrixPFjDF3zemI0yrnjd181XB57RV1N6PYpGJu2KN22LXuQ63R4YEXhIwivbfMbZjMro1Tffv2K+ZTqwnFgzyWSFz7GIedwn8e7+n2bqwnbThym/zCq0bE9lTH4xd7wgqQJ/qsVYrtcKMMfEV5QV92gHDAVowmOG9hTsI1YcLTKhpx9W+BMal3a0u3TMWbwgQ==
    username: AgB3K2FncMK2SOeM73c1o9zp958FcEcGPLrRiTENV+1s/XbMTrxludWgN2wWyNYAD/iKts0yaD26CpgpkosnCkPWsldsnq0CSqPTXDHd2tpqKWY0aLIDDtzdnNMQtV03LVD8L5sUBQahylvcyo98JfuYbHz4b+uvwhsBRXaikONbu9UAZ8cl3JpmJybUm+5zu+adukt2nFYvl1TuRvtHhcZ6b/mI4WiSB+ICj6sWa1hsbw19wNlRmidJE2eQ9Gsj2foC0ovAKXhP2tfRVpUnrwIaSxV6bfzXMFbm/5duiUIjf7hYEdnBM6xufWIwBpSq2NQ9nfZ+N70dIlYofuKKgT1faIxOT93N6UlKBUQOWf6VBiP/lTwF4+/5LCXYVp+NN3NLktDCbMhVgrJMuCHeJfe2E61MP1YdKO9URGlfGc7EGZDP+NFPKy424LxzagWYLSp+jKTNCpJkLeehuAtbRmXOEtOyb6GqqK9OXI08IdKfFgDHWUd2O4bxGL+tGqH6GLYdVZKn8L//SE6bGi7mF5XV/sWznt2F8TFd8Q5nD8y+uyODIjlq3W4gfz2+yzbo4BMHahjl9d5lErHMKpKRRxPVqzsqtpUPnCh3evHjevmhn7P3eYxWjwHjA+3BKvRiYCGpdyWlHMyeC3nqwm+TemhxlMJkUBOaiP+cSnBa3FWRD15/WP4nCLKS220Tv0zpl9+2/QWfgCH/OLd6c/scxL0=
  template:
    data: null
    metadata:
      annotations:
        sealedsecrets.bitnami.com/cluster-wide: "true"
      creationTimestamp: null
      name: app-crds


myk8scluster ~ ➜  curl -Lo argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v1.12.0/argocd-vault-plugin_1.12.0_linux_amd64
chmod +x argocd-vault-plugin 
mv argocd-vault-plugin /usr/local/bin
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 66.3M  100 66.3M    0     0  44.4M      0  0:00:01  0:00:01 --:--:-- 69.7M

myk8scluster ~ ➜  argocd-vault-plugin generate -c /root/vault.env - < /root/secret.yaml > /root/secret_updated.yaml

myk8scluster ~ ➜  cat /root/secret
cat: /root/secret: No such file or directory

myk8scluster ~ ✖ cat /root/secret.yaml 
kind: Secret
apiVersion: v1
metadata:
  name: app-crds
  annotations:
    avp.kubernetes.io/path: "credentials/data/app"
type: Opaque
stringData:
  apikey: <apikey>
  username: <username>
  password: <password>

myk8scluster ~ ➜  cat /root/secret
secret_updated.yaml  secret.yaml          

myk8scluster ~ ➜  cat /root/secret_updated.yaml 
apiVersion: v1
kind: Secret
metadata:
  annotations:
    avp.kubernetes.io/path: credentials/data/app
  name: app-crds
stringData:
  apikey: dgg7B3BaaeBleqE
  password: skdjD432JDjd
  username: bob
type: Opaque
---

myk8scluster ~ ➜  

myk8scluster ~ ➜  kubectl edit deployments.apps -n argocd argocd-repo-server
error: deployments.apps "argocd-repo-server" is invalid
deployment.apps/argocd-repo-server edited

myk8scluster ~ ➜  kubectl edit deployments.apps -n argocd argocd-repo-server
error: deployments.apps "argocd-repo-server" is invalid
error: deployments.apps "argocd-repo-server" is invalid
Edit cancelled, no changes made.

myk8scluster ~ ➜  kubectl edit -n argocd cm argocd-cm
configmap/argocd-cm edited

myk8scluster ~ ➜  
myk8scluster ~ ➜  kubectl get cm argocd-cm -n argocd -o json
{
    "apiVersion": "v1",
    "data": {
        "configManagementPlugins": "- name: argocd-vault-plugin\n  generate:\n    command: [\"argocd-vault-plugin\"]\n    args: [\"generate\", \"./\"]    "
    },
    "kind": "ConfigMap",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"ConfigMap\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/name\":\"argocd-cm\",\"app.kubernetes.io/part-of\":\"argocd\"},\"name\":\"argocd-cm\",\"namespace\":\"argocd\"}}\n"
        },
        "creationTimestamp": "2024-11-19T20:48:44Z",
        "labels": {
            "app.kubernetes.io/name": "argocd-cm",
            "app.kubernetes.io/part-of": "argocd"
        },
        "name": "argocd-cm",
        "namespace": "argocd",
        "resourceVersion": "19453",
        "uid": "8e01ef2f-619c-4b4a-8115-f761306f3ef2"
    }
}

myk8scluster ~ ➜  
```

```yaml
myk8scluster ~ ➜  kubectl get deployments.apps -n argocd argocd-repo-server -o json
{
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {
        "annotations": {
            "deployment.kubernetes.io/revision": "3",
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"apps/v1\",\"kind\":\"Deployment\",\"metadata\":{\"annotations\":{},\"labels\":{\"app.kubernetes.io/component\":\"repo-server\",\"app.kubernetes.io/name\":\"argocd-repo-server\",\"app.kubernetes.io/part-of\":\"argocd\"},\"name\":\"argocd-repo-server\",\"namespace\":\"argocd\"},\"spec\":{\"selector\":{\"matchLabels\":{\"app.kubernetes.io/name\":\"argocd-repo-server\"}},\"template\":{\"metadata\":{\"labels\":{\"app.kubernetes.io/name\":\"argocd-repo-server\"}},\"spec\":{\"affinity\":{\"podAntiAffinity\":{\"preferredDuringSchedulingIgnoredDuringExecution\":[{\"podAffinityTerm\":{\"labelSelector\":{\"matchLabels\":{\"app.kubernetes.io/name\":\"argocd-repo-server\"}},\"topologyKey\":\"kubernetes.io/hostname\"},\"weight\":100},{\"podAffinityTerm\":{\"labelSelector\":{\"matchLabels\":{\"app.kubernetes.io/part-of\":\"argocd\"}},\"topologyKey\":\"kubernetes.io/hostname\"},\"weight\":5}]}},\"automountServiceAccountToken\":false,\"containers\":[{\"command\":[\"sh\",\"-c\",\"entrypoint.sh argocd-repo-server --redis argocd-redis:6379\"],\"env\":[{\"name\":\"ARGOCD_RECONCILIATION_TIMEOUT\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"timeout.reconciliation\",\"name\":\"argocd-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_LOGFORMAT\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.log.format\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_LOGLEVEL\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.log.level\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_PARALLELISM_LIMIT\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.parallelism.limit\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_DISABLE_TLS\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.disable.tls\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_TLS_MIN_VERSION\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.tls.minversion\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_TLS_MAX_VERSION\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.tls.maxversion\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_TLS_CIPHERS\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.tls.ciphers\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_CACHE_EXPIRATION\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.repo.cache.expiration\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"REDIS_SERVER\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"redis.server\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"REDISDB\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"redis.db\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_DEFAULT_CACHE_EXPIRATION\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.default.cache.expiration\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_OTLP_ADDRESS\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"otlp.address\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_MAX_COMBINED_DIRECTORY_MANIFESTS_SIZE\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.max.combined.directory.manifests.size\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"ARGOCD_REPO_SERVER_PLUGIN_TAR_EXCLUSIONS\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"reposerver.plugin.tar.exclusions\",\"name\":\"argocd-cmd-params-cm\",\"optional\":true}}},{\"name\":\"HELM_CACHE_HOME\",\"value\":\"/helm-working-dir\"},{\"name\":\"HELM_CONFIG_HOME\",\"value\":\"/helm-working-dir\"},{\"name\":\"HELM_DATA_HOME\",\"value\":\"/helm-working-dir\"}],\"image\":\"quay.io/argoproj/argocd:v2.4.11\",\"imagePullPolicy\":\"Always\",\"livenessProbe\":{\"failureThreshold\":3,\"httpGet\":{\"path\":\"/healthz?full=true\",\"port\":8084},\"initialDelaySeconds\":30,\"periodSeconds\":5},\"name\":\"argocd-repo-server\",\"ports\":[{\"containerPort\":8081},{\"containerPort\":8084}],\"readinessProbe\":{\"httpGet\":{\"path\":\"/healthz\",\"port\":8084},\"initialDelaySeconds\":5,\"periodSeconds\":10},\"securityContext\":{\"allowPrivilegeEscalation\":false,\"capabilities\":{\"drop\":[\"all\"]},\"readOnlyRootFilesystem\":true,\"runAsNonRoot\":true},\"volumeMounts\":[{\"mountPath\":\"/app/config/ssh\",\"name\":\"ssh-known-hosts\"},{\"mountPath\":\"/app/config/tls\",\"name\":\"tls-certs\"},{\"mountPath\":\"/app/config/gpg/source\",\"name\":\"gpg-keys\"},{\"mountPath\":\"/app/config/gpg/keys\",\"name\":\"gpg-keyring\"},{\"mountPath\":\"/app/config/reposerver/tls\",\"name\":\"argocd-repo-server-tls\"},{\"mountPath\":\"/tmp\",\"name\":\"tmp\"},{\"mountPath\":\"/helm-working-dir\",\"name\":\"helm-working-dir\"},{\"mountPath\":\"/home/argocd/cmp-server/plugins\",\"name\":\"plugins\"}]}],\"initContainers\":[{\"command\":[\"cp\",\"-n\",\"/usr/local/bin/argocd\",\"/var/run/argocd/argocd-cmp-server\"],\"image\":\"quay.io/argoproj/argocd:v2.4.11\",\"name\":\"copyutil\",\"securityContext\":{\"allowPrivilegeEscalation\":false,\"capabilities\":{\"drop\":[\"all\"]},\"readOnlyRootFilesystem\":true,\"runAsNonRoot\":true},\"volumeMounts\":[{\"mountPath\":\"/var/run/argocd\",\"name\":\"var-files\"}]}],\"serviceAccountName\":\"argocd-repo-server\",\"volumes\":[{\"configMap\":{\"name\":\"argocd-ssh-known-hosts-cm\"},\"name\":\"ssh-known-hosts\"},{\"configMap\":{\"name\":\"argocd-tls-certs-cm\"},\"name\":\"tls-certs\"},{\"configMap\":{\"name\":\"argocd-gpg-keys-cm\"},\"name\":\"gpg-keys\"},{\"emptyDir\":{},\"name\":\"gpg-keyring\"},{\"emptyDir\":{},\"name\":\"tmp\"},{\"emptyDir\":{},\"name\":\"helm-working-dir\"},{\"name\":\"argocd-repo-server-tls\",\"secret\":{\"items\":[{\"key\":\"tls.crt\",\"path\":\"tls.crt\"},{\"key\":\"tls.key\",\"path\":\"tls.key\"},{\"key\":\"ca.crt\",\"path\":\"ca.crt\"}],\"optional\":true,\"secretName\":\"argocd-repo-server-tls\"}},{\"emptyDir\":{},\"name\":\"var-files\"},{\"emptyDir\":{},\"name\":\"plugins\"}]}}}}\n"
        },
        "creationTimestamp": "2024-11-19T20:48:45Z",
        "generation": 3,
        "labels": {
            "app.kubernetes.io/component": "repo-server",
            "app.kubernetes.io/name": "argocd-repo-server",
            "app.kubernetes.io/part-of": "argocd"
        },
        "name": "argocd-repo-server",
        "namespace": "argocd",
        "resourceVersion": "19176",
        "uid": "bc0f91b1-b6a7-4598-bff2-f25909b80fd8"
    },
    "spec": {
        "progressDeadlineSeconds": 600,
        "replicas": 1,
        "revisionHistoryLimit": 10,
        "selector": {
            "matchLabels": {
                "app.kubernetes.io/name": "argocd-repo-server"
            }
        },
        "strategy": {
            "rollingUpdate": {
                "maxSurge": "25%",
                "maxUnavailable": "25%"
            },
            "type": "RollingUpdate"
        },
        "template": {
            "metadata": {
                "creationTimestamp": null,
                "labels": {
                    "app.kubernetes.io/name": "argocd-repo-server"
                }
            },
            "spec": {
                "affinity": {
                    "podAntiAffinity": {
                        "preferredDuringSchedulingIgnoredDuringExecution": [
                            {
                                "podAffinityTerm": {
                                    "labelSelector": {
                                        "matchLabels": {
                                            "app.kubernetes.io/name": "argocd-repo-server"
                                        }
                                    },
                                    "topologyKey": "kubernetes.io/hostname"
                                },
                                "weight": 100
                            },
                            {
                                "podAffinityTerm": {
                                    "labelSelector": {
                                        "matchLabels": {
                                            "app.kubernetes.io/part-of": "argocd"
                                        }
                                    },
                                    "topologyKey": "kubernetes.io/hostname"
                                },
                                "weight": 5
                            }
                        ]
                    }
                },
                "automountServiceAccountToken": false,
                "containers": [
                    {
                        "command": [
                            "sh",
                            "-c",
                            "entrypoint.sh argocd-repo-server --redis argocd-redis:6379"
                        ],
                        "env": [
                            {
                                "name": "ARGOCD_RECONCILIATION_TIMEOUT",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "timeout.reconciliation",
                                        "name": "argocd-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_LOGFORMAT",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.log.format",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_LOGLEVEL",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.log.level",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_PARALLELISM_LIMIT",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.parallelism.limit",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_DISABLE_TLS",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.disable.tls",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_TLS_MIN_VERSION",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.tls.minversion",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_TLS_MAX_VERSION",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.tls.maxversion",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_TLS_CIPHERS",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.tls.ciphers",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_CACHE_EXPIRATION",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.repo.cache.expiration",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "REDIS_SERVER",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "redis.server",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "REDISDB",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "redis.db",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_DEFAULT_CACHE_EXPIRATION",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.default.cache.expiration",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_OTLP_ADDRESS",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "otlp.address",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_MAX_COMBINED_DIRECTORY_MANIFESTS_SIZE",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.max.combined.directory.manifests.size",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "ARGOCD_REPO_SERVER_PLUGIN_TAR_EXCLUSIONS",
                                "valueFrom": {
                                    "configMapKeyRef": {
                                        "key": "reposerver.plugin.tar.exclusions",
                                        "name": "argocd-cmd-params-cm",
                                        "optional": true
                                    }
                                }
                            },
                            {
                                "name": "HELM_CACHE_HOME",
                                "value": "/helm-working-dir"
                            },
                            {
                                "name": "HELM_CONFIG_HOME",
                                "value": "/helm-working-dir"
                            },
                            {
                                "name": "HELM_DATA_HOME",
                                "value": "/helm-working-dir"
                            }
                        ],
                        "image": "quay.io/argoproj/argocd:v2.4.11",
                        "imagePullPolicy": "Always",
                        "livenessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/healthz?full=true",
                                "port": 8084,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 30,
                            "periodSeconds": 5,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "name": "argocd-repo-server",
                        "ports": [
                            {
                                "containerPort": 8081,
                                "protocol": "TCP"
                            },
                            {
                                "containerPort": 8084,
                                "protocol": "TCP"
                            }
                        ],
                        "readinessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/healthz",
                                "port": 8084,
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 5,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "resources": {},
                        "securityContext": {
                            "allowPrivilegeEscalation": false,
                            "capabilities": {
                                "drop": [
                                    "all"
                                ]
                            },
                            "readOnlyRootFilesystem": true,
                            "runAsNonRoot": true,
                            "seccompProfile": {
                                "type": "RuntimeDefault"
                            }
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/usr/local/bin/argocd-vault-plugin",
                                "name": "custom-tools",
                                "subPath": "argocd-vault-plugin"
                            },
                            {
                                "mountPath": "/app/config/ssh",
                                "name": "ssh-known-hosts"
                            },
                            {
                                "mountPath": "/app/config/tls",
                                "name": "tls-certs"
                            },
                            {
                                "mountPath": "/app/config/gpg/source",
                                "name": "gpg-keys"
                            },
                            {
                                "mountPath": "/app/config/gpg/keys",
                                "name": "gpg-keyring"
                            },
                            {
                                "mountPath": "/app/config/reposerver/tls",
                                "name": "argocd-repo-server-tls"
                            },
                            {
                                "mountPath": "/tmp",
                                "name": "tmp"
                            },
                            {
                                "mountPath": "/helm-working-dir",
                                "name": "helm-working-dir"
                            },
                            {
                                "mountPath": "/home/argocd/cmp-server/plugins",
                                "name": "plugins"
                            }
                        ]
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "initContainers": [
                    {
                        "args": [
                            "wget -O argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64 \u0026\u0026 chmod +x argocd-vault-plugin \u0026\u0026 mv argocd-vault-plugin /custom-tools/"
                        ],
                        "command": [
                            "sh",
                            "-c"
                        ],
                        "env": [
                            {
                                "name": "AVP_VERSION",
                                "value": "1.7.0"
                            }
                        ],
                        "image": "alpine:3.8",
                        "imagePullPolicy": "IfNotPresent",
                        "name": "download-tools",
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/custom-tools",
                                "name": "custom-tools"
                            }
                        ]
                    },
                    {
                        "command": [
                            "cp",
                            "-n",
                            "/usr/local/bin/argocd",
                            "/var/run/argocd/argocd-cmp-server"
                        ],
                        "image": "quay.io/argoproj/argocd:v2.4.11",
                        "imagePullPolicy": "IfNotPresent",
                        "name": "copyutil",
                        "resources": {},
                        "securityContext": {
                            "allowPrivilegeEscalation": false,
                            "capabilities": {
                                "drop": [
                                    "all"
                                ]
                            },
                            "readOnlyRootFilesystem": true,
                            "runAsNonRoot": true
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/var/run/argocd",
                                "name": "var-files"
                            }
                        ]
                    }
                ],
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "serviceAccount": "argocd-repo-server",
                "serviceAccountName": "argocd-repo-server",
                "terminationGracePeriodSeconds": 30,
                "volumes": [
                    {
                        "emptyDir": {},
                        "name": "custom-tools"
                    },
                    {
                        "configMap": {
                            "defaultMode": 420,
                            "name": "argocd-ssh-known-hosts-cm"
                        },
                        "name": "ssh-known-hosts"
                    },
                    {
                        "configMap": {
                            "defaultMode": 420,
                            "name": "argocd-tls-certs-cm"
                        },
                        "name": "tls-certs"
                    },
                    {
                        "configMap": {
                            "defaultMode": 420,
                            "name": "argocd-gpg-keys-cm"
                        },
                        "name": "gpg-keys"
                    },
                    {
                        "emptyDir": {},
                        "name": "gpg-keyring"
                    },
                    {
                        "emptyDir": {},
                        "name": "tmp"
                    },
                    {
                        "emptyDir": {},
                        "name": "helm-working-dir"
                    },
                    {
                        "name": "argocd-repo-server-tls",
                        "secret": {
                            "defaultMode": 420,
                            "items": [
                                {
                                    "key": "tls.crt",
                                    "path": "tls.crt"
                                },
                                {
                                    "key": "tls.key",
                                    "path": "tls.key"
                                },
                                {
                                    "key": "ca.crt",
                                    "path": "ca.crt"
                                }
                            ],
                            "optional": true,
                            "secretName": "argocd-repo-server-tls"
                        }
                    },
                    {
                        "emptyDir": {},
                        "name": "var-files"
                    },
                    {
                        "emptyDir": {},
                        "name": "plugins"
                    }
                ]
            }
        }
    },
    "status": {
        "availableReplicas": 1,
        "conditions": [
            {
                "lastTransitionTime": "2024-11-19T20:49:26Z",
                "lastUpdateTime": "2024-11-19T20:49:26Z",
                "message": "Deployment has minimum availability.",
                "reason": "MinimumReplicasAvailable",
                "status": "True",
                "type": "Available"
            },
            {
                "lastTransitionTime": "2024-11-19T20:48:45Z",
                "lastUpdateTime": "2024-11-19T21:10:31Z",
                "message": "ReplicaSet \"argocd-repo-server-6bf55f7c\" has successfully progressed.",
                "reason": "NewReplicaSetAvailable",
                "status": "True",
                "type": "Progressing"
            }
        ],
        "observedGeneration": 3,
        "readyReplicas": 1,
        "replicas": 1,
        "updatedReplicas": 1
    }
}

myk8scluster ~ ➜  
```

```sh
controlplane ~ ➜  kubectl get all -n argocd
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                    1/1     Running   0          31m
pod/argocd-applicationset-controller-d7c857898-v5r75   1/1     Running   0          31m
pod/argocd-dex-server-75d98bff7c-24nq7                 1/1     Running   0          31m
pod/argocd-notifications-controller-684947df85-2b2mw   1/1     Running   0          31m
pod/argocd-redis-84c8cd4d8-4wp4j                       1/1     Running   0          31m
pod/argocd-repo-server-6bf55f7c-8gqsv                  1/1     Running   0          10m
pod/argocd-server-5f8984f889-qdd65                     1/1     Running   0          31m
pod/vault-0                                            1/1     Running   0          31m
pod/vault-agent-injector-6496bfcc9c-pq654              1/1     Running   0          31m

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   172.20.24.54     <none>        7000/TCP,8080/TCP            31m
service/argocd-dex-server                         ClusterIP   172.20.93.207    <none>        5556/TCP,5557/TCP,5558/TCP   31m
service/argocd-metrics                            ClusterIP   172.20.102.127   <none>        8082/TCP                     31m
service/argocd-notifications-controller-metrics   ClusterIP   172.20.20.194    <none>        9001/TCP                     31m
service/argocd-redis                              ClusterIP   172.20.127.227   <none>        6379/TCP                     31m
service/argocd-repo-server                        ClusterIP   172.20.246.184   <none>        8081/TCP,8084/TCP            31m
service/argocd-server                             NodePort    172.20.40.45     <none>        80:32765/TCP,443:32766/TCP   31m
service/argocd-server-metrics                     ClusterIP   172.20.189.185   <none>        8083/TCP                     31m
service/vault                                     ClusterIP   172.20.135.89    <none>        8200/TCP,8201/TCP            31m
service/vault-agent-injector-svc                  ClusterIP   172.20.18.252    <none>        443/TCP                      31m
service/vault-internal                            ClusterIP   None             <none>        8200/TCP,8201/TCP            31m
service/vault-ui                                  NodePort    172.20.214.120   <none>        8200:30711/TCP               31m

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           31m
deployment.apps/argocd-dex-server                  1/1     1            1           31m
deployment.apps/argocd-notifications-controller    1/1     1            1           31m
deployment.apps/argocd-redis                       1/1     1            1           31m
deployment.apps/argocd-repo-server                 1/1     1            1           31m
deployment.apps/argocd-server                      1/1     1            1           31m
deployment.apps/vault-agent-injector               1/1     1            1           31m

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-d7c857898   1         1         1       31m
replicaset.apps/argocd-dex-server-75d98bff7c                 1         1         1       31m
replicaset.apps/argocd-notifications-controller-684947df85   1         1         1       31m
replicaset.apps/argocd-redis-84c8cd4d8                       1         1         1       31m
replicaset.apps/argocd-repo-server-6b5cf8488                 0         0         0       31m
replicaset.apps/argocd-repo-server-6bf55f7c                  1         1         1       10m
replicaset.apps/argocd-repo-server-7bbc57875d                0         0         0       31m
replicaset.apps/argocd-server-5f8984f889                     1         1         1       31m
replicaset.apps/vault-agent-injector-6496bfcc9c              1         1         1       31m

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     31m
statefulset.apps/vault                           1/1     31m

controlplane ~ ➜  
```

