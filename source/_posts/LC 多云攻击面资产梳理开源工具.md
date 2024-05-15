---
title: LC 多云攻击面资产梳理开源工具
date: 2024-05-15 21:16:40
id: 240515-211640
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202405152124839.png
tags:
- 云安全
- 工具分享
- 云服务
- LC
categories:
- 云安全
---

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202405152119889.png)

LC（List Cloud）是一个多云攻击面资产梳理的工具，使用 LC 可以让甲方蓝队在管理多云时快速梳理出可能暴露在公网上的资产。

## 功能

- 列出多个配置的云资产
- 支持多个云服务商
- 支持多个云服务
- 支持过滤内网 IP
- 高度可扩展性，可方便添加更多云服务商和云服务
- 可以使用管道符和其他工具结合使用

运行截图：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202405152124839.png)


### 支持列出的云服务

| 序号 | 云服务商 |    服务名称    |
|:--:|:----:|:----------:|
| 1  | 阿里云  |  ECS 云服务器  |
| 2  | 阿里云  |  OSS 对象存储  |
| 3  | 阿里云  |  RDS 数据库   |
| 4  | 腾讯云  |  CVM 云服务器  |
| 5  | 腾讯云  | LH 轻量应用服务器 |
| 6  | 腾讯云  |  COS 对象存储  |
| 7  | 华为云  |  OBS 对象存储  |
| 8  | 天翼云  |  OOS 对象存储  |
| 9  | 百度云  |  BOS 对象存储  |
| 10 | 百度云  |  BCC 云服务器  |
| 11 | 联通云  |  OSS 对象存储  |
| 12 | 七牛云  | Kodo 对象存储  |
| 13 | 移动云  |  EOS 对象存储  |

## 使用手册

详细使用手册请参见：[LC 使用手册](https://wiki.teamssix.com/lc)

## 安装

### 使用 brew 安装

安装

```bash
brew tap wgpsec/tap
brew install wgpsec/tap/lc
```

更新

```bash
brew update
brew upgrade lc
```

### 下载二进制文件

直接在 LC 下载地址：[github.com/wgpsec/lc/releases](https://github.com/wgpsec/lc/releases) 中下载系统对应的压缩文件，解压后在命令行中运行即可。

## 用法

```sh
lc -h
```

使用 `-h` 参数查看 lc 的帮助信息，这是目前 lc 所支持的用法。

```yaml
lc (list cloud) 是一个多云攻击面资产梳理工具

Usage:
  lc [flags]

Flags:
配置:
  -c, -config string  指定配置文件路径 (default "$HOME/.config/lc/config.yaml")
  -t, -threads int    指定扫描的线程数量 (default 3)

过滤:
  -i, -id string[]        指定要使用的配置（以逗号分隔）
  -p, -provider string[]  指定要使用的云服务商（以逗号分隔）
  -ep, -exclude-private   从输出的结果中排除私有 IP

输出:
  -o, -output string  将结果输出到指定的文件中
  -s, -silent         只输出结果
  -v, -version        输出工具的版本
  -debug              输出调试日志信息
```

## 简单上手

在第一次使用时，LC 会在 `$HOME/.config/lc` 目录下创建一个 `config.yaml`，因此在第一次执行 `lc` 命令后，将您的云访问凭证填写到 `$HOME/.config/lc/config.yaml` 文件中后，就可以开始正式使用 LC 了。

直接运行 `lc` 命令来列举您的云上资产。

```sh
lc
```

如果没有列举出结果，那么可能是因为本身云上没有资产，或者访问凭证的权限不足，这里我们建议为访问凭证赋予全局可读权限即可。

如果要排除结果中的内网 IP，只需要加上 `-ep` 参数。

```sh
lc -ep
```

如果想把 LC 和其他工具结合使用，例如使用 httpx 检测资产是否能从公网访问，那么可以使用下面的命令。

```sh
lc -ep -s | httpx -sc -title -silent
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202405152124453.png)

更多用法可以查看 [LC 使用手册](https://wiki.teamssix.com/lc)

## 后记

这个工具使用 MIT 协议开源，非常欢迎师傅们能一起为这个项目贡献代码，有想贡献代码的师傅可以直接向这个项目的 dev 分支提交 PR 就行啦。

另外如果这个工具能帮助到你，欢迎点个 Star，感谢你使用我的工具 ～

>  更多信息欢迎关注我的个人微信公众号：TeamsSix
>

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204152148071.png)