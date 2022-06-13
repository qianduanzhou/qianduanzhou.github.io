---
title: qiankun的基本使用
date: 2022-06-07 10:20:44
cover: /img/thumbnail/学习/前端/微前端/qiankun/qiankun.png
thumbnail: /img/thumbnail/学习/前端/微前端/qiankun/qiankun.png
tags:
- 微前端
- qiankun
- vite
- vue
- react
- typeScript
toc: true
---

qiankun 是一个基于 [single-spa](https://github.com/CanopyTax/single-spa) 的[微前端](https://micro-frontends.org/)实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。

详细请看[官网](https://qiankun.umijs.org/zh/guide)。

<!--more-->

## 特性

- 📦 **基于 [single-spa](https://github.com/CanopyTax/single-spa)** 封装，提供了更加开箱即用的 API。
- 📱 **技术栈无关**，任意技术栈的应用均可 使用/接入，不论是 React/Vue/Angular/JQuery 还是其他等框架。
- 💪 **HTML Entry 接入方式**，让你接入微应用像使用 iframe 一样简单。
- 🛡 **样式隔离**，确保微应用之间样式互相不干扰。
- 🧳 **JS 沙箱**，确保微应用之间 全局变量/事件 不冲突。
- ⚡️ **资源预加载**，在浏览器空闲时间预加载未打开的微应用资源，加速微应用打开速度。
- 🔌 **umi 插件**，提供了 [@umijs/plugin-qiankun](https://github.com/umijs/plugins/tree/master/packages/plugin-qiankun) 供 umi 应用一键切换成微前端架构系统。

## 使用

qiankun应用分成主应用和微应用，通过主应用可以控制微应用及主微应用之间的数据传递。

本项目主应用是vite搭建的vue3，微应用是react和vue2。

### 安装

```shell
yarn add qiankun # 或者 npm i qiankun -S
```

### 在主应用中注册微应用

1. 在main.ts中注册微应用

```typescript
import { registerMicroApps, start } from 'qiankun';
registerMicroApps([
  {
    name: 'reactApp', // app name registered
    entry: '//localhost:4000',
    container: '#reactApp',
    activeRule: '/reactApp',
  },
  {
    name: 'vueApp',
    entry: '//localhost:8080',
    container: '#vueApp',
    activeRule: '/vueApp',
  },
]);
```

2. 在路由文件router/index.ts中注册路由，可以通过路由控制不同微应用的显示。记得微应用相关的路由都要注册，不过可以指向同一个组件。

```typescript
import { createRouter, createWebHistory } from "vue-router";

const routes = [
	//vue微应用
	{
		path: '/vueApp',
		name: 'vueApp-home',
		component: () => import("@/view/vueApp/index.vue")
	},
	{
		path: '/vueApp/about',
		name: 'vueApp-about',
		component: () => import("@/view/vueApp/index.vue")
	},
	//react微应用
	{
		path: '/reactApp',
		name: 'reactApp-home',
		component: () => import("@/view/reactApp/index.vue")
	},
	{
		path: '/reactApp/list',
		name: 'reactApp-list',
		component: () => import("@/view/reactApp/index.vue")
	},
	{
		path: '/reactApp/collection',
		name: 'reactApp-collection',
		component: () => import("@/view/reactApp/index.vue")
	},
	{
		path: '/reactApp/detail/:id',
		name: 'reactApp-detail',
		component: () => import("@/view/reactApp/index.vue")
	},
]

export default createRouter({
	history: createWebHistory(),
	routes
});
```

3. 新建两个组件分别代表不同微应用。记得id要对应container。然后再其中调用start方法可以启动微应用。

以下是react的微应用vue文件，vue的也是类似。

```vue
<template>
  <div id="reactApp"></div>
</template>

<script lang="ts">
import { onMounted } from "vue";
import { registerMicroApps, start } from "qiankun";
let qiankunStarted = false;
export default {
  setup() {
    onMounted(() => {
      if (!qiankunStarted) {
        qiankunStarted = true;
        start();
      }
    });
  }
};
</script>

<style lang='scss' scoped>
#vueApp {
  width: 100%;
  height: 100%;
}
</style>
```

这样一来主应用的配置就基本完成了。

### react微应用

可以通过[官网](https://qiankun.umijs.org/zh/guide/tutorial#react-%E5%BE%AE%E5%BA%94%E7%94%A8)的方式创建（推荐）。

以下方式和官网有一点不同，主要就是webpack配置，由于本项目直接通过npm run eject暴露了webpack相关配置，所以直接在其中进行修改。

1. 在 `src` 目录新增 `public-path.js`：

   ```js
   if (window.__POWERED_BY_QIANKUN__) {
     __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
   }
   ```

2. 在src/index.tsx中修改。

```tsx
import './public-path';
import { createRoot } from 'react-dom/client';
import { Provider } from 'react-redux';
import configureStore from './store';
import './index.scss';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { BrowserRouter } from 'react-router-dom';
function render(props: any) {
	const { container } = props;
	const rootDom = document.getElementById('root');
	const root = createRoot(container ? container.querySelector('#root') : rootDom);
	root.render(
		<BrowserRouter basename={window.__POWERED_BY_QIANKUN__ ? '/reactApp' : '/'}>
			<Provider store={configureStore}>
				<App />
			</Provider>
		</BrowserRouter>
	);

	// If you want to start measuring performance in your app, pass a function
	// to log results (for example: reportWebVitals(console.log))
	// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
	reportWebVitals();
}

if (!window.__POWERED_BY_QIANKUN__) {
	render({});
}

export async function bootstrap() {
	console.log('[react16] react app bootstraped');
}

export async function mount(props: any) {
	console.log('[react16] props from main framework', props);
	render(props);
}

export async function unmount(props: any) {
	const { container } = props;
	ReactDOM.unmountComponentAtNode(container ? container.querySelector('#root') : document.querySelector('#root'));
}
```

3. 在config/webpack.config.js中修改

```js
const {
  name
} = path.resolve(__dirname, '/package');
module.exports = function (webpackEnv) {
    return {
        //...
        output: {
            //...
            library: `${name}-[name]`,
            libraryTarget: 'umd',
            chunkLoadingGlobal: `webpackJsonp_${name}`,
            globalObject: 'window',
            //...
        },
        //...
    }
}
```

4. 在config/webpackDevServer.config.js中修改

```js
module.exports = function (proxy, allowedHost) {
	return {
        //...
        	hot: false,
        	historyApiFallback: true,
    		liveReload: false,
        	static: {
            	watch: false  
            }
        //...
    }
}
```

基本配置差不多了，其他就是一些兼容，如\__POWERED_BY_QIANKUN__变量需要在src/react-app-env.d.ts中声明。还有就是一些路径的适配，如public下的文件由于没有通过webpack打包，所以在代码中使用的时候需要判断路径。

### vue微应用

vue微应用和react也是类似，修改相关配置，暴露出生命周期钩子。

详细可以看[官网](https://qiankun.umijs.org/zh/guide/tutorial#vue-%E5%BE%AE%E5%BA%94%E7%94%A8)。

## 最终效果

{% asset_img vue.png %}

{% asset_img react.png %}

## 总结

以上就是qiankun的基本使用，当然qiankun还有更多用法，比如不通过路由控制（registerMicroApps）微应用，直接手动方式控制（loadMicroApp）等等。
