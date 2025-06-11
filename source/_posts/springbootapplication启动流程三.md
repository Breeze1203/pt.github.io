---
title: "SpringBootApplication启动流程三"
date: 2025-06-11 19:23:39
categories: SpringBoot
tags: SpringBoot
---

#### <span style="font-size: 12px">public ConfigurableApplicationContext run(String... args) {} run方法</span>

<span style="font-size: 14px; color: rgb(44, 62, 80)">就是给上下文做准备，会调用</span>`spring`<span style="font-size: 14px; color: rgb(44, 62, 80)">的初始化，会进行不同初始化阶段的广播，去通知监听器，监听器就可以做一些扩展的事情啦，比如初始化自己的环境什么的</span>

``` java
运行Spring应用程序，创建并刷新一个新的{@link ApplicationContext}。
* @param args 应用程序参数（通常由Java主方法传递）
* @return 一个正在运行的{@link ApplicationContext}

public ConfigurableApplicationContext run(String... args) {
        Startup startup = Startup.create();
        if (this.registerShutdownHook) {
            SpringApplication.shutdownHook.enableShutdownHookAddition();
        }
        DefaultBootstrapContext bootstrapContext = createBootstrapContext();
        ConfigurableApplicationContext context = null;
        configureHeadlessProperty();
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);
        try {
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            Banner printedBanner = printBanner(environment);
            context = createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            refreshContext(context);
            afterRefresh(context, applicationArguments);
            startup.started();
            if (this.logStartupInfo) {
                new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), startup);
            }
            listeners.started(context, startup.timeTakenToStarted());
            callRunners(context, applicationArguments);
        }
        catch (Throwable ex) {
            throw handleRunFailure(context, ex, listeners);
        }
        try {
            if (context.isRunning()) {
                listeners.ready(context, startup.ready());
            }
        }
        catch (Throwable ex) {
            throw handleRunFailure(context, ex, null);
        }
        return context;
    }
```

<span style="font-size: 14px">Startup startup = Startup.create();</span>

<span style="font-size: 14px">在Spring的上下文中，</span>`Startup`<span style="font-size: 14px">类提供了一种方式来跟踪和分析应用程序启动时各个阶段所花费的时间。这对于性能调优和理解应用程序启动过程中的耗时点非常有用。</span>

<span style="font-size: 14px">以下是</span>`Startup.create()`<span style="font-size: 14px">方法可能执行的一些操作：</span>

1.  **<span style="font-size: 14px">记录启动信息</span>**<span style="font-size: 14px">：创建</span>`Startup`<span style="font-size: 14px">实例时，它会开始记录应用程序启动的时间。</span>

2.  **<span style="font-size: 14px">测量启动时间</span>**<span style="font-size: 14px">：</span>`Startup`<span style="font-size: 14px">实例可以测量应用程序启动过程中各个阶段的耗时，例如加载配置、初始化上下文、加载和创建bean等。</span>

3.  **<span style="font-size: 14px">打印报告</span>**<span style="font-size: 14px">：在应用程序启动完成后，</span>`Startup`<span style="font-size: 14px">实例可以生成一个报告，显示启动过程中各个阶段的耗时，这有助于识别性能瓶颈。</span>

4.  **<span style="font-size: 14px">条件性记录</span>**<span style="font-size: 14px">：</span>`Startup`<span style="font-size: 14px">类可能包含一些条件，例如只有在应用程序的某些配置下才会记录启动信息，或者只有在特定的环境（如开发环境）中才会打印报告</span>

<!-- -->

    (this.registerShutdownHook) {          SpringApplication.shutdownHook.enableShutdownHookAddition();        }

<span style="font-size: 14px">这段代码是Spring Boot SpringApplication 类的一部分，用于处理应用程序的优雅关闭。</span>

<span style="font-size: 14px">1. if (this.registerShutdownHook)：这是一个条件判断，检查\`registerShutdownHook\`字段是否为\`true\`。这个字段通常在\`SpringApplication\`的配置中设置，用来决定是否需要注册一个JVM关闭钩子（shutdown hook）。</span>

<span style="font-size: 14px">2. SpringApplication.shutdownHook.enableShutdownHookAddition();：如果条件判断为真，那么会调用\`SpringApplication\`类上的\`shutdownHook\`静态字段的\`enableShutdownHookAddition()\`方法。这个方法的作用是允许或禁止添加更多的关闭钩子。在Spring Boot中，这通常用于确保在应用程序关闭时，能够执行一些清理工作，比如关闭数据库连接、停止运行的任务等。</span>

``` java
DefaultBootstrapContext bootstrapContext = createBootstrapContext();

private DefaultBootstrapContext createBootstrapContext() {
        DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
        this.bootstrapRegistryInitializers.forEach((initializer) -> initializer.initialize(bootstrapContext));
        return bootstrapContext;
    }
```

<span style="font-size: 14px">用于在Spring应用的引导阶段（bootstrap phase）提供上下文信息。这个上下文是在应用的主方法（main method）执行之前建立的，用于初始化和配置底层的Spring环境。</span>

<span style="font-size: 14px">以下是</span>`createBootstrapContext()`<span style="font-size: 14px">方法可能执行的一些操作：</span>

1.  **<span style="font-size: 14px">初始化上下文</span>**<span style="font-size: 14px">：创建一个新的</span>`DefaultBootstrapContext`<span style="font-size: 14px">实例，这个实例将持有引导阶段所需的所有配置和环境信息。</span>

2.  **<span style="font-size: 14px">配置属性</span>**<span style="font-size: 14px">：设置引导上下文中的属性，这些属性可能来自于命令行参数、环境变量、配置文件等。</span>

3.  **<span style="font-size: 14px">注册监听器</span>**<span style="font-size: 14px">：注册一些监听器或回调函数，以便在引导阶段的特定时间点执行特定的逻辑。</span>

4.  **<span style="font-size: 14px">设置环境</span>**<span style="font-size: 14px">：配置</span>`Environment`<span style="font-size: 14px">对象，确保它包含了所有必要的配置源和属性转换服务。</span>

5.  **<span style="font-size: 14px">返回上下文</span>**<span style="font-size: 14px">：方法返回配置好的</span>`DefaultBootstrapContext`<span style="font-size: 14px">实例，以便在应用程序的引导阶段使用。</span>

``` java
configureHeadlessProperty();


private void configureHeadlessProperty() {
    System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,           System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
    }
```

<span style="font-size: 14px">配置</span><span style="font-size: 14px; color: rgb(6, 6, 7)">作用是配置 Java 系统属性 </span>`java.awt.headless`

``` java
SpringApplicationRunListeners listeners = getRunListeners(args);

private SpringApplicationRunListeners getRunListeners(String[] args) {
        ArgumentResolver argumentResolver = ArgumentResolver.of(SpringApplication.class, this);
        argumentResolver = argumentResolver.and(String[].class, args);
        List<SpringApplicationRunListener> listeners = getSpringFactoriesInstances(SpringApplicationRunListener.class,
                argumentResolver);
        SpringApplicationHook hook = applicationHook.get();
        SpringApplicationRunListener hookListener = (hook != null) ? hook.getRunListener(this) : null;
        if (hookListener != null) {
            listeners = new ArrayList<>(listeners);
            listeners.add(hookListener);
        }
        return new SpringApplicationRunListeners(logger, listeners, this.applicationStartup);
    }
```

<span style="font-size: 14px">用于获取一个 SpringApplicationRunListeners 实例，该实例代表一组应用程序启动监听器。</span>

1.  <span style="font-size: 14px">getRunListeners(args)：这是一个调用方法，它接收应用程序参数 args 作为输入，这些参数通常从Java的主方法传递过来。这个方法的作用是从这些参数中提取和初始化启动监听器。</span>

2.  <span style="font-size: 14px">SpringApplicationRunListeners：这是一个类，封装了一组 ApplicationListener 对象，这些监听器对 ApplicationStartingEvent、ApplicationEnvironmentPreparedEvent、ApplicationPreparedEvent、ApplicationStartedEvent、ApplicationFailedEvent 等事件感兴趣。这些事件分别对应于Spring Boot应用程序生命周期的不同阶段。</span>

3.  <span style="font-size: 14px">listeners：这是一个变量，它将存储 getRunListeners(args) 方法返回的 SpringApplicationRunListeners 实例。</span>

<span style="font-size: 14px">这行代码的作用是初始化并获取一个 SpringApplicationRunListeners 实例，该实例将在应用程序启动的不同阶段发布相应的事件，以便注册的监听器可以做出响应</span>

``` java
listeners.starting(bootstrapContext, this.mainApplicationClass);
```

<span style="font-size: 14px">这行代码是在Spring Boot的启动过程中调用的，用于通知所有的\`ApplicationListener\`监听器应用程序即将开始启动。具体来说：</span>

<span style="font-size: 14px">1. listeners：这是之前通过\`getRunListeners(args)\`方法获取的\`SpringApplicationRunListeners\`实例，它包含了一组监听器，这些监听器监听Spring Boot应用程序生命周期中的各种事件。</span>

<span style="font-size: 14px">2. starting：这是\`SpringApplicationRunListeners\`类中的一个方法，它用于触发\`ApplicationStartingEvent\`事件。这个方法通常接受两个参数：一个是\`bootstrapContext\`，另一个是\`mainApplicationClass\`。</span>

<span style="font-size: 14px">3. bootstrapContext：这是一个\`BootstrapContext\`实例，它提供了引导应用程序时的上下文信息。</span>

<span style="font-size: 14px">4. this.mainApplicationClass：这是\`SpringApplication\`实例中的一个字段，表示包含\`main\`方法的主应用程序类。</span>

<span style="font-size: 14px">当调用\`listeners.starting(bootstrapContext, this.mainApplicationClass);\`这行代码时，它会做以下几件事情：</span>

<span style="font-size: 14px">- 创建一个\`ApplicationStartingEvent\`事件，包含有关启动过程的详细信息，如引导上下文和主应用程序类。</span>

<span style="font-size: 14px">- 将这个事件发布给所有注册的\`ApplicationListener\`监听器。</span>

<span style="font-size: 14px">- 监听器可以响应这个事件，执行一些在应用程序正式启动之前的初始化或配置工作。</span>

<span style="font-size: 14px">这个步骤是在Spring Boot的启动流程中非常早期的阶段执行的，它标志着应用程序启动过程的开始。</span>

<span style="font-size: 14px">ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);</span>

<span style="font-size: 14px">这行代码创建了一个新的 ApplicationArguments 对象，用于封装和提供对传递给 Spring Boot 应用程序入口点的命令行参数的访问。具体来说：</span>

<span style="font-size: 14px">1. \`ApplicationArguments\`：这是一个接口，定义了访问应用程序参数的方法。</span>

<span style="font-size: 14px">2. \`DefaultApplicationArguments\`：这是 ApplicationArguments 接口的一个实现类，它提供了对命令行参数的默认处理。</span>

<span style="font-size: 14px">3. \`args\`：这是传递给 SpringApplication.run() 方法的原始字符串数组，通常由 Java 的 main 方法传入。</span>

<span style="font-size: 14px">4. \`new DefaultApplicationArguments(args)\`：这里通过传递 args 数组给 DefaultApplicationArguments 的构造函数来创建一个新的实例。这个实例将存储和处理这些参数，允许你之后可以通过 ApplicationArguments 提供的方法来查询这些参数。</span>

<span style="font-size: 14px">DefaultApplicationArguments 类允许你查询所有的参数、选项（以 -- 开头的参数），以及非选项参数（不以 -- 开头的参数）。它还支持通过选项名称来查询特定的选项值，并且能够处理短选项和长选项的不同组合。</span>
