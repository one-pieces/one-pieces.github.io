---
title: 24.Promises/A+规范
date: 2018-11-14 17:13:13
tags: [Promise]
---

(原文 [Promises/A+](https://promisesaplus.com/))

一个具有互操作性的JavaScript promises的开源标准。

一个promise代表着异步操作的最终结果。与promise进行交互的主要方式是通过then方法，该方法注册回调来接收promise的最终值或promise无法完成（失败）的原因。

该规范详细说明了then方法的行为，它提供了一个可互操作的标准，使所有符合Promises/A+规范的promise依据该标准来实现。因此，该规范应该被认为是非常稳定（stable）的。即使Promises/A+组织偶尔会修改该规范，只是为了解决新发现的极端问题而加入一些比较小的向后兼容的更改，但只有经过仔细考虑，讨论和测试后，我们才会整合大的或向后不兼容的改动。

Promises/A+阐明了早期的Promises/A提案的行为条款，并将其扩展，使其覆盖到现实中的例子，且忽略了未确定和有问题的部分。

最后，该Promises/A+规范的核心并不是确定如何处理新建（create），完成（fulfill），拒绝（reject）promises，而是专注于提供一个可互操作的then方法。相关规范的后续工作可能会涉及到这些主题。

### 1. 术语

1.1 "promise" 是一个具有符合该规范规定行为的then方法的对象或函数

1.2 "thenable" 是一个定义了then方法的对象或函数。

1.3 "value" 是任意合法的JavaScript的值（包括`undefined`，一个thenable，或者一个promise）。

1.4 "exception" 是 一个使用了`throw`语法抛出的值

1.5 "reason" 是一个表示为什么promise被拒绝的原因。

### 2. 要求

#### 2.1 Promise状态

一个Promise必须是以下三个状态中的一个：`pending`，`fulfilled`，`rejected`。

- 2.1.1 当状态为`pending`时：  
    - 2.1.1.1 可以转换到`fulfilled`或者`rejected`。
    
- 2.1.2 当状态为`pending`时：
    - 2.1.2.1 不能转换为其它状态。
    - 2.1.2.2 必须有一个不可修改的`值(value)`。

- 2.1.3 当状态为`rejected`时：
    - 2.1.3.1 不能转换为其它状态。
    - 2.1.3.2 必须有一个不可修改的`原因(reason)`。
    
上文的“不可修改”的含义是一旦创建就不能修改的数据（类似`===`），但并不意味着这个不可变性是深嵌套的。

#### 2.2 `then`方法

一个promise必须提供一个`then`方法去获取它当前或最终的`值`或`原因`。

一个promise的`then`方法接收两个参数：
```js
promise.then(onFulfilled, onRejected)
```

- 2.2.1 `onFulfilled`和`onRejected`都是可选参数：
    - 2.2.1.1 如果`onFulfilled`不是函数，则必须忽略它。
    - 2.2.1.1 如果`onRejected`不是函数，则必须忽略它。
    
- 2.2.2 如果`onFulfilled`是函数：
    - 2.2.2.1 则必须在promise状态转换为`fulfilled`之后调用它，且它的第一个参数为promise的`值`。
    - 2.2.2.2 不能在promise状态转换为`fulfilled`之前调用它。
    - 2.2.2.2 不能多次调用它。
    
- 2.2.3 如果`onRejected`是函数：
    - 2.2.3.1 则必须在promise状态转换为`rejected`之后调用它，且它的第一个参数为promise的`原因`。
    - 2.2.3.2 不能在promise状态转换为`rejected`之前调用它。
    - 2.2.3.2 不能多次调用它。
    
- 2.2.4 在执行上下文堆栈仅包含平台代码之前，不能调用`onFulfilled`或`onRejected`。

- 2.2.5 `onFulfilled`和`onRejected`必须作为函数被调用（即没有`this`值）。

- 2.2.6 同一个promise上的`then`可以被多次调用。
    - 2.2.6.1 如果/当promise为`fulfilled`时，所有相应的`onFulfilled`回调必须按其原本的调用顺序执行。
    - 2.2.6.1 如果/当promise为`onRejected`时，所有相应的`onRejected`回调必须按其原本的调用顺序执行。

- 2.2.7 `then`必须返回一个promise。
    - 2.2.7.1 如果`onFulfilled`和`onRejected`返回一个`值`x，则运行Promise解析器（Promise Resolution Procedure）`[[Resolve]](promise2, x)`。
    - 2.2.7.2 如果`onFulfilled`和`onRejected`返回一个`异常（exception）`e，则promise2必须以e为`原因`被拒绝（reject）。
    - 2.2.7.3 如果`onFulfilled`不是函数且promise1为`fulfilled`，promise2必须以与promise1相同的`值`被完成（fulfilled）。
    - 2.2.7.3 如果`onRejected`不是函数且promise1为`rejected`，promise2必须以与promise1相同的`原因`被拒绝（rejected）。
    
#### 2.3 Promise解析器

Promise解析器（promise resolution procedure）是一个抽象操作，它将promise和`值`作为输入，我们将其表示为`[[Resolve]](promise, x)`。如果`x`是`thenable`，它会使promise采用`x`的状态，假设`x`的行为至少有点像promise。否则，它将使用`x`执行promise。

对`thenable`的这种处理使得promise具有互操作性，只要它们暴露出一个符合Promises/A+规范的`then`方法。它还允许符合Promises/A+规范的实现使用合理的`then`方法与“同化”不一致的实现。

为了运行`[[Resolve]](promise, x)`，需要实现以下几点：

- 2.3.1 如果promise和`x`引用了同个对象，则用`TypeError`作为`原因`去拒绝promise。
- 2.3.2 如果`x`是promise，则采用它的状态：
    - 2.3.2.1 如果`x`状态为`pending`，promise必须保持`pending`直到`x`被完成或被拒绝。
    - 2.3.2.2 如果/当`x`为`fulfilled`，用相同的`值`去完成promise。
    - 2.3.2.3 如果/当`x`为`rejected`，用相同的`原因`去拒绝promise。
    
- 2.3.3 否则，如果`x`是一个对象或函数，
    - 2.3.3.1 让`then`等于`x.then`。
    - 2.3.3.2 如果检测到属性`x.then`会导致抛出异常`e`，则用`e`作为`原因`去拒绝promise。
    - 2.3.3.3 如果`then`是函数，则将`x`作为`this`调用它，第一个参数为`resolvePromise`，第二个参数为`rejectPromise`，其中：
        - 2.3.3.3.1 如果/当`resolvePromise`被用`值``y`调用时，
