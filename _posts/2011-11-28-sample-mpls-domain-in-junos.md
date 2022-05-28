---

layout: single
title:  " A Sample MPLS Domain in Junos"
date:   2011-11-28 07:59:04 +0530
categories: Networking
tags: MPLS
show_date: true
toc: true
toc_sticky: true
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
## Bangalore Configuration
```sh
--- JUNOS 11.2R1.10 built 2011-07-29 08:46:06 UTC
pradeep@Bangalore> show configuration   | no-more 
## Last commit: 2011-11-28 16:09:43 UTC by pradeep
version 11.2R1.10;
system {
    host-name Bangalore;
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
                address 192.168.1.21/30;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 192.168.1.34/30;
            }
            family mpls;
        }
    }
    fe-0/0/2 {
        unit 0 {
            family inet {
                address 192.168.1.13/30;
            }
            family mpls;
        }
    }
    fe-0/0/3 {
        unit 0 {
            family inet {
                address 192.168.1.17/30;
            }
            family mpls;
        }
    }
    fe-0/0/7 {
        unit 0 {
            family inet {
                address 172.16.1.4/32;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.10.10.4/32;
            }
        }
    }
}
routing-options {
    max-interface-supported 0;
}
protocols {
    rsvp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
        interface fe-0/0/2.0;
        interface fe-0/0/3.0;
    }
    mpls {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
        interface fe-0/0/2.0;
        interface fe-0/0/3.0;
    }
    ospf {
        area 0.0.0.0 {
            interface lo0.0;
            interface ge-0/0/0.0;
            interface ge-0/0/1.0;
            interface fe-0/0/2.0;
            interface fe-0/0/3.0;
        }
    }
    ldp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
        interface fe-0/0/2.0;
        interface fe-0/0/3.0;
    }
}
security {
    policies {
        default-policy {
            permit-all;
        }
    }
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
}

pradeep@Bangalore>  edit 
Entering configuration mode
Users currently editing the configuration:
  pradeep terminal u0 (pid 1138) on since 2011-11-28 16:02:39 UTC, idle 00:26:20
      [edit]

[edit]
pradeep@Bangalore# show interfaces fe-0/0/7 
unit 0 {
    family inet {
        address 172.16.1.4/32;
    }
}

[edit]
pradeep@Bangalore# replace pattern 172.16.1.4/32 with 172.16.1.4/24 

[edit]
pradeep@Bangalore# commit 
commit complete
```
## Bangalore Routes
```sh 
[edit]
pradeep@Bangalore# run show route | no-more 

inet.0: 27 destinations, 27 routes (27 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[OSPF/10] 00:19:38, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0
10.10.10.2/32      *[OSPF/10] 00:37:00, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0
10.10.10.3/32      *[OSPF/10] 00:19:38, metric 2
                    > to 192.168.1.33 via ge-0/0/1.0
                      to 192.168.1.18 via fe-0/0/3.0
10.10.10.4/32      *[Direct/0] 00:38:27
                    > via lo0.0
10.10.10.5/32      *[OSPF/10] 00:19:07, metric 3
                      to 192.168.1.22 via ge-0/0/0.0
                      to 192.168.1.14 via fe-0/0/2.0
                    > to 192.168.1.18 via fe-0/0/3.0
10.10.10.6/32      *[OSPF/10] 00:19:07, metric 2
                      to 192.168.1.22 via ge-0/0/0.0
                    > to 192.168.1.18 via fe-0/0/3.0
10.10.10.7/32      *[OSPF/10] 00:36:08, metric 2
                    > to 192.168.1.14 via fe-0/0/2.0
10.10.10.8/32      *[OSPF/10] 00:37:05, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0
10.10.10.9/32      *[OSPF/10] 00:37:05, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0
172.16.1.4/32      *[Local/0] 00:00:08
                      Reject
192.168.1.0/30     *[OSPF/10] 00:37:05, metric 2
                    > to 192.168.1.14 via fe-0/0/2.0
192.168.1.4/30     *[OSPF/10] 00:19:07, metric 3
                    > to 192.168.1.22 via ge-0/0/0.0
                      to 192.168.1.18 via fe-0/0/3.0
192.168.1.8/30     *[OSPF/10] 00:19:38, metric 2
                    > to 192.168.1.18 via fe-0/0/3.0
192.168.1.12/30    *[Direct/0] 00:37:51
                    > via fe-0/0/2.0
192.168.1.13/32    *[Local/0] 00:37:55
                      Local via fe-0/0/2.0
192.168.1.16/30    *[Direct/0] 00:20:30
                    > via fe-0/0/3.0
192.168.1.17/32    *[Local/0] 00:37:55
                      Local via fe-0/0/3.0
192.168.1.20/30    *[Direct/0] 00:37:51
                    > via ge-0/0/0.0
192.168.1.21/32    *[Local/0] 00:37:56
                      Local via ge-0/0/0.0
192.168.1.24/30    *[OSPF/10] 00:19:38, metric 2
                    > to 192.168.1.18 via fe-0/0/3.0
192.168.1.28/30    *[OSPF/10] 00:26:29, metric 3
                    > to 192.168.1.14 via fe-0/0/2.0
192.168.1.32/30    *[Direct/0] 00:37:50
                    > via ge-0/0/1.0
192.168.1.34/32    *[Local/0] 00:37:55
                      Local via ge-0/0/1.0
192.168.1.36/30    *[OSPF/10] 00:37:05, metric 2
                    > to 192.168.1.33 via ge-0/0/1.0
192.168.1.40/30    *[OSPF/10] 00:26:05, metric 2
                      to 192.168.1.22 via ge-0/0/0.0
                    > to 192.168.1.14 via fe-0/0/2.0
192.168.1.44/30    *[OSPF/10] 00:19:23, metric 2
                    > to 192.168.1.22 via ge-0/0/0.0
224.0.0.5/32       *[OSPF/10] 00:38:39, metric 1
                      MultiRecv

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:19:38, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0
10.10.10.2/32      *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0
10.10.10.3/32      *[LDP/9] 00:19:38, metric 1
                      to 192.168.1.33 via ge-0/0/1.0, Push 299776
                    > to 192.168.1.18 via fe-0/0/3.0, Push 299776
10.10.10.5/32      *[LDP/9] 00:19:07, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Push 299952
                      to 192.168.1.14 via fe-0/0/2.0, Push 299872
                      to 192.168.1.18 via fe-0/0/3.0, Push 299872
10.10.10.6/32      *[LDP/9] 00:19:07, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Push 299984
                      to 192.168.1.18 via fe-0/0/3.0, Push 299888
10.10.10.7/32      *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Push 299776
10.10.10.8/32      *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0
10.10.10.9/32      *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0

mpls.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:38:31, metric 1
                      Receive
1                  *[MPLS/0] 00:38:31, metric 1
                      Receive
2                  *[MPLS/0] 00:38:31, metric 1
                      Receive
299776             *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0, Pop      
299776(S=0)        *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0, Pop      
299792             *[LDP/9] 00:19:38, metric 1
                      to 192.168.1.33 via ge-0/0/1.0, Swap 299776
                    > to 192.168.1.18 via fe-0/0/3.0, Swap 299776
299808             *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Pop      
299808(S=0)        *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Pop      
299824             *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Swap 299776
299856             *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:33:30, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Pop      
299872             *[LDP/9] 00:19:07, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Swap 299952
                      to 192.168.1.14 via fe-0/0/2.0, Swap 299872
                      to 192.168.1.18 via fe-0/0/3.0, Swap 299872
299888             *[LDP/9] 00:19:38, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0, Pop      
299888(S=0)        *[LDP/9] 00:19:38, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0, Pop      
299904             *[LDP/9] 00:19:07, metric 1
                      to 192.168.1.22 via ge-0/0/0.0, Swap 299984
                    > to 192.168.1.18 via fe-0/0/3.0, Swap 299888

[edit]
pradeep@Bangalore# run show route protocol ldp | no-more 

inet.0: 27 destinations, 27 routes (27 active, 0 holddown, 0 hidden)

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:19:43, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0
10.10.10.2/32      *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0
10.10.10.3/32      *[LDP/9] 00:19:43, metric 1
                      to 192.168.1.33 via ge-0/0/1.0, Push 299776
                    > to 192.168.1.18 via fe-0/0/3.0, Push 299776
10.10.10.5/32      *[LDP/9] 00:19:12, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Push 299952
                      to 192.168.1.14 via fe-0/0/2.0, Push 299872
                      to 192.168.1.18 via fe-0/0/3.0, Push 299872
10.10.10.6/32      *[LDP/9] 00:19:12, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Push 299984
                      to 192.168.1.18 via fe-0/0/3.0, Push 299888
10.10.10.7/32      *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Push 299776
10.10.10.8/32      *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0
10.10.10.9/32      *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0

mpls.0: 15 destinations, 15 routes (15 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

299776             *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0, Pop      
299776(S=0)        *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.33 via ge-0/0/1.0, Pop      
299792             *[LDP/9] 00:19:43, metric 1
                      to 192.168.1.33 via ge-0/0/1.0, Swap 299776
                    > to 192.168.1.18 via fe-0/0/3.0, Swap 299776
299808             *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Pop      
299808(S=0)        *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Pop      
299824             *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.14 via fe-0/0/2.0, Swap 299776
299856             *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:33:35, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Pop      
299872             *[LDP/9] 00:19:12, metric 1
                    > to 192.168.1.22 via ge-0/0/0.0, Swap 299952
                      to 192.168.1.14 via fe-0/0/2.0, Swap 299872
                      to 192.168.1.18 via fe-0/0/3.0, Swap 299872
299888             *[LDP/9] 00:19:43, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0, Pop      
299888(S=0)        *[LDP/9] 00:19:43, metric 1
                    > to 192.168.1.18 via fe-0/0/3.0, Pop      
299904             *[LDP/9] 00:19:12, metric 1
                      to 192.168.1.22 via ge-0/0/0.0, Swap 299984
                    > to 192.168.1.18 via fe-0/0/3.0, Swap 299888

```
## Bangalore MPLS LSPs
```sh
[edit]
pradeep@Bangalore# run show mpls lsp 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
pradeep@Bangalore# 
pradeep@Bangalore# run show mpls lsp        interface 
Interface        State       Administrative groups (x: extended)
ge-0/0/0.0       Up         <none>
fe-0/0/2.0       Up         <none>
ge-0/0/1.0       Up         <none>
fe-0/0/3.0       Up         <none>


pradeep@Bangalore# run show rsvp  interface 
RSVP interface: 4 active
                  Active Subscr- Static      Available   Reserved    Highwater
Interface   State resv   iption  BW          BW          BW          mark
fe-0/0/2.0  Up         0   100%  100Mbps     100Mbps     0bps        0bps       
fe-0/0/3.0  Up         0   100%  100Mbps     100Mbps     0bps        0bps       
ge-0/0/0.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       
ge-0/0/1.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       

[edit]
pradeep@Bangalore# 
pradeep@Bangalore# run show rsvp interface    
RSVP interface: 4 active
                  Active Subscr- Static      Available   Reserved    Highwater
Interface   State resv   iption  BW          BW          BW          mark
fe-0/0/2.0  Up         0   100%  100Mbps     100Mbps     0bps        0bps       
fe-0/0/3.0  Up         0   100%  100Mbps     100Mbps     0bps        0bps       
ge-0/0/0.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       
ge-0/0/1.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       

[edit]
pradeep@Bangalore# 
pradeep@Bangalore# run show rsvp neighbor 
RSVP neighbor: 0 learned
```



## Chennai Configuration

```sh
Chennai
[edit]
pradeep@Bangalore# run ssh 10.10.10.9 
The authenticity of host '10.10.10.9 (10.10.10.9)' can't be established.
ECDSA key fingerprint is ed:0e:2e:bd:10:56:a5:be:a4:cc:af:f2:72:15:6e:42.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.9' (ECDSA) to the list of known hosts.

pradeep@10.10.10.9's password: 
--- JUNOS 11.2R1.10 built 2011-07-29 08:46:06 UTC
pradeep@Chennai> 

pradeep@Chennai> 

pradeep@Chennai> show configuration | no-more 
## Last commit: 2011-11-28 18:16:03 UTC by root
version 11.2R1.10;
system {
    host-name Chennai;
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
                address 192.168.1.33/30;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 192.168.1.37/30;
            }
            family mpls;
        }
    }
    fe-0/0/7 {
        unit 0 {
            family inet {
                address 172.16.1.9/32;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.10.10.9/32;
            }
        }
    }
}
routing-options {
    max-interface-supported 0;
}
protocols {
    rsvp {
        interface ge-0/0/0.0;
        interface ge-0/0/1.0;
    }
    mpls {
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
    policies {
        default-policy {
            permit-all;
        }
    }
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
}

pradeep@Chennai> edit 
Entering configuration mode
Users currently editing the configuration:
  root terminal u0 (pid 1184) on since 2011-11-28 16:29:42 UTC, idle 00:28:36
      [edit]

[edit]
pradeep@Chennai# show interfaces fe-0/0/7 
unit 0 {
    family inet {
        address 172.16.1.9/32;
    }
}

[edit]
pradeep@Chennai# replace pattern 172.16.1.9/32 with 172.16.1.9/32      24 

[edit]
pradeep@Chennai# commit 
commit complete
```
## Chennai Routes
```sh
[edit]
pradeep@Chennai# run show route | no-more 

inet.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[OSPF/10] 00:21:16, metric 2
                      to 192.168.1.34 via ge-0/0/0.0
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.2/32      *[OSPF/10] 00:38:38, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.3/32      *[OSPF/10] 01:14:23, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.4/32      *[OSPF/10] 00:38:43, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[OSPF/10] 00:20:45, metric 4
                      to 192.168.1.34 via ge-0/0/0.0
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.6/32      *[OSPF/10] 00:20:45, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
                      to 192.168.1.38 via ge-0/0/1.0
10.10.10.7/32      *[OSPF/10] 00:37:47, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.8/32      *[OSPF/10] 00:38:43, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.9/32      *[Direct/0] 02:15:01
                    > via lo0.0
172.16.1.9/32      *[Local/0] 00:00:14
                      Reject
192.168.1.0/30     *[OSPF/10] 00:38:43, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.4/30     *[OSPF/10] 00:20:45, metric 4
                      to 192.168.1.34 via ge-0/0/0.0
                    > to 192.168.1.38 via ge-0/0/1.0
192.168.1.8/30     *[OSPF/10] 00:21:16, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
                      to 192.168.1.38 via ge-0/0/1.0
192.168.1.12/30    *[OSPF/10] 00:38:43, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.16/30    *[OSPF/10] 00:22:08, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.20/30    *[OSPF/10] 00:38:43, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.24/30    *[OSPF/10] 00:22:07, metric 2
                    > to 192.168.1.38 via ge-0/0/1.0
192.168.1.28/30    *[OSPF/10] 00:28:07, metric 4
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.32/30    *[Direct/0] 00:39:28
                    > via ge-0/0/0.0
192.168.1.33/32    *[Local/0] 02:15:01
                      Local via ge-0/0/0.0
192.168.1.36/30    *[Direct/0] 01:15:08
                    > via ge-0/0/1.0
192.168.1.37/32    *[Local/0] 02:15:01
                      Local via ge-0/0/1.0
192.168.1.40/30    *[OSPF/10] 00:27:44, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.44/30    *[OSPF/10] 00:21:01, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
224.0.0.5/32       *[OSPF/10] 02:15:03, metric 1
                      MultiRecv

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:21:16, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299888
                      to 192.168.1.38 via ge-0/0/1.0, Push 299952
10.10.10.2/32      *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299856
10.10.10.3/32      *[LDP/9] 01:08:16, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.4/32      *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[LDP/9] 00:20:45, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299872
                      to 192.168.1.38 via ge-0/0/1.0, Push 299936
10.10.10.6/32      *[LDP/9] 00:20:45, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Push 299904
                    > to 192.168.1.38 via ge-0/0/1.0, Push 299968
10.10.10.7/32      *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299824
10.10.10.8/32      *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299808

mpls.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 02:15:03, metric 1
                      Receive
1                  *[MPLS/0] 02:15:03, metric 1
                      Receive
2                  *[MPLS/0] 02:15:03, metric 1
                      Receive
299776             *[LDP/9] 01:08:16, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      
299776(S=0)        *[LDP/9] 01:08:16, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      
299856             *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299872             *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299808
299888             *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299824
299920             *[LDP/9] 00:35:08, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299856
299936             *[LDP/9] 00:20:45, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299872
                      to 192.168.1.38 via ge-0/0/1.0, Swap 299936
299952             *[LDP/9] 00:21:16, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Swap 299888
                    > to 192.168.1.38 via ge-0/0/1.0, Swap 299952
299968             *[LDP/9] 00:20:45, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Swap 299904
                    > to 192.168.1.38 via ge-0/0/1.0, Swap 299968

[edit]
pradeep@Chennai# run show route protocol ldp | no-more 

inet.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:21:20, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299888
                      to 192.168.1.38 via ge-0/0/1.0, Push 299952
10.10.10.2/32      *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299856
10.10.10.3/32      *[LDP/9] 01:08:20, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.4/32      *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[LDP/9] 00:20:49, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299872
                      to 192.168.1.38 via ge-0/0/1.0, Push 299936
10.10.10.6/32      *[LDP/9] 00:20:49, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Push 299904
                    > to 192.168.1.38 via ge-0/0/1.0, Push 299968
10.10.10.7/32      *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299824
10.10.10.8/32      *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299808

mpls.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

299776             *[LDP/9] 01:08:20, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      
299776(S=0)        *[LDP/9] 01:08:20, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      
299856             *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299872             *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299808
299888             *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299824
299920             *[LDP/9] 00:35:12, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299856
299936             *[LDP/9] 00:20:49, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299872
                      to 192.168.1.38 via ge-0/0/1.0, Swap 299936
299952             *[LDP/9] 00:21:20, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Swap 299888
                    > to 192.168.1.38 via ge-0/0/1.0, Swap 299952
299968             *[LDP/9] 00:20:49, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Swap 299904
                    > to 192.168.1.38 via ge-0/0/1.0, Swap 299968
```
## Chennai MPLS LSPs
```sh
[edit]
pradeep@Chennai# run show mpls lsp 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]
t

pradeep@Chennai# run show mpls lsp de    tail 
Ingress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Egress LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

Transit LSP: 0 sessions
Total 0 displayed, Up 0, Down 0

[edit]


pradeep@Chennai# run show rsvp interface 
RSVP interface: 2 active
                  Active Subscr- Static      Available   Reserved    Highwater
Interface   State resv   iption  BW          BW          BW          mark
ge-0/0/0.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       
ge-0/0/1.0  Up         0   100%  1000Mbps    1000Mbps    0bps        0bps       

[edit]
pradeep@Chennai# 
pradeep@Chennai# run show rsvp  neighbor 
RSVP neighbor: 0 learned

[edit]
```

```sh
[edit]
pradeep@Chennai# run show route 

inet.0: 25 destinations, 25 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[OSPF/10] 00:05:46, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.2/32      *[OSPF/10] 00:44:50, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.4/32      *[OSPF/10] 00:44:55, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[OSPF/10] 00:05:46, metric 4
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.6/32      *[OSPF/10] 00:05:46, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.7/32      *[OSPF/10] 00:43:59, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.8/32      *[OSPF/10] 00:44:55, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.9/32      *[Direct/0] 02:21:13
                    > via lo0.0
172.16.1.9/32      *[Local/0] 00:06:26
                      Reject
192.168.1.0/30     *[OSPF/10] 00:44:55, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.4/30     *[OSPF/10] 00:05:46, metric 4
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.8/30     *[OSPF/10] 00:05:46, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.12/30    *[OSPF/10] 00:44:55, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.16/30    *[OSPF/10] 00:28:20, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.20/30    *[OSPF/10] 00:44:55, metric 2
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.24/30    *[OSPF/10] 00:00:46, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.28/30    *[OSPF/10] 00:34:19, metric 4
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.32/30    *[Direct/0] 00:45:40
                    > via ge-0/0/0.0
192.168.1.33/32    *[Local/0] 02:21:13
                      Local via ge-0/0/0.0
192.168.1.36/30    *[Direct/0] 00:00:48
---(more)---
                                        
                    > via ge-0/0/1.0
192.168.1.37/32    *[Local/0] 02:21:13
                      Local via ge-0/0/1.0
192.168.1.40/30    *[OSPF/10] 00:33:56, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.44/30    *[OSPF/10] 00:27:13, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
192.168.1.48/30    *[OSPF/10] 00:00:51, metric 3
                    > to 192.168.1.34 via ge-0/0/0.0
224.0.0.5/32       *[OSPF/10] 02:21:15, metric 1
                      MultiRecv

inet.3: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299888
10.10.10.2/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299856
10.10.10.4/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299872
10.10.10.6/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299904
10.10.10.7/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299824
10.10.10.8/32      *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299808

mpls.0: 11 destinations, 11 routes (11 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 02:21:15, metric 1
                      Receive
1                  *[MPLS/0] 02:21:15, metric 1
                      Receive
2                  *[MPLS/0] 02:21:15, metric 1
                      Receive
299856             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
---(more 87%)---
                                        
299872             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299808
299888             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299824
299920             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299856
299936             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299872
299952             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299888
299968             *[LDP/9] 00:05:46, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299904

[edit]

pradeep@Chennai# run show route protocol ldp    

inet.0: 26 destinations, 26 routes (26 active, 0 holddown, 0 hidden)

inet.3: 8 destinations, 8 routes (8 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.10.10.1/32      *[LDP/9] 00:00:01, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Push 299888
                    > to 192.168.1.38 via ge-0/0/1.0, Push 299840
10.10.10.2/32      *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299856
10.10.10.3/32      *[LDP/9] 00:00:04, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0
10.10.10.4/32      *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0
10.10.10.5/32      *[LDP/9] 00:00:01, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299872
                      to 192.168.1.38 via ge-0/0/1.0, Push 299872
10.10.10.6/32      *[LDP/9] 00:00:01, metric 1
                      to 192.168.1.34 via ge-0/0/0.0, Push 299904
                    > to 192.168.1.38 via ge-0/0/1.0, Push 299888
10.10.10.7/32      *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299824
10.10.10.8/32      *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Push 299808

mpls.0: 13 destinations, 13 routes (13 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

299856             *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299856(S=0)        *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Pop      
299872             *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299808
299888             *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299824
299920             *[LDP/9] 00:05:53, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299856
299936             *[LDP/9] 00:00:01, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299872
                      to 192.168.1.38 via ge-0/0/1.0, Swap 299872
299952             *[LDP/9] 00:00:01, metric 1
                                        
                      to 192.168.1.34 via ge-0/0/0.0, Swap 299888
                    > to 192.168.1.38 via ge-0/0/1.0, Swap 299840
299968             *[LDP/9] 00:00:01, metric 1
                    > to 192.168.1.34 via ge-0/0/0.0, Swap 299904
                      to 192.168.1.38 via ge-0/0/1.0, Swap 299888
299984             *[LDP/9] 00:00:04, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      
299984(S=0)        *[LDP/9] 00:00:04, metric 1
                    > to 192.168.1.38 via ge-0/0/1.0, Pop      

```

