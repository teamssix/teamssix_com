---
title: 【经验总结】解决 BurpSuite Pro v2020.1 版本中文乱码问题
date: 2020-02-17 13:44:45
id: 200217-134445
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/3Snipaste_2020-02-17_13-38-35.png
tags:
- 经验总结
- BurpSuite
- 软件
categories:
- 经验总结
---

# 0x00 前言

之前在我的公众号分享了 BurpSuite Pro v2020.1 版本，但是在使用过程中发现总是会有中文乱码的情况出现，后来使用 Lucida 字体，乱码的情况得到了缓解，但是有些网页依旧会出现乱码的情况，直到后来才意识到问题其实不在于字体的选择。

<!--more-->

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/1Snipaste_2020-02-17_13-34-32.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/2Snipaste_2020-02-17_13-37-02.png)

# 0x01 解决办法

来到 User Options --> Display --> Character Sets，在第四个选项中选择 UTF-8，中文乱码的问题就可以得到解决，这个时候不管我字体选择的是哪一个，中文都是显示正常的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/3Snipaste_2020-02-17_13-38-35.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/4Snipaste_2020-02-17_13-39-09.png)

# 0x02 总结

在网上找了很多解决办法，但是大部分文章都是说更改中文字体就能解决，但是我这个版本没有中文字体，所以那些文章里的办法都不能用，最后才意识到通过修改编码就能解决。

> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)