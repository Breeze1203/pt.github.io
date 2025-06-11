---
title: "spring事务"
date: 2025-06-11 19:23:39
categories: spring
tags: spring
---

## springboot事务

#### 1.编程式事务

基于底层的 API，如 PlatformTransactionManager、TransactionDefinition 和 TransactionTemplate 等核心接口，开发者完全可以通过编程的方式来进行事务管理

##### 使用transactionTemplate

``` java
@Autowired
    private TransactionTemplate transactionTemplate;

    @Test
    void testAffairs(){
        System.out.println("未插入前:"+service.getAllStudent().size());
        transactionTemplate.execute(new TransactionCallback() {
            @Override
            public Object doInTransaction(TransactionStatus status) {
                service.addStudent(new Student("pt","3548297839@qq.com","深圳",22));
                System.out.println("插入后:"+service.getAllStudent().size());
                status.setRollbackOnly();
                return "完成";
            }
        });
        System.out.println("回滚后:"+service.getAllStudent().size());
    }
```

<img src="/upload/截屏2024-04-15%2022.15.54.png" style="display: inline-block;width:100.0%;height:100.0%" />这个执行事务有返回值，无返回值

``` java
@Test
    void test(){
        System.out.println("未插入前:"+service.getAllStudent().size());
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                service.addStudent(new Student("pt","3548297839@qq.com","深圳",22));
                System.out.println("插入后:"+service.getAllStudent().size());
                status.setRollbackOnly();
            }
        });
        System.out.println("回滚后:"+service.getAllStudent().size());
    }
```

``` java
//设置事务传播属性
        transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        // 设置事务的隔离级别,设置为读已提交（默认是ISOLATION_DEFAULT:使用的是底层数据库的默认的隔离级别）
        transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        // 设置是否只读，默认是false
        transactionTemplate.setReadOnly(true);
        // 默认使用的是数据库底层的默认的事务的超时时间
        transactionTemplate.setTimeout(30000);
```

##### 使用TransactionManager

``` java
// 来定义事务的各种属性 然后将其传递给事务管理
        DefaultTransactionDefinition defaultTransactionDefinition = new DefaultTransactionDefinition();
        // 设置事务属性
        defaultTransactionDefinition.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        // 设置事务的传播特性
        defaultTransactionDefinition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        defaultTransactionDefinition.setTimeout(30); // 设置超时时间为30秒
        defaultTransactionDefinition.setReadOnly(false); // 设置为读写事务
        // 开启事务
        TransactionStatus status = transactionManager.getTransaction(defaultTransactionDefinition);
        try {
            // 在这里执行你的事务操作
            // 例如：数据库操作、调用其他事务方法等
            // 提交事务
            transactionManager.commit(status);
        } catch (Exception ex) {
            // 发生异常时回滚事务
            transactionManager.rollback(status);
            throw ex;
        }
```

#### 2.声明式事务

<span style="font-size: 16px; color: rgb(0, 0, 0)">使用声明式事务的方法很简单，其实就是在 Service 层对应方法上配置 </span>@Transaction<span style="font-size: 16px; color: rgb(0, 0, 0)"> 注解即可</span>

<span style="font-size: 16px; color: rgb(0, 0, 0)">假设我们的业务需求是：往 user 和 student 插入的数据，要么都完成，要么都不完成</span>

这时候，我们应该怎么操作呢？

首先，我们需要在 TransactionServiceImpl 类的 addUser() 方法上配置 @Transaction 注解，同时也在 TransactionServiceImpl 类的 addStudent() 方法上配置 @Transaction 注解，修改后的代码如下：

``` java
@Service
public class TransactionServiceImpl {
    @Resource
    StudentServiceImpl studentService;

    @Resource
    UserServiceImpl userService;

    @Transactional
    public void addUser(){
        System.out.println("add user");
        userService.addUser(new User("pt","3548297839@qq.com"));
        addStudent();
    }
    // TransactionServiceB
    @Transactional
    public void addStudent(){
        System.out.println("add student");
        studentService.addStudent(new Student("张三","3435@qq.com","423432",19));
        throw new RuntimeException();
    }
}
```

可以看到，我们在 addStudent() 中模拟了业务异常，我们看看是否 user和 student 都没有插入数据，在SpringBootTest里面测试，如下：

<img src="/upload/截屏2024-04-16%2016.44.55.png" style="display: inline-block;width:100.0%;height:100.0%" />

<img src="/upload/截屏2024-04-16%2016.45.02.png" style="display: inline-block;width:100.0%;height:100.0%" /><img src="/upload/截屏2024-04-16%2016.45.08.png" style="display: inline-block;width:100.0%;height:100.0%" />控制台可以看到，执行了具体方法，<span style="font-size: 16px; color: rgb(0, 0, 0)">这时候我们查看数据库，会发现 user 和 student 都没有插入数据，这说明事务起作用了</span>

#### 3.事务传播类型

事务传播类型，指的是事务与事务之间的交互策略。例如：在事务方法 A 中调用事务方法 B，当事务方法 B 失败回滚时，事务方法 A 应该如何操作？这就是事务传播类型。Spring 事务中定义了 7 种事务传播类型，默认的是分别是：REQUIRED

<img src="/upload/截屏2024-04-16%2021.03.54.png" style="display: inline-block;width:100.0%;height:100.0%" />

REQUIRED、SUPPORTS、MANDATORY、REQUIRES_NEW、NOT_SUPPORTED、NEVER、NESTED。其中最常用的只有 3 种，即：REQUIRED、REQUIRES_NEW、NESTED。

<img src="/upload/截屏2024-04-16%2021.04.19.png" style="display: inline-block;width:100.0%;height:100.0%" />

针对事务传播类型，我们要弄明白的是 4 个点：

1.  子事务与父事务的关系，是否会启动一个新的事务？

2.  子事务异常时，父事务是否会回滚？

3.  父事务异常时，子事务是否会回滚？

4.  父事务捕捉异常后，父事务是否还会回滚？

#### REQUIRED

是 Spring <span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">默认</span>的事务传播类型，该传播类型的特点是：<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">当前方法存在事务时，子方法加入该事务。此时父子方法共用一个事务，无论父子方法哪个发生异常回滚，整个事务都回滚。即使父方法捕捉了异常，也是会回滚。而当前方法不存在事务时，子方法新建一个事务</span>**。**

``` java
@Service
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    public void addUser(){
        System.out.println("add user");
        userService.addUser(new User("pt","3548297839@qq.com"));
        studentService.addStudent(new Student("ce","343","3423",22));
    }
}

@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

执行代码：

<img src="/upload/截屏2024-04-16%2020.29.30.png" style="display: inline-block;width:100.0%;height:100.0%" /><img src="/upload/截屏2024-04-16%2020.29.24.png" style="display: inline-block;width:100.0%;height:100.0%" />可以看到user插入了数据，student没有，符合我们的预期

<span style="font-size: 16px; color: rgb(0, 0, 0)">当 addUser 开启事务，addStudent 也开启事务。按照我们的结论，此时 addUser 会加入 addStudent 的事务。此时，我们验证当父子事务分别回滚时，另外一个事务是否会回滚;</span>

<span style="font-size: 16px; color: rgb(0, 0, 0)">我们先验证第一个：当父方法事务回滚时，子方法事务是否会回滚？</span>

``` java
@Service
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        studentService.addStudent(new Student("ce", "343", "3423", 22));
        throw new RuntimeException();
    }
}

@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

结果是：user和student都没有插入数据，结论是，父事务回滚了，子事务也会滚了

<span style="font-size: 16px; color: rgb(0, 0, 0)">我们继续验证第三个：当子方法事务回滚时，父方法捕捉了异常，父方法事务是否会回滚？</span>

``` java
@Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        try {
            studentService.addStudent(new Student("ce", "343", "3423", 22));
        }catch (Exception e){
            System.out.println(e.getMessage());
        }
    }

  @Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

得出结果：<span style="font-size: 16px; color: rgb(0, 0, 0)">user 和 student 都没有插入数据，即：子事务回滚时，父事务也回滚了。所以说，这也进一步验证了我们之前所说的：REQUIRED 传播类型，它是父子方法共用同一个事务的</span>

#### **REQUIRES_NEW**

REQUIRES_NEW 也是常用的一个传播类型，该传播类型的特点是：<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">无论当前方法是否存在事务，子方法都新建一个事务。此时父子方法的事务时独立的，它们都不会相互影响。但父方法需要注意子方法抛出的异常，避免因子方法抛出异常，而导致父方法回滚</span>**。** 为了验证 REQUIRES_NEW 事务传播类型的特点，我们来做几个测试。

首先，我们来验证一下：当父方法事务发生异常时，子方法事务是否会回滚？

``` java
@Service
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        studentService.addStudent(new Student("ce", "343", "3423", 22));
        throw new RuntimeException();
    }
}
@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW )
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
    }
}
```

可以看到，student插入了数据，user没有，得出结论，子方法没回滚，父方法回滚了，<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">证</span><span style="font-size: 16px; color: rgb(239, 68, 68)">明父子方法的事务是独立的，不相互影响</span>

<span style="font-size: 16px; color: rgb(0, 0, 0)">下面，我们来看看：当子方法事务发生异常时，父方法事务是否会回滚？</span>

``` java
@Service
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        studentService.addStudent(new Student("ce", "343", "3423", 22));
    }
}


@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW )
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

数据库结果为user和student都没有插入数据，<span style="font-size: 16px; color: rgb(0, 0, 0)">从这个结果来看，貌似是子方法事务回滚，导致父方法事务也回滚了。但我们不是说父子事务都是独立的，不会相互影响么？怎么结果与此相反呢？</span>

<span style="font-size: 16px; color: rgb(0, 0, 0)">其实是因为子方法抛出了异常，而父方法并没有做异常捕捉，此时父方法同时也抛出异常了，于是 Spring 就会将父方法事务也回滚了。如果我们在父方法中捕捉异常，那么父方法的事务就不会回滚了，修改之后的代码如下所示</span>

``` java
@Service
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        try{
            studentService.addStudent(new Student("ce", "343", "3423", 22));
        }catch (Exception e){
            System.out.println(e.getMessage());
        }
    }
}
@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW )
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

结果是：user 插入了数据，student 没有插入数据。这正符合我们刚刚所说的：父子事务是独立的，并不会相互影响。

这其实就是我们上面所说的：父方法需要注意子方法抛出的异常，避免因子方法抛出异常，而导致父方法回滚。因为如果执行过程中发生 RuntimeException 异常和 Error 的话，那么 Spring 事务是会自动回滚的

#### **NESTED**

<span style="font-size: 16px; color: rgb(0, 0, 0)">NESTED 也是常用的一个传播类型，该方法的特性与 REQUIRED 非常相似，其特性是：</span><span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">当前方法存在事务时，子方法加入在嵌套事务执行。当父方法事务回滚时，子方法事务也跟着回滚。当子方法事务发送回滚时，父事务是否回滚取决于是否捕捉了异常。如果捕捉了异常，那么就不回滚，否则回滚</span>

可以看到 NESTED 与 REQUIRED 的区别在于：父方法与子方法对于共用事务的描述是不一样的，REQUIRED 说的是共用同一个事务，而 NESTED 说的是在嵌套事务执行。这一个区别的具体体现是：<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">在子方法事务发生异常回滚时，父方法有着不同的反应动作</span>**<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">。</span>**

对于 REQUIRED 来说，无论父子方法哪个发生异常，全都会回滚。而 REQUIRED 则是：父方法发生异常回滚时，子方法事务会回滚。而子方法事务发送回滚时，父事务是否回滚取决于是否捕捉了异常。

为了验证 NESTED 事务传播类型的特点，我们来做几个测试。

首先，我们来验证一下：当父方法事务发生异常时，子方法事务是否会回滚？

``` java
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        studentService.addStudent(new Student("ce", "343", "3423", 22));
        throw new RuntimeException();
    }
}

@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
    }
}
```

<span style="font-size: 16px; color: rgb(0, 0, 0)">结果是：user 和 student 都没有插入数据，即：父子方法事务都回滚了。这说明父方法发送异常时，子方法事务会回滚</span>

<span style="font-size: 16px; color: rgb(0, 0, 0)">接着，我们继续验证一下：当子方法事务发生异常时，如果父方法没有捕捉异常，父方法事务是否会回滚？</span>

``` java
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        studentService.addStudent(new Student("ce", "343", "3423", 22));
    }
}


@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

结果是：user 和 studnet 都没有插入数据，即：父子方法事务都回滚了。这说明子方法发送异常回滚时，如果父方法没有捕捉异常，那么父方法事务也会回滚。

最后，我们验证一下：当子方法事务发生异常时，如果父方法捕捉了异常，父方法事务是否会回滚？

``` java
public class TransactionServiceImpl {
    @Resource(name = "StudentService")
    StudentServiceImpl studentService;

    @Resource(name = "UserService")
    UserServiceImpl userService;

    @Transactional
    public void addUser() {
        System.out.println("add user");
        userService.addUser(new User("pt", "3548297839@qq.com"));
        try {
            studentService.addStudent(new Student("ce", "343", "3423", 22));
        }catch (Exception e){
            System.out.println("---");
        }
    }
}

@Service(value = "StudentService")
public class StudentServiceImpl implements StudentService {
    @Resource(name = "StudentMapper")
    private StudentMapper studentMapper;

    @Override
    public List<Student> getAllStudent() {
        return studentMapper.getAllStudent();
    }


    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addStudent(Student student) {
        studentMapper.addStudent(student);
        throw new RuntimeException();
    }
}
```

结果是：user 插入了数据，student没有插入数据，即：父方法事务没有回滚，子方法事务回滚了。这说明子方法发送异常回滚时，如果父方法捕捉了异常，那么父方法事务就不会回滚。

看到这里，相信大家已经对 REQUIRED、REQUIRES_NEW 和 NESTED 这三个传播类型有了深入的理解了。最后，让我们来总结一下：

<div style="overflow-x: auto; overflow-y: hidden;">

|  |  |
|----|----|
| **<span style="font-size: 14px; color: rgb(0, 0, 0)">事务传播类型</span>** | **<span style="font-size: 14px; color: rgb(0, 0, 0)">特性</span>** |
| <span style="font-size: 14px; color: rgb(0, 0, 0)">REQUIRED</span> | <span style="font-size: 14px; color: rgb(0, 0, 0)">当前方法存在事务时，子方法加入该事务。此时父子方法共用一个事务，无论父子方法哪个发生异常回滚，整个事务都回滚。即使父方法捕捉了异常，也是会回滚。而当前方法不存在事务时，子方法新建一个事务。</span> |
| <span style="font-size: 14px; color: rgb(0, 0, 0)">NESTED</span> | <span style="font-size: 14px; color: rgb(0, 0, 0)">当前方法存在事务时，子方法加入在嵌套事务执行。当父方法事务回滚时，子方法事务也跟着回滚。当子方法事务发送回滚时，父事务是否回滚取决于是否捕捉了异常。如果捕捉了异常，那么就不回滚，否则回滚。</span> |
| <span style="font-size: 14px; color: rgb(0, 0, 0)">REQUIRES_NEW</span> | <span style="font-size: 14px; color: rgb(0, 0, 0)">无论当前方法是否存在事务，子方法都新建一个事务。此时父子方法的事务时独立的，它们都不会相互影响。但父方法需要注意子方法抛出的异常，避免因子方法抛出异常，而导致父方法回滚。</span> |

</div>

#### 使用方法论

看完了事务的传播类型，我们对 Spring 事务又有了深刻的理解。

看到这里，你应该也明白：使用事务，不再是简单地使用 @Transaction 注解就可以，还需要根据业务场景，选择合适的传播类型。那么我们再升华一下使用 Spring 事务的方法论。一般来说，使用 Spring 事务的步骤为：

1.  根据业务场景，分析要达成的事务效果，确定使用的事务传播类型。

2.  在 Service 层使用 @Transaction 注解，配置对应的 propogation 属性。

下次遇到要使用事务的情况，记得按照这样的步骤去做哦~

#### **Spring 事务失效**

什么时候 Spring 事务会失效？

若同一类中的其他没有 @Transactional 注解的方法内部调用有 @Transactional 注解的方法，有 @Transactional 注解的方法的事务会失效。

这是由于 Spring AOP 代理的原因造成的，因为只有当 @Transactional 注解的方法在类以外被调用的时候，Spring 事务管理才生效。

另外，如果直接调用，不通过对象调用，也是会失效的。因为 Spring 事务是通过 AOP 实现的。

@Transactional 注解只有作用到 public 方法上事务才生效。

被 @Transactional 注解的方法所在的类必须被 Spring 管理。

底层使用的数据库必须支持事务机制，否则不生效

<span style="font-size: 16px; color: rgb(0, 0, 0)">Spring 事务执行过程中，如果抛出非 RuntimeException 和非 Error 错误的其他异常，那么是不会回滚的哦</span>

源代码：<a href="https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-boot-affairs" rel="noopener noreferrer nofollow" target="_blank">https://github.com/Breeze1203/JavaAdvanced/tree/main/springboot-demo/spring-boot-affairs</a>
