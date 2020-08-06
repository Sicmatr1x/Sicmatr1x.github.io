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

- 每个方法执行，伴随着进栈（入栈、压栈）
- 执行结束后的出栈工作

对于栈来说不存在垃圾回收问题（栈存在溢出的情况）

虚拟机栈包括多个栈帧，每个栈帧包括：
- 局部变量表
- 操作数栈
- 动态链接
- 方法返回地址

#### 栈中可能出现的异常

Java 虚拟机规范允许Java栈的大小是动态的或者是固定不变的。

如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个`StackOverFlowError` 异常。

如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那Java虚拟机将会抛出一个`OutOfMemoryError` 异常。

#### 设置栈内存大小

我们可以使用参数 -Xss选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度，可以设置到idea的VM Options里面

```java
-Xss1m
-Xss1k
```

### 栈的存储单位

每个线程都有自己的栈，栈中的数据都是以栈帧（Stack Frame）的格式存在。

在这个线程上正在执行的每个方法都各自对应一个栈帧（Stack Frame）。

栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

在一条活动线程中，一个时间点上，只会有一个活动的栈帧。即只用当前正在执行方法的栈帧(栈顶栈帧)是有效的，也被称为当前栈帧(Current Frame)，与当前栈帧相对应的方法就是当前方法（Current Method），定义这个方法的类就是当前类（Current Class）

执行引擎运行的所有字节码指令只针对当前栈帧进行操作。

如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前帧。

#### 栈运行原理

不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧之中引用另外一个线程的栈帧。

如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。

Java方法有两种返回函数的方式，一种是正常的函数返回，使用return指令；另外一种是抛出异常。不管使用哪种方式，都会导致栈帧被弹出。

#### 栈帧的内部结构

每个栈帧中存储着：

- 局部变量表（Local Variables）
- 操作数栈（operand Stack）（或表达式栈）
- 动态链接（DynamicLinking）（或指向运行时常量池的方法引用）
- 方法返回地址（Return Address）（或方法正常退出或者异常退出的定义）
- 一些附加信息

<img src="image-20200705204836977.png">

### 局部变量表(Local Variables)

局部变量表：Local Variables，被称之为局部变量数组或本地变量表

定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量这些数据类型包括各类基本数据类型、对象引用（reference），以及returnAddress类型。

由于局部变量表是建立在线程的栈上，是线程的私有数据，因此不存在数据安全问题

局部变量表所需的容量大小是在编译期确定下来的，并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的。

方法嵌套调用的次数由栈的大小决定。一般来说，栈越大，方法嵌套调用次数越多。对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。

局部变量表中的变量只在当前方法调用中有效。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁。

#### 关于Slot的理解

参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束。

局部变量表，最基本的存储单元是Slot（变量槽）局部变量表中存放编译期可知的各种基本数据类型（8种），引用类型（reference），returnAddress类型的变量。

在局部变量表里，32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（1ong和double）占用两个slot。

>byte、short、char 在存储前被转换为int，boolean也被转换为int，0表示false，非0表示true。
>1ong和double则占据两个slot。

JVM会为局部变量表中的每一个Slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值

当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会按照顺序被复制到局部变量表中的每一个slot上

如果需要访问局部变量表中一个64bit的局部变量值时，只需要使用前一个索引即可。（比如：访问1ong或doub1e类型变量）

如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this将会存放在index为0的s1ot处，其余的参数按照参数表顺序继续排列。

<img src="image-20200705212454445.png">

栈帧中的局部变量表中的槽位是可以重用的，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变就很有可能会复用过期局部变量的槽位，从而达到节省资源的目的。

#### 静态变量与局部变量的对比

变量的分类：

- 按数据类型分：基本数据类型、引用数据类型
- 按类中声明的位置分：成员变量（类变量，实例变量）、局部变量
  - 类变量：linking的paper阶段，给类变量默认赋值，init阶段给类变量显示赋值即静态代码块
  - 实例变量：随着对象创建，会在堆空间中分配实例变量空间，并进行默认赋值
  - 局部变量：在使用前必须进行显式赋值，不然编译不通过。

我们知道类变量表有两次初始化的机会，第一次是在“准备阶段”，执行系统初始化，对类变量设置零值，另一次则是在“初始化”阶段，赋予程序员在代码中定义的初始值。

和类变量初始化不同的是，局部变量表不存在系统初始化的过程，这意味着一旦定义了局部变量则必须人为的初始化，否则无法使用。

在栈帧中，与性能调优关系最为密切的部分就是前面提到的局部变量表。在方法执行时，虚拟机使用局部变量表完成方法的传递。

局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收。

### 操作数栈

操作数栈：Operand Stack

每一个独立的栈帧除了包含局部变量表以外，还包含一个后进先出（Last - In - First -Out）的 **操作数栈**，也可以称之为 **表达式栈**（Expression Stack）

操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈（push）和 出栈（pop）

- 某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈。使用它们后再把结果压入栈
- 比如：执行复制、交换、求和等操作

<img src="image-20200706090618332.png">

操作数栈，主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间。

操作数栈就是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是空的。

#### 反编译字节码解读示例

```java
package com.sicmatr1x.java;

public class OperandStackTest {
    public void testAddOperation() {
        byte i = 15;
        int j = 8;
        int k = i + j;
    }
}
```

```
  Last modified 2020-8-5; size 459 bytes
  MD5 checksum 0c86be65af756c867db277302bedf1fa
  Compiled from "OperandStackTest.java"
public class com.sicmatr1x.java.OperandStackTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#19         // java/lang/Object."<init>":()V
   #2 = Class              #20            // com/sicmatr1x/java/OperandStackTest
   #3 = Class              #21            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lcom/sicmatr1x/java/OperandStackTest;
  #11 = Utf8               testAddOperation
  #12 = Utf8               i
  #13 = Utf8               B
  #14 = Utf8               j
  #15 = Utf8               I
  #16 = Utf8               k
  #17 = Utf8               SourceFile
  #18 = Utf8               OperandStackTest.java
  #19 = NameAndType        #4:#5          // "<init>":()V
  #20 = Utf8               com/sicmatr1x/java/OperandStackTest
  #21 = Utf8               java/lang/Object
{
  public com.sicmatr1x.java.OperandStackTest();
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
            0       5     0  this   Lcom/sicmatr1x/java/OperandStackTest;

  public void testAddOperation();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        15
         2: istore_1
         3: bipush        8
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 8: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/sicmatr1x/java/OperandStackTest;
            3       8     1     i   B
            6       5     2     j   I
           10       1     3     k   I
}
SourceFile: "OperandStackTest.java"
```

常量池：

```
Constant pool:
   #1 = Methodref          #3.#19         // java/lang/Object."<init>":()V
   #2 = Class              #20            // com/sicmatr1x/java/
```

如果操作指令有用到常量池里的常量会出现`#2`这样的表示调用的是哪个常量

```
    Code:
      stack=1, locals=1, args_size=1
```

这里的`stack=1`表示操作数栈为1，`locals=4`表示本地变量表长度为4

本地变量表：

```
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/sicmatr1x/java/OperandStackTest;
            3       8     1     i   B
            6       5     2     j   I
           10       1     3     k   I
```

这里的start和length指的是该变量的生命周期对应操作指令的地址。这里因为没有double, long所以都是每个变量占用一个slot。byte, short, char, boolean都以int型来保存

分析操作指令：

```
0: bipush        15
2: istore_1
3: bipush        8
5: istore_2
6: iload_1
7: iload_2
8: iadd
9: istore_3
10: return
```

手动执行指令，因为局部变量表所需的容量大小是在编译期确定下来的，所以这里没值的用[]来占位，且省略局部变量表第0号元素`this`：

- `0: bipush        15`
  - PC寄存器: 0
  - 局部变量表: 
    1. []
    2. []
    3. []
  - 操作数栈 : 
    - []
    - [15] <-栈顶
- `2: istore_1`: i指int类型，store存放到局部变量表，_1索引位1的位置
  - PC寄存器: 2
  - 局部变量表: 
    1. [15]
    2. []
    3. []
  - 操作数栈 : 
    - []
    - [] <-栈顶
- `3: bipush        8`
  - PC寄存器: 3
  - 局部变量表: 
    1. [15]
    2. []
    3. []
  - 操作数栈 : 
    - []
    - [8] <-栈顶
- `5: istore_2`
  - PC寄存器: 2
  - 局部变量表: 
    1. [15]
    2. [8]
    3. []
  - 操作数栈 : 
    - []
    - [] <-栈顶
- `6: iload_1`: 从局部比那里表中取索引位1的数据到操作数栈
  - PC寄存器: 6
  - 局部变量表: 
    1. [15]
    2. [8]
    3. []
  - 操作数栈 : 
    - []
    - [15] <-栈顶
- `7: iload_2`: 从局部比那里表中取索引位2的数据到操作数栈
  - PC寄存器: 7
  - 局部变量表: 
    1. [15]
    2. [8]
    3. []
  - 操作数栈 : 
    - [8] <-栈顶
    - [15]
- `8: iadd`: 操作数栈出栈并做加法运算
  - PC寄存器: 8
  - 局部变量表: 
    1. [15]
    2. [8]
    3. []
  - 操作数栈 : 
    - []
    - [23] <-栈顶
- `9: istore_3`
  - PC寄存器: 9
  - 局部变量表: 
    1. [15]
    2. [8]
    3. [23]
  - 操作数栈 : 
    - []
    - [] <-栈顶


如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中，并更新PC寄存器中下一条需要执行的字节码指令。

操作数栈中元素的数据类型必须与字节码指令的序列严格匹配，这由编译器在编译器期间进行验证，同时在类加载过程中的类检验阶段的数据流分析阶段要再次验证。|

另外，我们说Java虚拟机的解释引擎是基于栈的执行引擎，其中的栈指的就是操作数栈。

最后PC寄存器的位置指向10，也就是return方法，则直接退出方法

i++和++i的区别

### 栈顶缓存技术

栈顶缓存技术：Top Of Stack Cashing

前面提过，基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数。

由于操作数是存储在内存中的，因此频繁地执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM的设计者们提出了栈顶缓存（Tos，Top-of-Stack Cashing）技术，将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率。

> 寄存器：指令更少，执行速度快

### 动态链接(Dynamic Linking)

<img src="image-20200706100311886.png">

> 动态链接、方法返回地址、附加信息 ： 有些地方被称为帧数据区

每一个栈帧内部都包含一个指向**运行时常量池**中该栈帧所属方法的引用包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接（Dynamic Linking）。比如：invokedynamic指令

在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用（symbolic Reference）保存在class文件的常量池里。

比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用。

<img src="image-20200706101251847.png">

> 为什么需要运行时常量池？
>
> 因为在不同的方法，都可能调用常量或者方法，所以只需要存储一份即可，节省了空间
>
> 常量池的作用：就是为了提供一些符号和常量，便于指令的识别

### 方法调用：解析与分配

在JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制相关

#### 静态链接

当一个字节码文件被装载进JVM内部时，如果被调用的目标方法在编译期可知，且运行期保持不变时，这种情况下降调用方法的符号引用转换为直接引用的过程称之为静态链接

#### 动态链接

如果被调用的方法在编译期无法被确定下来，也就是说，只能够在程序运行期将调用的方法的符号转换为直接引用，由于这种引用转换过程具备动态性，因此也被称之为动态链接。

### 绑定机制

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次。

早期绑定：

早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。

晚期绑定：

如果被调用的方法在编译期无法被确定下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式也就被称之为晚期绑定。

Java中任何一个普通的方法其实都具备虚函数的特征，它们相当于C++语言中的虚函数（C++中则需要使用关键字virtual来显式定义）。如果在Java程序中不希望某个方法拥有虚函数的特征时，则可以使用关键字final来标记这个方法。

#### 虚方法和非虚方法

- 如果方法在编译期就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为非虚方法。
- 静态方法、私有方法、fina1方法、实例构造器、父类方法都是非虚方法。
- 其他方法称为虚方法。

> 子类对象的多态的使用前提
>
> - 类的继承关系
> - 方法的重写

虚拟机中提供了以下几条方法调用指令：

普通调用指令：
- invokestatic：调用静态方法，解析阶段确定唯一方法版本
- invokespecial：调用`<init>`方法、私有及父类方法，解析阶段确定唯一方法版本
- invokevirtual：调用所有虚方法
- invokeinterface：调用接口方法

动态调用指令：
- invokedynamic：动态解析出需要调用的方法，然后执行

前四条指令固化在虚拟机内部，方法的调用执行不可人为干预，而invokedynamic指令则支持由用户确定方法版本。其中invokestatic指令和invokespecial指令调用的方法称为非虚方法，其余的（fina1修饰的除外）称为虚方法。

#### invokednamic指令

JVM字节码指令集一直比较稳定，一直到Java7中才增加了一个invokedynamic指令，这是Java为了实现动态类型语言】支持而做的一种改进。

但是在Java7中并没有提供直接生成invokedynamic指令的方法，需要借助ASM这种底层字节码工具来产生invokedynamic指令。直到Java8的Lambda表达式的出现，invokedynamic指令的生成，在Java中才有了直接的生成方式。

Java7中增加的动态语言类型支持的本质是对Java虚拟机规范的修改，而不是对Java语言规则的修改，这一块相对来讲比较复杂，增加了虚拟机中的方法调用，最直接的受益者就是运行在Java平台的动态语言的编译器。

#### 动态类型语言和静态类型语言

动态类型语言和静态类型语言两者的区别就在于对类型的检查是在编译期还是在运行期，满足前者就是静态类型语言，反之是动态类型语言。

说的再直白一点就是，静态类型语言是判断变量自身的类型信息；动态类型语言是判断变量值的类型信息，变量没有类型信息，变量值才有类型信息，这是动态语言的一个重要特征。

> Java：String info = "mogu blog";     (Java是静态类型语言的，会先编译就进行类型检查)
>
> JS：var name = "shkstart";    var name = 10;    （运行时才进行检查）

#### 方法重写的本质

注意：在调用对象方法前会将这个对象的引用压入操作数栈顶，因为后面需要需到其实际类型

Java 语言中方法重写的本质：
- 找到操作数栈顶的第一个元素所执行的对象的实际类型，记作C。
- 如果在类型C中找到与常量中的描述符合简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果不通过，则返回java.1ang.I1legalAccessError 异常。
- 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程。
- 如果始终没有找到合适的方法，则抛出java.1ang.AbstractMethodsrror异常。

IllegalAccessError介绍：
程序试图访问或修改一个属性或调用一个方法，这个属性或方法，你没有权限访问。一般的，这个会引起编译器异常。这个错误如果发生在运行时，就说明一个类发生了不兼容的改变

#### 方法的调用：虚方法表

在面向对象的编程中，会很频繁的使用到动态分派，如果在每次动态分派的过程中都要重新在类的方法元数据中搜索合适的目标的话就可能影响到执行效率。因此，为了提高性能，JVM采用在类的方法区建立一个虚方法表
（virtual method table）（非虚方法不会出现在表中）来实现。使用索引表来代替查找。

每个类中都有一个虚方法表，表中存放着各个方法的实际入口。

虚方法表是什么时候被创建的呢？

虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始值准备完成之后，JVM会把该类的方法表也初始化完毕。




# 字节码与类的加载


# 性能监控与调优


# 大厂面试


