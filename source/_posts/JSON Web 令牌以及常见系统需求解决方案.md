---
title: JSON Web 令牌以及常见系统需求解决方案
date: 2025-02-09 12:02:35
categories:
 - jwt
tags: 
 - jwt
---

**什么是JSON Web令牌？**
JSON Web令牌（JWT）是一个开放标准（ RFC 7519 ），它定义了一种紧凑且具有独立的方式，可将各方之间的信息安全地传输为JSON对象。可以验证和信任此信息，因为它是数字签名的。可以使用RSA或ECDSA使用秘密（带有HMAC算法）或公共/私钥对签名JWT。
尽管可以对JWT进行加密以在各方之间提供保密性，但我们将专注于签名的令牌。签名的令牌可以验证其中包含的索赔的完整性，而加密的令牌则将这些索赔隐藏在其他方中。当使用公共/私钥对签名令牌时，签名还证明只有持有私钥的一方才是签名的一方。

**您什么时候应该使用JSON Web令牌？**
**授权**：这是使用JWT的最常见情况。登录用户后，每个后续请求将包括JWT，允许用户访问该令牌允许的路由，服务和资源。单个标志是当今广泛使用JWT的功能，因为它的开销很小，并且能够轻松地在不同域中使用。
**信息交换**：JSON Web令牌是在各方之间安全传输信息的好方法。因为可以签署JWT（例如，使用公共/私钥对），您可以确保发件人是他们说的。此外，由于使用标头和有效载荷计算签名，您还可以验证内容尚未篡改。

**什么是JSON Web令牌结构？**
JSON Web令牌以紧凑的形式由三个部分组成，这些部分由点（ . ）隔开，这是：
标题，有效载荷，签名
因此，JWT通常看起来如下
xxxxx.yyyyy.zzzzz
让我们分解不同的部分。
- 标题：标头通常由两个部分组成：令牌的类型，即JWT和使用的签名算法，例如HMAC SHA256或RSA。
例如：
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
然后，该JSON是基本64url编码以形成JWT的第一部分。
- 有效载荷
有效载荷：其中包含索赔。索赔是关于实体（通常是用户）和其他数据的语句。索赔有三种类型：注册，公共和私人索赔。
注册索赔：这些是一组预定义的索赔，不是强制性的，但建议提供一组有用的可互操作索赔。其中一些是： ISS （发行人）， EXP （到期时间），子（主题）， AUD （受众）等。
请注意，索赔名称只有三个字符，因为JWT应该是紧凑的。
公开主张：使用JWT的人可以随意定义这些要求。但是为了避免碰撞，应在IANA JSON Web令牌注册表中定义它们，或将其定义为包含抗碰撞命名空间的URI。
私人索赔：这些是为在使用它们达成共识的当事方之间共享信息的自定义主张，既不是注册或公开索赔。
一个示例有效载荷可能是：
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
然后对有效负载进行编码，以形成JSON Web令牌的第二部分。
请注意，对于签名令牌，尽管受到保护，但任何人都可以读取此信息。除非加密，否则请勿将秘密信息放在JWT的有效载荷或标头元素中。
- 签名：要创建签名部分，您必须采用编码的标头，编码的有效载荷，一个秘密，标题中指定的算法，然后签名。
例如，如果要使用HMAC SHA256算法，则将以以下方式创建签名：
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
该签名用于验证该消息在此过程中没有更改，并且，对于用私钥签名的令牌，它还可以验证JWT的发件人是否是它说的。将所有人放在一起
输出是三个基本的64-url字符串，这些字符串被点隔开，可以在HTML和HTTP环境中轻松传递，而与基于XML的标准（例如SAML）相比，它更加紧凑。
以下显示了一个JWT，该JWT已编码了先前的标头和有效载荷，并且签名为秘密
![截屏2025-02-09 12.07.50.png](/upload/截屏2025-02-09%2012.07.50.png)

**JSON网络令牌如何工作？**
在身份验证中，当用户成功使用其凭据登录时，将返回JSON Web令牌。由于令牌是凭证，因此必须格外小心以防止安全问题。通常，您不应将令牌保留的时间超过要求。由于缺乏安全性，您也不应将敏感的会话数据存储在浏览器存储中。
每当用户想要访问受保护的路由或资源时，用户代理都应使用载体模式在授权标题中发送JWT。标头的内容看起来如下：
Authorization: Bearer <token>在某些情况下，这可能是无状态授权机制。服务器的受保护路线将在Authorization标头中检查有效的JWT，如果存在，则将允许用户访问受保护的资源。如果JWT包含必要的数据，则需要减少数据库以查询数据库以减少某些操作，尽管情况并非总是如此。
请注意，如果您通过HTTP标头发送JWT令牌，则应尝试防止它们变得太大。一些服务器在标题中的接受不超过8 kb。如果您试图将过多的信息嵌入JWT令牌中，例如包括所有用户的权限，您可能需要一个替代解决方案，例如Auth0 Fine Graining授权。如果令牌是在Authorization标题中发送的，则交叉元素资源共享（CORS）将不会是问题，因为它不使用cookie。以下图显示了如何获得JWT并用于访问API或资
![截屏2025-02-09 12.09.58.png](/upload/截屏2025-02-09%2012.09.58.png)
请注意，在签名令牌中，即使他们无法更改它，代币中包含的所有信息都暴露于用户或其他方。这意味着您不应将秘密信息放在令牌中

**需求解决方案，令牌过期怎么办（自动续期）？**
1. 单token模式
将 token 过期时间设置为15分钟，前端发起请求，后端验证 token 是否过期；如果过期，前端发起刷新token请求，后端为前端返回一个新的token；前端用新的token发起请求，请求成功；如果要实现每隔72小时，必须重新登录，后端需要记录每次用户的登录时间；用户每次请求时，检查用户最后一次登录日期，如超过72小时，则拒绝刷新token的请求，请求失败，跳转到登录页面。
另外后端还可以记录刷新token的次数，比如最多刷新50次，如果达到50次，则不再允许刷新，需要用户重新授权。
上面介绍的单token方案原理比较简单
```
public static Integer verifyToken(String token) {
        byte[] bytes = generateKey().getBytes();
        Key secretKey = Keys.hmacShaKeyFor(bytes);

        try {
            Jws<Claims> claimsJws=Jwts.parser()
                    .setSigningKey(secretKey)
                    .build()
                    .parseClaimsJws(token);
            // Token 验证通过，可以从 claimsJws 对象中获取相关信息
            Claims claims = claimsJws.getBody();
            String id=claims.get("id").toString();
            return Integer.valueOf(id);
        }catch (Exception e){
            // 捕获异常，判断是否是因为过期
            if (e instanceof io.jsonwebtoken.ExpiredJwtException) {
                // 获取过期的 Claims
                Claims claims = ((io.jsonwebtoken.ExpiredJwtException) e).getClaims();
                // 判断是否在允许的刷新时间内
                long now = System.currentTimeMillis();
                long exp = claims.getExpiration().getTime();
                long diff = now - exp;
                // 如果过期时间不超过一定分钟数（例如 30 分钟）
                if (diff <= 30 * 60 * 1000) {
                    // 重新生成新的令牌
                    Map<String, Object> newClaims = new HashMap<>();
                    newClaims.put("id", claims.get("id"));
                    newClaims.put("exp", new Date(now + 60 * 60 * 1000)); // 设置新的过期时间为 1 小时
                    String newToken = Jwts.builder()
                            .setClaims(newClaims)
                            .signWith(secretKey)
                            .compact();
                    System.out.println("New token generated: " + newToken);
                    return Integer.valueOf(claims.get("id").toString());
                }
            }
            return -1;
        }
    }
```
2. 双token模式
登录成功以后，后端返回 access_token 和 refresh_token，客户端缓存此两种token;
使用 access_token 请求接口资源，成功则调用成功；如果token超时，客户端携带 refresh_token 调用token刷新接口获取新的 access_token;
后端接受刷新token的请求后，检查 refresh_token 是否过期。如果过期，拒绝刷新，客户端收到该状态后，跳转到登录页；如果未过期，生成新的 access_token 返回给客户端。
客户端携带新的 access_token 重新调用上面的资源接口。
客户端退出登录或修改密码后，注销旧的token，使 access_token 和 refresh_token 失效，同时清空客户端的 access_token 和 refresh_toke

**问题：双token模式是不是可以看作Oauth2模式呢？**
回顾一下Oauth2，它是用户先同意后，先申请授权码code，然后用code去申请token，然后用户接收到返回的token（这个token持久化时间只需要很短，请求认证通过就可以删除掉）到服务端检验，如果通过，则OAuth认证通过，很显然两者有一定区别
