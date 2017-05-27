---
title: JavaScript 数据类型
date: 2017-04-07 09:25:30
tags: js
---

JavaScript 数据类型共有 6 种（es6 又新增第 7 种 symbol）：数值（number）、字符串（string）、布尔值（boolean）、null、undefined、对象（object）。对象又可以分为 3 个子类型：狭义的对象（object）、数组（array）、函数（function）。

<!-- more -->

按照数据存储方式，JavaScript 数据可以分为三大类：
>（1）**基本类型：** number、string、boolean
>（2）**引用类型：** object、array、function
>（3）**特殊类型：** null、undefined

js 标识符是严格区分大小写的，所以必须注意 null、undefined 等关键词全为小写字母构成。

**那么，基本类型数据怎么能调用方法呢？**

5 种基本类型，除了 null 和 undefined 以外，其他 3 种都有与之对应的引用类型，也叫包装类型。当基本类型需要访问方法或属性时，底层会对基本类型做一个类型转换，转换成相应的引用类型，然后就可以访问该引用类型的方法（或属性）了。

```
"enjoy javascript".length;  // 16
"enjoy javascript".toUpperCase();  // "ENJOY JAVASCRIPT"

// 相当于：
var oStringObject = new String("enjoy javascript");
oStringObject.length  // 16
oStringObject.toUpperCase();  // "ENJOY JAVASCRIPT"

// 再看
1.toSting();
// 报错 SyntaxError: Invalid or unexpected token
// 解析器把 1 后面的 . 当初小数点了

// 正确姿势
(1).toString()
// "1"
```

这里提到 toString 方法，就多说两句。该方法属于 Object 对象，用于**将当前对象以字符串形式返回**。由于所有的对象都“继承”了 Object 对象，因此所有的实例对象都可以使用该方法。但是，Array、Boolean、Function、Number 以及其他本地对象都重写了该函数，以满足其自身需要。

下面分别看看各类型数据的 toString 方法返回怎样的值（chrome 控制台执行以下代码）：

```
// 对象 Object
var o = {site : "Enjoy javascript"};
o.toString();
// "[object Object]"

// 数组 Array
var arr = ['I','love','zx',1314];
arr.toString()
// "I,love,zx,1314"

// 数字 Number
var num = 0.37208511093539554;
num.toString();
// "0.37208511093539554"

// 布尔值 Boolean
var bool = true;
bool.toString();
// "true"

// 日期 Date
var now = new Date();
now.toString();
// "Thu Apr 13 2017 11:06:32 GMT+0800 (中国标准时间)"

// DOM 节点
var eles = document.getElementsByTagName("body");
eles.toString()
// "[object HTMLCollection]"
eles[0].toString()
// "[object HTMLBodyElement]"
```

这里着重说一下 Number.prototype.toString 方法。该方法将数字包装对象转换为字符串，其接收 1 个参数，表示输出字符串的进制，取值范围为整数 2-36，不在这个范围的参数会报错。如果省略参数，则默认取 10 进制。

```
// 默认 10 进制
(30).toString();
// "30"

// 36 进制（0-9,a-z 共 36 个字符）
(30).toString(36);
// "u"
```

于是，可以利用 Math.random 和 Number.prototype.toString 方法来生成随机字符串。

```
function generateRandomAlphaNum(len) {
    var s = "";
    for (;s.length < len;){
        s += Math.random().toString(36).substr(2);
    }
    return s.substr(0,len);   
}

generateRandomAlphaNum(6);
// "uhfjlr"
```

好了，言归正传。JavaScript 各个类型数据的定义就不多说了。JavaScript 是一门动态类型语言，也就是说变量声明后，可以被赋予各种类型的值。以下主要探讨如何确定一个变量最终是什么类型的值。

**JavaScript 有 3 种方法，可以确定一个值到底什么类型：**

> ① typeof 运算符
> ② instanceof 运算符
> ③ Object.prototype.toString 方法

**① typeof 运算符**

数值、字符串、布尔值分别返回 "number"、"string"、"boolean"

```
typeof 123         // "number"
typeof '123'       // "string"
typeof false       // "boolean"
```

函数返回 "function"

```
function f() {}
typeof f           // "function"
```

undefined 返回 "undefined"

```
typeof undefined   // "undefined"
```

其余都返回返回 "object"

```
typeof window      // "object"
typeof {}          // "object"
typeof []          // "object"
typeof null        // "object"
```

**typeof 运算符还可以检测一个变量是否已经声明**

```
// 没有用 var 声明也没有被赋值的变量，直接使用会报错！
v
// Uncaught ReferenceError: v is not defined

// 用 typeof 运算符不报错
typeof v
// undefined
```

没有用 var 声明也没有被赋值的变量，直接使用会报错！但是，用 typeof 对变量进行运算，如果返回值为 undefined，就知道其未定义。

```
// 变量 v 未定义
// 这样写会报错
if (v){
    //...
    // error 报错！！
}

// 正确姿势
if (typeof v === "undefined"){
    //...
}

// 注意 typeof undefined 返回值是 "undefined"
// 而不是 undefined
typeof undefined
// "undefined"
```

**② instanceof 运算符**

instanceof 运算的实质是：检查运算符右边的构造函数的 prototype 属性是否在左边的实例对象的原型链上。

那么，对以下结果我们应该也不意外：

```
var arr = [];
arr instanceof Array;		//true
arr instanceof Object;		//true

arr.__proto__ === Array.prototype  //true
Array.prototype.__proto__ === Object.prototype //true
```
所有对象（除了null）的原型链的顶层对象就是 Object.prototype，所以，任何对象对 Object 进行 instanceof 运算都会返回 true。

**③ Object.prototype.toString 方法**

该方法返回值为 "[object 参数的构造函数]"

```
Object.prototype.toString.call(2) 
// "[object Number]"

Object.prototype.toString.call('') 
// "[object String]"

Object.prototype.toString.call(true) 
// "[object Boolean]"

Object.prototype.toString.call(undefined) 
// "[object Undefined]"

Object.prototype.toString.call(null) 
// "[object Null]"

Object.prototype.toString.call(Math) 
// "[object Math]"

Object.prototype.toString.call({}) 
// "[object Object]"

Object.prototype.toString.call([]) 
// "[object Array]"
```

于是，可以写一个准确的判断数据类型的方法：

```
var type = function (o){
    var s = Object.prototype.toString.call(o);
    return s.match(/\[object (.*?)\]/)[1].toLowerCase();
};
type([]); // "array"
```
    
**null 和 undefined 区别：**

将一个变量赋值为 undefined 或 null，语法效果几乎没区别。

```
// 分别与自身严格相等
undefined === undefined  // true
null === null  // true

// 相互之间相等，严格不相等
undefined == null  // true
undefined !== null // true
undefined === null // false
```

null 是一个表示“无”的对象，转为数值时为 0；
undefined 是一个表示“无”的原始值，转为数值时为 NaN。

```
Number(null)   // 0
Number(undefined)   // NaN
```

null 的特殊之处在于，JavaScript 把它包含在对象类型（object）之中。

```
typeof null   // "object"
typeof undefined   // "undefined"
```

这里还有一点需要注意：

```
undefined == true     // false
undefined == false    // false

null == true     // false
null == false    // false
```

undefined 和 null 既不等于 true 也不等于 false，奇怪吗？

我们知道布尔值强制转换有个转换规则，以下 5 个值转换为 false，其他的值全部转为 true：

```
undefined  
null  
0  
NaN  
''(空字符串)
```

相等运算符 == 的实质是：比较相同类型的数据时，与【严格相等运算符】完全一样。比较不同类型的数据时，先将数据进行【类型转换】，然后再用严格相等运算符比较。

严格相等运算符 === 实质是：如果两个值的类型不同，直接返回 false；同一类型的原始类型的值（数值、字符串、布尔值）比较时，值相同就返回 true，值不同就返回 false；两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是比较它们是否指向同一个内存地址，两个对象对象引用同一个对象时，就返回 true。

既然相等运算符 == 会进行“类型转换”，根据布尔值强制转换规则，undefined == false 不应该返回 true 吗？并不是。原来这里的“类型转换”并不同于布尔值转换规则：

**这里的“类型转换”将 == 左右两边的运算子都转换为数值，然后再进行比较。也就是说，不管 == 两边的值是什么类型，最后统统转换为数值，而不会进行其他类型（如转为布尔型）的转换。**

有了以上理论是不是可以解释上面的 undefined 和 null 既不等于 true 也不等于 false 的现象了？

```
Number(undefined);
// NaN

Number(true);
// 1

Number(false);
// 0

NaN == 1;   // false
NaN == 0;   // false

// 事实上 NaN 跟自身都不相等，这也是 js 语言中唯一不等于自身的值
NaN === NaN   // false
NaN == NaN    // false
```

这种分析方式也许是有理的，但是，我们把再来把 null 与 false 比较就知道有问题了： 

```
Number(null);
// 0

Number(false);
// 0

null == false  // false，为什么?
```

其实，以上关于相等运算符 == 的比较规则在比较其他值的时候都适用，唯独不适用于 null。那我们把 null 和 undefined 单独分出来，记住一个事实，**null 和 undefined 之间相等，它们与其他值比较都不相等。**







参考：
[1] http://javascript.ruanyifeng.com/grammar/types.html
[2] http://www.cnblogs.com/sharpxiajun/p/4133462.html
[3] http://www.cnblogs.com/Wayou/p/things_you_dont_know_about_frontend.html
[4] http://www.cnblogs.com/fybsp58/p/5683206.html