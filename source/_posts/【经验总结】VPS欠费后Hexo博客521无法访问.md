---
title: 【经验总结】VPS欠费后Hexo博客521无法访问
date: 2019-10-09 16:46:24
id: 191009-164624
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/521_1.png
tags:
- VPS
- Hexo
- 经验总结
categories:
- 经验总结
---
# 0x00 前言
最近自己博客的VPS欠费了，但是充值之后，启动VPS发现博客依旧无法访问，经过多次排查后，最后的结果真的是哭笑不得，下面就记录一下我最后的解决办法。
<!--more-->
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/521_1.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/521_2.png)

# 0x01 排查过程
排查的过程中，碰到的第一个问题就是我发现SSH连接不上了，第一反应是博客被黑了？之后修改密码后才登上，估计只是我忘记密码了吧
之后又发现hexo同步本地数据同步不上去，怎么搞都不行，之后过了一天，发现又可以同步了，这……玄学问题？
直到博客无法访问第三天，我到网上四处找寻结果，还是没找到我碰到的这个问题，最后突然看到有人提到hexo使用的是nginx网页服务器，这才恍然大悟，我博客的nginx没有开！
# 0x02 解决步骤
直接进入自己VPS的命令行输入nginx开启nginx服务就行了，之后如果不放心可以输入netstat -ant看看自己的80端口有没有开。
```bash
[root@VPS_name ~]# nginx
[root@VPS_name ~]# netstat -ant
```
# 0x03 一点思考
讲道理，最后发现是这样的一个原因，还是挺尴尬的，博客自从搭建好后，几个月都没有碰过这些环境的问题，以前VPS重启nginx也是自动开启的，这次不知道为什么突然不行了，同时也感觉到有些东西长时间不碰，即使当初看着多么简单的东西也变难了起来。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)