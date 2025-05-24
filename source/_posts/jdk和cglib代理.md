---
title: "jdk和CGLIB代理"
date: 2024-11-18 15:41:52
layout: tags
categories: Java
tags: Java
---

#### 什么是代理？

代理（Proxy）是一种设计模式，它充当另一个对象的接口或占位符，以控制对该对象的访问。

静态代理是在<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">编译时就确定代理关系的代理模式</span>。在静态代理中，代理类和目标类在编译期间就确定了。代理类通常持有目标类的引用，并在调用目标方法前后进行一些额外的操作，例如记录日志、执行权限控制等。静态代理的缺点是代理类和目标类是一一对应的，如果需要代理的类很多，就需要编写大量的代理类。

动态代理是在<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">运行时动态生成代理类的代理模式</span>。在动态代理中，代理类的代码在运行时生成，而不是在编译时确定。Java 中的动态代理通常使用 Java 反射机制实现，通过动态生成字节码来创建代理类。动态代理的优点是可以减少代理类的数量，因为一个代理类可以代理多个目标类，从而简化了代码维护

#### 代理三要素

1.  目标角色(真实角色)

2.  代理角色(用来增强目标角色行为)

3.  实现共同接口

以租房子为例，租客(目标角色)，中介(代理角色)，租房子(共同行为)，在这里，租客和中介都能租房子，但中间似乎能够能实现的行为更多（这里就表示能增强租客行为）

<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">静态代理就是确定了租客是谁，动态代理就是我目前不知道租客是谁</span>

代码实现：

首先我们定义一个共同行为的接口(租房子)

``` java
/**
 * 定义接口 - 定义行为
 */
public interface RentHouse {

    public void toRentHouse();

}
```

租客(目标角色)实现这个接口

``` java
public class Owner implements RentHouse {
    @Override
    public void toRentHouse() {
        System.out.println("两室一厅，月租五千！");
    }
}
```

中介(代理角色)，首先，中介知道租客是谁

``` java
public class AgentProxy implements RentHouse {

    // 目标对象
    private RentHouse target;
    // 通过带参构造器获取目标对象
    public AgentProxy(RentHouse target) {
        this.target = target;
    }

    // 实现行为
    @Override
    public void toRentHouse() {
        // 用户增强行为
        System.out.println("房型朝南，采光好！");
        // 代理对象调用目标对象的方法
        target.toRentHouse();
        // 用户增强行为
        System.out.println("价格可议！");

    }
}
```

这里我们调用了目标角色的方法，同时在目标角色行为上进行了加强

测试:

``` java
public class StaticProxy {

    public static void main(String[] args) {
       // 目标对象
        Owner owner = new Owner();
        // 代理对象
        AgentProxy agentProxy = new AgentProxy(owner);
        // 通过代理对象调用目标对象中的方法
        agentProxy.toRentHouse();
    }
}
```

#### 动态代理

根据需要，通过反射机制在程序运行期间，动态的为目标对象创建对象，动态代理的两种实现方式：

1.JDK动态代理

2.CGLIB动态代理

#### 动态代理的特点

1.目标对象不固定

2.在程序运行时，动态创建目标对象

3.代理对象会增强目标对象的行为

##### JDK动态代理

Proxy类是专门完成代理的操作类，可以通过此类为一个接口或多个接口动态的生成实现类，此类提供如下操作方法

``` java
public static Object newProxyInstance(ClassLoader loader,
                                      类<?>[] interfaces,
                                      InvocationHandler h)
                               throws IllegalArgumentException
返回指定接口的代理类的实例，该接口将方法调用分派给指定的调用处理程序。
Proxy.newProxyInstance因为与IllegalArgumentException相同的原因而Proxy.getProxyClass 。

参数
loader - 类加载器来定义代理类
interfaces - 代理类实现的接口列表
h - 调度方法调用的调用处理函数
结果
具有由指定的类加载器定义并实现指定接口的代理类的指定调用处理程序的代理实例
异常
IllegalArgumentException - 如果对可能传递给 getProxyClass有任何 getProxyClass被违反
SecurityException -如果安全管理器，S存在任何下列条件得到满足：
给定的loader是null ，并且调用者的类加载器不是null ，并且调用s.checkPermission与RuntimePermission("getClassLoader")权限拒绝访问;
对于每个代理接口， intf ，呼叫者的类加载器是不一样的或类加载器的祖先intf和调用s.checkPackageAccess()拒绝访问intf ;
任何给定的代理接口的是非公和呼叫者类是不在同一runtime package作为非公共接口和调用s.checkPermission与ReflectPermission("newProxyInPackage.{package name}")权限拒绝访问。
NullPointerException - 如果 interfaces数组参数或其任何元素是 null ，或者如果调用处理程序 h是 null
```

###### 实现步骤

1.定义类实现InvocationHandler接口

``` java
public class JdkDynamicProxy implements InvocationHandler {
    // 目标对象
    private Object target;

    public JdkDynamicProxy(Object target) {
        this.target = target;
    }

    // 调用目标对象中的方法 （返回Object）
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("新娘是....");
        Object invoke = method.invoke(target,args);
        return invoke;
    }

    /**
     * 获取代理对象
     *  public static Object newProxyInstance(ClassLoader loader,
     *                                       Class[] interfaces,
     *                                       InvocationHandler h)
     *   loader：类加载器
     *   interfaces：接口数组
     *          target.getClass().getInterfaces()：目标对象的接口数组
     *   h：InvocationHandler接口（传入InvocationHandler接口的实现类）
     *   Object o = Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(),new Class );
     *
     * @return
     */
    // 获取代理对象
    public Object getProxy(){
        Object o = Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(),this );
        return o;
    }
}
```

``` java
public class JdkHandlerTest {

    public static void main(String[] args) {
        Owner owner = new Owner();
        JdkDynamicProxy jdkDynamicProxy = new JdkDynamicProxy(owner);
        // 注意这里应该是接口不是实现类
        RentHouse rentHouse= (RentHouse) jdkDynamicProxy.getProxy();
        rentHouse.toRentHouse();
    }
}
```

通过接口获取到代理对象，调用代理对象的方法；

##### CGLIB动态代理

jdk动态代理机制只能代理实现接口的累，而不能实现接口的类就不能使用jdk的动态代理，cglb是针对类来实现代理的，它的原理是对指定的目标类生成一个字类，并覆盖其中方法实现增强，但因为采用的是继承，所以并不能对final修饰的类进行代理。

###### 实现步骤

1.添加依赖

``` xml
<!-- cglib -->
    <!-- https://mvnrepository.com/artifact/cglib/cglib -->
    <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib</artifactId>
      <version>2.2.2</version>
    </dependency>
```

2.定义类实现MethodInterceptor接口

``` java
public class CglibInterceptor implements MethodInterceptor {

    // 目标对象
    private Object target;
    // 通过构造器传入目标对象
    public CglibInterceptor(Object target) {
        this.target = target;
    }

    /**
     * 获取代理对象
     * @return
     */
    public Object getProxy(){
        // 通过Enhancer对象中的create()方法生成一个类，用于生成代理对象
        Enhancer enhancer = new Enhancer();
        // 设置父类 （将目标类作为代理类的父类）
        enhancer.setSuperclass(target.getClass());
        // 设置拦截器(实现了MethodInterceptor接口的类) 这里的回调对象为本身对象
        enhancer.setCallback(this);
        // 生成代理类对象，并返回给调用者
        return enhancer.create();
    }


    /**
     * 拦截器
     *    1. 目标对象的方法调用
     *    2. 行为增强
     * @param o cglib动态生成的代理类的实例
     * @param method    实体类所调用的都被代理的方法的引用
     * @param objects   参数列表
     * @param methodProxy   生成的代理类对方法的代理引用
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

        // 增强行为
        System.out.println("===================方法前执行");

        // 调用目标类中的方法
        Object object = methodProxy.invoke(target, objects);

        // 增强行为
        System.out.println("方法后执行===================");

        return object;
    }


}
```

3.测试

``` java
public class CglibInterceptorTest {

    public static void main(String[] args) {
        //得到目标对象
        Owner owner = new Owner();
        // 得到拦截器
        CglibInterceptor cglibInterceptor = new CglibInterceptor（owner);
        // 得到代理对象
        Owner owner= cglibInterceptor.getProxy();
        // 通过代理对象调用目标对象中的方法
        owner.toRentHouse();
```

总结：jdk动态代理于CGLIB动态代理的区别

jdk动态代理是基于接口，CGLIB是基于类，也就是说jdk动态代理生成代理对象，目标对象必须实现某个接口，CGLIB是基于类(生成一个代理类，该类生成一个目标类的字类，重写目标类的方法)

``` java
Owner owner = new Owner();
        JdkDynamicProxy jdkDynamicProxy = new JdkDynamicProxy(owner);
        // 注意这里应该是接口不是实现类
        RentHouse rentHouse= (RentHouse) jdkDynamicProxy.getProxy();
        rentHouse.toRentHouse();
```

jdk动态获取代理对象的返回值是实现的那个接口

``` java
 //得到目标对象
        Owner owner = new Owner();
        // 得到拦截器
        CglibInterceptor cglibInterceptor = new CglibInterceptor（owner);
        // 得到代理对象
        Owner owner= cglibInterceptor.getProxy();
        // 通过代理对象调用目标对象中的方法
        owner.toRentHouse();
```

CGLIB是基于类的，代理对象必须是这里设置的

``` java
 // 通过Enhancer对象中的create()方法生成一个类，用于生成代理对象
        Enhancer enhancer = new Enhancer();
        // 设置父类 （将目标类作为代理类的父类）
        enhancer.setSuperclass(target.getClass());
        // 设置拦截器 回调对象为本身对象
        enhancer.setCallback(this);
        // 生成代理类对象，并返回给调用者
        return enhancer.create();
```

设置的父类是target，所以返回的类型也应该是target

源代码<a href="https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-aop-demo" rel="noopener noreferrer nofollow" target="_blank">https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-aop-demo</a>
