---
title: 理解 call + bind
date: 2017-04-21 10:45:37
tags: grammar
---

JavaScript 提供了 call、apply、bind 等三个方法，来切换/固定函数调用时其内部 this 的指向。它们的第一个参数都是函数内 this 所要指向的对象，如果该参数设为 null 或 undefined，则指向全局对象（浏览器环境下，指 window）。这三个方法的基本用法在 [理解 JavaScript 中 this](http://nanchao.win/2016/11/02/this/) 一文已经做了介绍，这里就不再赘述。下面更深入地讨论一下 call 和 bind 方法。

<!-- more -->

**强调一下，call 方法是某函数 f 执行过程中调用的，而 bind 方法是声明函数的时候调用的。即，调用 call 方法时会先绑定函数 f 内部的 this，然后执行函数 f 并返回执行结果；调用 bind 方法会绑定函数 f 内部的 this，然后返回一个新的函数 newF。**

假定我们的需求是：从原数组 [1,2,3] 中索引为 0 的位置开始，分隔出长度为 1 的新数组。首先，我们会想到这么写（以下代码均在 chrome 控制台执行）：

```
[1,2,3].slice(0,1)
// [1]

// call 方法的作用是将一个方法的 this 
// 绑定到一个新的对象

// 所以，以上写法相当于：
Array.prototype.slice.call([1,2,3],0,1)
// [1]
```

这里的 call 方法还是它以前的样子。

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

为了加深理解，我们再换个角度看：

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

以上代码，将全局的 call、slice 变量都分别指向 Function.prototype.call、Array.prototype.slice 方法。当然了，我这么写是为了偷懒，其实完全没必要新增加这两个全局变量的。

再看：

```
function f(){
    console.log(this.v);
}

var o = {v : 123};

f.bind(o)();
// 123

f.bind === Function.prototype.bind;
// true
```

注意看：

```
f.bind(o)();
```

这句代码本质是：将函数 Function.prototype.bind 内部 this 指向 f（f 是激活 bind 执行上下文的执行者），然后传入实参对象 o，执行该 bind 方法，返回一个新的匿名方法，最后执行该匿名方法。

换种写法：

```
Function.prototype.bind.call(f,o)();
// 123
```

这句代码本质是：把函数 Function.prototype.bind 内部的 this 绑定到 f 对象（函数），然后将第二个实参（对象 o） 作为 Function.prototype.bind 函数的实参，最后执行该方法，返回一个匿名方法，最后执行该匿名方法。

下面利用 Function.prototype.call 方法结合 bind 方法，新生成一个方法：

```
var call = Function.prototype.call;
var bind = Function.prototype.bind;

var newCall = call.bind(bind);

newCall(f,o)
// function f(){
//    console.log(this.v);
// }

newCall(f,o)()
// 123
```

来理解一下这句：

```
var newCall = call.bind(bind);
```

把函数 Function.prototype.call 内部的 this 绑定到函数 Function.prototype.bind，然后返回一个新的函数 newCall，newCall 与 Function.prototype.call 的区别在于，前者在被调用的时候 this 始终指向 Function.prototype.bind，而后者每次被调用时 this 都会随着激活该函数上下文的执行者（比如调用该函数的对象，再比如调用该函数的外层上下文对象）变化。所以，我们暂且把 newCall 当做 Function.prototype.call 吧，只是 newCall 在被调用时 this 是固定的，所以，newCall 相当于：

```
Function.prototype.bind.call
```

再看：

```
newCall(f,o)

// 相当于：
Function.prototype.bind.call(f,o);

//所以：
newCall(f,o)();
// 123

// 相当于：
Function.prototype.bind.call(f,o)();
// 123
```

以上推理过程也许有点绕，但是，我相信读懂它们对于理解 call 和 bind 方法是很有帮助的。











参考：
[1] http://javascript.ruanyifeng.com/oop/this.html