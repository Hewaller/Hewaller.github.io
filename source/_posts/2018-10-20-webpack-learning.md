---
layout: post
title: webpack前进之路
date: 2018-11-06
categories: webpack
tags:
  - webpack
  - JavaScript
  -
---

webpack 学习笔记
最后更改日期：2019 年 02 月 23 日

<!-- more -->

`webpack` 学习笔记，参考了官网和很多的文章，由于当时没有及时记录，参考的文章就不一一列出了

# webpack 学习之路

## css

Webpack 把所有的文件都都当做模块处理，JavaScript 代码，CSS 和 fonts 以及图片等等通过合适的 loader 都可以被处理。
css 有两个工具处理样式表，`css-loader` 和 `style-loader`, `css-loader`能够使用类似`@import` 和 `url(...)`的方法实现`require()`的功能，`style-loader`将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入 webpack 打包后的 JS 文件中。

```js
// 简单的css打包配置
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        {
          loader: 'style-loader'
        },
        {
          loader: 'css-loader',
          options: {
            modules: true, // 指定启用css modules
            localIdentName: '[name]__[local]--[hash:base64:5]' // 指定css的类名格式,防止全局类名的污染
          }
        }
      ]
    }
  ]
}
```

**css 与处理器**
css 处理 loaders:

- `Less loader`
- `Sass loader`
- `Stylus loader`

还有一个更大的处理平台`PostCSS`， 使用  如下

```js
rules: [
  {
    test: /\.css$/,
    use: [
      {
        loader: 'style-loader'
      },
      {
        loader: 'css-loader',
        options: {
          modules: true // 指定启用css modules
        }
      },
      {
        loader: 'postcss-loader'
      }
    ]
  }
]
```

## babel

将 ES6 等高版本的 JS 转义成浏览器和 node 环境下可以被成功运行的 js 代码

```js
$ npm install babel-cli -g //全局安装
$ cd .....
$ pwd（li） // 看一下当前的环境/文件下包含的目录（防止进错目录了，最好确认一下比较稳妥）
$ npm init // 初始化项目
$ npm install babel-cli --save-dev //本地安装（当前文件夹或者环境中安装）
```

## babel 环境设置

```js
{
    'presets': ['es2015'], //预置配置， 项目的依赖和最后输出的结果
    'plugins': []  //插件
}
$ -o // --out-file 输出单文件
$ -d // --out-dir 输出文件夹
$ bable --watch src -d build //编译后热更新
$ ctrl + c //终止服务
```

## browser-sync

简单的浏览器同步测试工具（起一个服务）

```js
$ browser-sync start --server
$ browser-sync start --help  // 了解该方法下有哪些具体的用法
```

## browserify 打包

node 在网页上三种 js 规范： commonjs \ amd \cmd

1. `commonjs`: 基于后台的规范（系统架构、系统自动化），模块化定义，允许把 js 代码抽离出来，形成一个单独的文件。特点：同步，加载完成后，但是在浏览器中会导致浏览器假死的状况（网速慢，资源不存在等）
2. `amd`:异步加载，成功和失败都有一个返回值，加载 css 后再引入模块
3. `cmd`:异步加载-按需加载

### webpack 核心概念

本质上： 静态模块打包器
核心概念：

- 入口（entry）：内部依赖图的开始，也可以说就是需要被打包的目录
- 输出（output）： 输出打包完成的文件，默认文件为 `./dist`, 主输出文件默认为 `./dist/main.js`
- loader: 够让 webpack 处理那些非 JavaScript 文件，并且先将它们转换为有效 模块，然后添加到依赖图中
- 插件（plugins）: 打包优化、资源管理和注入环境变量
- 模式： dev、production 和 none

### 入口

用法： `entry: string| Array<string>`
使用场景：

1. 分离应用程序(app)和第三方库（vendor）入口

```js
 entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
```

2.多页面应用：设置多个页面入口（但是我们目前的项目都是单页面入口）

> 对零配置来说，默认入口文件夹为当前工程的根目录 `src` 文件夹

```js
webpackProject
│
└───src
│   │--index.js  //默认的入口文件
```

### 输出（output）

注意点： 虽然可以存在多个 `入口` 但是只能指定一个 `输出` 配置，输出 `path` 必须是一个绝对路径，它是 [Node.js](http://nodejs.cn/api/path.html) 的一个核心模块。称之为路径模块，用于操作文件路径。

```js
   // 简单用法
   output: {
   filename: 'bundle.js', //[name].js
   path: '/home/proj/public/assets' // _dirname + './dist'
 }
 // 将单独的 `bundle.js` 问价输出到指定目录中
```

> 零配置默认的输出 v

```js
webpackProject
│
└───dist
|   |--main.js
 //读取 src 目录下的 index.js文件，打包后默认输出到同级的 dist 目录下，文件名为 main.js
```

### 生成 Source Maps 调试

在 webpack 的配置文件中配置 source maps，需要配置 `devtool`，它有四种不同的配置选项:

1. source-map 生成一个完整且功能完全的文件，不压缩
2. cheap-module-source-map 在一个单独的文件中生成一个不带  映射的 map，调试不能对应到具体的列
3. eval-source-map 使用 eval 打包源文件模块，在同一个文件中生成干净的完整的 `source map`， 在开发阶段这是一个非常好的选项，在生产阶段则一定不要启用这个选项
4. cheap-module-eval-source-map 这是在打包文件时最快的生成 source map 的方法，生成的 Source Map 会和打包后的 JavaScript 文件同行显示，没有列映射

### 热更新

```js
npm install --save-dev webpack-dev-server
```

> 配置

```js
devServer: {
    contentBase: "./dist",//本地服务器所加载的页面所在的目录
    historyApiFallback: true,//不跳转
    inline: true,  //实时刷新
    port: 8080   // 端口号： 默认8080
  }
```

### loaders

主要用于对模块的源代码进行转换。

> 在 webpack 打包中,会对入口文件的直接依赖或者间接依赖进行解析,但是 webpack 只能解析某些文件(比如 `.js` 结尾的文件)在 4.0 之后 `.txt` 、 `.json` 文件都能支持解析。而 `loader` 做的事情就是把其它类型的所有文件转换为 `webpack` 能够处理的模块。然后你就可以利用 `webpack` 的打包能力，对它们进行处理。

**loaders 的配置具体包括这几个方面**

- `test`: 一个用以匹配 loaders 所处理文件的拓展名的正则表达式（必须）
- `loader`: loader 的名称
- `include/exclude`: 手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
- `query`: 为 loaders 提供额外的设置选项（可选）

推荐用法： 在`weboack.confing.js` 中配置指定 loader， `module.rules` 允许在配置中指定多个 loader。

```js
module: {
  rules: [
    { test: /\.css$/, use: 'css-loader' },
    {
      test: /\.css$/,
      use: [
        { loader: 'style-loader' },
        {
          loader: 'css-loader',
          options: {
            modules: true
          }
        }
      ]
    }
  ]
}
```

> 简单的使用方法： 在项目中添加了一个图片

```js
webpackProject
│
└───src
|   |--image.jpeg  添加一个图片
│   │--util.js

// util.js 文件
import myImage from  './image.jpeg'

const img = new Image()
img.src = myImage
....

```

> 因为 `webpack` 并不支持对图片的解析,需要配置相应的 loader 做转换处理为 `webpack` 打包的文件格式, webpack 能够解析任何 import 导入的模块，比如.css, .jpg 文件等。

```js
const config = {
  module: {
    rules: [
      {test: /\.jpeg$/, use: 'url-loader'}
      //所有.jpg格式的图片都会被转化, use: 解析模块对应的解析器(loader)
  }
}
```

### 插件(plugins)

插件目的在于解决 `loader` 无法实现的其他事。

> loader: 解析默写特定的文件， plugins： 几乎所有的自动化操作都通过插件来完成

由于插件可以携带参数/选项，必须在 `webpack` 配置中，向 `plugins` 属性传入 `new` 实例。

```js
plugins: [
  new webpack.optimize.UglifyJsPlugin(),
  new HtmlWebpackPlugin({ template: './src/index.html' }) //自动引用你打包后的JS文件的新index.html
  new webpack.HotModuleReplacementPlugin()//热加载插件
]
```

### 配置(configuration)

> plugin 和 loader 的区别是，loader 是在 import 时根据不同的文件名，匹配不同的 loader 对这个文件做处理，而 plugin, 关注的不是文件的格式，而是在编译的各个阶段，会触发不同的事件，让你可以干预每个编译阶段。

webpack 的配置文件，是导出一个对象的 JavaScript 文件。此对象，由 webpack 根据对象定义的属性进行解析。
如果 webpack.config.js 存在，则 webpack 命令将默认选择使用它。但是在生产环境和开发环境中所需要配置的 webpack 是不一致的，所以现在将 webpack 的配置更改为一个公用文件和两个环境文件，`webpack.common.js`、`webpack.dev.js`和 `webpack.prod.js`，使用`webpack-merge`将不同的配置结合在一起。

```js
const merge = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    contentBase: './dist'
  }
})
```

### 模块(modules)

## 项目常用配置

- 设置静态资源的 url 路径前缀
- 各个页面分开打包
- 配置 favicon
- 代码中插入环境变量
- 简化 import 路径
- 使用 postcss 的插件 autoprefixer 自动创建 css 的 vendor prefixes

### 设置静态资源的 URL 路径前缀

> 一般我们的项目开发环境中，我们的资源文件都是在根目录中，8080/index.js，但是在生产环境中，需要考虑到缓存控制和 CDN，一版会给 url 加一个前缀： 8080/m/index.js

```js
{
  output: {
    publicPath: '/m/'
  }
}

// webpack配置

{
  output: {
    /*
    代码中引用的文件（js、css、图片等）会根据配置合并为一个或多个包，我们称一个包为 chunk。
    每个 chunk 包含多个 modules。无论是否是 js，webpack 都将引入的文件视为一个 module。
    chunkFilename 用来配置这个 chunk 输出的文件名。

    [chunkhash]：这个 chunk 的 hash 值，文件发生变化时该值也会变。使用 [chunkhash] 作为文件名可以防止浏览器读取旧的缓存文件。

    还有一个占位符 [id]，编译时每个 chunk 会有一个id。
    我们在这里不使用它，因为这个 id 是个递增的数字，增加或减少一个chunk，都可能导致其他 chunk 的 id 发生改变，导致缓存失效。
    */
    chunkFilename: '[id].[chunkhash].js',
  }
}
```

### 各个页面分开打包

应用路由时可以按需加载，使用 async/await 配合`import`使用

```js
import Test = () => import('@/views/test')
```

### 配置 favicon

> 网页图标： 在 src 目录中放一张 favicon.png，然后 src/index.html 中插入

```html
<link rel="icon" type="image/png" href="favicon.png" />
```

webpack 配置：

```js
{
  module: {
    rules: [
      {
        test: /\.html$/,
        use: [
          {
            loader: 'html-loader',
            options: {
              /*
              html-loader 接受 attrs 参数，表示什么标签的什么属性需要调用 webpack 的 loader 进行打包。
              比如 <img> 标签的 src 属性，webpack 会把 <img> 引用的图片打包，然后 src 的属性值替换为打包后的路径。
              使用什么 loader 代码，同样是在 module.rules 定义中使用匹配的规则。
              如果 html-loader 不指定 attrs 参数，默认值是 img:src, 意味着会默认打包 <img> 标签的图片。
              这里我们加上 <link> 标签的 href 属性，用来打包入口 index.html 引入的 favicon.png 文件。
              */
              attrs: ['img:src', 'link:href']
            }
          }
        ]
      },

      {
        /*
        匹配 favicon.png
        上面的 html-loader 会把入口 index.html 引用的 favicon.png 图标文件解析出来进行打包
        打包规则就按照这里指定的 loader 执行
        */
        test: /favicon\.png$/,

        use: [
          {
            // 使用 file-loader
            loader: 'file-loader',
            options: {
              /*
              name：指定文件输出名
              [hash] 为源文件的hash值，[ext] 为后缀。
              */
              name: '[hash].[ext]'
            }
          }
        ]
      },

      // 图片文件的加载配置增加一个 exclude 参数
      {
        test: /\.(png|jpg|jpeg|gif|eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,

        // 排除 favicon.png, 因为它已经由上面的 loader 处理了。如果不排除掉，它会被这个 loader 再处理一遍
        exclude: /favicon\.png$/,

        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10000
            }
          }
        ]
      }
    ]
  }
}
```
