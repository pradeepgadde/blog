---
layout: single
title:  "LDP Signaled Pseudowires"
date:   2022-10-25 01:30:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
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


# MPLS L2Circuits: LDP Signaled Pseudowires

Also called Martini Circuits
Defined in RFC 4447

L2VPN and L2Circuit have a lot in common:

- Attachment circuits
- Customer facing interface config
- Same encapsulation
- Dataplane is identical (inner VPN Label, outer Transport label)
- Similar verification and troubleshooting steps
- Both use Martini Encapsulation



LSP creation is separate from the VPN Signalling.



## Targeted LDP

LDP between remote PEs (not directly connected)

using loopbacks.

Unlike L2VPN, we dont have auto discovery feature possible because of the use of Route Reflectors (RRs).

With L2Circuits, if a new PE comes up, it. requires manual config on all existing PEs (a disadvantage compared to L2VPN).

LDP Uses TCP/UDP 646

Config:

1. Enable LDP on the loopback `set protocols ldp interface lo0.0`
2. Configure L2Circuit (this triggers the creation of the targeted LDP session)

3. CE facing interface configu uses either `ethernet-ccc` or `vlan-ccc`

```sh
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> virtual-circuit-id X
set protocols l2circuit neighbor <X.X.Y.Y> interface <ge-0/0/1.0> virtual-circuit-id Y
```

Virtual Circuit ID must match on both sides, similar to site ID in L2VPNs.

Remote PE IP

Local interface 

To verify, use `show ldp session` or `show ldp neighbor` commands.

`show ldp database`

Label and Prefix are the two columns that we see in the LDP database.

Input Label Database and Output Label Database

`Prefix L2CKT CtrlWord ETHERNET VC 1234`

`Prefix  L2CKT CtrlWord VLAN VC 9876`

`show l2circuit connections`





## Packet capture of L2Circuit/LDP

**FEC 128**

Label Mapping Message in LDP 

FEC Elements

Element Type: PWid FEC Element (128)

PW ID: 1234  (Pseudowire ID)

Generic Label (VPN Label)



L2 Circuit does not offer over provisioning.

If we use LDP for transport, for establishing transport LSP, then the FEC advertised is `Prefix FEC (2)` 
