---
title: java基础-Thread类
author: mmy83
date: 2024-03-18 13:49:00 +0800
categories: [语言, java]
tags: [线程, Thread, Runnable, java]
math: true
mermaid: true
image:
  path: /images/2024-03-18/java基础-Thread类/java基础-Thread类-00.webp
  lqip: data:image/webp;base64,UklGRmQAAABXRUJQVlA4IFgAAADQAQCdASoIAAYAAUAmJYwCdAD0Z/xR2AD+T8li7XJjlsgoRR/4dTYRvO/Jq/QcitH13zq50m1PnBjNDTYw4ofR46rbjqLPGAStMrCFgLfGApWMp4wYAAAA
  alt: java基础-Thread类
---

## 前言

&emsp;&emsp;很早以前就接触了java的并发，当时还是做一个抓取数据，主线程打开目录，创建一个下载线程来下载文件。需求很简单，用java开发的，目的就是学习一下java语言。但是没有更深入的了解过，只是看各种资料上说“创建一个线程有两个方法，一个是继承Thread类，一个是实现一个Runnable接口”，当时认为java是单继承，接口是多实现，所有提供两个方式：

1、当你需要继承其他类的时候，就只能用接口，因为一个类不能同时继承两个类；

2、当不需要继承其他类的时候就随便用哪个方法都可以；

&emsp;&emsp;现在想想也没啥大错，只是随着对java线程了解的增加，“创建线程有两个方法”的说法不太严谨。为什么这样说呢，原因很简单，即便是我们使用Runnable接口来实现，你也离不开Thread这个类，只不过是把Runnable接口的实现作为参数传递给Thread了（或者说注入进去），而且Thread本身又是Runnable的一个实现。那就是说其实真正创建线程的和这个Runnable接口没啥直接关系，这个Runnable接口只是用来管理新创建的线程里要执行的代码的，而真正创建线程的还是Thread类。

## Thread类

```java

// java jdk 21
public class Thread implements Runnable {

    //这里省略n行代码

    // Additional fields for platform threads.
    // All fields, except task, are accessed directly by the VM.
    private static class FieldHolder {
        final ThreadGroup group;
        final Runnable task;
        final long stackSize;
        volatile int priority;
        volatile boolean daemon;
        volatile int threadStatus;

        FieldHolder(ThreadGroup group,
                    Runnable task,
                    long stackSize,
                    int priority,
                    boolean daemon) {
            this.group = group;
            this.task = task;
            this.stackSize = stackSize;
            this.priority = priority;
            if (daemon)
                this.daemon = true;
        }
    }
    private final FieldHolder holder;

    //这里省略n行代码
    Thread(ThreadGroup g, String name, int characteristics, Runnable task,
           long stackSize, AccessControlContext acc) {

        Thread parent = currentThread();
        boolean attached = (parent == this);   // primordial or JNI attached

        if (attached) {
            if (g == null) {
                throw new InternalError("group cannot be null when attaching");
            }
            this.holder = new FieldHolder(g, task, stackSize, NORM_PRIORITY, false);
        } else {
            SecurityManager sm = System.getSecurityManager();
            if (g == null) {
                // the security manager can choose the thread group
                if (sm != null) {
                    g = sm.getThreadGroup();
                }

                // default to current thread's group
                if (g == null) {
                    g = parent.getThreadGroup();
                }
            }

            // permission checks when creating a child Thread
            if (sm != null) {
                sm.checkAccess(g);
                if (isCCLOverridden(getClass())) {
                    sm.checkPermission(SecurityConstants.SUBCLASS_IMPLEMENTATION_PERMISSION);
                }
            }

            int priority = Math.min(parent.getPriority(), g.getMaxPriority());
            this.holder = new FieldHolder(g, task, stackSize, priority, parent.isDaemon());
        }

        //这里省略构造函数的n行代码
    }

    //这里省略n行代码

    public Thread() {
        this(null, null, 0, null, 0, null);
    }

    //这里没省略代码，只省略了n行注释

    public Thread(Runnable task) {
        this(null, null, 0, task, 0, null);
    }

    //这里省略n行代码

    public void start() {
        synchronized (this) {
            // zero status corresponds to state "NEW".
            if (holder.threadStatus != 0)
                throw new IllegalThreadStateException();
            start0();
        }
    }

    //这里省略n行代码
    private native void start0();

    /**
     * This method is run by the thread when it executes. Subclasses of {@code
     * Thread} may override this method.
     *
     * <p> This method is not intended to be invoked directly. If this thread is a
     * platform thread created with a {@link Runnable} task then invoking this method
     * will invoke the task's {@code run} method. If this thread is a virtual thread
     * then invoking this method directly does nothing.
     *
     * @implSpec The default implementation executes the {@link Runnable} task that
     * the {@code Thread} was created with. If the thread was created without a task
     * then this method does nothing.
     */
    @Override
    public void run() {
        Runnable task = holder.task;
        if (task != null) {
            Object bindings = scopedValueBindings();
            runWith(bindings, task);
        }
    }
    //这里省略n行代码

}

```

&emsp;&emsp;上面这一大段代码，是Thread类中摘取的一部分，为了简单，省略了很多暂时不关心的代码，Thread有很多构造函数，这里只摘取了三个，不难发现是通过调用完整签名的构造函数（也有不是的），现在捋一下这些代码：

1、Thread是一个Runnable接口的实现，那么Thread里一定有一个run方法的实现。

2、我们实例化一个Thread对象或者Thread子类的对象，并执行start方法，可以看到start方法又去执行了start0方法，而这个start0是一个native方法，也就是说，这个不是java实现的，而是调用了系统接口，这个也很好理解，创建并管理线程都是由系统统一实现的。所有在这里调用jni方法很正常。而从start0到run这段代码在jdk底层里。因为Oracle JDK不开源，这里引用一篇高手的博文，是通过openjdk来分析java线程的：

[java中线程启动过程分析及本地方法 start0源代码的追踪学习](https://blog.csdn.net/fengxiandada/article/details/123432662)

从上面博文可以看出，他最终还是运行到了run方法。

3、这里面有一个属性```holder```，其中有一个```Runnable```类型的```task```。可以发现这个才是最终运行的代码。

4、而这个run是Thread的默认实现，如果我们通过继承Thread重写run方法来创建线程，这个方法就是我们重写的代码。而如果我们使用实现Runnable接口的方法来创建线程，那么这个```task```就是Runnable实现的run。

## 总结

&emsp;&emsp;通过上面代码的分析，不难发现不管是实现Runnable还是继承Thread，最终真正调用系统创建线程的都是Thread，而继承Thread方式运行的代码是重写run的代码；实现Runnable接口则是通过实现Runnable的run方法来管理运行代码的。

&emsp;&emsp;技术还是不行，底层代码没有看过，只能看看别人的代码分析，如有错误欢迎大家指出。
