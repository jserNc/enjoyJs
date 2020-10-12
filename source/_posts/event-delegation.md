---
title: JavaScript 事件代理（委托）
date: 2016-10-24 13:09:38
tags: method
---

在浏览器设计过程中，关于交互，面临着一个很基本的问题：页面元素层层嵌套，那么当用户点击页面上某个区域的时候，用户真正感兴趣的到底是哪个元素呢？

<!-- more -->

举个例子：当你点击了一个按钮，你确实是点击了这个按钮，同时你也点击了该按钮所在的 div 区域，body 元素甚至 html 元素等等，浏览器怎么知道你想点的到底是哪个元素呢？

对此，dom 标准事件流规定：当一个事件发生后，会经过 3 个传播阶段：

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

事件到达最深层的节点，这个节点称为**目标节点**，在这里是 p 节点；

③ 事件冒泡

与事件捕获相反，事件从最深层元素依次向外层元素传播，直到最外层元素。冒泡事件的触发顺序是：
p -> div -> body -> document -> window。

**注意：尽管事件发生后，会在多个节点之间传播、轮流触发。但是，可以确定的是，浏览器认定的事件目标节点只有一个：那就是嵌套最深的那个节点。在这里，就是 p 节点。**

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

ie8 及其以下版本，事件对象不作为参数传递，而是通过 window 对象的 event 属性读取，并且事件对象的 target 属性叫做 srcElement 属性。所以，获取事件信息，往往要写成下面这样。

```
function handler(event) {
  var event = event || window.event;
  var target = event.target || event.srcElement;
  // ...
}
```

再注意一下 event.target，将其和 event.currentTarget 对比一下：

**event.currentTarget**：返回事件当前所在的节点，会随着事件捕获和事件冒泡改变。也就是事件监听函数中的 this。

**event.target**：返回目标节点（最深层节点），固定的。正是这个属性使得事件代理成为可能。

为了阻止事件冒泡和事件默认操作，有时候，我们还会采取以下事件处理方法：

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

以上代码中取消了事件的传播，这个处理不是必须的，可以省略。但是如果不这样，会导致事件传播到最顶层元素，甚至是 window 对象。

### 事件绑定：

除了以上把回调函数赋值给元素的 onclick 属性来监听点击事件以外，我们还可以用以下方式给元素添加各种类型事件绑定。

```
var addEvent = function (el, type, fn ){
    if (el.attachEvent) {
        // ie
        el.attachEvent( 'on' + type, fn );
    } else {
        // w3c
        el.addEventListener( type, fn, false );
    }
};
```

这里需要注意的是 addEventListener 函数的第三个参数 useCapture。这个参数布尔值，指定事件是在捕获还是冒泡阶段执行。如果指定为 true，那么事件就在捕获阶段执行；如果是 false，事件在冒泡阶段执行。一般情况下，我们都会选择 false。

举例说明：

```
<div>
    <p>点击这里</p>
</div>
```

对于以上 dom 结构，我们在 div 和 p 标签上都绑定了 click 事件。如果我们点击 p 标签，不止会触发 p 标签的 click 事件，还会触发它的父元素 div 的 click 事件。而 useCapture 这个参数可以控制这两个事件发生的先后顺序。

如果 useCapture 为 true，那就是【事件捕获】，先触发 div 的 click 事件，再触发 p 的 click 事件；如果 useCapture 为 false，那就是【事件冒泡】，先触发 p 的 click 事件，再触发 div 的 click 事件。

如果我们给所有的元素监听事件时，都设定 useCapture 为 false，那就所有的事件都在冒泡的阶段触发。而所有子元素的事件都会冒泡到祖先元素，所以，我们可以通过祖先元素来代理（委托）子元素的事件了。

如果给不同的元素设置不一样的 useCapture 值，有的设为 true，有的设为 false。那就就先从最外层向【目标节点】方向寻找 useCapture 设为 true 的捕获事件，然后到达【目标节点】执行目标元素事件后，再往外层方向寻找 useCapture 设为 false 的冒泡事件。

总结一下：两个概念不要混淆，一个是【事件传播阶段】，一个是【事件回调函数】的执行。【事件传播阶段】是固定的，捕获阶段->处于目标阶段->冒泡阶段，这是事件流的固定过程。【事件回调函数】可以绑定到捕获阶段，也可以绑定到冒泡阶段，若某个元素的回调函数绑定到捕获阶段，那么这个回调函数执行时间会早于其子元素的回调函数，若某个元素的回调绑定到冒泡阶段，那么这个回调函数的执行时间会晚于其子元素的回调函数。

问题来了，若是某个元素的捕获阶段和冒泡阶段都绑定了回调函数，那么事件发生后会触发两个回调函数吗？答案：会的。那么为啥我们写应用过程中绑定的回调函数不会执行两遍呢？那是因为我们绑定回调时一般会单独绑定在捕获阶段或单独绑定在冒泡阶段，若是分开绑定的话，确实会执行两遍的。

参考：
[1] http://javascript.ruanyifeng.com/dom/event.html
[2] http://www.diguage.com/archives/71.html
[3] https://blog.othree.net/log/2007/02/06/third-argument-of-addeventlistener/