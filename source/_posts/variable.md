---
title: JavaScript 变量
date: 2017-04-07 16:17:17
tags: js
---

JavaScript 变量分为两种：基本类型和引用类型。6 种数据类型里，除了对象是引用类型变量，其他的都是基本类型变量。引用类型的变量由多个值构成。当然了，每种基本类型（null 和 undefined 除外）都有与其对应的引用类型，在必要时候，基本类型变量也会自动转换成对应引用类型变量，然后再继续运算。

<!-- more -->

**基本类型数据不能为之添加属性，引用类型则可以。**

```
// 基本类型属性不能添加属性，即便添加了也无效
var site = 'Enjoy JavaScript';
console.log(site.author);
// undefined

site.author = 'nanc';
console.log(site.author);
// undefined

// 引用类型变量可以添加属性
// 引用类型数据包括 function、array、object
var f = function(){}
console.log(f.author);
// undefined

f.author = 'nanc';
console.log(f.author);
// nanc
```

理解基本类型变量和引用类型变量区别，关键一点是它们的在内存中存储方式的不同。

**JavaScript 变量的存储包含三部分：**
> 部分一：栈区变量的标识符
> 部分二：栈区变量的值
> 部分三：堆区存储的对象

我们定义变量如下：

```
// 基本类型：
var str = "Enjoy javascript";
var num = 666;
var isOpen = true;

// 引用变量：
var obj1 = {};
var obj2 = obj1;
var obj3 = {};
```

**基本类型变量只存放在内存的栈区（栈内存）：**

![作用域链](/css/images/variable/basic-type.png)

**引用类型变量由栈区和堆区（堆内存）配合存放：**

![作用域链](/css/images/variable/reference-type.png)

一般情况下，基本类型变量只有部分一和部分二；引用类型变量有三个部分，其中，部分二中“栈区变量的值”是部分三种对象堆内存的地址。

这里也可以解释为什么 null 比 undefined 更耗内存的问题：
当变量的值为 undefined 时候，它是基本类型变量，并且该变量只有栈区的标示符，如果我们对这个变量进行赋值操作，那么栈区的值就有值了；如果栈区是对象标识符那么堆区会有一个对象，而栈区的值则是堆区对象的地址，对于 null ，JavaScript 认为它是一个对象，虽然它的堆区是个空对象，但是栈区的标示符和值都会有值，堆区也有，这么说来 null 确实比 undefined 更耗内存了。

**JavaScript 变量复制**

（1）基本类型变量

```
var str1 = "Enjoy javascript";
var str2 = str1;

console.log(str1);
// "Enjoy javascript";
console.log(str2);
// "Enjoy javascript";

// 修改 str2 的值，str1 不会跟着改变
str2 = "Enjoy css";

console.log(str1);
// "Enjoy javascript";
console.log(str2);
// "Enjoy css"
```

（2）引用类型变量

```
var obj1 = {};
obj1.name = 'zx';
var obj2 = obj1;

console.log(obj1.name);
// zx
console.log(obj2.name);
// zx

// 修改 obj2.name 的值，obj1.name 会跟着改变
obj2.name = 'zc';

console.log(obj1.name);
// zc
console.log(obj2.name);
// zc
```

以上对变量复制的过程，引用类型变量的“原变量”和“新变量”会“绑定在一起”，它们其中一个的属性变化，另一个也会跟着变化，这叫变量复制也叫做浅拷贝；基本类型变量的“原变量”和“新变量”是独立的，一个变化不会影响另一个，这种变量复制也叫做深拷贝。

为什么会这样呢？

我们再回顾一下上文 JavaScript 变量的存储方式。基本类型变量只有栈区的变量标识符和栈区的变量值，而引用类型变量有栈区的变量标识符、栈区的变量值（堆区的地址）以及堆区的对象数据。**变量复制的本质是传值，这个值就是栈区的变量值。**基本类型变量的的值就是存放在栈区里的值，复制后，两个变量互不影响；复制引用类型变量的时候，复制的也是栈区的变量值，而这个值是对象数据在堆区的地址，堆内存的对象数据并没有复制，然后两个标识符指向同一个堆内存，当我们对其中一个变量修改（修改堆内存数据），另一个变量当然也会跟着改变。

引用类型变量（函数、数组、对象）直接复制都是浅拷贝，下面，我们举例说明如何实现其深拷贝。

① 数组深拷贝：

```
//slice 方法或 concat 方法
var arr1 = [0,1,2];
var arr2 = arr1.slice(0);
// 这句相当于 var arr2 = arr1.concat();

arr2[0] = 100;

console.log(arr1);
// [0,1,2]
console.log(arr2);
// [100,1,2]
```

② 对象深拷贝：

```
var deepCopy= function(source) { 
    var result={};
    for (var key in source) {
        if (typeof source[key] === 'object'){
            result[key] = deepCoyp(source[key]);
        } else {
            result[key] = source[key];
        }
    } 
    return result; 
}
```

**JavaScript 函数传参**

看一个简单例子：

```
function f(str,obj){
    str = 'changed string';
    obj.name = 'changed object';
}

var s = 'Enjoy javascript';
var o = {name : 'new object'};

f(s,o);

console.log(s);
// Enjoy javascript

console.log(o.name);
// changed object
```

基本类型值经过函数调用后不会改变原值，而引用类型变量却被改变了。不要觉得奇怪，因为 **JavaScript 中函数传递参数的本质也是变量复制，外部变量的值复制给函数参数。**

再看一个例子：

```
function changeFunc(f){
    f = f2;
}

function f1(){
    console.log("I'm f1 function");
}

function f2(){
    console.log("I'm f2 function");
}

changeFunc(f1);

f1();
// I'm f1 function
```

对于以上执行结果有没有感到意外呢？不必感到意外，就是这样的。函数 changeFunc 执行的时候，先将参数变量 f 指向函数 f1，然后让参数变量 f 指向函数 f2，切断了 f 和 f1 的联系，所以 f1 还是原来的 f1，不会因为 f 的参与，让它变成 f2。

**总之，记住一句话：javascript 里变量复制和函数传参都是在传递栈区的值。**

最后，看一种不声明第三个变量情况下交换俩变量值的方法：

```
var a = 1,b = 2;
a = [b,b=a][0];
a // 2
b // 1
```


参考：
[1] http://www.cnblogs.com/sharpxiajun/p/4133462.html
[2] http://www.cnblogs.com/yichengbo/archive/2014/07/10/3835882.html