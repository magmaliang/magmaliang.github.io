---
layout: post
title:  "直出"
date:   2017-05-19
author: 梁龙飞
tags: 编程
---


## 概述

直出是前端性能优化的一种方案，主要思想是在后端直接请求页面中所需的数据，完成部分或全部渲染后再传给前端。它减少的时间主要来自于两次或者多次http请求的建联时间：尤其对于如今前后端分离方式造成的性能问题——先把html和js发送到客户端，然后js再请求接口，拿到数据后和模版合并渲染。

## 两种直出方案

直出分为以下两种，逐渐增强。

### A. 数据直出

后端将页面渲染所需要的数据取出，但并不执行js渲染页面，取出的数据存放在页面内——插入全局js对象，或者是直接存放在Dom节点上。


### B. 服务端渲染

服务端渲染完整的页面。

## 总结

这种优化一般针对功能复杂的首页，入口页，首先让页面渲染出来，达到可交互的状态。

## 参考
[https://segmentfault.com/a/1190000005641012](https://segmentfault.com/a/1190000005641012)
[http://web.jobbole.com/86412/](http://web.jobbole.com/86412/)