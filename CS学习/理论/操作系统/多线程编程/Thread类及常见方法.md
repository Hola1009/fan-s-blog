Thread类是JVM用来管理线程的一个类, 换句话说, 每一个线程都有唯一的Thread对象与之关联
用之前的例子看, 每个执行流, 也需要有一个对象来描述, 每个Thread类对象就是用来描述应给线程的执行流的, jvm会将这些Thread对象组织起来, 用于线程调度, 线程管理.

# Thread的常见构造方法
| 方法 | 说明 |
| ---- | ---- |
| Thread() | 创建线程对象 |
| Thread(Runnable target) | 使用Runnable对象创建线程实例 |
| Thread(String name) | 创建线程对象, 并命名 |
| Thread(Runnable target, String name) | 使用Runnable对象创建实例, 并命名 |
|  |  |
# Thread的几个常见属性
| 属性 | 获取方法 |
| ---- | ---- |
| ID | getId() |
| 名称 | getName() |
| 状态 | getState() |
| 优先级 | getPriority |
| 是否后有台线程 | isDaemon() |
| 是否存活 | isAlive() |
| 是否中断 | isInterrupted() |
- id是线程唯一的标识, 不同线程不会重复
- 名称是个各种调试工具用到的
- 状态表示线程当处的一个情况
- 优先级高的线程理论上更容易被调度到
- 关羽后台线程, 需要记住, ==JVM会在一个进程的所有非后台线程结束后, 才会结束运行==
- 是否存活, 即run方法是否运行结束
```java
package demo.ThreadInterrupt;  
  
public class Demo {  
    public static void main(String[] args) {  
        Thread thread = new Thread(() -> {  
            for (int i = 0; i < 10; i++) {  
  
                try {  
                    System.out.println(Thread.currentThread().getName() + " still live");  
                    Thread.sleep(10);  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
            System.out.println(Thread.currentThread().getName() + " will died");  
        });  
        System.out.println(Thread.currentThread().getName() + ": ID: " + thread.getId());  
        System.out.println(Thread.currentThread().getName() + ": Name: " + thread.getName());  
        System.out.println(Thread.currentThread().getName() + ": State: " + thread.getState());  
        System.out.println(Thread.currentThread().getName() + ": State: " + thread.getState());  
        System.out.println(Thread.currentThread().getName() + ": priority: " + thread.getPriority());  
        System.out.println(Thread.currentThread().getName() + ": isDaemon: " + thread.isDaemon());  
        System.out.println(Thread.currentThread().getName() + ": isInterrupted: " + thread.isInterrupted());  
        thread.start();  
        while (thread.isAlive()) {  
            System.out.println(Thread.currentThread().getName() + ": 状态: " + thread.getState());  
        }  
    }  
}
```

```
main: ID: 20
main: Name: Thread-0
main: State: NEW
main: State: NEW
main: priority: 5
main: isDaemon: false
main: isInterrupted: false
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
main: 状态: RUNNABLE
Thread-0 still live
main: 状态: BLOCKED
```

# 启动一个线程 - start()
我们已经了解了如何通过重写run方法来创建一个线程对象, 但创建了线程对象并不意味着线程就开始运行了
- 重写run方法即提供给线程要做的事情的指令清单
- 线程对象可以认为是把张三, 李四 叫过来
- 二调用start()方法, 就是让被叫过来的张三, 李四 动起来
![[start方法|left|500]]
> [!summary] 
> 调用start方法, 才真的在操作系统的底层创建出一个线程

# 中断一个线程
张三一旦进入到工作状态, 他就会按照行动指南上的步骤进行工作, 不完成是不会结束的. 但有时我们需要增加一些机制, 例如上头突然来电话, 说那两小伙不是这工地的, 就需要赶紧停止搬砖, 那包工头如何通知张三停止呢, 这就涉及到我们的停止线程的方式了
目前常见的有一下两种方式:
1. 通过共享的标记来进行沟通
2. 通过interrupt()方法来通知

> [!example] 示例1
> 使用自定义的变量来作为标志位

- 需要给标志位加volatile关键字
```java
public class Demo1 {  
    private static class MyRunnable implements Runnable {  
        public volatile boolean isQuit = false;  
  
        @Override  
        public void run() {  
            while (!isQuit) {  
                System.out.println(Thread.currentThread().getName() + ":在搬砖呢...");  
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
            System.out.println("啊!!!要跑路...");  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        MyRunnable myRunnable = new MyRunnable();  
        Thread thread = new Thread(myRunnable, "张三");  
        thread.start();  
        Thread.sleep(3000);  
        System.out.println("包工头接到通知");  
        System.out.println("那两精神小伙, 别搬砖");  
        myRunnable.isQuit = true;  
    }  
}
```
运行结构:
```
张三:在搬砖呢...
张三:在搬砖呢...
张三:在搬砖呢...
包工头接到通知
张三:在搬砖呢...
...
...
张三:在搬砖呢...
...
那两精神小伙, 别搬砖
张三:啊!!!要跑路...

```
> [!example] 示例2 
> 使用Thread.interrupted() 或者 Thread.currentThread().isInterrupted()代替自定义标志位

> Thread 内部包含了应给boolean类型的变量作为线程是否被中断的标记

| 方法 | 说明 |
| ---- | ---- |
| interrupt() | 中断对象关联的线程, 如果线程正在阻塞, 则以异常方式通知, 否之设置标志位 |
| interrupted() | 判断当前线程的中断标志是否被设置, 调用后清楚标志位 |
| isInterrupted() | 判断对象关联的线程的标志位是否设置, 调用后不清楚标志位 |
- 适用thread对象的interrupted()方法通知线程结束
```java
public class Demo2 {  
    private static class MyRunnable implements Runnable {  
  
        @Override  
        public void run() {  
//            Thread.currentThread().isInterrupted();  
            while (!Thread.interrupted()) {  
                System.out.println(Thread.currentThread().getName() + ":在搬砖呢");  
                try {  
                    Thread.sleep(1000);  
                } catch (InterruptedException e) {  
                    System.out.println("该跑路了...");  
                    break;//细节break  
                }  
            }  
            System.out.println("溜了, 溜了");  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        MyRunnable myRunnable = new MyRunnable();  
        Thread thread = new Thread(myRunnable, "李四");  
        System.out.println("包工头:小伙干活");  
        thread.start();  
        Thread.sleep(10000);  
        System.out.println("包工头:你两不是这工地的?!");  
        thread.interrupt();  
    }  
}
```
运行结果
```
包工头:小伙干活
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
李四:在搬砖呢
包工头:你两不是这工地的?!
该跑路了...
溜了, 溜了
```

thread收到通知的方式有两种
1. 如果线程因为调用wait/join/sleep等方法而阻塞挂起, 则以InterruptedCException异常的形式通知, 清除中断标志
	- 当出现InterruptedException的时候, 要不要结束线程取决于catch中代码的写法, 可以选择忽略这个异常, 也可以跳出循环结束线程 所以catch代码块内有没有`break;`是个细节
2. 否则, 只是内部的一个中断标志, thread可以通过
	- Thread.interrupted()判断当前线程的中断标志位被设置, *清楚中断标志*
	- Thread.currentThread().isInterrupted()判断指定线程的中断标志被设置, *不清楚中断标志位*
**这种方式通知收到的更及时, 即使线程正在sleep也可以马上收到**

> [!tip] 抛出interrupted exception这个异常会将线程的中中断状态重新设置为false。

> [!example] 示例3 观察标志位是否清楚

标志位是否清楚, 就类似于一个开关
	Thread.isInterrupted()相当于按下开关, 开关就自动弹起来, 这个称为"清除标志位"
	Thread.currentThread().isInterrupted()相当于按下开关之后, 开关不弹起来, 这个分称为"不清除标志位"
- 使用`Thread.isInterrupted()` 线程中断后就会清除标志位
```java
private static class MyRunnable implements Runnable {  
    @Override  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            System.out.println(Thread.interrupted());  
        }  
    }  
}  
  
public static void main(String[] args) {  
    MyRunnable myRunnable = new MyRunnable();  
    Thread thread = new Thread(myRunnable);  
    thread.start();  
    thread.interrupt();  
}
```
运行结构
```
true //只有一开始是true
false
false
false
false
false
false
false
false
false
```

- 使用`Thread.currentThread().isInterrupted()`, 线程中断标记位不会清除
```java
private static class MyRunnable implements Runnable {  
    @Override  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            System.out.println(Thread.currentThread().isInterrupted());  
        }  
    }  
}  
  
public static void main(String[] args) {  
    MyRunnable myRunnable = new MyRunnable();  
    Thread thread = new Thread(myRunnable);  
    thread.start();  
    thread.interrupt();  
}
```
```java
private static class MyRunnable implements Runnable {  
    @Override  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            System.out.println(Thread.currentThread().isInterrupted());  
        }  
    }  
}  
  
public static void main(String[] args) {  
    MyRunnable myRunnable = new MyRunnable();  
    Thread thread = new Thread(myRunnable);  
    thread.start();  
    thread.interrupt();  
}
```
运行结果
```
true
true
true
true
true
true
true
true
true
true
```

# 等待一个线程-join()
有时, 我们需要等待一个线程完成它的工作后, 才能进行自己的下一步工作. 例如, 张三只有等李四转账成功, 才决定是否存钱, 有时我们需要一个方法明确等待线程的结束
```java
public class Demo {  
    public static void main(String[] args) throws InterruptedException {  
        Runnable target = new Runnable() {  
            @Override  
            public void run() {  
                for (int i = 0; i < 3; i++) {  
                    try {  
                        System.out.println(Thread.currentThread().getName() + ":在干活呢");  
                        Thread.sleep(1000);  
                    } catch (InterruptedException e) {  
                        throw new RuntimeException(e);  
                    }  
                }  
                System.out.println(Thread.currentThread().getName() + ":干完了");  
            }  
        };  
        Thread thread1 = new Thread(target, "张三");  
        Thread thread2 = new Thread(target, "李四");  
        System.out.println("先让张三搬砖");  
        thread1.start();  
        thread1.join();  
        System.out.println("张三搬完了, 让李四去涂水泥");  
        thread2.start();  
        thread2.join();  
        System.out.println("李四也干完活了");  
    }  
}
```
运行结果
```
先让张三搬砖
张三:在干活呢
张三:在干活呢
张三:在干活呢
张三:干完了
张三搬完了, 让李四去涂水泥
李四:在干活呢
李四:在干活呢
李四:在干活呢
李四:干完了
李四也干完活了
```
> [!tip] 附录     

| 方法 | 说明 |
| ---- | ---- |
| public void join() | 等待线程结束 |
| public void join(long millis) | 等待线程结束，最多等 millis 毫秒 |
| public void join(long millis, int nanos) | 同理，但可以更高精度 |

# 获取当前线程的引用
| 方法 | 说明 |  |
| ---- | ---- | ---- |
| public static Thread currentThread(); | 返回当前线程对象的引用 |  |
