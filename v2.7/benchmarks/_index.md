---
lastmod: 2016-10-28
old_version: true
date: 2016-10-28
linktitle: Benchmarks Overview
title: API Gateway Performance Benchmarks with KrakenD
description: Explore the performance benchmarks conducted with KrakenD API Gateway to ensure optimal performance and scalability for your APIs.
weight: 1
menu:
  community_v2.7:
    parent: "300 Benchmarks"
---
## KrakenD, the **ultra performer** API Gateway
An API Gateway is a component that needs to deliver really fast, as it is an added layer in the infrastructure. KrakenD
was built with performance in mind. In this page and inner pages, you'll find several tests we did to measure the performance.
We also invite you to do them for yourself!

## TL;DR: **Benchmark results**
**~18,000 requests/second** on an ordinary laptop.

The following table summarizes different performance tests using Amazon EC2 virtual instances and an example with a laptop.

 <table class="table table-striped">
    <tbody><tr>
        <th style="width: 10px">#</th>
        <th>Hardware specs</th>
        <th>Requests second</th>
        <th>Average response</th>
    </tr>
    <tr>
        <td>1.</td>
        <td>Amazon EC2 (c4.2xlarge)</td>
        <td>10126.1613 reqs/s</td>
        <td>9.8ms</td>
    </tr>
    <tr>
        <td>2.</td>
        <td>Amazon EC2 (c4.xlarge)</td>
        <td>8465.4012 reqs/s</td>
        <td>11.7ms</td>
    </tr>
    <tr>
        <td>3.</td>
        <td>Amazon EC2 (m4.large)</td>
        <td>3634.1247 reqs/s</td>
        <td>27.3ms</td>
    </tr>
    <tr>
        <td>4.</td>
        <td>Amazon EC2 (t2.medium)</td>
        <td>2781.8611 reqs/s</td>
        <td>351.3ms</td>
    </tr>
    <tr>
        <td>5.</td>
        <td>Amazon EC2 (t2.micro)</td>
        <td>2757.6407 reqs/s</td>
        <td>35.8ms</td>
    </tr>
    <tr>
        <td>6.</td>
        <td>MacBook Pro (Aug 2015) 2,2 GHz Intel Core i7</td>
        <td>18157.4274 reqs/s</td>
        <td>5.5ms</td>
    </tr>
    </tbody>
 </table>

### Benchmark in a MacBook Pro

[Here you will find the results of the benchmarks](/docs/v2.7/benchmarks/local/)

### Benchmark in Amazon AWS EC2 Instances

[Here you will find the results of the benchmarks](/docs/v2.7/benchmarks/aws/)

### API Gateway Benchmark

[Here you will find the results of the comparisons](/docs/v2.7/benchmarks/api-gateway-benchmark/)

## Tooling

### hey

Some local benchmarks used the [hey](https://github.com/rakyll/hey) tool, which is an Apache Benchmark (ab) replacement tool.

### api-gateway-benchmark

> This project aims to provide a complete set of tools needed to do simple performance comparisons in the API manager/gateway space. It is inspired by the great Framework Benchmarks project by TechEmpower.

Check the [varnish/api-gateway-benchmarks](https://github.com/varnish/api-gateway-benchmarks) project for more info.

### Lwan

[LWAN](https://lwan.ws/) is a high performance web server used to build the backends REST APIs for KrakenD to load the data during the benchmarks.
