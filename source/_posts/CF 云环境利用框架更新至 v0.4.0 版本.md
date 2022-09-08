---
title: CF 云环境利用框架更新至 v0.4.0 版本
date: 2022-09-08 17:18:40
id: 220908-171840
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737405.png
tags:
- 云安全
- 工具分享
- 云服务
categories:
- 云安全
---

CF 是一个云环境利用框架，适用于在红队场景中对云上内网进行横向、SRC 场景中对 Access Key 即访问凭证的影响程度进行判定、企业场景中对自己的云上资产进行自检等等。

CF 项目地址：[github.com/teamssix/cf](https://github.com/teamssix/cf)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737418.png)

当前已支持的云：

- [x] 阿里云
- [x] 腾讯云

## 使用手册

使用手册请参见：[wiki.teamssix.com/cf](https://wiki.teamssix.com/cf)

[![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209081100625.png)](https://wiki.teamssix.com/cf)

## 简单上手

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737405.png)

> 这里以阿里云为例，其他更多操作可以查看上面的使用手册。

配置访问配置

```bash
cf config
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737407.png)

一键列出当前访问凭证的权限

```bash
cf alibaba perm
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737408.png)

一键接管控制台

```bash
cf alibaba console
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737409.png)

一键列出当前访问凭证的云服务资源

```bash
cf alibaba ls
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737410.png)

查看 CF 为实例执行命令的操作的帮助信息

```bash
cf alibaba ecs exec -h
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737411.png)

一键为所有实例执行三要素，方便 HVV

```bash
cf alibaba ecs exec -b
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737412.png)

一键获取实例中的临时访问凭证数据

```bash
cf alibaba ecs exec -m
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737413.png)

一键下载 OSS 对象存储数据

```bash
cf alibaba oss obj get
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737414.png)

一键升级 CF 版本

```bash
cf upgrade
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202209071737416.png)

## 版本更新

截止 2022 年 9 月 8 号，CF 已更新至 v0.4.0 版本，更新功能如下：

### 新增功能

- 增加对已有的访问凭证修改功能
- 增加控制台接管历史记录查看功能
- 增加接管控制台指定用户名功能

### 功能优化

- 优化阿里云 OSS 相关功能
- 全面优化配置访问凭证功能
- 全面优化程序缓存功能

值得注意的是，由于该版本对配置访问凭证的功能进行了优化，因此该版本需要重新配置访问凭证。

## 注意事项

* 本工具仅用于合法合规用途，严禁用于违法违规用途。
* 本工具中所涉及的风险点均属于租户责任，与云厂商无关。
