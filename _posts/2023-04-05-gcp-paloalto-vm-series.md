---


layout: single
title:  "Scaling VM-Series to Secure Google Cloud Networks"
date:   2023-04-05 07:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Scaling VM-Series to Secure Google Cloud Networks

This lab shows how to deploy and scale Palo Alto Networks VM-Series Next Generation Firewall to secure a hub and spoke architecture in Google  Cloud.  The VM-Series enables enterprises to secure their applications,  users, and data deployed across Google Cloud and other virtualization  environments.

The VM-Series prevention capabilities include:

1. Prevent inbound threats from the internet to resources deployed in VPC networks, on-premises, and in other cloud environments.

2. Stop outbound connections from VPC networks to suspicious  destinations, such as command-and-control servers, or malicious code  repositories.

3. Prevent threats from moving laterally between workload VPC networks and stealing data.

4. Secure traffic between remote networks connected through Cloud Interconnect, Network Connectivity Center, or Cloud VPN.

5. Extend security to remote users and mobile devices to provide granular application access to Google Cloud resources.

   

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-601.png)



### Objectives

- Review hub-and-spoke network topology secured by VM-Series
- Access the VM-Series Firewall
- Safely enable applications with App-ID
- Autoscale the VM-Series



Topology

Below is a diagram of the lab environment.  VM-Series firewalls are  deployed within a regional managed instance group to secure north/south  and east/west traffic for two spoke VPC networks.

The VM-Series inspects traffic as follows:

1. Traffic from the internet to applications in the spoke networks are  distributed by the External TCP/UDP Load Balancer to the VM-Series  untrust interfaces (NIC0). The VM-Series inspects the traffic and  forwards permissible traffic through its trust interface (NIC2) to the  application in the spoke network.
2. Traffic from the spoke networks destined to the internet is routed to the Internal TCP/UDP Load Balancer in the hub VPC. The VM-Series  inspects the traffic and forwards permissible traffic through its  untrust interface (NIC0) to the internet.
3. Traffic between spoke networks is routed to the Internal TCP/UDP Load Balancer in the hub VPC. The VM-Series inspects and forwards the  traffic through the trust interface (NIC2) into the hub network which  routes permissible traffic to the destination spoke network.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-602.png)



## 1. Access the VM-Series firewall

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-03-a5776b855858.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_04_c5c738fca7c3@cloudshell:~ (qwiklabs-gcp-03-a5776b855858)$ gcloud compute instances list \
    --filter='tags.items=(vmseries-tutorial)' \
    --format='value(EXTERNAL_IP)'
34.30.173.115
student_04_c5c738fca7c3@cloudshell:~ (qwiklabs-gcp-03-a5776b855858)$
```

Welcome to PAN-OS 10.2!


With this release, Palo Alto Networks introduces new and enhanced cloud-delivered security services. In concert with our ML-Powered Next-Generation firewalls, these services maximize ROI and extend best-in-class security without requiring independent infrastructures. PAN-OS 10.2 leverages cloud compute for artificial intelligence (AI) and deep learning techniques to secure the modern enterprise with unmatched performance. Highlights include:

Advanced Threat Prevention—Detects and prevents the latest advanced threats from infiltrating your network by leveraging deep learning models trained on high fidelity threat intelligence gathered by Palo Alto Networks. This inline cloud-based threat detection and prevention engine defends your network from evasive and unknown command-and-control (C2) threats by inspecting all network traffic.

Advanced URL Filtering—Operates a new inline deep learning engine that analyzes suspicious web page content in real-time to protect users against zero-day web attacks. By employing cloud-based inline web page payload analysis, Advanced URL Filtering is capable of detecting and preventing advanced and targeted phishing attacks, and other web-based attacks that use advanced evasion techniques such as cloaking, multi-step attacks, CAPTCHA challenges, and previously unseen one-time-use URLs.

Advanced Routing Engine—Uses an industry-standard configuration methodology to reduce your learning curve. Create profile-based filtering lists and conditional route maps, all of which can be used across logical routers. These profiles provide finer granularity to filter routes for each dynamic routing protocol and improve route redistribution across multiple protocols. Enable advanced routing and configure at least one logical router to use the Advanced Routing Engine.

Simplified Software Upgrade—Validates software upgrades before you install them on your firewalls and Panorama management servers. Prior to downloading the target release, the appliance displays any required software, including intermediate software versions and content dependencies, which you can download along with the target release in one step. This accelerates the upgrade process and provides early systemic validation to improve confidence for the upgrade.

Panorama Automatic Content Push for VM-Series and CN-Series Firewalls—Eliminates the operational overhead required to regenerate the VM-Series and CN-Series firewall images with the latest content updates. Automatically pushes content updates when onboarding new VM-Series and CN-Series firewalls to the Panorama management server. When leveraging Auto Scale, enabling this feature maintains existing dynamic content (such as for policy rules using App-ID) in the image configurations.

CN-Series Firewall as a Kubernetes CNF—You can now deploy the CN-Series firewall as a Kubernetes Container Network Function (CNF). Deploying the CN-Series as a CNF provides additional insertion options and I/O support. Additionally, it allows for higher achievable throughput for application pods that require inspection and use multiple network interfaces when compared to K8s Service or Daemonset deployment options.

High Availability Support for CN-Series Firewall as a Kubernetes CNF—Deploy the CN-Series as a Kubernetes CNF in High Availability (HA) mode to support active/passive HA with session and configuration synchronization.

Simplified IoT Onboarding—Select a predefined Log Forwarding profile when onboarding IoT Security and apply it to multiple Security policy rules. This simplifies the previous onboarding process where you had to create a Log Forwarding profile and apply it individually to each Security policy rule.

For a complete description of the new features and instructions for how to use them, refer to the PAN-OS 10.2 New Features Guide. For associated software and content releases, changes in default behavior, and other release information, refer to the PAN-OS 10.2 Release Notes. 

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-603.png)



## 2. Secure internet inbound traffic

Internet traffic is distributed by an external TCP/UDP load balancer  to the VM-Series untrust interfaces. The VM-Series inspects and  translates the traffic to `VM A` in the `spoke 1` network. `VM A`  runs a generic web service and Jenkins.

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-510.png)

The request to the Jenkins server fails because the Jenkins application has not been enabled in the VM-Series security policies. Palo Alto Networks firewalls leverage App-ID™ to identify and enable applications with layer-7 controls.

Access the Jenkins service again. The Jenkins page resolves because you enabled the jenkins application within the VM-Series security policies.

Notice the jenkins application was denied before the jenkins application was added to the inbound-web security policy. This is because all Palo Alto Networks firewalls use multiple identification techniques to determine the exact identity of applications traversing your network, including those that try to evade detection by masquerading as legitimate traffic, by hopping ports or by using encryption.

Internet outbound and east/west traffic

The VM-Series secures outbound internet traffic from the spoke networks and east-west traffic traversing between spoke networks. All egress traffic from the spoke networks is routed to an internal TCP/UDP load balancer that distributes traffic to the VM-Series trust interfaces for inspection.

In Qwiklabs, copy and paste the SSH VM B output into Cloud Shell. This establishes a SSH session with VM B in the Spoke 2 network. The external load balancer distributes the request to the VM-Series. The VM-Series inspects and translates the traffic to VM B.

```sh
student_04_c5c738fca7c3@cloudshell:~ (qwiklabs-gcp-03-a5776b855858)$ ssh paloalto@34.30.172.4
The authenticity of host '34.30.172.4 (34.30.172.4)' can't be established.
ECDSA key fingerprint is SHA256:OxN39bUcD9c2IKjL0dPkh+w3h4TI8R38KcFBojICCIY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '34.30.172.4' (ECDSA) to the list of known hosts.
paloalto@34.30.172.4's password:
Permission denied, please try again.
paloalto@34.30.172.4's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-1018-gcp x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Apr  6 16:06:21 UTC 2023

  System load:  0.07              Processes:             99
  Usage of /:   20.7% of 9.52GB   Users logged in:       0
  Memory usage: 33%               IPv4 address for ens4: 10.2.0.10
  Swap usage:   0%


72 updates can be applied immediately.
32 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Nov 30 23:41:16 2021 from 74.97.22.10
paloalto@spoke2-vm1:~$
```
Attempt to download a pseudo malicious file from the internet. Note, the file is safe and is used for threat prevention testing only.

```sh
paloalto@spoke2-vm1:~$ wget www.eicar.org/download/eicar.com.txt
--2023-04-06 16:07:07--  http://www.eicar.org/download/eicar.com.txt
Resolving www.eicar.org (www.eicar.org)... 89.238.73.97, 2a00:1828:1000:2497::2
Connecting to www.eicar.org (www.eicar.org)|89.238.73.97|:80... connected.
HTTP request sent, awaiting response... 503 Service Unavailable
2023-04-06 16:07:08 ERROR 503: Service Unavailable.

paloalto@spoke2-vm1:~$
```

Generate pseudo malicious traffic between VM B and VM A:

```sh
paloalto@spoke2-vm1:~$ curl http://10.1.0.10/cgi-bin/../../../..//bin/cat%20/etc/passwd
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.1.0.10 Port 80</address>
</body></html>
paloalto@spoke2-vm1:~$ curl -H 'User-Agent: () { :; }; 123.123.123.123:9999' http://10.1.0.10/cgi-bin/test-critical
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.1.0.10 Port 80</address>
</body></html>
paloalto@spoke2-vm1:~$
```

On the VM-Series, go to Monitor > Threat to view the threat logs.


The firewall’s security policies enable you to allow or block traffic on your network based on the user, application, and device. When traffic matches the allow rule defined in the security policy, the security profiles that are attached to the rule provide further content inspection. Security profiles include:

    Antivirus
    Anti-Spyware
    Vulnerability Protection
    URL Filtering
    File Blocking
    WildFire Analysis


## 3. Autoscale the VM-Series

This lab uses a regional managed instance group to deploy and scale  VM-Series firewalls across zones within a region.  Autoscaling enables  you to scale the security protecting your cloud assets while providing  high availability through cross-zone redundancy.

### Viewing metrics in Cloud Monitoring

The VM-Series firewall can publish native PAN-OS metrics to Google  Cloud Monitoring.   Each metric can be set as an autoscaling parameter  within the managed instance group.  Custom PAN-OS metrics include:

- Dataplane CPU utilization
- Dataplane packet buffer utilization
- New connections per second
- Throughput (Kbps)
- Throughput (packets per second)
- Total number of active sessions
- Session utilization
- SSL forward proxy utilization

The lab creates a custom Cloud Monitoring dashboard that displays  several of the VM-Series metrics.  To view the dashboard, perform the  following:

In the Google Cloud console, select **Monitoring > Dashboards**. Select the dashboard named **VM-Series Metrics**.

The dashboard displays various PAN-OS metrics of the VM-Series firewall instance group.

These metrics can be used within the regional managed instance group to  scale the VM-Series firewalls.  For example, you can automatically add  VM-Series if `Dataplane CPU utilization` exceeds 90% for more than 5 minutes.

### Scaling the VM-Series

The managed instance group created within the lab sets the minimum and the maximum number of VM-Series replicas to `1`.  Here, you will modify the minimum and the maximum number of replicas to manually increase the number of running firewalls.

1. In Google Cloud, go to **Compute Engine > Instance Groups**.  Open the `vmseries` instance group.  Click **EDIT**.

1. Within the **Autoscaling** section set:

- **Minimum number of instances** to `2`.
- **Maximum number of instances** to `3`.

A new firewall should now be deployed.

Access the user interface of the new firewall. Copy the `NIC1/MGT` public IP attached to the new firewall and paste it into a web-browser `https://<EXTERNAL_IP>`.

On the scaled VM-Series, navigate to **Monitor > Traffic**.  The traffic logs should be populated demonstrating the scaled VM-Series is now processing traffic.



You have learned about the fundamental networking concepts that enable  you to deploy and scale Palo Alto Networks VM-Series next generation  firewall in Google Cloud.

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-8.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-9.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-10.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-11.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-12.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-13.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-14.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-15.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-16.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-17.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-18.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-19.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-20.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-21.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-22.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-23.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-24.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-25.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-26.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-27.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-28.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-29.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-30.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-31.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-32.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-33.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/pa-vm-34.png)





