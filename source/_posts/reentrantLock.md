---
title: ReentrantLock源码分析
date: 2018-03-06 09:16:18
tags:
  -  Java  
categories:  
  -  Java多线程集合 
type: tags
---
### ReentrantLock定义
之前看过ReentrantLock来实现多个线程之间的同步，但是确没有仔细分析过其底层实现原理，故此篇来分析其底层原理的实现。

ReentrantLock的实现前提是AbstractQueuedSynchronizer，即AQS,下面来介绍一下ReentrantLock是如何实现多个锁的并发操作的。
在创建一个ReentrantLock区执行锁的方法时，一般用的比较多的是非公平锁。
####  ReentrantLock内部的实现
在调用lock()方法时，其执行流程是这样子的：

```
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
首先通过cas方法获取当前的state,如果成功的话则当前线程获取了对应的锁，否则则执行acquire方法。

```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
acquire方法中有一个方法叫做tryAcquire方法，这个方法是尝试获取一次锁，如果没有获取到锁的话则走第二个流程尝试将该节点天骄到队列的尾部。而这个方法又会调用到nonfairTryAcquire方法。

```
     final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```
来分析下这段代码，首先需要获取当前的状态state，由于它是volatile的，对线程具有可见性，所以可能之前一个线程已经释放了锁，在这里需要重新判断一次是否获取到了锁。  
下一个if判断语句主要是用来判断重入锁，如果当前线程已经获取到了锁，那么在该线程代码块中是可以进行锁重入的，每一次锁重入就加1，最多重入的次数Integer.MAX_VALUE次，也就是2147483647。如果nextc<0,则抛出异常。  
接下来就进入到addWaiter方法了。

```
 private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
```
这段代码的意思是如果头节点不为空，则创建一个空的头结点然后将这个节点放在尾节点的下一个节点。  
否则进行入队列操作：

```
   private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```
这里可能出现的情况是有可能设置头节点的时候出现CPU切换了，此时有头节点了只需要将该node节点放置在头结点的下一个节点就可以看了。  
这一步走完了就直接走acquireQueued方法了。

```
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
这段代码是尝试从队列里面获取某个Node节点，进行阻塞操作。

```
   private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

```
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        unsafe.park(false, 0L);
        setBlocker(t, null);
    }
```
下面来看看如何进行unlock操作：

```
  public void unlock() {
        sync.release(1);
    }
```

```
   public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```
在release方法中，先调用tryRelease方法尝试进行释放锁。

```
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
```
每次拿到最新的state然后尝试进行减1操作，如果c等于0的时候则表示锁释放完毕，将当前持有锁设置NULL。
。
### 总结
- ReentrantLock通过大量采用AQS中的类方法来控制state变量的值的状态来使线程获取到锁。
- 它包括公平锁和非公平锁，其原理本质上都是相同的。





