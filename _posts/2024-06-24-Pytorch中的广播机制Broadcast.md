---
title: Pytorch中的广播机制Broadcast
author: mmy83
date: 2024-06-24 11:54:00 +0800
categories: [专题, "PyTorch"]
tags: [AI, 人工智能, 'PyTorch', 框架, 机器学习, 深度学习, 广播, Broadcast]
math: true
mermaid: true
image:
  path: /images/2024-06-24/Pytorch中的广播机制Broadcast/Pytorch中的广播机制Broadcast-00.jpg
  lqip: data:image/webp;base64,UklGRjIAAABXRUJQVlA4ICYAAAAwAQCdASoIAAYAAUAmJaQAA3AA/vzdZpLY794K8Pna2+WVzcpAAA==
  alt: Pytorch中的广播机制Broadcast
---

## 前言

&emsp;&emsp;在学习 PyTorch 的过程中，遇到计算的时候经常会遇到广播机制（Broadcast），而Pytorch官方文档中关于广播机制的介绍比较简单，而且说明支持的是```NumPy```的广播机制。

![PyTorch广播介绍](/images/2024-06-24/Pytorch中的广播机制Broadcast/Pytorch中的广播机制Broadcast-01.png)

## 广播的意义

&emsp;&emsp;在机器学习过程中，张量的计算往往是有维度要求的，需要满足一定的要求才能进行计算。在实际操作的时候就需要为了满足这种要求进行一些操作，如维度扩展。为了计算方便，就引入了广播机制（Broadcast），简化计算。

## 适用情况

- 每个张量至少有一个维度。
- 在维度大小上迭代时，从尾部维度开始，维度大小必须相等，其中一个为1，或者其中一个不存在。

&emsp;&emsp;首先广播的时候他不会给张量胡乱赋值，而是扩展（复制）已有的数据，所有这里必须有一个维度的数据作为复制的基础。

&emsp;&emsp;第二，在扩展的时候他不会扩展一半或者部分，也不会填充部分，而是把0或者1变n的过程。如：一个shape(3,3)可以变成shape(1,3,3)或者shape(3,3,3),但不能变成shape(4,3)。

## 实例

```python
A      (2d array):  5 x 4
B      (1d array):      1
Result (2d array):  5 x 4

A      (2d array):  5 x 4
B      (1d array):      4
Result (2d array):  5 x 4

A      (3d array):  15 x 3 x 5
B      (3d array):  15 x 1 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 5
Result (3d array):  15 x 3 x 5

A      (3d array):  15 x 3 x 5
B      (2d array):       3 x 1
Result (3d array):  15 x 3 x 5
```

&emsp;&emsp;通过上面的例子可以更好的理解，在维度大小上迭代时，从尾部维度开始，维度大小必须相等，其中一个为1，或者其中一个不存在。要么相等，要么为1，要么不存在。

## 相关资料

- [Numpy官方文档](https://numpy.org/doc/stable/user/basics.broadcasting.html)
- [PyTorch官方文档](https://pytorch.org/docs/stable/notes/broadcasting.html)
