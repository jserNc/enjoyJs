---
title: JSONP 和图像灯塔
date: 2016-12-28 17:13:08
tags: js
---

当我们需要和服务器交换数据并更新网页部分内容的时候，我们很容易想到用 ajax。即通过原生的 XMLHttpRequest 对象（ IE5 和 IE6 使用 ActiveXObject）发出 HTTP 请求，得到服务器返回的数据后，再进行处理。不过，遗憾的是：受浏览器同源策略的限制，ajax 只能向同源（协议、域名、端口都相同）网址发出 http 请求，跨源请求会被阻止、报错。

<!-- more -->

然而，我们跨域请求资源的需求是很强烈的，JSONP 就是其中一种跨域办法。

** JSONP **

** 原理：** script 标签没有同源限制，其 src 属性可以指向任何地址。我们通过在页面中动态增加一个 script 标签，标签的 src 属性指向的是另外一个域的资源文件地址。

具体应用时，我们先在本地定义一个方法 callback，我们想要执行的任务代码写在该方法里；和服务器端约定好，在待请求的资源文件里调用该 callback 方法；当资源文件请求成功后，就会执行 callback 方法，完成我们的任务。

下面是一个 jsonp 获取资源文件的方法实例：
```
getScript = function (url, config) {
    var s = document.createElement('script'),
        head = document.getElementsByTagName('head')[0],
        config= config || {};

    if (config.charset) {
        s.charset = config.charset;
    };
    s.onreadystatechange = s.onload = function() {
        if (!this.readyState || 
            this.readyState == 'loaded' || 
            this.readyState == 'complete') {
            if (config.callback) {
              config.callback()
            };
            s.onreadystatechange = s.onload = null;
            s.parentNode.removeChild(s);
        }
    };
    s.onerror = function (){
        if (config.errCb) {config.errCb()};
    };
    s.src = url;
    head.insertBefore(s, head.firstChild);
};
```

这个方法第一个参数 url 是待请求的资源地址，第二个参数 config 是配置对象，可以定义接口请求成功后的回调函数 config.callback 或接口出错的回调函数 config.errCb 以及脚本编码 config.charset 等等。

例如：地址 //www.ad.com/index.php 有我们想要的广告数据，我们想拿到这个页面里的广告数据，然后在我们的页面里把数据拼接渲染出来。等到数据渲染完成后控制台打印“广告成功渲染”，若是广告数据请求失败，控制台打印“数据请求失败”。

** 具体操作步骤如下：**

** 1. 我们自己页面里定义数据拼接渲染方法（即上文提到的回调方法）**
```
function renderAd(data){
    //code
}
```

** 2. //www.ad.com/index.php 页面里调用 renderAd 方法 **
```
adData = ['ad1','ad2','ad3'];
renderAd(adData);
```

** 3. 我们自己页面里调用 getScript 请求资源 **
```
getScript('//www.ad.com/index.php', {
    callback: function(){
        console.log('广告成功渲染');
    },
    errCb: function() {
        console.log('数据请求失败');
    }
});
```

若是接口请求成功，先执行 renderAd(adData) 方法渲染数据，然后执行请求成功回调函数 callback；若是接口请求失败，执行请求失败回调方法 errCb。

JSONP 的局限性在于其只支持 GET 请求，其优点在于其兼容性好，支持老式浏览器。

** 图像灯塔 **

使用远程脚本的最简单场景是只向服务器发送数据，而并不需要得到服务器的回应。这种情况下，可以创建一个新图像，将其 src 属性指向服务器脚本地址，例如：
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
这种模式称为图像灯塔（image beacon）。这在向服务器发送日志数据时很有用的，比如收集网页访问者的统计信息。

最后再强调一点，** 只创建 script 节点和设置其 src 属性并不会发起加载 js 的请求（ie6~9除外），必须等到 scriptNode 插入到 dom 后才会发起请求。这和图片 img 的预加载是不同的，图片设置 src 属性后会立即发起请求，所以以上 beacon 方法中，img 的 onload、onerror、onabort 等回调函数都需要定义在 src 属性之前！ ** 

参考：
[1] 《JavaScript模式》Stoyan Stefanov 著
[2] http://javascript.ruanyifeng.com/bom/ajax.html



