
转载于:https://ifeve.com/buffers/


# Buffer概述


&nbsp;&nbsp;&nbsp;&nbsp;Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

# Buffer的基本用法

&nbsp;&nbsp;&nbsp;&nbsp;一般包含四个步骤:

```text
1. 写入数据到Buffer

2. 调用flip()方法

3. 从Buffer中读取数据

4. 调用clear()方法或者compact()方法
```

&nbsp;&nbsp;&nbsp;&nbsp;当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。
&nbsp;&nbsp;&nbsp;&nbsp;一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

```text
 public static void main(String[] args) throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("文件名", "rw");
        FileChannel inChannel = aFile.getChannel();
        ByteBuffer buf = ByteBuffer.allocate(1024);
        int bytesRead = inChannel.read(buf);
        while (bytesRead != -1) {
            System.out.println("Read " + bytesRead);
            buf.flip();
            while(buf.hasRemaining()){
                System.out.print((char) buf.get());
            }
            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        aFile.close();
    }
```

# Buffer的capacity,position和limit


&nbsp;&nbsp;&nbsp;&nbsp;缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。为了理解Buffer的工作原理，需要熟悉它的三个属性：capacity,position,limit。


## capacity

&nbsp;&nbsp;&nbsp;&nbsp;position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

## position

&nbsp;&nbsp;&nbsp;&nbsp;当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

&nbsp;&nbsp;&nbsp;&nbsp;当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

## limit

&nbsp;&nbsp;&nbsp;&nbsp;在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

&nbsp;&nbsp;&nbsp;&nbsp;当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）


# Buffer的类型

```java
ByteBuffer
MappedByteBuffer
CharBuffer
DoubleBuffer
FloatBuffer
IntBuffer
LongBuffer
ShortBuffer
```

&nbsp;&nbsp;&nbsp;&nbsp;如你所见，这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float 或 double类型来操作缓冲区中的字节。


# Buffer的分配

&nbsp;&nbsp;&nbsp;&nbsp;要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配1024字节capacity的ByteBuffer的例子。

```java
ByteBuffer buf = ByteBuffer.allocate(1024);
```


# 向Buffer中写入数据的两种方式

1. 从Channel写到Buffer

```java
 ByteBuffer buf = ByteBuffer.allocate(1024);
int bytesRead = inChannel.read(buf);
````

2. 通过put方法

```java
buf.put(127);
```

# 向Buffer中读数据的两种方式

1. 从Buffer读取数据到Channel。这里给出一个例子

```java
  public static void main(String[] args) throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("文件名", "rw");
        FileChannel inChannel = aFile.getChannel();
        ByteBuffer buf = ByteBuffer.allocate(1024);
        FileOutputStream fileOutputStream = new FileOutputStream(new File("D:\\wuxiaobo.log"));
        FileChannel fileOutputStreamChannel = fileOutputStream.getChannel();
        int bytesRead = inChannel.read(buf);
        while (bytesRead != -1) {
            buf.flip();
            while(buf.hasRemaining()){
               fileOutputStreamChannel.write(buf);
            }
            buf.clear();
            bytesRead = inChannel.read(buf);
        }
        aFile.close();
    }
```

2. 通过get方式读数据

```java
byte aByte = buf.get();
```


# 其他方法


## rewind()方法

&nbsp;&nbsp;&nbsp;&nbsp;Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素

## clear()与compact()方法

&nbsp;&nbsp;&nbsp;&nbsp;一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。
&nbsp;&nbsp;&nbsp;&nbsp;如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。
&nbsp;&nbsp;&nbsp;&nbsp;如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
&nbsp;&nbsp;&nbsp;&nbsp;如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。
&nbsp;&nbsp;&nbsp;&nbsp;compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

## mark()与reset()方法

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如:


```java
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.
```