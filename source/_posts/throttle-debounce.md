---
title: 去抖和节流（debounce & throttle）
date: 2017-01-06 09:43:00
tags: method
---

关于函数去抖和节流，一言以蔽之，去抖是为了防止短时间内多次触发同一事件，造成意外的结果，比如连续点击抽奖按钮；节流是为了防止某事件触发的频率过高，造成性能问题，比如拖动滚动条连续密集触发scroll事件。

<!-- more -->

举两个例子：

** 场景一：**

窗口的resize事件会在改变浏览器大小事连续触发； onmousemove事件会在鼠标移动时被连续触发。如果这些事件的回调函数过重，而不采取节流处理，可能会使浏览器崩溃。

** 场景二：**

年会现场，主持人邀请嘉宾抽取1位特等奖幸运小伙伴，如果嘉宾不小心连续点击了两次【特等奖】按钮，而没有采取去抖处理，可能会连续触发两次抽取程序，然后出现2位获奖小伙伴，这就尴尬了。

下面详细地阐述一下去抖和节流的概念以及实现。

### 去抖（debounce） ###

如果用手指一直按住一个弹簧，虽然它有弹起的趋势，但是只要你按住不放（保持对它施力），它将不会弹起直到你松手为止。

也就是说，假定事件A会触发回调函数callback，当事件A连续发生时，理论上会连续触发回调函数callback。但是，只要短时间内连续触发的callback我们就不让其执行，直到时间A发生超过指定时间后，我们才调用一次callback。即：当调用动作n毫秒后，才会执行该动作，若在这n毫秒内又调用此动作则将重新计算执行时间。

debounce函数简单实现如下：

```
/**
* 空闲时间必须大于或等于 idle，action 才会执行
* @param idle   {number}    空闲时间，单位毫秒
* @param action {function}  实际执行的函数
* @return {function}        返回事件需绑定的回调函数
*/
var debounce = function(idle, action){
    var last = 0;
    return function(){
        var ctx = this, 
            args = arguments;

        clearTimeout(last);

        last = setTimeout(function(){
            action.apply(ctx, args);
        }, idle);
    };
};
```

假如我们的需求是，拖动滚动条2秒后，打印“你拖动了滚动条”，如果持续拖动，则不打印。可以这么写：

```
var f = function (){
    console.log('你拖动了滚动条');
};
window.addEventListener('scroll',debounce(2000,f),false);
```

### 节流（throttle） ###

如果将水龙头拧紧直到水是以水滴的形式流出，发现每隔一段时间，就会有一滴水流出。

也就是说，假定事件A会触发回调函数callback，当事件A连续发生时，理论上会连续触发回调函数callback。但是，如果两次调用callback函数的事件间隔小于指定时间，就不让callback函数执行，当下次调用callback函数并且和上次调用时间间隔超过指定时间，再执行一次callback。即：函数无法在很短的时间间隔内连续调用，当过了规定的时间间隔，才能进行该函数的下一次调用。

throttle函数简单实现如下：

```
/**
* 频率控制，action 执行频率限定为 1次 / delay
* @param delay  {number}    延迟时间，单位毫秒
* @param action {function}  实际执行的函数
* @return {function}    返回事件需绑定的回调函数
*/
var throttle = function(delay, action){
    var last = 0;
    return function(){
        var curr = +new Date();
        if (curr - last > delay){
            action.apply(this, arguments);
            last = curr;
        }
    };
}
```

假如我们的需求是，拖动滚动条的时候，每隔最少2秒打印一次“你拖动了滚动条”。可以这么写：

```
var f = function (){
    console.log('你拖动了滚动条');
};
window.addEventListener('scroll',throttle(2000,f),false);
```

参考：
[1] http://www.cnblogs.com/fsjohnhuang/p/4147810.html
[2] http://blog.csdn.net/dyllove98/article/details/9281507