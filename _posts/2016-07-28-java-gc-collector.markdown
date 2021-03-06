---
layout:     post
title:      "Java的GC的收集器"
subtitle:   "GC机制收集器详解"
date:       2016-07-28
author:     "MAI HAO"
header-img: "img/post-bg-js-module.jpg"
tags:
    - gc
---

## Java内存分配机制

要了解Java的GC机制，首先需要了解的是Java的内存分配机制。Java内存分配和回收的机制概括的说，就是：分代分配，分代回收。对象将根据存活的时间被分为：年轻代（Young Generation）、年老代（Old Generation）、永久代（Permanent Generation，也就是方法区）。如下图所示：
![gc](/img/gc/gc.jpg)

**年轻代（Young Generation）**：对象被创建时，内存的分配首先发生在年轻代（大对象可以直接 被创建在年老代），大部分的对象在创建后很快就不再使用，因此很快变得不可达，于是被年轻代的GC机制清理掉（IBM的研究表明，98%的对象都是很快消亡的），这个GC机制被称为Minor GC或叫Young GC。

> 注意，Minor GC并不代表年轻代内存不足，它事实上只表示在Eden区上的GC。

年轻代上的内存分配是这样的，年轻代可以分为3个区域：Eden区（伊甸园，亚当和夏娃偷吃禁果生娃娃的地方，用来表示内存首次分配的区域，再 贴切不过）和两个存活区（Survivor 0 、Survivor 1）。在GC算法中，使用Copying算法来处理年轻代，所以可以理解为两个survivor就是from和to空间。

> 当存活次数超过15次，或者survivor放不下的时候，就会去到年老代。次数可以用参数来调整，15次是默认

**年老代（Tenured Generation）**：对象如果在年轻代存活了足够长的时间而没有被清理掉（即在几次 Young GC后存活了下来），则会被复制到年老代，年老代的空间一般比年轻代大，能存放更多的对象，在年老代上发生的GC次数也比年轻代少。当年老代内存不足时， 将执行Major GC，也叫 Full GC。老年代存储的对象比年轻代多得多，而且不乏大对象，对老年代进行内存清理时，如果使用停止-复制算法，则相当低效。一般，老年代用的算法是Mark-Swap算法.

**永久代（Perm）**:主要是用来存储方法的参数，常量等的。

## GC中的垃圾收集器

* **Serial收集器**：新生代收集器，使用停止复制算法，使用一个线程进行GC，其它工作线程暂停。

* **ParNew收集器**：新生代收集器，使用停止复制算法，Serial收集器的多线程版，用多个线程进行GC，其它工作线程暂停，关注缩短垃圾收集时间。

* **Parallel Scavenge 收集器**：新生代收集器，使用停止复制算法，关注CPU吞吐量，即运行用户代码的时间/总时间，比如：JVM运行100分钟，其中运行用户代码99分钟，垃 圾收集1分钟，则吞吐量是99%，这种收集器能最高效率的利用CPU，适合运行后台运算。

* **Serial Old收集器**：老年代收集器，单线程收集器，使用Mark-Swap算法，使用单线程进行GC，其它工作线程暂停。

* **Parallel Old收集器**：老年代收集器，多线程，多线程机制与Parallel Scavenge差不错，使用Mark-Swap算法。

* **CMS（Concurrent Mark Sweep）收集器**：老年代收集器，致力于获取最短回收停顿时间，使用Mark-Swap算法。
在CMS算法中，有六大步骤，分别为：初始标记(CMS-initial-mark) -> 并发标记(CMS-concurrent-mark) -> 重新标记(CMS-remark) -> 并发清除(CMS-concurrent-sweep) ->并发重设状态等待下次CMS的触发(CMS-concurrent-reset)。其中第一个和第三个都需要暂停其他线程的，第一个过程从根节点开始标记，第三个是由于并发标记过程中存在遗漏（对象有更新），所以要重新标记一次。

> 并发指的是一个或者多个垃圾回收线程和应用程序线程并发地运行，垃圾回收线程不会暂停应用程序的执行，如果你有多于一个处理器，那么并发收集线程将与应用线程在不同的处理器上运行，显然，这样的开销就是会降低应用的吞吐量。

