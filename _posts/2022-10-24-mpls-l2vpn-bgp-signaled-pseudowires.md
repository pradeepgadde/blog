---
layout: single
title:  "BGP Signaled Pseudowires"
date:   2022-10-24 01:30:04 +0530
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


# MPLS L2VPN BGP Signaled Pseudowires

- P, PE, and CE Routers
- Attachment Circuits
- Ethernet and Ethernet-VLAN Encapsulation
- Kompella Circuits
- Route Targets
- Route Distinguishers
- Site IDs
- Label Block

Attachment circuit is the logical connection between a CE and a PE.

With VLAN Tags, there can be many attachment circuits on a single wire.



Encapsulation type for CE-facing interface : Ethernet, Ethernet-VLAN

Every BGP VPN Route advertisement consists of two parts

- NLRI Network Layer Reachability Information 

- BGP Attributes

L2VPN Address Familty (**AFI 25 , SAFI 65)**

AFI: Layer-2 VPN (25)

SAFI: VPLS (65)

Remote PEs are auto discovered, no need of explicit configuration 

If an attachment circuit is migrated to a different PE, the other PE will autodiscover the change.

  

Route Target : imports/exports advertisements into/out of VPN

Site ID: unique identifier for each end of the L2VPN, used to calculate VPN Label

Label Block: advertises a range of labels for multiple attachment circuits at the same site; calculated using Label Base, Size, Offset, and the Site ID



Route Distinguisher Formats:

- Type 0 (2B+4B+NLRI) Uses 2B ASN

64512:1234567:10.10.10.0/24

- **Type 1** (4B+2B+NLRI) Uses IP, Recommended by RFC

192.168.1.1:12345:10.10.10.0/24

- Type 2 (4B+2B+NLRI) (Uses 4B ASN)

4200000000:12345:10.10.10.0/24

Type 1 enables faster failover.



Pre-requisites for creating an L2VPN

- IS-IS or OSPF in the core
- MPLs LSPs between PEs (LDP or RSVP) (populates `inet.3` table)
- BGP between PEs (via route reflector) with `family l2vpn signalling`

Idle, Connect, Active, OpenSent, OpenConfirmed,Established 

Route Target, Site ID, Label Base/Size/Offset, Route Distinguisher, VPN Label, Encapsulation

For Standard BGP, IPv4

RIB-IN --> RIB-Local ---> RIB-OUT

RIB-Local:  inet.0

With L2VPNs (AFI/SAFI  25/65), we create a routing instance 
`bgp.l2vpn.0` ---> `<instance-name>.l2vpn.0` --->

Transport label is resolved in `mpls.0` table 



Layer 2 Information Community 

There is another table named `<instance-name>.l2id.4`.
