---
title: "一口气了解OAuth"
date: 2025-06-11 19:23:37
categories: OAuth
tags: OAuth
---

### 一口气了解OAuth

##### OAuth2.0为何物？

OAuth<span style="font-size: 16px; color: rgb(74, 74, 74)"> 简单理解就是一种授权机制，它是在客户端和资源所有者之间的授权层，用来分离两种不同的角色。在资源所有者同意并向客户端颁发令牌后，客户端携带令牌可以访问资源所有者的资源；</span>

OAuth2.0<span style="font-size: 16px; color: rgb(74, 74, 74)"> 是</span>OAuth<span style="font-size: 16px; color: rgb(74, 74, 74)"> 协议的一个版本，有</span>2.0<span style="font-size: 16px; color: rgb(74, 74, 74)">版本那就有</span>1.0<span style="font-size: 16px; color: rgb(74, 74, 74)">版本，有意思的是</span>OAuth2.0<span style="font-size: 16px; color: rgb(74, 74, 74)"> 却不向下兼容</span>OAuth1.0<span style="font-size: 16px; color: rgb(74, 74, 74)"> ，相当于废弃了</span>1.0<span style="font-size: 16px; color: rgb(74, 74, 74)">版本</span>

##### 举个小例子解释一下什么是 OAuth 授权？

<span style="font-size: 16px; color: rgb(74, 74, 74)">我定了一个外卖，外卖小哥30秒火速到达了我家楼下，奈何有门禁进不来，可以输入密码进入，但出于安全的考虑我并不想告诉他密码</span>

<span style="font-size: 16px; color: rgb(74, 74, 74)">此时外卖小哥看到门禁有一个高级按钮“</span>**一键获取授权**<span style="font-size: 16px; color: rgb(74, 74, 74)">”，只要我这边同意，他会获取到一个有效期 2小时的令牌（</span>token<span style="font-size: 16px; color: rgb(74, 74, 74)">）正常出入。</span>

<img src="/upload/截屏2024-04-27%2019.06.05.png" style="display: inline-block;width:100.0%;height:100.0%" /><span style="font-size: 16px; color: rgb(74, 74, 74)">令牌（</span>token<span style="font-size: 16px; color: rgb(74, 74, 74)">）和 </span>密码<span style="font-size: 16px; color: rgb(74, 74, 74)"> 的作用虽然相似都可以进入系统，但还有点不同。</span>token<span style="font-size: 16px; color: rgb(74, 74, 74)"> 拥有权限范围，有时效性的，到期自动失效，而且无效修改</span>

### <span style="font-size: 16px; color: rgb(74, 74, 74)">OAuth2授权模式</span>

`OAuth2.0` 的授权简单理解其实就是获取令牌（`token`）的过程，`OAuth` 协议定义了四种获得令牌的授权方式（`authorization grant` ）如下：

- 授权码（`authorization-code`）

- 隐藏式（`implicit`）

- 密码式（`password`）：

- 客户端凭证（`client credentials`）

但值得注意的是，不管我们使用哪一种授权方式，在三方应用申请令牌之前，都必须在系统中去申请身份唯一标识：客户端 ID（`client ID`）和 客户端密钥（`client secret`）。这样做可以保证 `token` 不被恶意使用。

下面我们会分析每种授权方式的原理，在进入正题前，先了解 `OAuth2.0` 授权过程中几个重要的参数：

- `response_type`：code 表示要求返回授权码，token 表示直接返回令牌

- `client_id`：客户端身份标识

- `client_secret`：客户端密钥

- `redirect_uri`：重定向地址

- `scope`：表示授权的范围，`read`只读权限，`all`读写权限

- `grant_type`：表示授权的方式，`AUTHORIZATION_CODE`（授权码）、`password`（密码）、`client_credentials`（凭证式）、`refresh_token` 更新令牌

- `state`：应用程序传递的一个随机数，用来防止`CSRF`攻击

##### 授权码

OAuth2.0四种授权中授权码方式是最为复杂，但也是安全系数最高的，比较常用的一种方式。这种方式适用于兼具前后端的Web项目，因为有些项目只有后端或只有前端，并不适用授权码模式。

下图我们以用WX登录掘金为例，详细看一下授权码方式的整体流程

<img src="/upload/截屏2024-04-27%2019.09.38.png" style="display: inline-block;width:100.0%;height:100.0%" />

<span style="font-size: 16px; color: rgb(74, 74, 74)">用户选择</span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)">登录掘金，掘金会向</span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)">发起授权请求，接下来 </span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)">询问用户是否同意授权（常见的弹窗授权）。</span>response_type<span style="font-size: 16px; color: rgb(74, 74, 74)"> 为 </span>code<span style="font-size: 16px; color: rgb(74, 74, 74)"> 要求返回授权码，</span>scope<span style="font-size: 16px; color: rgb(74, 74, 74)"> 参数表示本次授权范围为只读权限，</span>redirect_uri<span style="font-size: 16px; color: rgb(74, 74, 74)"> 重定向地址</span>

``` html
https://wx.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=http://juejin.im/callback&
  scope=read
```

<span style="font-size: 16px; color: rgb(74, 74, 74)">用户同意授权后，</span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)"> 根据 </span>redirect_uri<span style="font-size: 16px; color: rgb(74, 74, 74)">重定向并带上授权码</span>

``` html
http://juejin.im/callback?code=AUTHORIZATION_CODE
```

<span style="font-size: 16px; color: rgb(74, 74, 74)">当掘金拿到授权码（code）时，带授权码和密匙等参数向</span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)">申请令牌。</span>grant_type<span style="font-size: 16px; color: rgb(74, 74, 74)">表示本次授权为授权码方式 </span>authorization_code<span style="font-size: 16px; color: rgb(74, 74, 74)"> ，获取令牌要带上客户端密匙 </span>client_secret<span style="font-size: 16px; color: rgb(74, 74, 74)">，和上一步得到的授权码 </span>code

    https://wx.com/oauth/token?
     client_id=CLIENT_ID&
     client_secret=CLIENT_SECRET&
     grant_type=authorization_code&
     code=AUTHORIZATION_CODE&
     redirect_uri=http://juejin.im/callback

<span style="font-size: 16px; color: rgb(74, 74, 74)">最后 </span>WX<span style="font-size: 16px; color: rgb(74, 74, 74)"> 收到请求后向 </span>redirect_uri<span style="font-size: 16px; color: rgb(74, 74, 74)"> 地址发送 </span>JSON<span style="font-size: 16px; color: rgb(74, 74, 74)"> 数据，其中的</span>access_token<span style="font-size: 16px; color: rgb(74, 74, 74)"> 就是令牌</span>

     {    
      "access_token":"ACCESS_TOKEN",
      "token_type":"bearer",
      "expires_in":2592000,
      "refresh_token":"REFRESH_TOKEN",
      "scope":"read",
      ......
    }

##### 隐藏式

上边提到有一些`Web`应用是没有后端的， 属于纯前端应用，无法用上边的授权码模式。令牌的申请与存储都需要在前端完成，跳过了授权码这一步。

前端应用直接获取 `token`，`response_type` 设置为 `token`，要求直接返回令牌，跳过授权码，`WX`授权通过后重定向到指定 `redirect_uri`

    https://wx.com/oauth/authorize?
      response_type=token&
      client_id=CLIENT_ID&
      redirect_uri=http://juejin.im/callback&
      scope=read

##### 密码式

    https://wx.com/token?
      grant_type=password&
      username=USERNAME&
      password=PASSWORD&
      client_id=CLIENT_ID

<span style="font-size: 16px; color: rgb(74, 74, 74)">这种授权方式缺点是显而易见的，非常的危险，如果采取此方式授权，该应用一定是可以高度信任的。</span>

##### <span style="font-size: 17px">凭证式</span>

凭证式和密码式很相似，主要适用于那些没有前端的命令行应用，可以用最简单的方式获取令牌，在请求响应的 `JSON` 结果中返回 `token`。

`grant_type` 为 `client_credentials` 表示凭证式授权，`client_id` 和 `client_secret` 用来识别身份。

    https://wx.com/token?
      grant_type=client_credentials&
      client_id=CLIENT_ID&
      client_secret=CLIENT_SECRET

参考文章：<a href="https://mp.weixin.qq.com/s/in_E1pKqQc8wkPXT61g8gQ" rel="noopener noreferrer nofollow" target="_blank">https://mp.weixin.qq.com/s/in_E1pKqQc8wkPXT61g8gQ</a>

Oauth认证过程演练：<a href="https://www.oauth.com/playground/" target="_blank">https://www.oauth.com/playground/</a>
