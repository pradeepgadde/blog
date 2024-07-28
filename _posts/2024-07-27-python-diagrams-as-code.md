---
layout: single
title:  "Diagrams as Code"
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
# Diagrams as Code

Diagrams lets you draw the cloud system architecture **in Python code**.

`Diagram as Code` allows you to **track** the architecture diagram changes in any **version control** system.

Diagrams currently supports main major providers including: `AWS`, `Azure`, `GCP`, `Kubernetes`, `Alibaba Cloud`, `Oracle Cloud` etc... It also supports `On-Premise` nodes, `SaaS` and major `Programming` frameworks and languages.

## Installation

It requires Python 3.6 or higher, check your Python version first.

It uses Graphviz to render the diagram, so you need to install Graphviz to use diagrams. After installing graphviz (or already have it), install the diagrams.

```python
👋 Welcome to Codespaces! You are on our default image. 
   - It includes runtimes and tools for Python, Node.js, Docker, and more. See the full list here: https://aka.ms/ghcs-default-image
   - Want to use a custom image instead? Learn more here: https://aka.ms/configure-codespace

🔍 To explore VS Code to its fullest, search using the Command Palette (Cmd/Ctrl + Shift + P or F1).

📝 Edit away, run your app as usual, and we'll automatically make it available for you to access.


@pradeepgadde ➜ /workspaces/codespaces-blank $ 
@pradeepgadde ➜ /workspaces/codespaces-blank $ uname -a
Linux codespaces-4d67cb 6.5.0-1022-azure #23~22.04.1-Ubuntu SMP Thu May  9 17:59:24 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
@pradeepgadde ➜ /workspaces/codespaces-blank $ sudo apt install graphviz
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package graphviz
@pradeepgadde ➜ /workspaces/codespaces-blank $ sudo apt-get update
Get:1 https://packages.microsoft.com/repos/microsoft-ubuntu-focal-prod focal InRelease [3632 B]
Get:2 https://dl.yarnpkg.com/debian stable InRelease [17.1 kB]                                                          
Get:3 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]                                                         
Get:4 https://repo.anaconda.com/pkgs/misc/debrepo/conda stable InRelease [3961 B]                                       
Get:5 http://security.ubuntu.com/ubuntu focal-security InRelease [128 kB]                                               
Get:6 https://packages.microsoft.com/repos/microsoft-ubuntu-focal-prod focal/main amd64 Packages [303 kB]        
Get:7 https://packages.microsoft.com/repos/microsoft-ubuntu-focal-prod focal/main all Packages [2714 B]                 
Get:8 https://dl.yarnpkg.com/debian stable/main all Packages [11.8 kB]                                                  
Get:9 https://dl.yarnpkg.com/debian stable/main amd64 Packages [11.8 kB]                                                
Get:10 https://repo.anaconda.com/pkgs/misc/debrepo/conda stable/main amd64 Packages [4557 B]                            
Get:12 http://archive.ubuntu.com/ubuntu focal-updates InRelease [128 kB]                                                
Get:13 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [3805 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-backports InRelease [128 kB]    
Get:15 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]          
Get:16 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]              
Get:11 https://packagecloud.io/github/git-lfs/ubuntu focal InRelease [28.0 kB]                                 
Get:17 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]          
Get:18 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [1252 kB]                            
Get:19 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [3836 kB]                                   
Get:20 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [30.9 kB]                             
Get:21 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]                                
Get:22 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [33.5 kB]
Get:23 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [4302 kB]
Get:25 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1539 kB]
Get:26 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [3957 kB]
Get:27 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [55.2 kB]
Get:28 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [28.6 kB]
Get:24 https://packagecloud.io/github/git-lfs/ubuntu focal/main amd64 Packages [3690 B]
Fetched 32.7 MB in 3s (12.4 MB/s)                           
Reading package lists... Done
@pradeepgadde ➜ /workspaces/codespaces-blank $ sudo apt install graphviz
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  fonts-liberation libann0 libcdt5 libcgraph6 libgd3 libgts-0.7-5 libgts-bin libgvc6 libgvpr2 liblab-gamut1
  libpathplan4 libxaw7 libxmu6 libxpm4
Suggested packages:
  gsfonts graphviz-doc libgd-tools
The following NEW packages will be installed:
  fonts-liberation graphviz libann0 libcdt5 libcgraph6 libgd3 libgts-0.7-5 libgts-bin libgvc6 libgvpr2 liblab-gamut1
  libpathplan4 libxaw7 libxmu6 libxpm4
0 upgraded, 15 newly installed, 0 to remove and 13 not upgraded.
Need to get 3074 kB of archives.
After this operation, 12.5 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 fonts-liberation all 1:1.07.4-11 [822 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/universe amd64 libann0 amd64 1.1.2+doc-7build1 [26.0 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/universe amd64 libcdt5 amd64 2.42.2-3build2 [18.7 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/universe amd64 libcgraph6 amd64 2.42.2-3build2 [41.3 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libxpm4 amd64 1:3.5.12-1ubuntu0.20.04.2 [34.9 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 libgd3 amd64 2.2.5-5.2ubuntu2.1 [118 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/universe amd64 libgts-0.7-5 amd64 0.7.6+darcs121130-4 [150 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/universe amd64 libpathplan4 amd64 2.42.2-3build2 [21.9 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/universe amd64 libgvc6 amd64 2.42.2-3build2 [647 kB]
Get:10 http://archive.ubuntu.com/ubuntu focal/universe amd64 libgvpr2 amd64 2.42.2-3build2 [167 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/universe amd64 liblab-gamut1 amd64 2.42.2-3build2 [177 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/main amd64 libxmu6 amd64 2:1.1.3-0ubuntu1 [45.8 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal/main amd64 libxaw7 amd64 2:1.0.13-1 [173 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal/universe amd64 graphviz amd64 2.42.2-3build2 [590 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal/universe amd64 libgts-bin amd64 0.7.6+darcs121130-4 [41.3 kB]
Fetched 3074 kB in 0s (16.6 MB/s) 
Selecting previously unselected package fonts-liberation.
(Reading database ... 70090 files and directories currently installed.)
Preparing to unpack .../00-fonts-liberation_1%3a1.07.4-11_all.deb ...
Unpacking fonts-liberation (1:1.07.4-11) ...
Selecting previously unselected package libann0.
Preparing to unpack .../01-libann0_1.1.2+doc-7build1_amd64.deb ...
Unpacking libann0 (1.1.2+doc-7build1) ...
Selecting previously unselected package libcdt5:amd64.
Preparing to unpack .../02-libcdt5_2.42.2-3build2_amd64.deb ...
Unpacking libcdt5:amd64 (2.42.2-3build2) ...
Selecting previously unselected package libcgraph6:amd64.
Preparing to unpack .../03-libcgraph6_2.42.2-3build2_amd64.deb ...
Unpacking libcgraph6:amd64 (2.42.2-3build2) ...
Selecting previously unselected package libxpm4:amd64.
Preparing to unpack .../04-libxpm4_1%3a3.5.12-1ubuntu0.20.04.2_amd64.deb ...
Unpacking libxpm4:amd64 (1:3.5.12-1ubuntu0.20.04.2) ...
Selecting previously unselected package libgd3:amd64.
Preparing to unpack .../05-libgd3_2.2.5-5.2ubuntu2.1_amd64.deb ...
Unpacking libgd3:amd64 (2.2.5-5.2ubuntu2.1) ...
Selecting previously unselected package libgts-0.7-5:amd64.
Preparing to unpack .../06-libgts-0.7-5_0.7.6+darcs121130-4_amd64.deb ...
Unpacking libgts-0.7-5:amd64 (0.7.6+darcs121130-4) ...
Selecting previously unselected package libpathplan4:amd64.
Preparing to unpack .../07-libpathplan4_2.42.2-3build2_amd64.deb ...
Unpacking libpathplan4:amd64 (2.42.2-3build2) ...
Selecting previously unselected package libgvc6.
Preparing to unpack .../08-libgvc6_2.42.2-3build2_amd64.deb ...
Unpacking libgvc6 (2.42.2-3build2) ...
Selecting previously unselected package libgvpr2:amd64.
Preparing to unpack .../09-libgvpr2_2.42.2-3build2_amd64.deb ...
Unpacking libgvpr2:amd64 (2.42.2-3build2) ...
Selecting previously unselected package liblab-gamut1:amd64.
Preparing to unpack .../10-liblab-gamut1_2.42.2-3build2_amd64.deb ...
Unpacking liblab-gamut1:amd64 (2.42.2-3build2) ...
Selecting previously unselected package libxmu6:amd64.
Preparing to unpack .../11-libxmu6_2%3a1.1.3-0ubuntu1_amd64.deb ...
Unpacking libxmu6:amd64 (2:1.1.3-0ubuntu1) ...
Selecting previously unselected package libxaw7:amd64.
Preparing to unpack .../12-libxaw7_2%3a1.0.13-1_amd64.deb ...
Unpacking libxaw7:amd64 (2:1.0.13-1) ...
Selecting previously unselected package graphviz.
Preparing to unpack .../13-graphviz_2.42.2-3build2_amd64.deb ...
Unpacking graphviz (2.42.2-3build2) ...
Selecting previously unselected package libgts-bin.
Preparing to unpack .../14-libgts-bin_0.7.6+darcs121130-4_amd64.deb ...
Unpacking libgts-bin (0.7.6+darcs121130-4) ...
Setting up libxmu6:amd64 (2:1.1.3-0ubuntu1) ...
Setting up libxpm4:amd64 (1:3.5.12-1ubuntu0.20.04.2) ...
Setting up liblab-gamut1:amd64 (2.42.2-3build2) ...
Setting up libxaw7:amd64 (2:1.0.13-1) ...
Setting up libgts-0.7-5:amd64 (0.7.6+darcs121130-4) ...
Setting up libpathplan4:amd64 (2.42.2-3build2) ...
Setting up libann0 (1.1.2+doc-7build1) ...
Setting up libgd3:amd64 (2.2.5-5.2ubuntu2.1) ...
Setting up fonts-liberation (1:1.07.4-11) ...
Setting up libcdt5:amd64 (2.42.2-3build2) ...
Setting up libcgraph6:amd64 (2.42.2-3build2) ...
Setting up libgts-bin (0.7.6+darcs121130-4) ...
Setting up libgvc6 (2.42.2-3build2) ...
Setting up libgvpr2:amd64 (2.42.2-3build2) ...
Setting up graphviz (2.42.2-3build2) ...
Processing triggers for libc-bin (2.31-0ubuntu9.16) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for fontconfig (2.13.1-2ubuntu3) ...
@pradeepgadde ➜ /workspaces/codespaces-blank $ 


```



```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ pip install diagrams
Collecting diagrams
  Downloading diagrams-0.23.4-py3-none-any.whl.metadata (7.0 kB)
Collecting graphviz<0.21.0,>=0.13.2 (from diagrams)
  Downloading graphviz-0.20.3-py3-none-any.whl.metadata (12 kB)
Requirement already satisfied: jinja2<4.0,>=2.10 in /home/codespace/.local/lib/python3.10/site-packages (from diagrams) (3.1.4)
Collecting typed-ast<2.0.0,>=1.5.4 (from diagrams)
  Downloading typed_ast-1.5.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (1.7 kB)
Requirement already satisfied: MarkupSafe>=2.0 in /home/codespace/.local/lib/python3.10/site-packages (from jinja2<4.0,>=2.10->diagrams) (2.1.5)
Downloading diagrams-0.23.4-py3-none-any.whl (24.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 24.6/24.6 MB 38.8 MB/s eta 0:00:00
Downloading graphviz-0.20.3-py3-none-any.whl (47 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 47.1/47.1 kB 1.2 MB/s eta 0:00:00
Downloading typed_ast-1.5.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (824 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 824.7/824.7 kB 19.0 MB/s eta 0:00:00
Installing collected packages: typed-ast, graphviz, diagrams
Successfully installed diagrams-0.23.4 graphviz-0.20.3 typed-ast-1.5.5
@pradeepgadde ➜ /workspaces/codespaces-blank $ 
```
## Code

```py
@pradeepgadde ➜ /workspaces/codespaces-blank $ cat k8s-diagram.py 
from diagrams import Diagram
from diagrams.k8s.clusterconfig import HPA
from diagrams.k8s.compute import Deployment, Pod, ReplicaSet
from diagrams.k8s.network import Ingress, Service

with Diagram("Sample K8s Diagram with Python Code", show=False):
    net = Ingress("pradeepgadde.com") >> Service("svc")
    net >> [Pod("pod1"),
            Pod("pod2"),
            Pod("pod3")] << ReplicaSet("rs") << Deployment("dp") << HPA("hpa")
@pradeepgadde ➜ /workspaces/codespaces-blank $ python k8s-diagram.py 
@pradeepgadde ➜ /workspaces/codespaces-blank $ 
```
## Diagram
Here is the diagram generated

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ ls
k8s-diagram.py  sample_k8s_diagram_with_python_code.png
@pradeepgadde ➜ /workspaces/codespaces-blank $ 
```

![sample_k8s_diagram_with_python_code]({{ site.url }}{{ site.baseurl }}/assets/images/sample_k8s_diagram_with_python_code.png)

Another example, this time from GCP

```py
from diagrams import Cluster, Diagram
from diagrams.gcp.analytics import BigQuery, Dataflow, PubSub
from diagrams.gcp.compute import AppEngine, Functions
from diagrams.gcp.database import BigTable
from diagrams.gcp.iot import IotCore
from diagrams.gcp.storage import GCS

with Diagram("Message Collecting", show=False):
    pubsub = PubSub("pubsub")

    with Cluster("Source of Data"):
        [IotCore("core1"),
         IotCore("core2"),
         IotCore("core3")] >> pubsub

    with Cluster("Targets"):
        with Cluster("Data Flow"):
            flow = Dataflow("data flow")

        with Cluster("Data Lake"):
            flow >> [BigQuery("bq"),
                     GCS("storage")]

        with Cluster("Event Driven"):
            with Cluster("Processing"):
                flow >> AppEngine("engine") >> BigTable("bigtable")

            with Cluster("Serverless"):
                flow >> Functions("func") >> AppEngine("appengine")

    pubsub >> flow



```

GCP Diagram

![message_collecting]({{ site.url }}{{ site.baseurl }}/assets/images/message_collecting.png)
