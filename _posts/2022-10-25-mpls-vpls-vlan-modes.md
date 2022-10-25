---
layout: single
title: "VPLS VLAN Modes"
date:   2022-10-25 03:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  overlay_image: /assets/images/networking-4.jpg
  og_image: /assets/images/networking-4.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# VPLS VLAN Modes

- default VLAN mode
- VLAN-Aware mode
- VLAN-Normalizing mode
- No-VLAN mode

VPLS, VLANs and Broadcast Domains



# Default Mode

In default VLAN mode, if BUM traffic is sent, flooding happens, across all VLANs , also towards remote PEs.

By default, a VPLS does not care/know about VLAN tags

`show vpls mac-table instance <name>`output shows `VLAN : NA`

>  Default VLAN mode floods all traffic out all interfaces, with sender's VLAN tag intact.



## VLAN-Aware Mode

```sh
set routing-instances <name> vlan-id all
```

Now, Each bridge domain is aware of the VLAN associated For example, with two VLANs, two bridge domains 

`VLAN : 100` and `VLAN : 200`

BUM traffic will be restricted to the VLAN only;

## VLAN-Normalizing Mode

Normalize VLANs in and out

```sh
set routing-instances <name> vlan-id X
```

all traffic is normalized to this single VLAN.

There is one single bridge domain `VLAN : X`

## No-VLAN Mode

It is same as VLAN-normalizing mode, except that the VLAN is removed 

```sh
set routing-instances <name> vlan-id none
```

VLAN is popped on input, pushed on output.

VPLS MAC table shows a single bridge domain, with `VLAN : none`

## Dual-stacked VLAN Tags

C-Tag inner tag (customer)

S-Tag outer tag (Service provider)

`vlan-tags outer X inner Y` under interfaces 

Dual-stacked VLANs in VPLS - behaviour in each of the VLAN modes

