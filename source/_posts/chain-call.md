---
title: 浅谈JavaScript链式调用
date: 2016-10-25 16:34:44
tags: js
---

函数链式调用jQuery等框架中经常出现的一种函数调用方式，这种方式在面向对象编程中用得比较多。JavaScript的灵活性，使得实现链式调用的方式有多种，下面，我们简单地实现一下链式调用，以展示其原理。

<!-- more -->

先看一段简单的代码：

```
var n = 1;

function f(){
    n++;
    return f;
}

f();
console.log(n);     //2

f()()();
console.log(n);     //5
```

这里的函数f返回了它自身，无论其怎么执行返回的都是自己，所以，调用函数f时其后可以跟任意多个括号。

函数返回自身还有个特殊的地方：

```
new f === f     //true
new f() === f   //true
```

我们看到，对f进行new操作来新建实例对象也“失效”了，这是因为f函数返回的是f自身（一个对象）。当构造函数不带实参时，new操作可以不带括号，在这里以上两种写法等效。

再来看一个基于对象实现链式调用的例子：

```
var n = 1;
var o = {
    add : function(){
        n++;
        return o;
    },
    sub : function(){
        n--;
        return o;
    }
};

o.add().add().sub();
console.log(n);     //2
```

我们看到到以上对象的每个方法都返回该对象自身，所以方法的调用结果又可以作为下一个方法的“驱动对象”，这样就可以实现多个方法链式调用的效果。只是，实际开发中我们一般不会这么写，下面我们看个更专业的例子：

```
window.$ = function(id){
    return new _$(id);
};

function _$(id){
    this.ele = document.getElementById(id)
}

_$.prototype = {
    constructor : _$,
    hide : function(){
        console.log('hide');
        return this;
    },
    show : function(){
        console.log('show');
        return this;
    }
}

$('id').show().hide();
```

$方法执行后，会得到_$方法的实例对象，实例对象的每个方法执行后都返回该实例对象，于是，实现了链式调用。