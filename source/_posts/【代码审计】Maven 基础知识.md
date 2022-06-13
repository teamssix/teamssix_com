---
title: 【代码审计】Maven 基础知识
date: 2021-11-03 16:18:07
id: 211103-161807
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111031614583.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x00 前言

Maven 是一个项目构建和管理工具，利用它可以对 JAVA 项目进行构建和管理。

Maven 采用项目对象模型 POM（Project Object Model）来管理项目。

Maven 的主要工作就是用来解析一些 XML 文档、管理生命周期与插件。

Maven 被设计成将主要的职责委派给一组 Maven 插件，这些插件可以影响 Maven 生命周期，提供对目标的访问。

## 0x01 pom.xml 文件介绍

pom.xml 文件被用于管理源代码、配置文件、开发者的信息和角色等，Maven 项目中可以没有其他文件但必须包含 pom.xml 文件，该文件也是 Maven 项目的核心配置文件。

一个引入 Fastjson 1.2.24 版本组件的配置信息如下：

```
<dependencies>
  <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.24</version>
  </dependency>
</dependencies>
```

> 该配置信息可直接从 [https://mvnrepository.com/artifact/com.alibaba/fastjson/1.2.24](https://mvnrepository.com/artifact/com.alibaba/fastjson/1.2.24) 获得

配置文件里的 dependencies 和 dependency 用于定义依赖关系，dependency 里的 groupId、artifactId、version 用来定义所依赖的项目。

# 0x02 Maven 的使用

这里以 IntelliJ IDEA 为例，在新建项目时选择创建 Maven 项目即可。

在创建完 Maven 项目后，就会看到项目里包含的 pox.xml 文件了，对于安全人员就可以通过该文件去判断当前项目里是否包含了存在隐患的组件。

这时如果想搭建一个 Fastjson <= 1.2.24 版本的漏洞环境，就需要将 Fastjson <= 1.2.24 版本的组件引入。

这里只需要把上面的配置信息复制到 pom.xml 文件中，然后右击 pom.xml 文件选择「Maven」--「重新加载项目」，就可以自动进行组件的获取了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202111031614583.png)

稍后等组件被自动下载到本地并自动加入到项目依赖中后，就可以在项目代码使用该组件了。

> 参考文章：
>
> [https://www.jianshu.com/p/43ed2e9e386b](https://www.jianshu.com/p/43ed2e9e386b)
>
> [https://www.cnblogs.com/bndong/p/9762067.html](https://www.cnblogs.com/bndong/p/9762067.html)
>
> [https://www.cnblogs.com/cainiaomahua/p/10651014.html](https://www.cnblogs.com/cainiaomahua/p/10651014.html)
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
