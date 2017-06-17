---
layout: post
title:  "使用jekyll搭建静态博客"
date:   2015-02-21 
author: "梁龙飞"
keywords: jekyll blog
tags: 工具
---


## 安装
jekyll 是 ruby 写的，通过 rubygem 安装，所以先安装 rubygem。

安装完了rubygem,执行 gem install jekyll 来安装 jekyll,极有可能因为 SSL 的原因报错。
下面这个链接解决了 SSL 的问题：[gem ssl的问题解决方法][1]

## 使用
这个年代什么第三方的搬运工都不如官方文档
[jekyll中文网][2]

## gitpages
初始化之后的 jekyll 项目 push 到同名 github pages 项目上就可以访问了，github 天然支持 jekyll,所以只管 push 完整的 jekyll 项目。

[1]:https://gist.github.com/fnichol/867550
[2]:http://jekyll.bootcss.com/