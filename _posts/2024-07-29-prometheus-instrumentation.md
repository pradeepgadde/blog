---

layout: single
title:  "Prometheus: Instrumenting a HTTP Server"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/prometheus.webp
  og_image: /assets/images/prometheus.webp
  teaser: /assets/images/prometheus.webp
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Prometheus: Instrumenting a HTTP Server

In this tutorial we will create a simple Go HTTP server and instrumentation it by adding a counter metric to keep count of the total number of requests processed by the server.

Here we have a simple HTTP server with `/ping` endpoint which returns `pong` as response.

```go
package main

import (
   "fmt"
   "net/http"
)

func ping(w http.ResponseWriter, req *http.Request){
   fmt.Fprintf(w,"pong")
}

func main() {
   http.HandleFunc("/ping",ping)

   http.ListenAndServe(":8090", nil)
}
```

Compile and run the server

```go
go build server.go
./server
```

Now open `http://localhost:8090/ping` in your browser and you must see `pong`

```go
@pradeepgadde ➜ /workspaces/codespaces-blank $ cat server.go 
package main

import (
   "fmt"
   "net/http"
)

func ping(w http.ResponseWriter, req *http.Request){
   fmt.Fprintf(w,"pong")
}

func main() {
   http.HandleFunc("/ping",ping)

   http.ListenAndServe(":8090", nil)
}

```

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ go build server.go
@pradeepgadde ➜ /workspaces/codespaces-blank $ ./server 

```

https://cuddly-bassoon-gjqv7rwpgfwvpq-8090.app.github.dev/ping

Now lets add a metric to the server which will instrument the number  of requests made to the ping endpoint,the counter metric type is  suitable for this as we know the request count doesn’t go down and only  increases.

Create a Prometheus counter

Create a Prometheus counter

```
var pingCounter = prometheus.NewCounter(
   prometheus.CounterOpts{
       Name: "ping_request_count",
       Help: "No of request handled by Ping handler",
   },
)
```

Next lets update the ping Handler to increase the count of the counter using `pingCounter.Inc()`.

```
func ping(w http.ResponseWriter, req *http.Request) {
   pingCounter.Inc()
   fmt.Fprintf(w, "pong")
}
```

Then register the counter to the Default Register and expose the metrics.

```
func main() {
   prometheus.MustRegister(pingCounter)
   http.HandleFunc("/ping", ping)
   http.Handle("/metrics", promhttp.Handler())
   http.ListenAndServe(":8090", nil)
}
```

The `prometheus.MustRegister` function registers the pingCounter to the default Register. To expose the metrics the Go Prometheus client library provides the promhttp package. `promhttp.Handler()` provides a `http.Handler` which exposes the metrics registered in the Default Register.

The sample code depends on the  

```go
@pradeepgadde ➜ /workspaces/codespaces-blank $ cat prom_example.go 
package main

import (
   "fmt"
   "net/http"

   "github.com/prometheus/client_golang/prometheus"
   "github.com/prometheus/client_golang/prometheus/promhttp"
)

var pingCounter = prometheus.NewCounter(
   prometheus.CounterOpts{
       Name: "ping_request_count",
       Help: "No of request handled by Ping handler",
   },
)

func ping(w http.ResponseWriter, req *http.Request) {
   pingCounter.Inc()
   fmt.Fprintf(w, "pong")
}

func main() {
   prometheus.MustRegister(pingCounter)

   http.HandleFunc("/ping", ping)
   http.Handle("/metrics", promhttp.Handler())
   http.ListenAndServe(":8090", nil)
}
@pradeepgadde ➜ /workspaces/codespaces-blank $ go mod init prom_example
.gogo: creating new go.mod: module prom_example
go: to add module requirements and sums:
        go mod tidy
@pradeepgadde ➜ /workspaces/codespaces-blank $ go mod tidy
go: finding module for package github.com/prometheus/client_golang/prometheus
go: finding module for package github.com/prometheus/client_golang/prometheus/promhttp
go: downloading github.com/prometheus/client_golang v1.19.1
go: found github.com/prometheus/client_golang/prometheus in github.com/prometheus/client_golang v1.19.1
go: found github.com/prometheus/client_golang/prometheus/promhttp in github.com/prometheus/client_golang v1.19.1
go: downloading github.com/beorn7/perks v1.0.1
go: downloading github.com/cespare/xxhash/v2 v2.2.0
go: downloading github.com/prometheus/client_model v0.5.0
go: downloading github.com/prometheus/common v0.48.0
go: downloading github.com/prometheus/procfs v0.12.0
go: downloading golang.org/x/sys v0.17.0
go: downloading google.golang.org/protobuf v1.33.0
go: downloading github.com/davecgh/go-spew v1.1.1
go: downloading github.com/google/go-cmp v0.6.0
@pradeepgadde ➜ /workspaces/codespaces-blank $ go run server.go
@pradeepgadde ➜ /workspaces/codespaces-blank $ 
```

Now hit the localhost:8090/ping endpoint a couple of times and sending a request to localhost:8090 will provide the metrics.

Here the `ping_request_count` shows that `/ping` endpoint was called 3 times.

The Default Register comes with a collector for go runtime metrics and that is why we see other metrics like `go_threads`, `go_goroutines` etc.

We have built our first metric exporter. Let’s update our Prometheus config to scrape the metrics from our server.

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: simple_server
    static_configs:
      - targets: ["localhost:8090"]
```
