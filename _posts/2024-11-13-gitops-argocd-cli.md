---
layout: single
title:  "GitOps: ArgoCD Creating applications through CLI"
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

# GitOps: ArgoCD Creating applications through CLI

In this post, let us create an application using Argo CD CLI.

```sh
myk8scluster ~ ➜  argocd -h
argocd controls a Argo CD server

Usage:
  argocd [flags]
  argocd [command]

Available Commands:
  account     Manage account settings
  admin       Contains a set of commands useful for Argo CD administrators and requires direct Kubernetes access
  app         Manage applications
  appset      Manage ApplicationSets
  cert        Manage repository certificates and SSH known hosts entries
  cluster     Manage cluster credentials
  completion  output shell completion code for the specified shell (bash, zsh or fish)
  context     Switch between contexts
  gpg         Manage GPG keys used for signature verification
  help        Help about any command
  login       Log in to Argo CD
  logout      Log out from Argo CD
  proj        Manage projects
  relogin     Refresh an expired authenticate token
  repo        Manage repository connection parameters
  repocreds   Manage repository connection parameters
  version     Print version information

Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/root/.config/argocd/config")
      --controller-name string          Name of the Argo CD Application controller; set this or the ARGOCD_APPLICATION_CONTROLLER_NAME environment variable when the controller's name label differs from the default, for example when installing via the Helm chart (default "argocd-application-controller")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
  -h, --help                            help for argocd
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --kube-context string             Directs the command to the given kube-context
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --redis-haproxy-name string       Name of the Redis HA Proxy; set this or the ARGOCD_REDIS_HAPROXY_NAME environment variable when the HA Proxy's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis-ha-haproxy")
      --redis-name string               Name of the Redis deployment; set this or the ARGOCD_REDIS_NAME environment variable when the Redis's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis")
      --repo-server-name string         Name of the Argo CD Repo server; set this or the ARGOCD_REPO_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-repo-server")
      --server string                   Argo CD server address
      --server-crt string               Server certificate file
      --server-name string              Name of the Argo CD API server; set this or the ARGOCD_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-server")

Use "argocd [command] --help" for more information about a command.



myk8scluster ~ ✖ argocd app list
NAME                       CLUSTER                         NAMESPACE     PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS                REPO                                                                          PATH            TARGET
argocd/solar-system-app-1  https://kubernetes.default.svc  solar-system  default  OutOfSync  Healthy  Manual      SharedResourceWarning(2)  https://3000-port-f6b34b1681704a7e.labs.kodekloud.com//bob/gitops-argocd.git  ./solar-system  HEAD
argocd/solar-system-app-2  https://kubernetes.default.svc  solar-system  default  Synced     Healthy  Manual      <none>                    https://3000-port-f6b34b1681704a7e.labs.kodekloud.com//bob/gitops-argocd.git  ./solar-system  

myk8scluster ~ ➜  argocd proj list
NAME     DESCRIPTION  DESTINATIONS  SOURCES  CLUSTER-RESOURCE-WHITELIST  NAMESPACE-RESOURCE-BLACKLIST  SIGNATURE-KEYS  ORPHANED-RESOURCES
default               *,*           *        */*                         <none>                        <none>          disabled

myk8scluster ~ ➜  argocd cluster list
SERVER                          NAME        VERSION  STATUS      MESSAGE  PROJECT
https://kubernetes.default.svc  in-cluster  1.30     Successful           

myk8scluster ~ ➜  argocd app create solar-system-app- --repo https://3000-port-f6
b34b1681704a7e.labs.kodekloud.com/bob/gitops-argocd.git --path ./solar-system --d
est-namespace solar-system --dest-server https://kubernetes.default.svc
FATA[0000] rpc error: code = InvalidArgument desc = error creating application: Application.argoproj.io "solar-system-app-" is invalid: metadata.name: Invalid value: "solar-system-app-": a lowercase RFC 1123 subdomain must consist of lower case alphanumeric characters, '-' or '.', and must start and end with an alphanumeric character (e.g. 'example.com', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*') 

myk8scluster ~ ✖ argocd app create solar-system-app-3 --repo https://3000-port-f6
b34b1681704a7e.labs.kodekloud.com/bob/gitops-argocd.git --path ./solar-system --d
est-namespace solar-system --dest-server https://kubernetes.default.svc
application 'solar-system-app-3' created

myk8scluster ~ ➜  argocd cluster list
SERVER                          NAME        VERSION  STATUS      MESSAGE  PROJECT
https://kubernetes.default.svc  in-cluster  1.30     Successful           

myk8scluster ~ ➜  argocd app list
NAME                       CLUSTER                         NAMESPACE     PROJECT  STATUS     HEALTH   SYNCPOLICY  CONDITIONS                REPO                                                                          PATH            TARGET
argocd/solar-system-app-1  https://kubernetes.default.svc  solar-system  default  OutOfSync  Healthy  Manual      SharedResourceWarning(2)  https://3000-port-f6b34b1681704a7e.labs.kodekloud.com//bob/gitops-argocd.git  ./solar-system  HEAD
argocd/solar-system-app-2  https://kubernetes.default.svc  solar-system  default  Synced     Healthy  Manual      <none>                    https://3000-port-f6b34b1681704a7e.labs.kodekloud.com//bob/gitops-argocd.git  ./solar-system  
argocd/solar-system-app-3  https://kubernetes.default.svc  solar-system  default  OutOfSync  Healthy  Manual      SharedResourceWarning(2)  https://3000-port-f6b34b1681704a7e.labs.kodekloud.com/bob/gitops-argocd.git   ./solar-system  

myk8scluster ~ ➜  argocd repo list
TYPE  NAME  REPO  INSECURE  OCI  LFS  CREDS  STATUS  MESSAGE  PROJECT

myk8scluster ~ ➜  argocd version
argocd: v2.12.6+4dab5bd
  BuildDate: 2024-10-18T18:51:56Z
  GitCommit: 4dab5bd6a60adea12e084ad23519e35b710060a2
  GitTreeState: clean
  GoVersion: go1.22.8
  Compiler: gc
  Platform: linux/amd64
argocd-server: v2.4.11+3d9e9f2
  BuildDate: 2022-08-22T09:13:16Z
  GitCommit: 3d9e9f2f95b7801b90377ecfc4073e5f0f07205b
  GitTreeState: clean
  GoVersion: go1.18.5
  Compiler: gc
  Platform: linux/amd64
  Kustomize Version: v4.4.1 2021-11-11T23:36:27Z
  Helm Version: v3.8.1+g5cb9af4
  Kubectl Version: v0.23.1
  Jsonnet Version: v0.18.0

myk8scluster ~ ➜  argocd help
argocd controls a Argo CD server

Usage:
  argocd [flags]
  argocd [command]

Available Commands:
  account     Manage account settings
  admin       Contains a set of commands useful for Argo CD administrators and requires direct Kubernetes access
  app         Manage applications
  appset      Manage ApplicationSets
  cert        Manage repository certificates and SSH known hosts entries
  cluster     Manage cluster credentials
  completion  output shell completion code for the specified shell (bash, zsh or fish)
  context     Switch between contexts
  gpg         Manage GPG keys used for signature verification
  help        Help about any command
  login       Log in to Argo CD
  logout      Log out from Argo CD
  proj        Manage projects
  relogin     Refresh an expired authenticate token
  repo        Manage repository connection parameters
  repocreds   Manage repository connection parameters
  version     Print version information

Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/root/.config/argocd/config")
      --controller-name string          Name of the Argo CD Application controller; set this or the ARGOCD_APPLICATION_CONTROLLER_NAME environment variable when the controller's name label differs from the default, for example when installing via the Helm chart (default "argocd-application-controller")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
  -h, --help                            help for argocd
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --kube-context string             Directs the command to the given kube-context
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --redis-haproxy-name string       Name of the Redis HA Proxy; set this or the ARGOCD_REDIS_HAPROXY_NAME environment variable when the HA Proxy's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis-ha-haproxy")
      --redis-name string               Name of the Redis deployment; set this or the ARGOCD_REDIS_NAME environment variable when the Redis's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis")
      --repo-server-name string         Name of the Argo CD Repo server; set this or the ARGOCD_REPO_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-repo-server")
      --server string                   Argo CD server address
      --server-crt string               Server certificate file
      --server-name string              Name of the Argo CD API server; set this or the ARGOCD_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-server")

Use "argocd [command] --help" for more information about a command.

myk8scluster ~ ➜  argocd repocreds
Manage repository connection parameters

Usage:
  argocd repocreds [flags]
  argocd repocreds [command]

Examples:
  # Add credentials with user/pass authentication to use for all repositories under the specified URL
  argocd repocreds add URL --username USERNAME --password PASSWORD
  
  # List all the configured repository credentials
  argocd repocreds list
  
  # Remove credentials for the repositories with speficied URL
  argocd repocreds rm URL

Available Commands:
  add         Add git repository connection parameters
  list        List configured repository credentials
  rm          Remove repository credentials

Flags:
      --as string                      Username to impersonate for the operation
      --as-group stringArray           Group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --as-uid string                  UID to impersonate for the operation
      --certificate-authority string   Path to a cert file for the certificate authority
      --client-certificate string      Path to a client certificate file for TLS
      --client-key string              Path to a client key file for TLS
      --cluster string                 The name of the kubeconfig cluster to use
      --context string                 The name of the kubeconfig context to use
      --disable-compression            If true, opt-out of response compression for all requests to the server
  -h, --help                           help for repocreds
      --insecure-skip-tls-verify       If true, the server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kubeconfig string              Path to a kube config. Only required if out-of-cluster
  -n, --namespace string               If present, the namespace scope for this CLI request
      --password string                Password for basic authentication to the API server
      --proxy-url string               If provided, this URL will be used to connect via proxy
      --request-timeout string         The length of time to wait before giving up on a single server request. Non-zero values should contain a corresponding time unit (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests. (default "0")
      --tls-server-name string         If provided, this name will be used to validate server certificate. If this is not provided, hostname used to contact the server is used.
      --token string                   Bearer token for authentication to the API server
      --user string                    The name of the kubeconfig user to use
      --username string                Username for basic authentication to the API server

Global Flags:
      --auth-token string               Authentication token
      --client-crt string               Client certificate file
      --client-crt-key string           Client certificate key file
      --config string                   Path to Argo CD config (default "/root/.config/argocd/config")
      --controller-name string          Name of the Argo CD Application controller; set this or the ARGOCD_APPLICATION_CONTROLLER_NAME environment variable when the controller's name label differs from the default, for example when installing via the Helm chart (default "argocd-application-controller")
      --core                            If set to true then CLI talks directly to Kubernetes instead of talking to Argo CD API server
      --grpc-web                        Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2.
      --grpc-web-root-path string       Enables gRPC-web protocol. Useful if Argo CD server is behind proxy which does not support HTTP2. Set web root.
  -H, --header strings                  Sets additional header to all requests made by Argo CD CLI. (Can be repeated multiple times to add multiple headers, also supports comma separated headers)
      --http-retry-max int              Maximum number of retries to establish http connection to Argo CD server
      --insecure                        Skip server certificate and domain verification
      --kube-context string             Directs the command to the given kube-context
      --logformat string                Set the logging format. One of: text|json (default "text")
      --loglevel string                 Set the logging level. One of: debug|info|warn|error (default "info")
      --plaintext                       Disable TLS
      --port-forward                    Connect to a random argocd-server port using port forwarding
      --port-forward-namespace string   Namespace name which should be used for port forwarding
      --redis-haproxy-name string       Name of the Redis HA Proxy; set this or the ARGOCD_REDIS_HAPROXY_NAME environment variable when the HA Proxy's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis-ha-haproxy")
      --redis-name string               Name of the Redis deployment; set this or the ARGOCD_REDIS_NAME environment variable when the Redis's name label differs from the default, for example when installing via the Helm chart (default "argocd-redis")
      --repo-server-name string         Name of the Argo CD Repo server; set this or the ARGOCD_REPO_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-repo-server")
      --server string                   Argo CD server address
      --server-crt string               Server certificate file
      --server-name string              Name of the Argo CD API server; set this or the ARGOCD_SERVER_NAME environment variable when the server's name label differs from the default, for example when installing via the Helm chart (default "argocd-server")

Use "argocd repocreds [command] --help" for more information about a command.

myk8scluster ~ ✖ argocd repocreds list
URL PATTERN  USERNAME  SSH_CREDS  TLS_CREDS

myk8scluster ~ ➜  argocd repo list
TYPE  NAME  REPO  INSECURE  OCI  LFS  CREDS  STATUS  MESSAGE  PROJECT

myk8scluster ~ ➜  argocd 
```

