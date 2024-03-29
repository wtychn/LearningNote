#  JVM 知识总结
---
layout: post
title: "JVM 牛逼！"
date: 2021-05-30 19:50
comments: true
tags: 
	- JVM
	- 知识总结

---

## 1. Java 内存区域（运行时数据区）

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。

- JDK 1.8 之前：

<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F.png" alt="img" style="zoom:67%;" />

- JDK 1.8 之后：

<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/Java%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9FJDK1.8.png" alt="img" style="zoom:65%;" />

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区（JDK 1.8 之后变为使用直接内存的元空间）
- 直接内存 (非运行时数据区的一部分)

### 1.1 线程私有

#### 1.1.1 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。

程序计数器主要有两个作用：

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意**：程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。

#### 1.1.2 虚拟机栈

与程序计数器一样，Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。

Java 内存可以粗糙的区分为**堆内存（Heap）**和**栈内存 (Stack)**，**其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

局部变量表主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。

- **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常异常。

#### 1.1.3 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 **Native 方法（非 Java 代码方法）**服务。 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误。

### 1.2 线程共有

#### 1.2.1 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

**Java 堆是垃圾收集器管理的主要区域**，因此也被称作**GC 堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。进一步划分的目的是更好地回收内存，或者更快地分配内存。

在 JDK 7 版本及JDK 7 版本之前，堆内存被通常被分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/JVM%E5%A0%86%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84-JDK7.png" alt="JVM堆内存结构-JDK7" style="zoom: 80%;" />

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/JVM%E5%A0%86%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84-jdk8.png" alt="JVM堆内存结构-JDK8" style="zoom:80%;" />

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1（Eden 区->Survivor 区后对象的初始年龄变为 1），当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当JVM花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发`java.lang.OutOfMemoryError: Java heap space` 错误。(和本机物理内存无关，和你配置的内存大小有关！)
3. ......

#### 1.2.2 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息**（对象类型信息）**、常量、静态变量、即时编译器编译后的代码等数据。虽然 **Java 虚拟机规范把方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

##### 方法区与永久代之间的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

##### 使用元空间替代方法区的原因

1. 整个永久代有一个 JVM 本身设置固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

   > 当你元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

1. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。
2. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

#### 1.2.3 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

### 1.3 直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

## 2. 对象的创建

### 2.1 判断是否执行类加载

判断常量池中是否能够找到符号引用，是否被类加载过；

### 2.2 内存分配

根据堆中内存是否规整来执行内存分配，内存是否规整主要由垃圾收集器决定。内存分配规则有两种：

1. **指针碰撞（Bump the Pointer）**
   如果 Java 堆中的**内存是绝对规整的**，所有用过的内存放一边，空闲的内存放到一边，中间放着指针为分界点，分配内存就是把指针向空闲的一边挪动一段与对象大小相等的距离。
2. **空闲列表（Free List）**
   如果 Java 堆中的**内存并不是规整**的，已使用的内存和空间相互交错，虚拟机会将可以用的内存维护到一个列表上，在分配内存时从这个列表中找到一块足够大的空间划给对象。然后更新列表记录。

### 2.3 线程安全

在虚拟机上创建对象是非常频繁的行为，所以要做到线程安全，有以下两种方式可实现：

1. 堆分配内存空间的动作进行同步处理，实际上 JVM 采用 CAS (Cmpare And Set) 配上失败重试的方式保证更新操作的原子性；
2. 把内存分配的动作按照线程划分在不同的空间之中进行，即为每个线程在 java 堆中预先分配一块小内存，称为**本地线程分配缓冲区**（Thread Local Allocation Buffer, TLAB）。分配内存时在线程的 TLBA 上分配，只有 TLAB 用完并分配新的 TLAB 时，才需要同步锁定。JVM 是否使用 TLAB 可以通过`-XX:+UseTLAB`参数来设定。

### 2.4 初始化对象内存空间

内存分配完成后，JVM 将分配到的内存空间都初始化为零值（不包括对象头）。

### 2.5 对象头的设置

将对象的类、哈希码、对象的 GC 分代年龄等信息设置到对象头之中。

### 2.6 执行 Java 的 <init>() 方法

设置完对象头后，从JVM的角度来看一个对象已经完成了，但是从 java 程序的角度来看还没有创建完成呢。此时就需要执行 <init>() 方法，调用构造方法等过程，这样一个真正可用的对象才算完全的产生出来。

## 3. 类加载机制

类加载的整个生命周期会经历加载、验证、准备、解析、初始化、使用、卸载七个阶段。

<img src="https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=898673103,546370995&fm=26&gp=0.jpg" alt="img" style="zoom:80%;" />

### 3.1 流程

#### 3.1.1 加载

1. 通过类型的完全限定名，产生一个代表该类型的二进制数据流；
2. 解析这个二进制数据流所代表的的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个表示该类型的 java.lang.Class 类的对象，作为方法区这个类的各种数据的访问入口。

#### 3.1.2 验证

确保 Class 文件的字节流中包含的信息符合《Java 虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身安全。

#### 3.1.3 准备

正式为类中定义的变量（即静态变量，被 static 修饰的变量）分配内存并设置类变量初始值。

**注意**：此处并没有赋值操作，所以初始化值应该是数据零值。

#### 3.1.4 解析

虚拟机将常量池内的符号引用替换为直接引用的过程。

#### 3.1.5 初始化

正式执行 java 代码。首先执行 <clinit>() 方法，该方法会首先收集所有类变量的赋值动作和静态语句块中的语句合并产生，再此之前还会执行父类的这部分代码。

### 3.2 类加载器

<img src="https://img-blog.csdn.net/20170625231013755?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" alt="img" style="zoom:80%;" />

#### 3.2.1 类加载器种类

| 名称                        | 加载路径                                                     |
| --------------------------- | ------------------------------------------------------------ |
| 启动类加载器（Bootstrap）   | `<JAVA_HOME>/lib`路径下的核心类库或`-Xbootclasspath`参数指定的路径下的jar包 |
| 扩展类加载器（Extension）   | `<JAVA_HOME>/lib/ext`路径下或被系统变量`java.ext.dirs`指定位路径中的所有类库 |
| 应用类加载器（Application） | 用户类路径（ClassPath）上的所有类库，没有自定义类加载器时作为程序中默认的类加载器 |

#### 3.2.2 双亲委派机制

如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派机制。

采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关**可以避免类的重复加载**，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。

## 4. 垃圾收集 GC

### 4.1 判断对象死亡的方法

#### 4.1.1 判断算法

1. **引用计数法**：主流虚拟机不使用，难以解决循环引用问题。
2. **可达性分析法**：GC Roots 的引用链。可以作为 GC Roots 的对象包括：虚拟机栈中引用对象、方法区中类静态属性引用的对象、方法区中常量引用对象、本地方法栈中引用对象、虚拟机内部引用、同步锁持有对象。

#### 4.1.2 死了，但没完全死

1. 第一次标记：没有与 GC Roots 相连接的引用链；
2. 第二次标记：判断是否有必要执行 finalize() 方法，如果没有覆盖该方法或该方法已经执行过一次，则没必要执行，直接第二次标记，即将回收。
3. 如果需要执行 finalize() 方法，那么该对象被加入 F-Queue ，如果该对象在 finalize() 方法中拯救自己（重新与引用链上任意对象建立关联），则会在第二次标记中被移出即将回收的集合。

**注意：**finalize() 方法运行代价高昂，不确定性大，无法保证各个对象的调用顺序，如今已被官方声明为不推荐使用的语法。

#### 4.1.3 引用的定义

##### 1）强引用

最传统的引用。如果一个对象具有强引用，垃圾回收器绝不会回收它。当內存空间不足，Java 虛拟机宁愿抛出 OutofMemoryError 错误，使程序异常终止。

##### 2）软引用

如果內存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

##### 3）弱引用

相对于软引用，弱引用关联的对象只能生存到下一次了垃圾回收之前。无论当前内存是否足够，都会回收掉只被弱引用关联的对象。

##### 4）虚引用

如果一个对象仅持有虚引用，那么它就和没有任何引用一样在任何时候都可能被垃圾回收。只会在被回收时收到一个通知。

### 4.2 垃圾收集算法

#### 4.2.1 分代收集理论

现在的 java 虚拟机会把 java 堆分成新生代和老年代主要是由分代收集理论决定的。

根据分代收集理论，垃圾回收可以分为：

1.  **Partial GC** 部分收集
   1. Minor GC 新生代收集
   2. Major GC 老年代收集（目前只有 CMS 收集器支持）
   3. Mixed GC 混合收集

2. **Full GC** 整堆收集

#### 4.2.2 标记-清除算法

标记-清除算法采用从根集合（GC Roots）进行扫描，对存活的对象进行标记，标记完毕后，再扫描整个空间中未被标记的对象，进行回收。

**缺点：**

1. 执行效率不稳定，对象越多效率越低；
2. 标记清除后产生大量不连续的内存碎片。

#### 4.2.3 标记-复制算法

**主要用于新生代。**

为了解决标记清除算法的效率问题，出现了复制算法，它将可用内存按容量划分为大小相等的两块，每次使用其中的一块。当这块的内存用完了，就将还存活着的对象复制到另一块上面，然后再把已使用过的内存空间一次清理掉。

**缺点：**

内存缩小为原来的一半，代价太高。

考虑到绝大部分对象都是朝生夕死的，在虚拟机中将内存分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 和其中的一块 Survivor。当回收时，将 Eden 和 Survivor 中还存活着的对象一次性地拷贝到另外一块 Survivor 空间上，最后清理掉 Eden 和刚才用过的 Survivor 空间。在 HotSpot 虚拟机中默认将 Eden 和 Survivor 区比例设为 8 : 1, 这样每次新生代的内存空间就为整个新生代容量的 90%

#### 4.2.4 标记-整理算法

**主要用于老年代。**

标记-整理算法采用标记-清除算法一样的方式进行对象的标记，但在清除时不同，在回收不存活的对象占用的空间后，会将所有的存活对象往左端空闲空间移动，并更新对应的指针。标记-整理算法是在标记-清除算法的基础上，又进行了对象的移动，因此成本更高，但是却解决了内存碎片的问题。

**缺点：**

整理时移动的话就会造成整个系统运行中断，即 "stop the world"；不整理的话就会造成内存碎片化。

因此需要好好规划移动的时机。

### 4.3 垃圾收集器

<img src="https://gitee.com/wtychn/ImageBed/raw/master/img/image-20210709162158245.png" alt="image-20210709162158245" style="zoom: 70%;" />

相连的垃圾收集器可以搭配使用。

#### 4.3.1 Serial / Serial Old

单线程垃圾收集器。

特点：

1. 所有收集器中内存开销最小的；
2. 回收时会暂停所有用户线程。

#### 4.3.2 ParNew

Serial 收集器的多线程版本，是除了 Serial 之外唯一能与 CMS 搭配使用的新生代收集器。

#### 4.3.3 Parallel Scavenge / Parallel Old

追求高吞吐量，高效利用CPU。吞吐量一般为99%

```
吞吐量 = 用户线程时间 / (用户线程时间 + GC线程时间)
```

适合后台应用等对交互相应要求不高的场景。是server级别默认采用的GC方式

#### 4.3.4 CMS

首个实现并发标记清除垃圾收集器。

**应用场景**：适用于注重服务的响应速度，希望系统停顿时间最短，给用户带来更好的体验等场景下。如web程序、b/s服务。

执行流程为：初始标记 -> 并发标记 -> 重新标记 -> 并发清除

缺点：

1. 资源敏感，无法处理"浮动垃圾"，即清除过程中虚拟机产生的垃圾。
2. 容易造成空间碎片的产生。

#### 4.3.5 G1

与其他收集器不同，G1将堆内存“化整为零”，将堆内存划分成多个大小相等独立区域（Region），每一个 Region 都可以根据需要，扮演新生代的 Eden 空间、Survivor 空间，或者老年代空间。收集器能够对扮演不同角色的 Region 采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。

其运行流程与 CMS 相似

<img src="https://img-blog.csdnimg.cn/20200730161610159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMjI4MTI4NDk=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:70%;" />

