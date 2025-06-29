---
title: "SpringAop"
date: 2025-06-11 19:23:40
categories: spring aop
tags: spring aop
---

## SpringAop

##### 注解配置方式

``` java
package com.xxxx.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 切面
 *      切入点和通知的抽象
 *      定义 切入点  和 通知
 *         切入点：定义要拦截哪些类的哪些方法
 *         通知：定义了拦截之后方法要做什么
 */
@Component // 将对象交给IOC容器进行实例化
@Aspect // 声明当前类是一个切面
public class LogCut {

    /**
     * 切入点
     *      定义要拦截哪些类的哪些方法
     *      匹配规则，拦截什么方法
     *
     *      定义切入点
     *          @Pointcut("匹配规则")
     *
     *      Aop切入点表达式
     *          1. 执行所有的公共方法
     *              execution(public *(..))
     *         2. 执行任意的set方法
     *              execution(* set*(..))
     *         3. 设置指定包下的任意类的任意方法 （指定包：com.xxxx.service）
     *              execution(* com.xxxx.service.*.*(..))
     *         4. 设置指定包及子包下的任意类的任意方法 （指定包：com.xxxx.service）
     *             execution(* com.xxxx.service..*.*(..))
     *
     *        表达式中的第一个*
     *              代表的是方法的修饰范围  （public、private、protected）
     *              如果取值是*，则表示所有范围
     */
    @Pointcut("execution(* com.xxxx.service..*.*(..))")
    public void cut(){

    }

    /**
     * 声明前置通知，并将通知应用到指定的切入点上
     *      目标类的方法执行前，执行该通知
     */
    @Before(value = "cut()")
    public void before() {
        System.out.println("前置通知...");
    }

    /**
     * 声明返回通知，并将通知应用到指定的切入点上
     *      目标类的方法在无异常执行后，执行该通知
     */
    @AfterReturning(value = "cut()")
    public void afterReturn(){
        System.out.println("返回通知...");
    }

    /**
     * 声明最终通知，并将通知应用到指定的切入点上
     *      目标类的方法在执行后，执行该通知 （有异常和无异常最终都会执行）
     */
    @After(value = "cut()")
    public void after(){
        System.out.println("最终通知...");
    }

    /**
     * 声明异常通知，并将通知应用到指定的切入点上
     *      目标类的方法在执行异常时，执行该通知
     */
    @AfterThrowing(value = "cut()", throwing = "e")
    public void afterThrow(Exception e){
        System.out.println("异常通知...  ===== 异常原因：" + e.getMessage());
    }


    /**
     * 声明环绕通知，并将通知应用到指定的切入点上
     *      目标类的方法执行前后，都可通过环绕通知定义响应的处理
     *          需要通过显式调用的方法，否则无法访问指定方法 pjp.proceed();
     * @param pjp
     * @return
     */
    @Around(value = "cut()")
    public Object around(ProceedingJoinPoint pjp)  {
        System.out.println("环绕通知-前置通知...");

        Object object = null;
        //object = pjp.proceed();

        try{
            // 显式调用对应的方法
            object = pjp.proceed();
            System.out.println(pjp.getTarget());
            System.out.println("环绕通知-返回通知...");
        } catch (Throwable throwable){
            throwable.printStackTrace();
            System.out.println("环绕通知-异常通知...");
        }
        System.out.println("环绕通知-最终通知...");

        return object;
    }



}
```

##### xml文件配置方式

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

        <!-- 开启自动化扫描 -->
        <context:component-scan base-package="com.xxxx"/>

        <!-- aop 相关配置 -->
        <aop:config>
                <!-- aop 切面 -->
                <aop:aspect ref="logCut02">
                        <!-- 定义 aop 切入点 -->
                        <aop:pointcut id="cut" expression="execution(* com.xxxx.service..*.*(..))"/>
                        <!-- 配置前置通知，设置前置通知对应的方法名及切入点 -->
                        <aop:before method="before" pointcut-ref="cut"/>
                        <!-- 配置返回通知，设置返回通知对应的方法名及切入点 -->
                        <aop:after-returning method="afterReturn" pointcut-ref="cut"/>
                        <!-- 配置最终通知，设置最终通知对应的方法名及切入点 -->
                        <aop:after method="after" pointcut-ref="cut"/>
                        <!-- 配置异常通知，设置异常通知对应的方法名及切入点 -->
                        <aop:after-throwing method="afterThrow" pointcut-ref="cut" throwing="e" />
                        <!-- 配置环绕通知，设置环绕通知对应的方法名及切入点 -->
                        <aop:around method="around" pointcut-ref="cut"/>
                </aop:aspect>
        </aop:config>

</beans>
```

源代码 <a href="https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-aop-demo" rel="noopener noreferrer nofollow" target="_blank">https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-aop-demo</a>
