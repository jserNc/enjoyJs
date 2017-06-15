---
title: css 居中
date: 2016-12-26 18:07:32
tags: css
---

利用 css 来实现居中有多种方法，每种方法都有其优缺点，不能简单地断定某种方法是最好的，应该根据实际应用场景来选择合适的方案。下面分别探讨一下水平和垂直居中的实现方法。

<!-- more -->

## 垂直居中

** ① 使用表格的 vertical-align 属性**

```
<style>
    #wrapper {
        height: 100px;
        width: 600px;
        background-color: #0f0;
        display: table;
    }  
    #cell {
        display: table-cell;
        vertical-align: middle;
    }
    .content {
        height: 20px;
        width: 600px;
        background-color: #ff0;
    }
</style>
<div id="wrapper">
    <div id="cell">
        <div class="content">content</div>
    </div>
</div>
```
<div id="wrapper" style="height: 100px;width: 600px;background-color: #0f0;display: table;margin-bottom:15px;"><div id="cell" style="display: table-cell;vertical-align: middle;"><div class="content" style="height: 20px;width: 600px;background-color: #ff0;">content</div></div></div>

这种方法可以动态改变 div.content 的高度，但是 ie 兼容性不好。另外，嵌套标签也比较多。

** ② 使用绝对定位，结合 top、margin-top 等属性**
```
<style>
<style>
    #wrapper {
        position: absolute; 
        height:100px;
        width:600px;
        background-color: #0f0;
    }  
    
    .content {
        position: absolute;
        top: 50%;
        margin-top: -10px;
        height: 20px;
        background-color: #ff0;
    }
</style>
<div id="wrapper">
    <div class="content">content</div>
</div>
```

<div style="height:100px;margin-bottom:15px;"><div id="wrapper" style="position: absolute; height:100px;width:600px;background-color: #0f0;"><div class="content" style="position: absolute;top: 50%;margin-top: -10px;height: 20px;background-color: #ff0;">content</div></div></div>

margin-top 设置为负的 div.content 高度的一半。这种方法兼容性比较好，只是 div.content 空间不够时，content 会消失。

** ③ 同样使用绝对定位，结合 top、bottom、margin 等属性**
```
<style>
	#wrapper {
        height:100px;
        width:600px;
        background-color: #0f0;
		position: absolute;
    } 
    #content {
        position: absolute;
        top: 0;
        bottom: 0;
        margin: auto;
        height: 20px;
        background-color: #ff0;
    }
</style>
<div id="wrapper">
	<div id="content"> Content goes here</div>
</div>
```
<div style="height:100px;margin-bottom:15px;"><div id="wrapper" style="height:100px;width:600px;background-color: #0f0;position: absolute;"><div id="content" style="position: absolute;top: 0;bottom: 0;margin: auto;height: 20px;background-color: #ff0;"> Content goes here</div></div></div>

使用绝对定位。这个 div 的 top 和 bottom 均为 0，但是它有高度，实际上并不能上下偏移都为 0，margin 设为 auto 会使它居中（如果需要水平也居中，可以说将 left、right 也设置为 0）。只是 ie 兼容性不好，空间不够时，content 会被截断。

** ④ css3 属性 transform**
```
<style>
	#wrapper {
        height:100px;
        width:600px;
        background-color: #0f0;
        position: absolute;
    } 
    .content {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        height: 20px;
        background-color: #ff0;
    }  
</style>
<div id="wrapper">
    <div class="content">Content goes here</div>
</div>
```
<div style="height:100px;margin-bottom:15px;"><div id="wrapper" style="height:100px;width:600px;background-color: #0f0;position: absolute;"><div class="content" style="position: absolute;top: 50%;transform: translateY(-50%);height: 20px;background-color: #ff0;">Content goes here</div></div></div>

这里 translateY 表示将元素在 Y 方向上移动，-50% 表示向上移动自身的 50% 高度。

** ⑤ 文本在 div 中垂直居中**

```
<style>
    #content {
        height: 100px;
        line-height: 100px;
        background-color: #0f0;
    }
</style>
<div id="content"> Content here</div>
```
<div id="content" style="height: 100px;line-height: 100px;background-color: #0f0;margin-bottom:15px;"> Content here</div>

将 line-height 属性设置为当前 div 的 height 值即可使文本垂直居中。

** ⑥ css3 flex 布局**
```
<style>
    .box {
        display: flex;
        align-items: center;
        height: 100px;
        background-color: #0f0;
    }
    .content {
        height: 20px;
        background-color: #ff0;
    }
</style>
<div class="box">
    <div class="content">Content goes here</div>
</div>
```
<div class="box" style="display: flex;align-items: center;height: 100px;background-color: #0f0;margin-bottom:15px;"><div class="content" style="height: 20px;background-color: #ff0;">Content goes here</div></div>

将父容器 div.box 的 display 属性设置为 flex，align-items 属性指定项目在交叉轴上居中。

## 水平居中

** ① 子元素为行内元素 text-align:center**

```
<style>
    .parent-span {
        text-align :center;
        height:100px;
        background:#0f0;
    }
</style>
<div class="parent-span">
    <span>行内元素span标签</span>
</div>
```

<div style="text-align :center;height:100px;background:#0f0;margin-bottom:25px;"><span>行内元素span标签</span></div>

** ② 子元素为定宽块元素 margin:auto**

```
<style>
    .parent {
        height:100px;
        background-color:#0f0;
    }

    .child-div {
        width:200px;
        margin:auto;
        background-color:#f0f;
    }
</style>
<div class="parent">
    <div class="child-div">定宽块元素</div>
</div>
```

<div style="height:100px;background-color:#0f0;margin-bottom:25px;"><div style="width:200px;margin:auto;background-color:#f0f;">定宽块元素</div></div>

** ③ 子元素为不定宽块元素 display:inline;text-align:center;**

```
<style>
    .child-ul {
        background-color:#f0f;
        display:inline;
    }
    .parent-ul {
        height:100px;
        background-color:#0f0;
        text-align:center;
    }
</style>
<div class="parent-ul">
    <div class="child-ul">不定宽块元素</div>
</div>
```

<div style="height:100px;background-color:#0f0;text-align:center;margin-bottom:25px;"><div style="background-color:#f0f;display:inline;">不定宽块元素</div></div>

** ④ 子元素为块级元素 display:flex; align-items:center**

```
<style>
    .parent-flex {
        display:flex;
        justify-content:center;
        height:100px;
        background-color:#0f0;
    }
    .child {
        background-color:#f0f;
    }
</style>
<div class="parent-flex">
    <div class="child">子元素div</div>
</div>
<br>
<div class="parent-flex">
    <a href="#" class="child">子元素a</a>
</div>
```

<div style="display:flex;justify-content:center;height:100px;background-color:#0f0;"><div style="background-color:#f0f;">子元素div</div></div>
<div style="display:flex;justify-content:center;height:100px;background-color:#0f0;margin-bottom:50px;"><a href="#" style="background-color:#f0f;">子元素a</a></div>

参考：
[1] http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
[2] http://web.jobbole.com/90844/
[3] http://zerosixthree.se/vertical-align-anything-with-just-3-lines-of-css/
[4] https://www.qianduan.net/css-to-achieve-the-vertical-center-of-the-five-kinds-of-methods/