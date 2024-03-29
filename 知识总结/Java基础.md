# Java 基础知识学习笔记

---
layout: post
title: "Java 基础知识点"
date: 2020-12-29 22:10
comments: true
tags: 
	- Java
	- 知识总结

---

可能不全，但是是个人感觉比较重要的 Java 基础知识。

## 1. 面向对象三大特性

### 1.1 封装

定义：将类的某些信息隐藏在类内部，不允许外部程序直接访问，而是通过该类提供的方法来实现对隐藏信息的操作和访问。

实质就是将对象的部分属性私有化（private），使用公用（public）方法供外界访问修改。

优点：

1. 只能通过规定的方法访问数据。
2. 隐藏类的实例细节，方便修改和实现。

### 1.2 继承

就是子类继承父类，Java 不支持多继承，而 C++ 支持，继承有三点需要注意：

- 子类拥有父类的所有属性和方法，包括私有属性和私有方法，但是**父类中的私有属性和私有方法子类无法访问，只是拥有**；
- 子类可以拥有自己的属性和方法；
- 子类可以用自己的方法实现父类的方法（方法**重写**）。

优点：子类拥有父类的所有属性和方法从而实现代码复用。

#### *重写与重载的区别：

- **重载**： 发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同，发生在编译时。
- **重写**： 发生在父子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类；如果父类方法访问修饰符为 private 则子类就不能重写该方法。

|            | 重载                    | 重写             |
| ---------- | ----------------------- | ---------------- |
| 发生的类   | 同一个类                | 父子类           |
| 方法名     | 相同                    | 相同             |
| 参数       | 类型 / 个数 / 顺序 不同 | 相同             |
| 返回值     | 可以不同                | 范围小于等于父类 |
| 访问修饰符 | 可以不同                | 范围大于等于父类 |

### 1.3 多态

所谓多态就是指：

程序中定义的**引用变量所指向的具体类型和通过该引用变量使用方法调用在编程时并不确定，而是在程序运行期间才确定**，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

java 中多态分为两种：

#### 1.3.1 引用多态

父类的引用可以指向本类的对象；

父类的引用可以指向子类的对象；

```java
public static void main(String[] args) {
    Animal obj1 = new Animal();	// 父类的引用指向本类
    Animal obj2 = new Dog();	// 父类的引用指向子类
}
```

#### 1.3.2 方法多态

根据上述创建的两个对象：本类对象和子类对象，同样都是父类的引用，当我们指向不同的对象时，它们调用的方法也是多态的。

创建本类对象时，调用的方法为本类方法；

创建子类对象时，调用的方法为子类重写的方法或者继承的方法；

使用多态的时候要注意：**如果我们在子类中编写一个独有的方法（没有继承父类的方法），此时就不能通过父类的引用创建的子类对象来调用该方法！！！**

Java 中实现多态有两种方式：

- 继承：多个子类对同一个方法重写
- 接口：多个接口实现类覆盖接口中的同一种方法

#### *接口与抽象类

|          | 接口                                                         | 抽象类                                                       |
| :------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 方法     | 默认是`public`方法，方法在接口中不能有实现（Java 8 中可以有默认实现） | 可以有`public`、`protected`和`default`方法，可以有默认实现，但是为了防止被重写不能使用`private`方法 |
| 变量     | 只能有`static`和`final`变量                                  | 都可以                                                       |
| 继承     | 一个类可以实现多个接口，但是一个接口可以扩展多个接口         | 一个类只能实现一个抽象类                                     |
| 设计思路 | 对行为的抽象，行为规范                                       | 对类的抽象，模板设计                                         |

## 2. 关键字

### 2.1 static

该关键字有两个作用：

1. 为某特定数据类型或对象分配存储空间，与创建的类无关；
2. 实现某个方法或属性与类而不是对象关联在一起，即在不创建对象的情况下就可以通过类来直接使用类的方法或属性。

可以修饰以下元素：

- 变量：静态变量属于类而不属于对象，该类所有的实现对象共享同一个静态变量；
- 方法：静态方法可以不通过对象而通过类直接调用，静态方法只能访问本类中的静态变量；
- 代码块：静态代码块只能定义在类定义下（方法外），在类被加载时执行；
- 内部类：一旦内部类使用static修饰，那么此时这个内部类就升级为顶级类，可以在外部类不创建对象的情况下直接创建对象；静态内部类不能访问任何外围类的非静态变量和方法；
- 导入包：静态导入包能够导入指定的包下的所有静态变量。

### 2.2 final

可以用于修饰变量、方法和类，分别表示**变量不可变、方法不可覆盖、类不可被继承**：

- 变量：如果是基本数据类型，则初始化后数值就不可再改变；如果是引用类型的变量，则初始化后不能再让他指向另一个对象，即引用不可变；
- 方法：使得继承类无法对其进行修改，类中所有的 private 方法都隐式地指定为 final 方法；
- 类：使得类无法被继承，final 类中所有的成员方法都被隐式地指定为 final 方法。

## 3. == 与 equals

### 3.1 ==

`==`在基本数据类型中比较的是值，在引用数据类型中是比较两个对象的地址是否相同，即判断被比较的二者是否是同一个对象

### 3.2 equals()

- 当被比较的类没有重写`equals()`方法时，该方法相当于用`==`比较两个对象；
- 当被比较的类覆盖了`equals()`方法时，一般是将其重写为比较被比较类型的值，相等即返回`true`。
- 在本质是散列表的类（HashMap, HashTable, HashSet）中，重写`equals()`需要同时重写`hashCode()`，不然散列值不同时，无论如何两个类也不会相同。

## 4. I/O

### 4.1 I/O分类

Java I/O 都是由`InputStream` 、`Reader`、`OutputStream`、`Writer`这四个基类派生出来的，可以按照以下分类方法进行分类。

按操作方式分类：

![image-20210318214142677](https://gitee.com/wtychn/ImageBed/raw/master/img/image-20210318214142677.png)

按操作对象分类：

![image-20210318214237185](https://gitee.com/wtychn/ImageBed/raw/master/img/image-20210318214237185.png)

### 4.2 BIO、NIO、AIO

- **同步、异步**：客户端在请求数据的过程中，能否做其他事情。
- **阻塞、非阻塞**：客户端与服务端是否从头到尾始终都有一个持续连接，以至于占用了通道，不让其他客户端成功连接。

#### 4.2.1 **BIO（Blocking I/O）同步阻塞**

客户端在请求数据的过程中保持一个连接，不能做其他事。

在这一个连接中，服务端也需要一个线程来维护这个连接，由于服务端对应多个客户端，因此在阻塞过程中服务端压力更大。而且在等待过程中客户端做不了其他事，所以等待过程中其本身的性能也没得到充分释放。

BIO 的问题可以通过线程池解决，在活动连接数较低的情况下这种模型编程简单，可以让每一个连接专注于自己的I/O，不用考虑系统过载、限流等问题。但是这种模型**无法应对高并发**情况。

#### 4.2.2 NIO（Non-blocking/New I/O）同步非阻塞

客户端在请求数据的过程中，不用保持一个连接，不能做其他事情。

客户端发送一个请求，并建立一个连接，服务端接收到了。如果服务端没有数据，就告知客户端“没有数据”；如果有数据，则返回数据。客户端接到了服务端回复的“没有数据”就断开连接，过了一段时间后，客户端重新问服务端是否有数据。服务器重复以上步骤。

客户端反复建立连接询问，如果没有数据则断开连接。这个过程称为**轮询**。NIO用轮询代替了始终保持一个连接。

Java 1.4 中引入了 NIO 模型，对应`java.nio`包，他与 BIO 模型存在对应关系：

| NIO                   | BIO            |
| --------------------- | -------------- |
| `SocketChannel`       | `Socket`       |
| `ServerSocketChannel` | `ServerSocket` |

NIO 的操作方法是基于通道、面向缓冲的，能够应对高负载、高并发的情况。

NIO 有三个实体，缓冲区 Buffer、通道 Channel、多路复用器 Selector

- **Buffer** 是客户端存放服务端信息的一个**容器**，服务端如果把数据准备好了，就会通过Channel往Buffer里面传。
- **Channel** 是客户端与服务端之间的**双工连接通道**。所以在请求的过程中，客户端与服务端中间的Channel就在不停的执行“连接、询问、断开”的过程。直到数据准备好，再通过Channel传回来。
- **Selector** 是服务端选择 Channel 的一个复用器。Selector 有两个核心任务：**监控数据是否准备好**，**应答Channel**。具体说来，多个 Channel 反复轮询时，Selector 就看该 Channel 所需的数据是否准备好了；如果准备好了，则将数据通过 Channel 返回给该客户端的 Buffer，该客户端再进行后续其他操作；如果没准备好，则告诉Channel还需要继续轮询；多个 Channel 反复询问Selector，Selector 为这些 Channel 一一解答。

#### 4.2.3 AIO（Asynchronous I/O）异步非阻塞

客户端在请求数据的过程中，不用保持一个连接，能做其他事情。

客户端向服务端请求数据。服务端若有，则返回数据；若无，则告诉客户端“没有数据”。客户端收到“没有数据”的回复后，就做自己的其他事情。服务端有了数据之后，就主动通知客户端，并把数据返回去。

但是，服务端需要主动通知客户端，关于“通知”的业务逻辑肯定是需要消耗资源的。客户端本来在做别的事情，突然前面的事情又插过来要做了，必然引入了一个多线程的协调工作。

AIO 目前的实际应用仍然较少。

## 5. 集合

### 5.1 HashMap

#### 5.1.1 Java  8 之前

所有发生哈希冲突的数据会被放在同一条**链表**中。

存放 Entry<K, V> 链表的 table 数组初始大小为 16.

table 数组的长度始终保持为 2 的幂次方。

##### 1) hash()

1. 首先求对象的 hashCode；
2. 将 hashCode 进行分散（1.8 之前的散列算法较为复杂，不详细描述）

##### 2) indexFor

根据 hashCode 和 table 数组长度计算存放下标。计算方式：

```
h & (length - 1)
```

注意：table 数组的长度始终保持为 2 的幂次方的好处是通过下标计算后的结果为 hashCode 对数组长度取余。

##### 3) put()

插入键值对时确认对象是否相等，HashMap 判定是同一个对象的条件：hashCode 一致，且 equals 比对一致的对象。

因此修改 equal() 方法时需要先修改 hashCode() 方法。

##### 4) resize()

扩容公式：

```
threshold = size * loadFactor
```

其中 threshold 为扩展后的阈值，size 为当前 table 长度，loadFactor 为负载因子（默认 0.75）。

当 size 达到 threshold 之后就将 size * 2，并使用 transfer() 方法遍历所有键值对重新计算下标。

#### 5.1.2 Java 8 

根据泊松分布概率计算得到，链表长度大于等于 8 时转为红黑树，红黑树节点小于等于 6 时退化为链表。

##### 1) hash()

计算方式简化为`h >>> 16`

##### 2) resize()

取消了 transfer() 方法，全都在 resize() 中完成。

优化为：当重新计算出的下标未超出旧容量的链表，即`hash & oldCap == 0`的部分不做处理；超出的部分：新下标 = 原下标 + 旧容量

##### 3）红黑树退化为链表

有两个地方会判断并退化成链表的条件

1. remove时退化

```java
// 在 removeTreeNode的方法中
// 在红黑树的root节点为空 或者root的右节点、root的左节点、root左节点的左节点为空时 说明树都比较小了
if (root == null || (movable && (root.right == null || (rl = root.left) == null || rl.left == null))) {
  	tab[index] = first.untreeify(map);  // too small
    return;
}
```

2. 在扩容时 low、high 两个 TreeNode 长度小于等于 6 时 会退化为链表。

<img src="https://img-blog.csdnimg.cn/20181105181728652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvc2hpbWF4aWFvMQ==,size_16,color_FFFFFF,t_70" alt="hashmap流程图" style="zoom:50%;" />

### 5.2 ConcurrentHashMap

#### 5.2.1 java 8 之前

<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/image-20210717223906105.png" alt="image-20210717223906105" style="zoom:40%;" />

由 Segment 数组、HashEntry 组成，和 HashMap 一样，仍然是数组加链表。

采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。

#### 5.2.2 java 8 之后

<img src="https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaIPf1sDCN5zcDdGsibZwyzy9r6hAjXjUBUbenB9FQ9BKXGQaQ0M64Q7HP9fTYSKBWkrrZfmsibdEcKDg/640?wx_fmt=png" alt="640?wx_fmt=png" style="zoom:90%;" />

抛弃了原有的 Segment 分段锁，而采用了 `CAS + synchronized` 来保证并发安全性。

将之前存放数据的 HashEntry 改为 Node.

### 5.3 ArrayList

扩容机制：简而言之就是不足时将容量扩充为原来的 1.5 倍

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210528165036452.png" alt="image-20210528165036452" style="zoom:80%;" />

## 6. Java 8 新特性

### 6.1 Lambda 表达式

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

lambda 表达式的语法格式如下：

``` java
(parameters) -> expression or (parameters) ->{statements; }
```

### 6.2 方法引用

方法引用通过方法的名字来指向一个方法。方法引用可以使语言的构造更紧凑简洁，减少冗余代码。方法引用使用一对冒号 :: 。

```java
public class Java8Tester {
    public static void main(String args[]) {
        List names = new ArrayList();
        names.add("Google");
        names.add("Runoob");
        names.add("Taobao");
        names.forEach(System.out::println);
    }
}
```

### 6.3 函数式接口

函数式接口 (Functional Interface) 就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为lambda表达式。函数式接口可以现有的函数友好地支持 lambda。

- **java.lang.Runnable**
- **java.util.concurrent.Callable**
- java.security.PrivilegedAction
- **java.util.Comparator**
- java.io.FileFilter
- java.nio.file.PathMatcher
- java.lang.reflect.InvocationHandler
- java.beans.PropertyChangeListener
- java.awt.event.ActionListener
- javax.swing.event.ChangeListener
- java.util.function (Java 8 新增)

### 6.4  默认方法

Java 8 新增了接口的默认方法。简单说，默认方法就是接口可以有实现方法，而且不需要实现类去实现其方法。我们只需在方法名前面加个 default 关键字即可实现默认方法。

```java
public interface vehicle {
    default void print() {
        System.out.println("我是一辆车!");
    }
}
```

### 6.5 流式编程

在Java 8中,集合接口有两个方法来生成流：

- stream() −为集合创建串行流
- parallelStream() − 为集合创建并行流

常用方法：

- forEach
- map
- filter
- limit
- sorted
- Collectors

### 6.6 Optional 实例

```java
public class Java8Tester {
    public static void main(String args[]) {
        Java8Tester java8Tester = new Java8Tester();
        Integer value1 = null;
        Integer value2 = new Integer(10);
        // Optional.ofNullable - 允许传递为 null 参数
        Optional<Integer> a = Optional.ofNullable(value1);
        // Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
        Optional<Integer> b = Optional.of(value2);
        System.out.println(java8Tester.sum(a, b));
    }
 
    public Integer sum(Optional<Integer> a, Optional<Integer> b) {
        // Optional.isPresent - 判断值是否存在
        System.out.println("第一个参数值存在: " + a.isPresent());
        System.out.println("第二个参数值存在: " + b.isPresent());
        // Optional.orElse - 如果值存在，返回它，否则返回默认值
        Integer value1 = a.orElse(new Integer(0));
        //Optional.get - 获取值，值需要存在
        Integer value2 = b.get();
        return value1 + value2;
    }
}
```

