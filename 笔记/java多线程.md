# 1. 线程的创建

## 创建线程有三种方式

1. **继承Thread类，重写run方法**

   ~~~Java
   public class MyThread1 {
   
       public static void main(String[] args) {
           Thread thread = new Thread(new MyThread3());
           thread.start();
       }
   
       static class MyThread3 extends Thread{
           @Override
           public void run(){
               System.out.println("running Thread");
           }
       }
   }
   ~~~

   

2. **实现Runnable接口，实现run方法**

   ~~~java 
   //不需要导入额外的包
   
   public class MyThread1 {
       
       public static void main(String[] args) {
           //注意，这里是要新建Thread类实例，然后将自己的实现接口的类传进去
           Thread thread = new Thread(new MyThread2());
           //调用start方法
           thread.start();
       }
   	//记一下单词的拼写
       static class MyThread2 implements Runnable{
           //这个方法的修饰符和返回类型记一下
           @Override
           public void run() {
               System.out.println("run2");
           }
       }
   }
   ~~~

3. **实现Callable接口，重写call()方法**

   [用法](https://juejin.im/post/6844904134504611854)
   
   一般情况下是配合ExecutorService来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本：

   ~~~Java
<T> Future<T> submit(Callable<T> task);
   
   ~~~

<T> Future<T> submit(Runnable task, T result);

Future<?> submit(Runnable task);
   ~~~

   Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

   　　Future类位于java.util.concurrent包下，它是一个接口：

   ~~~Java
   <T> Future<T> submit(Callable<T> task);
   
   <T> Future<T> submit(Runnable task, T result);
   
   Future<?> submit(Runnable task);
   ~~~

   ## 也就是说Future提供了三种功能：

   　　1）判断任务是否完成；

   　　2）能够中断任务；

   　　3）能够获取任务执行结果。

   Future用于表示异步计算的结果。它的实现类有java.util.concurrent.FutureTask<V>和 javax.swing.SwingWorker<T,V>。

   使用：

   ~~~Java
   import java.util.Random;
   import java.util.concurrent.Callable;
   import java.util.concurrent.FutureTask;
   
   public class MyCallable {
       public static void main(String[] args) {
           Callable<Integer> callable = new Callable<Integer>(){
               @Override
               public Integer call() throws Exception{
                   return new Random().nextInt(100);
               }
           };
   
           FutureTask<Integer> future = new FutureTask<>(callable);
           new Thread(future).start();
   
           try {
               Thread.sleep(1000);
               Integer integer = future.get();
               System.out.println(integer);
           } catch (Exception e) {
               e.printStackTrace();
           }
       }
   }
   ~~~

   

   **面试常问**

   3方法和4方法的区别

   答：Runnable接口的run方法没有返回值，不能抛出异常；

   ​		Callable接口的call()方法能够抛出异常，且能够得到返回值，通过Future类接收返回值。

   **新的面试问题：**

   继承Thread类与实现Runnable接口的区别？

# 2. volatile与synchronized关键字的内存语义

## 1. volatile

volatile关键字修饰成员变量，就是告知程序任何对该变量的访问均需要从共享内存中获取，而对他的改变必须同步刷新回共享内存，它能保证多线程对变量访问的可见性；

volatile能够保证：

1. 有限原子性
2. 可见性
3. 有序性

volatile可见性的实现就是借助了CPU的lock指令，通过在写volatile的机器指令前加上lock前缀，使写volatile具有以下两个原则：

1. 写volatile时处理器会将缓存写回到主内存。
2. 一个处理器的缓存写回到内存会导致其他处理器的缓存失效。

volatile有序性的实现原理

volatile有序性的保证就是通过禁止指令重排序来实现的。指令重排序包括编译器和处理器重排序，JMM会分别限制这两种指令重排序。

JMM通过加内存屏障来禁止指令重排序，JMM为volatile加内存屏障有以下4种情况：

1. 在每个volatile写操作的前面插入一个StoreStore屏障，防止写volatile与后面的写操作重排序。
2. 在每个volatile写操作的后面插入一个StoreLoad屏障，防止写volatile与后面的读操作重排序。
3. 在每个volatile读操作的后面插入一个LoadLoad屏障，防止读volatile与后面的读操作重排序。
4. 在每个volatile读操作的后面插入一个LoadStore屏障，防止读volatile与后面的写操作重排序。

## 2. synchronized关键字

### 2.1 修饰同步块

修饰同步代码块时，使用了monitorenter和monitorexit指定修饰代码块；

### 2.2 修饰同步方法

修饰同步方法的时候是在方法修饰符上添加ACC_SYNCHRONIZED来完成的。

~~~java 
flags:ACC_PUBLIC,ACC_STATIC,ACC_SYNCHRONIZED
~~~

两种方式的本质都是对一个对象的监视器(monitor)进行获取。

# 3. ThreadLocal

[推荐文章](https://www.jianshu.com/p/6fc3bba12f38)

在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量threadLocals，如下所示

~~~Java
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
~~~

这个threadLocals就是用来存储实际的变量副本的，键值为当前ThreadLocal变量，value为变量副本（即T类型的变量）。

初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

**注意事项：**

1. 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的；

2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；

3. 在进行get之前，必须先set，否则会报空指针异常；如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法。 因为在上面的代码分析过程中，我们发现如果没有先set的话，即在map中查找不到对应的存储，则会通过调用setInitialValue方法返回i，而在setInitialValue方法中，有一个语句是T value = initialValue()， 而默认情况下，initialValue方法返回的是null。

**重点理解：**

虽然使用ThreadLocal.ThreadLocalMap类型的成员变量threadLocals使用得key值是同一个ThreadLocal，但是它使用得map是不同的，即在不同的线程下使用的都是当前线程种的那个ThreadLocalMap。也就是说key相同，但是存储的位置不同。

**实际使用方法：**

~~~java
/**
 * 描述 Java中的ThreadLocal类允许我们创建只能被同一个线程读写的变量。
 * 因此，如果一段代码含有一个ThreadLocal变量的引用，即使两个线程同时执行这段代码，
 * 它们也无法访问到对方的ThreadLocal变量。
 */
public class ThreadLocalExsample {
​
 /**
 * 创建了一个MyRunnable实例，并将该实例作为参数传递给两个线程。两个线程分别执行run()方法，
 * 并且都在ThreadLocal实例上保存了不同的值。如果它们访问的不是ThreadLocal对象并且调用的set()方法被同步了，
 * 则第二个线程会覆盖掉第一个线程设置的值。但是，由于它们访问的是一个ThreadLocal对象，
 * 因此这两个线程都无法看到对方保存的值。也就是说，它们存取的是两个不同的值。
 */
 public static class MyRunnable implements Runnable {
 /**
 * 例化了一个ThreadLocal对象。我们只需要实例化对象一次，并且也不需要知道它是被哪个线程实例化。
 * 虽然所有的线程都能访问到这个ThreadLocal实例，但是每个线程却只能访问到自己通过调用ThreadLocal的
 * set()方法设置的值。即使是两个不同的线程在同一个ThreadLocal对象上设置了不同的值，
 * 他们仍然无法访问到对方的值。
 */
 private ThreadLocal threadLocal = new ThreadLocal();
 @Override
 public void run() {
 //一旦创建了一个ThreadLocal变量，你可以通过如下代码设置某个需要保存的值
 threadLocal.set((int) (Math.random() * 100D));
 try {
 Thread.sleep(2000);
 } catch (InterruptedException e) {
 }
 //可以通过下面方法读取保存在ThreadLocal变量中的值
 System.out.println("-------threadLocal value-------"+threadLocal.get());
 }
 }
​
 public static void main(String[] args) {
 MyRunnable sharedRunnableInstance = new MyRunnable();
 Thread thread1 = new Thread(sharedRunnableInstance);
 Thread thread2 = new Thread(sharedRunnableInstance);
 thread1.start();
 thread2.start();
 }
}
​
运行结果
-------threadLocal value-------38
-------threadLocal value-------88
~~~

**内存泄漏问题**

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法

~~~Java
      static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

~~~





# Java Atomic原子类

并发包 `java.util.concurrent` 的原子类都存放在`java.util.concurrent.atomic`下

- AtomicInteger：整型原子类
- AtomicLong：长整型原子类
- AtomicBoolean ：布尔型原子类

上面三个类提供的方法几乎相同，以 AtomicInteger 为例。

**AtomicInteger 类常用方法**

~~~Java
public final int get() //获取当前的值
public final int getAndSet(int newValue)//获取当前的值，并设置新的值
public final int getAndIncrement()//获取当前的值，并自增
public final int getAndDecrement() //获取当前的值，并自减
public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
boolean compareAndSet(int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）
public final void lazySet(int newValue)//最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
~~~

**AtomicInteger 线程安全原理**

[源码解析](https://www.jianshu.com/p/cea1f9619e8f)

AtomicInteger 类的部分源码：

~~~Java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

     // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用）
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
}
~~~

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址。另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。



根据valueOffset代表的该变量值在内存中的偏移地址，从而获取数据的。

变量value用volatile修饰，保证了多线程之间的内存可见性。



以getAndIncrement为例，说明其原子操作过程

~~~java 
   public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
~~~

~~~Java
    //unsafe.getAndAddInt
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
~~~

- 假设线程1和线程2通过getIntVolatile拿到value的值都为1，线程1被挂起，线程2继续执行

- 线程2在compareAndSwapInt操作中由于预期值和内存值都为1，因此成功将内存值更新为2

- 线程1继续执行，在compareAndSwapInt操作中，预期值是1，而当前的内存值为2，CAS操作失败，什么都不做，返回false

- 线程1重新通过getIntVolatile拿到最新的value为2，再进行一次compareAndSwapInt操作，这次操作成功，内存值更新为3

# 4. 线程间通信

### 1. volatile与synchronized关键字

Java支持多个线程同时访问一个对象的成员变量。

### 2. 等待/通知机制

等待通知机制依赖于所有类的超类Object类中定义的方法。包括：

​	notify()		通知一个在对象上等待的线程，使其从wait()方法返回，而返回的前提是该线程获取到了对象的锁

​	notifyAll()		

​	wait()			线程进入WAITING状态，会释放对象的锁

​	wait(long)

​	wait(long,int)

### 3. 管道输入/输出流

管道输入/输出流只要用于线程之间的数据传输，传输的媒介为内存。

管道输入输出流主要包括如下4种具体实现：

​	PipedOutputStream

​	PipedInputStrean

​	PipedReader

​	PipedWriter

### 4. Thread.join()方法的使用

如果一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止之后才从thread.join()返回。

### 5. ThreadLocal的使用

即线程变量，是以ThreadLocal对象为键、任意对象为值得存储结构。

# 进程间通信（IPC）

[可以看看这个](https://mp.weixin.qq.com/s/iNz6sTen2CSOdLE0j7qu9A)


## 1.匿名管道
管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系

## 2. 命名管道（FIFO）

   命名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信

## 3. 信号量

   信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。

## 4. 消息队列(MessageQueue) 

   消息队列是消息的链表，存放在内核中。一个消息队列由一个标识符（即队列ID）来标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

## 5. 共享内存

   指两个或多个进程共享一个给定的存储区。共享内存是最快的一种 IPC，因为进程是直接对内存进行存取。信号量+共享内存通常结合在一起使用，信号量用来同步对共享内存的访问。

## 6. Socket

   套接字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同机器间的进程通信。

# Java NIO, BIO, AIO

[需要搞懂的概念](https://blog.csdn.net/qq_24693837/article/details/70335491?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

**同步：** 使用同步IO时，Java自己处理IO读写。

**异步：** 使用异步IO时，Java将IO读写委托给OS处理，需要将数据缓冲区地址和大小传给OS，完成后OS通知Java处理（回调）。

**阻塞：** 使用阻塞IO时，Java调用会一直阻塞到读写完成才返回。

**非阻塞：** 使用非阻塞IO时，如果不能立马读写，Java调用会马上返回，当IO事件分发器通知可读写时在进行读写，不断循环直到读写完成。

[比较清晰的解释](https://blog.csdn.net/historyasamirror/article/details/5778378)

[IO](https://mp.weixin.qq.com/s/kXkOl9Km58vlx7PEPDbxBA)

## BIO

同步阻塞IO。**blocking IO**

当在使用**阻塞IO**的时候，应用程序会被无情的**挂起**，等待内核完成操作，因为此时的内核可能将CPU时间切换到了其他需要的进程中，在我们的应用程序看来感觉被卡主(阻塞)了。

BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。

![img](D:/JAVA/JAVA_Stady_Project/简历与笔记/程序员的套路分库/images/阻塞IO.png)

## NIO

同步非阻塞IO。**non-blocking IO**

当使用非阻塞函数的时候，和阻塞IO类比，内核会立即返回，返回后获得足够的CPU时间继续做其他的事情。

NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

![NIO](D:/JAVA/JAVA_Stady_Project/简历与笔记/程序员的套路分库/images/NIO.png)

## AIO

异步IO。

用程序告知内核启动某个操作，并让内核在整个操作（包括将数据从内核拷贝到应用程序的缓冲区）完成后通知应用程序。

AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

![](D:/JAVA/JAVA_Stady_Project/简历与笔记/程序员的套路分库/images/AIO.png)

## IO复用模型

IO多路复用意味着可以将标准输入、套接字等都当做IO的一路，任何一路IO有事件发生，都将通知相应的应用程序去处理相应的IO事件，在我们看来就反复**同时**可以处理多个事情。这就是**IO复用**。

![](D:/JAVA/JAVA_Stady_Project/简历与笔记/程序员的套路分库/images/IO复用.png)

# 5. 线程池

> **线程池、数据库连接池、Http 连接池等等都是对这个思想的应用。池化技术的思想主要是为了减少每次获取资源的消耗，提高对资源的利用率。**

## 线程池的好处

1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. **提高响应速度 。**当任务到达时，任务可以不需要等待线程创建就能立即执行。
3. **提高线程的可管理性。**线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

##  执行 execute()方法和 submit()方法的区别是什么呢？

1. **execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**
2. **submit()方法用于提交需要返回值的任务。线程池会返回一个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执行成功**，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

## 线程池的状态//todo

五种状态：

![](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\线程池的状态.jpg)

1. running

   线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。 线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0

   ~~~java 
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   ~~~

2. shutdown

   (1) 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。 
   (2) 状态切换：调用线程池的shutdown()方法时，线程池由RUNNING -> SHUTDOWN。

3. stop

   (1) 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。 
   (2) 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

4. tidying

   (1) 状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。 
   (2) 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。 
   当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

5. terminated

   (1) 状态说明：线程池彻底终止，就变成TERMINATED状态。 
   (2) 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

## 怎么创建线程池

### 三种线程池

#### **FixedThreadPool** ： 

被称为可重用固定线程数的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。 

**FixedThreadPool 使用无界队列 LinkedBlockingQueue（队列的容量为 Intger.MAX_VALUE）作为线程池的工作队列**

运行中的 `FixedThreadPool`（未执行 `shutdown()`或 `shutdownNow()`）不会拒绝任务，在任务比较多的时候会导致 OOM（内存溢出）。

~~~java 
   /**
     * 创建一个可重用固定数量线程的线程池
     */
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
~~~

**从上面源代码可以看出新创建的 FixedThreadPool 的 corePoolSize 和 maximumPoolSize 都被设置为 nThreads，这个 nThreads 参数是我们使用的时候自己传递的。**

#### **SingleThreadExecutor：**

 方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。 

`SingleThreadExecutor` 使用无界队列 `LinkedBlockingQueue` 作为线程池的工作队列（队列的容量为 Intger.MAX_VALUE）。

允许请求的队列长度为 Integer.MAX_VALUE,可能堆积大量的请求，从而导致 OOM。

~~~java 
   /**
     *返回只有一个线程的线程池
     */
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
~~~

从上面源代码可以看出新创建的 `SingleThreadExecutor` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 1.其他参数和 `FixedThreadPool` 相同。

#### **CachedThreadPool：**

 该方法返回一个可根据实际情况调整线程数量的线程池。

**CachedThreadPool：** 该方法返回一个可根据实际情况调整线程数量的线程池。

`CachedThreadPool` 的`corePoolSize` 被设置为空（0），`maximumPoolSize`被设置为 Integer.MAX.VALUE，即它是无界的，这也就意味着如果主线程提交任务的速度高于 `maximumPool` 中线程处理任务的速度时，`CachedThreadPool` 会不断创建新的线程。极端情况下，这样会导致耗尽 cpu 和内存资源。

允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM。

~~~java 
    /**
     * 创建一个线程池，根据需要创建新线程，但会在先前构建的线程可用时重用它。
     */
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
~~~

## ScheduledThreadPoolExecutor 

**ScheduledThreadPoolExecutor 主要用来在给定的延迟后运行任务，或者定期执行任务。**

## 实例

~~~Java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadPoolExecutorDemo {

    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;
    private static final Long KEEP_ALIVE_TIME = 1L;
    public static void main(String[] args) {

        //使用阿里巴巴推荐的创建线程池的方式
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_TIME,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 10; i++) {
            //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
            Runnable worker = new MyRunnable("" + i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}

~~~



## ThreadPoolExecutor类的分析

[源码分析](http://www.throwable.club/2019/07/15/java-concurrency-thread-pool-executor/)

## 构造方法

~~~java
    /**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,//线程池的核心线程数量
                              int maximumPoolSize,//线程池的最大线程数
                              long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
                              ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
                              RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
                               ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
~~~



## 重要参数

**ThreadPoolExecutor 3 个最重要的参数：**

- **corePoolSize :** 核心线程数线程数定义了最小可以同时运行的线程数量。
- **maximumPoolSize :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **workQueue:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

`ThreadPoolExecutor`其他常见参数:

1. **keepAliveTime**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **unit** : `keepAliveTime` 参数的时间单位。
3. **threadFactory** :executor 创建新线程的时候会用到。
4. **handler** :饱和策略。关于饱和策略下面单独介绍一下。

## 饱和策略，又叫拒绝策略⭐

不一定是拒绝不执行，而是有可能执行，有可能不执行！！！看具体的饱和策略怎么设置的。

- **`ThreadPoolExecutor.AbortPolicy`**：抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：调用执行自己的线程运行任务，也就是直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。因此这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求**任何一个任务请求都要被执行**的话，你可以选择这个策略。
- **ThreadPoolExecutor.DiscardPolicy：** 不处理新任务，直接丢弃掉。
- **ThreadPoolExecutor.DiscardOldestPolicy：** 此策略将丢弃最早的未处理的任务请求。

## 执行流程

![图解线程池实现原理](https://snailclimb.gitee.io/javaguide/docs/java/multi-thread/images/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93/%E5%9B%BE%E8%A7%A3%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86.png)

## (重要)⭐线程池大小的确定

> 上下文切换：
>
> 多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。
>
> 上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。
>
> Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

### **CPU 密集型任务(N+1)**

这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。

### **I/O 密集型任务(2N)**

这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。

**如何判断是 CPU 密集任务还是 IO 密集任务？**

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。单凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。

[美团技术团队对线程池的骚操作](https://mp.weixin.qq.com/s/9HLuPcoWmTqAeFKa1kj-_A)

## 常见对比

### Runnable vs Callable

`Runnable` 接口**不会返回结果或抛出检查异常，但是**`Callable` 接口**可以。**

### execute() vs submit()

上面已经提到，不再赘述。

### shutdown()VSshutdownNow()

- **shutdown（）** :关闭线程池，线程池的状态变为 `SHUTDOWN`。线程池不再接受新任务了，但是队列里的任务得执行完毕。
- **shutdownNow（）** :关闭线程池，线程的状态变为 `STOP`。线程池会终止当前正在运行的任务，并停止处理排队的任务并返回正在等待执行的 List。

### isTerminated() VS isShutdown()

- **isShutDown** 当调用 `shutdown()` 方法后返回为 true。
- **isTerminated** 当调用 `shutdown()` 方法后，并且所有提交的任务完成后返回为 true。

# 锁LOCK

## AQS（队列同步器）

### 简介

AQS 的全称为（AbstractQueuedSynchronizer），这个类在 java.util.concurrent.locks 包下面。

### AQS原理

**AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

~~~Java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
~~~

状态信息通过 protected 类型的`getState`，`setState`，`compareAndSetState`进行操作

~~~Java
//返回同步状态的当前值
protected final int getState() {
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) {
        state = newState;
}
//原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值）
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
~~~

![img](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\AQS)

## CountDownLatch（倒计时器）

**倒计时器**

**作用：**CountDownLatch允许 count 个线程阻塞在一个地方，直至所有线程的任务都执行完毕。（京东面试题：想让线程等待另几个线程执行完毕再继续执行，使用的就是这个，当时不会啊）。

**原理：** CountDownLatch是共享锁的一种实现,它默认构造 AQS 的 state 值为 count。当线程使用countDown方法时,其实使用了`tryReleaseShared`方法以CAS的操作来减少state,直至state为0就代表所有的线程都调用了countDown方法。当调用await方法的时候，如果state不为0，就代表仍然有线程没有调用countDown方法，那么就把已经调用过countDown的线程都放入阻塞队列Park,并自旋CAS判断state == 0，直至最后一个线程调用了countDown，使得state == 0，于是阻塞的线程便判断成功，全部往下执行。

## ReentrantLock （可重入锁）

## ReadWriteLock（读写锁）

[源码解析文章](https://segmentfault.com/a/1190000015768003)

**##############读多写少用读写锁################**

主要实现读共享，写互斥功能，对比单纯的互斥锁在共享资源使用场景为**频繁读取及少量修改**的情况下可以较好的提高性能。

### ReadWriteLock接口

**本质还是基于AQS去实现的。**

ReadWriteLock接口只定义了两个方法：

~~~Java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
~~~

通过调用相应方法获取读锁或写锁，获取的读锁及写锁都是`Lock`接口的实现，可以如同使用`Lock`接口一样使用（其实也有一些特性是不支持的）。

### ReentrantReadWriteLock使用示例

~~~Java
   class RWDictionary {
    private final Map<String, Data> m = new TreeMap<String, Data>();
    private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private final Lock r = rwl.readLock();
    private final Lock w = rwl.writeLock();
 
    public Data get(String key) {
      r.lock();
      try { return m.get(key); }
      finally { r.unlock(); }
    }
    public String[] allKeys() {
      r.lock();
      try { return m.keySet().toArray(); }
      finally { r.unlock(); }
    }
    public Data put(String key, Data value) {
      w.lock();
      try { return m.put(key, value); }
      finally { w.unlock(); }
    }
    public void clear() {
      w.lock();
      try { m.clear(); }
      finally { w.unlock(); }
    }
  }
~~~

![img](D:\JAVA\JAVA_Stady_Project\简历与笔记\程序员的套路分库\images\读写锁)

从图中可见读写锁的加锁解锁操作最终都是调用`ReentrantReadWriteLock`类的内部类`Sync`提供的方法。`Sync`对象通过继承`AbstractQueuedSynchronizer`进行实现。