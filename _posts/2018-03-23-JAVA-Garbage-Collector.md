---
layout: post
catalog: true
title: JVM的垃圾回收(三) -- JVM垃圾收集器
date: 2018-03-23
tags: 
- Java
- JVM
- 垃圾回收
- 垃圾收集器
---

# 前言

在之前的两篇博客，我们大致介绍了，常见的垃圾回收算法及JVM中常见的分类回收算法。这些都是从算法和规范上分析Java中的垃圾回收，属于方法论。

在JVM中，垃圾回收的具体实现是由垃圾收集器（Garbage Collector）负责的。

在JVM中，具体实现有Serial、ParNew、Parallel Scavenge、CMS、 Serial Old（MSC）、Parallel Old、G1等。在下图中，你可以看到不同垃圾收集器适合于不同的内存区域，如果两个垃圾收集器之间存在连线，那么表示两者可以配合使用。

如果当垃圾收集器进行垃圾清理时，必须暂停其他所有的工作线程，直到它完全收集结束。我们称这种需要暂停工作线程才能进行清理的策略为`Stop-the-World`.
以上收集器中，Serial、ParNew、Parallel Scavenge、 Serial Old、Parallel Old均采用的是Stop-the-World的策略。

![](/imgs/Garbage-Collector-set.jpg)
# 收集器介绍

## 单线程版的收集器
### Serial 收集器
Serial 收集器是最基本的新生代垃圾收集器，是**单线程**的垃圾收集器。

由于垃圾清理时，Serial收集器不存在线程间的切换，因此，特别是在**单CPU**的环境下，它的垃圾清除效率比较高。对于Client运行模式的程序，选择Serial收集器是一个不错的选择。


### Serial Old收集器

Serial Old收集器是Serial 收集器的老生代版本，属于单线程收集器。它常用于Client模式下的虚拟机；对于Server模式下的虚拟机，在JDK1.5及其以前，它常与Parallel Scavenge收集器配合使用，达到较好的吞吐量，另外它也是CMS收集器在Concurrent Mode Failure时的后备方案。
Serial 收集器和Serial Old收集器的执行效果如下：
![](/imgs/serail-and-serail-old.png)


## 多线程版的收集器


### ParNew收集器
ParNew收集器是在Serial 收集器的基础上演化而来的，属于Serial 收集器的**多线程**版本，同样运行在**新生代**区域。在实现上，两者共用很多代码。
在不同运行环境下，根据CPU的核数，开启不同的线程数，从而达到最优的垃圾收集效果。
对于那些Server模式的应用程序，如果考虑采用CMS作为老生代回收器时，ParNew收集器是一个不错的选择。
![](/imgs/parnew-and-serail-old.png)


### Parallel Scavenge收集器

和ParNew收集器一样，Parallel Scavenge收集器也是运行在**新生代**区域，属于多线程的收集器，但不同的是，ParNew收集器是通过控制垃圾回收的线程数来进行参数调整，而**Parallel Scavenge收集器更关心的是程序运行的吞吐量，即一段时间内，用户代码运行时间占总运行时间的百分比**。

### Parallel Old收集器
Parallel Old收集器是 Parallel Scavenge 收集器的**老生代**版本，属于多线程收集器，采用标记-整理算法。Parallel Old收集器和Parallel Scavenge收集器同样考虑了吞吐量优先这一指标，非常适合那些注重吞吐量和CPU资源敏感的场合。
![](/imgs/parallel-old-and-parallel-scavenge.png)



## 另一种收集器

### CMS收集器
CMS（Concurrent Mark Sweep）收集器是在最短回收停顿时间为前提的收集器，属于多线程收集器，采用**标记-清除**算法。

相比之前的收集器，CMS收集器的运作过程比较复杂，分为四步：
1. 初始标记（CMS initial mark）
2. 并发标记（CMS concurrent mark）
3. 重新标记（CMS remark）
4. 并发清除（CMS concurrent sweep）

其中，初始标记、重新标记需要stop the world。初始标记仅仅是标记GC Roots内直接关联的对象；并发标记进行的是GC Tracing；重新标记阶段为了修正`并发期间由于用户进行运作导致的标记变动的那一部分对象的标记记录`。

在整个过程中，CMS收集器的内存回收基本上和用户线程并发执行，如下所示：
![](/imgs/concurrent-mark-sweep-garbage-collector.png)

由于CMS收集器并发收集、低停顿，因此有些地方成为并发低停顿收集器（Concurrent Low Pause Sweep Collector）。
#### CMS收集器的缺点
但是，CMS收集器也不是没有缺点：
1. **CMS收集器对CPU资源非常依赖**<br/>
    CMS收集器过分依赖于多线程环境，默认情况下，开启的线程数为（CPU的数量+3）/4，当CPU数量少于4个时，CMS对用户查询的影响将会很大，因为他们要分出一半的运算能力去执行收集器线程；
2. **CMS收集器无法清除浮动垃圾**<br/>
    由于CMS收集器清除已标记的垃圾（处于最后一个阶段）时，用户线程还在运行，因此会有新的垃圾产生，但是这部分垃圾未被标记，在下一次GC才能清除，因此被成为浮动垃圾。由于内存回收和用户线程是同时进行的，内存在被回收的同时，也在被分配。当**老生代中的内存使用超过一定的比例**时，系统将会进行垃圾回收；当**剩余内存不能满足程序运行要求**时，系统将会出现Concurrent Mode Failure，临时采用Serial Old算法进行清除，此时的性能将会降低。
3. **垃圾收集结束后残余大量空间碎片**<br/>
    CMS收集器采用的标记清除算法，本身存在垃圾收集结束后残余大量空间碎片的缺点。CMS配合适当的内存整理策略在一定程度上可以解决这个问题。

# 参考资料
1. [深入理解Java虚拟机（第2版）](https://book.douban.com/subject/24722612/)
2. [JVM（二）垃圾收集算法与收集器](http://alicharles.com/article/jvm-gc/)