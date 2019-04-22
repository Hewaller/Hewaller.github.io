---
layout: post
title: require和import
date: 2018-11-07
categories: ES6
tags:
  - JavaScript
  - 前端
  - ES6
---

`require` 和 `import`
2018 年 11 月 07 日

<!-- more  -->

# 模块化

> 定义： 模块是自动运行在严格模式下并且没有那么办法退出的 `JavaScript` 代码。 与共享一切框架相反的是，在模块顶部创建的变量不会自动添加到全局共享作用域，而且模块必须导出一些外部代码可以访问的元素（变量或者函数）

- 模块中 `this` 的值是 `undefined`
- 可以支持导入和导出
- 不支持 `HTML` 格式的注释

## 先谈导出

关键字 `export`

> 可以通过 `export` 将任何变量、函数或类导出

```js
let template = ''
//  导出变量
export template
//  导出函数
export function name() {}
// 到处类
export class Test{
  constructor(...) {
    ...
  }
}
----------------------
function test() {
  ...
}
export test

```

模块的默认值（在学习 node 中了解的）

**_ 通过 `default` 关键字导出 _**

```js
// template.js 导出方法或变量
export default function test(a) {
    ...
}

// index.js  引入方法
import {test} from './template.js'

```

> default: 其实就是语法糖， 好处在于 import 引入的时候可以省去{}

```js
// test.js
export default function () {}

// 引入
import test from './test.js'
test()

// 等同于
function test () {}
export {test as default}  //向外界暴露的是test这个变量

//
import {default as test} from './test.js'
test()
```

## require 模块

> node 中最重要的思想之一：模块。node 的 module 遵循 CommonJs 规范，requierjs 遵循 AMD，seajs 遵循 CMD，都是使代码风格保持一致的手段

```js
// node
module.exports = {
  // module.exports 要加上s,但是并不是模块化的规范，只是node的一个私有属性
  test: function(a) {
    console.log(a)
  }
}
// 引入
const a = requier('./test')
a.test(1) // 1

// AMD or CMD
define(function(require, exports, module) {
  module.exports = {
    test: function(a) {
      console.log(a)
    }
  }
})
// 引入
define(function(require, exports, module) {
  const a = requier('./test')
  a.test(1) // 1
})

// 区别在于最外层是够使用了define提供的闭包函数
```

> 区别：  
> import 是编译时加载的,不会添加多余的变量，性能上也较好;require 是代码实际运行是加载的，所以 require 可以在代码的任何地方使用

```js
requier('./test.js')()
```
