---
title: webpack5模块联邦
cover: /img/thumbnail/学习/前端/webpack/webpack.png
thumbnail: /img/thumbnail/学习/前端/webpack/webpack.png
date: 2021-12-28 18:16:09
updated: 2021-12-28 18:16:09
toc: true
categories: 
- 学习
- 前端
tags: 
- vue
- webpack
---

[demo](https://github.com/qianduanzhou/moduleFederation-study)

## 介绍
[官网](https://webpack.docschina.org/concepts/module-federation/)
多个独立的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署它们。
这通常被称作微前端，但并不仅限于此。
简单来说就是可以实现不同环境下的组件数据互通。
<!--more-->
## webpack5模块联邦测试
### appA 子容器
``` javascript
//appA/build/webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");

module.exports = {
    mode: 'production',
    entry: path.resolve(__dirname, '../src/index.js'),
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, '../dist'),
    },
    plugins: [
        new HtmlWebpackPlugin(),
        new ModuleFederationPlugin({
            name: 'appA',
            library: { type: "var", name: "appA" },
            filename: 'appA.js',
            exposes: {
                './Button': "./src/component/buttton"
            }
        })
    ],
};
```
### appB 父容器
``` javascript
//appB/build/webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");

module.exports = {
    mode: 'production',
    entry: path.resolve(__dirname, '../src/index.js'),
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, '../dist'),
    },
    plugins: [
        new HtmlWebpackPlugin(),
        new ModuleFederationPlugin({
            name: 'appB',
            remotes: {
                appA: "appA@http://127.0.0.1:5500/appA/dist/appA.js",
            },
        })
    ],
};
```
### appB B中引用了A的button组件