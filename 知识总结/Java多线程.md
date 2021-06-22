# Java 多线程

Java 有关多线程的相关知识总结。

## 相关概念

### 1. 基础

#### 1.1 进程与线程

线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反。

他们两个本质的区别是**是否单独占有内存地址空间及其它系统资源（比如I/O）**

#### 1.2 程序计数器、虚拟机栈、本地方法栈为什么是线程私有的？

程序计数器私有主要是为了**线程切换后能恢复到正确的执行位置**；

为了**保证线程中的局部变量不被别的线程访问到**，虚拟机栈和本地方法栈是线程私有的。

#### 1.3 并发与并行

- **并发：** 同一时间段，多个任务都在执行 (单位时间内不一定同时执行)；
- **并行：** 单位时间内，多个任务同时执行。

### 2. 线程的生命周期和状态

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png" alt="Java 线程的状态 " style="zoom: 80%;" />

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java+%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png" alt="Java 线程状态变迁 " style="zoom:80%;" />

- 线程创建之后它将处于 **NEW（新建）** 状态，调用 `start()` 方法后开始运行，线程这时候处于 **READY（可运行）** 状态。可运行状态的线程获得了 CPU 时间片（timeslice）后就处于 **RUNNING（运行）** 状态。
- 当线程执行 `wait()`方法之后，线程进入 **WAITING（等待）** 状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而 **TIME_WAITING（超时等待）** 状态相当于在等待状态的基础上增加了超时限制，比如通过 `sleep（long millis）`方法或 `wait（long millis）`方法可以将 Java 线程置于 TIMED WAITING 状态。当超时时间到达后 Java 线程将会返回到 RUNNABLE 状态。
- 当线程调用同步方法时，在没有获取到锁的情况下，线程将会进入到 **BLOCKED（阻塞）** 状态。线程在执行 Runnable 的`run()`方法之后将会进入到 **TERMINATED（终止）** 状态。

操作系统隐藏 Java 虚拟机（JVM）中的 READY 和 RUNNING 状态，它只能看到 RUNNABLE 状态，所以 Java 系统一般将这两个状态统称为 **RUNNABLE（运行中）** 状态 。

#### 2.1 sleep() 方法和 wait() 方法区别和共同点

这两个方法都能使线程暂停执行。

|      | `sleep()` | `wait()` |
| ---- | --------- | -------- |
| 锁 | 不释放锁 |释放锁|
| 用途 | 暂停执行 | 线程间交互/通信 |
| 苏醒 | `sleep()`执行完线程自动苏醒 | 需要别的线程调用`notify()`或`notifyAll()`方法使其苏醒，或者使用`wait(long timeout)`使其超时后自动苏醒 |

#### 2.2 调用 start() 方法时会执行 run() 方法，为什么不能直接调用 run() 方法？

new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

### 3. 多线程中可能出现的问题

#### 3.1 上下文切换

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。

概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。

上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

#### 3.2 死锁

##### 3.2.1 死锁产生条件

1. 互斥条件：该资源任意一个时刻只由一个线程占用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4. 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

例子

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

执行结果

```java
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

##### 3.2.2 如何避免死锁

1. 破坏互斥条件 ：这个条件我们没有办法破坏，因为我们用锁本来就是想让他们互斥的（临界资源需要互斥访问）。
2. 破坏请求与保持条件 ：一次性申请所有的资源。
3. 破坏不剥夺条件 ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
4. 破坏循环等待条件 ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

为了破坏上面例子中的死锁，可以将 线程2 改为首先申请`resource2`

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        // 修改线程2，使其资源申请顺序与线程1 相同
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 2").start();
    }
}
```

输出

```java
Thread[线程 1,5,main]get resource1
Thread[线程 1,5,main]waiting get resource2
Thread[线程 1,5,main]get resource2
Thread[线程 2,5,main]get resource1
Thread[线程 2,5,main]waiting get resource2
Thread[线程 2,5,main]get resource2

Process finished with exit code 0
```

### 4. 锁

<img src="https://img-blog.csdnimg.cn/20190514175454461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDc0MzI2MQ==,size_16,color_FFFFFF,t_70" alt="锁分类" style="zoom: 80%;" />

#### 4.1 悲观锁/乐观锁

- 悲观锁：我们假设在多线程使用同一资源时会互相抢占资源，这种态度引起的措施叫悲观锁。悲观锁一般用`synchronized`或者`Lock`来加锁。
- 乐观锁：在使用资源时认为其他资源不会抢占资源，就不用使用加锁的方式，这就是乐观锁，一般使用 CAS 算法处理。

#### 4.2 互斥锁/自旋锁

自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。

- 互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。
- 但是**自旋锁不会引起调用者睡眠**，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，“自旋”一词就是因此而得名。

自旋锁在内核中大量应用于中断处理等部分（对于单处理器来说，防止中断处理中的并发可简单采用关闭中断的方式，即在标志寄存器中关闭/打开中断标志位，不需要自旋锁）。

自旋锁的初衷就是：在短期间内进行轻量级的锁定。一个被争用的自旋锁使得请求它的线程在等待锁重新可用的期间进行自旋（特别浪费处理器时间），所以自旋锁不应该被持有时间过长。如果需要长时间锁定的话，最好使用信号量。

自旋锁只有在内核可抢占或 SMP（多处理器）的情况下才真正需要，在单CPU且不可抢占的内核下，自旋锁的所有操作都是空操作。

#### 4.3 公平锁/非公平锁

- 公平锁是指多个线程按照申请锁的顺序来获取锁。
- 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。

对于`ReentrantLock`而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

对于`Synchronized`而言，也是一种非公平锁。由于其并不像`ReentrantLock`是通过 AQS 的来实现线程调度，所以并没有任何办法使其变成公平锁。

#### 4.4 可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。

```java
synchronized void setA() throws Exception{
    Thread.sleep(1000);
    setB();
}

synchronized void setB() throws Exception{
    Thread.sleep(1000);
}
```

上面的代码就是一个可重入锁的一个特点，如果不是可重入锁的话，`setB()`可能不会被当前线程执行，可能造成死锁。

#### 4.5 排他锁/共享锁

- 排他锁是指该锁一次只能被一个线程所持有。
- 共享锁是指该锁可被多个线程所持有。

对于`ReentrantLock`而言，其是排他锁。但是对于`Lock`的另一个实现类`ReadWriteLock`，其读锁是共享锁，其写锁是排他锁。

读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。

排他锁与共享锁也是通过 AQS 来实现的，通过实现不同的方法，来实现独享或者共享。

对于`Synchronized`而言，当然是排他锁。

#### 4.6 偏向锁/轻量级锁/重量级锁（锁升级）

这三种锁是指锁的状态，并且是针对`Synchronized`。在 Java 5 通过引入锁升级的机制来实现高效`Synchronized`。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。

- 偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。大白话就是对锁置个变量，如果发现为true，代表资源无竞争，则**无需再走各种加锁/解锁流程**。如果为false，代表存在其他线程竞争资源，那么就会走后面的流程。
- 轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
- 重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。

## 实际操作

### 1. 线程创建

#### 1.1 继承 Theard 类并重写 run 方法

```java
public class ThreadTest {
    // 继承Thread类并重写run方法
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("I am a child Thread");
        }
    }
 
    public static void main(String[] args) {
        // 创建线程
        MyThread thread = new MyThread();
        // 启动线程
        thread.start();
    }
}
```

- 优点：可以直接在 run 方法中使用 this 获取当前线程，不需要使用 Thread.currentThread() 方法。
- 缺点：不支持多继承；任务与代码没有分离，多个线程执行相同任务时需要多份代码；没有返回值。

#### 1.2 实现 Runnable 接口的 run 方法

```java

public class ThreadTest {
    public static class RunnableTask implements Runnable {
        @Override
        public void run() {
            System.out.println("I am a child Thread implements Runnable");
        }
    }
 
    public static void main(String[] args) {
        RunnableTask task = new RunnableTask();
        new Thread(task).start();
        new Thread(task).start();
    }
}
```

- 优点：如代码所示，可以共用 task 代码逻辑。
- 缺点：没有返回值。

#### 1.3 FutureTask

```java
// 创建任务类，类似Runnable
public class CallerTask implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "hello";
    }
 
    public static void main(String[] args) {
        // 创建异步任务
        FutureTask<String> futureTask = new FutureTask<>(new CallerTask());
        // 启动线程
        new Thread(futureTask).start();
        try {
            // 等待任务执行完毕，并返回结果
            String result = futureTask.get();
            System.out.println(result);
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

优点：可以拿到返回值。

### 2. TheadLocal

```java
public class ThreadLocalTest {
    // (1) print函数
    static void print(String str) {
        // 1.1 打印当前线程本地内存中的localVariable变量的值
        System.out.println(str + ":" + localVariable.get());
        // 1.2 清除当前线程本地内存中的localVariable变量
        localVariable.remove();
    }
    // (2) 创建ThreadLocal变量
    static ThreadLocal<String> localVariable = new ThreadLocal<>();
 
    public static void main(String[] args) {
        // (3) 创建线程one
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                // 3.1 设置线程One中本地变量localVariable的值
                localVariable.set("threadOne local variable");
                // 3.2 调用打印函数
                print("threadOne");
                // 3.3 打印本地变量值
                System.out.println("threadOne remove after" + ":" + localVariable.get());
            }
        });
        // (4) 创建线程two
        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                // 4.1 设置线程Two中本地变量localVariable的值
                localVariable.set("threadTwo local variable");
                // 4.2 调用打印函数
                print("threadTwo");
                // 4.3 打印本地变量值
                System.out.println("threadTwo remove after" + ":" + localVariable.get());
            }
        });
        // (5) 启动线程
        threadOne.start();
        threadTwo.start();
 
    }
}
```

执行结果：

```
threadOne:threadOne local variable
threadOne remove after:null
threadTwo:threadTwo local variable
threadTwo remove after:null
```



#### 2.1 原理

创建了一个 ThreadLocal 变量后，访问这个变量的每个线程都会有个这个变量的本地副本。实现方式是在每个线程内部有个名为 threadLocals 的 HashMap，其中 key 为我们定义的 ThreadLocal 变量的 this 引用，value 则为我们使用 set 方法设置的值。

<img src="https://gitee.com/wtychn/ImageBed/raw/master/image-20210524160356061.png" alt="image-20210524160356061" style="zoom:50%;" />

#### 2.2 InheritableThreadLocal

使子线程可以使用父线程中设置的本地变量。

### 3. synchronized

会导致上下文切换，导致锁十分笨重。

```java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    Object o = new Object();
    synchronized (o) {
        // code
    }
}
```

synchronized 的内存语义为：

- 进入 synchronized 块时：把块内要用到的变量从线程的工作内存中清除，这样在 synchronized 块内使用到该变量时就不会从线程工作内存中获取，而是从主线程中获取。
- 离开 synchronized 块时：把块内的共享变量修改刷新到主内存。

### 4. volatile

```java
public class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1; // step 1
        flag = true; // step 2
    }

    public void reader() {
        if (flag) { // step 3
            System.out.println(a); // step 4
        }
    }
}
```

在Java中，volatile关键字有特殊的内存语义。volatile主要有以下两个功能：

- 保证变量的**内存可见性**，线程在写入变量时不会把值缓存在寄存器或者其他地方，而是会把值刷新会主内存。
- 禁止volatile变量与普通变量**重排序**（JSR133提出，Java 5 开始才有这个“增强的volatile内存语义”）

volatile 与锁不同的是他不保证操作的原子性，使用 volatile 的场景有：

- 写入变量不依赖变量的当前值时。
- 读写变量值时没有加锁。

### 5. CAS

```java
boolean compareAndSwapLong(Object obj, long offset, long expect, long update)
```

比较对象 obj 中的偏移量为 offset 的变量的值是否与 expect 相等，相等则使用 update 值更新，然后返回 true，否则返回 true。

### 6. 线程池 ThreadPoolExecutor

[图解 | 你管这破玩意叫线程池？](https://mp.weixin.qq.com/s?__biz=Mzk0MjE3NDE0Ng==&mid=2247491549&idx=1&sn=1d5728754e8c06a621bbdca336d85452&chksm=c2c66570f5b1ec66df623e5300084257bd943b134d34e16abaacdb58834702dbbc4599868b89&scene=21#wechat_redirect)这文章写的很好啊，大佬的解释是真的清晰。

<img src="https://mmbiz.qpic.cn/mmbiz_png/GLeh42uInXTjxNaj1t0BwIBXIG4UCkqlNOR6EbX3EVhyfp7fSy80IweianoAMNR6fHWicCpU9f9iaEickMyDls6BZQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom: 75%;" />

说明一下 ThreadPoolExecutor 的一众参数接口到底是啥意思

1. **execute 方法**：启动线程池，将多线程任务添加到任务队列中；
2. **BlockingQueue**：任务队列，线程安全的阻塞对列；
3. **Worker 线程**：执行任务队列中任务的核心线程；
4. **corePoolSize**：Worker 核心线程的数量；
5. **RejectedExecutionHandler 接口**：由调用者决定实现类，以便在任务提交失败后执行 rejectedExecution 方法；
6. **ThreadFactory 接口**：增加工作线程时不再直接 new 线程，而是调用这个由调用者传入的 ThreadFactory 实现类的 newThread 方法；
7. **workCount**：刚初始化线程池时，不再立刻创建 corePoolSize 个工作线程，而是等待调用者不断提交任务的过程中，逐渐把工作线程 Worker 创建出来，等数量达到 corePoolSize 时就停止，把任务直接丢到队列里。workCount 用来记录已经有的 Worker 线程数量；
8. **maximumPoolSize**：当核心线程数和队列都满了时，新提交的任务仍然可以通过创建新的工作线程（叫它**非核心线程**），直到工作线程数达到 maximumPoolSize 为止，这样就可以缓解一时的高峰期了，而用户也不用设置过大的核心线程数；
9. **keepAliveTime**：**非核心线程**超时时间，当这么长时间没能从队列里获取任务时，就不再等了，销毁线程。

