---
layout: post
title: effective java阅读笔记-并发
---

第十章

**并发(concurrency)和并行(Parallelism)的区别，来个比喻：并发和并行的区别就是一个人同时吃三个馒头和三个人同时吃三个馒头。**
<br>
当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程,它只能把CPU运行时间划分成若干个时间段,再将时间段分配给各个线程执行,在一个时间段的线程代码运行时,其它线程处于挂起状态.这种方式我们称之为并发(Concurrency).
<br>
当系统有一个以上CPU时,则线程的操作有可能非并发.当一个CPU执行一个线程时,另一个CPU可以执行另一个线程,两个线程互不抢占CPU资源,可以同时进行,这种方式我们称之为并行(Parallelism)

<!--more-->

## 66. 同步访问共享的可变数据

### tips:
* 同步不仅可以阻止一个线程看到的对象处于不一致的状态之中，还可以保证进入到同步方法或者同步代码块的每个线程，都能看到由同一个锁保护的之前所有修改效果
* 语言规范保证了线程在读取原子数据的时候，不会看到任意的数值，但是不保证一个线程的写入的值对于另一个线程是可见的
* 不要使用`Thread.stop`
* `volatile`修饰符不执行互斥访问，但是可以保证任何一个线程在读取该域的时候都可以看到最近刚刚被写入的值，前提是该读取和写入操作都是原子的
* 如果只需要线程直接的相互通信，而不需要互斥，可以考虑`volatile`修饰符
* 最佳办法是不共享可变的数据，要么共享不可变的数据，将可变数据限制在单个线程中

## 67. 避免过度同步

### tips:
* 在一个被同步的区域内部，不要调用设计成要被覆盖的方法，或者是由客户端提供函数对象提供的方法
* 过度同步的实际成本不是指获取锁花费的cpu时间，而是指失去了并行的机会，以及因为需要确保每个核都有一个一致的内存视图而导致的延迟

## 68. executor和task优于线程

### tips:
* java 1.5以后，新增了java.util.concurrent，包括了**Executor Framework**
* 大负载服务来说，CacheThreadPool不是一个好的选择
* 尽量不直接使用线程，使用task，即`Runnable`和`Callable`

## 69. 并发工具优于wait和notify

### tips:
* java.util.concurrent中更高级的工具分为三类：Executor Framework, Concurrent Collection, Synchronizer

## 70. 线程安全性的文档化

### tips:
* 不可变的(immutable)，无条件的线程安全(unconditionally thread-safe)，有条件的线程安全(conditionally thread-safe)，非线程安全(not thread-safe)，线程对立的(thread-hostile)
* 把锁封装在他所同步的对象中

## 71. 慎用延迟初始化(lazy initialization)

### tips:
* 同步访问方法，如(71.1)
* lazy initialization holder class，如(71.2)
* 双重检查(double check)，如(71.3)，域需要被申明为volatile

### sample code:
{% highlight java %}
// sample code 71.1
private Field field;

synchronized Field getField() {
    if (field == null) {
        field = newField();
    }
    return field;
}
{% endhighlight %}

{% highlight java %}
// sample code 71.2
private static class FieldHolder {
    static final Field field = newField();
}

static Field getField() {
    return FieldHolder.field;
}
{% endhighlight %}

{% highlight java %}
// sample code 71.3
private volatile Field field;

Field getField() {
    Field result = field;
    if (result == null) {
        synchronized(this) {
            result = field;
            if (result == null) {
                result = field = newField;
            }
        }
    }
    return result;
}
{% endhighlight %}

## 72. 不要依赖线程调度器

### tips:
* 任何依赖线程调度器来达到正确性或者性能要求的程序，很有可能都是不可移植的
* 确保可运行的线程平均数量不明显多于处理器的数量
* 线程不应该处于忙-等(busy-wait)状态，即反复的检查某个共享对象，以等待某个事件的发生

## 73. 避免使用线程组

### tips:
* 忽略掉线程组，使用executor
