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



# HashMap 与 ConcurrentHashMap的实现原理

## HashMap介绍

1. HashMap是一种散列表,以Key/Value形式存储,Key和Value都可以为空,在JDK1.7时是由数组+链表组成,J DK1.8则由数组+链表+红黑树组成.

2. 在JDK1.8 版本的HashMap,初始默认的容量是16,负载因子是0.75,当链表中的元素大于8且数组容量大于64时链表进行红黑树化,当红黑树元素减少到6时,红黑树会退化为链表.

3. 数组的查询效率是O(1),链表的查询效率是O(N),红黑树的查询效率是O(logN)

4. HashMap的容量必须是2的次幂,在构造方法中传入的容量如果不是2的次幂,难免HashMap会调用tableSizeFor()方法来获取一个最接近传入容量且大于传入容量的2的次幂的值,比如传入的容量是17则tableSizeFor()会返回32

5. HashMap空参数的构造方法创建出来的HashMap对象是不会初始化空间的.当使用空参数构造方法创建出对象后HashMap将在第一次插入元素时进行空间的初始化.

   ```java
   //空参数构造方法
   public HashMap(){
   	this.loadFactor = DEFAULT_LOAD_FACTOR;
   }
   ```

   

6. HashMap主要由2个重要方法,put/get,put()方法就是将元素无序的加入到HashMap中,也就是无法保证元素的插入顺序,get方法就是获取HashMap中的元素.

   1. 当元素进行put时,首先要计算key的Hash值,为了更加分散,获取hash值后,HashMap会让hash值的高16位与hash进行异或运算

   ```java
   return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   ```

   2. 然后判断HashMap有没有进行初始化,如果没有则进行初始化,再找到Key对应数组的位置,如果该位置没有元素则直接插入,如果有元素则判断是不是Key与所在元素Key是不是一致(**先判断hash值是不是相等,如果相等再进行equals()判断**),一致则新值替换旧值,如果不一致继续遍历这个节点下的链表或红黑树.
   3. 如果遍历过程中有元素的key与所在元素的key是一支的新值替换旧值,否则将该元素加入到链表或红黑树中.
   4. Get()方法首先计算key的hash值,然后找到hash对应数组中的位置的第一个元素,判断key是否一致,如果一致则返回否则继续查找这个元素下的链表或红黑树

7. HashMap的扩容会调用resize()方法,扩容阈值位当前容量*负载因子,默认情况初始容量16,负载因子0.75,则元素个数大于12就会触发扩容,resize这个方法会先判断HashMap是不是初始化,扩容的时候会创建一个新的数组,然后旧数组中的元素进行搬移,JDK1.7的HashMap在并发场景下扩容有可能造成死循环,CPU飙升100%.

   

### 为啥链表转红黑树的阈值是8(负载因子为0.75)

在HashMap源码中有介绍: 在使用分布良好的哈希码时,树是很少被使用的.理想情况下,随机哈希码遵循泊松分布,一个链表上有8个节点的概率为0.00000006这个值要小于千万分之一.当这个阈值为8已经很小了,所以这就是阈值选8的原因



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

![image-20210827172226915](/Users/lushengyang/Desktop/LSY/StudeyNotes/image/image-20210827172226915.png)



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



## HashMap为什么引入红黑树而不是AVL树(有了二叉查找树、平衡树(AVL)为啥还需要红黑树)

二叉查找树,也称有序二叉树,或已排序二叉树,是指一棵空树或者具有以下性质的二叉树:

- 若任意即诶单的左子树不为空,则左子树上所有结点的值均小于它的根节点的值
- 若任意节点的右子树不空,则右子树上所有结点的值均大于它的根节点的值
- 任意节点的左、右子树也分别为二叉查找树
- 没有键值相等的节点

> 因为一棵由N个节点随机构造的二叉查找树的高度为logN,所以顺理成章,二叉查找树的一般操作的执行时间为O(logN).但二叉查找树若退化成了一棵具有N个节点的线性链后,则这些操作最坏情况运行时间为O(N)
>
> 为了解决这个问题,于是引出了平衡二叉树,即AVL树,
>
> 它对二叉查找树做了改进,在我们每插入一个节点的时候,必须保证每个节点对应的左子树和右子树的树高度差不超过1.如果超过了就对其进行左旋或右旋,使之平衡



平衡树解决了二叉树查找树退化为近似链表的缺点,能够把查找时间控制在O(logN),不过却不是最佳的,因为平衡树要求==每个节点的左子树和右子树的高度差至多等于1==这个要求太严了,导致每次进行插入/删除节点的时候,几乎都会破坏平衡树的规则,都需要通过左旋和右旋来进行调整,使之再次成为一颗符合要求的平衡树.

如果在那种插入、删除很频繁的场景中,平衡树需要频繁进行调整,这会使平衡树的性能大打折扣,为了解决这个问题,就有了红黑树.

==红黑树是不符合AVL树的平衡条件的,即每个节点的左子树和右子树的高度最多差1的二叉查找树,但是提出了为节点增加颜色,红黑树是用非严格的平衡来换取增删节点时候旋转次数的降低,任何不平衡都会在三次旋转之内解决,相较于AVL树为了维持平衡的开销要小得多.==



AVL树是严格平衡树,因此在增加或者删除节点的时候,根据不同情况,旋转的次数比红黑树要多,==所以红黑树的插入效率相对于AVL树更高.单单在查找方面比较效率的话,由于AVL高度平衡,因此AVL树的查找效率比红黑树更高==.



#### 总结

针对于几种平衡树做个比较,发现红黑树有着良好的稳定性和完整的功能,性能表现也很不错,综合实力强.实际应用中,若搜索的效率远远大于插入和删除,那么选择AVL,如果发生搜索和插入删除操作的效率差不多,应该选择红黑树



## LinkedHashMap概述









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







# 二维码扫描登录的原理

## 移动端基于token的认证机制

#### 基于token的认证机制流程图

![image-20210909224407818](http://qiliu.luxiaobai.cn/img/image-20210909224407818.png)





## 二维码扫码登录的原理

#### 二维码扫码登录流程图

![image-20210909231553806](/Users/lushengyang/Desktop/LSY/StudeyNotes/interview/Java.assets/image-20210909231553806.png)

> 扫码登录可以分为三个阶段:**待扫码、已扫码待确认、已确认.**

#### 1、待扫码阶段

- PC端携带设备信息向服务端发起生成二维码请求
- 服务端生成唯一的二维码ID,(UUID),将二维码ID跟设备信息关联起来.
- PC 端接受到二维码 ID 之后，将二维码 ID 以二维码的形式展示，等待移动端扫码
- 在 PC 端会启动一个定时器，轮询查询二维码的状态。**如果移动端未扫描的话，那么一段时间后二维码将会失效。**

#### 2、已扫码待确认阶段

- 移动端扫描二维码，获取二维码 ID，**然后将手机端登录的信息凭证（token）和 二维码 ID 作为参数发送给服务端**，此时的手机一定是登录的，不存在没登录的情况。

- 服务端接受请求后，会将 token 与二维码 ID 关联

- 使用微信时，移动端退出， PC 端是不是也需要退出，这个关联就有点把子作用了。然后会**生成一个一次性 token，这个 token 会返回给移动端，一次性 token 用作确认时候的凭证**。

- PC 端的定时器，会轮询到二维码的状态已经发生变化，会将 PC 端的二维码更新为已扫描，请确认。

  

#### 3、已确认

- 移动端携带上一步骤中获取的临时 token ，确认登录，**服务端校对完成后，会更新二维码状态，并且给 PC 端生成一个正式的 token ，后续 PC 端就是持有这个 token 访问服务端**。
- PC 端的定时器，轮询到了二维码状态为登录状态，并且会获取到了生成的 token ，完成登录，后续访问都基于 token 完成。
- 在服务器端会给手机端一样,维护着token跟二维码、PC设备信息、账号等信息.









# 抽象类与接口

## 抽象类

抽象类不能创建实例，它只能作为父类被继承。抽象类是从多个具体类中抽象出来的父类，它具有更高层次的抽象。从多个具有相同特征的类中抽象出一个抽象类，以这个抽象类作为其子类的模板，从而避免了子类的随意性。

### 特点

1. 抽象类无法被实例化（因为它不是具体的类，但是有构造方法）
2. 抽象类有构造方法，是给子类创建对象的
3. 抽象类中可以定义抽象方法（在方法的修饰列表中添加abstract关键字，并且以“;”结束，不能带有“{}”）`public abstract void m1();`
4. 抽象类中不一定有抽象方法，抽象方法一定在抽象类中
5. 一个非抽象类继承抽象类，必须将抽象类中的抽象方法覆盖，实现，重写

#### 抽象类的成员特点

1）成员变量：既可以是变量也可以是常量。

2）构造方法：有构造方法，用于子类访问父类数据的初始化。

3）成员方法：抽象类中方法既可以是抽象的，也可以是非抽象方法



在父类中，非抽象方法：子类继承，提高代码的复用性；抽象方法：强制要求子类做的事情

抽象类中注意的问题：一个类如果没有抽象方法，可以是抽象类，即抽象类中可以完全没有抽象方法。这样类的主要目的就是不让创建该类对象。



#### abstract关键字不可以与哪些关键字使用。

   1）private冲突：private修饰的成员不能被继承，从而不可以被子类重写，而abstract修饰的是要求被重写的。

   2）final冲突：final修饰的成员是最终成员，不能被重写，所以冲突，static无意义；

   3）static冲突；static修饰成员用类名可以直接访问，但是abstract修饰成员没有方法体，所以访问没有方法体的成员无意义。





## 接口

接口的初步理解是一个特殊的抽象类，当抽象类中全部都是抽象方法时，可以通过接口的方式来体现。

### 特点

1. 接口不能被实例化
2. 接口只能包含方法的声明
3. 接口的成员方法包括方法，属性，索引器，事件
4. 接口中不能包含常量，字段(域)，构造函数，析构函数，静态成员





## 抽象类与接口区别

1. 抽象类可以有构造方法，接口中不能有构造方法。
2. 抽象类中可以有普通成员变量，接口中没有普通成员变量
3. 抽象类中可以包含静态方法，接口中不能包含静态方法
4. 一个类可以实现多个接口，但只能继承一个抽象类
5. 接口可以被多重实现，抽象类只能被单一继承
6. 如果抽象类实现接口，则可以把接口中方法映射到抽象类中作为抽象方法而不必实现，而在抽象类的子类中实现接口中方法。



## 抽象类与接口相同点

1. 都可以被继承
2. 都不能被实例化
3. 都可以包含方法声明
4. 派生类必须实现未实现的方法



接口带来的最大好处就是避免了多继承带来的复杂性和低效性，并且同时可以提供多重继承的好处。接口和抽象类都可以提现多态性，但是抽象类对事物进行抽象，更多的是为了继承，为了扩展，为了实现代码的重用，子类和父类之间提现的是is-a关系，接口则更多的体现一种行为约束，一种规则，一旦实现了这个接口，就要给出这个接口中所以方法的具体实现，也就是实现类对于接口中所有的方法都是有意义是的。



## 接口和抽象类的应用场景

- 如果拥有一些方法并且想让它们中的有一些默认实习,那么使用抽象类
- 如果想使用多重继承,那么必须使用接口.由于Java不支持多继承,即一个类只能有一个超类.但是可以实现多个接口,因此可以使用接口来解决
- 如果基本功能在不断改变,那么就需要使用抽象类,达到解耦目的.如果不断改变基本功能并且使用接口,那么就需要改变所有实现了该接口的类.



> 接口更多的是在系统架构设计方面发挥作用,主要用于定义模块之间的通信契约.
>
> 抽象类在代码实现方面发挥作用,可以实现代码的重用





# Spring Boot的事务传播机制

多个事务方法相互调用时,事务如何在这些方法间传播

> 方法A是一个事务的方法,方法A执行过程中调用了方法B,那么方法B有无事务以及对事务的要求不同都会对方法A的事务具体执行造成影响,同时方法A的事务对方法B的事务执行也有影响,这种影响具体是什么就由两个方法所定义的事务传播类型决定

- REQUIRED (**required**)(Spring默认的事务传播类型): 如果当前没有事务,则自己新建一个事务,如果当前存在事务,则加入这个事务
- SUPPORTS(supports): 当前存在事务,则加入当前事务,如果当前没有事务,就以非事务方法执行
- MANDATORY(mandatory): 当前存在事务,则加入当前事务;如果当前事务不存在,则抛出异常
- REQUIRES_NEW(requires_new): 创建一个新事务,如果存在当前事务,则挂起当前事务
- NEVER(never): 不使用事务,如果当前事务存在,则抛出异常
- NESTED(nested): 如果当前事务存在,则在嵌套事务中执行,否则和REQUIRED的操作一样(开启一个事务)

> ==**NESTED 和REQUIRES_NEW的区别**==
>
> REQUIRES_NEW是新建一个事务并且新开启的这个事务与原有事务无关,而NESTED则是当前存在事务时(我们把当前事务称之为父事务)会开启一个嵌套事务(称之为一个子事务).在NESTED情况下父事务回滚时,子事务也会回滚,而在REQUIRES_NEW情况下,原有事务回滚,不会影响新开启的事务
>
> 
>
> ==**NESTED和REQUIRED的区别**==
>
> REQUIRED情况下,调用方存在事务时,则被调用方和调用方使用同一事务,那么被调用方出现异常时,由于共用一个事务,所以无论调用方是否catch其异常,事务都会回滚,而在NESTED情况下,被调用方发生异常时,调用方可以catch其异常,这样只有子事务回滚,父事务不受影响





## Spring事务什么时候会失效

Spring事务的原理是AOP,进行了切面增强,那么失效的根本原因是这个AOP不起作用了!常见的情况有如下几种:

1. 发生自调用,类里面使用this调用本类的方法(this通常省略),此时这个this对象不是代理类,而是`UserService`对象本身!  解决方式: 让那个this变成UserService的代理类即可!

2. 方法不是public的

   > @Transactional只能用于public的方法上,否则事务不会失效,如果要用在非public方法上,可以开启AspectJ代理模式

3. 数据库不支持事务

4. 没有被Spring管理

5. 异常被吃掉,事务不会回滚(或者抛出的异常没有被定义,默认为RuntimeException)





# Mysql主键设置为自增的优缺点

## 自增主键

> 自增ID是在设计表时将ID字段的值设置为自增的形式,这样当插入一行数据时无需指定ID会自动根据前一字段的ID值+1进行填充.通过SQL语句`AUTO_INCREMENT`来对待定的字段启用自增赋值,使用自增ID作为主键,能够保证字段的原子性



### 优点

1. 数据库自动编号,速度快,而且是增量增长,按顺序存放,对于检索非常有利
2. 数字型,占用空间小,易排序,在程序中传递也方便
3. 如果通过非系统增加记录时,可以不用指定该字段,不用担心主键重复问题



### 缺点

- 因为自动增长,在手动要插入指定ID的记录时会显得麻烦,尤其是当系统与其他系统集成时,需要数据导入时,很难保证原系统的ID*不发生主键冲突*



## UUID

### 优点

- 出现数据拆分、合并存储时,能达到全局的唯一性



### 缺点

- 影响插入速度,并且造成硬盘使用率低
- uuid之间比较大小相对数字慢不少,影响查询速度
- uuid占空间大,如果你建的索引越多,影响越严重





# 索引优化

![index](http://qiliu.luxiaobai.cn/img/index.png)

## 索引优化规则

1. like语句的前导模糊查询不能使用索引

2. union、in、or都能够命中索引,建议使用in

   - union能够命中索引,并且mysql耗费的CPU最少

     ```sql
     select * from doc where status=1
     union all
     select * from doc where status=2;
     ```

     

   - in能够命中索引,查询优化耗费的CPU比union all多,但可以忽略不计,一般情况下建议使用in

     ```sql
     select * from doc where status in(1,2)
     ```

     

   - or新版的MYSQL能够命中索引,查询优化耗费的CPU比in多,不建议耗费用or

     ```sql
     select * from doc where status=1 or status=2
     ```

     > 补充: 有些地方说在where条件中使用or,索引会失效,造成全表扫描,这是个误区:
     >
     > 1. 要求where子句使用的所有字段,都必须建立索引
     > 2. 如果数据量太少,mysql制定执行计划时发现全表扫描比索引查找更快,所以会不使用索引
     > 3. 确保mysql版本5.0以上,且查询优化器开启了index_merge_union=on,也就是变量optimizer_switch里存在index_merge_union且为on

3. 负向条件查询不能使用索引

   - 负向条件有: !=、<>、not in、not exists、not like等

   ```sql
   ###例如
   select * from doc where status!=1 and status !=2;
   ##可以优化为in查询:
   select * from doc where status in(0,3,4);
   ```

4. 联合索引最左前缀原则

   - 如果在(a,b,c)三个字段上建立联合索引,那么他会自动建立a|(a,b)|(a,b,c)组索引

   - 登录业务需求,SQL语句如下

     ```sql
     select uid, login_time from user where login_name=? and passwd=?
     ```

   - 可以建立(login_name,passwd)的联合索引.因为业务上几乎没有passwd的单条件查询需求,而有很多login_name的单条件查询需求,所以可以建立(login_name,passwd)的联合索引,而不是(passwd,login_name)

5. 不能使用索引中范围条件右边的列(范围列可以用到索引),范围列之后列的索引全失效

   - 范围条件有: <、<=、>、>=、between等

   - 索引最多用于一个范围列,如果查询条件中有两个范围列则无法全用到索引.

   - 假如有联合索引(empno、title、from date),那么下面的SQL中emp_no可以用到索引,而title和from_date则使用不到索引

     ```sql
     select * from employees.title where emp_no < 10010 'and title=' Senio Enginner 'and from_date between' 1985-01-01 'and' 1986-12-31'
     ```

6. 不要在索引列上面做任何操作(计算、函数),否则会导致索引失效而转向全表扫描

7. 强制类型转换会全表扫描

   - 字符串类型不加单引号会导致索引失效，因为mysql会自己做类型转换,相当于在索引列上进行了操作。如果 phone 字段是 varchar 类型，则下面的 SQL 不能命中索引

     ```sql
     select * from user where phone=138000001234
     #可以优化为:
     select * from user where phone='13800001234'
     ```

8. 更新十分频繁、数据区分度不高的列不宜建立索引

   - 更新会变更 B+ 树，更新频繁的字段建立索引会大大降低数据库性能。
   - “性别”这种区分度不大的属性，建立索引是没有什么意义的，不能有效过滤数据，性能与全表扫描类似。
   - 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算。

9. **利用覆盖索引来进行查询操作，避免回表，减少select \* 的使用**

   - 覆盖索引：查询的列和所建立的索引的列个数相同，字段相同。

   - 被查询的列，数据能从索引中取得，而不用通过行定位符 row-locator 再到 row 上获取，即“被查询列要被所建的索引覆盖”，这能够加速查询速度。

   - 例如登录业务需求，SQL语句如下。

     ```sql
     Select uid, login_time from user where login_name=? and passwd=? 
     ```

10. **索引不会包含有NULL值的列 **在数据库设计时，尽量使用not null 约束以及默认值。

11. **is null, is not null无法使用索引**

12. 如果有order by、group by的场景，请注意利用索引的有序性

    order by 最后的字段是组合索引的一部分，并且放在索引组合顺序的最后，避免出现file_sort 的情况，影响查询性能。

    - 例如对于语句 where a=? and b=? order by c，可以建立联合索引(a,b,c)。

    如果索引中有范围查找，那么索引有序性无法利用，如 WHERE a>10 ORDER BY b;，索引(a,b)无法排序

13. **使用短索引(前缀索引)**

    - 对列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR(255)的列，如果该列在前10个或20个字符内，可以做到既使得前缀索引的区分度接近全列索引，那么就不要对整个列进行索引。因为短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作，减少索引文件的维护开销。可以使用count(distinct leftIndex(列名, 索引长度))/count(*) 来计算前缀索引的区分度。
    - 但缺点是不能用于 ORDER BY 和 GROUP BY 操作，也不能用于覆盖索引。
    - 不过很多时候没必要对全字段建立索引，根据实际文本区分度决定索引长度即可。

14. **利用延迟关联或者子查询优化超多分页场景**

15. **如果明确知道只有一条结果返回，limit 1 能够提高效率 ** 自己明确知道只有一条结果，但数据库并不知道，明确告诉它，让它主动停止游标移动。

    ```sql
    select * from user where login_name=?;
    ##可以优化为
    select * from user where login_name=? limit 1;
    ```

16. **超过三个表最好不要 join**

    - 需要 join 的字段，数据类型必须一致，多表关联查询时，保证被关联的字段需要有索引。
    - 例如：left join是由左边决定的，左边的数据一定都有，所以右边是我们的关键点，建立索引要建右边的。当然如果索引在左边，可以用right join

17. **单表索引建议控制在5个以内**

18. **SQL 性能优化 explain 中的 type：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好**

    - consts：单表中最多只有一个匹配行(主键或者唯一索引)，在优化阶段即可读取到数据。
    - ref：使用普通的索引(Normal Index)。
    - range：对索引进行范围检索。
    - 当 type=index 时，索引物理文件全扫，速度非常慢。

19. **业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引**

    不要以为唯一索引影响了 insert 速度，这个速度损耗可以忽略，但提高查找速度是明显的。另外，即使在应用层做了非常完善的校验控制，只要没有唯一索引，根据墨菲定律，必然有脏数据产生。

20. **创建索引时避免以下错误观念**

    >索引越多越好，认为需要一个查询就建一个索引。
    >
    >宁缺勿滥，认为索引会消耗空间、严重拖慢更新和新增速度。
    >
    >抵制惟一索引，认为业务的惟一性一律需要在应用层通过“先查后插”方式解决。
    >
    >过早优化，在不了解系统的情况下就开始优化。

    

    **索引选择性与前缀索引**

    - 既然索引可以加快查询速度，那么是不是只要是查询语句需要，就建上索引?答案是否定的。因为索引虽然加快了查询速度，但索引也是有代价的：索引文件本身要消耗存储空间，同时索引会加重插入、删除和修改记录时的负担，另外，MySQL在运行时也要消耗资源维护索引，因此索引并不是越多越好。一般两种情况下不建议建索引。
    - 第一种情况是表记录比较少，例如一两千条甚至只有几百条记录的表，没必要建索引，让查询做全表扫描就好了。至于多少条记录才算多，这个个人有个人的看法，我个人的经验是以2000作为分界线，记录数不超过 2000可以考虑不建索引，超过2000条可以酌情考虑索引。
    - 另一种不建议建索引的情况是索引的选择性较低。所谓索引的选择性(Selectivity)，是指不重复的索引值(也叫基数，Cardinality)与表记录数(#T)的比值：

    



# 行锁和表锁



> 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低
>
> 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
>
> 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。
>

## 表锁

- 不会出现死锁，发生锁冲突几率高，并发低。
- MyISAM在执行查询语句（select）前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。
- MySQL的表级锁有两种模式：表共享读锁和表独占写锁。
- 读锁会阻塞写，写锁会阻塞读和写
  - 对MyISAM表的读操作，不会阻塞其它进程对同一表的读请求，但会阻塞对同一表的写请求。只有当读锁释放后，才会执行其它进程的写操作。
  - 对MyISAM表的写操作，会阻塞其它进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作。

MyISAM不适合做写为主表的引擎，因为写锁后，其它线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。



## 行锁

- 会出现死锁，发生锁冲突几率低，并发高。
- 在MySQL的InnoDB引擎支持行锁，与Oracle不同，MySQL的行锁是通过索引加载的，也就是说，行锁是加在索引响应的行上的，要是对应的SQL语句没有走索引，则会全表扫描，行锁则无法实现，取而代之的是表锁，此时其它事务无法对当前表进行更新或插入操作。



# 实现线程同步

1. 同步方法 使用`synchronized`关键字修饰方法
2. 同步代码块 使用`synchronized`关键字修饰的语句块
3. 使用特殊域变量 `volatile`实现线程同步
   - volatile关键字为域变量的访问提供了一种免锁机制
   - 使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新
   - 因此每次使用该域就要重新计算，而不是使用寄存器中的值
   - volatile不会提供任何原子操作，它也不能用来修饰final类型的变量
4. 使用重入锁实现线程同步 ReentrantLock类是可重入、互斥、实现了Lock接口的锁，它与使用synchronized方法和快具有相同的基本行为和语义，并且扩展了其能力
   - ReentrantLock() : 创建一个ReentrantLock实例
   - lock() : 获得锁
   - unlock() : 释放锁
5. **使用局部变量实现线程同步** 如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本， 副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。
   -  ThreadLocal() : 创建一个线程本地变量
   - get() : 返回此线程局部变量的当前线程副本中的值
   - initialValue() : 返回此线程局部变量的当前线程的"初始值"
   - set(T value) : 将此线程局部变量的当前线程副本中的值设置为value
6. **使用阻塞队列实现线程同步**
7. **使用原子变量实现线程同步**

>那么什么是原子操作呢？
原子操作就是指将读取变量值、修改变量值、保存变量值看成一个整体来操作
即-这几种行为要么同时完成，要么都不完成。

在java的**util.concurrent.atomic包中提供了创建了原子类型变量的工具类**，使用该类可以简化线程同步。

其中**AtomicInteger** 表可以用原子方式更新int的值，可用在应用程序中(如以原子方式增加的计数器)，
但不能用于替换Integer；可扩展Number，允许那些处理机遇数字类的工具和实用工具进行统一访问。

AtomicInteger类常用方法：

- AtomicInteger(int initialValue) : 创建具有给定初始值的新的AtomicInteger
- addAddGet(int dalta) : 以原子方式将给定值与当前值相加
- get() : 获取当前值





# CAS问题

CAS全称Compare and Swap，也就是比较和交换

CAS的思想比较简单，主要涉及到三个值：当前内存值V、预期值(旧的内存值)O、即将更新的内存值U,当且仅当预期值O与当前内存值V相等时，将内存值V修改为更新值U，返回true，否则返回false。

## 应用场景

CAS主要使用在一些需要上锁的场景充当乐观锁解决方案，一般在一些简单且要上锁的操作但又不想引入锁场景，这时候来使用CAS代替锁。

 CAS主要涉及到三个问题：ABA问题、自旋带来的消耗、CAS只能单变量

### ABA问题

ABA问题是指有一个线程t1在进行CAS操作时，其他线程t2将变量A改成了B，然后又将其改成A，这时候t1发现A并没有改变，因此进行了交换操作，由于在交换操作进行前变量A其实是有变化的，只不过最终又修改回A了，此A非彼A，这时候进行交换操作在一些业务场景下很可能要出问题，要解决ABA问题有2种方案。

#### 解决方案

方案一：在对变量进行操作的时候给变量加一个版本号，每次对变量操作都将版本号加1，常见在数据库的乐观锁中可见。

2、Java提供了相应的原子引用类AtomicStampedReference，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。

### 自旋带来的消耗

自旋带来的消耗CAS自旋如果很长时间都不成功，这会给CPU带来很大的开销

#### 解决方案：

1、代码层面破坏掉for循坏，设置合适的循环次数。

2、使用JVM能支持处理器提供的pause指令来提升效率，它可以延迟流水线执行指令，避免消耗过多CPU资源。



## CAS只能单变量

 CAS只能单变量对于一个共享变量，可以使用CAS方式来保证原子操作，但是当多个共享变量时，那就无法使用CAS来保证原子性。JDK1.5开始，提供了AtomicReference类来保证引用对象之前的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

在JDK1.5中新增的java.util.concurrent(JUC),就是建立在CAS之上的，一般来说CAS这种乐观锁适合读多写少的场景。

























































































#### 相关资料来源

[1]: https://biscuitos.github.io/blog/ATOMIC/	" atomic的原理及Linux下的原子操作使用说明"
[2]: https://blog.csdn.net/qq_30379689/article/details/80785650	"java 的原子特性"

[3]: https://blog.csdn.net/u011552404/article/details/79980796	"数据结构-- String"

