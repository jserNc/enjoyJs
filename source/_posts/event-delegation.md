---
title: JavaScript 事件代理（委托）
date: 2016-10-24 13:09:38
tags: method
---

在浏览器设计过程中，关于交互，会面临一个很基本的问题：页面元素层层嵌套，那么当用户点击页面上某个区域的时候，用户真正感兴趣的到底是哪个元素呢？

举个例子：当你点击了一个按钮，你确实是点击了这个按钮，但同时你实际上也点击了按钮所在的 div 区域，body 元素以及 html 元素等等，浏览器怎么知道你想点的到底是哪个元素呢？

对此，dom 标准事件流规定：当一个事件发生后，会经过 3 个传播阶段：

<!-- more -->

**第一阶段：从顶层 window 对象传播到最深层节点（目标节点），称为：捕获阶段**
**第二阶段：事件到达最深层的目标节点，称为：处于目标阶段**
**第三阶段：事件从目标节点传播回 window 对象，称为：冒泡阶段**

以下面这段 dom 结构为例：

```
<html>
	<head>
    ...
	</head>
	<body>
        ...
		<div>
			<p>点击这里</p>
		</div>
	</body>
</html>
```
假设我们点击了 p 标签，事件传播会经过以下 3 个阶段：

① 事件捕获 

事件从最外层元素依次向深层次元素传播，直到最深层元素。捕获事件的触发顺序是：
window -> document -> body -> div -> p；

② 目标节点

事件到达最深层的节点，这个节点称为**目标节点**,在这里是 p 节点；

③ 事件冒泡

与事件捕获相反，事件从最深层元素依次向外层元素传播，直到最外层元素。冒泡事件的触发顺序是：
p -> div -> body -> document -> window。

**注意：尽管事件发生后，会在多个节点之间传播、轮流触发，但是，可以确定的是，浏览器认定的事件目标节点只有一个：那就是嵌套最深的那个节点。在这里，就是 p 节点。**

有了上面的分析，我们知道，子节点的事件总是会经过父节点。当需要监听的元素数量较多时，相对于每个节点单独的事件绑定，我们把所有子节点的监听函数定义在某个祖先节点上是一种比较好的方法。这种方法叫做**事件代理（委托）**。

一个简单的例子:

```
document.onclick = function(event){

    //IE doesn't pass in the event object
    var event = event || window.event;

    //IE uses srcElement as the target
    var target = event.target || event.srcElement;

    switch(target.id){
        case "help-btn":
                openHelp();
                break;
        case "save-btn":
                saveDocument();
                break;
        case "undo-btn":
                undoChanges();
                break;
    }
};
```

可以看到，我们是通过 target 这个变量来区分不同的子节点，来做到事件分发的。这个 target 变量就是我们上边说的**目标节点**，我们可以将其看作“事件源”。

另外，关于以上代码：

ie8 及其以下版本，事件对象不作为参数传递，而是通过 window 对象的 event 属性读取，并且事件对象的 target 属性叫做 srcElement 属性。所以，以前获取事件信息，往往要写成下面这样。

```
function handler(event) {
  var event = event || window.event;
  var target = event.target || event.srcElement;
  // ...
}
```

再注意一下 event.target,将其和 event.currentTarget 对比一下：

**event.currentTarget**：返回事件当前所在的节点，会随着事件捕获和事件冒泡改变。也就是事件监听函数中的 this。

**event.target**：返回目标节点（最深层节点），固定的。正是这个属性使得事件代理成为可能。

有时候，为了阻止事件冒泡和事件默认操作，我们还会采取以下事件处理方法：

```
function myHandler(e){
    var e = e || window.event;
    var src = e.target || e.srcElement;

    //do something

    //无冒泡
    if (typeof e.stopPropagation === 'function'){
        e.stopPropagation();    //w3c
    }

    if (typeof e.cancelBubble !== 'undefined'){
        e.cancelBubble = true;  //ie
    }

    //阻止默认操作
    if (typeof e.preventDefault === 'function'){
        e.preventDefault();     //w3c
    }

    if (typeof e.returnValue !== 'undefined'){
        e.returnValue = false;  //ie
    }
}
```



参考：
[1] http://javascript.ruanyifeng.com/dom/event.html
[2] http://www.diguage.com/archives/71.html
