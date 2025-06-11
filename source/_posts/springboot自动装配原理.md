---
title: springboot自动装配原理
date: 2025-01-09 11:39:52
categories: SpringBoot
tags: SpringBoot
---

# springboot自动装配原理
具体实现代码
```
this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
```
具体执行如下
```
 private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
        context.setEnvironment(environment);
        this.postProcessApplicationContext(context);
        this.addAotGeneratedInitializerIfNecessary(this.initializers);
        this.applyInitializers(context);
        listeners.contextPrepared(context);
        bootstrapContext.close(context);
        if (this.logStartupInfo) {
            this.logStartupInfo(context.getParent() == null);
            this.logStartupProfileInfo(context);
        }

        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
        if (printedBanner != null) {
            beanFactory.registerSingleton("springBootBanner", printedBanner);
        }

        if (beanFactory instanceof AbstractAutowireCapableBeanFactory autowireCapableBeanFactory) {
            autowireCapableBeanFactory.setAllowCircularReferences(this.allowCircularReferences);
            if (beanFactory instanceof DefaultListableBeanFactory listableBeanFactory) {
                listableBeanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
            }
        }

        if (this.lazyInitialization) {
            context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
        }

        if (this.keepAlive) {
            context.addApplicationListener(new KeepAlive());
        }

        context.addBeanFactoryPostProcessor(new PropertySourceOrderingBeanFactoryPostProcessor(context));
        if (!AotDetector.useGeneratedArtifacts()) {
            Set<Object> sources = this.getAllSources();
            Assert.notEmpty(sources, "Sources must not be empty");
            this.load(context, sources.toArray(new Object[0]));
        }

        listeners.contextLoaded(context);
    }
```
自动配置的执行过程
this.applyInitializers(context);：
这个方法会遍历所有注册的ApplicationContextInitializer，并调用它们的initialize()方法.
ApplicationContextInitializer接口的实现类可以对应用程序上下文进行进一步的配置和准备，包括加载自动配置类、注册Bean定义等.
在Spring Boot中，自动配置类通常通过@AutoConfigureAfter、@AutoConfigureBefore等注解来指定加载顺序，确保自动配置的正确性和一致性.
其他相关步骤
listeners.contextPrepared(context);：
发布ContextPreparedEvent事件，通知其他监听器上下文已经准备就绪。这个事件可以触发一些基于上下文准备好的操作，例如加载配置文件、初始化资源等.
this.load(context, sources.toArray(new Object[0]));：
加载应用程序的配置类和组件扫描路径，注册Bean定义到Bean工厂中。这个步骤会根据配置类上的注解（如@ComponentScan、@Import等）来扫描和注册Bean.