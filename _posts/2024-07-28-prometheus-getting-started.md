---

layout: single
title:  "Install, configure, and use a simple Prometheus instance"
categories: Cloud
tags: GCP
classes: wide
show_date: true
header:
  overlay_image: /assets/images/prometheus.png
  og_image: /assets/images/prometheus.png
  teaser: /assets/images/prometheus.png
author:
  name     : "Professional Cloud Architect"
  avatar   : "/assets/images/pca-gcp.png"

sidebar:
  - title: "Topics"
    nav: my-sidebar

---
# Install, configure, and use a simple Prometheus instance

- download and run Prometheus locally,
-  configure it to scrape itself 
- scrape an example application,
- work with queries, rules, and graphs to use collected time series data.

## Downloading and running Prometheus

[Download the latest release](https://prometheus.io/download) of Prometheus for your platform, then extract and run it:

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ wget https://github.com/prometheus/prometheus/releases/download/v2.54.0-rc.0/prometheus-2.54.0-rc.0.linux-amd64.tar.gz
--2024-08-01 02:16:35--  https://github.com/prometheus/prometheus/releases/download/v2.54.0-rc.0/prometheus-2.54.0-rc.0.linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.114.4
Connecting to github.com (github.com)|140.82.114.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/6838921/100b593f-8b0f-45ab-9794-3ee4895def3e?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240801%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240801T021619Z&X-Amz-Expires=300&X-Amz-Signature=87f55ad1ba04aef26a0605b4b09787686784baa8d7f9764aef372a8431564ea1&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=6838921&response-content-disposition=attachment%3B%20filename%3Dprometheus-2.54.0-rc.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2024-08-01 02:16:35--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/6838921/100b593f-8b0f-45ab-9794-3ee4895def3e?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240801%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240801T021619Z&X-Amz-Expires=300&X-Amz-Signature=87f55ad1ba04aef26a0605b4b09787686784baa8d7f9764aef372a8431564ea1&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=6838921&response-content-disposition=attachment%3B%20filename%3Dprometheus-2.54.0-rc.0.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 105692266 (101M) [application/octet-stream]
Saving to: ‘prometheus-2.54.0-rc.0.linux-amd64.tar.gz’

prometheus-2.54.0-rc.0.lin 100%[=====================================>] 100.80M  70.5MB/s    in 1.4s    

2024-08-01 02:16:37 (70.5 MB/s) - ‘prometheus-2.54.0-rc.0.linux-amd64.tar.gz’ saved [105692266/105692266]

@pradeepgadde ➜ /workspaces/codespaces-blank $ l
prometheus-2.54.0-rc.0.linux-amd64.tar.gz
@pradeepgadde ➜ /workspaces/codespaces-blank $ ls
prometheus-2.54.0-rc.0.linux-amd64.tar.gz
@pradeepgadde ➜ /workspaces/codespaces-blank $ tar xvfz prometheus-*.tar.gz
prometheus-2.54.0-rc.0.linux-amd64/
prometheus-2.54.0-rc.0.linux-amd64/promtool
prometheus-2.54.0-rc.0.linux-amd64/prometheus.yml
prometheus-2.54.0-rc.0.linux-amd64/consoles/
prometheus-2.54.0-rc.0.linux-amd64/consoles/prometheus-overview.html
prometheus-2.54.0-rc.0.linux-amd64/consoles/node.html
prometheus-2.54.0-rc.0.linux-amd64/consoles/node-disk.html
prometheus-2.54.0-rc.0.linux-amd64/consoles/index.html.example
prometheus-2.54.0-rc.0.linux-amd64/consoles/node-overview.html
prometheus-2.54.0-rc.0.linux-amd64/consoles/prometheus.html
prometheus-2.54.0-rc.0.linux-amd64/consoles/node-cpu.html
prometheus-2.54.0-rc.0.linux-amd64/NOTICE
prometheus-2.54.0-rc.0.linux-amd64/LICENSE
prometheus-2.54.0-rc.0.linux-amd64/prometheus
prometheus-2.54.0-rc.0.linux-amd64/console_libraries/
prometheus-2.54.0-rc.0.linux-amd64/console_libraries/prom.lib
prometheus-2.54.0-rc.0.linux-amd64/console_libraries/menu.lib
@pradeepgadde ➜ /workspaces/codespaces-blank $ ls
prometheus-2.54.0-rc.0.linux-amd64  prometheus-2.54.0-rc.0.linux-amd64.tar.gz
@pradeepgadde ➜ /workspaces/codespaces-blank $ cd prometheus-2.54.0-rc.0.linux-amd64/
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ ls
LICENSE  NOTICE  console_libraries  consoles  prometheus  prometheus.yml  promtool
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ 
```

## Configuring Prometheus to monitor itself

Prometheus collects metrics from *targets* by scraping metrics HTTP endpoints. Since Prometheus exposes data in the same manner about itself, it can also scrape and monitor its own health.

While a Prometheus server that collects only data about itself is not very useful, it is a good starting example. Save the following basic Prometheus configuration as a file named `prometheus.yml`:

```
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ cat prometheus.yml 
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ 
```

## Starting Prometheus

To start Prometheus with your newly created configuration file, change to the directory containing the Prometheus binary and run:

```sh
# Start Prometheus.
# By default, Prometheus stores its database in ./data (flag --storage.tsdb.path).
./prometheus --config.file=prometheus.yml
```

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ # Start Prometheus.
s its database in ./data (flag --storage.tsdb.path).
./prometheus --config.file=prometheus.yml@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ # By default, Prometheus stores its database in ./data (flag --storage.tsdb.path).
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ ./prometheus --config.file=prometheus.yml
ts=2024-08-01T02:19:42.316Z caller=main.go:601 level=info msg="No time or size retention was set so using the default time retention" duration=15d
ts=2024-08-01T02:19:42.316Z caller=main.go:645 level=info msg="Starting Prometheus Server" mode=server version="(version=2.54.0-rc.0, branch=HEAD, revision=2898d5d715738776bf8dd31d424a9b297380a2f3)"
ts=2024-08-01T02:19:42.316Z caller=main.go:650 level=info build_context="(go=go1.22.5, platform=linux/amd64, user=root@55e355ec479f, date=20240730-09:46:56, tags=netgo,builtinassets,stringlabels)"
ts=2024-08-01T02:19:42.316Z caller=main.go:651 level=info host_details="(Linux 6.5.0-1022-azure #23~22.04.1-Ubuntu SMP Thu May  9 17:59:24 UTC 2024 x86_64 codespaces-49988b (none))"
ts=2024-08-01T02:19:42.316Z caller=main.go:652 level=info fd_limits="(soft=1048576, hard=1048576)"
ts=2024-08-01T02:19:42.316Z caller=main.go:653 level=info vm_limits="(soft=unlimited, hard=unlimited)"
ts=2024-08-01T02:19:42.320Z caller=web.go:571 level=info component=web msg="Start listening for connections" address=0.0.0.0:9090
ts=2024-08-01T02:19:42.321Z caller=main.go:1160 level=info msg="Starting TSDB ..."
ts=2024-08-01T02:19:42.324Z caller=tls_config.go:313 level=info component=web msg="Listening on" address=[::]:9090
ts=2024-08-01T02:19:42.324Z caller=tls_config.go:316 level=info component=web msg="TLS is disabled." http2=false address=[::]:9090
ts=2024-08-01T02:19:42.325Z caller=head.go:626 level=info component=tsdb msg="Replaying on-disk memory mappable chunks if any"
ts=2024-08-01T02:19:42.325Z caller=head.go:713 level=info component=tsdb msg="On-disk memory mappable chunks replay completed" duration=1.492µs
ts=2024-08-01T02:19:42.325Z caller=head.go:721 level=info component=tsdb msg="Replaying WAL, this may take a while"
ts=2024-08-01T02:19:42.325Z caller=head.go:793 level=info component=tsdb msg="WAL segment loaded" segment=0 maxSegment=0
ts=2024-08-01T02:19:42.325Z caller=head.go:830 level=info component=tsdb msg="WAL replay completed" checkpoint_replay_duration=48.902µs wal_replay_duration=195.715µs wbl_replay_duration=150ns chunk_snapshot_load_duration=0s mmap_chunk_replay_duration=1.492µs total_replay_duration=302.504µs
ts=2024-08-01T02:19:42.327Z caller=main.go:1181 level=info fs_type=EXT4_SUPER_MAGIC
ts=2024-08-01T02:19:42.327Z caller=main.go:1184 level=info msg="TSDB started"
ts=2024-08-01T02:19:42.327Z caller=main.go:1367 level=info msg="Loading configuration file" filename=prometheus.yml
ts=2024-08-01T02:19:42.327Z caller=main.go:1404 level=info msg="updated GOGC" old=100 new=75
ts=2024-08-01T02:19:42.327Z caller=main.go:1415 level=info msg="Completed loading of configuration file" filename=prometheus.yml totalDuration=469.696µs db_storage=862ns remote_storage=1.132µs web_handler=221ns query_engine=510ns scrape=194.262µs scrape_sd=15.82µs notify=18.184µs notify_sd=7.905µs rules=1.143µs tracing=5.34µs
ts=2024-08-01T02:19:42.327Z caller=main.go:1145 level=info msg="Server is ready to receive web requests."
ts=2024-08-01T02:19:42.327Z caller=manager.go:164 level=info component="rule manager" msg="Starting rule manager..."

```

Prometheus should start up. You should also be able to browse to a status page about itself at [localhost:9090](http://localhost:9090). Give it a couple of seconds to collect data about itself from its own HTTP metrics endpoint.

You can also verify that Prometheus is serving metrics about itself by navigating to its metrics endpoint: [localhost:9090/metrics](http://localhost:9090/metrics)



In this case

https://cuddly-bassoon-gjqv7rwpgfwvpq-9090.app.github.dev/metrics

```sh
# HELP go_gc_cycles_automatic_gc_cycles_total Count of completed GC cycles generated by the Go runtime.
# TYPE go_gc_cycles_automatic_gc_cycles_total counter
go_gc_cycles_automatic_gc_cycles_total 6
# HELP go_gc_cycles_forced_gc_cycles_total Count of completed GC cycles forced by the application.
# TYPE go_gc_cycles_forced_gc_cycles_total counter
go_gc_cycles_forced_gc_cycles_total 0
# HELP go_gc_cycles_total_gc_cycles_total Count of all completed GC cycles.
# TYPE go_gc_cycles_total_gc_cycles_total counter
go_gc_cycles_total_gc_cycles_total 6
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.5226e-05
go_gc_duration_seconds{quantile="0.25"} 4.9483e-05
go_gc_duration_seconds{quantile="0.5"} 8.2851e-05
go_gc_duration_seconds{quantile="0.75"} 0.000223848
go_gc_duration_seconds{quantile="1"} 0.004804758
go_gc_duration_seconds_sum 0.005254825
go_gc_duration_seconds_count 6
# HELP go_gc_gogc_percent Heap size target percentage configured by the user, otherwise 100. This value is set by the GOGC environment variable, and the runtime/debug.SetGCPercent function.
# TYPE go_gc_gogc_percent gauge
go_gc_gogc_percent 75
# HELP go_gc_gomemlimit_bytes Go runtime memory limit configured by the user, otherwise math.MaxInt64. This value is set by the GOMEMLIMIT environment variable, and the runtime/debug.SetMemoryLimit function.
# TYPE go_gc_gomemlimit_bytes gauge
go_gc_gomemlimit_bytes 9.223372036854776e+18
# HELP go_gc_heap_allocs_by_size_bytes Distribution of heap allocations by approximate size. Bucket counts increase monotonically. Note that this does not include tiny objects as defined by /gc/heap/tiny/allocs:objects, only tiny blocks.
# TYPE go_gc_heap_allocs_by_size_bytes histogram
go_gc_heap_allocs_by_size_bytes_bucket{le="8.999999999999998"} 5226
go_gc_heap_allocs_by_size_bytes_bucket{le="24.999999999999996"} 35986
go_gc_heap_allocs_by_size_bytes_bucket{le="64.99999999999999"} 113247
go_gc_heap_allocs_by_size_bytes_bucket{le="144.99999999999997"} 149274
go_gc_heap_allocs_by_size_bytes_bucket{le="320.99999999999994"} 155664
go_gc_heap_allocs_by_size_bytes_bucket{le="704.9999999999999"} 156928
go_gc_heap_allocs_by_size_bytes_bucket{le="1536.9999999999998"} 157561
go_gc_heap_allocs_by_size_bytes_bucket{le="3200.9999999999995"} 157971
go_gc_heap_allocs_by_size_bytes_bucket{le="6528.999999999999"} 158261
go_gc_heap_allocs_by_size_bytes_bucket{le="13568.999999999998"} 158421
go_gc_heap_allocs_by_size_bytes_bucket{le="27264.999999999996"} 158479
go_gc_heap_allocs_by_size_bytes_bucket{le="+Inf"} 158583
go_gc_heap_allocs_by_size_bytes_sum 4.0746784e+07
go_gc_heap_allocs_by_size_bytes_count 158583
# HELP go_gc_heap_allocs_bytes_total Cumulative sum of memory allocated to the heap by the application.
# TYPE go_gc_heap_allocs_bytes_total counter
go_gc_heap_allocs_bytes_total 4.0746784e+07
# HELP go_gc_heap_allocs_objects_total Cumulative count of heap allocations triggered by the application. Note that this does not include tiny objects as defined by /gc/heap/tiny/allocs:objects, only tiny blocks.
# TYPE go_gc_heap_allocs_objects_total counter
go_gc_heap_allocs_objects_total 158583
# HELP go_gc_heap_frees_by_size_bytes Distribution of freed heap allocations by approximate size. Bucket counts increase monotonically. Note that this does not include tiny objects as defined by /gc/heap/tiny/allocs:objects, only tiny blocks.
# TYPE go_gc_heap_frees_by_size_bytes histogram
go_gc_heap_frees_by_size_bytes_bucket{le="8.999999999999998"} 2278
go_gc_heap_frees_by_size_bytes_bucket{le="24.999999999999996"} 17200
go_gc_heap_frees_by_size_bytes_bucket{le="64.99999999999999"} 42685
go_gc_heap_frees_by_size_bytes_bucket{le="144.99999999999997"} 55841
go_gc_heap_frees_by_size_bytes_bucket{le="320.99999999999994"} 57739
go_gc_heap_frees_by_size_bytes_bucket{le="704.9999999999999"} 58253
go_gc_heap_frees_by_size_bytes_bucket{le="1536.9999999999998"} 58575
go_gc_heap_frees_by_size_bytes_bucket{le="3200.9999999999995"} 58716
go_gc_heap_frees_by_size_bytes_bucket{le="6528.999999999999"} 58815
go_gc_heap_frees_by_size_bytes_bucket{le="13568.999999999998"} 58856
go_gc_heap_frees_by_size_bytes_bucket{le="27264.999999999996"} 58880
go_gc_heap_frees_by_size_bytes_bucket{le="+Inf"} 58942
go_gc_heap_frees_by_size_bytes_sum 2.4557192e+07
go_gc_heap_frees_by_size_bytes_count 58942
# HELP go_gc_heap_frees_bytes_total Cumulative sum of heap memory freed by the garbage collector.
# TYPE go_gc_heap_frees_bytes_total counter
go_gc_heap_frees_bytes_total 2.4557192e+07
# HELP go_gc_heap_frees_objects_total Cumulative count of heap allocations whose storage was freed by the garbage collector. Note that this does not include tiny objects as defined by /gc/heap/tiny/allocs:objects, only tiny blocks.
# TYPE go_gc_heap_frees_objects_total counter
go_gc_heap_frees_objects_total 58942
# HELP go_gc_heap_goal_bytes Heap size target for the end of the GC cycle.
# TYPE go_gc_heap_goal_bytes gauge
go_gc_heap_goal_bytes 2.183524e+07
# HELP go_gc_heap_live_bytes Heap memory occupied by live objects that were marked by the previous GC.
# TYPE go_gc_heap_live_bytes gauge
go_gc_heap_live_bytes 1.2102256e+07
# HELP go_gc_heap_objects_objects Number of objects, live or unswept, occupying heap memory.
# TYPE go_gc_heap_objects_objects gauge
go_gc_heap_objects_objects 99641
# HELP go_gc_heap_tiny_allocs_objects_total Count of small allocations that are packed together into blocks. These allocations are counted separately from other allocations because each individual allocation is not tracked by the runtime, only their block. Each block is already accounted for in allocs-by-size and frees-by-size.
# TYPE go_gc_heap_tiny_allocs_objects_total counter
go_gc_heap_tiny_allocs_objects_total 6139
# HELP go_gc_limiter_last_enabled_gc_cycle GC cycle the last time the GC CPU limiter was enabled. This metric is useful for diagnosing the root cause of an out-of-memory error, because the limiter trades memory for CPU time when the GC's CPU time gets too high. This is most likely to occur with use of SetMemoryLimit. The first GC cycle is cycle 1, so a value of 0 indicates that it was never enabled.
# TYPE go_gc_limiter_last_enabled_gc_cycle gauge
go_gc_limiter_last_enabled_gc_cycle 0
# HELP go_gc_pauses_seconds Deprecated. Prefer the identical /sched/pauses/total/gc:seconds.
# TYPE go_gc_pauses_seconds histogram
go_gc_pauses_seconds_bucket{le="6.399999999999999e-08"} 0
go_gc_pauses_seconds_bucket{le="6.399999999999999e-07"} 0
go_gc_pauses_seconds_bucket{le="7.167999999999999e-06"} 2
go_gc_pauses_seconds_bucket{le="8.191999999999999e-05"} 10
go_gc_pauses_seconds_bucket{le="0.0009175039999999999"} 11
go_gc_pauses_seconds_bucket{le="0.010485759999999998"} 12
go_gc_pauses_seconds_bucket{le="0.11744051199999998"} 12
go_gc_pauses_seconds_bucket{le="+Inf"} 12
go_gc_pauses_seconds_sum 0.001058048
go_gc_pauses_seconds_count 12
# HELP go_gc_scan_globals_bytes The total amount of global variable space that is scannable.
# TYPE go_gc_scan_globals_bytes gauge
go_gc_scan_globals_bytes 828608
# HELP go_gc_scan_heap_bytes The total amount of heap space that is scannable.
# TYPE go_gc_scan_heap_bytes gauge
go_gc_scan_heap_bytes 9.874336e+06
# HELP go_gc_scan_stack_bytes The number of bytes of stack that were scanned last GC cycle.
# TYPE go_gc_scan_stack_bytes gauge
go_gc_scan_stack_bytes 46448
# HELP go_gc_scan_total_bytes The total amount space that is scannable. Sum of all metrics in /gc/scan.
# TYPE go_gc_scan_total_bytes gauge
go_gc_scan_total_bytes 1.0749392e+07
# HELP go_gc_stack_starting_size_bytes The stack size of new goroutines.
# TYPE go_gc_stack_starting_size_bytes gauge
go_gc_stack_starting_size_bytes 2048
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 38
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.22.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 1.6189592e+07
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 4.0746784e+07
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.46971e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 65081
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 3.504768e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 1.6189592e+07
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 9.306112e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 1.916928e+07
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 99641
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 4.194304e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 2.8475392e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.7224787965593023e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 164722
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 2400
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 15600
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 225280
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 228480
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 2.183524e+07
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 668682
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 851968
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 851968
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 3.52146e+07
# HELP go_sched_gomaxprocs_threads The current runtime.GOMAXPROCS setting, or the number of operating system threads that can execute user-level Go code simultaneously.
# TYPE go_sched_gomaxprocs_threads gauge
go_sched_gomaxprocs_threads 2
# HELP go_sched_goroutines_goroutines Count of live goroutines.
# TYPE go_sched_goroutines_goroutines gauge
go_sched_goroutines_goroutines 38
# HELP go_sched_latencies_seconds Distribution of the time goroutines have spent in the scheduler in a runnable state before actually running. Bucket counts increase monotonically.
# TYPE go_sched_latencies_seconds histogram
go_sched_latencies_seconds_bucket{le="6.399999999999999e-08"} 112
go_sched_latencies_seconds_bucket{le="6.399999999999999e-07"} 293
go_sched_latencies_seconds_bucket{le="7.167999999999999e-06"} 318
go_sched_latencies_seconds_bucket{le="8.191999999999999e-05"} 337
go_sched_latencies_seconds_bucket{le="0.0009175039999999999"} 351
go_sched_latencies_seconds_bucket{le="0.010485759999999998"} 352
go_sched_latencies_seconds_bucket{le="0.11744051199999998"} 352
go_sched_latencies_seconds_bucket{le="+Inf"} 352
go_sched_latencies_seconds_sum 0.00222816
go_sched_latencies_seconds_count 352
# HELP go_sched_pauses_stopping_gc_seconds Distribution of individual GC-related stop-the-world stopping latencies. This is the time it takes from deciding to stop the world until all Ps are stopped. This is a subset of the total GC-related stop-the-world time (/sched/pauses/total/gc:seconds). During this time, some threads may be executing. Bucket counts increase monotonically.
# TYPE go_sched_pauses_stopping_gc_seconds histogram
go_sched_pauses_stopping_gc_seconds_bucket{le="6.399999999999999e-08"} 0
go_sched_pauses_stopping_gc_seconds_bucket{le="6.399999999999999e-07"} 5
go_sched_pauses_stopping_gc_seconds_bucket{le="7.167999999999999e-06"} 5
go_sched_pauses_stopping_gc_seconds_bucket{le="8.191999999999999e-05"} 10
go_sched_pauses_stopping_gc_seconds_bucket{le="0.0009175039999999999"} 11
go_sched_pauses_stopping_gc_seconds_bucket{le="0.010485759999999998"} 12
go_sched_pauses_stopping_gc_seconds_bucket{le="0.11744051199999998"} 12
go_sched_pauses_stopping_gc_seconds_bucket{le="+Inf"} 12
go_sched_pauses_stopping_gc_seconds_sum 0.001035584
go_sched_pauses_stopping_gc_seconds_count 12
# HELP go_sched_pauses_stopping_other_seconds Distribution of individual non-GC-related stop-the-world stopping latencies. This is the time it takes from deciding to stop the world until all Ps are stopped. This is a subset of the total non-GC-related stop-the-world time (/sched/pauses/total/other:seconds). During this time, some threads may be executing. Bucket counts increase monotonically.
# TYPE go_sched_pauses_stopping_other_seconds histogram
go_sched_pauses_stopping_other_seconds_bucket{le="6.399999999999999e-08"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="6.399999999999999e-07"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="7.167999999999999e-06"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="8.191999999999999e-05"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="0.0009175039999999999"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="0.010485759999999998"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="0.11744051199999998"} 0
go_sched_pauses_stopping_other_seconds_bucket{le="+Inf"} 0
go_sched_pauses_stopping_other_seconds_sum 0
go_sched_pauses_stopping_other_seconds_count 0
# HELP go_sched_pauses_total_gc_seconds Distribution of individual GC-related stop-the-world pause latencies. This is the time from deciding to stop the world until the world is started again. Some of this time is spent getting all threads to stop (this is measured directly in /sched/pauses/stopping/gc:seconds), during which some threads may still be running. Bucket counts increase monotonically.
# TYPE go_sched_pauses_total_gc_seconds histogram
go_sched_pauses_total_gc_seconds_bucket{le="6.399999999999999e-08"} 0
go_sched_pauses_total_gc_seconds_bucket{le="6.399999999999999e-07"} 0
go_sched_pauses_total_gc_seconds_bucket{le="7.167999999999999e-06"} 2
go_sched_pauses_total_gc_seconds_bucket{le="8.191999999999999e-05"} 10
go_sched_pauses_total_gc_seconds_bucket{le="0.0009175039999999999"} 11
go_sched_pauses_total_gc_seconds_bucket{le="0.010485759999999998"} 12
go_sched_pauses_total_gc_seconds_bucket{le="0.11744051199999998"} 12
go_sched_pauses_total_gc_seconds_bucket{le="+Inf"} 12
go_sched_pauses_total_gc_seconds_sum 0.001058048
go_sched_pauses_total_gc_seconds_count 12
# HELP go_sched_pauses_total_other_seconds Distribution of individual non-GC-related stop-the-world pause latencies. This is the time from deciding to stop the world until the world is started again. Some of this time is spent getting all threads to stop (measured directly in /sched/pauses/stopping/other:seconds). Bucket counts increase monotonically.
# TYPE go_sched_pauses_total_other_seconds histogram
go_sched_pauses_total_other_seconds_bucket{le="6.399999999999999e-08"} 0
go_sched_pauses_total_other_seconds_bucket{le="6.399999999999999e-07"} 0
go_sched_pauses_total_other_seconds_bucket{le="7.167999999999999e-06"} 0
go_sched_pauses_total_other_seconds_bucket{le="8.191999999999999e-05"} 0
go_sched_pauses_total_other_seconds_bucket{le="0.0009175039999999999"} 0
go_sched_pauses_total_other_seconds_bucket{le="0.010485759999999998"} 0
go_sched_pauses_total_other_seconds_bucket{le="0.11744051199999998"} 0
go_sched_pauses_total_other_seconds_bucket{le="+Inf"} 0
go_sched_pauses_total_other_seconds_sum 0
go_sched_pauses_total_other_seconds_count 0
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 8
# HELP net_conntrack_dialer_conn_attempted_total Total number of connections attempted by the given dialer a given name.
# TYPE net_conntrack_dialer_conn_attempted_total counter
net_conntrack_dialer_conn_attempted_total{dialer_name="alertmanager"} 0
net_conntrack_dialer_conn_attempted_total{dialer_name="default"} 0
net_conntrack_dialer_conn_attempted_total{dialer_name="prometheus"} 1
# HELP net_conntrack_dialer_conn_closed_total Total number of connections closed which originated from the dialer of a given name.
# TYPE net_conntrack_dialer_conn_closed_total counter
net_conntrack_dialer_conn_closed_total{dialer_name="alertmanager"} 0
net_conntrack_dialer_conn_closed_total{dialer_name="default"} 0
net_conntrack_dialer_conn_closed_total{dialer_name="prometheus"} 0
# HELP net_conntrack_dialer_conn_established_total Total number of connections successfully established by the given dialer a given name.
# TYPE net_conntrack_dialer_conn_established_total counter
net_conntrack_dialer_conn_established_total{dialer_name="alertmanager"} 0
net_conntrack_dialer_conn_established_total{dialer_name="default"} 0
net_conntrack_dialer_conn_established_total{dialer_name="prometheus"} 1
# HELP net_conntrack_dialer_conn_failed_total Total number of connections failed to dial by the dialer a given name.
# TYPE net_conntrack_dialer_conn_failed_total counter
net_conntrack_dialer_conn_failed_total{dialer_name="alertmanager",reason="refused"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="alertmanager",reason="resolution"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="alertmanager",reason="timeout"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="alertmanager",reason="unknown"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="refused"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="resolution"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="timeout"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="unknown"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="prometheus",reason="refused"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="prometheus",reason="resolution"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="prometheus",reason="timeout"} 0
net_conntrack_dialer_conn_failed_total{dialer_name="prometheus",reason="unknown"} 0
# HELP net_conntrack_listener_conn_accepted_total Total number of connections opened to the listener of a given name.
# TYPE net_conntrack_listener_conn_accepted_total counter
net_conntrack_listener_conn_accepted_total{listener_name="http"} 7
# HELP net_conntrack_listener_conn_closed_total Total number of connections closed that were made to the listener of a given name.
# TYPE net_conntrack_listener_conn_closed_total counter
net_conntrack_listener_conn_closed_total{listener_name="http"} 0
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 0.11
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 27
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 8.3267584e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.72247878169e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.358098432e+09
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# HELP prometheus_api_remote_read_queries The current number of remote read queries being executed or waiting.
# TYPE prometheus_api_remote_read_queries gauge
prometheus_api_remote_read_queries 0
# HELP prometheus_build_info A metric with a constant '1' value labeled by version, revision, branch, goversion from which prometheus was built, and the goos and goarch for the build.
# TYPE prometheus_build_info gauge
prometheus_build_info{branch="HEAD",goarch="amd64",goos="linux",goversion="go1.22.5",revision="2898d5d715738776bf8dd31d424a9b297380a2f3",tags="netgo,builtinassets,stringlabels",version="2.54.0-rc.0"} 1
# HELP prometheus_config_last_reload_success_timestamp_seconds Timestamp of the last successful configuration reload.
# TYPE prometheus_config_last_reload_success_timestamp_seconds gauge
prometheus_config_last_reload_success_timestamp_seconds 1.7224787823277137e+09
# HELP prometheus_config_last_reload_successful Whether the last configuration reload attempt was successful.
# TYPE prometheus_config_last_reload_successful gauge
prometheus_config_last_reload_successful 1
# HELP prometheus_engine_queries The current number of queries being executed or waiting.
# TYPE prometheus_engine_queries gauge
prometheus_engine_queries 0
# HELP prometheus_engine_queries_concurrent_max The max number of concurrent queries.
# TYPE prometheus_engine_queries_concurrent_max gauge
prometheus_engine_queries_concurrent_max 20
# HELP prometheus_engine_query_duration_seconds Query timings
# TYPE prometheus_engine_query_duration_seconds summary
prometheus_engine_query_duration_seconds{slice="inner_eval",quantile="0.5"} 6.432e-06
prometheus_engine_query_duration_seconds{slice="inner_eval",quantile="0.9"} 6.432e-06
prometheus_engine_query_duration_seconds{slice="inner_eval",quantile="0.99"} 6.432e-06
prometheus_engine_query_duration_seconds_sum{slice="inner_eval"} 6.432e-06
prometheus_engine_query_duration_seconds_count{slice="inner_eval"} 1
prometheus_engine_query_duration_seconds{slice="prepare_time",quantile="0.5"} 3.146e-06
prometheus_engine_query_duration_seconds{slice="prepare_time",quantile="0.9"} 3.146e-06
prometheus_engine_query_duration_seconds{slice="prepare_time",quantile="0.99"} 3.146e-06
prometheus_engine_query_duration_seconds_sum{slice="prepare_time"} 3.146e-06
prometheus_engine_query_duration_seconds_count{slice="prepare_time"} 1
prometheus_engine_query_duration_seconds{slice="queue_time",quantile="0.5"} 2.996e-06
prometheus_engine_query_duration_seconds{slice="queue_time",quantile="0.9"} 3.3673e-05
prometheus_engine_query_duration_seconds{slice="queue_time",quantile="0.99"} 3.3673e-05
prometheus_engine_query_duration_seconds_sum{slice="queue_time"} 3.6669000000000004e-05
prometheus_engine_query_duration_seconds_count{slice="queue_time"} 2
prometheus_engine_query_duration_seconds{slice="result_sort",quantile="0.5"} NaN
prometheus_engine_query_duration_seconds{slice="result_sort",quantile="0.9"} NaN
prometheus_engine_query_duration_seconds{slice="result_sort",quantile="0.99"} NaN
prometheus_engine_query_duration_seconds_sum{slice="result_sort"} 0
prometheus_engine_query_duration_seconds_count{slice="result_sort"} 0
# HELP prometheus_engine_query_log_enabled State of the query log.
# TYPE prometheus_engine_query_log_enabled gauge
prometheus_engine_query_log_enabled 0
# HELP prometheus_engine_query_log_failures_total The number of query log failures.
# TYPE prometheus_engine_query_log_failures_total counter
prometheus_engine_query_log_failures_total 0
# HELP prometheus_engine_query_samples_total The total number of samples loaded by all queries.
# TYPE prometheus_engine_query_samples_total counter
prometheus_engine_query_samples_total 0
# HELP prometheus_http_request_duration_seconds Histogram of latencies for HTTP requests.
# TYPE prometheus_http_request_duration_seconds histogram
prometheus_http_request_duration_seconds_bucket{handler="/",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/"} 3.4774e-05
prometheus_http_request_duration_seconds_count{handler="/"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/-/ready",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/-/ready"} 2.2291e-05
prometheus_http_request_duration_seconds_count{handler="/-/ready"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/label/:name/values",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/api/v1/label/:name/values"} 0.001248167
prometheus_http_request_duration_seconds_count{handler="/api/v1/label/:name/values"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/api/v1/query",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/api/v1/query"} 0.00046145
prometheus_http_request_duration_seconds_count{handler="/api/v1/query"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/favicon.ico",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/favicon.ico"} 0.000363688
prometheus_http_request_duration_seconds_count{handler="/favicon.ico"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/graph",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/graph"} 0.000177591
prometheus_http_request_duration_seconds_count{handler="/graph"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="0.1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="0.2"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="0.4"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="1"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="3"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="8"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="20"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="60"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="120"} 1
prometheus_http_request_duration_seconds_bucket{handler="/manifest.json",le="+Inf"} 1
prometheus_http_request_duration_seconds_sum{handler="/manifest.json"} 9.1821e-05
prometheus_http_request_duration_seconds_count{handler="/manifest.json"} 1
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.1"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.2"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="0.4"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="1"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="3"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="8"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="20"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="60"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="120"} 5
prometheus_http_request_duration_seconds_bucket{handler="/metrics",le="+Inf"} 5
prometheus_http_request_duration_seconds_sum{handler="/metrics"} 0.015117453
prometheus_http_request_duration_seconds_count{handler="/metrics"} 5
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="0.1"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="0.2"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="0.4"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="1"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="3"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="8"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="20"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="60"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="120"} 2
prometheus_http_request_duration_seconds_bucket{handler="/static/*filepath",le="+Inf"} 2
prometheus_http_request_duration_seconds_sum{handler="/static/*filepath"} 0.02561622
prometheus_http_request_duration_seconds_count{handler="/static/*filepath"} 2
# HELP prometheus_http_requests_total Counter of HTTP requests.
# TYPE prometheus_http_requests_total counter
prometheus_http_requests_total{code="200",handler="/"} 0
prometheus_http_requests_total{code="200",handler="/-/healthy"} 0
prometheus_http_requests_total{code="200",handler="/-/quit"} 0
prometheus_http_requests_total{code="200",handler="/-/ready"} 1
prometheus_http_requests_total{code="200",handler="/-/reload"} 0
prometheus_http_requests_total{code="200",handler="/alerts"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/*path"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/admin/tsdb/clean_tombstones"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/admin/tsdb/delete_series"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/admin/tsdb/snapshot"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/alertmanagers"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/alerts"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/format_query"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/label/:name/values"} 1
prometheus_http_requests_total{code="200",handler="/api/v1/labels"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/metadata"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/otlp/v1/metrics"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/query"} 1
prometheus_http_requests_total{code="200",handler="/api/v1/query_exemplars"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/query_range"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/read"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/rules"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/scrape_pools"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/series"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/buildinfo"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/config"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/flags"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/runtimeinfo"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/tsdb"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/status/walreplay"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/targets"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/targets/metadata"} 0
prometheus_http_requests_total{code="200",handler="/api/v1/write"} 0
prometheus_http_requests_total{code="200",handler="/classic/static/*filepath"} 0
prometheus_http_requests_total{code="200",handler="/config"} 0
prometheus_http_requests_total{code="200",handler="/consoles/*filepath"} 0
prometheus_http_requests_total{code="200",handler="/debug/*subpath"} 0
prometheus_http_requests_total{code="200",handler="/favicon.ico"} 1
prometheus_http_requests_total{code="200",handler="/federate"} 0
prometheus_http_requests_total{code="200",handler="/flags"} 0
prometheus_http_requests_total{code="200",handler="/graph"} 1
prometheus_http_requests_total{code="200",handler="/manifest.json"} 1
prometheus_http_requests_total{code="200",handler="/metrics"} 5
prometheus_http_requests_total{code="200",handler="/rules"} 0
prometheus_http_requests_total{code="200",handler="/service-discovery"} 0
prometheus_http_requests_total{code="200",handler="/starting"} 0
prometheus_http_requests_total{code="200",handler="/static/*filepath"} 2
prometheus_http_requests_total{code="200",handler="/status"} 0
prometheus_http_requests_total{code="200",handler="/targets"} 0
prometheus_http_requests_total{code="200",handler="/tsdb-status"} 0
prometheus_http_requests_total{code="200",handler="/version"} 0
prometheus_http_requests_total{code="302",handler="/"} 1
# HELP prometheus_http_response_size_bytes Histogram of response size for HTTP requests.
# TYPE prometheus_http_response_size_bytes histogram
prometheus_http_response_size_bytes_bucket{handler="/",le="100"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/"} 29
prometheus_http_response_size_bytes_count{handler="/"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="100"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/-/ready",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/-/ready"} 28
prometheus_http_response_size_bytes_count{handler="/-/ready"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="100"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/label/:name/values",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/api/v1/label/:name/values"} 60
prometheus_http_response_size_bytes_count{handler="/api/v1/label/:name/values"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/api/v1/query",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/api/v1/query"} 124
prometheus_http_response_size_bytes_count{handler="/api/v1/query"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="1000"} 0
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="10000"} 0
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/favicon.ico",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/favicon.ico"} 15086
prometheus_http_response_size_bytes_count{handler="/favicon.ico"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/graph",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/graph",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/graph"} 734
prometheus_http_response_size_bytes_count{handler="/graph"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="1000"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="10000"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="100000"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="1e+07"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="1e+08"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="1e+09"} 1
prometheus_http_response_size_bytes_bucket{handler="/manifest.json",le="+Inf"} 1
prometheus_http_response_size_bytes_sum{handler="/manifest.json"} 318
prometheus_http_response_size_bytes_count{handler="/manifest.json"} 1
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="1000"} 0
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="10000"} 0
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="100000"} 5
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="1e+06"} 5
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="1e+07"} 5
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="1e+08"} 5
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="1e+09"} 5
prometheus_http_response_size_bytes_bucket{handler="/metrics",le="+Inf"} 5
prometheus_http_response_size_bytes_sum{handler="/metrics"} 56395
prometheus_http_response_size_bytes_count{handler="/metrics"} 5
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="100"} 0
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="1000"} 0
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="10000"} 0
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="100000"} 0
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="1e+06"} 1
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="1e+07"} 2
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="1e+08"} 2
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="1e+09"} 2
prometheus_http_response_size_bytes_bucket{handler="/static/*filepath",le="+Inf"} 2
prometheus_http_response_size_bytes_sum{handler="/static/*filepath"} 2.834043e+06
prometheus_http_response_size_bytes_count{handler="/static/*filepath"} 2
# HELP prometheus_notifications_alertmanagers_discovered The number of alertmanagers discovered and active.
# TYPE prometheus_notifications_alertmanagers_discovered gauge
prometheus_notifications_alertmanagers_discovered 0
# HELP prometheus_notifications_dropped_total Total number of alerts dropped due to errors when sending to Alertmanager.
# TYPE prometheus_notifications_dropped_total counter
prometheus_notifications_dropped_total 0
# HELP prometheus_notifications_queue_capacity The capacity of the alert notifications queue.
# TYPE prometheus_notifications_queue_capacity gauge
prometheus_notifications_queue_capacity 10000
# HELP prometheus_notifications_queue_length The number of alert notifications in the queue.
# TYPE prometheus_notifications_queue_length gauge
prometheus_notifications_queue_length 0
# HELP prometheus_ready Whether Prometheus startup was fully completed and the server is ready for normal operation.
# TYPE prometheus_ready gauge
prometheus_ready 1
# HELP prometheus_remote_storage_exemplars_in_total Exemplars in to remote storage, compare to exemplars out for queue managers.
# TYPE prometheus_remote_storage_exemplars_in_total counter
prometheus_remote_storage_exemplars_in_total 0
# HELP prometheus_remote_storage_highest_timestamp_in_seconds Highest timestamp that has come into the remote storage via the Appender interface, in seconds since epoch. Initialized to 0 when no data has been received yet.
# TYPE prometheus_remote_storage_highest_timestamp_in_seconds gauge
prometheus_remote_storage_highest_timestamp_in_seconds 1.722478856e+09
# HELP prometheus_remote_storage_histograms_in_total HistogramSamples in to remote storage, compare to histograms out for queue managers.
# TYPE prometheus_remote_storage_histograms_in_total counter
prometheus_remote_storage_histograms_in_total 0
# HELP prometheus_remote_storage_samples_in_total Samples in to remote storage, compare to samples out for queue managers.
# TYPE prometheus_remote_storage_samples_in_total counter
prometheus_remote_storage_samples_in_total 3608
# HELP prometheus_remote_storage_string_interner_zero_reference_releases_total The number of times release has been called for strings that are not interned.
# TYPE prometheus_remote_storage_string_interner_zero_reference_releases_total counter
prometheus_remote_storage_string_interner_zero_reference_releases_total 0
# HELP prometheus_rule_evaluation_duration_seconds The duration for a rule to execute.
# TYPE prometheus_rule_evaluation_duration_seconds summary
prometheus_rule_evaluation_duration_seconds{quantile="0.5"} NaN
prometheus_rule_evaluation_duration_seconds{quantile="0.9"} NaN
prometheus_rule_evaluation_duration_seconds{quantile="0.99"} NaN
prometheus_rule_evaluation_duration_seconds_sum 0
prometheus_rule_evaluation_duration_seconds_count 0
# HELP prometheus_rule_group_duration_seconds The duration of rule group evaluations.
# TYPE prometheus_rule_group_duration_seconds summary
prometheus_rule_group_duration_seconds{quantile="0.01"} NaN
prometheus_rule_group_duration_seconds{quantile="0.05"} NaN
prometheus_rule_group_duration_seconds{quantile="0.5"} NaN
prometheus_rule_group_duration_seconds{quantile="0.9"} NaN
prometheus_rule_group_duration_seconds{quantile="0.99"} NaN
prometheus_rule_group_duration_seconds_sum 0
prometheus_rule_group_duration_seconds_count 0
# HELP prometheus_sd_azure_cache_hit_total Number of cache hit during refresh.
# TYPE prometheus_sd_azure_cache_hit_total counter
prometheus_sd_azure_cache_hit_total 0
# HELP prometheus_sd_azure_failures_total Number of Azure service discovery refresh failures.
# TYPE prometheus_sd_azure_failures_total counter
prometheus_sd_azure_failures_total 0
# HELP prometheus_sd_consul_rpc_duration_seconds The duration of a Consul RPC call in seconds.
# TYPE prometheus_sd_consul_rpc_duration_seconds summary
prometheus_sd_consul_rpc_duration_seconds{call="service",endpoint="catalog",quantile="0.5"} NaN
prometheus_sd_consul_rpc_duration_seconds{call="service",endpoint="catalog",quantile="0.9"} NaN
prometheus_sd_consul_rpc_duration_seconds{call="service",endpoint="catalog",quantile="0.99"} NaN
prometheus_sd_consul_rpc_duration_seconds_sum{call="service",endpoint="catalog"} 0
prometheus_sd_consul_rpc_duration_seconds_count{call="service",endpoint="catalog"} 0
prometheus_sd_consul_rpc_duration_seconds{call="services",endpoint="catalog",quantile="0.5"} NaN
prometheus_sd_consul_rpc_duration_seconds{call="services",endpoint="catalog",quantile="0.9"} NaN
prometheus_sd_consul_rpc_duration_seconds{call="services",endpoint="catalog",quantile="0.99"} NaN
prometheus_sd_consul_rpc_duration_seconds_sum{call="services",endpoint="catalog"} 0
prometheus_sd_consul_rpc_duration_seconds_count{call="services",endpoint="catalog"} 0
# HELP prometheus_sd_consul_rpc_failures_total The number of Consul RPC call failures.
# TYPE prometheus_sd_consul_rpc_failures_total counter
prometheus_sd_consul_rpc_failures_total 0
# HELP prometheus_sd_discovered_targets Current number of discovered targets.
# TYPE prometheus_sd_discovered_targets gauge
prometheus_sd_discovered_targets{config="config-0",name="notify"} 0
prometheus_sd_discovered_targets{config="prometheus",name="scrape"} 1
# HELP prometheus_sd_dns_lookup_failures_total The number of DNS-SD lookup failures.
# TYPE prometheus_sd_dns_lookup_failures_total counter
prometheus_sd_dns_lookup_failures_total 0
# HELP prometheus_sd_dns_lookups_total The number of DNS-SD lookups.
# TYPE prometheus_sd_dns_lookups_total counter
prometheus_sd_dns_lookups_total 0
# HELP prometheus_sd_failed_configs Current number of service discovery configurations that failed to load.
# TYPE prometheus_sd_failed_configs gauge
prometheus_sd_failed_configs{name="notify"} 0
prometheus_sd_failed_configs{name="scrape"} 0
# HELP prometheus_sd_file_read_errors_total The number of File-SD read errors.
# TYPE prometheus_sd_file_read_errors_total counter
prometheus_sd_file_read_errors_total 0
# HELP prometheus_sd_file_scan_duration_seconds The duration of the File-SD scan in seconds.
# TYPE prometheus_sd_file_scan_duration_seconds summary
prometheus_sd_file_scan_duration_seconds{quantile="0.5"} NaN
prometheus_sd_file_scan_duration_seconds{quantile="0.9"} NaN
prometheus_sd_file_scan_duration_seconds{quantile="0.99"} NaN
prometheus_sd_file_scan_duration_seconds_sum 0
prometheus_sd_file_scan_duration_seconds_count 0
# HELP prometheus_sd_file_watcher_errors_total The number of File-SD errors caused by filesystem watch failures.
# TYPE prometheus_sd_file_watcher_errors_total counter
prometheus_sd_file_watcher_errors_total 0
# HELP prometheus_sd_http_failures_total Number of HTTP service discovery refresh failures.
# TYPE prometheus_sd_http_failures_total counter
prometheus_sd_http_failures_total 0
# HELP prometheus_sd_kubernetes_events_total The number of Kubernetes events handled.
# TYPE prometheus_sd_kubernetes_events_total counter
prometheus_sd_kubernetes_events_total{event="add",role="endpoints"} 0
prometheus_sd_kubernetes_events_total{event="add",role="endpointslice"} 0
prometheus_sd_kubernetes_events_total{event="add",role="ingress"} 0
prometheus_sd_kubernetes_events_total{event="add",role="node"} 0
prometheus_sd_kubernetes_events_total{event="add",role="pod"} 0
prometheus_sd_kubernetes_events_total{event="add",role="service"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="endpoints"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="endpointslice"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="ingress"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="node"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="pod"} 0
prometheus_sd_kubernetes_events_total{event="delete",role="service"} 0
prometheus_sd_kubernetes_events_total{event="update",role="endpoints"} 0
prometheus_sd_kubernetes_events_total{event="update",role="endpointslice"} 0
prometheus_sd_kubernetes_events_total{event="update",role="ingress"} 0
prometheus_sd_kubernetes_events_total{event="update",role="node"} 0
prometheus_sd_kubernetes_events_total{event="update",role="pod"} 0
prometheus_sd_kubernetes_events_total{event="update",role="service"} 0
# HELP prometheus_sd_kubernetes_failures_total The number of failed WATCH/LIST requests.
# TYPE prometheus_sd_kubernetes_failures_total counter
prometheus_sd_kubernetes_failures_total 0
# HELP prometheus_sd_kuma_fetch_duration_seconds The duration of a Kuma MADS fetch call.
# TYPE prometheus_sd_kuma_fetch_duration_seconds summary
prometheus_sd_kuma_fetch_duration_seconds{quantile="0.5"} NaN
prometheus_sd_kuma_fetch_duration_seconds{quantile="0.9"} NaN
prometheus_sd_kuma_fetch_duration_seconds{quantile="0.99"} NaN
prometheus_sd_kuma_fetch_duration_seconds_sum 0
prometheus_sd_kuma_fetch_duration_seconds_count 0
# HELP prometheus_sd_kuma_fetch_failures_total The number of Kuma MADS fetch call failures.
# TYPE prometheus_sd_kuma_fetch_failures_total counter
prometheus_sd_kuma_fetch_failures_total 0
# HELP prometheus_sd_kuma_fetch_skipped_updates_total The number of Kuma MADS fetch calls that result in no updates to the targets.
# TYPE prometheus_sd_kuma_fetch_skipped_updates_total counter
prometheus_sd_kuma_fetch_skipped_updates_total 0
# HELP prometheus_sd_linode_failures_total Number of Linode service discovery refresh failures.
# TYPE prometheus_sd_linode_failures_total counter
prometheus_sd_linode_failures_total 0
# HELP prometheus_sd_nomad_failures_total Number of nomad service discovery refresh failures.
# TYPE prometheus_sd_nomad_failures_total counter
prometheus_sd_nomad_failures_total 0
# HELP prometheus_sd_received_updates_total Total number of update events received from the SD providers.
# TYPE prometheus_sd_received_updates_total counter
prometheus_sd_received_updates_total{name="notify"} 2
prometheus_sd_received_updates_total{name="scrape"} 2
# HELP prometheus_sd_updates_delayed_total Total number of update events that couldn't be sent immediately.
# TYPE prometheus_sd_updates_delayed_total counter
prometheus_sd_updates_delayed_total{name="notify"} 0
prometheus_sd_updates_delayed_total{name="scrape"} 0
# HELP prometheus_sd_updates_total Total number of update events sent to the SD consumers.
# TYPE prometheus_sd_updates_total counter
prometheus_sd_updates_total{name="notify"} 1
prometheus_sd_updates_total{name="scrape"} 1
# HELP prometheus_target_interval_length_seconds Actual intervals between scrapes.
# TYPE prometheus_target_interval_length_seconds summary
prometheus_target_interval_length_seconds{interval="15s",quantile="0.01"} 14.99999796
prometheus_target_interval_length_seconds{interval="15s",quantile="0.05"} 14.99999796
prometheus_target_interval_length_seconds{interval="15s",quantile="0.5"} 15.0001805
prometheus_target_interval_length_seconds{interval="15s",quantile="0.9"} 15.002564
prometheus_target_interval_length_seconds{interval="15s",quantile="0.99"} 15.002564
prometheus_target_interval_length_seconds_sum{interval="15s"} 60.003668463
prometheus_target_interval_length_seconds_count{interval="15s"} 4
# HELP prometheus_target_metadata_cache_bytes The number of bytes that are currently used for storing metric metadata in the cache
# TYPE prometheus_target_metadata_cache_bytes gauge
prometheus_target_metadata_cache_bytes{scrape_job="prometheus"} 15966
# HELP prometheus_target_metadata_cache_entries Total number of metric metadata entries in the cache
# TYPE prometheus_target_metadata_cache_entries gauge
prometheus_target_metadata_cache_entries{scrape_job="prometheus"} 217
# HELP prometheus_target_scrape_pool_exceeded_label_limits_total Total number of times scrape pools hit the label limits, during sync or config reload.
# TYPE prometheus_target_scrape_pool_exceeded_label_limits_total counter
prometheus_target_scrape_pool_exceeded_label_limits_total 0
# HELP prometheus_target_scrape_pool_exceeded_target_limit_total Total number of times scrape pools hit the target limit, during sync or config reload.
# TYPE prometheus_target_scrape_pool_exceeded_target_limit_total counter
prometheus_target_scrape_pool_exceeded_target_limit_total 0
# HELP prometheus_target_scrape_pool_reloads_failed_total Total number of failed scrape pool reloads.
# TYPE prometheus_target_scrape_pool_reloads_failed_total counter
prometheus_target_scrape_pool_reloads_failed_total 0
# HELP prometheus_target_scrape_pool_reloads_total Total number of scrape pool reloads.
# TYPE prometheus_target_scrape_pool_reloads_total counter
prometheus_target_scrape_pool_reloads_total 0
# HELP prometheus_target_scrape_pool_symboltable_items Current number of symbols in table for this scrape pool.
# TYPE prometheus_target_scrape_pool_symboltable_items gauge
prometheus_target_scrape_pool_symboltable_items{scrape_job="prometheus"} 0
# HELP prometheus_target_scrape_pool_sync_total Total number of syncs that were executed on a scrape pool.
# TYPE prometheus_target_scrape_pool_sync_total counter
prometheus_target_scrape_pool_sync_total{scrape_job="prometheus"} 1
# HELP prometheus_target_scrape_pool_target_limit Maximum number of targets allowed in this scrape pool.
# TYPE prometheus_target_scrape_pool_target_limit gauge
prometheus_target_scrape_pool_target_limit{scrape_job="prometheus"} 0
# HELP prometheus_target_scrape_pool_targets Current number of targets in this scrape pool.
# TYPE prometheus_target_scrape_pool_targets gauge
prometheus_target_scrape_pool_targets{scrape_job="prometheus"} 1
# HELP prometheus_target_scrape_pools_failed_total Total number of scrape pool creations that failed.
# TYPE prometheus_target_scrape_pools_failed_total counter
prometheus_target_scrape_pools_failed_total 0
# HELP prometheus_target_scrape_pools_total Total number of scrape pool creation attempts.
# TYPE prometheus_target_scrape_pools_total counter
prometheus_target_scrape_pools_total 1
# HELP prometheus_target_scrapes_cache_flush_forced_total How many times a scrape cache was flushed due to getting big while scrapes are failing.
# TYPE prometheus_target_scrapes_cache_flush_forced_total counter
prometheus_target_scrapes_cache_flush_forced_total 0
# HELP prometheus_target_scrapes_exceeded_body_size_limit_total Total number of scrapes that hit the body size limit
# TYPE prometheus_target_scrapes_exceeded_body_size_limit_total counter
prometheus_target_scrapes_exceeded_body_size_limit_total 0
# HELP prometheus_target_scrapes_exceeded_native_histogram_bucket_limit_total Total number of scrapes that hit the native histogram bucket limit and were rejected.
# TYPE prometheus_target_scrapes_exceeded_native_histogram_bucket_limit_total counter
prometheus_target_scrapes_exceeded_native_histogram_bucket_limit_total 0
# HELP prometheus_target_scrapes_exceeded_sample_limit_total Total number of scrapes that hit the sample limit and were rejected.
# TYPE prometheus_target_scrapes_exceeded_sample_limit_total counter
prometheus_target_scrapes_exceeded_sample_limit_total 0
# HELP prometheus_target_scrapes_exemplar_out_of_order_total Total number of exemplar rejected due to not being out of the expected order.
# TYPE prometheus_target_scrapes_exemplar_out_of_order_total counter
prometheus_target_scrapes_exemplar_out_of_order_total 0
# HELP prometheus_target_scrapes_sample_duplicate_timestamp_total Total number of samples rejected due to duplicate timestamps but different values.
# TYPE prometheus_target_scrapes_sample_duplicate_timestamp_total counter
prometheus_target_scrapes_sample_duplicate_timestamp_total 0
# HELP prometheus_target_scrapes_sample_out_of_bounds_total Total number of samples rejected due to timestamp falling outside of the time bounds.
# TYPE prometheus_target_scrapes_sample_out_of_bounds_total counter
prometheus_target_scrapes_sample_out_of_bounds_total 0
# HELP prometheus_target_scrapes_sample_out_of_order_total Total number of samples rejected due to not being out of the expected order.
# TYPE prometheus_target_scrapes_sample_out_of_order_total counter
prometheus_target_scrapes_sample_out_of_order_total 0
# HELP prometheus_target_sync_failed_total Total number of target sync failures.
# TYPE prometheus_target_sync_failed_total counter
prometheus_target_sync_failed_total{scrape_job="prometheus"} 0
# HELP prometheus_target_sync_length_seconds Actual interval to sync the scrape pool.
# TYPE prometheus_target_sync_length_seconds summary
prometheus_target_sync_length_seconds{scrape_job="prometheus",quantile="0.01"} 9.4886e-05
prometheus_target_sync_length_seconds{scrape_job="prometheus",quantile="0.05"} 9.4886e-05
prometheus_target_sync_length_seconds{scrape_job="prometheus",quantile="0.5"} 9.4886e-05
prometheus_target_sync_length_seconds{scrape_job="prometheus",quantile="0.9"} 9.4886e-05
prometheus_target_sync_length_seconds{scrape_job="prometheus",quantile="0.99"} 9.4886e-05
prometheus_target_sync_length_seconds_sum{scrape_job="prometheus"} 9.4886e-05
prometheus_target_sync_length_seconds_count{scrape_job="prometheus"} 1
# HELP prometheus_template_text_expansion_failures_total The total number of template text expansion failures.
# TYPE prometheus_template_text_expansion_failures_total counter
prometheus_template_text_expansion_failures_total 0
# HELP prometheus_template_text_expansions_total The total number of template text expansions.
# TYPE prometheus_template_text_expansions_total counter
prometheus_template_text_expansions_total 0
# HELP prometheus_treecache_watcher_goroutines The current number of watcher goroutines.
# TYPE prometheus_treecache_watcher_goroutines gauge
prometheus_treecache_watcher_goroutines 0
# HELP prometheus_treecache_zookeeper_failures_total The total number of ZooKeeper failures.
# TYPE prometheus_treecache_zookeeper_failures_total counter
prometheus_treecache_zookeeper_failures_total 0
# HELP prometheus_tsdb_blocks_loaded Number of currently loaded data blocks
# TYPE prometheus_tsdb_blocks_loaded gauge
prometheus_tsdb_blocks_loaded 0
# HELP prometheus_tsdb_checkpoint_creations_failed_total Total number of checkpoint creations that failed.
# TYPE prometheus_tsdb_checkpoint_creations_failed_total counter
prometheus_tsdb_checkpoint_creations_failed_total 0
# HELP prometheus_tsdb_checkpoint_creations_total Total number of checkpoint creations attempted.
# TYPE prometheus_tsdb_checkpoint_creations_total counter
prometheus_tsdb_checkpoint_creations_total 0
# HELP prometheus_tsdb_checkpoint_deletions_failed_total Total number of checkpoint deletions that failed.
# TYPE prometheus_tsdb_checkpoint_deletions_failed_total counter
prometheus_tsdb_checkpoint_deletions_failed_total 0
# HELP prometheus_tsdb_checkpoint_deletions_total Total number of checkpoint deletions attempted.
# TYPE prometheus_tsdb_checkpoint_deletions_total counter
prometheus_tsdb_checkpoint_deletions_total 0
# HELP prometheus_tsdb_clean_start -1: lockfile is disabled. 0: a lockfile from a previous execution was replaced. 1: lockfile creation was clean
# TYPE prometheus_tsdb_clean_start gauge
prometheus_tsdb_clean_start 1
# HELP prometheus_tsdb_compaction_chunk_range_seconds Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range_seconds histogram
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="100"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="400"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1600"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6400"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="25600"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="102400"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="409600"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1.6384e+06"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6.5536e+06"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="2.62144e+07"} 0
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="+Inf"} 0
prometheus_tsdb_compaction_chunk_range_seconds_sum 0
prometheus_tsdb_compaction_chunk_range_seconds_count 0
# HELP prometheus_tsdb_compaction_chunk_samples Final number of samples on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_samples histogram
prometheus_tsdb_compaction_chunk_samples_bucket{le="4"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="6"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="9"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="13.5"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="20.25"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="30.375"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="45.5625"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="68.34375"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="102.515625"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="153.7734375"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="230.66015625"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="345.990234375"} 0
prometheus_tsdb_compaction_chunk_samples_bucket{le="+Inf"} 0
prometheus_tsdb_compaction_chunk_samples_sum 0
prometheus_tsdb_compaction_chunk_samples_count 0
# HELP prometheus_tsdb_compaction_chunk_size_bytes Final size of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_size_bytes histogram
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="32"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="48"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="72"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="108"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="162"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="243"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="364.5"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="546.75"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="820.125"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="1230.1875"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="1845.28125"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="2767.921875"} 0
prometheus_tsdb_compaction_chunk_size_bytes_bucket{le="+Inf"} 0
prometheus_tsdb_compaction_chunk_size_bytes_sum 0
prometheus_tsdb_compaction_chunk_size_bytes_count 0
# HELP prometheus_tsdb_compaction_duration_seconds Duration of compaction runs
# TYPE prometheus_tsdb_compaction_duration_seconds histogram
prometheus_tsdb_compaction_duration_seconds_bucket{le="1"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="2"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="4"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="8"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="16"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="32"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="64"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="128"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="256"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="512"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="1024"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="2048"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="4096"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="8192"} 0
prometheus_tsdb_compaction_duration_seconds_bucket{le="+Inf"} 0
prometheus_tsdb_compaction_duration_seconds_sum 0
prometheus_tsdb_compaction_duration_seconds_count 0
# HELP prometheus_tsdb_compaction_populating_block Set to 1 when a block is currently being written to the disk.
# TYPE prometheus_tsdb_compaction_populating_block gauge
prometheus_tsdb_compaction_populating_block 0
# HELP prometheus_tsdb_compactions_failed_total Total number of compactions that failed for the partition.
# TYPE prometheus_tsdb_compactions_failed_total counter
prometheus_tsdb_compactions_failed_total 0
# HELP prometheus_tsdb_compactions_skipped_total Total number of skipped compactions due to disabled auto compaction.
# TYPE prometheus_tsdb_compactions_skipped_total counter
prometheus_tsdb_compactions_skipped_total 0
# HELP prometheus_tsdb_compactions_total Total number of compactions that were executed for the partition.
# TYPE prometheus_tsdb_compactions_total counter
prometheus_tsdb_compactions_total 0
# HELP prometheus_tsdb_compactions_triggered_total Total number of triggered compactions for the partition.
# TYPE prometheus_tsdb_compactions_triggered_total counter
prometheus_tsdb_compactions_triggered_total 1
# HELP prometheus_tsdb_data_replay_duration_seconds Time taken to replay the data on disk.
# TYPE prometheus_tsdb_data_replay_duration_seconds gauge
prometheus_tsdb_data_replay_duration_seconds 0.000302504
# HELP prometheus_tsdb_exemplar_exemplars_appended_total Total number of appended exemplars.
# TYPE prometheus_tsdb_exemplar_exemplars_appended_total counter
prometheus_tsdb_exemplar_exemplars_appended_total 0
# HELP prometheus_tsdb_exemplar_exemplars_in_storage Number of exemplars currently in circular storage.
# TYPE prometheus_tsdb_exemplar_exemplars_in_storage gauge
prometheus_tsdb_exemplar_exemplars_in_storage 0
# HELP prometheus_tsdb_exemplar_last_exemplars_timestamp_seconds The timestamp of the oldest exemplar stored in circular storage. Useful to check for what timerange the current exemplar buffer limit allows. This usually means the last timestampfor all exemplars for a typical setup. This is not true though if one of the series timestamp is in future compared to rest series.
# TYPE prometheus_tsdb_exemplar_last_exemplars_timestamp_seconds gauge
prometheus_tsdb_exemplar_last_exemplars_timestamp_seconds 0
# HELP prometheus_tsdb_exemplar_max_exemplars Total number of exemplars the exemplar storage can store, resizeable.
# TYPE prometheus_tsdb_exemplar_max_exemplars gauge
prometheus_tsdb_exemplar_max_exemplars 0
# HELP prometheus_tsdb_exemplar_out_of_order_exemplars_total Total number of out of order exemplar ingestion failed attempts.
# TYPE prometheus_tsdb_exemplar_out_of_order_exemplars_total counter
prometheus_tsdb_exemplar_out_of_order_exemplars_total 0
# HELP prometheus_tsdb_exemplar_series_with_exemplars_in_storage Number of series with exemplars currently in circular storage.
# TYPE prometheus_tsdb_exemplar_series_with_exemplars_in_storage gauge
prometheus_tsdb_exemplar_series_with_exemplars_in_storage 0
# HELP prometheus_tsdb_head_active_appenders Number of currently active appender transactions
# TYPE prometheus_tsdb_head_active_appenders gauge
prometheus_tsdb_head_active_appenders 0
# HELP prometheus_tsdb_head_chunks Total number of chunks in the head block.
# TYPE prometheus_tsdb_head_chunks gauge
prometheus_tsdb_head_chunks 729
# HELP prometheus_tsdb_head_chunks_created_total Total number of chunks created in the head
# TYPE prometheus_tsdb_head_chunks_created_total counter
prometheus_tsdb_head_chunks_created_total 729
# HELP prometheus_tsdb_head_chunks_removed_total Total number of chunks removed in the head
# TYPE prometheus_tsdb_head_chunks_removed_total counter
prometheus_tsdb_head_chunks_removed_total 0
# HELP prometheus_tsdb_head_chunks_storage_size_bytes Size of the chunks_head directory.
# TYPE prometheus_tsdb_head_chunks_storage_size_bytes gauge
prometheus_tsdb_head_chunks_storage_size_bytes 0
# HELP prometheus_tsdb_head_gc_duration_seconds Runtime of garbage collection in the head block.
# TYPE prometheus_tsdb_head_gc_duration_seconds summary
prometheus_tsdb_head_gc_duration_seconds_sum 0
prometheus_tsdb_head_gc_duration_seconds_count 0
# HELP prometheus_tsdb_head_max_time Maximum timestamp of the head block. The unit is decided by the library consumer.
# TYPE prometheus_tsdb_head_max_time gauge
prometheus_tsdb_head_max_time 1.722478856555e+12
# HELP prometheus_tsdb_head_max_time_seconds Maximum timestamp of the head block.
# TYPE prometheus_tsdb_head_max_time_seconds gauge
prometheus_tsdb_head_max_time_seconds 1.722478856e+09
# HELP prometheus_tsdb_head_min_time Minimum time bound of the head block. The unit is decided by the library consumer.
# TYPE prometheus_tsdb_head_min_time gauge
prometheus_tsdb_head_min_time 1.722478796552e+12
# HELP prometheus_tsdb_head_min_time_seconds Minimum time bound of the head block.
# TYPE prometheus_tsdb_head_min_time_seconds gauge
prometheus_tsdb_head_min_time_seconds 1.722478796e+09
# HELP prometheus_tsdb_head_out_of_order_samples_appended_total Total number of appended out of order samples.
# TYPE prometheus_tsdb_head_out_of_order_samples_appended_total counter
prometheus_tsdb_head_out_of_order_samples_appended_total{type="float"} 0
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total{type="float"} 3608
prometheus_tsdb_head_samples_appended_total{type="histogram"} 0
# HELP prometheus_tsdb_head_series Total number of series in the head block.
# TYPE prometheus_tsdb_head_series gauge
prometheus_tsdb_head_series 729
# HELP prometheus_tsdb_head_series_created_total Total number of series created in the head
# TYPE prometheus_tsdb_head_series_created_total counter
prometheus_tsdb_head_series_created_total 729
# HELP prometheus_tsdb_head_series_not_found_total Total number of requests for series that were not found.
# TYPE prometheus_tsdb_head_series_not_found_total counter
prometheus_tsdb_head_series_not_found_total 0
# HELP prometheus_tsdb_head_series_removed_total Total number of series removed in the head
# TYPE prometheus_tsdb_head_series_removed_total counter
prometheus_tsdb_head_series_removed_total 0
# HELP prometheus_tsdb_head_truncations_failed_total Total number of head truncations that failed.
# TYPE prometheus_tsdb_head_truncations_failed_total counter
prometheus_tsdb_head_truncations_failed_total 0
# HELP prometheus_tsdb_head_truncations_total Total number of head truncations attempted.
# TYPE prometheus_tsdb_head_truncations_total counter
prometheus_tsdb_head_truncations_total 0
# HELP prometheus_tsdb_isolation_high_watermark The highest TSDB append ID that has been given out.
# TYPE prometheus_tsdb_isolation_high_watermark gauge
prometheus_tsdb_isolation_high_watermark 5
# HELP prometheus_tsdb_isolation_low_watermark The lowest TSDB append ID that is still referenced.
# TYPE prometheus_tsdb_isolation_low_watermark gauge
prometheus_tsdb_isolation_low_watermark 5
# HELP prometheus_tsdb_lowest_timestamp Lowest timestamp value stored in the database. The unit is decided by the library consumer.
# TYPE prometheus_tsdb_lowest_timestamp gauge
prometheus_tsdb_lowest_timestamp 1.722478796552e+12
# HELP prometheus_tsdb_lowest_timestamp_seconds Lowest timestamp value stored in the database.
# TYPE prometheus_tsdb_lowest_timestamp_seconds gauge
prometheus_tsdb_lowest_timestamp_seconds 1.722478796e+09
# HELP prometheus_tsdb_mmap_chunk_corruptions_total Total number of memory-mapped chunk corruptions.
# TYPE prometheus_tsdb_mmap_chunk_corruptions_total counter
prometheus_tsdb_mmap_chunk_corruptions_total 0
# HELP prometheus_tsdb_mmap_chunks_total Total number of chunks that were memory-mapped.
# TYPE prometheus_tsdb_mmap_chunks_total counter
prometheus_tsdb_mmap_chunks_total 0
# HELP prometheus_tsdb_out_of_bound_samples_total Total number of out of bound samples ingestion failed attempts with out of order support disabled.
# TYPE prometheus_tsdb_out_of_bound_samples_total counter
prometheus_tsdb_out_of_bound_samples_total{type="float"} 0
# HELP prometheus_tsdb_out_of_order_samples_total Total number of out of order samples ingestion failed attempts due to out of order being disabled.
# TYPE prometheus_tsdb_out_of_order_samples_total counter
prometheus_tsdb_out_of_order_samples_total{type="float"} 0
prometheus_tsdb_out_of_order_samples_total{type="histogram"} 0
# HELP prometheus_tsdb_reloads_failures_total Number of times the database failed to reloadBlocks block data from disk.
# TYPE prometheus_tsdb_reloads_failures_total counter
prometheus_tsdb_reloads_failures_total 0
# HELP prometheus_tsdb_reloads_total Number of times the database reloaded block data from disk.
# TYPE prometheus_tsdb_reloads_total counter
prometheus_tsdb_reloads_total 2
# HELP prometheus_tsdb_retention_limit_bytes Max number of bytes to be retained in the tsdb blocks, configured 0 means disabled
# TYPE prometheus_tsdb_retention_limit_bytes gauge
prometheus_tsdb_retention_limit_bytes 0
# HELP prometheus_tsdb_retention_limit_seconds How long to retain samples in storage.
# TYPE prometheus_tsdb_retention_limit_seconds gauge
prometheus_tsdb_retention_limit_seconds 1.296e+06
# HELP prometheus_tsdb_size_retentions_total The number of times that blocks were deleted because the maximum number of bytes was exceeded.
# TYPE prometheus_tsdb_size_retentions_total counter
prometheus_tsdb_size_retentions_total 0
# HELP prometheus_tsdb_snapshot_replay_error_total Total number snapshot replays that failed.
# TYPE prometheus_tsdb_snapshot_replay_error_total counter
prometheus_tsdb_snapshot_replay_error_total 0
# HELP prometheus_tsdb_storage_blocks_bytes The number of bytes that are currently used for local storage by all blocks.
# TYPE prometheus_tsdb_storage_blocks_bytes gauge
prometheus_tsdb_storage_blocks_bytes 0
# HELP prometheus_tsdb_symbol_table_size_bytes Size of symbol table in memory for loaded blocks
# TYPE prometheus_tsdb_symbol_table_size_bytes gauge
prometheus_tsdb_symbol_table_size_bytes 0
# HELP prometheus_tsdb_time_retentions_total The number of times that blocks were deleted because the maximum time limit was exceeded.
# TYPE prometheus_tsdb_time_retentions_total counter
prometheus_tsdb_time_retentions_total 0
# HELP prometheus_tsdb_tombstone_cleanup_seconds The time taken to recompact blocks to remove tombstones.
# TYPE prometheus_tsdb_tombstone_cleanup_seconds histogram
prometheus_tsdb_tombstone_cleanup_seconds_bucket{le="+Inf"} 0
prometheus_tsdb_tombstone_cleanup_seconds_sum 0
prometheus_tsdb_tombstone_cleanup_seconds_count 0
# HELP prometheus_tsdb_too_old_samples_total Total number of out of order samples ingestion failed attempts with out of support enabled, but sample outside of time window.
# TYPE prometheus_tsdb_too_old_samples_total counter
prometheus_tsdb_too_old_samples_total{type="float"} 0
# HELP prometheus_tsdb_vertical_compactions_total Total number of compactions done on overlapping blocks.
# TYPE prometheus_tsdb_vertical_compactions_total counter
prometheus_tsdb_vertical_compactions_total 0
# HELP prometheus_tsdb_wal_completed_pages_total Total number of completed pages.
# TYPE prometheus_tsdb_wal_completed_pages_total counter
prometheus_tsdb_wal_completed_pages_total 0
# HELP prometheus_tsdb_wal_corruptions_total Total number of WAL corruptions.
# TYPE prometheus_tsdb_wal_corruptions_total counter
prometheus_tsdb_wal_corruptions_total 0
# HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of write log fsync.
# TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} NaN
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} NaN
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} NaN
prometheus_tsdb_wal_fsync_duration_seconds_sum 0
prometheus_tsdb_wal_fsync_duration_seconds_count 0
# HELP prometheus_tsdb_wal_page_flushes_total Total number of page flushes.
# TYPE prometheus_tsdb_wal_page_flushes_total counter
prometheus_tsdb_wal_page_flushes_total 7
# HELP prometheus_tsdb_wal_segment_current Write log segment index that TSDB is currently writing to.
# TYPE prometheus_tsdb_wal_segment_current gauge
prometheus_tsdb_wal_segment_current 0
# HELP prometheus_tsdb_wal_storage_size_bytes Size of the write log directory.
# TYPE prometheus_tsdb_wal_storage_size_bytes gauge
prometheus_tsdb_wal_storage_size_bytes 30646
# HELP prometheus_tsdb_wal_truncate_duration_seconds Duration of WAL truncation.
# TYPE prometheus_tsdb_wal_truncate_duration_seconds summary
prometheus_tsdb_wal_truncate_duration_seconds_sum 0
prometheus_tsdb_wal_truncate_duration_seconds_count 0
# HELP prometheus_tsdb_wal_truncations_failed_total Total number of write log truncations that failed.
# TYPE prometheus_tsdb_wal_truncations_failed_total counter
prometheus_tsdb_wal_truncations_failed_total 0
# HELP prometheus_tsdb_wal_truncations_total Total number of write log truncations attempted.
# TYPE prometheus_tsdb_wal_truncations_total counter
prometheus_tsdb_wal_truncations_total 0
# HELP prometheus_tsdb_wal_writes_failed_total Total number of write log writes that failed.
# TYPE prometheus_tsdb_wal_writes_failed_total counter
prometheus_tsdb_wal_writes_failed_total 0
# HELP prometheus_web_federation_errors_total Total number of errors that occurred while sending federation responses.
# TYPE prometheus_web_federation_errors_total counter
prometheus_web_federation_errors_total 0
# HELP prometheus_web_federation_warnings_total Total number of warnings that occurred while sending federation responses.
# TYPE prometheus_web_federation_warnings_total counter
prometheus_web_federation_warnings_total 0
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 5
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```

## Using the expression browser

Let us explore data that Prometheus has collected about itself. To use Prometheus's built-in expression browser, navigate to http://localhost:9090/graph and choose the "Table" view within the "Graph" tab.

As you can gather from [localhost:9090/metrics](http://localhost:9090/metrics), one metric that Prometheus exports about itself is named `prometheus_target_interval_length_seconds` (the actual amount of time between target scrapes). Enter the below into the expression console and then click "Execute":

```sh
prometheus_target_interval_length_seconds
```

This should return a number of different time series (along with the latest value recorded for each), each with the metric name `prometheus_target_interval_length_seconds`, but with different labels. These labels designate different latency percentiles and target group intervals.

```sh
prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.01"}
14.997459604
prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.05"}
14.997459604
prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.5"}
15.0001805
prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.9"}
15.002556683
prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.99"}
15.002564

```



If we are interested only in 99th percentile latencies, we could use this query:

```sh
prometheus_target_interval_length_seconds{quantile="0.99"}
```



```sh

prometheus_target_interval_length_seconds{instance="localhost:9090", interval="15s", job="prometheus", quantile="0.99"}
15.002564
```

To count the number of returned time series, you could write:

```
count(prometheus_target_interval_length_seconds)
```

```sh
{}
5
```

## Using the graphing interface

To graph expressions, navigate to http://localhost:9090/graph and use the "Graph" tab.

For example, enter the following expression to graph the per-second rate of chunks being created in the self-scraped Prometheus:



## Starting up some sample targets

Let's add additional targets for Prometheus to scrape.

The Node Exporter is used as an example target, for more information on using it [see these instructions.](https://prometheus.io/docs/guides/node-exporter/)

```
tar -xzvf node_exporter-*.*.tar.gz
cd node_exporter-*.*

# Start 3 example targets in separate terminals:
./node_exporter --web.listen-address 127.0.0.1:8080
./node_exporter --web.listen-address 127.0.0.1:8081
./node_exporter --web.listen-address 127.0.0.1:8082
```

The Prometheus [**Node Exporter**](https://github.com/prometheus/node_exporter) exposes a wide variety of hardware- and kernel-related metrics.

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
--2024-08-01 02:29:29--  https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
Resolving github.com (github.com)... 140.82.112.4
Connecting to github.com (github.com)|140.82.112.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/a7e04f41-5543-40e2-9060-26fefe32bb4b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240801%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240801T022929Z&X-Amz-Expires=300&X-Amz-Signature=2c5c5cc184341fa747f3133f3630494899225c60d8db7818244045ae8199697d&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.8.2.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2024-08-01 02:29:29--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/a7e04f41-5543-40e2-9060-26fefe32bb4b?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240801%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240801T022929Z&X-Amz-Expires=300&X-Amz-Signature=2c5c5cc184341fa747f3133f3630494899225c60d8db7818244045ae8199697d&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.8.2.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.110.133, 185.199.108.133, 185.199.109.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 10676343 (10M) [application/octet-stream]
Saving to: ‘node_exporter-1.8.2.linux-amd64.tar.gz’

node_exporter-1.8.2.linux- 100%[=====================================>]  10.18M  59.6MB/s    in 0.2s    

2024-08-01 02:29:30 (59.6 MB/s) - ‘node_exporter-1.8.2.linux-amd64.tar.gz’ saved [10676343/10676343]

@pradeepgadde ➜ /workspaces/codespaces-blank $ ls
node_exporter-1.8.2.linux-amd64.tar.gz  prometheus-2.54.0-rc.0.linux-amd64.tar.gz
prometheus-2.54.0-rc.0.linux-amd64
@pradeepgadde ➜ /workspaces/codespaces-blank $ 
```

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank $ tar -xzvf node_exporter-1.8.2.linux-amd64.tar.gz 
node_exporter-1.8.2.linux-amd64/
node_exporter-1.8.2.linux-amd64/NOTICE
node_exporter-1.8.2.linux-amd64/node_exporter
node_exporter-1.8.2.linux-amd64/LICENSE
@pradeepgadde ➜ /workspaces/codespaces-blank $ cd node_exporter-1.8.2.linux-amd64/
@pradeepgadde ➜ /workspaces/codespaces-blank/node_exporter-1.8.2.linux-amd64 $ 
```

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank/node_exporter-1.8.2.linux-amd64 $ ./node_exporter --web.listen-address 127.0.0.1:8080
ts=2024-08-01T02:30:33.951Z caller=node_exporter.go:193 level=info msg="Starting node_exporter" version="(version=1.8.2, branch=HEAD, revision=f1e0e8360aa60b6cb5e5cc1560bed348fc2c1895)"
ts=2024-08-01T02:30:33.951Z caller=node_exporter.go:194 level=info msg="Build context" build_context="(go=go1.22.5, platform=linux/amd64, user=root@03d440803209, date=20240714-11:53:45, tags=unknown)"
ts=2024-08-01T02:30:33.953Z caller=diskstats_common.go:111 level=info collector=diskstats msg="Parsed flag --collector.diskstats.device-exclude" flag=^(z?ram|loop|fd|(h|s|v|xv)d[a-z]|nvme\d+n\d+p)\d+$
ts=2024-08-01T02:30:33.953Z caller=diskstats_linux.go:265 level=error collector=diskstats msg="Failed to open directory, disabling udev device properties" path=/run/udev/data
ts=2024-08-01T02:30:33.953Z caller=filesystem_common.go:111 level=info collector=filesystem msg="Parsed flag --collector.filesystem.mount-points-exclude" flag=^/(dev|proc|run/credentials/.+|sys|var/lib/docker/.+|var/lib/containers/storage/.+)($|/)
ts=2024-08-01T02:30:33.953Z caller=filesystem_common.go:113 level=info collector=filesystem msg="Parsed flag --collector.filesystem.fs-types-exclude" flag=^(autofs|binfmt_misc|bpf|cgroup2?|configfs|debugfs|devpts|devtmpfs|fusectl|hugetlbfs|iso9660|mqueue|nsfs|overlay|proc|procfs|pstore|rpc_pipefs|securityfs|selinuxfs|squashfs|sysfs|tracefs)$
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:111 level=info msg="Enabled collectors"
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=arp
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=bcache
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=bonding
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=btrfs
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=conntrack
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=cpu
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=cpufreq
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=diskstats
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=dmi
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=edac
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=entropy
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=fibrechannel
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=filefd
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=filesystem
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=hwmon
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=infiniband
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=ipvs
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=loadavg
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=mdadm
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=meminfo
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=netclass
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=netdev
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=netstat
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=nfs
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=nfsd
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=nvme
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=os
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=powersupplyclass
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=pressure
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=rapl
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=schedstat
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=selinux
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=sockstat
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=softnet
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=stat
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=tapestats
ts=2024-08-01T02:30:33.953Z caller=node_exporter.go:118 level=info collector=textfile
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=thermal_zone
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=time
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=timex
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=udp_queues
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=uname
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=vmstat
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=watchdog
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=xfs
ts=2024-08-01T02:30:33.954Z caller=node_exporter.go:118 level=info collector=zfs
ts=2024-08-01T02:30:33.954Z caller=tls_config.go:313 level=info msg="Listening on" address=127.0.0.1:8080
ts=2024-08-01T02:30:33.954Z caller=tls_config.go:316 level=info msg="TLS is disabled." http2=false address=127.0.0.1:8080
```

similarly for other two ports

You should now have example targets listening on http://localhost:8080/metrics, http://localhost:8081/metrics, and http://localhost:8082/metrics.

## Configure Prometheus to monitor the sample targets

Now we will configure Prometheus to scrape these new targets. Let's group all three endpoints into one job called `node`. We will imagine that the first two endpoints are production targets, while the third one represents a canary instance. To model this in Prometheus, we can add several groups of endpoints to a single job, adding extra labels to each group of targets. In this example, we will add the `group="production"` label to the first group of targets, while adding `group="canary"` to the second.

To achieve this, add the following job definition to the `scrape_configs` section in your `prometheus.yml` and restart your Prometheus instance:

```yaml
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ cat prometheus.yml 
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
#
scrape_configs:
  - job_name:       'node'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ 
```

Go to the expression browser and verify that Prometheus now has information about time series that these example endpoints expose, such as `node_cpu_seconds_total`.



## Configure rules for aggregating scraped data into new time series

Though not a problem in our example, queries that aggregate over thousands of time series can get slow when computed ad-hoc. To make this more efficient, Prometheus can prerecord expressions into new persisted time series via configured *recording rules*. Let's say we are interested in recording the per-second rate of cpu time (`node_cpu_seconds_total`) averaged over all cpus per instance (but preserving the `job`, `instance` and `mode` dimensions) as measured over a window of 5 minutes. We could write this as:

```sh
avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

To record the time series resulting from this expression into a new metric called `job_instance_mode:node_cpu_seconds:avg_rate5m`, create a file with the following recording rule and save it as `prometheus.rules.yml`:

```yaml
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
```

```yaml
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ cat prometheus.rules.yml 
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ 
```

To make Prometheus pick up this new rule, add a `rule_files` statement in your `prometheus.yml`. The config should now look like this:

```yaml
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ cat prometheus.yml 
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "prometheus.rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
#
scrape_configs:
  - job_name:       'node'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
@pradeepgadde ➜ /workspaces/codespaces-blank/prometheus-2.54.0-rc.0.linux-amd64 $ 
```

Restart Prometheus with the new configuration and verify that a new time series with the metric name `job_instance_mode:node_cpu_seconds:avg_rate5m` is now available by querying it through the expression browser or graphing it.

## Reloading configuration

As mentioned in the [configuration documentation](https://prometheus.io/docs/prometheus/latest/configuration/configuration/) a Prometheus instance can have its configuration reloaded without restarting the process by using the `SIGHUP` signal. If you're running on Linux this can be performed by using `kill -s SIGHUP <PID>`, replacing `<PID>` with your Prometheus process ID.

## Shutting down your instance gracefully.

While Prometheus does have recovery mechanisms in the case that there is an abrupt process failure it is recommend to use the `SIGTERM` signal to cleanly shutdown a Prometheus instance. If you're running on Linux this can be performed by using `kill -s SIGTERM <PID>`, replacing `<PID>` with your Prometheus process ID.

```sh
@pradeepgadde ➜ /workspaces/codespaces-blank/node_exporter-1.8.2.linux-amd64 $ ps -aux |grep prome
codespa+   18382  0.1  0.9 1326524 79908 ?       Sl   02:34   0:00 ./prometheus --config.file=prometheus.yml
codespa+   22287  0.0  0.0   8172  2432 pts/4    S+   02:41   0:00 grep --color=auto prome
@pradeepgadde ➜ /workspaces/codespaces-blank/node_exporter-1.8.2.linux-amd64 $ 
```

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  scrape_protocols:
  - OpenMetricsText1.0.0
  - OpenMetricsText0.0.1
  - PrometheusText0.0.4
  evaluation_interval: 15s
runtime:
  gogc: 75
alerting:
  alertmanagers:
  - follow_redirects: true
    enable_http2: true
    scheme: http
    timeout: 10s
    api_version: v2
    static_configs:
    - targets: []
rule_files:
- prometheus.rules.yml
scrape_configs:
- job_name: node
  honor_timestamps: true
  track_timestamps_staleness: false
  scrape_interval: 5s
  scrape_timeout: 5s
  scrape_protocols:
  - OpenMetricsText1.0.0
  - OpenMetricsText0.0.1
  - PrometheusText0.0.4
  metrics_path: /metrics
  scheme: http
  enable_compression: true
  follow_redirects: true
  enable_http2: true
  static_configs:
  - targets:
    - localhost:8080
    - localhost:8081
    labels:
      group: production
  - targets:
    - localhost:8082
    labels:
      group: canary
```

