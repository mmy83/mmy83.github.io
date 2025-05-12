---
title: javascript基础-Fetch处理网络请求

author: mmy83
date: 2025-05-12 20:22:00 +0800
categories: [编程, javascript]
tags: [ajax, 前端, fetch, 网络请求, javascript]
math: true
mermaid: true
image:
  path: /images/2025/05/2025-05-12/javascript基础-Fetch处理网络请求/javascript基础-Fetch处理网络请求-00.png
  lqip: data:image/webp;base64,UklGRjwAAABXRUJQVlA4IDAAAACwAQCdASoIAAQAAUAmJZwAAu18sncAAP77DP/Ey6nYIXqWAtaLcjbtboF4c2yqAAA=
  alt: javascript基础-Fetch处理网络请求
---

## 介绍

在 Web 开发中，向服务器发送请求的方式有三种：XMLHttpRequest、axios、fetch。之前介绍了 axios ，这次介绍一下 fetch。

+ XMLHttpRequest

使用 XMLHttpRequest 对象通过 Ajax 向服务器请求数据，虽然可以实现功能，但代码显得冗长繁琐。

```javascript
// 1. 创建一个 XMLHttpRequest 对象
let xhr = new XMLHttpRequest();
// 2. 设置请求方式和地址
xhr.open('get', 'https://api.example.com/data');
// 3. 发送请求
xhr.send();
// 4. 监听 load 事件，获取响应
xhr.addEventListener('load', function () {
    console.log(JSON.parse(xhr.response));
});

```

+ axios

axios 底层基于 XMLHttpRequest，但是做了 Promise 的封装，因此代码更简洁

```javascript
axios.get('https://api.example.com/data')
     .then(response => console.log(response.data))
     .catch(error => console.error(error));
```

+ fetch

fetch 是现代浏览器原生支持的 API，被称为“下一代 Ajax 技术”。它以 Promise 形式返回结果，语法简洁，易于使用。并且，通过 Stream（流）来分块读取数据，对大文件请求更为友好。fetch 的兼容性良好，现代浏览器大多支持（IE 除外）。

```javascript
fetch('https://api.example.com/data')
    .then(response => {
        return response.json(); // 将响应解析为 JSON
    })
    .then(data => console.log(data))
    .catch(error => console.error(error));

```

## 语法

```javascript
fetch(url,options).then((response)=>{
//处理http响应
},(error)=>{
//处理错误
})
```

### 参数

+ url ：是发送网络请求的地址。

+ options：发送请求参数:

  + body - http请求参数

  + mode - 指定请求模式。默认值为cros:允许跨域;same-origin:只允许同源请求;no-cros:只限于get、post和head,并且    只能- 使用有限的几个简单标头。

    + cors：默认值，允许跨域请求。

    + same-origin：只允许同源请求。

    + no-cors：请求方法只限于 GET、POST 和 HEAD，并且只能使用有限的几个简单标头，不能添加跨域的复杂标头，相当于提交表单所能发出的请求。

  + cache - 用户指定缓存。

    + default：默认值，先在缓存里面寻找匹配的请求。

    + no-store：直接请求远程服务器，并且不更新缓存。

    + reload：直接请求远程服务器，并且更新缓存。

    + no-cache：将服务器资源跟本地缓存进行比较，有新的版本才使用服务器资源，否则使用缓存。

    + force-cache：缓存优先，只有不存在缓存的情况下，才请求远程服务器。

    + only-if-cached：只检查缓存，如果缓存里面不存在，将返回504错误。

  + method - 请求方法，默认GET

  + signal - 用于取消 fetch

  + headers - http请求头设置

  + keepalive - 用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据。

  + credentials - cookie设置，默认omit，忽略不带cookie，same-origin同源请求带cookie，inclue无论跨域还是同源都会带cookie。

    + same-origin：默认值，同源请求时发送 Cookie，跨域请求时不发送。

    + include：不管同源请求，还是跨域请求，一律发送 Cookie。

    + omit：一律不发送。

  + redirect：属性指定 HTTP 跳转的处理方法。

    + follow：默认值，fetch()跟随 HTTP 跳转。

    + error：如果发生跳转，fetch()就报错。

    + manual：fetch()不跟随 HTTP 跳转，但是response.url属性会指向新的 URL，response.redirected属性会变为true，由开发者自己决定后续如何处理跳转。

  + integrity：属性指定一个哈希值，用于检查 HTTP 回应传回的数据是否等于这个预先设定的哈希值。

  + referrer：属性用于设定 fetch() 请求的 referer 标头。

  + referrerPolicy：属性用于设定Referer标头的规则。

    + no-referrer-when-downgrade：默认值，总是发送Referer标头，除非从 HTTPS 页面请求 HTTP 资源时不发送。

    + no-referrer：不发送Referer标头。

    + origin：Referer标头只包含域名，不包含完整的路径。

    + origin-when-cross-origin：同源请求Referer标头包含完整的路径，跨域请求只包含域名。

    + same-origin：跨域请求不发送Referer，同源请求发送。

    + strict-origin：Referer标头只包含域名，HTTPS 页面请求 HTTP 资源时不发送Referer标头。

    + strict-origin-when-cross-origin：同源请求时Referer标头包含完整路径，跨域请求时只包含域名，HTTPS 页面请求 HTTP 资源时不发送该标头。

    + unsafe-url：不管什么情况，总是发送Referer标头。

### 响应（response）

+ status - http状态码，范围在100-599之间

+ statusText - 服务器返回状态文字描述

+ ok - 返回布尔值，如果状态码2开头的，则true,反之false

+ url - 返回请求的 URL。如果 URL 存在跳转，该属性返回的是最终 URL。

+ headers - 响应头，指向一个 Headers 对象，对应 HTTP 回应的所有标头。Headers 对象可以使用for...of循环进行遍历。

  + Headers.get()：根据指定的键名，返回键值。

  + Headers.has()： 返回一个布尔值，表示是否包含某个标头。

  + Headers.set()：将指定的键名设置为新的键值，如果该键名不存在则会添加。

  + Headers.append()：添加标头。

  + Headers.delete()：删除标头。

  + Headers.keys()：返回一个遍历器，可以依次遍历所有键名。

  + Headers.values()：返回一个遍历器，可以依次遍历所有键值。

  + Headers.entries()：返回一个遍历器，可以依次遍历所有键值对（[key, value]）。

  + Headers.forEach()：依次遍历标头，每个标头都会执行一次参数函数。

+ body - 响应体。响应体内的数据，根据类型各自处理。

+ type - 返回请求类型。
  
  + basic：普通请求，即同源请求。

  + cors：跨域请求。

  + error：网络错误，主要用于 Service Worker。

  + opaque：如果fetch()请求的type属性设为no-cors，就会返回这个值，详见请求部分。表示发出的是简单的跨域请求，类似 form 表单的那种跨域请求。

  + opaqueredirect：如果fetch()请求的redirect属性设为manual，就会返回这个值，详见请求部分。

+ redirected - 返回布尔值，表示是否发生过跳转。

### 读取

+ response.text() -- 得到文本字符串

+ response.json() - 得到 json 对象

+ response.blob() - 得到二进制 blob 对象

+ response.formData() - 得到 fromData 表单对象

+ response.arrayBuffer() - 得到二进制 arrayBuffer 对象

上述 5 个方法，返回的都是 promise 对象，必须等到异步操作结束，才能得到服务器返回的完整数据。

+ response.clone()

stream 对象只能读取一次，读取完就没了，这意味着，上边的五种读取方法，只能使用一个，否则会报错。因此 response 对象提供了 clone() 方法，创建 respons 对象副本，实现多次读取。

+ response.body()

body 属性返回一个 ReadableStream 对象，供用户操作，可以用来分块读取内容，显示下载的进度就是其中一种应用。

response.body.getReader() 返回一个遍历器，这个遍历器 read() 方法每次都会返回一个对象，表示本次读取的内容块。

### 取消请求

```javascript
let controller = new AbortController();
setTimeout(() => controller.abort(), 1000);

try {
  let response = await fetch('/long-operation', {
    signal: controller.signal
  });
} catch(err) {
  if (err.name == 'AbortError') {
    console.log('Aborted!');
  } else {
    throw err;
  }
}
```

## 注意

### 兼容性

![兼容](/images/2025/05/2025-05-12/javascript基础-Fetch处理网络请求/javascript基础-Fetch处理网络请求-01.png)

fetch 是相对较新的技术，IE浏览器不支持，还有其他低版本浏览器也不支持，因此如果使用fetch时，需要考虑浏览器兼容问题。

#### 解决办法

+ 引入 polyfill 完美支持 IE8 以上。

+ 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: es5-shim, es5-sham

+ 引入 Promise 的 polyfill：es6-promise

+ 引入 fetch 探测库：fetch-detector

+ 引入 fetch 的 polyfill： fetch-ie8

+ 可选：如果你还使用了 jsonp，引入 fetch-jsonp

+ 可选：开启 Babel 的 runtime 模式，现在就使用 async/await

polyfill 的原理就是探测fetch是否支持，如果不支持则用 xhr 实现。支持 fetch 的浏览器，响应中文会乱码，所以使用 fetch-detector 和 fetch-ie8 解决乱码问题。

### 不带cookie

传递cookie时，必须在header参数内加上 credentials:'include'，才会像 xhr 将当前cookie 带有请求中。

### 异常处理

fetch 不同于 xhr ，xhr 自带取消、错误等方法，所以服务器返回 4xx 或 5xx 时，是不会抛出错误的，需要手动处理，通过 response.status 字段来判断。另一种方法是判断 response.ok 是否为true

## 参考资料

1. [前后端数据交互(四)——fetch 请求详解](https://cloud.tencent.com/developer/article/1884235)

2. [Fetch API 教程](https://cloud.tencent.com/developer/article/1767262)

3. [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

4. [【前端】Fetch：数据请求](https://blog.csdn.net/2303_80346267/article/details/143507704)

5. [Fetch](https://javascript.info/fetch)

6. [node-fetch](https://github.com/node-fetch/node-fetch)
