---
layout: single
title:  "Citrix ADC CPX"
date:   2022-05-07 06:59:04 +0530
categories: Kubernetes
tags: Citrix
show_date: true
classes: wide
header:
  teaser: /assets/images/citrix.jpeg
author:
  name     : "Citrix"
  avatar   : "/assets/images/citrix.jpeg"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Citrix ADC CPX Container
In the previous post on [Kubernetes + Citrix ADC](https://pradeepgadde.com/blog/kubernetes/2022/05/07/kubernetes-citrix-adc.html) we have deployed a Citrix Ingress Controller to loadbalance traffic across a sample guestbook app.

In this post, let us look at the Citrix ADC CPX container details.

```sh
pradeep:~$kubectl exec -it cpx-ingress-64464fdb75-9wchv bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Defaulted container "cpx-ingress" out of: cpx-ingress, cic
root@cpx-ingress-64464fdb75-9wchv:/#
```

Out of the two containers in this Pod, it defaults to the `cpx-ingress` container.

Let us see if `cli_script.sh` is available or not
```sh
root@cpx-ingress-64464fdb75-9wchv:/# cli
cli_script.sh  client.ns.js   
```
As we see `cli_script.sh` is available, let us run few commands

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cli_script.sh "show capacity"
exec: show capacity
	Actualbandwidth:  20    MaxVcpuCount:  2        Unit:  Mbps             Maxbandwidth:  40000    Minbandwidth:  20       Instancecount:  0       
Done
root@cpx-ingress-64464fdb75-9wchv:/# 
```
```sh
root@cpx-ingress-64464fdb75-9wchv:/# cli_script.sh "show version"
exec: show version
	Version:  "NetScaler NS13.0: Build 79.64.nc, Date: May 27 2021, 18:41:13   (64-bit)"Mode:  0                
Done
root@cpx-ingress-64464fdb75-9wchv:/#
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cli_script.sh "show ip"
exec: show ip
  	Ipaddress        Traffic Domain  Type             Mode     Arp      Icmp     Vserver  State
  	---------        --------------  ----             ----     ---      ----     -------  ------
1)	172.17.0.3       0               NetScaler IP|VIP Active   Enabled  Enabled  NA       Enabled
2)	192.0.0.1        0               SNIP             Active   Enabled  Enabled  NA       Enabled
Done
root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cli_script.sh "show techsupport"
exec: show techsupport
	Scope:  NODE            Response:  "\nshowtechsupport data collector tool - $Revision$!\nNetScaler version 13.0\nCreating /var/tmp/support ....\nThe NS IP of this box is 172.17.0.3\nThis is not HA configuration\nCopying selected configuration files ....\nsysctl: cannot stat /proc/sys/netscaler/sysid: No such file or directory\nsysctl: cannot stat /proc/sys/netscaler/num_pe_running: No such file or directory\nRunning shell commands ....\ngrep: /var/nslog/dmesg.boot: No such file or directory\nRunning CLI show commands ....\nCollecting ns running configuration....\nCollecting running gslb configuration....\nRunning CLI stat commands ....\nRunning vtysh commands ....\nCopying newnslog files ....\nCopying core files from /var/crash .... \nCopying messages, ns.log, dmesg and other log files ....\nCreating archive ....\n/var/tmp/support/support.tgz  ---- points to ---> /var/tmp/support/collector_P_172.17.0.3_7May2022_13_25.tar.gz\n\n"
Done
root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cat nsconfig/ns.conf 
#NS13.0 Build 79.64
# Last modified by `save config`, Sat May  7 12:22:28 2022
set ns config -IPAddress 172.17.0.3 -netmask 255.255.0.0
enable ns feature LB CS SSL REWRITE RESPONDER AppFlow
enable ns mode L3 USNIP PMTUD
set system user nsroot 44df087d27d81afaac15c3cc44c27338f5fe2adf4075c4dd42b5ef6d8b70de6912405463886c72ede28cbb7d51e85e1c10f1039db85e6c80a64e1c8897b63f2b9 -encrypted
set rsskeytype -rsstype ASYMMETRIC
set lacp -sysPriority 32768 -mac 32:48:11:95:4c:2d
set ns hostName cpx-ingress-64464fdb75-9wchv
set interface 0/1 -haHeartbeat OFF -throughput 0 -bandwidthHigh 0 -bandwidthNormal 0 -ifnum 0/1
set interface 0/2 -speed 1000 -duplex FULL -throughput 0 -bandwidthHigh 0 -bandwidthNormal 0 -ifnum 0/2
set ssl parameter -defaultProfile ENABLED
add ns ip6 fe80::3048:11ff:fe95:4c2d/64 -scope link-local -type NSIP -vlan 1 -vServer DISABLED -mgmtAccess ENABLED -dynamicRouting ENABLED
add ns ip 192.0.0.1 255.255.255.0 -vServer DISABLED -telnet DISABLED -ftp DISABLED -gui DISABLED -ssh DISABLED -snmp DISABLED
set nd6RAvariables -vlan 1
set snmp alarm CLUSTER-BACKPLANE-HB-MISSING -time 86400
set snmp alarm CLUSTER-NODE-HEALTH -time 86400
set snmp alarm CLUSTER-NODE-QUORUM -time 86400
set snmp alarm CLUSTER-VERSION-MISMATCH -time 86400
set snmp alarm COMPACT-FLASH-ERRORS -time 86400
set snmp alarm HA-BAD-SECONDARY-STATE -time 86400
set snmp alarm HA-NO-HEARTBEATS -time 86400
set snmp alarm HA-SYNC-FAILURE -time 86400
set snmp alarm HA-VERSION-MISMATCH -time 86400
set snmp alarm HARD-DISK-DRIVE-ERRORS -time 86400
set snmp alarm PORT-ALLOC-FAILED -time 3600
add policy stringmap CICINFO_default_k8s_HXGRQNQA3N2EDC3QCPBFP5B -comment "Prefix: k8s; Version: 1.18.5; Namespace: default; info used by CIC, do not delete;"
bind policy patset ns_aaa_activesync_useragents Apple-iPhone -index 1 -charset ASCII
bind policy patset ns_aaa_activesync_useragents Apple-iPad -index 2 -charset ASCII
bind policy patset ns_aaa_activesync_useragents SAMSUNG-GT -index 3 -charset ASCII
bind policy patset ns_aaa_activesync_useragents "SAMSUNG GT" -index 4 -charset ASCII
bind policy patset ns_aaa_activesync_useragents AirWatch -index 5 -charset ASCII
bind policy patset ns_aaa_activesync_useragents "TouchDown(MSRPC)" -index 6 -charset ASCII
set ns encryptionParams -method AES256 -keyValue 0702864d94261ab0576b57cea0eb5e351cdb63e942222d580ee056869c74cd8c555910f6a57146472ddebb8661f74dc8c55fffd8a90a035d20d3f34a915cf825709b13906053b35ce1da2b62f3678169 -encrypted -encryptmethod ENCMTHD_3 -kek -suffix 2022_05_07_12_21_35
set ns timeout -anyTcpClient 18000 -anyTcpServer 18000
set cmp parameter -policyType ADVANCED -externalCache YES
add server 10.96.0.10 10.96.0.10
add serviceGroup cpx_default_dns_servicegroup DNS -maxClient 0 -maxReq 0 -cip DISABLED -usip NO -useproxyport NO -cltTimeout 120 -svrTimeout 120 -CKA NO -TCPB NO -CMP NO
add serviceGroup cpx_default_dns_tcp_servicegroup DNS_TCP -maxClient 0 -maxReq 0 -cip DISABLED -usip NO -useproxyport YES -cltTimeout 180 -svrTimeout 360 -CKA NO -TCPB NO -CMP NO
add authentication noAuthAction NO_AUTHN
add ssl certKey ns-server-certificate -cert ns-server.cert -key ns-server.key
set lb parameter -sessionsThreshold 150000
add lb vserver cpx_default_dns_vserver DNS 0.0.0.0 0 -persistenceType NONE -cltTimeout 120
add lb vserver cpx_default_dns_tcp_vserver DNS_TCP 0.0.0.0 0 -persistenceType NONE -cltTimeout 180
add lb vserver k8s_def_trans_server LOGSTREAM 0.0.0.0 0 -persistenceType NONE -cltTimeout 180
set cache parameter -via "NS-CACHE-10.0:   3"
set ns rpcNode 172.17.0.3 -password df1d5631686fc78301026e2ea726f430efb44f87d9f1103ba612642bba0177893f5537ad21e4447cb3a26cb7e80adaf7 -encrypted -encryptmethod ENCMTHD_3 -kek -suffix 2022_05_07_12_21_35 -srcIP 172.17.0.3
set ns rpcNode 192.0.0.1 -password c9cc7427f34796f6301ee47a96d990730c2ae940abc2c9d9bb9192e006c4c513a74210d285f929b9483536803e4990a9 -encrypted -encryptmethod ENCMTHD_3 -kek -suffix 2022_05_07_12_21_35 -srcIP *
set ns rpcNode 192.0.0.2 -password 741320d9ecffb871493045a832b42b3f4cabcbf68cb7f00116697c49c8ef863c6e4c8c0d79f65ad9716faa99241d79ca -encrypted -encryptmethod ENCMTHD_3 -kek -suffix 2022_05_07_12_21_35 -srcIP *
set appflow param -observationPointId 50336172
set cache contentGroup NSFEO -maxResSize 1994752
add appfw policy cpx_import_bypadd "client.ip.src.eq(192.0.0.2)" APPFW_BYPASS
bind appfw global cpx_import_bypadd 1 END -type REQ_OVERRIDE
bind lb vserver cpx_default_dns_vserver cpx_default_dns_servicegroup
bind lb vserver cpx_default_dns_tcp_vserver cpx_default_dns_tcp_servicegroup
add dns nsRec . a.root-servers.net -TTL 3600000
add dns nsRec . b.root-servers.net -TTL 3600000
add dns nsRec . c.root-servers.net -TTL 3600000
add dns nsRec . d.root-servers.net -TTL 3600000
add dns nsRec . e.root-servers.net -TTL 3600000
add dns nsRec . f.root-servers.net -TTL 3600000
add dns nsRec . g.root-servers.net -TTL 3600000
add dns nsRec . h.root-servers.net -TTL 3600000
add dns nsRec . i.root-servers.net -TTL 3600000
add dns nsRec . j.root-servers.net -TTL 3600000
add dns nsRec . k.root-servers.net -TTL 3600000
add dns nsRec . l.root-servers.net -TTL 3600000
add dns nsRec . m.root-servers.net -TTL 3600000
add dns nameServer cpx_default_dns_vserver
add dns nameServer cpx_default_dns_tcp_vserver -type TCP
set ns diameter -identity netscaler.com -realm com
set subscriber gxInterface -pcrfRealm pcrf.com
set ns tcpbufParam -size 128
add dns addRec k.root-servers.net 193.0.14.129 -TTL 3600000
add dns addRec l.root-servers.net 199.7.83.42 -TTL 3600000
add dns addRec a.root-servers.net 198.41.0.4 -TTL 3600000
add dns addRec b.root-servers.net 192.228.79.201 -TTL 3600000
add dns addRec c.root-servers.net 192.33.4.12 -TTL 3600000
add dns addRec d.root-servers.net 199.7.91.13 -TTL 3600000
add dns addRec m.root-servers.net 202.12.27.33 -TTL 3600000
add dns addRec i.root-servers.net 192.36.148.17 -TTL 3600000
add dns addRec j.root-servers.net 192.58.128.30 -TTL 3600000
add dns addRec g.root-servers.net 192.112.36.4 -TTL 3600000
add dns addRec h.root-servers.net 128.63.2.53 -TTL 3600000
add dns addRec e.root-servers.net 192.203.230.10 -TTL 3600000
add dns addRec f.root-servers.net 192.5.5.241 -TTL 3600000
set lb monitor ldns-dns LDNS-DNS -query . -queryType Address -deviation 0 -interval 6 -resptimeout 3 -downTime 20
set lb monitor stasecure CITRIX-STA-SERVICE -deviation 0 -interval 2 MIN -resptimeout 4 -downTime 5
set lb monitor sta CITRIX-STA-SERVICE -deviation 0 -interval 2 MIN -resptimeout 4 -downTime 5
add lb monitor cpx_default_dns_tcp_monitor TCP
bind serviceGroup cpx_default_dns_servicegroup 10.96.0.10 53
bind serviceGroup cpx_default_dns_servicegroup -monitorName cpx_default_dns_tcp_monitor
bind serviceGroup cpx_default_dns_tcp_servicegroup 10.96.0.10 53
bind serviceGroup cpx_default_dns_tcp_servicegroup -monitorName cpx_default_dns_tcp_monitor
add route 0.0.0.0 0.0.0.0 172.17.0.1
set ssl service nsrnatsip-127.0.0.1-5061 -sslProfile ns_default_ssl_profile_frontend
set ssl service nskrpcs-192.0.0.1-3009 -sslProfile ns_default_ssl_profile_frontend
set ssl service nskrpcs-127.0.0.1-3009 -sslProfile ns_default_ssl_profile_frontend
set ssl service nshttps-::1l-9443 -sslProfile ns_default_ssl_profile_frontend
set ssl service nsrpcs-::1l-3008 -sslProfile ns_default_ssl_profile_frontend
set ssl service nshttps-127.0.0.1-9443 -sslProfile ns_default_ssl_profile_frontend
set ssl service nsrpcs-127.0.0.1-3008 -sslProfile ns_default_ssl_profile_frontend
bind audit syslogGlobal -policyName SETSYSLOGPARAMS_ADV_POL -priority 2000000000
bind audit nslogGlobal -policyName SETNSLOGPARAMS_ADV_POL -priority 2000000000
bind tm global -policyName SETTMSESSPARAMS_ADV_POL -priority 65534 -gotoPriorityExpression NEXT
bind ssl profile ns_default_ssl_profile_frontend -eccCurveName P_256
bind ssl profile ns_default_ssl_profile_frontend -eccCurveName P_384
bind ssl profile ns_default_ssl_profile_frontend -eccCurveName P_224
bind ssl profile ns_default_ssl_profile_frontend -eccCurveName P_521
bind ssl profile ns_default_ssl_profile_backend -eccCurveName P_256
bind ssl profile ns_default_ssl_profile_backend -eccCurveName P_384
bind ssl profile ns_default_ssl_profile_backend -eccCurveName P_224
bind ssl profile ns_default_ssl_profile_backend -eccCurveName P_521
bind ssl profile ns_default_ssl_profile_secure_frontend -eccCurveName P_256
bind ssl profile ns_default_ssl_profile_secure_frontend -eccCurveName P_384
bind ssl profile ns_default_ssl_profile_secure_frontend -eccCurveName P_224
bind ssl profile ns_default_ssl_profile_secure_frontend -eccCurveName P_521
bind ssl service nsrnatsip-127.0.0.1-5061 -certkeyName ns-server-certificate
bind ssl service nskrpcs-192.0.0.1-3009 -certkeyName ns-server-certificate
bind ssl service nskrpcs-127.0.0.1-3009 -certkeyName ns-server-certificate
bind ssl service nshttps-::1l-9443 -certkeyName ns-server-certificate
bind ssl service nsrpcs-::1l-3008 -certkeyName ns-server-certificate
bind ssl service nshttps-127.0.0.1-9443 -certkeyName ns-server-certificate
bind ssl service nsrpcs-127.0.0.1-3008 -certkeyName ns-server-certificate
set ip6TunnelParam -srcIP ::
set ptp -state ENABLE
set ns vpxparam -cpuyield YES
set qos parameters -lazybatch 0 -maxqueuedepth 0 -debuglevel 0 -dumpsession 0 -dumpqp 0
set videooptimization parameter -RandomSamplingPercentage 0.00e+00
root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cat nsconfig/ZebOS.conf 
!
log syslog
log record-priority
!
end

root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# cat nsconfig/nsroute.conf 
add route 0 0  172.17.0.1
add route default-172.17.0.1  0.0.0.0 172.17.0.1
add route 172.17.0.0  255.255.0.0 eth0
root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
1root@cpx-ingress-64464fdb75-9wchv:/# cat nsconfig/nsboot.conf 
add ns ip 172.17.0.3 255.255.0.0 -type SNIP
add route 0 0  172.17.0.1
add route default-172.17.0.1  0.0.0.0 172.17.0.1
add route 172.17.0.0  255.255.0.0 eth0
add ssl certkey ns-server-certificate -cert ns-server.cert -key ns-server.key
set tcpprofile nstcp_default_profile mss  1460
set tcpprofile nstcp_internal_apps mss  1460
set ns hostname cpx-ingress-64464fdb75-9wchv
add appfw policy cpx_import_bypadd "client.ip.src.eq(192.0.0.2)" APPFW_BYPASS
bind appfw global cpx_import_bypadd 1 END -type REQ_OVERRIDE
add servicegroup cpx_default_dns_servicegroup dns
add servicegroup cpx_default_dns_tcp_servicegroup dns_tcp
add monitor cpx_default_dns_tcp_monitor tcp
bind servicegroup cpx_default_dns_servicegroup -monitorName cpx_default_dns_tcp_monitor
bind servicegroup cpx_default_dns_tcp_servicegroup -monitorName cpx_default_dns_tcp_monitor
add lb vserver cpx_default_dns_vserver dns
add lb vserver cpx_default_dns_tcp_vserver dns_tcp
bind lb vserver cpx_default_dns_vserver cpx_default_dns_servicegroup
bind lb vserver cpx_default_dns_tcp_vserver cpx_default_dns_tcp_servicegroup
bind servicegroup cpx_default_dns_servicegroup 10.96.0.10 53
bind servicegroup cpx_default_dns_tcp_servicegroup 10.96.0.10 53
add dns nameserver cpx_default_dns_vserver
add dns nameserver cpx_default_dns_tcp_vserver -type TCP
root@cpx-ingress-64464fdb75-9wchv:/# 
```

```sh
root@cpx-ingress-64464fdb75-9wchv:/# exit          
exit
pradeep:~$
```

