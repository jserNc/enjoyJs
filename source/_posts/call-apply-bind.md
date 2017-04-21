---
title: 理解 call + bind
date: 2017-04-21 10:45:37
tags:
---

JavaScript 提供了 call、apply、bind 等三个方法，来切换/固定函数内部 this 的指向。它们的第一个参数都是函数内 this 所要指向的对象，如果设为 null 或 undefined，则指向全局对象（浏览器环境下，指 window）。它们的基本用法在 [理解 JavaScript 中 this](http://nanchao.win/2016/11/02/this/) 一文已经做了介绍，这里不再赘述。本文更深入地讨论一下 call 和 bind方法。

<!-- more -->

我们的需求是：从原数组 [1,2,3] 中索引为 0 的位置开始，分隔出长度为 1 的新数组。

```
// 首先会想到这么写：
[1,2,3].slice(0,1)
// [1]

// call 方法的作用是将一个方法的 this 
// 绑定到一个新的对象

// 所以，以上写法相当于：
Array.prototype.slice.call([1,2,3],0,1)
// [1]
```

我们看到 call 方法的调用主体是 slice 方法，并且这时的 call 方法还是以前的样子。

```
Array.prototype.slice.call === Function.prototype.call
```

换个角度看，只要 call 的调用主体是 slice 方法，那么实参为 [1,2,3],0,1 这种形式，就可以完成从一个数组中切分出一个新的数组的功能。那么，我们用 bind 方法，把 call 方法的调用主体绑定为 slice 方法会怎样呢？

```
var call = Function.prototype.call;
var slice = Array.prototype.slice;

var newCall = call.bind(slice);
newCall([1,2,3],0,1);
// [1]

// 也就是说，以下两种方式等效：
newCall([1,2,3],0,1);
[1,2,3].slice(0,1);
```

为了加深理解，我们再换个角度看以上写法：

```
var newCall = call.bind(slice);

// 于是：
newCall([1,2,3],0,1);
// 相当于：
call.bind(slice)([1,2,3],0,1);
// [1]
// 即 call 方法的 this 绑定为 slice，
// 然后调用实参 [1,2,3],0,1

// 等效于：
call.call(slice,[1,2,3],0,1)
// [1]

// 也相当于在 slice 方法上调用 call 方法：
slice.call([1,2,3],0,1);
// [1]

// 也相当于：
[1,2,3].slice(0,1);
// [1]
```

也就是说，以下四种写法等效：

```
// 这里的定义会影响写法一二三：
var call = Function.prototype.call;
var slice = Array.prototype.slice;

// 写法一：
call.bind(slice)([1,2,3],0,1);

// 写法二：
call.call(slice,[1,2,3],0,1);

// 写法三：
slice.call([1,2,3],0,1);

// 写法四：
[1,2,3].slice(0,1);
```

以上四种写法中，前提是将全局的 call、slice 方法都分别指向 Function.prototype.call、Array.prototype.slice 方法。当然了，我这么写是为了偷懒，完全没必要特地去新增加这两个全局变量的。










参考：
[1] http://javascript.ruanyifeng.com/oop/this.html