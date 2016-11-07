---
layout: post
title:  "chrome 缓存压缩--记一次脚本加载时间过长的分析"
date:   2016-10-31
author: 梁龙飞
tags: 编程
---

> 本文的结论在很久前就注意过，并且了解到浏览器的缓存压缩，但是没有记录、没有深入调研，不够了解自然不能运用自如。

## 问题

这个问题很早就出现，因为其他更紧急的事，没有分析。情况是这样的，我们有几个比较大的js脚本，平均在1MB~2MB之间（压缩前），在chrome下304的情况下加载都需要平均600毫秒的时间。

## 问题分析经过

> 以下测试数据皆在我自己的mac上进行，在其他机器上数据偏差量可能非常大。

### 一.是否是服务器原因

之所以会考虑这个原因，是因为我们使用了nginx rewrite模块，这还不算完，rewrite之后还要使用proxy_pass代理到node启的端口上，我害怕这条路太过曲折。所以我把一个大文件直接在nginx里location出去。

然后……chrome下<br/><br/>
<img src="/assets/img/chrome-200.png" alt="/assets/img/chrome-200.png" >


### 二.是否是网络原因

因为使用的是本地的静态资源服务器(127.0.0.1)，网络出现原因只能是到本地服务器的建连时间。但是我还是手贱地ping了下

<img src="/assets/img/pinglocal.png" style="width:70%;">

这个时间，业界良心，没得说了。

如何彻底绝了这念想，我只好触发200 from cache了。

chrome下的结果：

<img src="/assets/img/chrome-200cache.png" alt="/assets/img/chrome-200.png" >


完全跟网络无关的加载都如此耗时，这....绝逼是浏览器的原因了，为了证实此问题。

### 三.证明是chrome的原因

这时候我打开了尘封已久的safari,看了下资源304时的timeline.

<img src="/assets/img/safari-200cache.png" alt="/assets/img/chrome-200.png" >


尼玛，这数据似曾相识？<br/><br/>
<img src="/assets/img/ping.jpg"  width="30%">

我又浪费公司带宽去下载了一个80MB的firefox,304缓存时间如下：<br/><br/>
<img src="/assets/img/firefox_304.png">

### 四.理论依据

作为一个程序员，仅仅用事实说话是不行的……还得用代码说话。。。只可惜。。
我看不懂chrome的代码，那时候整天哔哔哔微软不开源，现在开源的也看不懂，尴尬。所以又去读了几遍chrome timing相关的[文档](https://developers.google.com/web/tools/chrome-devtools/network-performance/resource-loading#view-network-timing-details-for-a-specific-resource)，还是没搞明白。

只好low爆地去搜，“Chrome spend too much time downloading resourece from cache”，结果只找到了这一篇[Why does Chrome spend time “downloading” content from cache?](http://stackoverflow.com/questions/33250802/why-does-chrome-spend-time-downloading-content-from-cache),这个回答指引我找到了最终的答案，但只有可怜的3个赞，我顺手一点，结果说我声望不够，不准我点赞，于是我又google如何获取声望~~~。

这位兄台说到：

> You'll also notice that Chrome is spending time "downloading" every file. Why is this? Well, Chrome's cache is a database, and that database is also compressed to save space. When you retrieve a document from cache, the price of that retrieval is not zero. Chrome has to look up the item in the cache database, and then inflate that entry into memory so Chrome can work with it. I don't know the exact details concerning how the Network chrome-dev-tools panel shows the times, but I would guess that getting that file from disk, uncompressing it, and then parsing and working with the result is what you're seeing reflected in "Time downloaded."

进一步搜索找到这篇[文章](https://www.stevesouders.com/blog/2012/03/27/cache-compressed-or-uncompressed/)。可以看到chrome,firefox和Opera进行了压缩，IE和Safari没有。这说明了我上面测试的结果，从safari加载304缓存的速度几乎和ping的速度一样。chrome相比于firefox，解压加载的速度几乎慢了两个数量级，这应该是chrome的压缩率更高，所以解压速度更长。



## 结论

> Chrome压缩了缓存，从缓存中加载需要解压 —— No free lunch.

<script>
	var imgs = document.images;
	for(var i =0;i<imgs.length;i++){
		imgs[i].addEventListener("mousedown",function(e){
			var str = e.target.getAttribute("src");
			window.open(str,"_blank")
		})
	}
</script>