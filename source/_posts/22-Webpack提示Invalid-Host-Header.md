---
title: 22-Webpack提示Invalid Host Header
date: 2018-11-08 15:49:09
tags: [webpack]
---

## 问题
用[`vue-cli 3`](https://cli.vuejs.org/zh/)新建项目，将自定义域名映射到localhost，这时用自定义域名访问项目，webpack-dev-server返回了Invalid Host Header，而使用localhost或127.0.0.1则没报错。

## 原因
网上查了一些资料，发现这是因为webpack-dev-server在[`2.4.3`](https://github.com/webpack/webpack-dev-server/releases/tag/v2.4.3)和[`1.16.4`](https://github.com/webpack/webpack-dev-server/releases/tag/v1.16.4)这两个版本增加了security fix。这个fix增加了对`Host` header 的检验，来防止恶意网站访问你的`assets`。同时`webpack-dev-middleware`的`1.10.2`也增加了这个修改。

## 解决方案
考虑到这个fix可能会break项目启动，webpack-dev-server提供了几个解决方案

- 新增`disableHostCheck`参数，为`true`时host check失效
- 执行webpack-dev-server时手动添加`--public`参数，值为授权的host

在后续的版本里，webpack-dev-server也增加了相关的一些参数，比如`allowedHosts`([`2.5.0`](https://github.com/webpack/webpack-dev-server/releases/tag/v2.5.0))。

#### 参考
[解决 Webpack "Invalid Host Header"](https://tonghuashuo.github.io/blog/webpack-dev-server-invalid-host-header.html)