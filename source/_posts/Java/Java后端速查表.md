date: 2023-11-18 10:04:40
title: Java 后端速查表
categories:
    - Java
tags: 
    - JVM
    - Java
    - 锁
    - 多线程
    - 微服务
    - SpringCloud
---

# 基础概念

## 锁

> [Java 锁详解](https://juejin.cn/post/7035497943625334814)
> [图解ReentrantLock公平锁和非公平锁实现](https://juejin.cn/post/7153270781084958750)
> [【深入AQS原理】我画了35张图就是为了让你深入 AQS](https://juejin.cn/post/6844904146127044622)

### 公平锁 非公平锁
- 公平锁：每个线程获取锁的顺序是按照线程访问锁的先后顺序获取的，最前面的线程总是最先获取到锁。 
- 非公平锁：每个线程获取锁的顺序是随机的，并不会遵循先来先得的规则，所有线程会竞争获取锁。

在Java中 `synchronized` 和 `ReentrantLock` 默认都是非公平锁，当然我们在创建 `ReentrantLock` 时，可以手动指定其为公平锁，但 `synchronized` 只能为非公平锁

公平锁：获取锁时，先将线程自己添加到等待队列的队尾并休眠，当某线程用完锁之后，会去唤醒等待队列中队首的线程尝试去获取锁，锁的使用顺序也就是队列中的先后顺序，在整个过程中，线程会从运行状态切换到休眠状态，再从休眠状态恢复成运行状态，但线程每次休眠和恢复都需要从用户态转换成内核态，而这个状态的转换是比较慢的，所以公平锁的执行速度会比较慢。

非公平锁: 当线程获取锁时，会先通过 CAS 尝试获取锁，如果获取成功就直接拥有锁，如果获取锁失败才会进入等待队列，等待下次尝试获取锁。这样做的好处是，获取锁不用遵循先到先得的规则，从而避免了线程休眠和恢复的操作，这样就加速了程序的执行效率。 

### 乐观锁 悲观锁
乐观锁:
用数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加1。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。
```sql
select id,value,version from TABLE where id=#{id}

update TABLE
set value=2,version=version+1
where id=#{id} and version=#{version};
```

悲观锁:
悲观锁就是在操作数据时，认为此操作会出现数据冲突，所以在进行每次操作时都要通过获取锁才能进行对相同数据的操作，这点跟Java中的`synchronized`很相似，所以悲观锁需要耗费较多的时间。另外与乐观锁相对应的，悲观锁是由数据库自己实现了的，要用的时候，我们直接调用数据库的相关语句就可以了。

**共享锁和排它锁是悲观锁的不同的实现，它俩都属于悲观锁的范畴。**

### 读锁 写锁

### 共享锁(读锁/S锁) 排它锁(写锁/X锁/互斥锁) 意向共享锁 意向排它锁

共享锁(read lock)，又称之为读锁，简称S锁，当事务A对数据加上读锁后，其他事务只能对该数据加读锁，不能做任何修改操作，也就是不能添加写锁。只有当事务A上的读锁被释放后，其他事务才能对其添加写锁。

排它锁(exclusive lock)，又称之为写锁，简称X锁，当事务对数据加上写锁后，其他事务既不能对该数据添加读写，也不能对该数据添加写锁，写锁与其他锁都是互斥的。只有当前数据写锁被释放后，其他事务才能对其添加写锁或者是读锁。
MySQL InnoDB引擎默认`update,delete,insert`都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型。

### 全局锁 表级锁 页级锁 行级锁(记录锁) (MySQL)
- 全局锁: 加全局读锁，让整个库处于只读状态 `Flush tables with read lock (FTWRL)`
- 表级锁: `lock tables … read/write`
- 页级锁: 一次锁定相邻的一组记录，BDB 引擎支持页级锁
- 行级锁: 只有InnoDB支持行级锁，行级锁分为共享锁和排他锁。行级锁并不是直接锁记录，而是锁索引。索引分为主键索引和非主键索引两种，如果一条sql语句操作了主键索引，MySQL就会锁定这条主键索引；如果一条语句操作了非主键索引，MySQL会先锁定该非主键索引，再锁定相关的主键索引。在UPDATE、DELETE操作时，MySQL不仅锁定WHERE条件扫描过的所有索引记录，而且会锁定相邻的键值，即所谓的next-key locking

### 间隙锁 临键锁 记录锁 (MySQL)
- 间隙锁 基于非唯一索引，它锁定一段范围内的索引记录。使用间隙锁锁住的是一个区间，而不仅仅是这个区间中的每一条数据。例如 `select* from goods where id between 1 and 10 for update;`
- 临键锁 是记录锁与间隙锁的组合，它的封锁范围，既包含索引记录，又包含索引区间，是一个左开右闭区间。临键锁的主要目的，也是为了避免幻读(Phantom Read)。如果把事务的隔离级别降级为RC，临键锁则也会失效。每个数据行上的非唯一索引列上都会存在一把临键锁，当某个事务持有该数据行的临键锁时，会锁住一段左开右闭区间的数据。需要强调的一点是，InnoDB 中行级锁是基于索引实现的，临键锁只与非唯一索引列有关，在唯一索引列（包括主键列）上不存在临键锁。例如 `update goods set name = 'Apple' where number = 96;` 则 goods表中隐藏的临键锁有：(-∞, 96],(96, 99],(99, +∞]
- 记录锁 是锁记录，记录锁也叫行锁。例如 `select * from goods where **`id`=**1 for update;`

### 自旋锁 (Java)
自旋锁加锁失败后，线程会忙等待，直到它拿到锁  
自旋锁是通过 CPU 提供的 CAS 函数（Compare And Swap），在「用户态」完成加锁和解锁操作，不会主动产生线程上下文切换，所以相比互斥锁来说，会快一些，开销也小一些。

一般加锁的过程，包含两个步骤：
1. 查看锁的状态，如果锁是空闲的，则执行第二步；
2. 将锁设置为当前线程持有；

CAS 函数就把这两个步骤合并成一条硬件级指令，形成原子指令，这样就保证了这两个步骤是不可分割的，要么一次性执行完两个步骤，要么两个步骤都不执行。
使用自旋锁的时候，当发生多线程竞争锁的情况，加锁失败的线程会「忙等待」，直到它拿到锁。这里的「忙等待」可以用 `while` 循环等待实现，不过最好是使用 CPU 提供的 `PAUSE` 指令来实现「忙等待」，因为可以减少循环等待时的耗电量。

Atomic类中设置值使用自旋锁，不断取内存中的value值，然后CAS更新，若失败则持续自旋重试更新操作

### 无锁 -> 偏向锁 -> 轻量级锁(CAS) -> 重量级锁 (JVM Java锁升级)
1. 偏向锁
  - 如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也就变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作
2. 轻量级锁
  - 当存在第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。注意这里的第二个线程只是申请锁，不存在两个线程同时竞争锁，可以是一前一后地交替执行同步块
  - 只需要将lock属性地址(ThreadID)通过CAS写入对象头即视为加锁成功，因为BasicLock只有一个8字节属性。当存在多个线程抢占轻量级锁的时候，只有一个能够抢占成功，获取轻量级锁恢复正常执行，其他线程都会尝试将该轻量级锁膨胀成重量级锁，也只有一个线程完成锁膨胀。
3. 重量级锁
  - 当同一时间有多个线程竞争锁时，锁就会被升级成重量级锁，此时其申请锁带来的开销也就变大

## 各种一致性问题

### 缓存一致性问题 (Redis 和 DB)
先更新数据库，再删除缓存。
如果第二步执行异常可能会导致数据不一致问题，所以可以采用异步重试的办法
可采用开源组件Canal: 消息队列订阅数据库变更日志`BinLog`，然后投递消息到消息队列，消费消息队列消息再操作缓存

![Alt text](image.png)

> [数据库和缓存的一致性问题，看这一篇就够了](https://juejin.cn/post/7024825393018306574)

### 缓存一致性协议(MESI) (CPU 和 内存)
带有高速缓存的CPU执行计算的流程
1. 程序以及数据被加载到主内存
2. 指令和数据被加载到CPU的高速缓存
3. CPU执行指令，把结果写到高速缓存
4. 高速缓存中的数据写回主内存

![Alt text](image-1.png)

目前流行的多级缓存结构：多级缓存结构

![Alt text](image-2.png)

多核CPU多级缓存一致性协议MESI
- 作用: 多核CPU有多个一级缓存，保证缓存内部数据的一致,不让系统数据混乱
- Intel CPU对缓存一致性协议的实现(MESI = modified + exclusive + shared + invalid)
- MESI中每个缓存行(Cache line:CPU中缓存存储数据的单元，一般为64字节)都有四个状态(假设线程 A 和线程 B 同时对一个变量执行 i++)

MESI 是指4中状态的首字母。每个Cache line有4个状态，可用2个bit表示
1. `M` 修改 (Modified): 该Cache line有效，数据被修改了，和内存中的数据不一致，数据只存在于本Cache中。缓存行必须时刻监听所有试图读该缓存行相对就主存的操作，这种操作必须在缓存将该缓存行写回主存并将状态变成S（共享）状态之前被延迟执行
2. `E` 独享、互斥 (Exclusive): 该Cache line有效，数据和内存中的数据一致，数据只存在于本Cache中。缓存行也必须监听其它缓存读主存中该缓存行的操作，一旦有这种操作，该缓存行需要变成S（共享）状态。
3. `S` 共享 (Shared): 该Cache line有效，数据和内存中的数据一致，数据存在于很多Cache中。缓存行也必须监听其它缓存使该缓存行无效或者独享该缓存行的请求，并将该缓存行变成无效（Invalid）。
4. `I` 无效 (Invalid): 该Cache line无效

MESI状态转换

![Alt text](image-3.png)

![Alt text](image-4.png)

> [最详细的并发研究之CPU缓存一致性协议(MESI)有这一篇就够了！](https://juejin.cn/post/7085303446223454244)

# Java

## Java语言特性

- JDK(Java Development Kit)
- JRE(Java Runtime Environment)

面向对象三大特征:
1. 封装: 封装是指把一个对象的状态信息隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。
2. 继承: 继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。提高代码的重用，程序的可维护性
3. 多态: 表示一个对象具有多种的状态。具体表现为父类的引用指向子类的实例。

Java 和 C++的区别
- 都是面向对象的语言，都支持封装、继承和多态
- Java 不提供指针来直接访问内存，程序内存更加安全
- Java 的类是单继承的，C++ 支持多重继承；虽然 Java 的类不可以多继承，但是接口可以多继承。
- Java 有自动内存管理垃圾回收机制(GC)，不需要程序员手动释放无用内存

为什么说 Java 语言编译与解释并存？
- 编译型语言: 是指编译器针对特定的操作系统将源代码一次性翻译成可被该平台执行的机器码
- 解释型语言: 是指解释器对源程序逐行解释成特定平台的机器码并立即执行。
- Java 程序要经过先编译，后解释两个步骤，由 Java 编写的程序需要先经过编译步骤，生成字节码（*.class 文件），这种字节码必须由 Java 解释器来解释执行。

为什么重写`equals()`时必须重写`hashCode()`方法？
- 两个对象调用 equals 方法返回 true, 那么调用 hashCode 返回的值也必须一样
- 反之 hashCode 返回值一样 equals 可以返回 false，这种情况为哈希碰撞

重载(Overload)和重写(`@Override`)的
- 重载就是同样的一个方法能够根据输入数据的不同，做出不同的处理
- 重写就是当子类继承自父类的相同方法，输入数据一样，但要做出有别于父类的响应时，你就要覆盖父类方法

重写 equals 时为什么一定要重写 hashCode？
- 通常情况下，我们要判断两个对象是否相等，一定要重写 equals 方法。因为在 Object 类中，equals 方法是直接比较对象的引用是否相等。
- hashCode是由对象推导出的一个整型值。Set 集合是用来保存不同对象的，相同的对象就会被 Set 合并，set根据对象的hashCode来判断

深拷贝 vs 浅拷贝
- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

## Java基本类

异常(Throwable)： 
- Exception（异常）：是程序本身可以处理的异常。Exception 类有一个重要的子类 RuntimeException。
  - RuntimeException
    - 例如：ArithmeticException（算术运算异常，一个整数除以 0 时，抛出该异常）
- Error（错误）：是程序无法处理的错误，表示运行应用程序中较严重问题。表示代码运行时 JVM（Java 虚拟机）出现的问题。
  - 例如：当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

`java.util.HashMap`
- JDK1.7: 数组+单链表+链表(链地址法)(头插法)
- JDK1.8: 数组+单链表+红黑树(当链表(尾插法)的深度达到8的时候，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O(n)变成O(logN)提高了效率)
- put操作的流程：
  1. `key.hashcode()`，时间复杂度O(1)
  2. 找到桶以后，判断桶里是否有元素，如果没有，直接new一个entey节点插入到数组中。时间复杂度O(1)
  3. 如果桶里有元素，并且元素个数小于6，则调用equals方法，比较是否存在相同名字的key，不存在则new一个entry插入都链表尾部。时间复杂度O(n)
  4. 如果桶里有元素，并且元素个数大于6，则调用equals方法，比较是否存在相同名字的key，不存在则new一个entry插入都链表尾部。时间复杂度O(logn)
- 如果`new HashMap()`不传值，默认大小是 16，负载因子是 0.75，如果自己传入初始大小 k，初始化大小为 大于 k 的 2 的整数次方，例如如果传 10，大小为 16
- HashMap 的哈希函数怎么设计的吗？
  - hash 函数是先拿到通过 key 的 hashcode，是 32 位的 int 值，然后让 hashcode 的高 16 位和低 16 位进行异或操作。哈希函数也叫扰动函数，一定要尽可能降低 hash 碰撞，越分散越好；算法一定要尽可能高效，因为这是高频操作, 因此采用位运算；
- 为什么采用 hashcode 的高 16 位和低 16 位异或能降低 hash 碰撞？hash 函数能不能直接用 key 的 hashcode？
  - 因为 `key.hashCode()`函数调用的是 key 键值对象自带的哈希函数，返回 int 型散列值(int 值范围为-2147483648~2147483647)，前后加起来大概 40 亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。你想，如果 HashMap 数组的初始大小才 16，用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。

`java.util.concurrent.ConcurrentHashMap`
- 采用了分段锁技术
- 构造方法(Segment数组里面的Entry数组全部加起来的初始化大小, Segment数组的大小)
- 理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment
- 这里给Segment加锁采用的机制是CAS，否则自旋，同时还用到了ReentrantLock(可重入锁)
- JDK1.8中，ConcurrentHashMap摒弃了Segment，而是采用synchronized+CAS+红黑树来实现的。锁的粒度也从段锁缩小为结点(Node)锁

1.7
- put(): 
  1. 通过 key 定位到 Segment，之后在对应的 Segment 中进行具体的 put
  2. 尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 scanAndLockForPut() 自旋获取锁
  3. 重试的次数达到了 MAX_SCAN_RETRIES 则改为阻塞锁获取
  4. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry
  5. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value
  6. 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容
  7. 释放当前 Segment 的锁
- get():
  1. Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上
  2. 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值

1.8与1.7的区别
- put(): 
  1. 直接定位到桶，拿到 first 节点后进行判断:
  2. 为空则 CAS 插入
  3. 为 -1 则说明在扩容，则跟着一起扩容；else 则加锁 put(类似1.7)
- get():
  1. 由于 value 声明为 volatile，保证了修改的可见性，因此不需要加锁

面试题:
1. HashMap数据结构及扩容机制
2. HashMap 1.7和1.8区别，红黑树怎么遍历的.
3. HashMap尾插法和头插法区别: 就是插入时，如果数组位置上已经有元素，1.7 将新元素放到数组中，原始节点作为新节点的后继节点，1.8 遍历链表，将元素放置到链表的最后。这样做可以避免多线程操作时头插法可能会出现环形链表。

## 红黑树

红黑树(Red-Black Tree，简称R-B Tree)，它一种特殊的二叉查找树。
红黑树是特殊的二叉查找树，意味着它满足二叉查找树的特征：任意一个节点所包含的键值，大于等于左孩子的键值，小于等于右孩子的键值。
在自平衡二叉搜索树的基础上，有颜色。即通过与颜色相关的《红黑树5性质》限定了红黑树自平衡的程度，使其不是严格意义上的平衡二叉树。平衡二叉树过于严格的限制了高度差不得超过1，会使树的结构调整过于频繁。这也是为什么要有红黑树。
1. 性质1.(红黑)节点是红色或黑色。
2. 性质2.(黑根)根节点是黑色。
3. 性质3.(黑叶)所有叶子都是黑色。(叶子是NULL节点)
4. 性质4.(二黑)每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
5. 性质5.(黑高)从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

## I/O

传统拷贝流程(下载文件为例):
-  磁盘 -> 内核缓冲区 -> 用户缓冲区 -> 网络堆栈相关的内核缓冲区 -> 网卡
-  DMA从磁盘读取文件到内核缓冲区 -> CPU从内核缓冲区拷贝到用户缓冲区 -> 应用程序调write系统调用把用户缓冲区的内容拷贝到网络堆栈相关的内核缓冲区 -> socket把内核缓冲区的内容发送到网卡上

零拷贝(下载文件为例):
- 磁盘 -> 内核缓冲区 -> 网络堆栈相关的内核缓冲区 -> 网卡
- 磁盘上的数据会通过DMA被拷贝的内核缓冲区 -> 操作系统把内核缓冲区与应用程序共享 -> 应用程序调write系统调用将内核缓冲区的内容拷贝到socket缓冲区中 -> socket把内核缓冲区的内容发送到网卡上

BIO(Blocked I/O): 面向流(单向); 同步阻塞I/O
1. 服务端: 通过ServerSocket注册端口
2. 服务端: 调用`accept()`监听客户端Socket请求
3. 客户端: 调用`connect()`连接服务端
4. 服务端/客户端: 从Socket中获取字节输入流或输出流对数据进行读写操作

NIO(Non-blocked I/O): 面向缓冲区(双向); 非阻塞I/O
- 一个线程对应一个Selector选择器
- 一个Selector对应多个Channel通道
- 一个Channel对应一个Buffer(底层是一个数组)
- 直接缓冲区与非直接缓冲区
  - 直接缓冲区(非堆内存): 本地IO -> 直接内存 -> 本地IO
  - 非直接缓冲区(堆内存): 本地IO -> 直接内存 -> 非直接内存 -> 直接内存 -> 本地IO

Selector选择器
- 可以通过Selector来实现一个I/O线程并发处理N个客户端连接和读写操作
- Selector接多个Channel并监听这些Channel上的事件，使用选择器的事件迭代器遍历获取选择器监听到的事件并判断事件类型分别处理，处理完后清除事件

AIO(Async-Blocked I/O): 异步非阻塞I/O

## 多线程

多线程
- 进程(资源分配的基本单位): 是程序的一次执行过程，是系统运行程序的基本单位。系统运行一个程序即是一个进程从创建，运行到消亡的过程。
- 线程(执行调度的基本单位): 与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是多个线程可以共享同一块内存空间和一组系统资源(堆)，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多。但是频繁的切换线程可能会消耗大量的CPU资源，因为需要频繁的保存和恢复线程运行上下文。
  - 特性
    1. 可见性
    2. 有序性
    3. 原子性
- 纤程: Java不涉及

线程撕裂者: 一个核里面可以跑多个线程
- 一颗CPU可以有多个核，正常的CPU一个核可以同时跑一个线程
- 线程撕裂者就是一个核里面有一个ALU(arithmetic and logic unit)和2寄存器组，ALU可以在2个寄存器组之间快速切换，一个寄存器组存一个线程的工作数据，这样一个核看起来就是同时跑2个线程。例如4核8线程

并行与并发
- 并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是真正的同时。
- 并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS(Transaction per Second 事物数/秒)或者QPS(Queries Per Second 查询数/秒)来反应这个系统的处理能力

线程安全: 
经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。

死锁:
产生死锁的四个必要条件
1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
2. 不可剥夺条件: 进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放(只能是主动释放)。
3. 请求与保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
4. 循环等待条件: 存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, …, pn}，其中Pi等 待的资源被P(i+1)占有(i=0, 1, …, n-1)，Pn等待的资源被P0占有

### Java 线程状态

![Alt text](image.png)



同步: 
Java中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。常见的解决方法是使用`synchronized`关键字。

### `synchronized`关键字
- `synchronized(expression) {// 同步代码块}`: 对表达式`expresssion`求值(值的类型须是引用类型reference type)，获取它所代表的对象，然后尝试获取这个对象的锁 -> 如果能获取锁，则进入同步块执行，执行完后退出同步块，并归还对象的锁(异常退出也会归还); 如果不能获取锁，则阻塞在这里，直到能够获取锁;
- 特性:
  1. 原子性: 同步代码块中的内容要么全部执行要么都不执行
  2. 可见性: 多个线程访问一个资源时，该资源的状态、值信息等对于其他线程都是可见的。一个线程如果要访问该类或对象必须先获得它的锁，而这个锁的状态对于其他任何线程都是可见的，并且在释放锁之前会将对变量的修改刷新到主存当中，保证资源变量的可见性。这点和`volatile`的实现类似，被`volatile`修饰的变量，每当值需要修改时都会立即更新主存，主存是共享的，所有线程可见，所以确保了其他线程读取到的变量永远是最新值，保证可见性
  3. 有序性: 程序执行的顺序按照代码先后执行，每个时刻都只有一个线程访问同步代码块，也就确定了线程执行同步代码块是分先后顺序的，保证了有序性
  4. 可重入性: `synchronized`关键字属于可重入锁。当一个线程试图操作一个由其他线程持有的对象锁的临界资源时，将会处于阻塞状态，但当一个线程再次请求自己持有对象锁的临界资源时，这种情况属于重入锁。通俗一点讲就是说一个线程拥有了锁仍然还可以重复申请锁。
- 源码解读
  - 反编译使用了`synchronized`关键字的类的class文件可以看到两种实现方法:
    1. 字节码指令(`monitorenter`,`monitorexit`): 修饰同步代码块
      - `synchronized`修饰在方法块: 通过 `monitorenter` 和 `monitorexit` 这两个字节码指令获取线程的执行权的。当方法执行完毕退出以后或者出现异常的情况下会自动释放锁
      - JVM执行到`monitorenter`指令时它会尝试获取对象的锁，如果该对象没有锁，或者当前线程已经拥有了这个对象的锁时，它会把计数器+1；然后当执行到`monitorexit` 指令时就会将计数器-1；然后当计数器为0时，锁就释放了。如果获取锁失败，那么当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。
      - 反编译后可以看到一个`monitorenter`和两个`monitorexit`: 这是因为第二个`monitorexit`是给异常处理释放锁用的
      - monitor到底是什么: monitor它就是个监视器，底层源码是C++编写的
    2. `flag=ACC_SYNCHRONIZED`: 修饰同步方法
      - 这标志用来告诉JVM这是一个同步方法，在进入该方法之前先获取相应的锁，锁的计数器加1，方法结束后计数器-1，如果获取失败就阻塞住，知道该锁被释放。

- 可重入锁: 就是一个线程不用释放，可以重复的获取一个锁n次，只是在释放的时候，也需要相应的释放n次。(简单来说：A线程在某上下文中或得了某锁，当A线程想要在次获取该锁时，不会应为锁已经被自己占用，而需要先等到锁的释放)假使A线程即获得了锁，又在等待锁的释放，就会造成死锁。

`monitorenter`, `monitorexit`的指令解析是通过InterpreterRuntime.cpp中的两个方法实现

```c++
// monitorenter(JavaThread 当前获取锁的线程, BasicObjectLock 基础对象锁)
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  if (PrintBiasedLockingStatistics) {
    Atomic::inc(BiasedLocking::slow_path_entry_count_addr());
  }
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) { // UseBiasedLocking是在JVM启动的时候，是否启动偏向锁的标识
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    // 当处于不安全点时，通过 revoke_and_rebias尝试获取偏向锁，如果成功则直接返回，如果失败则进入轻量级锁获取过程
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {// 如果偏向锁未开启，则进入 slow_enter获取轻量级锁的流程
    // BasicObjectLock对象的lock属性的地址用于实现轻量级锁，即所谓的Thread ID
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  assert(Universe::heap()->is_in_reserved_or_null(elem->obj()),
         "must be NULL or an object");
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END

// monitorexit(JavaThread 当前获取锁的线程, BasicObjectLock 基础对象锁)
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorexit(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (elem == NULL || h_obj()->is_unlocked()) {
    THROW(vmSymbols::java_lang_IllegalMonitorStateException());
  }
  ObjectSynchronizer::slow_exit(h_obj(), elem->lock(), thread);
  // Free entry. This must be done here, since a pending exception might be installed on
  // exit. If it is not cleared, the exception handling code will try to unlock the monitor again.
  elem->set_obj(NULL);
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END
```

```c++
// slow_enter
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");

  if (mark->is_neutral()) {// 如果当前是无锁状态, markword的
    // 直接把mark保存到BasicLock对象的_displaced_header字段
    lock->set_displaced_header(mark);
    // 通过CAS将mark word更新为指向BasicLock对象的指针，更新成功表示获得了轻量级锁
    // BasicObjectLock对象的lock属性为Thread ID
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
    // Fall through to inflate() ...
  } 
  // 如果markword处于加锁状态、且markword中的ptr指针指向当前线程的栈帧，表示为重入操作，不需要争抢锁
  else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    lock->set_displaced_header(NULL);
    return;
  }

#if 0
  // The following optimization isn't particularly useful.
  if (mark->has_monitor() && mark->monitor()->is_entered(THREAD)) {
    lock->set_displaced_header (NULL) ;
    return ;
  }
#endif
	// 代码执行到这里，说明有多个线程竞争轻量级锁，轻量级锁通过`inflate`进行膨胀升级为重量级锁
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}
```

> [深入理解synchronized底层源码](https://blog.csdn.net/realize_dream/article/details/106968443)
> [HotSpot 三种锁实现总结](https://blog.csdn.net/qq_31865983/article/details/105024397)

### 无锁 -> 偏向锁 -> 轻量级锁(CAS) -> 重量级锁
锁的实现本质上都对应着一个入口的等待队列

公平锁和非公平锁
- 公平锁: 多个线程按照申请锁的顺序去获得锁，线程会按顺序进入队列，永远是队列第一位先获得锁
- 非公平锁: 多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁

ReentrantLock 中就有公平锁和非公平锁的实现。默认是采用非公平锁的策略来实现锁的竞争逻辑，它内部是使用AQS来实现所资源的竞争，没有竞争到锁资源的线程，会加入到AQS的同步队列里，这个队列是一个FIFO的双向链表。

JDK6之前: 无锁、有锁(重量级锁)
JDK6之后: 无锁 -> 偏向锁 -> 轻量级锁(CAS) -> 重量级锁
1. 偏向锁
  - 如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也就变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作
2. 轻量级锁
  - 当存在第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。注意这里的第二个线程只是申请锁，不存在两个线程同时竞争锁，可以是一前一后地交替执行同步块
  - 只需要将lock属性地址(ThreadID)通过CAS写入对象头即视为加锁成功，因为BasicLock只有一个8字节属性。当存在多个线程抢占轻量级锁的时候，只有一个能够抢占成功，获取轻量级锁恢复正常执行，其他线程都会尝试将该轻量级锁膨胀成重量级锁，也只有一个线程完成锁膨胀。
3. 重量级锁
  - 当同一时间有多个线程竞争锁时，锁就会被升级成重量级锁，此时其申请锁带来的开销也就变大

Q: synchronized什么时候是偏向锁，轻量级锁以级重量级锁.
- 一个线程获得了锁，那么锁就进入偏向模式。第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。如果还有第三个或以上的线程竞争锁时，锁就会被升级成重量级锁。

JVM中对象实例的组成:
1. 对象头: 
  - Mark Word:
    - 对象的hashCode
    - 锁信息: 记录对象锁当前的状态，在申请锁、锁升级等过程中JVM都需要读取对象的Mark Word数据
    - 分代年龄
    - GC标志
  - Class Metadata Address: 类型指针指向对象的类元数据，JVM通过该指针确定该对象是哪个类的实例
1. 实例数据: 存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐
2. 对其填充

先行发生原则(happens-before): 在发生操作B之前，操作A产生的影响能被操作B观察到。先行发生原则是判断数据是否存在竞争、线程是否安全的主要依据

### `volatile`关键字
`volatile`关键字: 当一个变量定义为volatile之后，它将具备两种特性
1. 保证此变量对所有线程的可见性，即当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的
2. 禁止指令重排序优化
  - CPU指令重排序遵循2个原则:
    1. as-if-serial: 不管怎么重排序，(单线程)程序的执行结果不能被改变，即重排序前和排序后的执行结果应该是一样的
    2. happens-before: JDK5后引入，用于保证程序执行的原子性、可见性和有序性
3. 保证不指令重排序的机制: 
  - 字节码层面: 生成的字节码不会重排序
  - CPU层面: 内存屏障: 屏障指令(汇编指令)加载需要保证不重排序的指令之间起到屏障的作用

内存屏障
在介绍内存屏障前，需要知道编译器和 CPU 会在保证程序输出结果一致的情况下，会对代码进行重排序，从指令优化角度提升性能。而指令重排序可能会带来一个不好的结果，导致 CPU 的高速缓存和内存中数据的不一致，而内存屏障（Memory Barrier）就是通过阻止屏障两边的指令重排序从而避免编译器和硬件的不正确优化情况。  
在硬件层面上，内存屏障是 CPU 为了防止代码进行重排序而提供的指令，不同的硬件平台上实现内存屏障的方法可能并不相同。在 Java8 中，引入了 3 个内存屏障的函数，它屏蔽了操作系统底层的差异，允许在代码中定义、并统一由 JVM 来生成内存屏障指令，来实现内存屏障的功能。  
内存屏障可以看做对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作。以loadFence方法为例，它会禁止读操作重排序，保证在这个屏障之前的所有读操作都已经完成，并且将缓存数据设为无效，重新从主存中进行加载。  
Unsafe 中提供了下面三个内存屏障相关方法：
1. `public native void loadFence();`: 内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
2. `public native void storeFence();`: 内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
3. `public native void fullFence();`: 内存屏障，禁止load、store操作重排序

ReentrantLock(可重入锁):
- 指的是一个线程能够对一个临界资源重复加锁

LongAdder
- 采用分段CAS: 除了真正记录数值的base属性外，还有与base相同的数据类型的cell数组，如果存在多个线程同时对Lang做自增操作，则new一个cell元素放到cell数组里供新增的线程做操作(这里会根据线程数自动扩容或缩容cell数组)，使得同时对base做自增操作的线程数变少，自旋占用的CPU变少，最后再用sum求和操作对所有cell属性和base属性做求和操作并返回，这个求出来的和就是所有线程做的操作的总和

### Java线程
Java 中实现多线程的方法
1. 继承 `Thread` 类
2. 实现 `Runnable` 接口: 如果一个类继承 Thread类，则不适合于多个线程共享资源，而实现了 Runnable 接口，就可以方便的实现资源的共享

线程的状态变化:
1. New(创建状态): 在程序中用构造方法创建了一个线程对象后，新的线程对象便处于新建状态，此时它已经有了相应的内存空间和其他资源(程序计数器、本地方法栈、虚拟机栈)，但还处于不可运行状态。新建一个线程对象可采用`Thread` 类的构造方法来实现，例如 `Thread thread=new Thread()`
2. Ready(就绪状态): 新建线程对象后，调用该线程的 `start()` 方法就可以启动线程。当线程启动时，线程进入就绪状态。此时，线程将进入线程队列排队，等待 CPU 调度，这表明它已经具备了运行条件
3. Running(运行状态): 当就绪状态被调用并获得处理器资源时，线程就进入了运行状态。此时，自动调用该线程对象的 `run()` 方法。`run()` 方法定义该线程的操作和功能
4. Blocked(阻塞状态): 一个正在执行的线程遇到`synchronized`，会进入阻塞状态。线程都将进入阻塞状态，阻塞的线程进入调度队列entry set排队，获取到锁的线程才可以转入就绪状态
5. Waiting(等待): 调用`Object.wait()`, `Thread.join()`方法可使一个线程进入不带时限的等待状态，直到其它线程调用了方法`Object.notify()`或`Object.notifyAll()`唤醒了等待状态的线程，被唤醒后可能进入调度队列entry set继续等待获取锁(Blocked状态)或直接获取到锁(Runnable状态)
6. Time_Waiting(超时等待): 调用`Object.wait(long)`, `Thread.join(long)`, `Thread.sleep(long)`方法可使一个线程进入带时限的等待状态，直到其它线程调用了方法`Object.notify()`或`Object.notifyAll()`唤醒了等待状态的线程
7. Terminated(死亡状态): 线程调用 `stop()` 方法时或 `run()` 方法执行结束后，即处于死亡状态。

> [Java 线程状态之 WAITING](https://www.cnblogs.com/zhongchang/articles/10339134.html)

常用的线程池`java.util.concurrent.Executors`类下静态方法:
1. `newSingleThreadExecutor()`: 创建了一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序执行。
2. `newFixedThreadPool(int nThreads)`: 创建了一个固定大小的线程池，每次提交一个任务就创建一个线程，直到线程达到线程池的最大值`nThreads`。线程池的大小一旦达到最大值后，再有新的任务提交时则放入无界阻塞队列中，等到有线程空闲时，再从队列中取出任务继续执行。
3. `newCachedThreadPool()`: 创建了一个可缓存的线程池。当有新的任务提交时，有空闲线程则直接处理任务，没有空闲线程则创建新的线程处理任务，队列中不储存任务。线程池不对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。如果线程空闲时间超过了60秒就会被回收。

java.util.concurrent.Executors:
```java
public ThreadPoolExecutor(int corePoolSize, // 核心线程数(最小可以同时运行的线程数量)
                          int maximumPoolSize, // 线程池最大线程数(当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数)即:当运行的线程达到corePoolSize后新来的任务优先存队列等待，知道队列满后才会用到maximumPoolSize
                          long keepAliveTime, // 超出核心线程数的线程等待new task的时间，超过则terminated
                          TimeUnit unit, // keepAliveTime时间的单位
                          BlockingQueue<Runnable> workQueue, // 用于保存等待执行的任务的阻塞队列: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。用于存放等待线程执行的task的队列只存放由execute方法提交的Runnable任务
                          RejectedExecutionHandler handler // 线程池对拒绝任务的处理策略
                          ); 

public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>())); // 本身是有界队列但是这里未设置其大小限制，默认Integer.MAX_VALUE，此时相当于无界队列
}
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>()); // 这个阻塞队列没有存储空间，这意味着只要有请求到来，就必须要找到一条工作线程处理他，如果当前没有空闲的线程，那么就会再创建一条新的线程
}
```

`Executors`存在什么问题
- `SingleThreadPool`和`FixedThreadPool`使用的是无界队列，其请求队列长度为`Integer.MAX_VALUE`，可能会堆积大量的请求而导致OOM
- `CachedThreadPool`和`ScheduledThreadPool`允许创建的线程数量为`Integer.MAX_VALUE`，可能会大量创建线程而导致OOM

创建线程池的正确姿势，直接使用`java.util.concurrent.ThreadPoolExecutor`类的构造方法来创建
```java
ExecutorService executor = new ThreadPoolExecutor(10, 10, // 10个线程，最大也支持10个
        60L, TimeUnit.SECONDS, // 超时时间60秒
        new ArrayBlockingQueue(10), // 有界队列，容量为10，最多容纳10个task
        new ThreadFactoryBuilder().setNameFormat("my-pool-%d").build(), // 线程名称
        new ThreadPoolExecutor.AbortPolicy()); // 拒绝策略(默认)

```

Q: 线程池怎么设置核心线程数?
- 如果是CPU密集型服务线程数量等于CPU核心数
- 如果是I/O密集型服务: 线程数 = ((工作时间+休息时间)/工作时间) * CPU核心数 * CPU利用率

workQueue(工作队列): 
- `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO（先进先出）原则对任务进行排序。使用该队列，线程池中能创建的最大线程数为`maximumPoolSize`。 
- `LinkedBlockingQueue`: 基于链表结构的无界阻塞队列，按FIFO（先进先出）原则对任务进行排序，吞吐量高于`ArrayBlockingQueue`。使用该队列，线程池中能创建的最大线程数为`corePoolSize`。
- `SynchronousQueue`: 一个不存储元素的阻塞队列。添加任务的操作必须等到另一个线程的移除操作，否则添加操作一直处于阻塞状态。
- `PriorityBlockingQueue`: 一个支持优先级的无界阻塞队列。使用该队列，线程池中能创建的最大线程数为`corePoolSize`。

线程池(`ThreadPoolExecutor`)处理流程:
1. 提交任务 `execute(Runnable)`
2. 核心线程池是否已满? N: 创建线程执行任务
3. 队列是否已满? N: 将任务存储在队列中
4. 线程池是否已满? N: 创建非核心线程执行任务
5. 经过以上步骤还是有新任务则执行拒绝策略 

handler(饱和策略，或者又称拒绝策略): 当队列和线程池都满了，即线程池饱和了，必须采取一种策略处理提交的新任务。
- `AbortPolicy`: 无法处理新任务时，直接抛出异常，这是默认策略。 
- `CallerRunsPolicy`: 用调用者所在的线程来执行任务。
- `DiscardOldestPolicy`: 丢弃阻塞队列中最靠前的一个任务，并执行当前任务。
- `DiscardPolicy`: 直接丢弃任务。

线程池的状态:
1. RUNNING: 该状态的线程池既能接受新提交的任务，又能处理阻塞队列中任务。
2. SHUTDOWN: 该状态的线程池不能接收新提交的任务，但是能处理阻塞队列中的任务。处于 RUNNING 状态时，调用`shutdown()`方法会使线程池进入到该状态。 注意： `finalize()`方法在执行过程中也会隐式调用`shutdown()`方法。 
3. STOP: 该状态的线程池不接受新提交的任务，也不处理在阻塞队列中的任务，还会中断正在执行的任务。在线程池处于 RUNNING 或 SHUTDOWN 状态时，调用 `shutdownNow()` 方法会使线程池进入到该状态
4. TIDYING: 如果所有的任务都已终止，workerCount(有效线程数)=0 。线程池进入该状态后会调用 `terminated()` 钩子方法进入TERMINATED 状态。
5. TERMINATED: 在`terminated()`钩子方法执行完后进入该状态，默认`terminated()`钩子方法中什么也没有做。

线程池的关闭可通过`shutdown()`或者`shutdownNow()`方法 
- `shutdown()`将线程池的状态设置为`SHUTDOWN`状态，只会中断空闲的工作线程
- `shutdownNow()`将线程池的状态设置为`STOP`状态，会中断所有工作线程，不管工作线程是否空闲
- 调用两者中任何一种方法，都会使`isShutdown()`方法的返回值为true；
- 线程池中所有的任务都关闭后，`isTerminated()`方法的返回值为true

Q: 新的任务提交到线程池，线程池是怎样处理的？ 
步骤：
1. 线程池判断核心线程池里的线程是否都在执行任务。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则执行第二步。
2. 线程池判断工作队列是否已经满。如果没有满，则将新提交的任务存储在这个工作队列里进行等待。如果工作队列满了，则执行第三步。
3. 线程池判断线程池的线程是否都处于工作状态。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给饱和策略来处理这个任务。

Q: 父线程子线程怎么共享数据?
- 可以使用`InheritableThreadLocals`可继承线程变量这个类来实现，`ThreadLocals`是线程变量，相当于一个map，每个线程是map的key，value是`set()`进去的值，一个线程使用`get()`只能get到它自己set进去的值，所以不可用于获取父线程的数据。而`InheritableThreadLocals`会在子线程new出来的时候就把自己的value复制进去，所以子线程可以使用这个来共享获取父线程的数据

Q: 线程池怎么维护线程状态，怎么处理线程异常.什么时候task需要queued
- TODO

### AQS(AbstractQueuedSynchronized)
- 抽象队列同步器AQS: 是一个同步框架，它提供通用机制来原子性管理同步状态、阻塞和唤醒线程，以及维护被阻塞线程的队列. AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch
- 底层实现为: volatile + CAS
- AQS内部维护了一个volatile修饰的共享变量，`state`主要用来标记锁的状态。
- AQS通过自定义Node节点来维护一个队列，完成资源获取线程的排队工作。
- AQS通过`park`和`unParkSuccessor`方法来实现阻塞和唤醒线程。
- AQS内部的`compareAndSetState`方法保证了锁状态设置的原子性。

默认ReentrantLock采用的是非公平锁实现，一次ReebtrantLock加锁的过程：
1. 当线程A访问时，先判断`state`所标记值是否为0
2. 发现`state`标识为0，接着将`state`的值通过`compareAndSetState()`方法修改为1
3. 设置当前拥有独占访问权的线程A为自己当前线程
4. 其他线程B再次访问，也是一上来先去判断了一下`state`状态，发现是1，自然CAS失败了，只能乖乖进入等待队列
5. 经过一段时间，线程A访问资源结束，准备释放锁，修改`state`状态为0，准备去唤醒B线程
6. 这时候线程C也过来了，他也来抢占锁资源，发现`state`为0，线程C果断CAS成功，抢占了锁资源，还修改当前线程为自己
7. 线程B被A唤醒准备去获取锁，发现`state`已经是1了，锁资源已经被抢占，结果线程B又只能默默回去等等队列继续等待了

### CAS原理
- CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。
- CAS操作都是通过sun包下Unsafe类实现，而Unsafe类中的方法都是native方法CAS通过调用JNI(Java Native Interface)的c++代码实现的
- unsafe 的cas 依赖了的是 jvm 针对不同的操作系统实现的 `Atomic::cmpxchg`
- `Atomic::cmpxchg` 的实现使用了汇编的 cas 操作，并使用 cpu 硬件提供的 lock信号保证其原子性
- Atomic类中的value是`volatile`的，`volatile`可以保证可见性和有序性
- Atomic类中设置值使用自旋锁，不断取内存中的value值，然后CAS更新，若失败则持续自旋重试更新操作
- 缺点:
  1. ABA问题:
    - CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。`AtomicStampedReference`类具有版本号功能
  2. 只能保证一个共享变量的原子操作: 多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者把多个共享变量合并成一个共享变量来操作(JDK1.5之后提供了`AtomicReference`类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作)

`Atomic::cmpxchg`
```c++
#define LOCK_IF_MP(mp) "cmp $0, " #mp "; je 1f; lock; 1: "// 判断mp是否为多核CPU是则返回LOCK指令

inline jlong Atomic::cmpxchg (jlong exchange_value, volatile jlong* dest, jlong compare_value) {
  bool mp = os::is_MP(); // 
  __arm__ __volatile__ (LOCK_IF_MP(%4) "cmpxchgq %1,(%3)" // MP=mult prosser LOCK_IF_MP返回lock指令 汇编指令cmpxchgq执行原子的CAS
                        : "=a" (exchange_value)
                        : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                        : "cc", "memory");
  return exchange_value;
}
```

- 乐观锁底层实现: lock + cmpxchg 指令
- 悲观锁底层实现: lock 指令
- volatile的底层实现也是用的: lock指令

### 缓存一致性协议: 硬件级别的协议
- 作用: 多核CPU有多个一级缓存，保证缓存内部数据的一致,不让系统数据混乱
- Intel CPU对缓存一致性协议的实现(MESI = modified + exclusive + shared + invalid)
- MESI中每个缓存行(Cache line:CPU中缓存存储数据的单元，一般为64字节)都有四个状态(假设线程 A 和线程 B 同时对一个变量执行 i++)
  1. 核心 A 从内存中加载变量 i，并将缓存行设置为 E（独享），随后通过总线嗅探检查内存中对变量 i 的操作；
  2. 核心 B 从内存中加载变量 i，总线嗅探机制会将核心 A 与核心 B 的缓存行设置为 S（共享）
  3. 核心 A 对变量 i 进行修改，缓存行设置为 M（修改），而核心 B 被通知修改缓存行为 I（无 效）。如果存在高并发，则交给总线裁决
  4. 核心 A 将修改后数据同步回内存，并将变量设置为 E（独享）
  5. 核心 B 重新刷新缓存行，并将缓存行核心 A 和核心 B 的缓存行设置为 S（共享）
- CPU 是通过总线和内存进行数据传输的。在多核心时代下，多个核心通过同一条总线和内存以及其他硬件进行通信
- 通过在 inc 指令前添加 lock 前缀，即可让该指令具备原子性。多个核心同时执行同一条 inc 指令时，会以串行的方式进行
- 伪共享问题: 伪共享是指多个线程同时读写同一个缓存行中的变量，而导致缓存行失效的问题。尽管多个线程分别访问的是不同的数据，但由于它们存在同一个缓存行中，只要任何一方修改都会使得缓存失效，降低了运算效率。
- 解决方案: 
  1. 字节填充，在变量前后填充多个字节使得 变量大小+填充的字节=64字节，这样这个变量肯定会独占一个缓存行。
    - 在JDK 8 之前一般都是通过代码手动字节填充的方式来避免该问题，也就是创建一个变量时使用填充字段填充该变量所在的缓存行，这样就避免了将多个变量存放在同一个缓存行中。早期的`LinkedTransferQueue`。就是使用了追加字节填充来解决伪共享，另外在早起`ConcurrentHashMap`、以及无锁并发框架`Disruptor`中均使用这种技术。
  2. JDK8以及之后的版本 Java 提供了`sun.misc.Contended` 注解，通过`@Contented`注解就可以解决伪共享的问题。使用`@Contented`注解后会增加128字节的padding，并且需要开启`-XX:-RestrictContended`选项后才能生效。

> [CPU缓存一致性协议MESI](https://blog.csdn.net/m18870420619/article/details/82431319)
> [缓存行与MESI](https://www.cnblogs.com/jokerjason/p/9584402.html)

### 用户态内核态
- Linux操作系统的体系架构分为用户态和内核态
- 内核态: 本质上是一种软件，控制计算机硬件资源(CPU资源、存储资源、I/O资源等)
- 用户态: 上层应用程序的活动空间
- 上层应用想要访问计算机硬件资源需要通过内核提供的访问接口(系统调用)来调用
- 系统调用是操作系统的最小功能单位
- 从用户态到内核态切换可以通过三种方式:
  1. 系统调用
  2. 异常: 如果当前进程运行在用户态，如果这个时候发生了异常事件，就会触发切换。例如：缺页异常
  3. 外设中断: 当外设完成用户的请求时，会向CPU发送中断信号

### 多线程框架

Disruptor 框架
- 英国外汇交易公司LMAX开发的一个高性能队列。主要用于线程与线程之间的消息传递
- QPS: 600w
- 为什么快:
  1. CAS: ArrayBlockingQueue使用了重量级锁(lock锁)，而Disruptor采用CAS操作
  2. 消除伪共享: 解决方案: 字节填充，在变量前后填充多个字节使得 变量大小+填充的字节=64字节，这样这个变量肯定会独占一个缓存行。
  3. RingBuffer: 环形数组，没有删除操作，超过容量会直接覆盖原有数据，避免了垃圾回收。大小必须为2的n次方，因为取余运算直接使用的是位运算，使得元素定位更快。

参考
- [接口耗时优化与cpu飙高解决:多线程改造与压测](https://blog.csdn.net/ylx1066863710/article/details/117850439)

## JVM

### 类加载
类加载子系统: 根据给定的全限定名类名(如java.lang.Object)来装载class文件的内容到方法区(Method Area)
1. Bootstrap ClassLoader(启动类加载器): `$JAVA_HOME`中`jre/lib/rt.jar`里所有的class，由C++实现
2. Extension ClassLoader(扩展类加载器): 负责加载java平台中扩展功能的一些jar包，包括`$JAVA_HOME`中`jre/lib/*.jar`或`-D java.ext.dirs`指定目录下的jar包
3. App ClassLoader(系统类加载器): 负责加载classpath中指定的jar包及目录中class
4. Custom ClassLoader(用户自定义类加载器): 属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader

双亲委派机制: JVM对class文件采用按需加载的方式，在加载时JVM采用的是双亲委派机制，即把请求交由父类处理，它是一种任务委派模式
1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
2. 如果父类加载器还存在其父类加载器，则进一步向上委托，一次递归，请求最终将到达顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，若无法完成，子类加载器才会去加载。

### JVM 内存模型
运行时数据区(Runtime Data Area)
1. 程序计数器(Program Counter Register) <- 线程不共享
2. 本地方法栈(Native Method Stack) <- 线程不共享
3. 虚拟机栈(Java Virtual Machine Stack) <- 线程不共享
4. 方法区(Method Area) <- 线程共享
5. 堆(Heap) <- 线程共享

本地内存
1. 直接内存(Direct Memory)
2. 方法区(Method Area): 1.8之后挪到了本地内存

#### 1. 程序计数器(Program Counter Register)
每个线程都要它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址，若为native方法则为undefined

#### 2. 本地方法栈(Native Method Stack)
本地方法栈用于管理本地方法的调用

#### 3. 虚拟机栈(Java Virtual Machine Stack)
- 线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)
- 栈帧的内部结构：
  1. 局部变量表(Local Variables): 最基本的存储单元是Slot(变量槽)，容量大小是在编译期确定下来的，并保存在方法的Code属性的maximum local variables数据项中。JVM会为局部变量表中的每一个Slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值。
  2. 操作数栈(operand Stack)(或表达式栈): 用于保存计算过程的中间结果
  3. 动态链接(DynamicLinking)(或指向运行时常量池的方法引用): 每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接(Dynamic Linking)
  4. 方法返回地址(Return Address)(或方法正常退出或者异常退出的定义): 存放调用该方法的pc寄存器的值

#### 4. 方法区(Method Area)
方法区包含运行时常量池(Runtime Constant Pool)
元空间(Metaspace)是其实现，元空间并不在虚拟机中，而是使用本地内存  
1.8之前方法区在运行时数据区，1.8之后挪到了本地内存

#### 5. 堆(Heap)
年青代:老年代=1:2
- 年青代(Young): Eden:From:To=8:1:1
  - Eden: 新创建的对象绝大部分会分配在Eden区。当Eden区内存不够的时候，就会触发MinorGC
  - Survivor 0(From): 在GC开始的时候，对象只会存在于Eden区和名为From的Survivor区，To区是空的，一次MinorGc过后，Eden区和SurvivorFrom区存活的对象会移动到SurvivorTo区中，然后会清空Eden区和SurvivorFrom区，并对存活的对象的年龄+1，如果对象的年龄到`15`(对象年龄信息保存在对象头里)，则直接分配到老年代。
  - Survivor 1(To)
- 老年代(Tenured): 老年代存放从年轻代存活的对象。一般来说老年代存放的都是生命期较长的对象。注: 大对象(大小超过S0或S1区一半大小)的直接进入老年代

### GC(Generational Collecting)垃圾回收:
- Minor GC: 当伊甸园的空间满时，程序又需要创建对象，触发Minor GC
- Full GC: 当老年代内存不足时，对老年代进行垃圾回收。这时可能会伴随着STW(Stop The World)
- STW(Stop The World): 停止所有线程，进行垃圾回收，这时候线程会被阻塞，直到垃圾回收完成
  - Q: 为什么要STW？-> A: 如果不执行STW的话在Full GC的过程中如果有一个线程执行完毕，那么这个线程的局部变量表里面所指向的在堆里的对象都会变成垃圾，但是此时Full GC还没执行完，那么这次Full GC执行所得到的结果是不准确的

判断对象是否需要回收
#### 1. 引用计数法
给对象添加一个引用计数器，每当有一个地方引用这个对象时，就将计数器加一，引用失效时，计数器就减一。当一个对象的引用计数器为零时，说明此对象没有被引用,将会被垃圾回收。但是难以解决循环引用问题。

#### 2. 可达性分析法
通过一系列的 `GC Roots` 的对象作为起始点，从这些节点出发所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连的时候说明对象不可用。

可作为GC Roots的对象: 
1. 虚拟机栈中引用的对象: 引用栈帧中的本地变量表的所有对象
2. 方法区静态属性引用的对象: 引用方法区该静态属性的所有对象
3. 方法区常量引用的对象: 引用方法区中常量的所有对象
4. 本地方法栈中引用的对象: 引用Native方法的所有对象

#### 垃圾回收算法
1. 标记-清除算法(Mark-Sweep): 
  - 分为两个阶段：标记阶段(标记出所有需要被回收的对象) -> 清除阶段(回收被标记的对象所占用的空间)
  - 缺点: 效率不高、空间会产生大量碎片
2. 复制算法(Copying): 将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉，这样一来就不容易出现内存碎片的问题。Eden:From:To=8:1:1
3. 标记-整理算法(Mark-Compact): 在完成标记之后，它不是直接清理可回收对象，而是将存活对象都向一端移动，然后清理掉端边界以外的内存
4. 分代收集(Generational Collection): 根据对象的生命周期划分几块内存区，一般是分为新生代和老年代。新生代:老年代=1:2

#### 垃圾收集器:
1. Serial/Serial Old收集器: 单线程收集器，进行垃圾收集时，必须暂停所有用户线程
  - Serial: 新生代 Copying算法
  - Serial Old: 老年代 Mark-Compact算法
2. ParNew收集器: Serial收集器的多线程版本
3. Parallel Scavenge收集器: 新生代的多线程收集器回收期间不需要暂停其他用户线程 Copying算法
4. Parallel Old收集器: 多线程 Mark-Compact算法
5. CMS(Concurrent Mark Sweep)收集器: 并发收集器，优点是最短回收停顿时间 Mark-Sweep算法
6. G1收集器: 并行与并发收集器，并且它能建立可预测的停顿时间模型

### JVM虚拟机调优:
- 主要是减少STW发生的频率，因为发生STW的时候会阻塞全部的用户线程，在用户看来就是应用程序卡顿
- 能不能通过调整JVM参数是的几乎发生Full GC?
  - 默认年青代:老年代=1:2，改成2:1，那么当年轻代满的时候绝大部分昭生夕死的对象会被干掉，只有实在干不掉的才会挪到老年代
- jVisualVM: 可视化工具，可以查看JVM的内存使用情况，安装插件可以观察到堆分代模型的整个GC过程


## Java 定时任务

### 单机定时任务技术选型

#### `java.util.Timer`
Timer 内部使用一个叫做 TaskQueue 的类存放定时任务，它是一个基于最小堆实现的优先级队列。TaskQueue 会按照任务距离下一次执行时间的大小将任务排序，保证在堆顶的任务最先执行。这样在需要执行任务时，每次只需要取出堆顶的任务运行即可
```java
// 示例代码：
TimerTask task = new TimerTask() {
    public void run() {
        System.out.println("当前时间: " + new Date() + "n" +
                "线程名称: " + Thread.currentThread().getName());
    }
};
System.out.println("当前时间: " + new Date() + "n" + "线程名称: " + Thread.currentThread().getName());
Timer timer = new Timer("Timer");
long delay = 1000L;
timer.schedule(task, delay);
//输出：
//当前时间: Fri May 28 15:18:47 CST 2021n线程名称: main
//当前时间: Fri May 28 15:18:48 CST 2021n线程名称: Timer
```

不过其缺陷较多，比如一个 Timer 一个线程，这就导致 Timer 的任务的执行只能串行执行，一个任务执行时间过长的话会影响其他任务（性能非常差），再比如发生异常时任务直接停止(Timer 只捕获了 InterruptedException)。`ScheduledThreadPoolExecutor` 支持多线程执行定时任务并且功能更强大，是 Timer 的替代品。

#### `ScheduledExecutorService`

```java
// 示例代码：
TimerTask repeatedTask = new TimerTask() {
    @SneakyThrows
    public void run() {
        System.out.println("当前时间: " + new Date() + "n" +
                "线程名称: " + Thread.currentThread().getName());
    }
};
System.out.println("当前时间: " + new Date() + "n" + "线程名称: " + Thread.currentThread().getName());
ScheduledExecutorService executor = Executors.newScheduledThreadPool(3);
long delay  = 1000L;
long period = 1000L;
executor.scheduleAtFixedRate(repeatedTask, delay, period, TimeUnit.MILLISECONDS);
Thread.sleep(delay + period * 5);
executor.shutdown();
//输出：
//当前时间: Fri May 28 15:40:46 CST 2021n线程名称: main
//当前时间: Fri May 28 15:40:47 CST 2021n线程名称: pool-1-thread-1
//当前时间: Fri May 28 15:40:48 CST 2021n线程名称: pool-1-thread-1
//当前时间: Fri May 28 15:40:49 CST 2021n线程名称: pool-1-thread-2
//当前时间: Fri May 28 15:40:50 CST 2021n线程名称: pool-1-thread-2
//当前时间: Fri May 28 15:40:51 CST 2021n线程名称: pool-1-thread-2
//当前时间: Fri May 28 15:40:52 CST 2021n线程名称: pool-1-thread-2
```

缺点: 不论是使用 Timer 还是 ScheduledExecutorService 都无法使用 Cron 表达式指定任务执行的具体时间

#### Spring Task
Spring 自带的定时调度只支持单机，并且提供的功能比较单一。底层是基于 JDK 的 ScheduledThreadPoolExecutor 线程池来实现的
```java
/**
 * cron：使用Cron表达式。　每分钟的1，2秒运行
 */
@Scheduled(cron = "1-2 * * * * ? ")
public void reportCurrentTimeWithCronExpression() {
  log.info("Cron Expression: The time is now {}", dateFormat.format(new Date()));
}
```

### 分布式定时任务技术选型
通常情况下，一个定时任务的执行往往涉及到下面这些角色：
- 任务: 首先肯定是要执行的任务，这个任务就是具体的业务逻辑比如定时发送文章
- 调度器: 其次是调度中心，调度中心主要负责任务管理，会分配任务给执行器
- 执行器: 最后就是执行器，执行器接收调度器分派的任务并执行

#### Quartz
优缺点总结：
- 优点: 可以与 Spring 集成，并且支持动态添加任务和集群。
- 缺点: 分布式支持不友好，没有内置 UI 管理控制台、使用麻烦（相比于其他同类型框架来说）

#### Elastic-Job
基于Quartz和ZooKeeper的分布式调度解决方案  
ElasticJob 支持任务在分布式场景下的分片和高可用、任务可视化管理等功能  
Elastic-Job 没有调度中心这一概念，而是使用 ZooKeeper 作为注册中心，注册中心负责协调分配任务到不同的节点上。  
Elastic-Job 中的定时调度都是由执行器自行触发，这种设计也被称为去中心化设计（调度和处理都是执行器单独完成）

## Java Agent
在JDK1.5以后，我们可以使用agent技术构建一个独立于应用程序的代理程序（即为Agent），用来协助监测、运行甚至替换其他JVM上的程序。使用它可以实现虚拟机级别的AOP功能。
Agent分为两种，一种是在主程序之前运行的Agent，一种是在主程序之后运行的Agent（前者的升级版，1.6以后提供）

我们在主程序的VM options添加上启动参数 `-javaagent: 你的路径/test-1.0-SNAPSHOT.jar=hah` 其中hah为传入permain方法的agentArgs参数。


---
# Spring & SpringBoot
Spring Framework 它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发
Spring 提供的核心功能主要是 IoC 和 AOP

Core Container: Spring 框架的核心模块，主要提供 IoC 依赖注入功能的支持
- spring-core: Spring 框架基本的核心工具类
- spring-beans: 提供对 bean 的创建、配置和管理等功能的支持
- spring-context: 提供对国际化、事件传播、资源加载等功能的支持
- spring-expression: 提供对表达式语言(Spring Expression Language)SpEL 的支持，只依赖于 core 模块，不依赖于其他模块，可以单独使用

### IoC(Inversion of Control:控制反转)
IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理
- 控制: 指的是对象创建(实例化、管理)的权力
- 反转: 控制权交给外部环境(Spring 框架、IoC 容器)

IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map(key，value)，Map 中存放的是各种对象

Spring Bean: 代指的就是那些被 IoC 容器所管理的对象
`org.springframework.beans` 和 `org.springframework.context` 这两个包是 IoC 实现的基础

将一个类声明为 Bean 的注解
- `@Component`: 通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注
- `@Repository`: 对应持久层即 Dao 层，主要用于数据库相关操作
- `@Service`: 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller`: 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面

`@Component` 和 `@Bean` 的区别
- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。
- `@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 `@Bean`来实现

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

注入 Bean 的注解
Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以用于注入 Bean
- `@Autowired`: `org.springframework.bean.factory`
- `@Resource`: `javax.annotation`
- `@Inject`: `javax.inject`

`@Autowired` 和 `@Resource` 的区别
- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- Autowired 默认的注入方式为byType（根据类型进行匹配），`@Resource`默认注入方式为 byName（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。Autowired 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 name 属性来显式指定名称

#### 循环依赖
Spring中发生循环依赖，简单讲就是A的bean依赖B的bean，B的bean又依赖A的bean。
Spring解决循环依赖的思路就是，当A的bean需要B的bean的时候，提前将A的bean放在缓存中（实际是将A的ObjectFactory放到三级缓存），然后再去创建B的bean，但是B的bean也需要A的bean，那么这个时候就去缓存中拿A的bean，B的bean创建完毕后，再回来继续创建A的bean，最终完成循环依赖的解决。

### AOP
- spring-aspects: 该模块为与 AspectJ 的集成提供支持。
- spring-aop: 提供了面向切面的编程实现。
- spring-instrument: 提供了为 JVM 添加代理(agent)的功能。 具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文件，就像这些文件是被类加载器加载的一样，这个模块的使用场景非常有限

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 Cglib 生成一个被代理对象的子类来作为代理

Spring AOP 和 AspectJ AOP 有什么区别
- Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。
- Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)

AOP常见术语
- 目标(Target): 被通知的对象
- 代理(Proxy): 向目标对象应用通知之后创建的代理对象
- 连接点(JoinPoint): 目标对象的所属类中，定义的所有方法均为连接点
- 切入点(Pointcut): 被切面拦截 / 增强的连接点（切入点一定是连接点，连接点不一定是切入点）
- 通知(Advice): 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情
- 切面(Aspect): 切入点(Pointcut)+通知(Advice)
- 织入(Weaving): 将通知应用到目标对象，进而生成代理对象的过程动作

### Spring MVC
MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码
Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)

Spring MVC 的核心组件
- `DispatcherServlet`: 核心的中央处理器，负责接收请求、分发，并给予客户端响应。
- `HandlerMapping`: 处理器映射器，根据 uri 去匹配查找能处理的 Handler ，并会将请求涉及到的拦截器和 Handler 一起封装。
- `HandlerAdapter`: 处理器适配器，根据 HandlerMapping 找到的 Handler ，适配执行对应的 Handler；
- `Handler`: 请求处理器，处理实际请求的处理器。
- `ViewResolver`: 视图解析器，根据 Handler 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 DispatcherServlet 响应客户端

### Data Access/Integration
- spring-jdbc: 提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响。
- spring-tx: 提供对事务的支持。
- spring-orm: 提供对 Hibernate、JPA 、iBatis 等 ORM 框架的支持。
- spring-oxm: 提供一个抽象层支撑 OXM(Object-to-XML-Mapping)，例如：JAXB、Castor、XMLBeans、JiBX 和 XStream 等。
- spring-jms: 消息服务。自 Spring Framework 4.1 以后，它还提供了对 spring-messaging 模块的继承

### Spring Web
- spring-web: 对 Web 功能的实现提供一些最基础的支持。
- spring-webmvc: 提供对 Spring MVC 的实现。
- spring-websocket: 提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
- spring-webflux: 提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步

### Spring Test
Spring 团队提倡测试驱动开发(TDD)。有了控制反转 (IoC)的帮助，单元测试和集成测试变得更简单。

Spring 的测试模块对 JUnit(单元测试框架)、TestNG(类似 JUnit)、Mockito(主要用来 Mock 对象)、PowerMock(解决 Mockito 的问题比如无法模拟 final, static， private 方法)等等常用的测试框架支持的都比较好

### Spring 框架中用到了哪些设计模式
- 工厂设计模式: Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
- 代理设计模式: Spring AOP 功能的实现。
- 单例设计模式: Spring 中的 Bean 默认都是单例的。
- 模板方法模式: Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- 包装器设计模式: 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源
- 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
- 适配器模式: Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller

### Spring 事务
- 编程式事务: 在代码中硬编码(不推荐使用): 通过 TransactionTemplate或者 TransactionManager 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
- 声明式事务: 在 XML 配置文件中配置或者直接基于注解（推荐使用）: 实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）

事务传播行为
1. `TransactionDefinition.PROPAGATION_REQUIRED`: 使用的最多的一个事务传播行为，我们平时经常使用的`@Transactional`注解默认使用就是这个事务传播行为。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务
2. `TransactionDefinition.PROPAGATION_REQUIRES_NEW`: 创建一个新的事务，如果当前存在事务，则把当前事务挂起。也就是说不管外部方法是否开启事务，`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰
3. `TransactionDefinition.PROPAGATION_NESTED`: 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`
4. `TransactionDefinition.PROPAGATION_MANDATORY`: 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常(mandatory强制性,使用的很少)

若是错误的配置以下 3 种事务传播行为，事务将不会发生回滚
1. `TransactionDefinition.PROPAGATION_SUPPORTS`: 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行
2. `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`: 以非事务方式运行，如果当前存在事务，则把当前事务挂起
3. `TransactionDefinition.PROPAGATION_NEVER`: 以非事务方式运行，如果当前存在事务，则抛出异常

事务中的隔离级别
```java
public enum Isolation {
    // 使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED 隔离级别
    DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
    // 低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
    READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
    // 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
    READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
    // 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生
    REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
    // 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别
    SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);

    private final int value;

    Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

`@Transactional(rollbackFor = Exception.class)`注解
当 `@Transactional` 注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚  
在 `@Transactional` 注解中如果不配置rollbackFor属性,那么事务只会在遇到`RuntimeException`的时候才会回滚，加上 `rollbackFor=Exception.class`,可以让事务在遇到非运行时异常时也回滚

### Spring JPA

实体之间的关联关系注解
```java
@OneToOne // 一对一。
@ManyToMany // 多对多。
@OneToMany // 一对多。
@ManyToOne // 多对一
```

在数据库中非持久化一个字段
```java
static String transient1; // not persistent because of static
final String transient2 = "Satish"; // not persistent because of final
transient String transient3; // not persistent because of transient
@Transient
String transient4; // not persistent because of @Transient
```

JPA 的审计功能
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate // 表示该字段为创建时间字段，在这个实体被 insert 的时候，会设置值
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy // 表示该字段为创建人，在这个实体被 insert 的时候，会设置值
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}
```

### Spring Security
Spring Security 重要的是实战

加密算法工具类
这些加密算法实现类的父类是 PasswordEncoder ，如果你想要自己实现一个加密算法的话，也需要继承 PasswordEncoder
```java
public interface PasswordEncoder {
    // 加密也就是对原始密码进行编码
    String encode(CharSequence var1);
    // 比对原始密码和数据库中保存的密码
    boolean matches(CharSequence var1, String var2);
    // 判断加密密码是否需要再次进行加密，默认返回 false
    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}
```

批量更换系统使用的加密算法
推荐的做法是通过 `DelegatingPasswordEncoder` 兼容多种不同的密码加密方案，以适应不同的业务需求  
从名字也能看出来，`DelegatingPasswordEncoder` 其实就是一个代理类，并非是一种全新的加密算法，它做的事情就是代理上面提到的加密算法实现类。在 Spring Security 5.0之后，默认就是基于 `DelegatingPasswordEncoder` 进行密码加密的

### Spring 启动原理

### SpringBoot 自动装配原理
通过 Spring Boot 的全局配置文件 `application.properties`或`application.yml`即可对项目进行设置比如更换端口号，配置 JPA 属性等等  
Spring Boot 中，我们直接引入一个 starter 即可。比如你想要在项目中使用 redis 的话，直接在项目中引入对应的 starter 即可
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

SpringBoot 的核心注解 SpringBootApplication:  
`@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`
- `@EnableAutoConfiguration`： 启用 SpringBoot 的自动配置机制
- `@Configuration`： 允许在上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan`： 扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。容器中将排除TypeExcludeFilter和AutoConfigurationExcludeFilter
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@ComponentScan // 扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描启动类所在的包下所有的类
@EnableAutoConfiguration // 启用 SpringBoot 的自动配置机制
public @interface SpringBootApplication {

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //实际上它也是一个配置类
public @interface SpringBootConfiguration {
}
```

`@EnableAutoConfiguration`: 实现自动装配的核心注解
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //作用：将main包下的所有组件注册到容器中
// 自动装配核心功能的实现实际是通过 AutoConfigurationImportSelector类
@Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

AutoConfigurationImportSelector: 加载自动装配类  
继承关系如下:
```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {

}

public interface DeferredImportSelector extends ImportSelector {

}

public interface ImportSelector {
    // 该方法主要用于获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中
    String[] selectImports(AnnotationMetadata var1);
}
```

selectImports方法主要用于获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中
```java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.判断自动装配开关是否打开
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>.获取所有需要装配的bean
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            // getAutoConfigurationEntry主要负责加载自动配置类的
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
```

`getAutoConfigurationEntry()`的源码:
```java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>. isEnabled: 判断自动装配开关是否打开。默认spring.boot.enableautoconfiguration=true，可在 application.properties 或 application.yml 中设置
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>. 用于获取EnableAutoConfiguration注解中的 exclude 和 excludeName
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>. 获取需要自动装配的所有配置类，读取spring-boot/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories
            // 一般名称为XXXAutoConfiguration的作用就是按需加载组件
            // 不光是这个依赖下的META-INF/spring.factories被读取到，所有 Spring Boot Starter 下的META-INF/spring.factories都会被读取到
            // eg: druid 数据库连接池的 Spring Boot Starter 就创建了META-INF/spring.factories文件
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>. 筛选，@ConditionalOnXXX 中的所有条件都满足，该类才会生效
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

```java
@Configuration
// 检查相关的类：RabbitTemplate 和 Channel是否存在
// 存在才会加载
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
```

### autoconfig原理

### `@Autowired`实现原理
@Autowired 是一个注释，它可以对类成员变量、方法及构造函数进行标注，让 spring 完成 bean 自动装配的工作。
@Autowired 默认是按照类去匹配，配合 @Qualifier 指定按照名称去装配 bean。




### SpringBoot 注解字典
`@SpringBootApplication`
```java
@SpringBootApplication // 创建 SpringBoot 项目之后会默认在主类加上
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合
```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration // 启用 SpringBoot 的自动配置机制
// 扫描被@Component (@Repository,@Service,@Controller)注解的 bean，注解默认会扫描该类所在的包下所有的类
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}

package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration // 允许在 Spring 上下文中注册额外的 bean 或导入其他配置类
public @interface SpringBootConfiguration {

}
```

`@Autowired`: 自动导入对象到类中，被注入进的类同样要被 Spring 容器管理
```java
@Service
public class UserService {
  //......
}

@RestController
@RequestMapping("/users")
public class UserController {
   @Autowired
   private UserService userService;
   //......
}
```

`@Component`, `@Repository`, `@Service`, `@Controller`: 
- `@Component`: 通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用@Component 注解标注。
- `@Repository`: 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service`: 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller`: 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

`@RestController`: `@Controller`+`@ResponseBody`, 表示这是个控制器 bean,并且是将函数的返回值直接填入 HTTP 响应体中,是 REST 风格的控制器。单独使用 `@Controller` 不加 `@ResponseBody`的话一般是用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。`@Controller` + `@ResponseBody` 返回 JSON 或 XML 形式数据

`@Scope`: 声明 Spring Bean 的作用域  
四种常见的 Spring Bean 的作用域：
1. singleton: 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
2. prototype: 每次请求都会创建一个新的 bean 实例。
3. request: 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
4. session: 每一个 HTTP Session 会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

`@Configuration`: 一般用来声明配置类，可以使用 `@Component`注解替代，不过使用`@Configuration`注解声明配置类更加语义化
```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

### 前后端传值

- `@PathVariable`: 用于获取路径参数
- `@RequestParam`: 用于获取查询参数
```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
    // 前端调/klasses/123456/teachers?type=web
    // klassId=123456,type=web
}
```

`@RequestBody`: 用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且`Content-Type` 为 `application/json` 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象
```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```

UserRegisterRequest对象:
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserRegisterRequest {
    @NotBlank
    private String userName;
    @NotBlank
    private String password;
    @NotBlank
    private String fullName;
}
```

我们发送 post 请求到这个接口，并且 body 携带 JSON 数据:
```json
{"userName":"coder","fullName":"shuangkou","password":"123456"}
```

后端就可以直接把 json 格式的数据映射到我们的 UserRegisterRequest 类上。一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PathVariable`

### 读取配置信息
数据源`application.yml`内容如下:
```yml
test2020: 2020年

my-profile:
  name: David
  email: nobodynowhere@163.com

library:
  location: 中国
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

`@Value`(常用)
```java
@Value("${wuhan2020}")
String wuhan2020;
```

`@ConfigurationProperties`(常用): 读取配置信息并与 bean 绑定
```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  //省略getter/setter
  //......
}
```

@PropertySource(不常用): 读取指定 properties 文件
```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  //省略getter/setter
  //......
}
```

### 参数校验
JSR(Java Specification Requests) 是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们 JavaBean 的属性上面  
SpringBoot 项目的 `spring-boot-starter-web` 依赖中已经有 `hibernate-validator` 包，不需要引用相关依赖
更新版本的 `spring-boot-starter-web` 依赖中不再有 `hibernate-validator` 包（如2.3.11.RELEASE），需要自己引入 `spring-boot-starter-validation` 依赖  
所有的注解，推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`

常用的字段验证的注解
- @NotEmpty 被注释的字符串的不能为 null 也不能为空
- @NotBlank 被注释的字符串非 null，并且必须包含一个非空白字符
- @Null 被注释的元素必须为 null
- @NotNull 被注释的元素必须不为 null
- @AssertTrue 被注释的元素必须为 true
- @AssertFalse 被注释的元素必须为 false
- @Pattern(regex=,flag=)被注释的元素必须符合指定的正则表达式
- @Email 被注释的元素必须是 Email 格式。
- @Min(value)被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- @Max(value)被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- @DecimalMin(value)被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- @DecimalMax(value) 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- @Size(max=, min=)被注释的元素的大小必须在指定的范围内
- @Digits(integer, fraction)被注释的元素必须是一个数字，其值必须在可接受的范围内
- @Past被注释的元素必须是一个过去的日期
- @Future 被注释的元素必须是一个将来的日期

验证请求体(RequestBody)
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;

}

@RestController
@RequestMapping("/api")
public class PersonController {
    // 需要验证的参数上加上了@Valid注解，如果验证失败，它将抛出MethodArgumentNotValidException
    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

验证请求参数(Path Variables 和 Request Parameters)
```java
@RestController
@RequestMapping("/api")
@Validated // 这个参数可以告诉 Spring 去校验方法参数
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }
}
```

### 全局处理 Controller 层异常
- `@ControllerAdvice`: 注解定义全局异常处理类
- `@ExceptionHandler`: 注解声明异常处理方法

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       //......
    }
}
```

### JPA 相关
暂时跳过，现在多使用MyBatis，很少用Bean和表结构对应的强结构的设计

### Json 数据处理
`@JsonIgnoreProperties` 作用在类上用于过滤掉特定字段不返回或者不解析
```java
//生成json时将userRoles属性过滤
@JsonIgnoreProperties({"userRoles"})
public class User {
    private String userName;
    private String fullName;
    private String password;
    private List<UserRole> userRoles = new ArrayList<>();
}
```

`@JsonIgnore`一般用于类的属性上，作用和上面的`@JsonIgnoreProperties` 一样
```java
public class User {
    private String userName;
    private String fullName;
    private String password;
   //生成json时将userRoles属性过滤
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```

格式化 json 数据: `@JsonForma`t一般用来格式化 json 数据
```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone="GMT")
private Date date;
```

扁平化对象
```java
@Getter
@Setter
@ToString
public class Account {
    private Location location;
    private PersonInfo personInfo;

  @Getter
  @Setter
  @ToString
  public static class Location {
     private String provinceName;
     private String countyName;
  }
  @Getter
  @Setter
  @ToString
  public static class PersonInfo {
    private String userName;
    private String fullName;
  }
}
```

未扁平化之前
```json
{
    "location": {
        "provinceName":"湖北",
        "countyName":"武汉"
    },
    "personInfo": {
        "userName": "coder1234",
        "fullName": "shaungkou"
    }
}
```

使用@JsonUnwrapped 扁平对象之后
```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;
    //......
}
```

```json
{
  "provinceName":"湖北",
  "countyName":"武汉",
  "userName": "coder1234",
  "fullName": "shaungkou"
}
```

### 测试相关
```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@ActiveProfiles("test") // 一般作用于测试类上， 用于声明生效的 Spring 配置文件
@Slf4j
public abstract class TestBase {
  //......
}
```

```java
@Test //声明一个方法为测试方法
@Transactional //被声明的测试方法的数据会回滚，避免污染测试数据
@WithMockUser(username = "user-id-18163138155", authorities = "ROLE_TEACHER") //Spring Security 提供的，用来模拟一个真实用户，并且可以赋予权限
void should_import_student_success() throws Exception {
    //......
}
```

---

# 微服务

常用网关框架: Nginx, Zuul, Gateway

Spring Cloud
1. Spring Cloud Eureka: 注册中心 微服务注册与发现
2. Spring Cloud Zuul: 服务网关 网络服务的入口，所有网络请求必须通过网关转发到具体的微服务上
  - 统一管理微服务请求
  - 权限控制
  - 负载均衡
  - 路由转发
  - 监控
  - 安全控制: 黑名单、白名单
3. Spring Cloud Ribbon: 客户端负载均衡
4. Spring Cloud Feign: 声明性web客户端
5. Spring Cloud Hystrix: 断路器
6. Spring Cloud Config: 分布式统一配置管理

Dubbo
1. zookeeper: 注册中心 微服务注册与发现

## 全家桶套装

### Spring Cloud Alibaba

- Nacos	动态服务发现、配置管理
- Sentinel	流量控制、熔断降级、系统负载保护
- Seata	高性能微服务分布式事务
- RocketMQ	高可用分布式消息系统
- Zuul+Oauth2	分布式网关鉴权系统
- SkyWalking	分布式链路追踪与监控系统

组件：
- Sentinel：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- RocketMQ：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- Dubbo：Apache Dubbo™ 是一款高性能 Java RPC 框架。
- Seata：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- Alibaba Cloud OSS: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- Alibaba Cloud SchedulerX: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- Alibaba Cloud SMS: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

功能：
- 服务限流降级：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- 服务注册与发现：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- 分布式配置管理：支持分布式系统中的外部化配置，配置更改时自动刷新。
消息驱动能力：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- 分布式事务：使用 `@GlobalTransactional` 注解， 高效并且对业务零侵入地解决分布式事务问题。
- 阿里云对象存储：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- 分布式任务调度：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- 阿里云短信服务：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。


## Eureka 注册中心
- 什么是Eureka的自我保护模式？
- 默认情况下，如果 Eureka Service 在一段时间内没有收到某个微服务的心跳，Eureka Service 会进入自我保护模式，在该模式下会保护服务注册表中的信息，不会立刻删除该微服务的信息，当网络故障恢复后 Eureka Service 会自动退出自我保护模式。反之，若一直没有收到该微服务的心跳，则会在一段时间后删除该微服务的信息。

- DiscoveryClient的作用
- 可以从注册中心根据微服务名获取注册的微服务信息

## RPC(Remote Procedure Call)
远程过程调用: RPC 可以帮助我们调用远程计算机上某个服务的方法，这个过程就像调用本地方法一样简单

- 客户端(服务消费端): 调用远程方法的一端。
- 客户端 Stub(桩): 这其实就是一代理类。代理类主要做的事情很简单，就是把你调用方法、类、方法参数等信息传递到服务端。
- 网络传输: 网络传输就是你要把你调用的方法的信息比如说参数啊这些东西传输到服务端，然后服务端执行完之后再把返回结果通过网络传输给你传输回来。网络传输的实现方式有很多种比如最近基本的 Socket或者性能以及封装更加优秀的 Netty(推荐)。
- 服务端 Stub(桩): 这个桩就不是代理类了。我觉得理解为桩实际不太好，大家注意一下就好。这里的服务端 Stub 实际指的就是接收到客户端执行方法的请求后，去指定对应的方法然后返回结果给客户端的类。
- 服务端(服务提供端): 提供远程方法的一端。

1. 服务消费端(client)以本地调用的方式调用远程服务；
2. 客户端 Stub(client stub) 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体(序列化)：RpcRequest；
3. 客户端 Stub(client stub) 找到远程服务的地址，并将消息发送到服务提供端；
4. 服务端 Stub(桩)收到消息将消息反序列化为Java对象: RpcRequest；
5. 服务端 Stub(桩)根据RpcRequest中的类、方法、方法参数等信息调用本地的方法；
6. 服务端 Stub(桩)得到方法执行结果并将组装成能够进行网络传输的消息体：RpcResponse(序列化)发送至消费方；
7. 客户端 Stub(client stub)接收到消息并将消息反序列化为Java对象:RpcResponse ，这样也就得到了最终结果。over!

## Feign调用

## Spring Cloud Gateway

### 降级
服务降级:系统有限的资源的合理协调
- 概念：服务降级一般是指在服务器压力剧增的时候，根据实际业务使用情况以及流量，对一些服务和页面有策略的不处理或者用一种简单的方式进行处理，从而释放服务器资源的资源以保证核心业务的正常高效运行。
- 原因： 服务器的资源是有限的，而请求是无限的。在用户使用即并发高峰期，会影响整体服务的性能，严重的话会导致宕机，以至于某些重要服务不可用。故高峰期为了保证核心功能服务的可用性，就需要对某些服务降级处理。可以理解为舍小保大
- 应用场景： 多用于微服务架构中，一般当**整个微服务**架构整体的负载超出了预设的上限阈值（和服务器的配置性能有关系），或者即将到来的流量预计会超过预设的阈值时（比如双11、6.18等活动或者秒杀活动）
- 服务降级是从**整个系统**的负荷情况出发和考虑的，对某些负荷会比较高的情况，为了预防某些功能（业务场景）出现负荷过载或者响应慢的情况，在其内部暂时舍弃对一些非核心的接口和数据的请求，而直接返回一个提前准备好的 fallback（退路）错误处理信息。这样，虽然提供的是一个有损的服务，但却保证了整个系统的稳定性和可用性。
- 需要考虑的问题：
  - 区分哪些服务为核心，哪些为非核心
  - 降级策略（处理方式，一般指如何给用户友好的提示或者操作）
  - 自动降级还是手动降

降级方式: 
1. 熔断降级(不可用): 熔断逻辑是这样的，A服务调用B服务，失败次数达到一定
2. 超时降级
3. 限流降级

#### 熔断 Hystrix
在 Spring Cloud 框架里，熔断机制通过 Hystrix 实现。Hystrix 会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。

服务熔断：应对雪崩效应的链路自我保护机制。可看作降级的特殊情况
- 概念：应对微服务雪崩效应的一种链路保护机制，类似股市、保险丝
- 原因： 微服务之间的数据交互是通过远程调用来完成的。服务A调用服务，服务B调用服务c，某一时间链路上对服务C的调用响应时间过长或者服务C不可用，随着时间的增长，对服务C的调用也越来越多，然后服务C崩溃了，但是链路调用还在，对服务B的调用也在持续增多，然后服务B崩溃，随之A也崩溃，导致雪崩效应
- 服务熔断是应对雪崩效应的一种微服务链路保护机制。例如在高压电路中，如果某个地方的电压过高，熔断器就会熔断，对电路进行保护。同样，在微服务架构中，熔断机制也是起着类似的作用。当调用链路的某个微服务不可用或者响应时间太长时，会进行服务熔断，不再有该节点微服务的调用，快速返回错误的响应信息。当检测到该节点微服务调用响应正常后，恢复调用链路。
- 服务熔断的作用类似于我们家用的保险丝，当某服务出现不可用或响应超时的情况时，为了防止整个系统出现雪崩，暂时停止对该服务的调用。


## 配置中心 & bus

## 微服务治理

---

# Database

## SQL

### 常见编写SQL性能建议
- 充分利用表上已经存在的索引
- 避免使用双%号的查询条件。如：`a like '%123%'`，（如果无前置%,只有后置%，是可以用到列上的索引的）
- 禁止使用 `SELECT *` 必须使用 `SELECT <字段列表>` 查询
  - `SELECT *` 消耗更多的 CPU 和 IO 以网络带宽资源
  - `SELECT <字段列表>` 可减少表结构变更带来的影响
  - `SELECT *` 无法使用覆盖索引
- 禁止使用不含字段列表的 INSERT 语句
- 建议使用预编译语句进行数据库操作
  - 预编译语句可以重复使用这些计划，减少 SQL 编译所需要的时间，还可以解决动态 SQL 所带来的 SQL 注入的问题。
  - 只传参数，比传递 SQL 语句更高效
  - 相同语句可以一次解析，多次使用，提高处理效率。
- 避免使用子查询，可以把子查询优化为 join 操作
- 避免使用 JOIN 关联太多的表
- 减少同数据库的交互次数
- 对应同一列进行 or 判断时，使用 in 代替 or
- WHERE 从句中禁止对列进行函数转换和计算

### 数据库操作规范
- 超 100 万行的批量写 (UPDATE,DELETE,INSERT) 操作,要分批多次进行操作
  - 大批量操作可能会造成严重的主从延迟
  - 避免产生大事务操作
- 程序连接不同的数据库使用不同的账号，禁止跨库查询
  - 为数据库迁移和分库分表留出余地
  - 降低业务耦合度
  - 避免权限过大而产生的安全风险 

## MySQL

### 存储引擎
MyISAM
- 非聚集索引: 索引文件与数据文件分开; `.MYI`文件存索引, `.MYD`文件存数据
- 底层数据结构: B+Tree 作为索引结构，叶节点的 data 域存放的是数据记录的地址
- 主键: 可以没有
- 辅助索引(Secondary key): 结构上与主索引没有任何区别，辅助索引的 key 可以重复

InnoDB
- 聚集索引: 数据文件本身就是索引文件
- 底层数据结构: B+Tree 作为索引结构，叶节点的 data 域保存了完整的数据记录。索引的 key 是数据表的主键，因此InnoDB 要求表必须有主键
- 主键: 必须有主键。如果没有显式指定，则会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则自动为 InnoDB 表生成一个隐含字段作为主键，类型为长整形。尽量在采用自增字段做表的主键，非单调的主键会造成在插入新记录时数据文件为了维持 B+Tree 的特性而频繁的分裂调整
- 辅助索引(Secondary key): 辅助索引 data 域存储相应记录主键的值而不是地址。这使得辅助索引搜索需要检索两遍索引(回表):首先检索辅助索引获得主键,然后用主键到主索引中检索获得记录。

### 日志
- bin log(归档日志): 逻辑日志，记录内容是语句的原始逻辑。binlog会记录所有涉及更新数据的逻辑规则，并且按顺序写。数据备份、主备、主主、主从都离不开binlog，需要依赖binlog来同步数据，保证数据一致性。
- undo log(回滚日志): 所有事务进行的修改都会先被记录到这个回滚日志，然后再执行其他相关的操作。如果执行过程中遇到异常的话，我们直接利用回滚日志中的信息将数据回滚到修改之前的样子。MVCC的实现依赖：隐藏字段、Read View、undo log。
- redo log(重做日志): 当MySQL实例挂了或者宕机了，重启的时候InnoDB存储引擎会使用rede log日志恢复数据，保证事务的持久性和完整性。定时刷盘写入。

### 索引 & 优化

聚簇(聚集)索引: 索引文件与数据文件在同一个磁盘文件中

最左前缀原则: 
假设某表存在如下索引 `index idx_rpps (region_code,polno,super_id),` 那么在编写查询语句的时候where条件语句中的条件在走索引时按从左到右的顺序查，和你编写的where中的顺序无关，会自动优化，但是和你创建索引时的顺序有关。
```
where region_code = '02' and polno = 'P1' and super_id = '123'
```

`where polno = 'P1' and super_id = '123'`  
如果最左边的字段不出现在where中，后面的字段不管出现几个都是全表扫描 

索引下推: `where region_code = '02' and super_id = '123'`  
如果出现最左边的字段和后面任意字段，则可以走索引，先使用`region_code`走索引查，查到多个都是`02`的数据后再用`super_id = '123'`来筛选，这种情况被成为索引下推 

覆盖索引: `select polno,super_id from info where polno > 'P1'`  
如果select中的字段可以直接在where条件查询的索引树中查询得到那么就不用回表，直接返回索引树中查询到的结果，这种情况被成为索引下推覆盖索引。



### 索引的分类
- 主键索引: primary key设定为主键后，数据库自动建立索引，InnoDB为聚簇索引，主键索引列值不能为空（Null）。
- 唯一索引: 索引列的值必须唯一，但允许有空值（Null），但只允许有一个空值（Null）。
- 复合索引: 一个索引可以包含多个列，多个列共同构成一个复合索引。
- 全文索引: Full Text（MySQL5.7之前，只有MYISAM存储引擎引擎支持全文索引）。全文索引类型为FULLTEXT，在定义索引的列上支持值的全文查找允许在这些索引列中插入重复值和空值。全文索引可以在Char、VarChar 上创建。
- 空间索引: MySQL在5.7之后的版本支持了空间索引，而且支持OpenGIS几何数据模型，MySQL在空间索引这方年遵循OpenGIS几何数据模型规则。
- 前缀索引: 在文本类型为char、varchar、text类列上创建索引时，可以指定索引列的长度，但是数值类型不能指定。

### 索引数据结构 B+Tree
mysql索引数据结构为什么选择B+Tree?
- 二叉排序树: 无平衡机制，插入递增元素，退化成链表
- 红黑树(是一种二叉平衡树): 数据量大时树的深度也会很深
- B Tree: 多叉，从左到右依次递增，叶子节点都在同一层
- B+ Tree: 非叶子节点不存储data，叶子节点包含所有索引字段并用指针从左往右链接成链表
- B-tree的非叶子节点中存储的是真实的数据行，而数据页的大小是16KB固定的，因此相同数据下，B-tree需要更多数据页才能存储数据，数据页增多势必会造成非叶子节点的层级变高，造成更多的磁盘IO，导致性能下降

B+ Tree: 每个节点大小16kb限制，16kb/每个节点大小(8字节索引元素+6字节孩子节点磁盘文件地址指针)=1170个索引
- 索引全部加载到内存，找到后根据磁盘文件地址进行一次磁盘I/O读取对应的数据
- 只用3层的B+ Tree就支持上千万行数据的查找: 树高度等于2时(2阶)，大概可以存两万多条数据；高度等于3时(3阶)，大概可以存两千多万条数据

索引优化
- 限制每张表上的索引数量,建议单张表索引不超过 5 个
- 每个 InnoDB 表必须有个主键
  - InnoDB 是一种索引组织表：数据的存储的逻辑顺序和索引的顺序是相同的。每个表都可以有多个索引，但是表的存储顺序只能有一种。
  - InnoDB 是按照主键索引的顺序来组织表的
  - 不要使用更新频繁的列作为主键
  - 不要使用 UUID,MD5,HASH,字符串列作为主键（无法保证数据的顺序增长），主键建议使用自增 ID 值
- 设置索引字段推荐
  - 出现在 SELECT、UPDATE、DELETE 语句的 WHERE 从句中的列、包含在 ORDER BY、GROUP BY、DISTINCT 中的字段
  - 并不要将符合 1 和 2 中的字段的列都建立一个索引， 通常将 1、2 中的字段建立联合索引效果更好
  - 多表 join 的关联列
- 建立索引的目的是：希望通过索引进行数据查找，减少随机 IO，增加查询性能 ，索引能过滤出越少的数据，则从磁盘中读入的数据也就越少
- 尽量避免使用外键约束

### 锁
InnoDB的锁定模式实际上可以分为四种
1. 共享锁(S)
2. 排他锁(X)
3. 意向共享锁(IS)
4. 意向排他锁(IX)

共享锁(S): `SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE` 获得共享锁，主要用在需要数据依存关系时来确认某行记录是否存在，并确保没有人对这个记录进行UPDATE或者DELETE操作。
排他锁(X): `SELECT * FROM table_name WHERE ... FOR UPDATE` 对于锁定行记录后需要进行更新操作的应用，应该使用`FOR UPDATE`方式获得排他锁。

### 事务
ACID
1. 原子性(Atomicity): 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部都执行，要么都不执行
2. 一致性(Consistency): 指在事务开始之前和事务结束以后，数据不会被破坏，假如A账户给B账户转10块钱，不管成功与否，A和B的总金额是不变的
3. 隔离性(Isolation): 多个事务并发访问时，事务之间是相互隔离的，一个事务不应该被其他事务干扰，多个并发事务之间要相互隔离
4. 持久性(Durability): 表示事务完成提交后，该事务对数据库所作的操作更改，将持久地保存在数据库之中

事务并发存在的问题
1. 脏读: 如果一个事务读取到了另一个未提交事务修改过的数据，我们就称发生了脏读现象
2. 不可重复读: 同一个事务内，前后多次读取，读取到的数据内容不一致
3. 幻读: 如果一个事务先根据某些搜索条件查询出一些记录，在该事务未提交时，另一个事务写入了一些符合那些搜索条件的记录(如insert、delete、update)，就意味着发生了幻读

事务隔离级别
- READ-UNCOMMITTED(读未提交): 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
- READ-COMMITTED(读已提交): 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
- REPEATABLE-READ(可重复读): 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
- SERIALIZABLE(可串行化): 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

InnoDB 存储引擎的默认支持的隔离级别是 REPEATABLE-READ(可重复读)。我们可以通过`SELECT @@tx_isolation;`命令来查看，MySQL 8.0 该命令改为`SELECT @@transaction_isolation;`  
InnoDB 存储引擎在分布式事务的情况下一般会用到 SERIALIZABLE 隔离级别

InnoDB 实现的 REPEATABLE-READ 隔离级别其实是可以解决幻读问题发生的，主要有下面两种情况：
- 快照读: 由 MVCC 机制来保证不出现幻读。
- 当前读: 使用 Next-Key Lock 进行加锁来保证不出现幻读，Next-Key Lock 是行锁(Record Lock)和间隙锁(Gap Lock)的结合，行锁只能锁住已经存在的行，为了避免插入新行，需要依赖间隙锁。

### MVCC(Multi-Version Concurrency Control)多版本并发控制
一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。数据库隔离级别读已提交、可重复读 都是基于MVCC实现的

- 当前读：读取的数据是最新版本，读取数据时还要保证其他并发事务不会修改当前的数据，当前读会对读取的记录加锁。比如：`select …… lock in share mode`（共享锁）、`select …… for update | update | insert | delete`（排他锁）
- 快照读：每一次修改数据，都会在 undo log 中存有快照记录，这里的快照，就是读取undo log中的某一版本的快照。这种方式的优点是可以不用加锁就可以读取到数据，缺点是读取到的数据可能不是最新的版本。一般的查询都是快照读，比如：`select * from t_user where id=1;` 在MVCC中的查询都是快照度

- redo log: 在修改数据的时候，会向 redo log 中记录修改的页内容 作用:在数据库宕机重启后恢复对数据库的操作
- undo log: 记录数据原来的快照 作用:回滚事务和实现MVCC

查询一条记录，基于MVCC，是怎样的流程
1. 获取事务自己的版本号，即事务ID
2. 获取Read View
3. 查询得到的数据，然后Read View中的事务版本号进行比较。
4. 如果不符合Read View的可见性规则， 即就需要Undo log中历史快照;
5. 最后返回符合规则的数据

InnoDB 实现MVCC，是通过 Read View + Undo Log 实现的，Undo Log 保存了历史快照，Read View可见性规则帮助判断当前版本的数据是否可见。
- Read View: 
  - 事务执行SQL语句时，产生的读视图。实际上在innodb中，每个SQL语句执行前都会得到一个Read View
  - 主要是用来做可见性判断的，即判断当前事务可见哪个版本的数据
- Undo Log:
  - 回滚日志，用于记录数据被修改前的信息。在表记录修改之前，会先把数据拷贝到undo log里，如果事务回滚，即可以通过undo log来还原数据
  - 当delete一条记录时，undo log 中会记录一条对应的insert记录
  - 用途: 
    1. 事务回滚时，保证原子性和一致性
    2. 用于MVCC快照读
- 快照读: 读取的是记录数据的可见版本（有旧的版本）。不加锁,普通的select语句都是快照读
- 当前读: 读取的是记录数据的最新版本，显式加锁的都是当前读

### 慢查询优化

在查询SQL前加`EXPLAIN`命令可以让MySQL打印分析结果，例如:
```sql
mysql> EXPLAIN SELECT * from vio_basic_domain_info where app_name like '%陈哈哈%' ;
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+---------+----------+-------------+
| id | select_type | table                 | partitions | type | possible_keys | key  | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | vio_basic_domain_info | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 1377809 |    11.11 | Using where |
+----+-------------+-----------------------+------------+------+---------------+------+---------+------+---------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

根据表信息可知：该SQL没有用到字段app_name上的索引，查询类型是全表扫描，扫描行数137w  
修改SQL，改为字符串前缀查询
```sql
mysql> EXPLAIN SELECT * from vio_basic_domain_info where app_name like '陈哈哈%' ;
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
| id | select_type | table                 | partitions | type  | possible_keys | key          | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | vio_basic_domain_info | NULL       | range | idx_app_name  | idx_app_name | 515     | NULL |  141 |   100.00 | Using index condition |
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
这条SQL使用到覆盖索引时，SQL如下：查询耗时：0.091s，查到141条数据

#### 慢查询分析常用到的属性
- `type`: 对表访问方式，表示MySQL在表中找到所需行的方式。ALL、index、range、 ref、eq_ref、const、system、NULL(从左到右，性能从低到高)
  - ALL: (Full Table Scan) MySQL将遍历全表以找到匹配的行，常说的全表扫描
  - index: (Full Index Scan) index与ALL区别为index类型只遍历索引树
  - range: 只检索给定范围的行，使用一个索引来选择行
- `key`: key列显示了SQL实际使用索引，通常是possible_keys列中的索引之一，MySQL优化器一般会通过计算扫描行数来选择更适合的索引，如果没有选择索引，则返回NULL
  - 强制使用一个索引: FORCE INDEX (index_name)、USE INDEX (index_name)
  - 强制忽略一个索引: IGNORE INDEX (index_name) 
- `rows`: 估计为了找到所需的行而要读取（扫描）的行数，可能不精确
- `Extra`: 额外信息
  - Using index: 通过索引查找就能直接找到符合条件的数据，无须回表
  - Using where: MySQL服务器将在存储引擎检索行后再进行过滤；即没有用到索引，回表查询
  - Using temporary: MySQL在对查询结果排序时会使用一个临时表
  - Using filesort: 对结果使用一个外部索引排序，而不是按索引次序从表里读取行
  - Using index condition: 查询的列不全在索引中，where条件中是一个前导列的范围
  - Using where;Using index: 查询的列被索引覆盖，并且where筛选条件是索引列之一，但不是索引的前导列或出现了其他影响直接使用索引的情况

#### 处理分页慢查询的方式一般有以下几种
思路一：构造覆盖索引: 通过修改SQL，使用上覆盖索引，比如我需要只查询表中的app_name、createTime等少量字段，那么我秩序在app_name、createTime字段设置联合索引，即可实现覆盖索引，无需全表扫描
```sql
mysql> EXPLAIN SELECT app_name,createTime from vio_basic_domain_info LIMIT 1000000,10;
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+---------+----------+-------------+
| id | select_type | table                 | partitions | type  | possible_keys | key          | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+---------+----------+-------------+
|  1 | SIMPLE      | vio_basic_domain_info | NULL       | index | NULL          | idx_app_name | 515     | NULL | 1377809 |   100.00 | Using index |
+----+-------------+-----------------------+------------+-------+---------------+--------------+---------+------+---------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

思路二：优化offset: 法用上覆盖索引，那么重点是想办法快速过滤掉前100w条数据。我们可以利用自增主键有序的条件，先查询出第1000001条数据的id值，再往后查10行；适用于主键id自增的场景
```sql
mysql> EXPLAIN SELECT * from vio_basic_domain_info where id >=(SELECT id from vio_basic_domain_info ORDER BY id limit 1000000,1) limit 10;
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
| id | select_type | table                 | partitions | type  | possible_keys | key     | key_len | ref  | rows    | filtered | Extra       |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
|  1 | PRIMARY     | vio_basic_domain_info | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL |      10 |   100.00 | Using where |
|  2 | SUBQUERY    | vio_basic_domain_info | NULL       | index | NULL          | PRIMARY | 8       | NULL | 1000001 |   100.00 | Using index |
+----+-------------+-----------------------+------------+-------+---------------+---------+---------+------+---------+----------+-------------+
2 rows in set, 1 warning (0.40 sec)
```

思路三：延迟关联: 延迟关联适用于数量级较大的表，用到了覆盖索引+延迟关联查询，相当于先只查询id列，利用覆盖索引快速查到该页的10条数据id，然后再把返回的10条id拿到表中通过主键索引二次查询
```sql
mysql> EXPLAIN SELECT * from vio_basic_domain_info inner join (select id from vio_basic_domain_info order by id limit 1000000,10) as myNew using(id);
+----+-------------+-----------------------+------------+--------+---------------+---------+---------+----------+---------+----------+-------------+
| id | select_type | table                 | partitions | type   | possible_keys | key     | key_len | ref      | rows    | filtered | Extra       |
+----+-------------+-----------------------+------------+--------+---------------+---------+---------+----------+---------+----------+-------------+
|  1 | PRIMARY     | <derived2>            | NULL       | ALL    | NULL          | NULL    | NULL    | NULL     | 1000010 |   100.00 | NULL        |
|  1 | PRIMARY     | vio_basic_domain_info | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | myNew.id |       1 |   100.00 | NULL        |
|  2 | DERIVED     | vio_basic_domain_info | NULL       | index  | NULL          | PRIMARY | 8       | NULL     | 1000010 |   100.00 | Using index |
+----+-------------+-----------------------+------------+--------+---------------+---------+---------+----------+---------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
```

#### 排查索引没起作用的情况
模糊查询尽量避免用通配符`%`开头，会导致数据库引擎放弃索引进行全表扫描

### 分库分表
使用开源组件: ShardingSphere

- 垂直分表: 将一个表按照字段分成多个表，每个表存储其中一部分字段
- 垂直分库: 
- 水平分表: 水平分表是在同一个数据库内，把同一个表的数据按一定规则拆到多个表中
- 水平分库: 

总结
- 垂直分表：热门数据、冷门数据分开存储，大字段放在冷门数据表中。
- 垂直分库：按业务拆分，放到不同的库中，这些库分别部署在不同的服务器，解决单一服务器性能的瓶颈，同时提升整体架构的业务清晰度。
- 水平分表：解决单一表数据量过大的问题
- 水平分库：把一个表的数据分别分到不同的库中，这些库分别部署在不同的服务器，解决单一服务器数据量过大的问题

#### 垂直分表
性能提升:
1. 为了避免IO争抢并减少锁表的几率；
2. 充分发挥热门数据的操作效率，热门字段和冷门字段分开存储，比如一个产品基本信息表、一个产品详细信息表，大字段一定要放在冷门字段的表中。

为什么大字段IO效率低？
1. 数据本身长度过长，需要更长的读取时间；
2. 跨页，页是数据库存储基本单位，很多查找及定位操作都是以页为单位，单页内的数据行越多数据库整体性能越好，而大字段占用空间大，单页存储数据少，因此IO效率低；
3. 数据以行为单位将数据加载到内存中，如果字段长度短，内存就可以加载更多的数据，减少磁盘IO，从而提高数据库性能；

#### 垂直分库
垂直分表只解决了单一表数据量大的问题，但没有将表分布到不同的服务器上，因此每张表还是竞争同一个物理机的CPU、内存、网络IO、磁盘。

性能提升:
1. 解决业务层面的耦合，业务清晰
2. 能对不同业务的数据进行分级管理、维护、监控、扩展等
3. 高并发场景下，垂直分库一定程序上提升IO、减少数据库连接数、降低单机硬件资源的瓶颈。

#### 水平分表
性能提升：
1. 优化单一表数据量过大而产生的性能问题
2. 避免IO争抢并减少锁表的几率
3. 单一数据库内的水平分表，解决了单一表数据量过大的问题，分出来的小表只包含一部分数据，从而使单表查询的速度更快，效率更好。

水平分表的方式:
1. Hash取模分表: 数据库分表一般都是采用这种方式，比如一个position表，根据positionId%4，并按照结果分成4张表。
  - 优点：数据分片较为平均，不容易出现热点和并发访问的瓶颈。
  - 缺点：容易产生跨分片查询的复杂问题。
2. 数值Range分表: 按照时间区间或ID区间进行切分。
  - 优点：单表大小可控，易于扩展，有效避免跨分片查询的问题
  - 缺点：热点数据成为性能瓶颈。例如按时间分片，有些分片存储在最近时间段的表内，可能被频繁的读写操作，而历史数据表则访问较少。
2. 一致性Hash算法
  - 较为复杂，小编暂时不做介绍，有兴趣的可以自行百度。

#### 水平分库
性能提升: 
1. 解决了单库数据量大，高并发的瓶颈
2. 提高了系统的稳定性和可用性

何时使用: 当一个应用难以再进行垂直切分，或垂直切分后数据量行数巨大，存在单库读写存储的性能瓶颈，这时候就可以考虑使用水平分库了。

使用弊端: 但水平分库的弊端也很明显，需要确定你所需要的数据在哪一个库中，因此大大提高了系统的复杂度




---

# 分布式

## Redis

redis单线程为什么快?
1. 纯内存操作: 数据存放在内存中，内存的响应时间大约是100纳秒
2. 单线程: Redis是基于内存的操作，CPU不是Redis的瓶颈。Redis的瓶颈最有可能是机器内存或者网络带宽，同时避免了线程切换和竞态产生的消耗 
3. 非阻塞I/O: Redis采用epoll做为I/O多路复用技术的实现 ，再加上Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为了时间，不在I/O上浪费过多的时间
4. 全局哈希表: 
5. 客户端调服务端:
  1. 发送命令
  2. 执行命令: 每一条到达服务端的命令不会立刻执行，所有的命令都会进入一个队列中，然后逐个被执行。并且多个客户端发送的命令的执行顺序是不确定的。但是可以确定的是不会有两条命令被同时执行，不会产生并发问题
  3. 返回结果

Redis数据结构底层实现
- String: Simple dynamic string(SDS)
  - `buf[]`字节数组，用于保存字符串, `len`保存字符串的长度, `free`buf 数组中未使用字节的数量
  - 优点: 不用担心字符串变更造成的内存溢出问题
- 链表: 双向链表上扩展了头、尾节点、元素数等属性
  - 优点: 可以直接获得头、尾节点
- 字典(Hash): 数组+链表的基础上，进行了一些rehash优化
  - 采用链地址法来处理冲突，然后它没有使用红黑树优化
  - 哈希表节点采用单链表结构
  - rehash优化: 哈希表保存的键值对会逐渐地增多或者减少， 为了让哈希表的负载因子(load factor)维持在一个合理的范围之内， 程序需要对哈希表的大小进行相应的扩展或者收缩
- 有序集合:
  - 底层实现为跳跃表: 跳表其实就是一种可以进行二分查找的有序链表。跳表在原有的有序链表上面增加了多级索引，通过索引来实现快速查找。首先在最高级索引上查找最后一个小于当前查找元素的位置，然后再跳到次高级索引继续查找，直到跳到最底层为止，这时候以及十分接近要查找的元素的位置了(如果查找元素存在的话)。由于根据索引可以一次跳过多个元素，所以跳查找的查找速度也就变快了

### RDB(Redis DataBase)
把当前数据生成快照保存在硬盘上。
RDB 是 Redis 默认的持久化方案。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个dump.rdb文件。Redis 重启会通过加载dump.rdb文件恢复数据。

### AOF(Append Only File)
记录每次对数据的操作到硬盘上。
Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个写操作，并追加到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

### 分布式锁
- 使用`setnx`命令(SET if Not eXists)实现: 在Redis中，`setnx`命令的作用是，如果key不存在，则设置key-value并返回true，如果key存在，则不做任何操作并返回false; 利用这一机制在获取锁为设置key，设置成功返回true表示获取到锁了，执行完业务逻辑之后删除掉设置的key即可
- 可以在使用`setnx`命令的同时设置超时时间，假设值key的那个服务器挂了，没有删除key，那么key会在超时后自动删除以释放锁
- 生产案例: 抢单，使用`setnx`实现分布式锁，锁机构库update，抢到保单的线程抢锁来update机构库对应保单数据的督导工号。没有设置过期时间，一个微服务实例在运行完`setnx db-update superId`之后没有运行`del key`来删除key，即设置了锁之后挂了，没有释放锁，导致一个机构库的update被锁住，无法抢单。那么为什么要设置key过期时间呢？如果请求执行因为某些原因意外退出了，导致创建了锁但是没有删除锁，那么这个锁将一直存在（redis不设置key的过期时间，默认是永久的），以至于一直处于加锁状态。

epoll: linux内核下的一个高效的处理大批量的文件操作符的一个实现

### 缓存雪崩、缓存穿透、缓存击穿
- 缓存雪崩: 缓存同一时间大面积失效，后面的请求都会落到数据库上，造成数据库短时间内无法承受大量请求而崩溃
  - 解决方案: 
    - 每个key的失效时间加个随机值，避免同一时间大量的key失效
    - 集群部署，可以将热点数据分布到各个不同的库
- 缓存穿透: 大量请求的key不存在于缓存中，例如某个黑客制造缓存中不存在的key发起大量请求，导致大量请求落到数据库
  - 解决方案:
    - 入参校验，将不合法的参数直接拦截
    - 缓存和数据库都查不到某个key的数据，就将key写入到redis，value为null，并设置过期时间，避免下次请求落到数据库上
    - 布隆过滤器: 布隆过滤器可以非常方便的判定一个给定的数据是否存在与海量数据中.可以将所有可能存在的请求的值存到布隆过滤器，当请求过来先判断用户发来的请求是否存在于布隆过滤器，不存在就直接拦截 
- 缓存击穿: 一个Key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key失效瞬间，持续的大并发就穿破缓存，直接请求到数据库
  - 解决方案:
    - 定时任务主动刷新缓存设计
    - jvm缓存+redis缓存的多级缓存: 两个缓存同时失效概率低，JVM缓存时间设随机值,比如 10秒~30秒
    - 利用redis实现的分布式锁setnx 来实现互斥的数据库操作: 如果缓存中没有则拿到锁的去查数据库
      1. 如果缓存没有,则尝试获取分布式锁(有超时设置)
      2. 如果没有拿到锁,则阻塞当前线程,n秒,之后再次尝试获取分布式锁(自旋)
      3. 拿到锁之后检查数据是否已经被其他线程放到redis缓存中,如果redis缓存已有,直接返回redis中的数据,释放分布式锁
      4. 如果缓存没有被刷新,则查数据库
      5. 将数据库查询的结果保存到redis缓存中
      6. 返回查询结果

### CAP 理论
起源于 2000 年，由加州大学伯克利分校的 Eric Brewer 教授在分布式计算原理研讨会（PODC）上提出，因此 CAP 定理又被称作 布鲁尔定理（Brewer’s theorem）

CAP: 分布式系统理论上不可能选择 CA 架构，只能选择 CP 或者 AP 架构
1. Consistency(一致性): 所有节点访问同一份最新的数据副本
2. Availability(可用性): 非故障的节点在合理的时间内返回合理的响应（不是错误或者超时的响应）
3. Partition Tolerance(分区容错性): 分布式系统出现网络分区的时候，仍然能够对外提供服务
  - 网络分区: 分布式系统中，多个节点之前的网络本来是连通的，但是因为某些故障（比如部分节点网络出了问题）某些节点之间不连通了，整个网络就分成了几块区域，这就叫 网络分区

- ZooKeeper 保证的是 CP。 任何时刻对 ZooKeeper 的读请求都能得到一致性的结果，但是， ZooKeeper 不保证每次请求的可用性比如在 Leader 选举过程中或者半数以上的机器不可用的时候服务就是不可用的。
- Eureka 保证的则是 AP。 Eureka 在设计的时候就是优先保证 A （可用性）。在 Eureka 中不存在什么 Leader 节点，每个节点都是一样的、平等的。因此 Eureka 不会像 ZooKeeper 那样出现选举过程中或者半数以上的机器不可用的时候服务就是不可用的情况。 Eureka 保证即使大部分节点挂掉也不会影响正常提供服务，只要有一个节点是可用的就行了。只不过这个节点上的数据可能并不是最新的。
- Nacos 不仅支持 CP 也支持 AP。

### BASE 理论
BASE 理论起源于 2008 年， 由 eBay 的架构师 Dan Pritchett 在 ACM 上发表。

BASE: BASE 理论是对 CAP 中一致性 C 和可用性 A 权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是基于 CAP 定理逐步演化而来的，它大大降低了我们对系统的要求
1. Basically Available(基本可用): 分布式系统在出现不可预知故障的时候，允许损失部分可用性。但是，这绝不等价于系统不可用
  - 损失部分可用性: 1.响应时间上的损失; 2.系统功能上的损失(eg:系统的部分非核心功能无法使用)
2. Soft-state(软状态): 软状态指允许系统中的数据存在中间状态（CAP 理论中的数据不一致），并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据同步的过程存在延时
3. Eventually Consistent(最终一致性): 最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性

分布式一致性的 3 种级别: 
1. 强一致性: 系统写入了什么，读出来的就是什么
2. 弱一致性: 不一定可以读取到最新写入的值，也不保证多少时间之后读取到的数据是最新的，只是会尽量保证某个时刻达到数据一致的状态
3. 最终一致性: 弱一致性的升级版，系统会保证在一定时间内达到数据一致的状态

实现最终一致性的具体方式:
- 读时修复: 在读取数据时，检测数据的不一致，进行修复。比如 Cassandra 的 Read Repair 实现，具体来说，在向 Cassandra 系统查询数据的时候，如果检测到不同节点的副本数据不一致，系统就自动修复数据
- 写时修复: 在写入数据，检测数据的不一致时，进行修复。比如 Cassandra 的 Hinted Handoff 实现。具体来说，Cassandra 集群的节点之间远程写数据的时候，如果写失败 就将数据缓存下来，然后定时重传，修复数据的不一致性
- 异步修复: 这个是最常用的方式，通过定时对账检测副本数据的一致性，并修复

BASE 理论本质上是对 CAP 的延伸和补充，更具体地说，是对 CAP 中 AP 方案的一个补充:  
如果系统没有发生“分区”的话，节点间的网络连接通信正常的话，也就不存在 P 了。这个时候，我们就可以同时保证 C 和 A 了。因此，如果系统发生“分区”，我们要考虑选择 CP 还是 AP。如果系统没有发生“分区”的话，我们要思考如何保证 CA 。
因此，AP 方案只是在系统发生分区的时候放弃一致性，而不是永远放弃一致性。在分区故障恢复后，系统应该达到最终一致性。这一点其实就是 BASE 理论延伸的地方。

### 缓存一致性
如果是要求强一致性，那就不能使用缓存，因为保证不了强一致性，只能保证最终一致性
TODO:


## SpringCloud-Eureka

## Kafka

- Partition: topic中的数据分割为一个或多个partition。每个topic至少有一个partition。每个partition中的数据使用多个segment文件存储。

消息传递模式
- 点对点传递模式: 消息持久化到一个队列中，一个或多个消费者消费队列中的数据，一条消息只能被消费一次
- 发布-订阅模式: 消息被持久化到一个topic中，消费者可以订阅一个或多个topic，消费者可以消费该topic中所有的数据，同一条数据可以被多个消费者消费，数据被消费后不会立马删除

RocketMQ与Kafka的区别
1. rocketMQ的NameServer和kafka的zookeeper对比
  - kafka具备选举功能: Master/Slave的选举，有2步
    1. 先通过ZK在所有机器中，选举出一个KafkaController
    2. 再由这个Controller，决定每个partition的Master是谁，Slave是谁
  - 因为有了选举功能，所以kafka某个partition的master挂了，该partition对应的某个slave会升级为主对外提供服务
  - rocketMQ不具备选举，Master/Slave的角色也是固定的。当一个Master挂了之后，你可以写到其他Master上，但不能让一个Slave切换成Master
  - rocketMq的所有broker节点的角色都是一样，上面分配的topic和对应的queue的数量也是一样的，Mq只能保证当一个broker挂了，把原本写到这个broker的请求迁移到其他broker上面
2. kafka为什么比RocketMQ有更大的吞吐量
  - kafka在消息存储过程中会根据topic和partition的数量创建物理文件，也就是说我们创建一个topic并指定了3个partition，那么就会有3个物理文件目录，也就说说partition的数量和对应的物理文件是一一对应的
  - RocketMQ在消息存储方式是采用commitLog，RocketMQ的queue的数量其实是在consumeQueue里面体现的，在真正存储消息的commitLog其实就只有一个物理文件
  - kafka的多文件并发写入 VS RocketMQ的单文件写入，性能差异kafka完胜可想而知
  - kafka的大量文件存储会导致一个问题，也就说在partition特别多的时候，磁盘的访问会发生很大的瓶颈，毕竟单个文件看着是append操作，但是多个文件之间必然会导致磁盘的寻道

## RocketMQ

组成:
1. Name Server: 名称服务充当路由消息的提供者，可集群部署，节点之间无任何信息同步，提供命名服务，更新和发现 Broker 服务
2. Producer(生产者): 
3. Broker: 消息中转角色，负责存储消息，转发消息。Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker
4. Consumer(消费者): 

- Topic: 表示消息的第一级类型，一条消息必须有一个Topic
- Queue: Queue是Topic在一个Broker上的分片，在分片基础上再等分为若干份（可指定份数）后的其中一份，是负载均衡过程中资源分配的基本单元
- tags: Tags是Topic下的次级消息类型/二级类型，可以在同一个Topic下基于Tags进行消息过滤。Tags的过滤需要经过两次比对，首先会在Broker端通过Tag hashcode进行一次比对过滤，匹配成功传到consumer端后再对具体Tags进行比对，以防止Tag hashcode重复的情况
- commitLog: 用于存储消息的文件。顺序写入，随机读写。消息只要被写入 commitlog 那么该消息就不会丢失。
- ConsumeQueue: ConsumeQueue中并不需要存储消息的内容，而存储的是消息在CommitLog中的offset。通过broker保存的offset可以在ConsumeQueue中获取消息，从而快速的定位到commitLog的消息位置

集群: 推荐多主多从
- 多个master节点多个slave节点，一个master节点配一个slave节点，slave是master节点的备份
- slave节点不接受生产者的消息，生产者的消息发给master节点
- 消费者一般从master节点消费消息
- 若master节点宕机，则消费者从对应的slave节点消费消息，注意：这里slave即使master宕机也不会升级，依然是slave节点
- 同步复制: 生产者发送同步消息需主节点和从节点都写入文件或内存(异步刷盘是内存，一般选择异步刷盘)之后才会返回确认信息给生产者
- 异步刷盘(高性能): 注意这里因为消息同时存在于主节点和从节点所以这里可以采用异步刷盘，丢失的概率不大

- Topic: 表示消息的第一级类型，一条消息必须有一个Topic
- Queue: Queue是Topic在一个Broker上的分片，在分片基础上再等分为若干份（可指定份数）后的其中一份，是负载均衡过程中资源分配的基本单元
- tags: Tags是Topic下的次级消息类型/二级类型，可以在同一个Topic下基于Tags进行消息过滤。Tags的过滤需要经过两次比对，首先会在Broker端通过Tag hashcode进行一次比对过滤，匹配成功传到consumer端后再对具体Tags进行比对，以防止Tag hashcode重复的情况
- commitLog: 用于存储消息的文件。顺序写入，随机读写。消息只要被写入 commitLog 那么该消息就不会丢失。
- ConsumeQueue: ConsumeQueue中并不需要存储消息的内容，而存储的是消息在CommitLog中的offset。通过broker保存的offset可以在ConsumeQueue中获取消息，从而快速的定位到commitLog的消息位置

消息类型
- 同步消息: 消息发送方发出数据后，生产者会阻塞直到MQ服务方发回响应消息，表示已经写入数据到queue里了
- 异步消息: MQ 的异步发送，需要用户实现异步发送回调接口(SendCallback)，在执行消息的异步发送时，应用不需要等待服务器响应即可直接返回，通过回调接口接收服务器响应
- 单向(one-way)消息: 只负责发送消息，不等待服务器回应且没有回调函数触发，即只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别

消息丢失分析:
- 生产者发送时丢失: 同步复制+重试
- RocketMQ自身丢失: 主从架构+持久化
- 消费者消费消息丢失: 重试+死信队列

消息重复消费问题:
- MVCC(Multi-Version Concurrency Control多版本并发控制): 生产者发送到queue的消息需要带上这个版本号，消费者在执行业务逻辑的同时带上版本号，sql的update语句的where带上version号保证语句的幂等性。
  - 缺点: 这意味着生产者在发送消息之前就需要查表拿到最新的版本号，增加了生产者和消费者的耦合度
- 去重表方案: 每个消息带唯一id，然后消费者维护消息表，id设成唯一，消费消息的同时insert这张表，如果抛异常就把异常吃了直接返回，后续业务逻辑不继续进行了

顺序消息消费问题:
1. 一个topic对应一个queue，这样需要顺序消费的消息就在同一条queue里
2. 重试参数改成0

---

# 高并发

## 缓存、降级 和 限流 被称为高并发、分布式系统的三驾马车


## 一致性哈希算法
- 假设你需要对文件进行缓存，你有缓存集群，对每个文件根据哈希对服务器数量取模，将每一张文件映射到一台缓存服务器上面。缺点: 如果你要增加一台服务器，那么需要对所有的文件都重新计算一遍，这样就会导致缓存雪崩，全部缓存到文件失效
- 为解决这个问题可以采用一致性哈希算法: 将服务器映射到一个环上面，然后根据哈希值取模，将文件映射到环上某个位置，然后这个文件存储的服务器就是映射位置顺时针遇到的第一个服务器，这样如果增加一台服务器也只会使得环上的一部分文件缓存失效，避免了缓存雪崩
- 缺点: 有时候可能会出现缓存不均匀的情况，即服务器映射到环上的位置是不均匀的，可以通过增加虚拟节点解决，文件顺时针遇到虚拟节点就放到虚拟节点对应的那个真实节点对应的服务器上

## 常用高并发相关工具
Apache JMeter

# 高可用

## 常见限流算法

### 固定窗口计数器算法
固定窗口其实就是时间窗口。固定窗口计数器算法 规定了我们单位时间处理的请求数量
- 给定一个变量 counter 来记录当前接口处理的请求数量，初始值为 0（代表接口当前 1 分钟内还未处理请求）。
- 1 分钟之内每处理一个请求之后就将 counter+1 ，当 counter=33 之后（也就是说在这 1 分钟内接口已经被访问 33 次的话），后续的请求就会被全部拒绝。
- 等到 1 分钟结束后，将 counter 重置 0，重新开始计数

缺点: 这种限流算法无法保证限流速率，因而无法保证突然激增的流量
就比如说我们限制某个接口 1 分钟只能访问 1000 次，该接口的 QPS 为 500，前 55s 这个接口 1 个请求没有接收，后 1s 突然接收了 1000 个请求。然后，在当前场景下，这 1000 个请求在 1s 内是没办法被处理的，系统直接就被瞬时的大量请求给击垮了

### 滑动窗口计数器算法
固定窗口计数器算法的升级版，优化点：把时间以一定比例分片   
例如我们的接口限流每分钟处理 60 个请求，我们可以把 1 分钟分为 60 个窗口。每隔 1 秒移动一次，每个窗口一秒只能处理 不大于 60(请求数)/60（窗口数） 的请求， 如果当前窗口的请求计数总和超过了限制的数量的话就不再处理其他请求。

### 漏桶算法
我们可以把发请求的动作比作成注水到桶中，我们处理请求的过程可以比喻为漏桶漏水。我们往桶中以任意速率流入水，以一定速率流出水。当水超过桶流量则丢弃，因为桶容量是不变的，保证了整体的速率。
如果想要实现这个算法的话也很简单，准备一个队列用来保存请求，然后我们定期从队列中拿请求来执行就好了（和消息队列削峰/限流的思想是一样的）

### 令牌桶算法
令牌桶算法也比较简单。和漏桶算法算法一样，还是桶（这限流算法和桶过不去啊）。不过现在桶里装的是令牌了，请求在被处理之前需要拿到一个令牌，请求处理完毕之后将这个令牌丢弃（删除）。我们根据限流大小，按照一定的速率往桶里添加令牌。如果桶装满了，就不能继续往里面继续添加令牌了

## 限流
分类:
1. 请求频率限流(Request rate limiting)
2. 并发量限流(Concurrent requests limiting)
3. 传输速率限流

处理方式:
- 拒绝服务
- 排队等待: 消息队列可以起到削峰和限流的作用
- 服务降级: 当触发限流条件时，直接返回兜底数据

### 单机限流
Google Guava 自带的限流工具类 RateLimiter 。 RateLimiter 基于令牌桶算法，可以应对突发流量

### 分布式限流
1. 借助中间件架限流 ：可以借助 Sentinel 或者使用 Redis 来自己实现对应的限流逻辑。
2. 网关层限流 ：比较常用的一种方案，直接在网关层把限流给安排上了。不过，通常网关层限流通常也需要借助到中间件/框架。就比如 Spring Cloud Gateway 的分布式限流实现RedisRateLimiter就是基于 Redis+Lua 来实现的，再比如 Spring Cloud Gateway 还可以整合 Sentinel 来做限流

为什么建议 Redis+Lua 的方式
- 减少了网络开销: 我们可以利用 Lua 脚本来批量执行多条 Redis 命令，这些 Redis 命令会被提交到 Redis 服务器一次性执行完成，大幅减小了网络开销。
- 原子性: 一段 Lua 脚本可以视作一条命令执行，一段 Lua 脚本执行过程中不会有其他脚本或 Redis 命令同时执行，保证了操作不会被其他指令插入或打扰

## 超时&重试

### 超时
超时机制说的是当一个请求超过指定的时间（比如 1s）还没有被处理的话，这个请求就会直接被取消并抛出指定的异常或者错误
- 连接超时(ConnectTimeout): 客户端与服务端建立连接的最长等待时间。
- 读取超时(ReadTimeout): 客户端和服务端已经建立连接，客户端等待服务端处理完请求的最长时间。实际项目中，我们关注比较多的还是读取超时。

超时时间应该如何设置: 超时值设置太高或者太低都有风险。
- 如果设置太高的话，会降低超时机制的有效性，比如你设置超时为 10s 的话，那设置超时就没啥意义了，系统依然可能会出现大量慢请求堆积的问题。
- 如果设置太低的话，就可能会导致在系统或者服务在某些处理请求速度变慢的情况下（比如请求突然增多），大量请求重试（超时通常会结合重试）继续加重系统或者服务的压力，进而导致整个系统或者服务被拖垮的问题

建议读取超时设置为 1500ms ,这是一个比较普适的值。如果你的系统或者服务对于延迟比较敏感的话，那读取超时值可以适当在 1500ms 的基础上进行缩短。反之，读取超时值也可以在 1500ms 的基础上进行加长，不过，尽量还是不要超过 1500ms 。连接超时可以适当设置长一些，建议在 1000ms ~ 5000ms 之内

### 重试机制
重试机制一般配合超时机制一起使用，指的是多次发送相同的请求来避免瞬态故障和偶然性故障
- 瞬态故障可以简单理解为某一瞬间系统偶然出现的故障，并不会持久。偶然性故障可以理解为哪些在某些情况下偶尔出现的故障，频率通常较低

重试的次数如何设置: 重试的次数通常建议设为 3 次。并且，我们通常还会设置重试的间隔，比如说我们要重试 3 次的话，第 1 次请求失败后，等待 1 秒再进行重试，第 2 次请求失败后，等待 2 秒再进行重试，第 3 次请求失败后，等待 3 秒再进行重试

重试幂等: 需要注意保证同一个请求没有被多次执行

---

# 算法

## 常见数据结构



## 常见设计模式
1. 工厂模式(Factory)
2. 单例模式(Singleton)
3. 装饰模式(Decorator)
4. 策略模式(Strategy)
5. 代理模式(Proxy)
6. 观察者模式(Observer)

#### 工厂模式(Factory)
创建一个对象常常需要复杂的过程，所以不适合包含在一个复合对象中。创建对象可能会导致大量的重复代码，可能会需要复合对象访问不到的信息，也可能提供不了足够级别的抽象，还可能并不是复合对象概念的一部分。工厂方法模式通过定义一个单独的创建对象的方法来解决这些问题。
```java
public class ShapeFactory {
    
   //使用 getShape 方法获取形状类型的对象
   public Shape getShape(String shapeType){
      if(shapeType == null){
         return null;
      }        
      if(shapeType.equalsIgnoreCase("CIRCLE")){
         return new Circle();
      } else if(shapeType.equalsIgnoreCase("RECTANGLE")){
         return new Rectangle();
      } else if(shapeType.equalsIgnoreCase("SQUARE")){
         return new Square();
      }
      return null;
   }
}
```

#### 单例模式(Singleton)
- 饿汉式: 即直接创建对象，不存在线程安全问题
- 懒汉式: 延迟创建对象，有的方式存在线程安全问题

单例模式使用的场景: 需要频繁的进行创建和销毁的对象、创建对象时耗时过多或耗费资源过多(即重量级对象)，但又经常用到的对象、工具类对象、频繁访问数据库或文件的对象(比如数据源、session工厂等)

懒汉式: 双检锁
```java
public class SingleTon {
    private static volatile SingleTon singleTon;

    private SingleTon() {}

    public static SingleTon getSingleTon() {
        if (singleTon == null) {
            synchronized (SingleTon.class) {
                if (singleTon == null)
                    singleTon = new SingleTon();
            }
        }
        return singleTon;
    }
}
```

#### 装饰模式(Decorator)
允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
即: 原有的对象不能修改，就在原有的对象外面包一层
```java
public interface Shape {
   void draw();
}

// 装饰Shape
public abstract class ShapeDecorator implements Shape {
   protected Shape decoratedShape;
 
   public ShapeDecorator(Shape decoratedShape){
      this.decoratedShape = decoratedShape;
   }
 
   public void draw(){
      decoratedShape.draw();
   }  
}

// 实现RedShapeDecorator
public class RedShapeDecorator extends ShapeDecorator {
 
   public RedShapeDecorator(Shape decoratedShape) {
      super(decoratedShape);     
   }
 
   @Override
   public void draw() {
      decoratedShape.draw();         
      setRedBorder(decoratedShape);
   }
 
   private void setRedBorder(Shape decoratedShape){
      System.out.println("Border Color: Red");
   }
}

public class Rectangle implements Shape {
 
   @Override
   public void draw() {
      System.out.println("Shape: Rectangle");
   }
}

   public static void main(String[] args) {
      ShapeDecorator redRectangle = new RedShapeDecorator(new Rectangle());
      System.out.println("Rectangle of red border");
      redRectangle.draw();
      //Rectangle of red border
      //Shape: Rectangle 注: 这里调了RedShapeDecorator.draw() 所以打印了2行输出
      //Border Color: Red 
   }
```

#### 策略模式(Strategy)


#### 代理模式(Proxy)
在代理模式(Proxy Pattern)中，一个类代表另一个类的功能。这种类型的设计模式属于结构型模式
```java
public class RealImage implements Image {

   private String fileName;
 
   public RealImage(String fileName){
      this.fileName = fileName;
      loadFromDisk(fileName);
   }
 
   @Override
   public void display() {
      System.out.println("Displaying " + fileName);
   }
 
   private void loadFromDisk(String fileName){
      System.out.println("Loading " + fileName);
   }
}

public class ProxyImage implements Image{

   private RealImage realImage; // 被代理的对象
   private String fileName;
 
   public ProxyImage(String fileName){
      this.fileName = fileName;
   }
 
   @Override
   public void display() {
      if (realImage == null) {
         realImage = new RealImage(fileName);
      }
      // bufore: 这里可以添加需要在目标方法之前执行的代码
      realImage.display();
      // after: 这里可以添加需要在目标方法之后执行的代码
   }
}
```

#### 观察者模式(Observer)
当对象间存在一对多关系时，则使用观察者模式。比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式
```java

```


## LeetCode

### 动态规划(Dynamic Programming)

### 回溯法(Backtracking)

### 树(Tree, DFS)

---

# 其它

## SSO 单点登录
SSO 英文全称 Single Sign On，单点登录。SSO 是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统

核心功能:
- 单点登录
- 单点登出
- 支持跨域单点登录
- 支持跨域单点登出

常见的 Web 框架对于 Session 的实现都是生成一个 SessionId 存储在浏览器 Cookie 中。然后将 Session 内容存储在服务器端内存中

1. 用户登录成功之后，生成 AuthToken 交给客户端保存。如果是浏览器，就保存在 Cookie 中。如果是手机 App 就保存在 App 本地缓存中
2. 用户在浏览需要登录的页面时，客户端将 AuthToken 提交给 SSO 服务校验登录状态/获取用户登录信息

对于登录信息的存储，建议采用 Redis，使用 Redis 集群来存储登录信息，既可以保证高可用，又可以线性扩充。同时也可以让 SSO 服务满足负载均衡/可伸缩的需求




---

# 面试题

## 淘天集团

### 一二面

1. 自我介绍（简明扼要 突出重点）
2. 为什么离职（一定不要说前公司的不好，领导不好，不要有太多的抱怨，以积极的方式解释）
3. 业务介绍
4. 基础知识

1. 不用redis，实现一个本地缓存高并的发读写支持lru
2. 业务外部依赖有哪些
3. 写一个设计模式=策略模式 （设计模式是分两种场景设计，一个是单机场景，一个是分布式场景，分别说说怎么设计缓存）
4. 问项目的设计和思考的点 项目难点 和挑战的地方 然后就问了并发的原子性可见性顺序性
5. mq redis 和mysql的常规问题
6. 图书馆系统的设计题，以此提问如果要实现某些功能应该怎么改
7. 主要是根据具体项目问的 并不会单纯的问八股文那种
8. 大概就登录系统设计，库存超卖，然后接口幂等，中间件之类的
9. 二个影响深刻的项目或功能点，生产问题
10. 主要聊项目，还有redis主从复制原理，es集群架构

#### 不用redis，实现一个本地缓存高并的发读写支持LRU
设计和构建一个“最近最少使用”缓存，该缓存会删除最近最少使用的项目。缓存应该从键映射到值(允许你插入和检索特定键对应的值)，并在初始化时指定最大容量。当缓存被填满时，它应该删除最近最少使用的项目。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果密钥 (key) 存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果密钥不存在，则写入其数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

思路：LinkedList + HashMap: LinkedList用来保存key的访问情况，最近访问的key将会放置到链表的最尾端，如果链表大小超过容量，移除链表的第一个节点，同时移除该key在hashmap中对应的键值对。

#### 业务外部依赖有哪些
1. Redis
2. RocketMQ
3. DB
4. 部分业务接口调关联方微服务接口查

#### 写一个设计模式 策略模式
策略设计模式是一种行为设计模式。当在处理一个业务时，有多种处理方式，并且需要再运行时决定使哪一种具体实现时，就会使用策略模式。

例子：
实时缴费，根据不同的银行设置不同的策略，共同继承一个缴费参数生成接口。根据不同的银行new不同的对象，这里也用到了面向对象的多态特性。

#### 问项目的设计和思考的点 项目难点 和挑战的地方

#### 登录系统设计
登录系统的登录流程如下：
1. 用户选择登录方式（账号密码登录或第三方登录）。
2. 用户输入相应的登录信息（用户名和密码或第三方平台的授权）。
3. 系统验证用户输入的信息是否正确。
4. 验证成功则登录系统，否则提示相应的错误信息。

第三方登录的流程如下：
1. 用户选择第三方登录方式，系统生成相应的第三方登录链接。
2. 用户点击链接跳转到第三方登录平台进行授权并获取相应的授权码。
3. 系统使用授权码请求第三方平台获取用户的用户ID和访问令牌。
4. 系统根据用户ID和访问令牌验证用户的身份，并根据需要进行用户注册或登录。
5. 验证成功则登录系统，否则提示相应的错误信息。

#### 库存超卖
采用幂等机制，多次请求和一次请求产生的效果是一样的  
利用数据库自身特性 “主键唯一约束”，在插入订单记录时，带上主键值，如果订单重复，记录插入会失败

#### redis主从复制原理
Redis 主从同步最终一致性
Redis 的主从数据是异步同步的，所以分布式的 Redis 系统并不满足「一致性」要求。
Redis 保证「最终一致性」，从节点会努力追赶主节点，最终从节点的状态会和主节点的状态将保持一致。如果网络断开了，主从节点的数据将会出现大量不一致，一旦网络恢复，从节点会采用多种策略努力追赶上落后的数据，继续尽力保持和主节点一致。

HR沟通：离职原因，职业规划等

### 出现过的算法题
- 多线程控制输出 递归。前序遍历可解决
- 求字符串最长不重复子串的长度 
  - 滑动窗口+map<右指针指向的值, 左指针的index + 1>
- 字符串截取，把相邻的a和s之间的字符去掉
  - 双指针
- 有三个任务同时执行，然后全部执行完才返回，如果其中一个任务超过三秒也返回
- 两个排序链表合并 一个升序一个降序 合并排序成一个
  - 反转降序的链表
- 懒加载线程安全的单例
  - 双检锁 
- 两个线程输出奇数偶数，交替输出1234
  - 使用AtomicInteger保证原子性 or volatile int
- 两数之和等于一个目标值，求这两个数的位置
  - map缓存
- 三数之和
  - 2重循环+map缓存
- 判断字符串是否匹配abba模式
  - 双指针 左右一起往中间移动
- leetcode 79. 在二维网格内搜索单词
- 多线程交替打印ABC10次
- n个数量括号，请输出所有个合法的排列
  - 22. Generate Parentheses 括号生成:回溯法+剪枝+记忆化 时间复杂度:卡特兰数:O(4^n / 根号n)


#### 懒加载线程安全的单例
```java
public class DoubleCheckSingleton {
    /**
     * 加volatile关键字 避免指令重排、保证变量多线程可见性
     * 
     * instance = new DoubleCheckSingleton();
     * 1. 给DoubleCheckSingleton类的实例instance分配内存
     * 2. 调用实例instance的构造函数来初始化成员变量
     * 3. 将instance指向分配的内存地址
     * 
     * 使用了volatile修饰成员变量，那么在变量赋值之后，会有一个内存屏障。也就说只有执行完1,2,3步操作后，读取操作才能看到，读操作不会被重排序到写操作之前。这样以来就解决了对象状态不完整的问题。
     **/
    private volatile static DoubleCheckSingleton instance;

    //私有的构造方法
    private DoubleCheckSingleton() {}

    public static DoubleCheckSingleton getInstance() {
        if (instance == null) { //第一层检查
            synchronized (DoubleCheckSingleton.class) {
                if (instance == null) { //第二层检查
                    instance = new DoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}

/**
 * 通过静态内部类实现
 * 静态内部类是由JVM内部的锁机制来保证不会创建多个实例，非常巧妙的避开了多线程问题
 **/
public class HolderFactory {
    public static Singleton get() {
        return Holder.instance;
    }

    private static class Holder {
        public static final Singleton instance = new Singleton();
    }
}
```

#### 多线程交替打印ABC10次
```java
public class PrintABCUsingLock {

    private int times; // 控制打印次数
    private int state;   // 当前状态值：保证三个线程之间交替打印
    private Lock lock = new ReentrantLock();

    public PrintABCUsingLock(int times) {
        this.times = times;
    }

    /**
     * main 方法启动后，3 个线程会抢锁，但是 state 的初始值为 0，所以第一次执行 if 语句的内容只能是 线程 A，然后还在 for 循环之内，此时state = 1，只有 线程 B 才满足 1% 3 == 1，所以第二个执行的是 B，同理只有 *线程 C 才满足 2% 3 == 2，所以第三个执行的是 C，执行完 ABC 之后，才去执行第二次 for 循环，所以要把 i++ 写在 for 循环里边
     * 
     **/
    private void printLetter(String name, int targetNum) {
        for (int i = 0; i < times; ) {
            lock.lock();
            if (state % 3 == targetNum) {
                state++;
                i++;
                System.out.print(name);
            }
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        PrintABCUsingLock loopThread = new PrintABCUsingLock(1);

        new Thread(() -> {
            loopThread.printLetter("B", 1);
        }, "B").start();
        
        new Thread(() -> {
            loopThread.printLetter("A", 0);
        }, "A").start();
        
        new Thread(() -> {
            loopThread.printLetter("C", 2);
        }, "C").start();
    }
}
```




其他：
最重要的是面试中要能够对自己的项目清晰的描述，逻辑清晰，明确自己在其中的贡献，复盘时建议关注项目中的难点，是怎么解决的，是否有可优化的方法（项目中自己写的东西面试官问的时候一定要接得住）
面试尽量表现的积极向上，好学，且谦虚，就算跟面试官想法不一致，也不要当场起冲突顶撞面试官，不管是薪资还是级别等等没达到，都不要一口拒绝面试官，后面都可以争取的，就说考虑一下。
不主动谈薪酬如客户问薪酬问题，要说机会比薪酬重要；如果客户坚持问你的薪酬的话，你要告诉客户薪酬范围，而不是具体薪酬。
- 现在 12000 * 17 = 204000
- 期望 12000 * 20 = 240000 = 20000 * 12
-     12000 * 21 = 252000
-     20000 * 13 = 260000
-     21500 * 13 = 280000
- 区间 240000 -> 280000

