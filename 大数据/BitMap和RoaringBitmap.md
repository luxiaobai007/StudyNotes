---
typora-copy-images-to: upload
---

[toc]

---



# BitMap和RoaringBitmap



## BitMap（位图）

使用BitMap主要是为了**节省存储空间**，主要Bit为单位来存储数据，bit位来标记某个元素对应的Value，而Key即是该元素

>Java中，int占4个字节，1个字节=8位（1byte=8bit)
>
>用int存储数字，20亿个int，占用空间约为：2000000000*4/1024/1024/1024)≈**7.45**G
>
>用位存储，20亿个数据就是20亿位，占用空间约为：2000000000/8/1024/1024/1024)≈**0.2****33**G



### 实例

使用Bit进行存储，每一位代表一个数，0表示不存在，1表示存在。

如果要存储{1，4，5，7}这几个数，则如下所示：一个字节占8位。

![image-20211026104715676](http://qiliu.luxiaobai.cn/img/image-20211026104715676-16424952781281.png)

存储{15,13,12,9}，则在另外一个8位上。就变成了二维数组了

![image-20211026104903861](http://qiliu.luxiaobai.cn/img/image-20211026104903861-16424952844002.png)



而1个int占32位，只需要申请一个int数组长度为 int tmp[1+N/32] 即可存储，其中N表示要存储的这些数中的最大值

> tmp[0]：可以表示0~31
>
> tmp[1]：可以表示32~63
>
> tmp[2]：可以表示64~95
>
> ......

存储某一个数K，则K/32就可以得到存储数据的下标，K%32可以得到此下标在那个位置

比如数字5，则5/32=0，5%32=5，则在tmp[0],第5个位置。



>在数字没有溢出的前提下，对于正数和负数，左移一位都相当于乘以2的1次方，左移n位就相当于乘以2的n次方，右移一位相当于除2，右移n位相当于除以2的n次方。
>
><< 左移，相当于乘以2的n次方，例如：1<<6  相当于1×64=64，3<<4 相当于3×16=48
>
>\>> 右移，相当于除以2的n次方，例如：64>>3 相当于64÷8=8
>
>^ 异或，相当于求余数，例如：48^32 相当于 48%32=16

### 使用场景

大量数据的快速排序、查找、去重

#### 快速排序

对0-7内的5个元素（4，7，2，3，5）进行排序，只需要8个Bit（1个bytes）将这些空间的所有Bit位都置为0，然后将对应位置为1。最后，遍历一遍Bit区域，将该位是一的位的编号输出（2，3，4，5，7），这样就达到了排序的目的，时间复杂度O(n)。

优点：

- 运算效率高，不需要进行比较和移位；
- 占用内存少，比如N=10000000；只需占用内存为N/8=1250000Byte=1.25M

缺点：

- 所有的数据不能重复。即不可对重复的数据进行排序和查找。
- 只有当数据比较密集时才有优势



#### 快速查找

int数组中的一个元素是4字节占32位，那么除以32就知道元素的下标，对32求余数（%32）就知道它在哪一位，如果该位是1，则表示存在。



#### 去重

在3亿个整数中找出不重复的整数，限制内存不足以容纳3亿个整数。

　采用2-BitMap来解决，即为每个整数分配2bit，用不同的0、1组合来标识特殊意思，如00表示此整数没有出现过，01表示出现一次，11表示出现过多次，就可以找出重复的整数了，其需要的内存空间是正常BitMap的2倍，为：3亿*2/8/1024/1024=71.5MB。具体的过程如下：

　　扫描着3亿个整数，组BitMap，先查看BitMap中的对应位置，如果00则变成01，是01则变成11，是11则保持不变，当将3亿个整数扫描完之后也就是说整个BitMap已经组装完毕。最后查看BitMap将对应位为11的整数输出即可。



### Java中的Bitset的使用

#### BitSet

一个Bitset类创建一种特殊类型的数组来保存位值。

BitSet中数组大小会随需要增加    

BitSet定义了两个构造方法： 

​	**BitSet()**  

​	**BitSet(int size)** 

BitSet的底层实现是使用**long数组**作为内部存储结构

#### 常用方法

```java
cardinality( ) 	返回此 BitSet 中设置为true的位数   //就是BitSet中存放的有效位的个数，如果有重复运算会进行自动去重    
length()  返回此 BitSet 的"逻辑大小"： BitSet 中最高设置位的索引加 1    
size( )   返回此 BitSet 表示位值时实际使用空间的位数(即通过使用此BitSet表示位来分配给实际位数的内存)。128bit  
```

> null 参数传递给 BitSet 中的任何方法都将导致 NullPointerException



#### BitMap

  ```java
  set() 初始化一个bitset，指定大小
  clear() 清空bitset。
  flip() 反转某一指定位 设置某一指定位 获取某一位的状态。
  
  逻辑计算   and  or andNot  xor
  ```





## RoaringBitMap

>位集bitset（也称为位图）通常用作快速数据结构。不幸的是，它们会占用过多的内存。为了补偿，我们经常使用压缩的位图。
>
>RoaringBitmap是压缩的位图，其性能通常优于传统的压缩位图，例如WAH，EWAH或Concise。在某些情况下，RoaringBitmap可以快数百倍，并且它们通常提供更好的压缩效果。它们甚至可以比未压缩的位图更快。

container是RBM新创造的概念：    由3种容器构成 **ArrayContainer**/**BitmapContainer**/**RunContainer**，它们的容量均有上限，容器中包含的元素个数以其成员变量cardinality表示。
容器类型随着数据量的增多而改变：**ArrayContainer (0<=数据量<4096，2^12) -> BitmapContainer (4096<=数据量<65536，2^16) -> RunContainer （调用runOptimize方法）**



###  开源工具类

 JavaEWAH 或者 RoaringBitmap   

#### EWAHCompressedBitmap(谷歌对Bitmap的实现)    

```xml
<dependency>
        <groupId>com.googlecode.javaewah</groupId>
        <artifactId>JavaEWAH</artifactId>
        <version>1.1.7</version>
</dependency>
```



#### RoaringBitmap

```xml
<dependency>
        <groupId>org.roaringbitmap</groupId>
        <artifactId>RoaringBitmap</artifactId>
        <version>0.8.1</version>
    </dependency>
```

RoaringBitmap处理的是int类型的数据，但是在实际中使用的都是long类型的，可以使用Roaring64NavigableMap。 

 Roaring64NavigableMap  将一个long类型数据，拆分为高32位与低32位，   高32位代表索引，低32位存储到对应RoaringBitmap中，其内部是一个TreeMap类型的结构，会按照signed或者unsigned进行排序，key代表高32位，value代表对应的RoaringBitmap。



### 常用方法示例

```java
import org.roaringbitmap.RoaringBitmap;

public class Basic {

  public static void main(String[] args) {
       //向rr中添加1、2、3、1000四个数字
        RoaringBitmap rr = RoaringBitmap.bitmapOf(1,2,3,1000);
        //创建RoaringBitmap rr2
        RoaringBitmap rr2 = new RoaringBitmap();
        rr.add(1);
        rr.add(2);
        rr.add(120313111);
        //向rr2中添加10000-12000共2000个数字
        rr2.add(10000L,12000L);
        //返回第3个数字是1000，第0个数字是1，第1个数字是2，则第3个数字是1000
        rr.select(3);
        //返回value = 2 时的索引为 1。value = 1 时，索引是 0 ，value=3的索引为2
        rr.rank(2);
        //判断是否包含1000
        rr.contains(1000); // will return true
        //判断是否包含7
        rr.contains(7); // will return false
        rr2.add(1);
        rr2.add(2);
        rr2.add(3);
        rr2.add(1000);
        //两个RoaringBitmap进行or操作，数值进行合并，合并后产生新的RoaringBitmap叫rror
        RoaringBitmap rror = RoaringBitmap.and(rr, rr2);
        //rr与rr2进行位运算，并将值赋值给rr
        rr.or(rr2);
        //判断rror与rr是否相等，显然是相等的
        boolean equals = rror.equals(rr);
        //if(!equals) throw new RuntimeException("bug");
        // 查看rr中存储了多少个值，1,2,3,1000和10000-12000，共2004个数字
        long cardinality = rr.getLongCardinality();
        long carrror = rror.getLongCardinality();
        System.out.println("carror" + carrror);
        System.out.println(cardinality);
        //遍历rr中的value
        for(int i : rr) {
            System.out.println(i);
        }
        //这种方式的遍历比上面的方式更快
        rr.forEach((Consumer<? super Integer>) i -> {
            System.out.println(i.intValue());
        });
  }
}
```





### [何时用RoaringBitMap?](https://blog.csdn.net/lao000bei/article/details/105725579)

BitSet会占用大量内存。例如，如果使用BitSet并将位置1,000,000的位设置为true，则刚好超过100kB。存储一个位的位置超过100kB。,哪怕不考虑内存，这也是浪费的 。假设需要计算此BitSet和另一个在位置1,000,001处具有真值的真位之间的交集，那么需要遍历所有这些零，这就可能变得非常浪费。

在某些情况下，尝试使用压缩的位图肯定是浪费的。例如，如果范围尺寸较小。例如有一个位图表示从[0，n）开始的整数集，其中n很小（例如n = 64或n = 128）。如果能够解压缩BitSet并且不会消耗内存，那么压缩的位图可能对您没有用。实际上，如果不需要压缩，则BitSet可以提供惊人的速度。

稀疏方案是不应该使用压缩位图的另一个用例。请记住，看起来随机的数据通常不可压缩。例如，如果有一小组32位随机整数，则在数学上不可能每个整数使用少于32位，并且尝试压缩可能会适得其反。


## 参考：

[1]: https://www.pianshen.com/article/74701427722/	"Bitmap的原理和实现"
[2]: https://www.cnblogs.com/ytwang/p/13965856.html	"数据结构与算法-BitMap和RoaringBitmap"
[3]: https://blog.csdn.net/lao000bei/article/details/105725579	"RoaringBitmap精确去重"
[4]: https://cloud.tencent.com/developer/article/1753528	"一文读懂比BitMap有更好性能的Roaring Bitmap"
[5]: https://www.cnblogs.com/cjsblog/p/11613708.html	"Bitmap简介"
[6]: https://www.jianshu.com/p/818ac4e90daf	"高效压缩位图RoaringBitmap的原理与应用"
[7]: https://blog.csdn.net/yizishou/article/details/78342499	"RoaringBitmap数据结构及原理"

