


# 1. 概述

>&nbsp;&nbsp;&nbsp;&nbsp;synchronized关键字相信大家都已经会使用了,这里来介绍一些JDK中实现的锁--ReentrantLock。首先ReentrantLock可以实现synchronized的所有功能,而且ReentrantLock是通过java API的方式来操作锁。同时ReentrantLock是可重入锁。之后会一一解释ReentrantLock的用法。下面给出一个ReentrantLock的小例子:
```java
public class ReentrantLockTest implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();
    public static int i =0;
    @Override
    public void run() {
        for (int j=0; j< 100; j++) {
            lock.lock();
            try {
                i++;
            }finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReentrantLockTest lockTest = new ReentrantLockTest();
        Thread thread1= new Thread(lockTest);
        Thread thread2 = new Thread(lockTest);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(i);
    }
}


```

>&nbsp;&nbsp;&nbsp;&nbsp;上述代码给出了使用ReentrantLock的一个小例子。是不是觉得很easy呢。

# 2. 什么是可重入锁

>&nbsp;&nbsp;&nbsp;&nbsp;可重入性描述这样的一个问题：一个线程在持有一个锁的时候，它内部能否再次（多次）申请该锁。如果一个线程已经获得了锁，其内部还可以多次申请该锁成功。那么我们就称该锁为可重入锁。通过以下代码说明：

```java
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
```
>&nbsp;&nbsp;&nbsp;&nbsp;我们对上面的代码稍加以改造,就可以得到一个重入锁。虽然这个重入锁并没有任何意义,只想从代码层面可以说明可以这么实现。可以认为是锁上加锁,同时解锁的时候也要相应的做对应的解锁。(大家可以联想一下从家里出门和回家的场景,就可以很简单的理解了)。

# 3. ReentrantLock和synchronized有什么区别和联系

|比较方面 |SynChronized | ReentrantLock（实现了 Lock接口）|
|--|--|--|
| 原始构成|它是java语言的关键字，是原生语法层面的互斥，需要jvm实现 | 它是JDK 1.5之后提供的API层面的互斥锁类|
|代码编写 | 采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用，更安全，| 而ReentrantLock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。需要lock()和unlock()方法配合try/finally语句块来完成，|
|灵活性 |锁的范围是整个方法或synchronized块部分 |Lock因为是方法调用，可以跨方法，灵活性更大 |
| 等待可中断|不可中断，除非抛出异常(释放锁方式：1.代码执行完，正常释放锁；2.抛出异常，由JVM退出等待) |持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待,(方法：1.设置超时方法 tryLock(long timeout, TimeUnit unit)，时间过了就放弃等待；2.lockInterruptibly()放代码块中，调用interrupt()方法可中断，而synchronized不行)！ |
|是否公平锁 |非公平锁 |	两者都可以，默认公平锁，构造器可以传入boolean值，true为公平锁，false为非公平锁， |
|条件Condition | | 通过多次newCondition可以获得多个Condition对象,可以简单的实现比较复杂的线程同步的功能.|
|提供的高级功能| | 提供很多方法用来监听当前锁的信息，如getHoldCount() getQueueLength()isFair()isHeldByCurrentThread()isLocked()|

# 4. 补充(synchronized的实现原理)

>&nbsp;&nbsp;&nbsp;&nbsp;原始博客地址:https://blog.csdn.net/javazejian/article/details/72828483#理解java对象头与monitor 


# 5. ReentrantLock源码分析


## 5.1. 理解AbstractQueuedSynchronized(简称AQS)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-03/1.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;具体的参考 [理解AbstractQueuedSynchronized](https://github.com/wuxiaobo000111/markdown/blob/master/jdk/thread/%E7%90%86%E8%A7%A3AbstractQueuedSynchronized.md "理解AbstractQueuedSynchronized")。如果大家理解了AQS之后,我们通过源代码的形式来分析一下ReentrantLock的内部是如何实现的。

# 6. Sync

>&nbsp;&nbsp;&nbsp;&nbsp;这个类实际上是继承了AbstractQueuedSynchronizer(文章中都简写为AQS)。这个类是公平锁和非公平锁继承的类。
```java
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        //由其子类实现
        abstract void lock();

        //非公平锁去获取锁的实现方式 
        final boolean nonfairTryAcquire(int acquires) {
            //获取当前线程
            final Thread current = Thread.currentThread();
            //getState方法是在AQS中的实现,getState获取当前线程的状态
            int c = getState();
            //如果为0(证明该独占锁已被释放，当下没有线程在使用)
            // 也就是AQS中的那个状态
            if (c == 0) {
                // 如果CAS尝试成功
                if (compareAndSetState(0, acquires)) {
                    //则当前的线程获取到锁
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
              else if (current == getExclusiveOwnerThread()) {//查看当前线程是不是就是独占锁的线程
                int nextc = c + acquires;//如果是，锁状态的数量为当前的锁数量+1
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);//设置当前的锁数量
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
    }   
```
## 6.1. 补充(公平锁和非公平锁有何区别)
```text
公平锁（Fair）：加锁前检查是否有排队等待的线程，优先排队等待的线程，先来先得
非公平锁（Nonfair）：加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待
```
# 7. NonfairSync

```java
 static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

      
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

```text
1）首先基于CAS将state（锁数量）从0设置为1，如果设置成功，设置当前线程为独占锁的线程；-->请求成功-->第一次插队
2）如果设置失败(即当前的锁数量可能已经为1了，即在尝试的过程中，已经被其他线程先一步占有了锁)，这个时候当前线程执行acquire(1)方法
2.1）acquire(1)方法首先调用下边的tryAcquire(1)方法，在该方法中，首先获取锁数量状态，
2.1.1）如果为0(证明该独占锁已被释放，当下没有线程在使用)，这个时候我们继续使用CAS将state（锁数量）从0设置为1，如果设置成功，当前线程独占锁；-->请求成功-->第二次插队；当然，如果设置不成功，直接返回false
2.2.2）如果不为0，就去判断当前的线程是不是就是当下独占锁的线程，如果是，就将当前的锁数量状态值+1（这也就是可重入锁的名称的来源）-->请求成功
下边的流程一句话：请求失败后，将当前线程链入队尾并挂起，之后等待被唤醒。
2.2.3）如果最后在tryAcquire(1)方法中上述的执行都没成功，即请求没有成功，则返回false，继续执行acquireQueued(addWaiter(Node.EXCLUSIVE), arg)方法
2.2）在上述方法中，首先会使用addWaiter(Node.EXCLUSIVE)将当前线程封装进Node节点node，然后将该节点加入等待队列（先快速入队，如果快速入队不成功，其使用正常入队方法无限循环一直到Node节点入队为止）
2.2.1）快速入队：如果同步等待队列存在尾节点，将使用CAS尝试将尾节点设置为node，并将之前的尾节点插入到node之前
2.2.2）正常入队：如果同步等待队列不存在尾节点或者上述CAS尝试不成功的话，就执行正常入队（该方法是一个无限循环的过程，即直到入队为止）-->第一次阻塞
2.2.2.1）如果尾节点为空（初始化同步等待队列），创建一个dummy节点，并将该节点通过CAS尝试设置到头节点上去，设置成功的话，将尾节点也指向该dummy节点（即头节点和尾节点都指向该dummy节点）
2.2.2.1）如果尾节点不为空，执行与快速入队相同的逻辑，即使用CAS尝试将尾节点设置为node，并将之前的尾节点插入到node之前
最后，如果顺利入队的话，就返回入队的节点node，如果不顺利的话，无限循环去执行2.2)下边的流程，直到入队为止
2.3）node节点入队之后，就去执行acquireQueued(final Node node, int arg)（这又是一个无限循环的过程，这里需要注意的是，无限循环等于阻塞，多个线程可以同时无限循环--每个线程都可以执行自己的循环，这样才能使在后边排队的节点不断前进）
2.3.1）获取node的前驱节点p，如果p是头节点，就继续使用tryAcquire(1)方法去尝试请求成功，-->第三次插队（当然，这次插队不一定不会使其获得执行权，请看下边一条），
2.3.1.1）如果第一次请求就成功，不用中断自己的线程，如果是之后的循环中将线程挂起之后又请求成功了，使用selfInterrupt()中断自己
（注意p==head&&tryAcquire(1)成功是唯一跳出循环的方法，在这之前会一直阻塞在这里，直到其他线程在执行的过程中，不断的将p的前边的节点减少，直到p成为了head且node请求成功了--即node被唤醒了，才退出循环）
2.3.1.2）如果p不是头节点，或者tryAcquire(1)请求不成功，就去执行shouldParkAfterFailedAcquire(Node pred, Node node)来检测当前节点是不是可以安全的被挂起，
2.3.1.2.1）如果node的前驱节点pred的等待状态是SIGNAL（即可以唤醒下一个节点的线程），则node节点的线程可以安全挂起，执行2.3.1.3）
2.3.1.2.2）如果node的前驱节点pred的等待状态是CANCELLED，则pred的线程被取消了，我们会将pred之前的连续几个被取消的前驱节点从队列中剔除，返回false（即不能挂起），之后继续执行2.3）中上述的代码
2.3.1.2.3）如果node的前驱节点pred的等待状态是除了上述两种的其他状态，则使用CAS尝试将前驱节点的等待状态设为SIGNAL，并返回false（因为CAS可能会失败，这里不管失败与否，都返回false，下一次执行该方法的之后，pred的等待状态就是SIGNAL了），之后继续执行2.3）中上述的代码
2.3.1.3）如果可以安全挂起，就执行parkAndCheckInterrupt()挂起当前线程，之后，继续执行2.3）中之前的代码
最后，直到该节点的前驱节点p之前的所有节点都执行完毕为止，我们的p成为了头节点，并且tryAcquire(1)请求成功，跳出循环，去执行。
（在p变为头节点之前的整个过程中，我们发现这个过程是不会被中断的）
2.3.2）当然在2.3.1）中产生了异常，我们就会执行cancelAcquire(Node node)取消node的获取锁的意图。
```

# 8. FairSync

```java
 static final class FairSync extends Sync {

        final void lock() {
            acquire(1);
        }

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
    }
获取一次锁数量，

B1、如果锁数量为0，如果当前线程是等待队列中的头节点，基于CAS尝试将state（锁数量）从0设置为1一次，如果设置成功，设置当前线程为独占锁的线程；

B2、如果锁数量不为0或者当前线程不是等待队列中的头节点或者上边的尝试又失败了，查看当前线程是不是已经是独占锁的线程了，如果是，则将当前的锁数量+1；如果不是，则将该线程封装在一个Node内，并加入到等待队列中去。等待被其前一个线程节点唤醒。

 
```