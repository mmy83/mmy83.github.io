---
title: java基础-synchronized实现原理
author: mmy83
date: 2024-03-26 14:57:00 +0800
categories: [编程, java]
tags: [线程, synchronized, monitor, 同步, java]
math: true
mermaid: true
image:
  path: /images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-00.jpg
  lqip: data:image/webp;base64,UklGRlQAAABXRUJQVlA4IEgAAADQAQCdASoIAAUAAUAmJQBOgB8xitA9gAD+CyM/07vlY32ZX2F0u+fo9rdR0rnr3X3ptNmxgXpzuIBSWpkEkjwfvtrACQYAAAA=
  alt: java基础-synchronized实现原理
---

## 简介

&emsp;&emsp;synchronized是互斥同步的同步机制，互斥同步又称堵塞同步。synchronized在多线程环境下，其中一条线程获得锁，其他线程需要堵塞等待持有锁的线程释放锁。

&emsp;&emsp;synchronized是块结构的同步语法，synchronized需要指定对象参数。如果synchronized没有指定对象，Java编译器通过synchronized修饰的方法检查synchronized修饰的是对象方法还是类方法，如果是对象方法，那么指定的就是当前对象实例，如果是类方法，就会指定当前类。

## Monitor

&emsp;&emsp;在Java中，Monitor（监视器）是一种同步机制，用于管理多线程之间的互斥访问和并发控制。每个对象都有一个关联的Monitor，通过使用Monitor，可以确保在任何给定时间只有一个线程可以进入带有相同监视器的同步代码。这样可以避免多个线程同时访问和修改共享的数据，从而防止数据竞争和一致性问题的发生。这也是为啥synchronized必须要有一个对象的原因，也是synchronized实现同步的关键。

## synchronized原理

&emsp;&emsp;synchronized是通过monitorenter和monitorexit字节码指令实现的，monitorenter是获取指定对象的锁，monitorexit是释放指定对象的锁，分别对应Java虚拟机字节码指令lock和unlock。Java虚拟机字节码指令lock和unlock都是原子操作，所以，synchronized具有原子性。

&emsp;&emsp;monitorenter和monitorexit字节码指令需要指定reference类型的参数来说明锁定和解锁的对象。synchronized需要指定对象参数，对象参数引用就是monitorenter和monitorexit字节码指令需要的reference。

&emsp;&emsp;执行monitorenter字节码指令时，执行字节码指令的线程需要尝试获取指定对象的锁，如果，获取指定的对象没有锁，又或者获取指定对象的锁已经被该线程锁定，那么，该对象锁的计数器加1。如果，指定对象的锁已经被其他线程持有，线程获取锁失败，那么，获取该对象锁失败的线程需要堵塞等待，直到持有该对象锁的线程释放锁为止。

&emsp;&emsp;执行monitorexit字节码指令时，指定对象锁的计数器减1，直到计数器到0，持有该对象锁的线程释放该对象的锁。

&emsp;&emsp;通过monitorenter和monitorexit字节码指令的机制，可以推测出synchronized的两点结论：

+ 以获取锁的线程可以重复获取锁：synchronized修饰的同步代码块，允许线程重复获取指定对象的锁，只是对象的锁的计数器加1。也就是说，获取该对象锁的线程允许重入获取对象的锁，而不用担心死锁问题。
+ 其他线程不能强制以获取锁的线程释放锁：synchronized修饰的同步代码块，一旦指定对象的锁被线程持有，其他获取该对象锁的线程必须进入堵塞等待状态，等待持有该对象锁的线程释放该对象锁。也就是说，一旦指定对象的锁被线程持有，其他获取该对象锁的线程不能强制已持有该对象锁的线程释放锁，也不能将获取该对象锁失败的被堵塞的线程中止或者超时退出等待。

## synchronized的使用

&emsp;&emsp;以下代码是通过不同的方式对线程进行同步，效果均为t1先获取锁，然后等待1一分钟后执行完毕释放锁，t2获取锁开始执行，等待一分钟后t2释放锁，程序结束。这里需要用到JDK的两个命令jps、jstack命令：

+ jps：用于列出当前系统中正在运行的 Java 进程（Java 虚拟机进程）。它能够显示每个 Java 进程的进程 ID（PID）以及 Java 主类的名称或 JAR 文件的路径。jps 是一个轻量级的工具，但在查看 Java 进程的状态和调试问题时非常方便。

+ jstack：JVM自带的Java堆栈跟踪工具，它用于打印出给定的java进程ID、core file、远程调试服务的Java堆栈信息.

```shell
# 查看java运行pid
jps

21905 Launcher
21906 ThisMonitor   // 这个是示例程序
21907 Jps
84077 Main

# 查看堆栈信息
jstack  21906

```

### 同步代码块

```java
import java.util.concurrent.TimeUnit;

public class ThisMonitor {

    private final Object MUTEX = new Object();

    public void func1(){
        synchronized(MUTEX){
            System.out.println(Thread.currentThread().getName()+" enter to func1");
            try {
                TimeUnit.MINUTES.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void func2(){
        synchronized(MUTEX){
            System.out.println(Thread.currentThread().getName()+" enter to func2");
            try {
                TimeUnit.MINUTES.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        ThisMonitor thisMonitor = new ThisMonitor();
        new Thread(thisMonitor::func1,"t1").start();
        new Thread(thisMonitor::func2,"t2").start();
    }
}

```

![同步代码块图](/images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-01.png)

通过堆栈信息可以清楚的看到有两个线程，一个处于```TIMED_WAITING (sleeping)```状态，一个处于```BLOCKED (on object monitor)```状态，并且可以看到是被同一个```java.lang.Object```锁.

### 同步实例方法

```java

// 两个实例方法同步
import java.util.concurrent.TimeUnit;

public class ThisMonitor {

    public synchronized void func1(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void func2(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
        ThisMonitor thisMonitor = new ThisMonitor();
        new Thread(thisMonitor::func1,"t1").start();
        new Thread(thisMonitor::func2,"t2").start();
    }
}

```

![两个实例方法同步](/images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-02.png)

通过堆栈信息可以清楚的看到有两个线程，一个处于```TIMED_WAITING (sleeping)```状态，一个处于```BLOCKED (on object monitor)```状态，并且可以看到是被同一个```ThisMonitor```锁.

```java
// 实例方法与代码块同步，证明实例方法使用的是当前对象（this）作为锁

import java.util.concurrent.TimeUnit;

public class ThisMonitor {

    public synchronized void func1(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public void func2(){
        synchronized(this){
            System.out.println(Thread.currentThread().getName()+" enter to func1");
            try {
                TimeUnit.MINUTES.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
        ThisMonitor thisMonitor = new ThisMonitor();
        new Thread(thisMonitor::func1,"t1").start();
        new Thread(thisMonitor::func2,"t2").start();
    }
}

```

![实例方法与代码块同步](/images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-03.png)

通过堆栈信息可以清楚的看到有两个线程，一个处于```TIMED_WAITING (sleeping)```状态，一个处于```BLOCKED (on object monitor)```状态，并且可以看到是被同一个```ThisMonitor```锁.这里说明实例方法使用的就是当前实例作为锁。

### 同步类方法

```java

// 静态方法同步
import java.util.concurrent.TimeUnit;

public class ThisMonitor {

    public synchronized static void func1(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized static void func2(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args){
//        ThisMonitor thisMonitor = new ThisMonitor();
        new Thread(ThisMonitor::func1,"t1").start();
        new Thread(ThisMonitor::func2,"t2").start();
    }
}

```

![静态方法同步](/images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-04.png)

通过堆栈信息可以清楚的看到有两个线程，一个处于```TIMED_WAITING (sleeping)```状态，一个处于```BLOCKED (on object monitor)```状态，并且可以看到是被同一个```java.lang.Class for ThisMonitor```锁。

```java

//通过静态方法和代码块同步，证明静态方法使用的是类对象作为锁

import java.util.concurrent.TimeUnit;

public class ThisMonitor {

    public synchronized static void func1(){
        System.out.println(Thread.currentThread().getName()+" enter to func1");
        try {
            TimeUnit.MINUTES.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void func2(){
        synchronized(ThisMonitor.class){
            System.out.println(Thread.currentThread().getName()+" enter to func1");
            try {
                TimeUnit.MINUTES.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args){
//        ThisMonitor thisMonitor = new ThisMonitor();
        new Thread(ThisMonitor::func1,"t1").start();
        new Thread(ThisMonitor::func2,"t2").start();
    }
}

```

![静态方法和代码块同步](/images/2024-03-26/java基础-synchronized实现原理/java基础-synchronized实现原理-05.png)

通过堆栈信息可以清楚的看到有两个线程，一个处于```TIMED_WAITING (sleeping)```状态，一个处于```BLOCKED (on object monitor)```状态，并且可以看到是被同一个```java.lang.Class for ThisMonitor```锁。这里说明类方法使用的就是当前类实例作为锁。

## 注意

+ synchronized的对象不能为空，因为空没有关联Monitor。
+ synchronized作用域不要太大，影响效率。
+ 不要企图用不同的Monitor锁住同一个代码块或方法，即：synchronized的对象必须是一个才可以锁住同一段代码或方法。
+ 注意多个锁出现的交叉锁死锁问题。
