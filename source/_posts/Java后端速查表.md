---
title: Java后端速查表
date: 2022-02-20 10:04:40
categories:
    - Java
tags: 
    - 后端
    - JVM
---

# Java后端

### Java语言特性

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

深拷贝 vs 浅拷贝
- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

### Java基本类

异常(Throwable)： 
- Exception（异常）：是程序本身可以处理的异常。Exception 类有一个重要的子类 RuntimeException。
  - RuntimeException
    - 例如：ArithmeticException（算术运算异常，一个整数除以 0 时，抛出该异常）
- Error（错误）：是程序无法处理的错误，表示运行应用程序中较严重问题。表示代码运行时 JVM（Java 虚拟机）出现的问题。
  - 例如：当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

`java.util.HashMap`
- JDK1.7: 数组+单链表 链地址法
- JDK1.8: 数组+单链表+红黑树(当链表的深度达到8的时候，就会自动扩容把链表转成红黑树的数据结构来把时间复杂度从O(n)变成O(logN)提高了效率)
- put操作的流程：
  1. `key.hashcode()`，时间复杂度O(1)
  2. 找到桶以后，判断桶里是否有元素，如果没有，直接new一个entey节点插入到数组中。时间复杂度O(1)
  3. 如果桶里有元素，并且元素个数小于6，则调用equals方法，比较是否存在相同名字的key，不存在则new一个entry插入都链表尾部。时间复杂度O(n)
  4. 如果桶里有元素，并且元素个数大于6，则调用equals方法，比较是否存在相同名字的key，不存在则new一个entry插入都链表尾部。时间复杂度O(logn)

`java.util.concurrent.ConcurrentHashMap`
- 采用了分段锁技术
- JDK7采用Segment分段锁(默认16个Segment)
- JDK8改成了CAS+synchronized(这里只加锁在hashmap中的一个元素上面)
- 理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment
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

### 多线程

多线程
- 进程: 是程序的一次执行过程，是系统运行程序的基本单位。系统运行一个程序即是一个进程从创建，运行到消亡的过程。
- 线程: 与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是多个线程可以共享同一块内存空间和一组系统资源(堆)，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多。但是频繁的切换线程可能会消耗大量的CPU资源，因为需要频繁的保存和恢复线程运行上下文。

并行与并发
- 并行：多个cpu实例或者多台机器同时执行一段处理逻辑，是真正的同时。
- 并发：通过cpu调度算法，让用户看上去同时执行，实际上从cpu操作层面不是真正的同时。并发往往在场景中有公用的资源，那么针对这个公用的资源往往产生瓶颈，我们会用TPS或者QPS来反应这个系统的处理能力

线程安全: 
经常用来描绘一段代码。指在并发的情况之下，该代码经过多线程使用，线程的调度顺序不影响任何结果。这个时候使用多线程，我们只需要关注系统的内存，cpu是不是够用即可。

死锁:
产生死锁的四个必要条件
1. 互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
2. 不可剥夺条件: 进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能由获得该资源的进程自己来释放(只能是主动释放)。
3. 请求与保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
4. 循环等待条件: 存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, …, pn}，其中Pi等 待的资源被P(i+1)占有(i=0, 1, …, n-1)，Pn等待的资源被P0占有

Java 中实现多线程的方法
1. 继承 `Thread` 类
2. 实现 `Runnable` 接口: 如果一个类继承 Thread类，则不适合于多个线程共享资源，而实现了 Runnable 接口，就可以方便的实现资源的共享

同步: 
Java中的同步指的是通过人为的控制和调度，保证共享资源的多线程访问成为线程安全，来保证结果的准确。如上面的代码简单加入`synchronized`关键字。

`synchronized`关键字:
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
      - JVM执行到`monitorenter`指令时它会尝试获取对象的锁，如果该对象没有锁，或者当前线程已经拥有了这个对象的锁时，它会把计数器+1；然后当执行到`monitorexit` 指令时就会将计数器-1；然后当计数器为0时，锁就释放了。如果获取锁 失败，那么当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。
      - 反编译后可以看到一个`monitorenter`和两个`monitorexit`: 这是因为第二个`monitorexit`是给异常处理释放锁用的
      - monitor到底是什么: monitor它就是个监视器，底层源码是C++编写的2. `flag=ACC_SYNCHRONIZED`: 修饰同步方法
      - 这标志用来告诉JVM这是一个同步方法，在进入该方法之前先获取相应的锁，锁的计数器加1，方法结束后计数器-1，如果获取失败就阻塞住，知道该锁被释放。

> [深入理解synchronized底层源码](https://blog.csdn.net/realize_dream/article/details/106968443)

先行发生原则(happens-before): 在发生操作B之前，操作A产生的影响能被操作B观察到。先行发生原则是判断数据是否存在竞争、线程是否安全的主要依据

`volatile`关键字: 当一个变量定义为volatile之后，它将具备两种特性
1. 保证此变量对所有线程的可见性，即当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的
2. 禁止指令重排序优化

JVM中对象实例的组成:
1. 对象头: 
  - Mark Word:
    - 对象的hashCode
    - 锁信息: 记录对象锁当前的状态，在申请锁、锁升级等过程中JVM都需要读取对象的Mark Word数据
      - JDK6之前: 无锁、有锁(重量级锁)
      - JDK6之后: 无锁 -> 偏向锁 -> 轻量级锁(CAS) -> 重量级锁
        - 偏向锁: 如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也就变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作
      - 轻量级锁(2个线程版本的偏向锁): 当存在第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。注意这里的第二个线程只是申请锁，不存在两个线程同时竞争锁，可以是一前一后地交替执行同步块
      - 重量级锁: 当同一时间有多个线程竞争锁时，锁就会被升级成重量级锁，此时其申请锁带来的开销也就变大
    - 分代年龄
    - GC标志
  - Class Metadata Address: 类型指针指向对象的类元数据，JVM通过该指针确定该对象是哪个类的实例
2. 实例数据: 存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐
3. 对其填充

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

```java
public ThreadPoolExecutor(int corePoolSize, // 核心线程数
                          int maximumPoolSize, // 线程池最大线程数
                          long keepAliveTime, // 超出核心线程数的线程等待new task的时间，超过则terminated
                          TimeUnit unit, // keepAliveTime时间的单位
                          BlockingQueue<Runnable> workQueue, // 用于保存等待执行的任务的阻塞队列: 用于存放等待线程执行的task的队列只存放由execute方法提交的Runnable任务
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
        new ArrayBlockingQueue(10)); // 有界队列，容量为10，最多容纳10个task
```

workQueue(工作队列): 
- `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO（先进先出）原则对任务进行排序。使用该队列，线程池中能创建的最大线程数为`maximumPoolSize`。 
- `LinkedBlockingQueue`: 基于链表结构的无界阻塞队列，按FIFO（先进先出）原则对任务进行排序，吞吐量高于`ArrayBlockingQueue`。使用该队列，线程池中能创建的最大线程数为`corePoolSize`。
- `SynchronousQueue`: 一个不存储元素的阻塞队列。添加任务的操作必须等到另一个线程的移除操作，否则添加操作一直处于阻塞状态。
- `PriorityBlockingQueue`: 一个支持优先级的无界阻塞队列。使用该队列，线程池中能创建的最大线程数为`corePoolSize`。

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

AQS(AbstractQueuedSynchronized):
- 抽象队列同步器AQS: 是一个同步框架，它提供通用机制来原子性管理同步状态、阻塞和唤醒线程，以及维护被阻塞线程的队列. AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch

CAS原理:
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

缓存一致性协议(MESI = modified + exclusive + shared + invalid)
- 作用: 多核CPU有多个一级缓存，保证缓存内部数据的一致,不让系统数据混乱
- MESI中每个缓存行(Cache line:CPU中缓存存储数据的单元)都有四个状态
  1. M(修改): 该Cache line有效，数据被修改了，和内存中的数据不一致，数据只存在于本Cache中
  2. E(互斥): 该Cache line有效，数据和内存中的数据一致，数据只存在于本Cache中
  3. S(共享): 该Cache line有效，数据和内存中的数据一致，数据存在于很多Cache中。当有一个CPU修改该缓存行对应的内存的内容时会使该缓存行变成 I 状态
  4. I(无效): 该Cache line无效
- CPU 是通过总线和内存进行数据传输的。在多核心时代下，多个核心通过同一条总线和内存以及其他硬件进行通信
- 通过在 inc 指令前添加 lock 前缀，即可让该指令具备原子性。多个核心同时执行同一条 inc 指令时，会以串行的方式进行

> [CPU缓存一致性协议MESI](https://blog.csdn.net/m18870420619/article/details/82431319)

用户态内核态
- Linux操作系统的体系架构分为用户态和内核态
- 内核态: 本质上是一种软件，控制计算机硬件资源(CPU资源、存储资源、I/O资源等)
- 用户态: 上层应用程序的活动空间
- 上层应用想要访问计算机硬件资源需要通过内核提供的访问接口(系统调用)来调用
- 系统调用是操作系统的最小功能单位
- 从用户态到内核态切换可以通过三种方式:
  1. 系统调用
  2. 异常: 如果当前进程运行在用户态，如果这个时候发生了异常事件，就会触发切换。例如：缺页异常
  3. 外设中断: 当外设完成用户的请求时，会向CPU发送中断信号

### JVM

类加载子系统: 根据给定的全限定名类名(如java.lang.Object)来装载class文件的内容到方法区(Method Area)
1. Bootstrap ClassLoader(启动类加载器): `$JAVA_HOME`中`jre/lib/rt.jar`里所有的class，由C++实现
2. Extension ClassLoader(扩展类加载器): 负责加载java平台中扩展功能的一些jar包，包括`$JAVA_HOME`中`jre/lib/*.jar`或`-D java.ext.dirs`指定目录下的jar包
3. App ClassLoader(系统类加载器): 负责加载classpath中指定的jar包及目录中class
4. Custom ClassLoader(用户自定义类加载器): 属于应用程序根据自身需要自定义的ClassLoader，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader

双亲委派机制: JVM对class文件采用按需加载的方式，在加载时JVM采用的是双亲委派机制，即把请求交由父类处理，它是一种任务委派模式
1. 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
2. 如果父类加载器还存在其父类加载器，则进一步向上委托，一次递归，请求最终将到达顶层的启动类加载器
3. 如果父类加载器可以完成类加载任务，就成功返回，若无法完成，子类加载器才会去加载。

运行时数据区(Runtime Data Area)
1. 程序计数器(Program Counter Register) <- 线程不共享
2. 本地方法栈(Native Method Stack) <- 线程不共享
3. 虚拟机栈(Java Virtual Machine Stack) <- 线程不共享
4. 方法区(Method Area) <- 线程共享
5. 堆(Heap) <- 线程共享

堆(Heap): 年青代:老年代=1:2
- 年青代(Young): Eden:From:To=8:1:1
  - Eden: 新创建的对象绝大部分会分配在Eden区。当Eden区内存不够的时候，就会触发MinorGC
  - Survivor 0(From): 在GC开始的时候，对象只会存在于Eden区和名为From的Survivor区，To区是空的，一次MinorGc过后，Eden区和SurvivorFrom区存活的对象会移动到SurvivorTo区中，然后会清空Eden区和SurvivorFrom区，并对存活的对象的年龄+1，如果对象的年龄到`15`，则直接分配到老年代。
  - Survivor 1(To)
- 老年代(Tenured): 老年代存放从年轻代存活的对象。一般来说老年代存放的都是生命期较长的对象

GC(Generational Collecting)垃圾回收:
- Minor GC: 当伊甸园的空间满时，程序又需要创建对象，触发Minor GC
- Full GC: 当老年代内存不足时，对老年代进行垃圾回收。这时可能会伴随着STW(Stop The World)
- STW(Stop The World): 停止所有线程，进行垃圾回收，这时候线程会被阻塞，直到垃圾回收完成
  - Q: 为什么要STW？-> A: 如果不执行STW的话在Full GC的过程中如果有一个线程执行完毕，那么这个线程的局部变量表里面所指向的在堆里的对象都会变成垃圾，但是此时Full GC还没执行完，那么这次Full GC执行所得到的结果是不准确的

JVM虚拟机调优:
- 主要是减少STW发生的频率，因为发生STW的时候会阻塞全部的用户线程，在用户看来就是应用程序卡顿
- 能不能通过调整JVM参数是的几乎发生Full GC?
  - 默认年青代:老年代=1:2，改成2:1，那么当年轻代满的时候绝大部分昭生夕死的对象会被干掉，只有实在干不掉的才会挪到老年代
- jVisualVM: 可视化工具，可以查看JVM的内存使用情况，安装插件可以观察到堆分代模型的整个GC过程

判断对象是否需要回收
1. 引用计数法: 给对象添加一个引用计数器。但是难以解决循环引用问题。
2. 可达性分析法: 通过一系列的 `GC Roots` 的对象作为起始点，从这些节点出发所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连的时候说明对象不可用。

垃圾回收算法
1. 标记-清除算法(Mark-Sweep): 
  - 分为两个阶段：标记阶段(标记出所有需要被回收的对象) -> 清除阶段(回收被标记的对象所占用的空间)
  - 缺点: 效率不高、空间会产生大量碎片
2. 复制算法(Copying): 将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用的内存空间一次清理掉，这样一来就不容易出现内存碎片的问题。Eden:From:To=8:1:1
3. 标记-整理算法(Mark-Compact): 在完成标记之后，它不是直接清理可回收对象，而是将存活对象都向一端移动，然后清理掉端边界以外的内存
4. 分代收集(Generational Collection): 根据对象的生命周期划分几块内存区，一般是分为新生代和老年代。新生代:老年代=1:2

垃圾收集器:
1. Serial/Serial Old收集器: 单线程收集器，进行垃圾收集时，必须暂停所有用户线程
  - Serial: 新生代 Copying算法
  - Serial Old: 老年代 Mark-Compact算法
2. ParNew收集器: Serial收集器的多线程版本
3. Parallel Scavenge收集器: 新生代的多线程收集器回收期间不需要暂停其他用户线程 Copying算法
4. Parallel Old收集器: 多线程 Mark-Compact算法
5. CMS(Concurrent Mark Sweep)收集器: 并发收集器，优点是最短回收停顿时间 Mark-Sweep算法
6. G1收集器: 并行与并发收集器，并且它能建立可预测的停顿时间模型

程序计数器(Program Counter Register): 每个线程都要它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址，若为native方法则为undefined

本地方法栈(Native Method Stack): 本地方法栈用于管理本地方法的调用

虚拟机栈(Java Virtual Machine Stack): 
- 线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame)
- 栈帧的内部结构：
  1. 局部变量表(Local Variables): 最基本的存储单元是Slot(变量槽)，容量大小是在编译期确定下来的，并保存在方法的Code属性的maximum local variables数据项中。JVM会为局部变量表中的每一个Slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值。
  2. 操作数栈(operand Stack)(或表达式栈): 用于保存计算过程的中间结果
  3. 动态链接(DynamicLinking)(或指向运行时常量池的方法引用): 每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用，包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接(Dynamic Linking)
  4. 方法返回地址(Return Address)(或方法正常退出或者异常退出的定义): 存放调用该方法的pc寄存器的值

方法区(Method Area): 元空间(Metaspace)是其实现，元空间并不在虚拟机中，而是使用本地内存

### Redis

redis单线程为什么快?
1. 纯内存操作: 数据存放在内存中，内存的响应时间大约是100纳秒
2. 单线程: Redis是基于内存的操作，CPU不是Redis的瓶颈。Redis的瓶颈最有可能是机器内存或者网络带宽，同时避免了线程切换和竞态产生的消耗 
3. 非阻塞I/O: Redis采用epoll做为I/O多路复用技术的实现 ，再加上Redis自身的事件处理模型将epoll中的连接，读写，关闭都转换为了时间，不在I/O上浪费过多的时间 
4. 客户端调服务端:
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

分布式锁
- 使用`setnx`命令实现: 在Redis中，`setnx`命令的作用是，如果key不存在，则设置key-value并返回true，如果key存在，则不做任何操作并返回false; 利用这一机制在获取锁为设置key，设置成功返回true表示获取到锁了，执行完业务逻辑之后删除掉设置的key即可

epoll: linux内核下的一个高效的处理大批量的文件操作符的一个实现

### MySQL

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

mysql索引数据结构为什么选择B+Tree?
- 二叉树: 无平衡机制，插入递增元素，退化成链表
- 红黑树(是一种二叉平衡树): 数据量大时树的深度也会很深
- B Tree: 多叉，从左到右依次递增，叶子节点都在同一层
- B+ Tree: 非叶子节点不存储data，叶子节点包含所有索引字段并用指针从左往右链接成链表。

B+ Tree: 每个节点大小16kb限制，16kb/每个节点大小(8字节索引元素+6字节孩子节点磁盘文件地址指针)=1170个索引
- 索引全部加载到内存，找到后根据磁盘文件地址进行一次磁盘I/O读取对应的数据
- 只用3层的B+ Tree就支持上千万行数据的查找

MVCC


### 高并发

#### 常用高并发相关工具

Apache JMeter