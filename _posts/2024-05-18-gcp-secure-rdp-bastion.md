---

layout: single
title:  "Configure Secure RDP using a Windows Bastion Host"
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
# Configure Secure RDP using a Windows Bastion Host

## Challenge scenario

Your company has decided to deploy new application services in the cloud and your assignment is developing a secure framework for managing the Windows services that will be deployed. You will need to create a new VPC network environment for the secure production Windows servers.

Production servers must initially be completely isolated from external networks and cannot be directly accessed from, or be able to connect directly to, the internet. In order to configure and manage your first server in this environment, you will also need to deploy a bastion host, or jump box, that can be accessed from the internet using the Microsoft Remote Desktop Protocol (RDP). The bastion host should only be accessible via RDP from the internet, and should only be able to communicate with the other compute instances inside the VPC network using RDP.

Your company also has a monitoring system running from the default VPC network, so all compute instances must have a second network interface with an internal only connection to the default VPC network.

## Your challenge

Deploy the secure Windows machine that is not configured for external communication inside a new VPC subnet, then deploy the Microsoft Internet Information Server on that secure machine. For the purposes of this lab, all resources should be provisioned in the following region and zone:

Tasks

The key tasks are listed below. 

Create a new VPC network with a single subnet.
Create a firewall rule that allows external RDP traffic to the bastion host system.
Deploy two Windows servers that are connected to both the VPC network and the default network.
Create a virtual machine that points to the startup script.
Configure a firewall rule to allow HTTP access to the virtual machine.


```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-137f13e5d2c6.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute reset-windows-password vm-bastionhost --user app_admin --zone us-east4-b

This command creates an account and sets an initial password for the
user [app_admin] if the account does not already exist.
If the account already exists, resetting the password can cause the
LOSS OF ENCRYPTED DATA secured with the current password, including
files and stored passwords.

For more information, see:
https://cloud.google.com/compute/docs/operating-systems/windows#reset

Would you like to set or reset the password for [app_admin] (Y/n)?  y

Resetting and retrieving password for [app_admin] on [vm-bastionhost]
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-137f13e5d2c6/zones/us-east4-b/instances/vm-bastionhost].
ip_address: 34.86.17.221
password:   3#u~Q{no7(b\{Cx
username:   app_admin
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute reset-windows-password vm-securehost --user app_admin --zone us-east4-b

This command creates an account and sets an initial password for the
user [app_admin] if the account does not already exist.
If the account already exists, resetting the password can cause the
LOSS OF ENCRYPTED DATA secured with the current password, including
files and stored passwords.

For more information, see:
https://cloud.google.com/compute/docs/operating-systems/windows#reset

Would you like to set or reset the password for [app_admin] (Y/n)?  y

Resetting and retrieving password for [app_admin] on [vm-securehost]
Updated [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-03-137f13e5d2c6/zones/us-east4-b/instances/vm-securehost].
WARNING: Instance [vm-securehost] does not appear to have an external IP
address, so it will not be able to accept external connections.
To add an external IP address to the instance, use
gcloud compute instances add-access-config.
password: ,=O}4=h=(QIiKd{
username: app_admin
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute firewall-rules list
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

NAME: secure-rdp
NETWORK: securenetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute instances list
NAME: lab-monitor
ZONE: us-east4-b
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.150.0.2
EXTERNAL_IP: 34.86.237.223
STATUS: RUNNING

NAME: vm-bastionhost
ZONE: us-east4-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.1.3,10.150.0.4
EXTERNAL_IP: 34.86.17.221
STATUS: RUNNING

NAME: vm-securehost
ZONE: us-east4-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.1.2,10.150.0.3
EXTERNAL_IP: 
STATUS: RUNNING
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute instance list
ERROR: (gcloud.compute) Invalid choice: 'instance'.
Maybe you meant:
  gcloud compute instances list
  gcloud compute instance-groups list-instances
  gcloud compute instance-groups list
  gcloud compute instance-groups managed instance-configs list
  gcloud compute instance-templates list
  gcloud compute target-instances list
  gcloud compute instances add-access-config
  gcloud compute instances add-iam-policy-binding
  gcloud compute instances add-labels
  gcloud compute instances add-metadata

To search the help text of gcloud commands, run:
  gcloud help -- SEARCH_TERMS
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute instances list
NAME: lab-monitor
ZONE: us-east4-b
MACHINE_TYPE: e2-small
PREEMPTIBLE: 
INTERNAL_IP: 10.150.0.2
EXTERNAL_IP: 34.86.237.223
STATUS: RUNNING

NAME: vm-bastionhost
ZONE: us-east4-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.1.3,10.150.0.4
EXTERNAL_IP: 34.86.17.221
STATUS: RUNNING

NAME: vm-securehost
ZONE: us-east4-b
MACHINE_TYPE: e2-medium
PREEMPTIBLE: 
INTERNAL_IP: 10.128.1.2,10.150.0.3
EXTERNAL_IP: 
STATUS: RUNNING
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute networks list
NAME: default
SUBNET_MODE: AUTO
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

NAME: securenetwork
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 
student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ gcloud compute firewall-rules list
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

NAME: secure-http
NETWORK: securenetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False

NAME: secure-rdp
NETWORK: securenetwork
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: secure-rdp-default
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_04_42ac8558a1e8@cloudshell:~ (qwiklabs-gcp-03-137f13e5d2c6)$ 
```