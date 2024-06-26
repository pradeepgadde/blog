---

layout: single
title:  "Securing Google Kubernetes Engine with Cloud IAM and Pod Security Admission "
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/k8s-banner.png
  og_image: /assets/images/k8s-banner.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Securing Google Kubernetes Engine with Cloud IAM and Pod Security Admission

- Use Cloud IAM to control GKE access
- Create and use Pod security policies to control Pod creation
- Perform IP address and credential rotation

## Use Cloud IAM roles to grant administrative access to all the GKE clusters in the project

### Sign in to the Google Cloud Console as the first user

### Sign in to the Google Cloud Console as the first user

1. Sign in to the Google Cloud Console in an Incognito window as usual with the **Username 1** provided. Note that both user names use the same password.
2. On the Google Cloud Console title bar, click **Activate Cloud Shell** (![Cloud Shell icon](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).
3. Click **Continue**.

After a moment of provisioning, the Cloud Shell prompt appears.

### Sign in and explore the Google Cloud Console as the second user

1. Open another tab in your incognito window.
2. Browse to  [console.cloud.google.com](http://console.cloud.google.com/).
3. Click on the user icon in the top-right corner of the screen, and then click **Add account**.
4. Sign in to the Google Cloud Console with the **Username 2** provided. Again, note that both user names use the same password.
5. While logged in as **Username 2**, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**.
6. Make sure that your lab Project ID is selected at the top of the page.

Notice that the option to create a cluster is disabled.

### Grant the GKE Admin Cloud IAM role to Username 2

You will now allow **Username 2** to create a GKE  cluster and deploy workloads by using primitive roles to grant a user  permissions to administer all GKE clusters and manage resources inside  those clusters in this project. The **Username 1** account has project owner rights and you will use that account to grant **Username 2** more rights.

1. Switch back to the **Username 1** Google Cloud Console tab.
2. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **IAM & admin > IAM**.
3. In the **IAM console**, locate the row that corresponds to **Username 2**, and then click on the pencil icon at the right-end of that row to edit that user's permissions.
4. Notice that **Username 2** currently has the **Viewer** role, which provides read access to all resources within the project.
5. Click **ADD ANOTHER ROLE** to add another dropdown selection for roles.
6. In the **Select a role** dropdown box, choose **Kubernetes Engine > Kubernetes Engine Cluster Admin**.
7. Click **SAVE**.

### Test the access of Username 2

You will now verify your work by using **Username 2** to create a GKE cluster.

1. Switch back to the **Username 2** Google Cloud Console tab.
2. While logged in as **Username 2**, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**.

You should now see that the option to create a cluster is now enabled. You may need to refresh the web browser tab for **Username 2** to see the changes.

1. Click **Create** to begin creating a GKE cluster.
2. Click **Switch to Standard Cluster** and confirm the same on next pop-up.
3. Set the name of the cluster to **standard-cluster-1**, if that is not the default.
4. Confirm that a zonal, rather than regional, cluster is selected.
5. Choose zone  for the cluster, if that is not the default.
6. Leave all other values at their defaults and click **Create**.

The cluster begins provisioning, but soon fails.

1. Click the notification icon in the toolbar at the top of the screen to view the error message.

Username 2 still lacks some of the rights necessary to deploy a  cluster. This is because GKE leverages Google Cloud Compute Engine  instances for the nodes.

To deploy a GKE cluster, a user must also be assigned the  iam.serviceAccountUser role on the Compute Engine default service  account.

```sh
The user does not have access to service account "743956172483-compute@developer.gserviceaccount.com". Ask a project owner to grant you the iam.serviceAccountUser role on the service account. 
```



### Grant the ServiceAccountUser IAM role to Username 2

You will now use IAM to grant **Username 2** the iam.serviceAccountUser role so that **Username 2** may successfully deploy a GKE cluster.

1. Switch back to the **Username 1** Google Cloud Console tab.
2. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **IAM & admin > Service accounts**.
3. In the **IAM console**, click the row that corresponds to the **Compute Engine default service account** to select it.
4. Click on **Permission** to open the permissions information panel.
5. On the Permission page, click on **Grant access**.

The permissions information panel will open on the right side of the window.

1. Type the username for **Username 2** into the **New principals** box. You can copy this name from the Lab Details page.

2. In the **Select a role** box, make sure **Service Accounts > Service Account User** is selected.

   Click **Save**.

### Verify that Username 2 can create a GKE cluster

You will now verify your work by using **Username 2** to create a GKE cluster.

1. Switch back to the **Username 2** Google Cloud Console tab.
2. While logged in as **Username 2**, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **Kubernetes Engine > Clusters**. You may need to refresh your web browser.
3. Click **Create** to begin creating a GKE cluster.
4. Select **Configure** option for **Standard: You manage your cluster**.
5. Set the name of the cluster to **standard-cluster-1**, if that is not the default.
6. Confirm that a zonal, rather than regional, cluster is selected.
7. Choose zone  for the cluster, if that is not the default.
8. Leave all other values at their defaults and click **Create**.

**Note:** You need to wait a few minutes for the cluster deployment to complete.

The cluster will successfully deploy this time.

## Define and use pod security admission

PodSecurity is a Kubernetes admission controller that lets you apply  Pod Security Standards to Pods running on your GKE clusters. Pod  Security Standards are predefined security policies that cover the  high-level needs of Pod security in Kubernetes. These policies range  from being highly permissive to highly restrictive.

In this task, you create a pod security policy that allows the  creation of unprivileged Pods in the default namespace of the cluster.  Unprivileged Pods do not allow users to execute code as root, and have  limited access to devices on the host.

You create a ClusterRole that can then be used in a role binding that ties the policy to accounts that require the ability to deploy pods  with unprivileged access.

Users that require the ability to deploy privileged Pods can be  granted access to the built in PSP that is provided to allow admin users to deploy pods after Pod Security Policies are enabled.

When you have the components configured you will enable the  PodSecurityPolicy controller, which enforces these policies, and then  test how they impact users with different privileges.

### Connect to the GKE cluster

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_01_4f5493923909@cloudshell:~$ export my_zone=us-west1-c
export my_cluster=standard-cluster-1
student_01_4f5493923909@cloudshell:~$ source <(kubectl completion bash)
student_01_4f5493923909@cloudshell:~$ gcloud config set project qwiklabs-gcp-01-89f4cbcd0816 
Updated property [core/project].
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```



### Apply Pod Security Standards using PodSecurity

To use the PodSecurity admission controller, you must apply specific  Pod Security Standards in specific modes to specific namespaces

### Create new namespaces

Create namespaces in your cluster:

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl create ns baseline-ns
 kubectl create ns restricted-ns
namespace/baseline-ns created
namespace/restricted-ns created
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```



This command creates the following namespaces:

- baseline-ns: For permissive workloads
- restricted-ns: For highly restricted workloads

### Use labels to apply security policies

Apply the following Pod Security Standards:

- baseline: Apply to baseline-ns in the warn mode
- restricted: Apply to restricted-ns in the enforce mode

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl label --overwrite ns baseline-ns pod-security.kubernetes.io/warn=baseline
 kubectl label --overwrite ns restricted-ns pod-security.kubernetes.io/enforce=restricted
namespace/baseline-ns labeled
namespace/restricted-ns labeled
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

These commands achieve the following result:

- Workloads in the baseline-ns namespace that violate the baseline policy are allowed, and the client displays a warning message.
- Workloads in the restricted-ns namespace that violate the restricted policy are rejected, and GKE adds an entry to the audit logs.

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl get ns --show-labels
NAME                 STATUS   AGE     LABELS
baseline-ns          Active   100s    kubernetes.io/metadata.name=baseline-ns,pod-security.kubernetes.io/warn=baseline
default              Active   6m10s   kubernetes.io/metadata.name=default
gke-managed-system   Active   5m44s   addonmanager.kubernetes.io/mode=Reconcile,kubernetes.io/metadata.name=gke-managed-system
gmp-public           Active   5m28s   addonmanager.kubernetes.io/mode=Reconcile,kubernetes.io/metadata.name=gmp-public
gmp-system           Active   5m28s   addonmanager.kubernetes.io/mode=Reconcile,kubernetes.io/metadata.name=gmp-system
kube-node-lease      Active   6m10s   kubernetes.io/metadata.name=kube-node-lease
kube-public          Active   6m11s   kubernetes.io/metadata.name=kube-public
kube-system          Active   6m11s   kubernetes.io/metadata.name=kube-system
restricted-ns        Active   99s     kubernetes.io/metadata.name=restricted-ns,pod-security.kubernetes.io/enforce=restricted
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```



### Test the configured policies

To verify that the PodSecurity admission controller works as  intended, deploy a workload that violates the baseline and the  restricted policy to both namespaces. The following example manifest  deploys an nginx container that allows privilege escalation.

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ cat psa-workload.yaml 
 apiVersion: v1
 kind: Pod
 metadata:
   name: nginx
   labels:
     app: nginx
 spec:
   containers:
   - name: nginx
     image: nginx
     securityContext:
       privileged: true
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl apply -f psa-workload.yaml --namespace=baseline-ns
Warning: would violate PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
pod/nginx created
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

The baseline policy allows the Pod to deploy in the namespace.

Verify that the Pod deployed successfully:

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl get pods --namespace=baseline-ns -l=app=nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          47s
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

Apply the manifest to the restricted-ns namespace:

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$  kubectl apply -f psa-workload.yaml --namespace=restricted-ns
Error from server (Forbidden): error when creating "psa-workload.yaml": pods "nginx" is forbidden: violates PodSecurity "restricted:latest": privileged (container "nginx" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

The Pod won't deploy in the namespace. An audit entry is added to the log.

### View policy violations in the audit logs

Policy violations in the audit and enforce modes are recorded in the  audit logs for your cluster. You can view these logs using the Logs  Explorer in the Google Cloud console.

1. On the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **VIEW ALL PRODUCTS**. In the **Observability** section, click **Logging > Logs Explorer**.
2. In the **Query** field, specify the following:

```sh
 resource.type="k8s_cluster"
 protoPayload.response.reason="Forbidden"
 protoPayload.resourceName="core/v1/namespaces/restricted-ns/pods/nginx"
```

```json
{
  "protoPayload": {
    "@type": "type.googleapis.com/google.cloud.audit.AuditLog",
    "authenticationInfo": {
      "principalEmail": "student-01-4f5493923909@qwiklabs.net"
    },
    "authorizationInfo": [
      {
        "permission": "io.k8s.core.v1.pods.create",
        "resource": "core/v1/namespaces/restricted-ns/pods/nginx"
      }
    ],
    "methodName": "io.k8s.core.v1.pods.create",
    "request": {
      "@type": "core.k8s.io/v1.Pod",
      "apiVersion": "v1",
      "kind": "Pod",
      "metadata": {
        "annotations": {
          "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"labels\":{\"app\":\"nginx\"},\"name\":\"nginx\",\"namespace\":\"restricted-ns\"},\"spec\":{\"containers\":[{\"image\":\"nginx\",\"name\":\"nginx\",\"securityContext\":{\"privileged\":true}}]}}\n"
        },
        "creationTimestamp": null,
        "labels": {
          "app": "nginx"
        },
        "name": "nginx",
        "namespace": "restricted-ns"
      },
      "spec": {
        "containers": [
          {
            "image": "nginx",
            "imagePullPolicy": "Always",
            "name": "nginx",
            "resources": {},
            "securityContext": {
              "privileged": true
            },
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File"
          }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "terminationGracePeriodSeconds": 30
      },
      "status": {}
    },
    "requestMetadata": {
      "callerIp": "34.143.239.200",
      "callerSuppliedUserAgent": "kubectl/v1.29.5 (linux/amd64) kubernetes/59755ff"
    },
    "resourceName": "core/v1/namespaces/restricted-ns/pods/nginx",
    "response": {
      "@type": "core.k8s.io/v1.Status",
      "apiVersion": "v1",
      "code": 403,
      "details": {
        "kind": "pods",
        "name": "nginx"
      },
      "kind": "Status",
      "message": "pods \"nginx\" is forbidden: violates PodSecurity \"restricted:latest\": privileged (container \"nginx\" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container \"nginx\" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container \"nginx\" must set securityContext.capabilities.drop=[\"ALL\"]), runAsNonRoot != true (pod or container \"nginx\" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container \"nginx\" must set securityContext.seccompProfile.type to \"RuntimeDefault\" or \"Localhost\")",
      "metadata": {},
      "reason": "Forbidden",
      "status": "Failure"
    },
    "serviceName": "k8s.io",
    "status": {
      "code": 7,
      "message": "pods \"nginx\" is forbidden: violates PodSecurity \"restricted:latest\": privileged (container \"nginx\" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container \"nginx\" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container \"nginx\" must set securityContext.capabilities.drop=[\"ALL\"]), runAsNonRoot != true (pod or container \"nginx\" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container \"nginx\" must set securityContext.seccompProfile.type to \"RuntimeDefault\" or \"Localhost\")"
    }
  },
  "insertId": "44d5f968-2cea-4d09-839a-f8a3c3d4a326",
  "resource": {
    "type": "k8s_cluster",
    "labels": {
      "cluster_name": "standard-cluster-1",
      "project_id": "qwiklabs-gcp-01-89f4cbcd0816",
      "location": "us-west1-c"
    }
  },
  "timestamp": "2024-06-26T04:10:49.129747Z",
  "labels": {
    "authorization.k8s.io/decision": "allow",
    "pod-security.kubernetes.io/enforce-policy": "restricted:latest",
    "mutation.webhook.admission.k8s.io/round_0_index_4": "{\"configuration\":\"warden-mutating.config.common-webhooks.networking.gke.io\",\"webhook\":\"warden-mutating.common-webhooks.networking.gke.io\",\"mutated\":false}",
    "mutation.webhook.admission.k8s.io/round_0_index_3": "{\"configuration\":\"pod-ready.config.common-webhooks.networking.gke.io\",\"webhook\":\"pod-ready.common-webhooks.networking.gke.io\",\"mutated\":false}",
    "authorization.k8s.io/reason": "access granted by IAM permissions."
  },
  "logName": "projects/qwiklabs-gcp-01-89f4cbcd0816/logs/cloudaudit.googleapis.com%2Factivity",
  "operation": {
    "id": "44d5f968-2cea-4d09-839a-f8a3c3d4a326",
    "producer": "k8s.io",
    "first": true,
    "last": true
  },
  "receiveTimestamp": "2024-06-26T04:11:07.021037923Z"
}
```



## Rotate IP Address and Credentials

You perform IP and credential rotation on your cluster. It is a  secure practice to do so regularly to reduce credential lifetimes. While there are separate commands to rotate the serving IP and credentials,  rotating credentials **additionally** rotates the IP as well.

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ gcloud container clusters update $my_cluster --zone $my_zone --start-credential-rotation
This will start an IP and Credentials Rotation on cluster [standard-cluster-1]. The master will be updated to serve on a new IP address in 
addition to the current IP address, and cluster credentials will be rotated. Kubernetes Engine will then schedule recreation of all nodes (3 
nodes) to point to the new IP address. If maintenence window is used, nodes are not recreated until a maintenance window occurs. See 
documentation https://cloud.google.com/kubernetes-engine/docs/how-to/credential-rotation on how to manually update nodes. This operation is 
long-running and will block other operations on the cluster (including delete) until it has run to completion.

Do you want to continue (Y/n)?  y

Updating standard-cluster-1...done.                                                                                                         
Updated [https://container.googleapis.com/v1/projects/qwiklabs-gcp-01-89f4cbcd0816/zones/us-west1-c/clusters/standard-cluster-1].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-c/standard-cluster-1?project=qwiklabs-gcp-01-89f4cbcd0816
kubeconfig entry generated for standard-cluster-1.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```



After the command completes in the Cloud Shell the cluster will initiate the process to update each of the nodes. That process can take `up to 15 minutes` for your cluster. The process also automatically updates the kubeconfig entry for the current user.

The cluster master now temporarily serves the new IP address in addition to the original address.

You must update the kubeconfig file on any other system that uses  kubectl or API to access the master before completing the rotation  process to avoid losing access. 

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ kubectl get nodes -o wide
NAME                                                STATUS   ROLES    AGE   VERSION               INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
gke-standard-cluster-1-default-pool-cafc2d20-3z6g   Ready    <none>   19m   v1.29.4-gke.1043002   10.138.0.3    34.83.1.146      Container-Optimized OS from Google   6.1.75+          containerd://1.7.13
gke-standard-cluster-1-default-pool-cafc2d20-q4qz   Ready    <none>   19m   v1.29.4-gke.1043002   10.138.0.5    34.168.81.155    Container-Optimized OS from Google   6.1.75+          containerd://1.7.13
gke-standard-cluster-1-default-pool-cafc2d20-xgql   Ready    <none>   19m   v1.29.4-gke.1043002   10.138.0.4    35.185.253.204   Container-Optimized OS from Google   6.1.75+          containerd://1.7.13
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$
```

To complete the credential and IP rotation tasks execute the following command:

```ssh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ gcloud container clusters update $my_cluster --zone $my_zone --complete-credential-rotation
This will complete the in-progress Credential Rotation on cluster [standard-cluster-1]. The master will be updated to stop serving on the old
 IP address and only serve on the new IP address. Old cluster credentials will be invalidated. Make sure all API clients have been updated to
 communicate with the new IP address (e.g. by running `gcloud container clusters get-credentials --project qwiklabs-gcp-01-89f4cbcd0816 
--location us-west1-c standard-cluster-1`). If maintenence window is used, nodes are not recreated until a maintenance window occurs. See 
documentation https://cloud.google.com/kubernetes-engine/docs/how-to/credential-rotation on how to manually update nodes. This operation is 
long-running and will block other operations on the cluster (including delete) until it has run to completion.

Do you want to continue (Y/n)?  y

ERROR: (gcloud.container.clusters.update) ResponseError: code=400, message=Node pool "default-pool" requires recreation.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

This finalizes the rotation processes and removes the original cluster ip-address.

If the credential rotation fails to complete and returns an error message, run the below command.

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ gcloud container clusters upgrade $my_cluster     --node-pool=default-pool   --zone $my_zone
All nodes in node pool [default-pool] of cluster [standard-cluster-1] will be upgraded from version [1.29.4-gke.1043002] to version 
[1.29.4-gke.1043002]. This operation is long-running and will block other operations on the cluster (including delete) until it has run to 
completion.

Do you want to continue (Y/n)?  Y

ERROR: (gcloud.container.clusters.upgrade) ResponseError: code=400, message=Cluster is running incompatible operation operation-1719375723350-a8ca6c87-b62b-493d-8cd1-0c0ce45e3407.
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

## History

```sh
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ history 
    1  export my_zone=us-west1-c
    2  export my_cluster=standard-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud config set project qwiklabs-gcp-01-89f4cbcd0816 
    5  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    6   kubectl create ns baseline-ns
    7   kubectl create ns restricted-ns
    8   kubectl label --overwrite ns baseline-ns pod-security.kubernetes.io/warn=baseline
    9   kubectl label --overwrite ns restricted-ns pod-security.kubernetes.io/enforce=restricted
   10   kubectl get ns --show-labels
   11  nano psa-workload.yaml
   12  cat psa-workload.yaml 
   13   kubectl apply -f psa-workload.yaml --namespace=baseline-ns
   14   kubectl get pods --namespace=baseline-ns -l=app=nginx
   15   kubectl apply -f psa-workload.yaml --namespace=restricted-ns
   16  gcloud container clusters update $my_cluster --zone $my_zone --start-credential-rotation
   17  kubectl get nodes -o wide
   18  gcloud container clusters update $my_cluster --zone $my_zone --complete-credential-rotation
   19  gcloud container clusters upgrade $my_cluster     --node-pool=default-pool   --zone $my_zone
   20  gcloud container clusters upgrade $my_cluster     --node-pool=default-pool   --zone $my_zone
   21  gcloud container clusters update $my_cluster --zone $my_zone --complete-credential-rotation
   22  history 
student_01_4f5493923909@cloudshell:~ (qwiklabs-gcp-01-89f4cbcd0816)$ 
```

