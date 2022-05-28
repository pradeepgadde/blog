---

layout: single
title:  " A Sample MPLS Domain in Junos"
date:   2011-11-28 07:59:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
header:
  teaser: /assets/images/junos.png
author:
  name     : "Junos"
  avatar   : "/assets/images/junos.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

Here is the topology of our setup.
![MPLS Domain]({{ site.url }}{{ site.baseurl }}/assets/images/mpls-setup.jpeg)
## Delhi Configuration
```sh
pradeep@Delhi> 

pradeep@Delhi> show configuration | no-more 
## Last commit: 2011-11-28 16:06:51 UTC by pradeep
version 10.0R1.8;
system {
    host-name Delhi;
    root-authentication {
        encrypted-password "$1$WIXfdr97$6UJDQGFv/IbpofkSMDsAx1"; ## SECRET-DATA
    }
    login {
        user pradeep {
            uid 2000;
            class super-user;
            authentication {
                encrypted-password "$1$o07uunIS$1dNBCv8EklJK7pXwUaqfZ/"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh;
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 192.168.1.30/30;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 192.168.1.6/30;
            }
            family mpls;
        }
    }
    ge-0/0/3 {
        unit 0 {
            family inet {
                address 172.16.1.5/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.10.10.5/32;
            }
        }
    }
}
protocols {
    rsvp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
    }
    mpls {
        label-switched-path Delhi_to_Tiruvanantapuram {
            to 10.10.10.1;
        }
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
    }
    ospf {
        area 0.0.0.0 {
            interface lo0.0;
            interface ge-0/0/0.0;
            interface ge-0/0/1.0;
        }
    }
    ldp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
    }
}
security {
    zones {
        security-zone trust {
            interfaces {
                all {
                    host-inbound-traffic {
                        system-services {
                            all;
                        }
                        protocols {
                            all;
                        }
                    }
                }
            }
        }
    }
    policies {
        default-policy {
            permit-all;
        }
    }
}
```
## Delhi Routing Table
```
pradeep@Delhi> show route | no-more 

inet.0: 26 destinations, 26 routes (26 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[OSPF/10] 00:05:51, metric 2
                    > to 192.168.1.5 via ge-0/0/1.0
10.10.10.2/32      *[OSPF/10] 00:05:57, metric 2
                    > to 192.168.1.5 via ge-0/0/1.0
10.10.10.3/32      *[OSPF/10] 00:05:51, metric 3
                    > to 192.168.1.5 via ge-0/0/1.0
10.10.10.4/32      *[OSPF/10] 00:05:57, metric 3
                    > to 192.168.1.29 via ge-0/0/0.0
                      to 192.168.1.5 via ge-0/0/1.0
10.10.10.5/32      *[Direct/0] 01:07:03
                    > via lo0.0
10.10.10.6/32      *[OSPF/10] 00:05:57, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0
10.10.10.7/32      *[OSPF/10] 00:12:30, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0
10.10.10.8/32      *[OSPF/10] 00:12:30, metric 2
                    > to 192.168.1.29 via ge-0/0/0.0
10.10.10.9/32      *[OSPF/10] 00:05:57, metric 4
                    > to 192.168.1.29 via ge-0/0/0.0
                      to 192.168.1.5 via ge-0/0/1.0
172.16.1.0/24      *[Direct/0] 00:00:57
                    > via ge-0/0/3.0
172.16.1.5/32      *[Local/0] 00:16:41
                      Local via ge-0/0/3.0
192.168.1.0/30     *[OSPF/10] 00:12:30, metric 2
                    > to 192.168.1.29 via ge-0/0/0.0
192.168.1.4/30     *[Direct/0] 00:06:52
                    > via ge-0/0/1.0
192.168.1.6/32     *[Local/0] 01:06:41
                      Local via ge-0/0/1.0
192.168.1.8/30     *[OSPF/10] 00:05:57, metric 2
                    > to 192.168.1.5 via ge-0/0/1.0
192.168.1.12/30    *[OSPF/10] 00:12:30, metric 3
                    > to 192.168.1.29 via ge-0/0/0.0
192.168.1.16/30    *[OSPF/10] 00:05:51, metric 3
                    > to 192.168.1.5 via ge-0/0/1.0
192.168.1.20/30    *[OSPF/10] 00:05:57, metric 3
                    > to 192.168.1.5 via ge-0/0/1.0
192.168.1.24/30    *[OSPF/10] 00:05:51, metric 3
                    > to 192.168.1.5 via ge-0/0/1.0
192.168.1.28/30    *[Direct/0] 00:13:14
                    > via ge-0/0/0.0
192.168.1.30/32    *[Local/0] 01:06:41
                      Local via ge-0/0/0.0
192.168.1.32/30    *[OSPF/10] 00:05:57, metric 4
                    > to 192.168.1.29 via ge-0/0/0.0
                      to 192.168.1.5 via ge-0/0/1.0
192.168.1.36/30    *[OSPF/10] 00:05:51, metric 4
                    > to 192.168.1.5 via ge-0/0/1.0
192.168.1.40/30    *[OSPF/10] 00:05:57, metric 3
                    > to 192.168.1.29 via ge-0/0/0.0
                      to 192.168.1.5 via ge-0/0/1.0
192.168.1.44/30    *[OSPF/10] 00:05:57, metric 2
                    > to 192.168.1.5 via ge-0/0/1.0
224.0.0.5/32       *[OSPF/10] 01:07:04, metric 1
                      MultiRecv

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:05:51, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Push 299856
10.10.10.2/32      *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Push 299776
10.10.10.3/32      *[LDP/9] 00:05:51, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Push 299824
10.10.10.4/32      *[LDP/9] 00:05:56, metric 1
                      to 192.168.1.29 via ge-0/0/0.0, Push 299808
                    > to 192.168.1.5 via ge-0/0/1.0, Push 299792
10.10.10.6/32      *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0
10.10.10.7/32      *[LDP/9] 00:12:29, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0
10.10.10.8/32      *[LDP/9] 00:12:29, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0, Push 299776
10.10.10.9/32      *[LDP/9] 00:05:56, metric 1
                      to 192.168.1.29 via ge-0/0/0.0, Push 299824
                    > to 192.168.1.5 via ge-0/0/1.0, Push 299808

mpls.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 01:07:03, metric 1
                      Receive
1                  *[MPLS/0] 01:07:03, metric 1
                      Receive
2                  *[MPLS/0] 01:07:03, metric 1
                      Receive
299952             *[LDP/9] 00:12:29, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0, Pop      
299952(S=0)        *[LDP/9] 00:12:29, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0, Pop      
299968             *[LDP/9] 00:12:29, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0, Swap 299776
299984             *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.29 via ge-0/0/0.0, Swap 299808
                      to 192.168.1.5 via ge-0/0/1.0, Swap 299792
300000             *[LDP/9] 00:05:56, metric 1
                      to 192.168.1.29 via ge-0/0/0.0, Swap 299824
                    > to 192.168.1.5 via ge-0/0/1.0, Swap 299808
300016             *[LDP/9] 00:05:51, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Swap 299824
300032             *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Swap 299776
300048             *[LDP/9] 00:05:51, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Swap 299856
300064             *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Pop      
300064(S=0)        *[LDP/9] 00:05:56, metric 1
                    > to 192.168.1.5 via ge-0/0/1.0, Pop      
```
## Delhi MPLS LSPs
```sh

pradeep@Delhi> show mpls lsp 
Ingress LSP: 1 sessions
To              From            State Rt P     ActivePath       LSPname
10.10.10.1      0.0.0.0         Dn     0       -                Delhi_to_Tiruvanantapuram
Total 1 displayed, Up 0, Down 1

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0


pradeep@Delhi> show mpls lsp detail 
Ingress LSP: 1 sessions

10.10.10.1
  From: 0.0.0.0, State: Dn, ActiveRoute: 0, LSPname: Delhi_to_Tiruvanantapuram
  ActivePath: (none)
  LoadBalance: Random
  Encoding type: Packet, Switching type: Packet, GPID: IPv4
  Primary                    State: Dn
    Priorities: 7 0
    SmartOptimizeTimer: 180
    Will be enqueued for recomputation in 22 second(s).
    1 Nov 28 16:24:13.810 CSPF: could not determine self[140 times]
Total 1 displayed, Up 0, Down 1

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0
```




