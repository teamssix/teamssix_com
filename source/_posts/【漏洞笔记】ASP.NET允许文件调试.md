---
title: 【漏洞笔记】ASP.NET允许文件调试
date: 2019-11-26 21:58:09
id: 191126-215809
tags:
- 漏洞笔记
- ASP.NET
- 管理员设置问题
categories:
- 漏洞笔记
---
# 0x00 概述
漏洞名称：ASP.NET允许文件调试

风险等级：低

问题类型：管理员设置问题

# 0x01 漏洞描述
发送DEBUG动作的请求，如果服务器返回内容为OK，那么服务器就开启了调试功能，可能会导致有关Web应用程序的敏感信息泄露，例如密码、路径等。
<!--more-->
# 0x02 漏洞危害
可能会泄露密码、路径等敏感信息。

# 0x03 修复建议
编辑Web.config文件，设置```&lt;compilation debug="false"/&gt;```

>更多信息欢迎关注我的个人微信公众号：TeamsSix