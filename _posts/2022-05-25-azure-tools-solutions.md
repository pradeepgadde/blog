---

layout: single
title:  "Azure IoT Services"
date:   2022-05-2507:59:04 +0530
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/microsoft-certified-fundamentals-badge.svg
author:
  name     : "Microsoft"
  avatar   : "/assets/images/azure.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

Core solutions encompass a wide array of tools and services from Microsoft Azure. In this post, let us look at some of these tools and services, starting with Azure IoT Services.

## Azure IoT services
IoT enables devices to gather and then relay information for data analysis. Smart devices are equipped with sensors that collect data.

By using Azure IoT services, devices that are equipped with these kinds of sensors and that can connect to the internet could send their sensor readings to a specific endpoint in Azure via a message. The message's data is then collected and aggregated, and it can be converted into reports and alerts. Alternately, all devices could be updated with new firmware to fix issues or add new functionality by sending software updates from Azure IoT services to each device.

### Azure IoT Hub
Azure IoT Hub is a managed service that's hosted in the cloud and that acts as a central message hub for bi-directional communication between your IoT application and the devices it manages.

From a cloud-to-device perspective, IoT Hub allows for command and control. That is, you can have either manual or automated remote control of connected devices, so you can instruct the device to open valves, set target temperatures, restart stuck devices, and so on.

### Azure IoT Central
Azure IoT Central builds on top of IoT Hub by adding a dashboard that allows you to connect, monitor, and manage your IoT devices. The visual user interface (UI) makes it easy to quickly connect new devices and watch as they begin sending telemetry or error messages. You can watch the overall performance across all devices in aggregate, and you can set up alerts that send notifications when a specific device needs maintenance. Finally, you can push firmware updates to the device.

### Azure Sphere
Azure Sphere creates an end-to-end, highly secure IoT solution for customers that encompasses everything from the hardware and operating system on the device to the secure method of sending messages from the device to the message hub. Azure Sphere has built-in communication and security features for internet-connected devices.

Azure Sphere comes in three parts:
- Azure Sphere micro-controller unit (MCU)
- Customized Linux operating system (OS)
- Azure Sphere Security Service, also known as AS3.

Following are some example scenarios where each of these three best fit:

A company wants to build a new voting kiosk for sales to governments around the world. Which IoT technologies should the company choose to ensure the highest degree of security?
Azure Sphere
Azure Sphere provides the highest degree of security to ensure the device has not been tampered with.

2. A company wants to quickly manage its individual IoT devices by using a web-based user interface. Which IoT technology should it choose?

IoT Central
IoT Central quickly creates a web-based management portal to enable reporting and communication with IoT devices.

3. You want to send messages from the IoT device to the cloud and vice versa. Which IoT technology can send and receive messages?

IoT Hub
An IoT hub communicates to IoT devices by sending and receiving messages.


![Azure IoT]({{ site.url }}{{ site.baseurl }}/assets/images/azure-iot.png)




