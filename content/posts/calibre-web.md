---
title: "Docker安装Calibre Web电子书应用程序"
date: 2022-06-30T15:16:26+08:00
tags: ["docker"]
draft: false
---

- 写在前面
- 安装步骤

<!--more-->

#### 写在前面
`Calibre Web`用于浏览、阅读和下载存储在 Calibre 数据库中的电子书的 Web 应用程序。
项目地址`https://github.com/janeczku/calibre-web`

#### 安装前准备
注：
- 本文基于 Docker & docker-compose 安装
- calibre-web docker 镜像常用的有两个版本。
	- https://github.com/linuxserver/docker-calibre-web
	- https://hub.docker.com/r/technosoft2000/calibre-web/
- 前者基于`linuxserver`的版本简洁，但是没有在线阅，转码，数据库等
- 本文基于后者镜像搭建

#### 安装
1. docker-compose.yml
```
version: "2.1"
services:
  calibre-web:
    image: technosoft2000/calibre-web
    container_name: calibre-web
    volumes:
      - /mnt/docker/calibre/data/:/config
      - /mnt/docker/calibre/library:/books
    ports:
      - 8083:8083
    restart: unless-stopped
```
2. 挂载目录解释
	- `/mnt/docker/calibre/data/ & /mnt/docker/calibre/library` 为自己创建目录，最好给予777的权限，我懒得试别的权限是否可行
	- chmod 777 /mnt/docker/calibre/data/
3. docker-compose up -d
4. 服务默认运行在8083端口
5. 默认用户名密码：admin & admin123
6. 首次登录后需要选择电子书数据库位置，选择`/books`即可