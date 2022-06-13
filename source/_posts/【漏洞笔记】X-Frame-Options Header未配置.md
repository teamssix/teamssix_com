---
title: 【漏洞笔记】X-Frame-Options Header未配置
date: 2019-11-19 14:46:43
id: 191119-144643
tags:
- 漏洞笔记
- X-Frame-Options
categories:
- 漏洞笔记
---
# 0x00 概述
漏洞名称：X-Frame-Options Header未配置

风险等级：低危

问题类型：管理员设置问题

# 0x01 漏洞描述
X-Frame-Options HTTP 响应头是用来给浏览器指示允许一个页面可否在<忽略frame>,<忽略iframe>,<忽略embed>或者<忽略object>中展现的标记。

网站可以使用此功能，来确保自己网站的内容没有被嵌到别人的网站中去，从而避免点击劫持（clickjacking）攻击。

X-Frame-Options有三个值：
<!--more-->

### deny
表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。

### sameorigin
表示该页面可以在相同域名页面的 frame 中展示。

### allow-from uri
表示该页面可以在指定来源的 frame 中展示。

换一句话说，如果设置为DENY，不光在别人的网站frame嵌入时会无法加载，在同域名页面中同样会无法加载。

另一方面，如果设置为SAMEORIGIN，那么页面就可以在同域名页面的frame中嵌套。正常情况下我们通常使用SAMEORIGIN参数。

# 0x02 漏洞危害
攻击者可以使用一个透明的、不可见的iframe，覆盖在目标网页上，然后诱使用户在该网页上进行操作，此时用户将在不知情的情况下点击透明的iframe页面。通过调整iframe页面的位置，可以诱使用户恰好点击iframe页面的一些功能性按钮上，导致被劫持。

也就是说网站内容可能被其他站点引用，可能遭受到点击劫持攻击。

# 0x03 修复建议
### 配置 Apache
配置 Apache 在所有页面上发送 X-Frame-Options 响应头，需要把下面这行添加到 'site' 的配置中:

```
Header always set X-Frame-Options "sameorigin"
```
要将 Apache 的配置 X-Frame-Options 设置成 deny , 按如下配置去设置你的站点：
```
Header set X-Frame-Options "deny"
```
要将 Apache 的配置 X-Frame-Options 设置成 allow-from，在配置里添加：
```
Header set X-Frame-Options "allow-from https://example.com/"
```
### 配置 nginx配置
nginx 发送 X-Frame-Options 响应头，把下面这行添加到 'http', 'server' 或者 'location' 的配置中:

```
add_header X-Frame-Options sameorigin always;
```
### 配置 IIS配置
IIS 发送 X-Frame-Options 响应头，添加下面的配置到 Web.config 文件中：

```
<system.webServer>
  ...
  <httpProtocol>
    <customHeaders>
      <add name="X-Frame-Options" value="sameorigin" />
    </customHeaders>
  </httpProtocol>
  ...
</system.webServer>
```
### 配置 HAProxy
配置 HAProxy 发送 X-Frame-Options 头，添加这些到你的前端、监听 listen，或者后端的配置里面：

```
rspadd X-Frame-Options:\ sameorigin
```
或者，在更加新的版本中：
```
http-response set-header X-Frame-Options sameorigin
```
### 配置 Express
要配置 Express 可以发送 X-Frame-Options header，你可以用借助了 frameguard 来设置头部的 helmet。在你的服务器配置里面添加：

```
const helmet = require('helmet');
const app = express();
app.use(helmet.frameguard({ action: "sameorigin" }));
```
或者，你也可以直接用 frameguard：
```
const frameguard = require('frameguard')
app.use(frameguard({ action: 'sameorigin' }))
```

>更多信息欢迎关注我的个人微信公众号：TeamsSix

>参考文章：
>https://blog.whsir.com/post-3919.html
>https://blog.csdn.net/qq_25934401/article/details/81384876
>https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options