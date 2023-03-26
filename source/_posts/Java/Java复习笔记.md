---
title: Java复习笔记
date: 2017-10-10 09:54:47
categories:
    - Back-end
tags: 
    - Java
---


## 索引

1. <a href="#Java_env"> Java运行环境 </a>
1. <a href="#Java_basic"> Java语言 </a>
1. <a href="#Java_grammer"> Java语法 </a>
1. <a href="#Java_array"> Java数组 </a>
1. <a href="#Java_collections"> 集合框架 </a>
1. <a href="#Java_io"> Java I/O 输入/输出流 </a>
1. <a href="#Java_reflect"> 反射 </a>
1. <a href="#Java_network"> Java网络编程 </a>
1. <a href="#Java_web"> Java Web 开发相关技术 </a>
1. <a href="#algorithm_problem"> 算法题 </a>

## <a id="Java_env"> Java运行环境 </a>

### JDK和JRE的作用

JRE(Java Runtime Environment)是Java程序的运行环境。包含JVM在%JRE安装目录%/bin/client/jvm.dll，所有的Java类库的class文件，在lib目录下打包成jar包。</br>
JKD(Java Development Kit)是Java开发工具包。在%JDK安装目录%/bin/client/jvm.dll与在%JKD安装目录%/bin/server/jvm.dll下含有两个JVM虚拟机。同时只有JDK下才有javac。</br>

### 配置JDK环境

1. 我的电脑->属性->系统属性->高级->环境变量->系统环境变量，新建JAVA_HOME环境变量，值为 %jdk安装目录%，例：C:\Program Files\Java\jkd1.6.1_13
2. 配置path环境变量：在path后加上;%JAVA_HOME%\bin
3. 编译java代码：

```java
public class HelloWorld {
    public static void main(String[] args){
        System.out.println("Hello world!");
    }
}
```

保存为HelloWorld.java，命令行进入当前目录使用命令：javac HelloWorld.java编译上传HelloWorld.class文件(class文件是字节码文件，字节码需要在JVM虚拟机里面运行)

4. 使用命令java HelloWorld 运行

如果类指定了包名则需用命令javac -d HelloWorld.java来进行编译</br>

### .class文件

.class文件是字节码文件，字节码需要在JVM虚拟机里面运行。对一个.java文件进行编译可以产生一个同名的.class文件。若对一个含有内部类的.java文件进行编译会产生 外部类名$内部类名.class的文件。</br>

### 类的加载机制

Java提供两种类的装载方式：
1. 预先装载
2. 按需装载(大部分类延迟到使用时才动态加载被称为Java的运行时动态装载机制)

Java的运行时动态装载机制：使得Java可以在动态运行时装载软件部件，修改代码无需全盘编译，为软件系统的开发提供了极大的灵活性。</br>

1. 预先装载

当启动一个程序时：Java首先在JDK目录下找到并载入jvm.dll->启动虚拟机->虚拟机进行初始化操作(设置操作系统参数等)->创建Bootstrap Loader对象(启动类装载器，由C++编写)->Bootstrap Loader一次性加载JVM的所有基础类</br>

Bootstrap Loader还会装载定义在sun.misc命名空间下的Launcher类，Launcher类拥有两个内部类：ExtClassLoader, AppClassLoader. 其继承关系如下：
Bootstrap Loader<-ExtClassLoader<-AppClassLoader(拥有main()函数的入口类)

2. 按需装载

装载条件：</br>
1)静态方法</br>
2)静态属性</br>
3)构造方法</br>
除外：</br>
1)当访问静态常量属性时，JVM加载类的过程中不会进行类的初始化工作，只会进行到解析阶段</br>
2)构造方法没有被显示的声明为静态方法，但它仍作为内地 静态成员特例</br>

按需装载流程：</br>
当需要使用一个类时JVM会检查这个类的Class对象是否已经加载，若未加载则开始装载：
1. 加载：查找并导入类的二进制字节码文件，根据这些字节码文件创建一个Class对象
1. 链接：分为校验、准备、解析
1. 校验：检查导入的二进制字节码的完整性、正确性、安全性
1. 准备：为静态域分配存储空间
1. 解析：将符号引用转折为直接引用
1. 初始化：初始化静态变量并执行静态域代码

类加载器：JVM使用类加载器来加载类，Java加载器在Java核心类库和CLASSPATH环境下面的所有类中查找类，若找不到则会抛出java.lang.ClassNotFoundException异常</br>

从J2SE1.2开始：JVM使用了3中加载器：bootstrap、extension、system类加载器，依次为父子关系。</br>

### 环境变量CLASSPATH的作用

环境变量CLASSPATH是在编译Java源码和运行程序时使用的，用于为Java程序指定所依赖的接口、类等搜索路径。</br>

```
.;c:\jar\log4j.jar;d:\work\java
```

### 如何为Java程序动态的指定类搜索路径

使用-cp选项，此时JVM会把指定的jar文件作为CLASSPATH的一部分</br>

```
javac -cp D:\work\java\log4j.jar HelloWorld.java
```

### 如何使用cmd把Java程序打包成jar文件

jar包一般包含class文件、配置文件和清单文件manifest.mf</br>

格式如下：</br>

```
jar {c t x u f}[v m e 0 M i][-C 目录]文件名
{c t x u f}四个参数必须选其一
[v m e 0 m i]为可选参数
-c 创建一个jar包
-t 显示jar包中内容列表
-x 解压jar包
-u 添加文件到jar包
-f 指定jar包的文件名

-v 生成详细报告并输出到标准设备
-m 指定manifest.mf文件
-0 产生jar包时不对内容进行压缩
-M 不产生所有文件的清单文件manifest.mf
-i 为指定的jar文件创建索引文件

-C 表示转到相应的目录下指定jar命令
```

```
生成hello.jar包
jar cf hello.jar HelloWorld.class
显示打包过程
jar vcf hello.jar HelloWorld.class
```

### Java Web项目生成(Build)、部署(Deploy)、配置(Configuration)

目录结构：</br>

```
javaweb
    |- META-INF
    |- resource
    |- WEB-INF
        |- classes
        |- lib
```

web.xml是整个Web应用程序的配置文件，通过它来定义Servlet、过滤器、监听器等，Web容器通过该文件的配置来控制整个Web应用程序的行为方式。须放在WEB-INF目录下</br>

Servlet是服务器端处理HTTP请求的基本组成单元。JSP、过滤器都由其实现。Servlet存活在Web容器中，由Web容器来控制其生命周期。</br>
JSP的脚本语言是Java，其本质是Servlet。</br>

打包出来的文件为.war后缀，Java Web容器是符合Java EE规范的，所以每个Java Web应用程序都可以部署到任何平台的任何Java EE容器中。</br>

---

## <a id="Java_basic"> Java语言 </a>

### Java与C++程序的区别

C、C++：由编译器把源码直接编译成计算机可识别的机器码(exe、dll等)，再直接运行。</br>
Java：由javac命令把源文件编译成class文件，在Java程序启动时先启动Java虚拟机再由虚拟机去加载class文件。</br>

### 简述JCM及其工作原理

JVM是一种用软件模拟出来的计算机，它用于执行Java程序，有一套非常严格的技术规范，是Java程序实现跨平台特性的基础。Java虚拟机有虚拟出来的计算机硬件如：处理器、寄存器、堆栈等，还具有与之配套的指令系统。它运行Java程序就行普通计算机运行C、C++程序一样。</br>

### Java程序为什么无需delete语句进行内存回收(JVM的垃圾回收机制)

JVM把程序创建的对象存放在堆空间中</br>

堆(Heap)：是一个运行时的数据存储区。分配和释放由程序中显示分配，没有垃圾自动回收机制，且须由程序代码显示释放这些实体。类似于C中的malloc()和free()。JVM会把程序创建的对象放在堆中，在Java中则由JVM自动释放(一般是垃圾回收器检测出一个对象不再被引用就就行回收)。</br>
栈(Stack)：一般存放非static的自动变量、函数参数、表达式的临时结果和函数返回值。分配和释放均由系统自动完成。</br>

---

## <a id="Java_grammer"> Java语法 </a>

### 变量及其作用域

全局变量：可以被所有函数在任何地址使用的变量</br>
局部变量：在某一特定的代码范围才能看见的变量</br>

根据生存周期来分：</br>

1. 静态变量：类中由static修饰的变量，当类加载时就生成并初始化</br>
2. 成员变量：类中没有使用static修饰的变量，当对象加载时就生成并初始化，随着垃圾回收器回收而消失</br>
3. 局部变量：定义在方法中的变量或方法的参数或定义在代码块中(用大括号包括的)的变量</br>

### Java变量数据类型

分为：</br>

基本数据类型和引用数据类型</br>

### Java包含哪些基本数据类型及其包装类

基本数据类型：byte, short, int, long, float, double, boolean, char</br>
包装类：Byte, Short, Integer, Long, Float, Double, Boolean, Character</br>

int取值范围：int长度为4字节，共4*8=32位，第一位为符号位，最大值为2^31-1，最小值为-2^31</br>

```java
int otc = 0123; // 八进制
int hex = 0x123; // 十六进制
```

long取值范围：长度为8字节，共8*8=64位，[-2^63, 2^63-1]</br>

float取值范围：长度为4字节，共4*8=32位，[3.4E+10^-38, 3.4E+10^38]</br>

double取值范围：长度为8字节，共8*8=64位，[1.7E+10^-308, 1.7E+10^308]</br>

类型转换：分为显示转换和隐式转换</br>

boolean存于栈空间，Boolean对象存放在堆空间中</br>

char采用Unicode编码，用2字节表示一个字符，char长度为2字节，16位，[0, 2^16-1]</br>

JVM启动时会实例化9个对象池，分别用来存储8种基本类型和String对象：对象池的作用是为了避免频繁的创建和销毁对象影响系统性能</br>

StringBuffer线程安全，StringBuilder线程不安全。</br>

使用指定的字符集创建String对象：String str = new String("中午".getBytes(), "GBK")，可用"GBK", "UTF-8", "ISO-8859-1"</br>


### 装箱与拆箱

Java5.0提供的功能，用于打包基本数据类型，同时隐藏一些细节。自动装箱与拆箱是在编译阶段进行的。</br>

### 转义字符

```
\a:响铃
\b:退格BS
\f:换页FF
\n:换行LF
\r:回车CR
\t:水平制表HT
\v:垂直制表VT
\\:反斜杠
\?:问号字符
\':单引号字符
\":双引号字符
\0:空字符NULL
\ddd:任意字符 三位八进制
\xhh:任意字符 二位十六进制
```

### Java的引用与C++的指针的区别

相同：都是指向一块内存地址的，通过引用指针来完成对内存数据的操作。</br>
区别：</br>
1. 类型：引用的值为地址的数据元素，Java封装了的地址，可以转成字符查看，不必关心长度。C++指针是一个装地址的变量。</br>
2. 所占内存：引用声明没有实体，不占空间，C++指针用到才会赋值</br>
3. 初始值：java初始值为null，C++为原内存里所保存的值</br>
5. 计算：引用不可计算，C++相当于int可计算</br>
6. 控制：引用不可控制，C++可以使用计算来控制指针指向</br>
7. 内存泄漏：java不会，C++容易产生内存泄漏</br>

### Java中的main()方法

```java
public static void main(String[] args){

}
```

作为程序的入口函数，可以通过args接受外部参数。</br>

### equal与==

==为直接比较值，若为基本数据类型则比较是否相同，若为引用则比较引用是否指向同一个对象。</br>

equal则是调用java.lang.Object里的equal()方法或对象里面重写的equal方法来比较</br>

### Java中的三元运算符

```java
tmp = a > b ? "a>b" : "a<b";
```

### 注释

```java
// 行注释

/*
块注释
*/

/**
*文档注释
*/
public int test(String arg0){

}
```

### 静态成员的特点

在类中通过static关键字修饰，包括：静态成员变量、静态方法、静态代码块</br>

1. 在类加载的时候就进行创建、初始化或执行代码
2. 一个类只有一个
3. 类的所有实例都可以访问

### 子类构造方法调用父类的构造方法

使用super()方法，且super()方法必须放在子类构造方法的第一行，若super()无参数则可省略</br>

### 接口和抽象类的区别

抽象类是功能不全的类，里面可以有非抽象方法</br>
接口是抽象方法声明和静态不能被修改数据的集合</br>
两者都不能被实例化</br>
一个类一次只能继承一个抽象类但可以实现多个接口

### 内部类

```java
package abc;
class A{
    class B{

    }
}
```

B类的全类名是abc.A.B，且B会依赖于A。</br>

下面分类讨论：</br>

根据定义结构分类：</br>
1. 成员式：定义的方法与成员变量相似
2. 局部式：定义在方法体重

成员内部类：</br>
1. 静态内部类：使用static关键字修饰的内部类，当加载外部类的时候也会加载静态内部类。无法访问外部类的非静态成员。全类名：abc.A.B，class文件名：A$B.class

```java
package abc;
class A{
    static class B{

    }
}
```

2. 成员内部类：需要等外部类创建对象以后才会被加载到JVM中，属于外部类的某个实例，可访问外部类的静态与非静态成员。

```java
package abc;
class A{
    class B{

    }
}
```

创建成员内部类语法：</br>

```java
public static void main(String[] args){
    A a = new A();
    A.B b = a.new B();
}
```

局部式内部类：</br>
1. 普通局部内部类：位于方法中
2. 匿名内部类：没有类名，匿名内部类的class文件命名方法按照匿名内部类的排列顺序来进行：Outter$1.class

### 可见性private, protected, public, default

public:可被所有其它类访问</br>
private:自身所在类内可见</br>
protected:自身，子类及同一个包中类可访问</br>
default:自身，同一个包中类可访问</br>

## <a id="Java_array"> Java数组 </a>

### Java数组的本质

Java数组的本质是一个特殊的类，该类好保存了数据类型的信息。该类通过成员变量的形式保存数据，并通过[]符号来访问数据。基本数据类型的数组保存的是值(初始化为0)，而应用类型的数组保存的是对象的引用(初始化为null)。</br>

### 拷贝数组的数据

通过for遍历来赋值只是复制了对象的引用，若需要复制对象则可用：</br>

```java
int[] arr = new int[][1,2,3];
int[] arr2 = new int[3];
System.arraycopy(arr, 0, arr, 0, arr.length);
```

--

## <a id="Java_collections"> 集合框架 </a>

<img src="JAVA集合框架.PNG" /></br>

列表List：有序，允许重复</br>
集合Set：无序，不允许重复</br>
SortedSet：有序的Set</br>
映射Map：无序，不允许重复，键值对</br>
SortedMap：有序的Map</br>

### 迭代器

迭代器(Iterator)模式，又叫游标(Cursor)模式。提供一种方法来访问一个容器对象中的各个元素。</br>

### 比较器

用于比较元素，需要实现Comparable或Comparator接口。</br>

1. Comparable接口：进行比较类需要实现的接口，仅包含一个compareTo()方法，返回值大于0时表示本对象<参数对象

```java
public class User implements Comparable{
    public int age;
    public int compareTo(Object o){
        return this.age-(User o).age;
    }
}
```

2. Comparator接口：实现该接口的类被称为比较器，包含compare()方法。</br>

```java
public class User{
    public int age;

    public User(int age){
        this.age = age;
    }

    public static void main(String[] args){
        User u1 = new User(16);
        User u2 = new User(18);
        Comparator comp = new UserComparator();
        int result = comp.compare(u1, u2);
        System.out.println(result);
    }
}

public class UserComparator implements Comparator{
    public int compare(Object arg0, Object arg1){
        User u1 = (User) arg0;
        User u2 = (User) arg1;
        return u1.age - u2.age;
    }
}

```

### Vector 与 ArrayList 的区别

Vector是线程安全的，它操作元素的方法都是同步方法。ArrayList不是，但效率更高。</br>

### HashMap 与 HashTable 的区别

HashTable的方法是同步的，HashMap不同步</br>
HashTable不允许null，HashMap允许null</br>
HashTable使用Enumeration遍历，HashMap使用Iterator遍历</br>
HashTable直接使用对象的hashCode，HashMap会重新计算</br>

### 集合使用泛型

可以明确集合里存储的元素的类型，避免了手动类型转换的过程</br>

### 集合元素排序

使用java.util.Collections类中的sort()方法对List元素进行排序</br>
如果类中的元素全部实现了Comparable接口则可通过Collections.sort()排序</br>

```java
//REVIEW:test this code
public class User implements Comparable<User>{
    public int age;

    public User(int age){
        this.age = age;
    }

    public int compareTo(Object o){
        return this.age-(User o).age;
    }
}

public class Test{
    public static void main(String[] args){
        List<User> list = new ArrayList<User>();
        list.add(new User(16));
        list.add(new User(18));
        list.add(new User(22));
        // 默认排序
        Collections.sort(list);
        // 降序排序
        // 若没有实现Comparable接口，也可以提供比较器
        Comparator comp = Collections.reverseOrder();
        Collections.sort(list, comp);

    }
}
```

若没有实现Comparable接口，也可以提供比较器来进行排序</br>

### 什么集合可以使用 foreach

foreach运行步骤如下：</br>
1. 调用指定集合对象的Iterator()方法，得到迭代器
2. 使用迭代器的hasNext()方法判断有无下一个元素进行循环
3. 每次循环都用next()方法得到元素

数组或实现了Iterable接口的类实例，Jav集合框架中的集合大多符合第二条</br>

---

## <a id="Java_io"> Java I/O 输入/输出流 </a>

### 复制文件程序

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class FileCopy{
    public static void main(String[] args){
        // 输入文件流
        FileInputStream fin = new FileInputStream("d:/test/a.txt");
        // 输出文件流
        FileOutputStream fout = new FileOutputStream("d:/test/b.txt");
        byte[] buff = new byte[256]; // 缓冲区
        int len = 0; // 每次读到的数据长度
        while((len = fin.read(buff)) > 0){
            fout.write(buff, 0, len);
        }
        fin.close();
        fout.close();
    }
}
```

如果不关闭流，会造成资源的浪费，还可能会导致文件锁住，其他程序无法操作文件。</br>

### 字节流

字节流处理的是最基本的单位byte，它可以处理任何形式的数据，主要操作byte数组。Java中可以使用java.io.FileInputStream和java.io.FileOutputStream来进行字节流的处理。</br>

你也可以使用包装过的具有特定功能的字节流，基本使用思路如下：</br>

1. 获取输入或输出的流对象，从File获得或网络等
2. 根据特定的字符格式创建InputStreamReader或InputStreamWriter
3. 使用read()或readLine()方法读取数据，write()或print()
4. 关闭流

### 序列化

把对象内存中的数据按照规则变成一系列的字节数据并写入到流中。须Serializable，必要时还需提供serialVersionUID</br>

## 多线程

### 线程(Thread)与进程(Process)的区别

进程包含线程，每个应用程序的执行都在操作系统内核中登记一个进程标志，操作系统根据分配的标志对应用程序的执行进行调度和系统资源分配。进程是占用系统资源的基本单位。</br>
进程在执行过程中拥有独立的内存单元，而多个线程共享内存。</br>
进程拥有固定的入口、执行顺序、出口，而线程会被应用程序控制。</br>

### 并发(Concurrent)与并行(Parallel)的区别

<img src="cocurrent_parallel.jpg" />

并发：交替使用一台咖啡机；并行同时使用两台咖啡机。</br>

并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。并行是并发的子集。</br>

在操作系统中，并发是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行，但任一个时刻点上只有一个程序在处理机上运行。</br>


### 让一个类成为线程类

1. 实现Runnable接口
2. 继承Thread类

继承Thread类之后就不能继承其它类了，实现Runnable接口则可以。</br>
继承Thread类更方便。</br>
实现Runnable接口的线程类更方便访问同一个变量，而Thread则需要使用内部类。</br>

继承自Thread类可以通过new创建对象再调用start()方法。</br>
实现Runnable接口的线程类需要将其对象作为Thread构造方法的参数，然后调用Thread对象的start()方法</br>

### 使用sychronized让线程同步

每个对象都可以有一个线程锁，sychronized可以用任何一个对象的线程锁来锁住一段代码，任何想要进入该段代码的线程必须在解锁以后才能继续执行，否则进入等待状态。只有占用锁资源的线程执行完毕后，锁资源才会被释放。</br>

java会为每个object对象分配一个monitor，当某个对象的同步方法（synchronized methods ）被多个线程调用时，该对象的monitor将负责处理这些访问的并发独占要求。</br>
当一个线程调用一个对象的同步方法时，JVM会检查该对象的monitor。如果monitor没有被占用，那么这个线程就得到了monitor的占有 权，可以继续执行该对象的同步方法；如果monitor被其他线程所占用，那么该线程将被挂起，直到monitor被释放。</br>
当线程退出同步方法调用时，该线程会释放monitor，这将允许其他等待的线程获得monitor以使对同步方法的调用执行下去。</br>


### 编写一个生产者消费者模型的多线程例子

每个生产者在添加货物之前检查仓库是否已满，若已满则等待并通知消费者进行消费，直到消费者消费了至少一个货物以后再继续添加；消费者在消费一个货物之前检查仓库是否为空，若为空着等待并通知生产者进行生产，直到生产者添加了至少一个货物后，再进行消费。</br>

sleep()函数是Thread类的静态函数，不涉及到线程间同步概念，仅仅为了让一个线程自身获得一段沉睡时间。sleep可以在任何地方使用。</br>
wait函数是object类的函数，要解决的问题是线程间的同步，该过程包含了同步锁的获取和释放，调用wait方法将会将调用者的线程挂起，直到其他线程调用同一个对象的notify方法才会重新激活调用者。</br>

```java
package review.thread;

public class Store {
	/**
	 * 仓库的最大容量
	 */
	private final int MAX_SIZE;
	/**
	 * 当前的货物数量
	 */
	private int count;
	/**
	 * 初始化最大容量的构造方法
	 * @param n 仓库的最大容量
	 */
	public Store(int n){
		MAX_SIZE = n;
		count=0;
	}
	
	/**
	 * 向仓库添加货物
	 */
	public synchronized void add(){
		while(count >= MAX_SIZE){
			System.out.println("仓库已满");
			try{
				this.wait(); // 进入等待池
			}catch(InterruptedException e){
				e.printStackTrace();
			}
		}
		count++; // 增加库存
		System.out.println(Thread.currentThread().toString() + " add " + count);
		this.notifyAll(); // 通知所有消费者线程来拿，同时唤醒所有挂起的生产者
	}
	
	public synchronized void remove(){
		while(count <= 0){
			System.out.print("empty");
			try {
				this.wait();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		System.out.println(Thread.currentThread().toString() + " remove " + count);
		count--;
		this.notify(); // 通知生产者添加
	}
	
	public static void main(String[] args){
		Store store = new Store(5);
		
		Thread pro1 = new Proceducer(store);
		Thread pro2 = new Proceducer(store);
		Thread con1 = new Consumer(store);
		Thread con2 = new Consumer(store);
		pro1.setName("producer1");
		pro2.setName("producer2");
		con1.setName("consumer1");
		con2.setName("consumer2");
		
		pro1.start();
		pro2.start();
		con1.start();
		con2.start();
	}
}

package review.thread;

public class Proceducer extends Thread{
	private Store store;
	public Proceducer(Store store){
		this.store = store;
	}
	public void run(){
		while(true){
			this.store.add();
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}

package review.thread;

public class Consumer extends Thread{
	private Store store;
	public Consumer(Store store){
		this.store = store;
	}
	public void run(){
		while(true){
			this.store.remove();
			try {
				Thread.sleep(1500);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}

```

### 如何使用Java线程池

线程池属于对象池，其目的在于最大限度的复用对象。还可以使线程代码与业务代码分离。</br>

```java
java.util.concurrent.ThreadPoolExecutor(
    int corePoolSize, // 最大核心线程数
    int maximumPoolSize, // 允许最大线程数
    long keepAliveTime,
    TimeUnit unit,
    BlokingQueue<Runnable> workQueue, // 缓冲队列
    RejectedExecutionHandler handler
);
```

一个Runnable类型的对象通过execute(Runnable)方法添加到线程池。</br>
当一个任务通过execute(Runnable)方法添加到线程池时：</br>
1. 如果此时线程池中线程数<corePoolSize，即使线程池中的线程都处于空闲状态也要添加新线程来处理任务
2. 如果此时线程池中线程数=corePoolSize，但缓冲队列workQueue未满，那么任务被放入缓冲队列
3. 如果此时线程池中线程数>corePoolSize，缓冲队列workQueue满，但线程池中线程数<maximumPoolSize，创建新线程来处理任务
4. 如果此时线程池中线程数>corePoolSize，缓冲队列workQueue满，但线程池中线程数=maximumPoolSize，通过handler所指定的策略来处理此任务

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class TestThreadPool {
	private static int produceTaskSleepTime = 2000;
	public static void main(String[] args){
		// 创建线程池
		ThreadPoolExecutor producerPool = new ThreadPoolExecutor(
				1, 1, 0, TimeUnit.SECONDS, 
				new ArrayBlockingQueue(3), 
				new ThreadPoolExecutor.DiscardOldestPolicy());
		
		int i = 1;
		while(true){
			try {
				Thread.sleep(produceTaskSleepTime);
				String task = "task@" + i;
				System.out.println("put " + task);
				// 用execute方法启动任务
				producerPool.execute(new ThreadPoolTask(task));
				i++;
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}

import java.io.Serializable;

public class ThreadPoolTask implements Runnable, Serializable{

	private static final long serialVersionUID = 0;

	private static int consumeTaskSleepTime = 2000;
	
	private String threadPoolTaskData;
	
	public ThreadPoolTask(String tasks){
		this.threadPoolTaskData = tasks;
	}
	
	public void run() {
		System.out.println("start.." + threadPoolTaskData);
		try {
			Thread.sleep(consumeTaskSleepTime);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		threadPoolTaskData = null;
	}
	
}

```

---

## <a id="Java_reflect"> 反射 </a>

### 反射原理

反射能够动态的加载一个类，动态的调用一个方法，动态的访问一个属性。JVM会为每个类创建一个java.lang.Class类的实例，通过该对象可以获取这个类的信息，然后在通过java.lang.reflect包下的API来进行操作。</br>

### 类型信息的存储

如果Java类文件存在内部类，那么编译这个文件时就会产生多个.class文件，命名规则为：外部类名$内部类名.class

例如：

```java
public class Person{
	class Tool{

	}
	interface Communication{
		public void speak();
	}
}
```

会产生 Perlon.class, Person$Tool.class, Person$Communitcation.class 三个文件

.class 文件结构

<table border="1">
	<tbody>
		<tr>
			<td valign="top" style="background:rgb(79,129,189)">
				<p align="left"><span style="color:white">类型</span></p>
			</td>
			<td valign="top" style="background:rgb(79,129,189)">
				<p align="left"><span style="color:white">名称</span></p>
			</td>
			<td valign="top" style="background:rgb(79,129,189)">
				<p align="left"><span style="color:white">数量</span></p>
			</td>
			<td valign="top" style="background:rgb(79,129,189)">
				<p align="left"><span style="color:white">长度</span></p>
			</td>
			<td valign="top" style="background:rgb(79,129,189)">
				<p align="left"><span style="color:white">备注</span></p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u4</p>
			</td>
			<td valign="top">
				<p align="left">magic</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">4Byte</p>
			</td>
			<td valign="top">
				<p align="left">魔数：0xCAFEBABE</br>Od -x命令可以看到，保证虚拟机可以轻松分辨Java文件和非Java文件。
				</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">minor_version</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">2Byte</p>
			</td>
			<td valign="top">
				<p align="left">主版本号，class文件格式变化而变化</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">major_version</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">2Byte</p>
			</td>
			<td valign="top">
				<p align="left">主版本号，class文件格式变化而变化</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">constant_pool_count</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">常量个数</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">cp_info</p>
			</td>
			<td valign="top">
				<p align="left">constant_pool</p>
			</td>
			<td valign="top">
				<p align="left">constant_pool_count-1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">常量池：包含文件中类和接口相关常量。文字字符串、final变量值、类名和方法名的常量。通常占整个类大小的60%</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">access_flags</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">2Byte</p>
			</td>
			<td valign="top">
				<p align="left">访问标志：定义了类或接口。</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">this_class</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">2Byte</p>
			</td>
			<td valign="top">
				<p align="left">常量池索引，指向常量池中该类全限定名的常量池入口</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">super_class</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">2Byte</p>
			</td>
			<td valign="top">
				<p align="left">指向父类全限定名</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">interfaces_count</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">该类实现的接口数量</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">interfaces</p>
			</td>
			<td valign="top">
				<p align="left">interfaces_count</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">由该类实现的接口的常量池引用</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">fields_count</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">字段数量</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">field_info</p>
			</td>
			<td valign="top">
				<p align="left">fields</p>
			</td>
			<td valign="top">
				<p align="left">fields_count</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">字段信息表，描述字段的类型、描述符等</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">methods_count</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">方法数量</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">method_info</p>
			</td>
			<td valign="top">
				<p align="left">methods</p>
			</td>
			<td valign="top">
				<p align="left">methods_count</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">方法本身，每个方法都有一个method_info表，记录了方法的方法名、字段类型、描述符等</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">u2</p>
			</td>
			<td valign="top">
				<p align="left">attributes_count</p>
			</td>
			<td valign="top">
				<p align="left">1</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">属性数量</p>
			</td>
		</tr>
		<tr>
			<td valign="top">
				<p align="left">attribute_info</p>
			</td>
			<td valign="top">
				<p align="left">attributes</p>
			</td>
			<td valign="top">
				<p align="left">attributes_count</p>
			</td>
			<td valign="top">
				<p align="left">?</p>
			</td>
			<td valign="top">
				<p align="left">属性本身</p>
			</td>
		</tr>
	</tbody>
</table>



代理

静态代理

```java
public interface Speakable{
	public void speak(String msg);
}

public class Person implements Speakable{
	@Override
	public void speak(String msg){
		System.out.println("Speak:" + msg);
	}
}

public class PersonProxy implements Speakable{
	private Person person;
	public PersonProxy(Person person){
		this.person=person;
	}
	@Override
	public void speak(String msg){
		this.person.speak(msg);
		System.out.println("运行时间:" + System.currentTimeMillis());
	}
}

public class Boostrap{
	public static void main(String[] args){
		Person person = new Person();
		PersonProxy proxy = new PersonProxy(person);
		proxy.speak("static proxy");
	}
}
```

动态代理

```java
// 调用处理器
public class MyProxy implements InvocationHandler{
	private Object proxied;
	public MyProxy(Object proxied){
		this.proxied=proxied;
	}

	// (代理对象由java动态生成, 被执行的委托方法, 执行委托方法所需要的参数)
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
		method.invoke(this.proxied, args);
		System.out.println("运行时间:" + System.currentTimeMillis());
		return null;
	}
}

public class Bootstrap{
	public static void main(String[] args){
		Person person = new Person();
		Speakable speakable = (Speakable)Proxy.newProxyInstance(
			Speakable.class.getClassLoader(),
			new Class[] {Speakable.class},
			new MyProxy(person)
		);
		speakable.speak("dynamic proxy");
	}
}
```


---

## <a id="Java_network"> Java网络编程 </a>

### TCP/IP协议

TCP/IP(Transmission Control Protocel/Internet Protocol)，传输控制协议/因特网互联协议，网络通讯协议。由网络层的IP协议与传输层的TCP协议组成。</br>

1. 应用层(Application Layer)
2. 传输层(Transport Layer)
3. 网络层(Network Layer)
4. 链接层(Link Layer)
5. 物理层(Physical Layer)

早期的时候，每家公司都有自己的电信号分组方式。逐渐地，一种叫做"以太网"（Ethernet）的协议，占据了主导地位。
以太网规定，一组电信号构成一个数据包，叫做"帧"（Frame）。每一帧分成两个部分：包头（Head）和数据（Data）。

"传输层"的功能，就是建立"端口到端口"的通信。相比之下，"网络层"的功能是建立"主机到主机"的通信。只要确定主机和端口，我们就能实现程序之间的交流。

<!--TODO:P173-->


---

## <a id="Java_web"> Java Web 开发相关技术 </a>

```
javaweb // 应用程序名字
    |- META-INF
    |- resource
    |- WEB-INF
        |- classes // 存放class文件，类加载路径
        |- lib // 第三方jar类库
		|- web.xml // 整个web应用程序的描述文件，通过它配置信息资源
```

用户发送HTTP请求，Web容器通过http://<域名或IP地址>/<应用的名字>/<资源的地址>去定位资源</br>

```xml
<servlet>
	<servlet-name></servlet-name>
	<servlet-class></servlet-class>
</servlet>

<servlet-mapping>
	<servlet-name></servlet-name>
	<servlet-pattern></servlet-pattern>
</servlet-mapping>
```

---

## <a id="algorithm_problem"> 算法题 </a>

### 反转字符串输出

```java
package algorithm;

public class BackString {
	public static void main(String[] args){
		String str = "Hello world!";
		StringBuffer sb = new StringBuffer();
		for(int i = str.length() - 1; i >= 0; i--){
			sb.append(str.substring(i,i+1));
		}
		System.out.print(sb);
	}
}

```

### 求素数

```java
package algorithm.prime;


public class Prime {
	
	private static boolean isPrime(int num){
		if(num < 2){
			return false;
		}
		for(int i = 2; i < num; i++){
			if(num % i == 0){
				return false;
			}
		}
		return true;
	}
	
	public static void main(String[] args){
		int count=0;
		for(int i = 0; i < 1000000; i++){
			boolean flag = Prime.isPrime(i);
//			System.out.println(i + "," + flag);
			if(flag){
				count++;
			}
		}
		System.out.println("count=" + count);
	}
}

```

改进版</br>

```java
package algorithm.prime;


public class Prime {
	
	private static final int MaxLimit=1000000;
	
	private static boolean[] primeList = new boolean[MaxLimit+1];
	
	private static boolean isPrime(int num){
		if(num < 2){
			return false;
		}
		if(num == 2){
			primeList[2] = true;
			return true;
		}
		if(num % 2 == 0){
			return false;
		}
		for(int i = 3; i*i < num; i+=2){
			if(primeList[num]){
				if(num % i == 0){
					return false;
				}
			}
		}
		for(int i = 3; i*i < num; i+=2){
			if(num % i == 0){
				return false;
			}
		}
		primeList[num] = true;
		return true;
	}
	
	public static void main(String[] args){
		int count=0;
		for(int i = 0; i < MaxLimit; i++){
			boolean flag = Prime.isPrime(i);
//			System.out.println(i + "," + flag);
			if(flag){
				count++;
			}
		}
		System.out.println("count=" + count);
	}
}

```

### 打印回文数字

```java
package algorithm;

public class MirrorNum {
	public static boolean isMirrorNumber(int num){
		int temp = num;
		int result = 0;
		while(temp > 0){
			result = result*10 + temp%10;
			temp /= 10;
		}
		return num == result;
	}
	
	public static void main(String[] args){
		for(int i = 10; i < 1000; i++){
			if(isMirrorNumber(i)){
				System.out.println(i);
			}
		}
	}
}

```

### 冒泡排序 BubbleSort

```java
package demo;

public class BubbleSort {
	
	public static void bubbleSort(int[] array){
		for(int i = 1; i < array.length; i++){
			for(int j = 0; j < array.length-i; j++){
				if(array[j] > array[j+1]){
					int temp = array[j];
					array[j] = array[j+1];
					array[j+1] = temp;
				}
			}
		}
	}
	
	public static void main(String[] args){
		int[] array = {5,3,2,1,4};
		bubbleSort(array);
		for(int i = 0; i < array.length; i++){
			System.out.print(array[i] + ",");
		}
	}
}

```

### 插入排序 InsertSort

```java
package demo;

public class InsertSort {
	
	public static void insertSort(int[] array){
		for(int i = 1; i < array.length; i++){
			int temp = array[i];
			int j;
			for(j = i; j > 0; j--){
				if(array[j-1] > temp){
					array[j]=array[j-1];
				}else{
					break;
				}
			}
			array[j] = temp;
		}
		
	}
	
	public static void main(String[] args){
		int[] array = {5,3,2,1,4};
		insertSort(array);
		for(int i = 0; i < array.length; i++){
			System.out.print(array[i] + ",");
		}
	}
}

```

### 快速排序 QuickSort

```java
package demo;

public class QuickSort {
	
	public static void quickSort(int[] a,int low,int high){
		int i,j;
		i=low;
		j=high;

		if(i>j)
			return;
		int temp=a[i];
		while(i<j){
			while(i<j&&a[j]>temp){
				j--;
			}
			if(i<j){
				a[i]=a[j];
				i++;
			}

			while(i<j&&a[i]<temp){
				i++;
			}
			if(i<j){
				a[j]=a[i];
				j--;
			}
		}
		a[i]=temp;
		quickSort(a,low,i-1);
		quickSort(a,i+1,high);
	}
	
	public static void main(String[] args){
		int[] array = {5,3,2,1,4};
		quickSort(array, 0, 4);
		for(int i = 0; i < array.length; i++){
			System.out.print(array[i] + ",");
		}
	}
}

```

### 归并排序

```java
package demo;

public class QuickSort {

	public static void esort(int[] a,int p,int r){
		if(p>=r)
			return;
		int q= (p+r)/2;
		esort(a,p,q);
		esort(a,q+1,r);
		msort(a,p,q,r);
	}

	public static void msort(int[] a,int p,int q,int r){
		int n1=q-p+1;
		int n2=r-q;
		int i,j,k;
		int L[]=new int[n1];
		int R[]=new int[n2];
		for(i=0,k=p;i<n1;i++,k++)
			L[i]=a[k];
		for(j=0;j<n2;j++,k++)
			R[j]=a[k];
		for(i=0,j=0,k=p;i<n1&&j<n2;k++){
			if(L[i]<R[j]){
				a[k]=L[i];
				i++;
			}else {
				a[k]=R[j];
				j++;
			}
		}
		while(i<n1){
			a[k]=L[i];
			i++;
			k++;
		}
		while(j<n2){
			a[k]=R[j];
			j++;
			k++;
		}
	}

	public static void main(String[] args){
		int[] array = {5,3,2,1,4};
		esort(array, 0, 4);
		for(int i = 0; i < array.length; i++){
			System.out.print(array[i] + ",");
		}
	}
}
```



<!--TODO:P320-->
