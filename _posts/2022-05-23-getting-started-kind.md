---
layout: single
title:  "Getting Started with Kind"
date:   2022-05-23 10:55:04 +0530
categories: Kubernetes
tags: kind
author: "Pradeep Gadde"
classes: wide
show_date: true
header:
  teaser: /assets/images/kind.png
author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"

sidebar:
  - title: "Blog"
    text: "Checkout other topics"
    nav: my-sidebar
---

In this post, let us look at another option to try and learn Kubernetes locally, an alternative to minikube. It is called `kind` which stands for `Kubernetes in Docker`.

kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

If you have go (1.17+) and docker installed `go install sigs.k8s.io/kind@v0.14.0 && kind create cluster` is all you need!

According to the official documentation, kind consists of:

Go packages implementing cluster creation, image build, etc.
A command line interface (kind) built on these packages.
Docker image(s) written to run systemd, Kubernetes, etc.
kubetest integration also built on these packages (WIP)
kind bootstraps each “node” with kubeadm.

As I have recently started learning Go, I wanted to try the `kind` install directly from the source.

First, let us check the version of `Go`.

```go
pradeep:~$go version
go version go1.18.1 darwin/amd64
pradeep:~$
```

Per official docs,  we have two options: `go get` or `go install`.

For Go versions go1.17 and higher, you should use to `go install sigs.k8s.io/kind@v0.14.0` per https://tip.golang.org/doc/go1.17#go-get

For older versions use `GO111MODULE="on" go get sigs.k8s.io/kind@v0.14.0`.

Since we have `go1.18.1`, let us use the `go install` command.

```go
pradeep:~$go install sigs.k8s.io/kind@v0.14.0
go: downloading sigs.k8s.io/kind v0.14.0
go: downloading github.com/spf13/pflag v1.0.5
go: downloading github.com/spf13/cobra v1.4.0
go: downloading github.com/alessio/shellescape v1.4.1
go: downloading github.com/mattn/go-isatty v0.0.14
go: downloading github.com/pkg/errors v0.9.1
go: downloading golang.org/x/sys v0.0.0-20210630005230-0f9fa26af87c
go: downloading github.com/pelletier/go-toml v1.9.4
go: downloading gopkg.in/yaml.v3 v3.0.0-20210107192922-496545a6307b
go: downloading github.com/BurntSushi/toml v1.0.0
go: downloading github.com/evanphx/json-patch/v5 v5.6.0
go: downloading sigs.k8s.io/yaml v1.3.0
go: downloading gopkg.in/yaml.v2 v2.4.0
pradeep:~$
```

`go get` / `go install` will typically put the `kind` binary inside the `bin` directory under `go env` ` GOPATH`.

```go
pradeep:~$go env | grep PATH      
GOPATH="/Users/pradeep/go"
pradeep:~$
```

```go
pradeep:~$ls /Users/pradeep/go
bin	example	pkg
pradeep:~$ls /Users/pradeep/go/bin 
kind
pradeep:~$
```

I have moved this binary to the standard `bin` directory.

```sh
pradeep:~$sudo mv /Users/pradeep/Go/bin/kind /usr/local/bin 
Password:
```
Now we can see, `kind` command is recognized.

```sh
pradeep:~$kind
kind creates and manages local Kubernetes clusters using Docker container 'nodes'

Usage:
  kind [command]

Available Commands:
  build       Build one of [node-image]
  completion  Output shell completion code for the specified shell (bash, zsh or fish)
  create      Creates one of [cluster]
  delete      Deletes one of [cluster]
  export      Exports one of [kubeconfig, logs]
  get         Gets one of [clusters, nodes, kubeconfig]
  help        Help about any command
  load        Loads images into nodes
  version     Prints the kind CLI version

Flags:
  -h, --help              help for kind
      --loglevel string   DEPRECATED: see -v instead
  -q, --quiet             silence all stderr output
  -v, --verbosity int32   info log verbosity, higher value produces more output
      --version           version for kind

Use "kind [command] --help" for more information about a command.
pradeep:~$
```
Let us make use of `help` option here 

```sh
pradeep:~$kind help create
Creates one of local Kubernetes cluster (cluster)

Usage:
  kind create [flags]
  kind create [command]

Available Commands:
  cluster     Creates a local Kubernetes cluster

Flags:
  -h, --help   help for create

Global Flags:
      --loglevel string   DEPRECATED: see -v instead
  -q, --quiet             silence all stderr output
  -v, --verbosity int32   info log verbosity, higher value produces more output

Use "kind create [command] --help" for more information about a command.
pradeep:~$
```

