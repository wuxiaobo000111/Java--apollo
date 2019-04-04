
# 概述

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

# 什么是可重入锁

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

# ReentrantLock和synchronized有什么区别和联系

|比较方面 |SynChronized | ReentrantLock（实现了 Lock接口）|
|--|--|--|
| 原始构成|它是java语言的关键字，是原生语法层面的互斥，需要jvm实现 | 它是JDK 1.5之后提供的API层面的互斥锁类|
|代码编写 | 采用synchronized不需要用户去手动释放锁，当synchronized方法或者synchronized代码块执行完之后，系统会自动让线程释放对锁的占用，更安全，| 而ReentrantLock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。需要lock()和unlock()方法配合try/finally语句块来完成，|
|灵活性 |锁的范围是整个方法或synchronized块部分 |Lock因为是方法调用，可以跨方法，灵活性更大 |
| 等待可中断|不可中断，除非抛出异常(释放锁方式：1.代码执行完，正常释放锁；2.抛出异常，由JVM退出等待) |持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待,(方法：1.设置超时方法 tryLock(long timeout, TimeUnit unit)，时间过了就放弃等待；2.lockInterruptibly()放代码块中，调用interrupt()方法可中断，而synchronized不行)！ |
|是否公平锁 |非公平锁 |	两者都可以，默认公平锁，构造器可以传入boolean值，true为公平锁，false为非公平锁， |
|条件Condition | | 通过多次newCondition可以获得多个Condition对象,可以简单的实现比较复杂的线程同步的功能.|
|提供的高级功能| | 提供很多方法用来监听当前锁的信息，如getHoldCount() getQueueLength()isFair()isHeldByCurrentThread()isLocked()|

# 补充(synchronized的实现原理)

>&nbsp;&nbsp;&nbsp;&nbsp;原始博客地址:https://blog.csdn.net/javazejian/article/details/72828483#理解java对象头与monitor 


# ReentrantLock源码分析


## 理解AbstractQueuedSynchronized(简称AQS)

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-04-03/1.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;具体的参考可以参考同级目录下的