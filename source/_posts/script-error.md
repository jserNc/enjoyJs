---
title: script.onerror 的兼容写法
date: 2016-10-27 18:26:33
tags: problem
---

script的onerror事件IE6~8与opera11都不支持，但是，我们又经常遇到需要检测脚本是否出错的场景。下面给出一种可以兼容各种浏览器的解决方法：

<!-- more -->

```
function loadjs(url, obj){
    var s = document.createElement('script');
    var head = document.getElementsByTagName('head')[0];
    s.onreadystatechange = s.onerror = function(){
        if( !this.readyState || 
            ( (this.readyState ==='loaded' || 
               this.readyState ==='complete') && 
               !window[obj]) ){
            console.log('File Loaded Error');
        }
    };
    s.src = url;
    head.appendChild(s);
}
loadjs(myUrl, 'objName');
```
对非ie的其他浏览器，!this.readyState为true；而ie浏览器，当script.readyState为'loaded'或'complete'，则表示script加载完成或失败（404的时候，同样为loaded或complete），所以还要判断，此时某特定对象是否存在，来确定，究竟是文件加载完成，还是404等。

参考：
[1] http://www.cnblogs.com/chyingp/archive/2012/11/15/scriptOnerrorNotSupportedInIE6to8.html