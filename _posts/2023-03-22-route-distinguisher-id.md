---
layout: single
title: "Route Distinguisher ID"
date:   2023-03-22 01:55:04 +0530
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

# Route Distinguisher ID

Each routing instance must have a  unique route distinguisher (RD) associated with it. The RD places bounds around a VPN so the device can use the same IP address prefixes  in different VPNs without having the addresses overlap.

 There are  3 Types of RDs

- Type 0: 2 byte Type + 2 byte AS + 4 byte number + Prefix

- Type 1: 2 byte Type + 4 byte IP address (router ID)+ 2 byte number + Prefix (Recommended)

- Type 2: 2 byte Type + 4 byte AS + 2 byte number + Prefix

Type1 RD enables faster failover when a CE is multihomed to multiple PEs

Ingress PE router adds RD.

```sh
root@R6> show configuration routing-instances 
CUST-A {
    instance-type vrf;
    routing-options {
        static {
            route 192.168.200.0/24 next-hop 172.16.3.103;
        }
    }
    interface ge-0/0/3.0;
    route-distinguisher 192.168.1.6:111;
    vrf-target target:65412:111;
    vrf-table-label;
}

root@R6> configure 
Entering configuration mode

[edit]
root@R6# delete routing-instances CUST-A route-distinguisher 

[edit]
root@R6# commit check 
[edit routing-instances]
  'CUST-A'
    RT Instance: Route-distinguisher must be configured for vrf instance: CUST-A
error: configuration check-out failed

[edit]
root@R6# 
```

```sh
root@R6> show route advertising-protocol bgp 192.168.1.5 detail 

CUST-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
* 172.16.3.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 192.168.1.6:111
     VPN Label: 16
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

* 192.168.200.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 192.168.1.6:111
     VPN Label: 16
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

root@R6>
```
As we see here, RD is mandatory. But instead of configuring RD for each instance manually, we can auto generate these using the `route-distinguisher-id` statement.

```sh
[edit]
root@R6# set routing-options route-distinguisher-id 192.168.1.6 

[edit]
root@R6# show | compare 
[edit routing-instances CUST-A]
-    route-distinguisher 192.168.1.6:111;
[edit routing-options]
+  route-distinguisher-id 192.168.1.6;

[edit]
root@R6# commit 
commit complete

[edit]
root@R6# 
```



```sh
root@R6> show route advertising-protocol bgp 192.168.1.5 detail    

CUST-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
* 172.16.3.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 192.168.1.6:9
     VPN Label: 17
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

* 192.168.200.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 192.168.1.6:9
     VPN Label: 17
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

root@R6> 
```

We can see a route-distinguisher (`192.168.1.6:9`) is generated automatically.

What if both options are configured? If both options are present, the value configured for `route-distinguisher`  takes preference over the value generated from `route-distinguisher-id`. 



```sh
root@R6> show configuration routing-options 
route-distinguisher-id 192.168.1.6;
router-id 192.168.1.6;
autonomous-system 64512;

root@R6> configure 
Entering configuration mode

[edit]
root@R6# set routing-instances CUST-A route-distinguisher 64512:999 

[edit]
root@R6# show | compare 
[edit routing-instances CUST-A]
+    route-distinguisher 64512:999;

[edit]
root@R6# commit 
commit complete

[edit]
root@R6# run show route advertising-protocol bgp 192.168.1.5 detail 

CUST-A.inet.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
* 172.16.3.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 64512:999
     VPN Label: 18
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

* 192.168.200.0/24 (1 entry, 1 announced)
 BGP group int-group type Internal
     Route Distinguisher: 64512:999
     VPN Label: 18
     Nexthop: Self
     Flags: Nexthop Change
     Localpref: 100
     AS path: [64512] I 
     Communities: target:65412:111

[edit]
root@R6# 
```

As we see above, though we still have the `route-distinguisher-id`, the value configured under the routing-instance (`64512:999`)is advertised along with the VPN routes.

