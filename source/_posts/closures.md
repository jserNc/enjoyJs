---
title: JavaScript闭包（译）
date: 2016-11-01 15:55:44
tags: js
---

原文： http://dmitrysoshnikov.com/ecmascript/javascript-the-core/


**在ECMAScript中，函数是第一级（first-class）对象。**这意味着函数可以做为参数传递给其他函数（在那种情况下，这些参数叫作「函数类型参数」）。接收「函数类型参数」的函数叫作**高阶函数**。同样函数也可以从其他函数中返回。返回其他函数的函数叫作**以函数为值的函数**。

<!-- more -->

首先，我们说，**JavaScript有两种作用域：全局作用域和函数作用域**。函数内部可以直接读取全局变量，反之则不行！

```
var n = 999;

function f1() {
    console.log(n);
}

f1() // 999
```

但是，在函数外部无法读取函数内部声明的变量。

```
function f1() {
  var n = 999;
}

console.log(n)
// Uncaught ReferenceError: n is not defined(…)
```

如果一个函数（子函数）是从另一个函数（父函数）的返回值。为了在即使父函数上下文结束的情况下也能访问其中的变量，子函数在被创建的时候会在它的 [[Scope]] 属性中保存父函数的作用域链。所以当子函数被调用的时候，它上下文的对象的作用域链属性会被格式化成当前函数的变量对象（函数的变量对象[也叫活动对象 AO]也是一个同样简单的变量对象，但是除了变量和函数声明之外，它还存储了形参和 arguments 对象）与[[Scope]]属性（父函数的作用域链）的和：

**Scope chain = Activation object + [[Scope]]**

强调一个关键点：子函数在**创建时刻**会保存父函数的作用域链，将来子函数调用时会在这个作用域链中查找变量。注意是**创建时刻**，而不是**调用时刻**。

```
function foo() {
  var x = 10;
  return function bar() {
    console.log(x);
  };
}

var returnedFunction = foo();

var x = 20;

returnedFunction();     //10,而不是20
```

**js 中作用域叫作静态（或者词法）作用域。**我们看到变量 x 在返回的 bar 函数的 [[Scope]] 属性中被找到。如果说 bar 函数的 [[Scope]] 属性中找不到，继续去全局环境中找，我们可以去掉 foo 函数中的 x 变量，验证一下这个结论。

```
function foo() {
  return function bar() {
    console.log(x);
  };
}

var returnedFunction = foo();

var x = 20;

returnedFunction();     //20
```

果然，父函数找不到时，就会去全局作用域中找。

我们再看另外一个问题：** 当一个全局函数在另一个函数内部被调用，变量会怎么取值？**

```
var x = 10;

function foo() {
  console.log(x);
}

(function (funArg) {

  var x = 20;

  funArg(); // 10, but not 20

})(foo); 
```

我们看到 x 并未到父函数去取值，而是去其声明时所在环境（全局作用域）中取值。那么，如果，我们去掉全局作用域中变量 x 的声明，会怎样？

```
function foo() {
  console.log(x);
}

(function (funArg) {

  var x = 20;

  funArg(); 
  //Uncaught ReferenceError: x is not defined(…) 

})(foo); 
```

我们看到，报错了！！

其实，这里有一个重要结论：**函数（function）声明时所在的环境代表其作用域方向！！也就是说函数的作用域链是在其声明时确定，而不是其调用时！！**

换句话讲，函数内标识符该使用哪个作用域的值——以静态的方式存储在函数创建时刻的作用域还是在执行过程中以动态方式生成的作用域？JavaScript采用的是前者！**JavaScript没有动态作用域。**

下面，我们给闭包下个定义：

> **闭包：**在 JavaScript 语言中，只有函数内部的子函数才能读取内部变量，因此可以把闭包简单理解成**“定义在一个函数内部的函数”**。闭包最大的特点，就是它可以**“记住”**诞生的环境，比如内部函数 f2 记住了它诞生的环境外部函数 f1，所以从 f2 可以得到 f1 的内部变量。在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

闭包的最大用处有两个，**一个是可以读取函数内部的变量，另一个就是让这些变量始终保持在内存中**，即闭包可以使得它诞生环境一直存在。下面的例子，闭包使得内部变量记住上一次调用时的运算结果。

```
function createIncrementor(start) {
  return function () {
    return start++;
  };
}

var inc = createIncrementor(5);

inc() // 5
inc() // 6
inc() // 7
```

上面代码中，start 是函数 createIncrementor 的内部变量。通过闭包，start 的状态被保留了，每一次调用都是在上一次调用的基础上进行计算。从中可以看到，闭包 inc 使得函数 createIncrementor 的内部环境，一直存在。所以，闭包可以看作是函数内部作用域的一个接口。

为什么会这样呢？原因就在于 inc 始终在内存中，而 inc 的存在依赖于 createIncrementor，因此也始终在内存中，不会在调用结束后，被垃圾回收机制回收。

我们可以断定**静态作用域**是一门语言拥有**闭包的必需条件**。下面从另一个角度来定义闭包：

> **闭包**是一个代码块（在ECMAScript是一个函数）和以静态方式/词法方式进行存储的所有父作用域的一个集合体。所以，通过这些存储的作用域，函数可以很容易的找到自由变量。

由于每个（标准的）函数都在创建的时候保存了 [[Scope]]，所以理论上来讲，ECMAScript 中的所有函数都是闭包。

另一个问题，多个函数可能拥有相同的父作用域（这是很常见的情况，比如当我们拥有两个内部/全局函数的时候）。在这种情况下，[[Scope]] 属性中存储的变量是在拥有相同父作用域链的所有函数之间共享的。一个闭包对变量进行的修改会体现在另一个闭包对这些变量的读取上：

```
function baz() {
  var x = 1;
  return {
    foo: function foo() { return ++x; },
    bar: function bar() { return --x; }
  };
}

var closures = baz();

console.log(
  closures.foo(), // 2
  closures.bar()  // 1
);
```

这个问题在创建的函数中使用循环计数器的时候就很明显了:

```
var data = [];

for (var k = 0; k < 3; k++) {
  data[k] = function () {
    console.log(k);
    //多个函数共享一个[[Scope]]，而[[Scope]]中k值一直变
  };
}

data[0](); // 3, but not 0
data[1](); // 3, but not 1
data[2](); // 3, but not 2
```

以上这些函数拥有同一个[[Scope]]，这个属性中的循环计数器的值是最后一次所赋的值。相当于以下写法：

```
var data = [];
var k = 0;

data[0] = function(){
    console.log(k);
}
k++;

data[1] = function(){
    console.log(k);
}
k++;

data[2] = function(){
    console.log(k);
}
k++;

data[0](); // 3
data[1](); // 3
data[2](); // 3
```

要想依次输出0,1,2，我们可以这样定义函数：

```
var data = [];

for (var k = 0; k < 3; k++) {
  data[k] = (function (x) {
    return function () {
      console.log(x);
    };
  })(k); // pass "k" value
}

// now it is correct
data[0](); // 0
data[1](); // 1
data[2](); // 2
```

以上相当于：

```
var data = [];

data[0] = (function(x){
    return function () {
        console.log(x);
    };
})(0);

data[1] = (function(x){
    return function () {
        console.log(x);
    };
})(1);

data[2] = (function(x){
    return function () {
        console.log(x);
    };
})(2);
```

我们再看一个函数：

```
var fn = (function(x){
    return function () {
        console.log(x);
    };
})();

fn();    //undefined
fn(1);   //undefined
```

如果去掉外层函数的形参x，会怎样？

```
var fn = (function(){
    return function () {
        console.log(x);
    };
})();

fn();
Uncaught ReferenceError: x is not defined(…)
```

可以看到，内层函数的作用域确实是和其外层函数息息相关！


参考：
[1] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
[2] http://ourjs.com/detail/529bc7e44c742ef031000002
[3] http://javascript.ruanyifeng.com/grammar/function.html

