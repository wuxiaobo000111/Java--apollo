>摘要: 原创出处 http://cmsblogs.com/?p=2241 「小明哥」欢迎转载，保留摘要，谢谢！


# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;CyclicBarrier ，一个同步辅助类，在 AP I中是这么介绍的:它允许一组线程互相等待，直到到达某个公共屏障点 (Common Barrier Point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 Barrier 在释放等待线程后可以重用，所以称它为循环( Cyclic ) 的 屏障( Barrier ) 。
&nbsp;&nbsp;&nbsp;&nbsp;通俗点讲就是：让一组线程到达一个屏障时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。


# 示例代码

```java

public class CyclicBarrierTest {

    public static void main(String[] args) throws InterruptedException {

        CyclicBarrier cyclicBarrier = new CyclicBarrier(2);
        //通过barrier.getParties()获取开启屏障的方数
        System.out.println("barrier.getParties()获取开启屏障的方数：" + cyclicBarrier.getParties());
        System.out.println();
        //通过barrier.getNumberWaiting()获取正在等待的线程数
        System.out.println("通过barrier.getNumberWaiting()获取正在等待的线程数：初始----" + cyclicBarrier.getNumberWaiting());
        System.out.println();


        new Thread(() -> {
            //添加一个等待线程
            System.out.println("添加第1个等待线程----" + Thread.currentThread().getName());
            try {
                cyclicBarrier.await();
                System.out.println(Thread.currentThread().getName() + " is running...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is terminated.");
        }).start();
        Thread.sleep(10);

        System.out.println("通过barrier.getNumberWaiting()获取正在等待的线程数：添加第1个等待线程---" + cyclicBarrier.getNumberWaiting());
        Thread.sleep(10);

        System.out.println();
        new Thread(() -> {
            //添加一个等待线程
            System.out.println("添加第2个等待线程----" + Thread.currentThread().getName());
            try {
                cyclicBarrier.await();
                System.out.println(Thread.currentThread().getName() + " is running...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is terminated.");
        }).start();
        Thread.sleep(100);
        System.out.println();
        //通过barrier.getNumberWaiting()获取正在等待的线程数
        System.out.println("通过barrier.getNumberWaiting()获取正在等待的线程数：打开屏障之后---" + cyclicBarrier.getNumberWaiting());



        //已经打开的屏障，再次有线程等待的话，还会重新生效--视为循环
        new Thread(() -> {
            System.out.println("屏障打开之后，再有线程加入等待：" + Thread.currentThread().getName());
            try {
                //BrokenBarrierException
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is terminated.");

        }).start();
        System.out.println();
        Thread.sleep(10);
        System.out.println("通过barrier.getNumberWaiting()获取正在等待的线程数：打开屏障之后---" + cyclicBarrier.getNumberWaiting());
        Thread.sleep(10);
        new Thread(() -> {
            System.out.println("屏障打开之后，再有线程加入等待：" + Thread.currentThread().getName());
            try {
                //BrokenBarrierException
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " is terminated.");

        }).start();
        Thread.sleep(10);
        System.out.println("通过barrier.getNumberWaiting()获取正在等待的线程数：打开屏障之后---" + cyclicBarrier.getNumberWaiting());
    }



    public void  test01 () {

        //数组大小
        int size = 50000;
        //定义数组
        int[] numbers = new int[size];
        //随机初始化数组
        for (int i = 0; i < size; i++) {
            Random random = new Random();
            numbers[i] = random.nextInt(1000);
        }

        //单线程计算结果
        System.out.println();
        Long sum = 0L;
        for (int i = 0; i < size; i++) {
            sum += numbers[i];
        }
        System.out.println("单线程计算结果：" + sum);

        //多线程计算结果
        //定义线程池
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        //定义五个Future去保存子数组计算结果
        final int[] results = new int[5];

        //定义一个循环屏障，在屏障线程中进行计算结果合并
        CyclicBarrier barrier = new CyclicBarrier(5, () -> {
            int sums = 0;
            for (int i = 0; i < 5; i++) {
                sums += results[i];
            }
            System.out.println("多线程计算结果：" + sums);
        });

        //子数组长度
        int length = 10000;
        //定义五个线程去计算
        for (int i = 0; i < 5; i++) {
            //定义子数组
            int[] subNumbers = Arrays.copyOfRange(numbers, (i * length), ((i + 1) * length));
            //盛放计算结果
            int finalI = i;
            executorService.submit(() -> {
                for (int j = 0; j < subNumbers.length; j++) {
                    results[finalI] += subNumbers[j];
                }
                //等待其他线程进行计算
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
        //关闭线程池
        executorService.shutdown();
    }
}
```
# 实现分析

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/43.png?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;通过上图，我们可以看到 CyclicBarrier 的内部是使用重入锁 ReentrantLock 和 Condition 。
&nbsp;&nbsp;&nbsp;&nbsp;它有两个构造函数：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CyclicBarrier(int parties)：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CyclicBarrier(int parties, Runnable barrierAction) ：创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。

```java

    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }


    public CyclicBarrier(int parties) {
        this(parties, null);
    }


    parties 变量，表示拦截线程的总数量。

    count 变量，表示拦截线程的剩余需要数量。

    barrierAction 变量，为 CyclicBarrier 接收的 Runnable 命令，用于在线程到达屏障时，
    优先执行 barrierAction ，用于处理更加复杂的业务场景。

    generation 变量，表示 CyclicBarrier 的更新换代。
```

## await方法

```java
    public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }

   //await(long timeout, TimeUnit unit) 方法，在 #await() 的基础上，增加了等待超时的特性。代码如下：
    public int await(long timeout, TimeUnit unit)
        throws InterruptedException,
               BrokenBarrierException,
               TimeoutException {
        return dowait(true, unit.toNanos(timeout));
    }

```

>&nbsp;&nbsp;&nbsp;&nbsp;或者说，每个线程调用 #await() 方法，告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。当所有线程都到达了屏障，结束阻塞，所有线程可继续执行后续逻辑。
&nbsp;&nbsp;&nbsp;&nbsp;内部调用 #dowait(boolean timed, long nanos) 方法，执行阻塞等待( timed=true )。

## dowait

```java
 private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        //获取锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //generation 变量，表示 CyclicBarrier 的更新换代。
            final Generation g = generation;
            //当前generation“已损坏”，抛出BrokenBarrierException异常
            //抛出该异常一般都是某个线程在等待某个处于“断开”状态的CyclicBarrie
            if (g.broken)
            //当某个线程试图等待处于断开状态的 barrier 时，或者 barrier
            // 进入断开状态而线程处于等待状态时，抛出该异常
                throw new BrokenBarrierException();
            // 如果线程终端
            if (Thread.interrupted()) {
                breakBarrier();
                throw new InterruptedException();
            }
            // 进来一个线程,count就减少一
            int index = --count;
            //count == 0 表示所有线程均已到位，触发Runnable任务
            if (index == 0) {  // tripped
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    //触发任务
                    if (command != null)
                        command.run();
                    ranAction = true;
                    //唤醒所有等待线程,更新generation
                    nextGeneration();
                    return 0;
                } finally {
                    // 未执行，说明 barrierCommand 执行报错，或者线程打断等等情况。
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // loop until tripped, broken, interrupted, or timed out
            for (;;) {
                try {
                    //如果不是超时等待，则调用Condition.await()方法等待
                    if (!timed)
                        trip.await();
                    else if (nanos > 0L)
                        //超时等待，调用Condition.awaitNanos()方法等待
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // We're about to finish waiting even if we had not
                        // been interrupted, so this interrupt is deemed to
                        // "belong" to subsequent execution.
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

              //generation已经更新，返回index
                if (g != generation)
                    return index;

                 //“超时等待”，并且时间已到,终止CyclicBarrier，并抛出异常
                 if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                 }
            }
        } finally {
            lock.unlock();
        }
    }

```

## Generation 

>&nbsp;Generation 是 CyclicBarrier 内部静态类，描述了 CyclicBarrier 的更新换代。在CyclicBarrier中，同一批线程属于同一代。当有 parties 个线程全部到达 barrier 时，generation 就会被更新换代。其中 broken 属性，标识该当前 CyclicBarrier 是否已经处于中断状态。代码如下：


```java
private static class Generation {
    boolean broken = false;
}
```

## breakBarrier

>&nbsp;&nbsp;&nbsp;&nbsp;当 barrier 损坏了，或者有一个线程中断了，则通过 #breakBarrier() 方法，来终止所有的线程。代码如下：

```java
private void breakBarrier() {
    generation.broken = true;
    count = parties;
    trip.signalAll();
}
```

>&nbsp;nbsp;nbsp;nbsp;在 breakBarrier() 方法中，中除了将 broken设置为 true ，还会调用 #signalAll() 方法，将在 CyclicBarrier 处于等待状态的线程全部唤醒。

## nextGeneration

>&nbsp;&nbsp;&nbsp;&nbsp;当所有线程都已经到达 barrier 处（index == 0），则会通过 nextGeneration() 方法，进行更新换代操作。在这个步骤中，做了三件事：

```text
唤醒所有线程。
重置 count 。
重置 generation 。
```

```java
private void nextGeneration() {
    trip.signalAll();
    count = parties;
    generation = new Generation();
}
```

## reset

>&nbsp;&nbsp;&nbsp;&nbsp;#reset() 方法，重置 barrier 到初始化状态。代码如下：

```java
public void reset() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        breakBarrier();   // break the current generation
        nextGeneration(); // start a new generation
    } finally {
        lock.unlock();
    }
}
```

## getNumberWaiting

>&nbsp;&nbsp;&nbsp;&nbsp;#getNumberWaiting() 方法，获得等待的线程数。代码如下：

```java
public int getNumberWaiting() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return parties - count;
    } finally {
        lock.unlock();
    }
}
```

## isBroken

>&nbsp;&nbsp;&nbsp;&nbsp;#isBroken() 方法，判断 CyclicBarrier 是否处于中断。代码如下：

```java
public boolean isBroken() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return generation.broken;
    } finally {
        lock.unlock();
    }
}
```

# 应用场景

>&nbsp;&nbsp;&nbsp;&nbsp;CyclicBarrier 适用于多线程结果合并的操作，用于多线程计算数据，最后合并计算结果的应用场景。比如，我们需要统计多个 Excel 中的数据，然后等到一个总结果。我们可以通过多线程处理每一个 Excel ，执行完成后得到相应的结果，最后通过 barrierAction 来计算这些线程的计算结果，得到所有Excel的总和。