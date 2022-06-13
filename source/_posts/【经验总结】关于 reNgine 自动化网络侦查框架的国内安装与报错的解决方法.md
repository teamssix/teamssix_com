---
title: 【经验总结】关于 reNgine 自动化网络侦查框架的国内安装与报错的解决方法
date: 2020-09-20 14:26:41
id: 200920-142641
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/86880620-92814300-c10a-11ea-9b27-627f43934221.png
summary: reNgine 是 Yogesh Ojha 写的一款自动化网络侦查框架，或者说是信息收集聚合工具，同时也是我个人比较满意的一款信息收集聚合工具。
tags:
- reNgine
- 工具分享
- 经验总结
categories:
- 经验总结
---

# 0x00 关于

reNgine 是 `Yogesh Ojha` 写的一款自动化网络侦查框架，或者说是信息收集聚合工具，他的推特：[@ojhayogesh11](https://twitter.com/ojhayogesh11)

该工具集合了子域名扫描、目录扫描、端口扫描、CMS扫描等等，扫描结束后，还能在手机上通知你。

在平时渗透测试的过程中使用这个工具可以节约很多信息收集的时间，项目地址为：[https://github.com/yogeshojha/reNgine](https://github.com/yogeshojha/reNgine)

在去年年底的时候我写了一款被动信息收集聚合工具，已经放在了我的 GitHub 上，当时还打算写个 Web 页面，为此还特地去学了一下 Django。

当时设计的 Web 页面大概长这个样子，但最后因为自己实在太菜，Web 页面没能写下去，只写了命令行版的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-09-11_10-09-30.png)

在今年年初又写了一款主动信息收集工具，但是效果不太理想，所以就没放在我的 GitHub 上。

直至今年7月份在逛推特的时候，偶然看到大佬分享的 reNgine 这款工具。打开这个项目一看，这不就是我理想中的信息收集聚合工具嘛，虽然目前使用起来还有不少的 bug ，但是整体上个人觉着已经很不错了，至少比自己写的不知道好了多少倍。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/86880620-92814300-c10a-11ea-9b27-627f43934221.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/86752434-f9482300-c05c-11ea-954b-b0f538c1ecef.png)

> 在 FreeBuf 上也有其对应的中文介绍：[https://www.freebuf.com/sectool/245292.html](https://www.freebuf.com/sectool/245292.html)

# 0x01 安装

## 1、环境准备

需要有 docker、docker-compose、make 环境

## 2、下载项目

```
git clone https://github.com/yogeshojha/reNgine.git
```

## 3、开始安装

```
cd reNgine
make certs		#使用https访问，个人觉着可省略
make build		#构建项目
make up			#启动项目
make username	#创建登陆用户
```

## 4、更新

```
git pull
make build
make up
```

看起来是非常的简单，但是对于在国内的我们来说，这其中包含了不少的坑。

# 0x02 过程

由于是老外写的东西，在这款工具安装过程中也引用下载了很多国外的文件，所以难免包含了一些被墙的东西。

最初在本地安装报错、安装报错这样过了几天之后就懒得整了，最后直接在国外的 vps 上去安装了，然后几分钟，真的就只要几分钟就安装好了。

但是一个月过去了、两个月过去了，随着国外 vps 的使用频率变低了，最后 vps 上最常使用的就是这个工具了，如果只是为了使用这个工具而去租个 vps ，实在觉着有些划不来。

于是又开始了在本地安装的折腾之旅，下面就来看看安装过程中的报错与解决方法。

# 0x03 问题

## 1、下载安装很慢

一开始是以为 docker 下载慢的原因，所以试着给 docker 加代理，又或者给 docker-compose 加代理等等方法都不行。最后试了亿下后，意识到应该是 Dockers 容器里下载文件比较慢，之后修改了 reNgine 项目目录下的 Dockerfile 文件才解决了这个问题。

通过观察发现，在 build 的过程中，会访问默认系统镜像源下载安装文件，同时也会访问 pip 默认镜像源下载安装文件，因此我们只需要把这两个默认的镜像源替换成国内的就可以了。

打开 Dockerfile 文件，在第一个 `RUN` 命令前，加上以下命令。修改后，下面的命令在我这里是 Dockerfile 文件的第 13 行左右的样子。

```
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && mkdir ~/.pip/ \
    && echo '[global]' > ~/.pip/pip.conf \
    && echo 'index-url = https://mirrors.aliyun.com/pypi/simple/' >> ~/.pip/pip.conf
```

上面的命令在我理解就是把 Docker 容器中的 `/etc/apk/repositories` 文件里的 `dl-cdn.alpinelinux.org` 字符串替换成了 `mirrors.aliyun.com` ，以达到将系统默认镜像源替换成阿里云镜像源的目的。

之后创建 `~/.pip/pip.conf` 文件，并添加阿里云 pip 源，以达到替换 pip 源的目的。

当然如果有不喜欢阿里云源的同学可以换成其他家的源。

## 2、go get 被墙

在默认配置下，直接使用 `make build` ，我在进行到第 12 步的时候报了下面这个错误。

```
Step 12/24 : RUN go get -u github.com/tomnomnom/assetfinder github.com/hakluke/hakrawler github.com/haccer/subjack
 ---> Running in e38c60f832f0
package golang.org/x/net/html: unrecognized import path "golang.org/x/net/html" (https fetch: Get https://golang.org/x/net/html?go-get=1: dial tcp 216.239.37.1:443: connect: connection refused)
package golang.org/x/net/html/charset: unrecognized import path "golang.org/x/net/html/charset" (https fetch: Get https://golang.org/x/net/html/charset?go-get=1: dial tcp 216.239.37.1:443: connect: connection refused)
package google.golang.org/appengine/urlfetch: unrecognized import path "google.golang.org/appengine/urlfetch" (https fetch: Get https://google.golang.org/appengine/urlfetch?go-get=1: dial tcp 216.239.37.1:443: connect: connection refused)
package golang.org/x/text/encoding: unrecognized import path "golang.org/x/text/encoding" (https fetch: Get https://golang.org/x/text/encoding?go-get=1: dial tcp 216.239.37.1:443: connect: connection refused)
```

后来去网站查了亿下，是因为在使用 `go get`命令获取资源会被墙的关系。知道了原因就好办了，我们只要加代理就行了。

在 reNgine 项目目录下的 Dockerfile 文件中找到 `# Download Go packages` 这一行，在这一行的下面添加以下两条命令，此时 go get 就会去走代理，访问 `goproxy.io` 这个镜像下载资源了。

```
ENV GO111MODULE=on
ENV GOPROXY=https://goproxy.io
```

> 如果发现 goproxy.io 无法访问，可以试试 gocenter.io 这个代理

再次执行 `make build` ，就会发现第 12 步成功运行，但这还没完，报错依旧继续。

## 3、安装 psycopg2 报错

在继续安装的过程中，又给我报了这些问题

```
Building wheel for psycopg2 (setup.py): started
  Building wheel for psycopg2 (setup.py): finished with status 'error'
  ERROR: Command errored out with exit status 1:
   command: /usr/local/bin/python -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-eps_y188/psycopg2/setup.py'"'"'; __file__='"'"'/tmp/pip-install-eps_y188/psycopg2/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d /tmp/pip-wheel-fisuy49f
       cwd: /tmp/pip-install-eps_y188/psycopg2/
……
Moving to /usr/local/lib/python3.8/site-packages/psycopg2/
   from /usr/local/lib/python3.8/site-packages/~sycopg2
ERROR: Command errored out with exit status 1: /usr/local/bin/python -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-kwlcc4ta/psycopg2/setup.py'"'"'; __file__='"'"'/tmp/pip-install-kwlcc4ta/psycopg2/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-r7w9n2l8/install-record.txt --single-version-externally-managed --compile --install-headers /usr/local/include/python3.8/psycopg2 Check the logs for full command output.
The command '/bin/sh -c pip3 install -r /tmp/requirements.txt' returned a non-zero code: 1
```

最初判断可能是因为源的问题，但是后来发现不管是默认的源还是国内源都会报错。

然后觉着可能还是和之前两个问题一样需要修改 Dockerfile 文件，但是在修改了无数次 Dockerfile 文件后发现都不行。

直到最后判断可能是版本问题，于是把 reNgine 目录下的 requirements.txt 文件里 psycopg2 后的版本号删除，再运行果然就可以了。

如果在 `pip install` 安装其他模块也报类似的错误时，也可以尝试删除 requirements.txt 文件的里版本号试试。不过这种操作可能会给后期带来一些不兼容的问题，但总强于安都安装不上的情况。

> 在 pip 安装的过程中如果报错，可以再尝试几遍，因为有时仅仅可能是因为本地网络的原因。

如果你碰到了除上面三个问题之外的其他问题，欢迎在下方留言。

# 0x04 最后

由于国内关于这款工具的安装说明少之又少，所以只能自己一步一步的去摸索，在经历了两天的报错—》排错—》报错……之后，终于安装好了这款工具。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/Snipaste_2020-09-11_12-21-28.png)

另外因为整个过程是我自己摸索出来的，因此难免中间存在有问题或者可以优化的地方，如有错误欢迎各位大佬指正。

> 参考链接：
>
> [https://www.liwenzhou.com/posts/Go/fix_go_get/](https://www.liwenzhou.com/posts/Go/fix_go_get/)
>
> [https://cloud.tencent.com/developer/article/1520882](https://cloud.tencent.com/developer/article/1520882)
>
>  [https://blog.csdn.net/u013360850/article/details/90602149](https://blog.csdn.net/u013360850/article/details/90602149)
> 
>  [https://blog.csdn.net/xuezhangjun0121/article/details/81664260](https://blog.csdn.net/xuezhangjun0121/article/details/81664260)
> 
>  更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)