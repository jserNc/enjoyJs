---
title: 关于document.write引入第三方脚本问题
date: 2016-10-13 16:10:38
tags:
---

一般情况下，我们会选择异步方式调取js文件，但是某些场景又不得不使用同步方式来调用js。除了直接用script标签插入同步脚本，document.write也是一种选择。然而，chrome认为其会拖慢页面加载速度，牺牲用户体验。所以，新版chrome决定对用document.write注入第三方脚本的方式采取一些干预措施。

最近，可能也有人像我一样，看某些网页的时候，chrome控制台出现了下面的警告：

<!-- more -->

``` 
A Parser-blocking, cross-origin script, https://paul.kinlan.me/ad-inject.js, is invoked via document.write. This may be blocked by the browser if the device has poor network connectivity.
```
 
[官方](https://developers.google.com/web/updates/2016/08/removing-document-write)指出chrome即将干预通过document.write引入的脚本了，[我们应该避免使用这种写法](http://blog.dareboost.com/en/2016/09/avoid-using-document-write-scripts-injection/)：

``` 
document.write('<script src="https://paul.kinlan.me/ad-inject.js"></script>');
```

使用document.write会有什么问题？


在浏览器渲染页面之前，它必须先解析html标签来构建dom树。每当解析器遇到了一个脚本，它就会暂停解析html，下载并解析执行这个脚本，脚本执行完再继续解析dom，如果，这个脚本又动态注入另一个或多个脚本，那解析器还得等待更长时间才能继续往下走，这会明显拖慢网页渲染速度。

chrome测试发现，使用2g等连接缓慢的用户，通过document.write动态注入的脚本可能会使得页面显示延迟数十秒，甚至加载失败。另外，在2g网络上，使用document.write插入第三方脚本的网页通常比其他网页加载慢两倍。

当满足以下所有条件时，chrome就不会执行通过document.write注入的script脚本：

1.用户网络缓慢，特别是2g网这种（不过，以后可能会包括比较缓慢的3g，wifi网络）；

2.document.write用在顶级文档中（之所以不包括iframe页面，是因其不会阻止主页面的渲染）；

3.document.write引入的同步脚本（具有async或defer属性的异步脚本不会阻塞页面渲染）；

4.该脚本不在浏览器缓存中（缓存了的脚本并不会产生网络延迟，所以仍会执行）；

5.页面不是重新加载（如果用户主动重新加载该页面，chrome就不会干预，并正常执行该网页）。

自chrome 53开始，如果document.write请求符合以上的条件2-5,chrome就会发出上面的警告。自10月中旬发布的chrome 54开始2g用户的注入脚本将被干预，以后，任何连接缓慢的用户（即缓慢的3g或WiFi）都可能被干预。
