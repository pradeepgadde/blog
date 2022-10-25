---
layout: single
title:  "FEC 129 Pseudowires"
date:   2022-10-25 02:50:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
header:
  overlay_image: /assets/images/networking-3.jpg
  og_image: /assets/images/networking-3.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# FEC 129 Pseudowires

To get the best of both worlds (BGP +LDP) and have features like Auto discovery

It combines BGP and LDP
BGP for autodiscovery
LDP for signalling 

Defined in RFC 4447

FEC 128: PWid FEC Element, contains Virtual Circuit ID, requires explicit configuration of the remote PE; Does not understand Multi-tenancy;

FEC 129: The Generalized PWid FEC Element, works more like L2VPN
Source/Target Attachment Individual Identifier (SAII/TAII)
like source/remote Site ID of L2VPN
Attachment Group Identifier (AGI) is a general VPN Identifier, to denote customer/tenant

FEC 129 is advantageous in inter-AS VPNs


## Configure FEC 129

```sh
set protocols bgp group <group-name> type internal
set protocols bgp group <group-name> local-address Y.Y.Y.Y
set protocols bgp group <group-name> family l2vpn auto-discovery-only
set protocols bgp group <group-name> neighbor X.X.X.X (Route Reflector IP)
```

Address families configured `l2vpn-auto-discovery-only` in `show bgp neighbor` output

Under routing-instance (l2vpn)

we need to specify `l2vpn-id` with a value like `l2vpn-id:64512:456` (AGI)

and within the site, `source-attachment-identifier` (SAII) and `target-attachment-identifier` (TAII) needs to be specified.

in `bgp.l2vpn.0`table we find entries in this format : route distinguisher + SAII 

`192.168.1.1:123:0.0.0.1/96`

`show ldp database` show FEC129 entries.



`show l2vpn connections`



