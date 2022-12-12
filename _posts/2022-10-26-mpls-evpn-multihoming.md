---
layout: single
title: "EVPN Multi-homing Config"
date:   2022-10-26 03:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  overlay_image: /assets/images/networking-2.jpg
  og_image: /assets/images/networking-2.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# EVPN Multi-homing

Type 1 and Type 4 EVPN routes facilitate EVPN multi-homing.

With EVPN-MPLS, whether we have Multicast traffic or not, Type 3 routes will be present.
With EVPN-VXLAN, Type3 will be present only fi Multicast (IGMP) traffic is in the network.


If all are single-homed, there is no need for Type 4.
Type 4 is required only for multi-homing scenarios.

But if we use distributed IRB Gateway, even in EVPN-MPLS, Type 4 routes will be present (because of the virtual address).
