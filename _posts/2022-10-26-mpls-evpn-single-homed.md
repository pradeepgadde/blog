---
layout: single
title: "EVPN Single-homed Config"
date:   2022-10-26 02:55:04 +0530
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

# EVPN Single-home Config



RFC7432 defines two ways to host multiple VLANs in an EVI

- VLAN-aware bundle : one MAC-VRF which lists each VLAN separately; lengthy config; Most popular way; Each VLAN gets its own named bridge domain inside the routing instance. Instance-type virtual-switch;

- VLAN-bundle: one MAC-VRF that has no awareness of VLANs; easier to config; instance-type evpn;



## VLAN-based EVI

## VLAN-aware EVI

