---
title: "SpringBoot启动debug"
date: 2024-11-18 15:41:52
---

### 1.SpringApplication.run(AdminApplication.class, args);

这是SpringApplication.Class一个静态方法，debug进去会执行下面这个方法

``` java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}
```

第一个参数为<span style="font-size: 14px; color: rgb(6, 6, 7)">是一个泛型参数，表示主配置类，第二个是可变长度参数，返回的是执行另一个run方法</span>

``` java
 public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```

这两个方法比较类似，这个是接收的是数组（可变长度参数贬义时被转化为数组），这里先new一个SpringApplication对象，在执行他的run方法，先看SpringApplication对象，属性有下面这些，具体解释请看api文档

``` java
public static final String BANNER_LOCATION_PROPERTY_VALUE = "banner.txt";
    public static final String BANNER_LOCATION_PROPERTY = "spring.banner.location";
    private static final String SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = "java.awt.headless";
    private static final Log logger = LogFactory.getLog(SpringApplication.class);
    static final SpringApplicationShutdownHook shutdownHook = new SpringApplicationShutdownHook();
    private static final ThreadLocal<SpringApplicationHook> applicationHook = new ThreadLocal();
    private final Set<Class<?>> primarySources;
    private Set<String> sources;
    private Class<?> mainApplicationClass;
    private Banner.Mode bannerMode;
    private boolean logStartupInfo;
    private boolean addCommandLineProperties;
    private boolean addConversionService;
    private Banner banner;
    private ResourceLoader resourceLoader;
    private BeanNameGenerator beanNameGenerator;
    private ConfigurableEnvironment environment;
    private WebApplicationType webApplicationType;
    private boolean headless;
    private boolean registerShutdownHook;
    private List<ApplicationContextInitializer<?>> initializers;
    private List<ApplicationListener<?>> listeners;
    private Map<String, Object> defaultProperties;
    private final List<BootstrapRegistryInitializer> bootstrapRegistryInitializers;
    private Set<String> additionalProfiles;
    private boolean allowBeanDefinitionOverriding;
    private boolean allowCircularReferences;
    private boolean isCustomEnvironment;
    private boolean lazyInitialization;
    private String environmentPrefix;
    private ApplicationContextFactory applicationContextFactory;
    private ApplicationStartup applicationStartup;
```

看一下run方法，官方解释作用为运行 <span color="#ef4444" style="color: #ef4444">Spring 应用程序，创建并刷新一个新的 </span>`ApplicationContext`

``` java
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        Banner printedBanner = this.printBanner(environment);
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }

        listeners.started(context, timeTakenToStartup);
        this.callRunners(context, applicationArguments);
    } catch (Throwable var12) {
        if (var12 instanceof AbandonedRunException) {
            throw var12;
        }

        this.handleRunFailure(context, var12, listeners);
        throw new IllegalStateException(var12);
    }

    try {
        if (context.isRunning()) {
            Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
            listeners.ready(context, timeTakenToReady);
        }

        return context;
    } catch (Throwable var11) {
        if (var11 instanceof AbandonedRunException) {
            throw var11;
        } else {
            this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var11);
        }
    }
}
```

### DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();

这个方法返回一个DefaultBootstrapContext对象，看一下具体执行流程

``` java
private DefaultBootstrapContext createBootstrapContext() {     DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();     this.bootstrapRegistryInitializers.forEach((initializer) -> {         initializer.initialize(bootstrapContext);     });     return bootstrapContext; }
```

new了一个DefaultBootstrapContext，可以查看这个类，api文档或者直接搜索

<img src="/upload/截屏2024-10-18%2010.30.13-gqqz.png" style="display: inline-block;width:100.0%;height:100.0%" />

执行这个构造方法时，会初始化属性SimpleApplicationEventMulticaster()

### SimpleApplicationEventMulticaster

`SimpleApplicationEventMulticaster` 是Spring框架中用于事件发布和分发的一个类。它实现了 `ApplicationEventMulticaster` 接口，负责在Spring应用程序中管理和分发各种事件。这个类是Spring事件机制的核心组件之一。

默认情况下，所有侦听器都在调用线程中调用。这样可能会出现恶意侦听器阻塞整个应用程序的危险，但会增加最小的开销。指定备用任务执行器，让侦听器在不同线程中执行，例如从线程池中执行

以下是 `SimpleApplicationEventMulticaster` 的主要特点和功能：

1\. **事件监听器管理**：

\- `SimpleApplicationEventMulticaster` 维护一个事件监听器列表，用于存储和管理所有注册的事件监听器（\`ApplicationListener\` 实现）。

\- 它允许你在运行时添加或移除事件监听器。

2\. **事件发布**：

\- 当一个事件被发布时，\`SimpleApplicationEventMulticaster\` 负责将事件分发给所有注册的监听器。

\- 它确保每个监听器都能接收到事件，并在单独的线程中处理，以避免阻塞事件发布者。

3\. **异步事件处理**：

\- 从Spring 5开始，\`SimpleApplicationEventMulticaster\` 支持异步事件处理。这意味着事件监听器可以在不同的线程中异步执行，从而提高应用程序的响应性和吞吐量。

\- 你可以通过设置 `setAsync` 属性来启用或禁用异步事件处理。

4\. **事件过滤**：

\- `SimpleApplicationEventMulticaster` 允许你在运行时添加或移除事件类型过滤器（\`SmartApplicationListener\` 实现）。

\- 这些过滤器可以根据事件类型或其他条件来决定是否应该将事件分发给特定的监听器。

5\. **初始化和销毁回调**：

\- `SimpleApplicationEventMulticaster` 实现了 `ApplicationContextAware` 和 `InitializingBean` 接口，这意味着它可以在Spring容器初始化和销毁时执行自定义逻辑。

\- 在容器启动时，它会初始化事件监听器列表；在容器关闭时，它会销毁所有注册的监听器。

6\. **配置和自定义**：

\- 你可以通过继承 `SimpleApplicationEventMulticaster` 类并重写其方法来自定义事件分发逻辑。

\- Spring还提供了其他扩展点，如 `ApplicationListenerAdapter` 和 `ApplicationEventFilter`，允许你进一步自定义事件处理行为。

在Spring应用程序中，默认情况下，\`SimpleApplicationEventMulticaster\` 被配置为事件多播器，负责处理所有事件的发布和分发。你可以通过编程方式或在配置文件中自定义其行为，以满足特定需求。

例如，以下代码演示了如何在Spring配置中自定义 `SimpleApplicationEventMulticaster`：

``` java
import org.springframework.context.ApplicationEventMulticaster;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;

import org.springframework.context.event.SimpleApplicationEventMulticaster;

@Configuration

public class EventConfig {

    @Bean

    public ApplicationEventMulticaster applicationEventMulticaster() {

        SimpleApplicationEventMulticaster multicaster = new SimpleApplicationEventMulticaster();

        // 这里可以自定义multicaster的配置，如设置监听器、启用异步处理等

        return multicaster;

    }

}
```

通过这种方式，你可以灵活地控制Spring应用程序中的事件处理机制，以满足不同的业务需求。

执行SimpleApplicationEventMulticaster构造器后会执行父类构造器AbstractApplicationEventMulticaster

<img src="/upload/截屏2024-10-18%2010.39.48.png" style="display: inline-block;width:100.0%;height:100.0%" />

### BootstrapRegistryInitializer

回到createBootstrapContext方法执行体中，遍历bootstrapRegistryInitializers，他是一个不可变的List\<BootstrapRegistryInitializer\>

``` java
private final List<BootstrapRegistryInitializer> bootstrapRegistryInitializers;
```

在Spring框架中，\`BootstrapRegistryInitializer\` 是一个接口

<img src="/upload/截屏2024-10-18%2010.49.54-yusn.png" style="display: inline-block;width:100.0%;height:100.0%" />

它定义了在Spring Boot的自动配置过程中，初始化 `BootstrapContext` 的行为。目的是为了允许开发者在自动配置开始之前，注册额外的配置和组件。

`BootstrapRegistryInitializer` 接口包含一个方法 `initialize`，该方法会在Spring Boot的自动配置初始化阶段被调用。这使得开发者可以在自动配置类被加载和条件评估之前，向 `BootstrapContext` 添加额外的配置。

以下是 `BootstrapRegistryInitializer` 接口的典型用法：

``` java

import org.springframework.boot.autoconfigure.AutoConfigurationPackages;

import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;

import org.springframework.boot.autoconfigure.domain.EntityScanPackages;

import org.springframework.boot.context.event.ApplicationStartingEvent;

import org.springframework.context.annotation.Configuration;

import org.springframework.core.io.support.SpringFactoriesLoader;

import org.springframework.core.type.filter.AnnotationTypeFilter;

import java.util.List;

@Configuration

public class MyBootstrapInitializer {

    @EventListener

    public void init(ApplicationStartingEvent event) {

        List<Class<? extends BootstrapRegistryInitializer>> initializers =

                SpringFactoriesLoader.loadFactories(

                        BootstrapRegistryInitializer.class,

                        getClass().getClassLoader());

        for (Class<? extends BootstrapRegistryInitializer> initializerClass : initializers) {

            try {

                BootstrapRegistryInitializer initializer = initializerClass.newInstance();

                initializer.initialize(event.getEnvironment(), event.getSpringApplication().getClassLoader());

            } catch (InstantiationException | IllegalAccessException ex) {

                throw new RuntimeException("Failed to instantiate " + initializerClass, ex);

            }

        }

    }

}
```

在这个例子中，我们定义了一个 `@Configuration` 类，它监听 `ApplicationStartingEvent` 事件。当Spring Boot应用开始时，它会加载所有通过 `SpringFactoriesLoader` 指定的 `BootstrapRegistryInitializer` 实现，并调用它们的 `initialize` 方法。

`initialize` 方法的签名如下：

``` java
void initialize(ConfigurableBootstrapContext bootstrapContext);
```

\- `ConfigurableBootstrapContext` 是一个可以配置的 `BootstrapContext`，它允许你注册额外的 `BeanDefinition` 或者修改现有的 `BeanDefinition`。

\- `bootstrapContext` 参数提供了一个注册器，你可以使用它来注册组件和配置。

通过实现 `BootstrapRegistryInitializer` 接口，开发者可以控制Spring Boot自动配置的加载过程，例如添加自定义的自动配置类、修改自动配置的条件，或者为自动配置提供必要的上下文信息。

请注意，\`BootstrapRegistryInitializer\` 是一个高级特性，通常只在需要深度定制Spring Boot自动配置时使用。对于大多数应用，标准的自动配置机制已经足够灵活，能够满足大多数需求

继续回到run方法，this.configureHeadlessProperty()，方法为

``` java
private void configureHeadlessProperty() {
        System.setProperty("java.awt.headless", System.getProperty("java.awt.headless", Boolean.toString(this.headless)));
    }
```

可以看到设计GUI,可自行google或ai，下一步SpringApplicationRunListeners listeners = this.getRunListeners(args);

``` java
private SpringApplicationRunListeners getRunListeners(String[] args) {
        SpringFactoriesLoader.ArgumentResolver argumentResolver = ArgumentResolver.of(SpringApplication.class, this);
        argumentResolver = argumentResolver.and(String[].class, args);
        List<SpringApplicationRunListener> listeners = this.getSpringFactoriesInstances(SpringApplicationRunListener.class, argumentResolver);
        SpringApplicationHook hook = (SpringApplicationHook)applicationHook.get();
        SpringApplicationRunListener hookListener = hook != null ? hook.getRunListener(this) : null;
        if (hookListener != null) {
            listeners = new ArrayList((Collection)listeners);
            ((List)listeners).add(hookListener);
        }

        return new SpringApplicationRunListeners(logger, (List)listeners, this.applicationStartup);
    }
```

[](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/SpringFactoriesLoader.ArgumentResolver.html)这段代码是 `SpringApplication` 类中用于获取和配置 `SpringApplicationRunListeners` 的一个方法。这个方法负责创建和初始化一组监听器，这些监听器会在Spring应用启动的不同阶段被通知。下面是对这个方法的详细解释：

1\. **创建参数解析器**：

\`\`\`java

SpringFactoriesLoader.ArgumentResolver argumentResolver = ArgumentResolver.of(SpringApplication.class, this);

\`\`\`

这行代码创建了一个 `ArgumentResolver` 实例，用于解析与 `SpringApplication` 相关的参数。\`ArgumentResolver\` 会根据 `SpringApplication` 类中定义的参数类型来解析传递给 `SpringApplication` 的参数。

2\. **添加命令行参数**：

\`\`\`java

argumentResolver = argumentResolver.and(String\[\].class, args);

\`\`\`

这行代码将命令行参数 `args` 添加到参数解析器中。这样，\`ArgumentResolver\` 就可以解析命令行参数，并将其作为 `String[]` 类型的参数处理。

3\. **加载监听器实例**：

\`\`\`java

List\<SpringApplicationRunListener\> listeners = this.getSpringFactoriesInstances(SpringApplicationRunListener.class, argumentResolver);

\`\`\`

这行代码使用 `SpringFactoriesLoader` 加载所有实现了 `SpringApplicationRunListener` 接口的类的实例。这些实例会在Spring应用启动的不同阶段接收通知。\`argumentResolver\` 用于传递解析后的参数给这些监听器的构造函数。

4\. **添加SpringApplicationHook监听器**：

\`\`\`java

SpringApplicationHook hook = (SpringApplicationHook)applicationHook.get();

SpringApplicationRunListener hookListener = hook != null ? hook.getRunListener(this) : null;

\`\`\`

这段代码检查是否有 `SpringApplicationHook` 实例，如果有，则获取其关联的 `SpringApplicationRunListener` 实例。

5\. **合并监听器列表**：

\`\`\`java

if (hookListener != null) {

listeners = new ArrayList((Collection)listeners);

((List)listeners).add(hookListener);

}

\`\`\`

如果存在 `hookListener`，则将其添加到监听器列表中。这里首先将 `listeners` 转换为 `ArrayList`，以便可以添加新的监听器。

6\. **创建SpringApplicationRunListeners实例**：

\`\`\`java

return new SpringApplicationRunListeners(logger, (List)listeners, this.applicationStartup);

\`\`\`

最后，这段代码创建一个新的 `SpringApplicationRunListeners` 实例，它包含了所有配置好的监听器。这个实例会在Spring应用启动的不同阶段通知这些监听器。

这个方法的目的是确保在Spring应用启动过程中，所有相关的监听器都能接收到适当的事件通知，从而允许开发者在应用启动的不同阶段执行自定义逻辑。
