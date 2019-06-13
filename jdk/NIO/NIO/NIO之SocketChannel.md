# 概述
&nbsp;&nbsp;&nbsp;&nbsp;Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel。


# 代码示例

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class NIOClient {
    /**
     * 通道
     */
    SocketChannel channel;

    public void initClient(String host, int port) throws IOException
    {
        //构造socket连接
        InetSocketAddress servAddr = new InetSocketAddress(host, port);

        //打开连接
        this.channel = SocketChannel.open(servAddr);
    }

    public void sendAndRecv(String words) throws IOException
    {
        byte[] msg = new String(words).getBytes();
        ByteBuffer buffer = ByteBuffer.wrap(msg);
        System.out.println("Client sending: " + words);
        channel.write(buffer);
        buffer.clear();
        channel.read(buffer);
        System.out.println("Client received: " + new String(buffer.array()).trim());

        channel.close();
    }

    public static void main(String[] args) throws IOException
    {
        NIOClient client = new NIOClient();
        client.initClient("localhost", 10000);
        client.sendAndRecv("I am a client");
    }
}

```