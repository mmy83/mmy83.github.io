---
title: Linux日志管理-logrotate简介
author: mmy83
date: 2024-11-08 09:15:00 +0800
categories: [IT技术, Linux]
tags: [IT技术, linux, 日志, "logrotate", 切割]
math: true
mermaid: true
image:
  path: /images/2024/11/2024-11-08/Linux日志管理-logrotate简介/Linux日志管理-logrotate简介-00.png
  lqip: data:image/webp;base64,UklGRkYAAABXRUJQVlA4IDoAAACwAQCdASoIAAYAAUAmJQBOgCP/2k0QAP77SDYSx2/tnVURvq9zm0YFkwP3DyMRLbc7oDBBludPAAAA
  alt: Linux日志管理-logrotate简介
---

## 介绍

&emsp;&emsp;logrotate旨在简化生成大量日志文件的系统上日志文件的管理。logrotate允许自动滚动、压缩、删除和邮寄日志文件。logrotate可以设置为每小时、每天、每周、每月或当日志文件达到一定大小时处理日志文件。因此，logrotate对于维护系统的稳定性和可靠性非常重要。它可以确保系统管理员能够及时发现和解决潜在的问题，并避免因日志文件过大而导致的性能问题。用于分割日志文件，删除旧的日志文件，并创建新的日志文件，起到“转储”作用，可以节省磁盘空间；是linux里内置的日志分割工具，若没有可默认安装logrotate包

## 机制

&emsp;&emsp;logrotate的运行依赖于Linux上的定时任务命令crontab。crontab命令会定时运行logrotate，一般是每天一次。crontab会每天定时执行/etc/cron.daily目录下的脚本，而这个目录下有个文件叫logrotate。

```shell
#cat /etc/cron.daily/logrotate

#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

&emsp;&emsp;logrotate会读取配置文件```/etc/logrotate.conf```和```/etc/logrotate.d/```目录下的文件，这些配置文件中定义了所有日志文件的轮转规则。logrotate会根据这些规则对日志文件进行轮转。

## 安装

```shell
# 一般linux都自带了logrotate包，如果不带需要安装
# Debian或Ubuntu安装
apt-get install logrotate cron

# Fedora,CentOS或RHEL安装
yum install logrotate crontabs
```

## 使用

```shell
logrotate --help
Usage: logrotate [OPTION...] <configfile>
  -d, --debug               debug模式，测试配置文件是否有错误。(可配合 -v)
  -f, --force               强制转储文件。
  -m, --mail=command        压缩日志后，发送日志到指定邮箱。（而不是'/bin/mail'）
  -s, --state=statefile     使用指定的状态文件。
  -v, --verbose             显示转储过程信息。
  -l, --log=STRING          日志文件
  --version                 版本

Help options:
  -?, --help                帮助
  --usage                   显示简要使用信息
```

&emsp;&emsp;命令简单使用方便，直接指定配置文件即可。

```shell
# 测试配置文件，不会做任何切割或转存
logrotate -d 配置文件

# 手动强制执行，并可以看到转存过程信息
logrotate -vf 配置文件
```

## 配置文件

```shell
[root@ceph01 ceph]# cat /etc/logrotate.conf 
# see "man logrotate" for details
# rotate log files weekly
weekly # 默认每周执行一次rotate轮转任务
 
# keep 4 weeks worth of backlogs
rotate 4 # 保留多少日志文件，默认保留4个，0表示没有备份
 
# create new (empty) log files after rotating old ones
create # 自动创建新的日志文件，新的日志文件具有和原来的文件相同的权限；因为日志被改名,因此要创建一个新的来继续存储之前的日志
 
# use date as a suffix of the rotated file
dateext # 切割后日志文件以当前日期为后缀结尾，如xxx.log-20230912，如果注释掉，切割出来是按数字递增，即xxx.log-1这种格式
 
# uncomment this if you want your log files compressed
#compress # 是否通过gzip压缩转储以后的日志文件，如xxx.log-20131216.gz ；如果不需要压缩，注释掉
 
# RPM packages drop log rotation information into this directory
include /etc/logrotate.d # 将/etc/logrotate.d/ 目录中的所有文件都加载进来
 
# system-specific logs may be also be configured here.
```

## 参数说明

|参数|说明|
|-------|------|
|compress|通过gzip 压缩转储以后的日志|
|nocompress|                                不做gzip压缩处理|
|copytruncate|                              用于还在打开中的日志文件，把当前日志备份并截断；是先拷贝再清空的方式，拷贝和清空之间有一个时间差，可能会丢失部分日志数据。|
|nocopytruncate|                           备份日志文件不过不截断|
|create mode owner group|             轮转时指定创建新文件的属性，如create 0777 nobody nobody|
|nocreate|                                    不建立新的日志文件|
|delaycompress|                           和compress 一起使用时，转储的日志文件到下一次转储时才压缩|
|nodelaycompress|                        覆盖 delaycompress 选项，转储同时压缩。|
|missingok|                                 如果日志丢失，不报错继续滚动下一个日志|
|errors address|                           专储时的错误信息发送到指定的Email 地址|
|ifempty|                                    即使日志文件为空文件也做轮转，这个是logrotate的缺省选项。|
|notifempty|                               当日志文件为空时，不进行轮转|
|mail address|                             把转储的日志文件发送到指定的E-mail 地址|
|nomail|                                     转储时不发送日志文件|
|olddir directory|                         转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统|
|noolddir|                                   转储后的日志文件和当前日志文件放在同一个目录下|
|sharedscripts|                           运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本|
|prerotate|                                 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行|
|postrotate|                               在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行|
|daily|                                       指定转储周期为每天|
|weekly|                                    指定转储周期为每周|
|monthly|                                  指定转储周期为每月|
|rotate count|                            指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份|
|dateext|                                  使用当期日期作为命名格式|
|dateformat %Y%m%d%H.%s|                       配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数|
|size(或minsize) log-size|            当日志文件到达指定的大小时才转储，log-size能指定bytes(缺省)及KB (sizek)或MB(sizem). <br /> 当日志文件 >= log-size 的时候就转储。 以下为合法格式：（其他格式的单位大小写没有试过）<br /> size = 5 或 size 5 （>= 5 个字节就转储）<br /> size = 100k 或 size 100ksize = 100M 或 size 100M|

## 日志滚动原理

&emsp;&emsp;在linux中，一切皆文件，目录也是文件，目录文件里存着文件名和对应的```inode```编号。通过这个```inode```编号可以查到文件的元数据和文件内容。文件的元数据有引用计数、操作权限、拥有者ID、创建时间、最后修改时间等等。文件名并不在元数据里而是在目录文件中。因此文件改名、移动，都不会修改文件，而是修改目录文件。

![文件系统](/images/2024/11/2024-11-08/Linux日志管理-logrotate简介/Linux日志管理-logrotate简介-01.png)

&emsp;&emsp;进程每新打开一个文件，系统会分配一个新的文件描述符给这个文件。文件描述符对应着一个文件表。表里面存着文件的状态信息（```O_APPEND```/```O_CREAT```/```O_DIRECT```/...）、当前文件位置和文件的```inode```信息。系统会为每个进程创建独立的文件描述符和文件表，不同进程是不会共用同一个文件表。正因为如此，不同进程可以同时用不同的状态操作同一个文件的不同位置。文件表中存的是```inode```信息而不是文件路径，所以文件路径发生改变不会影响文件操作。

### 方案一

&emsp;&emsp;这个方案的思路是重命名原日志文件，创建新的日志文件。详细步骤如下：

&emsp;&emsp;重命名程序当前正在输出日志的程序。因为重命名只会修改目录文件的内容，而进程操作文件靠的是```inode```编号，所以并不影响程序继续输出日志。
创建新的日志文件，文件名和原来日志文件一样。虽然新的日志文件和原来日志文件的名字一样，但是```inode```编号不一样，所以程序输出的日志还是往原日志文件输出。

&emsp;&emsp;通过某些方式通知程序，重新打开日志文件。程序重新打开日志文件，靠的是文件路径而不是```inode```编号，所以打开的是新的日志文件。
什么方式通知程序我重新打开日志呢，简单粗暴的方法是杀死进程重新打开。很多场景这种作法会影响在线的服务，于是有些程序提供了重新打开日志的接口，比如可以通过信号通知nginx。各种IPC方式都可以，前提是程序自身要支持这个功能。

&emsp;&emsp;有个地方值得一提，一个程序可能输出了多个需要滚动的日志文件。每滚动一个就通知程序重新打开所有日志文件不太划得来。有个```sharedscripts```的参数，让程序把所有日志都重命名了以后，只通知一次。

### 方案二

&emsp;&emsp;如果程序不支持重新打开日志的功能，又不能粗暴地重启程序，怎么滚动日志呢？```copytruncate```的方案出场了。

&emsp;&emsp;这个方案的思路是把正在输出的日志拷(copy)一份出来，再清空(trucate)原来的日志。详细步骤如下：

&emsp;&emsp;拷贝程序当前正在输出的日志文件，保存文件名为滚动结果文件名。这期间程序照常输出日志到原来的文件中，原来的文件名也没有变。
清空程序正在输出的日志文件。清空后程序输出的日志还是输出到这个日志文件中，因为清空文件只是把文件的内容删除了，文件的```inode```编号并没有发生变化，变化的是元信息中文件内容的信息。
&emsp;&emsp;结果上看，旧的日志内容存在滚动的文件里，新的日志输出到空的文件里。实现了日志的滚动。

### 这个方案有两个有趣的地方

文件清空并不影响到输出日志的程序的文件表里的文件位置信息，因为各进程的文件表是独立的。那么文件清空后，程序输出的日志应该接着之前日志的偏移位置输出，这个位置之前会被\0填充才对。但实际上logrotate清空日志文件后，程序输出的日志都是从文件开始处开始写的。这是怎么做到的？这个问题让我纠结了很久，直到某天灵光一闪，这不是logrotate做的，而是成熟的写日志的方式，都是用```O_APPEND```的方式写的。如果程序没有用```O_APPEND```方式打开日志文件，变会出现```copytruncate```后日志文件前面会被一堆\0填充的情况。
日志在拷贝完到清空文件这段时间内，程序输出的日志没有备份就清空了，这些日志不是丢了吗？是的，```copytruncate```有丢失部分日志内容的风险。所以能用create的方案就别用```copytruncate```。所以很多程序提供了通知我更新打开日志文件的功能来支持create方案，或者自己做了日志滚动，不依赖logrotate。

## 案例

&emsp;&emsp;这里给出两个例子，方便使用。

### Nginx

```console
# cat /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    create 0640 nginx root
    daily
    rotate 10
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```

### Redis

```console
# cat /etc/logrotate.d/redis
/var/log/redis/*.log {
    weekly
    rotate 10
    copytruncate
    delaycompress
    compress
    notifempty
    missingok
}
```

## 参考资料

1、[开源代码](https://github.com/logrotate/logrotate)

2、[Linux日志管理-logrotate（crontab定时任务、Ceph日志转储）](https://www.cnblogs.com/gengduc/p/17725359.html)
