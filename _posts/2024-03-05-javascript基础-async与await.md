---
title: javascript基础-async与await
author: mmy83
date: 2024-03-05 14:27:00 +0800
categories: [编程, javascript]
tags: [async, await, 异步, javascript]
math: true
mermaid: true
image:
  path: /images/2024-03-05/javascript基础-async与await/javascript基础-async与await-00.webp
  lqip: data:image/webp;base64,UklGRlgAAABXRUJQVlA4IEwAAACwAQCdASoIAAUAAUAmJZgCdADzes2gAP6t+soCze+jhfMw/h1Vdz/MmzokpHdi/Fou7dcpQbmVY8Wz3Vs0mKc8twSXoYWYbpYhIAAA
  alt: javascript基础-async与await
---

## async与await

&emsp;&emsp;```async```、```await```是处理javascript中的异步问题的。

+ async：声明函数时使用，用于表明该函数是一个异步函数。该函数返回值是一个```Promise```。既然是Promise，就可以用```then```来回去值。
+ await：只能在```async```标记的函数内使用。在函数调用时使用，表示阻塞直到这个函数返回。函数的返回结果如果是一个Promise，则相当于用then获取返回值而不是Promise这个对象，如果是普通函数，则直接返回。

```javascript
    function a() {
      console.log('a start');
        return new Promise((resolve, reject) => {
            console.log('a p start');
            setTimeout(() => {
                resolve('resolve a');
            }, 5000)
        })
    }

    function b() {
        console.log('b start');
        return new Promise((resolve, reject) => {
            console.log('b p start');
            setTimeout(() => {
                return resolve('resolve b');
            }, 6000)
        })
    }

    async function t() {
        console.log('t start');
        let a1 = await a();
        console.log('a end:', a1);
        let b1 = await b();
        console.log('b end:', b1);
        console.log('t end');

        return 'ok';
    }

    p = t()
    
    p.then(res => {
        console.log("res:",res)
        // return Promise.resolve(res)
        // return Promise.reject(res)
    });

```

&emsp;&emsp;结果如下：

```console
t start
a start
a p start
Promise {<pending>}
a end: resolve a
b start
b p start
b end: resolve b
t end
res: ok
```

&emsp;&emsp;通过观察结果发现，除了```Promise {<pending>}```之外，其他的结果都是顺序执行的。这就是```async```、```await```的作用。那么这个```Promise {<pending>}```是啥呢，其实是```p.then```的返回结果，可以把里面的注释打开观察结果

```console

// 打开 return Promise.resolve(res)
[[Prototype]]: Promise
[[PromiseState]]: "fulfilled"
[[PromiseResult]]: "ok"

// 打开return Promise.reject(res)
[[Prototype]]: Promise
[[PromiseState]]: "rejected"
[[PromiseResult]]: "ok"

```

&emsp;&emsp;到这里基本上也就能明白```async```、```await```了，但是网上很多资料讲的更多，这里也多说一下

## Generator函数

&emsp;&emsp;```Generator```函数返回一个迭代器，通过内部的```yield```和外部的```next```来控制停止和运行。```Generator```有如下特点：

1、```Generator```函数再定义的时候function关键字和函数名之间有一个*；

2、```Generator```函数的内部通过```yield```来暂停

3、```Generator```函数通过外部调用```next```来继续执行，直到遇到下一个```yield```暂停，或者```return```结束。

```javascript
function* fun() {
        let a = yield 'aaa'
        let b = yield 'bbb'
        console.log(a + ' ' + b)
    }

let d = fun()
console.log(d.next())
console.log(d.next())
console.log(d.next())
```

结果：

```console
{value: 'aaa', done: false}
{value: 'bbb', done: false}
undefined undefined
{value: undefined, done: true}
```

> 注:可以看到这里的```next```方法是有返回值的，返回值是```yield```后面的表达式的结果和运行状态组成的对象```{value: yield后面的表达式的结果, done: 运行状态}```。
{: .prompt-tip }

&emsp;&emsp;这里解释一下：

1、这里定义了一个```Generator```函数，实例化```let d = fun()```后并没有直接执行完成，而是执行到```let a = yield 'aaa'```时因为有```yield```暂停了，而且这个暂停是在给```a```赋值前。

2、外面通过```console.log(d.next())```第一次调用```next```，输出结果```{value: 'aaa', done: false}```，程序继续执行，给```a```赋值，可是这时候没有值。因为```yield```的值只能通过外面的```next```传入。所有这时候```a```是```undefined```,然后运行```let b = yield 'bbb'```

3、同理遇到```yield```暂停了，而且这个暂停是在给```b```赋值前.

4、外面通过```console.log(d.next())```，输出结果```{value: 'bbb', done: false}```，同理```b```是```undefined```

5、这时候没有```yield```，继续运行```console.log(a + ' ' + b)```，因为```a```、```b```都为```undefined```所有输出```undefined undefined```。

6、然后外面又调用```console.log(d.next())```，这时候程序已经执行完成了，所有结果是```{value: undefined, done: true}```

&emsp;&emsp;到这里，可能比较迷茫，这个和```async```、```await```有啥关系吗？还真有。想一想，如果我们```yield```后面跟的是一个```Promise```，在```next```的时候，使用```then```处理来```Promise```中的结果，在```then```的回调函数中再进行下一个```next```会咋样，是不是就是等第一个```Promise```完成后，继续执行下一个```Promise```，是不是就可以达到await效果。

## Thunk函数

&emsp;&emsp;这里还要有一个能调用```next```的函数。在JavaScript中Thunk函数是指将一个多参数函数替换成一个只接受回调函数作为参数的单参数函数。看着好像又没啥用。这个函数的作用就是包装一层：定义一个函数，这个函数的返回值是一个可以接受回调函数的函数。看下面的例子：

```javascript
function p1() {
    return new Promise((resolve, reject) => {
        console.log('p1 start');
        setTimeout(() => {
            resolve('p1 callback');
        }, 3000)
    })
}

function p2() {
    return new Promise((resolve, reject) => {
        console.log('p2 start');
        setTimeout(() => {
            resolve('p2 callback');
        }, 3000)
    })
}

function* fun() {
    console.log('fun start');
    let v1 = yield p1();
    console.log('v1:',v1);
    let v2 = yield p2();
    console.log('v2:',v2);
    console.log(v1 + ' ' + v2);
    console.log('fun end');
}

function tt(fn) {
    return new Promise(function(resolve, reject) {
        const f = fn();
        function next(data) {
            let result = f.next(data);
            if (result.done) {
                return resolve(result.value);
            }
            Promise.resolve(result.value).then(res => {
                next(res); // Promise执行完成调用 next
            })
        }
        next(undefined) // 第一次调用 next
    })
}

tt(fun)

```

运行结果：

```console
fun start
p1 start
Promise {<pending>}
v1: p1 callback
p2 start
v2: p2 callback
p1 callback p2 callback
fun end
```
