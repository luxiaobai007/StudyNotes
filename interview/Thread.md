[toc]

# JUC

JUC就是 java.util .concurrent 工具包的简称。这是一个处理线程的工具包，JDK1.5 开始出现的。



**进程**：指在系统中正在运行的一个应用程序；程序一旦运行就是进程；进程——资源分配的最小单位。

**线程**：系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位。



## **wait/sleep 的区别**

- sleep 是 Thread 的静态方法，wait 是 Object 的方法，任何对象实例都能调用。
- sleep 不会释放锁，它也不需要占用锁。wait 会释放锁，但调用它的前提是当前线程占有锁(即代码要在 synchronized 中)。

它们都可以被interrupted方法中断.





## 并发与并行

### 串行模式

**串行是一次只能取得一个任务，并执行这个任务。**



### 并行模式

并行意味着可以同时取得多个任务，并同时去执行所取得的这些任务。并行模式相当于将长长的一条队列，划分成了多条短队列，所以并行缩短了任务队列的长度。并行的效率从代码层次上强依赖于多进程/多线程代码，从硬件角度上则依赖于多核 CPU。





## 管程

管程(monitor)是**保证了同一时刻只有一个进程在管程内活动,**即管程内定义的操作在同一时刻只被一个进程调用(由编译器实现).但是这样并不能保证进程以设计的顺序执行.JVM 中同步是**基于进入和退出管程(monitor)对象**实现的，每个对象都会有一个管程(monitor)对象，管程(monitor)会随着 java 对象一同创建和销毁执行线程首先要持有管程对象，然后才能执行方法，当方法完成之后会释放管程，方法在执行时候会持有管程，其他线程无法再获取同一个管程



## 用户线程和守护线程

- **用户线程**:平时用到的普通线程,自定义线程
- **守护线程**:运行在后台,是一种特殊的线程,比如垃圾回收

当主线程结束后,用户线程还在运行,JVM 存活

如果没有用户线程,都是守护线程,JVM 结束



# 集合的线程安全

## CopyOnWriteArrayList

它相当于线程安全的 ArrayList。和 ArrayList 一样，它是个可变数组；但是和ArrayList 不同的是，它具有以下特性：

1. 它最适合于具有以下特征的应用程序：List 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove()等等）的开销很大。
4. 迭代器支持 hasNext(), next()等不可变操作，但不支持可变 remove()等操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

- **独占锁效率低：采用读写分离思想解决**
- **写线程获取到锁，其他写线程阻塞**
- **复制思想**

当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这时候会抛出来一个新的问题，也就是数据不一致的问题。如果写线程还没来得及写会内存，其他的线程就会读到了脏数据。==这就是 CopyOnWriteArrayList 的思想和原理。就是拷贝一份。



## 动态数组和线程安全

### 动态数组

- 它内部有个“volatile 数组”(array)来保持数据。在“添加/修改/删除”数据时，都会新建一个数组，并将更新后的数据拷贝到新建的数组中，最后再将该数组赋值给“volatile 数组”, 这就是它叫做 CopyOnWriteArrayList 的原因
- 由于它在“添加/修改/删除”数据时，都会新建数组，所以涉及到修改数据的操作，CopyOnWriteArrayList 效率很低；但是单单只是进行遍历查找的话，效率比较高。



### 线程安全机制

- 通过 volatile 和互斥锁来实现的。
- 通过“volatile 数组”来保存数据的。一个线程读取 volatile 数组时，总能看到其它线程对该 volatile 变量最后的写入；就这样，通过 volatile 提供了“读取到的数据总是最新的”这个机制的保证。
- 通过互斥锁来保护数据。在“添加/修改/删除”数据时，会先“获取互斥锁”，再修改完毕之后，先将数据更新到“volatile 数组”中，然后再“释放互斥锁”，就达到了保护数据的目的。



## **Collections 构建的线程安全集合**

**java.util.concurrent 并发包下**

CopyOnWriteArrayList CopyOnWriteArraySet 类型,通过动态数组与线程安全个方面保证线程安全



# 多线程锁

```java
public class Lock {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        Phone phone2 = new Phone();
        new Thread(()->{
            try {
                phone.sendSMS();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"AA").start();

        Thread.sleep(100);

        new Thread(()->{
            try {
                phone.sendEmail();
//                phone.getHello();
//                phone2.sendEmail();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"BB").start();
    }
}
/**
 * @Description: 8锁
 *
1 标准访问，先打印短信还是邮件
------sendSMS
------sendEmail

2 停4秒在短信方法内，先打印短信还是邮件
-------sendSMS
-------sendEmail

3 新增普通的hello方法，是先打短信还是hello
------getHello
------sendSMS

4 现在有两部手机，先打印短信还是邮件
------sendEmail
------sendSMS

5 两个静态同步方法，1部手机，先打印短信还是邮件
------sendSMS
------sendEmail

6 两个静态同步方法，2部手机，先打印短信还是邮件
------sendSMS
------sendEmail

7 1个静态同步方法,1个普通同步方法，1部手机，先打印短信还是邮件
------sendEmail
------sendSMS

8 1个静态同步方法,1个普通同步方法，2部手机，先打印短信还是邮件
------sendEmail
------sendSMS

 */
class Phone{
    public static synchronized void sendSMS()throws Exception{
        //停留4秒
        TimeUnit.SECONDS.sleep(4);
        System.out.println("-------sendSMS");
    }

    public synchronized void sendEmail()throws Exception{
        System.out.println("-------sendEmail");
    }

    public void getHello(){
        System.out.println("--------getHello");
    }
}
```

结论

一个对象里面如果有多个 synchronized 方法，某一个时刻内，只要一个线程去调用其中的一个 synchronized 方法了，其它的线程都只能等待，换句话说，某一个时刻内，只能有唯一一个线程去访问这些synchronized 方法 锁的是当前对象 this，被锁定后，其它的线程都不能进入到当前对象的其它的synchronized 方法

加个普通方法后发现和同步锁无关

换成两个对象后，不是同一把锁了，情况立刻变化。

synchronized 实现同步的基础：Java 中的每一个对象都可以作为锁。

具体表现为以下 3 种形式。

- **对于普通同步方法，锁是当前实例对象。**
- **对于静态同步方法，锁是当前类的 Class 对象。**
- **对于同步方法块，锁是 Synchonized 括号里配置的对象**

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以毋须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。所有的静态同步方法用的也是同一把锁——类对象本身，这两把锁是两个不同的对象，所以**静态同步方法与非静态同步方法之间是不会有竞态条件**的。但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁，而不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同步方法之间，只要它们同一个类的实例对象！



# Callable&Future接口

```java
public class Demo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
       FutureTask<Integer> futureTask = new FutureTask<>(()->{
           System.out.println(Thread.currentThread().getName() + " come in");
           return 1024;
       });
       new Thread(futureTask,"lxb").start();
       while (!futureTask.isDone()){
           System.out.println("wait.....");
       }

        System.out.println(futureTask.get());
        System.out.println(Thread.currentThread().getName() + " come over");
    }
}

wait.....
wait.....
wait.....
lxb come in
wait.....
1024
main come over
```



## FutureTask

Java 库具有具体的 FutureTask 类型，该类型实现 Runnable 和 Future，并方便地将两种功能组合在一起。 可以通过为其构造函数提供 Callable 来创建FutureTask。然后，将 FutureTask 对象提供给 Thread 的构造函数以创建Thread 对象。因此，间接地使用 Callable 创建线程。

**核心原理:(重点)**

在主线程中需要执行比较耗时的操作时，但又不想阻塞主线程时，可以把这些作业交给 Future 对象在后台完成

- 当主线程将来需要时，就可以通过 Future 对象获得后台作业的计算结果或者执行状态• 一般 FutureTask 多用于耗时的计算，主线程可以在完成自己的任务后，再去获取结果。
- 仅在计算完成时才能检索结果；如果计算尚未完成，则阻塞 get 方法
- 一旦计算完成，就不能再重新开始或取消计算
- get 方法而获取结果只有在计算完成时获取，否则会一直阻塞直到任务转入完成状态，然后会返回结果或者抛出异常
- get 只计算一次,因此 get 方法放到最后

```java
public class CallableDemo {
    static class MyThread1 implements Runnable{
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "线程进入来run方法");
        }
    }

    static class MyThread2 implements Callable{

        @Override
        public Object call() throws Exception {
            System.out.println(Thread.currentThread().getName() + "线程进入来Call方法，开始准备睡觉");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "睡醒了");
            return System.currentTimeMillis();
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //声明runable
        Runnable runnable = new MyThread1();
        //声明callable
        Callable callable = new MyThread2();
        //future-callable
        FutureTask<Long> futureTask2 = new FutureTask(callable);
        new Thread(futureTask2,"线程二").start();
        for (int i = 0; i < 10; i++) {
            Long aLong = futureTask2.get();
            System.out.println(aLong);
        }
        new Thread(runnable,"线程一").start();
    }
}
```



# JUC三大辅助类

JUC 中提供了三种常用的辅助类，通过这些辅助类可以很好的解决线程数量过多时 Lock 锁的频繁操作。这三种辅助类为：

- CountDownLatch: 减少计数
- CyclicBarrier: 循环栅栏
- Semaphore: 信号灯



## 减少计数CountDownLatch

CountDownLatch 类可以设置一个计数器，然后通过 countDown 方法来进行减 1 的操作，使用 await 方法等待计数器不大于 0，然后继续执行 await 方法之后的语句。

- CountDownLatch 主要有两个方法，当一个或多个线程调用 await 方法时，这些线程会阻塞
- 其它线程调用 countDown 方法会将计数器减 1(调用 countDown 方法的线程不会阻塞)
- 当计数器的值变为 0 时，因 await 方法阻塞的线程会被唤醒，继续执行



错误的线程不安全

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
//        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + " 号同学离开");
//                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        //等待
//        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + " stop");
    }
}

1 号同学离开
4 号同学离开
3 号同学离开
2 号同学离开
5 号同学离开
main stop
6 号同学离开
```



```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName() + " 号同学离开");
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        //等待
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + " stop");
    }
}
```



## 循环栅栏CyclicBarrier

CyclicBarrier 看英文单词可以看出大概就是**循环阻塞**的意思，在使用中CyclicBarrier 的构造方法第一个参数是目标障碍数，每次执行 CyclicBarrier 一次障碍数会加一，如果达到了目标障碍数，才会执行 cyclicBarrier.await()之后的语句。可以将 CyclicBarrier 理解为加 1 操作

```java
public class CyclicBarrierDemo {
    private final static int NUMBER = 7;

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(NUMBER, () -> {
            System.out.println("集齐" + NUMBER + "颗龙珠");
        });
        for (int i = 1; i <= 7; i++) {
            new Thread(() -> {
                try {
                    if (Thread.currentThread().getName().equals("龙珠3号")) {
                        System.out.println("龙珠3号抢夺战开始");
                        Thread.sleep(5000);
                        System.out.println("龙珠3号抢夺战结束");
                    } else {
                        System.out.println(Thread.currentThread().getName() + "收集到了");
                    }
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, "龙珠" + i + "号").start();
        }
    }
}
```



## 信号灯

Semaphore 的构造方法中传入的第一个参数是最大信号量（可以看成最大线程池），每个信号量初始化为一个最多只能分发一个许可证。使用 acquire 方法获得许可证，release 方法释放许可

```java
public class SemaphoreDemo {
    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            Thread.sleep(100);
            new Thread(()->{
                try{
                    System.out.println(Thread.currentThread().getName() + "找车位ing");
                    //获取许可
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "停车成功");
                    Thread.sleep(1000);
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    System.out.println(Thread.currentThread().getName() + "离开");
                    semaphore.release();
                }
            },"汽车" + i).start();
        }
    }
}
汽车1找车位ing
汽车1停车成功
汽车2找车位ing
汽车2停车成功
汽车3找车位ing
汽车3停车成功
汽车4找车位ing
汽车5找车位ing
汽车6找车位ing
汽车1离开
汽车4停车成功
汽车2离开
汽车5停车成功
汽车3离开
汽车6停车成功
汽车4离开
汽车5离开
汽车6离开
```



# 读写锁

**写锁:一个资源可以被多个读线程访问,或者可以被一个写线程访问,但是不能同时存在读写线程,读写互斥,读读共享的**



现实中有这样一种场景：对共享资源有读和写的操作，且写操作没有读操作那么频繁。在没有写操作的时候，多个线程同时读一个资源没有任何问题，所以应该允许多个线程同时读取共享资源；但是如果一个线程想去写这些共享资源，就不应该允许其他线程对该资源进行读和写的操作了。

针对这种场景，JAVA 的并发包提供了**读写锁 ReentrantReadWriteLock**，它表示两个锁，一个是**读操作相关的锁**，称为**共享锁**；一个是**写相关的锁**，称为**排他锁**

## 1、线程进入读锁的前提条件：

- 没有其他线程的写锁
- 没有写请求, 或者==有写请求，但调用线程和持有锁的线程是同一个(可重入锁)。==

## 2、线程进入写锁的前提条件：

- 没有其他线程的读锁
- 没有其他线程的写锁



而读写锁有以下三个重要的特性：

（1）**公平选择性**：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

（2）**重进入**：读锁和写锁都支持线程重进入。

（3）**锁降级**：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。

![thread](http://qiliu.luxiaobai.cn/img/thread.png)

![thread2](http://qiliu.luxiaobai.cn/img/thread2-2214714.png)



![thread3](http://qiliu.luxiaobai.cn/img/thread3.png)

## **ReentrantReadWriteLock**

```java
public class ReentrantReadWriteLock implements ReadWriteLock,java.io.Serializable {
    /** 读锁 */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** 写锁 */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    final Sync sync;
    
    /** 使用默认（非公平）的排序属性创建一个新的ReentrantReadWriteLock */
    public ReentrantReadWriteLock() {
        this(false);
    }
    
    /** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
    /** 返回用于写入操作的锁 */
    public ReentrantReadWriteLock.WriteLock writeLock() { 
        return writerLock; 
    }
    /** 返回用于读取操作的锁 */
    public ReentrantReadWriteLock.ReadLock readLock() { 
        return readerLock; 
    }
    
    abstract static class Sync extends AbstractQueuedSynchronizer {}
    static final class NonfairSync extends Sync {}
    static final class FairSync extends Sync {}
    public static class ReadLock implements Lock, java.io.Serializable {}
    public static class WriteLock implements Lock, java.io.Serializable {}
}    
```

ReentrantReadWriteLock 实现了 ReadWriteLock 接口，ReadWriteLock 接口定义了获取读锁和写锁的规范，具体需要实现类去实现；同时其还实现了 Serializable 接口，表示可以进行序列化，在源代码中可以看到 ReentrantReadWriteLock 实现了自己的序列化逻辑。



## 案例

```java
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        
        for (int i = 1; i <= 5; i++) {
           final int num=i;
           new Thread(()->{
               myCache.put(num+"",num+"");
           },String.valueOf(i)).start();
        }

        for (int i = 0; i < 5; i++) {
            final int num=i;
            new Thread(()->{
                myCache.get(num+"");
            },String.valueOf(i)).start();
        }
    }
}

//资源类
class MyCache {
    //创建map集合
    private volatile Map<String, Object> map = new HashMap<>();
    //创建读写锁对象
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();

    //放数据
    public void put(String key, Object value) {
        //添加读写锁
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + " 正在写操作" + key);
            //暂停一会
            TimeUnit.MICROSECONDS.sleep(300);
            //放数据
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + " 写完了" + key);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //释放写锁
            rwLock.writeLock().unlock();
        }
    }


    //取数据
    public Object get(String key) {
        rwLock.readLock().lock();
        Object result = null;
        try {
            System.out.println(Thread.currentThread().getName() + "正在读操作" + key);
            //暂停
            TimeUnit.MICROSECONDS.sleep(300);
            result = map.get(key);
            System.out.println(Thread.currentThread().getName() + " 取完了" + key);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.readLock().unlock();
        }
        return result;
    }
}
```

- 在线程持有读锁的情况下，该线程不能取得写锁(因为获取写锁的时候，如果发现当前的读锁被占用，就马上获取失败，不管读锁是不是被当前线程持有)。
- 在线程持有写锁的情况下，该线程可以继续获取读锁（获取读锁时如果发现写锁被占用，只有写锁没有被当前线程占用的情况才会获取失败）。

原因: 当线程获取读锁的时候，可能有其他线程同时也在持有读锁，因此不能把获取读锁的线程“升级”为写锁；而对于获得写锁的线程，它一定独占了读写锁，因此可以继续让它获取读锁，当它同时获取了写锁和读锁后，还可以先释放写锁继续持有读锁，这样一个写锁就“降级”为了读锁。

```java
//读写锁
ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock.writeLock();//读锁
ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock.readLock();//写锁


//获取写锁
writeLock.lock();
System.out.println("--wriete");
//获取读锁
readLock.lock();
System.out.println("--read");

//释放写锁
writeLock.unlock();
//释放读锁
readLock.unlock();
```



# 阻塞队列

## Blocking Queue

阻塞队列，顾名思义，首先它是一个队列, 通过一个共享的队列，可以使得数据由队列的一端输入，从另外一端输出；

![queue1](http://qiliu.luxiaobai.cn/img/queue1.png)

当队列是空的，从队列中获取元素的操作将会被阻塞

当队列是满的，从队列中添加元素的操作将会被阻塞

试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素

试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素或者完全清空，使队列变得空闲起来并后续新增



常见队列:

- 先进先出（FIFO）：先插入的队列的元素也最先出队列，类似于排队的功能。从某种程度上来说这种队列也体现了一种公平性
- 后进先出（LIFO）：后插入队列的元素最先出队列，这种队列优先处理最近发生的事件(栈)

在多线程领域:所谓阻塞,在某些情况下会挂起线程(即阻塞),一旦条件满足,被挂起的线程又会自动被唤起





### 生产者和消费者

- 当队列中没有数据的情况下，消费者端的所有线程都会被自动阻塞（挂起），直到有数据放入队列
- 当队列中填满数据的情况下，生产者端的所有线程都会被自动阻塞（挂起），直到队列中有空的位置，线程被自动唤醒



### BlockingQueue核心方法

![queue2](http://qiliu.luxiaobai.cn/img/queue2.png)



## 常见BlockingQueue

### ArrayBlockingQueue(常用)

基于数组的阻塞队列实现，在 ArrayBlockingQueue 内部，维护了一个**定长数组**，以便缓存队列中的数据对象，这是一个常用的阻塞队列，除了一个定长数组外，ArrayBlockingQueue 内部还**保存着两个整形变量，**分别标识着**队列的头部和尾部**在数组中的位置。ArrayBlockingQueue 在生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此也意味着两者无法真正并行运行，这点尤其不同于LinkedBlockingQueue；按照实现原理来分析，ArrayBlockingQueue 完全可以采用分离锁，从而实现生产者和消费者操作的完全并行运行。Doug Lea 之所以没这样去做，也许是因为 ArrayBlockingQueue 的数据写入和获取操作已经足够轻巧，以至于引入独立的锁机制，除了给代码带来额外的复杂性外，其在性能上完全占不到任何便宜。 ArrayBlockingQueue 和LinkedBlockingQueue 之间还有一个明显的不同之处在于，**前者在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node 对象**。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC 的影响还是存在一定的区别。而在创建 ArrayBlockingQueue 时，我们还可以**控制对象的内部锁是否采用公平锁，默认采用非公平锁****。**

**由数组结构组成的有界阻塞队列**



### **LinkedBlockingQueue(常用)**

基于链表的阻塞队列，同 ArrayListBlockingQueue 类似，其内部也维持着一个数据缓冲队列（该队列由一个链表构成），当生产者往队列中放入一个数据时，队列会从生产者手中获取数据，并缓存在队列内部，而生产者立即返回；只有当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue 可以通过构造函数指定该值），才会阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者这端的处理也基于同样的原理。而 LinkedBlockingQueue 之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。

**由链表结构组成的有界（但大小默认值为integer.MAX_VALUE）阻塞队列。==**



> **ArrayBlockingQueue 和 LinkedBlockingQueue 是两个最普通也是最常用的阻塞队列，一般情况下，在处理多线程间的生产者消费者问题，使用这两个类足以。**



### **DelayQueue**

DelayQueue 中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue 是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

**==一句话总结: 使用优先级队列实现的延迟无界阻塞队列。==**



### **PriorityBlockingQueue**

基于优先级的阻塞队列（优先级的判断通过构造函数传入的 Compator 对象来决定），但需要注意的是 PriorityBlockingQueue 并不会阻塞数据生产者，而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。在实现 PriorityBlockingQueue 时，内部控制线程同步的锁采用的是公平锁。

**==一句话总结: 支持优先级排序的无界阻塞队列。==**



### **SynchronousQueue**

一种无缓冲的等待队列，类似于无中介的直接交易，有点像原始社会中的生产者和消费者，生产者拿着产品去集市销售给产品的最终消费者，而消费者必须亲自去集市找到所要商品的直接生产者，如果一方没有找到合适的目标，那么对不起，大家都在集市等待。相对于有缓冲的 BlockingQueue 来说，少了一个中间经销商的环节（缓冲区），如果有经销商，生产者直接把产品批发给经销商，而无需在意经销商最终会将这些产品卖给那些消费者，由于经销商可以库存一部分商品，因此相对于直接交易模式，总体来说采用中间经销商的模式会吞吐量高一些（可以批量买卖）；但另一方面，又因为经销商的引入，使得产品从生产者到消费者中间增加了额外的交易环节，单个产品的及时响应性能可能会降低。声明一个 SynchronousQueue 有两种不同的方式，它们之间有着不太一样的行为。

**公平模式和非公平模式的区别:**

- 公平模式：SynchronousQueue 会采用公平锁，并配合一个 FIFO 队列来阻塞多余的生产者和消费者，从而体系整体的公平策略；
- 非公平模式（SynchronousQueue 默认）：SynchronousQueue 采用非公平锁，同时配合一个 LIFO 队列来管理多余的生产者和消费者，而后一种模式，如果生产者和消费者的处理速度有差距，则很容易出现饥渴的情况，即可能有某些生产者或者是消费者的数据永远都得不到处理。

**==一句话总结: 不存储元素的阻塞队列，也即单个元素的队列。==**



### **LinkedTransferQueue**

LinkedTransferQueue 是一个由链表结构组成的无界阻塞 TransferQueue 队列。相对于其他阻塞队列，LinkedTransferQueue 多了 tryTransfer 和transfer 方法。LinkedTransferQueue 采用一种预占模式。意思就是消费者线程取元素时，如果队列不为空，则直接取走数据，若队列为空，那就生成一个节点（节点元素为 null）入队，然后消费者线程被等待在这个节点上，后面生产者线程入队时发现有一个元素为 null 的节点，生产者线程就不入队了，直接就将元素填充到该节点，并唤醒该节点等待的线程，被唤醒的消费者线程取走元素，从调用的方法返回。

==**一句话总结: 由链表组成的无界阻塞队列。**==



### **LinkedBlockingDeque**

LinkedBlockingDeque 是一个由链表结构组成的双向阻塞队列，即可以从队列的两端插入和移除元素。对于一些指定的操作，在插入或者获取队列元素时如果队列状态不允许该操作可能会阻塞住该线程直到队列状态变更为允许操作，这里的阻塞一般有两种情况

- 插入元素时: 如果当前队列已满将会进入阻塞状态，一直等到队列有空的位置时再讲该元素插入，该操作可以通过设置超时参数，超时后返回 false 表示操作失败，也可以不设置超时参数一直阻塞，中断后抛出 InterruptedException 异常
- 读取元素时: 如果当前队列为空会阻塞住直到队列不为空然后返回元素，同样可以通过设置超时参数

**==一句话总结: 由链表组成的双向阻塞队列==**



### **小结**

1. 在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起
2. 为什么需要 BlockingQueue? 在 concurrent 包发布以前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。使用后我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切 BlockingQueue 都给你一手包办了





# ThreadPool线程池

> **线程池**（英语：thread pool）：一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待着监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度。



## 线程池的优势

线程池做的工作只要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

**它的主要特点为：**

- 降低资源消耗: 通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
- 提高响应速度: 当任务到达时，任务可以不需要等待线程创建就能立即执行。
- 提高线程的可管理性: 线程是稀缺资源，如果无限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
- **Java 中的线程池是通过 Executor 框架实现的，该框架中用到了 Executor，Executors，ExecutorService，ThreadPoolExecutor 这几个类**



### 线程池参数说明

- corePoolSize 线程池的核心线程数
- maximumPoolSize 能容纳的最大线程数
- keepAliveTime 空闲线程存活时间
- unit 存活的时间单位
- workQueue 存放提交但未执行任务的队列
- threadFactory 创建线程的工厂类
- handler 等待队列满后的拒绝策略

线程池中，有三个重要的参数，决定影响了拒绝策略：

- corePoolSize - 核心线程数，也即最小的线程数。
- workQueue - 阻塞队列 。
-  maximumPoolSize -最大线程数当提交任务数大于 corePoolSize 的时候，会优先将任务放到 workQueue 阻塞队列中。当阻塞队列饱和后，会扩充线程池中线程数，直到达到maximumPoolSize 最大线程数配置。此时，再多余的任务，则会触发线程池的拒绝策略了。

**当提交的任务数大于（workQueue.size() +maximumPoolSize ），就会触发线程池的拒绝策略。**



### 拒绝策略

**CallerRunsPolicy**: 当触发拒绝策略，只要线程池没有关闭的话，则使用调用线程直接运行任务。一般并发比较小，性能要求不高，不允许失败。但是，由于调用者自己运行任务，如果任务提交速度过快，可能导致程序阻塞，性能效率上必然的损失较大

**AbortPolicy**: 丢弃任务，并抛出拒绝执行 RejectedExecutionException 异常信息。线程池默认的拒绝策略。必须处理好抛出的异常，否则会打断当前的执行流程，影响后续的任务执行。

**DiscardPolicy**: 直接丢弃，其他啥都没有

**DiscardOldestPolicy**: 当触发拒绝策略，只要线程池没有关闭的话，丢弃阻塞队列 workQueue 中最老的一个任务，并将新任务加入



## 线程的使用方式

- Executors.newFixedThreadPool(int i);   一池多线程
- Executors.newSingleThreadExecutor();  一个任务一个任务执行,一池一线程
- Executors.newCachedThreadPool();	线程池根据需求创建线程,可扩容

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        //一池五线程
        //ExecutorService threadPool1 = Executors.newFixedThreadPool(5);
        //一池一线程
       // ExecutorService threadPool2 = Executors.newSingleThreadExecutor();
        //一池可扩容线程
        ExecutorService threadPool3 = Executors.newCachedThreadPool();
        //10个顾客
        try {
            for (int i = 1; i <= 20; i++) {
                threadPool3.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + " 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool3.shutdown();
        }
    }
}
```



## 线程池的种类与创建

### **newCachedThreadPool(常用)**

**作用**：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程.

**特点**:

- 线程池中数量没有固定，可达到最大值（Interger. MAX_VALUE）
- 线程池中的线程可进行缓存重复利用和回收（回收默认时间为 1 分钟）
- 当线程池中，没有可用线程，会重新创建一个线程

**创建方式：**

```java
/**
 * 可缓存线程池
 * @return
 */
public static ExecutorService newCachedThreadPool(){
    /**
     * corePoolSize 线程池的核心线程数
     * maximumPoolSize 能容纳的最大线程数
     * keepAliveTime 空闲线程存活时间
     * unit 存活的时间单位
     * workQueue 存放提交但未执行任务的队列
     * threadFactory 创建线程的工厂类:可以省略
     * handler 等待队列满后的拒绝策略:可以省略
     */
    return new ThreadPoolExecutor(0,
            Integer.MAX_VALUE,
            60L,
            TimeUnit.SECONDS,
            new SynchronousQueue<>(),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());
}
```

**场景:**适用于创建一个可无限扩大的线程池，服务器负载压力较轻，执行时间较短，任务多的场景



### newFixedThreadPool(常用)

**作用**

​	创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在

**特征:**

- 线程池中的线程处于一定的量，可以很好的控制线程的并发量
- 线程可以重复被使用，在显示关闭之前，都将一直存在
- 超出一定量的线程被提交时候需在队列中等待

```java
/**
 * 固定长度线程池
 *
 * @return
 */
public static ExecutorService newFixedThreadPool() {
    /**
     * corePoolSize 线程池的核心线程数
     * maximumPoolSize 能容纳的最大线程数
     * keepAliveTime 空闲线程存活时间
     * unit 存活的时间单位
     * workQueue 存放提交但未执行任务的队列
     * threadFactory 创建线程的工厂类:可以省略
     * handler 等待队列满后的拒绝策略:可以省略
     */
    return new ThreadPoolExecutor(10,
            10,
            0L,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());
}
```

**场景:**适用于可以预测线程数量的业务中，或者服务器负载较重，对线程数有严格限制的场景





### **newSingleThreadExecutor(常用)**

**作用：**

创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。（注意，如果因为在关闭前的执行期间出现失败而终止了此单个线程，那么如果需要，一个新线程将代替它执行后续的任务）。可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。与其他等效的newFixedThreadPool 不同，可保证无需重新配置此方法所返回的执行程序即可使用其他的线程。

**特征：** 线程池中最多执行 1 个线程，之后提交的线程活动将会排在队列中以此执行

**创建方式：**

```java
/**
 * 单一线程池
 * @return
 */
public static ExecutorService newSingleThreadExecutor(){
    /**
     * corePoolSize 线程池的核心线程数
     * maximumPoolSize 能容纳的最大线程数
     * keepAliveTime 空闲线程存活时间
     * unit 存活的时间单位
     * workQueue 存放提交但未执行任务的队列
     * threadFactory 创建线程的工厂类:可以省略
     * handler 等待队列满后的拒绝策略:可以省略
     */
    return new ThreadPoolExecutor(1,
            1,
            0L,
            TimeUnit.SECONDS,
            new LinkedBlockingDeque<>(),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.AbortPolicy());
}
```

**场景:**适用于需要保证顺序执行各个任务，并且在任意时间点，不会同时有多个线程的场景



### **newScheduleThreadPool(了解)**

**作用:** 线程池支持定时以及周期性执行任务，创建一个 corePoolSize 为传入参数，最大线程数为整形的最大数的线程池**

**特征:**

1. 线程池中具有指定数量的线程，即便是空线程也将保留 
2. 可定时或者延迟执行线程活动

**创建方式:**

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize,ThreadFactory threadFactory){
    return new ScheduledThreadPoolExecutor(corePoolSize,threadFactory);
}
```

**场景:** 适用于需要多个后台线程执行周期任务的场景



### newWorkStealingPool

jdk1.8 提供的线程池，底层使用的是 ForkJoinPool 实现，创建一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用 cpu 核数的线程来并行执行任务

```java
public static ExecutorService newWorkStealingPool(int parallelism){
    /**
     * parallelism：并行级别，通常默认为 JVM 可用的处理器个数
     * factory：用于创建 ForkJoinPool 中使用的线程。
     * handler：用于处理工作线程未处理的异常，默认为 null
     * asyncMode：用于控制 WorkQueue 的工作模式:队列---反队列
     **/
    return new ForkJoinPool(parallelism,
            ForkJoinPool.defaultForkJoinWorkerThreadFactory,
            null,
            true);
}
```





## **线程池底层工作原理(重要)**

![thread4](http://qiliu.luxiaobai.cn/img/thread4.png)

- 线程池只有执行execute()方法时,才真正创建线程.
- 当常驻线程池数(corePool)满了之后,再来新的任务,则添加到阻塞队列(workQueue)进行等待
- 当阻塞队列满时,再来新的任务,则创建新的线程(maximumPoolSize)进行处理新来的任务,阻塞队列中的任务继续等待
- 当最大线程数(maximumPoolSize)满了之后,在来新的任务,则触发拒绝策略(handler)



### **JDK内置的拒绝策略**

- AbortPolicy(默认):直接抛出RejectedExecutionException异常阻止系统正常运行
- CallerRunsPolicy:“调用者运行”一种调节机制,该策略即不会抛弃任务,也不会抛出异常,而是将某些任务回退到调用者,从而降低新任务的流量
- DiscardOldestPolicy:抛弃队列中等待最久的任务,然后把当前任务加入队列中尝试再次提交当前任务
- DiscardPolicy:该策略默默地丢弃无法处理的任务,不予任何处理也不抛出异常.如果允许任务丢失,这是最好的一种策略.





### **注意事项(重要)**

1. 项目中创建多线程时，使用常见的三种线程池创建方式，单一、可变、定长都有一定问题，原因是 FixedThreadPool 和 SingleThreadExecutor 底层都是用LinkedBlockingQueue 实现的，这个队列最大长度为 Integer.MAX_VALUE，容易导致 OOM。所以实际生产一般自己通过 ThreadPoolExecutor 的 7 个参数，自定义线程池
2. 创建线程池推荐适用 ThreadPoolExecutor 及其 7 个参数手动创建

- - corePoolSize 线程池的核心线程数
  - maximumPoolSize 能容纳的最大线程数
  - keepAliveTime 空闲线程存活时间
  - unit 存活的时间单位
  - workQueue 存放提交但未执行任务的队列
  - threadFactory 创建线程的工厂类
  - handler 等待队列满后的拒绝策略

3.为什么不允许适用不允许 Executors.的方式手动创建线程池,如下图

![thread5](http://qiliu.luxiaobai.cn/img/thread5.png)



# Fork/Join

Fork/Join 它可以将一个大的任务拆分成多个子任务进行并行处理，最后将子任务结果合并成最后的计算结果，并进行输出。Fork/Join 框架要完成两件事情：

```markdown
Fork:把一个复杂任务进行分拆,大事化小
Join:把分拆任务的结果进行合并
```

![thread6](http://qiliu.luxiaobai.cn/img/thread6.png)

1. **任务分割：**首先 Fork/Join 框架需要把大的任务分割成足够小的子任务，如果子任务比较大的话还要对子任务进行继续分割
2. **执行任务并合并结果**：分割的子任务分别放到双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都放在另外一个队列里，启动一个线程从队列里取数据，然后合并这些数据



在 Java 的 Fork/Join 框架中，使用两个类完成上述操作

- **ForkJoinTask**:我们要使用 Fork/Join 框架，首先需要创建一个 ForkJoin 任务。该类提供了在任务中执行 fork 和 join 的机制。通常情况下我们不需要直接集成 ForkJoinTask 类，只需要继承它的子类，Fork/Join 框架提供了两个子类：

- - a.RecursiveAction：用于没有返回结果的任务
  - b.RecursiveTask:用于有返回结果的任务

- **ForkJoinPool**:ForkJoinTask 需要通过 ForkJoinPool 来执行

- **RecursiveTask**: 继承后可以实现递归(自己调自己)调用的任务



## **Fork/Join 框架的实现原理**

ForkJoinPool 由 **ForkJoinTask 数组**和 **ForkJoinWorkerThread 数组**组成，ForkJoinTask 数组负责将存放以及将程序提交给 ForkJoinPool，而ForkJoinWorkerThread 负责执行这些任务。



## Fork方法

![thread7](http://qiliu.luxiaobai.cn/img/thread7.png)



![thread8](http://qiliu.luxiaobai.cn/img/thread8-2234925.png)

### **Fork 方法的实现原理**

当我们调用 ForkJoinTask 的 fork 方法时，程序会把任务放在 ForkJoinWorkerThread 的 pushTask的 workQueue 中，异步地执行这个任务，然后立即返回结果

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);
    return this;
}
```

pushTask 方法把当前任务存放在 ForkJoinTask 数组队列里。然后再调用ForkJoinPool 的 signalWork()方法唤醒或创建一个工作线程来执行任务。代码如下：

```java
final void push(ForkJoinTask<?> task) {
    ForkJoinTask<?>[] a; ForkJoinPool p;
    int b = base, s = top, n;
    if ((a = array) != null) { // ignore if queue removed
        int m = a.length - 1; // fenced write for task visibility
        U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);
        U.putOrderedInt(this, QTOP, s + 1);
        if ((n = s - b) <= 1) {
            if ((p = pool) != null)
                p.signalWork(p.workQueues, this);//执行
        }
        else if (n >= m)
            growArray();
    }
}
```



## join方法

Join 方法的主要作用是阻塞当前线程并等待获取结果

```java
public final V join() {
    int s;
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        reportException(s);
        return getRawResult();
    }
```

它首先调用 doJoin 方法，通过 doJoin()方法得到当前任务的状态来判断返回什么结果，任务状态有 4 种：

**==已完成（NORMAL）、被取消（CANCELLED）、信号（SIGNAL）和出现异常（EXCEPTIONAL）==**

- 如果任务状态是已完成，则直接返回任务结果。
- 如果任务状态是被取消，则直接抛出 CancellationException
- 如果任务状态是抛出异常，则直接抛出对应的异常



doJoin()方法流程:

1. 首先通过查看任务的状态，看任务是否已经执行完成，如果执行完成，则直接返回任务状态；
2. 如果没有执行完，则从任务数组里取出任务并执行。
3. 如果任务顺利执行完成，则设置任务状态为 NORMAL，如果出现异常，则记录异常，并将任务状态设置为 EXCEPTIONAL。



## Fork/Join框架的异常处理

ForkJoinTask 在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，所以 ForkJoinTask 提供了 isCompletedAbnormally()方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过 ForkJoinTask 的

getException 方法获取异常。getException 方法返回 Throwable 对象，如果任务被取消了则返回CancellationException。如果任务没有完成或者没有抛出异常则返回 null。



### 案例

```java
public class TaskExample extends RecursiveTask<Long> {
    private int start;
    private int end;
    private long sum;

    public TaskExample(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        System.out.println("任务：" + start + "=======" + end + "累加开始");
        //大于100个数相加切分，小于直接加
        if (end-start<=100){
            for (int i = start;i<=end;i++){
                //累加
                sum+=i;
            }
        }else{
            //切分为2块
            int middle = start + 100;
            //递归调用，切分为2个小任务
            TaskExample taskExample1 = new TaskExample(start, middle);
            TaskExample taskExample2 = new TaskExample(middle + 1, end);
            //执行：异步
            taskExample1.fork();
            taskExample2.fork();
            //同步阻塞获取执行结果
            sum = taskExample1.join() + taskExample2.join();
        }
        return sum;
    }
}

public class ForkJoinPoolDemo {
    public static void main(String[] args) {
        //定义任务
        TaskExample taskExample = new TaskExample(1, 1000);
        //定义执行对象
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //加入任务执行
        ForkJoinTask<Long> submit = forkJoinPool.submit(taskExample);
        try {
            System.out.println(submit.get());
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            forkJoinPool.shutdown();
        }
    }
}
```





# Thread

## 创建线程方式

### 继承Thread类

```java
//创建线程方式一：继承Thread类，重写run方法，调用start开启线程
public class TestThread1 extends Thread{
    @Override
    public void run() {

        for (int i = 0; i<20;i++){
            System.out.println("我在看代码" + i);
        }
    }

    public static void main(String[] args) {
        //main线程 主线程

        //创建一个线程对象
        TestThread1 testThread1 = new TestThread1();
        //调用start方法开启线程
        testThread1.start();

        for (int i = 0;i<20;i++){
            System.out.println("I'm see code " + i);
        }
    }
}
```

- 自定义线程类继承Thread类
- 重写run()方法,编写线程执行体
- 创建线程对象,调用start()方法启动线程

**注意 线程开启不一定立即执行,由CPU调度执行**



```java
//实现多线程同步下载图片
public class TestThread2 extends Thread {
    private String url;//网络图片地址
    private String fileName;//保存的文件名

    public TestThread2(String url, String fileName) {
        this.url = url;
        this.fileName = fileName;
    }

    @Override
    public void run() {
        WebDownloader webDownloader = new WebDownloader();
        webDownloader.downloader(url,fileName);
        System.out.println("下载文件名为：" + fileName);
    }

    public static void main(String[] args) {
        TestThread2 testThread1 = new TestThread2("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","1.jpg");
        TestThread2 testThread2 = new TestThread2("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","2.jpg");
        TestThread2 testThread3 = new TestThread2("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","3.jpg");
        testThread2.start();
        testThread1.start();
        testThread3.start();
    }
}

//下载器
class WebDownloader {
    public void downloader(String url, String name) {
        try {
            FileUtils.copyURLToFile(new URL(url), new File(name));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



### 实现Runnable

- 实现MyRunnable类实现Runnable接口
- 实现run方法,编写线程执行体
- 创建线程对象,调用start()方法启动线程

```java
//实现runnable接口，重写run方法，执行线程需要丢入runnable接口实现类，调用start
public class TestThread3 implements Runnable{
    @Override
    public void run() {

        for (int i = 0; i<20;i++){
            System.out.println("我在看代码" + i);
        }
    }

    public static void main(String[] args) {
        //创建runnable接口的实现类对象
        TestThread3 testTh  = new TestThread3();

        //创建线程对象，通过线程对象来开启线程，代理
        Thread thread = new Thread(testTh);
        thread.start();

//        new Thread(testTh).start();

        for (int i = 0;i<20;i++){
            System.out.println("I'm see code " + i);
        }
    }
}
```



### 小结

#### **继承Thread类**

- 子类继承Thread类具备多线程能力
- 启动线程:子类对象 start()
- 不建议使用:避免OOP单继承局限性

#### **实现Runnable接口**

- 实现接口Runnable具有多线程能力
- 启动线程:传入目标对象+Thread对象.start()
- 推荐使用:避免单继承局限性,灵活方便,方便同一个对象被多个线程使用



```java
//发现问题：多个线程操作同一个资源的情况下，线程不安全，数据紊乱
public class TestThread4 implements Runnable {
    private int ticketNum = 50;

    @Override
    public void run() {
        while (true) {
            if (ticketNum <= 0) {
                break;
            }
            //模拟延时
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ">>拿到了" + ticketNum-- + "票");
        }
    }

    public static void main(String[] args) {
        TestThread4 testThread4 = new TestThread4();
        new Thread(testThread4, "小米").start();
        new Thread(testThread4, "小明").start();
        new Thread(testThread4, "黄牛").start();
    }
}
```



### 实例龟兔赛跑

```java
//模拟龟兔赛跑
public class Race implements Runnable {

    private static String winner;

    @Override
    public void run() {
        for (int i = 0; i <= 100; i++) {
            //模拟兔子休息
            if (Thread.currentThread().getName().equals("兔") && i % 10 == 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            //判断比赛是否结束
            boolean flag = gameOver(i);
            //如果比赛结束，就停止程序
            if (flag) {
                break;
            }
            System.out.println(Thread.currentThread().getName() + "-->跑了" + i + "步");
        }
    }

    //判断是否完成比赛
    private boolean gameOver(int steps) {
        //判断是否有胜利者
        if (winner != null) {//已经存在胜利者
            return true;
        } else {
            if (steps >= 100) {
                winner = Thread.currentThread().getName();
                System.out.println("winner is:" + winner);
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        Race race = new Race();
        new Thread(race, "龟").start();
        new Thread(race, "兔").start();
    }

}
```



### 实现Callable接口

1. 实现Callable接口,需要返回值类型
2. 重写call方法,需要抛出异常
3. 创建目标对象
4. 创建执行服务:ExecutorService ser = Executors.newFixedThreadPool(1);
5. 提交执行:Future<Boolean> result1 = ser.submit(t1);
6. 获取结果:boolean r1 = result1.get()
7. 关闭服务:ser.shutdownNow();

```java
/**
 * callable
 * 1 可以定义返回值
 * 2 可以抛出异常
 */
public class TestCallable implements Callable {
    private String url;//网络图片地址
    private String fileName;//保存的文件名

    public TestCallable(String url, String fileName) {
        this.url = url;
        this.fileName = fileName;
    }

    @Override
    public Boolean call() {
        WebDownloader2 webDownloader = new WebDownloader2();
        webDownloader.downloader(url,fileName);
        System.out.println("下载文件名为：" + fileName);
        return true;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        TestCallable testThread1 = new TestCallable("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","1.jpg");
        TestCallable testThread2 = new TestCallable("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","2.jpg");
        TestCallable testThread3 = new TestCallable("https://img2.baidu.com/it/u=2872135032,4144008915&fm=15&fmt=auto&gp=0.jpg","3.jpg");

        //创建执行服务
        ExecutorService service = Executors.newFixedThreadPool(3);

        //提交执行
        Future<Boolean> r1 = service.submit(testThread1);
        Future<Boolean> r2 = service.submit(testThread2);
        Future<Boolean> r3 = service.submit(testThread3);

        //获取结果
        boolean res1 = r1.get();
        boolean res2 = r2.get();
        boolean res3 = r3.get();

        //关闭服务
        service.shutdownNow();

    }

}
//下载器
class WebDownloader2 {
    public void downloader(String url, String name) {
        try {
            FileUtils.copyURLToFile(new URL(url), new File(name));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



```java
//静态代理模式总结：
    //真实对象和代理对象都要实现同一个接口
//好处
    //代理对象可以做很多真实对象做不了的事情
    //真实对象专注做自己的事情
public class StaticProxy {
    public static void main(String[] args) {
      new Thread(() -> System.out.println("i l y")).start();

      new WeddingCompany(new You()).HappyMarry();
    }

}

interface Marry{
    void HappyMarry();
}

class You implements Marry{

    @Override
    public void HappyMarry() {
        System.out.println("结婚了");
    }
}

//代理角色，帮助你结婚
class WeddingCompany implements Marry{
    //代理谁-》真实目标对象
    private Marry target;

    public WeddingCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void HappyMarry() {
        before();
        this.target.HappyMarry();
        after();
    }

    private void after(){
        System.out.println("结婚之后，收红包");
    }

    private void before(){
        System.out.println("结婚之前，布置");
    }
}
```



## Lambda表达式

是一个匿名函数，即没有函数名的函数.

使用它设计的代码会更加简洁、可读

### **函数式接口**

- 任何接口,如果只包含唯一一个抽象方法,那么它就是一个函数式接口
- 对于函数式接口,可以通过lambda表达式来创建该接口的对象



```java
public class TestLambda {

    //3 静态内部类
    static class Like2 implements ILike {
        @Override
        public void lambda() {
            System.out.println("I like lambda2");
        }
    }

    public static void main(String[] args) {
        ILike like = new Like();
        like.lambda();

        like = new Like2();
        like.lambda();


        //4 局部内部类
        class Like3 implements ILike {
            @Override
            public void lambda() {
                System.out.println("I like lambda3");
            }
        }

        like = new Like3();
        like.lambda();

        //5 匿名内部类, 没有类的名称，必须借助接口或者父类
        like = new ILike() {
            @Override
            public void lambda() {
                System.out.println("I like lambda4");
            }
        };
        like.lambda();

        //6 用lambda简化
        like = ()->{
            System.out.println("I like lambda5");
        };
        like.lambda();
    }
}

//1 定义一个函数式接口
interface ILike {
    void lambda();
}

//2 实现类
class Like implements ILike {

    @Override
    public void lambda() {
        System.out.println("I like lambda");
    }
}
```



## 线程状态

![thread9](http://qiliu.luxiaobai.cn/img/thread9.png)

### 线程方法

- setPriority(int newPriority): 更改线程的优先级
- static void sleep(long millis): 在指定的毫秒数内让当前正在执行的线程休眠
- void join(): 等待该线程终止
- static void yield() : 暂停当前正在执行的线程对象,并执行其他线程
- void interrupt(): 中断线程,别用这个方式
- boolean isAlive():测试线程是否处于活动状态



#### 停止线程

- 不推荐使用JDK提供的stop()、destroy()方法(已废弃)(前者会产生异常,后者是强制终止,不会释放锁)
- 推荐线程自己停止下来
- 建议使用一个标志位进行终止变量,当flag=false,则终止线程运行

```java
//1 建议线程正常停止-->利用次数，不建议死循环
//2 建议使用标志位--->设置一个标志位
//3 不要使用stop或  destroy等过时或者JDK不建议的方法
public class TestStop implements Runnable{
    //1 线程中定义线程体使用的标识
    private boolean flag = true;

    @Override
    public void run() {
        //2 线程体使用该标识
        while(flag){
            System.out.println("run...Thread");
        }
    }

    //对外提供方法改变标识
    public void stop(){
        this.flag = false;
    }
}
```



#### 线程休眠

- sleep(时间)指定当前线程阻塞的毫秒数;
- sleep存在异常InterruptedException;
- sleep时间达到后线程进入就绪状态;
- sleep可以模拟网络延时,倒计时等
- 每一个对象都有一个锁,sleep不会释放锁



#### 线程礼让

- 礼让线程,让当前正在执行的线程暂停,但不阻塞
- 将线程从运行状态转为就绪状态
- 让CPU重新调度,礼让不一定成功,看CPU

```java
public class TestYield {
    public static void main(String[] args) {
        MyYield myYield = new MyYield();
        new Thread(myYield, "a").start();
        new Thread(myYield, "b").start();
    }
}

class MyYield implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程开始执行");
        Thread.yield();//礼让
        System.out.println(Thread.currentThread().getName() + "线程停止执行");
    }
}
```



#### Join

- Join合并线程,待此线程执行完成后,再执行其他线程,其他线程阻塞 
- 可以想象成插队

```java
public class TestJoin implements Runnable{
    @Override
    public void run() {
        for (int i = 0;i< 1000;i++){
            System.out.println("线程VIP来了" + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        TestJoin testJoin = new TestJoin();
        Thread thread = new Thread(testJoin);
//        thread.start();
        for (int i =0;i<500;i++){
            if (i == 200){
                thread.start();
                thread.join();
            }
            System.out.println("main" + i);
        }
    }
}
```



### **线程状态观测**

-  Thread.State

- - NEW

  - - 尚未启动的线程处于此状态

  - RUNNABLE

  - - 在Java虚拟机中执行的线程处于此状态

  - BLOCKED

  - - 被阻塞等待监视器锁定的线程处于此状态

  - WAITING

  - - 正在等待另一个线程执行特定动作的线程处于此状态

  - TIMED_WAITING

  - - 正在等待另一个线程执行动作达到指定等待时间的线程处于此状态

  - TERMINATED

  - - 已退出的线程处于此状态

一个线程可以在给定时间点处于一个状态.这些状态是不反映任务操作系统线程状态的虚拟机状态

```java
public class TestState {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("///////////////");
        });

        //观测状态
        Thread.State state = thread.getState();
        System.out.println(state); //new

        //观测启动后
        thread.start();//启动线程
        state = thread.getState();
        System.out.println(state);

        while (state != Thread.State.TERMINATED) {//只要线程不终止，就一直输出状态
            Thread.sleep(100);
            state = thread.getState();//更新线程状态
            System.out.println(state);//输出状态
        }
    }
}
```



## 线程优先级

- Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程,线程调度器按照优先级决定应该调度那个线程来执行

- 线程的优先级用数字表示,范围从1~10.

- - Thread.MIN_PRIORITY = 1;
  - Thread.MAX_PRIORITY = 10;
  - Thread.NORM_PRIORITY = 5;

- 使用以下方式改变或获取优先级

- - getPriority() setPriority(int xxx)



```java
public class TestPriotiry {
    public static void main(String[] args) {
        //主线程默认优先级
        System.out.println(Thread.currentThread().getName()+ "-->" + Thread.currentThread().getPriority());

        MyPriotriy myPriotriy = new MyPriotriy();

        Thread t1 = new Thread(myPriotriy);
        Thread t2 = new Thread(myPriotriy);
        Thread t3 = new Thread(myPriotriy);
        Thread t4 = new Thread(myPriotriy);
        Thread t5 = new Thread(myPriotriy);
        Thread t6 = new Thread(myPriotriy);

        //先设置优先级，在启动
        t1.start();

        t2.setPriority(1);
        t2.start();

        t3.setPriority(4);
        t3.start();

        t4.setPriority(Thread.MAX_PRIORITY);
        t4.start();

        t5.setPriority(7);
        t5.start();

        t6.setPriority(8);
        t6.start();

    }
}

class MyPriotriy implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() +"-->" + Thread.currentThread().getPriority());
    }
}
```

**优先级的设定建议在start()调度前**

**优先级低意味着获得调度的概率低,并不是优先级低就不会被调用了,这都是看CPU的调度**



## 守护(daemon)线程

- 线程分为**用户线程**和**守护线程**
- 虚拟机必须确保用户线程执行完毕
- 虚拟机不用等待守护线程执行完毕
- 如:后台记录操作日志,监控内存,垃圾回收等待

```java
public class TestDaemon {
    public static void main(String[] args) {
        God god = new God();
        Thread thread = new Thread(god);
        thread.setDaemon(true);//默认是false表示用户线程，正常的线程都是用户线程
        thread.start();

        You2 you2 = new You2();
        Thread thread1 = new Thread(you2);
        thread1.start();
    }
}

class God implements Runnable{

    @Override
    public void run() {
        while (true){
            System.out.println("上帝保佑着你");
        }
    }
}


class You2 implements Runnable{
    @Override
    public void run() {
        for (int i =0 ;i<36500;i++){
            System.out.println("你一生都开心活着");
        }
        System.out.println("-=======goodbye world!======");
    }
}
```



## 线程同步

### 队列+锁

- 为保证数据在方法中被访问时的正确性,在访问时加入锁机制synchronized,当一个线程获得对象的排它锁,独占资源,其他线程必须等待,使用后释放锁即可,存在以下问题:

- - 一个线程持有锁回导致其他所欲需要此锁的线程挂起;
  - 在多线程竞争下,加锁,释放锁会导致比较多的上下文切换和调度延时,引起性能问题;
  - 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置,引起性能问题 

```java
//线程不安全的买票
public class UnsafeBufTicket {
    public static void main(String[] args) {
        BuyTicket buyTicket = new BuyTicket();

        new Thread(buyTicket, "我").start();
        new Thread(buyTicket, "你").start();
        new Thread(buyTicket, "黄牛").start();
    }
}

class BuyTicket implements Runnable{
    //票
    private int ticketNums = 10;
    boolean flag = true;//外部停止方式

    @Override
    public void run() {
        //买票
        while (flag){
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    private void buy() throws InterruptedException {
        if(ticketNums<=0){
            flag = false;
            return;
        }
        //模拟延时
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName() + "拿到" + ticketNums--);
    }
}


public class UnsafeBank {
    public static void main(String[] args) {
        Account account = new Account(50,"基金");
        Drawing you = new Drawing(account,25,"你");
        Drawing girlFriend = new Drawing(account,50,"girlFriend");
        you.start();
        girlFriend.start();
    }
}

class Account{
    int money;
    String name;

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }
}

class Drawing extends Thread{
    Account account;
    int drawingMoney;
    int nowMoney;

    public Drawing(Account account,int drawingMoney,String name){
        super(name);
        this.account = account;
        this.drawingMoney = drawingMoney;

    }

    @Override
    public void run() {
        if(account.money-drawingMoney<0){
            System.out.println(Thread.currentThread().getName() + "钱不够");
            return;
        }
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //卡内余额 = 余额-你取的钱
        account.money = account.money-drawingMoney;
        //你手里的钱
        nowMoney = nowMoney + drawingMoney;
        System.out.println(account.name + "余额为：" + account.money);
        //Thread.currentThread().getName() = this.getName()
        System.out.println(this.getName() + "手里的钱：" + nowMoney);
    }
}
```



### 同步块

- 同步块:synchronized(Obj){}

- Obj称之为同步监视器

- - Obj可以是任何对象,但是推荐使用共享资源作为同步监视器
  - 同步方法中无需指定不同监视器,因为同步方法的同步监视器就是this,就是这个对象本身,或者是class

- 同步监视器的执行过程

- - 第一个线程访问,锁定同步监视器,执行其中代码
  - 第二个线程访问,发现同步监视器被锁定,无法访问
  - 第一个线程访问完毕,解锁同步监视器
  - 第二个线程访问,发现同步监视器没有锁,然后锁定并访问



### 死锁

- 多个线程各自占有一些共享资源,并且互相等待其他线程占有的资源才能运行,而导致两个或者多个线程都在等待对方释放资源,都停止执行的情形.某一个同步块同时拥有“两个以上对象的锁”时,就可能会发生“死锁”的问题



## Lock锁

- 通过显式定义同步锁对象来实现同步.同步锁使用Lock对象充当
- java.util.concurrent.locks.Lock接口时控制多个线程对共享资源进行访问的工具.锁提供了对共享资源的独占访问,每次只能有一个线程对Lock对象加锁,线程开始访问共享资源之前应先获得Lock对象
- ReentrantLock类实现类Lock,它拥有与synchronized相同的并发性和内存活义,在实现线程安全的控制总,比较常用的是ReentrantLock,可以显式加锁、释放锁

```java
public class Testlock {
    public static void main(String[] args) {
        Lock lock = new Lock();
        new Thread(lock).start();
        new Thread(lock).start();
        new Thread(lock).start();
    }
}

class Lock implements Runnable{

    int tickNum =10;
    private final ReentrantLock lock = new ReentrantLock();
    @Override
    public void run() {
        while (true){
            try {
                lock.lock();//加锁
                if(tickNum>0){
                    try{
                        Thread.sleep(1000);
                    }catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(tickNum--);
                }else{
                    break;
                }
            } finally {
                lock.unlock();//解锁
            }
        }
    }
}
```



### **synchronized与Lock对比**

- Lock是显式锁(手动开启和关闭锁,别忘记关闭锁)synchronized是隐式锁,出了作用域自动释放

- Lock只有代码块锁,synchronized有代码块锁和方法锁

- 使用Lock锁,JVM将花费较少的时间来调度线程,性能更好.并且具有更好的扩展性(提供更多的子类)

- 优先使用顺序

- - Lock>同步代码块(已经进入了方法体,分配了相应资源)>同步方法(在方法体之外)



## 线程通信

- wait():表示线程一直等待,直到其他线程通知,与sleep不同,会释放锁
- wait(long timeout): 指定等待的毫秒数
- notify():唤醒一个处于等待状态的线程
- notifyAll():唤醒同一个对象上所有调用wait()方法的线程,优先级别高的线程优先调度



### 解决方式1-并发协作模型“生产者/消费者模式”-->管程法

生产者将生产好的数据放入缓冲区,消费者从缓冲区拿出数据

```java
public class TestPC {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Provider(container).start();
        new Consumer(container).start();
    }
}

class Provider extends Thread {
    SynContainer container;

    public Provider(SynContainer container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了" + i + "只鸡");
        }
    }
}

class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer container) {
        this.container = container;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了-->" + container.pop().id + "鸡");
        }
    }
}

class Chicken {
    int id;

    public Chicken(int id) {
        this.id = id;
    }
}

class SynContainer {
    //需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    //容器计数器
    int count = 0;

    //生产者放入产品
    public synchronized void push(Chicken chicken) {
        //如果容器满了，就需要等待消费者消费
        if (count == chickens.length) {
            //通知消费者消费，生产者等待
            try{
                this.wait();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
        //如果没有满
        chickens[count] = chicken;
        count++;
        //通知消费者消费了
        this.notifyAll();

    }

    //消费者消费产品
    public synchronized Chicken pop() {
        if (count == 0) {
            //等待生产者生产，消费者等待
            try{
                this.wait();
            }catch(InterruptedException e){
                e.printStackTrace();
            }
        }
        count--;
        Chicken chicken = chickens[count];

        //吃完了，通知生产者生产
        this.notifyAll();
        return chicken;
    }
}
```



### **解决方式2-并发协作模型 “生产者/消费者模式”-->信号灯法**

```java
public class TestPC2 {
    public static void main(String[] args) {
        TV tv = new TV();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}

//生产者->演员
class Player extends Thread {
    TV tv ;
    public Player(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if(i%2==0){
                this.tv.play("快乐大本营");
            }else{
                this.tv.play("记录美好生活");
            }
        }
    }
}

//消费者->观众
class Watcher extends Thread {
    TV tv;
    public Watcher(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

//产品->节目
class TV {
    //演员表演，观众等待  T
    //观众观看，演员等待  F
    String voice;
    boolean flag = true;

    //表演
    public synchronized void play(String voice) {
        System.out.println("演员表演了" + voice);
        if (!flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //通知观众观看
        this.notifyAll();
        this.voice = voice;
        this.flag = !this.flag;
    }

    public synchronized void watch() {
        if (flag) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("观看了：" + voice);
        //通知演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }
}
```









































































































































































































































































































































































































































































































































































































































































































































































