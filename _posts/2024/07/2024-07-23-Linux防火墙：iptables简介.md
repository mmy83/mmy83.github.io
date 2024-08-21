---
title: Linux防火墙：iptables简介
author: mmy83
date: 2024-07-23 15:56:00 +0800
categories: [IT技术, "Linux"]
tags: [IT技术, "linux", '防火墙', iptables]
math: true
mermaid: true
image:
  path: /images/2024/07/2024-07-23/Linux防火墙：iptables简介/Linux防火墙：iptables简介-00.png
  lqip: data:image/webp;base64,UklGRl4AAABXRUJQVlA4IFIAAAAQAgCdASoIAAUAAUAmJbACdLoAAx8Tj8KAAP74q60fl6a8RxKD2v61ES2FjZcndX+vTfve5UyxUuH+rNRuz2xAJ//lOpHgixhK7d/mIfKbAAAA
  alt: Linux防火墙：iptables简介
---

## 前言

&emsp;&emsp;其实我对防火墙的研究很少，大多时间都是遇到了就网上搜一下资料，现在有了AI就更简单了，都不用费力找了。但是我还是希望自己能会，毕竟省去了找的时间，所有前面写了[Linux防火墙：firewall-cmd命令介绍](/posts/Linux防火墙-firewall-cmd命令介绍/)和[Linux防火墙：firewalld-RichRule](/posts/linux防火墙-firewalld-richrule/)两篇博文，索性这次就把```iptables```也写一下，但是要想一篇博文把```iptables```讲明白还挺难的，所有我加入了很多我的理解，可能有不对的地方望指正。。

## 介绍

&emsp;&emsp;```iptables```和之前提到的```firewall-cmd```一样，都只是管理防火墙规则的配置工具，但是```iptables```更早更长用一些，现在很多服务器还是使用它来配置规则。他对规则的管理其实很简单，他把规则分为五表五链，即五张表，五个链，链里面是规则。用我的理解是：表提现了功能，链提现了数据流向，规则是流动上的放行还是不放行的条件。毕竟防火墙的核心是 **netfilter** 从名字上看就是一个对网络数据过滤的功能。现在就可以理解为：iptables就是通过增/删/改/查来配置表、链、检查项、是否放行的规则，让 **netfilter** 来安装这个规则来执行过滤。从这个角度讲，其实我更喜欢iptables这种直观的方式。

## 五表(table)

- raw：关闭启用的连接跟踪机制，加快封包穿越防火墙速度（跟踪数据包）

- mangle：修改数据标记位规则表（标记数据包）

- nat：network address translation 地址转换，公网和私网的地址转换

- filter：过滤规则表，根据预定义的规则过滤符合条件的数据包，默认表（允许、拒绝）

- security：用于强制访问控制（MAC）网络规则，由Linux安全模块（如SELinux）实现 (了解)

&emsp;&emsp;优先级由高到低的顺序为：security -->raw-->mangle-->nat-->filter

&emsp;&emsp;看到上面的五表，相信就理解为啥我说表提现了功能了，这五张表分明代表五个不同的功能。那么使用的时候，你只要选择对表，就可以了。但是这里其实也不是那么简单。

## 五链(chain)

- input： 处理入站数据

- output：处理出站数据

- forward：转发数据

- prerouting：处理路由选择前数据

- postrouting：处理路由选择后数据

![五链](/images/2024/07/2024-07-23/Linux防火墙：iptables简介/Linux防火墙：iptables简介-01.png)

&emsp;&emsp;这就是我为啥说链代表数据流向。五链是网络数据在内与外之间流通的几个检查点，就像五个哨卡。

## 表和链的关系

&emsp;&emsp;上面说了，五链就像五个哨卡，设立哨卡自然是为了检查，而检查的规则又和上面五表来提供的功能有关。这就是我说的，选对表其实也不容易。我们经常说五表五链，其实我更愿意说是五链五表。我们分别从五表五链和五链五表来看看这两者的对应关系。

### 五表五链

|表|链|说明|
|--|--|--|
|raw|PREROUTING，OUTPUT||
|mangle|PREROUTING，INPUT，FORWARD，OUTPUT，POSTROUTING||
|nat|PREROUTING，OUTPUT，POSTROUTING|centos7中还有INPUT，centos6中没有|
|filter|INPUT，FORWARD，OUTPUT||
|security|INPUT，FORWARD，OUTPUT||

### 五链五表

|链|表|说明|
|--|--|--|
|PREROUTING|raw表，mangle表，nat表||
|INPUT|mangle表，filter表|centos7中还有nat表，centos6中没有|
|FORWARD|mangle表，filter表||
|OUTPUT|raw表mangle表，nat表，filter表||
|POSTROUTING|mangle表，nat表||

&emsp;&emsp;通过这样一列，是不是发现还是哨卡具有啥功能更好理解。

## 规则

&emsp;&emsp;这里加入一些我个人理解，我们的哨卡有了，功能有了，下面就是像安检一样检查，检查什么呢？网卡、端口、来源、目标、协议、甚至可以检查内容，无非也就是这几项。

### 通用语法

```console
#通用语法，不指定默认filter表，因为filter表是最常用的
iptables [-t 表] -管理选项 [链] [通用规则匹配] [-j 控制类型]
```

### 管理选择

|管理选项|用法示例|
|--|--|
|-A|在指定链末尾追加一条 iptables -A INPUT （操作）|
|-I|在指定链中插入一条新的，未指定序号默认作为第一条 iptables -I INPUT （操作）|
|-P|指定默认规则 iptables -P OUTPUT ACCEPT （操作）|
|-D|删除 iptables -t nat -D INPUT 2 （操作）|
|-p|服务名称 icmp tcp|
|-R|修改、替换某一条规则 iptables -t nat -R INPUT （操作）|
|-L|查看 iptables -t nat -L （查看）|
|-n|所有字段以数字形式显示（比如任意ip地址是0.0.0.0而不是anywhere，比如显示协议端口号而不是服务名）iptables -L -n,iptables -nL,iptables -vnL（查看）|
|-v|查看时显示更详细信息，常跟-L一起使用 （查看）|
|--line-number|规则带编号 iptables -t nat -L -n --line-number /iptables -t nat -L --line-number|
|-F|清除链中所有规则 iptables -F （操作）|
|-N|新加自定义链|
|-X|清空自定义链的规则，不影响其他链 iptables -X|
|-Z|清空链的计数器（匹配到的数据包的大小和总和）iptables -Z|
|-S|看链的所有规则或者某个链的规则/某个具体规则后面跟编号|

### 通用匹配规则

- 协议匹配: -p协议名

- 地址匹配: -s 源地址、-d目的地址。可以是IP、网段、域名、空(任何地址)

- 接口匹配: -i入站网卡、-o出站网卡  

### 控制类型

- ACCEPT：允许数据包通过

- DROP：直接丢弃数据包，不给出任何回 应信息 

- REJECT：拒绝数据包通过，必要时会给数据发送端一个响应信息

- LOG：在/var/log/messages 文件中记录日志信息，然后将数据包传递给下一条规则

- SNAT:修改数据包的源地址

- DNAT:修改数据包的目的地址

- MASQUERADE:伪装成一个非固定公网IP地址

![iptables命令](/images/2024/07/2024-07-23/Linux防火墙：iptables简介/Linux防火墙：iptables简介-02.png)

&emsp;&emsp;剩下的就简单了，按照你的需求连线即可，但是这里要注意表和链的对应关系，你不能让一个哨卡使用自己不具备的功能来检查。

## 结束

&emsp;&emsp;其实除了上面这些，还有自定义链，其实就是自定义一个规则组合，这个规则组合可以当控制类型使用。还有扩展模块，通过扩展模块可以实现更多的功能。

## 参考

- [朱双印的个人博客-iptables详解](https://www.zsythink.net/archives/category/运维相关/iptables)：第一次完整的看iptables的时候就是看的他的博客

- [Linux防火墙与iptables五表五链规则介绍](https://blog.csdn.net/qq_64612585/article/details/135972548)：文中绝大部分内容都是从这篇文章来的，仅仅增加了点自己的理解。

- [iptables查看默认策略 iptables默认规则](https://blog.51cto.com/u_16099296/10244580)
