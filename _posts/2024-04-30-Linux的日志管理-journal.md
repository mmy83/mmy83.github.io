---
title: Linux的日志管理-journal
author: mmy83
date: 2024-04-30 14:07:00 +0800
categories: [IT技术, Linux]
tags: [日志, journal, journald, journalctl, linux]
math: true
mermaid: true
image:
  path: /images/2024-04-30/Linux的日志管理-journal/Linux的日志管理-journal-00.png
  lqip: data:image/webp;base64,UklGRmQAAABXRUJQVlA4IFgAAAAwAwCdASoIAAYAAUAmJbACdLoB+AH4gM4A/UAEJe0uE3SAAP5NCWrWMOOYw+Fc/k+bb6xpBfhOPP/z05GrZipyo6uXj4+gH/9ZrsefBK+U4jzqJ8+98AAA
  alt: Linux的日志管理-journal
---

## 介绍

&emsp;&emsp;Linux Journal 日志系统，更准确来说是 Linux 系统中的 systemd-journald 服务。它提供了一个低开销、结构化的日志系统，用于收集来自系统、内核以及用户空间应用程序的消息。可以通过journalctl来对日志进行管理。目前大部分 Linux 系统都默认开启，所以可以直接使用journalctl来查看日志。而且如果不能妥善管理，将会占用大量的磁盘空间。可以使用```systemctl status systemd-journald```命令来查看存储状态。

&emsp;&emsp;journal 使用 systemd-journald 服务来存储二进制格式的日志文件。通常存储在 ```/run/log/journal/``` 或 ```/var/log/journal/```。journalctl 提供了更丰富和强大的查询和过滤功能，可以按时间、服务单元、日志级别等多个条件进行过滤，功能强大。

## 配置

```shell
# /etc/systemd/journald.conf
[Journal]
#Storage=auto
#Compress=yes
#Seal=yes
#SplitMode=uid
#SyncIntervalSec=5m
#RateLimitIntervalSec=30s
#RateLimitBurst=10000
#SystemMaxUse=
#SystemKeepFree=
#SystemMaxFileSize=
#SystemMaxFiles=100
#RuntimeMaxUse=
#RuntimeKeepFree=
#RuntimeMaxFileSize=
#RuntimeMaxFiles=100
#MaxRetentionSec=
#MaxFileSec=1month
#ForwardToSyslog=yes
#ForwardToKMsg=no
#ForwardToConsole=no
#ForwardToWall=yes
#TTYPath=/dev/console
#MaxLevelStore=debug
#MaxLevelSyslog=debug
#MaxLevelKMsg=notice
#MaxLevelConsole=info
#MaxLevelWall=emerg
#LineMax=48K
#ReadKMsg=yes
#Audit=no
```

* __Storage=__

在哪里存储日志文件： "volatile" 表示仅保存在内存中， 也就是仅保存在 /run/log/journal 目录中(将会被自动按需创建)。 "persistent" 表示优先保存在磁盘上， 也就优先保存在 /var/log/journal 目录中(将会被自动按需创建)， 但若失败(例如在系统启动早期"/var"尚未挂载)， 则转而保存在 /run/log/journal 目录中(将会被自动按需创建)。 "auto"(默认值) 与 "persistent" 类似， 但不自动创建 /var/log/journal 目录， 因此可以根据该目录的存在与否决定日志的保存位置。 "none" 表示不保存任何日志(直接丢弃所有收集到的日志)， 但日志转发(见下文)不受影响。 默认值是 "auto"

* __Compress=__

默认值"yes"表示 压缩存储大于特定阈值(默认为512字节)的对象。 也可以直接设置一个 字节值(可以带有 K, M, G 后缀) 表示的阈值， 表示压缩存储 大于指定阈值的对象。

* __Seal=__

默认值"yes"表示： 如果存在一个"sealing key"(由 journalctl(1) 的 --setup-keys 命令创建)， 那么就为所有持久保存的日志文件启用 FSS(Seekable Sequential Key Generators)保护， 以避免日志文件 被恶意或无意的修改。

* __SplitMode=__

设置是否按照每个用户分割日志文件，以实现对日志的访问控制(日志守护进程会确保每个用户都能读取自己的日志文件)。 可以使用的分割策略如下： "uid" 表示每个用户都有自己专属的日志文件(无论该用户是否拥有登录会话)，但系统用户的日志依然记录到系统日志中。这是默认值。 "none" 表示不对日志文件按不同用户进行分割，而是将所有日志都记录到系统日志中。这意味着非特权用户根本无法读取属于自己的日志信息。 注意， 仅分割持久保存的日志(/var/log/journal)， 永不分割内存中的日志(/run/log/journal)。

* __RateLimitIntervalSec=, RateLimitBurst=__

限制日志的生成速率 (设为零表示不作限制)。 RateLimitIntervalSec= 用于设置一个时间段长度，默认值是30秒。 RateLimitBurst= 用于设置一个正整数，表示消息条数，默认值是10000条。 表示在 RateLimitIntervalSec= 时间段内， 每个服务最多允许产生 RateLimitBurst= 数量(条数)的日志。 在同一个时间段内，超出数量限制的日志将被丢弃，直到下一个时间段才能再次开始记录。 对于所有被丢弃的日志消息，仅用一条类似"xxx条消息被丢弃"的消息来代替。 这个限制是针对每个服务的限制，一个服务超限并不会影响到另一个服务的日志记录。 RateLimitIntervalSec= 可以使用下面的时间单位： "ms", "s", "min", "h", "d"

如果一个服务已经通过 LogRateLimitIntervalSec= 和/或 LogRateLimitBurst= (参见 systemd.exec(5) 手册) 限制了自身的日志生成速率，那么将会覆盖此处的设置。

* __SystemMaxUse=, SystemKeepFree=, SystemMaxFileSize=, SystemMaxFiles=, RuntimeMaxUse=, RuntimeKeepFree=, RuntimeMaxFileSize=, RuntimeMaxFiles=__

限制日志文件的 大小上限。 以 "System" 开头的选项用于限制磁盘使用量， 也就是 /var/log/journal 的使用量。 以 "Runtime" 开头的选项用于限制内存使用量， 也就是 /run/log/journal 的使用量。 以 "System" 开头的选项仅在 /var/log/journal 目录确实存在且可写时才有意义。 但以 "Runtime" 开头的选项永远有意义。 也就是说， 在系统启动早期 /var 尚未挂载时、 或者系统管理员禁止在磁盘上存储日志的时候， 仅有 "Runtime" 开头的选项有意义。 journalctl 与 systemd-journald 工具会忽略日志目录中 所有后缀名不等于 ".journal" 或 ".journal~" 的文件。 换句话说，日志目录中不应该存在后缀名不等于 ".journal" 或 ".journal~" 的文件， 因为这些文件 永远不会被清理。

SystemMaxUse= 与 RuntimeMaxUse= 限制全部日志文件加在一起最多可以占用多少空间。 SystemKeepFree= 与 RuntimeKeepFree= 表示除日志文件之外，至少保留多少空间给其他用途。 systemd-journald 会同时考虑这两个因素， 并且尽量限制日志文件的总大小，以同时满足这两个限制。

SystemMaxUse= 与 RuntimeMaxUse= 的默认值是10%空间与4G空间两者中的较小者； SystemKeepFree= 与 RuntimeKeepFree= 的默认值是15%空间与4G空间两者中的较大者； 如果在 systemd-journald 启动时，文件系统即将被填满并且已经超越了 SystemKeepFree= 或 RuntimeKeepFree= 的限制，那么日志记录将被暂停。 也就是说，如果在创建日志文件时，文件系统有充足的空闲空间， 但是后来文件系统被其他非日志文件过多占用， 那么 systemd-journald 只会立即暂停日志记录， 但不会删除已经存在的日志文件。 注意，只会删除已归档的日志文件以释放空间。 也就是说，即使在完成日志清理之后， 日志所占用的空间仍然可能大于 SystemMaxUse= 或 RuntimeMaxUse= 的限制。

SystemMaxFileSize= 与 RuntimeMaxFileSize= 限制单个日志文件的最大体积， 到达此限制后日志文件将会自动滚动。 默认值是对应的 SystemMaxUse= 与 RuntimeMaxUse= 值的1/8 ， 这也意味着日志滚动 默认保留7个历史文件。

日志大小 可以使用以1024为基数的 K, M, G, T, P, E 后缀， 分别对应于 1024, 1024², … 字节。

SystemMaxFiles= 与 RuntimeMaxFiles= 限制最多允许同时存在多少个日志文件， 超出此限制后， 最老的日志文件将被删除， 而当前的活动日志文件 则不受影响。 默认值为100个。

* __MaxFileSec=__

日志滚动的时间间隔。 通常 并不需要使用基于时间的日志滚动策略， 因为由 SystemMaxFileSize= 与 RuntimeMaxFileSize= 控制的基于文件大小的日志滚动策略 已经可以确保日志文件的大小不会超标。 默认值是一个月， 设为零表示禁用基于时间的日志滚动策略。 可以使用 "year", "month", "week", "day", "h", "m" 时间后缀， 若不使用后缀则表示以秒为单位。

* __MaxRetentionSec=__

日志文件的最大保留期限。 当日志文件的最后修改时间(mtime)与当前时间之差， 大于此处设置的值时，日志文件将会被删除。 默认值零表示不使用基于时间的日志删除策略。 通常并不需要使用基于时间的日志删除策略，因为由 SystemMaxUse= 与 RuntimeMaxUse= 控制的基于文件大小的日志滚动策略 已经可以确保日志文件的大小不会超标。 可以使用 "year", "month", "week", "day", "h", "m" 时间后缀， 若不使用后缀则表示以秒为单位。

* __SyncIntervalSec=__

向磁盘刷写日志文件的时间间隔， 默认值是五分钟。 刷写之后，日志文件将会处于离线(OFFLINE)状态。 注意，当接收到 CRIT, ALERT, EMERG 级别的日志消息后， 将会无条件的立即刷写日志文件。 因此该设置仅对 ERR, WARNING, NOTICE, INFO, DEBUG 级别的日志消息有意义。

* __ForwardToSyslog=, ForwardToKMsg=, ForwardToConsole=, ForwardToWall=__

ForwardToSyslog= 表示是否将接收到的日志消息转发给传统的 syslog 守护进程，默认值为"no"。 如果设为"yes"，但是没有任何进程监听对应的套接字，那么这种转发是无意义的。 此选项可以被内核引导选项 "systemd.journald.forward_to_syslog" 覆盖。 ForwardToKMsg= 表示是否将接收到的日志消息转发给内核日志缓冲区(kmsg)，默认值为"no"。 此选项可以被内核引导选项 "systemd.journald.forward_to_kmsg" 覆盖。 ForwardToConsole= 表示是否将接收到的日志消息转发给系统控制台，默认值为"no"。 如果设为"yes"，那么可以通过下面的 TTYPath= 指定转发目标。 此选项可以被内核引导选项 "systemd.journald.forward_to_console" 覆盖。 ForwardToWall= 表示是否将接收到的日志消息作为警告信息发送给所有已登录用户，默认值为"yes"。 此选项可以被内核引导选项 "systemd.journald.forward_to_wall" 覆盖。

* __MaxLevelStore=, MaxLevelSyslog=, MaxLevelKMsg=, MaxLevelConsole=, MaxLevelWall=__

MaxLevelStore= 设置记录到日志文件中的最高日志等级，默认值为"debug"； MaxLevelSyslog= 设置转发给传统的 syslog 守护进程的最高日志等级， 默认值为"debug"； MaxLevelKMsg= 设置转发给内核日志缓冲区(kmsg)的最高日志等级，默认值为"notice"； MaxLevelConsole= 设置转发给系统控制台的最高日志等级，默认值为"info"； MaxLevelWall= 设置作为警告信息发送给所有已登录用户的最高日志等级，默认值为"emerg"； 这些选项既可以设为日志等级的名称， 也可以设为日志等级对应的数字： "emerg"(0), "alert"(1), "crit"(2), "err"(3), "warning"(4), "notice"(5), "info"(6), "debug"(7) 。 所有高于设定等级的日志消息都将被直接丢弃， 仅保存/转发小于等于设定等级的日志消息。 上述设置可以被如下内核引导选项覆盖： "systemd.journald.max_level_store=", "systemd.journald.max_level_syslog=", "systemd.journald.max_level_kmsg=", "systemd.journald.max_level_console=", "systemd.journald.max_level_wall="

* __ReadKMsg=__

是否收集内核日志。 默认值 yes 表示从 /dev/kmsg 中读取内核产生的日志消息。

* __TTYPath=__

指定 ForwardToConsole=yes 时所使用的控制台TTY， 默认值是 /dev/console

* __LineMax=__

在将日志流转化为日志记录时，每条日志记录最大允许的长度(字节)。 如果将单元的标准输出(STDOUT)/标准错误(STDERR)通过流套接字连接到日志中， 那么将会以换行符("\n", ASCII 10)与NUL字符("\0", ASCII 0)作为分割符， 把日志流切分成一条条独立的日志记录。 如果超过此处设置的长度之后仍然没有遇到分割符， 那么将会自动插入一个分割符，以强制将单行超长日志截断为多行。 此选项的值越大，每个日志流客户端日志守护进程占用的内存也越大(最大值等于此选项的值)。 另外，此选项的值太大也会造成与传统日志传输协议的不兼容(太长的日志无法封装在单个 AF_UNIX 或 AF_INET 报文内)。 此选项的值以字节为单位，同时也可以在数字的末尾加上 K, M, G, T 后缀(以1024为基准)。 默认值 48K 是一个足够大并且也能保持与传统日志传输协议兼容的值。 注意， 不能设为小于 79 的值(将被自动提升到79)。

## 常用命令

```shell
# 查看所有日志（默认情况下只保存本次启动的日志）
$ sudo journalctl

# 显示尾部指定行数的日志，默认10行
$ sudo journalctl -n 20

# 实时滚动显示最新日志
$ sudo journalctl -f

# 查看指定时间的日志
$ sudo journalctl --since yesterday
$ sudo journalctl --since="2023-12-22 16:52:18"
$ sudo journalctl --since "30 min ago"
$ sudo journalctl --since "2023-12-22 16:52:18" --until "2023-12-22 23:52:18"
$ sudo journalctl --since 09:00 --until "1 hour ago"

# 查看内核日志，过滤掉应用日志
$ sudo journalctl -k

# 查看系统本次启动的日志
$ sudo journalctl -b
$ sudo journalctl -b -0

# 查看上一次启动的日志
$ sudo journalctl -b -1

# 查看指定优先级及其以上级别的日志，共有8级 0: emerg 1: alert 2: crit 3: err 4: warning 5: notice 6: info 7: debug
# -b 不加任何参数时，表示显示当前引导周期的日志。这意味着只显示自最近一次启动以来的日志。
# -b N： N 是一个整数，表示要显示第 N 个引导周期的日志。例如，-b 0 表示显示最新的引导周期，-b 1 表示显示上一个引导周期，以此类推
$ sudo journalctl -p err -b

# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager

# 以单行 JSON 格式输出
$ sudo journalctl -b -u nginx.service -o json

# 以多行 JSON 可读性更好的格式输出
$ sudo journalctl -b -u nginx.service  -o json-pretty

# 查看指定服务的日志
$ sudo journalctl /usr/sbin/sshd

# 查看指定进程的日志
$ sudo journalctl _PID=1

# 查看某个路径的脚本的日志
$ sudo journalctl /bin/bash

# 查看指定用户的日志
$ sudo journalctl _UID=1000 --since today

# 查看某个 Unit 的日志
# 单元（unit）通常是 systemd 服务的一个抽象，用于表示系统中正在运行的各种服务或任务
$ sudo journalctl -u nginx.service
$ sudo journalctl -u nginx.service --since today

# 合并显示多个 Unit 的日志
$ journalctl -u nginx.service -u ssh.service --since today

# 显示日志占据的硬盘空间
$ sudo journalctl --disk-usage

# 仅保留500MB大小的日志文减
$ sudo journalctl --vacuum-size=500M

# 指定日志文件保存多久
$ sudo journalctl --vacuum-time=1years

# 仅保留最近一个月的日志文件
$ sudo journalctl --vacuum-time=1m

# 仅保留最近2天的日志文件
$ sudo journalctl --vacuum-time=2d
```

## 感谢

本文主要内容来源于网络，我只能算做整理，在此对链接作者表示感谢，如有侵权，请告知，谢谢！

* [《journald.conf 中文手册》](https://www.jinbuguo.com/systemd/journald.conf.html)

* [Linux环境下通过journal命令查看和管理日志](https://blog.csdn.net/albertsh/article/details/135161723)