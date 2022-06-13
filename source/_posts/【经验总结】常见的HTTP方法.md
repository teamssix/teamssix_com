---
title: 【经验总结】常见的HTTP方法
date: 2019-11-27 20:14:38
id: 191127-201438
tags:
- 经验总结
- HTTP方法
categories:
- 经验总结
---
# 0x00 概述
根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP1.1新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT方法。
<!--more-->
# 0x01 GET
GET方法用于请求指定的页面信息，并返回实体主体。

# 0x02 HEAD
HEAD方法请求一个与GET请求的响应相同的响应，但没有响应体。

# 0x03 POST
向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。

数据被包含在请求体中，POST请求可能会导致新的资源建立或已有资源的修改。

# 0x04 PUT
PUT方法用请求有效载荷替换目标资源的所有当前表示。

# 0x05 DELETE
请求服务器删除指定的页面。

# 0x06 CONNECT
HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。

# 0x07 OPTIONS
允许客户端查看服务器的性能。

# 0x08 TRACE
回显服务器收到的请求，主要用于测试或诊断。

# 0x09 PATCH
是对PUT方法的补充，用来对已知资源进行局部更新。

>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考文章：
>[https://www.runoob.com/http/http-methods.html](https://www.runoob.com/http/http-methods.html)
>[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)