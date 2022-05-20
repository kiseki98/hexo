---
title: JUC
date: 2022/7/17 10:46:25
tags:
- Java
- JVM
- JUC
categories:
- [Java, JVM]
description: JUC
---

# JUC(基于AQS实现)

## 进程/线程是什么？

1. 进程：操作系统分配资源最小单元，也是基本的执行单元
2. 线程：操作系统调度最小单元

## 并发/并行是什么？

1. 并发：同一时刻多个线程在访问同一个资源，多个线程对一个点 例子：春运抢票；电商秒杀...
2. 并行：多项工作一起执行，之后再汇总 例子：泡方便面：电水壶烧水，一边撕调料倒入桶中

```java
// 售票类
public class Ticket {
    private final Lock LOCK = new ReentrantLock(); // 可重入锁
    private int sum = 30;

    // 按照官方文档来，操作公共资源方法在资源类中，卖票的方法sale，需要上锁的方法用try包裹，上锁在try的上面，解锁在finally里面！
    // 这是规范模板，必须按下面的规范写
    public void sale() {
        LOCK.lock();
        try {
            if (sum > 0) {
                System.out.println(Thread.currentThread().getName() + "当前" + sum-- + "张票" + ",剩余" + sum + "张票");
            }
        } finally {
            LOCK.unlock(); // 手动释放锁，synchronized自动释放
        }
    }
}
```

```java
// 买票类：线程本身和操作的公共资源无关！解耦
public class HelloWorld {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }).start();//.start()启动之后代表就绪，什么时候真正启动待cpu
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                ticket.sale();
            }
        }).start();
    }
}
```

# 线程间通信

多线程交互中防止线程的虚假唤醒，即多线程中不许使用`if`只能用`while`，尤其是对于`wait()`

## synchronized实现方式

![无标题](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%97%A0%E6%A0%87%E9%A2%98.png)

## Lock实现方式

```java
// 线程间通信：lock，condition.await();condition.signal();//精准唤醒
public class Demo {
    private final Lock LOCK = new ReentrantLock(); // 替换synchronized
    private Condition condition1 = LOCK.newCondition();// 替换wait()和notify()
    private Condition condition2 = LOCK.newCondition()
    // 使用condition.await();condition.signalAll();替换this.wait();this.notifyAll()因为和lock是一套的！
    private int num = 0;

    public void incr() {
        LOCK.lock();
        try {
            while (num != 0) {
                try {
                    condition1.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num++;
            System.out.println(Thread.currentThread().getName() + ":" + num);
            condition2.signalAll();
        } finally {
            LOCK.unlock();
        }
    }

    public void decr() {
        LOCK.lock();
        try {
            while (num == 0) {
                try {
                    condition2.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num--;
            System.out.println(Thread.currentThread().getName() + ":" + num);
            condition1.signalAll();
        } finally {
            LOCK.unlock();
        }
    }
}
```

# 八锁问题

![批注 2020-04-06 204754 (2)](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E6%89%B9%E6%B3%A8-2020-04-06-204754-(2).png)

# 线程安全工具类

## List集合

### ArrayList线程不安全

1. `add()`没有加`synchronized`修饰
2. 扩容：默认大小10（构造方法可以设置大小），扩容为1.5倍

### 线程安全的List

1. `Vector`(性能低不使用)
    1. `Vector` 的 `API` 有 `synchronized` 修饰，所以是线程安全的，同时只有一个线程能对其操作
    2. 加了同步锁，数据安全，但是性能下降
2. 使用工具类转换：使用 `Collections.synchronizedList(list)` 能把线程不安全的 `List` 转换为线程安全的 `List`
3. `CopyOnWriteArrayList`(写时复制/读写分离)
    1. 每次添加数据，扩容1，使用 `Arrays.copyOf()` 扩容
    2. `CopyOnWriteArrayList  ` 是 `Arraylist ` 的一种线程安全变体，其中所有可变操作`add`、`set`都是通过生成底层数组的新副本来实现的。

```java
public class CopyOnWriteArrayList {
    final Object[] getArray() {
        return array; // 返回当前容器
    }

    final void setArray(Object[] a) {
        array = a; // 设置新容器为当前容器
    }

    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 不直接操作容器elements，而是copy到新容器newElements中，在使用setArray将原容器引用指向新容器，使用get()就不需要加锁了，是读写分离的思想
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
}
```

## Set集合

### HashSet线程不安全

1. `HashSet` 底层是 `HashMap`，`value` 是常量，`key`用来存储数据
2. 初始容量：和`HashMap`一致，扩容原则也一致

### 线程安全的Set

1. `Collections.synchronizedSet(set)`方法能把线程不安全的`Set`转换为线程安全的`Set`
2. `CopyOnWriteArraySet`是线程安全的和`CopyOnWriteArrayList`同理（底层实现相同多了去重逻辑）

## Map

### HashMap线程不安全

1. `HashMap`不是线程安全的，底层是：`Node`类型的单向链表和`Node`类型的数组+红黑树
2. 默认初始大小：16（可修改），负载因子0.75（可以修改），构造方法可以设置大小和负载因子
3. 扩容：当键值对数量>容量*负载因子时扩容（按2倍扩容），初始设置容量如果不是2的指数幂，`HashMap`会把容量调整为2的指数幂

### 线程安全的Map

1. 工具类转换：`Collections.synchronizedMap(map)`
2. `ConcurrentHashMap`：使用 `synchronized` 锁定数组的索引

```java
public class ConcurrentHashMap {
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K, V>[] tab = table; ; ) {
            Node<K, V> f;
            int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                        new Node<K, V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            } else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // f是tabAt()计算得到的数组下标
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K, V> e = f; ; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                        ((ek = e.key) == key ||
                                                (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K, V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K, V>(hash, key,
                                            value, null);
                                    break;
                                }
                            }
                        } else if (f instanceof TreeBin) {
                            Node<K, V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K, V>) f).putTreeVal(hash, key,
                                    value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
}
```

# 线程池

## 获取多线程的四种方法

创建线程只有`new Thread(runnable target,string name)`一种方法

1. 实现 `Runnable` 接口
2. 继承 `Thread`：`Thread `实现了 `Runnable` 接口
3. 实现`Callable`接口：不可以直接替换 `Runnable`，`Thread`的构造方法只能传 `Runnable` 的实现类，通过 `FutureTask` 使用多线程
4. 使用线程池 `ExecutorService` 的实现类 `ThreadPoolExecutor`

|    接口    |      返回值      | 抛出异常 | 接口方法 |
| :--------: | :--------------: | :------: | :------: |
| `Runnable` |        ×         |    ×     | `run()`  |
| `Callable` | 通过接口泛型指定 |    √     | `call()` |

```java
// 定义Runnable实现类
public class RunnableDemo implements Runnable {
    @Override
    public void run() {
    }
}

// 定义Callable实现类，泛型指定返回值类型
public class CallableDemo implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        return null;
    }
}

public class Demo {
    public static void main(String[] args) {
        // Callable使用FutureTask（实现Runnable和Future接口）实现多线程
        FutureTask<Integer> futureTask = new FutureTask(new CallableDemo());
        Thread t = new Thread(futureTask);
        // 使用run()方法不是异步的方式
        futureTask.run();
        // 使用get()方法获取Callable的call()的返回值抛出InterruptedException异常
        futureTask.get();
        // 使用get()，如果计算尚未完成会阻塞调用get()的线程，直到计算完成
        futureTask.cancel(true); // 取消计算
    }
}

```

## 使用线程池(ExecutorService的实现类ThreadPoolExecutor)

为什么使用线程池，多CPU电脑不用切换线程效率更高

### 线程池特点(线程复用，控制最大并发数，管理线程)

1. 线程复用，降低资源消耗，提高响应速度。通过重复利用已创建的线程降低线程创建和销毁造成的销耗。
2. 控制最大并发数
3. 管理线程，线程是稀缺资源，无限制创建线程会消耗系统资源降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控

### 三大内置线程池Executors工具类

1. `Executors.newFixedThreadPool(int i)`：创建一个线程池有N个固定的线程
2. `Executors.newSingleThreadExecutor()`：一池一线程
3. `Executors.newCachedThreadPool()`：执行很多短期异步任务，线程池根据需要创建新线程，但在先前构建的线程可用时将重用它们，可扩容

### 线程池七大参数

1. 常驻核心线程数：`int corePoolSize`
2. 最大线程数：`int maximumPoolSize`，线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
3. 空闲存活时间：`long keepAliveTime`，多余的空闲线程的存活时间，当前池中线程数量超过corePoolSize时，当空闲时间达到keepAliveTime时，多余线程会被销毁直到只剩下corePoolSize个线程为止
4. 空闲存活时间单位：`TimeUnit unit`
5. 任务队列：`BlockQueue workQueue`，实质是阻塞队列，被提交但尚未被执行的任务（候客区）
6. 线程工厂：`ThreadFactory threadFactory`，表示生成工作线程的线程工厂，用于创建线程
7. 拒绝策略：`RejectedExecutionHandler handler`，表示当队列满了，并且工作线程大于等于线程池的最大线程数（`maximumPoolSize`）时如何来拒绝请求执行的 `runnable` 的策略

```java
ThreadFactory build=new ThreadFactoryBuilder().build(); // 线程工厂
        RejectedExecutionHandler handler=new ThreadPoolExecutor.AbortPolicy(); // 拒绝策略
        ThreadPoolExecutor poolExecutor=new ThreadPoolExecutor(1,1,120,TimeUnit.MINUTES,new ArrayBlockingQueue(2),build,handler);
```

### 线程池方法

1. 执行任务
    1. `threadPool.execute(Runnable task)`
    2. `threadPool.submit(Runnable/Callable)`，可以传入`Callable`，返回值是`FutureTask`
2. 终止线程
    1. `threadPool.shutDown()`，温柔的终止线程池，不接受新任务，但会执行正在运行的任务和阻塞队列中的任务
    2. `threadPool.shutDownNow()`，不接受新任务，不处理阻塞队列中的任务，尝试中中断进行中的任务

## 拒绝策略(ThreadPoolExecutor的内部类)

1. `AbortPolicy`(默认)：直接抛出`RejectedExecutionException`异常阻止系统正常运行
2. `CallerRunsPolicy`：“调用者运行”一种调节机制，提交任务的线程去执行该任务，从而降低新任务的流量。
3. `DiscardOldestPolicy`：抛弃队列中最前面的任务，尝试再次提交当前任务到队列中。
4. `DiscardPolicy`：该策略默默地丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

## 线程池原理

1. 在创建了线程池后，开始等待请求。
2. 当调用`execute()`方法添加一个请求任务时，线程池会做出如下判断：
    1. 如果正在运行的线程数量小于`corePoolSize`，那么马上创建线程运行这个任务
    2. 如果正在运行的线程数量大于或等于`corePoolSize`，那么将这个任务放入队列
    3. 如果这个时候队列满了且正在运行的线程数量还小于`maximumPoolSize`，那么还是要创建非核心线程立刻运行这个任务；
    4. 如果队列满了且正在运行的线程数量大于或等于`maximumPoolSize`，那么线程池会启动饱和拒绝策略来执行
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行
4. 当一个线程无事可做超过一定的时间 `keepAliveTime` 时， 如果当前存活的线程数大于 `corePoolSize`，那么这个线程就被停掉。 它最终会收缩到`corePoolSize `的大小。

![3096083-2286629fbc77d3b5](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/3096083-2286629fbc77d3b5.jpg)

# JUC辅助类

## CountDownLatch减少计数

1. 计数到0，阻塞线程重新运行，只能使用一次
2. 场景：一个方法内部多个线程，如何保证main线程最后结束？只能使用一次
3. 原理：
    1. 调用`CountDownLatch`对象主要有两个方法，当一个或多个线程调用`await()`方法时会阻塞。
    2. 其它线程调用`countDown()`方法(不会被阻塞)会将计数器减1， 当计数器的值变为0时，因`await()`阻塞的线程会被唤醒，继续执行。

```java
// CountDownLatch：模拟同学（线程）在教室自习，要求班长（main线程）锁门（最后走）
public class Demo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(9); // 假定9个同学
        for (int i = 0; i < 9; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "离开教室");
                countDownLatch.countDown();//离开教室，计数减1
            }, i + "").start();
        }
        countDownLatch.await();//阻塞的线程，必须等到计数为0继续运行
        System.out.println("班长锁门");
    }
}
```

## CyclicBarrier循环栅栏

1. 场景：要求凑齐几个线程才能开始（没凑齐前都阻塞），可重复使用
2. 原理：
    1. `CyclicBarrier`的字面意思是可循环`Cyclic`使用的屏障`Barrier`
    2. 让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有 被屏障拦截的线程继续运行
    3. 线程进入屏障通过 `CyclicBarrier` 的 `await()` 方法。

```java
// CyclicBarrier：模拟凑齐七个同学开会，先到的同学（线程）先等着，到齐开会
public class Demo {
    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("班长开始开会");
        });//假定凑齐7位同学开会
        for (int i = 0; i < 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "到了");
                try {
                    cyclicBarrier.await();//先来的等着（阻塞），等凑到7个线程再一起开会（继续运行）
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, i + "").start();
        }
    }
}
```

## Semaphore信号灯

1. 场景：对线程并发控制，和资源的互斥
2. 原理： 在信号量上我们定义两种操作：
    1. `acquire()`获取，当一个线程调用`acquire()`操作时，它要么通过成功获取信号量(信号量减1)，要么一直等下去，直到有线程释放信号量，或超时
    2. `release()`释放，实际上会将信号量的值加1，然后唤醒等待的线程
    3. 信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。

```java
// Semaphore：模拟抢车位，车位4个，汽车9个。没抢到的等着（阻塞），用于对线程并发的控制，和资源的互斥
public class Demo {
    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(4); // 表示4个车位，如果设置为1，相当于synchornized的！！
        for (int i = 0; i < 9; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire(); // 抢到车位，车位减1，为0时其他汽车等待（线程阻塞）
                    System.out.println(Thread.currentThread().getName() + "抢到车位");
                    Thread.sleep(4000);
                    System.out.println(Thread.currentThread().getName() + "离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();// 离开车位，车位加1，等待的汽车可以停车（线程继续运行）
                }
            }, i + "").start();
        }
    }
}
```

# 读写锁

```java
// 下面的读写锁的使用：写入时用写锁，读取时用读锁。只有多线程读取允许并发
public class Demo {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在写" + key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t 写完了" + key);
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public Object get(String key) {
        rwLock.readLock().lock();
        Object result = null;
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在读" + key);
            System.out.println(Thread.currentThread().getName() + "\t 读完了" + result);
        } finally {
            rwLock.readLock().unlock();
        }
        return result;
    }
}
```

# 阻塞队列

## 阻塞队列

1. 阻塞队列：是一个队列，线程1往阻塞队列里添加元素，线程2从阻塞队列里移除元素
    1. 当队列是空的，试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素
    2. 当队列是满的，试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除元素使队列变得空闲起来并后续新增
2. 阻塞：在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤起为什么需要
1. `BlockingQueue`好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切`BlockingQueue`都给你一手包办了
2. 在`concurrent`包发布以前，在多线程环境下，程序员都必须去控制这些细节，还要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。

## 实现类(前三个重要)

1. `*ArrayBlockingQueue`：由数组结构组成的有界阻塞队列
2. `*LinkedBlockingQueue`：由链表结构组成的有界（大小默认值`Integer.MAX_VALUE`）阻塞队列
3. `*SynchronousQueue`：不存储元素的阻塞队列，也即单个元素的队列
4. `PriorityBlockingQueue`：优先级排序的无界阻塞队列。不是`FIFO`，根据优先级
5. `DelayQueue`：使用优先级队列实现的延迟无界阻塞队列
6. `LinkedTransferQueue`：由链表组成的无界阻塞队列
7. `LinkedBlockingDeque`：由链表组成的双向阻塞队列

## 阻塞队列的API

1. 抛出异常
    1. 插入：抛出异常当阻塞队列满时，再往队列里`add`插入元素会抛`IllegalStateException:Queue full`
    2. 移除：当阻塞队列空时，再往队列里`remove`移除元素会抛`NoSuchElementException`
2. 特殊值
    1. 插入，成功`ture`失败`false`
    2. 移除，成功返回出队列的元素，队列里没有就返回`null`
3. 阻塞超时组
    1. 插入：当阻塞队列满时，生产者线程继续往队列里`put`元素，队列会一直阻塞生产者线程直到`put`数据或者响应中断退出(超时)
    2. 移除：当阻塞队列空时，消费者线程试图从队列里`take`元素，队列会一直阻塞消费者线程直到队列可用或者超时退出当阻塞队列满时

|       API分组\操作       |                             插入                             |                             移除                             |                         检查                          |
| :----------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :---------------------------------------------------: |
|        抛出异常组        |           `add()`【成功：`true`，失败：抛出异常】            |   `remove()`【成功：移除并返回第一个元素，失败：抛出异常】   | `element()`【成功：返回队列第一个元素失败：抛出异常】 |
|      特殊值(返回值)      |           `offer()`【成功：`true`，失败：`false`】           |     `poll()`【成功：移除并返回第一个元素，失败：`null`】     |  `peek()`【成功：返回队列第一个元素，失败：`null`】   |
| 超时组(返回值同特殊值组) | `offer()`【阻塞：队列满，阻塞一定时间，超时：取消操作，未超时：队列有元素被移除，添加成功】 | `poll()`【阻塞：队列空，阻塞一定时间，超时：取消操作，未超时：队列有元素被添加，移除成功】 |                        不可用                         |
|          阻塞组          | `put()`【成功：`void`，阻塞：一直阻塞，直到可用（有空位）】  | `take()`【成功：`void`，阻塞：一直阻塞，直到可用（有元素）】 |                        不可用                         |

