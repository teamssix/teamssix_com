---
title: 【云原生】Terraform 初体验
date: 2022-04-08 14:56:22
id: 220408-145622
img: https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081515182.png
tags:
- 云原生
- 笔记
- Terraform
- 漏洞复现
categories:
- 云原生
---

# 0x00 前言

Terraform 是一种安全有效地构建、更改和版本控制基础设施的工具(基础架构自动化的编排工具)。

简单的说就是可以通过编写一些类似于 JSON 格式的文件，直接创建一批云上的服务资源，Terraform 和  AWS 的 CloudFormation 产品有些类似，但 CloudFormation 只支持 AWS，于是 HashiCorp 公司打造了一个多云 (Multi Cloud) 的开源的基础设施即代码 (IaC) 工具，即 Terraform

# 0x01 安装

Terraform 的安装很简单，不同操作系统的安装命令如下：

- Ubuntu

```plain
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository -y "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install -y terraform
```

- Centos

```plain
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform
```

- Mac

```plain
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

- Windows

```plain
choco install terraform
```

或者直接到 Terraform 官网下载可执行文件使用，官方下载地址：https://www.terraform.io/downloads

# 0x02 初体验

在使用 Terraform 之前，需要先在对应的云厂商控制台上生成一个 Access Key，这里以在 AWS 上创建一个 S3 服务为例。

首先新建一个文件夹，例如 demo 文件夹，接着在里面创建一个 s3demo.tf 文件，文件内容如下：

```plain
provider "aws" {
  region     = "us-west-1"
  access_key = "your-access-key"
  secret_key = "your-secret-key"
}

resource "aws_s3_bucket" "b" {
  bucket = "my-tf-test-bucket-asdqqsdasd"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}

resource "aws_s3_bucket_acl" "example" {
  bucket = aws_s3_bucket.b.id
  acl    = "private"
}
```

tf 文件采用的是 HCL 格式，HCL 格式是 Terraform 所属公司 HashiCorp 自己设计的一套配置语言

在 demo 文件夹下，运行一下初始化命令，这时 Terraform 会通过官方插件仓库下载对应的 Provider 插件

```plain
terraform init
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081514038.png)

因为我们这里的 s3demo.tf 里的 Provider 是 AWS，所以在初始化时，Terraform 就会去下载 AWS 的 Provider 插件

在 https://registry.terraform.io/browse/providers 可以看到 Terraform 所支持的厂商，这里基本上是涵盖了大部分云厂商的。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081515182.png)

接着使用 plan 命令查看接下来将要产生的变更

```plain
terraform plan
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081515816.png)

如果没什么问题，就可以应用了

```plain
terraform apply
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081516114.png)

中途会提示确认，输入 yes 即可



在 Terraform 执行完之后，查看 AWS 下的 S3 就可以看到刚刚通过 Terraform 创建的资源了。

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081516221.png)

这样就完成了使用 Terraform 部署云资源的一个过程，想要清理刚刚创建的资源也非常简单，直接 destroy 即可

```plain
terraform destroy
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081517821.png)

# 0x03 一些有意思的

## 1、启动插件缓存

在刚刚进行 init 初始化时，Terraform 会根据 tf 文件内的 Provider 下载对应的插件，这些插件往往体积比较大，例如上面初始化时下载的 AWS Provider 体积就有两百多 M，如果不启用插件缓存，那么在每个 Terraform 项目中都会反复下载这些插件，就很浪费磁盘空间与流量，因此建议将插件缓存开启。



Windows 下是在相关用户的 %APPDATA% 目录下创建名为 "terraform.rc" 的文件，Macos 和 Linux 用户则是在用户的 home 下创建名为 ".terraformrc" 的文件

.terraformrc 文件内容为：

```plain
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
```

这样每次下载 Provider 插件时，就会下载到 "$HOME/.terraform.d/plugin-cache" 目录下了，不过 Terraform 不会主动清理这个文件夹，因此可能随着插件版本的更迭，这个文件夹内会保存一些历史版本的 Provider 插件，这时就需要自己手动清理一下了。

## 2、可视化 Terraform

如果 Terraform 项目比较复杂，那么可以利用 tfviz 这个工具，可视化 Terraform 项目，tfviz 项目地址：https://github.com/steeve85/tfviz

安装

```plain
GO111MODULE=on go get -u github.com/steeve85/tfviz
```

到 Terraform 项目目录下使用

```plain
tfviz -input ./ -output tfimg.png
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081517920.png)

## 3、Terraform 代码安全性检查

如果想知道自己写的 Terraform 项目代码有没有什么安全风险，那么可以使用 tfsec 这个工具，tfsec 项目地址：https://github.com/aquasecurity/tfsec



Mac 可以直接使用 brew 安装

```plain
brew install tfsec
```

或者使用 go install 安装

```plain
go install github.com/aquasecurity/tfsec/cmd/tfsec@latest
```



使用也非常简单，直接来到 Terraform 项目目录下，使用 tfsec . 命令即可

```plain
tfsec .
```

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/202204081518327.png)

>  更多信息欢迎关注我的个人微信公众号：TeamsSix
>
>  参考文章：
>
>  [https://www.cnblogs.com/sparkdev/p/10052310.html](https://www.cnblogs.com/sparkdev/p/10052310.html)

![](https://cdn.jsdelivr.net/gh/teamssix/BlogImages/imgs/TeamsSix_Subscription_Logo2.png)
