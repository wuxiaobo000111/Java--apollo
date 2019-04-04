# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。该同步器（以下简称同步器）利用了一个int来表示状态，期望它能够成为实现大部分同步需求的基础。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似acquire和release的方式来操纵状态。然而多线程环境中对状态的操纵必须确保原子性，因此子类对于状态的把握，需要使用这个同步器提供的以下三个方法对状态进行操作：

```java
java.util.concurrent.locks.AbstractQueuedSynchronizer.getState()
java.util.concurrent.locks.AbstractQueuedSynchronizer.setState(int)
java.util.concurrent.locks.AbstractQueuedSynchronizer.compareAndSetState(int, int)
```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-03/2.png?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;AQS定义两种资源共享方式：Exclusive（独占，只有一个线程能执行，如ReentrantLock）和Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）。

>&nbsp;&nbsp;&nbsp;&nbsp;不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

```java
isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
```

# 节点

```java
static final class Node {
        //共享锁节点
        static final Node SHARED = new Node();
        //排他锁节点
        static final Node EXCLUSIVE = null;

        // CANCELLED表示该线程不参与竞争了，只有CANCELLED>0，也只有这个状态是表示没用的node
        static final int CANCELLED =  1;
         // 表示该节点的下一个节点需要被唤醒
        static final int SIGNAL    = -1;
        //表示当前节点在等待condition，即在condition队列中；
        static final int CONDITION = -2;
        //表示releaseShared需要被传播给后续节点（仅在共享模式下使用）
        static final int PROPAGATE = -3;

        
        volatile int waitStatus;

        //前继节点
        volatile Node prev;

        //后继节点
        volatile Node next;

        //该节点所对应的线程
        volatile Thread thread;

        //下一个等待condition的节点
        Node nextWaiter;

        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }

        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }

        Node() {    // Used to establish initial head or SHARED marker
        }

        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }

        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

# acquire

```java
     /**
     * 获取排它锁，忽略中断。方法会调用tryAcquire来获取锁，如果没有获取，则会将当前线程
     * 添加到阻塞队列中，期间可能会被不断阻塞，唤醒并调用tryAcquire，直到tryAcquire获取到锁
     */
    public final void acquire(int arg) {
        // 先调用tryAcquire尝试获取一次，如果获取到锁，则返回true
        // 如果tryAcquire返回false，则将当前线程加入阻塞队列，并不断尝试获取锁
        // 如果在排队过程中被中断，那acquireQueued返回true，那么会执行selfInterrupt中断当前线程。
        // 这个方法一般是由子类实现,一般实现都有fair和unfair两种策略       
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
     protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }

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
    static void selfInterrupt() {
        Thread.currentThread().interrupt();
    }

    //将当前线程与mode打包成一个Node，并添加到等待队列的队尾。
   private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            // // 使用CAS尝试将当前节点添加到队尾
            Node pred = tail;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
         // 如果队列没有初始化或者添加失败，则调用enq方法
        enq(node);
        return node;
    }
      /**
       
     * CAS tail field. Used only by enq.
     */
    private final boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }

    /**
     * * 这里使用经典的轮询CAS方式，将node添加到队尾。如果队列没有初始化，顺便初始化队列。
     * /
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
    //  为一个已经在等待队列中的Node获取排它锁
     final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                 // 获取该节点的前驱节点
                final Node p = node.predecessor();
                  // 如果前驱节点是head，并且获取到了锁，则将前驱节点删除
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                  // 寻找休息点，即从当前节点往head的方向找，跳过所有waitStatus为CANCELLED的节点，直接link到最近的waitStatus为SIGNAL的节点后面。
                // 找到休息点之后，将当前线程park起来，并返回Thread.interrupted()。
                // 如果parkAndCheckInterrupt返回true，那将interrupted置位true。
                // ********这里会被release唤醒，然后重新竞争
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

# release

```java
    /**
     * 释放一个排它锁。
     */
    public final boolean release(int arg) {
        // 如果成功释放锁
        if (tryRelease(arg)) {
            // 获取head node，waitStatus == 0 表示空队列
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);// 唤醒下一个node
            return true;
        }
        return false;
    }

    /**
     * 由子类实现
     */
    protected boolean tryRelease(int arg) {
        throw new UnsupportedOperationException();
    }


    /**
     * 唤醒node的后继节点
     *
     */
    private void unparkSuccessor(Node node) {
        // 如果waitStatus<0，则先将node的waitStatus置成0，因为node是需要release的node
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);


        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;

            // 从tail往node遍历，找一个waitStatus<0的有效节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }

        // 如果找到了需要唤醒的节点，则唤醒。
        // ********这里唤醒了之后，仍然需要争抢锁，如果没抢到，还是会继续等待下一次唤醒。
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

# acquireShared

```java
    /**
     * 获取共享锁
     *
     */
    public final void acquireShared(int arg) {
        // 若果没获取到锁，tryAcquireShared返回值<0，则将当前线程加入等待队列
        // 如果当前线程获取到了，但是没有其它线程可以获取了(理解为信号量用完了)，tryAcquireShared返回0
        // 如果当前线程获取到了，其它线程也可以获取(理解为信号量没用完)，tryAcquireShared返回值>0
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }

    /**
    * 尝试获取共享锁，由子类重写。
    */
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }

     /**
     * 在队列中获取共享锁
     */
    private void doAcquireShared(int arg) {
        // 将当前线程创建一个mode为SHARED的node
        final Node node = addWaiter(Node.SHARED);

        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    // 如果获取到了，将p出队，使当前节点成为head，并让后继节点也尝试获取。这是与抢占式的区别。
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
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

    /**
     * 将node设置为head，并且如果propagate>0，则继续唤醒后继节点
     */
    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();// 唤醒后继节点
        }
    }

    /**
     * 唤醒后继节点，并传播下去
     */
    private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                //如果head的状态为SIGNAL，尝试将其状态变成0，并唤醒后继节点
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue; 
                    unparkSuccessor(h);
                }
                //如果head的状态为0，则将其状态变成PROPAGATE
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;           
            }
            if (h == head)//head发送过改变，则循环；如果head不变，则break
                break;
        }
    }

```

# releaseShared
```java
    /**
     * 释放共享锁
     */
    public final boolean releaseShared(int arg) {
        // tryReleaseShared释放成功返回true
        if (tryReleaseShared(arg)) {
            doReleaseShared(); //执行子类重写的doReleaseShared方法
            return true;
        }
        return false;
    }
```