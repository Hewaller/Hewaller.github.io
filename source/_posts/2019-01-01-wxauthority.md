---
layout: post
title: wx存储用户信息
date: 2019-01-02
categories: weChat
tags: [weChat, localStorage, user]
description: 微信存储信息至本地
---

## 用户权限

```$javascript
const key = 'user_info' // 定义储存的key，一定要确保是单一的
const maxAge = 1000 * 60 * 60 * 24 * 60 // 设置过期时间，60天过期

export default {
  get() {
    // 获取存储到localStorage中的数据，一般是用户信息，使用try/catch捕捉错误
    try {
      const user = wx.getStorageSync(key)
      if (!user || user.time + maxAge < Date.now()) return null
      // 如果储存的时间过期了，直接返回null，重新发送登陆接口获取
      return user || {}
    } catch (e) {
      return {}
    }
  },
  set(user) {
    if (!user) return null
    // 存储数据，没有数据直接返回null，外部捕捉这个null来判定是否成功
    user.time = Date.now()
    // 设置储存时间，用来判断是否过期
    const oldUser = this.get() || {}
    const newUser = { ...oldUser, ...user } //将以前的数据和新的数据一起整合在一起
    console.log(newUser)
    wx.setStorageSync(key, newUser) //微信存储数据的静态方法
    return newUser
  },
  clear() {
    const user = this.get() || {}
    const { regionCode = '', currentCity = '', shop_id = '' } = user
     // 即使是清除数据，也保存重要的登陆信息
    wx.clearStorageSync()
    this.set({ regionCode, currentCity, shop_id })
    // 清空完之后再次储存
    return user
  }
}
```

最后更新日期 2019-03-01
