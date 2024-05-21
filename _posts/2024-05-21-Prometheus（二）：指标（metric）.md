---
title: Prometheus（二）：指标（metric）
author: mmy83
date: 2024-05-21 19:24:00 +0800
categories: [专题, Prometheus]
tags: [监控, 指标, 样本, metric, Prometheus]
math: true
mermaid: true
image:
  path: /images/2024-05-21/Prometheus（二）：指标（metric）/Prometheus（二）：指标（metric）-00.png
  lqip: data:image/webp;base64,UklGRjoAAABXRUJQVlA4IC4AAACQAQCdASoIAAUAAUAmJZQAAudZt7gA/vzJsdqgU0eBYW+YMGCQ8L8+DUZ6AAAA
  alt: Prometheus（二）：指标（metric）
---

## 样本

&emsp;&emsp;Prometheus会将所有采集到的样本数据以时间序列（time-series）的方式保存在内存数据库中，并且定时保存到硬盘上。time-series是按照时间戳和值的序列顺序存放的，我们称之为向量(vector). 每条time-series通过指标名称(metrics name)和一组标签集(labelset)命名。Prometheus采集到的数据称为样本(sample)，一个样本由以下三部分组成：

* 指标(metric)：metric name和描述当前样本特征的labelsets;

* 时间戳(timestamp)：一个精确到毫秒的时间戳;

* 样本值(value)： 一个float64的浮点型数据表示当前样本的值。

## 指标(Metric)

&emsp;&emsp;通过访问[http://localhost:9100/metrics](http://localhost:9100/metrics)，可以查看到Prometheus采集的数据。部分内容片段如下：

```console
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 8
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.22.2"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 406352
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 406352
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.448441e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 77
```

&emsp;&emsp;这里面包含了很多监控指标，这些指标都是以`# HELP`和`# TYPE`开头的。

* `# HELP`：描述监控指标的含义；
* `# TYPE`：描述监控指标的类型。
* 没有`# HELP`和`# TYPE`开头的都是监控指标。

&emsp;&emsp;Prometheus定义了4种不同的指标类型(metric type)：Counter（计数器）、Gauge（仪表盘）、Histogram（直方图）、Summary（摘要）。

* Counter（计数器）：只增不减的计数器
* Gauge（仪表盘）：可增可减的仪表盘
* Histogram（直方图）：分析样本的分布情况
* Summary（摘要）：用于统计样本

&emsp;&emsp;在形式上，所有的指标(Metric)都通过如下格式标示：

```console
<metric name>{<label name>=<label value>, ...}
```

&emsp;&emsp;在Prometheus的底层实现中指标名称实际上是以 ```__name__=<metric name>``` 的形式保存在数据库中的，因此以下两种方式均表示的同一条time-series：

```console
api_http_requests_total{method="POST", handler="/messages"}
# 相当于
{__name__="api_http_requests_total"，method="POST", handler="/messages"}
```

## 结束

&emsp;&emsp;Prometheus通过指标名称（metrics name）以及对应的一组标签（labelset）唯一定义一条时间序列。指标名称反映了监控样本的基本标识，而label则在这个基本特征上为采集到的数据提供了多种特征维度。用户可以基于这些特征维度过滤，聚合，统计从而产生新的计算后的一条时间序列。
