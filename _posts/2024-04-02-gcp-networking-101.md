---
layout: single
title:  "GCP Networking 101"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-1.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/pca-gcp.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# GCP Networking

## Set Region and Zone
Run the following gcloud commands in Cloud Shell to set the default region and zone for your lab:
```

Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-01-75cafcd730d1.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud config set compute/zone "us-east1-d"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "us-east1"
export REGION=$(gcloud config get compute/region)
Updated property [compute/zone].
Your active configuration is: [cloudshell-25578]
Updated property [compute/region].
Your active configuration is: [cloudshell-25578]
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ echo $ZONE
us-east1-d
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ echo $REGION
us-east1
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## Create a Custom Network

When manually assigning subnetwork ranges, you first create a custom subnet network, then create the subnetworks that you want within a region. You do not have to specify subnetworks for all regions right away, or even at all, but you cannot create instances in regions that have no subnetwork defined.

When you create a new subnetwork, its name must be unique in that project for that region, even across networks. The same name can appear twice in a project as long as each one is in a different region. Because this is a subnetwork, there is no network-level IPv4 range or gateway IP, so none will be displayed.

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute networks create taw-custom-network --subnet-mode custom
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network].
NAME: taw-custom-network
SUBNET_MODE: CUSTOM
BGP_ROUTING_MODE: REGIONAL
IPV4_RANGE: 
GATEWAY_IPV4: 

Instances on this network will not be reachable until firewall rules
are created. As an example, you can allow all internal traffic between
instances as well as SSH, RDP, and ICMP by running:

$ gcloud compute firewall-rules create <FIREWALL_NAME> --network taw-custom-network --allow tcp,udp,icmp --source-ranges <IP_RANGE>
$ gcloud compute firewall-rules create <FIREWALL_NAME> --network taw-custom-network --allow tcp:22,tcp:3389,icmp

student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute networks subnets create subnet-us-east1 \
   --network taw-custom-network \
   --region us-east1 \
   --range 10.0.0.0/16
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/regions/us-east1/subnetworks/subnet-us-east1].
NAME: subnet-us-east1
REGION: us-east1
NETWORK: taw-custom-network
RANGE: 10.0.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute networks subnets create subnet-europe-west4 \
   --network taw-custom-network \
   --region europe-west4 \
   --range 10.1.0.0/16
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/regions/europe-west4/subnetworks/subnet-europe-west4].
NAME: subnet-europe-west4
REGION: europe-west4
NETWORK: taw-custom-network
RANGE: 10.1.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute networks subnets create subnet-us-west1 \
   --network taw-custom-network \
   --region us-west1 \
   --range 10.2.0.0/16
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/regions/us-west1/subnetworks/subnet-us-west1].
NAME: subnet-us-west1
REGION: us-west1
NETWORK: taw-custom-network
RANGE: 10.2.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```
## List Subnets of a Network
```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute networks subnets list \
   --network taw-custom-network
NAME: subnet-us-west1
REGION: us-west1
NETWORK: taw-custom-network
RANGE: 10.2.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: subnet-us-east1
REGION: us-east1
NETWORK: taw-custom-network
RANGE: 10.0.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 

NAME: subnet-europe-west4
REGION: europe-west4
NETWORK: taw-custom-network
RANGE: 10.1.0.0/16
STACK_TYPE: IPV4_ONLY
IPV6_ACCESS_TYPE: 
INTERNAL_IPV6_PREFIX: 
EXTERNAL_IPV6_PREFIX: 
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## Add Firewall Rules
To allow access to VM instances, you must apply firewall rules. You will use an instance tag to apply the firewall rule to your VM instances. The firewall rule will apply to any VM using the same instance tag.

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules create nw101-allow-http \
--allow tcp:80 --network taw-custom-network --source-ranges 0.0.0.0/0 \
--target-tags http
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-http].                              
Creating firewall...done.                                                                                                                                                          
NAME: nw101-allow-http
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules create "nw101-allow-icmp" --allow icmp --network "taw-custom-network" --target-tags rules
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-icmp].                              
Creating firewall...done.                                                                                                                                                          
NAME: nw101-allow-icmp
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp
DENY: 
DISABLED: False
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules create "nw101-allow-internal" --allow tcp:0-65535,udp:0-65535,icmp --network "taw-custom-network" --source-ranges "10.0.0.0/16","10.2.0.0/16","10.1.0.0/16"
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-internal].                          
Creating firewall...done.                                                                                                                                                          
NAME: nw101-allow-internal
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules create "nw101-allow-ssh" --allow tcp:22 --network "taw-custom-network" --target-tags "ssh"
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-ssh].                               
Creating firewall...done.                                                                                                                                                          
NAME: nw101-allow-ssh
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules create "nw101-allow-rdp" --allow tcp:3389 --network "taw-custom-network"
Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-rdp].                               
Creating firewall...done.                                                                                                                                                          
NAME: nw101-allow-rdp
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY: 
DISABLED: False
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## List Firewall Rules
```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules list
NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY: 
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY: 
DISABLED: False

NAME: nw101-allow-http
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False

NAME: nw101-allow-icmp
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp
DENY: 
DISABLED: False

NAME: nw101-allow-internal
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: nw101-allow-rdp
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: nw101-allow-ssh
NETWORK: taw-custom-network
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```json
tudent_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute firewall-rules list --format=json
[
  {
    "allowed": [
      {
        "IPProtocol": "icmp"
      }
    ],
    "creationTimestamp": "2024-03-29T09:48:12.341-07:00",
    "description": "Allow ICMP from anywhere",
    "direction": "INGRESS",
    "disabled": false,
    "id": "9180459003064045091",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "default-allow-icmp",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/default",
    "priority": 65534,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/default-allow-icmp",
    "sourceRanges": [
      "0.0.0.0/0"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "0-65535"
        ]
      },
      {
        "IPProtocol": "udp",
        "ports": [
          "0-65535"
        ]
      },
      {
        "IPProtocol": "icmp"
      }
    ],
    "creationTimestamp": "2024-03-29T09:48:12.123-07:00",
    "description": "Allow internal traffic on the default network",
    "direction": "INGRESS",
    "disabled": false,
    "id": "6695118053441228323",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "default-allow-internal",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/default",
    "priority": 65534,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/default-allow-internal",
    "sourceRanges": [
      "10.128.0.0/9"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "3389"
        ]
      }
    ],
    "creationTimestamp": "2024-03-29T09:48:12.268-07:00",
    "description": "Allow RDP from anywhere",
    "direction": "INGRESS",
    "disabled": false,
    "id": "2183360268835966499",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "default-allow-rdp",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/default",
    "priority": 65534,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/default-allow-rdp",
    "sourceRanges": [
      "0.0.0.0/0"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "22"
        ]
      }
    ],
    "creationTimestamp": "2024-03-29T09:48:12.195-07:00",
    "description": "Allow SSH from anywhere",
    "direction": "INGRESS",
    "disabled": false,
    "id": "7994936008031030819",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "default-allow-ssh",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/default",
    "priority": 65534,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/default-allow-ssh",
    "sourceRanges": [
      "0.0.0.0/0"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "80"
        ]
      }
    ],
    "creationTimestamp": "2024-04-02T08:34:31.209-07:00",
    "description": "",
    "direction": "INGRESS",
    "disabled": false,
    "id": "8924984757635357544",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "nw101-allow-http",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network",
    "priority": 1000,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-http",
    "sourceRanges": [
      "0.0.0.0/0"
    ],
    "targetTags": [
      "http"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "icmp"
      }
    ],
    "creationTimestamp": "2024-04-02T08:35:01.499-07:00",
    "description": "",
    "direction": "INGRESS",
    "disabled": false,
    "id": "5428300527314246474",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "nw101-allow-icmp",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network",
    "priority": 1000,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-icmp",
    "sourceRanges": [
      "0.0.0.0/0"
    ],
    "targetTags": [
      "rules"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "0-65535"
        ]
      },
      {
        "IPProtocol": "udp",
        "ports": [
          "0-65535"
        ]
      },
      {
        "IPProtocol": "icmp"
      }
    ],
    "creationTimestamp": "2024-04-02T08:35:31.596-07:00",
    "description": "",
    "direction": "INGRESS",
    "disabled": false,
    "id": "2748152024492251948",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "nw101-allow-internal",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network",
    "priority": 1000,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-internal",
    "sourceRanges": [
      "10.0.0.0/16",
      "10.2.0.0/16",
      "10.1.0.0/16"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "3389"
        ]
      }
    ],
    "creationTimestamp": "2024-04-02T08:36:32.962-07:00",
    "description": "",
    "direction": "INGRESS",
    "disabled": false,
    "id": "4443461897821460719",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "nw101-allow-rdp",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network",
    "priority": 1000,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-rdp",
    "sourceRanges": [
      "0.0.0.0/0"
    ]
  },
  {
    "allowed": [
      {
        "IPProtocol": "tcp",
        "ports": [
          "22"
        ]
      }
    ],
    "creationTimestamp": "2024-04-02T08:36:02.395-07:00",
    "description": "",
    "direction": "INGRESS",
    "disabled": false,
    "id": "8622350822109453069",
    "kind": "compute#firewall",
    "logConfig": {
      "enable": false
    },
    "name": "nw101-allow-ssh",
    "network": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/networks/taw-custom-network",
    "priority": 1000,
    "selfLink": "https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/global/firewalls/nw101-allow-ssh",
    "sourceRanges": [
      "0.0.0.0/0"
    ],
    "targetTags": [
      "ssh"
    ]
  }
]
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## Create a VM in each Zone

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute instances create us-test-01 \
--subnet subnet-us-east1 \
--zone us-east1-d \
--machine-type e2-standard-2 \
--tags ssh,http,rules
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/zones/us-east1-d/instances/us-test-01].
NAME: us-test-01
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.0.0.2
EXTERNAL_IP: 35.231.193.203
STATUS: RUNNING
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute instances create us-test-02 \
--subnet subnet-europe-west4 \
--zone europe-west4-b \
--machine-type e2-standard-2 \
--tags ssh,http,rules
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/zones/europe-west4-b/instances/us-test-02].
NAME: us-test-02
ZONE: europe-west4-b
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.1.0.2
EXTERNAL_IP: 34.147.17.122
STATUS: RUNNING
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute instances create us-test-03 \
--subnet subnet-us-west1 \
--zone us-west1-b \
--machine-type e2-standard-2 \
--tags ssh,http,rules
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/zones/us-west1-b/instances/us-test-03].
NAME: us-test-03
ZONE: us-west1-b
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.2.0.2
EXTERNAL_IP: 104.198.106.2
STATUS: RUNNING
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## List VMs
```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute instances list
NAME: us-test-03
ZONE: us-west1-b
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.2.0.2
EXTERNAL_IP: 104.198.106.2
STATUS: RUNNING

NAME: us-test-01
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.0.0.2
EXTERNAL_IP: 35.231.193.203
STATUS: RUNNING

NAME: us-test-02
ZONE: europe-west4-b
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.1.0.2
EXTERNAL_IP: 34.147.17.122
STATUS: RUNNING
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

## SSH to an Instance
```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute ssh --zone "us-east1-d" "us-test-01" --project "qwiklabs-gcp-01-75cafcd730d1"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_00_8b7e8d92f879/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_00_8b7e8d92f879/.ssh/google_compute_engine
Your public key has been saved in /home/student_00_8b7e8d92f879/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:0NjAz3d7oT5BonQxdegGa51XJT5BVqHopEtGzNTTqE8 student_00_8b7e8d92f879@cs-1027064170958-default
The key's randomart image is:
+---[RSA 3072]----+
|     ..  ...+o*o=|
|      .=+ ++o=.o.|
|      ooo+.X.oo. |
|       .=.XEB o. |
|       .SOo* + . |
|        + ..+ .  |
|         . . o   |
|            o    |
|             .   |
+----[SHA256]-----+
Warning: Permanently added 'compute.6373579686217357312' (ECDSA) to the list of known hosts.
Linux us-test-01 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Apr  2 15:46:19 2024 from 172.217.41.163
student-00-8b7e8d92f879@us-test-01:~$ 
```

## Ping Internal and External IPs of other Instances
```sh
student-00-8b7e8d92f879@us-test-01:~$ ping -c 3 34.147.17.122
PING 34.147.17.122 (34.147.17.122) 56(84) bytes of data.
64 bytes from 34.147.17.122: icmp_seq=1 ttl=54 time=99.2 ms
64 bytes from 34.147.17.122: icmp_seq=2 ttl=54 time=97.9 ms
64 bytes from 34.147.17.122: icmp_seq=3 ttl=54 time=97.9 ms

--- 34.147.17.122 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 97.854/98.334/99.248/0.646 ms
student-00-8b7e8d92f879@us-test-01:~$ ping -c 3 104.198.106.2
PING 104.198.106.2 (104.198.106.2) 56(84) bytes of data.
64 bytes from 104.198.106.2: icmp_seq=1 ttl=52 time=65.8 ms
64 bytes from 104.198.106.2: icmp_seq=2 ttl=52 time=64.8 ms
64 bytes from 104.198.106.2: icmp_seq=3 ttl=52 time=64.8 ms

--- 104.198.106.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 64.761/65.113/65.768/0.463 ms
student-00-8b7e8d92f879@us-test-01:~$ ping -c 3 10.1.0.2
PING 10.1.0.2 (10.1.0.2) 56(84) bytes of data.
64 bytes from 10.1.0.2: icmp_seq=1 ttl=64 time=98.5 ms
64 bytes from 10.1.0.2: icmp_seq=2 ttl=64 time=97.6 ms
64 bytes from 10.1.0.2: icmp_seq=3 ttl=64 time=97.6 ms

--- 10.1.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 97.571/97.889/98.455/0.401 ms
student-00-8b7e8d92f879@us-test-01:~$ ping -c 3 10.2.0.2
PING 10.2.0.2 (10.2.0.2) 56(84) bytes of data.
64 bytes from 10.2.0.2: icmp_seq=1 ttl=64 time=65.5 ms
64 bytes from 10.2.0.2: icmp_seq=2 ttl=64 time=64.7 ms
64 bytes from 10.2.0.2: icmp_seq=3 ttl=64 time=64.7 ms

--- 10.2.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 64.677/64.942/65.472/0.374 ms
student-00-8b7e8d92f879@us-test-01:~$ 
```

## DNS Names

Each instance has a metadata server that also acts as a DNS resolver for that instance. DNS lookups are performed for instance names. The metadata server itself stores all DNS information for the local network and queries Google's public DNS servers for any addresses outside of the local network.

An internal fully qualified domain name (FQDN) for an instance looks like this: hostName.[ZONE].c.[PROJECT_ID].internal .

You can always connect from one instance to another using this FQDN. If you want to connect to an instance using, for example, just hostName, you need information from the internal DNS resolver that is provided as part of Compute Engine. 

```sh
student-00-8b7e8d92f879@us-test-01:~$ 
ping -c 3 us-test-02.europe-west4-b
PING us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.1.0.2) 56(84) bytes of data.
64 bytes from us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.1.0.2): icmp_seq=1 ttl=64 time=99.4 ms
64 bytes from us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.1.0.2): icmp_seq=2 ttl=64 time=97.3 ms
64 bytes from us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.1.0.2): icmp_seq=3 ttl=64 time=97.2 ms

--- us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 97.238/97.964/99.369/0.993 ms
student-00-8b7e8d92f879@us-test-01:~$ 
```

## Traceroute 
Install necessary packages

```sh
student-00-8b7e8d92f879@us-test-01:~$ history 
    1  ping -c 3 34.147.17.122
    2  ping -c 3 104.198.106.2
    3  ping -c 3 10.1.0.2
    4  ping -c 3 10.2.0.2
    5  ping -c 3 us-test-02.europe-west4-b
    6  sudo apt-get update
    7  sudo apt-get -y install traceroute mtr tcpdump iperf whois host dnsutils siege
    8  history 
student-00-8b7e8d92f879@us-test-01:~$ 
```

```sh
raceroute to www.icann.org (192.0.32.7), 30 hops max, 60 byte packets
 1  * * *
 2  216.239.47.208 (216.239.47.208)  12.746 ms  13.016 ms  12.700 ms
 3  ae19.cr1-was1.ip4.gtt.net (69.174.23.133)  12.987 ms  13.839 ms  12.913 ms
 4  ae14.cr5-lax2.ip4.gtt.net (89.149.180.234)  66.894 ms  66.746 ms  66.774 ms
 5  ip4.gtt.net (69.174.9.218)  66.848 ms  66.745 ms  67.180 ms
 6  www.icann.org (192.0.32.7)  67.165 ms  66.555 ms  66.630 ms
student-00-8b7e8d92f879@us-test-01:~$ 
```
### Tracroute using Internal IPs 
```sh
student-00-8b7e8d92f879@us-test-01:~$ traceroute 10.1.0.2
traceroute to 10.1.0.2 (10.1.0.2), 30 hops max, 60 byte packets
 1  * us-test-02.europe-west4-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.1.0.2)  99.270 ms  99.251 ms
student-00-8b7e8d92f879@us-test-01:~$ traceroute 10.2.0.2
traceroute to 10.2.0.2 (10.2.0.2), 30 hops max, 60 byte packets
 1  us-test-03.us-west1-b.c.qwiklabs-gcp-01-75cafcd730d1.internal (10.2.0.2)  62.720 ms  62.682 ms  62.663 ms
student-00-8b7e8d92f879@us-test-01:~$ 
```

## Test Performance
Between two hosts
When you use iperf to test the performance between two hosts, one side needs to be set up as the iperf server to accept connections.

```sh
tudent-00-8b7e8d92f879@us-test-02:~$ history 
    1  sudo apt-get update
    2  sudo apt-get -y install traceroute mtr tcpdump iperf whois host dnsutils siege
    3  history 
student-00-8b7e8d92f879@us-test-02:~$ 
```

SSH into us-test-01 and run:
```sh
student-00-8b7e8d92f879@us-test-01:~$ iperf -s #run in server mode
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
```

On us-test-02 SSH run this iperf:

```sh
student-00-8b7e8d92f879@us-test-02:~$ iperf -c us-test-01.us-east1-d #run in client mode
------------------------------------------------------------
Client connecting to us-test-01.us-east1-d, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.1.0.2 port 37322 connected with 10.0.0.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0358 sec   281 MBytes   235 Mbits/sec
student-00-8b7e8d92f879@us-test-02:~$ 
```

```sh
tudent-00-8b7e8d92f879@us-test-01:~$ iperf -s #run in server mode
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 37322
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0359 sec   281 MBytes   235 Mbits/sec
^Cstudent-00-8b7e8d92f879@us-test-01:~$ 
```

### Between VMs within a region

Now  deploy another instance (us-test-04)in a different zone than us-test-01. You will see that within a region, the bandwidth is limited by the 2 Gbit/s per core egress cap.

Between regions you reach much lower limits, mostly due to limits on TCP window size and single stream performance. You can increase bandwidth between hosts by using other parameters, like UDP.

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute instances create us-test-04 \
--subnet subnet-us-east1 \
--zone us-east1-b \
--tags ssh,http
Created [https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-01-75cafcd730d1/zones/us-east1-b/instances/us-test-04].
NAME: us-test-04
ZONE: us-east1-b
MACHINE_TYPE: n1-standard-1
PREEMPTIBLE: 
INTERNAL_IP: 10.0.0.3
EXTERNAL_IP: 34.138.134.20
STATUS: RUNNING
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ 
```

```sh
student_00_8b7e8d92f879@cloudshell:~ (qwiklabs-gcp-01-75cafcd730d1)$ gcloud compute ssh --zone "us-east1-b" "us-test-04" --project "qwiklabs-gcp-01-75cafcd730d1"
Warning: Permanently added 'compute.2517750728691389765' (ECDSA) to the list of known hosts.
Linux us-test-04 5.10.0-28-cloud-amd64 #1 SMP Debian 5.10.209-2 (2024-01-31) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-00-8b7e8d92f879'.
student-00-8b7e8d92f879@us-test-04:~$ 
```

On us-test-02 SSH run:
```sh
student-00-8b7e8d92f879@us-test-02:~$ iperf -s -u #iperf server side
------------------------------------------------------------
Server listening on UDP port 5001
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
```
On us-test-01 SSH run:

```sh
student-00-8b7e8d92f879@us-test-01:~$ iperf -c us-test-02.europe-west4-b -u -b 2G #iperf client side - send 2 Gbits/s
------------------------------------------------------------
Client connecting to us-test-02.europe-west4-b, UDP port 5001
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.2 port 59102 connected with 10.1.0.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0000 sec  2.50 GBytes  2.15 Gbits/sec
[  3] Sent 1826101 datagrams
[  3] Server Report:
[ ID] Interval       Transfer     Bandwidth        Jitter   Lost/Total Datagrams
[  3] 0.0000-10.0023 sec  1.17 GBytes  1.01 Gbits/sec   0.010 ms 969615/1826100 (53%)
student-00-8b7e8d92f879@us-test-01:~$ 
```

```sh
student-00-8b7e8d92f879@us-test-02:~$ iperf -s -u #iperf server side
------------------------------------------------------------
Server listening on UDP port 5001
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 10.1.0.2 port 5001 connected with 10.0.0.2 port 59102
[ ID] Interval       Transfer     Bandwidth        Jitter   Lost/Total Datagrams
[  3] 0.0000-10.0023 sec  1.17 GBytes  1.01 Gbits/sec   0.010 ms 969615/1826100 (53%)

^Cstudent-00-8b7e8d92f879@us-test-02:~$ 
```

This should be able to achieve a higher speed between EU and US. Even higher speeds can be achieved by running a bunch of TCP iperfs in parallel.

In the SSH window for us-test-01 run:
```sh
tudent-00-8b7e8d92f879@us-test-02:~$ iperf -c us-test-01.us-east1-d -P 20
[  5] local 10.1.0.2 port 51636 connected with 10.0.0.2 port 5001
[ 13] local 10.1.0.2 port 51650 connected with 10.0.0.2 port 5001
[  6] local 10.1.0.2 port 51656 connected with 10.0.0.2 port 5001
[ 11] local 10.1.0.2 port 51672 connected with 10.0.0.2 port 5001
[  7] local 10.1.0.2 port 51700 connected with 10.0.0.2 port 5001
[ 15] local 10.1.0.2 port 51684 connected with 10.0.0.2 port 5001
[ 14] local 10.1.0.2 port 51724 connected with 10.0.0.2 port 5001
[  4] local 10.1.0.2 port 51688 connected with 10.0.0.2 port 5001
[ 18] local 10.1.0.2 port 51738 connected with 10.0.0.2 port 5001
------------------------------------------------------------
Client connecting to us-test-01.us-east1-d, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[ 17] local 10.1.0.2 port 51750 connected with 10.0.0.2 port 5001
[ 10] local 10.1.0.2 port 51758 connected with 10.0.0.2 port 5001
[  9] local 10.1.0.2 port 51710 connected with 10.0.0.2 port 5001
[  3] local 10.1.0.2 port 51742 connected with 10.0.0.2 port 5001
[ 22] local 10.1.0.2 port 51762 connected with 10.0.0.2 port 5001
[ 19] local 10.1.0.2 port 51778 connected with 10.0.0.2 port 5001
[ 16] local 10.1.0.2 port 51784 connected with 10.0.0.2 port 5001
[  8] local 10.1.0.2 port 51790 connected with 10.0.0.2 port 5001
[ 20] local 10.1.0.2 port 51794 connected with 10.0.0.2 port 5001
[ 21] local 10.1.0.2 port 51804 connected with 10.0.0.2 port 5001
[ 12] local 10.1.0.2 port 51818 connected with 10.0.0.2 port 5001
[ ID] Interval       Transfer     Bandwidth
[ 11] 0.0000-10.0068 sec   236 MBytes   198 Mbits/sec
[ 15] 0.0000-10.0016 sec   258 MBytes   216 Mbits/sec
[ 22] 0.0000-10.0059 sec   229 MBytes   192 Mbits/sec
[ 18] 0.0000-10.0116 sec   211 MBytes   177 Mbits/sec
[  7] 0.0000-10.0048 sec   211 MBytes   177 Mbits/sec
[ 16] 0.0000-10.0160 sec   251 MBytes   210 Mbits/sec
[ 20] 0.0000-10.0025 sec   211 MBytes   177 Mbits/sec
[ 10] 0.0000-10.0285 sec   242 MBytes   203 Mbits/sec
[ 17] 0.0000-10.0246 sec   212 MBytes   177 Mbits/sec
[ 21] 0.0000-10.0066 sec   211 MBytes   177 Mbits/sec
[  4] 0.0000-10.0010 sec   209 MBytes   176 Mbits/sec
[  5] 0.0000-10.0352 sec   251 MBytes   209 Mbits/sec
[ 19] 0.0000-10.0391 sec   254 MBytes   212 Mbits/sec
[ 14] 0.0000-10.0210 sec   212 MBytes   177 Mbits/sec
[  9] 0.0000-10.0073 sec   210 MBytes   176 Mbits/sec
[ 13] 0.0000-10.0235 sec   211 MBytes   177 Mbits/sec
[ 12] 0.0000-10.0463 sec   211 MBytes   176 Mbits/sec
[  6] 0.0000-10.0522 sec   212 MBytes   177 Mbits/sec
[  8] 0.0000-10.0086 sec   210 MBytes   176 Mbits/sec
[  3] 0.0000-10.0146 sec   210 MBytes   176 Mbits/sec
[SUM] 0.0000-10.0161 sec  4.36 GBytes  3.74 Gbits/sec
[ CT] final connect times (min/avg/max/stdev) = 95.605/96.448/98.646/22.000 ms (tot/err) = 20/0
student-00-8b7e8d92f879@us-test-02:~$ 
```
```sh
tudent-00-8b7e8d92f879@us-test-01:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51636
[  5] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51650
[  6] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51656
[  7] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51672
[  8] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51700
[ 10] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51688
[ 12] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51684
[ 13] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51750
[ 14] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51758
[ 15] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51762
[ 16] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51710
[ 17] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51742
[ 18] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51778
[ 19] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51784
[ 20] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51790
[ 21] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51794
[ 22] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51804
[ 23] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51818
[  9] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51724
[ 11] local 10.0.0.2 port 5001 connected with 10.1.0.2 port 51738
[ ID] Interval       Transfer     Bandwidth
[  7] 0.0000-10.0056 sec   236 MBytes   198 Mbits/sec
[ 12] 0.0000-10.0003 sec   258 MBytes   216 Mbits/sec
[ 15] 0.0000-10.0035 sec   229 MBytes   192 Mbits/sec
[  8] 0.0000-10.0250 sec   211 MBytes   177 Mbits/sec
[ 10] 0.0000-10.0212 sec   209 MBytes   175 Mbits/sec
[ 14] 0.0000-10.0256 sec   242 MBytes   203 Mbits/sec
[ 19] 0.0000-10.0121 sec   251 MBytes   210 Mbits/sec
[ 21] 0.0000-10.0206 sec   211 MBytes   176 Mbits/sec
[ 11] 0.0000-10.0263 sec   211 MBytes   177 Mbits/sec
[ 16] 0.0000-10.0245 sec   210 MBytes   176 Mbits/sec
[ 18] 0.0000-10.0344 sec   254 MBytes   212 Mbits/sec
[ 22] 0.0000-10.0236 sec   211 MBytes   177 Mbits/sec
[  4] 0.0000-10.4477 sec   251 MBytes   201 Mbits/sec
[  5] 0.0000-10.0330 sec   211 MBytes   177 Mbits/sec
[  9] 0.0000-10.0311 sec   212 MBytes   177 Mbits/sec
[ 17] 0.0000-10.0278 sec   210 MBytes   176 Mbits/sec
[ 13] 0.0000-10.0297 sec   212 MBytes   177 Mbits/sec
[  6] 0.0000-10.0518 sec   212 MBytes   177 Mbits/sec
[ 23] 0.0000-10.0421 sec   211 MBytes   176 Mbits/sec
[ 20] 0.0000-10.0255 sec   210 MBytes   176 Mbits/sec
[SUM] 0.0000-10.4432 sec  4.36 GBytes  3.58 Gbits/sec
^Cstudent-00-8b7e8d92f879@us-test-01:~$ 
```
The combined bandwidth should be really close to the maximum achievable bandwidth.

As you can see, to reach the maximum bandwidth, just running a single TCP stream (for example, file copy) is not sufficient; you need to have several TCP sessions in parallel. Reasons are: TCP parameters such as Window Size; and functions such as Slow Start.

Tools like `bbcp` can help to copy files as fast as possible by parallelizing transfers and using configurable window size.


