---
title: "OAuth2  Client（核心的接口与类）"
date: 2024-11-18 15:41:52
---

### ClientRegistration

ClientRegistration 是一个在 OAuth 2.0 或 OpenID Connect 1.0 提供商（Provider）处注册的客户端的表示。

ClientRegistration 对象保存的信息包括客户端ID、客户端 secret、授权类型、重定向URI、scope、授权URI、token URI 和其他详细信息。

ClientRegistration 和它的属性定义如下。

``` java
public final class ClientRegistration {
    private String registrationId;  
    private String clientId;    
    private String clientSecret;    
    private ClientAuthenticationMethod clientAuthenticationMethod;  
    private AuthorizationGrantType authorizationGrantType;  
    private String redirectUri; 
    private Set<String> scopes;   
    private ProviderDetails providerDetails;
    private String clientName;  

    public class ProviderDetails {
        private String authorizationUri;    
        private String tokenUri;    
        private UserInfoEndpoint userInfoEndpoint;
        private String jwkSetUri;   
        private String issuerUri;   
        private Map<String, Object> configurationMetadata;  

        public class UserInfoEndpoint {
            private String uri; 
            private AuthenticationMethod authenticationMethod;  
            private String userNameAttributeName;   

        }
    }
}
```

### ClientRegistrationRepository

ClientRegistrationRepository 作为OAuth 2.0 / OpenID Connect 1.0 ClientRegistration 的存储库（repository）。

客户端注册信息最终由相关的授权服务器存储和拥有。这个存储库提供了检索主要客户注册信息子集的能力，这些信息存储在授权服务器上。

Spring Boot 2.x的自动配置将 <span color="#ef4444" style="color: #ef4444">spring.security.oauth2.client.registration.\[registrationId\] 下的每个属性绑定到 ClientRegistration 实例上，然后将每个 ClientRegistration 实例组合到 ClientRegistrationRepository 中。</span>

ClientRegistrationRepository 的默认实现是 InMemoryClientRegistrationRepository

<img src="/upload/截屏2024-10-31%2014.32.20.png" style="display: inline-block;width:100.0%;height:100.0%" />

这个类接收ClientRegistration 实例，然后将其放到map中<img src="/upload/截屏2024-10-31%2014.33.23.png" style="display: inline-block;width:100.0%;height:100.0%" />

自动配置也将 ClientRegistrationRepository 注册为 ApplicationContext 中的 @Bean，这样，如果应用程序需要，它就可以进行依赖注入。

``` java
@Controller
public class OAuth2ClientController {

    @Autowired
    private ClientRegistrationRepository clientRegistrationRepository;

    @GetMapping("/")
    public String index() {
        ClientRegistration oktaRegistration =
            this.clientRegistrationRepository.findByRegistrationId("okta");

        ...

        return "index";
    }
}
```

<span color="rgb(255, 255, 255)" fontsize="" style="color: rgb(255, 255, 255)">C</span>

### OAuth2AuthorizedClient

OAuth2AuthorizedClient 是一个 授权 客户端的代表。当终端用户（资源所有者）已经授权给客户端访问其受保护的资源时，客户端就被认为是被授权的。

OAuth2AuthorizedClient 的作用是将 OAuth2AccessToken（和可选的 OAuth2RefreshToken）与 ClientRegistration（client）和资源所有者联系起来，后者是许可授权的主要终端用户。<span color="rgb(255, 255, 255)" fontsize="" style="color: rgb(255, 255, 255)">opied!</span>

### OAuth2AuthorizedClientRepository 和 OAuth2AuthorizedClientService

OAuth2AuthorizedClientRepository 负责在网络请求之间持久保存\`OAuth2AuthorizedClient\`，而 OAuth2AuthorizedClientService 的主要作用是在应用层面管理 OAuth2AuthorizedClient。

从开发者的角度来看，OAuth2AuthorizedClientRepository 或 OAuth2AuthorizedClientService 提供了查询与客户端相关的 OAuth2AccessToken 的能力，从而可以用来启动受保护资源的请求。

``` java
@Controller
public class OAuth2ClientController {

    @Autowired
    private OAuth2AuthorizedClientService authorizedClientService;

    @GetMapping("/")
    public String index(Authentication authentication) {
        OAuth2AuthorizedClient authorizedClient =
            this.authorizedClientService.loadAuthorizedClient("okta", authentication.getName());

        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();

        ...

        return "index";
    }
}
```

<span color="rgb(255, 255, 255)" fontsize="" style="color: rgb(255, 255, 255)">Copied!</span>

Spring Boot 2.x的自动配置在 ApplicationContext 中注册了一个 OAuth2AuthorizedClientRepository 或一个 OAuth2AuthorizedClientService @Bean。但是，应用程序可以覆盖并注册一个自定义的 OAuth2AuthorizedClientRepository 或 OAuth2AuthorizedClientService @Bean。

OAuth2AuthorizedClientService 的默认实现是 InMemoryOAuth2AuthorizedClientService，它在内存中存储 OAuth2AuthorizedClient 对象。

另外，你可以配置JDBC实现 JdbcOAuth2AuthorizedClientService，将 OAuth2AuthorizedClient 实例持久化在数据库中。

JdbcOAuth2AuthorizedClientService 依赖于 OAuth 2.0 Client Schema 中描述的表定义，在数据库创建表结构，脚本如下

``` sql
CREATE TABLE oauth2_authorized_client (
  client_registration_id varchar(100) NOT NULL,
  principal_name varchar(200) NOT NULL,
  access_token_type varchar(100) NOT NULL,
  access_token_value blob NOT NULL,
  access_token_issued_at timestamp NOT NULL,
  access_token_expires_at timestamp NOT NULL,
  access_token_scopes varchar(1000) DEFAULT NULL,
  refresh_token_value blob DEFAULT NULL,
  refresh_token_issued_at timestamp DEFAULT NULL,
  created_at timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL,
  PRIMARY KEY (client_registration_id, principal_name)
);
```

注入JdbcOAuth2AuthorizedClientService，第三方授权登录，数据库会保存数据

``` java
@Bean
    public OAuth2AuthorizedClientService oAuth2AuthorizedClientService(ClientRegistrationRepository clientRegistrationRepository) {
        return new JdbcOAuth2AuthorizedClientService(jdbcTemplate, clientRegistrationRepository);
    }
```

<img src="/upload/截屏2024-10-31%2016.15.15.png" style="display: inline-block;width:100.0%;height:100.0%" />

当我们下次授权登录时候，会更新这条记录

### OAuth2AuthorizedClientManager 和 OAuth2AuthorizedClientProvider

OAuth2AuthorizedClientManager 负责 OAuth2AuthorizedClient 的整体管理。

其主要职责包括

- 通过使用 OAuth2AuthorizedClientProvider，授权（或重新授权）一个OAuth 2.0客户端。

- 委托一个 OAuth2AuthorizedClient 的持久性，通常是通过使用一个 OAuth2AuthorizedClientService 或 OAuth2AuthorizedClientRepository 。

- 当一个OAuth 2.0客户端被成功授权（或重新授权）时，委托给一个 OAuth2AuthorizationSuccessHandler。

- 当OAuth2.0客户端无法授权（或重新授权）时，委托给一个 OAuth2AuthorizationFailureHandler。

一个 OAuth2AuthorizedClientProvider 实现了授权（或重新授权）OAuth 2.0客户端的策略。实现通常实现一个授权许可类型，如 authorization_code、 client_credentials 等。

OAuth2AuthorizedClientManager 的默认实现是 DefaultOAuth2AuthorizedClientManager，它与一个 OAuth2AuthorizedClientProvider 相关联，该 Provider 可以使用基于 delegation 的复合方式支持多种授权许可类型。你可以使用 OAuth2AuthorizedClientProviderBuilder 来配置和建立基于 delegation 的复合。

下面的代码显示了一个如何配置和构建 OAuth2AuthorizedClientProvider 复合体的例子，它提供了对 authorization_code、refresh_token、 client_credentials 和 password 授权许可类型的支持。

``` java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
        ClientRegistrationRepository clientRegistrationRepository,
        OAuth2AuthorizedClientRepository authorizedClientRepository) {

    OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                    .authorizationCode()
                    .refreshToken()
                    .clientCredentials()
                    .password()
                    .build();

    DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                    clientRegistrationRepository, authorizedClientRepository);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    return authorizedClientManager;
}
```

当 授权尝试成功时<span color="#f87171" style="color: #f87171">，DefaultOAuth2AuthorizedClientManager 委托给 OAuth2AuthorizationSuccessHandler，后者（默认）通过 OAuth2AuthorizedClientRepository 保存 OAuth2AuthorizedClient。在重新授权失败的情况下（例如，刷新令牌不再有效），先前保存的 OAuth2AuthorizedClient 会通过 RemoveAuthorizedClientOAuth2AuthorizationFailureHandler 从 OAuth2AuthorizedClientRepository 中删除。你可以通过 setAuthorizationSuccessHandler(OAuth2AuthorizationSuccessHandler) 和 setAuthorizationFailureHandler(OAuth2AuthorizationFailureHandler) 自定义默认行为</span>。

DefaultOAuth2AuthorizedClientManager 还与一个类型为 Function\<OAuth2AuthorizeRequest, Map\<String, Object\> 的 contextAttributesMapper 相关联，它负责将 OAuth2AuthorizeRequest 中的属性映射到与 OAuth2AuthorizationContext 相关联的属性Map中。当你需要向 OAuth2AuthorizedClientProvider 提供所需的（支持的）属性时，这可能很有用，例如，PasswordOAuth2AuthorizedClientProvider 要求资源所有者的用户名和密码在 OAuth2AuthorizationContext.getAttributes() 中可用。

下面的代码显示了 contextAttributesMapper 的一个例子。

``` java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
        ClientRegistrationRepository clientRegistrationRepository,
        OAuth2AuthorizedClientRepository authorizedClientRepository) {

    OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                    .password()
                    .refreshToken()
                    .build();

    DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                    clientRegistrationRepository, authorizedClientRepository);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    // Assuming the `username` and `password` are supplied as `HttpServletRequest` parameters,
    // map the `HttpServletRequest` parameters to `OAuth2AuthorizationContext.getAttributes()`
    authorizedClientManager.setContextAttributesMapper(contextAttributesMapper());

    return authorizedClientManager;
}

private Function<OAuth2AuthorizeRequest, Map<String, Object>> contextAttributesMapper() {
    return authorizeRequest -> {
        Map<String, Object> contextAttributes = Collections.emptyMap();
        HttpServletRequest servletRequest = authorizeRequest.getAttribute(HttpServletRequest.class.getName());
        String username = servletRequest.getParameter(OAuth2ParameterNames.USERNAME);
        String password = servletRequest.getParameter(OAuth2ParameterNames.PASSWORD);
        if (StringUtils.hasText(username) && StringUtils.hasText(password)) {
            contextAttributes = new HashMap<>();

            // `PasswordOAuth2AuthorizedClientProvider` requires both attributes
            contextAttributes.put(OAuth2AuthorizationContext.USERNAME_ATTRIBUTE_NAME, username);
            contextAttributes.put(OAuth2AuthorizationContext.PASSWORD_ATTRIBUTE_NAME, password);
        }
        return contextAttributes;
    };
}
```

DefaultOAuth2AuthorizedClientManager 被设计为在 HttpServletRequest 的上下文中（context）使用。当在 HttpServletRequest 上下文之外操作时，请使用 AuthorizedClientServiceOAuth2AuthorizedClientManager 代替。

对于何时使用 AuthorizedClientServiceOAuth2AuthorizedClientManager，服务 应用是一个常见的用例。服务 应用程序通常在后台运行，没有任何用户互动，并且通常在系统级账户而不是用户账户下运行。一个配置了 client_credentials 授予类型的OAuth 2.0客户端可以被认为是一种服务应用程序。

下面的代码显示了一个如何配置 AuthorizedClientServiceOAuth2AuthorizedClientManager 的例子，它提供对 client_credentials 授予类型的支持。

``` java
@Bean
public OAuth2AuthorizedClientManager authorizedClientManager(
        ClientRegistrationRepository clientRegistrationRepository,
        OAuth2AuthorizedClientService authorizedClientService) {

    OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                    .clientCredentials()
                    .build();

    AuthorizedClientServiceOAuth2AuthorizedClientManager authorizedClientManager =
            new AuthorizedClientServiceOAuth2AuthorizedClientManager(
                    clientRegistrationRepository, authorizedClientService);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

    return authorizedClientManager;
}
```
