---

layout: single
title:  "Linux Login Sessions"
date:   2022-06-17 07:59:04 +0530
categories: Linux
tags: Linux
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/linux.png
author:
  name     : "Linux"
  avatar   : "/assets/images/linux.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
# Linux Basics
## whoami
```sh
[lab@desktop ~]$ whoami
lab
```
## w
```sh
[lab@desktop ~]$ w
 22:00:37 up 13 days, 23:14,  2 users,  load average: 0.07, 0.04, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
lab      :0       :0               02Jun22 ?xdm?   2:02m  0.01s /usr/libexec/gdm-x-session --register-session --run-script gnome-session
lab      pts/0    10.210.49.4      21:52    3.00s  7:09m  0.00s w
```
## loginctl
```sh

[lab@desktop ~]$ loginctl
SESSION  UID USER SEAT  TTY 
     11 1000 lab            
      2 1000 lab  seat0 tty2
     c1 1000 lab            

3 sessions listed.
```

```sh
[lab@desktop ~]$ loginctl -h
loginctl [OPTIONS...] {COMMAND} ...

Send control commands to or query the login manager.

  -h --help                Show this help
     --version             Show package version
     --no-pager            Do not pipe output into a pager
     --no-legend           Do not show the headers and footers
     --no-ask-password     Don't prompt for password
  -H --host=[USER@]HOST    Operate on remote host
  -M --machine=CONTAINER   Operate on local container
  -p --property=NAME       Show only properties by this name
  -a --all                 Show all properties, including empty ones
     --value               When showing properties, only print the value
  -l --full                Do not ellipsize output
     --kill-who=WHO        Who to send signal to
  -s --signal=SIGNAL       Which signal to send
  -n --lines=INTEGER       Number of journal entries to show
  -o --output=STRING       Change journal output mode (short, short-precise,
                             short-iso, short-iso-precise, short-full,
                             short-monotonic, short-unix, verbose, export,
                             json, json-pretty, json-sse, cat)
Session Commands:
  list-sessions            List sessions
  session-status [ID...]   Show session status
  show-session [ID...]     Show properties of sessions or the manager
  activate [ID]            Activate a session
  lock-session [ID...]     Screen lock one or more sessions
  unlock-session [ID...]   Screen unlock one or more sessions
  lock-sessions            Screen lock all current sessions
  unlock-sessions          Screen unlock all current sessions
  terminate-session ID...  Terminate one or more sessions
  kill-session ID...       Send signal to processes of a session

User Commands:
  list-users               List users
  user-status [USER...]    Show user status
  show-user [USER...]      Show properties of users or the manager
  enable-linger [USER...]  Enable linger state of one or more users
  disable-linger [USER...] Disable linger state of one or more users
  terminate-user USER...   Terminate all sessions of one or more users
  kill-user USER...        Send signal to processes of a user

Seat Commands:
  list-seats               List seats
  seat-status [NAME...]    Show seat status
  show-seat [NAME...]      Show properties of seats or the manager
  attach NAME DEVICE...    Attach one or more devices to a seat
  flush-devices            Flush all device associations
  terminate-seat NAME...   Terminate all sessions on one or more seats
```
## id
```sh
[lab@desktop ~]$ id -u lab
1000
```
## loginctl list-sessions
```sh

[lab@desktop ~]$ loginctl list-sessions 
SESSION  UID USER SEAT  TTY 
     11 1000 lab            
      2 1000 lab  seat0 tty2
     c1 1000 lab            

3 sessions listed.
```
## loginctl show-session
```sh
[lab@desktop ~]$ loginctl show-session 2
Id=2
User=1000
Name=lab
Timestamp=Thu 2022-06-02 22:47:34 PDT
TimestampMonotonic=82580840
VTNr=2
Seat=seat0
TTY=tty2
Remote=no
Service=gdm-autologin
Scope=session-2.scope
Leader=2792
Audit=2
Type=x11
Class=user
Active=yes
State=active
IdleHint=no
IdleSinceHint=0
IdleSinceHintMonotonic=0
LockedHint=no
```

```sh
[lab@desktop ~]$ loginctl show-session 11
Id=11
User=1000
Name=lab
Timestamp=Thu 2022-06-16 21:52:05 PDT
TimestampMonotonic=1206353144270
VTNr=0
Remote=yes
RemoteHost=10.210.49.4
Service=sshd
Scope=session-11.scope
Leader=334149
Audit=11
Type=tty
Class=user
Active=yes
State=active
IdleHint=no
IdleSinceHint=0
IdleSinceHintMonotonic=0
LockedHint=no
```

```sh
[lab@desktop ~]$ loginctl show-session c1
Id=c1
User=1000
Name=lab
Timestamp=Thu 2022-06-02 22:46:26 PDT
TimestampMonotonic=13857905
VTNr=0
Remote=no
RemoteUser=root
Service=runuser-l
Scope=session-c1.scope
Leader=1173
Audit=0
Type=unspecified
Class=background
Active=yes
State=closing
IdleHint=no
IdleSinceHint=0
IdleSinceHintMonotonic=0
LockedHint=no
[lab@desktop ~]$ 
```

```sh
[lab@desktop ~]$ loginctl 
SESSION  UID USER SEAT  TTY 
     11 1000 lab            
      2 1000 lab  seat0 tty2
     c1 1000 lab            

3 sessions listed.
```
## Common commands
```sh
[lab@desktop ~]$ date
Thu Jun 16 22:07:19 PDT 2022
[lab@desktop ~]$ cal
      June 2022     
Su Mo Tu We Th Fr Sa
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30      
                    
[lab@desktop ~]$ which cal
/usr/bin/cal
[lab@desktop ~]$ which date
/usr/bin/date
```
## loginctl list-users
```sh
[lab@desktop ~]$ loginctl list-users 
 UID USER
1000 lab 

1 users listed.
```
## loginctl session-status
```sh
[lab@desktop ~]$ loginctl session-status 
11 - lab (1000)
           Since: Thu 2022-06-16 21:52:05 PDT; 17min ago
          Leader: 334149 (sshd)
          Remote: 10.210.49.4
         Service: sshd; type tty; class user
           State: active
            Unit: session-11.scope
                  ├─334149 sshd: lab [priv]
                  ├─334151 sshd: lab@pts/0
                  ├─334162 -bash
                  ├─334575 loginctl session-status
                  └─334576 less
```
## sudo
```sh
[lab@desktop ~]$ sudo su -
[sudo] password for lab: 
[root@desktop ~]# w
 22:10:12 up 13 days, 23:24,  2 users,  load average: 0.02, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
lab      :0       :0               02Jun22 ?xdm?   2:02m  0.01s /usr/libexec/gdm-x-session --register-session --run-script gnome-session
lab      pts/0    10.210.49.4      21:52    4.00s  7:09m  0.02s sshd: lab [priv]                                                                     ```

```sh
[root@desktop ~]# whoami 
root
[root@desktop ~]# loginctl
SESSION  UID USER SEAT  TTY 
     11 1000 lab            
      2 1000 lab  seat0 tty2
     c1 1000 lab            

3 sessions listed.
```

```sh
[root@desktop ~]# loginctl list-sessions 
SESSION  UID USER SEAT  TTY 
     11 1000 lab            
      2 1000 lab  seat0 tty2
     c1 1000 lab            

3 sessions listed.
```

## List All Users

```sh
[root@desktop ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
polkitd:x:998:996:User for polkitd:/:/sbin/nologin
geoclue:x:997:994:User for geoclue:/var/lib/geoclue:/sbin/nologin
unbound:x:996:993:Unbound DNS resolver:/etc/unbound:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologin
pipewire:x:995:992:PipeWire System Daemon:/var/run/pipewire:/sbin/nologin
pulse:x:171:171:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
qemu:x:107:107:qemu user:/:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
usbmuxd:x:113:113:usbmuxd user:/:/sbin/nologin
gluster:x:994:988:GlusterFS daemons:/run/gluster:/sbin/nologin
chrony:x:993:987::/var/lib/chrony:/sbin/nologin
libstoragemgmt:x:992:985:daemon account for libstoragemgmt:/var/run/lsm:/sbin/nologin
saslauth:x:991:76:Saslauthd user:/run/saslauthd:/sbin/nologin
setroubleshoot:x:990:984::/var/lib/setroubleshoot:/sbin/nologin
dnsmasq:x:982:982:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/sbin/nologin
radvd:x:75:75:radvd user:/:/sbin/nologin
clevis:x:981:981:Clevis Decryption Framework unprivileged user:/var/cache/clevis:/sbin/nologin
cockpit-ws:x:980:980:User for cockpit-ws:/nonexisting:/sbin/nologin
sssd:x:979:979:User for sssd:/:/sbin/nologin
colord:x:978:978:User for colord:/var/lib/colord:/sbin/nologin
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
gnome-initial-setup:x:977:977::/run/gnome-initial-setup/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
pesign:x:976:976:Group for the pesign signing daemon:/var/run/pesign:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
lab:x:1000:1000:Lab User:/home/lab:/bin/bash
radiusd:x:95:95:radiusd user:/var/lib/radiusd:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
flatpak:x:975:973:User for flatpak system helper:/:/sbin/nologin
cockpit-wsinstance:x:974:972:User for cockpit-ws instances:/nonexisting:/sbin/nologin
rngd:x:973:971:Random Number Generator Daemon:/var/lib/rngd:/sbin/nologin
```