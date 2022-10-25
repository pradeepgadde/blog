---
layout: single
title: "BGP-Signaled L2VPN Config"
date:   2022-10-25 04:55:04 +0530
categories: Networking
tags: MPLS
show_date: true
classes: wide
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



# Configuration Example



## CE1-1

```sh
[edit]
pradeep@MX1:CE1-1# show interfaces 
ge-0/0/2 {
    unit 610 {
        vlan-id 610;
        family inet {
            address 10.1.0.1/24;
        }
    }
}
lo0 {
    unit 11 {
        family inet {
            address 10.1.20.1/32;
        }
    }
}

[edit]
pradeep@MX1:CE1-1# 

```

```sh
[edit]
pradeep@MX1:CE1-1# show routing-options 
autonomous-system 65101;
static {
    route 10.1.1.0/24 receive;
    route 10.1.2.0/24 receive;
    route 10.1.3.0/24 receive;
    route 10.1.0.0/24 receive;
}

[edit]
pradeep@MX1:CE1-1# 
```

```sh
[edit]
pradeep@MX1:CE1-1# show routing-options 
autonomous-system 65101;
static {
    route 10.1.1.0/24 receive;
    route 10.1.2.0/24 receive;
    route 10.1.3.0/24 receive;
    route 10.1.0.0/24 receive;
}

[edit]
pradeep@MX1:CE1-1# 
```

```sh
[edit]
pradeep@MX1:CE1-1# show protocols 
ospf {
    area 0.0.0.0 {
        interface ge-0/0/2.610;
        interface lo0.11;
    }
    export statics;
}

[edit]
pradeep@MX1:CE1-1# 

```

## CE1-2

```sh
[edit]
pradeep@MX2:CE1-2# show interfaces 
ge-0/0/1 {
    unit 610 {
        vlan-id 610;
        family inet {
            address 10.1.0.2/24;
        }
    }
}
lo0 {
    unit 12 {
        family inet {
            address 10.1.20.2/32;
        }
    }
}

[edit]
pradeep@MX2:CE1-2# 
```

```sh
[edit]
pradeep@MX2:CE1-2# show routing-options            
autonomous-system 65101;
static {
    route 10.1.4.0/24 receive;
    route 10.1.5.0/24 receive;
    route 10.1.6.0/24 receive;
    route 10.1.7.0/24 receive;
}

[edit]
pradeep@MX2:CE1-2# 
```

```sh
[edit]
pradeep@MX2:CE1-2# show policy-options 
policy-statement export-policy {
    term static {
        from protocol static;
        then accept;
    }
    term direct {
        from protocol direct;
        then accept;
    }
}
policy-statement statics {
    term accept-statics {
        from protocol static;
        then accept;
    }
}

[edit]
pradeep@MX2:CE1-2# 
```

```sh
[edit]
pradeep@MX2:CE1-2# show protocols 
ospf {
    area 0.0.0.0 {
        interface ge-0/0/1.610;
        interface lo0.12;
    }
    export statics;
}

[edit]
pradeep@MX2:CE1-2# 

```

## PE1

```sh
pradeep@MX1:PE-1> show configuration 
interfaces {
    ge-0/0/0 {
        unit 0 {
            family inet {
                address 172.17.23.1/30;
            }
            family mpls;
        }
    }
    ge-0/0/1 {
        unit 0 {
            family inet {
                address 172.17.23.5/30;
            }
            family mpls;
        }
    }
    ge-0/0/4 {
        unit 610 {
            encapsulation vlan-ccc;
            vlan-id 610;
        }
    }
    lo0 {
        unit 1 {
            family inet {
                address 172.17.20.1/32;
            }
        }
    }
}
protocols {
    bgp {                               
        group my-int-group {            
            type internal;              
            local-address 172.17.20.1;  
            family inet {               
                unicast;                
            }                           
            family l2vpn {              
                signaling;              
            }                           
            neighbor 172.17.20.6;       
        }                               
    }                                   
    ldp {                               
        interface ge-0/0/0.0;           
        interface ge-0/0/1.0;           
        interface lo0.1;                
    }                                   
    mpls {                              
        interface ge-0/0/0.0;           
        interface ge-0/0/1.0;           
    }                                   
    ospf {                              
        area 0.0.0.0 {                  
            interface ge-0/0/0.0;       
            interface ge-0/0/1.0;       
            interface lo0.1;            
        }                               
    }                                   
}                                       
routing-instances {                     
    vpn-a {                             
        instance-type l2vpn;            
        protocols {                     
            l2vpn {                     
                site CE1-1 {            
                    interface ge-0/0/4.610;
                    site-identifier 1;  
                }                       
                encapsulation-type ethernet-vlan;
            }                           
        }                               
        interface ge-0/0/4.610;         
        vrf-target target:65512:1;      
    }                                   
}                                       
routing-options {                       
    route-distinguisher-id 172.17.20.1; 
    autonomous-system 65512;            
}                                       
                                        
pradeep@MX1:PE-1>

```

## PE-2

```sh
pradeep@MX2:PE-2> show configuration 
interfaces {
    ge-0/0/2 {
        unit 0 {
            family inet {
                address 172.17.23.26/30;
            }
            family mpls;
        }
    }
    ge-0/0/3 {
        unit 0 {
            family inet {
                address 172.17.23.30/30;
            }
            family mpls;
        }
    }
    ge-0/0/6 {
        unit 610 {
            encapsulation vlan-ccc;
            vlan-id 610;
        }
    }
    lo0 {
        unit 6 {
            family inet {
                address 172.17.20.6/32;
            }
        }
    }
}
protocols {
    bgp {                               
        group my-int-group {            
            type internal;
            local-address 172.17.20.6;
            family inet {
                unicast;
            }
            family l2vpn {
                signaling;
            }
            neighbor 172.17.20.1;
        }
    }
    ldp {
        interface ge-0/0/2.0;
        interface ge-0/0/3.0;
        interface lo0.6;
    }
    mpls {
        interface ge-0/0/2.0;
        interface ge-0/0/3.0;
    }
    ospf {
        area 0.0.0.0 {
            interface ge-0/0/2.0;
            interface ge-0/0/3.0;
            interface lo0.6;
        }
     }
}
routing-instances {
    vpn-a {
        instance-type l2vpn;
        protocols {                     
            l2vpn {
                site CE1-2 {
                    interface ge-0/0/6.610;
                    site-identifier 2;
                }
                encapsulation-type ethernet-vlan;
            }
        }
        interface ge-0/0/6.610;
        vrf-target target:65512:1;
    }
}
routing-options {
    route-distinguisher-id 172.17.20.6;
    autonomous-system 65512;
}

pradeep@MX2:PE-2> 

```



## PE-1 Verification

```sh
pradeep@MX1:PE-1> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       0          0          0          0          0          0
bgp.l2vpn.0          
                       1          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.17.20.6           65512         95         96       0       0       39:30 Establ
  inet.0: 0/0/0/0
  bgp.l2vpn.0: 1/1/1/0
  vpn-a.l2vpn.0: 1/1/1/0

pradeep@MX1:PE-1> 
```

```sh
pradeep@MX1:PE-1> show l2vpn connections 
Layer-2 VPN connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NC -- interface encapsulation not CCC/TCC/VPLS
EM -- encapsulation mismatch     WE -- interface and instance encaps not same
VC-Dn -- Virtual circuit down    NP -- interface hardware not present 
CM -- control-word mismatch      -> -- only outbound connection is up
CN -- circuit not provisioned    <- -- only inbound connection is up
OR -- out of range               Up -- operational
OL -- no outgoing label          Dn -- down                      
LD -- local site signaled down   CF -- call admission control failure      
RD -- remote site signaled down  SC -- local and remote site ID collision
LN -- local site not designated  LM -- local site ID not minimum designated
RN -- remote site not designated RM -- remote site ID not minimum designated
XX -- unknown connection status  IL -- no incoming label
MM -- MTU mismatch               MI -- Mesh-Group ID not available
BK -- Backup connection         ST -- Standby connection
PF -- Profile parse failure      PB -- Profile busy
RS -- remote site standby SN -- Static Neighbor
LB -- Local site not best-site   RB -- Remote site not best-site
VM -- VLAN ID mismatch           HS -- Hot-standby Connection

Legend for interface status 
Up -- operational           
Dn -- down

Instance: vpn-a
Edge protection: Not-Primary
  Local site: CE1-1 (1)
    connection-site           Type  St     Time last up          # Up trans
    2                         rmt   Up     Oct 25 16:37:17 2022           1
      Remote PE: 172.17.20.6, Negotiated control-word: Yes (Null)
      Incoming label: 22, Outgoing label: 21
      Local interface: ge-0/0/4.610, Status: Up, Encapsulation: VLAN
      Flow Label Transmit: No, Flow Label Receive: No

pradeep@MX1:PE-1> 
```

```sh
pradeep@MX1:PE-1> show route table bgp.l2vpn.0 

bgp.l2vpn.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.17.20.6:65534:2:1/96                
                   *[BGP/170] 00:21:45, localpref 100, from 172.17.20.6
                      AS path: I, validation-state: unverified
                    >  to 172.17.23.2 via ge-0/0/0.0, Push 30
                       to 172.17.23.6 via ge-0/0/1.0, Push 31

pradeep@MX1:PE-1> 
```

```sh
vpn-a.l2vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.17.20.1:65534:1:1/96                
                   *[L2VPN/170/-101] 00:24:13, metric2 1
                       Indirect
172.17.20.6:65534:2:1/96                
                   *[BGP/170] 00:22:09, localpref 100, from 172.17.20.6
                      AS path: I, validation-state: unverified
                    >  to 172.17.23.2 via ge-0/0/0.0, Push 30
                       to 172.17.23.6 via ge-0/0/1.0, Push 31

pradeep@MX1:PE-1> 
```

## PE-2 Verification

```sh
pradeep@MX2:PE-2> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 1 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       0          0          0          0          0          0
bgp.l2vpn.0          
                       1          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
172.17.20.1           65512        106        104       0       0       44:00 Establ
  inet.0: 0/0/0/0
  bgp.l2vpn.0: 1/1/1/0
  vpn-a.l2vpn.0: 1/1/1/0

pradeep@MX2:PE-2> 
```

```sh
pradeep@MX2:PE-2> show route table bgp.l2vpn.0 

bgp.l2vpn.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.17.20.1:65534:1:1/96                
                   *[BGP/170] 00:23:29, localpref 100, from 172.17.20.1
                      AS path: I, validation-state: unverified
                    >  to 172.17.23.25 via ge-0/0/2.0, Push 35
                       to 172.17.23.29 via ge-0/0/3.0, Push 34

pradeep@MX2:PE-2>
```

```sh
pradeep@MX2:PE-2> show route table vpn-a.l2vpn.0 

vpn-a.l2vpn.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.17.20.1:65534:1:1/96                
                   *[BGP/170] 00:23:59, localpref 100, from 172.17.20.1
                      AS path: I, validation-state: unverified
                    >  to 172.17.23.25 via ge-0/0/2.0, Push 35
                       to 172.17.23.29 via ge-0/0/3.0, Push 34
172.17.20.6:65534:2:1/96                
                   *[L2VPN/170/-101] 00:23:59, metric2 1
                       Indirect

pradeep@MX2:PE-2> 
```

```sh
pradeep@MX2:PE-2> show l2vpn connections 
Layer-2 VPN connections:

Legend for connection status (St)   
EI -- encapsulation invalid      NC -- interface encapsulation not CCC/TCC/VPLS
EM -- encapsulation mismatch     WE -- interface and instance encaps not same
VC-Dn -- Virtual circuit down    NP -- interface hardware not present 
CM -- control-word mismatch      -> -- only outbound connection is up
CN -- circuit not provisioned    <- -- only inbound connection is up
OR -- out of range               Up -- operational
OL -- no outgoing label          Dn -- down                      
LD -- local site signaled down   CF -- call admission control failure      
RD -- remote site signaled down  SC -- local and remote site ID collision
LN -- local site not designated  LM -- local site ID not minimum designated
RN -- remote site not designated RM -- remote site ID not minimum designated
XX -- unknown connection status  IL -- no incoming label
MM -- MTU mismatch               MI -- Mesh-Group ID not available
BK -- Backup connection         ST -- Standby connection
PF -- Profile parse failure      PB -- Profile busy
RS -- remote site standby SN -- Static Neighbor
LB -- Local site not best-site   RB -- Remote site not best-site
VM -- VLAN ID mismatch           HS -- Hot-standby Connection

Legend for interface status 
Up -- operational           
Dn -- down

Instance: vpn-a
Edge protection: Not-Primary
  Local site: CE1-2 (2)
    connection-site           Type  St     Time last up          # Up trans
    1                         rmt   Up     Oct 25 16:37:17 2022           1
      Remote PE: 172.17.20.1, Negotiated control-word: Yes (Null)
      Incoming label: 21, Outgoing label: 22
      Local interface: ge-0/0/6.610, Status: Up, Encapsulation: VLAN
      Flow Label Transmit: No, Flow Label Receive: No
                                        
pradeep@MX2:PE-2> 
```

## CE1-1 Verification

```sh
pradeep@MX1:CE1-1> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.1.0.2         ge-0/0/2.610           Full            10.1.20.2        128    34
```

```sh
pradeep@MX1:CE1-1> show route protocol ospf   

inet.0: 12 destinations, 13 routes (12 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.1.4.0/24        *[OSPF/150] 00:21:19, metric 0, tag 0
                    >  to 10.1.0.2 via ge-0/0/2.610
10.1.5.0/24        *[OSPF/150] 00:21:19, metric 0, tag 0
                    >  to 10.1.0.2 via ge-0/0/2.610
10.1.6.0/24        *[OSPF/150] 00:21:19, metric 0, tag 0
                    >  to 10.1.0.2 via ge-0/0/2.610
10.1.7.0/24        *[OSPF/150] 00:21:19, metric 0, tag 0
                    >  to 10.1.0.2 via ge-0/0/2.610
10.1.20.2/32       *[OSPF/10] 00:21:19, metric 1
                    >  to 10.1.0.2 via ge-0/0/2.610
224.0.0.5/32       *[OSPF/10] 00:22:56, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

pradeep@MX1:CE1-1> 

```

```sh
pradeep@MX1:CE1-1> ping 10.1.20.2 source 10.1.20.1 
PING 10.1.20.2 (10.1.20.2): 56 data bytes
64 bytes from 10.1.20.2: icmp_seq=0 ttl=64 time=9.367 ms
64 bytes from 10.1.20.2: icmp_seq=1 ttl=64 time=5.596 ms
64 bytes from 10.1.20.2: icmp_seq=2 ttl=64 time=5.007 ms
^C
--- 10.1.20.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 5.007/6.657/9.367/1.932 ms

pradeep@MX1:CE1-1> 
```

## CE1-2 Verification

```sh
pradeep@MX2:CE1-2> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.1.0.1         ge-0/0/1.610           Full            10.1.20.1        128    39
```

```sh
pradeep@MX2:CE1-2> show route protocol ospf    

inet.0: 12 destinations, 12 routes (12 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.1.1.0/24        *[OSPF/150] 00:25:59, metric 0, tag 0
                    >  to 10.1.0.1 via ge-0/0/1.610
10.1.2.0/24        *[OSPF/150] 00:25:59, metric 0, tag 0
                    >  to 10.1.0.1 via ge-0/0/1.610
10.1.3.0/24        *[OSPF/150] 00:25:59, metric 0, tag 0
                    >  to 10.1.0.1 via ge-0/0/1.610
10.1.20.1/32       *[OSPF/10] 00:25:59, metric 1
                    >  to 10.1.0.1 via ge-0/0/1.610
224.0.0.5/32       *[OSPF/10] 00:26:19, metric 1
                       MultiRecv

inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)

pradeep@MX2:CE1-2> 
```

```sh
pradeep@MX2:CE1-2> ping 10.1.20.1 source 10.1.20.2 count 3 
PING 10.1.20.1 (10.1.20.1): 56 data bytes
64 bytes from 10.1.20.1: icmp_seq=0 ttl=64 time=6.615 ms
64 bytes from 10.1.20.1: icmp_seq=1 ttl=64 time=5.672 ms
64 bytes from 10.1.20.1: icmp_seq=2 ttl=64 time=4.860 ms

--- 10.1.20.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 4.860/5.716/6.615/0.717 ms

pradeep@MX2:CE1-2> 
```

So far we established a point-to-point BGP Layer 2 virtual private network (VPN) using LDP signaled label switched paths (LSP) between provider edge (PE) routers. Once the virtual LAN (VLAN)-based Layer 2 VPN is operational, we configured the customer edge (CE) routers to run OSPF routing protocol and advertise their static route and loopback address blocks. Because this is a BGP Layer 2 VPN, the PE routers will not interact with the routing protocols used on the CE routers. 



