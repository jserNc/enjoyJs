---
title: 关于 document.write 引入第三方脚本
date: 2016-10-13 16:10:38
tags: problem
---

一般情况下，我们会选择异步方式调取 js 文件，但是又有一些不得不使用同步方式来调用 js 的场景。对于这种场景，除了可以直接在 html 中静态写入 script 同步脚本，document.write 也是一种选择。然而，chrome 认为后者会拖慢页面加载速度，牺牲用户体验。所以，新版 chrome 决定对用document.write 注入第三方脚本的方式采取一些干预措施。

<!-- more -->

最近，可能也有人像我一样，看某些网页的时候，chrome 控制台出现了下面的警告：

``` 
A Parser-blocking, cross-origin script, https://paul.kinlan.me/ad-inject.js, is invoked via document.write. This may be blocked by the browser if the device has poor network connectivity.
```
 
[官方](https://developers.google.com/web/updates/2016/08/removing-document-write) 指出chrome 即将干预通过 document.write 引入的脚本了，[我们应该避免使用这种写法](http://blog.dareboost.com/en/2016/09/avoid-using-document-write-scripts-injection/)：

``` 
document.write('<scri'+'pt  src = "https://paul.kinlan.me/ad-inject.js"></scri'+'pt>');
```

注意，以上代码中 script 关键字需要拆开，否则会和前面或者后面的 script 配对，从而造成代码块提前结束，出现错误。

那么，使用 document.write 会有什么问题？

在浏览器渲染页面之前，它会先解析 html 标签来构建 dom 树。每当解析器遇到了一个脚本，它就会暂停解析 html，下载并解析执行这个脚本，脚本执行完再继续解析 dom，如果，这个脚本又动态注入另一个或多个脚本，那解析器还得等待更长时间才能继续往下走，这会明显拖慢网页渲染速度。

chrome 测试发现，使用 2g 这种网络速度缓慢的用户，通过 document.write 动态注入的脚本可能会使得页面显示延迟数十秒，甚至加载失败。另外，在 2g 网络上，使用 document.write 插入第三方脚本的网页通常比其他网页加载慢两倍。

于是，chrome 决定，当满足以下所有条件时，chrome 就不会执行通过 document.write 注入的script 脚本：

1.用户网络缓慢，特别是 2g 网络（不过，以后可能会包括比较缓慢的 3g，wifi 网络）；

2.document.write 用在顶级文档中（之所以不包括 iframe 页面，是因其不会阻止主页面的渲染）；

3.document.write 引入的同步脚本（具有 async 或 defer 等属性的异步脚本不会阻塞页面渲染）；

4.该脚本不在浏览器缓存中（缓存了的脚本并不会产生网络延迟，所以仍会执行）；

5.页面不是重新加载（如果用户主动重新加载该页面，chrome 就不会干预，并正常执行该网页）。

自 chrome 53 开始，如果 document.write 请求符合以上的条件 2-5,chrome 就会发出上文的警告。自 2016 年 10 月中旬发布的 chrome 54 开始 2g 用户的注入脚本将被干预，以后，任何连接缓慢的用户（即缓慢的 3g 或 WiFi）都可能被干预。
