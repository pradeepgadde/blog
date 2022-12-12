---
layout: single
title: "MPLS L3 VPNs"
date:   2022-11-28 01:55:04 +0530
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

# L3VPN Intro

- CE

- PE

- P

  

  

  VRF tables for each customer

  Overlapping address spaces problem solved with route distinguishers

  PE1 uses a separate VRF for each but other PEs it would be a problem (they cant distinguish them as belonging to different customers)

  3 Types

- Type0: 2 byte Type + 2 byte AS + 4 byte number + Prefix

- Type1: 2 byte Type + 4 byte IP address (router ID)+ 2 byte number + Prefix (Recommended)

- Type2: 2 byte Type + 4 byte AS + 2 byte number + Prefix

Type1 RD enables faster failover when a CE is multihomed to multiple PEs

Ingress PE router adds RD

VPN-IPv4 (AFI:1 SAFI: 128) routes are exchanged between PE routers using BGP

VRF Label chosen by the advertising PE router

Egress PE router converts VPN-IPv4 routes into IPv4 routes before inserting into site's VRF table

By default, all routes associated with the same VRF interface can share common label

Inner (VPN) label to reach the advertised prefix; advertised by BGP

Outer (Transport) label to reach the PE router; Advertised by label distribution protocols like LDP,RSVP etc

 Each router installed in a VRF table can be advertised to the CE devices associated with that VRF table.



L3VPN configuration happens on the PR routers

- inet.0

- inet.3

- mpls.0

- bgp.l3vpn.0

- <vpn-name>.inet.0

  ```sh
  set protocols bgp group <group-name> family inet-vpn unicast
  ```

  

- VRF Instance 

- VRF interfaces

- RD

- RT

- PE-CE routing

- Automatic Route target (vrf-target)

- VRF import or export policies

- BGP extended communities

```sh
set routing-instances <instance-name> instance-type vrf
set routing-instances <instance-name> interface <interface-name>
set routing-instances <instance-name> route-distinguisher 192.168.1.1:1
set routing-instances <instance-name> vrf-target target:65501:123
```

`vrf-target` shortcut for automatically creating VRF import and export

`target` parameter specified route target community, it is required. 

we can make policies that match specific RTs

For more granular control over routes, use `vrf-import` `vrf-export`

RD can be assigned manually or dynamically for every configured VRF

```sh
set routing-options route-distinguisher-id 192.168.1.1
```

```sh
set policy-options community <community-name> members target:65501:456
```

`vrf-import` `vrf-export` affects only PE-PE routes

Similarly we can write policies for CE-PE route control as well.

Use `as-override` parameter when CE routers belong to the same AS

or `autonomous-system loops `

`remove-private` can also work if private AS numbers are in user

CE-PE IBGP :Independent domain 

```sh
set routing-instances <instance-name> routing-options autonomous-system 65551 independent-domain
```



customer's attributes are preserved when advertised to the remote CE router

Customers do not see the providers BGP attributes



Site of Origin: when CE router is dual homed and as-override is required

SoO prevents advertising routes back to the source

SoO is an extended community 

```sh
set policy-options community <community-name> members origin:192.168.1.1:101
```

