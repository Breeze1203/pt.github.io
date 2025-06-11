---
title: "Netty类库简介"
date: 2025-06-11 19:23:38
categories: netty
tags: netty
---

## **类库简介**

`2002`年的时候，`Sum`公司推出了`JDK1.4`并且新增了`NIO`的类库，弥补了原来同步阻塞`I/O`带来的不足，官方称之为`New I/O`，寓意指新的`I/O`编程模型，但是由于旧版的`Block I/O`在民间更喜欢称它为`Non Block I/O`（非阻塞I/O）编程模型，在`NIO`的类库中，将原本`java.net.Socket` 以及 `java.net.ServerSocket` 分别升级成 `java.nio.SocketChannel` 和 `java.nio.ServerSocketChannel`，它们都支持阻塞与非阻塞模式，前者性能与可靠性较差，后者却恰恰相反，在开发过程中可以选取适合自己的模式，一般来说，低负载、低并发的应用程序可以选择同步阻塞IO以降低编程复杂度。但是对于高负载、高并发的网络应用，需要使用NIO的非阻塞模式进行开发….

### **Buffer**

`Buffer`是一个含读写数据操的作对象，在`NIO`库中，所有的对象都是用缓冲处理的，读写数据操作时都是通过缓冲区来处理，实际上它是一个数组，但通常它是一个字节数组（ByteBuffer），也可以使用其它种类的数组，但是一个缓冲区不仅仅是一个数组，缓冲区提供了对数据结构化访问及维护读写位置（limit）等信息…

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932861135.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

### **Channel**

`Channel`是一个`全双工`的通道（同时支持双向传输，在`BIO`中都是单向流，即`InputStream` 和 `OutputStream`），因为是双向的，所以它可以更好的映射底层操作系统的API，特别是在UNIX网络编程模型中，底层操作系统的通道都是双全工的，同时支持读写…

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932861390.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

### **Selector**

`Selector`是`NIO`中的基础，对`NIO`编程至关重要，它是一个多路复用器，提供选择已经准备就绪的任务功能，会不断轮训注册在它上面的`Channel`，如果某个`Channel`上面有新的TCP请求接入，它就会处于就绪状态，供`Selector`轮训出来，然后可与通过`SelectorKey`获取就绪的`Channel`集合，从而进行`I/O`操作…

### **异步非阻塞服务端实现**

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932861722.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

**注意事项**

- 与Selector 使用的 Channel 必须处于非阻塞模式

- 每次使用Selector时应该先判断下Selector是否已经被关闭，否则容易出现java.nio.channels.ClosedSelectorException错误

``` java
public static void main(String[] args) {
    int port = 4040;
    MultiplexerTimeServer timeServer = new MultiplexerTimeServer(port);
    new Thread(timeServer,"NIO-MultiplexerTimeServer-1").start();
}
```

> 通过`TimeServer`的时序图，我们来看下实现的`异步非阻塞的TimeServer`代码

``` java
public class MultiplexerTimeServer implements Runnable {
    private Selector selector;
    private ServerSocketChannel serverSocketChannel;
    public MultiplexerTimeServer(int port) {
        try {
            selector = Selector.open();//打开多路复用器
            serverSocketChannel = ServerSocketChannel.open();//打开ServerSocket通道
            serverSocketChannel.configureBlocking(false);//设置异步非阻塞模式,与Selector使用 Channel 必须处于非阻塞模式
            serverSocketChannel.bind(new InetSocketAddress(port), 1024);//绑定端口为4040并且初始化系统资源位1024个
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);//将Channel管道注册到Selector中去,监听OP_ACCEPT操作
            System.out.println("TimeServer启动成功,当前监听的端口 : " + port);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);//如果初始化失败，退出
        }
    }
    @Override
    public void run() {
        while (true) {
            try {
                //int select = this.selector.select(1000); 1S唤醒一次,加休眠时间则可以不要if(select > 0 )
                if (!selector.isOpen()) {
                    System.out.println("selector is closed");
                    break;
                }
                int select = selector.select();
                if (select > 0) {
                    Set<SelectionKey> selectionKeys = this.selector.selectedKeys();
                    Iterator<SelectionKey> it = selectionKeys.iterator();
                    while (it.hasNext()) {
                        SelectionKey key = it.next();
                        it.remove();//删掉处理过的key
                        try {
                            handleInput(key);
                        } catch (Exception e) {
                            if (key != null) {
                                key.cancel();
                                if (key.channel() != null)
                                    key.channel().close();
                            }
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    private void handleInput(SelectionKey key) throws IOException {
        if (key.isValid()) {//处理新接入的请求消息
            if (key.isAcceptable()) {
                SocketChannel accept = ((ServerSocketChannel) key.channel()).accept();
                accept.configureBlocking(false);
                //添加新的连接到selector中
                accept.register(this.selector, SelectionKey.OP_READ);
            }
            if (key.isReadable()) {//读取数据
                SocketChannel sc = (SocketChannel) key.channel();
                ByteBuffer buffer = ByteBuffer.allocate(1024);//一次最多读取1024
                int read = sc.read(buffer);
                if (read > 0) {
                    buffer.flip();//反转缓冲区
                    byte[] bytes = new byte[buffer.remaining()];
                    buffer.get(bytes);
                    String msg = new String(bytes, "UTF-8");
                    System.out.println("TimeServer 接收到的消息 :" + msg);
                    doWrite(sc, "挽歌君老帅了...");
                } else if (read < 0) {
                    key.cancel();
                    sc.close();
                } else {
                    //读取0个字节忽略
                }
            }
        }
    }
    private void doWrite(SocketChannel channel, String resp) throws IOException {
        if (resp != null && resp.trim().length() > 0) {
            byte[] bytes = resp.getBytes();
            ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);//根据字节大小创建一个Buffer
            writeBuffer.put(bytes);//将字节数组复制到缓冲区
            writeBuffer.flip();//反转缓冲区
            channel.write(writeBuffer);//调用管道API将数据写出
        }
    }
}
```

- select()阻塞到至少有一个通道在你注册的事件上就绪了。

- select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。

- selectNow()不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。）。

- select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。

- 如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。

- 如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。

- 调用Selector.wakeup()方法即可。阻塞在select()方法上的线程会立马返回。

### **客户端实现**

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932862092.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

``` java
public class TimeClient {
    public static void main(String[] args) {
        int port = 4040;
        new Thread(new TimeClientHandler("127.0.0.1",port),"NIO-MultiplexerTimeServer-1").start();
    }
}
```

> 与之前的`TimeClient`不同的是这次通过`TimeClientHandler`线程来处理异步连接和读写操作

``` java
public class TimeClientHandler implements Runnable {
    private String host;
    private int port;
    private Selector selector;
    private SocketChannel socketChannel;
    private volatile boolean stop;
    public TimeClientHandler(String host, int port) {
        this.host = host == null ? "127.0.0.1" : host;
        this.port = port;
        try {
            selector = Selector.open();//打开连接
            socketChannel = SocketChannel.open();//打开连接
            //TODO 该处存在一个小BUG,第一条数据会丢失
            //TODO 如果该处设为true阻塞,则客户端会报错,应该是本人写法还存在点问题
            socketChannel.configureBlocking(false);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);//异常情况断开连接
        }
    }
    @Override
    public void run() {
        try {
            doConnect();
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);//异常情况断开连接
        }
        while (!stop) {
            try {
                if (selector.select() > 0) {
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> iterator = selectionKeys.iterator();
                    while (iterator.hasNext()) {
                        SelectionKey selectionKey = iterator.next();
                        iterator.remove();
                        try {
                            handleInput(selectionKey);
                        } catch (Exception e) {
                            if (selectionKey != null) {
                                selectionKey.cancel();
                                if (selectionKey.channel() != null) {
                                    selectionKey.channel().close();
                                }
                            }
                        }
                    }
                }
            } catch (IOException e) {
                System.exit(1);
            }
        }
        if (selector != null) {
            try {
                selector.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    private void handleInput(SelectionKey key) throws IOException {
        if (key.isValid()) {//验证这个key是否有效
            SocketChannel channel = (SocketChannel) key.channel();
            if (key.isConnectable()) {//判断SocketChannel是否处于连接状态
                if (channel.finishConnect()) {//判断 SocketChannel 是否连接成功
                    channel.register(this.selector, SelectionKey.OP_READ);//连接成功则注册OP_READ事件到Selector选择器中
                    doWrite(channel);
                } else {
                    System.exit(1);
                }
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);//创建读取所需Buffer
                int read = channel.read(readBuffer);
                if (read > 0) {
                    readBuffer.flip();//反转缓冲区
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String msg = new String(bytes, "UTF-8");
                    System.out.println("TimeClient 接收到的消息:" + msg);
                }
                this.stop = true;//如果接收完毕退出循环
            }
        }
    }
    private void doConnect() throws IOException {
        if (socketChannel.connect(new InetSocketAddress(host, port))) {
            socketChannel.register(this.selector, SelectionKey.OP_READ);
            doWrite(socketChannel);
        } else {
            socketChannel.register(this.selector, SelectionKey.OP_CONNECT);//向Reactor线程的Selector注册OP_CONNECT事件
        }
    }
    private void doWrite(SocketChannel channel) throws IOException {
        byte[] req = "帅不帅".getBytes();
        ByteBuffer buffer = ByteBuffer.allocate(req.length);
        buffer.put(req);//将字节数组复制到缓冲区
        buffer.flip();//反转缓冲区
        channel.write(buffer);
        if (!buffer.hasRemaining()) {
            System.out.println("消息发送成功");
        }
    }
}
```

### **测试**

分别启动`TimeServer` 和 `TimeClient`

> 服务端

``` java
TimeServer 接收到的消息 :挽歌君帅不帅
```

> 客户端

``` java
消息发送成功
TimeClient 接收到的消息:老帅了...
```

#### **– 遗留问题**

**1、** 该处存在一个小BUG，客户端向服务端发送请求，服务端第一次数据回写客户端未接收到，但是后续的请求都有接收到，这应该是异步请求导致；
