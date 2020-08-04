---
title: JVM
date: 2020-07-29 15:22:36
categories:
    - 后端
tags: 
    - 面试
    - JVM
---

# 基础概念

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

# 内存与垃圾回收

## 内存结构概述

<img src="./内存结构概述.png">

<img src="./内存结构.png">


类加载子系统负责从文件系统或网络中加载class文件，class文件在文件开头有特定的文件标识

ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定

加载的类信息存放在叫做方法区的内存空间。除了类信息外，方法区还存放运行时常量池信息，可能还有字符串和数字常量

class file加载到JVM中被称为DNA元数据模板，放在方法区

### 字节码文件.class

使用二进制查看器打开任意字节码文件可以观察到期开头4个字节用十六进制表示为`CA FE BA BE`(可记为咖啡宝贝)，这是字节码文件固定头部

### 1. 加载 Loading

1. 通过一个类的全限定名获取定义此类的二进制字节流
2. 将这个字节流所代表的今天存储结构转化为方法区的运行时数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

可以从哪些地方加载.class文件呢：
- 本地文件加载
- 网络获取，典型场景：web Applet
- zip压缩包中读取
- 运行时计算生成，使用最多的是：动态代理技术
- 其它文件生成，例如：JSP应用
- 专有数据库提取.class文件，少见
- 加密文件中获取，可防止class文件被反编译

### 2. 链接 Linking

该阶段分为以下几步：
1. 验证(Verify)
  - 目的在于确保class文件的字节流中包含信息符合当前虚拟机的要求，保证被加载类的正确性，不会危害虚拟机自身安全
  - 主要包括四种验证：
    1. 文件格式验证
    2. 元数据验证
    3. 字节码验证
    4. 符号引用验证
2. 准备(Prepare)
  - 为变量分配内存并且设置该类变量的默认初始值，即零值
  - 这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显示初始化
  - 这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量会随着对象一起分配到java堆中
3. 解析(Resolve)
  - 将常量池内的符号引用转换为直接引用的过程
  - 解析操作往往伴随JVM在执行完初始化之后再执行
  - 符号引用就是一组符号来描述所引用的目标。
  - 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。

### 3. 初始化 Initialization

- 初始化阶段就是执行类构造器方法`<clinit>()`的过程
- 此方法不需定义，是由javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来
- 构造器方法中指令按语句在源文件中出现的顺序执行
- `<clinit>()`不同于类的构造器`<init>()`，若一个类中没有静态变量和静态代码块则字节码文件中无`<clinit>()`方法
- 虚拟机必须保证一个类的`<clinit>()`方法在多线程下被同步加锁

```java
package com.sicmatr1x.java;

public class ClinitTest {
    private int a = 1;
    private static int c = 3;

    public static void main(String[] args) {
        int b = 2;
    }

    public ClinitTest() {
        a = 10;
        int d = 20;
    }
}
```

反编译字节码文件可以看到存在`<clinit>()`和`<init>()`

`<init>`如下：

```
 0 aload_0
 1 invokespecial #1 <java/lang/Object.<init>>
 4 aload_0
 5 iconst_1
 6 putfield #2 <com/sicmatr1x/java/ClinitTest.a>
 9 aload_0
10 bipush 10
12 putfield #2 <com/sicmatr1x/java/ClinitTest.a>
15 bipush 20
17 istore_1
18 return
```

可以看到先给a赋值10再给d赋值20

## 类的加载过程

### 类加载器的分类

JVM支持两种类型的类加载器：
1. 引导类加载器(Bootstrap ClassLoader)
2. 自定义类加载器(User-Defined ClassLoader)：所有派生于抽象类ClassLoader的类加载器都可划分为自定义类加载器

启动类加载器(引导类加载器, Bootstrap ClassLoader)：
- 使用C/C++语言实现，嵌套在JVM内部
- 用于加载Java的核心库(JAVA_HOME/jre/lib/rt.jar, resources.jar或sun.boot.class.path路径下的内容)
- 不继承于java.lang.ClassLoader，没有父加载器
- 加载扩展类和应用程序类加载器，并制定为他们的父类加载器
- 处于安全考虑，Bootstrap启动类加载器只加载包名为java, javax, sun等开头的类

扩展类加载器(Extension ClassLoader)
- Java语言编写，由sun.misc.Launcher$ExtClassLoader实现
- 派生于ClassLoader类
- 父类加载器为启动类加载器
- 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录下加载类库。如果用户创建的JAR放在此目录，也会自动由扩展类加载器加载。

应用程序类加载器(系统类加载器, AppClassLoader)
- Java语言编写，由sun.misc.Launcher$AppClassLoader实现
- 派生于ClassLoader类
- 父类加载器为扩展类加载器
- 负责加载环境变量classpath或系统属性java.class.path指定路径下的类库
- 该类加载器是程序中默认的类加载器，通常Java应用的类都由它来加载
- 通过`ClassLoader#getSystemClassLoader()`方法可以获取到该类加载器

获取Bootstrap ClassLoader类加载器和Extension ClassLoader类加载器可以加载的路径和jar包

```java
package com.sicmatr1x.java;

import sun.misc.Launcher;
import sun.misc.URLClassPath;

import java.net.URL;

public class ClassLoaderTest1 {
    public static void main(String[] args) {
        System.out.println("引导类加载器：");
        URLClassPath urlClassPath = Launcher.getBootstrapClassPath();
        URL[] urls = urlClassPath.getURLs();
        for (URL url : urls) {
            System.out.println(url.toExternalForm());
        }
        System.out.println("扩展类加载器：");
        String extDirs = System.getProperty("java.ext.dirs");
        for (String dir : extDirs.split(";")) {
            System.out.println(dir);
        }
    }
}

```

```
引导类加载器：
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/resources.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/rt.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/sunrsasign.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/jsse.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/jce.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/charsets.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/lib/jfr.jar
file:/C:/Program%20Files/Java/jdk1.8.0_231/jre/classes
扩展类加载器：
C:\Program Files\Java\jdk1.8.0_231\jre\lib\ext
C:\Windows\Sun\Java\lib\ext
```

用户自定义类加载器：
- 什么时候需要用户自定义类加载器：
  - 隔离加载类：例如：确保应用中引用的jar包与中间件引用的第三方jar包不冲突
  - 修改类加载方式：例如：需要时候动态加载
  - 扩展加载源：例如：从数据库中加载
  - 防止源码泄露
- 用户自定义类加载器实现步骤：
  1. 开发人员可以通过继承抽象类java.lang.ClassLoader类的方式实现自己的类加载器
  2. 在JDK1.2之前，自定义类加载器需要继承ClassLoader类并重写loadClass()方法，从而实现自定义的类加载器。JDK1.2之后不建议用户覆盖loadClass()方法，建议把自定义的类加载逻辑写在findClass()方法中
  3. 编写自定义类加载器时若无过于复杂的需求建议直接继承URLClassLoader类，这样可以避免自己去编写findClass()方法及获取字节码流的方式，是自定义类加载器编写更加简洁

```java
package com.sicmatr1x.java;

import java.io.FileNotFoundException;

public class CustomClassLoader extends ClassLoader{
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {

        try {
            byte[] result = getClassFromCustomPath(name);
            if(result == null){
                throw new FileNotFoundException();
            }else{
                return defineClass(name,result,0,result.length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

        throw new ClassNotFoundException(name);
    }

    private byte[] getClassFromCustomPath(String name){
        //从自定义路径中加载指定类:细节略
        //如果指定路径的字节码文件进行了加密，则需要在此方法中进行解密操作。
        return null;
    }

    public static void main(String[] args) {
        CustomClassLoader customClassLoader = new CustomClassLoader();
        try {
            Class<?> clazz = Class.forName("One",true,customClassLoader);
            Object obj = clazz.newInstance();
            System.out.println(obj.getClass().getClassLoader());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### 关于ClassLoader

常用方法：
- getParent(): 返回该类加载器的超类加载器
- loadClass(String name): 加载名称为name的类，返回结果为java.lang.Class类的实例
- findClass(String name): 查找名称为name的类，返回结果为java.lang.Class类的实例
- findLoadedClass(String name): 查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例
- defineClass(String name, byte[] b, int off, int len): 把字节数组b中的内容转换为一个Java类，返回结果为java.lang.Class类的实例
- resolveClass(Class<?> c): 连接指定的一个Java类

### 双亲委派机制

JVM对class文件采用按需加载的方式，即需要使用到该类时才会把class文件加载到内存生成class对象。而且在加载时JVM采用的是双亲委派机制，即把请求交由父类处理，它是一种任务委派模式。

1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
2. 如果父类加载器还存在其父类加载器，则进一步向上委托，一次递归，请求最终将到达顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，若无法完成，之类加载器才会去加载。

优点：
- 避免类的重复加载：一旦一个类被父加载器加载则不会再被子加载器加载
- 保护程序安全，防止核心API被篡改：比如你自己定义一个java.lang.String就不会被AppClassLoader加载而是Bootstrap ClassLoader加载java的String

沙箱安全机制：
自定义String类在加载的时候回率先使用引导类加载器加载，而引导类加载器会先加载JDK自带的文件rt.jar包中的java\lang\String.class，这样可以保证对java核心源代码的保护

### 其它

JVM中表示两个class对象是否为同一个类存在的两个必要条件：
1. 类的完整类名必须一致，包括包名
2. 加载这个类的ClassLoader(指ClassLoader实例对象)必须相同

### 对类加载器的引用

JVM知道一个类时由启动类加载器加载器加载的还是由用户类加载器加载的。若是由用户类加载器加载的，JVM会将这个类加载器的一个引用作为类型信息的一部分保存在方法区中。当解析一个类到另一个类的时候，JVM需要保证这两个类的类加载器是相同的。

### 类的主动使用与被动使用

主动使用的七种情况：
1. 创建类的实例
2. 访问某个类或接口的静态变量，或对该静态变量赋值
3. 调用类的静态方法
4. 反射
5. 初始化一个类的子类
6. Java虚拟机启动时被标明为启动类的类
7. JDK7开始提供动态语言支持：
  - java.lang,invoke.MethodHandle实例的解析结果
  - REF_getStatic, REF_putStatic, REF_invokeStatic句柄对应的类没有初始化，则初始化

除了主动使用的七种情况外都算被动使用

## 运行时数据区(Runtime Data Area)内部结构

运行时数据区的完整图

<img src="image-20200705112416101.png">

Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁。

- 每个线程：独立包括程序计数器、栈、本地栈。
- 线程间共享：堆、堆外内存（永久代或元空间、代码缓存）

<img src="image-20200705112601211.png">

### 线程

线程是一个程序里的运行单元。JVM允许一个应用有多个线程并行的执行。
在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射。

- 当一个Java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。Java线程执行终止后，本地线程也会回收。

操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会调用Java线程中的run()方法。

#### JVM系统线程

如果你使用console或者是任何一个调试工具，都能看到在后台有许多线程在运行。这些后台线程不包括调用public static void main（String[]）的main线程以及所有这个main线程自己创建的线程。|
这些主要的后台系统线程在Hotspot JVM里主要是以下几个：

- 虚拟机线程：这种线程的操作是需要JVM达到安全点才会出现。这些操作必须在不同的线程中发生的原因是他们都需要JVM达到安全点，这样堆才不会变化。这种线程的执行类型包括"stop-the-world"的垃圾收集，线程栈收集，线程挂起以及偏向锁撤销。
- 周期任务线程：这种线程是时间周期事件的体现（比如中断），他们一般用于周期性操作的调度执行。
- GC线程：这种线程对在JVM里不同种类的垃圾收集行为提供了支持。
- 编译线程：这种线程在运行时会将字节码编译成到本地代码。
- 信号调度线程：这种线程接收信号并发送给JVM，在它内部通过调用适当的方法进行处理。

### 程序计数器

JVM中的程序计数寄存器（Program Counter Register）中，Register的命名源于CPU的寄存器，寄存器存储指令相关的现场信息。CPU只有把数据装载到寄存器才能够运行。这里，并非是广义上所指的物理寄存器，或许将其翻译为PC计数器（或指令计数器）会更加贴切（也称为程序钩子），并且也不容易引起一些不必要的误会。JVM中的PC寄存器是对物理PC寄存器的一种抽象模拟。

作用：PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。由执行引擎读取下一条指令。

在JVM规范中，每个线程都要它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。

任何时间都只有一个方法在执行，程序计数器会存储当前线程正在执行的Java方法的JVM指令地址，若为native方法则为undefined

#### 面试常见问题

1. 使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程的执行地址？

因为CPU需要不停的切换各个线程，这时候切换回来以后就需要知道从哪开始继续执行。JVM的字节码解释器需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。

2. PC寄存器为什么会被设定为线程私有？

我们都知道所谓的多线程在一个特定的时间段内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。

由于CPU时间片轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某个线程中的一条指令。

这样必然导致经常中断或恢复，如何保证分毫无差呢？每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

### 堆与栈

首先栈是运行时的单位，而堆是存储的单位

- 栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。
- 堆解决的是数据存储的问题，即数据怎么放，放哪里

### 虚拟机栈

Java虚拟机栈（Java Virtual Machine Stack），早期也叫Java栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（Stack Frame），对应着一次次的Java方法调用。





# 字节码与类的加载


# 性能监控与调优


# 大厂面试


