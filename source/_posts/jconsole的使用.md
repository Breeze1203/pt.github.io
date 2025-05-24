---
title: "Jconsole的使用"
date: 2024-11-18 15:41:52
---

Jconsole，JDK自带的，一种基于JMX的可视化监视、管理工具。

可以在系统有一定负荷的情况下使用；对垃圾回收算法有很详细的跟踪。

    jconsole [ -interval=n ] [ -notile ] [ -pluginpath <path> ] [ -version ] [ connection ... ]

      -interval   将更新间隔设置为 n 秒 (默认值为 4 秒)
      -notile     初始不平铺窗口 (对于两个或多个连接)
      -pluginpath 指定 jconsole 用于查找插件的路径
      -version    输出程序版本

      connection = pid || host:port || JMX URL (service:jmx:<协议>://...)
      pid         目标进程的进程 ID
      host        远程主机名或 IP 地址
      port        远程连接的端口号

      -J          指定运行 jconsole 的 Java 虚拟机
                  的输入参数

执行命令后，打开新建连接窗口，有两种

　　（1）本地进程

　　（2）远程进程

<img src="/upload/2-qxdh.png" style="display: inline-block;width:100.0%;height:100.0%" />

1.概述

显示有关Java VM和监视值的概述信息，包括CPU使用情况，内存使用情况，线程计数以及Java VM中加载的类的图形监视信息

<img src="/upload/1-nuem.png" style="display: inline-block;width:100.0%;height:100.0%" />

2.内存

显示有关内存消耗和内存池的信息

<img src="/upload/1-ayet.png" style="display: inline-block;width:100.0%;height:100.0%" />

“内存”选项卡具有“执行GC”按钮，可以随时单击该按钮以执行垃圾回收。

该图表显示了Java VM随时间的内存使用情况，堆和非堆内存以及特定内存池的内存使用情况。

Java VM管理两种内存：堆内存和非堆内存，这两种内存都是在Java VM启动时创建的。

（1）堆内存是运行时数据区，Java VM从中为所有类实例和数组分配内存。堆可以是固定的或可变的大小。垃圾收集器是一个自动内存管理系统，可回收对象的堆内存。

　　A.Eden Space：

　　　　伊甸区，对象被创建的时候首先放到Eden Space，进行垃圾回收后，不能被回收的对象被放入到空的Survivor区域

　　B.Survivor Space：

　　　　幸存者区，用于保存在eden space内存区域中经过垃圾回收后没有被回收的对象

　　　　Survivor Space分为两个空间大小一样的区域，分别是To Survivor和From Survivor，并且始终保持一个Survivor是空的

　　Eden Space和Survivor Space都属于新生代 

　　对新生代 进行垃圾回收被称为Minor GC（或Young GC），每一次Minor GC后留下来的对象age（就是用来判断对象是否进入老年的标志）加1

　　C.Old Gen：

　　　　老年代，用于存放新生代中经过多次垃圾回收仍然存活的对象，也有可能是新生代分配不了内存的大对象会直接进入老年代。

　　　　经过多次垃圾回收都没有被回收的对象，这些对象的age已经足够old了，就会放入到老年代。

　　　　当老年代被放满之后，虚拟机会进行垃圾回收，称之为Major GC。由于Major GC除并发GC外均需对整个堆进行扫描和回收，因此又称为Full GC

默认的，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )，即：新生代 ( Young ) = 1/3 的堆空间大小。 

<img src="/upload/1-xoqu.png" style="display: inline-block;width:100.0%;height:100.0%" />

老年代 ( Old ) = 2/3 的堆空间大小。其中，新生代 ( Young ) 被细分为 Eden 和 两个 Survivor 区域，这两个 Survivor 区域分别被命名为 from 和 to，以示区分。 

默认的，Edem : from : to = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。*  
*

<img src="/upload/1-afec.png" style="display: inline-block;width:100.0%;height:100.0%" />

heap区即堆内存，整个堆大小=年轻代大小 + 老年代大小

堆内存默认为物理内存的1/64(\<1GB)；默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制，可以通过MinHeapFreeRatio参数进行调整；默认空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制，可以通过MaxHeapFreeRatio参数进行调整。

（2）非堆内存包括在Java VM的内部处理或优化所需的所有线程和内存之间共享的方法区域。

　　它存储每类结构，例如运行时常量池，字段和方法数据，以及方法和构造函数的代码。

　　方法区域在逻辑上是堆的一部分，但是根据实现，Java VM可能不会垃圾收集或压缩它。

　　与堆存储器一样，方法区域可以是固定的或可变的大小。方法区域的内存不需要是连续的。

　　A.Metaspace：

　　　　元空间，是方法区的在HotSpot jvm 中的实现，方法区主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。

　　B.Code Cache：

　　　　HotSpot Java VM还包括代码缓存，其中包含用于编译和存储本机代码的内存。

　　C.Compressed Class Space：

　　　　压缩类空间

3.线程

显示有关线程使用的信息

<img src="/upload/1-yncb.png" style="display: inline-block;width:100.0%;height:100.0%" />

红色：峰值线程数，蓝色：活动线程数。

左下角的线程列表列出了所有活动线程

单击“线程”列表中的线程名称，以显示有关该线程的信息，包括线程名称，状态和堆栈跟踪。

4.类监控加载

显示有关类加载的信息

<img src="/upload/1-jdli.png" style="display: inline-block;width:100.0%;height:100.0%" />

红线是加载的类的总数（包括随后卸载的类），蓝线是当前加载的类的数量。

“详细信息”部分显示自Java VM启动以来加载的类的总数，当前加载的数量和卸载的数量。

通过选中右上角的复选框将类加载跟踪设置为详细输出

5.VM信息  
提供有关Java VM的信息

<img src="/upload/1-bltw.png" style="display: inline-block;width:100.0%;height:100.0%" />

6.MBean

显示了所有在platform. MBeanserver上注册的MBeans的信息

<img src="/upload/1-yjre.png" style="display: inline-block;width:100.0%;height:100.0%" />

左边的树形结构显示了所有的MBean

选择了一个MBean之后，其属性、操作、通知和其他信息会在右边显示
