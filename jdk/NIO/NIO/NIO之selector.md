转载于: https://ifeve.com/selectors/

#  概述

&nbsp;&nbsp;&nbsp;&nbsp;Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。


# 为什么使用selector 

&nbsp;&nbsp;&nbsp;&nbsp;仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。
&nbsp;&nbsp;&nbsp;&nbsp;但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。


# selector的创建

```java
Selector selector = Selector.open();
```

# 向Selector注册通道

```java
  SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress(8000));
  socketChannel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
```

&nbsp;&nbsp;&nbsp;&nbsp;与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。

&nbsp;&nbsp;&nbsp;&nbsp;注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

```java
    public static final int OP_READ = 1 << 0;


    public static final int OP_WRITE = 1 << 2;

   
    public static final int OP_CONNECT = 1 << 3;

    
    public static final int OP_ACCEPT = 1 << 4;

```

&nbsp;&nbsp;&nbsp;&nbsp;通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为“连接就绪”。一个server socket channel准备好接收新进入的连接称为“接收就绪”。一个有数据可读的通道可以说是“读就绪”。等待写数据的通道可以说是“写就绪”。

如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：

```java
 socketChannel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
```


# selectorKey

&nbsp;&nbsp;&nbsp;&nbsp;当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性：

```text

interest集合
ready集合
Channel
Selector
附加的对象（可选）
```

## interest集合

```java
   public static void test02 () throws Exception {
        Selector selector = Selector.open();
        ServerSocketChannel socketChannel = ServerSocketChannel.open();
        socketChannel.configureBlocking(false);
        socketChannel.socket().bind(new InetSocketAddress(8080));
        SelectionKey register = socketChannel.register(selector, SelectionKey.OP_ACCEPT);
        int interestOps = register.interestOps();
        System.out.println(interestOps&SelectionKey.OP_ACCEPT);

    }
```


## ready集合

ready 集合是通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。Selection将在下一小节进行解释。可以这样访问ready集合：


```java
int readySet = selectionKey.readyOps();
```
可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```


## Channel + Selector

```java
 SelectableChannel channel = register.channel();
 Selector selector1 = register.selector();
```

## 附加对象

&nbsp;&nbsp;&nbsp;&nbsp;可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加 与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法如下：

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

还可以在用register()方法向Selector注册Channel的时候附加对象。如：

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

# 通过selector选择通道

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。换句话说，如果你对“读就绪”的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。下面是select()方法：

```java
int select():
select()阻塞到至少有一个通道在你注册的事件上就绪了。

int select(long timeout):select(long timeout)和select()一样，除了最长会阻塞timeout
毫秒(参数)。

int selectNow():selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选
择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。
```


&nbsp;&nbsp;&nbsp;&nbsp;select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。


# selectedKeys()

&nbsp;&nbsp;&nbsp;&nbsp;一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```java
Set selectedKeys = selector.selectedKeys();
```

&nbsp;&nbsp;&nbsp;&nbsp;当像Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。可以通过SelectionKey的selectedKeySet()方法访问这些对象。

可以遍历这个已选择的键集合来访问就绪的通道。如下：

```java
Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.
    } else if (key.isConnectable()) {
        // a connection was established with a remote server.
    } else if (key.isReadable()) {
        // a channel is ready for reading
    } else if (key.isWritable()) {
        // a channel is ready for writing
    }
    keyIterator.remove();
}
```

&nbsp;&nbsp;&nbsp;&nbsp;这个循环遍历已选择键集中的每个键，并检测各个键所对应的通道的就绪事件。注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。

# wakeUp()

&nbsp;&nbsp;&nbsp;&nbsp;某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。只要让其它线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。如果有其它线程调用了wakeup()方法，但当前没有线程阻塞在select()方法上，下个调用select()方法的线程会立即“醒来（wake up）”。

# 完整示例

```java
package com.bobo.basic.io.nio.demo;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class NIOServer {

    /**
     * 选择器
     */
    private Selector selector;

    /**
     * 通道
     */
    ServerSocketChannel serverSocketChannel;

    public void initServer(int port) throws IOException
    {
        //打开一个通道
        serverSocketChannel = ServerSocketChannel.open();

        //通道设置非阻塞
        serverSocketChannel.configureBlocking(false);

        //绑定端口号
        serverSocketChannel.socket().bind(new InetSocketAddress("localhost", port));

        //注册
        this.selector = Selector.open();
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
    }

    public void listen() throws IOException
    {
        System.out.println("server started succeed!");

        while (true)
        {
            selector.select();
            Iterator<SelectionKey> ite = selector.selectedKeys().iterator();
            while (ite.hasNext())
            {
                SelectionKey key = ite.next();
                if (key.isAcceptable())
                {
                    SocketChannel channel = serverSocketChannel.accept();
                    channel.configureBlocking(false);
                    channel.register(selector, SelectionKey.OP_READ);
                }
                else if (key.isReadable())
                {
                    recvAndReply(key);
                }
                ite.remove();
            }
        }
    }

    public void recvAndReply(SelectionKey key) throws IOException
    {
        SocketChannel channel = (SocketChannel) key.channel();
        ByteBuffer buffer = ByteBuffer.allocate(256);
        int i = channel.read(buffer);
        if (i != -1)
        {
            String msg = new String(buffer.array()).trim();
            System.out.println("NIO server received message =  " + msg);
            System.out.println("NIO server reply =  " + msg);
            channel.write(ByteBuffer.wrap( msg.getBytes()));
        }
        else
        {
            channel.close();
        }
    }

    public static void main(String[] args) throws IOException
    {
        NIOServer server = new NIOServer();
        server.initServer(10000);
        server.listen();
    }

}

````