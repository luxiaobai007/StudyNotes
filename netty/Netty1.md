---
Netty入门学习(一)
---

[toc]

# Netty介绍和应用场景

## 介绍

1. Netty 是由 JBOSS 提供的一个 Java 开源框架，现为 Github 上的独立项目。
2.  Netty 是一个异步的、基于事件驱动的网络应用框架，用以快速开发高性能、高可靠性的网络 IO 程序。
3.  Netty 主要针对在 TCP 协议下，面向 Clients 端的高并发应用，或者 Peer-to-Peer 场景下的大量数据持续传输的应用。
4.  Netty 本质是一个 NIO 框架，适用于服务器通讯相关的多种应用场景



## 应用场景

### 互联网行业

- 互联网行业：在分布式系统中，各个节点之间需要远程服务调用，高性能的 RPC 框架必不可少，Netty 作为异步高性能的通信框架，往往作为基础通信组件被这些 RPC 框架使用。
- 典型的应用有：阿里分布式服务框架 Dubbo 的 RPC 框架使用 Dubbo 协议进行节点间通信，Dubbo 协议默认使用 Netty 作为基础通信组件，用于实现各进程节点之间的内部通信



### 游戏行业

- 无论是手游服务端还是大型的网络游戏，Java 语言得到了越来越广泛的应用
-  Netty 作为高性能的基础通信组件，提供了 TCP/UDP 和 HTTP 协议栈，方便定制和开发私有协议栈，账号登录服务器
-  地图服务器之间可以方便的通过 Netty 进行高性能的通信



### 大数据领域

经典的 Hadoop 的高性能通信和序列化组件 Avro 的 RPC 框架，默认采用 Netty 进行跨界点通信

 它的 Netty Service 基于 Netty 框架二次封装实现。





# Java BIO编程

## IO模型

1. I/O 模型简单的理解：就是用什么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能

2.  Java 共支持 3 种网络编程模型/IO 模式：BIO、NIO、AIO

3.  Java BIO ： **同步并阻塞(传统阻塞型)**，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销

   ![bio1](http://qiliu.luxiaobai.cn/img/bio1.png)

4. Java NIO ： **同步非阻塞**，服务器实现模式为一个线程处理多个请求(连接)，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有 I/O 请求就进行处理 

   ![bio2](http://qiliu.luxiaobai.cn/img/bio2.png)

5. Java AIO(NIO.2) ： **异步非阻塞**，AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用



## **BIO、NIO、AIO 适用场景分析**

1. BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。
2.  NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK1.4 开始支持。
3. AIO 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用 OS 参与并发操作，编程比较复杂，JDK7 开始支持



## Java BIO工作机制

### BIO编程简单流程

1. 服务器端启动一个 ServerSocket
2.  客户端启动 Socket 对服务器进行通信，默认情况下服务器端需要对每个客户 建立一个线程与之通讯
3. 客户端发出请求后, 先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝
4.  如果有响应，客户端线程会等待请求结束后，在继续执行



### BIO实例

```java
public class BIOServer {
    public static void main(String[] args) throws IOException {
        //线程池机制

//        思路
//        1 创建一个线程池
//        2 如果有客户端连接，就创建一个线程，与之通讯（单独写一个方法
        ExecutorService executorService = Executors.newCachedThreadPool();
        //创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);

        while (true) {
            System.out.println("等待连接。。。。。");
            final Socket socket = serverSocket.accept();
            System.out.println("连接一个客户端");

            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    handler(socket);
                }
            });
        }
    }

    public static void handler(Socket socket) {
        try {
            System.out.println("线程信息ID=" + Thread.currentThread().getId() + " 名字：" + Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            //通过socket获取输入流
            InputStream inputStream = socket.getInputStream();
            //循环读取客户端发送的数据
            while (true) {
                int read = inputStream.read(bytes);
                System.out.println("read....");
                if (read != -1) {
                    System.out.println(new String(bytes, 0, read));//输出客户端发送的数据
                } else {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```



### BIO问题分析

1. 每个请求都需要创建独立的线程，与对应的客户端进行数据 Read，业务处理，数据 Write 
2. 当并发数较大时，需要创建大量线程来处理连接，系统资源占用较大。
3. 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在 Read 操作上，造成线程资源浪费



# Java NIO

## 介绍

1. Java NIO 全称 java non-blocking IO，是指 JDK 提供的新 API。从 JDK1.4 开始，Java 提供了一系列改进的输入/输出的新特性，被统称为 NIO(即 New IO)，是同步非阻塞的
2.  NIO 相关类都被放**nn**在 java.nio 包及子包下，并且对原 java.io 包中的很多类进行改写。
3.  NIO 有三大核心部分：**Chael(通道)**，**Buffer(缓冲区)**, **Selector(选择器)**
4.  NIO 是 **面向缓冲区 ，或者面向 块 编程的**。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络
5.  Java NIO 的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
6. 通俗理解：NIO 是可以做到用一个线程来处理多个操作的。假设有 10000 个请求过来,根据实际情况，可以分配50 或者 100 个线程来处理。不像之前的阻塞 IO 那样，非得分配 10000 个。
7. HTTP2.0 使用了**多路复用**的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比 HTTP1.1 大了好几个数量级



## NIO和BIO的比较

1. BIO 以**流的方式**处理数据,而 NIO 以**块的方式**处理数据,块 I/O 的效率比流 I/O 高很多
2. BIO 是阻塞的，NIO 则是非阻塞的
3. BIO 基于**字节流**和**字符流**进行操作，而 NIO 基于 **Channel(通道)**和 **Buffer(缓冲区)**进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择器)用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用**单个线程就可以监听多个客户端通道**



## NIO三大核心原理

### **Selector、Channel、Buffer**

1. 每个 channel 都会对应一个 Buffer

2. Selector 对应一个线程， 一个线程对应多个 channel(连接)

3. 该图反应了有三个 channel 注册到 该 selector //程序

   ![nio1](http://qiliu.luxiaobai.cn/img/nio1.png)

4. 程序切换到哪个 channel 是有事件决定的, Event 就是一个重要的概念

5. Selector 会根据不同的事件，在各个通道上切换

6. Buffer 就是一个内存块 ， 底层是有一个数组

7. 数据的读取写入是通过 Buffer, 这个和 BIO , BIO 中要么是输入流，或者是输出流, 不能双向，但是 NIO 的 Buffer 是可以读也可以写, 需要 flip 方法切换channel 是双向的, 可以返回底层操作系统的情况, 比如 Linux ， 底层的操作系统通道就是双向的.



### Buffer(缓冲区)

```java
public class BasicBuffer {
    public static void main(String[] args) {
//        举例说明buffer的使用（简单说明）
//        创建一个Buffer，大小为5，即可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);
        //向buffer 存放数据
        for (int i = 0; i < intBuffer.capacity(); i++) {
            intBuffer.put(i * 2);
        }
        //如何从buffer读取数据
        //flip 将buffer转换，读写切换
        /**
         *   public final Buffer flip() {
         *         limit = position;
         *         position = 0;
         *         mark = -1;
         *         return this;
         *     }
         */
        intBuffer.flip();
        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }
}
```

>**介绍**
>
>缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象(含数组)，该对象提供了一组方法，可以更轻松地使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer.

![nio2](http://qiliu.luxiaobai.cn/img/nio2.png)



#### **Buffer类及其子类**

![nio3](http://qiliu.luxiaobai.cn/img/nio3.png)



#### **Buffer类相关方法**

![nio4](http://qiliu.luxiaobai.cn/img/nio4.png)



#### **ByteBuffer**

![nio5](http://qiliu.luxiaobai.cn/img/nio5.png)





### Channel(通道)

>**介绍**
>
>1. NIO 的通道类似于流，但有些区别如下：
>
>2. 1.  通道可以同时进行读写，而流只能读或者只能写
>   2.  通道可以实现异步读写数据
>   3.  通道可以从缓冲读数据，也可以写数据到缓冲:
>
>3.  BIO 中的 stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道(Channel)是双向的，可以读操作，也可以写操作。
>
>4.  Channel 在 NIO 中是一个接口
>
>- public interface Channel extends Closeable{}	
>
>1.  常 用 的 Channel 类 有 ： FileChannel 、 DatagramChannel 、 ServerSocketChannel 和 SocketChannel 。【ServerSocketChanne 类似 ServerSocket , SocketChannel 类似 Socket】
>2. FileChannel 用于文件的数据读写，DatagramChannel 用于 UDP 的数据读写，ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。



#### FileChannel类

- public int read(ByteBuffer dst) ，从通道读取数据并放到缓冲区中
- public int write(ByteBuffer src) ，把缓冲区的数据写到通道中
- public long transferFrom(ReadableByteChannel src, long position, long count)，从目标通道中复制数据到当前通道
- public long transferTo(long position, long count, WritableByteChannel target)，把数据从当前通道复制给目标通道



#### 实例1-本地文件获取

```java
public class NIOFileChannel01 {
    public static void main(String[] args) throws Exception {
        String str = "hello,luxiaobai";

        //创建一个输出流->channel
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/lushengyang/Desktop/Demo/test1.txt");

//      通过fileOutputStream获取对应的FileChannel
//      这个fileChannel真实类型是FileChannelImpl
        FileChannel fileChannel = fileOutputStream.getChannel();

        //创建一个缓冲区ByteBuffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        //将str放入byteBuffer
        byteBuffer.put(str.getBytes());
//      对byteBuffer，进行flip
        byteBuffer.flip();
//      将byteBuffer,数据写入到fileChannel
        fileChannel.write(byteBuffer);
        fileOutputStream.close();
    }
}
```





#### 实例2-本地文件读数据

```java
public class NIOFileChannel02 {
    public static void main(String[] args) throws Exception {
        //创建文件输入流
        File file = new File("/Users/lushengyang/Desktop/Demo/test1.txt");
        FileInputStream fileInputStream = new FileInputStream(file);
        //通过fileInputStream 获取对应的File Channel->实际类型 FileChannelImpl
        FileChannel fileChannel = fileInputStream.getChannel();
        //创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());
        //将通道数据读入到buffer
        fileChannel.read(byteBuffer);
        //将byteBuffer的字节数据转成string
        System.out.println(new String(byteBuffer.array()));
        fileInputStream.close();
    }
}
```



#### 实例3-使用一个Buffer完成文件读取

```java
public class NIOFileChannel03 {
    public static void main(String[] args) throws Exception {
        FileInputStream fileInputStream = new FileInputStream("1.txt");
        FileChannel fileChannel01 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("2.txt");
        FileChannel fileChannel02 = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(512);

        while (true){
            //这里有一个重要操作，不要忘了
            /**
             *  public final Buffer clear() {
             *         position = 0;
             *         limit = capacity;
             *         mark = -1;
             *         return this;
             *     }
             */
//            byteBuffer.clear();
            int read = fileChannel01.read(byteBuffer);
            System.out.println("read" + read);
            if(read ==-1){
                break;
            }
            //将buffer中的数据写入channel
            byteBuffer.flip();
            fileChannel02.write(byteBuffer);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
}
```



#### 实例4-拷贝文件transferFrom方法

```java
public class NIOFileIChannel04 {
    public static void main(String[] args) throws Exception {
        //创建相关的流
        FileInputStream fileInputStream = new FileInputStream("/Users/lushengyang/Desktop/1.png");
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/lushengyang/Desktop/2.png");

        //获取各个流对应的fileChannel
        FileChannel channel = fileInputStream.getChannel();
        FileChannel channel1 = fileOutputStream.getChannel();

        //使用transferForm完成拷贝
        channel1.transferFrom(channel,0,channel.size());
        //关闭相关通道和流
        channel.close();
        channel1.close();
        fileInputStream.close();
        fileOutputStream.close();
    }
```



#### 关于Buffer和Channel的注意事项和细节

1. ByteBuffer 支持类型化的 put 和 get, put 放入的是什么数据类型，get 就应该使用相应的数据类型来取出，否则可能有 BufferUnderflowException 异常。

```java
public class NIOByteBufferPutGet {

    public static void main(String[] args) {
        //创建一个Buffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(64);

        //类型化方式放入数据
        byteBuffer.putInt(100);
        byteBuffer.putLong(9);
        byteBuffer.putChar('路');
        byteBuffer.putShort((short) 4);

        //取出
        byteBuffer.flip();

        System.out.println();
        System.out.println(byteBuffer.getInt());
        System.out.println(byteBuffer.getLong());
        System.out.println(byteBuffer.getChar());
        System.out.println(byteBuffer.getShort());
    }
}
```

​	2.可以将一个普通 Buffer 转成只读 Buffer

```java
public class ReadOnlyBuffer {
    public static void main(String[] args) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(64);
        for (int i=0;i<64;i++){
            byteBuffer.put((byte) i);
        }
        byteBuffer.flip();

        //得到一个只读的buffer
        ByteBuffer readOnlyBuffer = byteBuffer.asReadOnlyBuffer();
        System.out.println(readOnlyBuffer.getClass());///class java.nio.HeapByteBufferR

        //读取
        while (readOnlyBuffer.hasRemaining()){
            System.out.println(readOnlyBuffer.get());
        }
        readOnlyBuffer.put((byte) 100);//
    }
}

Exception in thread "main" java.nio.ReadOnlyBufferException
	at java.nio.HeapByteBufferR.put(HeapByteBufferR.java:172)
	at com.luxiaobai.nio.ReadOnlyBuffer.main(ReadOnlyBuffer.java:29)
```

​	3.NIO 还提供了 MappedByteBuffer， 可以让文件直接在内存（堆外的内存）中进行修改， 而如何同步到文件由 NIO 来完成.

```java
/**
 * @author ：luxiaobai
 * @date ：Created in 2021/7/30 22:01
 * @description：
 * @modified By：`
 * @version: 1.0
 * 1.MappedByteBuffer 可让文件直接在内存修改，操作系统不需要拷贝一次
 */

public class MappedByteBufferTest {
    public static void main(String[] args) throws Exception {
        RandomAccessFile randomAccessFile = new RandomAccessFile("1.txt", "rw");
        FileChannel channel = randomAccessFile.getChannel();

        /**
         * 参数1：FileChannel.MapMode.READ_WRITE 使用的读写模式
         * 参数2：0：可以直接修改的起始位置
         * 参数3：5：是映射到内存的大小(不是索引位置)，即将1.txt的多少个字节映射到内存
         * 可以直接修改的范围是0-5
         */
        MappedByteBuffer map = channel.map(FileChannel.MapMode.READ_WRITE, 0, 5);
        map.put(0,(byte) 'H');
        map.put(3,(byte) '9');
        map.put(5,(byte) 'Y');//Exception in thread "main" java.lang.IndexOutOfBoundsException
        randomAccessFile.close();
        System.out.println("修改成功。。。");
    }
}
```

​	4.前面我们讲的读写操作，都是通过一个 Buffer 完成的，NIO 还支持 通过多个 Buffer (即 Buffer 数组) 完成读写操作，即 Scattering 和 Gathering 【举例说明】

```java
/**
 * Scattering:将数据写入到buffer时，可以采用buffer数组，依次写入
 * Gathering:从buffer读取数据时，可以采用buffer数组，依次读
 */
public class ScatteringAndGatheringTest {
    public static void main(String[] args) throws IOException {
        //使用ServerSocketChannel和SocketChannel 网络
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(7000);

        //绑定断开到socket,并启动
        serverSocketChannel.socket().bind(inetSocketAddress);

        //创建buffer数组
        ByteBuffer[] byteBuffers = new ByteBuffer[2];
        byteBuffers[0] = ByteBuffer.allocate(5);
        byteBuffers[1] = ByteBuffer.allocate(3);

        //等待客户端连接
        SocketChannel socketChannel = serverSocketChannel.accept();
        int messageLenght = 8;  //假定从客户端接收8个字节
        //循环读取
        while (true) {
            int byteRead = 0;
            while (byteRead < messageLenght) {
                long l = socketChannel.read(byteBuffers);
                byteRead += l;//累计读取的字节数
                System.out.println("byteRead=" + byteRead);
                //使用流打印，看看当前的这个buffer的position和limit
                Arrays.asList(byteBuffers).stream().map(buffer -> "position=" + buffer.position() + ", limit=" + buffer.limit()).forEach(System.out::println);
            }

            //将所有的buffer进行filp
            Arrays.asList(byteBuffers).forEach(buffer -> buffer.flip());

            //将数据读出显示到客户端
            long byteWrite = 0;
            while (byteWrite < messageLenght) {
                long l = socketChannel.write(byteBuffers);
                byteWrite += l;
            }

            //将所有的buffer进行clear
            Arrays.asList(byteBuffers).forEach(buffer -> {
                buffer.clear();
            });

            System.out.println("byteRead=" + byteRead + " byteWrite=" + byteWrite + ", messageLength=" + messageLenght);
        }
    }
}
```



## **Selector(选择器)**

### **介绍**

1. Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连接，就会使用Selector(选择器)
2.  Selector 能够检测多个注册的通道上是否有事件发生(注意:多个 Channel 以事件的方式可以注册到同一个Selector)，如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。【示意图】
3. 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程
4. 避免了多线程之间的上下文切换导致的开销

### **Select特点说明**

1. Netty 的 IO 线程 NioEventLoop 聚合了 Selector(选择器，也叫多路复用器)，可以同时并发处理成百上千个客户端连接。
2. 当线程从某客户端 Socket 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。
3. 由于读写操作都是非阻塞的，这就可以充分提升 IO 线程的运行效率，避免由于频繁 I/O 阻塞导致线程挂起。
4. 一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升

### **Select类相关方法**

1. selector.select()//阻塞
2. selector.select(1000);//阻塞 1000 毫秒，在 1000 毫秒后返回
3. selector.wakeup();//唤醒 selector
4. selector.selectNow();//不阻塞，立马返还

### **NIO非阻塞网络编程原理分析图**

![nio6](http://qiliu.luxiaobai.cn/img/nio6.png)

1. 当客户端连接时，会通过 ServerSocketChannel 得到 SocketChannel
2. Selector 进行监听 select 方法, 返回有事件发生的通道的个数.
3. 将 socketChannel 注册到 Selector 上, register(Selector sel, int ops), 一个 selector 上可以注册多个 SocketChannel
4. 注册后返回一个 SelectionKey, 会和该 Selector 关联(集合)
5. 进一步得到各个 SelectionKey (有事件发生)
6. 在通过 SelectionKey 反向获取 SocketChannel , 方法 channel()
7. 可以通过得到的 channel , 完成业务处理



#### 实例-NIOServer.java

```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
        //创建ServerSocketChannel -> ServerSocket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        //得到一个Selector对象
        Selector selector = Selector.open();

        //绑定一个端口
        serverSocketChannel.socket().bind(new InetSocketAddress(6666));
        //设置非阻塞
        serverSocketChannel.configureBlocking(false);

        //把serverSocketChannel注册到selector 关心事件为 OP_ACCEPT
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        //循环等待客户端连接
        while (true) {
            if (selector.select(1000) == 0) {
                System.out.println("服务器等待了1秒，无连接");
                continue;
            }

            //如果返回的>0,就获取到相关的selectKey集合
            //1.如果返回的>0,表示已经获取到关注的事件
            //2. select.selectoryKeys()返回关注事件的集合
            //通过selectionKeys 反向获取通道
            Set<SelectionKey> selectionKeys = selector.selectedKeys();

            //遍历Set<SelectionKey> ，使用迭代器遍历
            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
            while (keyIterator.hasNext()) {
                //获取到selectorKey
                SelectionKey key = keyIterator.next();
                //根据key 对应的通道发生的事件做相应的处理
                if (key.isAcceptable()) {//如果是OP_ACCEPT，有新的客户端连接
                    //给该客户端生成一个SocketChannel
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("客户端连接成功，生成一个socketChannel:" + socketChannel.hashCode());
                    //SocketChannel设置为非阻塞
                    socketChannel.configureBlocking(false);
                    //将socketChannel注册到selector,关注事件为OP_READ，同时给scoektChannel
                    //关联一个Buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }
                if (key.isReadable()) { //发生OP_READ
                    //通过key反向获取到对应channel
                    SocketChannel channel = (SocketChannel) key.channel();
                    //获取到该channel关联的buffer
                    ByteBuffer buffer = (ByteBuffer) key.attachment();
                    channel.read(buffer);
                    System.out.println("from 客户端 " + new String(buffer.array()));

                }

                //手动从集合中移动当前的SelectionKey,防止重复操作
                keyIterator.remove();
            }
        }
    }
}
```

#### NIOClient.java

```java
public class NIOClient {
    public static void main(String[] args) throws IOException {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 6666);
        if(!socketChannel.connect(inetSocketAddress)){
            while (!socketChannel.finishConnect()){
                System.out.println("因为连接需要时间，客户端不会阻塞，可以做其他工作");
            }
        }

        //如果连接成功，就发送数据
        String str = "hello, luxiaobai";
        ByteBuffer buffer = ByteBuffer.wrap(str.getBytes());
        socketChannel.write(buffer);
        System.in.read();
    }
}
```



### **SelectionKey**

SelectionKey，**表示 Selector 和网络通道的注册关系**

int OP_ACCEPT：有新的网络连接可以 accept，值为 16

int OP_CONNECT：代表连接已经建立，值为 8

int OP_READ：代表读操作，值为 1

int OP_WRITE：代表写操作，值为 4

源码中：

public static final int OP_READ = 1 << 0;

public static final int OP_WRITE = 1 << 2;

public static final int OP_CONNECT = 1 << 3;

public static final int OP_ACCEPT = 1 << 4;



#### **SelectionKey相关方法**

![nio7](http://qiliu.luxiaobai.cn/img/nio7.png)



### ServerSocketChannel

在服务器端监听新的客户端 Socket 连接

**相关方法**

![nio8](http://qiliu.luxiaobai.cn/img/nio8.png)



### **SocketChannel**

SocketChannel，网络 IO 通道，具体负责进行读写操作。NIO 把缓冲区的数据写入通道，或者把通道里的数据读到缓冲区。

**相关方法**

![nio9](http://qiliu.luxiaobai.cn/img/nio9.png)



## NIO网络编程实例--群聊系统

**GroupChatServer.java**

```java
public class GroupChatServer {
    //定义属性
    private Selector selector;
    private ServerSocketChannel serverSocketChannel;
    private static final int port = 6667;

    //构造器
    //初始化工作
    public GroupChatServer() {
        try {
            //得到选择器
            selector = Selector.open();
            //ServerSocketChannel
            serverSocketChannel = ServerSocketChannel.open();
            //绑定端口
            serverSocketChannel.socket().bind(new InetSocketAddress(port));
            //设置非阻塞模式
            serverSocketChannel.configureBlocking(false);
            //将serverSocketChannel组册到selector
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //监听
    public void listen() {
        try {
            while (true) {
                int count = selector.select();
                if (count > 0) {//有事件处理
                    //遍历得到SelectionKeys集合
                    Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                    while (keyIterator.hasNext()) {
                        //取出SelectionKey
                        SelectionKey key = keyIterator.next();
                        //监听到accept
                        if (key.isAcceptable()) {
                            SocketChannel socketChannel = serverSocketChannel.accept();
                            //将socketChannel设置为非阻塞
                            socketChannel.configureBlocking(false);
                            //将socketChannel注册到selector
                            socketChannel.register(selector, SelectionKey.OP_READ);
                            //上线提示
                            System.out.println(socketChannel.getRemoteAddress() + " 上线");
                        }
                        if (key.isReadable()) {//通道发生read事件，即通道是可读的状态
                            //处理读的事件
                            readData(key);
                        }
                        //当前的 key 删除，防止重复处理
                        keyIterator.remove();
                    }
                } else {
                    System.out.println("waiting...");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //发生异常处理
        }
    }

    //读取客户端信息
    public void readData(SelectionKey key) {
        SocketChannel socketChannel = null;
        try {
            socketChannel = (SocketChannel) key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int count = socketChannel.read(buffer);
            if (count > 0) {
                //把缓冲区的数据转成字符串
                String msg = new String(buffer.array());
                System.out.println("from 客户端： " + msg);

                //向其他客户端转发信息（去掉自己）
                sendInfoToOtherClients(msg, socketChannel);
            }
        } catch (IOException e) {
            try {
                System.out.println(socketChannel.getRemoteAddress() + " 离线了...");
                //取消注册
                key.cancel();
                //关闭通道
                socketChannel.close();
            } catch (IOException e2) {
                e.printStackTrace();
            }
        }
    }


    //转发信息给其他客户端（通道）
    public void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {
        System.out.println("服务器转发信息中...");
        //遍历所有注册到selector上的SocketChannel，并排除self
        for (SelectionKey key : selector.keys()) {
            //key->Channel
            Channel channel = key.channel();
            if (channel instanceof SocketChannel && channel != self) {
                //转型
                SocketChannel socketChannel = (SocketChannel) channel;
                //将msg存储到buffer
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                ///将buffer写入通道
                socketChannel.write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }
}
```

GroupClient.java

```java
public class GroupClient {
    private final String HOST = "127.0.0.1";
    private final int PORT = 6667;
    private Selector selector;
    private SocketChannel socketChannel;
    private String userName;

    public GroupClient() throws IOException {
        selector = Selector.open();
        socketChannel = SocketChannel.open(new InetSocketAddress(HOST,PORT));
//        socketChannel.open(new InetSocketAddress(HOST,PORT));
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);
        userName = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(userName + " is ok ...");
    }

    //客户端向服务器发送信息
    public void sendInfo(String info){
        info = userName + " 说： " + info;
        try{
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        }catch (IOException e){
            e.printStackTrace();
        }
    }
    
    //读取从服务器端回复的信息
    public void readInfo(){
        try{
            int readChannels = selector.select();
            if(readChannels>0){//有可用的通道
                Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                while (keyIterator.hasNext()){
                    SelectionKey key = keyIterator.next();
                    if (key.isReadable()){
                        //得到相关通道
                        SocketChannel channel = (SocketChannel) key.channel();
                        //得到一个buffer
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        //读取
                        channel.read(buffer);
                        //把读到的缓冲区的数据转成字符串
                        String msg = new String(buffer.array());
                        System.out.println(msg.trim());
                    }
                }
                keyIterator.remove();//删除当前的selectionKey，防止重复操作
            }else {
                System.out.println("没有可用的通道");
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception {
        GroupClient chatClient = new GroupClient();

        new Thread(){
            public void run(){
                while (true){
                    chatClient.readInfo();
                    try{
                        Thread.currentThread().sleep(3000);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        }.start();

        //发送数据给服务器端
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String msg = scanner.nextLine();
            chatClient.sendInfo(msg);
        }
    }
}
```



























































































































































































































































































































































































































