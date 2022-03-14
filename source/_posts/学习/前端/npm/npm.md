---
title: npm发包流程
cover: /img/thumbnail/学习/前端/npm/npm.png
thumbnail: /img/thumbnail/学习/前端/npm/npm.png
date: 2022-01-25 11:34:50
updated: 2022-01-25 11:34:50
toc: true
categories: 
- 学习
- 前端
tags: npm
---
### <font color=#a862ea>npm发布包相关操作</font>
<!--more-->
**登录：**npm login

**发布及更新：**npm publish

**更新版本：**执行以下命令后 npm publish

- npm version patch作用将修订号增加1，也就是版本号最后一位
- npm version minor将次版本号增加1，也就是中间那一位
- npm version major将主版本号增加1，也就是第一位

**删除版本：**npm unpublish 包名@版本号

**删除整个包：**npm unpublish 包名 --force
