---
title: Python上下文管理语句：with
author: mmy83
date: 2025-01-20 17:11:00 +0800
categories: [编程, python]
tags: [编程, python, 上下文, with]
math: true
mermaid: true
image:
  path: /images/2025/01/2025-01-20/Python上下文管理语句：with/Python上下文管理语句：with-00.jpeg
  lqip: data:image/webp;base64,UklGRkwAAABXRUJQVlA4IEAAAACwAQCdASoIAAUAAUAmJZwC7AED3+LgAP79d+uH7+vc2f4DWM/5X5tvwz9w1/yNZH3tbyBDjzgADRMUbGd+NAAA
  alt: Python上下文管理语句：with
---

## 介绍

with是Python的一个语法糖，用于上下文管理，主要用于在资源使用时出现异常，可以自动释放资源，减少手动关闭资源，简化代码。主要具有两个作用：

+ 简化资源管理，例如文件操作、网络连接等。

+ 异常处理，在with语句块中发生异常时，可以自动执行清理操作。

## 场景

+ **文件操作**：使用上下文管理器可以自动管理文件的打开和关闭，即使在读取或写入文件时发生异常也能确保文件被正确关闭。

+ **数据库连接**：数据库连接通常需要明确地关闭以释放资源，上下文管理器可以保证即使在查询过程中发生错误也能关闭连接。

+ **网络连接**：网络请求可能需要打开和关闭连接，使用上下文管理器可以自动处理这些操作。

+ **线程和锁**：在多线程编程中，使用上下文管理器可以自动获取和释放锁，避免死锁的发生。

+ **模拟资源环境**：在测试或某些特定操作中，可能需要模拟某些资源环境，上下文管理器可以在进入和退出时设置和清理环境。

+ **资源池管理**：对于从资源池中获取和释放资源的操作，上下文管理器可以确保资源被正确归还。

+ **异常处理**：在需要进行复杂异常处理的场景中，上下文管理器可以在退出时统一处理异常。

+ **配置上下文**：在需要临时改变配置并在操作完成后恢复原有配置的场景中，上下文管理器可以很方便地管理配置的变更。

## 上下文管理器协议

Python的上下文管理协议是一组特殊方法的集合，它们允许对象与with语句配合使用，以确保在代码块执行前后正确地管理资源。这个协议分为同步上下文管理协议和异步同步上下文管理协议。

### 同步协议

+ **{% raw %}__enter__(){% endraw %}** 方法：当进入with语句块时，该方法被调用。它应该返回一个对象，通常是管理器对象本身，用as命名。该对象在with块中使用。这个方法允许你执行一些设置工作，比如打开文件、获取锁或初始化资源。

+ **{% raw %}__exit__(exc_type, exc_value, traceback){% endraw %}** 方法：当退出with语句块时，无论是否发生异常，该方法都会被调用。```__exit__```方法允许你执行清理工作，比如关闭文件、释放锁或释放资源。如果```__exit__```方法返回False或没有返回值（这意味着返回了None），异常（如果发生了的话）将被重新抛出；如果返回True，则表明异常已经被处理，并且不会重新抛出。它接收三个参数：

  + ***exc_type***：如果with块中发生异常，此参数为异常类型；否则为None。

  + ***exc_value***：如果发生异常，此参数为异常实例；否则为None。

  + ***traceback***：如果发生异常，此参数为 traceback 对象；否则为None。

### 异步协议(Python 3.7+ 引入)

+ **{% raw %}__aenter__(){% endraw %}** 方法：
异步上下文管理器的进入方法，类似于```__enter__()```，但它是一个异步方法，可以使用await。

+ **{% raw %}__aexit__(exc_type, exc_value, traceback){% endraw %}** 方法：
异步上下文管理器的退出方法，也是一个异步方法。它接收与同步版本相同的参数，并在退出async with语句块时被调用。

## 使用

```python
with open('example.txt', 'r') as file:
    content = file.read()
```

open函数返回一个文件对象，它实现了上下文管理器协议。使用with语句可以确保文件在使用后自动关闭。

## 等同代码

```python
# 异常处理方式
db.begin()
try:
    # do some actions
except:
    db.rollback()
    raise
finally:
    db.commit()

# 使用上下文管理器
with transaction(db):
    # do some actions

```

## 基于contextmanager装饰器

Python 在 contextlib 模块中还提供了一个 contextmanager 的装饰器，更进一步简化了上下文管理器的实现方式。通过 yield 将函数分割成两部分，yield 之前的语句在  ```__enter__```  方法中执行，yield 之后的语句在 ```__exit__``` 方法中执行。紧跟在 yield 后面的值是函数的返回值。

```python
# 基于contextmanager装饰器
from contextlib import contextmanager

@contextmanager
def file_manager(name, mode):
    print("file_manager() called")
    try:
        f = open(name, mode)
        yield f
    finally:
        f.close()
        print("文件关闭操作")

with file_manager('test.txt', 'w') as f:
    print("with 代码块")
    f.write('hello world')
    print("with 代码块结束")
print("with 语句结束")
```

```plaintext
file_manager() called
with 代码块
with 代码块结束
文件关闭操作
with 语句结束
```

## 注意事项

+ **确保实现所有必要的方法**：自定义上下文管理器时，需要实现```__enter()__```()和```__exit__()```方法。对于异步上下文管理器，则需要实现```__aenter__()```和```__aexit__()```。

+ **异常处理**：在```__exit__()```或```__aexit__()```方法中，确保正确处理所有可能的异常。考虑是否需要捕获异常、记录日志或者重新抛出异常。

+ **资源清理**：上下文管理器的主要目的是管理资源的生命周期。确保在退出上下文时，所有资源（如文件句柄、网络连接、锁等）都被正确释放或重置。

+ **避免副作用**：```__enter()__```方法应该只负责初始化操作，避免产生副作用，比如修改外部状态或执行I/O操作。

+ **使用as子句**：当使用with语句时，使用as子句来赋予上下文管理器返回的对象一个名称，这样可以在块内引用该对象。

+ **注意上下文管理器的嵌套**：当上下文管理器嵌套使用时，确保内层上下文管理器的退出不会影响外层上下文管理器的状态。

+ **线程安全**：大多数同步上下文管理器不是线程安全的。如果你的上下文管理器涉及共享资源，确保在多线程环境中正确地同步访问。

+ **避免阻塞操作**：在异步上下文管理器中，避免在```__aenter__()```或```__aexit__()```中执行阻塞操作，这会破坏异步性能。

+ **使用上下文管理器协议**：如果你的类需要与上下文管理器一起使用，确保它遵循上下文管理器协议，即实现必要的特殊方法。

+ **避免循环依赖**：在使用上下文管理器时，避免创建循环依赖，这可能导致资源无法释放。

## 参考资料

1. [Python深度解析：上下文协议设计与应用技巧](https://blog.csdn.net/qq_32763643/article/details/141278701)

2. [Python with关键字原理详解](https://cloud.tencent.com/developer/article/1809089)
