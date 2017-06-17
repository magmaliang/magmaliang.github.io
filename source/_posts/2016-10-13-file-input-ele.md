---
layout: post
title:  "file input 元素"
date:   2016-10-13
author: 梁龙飞
tags: 编程
---

> <input type="file" />

## 特点

file type 的input元素与其他类型不一样：它的value值通过js只能设置为空string。

在ie8-ie10中，input(type=file)一直将value值设计为readonly，就是不能改，用户上传了一个文件，取消都不行——只能通过form的reset。IE 11+,chrome和firefox 允许通过脚本的方式改变 file_input 的 value 值为""(空字符串)，区别是IE会触发change事件，其它浏览器不会。可以看出IE的设计非常讲究逻辑，而不是易用和习惯——这在很多IE诟病的地方都得到体现。

## 为何不能通过脚本改上传元素的value值？

首先是逻辑问题，js并没有访问硬盘的能力，那么它就不知道有些什么文件，既然这样用js去更改上传文件元素的value值没什么意义。

然后就是安全问题，脚本瞎尼玛猜+改了一个文件名，上传了不想被传的文件。其实目前几乎所有新版本的浏览器在上传的时候，都不会暴露本地文件目录。一般value值会变成"c:/fakepath/文件名"。

## 为什么可以被设置为空值？










