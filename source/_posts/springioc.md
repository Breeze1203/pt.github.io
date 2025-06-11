---
title: "springioc"
date: 2025-06-11 19:23:39
categories: spring
tags: spring
---

# SpringIoc

#### **<span color="" fontsize="">Spring IOC 容器 Bean 对象实例化模拟</span>**

<span color="" fontsize="">思路:</span>

1.  <span color="" fontsize="">定义Bean 工厂接口，提供获取bean方法</span>

2.  <span color="" fontsize="">定义Bean工厂接口实现类，解析配置文件，实例化Bean对象</span>

3.  <span color="" fontsize="">实现获取Bean方法</span>

##### **<span color="" fontsize="">定义 Bean 属性对象</span>**

``` java
package com.xxxx.spring;

/**
 * bean对象
 *      用来接收配置文件中bean标签的id与class属性值
 */
public class MyBean {

    private String id; // bean对象的id属性值
    private String clazz; // bean对象的类路径

    public MyBean() {
    }

    public MyBean(String id, String clazz) {
        this.id = id;
        this.clazz = clazz;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getClazz() {
        return clazz;
    }

    public void setClazz(String clazz) {
        this.clazz = clazz;
    }
}
```

##### **<span color="" fontsize="">添加 dom4j 坐标依赖</span>**

``` xml
<!-- dom4j -->
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
<!-- XPath -->
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>1.1.6</version>
</dependency>
```

##### **<span color="" fontsize="">准备自定义配置文件</span>**<span color="" fontsize="">spring.xml</span>

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<beans>
    <bean id="userService" class="com.xxxx.service.UserService"></bean>
    <bean id="accountService" class="com.xxxx.service.AccountService"></bean>
</beans>
```

##### **<span color="" fontsize="">定义 Bean 工厂接口</span>**

``` java
package com.xxxx.spring;

/**
 * Bean 工厂接口定义
 */
public interface MyFactory {
    // 通过id值获取对象
    public Object getBean(String id);
}
```

##### **<span color="" fontsize="">定义 Bean 接口的实现类</span>**

``` java
package com.xxxx.spring;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.XPath;
import org.dom4j.io.SAXReader;
import java.net.URL;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 模拟Spring的实现
 *  1、通过构造器得到相关配置文件
 *  2、通过dom4j解析xml文件，得到List   存放id和class
 *  3、通过反射实例化得到对象   Class.forName(类的全路径).newInstance(); 通过Map<id,Class>存储
 *  4、得到指定的实例化对象
 */
public class MyClassPathXmlApplicationContext implements BeanFactory {

    private Map beans = new HashMap(); // 实例化后的对象放入map
    private List<MyBean> myBeans; // 存放已读取bean 配置信息

    /* 1、通过构造器得到相关配置文件 */
    public MyClassPathXmlApplicationContext(String fileName) {

        /* 2、通过dom4j解析xml文件，得到List （存放id和class） */
        this.parseXml(fileName);

        /* 3、通过反射实例化得到对象Class.forName(类路径).newInstance();  通过Map存储 */
        this.instanceBean();

    }

    /**
     * 通过dom4j解析xml文件，得到List   存放id和class
     *  1、获取解析器
     *  2、得到配置文件的URL
     *  3、通过解析器解析xml文件（spring.xml）
     *  4、通过xpath语法，获取beans标签下的所有bean标签
     *  5、通过指定语法解析文档对象，返回集合
     *  6、判断集合是否为空，遍历集合
     *  7、获取标签元素中的属性
     *  8、得到Bean对象，将Bean对象设置到集合中
     * @param fileName
     */
    private void parseXml(String fileName) {
        // 1、获取解析器
        SAXReader reader = new SAXReader();
        // 2、得到配置文件的URL
        URL url = this.getClass().getClassLoader().getResource(fileName);
        try {
            // 3、通过解析器解析xml文件（spring.xml）
            Document document = reader.read(url);
            // 4、通过xpath语法，获取beans标签下的所有bean标签
            XPath xPath = document.createXPath("beans/bean");
            // 通过指定语法解析文档对象，返回集合
            List<Element> list = xPath.selectNodes(document);
            // 判断集合是否为空，遍历集合
            if (list != null && list.size() > 0) {
                myBeans = new ArrayList<>();
                for(Element el : list) {
                    // 获取标签元素中的属性
                    String id = el.attributeValue("id"); // id 属性值
                    String clazz = el.attributeValue("class"); // class 属性值
                    System.out.println(el.attributeValue("id"));
                    System.out.println(el.attributeValue("class"));
                    // 得到Bean对象
                    MyBean bean = new MyBean(id, clazz);
                    // 将Bean对象设置到集合中
                    myBeans.add(bean);
                }
            }
        } catch (DocumentException e) {
            e.printStackTrace();
        }
    }

    /**
     * 通过反射实例化得到对象  
     *  Class.forName(类的全路径).newInstance();  
     *  通过Map<id,Class>存储
     */
    private void instanceBean() {
        // 判断bean集合是否为空，不为空遍历得到对应Bean对象
        if (myBeans != null && myBeans.size() > 0) {
            for (MyBean bean : myBeans){                                      
                try {
                    // 通过类的全路径实例化对象
                    Object object = Class.forName(bean.getClazz()).newInstance();
                    // 将id与实例化对象设置到map对象中
                    beans.put(bean.getId(), object);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * 通过key获取map中的指定value
     * @param id
     * @return
     */
    @Override
    public Object getBean(String id) {
        Object object = beans.get(id);
        return object;
    }
}
```

##### **<span color="" fontsize="">测试自定义 IOC 容器</span>**

<span color="" fontsize="">创建与配置文件中对应的Bean对象</span>

<span color="" fontsize="">UserService.java</span>

``` java
package com.xxxx.service;
 
public class UserService {
 
    public void test(){
        System.out.println("UserService Test...");
    }
}
```

<a href="http://AccountService.java" rel="noopener noreferrer nofollow" target="_blank"><span style="font-size: 16px; color: rgb(51, 51, 51)">AccountService.java</span></a>

``` java
package com.xxxx.service;

public class AccountService {

    public void test(){
        System.out.println("AccountService Test...");
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">2.测试是否可以获取实例化的Bean对象</span>

``` java
package com.xxxx;

import com.xxxx.spring.MyFactory;
import com.xxxx.spring.MyClassPathXmlApplicationContext;
import com.xxxx.service.AccountService;
import com.xxxx.service.UserService;

public class App {
    
    public static void main(String[] args) {
        MyFactory factory = new MyClassPathXmlApplicationContext("spring.xml");
        // 得到实例化对象
        UserService userService = (UserService) factory.getBean("userService");
        userService.test();

        UserService userService2 = (UserService) factory.getBean("userService");
        System.out.println(userService+"=====" + userService2);


        AccountService accountService = 
        (AccountService)factory.getBean("accountService");
        accountService.test();

    }
}
```

    Spring 容器在启动的时候 读取xml配置信息，并对配置的 bean 进行实例化（这里模拟的比较简单，仅用于帮助大家理解），同时通过上下文对象提供的 getBean() 方法拿到我们配置的 bean 对象，从而实现外部容器自动化维护并创建 bean 的效果

#### **<span color="" fontsize="">Spring IOC 容器 Bean 对象实例化</span>**

##### 1.**<span color="" fontsize="">构造器实例化</span>**

<span color="" fontsize="">注：</span>**<span color="" fontsize="">通过默认构造器创建 空构造方法必须存在 否则创建失败</span>**

1.  <span color="" fontsize="">设置配置文件 spring.xml</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.xxxx.service.UserService"></bean>

</beans>
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">2.获取实例化对象</span>

``` java
ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
UserService userService = (UserService) ac.getBean("userService");  
userService.test();
```

##### **<span color="" fontsize="">2.实例工厂实例化</span>**

- <span color="" fontsize="">要有该工厂类及工厂方法</span>

- <span color="" fontsize="">工厂方法为静态的</span>

<span style="font-size: 16px; color: rgb(51, 51, 51)">1.定义工厂类</span>

``` java
package com.xxxx.factory;

import com.xxxx.service.UserService;

/**
 * 定义工厂类
 */
public class InstanceFactory {
    /**
     * 定义方法，返回实例化对象
     * @return
     */
    public UserService createUserService() {
        return new UserService();
    }
}
```

2.<span style="font-size: 16px; color: rgb(51, 51, 51)">设置配置文件 spring.xml</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        实例化工厂
            1.定义实例化工厂bean
            2.引用工厂bean 指定工厂创建方法(方法为非静态)
    -->
    <bean id="instanceFactory" class="com.xxxx.factory.InstanceFactory"></bean>
    <bean id="userService" factory-bean="instanceFactory" factory-method="createUserService"></bean>

</beans>
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">3.获取实例化对象</span>

``` java
ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
UserService userService = (UserService) ac.getBean("userService");  
userService.test();
```

3.静态工厂实例化

- <span color="" fontsize="">工厂方法为非静态方法</span>

- <span color="" fontsize="">需要配置工厂bean，并在业务bean中配置factory-bean，factory-method属性</span>

1.<span style="font-size: 16px; color: rgb(51, 51, 51)">定义工厂类</span>

``` java
package com.xxxx.factory;

import com.xxxx.service.UserService;

/**
 * 定义工厂类
 */
public class InstanceFactory {
    /**
     * 定义方法，返回实例化对象
     * @return
     */
    public UserService createUserService() {
        return new UserService();
    }
}
```

2.<span style="font-size: 16px; color: rgb(51, 51, 51)">设置配置文件 spring.xml</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
        实例化工厂
            1.定义实例化工厂bean
            2.引用工厂bean 指定工厂创建方法(方法为非静态)
    -->
    <bean id="instanceFactory" class="com.xxxx.factory.InstanceFactory"></bean>
    <bean id="userService" factory-bean="instanceFactory" factory-method="createUserService"></bean>

</beans>
```

3.<span style="font-size: 16px; color: rgb(51, 51, 51)">获取实例化对象</span>

``` java
ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
UserService userService = (UserService) ac.getBean("userService");  
userService.test();
```

#### **<span color="" fontsize="">Spring三种实例化Bean的方式比较</span>**

- <span color="" fontsize="">方式一：</span>**<span color="" fontsize="">通过bean的缺省构造函数创建</span>**<span color="" fontsize="">，当各个bean的业务逻辑相互比较独立的时候或者和外界关联较少的时候可以使用。</span>

- <span color="" fontsize="">方式二：利用静态factory方法创建，可以统一管理各个bean的创建，如各个bean在创建之前需要相同的初始化处理，则可用这个factory方法险进行统一的处理等等。</span>

- <span color="" fontsize="">方式三：利用实例化factory方法创建，即将factory方法也作为了业务bean来控制，1可用于集成其他框架的bean创建管理方法，2能够使bean和factory的角色互换。</span>

<span color="" fontsize="">​ </span>**<span color="" fontsize="">开发中项目一般使用一种方式实例化bean，项目开发基本采用第一种方式，交给Spring托管，使用时直接拿来使用即可。另外两种了解</span>**
