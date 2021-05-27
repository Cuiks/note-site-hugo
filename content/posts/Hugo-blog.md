---
title: "Hugo + 腾讯云CloudBase 搭建个人博客"
date: 2021-03-30T20:50:09+08:00
tags: ["hugo", "CloudBase", "blog"]
draft: false
---

<!--more-->

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
  - 已购买SSL证书（可以免费申请一年证书，各大云服务厂商也都会提供一键申请服务）

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

#### 1、服务器端配置

最初考虑的方案是使用`github`的`pages`服务，之前的`hexo`也是使用的`pages`，但是该服务存在两个问题。

- 访问不通畅
- 经常404

然后考虑的是使用`gitee`的`pages`服务。这样最大的优势就是免费。但是我比较习惯使用`github`，并且考虑到响应速度和404的问题，最终决定使用云服务厂商的文件存储及CDN加速。后续可能会使用`gitee`做仓库备份和中转，因为`github`慢

之所以选用腾讯云，原因很简单：**便宜，便宜，便宜**:+1:，并且还有**免费额度，免费额度，免费额度**:+1:。对我们这种三分钟热度的简直太友好了。

最初定的方案是准备使用`腾讯云COS+CDN`部署服务。但是注册腾讯云的时候发现他们提供**静态网站托管服务**，这也太爽了。到底有多爽的呢：他们的官方文档手把手教给你 创建Hugo + GitHub托管 + 腾讯云部署 + GitHub CI/CD。我* 简直是神仙产品啊。（当然，他们提供的部分教程可能也并不完善，后续会说明 *坑1）

##### 服务购买及开通

直接访问[腾讯云控制台](https://console.cloud.tencent.com/)， 注册登录一气呵成。左上角“云产品--静态网站托管”，服务开通，一把梭。然后选择按量付费开通静态托管服务。因为腾讯与会给一些免费使用额度，所以是可以放心开通的，[腾讯云CloudBase免费额度](https://cloud.tencent.com/document/product/876/47816)。CloudBase就是腾讯云的ServerLess服务。（后面会讲 *坑2）

![CloudBase免费额度](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/cloudbase-免费额度.png)

创建模板：选择空模板，下一步，按量付费&开启免费资源，下一步，开通。

注：腾讯云静态网站托管使用的是`CDN+COS`部署，所以速度和稳定性是很不错的。

##### 服务配置

开通成功后，可以看到下面页面。我这边已经配置好了域名的解析。点击基础配置，添加域名。[域名配置](https://cloud.tencent.com/document/product/1210/42862)

![静态网站配置信息](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/静态网站托管配置信息.png)

到这里服务的配置基本完成。

#### 2、GitHub CI/CD

上面我是使用GitHub作为远程仓库，最终想要的效果是：本地push代码到远程仓库，远程仓库部署至静态资源代理服务器。（当前编译在本地，后续放在github action *坑3）

直接整活：

进入github项目页面，点击Action，然后"Skip this and set up a workflow yourself "。github会在项目根目录下的创建`.github/workflows/main.yml` [我的配置文件](https://github.com/Cuiks/note-site-hugo/blob/master/.github/workflows/tcb.yml)。此处用到的secrets key等可参考[腾讯云文档](https://cloud.tencent.com/document/product/1210/52636)，或自行google github secrets key设置。

配置文件里面deploy的部署action我是基于腾讯云提供的脚本，自己改写的[action脚本](https://github.com/Cuiks/tcb-action)，这个脚本在腾讯云提供脚本基础上添加了部署时，首先删除之前的文件，不然服务端之前的文件会一直存在不会删除。

到这里。我们就实现了push代码，自动部署至腾讯云静态网站托管。

![github action](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/github-action.png)



### 填坑

#### 坑1：

这里说腾讯云提供的教程不完善，主要是指自动部署的配置文件。他们提供的[配置文件](https://github.com/TencentCloudBase/cloudbase-action)最新的2.0版本运行时报错找不到`cloudbaserc.json`文件。该文件主要是为了使用`tcb framework deploy`命令 [行41](https://github.com/TencentCloudBase/cloudbase-action/blob/master/%40v2/run.sh)。于是我直接修改该脚本，使用CLI的方式进行部署，并在部署前添加清理历史数据问题。

注：处理该问题带来的问题，因为会删除文件，会导致10-30秒(取决于github action速度)网站404。因为是个人博客，暂时可以接受。

#### 坑2：

这里主要讲解腾讯云的免费额度及兜底方案。主要还是穷:cry:

因为在之前工作中做过服务的迁移，所以多云服务有一定的了解。也被LB的按量付费坑过，也看过serverless应用被攻击导致的流量暴增而造成的费用激增。因为静态网站托管使用的是按量付费，所以害怕被攻击导致的费用激增。就此问题咨询过腾讯云的技术，是否可以服务自动关闭，答案是：no。

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/hugo-腾讯云沟通记录.png)

他们技术给出方案是设置告警，然后手动关停。额度监控在“云开发 -- 环境 -- 安全配置 -- 额度监控”

![](https://note-site-pic-1259606004.cos.ap-beijing.myqcloud.com/img/hugo-额度监控.png)

这里就涉及到阈值的问题。文章前面讲到过腾讯云的免费额度。

腾讯云CloudBase按量计费免费额度规则：[DOC](https://cloud.tencent.com/document/product/876/47816)

- 每月更新免费用量
  - 云存储
    - 容量  5G/月
    - 下载次数  2000/月
    - 上传次数  1000/月
    - CDN回溯流量  1G/月
  - CDN
    - 流量  1G/月
  - 云函数
    - 资源使用量  1000GBs/月
  - 数据库
    - 读次数  500/天
    - 写次数  300/天
- 一次性免费用量
  - 静态网站托管
    - 流量  5G/月

按照上面的限制，把资源限额平分到每天，然后设置监控告警，这样就能实现0费用。

第二个月开始，静态网站托管流量不再赠送，网站托管流量是0.21元/GB。所以服务部署运行近乎0成本。

#### 坑3：

关于hugo编译的问题。当前编译是放在本地生成静态文件，然后push到github。后续会继续完善action，把项目的编译也自动完成，实现完整的CI/CD。





