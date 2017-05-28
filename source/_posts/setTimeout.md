---
title: 理解 JavaScript 中 setTimeout
date: 2016-11-03 14:46:59
tags: js
---

setTimeout(code,ms) 方法用于在 ms 毫秒后调用函数或计算表达式（code）。这种定时方法我们也叫定时器，该方法会返回一个整数 n，作为定时器的标识，之后可以用 clearTimeout(n) 方法来取消该定时器。

<!-- more -->

我们先看一个简单的例子：

```
var timer = setTimeout("console.log(1)",3000);
```
3000 毫秒（3 秒）后，会在控制台输出 1。

setTimeout 方法第 1 个参数可以是引号包含的代码、待执行的函数名或者匿名方法等，第 2 个参数是延迟执行的毫秒数，下面两种写法和上面实现同样的效果：

```
//写法一
function f(){
    console.log(1);
}
var timer = setTimeout(f,3000);

//写法二
var timer = setTimeout(function(){
    console.log(1);
}, 3000);
```

在定时任务定义后，实际执行之前，我们可以用 clearTimeout() 来取消该定时器：

```
var timer = setTimeout(function(){
    console.log(1);
}, 3000);

clearTimeout(timer);
```

这样 3 秒之后就不会再输出 1 了。

setTimeout 基本用法就是这样，下面我们再来看看它的其他作用。

```
var startTime = new Date();
setTimeout(function(){
    console.log('延时：',new Date() - startTime);
}, 200);
```

执行以上代码，会输出多少呢？

chrome控制台下连续执行 3 次，结果依次是：{ 延时：201,201,202 }。下面对以上代码稍作修改：

```
var startTime = new Date();

setTimeout(function(){
    console.log('延时：',new Date() - startTime);
}, 200);

while((+new Date() - startTime) < 1000){
    console.log('...');
}
```

再次执行 3 次，结果依次是  { 延时：1001,1000,1000 }。

根据以上对比，我们发现，上述打印结果 res 取决于后面的同步任务执行时间（忽略延时代码执行的几毫秒的误差），即 res 是“同步任务执行时间”和“指定的延迟时间”之间的较大者：

> **res = max(同步任务执行时间，200)**

好，再看：

```
setTimeout(function(){
    console.log(1);
}, 0);
console.log(2);
```

以上代码，我们写的延迟时间是 0，我们期待的是立刻输出 1。然而，执行以上代码我们发现，先输出 2，然后才输出 1。为什么会这样呢？

首先，我们明确一个概念：**JavaScript 是单线程的语言，在执行时必须等前面的任务处理完以后才会处理后面的。JavaScript 中的任务分为同步任务和异步任务，同步任务就是主线程上一个个排队执行的任务；异步任务则不进入主线任务而是被加入到任务队列中，任务队列的任务只有在主线任务执行完成之后才去处理任务队列中的任务。**

setTimeout 方法中待执行的代码就是异步任务。延时参数设为 0，仅仅表示同步任务执行完后尽可能早地执行该异步任务。0 毫秒的延迟在实际中也不存在的。为了防止多个 setTimeout(f,0) 连续执行造成性能问题，html5 规定，setTimeout 方法指定的时间最少是 4 毫秒。如果小于这个值，会自动增加到 4 毫秒。当然了，设置的延迟时间太长也不行。setTimeout 最多只能延迟 24.8 天（ 即 2147483647 毫秒，因为浏览器内部使用 32 位带符号的整数，来储存这个延迟执行的时间），超过这个时间，等同于延迟时间设为 0，即 setTimeout(f,0)。


由于要等到本次 Event Loop 的同步任务执行完才可能执行指定的异步任务。而前面的同步任务要多久才执行完是没法保证的。也就是说无法保证延迟任务一定按照 setTimeout 方法指定的延迟时间执行。

```
setTimeout(someTask,100);
veryLongTask();
```

上面代码指定 100 毫秒后运行一个任务。但是，如果后面立即执行的任务（当前脚本的同步任务）非常耗时，运行超过 100 毫秒无法结束。那么，推迟的 someTask 只有等着，等到前面的 veryLongTask 运行结束才开始执行。

实际应用中，为了防止阻塞，我们可以把耗时的大块任务分成一个个小块，每个小块用 setTimeout(f,0) 方法调用，即将大任务分成一个个小任务，让浏览器在空余时间尽早执行。

假如有三个函数处理时间都很长

```
longTask1();
longTask2();
longTask3();
```

可以用 setTimeout 把这些大任务拆开。这样就可以优先来响应其他小任务了，不至于堵在这种耗时大任务这里。

```
var arr = [longTask1, longTask2, longTask3],
    len = arr.length,
    i = 0;
for(; i < len; i++){
	setTimeout(arr[i], 0);
}
```

再看：

```
var div = document.getElementsByTagName('div')[0];

// 写法一
for (var i = 0xA00000; i < 0xFFFFFF; i++) {
  div.style.backgroundColor = '#' + i.toString(16);
}

// 写法二
var timer;
var i = 0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#' + i.toString(16);
  if (i++ === 0xFFFFFF) {
      clearTimeout(timer);
  } 
}

timer = setTimeout(func, 0);
```

上面代码有两种写法，都是改变一个网页元素的背景色。写法一会造成浏览器“堵塞”，因为 DOM 操作消耗较大，这种连续的 DOM 操作会造成大量 DOM 操作“堆积”，浏览器卡顿。而写法二将任务分散在空余时间执行，就不会造成“堵塞”，这就是 setTimeout(f, 0) 的好处。

setTimeout 和 setInterval 的运行机制是：将指定的代码移除本次执行队列，等到下一轮 Event Loop 时，再检查是否到了指定的延迟时间。如果到了，就执行指定的代码；否则在下一轮 Event Loop 时重新判断。

每一轮 Event Loop，都会将“任务队列”中需要执行的代码一次执行完。setTimeout 和 setInterval 都是把任务添加到“任务队列”的尾部。因此它们要等到当前脚本的所有同步任务执行完，然后再等到本次 Event Loop 中的“任务队列”中的所有任务执行完，才开始执行。而前面的任务到底要多久才能执行完是不确定的，所以，不能保证 setTimeout 和 setInterval 指定的任务一定会按照预定时间执行。



参考：
[1] http://javascript.ruanyifeng.com/advanced/timer.html