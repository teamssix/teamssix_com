---
title: 【随笔】Windows 11 的安装尝鲜
date: 2021-06-16 13:36:45
id: 210616-133645
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616125657.png
tags:
- Windows 11
- 尝鲜
categories:
- 随笔
---

# 0x00 前言

看到网上有人发 Windows 11 的安装包，心想，咦，Windows 11 都出来了？那不赶紧下载一个来尝尝鲜。

网上流传的下载地址：[http://wall4.kfire.net/win11.iso](http://wall4.kfire.net/win11.iso)

# 0x01 安装

这里以 Mac 下的 VMware Fusion 安装为例，Windows 下的操作也都类似。

首先打开 `VMware Fusion`，选择`新建`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616110553.png)

把下载好的 `win11.iso` 文件拖拽过来

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616110611.png)

拖拽过来后，会来到 `创建新的虚拟机` 界面，点击继续

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616110707.png)

接下来，选择操作系统，我这里选择的 `Windows 10 x64`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616123525.png)

选择固件类型，我这里选择的 `传统 BIOS`，亲自尝试发现选择 `UEFI` 会无法启动

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616111009.png)

接下来，虚拟机就创建完成了，但是默认分配的内存有点低，因此可以点击 `自定设置`，自己调一下配置

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616132656.png)

点击 `自定设置`后，可以修改个名字，我这里修改成了 `windows11`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616111717.png)

点击 `存储` 后，会自动打开设置界面，这里我修改了两个地方，分别是 `处理器和内存` 和 `硬盘（IDE）`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616111813.png)

打开 `处理器和内存`，我这里分配了 4 个处理器内核和 4096 MB 的内存

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616111940.png)

点击 `显示全部`，点击 `硬盘（IDE）`，我这里分配了 50 G

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616112125.png)

修改磁盘大小后，点击 `应用` ,然后关闭设置这个窗口，点击这个大大的启动按钮

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616112229.png)

启动后，可以看到 Windows 11 新的 logo

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616112434.png)

这个镜像里没有中文，所以就直接用默认的英文了，点击 `Next`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616112615.png)

点击 `Install now`

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616112619.png)

接下来需要输入激活码，这里有个 Windows 11 Pro 的激活码：

````
FKNPR-6C4GH-R3292-P4RTJ-GVJWB
````

输入激活码，`Next` 下一步

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616123715.png)

勾选同意，`Next` 即可

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616113117.png)

选择 `Custom`，自定义安装

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616113230.png)

`New` 一个分区，点击 `New` 后，再点击 `Apply` 应用一下，不过貌似直接 `Next` 也可以

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616113639.png)

点击 `Apply`后会有个通知，点击确认就行， 然后点击 `Next` 就开始安装了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616113826.png)

接下来，等它安装完成就行了。

# 0x02 使用

安装完成后，选择国家和地区，这里选择了中国

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616123935.png)

选择键盘布局，这里就直接默认了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616124044.png)

询问是否要再添加一个键盘，这里就直接跳过了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616124155.png)

接下来，系统会检查更新，检查完成后，输入自己的名字

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616124458.png)

然后输入密码，之后再设置三个安全问题，接受隐私协议，最后等待几分钟

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616124739.png)

最后，就进入桌面了

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616124854.png)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616125657.png)

# 0x03 最后

单纯就外观来看，变化不算小，而且窗口的打开关闭也有了动画过渡，UI 变得更好看了，同时操作逻辑还是原来的逻辑，上手也不会有什么难度，值得一提的是，目前还没遇到什么 bug，这点挺不错的。

但是再多打开几个窗口，比如说 CMD、计算机管理什么的，还是可以看到很多当年 Win7 的影子，看来 Win11 依然还是一个两种设计语言并存的系统。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/20210616130923.jpg)

从最早的 xp，到 7、8、10 一直到 11 ，不得不说 Windows 确实是变得越来越好看了，现在的 Windows 给我的感觉就是微软有品味了但又不完全有，希望 Windows 继续努力吧，我还是继续用我的 Mac 了。

> 更多信息欢迎关注我的微信公众号：TeamsSix

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
