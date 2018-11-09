---
title: 23.使用原生JS实现dialog组件
date: 2018-11-08 16:51:45
tags: [JavaScript]
---
## 前言
当时觉得这个蛮简单的，只要写好html、样式，然后append进document.body就行了。但真正写起来时才发现没那么简单。比如原生的appendChild需要参数为Node，而你现在是需要把HTML的字符串转化为Node。这要怎么做呢？

## String转化为DOM