---
title: (WIP)22.CommonJS模块规范
date: 2018-09-21 19:35:17
tags: [WIP]
---
对于JavaScript自身而言，它的规范是薄弱的。

- 没有模块系统。
- 标准库较少。
- 没有标准接口。
- 缺乏包管理系统。

这使得JavaScript难以达到像Python、Ruby和Java一样，具备开发大型应用的基础能力。CommonJS规范的提出，主要是为了弥补JavaScript没有标准的缺陷。

### CommonJS的模块规范

CommonJS对模块的定义很简单，主要分为模块引用、模块定义和模块标识3个部分。

1. 模块引用

我们可以通过`require()`方法，接受模块标识，来引入一个模块的API到当前上下文中。
```js
var math = require('math');
```

2. 模块定义

在Node中，一个文件就是一个模块。在模块中，上下文提供了exports对象用于到处当前模块的方法或变量，而且它是唯一导出的出口。
    