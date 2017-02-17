---
title: toString.call()引起的一些思考
date: 2016-10-14 17:20:13
tags: grammar
---

toString作为全局方法时，挂载于window对象。作为最顶层的原型对象 Object.prototype 有着自己的 Object.prototype.toString 方法，另外，Array、String、Boolean、Function、Date等构造函数也也都部署了自己的toString方法。

为了方便下文的理解，再强调一下原型链的作用：**读取对象的某个属性时，JavaScript引擎先寻找该对象本身的属性，如果找不到，就到它的原型去找，如果还是找不到，就到原型的原型去找，直到顶层原型对象 Object.prototype。如果一层一层到最顶层的还是找不到，则返回undefined。**

<!-- more -->

首先，所有全局方法均挂载于window对象，所有对象都继承自Object对象。
```
toString === window.toString  //true
window instanceof Object      //true
```

window是个对象，不是构造函数，其构造函数是Window
```
window.prototype              //undefined
```

```
window.constructor === Window         //true
window.constructor === window.Window  //true
window.__proto__ === Window.prototype //true
```

对象的静态方法和原型方法不同，静态方法可以继承自对象的原型也可以自行定义(优先级更高)
```
Object.prototype.toString === Object.toString //false
({}).toString === Object.prototype.toString   //true
```

Object的原型部署了toString方法，在其原型对象上调用该方法，始终返回"[object Object]"。换句话讲，Object.prototype.toString方法本不需要传参，传参也被忽视。字符串[object Object]本身没有太大的用处，但是其继承对象可以自定义该方法
```
Object.toString()                
//"function Object() { [native code] }"
```

```
Object.prototype.toString()      //"[object Object]"
Object.prototype.toString([])    //"[object Object]"
Object.prototype.toString(true)  //"[object Object]"
```

window.toString()也不需要传参，传参会被忽视
```
window.toString(1)      //"[object Window]"
toString(1)             //"[object Window]"
window.toString([])     //"[object Window]"
toString([])            //"[object Window]"
```

然而，window.toString和Object.prototype.toString并不相等
```
window.toString === Object.prototype.toString   //false
Array.toString === Array.prototype.toString     //false
Array.toString === Object.prototype.toString    //false
```

但是,window.toString和Object.prototype.toString通过call调用的时候能得到一样结果：
```
Object.prototype.toString.call([])    //"[object Array]"
window.toString.call([])              //"[object Array]"
Object.prototype.toString.call(1)     //"[object Number]"
window.toString.call(1)               //"[object Number]"
```

再来看看数组原型方法Array.prototype.toString
```
[1,2,3].toString === Array.prototype.toString  //true
[1,2,3].toString()                             //"1,2,3"
Array.prototype.toString([1,2,3])              //""
```

事实上，和toString类似，直接给Array.prototype.toString传参返回值都是""
```
Array.prototype.toString()    //""
Array.prototype.toString(1)   //""
Array.prototype.toString([])  //""
```

通过call调用，就完全不一样了，数组参数会格式化为字符串：
```
Array.prototype.toString.call([1,2,3])  //"1,2,3"
```

非数组参数返回结果同window.toString.call()
```
Array.prototype.toString.call(1)    //"[object Number]"
Array.prototype.toString.call({})   //"[object Object]"
```

再看布尔型构造函数原型方法Boolean.prototype.toString
```
true.toString === Boolean.prototype.toString   //true
true.toString()                                //"true"
false.toString()                               //"false"
Boolean.prototype.toString(true)               //"false"
Boolean.prototype.toString(false)              //"false"
```

同样，直接给Boolean.prototype.toString传参返回值都是"false"
```
Boolean.prototype.toString([])   //"false"
Boolean.prototype.toString(1)    //"false"
Boolean.prototype.toString({})   //"false"
```

通过call调用,布尔型参数会格式化为字符串
```
Boolean.prototype.toString.call(true)   //"true"
Boolean.prototype.toString.call(false)  //"false"
```

非布尔型参数会怎样？报错！！
```
Boolean.prototype.toString.call(1)   
//TypeError: Boolean.prototype.toString is not generic
Boolean.prototype.toString.call({})   
//TypeError: Boolean.prototype.toString is not generic
```

```
[].constructor === Array         //true
[].__proto__ === Array.prototype //true
Array.prototype                  //[]
[].__proto__ === []              //false
```

Array的原型对象不等于其构造函数的原型对象!
```
Array.prototype === Array.__proto__    //false
```

Funtion的原型对象等于其构造函数的原型对象!!
```
Function.prototype === Function.__proto__
//true
```
```
Function.prototype === Function.constructor.prototype
//true
```
```
Function.prototype.__proto__ === Function.__proto__.__proto__
//true
```

Function构造函数的原型是Object的实例,但是其原型的原型的构造函数就是Object!但是，Function的原型的原型却不是Object的实例！！
```
Function.__proto__
//function Empty() {}
Function.__proto__  instanceof Object
//true
Function.__proto__.__proto__
//Object {}
Function.__proto__.__proto__.constructor === Object
//true
Function.__proto__.__proto__  instanceof Object
//false 为什么？？
Object instanceof Object
//true
Function.__proto__.__proto__ === Object
//false
Function.__proto__.constructor === Function
//true
Function.prototype
//function Empty() {}
Function.prototype.__proto__ === Object.prototype
//true
Function.__proto__.__proto__ === Function.__proto__.constructor.prototype
//false 为什么？？
Function.__proto__.__proto__  ===  Function.__proto__.constructor.prototype
//false 为什么？？
```

一般情况下，p.\_\_proto\_\_ === p.constructor.prototype是恒等的
```
function Person(name) {
    this.name = name
}
var p = new Person('jack');
p.constructor === Person                  //true
p.__proto__ === p.constructor.prototype   //true
```

但是，总有例外，毕竟对象的属性是可以修改的
```
function Person(name) {
    this.name = name
}
```

重写原型
```
Person.prototype = {
    getName: function() {}
}
var p = new Person('jack');
p.__proto__ === Person.prototype         //true
p.__proto__ === p.constructor.prototype  //false 为什么？？
```

其实也很好理解,p.constructor不再指向Person
```
p.constructor === Person                        //false
p.constructor === Person.prototype.constructor  //true
```
prototype被重写后，constructor相应就变了，再看：
```
var o = new Object(),o1 = new Object(); 
o.__proto__  === o.constructor.prototype   //true
o.__proto__ = o1;
o.__proto__  === o.constructor.prototype   //false
```


目前JS内置的构造函数有12个，global环境下可以访问的有8个
所有的构造器都来自于Function.prototype
即所有构造器都继承了Function.prototype的属性及方法，也包括Function自己和Object
```
Number.__proto__ === Function.prototype  // true
Boolean.__proto__ === Function.prototype // true
String.__proto__ === Function.prototype  // true
Function.__proto__ === Function.prototype // true
Array.__proto__ === Function.prototype   // true
```
```
Number.__proto__ === Object.__proto__    //true
Number.__proto__ === Function.__proto__  //true
Number.__proto__ === Array.__proto__     //true
Number.__proto__ === String.__proto__    //true
Number.__proto__                         
//function Empty() {}
Empty                                    
//ReferenceError: Empty is not defined
```

也就是说Empty不是全局构造方法，但是，Number、Function、Array等是全局构造方法
```
Number === window.Number   //true  
```

当然了，“所有的”构造函数也包括自定义的构造函数
```
var MyArray = function () {};
MyArray instanceof Function               //true
MyArray.__proto__ === Function.prototype  //true
MyArray.__proto__ === Function.__proto__  //true
```
```
Function.constructor  
//function Function() { [native code] }
Function.constructor === Function                      
//true
Function.constructor.prototype === Function.__proto__  
//true
```

函数也是一等公民
```
Function.prototype.__proto__ === Object.prototype
```
说明了所有的构造器也是一个普通的对象，继承了Object.prototype所有的方法

可以先new一个对象，然后修改该对象的prototype属性,eg:
```
var MyArray = function () {};
MyArray.prototype                        //MyArray {}
MyArray.__proto__    
//function Empty() {}
MyArray.__proto__  === Object.__proto__  //true
MyArray.prototype = new Array();
MyArray.prototype.constructor = MyArray;
```

instanceof运算符的实质，它依次与实例对象的所有构造函数的constructor属性进行比较
只要有一个符合就返回true，否则返回false
```
var mine = new MyArray();
```

于是：
```
mine instanceof Array 
```

相当于：
```
(Array === MyArray.prototype.constructor) ||
(Array === Array.prototype.constructor) ||
(Array === Object.prototype.constructor )
```
```
Object.prototype                         //Object {}
Object.prototype === Object.__proto__    //false
Object.prototype.__proto__               //null
```

再回到toString(),以下都很好理解了
```
toString([])             //"[object Window]"
toString.call(window,[]) //"[object Window]"
window.toString([])      //"[object Window]"
window.toString()        //"[object Window]"
```
```
toString.call([])        //"[object Array]"
[].toString()            //""
```
```
toString.call({}) //"[object Object]"
({}).toString()   //"[object Object]"
```
```
toString.call(true) //"[object Boolean]"
true.toString()     //"true"
```
```
toString.call(function(){}) //"[object Function]"
(function(){}).toString()   //"function (){}"
```
```
toString.call(1) //"[object Number]"
(1).toString()   //"1"
```
```
toString.call(null)//"[object Window]"
null.toString()
//TypeError: Cannot read property 'toString' of null
```
```
toString.call(undefined) //"[object Window]"
undefined.toString()     
//TypeError: Cannot read property 'toString' of undefined
```
