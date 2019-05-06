---
title: 'computed、methods 和 watch'
date: 2019-04-25
categories: Vue # 分类只能有1个
tags: # 标签可以有多个
  - Vue
  - 前端
---

`Vue` 知识点： computed、methods 和 watch
最后更新日期 2019-04-25

<!-- more -->

## computed、methods 和 watch

** 参考官网 **

> - 计算属性： 是基于它的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要所依赖的属性还没有发生改变，多次访问计算属性会立即返回之前的计算结果，而不必再次执行函数。但是 **不能够动态的去给这个计算属性传值**
>   默认的计算属性都是读取的`getter` 函数， 但是需要时也可以提供一个 `setter`

```js
  computed: {
    fullName: {
      // getter
      get: function () {
        return this.firstName + ' ' + this.lastName
      },
      // setter
      set: function (newValue) {
        var names = newValue.split(' ')
        this.firstName = names[0]
        this.lastName = names[names.length - 1]
      }
    }
  }
```

> - 方法(methods)： 每次访问都会调用，不会缓存数据，可以动态的接受不同值。
> - 监听属性(watch): 允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

## watch

`handler`方法，`watch` 方法中默认写的就是这个 handler，`Vue`会自己去处理这个逻辑，`watch` 方法中最终编译出来其实就是这个 `handler`。

### immediate 和 deep 属性

- `deep` 属性： 为了发现对象内部值的变化，默认值是 false，代表是否深度监听，可以在选项参数中指定 `deep: true`。(注意监听数组的变动不需要这么做。)

```js
 data() {
    return {
     obj: {
        test: 1
      }
    }
  },
  watch: {
    obj(e) {
      console.log(e, 'change)
    }
  }

  // handler
  watch: {
    obj: {
       handler(new, old) {
        console.log(new, 'change');
      },
    }
  }
```

> 当`obj.a`的值改变，`watch` 并没有监听到变化。受现代 `JavaScript` 的限制，`Vue` 不能检测到对象属性的添加或删除。由于 `Vue` 会在实例初始化时对属性执行 `getter/setter` 转化过程，所有属性必须在 `data` 对象上存在， `Vue` 才能转换它，这就是响应式的远离。
> 默认情况下 `watch` 只监听 obj 这个属性引用的变化，而不能坚定到对象属性的改变，我们只有给`obj`重新整体赋值的时候它才会监听到

- `deep` 深层遍历
  > 监听器会一层层的往下遍历，给对象的所有属性都加上监听器，任何修改 `obj` 里面任何一个属性都会触发这个监听器里的 `handler`，性能开销太大。

```js
watch: {
  obj: {
    handler(new, old) {
      console.log('change');
    },
    deep: true // 深层遍历
  }
}
```

- 日程使用
  > 只监听我们所需要

```js
watch: {
    'obj.test'(e) {
      console.log(e, 'change)
    }
  }
```

- immediate
  在选项参数中指定 `immediate: true` 将立即以表达式的当前值触发回调：

```js
watch: {
  'obj.test': {
    handler(new, old) {
      console.log(new, 'change');
    },
    immediate: true, // 立即执行 obj.test 变化就执行 watch
  }
}
```

## `vm.$watch`

`vm.$watch` 返回一个取消观察函数，用来停止触发回调

```js
var unwatch = vm.$watch('a', cb)
// 之后取消观察
unwatch()
```

> 组件是经常要被销毁的，应该注销掉原来的 `watch` 的，否则可能会导致内置溢出。但是我们平时 `watch` 都是写在组件的选项中的，他会随着组件的销毁而销毁,不需要手动取消。
