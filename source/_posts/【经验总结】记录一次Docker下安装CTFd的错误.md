---
title: 【经验总结】记录一次Docker下安装CTFd的错误
date: 2019-07-20 12:11:44
id: 190720-121144
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/docker_CTFd1.png
tags:
- Docker
- CTFd
- 环境搭建
categories:
- 经验总结
---
# 0x01 提示错误
根据官方的步骤执行docker-compose up但是我得到了这样的一个错误
<!--more-->
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/docker_CTFd1.png)

```bash
~/CTFd# docker-compose up
ERROR: The Compose file './docker-compose.yml' is invalid because:
networks.internal value Additional properties are not allowed ('internal' was unexpected)
```
经过多次查询后，是因为版本问题导致，因此需要将原来的docker-compose版本卸载，安装新版本。
# 0x02 安装新版本
卸载docker-compose版本
```bash
pip uninstall docker-compose
```
先升级一下pip
```bash
pip install –upgrade pip
```
继续安装新版本
```bash
pip install -U docker-compose
```
也可以使用国内pip源进行加速，我使用的国内源进行的安装
```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -U docker-compose
```
之后再执行docker-compose up就没有问题了
![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/docker_CTFd2.png)

平时遇到问题还是需要先根据提示自己一步一步去找解决方法，之后再利用好Google。

>参考文章：[https://www.ilanni.com/?p=13371](https://www.ilanni.com/?p=13371)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)