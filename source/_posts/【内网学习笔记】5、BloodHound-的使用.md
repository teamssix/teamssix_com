---
title: 【内网学习笔记】5、BloodHound 的使用
date: 2021-02-26 19:08:53
id: 210226-190853
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-02-25_14-00-42.png
tags:
- 红队
- 学习笔记
- 内网
categories:
- 内网学习笔记
---

## 1、介绍

BloodHound 使用可视化图形显示域环境中的关系，攻击者可以使用 BloodHound 识别高度复杂的攻击路径，防御者可以使用 BloodHound 来识别和防御那些相同的攻击路径。蓝队和红队都可以使用 BloodHound 轻松深入域环境中的权限关系。

BloodHound 通过在域内导出相关信息，在将数据收集后，将其导入Neo4j 数据库中，进行展示分析。因此在安装 BloodHound 时，需要安装 Neo4j 数据库。

## 2、安装

因为 Neo4j 数据库需要 Java 支持，因此安装 BloodHound 需要先安装 Java，这里以 Windows 系统下的安装为例。

### Java

JDK 需要下载最新版本，不然 Neo4j 运行可能会报错，JDK 下载地址：[https://www.oracle.com/java/technologies/javase-downloads.html](https://www.oracle.com/java/technologies/javase-downloads.html)，下载之后，直接安装即可。

### Neo4j

Neo4j 直接下载最新版本，下载地址：[https://neo4j.com/download-center/#community](https://neo4j.com/download-center/#community)

下载最新版本之后解压下载文件，打开 bin 目录，执行命令`neo4j.bat console`，之后打开浏览器访问 http://localhost:7474 登陆后台，输入以下信息连接到数据库说明安装就完成了。

```
URL：neo4j://localhost:7687
用户名(默认)：neo4j
密码(默认)：neo4j
```

### BloodHound

BloodHound 项目地址：[https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)，下载后解压打开 BloodHound.exe，输入 Neo4j 数据库的账号密码即可完成安装。

## 3、使用

安装完成 BloodHound 后，需要进行数据的采集与导入，数据的采集可以使用 ps1 脚本或者使用 exe 程序收集，工具下载地址：[https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)

这里使用 SharpHound.exe 进行数据的采集，将 SharpHound.exe 拷贝到目标上，执行 `SharpHound.exe -c all` 进行数据采集。

```
C:\Users\daniel10>SharpHound.exe -c all
---------------------------------------------
Initializing SharpHound at 22:36 on 2021/2/25
---------------------------------------------
Resolved Collection Methods: Group, Sessions, LoggedOn, Trusts, ACL, ObjectProps, LocalGroups, SPNTargets, Container
[+] Creating Schema map for domain TEAMSSIX.COM using path CN=Schema,CN=Configuration,DC=teamssix,DC=com
[+] Cache File Found! Loaded 1332 Objects in cache
[+] Pre-populating Domain Controller SIDS
Status: 0 objects finished (+0) -- Using 24 MB RAM
Status: 673 objects finished (+673 134.6)/s -- Using 43 MB RAM
Enumeration finished in 00:00:05.3136324
Compressing data to .\20210225223622_BloodHound.zip
You can upload this file directly to the UI
SharpHound Enumeration Completed at 22:36 on 2021/2/25! Happy Graphing!
```

如果使用 ps1 脚本收集，命令为：

```
powershell -exec bypass -command "Import-Module ./SharpHound.ps1; Invoke-BloodHound -c all"
```

采集到的数据会以 zip 压缩包的格式保存，将其拷贝到 BloodHound 所在主机上，在 BloodHound 右侧图标里点击 Upload Data，之后上传刚才生成的压缩包就可以导入数据了。

> 或者直接将 zip 压缩包拖拽到 BloodHound 里也可以导入数据。

在 BloodHound 右上角有三个板块：

1、Database Info（数据库信息），可以查看当前数据库中的域用户、域计算机等统计信息。

2、Node Indo（节点信息），单击某个节点时，在这里可以看到对应节点的相关信息。

3、Analysis（分析查询），在 BloodHound 中预设了一些查询条件，具体如下：

```
1、查询所有域管理员
2、寻找到域管理员的最短路径
3、查找具有DCSync权限的主体
4、具有外部域组成员资格的用户
5、具有外部域名组成员资格的组
6、映射域信任
7、到无约束委托系统的最短路径
8、到达Kerberoastable用户的最短路径
9、从Kerberoastable用户到域管理员的最短路径
10、拥有的主体的最短路径
11、从拥有的主体到域管理员的最短路径
12、到高价值目标的最短路径
13、查找域用户是本地管理员的计算机
14、查找域用户可以读取密码的计算机
15、从域用户到高价值目标的最短路径
16、找到从域用户到高价值目标的所有路径
17、找到域用户可以RDP的工作站
18、找到域用户可以RDP的服务器
19、查找域用户组的危险权限
20、找到高价值群体中能够支持kerberoable的成员
21、列出所有kerberoable用户
22、查找具有大多数特权的Kerberoastable用户
23、查找到非域控制器的域管理登录
24、查找不支持操作系统的计算机
25、查找AS-REP Roastable用户(DontReqPreAuth)
```

比如这里查询到域管理员的最短路径

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2021-02-25_14-00-42.png)

> 路径由粗到细表示xx对xx有权限或关系

总的来说感觉 BloodHound 还是挺有意思的，可以很直观的看到域内主机间的关系。不过毕竟是辅助工具，还是需要不断提升自己的实力、经验才能更好的去分析这样的一个结果才是。

> 参考链接：
>
> [https://xz.aliyun.com/t/7311](https://xz.aliyun.com/t/7311)
>
> [https://www.freebuf.com/sectool/179002.html](https://www.freebuf.com/sectool/179002.html)
>
> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)