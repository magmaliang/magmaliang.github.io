layout: post
title:  "node canvas"
date:   2018-01-04
author: 梁龙飞
tags: 编程
---

node-canvas是基于cairo用node实现的Canvas。

## 安装

图形图像库的依赖都比较复杂，node-canvas有一系列依赖，直接npm-install是不行的，需要经过以下预热：

[依赖安装](https://www.npmjs.com/package/canvas)

mac上可以使用brew直接安装：
> brew install pkg-config cairo libpng jpeg giflib

## 