---
layout: post
title:  "服务器配置静态资源的缓存策略"
date:   2016-11-04
author: 梁龙飞
tags: 编程
---

> 这篇源自上篇博客，在调研chrome的cache comporessed的时候发现自己对服务器设置各种缓存策略不是很了解。你可能已经注意到这个标题有些奇怪，我本来的名字是【浏览器缓存策略】，但是容易与cookie、session，甚至是localstorage之流混为一谈，所以起了个有些复杂的名字。

## 环境、前提
> 以下测试阐述用例使用的是nginx 1.8.1。

## Expires
资源的过期时间，这是个时间点。它的运行机制如下：

1. 服务器设置资源A的过期时间为 **Mon, 07 Nov 2016 09:45:28 GMT**
2. 客户端第一次访问A时，下载资源200。再次访问资源A时，如果客户端时间在Expires之前，则直接从缓存中读取A，状态为 200 from cache.否则再次请求服务器。

Expires定义于http1.0，虽然浏览器依然处理此策略，但是http1.1定义的cache-control更犀利、全面。需要注意的是如果使用nginx测试，其中expires字段并不是Expires，见关联知识第二点。

## Cache-Control相关
Cache-Control的常用取值（其实是我理解的，其他的不理解不敢瞎写）：

1. max-age=t,在t秒内不会再次请求此资源，当此项与Expires同时存在时，此项优先级高于Exprires。
2. no-cache,不直接使用本地缓存，每次会去服务器验证有效性，所以可能会触发304 cache.
3. no-store,完全不使用缓存。
4. public,缓存所有资源。

**Cache-Control与Expires同时设置时，Cache-Control的优先级高于Expires。**

### Last-Modified/If-Modified-Since

如果服务器开启此功能，其运行机制如下：

1. 客户端第一次请求资源A时，返回资源A的 response-header 中有 Last-Modified字段
2. 第二次请求A资源，并且此资源过期（Expires的设置，或者Cache-Control设置），客户端发送一个带有If-Modified-Since字段的请求去服务器，服务器对比A资源的最后修改时间T与客户端发送过来的If-Modified-Since。如果T>If-Modified-Since,返回状态码200，并完全返回资源A的数据。如果 T = If-Modified-Since，返回304，客户端从缓存中读取A的数据。如果T < If-Modified-Since,那么这是个非法请求。

### Etag/If-None-Match

Etag与Last-Modified的机制一样，不同的是Etag有生成规则，用以区分同一文件在时间维度上的不同。Etag提出主要是解决以下Last-Modified不能解决的问题：

1. Last-Modified只能精确到秒级，如果1秒内多次修改，可能造成缓存到客户端的不是最新的资源。
2. 某些文件定时更新，客户端关心的资源并没有变化，但Last-Modified改变了，将无法使用缓存。
3. 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形。

注意，当Etag和Last-Modified一起设置时，Etag的优先级高于Last-Modified，但并不是覆盖。


## 缓存应用流程图

### 客户端第一次请求资源
<img src="/image/first_request.png" >

### 客户端第二次请求资源
<img src="/image/second_request.png" >

哈哈~，这两张图是盗的，来自于参考项的文章中。



## 关联知识

- GMT时间，格林尼治时间，北京时间是东八区，这个八就是相对格林尼治，所以北京时间减8个小时就是GMT。
- ngx_http_headers_module，此模块总是在response header上追加 Expires 和 Cache-Control字段，同时expires不设置为 off时，Expires和Cache-Control会因为expires的设置而被修改。参考:[Module ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html#expires)

## 参考
[http://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html](http://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html)
[https://www.mnot.net/cache_docs/](https://www.mnot.net/cache_docs/)

