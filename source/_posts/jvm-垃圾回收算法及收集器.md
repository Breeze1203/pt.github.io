---
title: "jvm 垃圾回收算法及收集器"
date: 2024-11-18 15:41:52
---

### **<span color="" fontsize="">主要内容</span>**

- <span color="" fontsize="">概述</span>

- <span color="" fontsize="">对象已死？</span>

- <span color="" fontsize="">垃圾收集器算法</span>

#### **<span color="" fontsize="">概述</span>**

<span color="" fontsize="">前面介绍了Java内存运行时区域的各个部分，其中程序计数器、虚拟机栈、本地方法栈三个区域随线程而生，随线程而灭，栈中的栈桢随着方法的进入和退出而有条不絮执行着出栈和入栈操作。每一个栈桢分配多少内存基本上是类结构确定下来时就已知的，尽管运行期由JIT编译器进行优化，因此这几个区域的内存分配和回收都举办确定性，在这几个区域不需要过多考虑内存回收问题。而</span><u><span color="" fontsize="">java堆和方法区</span></u><span color="" fontsize="">则不一样，一个接口多个实现类需要的内存不一样，一个方法中的多个分支需要的内存也不一样，只有在程序运行期间才知道会创建那些对象，这部分内存回收动态的</span>

#### **<span color="" fontsize="">对象是否存活</span>**

##### **<span color="" fontsize="">引用计数算法</span>**

<span color="" fontsize="">给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就减1，任何时刻计数器都为0的一些就是不可能在被使用的，客观地说，引用计数算法的实现简单，判定的效率也挺高，但是，java语言中没有选用引用计数算法来管理内存，其中最主要的原因时它很难结局是对象实现的相互引用问题</span>

<span color="" fontsize="">举个简单的例子，代码如下，对象ObjA和对象ObjB都有字段instance，赋值令objA.instance = objB及obj.instance=objA，除此之外，这两个对象在无任何引用，实际上这两个对象已经不肯在被访问，但是因为互相引用这对方，导致他们的引用计数都不为0，于是引用计数算法无法通知GC收集器回收他们</span>

    public class ReferenceCountingGC {
        public Object instance=null;
    ​
        private static final int _1MB=1024*1024;
    ​
        /*
         这个成员属性的唯一意义就是占点内存，以便能在GC日志中看清楚是否被回收过
         */
    ​
        private byte[] bigSize = new byte[2 * _1MB];
    ​
        public static void main(String[] args) {
            ReferenceCountingGC.testGC();
        }
        public static void testGC(){
            ReferenceCountingGC objA = new ReferenceCountingGC();
            ReferenceCountingGC objB = new ReferenceCountingGC ();
            objA. instance = objB;
            objB.instance = objA;
            objA = null;
            objB = null;
            //假设在这行发生GC，那么objA和objB是否能被回收?
            System.gc ();
        }
    }
    ​

<span color="" fontsize="">运行结果</span>

    [0.001s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
    [0.006s][info   ][gc] Using G1
    [0.006s][info   ][gc,init] Version: 17.0.8+9-LTS-211 (release)
    [0.006s][info   ][gc,init] CPUs: 8 total, 8 available
    [0.006s][info   ][gc,init] Memory: 8192M
    [0.006s][info   ][gc,init] Large Page Support: Disabled
    [0.006s][info   ][gc,init] NUMA Support: Disabled
    [0.006s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
    [0.006s][info   ][gc,init] Heap Region Size: 1M
    [0.006s][info   ][gc,init] Heap Min Capacity: 8M
    [0.006s][info   ][gc,init] Heap Initial Capacity: 128M
    [0.006s][info   ][gc,init] Heap Max Capacity: 2G
    [0.006s][info   ][gc,init] Pre-touch: Disabled
    [0.006s][info   ][gc,init] Parallel Workers: 8
    [0.006s][info   ][gc,init] Concurrent Workers: 2
    [0.006s][info   ][gc,init] Concurrent Refinement Workers: 8
    [0.006s][info   ][gc,init] Periodic GC: Disabled
    [0.010s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000007000000000-0x0000007000be4000-0x0000007000be4000), size 12468224, SharedBaseAddress: 0x0000007000000000, ArchiveRelocationMode: 1.
    [0.010s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000007001000000-0x0000007041000000, reserved size: 1073741824
    [0.010s][info   ][gc,metaspace] Narrow klass base: 0x0000007000000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
    [0.047s][info   ][gc,task     ] GC(0) Using 3 workers of 8 for full compaction
    [0.047s][info   ][gc,start    ] GC(0) Pause Full (System.gc())
    [0.047s][info   ][gc,phases,start] GC(0) Phase 1: Mark live objects
    [0.048s][info   ][gc,phases      ] GC(0) Phase 1: Mark live objects 0.800ms
    [0.048s][info   ][gc,phases,start] GC(0) Phase 2: Prepare for compaction
    [0.048s][info   ][gc,phases      ] GC(0) Phase 2: Prepare for compaction 0.184ms
    [0.048s][info   ][gc,phases,start] GC(0) Phase 3: Adjust pointers
    [0.049s][info   ][gc,phases      ] GC(0) Phase 3: Adjust pointers 0.682ms
    [0.049s][info   ][gc,phases,start] GC(0) Phase 4: Compact heap
    [0.049s][info   ][gc,phases      ] GC(0) Phase 4: Compact heap 0.143ms
    [0.050s][info   ][gc,heap        ] GC(0) Eden regions: 2->0(3)
    [0.050s][info   ][gc,heap        ] GC(0) Survivor regions: 0->0(0)
    [0.050s][info   ][gc,heap        ] GC(0) Old regions: 0->2
    [0.050s][info   ][gc,heap        ] GC(0) Archive regions: 2->2
    [0.050s][info   ][gc,heap        ] GC(0) Humongous regions: 6->0
    [0.050s][info   ][gc,metaspace   ] GC(0) Metaspace: 404K(576K)->404K(576K) NonClass: 380K(448K)->380K(448K) Class: 23K(128K)->23K(128K)
    [0.050s][info   ][gc             ] GC(0) Pause Full (System.gc()) 8M->1M(14M) 2.550ms
    [0.050s][info   ][gc,cpu         ] GC(0) User=0.01s Sys=0.00s Real=0.00s
    [0.051s][info   ][gc,heap,exit   ] Heap
    [0.051s][info   ][gc,heap,exit   ]  garbage-first heap   total 14336K, used 1514K [0x0000000780000000, 0x0000000800000000)
    [0.051s][info   ][gc,heap,exit   ]   region size 1024K, 1 young (1024K), 0 survivors (0K)
    [0.051s][info   ][gc,heap,exit   ]  Metaspace       used 410K, committed 576K, reserved 1114112K
    [0.051s][info   ][gc,heap,exit   ]   class space    used 24K, committed 128K, reserved 1048576K
    ​

<span color="" fontsize="">以下是对日志中关键部分的解释：</span>

1.  **<span color="" fontsize="">Deprecation Warning</span>**<span color="" fontsize="">:</span>

    - `-XX:+PrintGCDetails is deprecated`<span color="" fontsize="">: 表示</span>`-XX:+PrintGCDetails`<span color="" fontsize="">参数已被弃用，JVM将使用新的日志记录系统</span>`-Xlog:gc*`<span color="" fontsize="">代替。</span>

2.  **<span color="" fontsize="">GC Initialization</span>**<span color="" fontsize="">:</span>

    - `Using G1`<span color="" fontsize="">: 表示JVM使用的是G1垃圾收集器。</span>

    - `Version`<span color="" fontsize="">: JVM的版本信息。</span>

    - `CPUs`<span color="" fontsize="">: 可用的CPU核心数。</span>

    - `Memory`<span color="" fontsize="">: 系统总内存。</span>

    - `Large Page Support`<span color="" fontsize=""> 和 </span>`NUMA Support`<span color="" fontsize="">: 大页支持和非统一内存访问（NUMA）支持的状态。</span>

    - `Compressed Oops`<span color="" fontsize="">: 是否启用了对象指针压缩。</span>

    - `Heap Region Size`<span color="" fontsize="">: 堆区域大小。</span>

    - `Heap Min/Initial/Max Capacity`<span color="" fontsize="">: 堆的最小、初始和最大容量。</span>

3.  **<span color="" fontsize="">Metaspace Initialization</span>**<span color="" fontsize="">:</span>

    - `CDS archive`<span color="" fontsize="">: 类数据共享（Class Data Sharing）存档信息。</span>

    - `Compressed class space`<span color="" fontsize="">: 压缩类空间的内存映射和保留大小。</span>

    - `Narrow klass`<span color="" fontsize="">: 有关对象头中类元数据压缩的信息。</span>

4.  **<span color="" fontsize="">GC Event</span>**<span color="" fontsize="">:</span>

    - `GC(0) Using 3 workers of 8 for full compaction`<span color="" fontsize="">: 第0次GC使用3个工作线程进行全堆压缩。</span>

    - `Pause Full (System.gc())`<span color="" fontsize="">: 由</span>`System.gc()`<span color="" fontsize="">触发的全停顿GC。</span>

5.  **<span color="" fontsize="">GC Phases</span>**<span color="" fontsize="">:</span>

    - `Phase 1: Mark live objects`<span color="" fontsize="">: 标记存活对象阶段。</span>

    - `Phase 2: Prepare for compaction`<span color="" fontsize="">: 准备压缩阶段。</span>

    - `Phase 3: Adjust pointers`<span color="" fontsize="">: 调整指针阶段。</span>

    - `Phase 4: Compact heap`<span color="" fontsize="">: 堆压缩阶段。</span>

6.  **<span color="" fontsize="">GC Details</span>**<span color="" fontsize="">:</span>

    - `Eden regions`<span color="" fontsize="">: Eden区的使用情况，从2个区域减少到0，共有3个区域。</span>

    - `Survivor regions`<span color="" fontsize="">: Survivor区的使用情况，没有变化。</span>

    - `Old regions`<span color="" fontsize="">: Old区的使用情况，从0增加到2个区域。</span>

    - `Archive regions`<span color="" fontsize="">: 存档区的使用情况，没有变化。</span>

    - `Humongous regions`<span color="" fontsize="">: 大对象区域的使用情况，从6个区域减少到0。</span>

7.  **<span color="" fontsize="">Metaspace Details</span>**<span color="" fontsize="">:</span>

    - <span color="" fontsize="">显示了元空间的使用情况，包括非类空间（NonClass）和类空间（Class）的使用、提交和保留的大小。</span>

8.  **<span color="" fontsize="">GC Summary</span>**<span color="" fontsize="">:</span>

    - `Pause Full (System.gc()) 8M->1M(14M) 2.550ms`<span color="" fontsize="">: GC前后的堆使用情况，从8MB减少到1MB，总共有14MB的堆，GC暂停时间为2.550毫秒。</span>

9.  **<span color="" fontsize="">CPU Time</span>**<span color="" fontsize="">:</span>

    - `User=0.01s Sys=0.00s Real=0.00s`<span color="" fontsize="">: 用户时间、系统时间和实际时间，显示GC操作的持续时间。</span>

10. **<span color="" fontsize="">Heap at GC Exit</span>**<span color="" fontsize="">:</span>

    - <span color="" fontsize="">显示了GC退出时的堆信息，包括总大小、已使用大小、区域大小、年轻代和元空间的使用情况。</span>

<span color="" fontsize="">总结来说，日志显示了JVM的启动信息、G1垃圾收集器的使用、堆和元空间的配置，以及一次由</span>`System.gc()`<span color="" fontsize="">触发的全堆压缩GC事件的详细过程和结果。这次GC有效地减少了堆的使用量，并且在短时间内完成。</span>

<span color="" fontsize="">从运行结果中可以清楚地看到GC日志中包含“4603K-\>210K”，意味着虚拟机并没 有因为这两个对象互相引用就不回收它们，这也从侧面说明虚拟机并不是通过引用计数 算法来判断对象是否存活的。</span>

##### **<span color="" fontsize="">根搜索算法</span>**

<span color="" fontsize="">在主流的商用程序语言中(Java 和C#，甚至包括前面提到的古老的Lisp)，都是使 用 根 搜 索 算 法 (GCRootsTracing ) 判 定 对 象 是 否 存 活 的 。 这 个 算 法 的 基 本 思 路 就 是 通 过一系列的名为“GCRoots” 的对象作为起始点，从这些节点开始向下搜索，搜索所 走 过 的 路 径 称 为 引 用 链 (ReferenceChain)， 当 一个 对 象 到 GCRoots 没 有 任 何 引 用 链 相 连(用图论的话来说就是从GCRoots 到这个对象不可达)时，则证明此对象是不可用 的。如图3-1所示，对象object 5、object 6、object 7虽然互相有关联，但是它们到GC Roots 是不可达的，所以它们将会被判定为是可回收的对象。 在 Java 语 言 里 ， 可 作 为 GCRoots 的 对 象 包 括 下面 几 种 :</span>

- <span color="" fontsize="">虚拟机栈 (栈帧中的本地变量表)中的引用的对象。</span>

- <span color="" fontsize="">方法区中的类静态属性引用的对象。</span>

- <span color="" fontsize="">方法区中的常量引用的对象。</span>

- <span color="" fontsize="">本地方法栈中JNI (即一般说的Native方法)的引用的对象</span>

<span color="" fontsize=""><img src="file:///Users/pt/Documents/jvm/jpeg/gc%20roots.png?lastModify=1724657621" style="display: inline-block;width:100.0%" height="447" /></span>

##### **<span color="" fontsize="">在谈引用</span>**

<span color="" fontsize="">无论是通过引用计数算法判断对象的引用数量，还是通过根搜索算法判断对象的引 用链是否可达，判定对象是否存活都与</span>**<span color="" fontsize="">“引用”</span>**<span color="" fontsize=""> 有关。在JDK1.2之前，Java中的引用 的定义很传统:</span><u><span color="" fontsize="">如果reference 类型的数据中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着 一个引用</span></u><span color="" fontsize="">。这种定义很纯粹，但是太过狭隘，一个对象在这 种定义下只有被引用或者没有被引用两种状态，对于如何描述一些“ 食之无味，弃之可 惜”的对象就显得无能为力。</span><u><span color="" fontsize="">我们希望能描述这样一类对象 :当内存空间还足够时，则 能保留在内存之中;如果内存在进行垃圾收集后还是非常紧张，则可以抛弃这些对象</span></u><span color="" fontsize="">。 很多系统的缓存功能都符合这样的应用场景。</span>

<span color="" fontsize="">在 JDK1 . 2 之 后 ， Java 对 引 用 的 概 念 进 行 了 扩 充 ， 将 引 用 分 为 强 引 用 (Strong Reference)、 软 引 用 (SoftReference )、 弱 引 用 (WeakReference )、 虚 引 用 (PhantomReference )四种，这四种引用强度依次逐渐减弱。</span>

- <span color="" fontsize="">强引用就是指在程序代码之中普遍存在的，类似“Objectobj=newObject()” 这 类的引用，只要强引用还存在，</span><u><span color="" fontsize="">垃圾收集器永远不会回收掉被引用的对象</span></u><span color="" fontsize="">。</span>

- <span color="" fontsize="">软引用用来描述一些还有用，但并非必需的对象。</span><u><span color="" fontsize="">对 于软引用关联着的对象，系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中并进行第二 次回收</span></u><span color="" fontsize="">。如果这次回收还是没有足够的内存，才会抛出内存溢出异常。在JDK 1.2 之后，提供了SoftReference类来实现软引用</span>

- <span color="" fontsize="">弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用 关 联 的 对 象 只 能 生 存 到 下 一次 垃 圾 收 集 发 生 之 前 。 </span><u><span color="" fontsize="">当 垃 圾 收 集 器 工 作 时 ， 无 论 当 前内存是否足够，都会回收掉只被弱引用关联的对象</span></u><span color="" fontsize="">。在JDK 1.2 之后，提供了W ea k R e f e r e n c e 类来 实 现 弱 引 用</span>

- <span color="" fontsize="">虚引用它是最弱的 一种引用关系。 一个对象是否 有虚引用的存在，</span><u><span color="" fontsize="">完全不会对其生存时间构成影响，也无法通过虚引用来取得 一 个对象实例</span></u><span color="" fontsize="">。为 一个对象设置虚引用关联的唯 一目的就是希望能在这个对象被收 集器回收时收到 一个系统通知。在JDK1.2之后，提供了PhantomReference类来 实现虚引用</span>

##### **<span color="" fontsize="">生存还是死亡</span>**

<span color="" fontsize="">在根搜索算法中不可达的对象，也并非是“非死不可” 的，这时候它们暂时处于 </span>**<span color="" fontsize="">“缓刑”</span>**<span color="" fontsize=""> 阶段，要</span><u><span color="" fontsize="">真正宣告 一个对象死亡，至少要经历两次标记过程:如果对象在进行 根搜索后发现没有与GCRoots相连接的引用链，那它将会被第 一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize 方法。当对象没有覆盖finalize() 方法，或者finalize(方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要 执行”</span></u><span color="" fontsize="">。如果这个对象被判定为有必要执行finalize ()方法，那么这个对象将会被放置在 一个 名为F -Queue 的队列之中，并在稍后由一条由虚拟机自动建立的、低优先级的Finaliz er 线程去执行。这里所谓的“执行” 是指虚拟机会触发这个方法，但并不承诺会等待它运 行结束。这样做的原因是，如果 一个对象在finalize(方法中执行缓慢，或者发生了死 循 环 (更 极 端 的 情 况 )， 将 很 可 能 会 导 致 F - Q u e u e 队 列 中 的 其 他 对 象 永 久 处 于等 待 状 态 ， 甚至导致整个内存回收系统崩溃。finalize (方法是对象逃脱死亡命运的最后一次机会， 稍后 G C将对F- Qu eue中的对象进行第二次小规模的标记，</span><u><span color="" fontsize="">如果对象要在finalize(中成</span></u><span color="" fontsize=""> </span><u><span color="" fontsize="">功拯救自己 ，只要重新与引用链 上的任何 一个对象建立关联即可，</span><span color="rgb(167, 167, 167)" fontsize="" style="color: rgb(167, 167, 167)">\<u\></span><span color="" fontsize="">譬如把自己 (this 关键字)赋值给某个类变量或对象的成员变量，那在第二次标记时它将被移除出“ 即将回收” 的集合</span></u><span color="" fontsize="">;如果对象这时候还没有逃脱，那它就真的离死不远了。从下面代码中 中我们可以看到 一个对象的finalize()被执行，但是它仍然可以存活。</span>

    public class FinalizeEscapeGC {
        public static FinalizeEscapeGC SAVE_HOOK = null;
    ​
        public void isAlive() {
            System.out.println("yes, i am still alive :) ");
        }
    /*
        finalize()方法通常用于在对象被垃圾收集前进行清理工作
         */
        @Override
        protected void finalize() throws Throwable {
            super.finalize();
            System.out.println("finalize mehtod executed!");
            FinalizeEscapeGC.SAVE_HOOK = this;
        }
    ​
        public static void main(String[] args) throws InterruptedException {
            SAVE_HOOK=new FinalizeEscapeGC();
            // 对象第一次拯救自己
            SAVE_HOOK=null;
            System.gc();
            // 因为Finalize方法优先级很低，暂停0.5秒，以等待它
            Thread.sleep(500);
            if(SAVE_HOOK!=null){
                SAVE_HOOK.isAlive();
            }else {
                System.out.println("no,i am dead");
            }
            SAVE_HOOK=null;
            System.gc();
            // 因为Finalize方法优先级很低，暂停0.5秒，以等待它
            Thread.sleep(500);
            if(SAVE_HOOK!=null){
                SAVE_HOOK.isAlive();
            }else {
                System.out.println("no,i am dead");
            }
        }
    }
    运行结果
    finalize mehtod executed!
    yes, i am still alive :) 
    no,i am dead

<span color="" fontsize="">代码运行结果可以看到，SAVE_HOOK对象的finalizeO方法确实被 G C 收 集 器 触 发 过 ， 并 且 在 被 收 集 前 成 功 逃 脱 了。 另外 一个值得注意的地方就是，代码中有两段完全 一样的代码片段，</span>**<span color="" fontsize="">执行结果却是 一次逃脱成功， 一次失败，这是因为任何 一个对象的finalize(方法都只会被系统自动调 用 一次，如果对象面临下一次回收，它的finalize(方法不会被再次执行，因此第 二段代 码的自救行动失败 了</span>**<span color="" fontsize="">。 需要特别说明的是，上面关于对象死亡时finalize()方法的描述可能带有悲情的艺术 色彩，笔者并不鼓励大家使用这种方法来拯救对象。相反，笔者建议大家尽量避免使用 它 ， 因 为 它 不 是 C / C + + 中 的 析 构 函 数 ， 而 是 J ava 刚 诞 生 时 为 了使 C / C + + 程 序 员 更 容 易 接受它所做出的 一个妥协。它的运行代价高昂，不确定性大，无法保证各个对象的调用 顺序。有些教材中提到它适合做“关闭外部资源” 之类的工作，这完全是对这种方法的 用途的 一种自我安慰。finalize()能做的所有工作，使用try-finally或其他方式都可以做 得更好、更及时，大家完全可以忘掉Java 语言中还有这个方法的存在。</span>

##### **<span color="" fontsize="">回收方法区</span>**

<span color="" fontsize="">很多人认为方法区 (或者HotSpot虚拟机中的永久代)是没有垃圾收集的，Java虚 拟机规范中确实说过可以不要求虚拟机在方法区实现垃圾收集，而且在方法区进行垃圾 收集的 “性价比” 一般比较低:在堆中，尤其是在新生代中，常规应用进行一次垃圾收集一般可以回收70%~ 95%的空间，而永久代的垃圾收集效率远低于此。 </span>**<span color="" fontsize="">永久代的垃圾收集主要回收两部分内容:废弃常量和无用的类</span>**<span color="" fontsize="">。回收废弃常量 与 回 收Java 堆 中 的 对 象 非 常 类 似 。 以 常 量 池 中 字面 量 的 回 收 为 例 ， 假 如 一 个 字 符 串 “abc” 已经进入了常量池中，但是当前系统没有任何一个String对象是叫做“abc” 的，换句话说是没有任何String 对象引用常量池中的“abe” 常量，也没有其他地方 引用了这个字面量，如果在这时候发生内存回收，而且必要的话，这个“abc” 常量 就会被系统“请” 出常量池。常量池中的其他类(接口)、方法、字段的符号引用也 与此类似。 判定一个常量是否是“废弃常量” 比较简单，而要判定一个类是否是“ 无用的类” 的条件则相对苛刻许多。类需要同时满足下面3 个条件才能算是“ 无用的类” :</span>

- <span color="" fontsize="">该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。</span>

- <span color="" fontsize="">加 载 该 类的 C l a s s L o a d e r 已 经 被 回收 。</span>

- <span color="" fontsize="">该类对应的java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。</span>

- <span color="" fontsize="">虚拟机可以对满足上述3 个条件的无用类进行回收，这里说的仅仅是“ 可以”，而 不是和对象 一样，不使用了就必然会回收。是否对类进行回收，HotSpot 虚拟机提供 了 - X noclassge 参 数 进 行 控 制 ， 还 可 以 使 用 -verbose : class 及 - X X :+ TraceClassL oading 、 - X X :+TraceClassUnLoading 查 看 类 的 加 载 和 卸 载 信 息 。 -verbore : class 和 - X X : +TraceClassLoading 可以在Product 版的虚拟机中使用，但是-XX:+TraceClassLoa ding 参数需要f ast dcbug 版的虚拟机支持。</span> <span color="" fontsize="">在大量使用反射、动态代理、CGLi b 等bytecode 框架的场景，以及动态生成JSP 和 OSGi </span><u><span color="" fontsize="">这类频繁自定义ClassLoader 的场景都需要虚拟机具备类卸载的功能，以保证永久 代不会溢出。</span></u>

#### **<span color="" fontsize="">垃圾收集算法</span>**

##### **<span color="" fontsize="">标记-清除算法</span>**

<span color="" fontsize="">最 基 础 的 收 集 算 法 是 “ 标 记 一清 除 ” (Mark - Sweep ) 算 法 ， 如 它 的 名 字 一 样 ， 算 法 分 为“标记” 和“清除” 两个阶段:首先标记出所有需要回收的对象，在标记完成后统一回 收掉所有被标记的对象，它的标记过程其实在前 一节讲述对象标记判定时已经基本介绍过 了。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺 点进行改进而得到的。它的主要缺点有两个:一个是效率问题，标记和清除过程的效率都 不高:另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多 可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而 不 得 不 提 前 触 发 另 一次 垃 圾 收 集 动 作 。 标 记 一 清 除 算 法 的 执 行 过 程 如 下图所示</span>

<span color="" fontsize=""><img src="file:///Users/pt/Documents/jvm/jpeg/%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4.png?lastModify=1724657621" style="display: inline-block;width:100.0%" height="390" /></span>

##### **<span color="" fontsize="">标记-整理算法</span>**

<span color="" fontsize="">“标记一整理”(Mark-Compact)算法， 标记过程仍然与“标记一清除” 算法一样，但后续步骤不是直接对可回收对象进行清 理，</span><u><span color="" fontsize="">而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存</span></u>

##### **<span color="" fontsize="">分代收集算法</span>**

<span color="" fontsize="">当前商业虚拟机的垃圾收集都采用 “分代收集”(GenerationalCollection)算法，这 种算法并没有什么新的思想，只是根据对象的存活周期的不同将内存划分为几块。 </span><u><span color="" fontsize="">一般是把Java 堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算 法</span></u><span color="" fontsize="">。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复 制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存 活率高 、没有额外空间对它进行分配担保，就必须使用“标记 一清理” 或“标记一整 理” 算法来进行回收</span>

#### **<span color="" fontsize="">垃圾收集器</span>**

##### **<span color="" fontsize="">Serial收集器</span>**

<span color="" fontsize="">该收集器是 一个单线程的收集器，但它 的“单线程” 的意义并不仅仅是说明它只会使用一个CPU或一条收集线程去完成垃圾收 集 工 作 ， </span><u><span color="" fontsize="">更 重 要 的 是 在 它 进 行 垃 圾 收 集 时 ， 必 须 暂 停 其 他 所 有 的 工 作 线 程</span></u><span color="" fontsize=""> (S u n 将 这 件 事 情 称 之 为 “ S t o p T h e W o r l d ” )， 直 到 它 收 集 结 束</span>

##### **<span color="" fontsize="">ParNew收集器</span>**

<span color="" fontsize="">P a r N e w 收 集 器 其 实 </span><u><span color="" fontsize="">就 是 S e r i a l 收 集 器 的 多 线 程 版 本 ， 除 了使 用 多 条 线 程 进 行 垃 圾 收集之外，</span></u><span color="" fontsize="">其余行为包括Serial 收集器可用的所有控制参数 (例如:-XX:SurvivorRatio、 - X X : P r e t e n u r e S i z e T h r e s h o l d 、 - X X : H a n d l e P r o m o t i o n F a i l u r e 等 )、 收 集 算 法 、 S t o p T h e World、对象分配规则、回收策略等都与Serial 收集器完全 一样，实现上这两种收集器也 共 用 了相 当 多的 代 码 。 P a r N e w 收 集 器 的 工作 过 程 如 下图 所 示</span>

<span color="" fontsize="">ParNew收集器除 了多线程收集之外，其他与Serial收集器相比并没有太多创新之 处，但它却是许多运行在Server 模式下的虚拟机中首选的新生代收集器，其中有一个与 性能无关但很重要的原因是，除 了Serial 收集器外，目前只有它能与CMS收集器配合 工作</span>

##### **<span color="" fontsize="">Parallel Scavenge收集器</span>**

<span color="" fontsize="">Parallel Scavenge</span><u><span color="" fontsize="">收集器也是 一个新生代收集器，它也是使用</span></u>**<u><span color="" fontsize="">复制算法</span></u>**<u><span color="" fontsize="">的收集器</span></u><span color="" fontsize="">， 又是并行的多线程收集器......看上去和ParNew 都一样，那它有什么特别之处呢? Parallel Scavenge 收集器的特点是它的关注点与其他收集器不同，CMS 等收集器的 关注点尽可能地缩短垃圾收集时用户线程的停顿时间，</span><u><span color="" fontsize="">而Parallel Scavenge收集器的目 标则是达到 一个可控制的吞吐量(Throughput)</span></u><span color="" fontsize="">。所谓吞吐量就是CPU用于运行用户代 码的时间与CPU总消耗时间的比值，即吞吐量= 运行用户代码时间/ (运行用户代码时 间+垃圾收集时间)，虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐 量就是99%。 停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户的体 验;而高吞吐量则可以最高效率地利用CPU时间，尽快地完成程序的运算任务，主要适 合在后台运算而不需要太多交互的任务。</span><u><span color="" fontsize="">Parallel Scavenge 收集器提供 了两 个参数用 于精确控制吞吐量，分别是控制 最 大 垃 圾 收 集 停 顿 时 间 的 - X X: M a x G C P a u s e M i l l i s 参 数 及 直 接 设 置 吞 吐 量 大 小 的-XX:GCTimeRatio 参数</span></u><span color="" fontsize="">。Max GCPauseMillis 参数允许的值是 一个大于0的毫秒数，收集器将尽力保证内存回 收花费的时间不超过设定值。不过大家不要异想天开地认为如果把这个参数的值设置得 稍小 一点就能使得系统的垃圾收集速度变得更快，GC停顿时间缩短是以牺牲吞吐量和 新生代空间来换取的:系统把新生代调小 一些，收集300MB新生代肯定比收集500MB 快 吧 ， 这 也 直 接 导 致 垃 圾 收 集 发 生 得 更 频 繁 一些 ， 原 来 1 0 秒 收 集 一次 、 每 次 停 顿 1 0 0 毫秒，现在变成5秒收集一次、每次停顿70毫秒 。停顿时间的确在下降，但吞吐量也 降 下来 了。 GCTimeRatio 参数的值应当是一个大于0 小 于10 0的整数，也就是垃圾收集时间占 总 时 间 的 比 率 ， 相 当 于是 吞 吐 量 的 倒 数 。 如 果 把 此 参 数 设 置 为 1 9 ， 那 允 许 的 最 大 G C 时 间就占总时间的5% (即1/ (1+19)，默认值为99，就是允许最大1% (即1/ (1+99) 的垃圾收集时间。由于与吞吐量关系密切，Parallel Scavenge收集器也经常被称为“吞吐量 优先” 收集器。除上述两个参数之外，Parallel Scavenge收集器还有 一个参数XX:+UseAdaptiveSizePolicy 值得关注。这是一个开关参数，当这个参数打开之后，就 不 需 要 手 工 指 定 新 生 代 的 大 小 ( - X m n )、 E d e n 与 S u r v i v o r 区 的 比 例 ( - X X : S u r v i v o r R a t i o )、 晋 升 老 年 代 对 象 年 龄 (- X X : P r e t e n u r e S i z e T h r e s h o l d ) 等 细 节 参 数 了 ， 虚 拟 机 会 根 据 当 前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或 最 大的 吞 吐 量 ， 这 种 调 节 方 式 称 为 G C 自 适 应 的 调 节 策 略 (G C E r g o n o m i c s )®。 如 果 读者对于收集器运作原理不太了解，手工优化存在困难的时候，使用Parallel Scavenge 收集器配合自适应调节策略，把内存管理的调优任务交给虚拟机去完成将是 一个很 不错的选择。只需要把基本的内存数据设置好(如-Xmx设置最大堆)，然后使用 MaxGCPauseMillis 参数 (更关注最大停顿时间)或GCTimeRatio参数 (更关注吞吐量) 给虚拟机设立 一个优化目标，那具体细节参数的调节工作就由虚拟机完成了。</span><u><span color="" fontsize="">自适应调节策略也是ParallelScavenge收集器与ParNew收集器的 一个重要区别</span></u>

##### **<span color="" fontsize="">Parallel Old收集器</span>**

<span color="" fontsize="">Parallel Old 是Parallel Scavenge收集器的老年代版本，使用多线程和“标记一整理” 算法。这个收集器是在JDK 1.6 中才开始提供的，在此之前，新生代的Paral lel Scavenge 收集器一直处于比较尴尬的状态。原因是，如果新生代选择了Parallel Scavenge收集器， 老 年 代 除 了 S e r i a l O l d (P S M a r k S w e e p ) 收 集 器 外 别 无 选 择 (还 记 得 上 面 说 过 P a r a l l e l Scavenge收集器无法与CMS收集器配合工作吗?)。由于单线程的老年代Serial Old收 集 器 在 服 务 端 应 用 性 能 上的 “ 拖 累 ” ， 即 便 使 用 了 P a r a l l e l S c a v e n g e 收 集 器 也 未 必 能 在 整 体 应 用 上获 得 吞 吐 量 最 大 化 的 效 果 ， 又 因 为 老 年 代 收 集 中 无 法 充 分 利 用 服 务 器 多 C P U 的 处理能力，在老年代很大而且硬件比较高级的环境中，这种组合的吞吐量甚至还不 一定 有ParNew加CMS的组合“给力”。 直 到 P a r a l l e l O l d 收 集 器 出 现 后 ， “ 吞 吐 量 优 先 ” 收 集 器 终 于 有 了比 较 名 副 其 实 的 应 用组合，在注重吞吐量及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge 加 Parallel Old收集器。</span>

##### **<span color="" fontsize="">CMS收集器</span>**

<span color="" fontsize="">CMS (Con current Mark Sweep )收集器是一种以获取最短回收停顿时间 为目标的收 集器。目前很大一部分的Java 应用都集中在互联网站或B/S 系统的服务端 上，这类应用 尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS收 集器就非常符合这类应用的需求。</span>

<span color="" fontsize="">从 名 字 (包 含 “ M a r k S w e e p ” ) 上 就 可 以 看 出 C M S 收 集 器 是 基 于 “ 标 记 一 清 除 ” 算</span>

<span color="" fontsize="">法实现的，它的运作过程相对于前面几种收集器来说要更复杂 一些，整个过程分为4个 步骤，包括:</span>

- <span color="" fontsize="">初始 标记(CMSinitialmark)</span>

- <span color="" fontsize="">并 发 标 记 (C M S c o n c u r r e n t m a r k )</span>

- <span color="" fontsize="">重 新 标 记 (C M S r e m a r k )</span>

- <span color="" fontsize="">并 发 清 除 (C M S c o n c u r r e n t s w e ep )</span> <u><span color="" fontsize="">其中初始标记、重新标记这两个步骤仍然需要“St op The World”</span></u><span color="" fontsize="">。初始标记仅仅 只是标记一下GCRoots 能直接关联到的对象，速度很快，并发标记阶段就是进行GC Roots Tracing的过程，而重新标记阶段则是为了修正并发标记期间，因用户程序继续运 作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间 一般会比初始 标记阶段稍长 一些，但远比并发标记的时间短。 由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户 线程 一起工作，所以总体上来说，</span><u><span color="" fontsize="">CMS收集器的内存回收过程是与用户线程 一起并发地 执行的</span></u><span color="" fontsize="">。 CMS是 一款优秀的收集器，它的最主要优点在名字上已经体现出来了:</span><u><span color="" fontsize="">并发收集、 低停顿，Sun的 一些官方文档里面也称之为并发低停顿收集器</span></u><span color="" fontsize="">(ConcurrentLowPause C o l l e c t o r )。 但 是 C M S 还 远 达 不 到 完 美 的 程 度 ， 它 有 以 下 三 个 显 著 的 缺 点 :</span>

  - <span color="" fontsize="">C M S 收 集 器 对 C P U 资 源 非 常 敏 感 。 其实 ， 面 向 并 发 设 计 的 程 序 都 对 C P U 资 源 比 较敏感。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用了一部分 线 程 (或 者 说 C P U 资 源 ) 而 导 致 应 用 程 序 变 慢 ， 总 吞 吐 量 会 降 低 。</span>

  - <span color="" fontsize="">CMS 收集器无法处理浮动垃圾 (Floating Garbage )，可能出现 “ Concurrent Mode Failure” 失败而导致另 一次Full GC的产生。由于CMS 并发清理阶段用户线程还 在运行着，伴随程序的运行自然还会有新的垃圾不断产生，这 一部分垃圾出现在 标记过程之后 ，CMS 无法在本次收集中处理掉它们，只好留待下 一次 GC时再将 其清理掉。这一部分垃圾就称为“ 浮动垃圾”。也是由于在垃圾收集阶段用户线 程还需要运行，即还需要预留足够的内存空间给用户线程使用，因此CMS收集 器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留 一 部分空间提供并发收集时的程序运作使用。在默认设置下，CMS 收集器在老年代 使用了68%的空间后就会被激活，这是 一个偏保守的设置，如果在应用中老年代 增长不是太快，可以适当调高参数 -XX:CMSInitiatingOccupancyFraction 的值来 提高触发百分比，以便降低内存回收次数以获取更好的性能。要是CMS 运行期 间预留的内存无法满足程序需要，就会出现 一次 “Concurrent ModeFailure” 失 败，这时候虚拟机将启动后备预案:临时启用Serial Ol d收集器来重新进行老年 代的垃圾收集，这样停顿时间就很长了。所以说参数-XX:CMSInitiating Occupan cyFraction 设置得太高将会很容易导致大量“ Concurrent Mode Failure” 失败，性 能反而降低。</span>

  - <span color="" fontsize="">还 有 最 后 一个 缺 点 ， 在 本 节 在 开 头 说 过 ， C M S 是 一款 基 于 “ 标 记 一 清 除 ” 算 法 实 现的收集器，如果读者对前面这种算法介绍还有印象的话，就可能想到这意味着 收集结束时会产生大量空间碎片。空间碎片过多时，将会给大对象分配带来很大 的麻烦，往往会出现老年代还有很大的空间剩余，但是无法找到足够大的连续空 间 来 分 配 当 前 对 象 ， 不 得 不 提 前 触 发 一次 F u l l G C 。 为 了 解 决 这 个 问 题 ， C M S 收 集 器 提 供 了 一 个 - X X :+ U s e C M S C o m p a c t A t F u l l C o l l e c t i o n 开 关 参 数 ， 用 于 在 “ 享 64 第 二部分自动内存管理机制 受” 完Full GC服务之后额外免费附送一个碎片整理过程，内存整理的过程是无 法并发的。空间碎片问题没有了，但停顿时间不得不变长了。虚拟机设计者们还 提供了另外 一个参数XX:CMSFulIGCsBeforeCompaction，这个参数用于设置在执 行 多 少 次 不 压 缩 的 F u l l G C 后 ， 跟 着 来 一次 带 压 缩 的</span>

  ##### **<span color="" fontsize="">G1收集器</span>**

  <span color="" fontsize="">简单介绍”。 G1收集器是垃圾收集器理论进一步发展的产物，它与前面的CMS收集器相比有两 个显著的改进:一是G1收集器是基于“标记一整理” 算法实现的收集器，也就是说它 不会产生空间碎片，这对于长时间运行的应用系统来说非常重要。 二是它可以非常精确 地控制停顿，既能让使用者明确指定在 一个长度为M毫秒的时间片段内，消耗在垃圾收 集上的时间不得超过N 毫秒，这几乎已经是实时Java (RTSJ)的垃圾收集器的特征了。 G1收集器可以实现在基本不牺性吞吐量的前提下完成低停顿的内存回收，这是由于 它能够极力地避免全区域的垃圾收集，之前的收集器进行收集的范围都是整个新生代或 老年代，而</span><u><span color="" fontsize="">GI 将整个Java堆 (包括新生代、老年代)划分为多个大小固定的独立区域</span></u><span color="" fontsize=""> </span><u><span color="" fontsize="">(R e g i o n )， 并 且 跟 踪 这 些 区 域 里 面 的 垃 圾 堆 积 程 度 ， 在 后 台 维 护 一 个 优 先 列 表 ， 每 次 根 据允许的收集时间，优先回收垃圾最多的区域(这就是GarbageFirst 名称的来由)</span></u><span color="" fontsize="">。区 域划分及有优先级的区域回收，保证了G1收集器在有限的时间内可以获得最高的收集 效率</span>
