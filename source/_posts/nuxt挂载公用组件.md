---
title: 'nuxt挂载公用组件'
date: 2019-04-11
categories: nuxt # 分类只能有1个
tags:
  - nuxt
  - 公共组件
---

`nuxt` 挂载全局公共组件

<!-- more -->

### 判断是否是服务端还是客户端

`Node` 和浏览器之间存在许多微小的环境差异，全局化组件需要注册在 `window` 上，如果在 `Node` 中，由于 `window` 不存在，程序会报错，可以使用下面两种方法避免在服务端使用 `window`

- `typeof window === 'undefined'`

> 使用 `typeof` 对 `window` 进行判断，即使 `window` 不存在， 也不会报错，只会返回 `undefined`
> 使用 `typeof` 判断 `window` 是否存在，如果 返回的值是 `undefined，` 则不会继续后续对 `window` 的操作和注册全局的组件

- `process.browser`
  > 在浏览器中它返回 true，而在 Node 中它返回 false， 也可以用来判断是服务端还是客户端

### 全局化挂载组件

- 首先完成一个简单的组件

```html
<!-- SuccessBox.vue -->
<template>
  <div class="success-wrapper center-flex" @click="close" v-if="visible">
    <div class="success-container" @click.stop>
      <div class="bg-container">
        <img src="../assets/success_icon.png" alt="" class="bg-img" />
      </div>
      <div class="success-header">
        {{data.header}}
      </div>
      <div class="success-detail" v-for="(item, key) in data.detail" :key="key">
        {{item}}
      </div>
      <div class="success-tip" v-if="data.tip">
        {{data.tip}}
      </div>
      <div class="success-button" @click.stop="close">
        确认
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'SuccessBox',
    props: {},
    data() {
      return {
        visible: true,
        data: {
          header: '购买成功',
          detail: ['购买成功!', '请在门店支付订单后进行兑换！'],
          tip: ''
        }
      }
    },
    methods: {
      close() {
        this.visible = false
      }
    }
  }
</script>
```

- 挂在到全局对象 `window` 上

```js
// utils/SuccessBox.js

import Vue from 'vue'
const SuccessBox = Vue.extend(require('@/common/successBox.vue').default)

let instance

export default {
  show: (data = {}, callback) => {
    if (typeof document === 'undefined') return // 服务端渲染
    // 挂载到全局对象上
    if (!instance) {
      instance = new SuccessBox({
        el: document.createElement('div')
      })
      document.body.appendChild(instance.$el)
    }
    if (callback) instance.callback = callback
    instance.data = { ...instance.data, ...data }
    Vue.nextTick(() => {
      instance.visible = true
    })
  }
}
```

- 挂载到全局

```js
//  nuxt.config.js
  plugins: [
    { src: '~plugins/vue-awesome-swiper.js', ssr: false }, // swiper
    '~plugins/$SuccessBox.js'
  ],
```
