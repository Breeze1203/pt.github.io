---
title: SpringBoot执行异步任务
date: 2024-12-18 15:45:57
categories:
 - springboot
tags: 
 - java
 - springboot
---

#####  SpringBoot执行异步任务
1.@Async注解
![截屏2024-12-18 16.24.20.png](/upload/截屏2024-12-18%2016.24.20.png)
大概解释是异步执行的方法：
- 你可以在方法上使用@Async注解，这样Spring会在调用这个方法时异步执行它。
如果你在类级别使用@Async注解，那么这个类中的所有方法都会被视为异步方法。
配置类中的使用限制：
- @Async注解不能用于带有@Configuration注解的类中的方法。这是因为配置类通常用于定义Spring容器的配置，而不是作为业务逻辑的一部分。
方法签名：
- @Async注解的方法可以有任意类型的参数。
返回类型必须是void或者Future类型的，包括ListenableFuture和CompletableFuture，这些类型提供了更丰富的异步任务交互能力，并且可以立即与后续的处理步骤组合。
- 返回Future的处理：当方法被标记为@Async后，调用这个方法会返回一个Future对象，这个对象可以用来跟踪异步方法的执行结果。
- 由于目标方法需要与代理方法有相同的签名，如果方法本身不返回Future，那么它需要返回一个临时的Future对象，这个对象只是简单地传递值。Spring提供了AsyncResult类来帮助实现这一点，或者你可以使用EJB 3.1的AsyncResult，或者Java的CompletableFuture.completedFuture(Object)方法。
AnnotationAsyncExecutionInterceptor：这是一个拦截器，用于处理@Async注解的方法调用。
AsyncAnnotationAdvisor：这是一个Spring AOP顾问，用于创建代理以支持@Async注解

![截屏2024-12-18 16.29.58.png](/upload/截屏2024-12-18%2016.29.58.png)
- 限定符值（Qualifier Value）：这个属性是一个限定符值，用于确定执行异步操作时应该使用的目标执行器。
它匹配具有特定限定符值（或bean名称）的java.util.concurrent.Executor或org.springframework.core.task.TaskExecutor bean定义。
类级别和方法级别的使用：
- 如果在类级别的@Async注解中指定了这个限定符值，那么表示类中所有的方法都应该使用指定的执行器。
如果在方法级别使用了Async#value，则总是会覆盖在类级别配置的限定符值。
动态解析：
- 如果限定符值被提供为一个SpEL（Spring Expression Language）表达式或属性占位符，它将被动态解析。例如，可以使用SpEL表达式"#{environment['myExecutor']}"来动态指定执行器，或者使用属性占位符"${my.app.myExecutor}"来从配置中读取执行器的名称。
```
@Async
    public void asyncTaskTwo() {
        // 异步任务逻辑
        System.out.println("Async task two is running with thread: " + Thread.currentThread().getName());
    }
```
2.使用 CompletableFuture 实现异步任务
CompletableFuture 是 Java 8 新增的一个异步编程工具，它可以方便地实现异步任务。使用 CompletableFuture 需要满足以下条件：
 1.异步任务的返回值类型必须是 CompletableFuture 类型；
 2.在异步任务中使用 CompletableFuture.supplyAsync() 或 CompletableFuture.runAsync() 方法来创建异步任务；
 3.在主线程中使用 CompletableFuture.get() 方法获取异步任务的返回结果。
示例代码如下：
```
@Service
public class AsyncService {
    public CompletableFuture<String> asyncTask() {
        return CompletableFuture.supplyAsync(() -> {
            // 异步任务执行的逻辑
            return "异步任务执行完成";
        });
    }
}
```
3.利用Executor
在上下文中没有 Executor bean 的情况下， Spring Boot 会自动配置AsyncTaskExecutor。 启用虚拟线程（使用 Java 21+ 并设置为 ）时，这将是使用虚拟线程的 SimpleAsyncTaskExecutor。 否则，它将是具有合理默认值的 ThreadPoolTaskExecutor。 
如果您在上下文中定义了自定义 Executor，则常规任务执行（即 @EnableAsync）和 Spring for GraphQL 都将使用它。 但是，Spring MVC 和 Spring WebFlux 支持仅在它是 AsyncTaskExecutor 实现（名为 ）时才会使用它。 根据你的目标安排，你可以将 Executor 更改为 AsyncTaskExecutor，或者同时定义 AsyncTaskExecutor 和 AsyncConfigurer 来包装你的自定义 Executor。applicationTaskExecutor
自动配置的 ThreadPoolTaskExecutorBuilder 允许您轻松创建实例，这些实例可以重现自动配置默认执行的操作。
当 ThreadPoolTaskExecutor 被自动配置时，线程池使用 8 个核心线程，这些线程可以根据负载进行扩展和收缩。 可以使用命名空间对这些默认设置进行微调，如以下示例所示：spring.task.execution
spring.task.execution.pool.max-size=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
这会将线程池更改为使用有界队列，以便在队列已满（100 个任务）时，线程池增加到最多 16 个线程。 池的收缩更加激进，因为线程在空闲 10 秒（而不是默认 60 秒）时被回收。
如果需要将计划程序与计划任务执行相关联（例如，使用 @EnableScheduling），也可以自动配置计划程序。
如果启用了虚拟线程（使用 Java 21+ 并设置为 ），这将是使用虚拟线程的 SimpleAsyncTaskScheduler。 此 SimpleAsyncTaskScheduler 将忽略任何与池相关的属性。spring.threads.virtual.enabledtrue
如果未启用虚拟线程，它将是具有合理默认值的 ThreadPoolTaskScheduler。 默认情况下，ThreadPoolTaskScheduler 使用一个线程，并且可以使用命名空间对其设置进行微调，如以下示例所示：spring.task.scheduling
```
@Configuration
@EnableAsync
public class AppConfig implements AsyncConfigurer {

     @Override
     public Executor getAsyncExecutor() {
         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
         executor.setCorePoolSize(7);
         executor.setMaxPoolSize(42);
         executor.setQueueCapacity(11);
         executor.setThreadNamePrefix("MyExecutor-");
         executor.initialize();
         return executor;
     }

     @Override
     public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
         return new MyAsyncUncaughtExceptionHandler();
     }
 }
```
这是配置Executor，其实可以跟1一起搭配使用，因为@Async注解，也是通过异步操作时应该使用的目标执行器，我们还可以这样做
```
@Configuration
public class AsyncConfig{

    @Bean(value = "task")
    public Executor getAsyncExecutor() {
        return new ThreadPoolTaskExecutorBuilder().build();
    }
}
```
然后业务类，依赖注入这个bean，类似于下面
```
 public void asyncTask() {
       task.execute(()->{
           System.out.println("Async task is running with thread: " + Thread.currentThread().getName());
       });
    }
```
SpringBoot官方默认是配置 Executor的bean，搭配@Async注解