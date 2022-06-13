---
title: 对于连接ssh时连时断的解决方法
date: 2019-06-14 13:49:48
id: 190614-134948
tags:
- ssh
- 解决方案
categories:
- 经验总结
---
# 0x00 前言
最近用ssh连接VPS的时候，发生了很诡异的事件，有时能够连接上ssh，有时就死活连接不上，重新安装、公钥私钥都检查了，各种权限也都没有问题，端口监听地址什么的也都配置正常，总之能想到的办法都想过了，此时连不连的上仿佛就要看脸了，下面记录一下最终的解决办法。
<!--more-->
# 0x01 修改配置文件
```bash
vim /etc/ssh/sshd_config
```
在配置文件中加入以下内容：
```
UseDNS no
```
之后重启ssh即可
```
#Ubuntu:
service sshd restart 或者 /etc/init.d/sshd restart
#Centos7:
systemctl restart sshd.service
#Centos7以下版本：
service sshd restart
```
# 0x02 原因
以下解释来自本文方法参考文章：[文章地址](https://m.ancii.com/ayew4gv3/)

>ssh登录服务器时总是要停顿等待一下才能连接上,这是因为OpenSSH服务器有一个DNS查找选项UseDNS默认是打开的。
>UseDNS选项打开状态下,当客户端试图登录OpenSSH服务器时,服务器端先根据客户端的IP地址进行DNS PTR反向查询,查询出客户端的host name，然后根据查询出的客户端host name进行DNS 正向A记录查询，验证与其原始IP地址是否一致，这是防止客户端欺骗的一种手段,但一般我们的IP是动态的，不会有PTR记录的，打开这个选项不过是在白白浪费时间而已。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)