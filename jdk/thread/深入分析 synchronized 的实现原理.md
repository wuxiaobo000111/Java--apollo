# 实现原理

>&nbsp;&nbsp;&nbsp;&nbsp;synchronized 可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性。下面通过一个例子来实现以下保证共享变量的可见性。

```java

package com.bobo;


public class Test {


    public static void main(String[] args) {
        Game game = new Game();
        game.start();
        try {
            Thread.sleep(1000);
        } catch (Exception e) {

        }
        game.setIndex(-1);
        System.out.println("change......");
    }
}


class Game extends Thread {

    public int index = 0;
    public synchronized int getIndex() {
        return index;
    }

    public synchronized void setIndex(int index) {
        this.index = index;
    }

    @Override
    public void run() {
        System.out.println("Game Start");
        while (true) {
            if ( this.getIndex()== -1) {
                break;
            }
        }
        System.out.println("Game End");
    }
}


```

>&nbsp;&nbsp;&nbsp;&nbsp;Java 中每一个对象都可以作为锁，这是 synchronized 实现同步的基础：

```text
    1.普通同步方法，锁是当前实例对象
    2.静态同步方法，锁是当前类的 class 对象
    3.同步方法块，锁是括号里面的对象
```

>&nbsp;&nbsp;&nbsp;&nbsp;同步代码块是使用 monitorenter 和 monitorexit 指令实现的；2）同步方法（在这看不出来需要看JVM底层实现）依靠的是方法修饰符上的ACC_SYNCHRONIZED 实现。

>&nbsp;&nbsp;&nbsp;&nbsp;对于同步代码块:monitorenter 指令插入到同步代码块的开始位置，monitorexit 指令插入到同步代码块的结束位置，JVM 需要保证每一个 monitorenter 都有一个 monitorexit 与之相对应。任何对象都有一个 Monitor 与之相关联，当且一个 Monitor 被持有之后，他将处于锁定状态。线程执行到 monitorenter 指令时，将会尝试获取对象所对应的 Monitor 所有权，即尝试获取对象的锁。

>&nbsp;synchronized的实现依赖于Java 对象头和 Monitor 。所以先来介绍一下这两个东西。


## Java对象头


>&nbsp;&nbsp;&nbsp;&nbsp;Hotspot 虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。其中：Klass Point 是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

>&nbsp;&nbsp;&nbsp;&nbsp;Mark Word 用于存储对象自身的运行时数据，如哈希码（HashCode）、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等等。Java 对象头一般占有两个机器码（在 32 位虚拟机中，1 个机器码等于 4 字节，也就是 32 bits）。但是如果对象是数组类型，则需要三个机器码，因为 JVM 虚拟机可以通过 Java 对象的元数据信息确定 Java 对象的大小，无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。32位机器中的Mark Word结构如下所示:

![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/17.png?raw=true)


>&nbsp;&nbsp;&nbsp;&nbsp;这里涉及到了锁的升级的问题。如果想要了解这个问题,可以访问[synchronized锁升级](https://github.com/wuxiaobo000111/markdown/blob/master/jdk/thread/java%E9%94%81%E7%9A%84%E5%8D%87%E7%BA%A7.md
"synchronized锁升级")这个地址。介绍了一下基本原理。



## 

>&nbsp;&nbsp;&nbsp;Monitor Record 是线程私有的数据结构，每一个线程都有一个可用 Monitor Record 列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个 Monitor Record 关联（对象头的 MarkWord 中的 LockWord 指向 Monitor 的起始地址），Monitor Record 中有一个 Owner 字段，存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：


# 锁优化


>&nbsp;&nbsp;&nbsp;&nbsp;简单来说，在 JVM 中 monitorenter 和 monitorexit 字节码依赖于底层的操作系统的Mutex Lock 来实现的，但是由于使用 Mutex Lock 需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的。然而，在现实中的大部分情况下，同步方法是运行在单线程环境（无锁竞争环境），如果每次都调用 Mutex Lock 那么将严重的影响程序的性能。因此，JDK 1.6 对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

## 自旋锁

>&nbsp;&nbsp;&nbsp;&nbsp;所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。
&nbsp;&nbsp;&nbsp;&nbsp;怎么等待呢？执行一段无意义的循环即可（自旋）。
&nbsp;&nbsp;&nbsp;&nbsp;自旋等待不能替代阻塞，先不说对处理器数量的要求（多核，貌似现在没有单核的处理器了），虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，典型的占着茅坑不拉屎，这样反而会带来性能上的浪费。
&nbsp;&nbsp;&nbsp;&nbsp;所以说，自旋等待的时间（自旋的次数）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。
&nbsp;&nbsp;&nbsp;&nbsp;自旋锁在 JDK 1.4.2 中引入，默认关闭，但是可以使用 -XX:+UseSpinning 开开启。
在 JDK1.6 中默认开启。同时自旋的默认次数为 10 次，可以通过参数 -XX:PreBlockSpin 来调整。
&nbsp;&nbsp;&nbsp;&nbsp;如果通过参数 -XX:PreBlockSpin 来调整自旋锁的自旋次数，会带来诸多不便。假如我将参数调整为 10 ，但是系统很多线程都是等你刚刚退出的时候，就释放了锁（假如你多自旋一两次就可以获取锁），你是不是很尴尬。于是 JDK 1.6 引入自适应的自旋锁，让虚拟机会变得越来越聪明。

## 适应自旋锁

>&nbsp;&nbsp;&nbsp;&nbsp;JDK 1.6 引入了更加聪明的自旋锁，即自适应自旋锁。
&nbsp;&nbsp;&nbsp;&nbsp;所谓自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。它怎么做呢？
&nbsp;&nbsp;&nbsp;&nbsp;线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。
&nbsp;&nbsp;&nbsp;&nbsp;反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。
&nbsp;&nbsp;&nbsp;&nbsp;有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。

## 锁消除

>&nbsp;&nbsp;&nbsp;&nbsp;为了保证数据的完整性，我们在进行操作时需要对这部分操作进行同步控制。但是，在有些情况下，JVM检测到不可能存在共享数据竞争，这是JVM会对这些同步锁进行锁消除。如果不存在竞争，为什么还需要加锁呢？所以锁消除可以节省毫无意义的请求锁的时间。
&nbsp;&nbsp;&nbsp;&nbsp;锁消除的依据是逃逸分析的数据支持。变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是对于我们程序员来说这还不清楚么？我们会在明明知道不存在数据竞争的代码块前加上同步吗？但是有时候程序并不是我们所想的那样？我们虽然没有显示使用锁，但是我们在使用一些 JDK 的内置 API 时，如 StringBuffer、Vector、HashTable 等，这个时候会存在隐性的加锁操作。比如 StringBuffer 的 #append(..)方法，Vector 的 add(...) 方法。

## 锁粗化

>&nbsp;&nbsp;&nbsp;&nbsp;我们知道在使用同步锁的时候，需要让同步块的作用范围尽可能小：仅在共享数据的实际作用域中才进行同步。这样做的目的，是为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。
&nbsp;&nbsp;&nbsp;&nbsp;在大多数的情况下，上述观点是正确的。但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入锁粗化的概念。
&nbsp;&nbsp;&nbsp;&nbsp;锁粗话概念比较好理解，就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。
&nbsp;&nbsp;&nbsp;&nbsp;如上面实例：vector 每次 add 的时候都需要加锁操作，JVM 检测到对同一个对象（vector）连续加锁、解锁操作，会合并一个更大范围的加锁、解锁操作，即加锁解锁操作会移到 for 循环之外。