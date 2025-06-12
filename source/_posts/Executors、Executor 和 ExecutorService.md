---
title: Executors、Executor 和 ExecutorService
date: 2024-12-02 16:01:52
categories: JAVA
tags: 
 - 并发
 - JAVA
---

###### 1.Executors、Executor 和 ExecutorService
（1）Executors是一个帮助类，提供了创建几种预配置线程池实例的方法。如果你不需要应用任何自定义的微调，可以调用这些方法创建默认配置的线程池。
（2）Executor 和 ExecutorService接口则用于与 Java 中不同线程池的实现协同工作。
Executor 和 ExecutorService 这两个接口主要的区别是：ExecutorService 接口继承了 Executor 接口，是 Executor 的子接口
Executor 和 ExecutorService 第二个区别是：Executor 接口定义了 execute()方法用来接收一个Runnable接口的对象，而 ExecutorService 接口中的 submit()方法可以接受Runnable和Callable接口的对象。
Executor 和 ExecutorService 接口第三个区别是 Executor 中的 execute() 方法不返回任何结果，而 ExecutorService 中的 submit()方法可以通过一个 Future 对象返回运算结果。
Executor 和 ExecutorService 接口第四个区别是除了允许客户端提交一个任务，ExecutorService 还提供用来控制线程池的方法。比如：调用 shutDown() 方法终止线程池。可以通过 《Java Concurrency in Practice》 一书了解更多关于关闭线程池和如何处理 pending 的任务的知识。
2.用Executors创建的通用线程池
（1）Executors.newFixedThreadPool(n)
创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。超出的线程会在队列中等待，可控制线程最大并发数。创建的线程池 corePoolSize 和 maximumPoolSize 值是相等的，使用的是 LinkedBlockingQueue 阻塞队列。执行长期的任务，性能好很多。底层实现如下：
```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());

```
（2）Executors.newSingleThreadExecutor()
创建一个单线程的线程池，这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。将 corePoolSize 和 maximumPoolSize 都设置为1，使用的是 LinkedBlockingQueue 阻塞队列。适合一个任务一个任务执行的场景。底层实现如下：
```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>()));
}
```
（3）Executors.newCachedThreadPool()
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收线程，则新建线程。将 corePoolSize 设置为0，maximumPoolSize 设置为Integer.MAX_VALUE ，使用的阻塞队列是SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。适合执行很多短期异步的小程序或者负载较轻的服务器。
```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,
                        new SynchronousQueue<Runnable>());
}
```
（4）Executors.newScheduledThreadPool(n)
创建一个定长线程池，支持定时及周期性任务执行。
```
 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
```
（5） Executors.newWorkStealingPool()
JDK8引入，创建持有足够线程的线程池支持给定的并行度，并通过使用多个队列减少竞争。

    public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
禁止直接使用Executors创建线程池原因：
FixedThreadPool 和 SingleThreadPool:
允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
CachedThreadPool 和 ScheduledThreadPool:
允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。
3.ThreadPoolExecutor 创建线程池（推荐）
```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) 
```
（1）参数解释
corePoolSize（核心线程数） ： 线程池中常驻核心线程数。当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。
maximumPoolSize （最大线程数）： 线程池允许创建的最大线程数，此值>=1。如果队列满了，并且已创建的线程小于最大线程数，则线程池也会创建新的线程执行任务。
此值一般设置为多少？考虑这个问题首先要分析你的系统是 CPU 密集型，还是 IO 密集型的服务。再就是查看系统的内核数：Runtime.getRuntime().availableProcessors());
①、CPU 密集型：CPU 密集型任务只有在真正的多核 CPU 上才可能得到加速，CPU 一直全速运行。而在单核 CPU 上，无论你开几个模拟的多线程任务都不能得到加速，因为 CPU 总的运算能力就那些。一般公式：线程数=CPU核数+1
②、IO 密集型：IO 密集型的任务并不是一直在执行任务，则应配置尽可能多的线程。一般公式：线程数=CPU核数*2
③、IO 密集型：IO 密集型时，大部分线程都阻塞，故需多配置线程数。一般公式：线程数=CPU核数/1-阻塞系数。阻塞系数：一般阻塞系数取值在0.8~0.9 之间。
keepAliveTime （线程空闲时间）： 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间
unit： keepAliveTime 的时间单位，可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微妙（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微妙）。
workQueue（缓存队列） ： 用来储存等待执行任务的队列
threadFactory （线程工厂）： 用来生产一组相同任务的线程。主要用于设置生成的线程名词前缀、是否为守护线程以及优先级等。设置有意义的名称前缀有利于在进行虚拟机分析时，知道线程是由哪个线程工厂创建的。
使用开源框架guava 提供的 ThreadFactoryBuilder 可以快速给线程池里的线程设置有意义的名字，一般使用默认即可。如下：
new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
handler： 执行拒绝策略对象。当达到任务缓存上限时（即超过workQueue参数能存储的任务数），执行拒接策略，创建线程执行任务，当线程数量等于corePoolSize时，请求加入阻塞队列里，当队列满了时，接着创建线程，线程数等于maximumPoolSize。 当任务处理不过来的时候，线程池开始执行拒绝策略。
（2）阻塞队列
ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
DelayQueue： 一个使用优先级队列实现的无界阻塞队列。
SynchronousQueue： 一个不存储元素的阻塞队列。
LinkedTransferQueue： 一个由链表结构组成的无界阻塞队列。
LinkedBlockingDeque： 一个由链表结构组成的双向阻塞队列。
（3）拒绝策略
ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常。 (默认)
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务。（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务。
（4）提交任务
可以向ThreadPoolExecutor提交两种任务：Callable和Runnable
Callable
该类任务有返回结果，可以抛出异常。
通过submit函数提交，返回Future对象。
可通过get获取执行结果。
Runnable
该类任务只执行，无法获取返回结果，并在执行过程中无法抛异常。
通过execute提交。
（5）线程池关闭
shutdown：将线程池状态置为SHUTDOWN,并不会立即停止：停止接收外部submit的任务,内部正在跑的任务和队列里等待的任务，会执行完后，才真正停止

shutdownNow：将线程池状态置为STOP。企图立即停止，事实上不一定：跟shutdown()一样，先停止接收外部提交的任务，忽略队列里等待的任务，尝试将正在跑的任务interrupt中断，返回未执行的任务列表。

它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它也可能必须要等待所有正在执行的任务都执行完成了才能退出。

awaitTermination(long timeOut, TimeUnit unit)当前线程阻塞，直到等所有已提交的任务（包括正在跑的和队列中等待的）执行完或者等超时时间到或者线程被中断，抛出InterruptedException，然后返回true（shutdown请求后所有任务执行完毕）或false（已超时）
4.自定义阻塞提交的MyThread（防止拒绝忽略，任务得不到处理）
```
MyThread.java

public class MyThread  implements  Runnable{
    private Integer number;

    public MyThread(int number){
        this.number = number;
    }

    public Integer getNumber() {
        return number;
    }

    @Override
    public void run() {
        try {
             //to do
            TimeUnit.SECONDS.sleep(1);
            System.out.println("Hello! ThreadPoolExecutor - " + getNumber());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
CustomBlockThreadPoolExecutor.java



/**
 * 自定义阻塞提交的ThreadPoolExecutor
 */
public class CustomBlockThreadPoolExecutor {

    private ThreadPoolExecutor pool = null;
    private final  int  poolSize=2;
    private final  int  maxPoolSize=4;
    private final  Long  keepAliveTime=30L;
    private final  int  arrayBlockingQueueSize=30;

    /**
     * 线程池初始化方法
     *
     * corePoolSize 核心线程池大小----2
     * maximumPoolSize 最大线程池大小----4
     * keepAliveTime 线程池中超过corePoolSize数目的空闲线程最大存活时间----30+单位TimeUnit
     * TimeUnit keepAliveTime时间单位----TimeUnit.MINUTES
     * workQueue 阻塞队列----new ArrayBlockingQueue<Runnable>(10)==== 10容量的阻塞队列
     * threadFactory 新建线程工厂----new CustomThreadFactory()====定制的线程工厂
     * rejectedExecutionHandler 当提交任务数超过maxmumPoolSize+workQueue之和时,
     * 即当提交第15个任务时(前面线程都没有执行完,此测试方法中用sleep(100)),任务会交给RejectedExecutionHandler来处理
     */

    public void init() {
        pool = new ThreadPoolExecutor(poolSize,maxPoolSize,keepAliveTime,
                TimeUnit.SECONDS,new ArrayBlockingQueue<Runnable>(arrayBlockingQueueSize),new CustomThreadFactory(), new CustomRejectedExecutionHandler());
    }

    public void destory() {
        if(pool !=null) {
            pool.shutdownNow();
        }
    }

    public ExecutorService getCustomThreadPoolExecutor() {
        return this.pool;
    }


    /**
     * 自定义线程工厂类
     * 生成的线程名词前缀、是否为守护线程以及优先级等
     */
    private class CustomThreadFactory implements ThreadFactory {

        private AtomicInteger count = new AtomicInteger(0);

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            String threadName =  CustomBlockThreadPoolExecutor.class.getSimpleName()+count.addAndGet(1);
            t.setName(threadName);
            return t;
        }
    }


    /**
     * 自定义拒绝策略对象
     */
    private class CustomRejectedExecutionHandler implements RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            //核心改造点,将blockingqueue的offer改成put阻塞提交
            try {
                executor.getQueue().put(r);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 当提交任务被拒绝时，进入拒绝机制，我们实现拒绝方法，把任务重新用阻塞提交方法put提交，实现阻塞提交任务功能，防止队列过大，OOM
     */
    public static void main(String[] args){

        CustomBlockThreadPoolExecutor customlockThreadPoolExecutor = new CustomBlockThreadPoolExecutor();

        //初始化
        customlockThreadPoolExecutor.init();
        ExecutorService pool = customlockThreadPoolExecutor.getCustomThreadPoolExecutor();
        for(int i=1;i<51;i++) {
            MyThread myThread=new MyThread(i);
            System.out.println("提交第"+i+"个任务");
            pool.execute(myThread);
        }


        pool.shutdown();
        try {
            //阻塞，超时时间到或者线程被中断
            if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
                //立即关闭
                pool.shutdownNow();
            }
        } catch (InterruptedException e) {
            pool.shutdownNow();
        }

    }
