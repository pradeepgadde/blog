---

layout: single
title:  "Compute Engine: Qwik Start - Windows"
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
# Compute Engine: Qwik Start - Windows

Compute Engine lets you create and run virtual machines on Google  infrastructure. Compute Engine offers scale, performance, and value that allows you to easily launch large compute clusters on Google's  infrastructure.

You can run your Windows applications on Compute Engine and take  advantage of many benefits available to virtual machine instances, such  as reliable  [storage options](https://cloud.google.com/compute/docs/disks/), the speed of the  [Google network](https://cloud.google.com/compute/docs/vpc), and  [Autoscaling](https://cloud.google.com/compute/docs/autoscaler/).

In this hands-on lab, you learn how to launch a Windows Server  instance in Compute Engine and use Remote Desktop Protocol (RDP) to  connect to it.

## Create a virtual machine instance

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-b02e56e2f212.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ gcloud compute instances create instance-20240519-071626 --project=qwiklabs-gcp-03-b02e56e2f212 --zone=us-central1-c --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=647777874206-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=instance-20240519-071626,image=projects/windows-cloud/global/images/windows-server-2022-dc-v20240415,mode=rw,size=50,type=projects/qwiklabs-gcp-03-b02e56e2f212/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-b02e56e2f212/zones/us-central1-c/instances/instance-20240519-071626].
NAME: instance-20240519-071626
ZONE: us-central1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.67.12.108
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ 
```

## Remote Desktop (RDP) into the Windows Server

### Test the status of Windows Startup

After a short time, the Windows Server instance will be provisioned and listed on the VM Instances page with a green status icon![Green Status Icon](https://cdn.qwiklabs.com/zLABj4YctjgDvpZ2K07lk9smIYG2GsT91dK%2FC9AWNTM%3D).

The server instance may not be ready to accept RDP connections, as it takes a while for all OS components to initialize.

1. To see whether the server instance is ready for an RDP connection,  run the following command at your Cloud Shell terminal command line and  please make to replace `[instance]` with the VM Instance that you created earlier.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ gcloud compute instances get-serial-port-output instance-20240519-071626 --zone=us-central1-c
{snip}
2024/05/19 07:18:49 GCEGuestAgent: GCE Agent Started (version 20240109.00)
2024/05/19 07:18:49 GCEGuestAgent: Adding route to metadata server on adapter with index 2
2024/05/19 07:18:49 GCEGuestAgent: Starting the scheduler to run jobs
2024/05/19 07:18:49 GCEGuestAgent: Scheduler - start: []
2024/05/19 07:18:49 GCEGuestAgent: Skipping scheduling credential generation job, failed to reach client credentials endpoint(instance/credentials/certs) with error: error connecting to metadata server, status code: 404
2024/05/19 07:18:49 GCEGuestAgent: Failed to schedule job MTLS_MDS_Credential_Boostrapper with error: ShouldEnable() returned false, cannot schedule job MTLS_MDS_Credential_Boostrapper
2024/05/19 07:18:49 GCEGuestAgent: Starting the scheduler to run jobs
2024/05/19 07:18:49 GCEGuestAgent: Scheduling job: telemetryJobID
2024/05/19 07:18:49 GCEGuestAgent: Scheduling job "telemetryJobID" to run at 24.000000 hr interval
2024/05/19 07:18:50 GCEGuestAgent: Successfully scheduled job telemetryJobID
2024/05/19 07:19:28 GCEInstanceSetup: Enable google_osconfig_agent during the specialize configuration pass.
2024/05/19 07:19:29 GCEInstanceSetup: Starting sysprep specialize phase.
2024/05/19 07:19:29 GCEInstanceSetup: All networks set to DHCP.
2024/05/19 07:19:29 GCEInstanceSetup: VirtIO network adapter detected.
2024/05/19 07:19:32 GCEInstanceSetup: MTU set to 1460 for IPv4 and IPv6 using PowerShell for interface 2 - Google VirtIO Ethernet Adapter. Build 20348
2024/05/19 07:19:32 GCEInstanceSetup: Running 'route' with arguments '/p add 169.254.169.254 mask 255.255.255.255 0.0.0.0 if 2 metric 1'
2024/05/19 07:19:32 GCEInstanceSetup: --> OK!
2024/05/19 07:19:32 GCEInstanceSetup: Added persistent route to metadata netblock to netkvm adapter.


Specify --start=4007 in the next get-serial-port-output invocation to get only the new output starting from here.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ 

```

Repeat the command until you see the following in the command output,  which tells you that the OS components have initialized and the Windows  Server is ready to accept your RDP connection.



```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ gcloud compute instances get-serial-port-output instance-20240519-071626 --zone=us-central1-c --start=4007
2024/05/19 07:20:21 GCEGuestAgent: GCE Agent Started (version 20240109.00)
2024/05/19 07:20:21 GCEGuestAgent: Starting the scheduler to run jobs
2024/05/19 07:20:21 GCEGuestAgent: Scheduler - start: []
2024/05/19 07:20:22 GCEGuestAgent: Skipping scheduling credential generation job, failed to reach client credentials endpoint(instance/credentials/certs) with error: error connecting to metadata server, status code: 404
2024/05/19 07:20:22 GCEGuestAgent: Failed to schedule job MTLS_MDS_Credential_Boostrapper with error: ShouldEnable() returned false, cannot schedule job MTLS_MDS_Credential_Boostrapper
2024/05/19 07:20:23 GCEGuestAgent: Starting the scheduler to run jobs
2024-05-19T07:20:23.5078Z OSConfigAgent Info: OSConfig Agent (version 20240320.00.0+win@1) started.
2024/05/19 07:20:23 GCEGuestAgent: Scheduling job: telemetryJobID
2024/05/19 07:20:23 GCEGuestAgent: Scheduling job "telemetryJobID" to run at 24.000000 hr interval
2024/05/19 07:20:23 GCEGuestAgent: Invoking job "telemetryJobID"
2024/05/19 07:20:25 GCEInstanceSetup: Enable google_osconfig_agent during the specialize configuration pass.
2024/05/19 07:20:26 GCEInstanceSetup: WinRM certificate details: Subject: CN=instance-20240519-071626, Thumbprint: 5CAFDF82C890C927C73BA9039B9796E803B3C7C6
2024/05/19 07:20:26 GCEInstanceSetup: RDP certificate details: Subject: CN=instance-20240519-071626, Thumbprint: 
2024/05/19 07:20:27 GCEInstanceSetup: 0
2024/05/19 07:20:27 GCEInstanceSetup: 1
2024/05/19 07:20:27 GCEInstanceSetup: 2
2024/05/19 07:20:27 GCEInstanceSetup: 3
2024/05/19 07:20:27 GCEInstanceSetup: 4
2024/05/19 07:20:27 GCEInstanceSetup: 5
2024/05/19 07:20:27 GCEInstanceSetup: 6
2024/05/19 07:20:27 GCEInstanceSetup: 7
2024/05/19 07:20:27 GCEInstanceSetup: 8
2024/05/19 07:20:27 GCEInstanceSetup: 9
2024/05/19 07:20:27 GCEInstanceSetup: 10
2024/05/19 07:20:27 GCEInstanceSetup: 11
2024/05/19 07:20:27 GCEInstanceSetup: 12
2024/05/19 07:20:27 GCEInstanceSetup: 13
2024/05/19 07:20:28 GCEInstanceSetup: 14
2024/05/19 07:20:28 GCEInstanceSetup: 15
2024/05/19 07:20:28 GCEInstanceSetup: No BYOL license detected. Proceeding with activation.
2024/05/19 07:20:28 GCEInstanceSetup: Checking instance license activation status.
2024/05/19 07:20:29 GCEInstanceSetup: instance-20240519-071626 needs to be activated by a KMS Server.
2024/05/19 07:20:30 GCEInstanceSetup: Key Management Service machine name set to kms.windows.googlecloud.com successfully.
2024/05/19 07:20:32 GCEInstanceSetup: Installed product key WX4NM-KYWYW-QJJR4-XV3QB-6VM33 successfully.
2024/05/19 07:20:32 GCEInstanceSetup: Activating instance...
2024/05/19 07:20:45 GCEInstanceSetup: Activating Windows(R), ServerDatacenter edition (ef6cfc9f-8c5d-44ac-9aad-de6a2ea0ae03) ...
2024/05/19 07:20:45 GCEInstanceSetup: Product activated successfully.
2024/05/19 07:20:47 GCEInstanceSetup: Activation successful.
2024/05/19 07:20:47 GCEInstanceSetup: Running 'schtasks' with arguments '/change /tn GCEStartup /enable'
2024/05/19 07:20:47 GCEInstanceSetup: --> SUCCESS: The parameters of scheduled task "GCEStartup" have been changed.
2024/05/19 07:20:47 GCEInstanceSetup: Running 'schtasks' with arguments '/run /tn GCEStartup'
2024/05/19 07:20:47 GCEInstanceSetup: --> SUCCESS: Attempted to run the scheduled task "GCEStartup".
2024/05/19 07:20:47 GCEInstanceSetup: ------------------------------------------------------------
2024/05/19 07:20:47 GCEInstanceSetup: Instance setup finished. instance-20240519-071626 is ready to use.
2024/05/19 07:20:47 GCEInstanceSetup: ------------------------------------------------------------
2024/05/19 07:20:49 C:\Program Files\Google\Compute Engine\metadata_scripts\GCEMetadataScripts.exe: Starting startup scripts (version dev).
2024/05/19 07:20:49 C:\Program Files\Google\Compute Engine\metadata_scripts\GCEMetadataScripts.exe: No startup scripts to run.


Specify --start=7850 in the next get-serial-port-output invocation to get only the new output starting from here.
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ 
```

### RDP into the Windows Server

1. To set a password for logging into the RDP, run the following command in Cloud Shell. Be sure you replace `[instance]` with the VM Instance that you created and set `[username]` as **admin**.

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ gcloud compute reset-windows-password instance-20240519-071626 --zone us-central1-c --user admin

This command creates an account and sets an initial password for the
user [admin] if the account does not already exist.
If the account already exists, resetting the password can cause the
LOSS OF ENCRYPTED DATA secured with the current password, including
files and stored passwords.

For more information, see:
https://cloud.google.com/compute/docs/operating-systems/windows#reset

Would you like to set or reset the password for [admin] (Y/n)?  y

Resetting and retrieving password for [admin] on [instance-20240519-071626]
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-b02e56e2f212/zones/us-central1-c/instances/instance-20240519-071626].
ip_address: 34.67.12.108
password:   (Jx>FvKon}18;R[
username:   admin
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ 
```

```sh
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ gcloud compute instances list
NAME: instance-20240519-071626
ZONE: us-central1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.0.2
EXTERNAL_IP: 34.67.12.108
STATUS: RUNNING
student_01_aa8f3a74b714@cloudshell:~ (qwiklabs-gcp-03-b02e56e2f212)$ 
```

Connect to your server. There are different ways to connect to your  server through RDP, depending on whether you are on Windows or not:

- If you are using a Chromebook or other machine at a Google Cloud  event there is likely an RDP app already installed on the computer.  Click the icon as below, if it is present, in the lower left corner of  the screen and enter the external IP of your VM

1. - If you are not on Windows but using Chrome, you can connect to your server through RDP directly from the browser using the [Spark View](https://chrome.google.com/webstore/detail/spark-view-faster-than-an/ddnnpdbioplhcagobicknkjkbhdefjkg?hl=en) extension. Click on **Add to Chrome**. Then, click **Launch app**.
2. Once launched, the **Spark View (RDP)** window opens. Use your Windows username **admin** and password you previously recorded in Step 2.
3. Add your VM instance's External IP as your Domain. Click Connect to confirm you want to connect.

If you are on a Macintosh, there are several freely accessible RDP Client packages available to install, such as [CoRD](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=0ahUKEwjthJbPzPDVAhVBZFAKHclnDRQQFgg_MAI&url=http%3A%2F%2Fcord.sourceforge.net%2F&usg=AFQjCNGyH4EJo932rqm3QgiuHfDRmQfFVA). After installing, connect as above to the External IP address of the  Windows server. Once it has connected, it will open up a login page  where you can specify Windows username **admin** and password from the output of above mentioned command to log in (ignore the Domain: field).

Once logged in, you should see the Windows desktop!
