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

Here is a sample XML file 
```xml
<?xml version="1.0"?>
<inventory xmlns="https://www.acme.com/ns/routers"
           xmlns:qa="https://www.acme.com/ns/routers/qa"
           xmlns:dev="https://www.acme.com/ns/routers/dev">
    <qa:device name="R1">
        <qa:osversion>25.01</qa:osversion>
        <qa:uptime>2 days</qa:uptime>
        <qa:vendor>acme</qa:vendor>
        <qa:serial>ABC12341</qa:serial>
        <qa:snmp name="public" permission="ro"/>
        <qa:snmp name="private" permission="rw"/>
    </qa:device>
    <qa:device name="R2">
        <qa:osversion>25.01</qa:osversion>
        <qa:uptime>14 days</qa:uptime>
        <qa:vendor>acme</qa:vendor>
        <qa:serial>ABC12342</qa:serial>
        <qa:snmp name="public" permission="ro"/>
        <qa:snmp name="private" permission="rw"/>
    </qa:device>
    <dev:device name="R3">
        <dev:osversion>25.01</dev:osversion>
        <dev:uptime>182 days</dev:uptime>
        <dev:vendor>acme</dev:vendor>
        <dev:serial>ABC12343</dev:serial>
        <dev:snmp name="public" permission="ro"/>
        <dev:snmp name="private" permission="rw"/>
    </dev:device>
</inventory>
```

and the Python program

```py
#! /Users/pradeep/opt/anaconda3/bin/python3

print("Hello!\nWelcome to Data Serialization in Networking.\n")

import os
import yaml
import json
import xml.etree.ElementTree as ET
import xmltodict


tree = ET.parse('myXMLdemo.xml')
myroot = tree.getroot()
print(myroot.tag)
for child in myroot:
    print(child.tag, child.attrib)

for serial in myroot.iter('{https://www.acme.com/ns/routers/qa}serial'):
    print(serial.text)

for serial in myroot.iter('{https://www.acme.com/ns/routers/dev}serial'):
    print(serial.text)

for device in myroot.findall('{https://www.acme.com/ns/routers/qa}device'):
    name=device.attrib['name']
    serial=device.find('{https://www.acme.com/ns/routers/qa}serial').text
    print(name, serial)

for snmp in myroot.iter('{https://www.acme.com/ns/routers/dev}snmp'):
    print(snmp.attrib)

with open('myXMLdemo.xml') as xml_file:
    xml = xmltodict.parse(xml_file.read())

print("Our XML file in Dictionary format\n")
print(json.dumps(xml, indent=4))

print("This time, without the @ attribute prefix:\n")

print(json.dumps(xml, indent=4).replace("@", "")) # to get rid of @ attr_prefix
```

and the output:
```
Hello!
Welcome to Data Serialization in Networking.

{https://www.acme.com/ns/routers}inventory
{https://www.acme.com/ns/routers/qa}device {'name': 'R1'}
{https://www.acme.com/ns/routers/qa}device {'name': 'R2'}
{https://www.acme.com/ns/routers/dev}device {'name': 'R3'}
ABC12341
ABC12342
ABC12343
R1 ABC12341
R2 ABC12342
{'name': 'public', 'permission': 'ro'}
{'name': 'private', 'permission': 'rw'}
Our XML file in Dictionary format

{
    "inventory": {
        "@xmlns": "https://www.acme.com/ns/routers",
        "@xmlns:qa": "https://www.acme.com/ns/routers/qa",
        "@xmlns:dev": "https://www.acme.com/ns/routers/dev",
        "qa:device": [
            {
                "@name": "R1",
                "qa:osversion": "25.01",
                "qa:uptime": "2 days",
                "qa:vendor": "acme",
                "qa:serial": "ABC12341",
                "qa:snmp": [
                    {
                        "@name": "public",
                        "@permission": "ro"
                    },
                    {
                        "@name": "private",
                        "@permission": "rw"
                    }
                ]
            },
            {
                "@name": "R2",
                "qa:osversion": "25.01",
                "qa:uptime": "14 days",
                "qa:vendor": "acme",
                "qa:serial": "ABC12342",
                "qa:snmp": [
                    {
                        "@name": "public",
                        "@permission": "ro"
                    },
                    {
                        "@name": "private",
                        "@permission": "rw"
                    }
                ]
            }
        ],
        "dev:device": {
            "@name": "R3",
            "dev:osversion": "25.01",
            "dev:uptime": "182 days",
            "dev:vendor": "acme",
            "dev:serial": "ABC12343",
            "dev:snmp": [
                {
                    "@name": "public",
                    "@permission": "ro"
                },
                {
                    "@name": "private",
                    "@permission": "rw"
                }
            ]
        }
    }
}
This time, without the @ attribute prefix:

{
    "inventory": {
        "xmlns": "https://www.acme.com/ns/routers",
        "xmlns:qa": "https://www.acme.com/ns/routers/qa",
        "xmlns:dev": "https://www.acme.com/ns/routers/dev",
        "qa:device": [
            {
                "name": "R1",
                "qa:osversion": "25.01",
                "qa:uptime": "2 days",
                "qa:vendor": "acme",
                "qa:serial": "ABC12341",
                "qa:snmp": [
                    {
                        "name": "public",
                        "permission": "ro"
                    },
                    {
                        "name": "private",
                        "permission": "rw"
                    }
                ]
            },
            {
                "name": "R2",
                "qa:osversion": "25.01",
                "qa:uptime": "14 days",
                "qa:vendor": "acme",
                "qa:serial": "ABC12342",
                "qa:snmp": [
                    {
                        "name": "public",
                        "permission": "ro"
                    },
                    {
                        "name": "private",
                        "permission": "rw"
                    }
                ]
            }
        ],
        "dev:device": {
            "name": "R3",
            "dev:osversion": "25.01",
            "dev:uptime": "182 days",
            "dev:vendor": "acme",
            "dev:serial": "ABC12343",
            "dev:snmp": [
                {
                    "name": "public",
                    "permission": "ro"
                },
                {
                    "name": "private",
                    "permission": "rw"
                }
            ]
        }
    }
}


```
