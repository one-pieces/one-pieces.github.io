---
title: 使用css时的坑
tags: [css, 样式, 坑] 
---

1. height：100%的问题
有时我们为了让元素高度占满整屏，会把元素的height设置为100%，这样貌似没问题。
![image](../images/shi-yong-css-shi-de-keng/height100.png)

但只是这样会有一个问题。但元素内的内容高度大于元素高度时，超过的部分的背景色就不为元素的背景色。
![image](../images/shi-yong-css-shi-de-keng/height100-bug.png)

这是因为元素的overflow为visible(默认值)，这时元素的高度为可视部分的高度。

我们把overflow设置为auto，即可修复这个问题。
