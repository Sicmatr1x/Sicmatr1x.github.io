---
title: Java复习笔记：多线程与并发第一章
date: 2022-11-13 17:07:53
categories:
    - Back-end
tags: 
    - Java
---

## 基本概念回顾

### 进程和线程的区别

#### 进程和线程的由来

1. 串行：早起的计算机只能执行串行任务，并且遇到用户输入的操作时便会阻塞
2. 批处理：预先将用户的指令集中成清单，批量串行处理用户指令，仍无法并发执行
3. 进程：进程独占内存空间，保存各自运行状态，相互不干扰且可以互相切换，为并发处理任务提供了可能
4. 线程：共享进程的内存资源，相互间切换更便捷，支持更细粒度的任务控制，让进程内的子任务得以并发执行

#### 区别

**进程和线程都是一个时间段的描述，是CPU工作时间段的描述。进程和线程都是一个时间段的描述，是CPU工作时间段的描述，不过是颗粒大小不同。**

所有与进程相关的资源都被记录在[PCB(PCB Process Control Block)](https://baike.baidu.com/item/PCB/16067368?fr=aladdin)中。

PCB:
- 描述信息
- 控制信息
- 资源信息
  - 程序段
  - 数据段
- CPU现场

> 它是进程实体的一部分，是操作系统中最重要的记录性数据结构。它是进程管理和控制的最重要的数据结构，每一个进程均有一个PCB，在创建进程时，建立PCB，伴随进程运行的全过程，直到进程撤消而撤消。

进程拥有完整的[虚拟内存地址空间](https://blog.csdn.net/u014379540/article/details/52263114)，而同一进程下的线程则共享该进程拥有的内存空间。

线程的组成：
- 堆栈寄存器
- 程序计数器
- TCB

进程就是包括上下文切换的程序执行时间总和 = CPU加载上下文+CPU执行+CPU保存上下文。

<img src="进程与线程.png">

进程的颗粒度太大，每次都要有上下的调入，保存，调出。

假设存在进程A，其实际分成 a，b，c等多个块组合而成。那么这里具体的执行就可能变成：

进程A得到CPU->CPU加载上下文，开始执行程序A的a小段，然后执行A的b小段，然后再执行A的c小段，最后CPU保存A的上下文。

这里的a，b，c就是线程，也就是说线程是共享了进程的上下文环境，的更为细小的CPU时间段。

**进程是资源分配的最小单位，线程是CPU调度的最小单位。**

### Linux用户态和内核态转换

#### 为什么需要转换

#### 内核态的多线程是如何通过轻量级线程来实现的

### 什么是系统中断

## Java中的进程和线程

### Java进程和线程的关系

- 运行一个程序会产生一个进程，进程包含至少一个线程
- 每个进程对应一个JVM实例，多个线程共享JVM里的堆
- Java采用单线程编程模型，程序会自动创建主线程

### Thread中的`start`和`run`方法的区别

使用`run`方法会继续使用主线程来执行重写的`run`方法里面的内容，而使用`start`方法则会开一个新线程来执行。

我们看一下start方法源码

> Thread.java

```java
/**
     * Causes this thread to begin execution; the Java Virtual Machine
     * calls the <code>run</code> method of this thread.
     * <p>
     * The result is that two threads are running concurrently: the
     * current thread (which returns from the call to the
     * <code>start</code> method) and the other thread (which executes its
     * <code>run</code> method).
     * <p>
     * It is never legal to start a thread more than once.
     * In particular, a thread may not be restarted once it has completed
     * execution.
     *
     * @exception  IllegalThreadStateException  if the thread was already
     *               started.
     * @see        #run()
     * @see        #stop()
     */
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();
```

可以看到在`start`方法里面主要是使用到了一个native的方法`start0()`，该方法调用到了外部的非Java的源码。

可以访问[OpenJKD](http://hg.openjdk.java.net/)来查询

[JKD8 Thread.c源码](http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/dd10fb830ea9/src/share/native/java/lang/Thread.c)

> Thread.c

```c
#include "jni.h"
#include "jvm.h"

#include "java_lang_Thread.h"

#define THD "Ljava/lang/Thread;"
#define OBJ "Ljava/lang/Object;"
#define STE "Ljava/lang/StackTraceElement;"
#define STR "Ljava/lang/String;"

#define ARRAY_LENGTH(a) (sizeof(a)/sizeof(a[0]))

static JNINativeMethod methods[] = {
    {"start0",           "()V",        (void *)&JVM_StartThread},
    {"stop0",            "(" OBJ ")V", (void *)&JVM_StopThread},
    {"isAlive",          "()Z",        (void *)&JVM_IsThreadAlive},
    {"suspend0",         "()V",        (void *)&JVM_SuspendThread},
    {"resume0",          "()V",        (void *)&JVM_ResumeThread},
    {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
    {"yield",            "()V",        (void *)&JVM_Yield},
    {"sleep",            "(J)V",       (void *)&JVM_Sleep},
    {"currentThread",    "()" THD,     (void *)&JVM_CurrentThread},
    {"countStackFrames", "()I",        (void *)&JVM_CountStackFrames},
    {"interrupt0",       "()V",        (void *)&JVM_Interrupt},
    {"isInterrupted",    "(Z)Z",       (void *)&JVM_IsInterrupted},
    {"holdsLock",        "(" OBJ ")Z", (void *)&JVM_HoldsLock},
    {"getThreads",        "()[" THD,   (void *)&JVM_GetAllThreads},
    {"dumpThreads",      "([" THD ")[[" STE, (void *)&JVM_DumpThreads},
    {"setNativeName",    "(" STR ")V", (void *)&JVM_SetNativeThreadName},
};

#undef THD
#undef OBJ
#undef STE
#undef STR

JNIEXPORT void JNICALL
Java_java_lang_Thread_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls, methods, ARRAY_LENGTH(methods));
}
```

可以看到`start0`方法调用到了`JVM_StartThread`方法，而该方法引自`jvm.h`

[JDK8 jvm.cpp源码](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/b44df6c5942c/src/share/vm/prims/jvm.cpp)

在jvm.cpp下的`JVM_StartThread`方法里有下面这句话用于创建一个线程

> jvm.cpp

```c
native_thread = new JavaThread(&thread_entry, sz);
```

搜索上面用于创建线程的方法传入的参数`thread_entry`

```c
static void thread_entry(JavaThread* thread, TRAPS) {
  HandleMark hm(THREAD);
  Handle obj(THREAD, thread->threadObj());
  JavaValue result(T_VOID);
  JavaCalls::call_virtual(&result,
                          obj,
                          KlassHandle(THREAD, SystemDictionary::Thread_klass()),
                          vmSymbols::run_method_name(),
                          vmSymbols::void_method_signature(),
                          THREAD);
}
```

可以看到该方法最后会调用JVM虚拟机`JavaCalls::call_virtual`，并传入`run_method_name`

综上所述：
- 调用`start`方法会：Thread#start()->JVM_StartThread->thread_entry->Thread#run()
  - 在`thread_entry`时创建一个新的子线程并启动去运行Thread#run()里的方法体
- 调用`run`方法会：Thread#run()
  - 当做一个普通的方法调用去调用Thread#run()里的方法体

### Thread和 Runnable是什么关系

- Thread类实现了Runnable接口，使得run支持多线程
- 因为类的单一继承原则，推荐使用Runnable接口

实现了Runnable接口是没有start方法的，需要把其对象作为参数去创建一个Thread对象再调用start方法启动

### 如何给`run()`方法传参

1. 构造函数传参
2. 成员变量传参
3. 回调函数传参

### 处理线程的返回值

1. 主线程等待法：让主线程循环等待直到子线程返回
2. 使用Thread类的`join()`阻塞当前主线程以等待子线程处理完毕
```java
public class Test implements Runnable {
    @Override
    void run() {
        try {
            Thread.currentThread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        value = "data";
    }
    public static void main(String[] args) throws InterruptedException {
        Test test = new Test();
        Thread t = new Thread(test);
        t.start();
        t.join();
        System.out.println("value:" + cw.value);
    }
}
```
3. 通过Callable接口实现：通过FutureTask or 线程池获取

```java
// 通过FutureTask
public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception{
        String value="test";
        System.out.println("ready to work");
        Thread.currentThread().sleep(5000);
        System.out.println("Task down");
        return value;
    }
}

public class FutureTaskDemo {
    public static void main(String[] args) throws InterruptedException {
        FutureTask<String> task = new FutureTask<>(new MyCallable());
        new Thread(task);
        if(!task.isDone()){
            System.out.println("task has not finished, please wait");
        }
        // 显示MyCallable里面的返回值
        System.out.println("task return:" + task.get());
    }
}
```

```java
// 通过线程池
public class ThreadPoolDemo {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService newPool = Executors.newCachedThreadPool();
        Future<String> future = newPool.submit(new MyCallable());
        if (!task.isDone()) {
            System.out.println("task has not finished, please wait");
        }
        // 显示MyCallable里面的返回值
        try {
            System.out.println("task return:" + task.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } finally {
            newPool.shutdown();
        }
    }
}
```

### `sleep`和`wait`的区别

- `sleep`是Thread类的方法，`wait`是Object类中定义的方法
- `sleep`方法可以在任何地方使用
- `wait`方法只能在`synchronized`方法或`synchronized`快中使用
- `Thread.sleep`只会让出CPU，不会导致锁行为的改变
- `Object.wait`不仅会让出CPU，还会释放已经占有的同步资源

```java
public class WaitSleepDemo {

  public static void main(String[] args) {
    final Object lock = new Object();
    new Thread(new Runnable() {
      @Override
      public void run() {
        System.out.println("thread A is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread A get lock");
            Thread.sleep(20);
            System.out.println("thread A do wait lock");
            lock.wait(1000);
            System.out.println("thread A is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    new Thread(new Runnable() {
      @Override
      public void run() {
        System.out.println("thread B is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread B get lock");
            System.out.println("thread B sleeping 10 ms");
            Thread.sleep(10);
            System.out.println("thread B is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
  }
}
```

```
thread A is waiting to get lock
thread A get lock
thread B is waiting to get lock
thread A do wait lock
thread B get lock
thread B sleeping 10 ms
thread B is done
thread A is done
```

观察输出可发现，在A获得锁之后B开始等待锁，而A开始wait之后B就获得了锁

### `notify`和`notifyAll`的区别

锁池 EntryList：假设线程A已经拥有了某对象的锁，而其它线程B、C想要调用这个对象的某个`synchronized`方法(或块)之前必须先获得该对象锁的拥有权，而恰巧该对象的锁目前正被线程A占用，此时B、C线程就会被阻塞，进入一个地方去等待锁的释放，这个地方便是该对象的锁池。

等待池 WaitSet：假设线程A调用了某个对象的`wait()`方法，线程A就会释放该对象的锁，同时线程A就进入到了该对象的等待池中，进入到等待池中的线程不会去竞争该对象的锁。

- `notifyAll`会让所有处于等待池中的线程全部进入锁池去竞争获取锁的机会
- `notify`会随机选取一个处于等待池中的线程进入锁池去竞争获取锁的机会

```java
public class WaitSleepDemo {

  public static void main(String[] args) {
    final Object lock = new Object();
    new Thread(new Runnable() {
      @Override
      public void run() {
        System.out.println("thread A is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread A get lock");
            Thread.sleep(20);
            System.out.println("thread A do wait lock");
            lock.wait();
            System.out.println("thread A is done");
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    new Thread(new Runnable() {
      @Override
      public void run() {
        System.out.println("thread B is waiting to get lock");
        synchronized (lock) {
          try {
            System.out.println("thread B get lock");
            System.out.println("thread B sleeping 10 ms");
            Thread.sleep(10);
            System.out.println("thread B is done");
            lock.notify(); // or lock.notifyAll();
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
        }
      }
    }).start();
  }
}
```

```java
public class NotificationDemo {
  private volatile boolean go = false;
  private synchronized void go() {
    while (go == false) {
      System.out.println(Thread.currentThread() + " is going to notify all or one thread waiting on");
      go = true;
      notify();
    }
  }
  private synchronized void shouldGo() throws InterruptedException {
    while (go != true) {
      System.out.println(Thread.currentThread() + " is going to wait on this object");
      wait();
      System.out.println(Thread.currentThread() + " is woken up");
    }
    go = false;
  }
  public static void main(String[] args) throws InterruptedException {
    final NotificationDemo test = new NotificationDemo();
    Runnable waitTask = new Runnable() {
      @Override
      public void run() {
        try {
          test.shouldGo();
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " finished Execution");
      }
    };

    Runnable notifyTask = new Runnable() {
      @Override
      public void run() {
        test.go();
        System.out.println(Thread.currentThread().getName() + " finished Execution");
      }
    };

    Thread t1 = new Thread(waitTask, "WT1");
    Thread t2 = new Thread(waitTask, "WT2");
    Thread t3 = new Thread(waitTask, "WT3");
    Thread t4 = new Thread(notifyTask, "NT1");

    t1.start();
    t2.start();
    t3.start();

    Thread.sleep(200);

    t4.start();
  }
}
```

```
Thread[WT1,5,main] is going to wait on this object
Thread[WT2,5,main] is going to wait on this object
Thread[WT3,5,main] is going to wait on this object
Thread[NT1,5,main] is going to notify all or one thread waiting on
Thread[WT1,5,main] is woken up
NT1 finished Execution
WT1 finished Execution
```

1. 从输出来看前3行是WT1、WT2、WT3依次进入等待池，
2. 之后NT1调用`notify`方法随机唤醒一个线程将其置入锁池，并修改`go = true;`跳出循环
3. 这里是WT1被置入锁池，因为上一步中`go`的值被修改所以跳出循环，WT1获得锁并且修改了变量`go = false;`，然后因为NT1线程已经结束所以剩下两个线程WT2、WT3依然处于等待池。

### `yield`

当调用`Thread.yield()`时会给线程调度器`scheduler`一个当前线程愿意让出CPU的使用的信号，但是调度去可能会无视该暗示。

> Thread.java

```java
    /**
     * A hint to the scheduler that the current thread is willing to yield
     * its current use of a processor. The scheduler is free to ignore this
     * hint.
     *
     * <p> Yield is a heuristic attempt to improve relative progression
     * between threads that would otherwise over-utilise a CPU. Its use
     * should be combined with detailed profiling and benchmarking to
     * ensure that it actually has the desired effect.
     *
     * <p> It is rarely appropriate to use this method. It may be useful
     * for debugging or testing purposes, where it may help to reproduce
     * bugs due to race conditions. It may also be useful when designing
     * concurrency control constructs such as the ones in the
     * {@link java.util.concurrent.locks} package.
     */
    public static native void yield();
```

```java
public class YieldDemo {
  public static void main(String[] args) {
    Runnable yieldTask = new Runnable() {
      @Override
      public void run() {
        for (int i = 0; i <= 10; i++) {
          System.out.println(Thread.currentThread().getName() + i);
          if (i == 5) {
            Thread.yield();
          }
        }
      }
    };
    Thread t1 = new Thread(yieldTask, "A");
    Thread t2 = new Thread(yieldTask, "B");
    t1.start();
    t2.start();
  }
}
```

输出：

```
A0
B0
A1
B1
A2
B2
A3
B3
A4
B4
B5
A5
B6
B7
B8
B9
B10
A6
A7
A8
A9
A10
```

可以看出当A线程执行到5时把CPU让给了B来执行，直到B执行到10把B让给A

### 使用`interrupt`来中断线程

已被抛弃的方法：
- ~~`stop()`~~：过于暴力，被中断线程可能没有释放锁
- ~~`suspend()`~~, ~~`resume()`~~

`interrupt()`
- 如果线程处于被阻塞状态，那么线程将立即退出被阻塞状态，并抛出一个`InterruptedException`异常
- 如果线程处于正常状态，那么线程会将该线程的中断标志设置为true。被设置中断标志的线程将继续正常运行，不受影响。

```java
public class InterruptDemo {

  public static void main(String[] args) throws InterruptedException {
    Runnable interruptTask = new Runnable() {
      @Override
      public void run() {
        int i = 0;
        try {
          // 在正常运行任务时，经常检查本线程的中断标志位，如果设置了中断标志就自行停止线程
          while (!Thread.currentThread().isInterrupted()) {
            Thread.sleep(100);
            i++;
            System.out.println(Thread.currentThread().getName() + " (" + Thread.currentThread().getState()+ ") loop:i=" + i);
          }
        } catch (InterruptedException e) {
          // 在调用阻塞方法时正确处理InterruptedException异常
          System.out.println(Thread.currentThread().getName()+ " (" + Thread.currentThread().getState() + ") catch InterruptedException");
        }
      }
    };
    Thread t1 = new Thread(interruptTask, "t1");
    System.out.println(t1.getName() + " (" + t1.getState() + ") is new.");

    t1.start();
    System.out.println(t1.getName() + " (" + t1.getState() + ") is started.");

    Thread.sleep(300);
    t1.interrupt();
    System.out.println(t1.getName() + " (" + t1.getState() + ") is interrupted.");

    Thread.sleep(300);
    System.out.println(t1.getName() + " (" + t1.getState() + ") is interrupted now.");
  }
}
```

输出：

```
t1 (NEW) is new.
t1 (RUNNABLE) is started.
t1 (RUNNABLE) loop:i=1
t1 (RUNNABLE) loop:i=2
t1 (RUNNABLE) catch InterruptedException
t1 (RUNNABLE) is interrupted.
t1 (TERMINATED) is interrupted now.
```


## 线程的状态

> Thread.java

```java
    /**
     * A thread state.  A thread can be in one of the following states:
     * <ul>
     * <li>{@link #NEW}<br>
     *     A thread that has not yet started is in this state.
     *     </li>
     * <li>{@link #RUNNABLE}<br>
     *     A thread executing in the Java virtual machine is in this state.
     *     </li>
     * <li>{@link #BLOCKED}<br>
     *     A thread that is blocked waiting for a monitor lock
     *     is in this state.
     *     </li>
     * <li>{@link #WAITING}<br>
     *     A thread that is waiting indefinitely for another thread to
     *     perform a particular action is in this state.
     *     </li>
     * <li>{@link #TIMED_WAITING}<br>
     *     A thread that is waiting for another thread to perform an action
     *     for up to a specified waiting time is in this state.
     *     </li>
     * <li>{@link #TERMINATED}<br>
     *     A thread that has exited is in this state.
     *     </li>
     * </ul>
     *
     * <p>
     * A thread can be in only one state at a given point in time.
     * These states are virtual machine states which do not reflect
     * any operating system thread states.
     *
     * @since   1.5
     * @see #getState
     */
    public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```

Java线程状态：
1. 新建(NEW)：创建后尚未启动的线程状态
2. 运行(RUNNABLE)：包含Running(正在执行)和Ready(正在等待CPU分配时间片)
   - Ready(正在等待CPU分配时间片)：其它线程调用了该对象的`start()`方法，该线程位于可运行线程池中，等待被线程调度选中，获取CPU使用权。
   - Running(正在执行)：就绪状态的线程在获得CPU时间片后变为运行中状态(running)
3. 运行(RUNNING)：可运行状态(runnable)的线程获得了cpu 时间片(timeslice)，执行程序代码
4. 无限期等待(WAITING)：不会被分配CPU执行时间，需要被显示唤醒，进入该状态的线程需要等待其他线程做出一些特定动作(通知或中断)
   - 没有设置`Timeout`参数的`Object.wait()`方法
   - 没有设置`Timeout`参数的`Thread.join()`方法
   - `LockSupport.park()`方法
5. 限期等待(TIMED_WAITING)：在一定时间后会由系统自动唤醒
   - `Thread.sleep()`方法
   - 设置了`Timeout`参数的`Object.wait()`方法
   - 设置了`Timeout`参数的`Thread.join()`方法
   - `LockSupport.parkNanos()`方法
   - `LockSupport.parkUntil()`方法
6. 阻塞(BLOCKED)：等待获取排它锁，线程试图获取一个内部对象的`Monitor`（进入`synchronized`方法或`synchronized`块）但是其他线程已经抢先获取，那此线程被阻塞，知道其他线程释放`Monitor`并且线程调度器允许当前线程获取到`Monitor`，此线程就恢复到可运行状态。
7. 结束(TERMINATED)：已终止线程的状态，线程已经结束执行

<img src="./线程的状态.png">

<img src="./等待队列.png">

## 参考

- [一文读懂Java线程状态转换](https://segmentfault.com/a/1190000016197831?utm_source=tag-newest)

下图为Oracle支持各个JDK版本所到的年限

<img src="oracleJDK vs OpenJDK.png">

