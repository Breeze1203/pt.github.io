---
title: "Spring Ioc注入"
date: 2025-06-11 19:23:41
categories: spring
tags: spring
---

### SpringIoc注入

#### **Spring IOC 手动装配（注入）**

##### Spring 支持的注入方式共有四种：set 注入、构造器注入、静态工厂注入、实例化工厂注入。

#### **set方法注入**

注：

- 属性字段需要提供set方法

- 四种方式，推荐使用set方法注入

##### **业务对象 JavaBean**

1.  属性字段提供set方法

``` java
public class UserService {

    // 业务对象UserDao set注入（提供set方法）
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

    public class UserService {

        // 业务对象UserDao set注入（提供set方法）
        private UserDao userDao;
        public void setUserDao(UserDao userDao) {
            this.userDao = userDao;
        }
    }

<span style="font-size: 16px; color: rgb(51, 51, 51)">配置文件的bean标签设置property标签</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    
   <!--
        IOC通过property标签手动装配（注入）：
            Set方法注入
                name：bean对象中属性字段的名称
                ref：指定bean标签的id属性值
    --> 
    <bean id="userDao" class="com.xxxx.dao.UserDao"></bean>
    <bean id="userService" class="com.xxxx.service.UserService">
        <!--业务对象 注入-->
        <property name="userDao" ref="userDao"/>
    </bean>
</beans>
```

##### **常用对象和基本类型**

<span style="font-size: 16px; color: rgb(51, 51, 51)">1.属性字段提供set方法</span>

``` java
public class UserService {

    // 常用对象String  set注入（提供set方法）
    private String host;
    public void setHost(String host) {
        this.host = host;
    }

    // 基本类型Integer   set注入（提供set方法）
    private Integer port;
    public void setPort(Integer port) {
        this.port = port;
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">2.配置文件的bean标签设置property标签</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    
   <!--
        IOC通过property标签手动装配（注入）：
            Set方法注入
                name：bean对象中属性字段的名称
                value:具体的值（基本类型 常用对象|日期  集合）
    --> 
    <bean id="userService" class="com.xxxx.service.UserService">
        <!--常用对象String 注入-->
        <property name="host" value="127.0.0.1"/>
        <!--基本类型注入-->
        <property name="port" value="8080"/>
    </bean>

</beans>
```

##### **集合类型和属性对象**

<span style="font-size: 16px; color: rgb(51, 51, 51)">1.属性字段提供set方法</span>

``` java
public class UserService {

    // List集合  set注入（提供set方法）
    public List<String> list;
    public void setList(List<String> list) {
        this.list = list;
    }
   

    // Set集合  set注入（提供set方法）
    private Set<String> set;
    public void setSet(Set<String> set) {
        this.set = set;
    }


    // Map set注入（提供set方法）
    private Map<String,Object> map;
    public void setMap(Map<String, Object> map) {
        this.map = map;
    }
    

    // Properties set注入（提供set方法）
    private Properties properties;
    public void setProperties(Properties properties) {
        this.properties = properties;
    }
   
}
```

2.<span style="font-size: 16px; color: rgb(51, 51, 51)">配置文件的bean标签设置property标签</span>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    
   <!--
        IOC通过property标签手动装配（注入）：
            Set方法注入
                name：bean对象中属性字段的名称
                value:具体的值（基本类型 常用对象|日期  集合）
    --> 
    <!--List集合 注入-->
    <property name="list">
        <list>
            <value>上海</value>
            <value>北京</value>
            <value>杭州</value>
        </list>
    </property>

    <!--Set集合注入-->
    <property name="set">
        <set>
            <value>上海SH</value>
            <value>北京BJ</value>
            <value>杭州HZ</value>
        </set>
    </property>

    <!--Map注入-->
    <property name="map">
        <map>
            <entry>
                <key><value>周杰伦</value></key>
                <value>我是如此相信</value>
            </entry>
            <entry>
                <key><value>林俊杰</value></key>
                <value>可惜没如果</value>
            </entry>
            <entry>
                <key><value>陈奕迅</value></key>
                <value>十年</value>
            </entry>
        </map>
    </property>

    <!--Properties注入-->
    <property name="properties">
        <props>
            <prop key="上海">东方明珠</prop>
            <prop key="北京">天安门</prop>
            <prop key="杭州">西湖</prop>
        </props>
    </property>

</beans>
```

##### **多个Bean对象作为参数**

``` java
public class UserService {

    private UserDao userDao;  // JavaBean 对象
    private AccountDao accountDao  // JavaBean 对象
        
    public UserService(UserDao userDao, AccountDao accountDao) {
        this.userDao = userDao;
        this.accountDao = accountDao;
    }

    public  void  test(){
        System.out.println("UserService Test...");

        userDao.test();
        accountDao.test();
    }

}
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
     <!--
        IOC通过构造器注入：
            通过constructor-arg标签进行注入
                name：属性名称
                ref：指定bean标签的id属性值
    -->
    <bean id="userDao" class="com.xxxx.dao.UserDao" ></bean>
    <bean id="accountDao" class="com.xxxx.dao.AccountDao" ></bean>
    
    <bean id="userService" class="com.xxxx.service.UserService">
        <constructor-arg name="userDao" ref="userDao"></constructor-arg> 
        <constructor-arg name="accountDao" ref="accountDao"></constructor-arg>
    </bean>

</beans>
```

##### **Bean对象和常用对象作为参数**

``` java
public class UserService {

    private UserDao userDao;  // JavaBean 对象
    private AccountDao accountDao;  // JavaBean 对象
    private String uname;  // 字符串类型
        
    public UserService(UserDao userDao, AccountDao accountDao, String uname) {
        this.userDao = userDao;
        this.accountDao = accountDao;
        this.uname = uname;
    }

    public  void  test(){
        System.out.println("UserService Test...");

        userDao.test();
        accountDao.test();
        System.out.println("uname：" + uname);
    }

}
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--
        IOC通过构造器注入：
            通过constructor-arg标签进行注入
                name：属性名称
                ref：指定bean标签的id属性值
                value：基本类型 常用对象的值
                index：构造器中参数的下标，从0开始
    -->
    <bean id="userDao" class="com.xxxx.dao.UserDao" ></bean>
    <bean id="accountDao" class="com.xxxx.dao.AccountDao" ></bean>
    <bean id="userService" class="com.xxxx.service.UserService">
        <constructor-arg name="userDao" ref="userDao"></constructor-arg> 
        <constructor-arg name="accountDao" ref="accountDao"></constructor-arg>
        <constructor-arg name="uname" value="admin"></constructor-arg>
    </bean>

</beans>
```

##### **循环依赖问题**

Bean通过构造器注入，之间彼此相互依赖对方导致bean无法实例化

**问题展示：**

``` java
public class AccountService {

    private RoleService roleService;

   public AccountService(RoleService roleService) {
        this.roleService = roleService;
    }

    public void  test() {
        System.out.println("AccountService Test...");
    }
}

public class RoleService {

    private AccountService accountService;

   public RoleService(AccountService accountService) {
        this.accountService = accountService;
    }

    public void  test() {
        System.out.println("RoleService Test...");
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">XML配置</span>

``` xml
<!--
     如果多个bean对象中互相注入，则会出现循环依赖的问题
     可以通过set方法注入解决
-->
<bean id="accountService" class="com.xxxx.service.AccountService">
    <constructor-arg name="roleService" ref="roleService"/>
</bean>

<bean id="roleService" class="com.xxxx.service.RoleService">
    <constructor-arg name="accountService" ref="accountService"/>
</bean>
```

**如何解决：将构造器注入改为set方法注入**

``` java
public class AccountService {

    private RoleService roleService;

   /* public AccountService(RoleService roleService) {
        this.roleService = roleService;
    }*/

    public void setRoleService(RoleService roleService) {
        this.roleService = roleService;
    }

    public void  test() {
        System.out.println("AccountService Test...");
    }
}

public class RoleService {

    private AccountService accountService;

   /* public RoleService(AccountService accountService) {
        this.accountService = accountService;
    }*/

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    public void  test() {
        System.out.println("RoleService Test...");
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">XML配置</span>

``` xml
<!--
    <bean id="accountService" class="com.xxxx.service.AccountService">
    <constructor-arg name="roleService" ref="roleService"/>
    </bean>

    <bean id="roleService" class="com.xxxx.service.RoleService">
        <constructor-arg name="accountService" ref="accountService"/>
    </bean>
-->
<!--修改为set方法注入-->
<bean id="accountService" class="com.xxxx.service.AccountService">
    <property name="roleService" ref="roleService"/>
</bean>

<bean id="roleService" class="com.xxxx.service.RoleService">
    <property name="accountService" ref="accountService"/>
</bean>
```

#### **@Resource注解**

@Resource注解实现自动注入（反射）

- 默认根据属性字段名称查找对应的bean对象 （属性字段的名称与bean标签的id属性值相等）

- 如果属性字段名称未找到，则会通过类型（Class类型）查找

- 属性可以提供set方法，也可以不提供set方法

- 注解可以声明在属性级别 或 set方法级别

- 可以设置name属性，name属性值必须与bean标签的id属性值一致；如果设置了name属性值，就只会按照name属性值查找bean对象

- 当注入接口时，如果接口只有一个实现则正常实例化；如果接口存在多个实现，则需要使用name属性指定需要被实例化的bean对象

<span style="font-size: 16px; color: rgb(51, 51, 51)">1.默认根据属性字段名称查找对应的bean对象 （属性字段的名称与bean标签的id属性值相等）</span>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *  默认根据属性字段名称查找对应的bean对象 （属性字段的名称与bean标签的id属性值相等）
 */
public class UserService {

    @Resource
    private UserDao userDao; // 属性字段的名称与bean标签的id属性值相等

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">2.如果属性字段名称未找到，则会通过类型（Class类型）查找</span>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *   如果属性字段名称未找到，则会通过类型（Class类型）查找
 */
public class UserService {

    @Resource
    private UserDao ud; // 当在配置文件中属性字段名（ud）未找到，则会查找对应的class（UserDao类型）

    public void setUd(UserDao ud) {
        this.ud = ud;
    }

    public void test() {
        // 调用UserDao的方法
        ud.test();
    }
}
```

3.<span style="font-size: 16px; color: rgb(51, 51, 51)">属性可以提供set方法，也可以不提供set方法</span>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *   属性可以提供set方法，也可以不提供set方法
 */
public class UserService {

    @Resource
    private UserDao userDao; // 不提供set方法


    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">4.注解可以声明在属性级别 或 set方法级别</span>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *   注解可以声明在属性级别 或 set方法级别
 */
public class UserService {

    private UserDao userDao;

    @Resource // 注解也可设置在set方法上
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">可以设置name属性，name属性值必须与bean标签的id属性值一致；如果设置了name属性值，就只会按照name属性值查找bean对象</span>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *   可以设置name属性，name属性值必须与bean的id属性值一致；
 *   如果设置了name属性值，就只会按照name属性值查找bean对象
 */
public class UserService {

    @Resource(name = "userDao") // name属性值与配置文件中bean标签的id属性值一致
    private UserDao ud;


    public void test() {
        // 调用UserDao的方法
        ud.test();
    }
}
```

当注入接口时，如果接口只有一个实现则正常实例化；如果接口存在多个实现，则需要使用name属性指定需要被实例化的bean对象

定义接口类 <a href="http://IUserDao.java" rel="noopener noreferrer nofollow" target="_blank">IUserDao.java</a>

``` java
/**
 * 接口实现类
 */
public class UserDao02 implements IUserDao {

    @Override
    public void test(){
        System.out.println("UserDao02...");
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">XML配置文件</span>

``` xml
<!--开启自动化装配（注入）-->
<context:annotation-config/>

<bean id="userService" class="com.xxxx.service.UserService"></bean>

<bean id="userDao01" class="com.xxxx.dao.UserDao01"></bean>
<bean id="userDao02" class="com.xxxx.dao.UserDao01"></bean>
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">使用注解 </span><a href="http://UserService.java" rel="noopener noreferrer nofollow" target="_blank"><span style="font-size: 16px; color: rgb(51, 51, 51)">UserService.java</span></a>

``` java
/**
 * @Resource注解实现自动注入（反射）
 *   当注入接口时，如果接口只有一个实现则正常实例化；如果接口存在多个实现，则需要使用name属性指定需要被实例化的bean对象
 */
public class UserService {

    @Resource(name = "userDao01") // name属性值与其中一个实现类的bean标签的id属性值一致
    private IUserDao iUserDao; // 注入接口（接口存在多个实现）

    public void test() {
        iUserDao.test();
    }
}
```

#### **@Autowired注解**

@Autowired注解实现自动化注入：

- 默认通过类型（Class类型）查找bean对象 与属性字段的名称无关

- 属性可以提供set方法，也可以不提供set方法

- 注解可以声明在属性级别 或 set方法级别

- 可以添加@Qualifier结合使用，通过value属性值查找bean对象（value属性值必须要设置，且值要与bean标签的id属性值对应）

<span style="font-size: 16px; color: rgb(51, 51, 51)">默认通过类型（Class类型）查找bean对象 与属性字段的名称无关</span>

``` java
/**
 * @Autowired注解实现自动化注入
 *  默认通过类型（Class类型）查找bean对象   与属性字段的名称无关
 */
public class UserService {

    @Autowired
    private UserDao userDao; // 默认通过类型（Class类型）查找bean对象  与属性字段的名称无关

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">属性可以提供set方法，也可以不提供set方法</span>

``` java
/**
 * @Autowired注解实现自动化注入
 *  属性可以提供set方法，也可以不提供set方法
 */
public class UserService {

    @Autowired
    private UserDao userDao; // 不提供set方法

    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">注解可以声明在属性级别 或 set方法级别</span>

``` java
/**
 * @Autowired注解实现自动化注入
 *  注解可以声明在属性级别 或 set方法级别
 */
public class UserService {

    private UserDao userDao; 

    @Autowired// 注解可以声明在set方法级别
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void test() {
        // 调用UserDao的方法
        userDao.test();
    }
}
```

<span style="font-size: 16px; color: rgb(51, 51, 51)">可以添加@Qualifier结合使用，通过value属性值查找bean对象（value属性值必须要设置，且值要与bean标签的id属性值对应）</span>

    /**
     * @Autowired注解实现自动化注入
     *  可以添加@Qualifier结合使用，通过value属性值查找bean对象
            value属性值必须要设置，且值要与bean标签的id属性值对应
     */
    public class UserService {

        @Autowired
        @Qualifier(value="userDao") // value属性值必须要设置，且值要与bean标签的id属性值对应
        private UserDao userDao;

        public void test() {
            userDao.test();
        }
    }

<span style="font-size: 16px; color: rgb(51, 51, 51)">推荐使用</span>**<span color="" fontsize="">@Resource</span>**<span style="font-size: 16px; color: rgb(51, 51, 51)"> 注解是属于J2EE的，减少了与Spring的耦合</span>
