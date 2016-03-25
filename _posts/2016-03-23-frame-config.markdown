---
layout: post
title:  "前端框架与配置文件"
date:   2016-03-23
author: "UI框架开发团队"
categories: frame
---

### 加载
1. 浏览器中输入基于beisencloud运行态的产品url后，生成首页的流程如下（仅关注配置文件）
![img][1]

2. 首页根据配置文件加载前端的资源，platform-components的配置文件如下：

``` xml
<File Name="common-header">
  <Content><![CDATA[
	<link href="http://stnew03.beisen.com/ux/platform-components/release/app/styles/css/all-1603211353.min.css" rel="stylesheet" type="text/css" />
	 <script>
	 var BSGlobal = { root: "/" , start: new Date , prefillingData: {} , container: "#bs_layout_container" }; 
	 </script>
]]></Content>
</File>
<File Name="common-footer">
  <Content><![CDATA[
	<div id="bs_layout_container" class="top_container"></div> 
	    <script>BSGlobal.staticPath = 'http://stnew03.beisen.com/ux/platform-components/release/app';</script>
	<script src="http://stnew03.beisen.com/ux/platform-components/release/app/scripts/main-1603211207.min.js">
	 </script> 
	 <script type="text/javascript">
		 requirejs.config({
			baseUrl: BSGlobal.staticPath+'/scripts'
		}); 
	 </script>
]]></Content>
</File>
```

上面的xml文件有两个节点：<br/>
**common-header:**  这个节点的数据会被完整加载到首页的head元素中。<br/>
**common-footer:** 这个节点的数据会加载到body元素中。<br/>
 扩展资源的配置文件如下（大体与platform-components相同）：

```xml
<File Name="common-header">
    <Content><![CDATA[
<link href="http://stnew03.beisen.com/ux/ecm/release/app/styles/css/all-1603181807.min.css" rel="stylesheet" type="text/css" />
]]></Content>
    </File>
    <File Name="common-footer">
      <Content><![CDATA[
	<script>
        BSGlobal.entryPageId = "home";
		BSGlobal.appjssrc = "http://stnew03.beisen.com/ux/ecm/release/app/scripts/main-1603181807.min.js";
        BSGlobal.MenuConfig = {
            "RootAsModule":true,  
            "ModultRoutes":["ecm","vaas","http://hcm.tms.beisen.com/emp/Home/Index#home","http://hcm.tms.beisen.com/org/Home/Index#organization"]
        };
	</script>
]]></Content>
</File>
```

注意上面的**common-fooer**节点中的 BSGlobal.appjssrc 属性，它指定了扩展js的位置。

>打开任一基于platform-components开发的项目，查看其网页源代码可以看到配置文件与首页的关系。

3.介绍完上面两个配置文件后，我们来看扩展资源是如何加载到platfrom-components中：

{% highlight javascript linenos %}
/**
 * requireJs entry file
 */
require(['config'], function(mainConfigFile){
	require(['corebone'],function(Corebone){
		//如果扩展资源存在，则加载它
		if (BSGlobal.appjssrc) {
			require([BSGlobal.appjssrc],function(extension){
			    //挂载在corebone下的extension节点下
				Corebone.Extension = extension;
				Corebone.app.start(_.extend({},window.BSGlobal || {}));
			});
		}else{
			Corebone.app.start(_.extend({},window.BSGlobal || {}));
		};
		
	});
});
{% endhighlight %}
当platform-components得知某一组件是扩展的，则会从Corebone.Extension中加载扩展资源。

[1]:/img/flow1.png