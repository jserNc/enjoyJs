---
title: 跨域解决方案
date: 2017-05-24 09:17:21
tags: method
---

受浏览器同源策略的限制，ajax 只能向同源（协议、域名、端口都相同）网址发出 http 请求，跨源请求会被阻止、报错。下面介绍 5 种跨域解决方案，具体方案还需要根据实际应用场景来选择。

<!-- more -->

### ① jsonp

该方案的局限性在于其只支持 GET 请求，其优点是兼容性好，支持所有浏览器。一般场景下首推这种方案，其详细介绍见 [JSONP 和图像灯塔](http://nanchao.win/2016/12/28/jsonp-beacon/) 。

### ② 修改 window.name 跨域

window 对象有个 name 属性，用于设置当前浏览器窗口的名字。只要是在本窗口打开过的网页都能读写这个窗口的 window.name 属性，而不管这些网页是否属于同一个网站。

也就是说：即在一个窗口（window）的生命周期内，在该窗口载入的所有的页面都是共享同一个 window.name，并且这些页面对 window.name 都有读写的权限。window.name 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而被重置。

该属性只能保持字符串，并且当前窗口关闭以后这个值就会消失，但是与 iframe 结合使用还是挺有用的。

例如，A 域下有一个页面 a.htm，想要获取 B 域下的 b.htm 中的数据。

子页面 B.com/b.htm 代码：

```
<script>
 window.name = 'a.htm想要获取的数据';
</script>
```

我们可以在 a.htm 页面中设置一个隐藏的 iframe，iframe 地址指向 b.htm，等到 b.htm 加载完毕，再将 iframe 地址替换成 A 域下的 a1.htm，然后就可以通过该 iframe 的 contentWindow.name 属性获取到想要的数据。

父页面 A.com/a.htm 代码：

```
<iframe id="ifr" src="http://www.B.com/b.htm"></iframe>
<script>
    var flag = 0,
        frame = document.getElementById("ifr");

    frame.onload = function(){
        if (flag === 1){
            console.log(frame.contentWindow.name);
        } else if (flag === 0){
            flag = 1;
            frame.contentWindow.location = 'A.com/a1.htm'
        }
    }
</script>
```

### ③ 修改 document.domain 跨子域

该方法只适用于不同子域的框架间的交互

不同的框架之间（父子或同辈），是能够获取到彼此的 window 对象的，但是你却不能使用获取到的 window 对象的属性和方法（html5 中的 postMessage 方法是一个例外，还有些浏览器比如 ie6 也可以使用 top、parent 等少数几个属性），总之，你可以当做是只能获取到一个几乎无用的 window 对象。

如有一个页面，它的地址是 www.example.com/a.html  ， 在这个页面里面有一个 iframe，它的 src 是 example.com/b.html, 很显然，这个页面与它里面的 iframe 框架是不同域的

我们将 a.html 和 b.html 都设置成：

```
document.domain = 'example.com'
```

这样我们就可以通过 js 来访问 iframe 中各种属性和对象了

### ④ 使用 h5 中的 window.postMessage 方法

```
window.postMessage(message,targetOrigin) 
```

该方法可以向其他的 window 对象发送消息，无论这个对象属于同源或是不同源。

向其他 window 对象发送消息，是指一个页面有几个框架的情况下，每一个框架都有一个 window 对象，框架之间可以互发消息。这里发送消息不受同源政策限制，即不同域的框架之间也可以获取到对方的 window 对象，然后用 window.postMessage 方法发送消息。

需要接收消息的 window 对象，可是通过监听自身的 message 事件来获取传过来的消息，消息内容储存在该事件对象的 data 属性中。举例如下：

www.a.com/a.htm 代码:

```
<iframe id="ifr" src="http://www.b.com/b.htm"></iframe>
<script>
    document.getElementById("ifr").onload = function(){
        var win = this.contentWindow;
        win.postMessage('来自a页面的消息')
    }
</script>

```

www.b.com/b.htm 代码:

```
<script>
    window.onmessage = function(e){
        e = e || event;
        console.log(e.data)
    }
</script>
```

使用 postMessage 来跨域传送数据还是比较直观和方便的，缺点是 IE6、IE7 等浏览器不支持，所以用不用还得根据实际需求来决定。

### ⑤ CORS 跨源资源分享（Cross-Origin Resource Sharing）

它是 w3c 标准，是跨源 AJAX 请求的根本解决方法。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。实现 CORS 通信的关键是服务器,只要服务器实现了 CORS 接口，就可以跨源通信。

相比 jsonp 只能发 GET 请求，CORS 允许任何类型的请求。对开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。





参考：
[1] http://javascript.ruanyifeng.com/bom/cors.html