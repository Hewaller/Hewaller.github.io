---
layout: post
title: cookie 和 token
date: 2019-03-07
categories: 前端
tags: [cookie, token, 前端]
description: cookie 和 token
---

# cookie 和 token

[cookie 和 token](https://www.jianshu.com/p/ce9802589143)
`cookie` 服务员看你的身份证，给你一个编号，以后，进行任何操作，都出示编号后服务员去看查你是谁。

`token` 直接给服务员看自己身份证

## Cookie 与 Cookie 劫持

[Cookie 与 Cookie 劫持](https://g2ex.github.io/2015/06/29/Cookie-and-Cookie-Injection/)

## cooki 劫持

1. 攻击者通过 xss 拿到用户的 cookie 然后就可以伪造 cookie 了。

2. 或者通过 csrf 在同个浏览器下面通过浏览器会自动带上 cookie 的特性
   在通过 用户网站-攻击者网站-攻击者请求用户网站的方式 浏览器会自动带上 cookie

## token

- 随着单页面应用程序的流行，以及 Web API 和物联网的兴起，基于 token 的身份机制越来越被大家广泛采用。
- `token`是无状态的，后端服务不需要记录 token。每个令牌都是独立的。
- 服务器不记录哪些用户已登陆或者已经发布了哪些 JWT。对服务器的每个请求都需要带上验证请求的签名`token`。该标记既可以加在 header 中，可以在 POST 请求的主体中发送，也可以作为查询参数发送。不会被浏览器自动携带， `cooli #2`问题解决
- 服务器对 JWT 进行解码，如果 token 有效，则处理该请求
- token 是放在 jwtJSON Web Tokens(jwt)里面下发给客户端的而且不一定存储在哪里（可以存在 local storage，或者 cookie 中） 不能通过 document.cookie 直接拿到，通过 jwt+ip 的方式可以防止 被劫持即使被劫持也是无效的 jwt

最后更新日期 2019-03-07
