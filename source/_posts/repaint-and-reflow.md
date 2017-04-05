---
title: 重绘和回流（repaint & reflow）
date: 2017-03-30 15:56:54
tags: problem
---

重绘（repaint）：当元素的属性（如 color）发生变化时，浏览器会重新绘制相应的元素；

回流（reflow）：当元素布局（如 width）改变，浏览器抛弃原有属性，重新计算渲染树并重新布局。

<!-- more -->

**首先说下页面的渲染过程：**

> ① 解析 html 结构为 dom 树。包括 display : none 元素和用 js 动态添加的元素；
> ② 解析 css，产生 css 规则树；
> ③ 通过 dom 树和 css 规则树构造渲染（render）树；
> ④ 浏览器“画”出所有渲染树中的节点。

其中，渲染树不包括隐藏的节点，比如 display : none 的节点，head 节点等。

**以下情况会触发「重绘」和「回流」：**

（1）页面首次加载；
（2）dom 元素的添加，修改，删除；
（3）应用新的 css 样式；
（4）resize 浏览器窗口、滚动页面；
（5）读取元素的某些属性（scrollTop/Left、clientTop/Left、currentStyle等）。

前 4 条触发重绘或者回流的原因比较好理解。这里解释一下第 5 条：由于每次重绘、回流都会产生计算消耗，所以大多数浏览器都通过维护一个回流/重绘队列并批量执行来优化重排过程，而不是一句代码就回流/重绘一次的。但是，**获取布局信息的（返回最新的布局信息）**操作会导致队列被强制刷新（浏览器马上执行渲染队列并触发重排以返回正确值）

重绘不一定需要回流。比如改变某个元素的颜色，就只会发生重绘，而不会发生回流，因为页面布局没有变。但是回流必然会导致重绘，比如，改变一个元素的大小，就会同时触发重绘和回流。提高网页性能的手段之一就是减少页面重绘和回流，以下都是实践中的一些小技巧。

**减少「重绘」和「回流」：**

（1）**不要一条条修改样式，尽量通过 className 来修改；**

```
// 反模式，不推荐
var left = 10,top = 20;
el.style.left = left + "px";
el.style.top = top + "px";

// 推荐
el.className += " newClassName";

// 推荐
el.style.cssText += ";left:"+left+"px;top:"+top+"px;";

```
（2）**dom 离线后修改；**

  · clone 一份要修改的 DOM 节点，改完后，覆盖在线的对应节点；

  · 先把 DOM 给 display:none （有一次 reflow），然后修改。改好了再把它显示出来。

```
var oldNode = document.getElementById("res"),
    clone = oldNode.cloneNode(true);

// 处理克隆对象...

// 完成后：
oldNode.parentNode.replaceChild(clone,oldNode);
```

（3）**如果创建多个 dom 节点，使用 documentFragment 创建完后一次性加入 document；**

```
// 反模式，创建完节点立即添加进文档
var p,t;

p = document.createElement('p');
t = document.createTextNode('first paragraph');
p.appendChild(t);
document.body.appendChild(p);

p = document.createElement('p');
t = document.createTextNode('second paragraph');
p.appendChild(t);
document.body.appendChild(p);

// 推荐，使用文档碎片，添加一次碎片节点
var p,t,frag;

frag = document.createDocumentFragment();

p = document.createElement('p');
t = document.createTextNode('first paragraph');
p.appendChild(t);
frag.appendChild(p);

p = document.createElement('p');
t = document.createTextNode('second paragraph');
p.appendChild(t);
frag.appendChild(p);

document.body.appendChild(frag);
```

（4）**不要把 dom 节点的属性放在循环里，以免大量读取这个节点的属性；**

```
// 反模式
for (var i = 0;i < 100;i++){
    document.getElementById("res").innerHTML += i;
}

// 推荐
var i = 0,content = "";
for (;i < 100;i++){
    content += i;
}
document.getElementById("res").innerHTML += content;
```

（5）**尽可能地修改层级比较低的 dom；**

（6）**动画节点的 position 属性设置为 fixed 或 absolute；**

（7）**不要使用 table 布局。因为可能很小的一个小改动会造成整个 table 的重新布局。**


