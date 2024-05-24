---

layout: single
title:  "Analyzing Findings with Security Command Center"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/ace-gcp.png
author:
  name     : "Associate Cloud Engineer"
  avatar   : "/assets/images/ace-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Analyzing Findings with Security Command Center

[Security Command Center](https://cloud.google.com/security-command-center) (SCC) is a security monitoring platform that helps users:

- Discover security-related misconfigurations of Google Cloud resources.
- Report on active threats in Google Cloud environments.
- Fix vulnerabilities across Google Cloud assets.

- Create a continuous export pipeline to Pub/Sub.
- Export and analyze SCC findings with BigQuery.

## Create a continuous export pipeline to Pub/Sub.

Security Command Center can export security findings to external resources using several methods, including:

- Continuous exports to a BigQuery dataset.
- Continuous exports to Pub/Sub.
- One-time exports to CSV files.
- One-time exports to Cloud Storage buckets as JSON files.

Continuous exports to Pub/Sub are usually used for forwarding  findings to external security management systems such as Splunk or  QRadar.

In this task, you will export your findings to a Pub/Sub topic and  then simulate an application by fetching the messages from a Pub/Sub  subscription.



Before we start configuring an SCC export, we need to create a Pub/Sub topic and subscription.

1. Open the navigation menu and under the **Analytics** header, click **Pub/Sub > Topics**.
2. Click the **Create Topic** button located near the top of the page.
3. Enter in **export-findings-pubsub-topic** for the Topic ID.
4. Keep the other default settings and click **Create**.

This will automatically kick off the creation of both a Pub/Sub topic and an associated subscription.

1. Click **Subscriptions** from the left-hand menu.
2. Click on **export-findings-pubsub-topic-sub**.

This will provide you with a dashboard of statistics and metrics related to the messages published in this subscription.

1. Open the navigation menu and select **Security > Security Command Center > Overview > Settings**.
2. Click on the **Continuous Exports** tab.
3. Click the **Create Pub/Sub Export** button.
4. For the continuous export name, enter in **export-findings-pubsub**.
5. For the continuous export description, enter in **Continuous exports of Findings to Pub/Sub and BigQuery**.
6. For the project name, select the  Project ID of the project you are working in (*do not* select Qwiklabs Resources).
7. From the "Select a Cloud Pub/Sub Topic" dropdown, select **export-findings-pubsub-topic**.
8. Set the findings query to the following: ```state="ACTIVE"
   AND NOT mute="MUTED"```

This query ensures that all new `ACTIVE` and `NOT MUTED` findings will be forwarded to the newly created Pub/Sub topic. Now click **Save**.

You have now successfully created a continuous export from Security  Command Center to Pub/Sub. You will now create new findings and check  how they are exported to Pub/Sub.


1. Open a new Cloud Shell session (![Activate Cloud Shell icon](https://cdn.qwiklabs.com/ep8HmqYGdD%2FkUncAAYpV47OYoHwC8%2Bg0WK%2F8sidHquE%3D)).

2. Run the following command to create a new virtual machine:

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-6b8ce1a11104.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
gcloud compute instances create instance-1 --zone=us-east1-b \
--machine-type e2-micro \
--scopes=https://www.googleapis.com/auth/cloud-platform
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-6b8ce1a11104/zones/us-east1-b/instances/instance-1].
NAME: instance-1
ZONE: us-east1-b
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.142.0.2
EXTERNAL_IP: 34.73.109.105
STATUS: RUNNING
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```

This command will create a new VM instance with a Public IP address and a default Service Account attached.

This activity will immediately generate three new vulnerability findings:

- Public IP address
- Default service account used
- Compute secure boot disabled

1. Open the navigation menu and under the **Analytics** header, click **Pub/Sub > Subscriptions**.
2. Select the **export-findings-pubsub-topic-sub** subscription.
3. Select the **Messages** tab from the center of the Console.
4. Click the **Enable ack messages** checkbox.
5. Click on the **Pull** button.

You should see messages in this subscription that relate to the  public IP address, default service account used, and compute secure boot disabled vulnerabilities.

```sh
May 24, 2024, 2:42:20 PM 	
—
	
{
  "notificationConfigName": "projects/119749827605/notificationConfigs/export-findings-pubsub",
  "finding": {
    "name": "organizations/616463121992/sources/4699549208639402342/findings/b289c81ec55e961663a43fc4813606c1",
    "parent": "organizations/616463121992/sources/4699549208639402342",
    "resourceName": "//compute.googleapis.com/projects/qwiklabs-gcp-03-6b8ce1a11104/zones/us-east1-b/instances/2205207496165811979",
    "state": "ACTIVE",
    "category": "COMPUTE_SECURE_BOOT_DISABLED",
    "externalUri": "https://console.cloud.google.com/compute/instancesDetail/zones/us-east1-b/instances/instance-1?project\u003dqwiklabs-gcp-03-6b8ce1a11104",
    "sourceProperties": {
      "Recommendation": "Go to https://console.cloud.google.com/compute/instancesDetail/zones/us-east1-b/instances/instance-1?project\u003dqwiklabs-gcp-03-6b8ce1a11104 and click \"Stop\". Once the instance has stopped, scroll down to the \"Shielded VM\" section and check the \"Turn on Secure Boot\" checkbox. Then you can start the instance back up by clicking the \"Start\" button.",
      "ExceptionInstructions": "Add the security mark \"allow_compute_secure_boot_disabled\" to the asset with a value of \"true\" to prevent this finding from being activated again.",
      "Explanation": "Using secure boot helps protect your virtual machines against rootkits, boot- and kernel-level malware. Learn more at: https://cloud.google.com/security/shielded-cloud/shielded-vm",
      "ScannerName": "COMPUTE_INSTANCE_SCANNER",
      "ResourcePath": ["projects/qwiklabs-gcp-03-6b8ce1a11104/", "folders/108516522926/", "folders/874212290428/", "folders/365352270458/", "organizations/616463121992/"],
      "ReactivationCount": 0.0
    },
    "securityMarks": {
      "name": "organizations/616463121992/sources/4699549208639402342/findings/b289c81ec55e961663a43fc4813606c1/securityMarks"
    },
    "eventTime": "2024-05-24T09:12:14.426262Z",
    "createTime": "2024-05-24T09:12:15.093Z",
    "severity": "MEDIUM",
    "canonicalName": "projects/119749827605/sources/4699549208639402342/findings/b289c81ec55e961663a43fc4813606c1",
    "mute": "UNDEFINED",
    "findingClass": "MISCONFIGURATION",
    "muteUpdateTime": "1970-01-01T00:00:00Z",
    "parentDisplayName": "Security Health Analytics",
    "description": "Using secure boot helps protect your virtual machines against rootkits, boot- and kernel-level malware. Learn more at: https://cloud.google.com/security/shielded-cloud/shielded-vm"
  },
  "resource": {
    "name": "//compute.googleapis.com/projects/qwiklabs-gcp-03-6b8ce1a11104/zones/us-east1-b/instances/2205207496165811979",
    "project": "//cloudresourcemanager.googleapis.com/projects/119749827605",
    "projectDisplayName": "qwiklabs-gcp-03-6b8ce1a11104",
    "parent": "//cloudresourcemanager.googleapis.com/projects/119749827605",
    "parentDisplayName": "qwiklabs-gcp-03-6b8ce1a11104",
    "type": "google.compute.Instance",
    "folders": [{
      "resourceFolder": "//cloudresourcemanager.googleapis.com/folders/108516522926",
      "resourceFolderDisplayName": "gcp_low_extra navy-03"
    }, {
      "resourceFolder": "//cloudresourcemanager.googleapis.com/folders/874212290428",
      "resourceFolderDisplayName": "gcp_low_extra"
    }, {
      "resourceFolder": "//cloudresourcemanager.googleapis.com/folders/365352270458",
      "resourceFolderDisplayName": "Navy Projects"
    }],
    "displayName": "instance-1",
    "cloudProvider": "GOOGLE_CLOUD_PLATFORM",
    "organization": "organizations/616463121992",
    "service": "compute.googleapis.com",
    "location": "us-east1-b",
    "resourcePath": {
      "nodes": [{
        "nodeType": "GCP_PROJECT",
        "id": "projects/119749827605",
        "displayName": "qwiklabs-gcp-03-6b8ce1a11104"
      }, {
        "nodeType": "GCP_FOLDER",
        "id": "folders/108516522926",
        "displayName": "gcp_low_extra navy-03"
      }, {
        "nodeType": "GCP_FOLDER",
        "id": "folders/874212290428",
        "displayName": "gcp_low_extra"
      }, {
        "nodeType": "GCP_FOLDER",
        "id": "folders/365352270458",
        "displayName": "Navy Projects"
      }, {
        "nodeType": "GCP_ORGANIZATION",
        "id": "organizations/616463121992"
      }]
    },
    "resourcePathString": "organizations/616463121992/folders/365352270458/folders/874212290428/folders/108516522926/projects/119749827605"
  }
}
	

    notificationConfigName finding.name finding.parent finding.resourceName finding.state finding.category finding.externalUri finding.sourceProperties.Recommendation finding.sourceProperties.ExceptionInstructions finding.sourceProperties.Explanation finding.sourceProperties.ScannerName finding.sourceProperties.ResourcePath[0] finding.sourceProperties.ResourcePath[1] finding.sourceProperties.ResourcePath[2] finding.sourceProperties.ResourcePath[3] finding.sourceProperties.ResourcePath[4] finding.sourceProperties.ReactivationCount finding.securityMarks.name finding.eventTime finding.createTime finding.severity finding.canonicalName finding.mute finding.findingClass finding.muteUpdateTime finding.parentDisplayName finding.description resource.name resource.project resource.projectDisplayName resource.parent resource.parentDisplayName resource.type resource.folders[0].resourceFolder resource.folders[0].resourceFolderDisplayName resource.folders[1].resourceFolder resource.folders[1].resourceFolderDisplayName resource.folders[2].resourceFolder resource.folders[2].resourceFolderDisplayName resource.displayName resource.cloudProvider resource.organization resource.service resource.location resource.resourcePath.nodes[0].nodeType resource.resourcePath.nodes[0].id resource.resourcePath.nodes[0].displayName resource.resourcePath.nodes[1].nodeType resource.resourcePath.nodes[1].id resource.resourcePath.nodes[1].displayName resource.resourcePath.nodes[2].nodeType resource.resourcePath.nodes[2].id resource.resourcePath.nodes[2].displayName resource.resourcePath.nodes[3].nodeType resource.resourcePath.nodes[3].id resource.resourcePath.nodes[3].displayName resource.resourcePath.nodes[4].nodeType resource.resourcePath.nodes[4].id resource.resourcePathString 

	— 	COMPUTE_SECURE_BOOT_DISABLED 	2024-05-24T09:12:15.093Z 	2024-05-24T09:12:14.426262Z 	https://console.cloud.google.com/compute/instancesDetail/zones/us-east1-b/instances/instance-1?project=qwiklabs-gcp-03-6b8ce1a11104 	organizations/616463121992/sources/4699549208639402342/findings/b289c81ec55e961663a43fc4813606c1 	organizations/616463121992/sources/4699549208639402342 	//compute.googleapis.com/projects/qwiklabs-gcp-03-6b8ce1a11104/zones/us-east1-b/instances/2205207496165811979 	organizations/616463121992/sources/4699549208639402342/findings/b289c81ec55e961663a43fc4813606c1/securityMarks 	Add the security mark "allow_compute_secure_boot_disabled" to the asset with a value of "true" to prevent this finding from being activated again. 	Using secure boot helps protect your virtual machines against rootkits, boot- and kernel-level malware. Learn more at: https://cloud.google.com/security/shielded-cloud/shielded-vm 	0 	Go to https://console.cloud.google.com/compute/instancesDetail/zones/us-east1-b/instances/instance-1?project=qwiklabs-gcp-03-6b8ce1a11104 and click "Stop". Once the instance has stopped, scroll down to the "Shielded VM" section and check the "Turn on Secure Boot" checkbox. Then you can start the instance back up by clicking the "Start" button. 	projects/qwiklabs-gcp-03-6b8ce1a11104/ 	folders/108516522926/ 	folders/874212290428/ 	folders/365352270458/ 	organizations/616463121992/ 	COMPUTE_INSTANCE_SCANNER 	ACTIVE 	projects/119749827605/notificationConfigs/export-findings-pubsub 	Deadline exceeded	
	

```



By pulling the messages from the Pub/Sub subscription you have  simulated behavior of an application that can forward these messages to  another security monitoring system like Splunk.

In the next task, you will learn how to export and analyze SCC findings with BigQuery.

## Export and Analyze SCC findings with BigQuery

SCC findings can also be exported to a BigQuery dataset. This might  be useful for building analytical dashboards used for checking what type of findings appear in your organization most often.

As of now, configuring continuous exports can only be set using the Command Line Interface (not in the Console).

In your Cloud Shell session, run the following command to create a new BigQuery dataset:

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ PROJECT_ID=$(gcloud config get project)
bq --location=us-east1 --apilog=/dev/null mk --dataset \
$PROJECT_ID:continuous_export_dataset
Your active configuration is: [cloudshell-20500]
Dataset 'qwiklabs-gcp-03-6b8ce1a11104:continuous_export_dataset' successfully created.
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```



We have not used an SCC command line interface in this project yet, so we need to enable the SCC service in this project:

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ gcloud services enable securitycenter.googleapis.com
Operation "operations/acat.p2-119749827605-feace216-a6c7-4c34-9821-27ac46fafe3e" finished successfully.
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```



Now create a new export by entering this command

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ gcloud scc bqexports create scc-bq-cont-export --dataset=projects/qwiklabs-gcp-03-6b8ce1a11104/datasets/continuous_export_dataset --project=qwiklabs-gcp-03-6b8ce1a11104
Created.
dataset: projects/qwiklabs-gcp-03-6b8ce1a11104/datasets/continuous_export_dataset
mostRecentEditor: student-01-51c3500ad3c9@qwiklabs.net
name: projects/119749827605/bigQueryExports/scc-bq-cont-export
principal: service-org-616463121992@gcp-sa-scc-notification.iam.gserviceaccount.com
updateTime: '2024-05-24T09:16:33.310989Z'
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```



Once new findings are exported to BigQuery, SCC will create a new table. You will now initiate new SCC findings.

Run the following commands to create 3 new service accounts without  any IAM permissions and create 3 user-managed service account keys for  them.

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
for i in {0..2}; do
gcloud iam service-accounts create sccp-test-sa-$i;
gcloud iam service-accounts keys create /tmp/sa-key-$i.json \
--iam-account=sccp-test-sa-$i@qwiklabs-gcp-03-6b8ce1a11104.iam.gserviceaccount.com;
done
Created service account [sccp-test-sa-0].
created key [b455682ab103e6ebe155d6adedcadfe74422aa49] of type [json] as [/tmp/sa-key-0.json] for [sccp-test-sa-0@qwiklabs-gcp-03-6b8ce1a11104.iam.gserviceaccount.com]
Created service account [sccp-test-sa-1].
created key [73bfc7ed5bcf95aad98e2d7d5b899bcd2394892c] of type [json] as [/tmp/sa-key-1.json] for [sccp-test-sa-1@qwiklabs-gcp-03-6b8ce1a11104.iam.gserviceaccount.com]
Created service account [sccp-test-sa-2].
created key [a4739c1e71b99d42ebc0a4cce284751e4ddc04a1] of type [json] as [/tmp/sa-key-2.json] for [sccp-test-sa-2@qwiklabs-gcp-03-6b8ce1a11104.iam.gserviceaccount.com]
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```



Once new findings are created in SCC, they will be exported to  BigQuery. For storing them, the export pipeline will create a new table  “findings”.

Fetch from BigQuery information about newly created finding using BigQuery CLI:

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ bq query --apilog=/dev/null --use_legacy_sql=false  "SELECT finding_id,event_time,finding.category FROM continuous_export_dataset.findings"
+----------------------------------+---------------------+----------------------------------+
|            finding_id            |     event_time      |             category             |
+----------------------------------+---------------------+----------------------------------+
| ba5dc51b035f2b0fafee11269f0c99cb | 2024-05-24 09:17:13 | USER_MANAGED_SERVICE_ACCOUNT_KEY |
+----------------------------------+---------------------+----------------------------------+
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```



Very often Security Command Center is enabled in pre-existing and mature Google Cloud infrastructure.

As soon as the SCC is enabled, it starts scanning existing  vulnerabilities and eventually might report thousands of findings on  existing infrastructure.

The SCC interface might not provide the best way to sort and filter  those findings, so exporting these findings to a BigQuery database is a  common practice for running analytics against findings.

Direct exporting of findings to BigQuery is not supported yet.  Instead, you can use a Google Cloud Storage bucket as interim storage.

To export *existing* findings to a BigQuery interface, we will need to export them first to a GCS bucket.

1. Open the navigation menu and select **Cloud Storage > Buckets.**
2. Click the **Create** button.
3. Every bucket name in Google Cloud must be unique. Set the bucket name to **scc-export-bucket-**.
4. Click **Continue**.
5. Set Location type to **Region**.
6. Choose  for the location.
7. Do not change any other settings and click **Create**.
8. Press the **Confirm** button when asked about Enforce public access prevention on this bucket.
9. Open the navigation menu and select **Security > Security Command Center > Findings**.
10. Click the **Export** button.
11. From the dropdown list, select **Cloud Storage**.
12. For the project name, select the Project ID  (*do not* select Qwiklabs Resources).
13. Then select the Export path by clicking the **BROWSE** button.
14. Click the arrow next to the **scc-export-bucket-** button.
15. Set the filename to `findings.jsonl` then click **SELECT**.
16. In the Format drop-down list select **JSONL**.
17. Change the Time Range to **All time**.
18. Do not modify the default findings query.
19. Final "Export to" form might look similar to:
20. Now click the **Export** button.
21. Open the navigation menu and select **BigQuery > BigQuery Studio**.
22. From the left-hand "Explorer" menu, click on the **+ADD** button.
23. In a new "Add" window click on the "Google Cloud Storage" and the set the following parameters:

| **Setting**                         | `Value`                             |
| ----------------------------------- | ----------------------------------- |
| **Create table from**               | `Google Cloud Storage`              |
| **Select the file from GCS bucket** | `scc-export-bucket-/findings.jsonl` |
| **File format**                     | `JSONL`                             |
| **Dataset**                         | `continuous_export_dataset`         |
| **Table**                           | `old_findings`                      |
| **Schema**                          | Enable the "Edit as text" toggle    |

Now paste in the following schema:

```json
[   
  {
    "mode": "NULLABLE",
    "name": "resource",
    "type": "JSON"
  },   
  {
    "mode": "NULLABLE",
    "name": "finding",
    "type": "JSON"
  }
]
```

Then click the **CREATE TABLE** button.

When the new table is created, click the link **GO TO TABLE**.

Click the preview tab and confirm you can view your existing findings.

```sh
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ bq query --apilog=/dev/null --use_legacy_sql=false  "SELECT finding_id,event_time,finding.category FROM continuous_export_dataset.findings"
+----------------------------------+---------------------+------------------------------------------+
|            finding_id            |     event_time      |                 category                 |
+----------------------------------+---------------------+------------------------------------------+
| ba5dc51b035f2b0fafee11269f0c99cb | 2024-05-24 09:17:13 | USER_MANAGED_SERVICE_ACCOUNT_KEY         |
| 637bd283372943c58e97388bc707987c | 2024-05-24 09:17:22 | Persistence: Service Account Key Created |
| bc135ccb79e2a4f01d27ddb93d60f7f2 | 2024-05-24 09:20:55 | BUCKET_LOGGING_DISABLED                  |
| e55ca8a43f59a877eaa11c17649fb872 | 2024-05-24 09:17:17 | USER_MANAGED_SERVICE_ACCOUNT_KEY         |
| d1555b72b4e5b1ef064af5eac4f5f858 | 2024-05-24 09:17:09 | USER_MANAGED_SERVICE_ACCOUNT_KEY         |
+----------------------------------+---------------------+------------------------------------------+
student_01_51c3500ad3c9@cloudshell:~ (qwiklabs-gcp-03-6b8ce1a11104)$ 
```

