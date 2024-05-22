---
title: Prometheus（一）：部署
author: mmy83
date: 2024-05-21 15:57:00 +0800
categories: [专题, Prometheus]
tags: [监控, Prometheus]
math: true
mermaid: true
image:
  path: /images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-00.png
  lqip: data:image/webp;base64,UklGRjAAAABXRUJQVlA4ICQAAACQAQCdASoIAAQAAUAmJaQAApz9zgAA/v2gfrdFivENpONmEAA=
  alt: Prometheus（一）：部署
---

## 介绍

&emsp;&emsp;Prometheus是一个用Golang语言编写的开放性的监控解决方案，用户可以非常方便的安装和使用Prometheus并且能够非常方便的对其进行扩展。为了能够更加直观的了解Prometheus Server，接下来我们将在本地部署并运行一个Prometheus Server实例，通过Node Exporter采集当前主机的系统资源使用情况。 并通过Grafana创建一个简单的可视化仪表盘。

## Prometheus Server 部署

&emsp;&emsp;Prometheus Server是一个独立的二进制文件，用户可以非常方便的安装和使用。Prometheus Server目前支持多种部署方法，这里我们介绍两种部署方法：

### 二进制部署

&emsp;&emsp;Prometheus基于Golang编写，编译后的软件包，不依赖于任何的第三方依赖。用户只需要下载对应平台的二进制包，解压并且添加基本的配置即可正常启动Prometheus Server。

&emsp;&emsp;从[https://prometheus.io/download/](https://prometheus.io/download/)找到最新版本的Prometheus Sevrer软件包，这里选择LTS版本。

![Prometheus Sevrer 下载](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-01.png)

&emsp;&emsp;下载完成后，解压到任意目录，目录下包含默认配置文件```prometheus.yml```，这里暂时不做修改：

```yml
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

```

&emsp;&emsp;启动Prometheus Server：

```bash
# 启动Prometheus Server，默认监听端口9090，默认数据存储目录为data/
./prometheus

# 或

#启动Prometheus Server，指定监听端口9091，指定数据存储目录data2/
./prometheus --web.listen-address=":9090" --storage.tsdb.path="data/"
```

&emsp;&emsp;启动成功后，访问```http://localhost:9090/```，可以看到Prometheus Server的Web页面：

![Prometheus Server的Web页面](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-02.png)

### docker部署

&emsp;&emsp;对于Docker用户，直接使用Prometheus的镜像即可启动Prometheus Server：

```bash
docker run -p 9090:9090 -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

## 安装Node Exporter

&emsp;&emsp;在Prometheus的架构设计中，Prometheus Server并不直接服务监控特定的目标，其主要任务负责数据的收集，存储并且对外提供数据查询支持。因此为了能够能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。这里为了能够采集到主机的运行指标如CPU, 内存，磁盘等信息。可以使用Node Exporter。

&emsp;&emsp;Node Exporter同样采用Golang编写，并且不存在任何的第三方依赖，只需要下载，解压即可运行。可以从[https://prometheus.io/download/](https://prometheus.io/download/)获取最新的node exporter版本的二进制包。

![Node Exporter 下载](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-03.png)

```bash
./node_exporter
# 或
./node_exporter --web.listen-address=":9100"
```

![页面](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-04.png)

![指标](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-05.png)

## 从Node Exporter收集监控数据

```yml
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
# 新增 采集node exporter监控数据
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

&emsp;&emsp;重新启动Prometheus Server，访问http://localhost:9090，进入到Prometheus Server。如果输入“up”并且点击执行按钮以后，可以看到如下结果：

![Prometheus Server up](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-06.png)

![node exporter监控数据](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-07.png)

## 安装Grafana

&emsp;&emsp;Prometheus UI提供了快速验证PromQL以及临时可视化支持的能力，而在大多数场景下引入监控系统通常还需要构建可以长期使用的监控数据可视化面板（Dashboard）。这时用户可以考虑使用第三方的可视化工具如Grafana，Grafana是一个开源的可视化平台，并且提供了对Prometheus的完整支持。

&emsp;&emsp;从[https://grafana.com/grafana/download](https://grafana.com/grafana/download)下载自己所需版本。

![Grafana 下载](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-08.png)

&emsp;&emsp;解压后执行：

```bash
./bin/grafana-server
```

&emsp;&emsp;访问[http://localhost:3000](http://localhost:3000)就可以进入到Grafana的界面中，默认情况下使用账户```admin/admin```进行登录。在Grafana首页中显示默认的使用向导，包括：安装、添加数据源、创建Dashboard、邀请成员、以及安装应用和插件等主要流程，到这里还没有学过PromQL，所以这里使用导入其他人做好面板来使用：

![登录](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-09.png)

![登入](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-10.png)

![添加数据源](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-11.png)

![选择prometheus](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-12.png)

![添加数据源url](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-13.png)

![保存并测试](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-14.png)

![创建可视化面板](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-15.png)

![导入一个面板](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-16.png)

![导入id](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-17.png)

&emsp;&emsp;这里的导入需要一个导入的ID，这个id可以从[https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/)获得，在上面选择自己需要的面板，点击```Export```按钮，然后复制面板的ID。

![选择面板](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-18.png)

![获取面板ID](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-19.png)

![填写ID，导入](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-20.png)

![选择数据源导入](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-21.png)

![效果](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-22.png)

![保存-01](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-23.png)

![保存-02](/images/2024-05-21/Prometheus（一）：部署/Prometheus（一）：部署-24.png)

## 结束

&emsp;&emsp;到此为止，一个监控系统就部署完成了，这里只是简单的导入了一个面板，后续还需要学习PromQL，以及如何自定义监控面板。
