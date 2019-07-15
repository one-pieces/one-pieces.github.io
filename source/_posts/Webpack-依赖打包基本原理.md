---
title: Webpack 依赖打包基本原理
date: 2019-07-09 16:40:15
tags: [性能优化, webpack]
---

今天通过一个简单的例子来说明一下 Webpack 是怎么解决模块依赖的。
> 本文使用 webpack 4，`mode` 为 `development`，因此其 `devtool` 为 `eval`。

## 一、生成产物
首先先来看看下面三个源文件
```js
// entry.js
import message from './message.js';
console.log(message);

// message.js
import { name } from './name.js';
export default `hello ${name}!`;

// name.js
export const name = 'world';

export const text = 'text';
```
Webpack 配置文件
```js
// webpack.config.js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './entry.js',
  output: {
    filename: 'entry.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```
运行 `npx webpack`，打包的产物在 `dist/entry.js`。

## 二、打包产物分析

> 这里使用了 `development` 模式，所以产物没有进行最小化处理。

现在让我们看看产物的整体结构
```js
/******/ (function(modules) { // webpackBootstrap
/******/    // ...
/******/ })
/************************************************************************/
/******/ ({

/***/ "./entry.js":
/*!******************!*\
  !*** ./entry.js ***!
  \******************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("...// 相关代码");

/***/ }),

/***/ "./message.js":
/*!********************!*\
  !*** ./message.js ***!
  \********************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("...// 相关代码");

/***/ }),

/***/ "./name.js":
/*!*****************!*\
  !*** ./name.js ***!
  \*****************/
/*! exports provided: name, text */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("...// 相关代码");

/***/ })

/******/ });
```
初看下产物文件很杂乱，但其实它只是一个闭包。
```js
(function(modules) { // webpackBootstrap
  ...
  function __webpack_require__ (moduleId) {
    ...
    return module.exports;
  }
  ...
  return __webpack_require__(__webpack_require__.s = "./entry.js");
})({
  "./entry.js": (function(module, __webpack_exports__, __webpack_require__) {
    "use strict";
    eval("...// 相关代码");
  })
  ...
})
```
闭包的参数是各个依赖的代码，函数体主要定义了一个 require 函数 `__webpack_require__`。下面我们分两部分来说明。

### 1. require 函数 `__webpack_require__`

webpack 最重要的部分就是这个 require 函数。它定义了一个模块变量 module。
```js
function __webpack_require__(moduleId) {
  ...
  var module = installedModules[moduleId] = {
    i: moduleId, // 模块 ID
    l: false, // 该模块是否已被加载
    exports: {} // export 出去的代码
  };
  ...
}
```
这里使用 installedModules 来缓存已加载的模块，如果模块已缓存，则直接返回。
```js
// 模块缓存
var installedModules = {};
function __webpack_require__(moduleId) {
  // 检查模块是否已存在缓存
  if(installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }
  // 创建一个新模块（同时把它放入缓存）
  var module = installedModules[moduleId] = {
    i: moduleId, // 模块 ID
    l: false, // 该模块是否已被加载
    exports: {} // export 出去的代码
  };
  ...
}
```
require 函数接收一个 moduleId 参数，用来从闭包参数 modules 里获取对应的模块代码。模块代码被分别封装到一个匿名函数中，require 函数会执行这个模块函数，因此对应模块的代码也会被执行。
```js
// 模块缓存
var installedModules = {};
function __webpack_require__(moduleId) {
  // 检查模块是否已存在缓存
  if(installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }
  // 创建一个新模块（同时把它放入缓存）
  var module = installedModules[moduleId] = {
    i: moduleId, // 模块 ID
    l: false, // 该模块是否已被加载
    exports: {} // export 出去的代码
  };
  // 执行模块函数
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
}
```
可以看到，模块函数的上下文 this 是 `module.exports`，因此模块的上下文 this 指向的是它本身。

然后模块会被标记为已加载，最终返回模块 export 的内容。
```js
// 模块缓存
var installedModules = {};
function __webpack_require__(moduleId) {
  // 检查模块是否已存在缓存
  if(installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }
  // 创建一个新模块（同时把它放入缓存）
  var module = installedModules[moduleId] = {
    i: moduleId, // 模块 ID
    l: false, // 该模块是否已被加载
    exports: {} // export 出去的代码
  };
  // 执行模块函数
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
  
  // 标记模块为已加载
  module.l = true;
  
  // 返回模块 export 的内容
  return module.exports;
}
```
最后在闭包内加载入口文件即可。
```js
// 加载模块入口，并返回 exports
return __webpack_require__(__webpack_require__.s = "./entry.js");
```

### 2. 依赖模块
闭包的参数是一个 Object，存放了各个依赖模块。Object 的 key 是模块的 moduleId，值为 `import from` 的文件路径。Object 的 value 是模块函数，它接收三个参数，`module`，`__webpack_exports__`，`__webpack_require__`。
```js
({
    "./message.js": (function(module, __webpack_exports__, __webpack_require__) {
      "use strict";
      eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _name_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./name.js */ \"./name.js\");\n\r\n/* harmony default export */ __webpack_exports__[\"default\"] = (`hello ${_name_js__WEBPACK_IMPORTED_MODULE_0__[\"name\"]}!`);\n\n//# sourceURL=webpack:///./message.js?");
    })
})
```
我们来看看 `message.js` 生成的代码。因为使用了 `eval` 模式，webpack 生成的 `generated code` 被 `eval` 包裹。可以分为三个部分。
#### 2.1. esModule 定义
```js
__webpack_require__.r(__webpack_exports__);
```
`__webpack_require__.r(__webpack_exports__)` 会在 exports 里定义了一个 `__esModule` 字段，其值为 true，表明这是一个 esModule。
#### 2.2. \_\_webpack_require__ 获取依赖
```js
/* harmony import */ var _name_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./name.js */ \"./name.js\");
```
然后它通过 `__webpack_require__` 获取其它模块（`name.js`）的 exports。
#### 2.3. \_\_webpack_exports__ 暴露依赖
```js
/* harmony default export */ __webpack_exports__[\"default\"] = (`hello ${_name_js__WEBPACK_IMPORTED_MODULE_0__[\"name\"]}!`);
```
最后在自己的 exports 定义了一个 `default` 字段，其值为源代码中 `exports default` 的值。

## 三. 总结
可以看出，整个模块依赖是通过 `module.exports` 进行的。使用 `__webpack_require__` 函数来获取其它模块的 exports，如果需要向外部提供数据，则使用 `__webpack_exports__` 将数据暴露到 exports。