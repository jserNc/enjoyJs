---
title: 理解 JavaScript 中 this
date: 2016-11-02 09:53:32
tags: js
---

**this 出现的地方无非两种：全局环境中和函数中。**不管哪种情况，this 都是执行上下文的一个属性，并且 this 总是返回一个对象。**（1）在全局上下文中，this 就是全局对象本身。（2）在函数上下文中，函数的每次调用对应 this 对象都可能不同。**不过，关于函数中的 this，我们可以记住一句结论：this 对象是在函数调用时，具体代码执行以前，激活该函数上下文的执行者（比如调用该函数的对象，再比如调用该函数的外层上下文对象）。

<!-- more -->

下面我们分别探讨这两种情况：

**1.全局环境中 this**

全局环境中的 this 就是 window 对象

```
this === window;    //true
//这里特指浏览器,只有浏览器中全局对象是window

var x = 10;
console.log(
    x,          // 10
    this.x,     // 10
    window.x    // 10
);
```

**2.函数中 this**

* js 语言有一个特点，就是允许“动态绑定”，即某些属性和方法到底属于哪一个对象不是在编译时确定的，而是在运行时确定的。

* this 是在函数调用的时候确定，指向的是函数此次调用的“所有者，owner”，例如，函数被哪个对象调用，this 就自动绑定到该对象。

* this 是在该函数上下文建立阶段确定的，每次调用该函数产生新的上下文的时候 this 值都不尽相同。一旦进入了函数内代码执行阶段，this 值就不能改变了。这时，想要给 this 赋一个新值是不可以的。

* 如果没有通过 new（包括对象字面量定义）、call、apply 和 bind 等改变函数的 this 对象，那么函数内的 this 都是指向 window。

**本质上讲，this 都是指向实例化对象。**貌似前面讲了很多情况 this 都指向了 window。这只是因为这时候仅做了一次实例化操作，即实例化 window 对象，于是 this 就是指向 window。如果我们要把 this 指向从 window 变成别的对象，就需要实例化，也就是执行 new 运算符。


```
function f(){
    return this;
}
```

如果函数 f 在全局环境中执行，f() 或者 window.f()，返回 window；如果 f 在对象上调用，如 obj.f()，返回的是 obj。

再次强调一下 this 的特殊性：**与变量不同，this 从不会参与到标识符的解析过程。**换句话说，在代码中访问 this 的时候，它的值是直接从执行上下文中获取的。this 的值在进入上下文的时候一次确定，并不需要在任何作用域链中查找！

举个例子：

```
var aa = "AAAAA";
var obj = {
    aa : 'objAAA',
    'pointer' : this,
    f : function(){
        return this;
    },
    oo : {
        'a' : this.aa,
        'pointer' : this,
        f : function(){
            return this;
        }
    }
};

obj.aa  // "objAAA"
obj.pointer === window  // true
obj.f() === obj // true

obj.oo.a    // "AAAAA"
obj.oo.pointer === window   // true
obj.oo.f() === obj.oo   // true
```
我们看到，对象中的 this 变量（非函数中）也是默认为 window。这里有个问题值得注意：如果全局的 aa 变量是在 obj 对象之后定义，以上打印结果会不一样。

```
var obj = {
    aa : 'objAAA',
    'pointer' : this,
    f : function(){
        return this;
    },
    oo : {
        'a' : this.aa,
        'b' : this.bb,
        'pointer' : this,
        f : function(){
            return this;
        }
    }
},
aa = "AAAAA";

obj.aa  // "objAAA"
obj.obj.pointer === window  // true
obj.f() === obj  // true

obj.oo.a    // undefined，这里和上面不一样

obj.oo.pointer.aa    // "AAAAA"
window.aa   // "AAAAA"

obj.oo.pointer === window   // true
obj.oo.f() === obj.oo   // true
```

obj.oo.a 打印出来竟然是 undefined，这是因为 window 对象的 aa 属性是在字面量 obj 之后定义的，window.aa 值修改已经不能影响 obj.oo 对象的 a 属性了。于是，我们推断，字面量变量定义的时候，其静态属性就已确定了。例如，字面量对象 o2 的属性 att2 指向另一个对象 o1 的属性 att1，后期再修改 o1 的属性 att1，o2 的 att2 属性不会跟着变。下面我们来验证一下这个问题：

```
var o1 = {
    att1 : 'aaa'
};

var o2 = {
    att2 : o1.att1
};

o2.att2     // "aaa"

o1.att1 = 'bbb'
o2.att2     // "aaa"
```

果然如此。**定义字面量变量的时候，其静态属性就已确定了。**这么规定是有道理的，如果字面量对象的属性还是一个变量，那就失去字面量的“本义”了。

下面我们看看 this 出现的另几个场景：

**（1）对象中拆出来的方法中的 this**

```
var car = {
    brand : 'bmw',
    getBrand : function(){
        console.log(this.brand);
    }
};

var getCarBrand = car.getBrand;

getCarBrand();      //undefined

window.getCarBrand();    //undefined

car.getBrand();     //bmw
```

我们看到，如果从对象中拆取出某个函数，那么这个方法就会变成一个普通的函数，它和原来对象之间的联系就不存在了。换句话说，一个拆取出来的函数就不再绑定到原来对象了。

**（2）函数内定义的函数中的 this**

```
var o = {
    f1 : function(){
        var f2 = function(){
            console.log(this === window);
        }
        f2();
    }
};

o.f1()  // true
```
还是上面那句话，如果没有通过 new（包括对象字面量定义）、call、apply 和 bind 等改变函数的 this 对象，那么函数内的 this 都是指向 window。

**（3）回调函数中的 this**

```
var el = document.getElementById("btn");
el.addEventListener("click",car.getBrand,false);
```

虽然使用的是 car.getBrand 方法，但是实际上是将 getBrand 方法关联到 el 对象。换句话讲：当我们在某个对象上执行一个方法，虽然这个方法之前是在其他对象里定义的，但是此时 this 已经不再指向原来的对象了，而是指向此时调用该方法的对象。

下面我们看个更详细的例子：

```
function hello(){
    console.log(this.id);
}

var id = "doc";
var para = document.getElementById("para");

para.addEventListener("click",hello,false);
//点击para之后，输出para
```

如此，hello 方法内的 this 指向了绑定的 para 对象。以上绑定方式类似于：

```
para.onclick = hello;
```

如果我们把监听函数部署在节点元素的 on- 属性上，监听函数内的 this 就不会再指向该节点元素了。

```
<p id='para' onclick='hello()'>Hello</p>
//点击p节点，输出doc
```

这种绑定方式相当于：

```
para.onclick = function(){
    hello();
}
```

所以，不难理解函数 hello 中 this 会指向 window 了。其实，想输出 para 也可以做到，稍做修改：

```
<p id="para" onclick="console.log(this.id)">Hello</p>
//点击p节点，输出para
```

这种方式，this 又指向 para 节点了。相当于：

```
para.onclick = function(){
    console.log(this.id);
}
```

总结一下，以下几种写法中 doSomething 中 this 均指向 window。

```
//写法一
element.onclick = function(){
    doSomething();    
}

//写法二
element.setAttribute('onclick','doSomething()')

//写法三
<element onclick="doSomething()">
```

** 固定 this 指向： **

函数中 this 指向的多变，在某些场景下可能使问题复杂化，不过，这种灵活语法不正是 JavaScript 的趣味性所在吗？

有时候我们想明确地知名函数执行时其内部 this 的指向。为此，JavaScript 提供了 call、apply、bind 等 3 个方法来切换/固定 this 的指向。这 3 个方法均是 Function 构造函数的原型方法，即定义于 Function.prototype 对象上。

**(1) Function.prototype.call**

将函数内部 this 指向 call 方法的第一个实参（一般为对象。如果不是对象的原始类型数据则强制转换成对应的包装对象），call 方法的其余参数作为函数的参数传入。

```
var o = {};
var f = function(a,b){
    console.log('a:',a,'b:',b);
    return this;
}
f() === window   
// a: undefined b: undefined
// true

f.call(o,1,2) === o;
// a: 1 b: 2
// true
```

函数 f 内部的 this 指向 call 方法第一个实参 o，其余参数（ 1 , 2 ）分别作为函数 f 执行时的实参。

**(2) Function.prototype.apply**

apply 方法和 call 方法作用和用法几乎一样，都是改变函数内部的 this 指向，然后执行该函数。唯一的区别就是接收的参数格式不一样。

```
f.call(thisValue,arg1,arg2,arg3...);
// 各个参数以逗号 , 分隔
f.apply(thisValue,[arg1,arg2,arg3...]);
// 第二个参数为数组，包含函数执行时的参数。
```

JavaScript 没有提供获取数组中最大值的原生 api，我们可以利用 apply 方法来获取数组的最大值。

```
Math.max(x...)
// 可返回多个指定的数中最大的那个数
// 在 ECMASCript v3 之前，该方法只有两个参数

var a = [10,2,3,45,5,66,20];
Math.max.apply(null,a);
// 66
```

如果 call / apply 方法的第一个参数为空、null 或者 undefined，则将函数内部 this 指向全局对象 window。

**(3) Function.prototype.bind**

将函数内部的 this 绑定到 bind 方法指定的第一个参数，然后返回一个新的函数。

```
var d = new Date();
d.getTime()
// 1490857265085

var print = d.getTime;
print();
// Uncaught TypeError: this is not a Date object.
```

全局函数 print 执行时，其内部的 this 指向全局函数 window（不是 Date 的实例对象），导致函数执行出错。可以使用 bind 方法，将 print 函数内部 this 指向 Date 实例对象 d。

```
var print = d.getTime.bind(d);
print()
// 1490857265085
```

对于不支持 bind 方法的 ie8 以下浏览器，自行定义 bind 方法：

```
if(!('bind' in Function.prototype)){
  Function.prototype.bind = function(){
    var fn = this;
    var context = arguments[0];
    var args = Array.prototype.slice.call(arguments,1);
    return function(){
      return fn.apply(context,args);
    }
  }
}
```


参考：
[1] http://javascript.ruanyifeng.com/oop/this.html
[2] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
