---
title: 三种GC场景模拟与分析.md
date: 2025-06-15 21:19:13
tags: jvm
---

## 堆内存的分布

#### 1. 经典分代模型 (适用于 ParNew, Parallel, CMS 等收集器)

在传统的垃圾收集中，堆内存被物理上划分为连续的两大块：

- **新生代 (Young Generation / New Generation)**
  - **作用**: 存放新创建的、生命周期通常很短的对象。新生代是 **Minor GC / Young GC** 发生的主要场所。
  - 内部结构
    - **伊甸园区 (Eden Space)**: 绝大多数对象诞生的地方。
    - **幸存者区 (Survivor Space)**: 有两个，通常称为 `S0` 和 `S1`。它们大小相等，采用“复制算法”进行垃圾回收，在任何时刻总有一个是空的。用于存放每次 Minor GC 后仍然存活的对象。
- **老年代 (Old Generation / Tenured Generation)**
  - **作用**: 存放生命周期长的对象。这些对象通常是在新生代的 Survivor 区中“熬过”了多次 Minor GC 后晋升上来的，或者是“大对象”。
  - 老年代的 GC 通常被称为 **Major GC** 或 **Full GC**，其回收频率远低于 Minor GC，但耗时更长。
- **方法区 (Method Area) / 元空间 (Metaspace)**
  - 这是一个重要的补充：在 JDK 8 之前，有一个叫做 **永久代 (Permanent Generation)** 的区域，它在逻辑上属于方法区，但在物理上是**堆的一部分**。
  - 从 JDK 8 开始，永久代被彻底移除，取而代之的是**元空间 (Metaspace)**。元空间使用的是**本地内存（Native Memory）**，而**不是堆内存**。
  - **作用**: 存放类的元数据信息（如类名、字段、方法）、常量池、即时编译器（JIT）编译后的代码等。

#### 2. G1 垃圾收集器的堆分布模型

G1（Garbage-First）虽然在逻辑上仍然遵循分代的概念，但在**物理上彻底改变了内存布局**。

- **不再是连续空间**: G1 将整个堆划分为大量大小相等的、不连续的**区域 (Region)**。每个 Region 的大小通常是 1MB 到 32MB 之间的 2 的幂。
- **动态的角色**: 每个 Region 在同一时间只扮演一种角色，但这个角色是**动态变化**的。一个 Region 在此刻可能是 Eden，在被回收后可能就变成了 Free（空闲），之后又可能被用于存放老年代对象。
- Region 的角色类型
  - **Eden Region**: 逻辑上构成新生代的 Eden 区。
  - **Survivor Region**: 逻辑上构成新生代的 Survivor 区。
  - **Old Region**: 逻辑上构成老年代。
  - **Humongous Region (大对象区)**: 专门用于存放“大对象”（大小超过 Region 容量 50% 的对象）。一个大对象可能会跨越多个连续的 Humongous Region。逻辑上它属于老年代。
  - **Free Region**: 未被使用的空闲区域。

这种基于 Region 的设计使得 G1 可以根据需要灵活地调整新生代和老年代的大小，并且在回收时可以选择**收益最高的若干个 Region**进行回收（这也是“Garbage-First”名字的由来），从而更好地满足设定的停顿时间目标

## 各种 GC 的触发条件

| GC 类型 (GC Type) | 回收区域 (Collection Area) | 核心触发条件 (Core Triggering Conditions) | 特点与影响 (Characteristics & Impact) |
| :--- | :--- | :--- | :--- |
| **Minor GC / Young GC** | 仅新生代 (Eden + From Survivor) | 1. **Eden 区空间不足**：当应用程序需要分配一个新对象，而 Eden 区没有足够的连续空间时，就会触发。这是**最常见**的触发条件。 | **高频、快速、STW短**。只回收新生代，存活对象被复制到 Survivor 区或晋升到老年代。这是常规且健康的 GC 模式。 |
| **G1 特有：因大对象分配的 Young GC** | 仅新生代 | 1. **分配大对象 (Humongous Object)**：当需要分配一个大对象时，G1 会检查是否有足够的连续空闲 Region。如果没有，它会**优先尝试触发一次 Young GC**，期望通过清理新生代来释放出足够的连续空间。 | **STW 同样很短**。这是一种为满足大对象分配的**预处理**操作。日志标识为 `(G1 Humongous Allocation)`。 |
| **G1 特有：Mixed GC (混合回收)** | 整个新生代 + 部分老年代 Region | 1. **老年代占用率达到阈值**：当老年代的占用率超过 `InitiatingHeapOccupancyPercent` (IHOP，默认 45%) 阈值时，G1 会启动一个**并发标记周期**（与应用程序并行，基本无 STW）。<br>2. **并发标记完成**：在标记周期成功完成后，G1 就知道了哪些老年代 Region 的“垃圾”最多。它会**在接下来的几次 Young GC 中，顺带回收一部分“垃圾最多”的老年代 Region**。这种“Young GC + 部分 Old GC”的组合就叫 Mixed GC。 | **STW 可控**。这是 G1 的核心优势，它避免了一次性回收整个老年代带来的长停顿，通过多次、小规模地清理老年代来控制停顿时间。 |
| **Full GC (全局回收)** | 整个堆 (新生代 + 老年代) + 元空间 | 1. **`System.gc()` 调用**：代码中显式调用该方法，强烈建议 JVM 执行一次 Full GC。<br>2. **晋升失败 (Promotion Failure)**：在 Young GC 之后，存活的对象需要晋升到老年代，但老年代没有足够的连续空间来容纳它们。<br>3. **元空间不足 (Metaspace is full)**：如果加载的类过多，导致元空间耗尽，会触发 Full GC。<br>4. **并发模式失败 (Concurrent Mode Failure)**：在 CMS 或 G1 中，如果并发标记/回收的速度跟不上应用程序创建对象的速度，导致老年代被填满，JVM 会放弃并发，退化（Fallback）为一次有长时间 STW 的 Full GC。<br>5. **G1 大对象分配失败**：当分配一个大对象时，即使在触发了一次 Young GC 后，**依然没有**足够的连续空间来容纳它，可能会退化成 Full GC。 | **低频、极慢、STW长**。这是性能的“杀手”，会暂停所有应用线程，直到回收完成。在调优中，**避免或减少 Full GC 的发生**是首要目标之一。 |

## 测试代码

```java
package org.pt.gc;

/**
 * @ClassName GcBehaviorDemo
 * @Author pt
 * @Description
 * @Date 2025/6/14 21:36
 **/
import java.util.ArrayList;
import java.util.List;

public class GcBehaviorDemo {

    // 定义每个对象的大小 (1 MB)
    private static final int OBJECT_SIZE = 400 * 1024;

    // 用于持有对象的引用，防止被立即回收
    private static final List<byte[]> objectHolder = new ArrayList<>();

    public static void main(String[] args) throws InterruptedException {
        printMemoryUsage("程序启动");
        // --- 阶段1 在Eden区快速分配小对象，触发Minor GC ---
        System.out.println("\n===== 阶段1: 快速分配小对象，填满Eden区，触发Minor GC =====");
        for (int i = 0; i < 15; i++) {
            allocateAndPrint(i);
            Thread.sleep(2000); // 短暂休眠，让GC日志更容易观察
        }
        printMemoryUsage("阶段1结束");

        // --- 行为2: 让部分对象存活下来，观察其晋升到老年代 ---
        // 在阶段1中，部分对象（被objectHolder持有）已经在多次Minor GC中存活下来，
        // 应该已经被晋升到老年代了。我们再分配一些，确保老年代有数据。
        System.out.println("\n===== 阶段2: 持续分配，观察对象晋升至老年代 =====");
        for (int i = 0; i < 10; i++) {
            allocateAndPrint(15 + i);
            Thread.sleep(200);
        }
        printMemoryUsage("阶段2结束");
        // --- 行为3: 分配一个大对象，可能直接进入老年代 ---
        System.out.println("\n===== 阶段3: 分配一个大对象 =====");
        System.out.println("准备分配一个 8 MB 的大对象...");
        // G1中，超过Region大小一半的对象被视为大对象（Humongous Object）
        // 会被直接分配到专门的Humongous Region，逻辑上属于老年代
        byte[] largeObject = new byte[8 * 1024 * 1024];
        System.out.println("大对象分配完毕。");
        printMemoryUsage("阶段3结束");

        // --- 行为4: 手动触发Full GC ---
        System.out.println("\n===== 阶段4: 手动触发 Full GC =====");
        System.out.println("清空所有引用...");
        objectHolder.clear();
        largeObject = null; // 清除大对象引用
        System.out.println("调用 System.gc() 建议执行 Full GC...");
        System.gc();
        Thread.sleep(2000); // 等待GC完成
        printMemoryUsage("Full GC后");

        System.out.println("\n程序结束。请分析控制台输出的GC日志。");
    }

    /**
     * 分配一个1MB的对象，并将其加入到持有列表中
     */
    private static void allocateAndPrint(int count) {
        System.out.printf("分配第 %d 个对象 (1 MB)...%n", count + 1);
        objectHolder.add(new byte[OBJECT_SIZE]);
    }

    /**
     * 打印当前堆内存的使用情况
     */
    private static void printMemoryUsage(String stage) {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        System.out.printf("[%s] - 堆内存: 已用 %.2f MB, 空闲 %.2f MB, 总共 %.2f MB%n",
                stage,
                bytesToMb(usedMemory),
                bytesToMb(freeMemory),
                bytesToMb(totalMemory));
    }

    private static double bytesToMb(long bytes) {
        return (double) bytes / 1024 / 1024;
    }
}
```

## 运行命令

```java
java -Xms60m -Xmx60m -XX:+UseG1GC -XX:+UnlockExperimentalVMOptions -XX:MaxGCPauseMillis=200 -XX:G1NewSizePercent=40 -XX:G1MaxNewSizePercent=40 -XX:MaxTenuringThreshold=2 '-Xlog:gc*:file=gc.log' org.pt.gc.GcBehaviorDemo

```

## 日志输出

```xml
[0.005s][info][gc] Using G1
[0.005s][info][gc,init] Version: 17.0.11+7-LTS-207 (release)
[0.005s][info][gc,init] CPUs: 8 total, 8 available
[0.005s][info][gc,init] Memory: 8192M
[0.005s][info][gc,init] Large Page Support: Disabled
[0.005s][info][gc,init] NUMA Support: Disabled
[0.005s][info][gc,init] Compressed Oops: Enabled (Zero based)
[0.005s][info][gc,init] Heap Region Size: 1M
[0.005s][info][gc,init] Heap Min Capacity: 64M
[0.005s][info][gc,init] Heap Initial Capacity: 64M
[0.005s][info][gc,init] Heap Max Capacity: 64M
[0.005s][info][gc,init] Pre-touch: Disabled
[0.005s][info][gc,init] Parallel Workers: 8
[0.005s][info][gc,init] Concurrent Workers: 2
[0.005s][info][gc,init] Concurrent Refinement Workers: 8
[0.005s][info][gc,init] Periodic GC: Disabled
[0.009s][info][gc,metaspace] CDS archive(s) mapped at: [0x000000a800000000-0x000000a800be0000-0x000000a800be0000), size 12451840, SharedBaseAddress: 0x000000a800000000, ArchiveRelocationMode: 1.
[0.009s][info][gc,metaspace] Compressed class space mapped at: 0x000000a801000000-0x000000a841000000, reserved size: 1073741824
[0.009s][info][gc,metaspace] Narrow klass base: 0x000000a800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
[2.698s][info][gc,start    ] GC(0) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[2.698s][info][gc,task     ] GC(0) Using 2 workers of 8 for evacuation
[2.700s][info][gc,phases   ] GC(0)   Pre Evacuate Collection Set: 0.1ms
[2.700s][info][gc,phases   ] GC(0)   Merge Heap Roots: 0.0ms
[2.700s][info][gc,phases   ] GC(0)   Evacuate Collection Set: 1.3ms
[2.700s][info][gc,phases   ] GC(0)   Post Evacuate Collection Set: 0.2ms
[2.700s][info][gc,phases   ] GC(0)   Other: 0.5ms
[2.700s][info][gc,heap     ] GC(0) Eden regions: 2->0(24)
[2.700s][info][gc,heap     ] GC(0) Survivor regions: 0->1(4)
[2.700s][info][gc,heap     ] GC(0) Old regions: 0->0
[2.700s][info][gc,heap     ] GC(0) Archive regions: 2->2
[2.700s][info][gc,heap     ] GC(0) Humongous regions: 26->26
[2.700s][info][gc,metaspace] GC(0) Metaspace: 323K(512K)->323K(512K) NonClass: 299K(384K)->299K(384K) Class: 24K(128K)->24K(128K)
[2.700s][info][gc          ] GC(0) Pause Young (Concurrent Start) (G1 Humongous Allocation) 28M->27M(64M) 2.214ms
[2.700s][info][gc,cpu      ] GC(0) User=0.00s Sys=0.00s Real=0.01s
[2.700s][info][gc          ] GC(1) Concurrent Undo Cycle
[2.700s][info][gc,marking  ] GC(1) Concurrent Cleanup for Next Mark
[2.700s][info][gc,marking  ] GC(1) Concurrent Cleanup for Next Mark 0.435ms
[2.700s][info][gc          ] GC(1) Concurrent Undo Cycle 0.492ms
[2.901s][info][gc,start    ] GC(2) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[2.902s][info][gc,task     ] GC(2) Using 2 workers of 8 for evacuation
[2.903s][info][gc,phases   ] GC(2)   Pre Evacuate Collection Set: 0.1ms
[2.903s][info][gc,phases   ] GC(2)   Merge Heap Roots: 0.0ms
[2.903s][info][gc,phases   ] GC(2)   Evacuate Collection Set: 1.2ms
[2.903s][info][gc,phases   ] GC(2)   Post Evacuate Collection Set: 0.2ms
[2.903s][info][gc,phases   ] GC(2)   Other: 0.4ms
[2.903s][info][gc,heap     ] GC(2) Eden regions: 1->0(24)
[2.903s][info][gc,heap     ] GC(2) Survivor regions: 1->1(4)
[2.903s][info][gc,heap     ] GC(2) Old regions: 0->0
[2.903s][info][gc,heap     ] GC(2) Archive regions: 2->2
[2.903s][info][gc,heap     ] GC(2) Humongous regions: 28->28
[2.904s][info][gc,metaspace] GC(2) Metaspace: 324K(512K)->324K(512K) NonClass: 300K(384K)->300K(384K) Class: 24K(128K)->24K(128K)
[2.904s][info][gc          ] GC(2) Pause Young (Concurrent Start) (G1 Humongous Allocation) 29M->29M(64M) 2.060ms
[2.904s][info][gc,cpu      ] GC(2) User=0.00s Sys=0.00s Real=0.00s
[2.904s][info][gc          ] GC(3) Concurrent Mark Cycle
[2.904s][info][gc,marking  ] GC(3) Concurrent Clear Claimed Marks
[2.904s][info][gc,marking  ] GC(3) Concurrent Clear Claimed Marks 0.018ms
[2.904s][info][gc,marking  ] GC(3) Concurrent Scan Root Regions
[2.905s][info][gc,marking  ] GC(3) Concurrent Scan Root Regions 0.959ms
[2.905s][info][gc,marking  ] GC(3) Concurrent Mark
[2.905s][info][gc,marking  ] GC(3) Concurrent Mark From Roots
[2.905s][info][gc,task     ] GC(3) Using 2 workers of 2 for marking
[2.907s][info][gc,marking  ] GC(3) Concurrent Mark From Roots 2.221ms
[2.907s][info][gc,marking  ] GC(3) Concurrent Preclean
[2.907s][info][gc,marking  ] GC(3) Concurrent Preclean 0.039ms
[2.907s][info][gc,start    ] GC(3) Pause Remark
[2.907s][info][gc          ] GC(3) Pause Remark 31M->31M(64M) 0.264ms
[2.907s][info][gc,cpu      ] GC(3) User=0.00s Sys=0.00s Real=0.00s
[2.907s][info][gc,marking  ] GC(3) Concurrent Mark 2.665ms
[2.907s][info][gc,marking  ] GC(3) Concurrent Rebuild Remembered Sets
[2.907s][info][gc,marking  ] GC(3) Concurrent Rebuild Remembered Sets 0.009ms
[2.907s][info][gc,start    ] GC(3) Pause Cleanup
[2.907s][info][gc          ] GC(3) Pause Cleanup 31M->31M(64M) 0.017ms
[2.907s][info][gc,cpu      ] GC(3) User=0.00s Sys=0.00s Real=0.00s
[2.907s][info][gc,marking  ] GC(3) Concurrent Cleanup for Next Mark
[2.908s][info][gc,marking  ] GC(3) Concurrent Cleanup for Next Mark 0.501ms
[2.908s][info][gc          ] GC(3) Concurrent Mark Cycle 4.343ms
[3.111s][info][gc,start    ] GC(4) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[3.111s][info][gc,task     ] GC(4) Using 2 workers of 8 for evacuation
[3.113s][info][gc,phases   ] GC(4)   Pre Evacuate Collection Set: 0.1ms
[3.113s][info][gc,phases   ] GC(4)   Merge Heap Roots: 0.0ms
[3.113s][info][gc,phases   ] GC(4)   Evacuate Collection Set: 1.0ms
[3.113s][info][gc,phases   ] GC(4)   Post Evacuate Collection Set: 0.1ms
[3.113s][info][gc,phases   ] GC(4)   Other: 0.3ms
[3.113s][info][gc,heap     ] GC(4) Eden regions: 1->0(24)
[3.113s][info][gc,heap     ] GC(4) Survivor regions: 1->1(4)
[3.113s][info][gc,heap     ] GC(4) Old regions: 0->1
[3.113s][info][gc,heap     ] GC(4) Archive regions: 2->2
[3.113s][info][gc,heap     ] GC(4) Humongous regions: 30->30
[3.113s][info][gc,metaspace] GC(4) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[3.113s][info][gc          ] GC(4) Pause Young (Concurrent Start) (G1 Humongous Allocation) 31M->31M(64M) 1.671ms
[3.113s][info][gc,cpu      ] GC(4) User=0.00s Sys=0.00s Real=0.00s
[3.113s][info][gc          ] GC(5) Concurrent Mark Cycle
[3.113s][info][gc,marking  ] GC(5) Concurrent Clear Claimed Marks
[3.113s][info][gc,marking  ] GC(5) Concurrent Clear Claimed Marks 0.013ms
[3.113s][info][gc,marking  ] GC(5) Concurrent Scan Root Regions
[3.114s][info][gc,marking  ] GC(5) Concurrent Scan Root Regions 1.265ms
[3.114s][info][gc,marking  ] GC(5) Concurrent Mark
[3.114s][info][gc,marking  ] GC(5) Concurrent Mark From Roots
[3.114s][info][gc,task     ] GC(5) Using 2 workers of 2 for marking
[3.116s][info][gc,marking  ] GC(5) Concurrent Mark From Roots 2.053ms
[3.117s][info][gc,marking  ] GC(5) Concurrent Preclean
[3.117s][info][gc,marking  ] GC(5) Concurrent Preclean 0.062ms
[3.117s][info][gc,start    ] GC(5) Pause Remark
[3.117s][info][gc          ] GC(5) Pause Remark 33M->33M(64M) 0.480ms
[3.117s][info][gc,cpu      ] GC(5) User=0.00s Sys=0.00s Real=0.00s
[3.117s][info][gc,marking  ] GC(5) Concurrent Mark 2.905ms
[3.117s][info][gc,marking  ] GC(5) Concurrent Rebuild Remembered Sets
[3.118s][info][gc,marking  ] GC(5) Concurrent Rebuild Remembered Sets 0.511ms
[3.118s][info][gc,start    ] GC(5) Pause Cleanup
[3.118s][info][gc          ] GC(5) Pause Cleanup 33M->33M(64M) 0.043ms
[3.118s][info][gc,cpu      ] GC(5) User=0.00s Sys=0.00s Real=0.00s
[3.118s][info][gc,marking  ] GC(5) Concurrent Cleanup for Next Mark
[3.118s][info][gc,marking  ] GC(5) Concurrent Cleanup for Next Mark 0.083ms
[3.118s][info][gc          ] GC(5) Concurrent Mark Cycle 5.033ms
[3.320s][info][gc,start    ] GC(6) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[3.320s][info][gc,task     ] GC(6) Using 2 workers of 8 for evacuation
[3.321s][info][gc,phases   ] GC(6)   Pre Evacuate Collection Set: 0.1ms
[3.321s][info][gc,phases   ] GC(6)   Merge Heap Roots: 0.0ms
[3.321s][info][gc,phases   ] GC(6)   Evacuate Collection Set: 0.5ms
[3.321s][info][gc,phases   ] GC(6)   Post Evacuate Collection Set: 0.2ms
[3.321s][info][gc,phases   ] GC(6)   Other: 0.4ms
[3.321s][info][gc,heap     ] GC(6) Eden regions: 1->0(24)
[3.321s][info][gc,heap     ] GC(6) Survivor regions: 1->1(4)
[3.321s][info][gc,heap     ] GC(6) Old regions: 1->1
[3.321s][info][gc,heap     ] GC(6) Archive regions: 2->2
[3.321s][info][gc,heap     ] GC(6) Humongous regions: 32->32
[3.321s][info][gc,metaspace] GC(6) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[3.322s][info][gc          ] GC(6) Pause Young (Concurrent Start) (G1 Humongous Allocation) 33M->33M(64M) 1.452ms
[3.322s][info][gc,cpu      ] GC(6) User=0.00s Sys=0.00s Real=0.00s
[3.322s][info][gc          ] GC(7) Concurrent Mark Cycle
[3.322s][info][gc,marking  ] GC(7) Concurrent Clear Claimed Marks
[3.322s][info][gc,marking  ] GC(7) Concurrent Clear Claimed Marks 0.016ms
[3.322s][info][gc,marking  ] GC(7) Concurrent Scan Root Regions
[3.322s][info][gc,marking  ] GC(7) Concurrent Scan Root Regions 0.075ms
[3.322s][info][gc,marking  ] GC(7) Concurrent Mark
[3.322s][info][gc,marking  ] GC(7) Concurrent Mark From Roots
[3.322s][info][gc,task     ] GC(7) Using 2 workers of 2 for marking
[3.325s][info][gc,marking  ] GC(7) Concurrent Mark From Roots 2.776ms
[3.325s][info][gc,marking  ] GC(7) Concurrent Preclean
[3.325s][info][gc,marking  ] GC(7) Concurrent Preclean 0.044ms
[3.325s][info][gc,start    ] GC(7) Pause Remark
[3.325s][info][gc          ] GC(7) Pause Remark 35M->35M(64M) 0.256ms
[3.325s][info][gc,cpu      ] GC(7) User=0.00s Sys=0.00s Real=0.00s
[3.325s][info][gc,marking  ] GC(7) Concurrent Mark 3.214ms
[3.325s][info][gc,marking  ] GC(7) Concurrent Rebuild Remembered Sets
[3.326s][info][gc,marking  ] GC(7) Concurrent Rebuild Remembered Sets 0.489ms
[3.326s][info][gc,start    ] GC(7) Pause Cleanup
[3.326s][info][gc          ] GC(7) Pause Cleanup 35M->35M(64M) 0.218ms
[3.326s][info][gc,cpu      ] GC(7) User=0.00s Sys=0.00s Real=0.00s
[3.326s][info][gc,marking  ] GC(7) Concurrent Cleanup for Next Mark
[3.326s][info][gc,marking  ] GC(7) Concurrent Cleanup for Next Mark 0.193ms
[3.326s][info][gc          ] GC(7) Concurrent Mark Cycle 4.417ms
[3.528s][info][gc,start    ] GC(8) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[3.528s][info][gc,task     ] GC(8) Using 2 workers of 8 for evacuation
[3.529s][info][gc,phases   ] GC(8)   Pre Evacuate Collection Set: 0.0ms
[3.529s][info][gc,phases   ] GC(8)   Merge Heap Roots: 0.0ms
[3.529s][info][gc,phases   ] GC(8)   Evacuate Collection Set: 0.4ms
[3.529s][info][gc,phases   ] GC(8)   Post Evacuate Collection Set: 0.1ms
[3.529s][info][gc,phases   ] GC(8)   Other: 0.3ms
[3.529s][info][gc,heap     ] GC(8) Eden regions: 1->0(24)
[3.529s][info][gc,heap     ] GC(8) Survivor regions: 1->1(4)
[3.529s][info][gc,heap     ] GC(8) Old regions: 1->1
[3.529s][info][gc,heap     ] GC(8) Archive regions: 2->2
[3.529s][info][gc,heap     ] GC(8) Humongous regions: 34->34
[3.529s][info][gc,metaspace] GC(8) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[3.529s][info][gc          ] GC(8) Pause Young (Concurrent Start) (G1 Humongous Allocation) 35M->35M(64M) 1.077ms
[3.529s][info][gc,cpu      ] GC(8) User=0.00s Sys=0.00s Real=0.00s
[3.529s][info][gc          ] GC(9) Concurrent Mark Cycle
[3.529s][info][gc,marking  ] GC(9) Concurrent Clear Claimed Marks
[3.529s][info][gc,marking  ] GC(9) Concurrent Clear Claimed Marks 0.019ms
[3.529s][info][gc,marking  ] GC(9) Concurrent Scan Root Regions
[3.529s][info][gc,marking  ] GC(9) Concurrent Scan Root Regions 0.077ms
[3.529s][info][gc,marking  ] GC(9) Concurrent Mark
[3.529s][info][gc,marking  ] GC(9) Concurrent Mark From Roots
[3.529s][info][gc,task     ] GC(9) Using 2 workers of 2 for marking
[3.532s][info][gc,marking  ] GC(9) Concurrent Mark From Roots 3.013ms
[3.532s][info][gc,marking  ] GC(9) Concurrent Preclean
[3.532s][info][gc,marking  ] GC(9) Concurrent Preclean 0.048ms
[3.532s][info][gc,start    ] GC(9) Pause Remark
[3.532s][info][gc          ] GC(9) Pause Remark 37M->37M(64M) 0.294ms
[3.532s][info][gc,cpu      ] GC(9) User=0.00s Sys=0.00s Real=0.00s
[3.532s][info][gc,marking  ] GC(9) Concurrent Mark 3.487ms
[3.533s][info][gc,marking  ] GC(9) Concurrent Rebuild Remembered Sets
[3.533s][info][gc,marking  ] GC(9) Concurrent Rebuild Remembered Sets 0.562ms
[3.533s][info][gc,start    ] GC(9) Pause Cleanup
[3.533s][info][gc          ] GC(9) Pause Cleanup 37M->37M(64M) 0.044ms
[3.533s][info][gc,cpu      ] GC(9) User=0.00s Sys=0.00s Real=0.00s
[3.533s][info][gc,marking  ] GC(9) Concurrent Cleanup for Next Mark
[3.533s][info][gc,marking  ] GC(9) Concurrent Cleanup for Next Mark 0.171ms
[3.533s][info][gc          ] GC(9) Concurrent Mark Cycle 4.603ms
[3.730s][info][gc,start    ] GC(10) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[3.731s][info][gc,task     ] GC(10) Using 2 workers of 8 for evacuation
[3.731s][info][gc,phases   ] GC(10)   Pre Evacuate Collection Set: 0.1ms
[3.731s][info][gc,phases   ] GC(10)   Merge Heap Roots: 0.0ms
[3.731s][info][gc,phases   ] GC(10)   Evacuate Collection Set: 0.4ms
[3.731s][info][gc,phases   ] GC(10)   Post Evacuate Collection Set: 0.1ms
[3.731s][info][gc,phases   ] GC(10)   Other: 0.3ms
[3.731s][info][gc,heap     ] GC(10) Eden regions: 1->0(25)
[3.732s][info][gc,heap     ] GC(10) Survivor regions: 1->0(4)
[3.732s][info][gc,heap     ] GC(10) Old regions: 1->1
[3.732s][info][gc,heap     ] GC(10) Archive regions: 2->2
[3.732s][info][gc,heap     ] GC(10) Humongous regions: 36->36
[3.732s][info][gc,metaspace] GC(10) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[3.732s][info][gc          ] GC(10) Pause Young (Concurrent Start) (G1 Humongous Allocation) 37M->37M(64M) 1.106ms
[3.732s][info][gc,cpu      ] GC(10) User=0.00s Sys=0.00s Real=0.00s
[3.732s][info][gc          ] GC(11) Concurrent Mark Cycle
[3.732s][info][gc,marking  ] GC(11) Concurrent Clear Claimed Marks
[3.732s][info][gc,marking  ] GC(11) Concurrent Clear Claimed Marks 0.018ms
[3.732s][info][gc,marking  ] GC(11) Concurrent Scan Root Regions
[3.732s][info][gc,marking  ] GC(11) Concurrent Scan Root Regions 0.053ms
[3.732s][info][gc,marking  ] GC(11) Concurrent Mark
[3.732s][info][gc,marking  ] GC(11) Concurrent Mark From Roots
[3.732s][info][gc,task     ] GC(11) Using 2 workers of 2 for marking
[3.735s][info][gc,marking  ] GC(11) Concurrent Mark From Roots 2.881ms
[3.735s][info][gc,marking  ] GC(11) Concurrent Preclean
[3.735s][info][gc,marking  ] GC(11) Concurrent Preclean 0.052ms
[3.735s][info][gc,start    ] GC(11) Pause Remark
[3.735s][info][gc          ] GC(11) Pause Remark 39M->39M(64M) 0.282ms
[3.735s][info][gc,cpu      ] GC(11) User=0.00s Sys=0.00s Real=0.00s
[3.735s][info][gc,marking  ] GC(11) Concurrent Mark 3.411ms
[3.735s][info][gc,marking  ] GC(11) Concurrent Rebuild Remembered Sets
[3.736s][info][gc,marking  ] GC(11) Concurrent Rebuild Remembered Sets 0.561ms
[3.736s][info][gc,start    ] GC(11) Pause Cleanup
[3.736s][info][gc          ] GC(11) Pause Cleanup 39M->39M(64M) 0.049ms
[3.736s][info][gc,cpu      ] GC(11) User=0.00s Sys=0.00s Real=0.00s
[3.736s][info][gc,marking  ] GC(11) Concurrent Cleanup for Next Mark
[3.736s][info][gc,marking  ] GC(11) Concurrent Cleanup for Next Mark 0.127ms
[3.736s][info][gc          ] GC(11) Concurrent Mark Cycle 4.413ms
[3.935s][info][gc,start    ] GC(12) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[3.935s][info][gc,task     ] GC(12) Using 2 workers of 8 for evacuation
[3.936s][info][gc,phases   ] GC(12)   Pre Evacuate Collection Set: 0.1ms
[3.936s][info][gc,phases   ] GC(12)   Merge Heap Roots: 0.0ms
[3.936s][info][gc,phases   ] GC(12)   Evacuate Collection Set: 0.4ms
[3.936s][info][gc,phases   ] GC(12)   Post Evacuate Collection Set: 0.2ms
[3.936s][info][gc,phases   ] GC(12)   Other: 0.4ms
[3.936s][info][gc,heap     ] GC(12) Eden regions: 1->0(25)
[3.936s][info][gc,heap     ] GC(12) Survivor regions: 0->0(4)
[3.936s][info][gc,heap     ] GC(12) Old regions: 1->1
[3.936s][info][gc,heap     ] GC(12) Archive regions: 2->2
[3.937s][info][gc,heap     ] GC(12) Humongous regions: 38->38
[3.937s][info][gc,metaspace] GC(12) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[3.937s][info][gc          ] GC(12) Pause Young (Concurrent Start) (G1 Humongous Allocation) 39M->39M(64M) 1.941ms
[3.937s][info][gc,cpu      ] GC(12) User=0.00s Sys=0.00s Real=0.00s
[3.937s][info][gc          ] GC(13) Concurrent Mark Cycle
[3.937s][info][gc,marking  ] GC(13) Concurrent Clear Claimed Marks
[3.937s][info][gc,marking  ] GC(13) Concurrent Clear Claimed Marks 0.012ms
[3.937s][info][gc,marking  ] GC(13) Concurrent Scan Root Regions
[3.937s][info][gc,marking  ] GC(13) Concurrent Scan Root Regions 0.010ms
[3.937s][info][gc,marking  ] GC(13) Concurrent Mark
[3.937s][info][gc,marking  ] GC(13) Concurrent Mark From Roots
[3.937s][info][gc,task     ] GC(13) Using 2 workers of 2 for marking
[3.940s][info][gc,marking  ] GC(13) Concurrent Mark From Roots 3.079ms
[3.940s][info][gc,marking  ] GC(13) Concurrent Preclean
[3.941s][info][gc,marking  ] GC(13) Concurrent Preclean 0.058ms
[3.941s][info][gc,start    ] GC(13) Pause Remark
[3.941s][info][gc          ] GC(13) Pause Remark 41M->41M(64M) 0.269ms
[3.941s][info][gc,cpu      ] GC(13) User=0.00s Sys=0.00s Real=0.00s
[3.941s][info][gc,marking  ] GC(13) Concurrent Mark 3.664ms
[3.941s][info][gc,marking  ] GC(13) Concurrent Rebuild Remembered Sets
[3.942s][info][gc,marking  ] GC(13) Concurrent Rebuild Remembered Sets 0.585ms
[3.942s][info][gc,start    ] GC(13) Pause Cleanup
[3.942s][info][gc          ] GC(13) Pause Cleanup 41M->41M(64M) 0.037ms
[3.942s][info][gc,cpu      ] GC(13) User=0.00s Sys=0.00s Real=0.00s
[3.942s][info][gc,marking  ] GC(13) Concurrent Cleanup for Next Mark
[3.942s][info][gc,marking  ] GC(13) Concurrent Cleanup for Next Mark 0.083ms
[3.942s][info][gc          ] GC(13) Concurrent Mark Cycle 4.547ms
[4.143s][info][gc,start    ] GC(14) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[4.143s][info][gc,task     ] GC(14) Using 2 workers of 8 for evacuation
[4.144s][info][gc,phases   ] GC(14)   Pre Evacuate Collection Set: 0.1ms
[4.144s][info][gc,phases   ] GC(14)   Merge Heap Roots: 0.1ms
[4.144s][info][gc,phases   ] GC(14)   Evacuate Collection Set: 0.5ms
[4.145s][info][gc,phases   ] GC(14)   Post Evacuate Collection Set: 0.2ms
[4.145s][info][gc,phases   ] GC(14)   Other: 0.3ms
[4.145s][info][gc,heap     ] GC(14) Eden regions: 1->0(25)
[4.145s][info][gc,heap     ] GC(14) Survivor regions: 0->0(4)
[4.145s][info][gc,heap     ] GC(14) Old regions: 1->1
[4.145s][info][gc,heap     ] GC(14) Archive regions: 2->2
[4.145s][info][gc,heap     ] GC(14) Humongous regions: 40->40
[4.145s][info][gc,metaspace] GC(14) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[4.145s][info][gc          ] GC(14) Pause Young (Concurrent Start) (G1 Humongous Allocation) 41M->41M(64M) 1.359ms
[4.145s][info][gc,cpu      ] GC(14) User=0.00s Sys=0.00s Real=0.00s
[4.145s][info][gc          ] GC(15) Concurrent Mark Cycle
[4.145s][info][gc,marking  ] GC(15) Concurrent Clear Claimed Marks
[4.145s][info][gc,marking  ] GC(15) Concurrent Clear Claimed Marks 0.025ms
[4.145s][info][gc,marking  ] GC(15) Concurrent Scan Root Regions
[4.145s][info][gc,marking  ] GC(15) Concurrent Scan Root Regions 0.018ms
[4.145s][info][gc,marking  ] GC(15) Concurrent Mark
[4.145s][info][gc,marking  ] GC(15) Concurrent Mark From Roots
[4.145s][info][gc,task     ] GC(15) Using 2 workers of 2 for marking
[4.148s][info][gc,marking  ] GC(15) Concurrent Mark From Roots 3.433ms
[4.148s][info][gc,marking  ] GC(15) Concurrent Preclean
[4.148s][info][gc,marking  ] GC(15) Concurrent Preclean 0.046ms
[4.149s][info][gc,start    ] GC(15) Pause Remark
[4.149s][info][gc          ] GC(15) Pause Remark 43M->43M(64M) 0.284ms
[4.149s][info][gc,cpu      ] GC(15) User=0.00s Sys=0.00s Real=0.00s
[4.149s][info][gc,marking  ] GC(15) Concurrent Mark 3.897ms
[4.149s][info][gc,marking  ] GC(15) Concurrent Rebuild Remembered Sets
[4.149s][info][gc,marking  ] GC(15) Concurrent Rebuild Remembered Sets 0.418ms
[4.149s][info][gc,start    ] GC(15) Pause Cleanup
[4.149s][info][gc          ] GC(15) Pause Cleanup 43M->43M(64M) 0.040ms
[4.149s][info][gc,cpu      ] GC(15) User=0.00s Sys=0.00s Real=0.00s
[4.149s][info][gc,marking  ] GC(15) Concurrent Cleanup for Next Mark
[4.150s][info][gc,marking  ] GC(15) Concurrent Cleanup for Next Mark 0.117ms
[4.150s][info][gc          ] GC(15) Concurrent Mark Cycle 4.734ms
[4.346s][info][gc,start    ] GC(16) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[4.346s][info][gc,task     ] GC(16) Using 2 workers of 8 for evacuation
[4.347s][info][gc,phases   ] GC(16)   Pre Evacuate Collection Set: 0.1ms
[4.347s][info][gc,phases   ] GC(16)   Merge Heap Roots: 0.1ms
[4.347s][info][gc,phases   ] GC(16)   Evacuate Collection Set: 0.2ms
[4.347s][info][gc,phases   ] GC(16)   Post Evacuate Collection Set: 0.1ms
[4.347s][info][gc,phases   ] GC(16)   Other: 0.1ms
[4.347s][info][gc,heap     ] GC(16) Eden regions: 1->0(25)
[4.347s][info][gc,heap     ] GC(16) Survivor regions: 0->0(4)
[4.347s][info][gc,heap     ] GC(16) Old regions: 1->1
[4.347s][info][gc,heap     ] GC(16) Archive regions: 2->2
[4.347s][info][gc,heap     ] GC(16) Humongous regions: 42->42
[4.347s][info][gc,metaspace] GC(16) Metaspace: 325K(512K)->325K(512K) NonClass: 301K(384K)->301K(384K) Class: 24K(128K)->24K(128K)
[4.347s][info][gc          ] GC(16) Pause Young (Concurrent Start) (G1 Humongous Allocation) 43M->43M(64M) 0.753ms
[4.347s][info][gc,cpu      ] GC(16) User=0.00s Sys=0.01s Real=0.00s
[4.347s][info][gc          ] GC(17) Concurrent Mark Cycle
[4.347s][info][gc,marking  ] GC(17) Concurrent Clear Claimed Marks
[4.347s][info][gc,marking  ] GC(17) Concurrent Clear Claimed Marks 0.010ms
[4.347s][info][gc,marking  ] GC(17) Concurrent Scan Root Regions
[4.347s][info][gc,marking  ] GC(17) Concurrent Scan Root Regions 0.007ms
[4.347s][info][gc,marking  ] GC(17) Concurrent Mark
[4.347s][info][gc,marking  ] GC(17) Concurrent Mark From Roots
[4.347s][info][gc,task     ] GC(17) Using 2 workers of 2 for marking
[4.352s][info][gc,marking  ] GC(17) Concurrent Mark From Roots 4.678ms
[4.352s][info][gc,marking  ] GC(17) Concurrent Preclean
[4.352s][info][gc,marking  ] GC(17) Concurrent Preclean 0.031ms
[4.352s][info][gc,start    ] GC(17) Pause Remark
[4.352s][info][gc          ] GC(17) Pause Remark 45M->45M(64M) 0.189ms
[4.352s][info][gc,cpu      ] GC(17) User=0.00s Sys=0.00s Real=0.00s
[4.352s][info][gc,marking  ] GC(17) Concurrent Mark 5.034ms
[4.352s][info][gc,marking  ] GC(17) Concurrent Rebuild Remembered Sets
[4.353s][info][gc,marking  ] GC(17) Concurrent Rebuild Remembered Sets 0.711ms
[4.353s][info][gc,start    ] GC(17) Pause Cleanup
[4.353s][info][gc          ] GC(17) Pause Cleanup 45M->45M(64M) 0.208ms
[4.353s][info][gc,cpu      ] GC(17) User=0.00s Sys=0.00s Real=0.00s
[4.353s][info][gc,marking  ] GC(17) Concurrent Cleanup for Next Mark
[4.353s][info][gc,marking  ] GC(17) Concurrent Cleanup for Next Mark 0.328ms
[4.353s][info][gc          ] GC(17) Concurrent Mark Cycle 6.505ms
[4.548s][info][gc,start    ] GC(18) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[4.548s][info][gc,task     ] GC(18) Using 2 workers of 8 for evacuation
[4.549s][info][gc,phases   ] GC(18)   Pre Evacuate Collection Set: 0.1ms
[4.549s][info][gc,phases   ] GC(18)   Merge Heap Roots: 0.0ms
[4.549s][info][gc,phases   ] GC(18)   Evacuate Collection Set: 0.4ms
[4.549s][info][gc,phases   ] GC(18)   Post Evacuate Collection Set: 0.2ms
[4.549s][info][gc,phases   ] GC(18)   Other: 0.3ms
[4.549s][info][gc,heap     ] GC(18) Eden regions: 1->0(25)
[4.549s][info][gc,heap     ] GC(18) Survivor regions: 0->0(4)
[4.549s][info][gc,heap     ] GC(18) Old regions: 1->1
[4.550s][info][gc,heap     ] GC(18) Archive regions: 2->2
[4.550s][info][gc,heap     ] GC(18) Humongous regions: 44->44
[4.550s][info][gc,metaspace] GC(18) Metaspace: 326K(512K)->326K(512K) NonClass: 302K(384K)->302K(384K) Class: 24K(128K)->24K(128K)
[4.550s][info][gc          ] GC(18) Pause Young (Concurrent Start) (G1 Humongous Allocation) 45M->45M(64M) 1.235ms
[4.550s][info][gc,cpu      ] GC(18) User=0.00s Sys=0.00s Real=0.01s
[4.550s][info][gc          ] GC(19) Concurrent Mark Cycle
[4.550s][info][gc,marking  ] GC(19) Concurrent Clear Claimed Marks
[4.550s][info][gc,marking  ] GC(19) Concurrent Clear Claimed Marks 0.025ms
[4.550s][info][gc,marking  ] GC(19) Concurrent Scan Root Regions
[4.550s][info][gc,marking  ] GC(19) Concurrent Scan Root Regions 0.020ms
[4.550s][info][gc,marking  ] GC(19) Concurrent Mark
[4.550s][info][gc,marking  ] GC(19) Concurrent Mark From Roots
[4.550s][info][gc,task     ] GC(19) Using 2 workers of 2 for marking
[4.553s][info][gc,marking  ] GC(19) Concurrent Mark From Roots 3.313ms
[4.553s][info][gc,marking  ] GC(19) Concurrent Preclean
[4.553s][info][gc,marking  ] GC(19) Concurrent Preclean 0.044ms
[4.553s][info][gc,start    ] GC(19) Pause Remark
[4.554s][info][gc          ] GC(19) Pause Remark 47M->47M(64M) 0.262ms
[4.554s][info][gc,cpu      ] GC(19) User=0.00s Sys=0.00s Real=0.00s
[4.554s][info][gc,marking  ] GC(19) Concurrent Mark 3.834ms
[4.554s][info][gc,marking  ] GC(19) Concurrent Rebuild Remembered Sets
[4.554s][info][gc,marking  ] GC(19) Concurrent Rebuild Remembered Sets 0.739ms
[4.555s][info][gc,start    ] GC(19) Pause Cleanup
[4.555s][info][gc          ] GC(19) Pause Cleanup 47M->47M(64M) 0.065ms
[4.555s][info][gc,cpu      ] GC(19) User=0.00s Sys=0.00s Real=0.00s
[4.555s][info][gc,marking  ] GC(19) Concurrent Cleanup for Next Mark
[4.555s][info][gc,marking  ] GC(19) Concurrent Cleanup for Next Mark 0.164ms
[4.555s][info][gc          ] GC(19) Concurrent Mark Cycle 5.174ms
[4.756s][info][gc,start    ] GC(20) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[4.756s][info][gc,task     ] GC(20) Using 2 workers of 8 for evacuation
[4.757s][info][gc,phases   ] GC(20)   Pre Evacuate Collection Set: 0.0ms
[4.757s][info][gc,phases   ] GC(20)   Merge Heap Roots: 0.0ms
[4.757s][info][gc,phases   ] GC(20)   Evacuate Collection Set: 0.4ms
[4.757s][info][gc,phases   ] GC(20)   Post Evacuate Collection Set: 0.1ms
[4.757s][info][gc,phases   ] GC(20)   Other: 0.3ms
[4.757s][info][gc,heap     ] GC(20) Eden regions: 1->0(24)
[4.757s][info][gc,heap     ] GC(20) Survivor regions: 0->1(4)
[4.757s][info][gc,heap     ] GC(20) Old regions: 1->1
[4.757s][info][gc,heap     ] GC(20) Archive regions: 2->2
[4.757s][info][gc,heap     ] GC(20) Humongous regions: 46->46
[4.757s][info][gc,metaspace] GC(20) Metaspace: 326K(512K)->326K(512K) NonClass: 302K(384K)->302K(384K) Class: 24K(128K)->24K(128K)
[4.757s][info][gc          ] GC(20) Pause Young (Concurrent Start) (G1 Humongous Allocation) 47M->47M(64M) 1.073ms
[4.757s][info][gc,cpu      ] GC(20) User=0.00s Sys=0.00s Real=0.00s
[4.757s][info][gc          ] GC(21) Concurrent Mark Cycle
[4.757s][info][gc,marking  ] GC(21) Concurrent Clear Claimed Marks
[4.757s][info][gc,marking  ] GC(21) Concurrent Clear Claimed Marks 0.017ms
[4.757s][info][gc,marking  ] GC(21) Concurrent Scan Root Regions
[4.757s][info][gc,marking  ] GC(21) Concurrent Scan Root Regions 0.069ms
[4.757s][info][gc,marking  ] GC(21) Concurrent Mark
[4.757s][info][gc,marking  ] GC(21) Concurrent Mark From Roots
[4.757s][info][gc,task     ] GC(21) Using 2 workers of 2 for marking
[4.760s][info][gc,marking  ] GC(21) Concurrent Mark From Roots 3.337ms
[4.760s][info][gc,marking  ] GC(21) Concurrent Preclean
[4.760s][info][gc,marking  ] GC(21) Concurrent Preclean 0.042ms
[4.760s][info][gc,start    ] GC(21) Pause Remark
[4.761s][info][gc          ] GC(21) Pause Remark 49M->49M(64M) 0.265ms
[4.761s][info][gc,cpu      ] GC(21) User=0.00s Sys=0.00s Real=0.00s
[4.761s][info][gc,marking  ] GC(21) Concurrent Mark 3.786ms
[4.761s][info][gc,marking  ] GC(21) Concurrent Rebuild Remembered Sets
[4.761s][info][gc,marking  ] GC(21) Concurrent Rebuild Remembered Sets 0.626ms
[4.762s][info][gc,start    ] GC(21) Pause Cleanup
[4.762s][info][gc          ] GC(21) Pause Cleanup 49M->49M(64M) 0.045ms
[4.762s][info][gc,cpu      ] GC(21) User=0.00s Sys=0.00s Real=0.00s
[4.762s][info][gc,marking  ] GC(21) Concurrent Cleanup for Next Mark
[4.762s][info][gc,marking  ] GC(21) Concurrent Cleanup for Next Mark 0.135ms
[4.762s][info][gc          ] GC(21) Concurrent Mark Cycle 4.900ms
[4.963s][info][gc,start    ] GC(22) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[4.963s][info][gc,task     ] GC(22) Using 2 workers of 8 for evacuation
[4.964s][info][gc,phases   ] GC(22)   Pre Evacuate Collection Set: 0.1ms
[4.964s][info][gc,phases   ] GC(22)   Merge Heap Roots: 0.0ms
[4.964s][info][gc,phases   ] GC(22)   Evacuate Collection Set: 0.4ms
[4.964s][info][gc,phases   ] GC(22)   Post Evacuate Collection Set: 0.2ms
[4.964s][info][gc,phases   ] GC(22)   Other: 0.4ms
[4.964s][info][gc,heap     ] GC(22) Eden regions: 1->0(24)
[4.964s][info][gc,heap     ] GC(22) Survivor regions: 1->1(4)
[4.964s][info][gc,heap     ] GC(22) Old regions: 1->1
[4.964s][info][gc,heap     ] GC(22) Archive regions: 2->2
[4.964s][info][gc,heap     ] GC(22) Humongous regions: 48->48
[4.964s][info][gc,metaspace] GC(22) Metaspace: 326K(512K)->326K(512K) NonClass: 302K(384K)->302K(384K) Class: 24K(128K)->24K(128K)
[4.965s][info][gc          ] GC(22) Pause Young (Concurrent Start) (G1 Humongous Allocation) 49M->49M(64M) 1.350ms
[4.965s][info][gc,cpu      ] GC(22) User=0.00s Sys=0.00s Real=0.00s
[4.965s][info][gc          ] GC(23) Concurrent Mark Cycle
[4.965s][info][gc,marking  ] GC(23) Concurrent Clear Claimed Marks
[4.965s][info][gc,marking  ] GC(23) Concurrent Clear Claimed Marks 0.014ms
[4.965s][info][gc,marking  ] GC(23) Concurrent Scan Root Regions
[4.965s][info][gc,marking  ] GC(23) Concurrent Scan Root Regions 0.050ms
[4.965s][info][gc,marking  ] GC(23) Concurrent Mark
[4.965s][info][gc,marking  ] GC(23) Concurrent Mark From Roots
[4.965s][info][gc,task     ] GC(23) Using 2 workers of 2 for marking
[4.968s][info][gc,marking  ] GC(23) Concurrent Mark From Roots 3.004ms
[4.968s][info][gc,marking  ] GC(23) Concurrent Preclean
[4.968s][info][gc,marking  ] GC(23) Concurrent Preclean 0.050ms
[4.968s][info][gc,start    ] GC(23) Pause Remark
[4.968s][info][gc          ] GC(23) Pause Remark 51M->51M(64M) 0.289ms
[4.968s][info][gc,cpu      ] GC(23) User=0.00s Sys=0.00s Real=0.00s
[4.968s][info][gc,marking  ] GC(23) Concurrent Mark 3.530ms
[4.968s][info][gc,marking  ] GC(23) Concurrent Rebuild Remembered Sets
[4.969s][info][gc,marking  ] GC(23) Concurrent Rebuild Remembered Sets 0.496ms
[4.969s][info][gc,start    ] GC(23) Pause Cleanup
[4.969s][info][gc          ] GC(23) Pause Cleanup 51M->51M(64M) 0.039ms
[4.969s][info][gc,cpu      ] GC(23) User=0.00s Sys=0.00s Real=0.00s
[4.969s][info][gc,marking  ] GC(23) Concurrent Cleanup for Next Mark
[4.969s][info][gc,marking  ] GC(23) Concurrent Cleanup for Next Mark 0.106ms
[4.969s][info][gc          ] GC(23) Concurrent Mark Cycle 4.409ms
[5.172s][info][gc,start    ] GC(24) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[5.172s][info][gc,task     ] GC(24) Using 2 workers of 8 for evacuation
[5.173s][info][gc,phases   ] GC(24)   Pre Evacuate Collection Set: 0.0ms
[5.173s][info][gc,phases   ] GC(24)   Merge Heap Roots: 0.0ms
[5.173s][info][gc,phases   ] GC(24)   Evacuate Collection Set: 0.4ms
[5.173s][info][gc,phases   ] GC(24)   Post Evacuate Collection Set: 0.1ms
[5.173s][info][gc,phases   ] GC(24)   Other: 0.3ms
[5.173s][info][gc,heap     ] GC(24) Eden regions: 1->0(24)
[5.173s][info][gc,heap     ] GC(24) Survivor regions: 1->1(4)
[5.173s][info][gc,heap     ] GC(24) Old regions: 1->1
[5.173s][info][gc,heap     ] GC(24) Archive regions: 2->2
[5.173s][info][gc,heap     ] GC(24) Humongous regions: 50->50
[5.173s][info][gc,metaspace] GC(24) Metaspace: 327K(512K)->327K(512K) NonClass: 303K(384K)->303K(384K) Class: 24K(128K)->24K(128K)
[5.173s][info][gc          ] GC(24) Pause Young (Concurrent Start) (G1 Humongous Allocation) 51M->51M(64M) 0.978ms
[5.173s][info][gc,cpu      ] GC(24) User=0.01s Sys=0.00s Real=0.00s
[5.173s][info][gc          ] GC(25) Concurrent Mark Cycle
[5.173s][info][gc,marking  ] GC(25) Concurrent Clear Claimed Marks
[5.173s][info][gc,marking  ] GC(25) Concurrent Clear Claimed Marks 0.012ms
[5.173s][info][gc,marking  ] GC(25) Concurrent Scan Root Regions
[5.173s][info][gc,marking  ] GC(25) Concurrent Scan Root Regions 0.185ms
[5.173s][info][gc,marking  ] GC(25) Concurrent Mark
[5.173s][info][gc,marking  ] GC(25) Concurrent Mark From Roots
[5.174s][info][gc,task     ] GC(25) Using 2 workers of 2 for marking
[5.175s][info][gc,start    ] GC(26) Pause Young (Normal) (G1 Preventive Collection)
[5.175s][info][gc,task     ] GC(26) Using 2 workers of 8 for evacuation
[5.176s][info][gc,phases   ] GC(26)   Pre Evacuate Collection Set: 0.0ms
[5.176s][info][gc,phases   ] GC(26)   Merge Heap Roots: 0.0ms
[5.176s][info][gc,phases   ] GC(26)   Evacuate Collection Set: 0.1ms
[5.176s][info][gc,phases   ] GC(26)   Post Evacuate Collection Set: 0.1ms
[5.176s][info][gc,phases   ] GC(26)   Other: 0.1ms
[5.176s][info][gc,heap     ] GC(26) Eden regions: 0->0(24)
[5.176s][info][gc,heap     ] GC(26) Survivor regions: 1->1(1)
[5.176s][info][gc,heap     ] GC(26) Old regions: 1->1
[5.176s][info][gc,heap     ] GC(26) Archive regions: 2->2
[5.176s][info][gc,heap     ] GC(26) Humongous regions: 59->59
[5.176s][info][gc,metaspace] GC(26) Metaspace: 328K(512K)->328K(512K) NonClass: 304K(384K)->304K(384K) Class: 24K(128K)->24K(128K)
[5.176s][info][gc          ] GC(26) Pause Young (Normal) (G1 Preventive Collection) 60M->60M(64M) 0.445ms
[5.176s][info][gc,cpu      ] GC(26) User=0.00s Sys=0.00s Real=0.00s
[5.177s][info][gc,task     ] GC(27) Using 2 workers of 8 for full compaction
[5.177s][info][gc,start    ] GC(27) Pause Full (System.gc())
[5.178s][info][gc,phases,start] GC(27) Phase 1: Mark live objects
[5.179s][info][gc,phases      ] GC(27) Phase 1: Mark live objects 1.305ms
[5.179s][info][gc,phases,start] GC(27) Phase 2: Prepare for compaction
[5.179s][info][gc,phases      ] GC(27) Phase 2: Prepare for compaction 0.229ms
[5.179s][info][gc,phases,start] GC(27) Phase 3: Adjust pointers
[5.180s][info][gc,phases      ] GC(27) Phase 3: Adjust pointers 0.526ms
[5.180s][info][gc,phases,start] GC(27) Phase 4: Compact heap
[5.180s][info][gc,phases      ] GC(27) Phase 4: Compact heap 0.264ms
[5.180s][info][gc,heap        ] GC(27) Eden regions: 1->0(25)
[5.180s][info][gc,heap        ] GC(27) Survivor regions: 1->0(1)
[5.180s][info][gc,heap        ] GC(27) Old regions: 1->2
[5.180s][info][gc,heap        ] GC(27) Archive regions: 2->2
[5.180s][info][gc,heap        ] GC(27) Humongous regions: 59->0
[5.180s][info][gc,metaspace   ] GC(27) Metaspace: 329K(512K)->329K(512K) NonClass: 304K(384K)->304K(384K) Class: 24K(128K)->24K(128K)
[5.180s][info][gc             ] GC(27) Pause Full (System.gc()) 60M->1M(64M) 2.887ms
[5.180s][info][gc,cpu         ] GC(27) User=0.01s Sys=0.00s Real=0.01s
[5.180s][info][gc,marking     ] GC(25) Concurrent Mark From Roots 6.900ms
[5.180s][info][gc,marking     ] GC(25) Concurrent Mark Abort
[5.180s][info][gc             ] GC(25) Concurrent Mark Cycle 7.187ms
[7.188s][info][gc,heap,exit   ] Heap
[7.188s][info][gc,heap,exit   ]  garbage-first heap   total 65536K, used 2347K [0x00000007fc000000, 0x0000000800000000)
[7.188s][info][gc,heap,exit   ]   region size 1024K, 1 young (1024K), 0 survivors (0K)
[7.188s][info][gc,heap,exit   ]  Metaspace       used 329K, committed 512K, reserved 1114112K
[7.188s][info][gc,heap,exit   ]   class space    used 24K, committed 128K, reserved 1048576K

```

## 日志分析
| 程序阶段与时间戳 (Phase & Timestamp) | 核心代码行为 (Core Code Action) | GC 日志关键证据 (Key GC Log Evidence) | 深入解读与分析 (In-depth Interpretation and Analysis) |
| :--- | :--- | :--- | :--- |
| **阶段 1：启动至首次 GC** <br> (0.009s ~ 4.364s) | 程序启动，并开始循环分配 400KB 的“小对象”。 | `(此阶段无 GC 日志)` | **解读**：由于对象大小（400KB）小于大对象阈值（500KB），它们被作为普通对象分配在 **Eden 区**。**重要的是，在您的代码循环开始前，JVM自身初始化（加载类、JIT编译、执行`println`等）已经占用了部分Eden空间。** 因此，并不需要您的代码分配全部24MB来触发GC，JVM自身的内存消耗是填充Eden区的主要部分。 |
| **阶段 1：首次 GC** <br> **GC(0) @ 4.364s** | **JVM初始化**和**循环中最初几个400KB对象**的分配共同填满了Eden区。 | `[4.364s][info][gc,start ] GC(0) Pause Young (Normal) (G1 Evacuation Pause)`<br><br>`[4.366s][info][gc,heap ] GC(0) Eden regions: 24->0(24)`<br>`[4.366s][info][gc,heap ] GC(0) Survivor regions: 0->1(4)`<br>`[4.366s][info][gc,heap ] GC(0) Old regions: 15->16` | **解读**：<br>1. **触发原因**: 时间戳证明GC发生在阶段1循环的**早期**。是JVM自身内存占用 + **代码中最初1-2个400KB对象**这“最后一根稻草”共同填满了Eden区，触发了这次Young GC。<br>2. **Eden 区**: `24->0(24)`，24MB的Eden区被完全清空。<br>3. **Survivor 区**: `0->1(4)`，**追踪对象**：这些存活并被复制到Survivor区的对象，主要是**JVM自身的一些对象**，以及您代码中**最开始被创建并放入`objectHolder`的那一两个400KB的`byte[]`数组**。它们成功**逃离了本次Eden回收**，年龄变为1。<br>4. **老年代**: 晋升的1MB对象**不是**我们代码循环中的对象，因为它们的年龄才刚变为1，还未达到晋升阈值2。 |
| **阶段 2：快速分配** <br> **GC(1) @ 4.568s** <br> 至 <br> **GC(7) @ 5.176s** | 循环快速分配 400KB 对象，持续填满 Eden 区，反复触发 Young GC。 | `[4.568s][info][gc,start ] GC(1) Pause Young (Normal) (G1 Evacuation Pause)`<br><br>`[4.s][info][gc,heap ] GC(1) Survivor regions: 1->1(4)`<br>`[4.569s][info][gc,heap ] GC(1) Old regions: 31->36` | **解读**：<br>1. **GC 模式**: 这是标准的、由 Eden 耗尽驱动的 Young GC 循环。<br>2. **Survivor 区流转**: `Survivor regions: 1->1(4)` 体现了 Survivor 区的复制机制。`GC(0)` 的幸存者和本次 Eden 的幸存者（都是被`objectHolder`持有的 `byte[]` 数组）一起被复制到另一个 Survivor 区。<br>3. **对象晋升**: `Old regions: 31->36` 是**对象晋升到老年代**的直接证据。**追踪对象**：这增加的 **5MB 老年代空间**，主要就是用来存放那些在 `GC(0)` 中存活（年龄变为1），并在本次 `GC(1)` 中再次存活（年龄达到2）的 **400KB `byte[]` 对象**。它们现在**完成了从 Eden -> Survivor -> Old 的完整晋升路径**。 |
| **阶段 3：分配大对象** <br> **GC(8) @ 5.176s** | 代码执行 `new byte[8 * 1024 * 1024]`。 | `[5.176s][info][gc,start ] GC(8) Pause Young (Concurrent Start) (G1 Humongous Allocation)`<br><br>`[5.179s][info][gc,heap ] GC(8) Humongous regions: 0->8` | **解读**：<br>1. **行为突变**: **追踪对象**：与之前所有 400KB 的对象不同，这个 **8MB 的 `byte[]` 数组**由于体积巨大（>0.5MB），被 G1 识别为大对象，触发了 `(G1 Humongous Allocation)`。<br>2. **大对象区证据**: `Humongous regions: 0->8` 证明这个 8MB 对象没有进入 Eden，而是被**直接分配到了老年代**的 8 个连续 Humongous Region 中。 |
| **阶段 4：手动 Full GC** <br> **GC(9) @ 5.385s** | 执行 `objectHolder.clear()` 和 `System.gc()`。 | `[5.385s][info][gc,start ] GC(9) Pause Full (System.gc())`<br><br>`[5.388s][info][gc,heap ] GC(9) Humongous regions: 8->0`<br>`[5.388s][info][gc,heap ] GC(9) Old regions: 51->29` | **解读**：<br>1. **全局回收**: `(System.gc())` 触发了一次对整个堆的回收。<br>2. **大对象回收**: `Humongous regions: 8->0`，**追踪对象**：在阶段3创建的 **8MB `largeObject`** 因为引用被设为 `null`，在此次 Full GC 中被**完全回收**。<br>3. **老年代回收**: `Old regions: 51->29`，**追踪对象**：这被回收的 **22MB 对象**，正是**之前从新生代一路晋升上来、并被 `objectHolder` 持有的那些 400KB 的 `byte[]` 数组**。在调用 `objectHolder.clear()` 后，它们失去了强引用，因此在 Full GC 中被识别为垃圾并回收。 |