---
title: JavaScript 执行上下文（译）
date: 2016-10-31 10:23:26
tags: js
---

原文： http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/

执行上下文是 JavaScript 中一个最基础也是最重要的概念。理解了执行上下文才会对变量提升，this 指针，函数作用域等概念有更深入的认识。在介绍执行上下文之前，我们先来看个变量提升的例子：

<!-- more -->

```
var v = 1;
(function(){
    console.log('v:',v);
    var v = 2;
})();

//v: undefined
```

变量 v 明明在调用之前就已经在全局环境中定义过，可打印结果还是 undefined，为什么？

这里涉及到了“变量作用域”和“变量提升”两个概念。后续会对“变量作用域”做一个详细介绍，这里先简单说下变量提升。

不管全局环境中，还是函数调用时，JavaScript 引擎都先获取当前环境下用 var，function 等关键词声明的变量，然后再一行一行地执行代码。这样的结果就是所有的变量的声明语句，都会被提升到当前环境（全局环境或者函数内部）的头部，这就叫做“变量提升”（hoisting）。

以上代码的执行过程是：首先定义了一个全局变量 v，并对其赋值；然后定义了一个匿名函数，定义了同名的局部变量 v；最后自执行该匿名函数。因为函数内部用关键词 var 定义变量 v，所以函数执行时会将该变量声明（只是声明，不包括赋值）提升到函数内代码的头部，而不会去全局作用域中去取变量 v。整个过程相当于：

```
var v = 1;
(function(){
    var v;  //v = undefined
    console.log('v1:',v);
    v = 2;
    console.log('v2:',v);
})();
console.log('v3:',v);

//输出：
//v1: undefined
//v2: 2
//v3: 1
```

以上描述可能还不能够说清楚“变量提升”这个概念，相信看完后面关于执行上下文的内容，就会明白了。

## JavaScript 执行上下文

JavaScript代码有以下三种：

> ** 1.全局代码 ** 代码载入后，js引擎首先执行这里的代码；
> ** 2.函数代码 ** 函数调用时执行的代码；
> ** 3.eval代码 ** eval函数内运行的代码。

以上每种代码都是在其执行上下文中执行的，我们可以将“执行上下文”理解为当前代码运行的环境。全局上下文只有一个。函数执行上下文可以有多个。函数的每次调用（即便是这个函数递归调用自身）都会进入到一个新的函数执行上下文，也就是是说，同一个函数也可能会创建多个上下文。

例如：

![执行上下文](/css/images/execution-context/context1.png)

以上代码一共有 4 个执行上下文。最外层是全局上下文，其余 3 个为函数上下文。再强调一下：**函数上下文的个数是没有限制的，函数每次被调用时，引擎都会自动创建一个函数上下文。**

一个执行上下文可能会触发另一个上下文。比如，在全局上下文中调用一个函数或者一个函数调用另一个函数。这种关系是以栈的形式实现的，我们把它叫做“执行上下文栈”。

一个触发其他上下文的上下文叫作 caller。被触发的上下文叫作 callee。callee 同时也可能是一些其他 callee 的 caller（比如，一个在全局上下文中被调用的函数，在执行过程中又调用了其他函数）。

当一个 caller 触发（调用）了一个 callee，这个 caller 会暂停自身的执行，然后把控制权传递给 callee。这个 callee 被 push 到栈顶，并成为一个运行中（活动的）执行上下文。在 callee 的上下文结束后，它会把控制权返回给 caller，然后 caller 的上下文继续执行（它可能继续触发其他上下文）直到它结束，以此类推。

也就是说：JavaScript 程序运行时可以用“执行上下文栈”来表示，栈顶是当前“活跃上下文”。某个时刻只有唯一的一个上下文是激活（活跃）状态。

![执行上下文栈](/css/images/execution-context/stack.png)

当程序开始的时候它会进入全局执行上下文，此上下文位于栈底并且是栈中的第一个元素。当在全局上下文中调用执行一个函数时，程序流就进入该被调用函数内，此时引擎就会为该函数创建一个新的执行上下文，并且将其压入到执行上下文堆栈的顶部。浏览器总是执行当前在堆栈顶部的上下文，一旦执行完毕，该上下文就会从堆栈顶部被弹出，然后，进入其下的上下文执行代码。这样，堆栈中的上下文就会被依次执行并且弹出堆栈，直到回到全局的上下文。

例如：

```
//进入全局上下文
var a = 10;

function fn(){
    var b = 20;
    return a + b;
}

fn();   // 进入 fn 函数上下文
```
运行上述代码，执行上下文的压栈、出栈过程如下图：

![执行上下文栈](/css/images/execution-context/stack1.png)

（其中：EC 指 Execution Context，执行上下文）：

为了理解执行上下文这个抽象概念，有几点再强调一下：**单线程同步执行、唯一的一个全局上下文、函数上下文个数没限制，只要函数被调用就会创建一个新的上下文！**

其实执行上下文可以看作一个对象。该对象有三个必需属性: **变量对象、作用域链以及 this 对象：**

![执行上下文栈](/css/images/execution-context/context-object.png)

用对象表示为：

```
executionContextObj = {
    variableObject : { 
    /*函数的arguments对象, 参数, 内部的变量以及函数声明*/ 
    },
    scopeChain : { 
    /*variableObject以及所有父执行上下文中的variableObject*/ 
    },
    this : {}
}
```

在 js 引擎解释执行代码时，执行上下文的创建过程分为两个阶段：

**1.建立阶段（发生在函数调用时，但是在函数内具体代码执行以前）**：
> ① 建立 variableObject 对象（建立变量、函数、arguments 对象、函数参数）；
> ② 建立作用域链；
> ③ 确定 this。

**2.代码执行阶段**：
> ① 变量赋值；
> ② 执行代码。

下面具体地说明一下建立阶段和代码执行阶段细节问题：

**1.建立阶段**

确切地说，执行上下文对象是在函数被调用时，但是在函数体被真正执行以前所创建的。函数被调用的时候，在上述的两个阶段中的第一个阶段（建立阶段），js 引擎会检查函数中的参数，声明的变量以及内部函数，然后基于这些信息建立执行上下文对象。在这个阶段，variableObject 对象，作用域链，以及 this 所指向的对象都会被确定。

建立阶段又可细化为**建立 variableObject 对象**、**初始化作用域链**以及**确定 this 指向**等三个环节。

① 建立 variableObject 对象

1. 检查当前上下文中的参数，建立 arguments 对象，。

2. 检查当前上下文中的函数声明：每找到一个函数声明，就在 variableObject 下面用函数名建立一个属性，属性值就是指向该函数在内存中的地址的一个引用。如果上述函数名已经存在于 variableObject 下，那么对应的属性值会被新的引用所覆盖。

3. 检查当前上下文中的变量声明：每找到一个变量的声明，就在 variableObject 下，用变量名建立一个属性，属性值为 undefined。**如果该变量名已经存在于 variableObject 属性中，直接跳过，原属性值不会被修改。这么做也是为了防止指向函数的属性的值会被变量属性覆盖，所以，从这里可以看出函数声明相对于变量声明有较高的优先级。**

② 初始化作用域链

③ 确定 this 指向

**2.代码执行阶段**

一行一行地执行函数体内代码，给 variableObject 中的变量属性赋值。

**为了验证函数声明的优先级高于变量声明，我们验证如下：**

```
(function(){
    console.log('1. ',typeof f);
    function f(){
        console.log("I'm function f")
    }

    var f = 666;

    console.log('2. ',typeof f);
})();

// 1. function
// 2. number
```

我们将函数声明和变量声明换个位置，不出意外，执行结果应该不变。

```
(function(){
    console.log('1. ',typeof f);

    var f = 666;
    function f(){
        console.log("I'm function f")
    }

    console.log('2. ',typeof f);
})();

// 1. function
// 2. number
```

用 function 关键词声明的函数，在上下文建立阶段就确定了其属性值就是指向该函数在内存中的地址的一个引用，虽然在建立阶段该函数变量不会被普通变量覆盖，但是在代码执行阶段该变量属性值是可以被覆盖的。所以，以上代码段执行结束后，最后变量 f 都是数字 666。

下面再看个例子：

```
function foo(i) {
   var a = 'hello';
   var b = function(){};
   function c(){}
}

foo(22);
```
当执行到 foo(22) 时，函数上下文进入建立阶段（第一个阶段）：

```
fooExecutionContext = {
   variableObject: {
       arguments: {
           0: 22,
           length: 1
       },
       i: 22,
       c: pointer to function c()
       a: undefined,
       b: undefined
   },
   scopeChain: { ... },
   this: { ... }
}
```

我们看到，在建立阶段，除了 arguments，函数的声明，以及参数被赋予了具体的属性值，其它的变量属性默认的都是 undefined。一旦上述建立阶段结束，引擎就会进入代码执行阶段，这个阶段完成后，上述执行上下文对象如下:

```
fooExecutionContext = {
   variableObject: {
       arguments: {
           0: 22,
           length: 1
       },
       i: 22,
       c: pointer to function c()
       a: 'hello',
       b: pointer to function privateB()
   },
   scopeChain: { ... },
   this: { ... }
}
```

我们再回过头来看看上文提到的变量提升：**在函数中声明的变量以及函数，其作用域提升到函数顶部，换句话说，就是一进入函数体，就可以访问到其中声明的变量以及函数**。

```
(function() {

   console.log(typeof foo); // function
   console.log(typeof bar); // undefined

   var foo = 'hello',
       bar = function() {
           return 'world';
       };

   function foo() {
       return 'hello';
   }

}());
```

上面定义了一个自执行函数。函数执行时就有个上下文被创建，然后我们打印 foo 和 bar 的值并没有显示未定义错误。这里有两个问题：

** 1.为什么 foo 类型是 function，而不是 string 或者 undefined？**

因为在上下文的建立阶段，**先是处理 arguments, 参数，接着是函数的声明，最后是变量的声明**。那么，发现 foo 函数的声明后，就会在 variableObject 下面建立一个 foo 属性，其值是一个指向函数的引用。当处理变量声明的时候，发现有 var foo 的声明，但是 variableObject 已经具有了 foo 属性，所以直接跳过。当进入代码执行阶段的时候，就可以通过访问到 foo 属性了，因为它已经就存在，并且是一个函数引用。

** 2.为什么 bar 类型是 undefined？**

因为 bar 是变量的声明，在建立阶段的时候，被赋予的默认的值为 undefined。由于它只会在后续代码执行过程中才会被赋予具体的值，所以，当调用 typeof(bar) 的时候输出的值为 undefined。


参考：
[1] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
[2] http://blogread.cn/it/article/6178
[3] https://www.kancloud.cn/kancloud/javascript-prototype-closure/66352