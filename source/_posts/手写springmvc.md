---
title: 手写springmvc
date: 2025-01-02 16:24:34
categories: java
tags: 
 - java
---

#### 手写SpringMVC，理解工作原理
SpringMVC是Spring家族中的元老之一，它是一个基于MVC三层架构模式的Web应用框架，它的出现也一统了JavaWEB应用开发的项目结构，从而避免将所有业务代码都糅合在同一个包下的复杂情况。在该框架中通过把Model、View、Controller分离，如下：

M/Model模型：由service、dao、entity等JavaBean构成，主要负责业务逻辑处理。
V/View视图：负责向用户进行界面的展示，由jsp、html、ftl....等组成。
C/Controller控制器：主要负责接收请求、调用业务服务、根据结果派发页面。

SpringMVC贯彻落实了MVC思想，以分层工作的模式，把整个较为复杂的web应用拆分成逻辑清晰的几部分，从很大程度上也简化了开发工作，减少了团队协作开发时的出错几率。
   回想最初的servlet开发，或者说最初我们学习Java时，如稚子般的操作，当时也不会划分模块、划分包，所有代码一股脑的全都放在少数的几个包下。但不知从何时起，慢慢的，每当有一个新的项目需求出现时，我们都会先对其划分模块，再划分层次，SpringMVC这个框架已经让每位Java开发彻底将MVC思想刻入到了DNA中，无论是最初的单体开发，亦或是如今主流的分布式、微服务开发，相信大家都已经遵守着这个思想。

SpringMVC框架的设计，是以请求为驱动，围绕Servlet设计的，将请求发给控制器，然后通过模型对象，分派器来展示请求结果的视图。SpringMVC的核心类是DispatcherServlet，它是一个Servlet子类，顶层是实现的Servlet接口。

当然，此刻暂且避开其原理不谈，先回想最初的SpringMVC是如何使用的呢？一起来看看。
1.1、SpringMVC的使用方式
   对于SpringMVC框架的原生使用方式，估计大部分小伙伴都已经忘了，尤其是近些年SpringBoot框架的流行，由于其简化配置的特性，让我们几乎无需再关注最初那些繁杂的XML配置。

说到这块就引起了我早些年那些痛苦的回忆，在SpringBoot还未那么流行之前，几乎所有的配置都是基于XML来弄的，而且每当引入一个新的技术栈，都需要配置一大堆文件，比如Spring、SpringMVC、MyBatis、Shiro、Quartz、EhCache....，这个整合过程无疑是痛苦的。

但随着后续的SpringBoot流行，这些问题则无需开发者再关注，不过成也SpringBoot，败也SpringBoot，尤其是近几年新入行的Java程序员，正是由于未曾有过之前那种繁重的XML配置经历，因此对于application.yml中很多技术栈的配置项也并不是特别理解，项目开发中需要引入一个新的技术栈时，几乎靠在网上copy他人的配置信息，也就成了“知其然而不知其所以然”，这对后续想要深入研究底层也成了一道新的屏障。

就此打住，感慨也不多说了，咱们先来回忆回忆最初SpringMVC的使用方式：基于最普通的maven-web工程构建。

在使用SpringMVC框架时，一般会首先配置它的核心文件：springmvc-servlet.xml，如下：

```
xml 代码解读复制代码<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.3.xsd 
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc.xsd
```

    <!-- 通过context:component-scan元素扫描指定包下的控制器-->
    <!-- 扫描com.xxx.xxx及子孙包下的控制器(扫描范围过大，耗时)-->
    <context:component-scan base-package="com.xxx.controller"/>
    
    <!-- ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- viewClass需要在pom中引入两个包：standard.jar and jstl.jar -->
        <property name="viewClass"
                  value="org.springframework.web.servlet.view.JstlView"></property>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    
    <!-- 省略其他配置...... -->
    </beans>
在springmvc-servlet.xml这个核心配置文件中，最重要的其实是配置Controller类所在的路径，即包扫描的路径，以及配置一个视图解析器，主要用于解析请求成功之后的视图数据。
OK~，配置好了springmvc-servlet.xml文件后，紧接着我们会再修改maven-web项目核心文件web.xml中的配置项：
xml 代码解读复制代码

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>  
<!-- 再这里会添加一个SpringMVC的servlet配置项 -->
  <servlet>
  <!-- 首先指定SpringMVC核心控制器所在的位置 -->
    <servlet-name>SpringMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- DispatcherServlet启动时，从哪个文件中加载组件的初始化信息 -->
    <!--此参数可以不配置，默认值为：/WEB-INF/springmvc-servlet.xml-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/springmvc-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <!--web.xml 3.0的新特性，是否支持异步-->
    <!--<async-supported>true</async-supported>-->
  </servlet>
  <!-- 配置路由匹配规则，/ 代表匹配所有，类似于nginx的location规则 -->
  <servlet-mapping>
    <servlet-name>SpringMVC</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

修改web.xml中的配置时，主要就干了一件事情，也就是为SpringMVC添加了一对servlet的配置项，主要指定了几个值：

①指定了SpringMVC中DispatcherServlet类的全路径。
②指定DispatcherServlet初始化组件时，从哪个文件中加载组件的配置信息。
③配置了一条值为/的路由匹配规则，/代表所有请求路径都匹配。

经过上述配置后，服务器启动后，所有的请求都会根据配置好的路由规则，先去到DispatcherServlet中处理。
至此，大概的配置就弄好了，紧接着是在前面配置的com.xxx.controller包中编写对应的Controller类，如下：
java 代码解读复制代码package com.xxx.controller;

@Controller("/user")
public class UserController{
    // 省略......
}

一切就绪后，一般都会将WEB应用打成war包，然后放入到Tomcat中运行，而当Tomcat启动时，首先会找到对应的WEB程序，紧接着会去加载web.xml，加载web.xml时，由于前面在其中配置了DispatcherServlet，所以此时会先去加载DispatcherServlet，而加载这个类时，又会触发它的初始化方法，会调用initStrategies()方法对组件进行初始化，如下：
java 代码解读复制代码// DispatcherServlet类 → initStrategies()方法
protected void initStrategies(ApplicationContext context) {
        // 在这里面初始化SpringMVC工作时，需要用到的各大组件
        initMultipartResolver(context);
        initLocaleResolver(context);
        initThemeResolver(context);
        initHandlerMappings(context);
        initHandlerAdapters(context);
        initHandlerExceptionResolvers(context);
        initRequestToViewNameTranslator(context);
        initViewResolvers(context);
        initFlashMapManager(context);
    }

那初始化组件时，肯定需要一些加载一些对应的组件配置，这些配置信息从哪儿来呢？也就是根据我们指定的<init-param></init-param>配置项，读取之前的核心文件：springmvc-servlet.xml中所配置的信息，对各大组件进行初始化。

所以，当Tomcat启动成功后，SpringMVC的各大组件也会初始化完成。

当然，DispatcherServlet除开是SpringMVC的初始化构建器外，还是SpringMVC的组件调用器，因为前面在web.xml还配置了一条路由规则，所有的请求都会先进入DispatcherServlet中处理，那既然所有的请求都进入了这个类，此时究竟该如何分发请求，就可以任由SpringMVC调度了。

但SpringMVC内部究竟是如何调用各大组件对请求进行处理的，这就涉及到了本文开头抛出的面试题了，也就是SpringMVC的工作原理，接下来我们简单聊一聊。

二、SpringMVC工作原理详解
   在了解SpringMVC的工作原理之前，首先认识一些常用组件：

DispatcherServlet前端控制器：接收请求，响应结果，相当于转发器，是整个流程控制的中心，由它调用其它组件处理用户的请求，因此也可称为中央处理器。有了它之后，可以很大程度上减少其它组件之间的耦合度。


HandlerMapping处理映射器：主要负责根据请求路径查找Handler处理器，也就是根据用户的请求路径找到具体的Java方法，具体是如何找到的呢？是根据映射关系查找的，SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。


HandlerAdapter处理适配器：就是一个用于执行Handler处理器的组件，会根据客户端不同的请求方式（get/post/...），执行对应的Handler。说人话就是前面的组件定位到具体Java方法后，用来执行Java方法的组件。


Handler处理器：其实这也就是包含具体业务操作的Java方法，在SpringMVC中会被包装成一个Handler对象。


ViewResolver视图解析器：：对业务代码执行完成之后的结果进行视图解析，根据逻辑视图名解析成真正的视图，比如controller方法执行完成之后，return的值是index，那么会对这个结果进行解析，将结果生成例如index.jsp这类的View视图。
ViewResolver工作时，会首先根据逻辑视图名解析成物理视图名，即具体的页面地址，然后再生成View视图对象，最后对视图进行渲染，将处理结果通过页面展示给用户。
SpringMVC提供了很多的View视图类型，如：jstlView、freemarkerView、pdfView等，前面我们配置的JSP视图解析器则是JstlView，这里也可以根据模板引擎的不同，选择不同的解析器。


View视图：View在SpringMVC中是一个接口，实现类支持不同的类型，例如jsp、freemarker、ftl...，不过现在一般都是前后端分离的项目，因此也很少再用到这块内容，视图一般都成了html页面，数据结果的渲染工作也交给了前端完成。

大致对于SpringMVC的核心组件有了了解之后，再上一张图：

对于这张图，相信大家都多多少少有在“面试八股文”中看到过，这也是涵盖了SpringMVC内部调度时的完整流程图，请求到来后都会经过这一系列步骤，如下：

①用户发送请求至会先进入DispatcherServlet控制器进行相应处理。
②DispatcherServlet会调用HandlerMapping根据请求路径查找Handler。
③处理器映射器找到具体的处理器后，生成Handler对象及Handler拦截器（如果有则生成），然后返回给DispatcherServlet。
④DispatcherServlet紧接着会调用HandlerAdapter，准备执行Handler。
⑤HandlerAdapter底层会利用反射机制，对前面生成的Handler对象进行执行。
⑥执行完对应的Java方法后，HandlerAdapter会得到一个ModelAndView对象。
⑦HandlerAdapter将ModelAndView再返回给DispatcherServlet控制器。
⑧DisPatcherServlet再调用ViewReslover，并将ModelAndView传递给它。
⑨ViewReslover视图解析器开始解析ModelAndView并返回解析出的View视图。
⑩解析出View视图后，对视图进行数据渲染（即将模型数据填充至视图中）。
⑪DispatcherServlet最终将渲染好的View视图响应给用户浏览器。

其实观察如上流程，SpringMVC中的其他组件几乎不存在太多的耦合关系，大部分的工作都是由DispatcherServlet来调度组件完成的，因此这也是它被称为“中央控制器”的原因，DispatcherServlet本质上并不会处理用户请求，它仅仅是作为请求统一的访问点，负责请求处理时的全局流程控制。

当然，最开始由于我们在springmvc-servlet.xml中配置了扫包路径，因此在项目启动时，就会去扫描对应目录下的所有类，然后将带有对应注解的类与方法，与注解上指定的请求路径生成映射关系，方便后续请求到来时能够精准定位（稍后看完手写案例大家就理解这点了）。

经过上述一系列分析后会发现，SpringMVC的核心就是DispatcherServlet，由它去调用各类组件完成工作。而DispatcherServlet其实本质上就是一个Servlet子类，一般WEB层框架本质上都离不开Servlet，就好比ORM框架离不开JDBC，比如Zuul、GateWay等框架，本质上也是依赖于Servlet技术作为底层的。
三、手写Mini版SpringMVC框架
   到目前为止，相对来说已经将SpringMVC的工作原理做了简单概述，接下来就来到本文的核心：自己手写一个Mini版的SpringMVC框架。步骤主要分为五步：

①自定义相关注解。
②实现核心组件。
③实现DispatcherServlet。
④编写相关的视图（jsp网页）。
⑤编写测试用例。

不过在手写之前，咱们得先创建一个普通的Maven-Web工程。
3.1、自定义相关注解
   SpringMVC中的注解实际上并不少，所以在这里不会全部实现，重点就自定义@Controller、@RequestMapping、@ResponseBody这几个常用的核心注解。
3.1.1、@Controller注解的定义
java 代码解读复制代码// 声明注解的生命周期：RUNTIME表示运行时期有效
@Retention(RetentionPolicy.RUNTIME)
// 注解的生效范围：只能生效于类上面
@Target(ElementType.TYPE)
public @interface Controller {
    //@interface是元注解：JDK封装的专门用来实现自定义注解的注解
}

这个注解稍后会加载咱们要扫描的Controller类上，主要是为了标注出扫描时的目标类。
3.1.2、@RequestMapping注解的定义
java 代码解读复制代码// 声明注解的生命周期：RUNTIME表示运行时期有效
@Retention(RetentionPolicy.RUNTIME)
// 注解的生效范围：可应用在类上面、方法上面
@Target({ElementType.METHOD,ElementType.TYPE})
public @interface RequestMapping {
    // 允许该注解可以填String类型的参数，默认为空
    String value() default "";
}

这个注解可以加在类或方法上，主要是用来给类或方法映射请求路径。
3.1.3、@ResponseBody注解的定义
java 代码解读复制代码// 声明注解的生命周期：RUNTIME表示运行时期有效
@Retention(RetentionPolicy.RUNTIME)
// 注解的生效范围：只能应用在方法上面
@Target(ElementType.METHOD)
public @interface ResponseBody {
}

这个注解的作用是在于控制返回时的响应方式，不加该注解的方法，默认会跳转页面，也加了该注解的方法，则会直接响应数据。
OK~，在上面定义了三个注解，其中使用到了两个JDK提供的元注解：@Retention、@Target，前者用于控制注解的生命周期，表示自定义的注解在何时生效。后者则控制了注解的生效范围，可以控制自定义注解在类、方法、属性上生效。

不过在这里并未对这些注解进行处理，只是简单的定义，如果想要注解生效，一般有两种方式：①使用AOP切面对注解进行处理。②使用反射机制对注解进行处理。

稍后我们会采用上述的第二种方式对自定义的注解进行处理。
3.2、实现核心组件
自定义注解的工作完成后，紧接着再来实现一些运行时需要用到的核心组件。当然，这里也不会将之前SpringMVC拥有的所有组件全部实现，仅实现几个核心的组件，能够达到效果即可。（在完成之后，大家有兴趣可自行完善）。
3.2.1、InvocationHandler组件
InvocationHandler这个组件，主要是为了待会儿配合扫描包使用的，可以简单理解成Java方法的封装对象，如下：
java 代码解读复制代码

    public class InvocationHandler {
        // 这里会存放方法对应的对象实例
        private Object object;
        // 这里会存放对应的Java方法
        private Method method;
    // 构造方法：无参和全参构造
    public InvocationHandler(){}
    public InvocationHandler(Object object, Method method) {
        this.object = object;
        this.method = method;
    }
    
    // Get and Set方法
    public Object getObject() {
        return object;
    }
    public void setObject(Object object) {
        this.object = object;
    }
    public Method getMethod() {
        return method;
    }
    public void setMethod(Method method) {
        this.method = method;
    }
    
    // 这里重写了toString()方法
    @Override
    public String toString() {
        return "InvocationHandler{" +
                "object=" + object +
                ", method=" + method +
                '}';
    }
}

这个组件很简单，相信大家也能直接看明白，这也对应着之前SpringMVC中的Handler组件。
3.2.2、HandlerMapping组件
这个组件主要负责扫描包，在项目启动时，将指定的包目录下，所有的请求路径与Java方法形成映射关系。
typescript 代码解读复制代码

    public class HandlerMapping {
        public Map<String,InvocationHandler> urlMapping(Set<Class<?>> classSet){
            // 初始化一个 Map 集合，用于存放映射关系
            HashMap<String, InvocationHandler> HandlerHashMap = new HashMap<>();
            // 遍历 Controller 集合（也就是所有带@Controller注解的类）
            for (Class<?> aClass : classSet) {
                //获取类上@RequestMapping注解的值
                String classReqPath = AnnotationUtil.
                        getAnnotationValue(aClass, RequestMapping.class);
                System.out.println("类的请求路径:" + classReqPath);
            // 获取这个 class 类中的所有方法
            Method[] methods = aClass.getDeclaredMethods();
            System.out.println("类中方法数量为：" + methods.length);
    
            // 如果这个类中方法数量不为空
            if (methods.length != 0) {
                // 开始遍历这个类中的所有方法
                for (Method method : methods) {
                    // 判断每个方法上是否带有@RequestMapping注解
                    boolean flag = method.isAnnotationPresent(RequestMapping.class);
                    // 如果当前方法上带有这个注解
                    if (flag){
                        // 获取方法上@RequestMapping注解的值
                        String methodReqPath = AnnotationUtil.
                                getAnnotationValue(method, RequestMapping.class);
                        // 判断得到的值是否为空，不为空则获取对应的值
                        String reqPath = methodReqPath == null ||
                                methodReqPath.equals("") ? "" : methodReqPath;
                        System.out.println("方法上的请求路径:" + reqPath);
                        // 将得到的值封装成 InvocationHandler 对象
                        try {
                            // 放入一个当前类的实例对象，用于执行后面的类方法
                            InvocationHandler invocationHandler = new 
                                    InvocationHandler(aClass.newInstance(), method);
                            // 使用 类的请求路径 + 方法的请求路径 作为Key
                            HandlerHashMap.put(classReqPath + reqPath,
                                    invocationHandler);
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
        // 将存放映射关系的Map集合返回
        return HandlerHashMap;
    }
}

在这个类中，主要定义了一个urlMapping()方法，这个方法做的主要工作就是：对于所有存在@Controller注解的类做扫描，对于这些类中的方法进行判断，将所有带@RequestMapping注解的方法，全部封装成InvocationHandler对象作为Value，然后再以类的请求路径 + 方法的请求路径作为Key，放入到一个Map集合中保存。
3.3、实现DispatcherServlet中央控制器
自定义注解和组件的工作完成后，接下来再开始编写最核心的DispatcherServlet类，同样，在定义时记得继承HttpServlet：
java 代码解读复制代码public class DispacherServlet extends HttpServlet {

    // 定义一个 Map 容器，存储映射关系
    private static Map<String, InvocationHandler> HandlerMap;
    
    @Override
    public void init() throws ServletException {
        System.out.println("项目启动了.....");
        // 指定要扫描的包路径（原本是从xml文件中读取的）
        String packagePath = "com.xxx.controller";
        // 在指定的包路径下扫描带有@Controller注解的类
        Set<Class<?>> classSet = ClassUtil.
                scanPackageByAnnotation(packagePath, Controller.class);
        System.out.println("扫描到类的数量为:" + classSet.size());
        // 创建一个HandlerMapping并调用urlMapping()方法
        HandlerMapping handlerMapping = new HandlerMapping();
        HandlerMap = handlerMapping.urlMapping(classSet);
        // 最终获取到一个带有所有映射关系的 Map 集合
        System.out.println("HandlerMap的长度：" + HandlerMap.size());
    }
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doPost(req,resp);
    }
    
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // 获取客户端的请求路径
        StringBuffer requestURL = req.getRequestURL();
        System.out.println("客户端请求路径：" + requestURL);
        // 判断请求路径中是否包含项目名，包含的话使用空字符替换掉
        String path = new String(requestURL).replace("http://" +
                req.getServerName() + ":" + req.getServerPort(), "");
        System.out.println("处理后的客户端请求路径：" + path);
        // 根据处理好的 path 作为条件去map中查找对应的方法
        InvocationHandler handler = HandlerMap.get(path);
        // 获取到对应的类实例对象和Java方法
        Object object = handler.getObject();
        Method method = handler.getMethod();
    
        // 判断该方法上是否添加了@ResponseBody注解：
        //      true：直接返回数据  false：跳转页面
        boolean f = method.isAnnotationPresent(ResponseBody.class);
        System.out.println("是否添加了@ResponseBody注解：" + f);
        // 如果方法上存在@ResponseBody注解
        if (f){
            try {
                // 通过反射的方式调用方法并执行
                Object invoke = method.invoke(object);
                // 将结果通过Response直接写回给客户端
                resp.getWriter().print(invoke.toString());
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else{
            // 获取客户端的请求路径作为返回时的前路径
            String URL = "http://" + req.getServerName() + ":" +
                    req.getServerPort() + "/" + req.getContextPath();
            System.out.println("URL:" + URL);
            // 自定义的前后缀（原本也是在xml中读取）
            String prefix = "";
            String suffix = ".jsp";
            try {
                // 通过反射机制，执行对应的Java方法
                Object invoke = method.invoke(object);
                if(invoke instanceof ModelAndView){
                    // 如果是返回的ModelAndView对象，这里做额外处理....
                } else{
                    // 获取Java方法执行之后的返回结果
                    String str = (String)invoke;
                    // 如果指定了跳转方法为 forward: 转发
                    if(str.contains("forward:")){
                        System.out.println("以转发的方式跳转页面...");
                        req.getRequestDispatcher("index.jsp").forward(req,resp);
                    }
                    // 如果指定了跳转方法为 redirect: 重定向
                    if(str.contains("redirect:")){
                        System.out.println("以重定向的方式跳转页面...");
                        resp.sendRedirect(URL + prefix +
                            str.replace("redirect:","") + suffix);
                    }
                    // 如果没有指定，则默认使用转发的方式跳转页面
                    if(!str.contains("forward:") && !str.contains("redirect:")){
                        resp.sendRedirect(URL + prefix + str + suffix);
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    }
}

由于DispacherServlet实现了HttpServlet抽象类，因此也重写了它的三个方法：init()、doGet()、doPost()，其中init()方法会在项目启动时执行，而doGet()、doPost()则会在客户端请求时被触发。
总结一下上述DispacherServlet所做的工作：

①初始化所有请求路径与Java方法之间的映射关系。
②根据客户端的请求路径，查找对应的Java方法并执行。
③判断方法上是否添加了@ResponseBody注解：

添加了：直接向客户端返回数据。
未添加：跳转对应的页面。


④以重定向或转发的方式跳转对应的页面。

OK~，最后也不要忘了在web.xml配置一下我们自己的DispacherServlet：
xml 代码解读复制代码<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
    <servlet-name>dispacherServlet</servlet-name>
    <!-- 这里配置的DispacherServlet是我们自己的 -->
    <servlet-class>com.xxx.DispacherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>dispacherServlet</servlet-name>
    <!-- 匹配规则依旧是所有请求路径都会匹配 -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>

3.4、编写View视图
当然，不追求外观了，简单编写两个视图页面：index.jsp、edit.jsp：
html 代码解读复制代码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>

  <head>
    <title>首页</title>
    <link href="favicon.ico" rel="shortcut icon">
  </head>

  <body>
        <h1>欢迎来到熊猫高级会所，我是竹子一号！</h1>
  </body>
</html>

<!-- edit.jsp -->
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>

  <head>
    <title>修改</title>
    <link href="favicon.ico" rel="shortcut icon">
  </head>

  <body>
        <h1>修改页面</h1>
        <a href="#">跳转</a>
  </body>
</html>
```

3.5、编写测试用例
为了方便测试，先写一个实体类：User.java，如下：
java 代码解读复制代码

```java
public class User {
    private Integer id;
    private String name;
    private String sex;
    private Integer age;
public User(){}

public User(Integer id, String name, String sex, Integer age) {
    this.id = id;
    this.name = name;
    this.sex = sex;
    this.age = age;
}

public Integer getId() {
    return id;
}

public void setId(Integer id) {
    this.id = id;
}

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}

public String getSex() {
    return sex;
}

public void setSex(String sex) {
    this.sex = sex;
}

public Integer getAge() {
    return age;
}

public void setAge(Integer age) {
    this.age = age;
}

@Override
public String toString() {
    return "User{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", sex='" + sex + '\'' +
            ", age=" + age +
            '}';
}
}
```
这个实体类主要方便为了待会儿测试@ResponseBody注解的功能，接下来写两个Controller类：
java 代码解读复制代码/* ------ UserController类 ------- */ 
    

    @Controller
    @RequestMapping("/user")
    public class UserController {
    // 测试@ResponseBody的功效
    @RequestMapping("/get")
    @ResponseBody
    public User get(){
        return new User(1,"竹子爱熊猫","男",18);
    }
    
    // 跳转首页的方法
    @RequestMapping("/")
    public String test(){
        return "index";
    }
    
    // 测试重定向的功效
    @RequestMapping("/edit")
    public String toEdit(){
        return "redirect:edit";
    }
    
    public String TEST(){
        return null;
    }
}

/* ------OrderController类------- */ 
public class OrderController {
}

在上述测试案例中，编写了UserController、OrderController两个类，其中仅有UserController加了@Controller注解，下面来测试，首先将这个Maven工程打成war包部署在Tomcat中，然后启动，日志如下：
java 代码解读复制代码项目启动了.....
扫描到类的数量为:1
类的请求路径:/user
类中方法数量为：4
方法上的请求路径:/get
方法上的请求路径:/test
方法上的请求路径:/edit
HandlerMap的长度：3

从上述日志输出中，很明显可以看出，未添加@Controller注解的OrderController类并未被扫描，同时，UserController类中未添加@RequestMapping注解的TEST()方法，也没有被加入到HandlerMap集合中，该集合中仅存放了有映射关系的Java方法。
OK~，接下来使用浏览器测试我们手写的SpringMVC是否可以做到原本的效果：

测试首页跳转效果：http://localhost:80

 参考文章：https://juejin.cn/post/7139807630024769549
 源代码：https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-mvc-pt