---
title: 【经验总结】Python3 Requests 模块请求内容包含中文报错的解决办法
date: 2020-02-06 20:29:51
id: 200206-202951
tags:
- 经验总结
- Python3
- 解决办法
categories:
- 经验总结
---

# 0x00 前言

最近在写一个爬虫代码，里面需要使用 get 传参中文，但是如果直接使用中文而不对其编码的话，程序将会报错。

```
UnicodeEncodeError: 'latin-1' codec can't encode characters in position 38-39: ordinal not in range(256)
```

<!--more-->

# 0x01 网上的一些解决办法

参考网上的解决办法，比如下面的几种办法。

```
1、在中文后加上".encode('GBK')"
2、在文件头部加上"＃coding = utf-8"
3、在中文后加上".encode('utf-8')"
```

这几种方法在我这里都行不通，抓包也可以看到数据包里的中文并不是我们想象的经过 URL 编码的字符。

```
GET /test=b'%5Cxe6%5Cxb5%5Cx8b%5Cxe8%5Cxaf%5Cx95' HTTP/1.1
```

# 0x02 可行的办法

最后才意识到，其实并不需要对中文进行 GBK、UTF-8 转码，而应该对其进行 URL 编码。

```
from urllib.parse import quote
text = quote("测试", 'utf-8')
```

利用 quote 函数对 "测试" 进行 URL 编码后，再次抓包可以看到中文部分已经是 URL 格式了。

```
GET /test=%E6%B5%8B%E8%AF%95 HTTP/1.1
```

此时，程序也不再报错，可以顺利执行了。

>更多信息欢迎关注我的个人微信公众号：TeamsSix
>本文原文地址：[https://www.teamssix.com/year/200206-202951.html](https://www.teamssix.com/year/200206-202951.html)
>参考文章：[https://blog.csdn.net/qq_33876553/article/details/79730246](https://blog.csdn.net/qq_33876553/article/details/79730246)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)

