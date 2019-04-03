---
title: '400 客服电话'
date: 2019-04-03
categories: CRM # 分类只能有1个
tags: # 标签可以有多个
  - 400
  - CRM
  - promise
# cover: "https://text.com/demo.png" # 文章封面图片URL
# description: 文章摘要
---

文章摘要写在前面，支持 markdown 左右语法。

<!-- more -->

文章正文写在这里。

## 使用 promise 封转呼叫中心暴露出来的公用方法

Promise.resolve() 有三种类型， 本身就是一个语法糖

- Promise.resolve(value);
- Promise.resolve(promise);
- Promise.resolve(theanable);

```js
var foo = {
    then: (resolve, reject) => resolve('foo')
};
var resolved = Promise.resolve(foo);
// 相当于
var resolved = new Promise((resolve, reject) => {
    foo.then(resolve, reject)
});

resolved.then((str) =>
    console.log(str);//foo
)
```

> promise 使用的注意事项
>
> - 接受两个参数(resolve, reject)，在最后必须要其中的一个，在返回 `reject()` 时，需要配合 `return reject(...)` 来打断运行和返回错误信息，在外部 `catch()` 捕捉
> - 应用 promise 对象时不能够直接赋值给一个参数，可以通过以下两种简答的方法
>   - `async await` 一直到最后 resolve 或者 reject 返回的不再是一个 promise 对象是才会真正的结束 (推荐使用)
>   - `then` 使用 `then`接受 promise 返回的值

### 主要方法的封装

```js
import init from './acc/init'
import { Message } from 'element-ui'
import authority from './authority'
const telephone = {
  init() {
    return init()
  },
  onLine: async () => {
    // 在线
    const workbench = await init()
    return new Promise(resolve => {
      workbench.online()
      resolve(workbench)
    })
  },
  onLeave: async () => {
    // 离开
    const workbench = await init()
    return new Promise(resolve => {
      setTimeout(() => {
        workbench.applyForBreak()
        resolve(workbench)
      }, 1000)
    })
  },
  removePhone: async (mobileNumber, caller) => {
    // mobileNumber 转接手机 caller客服电话
    const workbench = await init()
    return new Promise(resolve => {
      workbench.offline(mobileNumber, caller)
      resolve(workbench)
    })
  },
  out: async () => {
    const workbench = await init()
    return new Promise(resolve => {
      workbench.onLogOut()
      window.workbench = ''
      resolve()
    })
  },
  ready: async () => {
    // 上线和结束处理后调用
    const workbench = await init()
    return new Promise(resolve => {
      workbench.ready()
      window.workbench = ''
      resolve()
    })
  },
  call: async (phoneNum, boolean) => {
    const workbench = await init()
    return new Promise(resolve => {
      // boolean: true 虚拟外呼
      workbench.call(phoneNum, '', '', boolean)
      resolve(workbench)
    })
  },
  answerPhone: async () => {
    const workbench = await init()
    return new Promise(resolve => {
      workbench.answer()
      resolve(workbench)
    })
  },
  hangUp: async () => {
    const workbench = await init()
    return new Promise(resolve => {
      workbench.hangUp()
      resolve(workbench)
    })
  },
  thirdCallTransfer: async callee => {
    // 转接客服 callee坐席的分机号，转接坐席
    const workbench = await init()
    return new Promise(resolve => {
      workbench.thirdCallTransfer(callee, '')
      resolve(workbench)
    })
  },
  checkStatus: async () => {
    const workbench = await init()
    return new Promise((resolve, reject) => {
      const { code } = workbench.getStatusCode()
      const { callStatus } = authority.get() || {}
      if (callStatus === 'removePhone') {
        Message.warning('当前处于转接手机状态，请切换至在线状态')
        return reject()
      }
      if (code === 4) {
        Message.warning('当前处于离开状态，请切换至在线状态')
        return reject()
      }
      resolve()
    })
  }
}

export default telephone
```
