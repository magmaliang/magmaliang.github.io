---
layout: post
title:  "简述AMD的实现库: requirejs"
date:   2016-10-31
author: 梁龙飞
tags: 编程
---


requirejs是AMD的一种实现，主要通过按依赖顺序向html中插入scripts节点来实现模块的依赖加载，它也支持在web works中使用（当然不是insert node）。

## 原理
requirejs先加载入口文件，如果入口文件有依赖，则加载这些依赖文件，如果依赖还有自己的依赖...直到加载完所有的依赖。requrejs的define函数用来定义可加载的js模块，它的语法是这样的：

```javascript
define(id?,deps?,factory)
```

### requirejs的内部变量
id 和 deps都不是必选的，factory才是模块的内容。那么define到底做了什么事呢？这得说到requirejs内部的一些对象，下面的代码是requirejs一开始定义的变量，它秉承了先声明后使用的原则把内部所用的变量一股脑声明在了函数内部的最前面：

```javascript
 var req, s, head, baseElement, dataMain, src,
        interactiveScript, currentlyAddingScript, mainScript, subPath,
        version = '2.3.2',
        commentRegExp = /\/\*[\s\S]*?\*\/|([^:"'=]|^)\/\/.*$/mg,
        cjsRequireRegExp = /[^.]\s*require\s*\(\s*["']([^'"\s]+)["']\s*\)/g,
        jsSuffixRegExp = /\.js$/,
        currDirRegExp = /^\.\//,
        op = Object.prototype,
        ostring = op.toString,
        hasOwn = op.hasOwnProperty,
        isBrowser = !!(typeof window !== 'undefined' && typeof navigator !== 'undefined' && window.document),
        isWebWorker = !isBrowser && typeof importScripts !== 'undefined',
        //PS3 indicates loaded and complete, but need to wait for complete
        //specifically. Sequence is 'loading', 'loaded', execution,
        // then 'complete'. The UA check is unfortunate, but not sure how
        //to feature test w/o causing perf issues.
        readyRegExp = isBrowser && navigator.platform === 'PLAYSTATION 3' ?
                      /^complete$/ : /^(complete|loaded)$/,
        defContextName = '_',
        //Oh the tragedy, detecting opera. See the usage of isOpera for reason.
        isOpera = typeof opera !== 'undefined' && opera.toString() === '[object Opera]',
        contexts = {},
        cfg = {},
        globalDefQueue = [],
        useInteractive = false;
```

除了contexts之外，其他的类似于程序设计中的静态单例，或是辅助空间，所有的逻辑还是隐藏在了contexts中。在这个2.3.2的版本中，用来生成contexts实例的newContext函数从200行开始，直到1798行结束。。。所以没办法，我们打开newContext函数，把它的变量也扒出来。

### newContext

```javascript
var inCheckLoaded, Module, context, handlers,
            checkLoadedTimeoutId,
            config = {
                //Defaults. Do not set a default for map
                //config to speed up normalize(), which
                //will run faster if there is no default.
                waitSeconds: 7,
                baseUrl: './',
                paths: {},
                bundles: {},
                pkgs: {},
                shim: {},
                config: {}
            },
            registry = {},
            //registry of just enabled modules, to speed
            //cycle breaking code when lots of modules
            //are registered, but not activated.
            enabledRegistry = {},
            undefEvents = {},
            defQueue = [],
            defined = {},
            urlFetched = {},
            bundlesMap = {},
            requireCounter = 1,
            unnormalizedCounter = 1;
```

通过官方文档我们知道入口文件用 requirejs.config 来启动的，这个函数执行时即开始加载依赖，我们把这个入口文件叫做依赖树的根——尽管存在一个文件可能被好几个上级资源依赖的情况，这里为叙述方便仍称其为树。除了根之外节点都会被构建为Module（newContext内部定义的模块对象）对象，临时存放于**enabledRegistry**这个变量中。需要注意的是，只是刚刚激活的模块才会在这个变量中短暂停留，随后就会被清除，它的作用像是一个buffer。

### define函数的作用

我们回到前面的问题，define到底做了什么？

define将模块格式化为一个数组：

```javascript
[id,deps,factory]
```

并将其 push 进全局变量 globalDefQueue 中 —— 上面说过，这是个缓冲池。然后contexts会从其中拿数据并构建为Module对象存放于自己内部变量中,Module对象主要模式如下：

```javascript
{
    exports:define中factory执行返回的对象，如果factory不是可执行函数则直接返回factory
    ,factory:即define中的factory
}
```

当此Module被上层函数依赖时，则返回其exports到上层函数的arguments中，执行这个操作的关键函数为：

```javascript
 /**
 * Executes a module callback function. Broken out as a separate function
 * solely to allow the build system to sequence the files in the built
 * layer in the right sequence.
 *
 * @private
 */
execCb: function (name, callback, args, exports) {
    return callback.apply(exports, args);
}
```

## 简单的requirejs实现

好了，专业库因为要考虑很多方面，不容易直接洞察其主要逻辑。其实我些年的经验时，需求是最好的文档，当你了解需求，直接读源码就行了——甚至直接去造轮子了。下面我们就造个劣质小轮子 —— 这是了解一个库最好的方式了。

### 源码

```javascript
var define,require;
require = function(options){
	let {entry,baseUrl} = options;
	//存放所有生成的模块
	var modules  = {},
		unLoadedModules = [];//每当有script_onload触发时，则shift一个模块，生成Module

	//根据id加载一个模块，数据结构如下
	function loadModule(id){
		var node = document.createElement('script');
        node.setAttribute('data-modid', id);
        node.src = baseUrl+id+".js";

    	node.addEventListener('load', function(e){
    		//当前脚本加载的模块名
    		var id = e.target.getAttribute("data-modid");
    		completeLoaded(id);

    		//对所有可以导出export的模块进行导出
    		tryLoading()
    	}, false);

    	document.head.appendChild(node);
	}

	/*   *   *   *   *   *   *   *   *   *   *
	 *  Module用于构建存放于modules中的模块实例    
	 *   *   *   *   *   *   *   *   *   *   */
	function Module(id,deps,factory){
		this.id = id;
		this.deps = deps;//["a","b"]
		this.factory = factory;
		this.loaded = false;
		this.export = null;
	}

	Module.prototype = {
		getExport:function(){
			//如果export不存在，则构建
			if (!this.export) {
				var _args = this.deps.map(item=>{
					return modules[item].export;
				})
				this.export = this.factory.apply(null,_args);
				this.loaded = true;
			}
			return this.export;
		}
	}

	//当一个远程脚本完成加载时,从unLoadedModules中左移出一个模块.
	//将此模块格式化成Module对象，存放于modules中
	function completeLoaded(id){
		if (unLoadedModules.length) {
			var mod = unLoadedModules.shift();
			mod[0] = id;

			modules[id] = new Module(mod[0],mod[1],mod[2]);

			//如果是无依赖模块，则立即得到它的export()
			if (mod[1].length == 0) {
				modules[id].getExport()
			}
		}	
	}

	//尝试对modules中依赖加载完成的模块生成export
	function tryLoading(){
		var _modsname = Object.keys(modules);

		_modsname.map(modname=>{
			var __Mod = modules[modname];

			var deps = __Mod.deps;
			var canExport = true;

			if (deps.length>0) {
				//检测是否所有依赖都已完成加载
				deps.map(depname=>{
					if (!modules[depname] || modules[depname].loaded == false) {
						canExport = false;
					}
				})
			}

			if (canExport && __Mod.loaded == false) {
				__Mod.getExport()
			}
			
		})
	}

	//两个参数都是必须的
	define = function(deps,factory){
		unLoadedModules.push([null,deps,factory]);
		if (deps.length>0) {
			deps.map(_mod=>{
				loadModule(_mod);
			})
		}
	}

	//从入口启动
	loadModule(entry)

	function waitForEntry(){
		if (modules[entry] && modules[entry].loaded == false) {
			tryLoading();
		}else{
			setTimeout(waitForEntry,50)
		}
	}

	waitForEntry()
}
```

实现requirejs有几个重要的点

- 在根据名称加载依赖时，依赖的define未知。即依赖的名称如何与返回的脚本定义绑定。当然可以在定义的时候把id也写上，但是这样就需要跟文件名再来一次绑定，增加了复杂度。上面这个例子中利用插入script标签的顺序，和这些标签的onload事件触发的顺序完成一致这个特性来完成绑定的。**如果当中有标签加载失败，则会导致它们的顺序错乱而对应不上。**
- 其次只有一个模块的依赖全部加载完，才能执行factory生成export。上面这个简单的例子，是每次有一个模块load完的时候，就检测所有模块是否具有执行factory的条件。这显然不是一个好方法。而且最后的入口模块无法检测（所以我写了个循环，waitForEntry）。


### 使用

这个劣质轮子的使用非常简单，我写了个[demo](https://github.com/magmaliang/simple_require),一看便知。

### 


