---
layout: post
title: Vue全家桶搭档Typescript
subtitle: Typescript && Vue
date: 2018-06-26
author: Rick
catalog: true
header-img: img/vue-typescript.png
tags:
  -Vue Vuex Vue-Router Typescript
---

### Typescript是什么
Typescript是微软基于javascript基础之上的一门语言，其本身是javascript的超集，以兼容最新的ECMAScript标准为目的，并在基础之上集成强类型、接口、泛型等js本身不具备的强大特性！用好typescript能使我们的程序更为健壮、安全，提升开发效率。

### 开发环境搭建
本来想使用vue-cli3.0beta直接一键生成vue-typescript模板的，但是折腾半天以后发现还是问题多多，还是等正式版本出来以后再使用这个命令吧！退而求其次，我们先使用vue-cli命令生成一个vue全家桶项目的模板, 执行以下命令，然后一路回车确认即可
```zsh
vue init webpack vue-ts-demo 
```
第二步 添加typescript相关的包:
```zsh
npm install --save-dev typescript ts-loader tslint tslint-config-standard tslint-loader
```
第三步 在根目录下创建tsconfig.json的配置文件, 配置如下
```json
{
  "include": [
    "src/*",
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ],
   "compilerOptions": {
    "types": [
      "node"
    ],
    "typeRoots": [
      "node_modules/@types"
    ],
    "noImplicitThis": true,
    "jsx": "preserve",
    "jsxFactory": "h",
    "allowJs": true,
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "module": "esnext",
    "target": "es5",
    "moduleResolution": "node",
    "noImplicitAny": true,
    "lib": [
      "dom",
      "es2015",
      "es2017",
      "es2015.promise"
    ],
    "sourceMap": true,
    "pretty": true
  }
}
```
具体配置项的含义以及更多的配置项查看typescript官方文档<https://www.typescriptlang.org/>

第四步 在根目录下新建一个tslint.json 的配置文件，配置内容如下
```json
{
  "extends": "tslint-config-standard",
  "globals": {
    "require": true
  }
}
```
第五步 修改webpack的配置项，使之能编译ts，在build目录的webpack.base.conf.js配置文件下添加如下配置
```js
{
  test: /\.vue$/,
  loader: 'vue-loader',
  options: Object.assign(vueLoaderConfig, {
    loaders: {
      ts: 'ts-loader',
      tsx: 'babel-loader!ts-loader'
    }
  })
},
{
  test: /\.tsx?$/,
  exclude: /node_modules/,
  enforce: 'pre',
  loader: 'tslint-loader'
},
{
  test : /\.tsx?$/,
  use: [
    'babel-loader',
    {
      loader: 'ts-loader',
      options: { appendTsxSuffixTo: [/\.vue$/]}
    }
  ],
  exclude : /node_modules/
},
```
同时入口文件改成main.ts, resolve配置项的extension改成如下
```js
entry: {
  app: './src/main.ts'
},
extensions: ['.ts', '.js', '.vue', '.json']
```

重新启动服务，这时候就能愉快的使用tpyescipt开发Vue应用了

### 常见的坑有哪些
1. 找不到 *.vue 模块, 由于typescript自带的类型推断，目前typescript还是无法识别 *.vue 的单文件组件，
解决办法就是在src目录下新建一个vue-shims.d.ts, 内容如下
```ts
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

2. import 组件时不支持@ alias的写法，必须使用相对路径，同时必须把后缀名加上, 如下
```ts
import HelloWorld from './components/Helloworld.vue'
```
而下面这种写法是不被ts识别的
```ts
import HelloWorld from '@/components/Helloworld'
```

3. 无法推断 Vue 实例内部属性类型
在js开发时, this.$router 这样的代码在 ts 下会报错说属性不存在，解决的办法时是引入新的包 vue-class-component,
这是Vue官方推出的配合Typescript而写的npm包，定义组件时使用下面这种写法

```ts
import Vue from 'vue'
import { Component } from 'vue-class-component'

@Component({})
export default class HelloWorld extends Vue {}
```

4. *.vue 文件中的script 标签必须加上 lang="ts" 属性, 否则 webpack会把脚本默认当成 js

5. 在 template 调用自定义的函数类型时，typescript 无法检测到传入的参数类型是否符合要求, 解决办法是把vue
组件使用tsx的方法写，但是这种写法对于习惯Vue常规写法的人来说变化太大，而且Vue自身内置的一些v-for、v-model指
令无法在tsx中识别

6. 某些第三方库无法被typescript识别到，有些npm包默认都是基于javascript编写的，所以在ts中引用时会因为无法识别
而报错，解决办法是在src目录下创建一个typing目录，里面存放vue-shims.d.ts和global.d.ts两个文件，global.d.ts
文件如下

```ts
declare module 'element-ui'
declare module 'vue-lazyload'
declare module 'vue-ueditor'
```

### 为什么使用Typescript
在前后端分离的趋势下，后端一般只负责提供接口数据给前端，由前端自行决定将数据怎么渲染出来。这个过程中，前后台的通信都是通过各种各样的数据
和状态进行的，假如前端需要的是一个数字类型，后台却返回了一个字符串类型，这种因为数据类型不一致而遗留下的定时炸弹，往往不知道什么时候就暴
露出来。在测试阶段去找到这些问题的难度系数和花费的时间远远比在开发阶段就避开这些问题难得多。而Typescript自带的强类型可以帮助我们在开发
阶段去避免这些问题的困扰。有关Typescript的一些入门知识我会单独写一篇文章介绍，这里就不细说了。

### Typescript如何写Vue组件
typescript的写法和js的写法区别比较大, 话不多说上代码

```ts
import { Vue, Component } from 'vue-property-decorator'

@Component({})
export default class Order extends Vue {
  created () {
  }
  
  id: number = 0
  params = {
    page: 10,
    currentPage: 1
  }

  getId (num: number):number {
    return this.id
  }
}
```
首先ts的写法是声明一个class, 而不是js那样声明一个object, 其次这个类必须继承Vue这个基类，同时使用@Component({})这个装饰器去修饰这个类，
@Component()接受一个对象作为参数，对象的属性是Vue组件的选项，比如props, mixins这些, 而Vue内置的一些钩子函数和自定义的普通函数都是作为
类的普通方法，属性的声明也不需要放到data选项中，直接声明为一个类的实例属性即可。属性声明时可以同时声明该属性的类型，也可以不声明，由ts自行
推断。更加完整的用例和写法请直接看源码<https://github.com/rickydan/vue-ts-demo>