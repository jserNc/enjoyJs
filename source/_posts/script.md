---
title: 关于 script 脚本加载方式（译）
date: 2016-10-28 14:10:12
tags: problem
---

原文： http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html

我们知道，js 同步脚本加载会阻塞浏览器对 dom 的解析，体验比较差。如果可以，我们会尽可能地采取异步方式调取 js 脚本。鉴于 js 代码中有可能包含输出 document 内容、修改 dom、重定向等行为，所以默认同步执行脚本才是安全的。

<!-- more -->

下面我们对比一下 **&lt;script&gt;**、**&lt;script async&gt;**、**&lt;script defer&gt;** 等三种脚本加载方式：

图例：
![图例](/css/images/script/legend.svg)

**&lt;script&gt;**

不带任何特殊属性的 &lt;script&gt; 标签。当 html 页面解析到该标签时会停止，然后浏览器立即发出请求去下载该脚本（如果该脚本是外部脚本的话），等到脚本下载完成，马上解析执行该脚本。脚本执行完毕再接着解析 html 文档。

<img src="/css/images/script/script.svg" width="700" alt="script"/>

**&lt;script async&gt;**

有 async 属性的 script 脚本下载过程并不会阻塞页面继续解析，但是，等到脚本下载完成后，会暂停页面解析并立即执行该脚本。脚本执行完毕再接着解析 html 文档。由于脚本下载完成时刻不可控，所以，无法确定该脚本和其他代码执行顺序。

<img src="/css/images/script/script-async.svg" width="700" alt="script-async"/>

**&lt;script defer&gt;**

有 defer 属性的 script 脚本下载过程也不会阻塞页面继续解析，它会等到这个页面解析完成后再执行，而且多个带 defer 的脚本会按其在页面中出现的顺序依次执行。

<img src="/css/images/script/script-defer.svg" width="700" alt="script-defer"/>

那么，我们该如何选择呢？

一般说来，首选 async,其次是 defer,最后是不带属性。具体规则如下：

* 如果是模块化的脚本，并且不依赖于其他脚本，用 async 属性。

* 如果当前脚本依赖于其他脚本或者被其他脚本依赖，用 defer 属性。

* 如果当前脚本比较小并且被其他脚本依赖，则不需带以上两种属性。

注意：IE9 及其以下版本对 defer 属性并不是支持得很好，并不能保证 defer 脚本的执行顺序。所以，IE9 及其以下版本并不推荐使用 defer 属性。比如，如果非要保证脚本执行顺序，就不带任何属性，使其同步执行。

总结：

* async 和 defer 在网络读取（下载）这个过程是一样的，都不会阻塞页面解析。

* defer 脚本会在页面渲染完成后按顺序执行，而 async 不会。

另外，关于 script 标签属性，有以下标准：

* **如果 src 属性没有设置，则执行 script 标签内的脚本内容；如果 src 有指定的 url，则忽略 script 脚本里的内容，只是去请求指定的 url 内容。**

* **若 script 标签已存在于 dom 文档中，之后再去动态的修改 script 标签的 src、type、charset、async 以及 defer 等属性是没有效果的。**

例如：不要使用类似下面这种方式来加载 js (请求根本不会发出去)

```
div.innerHTML = '<script src="main.js"></script>';
```

任何动态添加 script 节点（如 appendChild(scriptNode) ）的方式引入的 js 文件都是异步执行的。需要引起我们注意的是：**只创建 script 节点和设置其 src 属性并不会发起加载 js 的请求（ie6~9除外），必须等到 scriptNode 插入到 dom 后才会发起请求。这和图片 img 的预加载是不同的**

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

这个方法预加载图片，利用的原理就是：只要图片对象被设置了 src 属性，就会立即发出 http 请求。也正是因为这个，所以以上 img 的 onload、onerror 以及 onabort 等属性必须在 src 属性之前定义！



参考：
[1] http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html
[2] http://zhangyaochun.iteye.com/blog/1522144
