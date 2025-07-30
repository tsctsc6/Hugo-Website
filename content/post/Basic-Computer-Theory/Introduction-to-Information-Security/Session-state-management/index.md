+++
date = '2025-07-30T19:47:38+08:00'
draft = false
title = '会话状态管理'
categories = ['Sub Sections']
+++

## 为什么需要会话状态管理
网络的设计是无状态的，也就是说，服务器会把每一次网络请求视为独立的。但是有时我们往往需要服务器记住我们的会话状态，比如在某网站登录一次后，短时间内不需要登录第二次，进入网站，网站会自动加载账号资料。

接下来将介绍两种会话状态管理方式。

## Cookie
Cookie 是网站存储在用户浏览器中的小型文本，用于记录用户身份或行为信息。

### 工作原理
用户登录后，服务器在 HTTP 响应头中添加 Set-Cookie 字段，例如：

```txt
Set-Cookie: session_id=abc123; Expires=Wed, 30-Jul-2025 12:00:00 GMT; Secure; HttpOnly
```

浏览器下次访问时，自动在 HTTP 请求头加入 Cookie 字段，内容就是服务器响应头的 Set-Cookie 字段的内容。

服务端会记录 session_id 以及其他会话状态的信息，据此验证客户端的 Cookie 的合法性。

可以看出， Cookie 的主要结构是一些键值对，以及一些属性；它们使用分号间隔。

但是上面的 Cookie 有一个弊端，就是它没有防篡改机制。当然，可以主动加入[数字签名](../Message-Digest/index.md#数字签名)。

## JWT
JWT 是 JSON Web Token 的缩写。

### 工作原理
#### 服务端生成令牌
用户登录成功后，服务器生成令牌。分为三个部分：

##### Header（头部）
包含Token类型（固定为JWT）和所使用的签名算法（如HS256、RS256）。

例如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

##### Payload（载荷）
包含需要传递的​​声明（Claims）​​。常见的有：

* iss (issuer)：签发者
* sub (subject)：主题（用户ID等）
* aud (audience)：受众（目标接收方）
* exp (expiration time)：过期时间
* nbf (not before)：生效时间
* iat (issued at)：签发时间

例如：

```json
{
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true,
    "iat": 1516235422,
    "exp": 1516239022
}
```

##### Signing（签名）
使用Header中指定的算法进行[数字签名]。具体步骤如下：

1. 将 ​​Base64Url 编码的 Header​​ + '.' + ​​Base64Url 编码的 Payload​​ 拼接起来。
1. 使用一个​私钥（Private Key）​​对这个拼接后的字符串进行[数字签名](../Message-Digest/index.md#数字签名)。

最终的 JWT 内容为： ​​Base64Url 编码的 Header​​ + '.' + ​​Base64Url 编码的 Payload​​​ + '.' + ​​Base64Url 编码的数字签名。

#### 传递令牌
服务器将生成的 JWT 返回给客户端（通常通过 HTTP 响应体或响应头的 Authorization 字段）。

客户端在后续需要认证的请求中，将这个 JWT 发送回服务器。通常放在 HTTP 请求头的 Authorization 字段中，值为 Bearer <token> :

```txt
Authorization: Bearer eyJxxx.eyJxxx.ZZjiNma0o8gDSiVvVkkLcyA9VWF4FmxGrj5F8JxBoC0
```

#### 服务端验证令牌
验证数字签名即可。

有人说 JWT 不能从服务器主动吊销，其实办法还是有的，可以添加 `ConcurrencyStamp` 字段，格式为 UUID ，或者别的什么。用户表里也设有 `ConcurrencyStamp` 字段。每次验证时，也对比以下 `ConcurrencyStamp` 。服务端想主动吊销 JWT 时，更改 `ConcurrencyStamp` 即可。

