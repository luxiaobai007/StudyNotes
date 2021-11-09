[toc]



# JVM指令

## 1、加载或存储指令

### 将局部变量加载到操作栈中:

**ILOAD**(将int类型的局部变量压入栈)和**ALOAD**(将对象引用的局部变量压入栈中)



### 从操作栈顶存储到局部变量表

**ISTORE**、**ASTORE**等



### 将常量加载到操作栈顶,这是极为高频使用的指令

**ICONST**、**BIPUSH**、**SIPUSH**、**LDC**等

- ICONST加载的是-1~5的数(ICONST与BIPUSH的加载界限)

- BIPUSH,即Byte Immediate PUSH, 加载-128~127之间的数

- SIPUSH,即Short Immediate PUSH, 加载-32768 ~ 32767之间的数

- ###### LDC,即Load Constant,-2^32 ~ 2^32-1或者是字符串时,JVM采用LDC指令压入栈中

  ```java
  int a = -2;				L5 BIPUSH -1 //在-1与5之外的数字使用BIPUSH指令加载
  int b = -1;				L6 ICONST_M1//-1, 直接使用ICONST加载的最小值
  int c = 0;				L7 ICONST_0
  int e = 20000;    L8 SIPUSH 20000
  int f = 40000;		L9 LDC 40000
  ```

  

## 2、运算指令

#### 对两个操作栈帧上的值进行运算,并把结果写入操作栈顶

**IADD**,**IMUL**等



## 3、类型转换指令

### 显式转换两种不同的数值类型

**I2L**、**D2F**等



## 4、对象创建于访问指令

1. 创建对象指令: **NEW**、**NEWARRAY**等
2. 访问属性指令: **GETFIELD**、**PUTFIELD**、**GETSTATIC**等
3. 检查实例类型指令: **INSTANCEOF**、**CHECKCAST**等



## 5、操作栈管理指令

1. 出栈操作:**POP**即一个元素,**POP2**即两个元素
2. 复制栈顶元素并压入栈: **DUP**



## 6、方法调用与返回指令

1. **INVOKEVIRTUAL**指令: 调用对象的实例方法
2. **INVOKESPECIAL**指令: 调用实例初始化方法、私有方法、父类方法等
3. **INVOKESTATIC**指令: 调用类静态方法
4. **RETURN**指令: 返回VOID类型



## 7、同步指令

JVM使用方法结构中的ACC_SYNCHRONIZED标志同步方法,指令集中有MONITORENTER和MONITOREXIT支持synchronized语义





# 内存布局

![image-20210828143719030](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/image-20210828143719030.png)



## Heap(堆区)

Heap是OOM故障最主要的发源地,存储着几乎所有的实例对象,堆由垃圾收集器自动回收,,堆区由各子线程共享使用.

堆内存空间即可以固定大小,也可以在运行时动态地调整,设定初始值和最大值:-Xms256M-Xmx1024M,- X表示它是JVM运行参数,ms是memory start的简称,mx是memory max的简称代表最小堆容量和最大堆容量.运行过程中,堆空间不断的扩容与回缩,势必造成不必要的系统压力,所以Xms Xmx设置成一样的,避免在GC后调整堆大小时带来的额外压力.

堆:

- 新生代: 新生代=1个Eden区+2个Survivor区.
  - 在大部分对象在Eden区生成,当Eden区装填满时,会触发YCG,在Eden区实现清除策略,没有被引用的对象则直接回收,依然存活的对象会被移送到Survivor区.Survivor区有2个区S0、S1.每次YCG时,将存活的对象复制到未使用的那块空间,然后将当前正在使用的空间完全清除,交换两块空间的使用状态.如果移送的对象大于Survivor区容量的上限,则直接移交给老年代.
  - 每个对象都有一个计数器,每次YCG都会加1. - XX:MaxTenuringThreshold参数能配置计数器的值到达某个阈值时,对象从新生代晋升到老年代.如果参数配置为1,那么从新生代的Eden区直接移至老年代.默认值是15,可以在Survivor区交换14次之后,晋升老年代.
- 老年代: 如果Survivor区无法放下,或者超过上限,则尝试在老年代进行分配;如果老年代也无法放下,则触发Full Garbage Collection,即FGC.如果依然无法放下,则抛出OOM.堆内存出现OOM的概率是所有内存耗尽异常中最高的.出错时,堆内信息堆解决问题非常有帮助,所以给JVM设置运行参数 -XX:+HeapDumpOnOutOfMemoryError,让JVM遇到OOM异常时输出堆内存信息.



## Metaspace(元空间)

JDK7以前,只有Hotspot才有Perm区,译为永久代,在启动时固定大小,很难进行调优,并且FGC时会移动类元信息.当动态加载类过多时,容易产生Perm区的OOM,

​	`Exception in thread ‘dubbo client x.x connector' java.lang.OutOfMemoryError: PermGenspace`

为解决该问题,需要设定运行参数`-XX:MaxPermSize=1280m`.如果部署到新机器上,往往因为JVM参数没有修改导致故障在线.所以JDK8使用元空间体会永久代.

元空间在本地内存中分配,Perm区中所有内容中字符串常量移至堆内存,其他内容包括类元信息、字段、静态属性、方法、常量等移动至元空间内 



## JVM Stack(虚拟机栈)

栈(Stack) 是一个先进后出的数据结构

JVM中的虚拟机栈是描述Java方法执行的内存区域,它是线程私有的.栈中的元素用于支持虚拟机进行方法调用,每个方法从开始调用到窒息完成的过程,就是栈帧从入栈到出栈的过程.

在活动线程中,只有位于栈顶的帧才是有效的,即当前栈帧,在执行引擎运行时,所有指令都只能针对当前栈帧进行操作.`StackOverflowError`表示请求的栈溢出,导致内存耗尽,通常出现在递归方法中.

### 局部变量表

存放方法参数和局部变量的区域.,没有准备阶段,必须显式初始化.如果是非静态方法,则在index[0]位置上存储的是方法所属对象的实例引用,随后存储的是参数和局部变量.



### 操作栈

操作栈是一个初始状态为空的桶式结构栈.

```java
public int simpleMethod(){
  int x = 13;
  int y = 14;
  int z = x + y;
  
  return z;
}

//字节码操作顺序
public simpleMethod();
descriptor: ()I
flags: ACC_PUBLIC
Code:
	stack=2, locals=4, args_Size=1 //最大栈深度为2,局部变量个数为4
    BIPUSH 13										//常量13压入操作栈
    ISTORE_1										//并保存到局部变量表的slot_1中
    
    BIPUSH 14										//常量14压入操作栈
    ISTORE_2 										//并保存到局部变量表的slot_2中
    
    ILOAD_1											//把局部变量表的slot_1元素(int x)压入操作栈
    ILOAD_2											//把局部变量表的slot_2元素(int y)压入操作栈
    IADD												//把上方两个数都取出来,在CPU里加一下,并压回操作栈的栈顶
    ISTORE_3										//把栈顶的结果存储到局部变量表的slot_3中
    
    ILOAD_3
    IRETURN											//返回栈顶元素值
```



### 动态连接

每个栈帧中包含一个在常量池中对当前方法的引用,目的是支持方法调用过程的动态连接



### 方法返回地址

方法执行时的退出情况:

1. 正常退出,即正常执行到任何方法的返回字节码指令,如RETURN、IRETURN、ARETURN等
2. 异常退出.

无论何种退出,都将返回至方法当前被调用的位置.

退出方式:

- 返回值压入上层调用栈帧
- 异常信息抛给能够处理的栈帧
- PC计数器指向方法调用后的下一条指令.



## Native Method Stacks(本地方法栈)







## Program Counter Register(程序计数寄存器)

每个线程在创建后,都会产生自己的程序计数器和栈帧,**程序计数器用来存放执行指令的偏移量和行号指示器等,线程执行或恢复都要依赖程序计数器.**

![image-20210828153846810](http://qiliu.luxiaobai.cn/img/image-20210828153846810.png)







# 垃圾回收

## 目的

清除不再使用的对象,自动释放内存.



## GC如何判断对象是否可以被回收的呢?

JVM引入了GC Roots.

如果一个对象与GC Roots之间没有直接或间接的引用关系,比如失去任何引用的对象,或者两个互相环岛循环引用的对象等,是可以回收的.

#### 什么对象可以作为GC Roots?

类静态属性中引用的对象、常量引用的对象、虚拟机栈 中引用的对象、本地方法栈中引用的对象等.



## 垃圾回收的相关算法

### 标记-清除算法

从每个GC Roots出发,依次标记有引用关系的对象,最后将没有被标记的对象清除

缺点:带来大量的空间碎片,容易触发FGC.



### 标记-整理算法

GC Roots出发标记存活的对象,然后将存活的对象整理到内存空间的一端,形成连续的已使用空间,最后把已使用空间之外的部分全部清除.这样就不会有空间碎片的问题.



### Mark-Copy算法

为能够并行的标记和整理,将空间分为两块,每次只激活其中一块,垃圾回收时只需要把存活的对象复制到另一块为激活空间上,将未激活空间标记为已激活,将已激活空间标记为未激活.然后清除原空间中的原对象.减少了内存空间的浪费.





## 垃圾回收器

### Serial

主要应用于YGC的垃圾回收器,采用串行单线程的方式完成GC任务,

STW(Stop The World),即垃圾回收的某个阶段会暂停整个应用程序的执行.FGC的时间相对较长,频繁FGC会严重影响应用程序的性能

![image-20210828143719030](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/image-20210828143719030.png)



### CMS

是回收停顿时间比较短、目前比较常用的垃圾回收器.

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

1、3依然会引发STW

2、4可以和应用程序并发执行,比较耗时,但不影响应用程序的正常执行

CMS采用的是标记-整理算法,会产生大量空间碎片,可以通过配置:`-XX:+UseCMSCompactAtFullCollection`参数,强制JVM在FGC完成后堆老年代进行压缩,执行一次空间碎片整理.整理阶段也会引发STW.

为减少STW次数,可以配置`-XX:CMSFULLGCsBeforeCompaction=n`参数,在执行了n次FGC后,JVM再再老年代执行空间碎片整理.



### G1

G1采用Mark-Copy

在JDK11中,已经将G1设为默认垃圾回收器

通过`-XX:+UseG1GC`参数启用.具备压缩功能,能避免碎片问题

通过`jstat`命令可以查看垃圾回收情况.

