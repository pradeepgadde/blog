---
layout: single
title: "VPLS Advanced Features"
date:   2022-10-26 00:55:04 +0530
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

# VPLS Advanced Features

- Protection and MAC Limiting in VPLS
- IRB Interfaces to VPLS instances
- Efficient Traffic Flooding

## Automatic Site IDs

```sh
set routing-instances <name> protocols vpls site <name> automatic-site-id
```

BGP VPLS advertisements are inspected to find the next available Site ID.

Approx 1 minute delay for VPLS to come up because of the negotiation of site IDs.

## VPLS Statistics

Use `show vpls statistics` to check Flooded packets, Current MAC count on each interface.

We can use these as a baseline for limiting MAC addresses and policing flood traffic.

## MAC Limiting

```sh
set routing-instances <name> protocols vpls mac-table-size 100
set routing-instances <name> protocols vpls interface-mac-limit 10
```

Optional `packet-action` of `drop` 

## Policing VPLS Flood Traffic

Create a policer and then create a Firewall Filter under `family vpls`  with `policer` action. Finally apply the filter at 

```sh
set routing-instances <name> forwarding-options family vpls flood input <filter-name>
```

## MAC Flapping

Could be a switching loop, misconfiguration

Solution is to shutdown the interface if a MAC flaps quickly

Define  `l2-learning global-mac-move` properites 

```sh
set protocols l2-learning global-mac-move threshold-time X
set protocols l2-learning global-mac-move threshold-count X
set protocols l2-learning global-mac-move statistical-approach-wait-time X
set protocols l2-learning global-mac-move interface-recovery-time X
set protocols l2-learning global-mac-move cooloff-time X
set protocols l2-learning global-mac-move virtual-mac XXXXXXX
```

Blocks of MACs to exclude are specified with `virtual-mac` option.

Apply

```sh
set routing-instances <name> protocols vpls enable-mac-move-action
```

## IRB Interface

Multiple PEs can be configured with IRBs and run VRRP to decide the active IRB

IRB interface included as part of `vpls` routing-instance.

## Multicast LSPs

```sh
set routing-instances <name> provider-tunnel rsvp-te label-swtiched-path-template default-template
```

Provider tunnel is a P2MP LSP for flood traffic 

Uses RSVP

## Troubleshooting VPLS

Use `show vpls connections instance <name>`

`NP -- interface hardware not present`

Check if you have a Tunnel Services PIC or not

`no-tunnel-services` if not present

a PE can host multiple sites in a BGP VPLS

Multiple sites, multiple pseudowires

```sh
set routing-instances <name> protocols vpls site <name_1> site-identifier 1
set routing-instances <name> protocols vpls site <name_1> interface <name_1>
outing-instances <name> protocols vpls site <name_2> site-identifier 2
set routing-instances <name> protocols vpls site <name_2> interface <name_2>
```

`LM -- local site ID not minimum designated`
