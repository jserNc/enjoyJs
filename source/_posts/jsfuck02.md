---
title: jsfuck 6 个字符写 js 代码（三）
date: 2017-01-11 09:49:12
tags: grammar
---

查看 [JSFuck源码](https://github.com/aemkei/jsfuck/blob/master/jsfuck.js) ，我们发现，之所以能用** [，]，(，)，!，+  **等 6 个字符来写 JavaScript 代码，是因为所有的字符（变量）都可以通过这 6 个字符来表示（直接或者间接转换），下面我们分析各个字符和这 6 个元字符之间的映射关系。

<!-- more -->

** 关键字 **

```
false === ![]

true === !![]

undefined === [][[]]

+[![]]
-> NaN
//NaN不等于其自身，所以，不能像上面那样用 === 表示

Infinity === +(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+
             [+!+[]]+[+[]]+[+[]]+[+[]])
```

** 构造函数 **

```
Array === [].constructor
-> function Array() { [native code] }

Number === (+[]).constructor
-> (0).constructor
-> function Number() { [native code] }

String === ([]+[]).constructor
-> ("").constructor
-> function String() { [native code] }

Boolean === (![]).constructor
-> (false).constructor
-> function Boolean() { [native code] }

Function === []["filter"].constructor
-> Array.prototype.filter.constructor
-> function Function() { [native code] }

RegExp === Function("return/"+false+"/")().constructor
-> /false/.constructor
-> function RegExp() { [native code] }
```

** 小写字母映射 **

```
"a" === (false+"")[1]
-> "false"[1]
-> "a"

"b" === (Function("return{}")()+"")[2]
-> ({} + "")[2]
-> "[object Object]"[2]
-> "b"

"c" === ([]["filter"]+"")[3]
-> (Array.prototype.filter + "")[3]
-> "function filter() { [native code] }"[3]
-> "c"

"d" === (undefined+"")[2]
-> "undefined"[2]
-> "d"

"e" === (true+"")[3]
-> "true"[3]
-> "e"

"f" === (false+"")[0]
-> "false"[0]
-> "f"

"g" === (false+[0]+String)[20]
-> "false0function String() { [native code] }"[20]
-> "g"

"h" === (+(101))["toString"](21)[1]
-> (101).toString(21)[1]
-> "4h"[1]
-> "h"
//其中，toString()可以传递一个参数，表示待转换的数值的基数

"i" === ([false]+undefined)[10]
-> "falseundefined"[10]
-> "i"

"j" === (Function("return{}")()+"")[10]
-> ({} + "")[10]
-> "[object Object]"[10]

"k" === (+(20))["toString"](21)
-> (20).toString(21)
-> "k"

"l" === (false+"")[2]
-> "false"[2]
-> "l"

"m" === (Number+"")[11]
-> "function Number() { [native code] }"[11]
-> "m"

"n" === (undefined+"")[1]
-> "undefined"[1]
-> "n"

"o" === (true+[]["filter"])[10]
-> "truefunction filter() { [native code] }"[10]
-> "o"

"p" === (+(211))["toString"](31)[1]
-> (211).toString(31)[1]
-> "6p"[1]
-> "p"

"q" === (+(212))["toString"](31)[1]
-> "6q"[1]
-> "q"

"r" === (true+"")[1]
-> "true"[1]
-> "r"

"s" === (false+"")[3]
-> "false"[3]
-> "s"

"t" === (true+"")[0]
-> "true"[0]
-> "t"

"u" === (undefined+"")[0]
-> "undefined"[0]
-> "u"

"v" === (+(31))["toString"](32)
-> (31).toString(32)
-> "v"

"w" === (+(32))["toString"](33)
-> (32).toString(33)
-> "w"

"x" === (+(101))["toString"](34)[1]
-> (101).toString(34)[1]
-> "2x"[1]
-> "x"

"y" === (NaN+[Infinity])[10]
-> "NaNInfinity"[10]
-> "y"

"z" === (+(35))["toString"](36)
-> (35).toString(36)
-> "z"
```

说明一点：将一个值转换成字符串有两种方法，一是使用 toString() 方法，二是使用转型函数 String()，几乎每个值都有 toString() 方法（ null 和 undefined 没有该方法）。

** 大写字母映射 **

```
"A" === (+[]+Array)[10]
-> "0function Array() { [native code] }"[10]
-> "A"

"B" === (+[]+Boolean)[10]
-> "0function Boolean() { [native code] }"
-> "B"

"C" === Function("return escape")()(("")["italics"]())[2]
-> scape("".italics())[2]
-> escape("<i></i>")[2]
-> "%3Ci%3E%3C/i%3E"[2]
-> "C"
//italics()方法会在字符串，外层加上<i>标签，如：
"字符变斜体".italics()
-> "<i>字符变斜体</i>"
//escape() 函数可对字符串进行编码,如：
escape("<")
-> "%3C"

"D" === Function("return escape")()([]["filter"])["slice"]("-1")
-> escape([]["filter"])["slice"]("-1")
-> escape(String([]["filter"]))["slice"]("-1")
-> escape("function filter() { [native code] }")["slice"]("-1")
-> "function%20filter%28%29%20%7B%20%5Bnative%20code%5D%20%7D"["slice"]("-1")
-> "D"

"E" === (RegExp+"")[12]
-> "function RegExp() { [native code] }"[12]
-> "E"

"F" === (+[]+Function)[10]
-> "0function Function() { [native code] }"[10]
-> "F"

"G" === (false+Function("return Date")()())[30]
-> (false+Date())[30]
-> (false+"Wed Jan 11 2017 13:32:46 GMT+0800 (中国标准时间)")[30]
-> "falseWed Jan 11 2017 13:32:46 GMT+0800 (中国标准时间)"[30]
-> "G"

"I" === (Infinity+"")[0]
-> "Infinity"[0]
-> "I"

"M" === (true+Function("return Date")()())[30]
-> "trueWed Jan 11 2017 13:36:13 GMT+0800 (中国标准时间)"[30]
-> "M"

"N" === (NaN+"")[0]
-> "NaN"[0]
-> "N"

"O" === (NaN+Function("return{}")())[11]
-> (NaN+{})[11]
-> "NaN[object Object]"[11]

"R" === (+[]+RegExp)[10]
-> "0function RegExp() { [native code] }"[10]
-> "R"

"S" === (+[]+String)[10]
-> "0function String() { [native code] }"[10]
-> "S"

"T" === (NaN+Function("return Date")()())[30]
-> "NaNWed Jan 11 2017 13:54:41 GMT+0800 (中国标准时间)"[30]
-> "T"

"U" === (NaN+Function("return{}")()["toString"]["call"]())[11]
-> (NaN + ({}).toString.call())[11]
-> (NaN + "[object Undefined]")[11]
-> "NaN[object Undefined]"[11]
-> "U"
//注意({}).toString.call()的用法
({}).toString.call(window)
-> "[object Window]"
({}).toString.call(null)
-> "[object Null]"
({}).toString.call(undefined)
-> "[object Undefined]"
({}).toString.call(1)
-> "[object Number]"
```

由于篇幅太长，不便查看，就先写到这里。下篇我们接着这里讨论。


参考：
[1] http://javascript.ruanyifeng.com/grammar/conversion.html
[2] http://www.jsfuck.com/
[3] https://gold.xitu.io/entry/5834a964570c35006c4ac205
