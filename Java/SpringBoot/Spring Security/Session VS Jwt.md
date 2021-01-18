# Session  vs Jwt

通过session来记录用户认证信息的方式我们可以理解为一种有状态登陆,而JWT则代表了一种无状态登陆.

## JWT介绍

jwt = json web token ,是一种json风格的轻量级的授权和身份认证规范,可以实现无状态,分布式的web应用授权

JWT 作为一种规范，并没有和某一种语言绑定在一起，常用的 Java 实现是 GitHub 上的开源项目 jjwt，地址如下：`https://github.com/jwtk/jjwt`

### JWT 数据格式

- Header

  - 声明类型 JWT
  - 加密算法 自定义

  我们会对头部进行base64url编码(可解码),得到第一部分数据

- Payload 荷载 就是有效数据,在官方文档(RFC7519),这里给了7个示例信息

  - iss(issuser)
  - exp(expiration time)
  - sub(subject)
  - aud(audience)
  - nbf(not before)
  - iat(Issued At)
  - jti(JWT id)

  这部分也会采用base4url编码,可以得到第二部分数据

- Signature 签名,是整个数据的认证信息.一般根据前两步的数据，再加上服务的的密钥 secret（密钥保存在服务端，不能泄露给客户端），通过 Header 中配置的加密算法生成。用于验证整个数据完整和可靠性。

JWT 生成的数据格式如下

```
"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOlsicmVzMSJdLCJ1c2VyX25hbWUiOiJqYXZhYm95Iiwic2NvcGUiOlsiYWxsIl0sImV4cCI6MTYxMTAyNzU0MSwiYXV0aG9yaXRpZXMiOlsiUk9MRV91c2VyIl0sImp0aSI6ImZkZDY4YjQyLTE1MmMtNGI0NC05YTEwLWFhZjEyMTRkNTYzNSIsImNsaWVudF9pZCI6ImphdmFib3kifQ._ETBwzH0MyUNX2yCzgtB9Siudws_IVx5LMTTaQ5RDxA"
```

### JWT交互流程

1. 应用程序或客户端向授权服务器请求授权
2. 获取授权后,授权服务器会向应用程序返回访问令牌
3. 应用程序使用访问令牌来访问受保护资源

```
因为JWT签发的token中已经包含了用户的身份信息,并且每次请求都会携带,这样服务就无需保存用户信息,设置无需去查询数据,这样就符合了Restful的无状态规范
```

### JWT 存在的问题

* 续签问题
* 注销问题
* 密码重置
* 建议不同用户取不同的secret

## 有状态 VS 无状态

### 有状态

有状态服务,即服务端需要记住每次会话的客户端信息,从而识别客户端身份,根据身份进行请求处理,典型设计比如Tomcat中的session,例如登录：用户登录后，我们把用户的信息保存在服务端 session 中，并且给用户一个 cookie 值，记录对应的 session，然后下次请求，用户携带 cookie 值来（这一步有浏览器自动完成），我们就能识别到对应 session，从而找到用户的信息。这种方式目前来看最方便，但是也有一些缺陷，如下：

* 服务端保存大量数据,增加服务端压力
* 服务端保存用户状态,不支持集群部署

### 无状态

服务端集群中的每个服务,都外提供的都是用restful风格的接口,而restful风格的一个最重要的规范就是服务的无状态性.即

* 服务端不保存任何客户端请求者的信息
* 客户端每次请求都必须具备自描述信息,通过这些信息识别客户身份

无状态的好处

* 客户端请求不依赖服务端的信息,多次请求不需要必须访问到同一台服务器
* 服务端的集群 和状态对客户端透明
* 服务端可以任意的迁移和伸缩
* 减小服务端存储压力

## 如何实现无状态

无状态登陆流程

1. 首先客户端发送账号名/密码到服务端进行认证
2. 认证通过后,服务端将用户信息加密并编码成一个token,返回个客户端
3. 以后客户端每次请求,都需要携带token
4. 服务端对客户端发来的token进行解密,判断是否有效,并且获取用户登录信息

## 各自优缺点

使用session的最大优点在于方便;不用做过多的处理,默认即可.

但是使用session的另外一个致命问题就是如果你的前端有android,ios,小程序等,这些App天然就没有cookie,如果非要用session就需要这些工程师在各自的设备上做适配,一般是模拟cookie.

Jwt无状态登陆依赖的token可以通过普通参数传递也可以通过请求头传递,怎么样都行,具有很强的灵活性;



