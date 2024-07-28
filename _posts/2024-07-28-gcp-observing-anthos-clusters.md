---

layout: single
title:  "Observing Anthos clusters on bare metal"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Observing Anthos clusters on bare metal

In this lab, you explore how to collect and analyze telemetry from Anthos clusters on bare metal deployments and services.

You leverage Google Cloud's operations suite to do so. Anthos clusters on bare metal write their logs into Cloud Logging and their metrics into Cloud Monitoring. Once the data is saved in the cloud, you can use Google Cloud's operations suite's dashboards to understand and troubleshoot service performance, and have the evolution history over time. In addition, you install the Anthos Service Mesh to collect tracing information automatically.

In this lab, you run Anthos clusters on bare metal atop of GCE VMs. This does require a little extra work as the load balancer VMs need Layer 2 connectivity, so the VMs have been created and configured to use VXLAN, which encapsulates Layer 2 connections on a Layer 3 network. In a pure bare metal deployment, you would just skip this step and everything else would remain the same.

- Learn how to configure the level of logging in your Anthos on bare metal clusters.
- Evaluate service performance using Cloud Monitoring and Cloud Logging features within Google Cloud.
- Install Anthos Service Mesh and investigate the telemetry in Cloud Tracing.
- Enable Node Problem Detector to detect issues in your cluster nodes.

## Explore the pre-created environment

This environment has been pre-configured with a hybrid Anthos cluster on bare metal running on Google Cloud's Compute Engine VMs. The hybrid  deployment model is a specialized multi-cluster deployment that enables  to run user workloads on your admin cluster. In this type of deployment, you could add more user clusters to support multiple teams or workload  types.

The pre-created VMs have the following functions:

- **setup-vm**: a vm used to create the rest of the infrastructure. Check out the startup_script to check the steps taken to create the infrastructure and install the Anthos software.
- **abm-ws**: an admin workstation in the same network as the cluster to perfom the configuration, creation, and management of the Anthos cluster on bare metal.
- **abm-user-cp1**: a control plane nodes.
- **abm-user-w1 & abm-user-w2**: two worker nodes.

1. Go to **Navigation menu > Kubernetes Engine > Clusters** and verify that you have a single Anthos cluster registered. 

## Log in to your Anthos cluster

When you create Anthos clusters in Google Cloud, AWS, or VMware, you typically use an environment-specific installation process that takes advantage of native APIs.

When you create a bare metal cluster, the installation process doesn't automatically create machines for you (typically, they are physical machines so they can't be created out of thin air). That doesn't mean, however, that you can't create "bare metal" clusters running on VMs in any of those environments.

In this lab, the "bare metal" cluster has been created on GCE VMs. It will behave almost identically to a bare metal cluster running on physical devices in your data center. Where the administration in the lab deviates from a pure bare metal scenario, the lab instructions will make it clear.

The cluster in the lab (see diagram below) is made of a single master node and two worker nodes. In a production environment, you might consider using three nodes for high availability of both the data and the control plane.

![Hybrid Cluster diagram](https://cdn.qwiklabs.com/k%2BGMcOBcEIIjRg3h00pkBJKcFEvansXeGJhWMRePN4M%3D)

```sh
student_04_31eb84f42862@cloudshell:~ (qwiklabs-gcp-00-6b98c2173360)$ ZONE=us-central1-b
student_04_31eb84f42862@cloudshell:~ (qwiklabs-gcp-00-6b98c2173360)$ gcloud compute ssh --ssh-flag="-A" root@abm-ws \
    --zone $ZONE \
    --tunnel-through-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_04_31eb84f42862/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_04_31eb84f42862/.ssh/google_compute_engine
Your public key has been saved in /home/student_04_31eb84f42862/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:oRWP/U924pMlvcxhuLsmsUUlgS5by1kYML4MLAMdJUo student_04_31eb84f42862@cs-421171858362-default
The key's randomart image is:
+---[RSA 3072]----+
|   E.oo..o. ...  |
|  . o.o .=.o . . |
|   . o o+.+ o o  |
|      oooo.= o.. |
|      . So= *.=o+|
|         . = *=*o|
|            +.=+ |
|           o ... |
|            oo.  |
+----[SHA256]-----+
Updating project ssh metadata...                                                                                                            
Updating project ssh metadata...working.                                                                                                    
Updating project ssh metadata...working..                                                                                                   
Updating project ssh metadata...working...                                                                                                  
Updating project ssh metadata...working...Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-6b98c2173360].            
Updating project ssh metadata...done.                                                                                                       
Waiting for SSH key to propagate.
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Warning: Permanently added 'compute.3410199921532884116' (ED25519) to the list of known hosts.
WARNING: 

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1062-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Jul 28 15:36:06 UTC 2024

  System load:  0.0                Processes:             128
  Usage of /:   1.4% of 247.92GB   Users logged in:       0
  Memory usage: 2%                 IPv4 address for ens4: 10.1.0.3
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


root@abm-ws:~# 
```



```sh
root@abm-ws:~# cd baremetal
root@abm-ws:~/baremetal# export CLUSTER_ID=abm-hybrid-cluster
root@abm-ws:~/baremetal# export KUBECONFIG=$HOME/baremetal/bmctl-workspace/$CLUSTER_ID/$CLUSTER_ID-kubeconfig
root@abm-ws:~/baremetal# kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION            INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
abm-user-cp1   Ready    control-plane   8h    v1.26.2-gke.1001   10.200.0.3    <none>        Ubuntu 20.04.6 LTS   5.15.0-1062-gcp   containerd://1.6.18-gke.0
abm-user-w1    Ready    worker          8h    v1.26.2-gke.1001   10.200.0.4    <none>        Ubuntu 20.04.6 LTS   5.15.0-1062-gcp   containerd://1.6.18-gke.0
abm-user-w2    Ready    worker          8h    v1.26.2-gke.1001   10.200.0.5    <none>        Ubuntu 20.04.6 LTS   5.15.0-1062-gcp   containerd://1.6.18-gke.0
root@abm-ws:~/baremetal# 
```

Look at the configuration file used to create the Anthos cluster on bare metal and check that the desired options matched the current  configuration:

```yaml
root@abm-ws:~/baremetal# cat bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster.yaml
# bmctl configuration variables. Because this section is valid YAML but not a valid Kubernetes
# resource, this section can only be included when using bmctl to
# create the initial admin/hybrid cluster. Afterwards, when creating user clusters by directly
# applying the cluster and node pool resources to the existing cluster, you must remove this
# section.
gcrKeyPath: bmctl-workspace/.sa-keys/qwiklabs-gcp-00-6b98c2173360-anthos-baremetal-gcr.json
sshPrivateKeyPath: /root/.ssh/id_rsa
gkeConnectAgentServiceAccountKeyPath: bmctl-workspace/.sa-keys/qwiklabs-gcp-00-6b98c2173360-anthos-baremetal-connect.json
gkeConnectRegisterServiceAccountKeyPath: bmctl-workspace/.sa-keys/qwiklabs-gcp-00-6b98c2173360-anthos-baremetal-register.json
cloudOperationsServiceAccountKeyPath: bmctl-workspace/.sa-keys/qwiklabs-gcp-00-6b98c2173360-anthos-baremetal-cloud-ops.json
---
apiVersion: v1
kind: Namespace
metadata:
  name: cluster-abm-hybrid-cluster
---
# Cluster configuration. Note that some of these fields are immutable once the cluster is created.
# For more info, see https://cloud.google.com/anthos/clusters/docs/bare-metal/1.15/reference/cluster-config-ref#cluster_configuration_fields
apiVersion: baremetal.cluster.gke.io/v1
kind: Cluster
metadata:
  name: abm-hybrid-cluster
  namespace: cluster-abm-hybrid-cluster
spec:
  # Cluster type. This can be:
  #   1) admin:  to create an admin cluster. This can later be used to create user clusters.
  #   2) user:   to create a user cluster. Requires an existing admin cluster.
  #   3) hybrid: to create a hybrid cluster that runs admin cluster components and user workloads.
  #   4) standalone: to create a cluster that manages itself, runs user workloads, but does not manage other clusters.
  type: hybrid
  # Cluster profile. This can be either 'default' or 'edge'.
  # The edge profile is tailored for deployments on the edge locations
  # and should be used together with the 'standalone' cluster type.
  profile: default
  # Anthos cluster version.
  anthosBareMetalVersion: 1.15.0
  # GKE connect configuration
  gkeConnect:
    projectID: qwiklabs-gcp-00-6b98c2173360
  # Control plane configuration
  controlPlane:
    nodePoolSpec:
      nodes:
      # Control plane node pools. Typically, this is either a single machine
      # or 3 machines if using a high availability deployment.
      - address: 10.200.0.3
  # Cluster networking configuration
  clusterNetwork:
    # Pods specify the IP ranges from which pod networks are allocated.
    pods:
      cidrBlocks:
      - 192.168.0.0/16
    # Services specify the network ranges from which service virtual IPs are allocated.
    # This can be any RFC1918 range that does not conflict with any other IP range
    # in the cluster and node pool resources.
    services:
      cidrBlocks:
      - 10.96.0.0/20
  # Load balancer configuration
  loadBalancer:
    # Load balancer mode can be either 'bundled' or 'manual'.
    # In 'bundled' mode a load balancer will be installed on load balancer nodes during cluster creation.
    # In 'manual' mode the cluster relies on a manually-configured external load balancer.
    mode: bundled
    # Load balancer port configuration
    ports:
      # Specifies the port the load balancer serves the Kubernetes control plane on.
      # In 'manual' mode the external load balancer must be listening on this port.
      controlPlaneLBPort: 443
    # There are two load balancer virtual IP (VIP) addresses: one for the control plane
    # and one for the L7 Ingress service.
    # If you use Layer2 load balancing, the VIPs must be in the same subnet as the load balancer nodes.
    # If you use bundled BGP-based load balancing (mode: 'bundled' and type: 'bgp'), the VIPs
    # must not come from the same subnet as any of the nodes in the cluster.
    # These IP addresses do not correspond to physical network interfaces.
    vips:
      # ControlPlaneVIP specifies the VIP to connect to the Kubernetes API server.
      # This address must not be in the address pools below.
      controlPlaneVIP: 10.200.0.99
      # IngressVIP specifies the VIP shared by all services for ingress traffic.
      # Allowed only in non-admin clusters.
      # This address must be in the address pools below.
      ingressVIP: 10.200.0.100
    # AddressPools is a list of non-overlapping IP ranges for the data plane load balancer.
    # All addresses must be in the same subnet as the load balancer nodes.
    # Address pool configuration is only valid for 'bundled' LB mode in non-admin clusters.
    addressPools:
    - name: pool1
      addresses:
    #   # Each address must be either in the CIDR form (1.2.3.0/24)
    #   # or range form (1.2.3.1-1.2.3.5).
      - 10.200.0.100-10.200.0.200
    # A load balancer node pool can be configured to specify nodes used for load balancing.
    # These nodes are part of the Kubernetes cluster and run regular workloads as well as load balancers.
    # If the node pool config is absent then the control plane nodes are used.
    # Node pool configuration is only valid for 'bundled' LB mode.
    # nodePoolSpec:
    #  nodes:
    #  - address: 10.200.0.3
  # Proxy configuration
  # proxy:
  #   url: http://[username:password@]domain
  #   # A list of IPs, hostnames or domains that should not be proxied.
  #   noProxy:
  #   - 127.0.0.1
  #   - localhost
  # Logging and Monitoring
  clusterOperations:
    # Cloud project for logs and metrics.
    projectID: qwiklabs-gcp-00-6b98c2173360
    # Cloud location for logs and metrics.
    location: us-central1
    # Whether collection of application logs/metrics should be enabled (in addition to
    # collection of system logs/metrics which correspond to system components such as
    # Kubernetes control plane or cluster management agents).
    enableApplication: true
  # Storage configuration
  storage:
    # lvpNodeMounts specifies the config for local PersistentVolumes backed by mounted disks.
    # These disks need to be formatted and mounted by the user, which can be done before or after
    # cluster creation.
    lvpNodeMounts:
      # path specifies the host machine path where mounted disks will be discovered and a local PV
      # will be created for each mount.
      path: /mnt/localpv-disk
      # storageClassName specifies the StorageClass that PVs will be created with. The StorageClass
      # is created during cluster creation.
      storageClassName: local-disks
    # lvpShare specifies the config for local PersistentVolumes backed by subdirectories in a shared filesystem.
    # These subdirectories are automatically created during cluster creation.
    lvpShare:
      # path specifies the host machine path where subdirectories will be created on each host. A local PV
      # will be created for each subdirectory.
      path: /mnt/localpv-share
      # storageClassName specifies the StorageClass that PVs will be created with. The StorageClass
      # is created during cluster creation.
      storageClassName: local-shared
      # numPVUnderSharedPath specifies the number of subdirectories to create under path.
      numPVUnderSharedPath: 5
  # NodeConfig specifies the configuration that applies to all nodes in the cluster.
  nodeConfig:
    # podDensity specifies the pod density configuration.
    podDensity:
      # maxPodsPerNode specifies at most how many pods can be run on a single node.
      maxPodsPerNode: 250
  # Authentication; uncomment this section if you wish to enable authentication to the cluster with OpenID Connect.
  # authentication:
  #   oidc:
  #     # issuerURL specifies the URL of your OpenID provider, such as "https://accounts.google.com". The Kubernetes API
  #     # server uses this URL to discover public keys for verifying tokens. Must use HTTPS.
  #     issuerURL: <URL for OIDC Provider; required>
  #     # clientID specifies the ID for the client application that makes authentication requests to the OpenID
  #     # provider.
  #     clientID: <ID for OIDC client application; required>
  #     # clientSecret specifies the secret for the client application.
  #     clientSecret: <Secret for OIDC client application; optional>
  #     # kubectlRedirectURL specifies the redirect URL (required) for the gcloud CLI, such as
  #     # "http://localhost:[PORT]/callback".
  #     kubectlRedirectURL: <Redirect URL for the gcloud CLI; optional, default is "http://kubectl.redirect.invalid">
  #     # username specifies the JWT claim to use as the username. The default is "sub", which is expected to be a
  #     # unique identifier of the end user.
  #     username: <JWT claim to use as the username; optional, default is "sub">
  #     # usernamePrefix specifies the prefix prepended to username claims to prevent clashes with existing names.
  #     usernamePrefix: <Prefix prepended to username claims; optional>
  #     # group specifies the JWT claim that the provider will use to return your security groups.
  #     group: <JWT claim to use as the group name; optional>
  #     # groupPrefix specifies the prefix prepended to group claims to prevent clashes with existing names.
  #     groupPrefix: <Prefix prepended to group claims; optional>
  #     # scopes specifies additional scopes to send to the OpenID provider as a comma-delimited list.
  #     scopes: <Additional scopes to send to OIDC provider as a comma-separated list; optional>
  #     # extraParams specifies additional key-value parameters to send to the OpenID provider as a comma-delimited
  #     # list.
  #     extraParams: <Additional key-value parameters to send to OIDC provider as a comma-separated list; optional>
  #     # proxy specifies the proxy server to use for the cluster to connect to your OIDC provider, if applicable.
  #     # Example: https://user:password@10.10.10.10:8888. If left blank, this defaults to no proxy.
  #     proxy: <Proxy server to use for the cluster to connect to your OIDC provider; optional, default is no proxy>
  #     # deployCloudConsoleProxy specifies whether to deploy a reverse proxy in the cluster to allow Google Cloud
  #     # Console access to the on-premises OIDC provider for authenticating users. If your identity provider is not
  #     # reachable over the public internet, and you wish to authenticate using Google Cloud Console, then this field
  #     # must be set to true. If left blank, this field defaults to false.
  #     deployCloudConsoleProxy: <Whether to deploy a reverse proxy for Google Cloud Console authentication; optional>
  #     # certificateAuthorityData specifies a Base64 PEM-encoded certificate authority certificate of your identity
  #     # provider. It's not needed if your identity provider's certificate was issued by a well-known public CA.
  #     # However, if deployCloudConsoleProxy is true, then this value must be provided, even for a well-known public
  #     # CA.
  #     certificateAuthorityData: <Base64 PEM-encoded certificate authority certificate of your OIDC provider; optional>
  # Node access configuration; uncomment this section if you wish to use a non-root user
  # with passwordless sudo capability for machine login.
  # nodeAccess:
  #   loginUser: <login user name>
---
# Node pools for worker nodes
apiVersion: baremetal.cluster.gke.io/v1
kind: NodePool
metadata:
  name: hybrid-cluster-pool-1
  namespace: cluster-abm-hybrid-cluster
spec:
  clusterName: abm-hybrid-cluster
  nodes:
  - address: 10.200.0.4
  - address: 10.200.0.5
root@abm-ws:~/baremetal# 
```

### Log in to your Anthos cluster on bare metal from the Console

1. In Cloud Shell, create a Kubernetes Service Account on your cluster and grant it the cluster-admin role:

2. Create a token that you can use to log in to the cluster from the Console

3. Select the token in the SSH session (this will copy the token - don't try to copy with CTRL+C).

   Find the **abm-admin-cluster** entry in the cluster list showing in the Console and click the **Actions** icon (three dots) at the far right of the row

   Select **Log in**, select **Token**, then paste the token from your Clipboard into the provided field. Click **Login**. When you're done, it should look like this:

Congratulations! You have successfully signed in to your Anthos on bare metal hybrid cluster!

## Understand installed observability software stack

- Check all the observability tools installed in the cluster:

```sh
root@abm-ws:~/baremetal# kubectl -n kube-system get pods -l "managed-by=stackdriver"
NAME                                                       READY   STATUS    RESTARTS   AGE
gke-metrics-agent-8kxjj                                    1/1     Running   0          8h
gke-metrics-agent-q8cfx                                    1/1     Running   0          8h
gke-metrics-agent-qsqwk                                    1/1     Running   0          8h
kube-state-metrics-cc8987686-g495r                         1/1     Running   0          8h
node-exporter-9v9l6                                        1/1     Running   0          8h
node-exporter-mf2d5                                        1/1     Running   0          8h
node-exporter-mhcjx                                        1/1     Running   0          8h
stackdriver-log-forwarder-czsjj                            1/1     Running   0          8h
stackdriver-log-forwarder-f87fs                            1/1     Running   0          8h
stackdriver-log-forwarder-w789h                            1/1     Running   0          8h
stackdriver-metadata-agent-cluster-level-fcd8f55f9-h9ppw   1/1     Running   0          8h
root@abm-ws:~/baremetal# 
```

Cloud Logging and Cloud Monitoring are installed and activated in each        cluster when you create a new admin, user, or hybrid cluster. The        Stackdriver agents include several components on each cluster:        



- **Stackdriver Operator** (stackdriver-operator-*). Manages            the lifecycle for all other Stackdriver agents deployed onto the            cluster.
- **Stackdriver Custom Resource.** A resource that is            automatically created as part of the Anthos clusters on bare metal            installation process.
- **Stackdriver Log Forwarder** (stackdriver-log-forwarder-*).            A Fluent Bit daemonset that forwards logs from each machine to the            Cloud Logging. The log Forwarder buffers the log entries on the node            locally and re-sends them for up to 4 hours. If the buffer gets full            or if the Log Forwarder can't reach the Cloud Logging API for more            than 4 hours, logs are dropped.
- **Stackdriver Metadata Collector**            (stackdriver-metadata-agent-). A deployment that sends metadata for            Kubernetes resources such as pods, deployments, or nodes to the            Stackdriver Resource Metadata API; this data is used to enrich            metric queries by enabling you to query by deployment name, node            name, or even Kubernetes service name.



​        You can always disable Cloud Logging and Cloud Monitoring by  deleting the resources using kubectl delete. If you want to use a  third-party service for monitoring your cluster, check out the  documentation to find guides to work with third-party solutions such as  Elastic Stack, Splunk Connect, or Datadog.

## Explore audit logs

1. Verify that the configuration file used to create the Anthos cluster on bare metal does not have the setting to disable the collection of  audit logs: Notice it's an empty output.

```sh
root@abm-ws:~/baremetal# cat bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster.yaml | grep disableCloudAuditLogging:
root@abm-ws:~/baremetal# 
```

Access the audit logs using the **gcloud** cli tool:

```sh
root@abm-ws:~/baremetal# export PROJECT_ID=$(gcloud config get-value project)
root@abm-ws:~/baremetal# echo "gcloud logging read 'logName="projects/${PROJECT_ID}/logs/externalaudit.googleapis.com%2Factivity"
> AND resource.type="k8s_cluster"
> AND protoPayload.serviceName="anthosgke.googleapis.com"'     --limit 2     --freshness 300d" > get_audit_logs.sh
root@abm-ws:~/baremetal# sh get_audit_logs.sh

```

Access the audit logs from the Console by navigating to **Navigation menu > Logging > Logs Explorer** and entering the following query. Then, replace the **PROJECT_ID** in the query with  and click **Run query**:

```sh
logName="projects/PROJECT_ID/logs/externalaudit.googleapis.com%2Factivity"
resource.type="k8s_cluster"
protoPayload.serviceName="anthosgke.googleapis.com"
```

## Explore cluster logs

1. On the **Logs Explorer** screen, remove all filters from the query box and click **Run query**.

Notice on the left side of the screen that you can choose to explore Kubernetes Container, Cluster, and Node logs. If you don't see them, click on the buble named "Log fields" to view them.

1. In the left-side logs field panel, select **Kubernetes Node**.
2. Now select the **abm-user-cp1** node name to further filter the logs.

## Explore application logs

1. Come back to the Cloud Shell window you were using to SSH into the admin workstation. Verify that the configuration file used to create the Anthos cluster on bare metal has the setting enabled to collect metrics and logs from our applications:

```sh
root@abm-ws:~/baremetal# cat bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster.yaml | grep enableApplication:
    enableApplication: true
root@abm-ws:~/baremetal# 
```

Create a hello-app application and a Kubernetes Service of type LoadBalancer to access the app:

```sh
root@abm-ws:~/baremetal# kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
deployment.apps/hello-app created
root@abm-ws:~/baremetal# kubectl expose deployment hello-app --name hello-app-service --type LoadBalancer --port 80 --target-port=8080
service/hello-app-service exposed
root@abm-ws:~/baremetal# 
```

Get the IP for this workload:

```sh
root@abm-ws:~/baremetal# kubectl get svc
NAME                TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
hello-app-service   LoadBalancer   10.96.15.125   10.200.0.101   80:31517/TCP   24s
kubernetes          ClusterIP      10.96.0.1      <none>         443/TCP        8h
root@abm-ws:~/baremetal# 
```

Access the public IP provided by the `hello-app-service` by running the below command multiple times.

```sh
root@abm-ws:~/baremetal# curl 10.200.0.101
Hello, world!
Version: 2.0.0
Hostname: hello-app-75d4498bb8-92766
root@abm-ws:~/baremetal# 
```

In the Console, go to **Navigation menu > Logging > Logs Explorer** and enter the following query and click **Run query**:

Notice that a log was created every time that you ran the **curl** command.

```json
{
  "insertId": "l48pxjfif6f3f",
  "jsonPayload": {
    "logtag": "F",
    "log": "2024/07/28 15:50:21 Serving request: /"
  },
  "resource": {
    "type": "k8s_container",
    "labels": {
      "pod_name": "hello-app-75d4498bb8-92766",
      "namespace_name": "default",
      "cluster_name": "abm-hybrid-cluster",
      "container_name": "hello-app",
      "project_id": "qwiklabs-gcp-00-6b98c2173360",
      "location": "us-central1"
    }
  },
  "timestamp": "2024-07-28T15:50:21.616951031Z",
  "labels": {
    "k8s-pod/pod-template-hash": "75d4498bb8",
    "k8s-pod/app": "hello-app"
  },
  "logName": "projects/qwiklabs-gcp-00-6b98c2173360/logs/stderr",
  "receiveTimestamp": "2024-07-28T15:50:22.384543464Z"
}
```

## Explore cluster metrics

1. In the Console, go to **Navigation menu > Monitoring > Dashboards**. Notice that the Anthos on bare metal installer has created some dashboards for you already.

Click the **Anthos cluster control plane uptime** to see the availability of your cluster. It should look something like this.

Explore the other dashboards and the metrics available.

## Explore application metrics

1. In the Console, go to **Navigation menu > Monitoring > Dashboards**.

2. In the dashboards list, locate the **Anthos cluster pod status** and click **Open dashboard settings icon > Copy Dashboard**.

3. Name it `Hello App Anthos cluster pod status` and click **Copy**.

4. Click the newly created dashboard. You see that it's gathering metrics for all pods in your cluster for a variety of metrics. Let's change it so that it only monitors the `hello-app` pod that we created earlier. For that, click the **Edit Dashboard** button the top right of the screen.

5. Change all the metrics to only monitor your `hello-app` pod. For that, select one of the graphs and see how an Options tab opens on the left side of the screen. Remove all filters, and create a single filter with the following details. Then, click **Done** and do the same in the next graph.

   

   Congratulations! You have successfully created a dashboard in Google  Cloud to monitor your hello-app pod running on your on-premises cluster.

## Install Anthos Service Mesh

1. Come back to the Cloud Shell window you were using to SSH into the admin workstation. Create an overlay file that will be used in the Anthos Service Mesh installation to enable Cloud Trace:

```sh
root@abm-ws:~/baremetal# cat <<'EOF' > cloud-trace.yaml
> apiVersion: install.istio.io/v1alpha1
> kind: IstioOperator
> spec:
>     meshConfig:
>         enableTracing: true
>     values:
>             global:
>                 proxy:
>                         tracer: stackdriver
> EOF
root@abm-ws:~/baremetal# 
```

Download the asmcli tool to install Anthos Service Mesh on your cluster:

```sh
root@abm-ws:~/baremetal# curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.15 > asmcli
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  182k  100  182k    0     0  1394k      0 --:--:-- --:--:-- --:--:-- 1394k
root@abm-ws:~/baremetal# chmod +x asmcli
root@abm-ws:~/baremetal# 
```

Clean up the environment variables to make sure there are no conflicts with Kubeconfig when installing Anthos Service Mesh:

```sh
root@abm-ws:~/baremetal# export PROJECT_ID=
root@abm-ws:~/baremetal# export CLUSTER_NAME=
root@abm-ws:~/baremetal# export CLUSTER_LOCATION=
root@abm-ws:~/baremetal# 
```

Install Anthos Service Mesh in your cluster using the Cloud Trace overlay:

```sh
root@abm-ws:~/baremetal# ./asmcli install \
>     --kubeconfig ./bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster-kubeconfig \
>     --fleet_id $FLEET_PROJECT_ID \
>     --output_dir . \
>     --platform multicloud \
>     --enable_all \
>     --ca mesh_ca \
>     --custom_overlay cloud-trace.yaml
asmcli: Reading cluster information for abm-hybrid-cluster-admin@abm-hybrid-cluster
asmcli: Setting up necessary files...
asmcli: Using /root/baremetal/bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster-kubeconfig as the kubeconfig...
asmcli: Checking installation tool dependencies...
asmcli: kubeconfig set to /root/baremetal/bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster-kubeconfig
asmcli: using context abm-hybrid-cluster-admin@abm-hybrid-cluster
asmcli: Getting account information...
asmcli: Downloading kpt..
asmcli: Downloading ASM..
asmcli: Downloading ASM kpt package...
fetching package "/asm" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "asm"
fetching package "/samples" from "https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages" to "samples"
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-00-6b98c2173360
asmcli: Enabling required APIs...
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-00-6b98c2173360
asmcli: Verifying cluster registration.
asmcli: Verified cluster is registered to qwiklabs-gcp-00-6b98c2173360
asmcli: Checking for project qwiklabs-gcp-00-6b98c2173360...
asmcli: Adding labels to abm-hybrid-cluster
authority:
  identityProvider: https://gkehub.googleapis.com/projects/qwiklabs-gcp-00-6b98c2173360/locations/global/memberships/abm-hybrid-cluster
  issuer: https://kubernetes.default.svc.cluster.local
  oidcJwks: eyJrZXlzIjpbeyJ1c2UiOiJzaWciLCJrdHkiOiJSU0EiLCJraWQiOiJzdEpia1JGQ0I2WDd0ODhjUU9FSWRWX1VlRmJwYmVhakRXME1xdnpGNG1FIiwiYWxnIjoiUlMyNTYiLCJuIjoicl9KTzI0WGpoa0hJRTZSMy1RWldTOWZXS1hEay1UYkQ2Tnp1THVJdkdZQmdoXzM1U2s4ZnExYmtnemE1SjQyUldHUXo4elJIVXo0bmtyRy1CV3VLY0dCckZTZEJ5ZWJKTzRfNnJ2WFNNOHFGNV8zS2VnQXNib0U5T3ZVbzl2VThfdmVRQm01OFVkanhNaVdMMTFSQkxtRThIUi1OamJtTUtqSW52Q0ttck9GUTNMTmI4azNnTkdTTDdFZTdyUEE0VnNIdXJBSHNjaTlJTFdidnBLYTF1YXAzdy0xSFZ2WGY3RC0tVGtsRVBqVC16MXNlX3dzT0FsSm41azlaX0xhUktqV3NRS2w1TFVYNEVoZUVZUnJTRlh3WUx0V0w3NDV2a1pxTklUblV1M0ZvUTMzVS05S0RIdVBxSXpSUWxURElxRGh2Q2xZQ0RDVlVWaEsxTjUzTlR3IiwiZSI6IkFRQUIifV19
  workloadIdentityPool: qwiklabs-gcp-00-6b98c2173360.svc.id.goog
createTime: '2024-07-28T07:15:15.483900886Z'
description: abm-hybrid-cluster
endpoint:
  kubernetesMetadata:
    kubernetesApiServerVersion: v1.26.2-gke.1001
    memoryMb: 33526
    nodeCount: 2
    nodeProviderId: baremetal
    updateTime: '2024-07-28T16:06:40.489873988Z'
    vcpuCount: 8
  kubernetesResource:
    resourceOptions:
      connectVersion: 20230224-00-00
      k8sVersion: '1.26'
externalId: d32d1228-bae4-46d8-9a8c-9bc53b7bc29b
labels:
  mesh_id: proj-1061033565780
lastConnectionTime: '2024-07-28T16:06:47.377415161Z'
name: projects/qwiklabs-gcp-00-6b98c2173360/locations/global/memberships/abm-hybrid-cluster
state:
  code: READY
uniqueId: 2268ee2b-0753-4b5e-a5eb-9f7f3648cc7a
updateTime: '2024-07-28T16:07:02.476725940Z'
asmcli: Querying for core/account...
asmcli: Binding abm-builder-sa@qwiklabs-gcp-00-6b98c2173360.iam.gserviceaccount.com to cluster admin role...
clusterrolebinding.rbac.authorization.k8s.io/abm-builder-sa-cluster-admin-binding created
asmcli: Creating istio-system namespace...
namespace/istio-system created
asmcli: Initializing meshconfig API...
asmcli: Cluster has Membership ID abm-hybrid-cluster in the Hub of project qwiklabs-gcp-00-6b98c2173360
asmcli: Binding serviceAccount:abm-builder-sa@qwiklabs-gcp-00-6b98c2173360.iam.gserviceaccount.com to required IAM roles...
asmcli: Configuring kpt package...
asm/
set 19 field(s) of setter "gcloud.core.project" to value "qwiklabs-gcp-00-6b98c2173360"
asm/
set 2 field(s) of setter "gcloud.project.projectNumber" to value "1061033565780"
asm/
set 16 field(s) of setter "gcloud.container.cluster" to value "abm-hybrid-cluster"
asm/
set 16 field(s) of setter "gcloud.compute.location" to value "global"
asm/
set 1 field(s) of setter "gcloud.compute.network" to value "default"
asm/
set 3 field(s) of setter "gcloud.project.environProjectNumber" to value "1061033565780"
asm/
set 2 field(s) of setter "anthos.servicemesh.rev" to value "asm-1157-23"
asm/
set 3 field(s) of setter "anthos.servicemesh.tag" to value "1.15.7-asm.23"
asm/
set 3 field(s) of setter "anthos.servicemesh.trustDomain" to value "qwiklabs-gcp-00-6b98c2173360.svc.id.goog"
asm/
set 1 field(s) of setter "anthos.servicemesh.tokenAudiences" to value "istio-ca,qwiklabs-gcp-00-6b98c2173360.svc.id.goog"
asm/
set 3 field(s) of setter "anthos.servicemesh.created-by" to value "asmcli-1.15.7-asm.23.config1"
asm/
set 2 field(s) of setter "anthos.servicemesh.idp-url" to value "https://gkehub.googleapis.com/projects/qwiklabs-gcp-00-6b98c2173360/locations/global/memberships/abm-hybrid-cluster"
asm/
set 2 field(s) of setter "anthos.servicemesh.trustDomainAliases" to value "qwiklabs-gcp-00-6b98c2173360.svc.id.goog"
namespace/istio-system labeled
asmcli: Installing validation webhook fix...
service/istiod created
asmcli: Installing ASM control plane...

Thank you for installing Istio 1.15.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/SWHFBmwJspusK1hv6
asmcli: ...done!
asmcli: Installing ASM CanonicalService controller in asm-system namespace...
namespace/asm-system created
customresourcedefinition.apiextensions.k8s.io/canonicalservices.anthos.cloud.google.com created
role.rbac.authorization.k8s.io/canonical-service-leader-election-role created
clusterrole.rbac.authorization.k8s.io/canonical-service-manager-role created
clusterrole.rbac.authorization.k8s.io/canonical-service-metrics-reader created
serviceaccount/canonical-service-account created
rolebinding.rbac.authorization.k8s.io/canonical-service-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/canonical-service-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/canonical-service-proxy-rolebinding created
service/canonical-service-controller-manager-metrics-service created
deployment.apps/canonical-service-controller-manager created
asmcli: Waiting for deployment...
deployment.apps/canonical-service-controller-manager condition met
asmcli: ...done!
asmcli: 
asmcli: *****************************
client version: 1.15.7-asm.23
istiod version: 1.15.7
istiod version: 1.15.7-asm.23
data plane version: none
asmcli: *****************************
asmcli: The ASM control plane installation is now complete.
asmcli: To enable automatic sidecar injection on a namespace, you can use the following command:
asmcli: kubectl label namespace <NAMESPACE> istio-injection- istio.io/rev=asm-1157-23 --overwrite
asmcli: If you use 'istioctl install' afterwards to modify this installation, you will need
asmcli: to specify the option '--set revision=asm-1157-23' to target this control plane
asmcli: instead of installing a new one.
asmcli: To finish the installation, enable Istio sidecar injection and restart your workloads.
asmcli: For more information, see:
asmcli: https://cloud.google.com/service-mesh/docs/proxy-injection
asmcli: The ASM package used for installation can be found at:
asmcli: /root/baremetal/asm
asmcli: The version of istioctl that matches the installation can be found at:
asmcli: /root/baremetal/istio-1.15.7-asm.23/bin/istioctl
asmcli: A symlink to the istioctl binary can be found at:
asmcli: /root/baremetal/istioctl
asmcli: The combined configuration generated for installation can be found at:
asmcli: /root/baremetal/asm-1157-23-manifest-raw.yaml
asmcli: The full, expanded set of kubernetes resources can be found at:
asmcli: /root/baremetal/asm-1157-23-manifest-expanded.yaml
asmcli: *****************************
asmcli: Successfully installed ASM.
root@abm-ws:~/baremetal# 
```

##  Explore application tracing

1. Create and label a demo namespace to enable automatic sidecar injection:

```sh
root@abm-ws:~/baremetal# export ASM_REV=$(kubectl -n istio-system get pods -l app=istiod -o json | jq -r '.items[0].metadata.labels["istio.io/rev"]')
root@abm-ws:~/baremetal# kubectl create namespace demo
namespace/demo created
root@abm-ws:~/baremetal# kubectl label namespace demo istio.io/rev=$ASM_REV --overwrite
namespace/demo labeled
root@abm-ws:~/baremetal# 
```

Install the Hipster Shop application in the **demo** namespace:

```sh
root@abm-ws:~/baremetal# kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml -n demo
deployment.apps/currencyservice created
service/currencyservice created
serviceaccount/currencyservice created
deployment.apps/loadgenerator created
serviceaccount/loadgenerator created
deployment.apps/productcatalogservice created
service/productcatalogservice created
serviceaccount/productcatalogservice created
deployment.apps/checkoutservice created
service/checkoutservice created
serviceaccount/checkoutservice created
deployment.apps/shippingservice created
service/shippingservice created
serviceaccount/shippingservice created
deployment.apps/cartservice created
service/cartservice created
serviceaccount/cartservice created
deployment.apps/redis-cart created
service/redis-cart created
deployment.apps/emailservice created
service/emailservice created
serviceaccount/emailservice created
deployment.apps/paymentservice created
service/paymentservice created
serviceaccount/paymentservice created
deployment.apps/frontend created
service/frontend created
service/frontend-external created
serviceaccount/frontend created
deployment.apps/recommendationservice created
service/recommendationservice created
serviceaccount/recommendationservice created
deployment.apps/adservice created
service/adservice created
serviceaccount/adservice created
root@abm-ws:~/baremetal# kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml -n demo
serviceentry.networking.istio.io/allow-egress-googleapis created
serviceentry.networking.istio.io/allow-egress-google-metadata created
virtualservice.networking.istio.io/frontend created
resource mapping not found for name: "istio-gateway" namespace: "" from "https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml": no matches for kind "Gateway" in version "gateway.networking.k8s.io/v1beta1"
ensure CRDs are installed first
resource mapping not found for name: "frontend-route" namespace: "" from "https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml": no matches for kind "HTTPRoute" in version "gateway.networking.k8s.io/v1beta1"
ensure CRDs are installed first
root@abm-ws:~/baremetal# kubectl patch deployments/productcatalogservice -p '{"spec":{"template":{"metadata":{"labels":{"version":"v1"}}}}}' -n demo
deployment.apps/productcatalogservice patched
root@abm-ws:~/baremetal# 
```

To be able to access the application from outside the cluster, install the ingress Gateway:

```sh
root@abm-ws:~/baremetal# git clone https://github.com/GoogleCloudPlatform/anthos-service-mesh-packages
Cloning into 'anthos-service-mesh-packages'...
remote: Enumerating objects: 12080, done.
remote: Counting objects: 100% (1485/1485), done.
remote: Compressing objects: 100% (506/506), done.
remote: Total 12080 (delta 1109), reused 1328 (delta 976), pack-reused 10595
Receiving objects: 100% (12080/12080), 3.04 MiB | 20.89 MiB/s, done.
Resolving deltas:   2% (169/8295)
Resolving deltas: 100% (8295/8295), done.
root@abm-ws:~/baremetal# kubectl apply -f anthos-service-mesh-packages/samples/gateways/istio-ingressgateway -n demo
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
deployment.apps/istio-ingressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
role.rbac.authorization.k8s.io/istio-ingressgateway created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway created
service/istio-ingressgateway created
serviceaccount/istio-ingressgateway created
root@abm-ws:~/baremetal# 
```

Configure the Gateway:

```sh
root@abm-ws:~/baremetal# kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml -n demo
serviceentry.networking.istio.io/allow-egress-googleapis unchanged
serviceentry.networking.istio.io/allow-egress-google-metadata unchanged
virtualservice.networking.istio.io/frontend unchanged
resource mapping not found for name: "istio-gateway" namespace: "" from "https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml": no matches for kind "Gateway" in version "gateway.networking.k8s.io/v1beta1"
ensure CRDs are installed first
resource mapping not found for name: "frontend-route" namespace: "" from "https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/istio-manifests.yaml": no matches for kind "HTTPRoute" in version "gateway.networking.k8s.io/v1beta1"
ensure CRDs are installed first
root@abm-ws:~/baremetal# 
```

View the Hipster Shop pods that have been created in the demo namespace. Notice that they have a 2/2 in a ready state. That means that the two containers are ready, including the application container and the mesh sidecar container.

```sh
root@abm-ws:~/baremetal# kubectl get pods -n demo
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-7d94897dd6-zvnwk               2/2     Running   0          115s
cartservice-55dbff556f-845j4             2/2     Running   0          116s
checkoutservice-86cd7d575b-q2frb         2/2     Running   0          117s
currencyservice-65cb46f6db-xzq8q         2/2     Running   0          118s
emailservice-59f785bbb-ktb6g             2/2     Running   0          116s
frontend-6d4b46bd79-724cv                2/2     Running   0          116s
istio-ingressgateway-78d5d78c6-n8vmd     1/1     Running   0          70s
istio-ingressgateway-78d5d78c6-rbfg9     1/1     Running   0          70s
istio-ingressgateway-78d5d78c6-w87n2     1/1     Running   0          70s
loadgenerator-d9c69fc8f-7p9p2            2/2     Running   0          118s
paymentservice-dcd974b78-h7x4p           2/2     Running   0          116s
productcatalogservice-798c7c4b65-6wjcp   2/2     Running   0          109s
recommendationservice-6969c69d8-pf6l5    2/2     Running   0          115s
redis-cart-79b899577-dt2mq               2/2     Running   0          116s
shippingservice-6fc554769d-74zf8         2/2     Running   0          117s
root@abm-ws:~/baremetal# 
```

Get the external IP from the **istio-ingressgateway** to access the Hipster Shop that you just deployed:

```sh
root@abm-ws:~/baremetal# kubectl get svc istio-ingressgateway -n demo
NAME                   TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.96.7.84   10.200.0.103   15021:31695/TCP,80:31280/TCP,443:32275/TCP   98s
root@abm-ws:~/baremetal# 
```

Access the Hipster Shop using the IP you copied in the previous task:

```sh
root@abm-ws:~/baremetal# curl 10.200.0.103
curl: (7) Failed to connect to 10.200.0.103 port 80: Connection refused
root@abm-ws:~/baremetal# curl 10.200.0.103
curl: (7) Failed to connect to 10.200.0.103 port 80: Connection refused
root@abm-ws:~/baremetal# curl 10.200.0.103
curl: (7) Failed to connect to 10.200.0.103 port 80: Connection refused
root@abm-ws:~/baremetal# 
```

In the Console, go to **Navigation menu > Trace > Trace List** and select a trace from the graph that you would like to review. You see the breakdown by service so that you can investigate bottlenecks and networking issues.

## Detect and repair node problems

Many node problems can affect the pods running on the node, such as issues in the kernel, hardware, or container runtime. These problems are invisible to the upstream layers in the cluster management stack.

[Node Problem Detector](https://github.com/kubernetes/node-problem-detector) (NDP) detects common node problems, and reports node events and conditions. NPD runs as a systemd service on each node and can be enabled and disabled.

1. Come back to the Cloud Shell window you were using to SSH into the admin workstation. Check that the following conditions are being checked on the worker nodes by default:

```sh
root@abm-ws:~/baremetal# kubectl describe node abm-user-w2
Name:               abm-user-w2
Roles:              worker
Labels:             baremetal.cluster.gke.io/cgroup=v1
                    baremetal.cluster.gke.io/k8s-ip=10.200.0.5
                    baremetal.cluster.gke.io/namespace=cluster-abm-hybrid-cluster
                    baremetal.cluster.gke.io/node-pool=hybrid-cluster-pool-1
                    baremetal.cluster.gke.io/version=1.15.0
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=hybrid-cluster-pool-1
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=abm-user-w2
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=
Annotations:        baremetal.cluster.gke.io/provider: baremetal
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    networking.gke.io/ipv4-subnet: 10.200.0.5/24
                    networking.gke.io/ipv6-subnet: 
                    networking.gke.io/network-status: [{"name":"pod-network"}]
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sun, 28 Jul 2024 07:10:15 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  abm-user-w2
  AcquireTime:     <unset>
  RenewTime:       Sun, 28 Jul 2024 16:14:29 +0000
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  ContainerRuntimeUnhealthy     False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   ContainerRuntimeIsHealthy       Container runtime on the node is functioning properly
  KernelDeadlock                False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KernelHasNoDeadlock             kernel has no deadlock
  ReadonlyFilesystem            False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   FilesystemIsNotReadOnly         Filesystem is not read-only
  FrequentUnregisterNetDevice   False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentUnregisterNetDevice   node is functioning properly
  KubeletUnhealthy              False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KubeletIsHealthy                kubelet on the node is functioning properly
  FrequentKubeletRestart        False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentKubeletRestart        kubelet is functioning properly
  FrequentDockerRestart         False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentDockerRestart         docker is functioning properly
  FrequentContainerdRestart     False   Sun, 28 Jul 2024 16:11:53 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentContainerdRestart     containerd is functioning properly
  MemoryPressure                False   Sun, 28 Jul 2024 16:11:12 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientMemory      kubelet has sufficient memory available
  DiskPressure                  False   Sun, 28 Jul 2024 16:11:12 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Sun, 28 Jul 2024 16:11:12 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         True    Sun, 28 Jul 2024 16:11:12 +0000   Sun, 28 Jul 2024 07:10:49 +0000   KubeletReady                    kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.200.0.5
  Hostname:    abm-user-w2
Capacity:
  cpu:                4
  ephemeral-storage:  259966896Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16369876Ki
  pods:               250
Allocatable:
  cpu:                3440m
  ephemeral-storage:  239585490957
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             13295828Ki
  pods:               250
System Info:
  Machine ID:                 0eb23e41c594f0fed31719f54d8c6735
  System UUID:                0eb23e41-c594-f0fe-d317-19f54d8c6735
  Boot ID:                    ee19bf1b-568f-4e07-be1e-a1159d3919dd
  Kernel Version:             5.15.0-1062-gcp
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.18-gke.0
  Kubelet Version:            v1.26.2-gke.1001
  Kube-Proxy Version:         v1.26.2-gke.1001
PodCIDR:                      192.168.2.0/23
PodCIDRs:                     192.168.2.0/23
ProviderID:                   baremetal://10.200.0.5
Non-terminated Pods:          (16 in total)
  Namespace                   Name                                                     CPU Requests  CPU Limits   Memory Requests  Memory Limits  Age
  ---------                   ----                                                     ------------  ----------   ---------------  -------------  ---
  asm-system                  canonical-service-controller-manager-8576bf9678-cfzkh    200m (5%)     200m (5%)    40Mi (0%)        600Mi (4%)     6m38s
  demo                        emailservice-59f785bbb-ktb6g                             200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    4m56s
  demo                        istio-ingressgateway-78d5d78c6-w87n2                     100m (2%)     2 (58%)      128Mi (0%)       1Gi (7%)       4m10s
  demo                        loadgenerator-d9c69fc8f-7p9p2                            400m (11%)    2500m (72%)  384Mi (2%)       1536Mi (11%)   4m58s
  demo                        paymentservice-dcd974b78-h7x4p                           200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    4m56s
  demo                        productcatalogservice-798c7c4b65-6wjcp                   200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    4m49s
  demo                        recommendationservice-6969c69d8-pf6l5                    200m (5%)     2200m (63%)  348Mi (2%)       1474Mi (11%)   4m55s
  demo                        shippingservice-6fc554769d-74zf8                         200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    4m57s
  gke-connect                 gke-connect-agent-20230224-00-00-654bf8d884-lqdmc        0 (0%)        0 (0%)       256Mi (1%)       256Mi (1%)     8h
  istio-system                istiod-asm-1157-23-96886d6f4-5qdmk                       500m (14%)    0 (0%)       2Gi (15%)        0 (0%)         6m53s
  kube-system                 anetd-c2rm2                                              110m (3%)     500m (14%)   164Mi (1%)       128Mi (0%)     9h
  kube-system                 gke-metrics-agent-8kxjj                                  50m (1%)      200m (5%)    200Mi (1%)       4608Mi (35%)   9h
  kube-system                 kube-proxy-ln2d6                                         100m (2%)     0 (0%)       15Mi (0%)        0 (0%)         9h
  kube-system                 localpv-ndrqv                                            0 (0%)        0 (0%)       0 (0%)           0 (0%)         9h
  kube-system                 node-exporter-mhcjx                                      100m (2%)     200m (5%)    50Mi (0%)        500Mi (3%)     9h
  kube-system                 stackdriver-log-forwarder-f87fs                          200m (5%)     600m (17%)   100Mi (0%)       600Mi (4%)     9h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2760m (80%)   17200m (500%)
  memory             4501Mi (34%)  15334Mi (118%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
Events:              <none>
root@abm-ws:~/baremetal# 
```

1. In the Conditions section of the output, verify that that the following conditions are present:

**Checks provided by the Node Problem Detector:**

- FrequentContainerdRestart
- NoFrequentContainerdRestart
- KernelHasNoDeadlock
- ReadonlyFilesystem
- FilesystemIsNotReadOnly
- FrequentUnregisterNetDevice
- NoFrequentUnregisterNetDevice
- ContainerRuntimeIsHealthy
- KubeletIsHealthy
- FrequentKubeletRestart
- FrequentDockerRestart

**Checks provided by Kubernetes:**

- MemoryPressure
- DiskPressure
- PIDPressure
- Ready

1. Also, verify that there are not any Events indicating there might be a problem.

Node Problem Detector is enabled by default in your cluster starting in Anthos clusters on bare metal v1.10.

1. Edit the configmap to enable or disable it:

```sh
root@abm-ws:~/baremetal# kubectl edit configmap node-problem-detector-config -n cluster-abm-hybrid-cluster
configmap/node-problem-detector-config edited
root@abm-ws:~/baremetal# 
```



Now, when you get the state of the node, nothing will have changed, but you could disable it by changing the value to false.

1. Run **kubectl describe** again to view that no changes were made:

```sh
root@abm-ws:~/baremetal# kubectl describe node abm-user-w2
Name:               abm-user-w2
Roles:              worker
Labels:             baremetal.cluster.gke.io/cgroup=v1
                    baremetal.cluster.gke.io/k8s-ip=10.200.0.5
                    baremetal.cluster.gke.io/namespace=cluster-abm-hybrid-cluster
                    baremetal.cluster.gke.io/node-pool=hybrid-cluster-pool-1
                    baremetal.cluster.gke.io/version=1.15.0
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=hybrid-cluster-pool-1
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=abm-user-w2
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=
Annotations:        baremetal.cluster.gke.io/provider: baremetal
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    networking.gke.io/ipv4-subnet: 10.200.0.5/24
                    networking.gke.io/ipv6-subnet: 
                    networking.gke.io/network-status: [{"name":"pod-network"}]
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sun, 28 Jul 2024 07:10:15 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  abm-user-w2
  AcquireTime:     <unset>
  RenewTime:       Sun, 28 Jul 2024 16:17:43 +0000
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  FrequentKubeletRestart        False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentKubeletRestart        kubelet is functioning properly
  FrequentDockerRestart         False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentDockerRestart         docker is functioning properly
  FrequentContainerdRestart     False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentContainerdRestart     containerd is functioning properly
  ContainerRuntimeUnhealthy     False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   ContainerRuntimeIsHealthy       Container runtime on the node is functioning properly
  KernelDeadlock                False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KernelHasNoDeadlock             kernel has no deadlock
  ReadonlyFilesystem            False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   FilesystemIsNotReadOnly         Filesystem is not read-only
  FrequentUnregisterNetDevice   False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentUnregisterNetDevice   node is functioning properly
  KubeletUnhealthy              False   Sun, 28 Jul 2024 16:16:54 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KubeletIsHealthy                kubelet on the node is functioning properly
  MemoryPressure                False   Sun, 28 Jul 2024 16:16:18 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientMemory      kubelet has sufficient memory available
  DiskPressure                  False   Sun, 28 Jul 2024 16:16:18 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Sun, 28 Jul 2024 16:16:18 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         True    Sun, 28 Jul 2024 16:16:18 +0000   Sun, 28 Jul 2024 07:10:49 +0000   KubeletReady                    kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.200.0.5
  Hostname:    abm-user-w2
Capacity:
  cpu:                4
  ephemeral-storage:  259966896Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16369876Ki
  pods:               250
Allocatable:
  cpu:                3440m
  ephemeral-storage:  239585490957
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             13295828Ki
  pods:               250
System Info:
  Machine ID:                 0eb23e41c594f0fed31719f54d8c6735
  System UUID:                0eb23e41-c594-f0fe-d317-19f54d8c6735
  Boot ID:                    ee19bf1b-568f-4e07-be1e-a1159d3919dd
  Kernel Version:             5.15.0-1062-gcp
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.18-gke.0
  Kubelet Version:            v1.26.2-gke.1001
  Kube-Proxy Version:         v1.26.2-gke.1001
PodCIDR:                      192.168.2.0/23
PodCIDRs:                     192.168.2.0/23
ProviderID:                   baremetal://10.200.0.5
Non-terminated Pods:          (16 in total)
  Namespace                   Name                                                     CPU Requests  CPU Limits   Memory Requests  Memory Limits  Age
  ---------                   ----                                                     ------------  ----------   ---------------  -------------  ---
  asm-system                  canonical-service-controller-manager-8576bf9678-cfzkh    200m (5%)     200m (5%)    40Mi (0%)        600Mi (4%)     9m51s
  demo                        emailservice-59f785bbb-ktb6g                             200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    8m9s
  demo                        istio-ingressgateway-78d5d78c6-w87n2                     100m (2%)     2 (58%)      128Mi (0%)       1Gi (7%)       7m23s
  demo                        loadgenerator-d9c69fc8f-7p9p2                            400m (11%)    2500m (72%)  384Mi (2%)       1536Mi (11%)   8m11s
  demo                        paymentservice-dcd974b78-h7x4p                           200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    8m9s
  demo                        productcatalogservice-798c7c4b65-6wjcp                   200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    8m2s
  demo                        recommendationservice-6969c69d8-pf6l5                    200m (5%)     2200m (63%)  348Mi (2%)       1474Mi (11%)   8m8s
  demo                        shippingservice-6fc554769d-74zf8                         200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    8m10s
  gke-connect                 gke-connect-agent-20230224-00-00-654bf8d884-lqdmc        0 (0%)        0 (0%)       256Mi (1%)       256Mi (1%)     9h
  istio-system                istiod-asm-1157-23-96886d6f4-5qdmk                       500m (14%)    0 (0%)       2Gi (15%)        0 (0%)         10m
  kube-system                 anetd-c2rm2                                              110m (3%)     500m (14%)   164Mi (1%)       128Mi (0%)     9h
  kube-system                 gke-metrics-agent-8kxjj                                  50m (1%)      200m (5%)    200Mi (1%)       4608Mi (35%)   9h
  kube-system                 kube-proxy-ln2d6                                         100m (2%)     0 (0%)       15Mi (0%)        0 (0%)         9h
  kube-system                 localpv-ndrqv                                            0 (0%)        0 (0%)       0 (0%)           0 (0%)         9h
  kube-system                 node-exporter-mhcjx                                      100m (2%)     200m (5%)    50Mi (0%)        500Mi (3%)     9h
  kube-system                 stackdriver-log-forwarder-f87fs                          200m (5%)     600m (17%)   100Mi (0%)       600Mi (4%)     9h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2760m (80%)   17200m (500%)
  memory             4501Mi (34%)  15334Mi (118%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
Events:              <none>
root@abm-ws:~/baremetal# 
```

1. Verify that the same conditions are being displayed:

- FrequentContainerdRestart
- NoFrequentContainerdRestart
- KernelHasNoDeadlock
- ReadonlyFilesystem
- FilesystemIsNotReadOnly
- FrequentUnregisterNetDevice
- NoFrequentUnregisterNetDevice
- ContainerRuntimeIsHealthy
- KubeletIsHealthy
- FrequentKubeletRestart
- FrequentDockerRestart
- MemoryPressure
- DiskPressure
- PIDPressure
- Ready

1. Also, verify that there are not any Events indicating there might be a problem.

### Introduction of a node problem

Let's introduce a problem in one of the nodes to see if NPD reports it correctly.

1. Open a new tab in the Google Cloud Console.
2. SSH into the **abm-user-w2** worker node:
3. Stop the container runtime to simulate a problem in the node:
4. Go back to the first Cloud Shell window and check the conditions of the node where we stopped containerd:

```sh
root@abm-ws:~/baremetal# kubectl describe node abm-user-w2
Name:               abm-user-w2
Roles:              worker
Labels:             baremetal.cluster.gke.io/cgroup=v1
                    baremetal.cluster.gke.io/k8s-ip=10.200.0.5
                    baremetal.cluster.gke.io/namespace=cluster-abm-hybrid-cluster
                    baremetal.cluster.gke.io/node-pool=hybrid-cluster-pool-1
                    baremetal.cluster.gke.io/version=1.15.0
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=hybrid-cluster-pool-1
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=abm-user-w2
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=
Annotations:        baremetal.cluster.gke.io/provider: baremetal
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    networking.gke.io/ipv4-subnet: 10.200.0.5/24
                    networking.gke.io/ipv6-subnet: 
                    networking.gke.io/network-status: [{"name":"pod-network"}]
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sun, 28 Jul 2024 07:10:15 +0000
Taints:             node.kubernetes.io/not-ready:NoExecute
                    node.kubernetes.io/not-ready:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  abm-user-w2
  AcquireTime:     <unset>
  RenewTime:       Sun, 28 Jul 2024 16:19:56 +0000
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  ContainerRuntimeUnhealthy     True    Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 16:19:25 +0000   ContainerdUnhealthy             cri:containerd was found unhealthy; repair flag : true
  KernelDeadlock                False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KernelHasNoDeadlock             kernel has no deadlock
  ReadonlyFilesystem            False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   FilesystemIsNotReadOnly         Filesystem is not read-only
  FrequentUnregisterNetDevice   False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentUnregisterNetDevice   node is functioning properly
  KubeletUnhealthy              False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KubeletIsHealthy                kubelet on the node is functioning properly
  FrequentKubeletRestart        False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentKubeletRestart        kubelet is functioning properly
  FrequentDockerRestart         False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentDockerRestart         docker is functioning properly
  FrequentContainerdRestart     False   Sun, 28 Jul 2024 16:19:26 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentContainerdRestart     containerd is functioning properly
  MemoryPressure                False   Sun, 28 Jul 2024 16:19:43 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientMemory      kubelet has sufficient memory available
  DiskPressure                  False   Sun, 28 Jul 2024 16:19:43 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Sun, 28 Jul 2024 16:19:43 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         False   Sun, 28 Jul 2024 16:19:43 +0000   Sun, 28 Jul 2024 16:19:43 +0000   KubeletNotReady                 container runtime is down
Addresses:
  InternalIP:  10.200.0.5
  Hostname:    abm-user-w2
Capacity:
  cpu:                4
  ephemeral-storage:  259966896Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16369876Ki
  pods:               250
Allocatable:
  cpu:                3440m
  ephemeral-storage:  239585490957
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             13295828Ki
  pods:               250
System Info:
  Machine ID:                 0eb23e41c594f0fed31719f54d8c6735
  System UUID:                0eb23e41-c594-f0fe-d317-19f54d8c6735
  Boot ID:                    ee19bf1b-568f-4e07-be1e-a1159d3919dd
  Kernel Version:             5.15.0-1062-gcp
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://Unknown
  Kubelet Version:            v1.26.2-gke.1001
  Kube-Proxy Version:         v1.26.2-gke.1001
PodCIDR:                      192.168.2.0/23
PodCIDRs:                     192.168.2.0/23
ProviderID:                   baremetal://10.200.0.5
Non-terminated Pods:          (16 in total)
  Namespace                   Name                                                     CPU Requests  CPU Limits   Memory Requests  Memory Limits  Age
  ---------                   ----                                                     ------------  ----------   ---------------  -------------  ---
  asm-system                  canonical-service-controller-manager-8576bf9678-cfzkh    200m (5%)     200m (5%)    40Mi (0%)        600Mi (4%)     12m
  demo                        emailservice-59f785bbb-ktb6g                             200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    10m
  demo                        istio-ingressgateway-78d5d78c6-w87n2                     100m (2%)     2 (58%)      128Mi (0%)       1Gi (7%)       9m36s
  demo                        loadgenerator-d9c69fc8f-7p9p2                            400m (11%)    2500m (72%)  384Mi (2%)       1536Mi (11%)   10m
  demo                        paymentservice-dcd974b78-h7x4p                           200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    10m
  demo                        productcatalogservice-798c7c4b65-6wjcp                   200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    10m
  demo                        recommendationservice-6969c69d8-pf6l5                    200m (5%)     2200m (63%)  348Mi (2%)       1474Mi (11%)   10m
  demo                        shippingservice-6fc554769d-74zf8                         200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    10m
  gke-connect                 gke-connect-agent-20230224-00-00-654bf8d884-lqdmc        0 (0%)        0 (0%)       256Mi (1%)       256Mi (1%)     9h
  istio-system                istiod-asm-1157-23-96886d6f4-5qdmk                       500m (14%)    0 (0%)       2Gi (15%)        0 (0%)         12m
  kube-system                 anetd-c2rm2                                              110m (3%)     500m (14%)   164Mi (1%)       128Mi (0%)     9h
  kube-system                 gke-metrics-agent-8kxjj                                  50m (1%)      200m (5%)    200Mi (1%)       4608Mi (35%)   9h
  kube-system                 kube-proxy-ln2d6                                         100m (2%)     0 (0%)       15Mi (0%)        0 (0%)         9h
  kube-system                 localpv-ndrqv                                            0 (0%)        0 (0%)       0 (0%)           0 (0%)         9h
  kube-system                 node-exporter-mhcjx                                      100m (2%)     200m (5%)    50Mi (0%)        500Mi (3%)     9h
  kube-system                 stackdriver-log-forwarder-f87fs                          200m (5%)     600m (17%)   100Mi (0%)       600Mi (4%)     9h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2760m (80%)   17200m (500%)
  memory             4501Mi (34%)  15334Mi (118%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
Events:
  Type     Reason               Age   From            Message
  ----     ------               ----  ----            -------
  Warning  ContainerdUnhealthy  40s   health-checker  Node condition ContainerRuntimeUnhealthy is now: True, reason: ContainerdUnhealthy, message: "cri:containerd was found unhealthy; repair flag : true"
  Normal   NodeNotReady         22s   kubelet         Node abm-user-w2 status is now: NodeNotReady
  Warning  ContainerGCFailed    1s    kubelet         rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /run/containerd/containerd.sock: connect: no such file or directory"
root@abm-ws:~/baremetal# 
```

Notice that the status of the **Condition** "ContainerRuntimeUnhealthy" is "true". Also, notice that the following events appar now:

- ContainerdUnhealthy
- NodeNotReady
- ContainerGCFailed

All errors might take a couple seconds to appear.

1. In the second Cloud Shell window, you can get more information by running **journalctl**. Input letter **q** to exit:

```sh
root@abm-user-w2:~# journalctl -u node-problem-detector
-- Logs begin at Sun 2024-07-28 06:51:55 UTC, end at Sun 2024-07-28 16:21:00 UTC. --
Jul 28 07:15:55 abm-user-w2 systemd[1]: Started Kubernetes node problem detector.
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.590102   13978 custom_plugin_monitor.go:80] Finish parsing custom plugin monitor config file /opt/node-pro>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.591259   13978 custom_plugin_monitor.go:80] Finish parsing custom plugin monitor config file /opt/node-pro>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.591615   13978 custom_plugin_monitor.go:80] Finish parsing custom plugin monitor config file /opt/node-pro>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.591937   13978 custom_plugin_monitor.go:80] Finish parsing custom plugin monitor config file /opt/node-pro>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.592647   13978 log_monitor.go:79] Finish parsing log monitor config file /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.592834   13978 log_watchers.go:40] Use log watcher of plugin "kmsg"
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.593183   13978 log_monitor.go:79] Finish parsing log monitor config file /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.593356   13978 log_watchers.go:40] Use log watcher of plugin "journald"
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.596971   13978 k8s_exporter.go:54] Waiting for kube-apiserver to be ready (timeout 5m0s)...
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686318   13978 node_problem_detector.go:63] K8s exporter started.
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686361   13978 custom_plugin_monitor.go:111] Start custom plugin monitor /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686379   13978 custom_plugin_monitor.go:111] Start custom plugin monitor /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686388   13978 custom_plugin_monitor.go:111] Start custom plugin monitor /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686406   13978 custom_plugin_monitor.go:111] Start custom plugin monitor /opt/node-problem-detector/config>
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686416   13978 log_monitor.go:111] Start log monitor /opt/node-problem-detector/config/kernel-monitor.json
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686490   13978 log_monitor.go:111] Start log monitor /opt/node-problem-detector/config/systemd-monitor.json
Jul 28 07:15:55 abm-user-w2 node-problem-detector[13978]: I0728 07:15:55.686691   13978 custom_plugin_monitor.go:300] Initialize condition generated: [{Type:ContainerRuntimeUnheal>
lines 1-19
```

Alternatively, you can also check the logs in Cloud Logging. To do so, in the Console, go to **Navigation menu > Logging > Logs Explorer** and enter the following query. Then, replace the **PROJECT_ID** in the query with  and click **Run query**:

```sh
resource.type="k8s_node"
resource.labels.node_name="abm-user-w2"
log_name="projects/PROJECT_ID/logs/node-problem-detector"
```

### Repairing the node problem

1. To repair the node problem, go to the second Cloud Shell window, and start the container runtime again:

```sh
root@abm-user-w2:~# sudo systemctl start containerd
root@abm-user-w2:~# 
```

Check the status of the node until the **Condition** "ContainerRuntimeUnhealthy" is false:

```sh
root@abm-ws:~/baremetal# kubectl describe node abm-user-w2
Name:               abm-user-w2
Roles:              worker
Labels:             baremetal.cluster.gke.io/cgroup=v1
                    baremetal.cluster.gke.io/k8s-ip=10.200.0.5
                    baremetal.cluster.gke.io/namespace=cluster-abm-hybrid-cluster
                    baremetal.cluster.gke.io/node-pool=hybrid-cluster-pool-1
                    baremetal.cluster.gke.io/version=1.15.0
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    cloud.google.com/gke-nodepool=hybrid-cluster-pool-1
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=abm-user-w2
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=
Annotations:        baremetal.cluster.gke.io/provider: baremetal
                    kubeadm.alpha.kubernetes.io/cri-socket: unix:///run/containerd/containerd.sock
                    networking.gke.io/ipv4-subnet: 10.200.0.5/24
                    networking.gke.io/ipv6-subnet: 
                    networking.gke.io/network-status: [{"name":"pod-network"}]
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Sun, 28 Jul 2024 07:10:15 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  abm-user-w2
  AcquireTime:     <unset>
  RenewTime:       Sun, 28 Jul 2024 16:22:18 +0000
Conditions:
  Type                          Status  LastHeartbeatTime                 LastTransitionTime                Reason                          Message
  ----                          ------  -----------------                 ------------------                ------                          -------
  FrequentKubeletRestart        False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentKubeletRestart        kubelet is functioning properly
  FrequentDockerRestart         False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentDockerRestart         docker is functioning properly
  FrequentContainerdRestart     False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentContainerdRestart     containerd is functioning properly
  ContainerRuntimeUnhealthy     False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 16:22:05 +0000   ContainerRuntimeIsHealthy       Container runtime on the node is functioning properly
  KernelDeadlock                False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KernelHasNoDeadlock             kernel has no deadlock
  ReadonlyFilesystem            False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   FilesystemIsNotReadOnly         Filesystem is not read-only
  FrequentUnregisterNetDevice   False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   NoFrequentUnregisterNetDevice   node is functioning properly
  KubeletUnhealthy              False   Sun, 28 Jul 2024 16:22:06 +0000   Sun, 28 Jul 2024 07:15:55 +0000   KubeletIsHealthy                kubelet on the node is functioning properly
  MemoryPressure                False   Sun, 28 Jul 2024 16:22:16 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientMemory      kubelet has sufficient memory available
  DiskPressure                  False   Sun, 28 Jul 2024 16:22:16 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasNoDiskPressure        kubelet has no disk pressure
  PIDPressure                   False   Sun, 28 Jul 2024 16:22:16 +0000   Sun, 28 Jul 2024 07:10:15 +0000   KubeletHasSufficientPID         kubelet has sufficient PID available
  Ready                         True    Sun, 28 Jul 2024 16:22:16 +0000   Sun, 28 Jul 2024 16:22:16 +0000   KubeletReady                    kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.200.0.5
  Hostname:    abm-user-w2
Capacity:
  cpu:                4
  ephemeral-storage:  259966896Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             16369876Ki
  pods:               250
Allocatable:
  cpu:                3440m
  ephemeral-storage:  239585490957
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             13295828Ki
  pods:               250
System Info:
  Machine ID:                 0eb23e41c594f0fed31719f54d8c6735
  System UUID:                0eb23e41-c594-f0fe-d317-19f54d8c6735
  Boot ID:                    ee19bf1b-568f-4e07-be1e-a1159d3919dd
  Kernel Version:             5.15.0-1062-gcp
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.18-gke.0
  Kubelet Version:            v1.26.2-gke.1001
  Kube-Proxy Version:         v1.26.2-gke.1001
PodCIDR:                      192.168.2.0/23
PodCIDRs:                     192.168.2.0/23
ProviderID:                   baremetal://10.200.0.5
Non-terminated Pods:          (16 in total)
  Namespace                   Name                                                     CPU Requests  CPU Limits   Memory Requests  Memory Limits  Age
  ---------                   ----                                                     ------------  ----------   ---------------  -------------  ---
  asm-system                  canonical-service-controller-manager-8576bf9678-cfzkh    200m (5%)     200m (5%)    40Mi (0%)        600Mi (4%)     14m
  demo                        emailservice-59f785bbb-ktb6g                             200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    12m
  demo                        istio-ingressgateway-78d5d78c6-w87n2                     100m (2%)     2 (58%)      128Mi (0%)       1Gi (7%)       11m
  demo                        loadgenerator-d9c69fc8f-7p9p2                            400m (11%)    2500m (72%)  384Mi (2%)       1536Mi (11%)   12m
  demo                        paymentservice-dcd974b78-h7x4p                           200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    12m
  demo                        productcatalogservice-798c7c4b65-6wjcp                   200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    12m
  demo                        recommendationservice-6969c69d8-pf6l5                    200m (5%)     2200m (63%)  348Mi (2%)       1474Mi (11%)   12m
  demo                        shippingservice-6fc554769d-74zf8                         200m (5%)     2200m (63%)  192Mi (1%)       1152Mi (8%)    12m
  gke-connect                 gke-connect-agent-20230224-00-00-654bf8d884-lqdmc        0 (0%)        0 (0%)       256Mi (1%)       256Mi (1%)     9h
  istio-system                istiod-asm-1157-23-96886d6f4-5qdmk                       500m (14%)    0 (0%)       2Gi (15%)        0 (0%)         14m
  kube-system                 anetd-c2rm2                                              110m (3%)     500m (14%)   164Mi (1%)       128Mi (0%)     9h
  kube-system                 gke-metrics-agent-8kxjj                                  50m (1%)      200m (5%)    200Mi (1%)       4608Mi (35%)   9h
  kube-system                 kube-proxy-ln2d6                                         100m (2%)     0 (0%)       15Mi (0%)        0 (0%)         9h
  kube-system                 localpv-ndrqv                                            0 (0%)        0 (0%)       0 (0%)           0 (0%)         9h
  kube-system                 node-exporter-mhcjx                                      100m (2%)     200m (5%)    50Mi (0%)        500Mi (3%)     9h
  kube-system                 stackdriver-log-forwarder-f87fs                          200m (5%)     600m (17%)   100Mi (0%)       600Mi (4%)     9h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests      Limits
  --------           --------      ------
  cpu                2760m (80%)   17200m (500%)
  memory             4501Mi (34%)  15334Mi (118%)
  ephemeral-storage  0 (0%)        0 (0%)
  hugepages-1Gi      0 (0%)        0 (0%)
  hugepages-2Mi      0 (0%)        0 (0%)
Events:
  Type     Reason                     Age                  From             Message
  ----     ------                     ----                 ----             -------
  Warning  ContainerdUnhealthy        3m3s                 health-checker   Node condition ContainerRuntimeUnhealthy is now: True, reason: ContainerdUnhealthy, message: "cri:containerd was found unhealthy; repair flag : true"
  Normal   NodeNotReady               2m45s                kubelet          Node abm-user-w2 status is now: NodeNotReady
  Warning  ContainerdStart            31s                  systemd-monitor  Starting containerd container runtime...
  Warning  ContainerGCFailed          24s (x3 over 2m24s)  kubelet          rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /run/containerd/containerd.sock: connect: no such file or directory"
  Normal   ContainerRuntimeIsHealthy  23s                  health-checker   Node condition ContainerRuntimeUnhealthy is now: False, reason: ContainerRuntimeIsHealthy, message: "Container runtime on the node is functioning properly"
  Normal   NodeReady                  12s (x2 over 9h)     kubelet          Node abm-user-w2 status is now: NodeReady
root@abm-ws:~/baremetal# 
```

## Troubleshooting

1. Set the Zone environment variable:

   ```sh
   ZONE=us-central1-b
   ```

```sh
gcloud compute ssh --ssh-flag="-A" root@abm-ws \
    --zone $ZONE \
    --tunnel-through-iap
```

```sh
# From the hybrid workstation (root@abm-ws)
export KUBECONFIG=$KUBECONFIG:~/baremetal/bmctl-workspace/abm-hybrid-cluster/abm-hybrid-cluster-kubeconfig
kubectl get nodes
```

In this lab, you explored logs and metrics from Anthos clusters on bare metal. You learned how to access cluster, application, and audit logs, as well as cluster and application metrics.
