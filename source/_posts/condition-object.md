---
title: ConditionObject源码分析
date: 2018-03-14 16:02:43
tags:
  -  Java  
categories:  
  -  Java多线程集合   
type: tags
---
### ConditionObject源码分析
ConditionObject是AbstractQueuedSynchronizer中的一部分，它是用来实现通知/等待机制的，和Object的wait/notify一样，Codition分为两个部分，分别为await和signal，前者应用为等待，后者用于唤醒。  
首先看看await方法的实现：
```
        public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            Node node = addConditionWaiter();
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```
其中有个addConditionWaiter方法，来看看它的实现：

```
    private Node addConditionWaiter() {
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            if (t != null && t.waitStatus != Node.CONDITION) {
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
```
这段代码主要的意思创建一个单向链表，每次调用都创建一个新的Node到尾部。而firstWaiter为第一个头部的Node。
fullyRelease方法是用来释放锁的，因为await方法之后需要将所有的Node的锁释放掉让其它的线程去竞争锁。

```
    final int fullyRelease(Node node) {
        boolean failed = true;
        try {
            int savedState = getState();
            if (release(savedState)) {
                failed = false;
                return savedState;
            } else {
                throw new IllegalMonitorStateException();
            }
        } finally {
            if (failed)
                node.waitStatus = Node.CANCELLED;
        }
    }
```
其本质还是调用了release方法释放AQS队列中的阻塞队列。  
有一个isOnSyncQueue方法的调用：

```
 final boolean isOnSyncQueue(Node node) {
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
        if (node.next != null) // If has successor, it must be on queue
            return true;
        return findNodeFromTail(node);
    }
```
该方法是判断当前线程是否在AQS队列中，如果不在，则将当前线程阻塞。然后调用acquireQueued方法不断尝试去获取锁。

#### Condition的signal()实现原理
那么线程是如何被通知往下执行的？另外一个线程调用singal方法时：

```
     public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }
```
首先判断当前线程是不是持有锁，没有持有锁则抛出IllegalMonitorStateException异常。接着调用doSignal方法。

```
     private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
```
这里主要有一个transferForSignal方法：

```
    final boolean transferForSignal(Node node) {
          if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```
这段代码直接加入到AQS队列中，如果该节点的状态为cancel或者CAS操作失败则直接唤醒该线程。
#### 实例
假如有2个线程先用调用来演示唤醒机制，其流程可能是这样子的。
1. 线程1调用ReentrantLock的lock方法时，优先获取到锁，不需要进入AQS队列。
2. 线程2调用ReentrantLock的lock方法时，由于线程1已经获取到锁时，线程2只能进入AQS队列
3. 线程1调用condition.await方法时，线程1释放锁并且无限轮询准备下一次获取锁且将线程1进入condition队列。
4. 线程1释放锁时，会触发AQS队列拿出线程2，使的线程2运行condition.signal方法，此时会将condition队列中的1个元素取出来放入AQS队列的末尾。
5. 此时线程2调用ReentrantLock的lock方法时，触发线程1去获取锁（由于AQS中只有一个Node），就直接出队列了。
### 总结
- Condition创建的头结点就是真正等待的节点，而AQS创建的头结点是没有Thread的空节点。
- Condition创建的头结点使用nextWaiter来标识下一个节点，而AQS创建的节点有next和pred相互标识该节点的前一个节点和后一个节点。











