---
title: Linux防火墙：firewalld-RichRule
author: mmy83
date: 2024-07-19 17:42:00 +0800
categories: [IT技术, "Linux"]
tags: [IT技术, "linux", '防火墙', firewalld, 命令]
math: true
mermaid: true
image:
  path: /images/2024/07/2024-07-19/Linux防火墙：firewalld-RichRule/Linux防火墙：firewalld-RichRule-00.png
  lqip: data:image/webp;base64,UklGRk4AAABXRUJQVlA4IEIAAADQAQCdASoIAAUAAUAmJQBOgCP7eU0cAAD+/A9RCgNXyO/43z8xQFFn/KsHgw65/R622/vr8/CP1lhjF3ybYK1OAAA=
  alt: Linux防火墙：firewalld-RichRule
---

## 前言

&emsp;&emsp;前面写了一篇[Linux防火墙：firewall-cmd命令介绍](/posts/Linux防火墙-firewall-cmd命令介绍/)，里面简单的介绍了```firewall-cmd```命令，主要是对```firewall-cmd -h```给出的文档做了简单的介绍和翻译。今天这篇主要介绍一下 **富规则** 涉及到的命令：```firewall-cmd --list-rich-rules```、```firewall-cmd --add-rich-rule=<rule>```、```firewall-cmd --remove-rich-rule=<rule>```、```firewall-cmd --query-rich-rule=<rule>```。

## Rich Language 介绍

&emsp;&emsp;```firewalld.richlanguage``` 是 ```firewalld``` 防火墙服务中的一部分，它提供了一种更为丰富和易于理解的方式来创建复杂的防火墙规则。这种丰富语言（Rich Language）使用带有值的关键词，并且是对 iptables 规则的一种抽象表示。

## Rich Language 特点

- 扩展性：它扩展了当前区域（zone）的元素（如服务、端口、ICMP 阻塞、ICMP 类型、伪装、端口转发和源端口），添加了额外的源和目的地址、日志记录、动作以及对日志和动作的限制。

- 直观性：通过使用易于理解的关键词和值，使得创建复杂的防火墙规则变得更加直观和简单。

- 规则与区域：一个规则是区域的一部分，一个区域可以包含多个规则。如果某些规则之间存在冲突或矛盾，那么第一个匹配的规则会“获胜”。

- 命令行客户端和 D-Bus 接口：在命令行客户端和 D-Bus 接口中使用的丰富语言。

## Rich Rule Priorities(富规则优先级)

&emsp;&emsp;富规则支持优先级字段 priority 之前，是通过规则的行为进行排序的：```log``` 规则始终在 ```deny``` 规则之前,```deny``` 规则始终在 ```allow``` 规则之前，所以规则执行顺序为：

> **log > drop/reject > accept**

&emsp;&emsp;新版的 firewalld 添加了新的 priority 字段。它可以是 -32768 到 32767 之间的任何数字，其中数字越小，优先级越高。此范围足够大，以允许从脚本或其他实体自动生成规则。

根据优先级，规则将被组织到不同的链中：

1. 如果优先级小于 0，则规则进入具有后缀 _pre 的链。
2. 如果优先级大于 0，则规则进入具有后缀 _post 的链。
3. 如果优先级等于 0，则规则根据其操作进入链（_log，_deny，_allow）。这与在添加优先级支持之前富规则的行为相同。

## 格式

### 一般规则结构

```shell
rule       #定义一个规则的开始。
  [source] #定义一个规则的开始。
  [destination]     #可选，定义规则应用的目标地址或地址集（只要它不与服务的目标冲突）。
  service|port|protocol|icmp-block|icmp-type|masquerade|forward-port|source-port
   #定义规则所适用的服务、端口、协议、ICMP 块、ICMP 类型、伪装、端口转发或源端口。
  [log|nflog]       #可选，定义是否记录匹配此规则的流量，以及使用哪种日志记录机制（log 为 firewalld 日志，nflog 为 Netfilter 日志）
  [audit]           #可选，定义是否将匹配此规则的流量发送到审计子系统
  [accept|reject|drop|mark] #定义当规则匹配时应该执行的操作（接受、拒绝、丢弃或标记）。
```

### 源黑名单或白名单的规则结构

```shell
rule
  source
  [log|nflog]
  [audit]
  accept|reject|drop|mark
```

## 规则中元素的选项

### rule

```shell
rule [family="ipv4|ipv6"] [priority="priority"]
```

- ```family="ipv4|ipv6"```: 指定规则适用的地址族，可以是 "ipv4" 或 "ipv6"。如果提供了地址族，则规则将限定为 IPv4 或 IPv6。如果未提供地址族，则规则将同时适用于 IPv4 和 IPv6。如果规则中使用了源地址（source）或目标地址（destination），或者涉及端口/数据包转发（port/packet forwarding），则必须指定family

- ```priority="priority"```: 规则的优先级设置，范围是 -32768 到 32767，数值越低表示优先级越高。富规则按照优先级排序。具有相同优先级值的规则的顺序未定义。负优先级值将在其他 firewalld 之前执行；正优先级值将在其他 firewalld 之后执行；优先级值为0将根据"日志和操作信息"中的操作放置规则在链中，与没有配置优先级时的规则是一样的。

### Source

```shell
source [not] address="address[/mask]"|mac="mac-address"|ipset="ipset"
```

- ```source address="address[/mask]"```: 用于指定连接的源地址，可以是单个IP地址、网络IP地址、MAC地址或IPSet。地址必须与规则所属的地址族（IPv4/IPv6）匹配。子网掩码可以用点分十进制（/x.x.x.x）或前缀表示法（/x）表示IPv4，对于IPv6网络地址，子网掩码以前缀表示法（/x）表示。

- ```not source address="address[/mask]"```: 使用not关键字可以反转地址的意义，添加not关键字之后将匹配除指定地址之外的所有地址。

### Destination

```shell
destination [not] address="address[/mask]"|ipset="ipset"
```

- ```destination address="address[/mask]"```: 用于指定连接的目标地址，可以是单个IP地址、网络IP地址或IPSet。目标地址与源地址的语法相同。

- ```not destination address="address[/mask]"```: 使用 not 关键字可以反转目标地址的含义，匹配除指定地址之外的所有地址。

&emsp;&emsp;通过 destination 选项，可以根据目标地址来限制连接的目标，确保防火墙规则只允许连接到特定目标地址，注意，不是所有元素都支持使用目标地址，具体取决于服务条目等元素是否使用目标地址。

### Service

```shell
service name="service name"
```

- ```service name="service name"```: 指定要添加到规则的服务名称，该服务名称是 firewalld 提供的服务之一。可以使用 firewall-cmd --get-services 命令获取受支持的服务列表。

&emsp;&emsp;如果一个服务提供了目标地址，它将与规则中的目标地址发生冲突，并导致错误。通常情况下，内部使用多播的服务会使用目标地址，这可能会与显式指定的目标地址冲突。请注意这一点，以避免出现冲突和错误。

### Port

```shell
port port="port value" protocol="tcp|udp|sctp|dccp"
```

- ```port port="port value"```: 指定要匹配的端口值，可以是单个端口号（portid）或端口范围（portid-portid）。

- ```protocol="tcp|udp|sctp|dccp"```: 指定端口所使用的协议，可以是 TCP、UDP、SCTP 或 DCCP 中的一个。

### Protocol

```shell
protocol value="protocol value"
```

&emsp;&emsp;通过指定协议值，可以根据特定协议配置防火墙规则，以控制对特定协议的访问权限。可以使用协议 ID 号或协议名称来定义要匹配的协议，这样可以确保规则按照所需的协议进行过滤。查看 /etc/protocols 文件获取受支持的协议列表。

### Tcp-Mss-Clamp

```shell
tcp-mss-clamp="value=pmtu|value=number >= 536|None"
```

- ```pmtu```: 将最大段大小设置为路径最大传输单元（Path Maximum Transmission Unit）。

- ```number >= 536```: 可以将最大段大小设置为大于或等于 536 的数字。

- ```None```: 如果未指定属性值，则自动设置为路径最大传输单元 "pmtu"。

&emsp;&emsp;通过调整最大段大小，您可以控制 TCP 数据包在网络中传输时的大小，这对于优化性能和处理某些网络问题非常有用。根据不同的网络配置和需求，您可以灵活地设置最大段大小以满足特定的通信要求。

### ICMP-Block

```shell
icmp-block name="icmptype name"
```

- ```icmp-block name="icmptype name"```: 指定要阻止的 ICMP 类型名称，这些类型必须是 firewalld 支持的 ICMP 类型之一。您可以使用 firewall-cmd --get-icmptypes 命令获取支持的 ICMP 类型列表。

&emsp;&emsp;在 icmp-block 中不允许指定操作（action），因为 icmp-block 内部使用的动作是拒绝（reject）。通过使用 icmp-block 选项，可以方便地阻止特定的 ICMP 类型，以加强网络安全并控制 ICMP 流量。

### Masquerade

```shell
masquerade
```

&emsp;&emsp;通过使用 masquerade 选项，可以启用源地址伪装，使得内部网络上的主机能够共享单个公共 IP 地址访问外部网络。在启用 masquerading 功能时，会隐式启用 IP 转发（IP forwarding），以确保数据包能够正确转发到目标地址。请注意，在 masquerade 中不允许指定操作（action）参数。

### ICMP-Type

```shell
icmp-type name="icmptype name"
```

- 和```icmp-block```一样，可以通过```firewall-cmd --get-icmptypes```查看支持的类型列表

### Forward-Port

```shell
forward-port port="port value" protocol="tcp|udp|sctp|dccp" to-port="port value" to-addr="address"
```

&emsp;&emsp;在 firewall-cmd 命令中，forward-port 选项用于配置端口转发规则，将本地端口的数据包通过指定协议（TCP、UDP、SCTP 或 DCCP）转发到另一个本地端口、另一台机器上的端口，或者另一台机器上。

- ```port="port value"```: 指定要转发的本地端口值，可以是单个端口号或端口范围

- ```protocol="tcp|udp|sctp|dccp"```: 指定转发所使用的协议，可以是 TCP、UDP、SCTP 或 DCCP 中的一个。

- ```to-port="port value"```: 指定要转发到的目标端口值。

- ```to-addr="address"```: 指定要转发到的目标 IP 地址。

&emsp;&emsp;通过 forward-port 选项，您可以实现灵活的端口转发设置，将流量从一个端口传输到另一个端口或另一台设备上。在配置转发规则时，不允许指定操作参数，因为 forward-port 内部使用的动作是接受（accept）。如果指定了 to-addr 参数，则会隐式启用 IP 转发功能。

### Source-Port

```shell
source-port port="port value" protocol="tcp|udp|sctp|dccp"
```

&emsp;&emsp;source-port 选项用于指定规则中的源端口

- ```port="port value"```: 指定要匹配的源端口值，可以是单个端口号（portid）或端口范围（portid-portid）。
- ```protocol="tcp|udp|sctp|dccp"```: 指定源端口所使用的协议，可以是 TCP、UDP、SCTP 或 DCCP 中的一个。

### Log

```shell
log [prefix="prefix text"] [level="log level"] [limit value="rate/duration"]
```

&emsp;&emsp;可以定义一个最大长度为 127 个字符的前缀文本，将其作为前缀添加到日志消息中。日志级别可选择为 "emerg"、"alert"、"crit"、"error"、"warning"、"notice"、"info" 或 "debug"，如果未指定，则默认为 "warning"。

### NFLog

```shell
nflog [group="group id"] [prefix="prefix text"] [queue-size="threshold"] [limit value="rate/duration"]
```

NFLOG 是 iptables 和 nftables 中的一个功能，它允许你将匹配到的网络数据包通过 netlink 套接字传递给用户空间的应用程序进行进一步的处理或记录。这对于需要自定义网络流量日志记录或分析的时候非常有用。

- ```group="group id"```：

  1. 这是一个多播组标识符，用于标识哪些用户空间的应用程序应该接收这个组的数据包。

  2. 最小和默认值是 0，最大值是 65535。

  3. 通过 netlink 套接字，多个用户空间应用程序可以订阅同一个组，从而接收该组的数据包。

- ```prefix="prefix text"```：

  1. 这允许你为日志消息添加一个前缀文本。

  2. 前缀文本的最大长度是 127 个字符。

- ```queue-size="threshold"```：

  1. 这个选项可以设置队列的阈值，以帮助限制上下文切换。

  2. 默认值是 1，最大值是 65535。

  3. 当队列中的数据包数量达到这个阈值时，新到达的数据包可能会被丢弃或采取其他动作。

- ```limit value="rate/duration"```：

  1. 这是一个速率限制选项，允许你指定在指定的时间段内应该记录或处理的数据包的最大速率。

  2. 例如，limit 5/minute 表示每分钟最多记录 5 个数据包。

  3. 这对于防止日志系统过载或降低网络流量分析的开销非常有用。

### Audit

```shell
audit [limit value="rate/duration"]
```

&emsp;&emsp;audit 是一个在 iptables 或 nftables 中用于将网络事件记录到审计系统（通常是通过 auditd 服务）的动作。与 NFLOG 类似，audit 提供了一种将网络流量信息传递给用户空间程序进行记录和分析的方法，但它是通过 Linux 审计系统（audit system）而不是 netlink 套接字。

&emsp;&emsp;audit 动作也支持速率限制（limit），允许管理员指定在指定时间段内应记录的最大事件数。例如，limit 5/minute 表示每分钟最多记录 5 个审计事件。这有助于防止审计日志过大或过载。

### Action

&emsp;&emsp;action可以是接受（accept）、拒绝（reject）、丢弃（drop）或标记（mark）中的一种。 规则可以包含一个元素，也可以仅包含源。如果规则包含一个元素，则与该元素匹配的新连接将使用该动作处理。如果规则不包含元素，则来自源地址的所有内容都将使用该动作处理。

```console
accept [limit value="rate/duration"]
accept 将接受新的连接尝试
 
reject [type="reject type"] [limit value="rate/duration"]
使用 reject 时，它们将不被接受，并且它们的源将收到一个拒绝的 ICMP(v6) 消息。可以设置 reject 类型以指定适当的 ICMP(v6) 错误消息。
 
drop [limit value="rate/duration"]
drop 会立即丢弃所有数据包，不会向源发送任何信息。
 
mark set="mark[/mask]" [limit value="rate/duration"]
使用 mark 将在 mangle 表的 PREROUTING 链中标记所有数据包，使用给定的标记和掩码组合。
```

### Limit

```shell
limit value="rate/duration"
```

&emsp;&emsp;limit 关键字用于限制日志记录（Log）、NFLOG、审计（Audit）和动作（Action）的频率。当一个规则使用了 limit 标签时，该规则将仅匹配直到达到指定的限制为止。

&emsp;&emsp;limit 的值由两部分组成：速率（rate）和持续时间（duration）。速率是一个正整数，表示在给定的持续时间内允许的最大匹配次数。持续时间可以使用 "s"（秒）、"m"（分钟）、"h"（小时）或 "d"（天）作为单位来指定。

&emsp;&emsp;例如，limit value="5/m" 表示每分钟最多允许 5 次匹配。而 limit value="2/d" 表示每天最多允许 2 次匹配。

&emsp;&emsp;在 firewalld 中，日志记录和动作是通过特定的动作（actions）来实现的，这些动作包括 log、nflog 和 audit。为了保持日志记录的顺序和逻辑，firewalld 在所有区域（zones）中添加了一个新的链（chain）叫做 zone_log。这个链位于 deny 链之前，以便在拒绝数据包之前先记录相关信息。

### Information about logging and actions

&emsp;&emsp;根据规则的优先级和动作，规则或其部分被放置在以下不同的链中：

- ```zone_pre```：当规则的优先级小于 0 时，富规则（rich rule）将被放置在这个链中。

- ```zone_log```：当规则的优先级等于 0 时，所有日志记录规则将被放置在这个链中。所有拒绝（reject）和丢弃（drop）规则将被放置在 

- zone_deny 链中，该链将在 log 链之后被遍历。所有允许（accept）规则将被放置在 zone_allow 链中，该链将在 deny 链之后被遍历。如果一个规则同时包含日志记录以及拒绝或允许动作，则这些部分将被放置在相应的链中。

- ```zone_deny```：这个链用于处理拒绝规则。

- ```zone_allow```：这个链用于处理允许规则。

- ```zone_post```：当规则的优先级大于 0 时，富规则将被放置在这个链中。

&emsp;&emsp;这样的链组织允许 firewalld 按照一定的逻辑顺序来处理网络流量，并允许在数据包被拒绝或允许之前或之后执行日志记录操作。

&emsp;&emsp;在配置 firewalld 时，可以使用 firewall-cmd 或直接编辑防火墙配置文件来添加和修改这些规则。通过使用 rich-rule 选项，可以定义复杂的规则，包括日志记录、允许、拒绝等动作，并指定这些规则的优先级和放置的链。

## 配置rich rule

```shell
firewall-cmd --zone=your_zone --add-rich-rule='your_rich_rule'
 
firewall-cmd --zone=your_zone --remove-rich-rule='your_rich_rule'
 
firewall-cmd --zone=your_zone --list-rich-rules/--list-all
```

## 参考

- [firewalld.richlanguage](https://firewalld.org/documentation/man-pages/firewalld.richlanguage.html)

- [firewalld(4) Rich Rule](https://blog.csdn.net/weixin_42064802/article/details/140044541)
