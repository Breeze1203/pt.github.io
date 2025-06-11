---
title: "粘包和拆包"
date: 2025-06-11 19:23:40
categories: netty
tags: netty
---

## **粘包和拆包**

学过`TCP`的都知道，它是属于传输层的协议，传输层除了有`TCP`协议外还有`UDP`协议，但是`UDP`是不存在`拆包和粘包`的。`UDP`是基于报文发送的，从`UDP`的帧结构可以看出，在`UDP`首部采用了`16bit`来指示`UDP`数据报文的长度，因此在应用层能很好的将不同的数据报文区分开，从而避免粘包和拆包的问题。

而`TCP`是基于字节流的，虽然应用层和`TCP`传输层之间的数据交互是大小不等的数据块，但是`TCP`把这些数据块仅仅看成一连串无结构的字节流，没有边界；另外从`TCP`的帧结构也可以看出，在`TCP`的首部没有表示数据长度的字段，基于上面两点，在使用`TCP`传输数据时，才有粘包或者拆包现象发生的可能。

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932862502.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

- 服务端分两次读取到了两个独立的数据包，分别是D1和D2没有粘包和拆包

- 服务端一次接受到两个粘在一起的数据包，D2和D1，被称为TCP粘包服务端分两次读取到了两个数据包，第一次读取到完整的D1，D2部分内容，第二次读取了D2的剩余内容，这被称之为TCP拆包操作

- 服务端分两次读取到了两个数据包，第一次读取到了D1_1，第二次读取到了D1包的剩余内容和完整的D2数据包

- 如果此时服务端TCP接收滑窗非常小，而数据包内容相对较大的情况，很可能发生服务端多次拆包才能将D1和D2数据接收完整

### **产生原因**

- 要发送的数据大于TCP发送缓冲区剩余空间大小，将会发生拆包。

- 待发送数据大于MSS（最大报文长度），TCP在传输前将进行拆包。

- 要发送的数据小于TCP发送缓冲区的大小，TCP将多次写入缓冲区的数据一次发送出去，将会发生粘包。

- 接收数据端的应用层没有及时读取接收缓冲区中的数据，将发生粘包。

### **解决方法**

通过以上分析，我们清楚了粘包或拆包发生的原因，那么如何解决这个问题呢？解决问题的关键在于如何给每个数据包添加边界信息，常用的方法有如下几个：

- 发送端给每个数据包添加包首部（类似UDP），首部中应该至少包含数据包的长度，这样接收端在接收到数据后，通过读取包首部的长度字段，便知道每一个数据包的实际长度了

- 发送端将每个数据包封装为固定长度（不够的可以通过补0填充），这样接收端每次从接收缓冲区中读取固定长度的数据就自然而然的把每个数据包拆分开来。

- 可以在数据包之间设置边界，如添加特殊符号（如\r\n），这样，接收端通过这个边界就可以将不同的数据包拆分开。

### **Netty 解决方案**

`io.netty.handler.codec.callDecode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out)`

``` java
protected void callDecode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    try {
        while (in.isReadable()) {
            int outSize = out.size();
            int oldInputLength = in.readableBytes();
            decode(ctx, in, out);
            // Check if this handler was removed before continuing the loop.
            // If it was removed, it is not safe to continue to operate on the buffer.
            //
            // See https://github.com/netty/netty/issues/1664
            if (ctx.isRemoved()) {
                break;
            }
            if (outSize == out.size()) {
                if (oldInputLength == in.readableBytes()) {
                    break;
                } else {
                    continue;
                }
            }
            if (oldInputLength == in.readableBytes()) {
                throw new DecoderException(
                        StringUtil.simpleClassName(getClass()) +
                        ".decode() did not read anything but decoded a message.");
            }
            if (isSingleDecode()) {
                break;
            }
        }
    } catch (DecoderException e) {
        throw e;
    } catch (Throwable cause) {
        throw new DecoderException(cause);
    }
}
```

当上面一个`ChannelHandlerContext`传入的`ByteBuf`有数据的时候，这里我们可以把`in`参数看成`网络流`，这里有不断的数据流入，而我们要做的就是从这个`byte`流中分离出`message`，然后把`message`添加给`out`

> 分开描述下代码逻辑：

- 当out中有message的时候，直接将out中的内容交给后面的ChannelHandlerContext去处理

- 当用户逻辑把当前ChannelHandlerContext移除的时候，立即停止对网络数据的处理

- 调用readableBytes记录当前in中可读字节数

- decode是抽象方法，交给子类具体实现

- 判断当前ChannelHandlerContext是移除的时候，立即停止对网络数据的处理

- 如果子类实现没有分理出任何message的时候，且子类实现也没有动ByteBuf中的数据的时候，这里直接跳出，等待后续有数据来了再进行处理

- 如果子类实现没有分理出任何message的时候，且子类实现动了ByteBuf的数据，则继续循环，直到解析出message或者不在对ByteBuf中数据进行处理为止

- 如果子类实现解析出了message但是又没有动ByteBuf中的数据，那么是有问题的，抛出异常。

- 如果标志位只解码一次，则退出

如果要实现具有处理粘包、拆包功能的子类，及decode实现，必须要遵守上面的规则，我们以实现处理第一部分的第二种粘包情况和第三种情况拆包情况的服务器逻辑来举例：

粘包：`decode`需要实现的逻辑对应于将客户端发送的两条消息都解析出来分为两个`message`加入`out`，这样的话`callDecode`只需要调用一次`decode`即可。

拆包：`decode`需要实现的逻辑主要对应于处理第一个数据包的时候，第一次调用`decode`的时候`out`的`size`不变，从`continue`跳出并且由于不满足继续可读而退出循环，处理第二个数据包的时候，对于`decode`的调用将会产生两个`message`放入`out`，其中两次进入`callDecode`上下文中的数据流将会合并为一个`ByteBuf`和当前`ChannelHandlerContext`实例关联，两次处理完毕即清空这个`ByteBuf`。

尽管介绍了`ByteToMessageDecoder`，用户自己去实现处理粘包、拆包的逻辑还是有一定难度的，`Netty`已经提供了一些基于不同处理粘包、拆包规则的实现，我们可以根据规则自行选择使用`Netty`提供的`Decoder`来进行具有粘包、拆包处理功能的网络应用开发。

- DelimiterBasedFrameDecoder 基于消息边界方式进行粘包拆包处理的。

- FixedLengthFrameDecoder 基于固定长度消息进行粘包拆包处理的。

- LengthFieldBasedFrameDecoder 基于消息头指定消息长度进行粘包拆包处理的。

- LineBasedFrameDecoder 基于行来进行消息粘包拆包处理的。

## **测试一把**

通过例子，来更加清晰的认识`TCP`粘包/拆包带来的问题，以及使用`Netty`内置的解决方案解决粘包/拆包的问题

### **异常情况**

> 继<a href="http://ddkk.com/?p=637" rel="nofllow noopener" target="_blank">上一章</a>的代码，我们改造`TimeServerHandler`中的`channelRead`方法

``` java
private static class TimeServerHandler extends ChannelHandlerAdapter {
    private int counter;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8").substring(0, req.length - System.getProperty("line.separator").length());
        System.out.println("TimeServer 接收到的消息 :" + body + "; 当前统计:" + ++counter);
        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? String.valueOf(System.currentTimeMillis()) : "BAD ORDER";
        currentTime = currentTime + System.getProperty("line.separator");
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        ctx.write(resp);
    }
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //将消息队列中信息写入到SocketChannel中去,解决了频繁唤醒Selector所带来不必要的性能开销
        //Netty的 write 只是将消息放入缓冲数组,再通过调用 flush 才会把缓冲区的数据写入到 SocketChannel
        ctx.flush();
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();//发生异常时候，执行重写后的 exceptionCaught 进行资源关闭
    }
}
```

> 改造`TimeClientHandler`代码

``` java
private static class TimeClientHandler extends ChannelHandlerAdapter {
    private byte[] req;
    public TimeClientHandler() {
        req = ("QUERY TIME ORDER" + System.getProperty("line.separator")).getBytes();
    }
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf message = null;
        for (int i = 0; i < 100; i++) {
            message = Unpooled.buffer(req.length);
            message.writeBytes(req);
            ctx.writeAndFlush(message);
        }
    }
    private int counter;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("TimeClient 接收到的消息 :" + body + "; 当前统计:" + ++counter);
        ctx.close();//接受完消息关闭连接
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("释放资源:" + cause.getMessage());//不重写将会看到堆栈信息以及资源无法关闭
        ctx.close();
    }
}
```

> 分别启动`TimeServer`和`TimeClient`两个程序

``` java
绑定端口,同步等待成功......
TimeServer 接收到的消息 :QUERY TIME ORDER
QUERY TIME ORDER
......省略部分 QUERY TIME ORDER
QUERY TIME ORDER
QUERY TIME ORDER
QUERY TIME ORD; 当前统计:1
QUERY TIME ORDER
TimeServer 接收到的消息 :
......省略部分 QUERY TIME ORDER
QUERY TIME ORDER; 当前统计:2
```

``` java
TimeClient 接收到的消息 :BAD ORDER
BAD ORDER
; 当前统计:1
```

从上面的日志中，我们可以发现`服务端`发生`TCP`粘包的情况，正确情况应该是服务端输出`100`条含`TimeServer 接收到的消息 :QUERY TIME ORDER; 当前统计:counter`的日志，而且客户端只接收了部分断断续续的数据，说明返回时也发生了粘包…

### **解决之道**

我们在上文说了`LineBasedFrameDecoder`是一个基于行的解码器，从源码中可以看到它是根据`\n`或者`\r\n`判断的，当`ByteBuf`存在这样的字符就认为是一个完整的数据包，这样可以有效的避免数据粘包或者拆包的情况，从而保证我们消息的有效传输，接下来我们就玩一玩`Netty`的`LineBasedFrameDecoder`…..

``` java
/**
 * A decoder that splits the received {@link ByteBuf}s on line endings.
 * <p>
 * Both {@code "\n"} and {@code "\r\n"} are handled.
 * For a more general delimiter-based decoder, see {@link DelimiterBasedFrameDecoder}.
 */
public class LineBasedFrameDecoder extends ByteToMessageDecoder {
    protected Object decode(ChannelHandlerContext ctx, ByteBuf buffer) throws Exception {
        final int eol = findEndOfLine(buffer);
        if (!discarding) {
            if (eol >= 0) {
                final ByteBuf frame;
                final int length = eol - buffer.readerIndex();
                final int delimLength = buffer.getByte(eol) == '\r'? 2 : 1;
            }
        }
        ......
    }
}
```

修改`TimeServer`的`ChildChannelHandler`和`TimeServerHandler`内部类，添加了`LineBasedFrameDecoder`和`StringDecoder`两个解码器，同时向客户端回写系统当前时间戳，记得我们这里是用`\n`做换行处理的

``` java
private static class ChildChannelHandler extends ChannelInitializer {
    @Override
    protected void initChannel(Channel channel) throws Exception {
        channel.pipeline().addLast(new LineBasedFrameDecoder(1024));//划重点了，拿笔记下
        channel.pipeline().addLast(new StringDecoder());
        channel.pipeline().addLast(new TimeServerHandler());
    }
    private static class TimeServerHandler extends ChannelHandlerAdapter {
        private int counter;
        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            String body = (String) msg;
            System.out.println("TimeServer 接收到的消息 :" + body + "; 当前统计:" + ++counter);
            String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? String.valueOf(System.currentTimeMillis())+"\n"  : "BAD ORDER";
            ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
            ctx.write(resp);
        }
        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
            //将消息队列中信息写入到SocketChannel中去,解决了频繁唤醒Selector所带来不必要的性能开销
            //Netty的 write 只是将消息放入缓冲数组,再通过调用 flush 才会把缓冲区的数据写入到 SocketChannel
            ctx.flush();
        }
        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
            ctx.close();//发生异常时候，执行重写后的 exceptionCaught 进行资源关闭
        }
    }
}
```

修改`TimeClient`中`connect`的`handler`内部类

``` java
protected void initChannel(SocketChannel channel) throws Exception {
    channel.pipeline().addLast(new LineBasedFrameDecoder(1024));//划重点了，拿笔记下
    channel.pipeline().addLast(new StringDecoder());
    channel.pipeline().addLast(new TimeClientHandler());
}
```

修改`TimeClientHandler`中的`channelRead`读取数据的方法

``` java
private static class TimeClientHandler extends ChannelHandlerAdapter {
    private byte[] req;
    private int counter;
    public TimeClientHandler() {
        req = ("QUERY TIME ORDER\n").getBytes();
    }
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf message = null;
        for (int i = 0; i < 100; i++) {
            message = Unpooled.buffer(req.length);
            message.writeBytes(req);
            ctx.writeAndFlush(message);
        }
    }
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("TimeClient 接收到的消息 :" + body + "; 当前统计:" + ++counter);
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("释放资源:" + cause.getMessage());//不重写将会看到堆栈信息以及资源无法关闭
        ctx.close();
    }
}
```

> 分别启动`TimeServer`和`TimeClient`两个程序

``` java
绑定端口,同步等待成功......
TimeServer 接收到的消息 :QUERY TIME ORDER; 当前统计:1
TimeServer 接收到的消息 :QUERY TIME ORDER; 当前统计:2
.......此处省略一大堆日志
TimeServer 接收到的消息 :QUERY TIME ORDER; 当前统计:99
TimeServer 接收到的消息 :QUERY TIME ORDER; 当前统计:100
```

``` java
TimeClient 接收到的消息 :1504274365781; 当前统计:1
TimeClient 接收到的消息 :1504274365785; 当前统计:2
.......此处省略一大堆日志
TimeClient 接收到的消息 :1504274365785; 当前统计:99
TimeClient 接收到的消息 :1504274365785; 当前统计:100
```
