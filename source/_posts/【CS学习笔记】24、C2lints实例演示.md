---
title: 【CS学习笔记】24、C2lints实例演示
date: 2020-04-19 15:07:12
id: 200419-150712
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs24-2.png
summary: 续上一节，在GitHub 上有一些配置文件的示例，这一节将使用该项目中的 `Malleable-C2-Profiles/APT/havex.profile` 配置文件作为示例。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 前言

续上一节，在GitHub 上有一些配置文件的示例，项目地址：[https://github.com/rsmudge/Malleable-C2-Profiles](https://github.com/rsmudge/Malleable-C2-Profiles)

这一节将使用该项目中的 `Malleable-C2-Profiles/APT/havex.profile` 配置文件作为示例。

# 0x01 测试配置文件是否有效

可以使用 c2lint 工具对配置文件进行测试，以判断配置文件编写的是否有效。

来到 cobalt strike 目录下，可以看到有一个 c2lint 文件，该文件需要在 Linux 下运行。

```
./c2lint [profile]
```

在运行的结果中，绿色正常（这里更像青色），黄色告警，红色错误，比如运行 `Malleable-C2-Profiles` 项目里的 `havex.profile` 文件。

```
./c2lint ./Malleable-C2-Profiles/APT/havex.profile
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs24-1.png)

当配置文件存在错误的时候，就会以红色显示出来

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs24-2.png)

# 0x02 运行 teamserver

```
./teamserver [teamserver_ip] [teamserver_password] [profile]
```

```
> ./teamserver 192.168.12.2 password ./Malleable-C2-Profiles/APT/havex.profile
[*] Will use existing X509 certificate and keystore (for SSL)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
[+] I see you're into threat replication. ./Malleable-C2-Profiles/APT/havex.profile loaded.
[+] Team server is up on 50050
```

这里调用的 havex.profile 配置文件，该配置文件里对 cookie 进行了 base64 编码。

开启 cobalt strike 后，使主机上线，通过 wireshark 抓包可以发现数据包确实符合这些特征。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs24-3.png)

关于 Malleable C2 文件的使用，这里只是大概记录了一些，想了解更多关于 Malleable C2 文件的内容或者注意事项等，可以参考 A-TEAM 团队的 CS 4.0 用户手册。

> 参考链接：[https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)