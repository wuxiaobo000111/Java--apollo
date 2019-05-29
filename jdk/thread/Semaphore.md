>摘要: 原创出处 http://cmsblogs.com/?p=2263 「小明哥」欢迎转载，保留摘要，谢谢！

# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;信号量 Semaphore 是一个控制访问多个共享资源的计数器，和 CountDownLatch 一样，其本质上是一个“共享锁”。

```text

一个计数信号量。从概念上讲，信号量维护了一个许可集。
    如有必要，在许可可用前会阻塞每一个 acquire，然后再获取该许可。
    每个 release 添加一个许可，从而可能释放一个正在阻塞的获取者。
但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。


```


>&nbsp;下面我们就一个停车场的简单例子来阐述 Semaphore ：


```text


    为了简单起见我们假设停车场仅有 5 个停车位。一开始停车场没有车辆所有车位全部空着，然后先后到来三辆车
    ，停车场车位够，安排进去停车，然后又来三辆，这个时候由于只有两个停车位，所有只能停两辆，其余一辆必
    须在外面候着，直到停车场有空车位。当然，以后每来一辆都需要在外面候着。当停车场有车开出去，里面有空
    位了，则安排一辆车进去（至于是哪辆，要看选择的机制是公平还是非公平）。

    从程序角度看，停车场就相当于信号量 Semaphore ，其中许可数为 5 ，车辆就相对线程。当来一辆车时，许
    可数就会减 1 。当停车场没有车位了（许可数 == 0 ），其他来的车辆需要在外面等候着。如果有一辆车开出
    停车场，许可数 + 1，然后放进来一辆车。

    信号量 Semaphore 是一个非负整数（ >=1 ）。当一个线程想要访问某个共享资源时，它必须要先获取 Semaphore
    。当 Semaphore > 0 时，获取该资源并使 Semaphore – 1 。如果S emaphore 值 = 0，则表示全部的共享
    资源已经被其他线程全部占用，线程必须要等待其他线程释放资源。当线程释放资源时，Semaphore 则 +1 。
```


# 代码示例

```java
public class SemaphoreTest {

    static class Parking {
    
        //信号量
        private Semaphore semaphore;

        Parking(int count) {
            semaphore = new Semaphore(count);
        }

        public void park() {
            try {
                //获取信号量
                semaphore.acquire();
                long time = (long) (Math.random() * 10);
                System.out.println(Thread.currentThread().getName() + "进入停车场，停车" + time + "秒..." );
                Thread.sleep(time);
                System.out.println(Thread.currentThread().getName() + "开出停车场...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release();
            }
        }
    }


    static class Car extends Thread {
        Parking parking ;

        Car(Parking parking){
            this.parking = parking;
        }

        @Override
        public void run() {
            parking.park();     //进入停车场
        }
    }

    public static void main(String[] args){
        Parking parking = new Parking(3);

        for(int i = 0 ; i < 5 ; i++){
            new Car(parking).start();
        }
    }
}
```


# 源码分析

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/46.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;从上图可以看出，Semaphore 内部包含公平锁（FairSync）和非公平锁（NonfairSync），继承内部类 Sync ，其中 Sync 继承 AQS（再一次阐述 AQS 的重要性）。
&nbsp;&nbsp;&nbsp;&nbsp;Semaphore 提供了两个构造函数：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Semaphore(int permits) ：创建具有给定的许可数和非公平的公平设置的 Semaphore 。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Semaphore(int permits, boolean fair) ：创建具有给定的许可数和给定的公平设置的 Semaphore 。
&nbsp;&nbsp;&nbsp;&nbsp;实现如下：

```java

   public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }



    Semaphore 默认选择非公平锁。

    当信号量 Semaphore = 1 时，它可以当作互斥锁使用。其中 0、1 就相当于它的状态：1）当 =1 时表示，
    其他线程可以获取；2）当 =0 时，排他，即其他线程必须要等待。

    🙂 Semaphore 的代码实现结构，和 ReentrantLock 类似。
```

##  acquire

```java
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;内部调用 AQS 的 #acquireSharedInterruptibly(int arg) 方法，该方法以共享模式获取同步状态。代码如下：

```java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;在 #acquireSharedInterruptibly(int arg) 方法中，会调用 #tryAcquireShared(int arg) 方法。而 #tryAcquireShared(int arg) 方法，由子类来实现。对于 Semaphore 而言，如果我们选择非公平模式，则调用 NonfairSync 的#tryAcquireShared(int arg) 方法，否则调用 FairSync 的 #tryAcquireShared(int arg) 方法。若 #tryAcquireShared(int arg) 方法返回 < 0 时，则会阻塞等待，从而实现 Semaphore 信号量不足时的阻塞，代码如下：

```java
// AQS.java
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            /**
             * 对于 Semaphore 而言，如果 tryAcquireShared 返回小于 0 时，则会阻塞等待。
             */
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;公平情况的 FairSync 的方法实现，代码如下：

```java
// FairSync.java
@Override
protected int tryAcquireShared(int acquires) {
    for (;;) {
        //判断该线程是否位于CLH队列的列头，从而实现公平锁
        if (hasQueuedPredecessors())
            return -1;
        //获取当前的信号量许可
        int available = getState();

        //设置“获得acquires个信号量许可之后，剩余的信号量许可数”
        int remaining = available - acquires;

        //CAS设置信号量
        if (remaining < 0 ||
                compareAndSetState(available, remaining))
            return remaining;
    }
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;非公平情况的 NonfairSync 的方法实现，代码如下：

```java
// NonfairSync.java
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

// Sync.java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

## release 

>&nbsp;&nbsp;&nbsp;&nbsp;获取了许可，当用完之后就需要释放，Semaphore 提供 #release() 方法，来释放许可。代码如下：

```java
public void release() {
    sync.releaseShared(1);
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;内部调用 AQS 的 #releaseShared(int arg) 方法，释放同步状态。

```java
// AQS.java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;releaseShared(int arg) 方法，会调用 Semaphore 内部类 Sync 的 #tryReleaseShared(int arg) 方法，释放同步状态。

```java
// Sync.java
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        //信号量的许可数 = 当前信号许可数 + 待释放的信号许可数
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        //设置可获取的信号许可数为next
        if (compareAndSetState(current, next))
            return true;
    }
}
```