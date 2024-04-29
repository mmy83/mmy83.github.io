---
title: Kubernetes基础（一）：介绍
author: mmy83
date: 2024-04-29 11:48:00 +0800
categories: [专题, Kubernetes]
tags: [云平台, k8s, Kubernetes]
math: true
mermaid: true
image:
  path: /images/2024-04-29/Kubernetes基础（一）：介绍/Kubernetes基础（一）：介绍-00.png
  lqip: data:image/webp;base64,UklGRk4AAABXRUJQVlA4IEIAAAAwAgCdASoIAAQAAUAmJagCdLoB+AAC0mp6wAD+6VcWb+uIF2QP82dNqh0TokPf/2nIPSsfU5B94L/5Dwk6Auc1wAA=
  alt: Kubernetes基础（一）：介绍
---

## 引

&emsp;&emsp;通过对```MicroK8s```的学习，已经成功搭建了一个```Kubernetes```集群，开启了```Dashboard```，成功部署了```Nginx```应用。但是并没有详细介绍```Kubernetes```的相关知识，只是部署了一个应用。之所以这样做是因为学习```Kubernetes```需要一个环境，而```Kubernetes```的安装和部署比较复杂，所以才选择```MicroK8s```。

> 注：
> 因为我们的环境是```Microk8s```，使用的时候虽然大部分和```Kubernetes```一样，但是有些地方需要做些调整：
>
> * ```Kubernetes```里的命令在```Microk8s```中需要使用```microk8s```命令，例如： ```microk8s kubectl```或者```microk8s.kubectl```，有人建议用别名，但是我没有这样做。
> * 增加了一些命令，如：```microk8s.status```、```microk8s.enable```等。
{: .prompt-tip }

&emsp;&emsp;现在环境已经搭建成功，接下来需要介绍```Kubernetes```的相关知识。

## Kubernetes介绍

&emsp;&emsp;```Kubernetes``` 也称为 K8s，是用于自动部署、扩缩和管理容器化应用程序的开源系统。它将组成应用程序的容器组合成逻辑单元，以便于管理和服务发现。```Kubernetes``` 源自Google 15 年生产环境的运维经验，同时凝聚了社区的最佳创意和实践。

* __星际尺度__：Google 每周运行数十亿个容器，```Kubernetes``` 基于与之相同的原则来设计，能够在不扩张运维团队的情况下进行规模扩展。
* __永不过时__：无论是本地测试，还是跨国公司，```Kubernetes``` 的灵活性都能让你在应对复杂系统时得心应手。
* __处处适用__：```Kubernetes``` 是开源系统，可以自由地部署在企业内部，私有云、混合云或公有云，让您轻松地做出合适的选择。

## Kubernetes 特性

* __自动化上线和回滚__
```Kubernetes``` 会分步骤地将针对应用或其配置的更改上线，同时监视应用程序运行状况以确保你不会同时终止所有实例。如果出现问题，```Kubernetes``` 会为你回滚所作更改。你应该充分利用不断成长的部署方案生态系统。
* __服务发现与负载均衡__
你无需修改应用来使用陌生的服务发现机制。```Kubernetes``` 为每个 ```Pod``` 提供了自己的 IP 地址并为一组 ```Pod``` 提供一个 DNS 名称，并且可以在它们之间实现负载均衡。
* __自我修复__
重新启动失败的容器，在节点死亡时替换并重新调度容器， 杀死不响应用户定义的健康检查的容器， 并且在它们准备好服务之前不会将它们公布给客户端。
* __存储编排__
自动挂载所选存储系统，包括本地存储、公有云提供商所提供的存储或者诸如 iSCSI 或 NFS 这类网络存储系统。
* __Secret 和配置管理__
部署和更新 Secret 和应用程序的配置而不必重新构建容器镜像， 且不必将软件堆栈配置中的秘密信息暴露出来。
* __自动装箱__
根据资源需求和其他限制自动放置容器，同时避免影响可用性。 将关键性的和尽力而为性质的工作负载进行混合放置，以提高资源利用率并节省更多资源。
* __批量执行__
除了服务之外，```Kubernetes``` 还可以管理你的批处理和 CI 工作负载，在期望时替换掉失效的容器。
* __IPv4/IPv6 双协议栈__
为 Pod 和 Service 分配 IPv4 和 IPv6 地址
* __水平扩缩__
使用一个简单的命令、一个 UI 或基于 CPU 使用情况自动对应用程序进行扩缩。
* __为扩展性设计__
无需更改上游源码即可扩展你的 ```Kubernetes``` 集群。

## 概念

### 对象

&emsp;&emsp;在```Kubernetes```中，对象是```Kubernetes```集群中运行的实体。这里简单列举一下常用的资源对象。

* __Pod__:```Kubernetes```中最小的部署单元，可以包含一个或多个容器。这些容器共享存储、网络、以及进程命名空间，并作为单个实体进行管理。
* __Service__:定义了如何访问一组```Pod```。它提供负载均衡，并允许你通过单一稳定的IP地址和端口号访问一组```Pod```。
* __ReplicaSet__:确保指定数量的```Pod```副本正在运行。如果```Pod```数量少于指定数量，```ReplicaSet```会启动新的```Pod```；如果Pod数量超过指定数量，它会终止多余的```Pod```。
* __Deployment__:提供了声明式的方式来更新应用程序。它管理```ReplicaSet```和```Pod```，并提供滚动更新和回滚功能。
* __Namespace__:提供了一个虚拟的集群环境，用于将资源组隔离成不同的逻辑组。不同的团队或项目可以使用不同的Namespace来管理其资源。
* __Ingress__:网络管理，提供了HTTP(S)路由到集群内的服务，可以作为```Kubernetes```集群的入口点。
* __Secret__:用于存储小量的敏感信息，如密码、OAuth令牌或SSH密钥。这些信息可以安全地传递给```Pod```中的容器。
* __ConfigMap__:用于存储配置信息，这些信息可以被```Pod```中的容器使用。它允许你解耦配置与镜像，使得配置更改更容易管理。
* __PersistentVolumeClaim (PVC) 和 PersistentVolume (PV)__:PV提供了网络存储的抽象，而```PVC```是用户存储请求的声明。```PVC```与```PV```配对，以提供持久存储给```Pod```。
* __Job__:用于运行一次性任务，直到任务成功完成。
* __CronJob__:用于在```Kubernetes```集群中运行定时任务。
* __DaemonSet__:守护进程集，确保每个节点上都运行了指定的```Pod```副本。常用于运行日志收集、监控等守护进程。
* __StatefulSet__:有状态应用集，用于管理有状态的应用程序。与```Deployment```和```ReplicaSet```不同，```StatefulSet```提供了稳定的网络标识符和持久的存储。
* __ServiceAccount__:安全管理，为```Pod```中的进程提供身份认证信息，以便它们可以访问```Kubernetes API```。
* __Role 和 RoleBinding__:```Role```定义了一组权限，而```RoleBinding```将这些权限绑定到一个或多个```ServiceAccount```或用户。
* __ClusterRole 和 ClusterRoleBinding__:与```Role```和```RoleBinding```类似，但适用于集群范围的资源。
* __HorizontalPodAutoscaler (HPA)__:自动伸缩，HPA基于观察到的CPU利用率或其他自定义指标自动缩放```Pod```的数量。
* __NetworkPolicy__:网络管理，用于定义```Pod```之间允许的网络通信策略。
* __PodDisruptionBudget__:应用管理，用于确保在自愿或非自愿中断（如节点维护或驱逐）期间，有足够数量的```Pod```副本可用。
* __Volume__:存储管理，提供了```Pod```中容器的存储卷。这些卷可以来自多种来源，如本地磁盘、网络存储等。
* __VolumeSnapshot、VolumeSnapshotClass 和 VolumeSnapshotContent__:这些与存储卷的快照管理相关，允许用户创建、保存和恢复存储卷的快照。
* __StorageClass__:存储管理，定义了如何动态创建```PersistentVolumes```。它描述了```PV```的“类”，包括存储提供者、参数等。
* __VolumeAttachment__:存储管理，用于将一个卷附加到一个节点，而不需要该卷被```Pod```使用。

&emsp;&emsp;```Kubernetes```对象还是挺多的，而且还都很重要，在安装```MicroK8s```的时候，我们已经接触了```Pod```、```Deployment```、```ReplicaSet```、```Service```、```Namespace```、```Secret```、```Ingress```、等。

### 对象的描述

&emsp;&emsp;```Kubernetes```对象都是通过```YAML```或者```JSON```描述的，我们回头看一下之前我们部署```Nginx```应用的```Deployment```对象的描述文件：

```yml
# 接口版本
apiVersion: apps/v1
# 对象类型
kind: Deployment
# 对象元数据
metadata:
  name: nginx-deployment
# 对象对象规约（对象期望）
spec:
  # 设置副本数量
  replicas: 3  
  # 设置选择器
  selector:
    matchLabels:
      app: nginx
  # Pod模版
  template:
    # 设置元数据
    metadata:
      labels:
        app: nginx
    # 设置期望
    spec:
      # 设置容器
      containers:
      - name: nginx
        # 镜像拉取地址
        image: nginx:latest 
        ports:
        # 监听端口
        - containerPort: 80
```

&emsp;&emsp;```Kubernetes```对象描述文件遵循```YAML```格式或```JSON```格式，其中包含对象元数据（__metadata__）和期望（__spec__）。元数据包含对象名称和标签，而期望则定义了对象期望的状态。

&emsp;&emsp;元数据包含对象名称和标签，标签用于对象标识和分类。标签可以由用户自定义，也可以由```Kubernetes```自动生成。标签可以作为对象选择器（__selector__）的一部分，用于选择对象。

### 对象期望（Spec）

&emsp;&emsp;```Kubernetes```对象期望（__spec__）定义了对象期望的状态。期望包括对象类型、期望副本数量、选择器、容器、端口等。期望定义了对象应该具有的状态，包括副本数量、容器镜像、端口等。期望可以由用户自定义，也可以由```Kubernetes```自动生成。

### 状态（Status）

&emsp;&emsp;```Kubernetes```对象状态（__status__）定义了对象当前的状态。状态包括对象名称、标签、副本数量、容器镜像、端口等。状态可以由用户自定义，也可以由```Kubernetes```自动生成。

### 对象选择器（selector）

&emsp;&emsp;```Kubernetes```对象选择器（__selector__）用于选择对象。选择器可以由用户自定义，也可以由```Kubernetes```自动生成。选择器可以作为对象期望（__spec__）的一部分，用于选择对象。选择器可以匹配标签，也可以匹配名称。选择器可以匹配多个对象，也可以匹配单个对象。选择器可以匹配多个标签，也可以匹配单个标签。选择器可以匹配多个名称，也可以匹配单个名称。选择器可以匹配多个对象，也可以匹配单个对象。选择器可以匹配多个标签，也可以匹配单个标签。

## 有状态和无状态应用

&emsp;&emsp;```Kubernetes```支持两种类型的应用：有状态和无状态应用。有状态应用是指应用需要持久化存储，如数据库、消息队列等。无状态应用是指应用不需要持久化存储，如Web应用、API网关等。这两种类型很重要，因为有状态应用需要持久化存储，而无状态应用不需要持久化存储。

## 总结

&emsp;&emsp;```Kubernetes```对象是```Kubernetes```的核心对象，它们定义了```Kubernetes```集群中的各种资源。```Kubernetes```对象描述文件遵循YAML格式或JSON格式，其中包含对象元数据（__metadata__）和期望（__spec__）。元数据包含对象名称和标签，而期望则定义了对象期望的状态。```Kubernetes```对象状态（status）定义了对象当前的状态。```Kubernetes```对象选择器（__selector__）用于选择对象。```Kubernetes```对象支持多种类型，如```Pod```、```Deployment```、```ReplicaSet```等。

&emsp;&emsp;在这些对象中，```Pod```是最基本的对象，它定义了容器的运行环境。```Deployment```和```ReplicaSet```是```Pod```的集合，它们可以确保```Pod```的数量始终保持一致。```Service```是```Pod```的抽象，它定义了```Pod```的访问方式。```Ingress```是```Service```的抽象，它定义了外部访问方式。```Secret```是敏感信息的存储，它定义了敏感信息的访问方式。```ConfigMap```是配置信息的存储，它定义了配置信息的访问方式。```PersistentVolume```和```PersistentVolumeClaim```是存储的抽象，它们定义了存储的访问方式。```Volume```是存储的抽象，它定义了存储的访问方式。```VolumeMount```是存储的挂载，它定义了存储的挂载方式。```VolumeSnapshot```和```VolumeSnapshotClass```是存储的快照，它们定义了存储的快照方式。```VolumeSnapshotContent```是存储的快照内容，它定义了存储的快照内容方式。

&emsp;&emsp;通过这样一总结发现，虽然对象很多，但是其实各个对象的职责非常明确，而且各个对象之间也存在依赖关系，需要互相配合。再使用的时候，只需要按照自己的需求，选择合适的对象，就可以完成工作。
