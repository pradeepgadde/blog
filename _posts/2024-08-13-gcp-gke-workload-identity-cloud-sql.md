---

layout: single
title:  "Using Cloud SQL with Google Kubernetes Engine and Workload Identity"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-gke.png
  og_image: /assets/images/gcp-gke.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Using Cloud SQL with Google Kubernetes Engine and Workload Identity

First you create a GKE cluster, next you create a Cloud SQL Instance to  connect to, and a Service Account to provide permission for your Pods to access the Cloud SQL Instance, this will be authenticated using  Workload Identity. Finally you deploy WordPress on your GKE cluster,  with the SQL Proxy as a Sidecar, connected to your Cloud SQL Instance.

The SQL Proxy lets you interact with a Cloud SQL instance as if it were  installed locally (localhost:3306), and even though you are on an  unsecured port locally, the SQL Proxy makes sure you are secure over the wire to your Cloud SQL Instance.

- Create a Cloud SQL instance and database for Wordpress.
- Create credentials and Kubernetes Secrets for application authentication.
- Configure Workload Identity.
- Configure a Deployment with a Wordpress image to use SQL Proxy.
- Install SQL Proxy as a sidecar container and use it to provide SSL access to a CloudSQL instance external to the GKE Cluster.

## Connect to the lab GKE cluster

1. In Cloud Shell, type the following command to set the environment variable for the Google Cloud zone and cluster name:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-2911259c1b50.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ export my_cluster=autopilot-cluster-1
export my_project=$(gcloud config get-value project)
export my_region=us-east4
Your active configuration is: [cloudshell-11479]
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ source <(kubectl completion bash)
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud container clusters get-credentials $my_cluster --region $my_region
ERROR: (gcloud.container.clusters.get-credentials) You do not currently have an active account selected.
Please run:

  $ gcloud auth login

to obtain new credentials.

If you have already logged in with a different account, run:

  $ gcloud config set account ACCOUNT

to select an already authenticated account to use.
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud auth login

You are already authenticated with gcloud when running
inside the Cloud Shell and so do not need to run this
command. Do you wish to proceed anyway?

Do you want to continue (Y/n)?  y

Go to the following link in your browser, and complete the sign-in prompts:

    https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&redirect_uri=https%3A%2F%2Fsdk.cloud.google.com%2Fauthcode.html&scope=openid+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fappengine.admin+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fsqlservice.login+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcompute+https%3A%2F%2Fwww.googleapis.com%2Fauth%2Faccounts.reauth&state=v7LXCRFzPMvsNaAxsYfktoDk4FS15J&prompt=consent&token_usage=remote&access_type=offline&code_challenge=f8IknZ-m9HR4MSzfutOx_shAvhF6rU0YEq81QdkHep4&code_challenge_method=S256

Once finished, enter the verification code provided in your browser: 4/0AcvDMrD9o6802Yu7lsjpDNpSKN9JmMFSkQ4xULPpRb7X1nq7YBLGRcb3NsglYulls0HGJA

You are now logged in as [student-01-ceb2542405cf@qwiklabs.net].
Your current project is [qwiklabs-gcp-04-2911259c1b50].  You can change this setting by running:
  $ gcloud config set project PROJECT_ID
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

## Enable Cloud SQL APIs

1. In the Google Cloud Console, on the **Navigation menu** (![Navigation menu icon](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)), click **APIs & Services**.
2. Click **+ Enable APIs and Services**.
3. For **Search for APIs & Services**, type **SQL** and then click the **Cloud SQL** API tile.
4. Click **Enable** to enable Cloud SQL API.

If the API is already enabled, a **Manage** button appears instead, with an **API enabled** message. In that case, no action is required.

1. Repeat the above step to enable **sqladmin API**.

## Create a Cloud SQL instance

1. In the Cloud Shell, run the following command to create the instance:

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud sql instances create sql-instance --tier=db-n1-standard-2 --region=$my_region
Creating Cloud SQL instance for MYSQL_8_0...working.                                                                                                                               
Creating Cloud SQL instance for MYSQL_8_0...done.                                                                                                                                  
Created [https://sqladmin.googleapis.com/sql/v1beta4/projects/qwiklabs-gcp-04-2911259c1b50/instances/sql-instance].
NAME: sql-instance
DATABASE_VERSION: MYSQL_8_0
LOCATION: us-east4-b
TIER: db-n1-standard-2
PRIMARY_ADDRESS: 34.85.199.168
PRIVATE_ADDRESS: -
STATUS: RUNNABLE
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ export SQL_NAME=qwiklabs-gcp-04-2911259c1b50:us-east4:sql-instance
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud sql connect sql-instance
Allowlisting your IP for incoming connection for 5 minutes...done.                                                                                                                 
Connecting to database with SQL user [root].Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 47
Server version: 8.0.31-google (Google)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database wordpress;
Query OK, 1 row affected (0.07 sec)

mysql> use wordpress;
Database changed
mysql> show tables;
Empty set (0.07 sec)

mysql> exit
Bye
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

## Prepare a Service Account with permission to access Cloud SQL

1. To create a Service Account, in the Google Cloud console navigate to **IAM & Admin**> **Service Accounts**.
2. Click **+ Create Service Account**.
3. Specify the **Service account name** called `sql-access` then click **Create and Continue**.
4. Click **Select a role.**
5. Search for **Cloud SQL**, select **Cloud SQL Client** and click **Continue**.
6. Click **Done**.



## Create Kubernetes Service Account and configure Workload Identity

1. In the Cloud Shell, run the following command to create the Kubernetes Service Account:

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud container clusters get-credentials autopilot-cluster-1 --region us-east4
Fetching cluster endpoint and auth data.
kubeconfig entry generated for autopilot-cluster-1.
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl get nodes
No resources found
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl create serviceaccount gkesqlsa
serviceaccount/gkesqlsa created
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ gcloud iam service-accounts add-iam-policy-binding \
--role="roles/iam.workloadIdentityUser" \
--member="serviceAccount:$my_project.svc.id.goog[default/gkesqlsa]" \
sql-access@$my_project.iam.gserviceaccount.com
Updated IAM policy for serviceAccount [sql-access@qwiklabs-gcp-04-2911259c1b50.iam.gserviceaccount.com].
bindings:
- members:
  - serviceAccount:qwiklabs-gcp-04-2911259c1b50.svc.id.goog[default/gkesqlsa]
  role: roles/iam.workloadIdentityUser
etag: BwYfqvfHjJM=
version: 1
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl annotate serviceaccount \
gkesqlsa \
iam.gke.io/gcp-service-account=sql-access@$my_project.iam.gserviceaccount.com
serviceaccount/gkesqlsa annotated
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

## Create Secrets

You create two Kubernetes Secrets: one to provide the MySQL  credentials and one to provide the Google credentials (the service  account).

1. To create a Secret for your MySQL credentials, enter the following in the Cloud Shell:

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl create secret generic sql-credentials \
   --from-literal=username=sqluser\
   --from-literal=password=sqlpassword
secret/sql-credentials created
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

## Deploy the SQL Proxy agent as a sidecar container

Let's create a deployment manifest file called `sql-proxy.yaml` that deploys a demo Wordpress application container with the SQL Proxy agent as a sidecar container.

In the Wordpress container environment settings the WORDPRESS_DB_HOST is specified using the localhost IP address. The `cloudsql-proxy` sidecar container is configured to point to the Cloud SQL instance you  created in the previous task. The database username and password are  passed to the Wordpress container as secret keys, and Workload Identity  is configured. A Service is also created to allow you to connect to the  Wordpress instance from the internet.

Create and open a file called `sql-proxy.yaml` with **nano** using the following command:

```yaml
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ cat sql-proxy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      serviceAccountName: gkesqlsa
      containers:
        - name: web
          image: gcr.io/cloud-marketplace/google/wordpress:6.1
          #image: wordpress:5.9
          ports:
            - containerPort: 80
          env:
            - name: WORDPRESS_DB_HOST
              value: 127.0.0.1:3306
            # These secrets are required to start the pod.
            # [START cloudsql_secrets]
            - name: WORDPRESS_DB_USER
              valueFrom:
                secretKeyRef:
                  name: sql-credentials
                  key: username
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: sql-credentials
                  key: password
            # [END cloudsql_secrets]
        # Change '<INSTANCE_CONNECTION_NAME>' here to include your Google Cloud
        # project, the region of your Cloud SQL instance and the name
        # of your Cloud SQL instance. The format is
        # $PROJECT:$REGION:$INSTANCE
        # [START proxy_container]
        - name: cloudsql-proxy
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.8.0
          args: 
           - "--structured-logs"
           - "--port=3306"
           -  "<INSTANCE_CONNECTION_NAME>" 
          securityContext:
            runAsNonRoot: true 
---
apiVersion: "v1"
kind: "Service"
metadata:
  name: "wordpress-service"
  namespace: "default"
  labels:
    app: "wordpress"
spec:
  ports:
  - protocol: "TCP"
    port: 80
  selector:
    app: "wordpress"
  type: "LoadBalancer"
  loadBalancerIP: ""

student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

The important sections to note in this manifest are:

- In the `spec` section the Kubernetes Service Account is configured.
- In the Wordpress env section the variable  `WORDPRESS_DB_HOST` is set to `127.0.0.1:3306.` This will connect to a container in the same Pod listening on port  3306. This is the port that the SQL-Proxy listens on by default.
- In the Wordpress `env` section the variables `WORDPRESS_DB_USER` and `WORDPRESS_DB_PASSWORD` are set using values stored in the `sql-credential` Secret you created in the last task.
- In the `cloudsql-proxy` container section the command switch  that defines the SQL Connection name, `"INSTANCE_CONNECTION_NAME>` contains a placeholder variable that is not configured using a  ConfigMap or Secret and so must be updated directly in this example  manifest to point to your Cloud SQL instance.
- The Service section at the end creates an external LoadBalancer called `"wordpress-service`" that  allows the application to be accessed from external internet addresses.

1. Use `sed` to update the placeholder variable for the SQL Connection name to the instance name of your Cloud SQL instance:

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ sed -i 's/<INSTANCE_CONNECTION_NAME>/'"${SQL_NAME}"'/g'\
   sql-proxy.yaml
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ cat sql-proxy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      serviceAccountName: gkesqlsa
      containers:
        - name: web
          image: gcr.io/cloud-marketplace/google/wordpress:6.1
          #image: wordpress:5.9
          ports:
            - containerPort: 80
          env:
            - name: WORDPRESS_DB_HOST
              value: 127.0.0.1:3306
            # These secrets are required to start the pod.
            # [START cloudsql_secrets]
            - name: WORDPRESS_DB_USER
              valueFrom:
                secretKeyRef:
                  name: sql-credentials
                  key: username
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: sql-credentials
                  key: password
            # [END cloudsql_secrets]
        # Change 'qwiklabs-gcp-04-2911259c1b50:us-east4:sql-instance' here to include your Google Cloud
        # project, the region of your Cloud SQL instance and the name
        # of your Cloud SQL instance. The format is
        # $PROJECT:$REGION:$INSTANCE
        # [START proxy_container]
        - name: cloudsql-proxy
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.8.0
          args: 
           - "--structured-logs"
           - "--port=3306"
           -  "qwiklabs-gcp-04-2911259c1b50:us-east4:sql-instance" 
          securityContext:
            runAsNonRoot: true 
---
apiVersion: "v1"
kind: "Service"
metadata:
  name: "wordpress-service"
  namespace: "default"
  labels:
    app: "wordpress"
spec:
  ports:
  - protocol: "TCP"
    port: 80
  selector:
    app: "wordpress"
  type: "LoadBalancer"
  loadBalancerIP: ""

student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl apply -f sql-proxy.yaml
Warning: autopilot-default-resources-mutator:Autopilot updated Deployment default/wordpress: defaulted unspecified 'cpu' resource for containers [web, cloudsql-proxy] (see http://g.co/gke/autopilot-defaults).
deployment.apps/wordpress created
service/wordpress-service created
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl get deployment wordpress
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   1/1     1            1           2m48s
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

```sh
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ kubectl get services
NAME                TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
kubernetes          ClusterIP      34.118.224.1     <none>         443/TCP        35m
wordpress-service   LoadBalancer   34.118.232.191   34.48.143.75   80:30304/TCP   3m7s
student_01_ceb2542405cf@cloudshell:~ (qwiklabs-gcp-04-2911259c1b50)$ 
```

## Connect to your Wordpress instance

1. Open a new browser tab and connect to your Wordpress site using the  external LoadBalancer ip-address. This will start the initial Wordpress  installation wizard.

2. Select **English (United States)** and click **Continue**.

3. Enter a sample name for the **Site Title**.

4. Enter a **Username** and **Password** to administer the site.

5. Enter an email address.

6. None of these values are particularly important, you will not need to use them.

   1. Click **Install Wordpress**.

   After a few seconds you will see the **Success!** Notification.  You can log in if you wish to explore the Wordpress admin interface but it is not required for the lab.

   The initialization process has created new database tables and data  in the wordpress database on your Cloud SQL instance. You will now  validate that these new database tables have been created using the SQL  proxy container.

   1. Switch back to the Cloud Shell and connect to your Cloud SQL instance:

   2. ```
      gcloud sql connect sql-instance
      use wordpress;
      show tables;
      
      ```

This will now show a number of new database tables that were created  when Wordpress was initialized demonstrating that the sidecar SQL Proxy  container was configured correctly:

```sh
select * from wp_users;
```

