---
title: jenkins构建react项目
date: 2022-04-25 16:17:35
cover: /img/thumbnail/学习/后端/jenkins/jenkins.png
thumbnail: /img/thumbnail/学习/后端/jenkins/jenkins.png
tags:
- jenkins
- react
toc: true
---

## <font color=#a862ea>node插件</font>

1. 首先需要在Jenkins里安装node插件，然后配置，在系统管理里面的插件管理搜索node，然后安装并重启

<!--more-->

{% asset_img 安装node.png %}

2. 然后进行配置

新建一个任务，选择第一项，自由风格，然后在构建环境栏中进行node配置

{% asset_img 构建配置.png %}

## <font color=#a862ea>任务配置</font>

在新建的任务中进行相关配置。

1. 可以填写github的地址
2. 源码管理，这里填入的是github的clone地址（ssh），然后配置上密钥和项目分支。
3. 最后则是编写构建栏的代码，可以选择执行shell，并输入以下命令。

```shell
npm install --registry=https://registry.npm.taobao.org #使用淘宝镜像
export CI=false #防止CI报错
npm run build #执行打包命令
docker build -t nginx/react:v1.1.0 . #根据dockerfile生成镜像
[ "$(docker ps | grep three-shopping-mall)" ] && docker kill -s KILL three-shopping-mall #如果容器处于运行状态则清除
[ "$(docker ps -a | grep three-shopping-mall)" ] && docker rm three-shopping-mall #如果容器存在则移除
docker run -d -p 8030:80 --name three-shopping-mall nginx/react:v1.1.0 #生成容器
```

{% asset_img 构建.png %}

4. 由于服务器内存太小，需要对内存进行swap，不然构建经常会崩溃。

```sh
#操作命令：
#查看：
free -h
#创建（可以和内存1：1,内存1g可以用1024）：
dd if=/dev/zero of=/data/swapfile bs=1M count=1024 
#设置交换文件；
mkswap /data/swapfile
#启用：swapon /data/swapfile
```

其他命令

```shell
#停用
swapoff /data/swapfile
#删除
rm /data/swapfile -rf
```

## <font color=#a862ea>项目相关配置</font>

由于使用到docker，需要在项目下创建Dockerfile如下：

```dockerfile
FROM nginx
MAINTAINER zhb
COPY build  /usr/share/nginx/html/ 
COPY config/nginx  /etc/nginx/conf.d/
```

这里对打包后的文件进行拷贝到对应目录，还有就是将nginx配置也拷贝过去。

```nginx
server {
  listen       80;
  listen  [::]:80;
  server_name  localhost;
  #access_log  /var/log/nginx/host.access.log  main;
  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
}
```

