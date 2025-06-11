---
title: "NIO BIO AIO"
date: 2025-06-11 19:23:39
categories: IO
tags: IO
---

## java中的NIO.BIO和AIO

IO常写作I/O，是Input/Ouput的简称，即输入/输出。通常指数据在那边存储器（内存）和外部存储器（硬盘等）或其它周边设备之间的输入和输出

在java中，提供了一系列API，可以供开发这来读写外部数据或文件，我们称这些API为JAVA IO

IO是java中比较重要且比较难的知识点，主要是因为随着java的发展，目前有三种io共存，分别是BIO，NIO和AIO

##### BIO

全称Block IO，是一种**<span color="rgb(220, 38, 38)" fontsize="" style="color: rgb(220, 38, 38)">同步且阻塞</span>**的通信模式，是一个比较传统的通信方式，模式简单，使用方便，但并非处理能力低，通信耗时，依赖网速

##### NIO

全称None Block IO，是java SE 1.4版本之后，针对网络传输效能优化的新功能，是一种**<span color="#dc2626" style="color: #dc2626">同步非阻塞</span>**的通信方式

##### AIO

<span style="font-size: 16px; color: rgb(44, 62, 80)">全称 Asynchronous IO，是</span>**<span color="#dc2626" style="color: #dc2626">异步非阻塞</span>**<span style="font-size: 16px; color: rgb(44, 62, 80)">的 IO。是一种非阻塞异步的通信模式</span>

同步阻塞模式：这种模式下，我们的工作模式是先来到厨房，开始烧水，并坐在水壶面前一直等着水烧开

同步非阻塞模式：这种模式下，我们的工作模式是先来到厨房，开始烧水，但是我们不一直坐在水壶前面等，而是回到客厅看电视，然后每隔几分钟到厨房看一下水有没有烧开。

异步非阻塞 I/O 模型：这种模式下，我们的工作模式是先来到厨房，开始烧水，我们不一直坐在水壶前面等，也不隔一段时间去看一下，而是在客厅看电视，水壶上面有个开关，水烧开之后他会通知我

##### **适用场景**

BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。

NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。

AIO 方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用 OS 参与并发操作，编程比较复杂，JDK7 开始支持。

##### <span style="font-size: 16px; color: rgb(44, 62, 80)">使用 BIO 实现文件的读取和写入</span>

``` java
public class BioFileDemo {
    public static void main(String[] args) {
        BioFileDemo demo = new BioFileDemo();
        demo.writeFile();
        demo.readFile();
    }

    // 使用 BIO 写入文件
    public void writeFile() {
        String filename = "logs/coding.txt";
        try {
            FileWriter fileWriter = new FileWriter(filename);
            BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);

            bufferedWriter.write("BIO demo");
            bufferedWriter.newLine();

            System.out.println("写入完成");
            bufferedWriter.close();
            fileWriter.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 使用 BIO 读取文件
    public void readFile() {
        String filename = "logs/itwanger/paicoding.txt";
        try {
            FileReader fileReader = new FileReader(filename);
            BufferedReader bufferedReader = new BufferedReader(fileReader);

            String line;
            while ((line = bufferedReader.readLine()) != null) {
                System.out.println("读取的内容: " + line);
            }

            bufferedReader.close();
            fileReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

##### <span style="font-size: 16px; color: rgb(44, 62, 80)">使用 NIO 实现文件的读取和写入</span>

``` java
public class NioFileDemo {
    public static void main(String[] args) {
        NioFileDemo demo = new NioFileDemo();
        demo.writeFile();
        demo.readFile();
    }

    // 使用 NIO 写入文件
    public void writeFile() {
        Path path = Paths.get("logs/coding.txt");
        try {
            FileChannel fileChannel = FileChannel.open(path, EnumSet.of(StandardOpenOption.CREATE, StandardOpenOption.WRITE));

            ByteBuffer buffer = StandardCharsets.UTF_8.encode("NIO demo");
            fileChannel.write(buffer);

            System.out.println("写入完成");
            fileChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 使用 NIO 读取文件
    public void readFile() {
        Path path = Paths.get("logs/itwanger/paicoding.txt");
        try {
            FileChannel fileChannel = FileChannel.open(path, StandardOpenOption.READ);
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            int bytesRead = fileChannel.read(buffer);
            while (bytesRead != -1) {
                buffer.flip();
                System.out.println("读取的内容: " + StandardCharsets.UTF_8.decode(buffer));
                buffer.clear();
                bytesRead = fileChannel.read(buffer);
            }

            fileChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

<span style="font-size: 16px; color: rgb(44, 62, 80)">这个示例演示了如何使用 NIO 的 </span><a href="https://javabetter.cn/nio/buffer-channel.html" rel="noopener noreferrer" target="_blank"><strong>FileChannel<span color="" data-fontsize="">open in new window</span></strong></a><span style="font-size: 16px; color: rgb(44, 62, 80)"> 对文件进行读写操作。在 </span>`writeFile()`<span style="font-size: 16px; color: rgb(44, 62, 80)"> 方法中，我们首先打开文件通道并指定创建和写入选项。接着，将要写入的字符串转换为 ByteBuffer，然后使用 </span>`fileChannel.write()`<span style="font-size: 16px; color: rgb(44, 62, 80)"> 方法将其写入文件。在 </span>`readFile()`<span style="font-size: 16px; color: rgb(44, 62, 80)"> 方法中，我们打开文件通道并指定读取选项，然后创建一个 ByteBuffer 用于存储读取到的数据。使用 </span>`fileChannel.read()`<span style="font-size: 16px; color: rgb(44, 62, 80)"> 方法循环读取文件内容，直到返回 -1 表示读取完毕。在循环中，我们翻转缓冲区，将其解码为字符串并打印，然后清空缓冲区以进行下一次读取。最后，关闭文件通道</span>

##### <span style="font-size: 16px; color: rgb(44, 62, 80)">使用 AIO 实现文件的读取和写入</span>

``` java
public class AioDemo {

    public static void main(String[] args) {
        AioDemo demo = new AioDemo();
        demo.writeFile();
        demo.readFile();
    }

    // 使用 AsynchronousFileChannel 写入文件
    public void writeFile() {
        // 使用 Paths.get() 获取文件路径
        Path path = Paths.get("logs/coding.txt");
        try {
            // 用 AsynchronousFileChannel.open() 打开文件通道，指定写入和创建文件的选项。
            AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

            // 将要写入的字符串（"AIO demo"）转换为 ByteBuffer。
            ByteBuffer buffer = StandardCharsets.UTF_8.encode("AIO demo");
            // 调用 fileChannel.write() 方法将 ByteBuffer 中的内容写入文件。这是一个异步操作，因此需要使用 Future 对象等待写入操作完成。
            Future<Integer> result = fileChannel.write(buffer, 0);
            // 等待写操作完成
            result.get();

            System.out.println("写入完成");
            fileChannel.close();
        } catch (IOException | InterruptedException | java.util.concurrent.ExecutionException e) {
            e.printStackTrace();
        }
    }

    // 使用 AsynchronousFileChannel 读取文件
    public void readFile() {
        Path path = Paths.get("logs/coding.txt");
        try {
            // 指定读取文件的选项。
            AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.READ);
            // 创建一个 ByteBuffer，用于存储从文件中读取的数据。
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            // 调用 fileChannel.read() 方法从文件中异步读取数据。该方法接受一个 CompletionHandler 对象，用于处理异步操作完成后的回调。
            fileChannel.read(buffer, 0, buffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    // 在 CompletionHandler 的 completed() 方法中，翻转 ByteBuffer（attachment.flip()），然后使用 Charset.forName("UTF-8").decode() 将其解码为字符串并打印。最后，清空缓冲区并关闭文件通道。
                    attachment.flip();
                    System.out.println("读取的内容: " + StandardCharsets.UTF_8.decode(attachment));
                    attachment.clear();
                    try {
                        fileChannel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    // 如果异步读取操作失败，CompletionHandler 的 failed() 方法将被调用，打印错误信息。
                    System.out.println("读取失败");
                    exc.printStackTrace();
                }
            });

            // 等待异步操作完成
            Thread.sleep(1000);
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

##### 小结

BIO（Blocking I/O）：采用阻塞式 I/O 模型，线程在执行 I/O 操作时被阻塞，无法处理其他任务，适用于连接数较少且稳定的场景。

NIO（New I/O 或 Non-blocking I/O）：使用非阻塞 I/O 模型，线程在等待 I/O 时可执行其他任务，通过 Selector 监控多个 Channel 上的事件，提高性能和可伸缩性，适用于高并发场景。

AIO（Asynchronous I/O）：采用异步 I/O 模型，线程发起 I/O 请求后立即返回，当 I/O 操作完成时通过回调函数通知线程，进一步提高了并发处理能力，适用于高吞吐量场景
