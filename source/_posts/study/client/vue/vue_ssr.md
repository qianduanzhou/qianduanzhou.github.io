---
title: Vue服务端渲染
cover: /img/thumbnail/study/client/vue/vue_ssr.png
thumbnail: /img/thumbnail/study/client/vue/vue_ssr.png
date: 2021-04-09 15:43:19
updated: 2021-04-13 15:43:19
toc: true
categories: 
- study
- client
tags: 
- vue
- ssr
- node
---

## 背景

​		传统的 php，jsp，asp 等运行在服务端的语言，它们都是等服务器端把数据和模板转换成最终的 HTML 并返回客户端显示。而如今前端主流的框架如Vue，React，Angular等基本都是基于客户端渲染，它们所构建的单页应用（SPA）具有渲染性能好、可维护性高等优点。但是带来的缺点也是明显的，如首屏加载时间过长、不利于SEO。
​       单页应用使用JavaScript在客户端生成HTML来呈现内容，用户需要等待JS解析执行完成才能看到页面，这就使得首屏加载时间变长，影响用户体验。此外当搜索引擎爬取网站HTML文件时，单页应用的HTML没有内容，从而影响搜索排名。
​       为了解决以上两点问题，可以对我们的应用程序使用服务器端渲染 (SSR) ，服务器渲染的 Vue.js 和React.js应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在服务器和客户端上运行。

<!--more-->

## vue服务端渲染方案

- Nuxt.js

**基础：vue**

Nuxt.js 是一个基于 Vue.js 的通用应用框架。通过对客户端/服务端基础架构的抽象组织，Nuxt.js 预设了利用 Vue.js 开发服务端渲染的应用所需要的各种配置。也就是说使用nuxt开发学习成本低，基本上学习过Vue.js都可以很快上手Nuxt.js。

[demo](https://github.com/qianduanzhou/nuxt-demo "demo地址")

- 脚手架搭建

**基础：vue、webpack、node**

但是，如果你需要更直接地控制应用程序的结构，Nuxt.js 并不适合这种使用场景，此时最好的方式就是从零开始搭建一个基于Vue的服务端渲染框架，可以参考[官方指南](https://ssr.vuejs.org/zh/)

[demo](https://github.com/qianduanzhou/vue-ssr)

## 实现

### Nuxt.js

#### 安装

通过npx create-nuxt-app <项目名> 创建一个nuxt应用程序。（npx 在 NPM 版本 5.2.0 默认安装了）

#### 目录结构介绍

这是最后生成的文件目录结构

{% asset_img 图片1.png 目录结构 %}

- pages

这个文件夹一般存放页面文件，在执行npm run dev后会自动按规则生成对应的路由。

- component

存放公用的组件，如loading等。

- layout

layouts 目录中的每个文件 (顶级) 都将创建一个可通过页面组件中的 layout 属性访问的自定义布局。

- middleware

中间件目录，中间件允许您定义一个自定义函数运行在一个页面或一组页面渲染之前。执行顺序：

1. nuxt.config.js
2. 匹配布局
3. 匹配页面

- plugins

Nuxt.js 允许你在运行 Vue.js 应用程序之前执行 js 插件。这在您需要使用自己的库或第三方模块时特别有用。
使用方式：在plugins文件夹下创建一个模块js，如element-ui.js，然后再引入elementui并通过vue.use()方法注册，然后在nuxt.config.js中的plugins选项中加入该文件，由于是整个element引入，所以还得在css选项里加上element-ui的样式文件。这样我们就可以使用elementui的功能了。

#### 路由

nuxt.js可以自动生成路由，也可以手动配置。

##### 自动生成

可以根据nuxt的路由生成规则建立相应的文件结构，如下：

{% asset_img 图片2.png %}

{% asset_img 图片3.png %}

##### 手动创建

我们也可以手动创建路由，主要用到router选项下的extendRoutes创建自定义路由。
例如：新建一个router.js文件，在里面写入以下代码，然后在nuxt.config.js里引入并使用，最后生成的就是我们的自定义路由。

```js
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)
//() => import(/* webpackChunkName: "index" */'view/index.vue')
export function createRouter() {
	return new Router({
		mode: 'history',
		routes: [
			{
				"path": "/",
				"component": () => import(/* webpackChunkName: "index" */'view/index.vue')
			},
			{
				"path": "/item/:id",
				"component": () => import(/* webpackChunkName: "item-_id-index" */'view/item/_id/index.vue')
			}
		]
	})
}
```

#### 生命周期钩子

nuxt.js除了常见的vue生命周期外，还有多出来的两个生命周期钩子，asyncData和fetch

##### asyncData

- 这个钩子在客户端和服务端都会执行，它可以是异步的，并接收上下文作为参数。返回的对象将与data对象合并。由于这个方法是在组件初始化之前被调用的，所以无法通过this来使用组件的方法和属性。
- 第一次加载页面时会先执行这个钩子，所有在这个时候不能使用document等客户端的对象和方法，否则会报错。由于除了页面刷新加载会执行这个钩子，在路由跳转或者组件生成时也会执行，所以需要通过process.server参数判断当前环境。
- 在服务端调用时，即页面刷新时，会等待promise状态完成后才会继续执行接下来的生命周期钩子。

##### fetch

- 每次需要获取异步数据时都可以使用fetch。fetch在服务器端呈现路由时调用，在客户端导航时调用。
- 在服务端调用时，即页面刷新时，会等待promise状态完成后才会继续执行接下来的生命周期钩子。

##### 不同点

你会发现这两者其实很相似，都可以同时在服务端和客户端执行，在服务端执行时也都是用于处理数据，但是他们还是有以下的不同点： 

- asyncData只有在路由级页面能使用，即在注册了路由的组件才能使用，而fetch可以在所有vue组件下使用。
- asyncData中无法使用组件的this，因为组件还未初始化完成，而fetch可以，更便于处理数据。
- asyncData在服务端的执行时间在fetch和created之前，fetch在created之后，在客户端执行时asyncData依旧是排第一位，而fetch在beforeMount之后执行

#### 其他

除了以上讲的基础内容，还有一些其他内容，如自定义模块，路由动画和一些配置相关的就不细讲啦。

### 脚手架搭建

在开始之前，首先需要准备的条件有：

- vue & vue-server-renderer 2.3.0+
- vue-router 2.5.0+
- vue-loader 12.0.0+ & vue-style-loader 3.0.0+
- vuex
- express
- webpack及相关插件

#### 起步

- 我们可以新建一个文件夹如vue-ssr,然后打开终端执行npm init生成package.json文件
- 然后先安装vue与vue-server-renderer和express进行测试（npm install vue vue-server-renderer express --save）
- 新建一个文件夹server.js

```js
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()

server.get('*', (req, res) => {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>访问的 URL 是： {{ url }}</div>`
  })

  renderer.renderToString(app, (err, html) => {
    if (err) {
      res.status(500).end('Internal Server Error')
      return
    }
    res.setHeader("Content-type","text/html;charset=utf8");
    res.end(`
      <!DOCTYPE html>
      <html lang="en">
        <head><title>Hello</title></head>
        <body>${html}</body>
      </html>
    `)
  })
})

server.listen(8080)
```

这段代码的作用其实就是先创建一个vue实例，再创建一个renderer，通过renderer的renderToString方法将vue实例渲染为html，最后通过express创建的服务器，访问url后响应html到浏览器上
最后在页面可以看到：

{% asset_img 图片4.png %}

- 使用一个页面模板

我们现在可以用一个页面模板，然后将vue实例渲染的html写入到页面模板中，不过这个模板需要个HTML标记<!--vue-ssr-outlet-->。这样才能正确的写入到模板中

模板支持一些简单的插值，我们可以通过传入一个"渲染上下文对象"，作为 renderToString 函数的第二个参数，来提供插值数据：

```html
<html>
  <head>
    <!-- 使用双花括号(double-mustache)进行 HTML 转义插值(HTML-escaped interpolation) -->
    <title>{{ title }}</title>

    <!-- 使用三花括号(triple-mustache)进行 HTML 不转义插值(non-HTML-escaped interpolation) -->
    {{{ meta }}}
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

server.js修改为：

```
const Vue = require('vue')
const server = require('express')()
const template = require('fs').readFileSync('./index.template.html', 'utf-8')
const renderer = require('vue-server-renderer').createRenderer({
    template
})

server.get('*', (req, res) => {
    const app = new Vue({
        data: {
            url: req.url
        },
        template: `<div>访问的 URL 是： {{ url }}</div>`
    })
    const context = {
        title: 'vue-ssr',
        meta: `<meta name="keyword" content="vue,ssr">
    <meta name="description" content="vue srr demo">`
    }
    renderer.renderToString(app, context, (err, html) => {
        if (err) {
            console.log('err',err)
            res.status(500).end('Internal Server Error')
            return
        }
        res.setHeader("Content-type", "text/html;charset=utf8");
        res.end(html)
    })
})

server.listen(8080)
```

#### 编写通用代码

因为我们的程序是要同时运行在客户端和服务端上的，所以要做好区分，因为有些全局变量只能在服务端或者客户端使用，如window、document只能在客户端使用。
因此我们可以创建两个不同的js文件，分别处理不同端的情况。
通过webpack打包vue应用程序提供给客户端，由于一些webpack的特定功能不能在服务端运行，如css-loader导入css，所以我们要分开打包，服务端打包生成的bundle用于服务端渲染，客户端bundle用于混合静态标记，也就是激活客户端，之后会讲到，大致的流程如下图：

{% asset_img 图片5.png %}

1. 首先创建通用的app.js文件

在里面导出一个可以重复执行的工厂函数，该函数返回通用的vue实例，为什么要导出一个工厂函数而不是直接导出一个vue实例呢？我们以前在浏览器上使用vue基本都是直接用一个vue实例，但是现在是在服务端运行，每次请求都会获取一次vue，这样vue实例会在各个请求中被共享，容易造成交叉请求污染，所有要导出一个工厂函数，包括之后的vue-router。vuex也是如此。

```js
const Vue = require('vue');
module.exports = function createApp(context) {
    return new Vue({
        data: {
            url: context.url
        },
        template: `<div>访问的URL是：{{ url }}</div>`
    })
}
```

2. 创建客户端入口

新建文件entry.client.js,客户端 entry 只需创建应用程序，并且将其挂载到 DOM 中：

```js
import { createApp } from './app';
const { app } = createApp();
app.$mount('#app')
```

3. 创建服务端入口

新建文件entry-server.js，服务端入口导出一个函数，每次渲染都会重复调用此函数，之后我们将在此做服务端路由匹配和数据预取逻辑。

```
import { createApp } from './app';
export default context => {
	const { app } = createApp()
}
```

#### 路由与代码分割

- 使用 vue-router 的路由

我们之前的服务器代码匹配路由是用一个*号，也就是所有url都会匹配到，现在我们用vue-router处理访问的url，对客户端和服务器复用相同的路由配置！

(1) 创建router.js

新建router.js,并在写入代码，然后更新app.js

```
//router.js
import Vue from 'vue';
import Router from 'vue-router';
Vue.use(Router);
export function createRouter() {
	return new Router({
		mode: 'history',
		routes: []
	})
}
```

```
app.js
import Vue from 'vue'
import { createRouter } from './router'

// 导出一个工厂函数，用于创建新的
export function createApp () {
  const router = createRouter()
  const app = new Vue({
    // 根实例简单的渲染应用程序组件。
    router,
    render: h => h(App),
  })
  return { app, router }
}
```

(2) 处理服务端与客户端入口文件

我们先对entry.server.js文件加入对路由逻辑的处理，目的是根据url匹配路由，因为有可能是异步路由，如路由懒加载，所以需要使用onReady等待异步路由加载完成。

```js
import { createApp } from './app'
export default context => {
    //因为有可能会是异步路由钩子函数或组件，所以我们将返回一个Promise,
    //以便服务器能够等待所有的内容在渲染前，
    //就已经准备就绪。
    return new Promise((resolve, reject) => {
        const { app, router } = createApp()
        //设置服窝器端router的位置
        router.push(context.url)
        //等到router将可能的异步组件和钩子函数解析完
        router.onReady(() => {
        const matchedComponents = router.getMatchedComponents()
        //匹配不到的路由，执行reject函数，并返回404
            if (!matchedComponents.length) return reject({ code: 404 })
            // Promise应该resolve应用程序头例，以便匕可以渲染
            resolve(app)
        }, reject)
    ))
)
```

然后我们对客户端入口文件进行处理

需要注意的是，你仍然需要在挂载 app 之前调用 router.onReady，因为路由器必须要提前解析路由配置中的异步组件，才能正确地调用组件中可能存在的路由钩子。这一步我们已经在我们的服务器入口 (server entry) 中实现过了，现在我们只需要更新客户端入口 (client entry)：

```js
import { createApp } from './app'|
const { app, router } = createApp()
router.onReady(() => {
    app.$mount('#app')
))
```

#### 数据预取存储容器

在服务器端渲染(SSR)期间，所以如果应用程序依赖于一些异步数据，那么在开始渲染过程之前，需要先预取和解析好这些数据。因为客户端和服务端需要获取到完全相同的数据，所以我们可以使用vuex进行数据的存取来保证数据的一致性。

1. 创建store.js文件

创建store.js文件并写入代码，具体的逻辑可以忽略，我们只要知道主要的作用是保证数据的一致性，记得在app.js中也要修改，可以导出store，如下：

```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
//假定我们有一个可以返回Promise的
//通用API (请忽略此API具体实现细节)
import { fetchitem } from './api'
export function createStore () {
    return new Vuex.Store({
        state: {
            items: {}
        },
        actions: {
            fetchitem ({ commit }, id) {
                // ' store.dispatch()' 会返回 Promise,
                //以便我们能够知道数据在何时更新
                return fetchltem(id).then(item => {
                	commit('setitem', { id, item })
                })
            )
         },
         mutations: {
            setitem (state, { id, item }) {
            	Vue.set(state.items, id, item)
            }
         }
    })
)
```

```js
import { createRouter } from './router'
import { createStore } from './store'
import { sync } from 'vuex-router-sync'
export function createApp () {
    //创建router和store实例
    const router = createRouter()
    const store = createStore()
    //同步路由状态(route state)到store
    sync(store, router)
    //创建应用程序实例，将router和store注入
    const app = new Vue({
        router,
        store,
        render: h => h(App)
    })
    // 暴露 app, router 和 storeo
    return { app, router, store }
)
```

2. 界面与数据

现在我们具备渲染页面的条件，但是根据业务需求，我们可以分为以下两种方式：

- 等待数据加载完成再跳转页面

这种方式是当我们切换页面时，会先等待下一个页面的数据加载完成后，才跳转，具体的实现方式是改造客户端入口文件，使用路由钩子函数beforeResolve，获取到即将跳转的路由组件信息，然后等待组件中的asyncData执行完成后再跳转，如下：

```js
router.onReady(() => {
//添加路由钩子函数，用于处理asyncData.
//使用' router.beforeResolve()',以便确保所有异步组件都. resolve。
    router.beforeResolve((to, from, next) => {
        const matched = router.getMatchedComponents(to)
    	//这里如果有加载指示器(loading indicator),就触发
    	Promise.all(matched.map(c => {
            if (c.asyncData) {
            	return c.asyncData({ store, route: to ))
            }
        })).then(() => £
        //停止加载指示器(loading indicator)
        next()
        }).catch(next)
    ))
    app.$mount('#app')
})
```

- 先跳转页面再进行数据处理

这种方式和就比较常见了，和我们平时开发差不多，进入下一个页面后请求相关数据，渲染页面。具体实现是定义一个全局的mixin ，当组件中有asyncData函数时执行，并将其返回的promise赋值，以便之后使用。如下：

```js
Vue.mixin((
    beforeMount () {
        const { asyncData } = this.$options
        if (asyncData) {
            //将获取数据操作分配给promise
            //以便在组件中，我们可以在数据准备就绪后
            //通过运行'this.dataPromise.then(...来执行其他任务
            this.dataPromise = asyncData({
                store: this.$store,
                route: this.$route
        	))
		}
	}
))
```

#### 客户端激活

所谓客户端激活，指的是 Vue 在浏览器端接管由服务端发送的静态 HTML，使其变为由 Vue 管理的动态 DOM 的过程。
由于服务器已经渲染好了 HTML，我们显然无需将其丢弃再重新创建所有的 DOM 元素。相反，我们需要"激活"这些静态的 HTML，然后使他们成为动态的（能够响应后续的数据变化）。

如果你检查服务器渲染的输出结果，你会注意到应用程序的根元素上添加了一个特殊的属性：

```
<div id="app" data-server-rendered="true">
```

data-server-rendered 特殊属性，让客户端 Vue 知道这部分 HTML 是由 Vue 在服务端渲染的，并且应该以激活模式进行挂载。注意，这里并没有添加 id="app"，而是添加 data-server-rendered 属性：你需要自行添加 ID 或其他能够选取到应用程序根元素的选择器，否则应用程序将无法正常激活。
注意，在没有 data-server-rendered 属性的元素上，还可以向 $mount 函数的 hydrating 参数位置传入 true，来强制使用激活模式(hydration)：

```js
app.$mount('#app', true)
```

#### 构建配置

在完成两个入口的配置后，接下来就开始用webpack搭建基础框架了。服务器端渲染 (SSR) 项目的配置大体上与纯客户端项目类似，但是我们建议将配置分为三个文件：base, client 和 server。基本配置 (base config) 包含在两个环境共享的配置，例如，输出路径 (output path)，别名 (alias) 和 loader。服务器配置 (server config) 和客户端配置 (client config)，可以通过使用 webpack-merge 来简单地扩展基本配置。

- webpack.base.config.js文件

这是一个基础的webpack配置文件，服务端和客户端配置文件会通过webpack-merge插件来合并这个基础配置。

具体的配置：

```js
const VueLoaderPlugin = require('vue-loader/lib/plugin')
const path = require('path')
const resolve = file => path.resolve(__dirname, file)
// CSS 提取应该只用于生产环境
// 这样我们在开发过程中仍然可以热重载
const isProduction = process.env.NODE_ENV === 'production'

let config = {
    //启用webpack内置的优化,有'production'和'development'两个选项，将会添加不同的plugin，具体看webpack4官方文档
    mode: isProduction ? 'production' : 'development',
    //选择一种source map格式来增强调试过程。一般生产环境下不使用
    devtool: isProduction ? false : 'cheap-module-source-map',
    //打包后输出相关的配置，只能有一个
    output: {
        path: resolve('../dist'), //生成的目录，对应一个绝对路径。
        publicPath: '/dist/', //应用程序中所有资源的基础路径。
        filename: '[name].[hash].js' //生成文件的名字
    },
    //配置模块如何解析
    resolve: {
        //创建别名
        alias: {
            'public': resolve('../public'),
            'components': resolve('../src/components'),
            'view': resolve('../src/view'),
            'request': resolve('../src/request'),
            'utils': resolve('../src/utils/')
        }
    },
    //模块，处理项目中不同的模块类型
    module: {
        //创建模块时，匹配请求的规则数组
        rules: [{
                test: /\.js$/, //匹配的文件
                exclude: /node_modules/, //排除的目录
                loader: "babel-loader" //使用的loader
            },
            {
                test: /\.vue$/,
                loader: 'vue-loader',
                options: {
                    // hotReload: false // 关闭热重载
                }
            },
            {
                test: /\.(jpe?g|png|gif|svg)$/,
                use: {
                    loader: 'url-loader',
                    options: { //url-loader相关配置
                        outputPath: 'assets/img', //输出图片的地址，相对于output.path
                        publicPath: '../dist/assets/img', //图片的服务地址
                        limit: 3000, //文件大小低于limit时将图片打包在js中，生成base64
                        esModule: false, // 这里设置为false，显示图片地址
                        name: '[name].[ext]?[hash]', //生成图片的名字
                    }
                }
            },
            {
                test: /\.(woff|svg|eot|ttf)$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        outputPath: "assets/iconfont",
                        name: 'iconfont.[name].[ext]?[hash]'
                    }
                }
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin() //使用vue-loader必须添加到plugins里
    ]
}
module.exports = config
```

期间需要安装一些插件，如vue-loader，babel-loader，url-loader来分别处理vue文件，js文件和一些图片及字体图标文件。

- webpack.client.config.js文件

这是一个处理客户端入口的webpack配置文件，在这个文件需要引入一个比较关键的插件vue-server-renderer/client-plugin，其实就是我们之前渲染模板的插件下的一个js文件，用于处理入口文件并生成vue-ssr-client-manifest.json文件，之后会引入到node的server文件中。

具体配置：

```js
const webpack = require('webpack')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const {
	merge
} = require('webpack-merge')
const baseConfig = require('./webpack.base.config.js')
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')
const isProduction = process.env.NODE_ENV === 'production'
let config = merge(baseConfig, {
	entry: {
		app: '/src/entry-client.js'
	},
	optimization: {
		splitChunks: {
			chunks: 'initial',
			minSize: 30000,
			maxSize: 0,
			minChunks: 1,
			maxAsyncRequests: 5,
			maxInitialRequests: 3,
			automaticNameDelimiter: '~',
			name: '',
			 cacheGroups: {
				vendor: {
					test: /[\\/]node_modules[\\/]/,
					chunks: "all",
					priority: 10,
					enforce: true
				},
				commons: {
					chunks: "all",
					minChunks: 2,
					maxInitialRequests: 5,
					// minSize: 0,
					priority: 0
				}
			},
		},
		runtimeChunk: {
			name: "manifest"
		}
	},
	module: {
		rules: [{
			test: /\.(css|less)$/,
			use: [isProduction ? MiniCssExtractPlugin.loader : 'vue-style-loader',
				'css-loader',
				'postcss-loader',
				'less-loader'
			]
		}]
	},
	plugins: [
		new VueSSRClientPlugin(),
		new webpack.DefinePlugin({
			'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
			'process.env.VUE_ENV': "'client'"
		})
	]
})
if (isProduction) {
	config.plugins.push(new MiniCssExtractPlugin({
		filename: 'common.[chunkhash].css'//css文件名字
	}))
}
module.exports = config
```

在这里也会随便做一些打包优化，由于使用的webpack版本是4.x，所以js打包优化使用optimization下的splitChunks配置，3.x之前是使用CommonsChunkPlugin插件进行分包，css文件抽离也从extract-text-webpack-plugin插件换成extract-text-webpack-plugin插件。

- webpack.server.config.js文件

server的配置文件则主要用到vue-server-renderer插件下的sever-plugin文件，用于生成服务端的vue-ssr-server-bundle.json文件，之后也会在node的server文件中使用。
需要注意的点：由于我们使用mini-css-extract-plugin插件进行css文件抽离，但是这个插件在服务端使用会报错，因为使用到了document 等客户端的全局参数，所有我们在处理css文件时需要分成服务端和客户端两个loader处理。还有就是使用vue-style-loader代替style-loader，因为vue-style-loader多了些对服务端渲染的处理：

1. 客户端和服务器端的通用编程体验。
2. 在使用 bundleRenderer 时，自动注入关键 CSS(critical CSS)。
3. 通用 CSS 提取。

```js
const webpack = require('webpack')
const {
    merge
} = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.config.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')

module.exports = merge(baseConfig, {
    entry: '/src/entry-server.js',
    target: 'node',
    devtool: 'source-map',
    output: {
        libraryTarget: 'commonjs2'
    },
    externals: nodeExternals({
        allowlist: [/\.css$/]
    }),
    module: {
        rules: [{
            test: /\.(css|less)$/,
            use: ['vue-style-loader',
                'css-loader',
                'postcss-loader',
                'less-loader'
            ]
        }, ]
    },

    plugins: [
        new VueSSRServerPlugin(),
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
            'process.env.VUE_ENV': "'server'"
        }),
    ]
})
```

#### 配置开发环境

因为现在这个脚手架如果代码有改动，每次都需要重新build一遍，然后再start，开发起来比较麻烦，所有我们要配置下开发环境，使用热重载。

- setup-dev-server.js文件

在build文件夹下新建一个setup-dev-server.js，当做开发环境的配置文件，在这个文件里需要引入服务端和客户端的两个配置文件，然后安装webpack-dev-middleware和webpack-hot-middleware来使用热重载，监听文件变化然后触发热重载。

完整的代码：

```js
const fs = require('fs')
const path = require('path')
const MFS = require('memory-fs')//操作内存的文件系统
const webpack = require('webpack')
const chokidar = require('chokidar')//封装 Node.js 监控文件系统文件变化功能的库
const clientConfig = require('./webpack.client.config')
const serverConfig = require('./webpack.server.config')
const mfs = new MFS()
const writeRouter = require('../fs/index')
const renderRouter = process.env.RENDER_ROUTER === 'true'//是否执行动态路由表
const readFile = (fs, file) => {
  try {
    return fs.readFileSync(path.join(clientConfig.output.path, file), 'utf-8')
  } catch (e) {}
}
let isFirstRender = true //判断是否是第一次渲染，由于chokidar监听add事件第一次会监听所有文件的新增，所有需要排除掉第一次
module.exports = function setupDevServer (app, templatePath, cb) {
  let bundle
  let template
  let clientManifest

  let ready
  const readyPromise = new Promise(r => { ready = r })
  const update = () => {
    if (bundle && clientManifest) {//文件更新后且两个文件存在时resolve，并执行回调
      ready()
      isFirstRender = false
      cb(bundle, {
        template,
        clientManifest
      })
    }
  }

  // read template from disk and watch
  template = fs.readFileSync(templatePath, 'utf-8')
  chokidar.watch(templatePath).on('change', () => {//监听模板变化
    template = fs.readFileSync(templatePath, 'utf-8')
    console.log('index.html template updated.')
    update()
  })
  chokidar.watch(path.resolve(__dirname,'../src/view')).on('add', () => {//监听view文件下的新增
    if(!isFirstRender) {
      renderRouter && writeRouter()//路由写入方法
      update()
    }
  })
  chokidar.watch(path.resolve(__dirname,'../src/view')).on('unlink', () => {//监听view文件下的删除
    if(!isFirstRender) {
      renderRouter && writeRouter()//路由写入方法
      update()
    }
  })
  /**
   * https://github.com/webpack-contrib/webpack-hot-middleware
   * 添加热重载及相关参数 noInfo：不输出相关信息 reload：自动更新
   */
  clientConfig.entry.app = ['webpack-hot-middleware/client?noInfo=true&reload=true', clientConfig.entry.app]
  clientConfig.output.filename = '[name].js'
  clientConfig.plugins.push(
    // new webpack.optimize.OccurrenceOrderPlugin(),//webpack 1.x
    new webpack.HotModuleReplacementPlugin(),//启用热重载
    // new webpack.NoEmitOnErrorsPlugin()//webpack 1.x
  )

  // dev middleware
  const clientCompiler = webpack(clientConfig)
  /**
   * 使用该插件修改文件时无需重新编译，一般配合webpack-hot-middleware实现热重载
   * https://www.npmjs.com/package/webpack-dev-middleware
   */
  const devMiddleware = require('webpack-dev-middleware')(clientCompiler, {
    publicPath: clientConfig.output.publicPath,//绑定中间件的公共路径,与webpack配置的路径相同
    // quiet: true,//向控制台显示任何内容 
    noInfo: true//显示无信息到控制台（仅警告和错误） 
  })

  //使用webpack-dev-middleware中间件
  app.use(devMiddleware)
  clientCompiler.plugin('done', stats => {//在成功构建并且输出了文件后，Webpack 即将退出时发生；
    stats = stats.toJson()
    stats.errors.forEach(err => console.error(err))
    stats.warnings.forEach(err => console.warn(err))
    if (stats.errors.length) return
    clientManifest = JSON.parse(readFile(
      devMiddleware.fileSystem,
      'vue-ssr-client-manifest.json'
    ))
    update()
  })
  
  //使用热重载中间件
  app.use(require('webpack-hot-middleware')(clientCompiler))

  const serverCompiler = webpack(serverConfig)
  serverCompiler.outputFileSystem = mfs //将serverCompiler的文件系统改成内存系统
  serverCompiler.watch({}, (err, stats) => {//当文件发生改变时执行
    if (err) throw err
    stats = stats.toJson()
    if (stats.errors.length) return
    // read bundle generated by vue-ssr-webpack-plugin
    bundle = JSON.parse(readFile(mfs, 'vue-ssr-server-bundle.json'))
    update()
  })

  return readyPromise
}
```

- 修改server文件

我们需要对server文件进行修改，判断正式环境和测试环境，分别执行不同的方法，当前是测试环境时，执行setup-dev-server导出的setupDevServer方法，该方法返回一个promise，在服务端和客户端文件重新构建完成后resolve，之后再触发render方法，执行renderToString方法返回html给客户端，从而实现开发环境热重载。

完整代码：

```js
const express = require('express')
const app = express()
const favicon = require('serve-favicon')//网页图标
const path = require('path')
const writeRouter = require('../fs/index')
/**
 * 服务端渲染的关键
 * https://ssr.vuejs.org/zh/api/
 */
const { createBundleRenderer } = require('vue-server-renderer')

const resolve = file => path.resolve(__dirname, file)
const templatePath = resolve('../src/index.template.html')
const serverInfo =
  `express/${require('express/package.json').version} ` +
  `vue-server-renderer/${require('vue-server-renderer/package.json').version}`
const isProd = process.env.NODE_ENV === 'production'//判断是否是生成环境
const renderRouter = process.env.RENDER_ROUTER === 'true'//是否执行动态路由表

renderRouter && writeRouter()//路由写入方法
//静态目录配置
const serve = (path, cache) => express.static(resolve(path), {
  maxAge: cache && isProd ? 1000 * 60 * 60 * 24 * 30 : 0
})

let readyPromise,renderer
if(isProd) {
  const template = require('fs').readFileSync(templatePath, 'utf-8')
  const serverBundle = require(resolve('../dist/vue-ssr-server-bundle.json'))
  const clientManifest = require(resolve('../dist/vue-ssr-client-manifest.json'))
  /**
   * 第一个参数可以是以下之一：
   * 绝对路径，指向一个已经构建好的 bundle 文件（.js 或 .json）。必须以 / 开头才会被识别为文件路径。
   * webpack + vue-server-renderer/server-plugin 生成的 bundle 对象。
   * JavaScript 代码字符串（不推荐）。
   */
  renderer = createRenderer(serverBundle, {
    runInNewContext: false,//bundle 代码将与服务器进程在同一个 global 上下文中运行，所以请留意在应用程序代码中尽量避免修改 global。
    template,//模板
    clientManifest//由 vue-server-renderer/client-plugin 生成的客户端构建 manifest 对象
  })
} else {
  readyPromise = require(resolve('../build/setup-dev-server'))(
    app,
    templatePath,
    (bundle, options) => {
      renderer = createRenderer(bundle, options)
    }
  )
}
function createRenderer (bundle, options) {
  // https://github.com/vuejs/vue/blob/dev/packages/vue-server-renderer/README.md#why-use-bundlerenderer
  return createBundleRenderer(bundle, options)
}
function render (req, res) {
  const s = Date.now()

  res.setHeader("Content-Type", "text/html")
  res.setHeader("Server", serverInfo)

  const handleError = err => {
    if (err.url) {
      res.redirect(err.url)
    } else if(err.code === 404) {
      res.status(404).send('404 | Page Not Found')
    } else {
      // Render Error Page or Redirect
      res.status(500).send('500 | Internal Server Error')
      console.error(`error during render : ${req.url}`)
      console.error(err.stack)
    }
  }

  const context = {
    title: 'Vue ssr', // default title
    url: req.url
  }
  /**
   * renderToString将context传递给entry-server，
   * vue实例挂载的内容变成html，返回给客户端
   */
  renderer.renderToString(context, (err, html) => {
    if (err) {
      return handleError(err)
    }
    res.send(html)
    if (!isProd) {
      console.log(`whole request: ${Date.now() - s}ms`)
    }
  })
}

app.use('/dist', serve('../dist', true)) //访问/dist时访问dist目录下的文件
app.use(favicon(resolve('../public/img/logo-48.png')))

app.get('*', isProd ? render : (req, res) => {
  readyPromise.then(() => render(req, res))
}
)
app.listen(8080, () => {
  console.log(`server started at localhost: 8080`)
})
```

#### 配置启动参数

在项目根目录package.json中的srcipt参数中增加如下参数：

```js
 "scripts": {
    "dev": "cross-env RENDER_ROUTER=false node server/index",
    "start": "cross-env NODE_ENV=production node server/index",
    "build": "rimraf dist && npm run build:client && npm run build:server",
    "build:renderRouter": "cross-env FIRST_RENDER=true node fs/index && rimraf dist && npm run build:client && npm run build:server",
    "build:client": "cross-env NODE_ENV=production webpack --config build/webpack.client.config.js --progress",
    "build:server": "cross-env NODE_ENV=production webpack --config build/webpack.server.config.js --progress",
    "build:analyzer": "cross-env NODE_ENV=production ANALYZER=true webpack --config build/webpack.client.config.js --progress"
  },
```

- dev

启动开发环境，运行系统

- start

该命令传入一个参数NODE_ENV=production来设置node环境变量，在node进程中的文件使用process.env.production可以访问该参数的值，用于判断当前环境。然后执行node server启动。不过在前提下需要先执行npm run build命令，打包生成服务端和客户端的json文件，才可以启动。

- build

该命令先移除dist文件夹下的所有文件，然后再执行客户端和服务端文件的打包

- 其他

其他命令都是些优化和分析相关

## 总结

该文章介绍了服务端渲染的优势及vue服务端渲染（SSR）的方案，因为我们现在前端开发大多数接触的都是客户端渲染，往往忽略了一些其他的方案，如服务端渲染（SSR）和静态网站渲染（SSG）。通过这次分享可以了解vue服务端渲染的两个基本实现方法，nuxt.js和脚手架搭建，其中nuxt.js功能更加全面，开发成本低，不过可定制性较差，而自己搭建脚手架开发成本会高很多，但是可以根据自己的需求定制。对于已经开发完成的项目推荐在当前开发框架上修改，而对于新开发的项目，使用nuxt.js可能是最好的方式。
