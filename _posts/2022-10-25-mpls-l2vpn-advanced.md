---
layout: single
title: "L2VPN Advanced Concepts"
date:   2022-10-25 00:55:04 +0530
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

# L2VPN Advanced Concepts

With L2VPN Pseudowires, the PE routers are not aware of the VLAN used by the other end. 

In the BGP update message, there is no such field to carry the VLAN information. So, even if both PEs are using different VLANs, the Pseudowire stays up. 

BUM traffic will pass through the PEs, though the VLAN IDs are different.



With L2VPN, one bridge domain is not equal to one VLAN. L2VPN does not understand VLANs. From L2VPN point of view, VLAN number is not to restrict traffic flow to same VLAN.



With L2VPN, one bridge domain = one broadcast domain (irrespective of the VLAN number)



Topics to be familiar with:

- L2VPN Multihoming

- Martini Encapsulation

- VLAN normalization (vlan swapping)

- Traffic Policing, Out-of-band route reflection, route target constraint

  

From bgp.l2vpn.0 to `<instance-name>.l2vpn.0` , will the RD be there or stripped off?

Route Distinguisher (RD) is still maintained in the customer instance.

Instance specific routing tables 

L2VPN advertisements go through three stages of processing

`bgp.l2vpn.0` to `<instance-name>.l2vpn.0` to `<instance-name>.l2id.0`

RD will be there in first two tables, but will be removed in the 3rd table (`l2id.0`)

## Multihoming

Two ways

1. Multichassis LAG or similar
2. tag one site as `primary` and keep the `backup` circuit in a down site

```xml
root@Juniper#set routing-instances <name> protocols l2vpn site <name> site-preference <primary|backup>
```

On both PEs, same `site-identifier`is used, similarly same `remote-site-id`.

> Primary site uses BGP Local preference to 65535 and backup uses Local Preference of 1.



We will see `RN -- remote site not designated`  and `LN -- local site not designated` in the `show l2vpn connections` output.



## Martini Encapsulation

Martini Encapsulation is different from Martini Circuits (LDP based).

Martini Encapsulation describe encapsulating different Layer 2 protocols in MPLS.

`Control Word`

Defined in RFC 4448

32-bit header for the Control Word

FR/ATM makes great use of it but in Ethernet its mostly not required/used.

```sh
`MPLS Transport Label > MPLS VPN Label > Control Word > Customer Layer 2 Frame`
```

Ethernet Control word is mainly 0s, but still performs a vital function.

Junos load-balances based on hash of fields in layer 3 header.



First half set to all zeros. Second half is for sequence number.

4 bits + 12 bits + 16 bits



## Swapping VLAN Tags in L2VPN

Solution to mismatched VLANs

```xml
[edit interfaces <interface-name>]

unit <> {
  vlan-id X;
	input-vlan-map {
		swap;
		vlan-id Y;
}
  output-vlan-map swap;
}

```

Encapsulation `VLAN-CCC`

For example, `In (swap .100) Out (swap .200)`

## Traffic Policing

`root@PE#set interfaces <interface-name> unit <unit-no> family ccc policer input <policer-name>`

## Route Distinguisher ID

To automatically generate a unique RD for each VPN based on the PE's loopback IP

```sh
set routing-options route-distinguisher-id 192.168.1.1
```

## Route Reflectors

`bgp.l2vpn.0` 

For protocol next-hop resoltion, uses `inet.3` by default.



RRs usually dont participate in LSPs, so no inet.3 table.

We can tell RR to resolve L2VPN next-hops in `inet.0` instead of `inet.3`

by using 

```sh 
set routing-options resolution rib bgp.rtarget.0 resolution-ribs inet.0
```



## Route Target Filtering

BGP neighbors indicate route targets they are  interested in 

Neighbor routers only send advertisements for those route targets

Route Target BGP family

```sh
set protocols bgp group <group-name> family route-target
```

Add this family in addition to `family l2vpn signaling`

`bgp.rtarget.0` table gets populated ; This table will serve as export policy as well (to filter unnecessary routes)

`show route table bgp.rtarget.0`

`[RTarget/5]`
