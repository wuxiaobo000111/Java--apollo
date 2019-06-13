
# 示例代码

```java
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

```