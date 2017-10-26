---
layout: post
title:  "使用webpack+babel搭建react开发项目"
date:   2017-05-02
author: 梁龙飞
tags: 编程
---

> 写了两月 vue, 忽然之间 react 忘记了一些东西。人们总是说：“学习是与遗忘做斗争的过程”，但是我并不这么认为，我忘记的应该只是一些不重要的东西——就像现在我已经完全了解 webpack babel,react 一整套的原理，但是具体的 api，具体搭建项目时的配置，我还是得去查下。也许当我们储备了足够的知识，有自己的技术观时，我们的学习其实是印证修正自己技术观的过程，这正是快速学习的前提。

## 脚手架

```shell
// react
npm install react --save-dev
npm install react-dom  --save-dev

// babel
npm install babel-loader --save-dev
npm install babel-core --save-dev
npm install babel-preset-es2015 --save-dev
npm install babel-preset-react --save-dev

// webpack
npm install webpack --save-dev
npm install webpack-dev-server --save-dev

// sass
npm install node-sass --save-dev
npm install sass-loader --save-dev
```

## 配置

### wepack

```javascript
var path = require('path');

module.exports = {
	entry:"./src/main.js"
	,output:{
		path: path.resolve(__dirname, "./release")
		,filename:"[name].bundle.js"
		,libraryTarget:"umd"
	}
	,module: {
        //加载器配置
        rules: [
            { test: /\.css$/, loader: 'style-loader!css-loader' },
			{
				test: /\.jsx?$/,
				exclude: /node_modules/,
				loader: 'babel-loader'
			}
           
        ]
    }
}
```

webpack2的文档有了长足的进步，这对我很有启发，以前我设计一个项目的时候总是先做文档，这对于一个程序员不是一个很好的方式，因为文档写的过于详细很快就消耗完了精力。这是webpack2配置说明，包含所有配置节点：

[https://webpack.js.org/configuration/](https://webpack.js.org/configuration/)


### babel

babel的选项有两种配置方法

一种是查询字符串：

```javascript
loaders: [
	{
		test: /\.jsx?$/,
		exclude: /node_modules/,
		loader: 'babel?preset[]=es2015'
	}
   
]
```

一种是设置query 节点：
```javascript
loaders: [
	{
		test: /\.jsx?$/,
		exclude: /node_modules/,
		loader: 'babel-loader',
		options: {
			presets: ['es2015', 'react']
		}
	}
   
]
```

还有就是将 options 写在 .babelrc 中：

```javascript
{
	presets: ['es2015', 'react']
}

```

**babel-preset-env** 插件， 转化出支持特定环境的代码。

例如：打包出的代码仅支持chrome52
```javascript
{
  "presets": [
    ["env", {
      "targets": {
        "chrome": 52
      }
    }]
  ]
}
```

## 项目

配置完了，把文件路径什么的在webpack中同步好，直接执行 webpack 命令就ok了。 对于react 说个题外话。

很多新同学接手项目的时候，入口基本都被本地化的框架接手了，所以做的都是路由以下的事情，react的入口一般是这样的：

```javascript
import React from 'react'
import { render } from 'react-dom'

import { Router, Route, IndexRoute, Link, hashHistory } from 'react-router'

const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        {/* change the <a>s to <Link>s */}
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>

        {/*
          next we replace `<Child>` with `this.props.children`
          the router will figure out the children for us
        */}
        {this.props.children}
      </div>
    )
  }
})

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>
      <IndexRoute component={Home} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox} />
    </Route>
  </Router>
), document.body)

```

react在某个版本自己分离成了react + react-dom，我之前一直在关注react，中间突然分开着实坑了我一把 —— 这也让我设计规划的时候更加大胆，更改api和模块结构也在所不惜，因为我发现很多开源库都这样 —— 为了更好，再所不惜。

## redux 与高阶组件


## 备注（以下为碎片，尚未总结）

### dom 操作
所有的**MVx**前端框架都不提倡dom操作，但是有些时候避免不了，比如手动触发某些元素的原生事件。这个时候就需要对原生dom使用ref引用到React环境,看官方的例子：

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focus = this.focus.bind(this);
  }

  focus() {
    // Explicitly focus the text input using the raw DOM API
    this.textInput.focus();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={(input) => { this.textInput = input; }} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focus}
        />
      </div>
    );
  }
}
```

### 频繁使用setState

## 参考

[https://github.com/ReactTraining/react-router/blob/v3/docs/Introduction.md](https://github.com/ReactTraining/react-router/blob/v3/docs/Introduction.md)

































