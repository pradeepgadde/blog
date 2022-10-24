---
layout: single
title:  "BGP Signaled Pseudowires-L2VPN Configuration"
date:   2022-10-24 01:40:04 +0530
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



# L2VPN Configuration

BGP-Siganaled L2VPNs

- Accept all Ethernet traffic
- Accept specific VLAN tags

## Accept all Ethernet Traffic

1. Configure `family l2vpn signaling` under `[edit protocols bgp group <group-name>]`
2. Configure CE-Facing interface  on the PE with the encapsulation `ethernet-ccc`(for Ethernet mode) and `unit 0`; Encapsulation should match on both sides. `Ethernet raw mode (5)` can be seen in the packet capture.
3. Configure the routing instance on PE

```sh
root@PE> show configuration routing-instances my-customer-1
instance-type l2vpn;
interface ge-0/0/0.0;
route-distinguisher 192.168.1.1:100;
vrf-target target:64501:100;
protocols { 
    l2vpn {
         encapsulation-type ethernet;
         site customer-1-site-1 {
         				site-identifier 1;
         				interface ge-0/0/0.0;
         }
    }
}
```

`vrf-target` statement automatically generates import/export policies.

manual create and apply routing policies is possible using`vrf-import` and `vrf-export` statements.

4. Verify BGP is established  using`show bgp summary`

   `bgp.l2vpn.0` and `my-customer-1.l2vpn.0` tables are populated with correct entries.

5. Use `show route table bgp.l2vpn.0 detail | match [...]`

6. Verify routing instance table `show route table my-customer-1.l2vpn.0 detail | match [...]`

7. Verify that L2VPN is up using `show l2vpn connections instance my-customer-1`; Local site, connection-site, Status, remote PE, Incoming Label, Outgoing Label, Encapsulation etc can be seen in this command output.

8. Verify that the VPN lable is correct using `show route table mpls.0 label <label>` command; `[L2VPN/7]` can be seen for the entry.

9. Use the `show route table mpls.0 ccc ge-0/0/0.0 detail` command to verify customer facing itnerface ; We should see two labels pushed.

   

## Accept specific VLAN tags

On the CE-facing interface , two encapulations 

`extended-vlan-ccc` all plans from 1 and above on all interfaces 

and `vlan-ccc` vlans 512-4094 (legacy)... older M-series reserved vlans 1-511 for regular bridging.

Mx series can accept any valid VLAN.

```sh
root@PE> show configuration routing-instances my-customer-2
instance-type l2vpn;
interface ge-0/0/0.200;
route-distinguisher 192.168.1.2:200;
vrf-target target:64501:200;
protocols { 
    l2vpn {
         encapsulation-type ethernet-vlan;
         site customer-2-site-1 {
         				site-identifier 1;
         				interface ge-0/0/0.200;
         }
    }
}
```



EtherType/TPID:

Tag Protocol Identifier

- 0x8100 :VLAN tagged fram

- 0x9100: QinQ (Double vlan tagged frame)

Some vendors use proprietary 0X9901.



`ethernet-ccc`: any standard TPID/EtherType allowed

`vlan-ccc`: only 0x8100 allowed 

`extended-vlan-ccc`: only 0x8100,0x9100, 0x9901 allowed



## Common Errors

- `No connections found` 

- `EM -- encapsulation mismatch`

- `MM -- MTU Mismatch`

- `LD -- local site signaled down`

- `RD -- remote site signaled down`

- `OR -- Out of range` (Unexpected Site ID)

  