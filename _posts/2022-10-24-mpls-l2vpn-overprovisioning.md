---
layout: single
title: "L2VPN Overprovisioning"
date:   2022-10-24 01:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
header:
  overlay_image: /assets/images/networking-1.png
  og_image: /assets/images/networking-1.png
  teaser: /assets/images/junos.png

author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# L2VPN Overprovisioning

Many attachment circuits, one routing instance

Hub and spoke network

Defined in RFC6624 

pre-provisioning more attachment circuits than a site needs at the moment

Label blocks can contain up to 32 labels in one advertisement

Interface config for Ethernet-VLAN pseudowire type

`encapsulation extended-vlan-ccc` on physical interface  `family ccc` on logical interface 

```sh
site my-customer-1 {
		site-identifier 1;

      interface ge-0/0/0.200 {
            remote-site-id 2;
      }
      interface ge-0/0/0.300 {
            remote-site-id 3;
      }
}
```



Implicit vs Explicit Remote Site Configuration.

VPN label block allocated in advance.

When those sites come up, they know which label to use.

