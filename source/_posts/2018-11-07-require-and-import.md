---
layout: post
title: 前端模块化
date: 2018-11-07
categories: 前端
tags:
  - JavaScript
  - 前端
---

`require` 和 `import`
最后更新日期 2018 年 11 月 07 日

<!-- more  -->

# 模块化

> 定义： 模块是自动运行在严格模式下并且没有那么办法退出的 `JavaScript` 代码。 与共享一切框架相反的是，在模块顶部创建的变量不会自动添加到全局共享作用域，而且模块必须导出一些外部代码可以访问的元素（变量或者函数）

- 模块中 `this` 的值是 `undefined`
- 可以支持导入和导出
- 不支持 `HTML` 格式的注释

## 先谈导出

`js` 中模块导出的关键字 `export`

> 可以通过 `export` 将任何变量、函数或类导出

### 导出的基本用法

```js
let template = ''
function test() {
  ...
}
// 导出的几种形式
export template
export function name() {}
export {test, template}
export { test as alias, template};
export class Test{
  constructor(...) {
    ...
  }
}
```

### 模块的默认值（在学习 node 中了解的）

**通过 `default` 关键字导出**

> `export default`: 其实就是语法糖， 好处在于 import 引入的时候可以省去{}

```js
// test.js
// 一个模块中有切只能有一个 `export default`， 后面定义的会覆盖前面的内容
export default function () {}
// 引入
import test from './test.js'
test()

// 等同于使用别名导出
function test () {}
export {test as default}  //向外界暴露的是test这个变量

//
import {default as test} from './test.js'
test()
```

> 在 `node.js` 中， `module.exports` 提供了一个别名 `exports`，可以支持导出多个成员
> 借鉴 [基于 Node.js 的服务端开发](https://nodejs.lipengzhou.com/04-module.html#%E4%B8%BA%E4%BB%80%E4%B9%88-exports-xxx-%E4%B8%8D%E8%A1%8C)

```js
function fn() {
  // 每个模块内部有一个 module 对象
  // module 对象中有一个成员 exports 也是一个对象
  var module = {
    exports: {}
  }

  // 模块中同时还有一个成员 exports 等价于 module.exports
  var exports = module.exports

  console.log(exports === module.exports) // => true

  // 这样是可以的，因为 exports === module.exports
  // module.exports.a = 123
  // exports.b = 456

  // 这里重新赋值不管用，因为模块最后 return 的是 module.exports
  // exports = function () {
  // }

  // 这才是正确的方式
  module.exports = function() {
    console.log(123)
  }

  // 最后导出的是 module.exports
  return module.exports
}

var ret = fn()

console.log(ret)
```

```js
// 导出单个成员
module.exports = function(x, y) {
  return x + y
}
// 导出多个
exports.a = 123
exports.b = 456
exports.c = 789
exports.fn = function() {}
```

## 再谈导入 `import`

> import 是静态分析，导出的是对象引用

### 导入的基本用法

```js
import [name] from '....';
import * as name from '....';
import { export1, export2 as alias } from '....';
import defaultExport, { export1, export2} from '....';
import defaultExport, * as name from '....';
import '....';
```

## require 模块

> `node` 中最重要的思想之一：模块。`node` 的 `module` 遵循 `CommonJs` 规范，`requierjs` 遵循 `AMD`，`seajs` 遵循 `CMD`，都是使代码风格保持一致的手段

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
require('./test.js')()
```
