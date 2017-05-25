---
title: ready 方法的实现
date: 2017-05-02 16:12:03
tags: method
---

很多时候，我们需要在页面加载完成后执行某些方法。我们首先可能会想到 window.onload 事件，但是，该事件会等到 dom、script、css、image 以及 iframe 等所有资源都加载完后才会触发，这么多资源都加载完是很耗时的，所以，比较好的办法是只在 dom 加载完后就执行这些方法。

<!-- more -->

为了判断 dom 是否加载完成，对于一般主流浏览器我们可以利用 DOMContentLoaded 事件。而对于部分不支持 DOMContentLoaded 事件的 ie 浏览器，我们可以通过 document.readyState 的状态值来判断（看其是否为 loaded | complete）。如果页面还包含 iframe，ie6~8 等浏览器还会等 iframe 资源加载完才会更新状态值，可能会导致等待时间比较长，这不是我们想要的。

不过，办法还是有的。ie 下页面元素会有个 doScroll 方法，如果 dom 没有加载完成调用该方法会报错，也就是说，我们可以循环调用该方法，只要它不报错了就表示 dom 已加载完成。

下面是 jQuery 库实现的 ready 方法思路：

```
(function(){
   // 判断 onDOMReady 方法是否已经被执行过
   var isReady = false; 
   // 把需要执行的方法先暂存在这个数组里
   var readyList= [];
   var timer;
   var html = document.documentElement;

   // 全局方法
   ready = function(fn) {
      if (isReady){
          // 函数队列已销毁，直接执行新的函数
          fn.call(document);
      } else {
          // dom 没加载完，函数加入队列
          readyList.push(function() { 
              return fn.call(this);
          });
      }
      // 这里的 this 可根据实际情况修改或者去掉 
      return this;
   }
   var onDOMReady = function(){
        // 依次执行队列中函数
        for(var i=0; i<readyList.length; i++){
            readyList[i].apply(document);
        }
        // 销毁函数队列
        readyList = null;
   }
   var bindReady = function(evt){
        // 该方法只在 dom 加载完成后执行一次
        if(isReady) return;
        isReady=true;
        // 主要回调函数
        onDOMReady.call(window);
        // 主流浏览器移除事件绑定
        if(document.removeEventListener){
             document.removeEventListener(
                 "DOMContentLoaded",
                 bindReady,
                 false
             );
        // IE 浏览器移除事件绑定
        }else if(document.attachEvent){
             document.detachEvent(
                 "onreadystatechange",
                 bindReady
             );
             if(window == window.top){
                  clearInterval(timer);
                  timer = null;
             }
        }
   };
   // 主流浏览器事件绑定
   if(document.addEventListener){
      document.addEventListener(
          "DOMContentLoaded",
          bindReady,
          false
      );
   // IE 浏览器移除事件绑定
   }else if(document.attachEvent){
      document.attachEvent(
          "onreadystatechange", 
          function(){
              var isDone = document.readyState;
              if((/loaded|complete/).test(isDone)){
                  bindReady();
              }
          }
       );
      // 非嵌套
      if(window == window.top){
          timer = setInterval(function(){
              try{
                  //判断 dom 是否加载完毕，没加载完会报错
                  isReady || html.doScroll('left');
              }catch(e){
                  // 报错就在这里返回
                  return;
              }
              // 不报错才会执行这里
              bindReady();
          },5);
      }
   }
})();
```

如果有多个函数绑定需要绑定到 dom 加载完成后执行，为了防止重复绑定，这里利用函数队列来保证多个函数有序执行。



参考：
[1] http://yevon.is-programmer.com/posts/30046.html