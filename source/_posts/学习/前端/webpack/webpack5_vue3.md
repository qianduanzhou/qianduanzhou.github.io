---
title: webpack5搭建vue3
cover: /img/thumbnail/学习/前端/webpack/webpack.png
thumbnail: /img/thumbnail/学习/前端/webpack/webpack.png
date: 2022-01-10 18:42:09
updated: 2022-01-10 18:42:09
toc: true
categories: 
- 学习
- 前端
tags: 
- vue
- webpack
- 项目
---

[demo](https://github.com/qianduanzhou/webpack-vue3)
# <font color=#a862ea>开始</font>
## <font color=#a862ea>初始化</font>

- npm init创建package.json
- 然后安装webpack相关插件

```
npm i webpack webpack-cli --save-dev
```

<!--more-->

- 创建build文件夹并新建webpack配置文件

```javascript
const path = require('path');

function resolvePath(filePath) {
    return path.resolve(__dirname, filePath);
}

module.exports = {
    mode: 'production',
    entry: resolvePath('../src/main.js'),
    output: {
        path: resolvePath('../dist'),
        filename: '[contenthash].bundle.js',
    },
    module: {
        rules: [],
    },
    plugins: [
        new webpack.DefinePlugin({
            BASE_URL: '"../public/"'
        })
    ],
};
```



## <font color=#a862ea>安装vue相关依赖</font>

- 安装vue相关依赖

```
npm i vue -S                 //-S 等于 --save   -D 等于 --save-dev   i 等于 install
npm i vue-loader -D          //解析和转换.vue文件。提取出其中的逻辑代码 script,样式代码style,以及HTML 模板template，再分别把他们交给对应的loader去处理  
npm i @vue/compiler-sfc -D   //将解析完的vue单页面组件(sfc)编译为js
```

- 配置vue-loader

```javascript
//webpack.config.js
const { VueLoaderPlugin } = require('vue-loader');
module: {
    rules: [
        {
            test: /\.vue$/,
            loader: 'vue-loader',
        },
    ];
}
plugins: [
        new VueLoaderPlugin(),
]
```

- 创建main.js文件

```javascript
import { createApp } from 'vue'
import App from './App.vue'       // 测试文件 App.vue 和 用于挂载的index.html 自行创建
createApp(App).mount('#app')
```

## <font color=#a862ea>安装css，sass相关依赖</font>

- 安装

```js
npm i css-loader style-loader sass sass-loader -D
```

- 配置loader

```javascript
    module: {
        {
            test: /\.(sa)|css$/,
            use: [{
                loader: 'style-loader'
            }, {
                loader: 'css-loader'
            }, {
                loader: 'sass-loader'
            }]
        }],
    },
```

## <font color=#a862ea>使用模板</font>

- 安装

```js
npm i html-webpack-plugin -D  //在打包结束后，自动生成一个html文件，并把打包生成的js文件引入到这个html文件当中
```

- 配置plugins

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); 
plugins: [
    new webpack.DefinePlugin({
        BASE_URL: '"./"'
    }),
    new HtmlWebpackPlugin({
        title: 'vue3-app',
        filename: 'index.html',
        template: resolvePath('../public/index.html'),
    }),
],
```

- 创建index.html

```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

## <font color=#a862ea>自动删除文件</font>

- 安装

```js
npm i clean-webpack-plugin -D  //在打包之前清空output配置的文件夹
```

- 配置

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
plugins: [
    new CleanWebpackPlugin()
]
```

## <font color=#a862ea>拷贝文件</font>

```
npm i copy-webpack-plugin --save-dev
```

```js
const CopyWebpackPlugin = require('copy-webpack-plugin');
   plugins: [
        new CopyWebpackPlugin({
            patterns: [
                {
                    from: resolvePath('../public/favicon.ico'),
                    to: resolvePath('../dist/favicon.ico'),
                }
            ]
        })
    ],
```

## <font color=#a862ea>搭建开发环境</font>

拷贝一份webpack.config.js文件，命名为webpack.config.dev.js，删除拷贝插件相关的代码

- 安装

```js
npm i webpack-dev-server -D  //以file形式在浏览器打开打包后的文件，无法发送ajax请求。所以需要devServer在本地开启一个服务器，以http的形式发送请求。
```

- 配置

```javascript
  devServer: {
        static: {
          directory: resolvePath('../public'),
        },
        open: true,
        hot: true,
        compress: true,
        port: 9000,
  },
```

# <font color=#a862ea>优化效率</font>

## <font color=#a862ea>工具</font>

- [progress-bar-webpack-plugin](https://www.npmjs.com/package/progress-bar-webpack-plugin)：查看编译进度；
- [speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)：查看编译速度；
- [webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)：打包体积分析。

## <font color=#a862ea>加快构建速度</font>

### <font color=#a862ea>更新webpack版本为5</font>

### <font color=#a862ea>缓存</font>

#### <font color=#a862ea>cache</font>

通过配置 [webpack 持久化缓存](https://webpack.docschina.org/configuration/cache/#root) `cache: filesystem`，来缓存生成的 webpack 模块和 <font color=#a862ea>chunk，改善构建速度。</font>

简单来说，通过 `cache: filesystem` 可以将构建过程的 webpack 模板进行缓存，大幅提升二次构建速度、打包速度，当构建突然中断，二次进行构建时，可以直接从缓存中拉取，可提速 **90%** 左右。

配置如下：

```javascript
module.exports = {
  cache: {
    type: "filesystem", // 使用文件缓存
  },
};
```

### <font color=#a862ea>减少 loader、plugins</font>

#### <font color=#a862ea>指定 include</font>

为 loader 指定 include，减少 loader 应用范围，仅应用于最少数量的必要模块。

rule.exclude 可以排除模块范围，也可用于减少 loader 应用范围.

#### <font color=#a862ea>管理资源</font>

使用 [webpack 资源模块](https://webpack.docschina.org/guides/asset-modules/) (asset module) 代替旧的 assets loader（如  `file-loader`/`url-loader`/`raw-loader` 等），减少 loader 配置数量。

配置方式如下：

```javascript
module.exports = {
  rules: [
    {
      test: /\.(png|svg|jpg|jpeg|gif)$/i,
      include: [resolvePath('../src')],
      type: "asset/resource",
    },
  ],
};
```

### <font color=#a862ea>优化 resolve 配置</font>

#### <font color=#a862ea>alias</font>

alias 可以创建  `import` 或  `require` 的别名，用来简化模块引入。

配置方式如下：

```javascript
module.exports = {
  resolve: {
    alias: {
      "@": resolvePath('../src'), // @ 代表 src 路径
    },
  },
};
```

#### <font color=#a862ea>extensions</font>

extensions 表示需要解析的文件类型列表。

根据项目中的文件类型，定义 extensions，以覆盖 webpack 默认的 extensions，加快解析速度。

由于 webpack 的解析顺序是从左到右

`webpack.common.js` 配置方式如下：

```javascript
module.exports = {
  resolve: {
    extensions: [".js"],
  },
};
```

### <font color=#a862ea>多进程</font>

#### <font color=#a862ea>thread-loader</font>

通过 [thread-loader](https://webpack.docschina.org/loaders/thread-loader/#root) 将耗时的 loader 放在一个独立的 worker 池中运行，加快 loader 构建速度。

安装：

```javascript
npm i -D thread-loader
```

`webpack.common.js` 配置方式如下：

```javascript
module.exports = {
  rules: [
    {
            test: /\.s?css$/,
            include: resolvePath('../src'),
            use: [
                'style-loader',
                NODE_ENV !== 'dev' && MiniCssExtractPlugin.loader, //分离css
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        importLoaders: 2,
                    },
                },
                'sass-loader',
                {
                    loader: "thread-loader",
                    options: {
                        workerParallelJobs: 2,
                    },
                }
            ]
        },
  ],
};
```

# <font color=#a862ea>减小打包体积</font>

## <font color=#a862ea>代码压缩</font>

### <font color=#a862ea>JS 压缩</font>

使用  [TerserWebpackPlugin](https://webpack.docschina.org/plugins/terser-webpack-plugin/) 来压缩 JavaScript。

webpack5 自带最新的  `terser-webpack-plugin`，无需手动安装。

```
terser-webpack-plugin` 默认开启了 `parallel: true` 配置，并发运行的默认数量： `os.cpus().length - 1
```

 配置方式如下：

```javascript
const TerserPlugin = require("terser-webpack-plugin");
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: 4,
        terserOptions: {
          parse: {
            ecma: 8,
          },
          compress: {
            ecma: 5,
            warnings: false,
            comparisons: false,
            inline: 2,
          },
          mangle: {
            safari10: true,
          },
          output: {
            ecma: 5,
            comments: false,
            ascii_only: true,
          },
        },
      }),
    ],
  },
};
```

你可能有听过 ParallelUglifyPlugin 插件，它可以帮助我们多进程压缩 JS，webpack5 的 TerserWebpackPlugin 默认就开启了多进程和缓存，无需再引入 ParallelUglifyPlugin。

### <font color=#a862ea>CSS 压缩</font>

使用 [CssMinimizerWebpackPlugin](https://webpack.docschina.org/plugins/css-minimizer-webpack-plugin/#root) 压缩 CSS 文件。

和 [optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin) 相比，[css-minimizer-webpack-plugin](https://webpack.docschina.org/plugins/css-minimizer-webpack-plugin/#root) 在 source maps 和 assets 中使用查询字符串会更加准确，而且支持缓存和并发模式下运行。

`CssMinimizerWebpackPlugin` 将在 Webpack 构建期间搜索 CSS 文件，优化、压缩 CSS。

安装：

```
npm install -D css-minimizer-webpack-plugin
```

配置方式如下：

```javascript
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

module.exports = {
  optimization: {
    minimizer: [
      new CssMinimizerPlugin({
        parallel: 4,
      }),
    ],
  },
};
```

## <font color=#a862ea>代码分离</font>

代码分离能够把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的 bundle，以及控制资源加载优先级，可以缩短页面加载时间。

### <font color=#a862ea>抽离重复代码</font>

[SplitChunksPlugin](https://webpack.docschina.org/plugins/split-chunks-plugin) 插件开箱即用，可以将公共的依赖模块提取到已有的入口 chunk 中，或者提取到一个新生成的 chunk。

webpack 将根据以下条件自动拆分 chunks：

- 新的 chunk 可以被共享，或者模块来自于  `node_modules` 文件夹；
- 新的 chunk 体积大于 20kb（在进行 min+gz 之前的体积）；
- 当按需加载 chunks 时，并行请求的最大数量小于或等于 30；
- 当加载初始化页面时，并发请求的最大数量小于或等于 30；
  通过 splitChunks 把 react 等公共库抽离出来，不重复引入占用体积。

配置方式如下：

```javascript
module.exports = {
  splitChunks: {
    // include all types of chunks
    chunks: "all",
    // 重复打包问题
    cacheGroups: {
      vendors: {
        // node_modules里的代码
        test: /[\\/]node_modules[\\/]/,
        chunks: "all",
        // name: 'vendors', 一定不要定义固定的name
        priority: 10, // 优先级
        enforce: true,
      },
    },
  },
};
```

### <font color=#a862ea>CSS 文件分离</font>

[MiniCssExtractPlugin](https://webpack.docschina.org/plugins/mini-css-extract-plugin/) 插件将 CSS 提取到单独的文件中，为每个包含 CSS 的 JS 文件创建一个 CSS 文件，并且支持 CSS 和 SourceMaps 的按需加载。

安装：

```
npm install -D mini-css-extract-plugin
```

```js
module.exports = {
  rules: [
    {
            test: /\.s?css$/,
            include: resolvePath('../src'),
            use: [
                'style-loader',
                NODE_ENV !== 'dev' && MiniCssExtractPlugin.loader, //分离css
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        importLoaders: 2,
                    },
                },
                'sass-loader',
                {
                    loader: "thread-loader",
                    options: {
                        workerParallelJobs: 2,
                    },
                }
            ]
        },
  ],
};
```

**注意：MiniCssExtractPlugin.loader 要放在 style-loader 后面。**

### <font color=#a862ea>最小化 entry chunk</font>

通过配置 `optimization.runtimeChunk = true`，为运行时代码创建一个额外的 chunk，减少 entry chunk 体积，提高性能。

 配置方式如下：

```javascript
module.exports = {
    optimization: {
        runtimeChunk: true,
      },
    };
}
```

## <font color=#a862ea>Tree Shaking</font>

摇树，顾名思义，就是将枯黄的落叶摇下来，只留下树上活的叶子。枯黄的落叶代表项目中未引用的无用代码，活的树叶代表项目中实际用到的源码。

### <font color=#a862ea>JS</font>

[JS Tree Shaking](https://webpack.docschina.org/guides/tree-shaking/) 将 JavaScript 上下文中的未引用代码（Dead Code）移除，通过  `package.json` 的  `"sideEffects"` 属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是 "pure(纯正 ES2015 模块)"，由此可以安全地删除文件中未使用的部分。

Dead Code 一般具有以下几个特征：

- 代码不会被执行，不可到达；
- 代码执行的结果不会被用到；
- 代码只会影响死变量（只写不读）。

#### <font color=#a862ea>webpack5 sideEffects</font>

通过 package.json 的  `"sideEffects"` 属性，来实现这种方式。

```javascript
{
  "name": "your-project",
  "sideEffects": false
}
```

需注意的是，当代码有副作用时，需要将 `sideEffects` 改为提供一个数组，添加有副作用代码的文件路径：

```javascript
{
  "name": "your-project",
  "sideEffects": ["./src/some-side-effectful-file.js"]
}
```

### <font color=#a862ea>CSS</font>

上述对 JS 代码做了 Tree Shaking 操作，同样，CSS 代码也需要摇摇树，打包时把没有用的 CSS 代码摇走，可以大幅减少打包后的 CSS 文件大小。

使用 [purgecss-webpack-plugin](https://github.com/FullHuman/purgecss/tree/main/packages/purgecss-webpack-plugin) 对 CSS Tree Shaking。

安装：

```
npm i purgecss-webpack-plugin -D
```

因为打包时 CSS 默认放在 JS 文件内，因此要结合 webpack 分离 CSS 文件插件 `mini-css-extract-plugin` 一起使用，先将 CSS 文件分离，再进行 CSS Tree Shaking。

`webpack.prod.js` 配置方式如下：

```javascript
const glob = require("glob");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const PurgeCSSPlugin = require("purgecss-webpack-plugin");

module.exports = {
  plugins: [
    // CSS Tree Shaking
    new PurgeCSSPlugin({
      paths: glob.sync(`${resolvePath('../src')}/**/*`, { nodir: true }),
    }),
  ],
};
```

上面为了测试 CSS 压缩效果，我引入了大量无效 CSS 代码，因此 Tree Shaking 效果也非常明显。
