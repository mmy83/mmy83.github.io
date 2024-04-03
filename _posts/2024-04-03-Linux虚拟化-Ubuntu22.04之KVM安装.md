---
title: Linux虚拟化-Ubuntu22.04之KVM安装
author: mmy83
date: 2024-04-03 11:14:00 +0800
categories: [IT技术, 虚拟化]
tags: [linux, kvm, 虚拟化, ubuntu]
math: true
mermaid: true
image:
  path: /images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-00.png
  lqip: data:image/webp;base64,UklGRlYAAABXRUJQVlA4IEoAAADwAQCdASoIAAYAAUAmJQBOgCPw3M3S4gAA/gslH/djrSfOkiKEGAbCB7BGfs08QepXYTrq+tLd56TLhPWIzWSMPxMTEWbwbc/AAA==
  alt: Linux虚拟化-Ubuntu22.04之KVM安装
---

## KVM介绍

&emsp;&emsp;KVM的全称是kernel base virtual machine（基于内核的虚拟机）是一个开源的系统虚拟化模块，自Linux 2.6.20之后集成在Linux的各个主要发行版本中。它使用Linux自身的调度器进行管理，所以相对于Xen，其核心源码很少。KVM已成为学术界的主流VMM之一。KVM的虚拟化需要硬件支持（如Inter VT技术或者AMD V技术），是基于硬件的完全虚拟化。

## 前期准备

&emsp;&emsp;由于KVM的虚拟化需要硬件支持，所有首先cpu要支持虚拟化并且开启（不是太老的应该都支持）。首先检查是否支持并已开启。

```shell
# 方法一：
egrep -c '(vmx|svm)' /proc/cpuinfo  # 结果大于0表示支持并且开启
```

```shell
# 方法二：
sudo apt install -y cpu-checker # 安装一个检查软件
kvm-ok # 检查是否支持
```

输出如下内容表示支持虚拟化

```console
INFO: /dev/kvm exists
KVM acceleration can be used
```

&emsp;&emsp;如果检查后发现不支持，但是cpu并不是很老（i3/i5/i7都是支持的），应该都是没有开启虚拟化，只需要进入bios开启虚拟化支持即可（不同机器设置位置不同，但都在cpu相关选项里）。

## 在 Ubuntu 22.04 上安装 KVM

&emsp;&emsp;需要根据需求安装一系列软件（可以需要的时候在安装）：

* qemu-kvm – KVM虚拟化的基本包,一个提供硬件仿真的开源仿真器和虚拟化包,但是其实ubuntu22.04已经包含了。
* virt-manager – 一款通过 libvirt 守护进程，基于 QT 的图形界面的虚拟机管理工具。
* libvirt-daemon-system – 为运行 libvirt 进程提供必要配置文件的工具。
* virtinst – 一套为置备和修改虚拟机提供的命令行工具，virt-install命令就在其中。
* libvirt-clients – 一组客户端的库和API，用于从命令行管理和控制虚拟机和管理程序,virsh命令就在其中
* bridge-utils – 一套用于创建和管理桥接设备的工具

```shell
sudo apt install -y libvirt-daemon-system virtinst libvirt-clients bridge-utils
```

&emsp;&emsp;这里用的是ubuntu-22.04-server作为宿主机系统，并没有图形界面，就没有安装virt-manager。之前安装的时候因为要安装系统，还需要VNC，但是这次发现不需要VNC就可以安装了。系统自带qemu-system-x86,安装的时候提示不需要qemu-kvm。安装了bridge-utils，这是网桥工具，要想让虚拟机被其他机器访问，还是需要用它来建立网桥的。

## 创建虚拟机

&emsp;&emsp;由于使用了ubuntu-22.04-server作为宿主机系统，没有图形界面，也就不能用virt-manager来创建虚拟机，这里使用virt-install命令来创建虚拟机

```shell
virt-install \
  --connect=qemu:///system \
  --virt-type=kvm \
  --name=ubuntu-2204-server \
  --vcpus=8 \
  --memory=16384 \
  --location=/opt/ubuntu-22.04.4-live-server-amd64.iso \
  --disk path=/opt/kvm/vm/ubuntu-2204-server.qcow2,size=30,format=qcow2 \
  --network network=default \
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
> * --network network=default：虚拟机网络设置，如果想让虚拟机可以对外访问，这里可以使用网桥。
> * --graphics none：用来配置图形显示，之前用vnc的时候就在这 --graphics vnc,port=5901,listen=0.0.0.0，如果想通过console，这里最好关掉，否则不会自动通过console安装，只能使用virsh console 命令来连接。
> * --extra-args='console=ttyS0'：扩展参数，这里配置了虚拟机终端，这里是不需要vnc的关键。
>
> 除此之外，还有很多参数，可以通过-h获得帮助信息
{: .prompt-tip }

&emsp;&emsp;虚拟机系统安装的过程就不累述了。但是之前我都是用VNC来安装系统的，这次发现并不需要，我还测试了ubuntu-22.04.4-live-server-amd64和CentOS-7-x86_64-Minimal-2009，发现都可以正常安装。如图：

### ubuntu-22.04.4-live-server-amd64

![ubuntu选择终端](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-01.jpg)

&emsp;&emsp;会发现他有三个模式可供选择，选择任何一个都可以，下面是截图：

1、Continue in rich mode

![Continue in rich mode](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-02.jpg)

2、Continue in basic mode

![Continue in basic mode](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-03.jpg)

3、View SSH instructions

![View SSH instructions](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-04.jpg)

![View SSH instructions连接后](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-05.jpg)

### CentOS-7-x86_64-Minimal-2009

&emsp;&emsp;centos安装的时候并没有提供像ubuntu那样的选择，而是给了一个比较简单的可以通过命令行操作的安装界面，但是这并不影响安装。如图：

![CentOS-7-x86_64-Minimal-2009安装](/images/2024-04-03/Linux虚拟化-Ubuntu22.04之KVM安装/Linux虚拟化-Ubuntu22.04之KVM安装-06.jpg)

## 虚拟机管理virsh

&emsp;&emsp;当系统安装完之后，就可以来管理虚拟机了。简单看几个命令：

### 查看虚拟机

```shell
virsh list --all
```

### 开机

```shell
virsh start vm1
```

### 关机

```shell
virsh shutdown vm1
```

### 重启

```shell
virsh reboot vm1
```

### 连接到访客控制台

```shell
virsh console vm1
```

### 断电

```shell
virsh destroy vm1
```

### 虚拟机信息

```shell
virsh dominfo vm1
```

&emsp;&emsp;其实我和virsh之间就差一个```-h```好的翻译。

```shell
 virsh -h
```

> 注:
>
> 1、现在虚拟机还不能被主机之外的机器访问，需要配置网桥。
>
> 2、由于每次都要安装系统太麻烦，可以采用克隆的方式。
{: .prompt-tip }
