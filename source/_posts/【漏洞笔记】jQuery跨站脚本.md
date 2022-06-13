---
title: 【漏洞笔记】jQuery跨站脚本
date: 2019-11-20 21:41:29
id: 191120-214129
tags:
- 漏洞笔记
- jQuery
- XSS
categories:
- 漏洞笔记
---
# 0x00 概述
漏洞名称：jQuery跨站脚本

风险等级：低危

问题类型：使用已知漏洞的组件

# 0x01 漏洞描述
关于jQuery：jQuery是美国程序员John Resig所研发的一套开源、跨浏览器的JavaScript库。该库简化了HTML与JavaScript之间的操作，并具有模块化、插件扩展等特点。
<!--more-->
漏洞原理：jQuery中过滤用户输入数据所使用的正则表达式存在缺陷，可能导致 location.hash跨站漏洞

影响版本：

jquery-1.7.1~1.8.3

jquery-1.6.min.js，jquery-1.6.1.min.js，jquery-1.6.2.min.js

jquery-1.2~1.5

# 0x02 漏洞危害
jQuery 1.4.2版本中，远程攻击者可利用该漏洞向页面中注入任意的HTML。

jQuery 1.6.3之前版本中，当使用location.hash选择元素时，通过特制的标签，远程攻击者利用该漏洞注入任意web脚本或HTML。

jQuery 3.0.0之前版本中，攻击者可利用该漏洞执行客户端代码。

# 0x03 修复建议
目前厂商已发布升级补丁以修复漏洞，详情请关注厂商主页：[https://jquery.com/](https://jquery.com/)

>更多信息欢迎关注我的个人微信公众号：TeamsSix
>参考文章：
>[http://www.word666.com/wangluo/121052.html](http://www.word666.com/wangluo/121052.html)
>[https://blog.csdn.net/qq_36119192/article/details/89811603](https://blog.csdn.net/qq_36119192/article/details/89811603)
>[https://www.cnblogs.com/security4399/archive/2013/03/13/2958502.html](https://www.cnblogs.com/security4399/archive/2013/03/13/2958502.html)
>[http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201801-582](http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201801-582)
>[http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201801-798](http://www.cnnvd.org.cn/web/xxk/ldxqById.tag?CNNVD=CNNVD-201801-798)