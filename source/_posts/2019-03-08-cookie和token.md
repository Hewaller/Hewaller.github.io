---
layout: post
title: cookie 和 token
date: 2019-03-07
categories: 前端
tags:
  - token
  - cookie
  - 网络安全
---

`cookie` 和 `token`

最后更新日期 2019-05-06

<!-- more -->

# cookie 和 token

[cookie 和 token](https://www.jianshu.com/p/ce9802589143)
`cookie` 服务员看你的身份证，给你一个编号，以后，进行任何操作，都出示编号后服务员去看查你是谁。

`token` 直接给服务员看自己身份证

## 前端安全需要注意哪几方面问题

`xss`、`csrf`、`arp`、`xff`、中间人攻击、运营商劫持、防暴刷

## Cookie 与 Cookie 劫持

[Cookie 与 Cookie 劫持](https://g2ex.github.io/2015/06/29/Cookie-and-Cookie-Injection/)

### cookie 劫持

1. 攻击者通过 `xss` 拿到用户的 `cookie` 然后就可以伪造 `cookie` 了。

2. 或者通过 `csrf` 在同个浏览器下面通过浏览器会自动带上 `cookie` 的特性
   在通过 用户网站-攻击者网站-攻击者请求用户网站的方式 浏览器会自动带上 `cookie`

## `token` 加密和登陆验证

> `JWT（JSON Web Tokens）` 用户发送登陆信息至服务器，服务器认证后，生成一个 `JSON` 对象，返回给用户，服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。
> 缺点：由于服务器不保存 `session` 状态，因此无法在使用过程中废止某个 `token`，或者更改 `token` 的权限。也就是说，一旦 `JWT` 签发了，在到期之前就会始终有效。（设置较短的有效期）

- `token`是无状态的，后端服务不需要记录 `token`。每个令牌都是独立的。
- 服务器不记录哪些用户已登陆或者已经发布了哪些 `JWT`。对服务器的每个请求都需要带上验证请求的签名`token`，都要在`HTTP header`中 `Authorization` 字段里，携带服务器返回的 `JSON`
- 服务器对 `JWT` 进行解码，如果 `token` 有效，则处理该请求
- `token` 是放在 `jwtJSON Web Tokens(jwt)`里面下发给客户端的而且不一定存储在哪里（可以存在 `localStorage` `sessionStorage`，或者 `cookie` 中） 不能通过 `document.cookie` 直接拿到，通过 `jwt+ip` 的方式可以防止 被劫持即使被劫持也是无效的 `jwt`
- 也可以储存在 `Cookie` 里面，在请求的时候自动发送，但是不能跨域

```js
// Request Headers 请求中
Authorization: `Bearer ${token}`
```

** JWT 数据格式 **

`Header.Payload.Signature`

> 示例：eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6NjksInVzZXJuYW1lIjoiXHU1OTI3XHU2Y2IzXHU1NDExXHU0ZTFjXHU2ZDQxIiwicGhvbmUiOiIxMzYyMTg3OTc4NCIsImV4cCI6MTU2MTcwNzYwM30.G77w5AMqu1PbT6YL3P1xYHBhUD16rRYvUwa89Nhkvtw
> 需要使用 `Base64` 转为 字符串

- Header (头部)
- Payload (负载)
- Signature (签名)

### Header

JSON 对象

```js
{
 "alg": "HS256", // 签名所用的算法，默认 `HMAC SHA256（HS256)`
 "typ": "JWT" //token 类型
}
```

### Payload

JSON 对象

```js
  {
    "iss": "签发人",
    "exp": "过期时间",
    "nbf": "生效时间",
    "iat": "签发时间",
    "jti": "编号"
  }
```

### Signature

对 `Header` 和 `Payload` 的签名，防止被篡改， 一般是加上一个唯一的字符串，来防止 `token` 被伪造篡改

简单的使用方法：

- 需要一个 `secret`（随机数）
- 后端利用 `secret` 和加密算法(如：HMAC-SHA256)对 `payload`(如账号密码)生成一个字符串(`token`)，返回给前端
- 前端每次 `request` 在 `header` 中带上登陆返回的 `token`
- 后端用同样的算法解密
