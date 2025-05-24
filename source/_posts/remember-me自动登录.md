---
title: "Remember Me自动登录"
date: 2024-11-18 15:41:52
---

## RememberMe 的配置

RememberMe 功能，相信你在很多网站都见过，简单讲，就是可以让网站在一段时间内“记住我”，免除每次都需要填写用户名/密码登录的麻烦。

这样的功能通常需要在 Cookie 中存放一个 Token 字符串（或者类似的东西），服务端通过这个 Token 解析对应的用户信息和失效，从而实现自动登录。

在 Spring Security 中，给我们提供了这个功能，默认是关闭的。如果需要在一个配置好了 Spring Security 并且提供了用户名/密码表单登录的工程上，开启 RememberMe，你需要做这么几件事儿：

第一步，在 Spring Security 的配置类中，添加如下的内容：

``` java
public void customize(RememberMeConfigurer<HttpSecurity> httpSecurityRememberMeConfigurer) {
       httpSecurityRememberMeConfigurer.rememberMeParameter("remember") //标识记住我功能的参数名或者请求参数名
                .tokenRepository(customizeTokenRepository)
                .rememberMeCookieDomain("localhost")//存储库负责存储生成的令牌
                .tokenValiditySeconds(5*60*60);//设置生成的记住我令牌的有效时间
                //.rememberMeServices(rememberMeServices);//处理记住我功能的核心逻辑););//处理记住我功能的核心逻辑);
    }
```

以上代码中隐藏了其余配置的部分，其中，最关键的就是 `rememberMe()` 方法，使得 `RememberMeAuthenticationFilter` 被加入 Spring Security 的过滤器链，并完成相关的功能，这里的细节，后续再去分析。

之后的被注释的代码，这些方法是我们对这个功能进行自定义的内容，因此不是必需的，后面我们遇到相关内容的时候再讲。

第二步，如果你自定义了登录表单，需要在表单中增加一个复选框。

``` html
<input name="remember-me" type="checkbox" />记住我</td>
```

这里的「记住我」三个字，可以随意写，能表达意思即可。但是 `input` 标签的 `name` 属性中的 `remember-me` 属性值，默认情况下是 Spring Security 规定好的，它会作为这里的表单参数名，也会作为 RememberMe 功能需要用到的 Cookie 名称。

## 从用户名/密码认证说起

这里需要你了解 Spring Security 的用户名/密码认证的原理，不了解的话可以参考我之前的文章（<a href="https://juejin.cn/post/7054569250307964958" target="_blank">Spring Security 认证流程</a> ）。

<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">UsernamePasswordAuthenticationFilter 过滤器的 doFilter 方法在其父类 AbstractAuthenticationProcessingFilter 中实现，在方法中，如果用户信息通过了认证，会调用 successfulAuthentication 方法，处理之后的逻辑，我们看一下这个方法的代码</span>：

``` java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
        context.setAuthentication(authResult);
        this.securityContextHolderStrategy.setContext(context);
        this.securityContextRepository.saveContext(context, request, response);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
        }

        this.rememberMeServices.loginSuccess(request, response, authResult);
        if (this.eventPublisher != null) {
            this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
        }

        this.successHandler.onAuthenticationSuccess(request, response, authResult);
    }
```

<span style="font-size: 16px; color: rgb(37, 41, 51)">调用了 </span>`this.rememberMeServices.loginSuccess`<span style="font-size: 16px; color: rgb(37, 41, 51)"> 方法，这便是与 RememberMe 有关的代码</span>

## RememberMeServices

上面提到的的 rememberMeServices 是 RememberMeServices 类型，在 AbstractAuthenticationProcessingFilter 中是这样声明这个变量的：

``` java
private RememberMeServices rememberMeServices = new NullRememberMeServices();
```

`RememberMeServices` 这个接口，在 Spring Security 中只内置了两种直接的实现，上面代码中是默认使用的实现，其实就是不提供 RememberMe 功能时使用的实现，我们从 `NullRememberMeServices` 的名字中也能看得出来，实际上，它的所有方法实现都是空方法。

<u><span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">当我们在配置类中用 http.rememberMe() 开启了 RememberMe 功能后，这里的 rememberMeServices 会被替换成另一个实现，就是 AbstractRememberMeServices。AbstractRememberMeServices 是一个抽象类，它有两个非抽象的实现类，它们的层次结构是这样的：  
这里 AbstractRememberMeServices 的两个非抽象子类，Spring Security 的 RememberMe 的功能到底会使用哪个，取决于我们的配置</span></u>。

- <span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">默认情况下会使用 </span>`TokenBasedRememberMeServices`<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)"> ，提供了基础的功能。</span>

- <span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">如果我们在开启 RememberMe 功能的时候，同时配置了一个 </span>`PersistentTokenRepository`<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)">，那么 Spring Security 会自动选择 </span>`PersistentTokenBasedRememberMeServices`<span color="rgb(239, 68, 68)" fontsize="" style="color: rgb(239, 68, 68)"> 的实现。这样的配置表示我们会使用持久化的方式保存 RememberMe 功能用到的 Token。这一部分的细节会在下一篇文章中介绍。</span>

## 自定义令牌存储位置

默认实现有

- InMemoryTokenRepositoryImpl\`这只用于测试。

- JdbcTokenRepositoryImpl 它将令牌存储在数据库中。

我们可以自定义存储位置，我这里利用的是mybatis，

#### 初始化数据库脚本

``` sql
create table persistent_logins (username varchar(64) not null,
                                series varchar(64) primary key,
                                token varchar(64) not null,
                                last_used timestamp not null)
```

#### 实现PersistentTokenRepository接口

``` java
public class CustomizeTokenRepository implements PersistentTokenRepository {

    private final TokenMapper tokenMapper;

    public CustomizeTokenRepository(@Qualifier("RememberTokenMapper") TokenMapper tokenMapper) {
        this.tokenMapper = tokenMapper;
    }

    @Override
    public void createNewToken(PersistentRememberMeToken token) {
        tokenMapper.createToken(token);
    }

    @Override
    public void updateToken(String series, String tokenValue, Date lastUsed) {
        tokenMapper.updateUserToken(series, tokenValue, lastUsed);
    }

    @Override
    public PersistentRememberMeToken getTokenForSeries(String seriesId) {
        return tokenMapper.getTokenBySeries(seriesId);
    }

    @Override
    public void removeUserTokens(String username) {
        removeUserTokens(username);
    }
}
```

#### 将自定义的存储类注入

``` java
@Component
public class CustomizeRememberConfig implements Customizer<RememberMeConfigurer<HttpSecurity>>{
    @Autowired
    private CustomizeTokenRepository customizeTokenRepository;


    @Override
    public void customize(RememberMeConfigurer<HttpSecurity> httpSecurityRememberMeConfigurer) {
       httpSecurityRememberMeConfigurer.rememberMeParameter("remember") //标识记住我功能的参数名或者请求参数名
                .tokenRepository(customizeTokenRepository)
                .rememberMeCookieDomain("localhost")//存储库负责存储生成的令牌
                .tokenValiditySeconds(5*60*60);//设置生成的记住我令牌的有效时间
                //.rememberMeServices(rememberMeServices);//处理记住我功能的核心逻辑););//处理记住我功能的核心逻辑);
    }
}
```

## 自定义RememberService

``` java
package com.example.eachadmin.config.remember;

import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.RememberMeServices;
import org.springframework.security.web.authentication.rememberme.InvalidCookieException;
import org.springframework.util.StringUtils;

import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.util.Base64;
/*
这个方法可以配置springsecurity自动登录之后的一些逻辑
大概思路是：从浏览器cookie获取到token令牌
但这个令牌是被springsecurity加密算法加密过，所以你得先解密
解密后获取到series 通过这个series查询用户，去令牌存储位置查找令牌以进行登录判断
具体逻辑可以参考TokenBasedRememberMeServices这个类里面的实现方法
 */




public class CustomizeRememberMeServices implements RememberMeServices {
    private final int tokenValiditySeconds = 1209600;

    private static final String REMEMBER_ME_COOKIE = "remember-me";


    @Override
    public Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {
        Cookie[] cookies = request.getCookies();
        // If no cookies are present, return null
        if (cookies == null) {
            return null;
        }
        for (Cookie cookie : cookies) {
            if (REMEMBER_ME_COOKIE.equals(cookie.getName())) {
                String cookieValue = cookie.getValue();
                if (cookieValue != null) {
                    return null;
                }
            }
        }
        return null;
    }



    @Override
    public void loginFail(HttpServletRequest request, HttpServletResponse response) {
        // 可以在这里实现登录失败的逻辑，例如记录登录失败次数等
        System.out.println("Login failed...");
    }

    @Override
    public void loginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication
            successfulAuthentication) {
        // 可以在这里实现登录成功的逻辑，例如记录登录成功日志等
        System.out.println("Login succeeded for user: " + successfulAuthentication.getName());
    }

    /*
    这是翻源码找到的解密算法
     */
    public static  String[] decodeCookie(String cookieValue) throws InvalidCookieException {
        for(int j = 0; j < cookieValue.length() % 4; ++j) {
            cookieValue = cookieValue + "=";
        }
        String cookieAsPlainText;
        try {
            cookieAsPlainText = new String(Base64.getDecoder().decode(cookieValue.getBytes()));
        } catch (IllegalArgumentException var7) {
            throw new InvalidCookieException("Cookie token was not Base64 encoded; value was '" + cookieValue + "'");
        }

        String[] tokens = StringUtils.delimitedListToStringArray(cookieAsPlainText, ":");

        for(int i = 0; i < tokens.length; ++i) {
            try {
                tokens[i] = URLDecoder.decode(tokens[i], StandardCharsets.UTF_8.toString());
            } catch (UnsupportedEncodingException var6) {
                System.out.println("草泥马");
            }
        }

        return tokens;
    }
}
```

处理自动登录的方法autologin，这里是处理的逻辑，可以重写这个方法，并将rememberservice注入

大概实现流程是根据传过来的cookie解析出series和token，进行处理，是判断token令牌是否存在，过期等操作，进行相对应处理

可以参考源码

``` java
public final Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {
        String rememberMeCookie = this.extractRememberMeCookie(request);
        if (rememberMeCookie == null) {
            return null;
        } else {
            this.logger.debug("Remember-me cookie detected");
            if (rememberMeCookie.length() == 0) {
                this.logger.debug("Cookie was empty");
                this.cancelCookie(request, response);
                return null;
            } else {
                try {
                    String[] cookieTokens = this.decodeCookie(rememberMeCookie);
                    UserDetails user = this.processAutoLoginCookie(cookieTokens, request, response);
                    this.userDetailsChecker.check(user);
                    this.logger.debug("Remember-me cookie accepted");
                    return this.createSuccessfulAuthentication(request, user);
                } catch (CookieTheftException var6) {
                    this.cancelCookie(request, response);
                    throw var6;
                } catch (UsernameNotFoundException var7) {
                    this.logger.debug("Remember-me login was valid but corresponding user not found.", var7);
                } catch (InvalidCookieException var8) {
                    this.logger.debug("Invalid remember-me cookie: " + var8.getMessage());
                } catch (AccountStatusException var9) {
                    this.logger.debug("Invalid UserDetails: " + var9.getMessage());
                } catch (RememberMeAuthenticationException var10) {
                    this.logger.debug(var10.getMessage());
                }

                this.cancelCookie(request, response);
                return null;
            }
        }
    }
```

这里也是先解析cookie，获取token，然后调用processAutoLoginCookie，这个方法源码如下

``` java
protected UserDetails processAutoLoginCookie(String[] cookieTokens, HttpServletRequest request, HttpServletResponse response) {
        if (!this.isValidCookieTokensLength(cookieTokens)) {
            throw new InvalidCookieException("Cookie token did not contain 3 or 4 tokens, but contained '" + Arrays.asList(cookieTokens) + "'");
        } else {
            long tokenExpiryTime = this.getTokenExpiryTime(cookieTokens);
            if (this.isTokenExpired(tokenExpiryTime)) {
                Date var10002 = new Date(tokenExpiryTime);
                throw new InvalidCookieException("Cookie token[1] has expired (expired on '" + var10002 + "'; current time is '" + new Date() + "')");
            } else {
                UserDetails userDetails = this.getUserDetailsService().loadUserByUsername(cookieTokens[0]);
                Assert.notNull(userDetails, () -> {
                    UserDetailsService var10000 = this.getUserDetailsService();
                    return "UserDetailsService " + var10000 + " returned null for username " + cookieTokens[0] + ". This is an interface contract violation";
                });
                String actualTokenSignature = cookieTokens[2];
                RememberMeTokenAlgorithm actualAlgorithm = this.matchingAlgorithm;
                if (cookieTokens.length == 4) {
                    actualTokenSignature = cookieTokens[3];
                    actualAlgorithm = TokenBasedRememberMeServices.RememberMeTokenAlgorithm.valueOf(cookieTokens[2]);
                }

                String expectedTokenSignature = this.makeTokenSignature(tokenExpiryTime, userDetails.getUsername(), userDetails.getPassword(), actualAlgorithm);
                if (!equals(expectedTokenSignature, actualTokenSignature)) {
                    throw new InvalidCookieException("Cookie contained signature '" + actualTokenSignature + "' but expected '" + expectedTokenSignature + "'");
                } else {
                    return userDetails;
                }
            }
        }
    }
```

是对令牌进行一些认证处理，可以看到返回一个userDetails对象，返回来的userDetails对象，进行check（感兴趣可以去翻看源码）,最后调用createSuccessfulAuthentication
