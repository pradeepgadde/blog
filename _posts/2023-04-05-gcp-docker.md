---


layout: single
title:  "Introduction to Docker on GCP"
date:   2023-04-05 08:59:04 +0530
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/gcp-banner-2.png
  og_image: /assets/images/gcp-banner-2.png
  teaser: /assets/images/gpcne.png
author:
  name     : "Cloud Network Engineer"
  avatar   : "/assets/images/gpcne.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Introduction to Docker on GCP


Docker is an open platform for developing, shipping, and running applications. With Docker, you can separate your applications from your infrastructure and treat your infrastructure like a managed application. Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code.

Docker does this by combining kernel containerization features with workflows and tooling that helps you manage and deploy your applications.

Docker containers can be directly used in Kubernetes, which allows them to be run in the Kubernetes Engine with ease. After learning the essentials of Docker, you will have the skillset to start developing Kubernetes and containerized applications.

```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-542391075613.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-03-549093f60571@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ gcloud config list project
[core]
project = qwiklabs-gcp-04-542391075613

Your active configuration is: [cloudshell-4812]
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:ffb13da98453e0f04d33a6eee5bb8e46ee50d08ebe17735fc0779d0349e889e9
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   18 months ago   13.3kB
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$

student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED              STATUS                          PORTS     NAMES
b69394853641   hello-world   "/hello"   40 seconds ago       Exited (0) 39 seconds ago                 suspicious_bose
2a412eea2e00   hello-world   "/hello"   About a minute ago   Exited (0) About a minute ago             optimistic_thompson
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$
```



```sh

student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ mkdir test && cd test
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:lts
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ cat > app.js <<EOF
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
EOF process.exit();
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ ls
app.js  Dockerfile
```
```js
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ cat app.js
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
```
```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ cat Dockerfile
# Use an official Node runtime as the parent image
FROM node:lts
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker build -t node-app:0.1 .
[+] Building 21.4s (8/8) FINISHED
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 393B                                                                                                                                           0.0s
 => [internal] load metadata for docker.io/library/node:lts                                                                                                                    1.5s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 889B                                                                                                                                              0.0s
 => [1/3] FROM docker.io/library/node:lts@sha256:c21209748c829660e0b49cbd14d2f9d81ea82ffb02a8a7932ebacf70d01573a3                                                             18.4s
 => => resolve docker.io/library/node:lts@sha256:c21209748c829660e0b49cbd14d2f9d81ea82ffb02a8a7932ebacf70d01573a3                                                              0.0s
 => => sha256:68a71c865a2c34678c6dea55e4b0928f751ee3c0ca91cace6e4e0578c534d6cf 5.17MB / 5.17MB                                                                                 0.2s
 => => sha256:c21209748c829660e0b49cbd14d2f9d81ea82ffb02a8a7932ebacf70d01573a3 1.21kB / 1.21kB                                                                                 0.0s
 => => sha256:1d2e96403c7a507d3c7acddc70b5d58dec3e19fbd161be44ad545e9b9bea1f87 7.56kB / 7.56kB                                                                                 0.0s
 => => sha256:3e440a7045683e27f8e2fa04000e0e078d8dfac0c971358ae0f8c65c13321c8e 55.05MB / 55.05MB                                                                               0.7s
 => => sha256:33f306d574d22a441f6473d09c851763ff0d44459af682a2ff23b6ec8a06b03e 2.21kB / 2.21kB                                                                                 0.0s
 => => sha256:670730c27c2eacf07897a6e94fe55423ea50b884d9c28161a283bbbf064d1124 10.88MB / 10.88MB                                                                               0.3s
 => => sha256:5a7a2c95f0f8b221d776ccf35911b68eec2cf9414a44d216205a6f03e381d3d7 54.58MB / 54.58MB                                                                               1.0s
 => => sha256:6d627e120214bb28a729d4b54a0ecba4c4aeaf0295ca2d1f129480145fad2af6 196.81MB / 196.81MB                                                                             3.1s
 => => sha256:1dc4abe482bf348bfe83044caa1621e01a8cb9c0a2d89a77e77bad335fcccaf4 4.20kB / 4.20kB                                                                                 1.0s
 => => extracting sha256:3e440a7045683e27f8e2fa04000e0e078d8dfac0c971358ae0f8c65c13321c8e                                                                                      3.0s
 => => sha256:e4b205f7d691d38d3b25b3972658c116f259e3ba78d854a461bb7528cc2f6061 45.58MB / 45.58MB                                                                               1.7s
 => => sha256:47ae708a5b38406618b4d3d086b51c4ce4b1171064d5d25823b45528c605b411 2.28MB / 2.28MB                                                                                 1.2s
 => => sha256:c37e138d8eaccd769735e63f882c5e4bb2efbccec267c5216bc6001c9d1d3e91 449B / 449B                                                                                     1.4s
 => => extracting sha256:68a71c865a2c34678c6dea55e4b0928f751ee3c0ca91cace6e4e0578c534d6cf                                                                                      0.2s
 => => extracting sha256:670730c27c2eacf07897a6e94fe55423ea50b884d9c28161a283bbbf064d1124                                                                                      0.3s
 => => extracting sha256:5a7a2c95f0f8b221d776ccf35911b68eec2cf9414a44d216205a6f03e381d3d7                                                                                      2.5s
 => => extracting sha256:6d627e120214bb28a729d4b54a0ecba4c4aeaf0295ca2d1f129480145fad2af6                                                                                      7.0s
 => => extracting sha256:1dc4abe482bf348bfe83044caa1621e01a8cb9c0a2d89a77e77bad335fcccaf4                                                                                      0.0s
 => => extracting sha256:e4b205f7d691d38d3b25b3972658c116f259e3ba78d854a461bb7528cc2f6061                                                                                      2.5s
 => => extracting sha256:47ae708a5b38406618b4d3d086b51c4ce4b1171064d5d25823b45528c605b411                                                                                      0.1s
 => => extracting sha256:c37e138d8eaccd769735e63f882c5e4bb2efbccec267c5216bc6001c9d1d3e91                                                                                      0.0s
 => [2/3] WORKDIR /app                                                                                                                                                         1.4s
 => [3/3] ADD . /app                                                                                                                                                           0.0s
 => exporting to image                                                                                                                                                         0.1s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:c9a9cefdba4fd7a3012df5c49ac47c373f7741f5a9afefca29bab3f77b40d144                                                                                   0.0s
 => => naming to docker.io/library/node-app:0.1                                                                                                                                0.0s
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
node-app      0.1       c9a9cefdba4f   57 seconds ago   997MB
hello-world   latest    feb5d9fea6a5   18 months ago    13.3kB
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
node-app      0.1       c9a9cefdba4f   57 seconds ago   997MB
hello-world   latest    feb5d9fea6a5   18 months ago    13.3kB
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker run -p 4000:80 --name my-app node-app:0.1
Server running at http://0.0.0.0:80/

```



```sh
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to qwiklabs-gcp-04-542391075613.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ curl http://localhost:4000
Hello World
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ ^C
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker stop my-app && docker rm my-app
my-app
my-app
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker run -p 4000:80 --name my-app -d node-app:0.1
docker ps
6b0c27f8ae2b6d5f0afe8173e31396167856a1db83b6a1be239996140eb38669
CONTAINER ID   IMAGE          COMMAND                  CREATED                  STATUS                  PORTS                  NAMES
6b0c27f8ae2b   node-app:0.1   "docker-entrypoint.s…"   Less than a second ago   Up Less than a second   0.0.0.0:4000->80/tcp   my-app
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker logs 6b0c27f8ae2b
Server running at http://0.0.0.0:80/
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```



```js
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ cat app.js
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Welcome to Cloud\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker build -t node-app:0.2 .
[+] Building 0.5s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 393B                                                                                                                                           0.0s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/node:lts                                                                                                                    0.4s
 => [1/3] FROM docker.io/library/node:lts@sha256:c21209748c829660e0b49cbd14d2f9d81ea82ffb02a8a7932ebacf70d01573a3                                                              0.0s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 533B                                                                                                                                              0.0s
 => CACHED [2/3] WORKDIR /app                                                                                                                                                  0.0s
 => [3/3] ADD . /app                                                                                                                                                           0.0s
 => exporting to image                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:55b9141f933f08c107e20af6cd2593b3718be2bc8983e1ed4f5c2fe8a3ee71ad                                                                                   0.0s
 => => naming to docker.io/library/node-app:0.2                                                                                                                                0.0s
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps
83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76
CONTAINER ID   IMAGE          COMMAND                  CREATED                  STATUS                  PORTS                  NAMES
83ed57056c49   node-app:0.2   "docker-entrypoint.s…"   Less than a second ago   Up Less than a second   0.0.0.0:8080->80/tcp   my-app-2
6b0c27f8ae2b   node-app:0.1   "docker-entrypoint.s…"   2 minutes ago            Up 2 minutes            0.0.0.0:4000->80/tcp   my-app
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ curl http://localhost:8080
Welcome to Cloud
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$
```



```sh
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$ curl http://localhost:4000
Hello World
student_03_549093f60571@cloudshell:~ (qwiklabs-gcp-04-542391075613)$
```



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker logs -f 83ed57056c49
Server running at http://0.0.0.0:80/

^C

student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker exec -it 83ed57056c49 bash
root@83ed57056c49:/app# ls
Dockerfile  app.js
root@83ed57056c49:/app# exit
exit
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker inspect 83ed57056c49
[
    {
        "Id": "83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76",
        "Created": "2023-04-06T04:21:53.334779315Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "node",
            "app.js"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1746,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2023-04-06T04:21:53.702553737Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:55b9141f933f08c107e20af6cd2593b3718be2bc8983e1ed4f5c2fe8a3ee71ad",
        "ResolvConfPath": "/var/lib/docker/containers/83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76/hostname",
        "HostsPath": "/var/lib/docker/containers/83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76/hosts",
        "LogPath": "/var/lib/docker/containers/83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76/83ed57056c4921472e96c73630c44f20f2f5624f057a2d8eb13cccba9a8c1f76-json.log",
        "Name": "/my-app-2",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                14,
                180
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/6293a959fac07f5702dba605f566904e8dbf1994b177c8b189151285135fa414-init/diff:/var/lib/docker/overlay2/eug75w8ytpkb93493m11yeh4b/diff:/var/lib/docker/overlay2/ty1kg9e4i2tfeubgqdxsz5tco/diff:/var/lib/docker/overlay2/f811057e0180fcd9bedbfe003ecc935a00be3445f58f7e3e11305e890a043e54/diff:/var/lib/docker/overlay2/2380f9512f486fd67ccc741fefa44180e6bb8831121d6e390747cca723676ddf/diff:/var/lib/docker/overlay2/4b9c7245651f7472de5f5eba6224e18dbe57afbdbc72689aa9fbdcc8fde1c297/diff:/var/lib/docker/overlay2/6489b9f61a2d339b3765e73e1423ef491290087b153b3d4c5579b45044411878/diff:/var/lib/docker/overlay2/bfe3618d9550ce9e8815bd9b2db8efe8f69a905a33fe59d743665c5e30d43212/diff:/var/lib/docker/overlay2/48e64b9f9b89a65776a89fd53ab75a123fd5b7bfe09618f908b7b500e7d0a94e/diff:/var/lib/docker/overlay2/b64c33883ed551a91642f6be4d60e2567d18b09734beae07c9bb67c77514c438/diff:/var/lib/docker/overlay2/51c61f105aeed8cd6d7afd6078cea939d6d2f1350105595c1cc5e2f69fbccb5f/diff:/var/lib/docker/overlay2/4450ce32dc416ad00e18cdb8ed5f1203753bd3638420433670cef1c23b478014/diff",
                "MergedDir": "/var/lib/docker/overlay2/6293a959fac07f5702dba605f566904e8dbf1994b177c8b189151285135fa414/merged",
                "UpperDir": "/var/lib/docker/overlay2/6293a959fac07f5702dba605f566904e8dbf1994b177c8b189151285135fa414/diff",
                "WorkDir": "/var/lib/docker/overlay2/6293a959fac07f5702dba605f566904e8dbf1994b177c8b189151285135fa414/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "83ed57056c49",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NODE_VERSION=18.15.0",
                "YARN_VERSION=1.22.19"
            ],
            "Cmd": [
                "node",
                "app.js"
            ],
            "Image": "node-app:0.2",
            "Volumes": null,
            "WorkingDir": "/app",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "a288ebcce87ac8c6384de9f460ab54e226f039bb65509614652d09bd01df84bb",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/a288ebcce87a",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "eaef85dcafcfd3f126ec01eb4172b606d9be18da3f41bc0f4e7e1a3626937b9a",
            "Gateway": "172.18.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.18.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:12:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "9f2d6f45709814a7b0f9afcd95d8d8a4638ee4e947b3dfee77effbb42a644a6e",
                    "EndpointID": "eaef85dcafcfd3f126ec01eb4172b606d9be18da3f41bc0f4e7e1a3626937b9a",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 83ed57056c49
172.18.0.3
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

### Create the target Docker repository

You must create a repository before you can push any images to it.  Pushing an image can't trigger creation of a repository and the Cloud  Build service account does not have permissions to create repositories.

1. From the **Navigation Menu**, under CI/CD navigate to **Artifact Registry** > **Repositories**.
2. Click **Create Repository**.
3. Specify `my-repository` as the repository name.
4. Choose **Docker** as the format.
5. Under Location Type, select **Region** and then choose the location `us-central1 (Iowa)`.
6. Click **Create**.



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ gcloud auth configure-docker us-central1-docker.pkg.dev
WARNING: Your config file at [/home/student_03_549093f60571/.docker/config.json] contains these credential helper entries:

{
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud"
  }
}
Adding credentials for: us-central1-docker.pkg.dev
After update, the following will be written to your Docker config file located at [/home/student_03_549093f60571/.docker/config.json]:
 {
  "credHelpers": {
    "gcr.io": "gcloud",
    "us.gcr.io": "gcloud",
    "eu.gcr.io": "gcloud",
    "asia.gcr.io": "gcloud",
    "staging-k8s.gcr.io": "gcloud",
    "marketplace.gcr.io": "gcloud",
    "us-central1-docker.pkg.dev": "gcloud"
  }
}

Do you want to continue (Y/n)?  y

Docker configuration file updated.
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

The command updates your Docker configuration. You can now connect with  Artifact Registry in your Google Cloud project to push and pull images.



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ export PROJECT_ID=$(gcloud config get-value project)
cd ~/test
Your active configuration is: [cloudshell-4812]
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```



```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker build -t us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2 .
[+] Building 0.4s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                           0.0s
 => => transferring dockerfile: 393B                                                                                                                                           0.0s
 => [internal] load .dockerignore                                                                                                                                              0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [internal] load metadata for docker.io/library/node:lts                                                                                                                    0.4s
 => [1/3] FROM docker.io/library/node:lts@sha256:c21209748c829660e0b49cbd14d2f9d81ea82ffb02a8a7932ebacf70d01573a3                                                              0.0s
 => [internal] load build context                                                                                                                                              0.0s
 => => transferring context: 58B                                                                                                                                               0.0s
 => CACHED [2/3] WORKDIR /app                                                                                                                                                  0.0s
 => CACHED [3/3] ADD . /app                                                                                                                                                    0.0s
 => exporting to image                                                                                                                                                         0.0s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:55b9141f933f08c107e20af6cd2593b3718be2bc8983e1ed4f5c2fe8a3ee71ad                                                                                   0.0s
 => => naming to us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app:0.2                                                                            0.0s
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker images
REPOSITORY                                                                       TAG       IMAGE ID       CREATED          SIZE
node-app                                                                         0.2       55b9141f933f   8 minutes ago    997MB
us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app   0.2       55b9141f933f   8 minutes ago    997MB
node-app                                                                         0.1       c9a9cefdba4f   14 minutes ago   997MB
hello-world                                                                      latest    feb5d9fea6a5   18 months ago    13.3kB
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker push us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
The push refers to repository [us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app]
4a79688c3175: Pushed
def957a1522c: Pushed
180f3dd985d9: Pushed
15bce0822cc5: Pushed
748074be23e4: Pushed
7efd70d0bc36: Pushed
d4514f8b2aac: Pushed
5ab567b9150b: Pushed
a90e3914fb92: Pushed
053a1f71007e: Pushed
ec09eb83ea03: Pushed
0.2: digest: sha256:d7c39f0bd6c94b79805f321727c83f8598ae2d69f3a5f86923bd832f8a0d5dad size: 2628
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```



### Test the image

You could start a new VM, ssh into that VM, and install gcloud.  For  simplicity, just remove all containers and images to simulate a fresh  environment.

1. Stop and remove all containers:

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker stop $(docker ps -q)
docker rm $(docker ps -aq)
83ed57056c49
6b0c27f8ae2b
83ed57056c49
6b0c27f8ae2b
b69394853641
2a412eea2e00
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker rmi us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
docker rmi node:lts
docker rmi -f $(docker images -aq) # remove remaining images
docker images
Untagged: us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app:0.2
Untagged: us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app@sha256:d7c39f0bd6c94b79805f321727c83f8598ae2d69f3a5f86923bd832f8a0d5dad
Error response from daemon: No such image: node:lts
Untagged: node-app:0.2
Deleted: sha256:55b9141f933f08c107e20af6cd2593b3718be2bc8983e1ed4f5c2fe8a3ee71ad
Untagged: node-app:0.1
Deleted: sha256:c9a9cefdba4fd7a3012df5c49ac47c373f7741f5a9afefca29bab3f77b40d144
Untagged: hello-world:latest
Untagged: hello-world@sha256:ffb13da98453e0f04d33a6eee5bb8e46ee50d08ebe17735fc0779d0349e889e9
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

```sh
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ docker pull us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
docker run -p 4000:80 -d us-central1-docker.pkg.dev/$PROJECT_ID/my-repository/node-app:0.2
curl http://localhost:4000
0.2: Pulling from qwiklabs-gcp-04-542391075613/my-repository/node-app
3e440a704568: Already exists
68a71c865a2c: Already exists
670730c27c2e: Already exists
5a7a2c95f0f8: Already exists
6d627e120214: Already exists
1dc4abe482bf: Already exists
e4b205f7d691: Already exists
47ae708a5b38: Already exists
c37e138d8eac: Already exists
586b78f433fb: Already exists
eeb761a78337: Already exists
Digest: sha256:d7c39f0bd6c94b79805f321727c83f8598ae2d69f3a5f86923bd832f8a0d5dad
Status: Downloaded newer image for us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app:0.2
us-central1-docker.pkg.dev/qwiklabs-gcp-04-542391075613/my-repository/node-app:0.2
35ac9fbb0fda296c0fd0cc78979953aefa4a3fabe75e55c5f5db1d75779da58d
curl: (56) Recv failure: Connection reset by peer
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$ curl http://localhost:4000
Welcome to Cloud
student_03_549093f60571@cloudshell:~/test (qwiklabs-gcp-04-542391075613)$
```

Here the portability of containers is showcased. As long as Docker is installed on the host (either on-premise or VM), it can pull images from public or private registries and run containers based on that image. There are no application dependencies that have to be installed on the host except for Docker.

So far, we 
- Ran containers based on public images from Docker Hub.
- Built your own container images and pushed them to Google Artifact Registry.
- Learned ways to debug running containers.
- Ran containers based on images pulled from Google Artifact Registry.

![]({{ site.url }}{{ site.baseurl }}/assets/images/Docker_7.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/Docker_6.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/Docker_5.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/Docker_4.png)

![]({{ site.url }}{{ site.baseurl }}/assets/images/Docker_1.png)

