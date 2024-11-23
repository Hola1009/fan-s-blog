D器优ZZZZOSX锁`等问题导致无法继续执行。

3. 说说什么是进程和线程
进程说简单点就是我们在电脑上启动的一个个应用。进程是操作系统资源分配的最小单位，它包括了程序、数据和进程控制块等。

线程说简单点就是我们在 Java 程序中启动的一个 main 线程，一个进程至少会有一个线程。当然了，我们也可以启动多个线程，比如说一个线程进行 IO 读写，一个线程进行加减乘除计算，这样就可以充分发挥多核 CPU 的优势，因为 IO 读写相对 CPU 计算来说慢得多。线程是 CPU 分配资源的基本单位。
一个进程中可以有多个线程，多个线程共用进程的堆和方法区（Java 虚拟机规范中的一个定义，JDK 8 以后的实现为元空间）资源，但是每个线程都会有自己的程序计数器和栈。

4. 如何理解协程?
协程通常被视为比线程更轻量级的并发单元，它们主要在一些支持异步编程模型的语言中得到了原生支持，如 Kotlin、Go 等。
不过，我们可以使用 CompletableFuture 来模拟协程式的异步执行任务。

5. 说说线程的共享内存
在Java中是采用共享内存来完成线程之间的通信的。那 Java 采用的是共享内存的并发模型。

这个模型被称为 Java 内存模型，也就是 JMM，JMM 决定了一个线程对共享变量的写入何时对另外一个线程可见。

线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了共享变量的副本。当然了，本地内存是 JMM 的一个抽象概念，并不真实存在。
![sdaf](https://cdn.tobebetterjavaer.com/stutymore/javathread-20240315111143.png)
线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：
- 线程 A 把本地内存 A 中的共享变量副本刷新到主内存中。
- 线程 B 到主内存中读取线程 A 刷新过的共享变量，再同步到自己的共享变量副本中。

6. 说说线程有几种创建方式?
Java 中创建线程主要有三种方式，分别为继承 Thread 类、实现 Runnable 接口、实现 Callable 接口。

![二哥的 Java 进阶之路](https://cdn.tobebetterjavaer.com/stutymore/javathread-20240407172652.png)

二哥的 Java 进阶之路

第一种，继承 Thread 类，重写 `run()`方法，调用 `start()`方法启动线程。

```java
class ThreadTask extends Thread {
    public void run() {
        System.out.println("看完二哥的 Java 进阶之路，上岸了!");
    }

    public static void main(String[] args) {
        ThreadTask task = new ThreadTask();
        task.start();
    }
}
```

这种方法的缺点是，由于 Java 不支持多重继承，所以如果类已经继承了另一个类，就不能使用这种方法了。

第二种，实现 Runnable 接口，重写 `run()` 方法，然后创建 Thread 对象，将 Runnable 对象作为参数传递给 Thread 对象，调用 `start()` 方法启动线程。

```java
class RunnableTask implements Runnable {
    public void run() {
        System.out.println("看完二哥的 Java 进阶之路，上岸了!");
    }

    public static void main(String[] args) {
        RunnableTask task = new RunnableTask();
        Thread thread = new Thread(task);
        thread.start();
    }
}
```

这种方法的优点是可以避免 Java 的单继承限制，并且更符合面向对象的编程思想，因为 Runnable 接口将任务代码和线程控制的代码解耦了。

第三种，实现 Callable 接口，重写 `call()` 方法，然后创建 FutureTask 对象，参数为 Callable 对象；紧接着创建 Thread 对象，参数为 FutureTask 对象，调用 `start()` 方法启动线程。

```java
class CallableTask implements Callable<String> {
    public String call() {
        return "看完二哥的 Java 进阶之路，上岸了!";
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallableTask task = new CallableTask();
        FutureTask<String> futureTask = new FutureTask<>(task);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

这种方法的优点是可以获取线程的执行结果。


### part 11 

1. 一个 8G 内存的系统最多能创建多少线程
在确定一个系统最多可以创建多个线程时，除了需要考虑系统的内存大小外，Java 虚拟机栈的大小也是值得考虑的因素。\

默认的 JVM 栈大小是 1024 KB，也就是 1M。
换句话说，8GB = 8 _ 1024 MB = 8 _ 1024 _ 1024 KB，所以一个 8G 内存的系统可以创建的线程数为 8 _ 1024 = 8192 个。

但操作系统本身的运行也需要消耗一定的内存，所以实际上可以创建的线程数肯定会比 8192 少一些。\

2. 启动一个 Java 程序, 你能说说里面有哪些线程吗
首先是 main 线程，这是程序开始执行的入口。
然后是垃圾回收线程，它是一个后台线程，负责回收不再使用的对象
还有编译器线程，在及时编译中（JIT），负责把一部分热点代码编译后放到 codeCache 中，以提升程序的执行效率。

3. 调用 start() 方法会执行 run() 方法, 为什么不直接调用 run() 方法?
在 Java 中，启动一个新的线程应该调用其`start()`方法，而不是直接调用`run()`方法。

当调用`start()`方法时，会启动一个新的线程，并让这个新线程调用`run()`方法。这样，`run()`方法就在新的线程中运行，从而实现多线程并发。

也就是说，`start()` 方法的调用会告诉 JVM 准备好所有必要的新线程结构，分配其所需资源，并调用线程的 `run()` 方法在这个新线程中执行。


4. 线程有哪些常用的调度方法
![三分恶面渣逆袭：线程常用调度方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/javathread-6.png)

5. 说说线程的等待与通知
在 Object 类中有一些方法可以用于线程的等待与通知。
①、`wait()`：当一个线程 A 调用一个共享变量的 `wait()` 方法时，线程 A 会被阻塞挂起，直到发生下面几种情况才会返回 ：    
- 线程 B 调用了`共享对象` `notify()`或者 `notifyAll()` 方法；
- 其他线程调用了线程 A 的 `interrupt()` 方法，线程 A 抛出 InterruptedException 异常返回。
②、`wait(long timeout)` ：这个方法相比 `wait()` 方法多了一个超时参数，它的不同之处在于，如果线程 A 调用共享对象的 `wait(long timeout)`方法后，没有在指定的 timeout 时间内被其它线程唤醒，那么这个方法还是会因为超时而返回。
③、`wait(long timeout, int nanos)`，其内部调用的是 `wait(long timout)` 方法。
唤醒线程主要有下面两个方法：
①、`notify()`：一个线程 A 调用共享对象的 `notify()` 方法后，会唤醒一个在这个共享变量上调用 wait 系列方法后被挂起的线程。
一个共享变量上可能会有多个线程在等待，具体唤醒哪个等待的线程是随机的。
②、`notifyAll()`：不同于在共享变量上调用 `notify()` 方法会唤醒被阻塞到该共享变量上的一个线程，notifyAll 方法会唤醒所有在该共享变量上调用 wait 系列方法而被挂起的线程。 
Thread 类还提供了一个 `join()` 方法，意思是如果一个线程 A 执行了 `thread.join()`，当前线程 A 会等待 thread 线程终止之后才从 `thread.join()` 返回。

6. 说说线程休眠
sleep(long milis): Thread 类中的静态方法, 当一个执行中的线程, 比如说线程A, 调用了Thread 的 sleep 方法, 它就会暂时让出指定时间的执行权
但是线程 A 所拥有的**监视器资源**，比如锁，还是持有不让出的。指定的睡眠时间到了后该方法会正常返回，`接着参与 CPU 的调度`，获取到 CPU 资源后就可以继续运行。

7. 说说线程中断
Java 中的线程中断是一种线程间的协作模式，通过设置线程的中断标志并不能直接终止该线程的执行。被中断的线程会根据中断状态自行处理。

- `void interrupt()` 方法：中断线程，例如，当线程 A 运行时，线程 B 可以调用线程 `interrupt()` 方法来设置线程的中断标志为 true 并立即返回。设置标志仅仅是设置标志, 线程 A 实际并没有被中断，会继续往下执行。
- `boolean isInterrupted()` 方法： 检测当前线程是否被中断。
- `boolean interrupted()` 方法： 检测当前线程是否被中断，与 isInterrupted 不同的是，该方法如果发现当前线程被中断，则会清除中断标志。

stop 方法用来强制线程停止执行，目前已经处于废弃状态，因为 stop 方法会导致线程立即停止，`可能会在不一致的状态下释放锁，破坏对象的一致性`，导致难以发现的错误和资源泄漏。

8. 线程有几种状态
在 Java 中，线程共有 6 种状态：

| 状态           | 说明                                                                                                                                     |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| NEW          | 当线程被创建后，如通过`new Thread()`，它处于新建状态。此时，线程已经被分配了必要的资源，但还没有开始执行。                                                                           |
| RUNNABLE     | 当调用线程的`start()`方法后，线程进入可运行状态。在这个状态下，线程可能正在运行也可能正在等待获取 CPU 时间片，具体取决于线程调度器的调度策略。                                                         |
| BLOCKED      | 线程在试图获取一个锁以进入同步块/方法时，如果锁被其他线程持有，线程将进入阻塞状态，直到它获取到锁。                                                                                     |
| WAITING      | 线程进入等待状态是因为调用了如下方法之一：`Object.wait()`或`LockSupport.park()`。在等待状态下，线程需要其他线程显式地唤醒，否则不会自动执行。                                               |
| TIME_WAITING | 当线程调用带有超时参数的方法时，如`Thread.sleep(long millis)`、`Object.wait(long timeout)` 或`LockSupport.parkNanos()`，它将进入超时等待状态。线程在指定的等待时间过后会自动返回可运行状态。 |
| TERMINATED   | 当线程的`run()`方法执行完毕后，或者因为一个未捕获的异常终止了执行，线程进入终止状态。一旦线程终止，它的生命周期结束，不能再被重新启动。                                                                |
线程在自身的生命周期中，并不是固定地处于某个状态，而 是在不同的状态之间进行切换： 


9. 什么是线程上下文切换
使用多线程的目的是为了充分利用 CPU，但是我们知道，并发其实是一个 CPU 来应付多个线程。
![[Pasted image 20240905071709.png]]

为了让用户感觉多个线程是在同时执行的， CPU 资源的分配采用了时间片轮转也就是给每个线程分配一个时间片，线程在时间片内占用 CPU 执行任务。当线程使用完时间片后，就会处于就绪状态并让出 CPU 让其他线程占用，这就是上下文切换。

10. 线程可以被多核调度吗?
当然可以，在现代操作系统和多核处理器的环境中，线程的调度和管理是操作系统内核的重要职责之一。

操作系统的调度器负责将线程分配给可用的 CPU 核心，从而实现并行处理。

多核处理器提供了并行执行多个线程的能力。每个核心可以独立执行一个或多个线程，*操作系统的任务调度器会根据策略和算法，如优先级调度、轮转调度等，决定哪个线程何时在哪个核心上运行*。

### part 12
1. 守护线程了解吗
Java 中的线程分为两类，分别为 daemon 线程（守护线程）和 user 线程（用户线程）。

在 JVM 启动时会调用 main 方法，main 方法所在的线程就是一个用户线程。其实在 JVM 内部同时还启动了很多守护线程， 比如垃圾回收线程。

那么守护线程和用户线程有什么区别呢？区别之一是当最后一个非守护线程束时， JVM 会正常退出，而不管当前是否存在守护线程，也就是说守护线程是否结束并不影响 JVM 退出。换而言之，只要有一个用户线程还没结束，正常情况下 JVM 就不会退出。 

2. 线程间有哪些通信方式?
线程之间传递信息有多种方式，每种方式适用于不同的场景。比如说使用共享对象、`wait()` 和 `notify()`、Exchanger 和 CompletableFuture。

①、**使用共享对象**，多个线程可以访问和修改同一个对象，从而实现信息的传递，比如说 volatile 和 synchronized 关键字。

[关键字 volatile](https://javabetter.cn/thread/volatile.html) 用来修饰成员变量，告知程序任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，保证所有线程对变量访问的可见性。

[关键字 synchronized](https://javabetter.cn/thread/synchronized-1.html) 可以修饰方法，或者以同步代码块的形式来使用，确保多个线程在同一个时刻，只能有一个线程在执行某个方法或某个代码块。

②、**使用 wait() 和 notify()**，例如，生产者-消费者模式中，生产者生产数据，消费者消费数据，通过 `wait()` 和 `notify()` 方法可以实现生产和消费的协调。

一个线程调用共享对象的 `wait()` 方法时，它会进入该对象的等待池，并释放已经持有的该对象的锁，进入等待状态，直到其他线程调用相同对象的 `notify()` 或 `notifyAll()` 方法。

一个线程调用共享对象的 `notify()` 方法时，它会唤醒在该对象等待池中等待的一个线程，使其进入锁池，等待获取锁。

[Condition](https://javabetter.cn/thread/condition.html) 也提供了类似的方法，`await()` 负责等待、`signal()` 和 `signalAll()` 负责通知。

通常与锁（特别是 [ReentrantLock](https://javabetter.cn/thread/reentrantLock.html)）一起使用，为线程提供了一种等待某个条件成真的机制，并允许其他线程在该条件变化时通知等待线程。更灵活、更强大。

③、**使用 Exchanger**，Exchanger 是一个同步点，可以在两个线程之间交换数据。一个线程调用 exchange() 方法，将数据传递给另一个线程，同时接收另一个线程的数据。

3. 请说说 sleep 和 wait 的区别
答：`sleep()` 和 `wait()` 是 Java 中用于暂停当前线程的两个重要方法，sleep 是让当前线程休眠，不涉及对象类，也不需要获取对象的锁，属于 Thread 类的方法；wait 是让获得对象锁的线程实现等待，前提要获得对象的锁，属于 Object 类的方法。

它们之间的区别主要有以下几点：

①、所属类不同

- `sleep()` 方法专属于 `Thread` 类。
- `wait()` 方法专属于 `Object` 类。

②、锁行为不同

当线程执行 sleep 方法时，它不会释放任何锁。也就是说，如果一个线程在持有某个对象的锁时调用了 sleep，它在睡眠期间仍然会持有这个锁。
③、使用条件不同

- `sleep()` 方法可以在任何地方被调用。
- `wait()` 方法必须在同步代码块或同步方法中被调用，这是因为调用 `wait()` 方法的前提是当前线程必须持有对象的锁。否则会抛出 `IllegalMonitorStateException` 异常。
④、唤醒方式不同

- `sleep()` 方法在指定的时间过后，线程会自动唤醒继续执行。
- `wait()` 方法需要依靠 `notify()`、`notifyAll()` 方法或者 `wait()` 方法中指定的等待时间到期来唤醒线程。

⑤、抛出异常不同

- `sleep()` 方法在等待期间，如果线程被中断，会抛出 `InterruptedException`。
- 如果线程被中断或等待时间到期时，`wait()` 方法同样会在等待期间抛出 `InterruptedException`。

我们来通过代码再感受一下 `sleep()` 和 `wait()` 在用法上的区别，先看 `sleep()` 的用法：

4. 说一个线程安全的使用场景
线程安全是 Java 并发编程中一个非常重要的概念，它指的是多线程环境下，多个线程对共享资源的访问不会导致数据的不一致性。

一个常见的使用场景是在实现单例模式时确保线程安全。

单例模式确保一个类只有一个实例，并提供一个全局访问点。在多线程环境下，如果多个线程同时尝试创建实例，单例类必须确保只创建一个实例。

饿汉式是一种比较直接的实现方式，它通过在类加载时就立即初始化单例对象来保证线程安全。

懒汉式单例则在第一次使用时初始化，这种方式需要使用双重检查锁定来确保线程安全，volatile 用来保证可见性，syncronized 用来保证同步。

```java
public class LazySingleton {
    private static volatile LazySingleton instance;

    private LazySingleton() {}

    public static LazySingleton getInstance() {
        if (instance == null) { // 第一次检查
            synchronized (LazySingleton.class) {
                if (instance == null) { // 第二次检查
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

5. ThreadLocal 是什么
[ThreadLocal](https://javabetter.cn/thread/ThreadLocal.html) 是 Java 中提供的一种用于实现线程局部变量的工具类。它允许每个线程都拥有自己的独立副本，从而实现线程隔离，用于解决多线程中共享对象的线程安全问题。


### part 13

1. 除了 ThreadLocal, 还有什么解决线程安全问题的方法
①、Java 中的 synchronized 关键字可以用于方法和代码块，确保同一时间只有一个线程可以执行特定的代码段。

②、Java 并发包（java.util.concurrent.locks）中提供了 Lock 接口和一些实现类，如 ReentrantLock。相比于 synchronized，ReentrantLock 提供了公平锁和非公平锁。

③、Java 并发包还提供了一组原子变量类（如 AtomicInteger，AtomicLong 等），它们利用 CAS（比较并交换），实现了无锁的原子操作，适用于简单的计数器场景。

④、Java 并发包提供了一些线程安全的集合类，如 ConcurrentHashMap，CopyOnWriteArrayList 等。这些集合类内部实现了必要的同步策略，提供了更高效的并发访问

⑤、volatile 变量保证了变量的可见性，修改操作是立即同步到主存的，读操作从主存中读取。


2. 在开发中用到过 ThreadLocal 吗?
有, 我用它来存储用户信息, 因为在服务层和持久层要用到用户信息,  我就写了个过滤器, 在请求过滤的时候, 将 用户信息存入ThreadLocal

这样我们在任何一个地方，都可以取出 ThreadLocal 中存的用户信息。

3. ThreadLocal 是怎么实现的
ThreadLocal 的实现原理就是，每个线程维护一个 Map，key 为 ThreadLocal 对象，value 为想要实现线程隔离的对象。
1、当需要存线程隔离的对象时，通过 ThreadLocal 的 set 方法将对象存入 Map 中。

2、当需要取线程隔离的对象时，通过 ThreadLocal 的 get 方法从 Map 中取出对象。

3、Map 的大小由 ThreadLocal 对象的多少决定。

#todo 
4. 什么是弱引用, 什么是强引用

5. ThreadLocal 内存泄露是怎么回事
通常情况下，随着线程 Thread 的结束，其内部的 ThreadLocalMap 也会被回收，从而避免了内存泄漏。

但如果一个线程一直在运行，并且其 `ThreadLocalMap` 中的 Entry.value 一直指向某个强引用对象，那么这个对象就不会被回收，从而导致内存泄漏。当 Entry 非常多时，可能就会引发更严重的内存溢出问题。\

6. 怎么解决内存泄漏问题
很简单，使用完 ThreadLocal 后，及时调用 `remove()` 方法释放内存空间。

```java
try {
    threadLocal.set(value);
    // 执行业务操作
} finally {
    threadLocal.remove(); // 确保能够执行清理
}
```

`remove()` 方法会将当前线程的 ThreadLocalMap 中的所有 key 为 null 的 Entry 全部清除，这样就能避免内存泄漏问题。

7. 为什么要设计成弱引用
弱引用的好处是，当内存不足的时候，JVM 会主动回收掉弱引用的对象。

一旦 key 被回收，ThreadLocalMap 在进行 set、get 的时候就会对 key 为 null 的 Entry 进行清理。

#todo  
8. 你了解哪些 ThreadLocal 的改进方案

9. ThreadLocalMap 的源码看过吗?
ThreadLocalMap 虽然被叫做 Map，其实它是没有实现 Map 接口的，但是结构还是和 HashMap 比较类似的，主要关注的是两个要素：`元素数组`和`散列方法`。

- 元素数组

一个 table 数组，存储 Entry 类型的元素，Entry 是 ThreaLocal 弱引用作为 key，Object 作为 value 的结构

- 散列方法
    
    散列方法就是怎么把对应的 key 映射到 table 数组的相应下标，ThreadLocalMap 用的是哈希取余法，取出 key 的 threadLocalHashCode，然后和 table 数组长度减一&运算（相当于取余）。

10. ThreadLocalMap 怎么解决 Hash 冲突的
ThreadLocalMap 没有使用链表，自然也不是用链地址法来解决冲突了，它用的是另外一种方式——**开放定址法**。开放定址法是什么意思呢？简单来说，就是这个坑被人占了，那就接着去找空着的坑。


11. ThreadLocal 的扩容机制了解吗?
 ThreadLocalMap.set()方法的最后，如果执行完启发式清理工作后，未清理到任何数据，且当前散列数组中`Entry`的数量已经达到了列表的扩容阈值`(len*2/3)`，就开始执行`rehash()`逻辑：

```java
if (!cleanSomeSlots(i, sz) && sz >= threshold)
    rehash();
```

再着看 rehash()具体实现：这里会先去清理过期的 Entry，然后还要根据条件判断`size >= threshold - threshold / 4` 也就是`size >= threshold* 3/4`来决定是否需要扩容。

```java
private void rehash() {
    //清理过期Entry
    expungeStaleEntries();

    //扩容
    if (size >= threshold - threshold / 4)
        resize();
}

//清理过期Entry
private void expungeStaleEntries() {
    Entry[] tab = table;
    int len = tab.length;
    for (int j = 0; j < len; j++) {
        Entry e = tab[j];
        if (e != null && e.get() == null)
            expungeStaleEntry(j);
    }
}
```

接着看看具体的`resize()`方法，扩容后的`newTab`的大小为老数组的两倍，然后遍历老的 table 数组，散列方法重新计算位置，开放地址解决冲突，然后放到新的`newTab`，遍历完成之后，`oldTab`中所有的`entry`数据都已经放入到`newTab`中了，然后 table 引用指向`newTab`

19. 父子线程怎么共享数据?
父线程能用 ThreadLocal 来给子线程传值吗？毫无疑问，不能。那该怎么办？
这时候可以用到另外一个类——`InheritableThreadLocal` 。
使用起来很简单，在主线程的 InheritableThreadLocal 实例设置值，在子线程中就可以拿到了

原理很简单，在 Thread 类里还有另外一个变量：
```java
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```
在 Thread.init 的时候，如果父线程的`inheritableThreadLocals`不为空，就把它赋给当前线程（子线程）的`inheritableThreadLocals` 。

```java
if (inheritThreadLocals && parent.inheritableThreadLocals != null)
    this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```

### part 13
1. 说说你对 java 内存模型的理解
Java 内存模型（Java Memory Model）是一种抽象的模型，简称 JMM，主要用来定义多线程中变量的访问规则，用来解决变量的可见性、有序性和原子性问题，确保在并发环境中安全地访问共享变量。 

JMM 定义了线程内存和主内存之间的抽象关系：线程之间的共享变量存储在`主内存`（Main Memory）中，每个线程都有一个私有的`本地内存`（Local Memory），本地内存中存储了共享变量的副本，用来进行线程内部的读写操作。

- 当一个线程更改了本地内存中共享变量的副本后，它需要将这些更改刷新到主内存中，以确保其他线程可以看到这些更改。
- 当一个线程需要读取共享变量时，它可能首先从本地内存中读取。如果本地内存中的副本是过时的，线程将从主内存中重新加载共享变量的最新值到本地内存中。

本地内存是 JMM 中的一个抽象概念，并不真实存在。实际上，本地内存可能对应于 CPU 缓存、寄存器或者其他硬件和编译器优化。

2. 为什么线程要有自己的内存
在并发的环境中, 如果每个线程都直接操作主存中的共享变量, 就会引发更多的内存访问竞争, 影响性能那个 还增加了线程安全问题. 

现代的cup为了优化执行效率, 可能会对指令进行重排序, 使用本地内存, 可以在不影响最终执行结果的前提下, 可以让cup重排序的自由度更高, 提高性能,.

3. 说说你对原子性, 可见性, 有序性的理解?
原子性, 有序性, 可见性是并发编程中非常重要的基础概念, JMM 的很多技术都是围绕着着三大特性展开.
原子性: 指的是一个操作是不可分割的, 这个操作里面的指令要么全部执行, 要么全部不执行
可见性: 可见性指的是一个线程修改了一个共享变量的值时, 其他线程可以马上看到
有序性: 指的是对于一个线程执行代码, 是从前往后依次执行的, 单线程下是有序的, 但并发时可能会发生指令重排.

- 原子性：JMM 只能保证基本的原子性，如果要保证一个代码块的原子性，需要使用`synchronized` 。
- 可见性：Java 是利用`volatile`关键字来保证可见性的，除此之外，`final`和`synchronized`也能保证可见性。
- 有序性：`synchronized`或者`volatile`都可以保证多线程之间操作的有序性。

4. 那说说什么是指令重排
在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分 3 种类型。
- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应 机器指令的执行顺序。
- 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

5. 指令重排序有限制吗? happens-before 了解吗?

6. as-if-serial 又是什么? 单线程的程序一定是顺序执行的吗
as-if-serial 语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），**单线程程序的执行结果不能被改变**。编译器、runtime 和处理器都必须遵守 as-if-serial 语义。

为了遵守 as-if-serial 语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序

7. volatile 怎么保证有序性

### part 14
1. synchronized 用过吗? 怎么使用
在 Java 中，synchronized 是最常用的锁，它使用简单，并且可以保证线程安全，它可以加在方法或是代码块前来避免多线程并发访问时出现数据不一致的情况。

2. synchronized 是怎么加锁的
synchronized 是 JVM 帮我们实现的，因此在使用的时候不用手动去 lock 和 unlock，JVM 会帮我们自动加锁和解锁。

3. synchronized 锁住的是什么?


