---
title: 串口工具：Minicom和Screen
author: mmy83
date: 2025-04-09 22:31:00 +0800
categories: [IT技术, 软件]
tags: [IT技术, 软件, 串口, minicom, com, screen]
math: true
mermaid: true
image:
  path: /images/2025/04/2025-04-09/串口工具：Minicom和Screen/串口工具：Minicom和Screen.png
  lqip: data:image/webp;base64,UklGRlgAAABXRUJQVlA4IEwAAADQAQCdASoIAAgAAUAmJaQAAvd7lGJAAAD+/hJNVZSxlCrG5z9wqPa2j/Dzsn387d7OLXte8DRYhLXhR1cdg30BOXlXuScqCQvKCgAA
  alt: 串口工具：Minicom和Screen
---

## 背景

最近接触了一下摄像头，设计到单片机的开发，于是就需要通过串口连接到单片机上，用到了两款评价不错的串口工具，做一下记录。

## 查看串口设备

```shell
# 通过 usb 接口连接
# mac
# 在类unix系统中，一切皆文件，串口设备也不例外，可以使用 ls 命令查看 /dev目录下的设备
# 会看到一个形如 /dev/tty.usbserial-xxxx 这样的设备，这就是串口设备
# 我识别的方式是，拔下了执行命令，然后插上在执行，看看
ls /dev/tty.*

# linux好像有所不同
ls /dev/ttyUSB* # 查看 ttyUSB 开头的串口设备
```

## Minicom

Minicom 是一个广泛使用的基于文本的用户界面（TUI）的串口通信程序。它支持基本的串口配置选项，如波特率、数据位、停止位和校验位等。Minicom 简单易用，对于简单的串口调试和通信任务来说非常实用。

### 安装Minicom

```shell
# macOS
brew install minicom

# ubuntu
apt install minicom
```

### 使用Minicom

```shell
# 115200 是波特率，如果您的设备使用不同的波特率，则需要将其替换为相应的值。
minicom -D /dev/tty.usbserial-XXXXXX -b 115200
```

## Screen

Screen 虽然最初设计用于终端多路复用，但它也支持串口通信，并提供了强大的会话管理功能。Screen 对于需要同时管理多个串口会话或长时间运行的串口连接的用户来说非常有用。它提供了丰富的快捷键和命令，方便用户在不同会话之间切换和管理。

### 安装Screen

```shell
# macOS
brew install screen
# ubuntu
apt-get install screen
```

### 使用Screen

```shell
screen /dev/tty.usbserial-XXXXXX 115200
```

## 参考资料

1. [Ubuntu 下串口工具：Minicom、CuteCom 和 Screen](https://blog.csdn.net/weixin_43978579/article/details/138572627)

2. [MAC（M2芯片）上如何打开串口](https://blog.csdn.net/u012855585/article/details/145598206)

3. [MacBook通过Minicom连接串口](https://blog.csdn.net/2203_75758128/article/details/129640306)

4. [Mac终端自带screen连接串口终端](https://blog.csdn.net/fzxhub/article/details/118539712)
