---
title: 【代码审计】SQL 注入
date: 2021-11-17 09:16:53
id: 211117-091653
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111172027760.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x01 JDBC 拼接不当造成 SQL 注入

JDBC 有两种方法执行 SQL 语句，分别为 PrepareStatement 和 Statement，两个方法的区别在于 PrepareStatement 会对 SQL 语句进行预编译，而 Statement 在每次执行时都需要编译，会增大系统开销。

理论上 PrepareStatement 的效率和安全性会比 Statement 好，但不意味着就不会存在问题。

以下是一个使用 Statement 执行 SQL 语句的示例

```java
String sql = "select * from user where id ="+req.getParameter("id");
PrintWriter out = resp.getWriter();
out.println("Statement Demo");
out.println("SQL: "+sql);
try {
    Statement st = conn.createStatement();
    ResultSet rs = st.executeQuery(sql);
```

这里如果输入的 id 为 1 or 1 = 2，那么 SQL 语句就会被拼接为 select * from user where id = 1 or 1 = 2，改变了想要查询 id = 1 的语义。 

PreqareStatement 方法支持使用 ? 对变量位进行占位，在预编译阶段填入相应的值会构造出完整的 SQL 语句，从而避免 SQL 注入的产生。

但开发有时为了便利，会直接采取拼接的方式构造 SQL 语句，这样一来依然会存在 SQL 注入，如下代码所示。

```java
String sql = "select * from user where id ="+req.getParameter("id");

PrintWriter out = resp.getWriter();
out.println("prepareStatement Demo");
out.println("SQL: "+sql);
try {
    PreparedStatement pst = conn.prepareStatement(sql);
    ResultSet rs = pst.executeQuery();
```

此时如果使用 or 1 = 1 仍然可以判断存在 SQL 注入，但是如果使用 ? 作为占位符，填入的字段的值就会进行严格的类型检查，就可以有效的避免 SQL 注入的产生，如下代码所示。

```java
PrintWriter out = resp.getWriter();
out.println("prepareStatement Demo");
String sql = "select * from user where id = ?";
out.println(sql);
try {
    PreparedStatement pstt = conn.prepareStatement(sql);
    // 参数已经强制要求是整型
    pstt.setInt(1, Integer.parseInt(req.getParameter("id")));
    ResultSet rs = pstt.executeQuery();
    while (rs.next()){
```

# 0x02 框架使用不当造成 SQL 注入

通常框架底层已经实现了对 SQL 注入的防御，但是如果在开发未能恰当的使用框架的情况下，依然会存在 SQL 注入的风险。

## 1、MyBatis 框架

MyBatis 的思想是将 SQL 语句编入配置文件中，避免 SQL 语句在代码中大量出现，方便对 SQL 语句的修改和配置。

MyBatis 使用 parameterType 向 SQL 语句传参，在 SQL 引用传参的时候可以使用 #{} 和 ${} 两种方式，两种方式区别如下：

${}：SQL 拼接符号，直接将输入的语句拼接到 SQL 语句里，想避免 SQL 注入问题需要手动添加过滤

\#{}：占位符号，在对数据解析时会自动将输入的语句前后加上单引号从而避免 SQL 注入



也就是说在 MyBatis 框架中，如果使用了 ${} 方法，同时又没有进行过滤就会产生 SQL 注入，而使用 #{} 方法时可以避免 SQL 注入。

## 2、Hibernate 框架

Hibernate 是现今主流的 Java 数据库持久化框架，采用 Hibernate 查询语句（HQL）注入。

HQL 查询语句来自 Hibernate 引擎进行解析，因此产生的错误可能来自数据库，也有可能来自 Hibernate 引擎。 

HQL 和 SQL 的区别：

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111172027760.png)

HQL 注入和 SQL 注入的成因都一样，使用拼接 HQL 语句的写法可能会导致 SQL 注入

```java
Query query = session.createQuery("from User where name='"+queryString+"'");
```

但是受语法影响，HQL注入在漏洞利用上有一定的限制，比如不能利用联合查询、不能跨库查表、执行命令等。

对于 Hibernate 的注入，这里只作为简单了解一下，平时代审的时候注意一下即可。

> 参考文章：
>
> [https://www.redhatzone.com/ask/article/1448.html](https://www.redhatzone.com/ask/article/1448.html)
>
> [https://blog.csdn.net/qq_36594628/article/details/119461996](https://www.redhatzone.com/ask/article/1448.html)
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
