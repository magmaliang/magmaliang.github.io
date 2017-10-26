---
layout: post
title:  "webpack打包优化"
date:   2017-05-09
author: 梁龙飞
tags: 编程
---

## 概述

程序的编译基本都是**链接转化合并压缩**，有这过程中有很多可以优化的地方。js虽然是轻量的脚本语言，但近几年的发展让它已经有了开发大型程序的能力，webpack 引入了后端高级语言的编译打包功能，其构建流程已经成为了前端的标准。对于后端同学，这没有什么神奇的地方，但对于前端，打包的逻辑还是挺新鲜的。本文总结webpack打包时一些的一些优化配置。

写了个demo说明以下所有的优化： [https://github.com/magmaliang/webpack-package-optimize](https://github.com/magmaliang/webpack-package-optimize)

## code splitting

代码切割分包，主要是为提取公共代码，优化加载。其主要分为 code-spilitting-cs 和 code-splitiing-js。

### css

如果将css和js包混在一起打成一个大包，则需要等待主包加载完才会插入js。这样主要有两个弊端：

1. 在css的模块加载完之前，界面上无法应用样式。
2. 没有利用浏览器并发加载资源的特性。css这种和js无关的资源其实分开加载较好，它并没有什么依赖。

extract-text-webpack-plugin 可以将所有css单独抽离打成一个包，配置如下：

```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin');

// module 的rules结点
rules: [
    {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
            use: 'css-loader'
        })
    }
    , {
        test: /\.scss$/,
        use: ExtractTextPlugin.extract({
            fallbackLoader: "style-loader",
            loader: "css-loader!sass-loader",
        })
    }
]

// plugins 结点 
plugins: [new ExtractTextPlugin('styles.css')]
```

### js

所有以 import('module_name').then 或者 require.ensure 引入的模块，webpack 在打包时会将其打成额外的 bundle。主包中会定义 webpackJsonP 函数，子包以 webpackJsonP 包裹。在代码运行的时候异步加载。

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

单独打包的子包结构

```javascript
webpackJsonp([0], {
    84:
        (function(module, exports, __webpack_require__) {
            module.exports = CmpB;
        })
});
```
主要的逻辑在 webpackJsonP 和 __webpack_require__.e 定义。

## externals

externals 可以让引用的模块不被打包，主要适用于这样的场景：多个bundle依赖于同一个common库，这些bundle在打包的时候并不能都将common打进去 —— 因为每个项目可能引入多个bundle，而这些bundle又是单独维护的。

externals 按如下的方式配置：
```javascript
// webpack.config.json
externals:{
    "jquery": "jQuery"
}
```

## CommonsChunkPlugin

externals 解决的问题和externals一样，如果不嫌麻烦，用 externals 也能解决 commonsChunkPlugin 能解决的问题。假设有这样的场景，非单页面的应用中，多个entry中有相当一些公用的代码，这些代码即非jQuery、React这类第三方常用的库，也非稳定的约定范围的代码。但是我们又希望把这部分代码独立出来，以便利用浏览器的缓存。

这就是commonsChunkPlugin 解决的问题。

```javascript
// 在webpack的配置中plugins数组中添加以下代码即可
new webpack.optimize.CommonsChunkPlugin(options)

// options有以下参数
{
    name: // 导出的bundle名，string or string[]
    ,filename: // 和output的filename命名规则一样
    ,minChunks: // 定义多少个entry共用才被定义为 commonsChunk，数质在[2,x]，x为entry的数量
}

```

## tree-shaking

这尼玛的不好翻译，就叫树筛选吧。webpack2 在打包时会给没有用的模块内部对象打上标记，然后在执行webpack命令的时候使用 --optimize-minimize 这个选项，压缩之后的代码便没有多余的模块。

在本项目中，入口文件 src/main.js 中只引用了 src/common/utils 的部分对象，执行 npm start 后观察 release下的main.js可以发现，只打包了utils中的部分代码。

ps: 和babel结合使用的时候比较恶心，请看这篇文章 [http://2ality.com/2015/12/webpack-tree-shaking.html](http://2ality.com/2015/12/webpack-tree-shaking.html)


## Scope Hoisting

Scope Hoisting 是 webpack3 的主要功能。他的思想是可以合并的module在打包时尽量合并，假设有以下代码：

>a 模块在 enrty 文件中只引用了一次，那么使用 Scope Hoisting 之后，a模块的代码会直接放在 entry 的 module中

它的使用很简单，只需在plugins结点中添加如下代码：

```javascript
new webpack.optimize.ModuleConcatenationPlugin()
```
PS: 请注意这是webpack3才有的功能



## 参考 
- [https://webpack.js.org/guides/code-splitting-css/#components/sidebar/sidebar.jsx](https://webpack.js.org/guides/code-splitting-css/#components/sidebar/sidebar.jsx)




