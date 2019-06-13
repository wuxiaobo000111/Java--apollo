转载于:https://ifeve.com/channels/

# 概述

NIO和IO的区别

```text
1. 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。

2. 通道可以异步地读写。

3. 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。
```


![在这里插入图片描述](https://github.com/wuxiaobo000111/pictures/blob/master/2019-05-10/79.png?raw=true)


# Channel类型


```text
1. FileChannel : 从文件中读写数据。

2. DatagramChannel : 能通过UDP读写网络中的数据。

3. SocketChannel : 能通过TCP读写网络中的数据。

4. ServerSocketChannel : 可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。
```

# 给出一个示例


```java


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
