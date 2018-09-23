---
title: 20.使用wafer2-quickstart-nodejs-master小程序模板本地开发踩坑指南
date: 2018-09-23 22:31:07
tags: 微信小程序
---
## 使用wafer2-quickstart-nodejs-master小程序模板本地开发踩坑指南

#### 一、本地调试获取用户失败

在使用微信官方提供的 [wafer2-quickstart-nodejs-master](https://github.com/tencentyun/wafer2-quickstart-nodejs) 模板开发小程序时，除了可以连接腾讯云提供的开发环境，还可以直接在本地起服务进行调试。具体操作可参考[本地如何搭建开发环境](https://github.com/tencentyun/wafer2-startup/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98#%E6%9C%AC%E5%9C%B0%E5%A6%82%E4%BD%95%E6%90%AD%E5%BB%BA%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)。

但按照文档上的步骤操作，会报“ERR_REQUEST_PARAM”的错误。报这个错误的原因是，代码默认使用了腾讯云代理登录小程序。

打开`server/config.js`，将`useQcloudLogin`选项修改为false，同时还需要填上小程序的`appSecret`。因为`useQcloudLogin`为`true`时，`SDK`会通过与小程序关联的腾讯云账号获取到`appSecret`，授权通过后，再获取用户信息。否则为`false`时，则需要填写`appSecret`。

这样修改后，再次请求，就能获取到用户信息了。



#### 二、第一次获取用户信息

在模板中，`wafer2-client-sdk`封装了一下方法让我们使用。比如我们可以使用`request`发送请求。

```js
var qcloud = require('../vendor/wafer2-client-sdk/index')
...
// 请求A
qcloud.request({
    url,
   	data,
    method,
    login: false,
    success(result) {
    },
    fail(error) {
    }
})
```

通过参数`login`可以控制发送该请求`A`之前，是否先发送登录请求。如果请求`A`需要用户信息，则`login`需为`true`，否则后端的ctx.state.$wxInfo是没有用户信息的。但是在用户第一次登录之前（`cSessioninfo`表里没有该用户数据），`qcloud.request({ login: true })`会报错，返回`Cannot read property 'user_info' of undefined`错误。

这是因为`qcloud.request`只使用了`loginWithCode`来登录。这个方法会在`Header`加上`X-WX-Code`（其实就是`cSessioninfo`表里的`skey`）。后端通过这个code去获取对应用户的信息。但因为表里根本没有这个用户的记录，所以报了上面的错误。

要怎么解决这个问题？其实模板中有类似的调用。打开`client/page/pages/index/index.js`，答案就在`bindGetUserInfo`里。

```js
...
// 先获取session
const session = qcloud.Session.get()

// 1. 如果session不为空，则说明用户已经登录过，即数据库里有该用户记录，则使用qcloud.loginWithCode
// 2. 如果session为空，则说明用户第一次登录，则调用qcloud.login，将用户信息写入数据库
if (session) {
    // 第二次登录
    // 或者本地已经有登录态
    // 可使用本函数更新登录态
    qcloud.loginWithCode({
        success: res => {
            ...
        },
        fail: err => {
            ...
        }
    })
} else {
    // 首次登录
    qcloud.login({
        success: res => {
            ...
        },
        fail: err => {
            ...
        }
    })
}
```

`qcloud.login`会在`Header`增加三个字段，`X-WX-Code`、`X-WX-Encrypted-Data`、`X-WX-IV`，然后后端会在表里插入该用户的信息。

所以我们需要将这段逻辑加到`qcloud.request`里。`qcloud.request`里有个`doRequestWithLogin`方法，是发送登录逻辑的。

```js
// 登录后再请求
function doRequestWithLogin() {
    loginLib.loginWithCode({ success: doRequest, fail: callFail });
}
```

将其修改为

```js
// 登录后再请求
function doRequestWithLogin() {
    const session = Session.get()
    if (session) {
        loginLib.loginWithCode({ success: doRequest, fail: callFail });
    } else {
        loginLib.login({ success: doRequest, fail: callFail });
    }
}
```

Done！大功告成，再次发送请求成功！
