---
title: vite搭建ts版vue3移动端基础框架
cover: /img/thumbnail/学习/前端/vue/vite.png
thumbnail: /img/thumbnail/学习/前端/vue/vite.png
date: 2021-12-17 13:35:43
updated: 2021-12-17 17:40:50
toc: true
categories: 
- 学习
- 前端
tags: 
- vue
- vite
- typeScript
---
## <font color=#a862ea>总览</font>

[demo地址](https://github.com/qianduanzhou/vite-vue)

<!--more-->

{% asset_img vite_vue.png 功能脑图 %}

## <font color=#a862ea>开始</font>

### <font color=#a862ea>初始化</font>

```
npm init vite@latest my-vue-app 初始化一个项目，期间可以选中相应框架及语法
```

### <font color=#a862ea>启动</font>

接着我们打开安装的目录，使用npm install安装相关依赖。之后只用npm run dev即可运行。

{% asset_img 图片1.png %}

## <font color=#a862ea>安装库</font>

安装下面相关的库，注意版本问题

- vue-router（安装vue-router4.x版本进行路由控制）
- vuex（安装vuex4.x版本进行状态管理）
- element-plus（安装element-plus ui库）
- axios（安装axios http库方便网络请求）
- sass、sass-loader（安装sass和sass-loader支持sass语法）
- esri-loader（安装esri-loader便于引入arcgis）
- postcss-pxtorem（安装postcss-pxtorem转换px为rem，适配移动端）
- normalize.css（引入normalize.css初始化css）

## <font color=#a862ea>配置</font>

在vite.config.ts和tsconfig.json里进行基础路径,别名及跨域等配置。

```typescript
//vite.config.ts
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
const { resolve } = require('path')
function _resolve(path: string): string {
 return resolve(__dirname, path)
}
// https://vitejs.dev/config/
export default defineConfig(({mode}) => {
 const env = loadEnv(mode, __dirname)
 return {
  base: './',
  plugins: [
   vue(), 
   vueJsx(),
   Components({
    resolvers: [ElementPlusResolver()],
   }),
  ],
  resolve: {
   // 配置别名
   alias: {
    '@': _resolve('./src'),
   },
  },
  server: {
   proxy: {
    '/local': {
     target: env.VITE_NORMALURL,
     rewrite: path => path.replace(/^\/local/, ''),
    changeOrigin: true,
    }
   }
  }
 }
})
```

```json
//tsconfig.json
{
  "compilerOptions": {
    "target": "esnext",
    "useDefineForClassFields": true,
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "lib": ["esnext", "dom", "DOM.Iterable"],
    "baseUrl": "./",
    "paths":{
      "@": ["src"],
      "@/*": ["src/*"],
    },
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}

```



## <font color=#a862ea>工具函数与类</font>

### <font color=#a862ea>缓存桶</font>

提高移动端下拉列表的用户体验。将数据先缓存起来，下拉到底的时候先拿缓存的数据，期间判断并请求接口，用户几乎感知不到卡顿。

```typescript
import {toRefs} from 'vue';
import { request, DataIn } from "@/request";
import { deepClone } from "@/utils"

interface Data extends DataIn {
    limit: number
}
interface RequestOption {
    name: string,
    data: Data,
    recordKey ?: string
}



//缓存桶类
class CacheBucket {
    constructor() {}
    public cacheList: Array<any> = [];//缓存的数据列表
    public copyRes: any = null;//缓存的返回结果
    public defalutCount: number = 20;//默认一次返回数量
    public requestOption: RequestOption = {name: '', data: {limit: 0}};//请求和返回相关参数
    //初始化
    public init(requestOption: RequestOption, defalutCount: number) {
        let { limit } = requestOption.data
        if(defalutCount > limit) {
            throw new SyntaxError(`defalutCount不能大于limit`);
        }
        this.defalutCount = defalutCount !== undefined ? defalutCount : requestOption ? limit / 2 : this.defalutCount
        this.requestOption = requestOption || {}
    }

    //获取列表
    public getList() {
        return new Promise((resolve: any, reject: any) => {
            if(this.cacheList.length > this.defalutCount) {
                this.requestList(resolve, reject, false, false)
            } else if(this.cacheList.length === this.defalutCount) {
                this.requestList(resolve, reject, false)
            } else {
                this.requestList(resolve, reject)
            }
        })
    }
    
    /**
     * 请求列表
     * @param resolve 
     * @param reject 
     * @param isHandle 是否使用handleResult方法
     * @param isRequest 是否请求接口
     */
    private requestList(resolve: any, reject: any, isHandle: boolean = true, isRequest: boolean = true) {
        let {name, data, recordKey} = this.requestOption

        if(!isHandle) {
            if(recordKey) {
                this.copyRes[recordKey] = this.cacheList.splice(0, this.defalutCount)
            } else {
                this.copyRes = this.cacheList.splice(0, this.defalutCount)
            }
            let result = {
                res: this.copyRes,
                isRequest: isRequest ? true : false
            }
            resolve(result)
        }
        if(isRequest) {
            request({ name, data }).then(
                (res: any) => {
                    if(isHandle) {
                        this.handleResult(resolve, res);
                    } else {
                        let data = recordKey ? res[recordKey] : res
                        this.cacheList = this.cacheList.concat(data)
                    }
                }).catch((err: any) => {
                    reject(err)
            })
        }
    }

    //处理请求结果
    private handleResult(resolve: any, res: any) {
        this.copyRes = deepClone(res)
        let { recordKey } = this.requestOption
        if(recordKey) {
            this.copyRes[recordKey] = this.handleRecord(res[recordKey])
        } else {
            this.copyRes = this.handleRecord(res)
        }
        let result = {
            res: this.copyRes,
            isRequest: true
        }
        resolve(result)
    }
    /**
     * 处理列表
     * @param data 列表
     * @returns 
     */
    private handleRecord(data: any) {
        let record: Array<any> = [];
        this.cacheList = this.cacheList.concat(data)
        record = this.cacheList.splice(0, this.defalutCount)
        return record
    }
    /**
     * 获取缓存数据
     */
    public getCacheList() {
        return this.cacheList
    }
}

export default CacheBucket
```

### <font color=#a862ea>图片懒加载</font>

图片出现在视图内才进行请求，节约资源。

```typescript
//图片懒加载
class ImgLazyLoad {
    constructor() {}

    public imgList: Array<any> = [];//图片列表
    public length: number = 0;//图片总数
    public init() {
        this.imgList = [...document.querySelectorAll('img')];
        this.length = this.imgList.length;
        document.addEventListener('scroll', this.imgLazyLoad)
    }
    private imgLazyLoad = (() => {
        let count = 0
        return () => {
            let deleteIndexList: Array<number> = []
            this.imgList.forEach((img: any, index: number) => {
                let rect = img.getBounding前端Rect()
                if (rect.top < window.innerHeight) {//滚动到视图内触发
                    img.src = img.dataset.src
                    deleteIndexList.push(index)
                    count++
                    if (count === length) {
                        document.removeEventListener('scroll', this.imgLazyLoad)
                    }
                }
            })
            this.imgList = this.imgList.filter((img: any, index: number) => !deleteIndexList.includes(index))//删除已加载的图片
        }
    })()
}
export default ImgLazyLoad
```

## <font color=#a862ea>构建和部署</font>

### <font color=#a862ea>构建</font>

使用npm run build和npm run build-test分别构建测试和正式包。

### <font color=#a862ea>部署</font>

编写Dockerfile，使用docker部署：

```dockerfile
FROM nginx
MAINTAINER zhou 
COPY dist/  /usr/share/nginx/html/ 
```

## <font color=#a862ea>总结</font>

vite搭建的vue3版本的h5移动端框架，使用了vue全家桶及element ui库，并配置了一些实用的工具函数，可以完成项目开发的基本需求。
