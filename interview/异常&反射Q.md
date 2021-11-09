[toc]



# Error和Exception有什么区别

- error表示系统级的错误，是java运行环境内部错误或者硬件问题，不能指望程序来处理这样的问题，除了退出运行外别无选择，它是Java虚拟机抛出的。
- exception 表示程序需要捕捉、需要处理的异常，是由与程序设计的不完善而出现的问题，程序必须处理的问题。



# 5个常见的RuntimeException

1. Java.lang.NullPointerException 空指针异常;出现原因：调用了未经初始化的对象或者是不存在的对象。
2. Java.lang.NumberFormatException 字符串转换为数字异常;出现原因：字符型数据中包含非数字型字符。
3. Java.lang.IndexOutOfBoundsException 数组角标越界异常，常见于操作数组对象时发生。
4. Java.lang.IllegalArgumentException 方法传递参数错误
5. Java.lang.ClassCastException 数据类型转换异常。



# throw和throws的区别

## **throw**

(1)throw 语句用在方法体内，表示抛出异常，由方法体内的语句处理。

(2)throw 是具体向外抛出异常的动作，所以它抛出的是一个异常实例，执行 throw 一定是抛出了某种异常。



## **throws**

(1)@throws 语句是用在方法声明后面，表示如果抛出异常，由该方法的调用者来进行异常的处理。

(2)throws 主要是声明这个方法会抛出某种类型的异常，让它的使用者要知道需要捕获的异常的类型。

(3)throws 表示出现异常的一种可能性，并不一定会发生这种异常。



# **Java中异常分类**

按照异常处理时机:

**编译时异常**(受控异常(CheckedException))和**运行时异常**(非受控异常(UnCheckedException))





# 如何自定义异常

继承Exception是检查性异常，继承RuntimeException是非检查性异常，一般要复写两个构造方法，用throw抛出新异常.如果同时有很多异常抛出，那可能就是异常链，就是一个异常引发另一个异常，另一个异常引发更多异常，一般我们会找它的原始异常来解决问题，一般会在开头或结尾，异常可通过initCause串起来，可以通过自定义异常



# Java中异常处理

首先处理异常主要有两种方式:一种try catch，一种是throws。

## 1. **try catch**:

try{} 中放入可能发生异常的代码。catch{}中放入对捕获到异常之后的处理。

## 2.**throw throws**：

throw是语句抛出异常，出现于函数内部，用来抛出一个具体异常实例，throw被执行后面的语句不起作用，直接转入异常处理阶段。

throws是函数方法抛出异常，一般写在方法的头部，抛出异常，给方法的调用者进行解决。



# 异常打印信息组成

所处线程名字、异常类名、异常信息、异常堆栈、异常的源码，包名，类名，方法名，行数



# 常见方法

- getMessage：错误信息的字符串解释
- getCause：返回异常产生的原因，一般是原始异常如果不知道原因返回null
- printStackTrace：打印异常出现的位置或原因
- toString：返回String格式的Throwable信息，此信息包括Throwable的名字和本地化信息
- initCause：初始化原始异常
- PrintStream和PrintWriter作为产生实现重载，这样就能实现打印栈轨迹到文件或流中





# 什么是Java反射机制

Java的反射（reflection）机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。反射被视为动态语言的关键。



# 举例什么地方用到反射机制

\1. JDBC中，利用反射动态加载了数据库驱动程序。

\2. Web服务器中利用反射调用了Sevlet的服务方法。

\3. Eclispe等开发工具利用反射动态刨析对象的类型与结构，动态提示对象的属性和方法。

\4. 很多框架都用到反射机制，注入属性，调用方法，如Spring。



# **java反射机制的作用**

- 在运行时判定任意一个对象所属的类
- 在运行时构造任意一个类的对象；
- 在运行时判定任意一个类所具有的成员变量和方法；
- 在运行时调用任意一个对象的方法；
- 生成动态代理；



# 反射机制类

```java
java.lang.Class; //类
java.lang.reflect.Constructor;//构造方法
java.lang.reflect.Field; //类的成员变量
java.lang.reflect.Method;//类的方法
java.lang.reflect.Modifier;//访问权限
```



# 反射机制优缺点

优点：运行期类型的判断，动态加载类，提高代码灵活度。

缺点：性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的java代码要慢很多。



# 利用反射创建对象

1.通过一个全限类名创建一个对象 Class.forName(“全限类名”);

例如：com.mysql.jdbc.Driver Driver类已经被加载到 jvm中，并且完成了类的初始化工作就行了 类名.class; 获取Class<？> clz 对象 对象.getClass();

2.获取构造器对象，通过构造器new出一个对象 Clazz.getConstructor([String.class]);Con.newInstance([参数]);

3.通过class对象创建一个实例对象（就相当与new类名（）无参构造器)Cls.newInstance();



# Java反射的底层实现原理

众所周知Java有个Object 类，是所有Java 类的继承根源，其内声明了数个应该在所有Java 类中被改写的方法：hashCode()、equals()、clone()、toString()、getClass()等。其中getClass()返回一个Class 对象,而这个Class 类十分特殊。它和一般类一样继承自Object，当一个class被加载，或当加载器（class loader）的defifineClass()被JVM调用，JVM 便自动产生一个Class 对象。而Class对象是java反射故事起源。Class类提供了大量的实例方法来获取该Class对象所对应的详细信息



**RTTI原理**

Java在运行时识别对象和类的类型信息的主要方式有两种：

一种是反射，

另一种则是RTTI

RTTI即为Run-time Type Identifification，负责在运行时维护类的相关信息，通过运行时类型信息程序能够使用基类的指针或引用来检查这些指针或引用所指的对象的实际派生类型，多态则是基于这种能力而实现的。RTTI主要通过Class类实现，这里说的Class类是指classof classes（类的类），有人总结过：如果说类是对象的抽象和集合的话，那么Class类就是对类的抽象与集合，即其他类都是Class类的对象，当我们调用getClass()方法时，就能够得到对应Class对象的引用。

```java
public class RTTITest {
    public static void main(String[] args) {
        Animal oneAnimal = new Animal();
        System.out.println(oneAnimal.getClass().getName());
        Animal oneDog = new Dog();
        System.out.println(oneDog.getClass().getName());
        Dog dog = new Dog();
        Animal littleDog = dog.propagate();
        System.out.println(littleDog.getClass().getName());
    }
}

class Animal{
    private int age;
    public int grow(){
        this.age++;
        return this.age;
    }
}

class Dog extends Animal{
    public Animal propagate(){
        System.out.println("propagating:");
        return new Animal();
    }
}

com.luxiaobai.reflection.Animal
com.luxiaobai.reflection.Dog
propagating:
com.luxiaobai.reflection.Animal
```

可以看到，new出来的是animal，得到的就是animal；new出来的是dog，得到的就是dog，正应了那句“龙生龙，凤生凤”，不管进行怎么的类型转换，对象本身对应的Class对象（类）也不会发生改变，正是由于这样的机制才能保证多态不会迷失在复杂的类型转换之中。除了getClass()方法，常用的还有forName()方法，即通过类名去获得Class对象。当我们创建某个类对象时（这里的类对象需要区别与Class类对象），Java首先会去内存中检查是否存在该Class类对象，比如我们创建一个dog对象，首先就会去检查是否存在Dog对象，如果内存中不存在，会继续搜索.class文件，在其中搜索Dog类的定义并加载该Class对象，该对象加载成功后，其余的Human对象都参照该Class对象进行创建和相关操作。注：该加载过程都是在第一次使用该类时动态加载到JVM的（因此类中的静态块是在类加载时才完成初始化的）。该过程如下：

- 加载：由类加载器完成，找到对应的字节码，创建一个Class对象
- 链接：验证类中的字节码，为静态域分配空间
- 初始化：如果该类有超类，则对其初始化，执行静态初始化器和静态初始化块

特点：RTTI可以在运行时告诉我们某个对象的类型信息，但是有一个前提：这个类型必须是在编译时就已知的了，否则我们就需要反射机制的支持了。

















































































































































































