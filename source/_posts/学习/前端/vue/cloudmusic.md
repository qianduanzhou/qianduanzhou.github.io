---
title: vue+electron高仿网易云
cover: /img/thumbnail/学习/前端/vue/cloudmusic.jpg
thumbnail: /img/thumbnail/学习/前端/vue/cloudmusic.jpg
date: 2019-06-13 09:44:00
updated: 2019-06-13 09:44:00
toc: true
categories: 
- 学习
- 前端
tags: 
- vue
- electron
- node
---
### <font color=#a862ea>介绍</font>

 - 用vue+electron 高仿网易云pc版，后台api是网上一位大佬根据网易云api获取的
 - 这是api地址：[网易云api](https://binaryify.github.io/NeteaseCloudMusicApi/#/)
 - 后台文件到上面github地址下载，下载后的文件为NeteaseCloudMusicApi

<!--more-->

### <font color=#a862ea>安装教程</font>

1. 两个文件都需要npm install
2. 安装完成后在 NeteaseCloudMusicApi 文件里启动node服务，node app.js
3. 在 cloudmusic里启动vue，npm run dev

### <font color=#a862ea>上线（已失效）</font>

 - 目前已上线阿里云，打包好的exe文件在百度云：
 - 链接：https://pan.baidu.com/s/13VIKegK4mmSF1ds9FqP6mA
 - 提取码：jq98 
 - 打开exe文件即可一键安装，然后在桌面直接打开即可

### <font color=#a862ea>技术栈</font>

基本就是vue全家桶，还有外壳electron，后台api是网上一位大佬的
 - vue
 - vuex
 - axios
 - vue-router
 - electron
 - node

### <font color=#a862ea>gif动图演示</font>

{% asset_img cloudmusic.gif %}

### <font color=#a862ea>功能介绍</font>

#### <font color=#a862ea>登录</font>

 - 可以根据自己的手机号登录网易云

#### <font color=#a862ea>查看个人收藏，包括歌曲，歌单，专辑，歌手</font>

 - 登录后即可查看这些信息，如下图

{% asset_img 1.png%}

#### <font color=#a862ea>歌单大全，歌手大全，排行榜等</font>

- 这是其中的一张，歌手大全

{% asset_img 2.png%}

#### <font color=#a862ea>此外还有一些功能，如收藏，发表评论等</font>

- 喜欢音乐

{% asset_img 3.png%}

#### <font color=#a862ea>当然少不了音乐播放相关的操作</font>

如播放，暂停，下一首等操作
