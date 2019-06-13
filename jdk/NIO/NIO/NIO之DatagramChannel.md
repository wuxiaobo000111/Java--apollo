转载于:https://www.cnblogs.com/mjorcen/p/4199289.html


# 概述

&nbsp;&nbsp;&nbsp;&nbsp;最后一个socket通道是DatagramChannel。正如SocketChannel对应Socket，ServerSocketChannel对应ServerSocket，每一个DatagramChannel对象也有一个关联的DatagramSocket对象。不过原命名模式在此并未适用：“DatagramSocketChannel”显得有点笨拙，因此采用了简洁的“DatagramChannel”名称。正如SocketChannel模拟连接导向的流协议（如TCP/IP），DatagramChannel则模拟包导向的无连接协议（如UDP/IP）：

&nbsp;&nbsp;&nbsp;&nbsp;创建DatagramChannel的模式和创建其他socket通道是一样的：调用静态的open( )方法来创建一个新实例。新DatagramChannel会有一个可以通过调用socket( )方法获取的对等DatagramSocket对象。DatagramChannel对象既可以充当服务器（监听者）也可以充当客户端（发送者）。如果您希望新创建的通道负责监听，那么通道必须首先被绑定到一个端口或地址/端口组合上。绑定DatagramChannel同绑定一个常规的DatagramSocket没什么区别，都是委托对等socket对象上的API实现的：

 # 示例代码

 ```java
package com.bobo.basic.io.nio.datagramChannel;


import java.net.DatagramSocket;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.channels.DatagramChannel;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.charset.Charset;
import java.util.Iterator;

public class DatagramChannelTestMain {
    private static final int port = 30008;
    private static final int TIMEOUT = 3000;

    /**
     * @param args
     */
    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        Thread serverThread = new ServerThread();
        serverThread.start();
        Thread clientThread = new ClientThread();
        clientThread.start();
        Thread.sleep(2000);
    }

    static class ServerThread extends Thread {

        @Override
        public void run() {
            DatagramChannel channel = null;
            try {
                channel = DatagramChannel.open();
                channel.configureBlocking(false);
                DatagramSocket socket = channel.socket();
                socket.bind(new InetSocketAddress(port));
                Selector selector = Selector.open();
                channel.register(selector, SelectionKey.OP_READ);
                while(selector.isOpen()){
                    System.out.println("S....!!!");
                    if(selector.select() == 0){//select must be
                        System.out.println("s....");
                        Thread.sleep(2000);
                        continue;
                    }
                    Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                    while(it.hasNext()){
                        SelectionKey key = it.next();
                        if(key.isValid() && key.isReadable()){
                            read(key);
                        }
                    }
                }
            }catch(Exception e){
                e.printStackTrace();
            }finally{
                if(channel != null){
                    try{
                        channel.close();
                    }catch (Exception ex) {
                        ex.printStackTrace();
                    }
                }
            }
        }

        private void read(SelectionKey key) throws Exception {
            DatagramChannel channel = (DatagramChannel)key.channel();
            System.out.println("channel" + channel.toString());
            ByteBuffer byteBuffer = ByteBuffer.allocate(256);
            InetSocketAddress address = (InetSocketAddress)channel.receive(byteBuffer);
            if(address == null){
                System.out.println(".....");
            }else{
                System.out.println(address.getAddress() + "//" + address.getPort());
            }
            byteBuffer.flip();
            Charset charset = Charset.defaultCharset();
            CharBuffer charBuffer = charset.decode(byteBuffer);
            System.out.println("Server read:" + charBuffer.toString());
            //if write,please here.
            byteBuffer = charset.encode("this is server sent!");
            channel.send(byteBuffer, address);
            key.interestOps(SelectionKey.OP_READ);
        }

    }

    static class ClientThread extends Thread {
        @Override
        public void run() {
            DatagramChannel channel = null;
            try {
                channel = DatagramChannel.open();
                channel.configureBlocking(false);
                channel.connect(new InetSocketAddress("10.12.124.19",port));
                while(!channel.isConnected()){
                    System.out.println("....");
                }
                Selector selector = Selector.open();
                channel.register(selector, SelectionKey.OP_WRITE);
                while(selector.isOpen()){
                    if(selector.select() == 0){
                        Thread.sleep(2000);
                        continue;
                    }
                    Iterator<SelectionKey> it = selector.selectedKeys().iterator();
                    while(it.hasNext()){
                        SelectionKey key = it.next();
                        if(key.isValid() && key.isWritable()){
                            write(key);
                        }else if(key.isReadable()){
                            read(key);
                        }
                    }
                }
            }catch (Exception e) {
                e.printStackTrace();
            }finally{
                if(channel != null){
                    try{
                        channel.close();
                    }catch(Exception ex){
                        ex.printStackTrace();
                    }
                }
            }
        }

        private void write(SelectionKey key) throws Exception{
            DatagramChannel channel = (DatagramChannel)key.channel();
            Charset charset = Charset.defaultCharset();
            ByteBuffer byteBuffer = charset.encode("this is client sent!");
            System.out.println("p:" + byteBuffer.position() + "//l:" + byteBuffer.limit());
            while(byteBuffer.hasRemaining()){
                channel.write(byteBuffer);
            }
            System.out.println("cw ok!");
            key.interestOps(SelectionKey.OP_READ);
        }

        private void read(SelectionKey key) throws Exception{
            DatagramChannel channel = (DatagramChannel)key.channel();
            Charset charset = Charset.defaultCharset();
//			ByteBuffer byteBuffer = charset.encode("this is client sent!");
//			System.out.println("p:" + byteBuffer.position() + "//l:" + byteBuffer.limit());
            ByteBuffer byteBuffer = ByteBuffer.allocate(256);
            while(channel.read(byteBuffer) > 0){
                //
            }
            byteBuffer.flip();
            CharBuffer charBuffer = charset.decode(byteBuffer);
            System.out.println("Client read:" + charBuffer.toString());
            key.interestOps(SelectionKey.OP_WRITE);
        }
    }
}


 ```