---
title: Linux虚拟化-Ubuntu22.04之KVM桥接网络
author: mmy83
date: 2024-04-07 14:50:00 +0800
categories: [IT技术, 虚拟化]
tags: [linux, kvm, 虚拟化, ubuntu, 桥接]
math: true
mermaid: true
image:
  path: /images/2024/04/2024-04-07/Linux虚拟化-Ubuntu22.04之KVM桥接网络/Linux虚拟化-Ubuntu22.04之KVM桥接网络-00.png
  lqip: data:image/webp;base64,UklGRmwAAABXRUJQVlA4IGAAAADwAQCdASoIAAgAAUAmJbACdDBIgZCDPRgA4n3zyoQ1jlXjboz9dBDg01D60cSfMFrisAyF/9f6vbTmYGA63tbPfC/E29klONUeeWGn/4Ys1s3v+Nzj/sp4Zlmg89usQAA=
  alt: Linux虚拟化-Ubuntu22.04之KVM桥接网络
---

## 前言

&emsp;&emsp;在用virt-install创建虚拟机时，可以通过设置桥接网络来实现虚拟机被宿主机之外的机器访问。要设置桥接网络首先要在宿主机创建网桥，然后通过为虚拟机指定网桥来实现桥接网络。

## 创建网桥

&emsp;&emsp;在宿主机创建网桥，并为网桥指定静态ip地址。

### 方法一：手动修改配置文件

```shell
# vi /etc/netplan/00-installer-config.yaml

network:
  version: 2
  ethernets:
    eno1:
      dhcp4: false
      dhcp6: false
  bridges:
    br0:
      interfaces: [eno1]
      dhcp4: false
      addresses: [192.168.1.245/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [114.114.114.114, 202.106.0.20]
      parameters:
        stp: false
      dhcp6: false
```

```shell
# 应用修改
netplan apply
```

```shell
#修改后
# ifconfig
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.245  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::5c03:30ff:fea0:91ab  prefixlen 64  scopeid 0x20<link>
        ether 5e:03:30:a0:91:ab  txqueuelen 1000  (Ethernet)
        RX packets 771022  bytes 75246451 (75.2 MB)
        RX errors 0  dropped 3702  overruns 0  frame 0
        TX packets 57477  bytes 13754799 (13.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 7c:10:c9:9f:f9:2a  txqueuelen 1000  (Ethernet)
        RX packets 8701495  bytes 5061954256 (5.0 GB)
        RX errors 0  dropped 8594  overruns 0  frame 0
        TX packets 558286  bytes 56944955 (56.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 16  memory 0xa1300000-a1320000
```

### 方法二：使用 virsh 命令

```shell
#创建网桥
virsh iface-bridge eno1 br0
# 查看网桥,需要安装 bridge-utils
brctl show
```

## 使用网桥创建虚拟机

```shell
virt-install \
  --connect=qemu:///system \
  --virt-type=kvm \
  --name=ubuntu-2204-server \
  --vcpus=4 \
  --memory=8192 \
  --location=/opt/ubuntu-22.04.4-live-server-amd64.iso \
  --disk path=/opt/kvm/vm/ubuntu-2204-server.qcow2,size=30,format=qcow2 \
  --network bridge=br0 \
  --graphics none \
  --extra-args='console=ttyS0' \
  --force
```

> 说明：
>
> * --name=ubuntu-2204-server：虚拟机名，我这里虚拟机一样安装的ubuntu-2204-server
> * --vcpus=8：虚拟机cpu数
> * --memory=16384：虚拟机内存，单位M
> * --location=/opt/ubuntu-22.04.4-live-server-amd64.iso：系统镜像
> * --disk path=/opt/kvm/vm/ubuntu-2204-server.qcow2,size=30,format=qcow2：虚拟机磁盘路径、大小（单位G）、磁盘格式
> * --network bridge=br0：虚拟机网络设置，为了可以被外部访问，这里使用网桥。
> * --graphics none：用来配置图形显示，之前用vnc的时候就在这 --graphics vnc,port=5901,listen=0.0.0.0，如果想通过console，这里最好关掉，否则不会自动通过console安装，只能使用virsh console 命令来连接。
> * --extra-args='console=ttyS0'：扩展参数，这里配置了虚拟机终端，这里是不需要vnc的关键。
>
> 除此之外，还有很多参数，可以通过-h获得帮助信息
{: .prompt-tip }

## 修改已存在虚拟机使用网桥

```shell
# 宿主机执行编辑虚拟机，执行命令后会让选择编辑器
virsh edit ubuntu-2204-server
```

```shell
# 定位到网络配置
<interface type='network'>
    <source network='default'/>
# 改为
<interface type='bridge'>
    <source bridge='br0'/>

# 重启虚拟机生效
```

&emsp;&emsp;通过上面的操作，可以让虚拟机被宿主机同一个网络的机器访问。
