---
title: JavaScript中this该如何理解？
date: 2016-11-02 09:53:32
tags:
---

this出现的地方无非两种：全局环境中和函数中。不管哪种情况，this都是执行上下文的一个属性，并且this总是返回一个对象。（1）在全局上下文中，this就是全局对象本身。（2）在函数上下文中，函数的每次调用对应this对象都可能不同。不过，关于函数中的this，我们可以记住一句结论：this对象是在函数调用时，具体代码执行以前，激活该函数上下文的执行者（比如调用该函数的对象，再比如调用该函数的外层上下文）。

<!-- more -->

1.全局环境中this

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

2.函数中this

* js语言有一个特点，就是允许“动态绑定”，即某些属性和方法到底属于哪一个对象不是在编译时确定的，而是在运行时确定的。

* this是在函数调用的时候确定，指向的是函数此次调用的“所有者，owner”，例如，函数被哪个对象调用，this就自动绑定到该对象。

* this是在该函数上下文建立阶段确定的，每次调用该函数产生新的上下文的时候this值都不尽相同。一旦进入了函数内代码执行阶段，this值就不能改变了。这时，想要给this赋一个新值是不可以的。

* 如果没有通过new（包括对象字面量定义）、call、apply和bind等改变函数的this对象，那么函数内的this都是指向window。

本质上讲，this都是指向实例化对象。貌似前面讲了很多情况this都指向了window。这只是因为这时候仅做了一次实例化操作，即实例化window对象，于是this就是指向window。如果我们要把this指向从window变成别的对象，就需要实例化，也就是执行new运算符。


```
function f(){
    return this;
}
```

如果函数f在全局环境中执行，f()或者window.f()，返回window；如果f在对象上调用，如obj.f()，返回的是obj。

再次强调一下this的特殊性：与变量不同，this从不会参与到标识符的解析过程。换句话说，在代码中访问this的时候，它的值是直接从执行上下文中获取的，this的值在进入上下文的时候一次确定！并不需要在任何作用域链中查找。

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
我们看到，对象中的this变量（非函数中）也是默认为window。这里有个问题值得注意：如果全局的aa变量是在obj对象之后定义，以上打印结果会不一样。

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

obj.oo.a打印出来竟然undefined，只是因为window的aa属性是在字面量obj之后定义，window.aa值修改已经不能影响obj.oo对象的a属性了。于是，我们推断，字面量变量定义的时候，其静态属性就已确定了。例如，字面量对象o2的属性att2指向另一个对象o1的属性att1，后期再修改o1的属性att1，o2的att2属性不会跟着变。下面我们来验证一下这个问题：

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

果然如此。字面量变量定义的时候，其静态属性就已确定了。

下面我们看看this出现的另几个场景：

（1）对象中拆出来的方法中的this

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

（2）函数内定义的函数中的this

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
还是上面那句话，如果没有通过new（包括对象字面量定义）、call、apply和bind等改变函数的this对象，那么函数内的this都是指向window。

（3）回调函数中的this

```
var el = document.getElementById("btn");
el.addEventListener("click",car.getBrand,false);
```

虽然使用的是car.getBrand方法，但是实际上是将getBrand方法关联到el对象。换句话讲：当我们在某个对象上执行一个方法，虽然这个方法之前是在其他对象里定义的，但是此时this已经不再指向原来的对象了，而是指向此时调用该方法的对象。

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

如此，hello方法内的this指向了绑定的para对象。以上绑定方式类似于：

```
para.onclick = hello;
```

如果我们把监听函数部署在节点元素的on-属性上，监听函数内的this就不会再指向该节点元素了。

```
<p id="para" onclick="hello()">Hello</p>
//点击p节点，输出doc
```

这种绑定方式相当于：

```
para.onclick = function(){
    hello();
}
```

所以，不难理解hello中this会指向window了。其实，想输出para也可以做到，稍做修改：

```
<p id="para" onclick="console.log(this.id)">Hello</p>
//点击p节点，输出para
```

这种方式，this又指向para节点了。相当于：

```
para.onclick = function(){
    console.log(this.id);
}
```

总结一下，以下几种写法中doSomething中this均指向window。

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


参考：
[1] http://javascript.ruanyifeng.com/oop/this.html
[2] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/
