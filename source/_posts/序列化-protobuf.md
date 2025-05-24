---
title: "序列化 Protobuf"
date: 2024-11-18 15:41:52
---

## **Java序列化的弱点**

- 不支持跨语言，当我们进行跨应用之间的服务调用的时候如果另外一个应用使用PHP语言来开发，这个时候我们发送过去的序列化对象，别人是无法进行反序列化的因为其内部实现对于别人来说完全就是黑盒。

- 序列化效率低下，字节流过大，这个我们可以做一个实验，还是上一节中的OrderRequest类，我们分别用java的序列化和使用二进制编码来做一个对比

> 序列化后字节码对比

``` java
@Test
public void test1() throws IOException {
    Order order = new Order(1, "Levin", "Netty Book", "130****1912", "China");
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    ObjectOutputStream os = new ObjectOutputStream(out);
    os.writeObject(order);
    os.flush();
    System.out.println("JDK序列化后的长度： " + out.toByteArray().length);
    os.close();
    out.close();
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    buffer.put(order.getAddress().getBytes());
    buffer.put(order.getPhoneNumber().getBytes());
    buffer.put(order.getUserName().getBytes());
    buffer.put(order.getProductName().getBytes());
    buffer.flip();
    byte[] result = new byte[buffer.remaining()];
    buffer.get(result);
    System.out.println("使用二进制序列化的长度：" + result.length);
}
```

``` java
JDK序列化后的长度： 285
使用二进制序列化的长度：31
```

> JDK序列化`10000(1万)`次与`1000000(百万)`次所使用时长

``` java
@Test
public void test2() {
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    try {
        long jdkTime = System.currentTimeMillis();
        IntStream.range(1, 10000).forEach(id -> {
            Order order = new Order(id, "Levin", "Netty Book", "130****1912", "China");
            try {
                ObjectOutputStream os = new ObjectOutputStream(out);
                os.writeObject(order);
                os.flush();
                out.toByteArray();
                os.close();
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
        long endTime = System.currentTimeMillis();
        System.out.println("jdk序列化10000次耗时：" + (endTime - jdkTime));
        System.out.println("--------------------------------------------------------------------------------");
        long buffTime = System.currentTimeMillis();
        IntStream.range(1, 1000000).forEach(id -> {
            Order order = new Order(id, "Levin", "Netty Book", "130****1912", "China");
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put(order.getAddress().getBytes());
            buffer.put(order.getPhoneNumber().getBytes());
            buffer.put(order.getUserName().getBytes());
            buffer.put(order.getProductName().getBytes());
            buffer.flip();
            byte[] result = new byte[buffer.remaining()];
            buffer.get(result);
        });
        System.out.println("ByteBuffer序列化1000000次耗时：" + (System.currentTimeMillis() - buffTime));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

``` java
jdk序列化10000次耗时：4085
--------------------------------------------------------------------------------
ByteBuffer序列化1000000次耗时：316
```

**注意** 从单个简单的对象序列化结果来看，`JDK自带`的相比`ByteBuffer`的性能来说`惨不忍睹`，且`相差百倍`的序列化次数所带来的时间，耗损，从`test2`可以看到，`百万次`的`ByteBuffer`花费的时间远远低于`1万次`的`JDK序列化的时间`，更不用说对比`Hession`，`ProtoBuf`，`Kryo`这些专门做序列化的组件……

## **Protobuf序列化的使用**

Protobuf(以下简称PB)是google 的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件，由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域…..

我们先来使用`Protobuf`进行序列化，他和`XML`，`JSON`一样都有自己的语法，`xml`的后缀是`.xml`，`json`文件的后缀是`.json`，Protobuf文件的后缀就是`.proto`…..

Windows：点击<a href="https://image.battcn.com/article/images/20170911/netty/7/protoc-3.4.0-win32.zip" rel="nofllow noopener" target="_blank">https://image.battcn.com/article/images/20170911/netty/7/protoc-3.4.0-win32.zip</a>下载`protoc.exe`（`本章代码附该工具`），编译出来的`JAVA`文件需要与`Maven`配置一致，否则容易出现错误…

Linux：克隆官方代码去`make file`

``` java
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.4.0</version>
</dependency>
```

### **编写OrderProto.proto文件**

``` java
package netty;
option java_package = "com.battcn.netty.proto";
option java_outer_classname = "OrderProto";
message OrderRequest{
    required int32 orderId = 1;
    required string userName = 2;
    required string productName = 3;
    required string phoneNumber = 4;
    required string address = 5;
message OrderResponse{
    required int32 orderId = 1;
    required string respCode = 2;
    required string desc = 3;
}
```

- option java_package生成出来的JAVA包名

- option java_outer_classname生成出来的类名

- message OrderRequest代表序列化的对象

- required int32表示JAVA中的int

<img src="https://cdn.itdevtools.com/images/2023/12/7/157/1701932862721.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

``` java
protoc.exe ./OrderProto.proto --java_out=./
```

### **OrderServer**

``` java
@Override
protected void initChannel(SocketChannel channel) throws Exception {
    channel.pipeline().addLast(new ProtobufVarint32FrameDecoder());
    channel.pipeline().addLast(new ProtobufDecoder(OrderProto.OrderRequest.getDefaultInstance()));
    channel.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
    channel.pipeline().addLast(new ProtobufEncoder());
    channel.pipeline().addLast(new OrderServerHandler());
}
```

重写`initChannel`方法，相比`ObjectEncoder`，使用`Protobuf`相对复杂一点，需要配置`ProtobufVarint32FrameDecoder`半包处理，随后添加`ProtobufDecoder`解码器，告知`Protobuf`需要对那个`POJO`进行解码操作，否则从字节中我们是无法知道目标类型…

``` java
private static class OrderServerHandler extends ChannelHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        OrderProto.OrderRequest request = (OrderProto.OrderRequest) msg;
        System.out.println("Service Accept Client Order Request :[" + request + "]");
        OrderProto.OrderResponse.Builder builder = OrderProto.OrderResponse.newBuilder();
        builder.setOrderId(request.getOrderId());
        builder.setRespCode("200");
        builder.setDesc("Order Submit Successfully");
        ctx.writeAndFlush(builder);
    }
}
```

在`订单服务处理器`中，获取`POJO`需要调用`Builder`进行构建，由于`ProtobufDecoder`已经对消息进行解码操作，所以我们依旧只需要做一次强制类型转换即可，无需手工编码…

### **OrderClient**

``` java
@Override
protected void initChannel(SocketChannel channel) throws Exception {
    channel.pipeline().addLast(new ProtobufVarint32FrameDecoder());
    channel.pipeline().addLast(new ProtobufDecoder(OrderProto.OrderResponse.getDefaultInstance()));
    channel.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
    channel.pipeline().addLast(new ProtobufEncoder());
    channel.pipeline().addLast(new OrderClientHandler());
}
```

基本与服务端的`initChannel`一致，区别大概就是解码的对象不同了…..

``` java
@Override
public void channelActive(ChannelHandlerContext ctx) throws Exception {
    for (int i = 1; i <= 3; i++) {
        OrderProto.OrderRequest.Builder request = OrderProto.OrderRequest.newBuilder();
        request.setAddress("China");
        request.setOrderId(i);
        request.setPhoneNumber("130****1912");
        request.setProductName("Netty Book");
        request.setUserName("Levin");
        ctx.write(request);
    }
    ctx.flush();
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    System.out.println("Receive Server Response :[" + msg + "]");
}
```

与上一章一样，将`3条`数据打包发送到订单服务端

### **试验一把**

> 分别启动`OrderServer`和`OrderClient`，显示如下日志

``` java
绑定端口,同步等待成功......
Service Accept Client Order Request :[orderId: 1
userName: "Levin"
productName: "Netty Book"
phoneNumber: "130****1912"
address: "China"
Service Accept Client Order Request :[orderId: 2
userName: "Levin"
productName: "Netty Book"
phoneNumber: "130****1912"
address: "China"
Service Accept Client Order Request :[orderId: 3
userName: "Levin"
productName: "Netty Book"
phoneNumber: "130****1912"
address: "China"
//////////////////////////////////////////////////////////////////////////////////////////////
Receive Server Response :[orderId: 1
respCode: "200"
desc: "Order Submit Successfully"
Receive Server Response :[orderId: 2
respCode: "200"
desc: "Order Submit Successfully"
Receive Server Response :[orderId: 3
respCode: "200"
desc: "Order Submit Successfully"
]
```

运行结果表明，我们基于`Netty Protobuf`编解码开发订单程序是可以正确工作的，利用`Protobuf`我们无需了解它的实现与细节的情况下就可以开发出跨语言的`远程服务`调用与周边系统`异构`系统进行通信对接……

### **注意事项**

- ProtobufDecoder只负责解码，不支持半包导读，需要依赖能够处理半包导读的解码器

- ProtobufVarint32FrameDecoder 为 Netty 内置支持

- LengthFieldBasedFrameDecoder 通用半包解码器

- ByteToMessageDecoder 继承后需要自己处理半包消息，好处是灵活
