---
title: 前端模块化和打包工具 webpack
date: 2016-11-21 11:57:27
tags: tools 
---

Webpack 是一个模块打包器。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。这里说的模块不仅仅只是 JavaScript 模块化，还有 CSS、图片、字体等也需要模块化。

<!-- more -->

![作用域链](/css/images/webpack/webpack.png)

互相依赖的模块，经过webpack的处理，生成相应的静态资源。webpack在这里起到模块定义、依赖和导出的作用。

### 为什么要模块化加载？

目前，越来越多的网站已经从网页模式进化到了webapp模式，页面需要的JavaScript、css、图片等等各种资源也越来越多，特别是大量的js代码会给前端开发流程和资源组织带来巨大挑战。

前端资源是通过增量加载的方式在浏览器端运行，如何组织好前端代码和资源，使它们能够快速、优雅地加载和更新，是一个模块化系统需要解决的问题。

前端模块需要增量加载到用户浏览器中执行。关于这个过程，我们设想两种极端的方式：一种是每个模块都单独请求；另一种是把所有文件打包成一个文件，然后请求一次。很明显，这两种方式简单粗暴的方式都不够好，第一种方式请求数太多，网页响应速度太慢；第二种会导致流量浪费、初始化过程慢。

### 分块传输，按需进行懒加载，在实际用到某些模块的时候再增量更新，才是较为合理的模块加载方案。

要实现模块的按需加载，就需要一个对整个代码库中的模块进行静态分析、编译打包的过程。这就是webpack要做的工作。

下面，就以一个简单实例介绍webpack的使用流程。

### 1.安装webpack

首先，得具备node环境。安装node会顺带安装包管理器npm，我们用npm全局安装webpack。

```
npm install webpack -g
```

然后，我们新建一个项目目录myWebpack。项目本地安装webpack。

```
# 进入项目目录
# 确定已经有 package.json，没有就通过 npm init 创建
# 安装 webpack 依赖
npm install webpack --save-dev
```

### 2.小试牛刀

首先新建index.html页面：

```
<html>
<head>
  <meta charset="utf-8">
</head>
<body>
  <script src="bundle.js"></script>
</body>
</html>
```

然后，新建entry.js文件：

```
document.write('It works.');
```

最后，使用webpack编译entry.js并打包到bundle.js：

```
webpack entry.js bundle.js
```

用浏览器直接打开index.html(不需要web服务器)，我们会看到页面显示：It works.

这是没有文件依赖的情况。下面我们新增一个模块module.js：

```
module.exports = 'It works from module.js.'
```

同时，修改entry.js：

```
document.write('It works.')
document.write(require('./module.js'))
```

重新执行：

```
webpack entry.js bundle.js
```

刷新index.html,页面显示为：It works.It works from module.js.


### 3.loader

以上我们打包的模块都是js文件，实际上，webpack本身也只能处理JavaScript模块。如果要处理css、图片等非js文件，就需要使用loader进行转换。在这里，loader充当一个模块和资源之间的转换器。我们用npm来安装loader：

```
npm install css-loader style-loader
```

我们需要两种loader来转换css文件。css-loader用来读取css文件，style-loader用来将style标签插入html页面。

新建一个style.css文件：

```
body { background: yellow; }
```

修改entry.js：

```
require("!style!css!./style.css") // 载入 style.css
document.write('It works.')
document.write(require('./module.js'))
```
重新编译，打包资源：

```
webpack entry.js bundle.js
```

刷新index.html页面，我们看到，背景已经变成样式里写的黄色了。

如果觉得entry.js中加载style.css的方式比较冗长，我们可以将entry.js中的style.css加载方式改为：

```
require("./style.css")
```

相应地，编译打包方式修改为：

```
webpack entry.js bundle.js --module-bind 'css=style!css'
```

### 4.配置文件

如果我们向通过一个简单的

```
webpack
```

命令来实现编译，打包过程。我们只需要事先准备好一个配置文件webpack.config.js。

首先，我们更改package.json，添加webpack所需的依赖：

```
{
    "name": "webpack-example",
    "version": "1.0.0",
    "description": "A simple webpack example.",
    "main": "bundle.js",
    "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    },
    "keywords": [
    "webpack"
    ],
    "author": "yourself",
    "license": "MIT",
    "devDependencies": {
    "css-loader": "^0.21.0",
    "style-loader": "^0.13.0",
    "webpack": "^1.12.2"
    }
}
```

然后执行：

```
npm install
```

安装依赖。新建webpack.config.js配置文件：

```
module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'}
    ]
  }
}
```
不同的loader用感叹号（!）分隔，上面的style和css分别指Style-loader和CSS-loader。

同时简化 entry.js 中的 style.css 加载方式：

```
require('./style.css')
```

最后运行webpack编译打包，刷新页面，就可以看到之前的效果了。

```
webpack
```


参考:
[1] http://webpackdoc.com/
