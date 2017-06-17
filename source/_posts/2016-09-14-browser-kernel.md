---
layout: post
title:  "浏览器内核"
date:   2016-09-14
author: 梁龙飞
keywords: js,浏览器内核
tags: 编程
---

## 概念

程序语言的UI模块向来比较复杂，且因为依赖操作系统的具体呈现，在跨平台下更为难搞。C 语言就是因为如此一直没有UI模块。处理数据的模块可以抽象，可以运用各种高阶技艺，但是呈现出的UI相对来说比较具体，如何设计的优雅简单，我个人觉得需要的智慧不亚于操作系统（KDE的代码量已经达到400万行，linux内核有370万行）。

和其他规模大到一定程度需要标准和流程一样，UI也是如此。无论是 cs 还是 bs，在 UI 的呈现上其实有很多共通之处，就像各种程序语言之间的关系一样。桌面 UI 与浏览器的 UI(html css) 之间有着基本一致的设计。例如基于 KDE 的 webcore(webkit中的排版模块) 被选为 chrome 和 safari 的排版引擎，但是它最初是为了 *inux 的桌面系统而设计的。

java 在操作系统之上铺一层 jvm 实现跨平台，对于 web 程序来讲，浏览器就是它们的虚拟机。**而浏览器内核就是呈现网页应用的核心模块，它一般包括两部分 —— 渲染排版模块和 js 解析模块，即渲染引擎和 js 引擎**。

## 内核和引擎的区别

_我在 google 里搜索 brower kernel 并没有结果。_

中文社区经常会说到浏览器内核，然而内核与渲染引擎经常混用，我一直傻傻分不清，极度混淆，于是在今天使劲撸了一把，得出结论：

>**由于web出现的早期是没有js的，内核的功能仅仅是排版渲染，所以渲染引擎的模块一开始与内核是等价的，后来再加入的js模块多数有自己的模块名。—— 所以渲染引擎的名称几乎与内核一样，但是内核包含一些有自己名称的 js 引擎**


## 渲染引擎分类(rendering engine or layout engine)

> 读取标记语言(HTML,XML)和一些格式信息排版渲染成UI界面的程序， 我们平时所说的浏览器引擎一般指渲染引擎。

<table>
	<thead>
		<tr><th style="width:100px">名称</th><th>简介</th></tr>
	</thead>
	<tbody>
		<tr><td>Trident</td><td>IE 开发使用的内核。360 早期仅使用 Trident，后来又引入 chromium 成为双内核浏览器。</td></tr>
		<tr><td>Webkit</td><td>起源于 KDE。webkit 是目前最屌的内核，chrome 和 safari 以及几乎所有的移动端浏览器都是在 webkit 的的基础上衍生的。chrome 28+之后，使用基于webkit开发的 <a href="https://en.wikipedia.org/wiki/Blink_(web_engine)">Blink</a></td></tr>
		<tr><td>Gecko</td><td>由 Netscape 开发。继承衣钵的 FirFox 自然采用了此内核。</td></tr>
		<tr><td>Presto</td><td>Opera 浏览器采用的渲染引擎，不过 opera software 在2013年已经放弃它了，决定使用webkit + v8。</td></tr>
	</tbody>
</table>

## js 引擎分类
> 读取执行 JavaScript 代码的程序。

<table>
	<thead>
		<tr><th style="width:200px">名称</th><th>采用程序</th><th>备注</th></tr>
	</thead>
	<tbody>
		<tr><td>V8</td><td>Google Chrome，Node.js,V8.NET</td><td></td>chrome使用的渲染引擎是webkit</tr>
		<tr><td>Chakra(JScript engine)</td><td>IE9~IE11</td><td>MIT 开源</td></tr>
		<tr><td>Chakra(JavaScript engine)</td><td>Microsoft Edge</td><td>对你没有看错，微软就是样子多,这是JScript engine的一个fork，MIT开源</td></tr>
		<tr><td>Carakan</td><td>Opera</td><td>2013年 Opera15 正式采用 V8,放弃Carakan</td></tr>
		<tr><td>SpiderMonkey</td><td>FireFox</td><td></td></tr>
		<tr><td>JavaScriptCore</td><td>大名鼎鼎的webkit内核使用的js引擎</td><td></td></tr>
		<tr><td>Tamarin</td><td>Adobe Flash</td><td></td></tr>
		<tr><td>Nashore</td><td>JDK 8+</td><td></td></tr>
	</tbody>
</table>


#### 参考资料
* [https://en.wikipedia.org/wiki/Comparison_of_layout_engines_(HTML)](https://en.wikipedia.org/wiki/Comparison_of_layout_engines_(HTML))
* [https://en.wikipedia.org/wiki/Web_browser_engine](https://en.wikipedia.org/wiki/Web_browser_engine)
* [https://en.wikipedia.org/wiki/List_of_ECMAScript_engines](https://en.wikipedia.org/wiki/List_of_ECMAScript_engines)




