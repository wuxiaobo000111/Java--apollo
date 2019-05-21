> 摘要: 原创出处 http://cmsblogs.com/?p=2210 「小明哥」欢迎转载，保留摘要，谢谢！

# 简介

>&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock，可重入锁，是一种递归无阻塞的同步机制。它可以等同于 synchronized 的使用，但是 ReentrantLock 提供了比 synchronized 更强大、灵活的锁机制，可以减少死锁发生的概率。
&nbsp;&nbsp;&nbsp;&nbsp;一个可重入的互斥锁定 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。
&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 #isHeldByCurrentThread() 和 #getHoldCount() 方法来检查此情况是否发生。
&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock 还提供了公平锁和非公平锁的选择，通过构造方法接受一个可选的 fair 参数（默认非公平锁）：当设置为 true 时，表示公平锁；否则为非公平锁。
&nbsp;&nbsp;&nbsp;&nbsp;公平锁与非公平锁的区别在于，公平锁的锁获取是有顺序的。但是公平锁的效率往往没有非公平锁的效率高，在许多线程访问的情况下，公平锁表现出较低的吞吐量。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/33.png?raw=true)

```text
ReentrantLock 实现 Lock 接口，基于内部的 Sync 实现。
Sync 实现 AQS ，提供了 FairSync 和 NonFairSync 两种实现。
```
# 如何使用
```java

package com.bobo.basic.thread.concurrent;

import sun.misc.Unsafe;

import java.lang.reflect.Field;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author wuxiaobo
 */
public class ReentrantLockTest implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();
    public static int i =0;
    @Override
    public void run() {
        for (int j=0; j< 100; j++) {
            lock.lock();
            lock.lock();
            try {
                i++;
            }finally {
                lock.unlock();
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws Exception {
        /*ReentrantLockTest lockTest = new ReentrantLockTest();
        Thread thread1= new Thread(lockTest);
        Thread thread2 = new Thread(lockTest);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(i);*/

        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);
        System.out.println(unsafe.pageSize());
    }
}


```

## Sync 抽象类

>&nbsp;&nbsp;&nbsp;&nbsp;abstract static class Sync extends AbstractQueuedSynchronizer继承了AbstractQueuedSynchronizer。它使用 AQS 的 state 字段，来表示当前锁的持有数量，从而实现可重入的特性。

### lock

```java
abstract void lock();

    执行锁。抽象了该方法的原因是，允许子类实现快速获得非公平锁的逻辑。
```

### nonfairTryAcquire

>&nbsp;&nbsp;&nbsp;&nbsp;非公平获取锁

```java
final boolean nonfairTryAcquire(int acquires) {
    //当前线程
    final Thread current = Thread.currentThread();
    //获取同步状态
    int c = getState();
    //state == 0,表示没有该锁处于空闲状态
    if (c == 0) {
        //获取锁成功，设置为当前线程所有
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //线程重入
    //判断锁持有的线程是否为当前线程
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


```text

    该方法主要逻辑：首先判断同步状态 state == 0 ?
        如果是，表示该锁还没有被线程持有，直接通过CAS获取同步状态。
          如果成功，返回 true 。
            否则，返回 false 。
        如果不是，则判断当前线程是否为获取锁的线程？
                 如果是，则获取锁，成功返回 true 。成功获取锁的线程，再次获取锁，这是增加了同步状态 state 。
                 通过这里的实现，我们可以看到上面提到的 “它使用 AQS 的 state 字段，来表示当前锁的持有数量，
            从而实现可重入的特性”。
                否则，返回 false 。
````

### tryRelease

```java
protected final boolean tryRelease(int releases) {
    // 减掉releases
    int c = getState() - releases;
    // 如果释放的不是持有锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // state == 0 表示已经释放完全了，其他线程可以获取同步状态了
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

### 其他方法


```java

// 是否当前线程独占
@Override
protected final boolean isHeldExclusively() {
    // While we must in general read state before owner,
    // we don't need to do so to check if current thread is owner
    return getExclusiveOwnerThread() == Thread.currentThread();
}

// 新生成条件
final ConditionObject newCondition() {
    return new ConditionObject();
}

// Methods relayed from outer class

// 获得占用同步状态的线程
final Thread getOwner() {
    return getState() == 0 ? null : getExclusiveOwnerThread();
}

// 获得当前线程持有锁的数量
final int getHoldCount() {
    return isHeldExclusively() ? getState() : 0;
}

// 是否被锁定
final boolean isLocked() {
    return getState() != 0;
}

/**
 * Reconstitutes the instance from a stream (that is, deserializes it).
 * 自定义反序列化逻辑
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    setState(0); // reset to unlocked state
}
```

##  NonfairSync

>&nbsp;&nbsp;&nbsp;&nbsp;NonfairSync 是 ReentrantLock 的内部静态类，实现 Sync 抽象类，非公平锁实现类。

### lock

```java
  final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
```

>&nbsp;&nbsp;&nbsp;&nbsp;首先基于 AQS state 进行 CAS 操作，将 0 => 1 。若成功，则获取锁成功。若失败，执行 AQS 的正常的同步状态获取逻辑。

### tryAcquire

```java
    protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
```

## FairSync

### lock


```java
  final void lock() {
            acquire(1);
        }
```

>&nbsp;&nbsp;&nbsp;&nbsp;直接执行 AQS 的正常的同步状态获取逻辑。

### tryAcquire

```java
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

>&nbsp;&nbsp;&nbsp;&nbsp;多了一个hasQueuedPredecessors方法。判断是否有前序节点，即自己不是首个等待获取同步状态的节点.


## Lock

>&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock类实现了Lock接口。Lock接口中的方法如下所示:


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/34.png?raw=true)


## ReentrantLock

### 构造方法

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
    基于 fair 参数，创建 FairSync 还是 NonfairSync 对象。
```

### lock

```java
@Override
public void lock() {
    sync.lock();
}
```

### lockInterruptibly

```java
@Override
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

### tryLock

```java
@Override
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

### tryLock

```java
@Override
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}

```


### unlock

```java
@Override
public void unlock() {
    sync.release(1);
}
```

### newCondition

```java
@Override
public Condition newCondition() {
    return sync.newCondition();
}
```


### 其他实现方法

```java
public int getHoldCount() {
    return sync.getHoldCount();
}
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
public boolean isLocked() {
    return sync.isLocked();
}


public final boolean isFair() {
    return sync instanceof FairSync;
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
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}
public int getWaitQueueLength(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
}
protected Collection<Thread> getWaitingThreads(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```
