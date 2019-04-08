[TOC]
# 概述

>&nbsp;&nbsp;&nbsp;&nbsp;Unsafe类在jdk 源码的多个类中用到，这个类的提供了一些绕开JVM的更底层功能，基于它的实现可以提高效率。但是，它是一把双刃剑：正如它的名字所预示的那样，它是Unsafe的，它所分配的内存需要手动free（不被GC回收）。Unsafe类，提供了JNI某些功能的简单替代：确保高效性的同时，使事情变得更简单。

# UnSafe包含以下几种功能

```text
（1）Info相关。主要返回某些低级别的内存信息：addressSize(), pageSize()

（2）Objects相关。主要提供Object和它的域操纵方法：allocateInstance(),objectFieldOffset()

（3）Class相关。主要提供Class和它的静态域操纵方法：staticFieldOffset(),defineClass(),
defineAnonymousClass(),ensureClassInitialized()

（4）Arrays相关。数组操纵方法：arrayBaseOffset(),arrayIndexScale()

（5）Synchronization相关。主要提供低级别同步原语（如基于CPU的CAS（Compare-And-Swap）原语）：monitorEnter(),tryMonitorEnter(),
monitorExit(),compareAndSwapInt(),putOrderedInt()

（6）Memory相关。直接内存访问方法（绕过JVM堆直接操纵本地内存）：allocateMemory(),copyMemory(),freeMemory(),
getAddress(),getInt(),putInt()
```

# 如何初始化UnSafe

```java
   private static native void registerNatives();
    static {
        registerNatives();
        sun.reflect.Reflection.registerMethodsToFilter(Unsafe.class, "getUnsafe");
    }

    private Unsafe() {}

    private static final Unsafe theUnsafe = new Unsafe();

    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class<?> caller = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(caller.getClassLoader()))
            throw new SecurityException("Unsafe");
        return theUnsafe;
    }
```

>&nbsp;&nbsp;&nbsp;&nbsp;初始化的代码主要包括调用JVM本地方法registerNatives()和sun.reflect.Reflection#registerMethodsToFilter。然后新建一个Unsafe实例命名为theUnsafe，通过静态方法getUnsafe()获取，获取的时候需要做权限判断。由此可见，Unsafe使用了单例设计(可见构造私有化了)。Unsafe类做了限制，如果是普通的调用的话，它会抛出一个SecurityException异常；只有由主类加载器(BootStrap classLoader)加载的类才能调用这个类中的方法。最简单的使用方式是基于反射获取Unsafe实例。

>&nbsp;&nbsp;&nbsp;&nbsp;如果想自己创建UNSafe的实例,则只能通过反射机制来实现,代码如下所示:
```java
 public static void main(String[] args) throws Exception {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);
        System.out.println(unsafe.pageSize());
    }
```

# 类、对象和变量相关方法

>&nbsp;&nbsp;&nbsp;&nbsp;主要包括类的非常规实例化、基于偏移地址获取或者设置变量的值、基于偏移地址获取或者设置数组元素的值等。

## getObject

>&nbsp;&nbsp;&nbsp;&nbsp;public native Object getObject(Object o, long offset);
通过给定的Java变量获取引用值。这里实际上是获取一个Java对象o中，获取偏移地址为offset的属性的值，此方法可以突破修饰符的抑制，也就是无视private、protected和default修饰符。类似的方法有getInt、getDouble等等。

## putObject

>&nbsp;&nbsp;&nbsp;&nbsp;public native void putObject(Object o, long offset, Object x);
将引用值存储到给定的Java变量中。这里实际上是设置一个Java对象o中偏移地址为offset的属性的值为x，此方法可以突破修饰符的抑制，也就是无视private、protected和default修饰符。类似的方法有putInt、putDouble等等。

## getObjectVolatile

>&nbsp;&nbsp;&nbsp;&nbsp;public native Object getObjectVolatile(Object o, long offset);
此方法和上面的getObject功能类似，不过附加了'volatile'加载语义，也就是强制从主存中获取属性值。类似的方法有getIntVolatile、getDoubleVolatile等等。这个方法要求被使用的属性被volatile修饰，否则功能和getObject方法相同。

## putObjectVolatile

>&nbsp;&nbsp;&nbsp;&nbsp;public native void putObjectVolatile(Object o, long offset, Object x);
此方法和上面的putObject功能类似，不过附加了'volatile'加载语义，也就是设置值的时候强制(JMM会保证获得锁到释放锁之间所有对象的状态更新都会在锁被释放之后)更新到主存，从而保证这些变更对其他线程是可见的。类似的方法有putIntVolatile、putDoubleVolatile等等。这个方法要求被使用的属性被volatile修饰，否则功能和putObject方法相同。

## putOrderedObject

>&nbsp;&nbsp;&nbsp;&nbsp;public native void putOrderedObject(Object o, long offset, Object x);
设置o对象中offset偏移地址offset对应的Object型field的值为指定值x。这是一个有序或者有延迟的putObjectVolatile方法，并且不保证值的改变被其他线程立即看到。只有在field被volatile修饰并且期望被修改的时候使用才会生效。类似的方法有putOrderedInt和putOrderedLong。

## staticFieldOffset

>&nbsp;&nbsp;&nbsp;&nbsp;public native long staticFieldOffset(Field f);
返回给定的静态属性在它的类的存储分配中的位置(偏移地址)。不要在这个偏移量上执行任何类型的算术运算，它只是一个被传递给不安全的堆内存访问器的cookie。注意：这个方法仅仅针对静态属性，使用在非静态属性上会抛异常。下面源码中的方法注释估计有误，staticFieldOffset和objectFieldOffset的注释估计是对调了，为什么会出现这个问题无法考究。

## objectFieldOffset

>&nbsp;&nbsp;&nbsp;&nbsp;public native long objectFieldOffset(Field f);
返回给定的非静态属性在它的类的存储分配中的位置(偏移地址)。不要在这个偏移量上执行任何类型的算术运算，它只是一个被传递给不安全的堆内存访问器的cookie。注意：这个方法仅仅针对非静态属性，使用在静态属性上会抛异常。

## staticFieldBase

>&nbsp;&nbsp;&nbsp;&nbsp;public native Object staticFieldBase(Field f);
返回给定的静态属性的位置，配合staticFieldOffset方法使用。实际上，这个方法返回值就是静态属性所在的Class对象的一个内存快照。注释中说到，此方法返回的Object有可能为null，它只是一个'cookie'而不是真实的对象，不要直接使用的它的实例中的获取属性和设置属性的方法，它的作用只是方便调用上面提到的像getInt(Object,long)等等的任意方法。

## shouldBeInitialized

>&nbsp;&nbsp;&nbsp;&nbsp;public native boolean shouldBeInitialized(Class<?> c);
检测给定的类是否需要初始化。通常需要使用在获取一个类的静态属性的时候(因为一个类如果没初始化，它的静态属性也不会初始化)。 此方法当且仅当ensureClassInitialized方法不生效的时候才返回false。

## ensureClassInitialized

>&nbsp;&nbsp;&nbsp;&nbsp;public native void ensureClassInitialized(Class<?> c);
检测给定的类是否已经初始化。通常需要使用在获取一个类的静态属性的时候(因为一个类如果没初始化，它的静态属性也不会初始化)。

## arrayBaseOffset

>&nbsp;&nbsp;&nbsp;&nbsp;public native int arrayBaseOffset(Class<?> arrayClass);
返回数组类型的第一个元素的偏移地址(基础偏移地址)。如果arrayIndexScale方法返回的比例因子不为0，你可以通过结合基础偏移地址和比例因子访问数组的所有元素。Unsafe中已经初始化了很多类似的常量如ARRAY_BOOLEAN_BASE_OFFSET等。

## arrayIndexScale

>&nbsp;&nbsp;&nbsp;&nbsp;public native int arrayIndexScale(Class<?> arrayClass);
返回数组类型的比例因子(其实就是数据中元素偏移地址的增量，因为数组中的元素的地址是连续的)。此方法不适用于数组类型为"narrow"类型的数组，"narrow"类型的数组类型使用此方法会返回0(这里narrow应该是狭义的意思，但是具体指哪些类型暂时不明确，笔者查了很多资料也没找到结果)。Unsafe中已经初始化了很多类似的常量如ARRAY_BOOLEAN_INDEX_SCALE等。

## defineClass

>&nbsp;&nbsp;&nbsp;&nbsp;public native Class<?> defineClass(String name, byte[] b, int off, int len,ClassLoader loader,ProtectionDomain protectionDomain);
告诉JVM定义一个类，返回类实例，此方法会跳过JVM的所有安全检查。默认情况下，ClassLoader(类加载器)和ProtectionDomain(保护域)实例应该来源于调用者。

## defineAnonymousClass

>&nbsp;&nbsp;&nbsp;&nbsp;public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
这个方法的使用可以看R大的知乎回答：JVM crashes at libjvm.so，下面截取一点内容解释此方法。

>&nbsp;&nbsp;&nbsp;&nbsp;1、VM Anonymous Class可以看作一种模板机制，如果程序要动态生成很多结构相同、只是若干变量不同的类的话，可以先创建出一个包含占位符常量的正常类作为模板，然后利用sun.misc.Unsafe#defineAnonymousClass()方法，传入该类(host class，宿主类或者模板类)以及一个作为"constant pool path"的数组来替换指定的常量为任意值，结果得到的就是一个替换了常量的VM Anonymous Class。
>&nbsp;&nbsp;&nbsp;&nbsp;2、VM Anonymous Class从VM的角度看是真正的"没有名字"的，在构造出来之后只能通过Unsafe#defineAnonymousClass()返回出来一个Class实例来进行反射操作。
还有其他几点看以自行阅读。这个方法虽然翻译为"定义匿名类"，但是它所定义的类和实际的匿名类有点不相同，因此一般情况下我们不会用到此方法。在Jdk中lambda表达式相关的东西用到它，可以看InnerClassLambdaMetafactory这个类。

## allocateInstance

>&nbsp;&nbsp;&nbsp;&nbsp;public native Object allocateInstance(Class<?> cls) throws InstantiationException;
通过Class对象创建一个类的实例，不需要调用其构造函数、初始化代码、JVM安全检查等等。同时，它抑制修饰符检测，也就是即使构造器是private修饰的也能通过此方法实例化。

# 内存管理

## addressSize

>&nbsp;&nbsp;&nbsp;&nbsp;public native int addressSize();
获取本地指针的大小(单位是byte)，通常值为4或者8。常量ADDRESS_SIZE就是调用此方法。

## pageSize

>&nbsp;&nbsp;&nbsp;&nbsp;public native int pageSize();
获取本地内存的页数，此值为2的幂次方。

## allocateMemory

>&nbsp;&nbsp;&nbsp;&nbsp;public native long allocateMemory(long bytes);
分配一块新的本地内存，通过bytes指定内存块的大小(单位是byte)，返回新开辟的内存的地址。如果内存块的内容不被初始化，那么它们一般会变成内存垃圾。生成的本机指针永远不会为零，并将对所有值类型进行对齐。可以通过freeMemory方法释放内存块，或者通过reallocateMemory方法调整内存块大小。bytes值为负数或者过大会抛出IllegalArgumentException异常，如果系统拒绝分配内存会抛出OutOfMemoryError异常。

## reallocateMemory

>&nbsp;&nbsp;&nbsp;&nbsp;public native long reallocateMemory(long address, long bytes);
通过指定的内存地址address重新调整本地内存块的大小，调整后的内存块大小通过bytes指定(单位为byte)。可以通过freeMemory方法释放内存块，或者通过reallocateMemory方法调整内存块大小。bytes值为负数或者过大会抛出IllegalArgumentException异常，如果系统拒绝分配内存会抛出OutOfMemoryError异常。

## setMemory

>&nbsp;&nbsp;&nbsp;&nbsp;public native void setMemory(Object o, long offset, long bytes, byte value);
将给定内存块中的所有字节设置为固定值(通常是0)。内存块的地址由对象引用o和偏移地址共同决定，如果对象引用o为null，offset就是绝对地址。第三个参数就是内存块的大小，如果使用allocateMemory进行内存开辟的话，这里的值应该和allocateMemory的参数一致。value就是设置的固定值，一般为0(这里可以参考netty的DirectByteBuffer)。一般而言，o为null，所有有个重载方法是public native void setMemory(long offset, long bytes, byte value);，等效于setMemory(null, long offset, long bytes, byte value);。


# 多线程同步

>&nbsp;&nbsp;&nbsp;&nbsp;主要包括监视器锁定、解锁以及CAS相关的方法。

## monitorEnter

>&nbsp;&nbsp;&nbsp;&nbsp;public native void monitorEnter(Object o);
锁定对象，必须通过monitorExit方法才能解锁。此方法经过实验是可以重入的，也就是可以多次调用，然后通过多次调用monitorExit进行解锁。

## monitorExit

>&nbsp;&nbsp;&nbsp;&nbsp;public native void monitorExit(Object o);
解锁对象，前提是对象必须已经调用monitorEnter进行加锁，否则抛出IllegalMonitorStateException异常。

## tryMonitorEnter

>&nbsp;&nbsp;&nbsp;&nbsp;public native boolean tryMonitorEnter(Object o);
尝试锁定对象，如果加锁成功返回true，否则返回false。必须通过monitorExit方法才能解锁。

## compareAndSwapObject

>&nbsp;&nbsp;&nbsp;&nbsp;public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);
针对Object对象进行CAS操作。即是对应Java变量引用o，原子性地更新o中偏移地址为offset的属性的值为x，当且仅的偏移地址为offset的属性的当前值为expected才会更新成功返回true，否则返回false。

```text
o：目标Java变量引用。
offset：目标Java变量中的目标属性的偏移地址。
expected：目标Java变量中的目标属性的期望的当前值。
x：目标Java变量中的目标属性的目标更新值。
```
类似的方法有compareAndSwapInt和compareAndSwapLong，在Jdk8中基于CAS扩展出来的方法有getAndAddInt、getAndAddLong、getAndSetInt、getAndSetLong、getAndSetObject，它们的作用都是：通过CAS设置新的值，返回旧的值。

# 线程的挂起和恢复

## unpark

>&nbsp;&nbsp;&nbsp;&nbsp;public native void unpark(Object thread);
释放被park创建的在一个线程上的阻塞。这个方法也可以被使用来终止一个先前调用park导致的阻塞。这个操作是不安全的，因此必须保证线程是存活的(thread has not been destroyed)。从Java代码中判断一个线程是否存活的是显而易见的，但是从native代码中这机会是不可能自动完成的。

## park

>&nbsp;&nbsp;&nbsp;&nbsp;public native void park(boolean isAbsolute, long time);
阻塞当前线程直到一个unpark方法出现(被调用)、一个用于unpark方法已经出现过(在此park方法调用之前已经调用过)、线程被中断或者time时间到期(也就是阻塞超时)。在time非零的情况下，如果isAbsolute为true，time是相对于新纪元之后的毫秒，否则time表示纳秒。这个方法执行时也可能不合理地返回(没有具体原因)。并发包java.util.concurrent中的框架对线程的挂起操作被封装在LockSupport类中，LockSupport类中有各种版本pack方法，但最终都调用了Unsafe#park()方法。


# 内存屏障

>&nbsp;&nbsp;&nbsp;&nbsp;内存屏障相关的方法是在Jdk8添加的。内存屏障相关的知识可以先自行查阅。

## loadFence

>&nbsp;&nbsp;&nbsp;&nbsp;public native void loadFence();
在该方法之前的所有读操作，一定在load屏障之前执行完成。

## storeFence

>&nbsp;&nbsp;&nbsp;&nbsp;public native void storeFence();
在该方法之前的所有写操作，一定在store屏障之前执行完成

## fullFence

>&nbsp;&nbsp;&nbsp;&nbsp;public native void fullFence();
在该方法之前的所有读写操作，一定在full屏障之前执行完成，这个内存屏障相当于上面两个(load屏障和store屏障)的合体功能。



# 其他

## getLoadAverage

>&nbsp;&nbsp;&nbsp;&nbsp;public native int getLoadAverage(double[] loadavg, int nelems);
获取系统的平均负载值，loadavg这个double数组将会存放负载值的结果，nelems决定样本数量，nelems只能取值为1到3，分别代表最近1、5、15分钟内系统的平均负载。如果无法获取系统的负载，此方法返回-1，否则返回获取到的样本数量(loadavg中有效的元素个数)。实验中这个方法一直返回-1，其实完全可以使用JMX中的相关方法替代此方法。

## throwException

>&nbsp;&nbsp;&nbsp;&nbsp;public native void throwException(Throwable ee);
绕过检测机制直接抛出异常。