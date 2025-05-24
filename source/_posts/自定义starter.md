---
title: "自定义starter"
date: 2024-11-18 15:41:52
---

## **SpringBoot3.x自定义封装starter实战**

#### 1.创建一个普通的maven项目；

#### 2.导入自动配置依赖

``` xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>3.2.0</version>
        </dependency>
```

#### 3.编写实体类信息

``` java
package org.pt.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "user")
public class User {
    private String name;
    private String email;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
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
                "name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                '}';
    }
}
```

@ConfigurationProperties<span style="font-size: 16px; color: rgb(13, 13, 13)"> 注解在 Spring Boot 中用于将外部配置绑定到 JavaBean。在这里，</span>prefix = "user"<span style="font-size: 16px; color: rgb(13, 13, 13)"> 表示以 </span>user<span style="font-size: 16px; color: rgb(13, 13, 13)"> 开头的配置将会绑定到 JavaBean 类的字段上</span>

#### <span style="font-size: 16px; color: rgb(13, 13, 13)">3.定义自动配置类</span>

``` java
package org.pt.config;

import org.pt.service.UserService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;



@Configuration
@ConditionalOnClass({User.class})
@EnableConfigurationProperties({User.class})
public class UserAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public User user(){
        return new User();
    }


}
```

@EnableConfigurationProperties({User.class})<span style="font-size: 16px; color: rgb(13, 13, 13)"> 表示告诉 Spring Boot 应该处理名为 </span>`User`<span style="font-size: 16px; color: rgb(13, 13, 13)"> 的类，并将其配置绑定到外部配置；</span>

@ConditionalOnMissingBean<span style="font-size: 16px; color: rgb(13, 13, 13)"> 是 Spring Boot 中的一个条件注解，它表示只有当容器中不存在某个特定类型的 bean 时，才会创建被注解的 bean</span>

@ConditionalOnClass<span style="font-size: 16px; color: rgb(13, 13, 13)"> 是 Spring Boot 中的一个条件注解，它表示只有当类路径上存在指定的类时，才会创建被注解的 bean 或配置</span>

#### 4.**创建imports配置文件，把需要自动装载的类配置上**

自动配置类必须放进下面的文件里才算自动配置类META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

#### <img src="/upload/截屏2024-04-11%2021.20.00.png" style="display: inline-block;width:100.0%;height:100.0%" />5.使用mvn package install打包项目

#### 6.在其他模块引入打包后的项目

#### <img src="/upload/截屏2024-04-11%2021.23.05.png" style="display: inline-block;width:100.0%;height:100.0%" />7.新模块中注入对象

``` java
package com.example.springbootconfiguration.controller;


import jakarta.annotation.Resource;
import org.pt.config.User;
import org.pt.service.UserService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Resource
    public User user;

    @GetMapping("/getUser")
    public String getUser(){
        return user.toString();
    }
}
```

#### <a href="http://8.application.properties" rel="noopener noreferrer nofollow" target="_blank">8.application.properties</a>文件中给user对象赋值

``` yaml
spring.application.name=spring-boot-configuration
user.email=3548297839@qq.com
user.age=18
user.name=pt
```

#### 9.启动项目访问

<img src="/upload/截屏2024-04-11%2021.26.41.png" style="display: inline-block;width:100.0%;height:100.0%" />

源码地址：<a href="https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/user-spring-boot-starter" rel="noopener noreferrer nofollow" target="_blank">https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/user-spring-boot-starter</a>
