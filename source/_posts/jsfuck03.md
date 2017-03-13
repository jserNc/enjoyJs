---
title: jsfuck 6 个字符写 js 代码（四）
date: 2017-01-13 09:45:15
tags: js
---

前文我们说 JavaScript 语言中所有的字符都可以用** [，]，(，)，!，+ **等 6 个字符来表示，下面，我们接着探讨这个问题，这个问题弄清楚了，也就基本掌握了 JSFuck 的原理。

<!-- more -->

** 标点符号 **

```
// 空格字符' '，而不是空字符''
'' === ''   // true
'' === ' '  // false
' ' === (NaN+[]["filter"])[11]
-> "NaNfunction filter() { [native code] }"[11]
-> ' '

'"' === ("")["fontcolor"]()[12]
-> ("").fontcolor()[12]
-> '<font color="undefined"></font>'[12]
-> '"'
// 对字符串调用fontcolor()函数，
// 会用font标签将font标签将字符串包围，并且加color属性
'str'.fontcolor('red') 
-> '<font color="red">str</font>'

'%' === Function("return escape")()([]["filter"])[20]
-> escape(String([]["filter"]))[20]
-> escape("function filter() { [native code] }")[20]
-> "function%20filter%28%29%20%7B%20%5Bnative%20code%5D%20%7D"[20]
-> '%'

'(' === (false+[]["filter"])[20]
-> "falsefunction filter() { [native code] }"[20]
-> '('

')' === (true+[]["filter"])[20]
-> "truefunction filter() { [native code] }"[20]
-> ')'

// + 作为一元运算符，强制转换数字时，优先级高于 !
// + 作为二元运算符（加法或字符串连接），优先级低于 !
// !+[]
-> true
// +[]
-> 0
// 左边是字符 '+'，右边是一元（二元）运算符 +
'+' === (+(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]])+[])[2]
-> (+(+true+(true+[])[true+true+true]+[+true]+[0]+[0])+[])[2]
-> (+(+true+("true")[3]+[1]+[0]+[0])+[])[2]
-> (+(1+"e"+[1]+[0]+[0])+[])[2]
-> (+"1e100"+[])[2]
-> (1e+100+[])[2]
-> "1e+100"[2]
-> '+'

// 如果call方法的参数是一个原始值，
// 那么这个原始值会自动转成对应的包装对象
',' === ([]["slice"]["call"](false+"")+"")[1]
-> ([]["slice"]["call"]("false")+"")[1]
-> (Array.prototype.slice.call("false")+"")[1]
-> (["f", "a", "l", "s", "e"]+"")[1]
-> "f,a,l,s,e"[1]
-> ','

'.' === (+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]
-> (+(+true+[+true]+(!![]+[])[true+true+true]+[true+true]+[+[]])+[])[+true]
-> (+(1+[1]+(!![]+[])[3]+[2]+[+[]])+[])[1]
-> (+(1+[1]+(true+[])[3]+[2]+[0])+[])[1]
-> (+(1+[1]+("true")[3]+[2]+[0])+[])[1]
-> (+(1+[1]+"e"+[2]+[0])+[])[1]
-> (+("11e20")+[])[1]
-> (1.1e+21+[])[1]
-> "1.1e+21"[1]
-> '.'

'/' === (false+[0])["italics"]()[10]
-> "false0".italics()[10]
-> "<i>false0</i>"[10]
-> '/'

':' === (RegExp()+"")[3]
-> (/(?:)/+"")[3]
-> "/(?:)/"[3]
-> ':'

'<' === ("")["italics"]()[0]
-> "<i></i>"[0]
-> '<'

'=' === ("")["fontcolor"]()[11]
-> "<font color="undefined"></font>"[11]
-> '='

'>' === ("")["italics"]()[2]
-> "<i></i>"[2]
-> '>'

'?' === (RegExp()+"")[2]
-> "/(?:)/"[2]
-> '?'

'[' === (Function("return{}")()+"")[0]
-> ({} + "")[0]
-> "[object Object]"[0]
-> '['

']' === (Function("return{}")()+"")["slice"]("-1")
-> ({} + "").slice(-1)
-> "[object Object]".slice(-1)
-> ']'

'{' === (NaN+[]["filter"])[21]
-> "NaNfunction filter() { [native code] }"[21]
-> "{"

'}' === ([]["filter"]+"")["slice"]("-1")
-> "function filter() { [native code] }".slice(-1)
-> '}'
```

仔细看完前文，我们会发现有一部分大写字母（H、J、K...）和符号（#、$...）并没有出现在以上的映射表里。

** 下面我们补全这部分字符：**

```
'Function("return unescape")()("%"'+ key.charCodeAt(0).toString(16).replace(/(\d+)/g, "+($1)+\"") + '")'

// 假如其中的 key 为 'H'

'Function("return unescape")()("%"'+ "H".charCodeAt(0).toString(16).replace(/(\d+)/g, "+($1)+\"") + '")'
-> 'unescape("%"'+ "H".charCodeAt(0).toString(16).replace(/(\d+)/g, "+($1)+\"") + '")'
-> 'unescape("%"'+ (72).toString(16).replace(/(\d+)/g, "+($1)+\"") + '")'
-> 'unescape("%"'+ "48".replace(/(\d+)/g, "+($1)+\"") + '")'
-> 'unescape("%"'+ '+(48)+"' + '")'
-> 'unescape("%"+(48)+"")'

'H' === unescape("%"+(48)+"")
-> unescape("%48")
-> 'H'
```

以上代码中的 key 换成 J、K 等其他字符，以上表达式都能获得该字符本身。

** 数字 0~9 **

```
0 === +[]

1 === +!+[]
-> +!0
-> +1
-> 1

2 === 1 + 1
2 === +!+[] + +!+[]

3 = 1 + 2
3 === +!+[] + +!+[] + +!+[]
//当然了，最前面的 + 是可以不要的
3 === !+[] + +!+[] + +!+[]

// “一生二，二生三，三生无穷”，其他数字都可以这样得到
```

好了，暂时到这里，下篇我们继续。

参考：
[1] http://javascript.ruanyifeng.com/grammar/conversion.html
[2] http://www.jsfuck.com/
[3] https://github.com/aemkei/jsfuck
[4] https://gold.xitu.io/entry/5834a964570c35006c4ac205