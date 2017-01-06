---
title: JavaScript原型链
date: 2016-10-24 15:00:44
tags: js
---
js中，除了null，每个对象都会继承另一个对象，后者称为“原型对象”。**原型对象的所有属性和方法都可以被派生对象共享**，这是js的继承机制。通过构造函数生成实例对象的时候，会自动给实例对象分配原型对象。**实例对象生成时候的构造函数的prototype属性**即为实例对象的原型对象。（在此我们需要注意原型对象是实例对象生成时的构造函数的prototype属性，因为构造函数的prototype属性后来是可以更改的，而先前生成好的实例的原型是不会跟着变的。）

<!-- more -->

换个角度讲：由于所有的js对象都有构造函数，而所有的构造函数都有prototype属性（其实是所有函数都有该属性），所以，我们可以说所有的对象都有自己的原型对象(该对象的构造的prototype属性)。而原型对象也是对象，其也有自己的原型对象，于是，层层上溯就形成了一条“原型链”。

"原型链"的作用是：读取对象的某个属性的时候，js引擎会先遍历对象本身的属性，如果找不到，就去对象的原型对象去找，如果还是没有，就去对象的原型对象的原型对象去找，直到顶层的Object.prototype，如果还是找不到，那么就会返回undefined（所以，读取对象的某个属性时，哪怕它不存在也不会报错）。

一个对象就是一个属性集合，并拥有独立的原型。这个原型可以是一个对象或者null。

我们看一个关于对象的基本例子。一个对象的prototype是以内部的\[\[Prototype\]\]属性来引用的。但是，在示意图里边我们将会使用\_\_\_\_下划线标记来替代两个括号，对于原型对象来说是：\_\_proto\_\_。

对于以下对象：

```
var foo = {
    x : 10,
    y : 20
};
```

该对象拥有一个这样的结构，两个明显的自身属性和一个隐含的\_\_proto\_\_属性，这个属性是对foo原型属性（prototype）的引用：

![foo对象结构](/css/images/prototype/basic-object.png)

即原型可以用\_\_proto\_\_直观的表示,再看一个简单的例子：（当然了，标准做法是，用Object.getPrototypeof()（读取原型）和Object.setPrototypeOf()（设置原型），来读写对象的原型对象）

```
var A = {
  name: '张三'
};
var B = {
  name: '李四'
};

var proto = {
  print: function () {
    console.log(this.name);
  }
};

A.__proto__ = proto;
B.__proto__ = proto;

A.print() // 张三
B.print() // 李四
```

换一种写法，加深一下理解：

```
var a = {
    x : 10,
    calculate : function(z){
        return this.x + this.y + z;
    }
};

var b = {
    y : 20,
    __proto__ : a
};

var c = {
    y : 30,
    __proto__ : a
};

b.calculate(30);    //60

c.calculate(40);    //80
```
如果没有明确为一个对象指定原型，那么它将使用\_\_proto\_\_的默认值——Object.prototype，Object.prototype对象也有原型，值为null,这是原型链的终点。

我们也可以用ES5的Object.create()方式实现上面的对象关系：

```
var b = Object.create(a,{y : {value : 20}});
var c = Object.create(a,{y : {value : 30}});
```

我们再用构造函数重写上面的例子：

```
function Foo(y){
    this.y = y;
}

Foo.prototype.x = 10;
Foo.prototype.calculate = function(z){
    return this.x + this.y + z;
}

var b = new Foo(20);
var c = new Foo(30);

b.calculate(30);    //60
c.calculate(40);    //80
```
各对象之间的继承关系如下：
![对象继承关系](/css/images/prototype/constructor-proto-chain.png)

原型先说到这里，再看看instanceof运算符：

```
var v = new Vehicle();
v instanceof Vehicle	//true
```

以上结果并不意外，我们看到instanceof 运算符左边是实例对象，右边是构造函数。**其运算实质是：检查右边构造函数的prototype属性是否在左边的实例对象的原型链上。**上例即判断Vehicle.prototype是否在对象v的原型链上。

所以，对以下结果我们应该也不意外：

```
var arr = [];
arr instanceof Array;		//true
arr instanceof Object;		//true

arr.__proto__ === Array.prototype //true
Array.prototype.__proto__ === Object.prototype //true
```
所有对象的原型链的顶层就是 Object.prototype，所以，任何对象对Object进行instanceof运算都返回true

再看：

```
function A(){};
var a = new A();
a instanceof A;   //true

function B(){};
A.prototype = B.prototype;
a instanceof A;   //false why?
```
A.prototype改变后，再进行instanceof运算返回了false。我们怎么确定A.prototype不在a的原型链上？

回顾一下以上过程，定义A方法时，A的prototype属性就已确定，这时候新建一个实例对象a，那么a的原型对象（a.\_\_proto\_\_）就是此时的A.prototype，然后使得A的prototype属性指向B的prototype属性，而a的原型对象还是原来的A.prototype。

再执行一遍以上代码：

```
function A(){};
var a = new A();
a instanceof A;   //true

var protoA = A.prototype;

function B(){};
A.prototype = B.prototype;
a instanceof A;     //false why?

a.__proto__ === protoA;         //true
a.__proto__ === A.prototype;    //false
```

事实却是如此，a的原型对象依然是原来的protoA 。所以我们可以认为对象a的原型对象在对象生a成的时候就已经确定了，并不会随着构造函数的prototype属性改变自动改变。**总结一点：对象的原型对象就是该对象生成时刻其构造函数对应的prototype属性。**

所以，对下面的执行结果也很好理解了：

```
a.constructor === A 	 //true
A.prototype.constructor === B 	  //true
```

那么，这时候我们再新建一个A的实例会怎样？

```
var aa = new A();
aa.__proto__ === A.prototype    //true

aa.constructor === A;   //false，为什么？

//因为
aa.constructor === A.prototype.constructor;     //true
A.prototype.constructor === B.prototype.constructor;
//true
B.prototype.constructor === B;      //true

//所以
aa.constructor === B;   //true

aa instanceof A     //true
aa instanceof B     //true
```

我们发现，虽然a是A的实例，但修改了A.prototype以后，A的constructor属性的指向就跟着变了。

所以，修改构造函数的prototype属性时，一般也同时将新的prototype属性的constructor属性指向原来的构造函数。

```
function A(){};
var a = new A();
a instanceof A;		//true

function B(){};
A.prototype = B.prototype;
A.prototype.constructor = A;

a instanceof A;		//false,返回false不应该意外

aa = new A();
aa.constructor === A; 	
//true,是因为中间把constructor重新指向了A
```

如果让某个函数的prototype属性指向一个数组，那么该函数就可以当做数组的构造函数。

```
var MyArray = function () {};

MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;

var mine = new MyArray();

mine.push(1, 2, 3);
mine.length     // 3

mine instanceof Array   //true why?

MyArray.prototype.__proto__ === Array.prototype //true
```

可以看到，构造函数的prototype的方法都可以被实例对象调用。并再次验证了：只要instanceof运算符右边的运算子的prototype属性在左边运算子的原型链上，即返回true！

当然了，对象的原型对象是可以自行设置的

```
var a = {x : 1};

var b = {__proto__ : a};
b.x     //1

Object.getPrototypeOf(b) === a  //true
```
以上，把b对象的__proto__属性指向a对象，a对象便成了b对象的原型对象。其中，Object.getPrototypeOf()方法是获取一个对象的原型对象的标准方法！

也可以换种写法：

```
var a = {x : 1};

var b = Object.setPrototypeOf({},a);
b.x     //1

Object.getPrototypeOf(b) === a  //true
```
其中，Object.getPrototypeOf()方法为某对象（即第一个参数）设置原型对象（第二个参数），返回值为一个新对象。即将对象{}的原型对象设置为a,然后变量b指向返回的新对象。

以上多次用到new运算符，我们再来看看new：

new命令调用后会依次执行以下步骤：

1. 创建一个空对象，作为将要返回的对象实例；
2. 将这个空对象的原型对象（\_\_proto\_\_）指向构造函数的prototype属性；
3. 将构造函数内部的this指向该空对象；
4. 执行构造函数代码。

例如：

```
var F = function(){
    this.foo = 'bar';
};

var f = new F();
```

会发生：

1. 一个新的空对象{}会被创建，并且该对象的原型对象为F.prototype；
2. 构造函数F内的上下文（this）指定为这个新对象，然后执行构造函数；
3. 如果构造函数中明确地用return返回了一个其他对象，那么这个对象会作为整个new运算的返回结果。否则，返回刚才创建的那个对象。

总结一下：new命令通过构造函数创建新的实例对象，实质就是：**将实例对象的原型指向构造函数的prototype的属性，然后在实例对象上执行构造函数（实例对象为构造函数的执行上下文）**，所以，以上的new命名可以换种写法：

```
var f = Object.setPrototypeOf({}, F.prototype);
F.call(f);
f.foo   //"bar"
```


参考：
[1] http://javascript.ruanyifeng.com/oop/prototype.html
[2] http://dmitrysoshnikov.com/ecmascript/javascript-the-core/