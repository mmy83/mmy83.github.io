---
title: javascript基础-minimist命令行参数解析
author: mmy83
date: 2024-03-25 20:50:00 +0800
categories: [编程, javascript]
tags: [minimist, 命令行, 参数解析, javascript]
math: true
mermaid: true
image:
  path: /images/2024-03-25/javascript基础-minimist命令行参数解析/javascript基础-minimist命令行参数解析-00.webp
  lqip: data:image/webp;base64,UklGRmIAAABXRUJQVlA4IFYAAADQAQCdASoIAAUAAUAmJQBOiP/wNtQ+9AD+T6t3/7hPtbpQs3TYdP68+a78P6Vr4Hff4gZAl/oY93h6rmPbYuiOw2KxNXDdfebP+sU7ZUY6cJmA9AAAAA==
  alt: javascript基础-minimist命令行参数解析
---

## 前言

&emsp;&emsp;因为要写一个命令行小工具，不打算弄得太麻烦，只是简单的通过一个命令和几个参数就足够了，为了简单，选择用nodejs来做，只是一个辅助小功能而已。

&emsp;&emsp;因为要接受参数，起初只是通过```process.argv```获取参数，这个比较简单，按照参数个数逐个获取就可以了，但是使用起来发现有些问，因为这些参数是可以省略的，只需要按照需要传入就可以，这样就会造成参数位置不确定，这就需要更多的处理。

&emsp;&emsp;理想的形式是：```命令 -a 值 -b 值 -c 值 --par 值```

&emsp;&emsp;按照这个想法在网上找了一下（这个简单要求应该是有现成的包的），就找到了```minimist```。

## 安装

```shell
npm install minimist
```

## 使用

```javascript
// 完整参数
// var argv = require('minimist')(process.argv);

//需要的参数
var argv = require('minimist')(process.argv.slice(2));
console.dir(argv);
```

```shell
node index2.js aa bb cc -h --help -a=1 -b 2 --test 123 --dev=456 --arr 123 --arr=456
```

```console
{
  _: [
    'aa',
    'bb',
    'cc'
  ],
  h: true,
  help: true,
  a: 1,
  b: 2,
  test: 123,
  dev: 456,
  arr: [ 123, 456 ]
}
```

&emsp;&emsp;通过上面的测试完全满足我的需要，可以看到他只是对```process.argv```接收的参数进行解析。解析结果是一个对象：

+ 没有中划线的参数以数组的形式在```_```这个key中。
+ 中划线的短参数(```-```)或者双中划线的长参数(```--```)都是以参数名为key的。设置两次就以数组形式存在
+ 其实他没有长参数和短参数的区别，只是简单分开
+ 如果参数后面没有值，则表示是```boolean```类型

## 参数类型

&emsp;&emsp;开发的时候还是发现有些小问题，主要是参数的类型，经查看代码发现它有第二个参数，而这个参数恰恰是用来约束参数类型的。

```javascript
const options = {
    string: ["dir", "d", "word", "w", "to", "t"],
    number: ["resize", "r"],
    boolean: ["help", "h"],
}

const argv = require('minimist')(process.argv.slice(2), options);
```

&emsp;&emsp;通过上面的参数类型约束，可以更好的处理参数。

> 注：我需要这个参数约束的原因是因为上面的```dir```参数，这是一个目录，但是我的目录名是包括```001```这样的目录，总被转化为数字，导致找不到对应目录。
{: .prompt-tip }
