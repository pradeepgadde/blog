---
layout: single
title:  "Kubectl Plugins Manager: Krew"
date:   2022-07-20 04:55:04 +0530
categories: Kubernetes
tags: minikube 
classes: wide
show_date: true
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  teaser: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"

author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---

macOS
Bash or ZSH shells
Make sure that `git` is installed.

Run this command to download and install krew:

```sh
(base) pradeep:~$(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
+-zsh:16> mktemp -d
+-zsh:16> cd /var/folders/cf/vzmh318x285f0c1sbsnm14m40000gn/T/tmp.8NLbI0Fr
+-zsh:17> OS=+-zsh:17> uname
+-zsh:17> OS=+-zsh:17> tr '[:upper:]' '[:lower:]'
+-zsh:17> OS=darwin 
+-zsh:18> ARCH=+-zsh:18> uname -m
+-zsh:18> ARCH=+-zsh:18> sed -e s/x86_64/amd64/ -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/'
+-zsh:18> ARCH=amd64 
+-zsh:19> KREW=krew-darwin_amd64 
+-zsh:20> curl -fsSLO https://github.com/kubernetes-sigs/krew/releases/latest/download/krew-darwin_amd64.tar.gz
+-zsh:21> tar zxvf krew-darwin_amd64.tar.gz
x ./LICENSE
x ./krew-darwin_amd64
+-zsh:22> ./krew-darwin_amd64 install krew
Adding "default" plugin index from https://github.com/kubernetes-sigs/krew-index.git.
Updated the local copy of plugin index.
Installing plugin: krew
Installed plugin: krew
\
 | Use this plugin:
 | 	kubectl krew
 | Documentation:
 | 	https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  | 
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  | 
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  | 
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  | 
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/
(base) pradeep:~$
```

Update your .bashrc or .zshrc file and append the following line:

`export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"`



```sh
(base) pradeep:~$kubectl krew
krew is the kubectl plugin manager.
You can invoke krew through kubectl: "kubectl krew [command]..."

Usage:
  kubectl krew [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  index       Manage custom plugin indexes
  info        Show information about an available plugin
  install     Install kubectl plugins
  list        List installed kubectl plugins
  search      Discover kubectl plugins
  uninstall   Uninstall plugins
  update      Update the local copy of the plugin index
  upgrade     Upgrade installed plugins to newer versions
  version     Show krew version and diagnostics

Flags:
  -h, --help      help for krew
  -v, --v Level   number for the log level verbosity

Use "kubectl krew [command] --help" for more information about a command.
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl krew list 
PLUGIN  VERSION
krew    v0.4.3
(base) pradeep:~$kubectl krew info 
accepts 1 arg(s), received 0
(base) pradeep:~$kubectl krew update
Updated the local copy of plugin index.
(base) pradeep:~$
```


```sh
(base) pradeep:~$kubectl krew search
NAME                            DESCRIPTION                                         INSTALLED
access-matrix                   Show an RBAC access matrix for server resources     no
accurate                        Manage Accurate, a multi-tenancy controller         no
advise-policy                   Suggests PodSecurityPolicies and OPA Policies f...  no
advise-psp                      Suggests PodSecurityPolicies for cluster.           no
allctx                          Run commands on contexts in your kubeconfig         no
apparmor-manager                Manage AppArmor profiles for cluster.               no
assert                          Assert Kubernetes resources                         no
auth-proxy                      Authentication proxy to a pod or service            no
aws-auth                        Manage aws-auth ConfigMap                           no
azad-proxy                      Generate and handle authentication for azad-kub...  no
bd-xray                         Run Black Duck Image Scans                          no
blame                           Show who edited resource fields.                    no
bulk-action                     Do bulk actions on Kubernetes resources.            no
ca-cert                         Print the PEM CA certificate of the current clu...  no
capture                         Triggers a Sysdig capture to troubleshoot the r...  no
cert-manager                    Manage cert-manager resources inside your cluster   no
change-ns                       View or change the current namespace via kubectl.   no
cilium                          Easily interact with Cilium agents.                 no
cluster-group                   Exec commands across a group of contexts.           no
clusternet                      Wrap multiple kubectl calls to Clusternet           no
cm                              Provides commands for OCM/MCE/ACM.                  no
cnpg                            Manage your CloudNativePG clusters                  no
config-cleanup                  Automatically clean up your kubeconfig              no
config-registry                 Switch between registered kubeconfigs               no
cost                            View cluster cost information                       no
creyaml                         Generate custom resource YAML manifest              unavailable on darwin/amd64
ctx                             Switch between contexts in your kubeconfig          no
custom-cols                     A "kubectl get" replacement with customizable c...  no
cyclonus                        NetworkPolicy analysis tool suite                   no
datadog                         Manage the Datadog Operator                         no
datree                          Scan your cluster resources for misconfigurations   no
dds                             Detect if workloads are mounting the docker socket  no
debug-shell                     Create pod with interactive kube-shell.             no
deprecations                    Checks for deprecated objects in a cluster          no
df-pv                           Show disk usage (like unix df) for persistent v...  no
direct-csi                      CSI driver to manage drives in k8s cluster as v...  no
directpv                        Deploys and manages the lifecycle of DirectPV C...  no
doctor                          Scans your cluster and reports anomalies.           no
dtlogin                         Login to a cluster via openid-connect               unavailable on darwin/amd64
duck                            List custom resources with ducktype support         no
edit-status                     Edit /status subresources of CRs                    no
eds                             Interact and manage ExtendedDaemonset resources     no
eksporter                       Export resources and removes a pre-defined set ...  no
emit-event                      Emit Kubernetes Events for the requested object     no
evict-pod                       Evicts the given pod                                no
example                         Prints out example manifest YAMLs                   no
exec-as                         Like kubectl exec, but offers a `user` flag to ...  no
exec-cronjob                    Run a CronJob immediately as Job                    no
explore                         A better kubectl explain with the fuzzy finder      no
fields                          Grep resources hierarchy by field name              no
flame                           Generate CPU flame graphs from pods                 no
fleet                           Shows config and resources of a fleet of clusters   no
flyte                           Monitor, launch and manage flyte executions         no
fuzzy                           Fuzzy and partial string search for kubectl         no
gadget                          Gadgets for debugging and introspecting apps        no
get-all                         Like `kubectl get all` but _really_ everything      no
gke-credentials                 Fetch credentials for GKE clusters                  no
gopass                          Imports secrets from gopass                         no
graph                           Visualize Kubernetes resources and relationships.   no
grep                            Filter Kubernetes resources by matching their n...  no
gs                              Handle custom resources with Giant Swarm            no
hlf                             Deploy and manage Hyperledger Fabric components     no
hns                             Manage hierarchical namespaces (part of HNC)        no
htpasswd                        Create nginx-ingress compatible basic-auth secrets  no
ice                             View configuration settings of containers insid...  no
iexec                           Interactive selection tool for `kubectl exec`       no
images                          Show container images used in the cluster.          no
ingress-nginx                   Interact with ingress-nginx                         no
ingress-rule                    Update Ingress rules via command line               no
ipick                           A kubectl wrapper for interactive resource sele...  no
istiolog                        Manipulate istio-proxy logging level without is...  no
janitor                         Lists objects in a problematic state                no
kadalu                          Manage Kadalu Operator, CSI and Storage pods        no
karbon                          Connect to Nutanix Karbon cluster                   no
karmada                         Manage clusters with Karmada federation.            no
konfig                          Merge, split or import kubeconfig files             no
krew                            Package manager for kubectl plugins.                yes
kruise                          Easily handle OpenKruise workloads                  no
ks                              Simple management of KubeSphere components          no
ktop                            A top tool to display workload metrics              no
kubesec-scan                    Scan Kubernetes resources with kubesec.io.          no
kudo                            Declaratively build, install, and run operators...  no
kuota-calc                      Calculate needed quota to perform rolling updates.  no
kurt                            Find what's restarting and why                      no
kuttl                           Declaratively run and test operators                no
kyverno                         Kyverno is a policy engine for kubernetes           no
lineage                         Display all dependent resources or resource dep...  no
linstor                         View and manage LINSTOR storage resources           no
liqo                            Install and manage Liqo on your clusters            no
log2rbac                        Fine-tune your RBAC using log2rbac operator         no
match-name                      Match names of pods and other API objects           no
mc                              Run kubectl commands against multiple clusters ...  no
minio                           Deploy and manage MinIO Operator and Tenant(s)      no
moco                            Interact with MySQL operator MOCO.                  no
modify-secret                   modify secret with implicit base64 translations     no
mtail                           Tail logs from multiple pods matching label sel...  no
multiforward                    Port Forward to multiple Kubernetes Services        no
multinet                        Shows pods' network-status of multi-net-spec        no
neat                            Remove clutter from Kubernetes manifests to mak...  no
net-forward                     Proxy to arbitrary TCP services on a cluster ne...  no
node-admin                      List nodes and run privileged pod with chroot       no
node-restart                    Restart cluster nodes sequentially and gracefully   no
node-shell                      Spawn a root shell on a node via kubectl            no
np-viewer                       Network Policies rules viewer                       no
ns                              Switch between Kubernetes namespaces                no
nsenter                         Run shell command in Pod's namespace on the nod...  no
oidc-login                      Log in to the OpenID Connect provider               no
open-svc                        Open the Kubernetes URL(s) for the specified se...  no
openebs                         View and debug OpenEBS storage resources            no
operator                        Manage operators with Operator Lifecycle Manager    no
oulogin                         Login to a cluster via OpenUnison                   no
outdated                        Finds outdated container images running in a cl...  no
passman                         Store kubeconfig credentials in keychains or pa...  no
pexec                           Execute process with privileges in a pod            no
pod-dive                        Shows a pod's workload tree and info inside a node  no
pod-inspect                     Get all of a pod's details at a glance              no
pod-lens                        Show pod-related resources                          no
pod-logs                        Display a list of pods to get logs from             no
pod-shell                       Display a list of pods to execute a shell in        no
podevents                       Show events for pods                                no
popeye                          Scans your clusters for potential resource issues   no
preflight                       Executes application preflight tests in a cluster   no
print-env                       Build config files from k8s environments.           no
profefe                         Gather and manage pprof profiles from running pods  no
promdump                        Dumps the head and persistent blocks of Prometh...  no
prompt                          Prompts for user confirmation when executing co...  no
prune-unused                    Prune unused resources                              no
psp-util                        Manage Pod Security Policy(PSP) and the related...  no
pv-migrate                      Migrate data across persistent volumes              no
pvmigrate                       Migrates PVs between StorageClasses                 no
rabbitmq                        Manage RabbitMQ clusters                            no
rbac-lookup                     Reverse lookup for RBAC                             no
rbac-tool                       Plugin to analyze RBAC permissions and generate...  no
rbac-view                       A tool to visualize your RBAC permissions.          no
realname-diff                   Diffs live and local resources ignoring Kustomi...  no
reap                            Delete unused Kubernetes resources.                 no
relay                           Drop-in "port-forward" replacement with UDP and...  no
reliably                        Surfaces reliability issues in Kubernetes           no
rename-pvc                      Rename a PersistentVolumeClaim (PVC)                no
resource-capacity               Provides an overview of resource requests, limi...  no
resource-snapshot               Prints a snapshot of nodes, pods and HPAs resou...  no
resource-versions               Print supported API resource versions               no
restart                         Restarts a pod with the given name                  no
rm-standalone-pods              Remove all pods without owner references            no
rolesum                         Summarize RBAC roles for subjects                   no
roll                            Rolling restart of all persistent pods in a nam...  no
rook-ceph                       Rook plugin for Ceph management                     no
safe                            Prompts before running edit commands                no
schemahero                      Declarative database schema migrations via YAML     no
score                           Kubernetes static code analysis.                    no
secretdata                      Viewing decoded Secret data with search flags       no
service-tree                    Status for ingresses, services, and their backends  no
shovel                          Gather diagnostics for .NET Core applications       no
sick-pods                       Find and debug Pods that are "Not Ready"            no
skew                            Find if your cluster/kubectl version is skewed      no
slice                           Split a multi-YAML file into individual files.      no
snap                            Delete half of the pods in a namespace or cluster   no
sniff                           Start a remote packet capture on pods using tcp...  no
socks5-proxy                    SOCKS5 proxy to Services or Pods in the cluster     no
sort-manifests                  Sort manifest files in a proper order by Kind       no
split-yaml                      Split YAML output into one file per resource.       no
spy                             pod debugging tool for kubernetes clusters with...  no
sql                             Query the cluster via pseudo-SQL                    unavailable on darwin/amd64
ssh-jump                        Access nodes or services using SSH jump Pod         no
sshd                            Run SSH server in a Pod                             no
ssm-secret                      Import/export secrets from/to AWS SSM param store   no
starboard                       Toolkit for finding risks in kubernetes resources   no
status                          Show status details of a given resource.            no
stern                           Multi pod and container log tailing                 no
strace                          Capture strace logs from a running workload         no
sudo                            Run Kubernetes commands impersonated as group s...  no
support-bundle                  Creates support bundles for off-cluster analysis    no
switch-config                   Switches between kubeconfig files                   no
tail                            Stream logs from multiple pods and containers u...  no
tap                             Interactively proxy Kubernetes Services with ease   no
tmux-exec                       An exec multiplexer using Tmux                      no
topology                        Explore region topology for nodes or pods           no
trace                           Trace Kubernetes pods and nodes with system tools   no
tree                            Show a tree of object hierarchies through owner...  no
tunnel                          Reverse tunneling between cluster and your machine  no
unused-volumes                  List unused PVCs                                    no
vela                            Easily interact with KubeVela                       no
view-allocations                List allocations per resources, nodes, pods.        no
view-cert                       View certificate information stored in secrets      no
view-secret                     Decode Kubernetes secrets                           no
view-serviceaccount-kubeconfig  Show a kubeconfig setting to access the apiserv...  no
view-utilization                Shows cluster cpu and memory utilization            no
view-webhook                    Visualize your webhook configurations               no
viewnode                        Displays nodes with their pods and containers a...  no
virt                            Control KubeVirt virtual machines using virtctl     no
volsync                         Manage replication with the VolSync operator        unavailable on darwin/amd64
vpa-recommendation              Compare VPA recommendations to actual resources...  no
warp                            Sync and execute local files in Pod                 no
whisper-secret                  Create secrets with improved privacy                no
who-can                         Shows who has RBAC permissions to access Kubern...  no
whoami                          Show the subject that's currently authenticated...  no
windows-debug                   Windows node access via kubectl                     no
(base) pradeep:~$

```

Install a plugin called `example` 
```sh
(base) pradeep:~$kubectl krew install example
Updated the local copy of plugin index.
Installing plugin: example
Installed plugin: example
\
 | Use this plugin:
 | 	kubectl example
 | Documentation:
 | 	https://github.com/seredot/kubectl-example
/
WARNING: You installed plugin "example" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
(base) pradeep:~$
```

```sh
(base) pradeep:~$kubectl plugin list
The following compatible plugins are available:

/Users/pradeep/.krew/bin/kubectl-example
/Users/pradeep/.krew/bin/kubectl-krew
/usr/local/bin/kubectl-demo
Unable to read directory "/Applications/VMware Fusion.app/Contents/Public" from your PATH: open /Applications/VMware Fusion.app/Contents/Public: no such file or directory. Skipping...
(base) pradeep:~$
```


```sh
(base) pradeep:~$kubectl example
Not enough arguments
Usage: example <RESOURCE_NAME>
(base) pradeep:~$
```

```yaml
(base) pradeep:~$kubectl example pod
---
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      resources:
        limits:
          memory: '1Gi'
          cpu: '800m'
        requests:
          memory: '700Mi'
          cpu: '400m'
  # These containers are run during pod initialization
  initContainers:
    - name: install
      image: busybox
      command:
        - wget
        - '-O'
        - '/work-dir/index.html'
        - http://kubernetes.io
      volumeMounts:
        - name: workdir
          mountPath: '/work-dir'
  dnsPolicy: Default
  volumes:
    - name: workdir
      emptyDir: {}
(base) pradeep:~$
```
Another example 
```yaml
(base) pradeep:~$kubectl example Deployment
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
(base) pradeep:~$
````