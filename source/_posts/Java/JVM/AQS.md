---
title: AQS
date: 2022/7/16 20:46:25
tags:
- Java
- JVM
- AQS
categories:
- [Java, JVM]
description: AQS
---

# AQS 基本概念

1. **AQS** 继承了 **AbstractOwnableSynchronizer**
2. **AbstractOwnableSynchronizer** 内部属性  **exclusiveOwnerThread** 是当前持锁线程，上锁和释放锁要和 **exclusiveOwnerThread** 进行对比
3. 缩写：**AbstractQueuedSynchronizer** 抽象队列同步器
4. **JUC** 包的核心
5. **Java** 并发编程的核心

# ReentrantLock

现实案例：银行办业务

1. 看办理的人多不多，不多就去柜台，多的话就排队
2. 等待区
3. 如果行长小舅子来了，行长和柜台打了声招呼（接待完当前客户，接待小舅子），不需要排队，柜台空了，直接去办
4. 其他人看排队的人太多，领了号，出去逛，结果过号了，需要重新排队

```java
/**
 * 先看 lock 操作 
 */
public class ThreadTest_5 {
    public static final ReentrantLock LOCK = new ReentrantLock();
    public static int val = 0;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> count());
        Thread t2 = new Thread(() -> count());
        Thread t3 = new Thread(() -> count());
        t1.start();
        t2.start();
        t3.start();
        try {
            t1.join();
            t2.join();
            t3.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(val);
    }

    static void count() {
        try {
            LOCK.lock();
            for (int i = 0; i < 10000; i++) {
                val++;
            }
        } finally {
            LOCK.unlock();
        }
    }
}
```

## ReentrantLock 源码

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    // 内部类 Sync
    private final Sync sync;

    // 默认构造方法，使用非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    // 有参构造方法，根据参数判断使用公平锁还是非公平锁
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    // 上锁的方法，会调用 Sync（继承AQS）
    public void lock() {
        sync.lock();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public int getHoldCount() {
        return sync.getHoldCount();
    }

    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }

    public boolean isLocked() {
        return sync.isLocked();
    }

    protected Thread getOwner() {
        return sync.getOwner();
    }

    public final boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public final boolean hasQueuedThread(Thread thread) {
        return sync.isQueued(thread);
    }

    public final int getQueueLength() {
        return sync.getQueueLength();
    }

    protected Collection<Thread> getQueuedThreads() {
        return sync.getQueuedThreads();
    }

    public boolean hasWaiters(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject) condition);
    }

    public int getWaitQueueLength(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject) condition);
    }

    protected Collection<Thread> getWaitingThreads(Condition condition) {
        if (condition == null)
            throw new NullPointerException();
        if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
            throw new IllegalArgumentException("not owner");
        return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject) condition);
    }
}
```

## 内部类 Sync 源码

```java
/**
 * Base of synchronization control for this lock. Subclassed
 * into fair and nonfair versions below. Uses AQS state to
 * represent the number of holds on the lock.
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

    /**
     * Performs {@link Lock#lock}. The main reason for subclassing
     * is to allow fast path for nonfair version.
     */
    abstract void lock();

    /**
     * Performs non-fair tryLock.  tryAcquire is implemented in
     * subclasses, but both need nonfair try for trylock method.
     */
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        } else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class

    final Thread getOwner() {
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }

    final int getHoldCount() {
        return isHeldExclusively() ? getState() : 0;
    }

    final boolean isLocked() {
        return getState() != 0;
    }

    /**
     * Reconstitutes the instance from a stream (that is, deserializes it).
     */
    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}
```

## AQS 的内部类 Node

```java
static final class Node {
    // 共享锁
    static final Node SHARED = new Node();
    // 排他锁
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED = 1;
    /** waitStatus value to indicate successor's thread needs unparking */
    static final int SIGNAL = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;

    volatile int waitStatus;

    // 双向队列 前一个结点
    volatile Node prev;

    // 双向队列 后一个节点
    volatile Node next;

    volatile Thread thread;

    // CyclicBarrier 里面使用到
    Node nextWaiter;
}
```

## 内部类非公平锁 NonfairSync 源码

```java
/**
 * Sync object for non-fair locks
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

## 内部类公平锁 FairSync 源码

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
     * 公平锁和非公平锁抢锁就在于 hasQueuedPredecessors，非公平锁没这个方法
     * hasQueuedPredecessors 用于判断等待区是否有线程
     * compareAndSetState 加锁方法
     * setExclusiveOwnerThread 设置获得锁的线程 Thread
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        } else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

# AQS 的六个操作

## 抢锁 tryAcquire

1. 看锁标志位 **state** ，默认 0 ，这时候有线程抢到锁，需要改锁标志位 **state**
    1. **state** 是 0，没有线程占有锁
    2. **state** 是 1，有线程占有锁
    3. 需要一种机制 **CAS** 保证同一时刻只有一个线程能改成功 **state** 的值
2. 有线程占有锁，这时候有线程过来抢锁
    1. 当前抢锁的线程是不是占有锁的线程
        1. 是：重入，**state + 1**，释放的时候需要释放对应次数，所以加 1
        2. 不是：抢锁失败
    2. 判断锁被占有的优化：
        1. 看等待区有没有线程，有则表示锁被占用
        2. 公平锁使用 **hasQueuedPredecessors** 方法判断等待区是否有线程
3. 公平锁和非公平锁的差异
    1. 唯一的差别就是一个临界区
    2. 之前占有锁的线程刚好执行完任务释放锁，这时候恰好来了一个线程，不看等待区有没有线程等待，直接获取锁（类似之前给柜台打招呼，小舅子插队）

```java
public class ReentrantLock {
    // 调用 ReentrantLock 公平锁的 lock 方法
    final void lock() {
        acquire(1);
    }

    /**
     * tryAcquire 争抢锁
     * acquireQueued 抢锁失败，加入等待区
     * selfInterrupt 给当前线程打赏中断标记
     */
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        // 没有线程获得锁
        if (c == 0) {
            /**
             * 公平锁和非公平锁抢锁就在于 hasQueuedPredecessors，非公平锁没这个方法
             * hasQueuedPredecessors 用于判断等待区是否有线程
             * compareAndSetState 加锁方法
             * setExclusiveOwnerThread 设置获得锁的线程 Thread
             */
            if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 有线程获得锁，且是当前线程（锁重入）
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}

```

## 抢锁失败 addWaiter 入队

1. 队列中有没有节点
    1. 没有
        1. **AQS** 式队列：生成两个节点，头节点的 **waitStatus** 用来判断占有锁的线程是否执行完
        2. 普通队列：生成一个节点
    2. 有：尾部插入

### 队列为空（生成两个节点）

![队列为空时入队](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E9%98%9F%E5%88%97%E4%B8%BA%E7%A9%BA%E6%97%B6%E5%85%A5%E9%98%9F.png)

### 设置队列尾节点

![设置尾节点](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/%E8%AE%BE%E7%BD%AE%E5%B0%BE%E8%8A%82%E7%82%B9.png)

### 入队操作 acquireQueued（自旋两次）

**入队以后做什么？**

1. 入队后是第二个节点，再次抢锁**自旋两次**， **acquireQueued** 方法（第一个节点**对应**持有锁的线程，此线程不在队列内）
2. 其他线程**上闹钟**和
3. 其他线程**阻塞** ，自旋了两次，是 **shouldParkAfterFailedAcquire** 方法

```java
// 调用 ReentrantLock 公平锁的 lock 方法
public class ReentrantLock {
    final void lock() {
        acquire(1);
    }

    /**
     * tryAcquire 争抢锁
     * acquireQueued 抢锁失败，加入等待区
     * selfInterrupt 给当前线程打赏中断标记
     */
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
                // 加入等待区
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    // 入队方法 addWaiter 把线程封装成一个 Node 插入队列
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            // 尾部部位不为空
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 首次入队，如果 tail 为 null 生成两个节点
        enq(node);
        return node;
    }

    // 首次入队 addWaiter 调用的 enq
    private Node enq(final Node node) {
        for (; ; ) {
            Node t = tail;
            if (t == null) { // 初始化设置头节点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                // 设置尾部节点指向
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

    // acquireQueued 方法
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            /**
             * 下面整个 for 循环是一次自旋，两次都自旋失败才会阻塞当前线程
             * 		1.调用 acquireQueued 方法，第一次 for 循环，抢锁失败（自旋失败），设置当前入队节点的 pre 节点的 waitStatus 为 -1
             * 		2.第二次 for 循环，抢锁失败（自旋失败），获取到 pre 节点的 waitStatus 为 -1，阻塞当前入队节点对应的线程
             * 抢锁成功：头节点出队，设置当前入队 node 为头节点，当前 node 的 pre 节点的 next 节点为 null
             */
            for (; ; ) {
                // 获取当前节点的 pre 节点
                final Node p = node.predecessor();
                // 判断当前节点 node 是不是第二个节点，是就尝试抢锁（一般来讲锁的代码块内执行比较快，入队之后可能就，占有锁的线程释放锁了）
                if (p == head && tryAcquire(arg)) {
                    // 抢锁成功，头节点出队，当前节点成为头节点
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                /**
                 * 尝试上锁失败
                 * shouldParkAfterFailedAcquire 方法判断当前入队节点的前一个节点的状态
                 * 		1.入队节点是第二个节点（这时候抢锁失败了），前一个节点的 waitStatus 默认是 1，返回 true，parkAndCheckInterrupt 阻塞当前节点			 *		中的线程，调用 parkAndCheckInterrupt 方法内部使用 LockSupport.park(this)
                 *		2.入队节点非第二个节点，前一个节点的 waitStatus 默认是 0 ，设置前一个节点的 waitStatus 为 -1
                 */
                if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    // 设置闹钟 waitStatus
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            // 当前入队节点的 pre 节点的 waitStatus 为 -1， 返回 true，阻塞入队节点的线程
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            // 当前入队节点的 pre 节点的 waitStatus 为大于 0，表示 pre 节点被取消，重新指向 pre 节点的 pre 节点
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 当前入队节点的 pre 节点的 waitStatus 为 0，把当前节点的前一个节点设置为 -1
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
}
```

## 阻塞（入队自旋两次失败）

## 释放锁

1. 重入问题
2. 回复锁状态位 **state** 为 0
3. 唤醒队列中等待的线程

```java
// 调用 ReentrantLock 公平锁的 unlock 方法
public class ReentrantLock {
    public void unlock() {
        sync.release(1);
    }

    // 调用 Sync 的 release 方法
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            // 当锁释放完毕后如果头节点不为 null 且 waitStatus 不为 0
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

    // tryRelease 尝试释放锁，释放一次 state 减 1，当 state 回复到 0 的时候，setExclusiveOwnerThread 方法设置当前持锁线程为 null
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    private void unparkSuccessor(Node node) {
        // node 是头节点
        int ws = node.waitStatus;
        // 头节点的 waitStatus 小于 0 设置头节点的 waitStatus 为 0
        if (ws < 0) compareAndSetWaitStatus(node, ws, 0);

        // 头节点的 next 节点
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            // waitStatus 大于 0 表示取消，就跳过 next 节点
            s = null;
            // 从尾节点向前找，找到离头节点最近的一个节点唤醒
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0) s = t;
        }
        // 头节点的 next 节点不为 null，唤醒 next 节点
        if (s != null) LockSupport.unpark(s.thread);
    }
}
```

## 唤醒（释放锁后唤醒）

## 出队（被唤醒的线程来操作）

**在 acquireQueued 方法内自旋两次失败，被阻塞，被唤醒后继续执行 for 循环，抢锁成功后头节点出队**

### 唤醒失败，队列中部出队

**还有一种特殊情况，如海底捞吃饭叫号，人很多出去逛回来发现过号了，需要重新排，acquireQueued方法 中的 finally 中的 cancelAcquire 方法，唤醒失败会一处自身，并且唤醒下一个节点**

```java
public class ReentrantLock {
    private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;

        node.thread = null;

        // Skip cancelled predecessors
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;

        // predNext is the apparent node to unsplice. CASes below will
        // fail if not, in which case, we lost race vs another cancel
        // or signal, so no further action is necessary.
        Node predNext = pred.next;

        // Can use unconditional write instead of CAS here.
        // After this atomic step, other Nodes can skip past us.
        // Before, we are free of interference from other threads.
        node.waitStatus = Node.CANCELLED;

        if (node == tail && compareAndSetTail(node, pred)) {
            // 当前节点为尾节点，当前节点自身出队
            compareAndSetNext(pred, predNext, null);
        } else {
            // If successor needs signal, try to set pred's next-link
            // so it will get one. Otherwise wake it up to propagate.
            int ws;
            if (pred != head &&
                    ((ws = pred.waitStatus) == Node.SIGNAL ||
                            (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                    pred.thread != null) {
                Node next = node.next;
                // 设置当前节点的前一个未取消节点的 next 节点指向当前节点的后面节点
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }

            node.next = node; // help GC
        }
    }
}
```

## 公平锁加锁流程图

![ReentrantLock公平锁加锁](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/ReentrantLock%E5%85%AC%E5%B9%B3%E9%94%81%E5%8A%A0%E9%94%81.png)

## 非公平锁加锁流程图

![ReentrantLock非公平锁加锁](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/ReentrantLock%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81%E5%8A%A0%E9%94%81.png)

## 解锁流程图

![ReentrantLock公平锁与非公平锁解锁](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/ReentrantLock%E5%85%AC%E5%B9%B3%E9%94%81%E4%B8%8E%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81%E8%A7%A3%E9%94%81.png)

# AQS式的队列和普通队列

## CLH算法（自旋锁）更符合 CPU 内部执行结构

## AQS式队列

占有锁的线程执行完，判断等待队列第一个节点状态判断是否唤醒，队列的头部是占有锁的线程

![AQS队列](https://img-1301279461.cos.ap-nanjing.myqcloud.com/img/AQS%E9%98%9F%E5%88%97.png)