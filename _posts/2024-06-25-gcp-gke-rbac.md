---

layout: single
title:  "Implementing Role-Based Access Control with Google Kubernetes Engine"
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
# Implementing Role-Based Access Control with Google Kubernetes Engine

- Create namespaces for users to control access to cluster resources
- Create roles and RoleBindings to control access within a namespace

## Create namespaces for users to access cluster resources

### Sign in to the Google Cloud Console as the first user

1. Sign in to the Google Cloud Console in an Incognito window as usual with the **Username 1** provided in Qwiklabs. Note that both user names use the same password.
2. On the Google Cloud Console title bar, click **Activate Cloud Shell** (![Activate Cloud Shell icon](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).
3. When prompted, click **Continue**.

You don't need to wait for the Cloud Shell to start, you can proceed to the next task immediately.

Username 2 currently has access to the project, but only possesses the **Viewer** role, which makes all resources in the project visible, but read-only.

### Sign in to the Google Cloud Console as the second user

1. Open another tab in your incognito window.
2. Navigate to [console.cloud.google.com](http://console.cloud.google.com/).
3. Click on the user icon in the top-right corner of the screen, and then click **Add account**.
4. Sign in to the Google Cloud Console with the **Username 2** provided. Again, note that both user names use the same password.
5. On the Google Cloud Console title bar, click **Activate Cloud Shell** (![Activate Cloud Shell icon](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).
6. When prompted, click **Continue**.

### Connect to the lab GKE cluster

1. Switch back to the **Username 1** Google Cloud Console tab.

```sh
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_01_7e9c7754a0ff@cloudshell:~$ gcloud config set project qwiklabs-gcp-03-22b4c70ad62b 
Updated property [core/project].
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ export my_zone=us-west1-b
export my_cluster=standard-cluster-1
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ source <(kubectl completion bash)
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ cd ~/ak8s/RBAC/
-bash: cd: /home/student_01_7e9c7754a0ff/ak8s/RBAC/: No such file or directory
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 65001, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 65001 (delta 0), reused 0 (delta 0), pack-reused 64994
Receiving objects: 100% (65001/65001), 697.18 MiB | 13.21 MiB/s, done.
Resolving deltas: 100% (41534/41534), done.
Updating files: 100% (12864/12864), done.
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ ls ak8s/
Autoscaling  Cloud_SQL    GCP_Console  GKE_Networks  GKE_Shell    Jobs_CronJobs  Probes  README.md  Security  Upgrading_GKE
Cloud_Build  Deployments  GKE_Console  GKE_Services  Helm_Charts  Monitoring     RBAC    Secrets    Storage   v1.1
student_01_7e9c7754a0ff@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ cd ~/ak8s/RBAC/
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ ls
my-namespace.yaml  my-pod.yaml  pod-reader-role.yaml  production-pod.yaml  username2-editor-binding.yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```



### Create a namespace

A manifest file called `my-namespace.yaml` has been created for you that creates a new namespace called `production`.

```yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ cat my-namespace.yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: production
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl create -f ./my-namespace.yaml
namespace/production created
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get namespaces
NAME                 STATUS   AGE
default              Active   15h
gke-managed-system   Active   15h
gmp-public           Active   15h
gmp-system           Active   15h
kube-node-lease      Active   15h
kube-public          Active   15h
kube-system          Active   15h
production           Active   35s
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl describe namespaces production
Name:         production
Labels:       kubernetes.io/metadata.name=production
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```



### Create a resource in a namespace

If you do not specify the namespace of a Pod it will use the  namespace ‘default'. In this task you specify the location of our newly  created namespace when creating a new Pod. A simple manifest file called `my-pod.yaml` that creates a Pod that contains an nginx container has been created for you.

```yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ cat my-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl apply -f ./my-pod.yaml --namespace=production
pod/nginx created
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

Alternatively you could have specified the namespace in the yaml file. This requires the  `namespace: production` field in the metadata: section.

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get pods
No resources found in default namespace.
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get pods --namespace=production
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          36s
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

Now you should see your newly created Pod.

## About roles and RoleBindings

In this task you will create a sample custom role, and then create a RoleBinding that grants **Username 2** the editor role in the production namespace.

The role is defined in the `pod-reader-role.yaml` file that is provided for you. This manifest  defines a role called `pod-reader` that provides create, get, list, and watch permission for Pod objects in the `production` namespace. Note that this role cannot delete Pods.



### Create a custom Role

Before you can create a Role, your account must have the permissions  granted in the role being assigned. For cluster administrators this can  be easily accomplished by creating the following RoleBinding to grant  your own user account the cluster-admin role.

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user student-01-7e9c7754a0ff@qwiklabs.net
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ cat pod-reader-role.yaml 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","get", "list", "watch"]
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl apply -f pod-reader-role.yaml
role.rbac.authorization.k8s.io/pod-reader created
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get roles --namespace production
NAME         CREATED AT
pod-reader   2024-06-26T09:11:22Z
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```



### Create a RoleBinding

The role is used to assign privileges, but by itself it does nothing. The role must be bound to a user and an object, which is done in the  RoleBinding.

The `username2-editor-binding.yaml` manifest file creates a RoleBinding called `username2-editor` for the second lab user to the `pod-reader` role you created earlier. That role can create and view Pods but cannot delete them.

This file contains a placeholder,  `[USERNAME_2_EMAIL]`, that you must replace with  the email address of **Username 2** before your use apply it.

1. In the Cloud Shell create an environment variable that contains the full email address of **Username 2**:

```yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ export USER2=student-01-2e767bd78862@qwiklabs.net
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ sed -i "s/\[USERNAME_2_EMAIL\]/${USER2}/" username2-editor-binding.yaml
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ cat username2-editor-binding.yaml 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: username2-editor
  namespace: production
subjects:
- kind: User
  name: student-01-2e767bd78862@qwiklabs.net
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```



### Test access

Now you will test whether Username 2 can create a Pod in the  production namespace by using Username 2 to create a Pod using the  manifest file `production-pod.yaml`. This manifest deploys a simple Pod with a single nginx container.

Switch back to the **Username 2** Google Cloud Console tab.

In Cloud Shell for **Username 2**, type the following command to set the environment variable for the zone and cluster name:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-22b4c70ad62b.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ export my_zone=us-west1-b
export my_cluster=standard-cluster-1
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ source <(kubectl completion bash)
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ gcloud container clusters get-credentials $my_cluster --zone $my_zone
Fetching cluster endpoint and auth data.
kubeconfig entry generated for standard-cluster-1.
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ cd ~/ak8s/RBAC/
-bash: cd: /home/student_01_2e767bd78862/ak8s/RBAC/: No such file or directory
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ git clone https://github.com/GoogleCloudPlatform/training-data-analyst
Cloning into 'training-data-analyst'...
remote: Enumerating objects: 65001, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 65001 (delta 0), reused 0 (delta 0), pack-reused 64994
Receiving objects: 100% (65001/65001), 697.33 MiB | 14.01 MiB/s, done.
Resolving deltas: 100% (41557/41557), done.
Updating files: 100% (12864/12864), done.
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
student_01_2e767bd78862@cloudshell:~ (qwiklabs-gcp-03-22b4c70ad62b)$ cd ~/ak8s/RBAC/
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get namespaces
NAME                 STATUS   AGE
default              Active   16h
gke-managed-system   Active   16h
gmp-public           Active   16h
gmp-system           Active   16h
kube-node-lease      Active   16h
kube-public          Active   16h
kube-system          Active   16h
production           Active   11m
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

The production namespace appears at the bottom of the list, so we can continue.

1. In the Cloud Shell, execute the following command to create the resource in the namespace called production:

```sh
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl apply -f ./production-pod.yaml
Error from server (Forbidden): error when creating "./production-pod.yaml": pods is forbidden: User "student-01-2e767bd78862@qwiklabs.net" cannot create resource "pods" in API group "" in the namespace "production": requires one of ["container.pods.create"] permission(s).
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

This will fail indicating that **Username 2** does not have the correct permission to create Pods. **Username 2** only has the viewer permissions it started the lab with at this point  because you have not bound any other role to that account yet. You will  now change that.



Switch back to the **Username 1** Google Cloud Console tab.

In the Cloud Shell for **Username 1**, execute the following command to create the RoleBinding that grants Username 2 the `pod-reader` role that includes the permission to create Pods in the `production` namespace:

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl apply -f username2-editor-binding.yaml
rolebinding.rbac.authorization.k8s.io/username2-editor created
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ cat username2-editor-binding.yaml 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: username2-editor
  namespace: production
subjects:
- kind: User
  name: student-01-2e767bd78862@qwiklabs.net
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get rolebinding
No resources found in default namespace.
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

The rolebinding doesn't appear because kubectl is showing the default namespace.

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get rolebinding --namespace production
NAME               ROLE              AGE
username2-editor   Role/pod-reader   61s
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

Switch back to the **Username 2** Google Cloud Console tab.

In the Cloud Shell for **Username 2**, execute the following command to create the resource in the namespace called production:

```sh
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl apply -f ./production-pod.yaml
pod/production-pod created
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

This should now succeed as Username 2 now has the Create permission for Pods in the **production** namespace.

1. Verify the Pod deployed properly in the production namespace by using the following command:

```sh
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl get pods --namespace production
NAME             READY   STATUS    RESTARTS   AGE
nginx            1/1     Running   0          13m
production-pod   1/1     Running   0          33s
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

You should see your newly created Pod.

1. Verify that only the specific RBAC permissions granted by the pod-reader role are in effect for **Username 2** by attempting to delete the production-pod:

```sh
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ kubectl delete pod production-pod --namespace production
Error from server (Forbidden): pods "production-pod" is forbidden: User "student-01-2e767bd78862@qwiklabs.net" cannot delete resource "pods" in API group "" in the namespace "production": requires one of ["container.pods.delete"] permission(s).
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

This fails because **Username 2** does not have the delete permission for Pods.

## History

```sh
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ history 
    1  gcloud config set project qwiklabs-gcp-03-22b4c70ad62b 
    2  export my_zone=us-west1-b
    3  export my_cluster=standard-cluster-1
    4  source <(kubectl completion bash)
    5  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    6  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
    7  cd ~/ak8s/RBAC/
    8  kubectl get namespaces
    9  ls
   10  ls ak8s 
   11  ls ak8s/
   12  kubectl create -f ./my-namespace.yaml
   13  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
   14  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
   15  ls ak8s/
   16  cd ~/ak8s/RBAC/
   17  ls
   18  cat my-namespace.yaml 
   19  kubectl create -f ./my-namespace.yaml
   20  kubectl get namespaces
   21  kubectl describe namespaces production
   22  cat my-pod.yaml 
   23  kubectl apply -f ./my-pod.yaml --namespace=production
   24  kubectl get pods
   25  kubectl get pods --namespace=production
   26  kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user student-01-7e9c7754a0ff@qwiklabs.net
   27  cat pod-reader-role.yaml 
   28  kubectl apply -f pod-reader-role.yaml
   29  kubectl get roles --namespace production
   30  export USER2=student-01-2e767bd78862@qwiklabs.net
   31  sed -i "s/\[USERNAME_2_EMAIL\]/${USER2}/" username2-editor-binding.yaml
   32  cat username2-editor-binding.yaml 
   33  kubectl get namespaces
   34  kubectl apply -f username2-editor-binding.yaml
   35  cat username2-editor-binding.yaml 
   36  kubectl get rolebinding
   37  kubectl get rolebinding --namespace production
   38  history 
student_01_7e9c7754a0ff@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

```sh
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ history 
    1  export my_zone=us-west1-b
    2  export my_cluster=standard-cluster-1
    3  source <(kubectl completion bash)
    4  gcloud container clusters get-credentials $my_cluster --zone $my_zone
    5  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
    6  cd ~/ak8s/RBAC/
    7  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
    8  ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
    9  cd ~/ak8s/RBAC/
   10  kubectl get namespaces
   11  kubectl apply -f ./production-pod.yaml
   12  kubectl get pods --namespace production
   13  kubectl delete pod production-pod --namespace production
   14  history 
student_01_2e767bd78862@cloudshell:~/ak8s/RBAC (qwiklabs-gcp-03-22b4c70ad62b)$ 
```

