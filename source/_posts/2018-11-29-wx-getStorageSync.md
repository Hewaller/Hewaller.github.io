---
layout: post
title: wx.getStorageSync And wx.setStorageSync
date: 2018-11-29
categories: blog
tags: [JavaScript, 前端, 小程序]
description: 小程序存储信息
---

# 注意事项

1. 读取信息的时候要使用 try/catch 捕捉错误
2. key 要唯一
3. 清除数据之前要保存重要的信息： 用于登陆所需的信息

```javascript
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
    console.log(newUser, 'newuser')
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

2018 年 11 月 29 日
