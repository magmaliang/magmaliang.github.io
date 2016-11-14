---
layout: post
title:  "js模块化规范——AMD,CommonJS,UMD"
date:   2016-11-05
author: 梁龙飞
tags: 编程
---

> 我在搜索js模块化关键词时，搜索到一篇非常通俗易懂的文章，[http://davidbcalhoun.com/2014/what-is-amd-commonjs-and-umd/](http://davidbcalhoun.com/2014/what-is-amd-commonjs-and-umd/)，我写不出更好的，所以写了个读博的笔记。

## 总结
AMD是等依赖全部加载好了，才构建输出模块，模块的定义中不包含依赖的加载逻辑。CMD的依赖加载写在模块定义中。UMD是兼容AMD和CMD的模块定义方式。

## AMD

AMD的语法如下：

> define(id?,deps?,factory)

deps中的依赖会索引到依赖模块的实体作为参数数组传入模块的定义factory中，factory要等依赖全部加载完才会执行。我读完requirejs的源码，写了一个简易的实现：(simple_requirejs)[/2016/10/31/simple-require-js.html]。

下面定义了一个foo模块，依赖于jquery:

```javascript
// filename: foo.js
define(['jquery'], function ($) {
    //methods
    function myFunc(){};
    //exposed public methods
    return myFunc;
});
```
以上的factory执行的时候首先要获得jquery模块的输出，并作为$传入到factory中作为参数。

## CommonJS

定义语法如下：

> define(factory(){
	var dep1 = require("dep1");
	var dep2 = require("dep2");
})


CommonJS号称 as lazy as possible。

下面同样定义一个foo模块，依赖于jquery:

```javascript
//filename: foo.js

//dependencies
var $ = require('jquery');

//methods
function myFunc(){};

//exposed public method (single)
module.exports = myFunc;
```

## UMD

AMD与CMD差不多一样流行，于是UMD这种能被这两种模块定义加载的定义出现了。

语法如下：

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    methods
    function myFunc(){};

    //    exposed public method
    return myFunc;
}));
```

其实就是判断环境是什么，然后输出对应的模块定义。












