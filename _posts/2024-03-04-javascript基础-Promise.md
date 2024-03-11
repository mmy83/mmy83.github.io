---
title: javascript基础-Promise
author: mmy83
date: 2024-03-04 13:21:00 +0800
categories: [语言, javascript]
tags: [Promise, 异步, javascript]
math: true
mermaid: true
image:
  path: /images/2024-03-04/javascript基础-Promise/javascript基础-Promise-00.jpg
  lqip: data:image/jpeg;base64,/9j/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAAEAAgDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAf/xAAZEAEAAwEBAAAAAAAAAAAAAAABAAIRBjH/xAAVAQEBAAAAAAAAAAAAAAAAAAADBP/EABkRAAIDAQAAAAAAAAAAAAAAAAABAgMRIf/aAAwDAQACEQMRAD8AmFneNuAGpuHsREmoSxiWTlq6f//Z
  alt: javascript基础-Promise
---

## Promise是什么

&emsp;&emsp;```Promise```是javascript中进行异步编程的新解决方案，可以很大程度上解决回调地狱问题。这句话我个人觉得不太好理解。有点我无力反驳但是有感觉啥都没说的感觉。我想用我个人的理解说一下。

&emsp;&emsp;```Promise```中文翻译为```承诺```，我觉得这个更好理解。我们做异步调用的时候，他就像一个承诺：“我承诺，我立刻去做一个事，不管成功还是失败，我都将给你一个明确的结果，你随时可以来我这里获取结果，如果我正在做，我做完会把结果给你；如果我已经做完了，我将保存这个结果，等你来获取，你获取的时候我会立刻给你我保存的结果。”。我们从这个承诺可以得到几个信息：

1、我是一个承诺，在javascript中我就是一个对象，并且我应该有一个创建这个对象的构造函数。

2、当我说出这个承诺的时候我就会立刻，而不是等待你让我去做。

3、不管成功失败我都会给你明确结果。

4、当你获取结果的时候，我没有做完，我做完后保存并立刻给你结果；当你一直没获取结果，我做完后会把结果保存起来，等你获取；当你获取结果的时候我已经做完并保存了，我会把我做完的结果立刻给你。

## Promise对象

&emsp;&emsp;既然```Promise```是一个对象，那这个对象应该有自己的构造函数、属性和方法.

### Promise构造函数

&emsp;&emsp;接收一个回调函数作为参数，回调函数的参数也是两个函数：

1、resolve（解决）：函数类型的数据,对象返回成功时使用

2、reject（拒绝）：函数类型的数据,对象返回失败时使用

```javascript
// 创建一个简单的Promise
const test = new Promise(function(resolve,reject){
    if(1==1){
      resolve("abc")
    }else{
      reject("bcd")
    }
  })
console.log(test)

```

结果：

![Promise属性](/images/2024-03-04/javascript基础-Promise/javascript基础-Promise-01.jpg)

### 属性

&emsp;&emsp;从上图可以看到里面包含三个属性：Prototype、PromiseState、PromiseResult，了解javascript的都知道Prototype是原型，我们这里不多说。主要是另外两个：

#### PromiseState

&emsp;&emsp;刚才说了，承诺分为正在做和完成，完成又分为成功和是吧，但是不管成功还是失败，都是一个明确的结果，可以获取到。这三个状态```pending```（进行中）、```fulfilled```（已成功）和```rejected```（已失败）就是在这里存放的。

&emsp;&emsp;这个状态有一个特点，也是很好理解的特点：即，只能是从pending（进行中）到fulfilled（已成功）或从pending（进行中）到rejected（已失败），不能fulfilled（已成功）到rejected（已失败）或者从rejected（已失败）到fulfilled（已成功）。而且只能改变一次。这个应该很好理解也很合理。

#### PromiseResult

&emsp;&emsp;上面也说了，承诺的事做完后，不管成功还是失败都会有一个明确的结果.这个属性就是存放这个结果的。状态为fulfilled（已成功）存放的是成功的结果；状态为rejected（已失败）存放的是失败的结果；状态为pending（进行中）这个熟悉为空```undefined```

### 实例方法

![Promise方法](/images/2024-03-04/javascript基础-Promise/javascript基础-Promise-02.jpg)

&emsp;&emsp;从Prototype中可以看到他有四个方法：Promise、catch、finally、then。Promise：构造函数，不多说。catch、finally：都是用于异常处理的，不多说。then：这个就是上面说的，来获取承诺执行结果的，也是这里最关键的。

```javascript

const test = new Promise(function(resolve,reject){
    if(1==1){
      resolve("abc")
    }else{
      reject("bcd")
    }
  })
test.then((res)=>{console.log(res)},(err)=>{ console.log(err)})

```

then(onResolved, onRejected)：

1、onResolved 函数: 成功的回调函数 (value) => {}

2、onRejected 函数: 失败的回调函数 (reason) => {}

&emsp;&emsp;then接收两个参数，两个参数都是函数类型，第一个函数是对象成功时的回调，第二是失败时的回调。其实这个就是我们不用Promise的时候的回调函数。这里还要一个关键：即，then是有返回值的而且这个返回值也是一个```Promise```，这个很关键，也就是这个返回值实现了链式调用。

&emsp;&emsp;这里还是要介绍一下```catch```，这个方法不光会处理异常，还会在```reject```的时候被执行。```finally```不管是```resolve```还是```reject```都会被执行。另外```catch```、```finally```、```then```都是有返回值的，而且都是```Promise```类型，这也意味着都可以使用链式进行多次调用。

### 类方法

&emsp;&emsp;除了上面的then、catch、finally方法都属于Promise的实例方法，都是存放在Promise的prototype上的。Promise还有一些来方法：resolve、reject、all、allSettled、race、any

#### resolve

&emsp;&emsp;有时候已经有一个内容，只是想包装成Promise，这个时候可以使用```Promise.resolve```方法来完成。

```javascript

// Promise.resolve的用法相当于new Promise，并且执行resolve操作：
Promise.resolve("abc")
// 等价于
const promise = new Promise((resolve) => resolve("abc"))

```

#### reject

&emsp;&emsp;reject方法类似于resolve方法，只是会将Promise对象的状态设置为reject状态。这个时候可以使用```Promise.reject```方法来完成。

```javascript

Promise.reject("error")
// 等价于
const promise = new Promise((resolve, reject) => {
  reject("error")
})

```

#### all
&emsp;&emsp;```Promise.all```作用是将多个Promise包裹在一起形成一个新的Promise,新的Promise状态由包裹的所有Promise共同决定：

1、当所有的Promise都为成功 (fulfilled) 状态时，新的Promise状态为fulfilled，并且会将所有Promise的返回值组成一个数组；

```javascript

// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("aaa")
  }, 1000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("bbb")
  }, 2000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ccc")
  }, 3000)
})

// 会等待五秒, 当所有Promise都有结果才会打印
Promise.all([a, b, c]).then(res => {
  console.log("Promise res:", res) // Promise res: ['aaa', 'bbb', 'ccc']
})
```

2、当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数；

```javascript

// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("aaa")
  }, 5000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("bbb")
  }, 2000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ccc")
  }, 3000)
})

// a、b都有返回reject, 会将第一个reject的返回值, 也就是b的返回值作为参数
Promise.all([a, b, c]).then(res => {
  console.log("Promise res:", res)
}).catch(err => {
  console.log("Promise err:", err) // Promise err: bbb
})
```

> 其实就一句话：全都成功才成功。
{: .prompt-tip }

#### allSettled

&emsp;&emsp;all方法很暴力，当有其中一个Promise变成reject状态时，新Promise就会立即变成对应的reject状态。有时候我们可能更想知道每一个的状态.
&emsp;&emsp;```Promise.allSettled```方法会在所有的Promise都有结果（settled），无论是fulfilled，还是rejected时，才会有最终的状态,```并且这个Promise的结果一定是fulfilled的```。

> 其实就一句话：给出每一个明细。
{: .prompt-tip }

```javascript
// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("aaa")
  }, 1000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("bbb")
  }, 2000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ccc")
  }, 3000)
})

// allSettled方法
Promise.allSettled([a, b, c]).then(res => {
  console.log("allSettled:", res)   
})
```

我们来看一下打印结果:

```shell
allSettled: [
  {status: 'rejected', reason: 'aaa'}, 
  {status: 'rejected', reason: 'bbb'}, 
  {status: 'fulfilled', value: 'ccc'}
]
```

#### race

&emsp;&emsp;一个Promise的结果就决定最终新Promise的状态

```javascript
// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("aaa")
  }, 5000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("bbb")
  }, 1000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ccc")
  }, 3000)
})

// 由于b最先返回一个结果 并且是已拒绝状态 那么会执行catch的回调
Promise.race([a, b, c]).then(res => {
  console.log("res:", res)
}).catch(err => {
  console.log("err:", err) // err: bbb
})

```

#### any

&emsp;&emsp;any方法会等到一个fulfilled状态，才会决定新Promise的状态

```javascript
// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("aaa")
  }, 5000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("bbb")
  }, 2000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("ccc")
  }, 3000)
})

// 最先返回的b 状态是已拒绝, 那么便会等下一个是c 状态是以决议 就会返回c
Promise.any([a, b, c]).then(res => {
  console.log("any promise res:", res) // any promise res: ccc
})
```

&emsp;&emsp;如果所有的Promise都是rejected的，那么也会等到所有的Promise都变成rejected状态, 再执行catch中的回调；

```javascript
// 创建三个Promise
const a = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("aaa")
  }, 1000)
})

const b = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("bbb")
  }, 2000)
})

const c = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject("ccc")
  }, 3000)
})

// 当所有Promise的都是rejected时
Promise.any([a, b, c]).then(res => {
  console.log("any promise res:", res)
}).catch(err => {
  console.log("any promise err:", err) // any promise err: AggregateError: All promises were rejected
})
```

&emsp;&emsp;如果所有的Promise都是reject的，那么会报一个AggregateError的错误。
