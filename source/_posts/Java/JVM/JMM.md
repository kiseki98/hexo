---
title: JAVA内存模型
date: 2022/7/16 10:46:25
tags:
- Java
- JVM
- JMM
categories:
- [Java, JVM]
description: JAVA内存模型
---

# JMM(JAVA内存模型)

**JMM定义了一套在多线程读写共享数据时（成员变量，数组）对数据可见性，有序性和原子性的规则和保障**

# 原子性

## 原子性

原子性指该操作是不可再分的，用意是只能有一个线程来对它进行操作，在整个操作过程中不会被线程调度器中断，即一组操作要么全部执行 ，要么就都不执行

## 原子操作

1. 是指不会被线程调度机制打断的操作
2. 这种操作一旦开始，就一直运行到结束，中间不会有任何 `context switch`
3. 原子操作可以是一个或多个步骤，但其**顺序**不可以被打乱，也不可一直执行其中部分，将整个操作视为一个整体是云自行核心特征（**静态变量**的自增自减不是原子操作）

## 解决方法

使用 `synchronized` 关键字实现

```java
public class HelloWorld {
    static int i = 0;
    static Object object = new Object();

    public static void main(String[] args) throws InterruptedException {
        // 两个线程的锁对象都是object
        Thread t1 = new Thread(() -> {
            synchronized (object) {
                for (int j = 0; j < 1000000; j++) {
                    i++;
                }
            }
        });
        Thread t2 = new Thread(() -> {
            synchronized (object) {
                for (int j = 0; j < 1000000; j++) {
                    i--;
                }
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i); // 最终执行结果是0，不加synchronized最终结果不确定
    }
}
```

![批注 2020-04-05 171626](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-05-171626.png)

# 可见性

## 问题引入

![批注 2020-04-05 173100](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-05-173100.png)

## 解决方案（实际睡眠过短不会出现可见性问题）

1. 使用 `volatile` 修饰成员变量，它可以避免线程从自己的缓存中查找值，必须到主存中查找它的值，线程操作 `volatile` 变量都是直接操作主存
2. 它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存 中获取它的值，线程操作`volatile`变量都是直接操作主存
3. 前面例子体现的实际就是可见性，它保证的是在多个线程之间，一个线程对 `volatile` 变量的修改对另一个线程可见，不能保证**原子性**，仅用在一个写线程，多个读线程的情况： 上例从字节码理解是这样的：

注意：`synchronized` 语句块既可以保证代码块内变量可见性，也可以保证代码块原子性，缺点是 `synchronized` 属于重量级操作

# 有序性（多线程指定重排会影响正确性）

## 问题引入

![批注 2020-04-05 183544](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-05-183544.png)

## 经典单例模式解释指令重排：double check singleton

![批注 2020-04-05 185926](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-05-185926.png)

```java
public class Demo {
    // 问题出现在创建对象的语句 INSTANCE = new Singleton(); 上，在java中创建一个对象并非是一个原子操作，可以被分解成三行伪代码：
    //1：分配对象的内存空间
    memory = allocate();
    //2：初始化对象
    ctorInstance(memory);
    //3：设置instance指向刚分配的内存地址
    instance = memory;
    // 上面三行伪代码中的2和3之间，可能会被重排序（在一些JIT编译器中）,即编译器或处理器为提高性能改变代码执行顺序，这一部分的内容稍后会详细解释，重排序之后的伪代码是这样的：导致设置 instance 指向刚分配的空间，这个时候还没有初始化对象，如果这时候其他线程getInstance()。会发现INSTANCE != null（实际还没有初始化）

    //1：分配对象的内存空间
    memory = allocate();
    //3：设置instance指向刚分配的内存地址
    instance = memory;
    //2：初始化对象
    ctorInstance(memory);
}
```

# happens-before

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作；
2. 锁定规则：一个 `unLock` 操作先行发生于后面对同一个锁的 `lock` 操作；
3. `volatile` 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作
4. 传递规则：如果操作 `A` 先行发生于操作 `B`，而操作 `B` 又先行发生于操作 `C`，则可以得出操作A先行发生于操作C；
5. 线程启动规则：`Thread   ` 对象的 `start()` 方法先行发生于此线程的每个一个动作；
6. 线程中断规则：对线程 `interrupt()` 方法的调用先行发生于被中断线程的代码检测到中断事件的发生；
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 `Thread.join()` 方法结束、`Thread.isAlive() `的返回值手段检测到线程已经终止执行；
8. 对象终结规则：一个对象的初始化完成先行发生于他的 `finalize()` 方法的开始；

# CAS(无锁并发)

## 定义

`CAS` 即 `Compare And Swap`，它体现的是一种乐观锁思想，比如多个线程对一个共享变量执行+1操作

1. `CAS` 是基于乐观锁的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗。
2. `synchronized`是基于悲观锁的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会

![批注 2020-04-05 214358 (2)](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-05-214358-(2).png)

# 原子操作类

`JUC`（`java.util.concurrent`包下提供的类，可以提供线程安全的操作）（底层采用`CAS`和`volatile`来实现）

# 锁自旋和锁重入

## 锁自旋

1. 很多 `synchronized` 里面的代码只是一些很简单的代码，执行时间非常快，等待的线程都加锁可能是一种不太值得的操作， 因为线程阻塞涉及到用户态和内核态切换的问题。
2. 既然 `synchronized` 里面的代码执行得非常快，不妨让等待锁的线不要被阻塞，而是在`synchronized`的边界做忙循环，这就是自旋。如果做了多次忙循环发现还没有获得锁，再阻塞，这样可能是一种更好的策略。
3. 适应自旋锁：
    1. 线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。
    2. 反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源

## 锁重入

1. `JVM   `负责跟踪对象被加锁的次数。
2. 线程第一次给对象加锁的时候，计数变为1.每当这个相同线程在此对象上再次获得锁时，计数会递增。
3. 每当任务离开时，计数递减，当计数为0时，锁被完全释放。
    1. 可重入
        1. 如果线程已拿到锁之后，还想再次进入由这把锁所控制的方法中，而无需提前释放，可以直接进入。
        2. 指的是同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁。也叫做递归锁。
    2. `JAVA  ` 中两大递归锁：`Synchronized` 和 `ReentrantLock`

## 偏向锁

1. 偏向锁，顾名思义，它会偏向于第一个访问锁的线程，如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，则线程是不需要触发同步的，这种情况下，就会给线程加一个偏向锁。
2. 如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，`JVM  ` 会消除它身上的偏向锁，将锁恢复到标准的轻量级锁

# synchronized优化

1. `Java HotSpot  ` 虛拟机中，每个对象者`P`有对象头（包括`class`指针和`Mark Word`)。 `Mark Word`
   平时存储送个对象的哈希码、分代年龄，当加锁时，这些信息就根据情况被替换（把信息保留在栈帧中）为标记位、线程锁记录指针、重量级锁指针、线程`ID`等内容
2. 轻量级锁：如果一个对象虽然有多线程访问，但多线程访问的时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化。这就好比学生（线程A)
   用课本占座，上了半节课，出门了（CPU时间到），回来一看，发现课本没变，说明没有竞争，继续上他的课。如果这期间有其它学生（线程B)来了，会告知（线程A)
   有并发访问，线程A随即升级为重量级锁，进入重量级锁的流程。而重量级锁就不是那么用课本占座那么简单了，可以想象线程A走之前，把座位用一个铁栅栏围起来假设有两个方法同步块，利用同一个对象加锁
3. 锁膨胀：如果在尝试加轻量级锁的过程中，`CAS  ` 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要逬行锁膨胀，将轻量级锁变为重量级锁。
4. 重量级锁：重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。
5. 优化
    1. 减少上锁时间，同步代码块中尽量短
    2. 减少锁的粒度
        1. 将4锁拆分为多个锁提高并发度，例如：
            1. `ConcurrentHashMap`，LongAddei•分为base和cells两部分。没有并发争用的时候或者是cells数组正在初始化的时候，会使用，`CAS`来累加值到base, 有 并 发 争 用 ，
               会 初 始 化 cells数组，数组有多少个ceil, 就 允 许 有 多 少 线 程 并 行 修改，最后将数组中每个`cell`累加，再加上`base`就是最终的值，`LinkedBlockingQueue`
               入队和出队使用不同的锁，相对于 `LinkedBlockingArray `只有一个锁效率要高
    3. 锁粗化
        1. 多次循环进入同步块不如同步块内多次循环
        2. 另外JVM可能会做如下优化，把多次append的加锁操作粗化为一次（因为都是对同一个对象加锁，没必要重入多次）
        3. `new StningBuffer().append("a").append("b").append(11 c ");`
    4. 锁消除：`JVM` 会进行代码的逃逸分析，例如某个加锁对象是方法内局部变量，不会被其它线程所访问到，这时候就会被即时编译器忽略掉所有同步操作。
    5. 读写分离