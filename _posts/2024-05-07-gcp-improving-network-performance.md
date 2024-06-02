---

layout: single
title:  Improving Network Performance"
categories: Cloud
tags: GCP
toc: true
toc_sticky: true
show_date: true
header:
  overlay_image: /assets/images/gcp-banner.png
  og_image: /assets/images/gcp-banner.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---

# Improving Network Performance

- How to test network connectivity and performance using open source tools
- How to inspect network traffic using open source tools
- How the size of your machine can affect the performance of your network

The goal of this lab is to show you relationships between the core size  and throughput, so the lab comes with 6 instances already built in. They were created when you started the lab.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-00-a84b20d45434.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances list
NAME: instance-1
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.6
EXTERNAL_IP: 34.148.172.136
STATUS: RUNNING

NAME: instance-2
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-2
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.5
EXTERNAL_IP: 35.196.192.171
STATUS: RUNNING

NAME: instance-3
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-4
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.4
EXTERNAL_IP: 35.237.47.162
STATUS: RUNNING

NAME: instance-4
ZONE: us-east1-d
MACHINE_TYPE: e2-highcpu-4
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.7
EXTERNAL_IP: 34.75.213.37
STATUS: RUNNING

NAME: instance-5
ZONE: us-east1-d
MACHINE_TYPE: e2-standard-4
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.2
EXTERNAL_IP: 34.139.183.38
STATUS: RUNNING

NAME: instance-6
ZONE: us-east1-d
MACHINE_TYPE: e2-micro
PREEMPTIBLE: 
INTERNAL_IP: 10.10.0.3
EXTERNAL_IP: 34.148.72.118
STATUS: RUNNING
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

### Connection test

Run a quick connection test to make sure things are working well.

1. **SSH** into `instance-1` by clicking the SSH button next to its name in the console.
2. In your new shell window ping another one of your instances and run the following command, replacing `<external ip address of instance-2>` with the `instance-2` external IP address:

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute ssh --zone "us-east1-d" "instance-1" --project "qwiklabs-gcp-00-a84b20d45434"
WARNING: The private SSH key file for gcloud does not exist.
WARNING: The public SSH key file for gcloud does not exist.
WARNING: You do not have an SSH key for gcloud.
WARNING: SSH keygen will be executed to generate a key.
This tool needs to create the directory [/home/student_02_5ab7393498fb/.ssh] before being able to generate SSH keys.

Do you want to continue (Y/n)?  y

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/student_02_5ab7393498fb/.ssh/google_compute_engine
Your public key has been saved in /home/student_02_5ab7393498fb/.ssh/google_compute_engine.pub
The key fingerprint is:
SHA256:C2mGBesgTnsWANOr03Q2QogXPUB5yWN79Pcsh08QWQA student_02_5ab7393498fb@cs-197877446053-default
The key's randomart image is:
+---[RSA 3072]----+
|==+*..  E..+.    |
|o.* Oo.   o      |
|.+.=.=..   .     |
|o.=o*o... o      |
| * *oo= S. =     |
|o +  o . .o =    |
| .      .  =     |
|            .    |
|                 |
+----[SHA256]-----+
Warning: Permanently added 'compute.5978547865854751453' (ED25519) to the list of known hosts.
Linux instance-1 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-5ab7393498fb'.
student-02-5ab7393498fb@instance-1:~$ ping -c 5 35.196.192.171
PING 35.196.192.171 (35.196.192.171) 56(84) bytes of data.
64 bytes from 35.196.192.171: icmp_seq=1 ttl=64 time=4.11 ms
64 bytes from 35.196.192.171: icmp_seq=2 ttl=64 time=0.547 ms
64 bytes from 35.196.192.171: icmp_seq=3 ttl=64 time=0.497 ms
64 bytes from 35.196.192.171: icmp_seq=4 ttl=64 time=0.546 ms
64 bytes from 35.196.192.171: icmp_seq=5 ttl=64 time=0.542 ms

--- 35.196.192.171 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4053ms
rtt min/avg/max/mdev = 0.497/1.249/4.114/1.432 ms
student-02-5ab7393498fb@instance-1:~$ 
```

Ping another. Replace `<external ip address of instance-3>` with the `instance-3` external IP address and `ping` it:

```sh
student-02-5ab7393498fb@instance-1:~$ ping -c 5 35.237.47.162
PING 35.237.47.162 (35.237.47.162) 56(84) bytes of data.
64 bytes from 35.237.47.162: icmp_seq=1 ttl=64 time=3.46 ms
64 bytes from 35.237.47.162: icmp_seq=2 ttl=64 time=0.592 ms
64 bytes from 35.237.47.162: icmp_seq=3 ttl=64 time=0.538 ms
64 bytes from 35.237.47.162: icmp_seq=4 ttl=64 time=0.536 ms
64 bytes from 35.237.47.162: icmp_seq=5 ttl=64 time=0.497 ms

--- 35.237.47.162 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4067ms
rtt min/avg/max/mdev = 0.497/1.124/3.461/1.168 ms
student-02-5ab7393498fb@instance-1:~$ 
```

Everything looks good, continue on!

### Review firewall rules

Firewall rules were also created for this lab.

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute firewall-rules list
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

NAME: iperftesting
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: udp:5001,tcp:5001
DENY: 
DISABLED: False

NAME: nw-testing-allow-http
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:80
DENY: 
DISABLED: False

NAME: nw-testing-allow-https
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:443
DENY: 
DISABLED: False

NAME: nw-testing-allow-icmp
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp
DENY: 
DISABLED: False

NAME: nw-testing-allow-internal
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:0-65535,udp:0-65535
DENY: 
DISABLED: False

NAME: nw-testing-allow-ssh
NETWORK: nw-testing
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:22
DENY: 
DISABLED: False

To show all fields of the firewall, please show in JSON format: --format=json
To show all fields in table format, please see the examples in --help.

student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

The firewall rule **iperftesting** uses following configuration:

| **Field**            | **Value**                    | **Comments**                                                 |
| -------------------- | ---------------------------- | ------------------------------------------------------------ |
| Name                 | iperftesting                 | New rule name                                                |
| Targets              | All instances in the network |                                                              |
| Source IP ranges     | 0.0.0.0/0                    | We will open the firewall for any IP address from the Internet. |
| Protocols and ports  | tcp:5001; udp:5001           |                                                              |
| Direction of traffic | Ingress                      |                                                              |
| Action on match      | Allow                        |                                                              |

Now you're ready to start using the lab.

## Use case 1: Networking and Compute Engine core count

In this first scenario you'll see how the size of the machines being used affects throughput that you can measure.

*Dobermanifesto* is a video microblogging network exclusively  for pets. Animal based videos can be uploaded, worldwide, and sent  anywhere to be viewed & experienced.

While transferring data to/from their Compute Engine backends, their observed bandwidth was not as high as they were hoping for.

## Reproducing behavior

To try and reproduce this behavior, two instances in the same zone were created, and `iperf` was run between them 100x.

This performance is even worse! Obviously something was wrong with  the test. We need more information and a deeper set of reproduction  steps from the company.

Now you will set up the scenario.

### Dobermanifesto's environment

1. Return to the VM instances list in the Compute Engine console.
2. SSH into `instance-1` (1vCPU 3.75GB) and run this command, setting up an `iperf` "receiver":
3. Then SSH into `instance-2` (1vCPU 3.75GB) and generate `iperf` traffic pointing at `instance-1`:

```sh
student-02-5ab7393498fb@instance-1:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.6 port 5001 connected with 35.196.192.171 port 45062
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0068 sec  4.66 GBytes  4.00 Gbits/sec
^Cstudent-02-5ab7393498fb@instance-1:~$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute ssh --zone "us-east1-d" "instance-2" --project "qwiklabs-gcp-00-a84b20d45434"
Warning: Permanently added 'compute.1578266610252198621' (ED25519) to the list of known hosts.
Linux instance-2 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-5ab7393498fb'.
student-02-5ab7393498fb@instance-2:~$ iperf -c 34.148.172.136
------------------------------------------------------------
Client connecting to 34.148.172.136, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.5 port 45062 connected with 34.148.172.136 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0024 sec  4.66 GBytes  4.00 Gbits/sec
student-02-5ab7393498fb@instance-2:~$ 
```

### Test environment

1. Go back to the Compute Engine console and open another SSH window into `instance-6` (1vCPU e2-micro .6GB).
2. Run the following command, setting it up as a "receiver":
3. In the `instance-2` SSH window, test the connection to `instance-6`:

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute ssh --zone "us-east1-d" "instance-6" --project "qwiklabs-gcp-00-a84b20d45434"
Warning: Permanently added 'compute.8152207892119892701' (ED25519) to the list of known hosts.
Linux instance-6 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-5ab7393498fb'.
student-02-5ab7393498fb@instance-6:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.3 port 5001 connected with 10.10.0.5 port 54278
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0056 sec  4.66 GBytes  4.00 Gbits/sec
^Cstudent-02-5ab7393498fb@instance-6:~$ 
```

```sh
student-02-5ab7393498fb@instance-2:~$ iperf -c 10.10.0.3
------------------------------------------------------------
Client connecting to 10.10.0.3, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.5 port 54278 connected with 10.10.0.3 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0005 sec  4.66 GBytes  4.01 Gbits/sec
student-02-5ab7393498fb@instance-2:~$
```

What happened? Bandwidth appears to have gone up. It may have in your case as well. It may have been less.

In the next section you will see how bandwidth is constrained by  total core count, and that with core counts in this small range (core  count of 1), the bandwidth will never exceed 2 Gbits/sec or so. As a  result, the network speed is slow and bandwidth constrained, and similar to what *Dobermanifesto* was experiencing. When you test with 4-cpu machines in a minute, the results will be greater.

### The number of cores correlates to Gb/s

Why weren't the results much different? The [documentation for Compute Engine](https://cloud.google.com/compute/docs/networks-and-firewalls#egress_throughput_caps) states:

*Outbound or egress traffic from a virtual machine is subject to  maximum network egress throughput caps. These caps are dependent on the  number of vCPUs that a virtual machine instance has. Each core is  subject to a 2 Gbits/second (Gbps) cap for peak performance. Each  additional core increases the network cap, up to a theoretical maximum  of 16 Gbps for each virtual machine*

This means that the more virtual CPUs in your network, the more networking throughput you will get.

In order to figure out what this looks like in practice, different  core size groups were set up in the same zone, and iperf was run between them 1000 times.

![The bar chart: RCP Throughput by instance type, times 1000, displaying the difference between average and maximum.](https://cdn.qwiklabs.com/M8BRX1KjRhymUkIecOjMyxRidcAUdVhzj9DUyKfaE1w%3D) 

As the core count goes up, so does the avg and max throughput. Even with this simple test, you can see that hard 16Gbps limit on the higher  machines.

If you run iperf with multiple threads (~8 or so) you can exceed 10Gbps, and get up to about 16Gbps using a e2-standard-16 or larger. We don't  have that size of a machine in this lab environment, but let's do a test with multiple threads next. 

There are also [higher-cost VMs in the N2, N2D, C2, or C2D series](https://cloud.google.com/compute/docs/networking/configure-vm-with-high-bandwidth-configuration) that allow you to select a high bandwidth Tier 1 config and reach 50-100 Gbps.

## How to improve results

The network that *Dobermanifesto* has uses 1vCPU machines. Increasing the size of the core will probably help *Dobermanifesto* achieve better results. Time to test this theory.

1. SSH into `instance-3` (4vCPU 15GB memory) and run this command:

```sh
student-02-5ab7393498fb@instance-3:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 42018
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0028 sec  9.21 GBytes  7.91 Gbits/sec

```

SSH into `instance-4` (4vCPU 15GB memory):

```sh
student-02-5ab7393498fb@instance-4:~$ iperf -c 10.10.0.4
------------------------------------------------------------
Client connecting to 10.10.0.4, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.7 port 42018 connected with 10.10.0.4 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0002 sec  9.21 GBytes  7.91 Gbits/sec
student-02-5ab7393498fb@instance-4:~$ 
```

Now try it again with 4 threads:

```sh
student-02-5ab7393498fb@instance-4:~$ iperf -c 10.10.0.4 -P4
[  6] local 10.10.0.7 port 46384 connected with 10.10.0.4 port 5001
------------------------------------------------------------
Client connecting to 10.10.0.4, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.7 port 46374 connected with 10.10.0.4 port 5001
[  5] local 10.10.0.7 port 46376 connected with 10.10.0.4 port 5001
[  3] local 10.10.0.7 port 46372 connected with 10.10.0.4 port 5001
[ ID] Interval       Transfer     Bandwidth
[  6] 0.0000-10.0043 sec  2.30 GBytes  1.97 Gbits/sec
[  5] 0.0000-10.0018 sec  2.44 GBytes  2.10 Gbits/sec
[  4] 0.0000-10.0034 sec  2.07 GBytes  1.78 Gbits/sec
[  3] 0.0000-10.0009 sec  2.41 GBytes  2.07 Gbits/sec
[SUM] 0.0000-10.0009 sec  9.22 GBytes  7.92 Gbits/sec
[ CT] final connect times (min/avg/max/stdev) = 0.803/1.152/1.330/0.733 ms (tot/err) = 4/0
student-02-5ab7393498fb@instance-4:~$ 
```

And with 8 threads:

```sh
student-02-5ab7393498fb@instance-4:~$ iperf -c 10.10.0.4 -P8
[  7] local 10.10.0.7 port 50848 connected with 10.10.0.4 port 5001
[  6] local 10.10.0.7 port 50834 connected with 10.10.0.4 port 5001
[  9] local 10.10.0.7 port 50862 connected with 10.10.0.4 port 5001
[  4] local 10.10.0.7 port 50846 connected with 10.10.0.4 port 5001
------------------------------------------------------------
Client connecting to 10.10.0.4, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  5] local 10.10.0.7 port 50830 connected with 10.10.0.4 port 5001
[  3] local 10.10.0.7 port 50824 connected with 10.10.0.4 port 5001
[  8] local 10.10.0.7 port 50872 connected with 10.10.0.4 port 5001
[ 10] local 10.10.0.7 port 50880 connected with 10.10.0.4 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0055 sec  1.20 GBytes  1.03 Gbits/sec
[  7] 0.0000-10.0018 sec   632 MBytes   530 Mbits/sec
[  4] 0.0000-10.0042 sec  2.36 GBytes  2.03 Gbits/sec
[  5] 0.0000-10.0056 sec   456 MBytes   383 Mbits/sec
[ 10] 0.0000-10.0063 sec  2.32 GBytes  1.99 Gbits/sec
[  6] 0.0000-10.0059 sec   595 MBytes   499 Mbits/sec
[  8] 0.0000-10.0047 sec  1.10 GBytes   945 Mbits/sec
[  9] 0.0000-10.0043 sec   601 MBytes   504 Mbits/sec
[SUM] 0.0000-10.0051 sec  9.21 GBytes  7.91 Gbits/sec
[ CT] final connect times (min/avg/max/stdev) = 0.109/0.514/1.154/0.393 ms (tot/err) = 8/0
student-02-5ab7393498fb@instance-4:~$ 
```

1. Return to `instance-3` and enter **CTRL** + **C** to end the receiver.

In these experiments, both the server and client were 4vCPUs, and the speed was greatly increased. The transfer rate was increased by 6.64  GBytes, and the Bandwidth by 5.71 Gbits/sec. With multiple threads, the  performance was able to reach the cap for that core count.

```sh
student-02-5ab7393498fb@instance-3:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 42018
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0028 sec  9.21 GBytes  7.91 Gbits/sec
[  5] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 46372
[  4] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 46384
[  6] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 46374
[  7] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 46376
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0119 sec  2.30 GBytes  1.97 Gbits/sec
[  6] 0.0000-10.0088 sec  2.07 GBytes  1.78 Gbits/sec
[  7] 0.0000-10.0075 sec  2.44 GBytes  2.10 Gbits/sec
[  5] 0.0000-10.0106 sec  2.41 GBytes  2.07 Gbits/sec
[SUM] 0.0000-10.0106 sec  9.22 GBytes  7.91 Gbits/sec
[  8] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50862
[  5] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50824
[  4] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50846
[  6] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50880
[  7] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50848
[  9] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50872
[ 10] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50830
[ 11] local 10.10.0.4 port 5001 connected with 10.10.0.7 port 50834
[ ID] Interval       Transfer     Bandwidth
[  9] 0.0000-10.0175 sec  1.10 GBytes   943 Mbits/sec
[ 11] 0.0000-10.0153 sec   595 MBytes   499 Mbits/sec
[  8] 0.0000-10.0177 sec   601 MBytes   504 Mbits/sec
[  5] 0.0000-10.0153 sec  1.20 GBytes  1.03 Gbits/sec
[  7] 0.0000-10.0140 sec   632 MBytes   529 Mbits/sec
[  4] 0.0000-10.0155 sec  2.36 GBytes  2.02 Gbits/sec
[  6] 0.0000-10.0185 sec  2.32 GBytes  1.99 Gbits/sec
[ 10] 0.0000-10.0155 sec   456 MBytes   382 Mbits/sec
[SUM] 0.0000-10.0169 sec  9.21 GBytes  7.90 Gbits/sec
^Cstudent-02-5ab7393498fb@instance-3:~$ 
```

Continue testing with `instance-5`, which is a higher performance 4vCPU machine, instance type "highcpu-4".

This instance type has faster CPUs, but less memory. What differences do you see, if any? With multiple threads?

Now the *Dobermanifesto* team needs to decide what route to take. After profiling their CPU usage and taking a look at the [pricing info](https://cloud.google.com/compute/pricing), they decided to go with a e2-standard-4 machine, which gave them almost 4x the increase in average throughput, but cheaper than the  e2-standard-8 machines.

One of the nice things about moving to the larger machine is that it  actually runs less frequently. It turns out that their machines were  spending a lot of time staying awake, just to transfer data. With the  new machine size, their instances had more downtime, allowing the load  balancer to reduce the total number of instances on a daily basis. So on one hand, they ended up paying for a higher-grade machine, but on the  other hand, they will use less core-hours on a monthly basis.

Once your performance directly impacts the bottom line, there's a lot of nuanced tradeoffs to consider.

## Use case 2: Google Cloud networking with internal IPs

In this next example, you'll use `iperf` to test throughput speed. You'll set up one machine as the server, and then point other machines to it and compare the results.

*Gecko Protocol*, a B2B company offering a custom,  light-weight networking protocol built for gaming and other real-time  graphics systems, were seeing lower-than-expected throughput for their  backend machines which were responsible for transferring and transcoding large video & graphics files. When duplicating the test, the network setup was identical, but the test results were quite different:

.95GB / sec was much higher than what *Gecko Protocol* was seeing in their graphs. So what's going on?

Now recreate this scenario.

1. In the Console, SSH into `instance-1` and set up the iperf receiver:

SSH into `instance-2` and check the connection of the external IP address:

```sh
student-02-5ab7393498fb@instance-2:~$ iperf -c 34.148.172.136
------------------------------------------------------------
Client connecting to 34.148.172.136, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.5 port 37066 connected with 34.148.172.136 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0019 sec  4.66 GBytes  4.01 Gbits/sec
student-02-5ab7393498fb@instance-2:~$ 
```

```sh
student-02-5ab7393498fb@instance-1:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.6 port 5001 connected with 35.196.192.171 port 37066
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec

```

After further discussion with *Gecko Protocol*, it was learned that they were using *external* IPs to connect their machines, and the test used *internal* IPs. When the machines are in a network, connecting them with internal IPs will result in faster throughput.

1. Now check the connection with the internal address:

```sh
student-02-5ab7393498fb@instance-2:~$ iperf -c 10.10.0.6
------------------------------------------------------------
Client connecting to 10.10.0.6, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.5 port 34486 connected with 10.10.0.6 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0006 sec  4.66 GBytes  4.00 Gbits/sec
student-02-5ab7393498fb@instance-2:~$ 
```

```sh
student-02-5ab7393498fb@instance-1:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.6 port 5001 connected with 35.196.192.171 port 37066
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  5] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 42012
[ ID] Interval       Transfer     Bandwidth
[  5] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39812
[  5] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39836 (full-duplex)
[  6] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39824 (isoch)
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-0.1023 sec  82.0 Bytes  6.41 Kbits/sec
recv failed: Resource temporarily unavailable
[  6] 0.0000-2.0084 sec  0.000 Bytes  0.000 bits/sec
write failed: Broken pipe
shutdown failed: Transport endpoint is not connected
[  5] 0.0000-3.4879 sec  68.8 KBytes   161 Kbits/sec
[SUM] 0.0000-3.4924 sec  68.8 KBytes   161 Kbits/sec

[  7] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 58136
[ ID] Interval       Transfer     Bandwidth
[  7] 0.0000-10.0061 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 34486
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0060 sec  4.66 GBytes  4.00 Gbits/sec

```

Look at the two different Transfer and Bandwidth rates. In this  example, changing to the internal IP address resulted in a .9 GBytes  improvement in transfer rate, and .78 Gbits/sec improvement in  bandwidth. You just proved that the internal connection is faster.

Building on what you learned from solving the *Dobermanifesto* problem, can the network speed be improved even more by using a larger machine? `instance-2` is only 1vCPU. How fast will the connection be if the machine is a  little larger? Or a lot larger? Continue to test using the internal IP  address (but feel free to test the external one, too, if you have time).

### 4 x vCPU machine

- SSH into `instance-3` and test the connection with the internal IP address:

```sh
student-02-5ab7393498fb@instance-3:~$ iperf -c 10.10.0.6
------------------------------------------------------------
Client connecting to 10.10.0.6, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.4 port 50614 connected with 10.10.0.6 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0015 sec  9.22 GBytes  7.92 Gbits/sec
student-02-5ab7393498fb@instance-3:~$ 
```

```sh
student-02-5ab7393498fb@instance-1:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.6 port 5001 connected with 35.196.192.171 port 37066
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  5] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 42012
[ ID] Interval       Transfer     Bandwidth
[  5] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39812
[  5] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39836 (full-duplex)
[  6] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39824 (isoch)
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-0.1023 sec  82.0 Bytes  6.41 Kbits/sec
recv failed: Resource temporarily unavailable
[  6] 0.0000-2.0084 sec  0.000 Bytes  0.000 bits/sec
write failed: Broken pipe
shutdown failed: Transport endpoint is not connected
[  5] 0.0000-3.4879 sec  68.8 KBytes   161 Kbits/sec
[SUM] 0.0000-3.4924 sec  68.8 KBytes   161 Kbits/sec

[  7] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 58136
[ ID] Interval       Transfer     Bandwidth
[  7] 0.0000-10.0061 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 34486
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0060 sec  4.66 GBytes  4.00 Gbits/sec
[  6] local 10.10.0.6 port 5001 connected with 10.10.0.4 port 50614
[ ID] Interval       Transfer     Bandwidth
[  6] 0.0000-10.0041 sec  9.22 GBytes  7.91 Gbits/sec

```

### 4 highCPU machine

- SSH into `instance-5` and test the connection with the internal IP address:

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute ssh --zone "us-east1-d" "instance-5" --project "qwiklabs-gcp-00-a84b20d45434"
Warning: Permanently added 'compute.443220592027579101' (ED25519) to the list of known hosts.
Linux instance-5 5.10.0-29-cloud-amd64 #1 SMP Debian 5.10.216-1 (2024-05-03) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Creating directory '/home/student-02-5ab7393498fb'.
student-02-5ab7393498fb@instance-5:~$ iperf -c 10.10.0.6
------------------------------------------------------------
Client connecting to 10.10.0.6, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.0.2 port 54318 connected with 10.10.0.6 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3] 0.0000-10.0005 sec  9.21 GBytes  7.91 Gbits/sec
student-02-5ab7393498fb@instance-5:~$ 
```

```sh
student-02-5ab7393498fb@instance-1:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size:  128 KByte (default)
------------------------------------------------------------
[  4] local 10.10.0.6 port 5001 connected with 35.196.192.171 port 37066
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  5] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 42012
[ ID] Interval       Transfer     Bandwidth
[  5] 0.0000-10.0071 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39812
[  5] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39836 (full-duplex)
[  6] local 10.10.0.6 port 5001 connected with 185.224.128.17 port 39824 (isoch)
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-0.1023 sec  82.0 Bytes  6.41 Kbits/sec
recv failed: Resource temporarily unavailable
[  6] 0.0000-2.0084 sec  0.000 Bytes  0.000 bits/sec
write failed: Broken pipe
shutdown failed: Transport endpoint is not connected
[  5] 0.0000-3.4879 sec  68.8 KBytes   161 Kbits/sec
[SUM] 0.0000-3.4924 sec  68.8 KBytes   161 Kbits/sec

[  7] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 58136
[ ID] Interval       Transfer     Bandwidth
[  7] 0.0000-10.0061 sec  4.66 GBytes  4.00 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 10.10.0.5 port 34486
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0060 sec  4.66 GBytes  4.00 Gbits/sec
[  6] local 10.10.0.6 port 5001 connected with 10.10.0.4 port 50614
[ ID] Interval       Transfer     Bandwidth
[  6] 0.0000-10.0041 sec  9.22 GBytes  7.91 Gbits/sec
[  4] local 10.10.0.6 port 5001 connected with 10.10.0.2 port 54318
[ ID] Interval       Transfer     Bandwidth
[  4] 0.0000-10.0029 sec  9.21 GBytes  7.91 Gbits/sec

```

This really improves the throughput rate.

Looks like *Gecko Protocol* will also need to think about what size core will be best. This small debugging session resulted in their  video and graphics data transfer improving by ~14x. Which is immense,  considering their offering is built on performance backend services for  high-performance compute scenarios.

The tendency is to get the workload as close to 100% as possible, which leaves little space for the disk to defrag, etc.

90-93% if usage is healthy, but 98% usage will see performance go down since you'll end up with lots of contention.

When you're testing, if the IO performance decays, look at throttle  counters. If they're not being throttled, look at cpu usage. If it's  high, there's the problem.

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-1
Did you mean zone [asia-southeast1-a] for instance: [instance-1] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-1].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:56.340-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-1
  type: PERSISTENT
fingerprint: UGrNiBRHYX0=
id: '5978547865854751453'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:01.827-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-standard-2
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-1
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 34.148.172.136
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: D2LUr8Z7Nyc=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.6
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-1
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-1
Did you mean zone [asia-southeast1-a] for instance: [instance-1] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-1].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:56.340-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-1
  type: PERSISTENT
fingerprint: UGrNiBRHYX0=
id: '5978547865854751453'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:01.827-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-standard-2
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-1
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 34.148.172.136
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: D2LUr8Z7Nyc=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.6
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-1
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-3
Did you mean zone [asia-southeast1-a] for instance: [instance-3] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-3].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:56.109-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-3
  type: PERSISTENT
fingerprint: zmDPJWZ8nhQ=
id: '3564523181734703837'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:01.616-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-standard-4
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-3
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 35.237.47.162
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: 7F1P9n1MA9k=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.4
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-3
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-4
Did you mean zone [asia-southeast1-a] for instance: [instance-4] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-4].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:57.457-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-4
  type: PERSISTENT
fingerprint: mpEtBwlLvPs=
id: '1891380278865244893'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:03.292-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-highcpu-4
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-4
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 34.75.213.37
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: MNC5yBO1gPI=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.7
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-4
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-5
Did you mean zone [asia-southeast1-a] for instance: [instance-5] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-5].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:54.596-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-5
  type: PERSISTENT
fingerprint: EW5h4Vb4-EI=
id: '443220592027579101'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:00.232-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-standard-4
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-5
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 34.139.183.38
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: HqhhLIId9Ng=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.2
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-5
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

```sh
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ gcloud compute instances describe instance-6
Did you mean zone [asia-southeast1-a] for instance: [instance-6] (Y/n)?  n

No zone specified. Using zone [us-east1-d] for instance: [instance-6].
canIpForward: false
cpuPlatform: Intel Broadwell
creationTimestamp: '2024-06-02T08:13:55.608-07:00'
deletionProtection: false
disks:
- architecture: X86_64
  autoDelete: true
  boot: true
  deviceName: persistent-disk-0
  diskSizeGb: '10'
  guestOsFeatures:
  - type: UEFI_COMPATIBLE
  - type: VIRTIO_SCSI_MULTIQUEUE
  - type: GVNIC
  - type: SEV_CAPABLE
  index: 0
  interface: SCSI
  kind: compute#attachedDisk
  licenses:
  - https://www.googleapis.com/compute/v1/projects/debian-cloud/global/licenses/debian-11-bullseye
  mode: READ_WRITE
  source: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/disks/instance-6
  type: PERSISTENT
fingerprint: pqkYFZN26-w=
id: '8152207892119892701'
kind: compute#instance
labelFingerprint: 42WmSpB8rSM=
lastStartTimestamp: '2024-06-02T08:14:01.954-07:00'
machineType: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/machineTypes/e2-micro
metadata:
  fingerprint: Rda25r7P-FQ=
  items:
  - key: startup-script
    value: apt-get update && apt-get install iperf
  kind: compute#metadata
name: instance-6
networkInterfaces:
- accessConfigs:
  - kind: compute#accessConfig
    name: external-nat
    natIP: 34.148.72.118
    networkTier: PREMIUM
    type: ONE_TO_ONE_NAT
  fingerprint: KsyMZQTF48Y=
  kind: compute#networkInterface
  name: nic0
  network: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/global/networks/nw-testing
  networkIP: 10.10.0.3
  stackType: IPV4_ONLY
  subnetwork: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/regions/us-east1/subnetworks/us-east1
satisfiesPzi: false
scheduling:
  automaticRestart: true
  onHostMaintenance: MIGRATE
  preemptible: false
  provisioningModel: STANDARD
selfLink: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d/instances/instance-6
shieldedInstanceConfig:
  enableIntegrityMonitoring: true
  enableSecureBoot: false
  enableVtpm: true
shieldedInstanceIntegrityPolicy:
  updateAutoLearnPolicy: true
startRestricted: false
status: RUNNING
tags:
  fingerprint: 42WmSpB8rSM=
zone: https://www.googleapis.com/compute/v1/projects/qwiklabs-gcp-00-a84b20d45434/zones/us-east1-d
student_02_5ab7393498fb@cloudshell:~ (qwiklabs-gcp-00-a84b20d45434)$ 
```

In this lab, you learned how to test network connectivity and  performance using open source tools, and how the size of your machine  can affect the performance of your network.
