# SpringSecurity的整合应用

## 认证与授权

在我们的软件系统中，开发人员通常把访问的内容定义为资源，而安全性设计就是对这些资源进行保护。例如在web应用程序

中，对外暴露的HTTP端点就可以被理解成资源。在了解这些技术体系之前，先来说明两个比较混淆的概念

认证（ **authentication**）和授权（**authorization**）。

所谓的**认证**，解决的是“你是谁“的这一个问题。关于验证你的凭据，如用户名/邮箱和密码，以验证访问者的身份。一旦明确了“你是谁“，那么下一步就是判断“你能做什么”，这个步骤就是“**授权**”。

![image-20230312123447726](https://gitee.com/studentgitee/note-picture/raw/master/image-20230312123447726.png)



## 什么是OAuth2.0协议？

### 简述

OAuth2.0（开放授权）是一个关于**授权**的开放的网络协议。允许用户授权第三方应用访问他们存储在另外的服务提供者上的资源

（如：照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

比如使用登录百度（第三方应用）的时候，使用QQ（服务提供者）来登录，百度对于QQ方来说属于第三方应用。

### OAuth2协议的角色

Oauth2协议中定义了4种角色，分别为资源拥有者（Resouece Owner）、客户端（client）、资源服务器（Resource Server）、授权服务器（Authorization Server）。

![image-20230311195131585](https://gitee.com/studentgitee/note-picture/raw/master/image-20230311195131585.png)

1、资源拥有者

能够授予对受保护资源的访问权限的实体，如果资源的所有者为个人，也被成为最终用户（比如拥有QQ的账号密码）。

2、客户端

案例中的百度代表的是第三方应用程序，通常被称为客户端。

3、与客户端相对应的是，OAuth2协议中还存在一个服务提供商，案例中的QQ扮演了该角色。服务提供商拥有一个资源服务器和授权服务器。其中

- 资源服务器：存在用户资源，比如QQ中的头像图片、昵称、id等。、
- 授权服务器：完成这对用户的授权流程，验证资源拥有者并获取授权成功后，向客户颁发访问令牌



### OAuth2协议的令牌

令牌是OAuth中的核心组件，它代表了一次授权的结果，包含了资源拥有者、授权服务器、客户端、受保护资源、权限范围等信息。

如下所示的就是一种常见的令牌信息

```java
{
    "access_token": "U6mWxkh6BLodmRvgMKOWn9mZaOo",
    "token_type": "bearer",
    "refresh_token": "6xKVnEK3I9vQ1g0gjoVXI27fKj4",
    "expires_in": 43199,
    "scope": "all"
}
```

- access_token：代表的是OAuth2的令牌，当访问每个受保护的资源时，用户都需要携带该令牌进行验证。
- token_teyp：代表令牌类型，OAuth2协议中有许多可选的令牌类型，包括bearer类型、mac类型等。
- expires_in：拥有指定access_token的过期时间，一旦查过该有效时间，access_token将会自动失效。
- refresh_token：用于当access_token过期之后重新颁发下一个新的access_token。
- scope：指定可访问的权限范围。

### OAuth2认证流程

```java
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

   (A)  用户打开客户端以后，客户端要求用户给予授权。
（B）用户同意给予客户端授权。
（C）客户端使用上一步获得的授权，向认证服务器申请令牌。
（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
（E）客户端使用令牌，向资源服务器申请获取资源。
（F）资源服务器确认令牌无误，同意向客户端开放资源。

上面可以看出（B）步骤是最重要的，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。

### OAuth2协议的四种授权模式

#### 授权码模式

> 功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

|     参数      |                             解释                             |
| :-----------: | :----------------------------------------------------------: |
| response_type |             必须，固定为code，表示要求返回授权码             |
|   client_id   | 必须，在QQ授权服务器注册后得到的客户端标识，让 QQ授权服务器知道是谁在请求 |
| client_secret |                          客户端密钥                          |
| redirect_uri  |              可选， 接受或拒绝请求后的跳转网址               |
|     scope     |                   可选，表示请求资源的范围                   |

![image-20230311204219915](https://gitee.com/studentgitee/note-picture/raw/master/image-20230311204219915.png)

首先，用户在访问客户端的时候会被导向授权服务器，这时候，用户可以选择是否向客户端授权。

其次，一旦用户同意授权，授权服务器会调用客户端注册时所提供的回调地址，并在调用的程中将授权码直接返回给到客户端。

再次，客户端收到授权码后，会携带授权码和客户端密钥再一次向授权服务器请求访问令牌。

最后，授权服务器核对授权码以及客户端密钥，并向客户端发送访问令牌。

#### 简化模式

> 不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。

![image-20230312121955424](https://gitee.com/studentgitee/note-picture/raw/master/image-20230312121955424.png)

#### 密码模式

> 如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。

![image-20230311204857580](https://gitee.com/studentgitee/note-picture/raw/master/image-20230311204857580.png)

可以看到，密码模式比较简单，更加的容易理解。用户要 做的事情就是向客户端提供用户名和密码，然后客户端基于这些用户信息向授权服务器请求访问令牌。授权服务器成功的认证用户的信息后将会颁发令牌。

#### 客户端模式

![image-20230312122126641](https://gitee.com/studentgitee/note-picture/raw/master/image-20230312122126641.png)

## 搭建授权服务器

## 什么JWT令牌？

### 简述

> JWT全称为JSON Web Token，它本质上是一种基于JSON格式表示的令牌。是目前最流行的跨域认证解决方案。

### 跨域认证的问题

互联网服务离不开用户认证。一般流程是下面这样。

> 1、用户向服务器发送用户名和密码。
>
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
>
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
>
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
>
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

> 1、这种模式的问题在于，扩展性差。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 **session **
>
> **数据共享**，每台服务器都能够读取 session。
>
> 2、举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会
>
> 自动登录，请问怎么实现？
>
> 3、一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案
>
> 的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。另一种方案是服务器索性不保存 
>
> session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

### JWT的基本结构

> 从结构上来说，JWT本身有三段信息构成。
>
> 第一段位头部（Header）、第二段为有效负载（Payload）、第三段为签名（Signature）。

写成一行，就是下面的样子

```java
Header.Payload.Signature
```

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJhdWQiOlsicmVzMSJdLCJ1c2VyX25hbWUiOiJ1c2VyIiwic2NvcGUiOlsiYWxsIl0sImV4cCI6MTYzODYwNTcxOCwiYXV0aG9yaXRpZXMiOlsiUk9MRV91c2VyIl0sImp0aSI6ImRkNTVkMjEzLThkMDYtNGY4MC1iMGRmLTdkN2E0YWE2MmZlOSIsImNsaWVudF9pZCI6Im15anN6bCJ9.
koup5-wzGfcSVnaaNfILwAgw2VaTLvRgq2JVnIHYe_Q
```

```java
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2Nzg2MzU0NDEsInVzZXJfbmFtZSI6ImFkbWluIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiIsIlJPTEVfVVNFUiJdLCJqdGkiOiJlSmpHcWlZb1gybWRwSEdrUXdPQjM5MHhJeWsiLCJjbGllbnRfaWQiOiJtZXNzYWdpbmctY2xpZW50Iiwic2NvcGUiOlsiYWxsIl19.rlJNRneuJGaP9PaF47jGwyiHsiw4USwNPXPADZ7kYe8",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsInNjb3BlIjpbImFsbCJdLCJhdGkiOiJlSmpHcWlZb1gybWRwSEdrUXdPQjM5MHhJeWsiLCJleHAiOjE2ODExODQyNDEsImF1dGhvcml0aWVzIjpbIlJPTEVfQURNSU4iLCJST0xFX1VTRVIiXSwianRpIjoiVXhNbDlNQ0wybUdjWGdUakE0b2VNZHN1cmFBIiwiY2xpZW50X2lkIjoibWVzc2FnaW5nLWNsaWVudCJ9.Q91AFIbOqH-Smp8heqYmz02nCjFPYnHpdHAo4-TqhCI",
    "expires_in": 43199,
    "scope": "all",
    "jti": "eJjGqiYoX2mdpHGkQwOB390xIyk"
}
```

access_token和refresh_token都是经过Base64URL进行了加密。需要使用在线解析工具https://tooltt.com/jwt-decode/解析后得到如下结果：

```jwt
{
  "alg": "HS256",
  "typ": "JWT"
}.
{
  "exp": 1678635441,
  "user_name": "admin",
  "authorities": [
    "ROLE_ADMIN",
    "ROLE_USER"
  ],
  "jti": "eJjGqiYoX2mdpHGkQwOB390xIyk",
  "client_id": "messaging-client",
  "scope": [
    "all"
  ]
}.
[signature]
```

**头部(Header)**

```java
{
  "alg": "HS256",
  "typ": "JWT"
}
```

上面代码中，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

**有效负载(Payload)**

```java
{
  "exp": 1678635441,
  "user_name": "admin",
  "authorities": [
    "ROLE_ADMIN",
    "ROLE_USER"
  ],
  "jti": "eJjGqiYoX2mdpHGkQwOB390xIyk",
  "client_id": "messaging-client",
  "scope": [
    "all"
  ]
}
```

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

> - iss (issuer)：签发人
> - exp (expiration time)：过期时间
> - sub (subject)：主题
> - aud (audience)：受众
> - nbf (Not Before)：生效时间
> - iat (Issued At)：签发时间
> - jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。

```jwt
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

**签名（Signature）**

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户。

### 集成OAuth协议与JWT

首先在Maven的pom文件中引入对应的依赖包，该maven包含了jwt的依赖

```maven
        <dependency>
            <groupId>org.springframework.security.oauth</groupId>
            <artifactId>spring-security-oauth2</artifactId>
            <version>2.5.2.RELEASE</version>
        </dependency>
```

通过一个配置类来完成JWT的生成和转换。OAuth2协议专门提供了一个接口用于管理令牌的存储，即TokenStore。我们创建一个用于配置JwtTokenStore的配置类**JwtTokenStoreConfig**，如下所示。

```java
/**
 * @Description 管理jwt令牌存储
 * @Author MrWei
 * @Date 2023/3/12 11:17
 **/
@Configuration
public class JwtTokenStoreConfig {
    /**
     * JWT密钥
     * 实际项目中是放到配置文件里面，资源服务也需要用到
     */
    public final static String SIGN_KEY = "123456";

    /**
     * 令牌的存储策略：使用jwt令牌
     *
     * @return
     */
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        // 对称秘钥，资源服务器使用该秘钥来验证
        jwtAccessTokenConverter.setSigningKey(SIGN_KEY);
        return jwtAccessTokenConverter;
    }

    /**
     * 令牌管理服务
     *
     * @return
     */
    @Bean
    public DefaultTokenServices tokenService() {
        DefaultTokenServices service = new DefaultTokenServices();
        // 是否产生支持刷新令牌
        service.setSupportRefreshToken(true);
        // 令牌存储策略
        service.setTokenStore(tokenStore());
        // access_token令牌桶默认有效期2小时
        service.setAccessTokenValiditySeconds(60 * 60 * 2);
        // 刷新令牌默认有效期3天
        service.setRefreshTokenValiditySeconds(60 * 60 * 24 * 3);
        return service;
    }
}
```

上述**JwtTokenStoreConfig**的作用是创建一系列对象供Spirng容器使用，那么在什么时候会用到这些对象呢？答案是在将JWT集成到OAuth2授权服务的过程中。

通过构建一个配置类来覆写**AuthorizationServerConfigurerAdapter**中的**configure()**方法。在集成JWT之后，该方法的实现需要调整为一下形势。

```java
    /**
     * 配置端点行为
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // 设置令牌增强链
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Collections.singletonList(jwtAccessTokenConverter));

        /**
         * tokenStore:设置 AccessToken的存储介质tokenStore， 默认使用内存当做存储介质。
         * accessTokenConverter:设置jwt对象
         * authenticationManager:在授权模式为密码模式时，需要一个认证管理器，用于对用户名和密码进行认证。
         * userDetailsService：基于密码的授权模式，所以需要指定一个自定义的userDetailsService替换全局的实现。
         */
        endpoints
                .tokenStore(tokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);
    }
```

至此，我们在OAuth2协议中集成JWT。

### 更新令牌

## 什么单点登录SSO？

## 

