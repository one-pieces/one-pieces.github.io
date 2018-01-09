---
title: vue使用jsx时的坑
tags: [vue, jsx, 坑] 
---

1. v-model
在jsx中无法直接使用v-model，需要借助一个插件babel-plugin-jsx-v-model

2. <component>
在jsx中使用动态组件标签<component>会报无法找到component组件的错
解决方法：
直接使用判断语句控制该使用哪个组件

3. v-if
v-if在jsx中是没有作用的
解决办法：
跟<compnent>一样，直接使用判断语句