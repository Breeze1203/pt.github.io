---
title: "SpringBootApplication启动流程一（环境判断，REACTIVE，NONE，SERVLET）"
date: 2024-11-18 15:41:52
---

- 初始化基本流程

- SpringApplication.run

- SpringApplication构造方法

- WebApplicationType的deduceFromClasspath推断Web应用程序类型

- ClassUtils的isPresent是否存在类型

  - ClassUtils的forName

研究源码的目的其实是为了更好的使用，和更好的扩展，当然是为了实际项目服务，解决问题，不是为了看而看，只有真正了解了原理，你才可以用的得心应手，出了问题也不怕，万变不离其宗。

## **初始化基本流程**

<img src="https://cloud.cxykk.com/images/2024/1/27/951/1706320312734.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

## **SpringApplication.run**

我们从最简单的例子开始，就这些东西，我们看看`SpringApplication.run`到底做了什么事。  
<img src="https://cloud.cxykk.com/images/2024/1/27/951/1706320318645.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />  
内部还有`run`：  
<img src="https://cloud.cxykk.com/images/2024/1/27/952/1706320324620.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />  
这次是关键：  
<img src="https://cloud.cxykk.com/images/2024/1/27/952/1706320330773.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

### **SpringApplication构造方法**

<img src="https://cloud.cxykk.com/images/2024/1/27/952/1706320336597.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />  
首先创建一个集合存放我们传进去的类，然后推断应用程序的类型，是SERVLET，还是REACTIVE，然后获取很多jar包下的META-INF/spring.factories中的org.springframework.context.ApplicationContextInitializer属性的值和org.springframework.context.ApplicationListener属性的值，其实他们是接口，他们的值就是其实就是实现类，也就是说要获取这些接口的实现类，来做一些初始化工作，当然里面会做一些筛选，去重。然后推断出main方法的类，他用了一种很巧妙的方式，抛出一个异常，然后再方法栈里找有main方法的那个类，具体的细节后面会说。

``` java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        this.sources = new LinkedHashSet();
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = Collections.emptySet();
        this.isCustomEnvironment = false;
        this.lazyInitialization = false;
        this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
        this.applicationStartup = ApplicationStartup.DEFAULT;
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```

### **WebApplicationType的deduceFromClasspath推断Web应用程序类型**

要对照这些类看：  
<img src="https://cloud.cxykk.com/images/2024/1/27/952/1706320342282.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />  
这个逻辑我就不讲了，就是排他的，要么`REACTIVE`，要么`SERVLET`，要么就普通的。具体是根据`Class.forName`反射的，而且如果一次不行，还会进行内部类的反射，否没有的话才捕获异常，返回`false`。可以看到如果同时有`REACTIVE`和`SERVLET`的相关类，会判定是`SERVLET`。

``` java
static WebApplicationType deduceFromClasspath() {
        if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.servlet.ServletContainer", (ClassLoader)null)) {
            return REACTIVE;
        } else {
            String[] var0 = SERVLET_INDICATOR_CLASSES;
            int var1 = var0.length;

            for(int var2 = 0; var2 < var1; ++var2) {
                String className = var0[var2];
                if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                    return NONE;
                }
            }

            return SERVLET;
        }
    }
```

#### **ClassUtils的isPresent是否存在类型**

``` java
   public static boolean isPresent(String className, @Nullable ClassLoader classLoader) {
        try {
            forName(className, classLoader);
            return true;
        } catch (IllegalAccessError var3) {
            throw new IllegalStateException("Readability mismatch in inheritance hierarchy of class [" + className + "]: " + var3.getMessage(), var3);
        } catch (Throwable var4) {
            return false;
        }
    }
```

#### **ClassUtils的forName(底层也是用的反射)**

留下了主要的代码，`Class.forName`，都没有就抛异常。

``` java
public static Class<?> forName(String name, @Nullable ClassLoader classLoader) throws ClassNotFoundException, LinkageError {
        Assert.notNull(name, "Name must not be null");
        Class<?> clazz = resolvePrimitiveClassName(name);
        if (clazz == null) {
            clazz = (Class)commonClassCache.get(name);
        }

        if (clazz != null) {
            return clazz;
        } else {
            Class elementClass;
            String elementName;
            if (name.endsWith("[]")) {
                elementName = name.substring(0, name.length() - "[]".length());
                elementClass = forName(elementName, classLoader);
                return Array.newInstance(elementClass, 0).getClass();
            } else if (name.startsWith("[L") && name.endsWith(";")) {
                elementName = name.substring("[L".length(), name.length() - 1);
                elementClass = forName(elementName, classLoader);
                return Array.newInstance(elementClass, 0).getClass();
            } else if (name.startsWith("[")) {
                elementName = name.substring("[".length());
                elementClass = forName(elementName, classLoader);
                return Array.newInstance(elementClass, 0).getClass();
            } else {
                ClassLoader clToUse = classLoader;
                if (classLoader == null) {
                    clToUse = getDefaultClassLoader();
                }

                try {
                    return Class.forName(name, false, clToUse);
                } catch (ClassNotFoundException var9) {
                    int lastDotIndex = name.lastIndexOf(46);
                    if (lastDotIndex != -1) {
                        String var10000 = name.substring(0, lastDotIndex);
                        String nestedClassName = var10000 + "$" + name.substring(lastDotIndex + 1);

                        try {
                            return Class.forName(nestedClassName, false, clToUse);
                        } catch (ClassNotFoundException var8) {
                        }
                    }

                    throw var9;
                }
            }
        }
    }
```

啰嗦的有点多，不好意思了，下次补上。

好了，今天就到这里了，希望对学习理解有帮助，大神看见勿喷，仅为自己的学习理解，能力有限
