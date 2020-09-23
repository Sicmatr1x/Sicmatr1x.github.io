---
title: Java NIO
date: 2020-07-25 10:25:07
categories:
    - Java
tags: 
    - 后端
---

Java NIO(New IO或Non Bloking IO)是从Java1.4版本开始引入的一个新的IO API，NIO支持面向缓冲区的、基于通道的IO操作，读写文件更加高效。

## Java NIO

#### 区别

- IO
  - 面向流(Stream Oriented)：相当于水管，一个steam一个方向出水的水管
  - 阻塞IO(Blocking IO)
  - 无选择器
- NIO
  - 面向缓冲区(Buffer Oriented)：相当于铁路，一条铁路上面有一个缓冲区可以装数据然后双向运输
  - 非阻塞IO(Non Blocking IO)
  - 选择器(Selectors)

NIO系统的核心：通道表示打开到IO设备的连接。若需要使用NIO系统，需要获取用于连接IO设备的通道以及用于容纳数据的缓冲区，然后操作缓冲区，对数据进行处理。

### 缓冲区(Buffer)

1. 缓冲区(Buffer):在NIO中复制数据的春秋，缓冲区就是数组。
  - 根据数据类型的不同可以分配对应的缓冲区，boolean类型除外


2. 缓冲区存取数据的两个核心方法：
- put()
- get()


3. 缓冲区四个核心属性
   1. int capacity(容量): 表示缓冲区中最大存储数据的容量。一旦声明不能改变。
   2. limit: 界限，表示缓冲区中可以操作数据的大小，即limit后的数据不能进行读写
   3. position: 位置，表示缓冲区中正在操作数据的位置，且需满足(position <= limit <= capacity)
   4. mark: 标记，标记当前position的位置，可以通过`reset()`恢复到mark的位置

这几个属性位于`java.nio.Buffer`类里

```java
public abstract class Buffer {

    /**
     * The characteristics of Spliterators that traverse and split elements
     * maintained in Buffers.
     */
    static final int SPLITERATOR_CHARACTERISTICS =
        Spliterator.SIZED | Spliterator.SUBSIZED | Spliterator.ORDERED;

    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```

### 实例代码

```java
    public void test1() {
        String str = "abcde";
        // 分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        System.out.println(buf.position()); // 0
        System.out.println(buf.limit()); // 1024
        System.out.println(buf.capacity()); // 1024
        // 写数据
        buf.put(str.getBytes());
        System.out.println(buf.position()); // 5
        System.out.println(buf.limit()); // 1024
        System.out.println(buf.capacity()); // 1024
        // 切换读取数据模式
        buf.flip();
        System.out.println(buf.position()); // 0
        System.out.println(buf.limit()); // 5
        System.out.println(buf.capacity()); // 1024
        // 读数据
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst, 0, dst.length)); // abcd
        System.out.println(buf.position()); // 5
        System.out.println(buf.limit()); // 5
        System.out.println(buf.capacity()); // 1024
        // rewind() 可重复读数据
        buf.rewind();
        System.out.println(buf.position()); // 0
        System.out.println(buf.limit()); // 5
        System.out.println(buf.capacity()); // 1024
        // clear() 清空缓冲区，但是数据依然存在且处于被遗忘状态
        buf.clear();
        System.out.println(buf.position()); // 0
        System.out.println(buf.limit()); // 1024
        System.out.println(buf.capacity()); // 1024
    }
```

```java
    public void test2() {
        String str = "abcde";
        ByteBuffer buf = ByteBuffer.allocate(1024);
        buf.put(str.getBytes());
        buf.flip();
        byte[] dst = new byte[buf.limit()];
        buf.get(dst, 0, 2);
        System.out.println(new String(dst, 0, 2));
        System.out.println(buf.position()); // 2
        buf.mark();
        buf.get(dst, 2, 2);
        System.out.println(new String(dst, 0, 2));
        System.out.println(buf.position()); // 4
        buf.reset();
        System.out.println(buf.position()); // 2
    }
```

4. 直接缓冲区与非直接缓冲区

- 非直接缓冲区：通过`allocate()`方法分配的缓冲区，缓冲区建立在JVM的内存中
- 直接缓冲区：通过`allocateDirect()`方法分配的直接缓冲区或者使用FileChannel的map()方法返回MappedByteBuffer对象，将缓冲区建立在物理内存中。可以提高效率


### 通道(Channel)

用于源节点和目标节点间的连接，在NIO中复制缓冲区中数据的传输。Channel本身不存储数据，需要配合缓冲区使用。

#### 通道主要实现类

- java.nio.channels.CHannel
  - FileChannel
  - SocketChannel
  - ServerSocketChannel
  - DatagramChannel

#### 获取通道的几种方式

1. Java针对支持通道的类提供了`getChannel()`方法用于获取对应的通道
  - 本地IO
    - FileInputStream
    - FileOutputStream
    - RandomAccessFile
  - 网络IO
    - Socket
    - ServerSocket
    - DatagramSocket
2. JDK1.7提供的NIO.2针对各个通道提供了静态方法`open()`
3. JDK1.7中的NIO.2的Files工具类的`newByteChannel()`方法


#### 通道使用案例

```java
    /**
     * 直接缓冲区：利用通道实现本地文件复制(内存映射文件)
     */
    @Test
    public void test2() throws IOException {
        FileChannel inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("3.png"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);
        // 内存映射文件
        MappedByteBuffer inMappedBuffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
        MappedByteBuffer outMappedBuffer = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());
        // 直接对缓冲区进行数据的读写操作
        byte[] dst = new byte[inMappedBuffer.limit()];
        inMappedBuffer.get(dst);
        outMappedBuffer.put(dst);

        inChannel.close();
        outChannel.close();
    }

    @Test
    public void test3() throws IOException {
        FileChannel inChannel = FileChannel.open(Paths.get("1.png"), StandardOpenOption.READ);
        FileChannel outChannel = FileChannel.open(Paths.get("3.png"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW);

        inChannel.transferTo(0, inChannel.size(), outChannel);

        inChannel.close();
        outChannel.close();
    }

```


#### 分散(Scatter)于聚集(Gather)

- 分散读取(Scattering Reads): 将通道中的数据分散到多个缓冲区中
- 聚集写入(Gathering Writes): 将多个缓冲区中的数据聚集到通道中

```java
    public void test4() throws IOException {
        RandomAccessFile raf1 = new RandomAccessFile("1.txt", "r");
        // 获取通道
        FileChannel fileChannel1 = raf1.getChannel();
        // 分配多个缓冲区
        ByteBuffer buf1 = ByteBuffer.allocate(100);
        ByteBuffer buf2 = ByteBuffer.allocate(1024);
        // 分散读取
        ByteBuffer[] bufs = {buf1, buf2};
        fileChannel1.read(bufs);

        for (ByteBuffer byteBuffer : bufs) {
            byteBuffer.flip();
        }
        System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));

        // 聚集写入
        RandomAccessFile raf2 = new RandomAccessFile("2.txt", "rw");
        FileChannel fileChannel2 = raf2.getChannel();
        fileChannel2.write(bufs);

        fileChannel1.close();
        fileChannel2.close();
    }
```

#### 字符集(Charset)

```java
    public void test6() throws CharacterCodingException {
        Charset cs1 = Charset.forName("GBK");
        // 获取编码器与解码器
        CharsetEncoder ce = cs1.newEncoder();
        CharsetDecoder cd = cs1.newDecoder();

        CharBuffer buffer = CharBuffer.allocate(1024);
        buffer.put("获取编码器与");
        buffer.flip();

        // 编码
        ByteBuffer bBuf = ce.encode(buffer);

        for (int i = 0; i < 12; i++) {
            System.out.println(bBuf.get());
        }

        // 解码
        bBuf.flip();
        CharBuffer cBuf = cd.decode(bBuf);
        System.out.println(cBuf.toString());
    }
```

### 阻塞式网络通信

使用NIO完成网络通信需要：
1. 通道(Channel): 负责连接
   - java.nio.channels.Channel接口
     - SelectableChannel
       - SocketChannel
       - ServerSocketChannel
       - DatagramChannel
       - Pipe.SinkChannel
       - Pipe.SourceChannel
2. 缓冲区(Buffer): 负责数据的存取
3. 选择器(Selector): 是SelectableChannel的多路复用器。用于监控SelectableChannel的IO状况

### 非阻塞式网络通信

#### 使用ServerSocketChannel

```java
package com.sicmatr1x.nio;

import org.junit.Test;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.time.LocalDateTime;
import java.util.Iterator;
import java.util.Scanner;

public class TestNonBlockingNIO {
    @Test
    public void client() throws IOException {
        SocketChannel sChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9898));
        // 切换成非阻塞模式
        sChannel.configureBlocking(false);
        // 分配缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        // 发送数据
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNext()){
            String str = scanner.next();
            buf.put((LocalDateTime.now().toString() + ":" + str).getBytes());
            buf.flip();
            sChannel.write(buf);
            buf.clear();
        }

        sChannel.close();
    }

    @Test
    public void server() throws IOException {
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        // 切换成非阻塞模式
        ssChannel.configureBlocking(false);
        // 绑定连接
        ssChannel.bind(new InetSocketAddress(9898));
        // 获取选择器
        Selector selector = Selector.open();
        // 将通道注册到选择器，并且指定监听事件
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
        // 轮询式的获取选择器上已经准备就绪的事件
        while (selector.select() > 0) {
            // 获取当前选择器中所有注册的且已就绪的选择键
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            // 迭代
            while(iterator.hasNext()) {
                // 获取准备就绪的事件
                SelectionKey key = iterator.next();
                // 判断是什么事件就绪
                if (key.isAcceptable()) { // 若为接收就绪
                    SocketChannel sChannel = ssChannel.accept();
                    sChannel.configureBlocking(false);
                    sChannel.register(selector, SelectionKey.OP_READ);

                } else if (key.isReadable()) {
                    // 获取当前选择器上读就绪的通道
                    SocketChannel sChannel = (SocketChannel)key.channel();

                    ByteBuffer buffer = ByteBuffer.allocate(1024);

                    int len = 0;
                    while((len = sChannel.read(buffer)) > 0) {
                        buffer.flip();
                        System.out.println(new String(buffer.array(), 0, len));
                        buffer.clear();
                    }
                }
                // 取消选择键
                iterator.remove();
            }
        }
    }
}

```

#### 使用DatagramChannel

```java
package com.sicmatr1x.nio;

import org.junit.Test;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.util.Date;
import java.util.Iterator;
import java.util.Scanner;

public class TestNonBlockingNIO2 {
    @Test
    public void send() throws IOException {
        DatagramChannel dc = DatagramChannel.open();

        dc.configureBlocking(false);

        ByteBuffer buffer = ByteBuffer.allocate(1024);

        Scanner scan = new Scanner(System.in);

        while(scan.hasNext()) {
            String str = scan.next();
            buffer.put((new Date().toString() + ":" + str).getBytes());
            dc.send(buffer, new InetSocketAddress("127.0.0.1", 9898));
            buffer.clear();
        }
        dc.close();
    }

    @Test
    public void receive() throws IOException {
        DatagramChannel dc = DatagramChannel.open();

        dc.configureBlocking(false);

        dc.bind(new InetSocketAddress(9898));

        Selector selector = Selector.open();

        dc.register(selector, SelectionKey.OP_READ);

        while(selector.select() > 0) {
            Iterator<SelectionKey> it = selector.selectedKeys().iterator();
            while(it.hasNext()) {
                SelectionKey key = it.next();
                if(key.isReadable()) {
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    dc.receive(buffer);
                    buffer.flip();
                    System.out.println(new String(buffer.array(), 0, buffer.limit()));
                }
            }
            it.remove();
        }
    }
}

```

### 管道

```java
package com.sicmatr1x.nio;

import org.junit.Test;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.Pipe;

public class TestPip {
    @Test
    public void test1() throws IOException {
        // 获取管道
        Pipe pipe = Pipe.open();
        // 将缓冲区中的数据写入管道
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        Pipe.SinkChannel sinkChannel = pipe.sink();
        buffer.put("通过管道发送数据".getBytes());
        buffer.flip();
        sinkChannel.write(buffer);

        // 读取数据
        ByteBuffer buffer1 = ByteBuffer.allocate(1024);
        Pipe.SourceChannel sourceChannel = pipe.source();
        int len = sourceChannel.read(buffer1);
        System.out.println(new String(buffer1.array(), 0, len));

        sourceChannel.close();
        sinkChannel.close();
    }
}

```