---

layout: single
title:  "MPLS in Junos"
date:   2011-01-21 07:59:04 +0530
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

Multi Protocol Label Switching

Here’s a simple setup to understand the basic operation of MPLS.   Please refer to the sample configuration of  “Static Label Switched Paths”  .

The following post explains the basic MPLS operations – Push,Swap and Pop , with a sample traceroute in both directions.


## Ingress

```sh
root@Ingress# show | no-more                        
## Last changed: 2011-01-21 00:11:48 UTC
version 10.4R1.9;
system {
    host-name Ingress;
    root-authentication {
        encrypted-password "$1$cYyoUQl.$blO1yJaM39r/7rGw.a8k.1"; ## SECRET-DATA
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 10.20.30.1/24;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 100.1.1.1/30;
            }
            family mpls;
        }
    }
}
routing-options {
    static {
        route 0.0.0.0/0 next-hop 100.1.1.2;
    }
}
protocols {
    mpls {
        traffic-engineering bgp-igp;
        traceoptions {
            file mplstest;
            flag all;
        }
        static-label-switched-path path1 {
            ingress {
                next-hop 100.1.1.2;
                to 172.16.1.1;
                push 1000123;
            }
        }
        static-label-switched-path path2 {
            transit 1000222 {
                next-hop 10.20.30.5;
                pop;
            }
        }
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}
```

```
[edit]
root@Ingress# run show route         

inet.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 00:00:03
                    > to 100.1.1.2 via ge-0/0/1.0
10.20.30.1/32      *[Local/0] 00:00:03
                      Reject
100.1.1.0/30       *[Direct/0] 00:00:03
                    > via ge-0/0/1.0
100.1.1.1/32       *[Local/0] 00:00:03
                      Local via ge-0/0/1.0
172.16.1.1/32      *[MPLS/6/1] 00:00:03, metric 0
                    > to 100.1.1.2 via ge-0/0/1.0, Push 1000123

mpls.0: 3 destinations, 3 routes (3 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:00:03, metric 1
                      Receive
1                  *[MPLS/0] 00:00:03, metric 1
                      Receive
2                  *[MPLS/0] 00:00:03, metric 1
                      Receive           

[edit]
root@Ingress# 
```

```sh
[edit]
root@Ingress# run show mpls static-lsp extensive 
Ingress LSPs:
LSPname: path1, To: 172.16.1.1
  State: Up
  Nexthop: 100.1.1.2 Via ge-0/0/1.0
  LabelOperation: Push, Outgoing-label: 1000123
  Created: Thu Jan 20 23:17:00 2011
  Bandwidth: 0 bps
  Statistics: Packets 343, Bytes 19328
Total 1, displayed 1, Up 1, Down 0

Transit LSPs:
LSPname: path2, Incoming-label: 1000222
  State: Dn, Sub State: Nexthop interface down or has no family mpls
  Nexthop: 10.20.30.5 Via ge-0/0/0.0
  LabelOperation: Pop
  Created: Fri Jan 21 00:04:05 2011
  Bandwidth: 0 bps
Total 1, displayed 1, Up 0, Down 1

Bypass LSPs:
Total 0, displayed 0, Up 0, Down 0

```

```sh
[edit]
root@Ingress# run traceroute 172.16.1.1 
traceroute to 172.16.1.1 (172.16.1.1), 30 hops max, 40 byte packets
 1  100.1.1.2 (100.1.1.2)  4.251 ms  4.066 ms  3.515 ms
     MPLS Label=1000123 CoS=0 TTL=1 S=1
 2  200.1.1.1 (200.1.1.1)  4.088 ms  4.038 ms  4.703 ms
     MPLS Label=1000456 CoS=0 TTL=1 S=1
 3  172.16.1.1 (172.16.1.1)  10.342 ms  4.158 ms  10.461 ms

[edit]
root@Ingress# run traceroute 192.168.1.1   
traceroute to 192.168.1.1 (192.168.1.1), 30 hops max, 40 byte packets
 1  100.1.1.2 (100.1.1.2)  4.032 ms  9.723 ms  3.827 ms
     MPLS Label=1000123 CoS=0 TTL=1 S=1
 2  192.168.1.1 (192.168.1.1)  4.586 ms  4.392 ms  4.298 ms
     MPLS Label=1000456 CoS=0 TTL=1 S=1
 3  192.168.1.100 (192.168.1.100)  3.201 ms  2.647 ms  2.729 ms
 4  192.168.1.1 (192.168.1.1)  11.323 ms  4.700 ms  10.164 ms

[edit]
root@Ingress# run traceroute 192.168.1.100  
traceroute to 192.168.1.100 (192.168.1.100), 30 hops max, 40 byte packets
 1  100.1.1.2 (100.1.1.2)  4.069 ms  4.038 ms  3.863 ms
     MPLS Label=1000123 CoS=0 TTL=1 S=1
 2  200.1.1.1 (200.1.1.1)  9.844 ms  4.717 ms  17.136 ms
     MPLS Label=1000456 CoS=0 TTL=1 S=1
 3  192.168.1.100 (192.168.1.100)  4.209 ms  4.038 ms  4.301 ms

[edit]
```

## Transit
```sh
root@Transit# show | no-more 
## Last changed: 2011-01-21 03:27:36 UTC
version 10.2R3.10;
system {
    host-name Transit;
    root-authentication {
        encrypted-password "$1$uLjcwqNG$Elo7yksUMhtaopeQUj0Uf0"; ## SECRET-DATA
    }
}
interfaces {
    fe-0/0/0 {
        unit 0 {
            family inet {
                address 100.1.1.2/30;
            }
            family mpls;
        }
    }
    fe-0/0/1 {
        unit 0 {
            family inet {
                address 200.1.1.2/30;
            }
            family mpls;
        }
    }
}
protocols {
    mpls {
        traffic-engineering bgp-igp;
        traceoptions {
            file mplstest;
            flag all;
        }
        static-label-switched-path path1 {
            transit 1000123 {
                next-hop 200.1.1.1;
                swap 1000456;
            }
        }
        static-label-switched-path path2 {
            transit 1000111 {
                next-hop 100.1.1.1;
                swap 1000222;
            }
        }
        interface fe-0/0/0.0 {
            static {
                protection-revert-time 0;
            }
        }
        interface fe-0/0/1.0 {
            static {
                protection-revert-time 0;
            }
        }
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}

```

```sh
[edit]
root@Transit# run show route | no-more 

inet.0: 4 destinations, 4 routes (4 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

100.1.1.0/30       *[Direct/0] 00:50:48
                    > via fe-0/0/0.0
100.1.1.2/32       *[Local/0] 00:51:14
                      Local via fe-0/0/0.0
200.1.1.0/30       *[Direct/0] 00:49:56
                    > via fe-0/0/1.0
200.1.1.2/32       *[Local/0] 00:51:13
                      Local via fe-0/0/1.0

mpls.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 00:51:46, metric 1
                      Receive
1                  *[MPLS/0] 00:51:46, metric 1
                      Receive
2                  *[MPLS/0] 00:51:46, metric 1
                      Receive
1000111            *[MPLS/6] 00:05:24, metric 1
                    > to 100.1.1.1 via fe-0/0/0.0, Swap 1000222
1000123            *[MPLS/6] 00:49:56, metric 1
                    > to 200.1.1.1 via fe-0/0/1.0, Swap 1000456

```

```sh
[edit]
root@Transit# run show mpls static-lsp extensive 
Ingress LSPs:
Total 0, displayed 0, Up 0, Down 0

Transit LSPs:
LSPname: path1, Incoming-label: 1000123
  State: Up, Sub State: Traffic via primary but unprotected
  Nexthop: 200.1.1.1 Via fe-0/0/1.0
  LabelOperation: Swap, Outgoing-label: 1000456
  Created: Fri Jan 21 02:41:08 2011
  Bandwidth: 0 bps
  Statistics: Packets 267, Bytes 16180
LSPname: path2, Incoming-label: 1000111
  State: Up, Sub State: Traffic via primary but unprotected
  Nexthop: 100.1.1.1 Via fe-0/0/0.0
  LabelOperation: Swap, Outgoing-label: 1000222
  Created: Fri Jan 21 03:27:30 2011
  Bandwidth: 0 bps
  Statistics: Packets 16, Bytes 1336
Total 2, displayed 2, Up 2, Down 0

Bypass LSPs:
Total 0, displayed 0, Up 0, Down 0

[edit]
```

## Egress

```sh
root@Egress# show | no-more 
## Last changed: 2011-01-20 21:26:30 UTC
version 10.4R1.9;
system {
    host-name Egress;
    root-authentication {
        encrypted-password "$1$zBf6qAyP$Q9Emr5Fjyr464GRNb1tf31"; ## SECRET-DATA
    }
}
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 192.168.1.1/24;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 200.1.1.1/30;
            }
            family mpls;
        }
    }
}
routing-options {
    static {
        route 100.1.1.0/30 next-hop 200.1.1.2;
        route 0.0.0.0/0 next-hop 192.168.1.100;
    }
}
protocols {
    mpls {
        traffic-engineering bgp-igp;
        static-label-switched-path path1 {
            transit 1000456 {
                next-hop 192.168.1.100;
                pop;
            }
        }
        static-label-switched-path path2 {
            ingress {
                next-hop 200.1.1.2;
                to 10.20.30.5;
                push 1000111;
            }
        }
        interface ge-0/0/1.0;
        interface ge-0/0/0.0;
    }
}
security {
    forwarding-options {
        family {
            mpls {
                mode packet-based;
            }
        }
    }
}
```

```sh
[edit]
root@Egress# run show route | no-more 

inet.0: 7 destinations, 7 routes (7 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0.0.0.0/0          *[Static/5] 01:03:48
                    > to 192.168.1.100 via ge-0/0/0.0
10.20.30.5/32      *[MPLS/6/1] 00:20:15, metric 0
                    > to 200.1.1.2 via ge-0/0/1.0, Push 1000111
100.1.1.0/30       *[Static/5] 00:29:47
                    > to 200.1.1.2 via ge-0/0/1.0
192.168.1.0/24     *[Direct/0] 01:03:48
                    > via ge-0/0/0.0
192.168.1.1/32     *[Local/0] 01:03:52
                      Local via ge-0/0/0.0
200.1.1.0/30       *[Direct/0] 01:03:49
                    > via ge-0/0/1.0
200.1.1.1/32       *[Local/0] 01:03:52
                      Local via ge-0/0/1.0

mpls.0: 5 destinations, 5 routes (5 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

0                  *[MPLS/0] 01:05:10, metric 1
                      Receive
1                  *[MPLS/0] 01:05:10, metric 1
                      Receive
2                  *[MPLS/0] 01:05:10, metric 1
                      Receive
1000456            *[MPLS/6] 01:03:48, metric 1
                    > to 192.168.1.100 via ge-0/0/0.0, Pop      
1000456(S=0)       *[MPLS/6] 01:03:48, metric 1
                    > to 192.168.1.100 via ge-0/0/0.0, Pop      


```

```sh
root@Egress# run show mpls static-lsp extensive 
Ingress LSPs:
LSPname: path2, To: 10.20.30.5
  State: Up
  Nexthop: 200.1.1.2 Via ge-0/0/1.0
  LabelOperation: Push, Outgoing-label: 1000111
  Created: Thu Jan 20 21:26:27 2011
  Bandwidth: 0 bps
  Statistics: Packets 19, Bytes 1408
Total 1, displayed 1, Up 1, Down 0

Transit LSPs:
LSPname: path1, Incoming-label: 1000456
  State: Up, Sub State: Traffic via primary but unprotected
  Nexthop: 192.168.1.100 Via ge-0/0/0.0
  LabelOperation: Pop
  Created: Thu Jan 20 20:41:32 2011
  Bandwidth: 0 bps
  Statistics: Packets 273, Bytes 16156
Total 1, displayed 1, Up 1, Down 0

Bypass LSPs:
Total 0, displayed 0, Up 0, Down 0
```



```sh
[edit]
root@Egress# run traceroute 10.20.30.5 
traceroute to 10.20.30.5 (10.20.30.5), 30 hops max, 40 byte packets
 1  200.1.1.2 (200.1.1.2)  10.040 ms  4.406 ms  4.143 ms
     MPLS Label=1000111 CoS=0 TTL=1 S=1
 2  100.1.1.1 (100.1.1.1)  5.038 ms  4.568 ms  4.138 ms
     MPLS Label=1000222 CoS=0 TTL=1 S=1
 3  10.20.30.5 (10.20.30.5)  3.208 ms  2.632 ms  2.565 ms

```
