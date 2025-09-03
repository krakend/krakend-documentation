---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
notoc: true
linktitle: Prometheus
title: KrakenD API Gateway - Prometheus Telemetry Integration
description: Learn how to integrate Prometheus telemetry with KrakenD API Gateway for efficient monitoring and performance analysis of your APIs
weight: 80
menu:
  community_v2.5:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  log_prefix:
  - "[SERVICE: Opencensus]"
  scope:
  - service
---
[Prometheus](https://prometheus.io/) is an open-source systems monitoring and alerting toolkit.

The Opencensus exporter allows you to expose data to Prometheus, and publishes a `/metrics` endpoint in the selected port.

Example: `http://localhost:9091/metrics`

Enabling it only requires you to include in the root level of your configuration the Opencensus middleware with the `prometheus` exporter. Specify the `port` which Prometheus should hit, the `namespace` (optional), and Prometheus now can pull the data.
```json
{
  "version": 3,
  "extra_config": {
    "telemetry/opencensus": {
        "sample_rate": 100,
        "reporting_period": 0,
        "exporters": {
          "prometheus": {
              "port": 9091,
              "namespace": "krakend",
              "tag_host": false,
              "tag_path": true,
              "tag_method": true,
              "tag_statuscode": false
          }
      }
    }
  }
}
```

As with all [OpenCensus exporters](/docs/v2.5/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.5" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `prometheus` entry with the following properties:

{{< schema version="v2.5" data="telemetry/opencensus.json" property="exporters" filter="prometheus" >}}

## Example of `prometheus.yml`
This is a simple example to pull data from the Prometheus integration every minute:

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'krakend'
    static_configs:
      - targets: ['krakend:9091']
```

## Prometheus output example
```bash
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 8.0701e-05
go_gc_duration_seconds{quantile="0.25"} 0.000180217
go_gc_duration_seconds{quantile="0.5"} 0.000238727
go_gc_duration_seconds{quantile="0.75"} 0.000340108
go_gc_duration_seconds{quantile="1"} 0.003425704
go_gc_duration_seconds_sum 0.162472194
go_gc_duration_seconds_count 426
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 18
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.9"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 1.290344e+07
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 4.436000928e+09
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.513357e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 7.4315998e+07
# HELP go_memstats_gc_cpu_fraction The fraction of this program's available CPU time used by the GC since the program started.
# TYPE go_memstats_gc_cpu_fraction gauge
go_memstats_gc_cpu_fraction 0.01233588465264479
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 6.125576e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 1.290344e+07
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 1.929216e+07
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 2.3437312e+07
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 81569
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 1.3262848e+07
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 4.2729472e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.6535700877830684e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 7.4397567e+07
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 28800
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 32768
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 621248
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 737280
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 2.3062192e+07
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 6.026355e+06
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 3.407872e+06
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 3.407872e+06
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 6.057268e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 31
# HELP krakend_opencensus_io_http_server_latency Latency distribution of HTTP requests
# TYPE krakend_opencensus_io_http_server_latency histogram
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1"} 78792
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2"} 83989
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="3"} 93954
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4"} 114961
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="5"} 144535
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="6"} 168742
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="8"} 189585
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="10"} 194593
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="13"} 198487
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="16"} 199460
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="20"} 199757
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="25"} 199796
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="30"} 199799
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="40"} 199799
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="50"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="65"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="80"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="100"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="130"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="160"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="200"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="250"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="300"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="400"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="500"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="650"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="800"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="5000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="10000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="20000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="50000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="100000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="+Inf"} 199800
krakend_opencensus_io_http_server_latency_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 628394.4669860053
krakend_opencensus_io_http_server_latency_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1"} 91160
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2"} 97163
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="3"} 104034
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4"} 113157
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="5"} 124866
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="6"} 138026
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="8"} 165065
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="10"} 181831
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="13"} 192938
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="16"} 197200
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="20"} 199193
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="25"} 199685
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="30"} 199775
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="40"} 199799
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="50"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="65"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="80"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="100"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="130"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="160"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="200"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="250"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="300"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="400"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="500"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="650"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="800"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="5000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="10000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="20000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="50000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="100000"} 199800
krakend_opencensus_io_http_server_latency_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="+Inf"} 199800
krakend_opencensus_io_http_server_latency_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 763343.1322959992
krakend_opencensus_io_http_server_latency_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 199800
# HELP krakend_opencensus_io_http_server_request_bytes Size distribution of HTTP request body
# TYPE krakend_opencensus_io_http_server_request_bytes histogram
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1024"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2048"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4096"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="16384"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="65536"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="262144"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.048576e+06"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4.194304e+06"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.6777216e+07"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="6.7108864e+07"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2.68435456e+08"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.073741824e+09"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4.294967296e+09"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="+Inf"} 199800
krakend_opencensus_io_http_server_request_bytes_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 0
krakend_opencensus_io_http_server_request_bytes_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1024"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2048"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4096"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="16384"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="65536"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="262144"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.048576e+06"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4.194304e+06"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.6777216e+07"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="6.7108864e+07"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2.68435456e+08"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.073741824e+09"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4.294967296e+09"} 199800
krakend_opencensus_io_http_server_request_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="+Inf"} 199800
krakend_opencensus_io_http_server_request_bytes_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 0
krakend_opencensus_io_http_server_request_bytes_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 199800
# HELP krakend_opencensus_io_http_server_request_count Count of HTTP requests started
# TYPE krakend_opencensus_io_http_server_request_count counter
krakend_opencensus_io_http_server_request_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status=""} 199800
krakend_opencensus_io_http_server_request_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status=""} 199800
# HELP krakend_opencensus_io_http_server_request_count_by_method Server request count by HTTP method
# TYPE krakend_opencensus_io_http_server_request_count_by_method counter
krakend_opencensus_io_http_server_request_count_by_method{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status=""} 199800
krakend_opencensus_io_http_server_request_count_by_method{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status=""} 199800
# HELP krakend_opencensus_io_http_server_response_bytes Size distribution of HTTP response body
# TYPE krakend_opencensus_io_http_server_response_bytes histogram
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1024"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2048"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4096"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="16384"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="65536"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="262144"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.048576e+06"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4.194304e+06"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.6777216e+07"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="6.7108864e+07"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="2.68435456e+08"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="1.073741824e+09"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="4.294967296e+09"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500",le="+Inf"} 199800
krakend_opencensus_io_http_server_response_bytes_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} -199800
krakend_opencensus_io_http_server_response_bytes_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1024"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2048"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4096"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="16384"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="65536"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="262144"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.048576e+06"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4.194304e+06"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.6777216e+07"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="6.7108864e+07"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="2.68435456e+08"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="1.073741824e+09"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="4.294967296e+09"} 199800
krakend_opencensus_io_http_server_response_bytes_bucket{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200",le="+Inf"} 199800
krakend_opencensus_io_http_server_response_bytes_sum{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 1.000998e+08
krakend_opencensus_io_http_server_response_bytes_count{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 199800
# HELP krakend_opencensus_io_http_server_response_count_by_status_code Server response count by status code
# TYPE krakend_opencensus_io_http_server_response_count_by_status_code counter
krakend_opencensus_io_http_server_response_count_by_status_code{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/billing",http_status="500"} 199800
krakend_opencensus_io_http_server_response_count_by_status_code{http_host="localhost:8080",http_method="GET",http_path="/v1/soap-to-json/account/info",http_status="200"} 199800
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 48.24
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 11
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 7.286784e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.65357005173e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 7.93280512e+08
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes -1
```
