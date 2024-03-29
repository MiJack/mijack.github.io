---
layout: post
catalog: true
title: JVM的垃圾回收(二) -- JVM中的分代垃圾回收机制
date: 2018-03-18
tags: 
- Java
- JVM
- 垃圾回收
- 分代垃圾回收机制
- 内存模型
- 引用
---
> 声明：我已委托「维权骑士」（rightknights.com）为我的文章进行维权行动。

# JVM 的内存模型

JVM的内存模型，往往是指Java程序在运行时内存的模型，运行时内存模型，分为线程私有和共享数据区两大类，其中线程私有的数据区包含程序计数器、虚拟机栈、本地方法区，所有线程共享的数据区包含Java堆、方法区。

![](/imgs/JVM-memory.png)

###  方法区（Method Area）
 - 方法区的作用与传统语言中的编译代码储存区非常类似，它存储了每一个类的结构信息，例如运行时常量池（Runtime Constant Pool）、字段和方法数据、构造函数和普通方法的字节码内容，还包括一些在类、实例、接口初始化时用到的特殊方法。
 - 在 Java 虚拟机中，方法区（Method Area）是可供各条线程共享的运行时内存区域。
- 方法区在虚拟机启动的时候被创建，**虽然方法区是堆的逻辑组成部分，但是简单的虚拟机实现可以选择在这个区域不实现垃圾收集**。
- 方法区的容量可以是固定大小的，也可以随着程序执行的需求动态扩展或者收缩。方法区在实际内存空间中可以是不连续的。
- 当方法区内存空间不能满足内存分配请求，那 Java 虚拟机将抛出`OutOfMemoryError `异常。
###  堆（Heap）

- 堆是**可供各条线程共享**的运行时内存区域，用于**存储所有的Java 类实例或者数组对象**。

- 堆中的对象收到垃圾收集器的管理，后者是JVM中垃圾回收算法的实现者。虚拟机实现者可以根据系统的实际需要来选择自动内存管理技术。
- Java 堆的容量可以是固定大小的，也可以随着程序执行的需求动态扩展或者收缩。Java 堆所使用的内存不需要保证是连续的。


- Java 虚拟机实现应当提供给程序员或者最终用户调节 Java 堆初始容量的手段，对于可以动态扩展和收缩 Java 堆来说，则应当提供调节其最大、最小容量的手段。


- **如果实际所需的堆超过了系统能提供的最大容量**，那 Java 虚拟机将会抛出`OutOfMemoryError` 异常。


### Java 虚拟机栈（Stack）

- Java栈属于线程私有的，他的生命周期和线程相同，描述的是Java在执行方法过程中的内存模型。
- 当线程中的一个方法执行时，虚拟机就会在栈中创建一个栈帧，用于存储局部变量、操作数栈、动态链接、方法出口等信息。一个方法的执行过程就等同于栈帧进栈出栈的过程。
- 在JVM中，若线程请求栈的深度 **超过了虚拟机允许的最大深度** ，则会抛出`StackOverflowError`异常；当栈进行动态扩展，但 **无法申请到相应内存空间（此时，线程请求的栈深度未超过虚拟机允许的最大深度）**时，则会抛出`OutOfMemoryError`异常。


### 程序计数寄存器（Program Count Register，PC Register）
- 在任意时刻，一个Java虚拟机线程只会执行一个方法，而 PC Java 虚拟机允许多条线程同时执行。因此，**每一条 Java虚拟机线程都有自己的 PC 寄存器** ，他们是相互独立的。
- 我们称正在被线程执行的方法称为该线程的当前方法（Current Method）。如果这个方法不是 native 的，那 PC 寄存器就保存 Java 虚拟机正在执行的字节码指令的地址，如果该方法是 native 的，那 PC 寄存器的值是 undefined。


### 本地方法栈（Native Method Stack）

- Java 虚拟机在实现时，可以根据自身需求，确定是否实现本地方法栈。在实现上，JVM可能会使用到传统的栈（通常称之为“C Stacks”）来支持 native 方法（指使用 Java 以外的其他语言编写的方法）的执行，这个栈就是本地方法栈。当JVM支持本地方法栈时，这个栈会在线程创建时按照线程分配。
- Java 虚拟机规范允许本地方法栈被实现成固定大小的或者是根据计算动态扩展和收缩的。
- 在JVM中，本地方法栈可能发生的异常情况也分为Stack OverflowError和OutOfMemoryError异常，出错原因和虚拟机栈相似，此处不再赘述。

# 分代回收机制

## 为什么不能直接使用垃圾回收算法？

在上一篇文章中[ JVM的垃圾回收(一) -- 常见的垃圾标识和回收算法](/2018/03/03/JVM-Garbage-Collection)，我们一共介绍了标记-清除算法、复制算法以及标记整理算法等方法。由于标记-清除算法在完成垃圾回收后无法保证可以整理出更多更大的 **连续内存区域**，因此JVM在算法选择上采用的是后面两者。**而复制算法需要频繁的拷贝整个内存空间，对大内存空间对应的时间开销比较大，标记清除算法中标记的时间可能花费过长，不利于频繁使用**。为此，JVM采用了分代回收机制，按照对象存活时间对内存进行分区，在不同的区域采用不同的垃圾回收算法，提升垃圾回收的效率。

## 内存的分代

在具体实现上，JVM将需要进行垃圾回收的区域分为3类：新生代（Young Generation）、老年代(Old Generation)、永久代(Permanent Generation)。其中新生代又分为eden和survival（S0、S1）。具体结构如下：

![](/imgs/JVM-Runtime-Data-Area.png)

简单讲，新生代的eden、新生代的survival、老生代里的对象存活时间依次变长。



| **参数名称**      | **含义**                                            |
| ----------------- | ---------------------------------------------------------- |
| -Xms              | 初始堆大小                                                 |
| -Xmx              | 最大堆大小                                                 |
| -XX:NewSize       | 设置年轻代大小(for 1.3/1.4)                                |
| -XX:MaxNewSize    | 年轻代最大值(for 1.3/1.4)                                        |
| -XX:PermSize      | 设置持久代(perm gen)初始值                                       |
| -XX:MaxPermSize   | 设置持久代最大值                                                  |
| -Xss              | 每个线程的堆栈大小                                        |
| -XX:NewRatio      | 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代) |
| -XX:SurvivorRatio | Eden区与Survivor区的大小比值                               |



## GC root是谁？

在JVM中，垃圾标识算法使用采用的根搜索算法。对应的GC Root则是

- 虚拟机栈(栈桢中的本地变量表)中的引用的对象 
- 方法区中的类静态属性引用的对象 
- 方法区中的常量引用的对象 
- 本地方法栈中JNI的引用的对象

## 垃圾回收的具体策略

#### 新生代——复制算法

我们称新生代的垃圾回收为**Minor Garbage Collection**，同时，每个对象都有一个**“年龄”**，这个年龄实际上指的就是该对象经历过的minor gc的次数。在新生代采用的垃圾回收属于复制算法，主要原因是新生代区域中的对象数量较少，存活时间短，在对象复制过程中的耗时较少。具体的垃圾回收策略如下：

当GC进行后，

1. 对于Eden中还存活的对象，它们将被复制到survival的to区域；
2. 对于from中存活的对象，若它们的年龄小于规定的年龄，它们将被复制到survival的to区域，反之将被复制到老生代；
3. 当复制算法执行结束后，将会清空Eden和from区域，to区域变得非空闲，因此原来的from区域变成了to区域，而to区域变成了from区域

> 注：新生代中的两个survival区域大小上是保持一致的，在任意时刻，空闲的survival区域成为from区域，而非空闲的区域成为to区域，对象可以从from区域拷贝到to区域，但是不能从to区域拷贝到from区域。

#### 老生代——标记-整理算法

老生代采用的垃圾回收称为**Major Garbage Collection**，采用的是**标记-整理算法**，主要原因是老生代区域中的对象生命周期较长，采用复制算法将会产生频繁的复制操作，对应的开销较大，而标记整理算法可以很好的避免这个问题，提升垃圾回收的效率。

#### 永生代

永久代用于存放静态文件，如Java类、方法等，引用关系比较稳定，一般不进行垃圾管理。但是，有些应用可能会动态生成Class，在有些JVM的实现中会给永生代添加一些必要的回收算法。





# 参考资料

1. [Jvm内存模型](http://gityuan.com/2016/01/09/java-memory/)
2. [Java 的垃圾回收机制的主要原理是什么？什么情况下会影响到程序的运行效率？ - 胖胖的回答 - 知乎](https://www.zhihu.com/question/19912197/answer/113385483)
3. [Java Memory Management](https://www.dynatrace.com/resources/ebooks/javabook/how-garbage-collection-works/)
4. [JVM系列三:JVM参数设置、分析](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)
