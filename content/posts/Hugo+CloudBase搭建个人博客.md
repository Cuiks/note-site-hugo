---
title: "Hugo + CloudBase 搭建个人博客"
date: 2021-03-30T20:50:09+08:00
draft: true
---

### 前言

当前有很多成熟的个人博客搭建方案。

例如动态博客`WordPress`，静态博客`Hexo`、`Jekyll`等。相较于`Wordpress`，静态博客的使用及部署成本更低，更便于迁移。

选择`Hugo`主要是出于好奇，因为`Hugo`官网给出的slogan：`The world’s fastest framework for building websites` :sunglasses:

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/hugo_slogan.png)

### 准备工作

- Hugo [安装教程](https://gohugo.io/getting-started/installing/)

  - ```shell
    # windows 直接下载releases安装。然后设置环境变量添加hugo
    https://github.com/gohugoio/hugo/releases
    
    # mac。先安装homebrew 然后brew安装hugo
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    brew install hugo
    
    # linux。github releases下载安装 或 使用包管理工具
    sudo apt-get install hugo
    sudo pacman -Sy hugo
    sudo dnf install hugo
    ```

- git  [下载地址](https://git-scm.com/downloads)

- 远程仓库(本文使用GitHub)

- 腾讯云

  - 云开发 CloudBase -- 静态网站托管
  - 对象存储

- 域名

  - 已备案
  - 已购买SSL证书

### Hugo搭建网站几个环节（各环节的解决方案）

- 框架学习
- 代码的版本控制及存储
- 服务的部署
  - 服务器端配置
  - GitHub CI/CD
- 成本问题

### 框架学习

Hugo是一种用Go语言编写的快速，现代的静态网站生成器，旨在让网站创建再次变得有趣。（照抄原文:neutral_face:）

刚开始使用Hugo所以对Hugo也不熟悉，这边只会列举几条简单常用的命令，后续使用中继续深入了解Hugo:speak_no_evil:

[创建站点](https://gohugo.io/getting-started/quick-start/#step-2-create-a-new-site)

- `hugo new site quickstart`

[主题安装](https://gohugo.io/getting-started/quick-start/#step-3-add-a-theme)

- Hugo官方提供了多种[主题][https://themes.gohugo.io/]。下载到站点下的themes目录下

- ```shell
  cd quickstart
  git init
  git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
  # 使用主题
  echo theme = \"ananke\" >> config.toml
  ```

[新建文章](https://gohugo.io/getting-started/quick-start/#step-4-add-some-content)

- `hugo new posts/my-first-post.md`

[启动服务](https://gohugo.io/getting-started/quick-start/#step-5-start-the-hugo-server)

- `hugo server -D`

[生成静态文件](https://gohugo.io/getting-started/quick-start/#step-7-build-static-pages)

- `hugo -D`
- 该命令会在当前文件夹生成`public/`文件夹，包含生成的静态文件

### 代码的版本控制及存储

我是使用`git`作为我的版本控制工具，远程仓库使用`GitHub`。相关使用及配置可以自行Google。

### 服务的部署

##### 1、服务器端配置

最初考虑的方案是使用`github`的`pages`服务，之前的`hexo`也是使用的`pages`，但是该服务存在两个问题。

- 访问不通畅
- 经常404

然后考虑的是使用`gitee`的`pages`服务。这样最大的优势就是免费。但是我比较习惯使用`github`，并且考虑到响应速度和404的问题，最终决定使用云服务厂商的文件存储及CDN加速。

之所以选用腾讯云，原因很简单**<便宜，便宜，便宜>:+1:**，并且还有**<免费额度，免费额度，免费额度>**:+1:。对我们这种三分钟热度的简直太友好了。最初定的方案是准备使用`腾讯云COS+CDN`部署服务。但是注册腾讯云的时候发现他们提供**<静态网站托管服务>**，这也太爽了。到底有多爽的呢，就是他们的官方文档手把手教给你怎么创建Hugo + GitHub托管 + 腾讯云部署 + GitHub CI/CD。我* 简直是神仙产品啊。



## To Be Continue

