转载于:https://ifeve.com/file-channel/

# 概述

&nbsp;&nbsp;&nbsp;&nbsp;Java NIO中的FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。


# 示例

```java

  public static void test01 () throws Exception {
        RandomAccessFile aFile = new RandomAccessFile("文件路径", "rw");
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


# 其他方法


## position


&nbsp;&nbsp;&nbsp;&nbsp;有时可能需要在FileChannel的某个特定位置进行数据的读/写操作。可以通过调用position()方法获取FileChannel的当前位置。也可以通过调用position(long pos)方法设置FileChannel的当前位置。这里有两个例子:
```java
long pos = channel.position();
channel.position(pos +123);
```
&nbsp;&nbsp;&nbsp;&nbsp;如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 —— 文件结束标志。如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。

##  size

FileChannel实例的size()方法将返回该实例所关联文件的大小。如:


```java
long fileSize = channel.size();
```

## truncate

可以使用FileChannel.truncate()方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。如：

```java
channel.truncate(1024);
// 这个例子截取文件的前1024个字节。
```


## force方法

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点，需要调用force()方法。force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。下面的例子同时将文件数据和元数据强制写到磁盘上：

```java
channel.force(true);
```
