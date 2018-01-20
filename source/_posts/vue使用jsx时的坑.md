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

4. 使用on
在jsx中我们要监听一个事件，可以使用on-{eventName}或者on{EventName}的形式，如
```jsx harmony
<button on-click={this.myClickListener}></button>
```
```jsx harmony
<button onClick={this.myClickListener}></button>
```

同时，我们也可以使用on来让组件获得事件监听的能力，如
```jsx harmony
<button {...{ on: { click: this.myClickListener }}}></button>
```
当使用on时，只能像上面一样，用解构的写法才有效，如果像下面这样写是没有效果的。
```jsx harmony
<button on={{ click: this.myClickListener }}></button>
```

注意！on-{eventName}或on{EventName}跟on一起写时，on会无效。比如
```jsx harmony
<input onClick={this.myClickListener} {...{ on: { input: this.myInputListener } }} />
```
当触发input事件时会报错。