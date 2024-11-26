---
title: Linux下的hosts.allow与hosts.deny介绍
author: mmy83
date: 2024-11-08 09:15:00 +0800
categories: [IT技术, 安全]
tags: [IT技术, linux, 安全, "allow", "deny"]
math: true
mermaid: true
image:
  path: /images/2024/11/2024-11-25/Linux下的hosts.allow与hosts.deny介绍/Linux下的hosts.allow与hosts.deny介绍-00.png
  lqip: data:image/webp;base64,UklGRkwAAABXRUJQVlA4IEAAAACwAQCdASoIAAUAAUAmJYwCdADye+QAAP73iF9G3+GvWgzgy2lS4Iwt/CEqgD9yQwk83EmzXzcGLbFYwYEsAAAA
  alt: Linux下的hosts.allow与hosts.deny介绍
---

## 介绍

&emsp;&emsp;在 Linux 系统中，hosts.deny和hosts.allow是Linux系统中用于访问控制的重要工具，起源于TCP Wrapper软件，旨在提供对网络服务的访问控制。这两个文件在系统安全性方面扮演关键角色，hosts.deny文件作为黑名单，用于拒绝特定主机或网络的访问，而hosts.allow文件作为白名单，用于允许特定主机或网络的访问。通过配置这两个文件，系统管理员可以限制或允许特定主机对服务器上的服务的访问，提高系统的安全性。

## 机制

&emsp;&emsp;Linux系统处理hosts.deny和hosts.allow的机制是基于TCP Wrapper的规则。当有一个连接请求到达时，系统首先检查hosts.allow文件，确定是否允许该请求的主机进行连接。如果主机在hosts.allow中找到匹配项，则连接会被接受。如果主机不在hosts.allow中找到匹配项，系统会继续检查hosts.deny文件。如果主机在hosts.deny中匹配，则连接将被拒绝。如果主机既不在hosts.allow也不在hosts.deny中匹配，则系统使用默认策略（通常是允许所有连接，也有说是拒绝的可能不同linux发行版本的区别，Centos7 和 ubuntu 默认这两个文件都是空的）。

- 默认情况下，这两个文件都不会主动限制任何访问。也就是说，如果 hosts.allow 和 hosts.deny 文件中都没有任何规则配置，系统会默认允许所有连接。

- 优先级顺序：系统会优先检查 hosts.allow，再检查 hosts.deny。

  - 如果某个访问请求匹配了 hosts.allow 中的允许规则，则立即允许访问，忽略 hosts.deny 中的配置。

  - 如果访问请求不符合 hosts.allow 中的任何规则，系统再检查 hosts.deny，若匹配到拒绝规则，则会拒绝访问。

  - 如果两者中都没有匹配的规则，则系统默认允许访问。

> 即：
>
> - hosts.allow 优先，定义允许的例外规则。
>
> - hosts.deny 用于补充，定义拒绝的例外规则。
{: .prompt-tip }

## 语法规则

&emsp;&emsp;/etc/hosts.allow 和 /etc/hosts.deny 文件的语法规则基本一致，都用于定义哪些客户端可以访问系统上的服务，以及哪些客户端不允许访问服务。以下是具体的语法规则和示例。

### 语法格式
>
> {% raw %}< 服务名 >: < 客户端地址 > [ : < 选项 > ]{% endraw %}
>
{: .prompt-tip }

- **服务名**：指定服务名称，通常与 /etc/services 中的名称一致。可以使用 ALL 表示所有服务。

- **客户端地址**：指定被允许或拒绝的客户端，可以是 IP 地址、主机名、域名或网段，也可以用 ALL 表示所有地址。

- **选项（可选）**：可以添加自定义命令或日志记录。通常以 spawn 或 twist 等关键字开头。

### 说明

1、服务名

可以是一个具体服务（如 sshd），或使用 ALL 代表所有服务。

2、客户端地址

指定允许或拒绝的客户端，可以包括：

IP 地址：如 192.168.1.100

网段：如 192.168.1.，表示以 192.168.1. 开头的所有地址

主机名：如 example.com

域名：以 . 开头表示域，如 .example.com，代表来自该域的所有主机

关键字 ALL：表示所有客户端

3、选项（可选）

在规则后面可以添加选项，比如记录日志、发送自定义消息、执行特定命令等。

spawn：可以在访问控制生效时执行命令。

twist：将客户端请求重定向到其他服务或显示自定义消息。

## 示例

1、在 /etc/hosts.allow 中允许特定的客户端访问服务

```console
sshd: 192.168.1.100 # 允许 IP 192.168.1.100 访问 sshd 服务 
httpd: .example.com # 允许来自 example.com 域的所有主机访问 httpd 服务 
ALL: LOCAL # 允许本地网络的所有服务 
```

2、在 /etc/hosts.deny 中拒绝特定的客户端访问服务

```console
sshd: ALL # 拒绝所有客户端访问 sshd 服务 
telnet: 192.168.1.200 # 拒绝 IP 192.168.1.200 访问 telnet 服务 
ALL: .malicious.com # 拒绝 malicious.com 域中的所有客户端访问所有服务 
```

3、添加日志或自定义命令

```console
sshd: 192.168.1.100 : spawn (/bin/echo "Access granted to %c" >> /var/log/ssh_access.log) 
```

这条规则允许 192.168.1.100 访问 sshd，并将 "Access granted to %c" 写入日志文件。

4、重定向拒绝请求

```console
sshd: 192.168.1.200 : twist /bin/echo "Access denied" 
```

这条规则在拒绝 192.168.1.200 访问 sshd 时，返回 "Access denied" 消息。

## 注意事项

- **优先级**：hosts.allow 中的规则优先级高于 hosts.deny。如果在 hosts.allow 中找到匹配规则，则直接允许访问，不会检查 hosts.deny。

- **测试生效情况**：可以通过 /var/log/auth.log（或系统日志）查看访问控制规则的生效情况。

这种语法和规则机制提供了灵活的访问控制方式，在配置多个服务的权限时尤其有效。

> 提示：
>
> 在配置 hosts.allow 和 hosts.deny 文件时，不需要重启服务即可生效，看网上有些人说已经登录的不会受到影响，但是我在测试的过程中发行保存文件后就会立刻退出并要求重新登录(可能是不同linux发行版本间的区别)，所有在配置的时候还是要注意，避免配置错误导致登录不上去。
>
{: .prompt-tip }

## 扩展

### TCP Wrappers概述

TCP Wrapper有着轻量级的防火墙应用之称。libwrap 库是一个独立的库文件，通常以共享库的形式存在，一般命名为 libwrap.so。在 CentOS 或其他 Linux 系统上，这个文件的路径通常是 /lib64/libwrap.so 或 /usr/lib64/libwrap.so，取决于系统架构和库的安装位置。

libwrap 提供了 tcp_wrappers 的核心功能，使服务程序可以调用其函数（如 hosts_access() 和 hosts_ctl()）来解析 /etc/hosts.allow 和 /etc/hosts.deny 中的规则。因此，只要应用程序在编译时链接了 libwrap 库，就可以直接使用 tcp_wrappers 进行访问控制，而不需要额外的配置。

```console
[root@ct7_node01 pam.d]# ldd /usr/sbin/sshd | grep libwrap
libwrap.so.0 => /lib64/libwrap.so.0 (0x00007fd835824000)
```

### TCP Wrappers VS PAM

tcp_wrappers 并不是 PAM（Pluggable Authentication Module）中的一个模块。它是一种独立的访问控制机制，主要通过 libwrap 库提供对 TCP 服务的访问限制。tcp_wrappers 通过 libwrap 函数实现对 /etc/hosts.allow 和 /etc/hosts.deny 的解析，并根据规则决定是否允许客户端连接特定服务。

PAM 模块，如 pam_access.so 或 pam_hosts.so，可以提供类似的基于主机的访问控制功能，但它们与 tcp_wrappers 是独立的。事实上，某些服务（如 sshd）本身直接集成了对 tcp_wrappers 的支持，而不依赖 PAM，因此可以在没有 PAM 模块的情况下直接使用 /etc/hosts.allow 和 /etc/hosts.deny 文件来控制访问。

## 参考资料

1、[hosts.allow与hosts.deny详解](https://blog.csdn.net/zyqash/article/details/143337333)
