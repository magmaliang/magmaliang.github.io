---
layout: post
title:  "webpack打包优化"
date:   2017-05-09
author: 梁龙飞
tags: 编程
---

> 这里先介绍 code splitting，先说一些关键函数，再串原理。未完待续。。。

## code splitting 

### 打包完关键代码

使用 require.ensure 或者 import().then的代码，经过 webpack 打包，会生成以下结构的代码。其实如果读过requirejs的源码，会发现它们的逻辑基本是一样的。但webpack实现起来简单点，因为在编译的时候已经为异步的模块占位了chunkId。

主包中定义了 webpackJsonP，用以装载子包到全局包数组中去。源码的注释写的比较好。
```javascript
// 主包的代码结构
(function webpackUniversalModuleDefinition(root, factory) {
    if (typeof exports === 'object' && typeof module === 'object')
        module.exports = factory();
    else if (typeof define === 'function' && define.amd)
        define([], factory);
    else {
        var a = factory();
        for (var i in a)(typeof exports === 'object' ? exports : root)[i] = a[i];
    }
})(this, function() {
    return (function(modules) {
    		// install a JSONP callback for chunk loading
            var parentJsonpFunction = window["webpackJsonp"];
            window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
                // add "moreModules" to the modules object,
                // then flag all "chunkIds" as loaded and fire callback
                var moduleId, chunkId, i = 0,
                    resolves = [],
                    result;
                for (; i < chunkIds.length; i++) {
                    chunkId = chunkIds[i];
                    if (installedChunks[chunkId]) {
                        resolves.push(installedChunks[chunkId][0]);
                    }
                    installedChunks[chunkId] = 0;
                }
                for (moduleId in moreModules) {
                    if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
                        modules[moduleId] = moreModules[moduleId];
                    }
                }
                if (parentJsonpFunction) parentJsonpFunction(chunkIds, moreModules, executeModules);
                while (resolves.length) {
                    resolves.shift()();
                }
            };
            // The module cache
            var installedModules = {};
            // objects to store loaded and loading chunks
            var installedChunks = {
                2: 0
            };
            // The require function
            function __webpack_require__(moduleId) {
                // Check if module is in cache
                if (installedModules[moduleId]) {
                    return installedModules[moduleId].exports;
                }
                // Create a new module (and put it into the cache)
                var module = installedModules[moduleId] = {
                    i: moduleId,
                    l: false,
                    exports: {}
                };
                // Execute the module function
                modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
                // Flag the module as loaded
                module.l = true;
                // Return the exports of the module
                return module.exports;
            }
            // This file contains only the entry chunk.
            // The chunk loading function for additional chunks
            __webpack_require__.e = function requireEnsure(chunkId) {
                if (installedChunks[chunkId] === 0) {
                    return Promise.resolve();
                }
                // a Promise means "currently loading".
                if (installedChunks[chunkId]) {
                    return installedChunks[chunkId][2];
                }
                // setup Promise in chunk cache
                var promise = new Promise(function(resolve, reject) {
                    installedChunks[chunkId] = [resolve, reject];
                });
                installedChunks[chunkId][2] = promise;
                // start chunk loading
                var head = document.getElementsByTagName('head')[0];
                var script = document.createElement('script');
                script.type = 'text/javascript';
                script.charset = 'utf-8';
                script.async = true;
                script.timeout = 120000;
                if (__webpack_require__.nc) {
                    script.setAttribute("nonce", __webpack_require__.nc);
                }
                script.src = __webpack_require__.p + "" + chunkId + ".bundle.js";
                var timeout = setTimeout(onScriptComplete, 120000);
                script.onerror = script.onload = onScriptComplete;

                function onScriptComplete() {
                    // avoid mem leaks in IE.
                    script.onerror = script.onload = null;
                    clearTimeout(timeout);
                    var chunk = installedChunks[chunkId];
                    if (chunk !== 0) {
                        if (chunk) {
                            chunk[1](new Error('Loading chunk ' + chunkId + ' failed.'));
                        }
                        installedChunks[chunkId] = undefined;
                    }
                };
                head.appendChild(script);
                return promise;
            };
            // expose the modules object (__webpack_modules__)
            __webpack_require__.m = modules;
            // expose the module cache
            __webpack_require__.c = installedModules;
            // identity function for calling harmony imports with the correct context
            __webpack_require__.i = function(value) {
                return value;
            };
            // define getter function for harmony exports
            __webpack_require__.d = function(exports, name, getter) {
                if (!__webpack_require__.o(exports, name)) {
                    Object.defineProperty(exports, name, {
                        configurable: false,
                        enumerable: true,
                        get: getter
                    });
                }
            };
            // getDefaultExport function for compatibility with non-harmony modules
            __webpack_require__.n = function(module) {
                var getter = module && module.__esModule ?
                    function getDefault() {
                        return module['default'];
                    } :
                    function getModuleExports() {
                        return module;
                    };
                __webpack_require__.d(getter, 'a', getter);
                return getter;
            };
            // Object.prototype.hasOwnProperty.call
            __webpack_require__.o = function(object, property) {
                return Object.prototype.hasOwnProperty.call(object, property);
            };
            // __webpack_public_path__
            __webpack_require__.p = "./release/";
            // on error function for async loading
            __webpack_require__.oe = function(err) {
                console.error(err);
                throw err;
            };
            // Load entry module and return exports
            return __webpack_require__(__webpack_require__.s = 85);
        })
        (
        	// 所有chunk，传入形参 modules 中
        	[
	            /* 0 */
	            /***/
	            (function(module, exports) {
	            }),
	            /* 1 */
	            /***/
	            (function(module, exports, __webpack_require__) {
	            }),
	            /* 2 */
	            /***/
	            (function(module, exports, __webpack_require__) {
	            }),
	            ...
        	]
        );
});
```

单独打包的子包结构
```
webpackJsonp([0], {
	84:
		(function(module, exports, __webpack_require__) {
			module.exports = CmpB;
		})
});
```

## 原理

所有以 import('module_name').then 或者 require.ensure 引入的模块，webpack 在打包时会将其打成额外的 bundle。

主包中会定义 webpackJsonP 函数，子包以 webpackJsonP 包裹。

编译前后的异步引入 module 的代码如下：

```javascript
// 编译前
require.ensure(['./App','./cmps/cmpa'], function() {
	var App = require('./App');
	var CmpA = require('./cmps/cmpa');
	ReactDom.render(
		<div>
			<CmpA />
			<App />
		</div>
		, 
		document.getElementById('container'));
})
// 编译后
__webpack_require__.e /* require.ensure */ (0).then((function() {
    var App = __webpack_require__(51);
    var CmpA = __webpack_require__(52);
    _reactDom2.default.render(_react2.default.createElement(
        'div',
        null,
        _react2.default.createElement(CmpA, null),
        _react2.default.createElement(App, null)
    ), document.getElementById('container'));
}).bind(null, __webpack_require__)).catch(__webpack_require__.oe);
```

观察以上代码会发现，编译后的引入已经明确了 moduleId，说明 CmpA 和 App 在主 bundle 的编译里已经占坑。以 webpackJsonP 包裹的子包中有类似 {51: CmpA, 52: App} 这样的 Map，webpackJsonP 就是按照这个 Map 把相应 module 复制进 webpack 的 **modules** 数组中相应位置 —— **modules** 包含所有包，__webpack_require__ 函数即从此 数组中读取 module 的内容。在复制的同时，执行对应 installedChunks 的 resolve, installedChunks 返回的都是 **Promise**。这样 __webpack_require__.e(xxx).then 就会执行。




