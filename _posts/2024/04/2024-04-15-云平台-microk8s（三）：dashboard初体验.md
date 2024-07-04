---
title: 云平台-microk8s（三）：dashboard初体验
author: mmy83
date: 2024-04-18 17:49:00 +0800
categories: [专题, microk8s]
tags: [云平台, k8s, microk8s, Kubernetes, 高可用]
math: true
mermaid: true
image:
  path: /images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-00.png
  lqip: data:image/webp;base64,UklGRkQAAABXRUJQVlA4IDgAAADQAQCdASoIAAQAAUAmJZQCdAEUo1OUAAD+/dJ8SDq3fp69s//Rbxzm6AO0QdifipiHE6NZnfAAAA==
  alt: 云平台-microk8s（三）：dashboard初体验
---

## 介绍

&emsp;&emsp;MicroK8s 和完整版本的 Kubernetes 一样，能够使用 Dashboard ，通过可视化的方式来检查各个组件的运行状况。而且对于 MicroK8s 来说，开启 Dashboard 也很简单。然而开启是简单，但是真的要能访问还是要折腾一番的。

## 开启 Dashboard 组件

```shell
# 开启 Dashboard 组件
microk8s enable dashboard
```

&emsp;&emsp;等一会，查看一下平台情况，如果没有报错 Dashboard 组件就已经开启了，组件开启在 [云平台-microk8s（一）：单机部署](/posts/云平台-microk8s-一-单机部署/) 中已经讲过，这里不再累述。

```console
microk8s@microk8s-01:~$ microk8s kubectl get all --all-namespaces
NAMESPACE            NAME                                             READY   STATUS    RESTARTS       AGE
container-registry   pod/registry-6c9fcc695f-559lh                    1/1     Running   0              6d5h
kube-system          pod/calico-kube-controllers-77bd7c5b-9cqg6       1/1     Running   0              6d20h
kube-system          pod/calico-node-6s9k2                            1/1     Running   0              6d4h
kube-system          pod/calico-node-dlxj5                            1/1     Running   0              6d4h
kube-system          pod/calico-node-lmt28                            1/1     Running   2 (6d4h ago)   6d4h
kube-system          pod/coredns-864597b5fd-lhlfk                     1/1     Running   0              6d20h
kube-system          pod/dashboard-metrics-scraper-5657497c4c-5kt94   1/1     Running   0              6d16h
kube-system          pod/hostpath-provisioner-756cd956bc-jv4jr        1/1     Running   32 (23h ago)   6d5h
kube-system          pod/kubernetes-dashboard-54b48fbf9-qtj4s         1/1     Running   0              6d16h
kube-system          pod/metrics-server-848968bdcd-8lqff              1/1     Running   0              6d16h

NAMESPACE            NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
container-registry   service/registry                    NodePort    10.152.183.79    <none>        5000:32000/TCP           6d5h
default              service/kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                  6d20h
kube-system          service/dashboard-metrics-scraper   ClusterIP   10.152.183.98    <none>        8000/TCP                 6d16h
kube-system          service/kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   6d20h
kube-system          service/kubernetes-dashboard        ClusterIP   10.152.183.106   <none>        443/TCP                  6d16h
kube-system          service/metrics-server              ClusterIP   10.152.183.215   <none>        443/TCP                  6d16h

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node   3         3         3       3            3           kubernetes.io/os=linux   6d20h

NAMESPACE            NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
container-registry   deployment.apps/registry                    1/1     1            1           6d5h
kube-system          deployment.apps/calico-kube-controllers     1/1     1            1           6d20h
kube-system          deployment.apps/coredns                     1/1     1            1           6d20h
kube-system          deployment.apps/dashboard-metrics-scraper   1/1     1            1           6d16h
kube-system          deployment.apps/hostpath-provisioner        1/1     1            1           6d5h
kube-system          deployment.apps/kubernetes-dashboard        1/1     1            1           6d16h
kube-system          deployment.apps/metrics-server              1/1     1            1           6d16h

NAMESPACE            NAME                                                   DESIRED   CURRENT   READY   AGE
container-registry   replicaset.apps/registry-6c9fcc695f                    1         1         1       6d5h
kube-system          replicaset.apps/calico-kube-controllers-77bd7c5b       1         1         1       6d20h
kube-system          replicaset.apps/coredns-864597b5fd                     1         1         1       6d20h
kube-system          replicaset.apps/dashboard-metrics-scraper-5657497c4c   1         1         1       6d16h
kube-system          replicaset.apps/hostpath-provisioner-756cd956bc        1         1         1       6d5h
kube-system          replicaset.apps/kubernetes-dashboard-54b48fbf9         1         1         1       6d16h
kube-system          replicaset.apps/metrics-server-848968bdcd              1         1         1       6d16h
microk8s@microk8s-01:~$
```

![开启dashboard](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-01.png)

## 配置访问

&emsp;&emsp;上面的操作只是开启了 Dashboard 组件，并不能直接访问。而且为了安全，访问还要有访问秘钥。而且开启访问还不只一种方式。

### 获取秘钥

```console
microk8s@microk8s-01:~$ microk8s kubectl -n kube-system get secret |grep dashboard-token | cut -d " " -f1| xargs microk8s kubectl -n kube-system describe secret
Name:         microk8s-dashboard-token
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 44095ec3-86d4-465e-b53c-9c142f95c10b

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1123 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjlOZ185UnFxLU9BdUh5RHRIeXI5ODNMQ3ZMeU1tS0dYU1F4UENWNndEMWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJtaWNyb2s4cy1kYXNoYm9hcmQtdG9rZW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQ0MDk1ZWMzLTg2ZDQtNDY1ZS1iNTNjLTljMTQyZjk1YzEwYiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTpkZWZhdWx0In0.dNXfnT2D2lnPl-drvWVfS7k3NL-qKjM5hzHfIRUSwIeROjy5NiMcBSsDEbfu_ACOFyjvRiAF8sqqu75CtQGf-K3SFN_X17BsI1IMPex42re26LFiQZpb-5WVi6DKOWuo_HVhkqfrNPxfojsIwoBDsRA2vSqmwdP-LdUali7E8MMmtbHWf-jHQkND9iSkkwJ9rGZWTy8A3WeNFlidEVP3RkYduCGXW8HEiOr64x9eHbCER9C0hFbOWY7Uz-bvQWyzzRXUIz3uQ1hv935IRZS0oEpcN9E_LMU3s8ebltYPKAJkyMrAA8xJP7BUi1oo0fXRAQfx2hF9h8dXnZKOxjzBHg
microk8s@microk8s-01:~$
```

![dashboard的token](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-02.png)

### 代理方式(本机)

```shell
microk8s.kubectl proxy --accept-hosts=.* --address=0.0.0.0 --port=8001
```

&emsp;&emsp;开启代理后，会提示 ```Starting to serve on [::]:8001``` ，访问 ```http://192.168.1.248:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/login``` ,结果发现这样已经被认为不安全，不能登录了。

![代理失败](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-03.png)

&emsp;&emsp;由于安全问题，代理方式只能用于本地（localhost）登陆，经测试，即便加了参数,一样失败。

&emsp;&emsp;访问地址很长，但是具有规则：```/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/login```，分成三段来看：

1、```/api/v1```：接口版本

2、```/namespaces/kube-system```：命名空间

3、```/services/https:kubernetes-dashboard:/proxy/#/login```：服务及访问路径

### 端口映射方式

```shell
microk8s kubectl port-forward -n kube-system --address=0.0.0.0 service/kubernetes-dashboard 10443:443
```

&emsp;&emsp;执行命令后，会提示 ```Forwarding from 0.0.0.0:10443 -> 8443``` ,访问 ```https://192.168.1.248:10443/``` 即可登录。

![端口映射登录](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-04.png)

![登录后](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-05.png)

> 注:
>
> 登录后可能就是上面的样子，因为还没有部署应用，可以通过切换命令空间到 ```kube-system``` 体验一下。
{: .prompt-tip }

### 创建 NodePort 服务方式

```shell
vi kubernetes-dashboard-zdy.yml
```

```yml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-zdy
  namespace: kube-system
spec:
  type: NodePort   # NodePort 类型
  ports:
    - port: 443       # 这里是kubernetes-dashboard端口
      nodePort: 30443   # 这里是访问端口,这个端口的范围是 30000-32767，不配置会自动分配
      targetPort: 8443   # 这个端口是后端 Pod 的端口
  selector:
    k8s-app: kubernetes-dashboard
```

```console
# 创建服务
microk8s@microk8s-01:~$ microk8s kubectl create -f kubernetes-dashboard-zdy.yml
service/kubernetes-dashboard-zdy created
microk8s@microk8s-01:~$ microk8s kubectl get service --all-namespaces
NAMESPACE            NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
container-registry   registry                    NodePort    10.152.183.79    <none>        5000:32000/TCP           6d7h
default              kubernetes                  ClusterIP   10.152.183.1     <none>        443/TCP                  6d22h
kube-system          dashboard-metrics-scraper   ClusterIP   10.152.183.98    <none>        8000/TCP                 6d18h
kube-system          kube-dns                    ClusterIP   10.152.183.10    <none>        53/UDP,53/TCP,9153/TCP   6d22h
kube-system          kubernetes-dashboard        ClusterIP   10.152.183.106   <none>        443/TCP                  6d18h
kube-system          kubernetes-dashboard-zdy    NodePort    10.152.183.107   <none>        443:30443/TCP            2m36s
kube-system          metrics-server              ClusterIP   10.152.183.215   <none>        443/TCP                  6d18h
microk8s@microk8s-01:~$
```

![创建NodePort服务成功](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-06.png)

&emsp;&emsp;通过访问```https://192.168.1.248:30443``` 即可访问 Dashboard。

> 注:
>
> 其实不一定要创建新的，系统中已经有一个```kubernetes-dashboard```服务了，只不过是```type: ClusterIP```，只需要修改为```type: NodePort```即可。但是这样修改后，就不能用端口映射的方式了，所有这里还是重新创建新的服务。
> 如果这里修改了系统中的```kubernetes-dashboard```服务，下面的通过 Ingress 访问的方式也会受影响。
{: .prompt-tip }

### 通过Ingress方式

#### Ingress介绍

![ingress图](/images/2024/04/2024-04-18/云平台-microk8s（三）：dashboard初体验/云平台-microk8s（三）：dashboard初体验-07.png)

&emsp;&emsp;这是我最喜欢的方式，也是我认为最好的方式。Ingress 使用开源的反向代理负载均衡器来实现对外暴漏服务。

#### 开启Ingress组件

```console
# 开启 ingress
microk8s@microk8s-01:~$ microk8s enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
microk8s@microk8s-01:~$
```

#### 创建 Dashboard 的SSL证书

```shell
# mmy83.online.key私钥和mmy83.online.csr文件（我弄了一个自己域名的通配符证书，这一步就省了）
openssl genrsa -des3 -passout pass:123456 -out mmy83.online.pass.key 2048
openssl rsa -passin pass:123456 -in mmy83.online.pass.key -out mmy83.online.key
rm mmy83.online.pass.key
openssl req -new -key mmy83.online.key -out mmy83.online.csr
# SSL证书
openssl x509 -req -sha256 -days 365 -in mmy83.online.csr -signkey mmy83.online.key -out mmy83.online.crt
rm mmy83.online.csr
mkdir certs && mv mmy83.online.key mmy83.online.crt certs
```

```shell
# 删除旧的
microk8s kubectl delete secret  kubernetes-dashboard-certs -n kube-system
microk8s kubectl delete secret  kubernetes-dashboard-key-holder -n kube-system
# 上传证书
microk8s kubectl create secret generic  kubernetes-dashboard-certs  --from-file=tls.crt=certs/mmy83.online.crt --from-file=tls.key=certs/mmy83.online.key  -n kube-system
```

```console
#查看证书
microk8s@microk8s-01:~$ microk8s kubectl describe secret/kubernetes-dashboard-certs -n kube-system
Name:         kubernetes-dashboard-certs
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
tls.crt:  4200 bytes    # 这个名字还要用
tls.key:  1675 bytes    # 这个名字还要用
```

#### 编辑 Dashboard 的 deployments

```shell
microk8s kubectl edit deployments kubernetes-dashboard -n kube-system
```

```console
# 找到并增加
containers:
  args:
    - --auto-generate-certificates=false    #不让他自动创建ssl文件使用我们给的，不能删除，删除则不使用ssl
    - --namespace=kube-system
    - --token-ttl=3600                  # 新增
    - --bind-address=0.0.0.0            # 新增
    - --tls-cert-file=tls.crt     # 新增 ,这个名字就是证书里的名字
    - --tls-key-file=tls.key      # 新增 ,这个名字就是证书里的名字

# 退出就保存并应用更新了
```

#### 创建 Dashboard 的 Ingress

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    ingress.kubernetes.io/ssl-passthrough: "true"   # 开启https 透传
    nginx.org/ssl-backends: "kubernetes-dashboard"
    kubernetes.io/ingress.allow-http: "false"
    nginx.ingress.kubernetes.io/secure-backends: "true" # 后端backend 使用https
    # 开启use-regex，启用path的正则匹配
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  name: dashboard-ingress
  namespace: kube-system
spec:
  ingressClassName: nginx
  tls:
  - hosts:
      - dashboard.mmy83.online
    secretName: kubernetes-dashboard-certs
  rules:
  - host: dashboard.mmy83.online
    http:  
      paths:  
      - path: /  
        pathType: Prefix  
        backend:  
          service:  
            name: kubernetes-dashboard
            port:  
              number: 443
```

```console
microk8s@microk8s-01:~$ microk8s kubectl create -f dashboard-ingress.yml
ingress.networking.k8s.io/dashboard-ingress created
microk8s@microk8s-01:~$
```

#### 修改 hosts

```console
# 本机hosts中添加
192.168.1.248  dashboard.mmy83.online
```

&emsp;&emsp;现在通过访问```https://dashboard.mmy83.online``` 就可访问 Dashboard了。

## 结束语

&emsp;&emsp;通过这几篇文章完成了 MicroK8s 集群的部署。可以说 MicroK8s 和完整版本的 Kubernetes 基本是一样的，之所以使用 MicroK8s 作为学习 Kubernetes 的开始，主要还是因为他部署简单，虽然是万事开头难，然后一直难，但是如果开头有简单的方法，还是可以增加信心，让学习更顺利。
