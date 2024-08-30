---
layout: single
title:  "Cisco IOS Python Requests"
categories: Networking
tags: OSPF
show_date: true
classes: wide
header:
  teaser: /assets/images/cisco.png
author:
  name     : "Cisco"
  avatar   : "/assets/images/cisco.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar
---
# Cisco IOS Python Requests
In this example, there are two routers connected by a single link and OSPF is enabled to advertise the static routes into OSPF.

## Python Program
Here is the R0 configuration:

```py
import base64
import requests
import urllib3

urllib3.disable_warnings()
auth = 'juniper:juniper'.encode()
print(auth)
base_auth = base64.b64encode(auth)
print(base_auth)
base_auth = base64.b64decode(base_auth)
print(base_auth)

new_auth = base64.b64encode(bytes(u'juniper:juniper','utf-8'))
print(new_auth)

DNA_URL = "https://sandboxdnac.cisco.com"
HEADERS = {"content-type":"application/json"}
USER = "devnetuser"
PASS = "Cisco123!"
AUTH = (USER, PASS)
LOGIN_URL = DNA_URL + "/api/system/v1/auth/token"
result = requests.post(url=LOGIN_URL, auth=AUTH, headers=HEADERS, verify=False)
print(result.json())
TOKEN = result.json()['Token']
print(TOKEN)
HEADERS['X-Auth-Token'] = TOKEN
print(HEADERS)

INVENTORY_URL = DNA_URL +"/dna/intent/api/v1/network-device"
response = requests.get(url=INVENTORY_URL, headers=HEADERS, verify=False)
print(response.ok)
print(response.text)
```



## Output

```py
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/base64-demo.py
b'juniper:juniper'
b'anVuaXBlcjpqdW5pcGVy'
b'juniper:juniper'
b'anVuaXBlcjpqdW5pcGVy'
{'Token': 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2Njk3NzQwYWU4ZTc5NDc0ZjcxN2FiNWEiLCJhdXRoU291cmNlIjoiaW50ZXJuYWwiLCJ0ZW5hbnROYW1lIjoiVE5UMCIsInJvbGVzIjpbIjY2OTZmMDFhYTA0Y2FlNjVjM2MzN2IwMiJdLCJ0ZW5hbnRJZCI6IjY2OTZmMDE4YTA0Y2FlNjVjM2MzN2FmYiIsImV4cCI6MTcyNDg4MzIzNCwiaWF0IjoxNzI0ODc5NjM0LCJqdGkiOiI2OTdkNGQ1Zi0zMDk4LTQwZWQtYjQ4OC03NjQ2YTI5NTk1MGQiLCJ1c2VybmFtZSI6ImRldm5ldHVzZXIifQ.Ka_tMNNoJJN9dudmEfM2bNEHBF6YlGG8VRJo22QXYTJ5QZAZkAzKT0U1YqtHDcHnPoeyaSddnp6Et6XazLBx0gq5LpJGMOhMrJz44GmzuxWNLwajgSm8WCt0MhenA9a9vQKdTlvds2AG6p7LZ0_NFHXR6w_5ZDsLXE-k_zibwGwunOeZuDYAuZJogciWw2T3-Jv0yxVTGqAE95Q53JuGk6sBtOtETXb3VACDkIeb9nTnYHIM1M-8D_RFzSFf_wsu8l0YeOcqAqLDPXYH4T5tGjSxRRAs0TIrIOSInTdisGRt7U9xu1poefeZPtFBIdslWNUQd2GnYdtEsU9xfnh1uw'}
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2Njk3NzQwYWU4ZTc5NDc0ZjcxN2FiNWEiLCJhdXRoU291cmNlIjoiaW50ZXJuYWwiLCJ0ZW5hbnROYW1lIjoiVE5UMCIsInJvbGVzIjpbIjY2OTZmMDFhYTA0Y2FlNjVjM2MzN2IwMiJdLCJ0ZW5hbnRJZCI6IjY2OTZmMDE4YTA0Y2FlNjVjM2MzN2FmYiIsImV4cCI6MTcyNDg4MzIzNCwiaWF0IjoxNzI0ODc5NjM0LCJqdGkiOiI2OTdkNGQ1Zi0zMDk4LTQwZWQtYjQ4OC03NjQ2YTI5NTk1MGQiLCJ1c2VybmFtZSI6ImRldm5ldHVzZXIifQ.Ka_tMNNoJJN9dudmEfM2bNEHBF6YlGG8VRJo22QXYTJ5QZAZkAzKT0U1YqtHDcHnPoeyaSddnp6Et6XazLBx0gq5LpJGMOhMrJz44GmzuxWNLwajgSm8WCt0MhenA9a9vQKdTlvds2AG6p7LZ0_NFHXR6w_5ZDsLXE-k_zibwGwunOeZuDYAuZJogciWw2T3-Jv0yxVTGqAE95Q53JuGk6sBtOtETXb3VACDkIeb9nTnYHIM1M-8D_RFzSFf_wsu8l0YeOcqAqLDPXYH4T5tGjSxRRAs0TIrIOSInTdisGRt7U9xu1poefeZPtFBIdslWNUQd2GnYdtEsU9xfnh1uw
{'content-type': 'application/json', 'X-Auth-Token': 'eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2Njk3NzQwYWU4ZTc5NDc0ZjcxN2FiNWEiLCJhdXRoU291cmNlIjoiaW50ZXJuYWwiLCJ0ZW5hbnROYW1lIjoiVE5UMCIsInJvbGVzIjpbIjY2OTZmMDFhYTA0Y2FlNjVjM2MzN2IwMiJdLCJ0ZW5hbnRJZCI6IjY2OTZmMDE4YTA0Y2FlNjVjM2MzN2FmYiIsImV4cCI6MTcyNDg4MzIzNCwiaWF0IjoxNzI0ODc5NjM0LCJqdGkiOiI2OTdkNGQ1Zi0zMDk4LTQwZWQtYjQ4OC03NjQ2YTI5NTk1MGQiLCJ1c2VybmFtZSI6ImRldm5ldHVzZXIifQ.Ka_tMNNoJJN9dudmEfM2bNEHBF6YlGG8VRJo22QXYTJ5QZAZkAzKT0U1YqtHDcHnPoeyaSddnp6Et6XazLBx0gq5LpJGMOhMrJz44GmzuxWNLwajgSm8WCt0MhenA9a9vQKdTlvds2AG6p7LZ0_NFHXR6w_5ZDsLXE-k_zibwGwunOeZuDYAuZJogciWw2T3-Jv0yxVTGqAE95Q53JuGk6sBtOtETXb3VACDkIeb9nTnYHIM1M-8D_RFzSFf_wsu8l0YeOcqAqLDPXYH4T5tGjSxRRAs0TIrIOSInTdisGRt7U9xu1poefeZPtFBIdslWNUQd2GnYdtEsU9xfnh1uw'}
True

```

```json
{"response":[{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845034708,"macAddress":"52:54:00:01:c2:c0","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA2O","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:18:23.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:37:14","bootDateTime":"2024-04-29 22:19:14","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw1","locationName":null,"managementIpAddress":"10.10.20.175","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450480,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"a29650a1-bca7-4913-8b02-7474f0e8215c","instanceTenantId":"6696f018a04cae65c3c37afb","id":"a29650a1-bca7-4913-8b02-7474f0e8215c"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845089964,"macAddress":"52:54:00:0e:1c:6a","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA21","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:19:09.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:38:09","bootDateTime":"2024-04-29 22:19:09","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw2","locationName":null,"managementIpAddress":"10.10.20.176","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450485,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"85ce77aa-3627-4d69-99ea-085d700cbd0f","instanceTenantId":"6696f018a04cae65c3c37afb","id":"85ce77aa-3627-4d69-99ea-085d700cbd0f"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845129449,"macAddress":"52:54:00:0a:1b:4c","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA22","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"6 days, 9:00:31.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:38:49","bootDateTime":"2024-08-22 02:38:49","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw3","locationName":null,"managementIpAddress":"10.10.20.177","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":585305,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"f2ee94ae-c1f7-4114-9a00-a4348240204f","instanceTenantId":"6696f018a04cae65c3c37afb","id":"f2ee94ae-c1f7-4114-9a00-a4348240204f"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845163741,"macAddress":"52:54:00:0f:25:4c","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA23","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:20:38.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:39:23","bootDateTime":"2024-04-29 22:19:23","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw4","locationName":null,"managementIpAddress":"10.10.20.178","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450471,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"04591e4f-de5e-4683-8b89-cb5dc5699df2","instanceTenantId":"6696f018a04cae65c3c37afb","id":"04591e4f-de5e-4683-8b89-cb5dc5699df2"}],"version":"1.0"}
{"response":[{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845034708,"macAddress":"52:54:00:01:c2:c0","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA2O","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:18:23.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:37:14","bootDateTime":"2024-04-29 22:19:14","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw1","locationName":null,"managementIpAddress":"10.10.20.175","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450480,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"a29650a1-bca7-4913-8b02-7474f0e8215c","instanceTenantId":"6696f018a04cae65c3c37afb","id":"a29650a1-bca7-4913-8b02-7474f0e8215c"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845089964,"macAddress":"52:54:00:0e:1c:6a","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA21","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:19:09.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:38:09","bootDateTime":"2024-04-29 22:19:09","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw2","locationName":null,"managementIpAddress":"10.10.20.176","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450485,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"85ce77aa-3627-4d69-99ea-085d700cbd0f","instanceTenantId":"6696f018a04cae65c3c37afb","id":"85ce77aa-3627-4d69-99ea-085d700cbd0f"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845129449,"macAddress":"52:54:00:0a:1b:4c","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA22","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"6 days, 9:00:31.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:38:49","bootDateTime":"2024-08-22 02:38:49","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw3","locationName":null,"managementIpAddress":"10.10.20.177","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":585305,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"f2ee94ae-c1f7-4114-9a00-a4348240204f","instanceTenantId":"6696f018a04cae65c3c37afb","id":"f2ee94ae-c1f7-4114-9a00-a4348240204f"},{"description":"Cisco IOS Software [Cupertino], Catalyst L3 Switch Software (CAT9KV_IOSXE), Experimental Version 17.9.20220318:182713 [BLD_POLARIS_DEV_S2C_20220318_081310-10-g847b433944c4:/nobackup/rajavenk/vikagarw/git_ws/polaris_dev 101] Copyright (c) 1986-2022 by Cis","lastUpdateTime":1724845163741,"macAddress":"52:54:00:0f:25:4c","deviceSupportLevel":"Supported","softwareType":"IOS-XE","softwareVersion":"17.9.20220318:182713","serialNumber":"9SB9FYAFA23","inventoryStatusDetail":"<status><general code=\"DEV_UNREACHED\"/></status>","managementState":"Managed","collectionInterval":"Global Default","upTime":"120 days, 13:20:38.00","roleSource":"AUTO","lastUpdated":"2024-08-28 11:39:23","bootDateTime":"2024-04-29 22:19:23","apManagerInterfaceIp":"","collectionStatus":"Partial Collection Failure","family":"Switches and Hubs","hostname":"sw4","locationName":null,"managementIpAddress":"10.10.20.178","platformId":"C9KV-UADP-8P","reachabilityFailureReason":"SNMP Connectivity Failed","reachabilityStatus":"Unreachable","series":"Cisco Catalyst 9000 Series Virtual Switches","snmpContact":"","snmpLocation":"","associatedWlcIp":"","apEthernetMacAddress":null,"errorCode":"DEV-UNREACHED","errorDescription":"NCIM12013: SNMP timeouts are occurring with this device. Either the SNMP credentials are not correctly provided to Cisco DNA Center or the device is responding slow and SNMP timeout is low. If it’s a timeout issue, Cisco DNA Center will attempt to progressively adjust the timeout in subsequent collection cycles to get device to managed state. User can also run discovery again only for this device using the discovery feature after adjusting the timeout and SNMP credentials as required. Or user can update the timeout and SNMP credentials as required using update credentials.","interfaceCount":"0","lineCardCount":"0","lineCardId":"","managedAtleastOnce":true,"memorySize":"NA","tagCount":"0","tunnelUdpPort":null,"uptimeSeconds":10450471,"waasDeviceMode":null,"type":"Cisco Catalyst 9000 UADP 8 Port Virtual Switch","location":null,"role":"ACCESS","instanceUuid":"04591e4f-de5e-4683-8b89-cb5dc5699df2","instanceTenantId":"6696f018a04cae65c3c37afb","id":"04591e4f-de5e-4683-8b89-cb5dc5699df2"}],"version":"1.0"}
(base) pradeep:~$
```

## Session-Based Authentication

```py
import requests
APIC_URL = "https://sandboxapicdc.cisco.com"
USER = "devnetuser"
PASS = "Cisco123!"
AUTH_BODY = {
    "aaaUser":{
        "attributes": {"name": USER, "pwd": PASS}
    }
}
LOGIN_URL = APIC_URL + "/api/aaaLogin.json"

session = requests.Session()
response = session.post(url=LOGIN_URL, json=AUTH_BODY, verify=False)
print(response.ok)
```

```sh
(base) pradeep:~$/usr/local/bin/python3 /Users/pradeep/LearnPython/ios_session_auth.py
/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages/urllib3/connectionpool.py:1099: InsecureRequestWarning: Unverified HTTPS request is being made to host 'sandboxapicdc.cisco.com'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#tls-warnings
  warnings.warn(
False
(base) pradeep:~$
```

After ignoring the warnings,

```py
```

```py
student@student-vm:~/labs/lab10/task01$ more task01_gather_data.py
#! /usr/bin/env python

import requests
import json

# Check for module dependencies:
not_installed_modules = []

try:
    from requests.auth import HTTPBasicAuth
except ImportError:
    not_installed_modules.append("requests")
try:
    import yaml
except ImportError:
    not_installed_modules.append("PyYAML")


if not_installed_modules:
    print("Please install following Python modules:")

    for module in not_installed_modules:
        print("  - {module}").format(module=module)

    sys.exit(1)



def get_inventory():
    inventory = {
        "csr1kv1": {"username": "cisco", "password": "cisco"},
        "csr1kv2": {"username": "cisco", "password": "cisco"},
    }

    return inventory


def get_hostname(host):
    url = "https://{host}/restconf/data/Cisco-IOS-XE-native:native" "/hostname".format(
        host=host
    )
    response = requests.get(url, headers=HTTP_HEADERS, auth=HTTP_AUTH, verify=False)
    result = json.loads(response.text)
    hostname = result.get("Cisco-IOS-XE-native:hostname")

    return hostname


def structure_data():

    devices = {}
    net_inventory = get_inventory()
    for host in net_inventory:
        devices[host] = {}
        devices[host]["hostname"] = get_hostname(host)
    return devices

def main():

    structured_data = structure_data()

    heading = "| {:^10} | {:^10} |".format(
        "Device", "Hostname"
    )
    print(len(heading) * "-")
    print(heading)
    print(len(heading) * "-")
    for hostname, details in structured_data.items():
        print(
            "| {:^10} | {:^10} |".format(
                hostname,
                details["hostname"],
            )
        )
        print(len(heading) * "-")

if __name__ == "__main__":
    main()

```

