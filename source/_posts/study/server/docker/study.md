---
title: docker学习
cover: /img/thumbnail/study/server/docker/docker.png
thumbnail: /img/thumbnail/study/server/docker/docker.png
date: 2021-12-17 15:43:19
updated: 2021-12-17 15:43:19
toc: true
categories: 
- study
- server
- docker
tags: docker
---

## 相关文档

[菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)
[官网](https://www.docker.com/get-started)

## vue3部署

1. 先执行npm run build打包
2. 创建一个Dockerfile文件，写入docker相关配置
3. 通过Dockerfile生成container

## mysql部署

参考菜鸟教程，记得创建对应数据库

[docker-mysql](https://www.runoob.com/docker/docker-install-mysql.html "创建mysql映像及容器")

## beego部署

1. 先执行bee pack -be GOOS=linux
2. 创建一个Dockerfile文件，写入docker相关配置
3. 与mysql关联，通过network创建一个公用网络
4. 通过Dockerfile生成同个network下的container

## 容器连接
连接mysql和golang
docker network create -d bridge 网络名称
网络名称对应docker-compose.yml里面的network_mode

## Docker Compose

Compose 是用于定义和运行多容器 Docker 应用程序的工具。

[docker compose](https://www.runoob.com/docker/docker-compose.html "docker compose")