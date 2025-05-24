---
title: "springsecurity session"
date: 2024-11-18 15:41:52
---

## 什么是session(会话)?

> 会话是一种持久的网络协议，用于完成服务器和客户端之间的一些交互行为。会话是一个比连接粒度更大的概念，一次会话可能包含多次连接，每次连接都被认为是会话的一次操作。

在Web中，Session是指一个用户与网站服务器进行一系列交互的持续时间，通常指从注册进入系统到注销退出系统之间所经过的时间，以及在这段时间内进行的操作，还有，服务器端为保存用户状态开辟的存储空间。

### http协议的无状态性

每条http请求/响应是互相独立的，服务器并不知道两条http请求是同一用户发送的还是不同的用户发送的，就好比生活中常见的饮料自动贩售机，投入硬币，选择饮料，然后饮料出来，买饮料的人拿到饮料，整个过程中贩售机只负责识别要买的是哪种饮料并且给出饮料，并没有记录是哪个人买的，以及某个人买了哪几种，每种买了几瓶。如果考虑现在的购物网站的购物车功能，记录用户状态就很有必要了，这就需要用到会话，而http协议由于本身的无状态性就需要cookie来实现会话。

### cookie的概念

用户第一次访问网站时，服务器对用户没有任何了解，于是通过http响应给用户发送一个cookie，让浏览器存下来，浏览器记住这个cookie之后，用户再向这个网站发送http请求的时候就带上这个cookie，服务器收到这个cookie后就能识别出这个特定的用户，从而通过查询服务器数据库中为这个用户积累的一些特定信息来给用户一些个性化服务

### Session和cookie在基于express框架项目中的应用

    import session from "express-session";
    import mongo from "connect-mongo"; // 一般用来将session存储到数据库中

    const MongoStore = mongo(session);
    app.use(session({
        resave: true,// 强制保存session 
        saveUninitialized: true,// 强制保存未初始化内容
        secret: SESSION_SECRET, // 加密字符串,防止篡改cookie
        store: new MongoStore({ //将session存进数据库 
            url: MONGODB_URI,
            autoReconnect: true
        })
    }));

上述代码中的cookie项即是用来存储sessionID的，在express-session这个中间件中，存储sessionID的cookie的名字是connet.sid。

访问这个项目的页面，在浏览器控制台中可以看到response header部分的set-Cookie

<a href="https://camo.githubusercontent.com/0e157d812c270a3ff7b86e978ef7d8d4ed3e38cb792ee35b18029b3bb97fd6e5/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f303038333172535467793167643174713133746a636a333068663039396468362e6a7067" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/0e157d812c270a3ff7b86e978ef7d8d4ed3e38cb792ee35b18029b3bb97fd6e5/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f303038333172535467793167643174713133746a636a333068663039396468362e6a7067" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

当登录网站的时候，服务端就创建了一个session，把sessionID存在connect.sid这一cookie值中，然后通过Set-Cookie这个响应头把sessionID发回给浏览器，接着刷新页面就可以看到请求的request header部分

<a href="https://camo.githubusercontent.com/2b880b376825b4e8ef00c75b755e388ea4a15f59f6a29f2a9a7d9af24cb00f88/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303833317253546779316764317473627531396e6a333068343036386a73692e6a7067" rel="noopener noreferrer nofollow" target="_blank"><u><img src="https://camo.githubusercontent.com/2b880b376825b4e8ef00c75b755e388ea4a15f59f6a29f2a9a7d9af24cb00f88/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303833317253546779316764317473627531396e6a333068343036386a73692e6a7067" style="display: inline-block;width:100.0%;height:100.0%" /></u></a>

也就是说，浏览器通过cookie把这个sessionID存了下来，又通过这个http请求把刚才收到的cookie值也就是sessionID发回给了服务器，服务器就能识别出这个http请求还是同一个用户的会话。

创建了会话之后，就可以通过req.session 获取当前用户的会话对象（这个对象是存在服务端数据库中的），比如这个项目中，用户注册或者登陆以后就把user对象存到req.session.user中，登出时则req.session.user = null;，从而可以通过req.session.user判断用户的登录状态，控制用户访问页面的权限。

### 总结

http协议只完成请求和响应的工作，不能记录用户状态，服务器为了记录用户状态，跟踪用户行为，从而提供个性化服务而引入session，用户的一系列交互行为都包含在session内，用户状态信息也存储在服务器端为session开辟的一个存储空间内；当用户通过浏览器发送一系列http请求时，为了识别这些请求属于哪个session，服务端需要给每个session一个sessionID，cookie就是浏览器端用来存储和发送这个sessionID的

## 配置并发会话控制

<span style="font-size: 15.1111px; color: rgb(25, 30, 30)">如果你希望对单个用户登录你的应用程序的能力进行限制，</span> Spring Security 支持开箱即用，只需添加以下简单内容。首先，你需要在你的配置中添加以下 listener，以保持  Spring Security 对会话生命周期事件的更新

``` java
@Bean
public HttpSessionEventPublisher httpSessionEventPublisher() {
    return new HttpSessionEventPublisher();
}
```

然后在你的 security 配置中添加以下几行：

``` java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) {
    http
        .sessionManagement(session -> session
            .maximumSessions(1)
        );
    return http.build();
}
```

<span color="rgb(255, 255, 255)" fontsize="" style="color: rgb(255, 255, 255)">Copied</span>

## 检测超时

会话会自行过期，不需要做任何事情来确保 security context 被删除。也就是说， Spring Security 可以检测到会话过期的情况，并采取你指定的具体行动。例如，当用户用已经过期的会话发出请求时，你可能想重定向到一个特定的端点。这可以通过 HttpSecurity 中的 invalidSessionUrl 实现

### 定制失效会话的策略

invalidSessionUrl 是使用 SimpleRedirectInvalidSessionStrategy 实现 来设置 InvalidSessionStrategy 的方便方法。如果你想自定义行为，你可以实现 InvalidSessionStrategy 接口并使用 invalidSessionStrategy 方法进行配置

``` java
@Component
public class CustomizeInvalidSessionStrategy implements InvalidSessionStrategy {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Override
    public void onInvalidSessionDetected(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        // 设置响应的内容类型和字符编码
        response.setContentType("text/html;charset=UTF-8");
        // 输出会话已过期的提示信息到浏览器
        String id = request.getSession().getId();
        // 删除对应用户认证的信息
        redisTemplate.delete(CustomizeSecurityContextRepository.SECURITY_CONTEXT_KEY+id);
        // 删除对应的cookie
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals("JSESSIONID")) {
                    cookie.setMaxAge(0); // 设置cookie的过期时间为0，浏览器会将其删除
                    response.addCookie(cookie);
                    break;
                }
            }
        }
        response.getWriter().write("会话已过期，请重新登录");
    }
}
```
