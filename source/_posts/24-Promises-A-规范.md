---
title: 24.Promises/A+ 规范
date: 2018-11-14 17:13:13
tags: [Promise, 规范]
---

(原文 [Promises/A+](https://promisesaplus.com/))

一个具有互操作性的 JavaScript promises 的开源标准。

一个 promise 代表着异步操作的最终结果。与 promise 进行交互的主要方式是通过then方法，该方法注册回调来接收 promise 的最终值或 promise 无法完成（失败）的原因。

该规范详细说明了then方法的行为，它提供了一个可互操作的标准，使所有符合 Promises/A+ 规范的 promise 依据该标准来实现。因此，该规范应该被认为是非常稳定（stable）的。即使 Promises/A+ 组织偶尔会修改该规范，只是为了解决新发现的极端问题而加入一些比较小的向后兼容的更改，但只有经过仔细考虑，讨论和测试后，我们才会整合大的或向后不兼容的改动。

Promises/A+ 阐明了早期的 Promises/A 提案的行为条款，并将其扩展，使其覆盖到现实中的例子，且忽略了未确定和有问题的部分。

最后，该 Promises/A+ 规范的核心并不是规定如何新建（create），完成（fulfill），拒绝（reject） promises，而是专注于提供一个可互操作的then方法。相关规范的后续工作可能会涉及到这些主题。

### 1. 术语

1.1 "promise" 是一个具有符合该规范规定行为的then方法的对象或函数

1.2 "thenable" 是一个定义了then方法的对象或函数。

1.3 `值`（"value"） 是任意合法的JavaScript的值（包括`undefined`，一个 thenable，或者一个 promise）。

1.4 `异常`（"exception"） 是 一个使用了`throw`语法抛出的值

1.5 `原因`（"reason"） 是一个表示为什么 promise 被拒绝的原因。

### 2. 要求

#### 2.1 Promise状态

一个 Promise 必须处在以下三种状态中的一个：`pending`，`fulfilled`，`rejected`。

- 2.1.1 当状态为`pending`时：  
    - 2.1.1.1 可以转换到`fulfilled`或者`rejected`。
    
- 2.1.2 当状态为`fulfilled`时：
    - 2.1.2.1 不能转换为其它状态。
    - 2.1.2.2 必须有一个不可修改的`值(value)`。

- 2.1.3 当状态为`rejected`时：
    - 2.1.3.1 不能转换为其它状态。
    - 2.1.3.2 必须有一个不可修改的`原因(reason)`。
    
上文的“不可修改”的含义是一旦创建就不能被修改（类似`===`），但并不意味着这个不可变性是深嵌套的。

#### 2.2 `then`方法

一个 promise 必须提供一个`then`方法去获取它当前或最终的`值`或`原因`。

一个 promise 的`then`方法接收两个参数：
```js
promise.then(onFulfilled, onRejected)
```

- 2.2.1 `onFulfilled`和`onRejected`都是可选参数：
    - 2.2.1.1 如果`onFulfilled`不是函数，则必须忽略它。
    - 2.2.1.1 如果`onRejected`不是函数，则必须忽略它。
    
- 2.2.2 如果`onFulfilled`是函数：
    - 2.2.2.1 则必须在 promise 状态转换为`fulfilled`之后调用它，且它的第一个参数为 promise 的`值`。
    - 2.2.2.2 不能在 promise 状态转换为`fulfilled`之前调用它。
    - 2.2.2.2 不能多次调用它。
    
- 2.2.3 如果`onRejected`是函数：
    - 2.2.3.1 则必须在 promise 状态转换为`rejected`之后调用它，且它的第一个参数为 promise 的`原因`。
    - 2.2.3.2 不能在 promise 状态转换为`rejected`之前调用它。
    - 2.2.3.2 不能多次调用它。
    
- 2.2.4 在执行上下文堆栈中仅包含平台代码[[3.1]](#3-注释)之前，不能调用`onFulfilled`或`onRejected`。

- 2.2.5 `onFulfilled`和`onRejected`必须作为函数被调用（即没有`this`值）。[[3.2]](#3-注释)

- 2.2.6 同一个 promise 上的`then`可以被多次调用。
    - 2.2.6.1 如果/当 promise 为`fulfilled`时，所有相应的`onFulfilled`回调必须按其原本的调用顺序执行。
    - 2.2.6.1 如果/当 promise 为`onRejected`时，所有相应的`onRejected`回调必须按其原本的调用顺序执行。

- 2.2.7 `then`必须返回一个 promise。[[3.3]](#3-注释)
    ```js
    promise2 = promise1.then(onFulfilled, onRejected);
    ```
    - 2.2.7.1 如果`onFulfilled`或`onRejected`返回一个`值`x，则运行 Promise 解析器（Promise Resolution Procedure）`[[Resolve]](promise2, x)`。
    - 2.2.7.2 如果`onFulfilled`或`onRejected`返回一个`异常（exception）`e，则 promise2 必须以e为`原因`被拒绝。
    - 2.2.7.3 如果`onFulfilled`不是函数且 promise1 为`fulfilled`，promise2 必须以与 promise1 相同的`值`被完成。
    - 2.2.7.3 如果`onRejected`不是函数且 promise1 为`rejected`，promise2 必须以与 promise1 相同的`原因`被拒绝。
    
#### 2.3 Promise解析器

Promise 解析器（promise resolution procedure）是一个抽象操作，它将 promise 和`值`作为输入，我们将其表示为`[[Resolve]](promise, x)`。如果`x`是`thenable`，它会使 promise 采用`x`的状态，假设`x`的行为至少有点像 promise。否则，它将以`x`为`值`完成 promise。

对`thenable`的这种处理使得 promise 具有互操作性，只要它们暴露出一个符合 Promises/A+ 规范的`then`方法。它还允许符合 Promises/A+ 规范的实现使用合理的`then`方法与“同化”不一致的实现。

为了运行`[[Resolve]](promise, x)`，需要实现以下几点：

- 2.3.1 如果 promise 和`x`引用了同个对象，则用`TypeError`作为`原因`去拒绝 promise。
- 2.3.2 如果`x`是 promise，则采用它的状态：[[3.4]](#3-注释)
    - 2.3.2.1 如果`x`状态为`pending`，promise 必须保持`pending`直到`x`被完成或被拒绝。
    - 2.3.2.2 如果/当`x`为`fulfilled`，用相同的`值`去完成 promise。
    - 2.3.2.3 如果/当`x`为`rejected`，用相同的`原因`去拒绝 promise。
    
- 2.3.3 否则，如果`x`是一个对象或函数，
    - 2.3.3.1 让`then`等于`x.then`。[[3.5]](#3-注释)
    - 2.3.3.2 如果检测到属性`x.then`会导致抛出异常`e`，则用`e`作为`原因`去拒绝 promise。
    - 2.3.3.3 如果`then`是函数（译者注，也就是`x`为`thenable`），则将`x`作为`this`调用它，第一个参数为`resolvePromise`，第二个参数为`rejectPromise`，其中：
        - 2.3.3.3.1 如果/当`resolvePromise`被调用且参数为`y`时，执行`[[Resolve]](promise, y)`。
        - 2.3.3.3.2 如果/当`rejectPromise`被调用且参数为`r`时，则以`r`为`原因`拒绝 promise。
        - 2.3.3.3.3 如果`resolvePromise`和`rejectPromise`都被调用，或者对同一个参数进行多次调用时，则处理第一次调用，且忽略后续调用。
        - 2.3.3.3.4 如果调用`then`时抛出一个异常`e`，
            - 2.3.3.3.4.1 如果已调用`resolvePromise`或`rejectPromise`，请忽略它。
            - 2.3.3.3.4.2 否则，以`e`作为`原因`拒绝 promise。
    - 2.3.3.4 如果`then`不是函数，则以`x`作为`值`完成 promise。
- 2.3.4 如果`x`不是一个对象或函数，则以`x`作为`值`完成 promise。

如果使用了在`thenable`循环链（`thenable`的`then`是`thenable`）上的`thenable`去解决 promise，那么`[[Resolve]](promise, thenable)`的递归性质最终会导致`[[Resolve]](promise, thenable)`再次被调用，上述算法将导致无限递归。我们鼓励（但不是必需）对此类递归逻辑进行检测，如果有，则以一个信息性的`TypeError`作为`原因`去拒绝 promise。[[3.6]](#3-注释)

### 3. 注释

- 3.1 这里的`平台代码`特指执行引擎，运行环境和 promise 实现的代码。在实践中，这个要求确保了`onFulfilled`和`onRejected`的异步执行，是在调用`then`的事件轮训时间片之后，且调用栈为空。这可以使用诸如 [setTimeout](https://html.spec.whatwg.org/multipage/webappapis.html#timers) 或 [setImmediate](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/setImmediate/Overview.html#processingmodel) 之类的`宏任务`机制，或者 [MutationObserver](https://dom.spec.whatwg.org/#interface-mutationobserver) 或 [process.nextTick](https://nodejs.org/api/process.html#process_process_nexttick_callback) 之类的`微任务`机制来实现。由于 promise 的实现代码被认为是平台代码，因此它本身可能包含一个任务调度队列或`trampoline（跳板）`来调用处理程序。

- 3.2 也就是说，在严格模式（`use strict`）下，`this`将是`undefined`；在松散模式（`sloppy mode`）下，`this`将会是全局对象。

- 3.3 如果具体实现满足所有要求，我们可以允许`promise2 === promise1`。每个具体实现都应该说明它是否可以支持`promise2 === promise1`，以及在什么条件下支持。

- 3.4 一般来说，如果它来自于当前的具体实现，那`x`可以被认为是一个真正的 promise。这个条款允许我们使用特定的实现手段，让我们可以去获取被认为符合规范（译者注，比如一个自行实现了`then`方法的对象）的 promises 的状态。

- 3.5 该步骤需先存储`x.then`的引用，后续测试和调用的都是该引用，这样可以避免多次访问`x.then`属性。这些预防措施对于保证可访问属性的一致性非常重要。可访问属性的值可能会在检索之间发生变化。

- 3.6 该实现不应该对`thenable`链的深度设置某个限制值，假设超过这个限制值之后会发生无限递归。只有真正的循环才会引起一个`TypeError`错误；如果是不同的`thenable`组成一个无穷的调用链，那么一直递归其实是正确的行为。

在法律允许的范围内，Promises/A+ 组织已放弃 Promises/A+ Promise 规范的所有版权，以及相关或相近的权利。该声明发表于：美国。