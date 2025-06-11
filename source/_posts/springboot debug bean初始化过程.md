---
title: springboot debug bean初始化过程
date: 2025-01-09 10:29:51
categories:
 - springboot
tags: 
 - java
---

# springboot debug bean初始化过程
直接来看AbstractApplicationContext类的refresh()方法的实现。这个方法是Spring容器启动的核心方法，用于初始化和刷新Spring应用程序上下文
```
public void refresh() throws BeansException, IllegalStateException {
		this.startupShutdownLock.lock();
		try {
			this.startupShutdownThread = Thread.currentThread();
			StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
			// Prepare this context for refreshing.
			prepareRefresh();
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);
			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);
				StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				beanPostProcess.end();
				// Initialize message source for this context.
				initMessageSource();
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
				// Initialize other special beans in specific context subclasses.
				onRefresh();
				// Check for listener beans and register them.
				registerListeners();
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			catch (RuntimeException | Error ex ) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}
				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();
				// Reset 'active' flag.
				cancelRefresh(ex);
				// Propagate exception to caller.
				throw ex;
			}
			finally {
				contextRefresh.end();
			}
		}
		finally {
			this.startupShutdownThread = null;
			this.startupShutdownLock.unlock();
		}
	}
```
第一个：prepareRefresh();从注释可以看出，这是做准备（为刷新上下文做准备，包括设置活跃标志、初始化事件发布者等）
```
/**
	 * Prepare this context for refreshing, setting its startup date and
	 * active flag as well as performing any initialization of property sources.
	 */
	protected void prepareRefresh() {
		// Switch to active.
		this.startupDate = System.currentTimeMillis();
		this.closed.set(false);
		this.active.set(true);
		if (logger.isDebugEnabled()) {
			if (logger.isTraceEnabled()) {
				logger.trace("Refreshing " + this);
			}
			else {
				logger.debug("Refreshing " + getDisplayName());
			}
		}
		// Initialize any placeholder property sources in the context environment.
        // 初始化上下文环境中的任何占位符属性源
		initPropertySources();
		// Validate that all properties marked as required are 
         resolvable:seeConfigurablePropertyResolver#setRequiredProperties
        // 验证所有标记为必需的属性是否可解析：参见 ConfigurablePropertyResolver#setRequiredProperties
		getEnvironment().validateRequiredProperties();
		// Store pre-refresh ApplicationListeners...
        // 存储预刷新 ApplicationListeners
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}
		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}
```
第二步：刷新bean工厂准备beanfactory
```
// Tell the subclass to refresh the internal bean factory.
   告诉子类刷新内部bean工厂
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
// Prepare the bean factory for use in this context
    准备 bean 工厂以供在此上下文中使用
prepareBeanFactory(beanFactory);
```
第三步：允许在上下文子类中对 bean 工厂进行后处理
```
// Allows post-processing of the bean factory in context subclasses.
postProcessBeanFactory(beanFactory);
```
第四步：调用在上下文中注册为 bean 的工厂处理器
```
// Invoke factory processors registered as beans in the context.
invokeBeanFactoryPostProcessors(beanFactory);
```
第五步：注册拦截 bean 创建的 bean 处理器
```
// Register bean processors that intercept bean creation.
registerBeanPostProcessors(beanFactory);
beanPostProcess.end();
```
第六步：初始化此上下文的消息源
```
// Initialize message source for this context.
initMessageSource();
```
第七步：为该上下文初始化事件多播器
```
// Initialize event multicaster for this context.
initApplicationEventMulticaster();
```
第八步：初始化特定上下文子类中的其他特殊 bean
```
// Initialize other special beans in specific context subclasses.
onRefresh()
```
第九步：检查监听器 bean 并注册它们
```
// Check for listener beans and register them.
registerListeners();
```
第十步：实例化所有剩余的（非延迟初始化）单例
```
// Instantiate all remaining (non-lazy-init) singletons.
finishBeanFactoryInitialization(beanFactory);
```
十一步：最后一步，发布相应事件。
```
// Last step: publish corresponding event.
finishRefresh();
```
现在来看看bean的具体初始化，第十步
```
方法注释是 完成此上下文的 bean 工厂的初始化，初始化所有剩余的单例 bean
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Register a default embedded value resolver if no BeanFactoryPostProcessor
		// (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
		// at this point, primarily for resolution in annotation attribute values.
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			try {
				beanFactory.getBean(weaverAwareName, LoadTimeWeaverAware.class);
			}
			catch (BeanNotOfRequiredTypeException ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to initialize LoadTimeWeaverAware bean '" + weaverAwareName +
							"' due to unexpected type mismatch: " + ex.getMessage());
				}
			}
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// Allow for caching all bean definition metadata, not expecting further changes.
		beanFactory.freezeConfiguration();

		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();
	}

```
