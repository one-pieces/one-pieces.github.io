---
title: 23.浏览器JavaScript执行的时间线
date: 2018-11-09 15:29:40
tags: [JavaScript, 浏览器]
---

## 前言
我們知道浏览器的渲染线程和脚本线程是互斥的，JavaScript会阻塞浏览器HTML解析器，这也是为什么长时间的脚本运行可能会导致页面失去响应。因此，我们通常会把`<script>`放在`</body>`之前，来保证非脚本的其他一切元素能尽快地得到加载和解析。  
然而，我们可以通过设置`<scirpt>`的`async`和`defer`来改变`<script>`的加载和执行顺序。在说这具体是怎样之前,我们先介绍一下浏览器JavaScript执行的时间线。

## JavaScript的执行
1. 浏览器创建`Document`对象，并且开始解析Web页面，解析HTML元素和它们的文本内容后，会添加Element对象和Text节点到文档中。在这个阶段document.readystate属性的值是`loading`。
2. 当HTML解析器遇到没有`async`和`defer`属性的`<script>`元素时，它把这些元素添加到文档中，然后执行行内或者外部脚步。这些脚本会同步执行，并且在脚本下载（如果需要）和执行时，HTML解析器会暂停。这样脚本就可以

