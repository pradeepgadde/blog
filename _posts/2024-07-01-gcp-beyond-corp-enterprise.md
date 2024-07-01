---

layout: single
title:  "Securing Virtual Machines using BeyondCorp Enterprise (BCE)"
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
# Securing Virtual Machines using BeyondCorp Enterprise (BCE)

In this lab, you will learn how to use BeyondCorp Enterprise (BCE) and Identity-Aware Proxy (IAP) TCP forwarding to enable administrative access to VM instances that do not have external IP addresses or do not permit direct access over the internet.

- Enable IAP TCP forwarding in your Google Cloud project
- Test connectivity to your Linux and Windows instances
- Configure the required firewall rules for BCE
- Grant permissions to use IAP TCP forwarding
- Demonstrate tunneling using SSH and RDP connections

## Enable IAP TCP forwarding in your Google Cloud project

1. Open the **Navigation Menu** and select **APIs and Services > Library.**
2. Search for **IAP** and select **Cloud Identity-Aware Proxy API.**
3. Click **Enable.**

## Create Linux and Windows instances

Create three instances for this lab: - Two for demonstration purposes (Linux and Windows) - One for testing connectivity (Windows)

### Linux instance

From the **Navigation Menu** and select **Compute Engine**.

Click **Create instance** and use the following configuration to create a VM. Leave the rest as default.

- Name: **linux-iap**

- Click on **Advanced options** and select **Networking**. Under **network interfaces** click the defult network to edit. Then change the External IPV4 address to **None**.

  Click **Done**.

  Then click **create**. This VM will be referred to as  **linux-iap**

### Windows instances

To create the **Windows Demo VM** click **Create instance** and use the following configuration to create a VM. Leave the rest as default.

- Name: **windows-iap**
- Under the **boot disk** section, click on **Change**

For the OS, select the following:

- Public Images **>** Operating system **>** Windows Server
- Version **>** Windows Server 2016 Datacenter

Click **Select**.

Click on **Advanced options** and select **Networking**. Under **network interfaces** click the defult network to edit. Then change the External IPV4 address to **None**. Click **Done**.

Then click **create**. This VM will be referred to as **windows-iap**

To create the **Windows Connectivity VM**, click **Create instance** and use the following configuration to create a VM. Leave the rest as default.

- Name: **windows-connectivity**
- Under the **boot disk** section, click on **Change**

For the OS, set the following on the **Custom Images** tab:

- Source project for images: `Qwiklabs Resources`
- Image: `iap-desktop-v001`

Click **Select**.

For access scopes, select **Allow full access to all Cloud APIs**

**Do not** disable external IP for this instance

Then click **create**. This VM will be referred to as **windows-connectivity**

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-cdc4601dc5cc.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ gcloud compute instances list
NAME: linux-iap
ZONE: europe-west1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.3
EXTERNAL_IP: 
STATUS: RUNNING

NAME: startup-vm
ZONE: europe-west1-c
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.2
EXTERNAL_IP: 34.22.212.159
STATUS: RUNNING

NAME: windows-connectivity
ZONE: europe-west1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.5
EXTERNAL_IP: 34.76.52.206
STATUS: RUNNING

NAME: windows-iap
ZONE: europe-west1-c
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.132.0.4
EXTERNAL_IP: 
STATUS: RUNNING
student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ 
```



## Test connectivity to your Linux and Windows instances

After the instances are created, You will test access to **linux-iap** and **windows-iap** to ensure that you aren’t able to access the VMs without the external IP.

For **linux-iap**  click on the SSH button to get into the machine and ensure you get a message similar to the following.

```sh
This instance does not have an external IP and you don't have the iap.tunnelInstances.accessViaIAP permission 
```

For **windows-iap:** click the RDP button and ensure you get a message similar to the following: `This instance does not have an external IP `.

The following steps for configuring and using IAP will allow you to connect to the instances that don’t have external IPs.



## Configure the required firewall rules for BCE

1. Open the **Navigation Menu** and select **VPC Network > Firewall** and click **Create Firewall Rule**
2. Configure the following settings:

| Field                | Setting                                                      |
| -------------------- | ------------------------------------------------------------ |
| Name                 | **allow-ingress-from-iap**                                   |
| Direction of traffic | **Ingress**                                                  |
| Target               | **All instances in the network**                             |
| Source filter        | **IPv4 Ranges**                                              |
| Source IPv4 Ranges   | **35.235.240.0/20**                                          |
| Protocols and ports  | **Select TCP and enter 22, 3389** to allow both SSH and RDP respectively |

Click **CREATE** to create the firewall rule.

```sh
student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ gcloud compute firewall-rules list
NAME: allow-ingress-from-iap
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22,tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY: 
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ 
```

```sh
student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ gcloud compute firewall-rules describe allow-ingress-from-iap
allowed:
- IPProtocol: tcp
  ports:
  - '22'
  - '3389'
creationTimestamp: '2024-07-01T09:35:27.052-07:00'
description: ''
direction: INGRESS
disabled: false
id: '2304975046657926176'
kind: compute#firewall
logConfig:
  enable: false
name: allow-ingress-from-iap
network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-cdc4601dc5cc/global/networks/default
priority: 1000
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-cdc4601dc5cc/global/firewalls/allow-ingress-from-iap
sourceRanges:
- 35.235.240.0/20
student_01_c9545bc0502a@cloudshell:~ (qwiklabs-gcp-01-cdc4601dc5cc)$ 
```



## Grant permissions to use IAP TCP forwarding

Use the following steps to configure the iap.tunnelResourceAccessor role by VM.

1. Open **Navigation Menu** and select **Security > Identity-Aware Proxy**, switch to the **SSH and TCP Resources** tab (safely ignore the Oauth Consent screen error in the HTTPS section).

2. Select the **linux-iap** and **windows-iap** VM instances.

3. Click **Add principal**, then enter in the service account associated with your **Windows connectivity** VM. This should be of the form 

4. Select **Cloud IAP > IAP-Secured Tunnel User** for the role.

   Click **SAVE**.

   From the top-right of the page click the "S" icon to open your profile and copy the email of the student account.

   Click **Add principal** again to add your student account.

   Enter in the **student account**. You can copy this value from the lab details pane.

   Select **Cloud IAP > IAP-Secured Tunnel User** for the role.

   Click **SAVE**.

   The IAP-Secured Tunnel User role will grant the windows-connectivity  instance to connect to resources using IAP. Adding the student account  will help verify the step was done correctly.

## Use IAP Desktop to connect to the Windows and Linux instances

It is possible to use IAP Desktop to connect to instances using a  graphical user interface from an instance with Windows Desktop. You can  read more about [IAP Desktop](https://github.com/GoogleCloudPlatform/iap-desktop) on the GitHub repository hosting the download for the tool.

To use IAP Desktop to connect to the instances in this lab:

1. RDP to the `windows-connectivity` instance by downloading the RDP file. Go to the **Compute Engine > VM Instances** page. Select the down arrow next to the **windows-connectivity** instance on the Compute Engine landing page and download the file.
2. Open the RDP file to connect to the instance via Remote Desktop  Protocol. You will use the credentials below to connect to the instance  once prompted:

- Username: student
- Password: Learn123!

1. Once connected to the **windows-connectivity** instance, locate and open the IAP Desktop application on the desktop of the instance.
2. Once the application opens, click on the sign in with Google button  to log in. Use the username and password provided in the lab console to  authenticate with **IAP Desktop**. Make sure you select the option to "See, edit, configure and delete Google Cloud data."

You will need to add the project to connect to Compute Engine instances  within IAP Desktop after authentication. Select the lab project  associated with your lab instance:

Double click on the **windows-iap** instance in the IAP Desktop application to log into the the instance.

1. You may be prompted to provide credentials for the instance the  first time you try to connect to it through IAP Desktop. Select  "Generate new credentials" the first time logging into the instance.
2. After the credentials are created you will be taken to the desktop of the `windows-iap` instance and can see the end user experience.

## Demonstrate tunneling using SSH and RDP connections

1. You will testconnectivity to the RDP instance using an RDP client.  This is because you need to connect to the instance via an IAP tunnel  locally.
2. Go to the **Compute Engine > VM Instances** page.
3. For the **windows-connectivity** instance click the down arrow and select **Set windows password**. Copy the password and save it.

Then click down arrow next to connect and click download the RDP  file. Open the RDP file with your client and enter in your password.

1. Once you have connected to the **windows-connectivity** instance. Open the **Google Cloud Shell SDK**:

```sh
Welcome to the Google Cloud SDK! Run "gcloud -h" to get the list of available commands.
---

C:\Program Files (x86)\Google\Cloud SDK>ipconfig

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.
   Link-local IPv6 Address . . . . . : fe80::f067:41e8:5f2f:df47%4
   IPv4 Address. . . . . . . . . . . : 10.132.0.5
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 10.132.0.1

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Connection-specific DNS Suffix  . :
   IPv6 Address. . . . . . . . . . . : 2001:0:2851:782c:14e6:1595:f57b:fffa
   Link-local IPv6 Address . . . . . : fe80::14e6:1595:f57b:fffa%2
   Default Gateway . . . . . . . . . : ::

Tunnel adapter isatap.europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.

C:\Program Files (x86)\Google\Cloud SDK>
```



1. Now from the command line enter the following command to see if you can connect to the **linux-iap instance**:

`gcloud compute ssh linux-iap`.



Click **Y** when promopted to continue and to select the zone.

Make sure that you select the right zone for the instance when prompted.

Then **Accept** the Putty security alert.

You should receive a message that no external IP address was found and that it will use IAP tunneling.

Update Putty Settings to allow Tunnel connections locally. Click the top left corner of the Putty Window > Change Settings.

Allow local ports to accept connections from other hosts by checking the checkbox "Local ports accept connections from other hosts".

Close the Putty session and click **Apply**. Use the following command to create an encrypted tunnel to the RDP port of the VM instance:



```sh
gcloud compute start-iap-tunnel windows-iap 3389 --local-host-port=localhost:0  --zone=europe-west1-c
```

```sh
Welcome to the Google Cloud SDK! Run "gcloud -h" to get the list of available commands.
---

C:\Program Files (x86)\Google\Cloud SDK>gcloud compute ssh linux-iap
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: The PuTTY PPK SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [C:\Users\student\.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Did you mean zone [europe-west1-c] for instance: [linux-iap] (Y/n)?  y

External IP address was not found; defaulting to using IAP tunneling.


Updates are available for some Google Cloud CLI components.  To install them,
please run:
  $ gcloud components update


C:\Program Files (x86)\Google\Cloud SDK>gcloud compute ssh linux-iap
Did you mean zone [europe-west1-c] for instance: [linux-iap] (Y/n)?  y

External IP address was not found; defaulting to using IAP tunneling.

C:\Program Files (x86)\Google\Cloud SDK>gcloud compute start-iap-tunnel windows-iap 3389 --local-host-port=localhost:0  --zone=europe-west1-c
Picking local unused port [49938].
WARNING:

To increase the performance of the tunnel, consider installing NumPy. For instructions,
please see https://cloud.google.com/iap/docs/using-tcp-forwarding#increasing_the_tcp_upload_bandwidth

Testing if tunnel connection works.
Listening on port [49938].

```

Once you see the message about “Listening on port [XXX].” Copy the tunnel port number.

1. Return to the Google Cloud Console and go to the **Compute Engine > VM Instances** page.

Set and copy the password for the **windows-iap** instance.

Return to the RDP session now.

Leave gcloud running and open the Microsoft Windows **Remote Desktop Connection** app.

Enter the tunnel endpoint where the endpoint is the tunnel port number from the earlier step like so:

- localhost:endpoint

- Click **Connect**.

  Then enter the previous credentials you copied earlier. You will be successfully RDPed into your instance now!

  If prompted click **Yes**.

You were able to access the instance even without an external IP address using IAP.

```sh
Welcome to the Google Cloud CLI! Run "gcloud -h" to get the list of available commands.
---

C:\Program Files (x86)\Google\Cloud SDK>ipconfig

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.
   Link-local IPv6 Address . . . . . : fe80::5046:346f:c1be:7af0%3
   IPv4 Address. . . . . . . . . . . : 10.132.0.4
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . : 10.132.0.1

Tunnel adapter isatap.europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : europe-west1-c.c.qwiklabs-gcp-01-cdc4601dc5cc.internal.

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Connection-specific DNS Suffix  . :
   IPv6 Address. . . . . . . . . . . : 2001:0:2851:782c:10c9:3345:f57b:fffb
   Link-local IPv6 Address . . . . . : fe80::10c9:3345:f57b:fffb%8
   Default Gateway . . . . . . . . . : ::

C:\Program Files (x86)\Google\Cloud SDK>
```



Congratulations! You were able to successfully connect to both instances using IAP.

You learned how to use BeyondCorp Enterprise (BCE) and Identity-Aware Proxy (IAP) TCP forwarding by deploying 2 VM's, `windows-iap` and `linux-iap`, without IP addresses and configuring an IAP tunnel which gave you access to both instances using a third VM, `windows-connectivity`.
