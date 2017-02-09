---
layout: post
title:  "使用nginx和webpack多开项目"
date:   2017-01-16
author: 梁龙飞
tags: 编程
---

## 问题背景

由于主项目及其依赖库同时处于快速开发中，需要同时调试。然而依赖又不会打包进主项目，这导致本地开发时只能通过类似charles map的方式应用更新。然而这样就需要将依赖库编译生成最终包的时候才能应用更新，它需要放到实际的磁盘上。——文件更新到磁盘的成本非常高昂。


## 解决方案

使用 webpack-dev-server 将不同项目的资源启动到本地的不同端口，再使用nginx将不同的域名代理到不同的端口。

## 例子

假设我们有两个项目如下：

```
-- Main
部署地址：www.rs.com/Main/release/dist/main-a23k.bundle.js

-- Sub
部署地址：www.rs.com/Sub/release/dist/sub-a2dd.bundle.js
```

> Main的配置

使用 **webpack-dev-server** 将 **Main** 的资源直接启动到本地 **localhost:3000/main_local/** 下。

```
{
	output:{
		fileName: "main.bundle.js"
		,libraryTarget:"umd"
		,path:path.resolve(__dirname,"./dist/")
	}
	,devServer:{
		contentBase:path.resolve(__dirname,"./dist/")
		,port:"3000"
		,publicPath:"main_local"
	}
}

```

nginx 中将部署地址重写到 **webpack-dev-server** 启动的端口上

```
# 将/Main/下的js访问url全部重写
rewrite /Main(.*)/(?P<filename>(.*))\.js /main_local/$filename.js

# 重写后的url再代理到 webpack-dev-server启动的端口上
location ~ /main_local/(.*)\.js {
	poxy_pass http://localhost:3000
}

```


> Sub 的配置

使用 **webpack-dev-server** 将 **Main** 的资源直接启动到本地 **localhost:9000/sub_local/** 下。

```
{
	output:{
		fileName: "sub.bundle.js"
		,libraryTarget:"umd"
		,path:path.resolve(__dirname,"./dist/")
	}
	,devServer:{
		contentBase:path.resolve(__dirname,"./dist/")
		,port:"9000"
		,pubicPath:"sub_local"
	}
}

```

nginx中将部署地址重写到9000端口

```
# 将/Sub/下的js访问url全部重写
rewrite /Sub(.*)/(?P<filename>(.*))\.js /sub_local/$filename.js

# 重写后的url再代理到 webpack-dev-server启动的端口上
location ~ /sub_local/(.*)\.js {
	poxy_pass http://localhost:9000
}
```

基本上 Main 和 Sub 的配置是一样的。rewrite只能rewrite路由，不能涉及到域名，而代理可以更换域名，但是不能更改路径，两者结合才能获得如此高的自由度——真正的彻底改变整个url。

**？？？**这个方案有个问题，hot-replace会有问题，我感觉是我用法的问题，需要再作研究。

## 其他方案

仔细一想，感觉上面的方案还是显得麻烦。能否只启动一个webpack-dev-server的服务，打包多个库并从内存中访问？

如果只有一个服务，势必只有一个 webpack_config，而且这个webpack_config位于两个项目之外，指定多个entry，使用一个devServer。我使用demo验证了没有问题。在实际使用中可能会有麻烦，因为contentBase只能使用一个,这样所有资源的都要在最外层的webpack_config里页面重写打包方案，另外需要避免重名。















