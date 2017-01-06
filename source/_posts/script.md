---
title: 关于script脚本加载方式（译）
date: 2016-10-28 14:10:12
tags: problem
---

原文： http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html

我们知道，js同步加载会阻塞浏览器对dom解析，体验比较差。如果可以，我们会尽可能地采取异步方式调取js脚本。但由于某些js代码中有可能输出document内容、修改dom、重定向等行为，所以默认同步执行脚本才是安全的。

<!-- more -->

下面我们对比一下&lt;script&gt;、&lt;script async&gt;、&lt;script defer&gt;等三种脚本加载方式：

图例：
![图例](/css/images/script/legend.svg)

&lt;script&gt;

对这种不带特殊属性的&lt;script&gt;标签。当html页面解析到该标签时会停止，然后浏览器立即发出请求去下载该脚本（如果该脚本是外部脚本的话），等到脚本下载完成，马上解析执行该脚本。脚本执行完毕再接着解析html文档。

<img src="/css/images/script/script.svg" width="700" alt="script"/>

&lt;script async&gt;

有async属性的script脚本下载过程并不会阻塞页面继续解析，但是，等到脚本下载完成后，会暂停页面解析并立即执行该脚本。脚本执行完毕再接着解析html文档。由于脚本下载完成时刻不可控，所以，无法确定该脚本和其他代码执行顺序。

<img src="/css/images/script/script-async.svg" width="700" alt="script-async"/>

&lt;script defer&gt;

有defer属性的script脚本下载过程也不会阻塞页面继续解析，它会等到这个页面解析完成后再执行，而且多个带defer的脚本会按其在页面中出现的顺序依次执行。

<img src="/css/images/script/script-defer.svg" width="700" alt="script-defer"/>

那么，我们该如何选择呢？

一般说来，首选async,其次是defer,最后是不带属性。规则如下：

* 如果是模块化的脚本，并且不依赖于其他脚本，用async属性。

* 如果当前脚本依赖于其他脚本或者被其他脚本依赖，用defer属性。

* 如果当前脚本比较小并且被其他脚本依赖，则不需带以上两种属性。

注意：IE9及其以下版本对defer属性并不是支持得很好，并不能保证defer脚本的执行顺序。所以，IE9及其以下版本并不推荐使用defer属性。比如，如果非要保证脚本执行顺序，就不带任何属性，使其同步执行。

总结：

* async和defer在网络读取（下载）这个过程是一样的，都不会阻塞页面解析。
* defer脚本会在页面渲染完成后按顺序执行，而async不会。

另外，关于script标签属性，有以下标准：

* **如果src属性没有设置，则执行script标签内的脚本内容；如果src有指定的url，则忽略script脚本里的内容，只是去请求指定的url内容。**

* **若script标签已存在于dom文档中，之后再去动态的修改script标签的src、type、charset、async以及defer等属性是没有效果的。**

例如：不要使用类似下面这种方式来加载js(请求根本不会发出去)

```
div.innerHTML = '<script src="main.js"></script>';
```

任何动态添加script节点（如appendChild(scriptNode)）的方式引入的js文件都是异步执行的。需要引起我们注意的是：**只创建script节点和设置其src属性并不会发起加载js的请求（ie6~9除外），必须等到scriptNode插入到dom后才会发起请求。这和图片img的预加载是不同的**

我们看一个图片预加载方法：

```
beacon = function (src,cb) {
    var img = new Image();

    img.onload = img.onerror = img.onabort = function(){
        img.onload = img.onerror = img.onabort = null;
        img = null;
        cb && cb();
    };

    img.src = src;
};
```

这个方法预加载图片，利用的原理就是：只要图片对象被设置了src属性，就会立即发出http请求。也正是因为这个，所以以上img的onload、onerror以及onabort等属性必须在src属性之前定义！



参考：
[1] http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html
[2] http://zhangyaochun.iteye.com/blog/1522144
