---
title: 几个典型的 JavaScript 函数
date: 2017-04-14 09:33:53
tags: grammar
---

ECMAScript 中，没有独立存在的函数，所有的函数都是依附于某个对象。isNaN()、parseInt() 等看起来独立的函数，实际上它们都属于全局对象。客户端 JavaScript 中，全局对象就是 window 对象。讲函数之前，先来说说 JavaScript 对象。

<!-- more -->

JavaScript 对象细分一下，包括本地对象（native object）、内置对象（built-in object）、宿主对象以及自定义对象等四类。

> **本地对象（native object）**

本地对象是指独立于宿主环境的 ECMAScript 实现提供的对象。JavaScript 就是一种 ECMAScript 实现。ECMAScript 本地对象有：

```
Object   // 对象
Function // 函数
Array    // 数组
String   // 字符串
Boolean  // 布尔值
Number   // 数值
Date     // 日期
RegExp   // 正则
Error    // 错误
// 各种错误
EvalError RangeError ReferenceError 
SyntaxError TypeError URIError
```

> **内置对象（built-in object）**

内置对象是指由 ECMAScript 实现提供的、独立于宿主环境的所有对象，在 ECMAScript 程序开始执行时出现。按照定义，内置对象也属于本地对象。ECMAScript 内置对象有两个：

```
// 全局对象
Global

// 算数对象
Math
```

> **宿主对象**

由 ECMAScript 实现的宿主环境提供的对象。对于浏览器环境，宿主对象就是浏览器提供的对象，所有 DOM 和 BOM 对象都是宿主对象。

```
// 文档对象
document

// 窗口
window
```

BOM 的核心是 window，而 window 对象又具有双重角色，它既是通过 js 访问浏览器窗口的一个接口，又是一个 Global（全局）对象。这意味着在网页中定义的任何对象，变量和函数，都以 window 作为其 Global 对象。

> **自定义对象**

开发者自行定义的对象。

```
var o = {};
```

### 下面回到主题，讨论一下 sort、replace、match 等 3 个方法。

**(1) Array.prototype.sort**

我们都知道这个方法用于对数组元素进行排序。好，那么看一个例子：

```
var arr = [6,1,10];
arr.sort()
// [1, 10, 6]

arr
// [1, 10, 6]
// 排序后，原数组改变了
```

以上数组排序后，元素并没有按照升序排列，对这个结果是否感到意外呢？

实际上，调用该方法时，如果没有传入参数，默认根据字符串 Unicode 码点排序顺序。简单地说是按照「字典顺序」对数组元素进行排序。所谓字典顺序，即从字符串的第一个字母开始比较，如果第一个字母不同，以第一个字母的码点顺序为准；如果第一个字母相同则继续比较下一个字母，以此类推，直到比出结果。所以 10 排在 6 前面（因为 10 的第一个字符 1 在 6 前面）。

```
var fruit = ['cherries', 'apples', 'bananas'];
fruit.sort(); 
// ['apples', 'bananas', 'cherries']

var scores = [1, 10, 21, 2]; 
scores.sort(); 
// [1, 10, 2, 21]
// 字符串 '10' 比字符串 '2' 字典序靠前.

var things = ['word', 'Word', '1 Word', '2 Words'];
things.sort(); 
// ['1 Word', '2 Words', 'Word', 'word']
// 在 Unicode 码点中，数字比大写字母靠前
// 大写字母比小写字母靠前
```

如果想按照其他标准进行排序，就需要提供比较函数 compareFunction，然后数组按照调用这个函数的返回值进行排序。参数 a 和 b 是被比较的俩元素：

> 如果 compareFunction(a, b) < 0 ，那么 a 会被排列到 b 之前；
如果 compareFunction(a, b) === 0 ， a 和 b 的相对位置不变；
如果 compareFunction(a, b) > 0 ， b 会被排列到 a 之前。

compareFunction 函数格式如下：

```
function compareFunction(a, b) {
    if (根据某种排序标准 a < b) {
        return -1;
    }
    if (根据某种排序标准 a > b) {
        return 1;
    }
    // a === b
    return 0;
}
```

所以，如果需要对数字数组排序，可以这样：

```
var arr = [1, 10, 21, 2]; 
function compareFunction(a, b) {
    // a < b 的时候 a 在前
    if (a - b < 0) {
        return -1;
    }
    // a > b 的时候 a 在后
    if (a - b > 0) {
        return 1;
    }
    return 0;
}
arr.sort(compareFunction);
// [1, 2, 10, 21]

// 简化一下写法实现同样的功能：
var arr = [1, 10, 21, 2]; 
arr.sort(function(a,b){
    a - b;
});

arr
// [1, 10, 21, 2] 
//为什么会这样？？？


// 这是我敲这段代码的时候犯的一个错误
// 比较函数少写了 return 关键字

// 稍微晦涩一点的写法实现同样的功能：
var arr = [1, 10, 21, 2]; 
arr.sort(function(a,b){
    return +(a > b) || +(a === b) - 1;
});
// [1, 2, 10, 21]

// 对于表达式
var result = +(a > b) || +(a === b) - 1;
// a > b
return = +(true) -> 1
// a === b
return = +(false) || +(true) - 1 -> 1 - 1 -> 0
// a < b
return = +(false) || +(false) - 1 -> 0 - 1 -> -1
```

数字数组排序：

```
// 升序：
var arr = [1, 10, 21, 2]; 
arr.sort(function(a,b){
    return a - b;
});
// [1, 2, 10, 21]

// 降序：
var arr = [1, 10, 21, 2]; 
arr.sort(function(a,b){
    return b - a;
});

// 乱序，结果随机：
var arr = [1, 10, 21, 2]; 
arr.sort(function(a,b){
    return Math.random() > 0.5 ? -1 : 1;
});
// [10, 1, 2, 21]
```

对象按照某个属性排序：

```
var persons = [
  { name: 'Edward', age: 21 },
  { name: 'Sharpe', age: 18 },
  { name: 'And', age: 27 },
];

persons.sort(function (a, b) {
    if (a.age > b.age) {
        return 1;
    }
    if (a.age < b.age) {
        return -1;
    }
    return 0;
});
```

**(2) String.prototype.replace**

```
stringObject.replace(regexp|substr,replacement)
```

其中，第一个参数可以是字符串或者正则对象；第二个参数可以是字符串或者函数。返回值是一个新的字符串，也就是说不改变原字符串。

**（1）replacement 是字符串**

> ① 如果第一个参数是字符串 substr，用字符串 replacement 替换它；
② 如果第一个参数是正则对象 regexp，它将在 stringObject 中查找与 regexp 相匹配的子字符串，然后用 replacement 来替换这些子串。如果 regexp 具有全局标志 g，那么 replace() 方法将替换所有匹配的子串。否则，它只替换第一个匹配子串。

replacement 中的 $ 字符具有特定的含义：

```
$$ // 插入一个 "$"

$& // 插入与 regexp 匹配的子串

$` // 插入当前匹配的子串左边的内容

$' // 插入当前匹配的子串右边的内容

$1、$2、...、$99
// 与 regexp 中的第 1 到第 99 个子表达式相匹配的文本
```

交换两个单词

```
var re = /(\w+)\s(\w+)/;
var str = "Enjoy javascript";
var newstr = str.replace(re, "$2,$1");

newstr
// "javascript,Enjoy"
```

**（2）replacement 是函数**

每个匹配都调用该函数，它返回的字符串将作为替换文本使用。

该函数的第一个参数是匹配模式的字符串。接下来的参数是与模式中的子表达式匹配的字符串，可以有 0 个或多个这样的参数。接下来的参数是一个整数，声明了匹配在 stringObject 中出现的位置。最后一个参数是 stringObject 本身。

```
function change(match, p1, p2, p3, offset, string){
    // p1 非数字
    // p2 数字
    // p3 非字母(a-z,A-Z) 非数字(0-9) 非下划线 非汉字
    return [p1, p2, p3].join('-');  
}
var str = 'abc12345#$*%';
var newStr = str.replace(/([^\d]*)(\d*)([^\w]*)/,change);

str
// "abc12345#$*%"  原字符串不变

newStr
// "abc-12345-#$*%"
```

单词的首字母都转换为大写

```
var str = 'aaa bbb ccc';
var newStr = str.replace(/\b\w+\b/g, function(s){
  return s.substring(0,1).toUpperCase() + s.substring(1);
});
```

**(3) String.prototype.match**

```
// regexp 要匹配的模式的 RegExp 对象
stringObject.match(regexp)
```

如果传入一个非正则表达式对象，则会隐式地使用 new RegExp(obj) 将其转换为一个 RegExp 。

返回值为存放匹配结果的数组。该数组的内容依赖于 regexp 是否具有全局标志 g。

**（1）regexp 没有标志 g**

match() 方法就只能在 stringObject 中执行一次匹配。如果没有找到任何匹配的文本， match() 将返回 null。否则，它将返回一个数组，其中存放了与它找到的匹配文本有关的信息。该数组的第 0 个元素存放的是匹配文本，而其余的元素存放的是与正则表达式的子表达式匹配的文本。除了这些常规的数组元素之外，返回的数组还含有两个对象属性。index 属性声明的是匹配文本的起始字符在 stringObject 中的位置，input 属性声明的是对 stringObject 的引用。

```
var str = 't 17:3:28';
var res = str.match(/t (\d+?(:\d)+?)/i);

res
["t 17:3", "17:3", ":3", index: 0, input: "t 17:3:28"]
```

**（2）regexp 有标志 g**

match() 方法将执行全局检索，找到 stringObject 中的所有匹配子字符串。若没有找到任何匹配的子串，则返回 null。如果找到了一个或多个匹配子串，则返回一个数组。不过全局匹配返回的数组的内容与前者大不相同，它的数组元素中存放的是 stringObject 中所有的匹配子串，而且也没有 index 属性或 input 属性。

检索字符串中的所有数字

```
var str = "1 plus 2 equal 3";
var nums = str.match(/\d+/g)

nums
// ["1", "2", "3"]
```




参考：
[1] http://www.cnblogs.com/fanhc/archive/2013/04/09/3011354.html
[2] http://www.w3school.com.cn/jsref/
[3] https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/
[4] http://mao.li/javascript/javascript-sort/
[5] https://segmentfault.com/a/1190000000654274
