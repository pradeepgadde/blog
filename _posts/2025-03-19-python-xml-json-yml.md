---
layout: single
title:  "Python: Data Serialization using XML, YAML, JSON"
categories: Programming
tags: Python
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---

# Data Serialization using XML, YAMl, JSON

Here is a sample YAML file, named myYAMLdemo.yml

```yaml
---
# This is a sample YAML file describing a Juniper SRX configuration

vendor: "Juniper"
model: "SRX380"
version: "24.4R2"
hostname: "mySRX"
# vlan list
vlans:
 - 101
 - 102
 - 103
# list of dictionaries
vlan_settings:
 - id: 201
   name: employee_vlan
 - id: 202
   name: contractor_vlan
 - id: 203
   name: guest_vlan
# dictionary
vlans_dict:
  301: engg
  302: qa
  303: it
zones:
 - trust
 - untrust
 - dmz
interfaces: [ge-0/0/1, ge-0/0/2, ge-0/0/3]
security_settings:
- trust: ge-0/0/1
- untrust: ge-0/0/2
- dmz: ge-0/0/3
host_inbound_traffic:
- trust: all
- untrust: ping
- dmz: https
# root_key
snmp:
  # sub-key and its value
  rw: public
  # sub-key and its value
  ro: private
  
login:
  users:
   - alice
   - bob
  classes:
   - super-user
   - operator
  message: "Welcome!"
  idle_timeout: 30

```

and a Python script to play with this YAML file

```py
#! /Users/pradeep/opt/anaconda3/bin/python3

print("Hello!\nWelcome to Data Serialization in Networking.\n")

import os
import yaml
import json

if __name__ == "__main__":
    if os.path.exists('myYAMLdemo.yml'):
        fd=open('myYAMLdemo.yml', 'r')
        data=yaml.safe_load(fd)
        print(json.dumps(data, indent=4))
    else:
        print("There is no input file!")

# define a dictionary called device_facts, with the output from the script so far
device_facts= {
    "vendor": "Juniper",
    "model": "SRX380",
    "version": "24.4R2",
    "hostname": "mySRX",
    "vlans": [
        101,
        102,
        103
    ],
    "vlan_settings": [
        {
            "id": 201,
            "name": "employee_vlan"
        },
        {
            "id": 202,
            "name": "contractor_vlan"
        },
        {
            "id": 203,
            "name": "guest_vlan"
        }
    ],
    "vlans_dict": {
        "301": "engg",
        "302": "qa",
        "303": "it"
    },
    "zones": [
        "trust",
        "untrust",
        "dmz"
    ],
    "interfaces": [
        "ge-0/0/1",
        "ge-0/0/2",
        "ge-0/0/3"
    ],
    "security_settings": [
        {
            "trust": "ge-0/0/1"
        },
        {
            "untrust": "ge-0/0/2"
        },
        {
            "dmz": "ge-0/0/3"
        }
    ],
    "host_inbound_traffic": [
        {
            "trust": "all"
        },
        {
            "untrust": "ping"
        },
        {
            "dmz": "https"
        }
    ],
    "snmp": {
        "rw": "public",
        "ro": "private"
    },
    "login": {
        "users": [
            "alice",
            "bob"
        ],
        "classes": [
            "super-user",
            "operator"
        ],
        "message": "Welcome!",
        "idle_timeout": 30
    }
}

print(type(device_facts)) # dict
print(device_facts["hostname"]) #mySRX
print(device_facts) # all in one line
device_facts_str=json.dumps(device_facts)
print(type(device_facts_str)) # string, because, json.dumps converts dict to str
print(json.dumps(device_facts))
print(json.dumps(device_facts, indent=4))

another_device_facts = '{"hostname": "R1", "uptime": "365 days", "serial_number": "ABC1234"}'
print(another_device_facts)
print(type(another_device_facts)) # str
another_device_facts_dict=json.loads(another_device_facts) 
print(type(another_device_facts_dict)) # dict, becuase json.loads converts str to dict
print(another_device_facts_dict["uptime"])

```
and here is the output:
```
Hello!
Welcome to Data Serialization in Networking.

{
    "vendor": "Juniper",
    "model": "SRX380",
    "version": "24.4R2",
    "hostname": "mySRX",
    "vlans": [
        101,
        102,
        103
    ],
    "vlan_settings": [
        {
            "id": 201,
            "name": "employee_vlan"
        },
        {
            "id": 202,
            "name": "contractor_vlan"
        },
        {
            "id": 203,
            "name": "guest_vlan"
        }
    ],
    "vlans_dict": {
        "301": "engg",
        "302": "qa",
        "303": "it"
    },
    "zones": [
        "trust",
        "untrust",
        "dmz"
    ],
    "interfaces": [
        "ge-0/0/1",
        "ge-0/0/2",
        "ge-0/0/3"
    ],
    "security_settings": [
        {
            "trust": "ge-0/0/1"
        },
        {
            "untrust": "ge-0/0/2"
        },
        {
            "dmz": "ge-0/0/3"
        }
    ],
    "host_inbound_traffic": [
        {
            "trust": "all"
        },
        {
            "untrust": "ping"
        },
        {
            "dmz": "https"
        }
    ],
    "snmp": {
        "rw": "public",
        "ro": "private"
    },
    "login": {
        "users": [
            "alice",
            "bob"
        ],
        "classes": [
            "super-user",
            "operator"
        ],
        "message": "Welcome!",
        "idle_timeout": 30
    }
}
<class 'dict'>
mySRX
{'vendor': 'Juniper', 'model': 'SRX380', 'version': '24.4R2', 'hostname': 'mySRX', 'vlans': [101, 102, 103], 'vlan_settings': [{'id': 201, 'name': 'employee_vlan'}, {'id': 202, 'name': 'contractor_vlan'}, {'id': 203, 'name': 'guest_vlan'}], 'vlans_dict': {'301': 'engg', '302': 'qa', '303': 'it'}, 'zones': ['trust', 'untrust', 'dmz'], 'interfaces': ['ge-0/0/1', 'ge-0/0/2', 'ge-0/0/3'], 'security_settings': [{'trust': 'ge-0/0/1'}, {'untrust': 'ge-0/0/2'}, {'dmz': 'ge-0/0/3'}], 'host_inbound_traffic': [{'trust': 'all'}, {'untrust': 'ping'}, {'dmz': 'https'}], 'snmp': {'rw': 'public', 'ro': 'private'}, 'login': {'users': ['alice', 'bob'], 'classes': ['super-user', 'operator'], 'message': 'Welcome!', 'idle_timeout': 30}}
<class 'str'>
{"vendor": "Juniper", "model": "SRX380", "version": "24.4R2", "hostname": "mySRX", "vlans": [101, 102, 103], "vlan_settings": [{"id": 201, "name": "employee_vlan"}, {"id": 202, "name": "contractor_vlan"}, {"id": 203, "name": "guest_vlan"}], "vlans_dict": {"301": "engg", "302": "qa", "303": "it"}, "zones": ["trust", "untrust", "dmz"], "interfaces": ["ge-0/0/1", "ge-0/0/2", "ge-0/0/3"], "security_settings": [{"trust": "ge-0/0/1"}, {"untrust": "ge-0/0/2"}, {"dmz": "ge-0/0/3"}], "host_inbound_traffic": [{"trust": "all"}, {"untrust": "ping"}, {"dmz": "https"}], "snmp": {"rw": "public", "ro": "private"}, "login": {"users": ["alice", "bob"], "classes": ["super-user", "operator"], "message": "Welcome!", "idle_timeout": 30}}
{
    "vendor": "Juniper",
    "model": "SRX380",
    "version": "24.4R2",
    "hostname": "mySRX",
    "vlans": [
        101,
        102,
        103
    ],
    "vlan_settings": [
        {
            "id": 201,
            "name": "employee_vlan"
        },
        {
            "id": 202,
            "name": "contractor_vlan"
        },
        {
            "id": 203,
            "name": "guest_vlan"
        }
    ],
    "vlans_dict": {
        "301": "engg",
        "302": "qa",
        "303": "it"
    },
    "zones": [
        "trust",
        "untrust",
        "dmz"
    ],
    "interfaces": [
        "ge-0/0/1",
        "ge-0/0/2",
        "ge-0/0/3"
    ],
    "security_settings": [
        {
            "trust": "ge-0/0/1"
        },
        {
            "untrust": "ge-0/0/2"
        },
        {
            "dmz": "ge-0/0/3"
        }
    ],
    "host_inbound_traffic": [
        {
            "trust": "all"
        },
        {
            "untrust": "ping"
        },
        {
            "dmz": "https"
        }
    ],
    "snmp": {
        "rw": "public",
        "ro": "private"
    },
    "login": {
        "users": [
            "alice",
            "bob"
        ],
        "classes": [
            "super-user",
            "operator"
        ],
        "message": "Welcome!",
        "idle_timeout": 30
    }
}
{"hostname": "R1", "uptime": "365 days", "serial_number": "ABC1234"}
<class 'str'>
<class 'dict'>
365 days


```