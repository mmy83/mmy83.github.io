---
title: Nginx限速：limit_req、limit_conn、limit_rate
author: mmy83
date: 2024-08-06 10:15:00 +0800
categories: [IT技术, "软件"]
tags: [IT技术, "软件", 'nginx', '限速', '限流', limit_req, limit_conn, limit_rate]
math: true
mermaid: true
image:
  path: /images/2024/08/2024-08-06/Nginx限速：limit_req、limit_conn、limit_rate/Nginx限速：limit_req、limit_conn、limit_rate-00.png
  lqip: data:image/webp;base64,UklGRkoAAABXRUJQVlA4ID4AAADwAQCdASoIAAQAAUAmJaACdLoAAwkG+4AA/sFq+IRjGvkyLAtDXcWxTP0KNlExT/+ssZL/ozf/MsZL6hgAAA==
  alt: Nginx限速：limit_req、limit_conn、limit_rate
---

## 前言

&emsp;&emsp;通过Nginx进行网站限速，常用三个方法：

+ 限制请求数(request):```limit_req```

+ 限制连接数(connection):```limit_conn```

+ 限制响应速度(rate):```limit_rate```

## 原理

### 固定窗口算法(ngx_http_limit_req_module)

&emsp;&emsp;定义一个请求速率（比如每秒允许的请求数量）和一个时间窗口。当请求到达时，Nginx 会检查在当前的时间窗口内是否已经达到了设置的速率限制。如果请求超出了速率限制，Nginx 可以配置为返回错误状态码（如 503 Service Temporarily Unavailable）或者延迟处理请求。

### 漏桶算法(ngx_http_limit_conn_module)

&emsp;&emsp;这个模块限制了同时处理的连接数，可以用来限制对某个资源（如服务器、数据库等）的并发访问数。当达到限制的连接数时，新的连接会被拒绝，直到现有的连接数降低。

### 令牌桶算法(ngx_http_limit_req_module)

&emsp;&emsp;令牌桶算法通过固定速率向桶中添加令牌。每个请求都需要消耗一个令牌才能被处理。如果桶中有足够的令牌，请求将立即被处理；如果没有令牌，则请求可以被延迟处理或拒绝。这种算法允许一定程度的突发流量，因为桶中可以积累令牌以应对短时间内的请求峰值。

## limit_req

&emsp;&emsp;在```ngx_http_limit_req_module```模块中，有五个指令：```limit_req_zone```、```limit_req```、```limit_req_dry_run```、```limit_req_log_level```、```limit_req_status```,分别用于定义内存空间及限速、超出的处理方式、运行模式、日志级别、拒绝状态。

&emsp;&emsp;限速的思路是先用```limit_req_zone```开辟一定大小的内存空间，并确定内存空间内存储的数据的键和限制速度。然后通过```limit_req```来设置超出限制的处理办法。并可以通过```limit_req_dry_run```、```limit_req_log_level```、```limit_req_status```来设置运行模式、日志级别、拒绝状态。

+ ```limit_req_zone```：设置共享内存区域的参数，该区域将保存各种键的状态。可以key包含文本、变量及其组合。键值为空的请求不予处理。

> 句法: ```limit_req_zone key zone=name:size rate=rate [sync];```
>
> 默认: —
>
> 语境: http
{: .prompt-info }

```nginx
# 开辟一个10M的内存空间，并命名为one，以用户的二进制ip地址为key，并限制速度为1r/s
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    limit_req_zone $server_name zone=perserver:10m rate=10r/s;
    server {
        location /search/ {

        }
    }
}
```

+ ```limit_req```：设置共享内存区域和请求的最大突发大小。如果请求速率超过为区域配置的速率，则会延迟处理请求，以便以定义的速率处理请求。过多的请求将被延迟，直到其数量超过最大突发大小，在这种情况下，请求将以错误终止 。默认情况下，最大突发大小等于零。

> 句法： ```limit_req zone=name [burst=number] [nodelay | delay=number];```
>
> 默认： —
>
> 语境： http，server，​location
{: .prompt-info }

```nginx
# 开辟一个10M的内存空间，并命名为one，以用户的二进制ip地址为key，并限制速度为1r/s
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    server {
        location /search/ {
            # 限制最大突发大小为5，当请求速率超过1r/s时，将延迟处理请求，直到请求数量超过5个，超过5个后将拒绝请求
            limit_req zone=one burst=5; 
            # 限制最大突发大小为5，当请求速率超过1r/s时，不延迟处理请求，直到请求数量超过5个，超过5个后将拒绝请求
            limit_req zone=one burst=5 nodelay; 
            # 限制最大突发大小为5，当请求速率超过1r/s时，前3个不延迟处理请求，后两个延迟处理，直到请求数量超过5个，超过5个后将拒绝请求
            limit_req zone=one burst=5 delay=3; 
        }
    }
}
```

+ ```limit_req_dry_run```：启用试运行模式。此模式下，请求处理速率不受限制，但在共享内存区域，超出的请求数仍按正常方式计算。

> 句法： ```limit_req_dry_run on | off;```
>
> 默认： ```limit_req_dry_run off;```
>
> 语境： http，server，​location
>
> 该指令出现在1.17.1版本中。
{: .prompt-info }

+ ```limit_req_log_level```：设置服务器因速率超出而拒绝处理请求或延迟处理请求时所需的日志记录级别。延迟的日志记录级别比拒绝的日志记录级别低一个点。

> 句法： ```limit_req_log_level info | notice | warn | error;```
>
> 默认： ```limit_req_log_level error;```
>
> 语境： http，server，​location
>
> 该指令出现在0.8.18版本中。
{: .prompt-info }

+ ```limit_req_status```：设置响应被拒绝的请求而返回的状态代码。

> 句法： ```limit_req_status code;```
>
> 默认： ```limit_req_status 503;```
>
> 语境： http，server​，location
>
> 该指令出现在1.3.15版本中。
{: .prompt-info }

## limit_conn

&emsp;&emsp;在```ngx_http_limit_conn_module```模块中，包括```limit_conn```、```limit_conn_dry_run```、```limit_conn_log_level```、```limit_conn_status```、```limit_conn_zone```五个指令。

&emsp;&emsp;限速的思路是先用```limit_conn_zone```开辟一定大小的内存空间，并确定内存空间内存储的数据的键，然后通过```limit_conn```指令来设置这个内存空间记录的键一定时间内允许通过多少个连接数。并可以设置：日志级别```limit_conn_log_level```、运行模式```limit_conn_dry_run```、拒绝的状态```limit_conn_status```。

+ ```limit_conn_zone```：设置共享内存区域的参数，该区域将保存各种键的状态。具体来说，状态包括当前连接数。可以key包含文本、变量及其组合。键值为空的请求不予处理。

> 句法： ```limit_conn_zone key zone=name:size;```
>
> 默认： —
>
> 语境： http
{: .prompt-info }

```nginx
# 开辟一个10M的内存空间，并命名为addr，以用户的二进制ip地址为key
# 这样就可以记录每一个ip地址的连接数，并限制连接数。
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
        location /download/ {

        }
    }
}
```

+ ```limit_conn```：设置共享内存区域和给定键值的最大允许连接数。当超出此限制时，服务器将返回错误响应。具有继承能力，当且仅当当前级别上 没有定义指令时，这些指令才会从前一个配置级别继承。

> 句法： ```limit_conn zone number;```
>
> 默认： —
>
> 语境： http，server，​location
{: .prompt-info }

```nginx
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
        location /download/ {
            limit_conn addr 1;
        }
    }
}
```

+ ```limit_conn_dry_run```：启用试运行模式。此模式下，连接数不受限制，但在共享内存区域，超出的连接数将照常计算。

> 句法： ```limit_conn_dry_run on | off;```
>
> 默认： ```limit_conn_dry_run off;```
>
> 语境： http，server​，location
>
> 该指令出现在1.17.6版本中。
{: .prompt-info }

+ ```limit_conn_status```：设置响应被拒绝的请求而返回的状态代码。

> 句法： ```limit_conn_status code;```
>
> 默认： ```limit_conn_status 503;```
>
> 语境：http，server，​location
>
> 该指令出现在1.3.15版本中。
{: .prompt-info }

+ ```limit_conn_log_level```：当服务器限制连接数时，设置所需的日志记录级别。

> 句法： ```limit_conn_log_level info | notice | warn | error;```
>
> 默认： ```limit_conn_log_level error;```
>
> 语境： http，server，​location
>
> 该指令出现在0.8.18版本中。
{: .prompt-info }

## limit_rate

&emsp;&emsp;```limit_rate```并不是一个模块，而是在```ngx_http_core_module```模块中。他包含两个指令：```limit_rate_after```、```limit_rate```，使用就很简单了，而且对于限制下载速度来说，使用```limit_rate```是非常合适的。

+ ```limit_rate_after```：会在客户端成功建立连接之后，指定的大小后开始限制发送速度。这个指令的含义就是在连接建立后的 limit_rate_after 大小之后，数据发送速率将被限制。

> 语法: ```limit_rate rate;```
>
> 默认: ```limit_rate 0;```
>
> 上下文: http, server, location, if in location
{: .prompt-info }

+ 语法：```limit_rate```：设置之后的下载速度，单位是字节/秒。

> 语法: ```limit_rate_after size;```
>
> 默认值: ```limit_rate_after 0;```
>
> 上下文: http, server, location, if in location
>
> This directive appeared in version 0.8.0.
{: .prompt-info }

```nginx
# 50M以内不限速，超过50M限制下载速度为 1k/s
location / {
  limit_rate_after 50m;
  limit_rate 1k;
  root html;
}
```

## 参考

+ [玩转Nginx之限流-三大限流算法实现](https://baijiahao.baidu.com/s?id=1799106894031801548)

+ [官方手册#Module ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

+ [官方手册#Module ngx_http_limit_conn_module](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)

+ [官方手册#limit_rate_after](http://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate_after)

+ [官方手册#limit_rate](http://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate)
