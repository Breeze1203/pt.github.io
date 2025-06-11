---
title: "csrf"
date: 2025-06-11 19:23:41
categories: csrf
tags: csrf
---

## 什么是csrf?

CSRF，全称为Cross-Site Request Forgery（跨站请求伪造），是一种Web安全攻击。在CSRF攻击中，攻击者通过诱使受害者在已登录的情况下，访问一个恶意网站或点击恶意链接，来利用受害者的身份在受攻击网站上执行未经授权的操作。

具体来说，CSRF攻击利用了用户的浏览器对于同一站点的认可，即使用户在访问正常站点时是经过授权的（例如登录了一个网站），但用户在访问其他站点时，浏览器仍然会发送相同的认证信息（如Cookie），从而被攻击者利用来执行攻击。攻击者可以以受害者的名义执行例如转账、更改密码等操作，而受害者可能对此毫无察觉。

## 如何防范CSRF？

为了防范CSRF攻击，常见的措施包括：

1.  **CSRF令牌（Token）**: 服务器生成一个随机的令牌，嵌入到每个表单或者每个请求中，攻击者由于无法获取到这个随机的令牌，无法构造一个有效的请求。

2.  **SameSite Cookie属性**: 控制浏览器是否在跨站点请求时发送Cookie，可以设置为Strict或者Lax以限制Cookie的发送。

3.  **Referer检查**: 服务器验证请求来源的Referer头部，但这种方法可被伪造和篡改。

4.  **双重提交Cookie**: 将一个随机生成的Cookie和Form表单中的字段进行比较

## 这篇文章讲解csrf令牌?

在使用Spring Security进行CSRF保护时，确实需要在每个请求中携带CSRF令牌，并在服务器端验证令牌的正确性。以下是关键步骤和策略：

### 1. 生成和携带CSRF令牌

当用户登录成功后，Spring Security会生成一个CSRF令牌，并将其包含在响应中，通常是作为Cookie的一部分（名为\`XSRF-TOKEN\`）

\- 客户端（通常是浏览器）收到这个令牌后，会将它存储起来，并在后续的请求中自动发送给服务器。

### 2. 在请求中包含CSRF令牌

在每个涉及到修改数据或执行敏感操作的请求中，客户端需要将CSRF令牌作为参数或者请求头的一部分发送给服务器。

通常情况下，Spring Security会期望CSRF令牌以名为 `_csrf` 的参数名发送，或者作为名为 `X-XSRF-TOKEN` 的请求头发送。

### 3. 服务器端验证CSRF令牌

Spring Security会在后台进行CSRF令牌的验证。验证主要是通过比对请求中的CSRF令牌和服务器端存储的令牌（通常是存储在Cookie中）来实现的。

在Spring Security的配置中，设置了`CsrfTokenRequestAttributeHandler` 和 `CookieCsrfTokenRepository` 来处理和存储CSRF令牌。具体来说：

`CsrfTokenRequestAttributeHandler` 用于处理请求中的CSRF令牌参数

``` java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.security.web.csrf;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.util.function.Supplier;
import org.springframework.util.Assert;

public class CsrfTokenRequestAttributeHandler implements CsrfTokenRequestHandler {
    private String csrfRequestAttributeName = "_csrf";

    public CsrfTokenRequestAttributeHandler() {
    }

    public final void setCsrfRequestAttributeName(String csrfRequestAttributeName) {
        this.csrfRequestAttributeName = csrfRequestAttributeName;
    }

    public void handle(HttpServletRequest request, HttpServletResponse response, Supplier<CsrfToken> deferredCsrfToken) {
        Assert.notNull(request, "request cannot be null");
        Assert.notNull(response, "response cannot be null");
        Assert.notNull(deferredCsrfToken, "deferredCsrfToken cannot be null");
        request.setAttribute(HttpServletResponse.class.getName(), response);
        CsrfToken csrfToken = new SupplierCsrfToken(deferredCsrfToken);
        request.setAttribute(CsrfToken.class.getName(), csrfToken);
        String csrfAttrName = this.csrfRequestAttributeName != null ? this.csrfRequestAttributeName : csrfToken.getParameterName();
        request.setAttribute(csrfAttrName, csrfToken);
    }

    private static final class SupplierCsrfToken implements CsrfToken {
        private final Supplier<CsrfToken> csrfTokenSupplier;

        private SupplierCsrfToken(Supplier<CsrfToken> csrfTokenSupplier) {
            this.csrfTokenSupplier = csrfTokenSupplier;
        }

        public String getHeaderName() {
            return this.getDelegate().getHeaderName();
        }

        public String getParameterName() {
            return this.getDelegate().getParameterName();
        }

        public String getToken() {
            return this.getDelegate().getToken();
        }

        private CsrfToken getDelegate() {
            CsrfToken delegate = (CsrfToken)this.csrfTokenSupplier.get();
            if (delegate == null) {
                throw new IllegalStateException("csrfTokenSupplier returned null delegate");
            } else {
                return delegate;
            }
        }
    }
}
```

`CookieCsrfTokenRepository` 用于存储和检索CSRF令牌，它默认将CSRF令牌存储在名为 `XSRF-TOKEN` 的Cookie中，并在需要时使用该Cookie中的令牌值来验证请求中的CSRF令牌

可以查看源码

``` java
.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()))
```

这里是默认的存储和检索csrf令牌的方式，我们进去之后

<img src="/upload/截屏2024-06-24%2022.21.06.png" style="display: inline-block;width:100.0%;height:100.0%" />

可以看到返回一个CookieCsrfTokenRepository对象，我们在查看这个对象

``` java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.security.web.csrf;

import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import java.util.UUID;
import java.util.function.Consumer;
import org.springframework.http.ResponseCookie;
import org.springframework.util.Assert;
import org.springframework.util.StringUtils;
import org.springframework.web.util.WebUtils;

public final class CookieCsrfTokenRepository implements CsrfTokenRepository {
    static final String DEFAULT_CSRF_COOKIE_NAME = "XSRF-TOKEN";
    static final String DEFAULT_CSRF_PARAMETER_NAME = "_csrf";
    static final String DEFAULT_CSRF_HEADER_NAME = "X-XSRF-TOKEN";
    private static final String CSRF_TOKEN_REMOVED_ATTRIBUTE_NAME = CookieCsrfTokenRepository.class.getName().concat(".REMOVED");
    private String parameterName = "_csrf";
    private String headerName = "X-XSRF-TOKEN";
    private String cookieName = "XSRF-TOKEN";
    private boolean cookieHttpOnly = true;
    private String cookiePath;
    private String cookieDomain;
    private Boolean secure;
    private int cookieMaxAge = -1;
    private Consumer<ResponseCookie.ResponseCookieBuilder> cookieCustomizer = (builder) -> {
    };

    public CookieCsrfTokenRepository() {
    }

    public void setCookieCustomizer(Consumer<ResponseCookie.ResponseCookieBuilder> cookieCustomizer) {
        Assert.notNull(cookieCustomizer, "cookieCustomizer must not be null");
        this.cookieCustomizer = cookieCustomizer;
    }

    public CsrfToken generateToken(HttpServletRequest request) {
        return new DefaultCsrfToken(this.headerName, this.parameterName, this.createNewToken());
    }

    public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {
        String tokenValue = token != null ? token.getToken() : "";
        ResponseCookie.ResponseCookieBuilder cookieBuilder = ResponseCookie.from(this.cookieName, tokenValue).secure(this.secure != null ? this.secure : request.isSecure()).path(StringUtils.hasLength(this.cookiePath) ? this.cookiePath : this.getRequestContext(request)).maxAge(token != null ? (long)this.cookieMaxAge : 0L).httpOnly(this.cookieHttpOnly).domain(this.cookieDomain);
        this.cookieCustomizer.accept(cookieBuilder);
        Cookie cookie = this.mapToCookie(cookieBuilder.build());
        response.addCookie(cookie);
        if (!StringUtils.hasLength(tokenValue)) {
            request.setAttribute(CSRF_TOKEN_REMOVED_ATTRIBUTE_NAME, Boolean.TRUE);
        } else {
            request.removeAttribute(CSRF_TOKEN_REMOVED_ATTRIBUTE_NAME);
        }

    }

    public CsrfToken loadToken(HttpServletRequest request) {
        if (Boolean.TRUE.equals(request.getAttribute(CSRF_TOKEN_REMOVED_ATTRIBUTE_NAME))) {
            return null;
        } else {
            Cookie cookie = WebUtils.getCookie(request, this.cookieName);
            if (cookie == null) {
                return null;
            } else {
                String token = cookie.getValue();
                return !StringUtils.hasLength(token) ? null : new DefaultCsrfToken(this.headerName, this.parameterName, token);
            }
        }
    }

    public void setParameterName(String parameterName) {
        Assert.notNull(parameterName, "parameterName cannot be null");
        this.parameterName = parameterName;
    }

    public void setHeaderName(String headerName) {
        Assert.notNull(headerName, "headerName cannot be null");
        this.headerName = headerName;
    }

    public void setCookieName(String cookieName) {
        Assert.notNull(cookieName, "cookieName cannot be null");
        this.cookieName = cookieName;
    }

    /** @deprecated */
    @Deprecated(
        since = "6.1"
    )
    public void setCookieHttpOnly(boolean cookieHttpOnly) {
        this.cookieHttpOnly = cookieHttpOnly;
    }

    private String getRequestContext(HttpServletRequest request) {
        String contextPath = request.getContextPath();
        return contextPath.length() > 0 ? contextPath : "/";
    }

    public static CookieCsrfTokenRepository withHttpOnlyFalse() {
        CookieCsrfTokenRepository result = new CookieCsrfTokenRepository();
        result.cookieHttpOnly = false;
        return result;
    }

    private String createNewToken() {
        return UUID.randomUUID().toString();
    }

    private Cookie mapToCookie(ResponseCookie responseCookie) {
        Cookie cookie = new Cookie(responseCookie.getName(), responseCookie.getValue());
        cookie.setSecure(responseCookie.isSecure());
        cookie.setPath(responseCookie.getPath());
        cookie.setMaxAge((int)responseCookie.getMaxAge().getSeconds());
        cookie.setHttpOnly(responseCookie.isHttpOnly());
        if (StringUtils.hasLength(responseCookie.getDomain())) {
            cookie.setDomain(responseCookie.getDomain());
        }

        if (StringUtils.hasText(responseCookie.getSameSite())) {
            cookie.setAttribute("SameSite", responseCookie.getSameSite());
        }

        return cookie;
    }

    public void setCookiePath(String path) {
        this.cookiePath = path;
    }

    public String getCookiePath() {
        return this.cookiePath;
    }

    /** @deprecated */
    @Deprecated(
        since = "6.1"
    )
    public void setCookieDomain(String cookieDomain) {
        this.cookieDomain = cookieDomain;
    }

    /** @deprecated */
    @Deprecated(
        since = "6.1"
    )
    public void setSecure(Boolean secure) {
        this.secure = secure;
    }

    /** @deprecated */
    @Deprecated(
        since = "6.1"
    )
    public void setCookieMaxAge(int cookieMaxAge) {
        Assert.isTrue(cookieMaxAge != 0, "cookieMaxAge cannot be zero");
        this.cookieMaxAge = cookieMaxAge;
    }
}
```

可以看到里面主要有三个方法，一个是generateToken，saveToken，loadToken见名思义，一个是生成令牌，一个是保存令牌，最后大概是校验令牌了，csrfFilter 的处理流程很清晰，当一个请求到达时，首先会调用csrfTokenRepository 的loadToken方法加载该会话的CsrfToken值。如果加载不到，则证明请求是首次发起的，应该生成并保存一个新的CsrfToken 值。如果可以加载到CsrfToken 值，那么先排除部分不需要验证CSRF攻击的请求方法（默认忽略了GET、HEAD、TRACE和OPTIONS）

<img src="/upload/截屏2024-06-24%2022.29.02.png" style="display: inline-block;width:100.0%;height:100.0%" />

可以看到这里把令牌存到cookie中了

<img src="/upload/截屏2024-06-24%2022.29.56.png" style="display: inline-block;width:100.0%;height:100.0%" />

### 4.实现CSRF令牌验证的步骤

当客户端发送带有CSRF令牌的请求到服务器时，Spring Security会自动进行CSRF令牌验证。你不需要显式地在每个请求处理器中验证CSRF令牌，因为Spring Security框架已经集成了这一功能。

如果CSRF令牌验证失败，Spring Security会阻止请求的执行，并返回相应的错误状态码（通常是403 Forbidden），表明请求被拒绝。

### 5.自定义处理 CSRF 验证错误

要处理诸如<span style="font-size: 14.3556px; color: rgb(255, 255, 255)">k</span>`InvalidCsrfTokenException`之类的AccessDeniedException的你可以使用以下配置配置自定义拒绝访问页

``` java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ...
            .exceptionHandling((exceptionHandling) -> exceptionHandling
                .accessDeniedPage("/access-denied")
            );
        return http.build();
    }
}
```

### 6.禁用 CSRF 保护

默认情况下，CSRF 保护是启用的，这会影响 与后台的集成 和 应用程序的 测试。在禁用 CSRF 保护之前，请考虑这 对你的应用程序是否有意义。

你还可以考虑是否只有某些端点不需要 CSRF 保护，并配置忽略规则，如下例所示：

``` java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ...
            .csrf((csrf) -> csrf
                .ignoringRequestMatchers("/api/*")
            );
        return http.build();
    }
}
```

``` java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // ...
            .csrf((csrf) -> csrf.disable());
        return http.build();
    }
}
```

### 7.前后端分离同时开启Csrf认证

实现思路，当我们配置csrf时候，登录地址，不需要拦截，登录成功后会得到一个XSRF-TOKEN，前端发送请求之前获取到这个cookie，并将其添加到请求头中，如下图所示

<img src="/upload/截屏2024-07-21%2022.16.07-vclw.png" style="display: inline-block;width:100.0%;height:100.0%" />

### 8.额外注意事项

\- 确保前端（例如JavaScript应用程序）能够正确地从Cookie中获取CSRF令牌，并将其添加到每个请求的请求头中。

\- 在前后端分离的应用中，跨域请求可能需要特别处理，以确保CSRF令牌的正确传递和验证。

总结来说，Spring Security的CSRF保护机制会自动处理CSRF令牌的生成、发送和验证，你只需确保客户端和服务器之间正确地处理CSRF令牌的传递和使用即可。
