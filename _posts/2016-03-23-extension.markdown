---
layout: post
title:  "扩展机制"
date:   2016-03-21
author: "UI框架开发团队"
categories: frame
---
### 概述
本章描述的是当前CoreBone（暂时就叫CoreBone吧，我们正在设计实现更棒的框架）实现扩展的方式，非常简单，并没有什么高深的东西！

### 框图
![img][1]

### 框架如何检索扩展资源
```javascript
Corebone.AppRetriever = (function(){
	//上下文资源检索
	this.contextRetriever = function(key){
		if (!Corebone.Extension) {return null};
		var _target = Corebone.Extension.Contexts.FormContext[key];
		if (_target) {
			return _target;
		};
		return;
	};
	
	//formList检索
	this.formListRetriever = function(key){
		if (!Corebone.Extension) {return null};
		var _target = Corebone.Extension.Contexts.FormList[key];
		if (_target) {
			return _target;
		};
		return;
	};

	//顶层组件资源检索
	this.pageRetriever = function(key){
		if (!Corebone.Extension) {return null};
		var _target = Corebone.Extension.Cmps.TopPages[key];
		if (_target) {
			return _target;
		};
		return;
	};

	return this;
})()
```

### 组件加载扩展资源(伪代码描述)

```javascript
//组件已经渲染完成 ，并且此组件有扩展标记
if(cmp.displayed && cmp.extended){
	//以此组件的id去corebone.Extension中寻找隶属的扩展资源
	//再次强调是伪代码，这个传入的cmp.id只是能标志cmp唯一性的概念，并不是实际的参数
	var ext = Corebone.AppRetriever(cmp.id);
	//执行扩展代码
	ext.apply(this.context);
}

```

### 流程图
![flow][2]

[1]:/img/extension.png
[2]:/img/flow_extend.png