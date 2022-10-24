---

layout: single
title:  "MPLS Refresher"
date:   2022-10-24 01:10:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
header:
  overlay_image: /assets/images/networking-getty.jpg
  og_image: /assets/images/networking-getty.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# MPLS Refresher

- Difference between IPSec VPNs and MPLs VPNs
- Difference between L3 VPNs and L2 VPNs

Understand that MPLS (single label) is not the same as MPLS VPNs (multiple labels)!

Label Switched Path (LSP) is used as transport, hence the name Transport Label.

For providing services (Virtual Private Networks) to multiple customers, a stack of labels required.

1st label identifies the customer and then the second label which is used for transport.



In L2 VPNs, Each CE will be in the same subnet and will be sending L2 traffic. SP Core will act as a Switch (Bridge).

In L3VPNs, Each CE will be in a different subnet. SP core will act as a Router.



IPSec VPNs providing Confidentiality, Integrity, Authentication (Source)

3 Types of Authentication : Peer, User, Source 

It is the responsibility of customers to setup their IPSec VPNs, utilizes resources on CE routers, and lot of management overhead!



LSPs are created between router loopbacks.

Use the LSP to get to a next-hop advertised by BGP.

Traffic is isolated using Service labels or VPN Labels. This second label is used to identify different customers.



MPLS: IGP + BGP + LDP/RSVP 

MPLS VPN: All components from MPLS + Another Signalling protocol for advertising another Label for Customer VPN



Layer 3 VPNs: PE Router advertise VPN labels via BGP.

As the packet traverses the Service provider core, the packet header looks like this 

Ethernet Header > Transport Label > VPN Label > IP Header > TCP Header > Payload 

Layer 2 VPN:

SP Ethernet header > Transport Label > VPN Label > Customer Ethernet Header > IP Header > TCP Header > Payload 



Common considerations while choosing a VPN

- Topology

- Bandwidth and CoS

- Security

- Provisioning

- Build or Buy



