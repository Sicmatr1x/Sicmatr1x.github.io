---
title: JVM-8_堆
date: 2020-08-06 15:45:52
categories:
    - 后端
tags: 
    - 面试
    - JVM
---

## 堆的核心概念

堆针对一个JVM进程来说是唯一的，也就是一个进程只有一个JVM，但是进程包含多个线程，他们是共享同一堆空间的。

一个JVM实例只存在一个堆内存，堆也是Java内存管理的核心区域。

Java堆区在JVM启动的时候即被创建，其空间大小也就确定了。是JVM管理的最大一块内存空间。

- 堆内存的大小是可以调节的。

《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的。

所有的线程共享Java堆，在这里还可以划分线程私有的缓冲区（Thread Local Allocation Buffer，TLAB）。

> -Xms10m：最小堆内存
>
> -Xmx10m：最大堆内存

下图就是使用：Java VisualVM(C:\Program Files\Java\jdk1.8.0_231\bin\jvisualvm.exe)查看堆空间的内容，通过 jdk bin提供的插件

<img src="image-20200706200739392.png">

《Java虚拟机规范》中对Java堆的描述是：所有的对象实例以及数组都应当在运行时分配在堆上。（The heap is the run-time data area from which memory for all class instances and arrays is allocated）

我要说的是：“几乎”所有的对象实例都在这里分配内存。—从实际使用角度看的。

- 因为还有一些对象是在栈上分配的

数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。

在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。

- 也就是触发了GC的时候，才会进行回收
- 如果堆中对象马上被回收，那么用户线程就会收到影响，因为有stop the word

堆，是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域。

<img src="image-20200706201904057.png">

### 堆内存细分

Java 7及之前堆内存逻辑上分为三部分：新生区+养老区+永久区

- (Young Generation Space)新生区: Young/New, 又被划分为Eden区和Survivor区
- (Tenure generation space)养老区: Old/Tenure
- (Permanent Space)永久区: Perm

Java 8及之后堆内存逻辑上分为三部分：新生区养老区+元空间
- (Young Generation Space)新生区: Young/New, 又被划分为Eden区和Survivor区
- (Tenure generation space)养老区: Old/Tenure
- (Meta Space)元空间: Meta

约定同义词：
- Young Generation Space = 新生区 = 新生代 = 年轻代 
- Tenure generation space = 养老区 = 老年区 = 老年代
- Permanent Space = 永久区 = 永久代

<img src="image-20200706203419496.png">

堆空间内部结构，JDK1.8之前从永久代 ->  元空间

<img src="image-20200706203835403.png">

## 设置堆内存大小与OOM

Java堆区用于存储Java对象实例，那么堆的大小在JVM启动时就已经设定好了，大家可以通过选项"-Xmx"和"-Xms"来进行设置。

- `-Xms`用于表示堆区的起始内存，等价于-xx:InitialHeapSize
- `-Xmx`则用于表示堆区的最大内存，等价于-XX:MaxHeapSize

一旦堆区中的内存大小超过`-xmx`所指定的最大内存时，将会抛出outofMemoryError异常。

通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的是**为了能够在ava垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，避免频繁的扩容或释放，对系统造成额外的压力，从而提高性能**。

默认情况下:
- 初始内存大小：物理电脑内存大小/64
- 最大内存大小：物理电脑内存大小/4

如何查看堆内存的内存分配情况

```
jps  ->  staat -gc  进程id
```

在程序运行结束后打印堆空间详情：

```
-XX:+PrintGCDetails
```





