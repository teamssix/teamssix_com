---
title: 【代码审计】跨站脚本 XSS
date: 2021-12-20 16:45:19
id: 211220-164519
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112201646981.png
tags:
- 学习笔记
- 代码审计
categories:
- 代码审计
---

# 0x01 反射型 XSS

以下代码展示了反射型 XSS 漏洞产生的大概形式

```java
<%
    String name = request.getParameter("name");
    String studentId = request.getParameter("sid");
    out.println("name = "+name);
    out.println("studentId = "+studentId);
%>
```

当访问 [http://localhost:8080/xss_demo/reflected_xss.jsp?name=1&sid=%3Cscript%3Ealert(1)%3C/script%3E](http://localhost:8080/xss_demo/reflected_xss.jsp?name=1&sid=alert(1)) 时，就会触发 XSS 代码

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112201646020.png)

# 0x02 存储型 XSS

这里以 zrlog v1.9.1.0227 靶场进行示例

## 部署靶场

靶场 war 包下载地址：http://dl.zrlog.com/release/zrlog-1.9.1-cd87f93-release.war

安装参考官方安装文档即可：https://blog.zrlog.com/post/how-to-install-zrlog

## 漏洞复现

来到后台，编辑标题处，在标题的位置插入 XSS 语句

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112201646981.png)

访问主页，可以看到 XSS 语句被执行

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202112201646003.png)

通过抓包可以看到提交 XSS 语句时的 URL 为 /api/admin/website/update

## 漏洞分析

查看该项目的 Web.xml 可以看到通过类 com.zrlog.web.config.ZrLogConfig 进行访问控制

```xml
<init-param>
	<param-name>configClass</param-name>
	<param-value>com.zrlog.web.config.ZrLogConfig</param-value>
</init-param>
```

 查看 WEB-INF/classes/com/zrlog/web/config/ZrLogConfig.class 文件

可以看到这份源码的路由配置信息

```java
public void configRoute(Routes routes) {
    routes.add("/post", PostController.class);
    routes.add("/api", APIController.class);
    routes.add("/", PostController.class);
    routes.add("/install", InstallController.class);
    routes.add(new AdminRoutes());
}
```

查看 AdminRoutes() 方法

```java
class AdminRoutes extends Routes {
    AdminRoutes() {
    }

    public void config() {
        this.add("/admin", AdminPageController.class);
        this.add("/admin/template", AdminTemplatePageController.class);
        this.add("/admin/article", AdminArticlePageController.class);
        this.add("/api/admin", AdminController.class);
        this.add("/api/admin/link", LinkController.class);
        this.add("/api/admin/comment", CommentController.class);
        this.add("/api/admin/tag", TagController.class);
        this.add("/api/admin/type", TypeController.class);
        this.add("/api/admin/nav", BlogNavController.class);
        this.add("/api/admin/article", ArticleController.class);
        this.add("/api/admin/website", WebSiteController.class);
        this.add("/api/admin/template", TemplateController.class);
        this.add("/api/admin/upload", UploadController.class);
        this.add("/api/admin/upgrade", UpgradeController.class);
    }
}
```

从上面代码的第 16 行可以看到 /api/admin/website 对应到 WebSiteController 类

查看 WebSiteController 类的代码

```java
 public WebSiteSettingUpdateResponse update() {
     Map<String, Object> requestMap = (Map)ZrLogUtil.convertRequestBody(this.getRequest(), Map.class);
     Iterator var2 = requestMap.entrySet().iterator();

     while(var2.hasNext()) {
         Entry<String, Object> param = (Entry)var2.next();
         (new WebSite()).updateByKV((String)param.getKey(), param.getValue());
     }

     WebSiteSettingUpdateResponse updateResponse = new WebSiteSettingUpdateResponse();
     updateResponse.setError(0);
     return updateResponse;
 }
```

从上面代码的第 2 行可以看到，update 方法会把传输过来的数据存储到 requestMap 对象里，并通过 updateByKV 方法进行数据更新。

因此这里需要对 updateByKV 进行分析，判断其有没有对输入内容进行过滤。

查看 updateByKV 方法

```java
public boolean updateByKV(String name, Object value) {
    if (Db.queryInt("select siteId from " + TABLE_NAME + " where name=?", name) != null) {
        Db.update("update " + TABLE_NAME + " set value=? where name=?", value, name);
    } else {
        Db.update("insert " + TABLE_NAME + "(`value`,`name`) value(?,?)", value, name);
    }
    return true;
}
```

可以看到这里并没有对数据进行过滤，直接将数据存储到了数据库里。

接下来分析一下输出的地方

```html
<h1 class="site-name">
  <i class="avatar"></i>
  <a title="${_res.title}" href="${rurl}">${_res.title}</a>
  <span class="slogan">${website.title}</span>
</h1>
```

${website.title} 这种写法也没有做转义，也成为了触发 XSS 的一环。

> 原文链接：
>
> [https://www.teamssix.com/211220-164519.html](https://www.teamssix.com/211220-164519.html)
>
> 更多信息欢迎关注我的个人微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
