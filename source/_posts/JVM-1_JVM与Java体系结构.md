---
title: JVM-1_JVM与Java体系结构
date: 2020-03-06 10:04:40
categories:
    - Java
tags: 
    - 后端
    - JVM
---

## 基于栈的指令集架构与基于寄存器的指令集架构

Java编译器输入的指令流基本上是一种基于栈的指令集架构，另外一种指令集架构则是基于寄存器的指令集架构。

基于栈式架构：
- 设计和实现更简单，适用于资源受限的系统
- 避开了寄存器的分配难题：使用零地址指令方式分配
- 指令流中的指令大部分是零地址指令，其执行过程依赖于操作栈。指令集更小，编译器容易实现
- 不需要硬件支持

基于寄存器架构：
- 例如x86的二进制指令集
- 指令集架构完全依赖硬件
- 性能优秀，执行更高效
- 花费更少指令完成一项操作
- 通常为一地址指令、二地址指令、三地址指令。

例如：计算2+3

基于栈：

```
iconst_2 // 常量2入栈
istore_1
iconst_3 // 常量3入栈
istore_2
iload_1
iload_2
iadd  // 常量2，3出栈，执行相加
istore_0 // 结果5入栈
```

基于寄存器：

```
mov eax,2 // 将eax寄存器值设为2
add eax,3 // 使eax寄出器的值加3
```

### 反编译字节码案例

StackStruTest.java

```java
package com.sicmatr1x.java;

public class StackStruTest {
    public static void main(String[] args) {
        int i = 2;
        int j = 3;
        int k = i + j;
    }
}
```

使用javap反编译指令反编译StackStruTest.class字节码文件：

```
javap -v StackStruTest.class
```

反编译结果：

```
  Last modified 2020-7-30; size 486 bytes
  MD5 checksum 618bcdd36b54bf88b6efa145bd258086
  Compiled from "StackStruTest.java"
public class com.sicmatr1x.java.StackStruTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#21         // java/lang/Object."<init>":()V
   #2 = Class              #22            // com/sicmatr1x/java/StackStruTest
   #3 = Class              #23            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lcom/sicmatr1x/java/StackStruTest;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               args
  #14 = Utf8               [Ljava/lang/String;
  #15 = Utf8               i
  #16 = Utf8               I
  #17 = Utf8               j
  #18 = Utf8               k
  #19 = Utf8               SourceFile
  #20 = Utf8               StackStruTest.java
  #21 = NameAndType        #4:#5          // "<init>":()V
  #22 = Utf8               com/sicmatr1x/java/StackStruTest
  #23 = Utf8               java/lang/Object
{
  public com.sicmatr1x.java.StackStruTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/sicmatr1x/java/StackStruTest;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: iconst_2
         1: istore_1
         2: iconst_3
         3: istore_2
         4: iload_1
         5: iload_2
         6: iadd
         7: istore_3
         8: return
      LineNumberTable:
        line 5: 0
        line 6: 2
        line 7: 4
        line 8: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            2       7     1     i   I
            4       5     2     j   I
            8       1     3     k   I
}
SourceFile: "StackStruTest.java"
```

看code部分：

```
Code:
    stack=2, locals=4, args_size=1
        0: iconst_2 // 定义常量2
        1: istore_1 // 保存到操作数1的栈中
        2: iconst_3 // 定义常量3
        3: istore_2 // 保存到操作数2的栈中
        4: iload_1 // 加载操作数1的值
        5: iload_2 // 加载操作数2的值
        6: iadd // 求和
        7: istore_3 // 保存到操作数3的栈中
        8: return
```

## JVM的生命周期

1. 虚拟机的启动：JVM的启动时通过引导类加载器(bootstrap class loader)创建一个初始类(initial class)来完成的，这个类是由JVM的具体实现指定的。
2. 虚拟机的执行
3. 虚拟机的终止：
  - 程序正常执行结束
  - 程序在执行过程中遇到异常或错误而异常终止
  - 操作系统出现错误而导致JVM进程终止
  - 某线程调用Runtime类或System类的exit方法或Runtime类的halt方法，并且Java安全管理器也允许这个词exit或halt操作
  - JNI(Java Native Interface)规范描述了用JNI Invocation API来加载或卸载JVM时JVM的退出情况

## JVM发展历程

### Sum Classic VM

世界上第一款商用Java虚拟机，虚拟机内部只提供了解释器

### Exact VM

JDK1.2

准确式内存管理(Exact Memory Management)：虚拟机可以知道内存中某个位置的数据具体是什么类型

### HotSpot VM

JDK1.3

是JDK6, JDK8默认虚拟机

### BEA的JRockit

专注于服务器端应用

JRockit JVM是世界上最快的JVM

### IBM的J9

IBM Techology for Java Virtual Machine简称IT4J，内部代号J9

### KVM和CDC/CLDC Hotspot

目前移动领域地位尴尬，KVM简单、轻量、高度可移植，面向低端可移动设备

### Azul VM

Azul VM和BEA Liquid VM与特定硬件平台绑定、软硬件配合的专有虚拟机

每个Azul VM实例都可以管理至少数十个CPU和数百GB内存的硬件资源，提供在巨大内存内实现可控的GC时间的垃圾收集器、专有硬件优化的线程调度等

### BEA Liquid VM

BEA开发用于自家Hypervisor系统，不需要操作系统的支持，它本身实现了一个专用操作系统的必要功能

### Apache Harmony

JDK1.5, JDK1.6

由IBM于Intel联合开发

### Microsoft JVM

### Taobao JVM

基于OpenJDK HotSpot VM深度定制且开源的高性能服务器

### Dalvik VM

是虚拟机但不是Java虚拟机，不能直接执行Java的class文件，美原油遵循Java虚拟机规范

基于寄存器架构，不是JVM的栈架构，执行dex(Dalvik Executable)文件，可由class文件转化来

### Graal VM

Graal VM在HotSpot VM基础上增强而成的跨语言全栈虚拟机，可以作为任何语言的运行平台使用(包括：Java, Scala, Groovy, Kotlin; C, C++, JavaScript, Ruby, Python, R等)

支持不同语言中混用对方的接口和对象，支持这些语言使用已经编写好的本地库文件。工作原理是将这些语言的源码或源码编译后的中间格式通过解释器转换成能被Graal VM接受的中间表示。

还提供Truffle工具集快速构建面向一种新语言的解释器。