---
layout: post
title:  "自适应设计 vs 响应式设计"
date:   2016-09-18 
author: "梁龙飞"
keywords: 响应式,自适应,web设计
tags: 设计
---

![jif](/assets/img/RWD_vs_AWD.gif)

> 这两个词都是随着移动端的爆发而火起来的。pc时代使用固宽和留白基本搞定了一切。移动设备尺寸众多且与pc端有本质区别，需要专业的技术和理论来构建。

## 自适应设计(adaptive web design，AWD)

> 自适应设计针对特定的设备，使用媒体查询技术，应用特定的样式。这里的适应指网页适应设备，并非指元素适应容器宽度。

AWD 写多套特定尺寸的样式，可以对元素使用固定宽度。设备加载页面时只加载其中一套样式。因此，如果使用模拟的尺寸（比如用pc端浏览器模拟移动端），那么在变化尺寸时，样式不再发生变化。

一些老的站点，比如只有pc端的，因为不想动旧的代码使用AWD是个不错的选择。因为是重新设计，使用AWD的网页应用在PC端和移动端上看起来可能差别非常之大 —— 自适应设计可能针对不同的设备尺寸，尝试展现更合理的内容和布局，这包括元素和功能的缺失增加。

实际上 AWD 主要就是为老 PC 站点提供非 PC 端页面。



## 响应式设计(reponsive web design,RWD)

> 响应式设计主要针对屏幕宽度范围应用样式。很显然相对于 AWD 的点式针对，RWD是连续的。

RWD 强调 fluid 和 flexibility，在页面宽度的变化过程中可以看到元素的排列变化。与 AWD 相比，它更强调元素在不同尺寸范围下的“响应”，不会有元素和功能的缺失增多，它们在内容上保持着高度的一致 —— 它只有一套样式一种设计，因为样式是流式和伸缩的，才在各种设备上变形。

说 RWD 是一套设计有时候并不严格，因为在一些局部，比如导航条和列表上，实际是有不同的微设计，只是在整体上它基本还是一套样式，多套设备上的展现有非常高的相关相似性。


## 组合使用

好的方案是多方相辅相成、互相妥协的合成物，超级纯粹的东西一般是为了研究需要。实际上媒体查询针对性地写样式，在流式伸缩性技法为主的 RWD 中也经常使用，比如上面提到的导航条处理：在空间充足的情况下横向排列，在空间逼仄时变为竖向。



## 参考
- [https://css-tricks.com/the-difference-between-responsive-and-adaptive-design/](https://css-tricks.com/the-difference-between-responsive-and-adaptive-design/)
- [https://www.interaction-design.org/literature/article/adaptive-vs-responsive-design](https://www.interaction-design.org/literature/article/adaptive-vs-responsive-design)
