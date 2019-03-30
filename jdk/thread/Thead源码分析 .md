# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;多线程的问题相信大家都遇到过,但是大家关注过线程在java中是如何实现的吗？今天我们来讨论一下Thread是如何实现的。先来看一下Thread最简单的一个小例子。

```java
    class TestThreadOne extends Thread {
        @Override
        public void run() {
            System.out.println("one"+Thread.currentThread().getName());
        }
    }


    TestThreadOne threadOne = new TestThreadOne();
        threadOne.start();
```

>&nbsp;&nbsp;&nbsp;&nbsp;这样就开启了一个最简答的线程。下面我会根据线程状态流转图来说明一下
![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-29/1.jpg?raw=true)


# 新建状态

```java
    //其实就是new一个新的线程
    TestThreadOne threadOne = new TestThreadOne();

    class TestThreadOne extends Thread {
    @Override
    public void run() {
        System.out.println("one"+Thread.currentThread().getName());
    }
}
```

# 就绪状态

```java
    threadOne.start();

    //这个方法是thread中的start方法
    public synchronized void start() {
        //threadStatus这个值初始化的时候就是0,
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
        //加入到线程组中    
        group.add(this);
        //标记是否开始
        boolean started = false;
        try {
            //start0是一个native方法,调用这个方法就表示开启了线程，线程进入到了就绪状态
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();

    //在JDK中的注解就是表示线程的状态,初始化的时候表示这个线程还没有开始
    private volatile int threadStatus = 0;

    //group就是指定一个线程组,如果使用没有参数的构造函数的时候,就会去拿到
    //SecurityManager的group,具体的逻辑如下所示
    private ThreadGroup group;

     if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }


```

>&nbsp;&nbsp;&nbsp;&nbsp;这这里我们很有必要提一下这个init方法。在Thread的构造函数中都是调用了init这个方法。下面就让我们一起来看一下这个方法中。

```java
/**
 *ThreadGroup 线程组
 *Runnable 重写run方法,指定线程调度任务
 *name 线程的名字
 *stackSize 新线程需要的堆栈大小,如果为0表示该参数会被忽略
 *后面两个参数我也不知道是干啥的......
 *关于AccessControlContext如果有谁知道的话可以进行补充
 */
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;
        //获取当前线程
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        //设置是否是后台线程,如果设置为后台线程,则这个线程只有在程序退出的时候才会失去作用
        this.daemon = parent.isDaemon();
        //设置线程的优先级
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }

```

# 运行状态

```java
    //在这里就会就调用你重写的run方法。
    
    /* What will be run. */
    private Runnable target;

     @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }

```

# wait()和notify()

>JDK提供了wait方法和notify方法控制线程的状态。这两个方法是在Object类中。如下图所示:

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-29/2.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;那么这个时候就会有一个问题,为什么关于线程的方法会定义在Object类中?其实是因为无论是wait方法还是notify方法其实都需要首先获取一个目标对象的监视器。而这个目标对象就是synchronized关键字中的对象。所以把wait()、notify()、notifyAll()方法放在了object类中。

>&nbsp;&nbsp;&nbsp;&nbsp;如果之前对下线程了解的同学也可能知道在Thread类中有一个sleep方法,那么这个sleep方法和wait方法有什么区别呢？其实就是锁的对相关的释放问题,Thread.sleep方法虽然当前线程了停止了,但是当前线程获取的锁并没有失去,而是当前线程还拥有着这个锁,那么其他的对象肯定是获取不到锁,所以这个时候其他的线程也一直处于等待的状态。wait方法会直接释放掉这个锁。其他线程就可以竞争这把锁,可以继续往下执行。notify方法只是会唤醒阻塞队列线程中的某一个线程,而且唤醒之后也不会立即拿到锁,只是会和其他线程一直去竞争锁。如果想要唤醒所有的线程,则使用notifyAll方法。


>&nbsp;&nbsp;&nbsp;&nbsp;下面给出一个wait和notify方法使用的例子。

```java
public class WaitAndNotify {

    public static  final  Object lock = new Object();

    public static void main(String[] args) {

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + " get lock");
                    try {
                        // 这里调用了wait方法，线程会释放掉这个锁,这个也是和sleep方法最大的区别，sleep
                        // 方法病不会释放掉锁，在sleep期间还是占据着锁，调用了wait之后会进入到等待队列中去
                        // 直到有其他线程通过lock.notify();的方法去唤醒,这个notify并不是按照队列中的顺序去
                        // 唤醒的，而是唤醒任意一个，如果想唤醒全部，则通过lock.notifyAll();的方法去唤醒
                        /**
                         * 注意的一点是lock.wait();和lock.notify();和lock.notifyAll();这三个方法必须在
                         * synchronized语法中，也算是个局限吧
                         */
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " get lock again");
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + " get lock 2");
                    lock.notify();
                }
            }
        }).start();
    }
}

```

# suspend(挂起)和resume(继续执行)

```java
  @Deprecated
    public final void suspend() {
        checkAccess();
        suspend0();
    }
    @Deprecated
    public final void resume() {
        checkAccess();
        resume0();
    }

```

>&nbsp;&nbsp;&nbsp;&nbsp;我们可以看到在源码中这两个方法已经被废弃了。


>&nbsp;&nbsp;&nbsp;&nbsp;这两个方法被废弃的原因是suspend方法在导致线程暂停的同时,也是并不会放弃锁。因此其他任何线程要访问被他占用的锁的时候,都会被牵连,导致无法正常进行。直到对应的线程操作了resume方法,被挂起的线程才能继续.但是如果resume方法在suspend方法之前执行,那么这个锁将永远不会得到释放。导致整个系统都不会正常工作 。

```java
public class SuspendAndResume {

    private static final Object OBJECT = new Object();

    public static class ChangeObjectThread extends Thread {
        public ChangeObjectThread(String name) {
            super.setName(name);
        }

        @Override
        public void run() {
            synchronized (OBJECT) {
                System.out.println("in" + getName());
                /**
                 * suspend()并不会释放锁，会阻塞其他的线程
                 */
                Thread.currentThread().suspend();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ChangeObjectThread t1 = new ChangeObjectThread("线程1");
        ChangeObjectThread t2 = new ChangeObjectThread("线程2");

        t1.start();
        Thread.sleep(100);
        t2.start();
        t1.resume();
        t2.resume();
        t1.join();
        t2.join();
    }
}

```

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-03-29/3.jpg?raw=true)

>&nbsp;&nbsp;&nbsp;&nbsp;上面这个小例子展示了线程2虽然是被挂起的,但是根据arthas显示却是Runnable中,是因为t2.resume并没有生效。


# join(等待线程结束)和yeild(谦让)


>&nbsp;&nbsp;&nbsp;&nbsp;join方法会一直阻塞当前线程,直到目标线程执行完毕。
```java
public class JoinAndYeild {

    /**
     * volatile只能保证是可见性和有序性，不能保证原子性
     */
    public volatile static int i = 0;

    public static class AddThread extends Thread {
        @Override
        public void run() {
            while (i <1000) {
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AddThread addThread = new AddThread();
        addThread.start();
        /**
         * 现在有两个线程，如果执行如下的方法，main线程会无限等待，直到
         * 等到addThread这个线程执行完毕，Thread.yield方法会
         * 让当前线程让出CPU，之后线程会通过资源竞争的方法获取线程。
         */
        addThread.join();

        System.out.println(i);
    }
}

```

>&nbsp;&nbsp;&nbsp;&nbsp;yield方法会让出CPU,但是需要注意的是让出CPU并不是表示当前线程不执行了。当前线程让出CPU之后,还会对CPU资源进行争夺,但是是否能够被分配就不一定了。

# 线程的其他问题

## 线程组

```java
public class ThreaadGroupName implements Runnable {

    public static void main(String[] args) {

        ThreadGroup threadGroup = new ThreadGroup("threadgroup");
        Thread thread1= new Thread(threadGroup,new ThreaadGroupName(),"t1");
        Thread thread2 = new Thread(threadGroup,new ThreaadGroupName(),"t2");
        thread1.start();
        thread2.start();
        System.out.println(threadGroup.activeCount());
        threadGroup.list();
    }
    @Override
    public void run() {
        String s = Thread.currentThread().getThreadGroup().getName() + "--" + Thread.currentThread().getName();
        while (true) {
            System.out.println("I am" + s);
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

## 守护线程
>&nbsp;&nbsp;&nbsp;&nbsp;需要注意的一点就是声明一个守护线程必须在start方法之前。

## 线程优先级

>&nbsp;&nbsp;&nbsp;&nbsp;线程是有优先级的,具体如下所示

```text
1.记住当线程的优先级没有指定时，所有线程都携带普通优先级。
2.优先级可以用从1到10的范围指定。10表示最高优先级，1表示最低优先级，5是普通优先级。
3.记住优先级最高的线程在执行时被给予优先。但是不能保证线程在启动时就进入运行状态。
4.与在线程池中等待运行机会的线程相比，当前正在运行的线程可能总是拥有更高的优先级。
5.由调度程序决定哪一个线程被执行。
6.t.setPriority()用来设定线程的优先级。
7.记住在线程开始方法被调用之前，线程的优先级应该被设定。
8.你可以使用常量，如MIN_PRIORITY,MAX_PRIORITY，NORM_PRIORITY来设定优先级。
```

## 正确处理线程异常的姿势

### 在线程池中直接传入一个接口

```java
 public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        executorService.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println(3/2);
                    System.out.println(3/0);
                    System.out.println(3/1);
                }catch (Exception e) {
                    System.out.println(e.getMessage());
                }
            }
        });

        executorService.shutdown();
    }
```


>&nbsp;&nbsp;&nbsp;&nbsp;如果传入了一个Runnable的实现,则需要指定一个Thread.UncaughtExceptionHandler的实现类。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class NoCaughtThread
{
    public static void main(String[] args)
    {
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        executorService.execute(new Task());
    }
}

class Task implements Runnable
{
    @Override
    public void run()
    {
        Thread.currentThread().setUncaughtExceptionHandler(new MyException());
        System.out.println(3/2);
        System.out.println(3/0);
        System.out.println(3/1);
    }
}

class MyException implements Thread.UncaughtExceptionHandler {

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println(t.getName()+"-"+e.getMessage());
    }
}
```

>&nbsp;&nbsp;&nbsp;&nbsp;之后想到对的关于Thread的问题都会在这里进行补充。