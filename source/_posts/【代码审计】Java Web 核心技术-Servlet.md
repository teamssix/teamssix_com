---
title: 【代码审计】Java Web 核心技术-Servlet
date: 2021-11-15 16:57:45
id: 211115-165745
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151656032.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x00 前言

Servlet 是 Java Web 容器中运行的小程序，Servlet 原则上可以通过任何客户端-服务端协议进行通信，但它们常与 HTTP 一起使用，因此 Servlet 通常作为 “HTTP Servlet”的简写。

Servlet 是 Java EE 的核心，也是所有 MVC 框架实现的根本。

# 0x01 Servlet 的配置

版本不同，Servlet 的配置不同，Servlet 3.0 之前的版本都是在 web.xml 中配置的，在 3.0 之后的版本中则使用更为方便的注解方法来配置。

此外不同版本的 Servlet 所需要的 Java/JDK 版本也不同，具体如下图所示。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151655826.png)

## 1、基于 web.xml 的配置

以下是一个基于 web.xml 的 Servlet 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
         version="2.5">
  
  <display-name>manage</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
  
  <servlet>
    <description></description>
    <display-name>user</display-name>
    <servlet-name>user</servlet-name>
    <servlet-class>com.sec.servlet.UserServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>user</servlet-name>
    <url-pattern>/user</url-pattern>
  </servlet-mapping>
  </web-app>
```

在 web.xml 中，Servlet 的配置在 Servlet 标签中，Servlet 标签由 Servlet 和 Servlet-mapping 标签组成，两者通过标签中相同的 Servlet-name 实现关联，上述配置文件中的标签含义如下：

```
<servlet>				声明 Servlet 配置入口
<description>		声明 Servlet 描述信息
<display-name>	定义 Web 应用的名称
<servlet-name>	声明 Servlet 名称以便在后面的映射时使用
<servlet-class>	指定当前 Servlet 对应的类的路径
<servlet-mapping>	注册组件访问配置的路径入口
<servlet-name>	指定上文配置的 Servlet 名称
<url-pattern>		指定配置这个组件的访问路径
```

## 2、基于注解方式

Servlet 3.0 以上的版本中，开发者无须在 web.xml 里配置 Servlet，只需要添加 @WebServlet 注解即可修改 Servlet 的属性，如下代码所示。

```java
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * 基于注解开发Servlet
 */
@WebServlet(urlPatterns = "/ann.do")
public class AnnotationServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter out = resp.getWriter();
        out.println("<!DOCTYPE HTML PUBLIC '-//W3C//DTD HTML 4.0 Transitional//EN'>");
        out.println("<HTML>");
        out.println("<HEAD><TITLE> ITBZ </TITLE></HEAD>");
        out.println("<BODY>");
        out.println("Annotation Servlet!");
        out.println("</BODY>");
        out.println("</HTML>");
        out.flush();
        out.close();
    }
}
```

注解参数

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151656906.png)

通过图中的「等价于 web.xml 的标签」一栏可以看出，web.xml 可以配置的 Servlet 属性都可以通过 @WebServlet 的方式进行配置。

# 0x02 Servlet 的访问流程

以上文中的 web.xml 配置文件为例，其访问流程如下所示。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151656655.png)

首先用户在浏览器里输入URL，然后浏览器发起请求，服务器通过 servlet-mapping 标签找到文件名为 user 的 url-pattern，通过其对应的 servlet-name 寻找 servlet 标签中 servlet-name 相同的 servlet，再获取其 servlet 标签里的 servlet-class 参数，最后得到具体的 class 文件路径，从而执行相关文件。

# 0x03 Servlet 的接口方法

## 1、init() 接口

在 Servlet 实例化后，Servlet 容器会调用 init() 方法来初始化该对象，主要是为了让 Servlet 对象在处理客户请求前可以完成一些初始化的工作，例如建立数据库连接，获取配置信息等。

对于每一个 Servlet 实例，init() 方法只能被调用一次。

init() 方法的定义如下：

```java
public void init() throws ServletException{
    // 开发者自定义代码
}
```

## 2、service() 接口

service() 方法是执行实际任务的主要方法，Servlet 容器调用 service() 方法来处理来自客户端（浏览器）的请求，并将格式化的响应写回客户端，每次服务器接收到一个 Servlet 请求时，服务器都会产生一个新的线程并调用服务。

值得注意的是，在 service() 方法被容器调用之前，必须确保 init() 方法正确完成。

service() 方法的定义如下：

```java
public void service (ServletRequest request,ServletResponse response) throws ServletException,IOException{
     // 开发者自定义代码
}
```

## 3、doGet() / doPost() 等接口

doGet() 等方法需要根据 HTTP 的不同请求调用不同的方法，如果 HTTP 得到一个来自 URL 的 GET 请求，就会调用 doGet() 方法，同样的，如果得到 POST 请求，就会调用 doPost() 方法。

这类方法的定义如下：

```java
public void doGet(HttpServletRequest request,HttpServletResponse response) throws ServletException,IOException{
    // 开发者自定义代码
    // 如果是 POST 请求，则调用 public void doPost 方法
}
```

## 4、destroy() 接口



当容器检测到一个 Servlet 对象应该从服务中被移除的时候，容器会调用该对象的 destroy() 方法，以便让 Servlet 对象可以释放它所使用的资源，保存数据到持久存储设备中，例如将内存中的数据保存到数据库中，关闭数据库的连接等。

destroy() 方法与 init() 方法相同，都只会调用一次。

destroy() 方法定义如下：

```java
public void destroy(){
    // 开发者自定义代码
}
```

## 5、getServletConfig() 接口

该方法返回容器调用 init() 方法时传递给 Servlet 对象的 ServletConfig 对象，ServletConfig 对象包含了 Servlet 的初始化参数。

## 6、getServletInfo() 接口

该方法返回一个 String 类型的字符串，其中包括了关于 Servlet 的信息，例如，作者、版本和版权等。

# 0x04 Servlet 的生命周期

Servlet 生命周期指的是 Servlet 从创建直到销毁的整个过程。

在一个生命周期中，Servlet 经历了被加载、初始化、接收请求、响应请求以及提供服务的过程，具体如下图所示。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151656032.png)

> 参考链接：
>
> https://tomcat.apache.org/whichversion.html
>
> https://blog.csdn.net/bieleyang/article/details/76696131
>
> https://blog.csdn.net/qq_45017999/article/details/106696148
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
