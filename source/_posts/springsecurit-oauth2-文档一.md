---
title: "SpringSecurit  OAuth2 （文档一）"
date: 2024-11-18 15:41:52
---

#### 注册ClientRegistrationRepository（bean或配置文件）

``` java
@Bean
ClientRegistrationRepository clientRegistrationRepository() {
    ClientRegistration github = githubClientRegistration();
    ClientRegistration facebook = facebookClientRegistration();
    return new InMemoryClientRegistrationRepository(github, facebook);
}

private ClientRegistration githubClientRegistration() {
    return CommonOAuth2Provider.GITHUB.getBuilder("github").clientId("Ov23liCBLLUjii41pS7k")
            .clientSecret("9da8734b56aad52d91b268fe6834a8df12447d95").build();
}

private ClientRegistration facebookClientRegistration() {
    return CommonOAuth2Provider.FACEBOOK.getBuilder("facebook").clientId("974042741122392")
            .clientSecret("36d48c25c1767d58b3101551513d7e1e").build();
}
```

``` xml
spring.security.oauth2.client.registration.github.client-id=${GITHUB_CLIENT_ID:Ov23liCBLLUjii41pS7k}
spring.security.oauth2.client.registration.github.client-secret=${GITHUB_CLIENT_SECRET:9da8734b56aad52d91b268fe6834a8df12447d95}

spring.security.oauth2.client.registration.facebook.client-id=${GITHUB_CLIENT_ID:974042741122392}
spring.security.oauth2.client.registration.facebook.client-secret=${GITHUB_CLIENT_SECRET:36d48c25c1767d58b3101551513d7e1e}
```

#### springsecurity默认提供四种

``` java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.security.config.oauth2.client;

import org.springframework.security.oauth2.client.registration.ClientRegistration;
import org.springframework.security.oauth2.core.AuthorizationGrantType;
import org.springframework.security.oauth2.core.ClientAuthenticationMethod;

public enum CommonOAuth2Provider {
    GOOGLE {
        public ClientRegistration.Builder getBuilder(String registrationId) {
            ClientRegistration.Builder builder = this.getBuilder(registrationId, ClientAuthenticationMethod.CLIENT_SECRET_BASIC, "{baseUrl}/{action}/oauth2/code/{registrationId}");
            builder.scope(new String[]{"openid", "profile", "email"});
            builder.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth");
            builder.tokenUri("https://www.googleapis.com/oauth2/v4/token");
            builder.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs");
            builder.issuerUri("https://accounts.google.com");
            builder.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo");
            builder.userNameAttributeName("sub");
            builder.clientName("Google");
            return builder;
        }
    },
    GITHUB {
        public ClientRegistration.Builder getBuilder(String registrationId) {
            ClientRegistration.Builder builder = this.getBuilder(registrationId, ClientAuthenticationMethod.CLIENT_SECRET_BASIC, "{baseUrl}/{action}/oauth2/code/{registrationId}");
            builder.scope(new String[]{"read:user"});
            builder.authorizationUri("https://github.com/login/oauth/authorize");
            builder.tokenUri("https://github.com/login/oauth/access_token");
            builder.userInfoUri("https://api.github.com/user");
            builder.userNameAttributeName("id");
            builder.clientName("GitHub");
            return builder;
        }
    },
    FACEBOOK {
        public ClientRegistration.Builder getBuilder(String registrationId) {
            ClientRegistration.Builder builder = this.getBuilder(registrationId, ClientAuthenticationMethod.CLIENT_SECRET_POST, "{baseUrl}/{action}/oauth2/code/{registrationId}");
            builder.scope(new String[]{"public_profile", "email"});
            builder.authorizationUri("https://www.facebook.com/v2.8/dialog/oauth");
            builder.tokenUri("https://graph.facebook.com/v2.8/oauth/access_token");
            builder.userInfoUri("https://graph.facebook.com/me?fields=id,name,email");
            builder.userNameAttributeName("id");
            builder.clientName("Facebook");
            return builder;
        }
    },
    OKTA {
        public ClientRegistration.Builder getBuilder(String registrationId) {
            ClientRegistration.Builder builder = this.getBuilder(registrationId, ClientAuthenticationMethod.CLIENT_SECRET_BASIC, "{baseUrl}/{action}/oauth2/code/{registrationId}");
            builder.scope(new String[]{"openid", "profile", "email"});
            builder.userNameAttributeName("sub");
            builder.clientName("Okta");
            return builder;
        }
    };

    private static final String DEFAULT_REDIRECT_URL = "{baseUrl}/{action}/oauth2/code/{registrationId}";

    private CommonOAuth2Provider() {
    }

    protected final ClientRegistration.Builder getBuilder(String registrationId, ClientAuthenticationMethod method, String redirectUri) {
        ClientRegistration.Builder builder = ClientRegistration.withRegistrationId(registrationId);
        builder.clientAuthenticationMethod(method);
        builder.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE);
        builder.redirectUri(redirectUri);
        return builder;
    }

    public abstract ClientRegistration.Builder getBuilder(String registrationId);
}
```

<img src="/upload/截屏2024-10-31%2014.11.39.png" style="display: inline-block;width:100.0%;height:100.0%" />
