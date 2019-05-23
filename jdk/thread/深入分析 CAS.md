>摘要: 原创出处 http://cmsblogs.com/?p=2235 「小明哥」欢迎转载，保留摘要，谢谢！

# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;CAS，Compare And Swap，即比较并交换。Doug lea大神在同步组件中大量使用CAS技术鬼斧神工地实现了Java多线程的并发操作。整个AQS同步组件、Atomic原子类操作等等都是以CAS实现的，甚至ConcurrentHashMap在1.8的版本中也调整为了CAS+Synchronized。可以说CAS是整个JUC的基石。

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/36.png?raw=true)


# CAS分析

>&nbsp;&nbsp;&nbsp;&nbsp;在CAS中有三个参数：内存值V、旧的预期值A、要更新的值B，当且仅当内存值V的值等于旧的预期值A时才会将内存值V的值修改为B，否则什么都不干。其伪代码如下：

```java
if(this.value == A){
    this.value = B
    return true;
}else{
    return false;
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;JUC下的atomic类都是通过CAS来实现的，下面就以AtomicInteger为例来阐述CAS的实现。如下：

```java
  private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```


>&nbsp;&nbsp;&nbsp;&nbsp;Unsafe是CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe，它提供了硬件级别的原子操作。
&nbsp;&nbsp;&nbsp;&nbsp;valueOffset为变量值在内存中的偏移地址，unsafe就是通过偏移地址来得到数据的原值的。
&nbsp;&nbsp;&nbsp;&nbsp;value当前值，使用volatile修饰，保证多线程环境下看见的是同一个。
&nbsp;&nbsp;&nbsp;&nbsp;我们就以AtomicInteger的addAndGet()方法来做说明，先看源代码：

```java
/**
     * Atomically adds the given value to the current value.
     *
     * @param delta the value to add
     * @return the updated value
     */
    public final int addAndGet(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
    }

    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;该方法为本地方法，有四个参数，分别代表：对象、对象的地址、预期值、修改值（有位伙伴告诉我他面试的时候就问到这四个变量是啥意思…）。该方法的实现这里就不做详细介绍了，有兴趣的伙伴可以看看openjdk的源码。
&nbsp;&nbsp;&nbsp;&nbsp;CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，但是在多处理器上实现就有点儿复杂了。
&nbsp;&nbsp;&nbsp;&nbsp;CPU提供了两种方法来实现多处理器的原子操作：总线加锁或者缓存加锁。

&nbsp;&nbsp;&nbsp;&nbsp;CPU提供了两种方法来实现多处理器的原子操作：总线加锁或者缓存加锁。

```text

总线加锁：总线加锁就是就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住
,那么该处理器可以独占使用共享内存。但是这种处理方式显得有点儿霸道，不厚道，他把CPU和内存之间的通信锁住了，在锁定期间
，其他处理器都不能其他内存地址的数据，其开销有点儿大。所以就有了缓存加锁。

缓存加锁：其实针对于上面那种情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可。缓存加锁就是缓存在内存区
域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协
议来保证原子性。缓存一致性机制可以保证同一个内存区域的数据仅能被一个处理器修改，也就是说当CPU1修改缓存行中的i时使用
缓存锁定，那么CPU2就不能同时缓存了i的缓存行。
```


# CAS的缺陷

>&nbsp;&nbsp;&nbsp;&nbsp;带来的三个问题

```text
    循环时间太长

        如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。
        在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。

    只能保证一个共享变量原子操作

        看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量
        整成一个变量，利用CAS也不错。例如读写锁中state的高地位

    ABA问题

        CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，
        变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的
        ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，
        变成1A —> 2B —> 3A。

        CAS的ABA隐患问题，解决方案则是版本号，Java提供了AtomicStampedReference来解决。AtomicStampedReference通
        过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。
```