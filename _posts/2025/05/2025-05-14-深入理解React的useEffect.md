---
title: 深入理解React的useEffect

author: mmy83
date: 2025-05-14 20:22:00 +0800
categories: [框架, React]
tags: [react, 框架, 前端, 钩子, 副作用, useEffect, hook]
math: true
mermaid: true
image:
  path: /images/2025/05/2025-05-14/深入理解React的useEffect/深入理解React的useEffect-00.png
  lqip: data:image/webp;base64,UklGRlIAAABXRUJQVlA4IEYAAABQAgCdASoIAAUAAUAmJbACdLoB+AADJE89uYAA/vObiP9itvk2wdWZI7/Kcf/ONpRXdnH3T//uqJcQn7fCEfdF7GfwmAAA
  alt: 深入理解React的useEffect
---

## 介绍

在 React 中，useEffect 是一个非常重要的 Hook，用于在函数组件中处理副作用。它强大而灵活，是函数组件中替代类组件生命周期方法的核心工具。通过 useEffect，你可以轻松实现以下操作：

+ 数据获取（例如调用 API）

+ DOM 操作（如操作文档标题或动画效果）

+ 事件监听（例如窗口大小调整）

+ 清理任务（例如清理定时器或取消订阅）

+ 写日志

## 语法

useEffect 是 hook 函数

```javascript
  // 实例
  useEffect(() => {
    async function getList() {
      const res = await axios.get("http://localhost:3000/dataList");
      setList(res.data);
    }
    getList();
  }, []);
```

第一个参数（必要）： 自定义的处理函数（官方称呼为：副作用函数），可在副作用函数中 return 一个函数来清除副作用，从第二次执行副作用函数开始，每次都会先执行return中的清除副作用的函数，再执行副作用函数，当组件卸载时，所有清除副作用的函数会按顺序依次执行。

第二个参数（可选）： 依赖项 （不能为对象/数组等引用类型的数据，因为引用类型的数据每次页面渲染时都会生成新地址，若useEffect的处理函数会引发页面重新渲染，则会导致死循环）

## 控制执行时机

useEffect 的第二个参数是一个依赖数组，用于控制它的执行时机。根据是否传递依赖数组以及传递哪些依赖，可以实现不同的行为。

### 不依赖

组件每次渲染后（包括状态或属性变化时）都会执行该 useEffect。

```javascript
useEffect(() => {
  console.log('每次组件渲染后都会执行');
});
```

### 空依赖数组

只在组件挂载时执行一次，类似于类组件中的 componentDidMount。

```javascript
useEffect(() => {
  console.log('仅在组件挂载时执行一次');
}, []);

// 获取数据
useEffect(() => {
  async function fetchData() {
    const response = await fetch('<https://api.example.com/data>');
    const data = await response.json();
    console.log(data);
  }
  fetchData();
}, []); // 空数组表示仅在挂载时执行
 
```

### 确定依赖

只有当 count 的值发生变化时，useEffect 才会执行

```javascript
// 单个依赖
useEffect(() => {
  console.log(`计数值更新为: ${count}`);
}, [count]);

// 多个依赖
useEffect(() => {
  console.log(`count 或 otherState 发生了变化`);
}, [count, otherState]);
```

## 清理副作用

当组件卸载时，或在依赖变化时，我们可能需要清理一些副作用（如事件监听、定时器等）。可以通过 useEffect 返回一个清理函数来完成。

### 清理事件监听

```javascript
useEffect(() => {
  const handleResize = () => console.log('窗口大小变化');
  window.addEventListener('resize', handleResize);
 
  // 返回清理函数
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []); // 空数组表示只在挂载和卸载时运行
 
```

### 清理定时器

```javascript
useEffect(() => {
  const timer = setInterval(() => {
    console.log('计时器运行中');
  }, 1000);
 
  // 返回清理函数
  return () => {
    clearInterval(timer);
  };
}, []); // 确保在组件卸载时清理定时器
```

## 注意事项

### 避免依赖遗漏

在依赖数组中，React 要求包含所有在 useEffect 内部使用的变量，否则可能引发错误或意外行为：

```javascript
// 正确
useEffect(() => {
  console.log(value);
}, [value]); // 监听 value

// 错误
useEffect(() => {
  console.log(value); // 未声明依赖 value，可能导致问题
}, []); // 空数组，依赖不会触发
 
```

### 防止无限循环

在 useEffect 中更新状态时，需注意避免无限循环渲染：

```javascript
useEffect(() => {
  setCount(count + 1); // 这会更新 `count`，触发 `useEffect` 再次执行
}, [count]); // 由于 count 变化，`useEffect` 被多次调用，导致死循环
```

#### 解决一：使用条件判断

最简单的方式是通过在 useEffect 内部增加条件，来限制更新状态的行为。例如，你可以限制 count 的最大值或者根据其他条件来决定是否更新：

```javascript
useEffect(() => {
  if (count < 10) {
    setCount(count + 1); // 限制最大值，避免死循环
  }
}, [count]); // 只有在 count 小于 10 时才更新
```

在这个例子中，count 达到 10 时就不会再触发更新，从而避免了死循环。

#### 解决二：使用函数式更新

如果你需要依赖之前的 count 来计算新的状态，推荐使用 setCount 的函数式更新方法。这不仅能避免闭包问题，还能确保状态的更新是基于最新的 count 值，而不是依赖于 useEffect 传入的旧值。

```javascript
useEffect(() => {
  setCount((prevCount) => prevCount + 1); // 使用函数式更新，基于最新的 prevCount
}, [count]); // 注意：这里的 useEffect 可能仍然会导致死循环，建议重新考虑依赖条件
```

但是，这种方式仍然会导致死循环，因为 count 是依赖项。为了避免死循环，你可以通过优化依赖条件来解决。

#### 解决三：使用 useRef 存储先前的状态

如果你不希望 count 作为 useEffect 的直接依赖，但又需要通过 count 来计算新的值，可以使用 useRef 来存储上一次的 count，从而避免直接依赖 count 更新。

```javascript
import React, { useState, useEffect, useRef } from 'react';
 
function Example() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
 
  useEffect(() => {
    prevCountRef.current = count; // 存储上一次的 count 值
  }, [count]); // 每次 count 变化时更新 prevCountRef
 
  useEffect(() => {
    if (prevCountRef.current < 10) {
      setCount(count + 1); // 只有当 prevCount 小于 10 时才更新
    }
  }, []); // 空数组，避免依赖 count，减少触发次数
}
```
这种方法可以确保在每次更新时，useEffect 不会直接依赖 count，而是依赖一个 useRef 存储的值，从而避免死循环。

#### 解决四：使用 setTimeout 或 requestAnimationFrame 延迟更新

如果你希望延迟状态更新，避免立即触发副作用的反复执行，可以使用 setTimeout 或 requestAnimationFrame 进行延时操作：

```javascript
useEffect(() => {
  const timer = setTimeout(() => {
    setCount(count + 1); // 延迟更新
  }, 1000); // 延时 1 秒更新
 
  return () => clearTimeout(timer); // 清理定时器
}, [count]); // 每次 count 变化时触发
```

这种方式可以控制更新的节奏，避免过快地反复更新。

### 清理

每次 useEffect 执行时，React 会先执行上一次 useEffect 返回的清理函数。这种机制能有效避免内存泄漏问题。

## 生命周期方法与 useEffect 的对照表

useEffect 能够模拟类组件中的生命周期方法，通过合理设计依赖数组可以实现对应的逻辑。

|类组件生命周期 | 函数组件实现（useEffect）|触发时机|
|---|---|---|
|componentDidMount|useEffect(() => { ... }, [])|组件挂载后执行一次|
|componentDidUpdate|useEffect(() => { ... }, [依赖])|依赖项变化时执行（挂载时默认执行一次）|
|componentWillUnmount|useEffect(() => { return () => { ... }; }, [])|组件卸载时执行清理函数|

react官方的原话说：如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

+ componentDidMount 组件挂载

+ componentDidUpdate 组件更新

+ componentWillUnmount 组件将要摧毁

|依赖项|副作用函数执行时机|
|---|---|
|没有依赖项|组件初始渲染+组件更新时执行|
|空数组依赖|只在初始渲染时执行一次|
|添加特定依赖项|组件初始渲染+特性依赖项变化时执行|

## 参考资料

1. [深入理解 React 的 useEffect：全面指南](https://blog.csdn.net/u012446963/article/details/143835060)

2. [轻松学会 React 钩子：以 useEffect() 为例](https://cloud.tencent.com/developer/article/1699768)

3. [react18【系列实用教程】useEffect —— 副作用操作](https://blog.csdn.net/weixin_41192489/article/details/138706946)

4. [React第十五章(useEffect)](https://juejin.cn/post/7436589005838319635)

5. [【useEffect Hook】在组件中执行副作用操作](https://blog.csdn.net/qq_46032105/article/details/145270712)

6. [React中的useEffect(副作用)介绍](https://www.jb51.net/javascript/310819y3y.htm)
