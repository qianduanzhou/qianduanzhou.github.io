---
title: docker学习
cover: /img/thumbnail/学习/后端/docker/docker.png
thumbnail: /img/thumbnail/学习/后端/docker/docker.png
date: 2021-12-15 18:21:09
updated: 2021-12-15 18:21:09
toc: true
categories: 
- 学习
- 后端
tags: docker
---

## <font color=#a862ea>相关文档</font>

[菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)
[官网](https://www.docker.com/get-started)
<!--more-->
## <font color=#a862ea>vue3部署</font>

1. 先执行npm run build打包
2. 创建一个Dockerfile文件，写入docker相关配置
3. 通过Dockerfile生成container

## <font color=#a862ea>mysql部署</font>

参考菜鸟教程，记得创建对应数据库

[docker-mysql](https://www.runoob.com/docker/docker-install-mysql.html "创建mysql映像及容器")

## <font color=#a862ea>beego部署</font>

1. 先执行bee pack -be GOOS=linux
2. 创建一个Dockerfile文件，写入docker相关配置
3. 与mysql关联，通过network创建一个公用网络
4. 通过Dockerfile生成同个network下的container

## <font color=#a862ea>容器连接</font>
连接mysql和golang
docker network create -d bridge 网络名称
网络名称对应docker-compose.yml里面的network_mode

## <font color=#a862ea>Docker Compose</font>

Compose 是用于定义和运行多容器 Docker 应用程序的工具。

[docker compose](https://www.runoob.com/docker/docker-compose.html "docker compose")