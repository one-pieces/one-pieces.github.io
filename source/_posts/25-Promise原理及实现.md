---
title: 25.Promise原理及实现
date: 2018-11-21 20:42:27
tags:
---

上一篇文章 [24.Promises/A+ 规范](http://one-pieces.me/2018/11/14/24-Promises-A-%E8%A7%84%E8%8C%83/) 介绍了Promises/A+ 规范，我们来手动实现一个符合 Promises/A+ 规范的自己的Promise。

根据规范，Promise 共有三种状态 `pending`，`fulfilled`，`rejected`，我们使用 `state` 来表示 promise 当前的状态，使用 `value` 来表示当前`值`或`原因`。当然还有最重要的 `then` 方法。同时我们还需要一些辅助函数，比如 `isFunction`。
```js
// 2.1
const PENDING = 0;
const FULFILLED = 1;
const REJECTED = 2;

function Promise () {
  this.state = PENDING;
  // 值或原因
  this.value = void 0;
}

Promise.prototype.then = function (onFulfilled, onRejected) {
  // 2.2.1
  if (isFunction(onFulfilled) || isFunction(onRejected)) {
    this.onFulfilled = onFulfilled;
    this.onRejected = onRejected;
  }
  // 根据2.2.7，
  // then必须返回一个 promise，但它还有其他的规则，
  // 这里我们先不管，直接返回当前 promise
  return this;
};

function isFunction (func) {
  return typeof func === 'function';
}
function isObject(obj) {
  return typeof obj === 'object';
}
```

接着，我们定义两个函数，可以将 promise 转换为 `fulfilled` 或 `rejected`。
```js
function doResolve (promise, value) {
  promise.state = FULFILLED;
  promise.value = value;
  
  if (isFunction(promise.onFulfilled)) {
    // 2.2.4
    // 需要确保onFulfilled和onRejected的异步执行，
    // 是在调用then的事件轮训时间片之后，且调用栈为空。
    // 更多详情可看http://one-pieces.me/2018/11/14/24-Promises-A-%E8%A7%84%E8%8C%83/#3-%E6%B3%A8%E9%87%8A
    setTimeout(() => {
      promise.onFulfilled(promise.value);
    }, 0);
  }
  return promise;
}
function doReject (promise, reason) {
  promise.state = REJECTED;
  promise.value = reason;
  if (isFunction(promise.onRejected)) {
    setTimeout(() => {
      promise.onRejected(promise.value);
    }, 0);
  }

  return promise;
}
```

最后，我们还需要一个 resolver，将刚才定义的 `doResolve` 和 `doReject` 暴露给 Promise 的构造函数。想想我们平时是怎么 new 一个 Promise的？
```js
function safelyResolveThen(promise, then) {
  try {
    then(function (value) {
      doResolve(promise, value);
    }, function (reason) {
      doReject(promise, reason);
    })
  } catch (e) {
    doReject(promise, e);
  }
}

// 然后修改 Promise 构造函数
function Promise (resolver) {
  this.state = PENDING;
  // 值或原因
  this.value = void 0;
  safelyResolveThen(this, resolver);
}
```

到这里，Promise 最核心的功能已经实现。可以很容易地看出，这里 Promise 的底层实现使用了`回调（callback）`