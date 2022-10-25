---
layout: single
title: "VPLS Introduction"
date:   2022-10-25 02:55:04 +0530
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

# VPLS

VPWS - Virtual Private Wire Service 

L2VPN and L2Circuit falls under VPWS

VPLS: Virtual Private LAN Service

VPLS can be created using the same methods: 

- BGP
- LDP
- FEC 129

3 methods of signaling a VPLS



## BGP Signaled VPLS

`extended-vlan-vpls`

`family vpls`

`instance-type vpls`

```sh
set routing-instances <name> instance-type vpls
set routing-instances <name> interface <name>
set routing-instances <name> route-distinguosher 192.168.1.1:12345
set routing-instances <name> vrf-target target:64512:12345
set routing-instances <name> protocols vpls no-tunnel-services
set routing-instances <name> protocols vpls site <name> site-identifier X
set routing-instances <name> protocols vpls site <name> interface <name>
```

Depending on whether Tunnerl Services PIC hardware is present or not 
Junos created virtual interfaces to represent each pseudowire

`lsi` interfaces  Label Switched Interfaces 
`vt` Virtual Tunnel Interface 

Use `show vpls connections` to verify 

`Route Distinguisher : Remote Site: Offset`

`show vpls mac-table instance <instance-name>`

MAC addresses are learned with pseudowires as next-hops.

VPN Label = Label base + (Remote-site-identifier – Offset)

```sh
set routing-instances <name> protocols vpls label-block-size X
```

where X= 2,4,8,16 

default value is 8.



AFI/SAFI : 25/65

Encapsulation : VPLS

## LDP-Signaled VPLS

Enable LDP on lo0 interface

configure a `vpls-id` under` protocols vpls`

neighbors are explicitly configured in the instance (no autodiscovery)

```sh
set routing-instances <name> instance-type vpls
set routing-instances <name> interface <name>
set routing-instances <name> protocols vpls no-tunnel-services
set routing-instances <name> protocols vpls vpls-id 12345
set routing-instances <name> protocols vpls neighbor X.X.X.X
set routing-instances <name> protocols vpls neighbor Y.Y.Y.Y
```

LDP L2Circuit advertisements are identical to LDP VPLS (same FEC type 128)

## FEC 129 VPLS

FEC 129 VPLS is very similar to FEC 129 pseudowires.

No need to explicitly configure SAII/ TAII

They are automatically generated (numbers are based on the local and remote PE IP addresses)

```sh
set protocols bgp group <group-name> family l2vpn auto-discovery-only
```

```sh
set routing-instances <name> instance-type vpls
set routing-instances <name> interface <name>
set routing-instances <name> route-distinguosher 192.168.1.1:12345
set routing-instances <name> l2vpn-id l2vpn-id:64512:12345
set routing-instances <name> vrf-target target:64512:12345
set routing-instances <name> protocols vpls no-tunnel-services
```

FEC 129 prefixes have this format: route distinguisher : Source ID (SAII)

SAII is the remote PE's IP address

