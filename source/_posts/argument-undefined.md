---
title: 为什么函数形参设为 undefined
date: 2017-07-03 13:55:19
tags: problem
---

JavaScript 中函数形参的个数和实参个数并不要求相等。函数的 length 属性返回形参个数，在函数内部 argument.length 返回实参个数。形参个数小于实参个数是允许的，不会报错；形参个数大于实参个数时，被省略的参数值为 undefined。

<!-- more -->

```
function f(a,b){
    // 形参
    console.log('形参个数：',f.length);
    console.log('形参个数：',arguments.callee.length);

    //实参
    console.log('实参个数：',arguments.length);
}

// 形参个数： 2
// 形参个数： 2
// 实参个数： 1
```

例如，函数定义 2 个形参，调用时可以随便传入任意个实参：

```
function f(a,b){
    console.log('第一个参数：',a);
}

f()
// 第一个参数： undefined

f(1)
// 第一个参数： 1

f(1,2,3)
// 第一个参数： 1
```

**既然省略的参数默认是 undefined，那么 jQuery 源代码中为什么要传入形参 undefined 呢？**

```
// jQuery 源码用该匿名函数包围
(function (window,undefined) {
    // code here
})(window);
```

这绝不是多此一举。原来，在早期的 ECMAScript3 中，undefined 是可读可写的变量（ie8 及以下版本浏览器中验证），在 ECMAScript5 中才修正为只读变量。下面我们在 ie7 和 chrome 中分别运行以下代码：

```
undefined = 'newUndefined';
console.log(undefined);

// ie7 下结果为：newUndefined
// chrome 下结果为：undefined
```

如果在运行 jQuery 代码之前，我们修改了全局的 undefined 变量，并且我们不像 jQuery 源码那样显式地传入 undefined 形参，在 ie7 下会怎样？

```
undefined = 'newUndefined';
(function (window) {
    window.num = 1;
    console.log(undefined);
    console.log(typeof undefined);
})(window)

// newUndefined
// string 
```

果然，函数里打印出来的值是 newUndefined。其实，这样和显式地传入 undefined 实参得到同样的效果：

```
undefined = 'newUndefined';
(function (window) {
    window.num = 1;
    console.log(undefined);
    console.log(typeof undefined);
})(window,undefined)

// newUndefined
// string 
```

那我们和 jQuery 源码一样显式地传入 undefined 形参，在 ie7 下又会怎样？

```
undefined = 'newUndefined';
(function (window,undefined) {
    window.num = 1;
    console.log(undefined);
    console.log(typeof undefined);
})(window)

// undefined
// undefined
```

确实不一样了，函数内部的 undefined 恢复成了它本来的样子。

**所以，显式地传入 undefined 参数确实可以兼容低版本浏览器，使得函数内部代码运行的时候 undefined 就是 undefined，而不会是其他的值，毕竟函数内还有很多需要用到 undefined 的地方，如果它有新的含义，麻烦会不小。**

说到这里，我们看到的这个现象该怎么来解释呢？下面按照 ECMAScript3 中规定来理解一下上面两段代码：

```
undefined = 'newUndefined';
(function (window) {
    window.num = 1;
    console.log(undefined);
    console.log(typeof undefined);
})(window)
```

这里，没有显式传入 undefined 参数。那么根据 JavaScript 中有关作用域的理论，函数内的变量如果在函数内找不到定义，会到父函数找，父函数还找不到，就会到全局环境中找，所以，函数里的 undefined 值为 newUndefined 也就没什么奇怪的了。

```
undefined = 'newUndefined';
(function (window,undefined) {
    window.num = 1;
    console.log(undefined);
    console.log(typeof undefined);
})(window)
```

这里，显式地传入了 undefined 参数。undefined 作为函数的形参，内部 undefined 当然会优先取这个形参 undefined，而不是全局的 undefined。上面我们说到，**当形参个数大于实参个数时，被省略的参数值为 undefined（货真价实的 undefined）**。函数调用时，实参会复制给形参，所以，函数内部的 undefined 就会是最原始的 undefined 了。

如果还有疑虑，我们再看：

```
undefined = 'newUndefined';
(function (window) {
    window.num = 1;
    console.log(undefined);
    console.log(arguments[1]);
})(window)

// newUndefined 
// undefined
```

果然，不管 undefined 被重置为其他什么值，被省略的函数参数值就是最原始的 undefined。

**理解的关键点是：在 ie7 等低级浏览器下把 undefined 当做可读可写的变量看。在其他高级正常浏览器下，手动忽略 undefined，因为它就是一个只读的变量的而已。**

最后，我们用 uglify 压缩一下上段代码，对比一下：

```
// 源代码：
undefined = 'newUndefined';
(function (window,undefined) {
    window.num = 1;
    console.log(undefined);
    console.log(arguments[1]);
})(window)

// 压缩后（为了方便查看，格式化了）：
undefined = "newUndefined",
function(n, o) {
	n.num = 1,
	console.log(o),
	console.log(arguments[1])
} (window);
```

看压缩后的代码就比较明显了，形参 o 没有对应的实参，所以其值就是最原始的 undefined。**另外，这里将 window 对象作为实参传给函数，可以减少查找提高效率。因为 window 对象处于作用域的顶端，如果函数里一层层向上回溯查找 window 对象，效率比较低。**



参考：
[1] http://javascript.ruanyifeng.com/grammar/function.html#toc15
[2] http://www.cnblogs.com/SheilaSun/p/4779895.html
[3] http://thinking80s.iteye.com/blog/1113267