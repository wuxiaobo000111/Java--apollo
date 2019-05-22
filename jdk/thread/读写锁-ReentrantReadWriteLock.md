>摘要: 原创出处 http://cmsblogs.com/?p=2213 「小明哥」欢迎转载，保留摘要，谢谢！

# 为何要使用读写锁

>&nbsp;&nbsp;&nbsp;&nbsp;重入锁 ReentrantLock 是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而，读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

# 读写锁的主要特征

```text
    公平性：支持公平性和非公平性。

    重入性：支持重入。读写锁最多支持 65535 个递归写入锁和 65535 个递归读取锁。

    锁降级：遵循获取写锁，再获取读锁，最后释放写锁的次序，如此写锁能够降级成为读锁。
```

# 如何使用


```java
package com.bobo.basic.thread.concurrent;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 读写锁
 * @create 2019-03-05 5:57
 **/
public class ReadWriteLockDemo {

    private static Lock lock = new ReentrantLock();

    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private static Lock readLock = readWriteLock.readLock();

    public static Lock wrieteLock = readWriteLock.writeLock();

    private int value;

    public Object handleRead(Lock lock) {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName()+"is reading");
            Thread.sleep(1000);
            return value;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return null;
    }

    public void handlewrite(Lock lock) {
        try {
            lock.lock();
            System.out.println(Thread.currentThread().getName()+"is write");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }


    public static void main(String[] args) {
        ReadWriteLockDemo demo = new ReadWriteLockDemo();
        Runnable readRunnable = new Runnable() {
            @Override
            public void run() {
                demo.handleRead(readLock);
            }
        };

        Runnable writeRunnable = new Runnable() {
            @Override
            public void run() {
                demo.handlewrite(wrieteLock);
                /*demo.handlewrite(lock);*/
            }
        };

        for (int i=0; i<18;i++) {
            new Thread(writeRunnable).start();
        }
        for (int i=18; i<20;i++) {
            new Thread(readRunnable).start();
        }
    }

}

```

# 源码解析

## 读写锁接口

```java
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

## ReentrantReadWriteLock


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/35.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;这里显示了ReentrantReadWriteLock的继承关系。可以看到ReentrantReadWriteLock实现了ReadWriteLock接口。在它内部，维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 Writer 线程，读取锁可以由多个 Reader 线程同时保持。也就说说，写锁是独占的，读锁是共享的。

```java
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


/** 内部类  读锁 */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** 内部类  写锁 */
private final ReentrantReadWriteLock.WriteLock writerLock;

final Sync sync;

/** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock() {
    this(false);
}

/** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

/** 返回用于写入操作的锁 */
@Override
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
/** 返回用于读取操作的锁 */
@Override
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

abstract static class Sync extends AbstractQueuedSynchronizer {
    /**
     * 省略其余源代码
     */
}
public static class WriteLock implements Lock, java.io.Serializable {
    /**
     * 省略其余源代码
     */
}

public static class ReadLock implements Lock, java.io.Serializable {
    /**
     * 省略其余源代码
     */
}



```


```text
    ReentrantReadWriteLock 与 ReentrantLock一样，其锁主体也是 Sync，它的读锁、写锁都是通过 Sync 来实现的。Sync继承了AQS这个抽象类。所以 
    ReentrantReadWriteLock 实际上只有一个锁，只是在获取读取锁和写入锁的方式上不一样。

    它的读写锁对应两个类：ReadLock 和 WriteLock 。这两个类都是 Lock 的子类实现。
```

>&nbsp;&nbsp;&nbsp;&nbsp;在 ReentrantLock 中，使用 Sync ( 实际是 AQS )的 int 类型的 state 来表示同步状态，表示锁被一个线程重复获取的次数。但是，读写锁 ReentrantReadWriteLock 内部维护着一对读写锁，如果要用一个变量维护多种状态，需要采用“按位切割使用”的方式来维护这个变量，将其切分为两部分：高16为表示读，低16为表示写。
&nbsp;&nbsp;&nbsp;&nbsp;分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过位运算。假如当前同步状态为S，那么：写状态，等于 S & 0x0000FFFF（将高 16 位全部抹去）。读状态，等于 S >>> 16 (无符号补 0 右移 16 位)。

```java

        /*
         * Read vs write count extraction constants and functions.
         * Lock state is logically divided into two unsigned shorts:
         * The lower one representing the exclusive (writer) lock hold count,
         * and the upper the shared (reader) hold count.
         */

        static final int SHARED_SHIFT   = 16;
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

        /** Returns the number of shared holds represented in count  */
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
        /** Returns the number of exclusive holds represented in count  */
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

        /** Returns the number of shared holds represented in count  */
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
        /** Returns the number of exclusive holds represented in count  */
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

```

```text
    #exclusiveCount(int c) 静态方法，获得持有写状态的锁的次数。
    #sharedCount(int c) 静态方法，获得持有读状态的锁的线程数量。不同于写锁，读锁可以同时被多个线程持有。而每个
    线程持有的读锁支持重入的特性，所以需要对每个线程持有的读锁的数量单独计数，这就需要用到 HoldCounter 计数器。
    详细解析，见 「6. HoldCounter」 。
```

## 构造方法

```java
     public ReentrantReadWriteLock() {
        this(false);
    }

   
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;在构造函数中只是初始化了三个参数。这里还是再次提示一下公平锁和非公平锁的概念。公平锁即先到先得。在等待队列中,如果排在队列前面的锁获取锁之后,排在队列后的才有可能获取线程。

## getThreadId


>&nbsp;&nbsp;&nbsp;&nbsp;#getThreadId(Thread thread) 静态方法，获得线程编号。代码如下：
```java
static final long getThreadId(Thread thread) {
    return UNSAFE.getLongVolatile(thread, TID_OFFSET);
}

// Unsafe mechanics
private static final sun.misc.Unsafe UNSAFE;
private static final long TID_OFFSET;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> tk = Thread.class;
        TID_OFFSET = UNSAFE.objectFieldOffset
            (tk.getDeclaredField("tid"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```
## Sync抽象类

>&nbsp;&nbsp;&nbsp;&nbsp;Sync 是 ReentrantReadWriteLock 的内部静态类，实现 AbstractQueuedSynchronizer 抽象类，同步器抽象类。它使用 AQS 的 state 字段，来表示当前锁的持有数量，从而实现可重入和读写锁的特性。

### 构造方法

```java

private transient ThreadLocalHoldCounter readHolds; // 当前线程的读锁持有数量

private transient Thread firstReader = null; // 第一个获取读锁的线程
private transient int firstReaderHoldCount; // 第一个获取读锁的重入数

private transient HoldCounter cachedHoldCounter; // 最后一个获得读锁的线程的 HoldCounter 的缓存对象

Sync() {
    readHolds = new ThreadLocalHoldCounter();
    setState(getState()); // ensures visibility of readHolds
}
```

### writerShouldBlock

```java
abstract boolean writerShouldBlock();

```

>&nbsp;&nbsp;&nbsp;&nbsp;获取写锁时，如果有前序节点也获得锁时，是否阻塞。NonefairSync 和 FairSync 下有不同的实现。在FairSync中调用了hasQueuedPredecessors()方法:是否有前序节点。在NonfairSync中直接返回一个false。

### readerShouldBlock

```java
abstract boolean readerShouldBlock();

```

>&nbsp;&nbsp;&nbsp;&nbsp;获取读锁时，如果有前序节点也获得锁时，是否阻塞。NonefairSync 和 FairSync 下有不同的实现。在FairSync调用了hasQueuedPredecessors()方法，是否有前序节点。在NonfairSync中调用了apparentlyFirstQueuedIsExclusive()方法，判断当前是否有写操作已经获取了锁。

### 写锁

```java


@Override
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    //当前锁个数
    int c = getState();
    //写锁
    int w = exclusiveCount(c);
    if (c != 0) {
        //c != 0 && w == 0 表示存在读锁
        //当前线程不是已经获取写锁的线程
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //超出最大范围
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        setState(c + acquires);
        return true;
    }
    // 是否需要阻塞
    if (writerShouldBlock() ||
            !compareAndSetState(c, c + acquires))
        return false;
    //设置获取锁的线程为当前线程
    setExclusiveOwnerThread(current);
    return true;
}





        protected final boolean tryAcquire(int acquires) {
        
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```

>&nbsp;&nbsp;&nbsp;&nbsp;该方法和 ReentrantLock 的 #tryAcquire(int arg) 大致一样，差别在判断重入时，增加了一项条件：读锁是否存在。因为要确保写锁的操作对读锁是可见的。如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。
调用 #writerShouldBlock() 抽象方法，若返回 true ，则获取写锁失败。


### 【读锁】tryAcquireShared

```java
protected final int tryAcquireShared(int unused) {
    //当前线程
    Thread current = Thread.currentThread();
    int c = getState();
    //exclusiveCount(c)计算写锁
    //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
    //存在锁降级问题，后续阐述
    if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
        return -1;
    //读锁
    int r = sharedCount(c);

    /*
     * readerShouldBlock():读锁是否需要等待（公平锁原则）
     * r < MAX_COUNT：持有线程小于最大数（65535）
     * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
     */
    if (!readerShouldBlock() &&
            r < MAX_COUNT &&
            compareAndSetState(c, c + SHARED_UNIT)) { //修改高16位的状态，所以要加上2^16
        /*
         * holdCount部分后面讲解
         */
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```


>&nbsp;&nbsp;&nbsp;&nbsp;#tryAcqurireShared(int arg) 方法，尝试获取读同步状态，获取成功返回 >= 0 的结果，否则返回 < 0 的结果。
&nbsp;&nbsp;&nbsp;&nbsp;因为存在锁降级情况，如果存在写锁且锁的持有者不是当前线程，则直接返回失败，否则继续。
&nbsp;&nbsp;&nbsp;&nbsp;依据公平性原则，调用 #readerShouldBlock() 方法来判断读锁是否不需要阻塞，读锁持有线程数小于最大值（65535），且 CAS 设置锁状态成功，执行以下代码（对于 HoldCounter ，见 「6. HoldCounter」 中），并返回 1 。如果不满足任一条件，则调用 #fullTryAcquireShared(Thread thread) 方法，详细解析，见 「4.5.1 fullTryAcquireShared」 中。

### 【写锁】tryRelease


```java
protected final boolean tryRelease(int releases) {
    //释放的线程不为锁的持有者
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    //若写锁的新线程数为0，则将锁的持有者设置为null
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;写锁释放锁的整个过程，和独占锁 ReentrantLock 相似，每次释放均是减少写状态，当写状态为 0 时，表示写锁已经完全释放了，从而让等待的其他线程可以继续访问读、写锁，获取同步状态。同时，此次写线程的修改对后续的线程可见。

### 【读锁】tryReleaseShared

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    //如果想要释放锁的线程为第一个获取锁的线程
    if (firstReader == current) {
        //仅获取了一次，则需要将firstReader 设置null，否则 firstReaderHoldCount - 1
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    }
    //获取rh对象，并更新“当前线程获取锁的信息”
    else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    //CAS更新同步状态
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```


###  tryWriteLock

>&nbsp;&nbsp;&nbsp;&nbsp;#tryWriteLock() 方法，尝试获取写锁。若获取成功，返回 true 。若失败，返回 false 即可，不进行等待排队。


```java
final boolean tryWriteLock(){
    Thread current = Thread.currentThread();
    int c = getState();
    if(c != 0){
        int w = exclusiveCount(c); // 获得现在写锁获取的数量
        if(w == 0 || current != getExclusiveOwnerThread()){  // 判断是否是其他的线程获取了写锁。若是，返回 false
            return false;
        }
        if(w == MAX_COUNT){ // 超过写锁上限，抛出 Error 错误
            throw new Error("Maximum lock count exceeded");
        }
    }

    if(!compareAndSetState(c, c + 1)){ //  CAS 设置同步状态，尝试获取写锁。若失败，返回 false
        return false;
    }
    setExclusiveOwnerThread(current); // 设置持有写锁为当前线程
    return true;
}
```

### tryReadLock

>&nbsp;#tryReadLock() 方法，尝试获取读锁。若获取成功，返回 true 。若失败，返回 false 即可，不进行等待排队。

```java
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
       //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题，后续阐述
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        // 读锁
        int r = sharedCount(c);
        /*
         * HoldCount 部分后面讲解
         */
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
4.10 isHeldExcl
```