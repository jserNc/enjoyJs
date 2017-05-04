---
title: JavaScript 单例模式
date: 2017-05-04 10:14:45
tags: grammar
---

单例模式保证一个特定类只有一个实例对象。也就是说多次用同一个类来创建对象，得到的都是同样一个对象。严格意义上，JavaScript 没有类，只有对象，对象之间永远不会完全相等，除非它们是同一个对象，所以我们创建的每一个对象都可以看做是单例。比如，我们可以对象字面量创建单例对象。


<!-- more -->

```
var o1 = {name : 'singleton'};
var o2 = {name : 'singleton'};

o1 === o2;
// false;
o1 == o2;
// false;
```

我们看到，即便是具有相同成员的同类对象，它们也是彼此独立的个体。

除此之外，我们还可以用 new 运算符来新建实例对象，再探讨用这种方式创建单例对象之前，先复习一下 new 运算符：

** new 命令调用后会依次执行以下步骤：**

> 1.创建一个空对象，作为将要返回的对象实例；
> 2.将这个空对象的原型对象（\_\_proto\_\_）指向构造函数的 prototype 属性；
> 3.将构造函数内部的 this 指向该空对象；
> 4.执行构造函数代码。

例如：

```
var F = function(){
    this.foo = 'bar';
};

var f = new F();
```

会发生：

> 1.一个新的空对象 {} 会被创建，并且该对象的原型对象（\_\_proto\_\_）为 F.prototype；
> 2.构造函数 F 内的 this 指向这个新对象，然后执行构造函数；
> 3.如果构造函数中明确地用 return（返回）了一个其他对象，那么这个对象会作为整个 new 运算的返回结果。否则，返回刚才创建的那个对象。

总结一下：new 命令通过构造函数创建新的实例对象，实质就是：**将实例对象的原型指向构造函数的 prototype 的属性，然后在实例对象上执行构造函数（构造函数内 this 绑定为该实例对象）**，所以，以上的 new 命名可以换种写法：

```
var f = Object.setPrototypeOf({}, F.prototype);
F.call(f);
f.foo   //"bar"
```

首先看一个构造函数静态属性缓存单例的示例：

```
function Singleton(){
    if (typeof Singleton.instance === "object"){
        return Singleton.instance;
    }

    this.name = 'enjoyJs';
    this.age = 18;

    Singleton.instance = this;

    return this;
}

var o1 = new Singleton();
var o2 = new Singleton();

o1 === o2
// true
```

第一次执行 new 运算时，完整地执行该构造函数，然后返回一个对象 Singleton.instance，以后再执行 new 运算，都会直接返回对象 Singleton.instance。

这种方式直观易懂，但是 instance 属性是可以被访问修改的，下面用闭包将该属性私有化：

```
function Singleton(){
    var instance = this;

    this.name = 'enjoyJs';
    this.age = 18;

    Singleton = function(){
        return instance;
    }
}

Singleton.prototype.attr1 = 'attr1';
var o1 = new Singleton();

Singleton.prototype.attr2 = 'attr2';
var o2 = new Singleton();

o1.attr1
// attr1
o2.attr1
// attr1

o1.attr2
// undefined
o2.attr2
// undefined
```

为什么这里的 attr2 不被实例对象继承呢？

第一次用 new 运算符执行该构造函数之前，加到 Singleton.prototype 对象上的属性是会被实例对象继承的，这理所当然。这次执行会导致 Singleton 被重写，指向了新的函数。所以，第二次向 Singleton.prototype 上添加属性是不会被实例对象继承的，因为新的 Singleton 函数会返回第一次执行时返回的那个实例对象。

```
o1.constructor === Singleton
// false
```

o1.constructor 指向的是重定义之前的 Singleton 函数，所以这里结果为 false。

下面对以上的函数稍作修改：

```
function Singleton(){
    var instance;

    Singleton = function(){
        return instance;
    }

    Singleton.prototype = this;

    instance = new Singleton();

    instance.constructor = Singleton;

    instance.name = 'enjoyJs';
    instance.age = 18;

    return instance;
}

Singleton.prototype.attr1 = 'attr1';
var o1 = new Singleton();

Singleton.prototype.attr2 = 'attr2';
var o2 = new Singleton();

o1.attr1
// "attr1"
o2.attr1
// "attr1"

o2.attr2
// "attr2"
o1.attr2
// "attr2"

o1.constructor === Singleton
// true
```

得到了和上面截然不同的结果。仔细分析一下：

```
Singleton.prototype.attr1 = 'attr1';
var o1 = new Singleton();
```

这两句代码的执行过程是：
【1】创建一个新的空对象 o, o 对象的原型对象（\_\_proto\_\_）为 Singleton.prototype （该对象包含这里的属性 attr1），然后将 Singleton 函数内的 this 指向对象 o；
【2】接着执行构造函数 Singleton；
【3】声明一个局部变量 instance，初始值为 undefined；
【4】将 Singleton 变量指向新的函数，新函数返回值为刚才的局部变量 instance；
【5】将新的 Singleton 函数的 prototype 指向 this，即对象 o；
【6】实例化新的构造函数 Singleton，即：创建一个新的对象 oNew，oNew 对象的原型对象为对象 o，因为此时的返回值 instance 值为 undefined，不是对象，所以，返回值为对象 oNew；
【7】将 instance 指向对象 oNew；
【8】为 instance 对象添加各种属性；
【9】 返回对象 instance，即变量 o1 指向对象 instance。


```
Singleton.prototype.attr2 = 'attr2';
var o2 = new Singleton();
```

这两句代码的执行过程是：
【1】上面提到新的 Singleton 的 prototype 指向对象 o，这里修改 Singleton.prototype 即是修改对象 o，对象 o 的所有属性都会被其实例对象继承，包括之前创建的实例对象 oNew，亦 instance；
【2】实例化新的构造函数 Singleton，创建一个新的空对象 {}，不过，由于这次函数返回值 instance 不再是 undefined 了，而是一个对象，所以，返回会返回 instance，而不是空对象 {}，即变量 o2 指向对象 instance。

换一种写法：

```
var Singleton;
(function(){
    var instance;
    Singleton = function Singleton(){
        if (instance){
            return instance;
        }
        instance = this;
        
        this.name = 'enjoyJs';
        this.age = 18;
    };
}());

Singleton.prototype.attr1 = 'attr1';
var o1 = new Singleton();

Singleton.prototype.attr2 = 'attr2';
var o2 = new Singleton();

o1.attr1
// "attr1"
o2.attr1
// "attr1"

o2.attr2
// "attr2"
o1.attr2
// "attr2"

o1.constructor === Singleton
// true
```

下面分析一下执行过程：

```
Singleton.prototype.attr1 = 'attr1';
var o1 = new Singleton();
```

【1】新建全局变量 Singleton，执行匿名函数，将 Singleton 指向一个闭包函数；
【2】为构造函数 Singleton 的 prototype 对象添加新的属性 attr1；
【3】实例化构造函数 Singleton，创建一个空对象 o，此时，变量 instance 值为 undefined，不返回，继续往下走；
【4】this 指向对象 o，instance 指向 对象 o；
【5】给对象 o 添加属性，返回对象 o，即变量 o1 指向对象 o。

```
Singleton.prototype.attr2 = 'attr2';
var o2 = new Singleton();
```

【1】全局变量 Singleton，指向函数后，其指向一直不变，所以，prototype 对象的修改，都可以反映在实例对象上；
【2】实例化构造函数 Singleton，创建一个新的空对象 {}，不过，由于这次函数返回值 instance 不再是 undefined 了，而是一个对象，所以，返回会返回 instance，而不是空对象 {}，即变量 o2 指向对象 instance。




