---
title: css 垂直居中几种实现方法
date: 2016-12-26 18:07:32
tags: css
---

利用 css 来实现 div 的垂直居中有多种方法，每种方法都有其优缺点，不能简单地断定某种方法是最好的，应该根据实际应用场景来选择合适的方案。下面介绍几种垂直居中的实现方法。

<!-- more -->

** 方法一：使用表格的 vertical-align 属性**
```
<style>
    #wrapper {
        height: 200px;
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
<div id="wrapper" style="height: 200px;width: 600px;background-color: #0f0;display: table;"><div id="cell" style="display: table-cell;vertical-align: middle;"><div class="content" style="height: 20px;width: 600px;background-color: #ff0;">content</div></div></div>

这种方法可以动态改变 div.content 的高度，但是 ie 兼容性不好。另外，嵌套标签也比较多。

** 方法二：使用绝对定位，结合 top、margin-top 等属性**
```
<style>
<style>
    #wrapper {
        position: absolute; 
        height:200px;
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

<div style="height:200px;"><div id="wrapper" style="position: absolute; height:200px;width:600px;background-color: #0f0;"><div class="content" style="position: absolute;top: 50%;margin-top: -10px;height: 20px;background-color: #ff0;">content</div></div></div>

margin-top 设置为负的 div.content 高度的一半。这种方法兼容性比较好，只是 div.content 空间不够时，content 会消失。

** 方法三：同样使用绝对定位，结合 top、bottom、margin 等属性**
```
<style>
	#wrapper {
        height:200px;
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
<div style="height:200px;"><div id="wrapper" style="height:200px;width:600px;background-color: #0f0;position: absolute;"><div id="content" style="position: absolute;top: 0;bottom: 0;margin: auto;height: 20px;background-color: #ff0;"> Content goes here</div></div></div>

使用绝对定位。这个 div 的 top 和 bottom 均为 0，但是它有高度，实际上并不能上下偏移都为 0，margin 设为 auto 会使它居中（如果需要水平也居中，可以说将 left、right 也设置为 0）。只是 ie 兼容性不好，空间不够时，content 会被截断。

** 方法四：css3 属性 transform**
```
<style>
	#wrapper {
        height:200px;
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
<div style="height:200px;"><div id="wrapper" style="height:200px;width:600px;background-color: #0f0;position: absolute;"><div class="content" style="position: absolute;top: 50%;transform: translateY(-50%);height: 20px;background-color: #ff0;">Content goes here</div></div></div>

这里 translateY 表示将元素在 Y 方向上移动，-50% 表示向上移动自身的 50% 高度。

** 方法五：css3 flex 布局**
```
<style>
    .box {
        display: flex;
        align-items: center;
        height: 200px;
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
<div class="box" style="display: flex;align-items: center;height: 200px;background-color: #0f0;"><div class="content" style="height: 20px;background-color: #ff0;">Content goes here</div></div>

将父容器 div.box 的 display 属性设置为 flex，align-items 属性指定项目在交叉轴上居中。

** 最后，一个简单办法使得文本在 div 中垂直居中：**
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
<div id="content" style="height: 100px;line-height: 100px;background-color: #0f0;"> Content here</div>

将 line-height 属性设置为当前 div 的 height 值即可使文本垂直居中。

参考：
[1] http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
[2] http://zerosixthree.se/vertical-align-anything-with-just-3-lines-of-css/
[3] https://www.qianduan.net/css-to-achieve-the-vertical-center-of-the-five-kinds-of-methods/