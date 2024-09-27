---
title: Server-Sent Events
author: mmy83
date: 2024-09-27 12:04:00 +0800
categories: [IT技术, 网络]
tags: [IT技术, 网络, 编程, python, stream, 流, sse, "规范"]
math: true
mermaid: true
image:
  path: /images/2024/09/2024-09-27/Server-Sent Events/Server-Sent Events）-00.png
  lqip: data:image/webp;base64,UklGRjgAAABXRUJQVlA4ICwAAACQAQCdASoIAAMAAUAmJaQAAp1HI1AA/v3Yr4yctCyfUNvsM+dTPiS3e0AAAA==
  alt: Server-Sent Events
---

## 背景

&emsp;&emsp;最近一直研究AI，在做聊天接口对接的时候，有些平台是用流的方式返回的，第一次用流，觉得应该直接推流就可以了，但是到前端对接的时候问题出现了，我接收的流是一条条的，可是当我再推给前端的时候，流就不再是一条条的，前端收到的数据要么是连着好几条，要么是半条，这明显不符合我的需求，于是就研究了一下流。在朋友的提醒下，又看了对接平台的SDK，发现SDK中用到了Server-Sent Events，于是就研究了一下。

## 介绍

&emsp;&emsp;SSE（Server-Sent Events）是一种允许服务器向客户端浏览器推送信息的技术。它是 HTML5 的一部分，专门用于建立一个单向的从服务器到客户端的通信连接。SSE的使用场景非常广泛，包括实时消息推送、实时通知更新等。

![Server-Sent Events流程](/images/2024/09/2024-09-27/Server-Sent Events/Server-Sent Events）-01.png)

### SSE 的本质

&emsp;&emsp;严格地说，HTTP 无法做到服务器主动推送信息。但是，有一种变通方法，就是服务器向客户端声明，接下来要发送的是流信息（streaming）。也就是说，发送的不是一次性的数据包，而是一个数据流，会连续不断地发送过来。这时，客户端不会关闭连接，会一直等着服务器发过来的新的数据流，视频播放就是这样的例子。本质上，这种通信就是以流信息的方式，完成一次用时很长的下载。SSE 就是利用这种机制，使用流信息向浏览器推送信息。它基于 HTTP 协议，目前除了 IE/Edge，其他浏览器都支持。

### 特点

+ 持续连接：与传统的 HTTP 请求不同，SSE 保持连接开放，服务器可以随时发送消息。

+ 文本数据流：SSE 主要传输文本数据，这些数据以特定的格式流式传输，使得每条消息都是简单的文本格式。

+ 内置重连机制：浏览器会自动处理连接中断和重连，包括在重连请求中发送最后接收的事件 ID，以便服务器从正确的位置恢复发送事件。

+ 简单的客户端处理：在浏览器中，使用 JavaScript 的 EventSource 接口处理 SSE 非常简单，只需几行代码即可监听服务器发来的事件。

### 工作原理

+ 建立连接：客户端通过创建一个 EventSource 对象请求特定的 URL 来启动 SSE 连接。这个请求是一个标准的 HTTP 请求，但会要求服务器以特定方式响应。

+ 服务器响应：服务器响应必须设置 Content-Type 为 text/event-stream，然后保持连接打开。

+ 发送消息：服务器可以通过持续发送数据格式为特定事件流的消息来推送更新。每个消息包括一个可选的事件类型、数据和一个可选的 ID。

+ 数据：实际的消息内容，以 data: 开头，多行数据以双换行符 \n\n 结束。

+ 事件类型：允许客户端根据事件类型来监听，以 event: 开头。

+ ID：如果连接中断，客户端将发送包含上次接收的最后一个ID的 Last-Event-ID 头，以便服务器从断点继续发送数据。

## 实践

### 后端（python）

```python
# views.py
def chat(request):

    def create():
        for i in range(0,10):
            time.sleep(0.1)
            yield 'data: 这是第 {} 条数据\n\n'.format(i)
        yield "event: close\ndata: [DONE]\n\n"

    resp = StreamingHttpResponse(create(), content_type="text/event-stream")
    resp['Cache-Control'] = 'no-cache'
    return resp

# url设置为

```

### 前端（node后端代码）

```shell
# node后端调用需要安装依赖，浏览器则不需要，因为大部分浏览器都内置了 EventSource
npm install eventsource
npm install event-source-polyfill
```

```javascript
// index.js
// 引入event-source-polyfill
// node后端调用需要安装依赖，浏览器则不需要，因为大部分浏览器都内置了 EventSource
import { NativeEventSource, EventSourcePolyfill } from 'event-source-polyfill';

const EventSource = NativeEventSource || EventSourcePolyfill;
// OR: may also need to set as global property
global.EventSource =  NativeEventSource || EventSourcePolyfill;
```

```javascript

const evtSource = new EventSource("http://127.0.0.1:8000/ai/chat");

// 监听服务端发送的数据
evtSource.onmessage = function(event) {
    console.log(event.type + ': ' + event.data);
};

evtSource.onerror = function(event) {
    console.log(event.type + ': ' + event.data);

    // 服务器关闭了连接，这里会报错，需要这里手动断开连接，否则他会重新连接
    evtSource.close();
};

```

## 参考资料

1、[Rust 实战丨Server-Sent Events（SSE）](https://zhuanlan.zhihu.com/p/702069988)

2、[EventSource手册](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

3、[Server-Sent Events 教程](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)

4、[Server-Sent Events](https://python-abc.xyz/fe/5189)

## 感谢

感情QQ群里的兄弟!
