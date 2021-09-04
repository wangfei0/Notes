---
typora-copy-images-to: images
typora-root-url: images
---

# 1. 面向对象

## 封装

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部消息。但是可以提供一些可以被外界访问的方法来操作属性。

## 继承

继承是使用已存在的类作为基础创建新类的技术。子类继承父类，子类拥有父类的所有属性和方法（包括私有属性和方法）。子类拥有自己的属性和方法，也就是对父类进行扩展。子类可以用自己的方式实现父类的方法（重写）。

## 多态

表示一个对象具有多种形态。具体表现为父类引用指向子类的实例。如果子类重写了父类的方法，真正执行的是子类覆盖的方法。如果子类没有覆盖父类的方法，执行的是父类的方法。

# 2. 反射

## Java 程序中获得 Class 对象的三种方式

1. 使用 Class 类的 forName(String clazzName)静态方法
2. 调用某个类的 class 属性来获取该类对应的 Class 对象
3. 调用某个对象的 getClass()方法

## 概念

JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 java 语言的反射机制。

## 静态编译与动态编译

- **静态编译：** 在编译时确定类型，绑定对象
- **动态编译：** 运行时确定类型，绑定对象

## 应用场景

[应用场景举例](https://segmentfault.com/a/1190000010162647?utm_source=tuicool&utm_medium=referral)

1. 我们在使用 JDBC 连接数据库时使用 `Class.forName()`通过反射加载数据库的驱动程序；
2. Spring 框架也用到很多反射机制，最经典的就是 xml 的配置模式。Spring 通过 XML 配置模式装载 Bean 的过程：1) 将程序内所有 XML 或 Properties 配置文件加载入内存中; 2)Java 类里面解析 xml 或 properties 里面的内容，得到对应实体类的字节码字符串以及相关的属性信息; 3)使用反射机制，根据这个字符串获得某个类的 Class 实例; 4)动态配置实例的属性

# 3. 数据类型

## 基本数据类型

Java有8大基本数据类型。

| 基本类型 | 占用字节 | 位数 | 默认值  |
| :------: | :------: | :--: | :-----: |
|   byte   |    1     |  8   |    0    |
|  short   |    2     |  16  |    0    |
|   int    |    4     |  32  |    0    |
|   long   |    8     |  64  |   0L    |
|  float   |    4     |  32  |   0f    |
|  double  |    8     |  64  |   0d    |
|   char   |    2     |  16  | 'u0000' |
| boolean  |    4     |  1   |  false  |

对于boolean，官方文档未明确定义，它依赖于 JVM 厂商的具体实现。逻辑上理解是占用 1位，但是实际中会考虑计算机高效存储因素。

1. Java 里使用 long 类型的数据一定要在数值后面加上 **L**，否则将作为整型解析：
2. `char a = 'h'`char :单引号，`String a = "hello"` :双引号

**重点：**

每种基本数据类型都有包装类。

## 基本数据的包装类型的常量池

**Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。**

当使用自动装箱方式创建一个Integer对象时，当数值在-128 ~127时，会将创建的 Integer 对象缓存起来，当下次再出现该数值时，直接从缓存中取出对应的Integer对象。

这就解释了为什么Integer(100)==Integer(100)而Integer(1000)!=Integer(1000)。

> Integer.java 类，你会发现有一个内部私有类，IntegerCache.java，它缓存了从-128到127之间的所有的整数对象。

## 直接量

直接量是指在程序中通过源代码直接给出 的值 ， 例如在 int a = 5 ;这行代码中 ， 为变量 a 所分配的初始值 5 就是一个直接量。

## BigDecimal

用于解决精度丢失问题。

BigDecimal 的大小比较：

`a.compareTo(b)` : 返回 -1 表示 `a` 小于 `b`，0 表示 `a` 等于 `b` ， 1表示 `a` 大于 `b`。

BigDecimal 保留几位小数：

通过 `setScale`方法设置保留几位小数以及保留规则。

~~~Java
BigDecimal m = new BigDecimal("1.255433");
BigDecimal n = m.setScale(3,BigDecimal.ROUND_HALF_DOWN);
System.out.println(n);// 1.255
~~~


##  运算符
	++
		如果把++放在左边，
则先把操作数加1，然后才把操作数放入表达式中运算:如果把++放在右边，则先把操作数放入表达式
中运算，然后才把操作数加 1 。
	位运算符
		& : 按位与 。当两位同时为 1 时才返回 1 。
		| : 按位或 。 只要有一位为 1 即可返回 1 。
		~ : 按位非 。 单目运算符，将操作数的每个位(包括符号位 〉 全部取反 。
		^ :按位异或 。 当两位相同时返回 0 ， 不同时返回 l 。
		<<:左移运算符 。

		>> : 右移运算符 。
		>>> : 无符号右移运算符 。
	逻辑运算符
		&&: 与，前后两个操作数必须都是 true 才返回 true ， 否则返回 false 。
		&:不短路与，作用与 &&相同，但不会短路 。
		 ||: 或，只要两个操作数中有一个是 true ，就可以返回 true ，否则返回 fa lse 。
		|: 不短路或，作用与 11相同，但不会短路 。
		!: 非，只需要 一个操作数，如果操作数为 true ，则返回 fa l se ; 如果操作数为 fa l se ，则返回 true 。
		^ : 异或，当两个操作数不同时才返回 true ，如果两个操作数相同则返回 false 。
	三目运算符
		判断?真结果 : 假结果

# 4. Object类

Object类是所有类的基类，包含一些所有类共有的方法。

clone() 方法：实现对象的浅复制，只有实现了Cloneable接口才可以调用此方法

getClass() 方法：final方法，获得运行时的类型

toString() 方法：

finalize() 方法：用于释放资源，垃圾回收时调用

equals() 方法：一般equals和==是不一样的，但是在Object中两者是一样的

~~~java 
public boolean equals(Object obj) {
     return (this == obj);
}
~~~

hashCode() 方法：用于哈希查找，重写了equals方法一般都要重写hashCode()方法

wait() 方法：wait方法就是使当前线程释放该对象的锁

notify() 方法：唤醒在该对象上等待的某个线程

notifyAll() 方法：唤醒在该对象上等待的所有线程

# 5. 一些关键字

**final：** 

	1. 修饰类：类不能被继承
 	2. 修饰方法：方法不能被子类重写
 	3. 修饰属性：属性只能赋值一次，一旦赋值不可更改

> 一道阿里面试题：final修饰引用类型变量的时候，引用指向的是某个对象的地址，所以该地址不能改变，但是对象可以发生变化（他一直在那，但是他变了，哈哈）

abstract	定义抽象类，抽象类主要作为多个类的模板

interface 	定义接口，接口则定义了多类应该遵守的规范

enum	用于创建枚举类

static	static 修饰的类成员属于整个类，不属于单个实例 



# 6. ==与equals()与hashCode()

**==** : 它的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象。(**基本数据类型==比较的是值，引用数据类型==比较的是内存地址**)

**equals()** : 它的作用也是判断两个对象是否相等，它不能用于比较基本数据类型的变量。`equals()`方法存在于`Object`类中，而`Object`类是所有类的直接或间接父类。

Object类中的默认实现是：

~~~java 
public boolean equals(Object obj) {
     return (this == obj);
}
~~~

`hashCode()` 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。`Object` 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法通常用来将对象的 内存地址 转换为整数之后返回。

**注意：** 两个对象equals()方法返回true，则hashCode()方法一定返回true。

​			两个对象的hashCode()方法返回true，但是equasl()方法不一定相等。

# 7. 重载与重写

**重载：** 发生在同一个类中，**方法名必须相同**，参数**类型不同**、**个数不同**、**顺序不同**，**方法返回值和访问修饰符可以不同。**

**重写：** 重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写。

1. 返回值类型、方法名、参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。
2. 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 static 修饰的方法能够被再次声明。
3. 构造方法无法被重写



# 8. 深拷贝与浅拷贝

[详解](https://blog.csdn.net/baiye_xing/article/details/71788741?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

**浅拷贝：** 浅拷贝仅仅复制所拷贝的对象，而不复制它所引用的对象。对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。（面试回答，对引用数据类型只拷贝一份指向原引用实例的地址）。**Object中的clone() 方法就是浅拷贝。**

![这里写图片描述](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\20170513104405646.png)

**深拷贝：**  对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。（即：复制出来当前对象同事复制出当前对象所引用的对象）。

![img](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\20170513105503021.png)

# 9. 接口与抽象类

**抽象类：**

抽象类中定义非抽象方法的意义在于实现子类的共有特性。

抽象类的使用原则如下：
（1）**抽象方法必须为public或者protected**（因为如果为private，则不能被子类继承，子类便无法实现该方法），缺省情况下默认为public；
（2）抽象类不能直接实例化，需要依靠子类采用向上转型的方式处理；
（3）抽象类必须有子类，使用extends继承，一个子类只能继承一个抽象类；
（4）子类（如果不是抽象类）则必须覆写抽象类之中的全部抽象方法（如果子类没有实现父类的抽象方法，则必须将子类也定义为为abstract类。）；

**接口：**接口(interface)是抽象方法和常量值的定义的集合。

定义一个接口，使用interface关键字 。

接口是一种特殊的抽象类，这种抽象类中只包含常量和方法的定义。

**接口中所有的方法，都是抽象方法。**接口中的所有方法都默认是由public abstract修饰的。

**方法和属性默认都是public修饰，也可以使用protected，但不能用private**

**所有的属性都是静态的常量，默认省略了static和final修饰符，属性的值必须实例化（初始化）**

**接口也可以继承另一个接口，使用extends关键字。**



**接口与抽象类的区别（面试回答）**⭐⭐⭐

1. 接口的方法全都是抽象方法，而抽象类可以有非抽象的方法。
2. 接口中只能定义常量，public static final修饰，而抽象类中可以定义普通成员变量。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过 extends 关键字扩展多个接口。
4. 接口方法默认修饰符是 public，接口和抽象类的抽象方法可以有 public、protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰！）。
5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。

```
接口不能用final修饰的原因是接口本来就被编译器自动加上abstract修饰符。而abstract修饰符是无法和final一起使用的。两者设计的初衷相悖：
abstract就是用来设计一个模板，该模板被用来继承或者具体实现。
而final是用来限制一个元素的改动。
```

# 10. String\StringBuilder与StringBuffer

String是不可变类，底层通过字符数组实现

~~~java 
private final char value[]
~~~

> 在 Java 9 之后，String 类的实现改用 byte 数组存储字符串 `private final byte[] value`;

`StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类。

`AbstractStringBuilder` 中也是使用字符数组保存字符串`char[]value` 但是没有用 `final` 关键字修饰，所以这两种对象都是可变的。

StringBuilder线程不安全。

![1603763400650](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\StringBuilder)

StringBuffer线程安全。

![1603763441504](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\StringBuffer)

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

**补充：**

​		jvm对String相加“+”做了优化。[原文在此](https://segmentfault.com/a/1190000021328602)。编译器会把“+”优化成 StringBuilder 的方式。但再细致些，你会发现在编译器优化的代码中，每次循环都会生成一个新的 StringBuilder 实例，同样也会降低系统的性能。jdk1.8之后字符串拼接底层就是创建了一个StringBuilder，然后调用append方法，最后调用toString转化成String。

![1604112445428](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\1604112445428.png)

# 11. Collection(List、Set、Map、Queue)

集合里只能保存对象(实际上只是保存对象的引用变量）

## 11.1 List、Map、Set三个接口各有什么特点

List以特定次序来持有元素，可有重复元素。Set 无法拥有重复元素,内部排序。Map 保存key-value值，value可多值。

## 11.2 Set

### HashSet

#### HashSet 如何检查重复

当你把对象加入`HashSet`时，HashSet 会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用`equals()`方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让加入操作成功。（摘自我的 Java 启蒙书《Head fist java》第二版）

### LinkedHashSet

1. 使用链表维护元素的次序
2. LinkedHashSet 需要维护元素的插入顺序 ，因此性能略低于 HashSet 的性能

### TreeSet 

1. TreeSet 可以确保集合元素处于排序状态
2. TreeSet 采用红黑树 的数据结构来存储集合元素
3. 如果希望 TreeSet 能正常运作， TreeSet 只能添加同一种类型的对象。只有同一类型对象能够比较

 

### HashSet、LinkedHashSet 和 TreeSet 

HashSet 是 Set 接口的主要实现类 ，HashSet 的底层是 HashMap，线程不安全的，可以存储 null 值；

LinkedHashSet 是 HashSet 的子类，能够按照添加的顺序遍历；

TreeSet 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。



## 11.3 Map

### Map的key和value的数据类型

key 和 value 都可以是任何引用类型的数据。包括自定义类。

自定义类型作为key要重写equals方法和hashCode方法。

> 阿里面试题：用作 key 的对象必须实现 hashCode()方法和 equals()方法

### HashMap

[HashMap源码分析]([https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81+%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/HashMap(JDK1.8)源码+底层数据结构分析)

1. 初始容量16

2. 初始负载因子0.75，因此第一次扩容大小是12

3. 每次扩容为原来的2倍

4. key 和 value 都可以是任何引用类型的数据，包括自定义类。**用作 key 的对象必须实现 hashCode()方法**
   **和 equals()方法**。

5. key 不允许重复

6. 事实上 ， Map 提供了 一个 Entry内部类来封装 key-value对 ， 而计算 Entry存储时则只考虑 Entry 封装的key。

   从java源码来看， java是先实现了 Map ，然后通过包装一个所有 value为 null 的Map 就实现了集合 Set。

7. 允许存key和value为null值，如果key ==  null ，value存到第一个位置

#### HashMap为什么线程不安全

1. 扩容是会造成循环链表，在jdk1.8改成了尾插法，解决了这个问题

   HashMap 多线程操作导致死循环问题。

   主要原因在于并发下的 Rehash 会造成元素之间会形成一个循环链表。因为jdk1.7下使用的是头插法，容易形成循环链表。

   不过，jdk 1.8 后解决了这个问题（jdk1.8下改成了尾插法），但是还是不建议在多线程下使用 HashMap,因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。并发环境下推荐使用 ConcurrentHashMap 。

2.  put方法是不出现hash冲突时，多线程情况下直接存入数组会覆盖的问题。

   当两个线程同时put两个hash冲突的元素的时候，如果table[index]是空的if条件都判断过了。然后这样就只能有一个元素放进去了。

   在resize的过程中，先将table指针指向了刚初始化好的newtable，然后再转移元素，这个时候就会造成，如果有新的元素put进来。高低位迁移的时候是直接把head赋值给table的index位置的，所以这个时候会直接覆盖掉在resize过程中put进来的元素。

[多线程下循环链表的形成](https://coolshell.cn/articles/9606.html)

#### 为什么是红黑树不是AVL树

https://blog.csdn.net/21aspnet/article/details/88939297

红黑树牺牲了一些查找性能 但其本身并不是完全平衡的二叉树。因此插入删除操作效率略高于AVL树

AVL树用于自平衡的计算牺牲了插入删除性能，但是因为最多只有一层的高度差，查询效率会高一些。

https://www.jianshu.com/p/37436ed14cc6

#### HashMap 的长度为什么是 2 的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的范围值-2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。

数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）。

**“取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是 2 的幂次方。**

#### HashMap 的扩容

在常规构造器中，没有指定容量时，会在执行put操作的时候才真正构建table数组，初始容量为16，负载因子0.75，第一次扩容阈值为0.75*16=12

当发生哈希冲突并且size大于阈值的时候，需要进行数组扩容，扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去，扩容后的新数组长度为之前的2倍，所以扩容相对来说是个耗资源的操作。

#### HashMap 和 HashSet 区别

 `HashSet` 源码显示：HashSet 底层就是基于 HashMap 实现的。（HashSet 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法。

| HashMap                            | HashSet                                                      |
| :--------------------------------- | :----------------------------------------------------------- |
| 实现了 Map 接口                    | 实现 Set 接口                                                |
| 存储键值对                         | 仅存储对象                                                   |
| 调用 `put()`向 map 中添加元素      | 调用 `add()`方法向 Set 中添加元素                            |
| HashMap 使用键（Key）计算 Hashcode | HashSet 使用成员对象来计算 hashcode 值，对于两个对象来说 hashcode 可能相同，所以 equals()方法用来判断对象的相等性， |

### Hashtable 

1. 线程安全

2. 初始容量为11，每次扩容为原来的2倍加1

3. Hashtable 不允许使用 null 作为 key 和 value

   从HashTable源代码put方法可以看出，首先会对put方法中的值value进行null值判断，为null抛出空指针异常；然后会对key值进行Object类的hashCode方法计算hash值，为null值也会抛出空指针异常，故不能在HashTable键、值中放入null值。

**面试问题：** 

1. 哪些方法同步了，哪些方法没同步？
2. size方法同步了吗？

#### HashMap 和 Hashtable 

**1. 线程是否安全：** HashMap 是非线程安全的，HashTable 是线程安全的,因为 HashTable 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；

**2. 效率：** 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；

**3. 对 Null key 和 Null value 的支持：** HashMap 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；HashTable 不允许有 null 键和 null 值，否则会抛出 NullPointerException。

**4. 初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，Hashtable 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。HashMap 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为 2 的幂次方大小（HashMap 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 HashMap 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。

**5. 底层数据结构：** JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

下面这个方法保证了 HashMap 总是使用 2 的幂作为哈希表的大小。

~~~Java
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
~~~





#### Hashtable

线程安全

初始容量为11，每次扩容为原来的2倍加1 

Hashtable 不允许使用 null 作为 key 和 value

> 1. 哪些方法同步了，哪些方法没同步？
> 2. size方法同步了吗？答：同步了



### TreeMap

1. 红黑树数据结构 
2. 线程不安全
3. 有序

#### HashMap 和 TreeMap 区别

[treeMap源码解析](https://my.oschina.net/90888/blog/1626065)

`TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。

实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。

实现`SortMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力。默认是按 key 的升序排序，不过我们也可以指定排序的比较器。

### LinkedHashMap

使用双向链表来维护 key-value 对的次序

[源码解析](https://www.imooc.com/article/22931)

inkedHashMap 继承自 HashMap，在 HashMap 基础上，通过维护一条双向链表，解决了 HashMap 不能随时保持遍历顺序和插入顺序一致的问题。

### ConcurrentHashMap 

[源码解析](https://mp.weixin.qq.com/s/AHWzboztt53ZfFZmsSnMSw)

线程安全

1.7：segment数组+ReentrantLock 

1. Segment分段锁机制，类似多个HashTable

2. ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素

3. segment数组的hash是取高位

   (hash >>> segmentShift) & segmentMask;

   取低位，这样保证这个hash的高位和低位都参与了运算

   数组内部的hash是(tab.length - 1) & hash;

   因为要保证在segment数组的hash时没有分开的元素要在数组内部区分开  所以hash方法一定要不一样

4. get方法不加锁

   首先value和next是volatile的，所以肯定能获取到最新的值

   当get发生在put之前时，无所谓，因为get已经遍历到链表中间了，而put是头插的，所以put进来的元素不会被遍历到。

   当get放生在put之后，则通过UNSAFE.putOrderedObject保证刚刚插入表头的节点被读取。

5. size操作的三次重试机制（modcount）

1.8：CAS+Synchronized

1. CAS+Synchronized。ABA问题，解决加入version

2. 数组+链表+红黑树

3. get方法不需要加锁

   跟1.7差不多。主要是利用tabat中的unsafe方法获得最新的node

#### ConcurrentHashMap 和 Hashtable 的区别

//todo  hashtable源码解析

[HashTable源码解析]()

ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。

**实现线程安全的方式（重要）：** 

① **在 JDK1.7 的时候，ConcurrentHashMap（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。（JDK1.6 以后 对 synchronized 锁做了很多优化）** 整个看起来就像是优化过且线程安全的 HashMap，虽然在 JDK1.8 中还能看到 Segment 的数据结构，但是已经简化了属性，只是为了兼容旧版本；

② **Hashtable(同一把锁)** :使用 synchronized 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

### Map的遍历

**Entry**

Map中采用**Entry内部类**来表示一个映射项，映射项包含Key和Value (我们总说键值对键值对, 每一个键值对也就是一个Entry)

Map.Entry里面包含getKey()和getValue()方法

**entrySet**

entrySet是 java中 键-值 对的集合，Set里面的类型是Map.Entry，一般可以通过map.entrySet()得到。



entrySet实现了Set接口，里面存放的是键值对。一个K对应一个V。

用来遍历map的一种方法。



Set<Map.Entry<String, String>> entryseSet=map.entrySet();

 

for (Map.Entry<String, String> entry:entryseSet) {

 

  System.out.println(entry.getKey()+","+entry.getValue());

 

}

即通过getKey（）得到K，getValue得到V。

**keySet**

keySet是键的集合，Set里面的类型即key的类型

### Map与Set的联系

Set 和 Map 的关系十分密切， Java 源码就是先实现了HashMap 、 TreeMap等集合，然后通过包装一个所有的 value 都为 null 的 Map 集合实现了 Set 集合类 。

~~~java 
	// Dummy value to associate with an Object in the backing Map
	//空对象常量
    private static final Object PRESENT = new Object();
//构造函数
public HashSet() {
        map = new HashMap<>();
    }
//add方法
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
~~~



## 11.4 List

#### ArrayList

[源码]([https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ArrayList%E6%BA%90%E7%A0%81+%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6%E5%88%86%E6%9E%90](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/ArrayList源码+扩容机制分析))

1. 线程不安全

2. ArrayList类封装了 一个动态的、允许再分配的 Object[]数组 

3. 数组的默认大小
   private static final int DEFAULT_CAPACITY = 10;

4. 每次扩容为原来容量的1.5倍

5. 扩容是通过copy一份原来的数组生成新的数组,

   使用Arrays.copyOf() 方法

   ~~~java 
   private Object[] grow(int minCapacity) {
           return elementData = Arrays.copyOf(elementData,
                                              newCapacity(minCapacity));
       }
   ~~~

6. List 判断两个对象相等只要通过 equals()方法比较返回 true 即可

#### Vector

线程安全

#### LinkedList

[源码]([https://snailclimb.gitee.io/javaguide/#/docs/java/collection/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/LinkedList源码分析))

链表实现的

## 11.5  Queue

Queue 还有一个 Deque 接口， 代表一个"双端队列。双端队列可以同时从两端来添加、 删除元素

**Deque 接口：**

LinkedList 实现类。既可以被当成"栈"来使用 ，也可 以 当成队列使用。尽量避免使用 Stack一一因为 Stack 是古老的集合，性能较差。

## 11.6 Collections工具类

操作 Set 、 List 和 Map 等集合的工具类 

1. reverse(List !ist)
   	反转指定 List 集合中元素的顺序 

2. shuffie(List list)
   	对 List 集合元素进行随机排序 (shuffie 方法模拟了"洗牌"动作)

3. sort(List !ist)
   	根据元素的自然顺序对指定 List 集合的元素按升序进行排序。

4. swap(List !ist, int i, int j)
   	将指定 List 集合中的 i 处元素和 j 处元素进行交换。

5. binarySearch(List list, Object key)
   	二分搜索法搜索指定的 List 集合，以获得指定对象在 List 集合中的索引

6. frequency(Collection c, Object 0)
   	返回指定集合中指定元素的出现次数

## 11.7 线程安全的有哪些

1. Collections包装方法

   Collections针对每种集合都声明了一个线程安全的包装类，在原集合的基础上添加了锁对象，集合中的每个方法都通过这个锁对象实现同步

2. java.util.concurrent包中的集合

   ConcurrentHashMap
   CopyOnWriteArrayList
   CopyOnWriteArraySet

   ​		上面两个是加了写锁的ArrayList和ArraySet，锁住的是整个对象，但读操作可以并发执行

   ConcurrentSkipListMap
   ConcurrentSkipListSet
   ConcurrentLinkedQueue
   ConcurrentLinkedDeque



## 快速失败(fail-fast)？

**快速失败(fail-fast)** 是 Java 集合的一种错误检测机制。**在使用迭代器对集合进行遍历的时候，我们在多线程下操作非安全失败(fail-safe)的集合类可能就会触发 fail-fast 机制，导致抛出 ConcurrentModificationException 异常。 另外，在单线程下，如果在遍历过程中对集合对象的内容进行了修改的话也会触发 fail-fast 机制。**



举个例子：多线程下，如果线程 1 正在对集合进行遍历，此时线程 2 对集合进行修改（增加、删除、修改），或者线程 1 在遍历过程中对集合进行修改，都会导致线程 1 抛出 `ConcurrentModificationException` 异常。

**为什么呢？**

每当迭代器使用 `hashNext()`/`next()`遍历下一个元素之前，都会检测 `modCount` 变量是否为 `expectedModCount` 值，是的话就返回遍历；否则抛出异常，终止遍历。

如果我们在集合被遍历期间对其进行修改的话，就会改变 `modCount` 的值，进而导致 `modCount != expectedModCount` ，进而抛出 `ConcurrentModificationException` 异常。

## 安全失败(fail-safe)？

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。所以，在遍历过程中对原集合所作的修改并不能被迭代器检测到，故不会抛 `ConcurrentModificationException` 异常。

# 12. System.arraycopy() 和 Arrays.copyOf()方法

## System.arraycopy()

~~~Java
        //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
~~~

## Arrays.copyOf()

~~~Java
		//elementData：要复制的数组；size：要复制的长度，返回一个数组
        Arrays.copyOf(elementData, size);
~~~

**联系：**

看两者源代码可以发现 `copyOf()`内部实际调用了 `System.arraycopy()` 方法

**区别：**

`arraycopy()` 需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置。 `copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

# 13. comparable 和 Comparator 

- `comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
- `comparator`接口实际上是出自 java.util 包它有一个`compare(Object obj1, Object obj2)`方法用来排序

comparable用法：

~~~Java
// person对象没有实现Comparable接口，所以必须实现，这样才不会出错，才可以使treemap中的数据按顺序排列
// 前面一个例子的String类已经默认实现了Comparable接口，详细可以查看String类的API文档，另外其他
// 像Integer类等都已经实现了Comparable接口，所以不需要另外实现了
public  class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * T重写compareTo方法实现按年龄来排序
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.getAge()) {
            return 1;
        }
        if (this.age < o.getAge()) {
            return -1;
        }
        return 0;
    }
}
~~~

~~~Java
        //comparator用法
		ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.add(-1);
        arrayList.add(3);
        arrayList.add(3);
        arrayList.add(-5);
        arrayList.add(7);
        arrayList.add(4);
        arrayList.add(-9);
        arrayList.add(-7);
        System.out.println("原始数组:");
        System.out.println(arrayList);
        // void reverse(List list)：反转
        Collections.reverse(arrayList);
        System.out.println("Collections.reverse(arrayList):");
        System.out.println(arrayList);

        // void sort(List list),按自然排序的升序排序
        Collections.sort(arrayList);
        System.out.println("Collections.sort(arrayList):");
        System.out.println(arrayList);
        // 定制排序的用法
        Collections.sort(arrayList, new Comparator<Integer>() {

            @Override
            public int compare(Integer o1, Integer o2) {
                return o2.compareTo(o1);
            }
        });
        System.out.println("定制排序后：");
        System.out.println(arrayList);
~~~

# 14. 序列化

Java类中如果某些字段不想被序列化，可以使用transient修饰。transient 只能修饰变量，不能修饰类和方法。

**为什么不用java自带的序列化方法？**

1. 无法跨语言。这应该是java序列化最致命的问题了。由于java序列化是java内部私有的协议，其他语言不支持，导致别的语言无法反序列化，这严重阻碍了它的应用。
2. 序列后的码流太大。java序列化的大小是二进制编码的5倍多！
3. 序列化性能太低。java序列化的性能只有二进制编码的6.17倍，可见java序列化性能实在太差了

**为什么要序列化？**

将对象转化为二进制byte流的过程。 java对象序列化后可以很方便的存储或者在网络中传输。

> 序列化工具fastjson



# 15. 异常

## 异常类型



![img](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\Java异常类层次结构图.png)

![img](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\Java异常类层次结构图2.png)

在 Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 **Throwable 类**。Throwable： 有两个重要的子类：**Exception（异常）** 和 **Error（错误）** ，二者都是 Java 异常处理的重要子类，各自都包含大量子类。

**Error（错误）:是程序无法处理的错误**，表示运行应用程序中较严重问题。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时。

**Exception（异常）:是程序本身可以处理的异常**。Exception 类有一个重要的子类 **RuntimeException**。RuntimeException 异常由 Java 虚拟机抛出。**NullPointerException**（要访问的变量没有引用任何对象时，抛出该异常）、**ArithmeticException**（算术运算异常，一个整数除以 0 时，抛出该异常）和 **ArrayIndexOutOfBoundsException** （下标越界异常）。

**注意：异常和错误的区别：异常能被程序本身处理，错误是无法处理。**

## 异常捕获

**try-catch-finally** 

- **try 块：** 用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。
- **catch 块：** 用于处理 try 捕获到的异常。
- **finally 块：** 无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。

**注意：** 当 try 语句和 finally 语句中都有 return 语句时，在方法返回之前，finally 语句的内容将被执行，并且 finally 语句的返回值将会覆盖原始的返回值。如下：

~~~Java
public class Test {
    public static int f(int value) {
        try {
            return value * value;
        } finally {
            if (value == 2) {
                return 0;
            }
        }
    }
}
~~~

如果调用 `f(2)`，返回值将是 0，因为 finally 语句的返回值覆盖了 try 语句块的返回值。

## 异常抛出

### throw和throws

使用方法：

throw

```Java
public static void main(String[] args) {
		String s = "abc";
		if(s.equals("abc")) {
            //此处抛出异常
			throw new NumberFormatException();
		} else {
			System.out.println(s);
		}
		//function();
}
```

throws

```java
public static void function() throws NumberFormatException{
	String s = "abc";
	System.out.println(Double.parseDouble(s));
}

```

1. throw出现在方法体内，throws出现在方法头上
2. throws表示出现异常的可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某种异常类型。
3. 两者都是校级处理异常的方式，只是抛出或者可能抛出异常，不会由函数去处理异常，真正的处理异常由函数的上层调用处理。

# 16. 代理模式

简单来说就是 **我们使用代理对象来代替对真实对象(real object)的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。**

**代理模式的主要作用是扩展目标对象的功能，比如说在目标对象的某个方法执行前后你可以增加一些自定义的操作。**

## 静态代理

**静态代理中，我们对目标对象的每个方法的增强都是手动完成的，非常不灵活（_比如接口一旦新增加方法，目标对象和代理对象都要进行修改_）且麻烦(_需要对每个目标类都单独写一个代理类)。** 

从 JVM 层面来说， **静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。**

## 动态代理

**从 JVM 角度来说，动态代理是在运行时动态生成类字节码，并加载到 JVM 中的。**

**JDK动态代理：** **JDK 动态代理只能代理实现了接口的类。**

**CGLIB动态代理：** **CGLIB 可以代理未实现任何接口的类**  CGLIB 动态代理是通过生成一个被代理类的子类来拦截被代理类的方法调用，因此不能代理声明为 final 类型的类和方法。

# 17. 泛型

1, 适用于多种数据类型执行相同的代码（代码复用）

2, 泛型中的类型在使用时指定，不需要强制类型转换（类型安全，编译器会检查类型）

**Java中泛型的生存时间：** 泛型只存在于编译期

# 18. 类加载

[类加载解析](https://www.cnblogs.com/cl-rr/p/9081817.html)

## 类加载器

Bootstrap ClassLoader: 根类加载器 。称为引导 ( 也称为原始或根)类加载器，负责加载 Java 的核心类。JVM 的根类加载器并不是 Java 实现的。

Extension ClassLoader: 扩展类加载器 。

System ClassLoader: 系统类加载器。

**类加载器保证线程安全的方式：**

通过classLoader的loaderClass()方法种加synchronized进行方法同步。

## 类加载过程

1. 加载
   	ClassLoader通过一个类的完全限定名查找此类字节码文件，并利用字节码文件创建一个class对象
      		类加载指的是将类的 class 文件读入内存，并为之创建一个 java.lang. Class 对象
      	类的加载由类加载器完成，类加载器通常由 JVM 提供
      		JVM 提供的这些类加载器通常被称为系统类加载器
   连接阶段负责把类的二进制数据合并到 JRE 中
2. 验证 
   		目的在于确保class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身的安全
      		主要包括四种验证：文件格式的验证，元数据的验证，字节码验证，符号引用验证
3. 准备
   		类变量（static修饰的字段变量）分配内存并且设置该类变量的初始值
      		这里不包含final修饰的static ，因为final在编译的时候就已经分配了
4. 解析
   		主要的任务是把常量池中的符号引用替换成直接引用
5. 初始化
   	主要就是对类变量进行初始化
      		假如这个类还没有被加载和连接，则程序先加载并连接该类 
      		假如该类的直接父类还没有被初始化，则先初始化其直接父类 
      			所以 JVM 最先初始化的总是java .lang.Object 类
      		假如类中有初始化语句，则系统依次执行这些初始化语句 
      	当某个类 变量( 也叫 静态变量)使用了 final 修饰，而且它的值可以在编译时就确定下来，那么程序其他地方使用 该类变量时，实际上并没有使用该类变量，而是相当 于使用了常量 。

## 双亲委派机制

如果一个类收到了类加载的请求，它并不会自己先去加载，而是把这个请求委托给父类加载器去执行

**可以避免类的重复加载**

