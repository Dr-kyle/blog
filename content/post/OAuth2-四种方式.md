---
title: "OAuth2 四种认证方式"
date: 2020-05-24T17:52:14+08:00
lastmod: 2020-05-24T17:52:14+08:00
draft: false
keywords: ["OAuth2"]
description: "OAuth2 四种认证方式学习"
tags: ["OAuth2"]
categories: ["OAuth2"]
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."hugo 
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""



---

OAuth2 四种认证方式的区别

<!--more-->



# OAuth2 学习

本文参考[阮一峰](http://www.ruanyifeng.com/blog/2019/04/oauth_design.html)的博客，本人做一个学习记录(其实就是手敲一遍，建议看阮一峰的博客)，加深印象。 

OAuth2 四种获取令牌的方式

- 授权码 (authorization-code)
- 隐藏式 (implicit)
- 密码式 (password)
- 客户端凭证 (client credentials)

无论哪种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码: client ID & client secret.

## 一  授权码

指的是第三方应用先申请一个授权码，然后在用该码获取令牌

适用于有后端的WEB应用，授权码通过前端传送，令牌则是存储在后端，而且所有与资源服务器的通信都在后端完成，这样的前后端分离，可以避免令牌泄露。

#### 第一步：

A网站提供一个链接，用户点击后会跳转到B网站，授权用户数据给A网站使用，下面就是A网站跳转B网站的一个示例链接

```console
https://b.com/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

`response_type` 表示要求返回授权码(`code`), `client_id`参数让B知道是谁在请求，`redirect_uri`参数是B接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这是是只读）。

#### 第二步：

用户跳转后，B网站会要求用户登录，登录询问是否同意给予A网站授权，用户同意后B网站就会跳回`redirect_uri`参数指定的网址，跳转时，会传回一个授权码，`https://a.com/callback?code=AUTHORIZATION_CODE`,  `CODE`就是授权码

#### 第三步：

A网站拿到授权以后，就可以在后端，向B网站请求令牌

```
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

#### 第四步：

B 网站收到请求以后，就会颁发令牌，做法是向`redirect_uri`指定的网址，发送一段JSON数据。

```json
{
	// 令牌
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

## 二 隐藏式

有些WEB应用是纯前端，没有后端，这时就不能用上面的方式了，必须将令牌存储在前端，这种方式没有授权码这个中间步骤，所以称为隐藏式。

#### 第一步：

A网站提供一个链接，要求用户跳转到B网站，授权用户数据给A网站使用。

```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

`response_type`参数为 `token`, 表示要求直接返回令牌。

#### 第二步：

用户跳转到B网站，登录后同意给予A网站授权，这时B网站就会跳回 `redirect_uri`参数指定的跳转网址，并且把令牌作为URL参数，传给A网站。

```
https://a.com/callback#token=ACCESS_TOKEN
```

令牌的位置是URL锚点(fragment)， 而不是查询字符串(querystring), 这是因为OAuth2.0 允许跳转网址是HTTP协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄露令牌的风险。

这种方式把令牌直接传给前端，是很不安全的，因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间(session)有效，浏览器关掉，令牌就失效了。

## 三 密码式

如果高度信任某个应用，用户可以直接把用户名和密码，直接告诉该应用。该应用就会使用你的密码，申请令牌，这种方式称为 密码式。

#### 第一步：

A 网站要求用户提供B网站的用户名和密码，拿到以后，A就直接向B请求令牌。

```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

#### 第二步：

B网站验证身份通过后，直接给出令牌，注意，这时不需要跳转，而是把令牌放在JSON数据里面，作为HTTP回应，A因此拿到令牌。

## 四 凭证式

适用于没有前端的命令行应用，即在命令行下请求令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

#### 第一步：

A应用在命令行向B发出请求。

```
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

#### 第二步：

B 网站验证通过后，直接返回令牌

## 令牌的使用

在请求头上加上token即可。

```bash
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

## 更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，在申请一个新的令牌，很可能体验不好，而且也没有必要， Oauth 2.0 允许用户自动更新令牌。

具体方法：B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌(refresh_token 字段)。令牌到期前，用户使用refresh token发一个请求，去更新令牌。

```javascript
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

B 网站验证通过以后，就会颁发新的令牌。

​		