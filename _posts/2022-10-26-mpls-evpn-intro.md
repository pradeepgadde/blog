---
layout: single
title: "EVPN Introduction"
date:   2022-10-26 01:55:04 +0530
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

# EVPN Intro

L2VPN - BGP
VPLS - BGP
EVPN - BGP

For both AFI/SAFI : 25/65
For EVPN, same AFI 25 but SAFI 70 (25/70)



Topics to be familiar with:
- Advantages of EVPN over VPLS
- Purpose of EVPN Route Types 

<span style="color:red"> VPLS Problems:</span>

- Multihoming causes loops 

- Only one PE can host the active IRB

- MAC addresses learned in data plane

- ARP requests flooded across service provider core



From customer point of view, VPLS and EVPN are identical but behind the scenes, everything is BGP, even MAC advertisements.

<span style="color:green">Advantages of EVPN:</span>

- Address Learning (using BGP)
- Redundancy (Ethernet Segment Identifier - ESI)
- Local Breakout (multiple PEs can host an active IRB)
- Data plane options (MPLS, VXLAN)

<span style="color:green">Advantages of BGP MAC Learning:</span>

- When a PE learns a local MAC address, it can immediately advertise to remote PEs
- Proxy ARP (local PE can answer local ARP requests for remote MAC addresses)
- 



<span style="color:blue">**EVPN Terminology:**</span>

- EVPN Instance (EVI)
- MAC-VRF (virtual routing and forwarding)
- Ethernet Segment (ES)
- Ethernet Segment Identifier (ESI) (10-octet number, non-zero)
- Ethernet Tag ID (32 bit number, identifying the broadcast domain)
- VID 



In QFX switches, by default,

`bgp.evpn.0`

`default-switch.evpn.0` EVI 

MAC addresses placed in corresponding bridge domain

Multiple bridge domains

It is possible to create custom EVIs and separate MAC addresses by VLANs in multiple ways.

MAC-VRF



<span style="color:brown">**EVPN Route Types**</span>

8 route types in total, but for EVPN-MPLS, we mostly use four of them.

- Type 1 Ethernet Auto-Discovery Route (sends ESI number, Mass MAC withdrawal, protection against L2 loops, load balancing through aliasing)

- Type 2 MAC/IP Advertisement Route (MAC or MAC/IP binding)

- Type 3 Inclusive Multicast Ethernet Tag (how a PE wants to receive BUM traffic) Tunnel Type: Ingress Replication , PMSI

- Type 4 Ethernet Segment Route (Designated Forwarder Election)

In Type 2, NLRI + Attributes

In Type3, NLRI + PMSI + Attributes

Provider Multicast Service Interface (PMSI)

Ingress Replication or Underlay Replication  ; Junos supports only Ingress replication.

