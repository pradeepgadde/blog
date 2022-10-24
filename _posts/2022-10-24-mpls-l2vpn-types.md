---
layout: single
title:  "MPLS L2VPN Types"
date:   2022-10-24 01:20:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
header:
  overlay_image: /assets/images/networking-getty-4.jpg
  og_image: /assets/images/networking-getty-4.jpg
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---


# MPLS L2VPN Types

- Pseudowires
- VPLS
- EVPN

There are differences in the way MAC addresses are learnt.

In VPLS, MAC addresses are learnt in the data plane.

In EVPN, MAC addresses are learnt in the control plane (using BGP). Less resources used, more effective than VPLS


Service Provider acts as a

-  virtual wire 
- virtual switch 

There are multiple ways of creating both styles of Layer 2 VPN.


To generate Transport Label, IGP + BGP + LDP/RSVP

To generate a VPN Label (customer specific), another signalling protocol required 

- BGP
- LDP



## Pseduowires

Psuedowires: Point to Point Layer 2 VPN across a Service Provider network

Attachment Circuits (customer connections at either end)

One Pseudowire for each VLAN or all VLANs in a single pseudowire.

No MAC address learning needed over a single wire (on the PE devices). CE devices use ARP for learning the remote MACs.

PE devices simply forward with out any changes to the frame

Pseduowire use case in a Wholesaler Network

Data Center Interconnect - Pseudowires can be used for DCI

When a customer needs two sites to appear as one!

Two options to signal a Pseudowire:

- BGP

- LDP

  Both options are equally popular and commonly used.

Labelstack of a Pseudowire

Original L2-L7 encapsulated by Control Word + VPN Label + Transport Label + Service Provider Ethernet Header

What is **Control Word**?

it is like CRC or flow control. Was required when other L2 technologies were used like ATM, Frame Relay (instead of Ethernet)

Control Word is optional.

SP Ethernet Header + Transport Label + VPN Label + Control Word + Original L2-L7 frame



BGP-signalled pseudowires - Kompella circuits (Kireeti Kompella) (l2vpn)

LDP-signalled pseudowire - Martini circuits (Luca Martini) (l2circuit)

Other alternative terms 

VPWS - Virtual Private Wire Service 

Metro Ethernet Forum (MEF)

- E-Line  single point to point pseudo wire

- E-Tree multipoint pseudowires, point to multipoint Hub and Spoke


### Circuit Cross Connect (CCC) 
Third method of signalling pseudowires, to generate VPN label
Created by Juniper before Kompella/Martini circuits
Considered legacy nowadays, not recommended 

One Pseudowire is mapped to two unique **RSVP** LSPs, one in each direction 

CCC can also be used for encapsulation, not just for signalling! Be aware of the context!



With VPLS, PE devices start learning the MAC addresses, though we are building the pseudowires with VPLS.



L2VPN, L2Circuit, VPLS : all three are pseudowires





With first two options, PE does not learn MAC address

With VPLS, we will have an Instance and a table is maintained.



## VPLS

SP acts as a distributed switch, a series of pseudowires (BGP/LDP signalled)

PE router learns MAC addresses

Unknown MACs are flooded

IRB can be placed inside a VPLS as the gateway out of the subnet, to connect to internet (a new service). 

We can use BGP, LDP, or FEC 129 to create psuedowires for a VPLS.



In VPLS, customer routers form adjacencies directly. Service provider does not learn customer routes.

PE only learns the MAC addresses of the CE's WAN facing ports

Use cases for VPLS - Wholesale, Data Center, Distributed Firewalls

MAC addresses are learned in the data plane, by flood and learn

VRRP is required for redundancy (traffic might have to traverse the entire VPLS domain just to ge to the active IRB)

Multihomed linked can not run in active/active  (only one attachment circuit can be live, others must be shut down)

## EVPN

EVPN also offer a distributed switch 

MACs are now advertised in BGP

Old MACs can be withdrawn

MAC moves are advertised immediately

Multihomed links can be active/active. No need for Spanning Tree.

Remote PEs can automatically load balance

All PEs can host an active IRB (no need of VRRP, no need to travel across SP network, just to break out of the subnet)

Use cases: same as VPLS, but more popular in DC.



We can run all these VPN types on one physical customer-facing interface using `encapsulation flexible-ethernet-services` statement in Junos.

Each logical unit on a single physical interface can have different services, for example:

- Regular IPv4
- BGP signalled VPLS
- Untagged Layer 2
- VLAN tagged Layer 2
- BGP signalled Layer 2 VPN
- LDP signalled Layer 2 Circuit
- LDP singalled VPLS

