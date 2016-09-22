---
layout: post
title:  "setTimeout 与 js单线程"
date:   2016-09-19
author: 梁龙飞
tags: js
---

## 预备知识
js是单线程的，所有的代码只会在一个线程上运行，串行执行任务。

setTimeout在window环境下执行，this总是指向window(无论严格模式还是普通模式)。setTimeout会将作为参数传入的代码塞到js执行线程上，预设的时间过后再执行。但是要注意,**并不是在setTimeout预设的时间后，一定轮到其塞入的代码执行:**

```javascript
setTimeout(fooa,100);
setTimeout(foob,200);
```
<img src="/assets/img/settimeout.png" alt="" style="width:70%">

如图，假设fooa执行的时间超过100ms，那么纵使执行流上预定了在200ms之后执行foob也没用，必须等fooa执行完。我们可以检验下：
```javascript
function wait(t){
	return ()=>{
		console.log("start wait "+t+":"+(Date.now()-startT))
		var start = Date.now();

		while(1){
			if(Date.now() - start > t){
				break
			}
		}
		console.log("end wait "+t+":"+(Date.now()-startT))
	}
	
}
var startT = Date.now()

fooa = wait(2000);
foob = wait(10);

setTimeout(fooa,100);
setTimeout(foob,200);

```
在我的电脑上执行结果如下：
```
start wait 2000:104
VM4557:11 end wait 2000:2106
VM4557:3 start wait 10:2107
VM4557:11 end wait 10:2118
```
可以看到，foob 并不是 200ms 后执行，它等 fooa 执行完了才执行。因为 fooa 在 js 执行线程上"预约"的时间早于foob;

## 实际的例子
今天稍有时间就查看了项目一直报错的一个bug，因为这个bug不影响使用，所以一直没人管。而我有强迫症，总希望控制台什么异常警告都没有。

实际的例子较为复杂，这里在不丧失细节的情况做了抽象（原来的是react的项目）:

```javascript
//一个组件的定义
class cmp {
	constructor(props){
		this.dom = {}
	}

	willReceiveProps(props){
		setTimeout((function(){
			this.dom.a=1
		}).bind(this),0)
	}
	willUnmount(){

	}
	_close(){
		delete this.dom;
	}
}
//一个移除组件的方法，模仿react移除组件
//在移除组件之前，假设触发了willReceiveProps——
//实际上在复杂的应用中这是不可避免的，这个方法总是被不断地触发,如果你写了这个方法的话
function removeCmp(cmp){
	cmp.willRecieveProps();
	cmp._close();
}

removeCmp(new cmp())
```

这个代码是会报错的，**虽然setTimeout的时间设置为0，但是推到执行流上的函数并没有立即执行，反而先执行了_close**。原因在于（如下图）：

<img src="/assets/img/20160921-01.png" alt="" style="width:70%">

我们先调用的是removeCmp(上图山楂红部分)，_close是这个函数体的一部分，这玩意没执行完，如何轮到其内部setTimeout推上去的代码？






