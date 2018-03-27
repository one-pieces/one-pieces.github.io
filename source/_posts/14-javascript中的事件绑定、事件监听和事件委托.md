---
title: 14.JavaScript中的事件绑定、事件监听和事件委托
date: 2018-03-08 19:23:55
tags: [javascript]
---

【转自】https://www.cnblogs.com/Tacklingpioneer/p/6433959.html

在JavaScript的学习中，我们经常会遇到JavaScript的事件机制，例如，事件绑定、事件监听、事件委托（事件代理）等。这些名词是什么意思呢，有什么作用？

## 事件绑定

要想让JavaScript对用户的操作作出响应，首先要对 DOM 元素绑定事件处理函数。所谓事件处理函数，就是处理用户操作的函数，不同操作对应不同的名称。

在JavaScript中，有三种常用的绑定事件的方法：

- 在 DOM 元素中直接绑定
- 在JavaScript代码中绑定
- 绑定事件监听函数

### 在 DOM 中直接绑定事件

我们可以在 DOM 元素上绑定 onclick、onmouseover、onmouseout、onmousedown、onmouseup、onkeydown、onkeypress等。更多事件类型请查看[DOM事件](http://www.runoob.com/jsref/dom-obj-event.html)。
```html
<input type="button" value="click me" onclick="hello()">

<script>
  function hello() {
    alert("hello world!");
  }
</script>
```

### 在JavaScript代码中绑定事件

在JavaScript代码中（即 script 标签内）绑定事件可以使JavaScript代码与HTML标签分离，文档结构清晰，便于管理和开发。
```html
<input type="button" value="click me" id="btn">

<script>
  document.getElementById("btn").onclick = function() {
    alert("hello world!");
  }
</script>
```

### 使用事件监听绑定事件
绑定事件的另一种方法是用 addEventListener() 或 attachEvent() 来绑定事件监听函数。下面详细介绍，事件监听。

## 事件监听

关于事件监听，W3C规范中定义了3个事件阶段，依次是捕获阶段、目标阶段、冒泡阶段。

起初NetScape制定了JavaScript的一套事件驱动机制（即事件捕获）。随机IE也推出了自己的一套事件驱动机制（即事件冒泡）。最后W3C规范了两种事件机制，分为捕获阶段、目标阶段、冒泡阶段。IE8以前IE一直坚持自己的事件机制（前端人员一直头疼的兼容问题），IE9以后IE也支持了W3C规范。

### W3C规范

语法：  

element.addEventListener(event, function, useCapture)

- event：（必需）事件名，支持所有[DOM事件](http://www.runoob.com/jsref/dom-obj-event.html)
- function：（必需）指定要事件触发时执行的函数
- useCapture：（可选）指定事件是否在捕获或冒泡阶段执行。true，捕获。false，冒泡。默认false。

注：IE8以下不支持。

```html
<input type="button" value="click me" id="btn1">

<script>
  document.getElementById("btn1").addEventListener("click", hello);
  function hello() {
    alert("hello world!");
  }
</script>
```

### IE标准

语法：

element.attachEvent(event, function)

- event：（必需）事件类型。需加"on"，例如：onclick。
- function：（必需）指定要事件触发时执行的函数。

```html
<input type="button" value="click me" id="btn2">

<script>
  document.getElementById("btn2").attachEvent("onclick", hello);
  function hello() {
    alert("hello world!");
  }
</script>
```

## 事件监听的优点

1. 可以绑定多个事件。

```html
<input type="button" value="click me" id="btn3">

<script>
  var btn3 = document.getElementById("btn3");
  btn3.onclick = function() {
    alert("hello 1"); // 不执行
  }
  btn3.onclick = function() {
    alert("hello 2"); // 执行
  }
</script>
```

常规的事件绑定只执行最后绑定的事件。

```html
<input type="button" value="click me" id="btn4">

<script>
  var btn4 = document.getElementById("btn4");
  btn4.addEventListener("click", hello1);
  btn4.addEventListener("click", hello2);
  
  function hello1() {
    alert("hello 1");
  }
  function hello2() {
    alert("hello 2");
  }
</script>
```

两个事件都执行了。

2. 可以解绑响应的事件

```html
<input type="button" value="click me" id="btn5">

<script>
  var btn5 = document.getElementById("btn5");
  btn5.addEventListener("click", hello1); // 执行
  btn5.addEventListener("click", hello2); // 不执行
  btn5.removeEventListener("click", hello2);
  
  function hello1() {
    alert("hello 1");
  }
  function hello2() {
    alert("hello 2");
  }
</script>
```

## 封装事件监听

```html
<input type="button" value="click me" id="btn5">

// 绑定监听事件
```
