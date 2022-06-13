---
title: 【工具分享】分享一个jQuery多版本XSS漏洞检测工具
date: 2020-02-23 23:16:48
id: 200223-231648
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/1Snipaste_2020-02-23_23-05-36.png
tags:
- jQuery
- 工具分享
- XSS
categories:
- 工具分享
---

# 0x00 前言

最近在搞一个 jQuery v2.1.4 DOM-XSS 漏洞的复现，在网上找了很多Payload都不能用，大多数Payload都只适用于 jQuery v1.x 版本的。

后来看到有个文章说需要Safari浏览器，于是又废了半天劲装了个黑苹果（当时不知道原来Safari浏览器还有Windows版），用Safari浏览器一番折腾依旧没有复现，直到后来在GitHub上找到了这个检测工具，分享出来，避免踩坑。

<!--more-->

# 0x01 工具使用

> 下载地址：[https://github.com/mahp/jQuery-with-XSS](https://github.com/mahp/jQuery-with-XSS)


1、下载之后，解压，使用编辑器打开，修改第9行代码，将代码中 src 后的链接修改为自己要验证的js地址链接。

```
<script type="text/javascript" src="http://yourjquerylink/jquery.min.js"></script>
```

2、保存之后，使用浏览器打开，然后可以看到3个Demo.

```
bug-9521
bug-11290
bug-11974
```

3、可以依次点击这三个Demo，看看哪个会弹窗，我这里是 jQuery v2.1.4 的版本，在点击 bug-11974 发生了弹窗，说明此版本的漏洞被验证成功了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/1Snipaste_2020-02-23_23-05-36.png)

4、也可以点击页面中的 test version，来判断自己版本的  jQuery 版本存在的 bug 编号，例如我这里的  jQuery v2.1.4 版本就对应着 bug-2432和bug-11974。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2Snipaste_2020-02-23_23-09-03.png)

5、知道 bug 编号之后，再来到 test.html 页面点击对应的 bug编号即可。

# 0x02 小结

以上工具的使用方法是自己慢慢摸索出来的，如有不对的地方欢迎留言批评指正。

> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)