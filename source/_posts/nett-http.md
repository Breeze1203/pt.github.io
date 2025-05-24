---
title: "nett  http"
date: 2024-11-18 15:41:52
---

### pom依赖

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>netty-http</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>netty :: Http</name>
    <description>netty实现高性能http服务器</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <netty-all.version>4.1.100.Final</netty-all.version>
        <logback.version>1.1.7</logback.version>
        <commons.codec.version>1.10</commons.codec.version>
        <fastjson.version>1.2.51</fastjson.version>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.100.Final</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>${logback.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>${commons.codec.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>utf-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>


</project>
```

### Server

``` java
package com.example.netty;

import org.slf4j.LoggerFactory;
import org.slf4j.Logger;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;

public final class HttpServer {
    private static final Logger logger = LoggerFactory.getLogger(HttpServer.class);
// 创建一个日志记录器实例，用于记录日志信息。
    static final int PORT = 8888;
// 定义服务器监听的端口号为8888。
    public static void main(String[] args) throws Exception {
        /*
        bossGroup：这个事件循环组专门用于接受客户端的连接。在你的代码中，bossGroup 是通过 new NioEventLoopGroup(1) 创建的，
        这意味着它只有一个线程。这个单线程（或单个EventLoop）将负责监听服务器端口，并接受所有进入的连接。
        由于它只处理连接的接受，而不处理其他I/O操作，通常一个线程就足够了，这也是为什么你看到创建时传递了 1 作为参数。
        workerGroup：这个事件循环组用于处理已被 bossGroup 接受的连接之后的所有I/O操作。
        例如，它将负责读取请求数据、处理请求和发送响应。在你的代码中，workerGroup 是通过 new NioEventLoopGroup() 创建的，
        没有指定线程数量，这意味着Netty将根据默认的I/O工作器数量来创建EventLoops。这个组可以根据你的服务器硬件和负载情况配置多个线程，
        以提高并发处理能力。
         */
        EventLoopGroup bossGroup = new NioEventLoopGroup(1); // 监听端口
        EventLoopGroup workerGroup = new NioEventLoopGroup(5); // 处理连接
        try {
            ServerBootstrap b = new ServerBootstrap();
            // 使用ServerBootstrap辅助启动类来初始化服务器。
            b.option(ChannelOption.SO_BACKLOG, 1024);
            // 设置服务器可接受的连接队列大小为1024。
            b.childOption(ChannelOption.TCP_NODELAY, true);
            // 设置TCP_NODELAY选项为true，禁用Nagle算法，确保数据立即发送。
            b.childOption(ChannelOption.SO_KEEPALIVE, true);
            // 设置SO_KEEPALIVE选项为true，启用TCP保活机制。
            b.group(bossGroup, workerGroup)
                    // 设置接受连接的事件循环组和处理连接的事件循环组。
                    .channel(NioServerSocketChannel.class)
                    // 设置用于创建服务器通道的类。
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 添加日志处理器，记录服务器的日志信息。
                    .childHandler(new HttpServerInitializer());
            // 设置用于初始化每个新连接的Channel的ChannelInitializer。
            Channel ch = b.bind(PORT).sync().channel();
            // 绑定服务器到指定端口，并同步等待直到绑定成功。
            logger.info("Netty http server listening on port " + PORT);
            // 记录日志信息，告知服务器正在监听指定端口。
            ch.closeFuture().sync();
            // 等待直到Channel关闭。
        } finally {
            bossGroup.shutdownGracefully();
            // 优雅地关闭bossGroup，释放资源。
            workerGroup.shutdownGracefully();
            // 优雅地关闭workerGroup，释放资源。
        }
    }
}
```

### ServerHandler

    package com.example.netty;

    import java.util.Date;
    import java.util.List;
    import java.util.Map;
    import java.util.Objects;

    import com.example.netty.pojo.User;
    import com.example.netty.serialize.impl.JSONSerializer;
    import io.netty.channel.*;
    import io.netty.handler.codec.http.*;
    import io.netty.util.AsciiString;
    import org.apache.commons.codec.Charsets;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    import static io.netty.handler.codec.http.HttpMethod.GET;
    import static io.netty.handler.codec.http.HttpMethod.POST;

    /**
     * @author pengtao
     */
    public class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {
        private static final Logger logger = LoggerFactory.getLogger(HttpServerHandler.class);
        private static final String FAVICON_ICO = "/favicon.ico";
        private static final AsciiString CONTENT_TYPE_HDR = AsciiString.cached("Content-Type");
        private static final AsciiString CONTENT_LENGTH_HDR = AsciiString.cached("Content-Length");
        private static final AsciiString CONNECTION_HDR = AsciiString.cached("Connection");
        private static final AsciiString KEEP_ALIVE_VAL = AsciiString.cached("keep-alive");

        private HttpRequest request;
        private HttpHeaders headers;

        @Override
        protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
            // 检查是否为HttpRequest对象
            if (!(msg instanceof HttpRequest)) {
                return; // 如果不是HttpRequest，不进行处理
            }
            // 将HttpObject转换为HttpRequest对象
            request = (HttpRequest) msg;
            // 从HttpRequest对象中获取HttpHeaders
            headers = request.headers();
            // 调用自定义方法来进一步处理请求
            handleRequest(ctx); // ctx是ChannelHandlerContext对象，提供了对Channel的引用
        }

        private void handleRequest(ChannelHandlerContext ctx) throws Exception {
            String uri = request.uri();
            if (uri.equals(FAVICON_ICO)) {
                return;
            }
            HttpMethod method = request.method();
            User user = new User();
            user.setUserName("pengtao");
            user.setDate(new Date());
            user.setMethod(method.name());
            if (method.equals(GET)) {
                handleGetRequest();
            } else if (method.equals(POST)) {
                handlePostRequest(ctx);
            } else {
                throw new UnsupportedOperationException("HTTP method " + method + " is not supported");
            }
            writeResponse(ctx, serializeUser(user));
        }

        private void handleGetRequest() {
            logger.info("get请求....");
            QueryStringDecoder queryDecoder = new QueryStringDecoder(request.uri(), Charsets.UTF_8);
            for (Map.Entry<String, List<String>> param : queryDecoder.parameters().entrySet()) {
                for (String value : param.getValue()) {
                    logger.info("{}={}", param.getKey(), value);
                }
            }
        }

        private void handlePostRequest(ChannelHandlerContext ctx) throws Exception {
            logger.info("post请求....");
            dealWithContentType(ctx);
        }

        private void writeResponse(ChannelHandlerContext ctx, byte[] content) {
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK);
            response.content().writeBytes(content);
            response.headers().set(CONTENT_TYPE_HDR, "application/json; charset=UTF-8");
            response.headers().set(CONTENT_LENGTH_HDR, response.content().readableBytes());
            boolean keepAlive = HttpUtil.isKeepAlive(request);
            response.headers().set(CONNECTION_HDR, keepAlive ? KEEP_ALIVE_VAL : null);
            ctx.writeAndFlush(response)
                    .addListener(keepAlive ? ChannelFutureListener.CLOSE : ChannelFutureListener.CLOSE_ON_FAILURE);
        }

        private byte[] serializeUser(User user) {
            try {
                return new JSONSerializer().serialize(user);
            } catch (Exception e) {
                logger.error("Error serializing user", e);
                throw e;
            }
        }

        private void dealWithContentType(ChannelHandlerContext ctx) throws Exception {
            String contentType = getContentType();
            if (Objects.isNull(contentType)) return;
            switch (Objects.requireNonNull(contentType)) {
                case "application/json":
                    parseJsonRequest(ctx);
                    break;
                case "application/x-www-form-urlencoded":
                    parseFormRequest(ctx);
                    break;
                case "multipart/form-data":
                    parseMultipartRequest(ctx);
                    break;
                default:
                    logger.warn("Unsupported content type: {}", contentType);
                    sendError(ctx);
                    break;
            }
        }

        private String getContentType() {
            return headers.get(CONTENT_TYPE_HDR) == null ? null : headers.get(CONTENT_TYPE_HDR).toString().split(";")[0].trim();
        }

        private void parseJsonRequest(ChannelHandlerContext ctx) throws Exception {
        }

        private void parseFormRequest(ChannelHandlerContext ctx) throws Exception {

        }

        private void parseMultipartRequest(ChannelHandlerContext ctx) throws Exception {

        }

        private void sendError(ChannelHandlerContext ctx) {
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.UNSUPPORTED_MEDIA_TYPE);
            ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
        }


        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            logger.error("Exception caught", cause);
            ctx.close();
        }

        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) {
            ctx.flush();
        }
    }

### HttpServerInitializer

``` java
package com.example.netty;

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.HttpServerExpectContinueHandler;


public class HttpServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    public void initChannel(SocketChannel ch) {
        ChannelPipeline p = ch.pipeline();
        /**
         * HttpServerCodec是Netty提供的HTTP请求和响应编解码器，
         * 它将处理HTTP请求和响应的编码和解码。
         */
        p.addLast(new HttpServerCodec());

        /**
         * HttpObjectAggregator是Netty提供的处理器，用于聚合HTTP片段，
         * 例如将HttpRequest和随后的HttpContent聚合成一个FullHttpRequest，
         * 便于后续处理。这里设置的聚合字节大小为1MB。
         */
        p.addLast(new HttpObjectAggregator(1024 * 1024));

        /**
         * HttpServerExpectContinueHandler用于处理HTTP 'Expect: 100-continue' 头部，
         * 当客户端发送的请求包含这个头部时，服务器将先发送100 Continue响应，
         * 然后客户端再发送请求体。这可以用于支持需要分块传输的大型请求体。
         */
        p.addLast(new HttpServerExpectContinueHandler());

        /**
         * HttpHelloWorldServerHandler是我们自定义的HTTP请求处理器，
         * 它将根据接收到的HTTP请求生成响应。
         */
        p.addLast(new HttpServerHandler());
    }
```

<a href="https://github.com/Breeze1203/netty-learning/tree/master/netty-http" target="_blank"><u><span color="#ef4444" style="color: #ef4444">源码demo:https://github.com/Breeze1203/netty-learning/tree/master/netty-http</span></u></a>
