---
layout: single
title:  "L2Circuits Advanced Topics"
date:   2022-10-25 01:50:04 +0530
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

# L2Circuits: Advanced Topics

## VCCV 
Virtual Circuit Connectivity Verification

CC Type
CV Type



Control Channel (CC)

Connectivity Verification (Cv)

Uses BFD/ICMP Ping/LSP Ping

PEs can send these messages to detect problems with L2Circuits

```sh
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> oam ping-interval X
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> oam ping-multiplier X
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> oam bfd-liveness-detection minimum-interval X
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> oam bfd-liveness-detection multiplier X
```

Use `show bfd session` to verify 

VCCV Packet Capture shows `MPLS ECHO`

## L2Circuit Multihoming

Backup neighbor

```sh
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> backup-neighbor <Y.Y.Y.Y>
```

`BK -- Backup connection` in the `show l2circuit connections` output.

L2circuits prefer stability, so they do not revert to the primary pseudo wire. 

If you like to change the behaviour,

```sh
set protocols l2circuit neighbor <X.X.X.X> interface <ge-0/0/0.0> revert-time XX
```

where XX: 0-600 seconds

## L2Circuit Local Switching



Pseudowire between two sites on the same PE

in L2VPN, put both interfaces in the same routing instance

in L2Circuit, pick one starting interface and one end interface.



```sh
set protocols l2circuit local-switching interface <ge-0/0/0.0> end-interface interface <ge-0/0/1.0>
```

No need for a Virtual Circuit ID

## Interworking

Stitching Pseudowires together, useful during acquisitions

Even L2VPN and L2Circuit can be stitched

`iw0` interface 

Interworking interfaces

Create a pair of units to peer with on this `iw0` interface.

For eample, `iw0.0` and `iw0.1`

Protocols `l2iw` 

```sh
set interfaces iw0 unit 0 encapsulation vlan-ccc
set interfaces iw0 unit 0 vlan-id 100
set interfaces iw0 unit 0 peer-unit 1
set interfaces iw0 unit 1 encapsulation vlan-ccc
set interfaces iw0 unit 1 vlan-id 100
set interfaces iw0 unit 1 peer-unit 0
```
Turn on the protocol 
```sh
set protocols l2iw
```

Anchor the `iw` interfaces into relevant pseudowire config

BGP L2VPN Config
```sh
root@PE> show configuration routing-instances my-customer-1
instance-type l2vpn;
interface iw0.0;
route-distinguisher 192.168.1.1:100;
vrf-target target:64501:100;
protocols { 
    l2vpn {
         encapsulation-type ethernet-vlan;
         site customer-1-site-1 {
         				site-identifier 2;
         				interface iw0.0 {
         				  remote-site-id 1;
         				}
         }
    }
}
```

LDP L2Circuit
```sh
set protocols l2circuit neighbor X.X.X.X interface iw0.1 virtual-circuit-id 1
```
