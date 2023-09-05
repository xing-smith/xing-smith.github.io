---
layout:     post
title:      深入浅出webapck第一章 & webapck 文档
subtitle:   chaper_01: 入门
date:       2023-08-30
author:     forwardZ
header-img: img/the-first.png
catalog: false
tags:
    - 浏览器_工程化_Git
---

<!-- [webpack-chap01.png](https://postimg.cc/QFcgdnTM) -->


## chapter01:入门


### 一，前端的发展

#### 一）模块化
模块化是指将 一个复杂的系统分解为多个模块以方便编码 。

命名空间的缺陷：

1. 命名空间冲突；
2. 无法合理地管理项目的依赖和版本；
3. 无法方便地控制依赖的加载顺序；

模块化思想的实践

1. CommonJS：核心思想是通过 require 方法来同步加载依赖的其他模块，通过 module.exports 导出需要暴露的接口。
 * 优点：代码可复用于 Node.js 环境下井运行；通过Npm发布的很多第三方模块都采用了 CommonJS规范
 * 缺点：无法直接运行在浏览器环境下，必须通过工具转换成标准的 ES5
 * 补充：CommonJS 还可以细分为 CommonJSl 和 CommonJS，区别在于 CommonJSl 只能通过 exports . XX = XX 的方式导出，而 CommonJS2 在 CommonJSl 的基础上加入了 module. exports = XX 的导出方式。 CommonJS 通常指 CommonJS2
 
2. AMD：采用了异步的方式去加载依赖的模块，主要用于解决针对浏览器环境的模块化问题，最具代表性的实现是 requirejs。
 * 优点：直接在浏览器中运行；可异步加载依赖；可并行加载多个依赖；代码可运行在浏览器环境和 Node.js 环境下；
 * 缺点：JavaScript运行环境没有原生支持 A岛。， 需要先导入实现了 AMD 的库 后才能正常使用

3. ES6模块化：它将逐渐取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案 
 * 缺点：目前无法直接运行在大部分 JavaScript运行环境下，必须通过工具转换成标准的 ES5 后才能 正常运行

4. 样式文件申的模块化：
 * 定义：@mixin
 * 引入：@import
 * 使用：@include
 
#### 二）新框架
在 Web 应用变得庞大、复杂时，采用直接操作 DOM 的方式去开发会使代码变得复杂和难以维护。

* React：
* Vue：
* Angular2

#### 三）新语言
1. ES6：新增包括规范模块化；Class 语法；箭头函数；async 函数；set和Map数据结构等，但不同的浏览器对这些特性的支持不一致，需要将 ES6代码转换成 ES5代码。Babel（https://babeljs.io)是目前 解决这个问题的优秀工具。
2. Typescript：除支持 ES6 的所有功能，还提供了静态类型检查。其缺点在于语法相对于 JavaScript更啰嗦，并且无法在浏览器或 Node.js环境下直接运行。
3. Flow：为 JavaScript 提供静态类型检查，和 Typescript相似但更灵活，可以只在需要的地方加上类型检查 。
4. scss：一种 css 预处理器，可以方便地管理代码，抽离公共的部分，通过逻辑写出更灵活的代码。类似的预处理还有less

### 二，常见的构建工具及对比
前面提及的新框架新语言的一个共同特点是：源代码无法直接运行，必须通过转换后 才可 以正常运行。
构建就是将源代码转化为可以执行的JavaScript、 css、 HTML 代码，具体包括：

1. 代码转换:将 TypeScript编译成 JavaScript、将 scss 编译成 css 等 
2. 文件优化:压缩 JavaScript、 css、 HT鸟也代码，压缩合并图片等。
3. 代码分割:提取多个页面的公共代码，提取首屏不需要执行部分的代码让其异步加载 。
4. 模块合并：在采用模块化的项目中会有很多的模块和文件，需要通过构建功能将模块分类合并成一个文件。
5. 自动刷新:监昕本地源代码的变化，自动重新构建、刷新浏览器
6. 代码校验:在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过 。
7. 自动发布:更新代码后，自动构建出线上发布代码井传输给发布系统。

构建其实是工程化、自动化思想在前端开发中的体现，将一系列流程用 代码去实现，让 代码自动化地执行这一系列复杂的流程。
历史出现的构建工具：

1. Npm Script
2. Grunt
3. Gulp
4. Fis3
5. Webpack

#### webpack
Webpack 是一个打包模块化 JavaScript 的工具，在 Webpack 里一切文件皆模块，通过 Loader转换文件，通过 Plugin注入钩子，最后输出由多个模块组合成的文件。 Webpack 专注于构建模块化项目。

优点：

* 专注于处理模块化的项目，能做到开箱即用、 一步到位:
* 可通过 Plugin 扩展，完整好用又不失灵活
* 使用场景不局限于 Web 开发
* 社区庞大活跃 ， 经常引入紧跟时代发展的新特性，能为大多数场景找到已有的开源扩展
* 良好的开发体验

缺点：只能用于采用模块化开发的项目

### 三，安装 webpack
新建一个web项目，实践记录

```
npm init
// 只会生成一个 json 文件
```

#### 一）安装webpack到本项目

```
npm i D webpack
```
注：这里缺一个 webpack cli，后续运行命令时，需按提示增加
#### 二）安装webpack到全局

#### 三）使用webpack
案例一：通过webpack构建一个采用了 CommonJS 模块化编写的项目，该项目中的某个网页会通过 JavaScript显示 Hello,Webpack。

* 在项目目录下新增main.js，index.html，show.js，webpack.config.js，
* 终端运行 ```  node_modules/.bin/webpack ```，如果没有安装webpack cli，会提示安装cli，安装后继续执行命令
* 将 index.html 文件使用浏览器打开，会显示 ```hello，webpack```

<!-- [webpack-01-case-01.png](https://postimg.cc/sBv0Hv9t) -->

### 四，使用loader
Webpack 不原生支持解析 css 文件。 要支持非JavaScript类型的文件，则需要使用 Webpack的Loader机制。
案例二：使用 Webpack构建了一个采用 CommonJS规范的模块化项目，为项目引入 css 代码以让文字居中显示。

 * 新增main.css文件，并在 main.js 中引入
 * webpack.config.js 新增 module 配置
 * 安装loader，```  npm i -D style-loader css-loader ```
 * 终端运行 webpack 命令
 
 注：webpack 有些版本可能不支持 minimize：true，去掉即可构建成功。
 
 <!-- [webpack-01-case-02.png](https://postimg.cc/Z9MmRWLc) -->
 
### 五，使用plugin
Plugin是用来扩展 Webpack功能的，通过在构建流程里注入钩子实现，它为 Webpack带来了很大的灵活性。

案例三：提取出 JavaScript 代码里的 CSS 到一个单独的文件，插件 extract-text-webpack-plugin 应该是与 webpack 存在兼容问题，换成mini-css-extract-plugin 插件了。

 * 安装插件plugin ``` ```
 * 调整 index.html 文件 和 webpack 配置文件，
 * 终端运行 webpack 命令

<!-- [webpack-01-case-03.png](https://postimg.cc/R6z8L0t7) -->

### 六，使用 DevServer
webpack原生支持 

1. 监听文件的变化并自动刷新网页，做到实时预览
2. 支持 Source Map，以方便调试

借助 DevServer，可以通过 HTTP 服务进行预览。DevServer 会启动一个 HTTP 服务器用于服务网页请求，同时会帮助启动 Webpack ，并接收 Webpack 发出的文件更变信号，通过 WebSocket 协议自动刷新网页做到实时预览。

案例四：实现本地服务，实时预览，以及模块热替换和source-map

 * 安装 DevServer，``` npm i -D webpack-dev-server ```
 * 增加配置，
 
 ```js
module.exports = {
	// ...
	/* 开发模式，防止报错 */
     mode: 'development',
     /*  */
    devServer:{
    	  /* 指向项目，默认路径为项目中的index.html，没有该配置，报错：Cannot GET */
        static:'../DEMO_WEBPACK',
        /* 开启模块热替换，即无需刷新，实现实时预览 */
        hot:true
    },
    /* 开启源代码映射 */
    devtool: 'source-map'
}
 ```
 * 终端运行 ``` node_modules/.bin/webpack-dev-server ```

<!-- [webapck-01-case-04.png](https://postimg.cc/XXfLgjVL) -->

### 七，核心概念
Webpack 有以下几个核心概念，

1. Entry：入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入；
2. Module：模块，在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块；
3. Chunk：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割；
4. Loader：模块转换器，用于把模块原内容按照需求转换成新内容；
5. Plugin：扩展插件，在 Webpack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情；
6. Output：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果

Webpack 启动后会从 Entry 里配置的 Module 开始递归解析 Entry 依赖的所有 Module。 每找到一个 Module， 就会根据配置的 Loader 去找出对应的转换规则，对 Module 进行转换后，再解析出当前 Module 依赖的 Module。 

这些模块会以 Entry 为单位进行分组，一个 Entry 和其所有依赖的 Module 被分到一个组也就是一个 Chunk。

最后 Webpack 会把所有 Chunk 转换成文件输出。在整个流程中 Webpack 会在恰当的时机执行 Plugin 里定义的逻辑。





 
 

 
 
 











