---
layout:     post
title:      深入浅出 webapck & webapck 文档
subtitle:   chaper_02: 配置
date:       2023-08-31
author:     forwardZ
header-img: img/the-first.png
catalog: false
tags:
    - 前端
---

<!-- [webpack-chap01.png](https://postimg.cc/QFcgdnTM) -->




## chapter_02：配置
配置webpack 的两种方式，

1. 通过js 文件描述配置，如 ``` webpack.config.js ```
2. 通过命令行参数传入，如 ```webpack --devtool source-map ```, ``` webpack --config webpack-dev.config.js ```

两种方式可以搭配，如先通过 ``` webpack --config webpack-dev.config.js ```指定配置文件，再去配置文件``` webpack.config.js ```中描述部分配置，单纯通过命令行进行配置的较少。

配置按影响功能可以划分如下：

1. entry 配置模块的入口
2. output 配置输出最终想要的代码
3. module 配置处理模块的规则
4. resolve 配置寻找模块的规则
5. plugins 配置拓展的插件
6. devServer 配置 devServer（补充）
7. 其它零散的配置项
8. 整体地描述各配置项的结构
9. 配置文件不止可以返回一个 Object，还有其他返回形式
10. 配置总结，寻找配置 Webpack 的规律，减少思维负担

### 一，entry
entry 是配置模块的入口，可抽象为输入，webpack 执行构建的第一步就是从入口开始搜索，并递归解析出所有依赖的模块。

entry 是配置必填项。

#### 一）context
webpack 寻找相对路径的文件时，以context 为根目录。context 默认为执行webpack时所在的当前工作目录。如果想改变 context 的默认配置，则可以在配置文件里这样设置它：

```
const path = require('path');
module.exports = {
    context: path.resolve(__dirname,'app')
}
```

注：context 必须是一个绝对路径的字符串。

之所以先介绍 context，因为 Entry 的路径和其依赖的模块的路径可能采用相对于 context 的路径来描述，context 会影响到这些相对路径所指向的真实文件。

#### 二）entry 类型

Entry 类型可以是以下三种中的一种或者相互组合：

| 类型   | 例子                                                         | 含义                                 |
| ------ | ------------------------------------------------------------ | ------------------------------------ |
| string | `'./app/entry'`                                              | 入口模块的文件路径，可以是相对路径。 |
| array  | `['./app/entry1', './app/entry2']`                           | 入口模块的文件路径，可以是相对路径。 |
| object | `{ a: './app/entry-a', b: ['./app/entry-b1', './app/entry-b2']}` | 配置多个入口，每个入口生成一个 Chunk |

注：如果是 array 类型，则搭配 output.library 配置项使用时，只有数组里的最后一个入口文件的模块会被导出。

#### 二）chunk 名称
Webpack 会为每个生成的 Chunk 取一个名称，Chunk 的名称和 Entry 的配置有关

* 如果 entry 是一个 string 或 array，就只会生成一个 Chunk，这时 Chunk 的名称是 main

* 如果 entry 是一个 object，就可能会出现多个 Chunk，这时 Chunk 的名称是 object 键值对里键的名称

  PS：chunk 即输出的产物

#### 三）动态配置 entry
假如项目里有多个页面需要为每个页面的入口配置一个 Entry ，但Entry 的配置会受到到其他因素的影响导致不能写成静态的值，其解决方法是把 Entry 设置成一个函数去动态返回上面所说的配置。

```
entry: () => {
    return {
        a: './pages/a',
        b: './pages/b',
    }
}

entry: () => {
    return new Promise((resolve) => {
        resolve({
            a: './pages/a',
            b: './pages/b',
        })
    })
}
```



#### entry相关的整体配置

```js
module.exports = {
  context: __dirname, // Webpack 使用的根目录，string 类型必须是绝对路径
	// entry 表示 入口，Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入。
  // 类型可以是 string | object | array   
  entry: './app/entry', // 只有1个入口，入口只有1个文件
  entry: ['./app/entry1', './app/entry2'], // 只有1个入口，入口有2个文件
  entry: { // 有2个入口
    a: './app/entry-a',
    b: ['./app/entry-b1', './app/entry-b2']
  },
}
```





### 二，output

`output` 配置如何输出最终想要的代码。`output` 是一个 `object`，里面包含一系列配置项，下面为常用的配置项。

#### 一）filename

`output.filename` 配置输出文件的名称，为string 类型。

* 只有一个输出文件时，filename可以是静态不变的，如` filename: 'bundle.js'`
* 有多个 Chunk 要输出时，借助模版和变量，Webpack 会为每个 Chunk取一个名称，可以根据 Chunk 的名称来区分输出的文件名，如` filename: '[name].js'`。内置变量除了name（Chunk 的名称），还包括id（Chunk 的唯一标识，从0开始），hash（Chunk 的唯一标识的 Hash 值），chunkhash（Chunk 内容的 Hash 值）。其中 `hash` 和 `chunkhash` 的长度是可指定的，`[hash:8]` 代表取8位 Hash 值，默认是20位。





#### 二）chunkFilename

1.   `output.chunkFilename` 配置无入口的 Chunk 在输出时的文件名称。

2. chunkFilename 和上面的 filename 非常类似，但 chunkFilename 只用于指定在运行过程中生成的 Chunk 在输出时的文件名称。

3.   常见的会在运行时生成 Chunk 场景有在使用 CommonChunkPlugin、使用 `import('path/to/module')` 动态加载等时。 

4. chunkFilename 支持和 filename 一致的内置变量。

   PS：chunkFilename 用于指定运行过程生成的chunk的文件名称，运行时生成chunk的场景包括import动态加载、使用CommonChunkPlugin等。



#### 三）path

`output.path` 配置输出文件存放在本地的目录，必须是 string 类型的绝对路径。通常通过 Node.js 的 `path` 模块去获取绝对路径：

```js
module.exports = {
    // ...
    output:{
        // ...
        path: path.resolve(__dirname, 'dist_[hash]')
    }
}
```

output.path 支持字符串模版，内置变量只有一个：`hash` 代表一次编译操作的 Hash 值。



#### 四）publicPath

当需要把构建出的资源文件上传到 CDN 服务上，以利于加快页面的打开速度。配置代码如下：

```js
module.exports = {
    // ...
    output:{
        filename:`[name]_[chunkhash:8].js`,
        publicPath:'[name]_[chunkhash:8].js'
    }
}
```

这时发布到线上的 HTML 在引入 JavaScript 文件时就需要：

```html
<script src='https://cdn.example.com/assets/a_12345678.js'></script>
```

总结：

1. `output.publicPath` 配置发布到线上资源的 URL 前缀，为string 类型。 默认值是空字符串 `''`，即使用相对路径。
2. `output.publicPath` 支持字符串模版，内置变量只有一个：`hash` 代表一次编译操作的 Hash 值。

注：此处应该有一个上传的资源文件的插件，用于将构建的资源上传至 CDN。



#### 五）crossOriginLoading

Webpack 输出的部分代码块可能需要异步加载，而异步加载是通过 [JSONP](https://zh.wikipedia.org/wiki/JSONP) 方式实现的。JSONP 的原理是动态地向 HTML 中插入一个 `<script src="url"></script>` 标签去加载异步资源。 `output.crossOriginLoading` 则是用于配置这个异步插入的标签的 `crossorigin` 值。

script 标签的 crossorigin 属性可以取以下值：

* anonymous：(默认) 在加载此脚本资源时不会带上用户的 Cookies；
* use-credentials：在加载此脚本资源时会带上用户的 Cookies。

通常用设置 crossorigin 来获取异步加载的脚本执行时的详细错误信息。❓

❗️TODO：怎么配置，效果如何，待补充



#### 六）libraryTarget：导出方式 和 library：导出库名称

当用 Webpack 去构建一个可以被其他模块导入使用的库时需要用到它们

* `output.libraryTarget` 配置以何种方式导出库。
* `output.library` 配置导出库的名称。

`output.libraryTarget` 是字符串的枚举类型，支持以下七种配置（var、commonjs、commonjs2、this、window、global、libraryExport）。

* var（默认）

编写的库将通过var 被赋值给指定名称的变量，假如配置了 `output.library='LibraryName'`，

```js
// webpack 的配置
module.exports = {
    // ... 
    output: {
        // ...
        library: 'LibraryName',
        libraryTarget: 'var', // 默认也是 var
    }
}
// webpack输出的代码
var LibraryName = lib_code

// 使用库的方法
LibraryName.doSomething()
```

假如 `output.library` 为空，则将直接输出：

```js
lib_code
// 其中 lib_code 代指导出库的代码内容，是有返回值的一个自执行函数
```

* commonjs

编写的库将通过 CommonJS 规范导出，假如配置了 `output.library='LibraryName'`，则输出和使用的代码如下：

```js
// webpack 的配置
module.exports = {
    // ... 
    output: {
        // ...
        libraryTarget: 'commonjs', 
        library: 'LibraryName',
    }
}
// webpack输出的代码
var LibraryName = lib_code

// 使用库的方法，其中 library-name-in-npm 是指模块发布到 Npm 代码仓库时的名称
require('library-name-in-npm')['LibraryName'].doSomething()
```

* commonjs2

编写的库将通过 CommonJS2 规范导出，配置 `output.library` 将没有意义，输出和使用的代码如下：

```js
// webpack 的配置
module.exports = {
    // ... 
    output: {
        // ...
        libraryTarget: 'commonjs2', 
        library: 'LibraryName', // 该配置没有意义
    }
}
// webpack输出的代码
module.exports = lib_code
// 使用库的方法
require('library-name-in-npm').doSomething()
```

CommonJS2 和 CommonJS 规范很相似，差别在于 CommonJS 只能用 `exports` 导出，而 CommonJS2 在 CommonJS 的基础上增加了 `module.exports` 的导出方式

* this

编写的库将通过 `this` 被赋值给通过 `library` 指定的名称，输出和使用的代码如下：

```js
// Webpack 输出的代码
this['LibraryName'] = lib_code;

// 使用库的方法
this.LibraryName.doSomething();
```

* window

编写的库将通过 `window` 被赋值给通过 `library` 指定的名称，即把库挂载到 `window` 上，输出和使用的代码如下：

```js
// Webpack 输出的代码
window['LibraryName'] = lib_code;

// 使用库的方法
window.LibraryName.doSomething();
```

* global

编写的库将通过 `global` 被赋值给通过 `library` 指定的名称，即把库挂载到 `global` 上，输出和使用的代码如下：

```js
/ Webpack 输出的代码
global['LibraryName'] = lib_code;

// 使用库的方法
global.LibraryName.doSomething();
```

* libraryExport，❓（这块不是太理解，尤其是使用库的方法 === 1）

`output.libraryExport` 配置要导出的模块中哪些子模块需要被导出。 它只有在 `output.libraryTarget` 被设置成 `commonjs` 或者 `commonjs2` 时使用才有意义。

假如要导出的模块源代码是：

```js
export const a=1;
export default b=2;
```

现在你想让构建输出的代码只导出其中的 `a`，可以把 `output.libraryExport` 设置成 `a`，那么构建输出的代码和使用方法将变成如下：

```js
// Webpack 输出的代码
module.exports = lib_code['a'];

// 使用库的方法
require('library-name-in-npm')===1;
```



#### 整体的output配置

```js
module.exports = {
// 如何输出结果：在 Webpack 经过一系列处理后，如何输出最终想要的代码。
  output: {
    // 输出文件存放的目录，必须是 string 类型的绝对路径。
    path: path.resolve(__dirname, 'dist'),

    // 输出文件的名称
    filename: 'bundle.js', // 完整的名称
    filename: '[name].js', // 当配置了多个 entry 时，通过名称模版为不同的 entry 生成不同的文件名称
    filename: '[chunkhash].js', // 根据文件内容 hash 值生成文件名称，用于浏览器长时间缓存文件

    // 发布到线上的所有资源的 URL 前缀，string 类型
    publicPath: '/assets/', // 放到指定目录下
    publicPath: '', // 放到根目录下
    publicPath: 'https://cdn.example.com/', // 放到 CDN 上去

    // 导出库的名称，string 类型
    // 不填它时，默认输出格式是匿名的立即执行函数
    library: 'MyLibrary',

    // 导出库的类型，枚举类型，默认是 var
    // 可以是 umd | umd2 | commonjs2 | commonjs | amd | this | var | assign | window | global | jsonp ，
    libraryTarget: 'umd', 

    // 是否包含有用的文件路径信息到生成的代码里去，boolean 类型
    pathinfo: true, 

    // 附加 Chunk 的文件名称
    chunkFilename: '[id].js',
    chunkFilename: '[chunkhash].js',

    // JSONP 异步加载资源时的回调函数名称，需要和服务端搭配使用
    jsonpFunction: 'myWebpackJsonp',

    // 生成的 Source Map 文件名称
    sourceMapFilename: '[file].map',

    // 浏览器开发者工具里显示的源码模块名称
    devtoolModuleFilenameTemplate: 'webpack:///[resource-path]',

    // 异步加载跨域的资源时使用的方式
    crossOriginLoading: 'use-credentials',
    crossOriginLoading: 'anonymous',
    crossOriginLoading: false,
  },
} 
```







### 三，module

  配置如何处理模块

#### 一）配置 Loader

rules 配置模块的读取和解析规则，通常用来配置 Loader。其类型是一个数组，数组里每一项都描述了如何去处理部分文件。 配置一项 `rules` 时大致通过以下方式

* 条件匹配：通过 `test` 、 `include` 、 `exclude` 三个配置项来命中 Loader 要应用规则的文件；
* 应用规则：对选中后的文件通过 `use` 配置项来应用 Loader，可以只应用一个 Loader 或者按照从后往前的顺序应用一组 Loader，同时还可以分别给 Loader 传入参数；
* 重置顺序：一组 Loader 的执行顺序默认是从右到左执行，通过 `enforce` 选项可以让其中一个 Loader 的执行顺序放到最前或者最后

```js
module: {
  rules: [
    {
      // 命中 JavaScript 文件
      test: /\.js$/,
      // 用 babel-loader 转换 JavaScript 文件
      // ?cacheDirectory 表示传给 babel-loader 的参数，用于缓存 babel 编译结果加快重新编译速度
      use: ['babel-loader?cacheDirectory'],
      // 只命中src目录里的js文件，加快 Webpack 搜索速度
      include: path.resolve(__dirname, 'src')
    },
    {
      // 命中 SCSS 文件
      test: /\.scss$/,
      // 使用一组 Loader 去处理 SCSS 文件。
      // 处理顺序为从后到前，即先交给 sass-loader 处理，再把结果交给 css-loader 最后再给 style-loader。
      use: ['style-loader', 'css-loader', 'sass-loader'],
      // 排除 node_modules 目录下的文件
      exclude: path.resolve(__dirname, 'node_modules'),
    },
    {
      // 对非文本文件采用 file-loader 加载
      test: /\.(gif|png|jpe?g|eot|woff|ttf|svg|pdf)$/,
      use: ['file-loader'],
    },
  ]
}
```

在 Loader 需要传入很多参数时，还可以通过一个 Object 来描述，如：

```js
use: [ // 即 use的数组组成除了 loader的字符串，还可以是 对象
  {
    loader:'babel-loader',
    options:{
      cacheDirectory:true,
    },
    // enforce:'post' 的含义是把该 Loader 的执行顺序放到最后
    // enforce 的值还可以是 pre，代表把 Loader 的执行顺序放到最前面
    enforce:'post'
  },
  // 省略其它 Loader
]
```

条件配置关键字`test include exclude` 这三个命中文件的配置项除了一个字符串和正则，还支持数组类型，如下：

```js
{
  test:[ // 数组里的每项之间是或的关系，即文件路径符合数组中的任何一个条件就会被命中。
    /\.jsx?$/,
    /\.tsx?$/
  ],
  include:[ // 数组里的每项之间是或的关系，即文件路径符合数组中的任何一个条件就会被命中。
    path.resolve(__dirname, 'src'),
    path.resolve(__dirname, 'tests'),
  ],
  exclude:[ // 数组里的每项之间是或的关系，即文件路径符合数组中的任何一个条件就会被命中。
    path.resolve(__dirname, 'node_modules'),
    path.resolve(__dirname, 'bower_modules'),
  ]
}
```



#### 二）noParse：忽略非模块化文件

`noParse` 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析和处理，这样做的好处是能提高构建性能。 原因是一些库例如 jQuery 、ChartJS 它们庞大又没有采用模块化标准，让 Webpack 去解析这些文件耗时又没有意义。

`noParse` 是可选配置项，类型需要是 `RegExp`、`[RegExp]`、`function` 其中一个。

以忽略jQuery、ChartJS为例，具体使用方法为：

```js
// 使用正则表达式
noParse: /jquery|chartjs/

// 使用函数，从 Webpack 3.0.0 开始支持
noParse: (content)=> {
  // content 代表一个模块的文件路径
  // 返回 true or false
  return /jquery|chartjs/.test(content);
}
```

注意被忽略掉的文件里不应该包含 `import` 、 `require` 、 `define` 等模块化语句，不然会导致构建出的代码中包含无法在浏览器环境下执行的模块化语句。

换句话说：采用模块化的库，是支持webpack构建的，尽可能通过webpack进行构建



#### 三）parser

因为 Webpack 是以模块化的 JavaScript 文件为入口，所以内置了对模块化 JavaScript 的解析功能，支持 AMD、CommonJS、SystemJS、ES6。`parser` 属性可以更细粒度的配置哪些模块语法要解析哪些不解析，和 `noParse` 配置项的区别在于 `parser` 可以精确到语法层面， 而 `noParse` 只能控制哪些文件不被解析。 `parser` 使用如下：

```js
module: {
  rules: [
    {
      test: /\.js$/,
      use: ['babel-loader'],
      parser: {
      amd: false, // 禁用 AMD
      commonjs: false, // 禁用 CommonJS
      system: false, // 禁用 SystemJS
      harmony: false, // 禁用 ES6 import/export
      requireInclude: false, // 禁用 require.include
      requireEnsure: false, // 禁用 require.ensure
      requireContext: false, // 禁用 require.context
      browserify: false, // 禁用 browserify
      requireJs: false, // 禁用 requirejs
      }
    },
  ]
}
```



#### 整体的module的配置

```js
module.exports = {
// 配置模块相关
  module: {
    rules: [ // 配置 Loader
      {  
        test: /\.jsx?$/, // 正则匹配命中要使用 Loader 的文件
        include: [ // 只会命中这里面的文件
          path.resolve(__dirname, 'app')
        ],
        exclude: [ // 忽略这里面的文件
          path.resolve(__dirname, 'app/demo-files')
        ],
        use: [ // 使用那些 Loader，有先后次序，从后往前执行
          'style-loader', // 直接使用 Loader 的名称
          {
            loader: 'css-loader',      
            options: { // 给 html-loader 传一些参数
            }
          }
        ]
      },
    ],
    noParse: [ // 不用解析和处理的模块
      /special-library\.js$/  // 用正则匹配
    ],
  },
}
```



### 四，resolve

Webpack 在启动后会从配置的入口模块出发找出所有依赖的模块，Resolve 配置 Webpack 如何寻找模块所对应的文件。 Webpack 内置 JavaScript 模块化语法解析功能，默认会采用模块化标准里约定好的规则去寻找，但也可以根据自己的需要修改默认的规则。

#### 一）alias：别名

`resolve.alias` 配置项通过别名来把原导入路径映射成一个新的导入路径，如：

```js
module.exports = {
    entry: './main.js',
    // ...
    resolve: {
        alias:{
            // 把导入语句里的 components 关键字替换成 ./src/components/
            components: './src/components/'
        }
    }
}
```

当通过 `import Button from 'components/button'` 导入时，实际上被 `alias` 等价替换成了 `import Button from './src/components/button'`。

还可以通过 `$` 符号来缩小范围到只命中以关键字结尾的导入语句，如：

```js
resolve:{
  alias:{
    // react$ 只会命中以 react 结尾的导入语句
    'react$': '/path/to/react.min.js'
  }
}
```

只会把 `import 'react'` 关键字替换成 `import '/path/to/react.min.js'`。

#### 二）mainFields

有一些第三方模块会针对不同环境提供几分代码。 例如分别提供采用 ES5 和 ES6 的2份代码，这2份代码的位置写在 `package.json` 文件里，如下：

```json
{
  "jsnext:main": "es/index.js",// 采用 ES6 语法的代码入口文件
  "main": "lib/index.js" // 采用 ES5 语法的代码入口文件
}
```

Webpack 会根据 `mainFields` 的配置去决定优先采用那份代码（Webpack 会按照数组里的顺序去`package.json` 文件里寻找，只会使用找到的第一个），`mainFields` 默认是`mainFields: ['browser', 'main']`，假如你想优先采用 ES6 的那份代码，可以这样配置：

```js
mainFields: ['jsnext:main', 'browser', 'main']
```

#### 三）extensions & enforceExtension & enforceModuleExtension

* extensions

用于在导入语句没带文件后缀时，Webpack 会自动带上后缀后去尝试访问文件是否存在。`resolve.extensions` 用于配置在尝试过程中用到的后缀列表，默认是：

```js
module.exports = {
    resolve:{
        extensions:['.js', '.json']
    }
}
```

也就是说当遇到 `require('./data')` 这样的导入语句时，Webpack 会先去寻找 `./data.js` 文件，如果该文件不存在就去寻找 `./data.json` 文件， 如果还是找不到就报错。

假如想让 Webpack 优先使用目录下的 TypeScript 文件，可以这样配置：

```js
module.exports = {
    resolve:{
        extensions:['.ts','.js', '.json']
    }
}
```

* enforceExtension

`resolve.enforceExtension` 如果配置为 `true` 所有导入语句都必须要带文件后缀， 例如开启前 `import './foo'` 能正常工作，开启后就必须写成 `import './foo.js'`。

* enforceModuleExtension

`enforceModuleExtension` 和 `enforceExtension` 作用类似，但 `enforceModuleExtension` 只对 `node_modules` 下的模块生效。 `enforceModuleExtension` 通常搭配 `enforceExtension` 使用，在 `enforceExtension:true` 时，因为安装的第三方模块中大多数导入语句没带文件后缀， 所以这时通过配置 `enforceModuleExtension:false` 来兼容第三方模块。

#### 四）modules

`resolve.modules` 配置 Webpack 去哪些目录下寻找第三方模块，默认是只会去 `node_modules` 目录下寻找。

有时你的项目里会有一些模块会大量被其它模块依赖和导入，由于其它模块的位置分布不定，针对不同的文件都要去计算被导入模块文件的相对路径， 这个路径有时候会很长，就像这样 `import '../../../components/button'` 这时你可以利用 `modules` 配置项优化，假如那些被大量导入的模块都在 `./src/components` 目录下，把 `modules` 配置成：

```js
module.exports = {
    resolve:{
        modules:['./src/components','node_modules']
    }
}
```

之后就可以简单通过 `import 'button'` 导入。

#### 五）descriptionFiles

`resolve.descriptionFiles` 配置描述第三方模块的文件名称，也就是 `package.json` 文件。默认如下：

```js
descriptionFiles: ['package.json']
```

应用场景：❓



#### 整体的resolve配置

```js
module.exports = {
// 配置寻找模块的规则
  resolve: { 
    modules: [ // 寻找模块的根目录，array 类型，默认以 node_modules 为根目录
      'node_modules',
      path.resolve(__dirname, 'app')
    ],
    extensions: ['.js', '.json', '.jsx', '.css'], // 模块的后缀名
    alias: { // 模块别名配置，用于映射模块
       // 把 'module' 映射 'new-module'，同样的 'module/path/file' 也会被映射成 'new-module/path/file'
      'module': 'new-module',
      // 使用结尾符号 $ 后，把 'only-module' 映射成 'new-module'，
      // 但是不像上面的，'module/path/file' 不会被映射成 'new-module/path/file'
      'only-module$': 'new-module', 
    },
    alias: [ // alias 还支持使用数组来更详细的配置
      {
        name: 'module', // 老的模块
        alias: 'new-module', // 新的模块
        // 是否是只映射模块，如果是 true 只有 'module' 会被映射，如果是 false 'module/inner/path' 也会被映射
        onlyModule: true, 
      }
    ],
    symlinks: true, // 是否跟随文件软链接去搜寻模块的路径
    descriptionFiles: ['package.json'], // 模块的描述文件
    mainFields: ['main'], // 模块的描述文件里的描述入口的文件的字段名称
    enforceExtension: false, // 是否强制导入语句必须要写明文件后缀
  },
}
```



### 五，plugin

Plugin 用于扩展 Webpack 功能，各种各样的 Plugin 几乎让 Webpack 可以做任何构建相关的事情。

#### 一）配置plugin

`plugins` 配置项接受一个数组，数组里每一项都是一个要使用的 Plugin 的实例，Plugin 需要的参数通过构造函数传入。

```js
const CommonsChunkPlugin = require('webpack/lib/optimize/CommonsChunkPlugin');

module.exports = {
    plugins: [
        // 所有页面都会用到的公共代码提取到 common 代码块中
        new CommonsChunkPlugin({
            name:'common',
            chunks:['a','b']
        })
    ]
}
```

### 六，devServer

DevServer 提供了一些配置项可以改变 DevServer 的默认行为。注意只有在通过 DevServer 去启动 Webpack 时配置文件里 `devServer` 才会生效，因为这些参数所对应的功能都是 DevServer 提供的，Webpack 本身并不认识 `devServer` 配置项。

#### 一）hot：模块热替换

`devServer.hot` 配置是否启用模块热替换功能。 DevServer 默认的行为是在发现源代码被更新后会通过自动刷新整个页面来做到实时预览，开启模块热替换功能后将在不刷新整个页面的情况下通过用新模块替换老模块来做到实时预览。

#### 二）inline

DevServer 的实时预览功能依赖一个注入到页面里的代理客户端去接受来自 DevServer 的命令和负责刷新网页的工作。 `devServer.inline` 用于配置是否自动注入这个代理客户端到将运行在页面里的 Chunk 里去，默认是会自动注入。 DevServer 会根据你是否开启 `inline` 来调整它的自动刷新策略：

* 开启 `inline`，DevServer 会在构建完变化后的代码时通过代理客户端控制网页刷新。
* 关闭 `inline`，DevServer 将无法直接控制要开发的网页。这时它会通过 iframe 的方式去运行要开发的网页，当构建完变化后的代码时通过刷新 iframe 来实现实时预览。 但这时你需要去 `http://localhost:8080/webpack-dev-server/` 实时预览你的网页了。

如果你想使用 DevServer 去自动刷新网页实现实时预览，最方便的方法是直接开启 `inline`。

#### 三）historyApiFallback

`devServer.historyApiFallback` 用于方便开发使用了H5 history API的单页面应用。这类单页应用要求服务器在针对任何命中的路由时都返回一个对应的 HTML 文件，例如在访问 `http://localhost/user` 和 `http://localhost/home` 时都返回 `index.html` 文件， 浏览器端的 JavaScript 代码会从 URL 里解析出当前页面的状态，显示出对应的界面。

historyApiFallback最简单的配置：

```js
module.exports = {
    // ... 
    devServer:{
        // 这会导致任何请求都会返回 index.html 文件，这只能用于只有一个 HTML 文件的应用。
        historyApiFallback: true
    }
}
```

如果应用由多个单页应用组成，这就需要 DevServer 根据不同的请求来返回不同的 HTML 文件，配置如下：

```js
module.exports = {
    // ...
    devServer: {
        historyApiFallback: {
            // 使用正则匹配命中路由
            rewrites: [
                // /user 开头的都返回 user.html
                { from: /^\/user/, to: '/user.html' },
                { from: /^\/game/, to: '/game.html' },
                // 其它的都返回 index.html
                { from: /./, to: '/index.html' },
            ]
        }
    }
}
```



#### 四）contentBase

`devServer.contentBase` 配置 DevServer HTTP 服务器的文件根目录。 默认情况下为当前执行目录，通常是项目根目录，所有一般情况下你不必设置它，除非你有额外的文件需要被 DevServer 服务。例如你想把项目根目录下的 `public` 目录设置成 DevServer 服务器的文件根目录，你可以这样配置：

```js
module.exports = {
    // ...
    devServer:{
        contentBase:path.join(__dirname, 'public')
    }
}
```

DevServer 服务器通过 HTTP 服务暴露出的文件分为两类：

* 暴露本地文件
* 暴露 Webpack 构建出的结果，由于构建出的结果交给了 DevServer，所以你在使用了 DevServer 时在本地找不到构建出的文件

`contentBase` 只能用来配置暴露本地文件的规则，你可以通过 `contentBase:false` 来关闭暴露本地文件。

#### 五）headers：响应头

`devServer.headers` 配置项可以在 HTTP 响应中注入一些 HTTP 响应头，使用如下：

```js
devServer:{
  headers: {
    'X-foo':'bar'
  }
}
```

#### 六）host：主机 & disableHostCheck

`devServer.host` 配置项用于配置 DevServer 服务监听的地址。 例如你想要局域网中的其它设备访问你本地的服务，可以在启动 DevServer 时带上 `--host 0.0.0.0`。 `host` 的默认值是 `127.0.0.1` 即只有本地可以访问 DevServer 的 HTTP 服务。

补充：在服务器中，0.0.0.0指的是本机上的所有IPV4地址，[127.0.0.1和0.0.0.0地址的区别](https://zhuanlan.zhihu.com/p/72988255)

`devServer.disableHostCheck` 配置项用于配置是否关闭用于 DNS 重绑定的 HTTP 请求的 HOST 检查。 DevServer 默认只接受来自本地的请求，关闭后可以接受来自任何 HOST 的请求。 它通常用于搭配 `--host 0.0.0.0` 使用，因为你想要其它设备访问你本地的服务，但访问时是直接通过 IP 地址访问而不是 HOST 访问，所以需要关闭 HOST 检查。

#### 七）port：端口

`devServer.port` 配置项用于配置 DevServer 服务监听的端口，默认使用 8080 端口。若8080端口被占用，则使用8081

#### 八）allowedHosts：白名单

`devServer.allowedHosts` 配置一个白名单列表，只有 HTTP 请求的 HOST 在列表里才正常返回，使用如下：

```js
module.exports = {
    // ...
    devServer: {
        allowedHosts: [
            // 匹配单个域名
            'host.com',
            'sub.host.com',
            // host2.com 和所有的子域名 *.host2.com 都将匹配
            '.host2.com'
        ]
    }
}
```

#### 九）https

DevServer 默认使用 HTTP 协议服务，它也能通过 HTTPS 协议服务。 有些情况下你必须使用 HTTPS，例如 HTTP2 和 Service Worker 就必须运行在 HTTPS 之上。 要切换成 HTTPS 服务，最简单的方式是：

```js
devServer:{
  https: true
}
```

DevServer 会自动的为你生成一份 HTTPS 证书。想用自己的证书可以这样配置：

```js
devServer:{
  https: {
    key: fs.readFileSync('path/to/server.key'),
    cert: fs.readFileSync('path/to/server.crt'),
    ca: fs.readFileSync('path/to/ca.pem')
  }
}
```

#### 十）clientLogLevel：客户端日志等级

`devServer.clientLogLevel` 配置在客户端的日志等级，这会影响到你在浏览器开发者工具控制台里看到的日志内容。 `clientLogLevel` 是枚举类型，可取如下之一的值 `none | error | warning | info`。 默认为 `info` 级别，即输出所有类型的日志，设置成 `none` 可以不输出任何日志。

#### 十一）compress：gzip压缩

`devServer.compress` 配置是否启用 gzip 压缩。`boolean` 为类型，默认为 `false`。

#### 十二）open

`devServer.open` 用于在 DevServer 启动且第一次构建完时自动用你系统上默认的浏览器去打开要开发的网页。 同时还提供 `devServer.openPage` 配置项用于打开指定 URL 的网页。



#### devServer的整体配置

```json
module.exports = {
  devServer: { // DevServer 相关的配置
      proxy: { // 代理到后端服务接口
        '/api': 'http://localhost:3000'
      },
      contentBase: path.join(__dirname, 'public'), // 配置 DevServer HTTP 服务器的文件根目录
      compress: true, // 是否开启 gzip 压缩
      historyApiFallback: true, // 是否开发 HTML5 History API 网页
      hot: true, // 是否开启模块热替换功能
      https: false, // 是否开启 HTTPS 模式
      },

      profile: true, // 是否捕捉 Webpack 构建的性能信息，用于分析什么原因导致构建性能不佳

      cache: false, // 是否启用缓存提升构建速度

      watch: true, // 是否开始
      watchOptions: { // 监听模式选项
      // 不监听的文件或文件夹，支持正则匹配。默认为空
      ignored: /node_modules/,
      // 监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高
      // 默认为300ms 
      aggregateTimeout: 300,
      // 判断文件是否发生变化是不停的去询问系统指定文件有没有变化，默认每隔1000毫秒询问一次
      poll: 1000
    },
}
```



### 七，其他配置项

除前面所介绍的外，还有一些零散的配置项，包括：target，devtool，watch 和 watchOptions，externals，resolveLoader

#### 一）target

从浏览器到 Node.js，这些运行在不同环境的 JavaScript 代码存在一些差异。 `target` 配置项可以让 Webpack 构建出针对不同运行环境的代码。

| target值          | 描述                                              |
| ----------------- | ------------------------------------------------- |
| Web               | 针对浏览器 **(默认)**，所有代码都集中在一个文件里 |
| node              | 针对 Node.js，使用 `require` 语句加载 Chunk 代码  |
| async-node        | 针对 Node.js，异步加载 Chunk 代码                 |
| webworker         | 针对 WebWorker                                    |
| electron-main     | 针对 [Electron](http://electron.atom.io/) 主线程  |
| electron-renderer | 针对 Electron 渲染线程                            |

当设置 `target:'node'` 时，源代码中导入 Node.js 原生模块的语句 `require('fs')` 将会被保留，`fs` 模块的内容不会打包进 Chunk 里。

#### 二）devtool：source map

`devtool` 配置 Webpack 如何生成 Source Map，默认值是 `false` 即不生成 Source Map，想为构建出的代码生成 Source Map 以方便调试，可以这样配置：

```js
module.exports = {
    devtool:'source-map'
}
```

#### 三）watch & watchOptions：监听

Webpack支持监听文件更新，在文件发生变化时重新编译。在使用 Webpack 时监听模式默认是关闭的，想打开需要如下配置：

```js
module.exports = {
    watch: true
}
```

注：使用 DevServer 时，监听模式默认是开启的

Webpack 还提供了 `watchOptions` 配置项去更灵活的控制监听模式，使用如下：

```js
module.exports = {
  // 只有在开启监听模式时，watchOptions 才有意义
  // 默认为 false，也就是不开启
  watch: true,
  // 监听模式运行时的参数
  // 在开启监听模式时，才有意义
  watchOptions: {
    // 不监听的文件或文件夹，支持正则匹配
    // 默认为空
    ignored: /node_modules/,
    // 监听到变化发生后会等300ms再去执行动作，防止文件更新太快导致重新编译频率太高
    // 默认为 300ms  
    aggregateTimeout: 300,
    // 判断文件是否发生变化是通过不停的去询问系统指定文件有没有变化实现的
    // 默认每隔1000毫秒询问一次
    poll: 1000
  }
}
```

#### 四）Externals：外部模块，忽略构建

Externals 用来告诉 Webpack 要构建的代码中使用了哪些不用被打包的模块，也即这些模版是外部环境提供的，Webpack 在打包时可以忽略它们。

有些 JavaScript 运行环境可能内置了一些全局变量或者模块，如在HTML HEAD 标签里通过代码`<script src="path/to/jquery.js"></script>`引入 jQuery 后，全局变量 `jQuery` 就会被注入到网页的 JavaScript 运行环境里。

如果想在使用模块化的源代码里导入和使用 jQuery，可能需要这样：

```js
import $ from 'jquery';
$('.my-element');
```

构建后你会发现输出的 Chunk 里包含的 jQuery 库的内容，这导致 jQuery 库出现了2次，浪费加载流量，最好是 Chunk 里不会包含 jQuery 库的内容。

Externals 配置项就是为了解决这个问题。

通过 `externals` 可以告诉 Webpack JavaScript 运行环境已经内置了那些全局变量，针对这些全局变量不用打包进代码中而是直接使用全局变量。 要解决以上问题，可以这样配置 `externals`：

```js
module.export = {
  externals: {
    // 把导入语句里的 jquery 替换成运行环境里的全局变量 jQuery
    jquery: 'jQuery'
  }
}
```



#### 五）ResolveLoader

ResolveLoader 用来告诉 Webpack 如何去寻找 Loader，因为在使用 Loader 时是通过其包名称去引用的， Webpack 需要根据配置的 Loader 包名去找到 Loader 的实际代码，以调用 Loader 去处理源文件。

ResolveLoader 的默认配置如下：

```js
module.exports = {
    resolveLoader: {
        // 去哪个目录下寻找 Loader
        modules: ['node_modules'],
        // 入口文件的后缀
        extensions: ['.js', '.json'],
        // 指明入口文件位置的字段
        mainFields: ['loader', 'main']
    }
}
```

该配置项常用于加载本地的 Loader。



#### 其他配置示例

```js
module.exports = {
	// 输出文件性能检查配置
  performance: { 
    hints: 'warning', // 有性能问题时输出警告
    hints: 'error', // 有性能问题时输出错误
    hints: false, // 关闭性能检查
    maxAssetSize: 200000, // 最大文件大小 (单位 bytes)
    maxEntrypointSize: 400000, // 最大入口文件大小 (单位 bytes)
    assetFilter: function(assetFilename) { // 过滤要检查的文件
      return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
  },

  devtool: 'source-map', // 配置 source-map 类型

  // 配置输出代码的运行环境
  target: 'web', // 浏览器，默认
  target: 'webworker', // WebWorker
  target: 'node', // Node.js，使用 `require` 语句加载 Chunk 代码
  target: 'async-node', // Node.js，异步加载 Chunk 代码
  target: 'node-webkit', // nw.js
  target: 'electron-main', // electron, 主线程
  target: 'electron-renderer', // electron, 渲染线程

  externals: { // 使用来自 JavaScript 运行环境提供的全局变量
    jquery: 'jQuery'
  },

  stats: { // 控制台输出日志控制
    assets: true,
    colors: true,
    errors: true,
    errorDetails: true,
    hash: true,
  },
}
```



### 八，多种配置类型

除了导出一个object 描述 webpack所需配置外，还可以

#### 导出一个 Function



#### 导出一个返回 Promise 的函数

```js
module.exports = function(env = {}, argv) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({
        // ...
      })
    }, 5000)
  })
}
```



#### 导出多份配置

除了只导出一份配置外，Webpack 还支持导出一个数组，数组中可以包含每份配置，并且每份配置都会执行一遍构建。

```js
module.exports = [
  // 采用 Object 描述的一份配置
  {
    // ...
  },
  // 采用函数描述的一份配置
  function() {
    return {
      // ...
    }
  },
  // 采用异步函数描述的一份配置
  function() {
    return Promise();
  }
]
```

以上配置会导致 Webpack 针对这三份配置执行三次不同的构建。

这特别适合于用 Webpack 构建一个要上传到 Npm 仓库的库，因为库中可能需要包含多种模块化格式的代码，例如 CommonJS、UMD。



### 九，总结

通常可用如下经验去判断如何配置 Webpack：

- 想让源文件加入到构建流程中去被 Webpack 控制，配置 `entry`。
- 想自定义输出文件的位置和名称，配置 `output`。
- 想自定义寻找依赖模块时的策略，配置 `resolve`。
- 想自定义解析和转换文件的策略，配置 `module`，通常是配置 `module.rules` 里的 Loader。
- 其它的大部分需求可能要通过 Plugin 去实现，配置 `plugin`。





