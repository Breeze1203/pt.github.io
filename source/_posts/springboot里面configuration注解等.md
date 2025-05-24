---
title: "SpringBoot里面@Configuration注解等"
date: 2024-11-18 15:41:52
---

### SpringBoot中的@Configuration注解

1.  **标识配置类**：使用 `@Configuration` 注解的类被标识为配置类，告诉 Spring 这个类是用来配置 Spring 应用程序上下文的。

2.  **定义 Bean**：在 `@Configuration` 注解的类中，我们可以使用 `@Bean` 注解定义 Bean 对象，Spring 容器会自动扫描这些 Bean，并将它们纳入管理。

3.  **代替 XML 配置**：通过使用注解配置类，可以避免传统 XML 配置文件的烦琐，使得配置更加简洁和易于维护。

4.  **组件扫描**：`@Configuration` 注解会隐式地向 Spring 容器注册一个 Bean，类型为该配置类本身，从而使 Spring 可以基于组件扫描找到其他 Bean。

5.  **配合其他注解使用**：`@Configuration` 注解通常与其他注解一起使用，比如 `@Bean`、`@ComponentScan`、`@Import` 等，来实现复杂配置或者注入相关 Bean

当一个类被该注解标记后，被标记为配置类，在里面定义的Bean对象会被纳入spring容器，交由容器管理

``` java
@Configuration

public class MainConfig {

    @Bean(name = "MainConfig")
    public MyBean myBean(){
        return new MyBean("MainConfig");
    }
}
```

当我们启动容器后，name为MainConfig的这个bean会被自定注入到spring容器，我们可以获取到；

``` java
ConfigurableApplicationContext run = SpringApplication.run(SpringbootStudyApplication.class, args);
        Object mainConfig = run.getBean("MainConfig");
        System.out.println(mainConfig.toString());
```

结果

<img src="/upload/截屏2024-04-08%2022.25.51.png" style="display: inline-block;width:100.0%;height:100.0%" />

**<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">导入额外的 Configuration 类，@Import注解</span>**

<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">我们先定义</span>**<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">一个</span>**MyBean类型<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">bean，这个方法不被@C Configuration标记,可以是不是会被自动纳入容器；</span>

<img src="/upload/截屏2024-04-08%2022.31.32.png" style="display: inline-block;width:100.0%;height:100.0%" />

启动容器，获取这个名称的bean

<img src="/upload/截屏2024-04-08%2022.32.13.png" style="display: inline-block;width:100.0%;height:100.0%" />运行结果

<img src="/upload/截屏2024-04-08%2022.32.39.png" style="display: inline-block;width:100.0%;height:100.0%" />可以看到这个bean没有被加入到spring容器

当我们在一个被@<span color="rgb(51, 51, 51)" fontsize="" style="color: rgb(51, 51, 51)">Configuration标记的类上面使用@import注解引入这个名为</span>AdditionalConfig1的bean，如下

``` java
@Configuration
@Import(AdditionalConfig1.class)
public class MainConfig {

    @Bean(name = "MainConfig")
    public MyBean myBean(){
        return new MyBean("MainConfig");
    }
}
```

再次启动容器查看

<img src="/upload/截屏2024-04-08%2022.35.56.png" style="display: inline-block;width:100.0%;height:100.0%" />可以看到，容器成功捕获到

**@Import<span style="font-size: 14px; color: rgb(36, 41, 47)">注解通常用于引入其他Java配置类，而不是XML文件。如果你想引入XML配置文件中的bean定义，通常会使用</span>@ImportResource<span style="font-size: 14px; color: rgb(36, 41, 47)">注解</span>**

假设你在resources下有一bean.xml文件，内容如下

<img src="/upload/截屏2024-04-08%2022.42.38.png" style="display: inline-block;width:100.0%;height:100.0%" />通过@ImportResource导入xml文件里面这个bean

<img src="/upload/截屏2024-04-08%2022.43.35.png" style="display: inline-block;width:100.0%;height:100.0%" />启动容器

<img src="/upload/截屏2024-04-08%2022.44.03.png" style="display: inline-block;width:100.0%;height:100.0%" />

可以看到这个bean也被引入进来
