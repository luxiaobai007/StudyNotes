[toc]



# Java 使用Atomic

## 介绍

Atomic由java.util.concurrent包提供的一组原子操作的封装类===>>>java.uitl.concurrent.atomic包下



## Atomic成员

- 原子方式更新基本类型
- 原子方式更新布尔类型
- 原子方式更新引用
- 原子方式更新字段



### 原子方式更新基本类型

- AtomicBoolean：原子更新布尔类型
- AtomicInteger：原子更新整型
- AtomicLong：原子更新长整型



|         方法名          |                           方法作用                           |
| :---------------------: | :----------------------------------------------------------: |
|          get()          |                          直接返回值                          |
|        set(int)         |             设置数据(注意这里是没有原子性操作的)             |
|    getAndIncrement()    |        以原子方式将当前值加1，相当于线程安全的i++操作        |
|    incrementAndGet()    |        以原子方式将当前值加1，相当于线程安全的++i操作        |
|    getAndDecrement()    |        以原子方式将当前值减1，相当于线程安全的i–操作         |
|    decrementAndGet()    |        以原子方式将当前值减1，相当于线程安全的–i操作         |
|     getAndSet(int)      |               设置指定的数据，返回设置前的数据               |
|     addAndGet(int)      |               增加指定的数据，返回增加后的数据               |
|     getAndAdd(int)      |               增加指定的数据，返回变化前的数据               |
|      lazySet(int)       |                      仅仅当get时才会set                      |
| compareAndSet(int, int) | 比较源数据和期望数据(参数一)，若一致，则设置新数据(参数二)到源数据中并返回true，否则返回false |

```java
public class Main {
    public static void main(String[] args) {
        AtomicInteger ai = new AtomicInteger(1);
        //先获取，再自增
        System.out.println(ai.getAndIncrement());
        //先自增，再获取
        System.out.println(ai.incrementAndGet());
        //增加一个指定值，先add，再get
        System.out.println(ai.addAndGet(5));
        //增加一个指定值,先get，再set
        System.out.println(ai.getAndSet(5));
    }
}
1
3
8
8
```



### 原子方式更新数组

- AtomicIntegerArray：原子更新整型数组里的元素
- AtomicLongArray：原子更新长整型数组里的元素
- AtomicReferenceArray：原子更新引用类型数组里的元素

```java
  public static void main(String[] args) {
        //数组
        int[] valueArr = new int[]{1,2};
        //AtomicIntegerArray内部会拷贝一份数组
        AtomicIntegerArray aiArr = new AtomicIntegerArray(valueArr);
        System.out.println(aiArr.getAndSet(0,3));//返回旧值
        System.out.println(aiArr.get(0));
        System.out.println(valueArr[0]);

    }
}
1
3
1

```



### 原子方式更新引用

- AtomicReference：原子更新引用类型
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段
- AtomicMarkableReference：原子更新带有标记位的引用类型

```java
public static void main(String[] args) {
        AtomicReference<User> atomicUserRef = new AtomicReference<User>();
        User luxiao = new User("路小白", 21);
        User Jake = new User("jake", 18);
        atomicUserRef.set(luxiao);
        atomicUserRef.compareAndSet(luxiao,Jake);
        System.out.println(atomicUserRef.get().getName());
        System.out.println(atomicUserRef.get().getOld());
    }
```



### 原子方式更新字段

- AtomicIntegerFieldUpdater：原子更新整型字段的更新器
- AtomicLongFieldUpdater：原子更新长整型字段的更新器
- AtomicStampedReference：原子更新带有版本号的引用类型

```java
public class ThreadAtomicInteger {

    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class,"old");
        User user = new User("luxiaobai",21);
        System.out.println(a.getAndIncrement(user));
        System.out.println(a.get(user));
    }
}

class User{
    private String name;
    public volatile int old;

    public User(String name, int old) {
        this.name = name;
        this.old = old;
    }
    
}
```





# 数据结构---- String

String类型是Java编程中最为常见的数据结构（没有之一），与之相关联的还有StringBuilder和StringBuffer。其中String类型是不可变的；后者均是可变的字符串，但是StringBuilder是线程不安全的，StringBuffer线程安全；所以三者的效率排名为：StringBuilder>String>StringBuffer。另外，为了优化字符串的使用，Java定义了两种字符串变量，一个是字符串常量，另一个就是字符串对象。



## 字符串的底层实现机制

不论是字符串常量还是字符串对象，其底层都是String类，而String类存储字符串的方式是通过char型数组存储



## String字符串常量和String对象的区别是什么？

String字符串常量和String字符串对象底层都是char型数组罢了，但是实际在内存存储还是有区别的！因为字符串是Java中用的最多的一种数据类型，所以JVM专门开辟一个常量池空间针对String类型的数据作了特殊优化：即如果是字符串常量形式的声明首先会查看常量池中是否存在这个常量，如果存在就不会创建新的对象，否则在在常量池中创建该字符串并创建引用，此后不论以此种方式创建多少个相同的字符串都是指向这一个地址的引用，而不再开辟新的地址空间放入相同的数据；但是字符串对象每次new都会在堆区形成一个新的内存区域并填充相应的字符串，不论堆区是否已经存在该字符串！


### **常量池**

一种是class文件中的静态常量池，它只是java源码编译后形成的一类数据，不仅仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间，是存在硬盘中的数据类型的命名方式；
另一种是运行时常量池，即上文中一直提到的常量池，它是内存中存储一类数据的内存区域命名方式，那么常见的就是字符串常量池，专门为优化字符创常量而分配的一个空间。在JDK6时，常量池存在方法区的永久代中；JDK7、JDK8都把常量池转移到堆区了，而且到了JDK8的时候已经不存在永久代了，取而代之的是元空间的概念，但是元空间并不占用JVM内存而是直接共享系统内存，他本质上也是方法区的一种实现方式罢了

### 常见的字符串比较相等

#### 场景1

```java
String a = new String("hellojava");  //堆区
String b = "hellojava";								//常量池	
  
  a!=B
```

一个在堆区，一个在常量池，他们是不同的两个对象！



#### 场景2

```java
String b = "hellojava";
String c = "hello" + "java";
b = c
```

JVM在编译期间会自动把字符串常量相加操作计算完毕后再执行比较，所以计算完成后其实c=“helloJava”，因为字符串存在这样的常量，所以直接引用即可，使用的是同一内存地址的数据



#### 场景3

```java
String b = "hellojava";
String d1 = "hello";
String d2 = "java";
String d = d1 + d2;
String e = d1 + new String("java");
String f = "hello" + new String("java");
b！=d != e !=f
```

字符串变量的连接动作，在编译阶段会被转化成StringBuilder的append操作，变量d最终指向Java堆上新建String对象，变量b指向常量池的”helloJava”字符串，所以 b != d,也就是文章开头讲String不可变的机制,为什么JVM要这样处理？

这是因为d1和d2在编译阶段最终指向的对象是不可知的，所以不能当做常量来看待。但是如果添加final关键字的话，那么就是相等的了，因为final关键字强制性的使d1和d2无法再变更，所以可以当做常量看待！其他不等的情况同理

---

#### 场景4

```java
String a= new String("hellojava");
String a1 = a.intern();
System.out.println(a == a1);//false

String b = new String("String") + new String("Tests");
String b1 = b.intern();
System.out.println(b == b1);//true

String c = "helloWorld";
String c1 = c.intern();
System.out.println(c == c1);//true
```

1、JDK7/8的常量池区域都放在了堆区（JDK6的情况就不考虑了）
2、intern()方法的作用在于：首先查看常量池是否有字符串，如果有直接返回字符串的引用；否则在常量池添加该字符串在堆区的引用，并返回该引用；

对于第一个例子，执行第一句后，a指向是堆区字符串的引用，同时在常量池也创建了一份字符串，所以a1指向常量池字符串引用，所以引用地址不同自然不等；
对于第二个例子，只在堆区创建了字符串对象，常量池没有！所以执行intern()后实际在常量池中创建了一份该字符串在堆区的引用，b1实际指向的还是堆区的引用，所以相等！
对于第三个例子，通过前面的分析已经很简单了，直接返回常量池的引用就行了，所以是相等的！
这里对于第二个例子稍微做一下变形

```java
String b = new String("String") + new String("Tests");
b0 = "StringTests";//添加此句
String b1 = b.intern();
System.out.println(b == b1);//false
```

由上述分析可知：如果常量池存在该字符串，则直接返回常量池的引用，这里在intern()之前在常量池中创建了该字符串，所以最终返回的不是堆区的引用，而是字符串的引用！



# HashMap 与 ConcurrentMap的实现原理

## HashMap介绍

HashMap 是Map接口的实现,HashMap运行空的key-value键值对,HashMap被认为是Hashtable的增强版,HashMap 是一个非线程安全的容器,如果想构造线程安全的Map考虑使用ConcurrentHashMap.HashMap是无序的,因为HashMap无法保证内部存储的键值对的有序性.

HashMap的底层数据结构是**数组+链表**的集合体,数组在HashMap中又被称为桶(bucket).遍历HashMap需要的时间损耗为HashMap实例桶的数量+(key-value映射)的数量.如果遍历元素很重要的话,不要把初始容量设置的太高或者负载因子设置的太低.

### 初始容量

hash表桶的数量,

### 负载因子

是一种衡量哈希表填充程度的标准

当哈希表中存在足够数量的entry,以至于超过了负载因子和当前容量,这个哈希表会进行rehash操作,内部的数据结构重新rebuild.



## HashMap与HashTable的区别

### 相同点

都是机遇哈希表实现的,其内部每个元素都是 `key-value` 键值对，HashMap 和 HashTable 都实现了 Map、Cloneable、Serializable 接口。



### 不同点

- 父类不同：HashMap 继承了 `AbstractMap` 类，而 HashTable 继承了 `Dictionary` 类
- 空值不同:  HashMap 允许空的 key 和 value 值，HashTable 不允许空的 key 和 value 值。HashMap 会把 Null key 当做普通的 key 对待。不允许 null key 重复。
- 线程安全性：HashMap 不是线程安全的，如果多个外部操作同时修改 HashMap 的数据结构比如 add 或者是 delete，必须进行同步操作，仅仅对 key 或者 value 的修改不是改变数据结构的操作。可以选择构造线程安全的 Map 比如 `Collections.synchronizedMap` 或者是 `ConcurrentHashMap`。而 HashTable 本身就是线程安全的容器。
- 性能方面：虽然 HashMap 和 HashTable 都是基于单链表的，但是 HashMap 进行 put 或者 get操作，可以达到常数时间的性能；而 HashTable 的 put 和 get 操作都是加了 `synchronized` 锁的，所以效率很差。
- 初始容量不同：HashTable 的初始长度是11，之后每次扩充容量变为之前的 2n+1（n为上一次的长度）;HashMap 的初始长度为16，之后每次扩充变为原来的两倍。创建时，如果给定了容量初始值，那么HashTable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。



### 说说HashMap 底层数据结构是怎样的？

HashMap 底层是 hash 数组和单向链表实现，jdk8后采用数组+链表+红黑树的数据结构



### 说说HashMap 的工作原理

HashMap 底层是 hash 数组和单向链表实现，JDK8后采用数组+链表+红黑树的数据结构。

我们通过put和get存储和获取对象。当我们给put()方法传递键和值时，先对键做一个hashCode()的计算来得到它在bucket数组中的位置来存储Entry对象。当获取对象时，通过get获取到bucket的位置，再通过键对象的equals()方法找到正确的键值对，然后在返回值对象。



### 使用HashMap时，当两个对象的 hashCode 相同怎么办？

因为HashCode 相同，不一定就是相等的（equals方法比较），所以两个对象所在数组的下标相同，"碰撞"就此发生。又因为 HashMap 使用链表存储对象，这个 Node 会存储到链表中。



### HashMap 的哈希函数怎么设计的吗？

hash 函数是先拿到通过 key 的 hashCode ，是 32 位的 int 值，然后让 hashCode 的高 16 位和低 16 位进行异或操作。两个好处：

1. 一定要尽可能降低 hash 碰撞，越分散越好；
2. 算法一定要尽可能高效，因为这是高频操作, 因此采用位运算



### HashMap遍历方法有几种？

- Iterator 迭代器
- 最常见的使用方式，可同时得到 key、value 值
- 使用 foreach 方式（JDK1.8 才有）
- 通过 key 的 set 集合遍历



### 为什么采用 hashcode 的高 16 位和低 16 位异或能降低 hash 碰撞？

因为 key.hashCode()函数调用的是 key 键值类型自带的哈希函数，返回 int 型散列值。int 值范围为**-2147483648~2147483647**，前后加起来大概 40 亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。

设想，如果 HashMap 数组的初始大小才 16，用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。



### 解决hash冲突的有几种方法？

1、**再哈希法**：如果hash出的index已经有值，就再hash，不行继续hash，直至找到空的index位置，要相信瞎猫总能碰上死耗子。这个办法最容易想到。但有2个缺点：

- 比较浪费空间，消耗效率。根本原因还是数组的长度是固定不变的，不断hash找出空的index，可能越界，这时就要创建新数组，而老数组的数据也需要迁移。随着数组越来越大，消耗不可小觑。
- get不到，或者说get算法复杂。进是进去了，想出来就没那么容易了。

2、**开放地址方法**：如果hash出的index已经有值，通过算法在它前面或后面的若干位置寻找空位，这个和再hash算法差别不大。

3、**建立公共溢出区：** 把冲突的hash值放到另外一块溢出区。

4、**链式地址法：** 把产生hash冲突的hash值以链表形式存储在index位置上。HashMap用的就是该方法。优点是不需要另外开辟新空间，也不会丢失数据，寻址也比较简单。但是随着hash链越来越长，寻址也是更加耗时。好的hash算法就是要让链尽量短，最好一个index上只有一个值。也就是尽可能地保证散列地址分布均匀，同时要计算简单。



### 为什么要用异或运算符？

保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。



### HashMap 的 table 的容量如何确定

1. table 数组大小是由 capacity 这个参数确定的，默认是16，也可以构造时传入，最大限制是1<<30；
2. loadFactor 是装载因子，主要目的是用来确认table 数组是否需要动态扩展，默认值是0.75，比如table 数组大小为 16，装载因子为 0.75 时，threshold 就是12，当 table 的实际大小超过 12 时，table就需要动态扩容；
3. 扩容时，调用 resize() 方法，将 table 长度变为原来的两倍（注意是 table 长度，而不是 threshold）；
4. 如果数据很大的情况下，扩展时将会带来性能的损失，在性能要求很高的地方，这种损失很可能很致命。



### HashMap中put方法的过程

![image-20210827172226915](/Users/lushengyang/Desktop/LSY/document/image/image-20210827172226915.png)



1. 判断键值对数组table[i]是否为空或为null,否则执行resize()扩容
2. 根据键值key计算hash值得到插入数组索引i,如果table[i]==null,直接新建结点添加,转向`6`,如果table[i]!=null,转向`3`
3. 判断table[i]的首个元素是否和key一样,如果相同直接覆盖value,否则转向`4`,这里的相同指的是hashcode以及equals;
4. 判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向`5`；
5. 遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
6. 插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。



#### Resize()扩容机制

重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。

```java
public void resize(int newCapacity){ //传入新的容量
        Entry[] oldTable = table;
        int odlCapacity = oldTable.length;
        if(odlCapacity == MAXIMUM_CAPACITY){  ///扩容前的数组大小如果已经达到最大 (2^30)了
            threshold = Integer.MAX_VALUE;  //修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
            return;
        }
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable);   //将数据转移到新的Entry数组里
        table = newTable;    //HashMap的table属性引用新的 Entry数组
        threshold = (int)(newCapacity * loadFactory);//修改阈值


    }
//使用一个容量更大的数组来代替已有的容量小的数组，transfer()方法将原有Entry数组的元素拷贝到新的Entry数组里。
    void transfer(Entry[] newTable){
        Entry[] src = table; //src引用了旧的Entry数组
        int newCapacity = newTable.length;
        for (int i = 0; i < src.length; i++) { //遍历旧的Entry数组
                Entry<K,V> e = src[i];      ///取得旧Entry数组的每个元素
                if(e != null){
                    src[i] = null;//释放旧Entry数组的对象引用（for循环后，旧的Entry数组不再引用任何对象）
                    do{
                        Entry<K,V> next = e.next;
                        int j = indexFor(e.hash,newCapacity);//重新计算每个元素在数 组中的位置
                        e.next = newTable[j];//标记【1】
                        newTable[j] = e;  //将元素放在数组上
                        e = next;//访问下一个Entry链上的元素
                    }while (e != null);
                }
        }
    }
```







### 当链表长度 >= 8时，为什么要将链表转换成红黑树？

因为红黑树的平均查找长度是log(n)，长度为8的时候，平均查找长度为3，如果继续使用链表，平均查找长度为8/2=4，所以，当链表长度 >= 8时 ，有必要将链表转换成红黑树



### 说说hashMap中get是如何实现的？

对key的hashCode进行hash值计算，与运算计算下标获取bucket位置，如果在桶的首位上就可以找到就直接返回，否则在树中找或者链表中遍历找，如果有hash冲突，则利用equals方法去遍历链表查找节点。





### 拉链法导致的链表过深问题为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。



### JDK8中对HashMap做了哪些改变？

1.在java 1.8中，如果链表的长度超过了8，那么链表将转换为红黑树。（桶的数量必须大于64，小于64的时候只会扩容）

2.发生hash碰撞时，java 1.7 会在链表的头部插入，而java 1.8会在链表的尾部插入

3.在java 1.8中，Entry被Node替代(换了一个马甲)。



### HashMap，LinkedHashMap，TreeMap 有什么区别？

- LinkedHashMap是继承于HashMap，是基于HashMap和双向链表来实现的。
- HashMap无序；LinkedHashMap有序，可分为插入顺序和访问顺序两种。如果是访问顺序，那put和get操作已存在的Entry时，都会把Entry移动到双向链表的表尾(其实是先删除再插入)。
- LinkedHashMap存取数据，还是跟HashMap一样使用的Entry[]的方式，双向链表只是为了保证顺序。
- LinkedHashMap是线程不安全的。



### HashMap 和 HashTable 有什么区别？

①、HashMap 是线程不安全的，HashTable 是线程安全的；

②、由于线程安全，所以 HashTable 的效率比不上 HashMap；

③、HashMap最多只允许一条记录的键为null，允许多条记录的值为null，而 HashTable不允许；

④、HashMap 默认初始化数组的大小为16，HashTable 为 11，前者扩容时，扩大两倍，后者扩大两倍+1；

⑤、HashMap 需要重新计算 hash 值，而 HashTable 直接使用对象的 hashCode；











# java的平台无关性

Java语言可以一次编译,到处运行,由于Java语言为解释性语言,编译器会把Java代码变成中间代码(二进制文件),然后在JVM上进行解释执行 从而可以很好的进行跨平台执行,具有很好的可移植性



# &与&&的区别是什么

&是按位于操作符,a&b是把a和b都转换成二进制数后,然后在进行按位于的运算.

&&是逻辑与操作符,a&&b就是当且仅当两个操作数均为true时,其结果才为true,只要有一个为false,a&&b的结果就为false. &&还具有短路的功能,只有当第一个表达式的返回值为true时,才会去计算第二个表达式的值,如果第一个表达式的返回值为false,则此时&&运算的结果就为false,同时,不会去计算第二个表达式的值



## HashSet

`HashSet` 是一个不允许存储重复元素的集合，它的实现比较简单

#### 成员变量

```java
private transient HashMap<E,Object> map;   //用于存放最终数据的
private static final Object PRESENT = new Object();  //是所有写入map的value值
```

#### 构造函数

```java
public HashSet() {
        map = new HashMap<>();
    }
    
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }    
```

构造函数很简单，利用了 `HashMap` 初始化了 `map` 。



#### add

```java
public boolean add(E e){
  return map.put(e, PRESENT) == null;
}
```

它是将存放的对象当做了 `HashMap` 的健，`value` 都是相同的 `PRESENT` 。由于 `HashMap` 的 `key` 是不能重复的，所以每当有重复的值写入到 `HashSet` 时，`value` 会被覆盖，但 `key` 不会受到影响，这样就保证了 `HashSet` 中只能存放不重复的元素。











































































































































#### 相关资料来源

[1]: https://biscuitos.github.io/blog/ATOMIC/	" atomic的原理及Linux下的原子操作使用说明"
[2]: https://blog.csdn.net/qq_30379689/article/details/80785650	"java 的原子特性"

[3]: https://blog.csdn.net/u011552404/article/details/79980796	"数据结构-- String"

