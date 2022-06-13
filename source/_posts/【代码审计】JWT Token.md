---
title: 【代码审计】JWT Token
date: 2021-12-14 17:59:48
id: 211214-175948
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112141758858.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x00 介绍

JSON Web Token 缩写成 JWT，被用于和服务器的认证场景中，这一点有点类似于 Cookie 里的 Session id，关于这两者的区别可以看本文尾部的参考链接。

JWT 由三部分构成，分别为 Header（头部）、Payload（负载）、Signature（签名），三者以小数点分割，格式类似于这样：

```html
Header.Payload.Signature
```

实际遇到的 JWT 一般是这种样子

```html
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**JWT 的第一部分 Header 通常由两个部分组成：**

- alg 表示使用的签名算法，例如 RSA、HMAC SHA256（或简写为 HS256）
- typ 表示 Token 的类型 Type

通常写成以下 JSON 格式 的样子

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后使用 Base64URL 编码为 eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9，就形成了 JWT 的第一部分。

**JWT 的第二部分 Payload  也是 JSON 的格式，例如：**

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

然后将该 JSON 对象进行 Base64URL 编码为 eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ，就形成了 JWT 的第二部分。

对于 Payload 官方规定了 7 个字段：

```html
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

除此之外，也是可以定义私有字段的。

**JWT 的第三部分 Signature 是对 Header 和 Payload 部分的签名，起到防止数据篡改的作用。**

首先需要指定一个密钥，这个密钥只有服务器知道，然后利用 Header 里指定的加密算法（默认是 HMAC SHA256）按照下面的公式生成签名。

```html
HMACSHA256(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
```

# 0x01 靶场复现

## 方法一

这里使用 WebGoat 靶场进行 JWT Token 实验，直接 Docker 搭建即可。

```bash
sudo docker pull webgoat/goatandwolf
sudo docker run -p 8080:8080 -p 9090:9090 -e TZ=Europe/Amsterdam -d webgoat/goatandwolf
```

打开之后，来到 (A2) Broken Authentication 找到 JWT Token 的第 5 关，可以看到这一关是需要修改 JWT Token 的值以 admin 身份进行投票重置，那么需要先找到 JWT 的加密密钥。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112141758858.png)

在点击重置投票按钮时，请求的 URL 为 http://172.16.214.20:8080/WebGoat/JWT/votings

Clone 源码到本地，看看在源码里能否找到什么有价值的信息

```bash
git clone https://github.com/WebGoat/WebGoat.git
```

直接在代码里全局搜索 /JWT/votings

在 webgoat-lessons/jwt/src/main/java/org/owasp/webgoat/jwt/JWTVotesEndpoint.java 的第 163 行找到 @PostMapping("/JWT/votings")，通过函数名 resetVotes() 判断大概率是进行重置投票的函数。

```java
@PostMapping("/JWT/votings")
@ResponseBody
public AttackResult resetVotes(@CookieValue(value = "access_token", required = false) String accessToken) {
    if (StringUtils.isEmpty(accessToken)) {
        return failed(this).feedback("jwt-invalid-token").build();
    } else {
        try {
            Jwt jwt = Jwts.parser().setSigningKey(JWT_PASSWORD).parse(accessToken);
            Claims claims = (Claims) jwt.getBody();
            boolean isAdmin = Boolean.valueOf((String) claims.get("admin"));
            if (!isAdmin) {
                return failed(this).feedback("jwt-only-admin").build();
            } else {
                votes.values().forEach(vote -> vote.reset());
                return success(this).build();
            }
        } catch (JwtException e) {
            return failed(this).feedback("jwt-invalid-token").output(e.toString()).build();
        }
    }
}
```

通过分析代码，从上面代码的第 8 行中不难看出 JWT 的密码为 JWT_PASSWORD 变量，当 Cookie 中的 access_token 参数里的 Payload 部分（即上面代码里的第 10 行 claims.get）的 admin 参数值为 true 时则判断为 admin 用户，如果为 false 则返回 jwt-only-admin

通过在代码里查询 JWT_PASSWORD 变量，可以找到值为 victory

```java
public class JWTVotesEndpoint extends AssignmentEndpoint {
    public static final String JWT_PASSWORD = TextCodec.BASE64.encode("victory");
    private static String validUsers = "TomJerrySylvester";
```

知道了 jwt 密钥之后，在 [jwt.io](https://jwt.io/) 上将 Payload 部分的 admin 值修改为 true ，在 Signature 部分添加密钥重新生成 JWT，然后 Burp 替换就可以成功重置投票了。

## 方法二

除了以上通过源码找 JWT 密钥的方法还可以利用将加密算法修改为 none，即通过不加密的方式进行绕过。

{"alg":"none"} 编码后为 eyJhbGciOiJub25lIn0=，将 Payload 里的 admin 值修改为 true，最终构造 Token 如下：

```java
eyJhbGciOiJub25lIn0%3d.eyJpYXQiOjE2NDAzMTg4NTEsImFkbWluIjoidHJ1ZSIsInVzZXIiOiJUb20ifQ.
```

> 参考文章：
>
> [https://www.cnblogs.com/ittranslator/p/14595165.html](https://www.cnblogs.com/ittranslator/p/14595165.html)
>
> [https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
>
> 原文链接：
>
> [https://www.teamssix.com/211214-175948.html](https://www.teamssix.com/211214-175948.html)
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)

 
