---

layout: single
title:  "GKE Backup and Restore"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-gke.png
  og_image: /assets/images/gcp-gke.png
  teaser: /assets/images/pac-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pac-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# GKE Backup and Restore

[Backup for GKE](https://cloud.google.com/kubernetes-engine/docs/add-on/backup-for-gke/concepts/backup-for-gke) is a service for backing up and restoring workloads in GKE clusters. It has two components:

- A Google Cloud API serves as the control plane for the service.
- A GKE add-on (the Backup for GKE agent) must be enabled in each  cluster for which you wish to perform backup and restore operations.

Backups of your workloads may be useful for disaster recovery, CI/CD  pipelines, cloning workloads, or upgrade scenarios. Protecting your  workloads can help you achieve business-critical recovery point  objectives.

- Enable Backup for a GKE cluster
- Deploy a stateful application with a database on GKE
- Plan and backup GKE workloads
- Restore a backup

### Set up the environment

A GKE cluster and a static external IP address were provisioned as part of the lab setup.

- Run the following commands to set the required environment variables:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-3f9aaf332b35.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_802c2e6ada6b@cloudshell:~ (qwiklabs-gcp-03-3f9aaf332b35)$ echo "export ZONE=us-central1-a" >> ~/.bashrc
echo "export REGION=us-central1" >> ~/.bashrc
echo "export PROJECT_ID=`gcloud config get-value core/project`" >> ~/.bashrc
echo "export BACKUP_PLAN=my-backup-plan" >> ~/.bashrc
source ~/.bashrc
echo "export EXTERNAL_ADDRESS=$(gcloud compute addresses describe app-address --format='value(address)' --region $REGION)" >> ~/.bashrc
source ~/.bashrc
Your active configuration is: [cloudshell-3558]
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```



## Enable Backup for GKE

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud services enable gkebackup.googleapis.com
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container clusters update lab-cluster \
--project=$PROJECT_ID  \
--update-addons=BackupRestore=ENABLED \
--zone=$ZONE
Updating lab-cluster...working...                                                                                                                                                  
Updating lab-cluster...working..                                                                                                                                                   
Updating lab-cluster...working                                                                               Updating lab-cluster...done.          
Updated [https://container.googleapis.com/v1beta1/projects/qwiklabs-gcp-03-3f9aaf332b35/zones/us-central1-a/clusters/lab-cluster].
To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-central1-a/lab-cluster?project=qwiklabs-gcp-03-3f9aaf332b35
student_01_802c2e6ada6b@cs-52335668542-default:~$  
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container clusters describe lab-cluster \
--project=$PROJECT_ID  \
--zone=$ZONE | grep -A 1 gkeBackupAgentConfig:
  gkeBackupAgentConfig:
    enabled: true
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Create a backup plan

1. Run the following to create a backup plan:

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backup-plans create $BACKUP_PLAN \
--project=$PROJECT_ID \
--location=$REGION \
--cluster=projects/${PROJECT_ID}/locations/${ZONE}/clusters/lab-cluster \
--all-namespaces \
--include-secrets \
--include-volume-data \
--cron-schedule="10 3 * * *" \
--backup-retain-days=30
Create request issued for: [my-backup-plan]
Waiting for operation [projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/operations/operation-1716852455392-61977dcbc43f4-be64dcd7
-bced39eb] to complete...done.                                                                                                              
Created backup plan [my-backup-plan].
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backup-plans list \
--project=$PROJECT_ID \
--location=$REGION
NAME: my-backup-plan
LOCATION: us-central1
CLUSTER: lab-cluster
ACTIVE: Y
PAUSED: N
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backup-plans describe $BACKUP_PLAN \
--project=$PROJECT_ID \
--location=$REGION
backupConfig:
  allNamespaces: true
  includeSecrets: true
  includeVolumeData: true
backupSchedule:
  cronSchedule: 10 3 * * *
  nextScheduledBackupTime: '2024-05-28T03:10:00Z'
cluster: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1-a/clusters/lab-cluster
createTime: '2024-05-27T23:27:36.657552088Z'
etag: '2'
name: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/backupPlans/my-backup-plan
retentionPolicy:
  backupRetainDays: 30
rpoRiskLevel: 2
rpoRiskReason: No RPO config is defined. Switch to RPO schedule for better protection.
state: READY
stateReason: Resource has been created successfully.
uid: 16e421c9-6c9d-4772-9ae4-f647f87adf43
updateTime: '2024-05-27T23:29:30.476506293Z'
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Deploy WordPress with MySQL to the cluster

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ # Password for lab only. Change to a strong one in your environment.
YOUR_SECRET_PASSWORD=1234567890
kubectl create secret generic mysql-pass --from-literal=password=${YOUR_SECRET_PASSWORD?}
kubectl apply -f https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
kubectl apply -f https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
secret/mysql-pass created
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ patch_file=/tmp/loadbalancer-patch.yaml
cat <<EOF > ${patch_file}
spec:
  loadBalancerIP: ${EXTERNAL_ADDRESS}
EOF
kubectl patch service/wordpress --patch "$(cat ${patch_file})"
service/wordpress patched
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ while ! curl --fail --max-time 5 --output /dev/null --show-error --silent http://${EXTERNAL_ADDRESS}; do
  sleep 5
done
echo -e "\nhttp://${EXTERNAL_ADDRESS} is accessible\n"
curl: (28) Connection timed out after 5001 milliseconds
curl: (28) Connection timed out after 5001 milliseconds

http://35.184.216.238 is accessible

student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Verify the deployed workload

1. In the Cloud console, navigate to **Kubernetes Engine** > [**Workload**](https://console.cloud.google.com/kubernetes/workload/). You should see the WordPress application and its database.
2. Open a browser window and paste in the URL from the previous step. You should see the following page:
3. Click the **Continue** button and type in the required info. For example:

Make a note of the password and click the **Install WordPress** button.

After you log in to the WordPress application, try to create some new posts and add a few comments to existing posts. After backup/restore,  you want to verify your input still exists.

## Create a backup

1. Create a backup based on the backup plan:

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backups create my-backup1 \
--project=$PROJECT_ID \
--location=$REGION \
--backup-plan=$BACKUP_PLAN \
--wait-for-completion
Create in progress for backup my-backup1 [projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/operations/operation-1716853146830-6197805f2c6be-bd5a0b0e-75b5ca8c].
Creating backup my-backup1...done.                                                                                                          
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Waiting for backup to complete... Backup state: IN_PROGRESS.
Backup completed. Backup state: SUCCEEDED
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backups list \
--project=$PROJECT_ID \
--location=$REGION \
--backup-plan=$BACKUP_PLAN
NAME: my-backup1
LOCATION: us-central1
BACKUP_PLAN: my-backup-plan
CREATE_TIME: 2024-05-27T23:39:06 UTC
COMPLETE_TIME: 2024-05-27T23:40:30 UTC
STATE: SUCCEEDED
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore backups describe my-backup1 \
--project=$PROJECT_ID \
--location=$REGION \
--backup-plan=$BACKUP_PLAN
allNamespaces: true
clusterMetadata:
  backupCrdVersions:
    backupjobs.gkebackup.gke.io: v1
    protectedapplicationgroups.gkebackup.gke.io: v1
    protectedapplications.gkebackup.gke.io: v1
    restorejobs.gkebackup.gke.io: v1
  cluster: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1-a/clusters/lab-cluster
  gkeVersion: v1.28.8-gke.1095000
  k8sVersion: '1.28'
completeTime: '2024-05-27T23:40:30.604094428Z'
configBackupSizeBytes: '519094'
containsSecrets: true
containsVolumeData: true
createTime: '2024-05-27T23:39:06.836575056Z'
deleteLockExpireTime: '2024-05-27T23:39:06.825687927Z'
etag: '8'
manual: true
name: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/backupPlans/my-backup-plan/backups/my-backup1
podCount: 2
resourceCount: 426
retainDays: 30
retainExpireTime: '2024-06-27T00:03:06.825687927Z'
sizeBytes: '56571766'
state: SUCCEEDED
uid: 3a518917-7ec8-4bd0-b314-131c135ec00b
updateTime: '2024-05-27T23:40:32.846357333Z'
volumeCount: 2
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Delete the application

You can restore the backup on the same cluster or a different one. In this lab, you will perform a restore on the same cluster. 

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl delete secret mysql-pass
kubectl delete -f https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
kubectl delete -f https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
secret "mysql-pass" deleted
service "wordpress-mysql" deleted
persistentvolumeclaim "mysql-pv-claim" deleted
deployment.apps "wordpress-mysql" deleted
service "wordpress" deleted
persistentvolumeclaim "wp-pv-claim" deleted
deployment.apps "wordpress" deleted
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
No resources found in default namespace.
student_01_802c2e6ada6b@cs-52335668542-default:~$ echo -e "\nWordPress URL: http://${EXTERNAL_ADDRESS}\n"

WordPress URL: http://35.184.216.238

student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Plan a restore

1. Create a restore plan:

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore restore-plans create my-restore-plan1 \
--project=$PROJECT_ID \
--location=$REGION \
--backup-plan=projects/${PROJECT_ID}/locations/${REGION}/backupPlans/$BACKUP_PLAN \
--cluster=projects/${PROJECT_ID}/locations/${ZONE}/clusters/lab-cluster \
--namespaced-resource-restore-mode=delete-and-restore \
--volume-data-restore-policy=restore-volume-data-from-backup \
--all-namespaces
Create request issued for: [my-restore-plan1]
Waiting for operation [projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/operations/operation-1716853464816-6197818e6db20-f923f1bc
-e846236c] to complete...done.                                                                                                              
Created restore plan [my-restore-plan1].
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore restore-plans list \
--project=$PROJECT_ID \
--location=$REGION
NAME: my-restore-plan1
LOCATION: us-central1
CLUSTER: lab-cluster
BACKUP_PLAN: my-backup-plan
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore restore-plans describe my-restore-plan1 \
--project=$PROJECT_ID \
--location=$REGION
backupPlan: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/backupPlans/my-backup-plan
cluster: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1-a/clusters/lab-cluster
createTime: '2024-05-27T23:44:24.860740618Z'
etag: '2'
name: projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/restorePlans/my-restore-plan1
restoreConfig:
  allNamespaces: true
  namespacedResourceRestoreMode: DELETE_AND_RESTORE
  volumeDataRestorePolicy: RESTORE_VOLUME_DATA_FROM_BACKUP
state: READY
stateReason: Resource has been created successfully.
uid: f4fa83d2-d509-471b-9c32-bf7bc9695ec4
updateTime: '2024-05-27T23:44:25.312226455Z'
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## Restore a backup

1. Restore from the backup

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ gcloud beta container backup-restore restores create my-restore1 \
--project=$PROJECT_ID \
--location=$REGION \
--restore-plan=my-restore-plan1 \
--backup=projects/${PROJECT_ID}/locations/${REGION}/backupPlans/${BACKUP_PLAN}/backups/my-backup1 \
--wait-for-completion
Create in progress for restore my-restore1 [projects/qwiklabs-gcp-03-3f9aaf332b35/locations/us-central1/operations/operation-1716853583086-619781ff3841d-4beabcdd-a75869d2].
Creating restore my-restore1...done.                                                                                                        
Restore completed. Restore state: SUCCEEDED
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     Pending   0          20s
wordpress-mysql-68fd75cb99-dc69f   0/1     Pending   0          20s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     Pending   0          35s
wordpress-mysql-68fd75cb99-dc69f   0/1     Pending   0          35s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     ContainerCreating   0          40s
wordpress-mysql-68fd75cb99-dc69f   0/1     ContainerCreating   0          40s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     ContainerCreating   0          43s
wordpress-mysql-68fd75cb99-dc69f   0/1     ContainerCreating   0          43s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     ContainerCreating   0          46s
wordpress-mysql-68fd75cb99-dc69f   0/1     ContainerCreating   0          46s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     ContainerCreating   0          55s
wordpress-mysql-68fd75cb99-dc69f   0/1     ContainerCreating   0          55s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS              RESTARTS   AGE
wordpress-7867b7f788-nxclr         0/1     ContainerCreating   0          60s
wordpress-mysql-68fd75cb99-dc69f   1/1     Running             0          60s
student_01_802c2e6ada6b@cs-52335668542-default:~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-7867b7f788-nxclr         1/1     Running   0          63s
wordpress-mysql-68fd75cb99-dc69f   1/1     Running   0          63s
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ echo -e "\nWordPress URL: http://${EXTERNAL_ADDRESS}\n"

WordPress URL: http://35.184.216.238

student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

## History

```sh
student_01_802c2e6ada6b@cs-52335668542-default:~$ history 
    1  echo "export ZONE=us-central1-a" >> ~/.bashrc
    2  echo "export REGION=us-central1" >> ~/.bashrc
    3  echo "export PROJECT_ID=`gcloud config get-value core/project`" >> ~/.bashrc
    4  echo "export BACKUP_PLAN=my-backup-plan" >> ~/.bashrc
    5  source ~/.bashrc
    6  echo "export EXTERNAL_ADDRESS=$(gcloud compute addresses describe app-address --format='value(address)' --region $REGION)" >> ~/.bashrc
    7  source ~/.bashrc
    8  gcloud services enable gkebackup.googleapis.com
    9  gcloud beta container clusters update lab-cluster --project=$PROJECT_ID  --update-addons=BackupRestore=ENABLED --zone=$ZONE
   10  gcloud beta container clusters describe lab-cluster --project=$PROJECT_ID  --zone=$ZONE | grep -A 1 gkeBackupAgentConfig:
   11  gcloud beta container backup-restore backup-plans create $BACKUP_PLAN --project=$PROJECT_ID --location=$REGION --cluster=projects/${PROJECT_ID}/locations/${ZONE}/clusters/lab-cluster --all-namespaces --include-secrets --include-volume-data --cron-schedule="10 3 * * *" --backup-retain-days=30
   12  gcloud beta container backup-restore backup-plans list --project=$PROJECT_ID --location=$REGION
   13  gcloud beta container backup-restore backup-plans describe $BACKUP_PLAN --project=$PROJECT_ID --location=$REGION
   14  gcloud container clusters get-credentials lab-cluster --zone=$ZONE
   15  echo "EXTERNAL_ADDRESS=${EXTERNAL_ADDRESS}"
   16  # Password for lab only. Change to a strong one in your environment.
   17  YOUR_SECRET_PASSWORD=1234567890
   18  kubectl create secret generic mysql-pass --from-literal=password=${YOUR_SECRET_PASSWORD?}
   19  kubectl apply -f https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
   20  kubectl apply -f https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
   21  patch_file=/tmp/loadbalancer-patch.yaml
   22  cat <<EOF > ${patch_file}
spec:
  loadBalancerIP: ${EXTERNAL_ADDRESS}
EOF

   23  kubectl patch service/wordpress --patch "$(cat ${patch_file})"
   24  while ! curl --fail --max-time 5 --output /dev/null --show-error --silent http://${EXTERNAL_ADDRESS}; do   sleep 5; done
   25  echo -e "\nhttp://${EXTERNAL_ADDRESS} is accessible\n"
   26  gcloud beta container backup-restore backups create my-backup1 --project=$PROJECT_ID --location=$REGION --backup-plan=$BACKUP_PLAN --wait-for-completion
   27  gcloud beta container backup-restore backups list --project=$PROJECT_ID --location=$REGION --backup-plan=$BACKUP_PLAN
   28  gcloud beta container backup-restore backups describe my-backup1 --project=$PROJECT_ID --location=$REGION --backup-plan=$BACKUP_PLAN
   29  kubectl delete secret mysql-pass
   30  kubectl delete -f https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
   31  kubectl delete -f https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
   32  kubectl get pods
   33  echo -e "\nWordPress URL: http://${EXTERNAL_ADDRESS}\n"
   34  gcloud beta container backup-restore restore-plans create my-restore-plan1 --project=$PROJECT_ID --location=$REGION --backup-plan=projects/${PROJECT_ID}/locations/${REGION}/backupPlans/$BACKUP_PLAN --cluster=projects/${PROJECT_ID}/locations/${ZONE}/clusters/lab-cluster --namespaced-resource-restore-mode=delete-and-restore --volume-data-restore-policy=restore-volume-data-from-backup --all-namespaces
   35  gcloud beta container backup-restore restore-plans list --project=$PROJECT_ID --location=$REGION
   36  gcloud beta container backup-restore restore-plans describe my-restore-plan1 --project=$PROJECT_ID --location=$REGION
   37  gcloud beta container backup-restore restores create my-restore1 --project=$PROJECT_ID --location=$REGION --restore-plan=my-restore-plan1 --backup=projects/${PROJECT_ID}/locations/${REGION}/backupPlans/${BACKUP_PLAN}/backups/my-backup1 --wait-for-completion
   38  kubectl get pods
   39  echo -e "\nWordPress URL: http://${EXTERNAL_ADDRESS}\n"
   40  history 
student_01_802c2e6ada6b@cs-52335668542-default:~$ 
```

