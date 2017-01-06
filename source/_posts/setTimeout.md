---
title: JavaScript中setTimeout该如何理解？
date: 2016-11-03 14:46:59
tags: js
---

setTimeout(code,ms) 方法用于在指定的毫秒数后调用函数或计算表达式。这种定时功能我们又叫定时器，该方法返回一个整数num，标识该定时器，之后可以用clearTimeout(num)方法来取消该定时器。

<!-- more -->

我们先看一个简单的例子：

```
var timer = setTimeout("console.log(1)",3000);
```
3秒（3000毫秒）后，会输出1。setTimeout方法发第一个参数可以是引号包含的代码、待执行的函数名以及匿名方法等，第二个参数是延迟执行的毫秒数，是我们换成以下两种写法：

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

下面我们用clearTimeout()取消定时器：

```
var timer = setTimeout(function(){
    console.log(1);
}, 3000);

clearTimeout(timer);
```

这样3秒之后就不会再输出1了。

再看一个例子：

```
var startTime = new Date();
setTimeout(function(){
    console.log('延时：',new Date() - startTime);
}, 200);
```

执行以上代码，会输出多少呢？

chrome控制台下连续执行3次，结果依次是：{延时：201,201,202}。下面对以上代码稍作修改：

```
var startTime = new Date();

setTimeout(function(){
    console.log('延时：',new Date() - startTime);
}, 200);

while((+new Date() - startTime) < 1000){
    console.log('...');
}
```

再次执行3次，结果依次是{延时：1001,1000,1000}。

根据以上对比，我们可以断定，上述打印结果res取决于后面的同步任务执行时间（忽略延时代码执行的几毫秒的误差），即：

> res = max(同步执行时间，200)

再看一个例子：

```
setTimeout(function(){
    console.log(1);
}, 0);
console.log(2);
```

以上代码，我们写的延迟时间是0，我们期待的是立刻输出1。然而，执行以后我们发现，先输出2，然后才输出1。为什么会这样呢？

首先，我们明确一个概念：JavaScript是**单线程***的语言，在执行时必须等前面的任务处理完以后才会处理后面的；而且是一个一个连续同步执行的。JavaScript中的任务分为**同步任务**和**异步任务**，同步任务就是主线程上一个个排队执行的任务；异步任务则不进入主线任务而是被加入到任务队列中，任务队列的任务只有在主线任务执行完成之后才去处理任务队列中的任务。

setTimeout方法中待执行的代码就是异步任务。延时参数设为0，仅仅表示同步任务执行完后尽可能早地执行该异步任务。0毫秒的延迟在实际中也不存在的。为了防止多个setTimeout(f,0)连续执行造成性能问题，html5规定，setTimeout方法指定的时间最少是4毫秒。如果小于这个值，会自动增加到4毫秒。当然了，设置的延迟时间太长也不行。setTimeout最多只能延迟24.8天（2147483647毫秒，浏览器内部使用32位带符号的整数，来储存推迟执行的时间），超过这个时间，等同于延迟时间设为0，即setTimeout(f,0)。


由于要等到本次事件循环的同步任务执行完才可能执行指定的异步任务。而前面的同步任务要多久才执行完是没法保证的。也就是说无法保证延迟任务一定按照setTimeout方法指定的延迟时间执行。

```
setTimeout(someTask,100);
veryLongTask();
```

上面代码指定100毫秒后运行一个任务。但是，如果后面立即执行的任务（当前脚本的同步任务）非常耗时，运行超过100毫秒无法结束。那么，推迟的someTask只有等着，等到前面的veryLongTask运行结束才开始执行。

实际应用中，为了防止阻塞，我们可以把耗时的大块任务分成一个个小块，每个小块用setTimeout(f,0)方法调用，即将大任务分成一个个小任务，让浏览器在空余时间尽早执行。

假如有三个函数处理时间都很长

```
longTask1();
longTask2();
longTask3();
```

可以用setTimeout把这些大任务拆开。这样就可以腾出时间来响应其他任务了。

```
var arr = [longTask1, longTask2, longTask3],
    len = arr.length,
    i = 0;
for(; i < len; i++){
	setTimeout(arr[i], 0);
}
```

再看个例子：

```
var div = document.getElementsByTagName('div')[0];

// 写法一
for (var i = 0xA00000; i < 0xFFFFFF; i++) {
  div.style.backgroundColor = '#' + i.toString(16);
}

// 写法二
var timer;
var i=0x100000;

function func() {
  timer = setTimeout(func, 0);
  div.style.backgroundColor = '#' + i.toString(16);
  if (i++ == 0xFFFFFF) clearTimeout(timer);
}

timer = setTimeout(func, 0);
```

上面代码有两种写法，都是改变一个网页元素的背景色。写法一会造成浏览器“堵塞”，因为JavaScript执行速度远高于DOM，会造成大量DOM操作“堆积”，而写法二就不会，这就是setTimeout(f, 0)的好处。

参考：
[1] http://javascript.ruanyifeng.com/advanced/timer.html