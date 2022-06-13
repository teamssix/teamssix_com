---
title: 【内网学习笔记】25、Exchange 邮件服务器
date: 2021-09-08 10:52:49
id: 210908-105249
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908104558.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、Exchange 的基本操作

> 在 Exchange 服务器上的 PowerShell 里进行以下操作

将 Exchange 管理单元添加到当前会话中

```
add-pssnapin microsoft.exchange*
```

查看邮件数据库

```
Get-MailboxDatabase -server "dc"
```

查询数据库的物理路径

```
Get-MailboxDatabase -Identity 'Mailbox Database 0761701514' | Format-List Name,EdbFilePath,LogFolderPath
```

获取现有用户的邮件地址

```
Get-Mailbox | Format-table Name,WindowsEmailAddress
```

查看指定用户的邮箱使用信息

```
Get-Mailboxstatistics -Identity Administrator | Select Dispayname,ItemCount,TotalItemSize,TotalTimeSize,LastLogonTime
```

获取用户邮箱中的邮件数量，通过该命令还可以列出那些用户未登录过邮箱

```
Get-Mailbox -ResultSize Unlimited | Get-Mailboxstatistics | Sort-Object TotalItemSize -Descend
```

## 2、导出指定的电子邮箱

Exchange Server 2007 中需要使用 ExportMailBox 命令，在 Exchange Server 2010 SP1 及以后的版本中可以使用图形化界面导出，也可以使用 PowerShell

如果想要导出 PTS 格式的邮件文件，则需要为用户配置导出/导出权限。

### 配置用户的导入导出权限

查看用户权限

```
Get-ManagementRoleAssignment -role "Mailbox Import Export"
```

将 Administrator 用户添加到 Mailbox Import Export  角色组里，将用户添加到角色组后，需要重启 Exchange 服务才能执行导出操作

```
New-ManagementRoleAssignment -Name "Import Export_Domain Admins" -User "Administrator" -Role "Mailbox Import Export"
```

删除刚刚添加的 Mailbox Import Export 角色组中的用户

```
Remove-ManagementRoleAssignment "Import Export_Domain Admins" -Confirm:$false
```

### 设置网络共享文件夹

不论使用哪种方式导出邮件，都需要将文件放置在 UNC（Universal Naming Convention，通用命名规则，也称通用命名规范）路径下

类似于 “\\hostname\sharename”、“\\ipaddress\sharename” 的网络路径下，sharename 为网络共享名称。

首先开启共享，将 C 盘 inetpub 文件夹设置为 everyone 可读写，执行如下命令：

```
net share inetpub=c:\inetpub /grant:everyone,full
```

### 导出用户的电子邮件

使用 PowerShell 导出电子邮件，用户的电子邮箱目录一般为Inbox（收件箱）、SentItems（已发送邮件）、DeleteItems（已删除邮件）、Drafts（草稿）等

```
New-MailboxExportRequest -Mailbox administrator -FilePath \\192.168.7.77\inetpub\administrator.pst
```

使用图形化界面导出电子邮件，访问 https://127.0.0.1/ecp，打开 Exchange 管理中心的登录界面。

输入账号密码进入 Exchange 管理中心，点击「...」更多按钮，选择「导出到 PST 文件」即可进行导出操作。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908104558.png)

### 管理导出请求

不论是通过 Powershell 导出还是通过图形化的方式导出，都会在 Exchange 中生成一条告警信息，这些信息有助于 BT 发现服务器里的异常行为，通过以下命令，可以查看之前的导出请求记录信息。

```
Get-MailboxExportRequest
```

将指定用户已经完成的导出请求删除

```
Remove-MailboxExportRequest -Identity Administrator\MailboxExport
```

删除所有已完成的导出请求

```
Get-MailboxExportRequest -Status Completed | Remove-MailboxExportRequest
```

删除所有导出请求，包括完成和失败的请求

```
Get-MailboxExportRequest | Remove-MailboxExportRequest
```

> 参考文章：
>
> [https://www.cnblogs.com/micr067/p/12307519.html](https://www.cnblogs.com/micr067/p/12307519.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
