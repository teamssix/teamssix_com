---
title: 【代码审计】Java EE 基础知识
date: 2021-11-15 12:34:51
id: 211115-123451
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111151127644.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

Java 平台分为三个主要版本：

1. Java SE（Java 平台标准版）
2. Java EE（Java 平台企业版）
3. Java ME（Java 平台微型版）

Java EE 是 Java 应用最广泛的版本。

# 0x01 Java EE 的核心技术

Java EE 有十三种核心技术，它们分别是：JDBC、JNDI、EJB、RMI、Servlet、JSP、XML、JMS、Java IDL、JTS、JTA、JavaMail 和 JAF，这里重点介绍以下几种：

1. Java 数据库连接（Java Database Connectivity，JDBC）是 Java 语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。
2. Java 命名和目录接口（Java Naming and Directory Interface，JNDI），是 Java 的一个目录服务应用程序界面（API），它提供一个目录系统，并将服务名称与对象关联起来，从而使得开发人员在开发过程中可以使用名称来访问对象。简单的说就是比如以前连接数据库需要把参数写在 Java 类里，但现在可以直接写在配置文件里了，这个配置文件可以是 XML，也可以是 properties，或者 yml 文件，只要能解析都行。
3. 企业级 JavaBean（Enterprise JavaBean, EJB）是一个用来构筑企业级应用的、在服务器端可被管理组件。不过这个东西在 Spring 问世后基本凉凉了，知道是什么就行。
4. 远程方法调用（Remote Method Invocation，RMI）是 Java 的一组拥有开发分布式应用程序的 API，它大大增强了 Java 开发分布式应用的能力。
5. Servlet（Server Applet），是用 Java 编写的服务端程序。其主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。狭义的 Servlet 是指 Java 语言实现的一个接口，广义的 Servlet 是指任何实现了这个 Servlet 接口的类，一般情况下，人们将 Servlet 理解为后者。
6. JSP（全称 JavaServer Pages）是由 Sun 公司主导创建的一种动态网页技术标准。JSP 部署于网络服务器上，可以响应客户端发送的请求，并根据请求内容动态地生成 HTML、XML 或其他格式文档的 Web 网页，然后返回给请求者。
7. 可扩展标记语言（eXtensible Markup Language，XML）是被设计用于传输和存储数据的语言。
8. Java 服务消息（Java Message Service，JMS）是一个 Java 平台中关于面向消息中间件（MOM）的 API，用于在两个应用程序之间或分布式系统中发送消息，进行异步通信。

# 0x02 Java EE 分层模型

Java EE 应用的分层模型主要分为以下 5 层。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/img/202111151127644.png)

1. Domain Object（领域对象）层，也叫模型（Modole）层，此层由一系列的 POJO（Plain Old Java Object，普通的、传统的 java 对象）组成，这些对象是该系统的 Domain Object，往往包含了各自所需实现的业务逻辑方法。
2. DAO（Data Access Object，数据访问对象）层，此层由一系列的 DAO 组件组成，这些 DAO 实现了对数据库的创建、查询、更新和删除（CRUD）等原子操作。
3. Service（业务逻辑层）层，此层由一系列的业务逻辑对象组成，这些业务逻辑对象实现了系统所需要的业务逻辑方法。这些业务逻辑方法可能仅仅用于暴露Domain Object 对象所实现的业务逻辑方法，也可能是依赖 DAO 组件实现的业务逻辑方法。
4. Controller（控制器）层，此层由一系列控制器组成，这些控制器用于拦截用户请求，并调用业务逻辑组件的业务逻辑方法，并根据处理结果转发到不同的 View 组件。
5. View（表现）层，此层由一系列的 JSP 页面、Velocity 页面、PDF 文档视图组件组成，负责收集用户请求，并显示处理后的结果。

> 参考链接：
>
> [https://zhuanlan.zhihu.com/p/43884237](https://zhuanlan.zhihu.com/p/43884237)
>
> [https://www.codenong.com/cs105259462/](https://www.codenong.com/cs105259462/)
>
> [https://blog.csdn.net/mzc_love/article/details/107244053](https://blog.csdn.net/mzc_love/article/details/107244053)
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)

