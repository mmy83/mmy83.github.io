---
title: Linux防火墙：firewall-cmd命令介绍
author: mmy83
date: 2024-07-18 16:21:00 +0800
categories: [IT技术, "Linux"]
tags: [IT技术, "linux", '防火墙', firewalld, 命令]
math: true
mermaid: true
image:
  path: /images/2024/07/2024-07-18/Linux防火墙：firewall-cmd命令介绍/Linux防火墙：firewall-cmd命令介绍-00.png
  lqip: data:image/webp;base64,UklGRlIAAABXRUJQVlA4IEYAAACwAQCdASoIAAUAAUAmJZACdADx+XBAAP7ePxMd/XnZXdcb/8P4DGyzQ6hqryP0JLSp9EsTkf8Qkn7o7/5EWvhuu+uApgAA
  alt: Linux防火墙：firewall-cmd命令介绍
---

## 前言

&emsp;&emsp;严格来说，不管是```firwalld```还是```iptables```，都是Linux防火墙的命令行工具，而不是防火墙。他们只是配置防火墙的规则的工具。真正起到防火墙作用的是```Linux```内核中的```netfilter```。所有其实```firewalld```和```iptables```道理是一样的，只是操作方式不同，也一样涉及到底层的“五表五链”。只是```iptables```更强调这个“五表五链”的原理，而```firewalld```则把这个原理弱化了。所有相比较而言，```firewalld```更易用。另外就是```firewalld```增加了```zone```的概念和动态修改机制。

&emsp;&emsp;这里简单介绍一下```firewalld```的命令行工具```firewall-cmd```。详细的使用可以通过```firewall-cmd -h```查看。

> 注：firewalld是一个服务，除了命令行配置工具```firewall-cmd```外，还可以使用图形化配置工具```firewall-config```。也可以直接编辑```/etc/firewalld/```下的配置文件。
{: .prompt-info }

## 概念

&emsp;&emsp;运行```firewall-cmd -h```命令后，会有很长的帮助信息输出，看上去有些可怕，其实如果自己看就可以发现提示信息中分为几个组：

+ **General Options** ：常规选项，比如```--help```、```--version```等。

+ **Status Options** ：状态选择，主要是用于控制防火墙状态。

+ **Log Denied Options** ：拒绝日志选项，主要是用于控制拒绝日志的输出。

+ **Automatic Helpers Options** ：自动助手选项，主要用于设置和获取自动助手设置。

+ **Permanent Options** ：持久化选择，只有一个```--permanent```选项，代表持久化。

+ **Zone Options** ：区域选择，主要是用于设置和获取区域。这个是firewalld新增的。

+ **IPSet Options** ：ipset选项，主要是用于设置和获取ipset。

+ **IcmpType Options** ：icmp报文类型选项，主要是用于管理icmp报文类型。

+ **Service Options** ：服务选项，主要是用与管理配置服务。

+ **Options to Adapt and Query Zones** ：调整和查询区域的选项，用于对区域的查询与调整，规则的查询与调整

+ **Options to Handle Bindings of Interfaces** ：处理绑定网卡的选项，用于对网卡绑定和解绑等相关操作。

+ **Options to Handle Bindings of Sources** ：处理绑定源地址的选项，用于对源地址的绑定和解绑的操作。

+ **Helper Options** ：助手选项，用于管理助手。

+ **Direct Options** ：直接配置选项，该选项用于直接配置规则，这个和iptables的操作方式很像。

+ **Lockdown Options** ：锁定选项

+ **Lockdown Whitelist Options** ：锁定白名单选项

+ **Panic Options** ：恐慌选项，开启将拒绝所有访问。

&emsp;&emsp;通过上面的这几组的介绍，我们可以找出几个概念：ports、zone、service、helper、IPSet、IcmpType、Sources、Interfaces。这里介绍一下这几个概念：

+ **Ports**：端口，也就是访问的端口。这个概念很简单，但是在防火墙中却是一个很重要的概念，因为防火墙主要手段就是 *限制端口*、*限制IP*、*限制协议* 等。

+ **Sources**：源地址，也就是访问的源地址。这个概念很简单，一样很重要，因为就是通过它来 *限制来访IP* 的一般就是一个IP地址或网段。如：192.168.0.1/24。

+ **Interfaces**：网卡接口，也就是访问的网卡接口，可以通过 ```ip a``` 或者 ```ifconfig``` 查看到自己电脑的网卡接口。

+ **IPSet**：IPSet，也就是多个IP组成的IP集。主要是为了方便管理多个IP，这个对应封禁大批量恶意访问很方便。

+ **IcmpType**：ICMP报文类型，防火墙可以通过分析ICMP报文类型来控制访问。

+ **Zone**：区域，firewalld新增的概念。firewalld的默认区域是```public```，其他区域包括```drop```、```internal```、```home```、```work```、```external```、```dmz```、```public```、```block```、```trusted```，这些通常通过语义来处理不同的请求，如：```drop```里面往往都是直接删除请求。你也可以自定义 ```Zone``` 相关的选择都在```Zone Options```中。我们可以认为区域是一个规则集，规则集中的规则是按照优先级排序的。一个请求过来，只会匹配一个zone，然后再匹配zone中的规则。匹配到哪个zone也是有规则的：先匹配```Source```,再匹配```Interface```,都没有匹配到就去匹配设置的默认```Zone```。各个zone的说明如下：

  + drop（丢弃）：任何接收的网络数据包都被丢弃，没有任何回复。仅能有发送出去的网络连接。

  + block（限制）：任何接收的网络连接都被IPv4 的icmp-host-prohibited 信息和IPv6 的icmp6-adm-prohibited 信息所拒绝。

  + public（公共）：在公共区域内使用，不能相信网络内的其他计算机不会对您的计算机造成危害，只能接收经过选取的连接。

  + external（外部）：特别是为路由器启用了伪装功能的外部网。您不能信任来自网络的其他计算，不能相信它们不会对您的计算机造成危害，只能接收经过选择的连接。

  + dmz（非军事区）：用于您的非军事区内的电脑，此区域内可公开访问，可以有限地进入您的内部网络，仅仅接收经过选择的连接。

  + work（工作）：用于工作区。您可以基本相信网络内的其他电脑不会危害您的电脑。仅仅接收经过选择的连接。

  + home（家庭）：用于家庭网络。您可以基本信任网络内的其他计算机不会危害您的计算机。仅仅接收经过选择的连接。

  + internal（内部）：用于内部网络。您可以基本上信任网络内的其他计算机不会威胁您的计算机。仅仅接受经过选择的连接。

  + trusted（信任）：可接受所有的网络连接。

+ **Service**：服务。服务其实就是用一个服务名更直观的代表一些端口，如：http通常代表80端口、https则通常是443端口、ssh则通常是22端口等。你也可以另外定义服务，服务名不冲突即可。这些服务管理的选择都在```Service Options```组中。

+ **Helper**：助手。主要用于连接的跟踪，这样就可以实现“有状态的防火墙”，也就是将相关的连接管理到一起。如：ftp的连接，我们知道一般来说ftp使用的是21号端口，不过21号端口主要是用来传输命令的，实际传输文件又会使用一个其他的端口，不过这两个连接还有内在的联系，这种情况就可以使用netfilter中的helper来处理，通过```module```来指定```nf_conntrack_ftp```内核模块，然后通过这个内核模块来处理请求，如：拒绝ftp的RELATED状态的数据包。这些助手管理的选择都在```Helper Options```组中。

> 注：除了Sources和Interfaces之外，其他几个概念都有对应的 ```xxx Options```，而这这些```xxx Options```，其实就是对这几个概念的管理。而 Ports、Sources 和 Interfaces 是最基本的概念，也是最为核心的。

## 命令

&emsp;&emsp;这里以CentOS7为例，简单介绍一下```firewall-cmd```的命令。详细使用还是要请参考```firewall-cmd -h```。

### 启动、关闭、重启、查看状态

```shell
# 这几个命令没有什么可说的，直接看命令行提示即可。

# 查看防火墙状态
systemctl status firewalld

#开启防火墙服务 
systemctl start firewalld

#关闭防火墙服务 
systemctl stop firewalld

#重启防火墙服务
systemctl restart firewalld

#设置防火墙开机自启动
systemctl enable firewalld

#设置防火墙禁止开机自启动
systemctl disable firewalld

# 查看防火墙是否开机自启动，disabled：开机不启动，enabled：开机自启动；
systemctl is-enabled firewalld
```

### 常规操作

```shell
# 查看帮助
firewall-cmd -h
firewall-cmd --help

# 查看版本
firewall-cmd -V
firewall-cmd --version

# 退出，啥也不做
firewall-cmd -q
firewall-cmd --quiet



```

### 状态操作

```shell
# 使用firewall-cmd --state查看防火墙状态，running：运行，not running：未运行；
firewall-cmd --state

# 不中断重新加载配置
firewall-cmd --reload

# 中断重新加载配置
firewall-cmd --complete-reload

# 将运行时配置保存到永久配置
firewall-cmd --runtime-to-permanent

# 检查永久配置文件是否正确
firewall-cmd --check-config
```

### 拒绝日志操作

```shell
# 打印拒绝日志值，on：开启，off：关闭；
firewall-cmd --get-log-denied

# 设置拒绝日志值，on：开启，off：关闭；
firewall-cmd --set-log-denied=<value>
```

### 自动助手操作

```shell
# 获取自动助手值，如:system
firewall-cmd --get-automatic-helpers

# 设置自动助手值
firewall-cmd --set-automatic-helpers=<value>
```

### 持久化操作

```shell
# 持久化保存配置，注意有些操作是要去必须有这个参数的，在帮助文档中有[P only]或[P]标识
firewall-cmd --permanent
```

### 区域操作

```shell
# 打印默认区域，如：public
firewall-cmd --get-default-zone

# 设置默认区域，如：public
firewall-cmd --set-default-zone=<zone>

# 打印当前区域，如：public
firewall-cmd --get-active-zones

# 打印已定义区域，如：block、dmz、drop、external、home、internal、public、trusted、work 
firewall-cmd --get-zones

# 打印已定义服务，如：dns、ftp、http、https
firewall-cmd --get-services

# 打印已定义icmptypes，如：echo-reply、destination-unreachable、echo-request
firewall-cmd --get-icmptypes

# 打印网卡接口对应的zone，如：eth0: public
firewall-cmd --get-zone-of-interface=<interface>

# 打印源地址对应的zone，如：192.168.0.1/32: public
firewall-cmd --get-zone-of-source=<source>[/<mask>]|<MAC>|ipset:<ipset>

# 打印zone列表及对应规则
firewall-cmd --list-all-zones

# 新定义一个zone  [P only]
firewall-cmd --new-zone=<zone>

# 从一个文件定义一个zone [P only]
firewall-cmd --new-zone-from-file=<filename> [--name=<zone>]

# 删除一个zone [P only]
firewall-cmd --delete-zone=<zone>

# 加载一个zone作为设置 [P only]
firewall-cmd --load-zone-defaults=<zone>

# 指定一个zone，一般用于给被指定的zone设置规则
firewall-cmd --zone=<zone>

# 获取zone的target，zone默认是当前，也可以使用--zone=<zone>来指定 [P only]
firewall-cmd --get-target

# 设置zone的target，zone默认是当前，也可以使用--zone=<zone>来指定 [P only]
firewall-cmd --set-target=<target>

# 打印zone信息
firewall-cmd --info-zone=<zone>

# 打印zone的配置文件路径 [P only]
firewall-cmd --path-zone=<zone>
```

### IPSet操作

```shell
# 打印支持的IPSet类型，如：hash:ip hash:ip,mark hash:ip,port hash:ip,port,ip hash:ip,port,net hash:mac hash:net hash:net,iface hash:net,net hash:net,port hash:net,port,net
firewall-cmd --get-ipset-types

# 创建新的IPSet，该操作会创建一个文件 [P only]
firewall-cmd --new-ipset=<ipset> --type=<ipset type> [--option=<key>[=<value>]]..

# 从一个文件创建新的IPSet [P only]
firewall-cmd --new-ipset-from-file=<filename> [--name=<ipset>]

# 删除一个IPSet [P only]
firewall-cmd --delete-ipset=<ipset>

# 加载一个IPSet作为设置 [P only]
firewall-cmd --load-ipset-defaults=<ipset>

# 打印IPSet信息
firewall-cmd --info-ipset=<ipset>

# 打印IPSet的配置文件路径 [P only]
firewall-cmd --path-ipset=<ipset>

# 打印已定义的IPSet
firewall-cmd --get-ipsets

# 设置新的描述到ipset [P only]
firewall-cmd --ipset=<ipset> --set-description=<description>

# 打印ipset的描述 [P only]
firewall-cmd --ipset=<ipset> --get-description

# 设置新的short描述到ipset [P only]
firewall-cmd --ipset=<ipset> --set-short=<description>

# 打印ipset的short描述 [P only]
firewall-cmd --ipset=<ipset> --get-short

# 添加一个新条目到ipset
firewall-cmd --ipset=<ipset> --add-entry=<entry>

# 从ipset删除一个条目
firewall-cmd --ipset=<ipset> --remove-entry=<entry>

# 查询一个条目
firewall-cmd --ipset=<ipset> --query-entry=<entry>

# 打印ipset的条目
firewall-cmd --ipset=<ipset> --get-entries

# 从一个文件添加新条目到ipset
firewall-cmd --ipset=<ipset> --add-entries-from-file=<entry>

# 从ipset删除文件中的条目到
firewall-cmd --ipset=<ipset> --remove-entries-from-file=<entry>

```

### IcmpType操作

```shell
# 创建新的IcmpType，该操作会创建一个文件 [P only]
firewall-cmd --new-icmptype=<icmptype>

# 从一个文件创建新的IcmpType [P only]
firewall-cmd --new-icmptype-from-file=<filename> [--name=<icmptype>]

# 删除一个IcmpType [P only]
firewall-cmd --delete-icmptype=<icmptype>

# 加载一个IcmpType默认设置 [P only]
firewall-cmd --load-icmptype-defaults=<icmptype>

# 打印IcmpType信息
firewall-cmd --info-icmptype=<icmptype>

# 打印IcmpType的配置文件路径 [P only]
firewall-cmd --path-icmptype=<icmptype>

# 添加描述到IcmpType [P only]
firewall-cmd --icmptype=<icmptype> --set-description=<description>

# 打印IcmpType的描述 [P only]
firewall-cmd --icmptype=<icmptype> --get-description

# 设置新的short描述到IcmpType [P only]
firewall-cmd --icmptype=<icmptype> --set-short=<description>

# 打印IcmpType的short描述 [P only]
firewall-cmd --icmptype=<icmptype> --get-short

# 添加目标到IcmpType [P only]
firewall-cmd --icmptype=<icmptype> --add-destination=<ipv>

# 从IcmpType删除目标 [P only]
firewall-cmd --icmptype=<icmptype> --remove-destination=<ipv>

# 查询IcmpType的目标 [P only]
firewall-cmd --icmptype=<icmptype> --query-destination=<ipv>

# 打印IcmpType的目标 [P only]
firewall-cmd --icmptype=<icmptype> --get-destinations
```

### 服务操作

```shell
# 新建一个服务，该操作会创建一个文件 [P only]
firewall-cmd --new-service=<service>
# 从一个文件创建新服务 [P only]
firewall-cmd --new-service-from-file=<filename> [--name=<service>]
# 删除一个服务 [P only]
firewall-cmd --delete-service=<service>
# 加载一个服务默认设置 [P only]
firewall-cmd --load-service-defaults=<service>
# 打印服务信息
firewall-cmd --info-service=<service>
# 打印服务配置文件路径 [P only]
firewall-cmd --path-service=<service>
# 设置新的描述到服务 [P only]
firewall-cmd --service=<service> --set-description=<description>
# 打印服务描述 [P only]
firewall-cmd --service=<service> --get-description
# 设置新的short描述到服务 [P only]
firewall-cmd --service=<service> --set-short=<description>
# 打印服务short描述 [P only]
firewall-cmd --service=<service> --get-short
# 添加一个新端口到服务 [P only]
firewall-cmd --service=<service> --add-port=<portid>[-<portid>]/<protocol>
# 从服务中删除一个端口 [P only]
firewall-cmd --service=<service> --remove-port=<portid>[-<portid>]/<protocol>
# 查询服务中是否添加了端口 [P only]
firewall-cmd --service=<service> --query-port=<portid>[-<portid>]/<protocol>
# 打印服务中添加的端口 [P only]
firewall-cmd --service=<service> --get-ports
# 添加一个协议到服务 [P only]
firewall-cmd --service=<service> --add-protocol=<protocol>
# 从服务中删除一个协议 [P only]
firewall-cmd --service=<service> --remove-protocol=<protocol>
# 查询服务中是否添加了协议 [P only]
firewall-cmd --service=<service> --query-protocol=<protocol>
# 打印服务中添加的协议 [P only]
firewall-cmd --service=<service> --get-protocols
# 添加一个新源端口到服务 [P only]
firewall-cmd --service=<service> --add-source-port=<portid>[-<portid>]/<protocol>
# 从服务中删除一个源端口 [P only]
firewall-cmd --service=<service> --remove-source-port=<portid>[-<portid>]/<protocol>
# 查询服务中是否添加了源端口 [P only]
firewall-cmd --service=<service> --query-source-port=<portid>[-<portid>]/<protocol>
# 打印服务中添加的源端口 [P only]
firewall-cmd --service=<service> --get-source-ports
# 添加一个新模块到服务 [P only]
firewall-cmd --service=<service> --add-module=<module>
# 从服务中删除一个模块 [P only]
firewall-cmd --service=<service> --remove-module=<module>
# 查询服务中是否添加了模块 [P only]
firewall-cmd --service=<service> --query-module=<module>
# 打印服务中添加的模块 [P only]
firewall-cmd --service=<service> --get-modules
# 添加一个目标到服务 [P only]
firewall-cmd --service=<service> --set-destination=<ipv>:<address>[/<mask>]
# 从服务中删除目标 [P only]
firewall-cmd --service=<service> --remove-destination=<ipv>
# 查询服务中是否添加了目标 [P only]
firewall-cmd --service=<service> --query-destination=<ipv>:<address>[/<mask>]
# 打印服务中添加的目标 [P only]
firewall-cmd --service=<service> --get-destinations

# 从上面方法看服务中可以添加端口、源、模块、目标、协议，这些方法都要使用--permanent参数永久生效
```

### 调整和查询区域的操作

```shell
# 下面几个命令默认是default区域，也可以通过--zone=<zone>指定区域
# 打印默认区域所有信息，比较详细
firewall-cmd --list-all
# 打印默认区域服务信息，只有服务列表
firewall-cmd --list-services
#设置默认区域超时时间，h,m,s
firewall-cmd --timeout=<timeval>
# 设置描述到默认zone[P only]
firewall-cmd --set-description=<description>
# 打印默认zone的描述[P only]
firewall-cmd --get-description
# 设置新的short描述到默认zone[P only]
firewall-cmd --set-short=<description>
# 打印默认zone的short描述[P only]
firewall-cmd --get-short
# 添加一个服务到默认zone
firewall-cmd --add-service=<service>
# 从默认zone中删除一个服务
firewall-cmd --remove-service=<service>
# 查询默认zone中是否添加了服务
firewall-cmd --query-service=<service>
# 打印默认zone中添加的端口列表
firewall-cmd --list-ports
# 添加一个端口到默认zone
firewall-cmd --add-port=<portid>[-<portid>]/<protocol>
# 从默认zone中删除一个端口
firewall-cmd --remove-port=<portid>[-<portid>]/<protocol>
# 查询默认zone中是否添加了端口
firewall-cmd --query-port=<portid>[-<portid>]/<protocol>
# 打印默认zone中添加的协议列表
firewall-cmd --list-protocols
# 添加一个协议到默认zone
firewall-cmd --add-protocol=<protocol>
# 从默认zone中删除一个协议
firewall-cmd --remove-protocol=<protocol>
# 查询默认zone中是否添加了协议
firewall-cmd --query-protocol=<protocol>

#下面的都差不多，列表、添加、删除、查询，都是--zone=<zone>指定区域
firewall-cmd --list-source-ports
firewall-cmd --add-source-port=<portid>[-<portid>]/<protocol>
firewall-cmd --remove-source-port=<portid>[-<portid>]/<protocol>
firewall-cmd --query-source-port=<portid>[-<portid>]/<protocol>
firewall-cmd --list-icmp-blocks
firewall-cmd --add-icmp-block=<icmptype>
firewall-cmd --remove-icmp-block=<icmptype>
firewall-cmd --query-icmp-block=<icmptype>
firewall-cmd --add-icmp-block-inversion
firewall-cmd --remove-icmp-block-inversion
firewall-cmd --query-icmp-block-inversion
firewall-cmd --list-forward-ports
firewall-cmd --add-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<firewall-cmd ask>]]
firewall-cmd --remove-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[firewall-cmd <mask>]]
firewall-cmd --query-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[firewall-cmd <mask>]]
firewall-cmd --add-masquerade
firewall-cmd --remove-masquerade
firewall-cmd --query-masquerade
firewall-cmd --list-rich-rules
firewall-cmd --add-rich-rule=<rule>
firewall-cmd --remove-rich-rule=<rule>
firewall-cmd --query-rich-rule=<rule>
```

### 处理接口绑定的操作

```shell
# 打印所有网卡接口的绑定
firewall-cmd --list-interfaces
# 绑定一个网卡到默认zone
firewall-cmd --add-interface=<interface>
# 换一个网卡，这里注意，一个网卡只能属于一个zone
firewall-cmd --change-interface=<interface>
# 查询当前zone是否绑定了这个网卡
firewall-cmd --query-interface=<interface>
# 解除网卡与zone的绑定
firewall-cmd --remove-interface=<interface>
```

### 处理源绑定的操作

```shell
# 列出所有源的绑定
firewall-cmd --list-sources
# 绑定一个源到默认zone
firewall-cmd --add-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
# 换一个源，注意，这里一个源只能属于一个zone
firewall-cmd --change-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
# 查询当前zone是否绑定了这个源
firewall-cmd --query-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
# 解除源与zone的绑定
firewall-cmd --remove-source=<source>[/<mask>]|<MAC>|ipset:<ipset>
```

### 助手操作

```shell
# 新建一个helper
firewall-cmd --new-helper=<helper> --module=<module> [--family=<family>]
# 从文件新建一个hepler
firewall-cmd --new-helper-from-file=<filename> [--name=<helper>]
# 删除一个helper
firewall-cmd --delete-helper=<helper>
# 加载一个Helper默认
firewall-cmd --load-helper-defaults=<helper>
# 打印Helper信息
firewall-cmd --info-helper=<helper>
# 打印Helper配置文件路径
firewall-cmd --path-helper=<helper>
# 获取所有定义的Helper
firewall-cmd --get-helpers
# 设置Helper描述
firewall-cmd --helper=<helper> --set-description=<description>
# 打印Helper描述
firewall-cmd --helper=<helper> --get-description
# 设置Helper的short描述
firewall-cmd --helper=<helper> --set-short=<description>
# 打印Helper的short描述
firewall-cmd --helper=<helper> --get-short
# 添加一个端口到Helper [P only]
firewall-cmd --helper=<helper> --add-port=<portid>[-<portid>]/<protocol>
# 从Helper中删除一个端口 [P only]
firewall-cmd --helper=<helper> --remove-port=<portid>[-<portid>]/<protocol>
# 查询Helper中是否添加了端口 [P only]
firewall-cmd --helper=<helper> --query-port=<portid>[-<portid>]/<protocol>
# 打印Helper的所有端口 [P only]
firewall-cmd --helper=<helper> --get-ports
# 设置Helper的module [P only]
firewall-cmd --helper=<helper> --set-module=<module>
# 打印Helper的module [P only]
firewall-cmd --helper=<helper> --get-module
# 设置Helper的family [P only]
firewall-cmd --helper=<helper> --set-family={ipv4|ipv6|}
# 打印Helper的family [P only]
firewall-cmd --helper=<helper> --get-family
```

### 直接操作

```shell
# First option for all direct options 这个选择是代表直接操作，是后面所有操作的第一个参数
# 直接操作的方式是去操作表和链，而不是zone
firewall-cmd --direct
# 获取所有链
firewall-cmd --get-all-chains
#获取所有添加到表的链
firewall-cmd --get-chains {ipv4|ipv6|eb} <table>
# 添加一个链到表
firewall-cmd --add-chain {ipv4|ipv6|eb} <table> <chain>
# 从表移除一个链从表
firewall-cmd --remove-chain {ipv4|ipv6|eb} <table> <chain>
# 查询链是否添加到表
firewall-cmd --query-chain {ipv4|ipv6|eb} <table> <chain>
# 打印所有规则
firewall-cmd --get-all-rules
# 获取指定表 链的规则
firewall-cmd --get-rules {ipv4|ipv6|eb} <table> <chain>
# 添加规则到链在表
firewall-cmd --add-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
# 从链中删除规则
firewall-cmd --remove-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
# 删除链的全部规则
firewall-cmd --remove-rules {ipv4|ipv6|eb} <table> <chain>
# 查询规则是否添加到链
firewall-cmd --query-rule {ipv4|ipv6|eb} <table> <chain> <priority> <arg>...
# 传递命令（不受防火墙跟踪），Pass a command through (untracked by firewalld)
firewall-cmd --passthrough {ipv4|ipv6|eb} <arg>...
# 获取所有传递命令，Get all tracked passthrough rules [P]
firewall-cmd --get-all-passthroughs
# 获取规则传递命令，Get tracked passthrough rules [P]
firewall-cmd --get-passthroughs {ipv4|ipv6|eb} <arg>...
# 添加，Add a new tracked passthrough rule [P]
firewall-cmd --add-passthrough {ipv4|ipv6|eb} <arg>...
# 删除 ，Remove a tracked passthrough rule [P]
firewall-cmd --remove-passthrough {ipv4|ipv6|eb} <arg>...
# 查询，Return whether the tracked passthrough rule has been added [P]
firewall-cmd --query-passthrough {ipv4|ipv6|eb} <arg>...
```

### 锁定操作

```shell
# Enable lockdown.
firewall-cmd --lockdown-on
# Disable lockdown.
firewall-cmd --lockdown-off
# Query whether lockdown is enabled
firewall-cmd --query-lockdown
```

### 锁定白名单操作

```shell
# 列出白名单上的所有命令
firewall-cmd --list-lockdown-whitelist-commands
# 添加命令到白名单
firewall-cmd --add-lockdown-whitelist-command=<command>
# 删除
firewall-cmd --remove-lockdown-whitelist-command=<command>
# 查询
firewall-cmd --query-lockdown-whitelist-command=<command>
# 列出白名单上的所有协议，List all contexts that are on the whitelist [P]
firewall-cmd --list-lockdown-whitelist-contexts
# 添加
firewall-cmd --add-lockdown-whitelist-context=<context>
# 删除
firewall-cmd --remove-lockdown-whitelist-context=<context>
# 查询
firewall-cmd --query-lockdown-whitelist-context=<context>
# 列出白名单上的所有用户id，List all user ids that are on the whitelist [P]
firewall-cmd --list-lockdown-whitelist-uids
# 添加
firewall-cmd --add-lockdown-whitelist-uid=<uid>
# 删除
firewall-cmd --remove-lockdown-whitelist-uid=<uid>
# 查询
firewall-cmd --query-lockdown-whitelist-uid=<uid>
# 列出白名单上的所有用户名，List all user names that are on the whitelist [P]
firewall-cmd --list-lockdown-whitelist-users
# 添加
firewall-cmd --add-lockdown-whitelist-user=<user>
# 删除
firewall-cmd --remove-lockdown-whitelist-user=<user>
# 查询
firewall-cmd --query-lockdown-whitelist-user=<user>
```

### 恐慌操作

```shell
# 开启恐慌模式，该模式会阻止所有网络连接，慎用，如果是ssh链接很容易就再也连不上了
firewall-cmd --panic-on

# 关闭恐慌模式
firewall-cmd --panic-off

# 查询是否开启了恐慌模式
firewall-cmd --query-panic
```

## 总结

&emsp;&emsp;firewall-cmd命令是Linux防火墙的常用命令，通过该命令可以查看、添加、删除、查询防火墙规则，也可以直接操作表和链。上面只是把命令简单列了一下，这些命令都是通过```firewall-cmd -h```查看的，具体使用还是需要自己多看文档。其实要管理好防火墙，光了解这些命令还是不够的，还是要更好地理解防火墙的工作原理。
