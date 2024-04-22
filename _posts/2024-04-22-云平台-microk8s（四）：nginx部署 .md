---
title: 云平台-microk8s（四）：nginx部署
author: mmy83
date: 2024-04-22 09:39:00 +0800
categories: [专题, microk8s]
tags: [云平台, k8s, microk8s, Kubernetes, 高可用, 部署, nginx]
math: true
mermaid: true
image:
  path: /images/2024-04-22/云平台-microk8s（四）：nginx部署/云平台-microk8s（四）：nginx部署-00.png
  lqip: data:image/webp;base64,UklGRmgAAABXRUJQVlA4IFwAAADwAQCdASoIAAgAAUAmJYwCdAEfbnZfUwAA/vhGrE3dvjTjajhR+Jv3Kr3TZoLZaTp1vdTiGrL6InzXClQ/VEQI4yKRSsxg3WjG6syEYtz7Vsn68OH4BjVDnwAAAA==
  alt: 云平台-microk8s（四）：nginx部署
---

## 前言

&emsp;&emsp;通过前几篇文章完成了 MicroK8s 的部署。可以说 MicroK8s 和完整版本的 Kubernetes 基本是一样的，之所以使用 MicroK8s 主要是因为他部署简单。多年的学习经验告诉我，万事开头难，然后一直难，但是终究还是开头最难，而且大部分人学习一个新东西开头遇到困难都很难找到解决方案，导致无法继续下去。

&emsp;&emsp;既然 MicroK8s 部署好了，就来部署一个简单的 nginx 应用并访问，仅此来体验一下部署的过程。

## 部署nginx应用

&emsp;&emsp;首先我们需要创建一个3副本的 Deployment ，然后在创建一个 Service 来提供对外访问。同时通过删除一个副本来看看nginx应用的高可用。

### Deployment

```shell
vi nginx-deployment.yaml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # 设置副本数量
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest #镜像拉取地址
        ports:
        - containerPort: 80 #监听端口
```

```shell
microk8s kubectl apply -f nginx-deployment.yaml
```

```console
# 查看default命名空间下所有资源
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-4rjvl   1/1     Running   0          13m
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Running   0          13m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Running   0          13m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           13m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         3       13m
microk8s@microk8s-01:~$
```

&emsp;&emsp;从上面结果可以看出，他创建了三类资源：```pod```、```deployment```、```replicaset```。

* __Pod__：Kubernetes 系统中可以创建和管理的最小单元，是资源对象模型中由用户创建或部署的最小资源对象模型，也是在 Kubernetes 上运行容器化应用的资源对象，其它的资源对象都是用来支撑或者扩展 Pod 对象功能的。Pod与容器是不同的，一个Pod可以包含一个或多个容器。当前情况下在yml文件中制定了3个副本，可以认为每一个pod就是一个nginx的容器。而系统创建了三个pod，即三个副本。
* __Replicaset__：控制管理Pod，保证指定数量的Pod运行，并支持pod数量变更，镜像版本变更，三个副本就是由他来控制的。
* __Deployment__：管理Replicaset，并支持滚动升级，版本回退。

&emsp;&emsp;我们只是创建了一个 Deployment ，系统却做了这么多，这三者关系为： __```Deployment``` 控制 ```Replicaset``` 控制 ```Pod```__ ，可以看到其实nginx应用只在Pod里，而其他的都是为了更好地管理Pod

> 注：
> 如果已经用平台创建过其他服务，这里的结果可能会多一些其他的资源，可以通过创建时间进行区分。
>
> 这里创建的时候没有指定namespace，所有是default。
{: .prompt-tip }

### Service

```shell
vi nginx-service.yaml
```

```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080 #nortPort开在主机的端口,范围30000-32767,不指定会随机分配，建议不指定，避免冲突导致错误
  type: NodePort
```

```shell
microk8s kubectl apply -f nginx-service.yaml
```

```console
# 查看default命名空间下所有资源
microk8s@microk8s-01:~$  microk8s kubectl get all -n default
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-4rjvl   1/1     Running   0          55m
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Running   0          55m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Running   0          55m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   56s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           55m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         3       55m
microk8s@microk8s-01:~$
```

&emsp;&emsp;引入 ```Service``` 主要是解决 Pod 的动态变化，通过创建 Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址，并且将请求负载分发到后端的各个容器应用上。若提供服务的容器应用是分布式，所以存在多个 pod 副本，而 Pod 副本数量可能在运行过程中动态改变，比如水平扩缩容，或者服务器发生故障 Pod 的 IP 地址也有可能发生变化。当 pod 的地址端口发生改变后，客户端再想连接访问应用就得人工干预，很麻烦，这时就可以通过 service 来解决问题。

## 访问

&emsp;&emsp;现在可以通过集群中任何一台机器的ip对外提供服务了，[http://192.168.1.248:30080/]，[http://192.168.1.249:30080/]，[http://192.168.1.250:30080/]都可以.

## 高可用

### 删除Pod

&emsp;&emsp;上面创建了一个三副本的nginx应用，通过删除其中的一个或两个Pod，会发现他一样可以正常访问，并在一定时间后继续维持三个副本。

```console
# 查看当前状态
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-4rjvl   1/1     Running   0          66m
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Running   0          66m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Running   0          66m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   11m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           66m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         3       66m
microk8s@microk8s-01:~$

# 删除一个pod nginx-deployment-7c79c4bf97-4rjvl
microk8s@microk8s-01:~$ microk8s kubectl delete pod/nginx-deployment-7c79c4bf97-4rjvl
pod "nginx-deployment-7c79c4bf97-4rjvl" deleted

# 再次查看，发现nginx-deployment-7c79c4bf97-4rjvl没了，但是有一个新pod正在创建。现在访问nginx也正常。
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS              RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-8rmr7   0/1     ContainerCreating   0          27s
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Running             0          69m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Running             0          69m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   14m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/3     3            2           69m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         2       69m
microk8s@microk8s-01:~$

# 一段时间后，容器创建成功，又维持到了3个pod
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-8rmr7   1/1     Running   0          2m
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Running   0          71m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Running   0          71m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   16m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           71m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         3       71m
microk8s@microk8s-01:~$

```

&emsp;&emsp;会发现不管怎么删除Pod，最终都会自动维持3个Pod。

### 删除Replicaset

```console
# 删除 replicaset
microk8s@microk8s-01:~$ microk8s kubectl delete replicaset.apps/nginx-deployment-7c79c4bf97
replicaset.apps "nginx-deployment-7c79c4bf97" deleted

# 查看状态，注意pod、deployment、replicaset的状态值
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-8rmr7   1/1     Terminating   0          6m3s
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Terminating   0          75m
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Terminating   0          75m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   20m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           75m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         0         0       6s

# 再次查看
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-8rmr7   1/1     Terminating   0          6m16s
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Terminating   0          75m
pod/nginx-deployment-7c79c4bf97-kmmbq   0/1     Pending       0          6s
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Terminating   0          75m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   20m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   0/3     0            0           75m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         0         0       19s

# 再次查看
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS        RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-8rmr7   1/1     Terminating   0          6m18s
pod/nginx-deployment-7c79c4bf97-cs2tl   0/1     Pending       0          3s
pod/nginx-deployment-7c79c4bf97-kk572   1/1     Terminating   0          75m
pod/nginx-deployment-7c79c4bf97-kmmbq   0/1     Pending       0          8s
pod/nginx-deployment-7c79c4bf97-lwt4z   1/1     Terminating   0          75m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   20m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   0/3     0            0           75m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         0         0       21s

# 最终
microk8s@microk8s-01:~$ microk8s kubectl get all -n default
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7c79c4bf97-9lfzb   1/1     Running   0          47s
pod/nginx-deployment-7c79c4bf97-cs2tl   1/1     Running   0          47s
pod/nginx-deployment-7c79c4bf97-kmmbq   1/1     Running   0          52s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort    10.152.183.90   <none>        80:30080/TCP   21m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           76m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7c79c4bf97   3         3         3       65s
microk8s@microk8s-01:~$

```

&emsp;&emsp;如果删除Replicaset将导致其下的所有Pod都将重新创建。而且Replicaset也会被创建，但是和Pod不同的是，Pod的名字会变，而Replicaset的名字不会变，通过连续执行查看命令可以看到Replicaset不存在的过程，然后很快就又出来了。

> 上面的每一步操作都是通过命令来创建的，其实也是可以通过 Dashboard 来达到相同的目的。因为是图形操作界面，使用上更加简单方便，这里不做介绍。
{: .prompt-tip }
