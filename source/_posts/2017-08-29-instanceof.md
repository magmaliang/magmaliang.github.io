---
layout: post
title:  "js-原型"
date:   2017-08-29
author: 梁龙飞
tags: 编程
---

写这篇文章的原因，是因为对于“prototype”这个概念有些理解上的偏差。

发现偏差的契机来源于一次手贱写的代码。前几天我在控制台里面打开node的命令行环境，无聊写下下面一段代码：

```
var a = function (){};

var obj = {};

obj.__proto__.constructor = a;

obj instanceof a; // false
```

错误的理解就不说了，下面把基础概念过一遍，大概就能解决这个问题了。

## __proto__ 与 prototype

__proto__是js引擎实现时内部的属性，便于完成原型链式的访问，某些浏览器（例如chrome）暴露出来通过js可访问，es5开始有一个api可以获取对象的原型：Object.getPrototypeOf(O)。
所以所有具有原型链机制的对象均有__proto__属性。而prototype是函数的属性，当函数被用作constructor时用于构建__proto__。

构建时直接将__proto__指向constructor的prototype:
```js
var a = function(){};

(new a).__proto__ === a.prototype; // true
```

## 以function声明的函数是Function的实例

通过以下代码可以证明：
```js
var a = function(){};

a.__proto__ === Function.prototype; // true
```

注意 **a.prototype 与 a.__proto__** 并不是一回事，在上一节中已经说明: 只有函数才具有 prototype 这个属性，被用作constructor(使用new 构建)时，将新对象的 __proto__ 设置为函数的prototype。

## instanceof

> object instanceof constructor
instanceof 用于检测 constructor.prototype 是否存在于object的原型链上。所以回到开头的问题，instancof 检测的是 [[prototype]] 这个引用，与constructor无关。

## Object instanceof Object
这个表达式返回true，原因在于：

```js
Object.getPrototypeOf(Object.getPrototypeOf(Object)) === Object.prototype;
```

至于为什么会设计成这样，还没弄清楚。通过以下代码可以生成一个类似效果的函数；

```
function A(){};
function B(){};
var t = {};

A.prototype = B.prototype = t; 
A.__proto__ = new B();

A instanceof A // 返回true

```

## 终级答案

今天终于搞明白了，我在stackoverflow上问了一个问题，过了好久有个人回答了我，[问题](https://stackoverflow.com/questions/45936000/how-to-explain-object-instanceof-object-returns-true-in-javascript?noredirect=1#comment78831774_45936000)

原因并非是它们被设计成这样，这并非目的，而是因为Function和Object的定义（设计）规则，造成了这一结果:

**Object是个function, 所有的function都是对象，所有对象继承自Object.prototype**

即：

```js
Object instanceof Function // true
Function instanceof Object // true
```

所以 Function instanceof Function 也是true.









