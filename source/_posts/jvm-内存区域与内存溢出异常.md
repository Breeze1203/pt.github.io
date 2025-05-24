---
title: "jvm 内存区域与内存溢出异常"
date: 2024-11-18 15:41:52
---

### **主要内容**

- 运行时数据区域

- 对象访问

- OutOfMemoryError异常

### **运行时数据区域**

虚拟机会将java程序过程中所管理的内存划分为不同数据区域，这些区域都有各自的用途，以及创建和销毁时间，注意分为以下几个部分：

<img src="/upload/运行时数据区域.png" style="display: inline-block;width:100.0%;height:100.0%" />

##### **程序计数器**

**当前线程所执行的字节码的行号指示器**，字节码指示器通过改变计数器的值来选去需要执行的字节码指令，分支，循环，跳转，异常处理，线程恢复等。java虚拟机通过线程轮流切换并分配处理器时间来实现，因此，<u>为了线程能恢复到正确执行位置，每条线程都都有一块独立的程序计数器，保证每个线程之间计数器互不影响，独立存储，</u>**<u>线程私有</u>**

如果线程执行的是一个java方法，计数器记录的是字节码指令地址，如果是native，则计数器值为空（Undefined）,此区域是**<u>唯一</u>**<u>一个在虚拟机规范中没有规定任何内存溢（OutOfMemoryError）的区域</u>

##### **java虚拟机栈**

与程序计数器一样，java虚拟机栈也是**线程私有**的，<u>生命周期与线程相同</u>。描述的是**java方法执行的内存模型**，每个方法执行的时候会创建一个栈桢，栈桢用来存储局部变量表，操作栈，动态链接，方法出口等信息，当一个方法被调用直至完成的过程，标志着一个栈桢从虚拟机栈入栈到出栈的过程

局部变量表存放了编译器可知的各种**基本数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference类型，他可能是一个执行对象起始位置的引用指针，也可能指向一个代表对象的句柄或其它与此对象相关的位置）和**returnAddress类型**（指向一条字节码指令的地址）

虚拟机规范中，此区域有两种异常情况，线程请求的栈深度大于虚拟机所允许深度，将抛出Stac kOverflowError异常；虚拟机动态扩展是无法申请到足够内存抛出OutOfMemoryError

内存溢出（Memory Overflow）：内存溢出通常指的是程序在运行过程中，由于某种原因（如无限循环、递归调用过深等）导致其占用的内存超过了系统分配给它的内存空间

内存泄漏（Memory Leak）：内存泄漏是指在计算机程序中，由于疏忽或错误导致的一种内存分配后无法释放的现象，即分配出去的内存没有被正确回收

##### **本地方法栈**

本地方法栈与java虚拟机栈类似，虚拟机栈为执行java方法服务，本地方法栈为执行Native方法服务

##### **堆（Heap）**

堆是所有**线程共享**的一块内存区域，在虚拟机启动时创建，此内存区域唯一目的存放对象实例，**在虚拟机规范中，所有对象实例以及数组都要在堆上分配**，java堆也是垃圾收集器管理的主要区域（GC堆），如果从内存回收的角度看，由于现在垃圾收集器采用的<u>分代收集，所以java堆分为新生代，老年代，在细致一点有Eden、From Survivor、to Survivor</u>，在java虚拟机规范中，java堆可以处于不连续的内存空间中，只要在逻辑上连续即可，<u>堆的大小也是可以扩展的（通过-Xmx和-Xms控制）</u>，如果堆中没有内存完成实例分配且无法扩展，则抛出OutOfMemoryError

##### **方法区**

与堆一样，是各个**线程共享**的内存区域，**存放的是已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据**，在虚拟机规范中将方法区描述为堆的一个逻辑部分，但它有一个别名叫做Non-Heap（非堆），目的应该是与java堆区分开来，对于习惯在HotSpot虚拟机上开发的开发者来说，很多人又愿意将方法区称为“永久代”，对于其它虚拟机是没有这个概念的，java虚拟机对这个区域的限制是非常宽松的，不需要连续的内存和可以选择固定大小或可扩展外，还<u>可以选择不实现垃圾收集</u>，相对而言垃圾收集行为在这个区域还是比较少见的，这个内存区域回收目标为常量池的回收和对类型的卸载，当方法区无法满足内存分配需求，将抛出OutOfMemoryError异常

##### **运行时常量池**

是方法区的一部分，Class文件中除了类版本，字段，方法，接口等描述信息外，还有常量池，常量池用来存放编译器生成的各种字面量和符号应用，这部分内容在**类加载**后存放到方法区的**运行时常量池**，运行时常量池属于方法区，会受到方法区的限制，也会抛出OutOfMemoryError异常

### **对象访问**

Object obj=new Object()，假设这句代码出现在方法体里面，那<u>Object obj会反映到java栈中的本地方法表中，作为一个引用（reference）类型数据出现，而new Object（）这部分语句反映到java堆中，形成一块存储了Object类型所有实力数据值，对象中的各个实例子段的结构化内存，这块内存长度是不固定的，在java中还必须包含能查找到此对象的类型数据（如对象类型，父类，实现的接口，方法等）的地址信息，这些类型数据则存在方法区中</u>

引用类型定义这个引用应该通过那种方式去定位，主流的访问方式有两种：使用**句柄**和**直接指针**

##### **句柄访问**

java堆中会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了实例数据和类型数据各种各自具体地址信息

<img src="/upload/句柄池.png" style="display: inline-block;width:100.0%;height:100.0%" />

##### **指针访问**

堆对象的布局中要考虑如何放置访问类型数据的相关的信息，reference中直接存储的就是对象地址

<img src="/upload/指针.png" style="display: inline-block;width:100.0%;height:100.0%" />

两种各有优势，句柄好处是reference中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而reference本身不需要修改，指针访问最大的好处是访问速度更快，节省了一次指针定位时间开销

### **实战：OutOfMemoryError异常**

##### **堆溢出**

只要我们不断创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制来清除这些对象，就会在对象数量到达最大堆的容量限制后产生内存溢出异常 ，设置堆的最小值-Xms，最大值-Xms，另外通过参数-XX:+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照一以便事后分析

MV Args: -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError

``` java
public class HeapooM (
​
static class OOMObject {
}
​
public static void main (Stringl) args) {
List<00MObject> l i s t = new ArrayList<OOMObject>() ; while (true) (
list. ad (new OOMObject ()) :
​
}
```

##### **虚拟机栈和本地方法栈溢出**

-Xoss参数（设置本地方法栈大小，虽然存在，但实际上无效），栈容量只由-Xss参数设定，关于虚拟机栈和本地方法栈，虚拟机规范中描述了两种异常

- 如果线程请求的栈深度大虚拟机所允许的最大深度 ，将抛出StackOverflowError异常。

- 又如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError 异常

``` java
/*
* MV Args: -Xss128k * @author zzm
*/
public class JavaVMStackSOF {
  
private int stackLength= 1;
  
public void stackLeak (){
  stackLength++;
  s t a c k L e a k ();
}
  
  
public s t a t i c void main(String() args) throws
Throwable {
JavaVMStackSOF oom = new JavaVMStackSOF () ; 
try{
     oom.stackLeak () ; 
catch(Throwable e) {
     System.out.println("stack length:"+ oom.stackLength) ;
throw e;
}
  
  
运行结果:
stack length:2402
Exception ni thread "main" java. lang.StackOverflowError
at org.fenixsoft.oom.VMStackSOF.leak (VMStackSOF.java:20)
```

实验结果表明，无论是由于栈桢太大，还是虚拟机栈容量太小，当内存无法分配的时候，都是抛出StackOverflowError

##### **运行时常量池溢出**

如果要向运行时常量池中添加内容，最简单的做法是使用Strinf.intern()这个native方法，这个方法的作用是如果池中包含一个等一此String对象的字符串，则返回代表池中这个字符串的String对象，否则，将此String对象包含的字符串添加到常量池中，并返回此s tring对象的引用，由于常量池分配在方法区，可以通过-XX:PermSize和-XX:MaxPermSiza限制方法区的大小，从而限制常量池的容量

``` java
/*
*MV Args: X-X: PermSize=10M -XX:MaxPermSize=10M * @author zm
*/
public class RuntimeConstant Pool00M (
public static void main(String() args) (
// 使用List 保持着常量池引用，避免FU11 GC回收常量池行为 
  List<String> list = new ArrayList<String>();
// 10MB的Permsize在integer 范園內足够产生0OM了
  int i = 0;
while (true) {
list.add (String.valueof (i++) .intern());
}
​
运行结果:
Exception in thread "main" java.lang.OutOfMemoryError: Permen space
at java.lang.String. intern (Native Method)
at org.fenixsoft.oom. RuntimeConstantPool00M.main (RuntimeConstantPoolOOM.java:18)
```

从运行结果中可以看到，运行时常量池溢出，在OutOfMemoryError后面跟随的提 示 信 息 是 “ PermGenspace ” ， 说 明 **运 行 时 常 量 池 属 于 方 法 区** (HotSpot 虚 拟 机 中 的 永 久 代)的 一部分。

##### **方法区溢出**

方 法 区 用 于存 放 Class 的 相 关 信 息 ， 如 类 名 、 访 问 修 饰 符 、 常 量 池 、 字 段 描 述、方法描述等。对 于这个区域的测试，基本的思路是运行时产生大量的类去填 满方法区，直到溢出。虽然直接使用Java SE API 也可以动态产生类 (如反射时的 GeneratedConstructorAccessor 和动态代理等)，但在本次实验中操作起来比较麻 烦。下面，笔者借助CGLib“直接操作字节码运行时，生成了大量的 动态类。 值得特别注意的是，我们在这个例子中模拟的场景并非纯粹是一个实验，这样的应 用经常会出现在实际应用中:当前的很多主流框架，如Spring和Hibernat e对类进行增 强时，都会使用到CGLib 这类字节码技术，增强的类越多，就需要越大的方法区来保证 动态生成的Class 可以加载人内存

``` java
/**
* MV Args: -XX: Permsize=10M -XX:MaxPermSize=10M * @author zzm
*/
public class JavaMethodAreaOoM (
  
public static void main(String() args) {
  while(true) {
     Enhancer enhancer = new Enhancer (); 
     enhancer. setSuperclass (00MObject.class);
     enhancer. setUseCache (false);
     enhancer. setCallback (new MethodInterceptor () (
         public Object intercept (Object obj, Method method, Object () args, MethodProxy proxy) throws Throwable {
           return proxy.invokeSuper (obj, args) ;
         }
});
enhancer.create();
                            }
                         }
static class OOMObject{}
}
​
                            
运行结果:
Caused by: java.lang.OutOfMemoryError: PermGen space at java. lang.ClassLoader.defineClass1 (Native Method)
at java.lang.ClassLoader.defineClassCond(Classloader.java:632) at java.lang.ClassLoader.defineClass(ClassLoader.java:616)
... 8more
```

方法区溢出也是一种常见的内存溢出异常， 一个类如果要被垃圾收集器回收掉，判 定条件是非常苛刻的。在经常动态生成大量Class 的应用中，需要特别注意类的回收状 况。这类场景除了上面提到的程序使用了GCLib 字节码增强外，常见的还有:<u>大量JSP 或 动 态 产 生 JSP 文 件 的 应 用 (JSP 第 一次 运 行 时 需 要 编 译 为 Java 类 )</u>、 基 于 OSGi 的 应 用 (即使是同 一个类文件，被不同的加载器加载也会视为不同的类)等

##### **本地直接内存溢出**

DirectMemory容量可通过-XX:MaxDirectMemorySize指定，如果不指定，则 默 认 与 Java 堆 的 最 大 值 (- X m x 指 定 ) 一 样 。 代 码 清 单 2 - 6 越 过 了 DirectByteBuffer 类 ， 直 接 通 过 反 射 获 取 Unsafe 实 例 并 进 行 内 存 分 配 (Unsafe类 的getUnsafe ()方 法限制了只有引导类加载器才会返回实例，也就是设计者希望只有rt.jar 中的类 才能使用Unsafe的功能)。因为，虽然使用DirectByteBuffer 分配内存也会抛出 内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过 计 算 得 知 内 存 无 法 分 配 ， 于是 手动 抛 出 异 常 ， 真 正 申 请 分 配 内 存 的 方 法 是 unsafeallocateMemory.

``` java
MV Args: -Xmx20M -XX:MaxDirectMemorySize=10M * @author zzm
*/
public class DirectMemoryOOM {
  
     private static final int M_IB = 1024 * 1024;
  
     public static void main(String() args) throws Exception { 
       Field unsafeField = Unsafe.class.getDeclaredFields() [0);        unsafeField. setAccessible (true);
       Unsafe unsafe = (Unsafe) unsafeField.get (null) ;
       while (true){
       unsafe.allocateMemory (_1MB);
       }
    } 
}
```
