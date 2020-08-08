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

### OutOfMemory举例

```java
package com.sicmatr1x.java;

import java.util.ArrayList;
import java.util.Random;

public class OOMTest {
    public static void main(String[] args) {
        ArrayList<Picture> list = new ArrayList<>();
        while(true){
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024 * 1024)));
        }
    }
}

class Picture{
    private byte[] pixels;

    public Picture(int length) {
        this.pixels = new byte[length];
    }
}

```

然后设置启动参数

```
-Xms600m -Xmx600m
```

<img src="javaw_s9VJ3r66ne.png">

错误提示：

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.sicmatr1x.java.Picture.<init>(OOMTest.java:24)
	at com.sicmatr1x.java.OOMTest.main(OOMTest.java:15)

Process finished with exit code 1
```

<img src="image-20200706211652779.png">

## 年轻代与老年代

存储在JVM中的Java对象可以被划分为两类：
- 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
  - 生命周期短的，及时回收即可
- 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致

Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（oldGen）

其中年轻代又可以划分为Eden空间、Survivor0空间和Survivor1空间（有时也叫做from区、to区）

<img src="image-20200707075847954.png">

下面这参数开发中一般不会调：

<img src="image-20200707080154039.png">

- 默认-XX:NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3

- 可以修改-XX:NewRatio=4，表示新生代占1，老年代占4，新生代占整个堆的1/5

当发现在整个项目中，生命周期长的对象偏多，那么就可以通过调整 老年代的大小，来进行调优

查看新生代区占几份：`jinfo -flag NewRatio 60636`

* `-XX:NewRatio`: 设置新生代与老年代的比例。默认值是2.
* `-XX:SurvivorRatio`: 设置新生代中Eden区与Survivor区的比例。默认值是8。在HotSpot中，Eden空间和另外两个survivor空间缺省所占的比例是8：1：1当然开发人员可以通过选项`-xx:SurvivorRatio`调整这个空间比例。比如`-xx:SurvivorRatio=8`
* `-XX:-UseAdaptiveSizePolicy`: 关闭自适应的内存分配策略  （暂时用不到）
* `-Xmn`: 设置新生代的空间的大小。 （一般不设置）

几乎所有的Java对象都是在Eden区被new出来的。绝大部分的Java对象的销毁都在新生代进行了。（有些大的对象在Eden区无法存储时候，将直接进入老年代）

>IBM公司的专门研究表明，新生代中80%的对象都是“朝生夕死”的。
>
>可以使用选项"-Xmn"设置新生代最大内存大小
>
>这个参数一般使用默认值就可以了。

<img src="image-20200707084208115.png">

## 图解对象分配过程

为新对象分配内存是一件非常严谨和复杂的任务，JM的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。

1. new的对象先放伊甸园区。此区有大小限制。
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（MinorGC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区
3. 然后将伊甸园中的剩余对象移动到幸存者0区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。
6. 啥时候能去养老区呢？可以设置次数。默认是15次。
- 在养老区，相对悠闲。当养老区内存不足时，再次触发GC：Major GC，进行养老区的内存清理
- 若养老区执行了Major GC之后，发现依然无法进行对象的保存，就会产生OOM异常。


### 图解过程

我们创建的对象，一般都是存放在Eden区的，当我们Eden区满了后，就会触发GC操作，一般被称为 Y(oung)GC / Minor GC操作。**注意：S0或S1区满的时候是不会触发YGC的。如果Survivor区满了后，将会触发一些特殊的规则(参考下面的对象分配的特殊情况)，也就是可能直接晋升老年代**

<img src="image-20200707084714886.png">

1. 当我们进行一次垃圾收集后，红色的将会被回收，而绿色的还会被占用着，存放在S0(Survivor From)区。同时我们给每个对象设置了一个年龄计数器，一次回收后就是1。

<img src="image-20200707085232646.png">

2. 同时Eden区继续存放对象，当Eden区再次存满的时候，又会触发一个MinorGC操作，此时GC将会把 Eden和Survivor From中的对象 进行一次收集，把存活的对象放到 Survivor To区，同时让年龄 + 1

<img src="image-20200707085737207.png">

3. 我们继续不断的进行对象生成 和垃圾回收，当Survivor中的对象的年龄达到15的时候(可以设置参数：`-Xx:MaxTenuringThreshold=15`进行设置，默认为15)，将会触发一次 Promotion晋升的操作，也就是将年轻代中的对象晋升到老年代中


### 总结

- 针对Survivor区的总结：复制之后有交换，谁空谁是to
- 关于垃圾回收：频繁在新生代收集，很少在老年代收集，几乎不在永久代/元空间收集

### 对象分配的特殊情况

<img src="image-20200707091058346.png">


### 代码演示对象分配过程

```java
import java.util.ArrayList;
import java.util.Random;

public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) {
        ArrayList<HeapInstanceTest> list = new ArrayList<HeapInstanceTest>();
        while (true) {
            list.add(new HeapInstanceTest());
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

然后设置JVM参数

```bash
-Xms600m -Xmx600m
```

<img src="垃圾回收.gif">

最终，在老年代和新生代都满了，就出现OOM


### 常用的调优工具

- JDK命令行
- Eclipse：Memory Analyzer Tool
- Jconsole
- Visual VM（实时监控  推荐~）
- Jprofiler（推荐~）
- Java Flight Recorder（实时监控）
- GCViewer
- GCEasy


