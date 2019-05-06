---
layout: post
title: Vue小知识点
categories: Vue
tags:
  - other
  - 前端
  - Vue
---

Vue 小知识点

最后更新时间 2019-05-06

<!-- more -->

## event bus

不用单独在 `utils` 文件夹下建立一个 `bus.js` 文件来只挂载 `vue`，可以在 `main.js` 中全局注入 `Vue.prototype.$bus = new Vue()`

## @input 和 :value

`v-module` 的语法糖

> 默认情况下，一个组件上的 v-model 会把 value 用作 prop 且把 input 用作 event，但是一些输入类型比如单选框和复选框按钮可能想使用 value prop 来达到不同的目的。使用 model 选项可以回避这些情况产生的冲突。

```html
<my-checkbox :checked="foo" @change="val => { foo = val }" value="some value">
</my-checkbox>
```

```js
 model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    value: String,
    checked: {
      type: Number,
      default: 0
    }
  }
```

## 为什么不用 `map` 来遍历循环验证图片 `size`，而是使用 `for-in` 来循环

获得图片 size 的方法是异步的，会跑出来一个 Promise

## 两个 `image` 元素之间有间隙

根本原因在于 `img` 标签为 `inline` 元素，该元素默认垂直对齐方式为以父元素的 `baseline`，但是展示时又是以 `bottomline` 为对齐方式，因此造成了上下两个 `img` 标签之间的间隙。

- `img` 本来是行内元素，却可以用 `width` 和 `height`,当父元素没有设置高度的时候，用子元素们的高度计算出的高度给父元素的时候就会出现 `3px` 空隙这类的问题。

- `img` 图片默认排版为 `inline-block`;而所有的 `inline-block` 元素之间都会有空白。

解决方法：

- 给图片增加样式 `display：block`
- `img{vertical-align:top;}` 改变其垂直对齐方式
- `div{font-size:0}` 把父元素的文字大小设置为 0
- `div{ margin-bottom:-3px }`

### 0.1 + 0.2 ！= 0.3

浮点数比较方法：使用最小精度值
`Math.abs(0.1+0.2-0.3) <= Number.EPSILON`
