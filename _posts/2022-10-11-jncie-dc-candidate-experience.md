---

layout: single
title:  "JNCIE-DC Candidate Experience"
date:   2022-10-11 17:59:04 +0530
categories: Networking
tags: JNCIE-DC
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# JNCIE-DC

Today, I have attended an open learning webcast session on JNCIE-DC Candidate Experience presented by 5XJNCIE, Stefan Fouant.

Here is the summary of my notes in preparation for the Data Center Expert certification exam.

Recommended Courses: Data Center Automation using Juniper Apstra and Self-Study Bundle

In the new version of the exam, we have multiple DCs- Some with Apstra and some without any controllers.

6 hour practical exam. 

JPR-981, live from July 2022.

- Layer 2 , Layer 3 Underlay (Apstra Managed and Controllerless Underlay)
- Overlay (Apstra managed EVPN/VXLAN Overlay, Controllerless EVPN/VXLAN Overlay)
- DCI (Apstra managed and Controllerless EVPN DCI)
- Security
- CoS
- Management

Security and CoS sections are relatively easy.

JNCIE-DC Self-Study bundle is available now, which includes 12 lab sessions, 2 practice exams.

Can I do it? Can I Pass the JNCIE? Yes, you can!! Just do it.

Well-baked solution from Apstra for DC management... multi-vendor support included.

No Virtual Chassis (VC) or VCF. MC-LAG/ESI might be there!

Underlay is IPv4 only, some IPv6 in the overlay.

For DCI, no L3VPN. Its just a generic hand-off to connect two DCs, by leaking routes.

## Layer 3 Underlay

- Controllerless - build, deploy, troubleshoot an IP Fabric, use all available links for forwarding, use routing policiees to ensure that only rrequired addresses are advertised
- Controller based using Apstra- Same as above + create and use **custom tags**, restrict all fabric device configuration to Apstra

## Overlay

- EBGP/IBGP signalled EVPN-VXLAN overlay, multi-homed/single-homed end devices

- create multiple separate L2/L3 tenants environments

- Enable routing between different L3 tenants

- configure and optimize multicast communication

## DCI

Using Type 2 or Type 5 EVPN routes to enable tenant communication between two DCs.

Without Apstra configlets

seamless VXLAN-to-VXLAN stitching 

Identify incoming VXLAN VNI information from an unmanaged pre-configured peer device (use traceoptions to determine VNI informaiton)

Is it ERB or CRB? 

## Security

vSRX monitor and log excessive traffic events

track STP (Spanning Tree Protocol) messages and take required actions

restrict device access from unauthorized protocols/networks

Protect the DC infra and edge devices

## CoS

 CoS settings for server traffic connected to leaf nodes

## Management 

Use tags and probes to track critical services and trigger Apstra anomalies when specific conditions are met or exceeded

Create and use custom Apstra configlets

Enable BFD between leaf and external devices

Implement NTP

Local and remote syslog

Enable streaming telemetry on custom ports

## Additional Resources - Day One Books

- DC Fundamantals 

- DC Deployment with EVPN/VXLAN

- QFX5100/ QFX10000 Series Books

## Exam Tools:

- SecureCRT and a Web GUI

- A network topology map

- Tables describing network addresses

- List of tasks and associated point values

- Access to technical documentation

## Exam Infrastructure:
A combination of vQFX, vSRX, and Apstra Devices

## Exam Prep -Lab Environment 

>  VXLAN to VXLAN stiching doesnt work in publicly release version of vQFX.

Virtual box homelab

EVE-NG

GNS3

Vmware

Junos Software Versions

vQFX 21.3

vSRX 20.2

vMX 20.4 (mostly not there in the exam setup)

Apstra 4.0.2

## Time-saving Tips

Apply multiple tasks to a device when appropriate

Cut and paste identical config pieces

Allow ~1hr for verification and troubleshooting after completing all tasks

Keyboard and CLI shortcuts (like Ctrl+W, Ctrl+X, Ctrl+A, Ctrl+E, Ctrl+K)

`show |display set`and copy/paste

`show` and `load merge terminal relative`

`show | compare` and `load patch terminal`

`replace pattern`

Know when to use notepad vs. the CLI

Troubleshoot outputs before inspecting the configuration (for example, OSPF not coming up because of MTU mismatch)

Verify that the issue is not hardware related 

Move on and revisit the issue later when possible

As a last resort, break the exam rules 





