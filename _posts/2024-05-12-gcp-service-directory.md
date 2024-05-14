---

layout: single
title:  "Service Directory: Qwik Start"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
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

# Service Directory: Qwik Start

This lab illustrates a scenario for the **Cisco SD-WAN Cloud Hub with Google Cloud** solution, the application-centric multi-cloud networking fabric  developed in partnership by Cisco and Google. In this scenario, a video  is streamed from an application hosted in Google Cloud across a Wide  Area Network (WAN).



The lab shows how to leverage Google Service Directory and Cisco SD-WAN  to optimize the performance of the video streaming application. You will learn how to use Google Service Directory to configure a *traffic  profile* associated with the video streaming application, and  (optionally) you will use Cisco SD-WAN vManage to better understand how  SD-WAN optimizes the applications associated with that *traffic  profile*.

- Create a Compute Engine instance that hosts a streaming video  service, using a pre-built Docker container, and connect it to an SD-WAN edge router
- Log on to a Windows client VM and use the VLC application to start streaming a video clip from the video service created above
- Set up bandwidth monitoring on the client VM in order to observe the traffic optimization offered by the Cloud Hub solution
- Associate  *traffic profile* service metadata to the video  streaming application via Service Directory, and observe how Cisco  SD-WAN optimizes in real time the quality of the received video clip
- (Optional) Explore the Cisco SD-WAN management web UI (Cisco vManage) to better understand what's happening behind the scenes

### Lab network setup

The lab creates five VMs when deployed, and the user will manually  create one more VM for a total of six. The network topology is shown in  the figure below:



The `streaming-video-vm`, as the name suggests, hosts a  streaming video service in the cloud, and will be created in the next  step. The VM is connected to the `service-network` VPC, which connects using a virtual edge router (`sdwan-vedge-streaming-service`) to the Cisco SD-WAN. This is a "site" from the SD-WAN perspective, with site ID 111. The virtual edge router has two uplinks: a lower cost `sdwan-public-internet` with "best effort" traffic characteristics and a premium `sdwan-biz-internet` connection that offers guaranteed high throughput.

While the whole lab is created on Google Cloud, we simulate an  enterprise with SD-WAN site ID 100 on the left side of the diagram. The  enterprise deploying the Cisco SD-WAN has its own edge router (`sdwan-vedge-client`), using the same connectivity. A Windows PC in the enterprise (`streaming-client`) connects to the edge router over the client-network.

Finally, the Cisco SD-WAN control plane is contained in the `sdwan-in-a-box` VM.

A monitoring Compute Engine VM **vm-monitor** is used for evaluation of the lab tasks.



## Creating the streaming video service

To demonstrate how Cisco SD-WAN configuration optimizes network  traffic, you will observe the streaming of a video over the network. A  pre-built container uses VLC to stream a ~15 minute variable bitrate  video clip, reaching an average data rate of ~7 Mbps.

## Creating the Service Directory entry for the streaming video service

[Service Directory](https://cloud.google.com/service-directory) is a single place to publish, discover and connect services. The Cloud  Hub solution uses Service Directory to publish metadata associated with  those cloud services that want to benefit from Cisco SD-WAN network  optimizations.

In a typical Cloud Hub workflow, two teams are collaborating to offer improved end-to-end application experience: the *NetOps* configure and maintain the SD-WAN, the *DevOps* deploy the applications. The two teams agree on a set of metadata (called *traffic profiles*) that reflects the network needs for the services that are labeled with  that specific traffic profile. The example used in this lab will show  that:

- NetOps and DevOps agree to use two *traffic profiles*: `standard` and `video`
- DevOps associate the *traffic profile* to the application(s)
- The *traffic profile* is added as service metadata for the video streaming application via Service Directory. Note that the the metadata key used is `traffic-profile,` while the metadata value is either `standard` or `video`
- NetOps create appropriate SD-WAN policies for each of the different profiles agreed upon
- `standard` traffic is transported  with a "best effort" policy
- `video` traffic is steered towards a high bandwidth link

1. In the Cloud Console, use the **Navigation Menu** to browse to **Network services** > **Service Directory**.

First, you may need to enable the Service Directory API.

1. Once the API is enabled, you will be able to click **Register Service**.
2. For **Service Type** choose Standard and click **Next**. Services are defined in namespaces, and namespaces are associated with regions.
3. Choose the region corresponding to the zone that was determined at the creation of the video streaming VM.
4. You can’t choose an existing **Namespace**, so click **Create Namespace** and enter `cloud-hub-lab` for the **Namespace name**.
5. Click **Create**
6. For the **Service name** use `streaming-video`.
7. Click **Add Annotation**, for the **Key** use `traffic-profile` and the **Value** use `standard`. You will be using the standard traffic profile here to make sure that  the traffic flows through the best effort link initially.
8. Click **Create**.
9. Finally, click on the newly created service and click **Add Endpoint**.
10. For the **Endpoint name** use `streaming-video-vm`, and use the IP (`10.111.1.111`) and port (`8080`) of the streaming video service.
11. Click **Create**.

**Note:**  The annotation is associated with the service, not the endpoints.

## Preparing the client VM

In this section, you will log into the Windows VM that is used for  streaming the video clip using the VLC media player and set up the  network bandwidth monitoring. You will use an RDP client to log into the Windows VM. If you want to RDP directly from the browser, you can use  the [Chrome RDP for Google Cloud](https://chrome.google.com/webstore/detail/chrome-rdp-for-google-clo/mpbbnannobiobpnfblimoapbephgifkm) extension, but if you are using a Windows machine, it is highly recommended to use [Microsoft Remote Desktop](https://www.microsoft.com/en-us/p/microsoft-remote-desktop/9wzdncrfj3ps), as other solutions are very slow for a video streaming media player application.

1. In the Cloud Console, navigate to the compute instances by going to **Compute Engine** > **VM instances**.
2. Click on the `streaming-client` Windows machine.
3. Choose **Set Windows Password**.
4. Leave the default user and click **Set**.
5. Copy the password and close the message.
6. Click **RDP** to connect with either the Chrome extension or Microsoft Remote Desktop.
7. Once you are logged in, click "Yes" in the Networks dialog box, and  close the "Server Manager" application that is automatically started.
8. Start monitoring the network throughput usage with Task Manager: Click **Start > Task Manager > More details > Performance > Ethernet**.
9. Locate the **VLC media player** icon on the desktop.  Double click on the VLC icon to get it started, accept the Privacy and  Network Access Policy, and then click once on the two arrows shaped as a circle to configure  the loop video playing feature.

If you've started the lab more than 15 minutes ago, the SD-WAN connectivity required for streaming the video should be working.

1. You can check for basic connectivity using the ping CLI command in PowerShell.
2. To start streaming the video from the service configured in the previous steps, choose: **Media > Open Network Stream**. For the URL, enter `http://10.111.1.111:8080` (not https!) and click **Play**.

You should now have the video playing in VLC with the occasional hiccup  (corrupted or dropped frames, the video freezing for short amounts of  time). In the Task Manager window monitoring the network performance,  the received throughput should be around 5 Mbps.



## Changing traffic profile annotations

It is now time to annotate your streaming application with the video *traffic profile*, which is more appropriate, and allows the Cisco SD-WAN to optimize accordingly.

1. In the Cloud Console, use the **Navigation Menu** to browse to **Network services** > **Service Directory**.
2. Click on the `cloud-hub-lab` namespace, then click on the `streaming-video` service.
3. Next to the Service Details, click **Edit**.
4. Replace the value `standard` with the value `video` for the `traffic-profile` metadata key.
5. Now, switch back to the Windows client VM and watch how after a few  seconds the video quality improves as the Ethernet throughput changes  from a flat ~5Mbps to a variable bitrate around 7.7Mbps!

## Optional) Exploring the Cisco SD-WAN user interface

1. To understand what's happening at the SD-WAN level, open the Cisco  vMange UI in a new browser tab. You can find the link in the Connection  Details panel, under the "Start/Stop Lab" button.
2. Log in with username `admin`, password `cloudHub-lab`.

After a successful login, you are greeted with the Viptela dashboard, showing the number of connected control and data plane elements, and  the health of the SD-WAN connectivity.

1. Click on the left-hand navigation menu and navigate to **Monitor > Network > sdwan-vedge-client**.
2. Once on the **sdwan-vedge-client** page, click **Interface**, then click on **Real Time** on the top right of the graph.

You can keep changing the metadata labels to observe how the traffic  is shifted from one interface to the other (corresponding to WAN links), depending on the traffic profile associated with the video streaming  service.
