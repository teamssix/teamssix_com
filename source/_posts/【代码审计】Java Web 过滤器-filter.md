---
title: 【代码审计】Java Web 过滤器-filter
date: 2021-11-16 11:25:10
id: 211116-112510
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111161127002.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x00 前言

filter 被称为过滤器，是 Servlet 2.3 新增的一个特性，同时也是 Serlvet 技术中最实用的技术。

过滤器实际上就是对 Web 资源进行拦截，做一些处理后再交给下一个过滤器或 Servlet 处理，通常都是用来拦截 request 进行处理的，也可以对返回的 response 进行拦截处理。

开发人员利用 filter 技术，可以实现对所有 Web 资源的管理，例如实现权限访问控制、过滤敏感词汇、压缩响应信息等一些高级功能。

# 0x01 filter 的配置

filter 的配置类似于 Servlet，由 filter 和 filter-mapping 两组标签组成，和 Servlet 一样，如果版本大于 3.0 也可以使用注解的方式来配置 filter

## 1、基于 web.xml 的配置

以下是一个基于 web.xml 的配置内容

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
  
  <filter>
    <display-name>test</display-name>
    <filter-name>test</filter-name>
    <filter-class>com.sec.test.test</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>test</filter-name>
    <url-pattern>/test</url-pattern>
  </filter-mapping>
</web-app>
```

标签含义如下：

```
<filter> 指定一个过滤器
<filter-name> 为过滤器指定一个名称，该项不能为空
<filter-class> 用于指定过滤器的完整的限定类名
<init-param> 用于为过滤器指定初始化参数
  <param-name> 为 <init-param> 的子参数，用于指定参数的名称
  <param-value> 为 <init-param> 的子参数，用于指定参数的值
<filter-mapping> 用于设置一个 filter 负责拦截的资源
  <filer-name> 为 <filer-mapping> 的子参数，用于设置 filter 的注册名称，该值必须是在 <filter> 元素中声明过的过滤器的名称
  <url-pattern> 用于设置 filter 所拦截的请求路径
<servlet-name> 用于指定过滤器所拦截的 Servlet 名称
```

## 2、基于注解的方法

示例代码如下

```java
import java.io.IOException;

@WebFilter(description="this is filter test",urlPatterns={"/filterTest"})
public class filterTest implements Filter {
    public void destroy() {
        /*销毁时调用*/
    }
    
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        /*过滤方法 主要是对request和response进行一些处理，然后交给下一个过滤器或Servlet处理*/  
        chain.doFilter(req, resp);//交给下一个过滤器或servlet处理
    }
    
    public void init(FilterConfig config) throws ServletException {
        /*初始化方法  接收一个FilterConfig类型的参数 该参数是对Filter的一些配置*/
    }
    
}
```

与 Servlet 一样，使用 web.xml 可以配置的 filter 属性也都可以使用注解进行配置，但一般不推荐使用注解配置，因为使用 web.xml 可以控制过滤器的执行顺序，而使用注解的方式则不行。

# 0x02 filter 的流程

filter 的流程很简单，具体如下：

```
用户向服务器发送请求
|
服务器
|
filter 1
|
......
|
filter n
|
service()方法
|
filter n
|
......
|
filter 1
|
服务器
|
服务器返回结果给用户
```

首先用户向服务器发起请求，之后请求经过过滤器到达 Servlet 中的 service() 方法，最后再经过过滤器返回给用户。

这里值得注意的是过滤器在接收请求和返回请求的时候顺序是相反的。

# 0x03 filter 的接口方法

filter 接口方法如下：

## 1、init() 接口

该接口与 Servlet 里的 init() 方法类似，主要用来初始化过滤器。如果初始化代码中要用到 FilterConfig 对象，那这些初始化代码只能在 filter 的 init() 方法中编写。

init() 方法的定义如下：

```java
public void init(FilterConfig config) throws ServletException {
        /*初始化方法  接收一个FilterConfig类型的参数 该参数是对Filter的一些配置*/
}
```

## 2、doFilter() 接口

doFilter 方法类似于 Servlet 接口的 service() 方法，filter 通过该接口实现具体的过滤操作，当客户请求访问与过滤器关联的 URL 的时候，Servlet 过滤器将先执行doFilter 方法，FilterChain 参数用于访问后续过滤器。

doFilter() 方法定义如下：

```java
public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        /*过滤方法 主要是对request和response进行一些处理，然后交给下一个过滤器或Servlet处理*/  
        chain.doFilter(req, resp);//交给下一个过滤器或servlet处理
}
```

## 3、destroy() 接口

destroy() 接口和 servlet 里的 destroy() 作用类似，用于释放被 filter 打开的资源，如关闭数据库等。

destroy() 方法的定义如下：

```java
public void destroy() {
        /*销毁时调用*/
}
```

# 0x04 filter 的生命周期

filter 的生命周期与 servlet 的生命周期比较类似，如下图所示。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111161127002.png)

当 Web 容器启动时，会根据 web.xml 中声明的 filter 顺序依次实例化这些 filter。

这里会根据实际情况的不同，doFilter() 方法可能会被调用多次，最后当程序关闭或者卸载时调用 destroy() 方法。

> 参考文章：
>
> https://blog.csdn.net/Soinice/article/details/82787964
>
> https://www.cnblogs.com/tanghaorong/p/12811457.html
>
> https://blog.csdn.net/yuzhiqiang_1993/article/details/81288912
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
