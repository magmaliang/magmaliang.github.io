---
layout: post
title:  "浏览器如何渲染页面及据此的性能优化"
date:   2016-11-23
author: 梁龙飞
tags: 编程
---

## 概要

浏览器渲染页面的具体步骤如下：

- DOM tree 的构建
- CSSOM tree 的构建
- Render tree 的构建: 合并 DOM tree 和 CSSOM tree
- Layout (我们常说的 reflow 就是与此有关): 从根节点开始计算 render tree 上每个 node 的盒模型。
- Paint: 将上一步计算出的每个节点绘制到屏幕上，像素化、栅格化

render tree 上都是可见的、将要被渲染的元素。

## repaint

很多地方说repaint仅指不带有回流行为的再次 paint,但实际上 repaint 就是重新执行 paint，跟有没有回流没有什么关系，它们是两个不同的概念。当页面需要渲染成跟之前不一样的时候，浏览器就会重绘。绝大多数reflow都会引起repaint,但是repaint不一定都是由reflow引起的，比如滚动页面，render tree 和 Laout 完全不需要任何计算，仅需要重绘即可。



## reflow

页面回流指触发上述第四个步骤（Layout）的行为，比如将一个元素脱离普通文档流，它的后续元素和其父元素都有可能需要重新计算盒模型。

引起回流的操作：

- 改变window大小
- 改变字体大小
- 改变height,width,position,display
- 增删节点
- 移动设备的横竖屏切换

并不是所有的reflow都会引起repaint，例如在一个很长的页面底部插入了一个元素（当前scrollTop为0），触发了reflow，但是页面无需repaint。

## 性能优化

根据浏览器呈现页面的步骤，可以做以下的性能优化（仅仅针对reflow,repaint并没有多少程序员可以介入的地方——而且阻止了reflow，则很多无效的repaint自然而然就优化掉了）：

1. 将包含动画的元素独立出元素较多的文档流，这样动画引发的 reflow 仅需要 layout 动画所在的文档流。
2. 需要频繁操作一个元素及其后代的css属性，引起多次reflow的，可以使用切换类的方式，或者将此元素隐藏，操作完之后再显示。生成元素也是如此，一次将元素生成完整再将其插入到dom中。
3. 如果对window绑定了resize事件，做一个debounce。
4. 避免使用table布局，除了table之外，其他的元素渲染都是流式的（从左到右，从上到下），后代元素的渲染不会影响前面元素的位置，但是table中各个cell的渲染会彼此影响。所以改变任何cell都有可能引起整个table的reflow,而且这种reflow计算的成本比遵守流式布局的元素大很多。
5. 不要频繁读取一个元素的 computed style，这类style是动态的，而且是lazy属性的，读取的时候会触发layout去计算它的值，比如offsetWidth等等。








## 参考

(render-tree-construction)[https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction]
(What are reflows and repaints and how to avoid them)[http://javascript.tutorialhorizon.com/2015/06/06/what-are-reflows-and-repaints-and-how-to-avoid-them/]