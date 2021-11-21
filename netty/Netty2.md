---
Netty入门学习(二)
---

[toc]

# NIO与零拷贝

零拷贝是网络编程的关键，很多性能优化都离不开。

在 Java 程序中，常用的零拷贝有 mmap(内存映射) 和 sendFile。



## 传统IO数据读写

```java
File file = new File("test.txt");
RandomAccessFile raf = new RandomAccessFile(file, "rw");

byte[] arr = new byte[(int) file.length()];
raf.read(arr);

Socket socket = new ServerSocket(8080).accept();
socket.getOutputStream().write(arr);
```



### mmap优化

mmap 通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户空间的拷贝次数。



### sendFile优化

1. Linux 2.1 版本 提供了 sendFile 函数，其基本原理如下：数据根本不经过用户态，直接从内核缓冲区进入到Socket Buffer，同时，由于和用户态完全无关，就减少了一次上下文切换
2. 提示：零拷贝从操作系统角度，是没有 cpu 拷贝
3. Linux 在 2.4 版本中，做了一些修改，避免了从内核缓冲区拷贝到 Socket buffer 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。
4. 这里其实有 一次 cpu 拷贝kernel buffer -> socket buffer但是，拷贝的信息很少，比如 lenght , offset , 消耗低，可以忽略

![io1](http://qiliu.luxiaobai.cn/img/io1.png)



## **零拷贝的再次理解**

1. 零拷贝，是从操作系统的角度来说的。因为内核缓冲区之间，没有数据是重复的（只有 kernel buffer 有一份数据）。
2. 零拷贝不仅仅带来更少的数据复制，还能带来其他的性能优势，例如更少的上下文切换，更少的 CPU 缓存伪共享以及无 CPU 校验和计算。



## **mmap 和 sendFile 的区别**

1. mmap 适合小数据量读写，sendFile 适合大文件传输。
2. mmap 需要 4 次上下文切换，3 次数据拷贝；sendFile 需要 3 次上下文切换，最少 2 次数据拷贝。
3. sendFile 可以利用 DMA 方式，减少 CPU 拷贝，mmap 则不能（必须从内核拷贝到 Socket 缓冲区）。



## 零拷贝案例

### 传统IO文件读写

```java
public class OldIOServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(7001);
        while (true){
            Socket socket = serverSocket.accept();
            DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
            try{
                byte[] bytes = new byte[4096];
                while (true){
                    int read = dataInputStream.read(bytes, 0, bytes.length);
                    if(read == -1){
                        break;
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

```java
public class OldIOClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1",7001);
        String fileName = "/Users/lushengyang/Desktop/Demo.zip";
        InputStream inputStream = new FileInputStream(fileName);
        DataOutputStream dataOutputStream = new DataOutputStream(socket.getOutputStream());
        byte[] buffer = new byte[4096];
        long readCount;
        long total = 0;
        long startTime  = System.currentTimeMillis();
        while ((readCount = inputStream.read(buffer))>=0){
            total += readCount;
            dataOutputStream.write(buffer);
        }
        System.out.println("发送总字节数：" + total + ", 耗时：" + (System.currentTimeMillis() - startTime));
        dataOutputStream.close();
        socket.close();
        inputStream.close();
    }
}
```

```java
public class NewIOServer {
    public static void main(String[] args) throws Exception{
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        ServerSocket serverSocket = serverSocketChannel.socket();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(7001);
        serverSocket.bind(inetSocketAddress);

        ByteBuffer byteBuffer = ByteBuffer.allocate(4096);

        while (true){
            SocketChannel socketChannel = serverSocketChannel.accept();
            int readCount = 0;
            while(-1 != readCount){
                readCount = socketChannel.read(byteBuffer);
                byteBuffer.rewind();//倒带position=0 mark作废
            }
        }
    }
}


public class NewIOClient {
    public static void main(String[] args) throws Exception {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("localhost", 7001));
        String fileName = "/Users/lushengyang/Desktop/Demo.zip";

        FileChannel fileChannel = new FileInputStream(fileName).getChannel();

        long startTime = System.currentTimeMillis();

        //在Linux下 一个transferTo方法就可以完成传输
        //在windows下 一次调用transferTo 只能发送8m，就需要分段传输文件，而且要注意传输的位置
        //transferto底层使用到零拷贝
        long transferCount = fileChannel.transferTo(0,fileChannel.size(),socketChannel);
        System.out.println("发送的总的字节数 = " + transferCount + " 耗时：" + (System.currentTimeMillis() - startTime));
        fileChannel.close();
    }
}
```



# JAVA AIO

1. JDK 7 引入了 Asynchronous I/O，即 AIO。在进行 I/O 编程中，常用到两种模式：Reactor 和 Proactor。Java 的NIO 就是 Reactor，当有事件触发时，服务器端得到通知，进行相应的处理
2. AIO 即 NIO2.0，叫做异步不阻塞的 IO。AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用



## BIO、NIO、AIO对比表

![io2](http://qiliu.luxiaobai.cn/img/io2.png)

1. 同步阻塞:到理发店理发,就一直等理发师,知道轮到自己理发.
2. 同步非阻塞:到理发店理发,发现前面有其他人理发,给理发师说下,先干其他事情,一会过来看是否轮到自己
3. 异步非阻塞:给理发师打电话,让理发师上门服务,自己干其他事情,理发师自己来家给你理发



# Netty概述

## 原生NIO存在的问题

1. NIO 的类库和 API 繁杂，使用麻烦：需要熟练掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer等。
2. 需要具备其他的额外技能：要熟悉 Java 多线程编程，因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 NIO 程序。
3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。
4. JDK NIO 的 Bug：例如臭名昭著的 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU 100%。直到 JDK 1.7版本该问题仍旧存在，没有被根本解决



**Netty官网**

https://netty.io/

![netty1](http://qiliu.luxiaobai.cn/img/netty1.png)



## **Netty优点**

1. 设计优雅：适用于各种传输类型的统一 API 阻塞和非阻塞 Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型 - 单线程，一个或多个线程池.
2. 使用方便：详细记录的 Javadoc，用户指南和示例；没有其他依赖项，JDK 5（Netty 3.x）或 6（Netty 4.x）就足够了。
3. 高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制。
4. 安全：完整的 SSL/TLS 和 StartTLS 支持。
5. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 Bug 可以被及时修复，同时，更多的新功能会被加入



## Netty架构设计

### 线程模型基本介绍

1.  目前存在的线程模型有：

2. 1. 传统阻塞 I/O 服务模型
   2. Reactor 模式

3. 根据 Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现

4. 1. 单 Reactor 单线程；
   2. 单 Reactor 多线程；
   3. 主从 Reactor 多线程

5. Netty 线程模式(Netty 主要基于主从 Reactor 多线程模型做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor)



### **传统阻塞I/O服务模型**

**工作原理图**

- 黄色的框表示对象， 蓝色的框表示线程
- 白色的框表示方法(API)

![io3](http://qiliu.luxiaobai.cn/img/io3.png)

**模型特点**

- 采用阻塞 IO 模式获取输入的数据
- 每个连接都需要独立的线程完成数据的输入，业务处理,数据返回

**分析**

- 当并发数很大，就会创建大量的线程，占用很大系统资源
- 连接创建后，如果当前线程暂时没有数据可读，该线程会阻塞在 read 操作，造成线程资源浪费



## Reactor模式

**针对传统阻塞IO服务模型的两个缺点,解决方案:**

1. 基于 I/O 复用模型：多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理Reactor 对应的叫法: 1. 反应器模式 2. 分发者模式(Dispatcher) 3. 通知者模式(notifier)
2. 基于线程池复用线程资源：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。

![io4](http://qiliu.luxiaobai.cn/img/io4.png)

I/O复用结合线程池,就是Reactor模式基本设计思想

![reactor](http://qiliu.luxiaobai.cn/img/reactor.png)

1. Reactor 模式，通过一个或多个输入同时传递给服务处理器的模式(基于事件驱动)
2. 服务器端程序处理传入的多个请求,并将它们同步分派到相应的处理线程， 因此 Reactor 模式也叫 Dispatcher模式
3. Reactor 模式使用 IO 复用监听事件, 收到事件后，分发给某个线程(进程), 这点就是网络服务器高并发处理关键



### **Reactor模式中核心组成**

1. Reactor：Reactor 在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对 IO 事件做出反应。 它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人；
2. Handlers：处理程序执行 I/O 事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。Reactor通过调度适当的处理程序来响应 I/O 事件，处理程序执行非阻塞操作。



### Reactor模式分类

根据 Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现

- 单 Reactor 单线程
- 单 Reactor 多线程
- 主从 Reactor 多线程



## **单Reactor单线程**

![reactor2](http://qiliu.luxiaobai.cn/img/reactor2-4725486.png)



**方案说明:**

1. Select 是前面 I/O 复用模型介绍的标准网络编程 API，可以实现应用程序通过一个阻塞对象监听多路连接请求
2. Reactor 对象通过 Select 监控客户端请求事件，收到事件后通过 Dispatch 进行分发
3. 如果是建立连接请求事件，则由 Acceptor 通过 Accept 处理连接请求，然后创建一个 Handler 对象处理连接完成后的后续业务处理
4. 如果不是建立连接事件，则 Reactor 会分发调用连接对应的 Handler 来响应
5. Handler 会完成 Read→业务处理→Send 的完整业务流程

结合实例：服务器端用一个线程通过多路复用搞定所有的 IO 操作（包括连接，读、写等），编码简单，清晰明了，

但是如果客户端连接数量较多，将无法支撑，

**方案优缺点分析**

1. 优点：模型简单，没有多线程、进程通信、竞争的问题，全部都在一个线程中完成
2. 缺点：性能问题，只有一个线程，无法完全发挥多核 CPU 的性能。Handler 在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈
3. 缺点：可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障
4. 使用场景：客户端的数量有限，业务处理非常快速，比如 Redis 在业务处理的时间复杂度 O(1) 的情况



## 单React多线程

![react4](http://qiliu.luxiaobai.cn/img/react4.png)

**方案说明**

1. Reactor 对象通过 select 监控客户端请求事件, 收到事件后，通过 dispatch 进行分发
2. 如果建立连接请求, 则由 Acceptor 通过accept 处理连接请求, 然后创建一个 Handler 对象处理完成连接后的各种事件
3. 如果不是连接请求，则由 reactor 分发调用连接对应的 handler 来处理
4. handler 只负责响应事件，不做具体的业务处理, 通过 read 读取数据后，会分发给后面的 worker 线程池的某个线程处理业务
5. worker 线程池会分配独立线程完成真正的业务，并将结果返回给 handler
6. handler 收到响应后，通过 send 将结果返回给 client

**方案优缺点分析:**

- 优点：可以充分的利用多核 cpu 的处理能力
- 缺点：多线程数据共享和访问比较复杂， reactor 处理所有的事件的监听和响应，在单线程运行， 在高并发场景容易出现性能瓶颈.



## **主从Reactor多线程**

![reactor5](http://qiliu.luxiaobai.cn/img/reactor5.png)

**方案说明**

1. Reactor 主线程 MainReactor 对象通过 select 监听连接事件, 收到事件后，通过 Acceptor 处理连接事件
2. 当 Acceptor 处理连接事件后，MainReactor 将连接分配给 SubReactor
3. subreactor 将连接加入到连接队列进行监听,并创建 handler 进行各种事件处理
4. 当有新事件发生时， subreactor 就会调用对应的 handler 处理
5. handler 通过 read 读取数据，分发给后面的 worker 线程处理
6. worker 线程池分配独立的 worker 线程进行业务处理，并返回结果
7. handler 收到响应的结果后，再通过 send 将结果返回给 client
8. Reactor 主线程可以对应多个 Reactor 子线程, 即 MainRecator 可以关联多个SubReactor



![reactor6](http://qiliu.luxiaobai.cn/img/reactor6.png)



**方案优缺点说明**

\1) 优点：父线程与子线程的数据交互简单职责明确，父线程只需要接收新连接，子线程完成后续的业务处理。

\2) 优点：父线程与子线程的数据交互简单，Reactor 主线程只需要把新连接传给子线程，子线程无需返回数据。

\3) 缺点：编程复杂度较高

\4) 结合实例：这种模型在许多项目中广泛使用，包括 Nginx 主从 Reactor 多进程模型，Memcached 主从多线程，

Netty 主从多线程模型的支持



## **React模式具有的优点**

1. 响应快，不必为单个同步时间所阻塞，虽然 Reactor 本身依然是同步的
2. 可以最大程度的避免复杂的多线程及同步问题，并且避免了多线程/进程的切换开销
3. 扩展性好，可以方便的通过增加 Reactor 实例个数来充分利用 CPU 资源
4. 复用性好，Reactor 模型本身与具体事件处理逻辑无关，具有很高的复用性



# **Netty模型**

**Netty 主要基于主从 Reactors 多线程模型（如图）做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor**

## **简单版**

![netty2](http://qiliu.luxiaobai.cn/img/netty2.png)

1. BossGroup 线程维护 Selector , 只关注 Accecpt
2. 当接收到 Accept 事件，获取到对应的 SocketChannel, 封装成 NIOScoketChannel 并注册到 Worker 线程(事件循环), 并进行维护
3. 当 Worker 线程监听到 selector 中通道发生自己感兴趣的事件后，就进行处理(就由 handler)， 注意 handler 已经加入到通道



## 进阶版

![netty4](http://qiliu.luxiaobai.cn/img/netty4.png)



## 详细版

![netty3](http://qiliu.luxiaobai.cn/img/netty3.png)



**工作原理**

1. Netty 抽象出两组线程池 BossGroup 专门负责接收客户端的连接, WorkerGroup 专门负责网络的读写

2. BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup

3. NioEventLoopGroup 相当于一个事件循环组, 这个组中含有多个事件循环 ，每一个事件循环是 NioEventLoop

4. NioEventLoop 表示一个不断循环的执行处理任务的线程， 每个 NioEventLoop 都有一个 selector , 用于监听绑定在其上的 socket 的网络通讯

5. NioEventLoopGroup 可以有多个线程, 即可以含有多个 NioEventLoop

6. 每个 Boss NioEventLoop 循环执行的步骤有 3 步

7. 1. 轮询accept事件
   2. 处理accept事件,与client建立连接,生成NioSocketChannel,并将其注册到某个 worker NIOEventLoop 上的 selector
   3. 处理任务队列的任务 ， 即 runAllTasks

8. 每个 Worker NIOEventLoop 循环执行的步骤

- 轮询 read, write 事件
- 处理 i/o 事件， 即 read , write 事件，在对应 NioScocketChannel 处理
- 处理任务队列的任务 ， 即 runAllTasks

8.每个Worker NIOEventLoop 处理业务时，会使用pipeline(管道), pipeline 中包含了 channel , 即通过pipeline可以获取到对应通道, 管道中维护了很多的处理器



## Netty-TCP服务

```java
public class NettyServer {
    public static void main(String[] args) throws InterruptedException {

        //创建BossGroup和WorkerGroup
        //说明
        //1 创建两个线程组bossGroup和workerGroup
        //2 bossGroup只是处理连接请求，真正的和客户端业务处理，交给workerGroup完成
        //3 两个都是无限循环
        //4 bossGroup 和 workerGroup含有的子线程（NioEventLoop)的个数
        //  默认实际cpu核数 * 2
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGoup = new NioEventLoopGroup();
        try {
            //创建服务器端的启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();

            //使用链式编程进行设置
            bootstrap.group(bossGroup, workerGoup)//设置两个线程组
                    .channel(NioServerSocketChannel.class)//使用NioSocketChannel作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128)//设置线程队列得到连接个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true)//设置保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() {//创建一个通道测试对象（匿名对象）
                        //给pipeline设置处理器
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyServerHandler());
                        }
                    });//为workerGroup的EventLoop 对应的管道设置处理器
            System.out.println("....服务器 is ready ....");
            //绑定一个端口并且同步，生成来一个ChannelFuture对象
            //启动服务器（并绑定端口）
            ChannelFuture cf = bootstrap.bind(6668).sync();

            //对关闭通道进行监听
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
        }
    }
}
```

```java
/**
 * 1 自定义一个Handler需要继承netty规定好的某个HandlerAdapter
 * 2 这时自定义一个Handler，才能称为一个handler
 */
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 读取数据（读取客户端发送的信息）
     *
     * @param ctx：上下文对象，含有管道pipeline，通道channel，地址
     * @param msg：就是客户端发送的数据                      默认Object
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("服务器读取线程：" + Thread.currentThread().getName());
        System.out.println("server ctx =" + ctx);
        System.out.println("看看channel和pipeline的关系");
        Channel channel = ctx.channel();
        ChannelPipeline pipeline = ctx.pipeline();//本质是一个双向链表，出栈入栈

        //将msg转成一个ByteBuf
        //ByteBuf 是Netty提供的，不是NIO的ByteBuffer
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送信息是：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("客户端地址: " + channel.remoteAddress());
    }

    /**
     * 数据读取完毕
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //writeAndFlush 是Write + Flush
        //将数据写入到缓冲，并刷新
        //一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常，一般是需要关闭通道
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

}
```



```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        //客户端需要一个事件循环组
        EventLoopGroup eventExecutors = new NioEventLoopGroup();
        try {


            //创建客户端启动对象
            //注意客户端使用的不是ServerBootstrap 而是Bootstrap
            Bootstrap bootstrap = new Bootstrap();

            //设置相关参数
            bootstrap.group(eventExecutors)//设置线程组
                    .channel(NioSocketChannel.class)//设置客户端通道的实现类（反射）
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyClientHandler());//加入自己的处理器
                        }
                    });
            System.out.println("...客户端 OK..");
            //启动客户端去连接服务端
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6668).sync();
            //关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            eventExecutors.shutdownGracefully();
        }
    }
}
```



```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    /**
     * 当通道就绪就会触发该方法
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("client " + ctx);
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, server: ", CharsetUtil.UTF_8));
    }

    /**
     * 当通道有读取事件时，会触发该方法
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务器回复的信息：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("服务器的地址：" + ctx.channel().remoteAddress());
    }

    /**
     * 异常处理时，触发该方法
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```



## **任务队列中的Task有3种典型使用场景**

1. 用户程序自定义的普通任务
2. 用户自定义定时任务
3. 非当前 Reactor 线程调用 Channel 的各种方法

例如在推送系统的业务线程里面，根据用户的标识，找到对应的 Channel 引用，然后调用 Write 类方法向该用户推送消息，就会进入到这种场景。最终的 Write 会提交到任务队列中后被异步消费

```java
/**
 * 1 自定义一个Handler需要继承netty规定好的某个HandlerAdapter
 * 2 这时自定义一个Handler，才能称为一个handler
 */
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 读取数据（读取客户端发送的信息）
     *
     * @param ctx：上下文对象，含有管道pipeline，通道channel，地址
     * @param msg：就是客户端发送的数据                      默认Object
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
          //假设这有一个非常耗时长的业务->移步执行->提交该channel对应的
          //NIOEventLoop的taskQueue中，

          //解决方案1 用户程序自定义的普通任务
          ctx.channel().eventLoop().execute(new Runnable() {
              @Override
              public void run() {
                  try{
                      Thread.sleep(10 * 1000);
                      ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端(>..<) 喵2",CharsetUtil.UTF_8));

                  }catch (Exception e){
                      System.out.println("发生异常： " + e.getMessage());
                  }
              }
          });
        System.out.println("go on....");

        //用户自定义任务 -> 该任务提交到 scheduleTaskQueue中
        ctx.channel().eventLoop().schedule(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(10 * 1000);
                    ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端(>..<) 喵3",CharsetUtil.UTF_8));
                }catch (Exception e){
                    System.out.println("发生异常： " + e.getMessage());
                }
            }
        },5, TimeUnit.SECONDS);
    }

    /**
     * 数据读取完毕
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //writeAndFlush 是Write + Flush
        //将数据写入到缓冲，并刷新
        //一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常，一般是需要关闭通道
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }

}
```

**方案说明**

1. Netty 抽象出两组线程池，BossGroup 专门负责接收客户端连接，WorkerGroup 专门负责网络读写操作。
2. NioEventLoop 表示一个不断循环执行处理任务的线程，每个 NioEventLoop 都有一个selector，用于监听绑定在其上的 socket 网络通道。
3. NioEventLoop 内部采用串行化设计，从消息的读取->解码->处理->编码->发送，始终由 IO 线程 NioEventLoop负责
4. NioEventLoopGroup 下包含多个 NioEventLoop

- - 每个 NioEventLoop 中包含有一个 Selector，一个 taskQueue
  - 每个 NioEventLoop 的 Selector 上可以注册监听多个 NioChannel
  - 每个 NioChannel 只会绑定在唯一的 NioEventLoop 上
  - 每个 NioChannel 都绑定有一个自己的 ChannelPipeline



# 异步模型

## 基本介绍

1. 异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在完成后，通过状态、通知和回调来通知调用者。
2. Netty 中的 I/O 操作是异步的，包括 Bind、Write、Connect 等操作会简单的返回一个 ChannelFuture。
3. 调用者并不能立刻获得结果，而是通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得IO 操作结果
4. Netty 的异步模型是建立在 future 和 callback 的之上的。callback 就是回调。重点说 Future，它的核心思想是：假设一个方法 fun，计算过程可能非常耗时，等待 fun 返回显然不合适。那么可以在调用 fun 的时候，立马返回一个 Future，后续可以通过 Future 去监控方法 fun 的处理过程(即 ： Future-Listener 机制)



**Future说明**

1. 表示异步的执行结果, 可以通过它提供的方法来检测执行是否完成，比如检索计算等等.
2. ChannelFuture 是一个接口 ： public **interface** ChannelFuture **extends** Future<Void>,可以添加监听器，当监听的事件发生时，就会通知到监听器. 

**工作原理**

![netty5](http://qiliu.luxiaobai.cn/img/netty5.png)

- 在使用 Netty 进行编程时，拦截操作和转换出入站数据只需要您提供 callback 或利用 future 即可。这使得链式操作简单、高效, 并有利于编写可重用的、通用的代码。
- Netty 框架的目标就是让你的业务逻辑从网络基础应用编码中分离出来、解脱出来

 

## Future-Listener机制

1. 当 Future 对象刚刚创建时，处于非完成状态，调用者可以通过返回的 ChannelFuture 来获取操作执行的状态，注册监听函数来执行完成后的操作。
2. 常见有如下操作

- - 通过 isDone 方法来判断当前操作是否完成；
  - 通过 isSuccess 方法来判断已完成的当前操作是否成功；
  - 通过 getCause 方法来获取已完成的当前操作失败的原因；
  - 通过 isCancelled 方法来判断已完成的当前操作是否被取消；
  - 通过 addListener 方法来注册监听器，当操作已完成(isDone 方法返回完成)，将会通知指定的监听器；如果Future 对象已完成，则通知指定的监听器

绑定端口是异步操作，当绑定操作处理完，将会调用相应的监听器处理逻辑

```java
//启动服务器（并绑定端口）
ChannelFuture cf = bootstrap.bind(6668).sync();

//给cs注册监听器，监控我们关心的事件
cf.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture channelFuture) throws Exception {
        if(channelFuture.isSuccess()){
            System.out.println("端口6668 绑定成功");
        }else{
            System.out.println("端口 绑定失败");
        }
    }
});
```

相比传统阻塞I/O,执行I/O操作后线程会被阻塞住,直到操作完成:异步处理的好处是不会造成线程阻塞,程序在I/O操作期间可以执行别的程序,在高并发情形下会更稳定和更高的吞吐量



# Netty--HTTP服务

```java
public class TestServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();

            serverBootstrap.group(bossGroup, workGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new TestServerInitializer());

            ChannelFuture channelFuture = serverBootstrap.bind(10066).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();
        }
    }
}


public class TestServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        //向管道加入处理器
        // 得到管道
        ChannelPipeline pipeline = ch.pipeline();
        //加入一个 netty 提供的 httpServerCodec codec =>[coder - decoder]
        // HttpServerCodec 说明
        // 1. HttpServerCodec 是 netty 提供的处理 http 的 编-解码器
        pipeline.addLast("MyHttpServerCodec", new HttpServerCodec());
        //2. 增加一个自定义的 handler
        pipeline.addLast("MyTestHttpServerHandler", new TestHttpServerhandler());

    }
}


/**
 * SimpleChannelInboundHandler 是ChannelInboundHandlerAdapter的子类
 * HttpObject 客户端和服务器端相互通讯的数据被封装成HttpObject
 */
public class TestHttpServerhandler extends SimpleChannelInboundHandler<HttpObject> {

    /**
     * 读取客户端数据
     *
     * @param channelHandlerContext
     * @param httpObject
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, HttpObject httpObject) throws Exception {
        //判断msg是不是httprequest请求
        if(httpObject instanceof HttpRequest){
            System.out.println("msg 类型：" + httpObject.getClass());
            System.out.println("客户端地址： " + channelHandlerContext.channel().remoteAddress());

            //获取到
            HttpRequest httpRequest = (HttpRequest) httpObject;
            //获取uri,过滤指定的资源
            URI uri = new URI(httpRequest.uri());
            if ("/favicon.ico".equals(uri.getPath())){
                System.out.println("请求了 favicon.ico 不做响应");
                return;
            }
            //回复信息给浏览器【http协议】
            ByteBuf content = Unpooled.copiedBuffer("hello,我是服务器", CharsetUtil.UTF_8);

            //构造一个http的相应，即httpresponse
            DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);

            response.headers().set(HttpHeaderNames.CONTENT_TYPE,"text/plain;charset=utf-8");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH,content.readableBytes());

            //将构建好的 response 返回
            channelHandlerContext.writeAndFlush(response);


        }
    }
}
```



# **Netty核心模块组件**

## **Bootstrap、ServerBootstrap**

1. Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类
2. 常见的方法有

- - - public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个 EventLoop
    - public B group(EventLoopGroup group) ，该方法用于客户端，用来设置一个 EventLoop
    - public B channel(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现
    - public <T> B option(ChannelOption<T> option, T value)，用来给 ServerChannel 添加配置
    - public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)，用来给接收到的通道添加配置
    - public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的handler）
    - public ChannelFuture bind(int inetPort) ，该方法用于服务器端，用来设置占用的端口号
    - public ChannelFuture connect(String inetHost, int inetPort) ，该方法用于客户端，用来连接服务器端

## **Future、ChannelFuture**

1. Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件
2. 常见的方法有

- - - Channel channel()，返回当前正在进行 IO 操作的通道
    - ChannelFuture sync()，等待异步操作执行完毕

## **Channel**

1. Netty 网络通信的组件，能够用于执行网络 I/O 操作。
2. 通过 Channel 可获得当前网络连接的通道的状态
3. 通过 Channel 可获得 网络连接的配置参数 （例如接收缓冲区大小）
4. Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
5. 调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方
6. 支持关联 I/O 操作与对应的处理程序
7. 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型:

- - NioSocketChannel，异步的客户端 TCP Socket 连接。
  - NioServerSocketChannel，异步的服务器端 TCP Socket 连接。
  - NioDatagramChannel，异步的 UDP 连接。
  - NioSctpChannel，异步的客户端 Sctp 连接。
  - NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。

## **Selector**

1. Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。
2. 当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册的Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel

### **ChannelHandler 及其实现类**

1. ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline(业务处理链)中的下一个处理程序。
2. ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类
3. ChannelHandler 及其实现类一览图(后)
4. 需要自定义一个 Handler 类去继承 ChannelInboundHandlerAdapter，然后通过重写相应方法实现业务逻辑，一般都需要重写哪些方法

![netty](http://qiliu.luxiaobai.cn/img/netty.png)

![nettys1](http://qiliu.luxiaobai.cn/img/nettys1.png)



####  **Pipeline和ChannelPipeline**

**ChannelPipeline 是一个重点：**

1. ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于一个贯穿 Netty 的链。(也可以这样理解：ChannelPipeline 是 保存 ChannelHandler 的 List，用于处理或拦截Channel 的入站事件和出站操作)
2. ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel中各个的 ChannelHandler 如何相互交互
3. 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下

![nettys2](http://qiliu.luxiaobai.cn/img/nettys2.png)

![nettys3](http://qiliu.luxiaobai.cn/img/nettys3.png)

4.常用方法

- - - ChannelPipeline addFirst(ChannelHandler... handlers)，把一个业务处理类（handler）添加到链中的第一个位置
    - ChannelPipeline addLast(ChannelHandler... handlers)，把一个业务处理类（handler）添加到链中的最后一个位置



#### **ChannelHandlerContext**

1. 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象
2. 即 ChannelHandlerContext 中 包 含 一 个 具 体 的 事 件 处 理 器 ChannelHandler ， 同 时
3. ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler 进行调用.
4. 常用方法

```java
ChannelFuture close();//关闭通道
ChannelOutboundInvoker flush();//熟悉
ChannelFuture writeAndFlush(Object msg); //将数据写到ChannelPipeline中当前ChannelHandler 的下一个ChannelHandler开始处理(出站)
```



#### Channel Option

1. Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数。
2. ChannelOption 参数如下:

![nettys4](http://qiliu.luxiaobai.cn/img/nettys4.png)



### **EventLoopGroup 和其实现类 NioEventLoopGroup**

1. EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop同时工作，每个 EventLoop 维护着一个 Selector 实例。
2. EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。在 Netty服 务 器 端 编 程 中 ， 我 们 一 般 都 需 要 提 供 两 个 EventLoopGroup ， 例 如 ： BossEventLoopGroup 和WorkerEventLoopGroup。
3. 通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示

![nettys5](http://qiliu.luxiaobai.cn/img/nettys5.png)

**常用方法**

```java
public NioEventLoopGroup()，构造方法
public Future<?> shutdownGracefully()，断开连接，关闭线程
```



### **Unpooled类**

Netty 提供一个专门用来操作缓冲区(即 Netty 的数据容器)的工具类

```java
//通过给定的数据和字符编码返回一个ByteBuf对象(类似于NIO中的ByteBuffer但有区别)
public static ByteBuf copiedBuffer(CharSequence string, Charset charset)
```

实例--Unpooled 获取Netty的数据容器ByteBuf的基本使用

```java
public class NettyByteBuf01 {
    public static void main(String[] args) {
        //创建一个ByteBuf
        //说明
        //1 创建对象，该对象包含一个数组arr，是一个byte[10]
        //2 在netty的buffer中，不需要flip进行反转
        //  底层维护了readerindex和writeindex
        //3 通过readerindex和writeindex 和 capacity ，将buffer分成三个区域
        // 0------readerindex ---writerIndex， 可读的区域
        // writerIndex -- capacity, 可写的区域
        ByteBuf buffer = Unpooled.buffer(10);
        for (int i = 0; i< 10;i++){
            buffer.writeByte(i);
        }

        System.out.println("capacity:" + buffer.capacity());
        //输出
//        for (int i = 0; i< buffer.capacity();i++){
//            System.out.println(buffer.getByte(i));
//        }
        for (int i = 0; i< buffer.capacity();i++){
            System.out.println(buffer.readByte());
        }
    }
}
public class NettyByteBuff02 {
    public static void main(String[] args) {

        //创建ByteBuf
        ByteBuf buf = Unpooled.copiedBuffer("hello,world!", Charset.forName("utf-8"));
        if (buf.hasArray()){
            byte[] content = buf.array();

//            将content转成字符串
            System.out.println(new String(content,Charset.forName("utf-8")));
            System.out.println("byteBuf=" + buf);
            //获取数组的偏移量
            System.out.println(buf.arrayOffset()); //0
            System.out.println(buf.readerIndex()); //0
            System.out.println(buf.writerIndex()); //12
            System.out.println(buf.capacity());    //36

            int len = buf.readableBytes();          //可读的字节数量
            System.out.println(len);

            for (int i = 0; i< len;i++){
                System.out.println((char) buf.getByte(i));
            }

            //按照区间读取
            System.out.println(buf.getCharSequence(0,4,Charset.forName("utf-8")));
            System.out.println(buf.getCharSequence(4,6,Charset.forName("utf-8")));
        }
    }
}
```

[Netty通过WebSocket编程实现服务器和客户端长连接](https://download.csdn.net/download/weixin_45268711/41356981)

[Netty实例-群聊系统](https://download.csdn.net/download/weixin_45268711/41358365)

[**Netty心跳检测机制**](https://download.csdn.net/download/weixin_45268711/41358488)

