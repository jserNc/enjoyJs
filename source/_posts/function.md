---
title: JavaScript 函数
date: 2017-06-09 09:13:08
tags: grammar
---

【function 命令】和【函数表达式】都是定义函数的方式，他们用法大同小异，在很多场景中可以通用。本文主要讨论一下它们之间的差异，同时也可以更深入地理解执行上下文、this 等概念。

<!-- more -->

首先看一个小例子：

```
function f1() {}
(function f2() {});

console.log(typeof f1); // function
console.log(typeof f2); // undefined
```

对以上代码执行结果有没有感到意外呢？**其实，第一种方式就是【function 命令】定义函数；第二种方式是【函数表达式】，函数表达式中关键词 function 后的函数名只在函数内部有效，函数外无效。**

注意：【function 命令】定义函数语句后面不能跟分号（;），【函数表达式】后最好跟分号（;）。

一般情况下，我们以为的函数表达式都长这样：

```
var f = function() {};
```

实际上，**只要 function 关键词不在行首，就是函数表达式！**这是理解 JavaScript 中【立即调用函数】的关键。我们知道，在 JavaScript 中，() 也是一种运算符，跟在函数名后面表示调用该函数，如 f() 表示执行 f 函数。【立即调用函数】实质定义一个函数后，立即调用这个函数。很自然，会想到这么写：

```
function f(){}()
// Uncaught SyntaxError: Unexpected token )
```

很遗憾！这句代码报错了，语法错误。因为 function 关键词既可以出现在函数声明语句中，也可以出现在函数表达式中，这么写会让 JavaScript 引擎比较为难不知道怎么解析，从而报错。

于是，引擎规定：如果 function 关键词出现在行首，就是函数声明语句，其后是不可以跟 () 运算符的；其他情况下都是函数表达式，其后可以跟 ()，表示执行该函数。

一般，立即调用函数两种写法：

```
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```

当然，以下方式也都可以的：

```
// 逗号，赋值、与等运算符
var i = function () { return 10; } ();
true && function () { /* code */ } ();
0, function () { /* code */ } ();

// 一元运算符在前，function 不在行首
!function () { /* code */ } ();
~function () { /* code */ } ();
-function () { /* code */ } ();
+function () { /* code */ } ();
```

再回到开头的那句代码：

```
(function f2() {});
```

现在不难理解它是函数表达式了吧。上面还说到函数表达式中的函数名只可以在函数内部使用，外部无法取到，所以，这里相当于声明一个匿名函数，然后也没调用这个函数就不管它了。

```
var f1 = function f2(){
    console.log('函数内部：',typeof f2)
};

f1();
// 函数内部： function

console.log('函数外部：',typeof f2);
// 函数外部： undefined
```

不过，函数的 name 属性是可以取到 function 关键词后的函数名的。

```
var f = function myName() {};
f.name 
// "myName"

(function f2() {}).name
// "f2"
```

再来看看两种函数定义在代码执行时的差异：

```
// 代码段 1
f1();
// function f1
function f1(){
    console.log('function f1');
}

// 代码段 2
f2();
// Uncaught TypeError: f2 is not a function
var f2 = function(){
    console.log('function f2');
}
```

function 命令声明的函数会提升到代码头部，所以，函数 f1 可以顺利执行；而变量 f2 的值开始只是 undefined，只有执行完赋值语句，它才指向一个函数，所以先于 f2 的赋值语句执行 f2() 必然会报错。

如果用 function 命令和函数表达式方式声明同名函数会怎样呢？

```
// 代码段 1
var f = function(){
    console.log('函数表达式');
}
function f(){
    console.log('函数声明');
}
f();
// 函数表达式

// 代码段 2
function f(){
    console.log('函数声明');
}
var f = function(){
    console.log('函数表达式');
}
f();
// 函数表达式
```

我们看到，不管是 function 命令在先还是函数表达式在先，最后函数都是采用函数表达式的定义。这样的结果不应该意外，因为在“代码执行阶段”，变量属性值是可以被覆盖的。

if、try 等代码块中的 function 命令和函数表达式也会不一样：

```
// 代码段 1
if (false){
    function f(){
        console.log('函数声明')
    }
}
f(); 

// 代码段 2
if (false){
    var f = function(){
        console.log('函数表达式');
    }
}
f();
```

以上代码执行结果就不写了，对于新版的 chrome，以上两种写法都会报错。但是，在 ie 和其他较旧的浏览器下，代码段 1 中的函数 f 是可以正常执行的。这是因为，JavaScript 是不存在块级作用域，这种 if 语句包含的变量（var）和 function 命令声明的函数也是会提升到代码头部的。

最后，注意一下变量声明语句的方式：

```
// 片段 1
var result = function(){
    return 2 + 2;
}

// 片段 2
var result = function(){
    return 2 + 2;
}();

// 片段 3
var result = (function(){
    return 2 + 2;
})();

// 片段 4
var result = (function(){
    return 2 + 2;
}());
```

其中片段 1 变量 result 是一个函数，片段 2/3/4 变量 result 是一个数值变量。这些细微差别得注意。

好，再看：

```
function f() {
  if (this === window){
      console.log("this 指向 window");
  } else if (this === o){
      console.log("this 指向 o");
  }
}

f(); 
// this 指向 window

var o = {
  f1: f
};

o.f1(); 
// this 指向 o

(o.f1)(); 
// this 指向 o

(o.f1 = o.f1)(); 
// this 指向 window

(o.f1, o.f1)(); 
// this 指向 window

(false || o.f1)(); 
// this 指向 window

var newf = o.f1;
newf(); 
// this 指向 window
```

这里的赋值、逗号，非运算结果都是返回最后一个运算子的值，即 o.f1，相当于把 o.f1 这个函数返回给了一个新的变量 newf，然后调用这个新的变量函数 newf()。**如果一个函数是类似于全局函数的方式调用，即函数执行时函数名前没有任何对象 f()，或者函数名前是 window 对象，window.f()，那么我们就认为这个函数内部 this 指向全局对象 window。**

参考：
[1] http://javascript.ruanyifeng.com/grammar/function.html
[2] http://ourjs.com/detail/529bc7e44c742ef031000002
[3] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
[4] http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html