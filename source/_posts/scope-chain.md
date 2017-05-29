---
title: JavaScript 作用域链（译）
date: 2016-11-01 09:46:50
tags: js
---

原文： http://dmitrysoshnikov.com/ecmascript/javascript-the-core/

**作用域**（scope）控制着变量和参数的**可见性**及**生命周期**。

通常来说，一段程序代码中所用的变量并不是总是可用的，而限定这个变量的可用性的代码范围就是这个变量的作用域。

> **作用域链**是一个对象列表，**执行上下文**中出现的变量标识符在这个列表中进行查找。执行上下文的建立阶段会建立作用域链。

<!-- more -->

作用域链和原型链类似：如果一个变量在函数自身的作用域（自身的变量对象）中没有找到，那么将会在它的父函数（外层函数）的变量对象中查找，以此类推！

给作用域链下个定义：**当前执行上下文的变量对象（variableObject，包括函数的arguments对象, 参数, 内部的变量以及函数声明等）和其所有父执行上下文的变量对象组成的集合。**


举个例子：

```
var x = 10;

(function foo() {

  var y = 20;

  (function bar() {

    var z = 30;

    console.log(x + y + z);     //60
  })();
})();
```

我们看到最内层函数 bar 能读到其外层函数 foo 以及全局作用域中的变量。

我们可以假设通过隐式的 \_\_parent\_\_ 属性来和作用域链对象进行关联，这个属性指向作用域链中的下一个对象。利用 \_\_parent\_\_ 概念，我们可以用下面的图来表现上面的例子（并且父变量对象存储在函数的 [[Scope]] 属性中）：

![作用域链](/css/images/scope-chain/scope-chain.png)

（另外，作用域链可以通过使用 with 语句和 catch 从句对象来增强，这里就不讨论了。）

在 js 语言里，全局作用域可以理解为 window 对象，记住 window 是对象而不是构造函数或类，也就是说 window 是已经被实例化过的对象，这个实例化过程是在页面加载时由 js 引擎完成的。虽然我们在开发过程中不能控制这个实例化过程，但是我们不要忽略这个事实。

另外，在 js 语言里，任何匿名函数都是属于 window 对象，它们也都是在全局作用域构造时候完成定义和赋值的。但是匿名函数是没有名字的函数变量，只是在定义匿名函数时会返回其内存地址，如果此时有个变量接受了这个内存地址，那么这个匿名函数就能在其他地方调用了。

最后，说明一点：** 函数本身也是一个值，它的作用域绑定函数声明时所在的环境。**

```
var a = 1;
var f1 = function(){
    console.log(a);
}

function f2(){
    var a = 2;
    f1();
}

f2();
// 1, but not 2
```

以上打印结果是 1，而不是 2。函数 f1 是在 f2 外部的全局环境中声明的，f1 的作用域绑定全局环境，所以取不到 f2 内部的a。鉴于这点，容易犯个错误：** 函数 A 调用函数 B，却没有考虑到 B 并不会调用函数 A 内部定义的变量。** 还是上面的例子，做点修改：

```
var f1 = function(){
    console.log(a);
}

function f2(){
    var a = 2;
    f1();
}

f2();
// Uncaught ReferenceError: a is not defined
```

我们看到，这里会报错，f1 根本不能取到 f2 内部定义的变量 a。

下面再看一个“惰性函数/自定义函数”：

```
var v = 1;
var f = function(){
    var v = 2;
    console.log('第一次出现');
    f = function(){
        console.log(v);
    }
}

f()
// 第一次出现

f()
// 2
```

当函数有一些初始化准备工作要做，并且仅需执行一次，这种模式就很有用了。同时，我们也看到，虽然函数 f 是全局变量，但它第二次执行的时候，变量 v 取值是 2，而不是 1。

再看：

```
// html
<p id="p1">段落 1</p>

//js
var f = function(){
    console.log('f 函数的激活者：',this);
},
p1 = document.getElementById("p1"),
p2 = '<p id="p2" onclick="console.log(this)">段落2</p>',
p3 = '<p id="p3" onclick="f()">段落3</p>';

p1.addEventListener("click",f,false);
document.write(p2);		
document.write(p3);
```

点击段落 1、2、3，打印信息分别如下：

段落 1：
```
f 函数的激活者： <p id=​"p1">​段落1​</p>​
```

段落 2：
```
<p id="p2" onclick="console.log(this)">段落2</p>
```

段落 3：
```
f 函数的激活者：Window {stop: function, open: function,…}
```

下面把上面的 js 代码用自执行函数包围起来：

```
(function(){
  var f = function(){
      console.log('f 函数的激活者：',this);
  },
  p1 = document.getElementById("p1"),
  p2 = '<p id="p2" onclick="console.log(this)">段落2</p>',
  p3 = '<p id="p3" onclick="f()">段落3</p>';

  p1.addEventListener("click",f,false);
  document.write(p2);		
  document.write(p3);
})();
```

这时再点击段落1、2 打印的信息和以上一样，但是点击段落3 时报错了：

```
Uncaught ReferenceError: f is not defined
```

把 f 函数定义在全局环境下：

```
var f = function(){
      console.log('f 函数的激活者：',this);
};
(function(){
  var p1 = document.getElementById("p1"),
  p2 = '<p id="p2" onclick="console.log(this)">段落2</p>',
  p3 = '<p id="p3" onclick="f()">段落3</p>';

  p1.addEventListener("click",f,false);
  document.write(p2);		
  document.write(p3);
})();
```

这时点击段落3 的打印信息又和之前一样了：

```
f 函数的激活者：Window {stop: function, open: function,…}
```

其实，如果我们把回调函数部署在节点元素的 on- 属性上，回调函数内的 this 就不会再指向该节点元素了，而是指向全局对象 window，回调函数由 window 对象触发。这里 p3 正是这种方式绑定点击回调函数：

```
onclick="f()"
```

这里的 f 函数是由 window 对象触发的，若 f 是定义在自定义函数内部的函数，全局环境 window 找不到它那是必然的，所以才有了以上的报错。

段落1 的绑定方式是将函数 f 的赋值给 p1 的 onclick 属性，所以在点击事情发生时，不必再通过 f 了，这和段落 3 是有区别的。






参考：
[1] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
[2] http://ourjs.com/detail/529bc7e44c742ef031000002