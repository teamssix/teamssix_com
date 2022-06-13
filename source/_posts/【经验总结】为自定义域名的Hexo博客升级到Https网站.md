---
title: 为自定义域名的Hexo博客升级到Https网站
date: 2019-06-12 12:37:00
id: 191612-123700
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https21.png
tags:
- Hexo
- Https
- CloudFlare
- Namesilo
categories:
- 经验总结
---
# 0x00 前言
一把小绿锁，增加安全与安全感。cloudflare 是一家国外的CDN加速服务商，我们可以用它来把我们的网站升级到https，同时还能够提高网站的访问速度。
如果在设置的过程中，因为网站太多英文而困扰，可以利用浏览器的一些插件进行翻译，比如Chrome自带的翻译。
我使用的域名提供商是namesilo，博客工具是hexo，在网上找到很多教程都是用的Github官方提供的升级到https的教程，要不就是各种命令，然后按照教程去配置就各种依赖报错，十分心累。
本文配置过程中，没有涉及到命令的地方，利用CloudFlare配置https，整体体验还是很是很不错的，而且还能一定程度上的伪装自己的真实IP地址，废话不多说，下面就开整吧。
<!--more-->
# 0x01 注册**CloudFlare**
打开注册地址[https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)，输入邮箱和密码，对于下面是否接收广告的选项我是取消勾选，可以自行选择，勾选了平时邮箱会接收到来自CloudFlare的广告。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https1.png)
此时邮箱会收到验证邮件，点击Verify email按钮，跳转到登陆界面，输入正确的账号密码后，才算是验证成功。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https2.png)
# 0x02 添加配置网站
回到Cloudflare页面，输入自己的域名
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https3.png)
网页提示正在查询你的DNS记录，点击Next
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https4.png)
这里我选择0元/月，也就是免费的，具体每个Plan是什么意思，可以看下面翻译后的图片，选择后之后，点击Confirm plan
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https5.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https6.png)
弹出提示信息，问我们是否确认购买这个plan，我们直接确认，点击Confirm
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https7.png)
将所有DNS记录删除，添加类型为A，Name为www，Value为你的IP地址，TTL为自动的一条记录后，点击Continue
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https8.png)接下来需要修改你的域名服务器，这就需要到你购买域名的地方去修改了，我的域名是在namesilo购买的，因此这里以namesilo为例。
这里可以先把下图中红色框中的内容先复制下来。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https9.png)
# 0x03 配置域名服务器
打开自己购买域名的地方，这里我打开的是namesilo官网，对于其他域名网站都类似，具体配置域名服务器的教程可以谷歌之。
进入官网登陆后，点击Manage My Domains
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https10.png)
选择需要修改的域名后，点击Change Nameservers，namesilo默认有3个Nameservers，我这里之前已经修改过了，所以NameServers栏中会和默认的不一样。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https11.png)
先将原来信息删除，将上面复制的内容逐一复制进去即可，点击SUBMIT，这里因为我已经修改过了，因此界面会显示不大一样。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https12.png)
回到CloudFlasre页面，点击Continue，跳转到以下界面，稍等一段时间，来到namesilo查看配置进度
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https13.png)
下面Status显示Active说明域名服务器就已经配置好了。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https14.png)
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https15.png)
# 0x04 配置SSL
来到Crypto，点击using SSl
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https16.png)
点击Sign up
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https17.png)
跳转到如下界面
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https18.png)
等待一段时间，少则几十分钟一个小时，多则24小时，见到下面红色框内容出现，说明SSl配置就成功了
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https19.png)
此时登陆我们的域名就可以看到高贵的安全锁标志了。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https20.png)
# 0x05 设置Always Use HTTPS规则
虽然设置了http，但是发现输入域名还是会自动以http协议连接，因此我们来到Page Rules添加一下规则。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https21.png)
第一个框填写自己的域名，接着选择Always Use HTTPS，点击Save and Deploy。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/https22.png)
同样需要等待一段时间，输入自己域名后就可以发现可以直接跳转到https了。

# 0x06 总结
除此之外在宝塔上配置https也是很方便的，不过因为担心宝塔配置Web的过程中安装的LNMP什么的和本地一些已经安装好的环境发生冲突，最后还是没有继续选择使用宝塔，于是在网上查找了很多将http升级为https的方法，但基本都是用的国内的云，都有自带的证书服务，并不适用于我的情况，最后看到这篇文章：[文章链接](https://zhouhengheng.com/%E5%AE%9E%E7%8E%B0Hexo%E5%8D%9A%E5%AE%A2%E4%BB%8EHTTP%E5%88%B0HTTPS%E5%8A%A0%E5%AF%86.html)，不过写的比较简陋，于是结合这篇文章以及自己的摸索，记录下这篇文章。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)