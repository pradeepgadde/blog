---
layout: single
title:  "Containerize a Python Application and deploy it in Kubernetes"
date:   2022-04-21 03:59:04 +0530
categories: Kubernetes
tags: kubeadm minikube
author: "Pradeep Gadde"
toc: true
toc_sticky: true
show_date: true
header:
  teaser: /assets/images/kubernetes.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar
---

# Containerize a Python Application and deploy it in Kubernetes

In this post, let us take simple Python Hello World! application which uses Flask mico framework for website development and run it inside a container. Then build a docker image for the same application and finally run it as a deployment in Kubernetes.

## Python Hello World Application

Here is the sample source code for running a **Flask** based website, which simply renders a text message!

```python
pradeep@learnk8s$ cat hello.py 
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

This simple application requires the presence of Python module named `Flask`, so let us include it in a separate file called `requirements.txt` which seems to be a common Python best practce.

```sh
pradeep@learnk8s$ cat requirements.txt 
Flask
```

Let us install this `Flask` module with the `pip3` command.

```sh
pradeep@learnk8s$ pip3 install -r requirements.txt 
DEPRECATION: Configuring installation scheme with distutils config files is deprecated and will no longer work in the near future. If you are using a Homebrew or Linuxbrew Python, please see discussion at https://github.com/Homebrew/homebrew-core/issues/76621
Collecting Flask
  Downloading Flask-2.1.1-py3-none-any.whl (95 kB)
     |████████████████████████████████| 95 kB 3.2 MB/s             
Requirement already satisfied: Jinja2>=3.0 in /usr/local/lib/python3.9/site-packages (from Flask->-r requirements.txt (line 1)) (3.0.3)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Requirement already satisfied: importlib-metadata>=3.6.0 in /usr/local/lib/python3.9/site-packages (from Flask->-r requirements.txt (line 1)) (4.11.3)
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.1.1-py3-none-any.whl (224 kB)
     |████████████████████████████████| 224 kB 2.5 MB/s            
Requirement already satisfied: click>=8.0 in /usr/local/lib/python3.9/site-packages (from Flask->-r requirements.txt (line 1)) (8.0.4)
Requirement already satisfied: zipp>=0.5 in /usr/local/lib/python3.9/site-packages (from importlib-metadata>=3.6.0->Flask->-r requirements.txt (line 1)) (3.7.0)
Requirement already satisfied: MarkupSafe>=2.0 in /usr/local/lib/python3.9/site-packages (from Jinja2>=3.0->Flask->-r requirements.txt (line 1)) (2.1.1)
Installing collected packages: Werkzeug, itsdangerous, Flask
  DEPRECATION: Configuring installation scheme with distutils config files is deprecated and will no longer work in the near future. If you are using a Homebrew or Linuxbrew Python, please see discussion at https://github.com/Homebrew/homebrew-core/issues/76621
  DEPRECATION: Configuring installation scheme with distutils config files is deprecated and will no longer work in the near future. If you are using a Homebrew or Linuxbrew Python, please see discussion at https://github.com/Homebrew/homebrew-core/issues/76621
DEPRECATION: Configuring installation scheme with distutils config files is deprecated and will no longer work in the near future. If you are using a Homebrew or Linuxbrew Python, please see discussion at https://github.com/Homebrew/homebrew-core/issues/76621
Successfully installed Flask-2.1.1 Werkzeug-2.1.1 itsdangerous-2.1.2
WARNING: You are using pip version 21.3.1; however, version 22.0.4 is available.
You should consider upgrading via the '/usr/local/opt/python@3.9/bin/python3.9 -m pip install --upgrade pip' command.
pradeep@learnk8s$  

```

Test the Python program directly by running it with the `python3` command followed by the name of the file `hello.py` in our case.

```sh
pradeep@learnk8s$ python3 hello.py 
 * Serving Flask app 'hello' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
Address already in use
Port 5000 is in use by another program. Either identify and stop that program, or start the server with a different port.
On macOS, try disabling the 'AirPlay Receiver' service from System Preferences -> Sharing.
pradeep@Pradeeps-MacBook-Air ~ %
```

Flask uses the `5000` port by default, which seems to be in use by another application. As seen at the bottom of the screen, in macOS, `AirPlay Receiver` seems to be using the same port. So I have disabled that service for now by going to `System Preferences -> Sharing`.

After disabling the `AirPlay Receiver`, try again,

```sh
pradeep@learnk8s$ sudo python3 hello.py
 * Serving Flask app 'hello' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses (0.0.0.0)
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:5000
 * Running on http://192.168.1.115:5000 (Press CTRL+C to quit)

```

We can see that our Python application is successfully running and can be accessed on the localhost:5000 port.

Let us test it by pointing our browser to this URL ` http://127.0.0.1:5000`.

![kubernetes-python-flask-app]({{ site.url }}{{ site.baseurl }}/assets/images/k8s_python_flask_1.png)



## Create Docker Image

Now that we have verified the python source code running successfully, let us containerize this application using Docker.

```sh
pradeep@learnk8s$ ls
hello.py		requirements.txt
pradeep@learnk8s$ vi Dockerfile
```
```dockerfile
pradeep@learnk8s$ cat Dockerfile 
FROM python:3.7

RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -r requirements.txt

EXPOSE 5000
CMD ["python", "/app/hello.py"]
 
```

This file is a set of instructions Docker will use to build the image. For this simple application, Docker is going to:

1. Get the official [Python Base Image](https://hub.docker.com/_/python/) for version 3.7 from Docker Hub.
2. In the image, create a directory named app.
3. Set the working directory to that new app directory.
4. Copy the local directory’s contents to that new folder into the image.
5. Run the pip installer (just like we did earlier) to pull the requirements into the image.
6. Inform Docker the container listens on port 5000.
7. Configure the starting command to use when the container starts.

Now build the image with the `docker build` command from the `Dockerfile`

```sh
pradeep@learnk8s$ docker build -f Dockerfile -t hello-world:latest .
[+] Building 169.2s (10/10) FINISHED                                                                     
 => [internal] load build definition from Dockerfile                                                0.9s
 => => transferring dockerfile: 182B                                                                0.5s
 => [internal] load .dockerignore                                                                   0.3s
 => => transferring context: 2B                                                                     0.1s
 => [internal] load metadata for docker.io/library/python:3.7                                       5.6s
 => [1/5] FROM docker.io/library/python:3.7@sha256:8900536b0b13e695036615f597fab5d975b59180c7f31  131.3s
 => => resolve docker.io/library/python:3.7@sha256:8900536b0b13e695036615f597fab5d975b59180c7f3179  0.0s
 => => sha256:0827a5451746929d077de6f275c61cffffdec172af2220bb05cb3e0eb8e6efce 2.22kB / 2.22kB      0.0s
 => => sha256:7c891de3e2203aa107263daf8adeacb43cc10b1103b97b666fd30efb781a04e4 9.20kB / 9.20kB      0.0s
 => => sha256:6aefca2dc61dcbcd268b8a9861e552f9cdb69e57242faec64ac120d2355a9c1a 54.94MB / 54.94MB   27.8s
 => => sha256:967757d5652770cfa81b6cc7577d65e06d336173da116d1fb5b2d349d5d44127 5.16MB / 5.16MB      4.2s
 => => sha256:c357e2c68cb3bf1e98dcb3eb6ceb16837253db71535921d6993c594588bffe04 10.87MB / 10.87MB    4.9s
 => => sha256:8900536b0b13e695036615f597fab5d975b59180c7f31793683f6564208dd174 1.86kB / 1.86kB      0.0s
 => => sha256:c766e27afb21eddf9ab3e4349700ebe697c32a4c6ada6af4f08282277a291a28 54.58MB / 54.58MB   29.2s
 => => sha256:32a180f5cf85702e7680719c40c39c07972b1176355df5a621de9eb87ad07ce 196.70MB / 196.70MB  64.1s
 => => sha256:1535e3c1181a81ea66d5bacb16564e4da2ba96304506598be39afe9c82b21c5c 6.29MB / 6.29MB     31.1s
 => => sha256:8ae21bbc51925b5cdcfea1665de073524cfff64ed8f221bd03955c5bb3d7e2e9 15.48MB / 15.48MB   34.0s
 => => extracting sha256:6aefca2dc61dcbcd268b8a9861e552f9cdb69e57242faec64ac120d2355a9c1a          16.6s
 => => sha256:7d9c83d514b05ab8f6fa3e5128313a52b71e63a395f7097f28cf4131d4045a7e 232B / 232B         31.4s
 => => sha256:52e9a986c056fba2c2222312026992ca7809b9b3e515eeea4cf752978d9bd9fb 2.87MB / 2.87MB     33.1s
 => => extracting sha256:967757d5652770cfa81b6cc7577d65e06d336173da116d1fb5b2d349d5d44127           1.6s
 => => extracting sha256:c357e2c68cb3bf1e98dcb3eb6ceb16837253db71535921d6993c594588bffe04           1.6s
 => => extracting sha256:c766e27afb21eddf9ab3e4349700ebe697c32a4c6ada6af4f08282277a291a28          19.2s
 => => extracting sha256:32a180f5cf85702e7680719c40c39c07972b1176355df5a621de9eb87ad07ce2          47.5s
 => => extracting sha256:1535e3c1181a81ea66d5bacb16564e4da2ba96304506598be39afe9c82b21c5c           1.4s
 => => extracting sha256:8ae21bbc51925b5cdcfea1665de073524cfff64ed8f221bd03955c5bb3d7e2e9           4.0s
 => => extracting sha256:7d9c83d514b05ab8f6fa3e5128313a52b71e63a395f7097f28cf4131d4045a7e           0.0s
 => => extracting sha256:52e9a986c056fba2c2222312026992ca7809b9b3e515eeea4cf752978d9bd9fb           2.4s
 => [internal] load build context                                                                   0.1s
 => => transferring context: 425B                                                                   0.0s
 => [2/5] RUN mkdir /app                                                                            5.9s
 => [3/5] WORKDIR /app                                                                              0.1s
 => [4/5] ADD . /app/                                                                               0.3s
 => [5/5] RUN pip install -r requirements.txt                                                      22.7s
 => exporting to image                                                                              1.5s
 => => exporting layers                                                                             1.5s 
 => => writing image sha256:c7cbee6913703f5955f89ddc5d5c2b0d5d3ef8ca7f026720f7e56bbf3eb67bba        0.0s
 => => naming to docker.io/library/hello-world:latest                                               0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
pradeep@learnk8s$ 

```

## Verify Docker Image

Verify that the image is created successfully, with the `docker image ls` command.

```sh
pradeep@learnk8s$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    c7cbee691370   3 minutes ago   918MB

```

The application is now containerized, which means it can now run in Docker and Kubernetes!

Before moving to Kubernetes, let us first test it in Docker.

We need to create a Docker container using this newly created image and expose the internal port (`5000` in our simple application). For this, we can use the `docker run` command with `-p` flag to publish the internal port 5000 as an external port 6000 on the host machine.

```sh
pradeep@learnk8s$ docker run -p 6000:5000 hello-world
 * Serving Flask app 'hello' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses (0.0.0.0)
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000 (Press CTRL+C to quit)


```

Let us go to the brower and open http://127.0.0.1:6000 or use the `curl` command.

```sh
pradeep@learnk8s$ curl 127.0.0.1:6000
Hello World!% 
```

> Note: We have to use the external port, 6000 but not the internal port 5000 to access the same Python application.

## Run as a Kubernetes Deployment

```sh
pradeep@learnk8s$ kubectl create deployment hello-world --image=hello-world:latest --dry-run=client -o yaml > hello-world.yaml
```
```yaml
pradeep@learnk8s$ cat hello-world.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-world
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world
    spec:
      containers:
      - image: hello-world:latest
        name: hello-world
        resources: {}
status: {}
```
Set the `imagePullPolicy` to  `Never`

```yaml
pradeep@learnk8s$ cat hello-world.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-world
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world
    spec:
      containers:
      - image: hello-world:latest
        name: hello-world
        imagePullPolicy: Never
        resources: {}
status: {}
```

```sh
pradeep@learnk8s$ kubectl create -f hello-world.yaml 
deployment.apps/hello-world created
```
Verify if the Pod is created successfully
```sh
pradeep@Pradeeps-MacBook-Air app % kubectl get pods
NAME                           READY   STATUS              RESTARTS   AGE
hello-world-5dc467cdbc-jz8bz   0/1     ErrImageNeverPull   0          13s
```
It looks like there is some issue with the ImagePull. 
Let us describe the pod for additional details.

```sh
pradeep@learnk8s$ kubectl describe pods hello-world-5dc467cdbc-jz8bz
Name:         hello-world-5dc467cdbc-jz8bz
Namespace:    default
Priority:     0
Node:         minikube-m02/172.16.30.7
Start Time:   Thu, 21 Apr 2022 21:45:46 +0530
Labels:       app=hello-world
              pod-template-hash=5dc467cdbc
Annotations:  cni.projectcalico.org/containerID: bdeee05134f995667fa844dd58eda993dd7f92e4c3079f89a7d60eb3680d52f0
              cni.projectcalico.org/podIP: 10.244.205.207/32
              cni.projectcalico.org/podIPs: 10.244.205.207/32
Status:       Pending
IP:           10.244.205.207
IPs:
  IP:           10.244.205.207
Controlled By:  ReplicaSet/hello-world-5dc467cdbc
Containers:
  hello-world:
    Container ID:   
    Image:          hello-world:latest
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ErrImageNeverPull
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cnrgd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-cnrgd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason             Age                From               Message
  ----     ------             ----               ----               -------
  Normal   Scheduled          25s                default-scheduler  Successfully assigned default/hello-world-5dc467cdbc-jz8bz to minikube-m02
  Normal   SandboxChanged     18s                kubelet            Pod sandbox changed, it will be killed and re-created.
  Warning  ErrImageNeverPull  12s (x4 over 18s)  kubelet            Container image "hello-world:latest" is not present with pull policy of Never
  Warning  Failed             12s (x4 over 18s)  kubelet            Error: ErrImageNeverPull
pradeep@learnk8s$
```
From the Events, we can see that the scheduler has assigned this Pod to `minikube-m02` node which does not have our Docker image `hello-world:latest`.
Remember we created the image on the MacOS (host machine)! 

In situations like this, usually we deploy our own docker registry (or we can publish our app to public repos like Docker Hub). We will discuss the private docker registry setup in a separate post.

For this demo, let us make this Docker image available on one of the minukube nodes, by building the image directly there.

For example, ssh to `minikube-m02` node and create the `app` directory and inside it the necessary `hello.py` and `requirements.txt`.

```sh
pradeep@Pradeeps-MacBook-Air app % minikube ssh -n minikube-m02
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ vi Dockerfile
$ mkdir app
$ mv Dockerfile app
$ cd app/
$ vi requirements.txt
$ vi hello.py
```

Build the Docker image now on this `minikube-m02` node.
```sh
$ docker build -f Dockerfile -t hello-world:latest .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM python:3.7
3.7: Pulling from library/python
6aefca2dc61d: Pull complete 
967757d56527: Pull complete 
c357e2c68cb3: Pull complete 
c766e27afb21: Pull complete 
32a180f5cf85: Pull complete 
1535e3c1181a: Pull complete 
8ae21bbc5192: Pull complete 
7d9c83d514b0: Pull complete 
52e9a986c056: Pull complete 
Digest: sha256:6ec7d453d5296215512f68ae09d9d25dd12c651b69e7a20060ff8adb4dba535f
Status: Downloaded newer image for python:3.7
 ---> 7c891de3e220
Step 2/7 : RUN mkdir /app
 ---> Running in 43b842d749b4
Removing intermediate container 43b842d749b4
 ---> cde208ac7945
Step 3/7 : WORKDIR /app
 ---> Running in 1d1b5664143e
Removing intermediate container 1d1b5664143e
 ---> 39441095dc55
Step 4/7 : ADD . /app/
 ---> 24d9d95ea0e2
Step 5/7 : RUN pip install -r requirements.txt
 ---> Running in 307ea6a706e6
Collecting Flask
  Downloading Flask-2.1.1-py3-none-any.whl (95 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.2/95.2 KB 971.2 kB/s eta 0:00:00
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.1.1-py3-none-any.whl (224 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 224.7/224.7 KB 683.6 kB/s eta 0:00:00
Collecting click>=8.0
  Downloading click-8.1.2-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 KB 418.8 kB/s eta 0:00:00
Collecting Jinja2>=3.0
  Downloading Jinja2-3.1.1-py3-none-any.whl (132 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 132.6/132.6 KB 526.0 kB/s eta 0:00:00
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting importlib-metadata>=3.6.0
  Downloading importlib_metadata-4.11.3-py3-none-any.whl (18 kB)
Collecting zipp>=0.5
  Downloading zipp-3.8.0-py3-none-any.whl (5.4 kB)
Collecting typing-extensions>=3.6.4
  Downloading typing_extensions-4.2.0-py3-none-any.whl (24 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.1-cp37-cp37m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Installing collected packages: zipp, Werkzeug, typing-extensions, MarkupSafe, itsdangerous, Jinja2, importlib-metadata, click, Flask
Successfully installed Flask-2.1.1 Jinja2-3.1.1 MarkupSafe-2.1.1 Werkzeug-2.1.1 click-8.1.2 importlib-metadata-4.11.3 itsdangerous-2.1.2 typing-extensions-4.2.0 zipp-3.8.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Removing intermediate container 307ea6a706e6
 ---> 2693eb5bffbf
Step 6/7 : EXPOSE 5000
 ---> Running in 918b7081eaba
Removing intermediate container 918b7081eaba
 ---> 6334551e71bc
Step 7/7 : CMD ["python", "/app/hello.py"]
 ---> Running in d69e225fc3fb
Removing intermediate container d69e225fc3fb
 ---> 4a92f7aad6d2
Successfully built 4a92f7aad6d2
Successfully tagged hello-world:latest
$ 
```
Verify that the `hello-world` image is available on this node now.

```sh
$ docker image ls 
REPOSITORY                                TAG                 IMAGE ID       CREATED          SIZE
hello-world                               latest              4a92f7aad6d2   31 seconds ago   918MB
```
Exit out of the `minikube-m02` node and modify the deployment definition to manually schedule the pod on this specific node, using the `nodeName` specification.

```yaml
pradeep@learnk8s$ cat hello-world.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-world
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world
    spec:
      nodeName: minikube-m02
      containers:
      - image: hello-world:latest
        name: hello-world
        imagePullPolicy: Never
        resources: {}
status: {}
 
```
```sh
pradeep@learnk8s$ kubectl create -f hello-world.yaml
deployment.apps/hello-world created
```
Verify if the Pod is created successfully.

```sh
pradeep@Pradeeps-MacBook-Air app % kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-world-5bd479745-qd8lf   1/1     Running   0          67s
```
Describe the `hello-world` pod.
```sh
pradeep@learnk8s$ kubectl describe pods hello-world-5bd479745-qd8lf
Name:         hello-world-5bd479745-qd8lf
Namespace:    default
Priority:     0
Node:         minikube-m02/172.16.30.7
Start Time:   Thu, 21 Apr 2022 22:10:45 +0530
Labels:       app=hello-world
              pod-template-hash=5bd479745
Annotations:  cni.projectcalico.org/containerID: 97379022ea4d88015577a9afd7ce220337a50c5b2da7101f8f1e256c49e3c324
              cni.projectcalico.org/podIP: 10.244.205.214/32
              cni.projectcalico.org/podIPs: 10.244.205.214/32
Status:       Running
IP:           10.244.205.214
IPs:
  IP:           10.244.205.214
Controlled By:  ReplicaSet/hello-world-5bd479745
Containers:
  hello-world:
    Container ID:   docker://c5a212dbdd1329efbb8d997f00de6d041ccccf65fa0aee462288567ee055e453
    Image:          hello-world:latest
    Image ID:       docker://sha256:4a92f7aad6d2ddad6d01d2bedb0ff7408b659cce36d248f39d92337cd28198ac
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 21 Apr 2022 22:10:50 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ng695 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-ng695:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulled   107s  kubelet  Container image "hello-world:latest" already present on machine
  Normal  Created  107s  kubelet  Created container hello-world
  Normal  Started  107s  kubelet  Started container hello-world
pradeep@learnk8s$ 
```


We can see that the pod is using the local image from this Event `Container image "hello-world:latest" already present on machine`.

## Verify Kubernetes Deployment
As we have not yet created any Kubernetes service for this deployment, we can test the application using the Pod IP from within the cluster.

From the above description, Pod IP address is `IP:           10.244.205.214`.

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m02
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)


$ curl 10.244.205.214:5000
Hello World!$ 
```
Similarly, we can test from any other Kubernetes cluster node member.
For example,	from `minikube-m03` node

```sh
pradeep@learnk8s$ minikube ssh -n minikube-m03
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 10.244.205.214:5000
Hello World!$ exit
logout
pradeep@learnk8s$ 
```

Finally, let us create a Service for this deployment and make it available externally.

## Create a Kubernetes Service

Instead of creating the YAML file from scratch, let us make use of the help option of the `kubectl expose` command.

```sh
pradeep@learnk8s$ kubectl expose deployment hello-world -h        
Expose a resource as a new Kubernetes service.

 Looks up a deployment, service, replica set, replication controller or pod by name and uses the
selector for that resource as the selector for a new service on the specified port. A deployment or
replica set will be exposed as a service only if its selector is convertible to a selector that
service supports, i.e. when the selector contains only the matchLabels component. Note that if no
port is specified via --port and the exposed resource has multiple ports, all will be re-used by the
new service. Also if no labels are specified, the new service will re-use the labels from the
resource it exposes.

 Possible resources include (case insensitive):

 pod (po), service (svc), replicationcontroller (rc), deployment (deploy), replicaset (rs)

Examples:
  # Create a service for a replicated nginx, which serves on port 80 and connects to the containers
on port 8000
  kubectl expose rc nginx --port=80 --target-port=8000
  
  # Create a service for a replication controller identified by type and name specified in
"nginx-controller.yaml", which serves on port 80 and connects to the containers on port 8000
  kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000
  
  # Create a service for a pod valid-pod, which serves on port 444 with the name "frontend"
  kubectl expose pod valid-pod --port=444 --name=frontend
  
  # Create a second service based on the above service, exposing the container port 8443 as port 443
with the name "nginx-https"
  kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https
  
  # Create a service for a replicated streaming application on port 4100 balancing UDP traffic and
named 'video-stream'.
  kubectl expose rc streamer --port=4100 --protocol=UDP --name=video-stream
  
  # Create a service for a replicated nginx using replica set, which serves on port 80 and connects
to the containers on port 8000
  kubectl expose rs nginx --port=80 --target-port=8000
  
  # Create a service for an nginx deployment, which serves on port 80 and connects to the containers
on port 8000
  kubectl expose deployment nginx --port=80 --target-port=8000

Options:
      --allow-missing-template-keys=true: If true, ignore any errors in templates when a field or
map key is missing in the template. Only applies to golang and jsonpath output formats.
      --cluster-ip='': ClusterIP to be assigned to the service. Leave empty to auto-allocate, or set
to 'None' to create a headless service.
      --dry-run='none': Must be "none", "server", or "client". If client strategy, only print the
object that would be sent, without sending it. If server strategy, submit server-side request
without persisting the resource.
      --external-ip='': Additional external IP address (not managed by Kubernetes) to accept for the
service. If this IP is routed to a node, the service can be accessed by this IP in addition to its
generated service IP.
      --field-manager='kubectl-expose': Name of the manager used to track field ownership.
  -f, --filename=[]: Filename, directory, or URL to files identifying the resource to expose a
service
      --generator='service/v2': The name of the API generator to use. There are 2 generators:
'service/v1' and 'service/v2'. The only difference between them is that service port in v1 is named
'default', while it is left unnamed in v2. Default is 'service/v2'.
  -k, --kustomize='': Process the kustomization directory. This flag can't be used together with -f
or -R.
  -l, --labels='': Labels to apply to the service created by this call.
      --load-balancer-ip='': IP to assign to the LoadBalancer. If empty, an ephemeral IP will be
created and used (cloud-provider specific).
      --name='': The name for the newly created object.
  -o, --output='': Output format. One of:
json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file.
      --overrides='': An inline JSON override for the generated object. If this is non-empty, it is
used to override the generated object. Requires that the object supply a valid apiVersion field.
      --port='': The port that the service should serve on. Copied from the resource being exposed,
if unspecified
      --protocol='': The network protocol for the service to be created. Default is 'TCP'.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you
want to manage related manifests organized within the same directory.
      --save-config=false: If true, the configuration of current object will be saved in its
annotation. Otherwise, the annotation will be unchanged. This flag is useful when you want to
perform kubectl apply on this object in the future.
      --selector='': A label selector to use for this service. Only equality-based selector
requirements are supported. If empty (the default) infer the selector from the replication
controller or replica set.)
      --session-affinity='': If non-empty, set the session affinity for the service to this; legal
values: 'None', 'ClientIP'
      --show-managed-fields=false: If true, keep the managedFields when printing objects in JSON or
YAML format.
      --target-port='': Name or number for the port on the container that the service should direct
traffic to. Optional.
      --template='': Template string or path to template file to use when -o=go-template,
-o=go-template-file. The template format is golang templates
[http://golang.org/pkg/text/template/#pkg-overview].
      --type='': Type for this service: ClusterIP, NodePort, LoadBalancer, or ExternalName. Default
is 'ClusterIP'.

Usage:
  kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP|SCTP]
[--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]
[options]

Use "kubectl options" for a list of global command-line options (applies to all commands).
pradeep@learnk8s$ 
```

Using this help output, we can easily create a service for our deployment.

```yaml
pradeep@learnk8s$ kubectl expose deployment hello-world --type=LoadBalancer --port=6000 --target-port=5000 --name=hello-world --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: hello-world
  name: hello-world
spec:
  ports:
  - port: 6000
    protocol: TCP
    targetPort: 5000
  selector:
    app: hello-world
  type: LoadBalancer
status:
  loadBalancer: {}
```
Save this ouput to a file named `hello-world-service.yaml` and create the resource.
```sh
pradeep@learnk8s$ kubectl expose deployment hello-world --type=LoadBalancer --port=6000 --target-port=5000 --name=hello-world --dry-run=client -o yaml > hello-world-service.yaml
pradeep@learnk8s$ kubectl create -f hello-world-service.yaml 
service/hello-world created
```
Verify the service 

```sh
pradeep@learnk8s$ kubectl get svc
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
hello-world   LoadBalancer   10.111.70.62   <pending>     6000:32374/TCP   34s
kubernetes    ClusterIP      10.96.0.1      <none>        443/TCP          32d
```
We can see the `hello-world` service of type `LoadBalancer` but it is in `pending` state. In minikube setup, we know that we have to use the `minikube tunnel` command for this to change from `pending` to an IP address.

so let us run `minikube tunnel` in a new window.
```sh
pradeep@learnk8s$ minikube tunnel
Password:
Status:	
	machine: minikube
	pid: 70758
	route: 10.96.0.0/12 -> 172.16.30.6
	minikube: Running
	services: [hello-world, pacman, web]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors
```
Verify the service status now
```sh
pradeep@learnk8s$ kubectl get svc
NAME          TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
hello-world   LoadBalancer   10.111.70.62   10.111.70.62   6000:32374/TCP   2m29s
kubernetes    ClusterIP      10.96.0.1      <none>         443/TCP          32d
```

The `hello-world` service have an external-IP of `10.111.70.62` now.

Let us get the URL to test external access, using the `minikube service` command with the `--url` option
```sh
pradeep@learnk8s$ minikube service hello-world --url
http://172.16.30.6:32374
```

## Final Verfication of the Python App
Finally, we can use this URL http://172.16.30.6:32374 from our outside host to verify that our Flask app is successfully running in Kubernetes or not!

![kubernetes-python-flask-app]({{ site.url }}{{ site.baseurl }}/assets/images/k8s_python_flask_2.png)

This confirms a Success!

Also, this concludes our discussion on how to containerize an application, and get it running in Docker and in Kubernetes. 

Thank you for reading.



Happy Learning!

Best regards,

Pradeep

