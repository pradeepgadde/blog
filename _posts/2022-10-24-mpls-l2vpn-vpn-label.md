---
layout: single
title:  "L2VPN VPN Label Calculation"
date:   2022-10-24 01:50:04 +0530
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

# L2VPN VPN Label Calculation

L2VPN always advertise a block of labels
Remote sites calculate the labe they should use based on their Site ID

Label Blocks
Label Base
Site ID
Offset

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
         				interface ge-0/0/0.0; {
         				    remote-site-id 5;
         				}
         				
         }
    }
}
```

Label-base : 800000

Offset: 5

Connection-site : 5

Incoming Label: 800000

Outgoing Label: 800000X



VPN Label = Label Base + (Remote-site-identifier – offset)

=800000+(5-5)

=800000



We configured only site IDs, but not the label base and offset values; 

Offset, default value is `32` that means, we can have maximum 32 sites (0 to 31)

Label base default value is `800000`.

If we want more than 32 sites, we can configure another label base (multiple starting points)

