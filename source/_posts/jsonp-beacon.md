---
title: JSONP 和图像灯塔
date: 2016-12-28 17:13:08
tags: js
---

当我们需要和服务器交换数据并更新网页部分内容的时候，我们很容易想到用ajax。即通过原生的XMLHttpRequest对象（IE5 和 IE6 使用 ActiveXObject）发出HTTP请求，得到服务器返回的数据后，再进行处理。不过，遗憾的是：受浏览器同源策略的限制，ajax只能向同源（协议、域名、端口都相同）网址发出http请求，跨源请求会被阻止、报错。

<!-- more -->

然而，我们跨域请求资源的需求是很强烈的，JSONP就是其中一种跨域办法。

** JSONP **

** 原理：** script标签没有同源限制，其src属性可以指向任何地址。我们通过在页面中动态增加一个script标签，标签的src属性指向的是另外一个域的资源文件地址。

具体应用时，我们先在本地定义一个方法callback，我们想要执行的任务代码写在该方法里；和服务器端约定好，在待请求的资源文件里调用该callback方法；当资源文件请求成功后，就会执行callback方法，完成我们的任务。

下面是一个jsonp获取资源文件的方法实例：
```
getScript = function (url, config) {
    var s = doc.createElement('script'),
        head = doc.getElementsByTagName('head')[0],
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
    s.src =url;
    head.insertBefore(s, head.firstChild);
};
```

这个方法第一个参数url是待请求的资源地址，第二个参数config是配置对象，可以定义接口请求成功后的回调函数config.callback或接口出错的回调函数config.errCb以及脚本编码config.charset等等。

例如：地址//www.ad.com/index.php有我们想要的广告数据，我们想拿到这个页面里的广告数据，然后在我们的页面里把数据拼接渲染出来，等到数据渲染完成后控制台打印“广告成功渲染”。若是广告数据请求失败，控制台打印“数据请求失败”。

** 具体操作步骤如下：**

** 1. 我们自己页面里定义数据拼接渲染方法（即上文提到的回调方法）**
```
function renderAd(data){
    //code
}
```

** 2.//www.ad.com/index.php页面里调用renderAd方法 **
```
adData = ['ad1','ad2','ad3'];
renderAd(adData);
```

** 3.我们自己页面里调用getScript请求资源 **
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

若是接口请求成功，先执行renderAd(adData)方法渲染数据，然后执行请求成功回调函数callback；若是接口请求失败，执行请求失败回调方法errCb。

JSONP的局限性在于其只支持GET请求，其优点在于其兼容性好，支持老式浏览器。

** 图像灯塔 **

使用远程脚本的最简单场景是只向服务器发送数据，而并不需要得到服务器的回应。这种情况下，可以创建一个新图像，将其src属性指向服务器脚本地址，例如：
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

最后再强调一点，** 只创建script节点和设置其src属性并不会发起加载js的请求（ie6~9除外），必须等到scriptNode插入到dom后才会发起请求。这和图片img的预加载是不同的，图片设置src属性后会立即发起请求，所以以上beacon方法中，img的onload、onerror、onabort等回调函数都需要定义在src属性之前！ ** 

参考：
[1] 《JavaScript模式》Stoyan Stefanov 著
[2] http://javascript.ruanyifeng.com/bom/ajax.html



