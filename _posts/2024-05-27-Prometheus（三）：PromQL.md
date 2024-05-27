---
title: Prometheus（三）：PromQL
author: mmy83
date: 2024-05-27 16:27:00 +0800
categories: [专题, Prometheus]
tags: [监控, Prometheus]
math: true
mermaid: true
image:
  path: /images/2024-05-27/Prometheus（三）：PromQL/Prometheus（三）：PromQL-00.png
  lqip: data:image/webp;base64,UklGRjwAAABXRUJQVlA4IDAAAACwAQCdASoIAAUAAUAmJaQAAu0dpr8AAP795Dt/87k1gNovTneze7QcDml8Oo0ZCAA=
  alt: Prometheus（三）：PromQL
---

## 介绍

&emsp;&emsp;PromQL是Prometheus的自定义查询语言。通过PromQL用户可以非常方便地对监控样本数据进行统计分析，PromQL支持常见的运算操作符，同时PromQL中还提供了大量的内置函数可以实现对数据的高级处理。被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。

## 类型

&emsp;&emsp;PromQL支持以下四种类型：

- Instant vector：瞬时向量，表示一组时间序列，每个时间序列只有一个样本值。
- Range vector：区间向量，表示一组时间序列，每个时间序列有多个样本值。
- 标量：表示单个浮点值。
- 字符串：表示单个字符串值。

## 时间序列选择器

```
<metric name>{label=value,label=value,...}[range]
```

### 标签选择器

```
# 查询Prometheus http状态码为400的请求数量。
prometheus_http_requests_total{code="400"}
```

&emsp;&emsp;标签匹配运算符:

- =：与字符串匹配
- !=：与字符串不匹配
- =~：与正则匹配
- !~：与正则不匹配

```
# 查询Prometheus http状态码为4xx或5xx并且handler为/api/v1/query的请求数量
prometheus_http_requests_total{code=~"4..|5..",handler="/api/v1/query"}
# 或
{code=~"4.*|5.*",handler="/api/v1/query",__name__="prometheus_http_requests_total"}
```

> PromQL中，标签选择器可以同时使用多个标签，多个标签之间使用逗号分隔。
{: .prompt-tip }

### 范围选择器

```
# 查询过去5分钟Prometheus健康检查的采样记录。
prometheus_http_requests_total{code="200",handler="/-/healthy"}[5m]
```

> 单位：ms、s、m、h、d、w、y
>
> 时间串联：[1h5m]一小时5分钟
{: .prompt-tip }

### 通过offset

```
# 通过offset将时间倒退5分钟，即查询5分钟之前的数据。
prometheus_http_requests_total{code="200"} offset 5m
```

### @修饰符

```
# 修饰符@允许更改查询中各个瞬时向量和范围向量的评估时间。提供给@修饰符的时间是 unix 时间戳，用浮点文字描述。

prometheus_http_requests_total{code="200"} @ 1646089826
```

&emsp;&emsp;修饰符@支持上述所有数字文字的表示。它与offset修饰符一起使用，其中偏移量相对于修饰符时间应用@ 。无论修饰符的顺序如何，结果都是相同的。

```
# offset after @
prometheus_http_requests_total @ 1609746000 offset 5m
# offset before @
prometheus_http_requests_total offset 5m @ 1609746000
```

&emsp;&emsp;此外，start()和end()也可以作为@特殊值用作修饰符的值。对于范围查询，它们分别解析为范围查询的开始和结束，并且在所有步骤中保持不变。对于即时查询，start()和end()解析至评估时间。

```
prometheus_http_requests_total @ start()
rate(prometheus_http_requests_total[5m] @ end())
# 请注意，@修饰符允许查询提前查看其评估时间。
```

## 子查询

&emsp;&emsp;子查询允许您针对给定的范围和分辨率运行即时查询。子查询的结果是一个范围向量。

```
<instant_query> '[' <range> ':' [<resolution>] ']' [ @ <float_literal> ] [ offset <duration> ]
```

&emsp;&emsp;```<resolution>```是可选的。默认为全局评估间隔。

## 运算符

### 算术运算符

- +（加法）
- -（减法）
- *（乘法）
- /（除法）
- %（取模）
- ^（幂）

```
prometheus_http_response_size_bytes_sum / 1024
```

### 比较运算符

- ```==```（相等）
- ```!=```（不相等）
- ```>```（大于）
- ```<```（小于）
- ```>=```（大于或等于）
- ```<=```（小于或等于）

### 逻辑运算符

- and（与）
- or（或）
- unless（非）

## 向量匹配

&emsp;&emsp;这部分内容和sQL中的join类似。

### 向量匹配关键字

- on ：允许将考虑的标签集缩减为提供的列表
- ignoring ：允许在匹配时忽略某些标签

### 分组修饰符

&emsp;&emsp;这部分的内容很像sql中的```left join```和```righ join```。

- group_left
- group_right

&emsp;&emsp;如果两个瞬时向量数量不一致时可通过group_left、group_right指定以那一侧为准

### 一对一向量匹配

```
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>

# 输入示例：

method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
#示例查询：

method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
#结果：

{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

### 多对一和一对多向量匹配

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>

# 输入示例：
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120

示例查询：
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m

#结果：
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

### 聚合运算符

Prometheus 支持以下内置聚合运算符，可用于聚合单个即时向量的元素，从而生成具有聚合值且元素较少的新向量：

- sum（计算维度总和）
- min（选择最小尺寸）
- max（选择最大尺寸）
- avg（计算各个维度的平均值）
- group（结果向量中的所有值都是 1）
- stddev（计算维度上的总体标准差）
- stdvar（计算各维度的总体标准方差）
- count（计算向量中元素的数量）
- count_values（计算具有相同值的元素数量）
- bottomk（按样本值最小的 k 个元素）
- topk（按样本值最大的 k 个元素）
- quantile（计算维度上的 φ 分位数 (0 ≤ φ ≤ 1)）

> 手册是就算说这个是运算符而不是函数
{: .prompt-tip }

### 优先级

&emsp;&emsp;Prometheus 中二元运算符的优先级，从高到低。

1. ```^```
2. ```*```, ```/```, ```%```,```atan2```
3. ```+```,```-```
4. ```==```, ```!=```, ```<=```, ```<```, ```>=```,```>```
5. ```and```,```unless```
6. ```or```

## 函数

- abs()
- absent()
- absent_over_time()
- ceil()
- changes()
- clamp()
- clamp_max()
- clamp_min()
- day_of_month()
- day_of_week()
- day_of_year()
- days_in_month()
- delta()
- deriv()
- exp()
- floor()
- histogram_avg()
- histogram_count()和histogram_sum()
- histogram_fraction()
- histogram_quantile()
- histogram_stddev()和histogram_stdvar()
- holt_winters()
- hour()
- idelta()
- increase()
- irate()
- label_join()
- label_replace()
- ln()
- log2()
- log10()
- minute()
- month()
- predict_linear()
- rate()
- resets()
- round()
- scalar()
- sgn()
- sort()
- sort_desc()
- sort_by_label()
- sort_by_label_desc()
- sqrt()
- time()
- timestamp()
- vector()
- year()
- ```<aggregation>_over_time()```
- 三角函数

## 参考

[https://zhuanlan.zhihu.com/p/477177336](https://zhuanlan.zhihu.com/p/477177336)

[https://yunlzheng.gitbook.io/prometheus-book](https://yunlzheng.gitbook.io/prometheus-book)

[https://prometheus.io/docs/prometheus/latest/querying/basics/](https://prometheus.io/docs/prometheus/latest/querying/basics/)
