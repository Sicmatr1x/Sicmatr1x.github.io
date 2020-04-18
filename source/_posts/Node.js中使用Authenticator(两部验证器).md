---
title: Oracle PL
date: 2017-07-30 21:31:43
categories:
    - 后端
tags: 
    - Authenticator
    - Security
    - Node.js
---

Node.js Authenticator：https://www.npmjs.com/package/authenticator

Two- and Multi- Factor Authenication (2FA / MFA) for node.js

## 关于验证器

Google现在也推荐用户启用两步验证（Two-step verification）功能（Youtube上的视频介绍），并且除了以短信或者电话的方式发送一次性密码之外，还提供了另一种基于时间的一次性密码（Time-based One-time Password，简称TOTP），只需要在手机上安装密码生成应用程序，就可以生成一个随着时间变化的一次性密码，用于帐户验证，而且这个应用程序不需要连接网络即可工作。仔细看了看这个方案的实现原理，发现挺有意思的。下面简单介绍一下。

Google的两步验证算法源自另一种名为HMAC-Based One-Time Password的算法，简称HOTP。HOTP的工作原理如下：

客户端和服务器事先协商好一个密钥K，用于一次性密码的生成过程，此密钥不被任何第三方所知道。此外，客户端和服务器各有一个计数器C，并且事先将计数值同步。

进行验证时，客户端对密钥和计数器的组合(K,C)使用HMAC（Hash-based Message Authentication Code）算法计算一次性密码，公式如下：

```
HOTP(K,C) = Truncate(HMAC-SHA-1(K,C))
```

上面采用了HMAC-SHA-1，当然也可以使用HMAC-MD5等。HMAC算法得出的值位数比较多，不方便用户输入，因此需要截断（Truncate）成为一组不太长十进制数（例如6位）。计算完成之后客户端计数器C计数值加1。用户将这一组十进制数输入并且提交之后，服务器端同样的计算，并且与用户提交的数值比较，如果相同，则验证通过，服务器端将计数值C增加1。如果不相同，则验证失败。

这里的一个比较有趣的问题是，如果验证失败或者客户端不小心多进行了一次生成密码操作，那么服务器和客户端之间的计数器C将不再同步，因此需要有一个重新同步（Resynchronization）的机制。这里不作具体介绍，详情可以参看RFC 4226。

介绍完了HOTP，Time-based One-time Password（TOTP）也就容易理解了。TOTP将HOTP中的计数器C用当前时间T来替代，于是就得到了随着时间变化的一次性密码。非常有趣吧！

Google选择了30秒作为时间片，T的数值为从Unix epoch（1970年1月1日 00:00:00）来经历的30秒的个数。事实上，这个方法还有一个另外的功能。我们知道如果客户端和服务器的时钟有偏差，会造成与上面类似的问题，也就是客户端生成的密码和服务端生成的密码不一致。但是，如果服务器通过计算前n个时间片的密码并且成功验证之后，服务器就知道了客户端的时钟偏差。因此，下一次验证时，服务器就可以直接将偏差考虑在内进行计算，而不需要进行n次计算。


## Install

```bash
npm install authenticator --save
```

## Usage

```js
'use strict';
 
var authenticator = require('authenticator');
 
var formattedKey = authenticator.generateKey();
// "acqo ua72 d3yf a4e5 uorx ztkh j2xl 3wiz"
// 这个formattedKey就是客户端或令牌在手动输入模式下需要输入的密钥
 
var formattedToken = authenticator.generateToken(formattedKey);
// "957 124"
// 这个是根据密钥和时间动态生成的验证码用于和来自用户输入的验证码比较
 
authenticator.verifyToken(formattedKey, formattedToken);
// { delta: 0 }
// 验证验证码是否匹配
 
authenticator.verifyToken(formattedKey, '000 000');
// null
// 验证验证码是否匹配
 
authenticator.generateTotpUri(formattedKey, "john.doe@email.com", "ACME Co", 'SHA1', 6, 30);
// 生成otpauth用于生成二维码给客户端扫码
// otpauth://totp/ACME%20Co:john.doe@email.com?secret=HXDMVJECJJWSRB3HWIZR4IFUGFTMXBOZ&issuer=ACME%20Co&algorithm=SHA1&digits=6&period=30
```
