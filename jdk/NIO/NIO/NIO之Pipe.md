转载于:https://ifeve.com/pipe/

# 概述

Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/87.png?raw=true)


# 创建管道

```java
Pipe pipe = Pipe.open();

```

# 向管道写数据

```java
Pipe.SinkChannel sinkChannel = pipe.sink();
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}


```

# 从管道读数据

```java
Pipe.SourceChannel sourceChannel = pipe.source();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = sourceChannel.read(buf);
```