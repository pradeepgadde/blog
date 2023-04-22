---
layout: single
title:  "Creating Virtual Machines"
date:   2023-04-21 13:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-1.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# GCP Creating Virtual Machines

- Create several standard VMs
- Create advanced VMs



```sh
gcloud compute instances create pradeep-1 --project=qwiklabs-gcp-04-826073374c5a --zone=us-central1-c --machine-type=n1-standard-1 --network-interface=subnet=default,no-address --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=361019318543-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=pradeep-1,image=projects/debian-cloud/global/images/debian-10-buster-v20230411,mode=rw,size=10,type=projects/qwiklabs-gcp-04-826073374c5a/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```





```sh
gcloud compute instances create pradeep-3 --project=qwiklabs-gcp-04-826073374c5a --zone=us-central1-a --machine-type=e2-custom-2-4096 --network-interface=network-tier=PREMIUM,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=361019318543-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=pradeep-3,image=projects/debian-cloud/global/images/debian-10-buster-v20230411,mode=rw,size=10,type=projects/qwiklabs-gcp-04-826073374c5a/zones/us-central1-a/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=ec-src=vm_add-gcloud --reservation-affinity=any
```



```sh
student-01-b9934b2a0bae@pradeep-3:~$ free
              total        used        free      shared  buff/cache   available
Mem:        4040776      131052     3772504        5360      137220     3717864
Swap:             0           0           0
student-01-b9934b2a0bae@pradeep-3:~$ sudo dmidecode -t 17
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.4 present.

Handle 0x7000, DMI type 17, 21 bytes
Memory Device
        Array Handle: 0x0200
        Error Information Handle: Not Provided
        Total Width: 64 bits
        Data Width: 64 bits
        Size: 4096 MB
        Form Factor: DIMM
        Set: None
        Locator: DIMM 0
        Bank Locator: Not Specified
        Type: RAM
        Type Detail: Synchronous

student-01-b9934b2a0bae@pradeep-3:~$ nproc
2
student-01-b9934b2a0bae@pradeep-3:~$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
Address sizes:       46 bits physical, 48 bits virtual
CPU(s):              2
On-line CPU(s) list: 0,1
Thread(s) per core:  2
Core(s) per socket:  1
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               79
Model name:          Intel(R) Xeon(R) CPU @ 2.20GHz
Stepping:            0
CPU MHz:             2199.998
BogoMIPS:            4399.99
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            56320K
NUMA node0 CPU(s):   0,1
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single pti ssbd ibrs ibpb stibp fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm rdseed adx smap xsaveopt arat md_clear arch_capabilities
student-01-b9934b2a0bae@pradeep-3:~$ 
student-01-b9934b2a0bae@pradeep-3:~$ 
```



![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-1.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-2.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-3.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/gcp-vm-8.png)

In this lab, we created several virtual machine instances of different  types with different characteristics. One was a small utility VM for  administration purposes. We also created a standard VM and a custom VM. We launched both Windows and Linux VMs and deleted VMs.
