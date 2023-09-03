---
layout: single
title: "Juniper Networks 128T Session Smart Router PCLI"
date:   2023-09-03 01:55:04 +0530
categories: Networking
tags: SSR
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

## Session Smart Router PCLI

Juniper has acquired 128Technologies few years ago. In this post, let’s look into 128T’s Session Smart Networking Platform CLI. 

This CLI (called PCLI) is different from Junos OS CLI.

The default credentials are `root` and `128tRoutes`.

```sh
CentOS Linux 7 (Core)
Kernel 4.18.0-425.10.1.el8_7.x86_64 on an x86_64

sn40e72f87-5c86-4759-ba60-3e94d8d79a30 login: root
Password: 
Last login: Tue Aug 29 08:02:00 on ttyS0
+---------------------------------------+
|                                       |
|    Welcome to:                        |
|                                       |
|     | .   . ,---. . ,---. ,---. ,--.  |
|     | |   | |   | | |---' |---' |     |
|     | `---' '   ' ' '     `---' '     |
|  ---'                                 |
|        __ ___       __   __       __  |
|  |\ | |_   |  |  | /  \ |__) |_/ (_   |
|  | \| |__  |  |/\| \__/ | \  | \ __)  |
|                                       |
| Session Smart Networking Platform ... |
+---------------------------------------+
[root@sn40e72f87-5c86-4759-ba60-3e94d8d79a30 ~]# 
```

PCLI is based on CentOS Linux 7 and python based.

```
[root@sn40e72f87-5c86-4759-ba60-3e94d8d79a30 ~]# su admin
Starting the PCLI...
admin@node.router#
```

We can use `?` anytime, like Junos. This provides context-sensitive help. 

```admin@node.router#
admin@node.router#
adopt           Assign the current router to a Mist organization.
clear           <app-id> | <arp> | <bgp> | <history>
commit          Commit the candidate config as the new running config.
compare         <config>
configure       SSR configuration.
create          <capture-filter> | <certificate> | <config> | <session-capture>
                | <user>
delete          <capture-filter> | <certificate> | <config> | <flows> |
                <session-capture> | <sessions> | <system> | <user>
edit            <prompt> | <user>
export          <config>
import          <certificate> | <config> | <iso>
lookup          <application>
migrate         Migrate a SSR router to a new conductor
ping            Send an ICMP request through a network interface.
quit            Quit the PCLI.
refresh         <dns>
release         <dhcp>
repeat          Repeat any command multiple times.
replace         <config>
request         <idp> | <system>
restore         <config> | <prompt> | <system> | <users>
rotate          <log>
save            <runtime-stats> | <tech-support-info>
search          Search for any PCLI command or configuration data from the
                current location in the command tree.
send            <command>
service-ping    Ping that uses a tenant or service to make an ICMP request.
set             <config> | <dns> | <log> | <password> | <provisional-status> |
                <software> | <system> | <time>
shell           Execute a Unix shell command.
show            <alarms> | <app-id> | <application> | <arp> | <bfd> | <bgp> |
                <capacity> | <capture-filters> | <certificate> | <config> |
                <device-interface> | <dhcp> | <dns> | <domain-categories> |
                <domain-names> | <entitlement> | <events> | <fib> | <history> |
                <idp> | <load-balancer> | <lte> | <mist> | <network-interface> |
                <ntp> | <ospf> | <peers> | <platform> | <plugins> | <rib> |
                <roles> | <security> | <service> | <service-path> | <session-
                captures> | <sessions> | <stats> | <step> | <system> | <tenant>
                | <top> | <udp-transform> | <user> | <vrf>
time            Force another command to display its execution time.
trace           Trace the HTTP API calls of another command, for troubleshooting
                purposes.
traceroute      Print the route packets take to network host.
unrelease       <mist>
validate        Validate the candidate config.
where           Display the current location in the CLI hierarchy.
write           <log>
admin@node.router#
```
Use question mark anytime, for example, after `show system`

```
admin@node.router# show system
usage: system [node <node>]

Display detailed system state.

The 'show system' subcommand displays overall system health for the nodes that
comprise your SSR router. It includes the state of the node ("starting" is
displayed when the node is in the process of starting up and is not yet ready
for handling traffic, "running" means the node is active, "offline" means the
node is configured but not currently present), its role, software version, and
uptime.

keyword arguments:
node    The node for which to display system state

subcommands:
connectivity    Display inter-node connection statuses.
processes       Display a table summarizing the statuses of processes.
registry        Shows registered services from the system services coordinator
                for the specified process, node or router.
services        Display a table summarizing statuses of SSR systemd services.
software        <available> | <download> | <upgrade>
version         Show system version information.

see also:
show alarms    Display currently active or shelved alarms
admin@node.router# show system
```

Use `show system version` to retrieve software version.

```
admin@node.router# show system version
Sun 2023-09-03 14:03:52 UTC
Retrieving system version...

======== ====== ========= ====================== ==================
 Router   Node   Version   Build Date             Package
======== ====== ========= ====================== ==================
 router   node   6.1.2     2023-05-04T17:55:49Z   128T-6.1.2-7.el7

Completed in 1.20 seconds
admin@node.router#
```

Double Tab provides a list of options like Linux.

```sh
admin@node.router#
adopt        delete       ping         request      service-ping traceroute   
clear        edit         quit         restore      set          unrelease    
commit       export       refresh      rotate       shell        validate     
compare      import       release      save         show         where        
configure    lookup       repeat       search       time         write        
create       migrate      replace      send         trace        
admin@node.router#
```

### configure

Enter configuration mode

```
admin@node.router# configure
admin@node.router (configure)#
```

User `?` in the configuration mode

```
admin@node.router (configure)#
Configuration Attributes
------------------------
authority    Authority configuration is the top-most level in the 128T router
             configuration hierarchy.

Configuration Commands
----------------------
commit       Commit the candidate config as the new running config.
show         Show configuration data for 'config'
validate     Validate the candidate config.

General Commands
----------------
do           Execute a top-level command.
exit         Exit this menu (You can also press Ctrl+D).
quit         Quit the PCLI.
repeat       Repeat any command multiple times.
replace      <config>
search       Search for any PCLI command or configuration data from the current
             location in the command tree.
time         Force another command to display its execution time.
top          Returns to the root menu.
trace        Trace the HTTP API calls of another command, for troubleshooting
             purposes.
up           Exit this menu and navigate up the hierarchy the given number of
             levels.
where        Display the current location in the CLI hierarchy.
admin@node.router (configure)#
```
### authority

Configure Authority settings

```
admin@node.router (configure)# authority
Configuration Attributes
------------------------
access-management                   Role Based Access Control (RBAC)
                                    configuration.
asset-connection-resiliency         Configure Asset Connection Resiliency
bgp-service-generation              Configure Bgp Service Generation
cli-messages                        Configure Cli Messages
client-certificate                  The client-certificate configuration
                                    contains client certificate content.
conductor-address                   IP address or FQDN of the conductor
currency                            Local monetary unit.
district                            Districts in the authority.
dscp-map                            Configure Dscp Map
dynamic-hostname                    Hostname format for interfaces with dynamic
                                    addresses. It is a template with subsitution
                                    variables used to generate a unique hostname
                                    corresponding to Network Interfaces that
                                    have dynamically learned IP addresses. Uses
                                    the following substitution variables:
                                    {interface-id} for Network Interface Global
                                    Identifier {router-name} for Router Name
                                    {authority-name} for Authority Name For
                                    example, 'interface-{interface-id}.{router-
                                    name}.{authority-name}'.
forward-error-correction-profile    A profile for Forward Error Correection
                                    parameters, describing how often to send
                                    parity packets.
ipfix-collector                     Configuration for IPFIX record export.
ldap-server                         LDAP Servers against which to authenticate
                                    user credentials.
metrics-profile                     A collection of metrics
mist-wan-assurance                  Mist WAN Assurance configuration
name                                The identifier for the Authority.
password-policy                     Password policy for user's passwords.
pcli                                Configure the PCLI.
performance-monitoring-profile      A performance monitoring profile used to
                                    determine how often packets should be
                                    marked.
radius-server                       Radius Servers against which to authenticate
                                    user credentials.
rekey-interval                      Hours between security key regeneration.
                                    Recommended value 24 hours.
remote-login                        Configure Remote Login
resource-group                      Collect objects into a management group.
router                              The router configuration element serves as a
                                    container for holding the nodes of a single
                                    deployed router, along with their policies.
routing                             authority level routing configuration
security                            The security elements represent security
                                    policies for governing how and when the 128T
                                    router encrypts and/or authenticates
                                    packets.
service                             The service configuration is where you
                                    define the services that reside within the
                                    authority's tenants as well as the policies
                                    to apply to those services.
service-class                       Defines the association between DSCP value
                                    and a priority queue.
service-policy                      A service policy, which defines parameters
                                    applied to services that reference the
                                    policy
session-record-profile              A profile to describe how to collect session
                                    records.
session-recovery-detection          Configure Session Recovery Detection
session-type                        Type of session classification based on
                                    protocol and port, and associates it with a
                                    default class of service.
step-repo                           List of Service and Topology Exchange
                                    Protocol repositories.
tenant                              A customer or user group within the
                                    Authority.
traffic-profile                     A set of minimum guaranteed bandwidths, one
                                    for each traffic priority
trusted-ca-certificate              The trusted-ca-certificate configuration
                                    contains CA certificate content.
web-messages                        Configure Web Messages
web-theme                           Configure Web Theme

Configuration Commands
----------------------
clone                               Clone a list item
delete                              Delete configuration data
show                                Show configuration data for 'authority'
admin@node.router (configure)#
```
Find out about authority name
```
admin@node.router (configure)# authority name
usage: name [<name-id>]

The identifier for the Authority.

positional arguments:
name-id    The value to set for this field

name-id (string) (required):
----------------------------
A string identifier which only uses alphanumerics, underscores, or dashes, and
cannot exceed 63 characters.

Length: 0-63


admin@node.router (configure)#
```
Configure Authority name
```
admin@node.router (configure)# authority name myauthority
*admin@node.router (configure)#
```

Configure router name

```
*admin@node.router (configure)# authority router myrouter
*admin@node.router (router[name=myrouter])#
```

Use `where` to know about the current location within the PCLI hierarchy

```
*admin@node.router (router[name=myrouter])# where
configure authority router myrouter
*admin@node.router (router[name=myrouter])#
```

### top

Use `top` to go to the top

```
*admin@node.router (router[name=myrouter])# top
*admin@node.router# where

*admin@node.router#
```

Use `show config`

```
*admin@node.router# show config
candidate    Display candidate configuration data
exports      Display configuration exports.
running      Display running configuration data
version      Display running configuration version.
*admin@node.router# show config
```

### running config

Running configuration

```
*admin@node.router# show config running

config

    authority

        router  router
            name  router

            node  node
                name              node

                device-interface  bootstrapper
                    name               bootstrapper

                    network-interface  bootstrapper-intf
                        name       bootstrapper-intf
                        global-id  1
                    exit
                exit

                device-interface  ge-0-0
                    name               ge-0-0

                    network-interface  ge-0-0-intf
                        name       ge-0-0-intf
                        global-id  2
                    exit
                exit

                device-interface  ge-0-1
                    name               ge-0-1

                    network-interface  ge-0-1-intf
                        name       ge-0-1-intf
                        global-id  3
                    exit
                exit

                device-interface  ge-0-2
                    name               ge-0-2

                    network-interface  ge-0-2-intf
                        name       ge-0-2-intf
                        global-id  4
                    exit
                exit

                device-interface  ge-0-3
                    name               ge-0-3

                    network-interface  ge-0-3-intf
                        name       ge-0-3-intf
                        global-id  5
                    exit
                exit

                device-interface  ge-0-4
                    name               ge-0-4

                    network-interface  ge-0-4-intf
                        name       ge-0-4-intf
                        global-id  6
                    exit
                exit

                device-interface  ge-0-5
                    name               ge-0-5

                    network-interface  ge-0-5-intf
                        name       ge-0-5-intf
                        global-id  7
                    exit
                exit

                device-interface  ge-0-6
                    name               ge-0-6

                    network-interface  ge-0-6-intf
                        name       ge-0-6-intf
                        global-id  8
                    exit
                exit

                device-interface  ge-0-7
                    name               ge-0-7

                    network-interface  ge-0-7-intf
                        name       ge-0-7-intf
                        global-id  9
                    exit
                exit
            exit
        exit
    exit
exit

*admin@node.router#
```

### flat

Running config in `flat` format

```
*admin@node.router# show config running flat



config authority router router name  router

config authority router router node node name              node

config authority router router node node device-interface bootstrapper name               bootstrapper

config authority router router node node device-interface bootstrapper network-interface bootstrapper-intf name       bootstrapper-intf
config authority router router node node device-interface bootstrapper network-interface bootstrapper-intf global-id  1

config authority router router node node device-interface ge-0-0 name               ge-0-0

config authority router router node node device-interface ge-0-0 network-interface ge-0-0-intf name       ge-0-0-intf
config authority router router node node device-interface ge-0-0 network-interface ge-0-0-intf global-id  2

config authority router router node node device-interface ge-0-1 name               ge-0-1

config authority router router node node device-interface ge-0-1 network-interface ge-0-1-intf name       ge-0-1-intf
config authority router router node node device-interface ge-0-1 network-interface ge-0-1-intf global-id  3

config authority router router node node device-interface ge-0-2 name               ge-0-2

config authority router router node node device-interface ge-0-2 network-interface ge-0-2-intf name       ge-0-2-intf
config authority router router node node device-interface ge-0-2 network-interface ge-0-2-intf global-id  4

config authority router router node node device-interface ge-0-3 name               ge-0-3

config authority router router node node device-interface ge-0-3 network-interface ge-0-3-intf name       ge-0-3-intf
config authority router router node node device-interface ge-0-3 network-interface ge-0-3-intf global-id  5

config authority router router node node device-interface ge-0-4 name               ge-0-4

config authority router router node node device-interface ge-0-4 network-interface ge-0-4-intf name       ge-0-4-intf
config authority router router node node device-interface ge-0-4 network-interface ge-0-4-intf global-id  6

config authority router router node node device-interface ge-0-5 name               ge-0-5

config authority router router node node device-interface ge-0-5 network-interface ge-0-5-intf name       ge-0-5-intf
config authority router router node node device-interface ge-0-5 network-interface ge-0-5-intf global-id  7

config authority router router node node device-interface ge-0-6 name               ge-0-6

config authority router router node node device-interface ge-0-6 network-interface ge-0-6-intf name       ge-0-6-intf
config authority router router node node device-interface ge-0-6 network-interface ge-0-6-intf global-id  8

config authority router router node node device-interface ge-0-7 name               ge-0-7

config authority router router node node device-interface ge-0-7 network-interface ge-0-7-intf name       ge-0-7-intf
config authority router router node node device-interface ge-0-7 network-interface ge-0-7-intf global-id  9

*admin@node.router#
```

### commit

Use `commit` to apply changes

```
*admin@node.router# commit
Are you sure you want to commit the candidate config? [y/N]: y
Validating, then committing...
% Warning:
1. The field 'name' cannot be created while the system is running. To apply the
configuration,  must be restarted after committing this change.

    config
        authority
            router myrouter
                name

2. The field 'name' cannot be created while the system is running. To apply the
configuration,  must be restarted after committing this change.

    config
        authority
            router name
                name

% Error: Failed to commit:
1. inter-node-security is required

    config
        authority
            router name
                inter-node-security

2. inter-node-security is required

    config
        authority
            router myrouter
                inter-node-security

*admin@node.router#
```

### compare

Use `compare`

```
*admin@node.router# compare
config    Display the differences between two configurations.
*admin@node.router# compare config
usage: config [<old>] [<new>]

Display the differences between two configurations.

The 'compare' command presents a list of differences between the two
configurations specified as arguments on the command line. The one listed first
influences the output in a very important way: the SSR router will return a list
of configuration commands that will cause the configuration to be listed 'first'
to be brought to parity with the one listed 'second'. (Note: since the only
editable configuration is the "candidate" configuration, the changes outlined by
the 'compare' command cannot be directly applied to the "running"
configuration.)

positional arguments:
old    The original configuration against which differences should be computed
       (default: running)
new    The updated configuration for which differences should be computed

see also:
create config autogenerated       Run configuration generation.
delete config exported            Delete an exported configuration from disk.
export config                     Export a copy of the current running or
                                  candidate config.
import config                     Import a configuration as the candidate
                                  config.
restore config factory-default    Restore the candidate config to the factory
                                  defaults.
restore config running            Discard uncommitted changes from the candidate
                                  config.
set config encryption             Sets the encryption key for the SSR
                                  configuration
show config exports               Display configuration exports.
show config version               Display running configuration version.
show stats config                 Metrics pertaining to the get-config RPC
*admin@node.router#
```

```
*admin@node.router# compare config

config

    authority
        name    myauthority

        router  name
            name  name
        exit

        router  myrouter
            name  myrouter
        exit
    exit
exit

*admin@node.router#
```

### restore

Use `restore config running`

```
*admin@node.router# restore config running
Are you sure you want to discard uncommitted changes from the candidate config f
or user admin? [y/N]: y
Discarding uncommitted changes...
Candidate configuration changes successfully discarded for admin
admin@node.router#
```

### do

Use `do` to run any commands from `configure` mode

```
admin@node.router (configure)# do
adopt           Assign the current router to a Mist organization.
clear           <app-id> | <arp> | <bgp> | <history>
compare         <config>
configure       SSR configuration.
create          <capture-filter> | <certificate> | <config> | <session-capture>
                | <user>
delete          <capture-filter> | <certificate> | <config> | <flows> |
                <session-capture> | <sessions> | <system> | <user>
edit            <prompt> | <user>
export          <config>
import          <certificate> | <config> | <iso>
lookup          <application>
migrate         Migrate a SSR router to a new conductor
ping            Send an ICMP request through a network interface.
refresh         <dns>
release         <dhcp>
repeat          Repeat any command multiple times.
replace         <config>
request         <idp> | <system>
restore         <config> | <prompt> | <system> | <users>
rotate          <log>
save            <runtime-stats> | <tech-support-info>
search          Search for any PCLI command or configuration data from the
                current location in the command tree.
send            <command>
service-ping    Ping that uses a tenant or service to make an ICMP request.
set             <config> | <dns> | <log> | <password> | <provisional-status> |
                <software> | <system> | <time>
shell           Execute a Unix shell command.
show            <alarms> | <app-id> | <application> | <arp> | <bfd> | <bgp> |
                <capacity> | <capture-filters> | <certificate> | <config> |
                <device-interface> | <dhcp> | <dns> | <domain-categories> |
                <domain-names> | <entitlement> | <events> | <fib> | <history> |
                <idp> | <load-balancer> | <lte> | <mist> | <network-interface> |
                <ntp> | <ospf> | <peers> | <platform> | <plugins> | <rib> |
                <roles> | <security> | <service> | <service-path> | <session-
                captures> | <sessions> | <stats> | <step> | <system> | <tenant>
                | <top> | <udp-transform> | <user> | <vrf>
time            Force another command to display its execution time.
trace           Trace the HTTP API calls of another command, for troubleshooting
                purposes.
traceroute      Print the route packets take to network host.
unrelease       <mist>
validate        Validate the candidate config.
write           <log>
admin@node.router (configure)# do
```

### repeat

Use `repeat` to run any command repeatedly. For example, `show alarms`. By default, it runs every 2 seconds.

```
admin@node.router# repeat show alarms
Running "show alarms" every 2 seconds

Sun 2023-09-03 14:32:59 UTC
Retrieving alarms...

========= ===================== ========== ======== ===========
====================================
 ID        Time                  Severity   Source   Category    Message
========= ===================== ========== ======== ===========
====================================
 node:4    2023-09-03 14:27:15   MAJOR               SYSTEM      No active NTP
 server
 node:5    2023-09-03 14:27:23   CRITICAL            INTERFACE   Intf ge-0-0 (2)
 operationally down
 node:6    2023-09-03 14:27:26   CRITICAL            INTERFACE   Intf ge-0-1 (3)
 operationally down
 node:8    2023-09-03 14:27:29   CRITICAL            INTERFACE   Intf ge-0-2 (4)
 operationally down
 node:9    2023-09-03 14:27:32   CRITICAL            INTERFACE   Intf ge-0-3 (5)
 operationally down
 node:10   2023-09-03 14:27:35   CRITICAL            INTERFACE   Intf ge-0-4 (6)
 operationally down
 node:11   2023-09-03 14:27:38   CRITICAL            INTERFACE   Intf ge-0-5 (7)
 operationally down
 node:12   2023-09-03 14:27:40   CRITICAL            INTERFACE   Intf ge-0-6 (8)
 operationally down
 node:13   2023-09-03 14:27:42   CRITICAL            INTERFACE   Intf ge-0-7 (9)
 operationally down

There are no shelved alarms
Completed in 0.07 seconds

Running "show alarms" every 2 seconds

Sun 2023-09-03 14:33:01 UTC
Retrieving alarms...

========= ===================== ========== ======== ===========
====================================
 ID        Time                  Severity   Source   Category    Message
========= ===================== ========== ======== ===========
====================================
 node:4    2023-09-03 14:27:15   MAJOR               SYSTEM      No active NTP
 server
 node:5    2023-09-03 14:27:23   CRITICAL            INTERFACE   Intf ge-0-0 (2)
 operationally down
 node:6    2023-09-03 14:27:26   CRITICAL            INTERFACE   Intf ge-0-1 (3)
 operationally down
 node:8    2023-09-03 14:27:29   CRITICAL            INTERFACE   Intf ge-0-2 (4)
 operationally down
 node:9    2023-09-03 14:27:32   CRITICAL            INTERFACE   Intf ge-0-3 (5)
 operationally down
 node:10   2023-09-03 14:27:35   CRITICAL            INTERFACE   Intf ge-0-4 (6)
 operationally down
 node:11   2023-09-03 14:27:38   CRITICAL            INTERFACE   Intf ge-0-5 (7)
 operationally down
 node:12   2023-09-03 14:27:40   CRITICAL            INTERFACE   Intf ge-0-6 (8)
 operationally down
 node:13   2023-09-03 14:27:42   CRITICAL            INTERFACE   Intf ge-0-7 (9)
 operationally down

There are no shelved alarms
Completed in 0.01 seconds


admin@node.router#
```

### history

Use `show history`

```
admin@node.router# show history
Sun 2023-09-03 14:38:19 UTC
===================== ======================== ================================
 Timestamp             Menu                     Command
===================== ======================== ================================
 2023-08-29 08:25:56   pcli#                    adopt
 2023-08-29 09:00:26   pcli#                    ip a
 2023-08-29 09:00:32   pcli#                    show in
 2023-08-29 09:00:44   pcli#                    show config running flat
 2023-08-29 09:36:47   pcli#                    ping 192.168.128.1
 2023-08-29 09:36:52   pcli#                    ping 192.168.128.128
 2023-08-29 09:36:58   pcli#                    show alarms
 2023-08-29 09:39:29   pcli#                    show history
 2023-08-29 09:39:41   pcli#                    show user
 2023-09-03 14:03:52   pcli#                    show system version
 2023-09-03 14:07:42   pcli#                    configure
 2023-09-03 14:12:00   configure#               authority name myauthority
 2023-09-03 14:14:16   configure#               authority router name myrouter
 2023-09-03 14:14:42   configure#               authority router
 2023-09-03 14:14:52   configure#               authority router myrouter
 2023-09-03 14:15:33   router[name=myrouter]#   show
 2023-09-03 14:15:38   router[name=myrouter]#   where
 2023-09-03 14:16:26   router[name=myrouter]#   top
 2023-09-03 14:16:29   pcli#                    where
 2023-09-03 14:16:55   pcli#                    show
 2023-09-03 14:17:06   pcli#                    show config
 2023-09-03 14:17:30   pcli#                    show config running
 2023-09-03 14:17:55   pcli#                    show config running flat
 2023-09-03 14:18:31   pcli#                    sh config cand
 2023-09-03 14:18:37   pcli#                    show config candidate
 2023-09-03 14:19:26   pcli#                    commit
 2023-09-03 14:21:12   pcli#                    exit
 2023-09-03 14:21:18   pcli#                    quit
 2023-09-03 14:25:30   pcli#                    conf
 2023-09-03 14:26:13   pcli#                    config
 2023-09-03 14:26:58   pcli#                    exit
 2023-09-03 14:27:08   pcli#                    quit
 2023-09-03 14:27:20   pcli#                    configure
 2023-09-03 14:27:27   configure#               authority
 2023-09-03 14:27:36   authority#               name myauthority
 2023-09-03 14:27:45   authority#               router name myrouter
 2023-09-03 14:27:52   authority#               router myrouter
 2023-09-03 14:28:05   router[name=myrouter]#   top
 2023-09-03 14:28:19   pcli#                    compare config
 2023-09-03 14:29:30   pcli#                    restore config running
 2023-09-03 14:31:08   pcli#                    quit
 2023-09-03 14:31:36   pcli#                    configure
 2023-09-03 14:32:41   configure#               exit
 2023-09-03 14:32:54   pcli#                    repeat show alrms
 2023-09-03 14:32:59   pcli#                    repeat show alarms
 2023-09-03 14:37:52   pcli#                    show platform
 2023-09-03 14:38:19   pcli#                    show history
Completed in 0.01 seconds
admin@node.router#
```

### shell

```
admin@node.router# shell pwd
/home/admin
admin@node.router# 
```

```
admin@node.router# shell ls /etc/128technology
aide.conf                 processManagerProcessList.json
application-categories    process-metrics
applicationDirector.conf  pwquality.conf
application-modules       rappid-downloader.conf
applications              redis.1.conf
auditLogServiceList.json  redis.2.conf
cadillac.conf             redis.conf
config-exports            routing
default-acls.jsonc        routingEngineProcessList.json
factory-defaults.1.xml    rsyslog.1.conf
factory-defaults.2.xml    rsyslog.2.conf
factory-defaults.3.xml    rsyslog.conf
factory-defaults.4.xml    rsyslog.d
factory-defaults.5.xml    salt
factory-defaults.xml      salt.logrotate
factory-reset             snmpMetricsConfig.json
global.1.init             snmpObjectAgent.json
global.init               squid.conf
influxdb                  ssh
local.1.init              sync
local.init                synchronizedDirectories.json
log4j.properties          system.demon
mars.conf                 t128Version.json
monitoring-agent          tech-support-manifests
network-scripts           templates
password_policy           user-factory-defaults.xml
pki                       version-info
plugins
admin@node.router#
```

```
admin@node.router# shell cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

admin@node.router#
```

```
admin@node.router# shell uname -a
Linux sn40e72f87-5c86-4759-ba60-3e94d8d79a30 4.18.0-425.10.1.el8_7.x86_64 #1 SMP Thu Jan 12 11:31:50 PST 2023 x86_64 x86_64 x86_64 GNU/Linux
admin@node.router# 
```

128T has a management platform called `Conductor` which is GUI based, so usually there is no need to login to the PCLI except for some troubleshooting. Recently, SSRs are integrated with the Juniper Mist Cloud (part of WAN assurance), so the manamgement becomes even easier. Mist replaces the 128T Conductor platform.

We will look at the SSR-Mist integration in another post.

Hope this PCLI intro is useful.

