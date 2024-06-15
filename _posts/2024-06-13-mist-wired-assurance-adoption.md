---
layout: single
title:  "Juniper Mist Wired Assurance—Switch Adoption"
categories: Networking
tags: Mist
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/mist.png
author:
  name     : "Mist"
  avatar   : "/assets/images/mist.png"

sidebar:
  - title: "Blog"
    nav: my-sidebar

---

# Juniper Mist Wired Assurance—Switch Adoption
Apply the following CLI commands to adopt a Juniper switch

```sh
set system services ssh protocol-version v2
set system authentication-order password
set system login user mist class super-user
set system login user mist authentication encrypted-password $6$EhbbzQI97CiPFak2$dCs0jzflFjT9sFtE1Aa4nExpeFTggsVa/njO0.tY0tKJQUo59e8YjRuVuQRoITEP9XbPKJpK0V85zwkRZDSM/1
set system login user mist authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2tq6okBr3U3s/dOwP52jdggwjElYqmheyrow6Gg2Ksr0kASNTCO7OHlIWQv1IWBe4oXcqB65lwyGhyp0kYR9dQy3WqdTFax2fXxoIwjUvRBGt2Sf2K9PaRrLSiNiISAFCWV0Q0yl7OMWLhAudDW0Ha9UzPR/ZXtAfETRObY2TuEOqO9WObn2X8A37xjdAMuy9unZ5enbhAbTMDljJHF33Bfr9IoQyPJrtLQSrmQl19CmzuYpL3opWe1CT311bfEtDWnvVQJdTLZQqBAzLbZNl7N3XPnNFTJSoRPg6kKwE6sMHl27XJcuS+398MivdkdCXcWA6X8HHA1xUzPDKq94UU9tNP+1RpAX9jiDT/bOQx7U5XAWaWhj3OVIllTkU4V3o9a0VK+vh0/+BEJ5e6T7klu/NenotEFIUx2lI2a0ia4si0Je998T1HjIqTCagV/X6+a1nFfBM5NZ56pxPu1dfiEdiKpQtYrY7V0s1cUWlYEBUKoiVkN60fi3KHcWchB3E= mist@dc1d08f1-a7e9-47d1-88a3-e9c7c4b02e64"
set system services outbound-ssh client mist device-id dc1d08f1-a7e9-47d1-88a3-e9c7c4b02e64
set system services outbound-ssh client mist secret d0969a441c3294e845c4b1e3244b13cca7c9fb87193ffe24e3adf3ac02aebf5633ffad02734cfac9dd7051e52dd8433bc87449ac41fee08e498604947ba7c462
set system services outbound-ssh client mist services netconf keep-alive retry 12 timeout 5
set system services outbound-ssh client mist oc-term.mistsys.net port 2200 timeout 60 retry 1000
delete system phone-home
```

Download Junos Config

```sh
set system host-name myQFX
set system time-zone UTC
set protocols lldp interface all
set protocols lldp port-id-subtype interface-name
set protocols lldp port-description-type interface-alias
set protocols lldp-med interface all
set protocols rstp interface all
set protocols rstp bpdu-block-on-edge
set interfaces vme unit 0 family inet dhcp vendor-id Juniper
set interfaces vme unit 0 family inet dhcp force-discover
set interfaces vme unit 0 family inet dhcp retransmission-attempt 60
set interfaces vme unit 0 family inet dhcp client-identifier user-id ascii 5b4528ff7b41-M4aLquH9
set interfaces irb unit 0 family inet dhcp vendor-id Juniper
set interfaces irb unit 0 family inet dhcp force-discover
set interfaces irb unit 0 family inet dhcp retransmission-attempt 60
set interfaces irb unit 0 family inet dhcp client-identifier user-id ascii 5b4528ff7b41-0
set interfaces irb unit 0 description default
set vlans default vlan-id 1
set vlans default l3-interface irb.0
set groups top forwarding-options storm-control-profiles default all
set groups top system commit no-delta-synchronize
set groups top system name-server 8.8.8.8
set groups top system name-server 1.1.1.1
set groups top system ntp server 216.239.35.0
set groups top system ntp server 216.239.35.4
set groups top system ntp server 216.239.35.8
set groups top system ntp server 216.239.35.12
set apply-groups top
```

```sh
Warning: When a device is managed by Mist, the configuration changes made locally via shell will be overwritten with the configuration from the cloud. Please use the UI to make any config changes.

Last login: Sat Jun 15 05:12:15 2024 from 52.53.57.207
--- JUNOS 21.4R1.12 Kernel 64-bit  JNPR-12.1-20211117.16f0122_buil
{master:0}
mist@myQFX> show configuration | display set 
set version 21.4R1.12
set groups mist-script system scripts op file mist_helper.py arguments cmd
set groups mist-script system scripts op file mist_helper.py checksum sha-256 eb3c2c32e3b1289c80a9d2823daade27b86d5b2ddf07c27b07731675c09bd810
set groups mist-script system scripts op file mist_vccmd.py checksum sha-256 b4012cfbaa9ed42b776fa96d016dd9d1036ebe1a5706ede49567db366d5f5e4b
set groups mist-script system scripts op file mist_pyagent_tools.py arguments cmd
set groups mist-script system scripts op file mist_pyagent_tools.py arguments sha256sum
set groups mist-script system scripts op file mist_pyagent_tools.py arguments url
set groups mist-script system scripts op file mist_pyagent_tools.py arguments version
set groups mist-script system scripts op file mist_pyagent_tools.py checksum sha-256 e4f6b45db810817dc5a7e912808616fe15373c74c310d74e8efdc7184b966714
set groups mist-script event-options generate-event get-stats-every-three-minute time-interval 180
set groups mist-script event-options generate-event monitor-diskspace-now time-interval 86400
set groups mist-script event-options policy log-on-snmp-trap-link-up events snmp_trap_link_up
set groups mist-script event-options policy log-on-snmp-trap-link-up within 90 not events chassisd_vchassis_member_update_notice
set groups mist-script event-options policy log-on-snmp-trap-link-up attributes-match "{$$.interface-name}" matches "^[^.]+$"
set groups mist-script event-options policy log-on-snmp-trap-link-up then event-script mist_link_up_logger.py
set groups mist-script event-options policy log-on-snmp-trap-link-down events snmp_trap_link_down
set groups mist-script event-options policy log-on-snmp-trap-link-down within 90 not events chassisd_vchassis_member_update_notice
set groups mist-script event-options policy log-on-snmp-trap-link-down attributes-match "{$$.interface-name}" matches "^[^.]+$"
set groups mist-script event-options policy log-on-snmp-trap-link-down then event-script mist_link_down_logger.py
set groups mist-script event-options policy backup-cfg-after-commit events ui_commit_completed
set groups mist-script event-options policy backup-cfg-after-commit within 5 trigger on
set groups mist-script event-options policy backup-cfg-after-commit within 5 trigger 1
set groups mist-script event-options policy backup-cfg-after-commit then event-script mist_event_dispatcher.py
set groups mist-script event-options policy backup-cfg-after-no-confirmed events ui_commit_not_confirmed
set groups mist-script event-options policy backup-cfg-after-no-confirmed attributes-match ui_commit_not_confirmed.message matches .*complete
set groups mist-script event-options policy backup-cfg-after-no-confirmed then event-script mist_event_dispatcher.py
set groups mist-script event-options policy log-on-storm-ctrl-in-effect events l2ald_st_ctl_in_effect
set groups mist-script event-options policy log-on-storm-ctrl-in-effect then event-script mist_storm_control_event_logger.py
set groups mist-script event-options policy log-on-system-events events dot1xd_auth_session_deleted
set groups mist-script event-options policy log-on-system-events events dot1xd_rcvd_eaplogof_athntictd
set groups mist-script event-options policy log-on-system-events events dot1xd_usr_access_denied
set groups mist-script event-options policy log-on-system-events events dot1xd_usr_authenticated
set groups mist-script event-options policy log-on-system-events events dot1xd_usr_session_disconnected
set groups mist-script event-options policy log-on-system-events events dot1xd_usr_athntictd_gst_vlan
set groups mist-script event-options policy log-on-system-events events eswd_stp_state_change_info
set groups mist-script event-options policy log-on-system-events events l2cpd_receive_bpdu_block_enabled
set groups mist-script event-options policy log-on-system-events events authd_radius_server_status_change
set groups mist-script event-options policy log-on-system-events events chassisd_snmp_trap10
set groups mist-script event-options policy log-on-system-events events ddos_protocol_violation_set
set groups mist-script event-options policy log-on-system-events events ddos_protocol_violation_clear
set groups mist-script event-options policy log-on-system-events events evpn_bgp_peer_status_change
set groups mist-script event-options policy log-on-system-events events evpn_core_isolated
set groups mist-script event-options policy log-on-system-events events evpn_core_isolation_cleared
set groups mist-script event-options policy log-on-system-events events evpn_duplicate_mac
set groups mist-script event-options policy log-on-system-events within 60 trigger until
set groups mist-script event-options policy log-on-system-events within 60 trigger 10
set groups mist-script event-options policy log-on-system-events then event-script mist_event_dispatcher.py
set groups mist-script event-options policy log-critical-system-events events chassisd_vchassis_member_update_notice
set groups mist-script event-options policy log-critical-system-events events chassisd_vchassis_member_op_notice
set groups mist-script event-options policy log-critical-system-events events l2ald_mac_limit_reached_global
set groups mist-script event-options policy log-critical-system-events events l2ald_mac_limit_reset_global
set groups mist-script event-options policy log-critical-system-events events snmpd_trap_cold_start
set groups mist-script event-options policy log-critical-system-events then event-script mist_event_dispatcher.py
set groups mist-script event-options policy monitor-diskspace-policy events monitor-diskspace-now
set groups mist-script event-options policy monitor-diskspace-policy then event-script mist_monitor_diskspace.py
set groups mist-script event-options policy get-stats-policy events get-stats-every-three-minute
set groups mist-script event-options policy get-stats-policy then event-script mist_event_dispatcher.py
set groups mist-script event-options policy system-srx-route-events events rpd_ospf_nbrdown
set groups mist-script event-options policy system-srx-route-events events rpd_ospf_nbrup
set groups mist-script event-options policy system-srx-route-events events rpd_bgp_neighbor_state_changed
set groups mist-script event-options policy system-srx-route-events then event-script mist_srx_rt_event.py
set groups mist-script event-options policy log-on-vccp-port-up events vccpd_protocol_adjup
set groups mist-script event-options policy log-on-vccp-port-up attributes-match "{$$.interface-name}" matches "^[^.]+"
set groups mist-script event-options policy log-on-vccp-port-up then event-script mist_link_up_logger.py
set groups mist-script event-options policy log-on-vccp-port-down events vccpd_protocol_adjdown
set groups mist-script event-options policy log-on-vccp-port-down attributes-match "{$$.interface-name}" matches "^[^.]+"
set groups mist-script event-options policy log-on-vccp-port-down then event-script mist_link_down_logger.py
set groups mist-script event-options event-script file mist_event_dispatcher.py python-script-user mist
set groups mist-script event-options event-script file mist_event_dispatcher.py checksum sha-256 df5fc14607f9c0134da07f553f7822d1881563488784de9d1538b5dc67bd2f47
set groups mist-script event-options event-script file mist_link_up_logger.py python-script-user mist
set groups mist-script event-options event-script file mist_link_up_logger.py checksum sha-256 92f3090ff5fa38343e958f64ca7d03d12dcffffa2dbff25bb47f4f77a712d641
set groups mist-script event-options event-script file mist_link_down_logger.py python-script-user mist
set groups mist-script event-options event-script file mist_link_down_logger.py checksum sha-256 47c2f4b0c575b2a8ead65c36ef451c7705f0b040c7c1085dd443c7a5ac7a2153
set groups mist-script event-options event-script file mist_backup_cfg.py python-script-user mist
set groups mist-script event-options event-script file mist_backup_cfg.py checksum sha-256 11daea6cf3d3838053b78b015b70db9919526d37198c40d8a927a7521f56fd54
set groups mist-script event-options event-script file mist_storm_control_event_logger.py python-script-user mist
set groups mist-script event-options event-script file mist_storm_control_event_logger.py checksum sha-256 871b7efa4fe7d04d86034fcdcb63624c675956d7511b2a45ebb98299632be882
set groups mist-script event-options event-script file mist_link_event_capturer.py python-script-user mist
set groups mist-script event-options event-script file mist_link_event_capturer.py checksum sha-256 9d2c0fe73666e216e78ef8d9fc8f91173bbd6e4909f669f318356dba11938d4c
set groups mist-script event-options event-script file mist_monitor_diskspace.py python-script-user mist
set groups mist-script event-options event-script file mist_monitor_diskspace.py checksum sha-256 e751c8cf955c2fed7c5f38e3a6e5168c5550a1b24b272a0722421b917d697479
set groups mist-script event-options event-script file mist_dynamic_port_usages.py python-script-user mist
set groups mist-script event-options event-script file mist_dynamic_port_usages.py checksum sha-256 e6a667b08dc8714a043dc08f44a13b0480f35fae7c9eecfd1f932ca3821629a4
set groups mist-script event-options event-script file mist_dynamic_port_commit.py python-script-user mist
set groups mist-script event-options event-script file mist_dynamic_port_commit.py checksum sha-256 4fb25037a8ec5d57ac72f622c005baec2ab82b4f1e1a218f8eac03f6b39e71de
set groups mist-script event-options event-script file mist_event_aggregator.py python-script-user mist
set groups mist-script event-options event-script file mist_event_aggregator.py checksum sha-256 ca1426d833eddb1f519e7a5860cef6fedd08a2d091656add0537aceaf7a9eb7d
set groups mist-script event-options event-script file mist_vcsetup_agent.py python-script-user mist
set groups mist-script event-options event-script file mist_vcsetup_agent.py checksum sha-256 c336e83f4233c3287fd2f78596261f414603350927f75a10bd327ece41832a1f
set groups mist-script event-options event-script file mist_ssr.py python-script-user mist
set groups mist-script event-options event-script file mist_ssr.py checksum sha-256 59faa393b78e526bab64cddb4a99025e290b4edeeb583376816fc799903d3ff6
set groups mist-script event-options event-script file mist_srx_rt_event.py python-script-user mist
set groups mist-script event-options event-script file mist_srx_rt_event.py checksum sha-256 dd6fc6d40fed12444a264139ac3825461c3309d243f3176595d9d535ad37a384
set groups mist-dpc event-options generate-event timer-every-one-minute time-interval 60
set groups mist-dpc event-options policy dynamic-port-detect events lldp_neighbor_up
set groups mist-dpc event-options policy dynamic-port-detect events snmp_trap_link_up
set groups mist-dpc event-options policy dynamic-port-detect events snmp_trap_link_down
set groups mist-dpc event-options policy dynamic-port-detect events ui_commit_completed
set groups mist-dpc event-options policy dynamic-port-detect events dot1xd_usr_authenticated
set groups mist-dpc event-options policy dynamic-port-detect events get-stats-every-three-minute
set groups mist-dpc event-options policy dynamic-port-detect within 90 not events chassisd_vchassis_member_update_notice
set groups mist-dpc event-options policy dynamic-port-detect then event-script mist_dynamic_port_usages.py
set groups mist-dpc event-options policy dynamic-port-commit events timer-every-one-minute
set groups mist-dpc event-options policy dynamic-port-commit then event-script mist_dynamic_port_commit.py
set apply-groups mist-script
set system host-name myQFX
set system root-authentication encrypted-password "$1$KI99zGk6$MbYFuBbpLffu9tn2.sI7l1"
set system root-authentication ssh-dsa "ssh-dss AAAAB3NzaC1kc3MAAACBAMQrfP2bZyBXJ6PC7XXZ+MzErI8Jl6jah5L4/O8BsfP2hC7EvRfNoX7MqbrtCX/9gUH9gChVuBCB+ERULMdgRvM5uGhC/gs4UX+4dBbfBgKYYwgmisM8EoT25m7qI8ybpl2YZvHNznvO8h7kr4kpYuQEpKvgsTdH/Jle4Uqnjv7DAAAAFQDZaqA6QAgbW3O/zveaLCIDj6p0dwAAAIB1iL+krWrXiD8NPpY+w4dWXEqaV3bnobzPC4eyxQKBUCOr80Q5YBlWXVBHx9elwBWZwj0SF4hLKHznExnLerVsMuTMA846RbQmSz62vM6kGM13HFonWeQvWia0TDr78+rOEgWF2KHBSIxL51lmIDW8Gql9hJfD/Dr/NKP97w3L0wAAAIEAr3FkWU8XbYytQYEKxsIN9P1UQ1ERXB3G40YwqFO484SlyKyYCfaz+yNsaAJu2C8UebDIR3GieyNcOAKf3inCG8jQwjLvZskuZwrvlsz/xtcxSoAh9axJcdUfSJYMW/g+mD26JK1Cliw5rwp2nH9kUrJxeI7IReDp4egNkM4i15o= configurator@server1.he"
set system commit synchronize
set system scripts language python3
set system scripts synchronize
set system login user lab uid 2000
set system login user lab class super-user
set system login user lab authentication encrypted-password "$1$84J5Maes$cni5Hrazbd/IEHr/50oY30"
set system login user mist full-name mist
set system login user mist uid 63157
set system login user mist class super-user
set system login user mist authentication encrypted-password "$6$VBj18yZhDppkcZRC$sxujc2iPSwCUW3v1gh1E.z2Zn41viUnhcs4rTq/ugtOTYtnR8e/UpzTy3T60ZqoWkMoG5vLqUSa51uu5LezvK."
set system login user mist authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2tq6okBr3U3s/dOwP52j6wjElYqmheyrow6Gg2Ksr0kASNTCO7OHlIWQv1IWBe4oXcqB65lwyGhyp0kYR9dQy3WqdTFax2fXxoIwjUvRBGt2Sf2K9PaRrLSiNiISAFCWV0Q0yl7OMWLhAudDW0Ha9UzPR/ZXtAfETRObY2TuEOqO9WObn2X8A37xjdAMuy9unZ5enbhAbTMDljJHF33Bfr9IoQyPJrtLQSrmQl19CmzuvYpL3opWe1CT311bfEtDWnvVQJdTLZQqBAzLbZNl7N3XPnNFTJSoRPg6kKwE6sMHl27XJcuS+398MivdkdCXcWA6X8HHA1xUzPDKq94UU9tNP+1RpAX9jiDT/bOQx7U5XAWaWhj3OVIllTkU4V3o9a0VK+vh0/+BEJ5e6T7klu/NenotEFIUx2lI2a0ia4si0Je998T1HjIqTCagV/X6+a1nFfBM5NZ56pxPu1dfiEdiKpQtYrY7V0s1cUWlYEBUKoiVkN60fi3KHcWchB3E= mist@dc1d08f0-a7e9-47d1-88a2-e9c7c4b02e63"
set system login user mist authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDGSZXWQbRESKnHPyBrRFLz8pr+e7clzic0wjJFx2DC06bIaR1CqmxHQUKvuzUfmAedkrMcGw+wWNizYOlPFzWrHCNXpyzx2+Sr9ZUlk27UPfMI6XuB1GZ34ielCCLHt40eU4E1cdy4wnIZ3nxfL7vb6hxzFcC8/giRrujJBK87iSAoByaB9lulNvdHokRNnTP6mCJquOtGdaUTrqUqTr8sueEJTt8ehotdNSlrDreeHODZtB2sgQpmGNzQ2zEFskCqoZl9lgL8IlR/0NSvkOBPAOiikeemDzW3KpLdvXoXUvmMaZPkB0zpS7/T/gMtYLxZ51qO49PKVBvBXr+a3nIkJe1403J80lvmOwVbdO8bvywV6mNpjXe2J0m2Cm+LV7kdD1lZKhih4/DEvBJ4rkfZN/t+rxqms59EpTCkKSb/+N3j/q1OKfXY8o2VEvSkTn1Ag8UNiq1kh84F7CC00UtEbpBAV/Z/AOCaVrdzm3595+9oL2GsXP0i5cPx2ylaFzJW0hB4OerU4DzQ4n/2C8Oq4v/sgBPwjW4YRFmaidoQr2blcoqf916dRsmQR5G1Uc1g3ps+ph5BdjUiZgPvxCTCPIOzNsTbu888UDgnBBF1YeeGXRmfkwQobu8eWfjm+HI0H8/rSfIPDBrjI3hZsHcN0ZYKyzMwkWVmAdjuCaYsWw== noninteractive@mistsys.com"
set system login user mist authentication ssh-ecdsa "ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBItBz6WSD+YBbvX2qvKlLQ71h2qms2A5ARsFQK7GQSiBbh6450VcCNNq6YDmb1/pOrrJ8q7cR90vsvNROi4ZD7I= bot+interactive@mistsys.com"
set system login message "Reset Config has been loaded "
set system services ftp
set system services ssh root-login allow
set system services ssh protocol-version v2
set system services telnet
set system services netconf ssh
set system services outbound-ssh traceoptions file outbound-ssh.log
set system services outbound-ssh traceoptions file size 64k
set system services outbound-ssh traceoptions file files 5
set system services outbound-ssh traceoptions flag all
set system services outbound-ssh client mist device-id dc1d08f0-a7e9-47d1-88a2-e9c7c4b02e63.5c4527ff7b40
set system services outbound-ssh client mist secret "$9$k.PQAp0ESrAtBESrvMNdbwoJUjHqP5GUCtu0hcKM8LxdYgojk.LxHqP56/WLX-YgGUH5z3PfO1hcle8X7NYgDikf5FHkRhrlMWLxN-YgGDimPQHkfzn/0O1Rhyrv8LNVs2vMUjq.5TRhcreWLxN-wYBIclvWx7UjiHP5369AtO3ntOBRSyM8X7s2jHqmTz7-2aZGiH1RESvWXxdY2acyYg4aiHp0OBIcSrevMX369p01hclKMW7-YgoDHq2gaUDHmPz3690BIEclvWIRdbwYoaQF3np0cylKWL69vW8X-ds24JGiPfzAuOTQcyKv7Ndbs2JGq.5zF/gojqmfzFcyrevL-dsJUjdVP5QzCA0BIhevNdb"
set system services outbound-ssh client mist keep-alive retry 12
set system services outbound-ssh client mist keep-alive timeout 5
set system services outbound-ssh client mist services netconf
set system services outbound-ssh client mist oc-term.mistsys.net port 2200
set system services outbound-ssh client mist oc-term.mistsys.net retry 1000
set system services outbound-ssh client mist oc-term.mistsys.net timeout 60
set system arp aging-timer 5
set system authentication-order password
set system name-server 8.8.8.8
set system syslog file escript.log archive size 2m
set system syslog file escript.log archive files 5
set system syslog file interactive-commands interactive-commands any
set system syslog file interactive-commands match "!(.*mist.*)"
set system syslog file interactive-commands archive size 2m
set system syslog file interactive-commands archive files 5
set system syslog file messages any alert
set system syslog file messages authorization any
set system syslog file messages archive size 2m
set system syslog file messages archive files 5
set system syslog file op-script.log archive size 2m
set system syslog file op-script.log archive files 5
set system syslog file snapshot archive size 2m
set system syslog file snapshot archive files 5
set system processes dhcp-service traceoptions file dhcp_logfile
set system processes dhcp-service traceoptions file size 10m
set system processes dhcp-service traceoptions level all
set system processes dhcp-service traceoptions flag all
set system processes app-engine-virtual-machine-management-service traceoptions level notice
set system processes app-engine-virtual-machine-management-service traceoptions flag all
set system ntp server 172.25.11.254     
set interfaces ge-0/0/0 unit 0 family ethernet-switching vlan members default
set interfaces ge-0/0/0 unit 0 family ethernet-switching storm-control default
set interfaces em0 unit 0 family inet address 10.210.14.196/28
set forwarding-options storm-control-profiles default all
set routing-options static route 0.0.0.0/0 next-hop 10.210.14.206
set protocols lldp interface all
set protocols lldp-med interface all
set protocols igmp-snooping vlan default
set protocols rstp interface ge-0/0/0

{master:0}
mist@myQFX> 
```

Configuration Groups added by Mist

```sh
{master:0}
mist@myQFX> show configuration groups ?
Possible completions:
  <[Enter]>            Execute this command
  <group_name>         Group name
  mist-dpc             Group name
  mist-script          Group name
  |                    Pipe through a command
{master:0}
mist@myQFX> show configuration groups mist-dpc 
event-options {
    generate-event {
        timer-every-one-minute time-interval 60;
    }
    policy dynamic-port-detect {
        events [ lldp_neighbor_up snmp_trap_link_up snmp_trap_link_down ui_commit_completed dot1xd_usr_authenticated get-stats-every-three-minute ];
        within 90 {
            not events chassisd_vchassis_member_update_notice;
        }
        then {
            event-script mist_dynamic_port_usages.py;
        }
    }
    policy dynamic-port-commit {
        events timer-every-one-minute;
        then {
            event-script mist_dynamic_port_commit.py;
        }
    }
}

{master:0}
mist@myQFX> show configuration groups mist-script     
system {
    scripts {
        op {
            file mist_helper.py {
                arguments {
                    cmd;
                }
                checksum sha-256 eb3c2c32e3b1289c80a9d2823daade27b86d5b2ddf07c27b07731675c09bd810;
            }
            file mist_vccmd.py {
                checksum sha-256 b4012cfbaa9ed42b776fa96d016dd9d1036ebe1a5706ede49567db366d5f5e4b;
            }
            file mist_pyagent_tools.py {
                arguments {
                    cmd;
                    sha256sum;
                    url;
                    version;
                }
                checksum sha-256 e4f6b45db810817dc5a7e912808616fe15373c74c310d74e8efdc7184b966714;
            }
        }
    }
}
event-options {
    generate-event {
        get-stats-every-three-minute time-interval 180;
        monitor-diskspace-now time-interval 86400;
    }
    policy log-on-snmp-trap-link-up {
        events snmp_trap_link_up;
        within 90 {
            not events chassisd_vchassis_member_update_notice;
        }                               
        attributes-match {
            "{$$.interface-name}" matches "^[^.]+$";
        }
        then {
            event-script mist_link_up_logger.py;
        }
    }
    policy log-on-snmp-trap-link-down {
        events snmp_trap_link_down;
        within 90 {
            not events chassisd_vchassis_member_update_notice;
        }
        attributes-match {
            "{$$.interface-name}" matches "^[^.]+$";
        }
        then {
            event-script mist_link_down_logger.py;
        }
    }
    policy backup-cfg-after-commit {
        events ui_commit_completed;
        within 5 {
            trigger on 1;
        }
        then {
            event-script mist_event_dispatcher.py;
        }
    }
    policy backup-cfg-after-no-confirmed {
        events ui_commit_not_confirmed;
        attributes-match {
            ui_commit_not_confirmed.message matches .*complete;
        }
        then {
            event-script mist_event_dispatcher.py;
        }                               
    }
    policy log-on-storm-ctrl-in-effect {
        events l2ald_st_ctl_in_effect;
        then {
            event-script mist_storm_control_event_logger.py;
        }
    }
    policy log-on-system-events {
        events [ dot1xd_auth_session_deleted dot1xd_rcvd_eaplogof_athntictd dot1xd_usr_access_denied dot1xd_usr_authenticated dot1xd_usr_session_disconnected dot1xd_usr_athntictd_gst_vlan eswd_stp_state_change_info l2cpd_receive_bpdu_block_enabled authd_radius_server_status_change chassisd_snmp_trap10 ddos_protocol_violation_set ddos_protocol_violation_clear evpn_bgp_peer_status_change evpn_core_isolated evpn_core_isolation_cleared evpn_duplicate_mac ];
        within 60 {
            trigger until 10;
        }
        then {
            event-script mist_event_dispatcher.py;
        }
    }
    policy log-critical-system-events {
        events [ chassisd_vchassis_member_update_notice chassisd_vchassis_member_op_notice l2ald_mac_limit_reached_global l2ald_mac_limit_reset_global snmpd_trap_cold_start ];
        then {
            event-script mist_event_dispatcher.py;
        }
    }
    policy monitor-diskspace-policy {
        events monitor-diskspace-now;
        then {
            event-script mist_monitor_diskspace.py;
        }
    }
    policy get-stats-policy {           
        events get-stats-every-three-minute;
        then {
            event-script mist_event_dispatcher.py;
        }
    }
    policy system-srx-route-events {
        events [ rpd_ospf_nbrdown rpd_ospf_nbrup rpd_bgp_neighbor_state_changed ];
        then {
            event-script mist_srx_rt_event.py;
        }
    }
    policy log-on-vccp-port-up {
        events vccpd_protocol_adjup;
        attributes-match {
            "{$$.interface-name}" matches "^[^.]+";
        }
        then {
            event-script mist_link_up_logger.py;
        }
    }
    policy log-on-vccp-port-down {
        events vccpd_protocol_adjdown;
        attributes-match {
            "{$$.interface-name}" matches "^[^.]+";
        }
        then {
            event-script mist_link_down_logger.py;
        }
    }
    event-script {
        file mist_event_dispatcher.py {
            python-script-user mist;
            checksum sha-256 df5fc14607f9c0134da07f553f7822d1881563488784de9d1538b5dc67bd2f47;
        }
        file mist_link_up_logger.py {   
            python-script-user mist;
            checksum sha-256 92f3090ff5fa38343e958f64ca7d03d12dcffffa2dbff25bb47f4f77a712d641;
        }
        file mist_link_down_logger.py {
            python-script-user mist;
            checksum sha-256 47c2f4b0c575b2a8ead65c36ef451c7705f0b040c7c1085dd443c7a5ac7a2153;
        }
        file mist_backup_cfg.py {
            python-script-user mist;
            checksum sha-256 11daea6cf3d3838053b78b015b70db9919526d37198c40d8a927a7521f56fd54;
        }
        file mist_storm_control_event_logger.py {
            python-script-user mist;
            checksum sha-256 871b7efa4fe7d04d86034fcdcb63624c675956d7511b2a45ebb98299632be882;
        }
        file mist_link_event_capturer.py {
            python-script-user mist;
            checksum sha-256 9d2c0fe73666e216e78ef8d9fc8f91173bbd6e4909f669f318356dba11938d4c;
        }
        file mist_monitor_diskspace.py {
            python-script-user mist;
            checksum sha-256 e751c8cf955c2fed7c5f38e3a6e5168c5550a1b24b272a0722421b917d697479;
        }
        file mist_dynamic_port_usages.py {
            python-script-user mist;
            checksum sha-256 e6a667b08dc8714a043dc08f44a13b0480f35fae7c9eecfd1f932ca3821629a4;
        }
        file mist_dynamic_port_commit.py {
            python-script-user mist;    
            checksum sha-256 4fb25037a8ec5d57ac72f622c005baec2ab82b4f1e1a218f8eac03f6b39e71de;
        }
        file mist_event_aggregator.py {
            python-script-user mist;
            checksum sha-256 ca1426d833eddb1f519e7a5860cef6fedd08a2d091656add0537aceaf7a9eb7d;
        }
        file mist_vcsetup_agent.py {
            python-script-user mist;
            checksum sha-256 c336e83f4233c3287fd2f78596261f414603350927f75a10bd327ece41832a1f;
        }
        file mist_ssr.py {
            python-script-user mist;
            checksum sha-256 59faa393b78e526bab64cddb4a99025e290b4edeeb583376816fc799903d3ff6;
        }
        file mist_srx_rt_event.py {
            python-script-user mist;
            checksum sha-256 dd6fc6d40fed12444a264139ac3825461c3309d243f3176595d9d535ad37a384;
        }
    }
}

{master:0}
mist@myQFX> 
```

Python Event Scripts

```sh
{master:0}
mist@myQFX> file list /var/db/scripts/event      

/var/db/scripts/event:
memory_parameters.slax@ -> /packages/mnt/junos-runtime/var/db/scripts/event/memory_parameters.slax
mist_backup_cfg.py*
mist_dynamic_port_commit.py*
mist_dynamic_port_usages.py*
mist_event_aggregator.py*
mist_event_dispatcher.py*
mist_link_down_logger.py*
mist_link_event_capturer.py*
mist_link_up_logger.py*
mist_monitor_diskspace.py*
mist_srx_rt_event.py*
mist_ssr.py*
mist_storm_control_event_logger.py*
mist_vcsetup_agent.py*
pam_user_lock_deny_host.slax@ -> /packages/mnt/junos-runtime/var/db/scripts/event/pam_user_lock_deny_host.slax
vpp-events-handler.slax@ -> /packages/mnt/junos-runtime-qfx/var/db/scripts/event/vpp-events-handler.slax

{master:0}
mist@myQFX> 
```
Commit and Op Scripts

```sh
{master:0}
mist@myQFX> file list /var/db/scripts/commit/  

/var/db/scripts/commit/:

{master:0}
mist@myQFX> file list /var/db/scripts/op         

/var/db/scripts/op:
commit-config.slax*
mist_helper.py*
mist_pyagent_tools.py*
mist_vccmd.py*

{master:0}
mist@myQFX> 
```

```sh
master:0}
mist@myQFX> show system commit 
0   2024-05-06 06:38:48 UTC by mist via netconf
    oc-script
1   2024-04-19 11:47:52 UTC by mist via netconf
    oc-script
2   2024-04-02 03:35:34 UTC by mist via netconf commit confirmed, rollback in 10mins
    mist-authkeys
3   2024-04-02 03:35:26 UTC by mist via netconf
    oc-script
4   2024-04-02 03:35:13 UTC by mist via netconf
    oc-config
5   2024-04-02 03:12:00 UTC by mist via netconf
    install-mist-mgmt
6   2024-04-02 03:11:04 UTC by lab via cli
7   2024-04-02 03:07:55 UTC by lab via cli
8   2022-02-01 20:59:54 UTC by root via other
9   2018-10-15 19:28:52 UTC by lab via cli
10  2018-10-15 19:23:42 UTC by lab via cli
11  2018-10-15 19:22:21 UTC by lab via cli
12  2018-10-15 19:18:54 UTC by root via other
13  2018-10-15 19:09:18 UTC by root via junoscript
    commit-config.slax
14  2018-10-15 19:09:12 UTC by root via autoinstall
15  2018-10-15 18:57:26 UTC by root via autoinstall
16  2018-10-15 18:55:50 UTC by root via other
17  2018-10-15 18:55:24 UTC by root via other
rescue  2019-12-10 16:33:58 UTC by root via cli

{master:0}
mist@myQFX>
```
