---
layout:     post
title:      深入浅出 webapck & webapck 文档
subtitle:   chaper_03: 实战
date:       2023-09-01
author:     forwardZ
header-img: img/the-first.png
catalog: false
tags:
    - 浏览器_工程化_Git
---

[webpack-chap01.png](https://postimg.cc/QFcgdnTM)




## chapter_03：实战

使用webpack 解决如下场景的问题

* 使用新语言开发项目：ES6，TypeScript，Flow，Scss，PostCSS
* 使用新框架开发项目：react、vue、Angular2
* 用 Webpack 构建单页应用
* 用 Webpack 构建不同运行环境的项目：同构应用、Electron应用、Npm模块、离线应用
* Webpack 结合其它工具搭配使用：npm script、检查代码、node.js、webpack-dev-middleware 
* 用 Webpack 加载特殊类型的资源：图片、svg、Source Map



### 一，新语言

#### 一）使用 ES6 语言

把采用 ES6 编写的代码转换成目前已经支持良好的 ES5 代码，通常包含2件事：

1. 把新的 ES6 语法用 ES5 实现，例如 ES6 的 `class` 语法用 ES5 的 `prototype` 实现。

2. 给新的 API 注入 polyfill ，例如项目使用 `fetch` API 时，只有注入对应的 polyfill 后，才能在低版本浏览器中正常运行

babel可以方便的完成上面两件事。

##### 认识Babel

Babel 是一个 JavaScript 编译器，能将 ES6 代码转为 ES5 代码，让你使用最新的语言特性而不用担心兼容性问题，并且可以通过插件机制根据需求灵活的扩展。在 Babel 执行编译的过程中，会从项目根目录下的 `.babelrc` 文件读取配置，`.babelrc` 是一个 JSON 格式的文件，大致如下：

```json
{
  "plugins": [
    [
      "transform-runtime",
      {
        "polyfill": false
      }
    ]
   ],
  "presets": [
    [
      "es2015",
      {
        "modules": false
      }
    ],
    "stage-2",
    "react"
  ]
}
```

1. plugins

`plugins` 属性告诉 Babel 要使用哪些插件，插件可以控制如何转换代码。

以上配置文件里的 `transform-runtime` 对应的插件全名叫做 `babel-plugin-transform-runtime`，即在前面加上了 `babel-plugin-`，要让 Babel 正常运行我们必须先安装它。

```js
npm i -D babel-plugin-transform-runtime
```

`babel-plugin-transform-runtime` 是 Babel 官方提供的一个插件，作用是减少冗余代码。具体实现是通过注入一条如下导入语句，替换辅助函数。这样能减小 Babel 编译出来的代码的文件大小。

```js
var _extent = require('babel-runtime/helpers/_extent');
```

注意由于 `babel-plugin-transform-runtime` 注入了 `require('babel-runtime/helpers/_extent')` 语句到编译后的代码里，需要安装 `babel-runtime` 依赖到你的项目后，代码才能正常运行。也就是说， `babel-plugin-transform-runtime` 和 `babel-runtime` 需要配套使用，使用了 `babel-plugin-transform-runtime` 后一定需要 `babel-runtime`。

2. Presets

```json
{
  "presets": [
    [
      "es2015",
      {
        "modules": false
      }
    ],
    "stage-2",
    "react"
  ]
}
```

`presets` 属性告诉 Babel 要转换的源码使用了哪些新的语法特性，一个 Presets 对一组新语法特性提供支持，多个 Presets 可以叠加。

Presets 其实是一组 Plugins 的集合，每一个 Plugin 完成一个新语法的转换工作。

Presets 是按照 ECMAScript 草案来组织的，通常可以分为以下三大类：

* 已经被写入 ECMAScript 标准里的特性，可细分为：es2015、es2016、es2017、env（包含当前所有 ECMAScript 标准里的最新特性）；
* 被社区提出来的但还未被写入 ECMAScript 标准里特性，具体分为：stage0、stage1、stage2、stage3、stage4
* 支持一些特定应用场景下的语法，和 ECMAScript 标准没有关系，如`babel-preset-react` 是为了支持 React 开发中的 JSX 语法

##### 接入Babel

由于 Babel 所做的事情是转换代码，所以应该通过 Loader 去接入 Babel，Webpack 配置如下：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'],
      },
    ]
  },
  // 输出 source-map 方便直接调试 ES6 源码
  devtool: 'source-map'
};
```

配置命中了项目目录下所有的 JavaScript 文件，通过 `babel-loader` 去调用 Babel 完成转换工作。 在重新执行构建前，需要先安装新引入的依赖：

```js
# Webpack 接入 Babel 必须依赖的模块
npm i -D babel-core babel-loader
# 根据你的需求选择不同的 Plugins 或 Presets
npm i -D babel-preset-env
```

接入Babel总结：

1. 安装依赖：必须依赖（babel-core，babel-loader）、特定的plugins或者 presets
2. 配置.babelrc
3. webpack配置babel-loader
4. 验证：采用es6语法导入导出验证



#### 二）使用 TypeScript 语言

##### 认识 TypeScript

[TypeScript](http://www.typescriptlang.org/) 是 JavaScript 的一个超集，主要提供了类型检查系统和对 ES6 语法的支持，但不支持新的 API。 目前没有任何环境支持运行原生的 TypeScript 代码，必须通过构建把它转换成 JavaScript 代码后才能运行。

TypeScript 官方提供了能把 TypeScript 转换成 JavaScript 的编译器。 需要在当前项目根目录下新建一个用于配置编译选项的 `tsconfig.json` 文件，编译器默认会读取和使用这个文件，配置文件内容大致如：

```json
{
  "compilerOptions": {
    "module": "commonjs", // 编译出的代码采用的模块规范
    "target": "es5", // 编译出的代码采用 ES 的哪个版本
    "sourceMap": true // 输出 Source Map 方便调试
  },
  "exclude": [ // 不编译这些目录里的文件
    "node_modules"
  ]
}
```



##### 减少代码冗余

TypeScript 编译器会有和babel一样的问题：在把 ES6 语法转换成 ES5 语法时需要注入辅助函数， 为了不让同样的辅助函数重复的出现在多个文件中，可以开启 TypeScript 编译器的 `importHelpers` 选项，修改 `tsconfig.json` 文件如下：

```json
{
  "compilerOptions": {
    "importHelpers": true
  }
}
```

该选项的原理和 Babel 中介绍的 `babel-plugin-transform-runtime` 非常类似，会把辅助函数换成如下导入语句：

```js
var _tslib = require('tslib');
_tslib._extend(target);
```

这会导致编译出的代码依赖 `tslib` 这个迷你库，但避免了代码冗余。

##### 集成webapck

要让 Webpack 支持 TypeScript，需要解决2个问题：

* 通过 Loader 把 TypeScript 转换成 JavaScript：社区有可用的 Loader，这里以 [awesome-typescript-loader](https://github.com/s-panferov/awesome-typescript-loader)为例。 
* Webpack 在寻找模块对应的文件时需要尝试 `ts` 后缀：修改默认的 `resolve.extensions` 配置项

综上，相关 Webpack 配置如下

```js
const path = require('path');

module.exports = {
  // 执行入口文件
  entry: './main',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, './dist'),
  },
  resolve: {
    // 先尝试 ts 后缀的 TypeScript 源码文件
    extensions: ['.ts', '.js']
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'awesome-typescript-loader'
      }
    ]
  },
  devtool: 'source-map',// 输出 Source Map 方便在浏览器里调试 TypeScript 代码
};
```

在运行构建前需要安装上面用到的依赖：

```bash
npm i -D typescript awesome-typescript-loader
```

安装成功后重新执行构建，你将会在 `dist` 目录看到输出的 JavaScript 文件 `bundle.js`，和对应的 Source Map 文件 `bundle.js.map`（ps：个人实践时没有看到map文件，配置项sourceMap已经设置为true）。 在浏览器里打开 `index.html` 页面后，来开发工具里可以看到和调试用 TypeScript 编写的源码。

接入 TypeScript 总结：

1. 安装依赖，包括ts编译器和loader
2. 增加ts 配置项 tsconfig
3. 调整 webpack 配置项：增加loader，resolve的extenstions增加ts后缀补充
4. 将调整 show 为 ts 语法进行打包验证



#### 三）使用 Flow 检查器

##### 认识Flow

[Flow](https://flow.org/) 是一个 Facebook 开源的 JavaScript 静态类型检测器，它是 JavaScript 语言的超集。 你所需要做的就是在需要的地方加上类型检查，例如在两个由不同人开发的模块对接的接口出加上静态类型检查，能在编译阶段就指出部分模块使用不当的问题。 同时 Flow 也能通过类型推断检查出 JavaScript 代码中潜在的 Bug。

##### 使用Flow

 Flow 检测器由高性能跨平台的 [OCaml](http://ocaml.org/) 语言编写，它的可执行文件可以通过

```bash
npm i -D flow-bin
```

安装，安装完成后通过先配置 Npm Script

```json
"scripts": {
   "flow": "flow"
}
```

再通过 `npm run flow` 去调用 Flow 执行代码检查。

安装成功后，在项目根目录下执行 Flow 后，Flow 会遍历出所有需要检查的文件并对其进行检查，输出错误结果到控制台。

采用了 Flow 静态类型语法的 JavaScript 是无法直接在目前已有的 JavaScript 引擎中运行的，要让代码可以运行需要把这些静态类型语法去掉。 例如：

```js
// 采用 Flow 的源代码
function foo(one: any, two: number, three?): string {}

// 去掉静态类型语法后输出代码
function foo(one, two, three) {}
```

有两种方式可以做到这点：

1. [flow-remove-types](https://github.com/flowtype/flow-remove-types) 可单独使用，速度快。
2. [babel-preset-flow](https://babeljs.io/docs/plugins/preset-flow/) 与 Babel 集成。

##### 集成webpack

由于使用了 Flow 项目一般都会使用 ES6 语法，所以把 Flow 集成到使用 Webpack 构建的项目里最方便的方法是借助 Babel。

具体的步骤为：

1. 安装依赖，`npm i -D babel-preset-flow`
2. 修改`.babelrc`配置文件，加入 Flow Preset，

```json
"presets": [
...[],
"flow"
]
```

3. 修改show.js文件，进行验证

注：打包过程可能会报错，UnhandledPromiseRejectionWarning: Error: Plugin/Preset files are not allowed to export objects, only functions. 这是由于babel的兼容问题，具体参考：https://stackoverflow.com/questions/49182862/preset-files-are-not-allowed-to-export-objects。

实际解决操作：

```bash
 npm uninstall babel-loader
 npm i -D babel-loader@^7
```

#### 四）使用 SCSS 语言

##### 认识scss

是一种 CSS 预处理器，语法和 CSS 相似，但加入了变量、逻辑、等编程元素。

SCSS 又叫 SASS，区别在于 SASS 语法类似 Ruby，而 SCSS 语法类似 CSS，对于熟悉 CSS 的前端工程师来说会更喜欢 SCSS。

采用 SCSS 去写 CSS 的好处在于可以方便地管理代码，抽离公共的部分，通过逻辑写出更灵活的代码。 和 SCSS 类似的 CSS 预处理器还有 [LESS](http://lesscss.org/) 等。

使用 SCSS 可以提升编码效率，但是必须把 SCSS 源代码编译成可以直接在浏览器环境下运行的 CSS 代码。 SCSS 官方提供了多种语言实现的编译器，这里介绍偏前端技术栈的node-sass。

node-sass 核心模块是由 C++ 编写，再用 Node.js 封装了一层，以供给其它 Node.js 调用。 node-sass 还支持通过命令行调用，先安装，

```bash
npm i -g node-sass
```

再执行编译命令：

```bash
# 把 main.scss 源文件编译成 main.css
node-sass main.scss main.css
```

注：本次学习暂不做全局安装

##### 接入 Webpack

由于需要把 SCSS 源代码转换成 CSS 代码，最适合的方式是使用 Loader，Webpack 官方提供了对应的 [sass-loader](https://github.com/webpack-contrib/sass-loader)。具体配置如下：

```js
module.exports = {
  module: {
    rules: [
      {
        // 增加对 SCSS 文件的支持
        test: /\.scss$/,
        // SCSS 文件的处理顺序为先 sass-loader 再 css-loader 再 style-loader
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
    ]
  },
};
```

以上配置通过正则 `/\.scss$/` 匹配所有以 `.scss` 为后缀的 SCSS 文件，再分别使用3个 Loader 去处理。具体处理流程如下：

1. 通过 sass-loader 把 SCSS 源码转换为 CSS 代码，再把 CSS 代码交给 css-loader 去处理。
2. css-loader 会找出 CSS 代码中的 `@import` 和 `url()` 这样的导入语句，告诉 Webpack 依赖这些资源。同时还支持 CSS Modules、压缩 CSS 等功能。处理完后再把结果交给 style-loader 去处理。
3. style-loader 会把 CSS 代码转换成字符串后，注入到 JavaScript 代码中去，通过 JavaScript 去给 DOM 增加样式。如果你想把 CSS 代码提取到一个单独的文件而不是和 JavaScript 混在一起，可以使用之前介绍过的 ExtractTextPlugin/mini-css-extract-plugin。

由于接入 sass-loader，项目需要安装这些新的依赖：

```bash
# 安装 Webpack Loader 依赖
npm i -D  sass-loader css-loader style-loader
# sass-loader 依赖 node-sass
npm i -D node-sass
```

具体步骤：

1. 安装依赖，包括`sass-loader css-loader style-loader node-sass `

2. 修改webpack配置项
3. 修改main.css文件为main.scss及具体样式，验证



#### 五）使用PostCSS

##### 认识PostCss

[PostCSS](http://postcss.org/) 是一个 CSS 处理工具，和 SCSS 不同的地方在于它通过插件机制可以灵活的扩展其支持的特性，而不是像 SCSS 那样语法是固定的。PostCSS 的用处非常多，包括给 CSS 自动加前缀、使用下一代 CSS 语法等

* PostCSS 和 CSS 的关系就像 Babel 和 JavaScript 的关系，它们解除了语法上的禁锢，通过插件机制来扩展语言本身，用工程化手段给语言带来了更多的可能性。
* PostCSS 和 SCSS 的关系就像 Babel 和 TypeScript 的关系，PostCSS 更加灵活、可扩张性强，而 SCSS 内置了大量功能而不能扩展。

具体的例子：

* 给 CSS 自动加前缀，增加各浏览器的兼容性：

```css
/*输入*/
h1 {
  display: flex;
}
/*输出*/
h1 {
  display: -webkit-box;
  display: -webkit-flex;
  display: -ms-flexbox;
  display: flex;
}
```

* 使用下一代 CSS 语法：

```css
/*输入*/
:root {
  --red: #d33;
}
h1 {
  color: var(--red);
}
/*输出*/
h1 {
  color: #d33;
}
```

PostCSS 全部采用 JavaScript 编写，运行在 Node.js 之上，即提供了给 JavaScript 代码调用的模块，也提供了可执行的文件。 在 PostCSS 启动时，会从目录下的 `postcss.config.js` 文件中读取所需配置，所以需要新建该文件，文件内容大致如下：

```json
module.exports = {
  plugins: [
    // 需要使用的插件列表
    require('postcss-cssnext')
  ]
}
```

其中的 [postcss-cssnext](http://cssnext.io/) 插件可以让你使用下一代 CSS 语法编写代码，再通过 PostCSS 转换成目前的浏览器可识别的 CSS，并且该插件还包含给 CSS 自动加前缀的功能。(注：目前 Chrome 等现代浏览器已经能完全支持 cssnext 中的所有语法，也就是说按照 cssnext 语法写的 CSS 在不经过转换的情况下也能在浏览器中直接运行。)

##### 接入webapck

虽然使用 PostCSS 后文件后缀还是 `.css` 但这些文件必须先交给 [postcss-loader](https://github.com/postcss/postcss-loader) 处理一遍后再交给 css-loader。

接入 PostCSS 相关的 Webpack 配置如下：

```json
module.exports = {
  module: {
    rules: [
      {
        // 使用 PostCSS 处理 CSS 文件
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader'],
      },
    ]
  },
};
```

接入 PostCSS 给项目带来了新的依赖需要安装，如下：

```bash
# 安装 Webpack Loader 依赖
npm i -D postcss-loader css-loader style-loader
# 根据你使用的特性安装对应的 PostCSS 插件依赖
npm i -D postcss-cssnext
```

具体步骤：

1. 安装依赖，`postcss-loader postcss-cssnext `
2. 修改webpack 配置文件，增加postcss.config.js配置文件
3.  修改main.css文件，验证

### 二，新框架

#### 一）react

##### React 语法特征

使用了 React 项目的代码特征有 JSX 和 Class 语法。其中 JSX 语法是无法在任何现有的 JavaScript 引擎中运行的，所以在构建过程中需要把源码转换成可以运行的代码，例如：

```js
// 原 JSX 语法代码
return <h1>Hello,Webpack</h1>
// 被转换成正常的 JavaScript 代码
return React.createElement('h1', null, 'Hello,Webpack')
```

目前 Babel 和 TypeScript 都提供了对 React 语法的支持，下面分别来介绍如何在使用 Babel 或 TypeScript 的项目中接入 React 框架。

##### React 与 Babel

要在使用 Babel 的项目中接入 React 框架是很简单的，只需要加入 React 所依赖的 Presets [babel-preset-react](https://babeljs.io/docs/plugins/preset-react/)。具体步骤如下：

1. 安装依赖

```bash
# 安装 React 基础依赖
npm i -D react react-dom
# 安装 babel 完成语法转换所需依赖
npm i -D babel-preset-react
```

2. 安装新的依赖后，再修改 `.babelrc` 配置文件加入 React Presets

```json
"presets": [
    "react"
],
```

3. 修改main.js进行验证

```js
import React from 'react'
import { createRoot } from 'react-dom/client'
function Btn() {
    return (
        <div>Hello,react in webpack</div>
    )
}
const app = createRoot(document.getElementById('app'))
app.render(<Btn />)
```

##### react 与 TypeScript

TypeScript 相比于 Babel 的优点在于它原生支持 JSX 语法，你不需要重新安装新的依赖，只需修改一行配置。 但 TypeScript 的不同在于：

* 使用了 JSX 语法的文件后缀必须是 `tsx`
* 由于 React 不是采用 TypeScript 编写的，需要安装 `react` 和 `react-dom` 对应的 TypeScript 接口描述模块 `@types/react` 和 `@types/react-dom` 后才能通过编译。`npm i react react-dom @types/react @types/react-dom`

修改 TypeScript 编译器配置文件 `tsconfig.json` 增加对 JSX 语法的支持，如下：

```js
{
  "compilerOptions": {
    "jsx": "react" // 开启 jsx ，支持 React
  }
}
```

修改main.js的后缀为tax，并修改文件内容为jsx语法。

同时为了让 Webpack 对项目里的 ts 与 tsx 原文件都采用 `awesome-typescript-loader` 去转换， 需要注意的是 Webpack Loader 配置的 `test` 选项需要匹配到 tsx 类型的文件，并且 `extensions` 中也要加上 `.tsx`，配置如下：

```js
const path = require('path');

module.exports = {
  // TS 执行入口文件
  entry: './main',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, './dist'),
  },
  resolve: {
    // 先尝试 ts，tsx 后缀的 TypeScript 源码文件
    extensions: ['.ts', '.tsx', '.js',]
  },
  module: {
    rules: [
      {
        // 同时匹配 ts，tsx 后缀的 TypeScript 源码文件
        test: /\.tsx?$/,
        loader: 'awesome-typescript-loader'
      }
    ]
  },
  devtool: 'source-map',// 输出 Source Map 方便在浏览器里调试 TypeScript 代码
};
```

PS：个人理解，tsx编译器将react、react-dom编译为tsx，awesome-typescript-loader再将tsx编译为js。

疑问：

* 一般项目是怎么接入react（参考 react-great-app 脚手架）
* 一般项目是怎么同时接入 babel、tsx、react

### 三，用 Webpack 构建单页应用

#### 一）为单页应用生成 HTML

##### 引入问题

实际项目比较负责，一个页面常常有很多资源要加载，比如有些代码需要内嵌入HEAD 标签，有些代码需要异步加载以提升首屏加载速度等，如果还是手写`index.html` 文件去完成以上要求，这就会使工作变得复杂、易错，项目难以维护。

##### 解决方案

使用Webpack 插件 [web-webpack-plugin](https://github.com/gwuhaolin/web-webpack-plugin)，核心Webpack 配置：

```js
const path = require('path');
// ...
const { WebPlugin } = require('web-webpack-plugin');

module.exports = {
  // ...
  plugins: [
    // 使用本文的主角 WebPlugin，一个 WebPlugin 对应一个 HTML 文件
    new WebPlugin({
      template: './template.html', // HTML 模版文件所在的文件路径
      filename: 'index.html' // 输出的 HTML 的文件名称
    }),
    // ...
  ],
};
```

其中，`template: './template.html'` 所指的模版文件 `template.html` 的内容是：

```html
<html>
<head>
  <meta charset="UTF-8">
  <!--注入 Chunk app 中的 CSS-->
  <link rel="stylesheet" href="app?_inline">
  <!--注入 google_analytics 中的 JavaScript 代码-->
  <script src="./google_analytic.js?_inline"></script>
  <!--异步加载 Disqus 评论-->
  <script src="https://dive-into-webpack.disqus.com/embed.js" async></script>
</head>
<body>
<div id="app"></div>
<!--导入 Chunk app 中的 JS-->
<script src="app"></script>
<!--Disqus 评论容器-->
<div id="disqus_thread"></div>
</body>
</html>
```

该文件描述了哪些资源需要被以何种方式加入到输出的 HTML 文件中。资源链接 URL 字符串里问号前面的部分表示资源内容来自哪里，后面的 querystring 表示这些资源注入的方式。

以 `<link rel="stylesheet" href="app?_inline">` 为例，按照正常引入 CSS 文件一样的语法来引入 Webpack 生产的代码。 `href` 属性中的 `app?_inline` 可以分为两部分，前面的 `app` 表示 CSS 代码来自名叫 `app` 的 Chunk 中，后面的 `_inline` 表示这些代码需要被内嵌到这个标签所在的位置。

除了 `_inline` 表示内嵌外，还支持`_dist`、`_dev`、`_ie`，这些属性之间可以搭配使用，互不冲突。如 `app?_inline&_dist` 的写法。

问题记录：

1. UglifyJsPlugin插件：需安装依赖，调整引入路径，同时修改插件的入参
2. WebPlugin插件，版本兼容的问题，参照该库最新的提交进行修改
3. 构建的js，css文件的名称为main，而不是app，模版html需调整
4. template.html里面的google_analytic文件名称写错了，保证html中的文件名与项目中的该文件名一致

步骤总结：

1. 安装依赖，`web-webpack-plugin`、`uglifyjs-webpack-plugin`
2. 准备template.html文件，及其中对应的资源
3. 修改webpack配置，
4. 打包、验证、输出

涉及的插件：

| 插件名               | 对应的库                    | 是否需单独安装 | 用途                   | 替换                  | 备注 |
| -------------------- | --------------------------- | -------------- | ---------------------- | --------------------- | ---- |
| UglifyJsPlugin       | uglifyjs-webpack-plugin     | 需要           | 压缩输出的js           | terser-webpack-plugin |      |
| DefinePlugin         | webpack/lib/DefinePlugin    | 否             | 定义 NODE_ENV 环境变量 |                       |      |
| WebPlugin            | web-webpack-plugin          | 需要           | 生成html               |                       |      |
| MiniCssExtractPlugin | mini-css-extract-plugin     | 需要           | CSS提取                |                       |      |
| ExtractTextPlugin    | extract-text-webpack-plugin | 需要           | 文件的提取             |                       |      |

* `const UglifyJsPlugin = require('uglifyjs-webpack-plugin')`
* `const DefinePlugin = require('webpack/lib/DefinePlugin')`
* `const DefinePlugin = require('webpack/lib/DefinePlugin')`
* `const MiniCssExtractPlugin = require('mini-css-extract-plugin')`
* `const ExtractTextPlugin = require('extract-text-webpack-plugin')`



#### 二）管理多个单页应用

##### 引入问题

只生成了一个 HTML 文件，但在实际应用中一个完整的系统不会把所有的功能都做到一个网页中，因为这会导致这个网页性能不佳。 实际的做法是按照功能模块划分成多个单页应用，每个单页应用生成一个 HTML 文件。并且随着业务的发展更多的单页应用可能会逐渐被加入到项目中去。考虑如下例子：

* 项目目前共有2个单页应用组成，一个是主页 `index.html`，一个是用户登入页 `login.html`
* 多个单页应用之间会有公共的代码部分，需要把这些公共的部分抽离出来，放到单独的文件中去以防止重复加载。例如多个页面都使用一套 CSS 样式，都采用了 React 框架，这些公共的部分需要抽离到单独的文件中
* 随着业务的发展后面可能会不断的加入新的单页应用，但是每次新加入单页应用不能去改动构建相关的代码

疑问：使用 WebPlugin 如何实现 构建，下面是通过 AutoWebPlugin 实现的

##### 解决方案

通过AutoWebPlugin，实现多个单页面构建。目录结构要求如下：

* 所有单页应用的代码都需要放到一个目录下，例如都放在 pages 目录下
* 一个单页应用一个单独的文件夹，例如最后生成的 `index.html` 相关的代码都在 `index` 目录下，`login.html` 同理
* 每个单页应用的目录下都有一个 `index.js` 文件作为入口执行文件

webpack的配置如下：

```js
const { AutoWebPlugin } = require('web-webpack-plugin');

// 使用本文的主角 AutoWebPlugin，自动寻找 pages 目录下的所有目录，把每一个目录看成一个单页应用
const autoWebPlugin = new AutoWebPlugin('pages', {
  template: './template.html', // HTML 模版文件所在的文件路径
  postEntrys: ['./common.css'],// 所有页面都依赖这份通用的 CSS 样式文件
  // 提取出所有页面公共的代码
  commonsChunk: {
    name: 'common',// 提取出公共代码 Chunk 的名称
  },
});

module.exports = {
  // AutoWebPlugin 会为寻找到的所有单页应用，生成对应的入口配置，
  // autoWebPlugin.entry 方法可以获取到所有由 autoWebPlugin 生成的入口配置
  entry: autoWebPlugin.entry({
    // 这里可以加入你额外需要的 Chunk 入口
  }),
  plugins: [
    autoWebPlugin,
  ],
};
```

`template.html` 模版文件如下：

```html
<html>
<head>
  <meta charset="UTF-8">
  <!--在这注入该页面所依赖但没有手动导入的 CSS-->
  <!--STYLE-->
  <!--注入 google_analytics 中的 JS 代码-->
  <script src="./google_analytics.js?_inline"></script>
  <!--异步加载 Disqus 评论-->
  <script src="https://dive-into-webpack.disqus.com/embed.js" async></script>
</head>
<body>
<div id="app"></div>
<!--在这注入该页面所依赖但没有手动导入的 JavaScript-->
<!--SCRIPT-->
<!--Disqus 评论容器-->
<div id="disqus_thread"></div>
</body>
</html>
```

由于这个模版文件被当作项目中所有单页应用的模版，就不能再像上一节中直接写 Chunk 的名称去引入资源，因为需要被注入到当前页面的 Chunk 名称是不定的，每个单页应用都会有自己的名称。 `<!--STYLE-->` 和 `<!--SCRIPT-->` 的作用在于保证该页面所依赖的资源都会被注入到生成的 HTML 模版里去。

由于模版文件 `template.html` 里没有指出引入这些依赖资源的 HTML 语句，插件会自动将没有手动导入但页面依赖的资源按照不同类型注入到 `<!--STYLE-->` 和 `<!--SCRIPT-->` 所在的位置。

* CSS 类型的文件注入到 `<!--STYLE-->` 所在的位置，如果 `<!--STYLE-->` 不存在就注入到 HTML HEAD 标签的最后；
* JavaScrip 类型的文件注入到 `<!--SCRIPT-->` 所在的位置，如果 `<!--SCRIPT-->` 不存在就注入到 HTML BODY 标签的最后。

如果后续有新的页面需要开发，只需要在 `pages` 目录下新建一个目录，目录名称取为输出 HTML 文件的名称，目录下放这个页面相关的代码即可，无需改动构建代码。

##### 具体的实现

wepack的依赖：

```bash
npm i -D webpack
npm i -D webpack-cli
```

babel的依赖

```bash
// 核心依赖
npm i -D babel-core babel-loader
// Plugins 或 Presets 对应的自定义依赖
npm i -D babel-preset-env
npm i -D babel-preset-react
```

react框架的依赖

```bash
npm i -D react
npm i -D react-dom
```

css相关loader

```bash
npm i -D css-loader
```

插件：

```bash
npm i -D mini-css-extract-plugin
npm i -D uglifyjs-webpack-plugin
npm i -D web-webpack-plugin // 报错参考之前的解决方法
```

有一个报错（Error: Plugin/Preset files are not allowed to export objects, only functions. ），是babe-core与babel-loader版本兼容问题，解决方法：

```bash
npm i uninstall babel-loader
npm i -D babel-loader@^7
```

修改配置，运行webpack。







### 四，用 Webpack 构建不同运行环境的项目

#### 一）同构应用

同构应用是指写一份代码但可同时在浏览器和服务器中运行的应用。

##### 认识同构应用

现在大多数单页应用的视图都是通过 JavaScript 代码在浏览器端渲染出来的，但在浏览器端渲染的坏处有：

* 搜索引擎无法收录你的网页，因为展示出的数据都是在浏览器端异步渲染出来的，大部分爬虫无法获取到这些数据
* 对于复杂的单页应用，渲染过程计算量大，对低端移动设备来说可能会有性能问题，用户能明显感知到首屏的渲染延迟

为了解决以上问题，有人提出能否将原本只运行在浏览器中的 JavaScript 渲染代码也在服务器端运行，在服务器端渲染出带内容的 HTML 后再返回。这样就能让搜索引擎爬虫直接抓取到带数据的 HTML，同时也能降低首屏渲染时间。 

实际上现在主流的前端框架都支持同构，其中最先支持也是最成熟的同构方案是 React。

同构应用运行原理的核心在于虚拟 DOM，虚拟 DOM 的意思是不直接操作 DOM 而是通过 JavaScript Object 去描述原本的 DOM 结构。 在需要更新 DOM 时不直接操作 DOM 树，而是通过更新 JavaScript Object 后再映射成 DOM 操作。

虚拟 DOM 的优点在于：

* 因为操作 DOM 树是高耗时的操作，尽量减少 DOM 树操作能优化网页性能。而 DOM Diff 算法能找出2个不同 Object 的最小差异，得出最小 DOM 操作
* 虚拟 DOM 的在渲染的时候不仅仅可以通过操作 DOM 树来表示出结果，也能有其它的表示方式，例如把虚拟 DOM 渲染成字符串(服务器端渲染)，或者渲染成手机 App 原生的 UI 组件( React Native)。

以 React 为例，核心模块 `react` 负责管理 React 组件的生命周期，而具体的渲染工作则由 `react-dom` 模块来负责。`react-dom` 在渲染虚拟 DOM 树时有2中方式可选：

* 通过 `render()` 函数去操作浏览器 DOM 树来展示出结果
* 通过 `renderToString()` 计算出表示虚拟 DOM 的 HTML 形式的字符串

构建同构应用的最终目的是从一份项目源码中构建出2份 JavaScript 代码，一份用于在浏览器端运行，一份用于在 Node.js 环境中运行渲染出 HTML。 其中用于在 Node.js 环境中运行的 JavaScript 代码需要注意以下几点：

1. 不能包含浏览器环境提供的 API，例如使用 `document` 进行 DOM 操作，因为 Node.js 不支持这些 API；
2. 不能包含 CSS 代码，因为服务端渲染的目的是渲染出 HTML 内容，渲染出 CSS 代码会增加额外的计算量，影响服务端渲染性能；
3. 不能像用于浏览器环境的输出代码那样把 `node_modules` 里的第三方模块和 Node.js 原生模块(例如 `fs` 模块)打包进去，而是需要通过 CommonJS 规范去引入这些模块；
4. 需要通过 CommonJS 规范导出一个渲染函数，以用于在 HTTP 服务器中去执行这个渲染函数，渲染出 HTML 内容返回。

##### 解决方案

由于要从一份源码构建出2份不同的代码，需要有2份 Webpack 配置文件分别与之对应。 用于构建浏览器环境代码的 `webpack.config.js` 配置文件保留不变，新建一个专门用于构建服务端渲染代码的配置文件 `webpack_server.config.js`，其中关键地方为：

* `target: 'node'` 由于输出代码的运行环境是 Node.js，源码中依赖的 Node.js 原生模块没必要打包进去；
* `externals: [nodeExternals()]` [webpack-node-externals](https://github.com/liady/webpack-node-externals) 的目的是为了防止 node_modules 目录下的第三方模块被打包进去，因为 Node.js 默认会去 node_modules 目录下寻找和使用第三方模块；
* `{test: /\.css$/, use: ['ignore-loader']}` 忽略掉依赖的 CSS 文件，CSS 会影响服务端渲染性能，又是做服务端渲不重要的部分；
* `libraryTarget: 'commonjs2'` 以 CommonJS2 规范导出渲染函数，以供给采用 Node.js 编写的 HTTP 服务器代码调用。（这里做补充，TODO）

##### 具体操作

1. 安装依赖：express、react、react-dom 按项目文件执行版本安装；其他在保证babel-loader兼容的情况下可以不指定版本安装
2. wepback配置：两份配置，
3. 执行打包：
4. 启动服务：

##### 针对`http_server.js`补充：

1. 去掉render()，由于下面导入浏览器端的构建文件，没有影响
2. 去掉`<div id="app"></div>`结构，由于构建文件是基于id=app的div进行挂载的，所有没有展示了
3. 单纯去掉浏览器端的构建js，由于服务端的构建文件去掉了样式，所有只能看到黑色的h1标签

```json
const express = require('express');
const { render } = require('./dist/bundle_server');
const app = express();

// 调用构建出的 bundle_server.js 中暴露出的渲染函数，再拼接下 HTML 模版，形成完整的 HTML 文件
app.get('/', function (req, res) {
  res.send(`
<html>
<head>
  <meta charset="UTF-8">
</head>
<body>
<div id="app">${render()}</div>
<!--导入 Webpack 输出的用于浏览器端渲染的 JS 文件-->
<script src="./dist/bundle_browser.js"></script>
</body>
</html>
  `);
});

// 其它请求路径返回对应的本地文件
app.use(express.static('.'));

app.listen(3000, function () {
  console.log('app listening on port 3000!')
});
```

##### 针对script的补充

```js
  "scripts": {
    "build:browser": "webpack",
    "build:server": "webpack --config webpack_server.config.js",
    "http_server": "node ./http_server.js"
  },
```



#### 二）构建 Electron 应用

##### 认识Electron

[Electron](https://electron.atom.io/) 可以让你使用开发 Web 的技术去开发跨平台的桌面端应用，由 Github 主导和开源， Atom 和 VSCode 编辑器就是使用 Electron 开发的。

Electron 是 Node.js 和 Chromium 浏览器的结合体，用 Chromium 浏览器显示出的 Web 页面作为应用的 GUI，通过 Node.js 去和操作系统交互。当你在 Electron 应用中的一个窗口操作时，实际上是在操作一个网页。当你的操作需要通过操作系统去完成时，网页会通过 Node.js 去和操作系统交互。

采用这种方式开发桌面端应用的优点有：

* （开发门槛低）降低开发门槛，只需掌握网页开发技术和 Node.js 即可，大量的 Web 开发技术和现成库可以复用于 Electron；
* （跨平台）由于 Chromium 浏览器和 Node.js 都是跨平台的，Electron 能做到写一份代码在不同的操作系统运行

在运行 Electron 应用时，会从启动一个主进程开始。主进程的启动是通过 Node.js 去执行一个入口 JavaScript 文件实现的。主进程启动后会一直驻留在后台运行，眼睛所看得的和操作的窗口并不是主进程，而是由主进程新启动的窗口子进程。应用从启动到退出有一系列生命周期事件，通过 `electron.app.on()` 函数去监听生命周期事件，在特定的时刻做出反应。启动的窗口其实是一个网页，启动时会去加载在 `loadURL` 中传入的网页地址。 每个窗口都是一个单独的网页进程，窗口之间的通信需要借助主进程传递消息。

```js
const { app, BrowserWindow } = require('electron')
/* 保持一个对于 window 对象的全局引用，如果你不这样做，当 JavaScript 对象被垃圾回收，window 会被自动地关闭 */
let win
/* 打开主窗口 */
function createWindow() {
  win = new BrowserWindow({ width: 800, height: 600 })// 创建浏览器窗口
  const indexPageURL = `file://${__dirname}/dist/index.html`;
  win.loadURL(indexPageURL);// 加载应用的 index.html
  win.on('closed', () => { // 当 window 被关闭，这个事件会被触发
    // 取消引用 window 对象
    win = null
  })
}
/* Electron 会在创建浏览器窗口时调用这个函数。 */
app.on('ready', createWindow)
app.on('window-all-closed', () => { // 当全部窗口关闭时退出
  if (process.platform !== 'darwin') { // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，否则绝大部分应用会保持激活
    app.quit()
  }
})
```

总结：

开发 Electron 应用和开发 Web 应用很相似，区别在于 Electron 的运行环境同时内置了浏览器和 Node.js 的 API，在开发网页时除了可以使用浏览器提供的 API 外，还可以使用 Node.js 提供的 API。

##### 接入webpack

以如下简单的Electron应用为例，要求为应用启动后显示一个主窗口，在主窗口里有一个按钮，点击这个按钮后新显示一个窗口，且使用 React 开发网页。由于 Electron 应用中的每一个窗口对应一个网页，所以需要开发2个网页，分别是主窗口的 `index.html` 和新打开的窗口 `login.html`。 也就是说项目由2个单页应用组成，考虑将3-10转化为 Electron 应用。

页面改动的地方包括：

1. 在项目根目录下新建主进程的入口文件 `main.js`，如上面所示；
2. 主窗口网页index/index.js新增点击事件，事件通过 `electron` 库里提供的 API 去新打开一个窗口，并加载网页文件所在的地址。

构建方面需要做到：

1. 构建出2个可在浏览器里运行的网页，分别对应2个窗口的界面；
2. 由于在网页的 JavaScript 代码里可能会有调用 Node.js 原生模块或者 `electron` 模块，也就是输出的代码依赖这些模块。但由于这些模块都是内置支持的，构建出的代码不能把这些模块打包进去。

因为 Webpack 内置了对 Electron 的支持。 只需要在3-10的基础上，给 Webpack 配置文件加上一行代码即可，如下：

```js
target: 'electron-renderer',
```

该行代码可以让 Webpack 构建出用于 Electron 渲染进程用的 JavaScript 代码，也就是这2个窗口需要的网页代码。

##### 整体梳理

1. 通过构建完成网页资源
2. 通过main.js作为入口，执行electron应用

##### 注意点：

1. electron的执行命令为：`electron .`，不要遗漏点号，点号指的是为package.json中的main，所以package.json中的main需与项目实际的electron执行文件名一致；

2. electron的版本最好与书中项目保持一致，后续electron版本内容有所调整

3. alert也可以用于日志打印，尤其是目前这种暂时不知道怎么调试的

4. require无法在浏览器环境运行，webpack打包的target:electron-renderer，有解决这方面的作用。但是高版本会报错：Uncaught ReferenceError: require is not defined。初步解决方法是增加webPreferences属性：

   ```js
   function createWindow() {
       /* 创建浏览器窗口 */
       win = new BrowserWindow({
           width: 800,
           height: 600,
           webPreferences: {
               // preload: path.join(__dirname, 'preload.js')
               nodeIntegration: true,
               enableRemoteModule: true,
               contextIsolation: false,
           }
       })
       //...
   }
   ```

   但渲染页面还会报remote为undefined的错误

   ###### 相关参考：

   require is not defined electron webpack：https://juejin.cn/s/require%20is%20not%20defined%20electron%20webpack

   electron应用的html文件中无法使用require：https://blog.csdn.net/ThisEqualThis/article/details/127843668

   

#### 三）构建 Npm 模块

##### 认识 Npm

[Npm](https://www.npmjs.com/) 是目前最大的 JavaScript 模块仓库，里面有来自全世界开发者上传的可复用模块。发布到 Npm 仓库的模块有以下几个特点：

* 每个模块根目录下都必须有一个描述该模块的 `package.json` 文件。该文件描述了模块的入口文件、依赖等。
* 模块中的文件以 JavaScript 文件为主，但不限于 JavaScript 文件
* 模块中的代码大多采用模块化规范，因为你的这个模块可能依赖其它模块，而且别的模块又可能依赖你的这个模块。因为目前支持比较广泛的是 CommonJS 模块化规范，上传到 Npm 仓库的代码最好遵守该规范。

##### 抛出问题& 解决(构建)

Webpack 不仅可用于构建运行的应用，也可用于构建上传到 Npm 的模块。以可上传的 Npm 仓库的 React 组件为例，要求：

* 1，源代码采用 ES6 写，但发布到 Npm 仓库的需要是 ES5 的，并且遵守 CommonJS 模块化规范。如果发布到 Npm 上去的 ES5 代码是经过转换的，请同时提供 Source Map 以方便调试；

```js
/* 
使用 babel-loader 把 ES6 代码转换成 ES5 的代码。
通过开启 devtool: 'source-map' 输出 Source Map 以发布调试。
设置 output.libraryTarget='commonjs2' 使输出的代码符合CommonJS2 模块化规范，以供给其它模块导入使用。
注：无新增依赖
*/
```

* 2，该 UI 组件依赖的其它资源文件例如 CSS 文件也需要包含在发布的模块里；

```js
/*
通过 css-loader 和 extract-text-webpack-plugin 实现
新增依赖：
*/
```

* 3，尽量减少冗余代码，减少发布出去的组件的代码文件大小；

```js
/*
默认的情况下 Babel 会在每个输出文件中内嵌（将es6转化为es5）辅助函数的代码。如果多个源代码文件都依赖这些辅助函数，那么这些辅助函数的代码将会重复的出现很多次，造成代码冗余。 为了不让这些辅助函数的代重复出现，可以在依赖它们的时候通过 require('babel-runtime/helpers/createClass') 的方式去导入，这样就能做到只让它们出现一次。 babel-plugin-transform-runtime 插件就是用来做这个事情的。
由于加入 babel-plugin-transform-runtime 后生成的代码中会大量出现类似 require('babel-runtime/helpers/createClass') 这样的语句，所以输出的代码将依赖 babel-runtime 模块。
新增依赖：babel-plugin-transform-runtime，babel-runtime
*/
```

* 4，发布出去的组件的代码中不能含有其依赖的模块的代码，而是让用户可选择性的去安装。例如不能内嵌 React 库的代码，这样做的目的是在其它组件也依赖 React 库时，防止 React 库的代码被重复打包

```js
/* 
通过Externals实现：
Externals 用来告诉 Webpack 要构建的代码中使用了哪些不用被打包的模块，也就是说这些模版是外部环境提供的，Webpack 在打包时可以忽略它们。
*/
module.exports = {
  // 通过正则命中所有以 react 或者 babel-runtime 开头的模块
  // 这些模块通过注册在运行环境中的全局变量访问，不用被重复打包进输出的代码里
  externals: /^(react|babel-runtime)/,
};
/*
开启以上配置后，输出的代码中会存在导入 react 或者 babel-runtime 模块的代码，但是它们的 react 或者 babel-runtime 的内容不会被包含进去，如下：
*/
[
    (function (module, exports) {
        module.exports = require("babel-runtime/helpers/inherits");
    }),
    (function (module, exports) {
        module.exports = require("react");
    })
]
/*
这样就做到了在保持代码正确性的情况下，输出文件不存放 react 或者 babel-runtime 模块的代码。
*/
/*
当你在开发 Npm 模块时，需要对所有正在开发的模块所依赖的模块进行这样的处理。
*/
```

##### 发布到 Npm

1. 确保描述文件正确，如入口文件

```json
{
  "main": "lib/index.js", // 构建出的代码的入口文件
  "jsnext:main": "src/index.js" // 采用 ES6 编写的模块入口文件所在的位
}
```

2. 执行`npm publish`，将构建的代码发布到Npm 仓库中（确保已经 npm login）

注： Webpack 适合于构建完整不可分割的 Npm 模块，Lodash或组件库源码是一个个分割的模块化文件，不适合用webpack构建。

##### 疑问

通过如下的引用，无法在构建完成后，展示html文件

```js
const { default: HelloWebpack } = require('../lib/index.js')
```

 

#### 四）构建离线应用

##### 认识离线应用

离线应用是指通过离线缓存技术，让资源在第一次被加载后缓存在本地，下次访问它时就直接返回本地的文件，即使没有网络连接。离线应用具有以下优点：

* 在没有网络的情况下也能打开网页；
* 由于部分被缓存的资源直接从本地加载，用户端可以加速网页加载速度，网站运营端可以减少服务器压力以及传输流量费用

离线应用的核心是离线缓存技术，历史上曾先后出现2种离线缓存技术，它们分别是：

1. [AppCache](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache) 又叫 Application Cache，目前已经从 Web 标准中删除；
2. [Service Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers) 是目前最新的离线缓存技术，是 [Web Worker](http://javascript.ruanyifeng.com/htmlapi/webworker.html) 的一部分。 它通过拦截网络请求实现离线缓存，比 AppCache 更加灵活（可以通过 JavaScript 代码去控制缓存的逻辑）。它也是构建 [PWA](https://developer.mozilla.org/zh-CN/Apps/Progressive) 应用的关键技术之一。

##### 认识 Service Workers

Service Workers 是一个在浏览器后台运行的脚本，它生命周期完全独立于网页。它无法直接访问 DOM，但可以通过 postMessage 接口发送消息来和 UI 进程通信。 拦截网络请求是 Service Workers 的一个重要功能，通过它能完成离线缓存、编辑响应、过滤响应等功能。

###### 兼容性

目前 Chrome、Firefox、Opera 都已经全面支持 Service Workers，但对于移动端浏览器，只有高版本的 Android 支持。由于 Service Workers 无法通过注入 polyfill 去实现兼容，所以在你打算使用它前请先调查清楚你的网页的运行场景。

判断浏览器是否支持 Service Workers 的最简单的方法是通过以下代码：

```js
// 如果 navigator 对象上存在 serviceWorker 对象，就表示支持
if (navigator.serviceWorker) {
  // 通过 navigator.serviceWorker 使用
}
```

###### 注册 Service Workers

要给网页接入 Service Workers，需要在网页加载后注册一个描述 Service Workers 逻辑的脚本。 代码如下：

```js
if (navigator.serviceWorker) {
  window.addEventListener('DOMContentLoaded',function() {
    // 调用 serviceWorker.register 注册，参数 /sw.js 为脚本文件所在的 URL 路径
      navigator.serviceWorker.register('/sw.js');
  });
}
```

一旦这个脚本文件被加载，Service Workers 的安装就开始了。这个脚本被安装到浏览器中后，就算用户关闭了当前网页，它仍会存在。 也就是说第一次打开该网页时 Service Workers 的逻辑不会生效，因为脚本还没有被加载和注册，但是以后再次打开该网页时脚本里的逻辑将会生效。

`chrome://inspect/#service-workers`可以查看chrome所有注册了的 Service Workers。

###### 使用 Service Workers 实现离线缓存

Service Workers 在注册成功后会在其生命周期中派发出一些事件，通过监听对应的事件在特点的时间节点上做一些事情。

在 Service Workers 安装成功后会派发出 `install` 事件，需要在这个事件中执行缓存资源的逻辑，接下来监听网络请求事件去拦截请求，复用缓存。这样就实现了离线缓存。

```js
// 当前缓存版本的唯一标识符，用当前时间代替
var cacheKey = new Date().toISOString();

// 需要被缓存的文件的 URL 列表
var cacheFileList = [
  '/index.html',
  '/app.js',
  '/app.css'
];
// self（关键字）代表当前的 Service Workers 实例。
// 监听 install 事件
self.addEventListener('install', function (event) {
  // 等待所有资源缓存完成时，才可以进行下一步
  event.waitUntil(
    caches.open(cacheKey).then(function (cache) {
      // 要缓存的文件 URL 列表
      return cache.addAll(cacheFileList);
    })
  );
});
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // 去缓存中查询对应的请求
    caches.match(event.request).then(function(response) {
        // 如果命中本地缓存，就直接返回本地的资源
        if (response) {
          return response;
        }
        // 否则就去用 fetch 下载资源
        return fetch(event.request);
      }
    )
  );
});
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // 去缓存中查询对应的请求
    caches.match(event.request).then(function(response) {
        // 如果命中本地缓存，就直接返回本地的资源
        if (response) {
          return response;
        }
        // 否则就去用 fetch 下载资源
        return fetch(event.request);
      }
    )
  );
});
```

###### 更新缓存

线上的代码有时需要更新和重新发布，如果这个文件被离线缓存了，那就需要 Service Workers 脚本中有对应的逻辑去更新缓存。 （注：类似于内置包，是指发布的应用，而不是指请求（缓存））。这可以通过更新 Service Workers 脚本文件做到。

浏览器针对 Service Workers 有如下机制：

1. 每次打开接入了 Service Workers 的网页时，浏览器都会去重新下载 Service Workers 脚本文件（所以要注意该脚本文件不能太大），如果发现和当前已经注册过的文件存在字节差异，就将其视为“新服务工作线程”。
2. 新 Service Workers 线程将会启动，且将会触发其 install 事件。
3. 当网站上当前打开的页面关闭时，旧 Service Workers 线程将会被终止，新 Service Workers 线程将会取得控制权。
4. 新 Service Workers 线程取得控制权后，将会触发其 activate 事件。

新 Service Workers 线程中的 activate 事件就是最佳的清理旧缓存的时间点。

```js
// 当前缓存白名单，在新脚本的 install 事件里将使用白名单里的 key 
var cacheWhitelist = [cacheKey];
// 新 Service Workers 线程取得控制权后，将会触发其 activate 事件
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          // 不在白名单的缓存全部清理掉
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            // 删除缓存
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

##### 接入 Webpack

用Webpack构建接入Service Workers 的离线关键在于：

1. 生成上面提到的 `sw.js` 文件
2. 根据输出文件动态生成`sw.js`文件中`cacheFileList` 变量，即需要被缓存文件的 URL 列表

❗️注：由于依赖原因，暂时没有跑通。暂缓



### 五，Webpack 结合其它工具搭配使用

#### 一）搭配 Npm Script

##### 认识 Npm Script

[Npm Script](https://docs.npmjs.com/misc/scripts) 是一个任务执行者。 Npm 是在安装 Node.js 时附带的包管理器，Npm Script 则是 Npm 内置的一个功能，允许在 `package.json` 文件里面使用 `scripts` 字段定义任务。

```json
{
  "scripts": {
    "dev": "node dev.js",
    "pub": "node build.js"
  }
}
```

`scripts` 字段是一个对象，每一个属性对应一段脚本。Npm Script 底层是通过调用 Shell 去运行脚本命令，如执行`npm run pub` 命令等同于执行命令 `node build.js`。

Npm Script 能运行安装到项目目录里的 `node_modules` 里的可执行模块，如针对 webpack，是无法直接在项目根目录下通过命令 `webpack` 去执行 Webpack 构建的，而是通过 `./node_modules/.bin/webpack` 去执行。

通过 Npm Script只需要在 `scripts` 字段里定义一个任务，如下：

```json
{
  "scripts": {
    "build": "webpack"
  }
}
```

Npm Script 会先去项目目录下的 `node_modules` 中寻找有没有可执行的 `webpack` 文件，如果有就使用本地的，如果没有就使用全局的。 所以现在执行 Webpack 构建只需要通过执行 `npm run build` 去实现。



##### Webpack 为什么需要 Npm Script

Webpack 只是一个打包模块化代码的工具，并没有提供任何任务管理相关的功能。 但在实际场景中通常不会是只通过执行 `webpack` 就能完成所有任务的，而是需要多个任务才能完成。

使用 Npm Script 的好处是把一连串复杂的流程简化成了一个简单的命令，需要时只需要执行对应的那个简短的命令，而不用去手动的重复整个流程。 这会大大的提高我们的效率和降低出错率。



#### 二）检查代码

##### 了解代码检查

检查代码是通过机器去执行一些自动化的检查。主要包括以下几项：

* 代码风格：让项目成员强制遵守统一的代码风格，如缩进、注释等
* 潜在问题：分析代码运行时潜在的bug

##### 怎么做代码检查

做代码风格检查时需要按照不同的文件类型进行

###### 检查JavaScript

最常用的 JavaScript 检查工具是 [ESlint](https://eslint.org/)，不仅内置大量规则，而且可以通过插件灵活拓展。具体执行：

* 1，安装，``` npm i -g eslint```
* 2，初始化配置，项目目录下执行``` eslint --init```
* 3，执行检查``` eslint yourfile.js```

###### 检查 TypeScript

TSLint 只专注于检查 TypeScript 代码。使用方法类似ESlint，

###### 检查CSS

[stylelint](https://stylelint.io/) 是目前最成熟的 CSS 检查工具，内置规则且可以通过插件拓展。stylelint 基于 PostCSS，能检查任何 PostCSS 能解析的代码，诸如 SCSS、Less 等。具体使用：

* 1，安装 ``` npm i -g stylelint```
* 2，项目根目录下新建 `.stylelintrc` 配置文件
* 3，执行 ``` stylelint "yourfile.css"``` 检查`yourfile.css` 文件

##### 结合 webpack 检查代码

以上介绍的代码检查工具可以和 Webpack 结合起来，在开发过程中通过 Webpack 输出实时的检查结果。

###### 结合 ESLint

[eslint-loader](https://github.com/MoOx/eslint-loader) 可以方便的把 ESLint 整合到 Webpack 中，使用方法如下：

```json
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        // node_modules 目录的下的代码不用检查
        exclude: /node_modules/,
        loader: 'eslint-loader',
        // 把 eslint-loader 的执行顺序放到最前面，防止其它 Loader 把处理后的代码交给 eslint-loader 去检查
        enforce: 'pre',
      },
    ],
  },
}
```

接入 eslint-loader 后就能在控制台中看到 ESLint 输出的错误日志了。

###### 结合 TSLint

[tslint-loader](https://github.com/wbuchwalter/tslint-loader) 是一个和 eslint-loader 相似的 Webpack Loader， 能方便的把 TSLint 整合到 Webpack，其使用方法参考 ESLint。

###### 结合 stylelint

[StyleLintPlugin](https://github.com/JaKXz/stylelint-webpack-plugin) 能把 stylelint 整合到 Webpack，其使用方法如下：

```js
const StyleLintPlugin = require('stylelint-webpack-plugin');
module.exports = {
  // ...
  plugins: [
    new StyleLintPlugin(),
  ],
}
```

##### 建议

把代码检查功能整合到 Webpack 中会导致以下问题：

* 由于执行检查步骤计算量大，整合到 Webpack 中会导致构建变慢
* 在整合代码检查到 Webpack 后，输出的错误信息是通过行号来定位错误的，没有编辑器集成显示错误直观

针对此：

* 使用集成了代码检查功能的编辑器，让编辑器实时直观地显示错误；
* 把代码检查步骤放到代码提交时，也就是说在代码提交前去调用以上检查工具去检查代码，只有在检查都通过时才提交代码，这样就能保证提交到仓库的代码都是通过了检查的

Git 提供了 Hook 功能能做到在提交代码前触发执行脚本，[husky](https://github.com/typicode/husky) 可以方便快速地为项目接入 Git Hook， 执行：

```bash
npm i -D husky
```

安装 husky 时，husky 会通过 Npm Script Hook 自动配置好 Git Hook，你需要做的只是在 `package.json` 文件中定义几个脚本，方法如下：

```js
{
  "scripts": {
    // 在执行 git commit 前会执行的脚本  
    "precommit": "npm run lint",
    // 在执行 git push 前会执行的脚本  
    "prepush": "lint",
    // 调用 eslint、stylelint 等工具检查代码
    "lint": "eslint && stylelint"
  }
}
```

`precommit` 和 `prepush` 你需要根据自己的情况选择一个，无需两个都设置。



#### 三）通过 Node.js API 启动 Webpack

Webpack 除了提供可执行的命令行工具外，还提供可在 Node.js 环境中调用的库。 通过 Webpack 暴露的 API，可直接在 Node.js 程序中调用 Webpack 执行构建。

通过 API 去调用并执行 Webpack 比直接通过可执行文件启动更加灵活，可用在一些特殊场景。

注：Webpack 其实是一个 Node.js 应用程序，它全部通过 JavaScript 开发完成。 在命令行中执行 `webpack` 命令其实等价于执行 `node ./node_modules/webpack/bin/webpack.js`

##### 安装和使用 Webpack 模块

具体的的步骤如下：

* 1，在调用 Webpack API 前，需要先安装它，``` npm i -D webpack```
* 2，导入 Webpack 模块，``` const webpack = require('webpack')``` 或``` import webpack from "webpack"```
* 3，导出的 `webpack` 是一个函数，使用方法如下：

```js
webpack({
  // Webpack 配置，和 webpack.config.js 文件一致
}, (err, stats) => {
  if (err || stats.hasErrors()) {
    // 构建过程出错
  }
  // 成功执行完构建
});
// 如果你的 Webpack 配置写在 webpack.config.js 文件中，可以这样使用：
// 读取 webpack.config.js 文件中的配置
const config = require('./webpack.config.js');
webpack(config , callback);
```

##### 以监听模式运行

以上使用 Webpack API 的方法只能执行一次构建，无法以监听模式启动 Webpack，为了在使用 API 时以监听模式启动，需要获取 Compiler 实例，方法如下：

```js
// 如果不传 callback 回调函数，就会返回一个 Compiler 实例，用于让你去控制启动，而不是像上面那样立即启动
const compiler = webpack(config);

// 调用 compiler.watch 以监听模式启动，返回的 watching 用于关闭监听
const watching = compiler.watch({
  // watchOptions 对应 2-7 其它配置项 中介绍过的 Watch 和 WatchOptions
  aggregateTimeout: 300,
},(err, stats)=>{
  // 每次因文件发生变化而重新执行完构建后
});

// 调用 watching.close 关闭监听
watching.close(()=>{
  // 在监听关闭后
});
```

#### 四）使用 Webpack Dev Middleware

前文介绍的 DevServer 是一个方便开发的小型 HTTP 服务器，DevServer 其实是基于 [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) 和 [Expressjs](https://expressjs.com/) 实现的， 而 webpack-dev-middleware 其实是 Expressjs 的一个中间件。

实现 DevServer 基本功能的代码大致如下：

```js
const express = require('express');
const webpack = require('webpack');
const webpackMiddleware = require('webpack-dev-middleware');

// 从 webpack.config.js 文件中读取 Webpack 配置 
const config = require('./webpack.config.js');
// 用读取到的 Webpack 配置实例化一个 Compiler
const compiler = webpack(config);

// 实例化一个 Expressjs app
const app = express();
// 给 app 注册 webpackMiddleware 中间件
app.use(webpackMiddleware(compiler));
// 启动 HTTP 服务器，服务器监听在 3000 端口 
app.listen(3000);
```

`webpackMiddleware` 函数的返回结果是一个 Expressjs 的中间件，该中间件有以下功能：

- 接收来自 Webpack Compiler 实例输出的文件，但不会把文件输出到硬盘，而是保存在内存中；
- 往 Expressjs app 上注册路由，拦截 HTTP 收到的请求，根据请求路径响应对应的文件内容；

通过 webpack-dev-middleware 能够将 DevServer 集成到你现有的 HTTP 服务器中，让你现有的 HTTP 服务器能返回 Webpack 构建出的内容，而不是在开发时启动多个 HTTP 服务器。 这特别适用于后端接口服务采用 Node.js 编写的项目。

##### Webpack Dev Middleware 支持的配置项

在 Node.js 中调用 webpack-dev-middleware 提供的 API 时，还可以给它传入一些配置项，方法如下：

```js
// webpackMiddleware 函数的第二个参数为配置项
app.use(webpackMiddleware(compiler, {
    // webpack-dev-middleware 所有支持的配置项
    // 只有 publicPath 属性为必填，其它都是选填项
    // Webpack 输出资源绑定在 HTTP 服务器上的根目录，
    // 和 Webpack 配置中的 publicPath 含义一致 
    publicPath: '/assets/',
    // 不输出 info 类型的日志到控制台，只输出 warn 和 error 类型的日志
    noInfo: false,
    // 不输出任何类型的日志到控制台
    quiet: false,
    // 切换到懒惰模式，这意味着不监听文件变化，只会在请求到时再去编译对应的文件，
    // 这适合页面非常多的项目。
    lazy: true,
    // watchOptions
    // 只在非懒惰模式下才有效
    watchOptions: {
        aggregateTimeout: 300,
        poll: true
    },
    // 默认的 URL 路径, 默认是 'index.html'.
    index: 'index.html',
    // 自定义 HTTP 头
    headers: {'X-Custom-Header': 'yes'},
    // 给特定文件后缀的文件添加 HTTP mimeTypes ，作为文件类型映射表
    mimeTypes: {'text/html': ['phtml']},
    // 统计信息输出样式
    stats: {
        colors: true
    },
    // 自定义输出日志的展示方法
    reporter: null,
    // 开启或关闭服务端渲染
    serverSideRender: false,
}));
```

##### Webpack Dev Middleware 与模块热替换

DevServer 提供了一个方便的功能，可以做到在监听到文件发生变化时自动替换网页中的老模块，以做到实时预览。 DevServer 虽然是基于 webpack-dev-middleware 实现的，但 webpack-dev-middleware 并没有实现模块热替换功能，而是 DevServer 自己实现了该功能。

DevServer 提供了一个方便的功能，可以做到在监听到文件发生变化时自动替换网页中的老模块，以做到实时预览。 DevServer 虽然是基于 webpack-dev-middleware 实现的，但 webpack-dev-middleware 并没有实现模块热替换功能，而是 DevServer 自己实现了该功能。为了在使用 webpack-dev-middleware 时也能使用模块热替换功能去提升开发效率，需要额外的再接入 [webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware)。 需要做以下修改才能实现模块热替换。

* 1，修改 HTTP 服务器代码 `server.js` 文件，接入 `webpack-hot-middleware` 中间件，
* 2，修改 `webpack.config.js` 文件，加入 `HotModuleReplacementPlugin` 插件
* 3，修改执行入口文件 `main.js`，加入替换逻辑
* 4，第4步：安装新引入的依赖

安装成功后，通过 `node ./server.js` 就能启动一个类似于 DevServer 那样支持模块热替换的自定义 HTTP 服务了。

### 六，用 Webpack 加载特殊类型的资源

#### 一）加载图片

##### css和js导入图片资源

```css
#app {
  background-image: url(./imgs/a.png);
}
```

```js
import imgB from './imgs/b.png';

window.document.getElementById('app').innerHTML = `
<img src="${imgB}"/>
`;
```

##### 使用file-loader

[file-loader](https://github.com/webpack-contrib/file-loader) 可以把 JavaScript 和 CSS 中导入图片的语句替换成正确的地址，并同时把文件输出到对应的位置，相关配置如下：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/,
        use: ['file-loader']
      }
    ]
  }
};
```

被 file-loader 转换后输出的 CSS 输出：

```css
#app {
  background-image: url(5556e1251a78c5afda9ee7dd06ad109b.png);
}
```

经过 file-loader 处理后输出的 JavaScript 代码如下：

```js
module.exports = __webpack_require__.p + "0bcc1f8d385f78e1271ebfca50668429.png";
```

##### 使用 url-loader

[url-loader](https://github.com/webpack-contrib/url-loader) 可以把文件的内容经过 base64 编码后注入到 JavaScript 或者 CSS 中去。由于一般的图片数据量巨大， 这会导致 JavaScript、CSS 文件也跟着变大。 所以在使用 url-loader 时一定要注意图片体积不能太大，不然会导致 JavaScript、CSS 文件过大而带来的网页加载缓慢问题。

一般利用 url-loader 把网页需要用到的小图片资源注入到代码中去，以减少加载次数。因为在 HTTP/1 协议中，每加载一个资源都需要建立一次 HTTP 链接， 为了一个很小的图片而新建一次 HTTP 连接是不划算的。

url-loader 考虑到了以上问题，并提供了一个方便的选择 `limit`，该选项用于控制当文件大小小于 limit 时才使用 url-loader，否则使用 `fallback` 选项中配置的 loader。 相关 Webpack 配置如下：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.png$/,
        use: [{
          loader: 'url-loader',
          options: {
            // 30KB 以下的文件采用 url-loader
            limit: 1024 * 30,
            // 否则采用 file-loader，默认值就是 file-loader 
            fallback: 'file-loader',
          }
        }]
      }
    ]
  },
};
```

除此之外，你还可以做以下优化：

* 1，通过 [imagemin-webpack-plugin](https://www.npmjs.com/package/imagemin-webpack-plugin) 压缩图片；
* 2，通过 [webpack-spritesmith](https://www.npmjs.com/package/webpack-spritesmith) 插件制作雪碧图；

注：以上加载图片的方法同样适用于其它二进制类型的资源，例如 PDF、SWF 等等。

#### 二）加载 SVG

SVG 作为矢量图的一种标准格式，已经得到了各大浏览器的支持，它也成为了 Web 中矢量图的代名词。 在网页中采用 SVG 代替位图有如下好处：

1. SVG 相对于位图更清晰，在任意缩放的情况下后不会破坏图形的清晰度，SVG 能方便地解决高分辨率屏幕下图像显示不清楚的问题；
2. 在图形线条比较简单的情况下，SVG 文件的大小要小于位图，在扁平化 UI 流行的今天，多数情况下 SVG 会更小；
3. 图形相同的 SVG 比对应的高清图有更好的渲染性能；
4. SVG 采用和 HTML 一致的 XML 语法描述，灵活性很高

SVG 的导入方法和图片类似，可以使用在css或者html中。也即可以直接把 SVG 文件当成一张图片，**使用 file-loader** 和 **使用 url-loader**进行处理，只是 Loader test 配置中的文件后缀改成 `.svg`。

##### 使用 raw-loader

[raw-loader](https://github.com/webpack-contrib/raw-loader) 可以把文本文件的内容读取出来，注入到 JavaScript 或 CSS 中去。

源代码：

```js
import svgContent from './svgs/alert.svg';
// 输出：module.exports = "<svg xmlns=\"http://www.w3.org/2000/svg\"... </svg>" // 末尾省略 SVG 内容
// svgContent 的内容就等于字符串形式的 SVG，
// 由于 SVG 本身就是 HTML 元素，在获取到 SVG 内容后，可以直接通过以下代码将 SVG 插入到网页中：
window.document.getElementById('app').innerHTML = svgContent;
```

注：由于 raw-loader 会直接返回 SVG 的文本内容，并且无法通过 CSS 去展示 SVG 的文本内容，因此采用本方法后无法在 CSS 中导入 SVG。 也就是说在 CSS 中不可以出现 `background-image: url(./svgs/activity.svg)` 这样的代码，因为 `background-image: url(<svg>...</svg>)` 是不合法的。

##### 使用 svg-inline-loader

[svg-inline-loader](https://github.com/webpack-contrib/svg-inline-loader) 和上面提到的 raw-loader 非常相似， 不同在于 svg-inline-loader 会分析 SVG 的内容，去除其中不必要的部分代码，以减少 SVG 的文件大小。也就是说 svg-inline-loader 增加了对 SVG 的压缩功能。

#### 三）加载Source Map

Webpack 支持为转换生成的代码输出对应的 Source Map 文件，以方便在浏览器中能通过源码调试。 控制 Source Map 输出的 Webpack 配置项是 `devtool`，它有很多选项，如

| devtool                 | 含义                                                         |
| :---------------------- | :----------------------------------------------------------- |
| 空                      | 不生成 Source Map                                            |
| eval                    | 每个 module 会封装到 eval 里包裹起来执行，并且会在每个 eval 语句的末尾追加注释 `//# sourceURL=webpack:///./main.js` |
| source-map              | 会额外生成一个单独 Source Map 文件，并且会在 JavaScript 文件末尾追加 `//# sourceMappingURL=bundle.js.map` |
| hidden-source-map       | 和 source-map 类似，但不会在 JavaScript 文件末尾追加 `//# sourceMappingURL=bundle.js.map` |
| inline-source-map       | 和 source-map 类似，但不会额外生成一个单独 Source Map 文件，而是把 Source Map 转换成 base64 编码内嵌到 JavaScript 中 |
| eval-source-map         | 和 eval 类似，但会把每个模块的 Source Map 转换成 base64 编码内嵌到 eval 语句的末尾，例如 `//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW...` |
| cheap-source-map        | 和 source-map 类似，但生成的 Source Map 文件中没有列信息，因此生成速度更快 |
| cheap-module-source-map | 和 cheap-source-map 类似，但会包含 Loader 生成的 Source Map  |

其实以上只是列举了 devtool 可能取值的一部分， 它的取值其实可以由 `source-map`、`eval`、`inline`、`hidden`、`cheap`、`module` 这六个关键字随意组合而成。 这六个关键字每个都代表一种特性，它们的含义分别是

* eval：用 `eval` 语句包裹需要安装的模块；
* source-map：生成独立的 Source Map 文件；
* hidden：不在 JavaScript 文件中指出 Source Map 文件所在，这样浏览器就不会自动加载 Source Map；
* inline：把生成的 Source Map 转换成 base64 格式内嵌在 JavaScript 文件中；
* cheap：生成的 Source Map 中不会包含列信息，这样计算量更小，输出的 Source Map 文件更小；同时 Loader 输出的 Source Map 不会被采用；
* module：来自 Loader 的 Source Map 被简单处理成每行一个模块；

##### 如何选择

如果你不关心细节和性能，只是想在不出任何差错的情况下调试源码，可以直接设置成 `source-map`，但这样会造成两个问题：

* `source-map` 模式下会输出质量最高最详细的 Source Map，这会造成构建速度缓慢，特别是在开发过程需要频繁修改的时候会增加等待时间；
* `source-map` 模式下会把 Source Map 暴露出去，如果构建发布到线上的代码的 Source Map 暴露出去就等于源码被泄露；

为了解决以上两个问题，可以这样做：

* 在开发环境下把 `devtool` 设置成 `cheap-module-eval-source-map`，因为生成这种 Source Map 的速度最快，能加速构建。由于在开发环境下不会做代码压缩，Source Map 中即使没有列信息也不会影响断点调试；
* 在生产环境下把 `devtool` 设置成 `hidden-source-map`，意思是生成最详细的 Source Map，但不会把 Source Map 暴露出去。由于在生产环境下会做代码压缩，一个 JavaScript 文件只有一行，所以需要列信息。

注：

* 在生产环境下通常不会把 Source Map 上传到 HTTP 服务器让用户获取，而是上传到 JavaScript 错误收集系统，在错误收集系统上根据 Source Map 和收集到的 JavaScript 运行错误堆栈计算出错误所在源码的位置。

* 不要在生产环境下使用 `inline` 模式的 Source Map， 因为这会使 JavaScript 文件变得很大，而且会泄露源码。

##### 加载现有的 Source Map

有些从 Npm 安装的第三方模块是采用 ES6 或者 TypeScript 编写的，它们在发布时会同时带上编译出来的 JavaScript 文件和对应的 Source Map 文件，以方便你在使用它们出问题的时候调试它们；

默认情况下 Webpack 是不会去加载这些附加的 Source Map 文件的，Webpack 只会在转换过程中生成 Source Map。 为了让 Webpack 加载这些附加的 Source Map 文件，需要安装 [source-map-loader](https://github.com/webpack-contrib/source-map-loader) 。 使用方法如下：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        // 只加载你关心的目录下的 Source Map，以提升构建速度
        include: [path.resolve(root, 'node_modules/some-components/')],
        use: ['source-map-loader'],
        // 要把 source-map-loader 的执行顺序放到最前面，如果在 source-map-loader 之前有 Loader 转换了该 JavaScript 文件，会导致 Source Map 映射错误
        enforce: 'pre'
      }
    ]
  }
};
```

由于 source-map-loader 在加载 Source Map 时计算量很大，因此要避免让该 Loader 处理过多的文件，不然会导致构建速度缓慢。 通常会采用 `include` 去命中只关心的文件。

注意安装新引入的依赖：

```js
npm i -D source-map-loader
```

重启 Webpack 后，你就能在浏览器中调试 `node_modules/some-components/` 目录下的源码了。



