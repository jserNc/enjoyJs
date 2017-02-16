---
title: css垂直居中几种实现方法
date: 2016-12-26 18:07:32
tags: css
---

利用css来实现div的垂直居中有多种方法，每种方法都有其优缺点，不能断定某种方法就是最好的，我们应该根据应用场景来选择合适的方案。下面介绍几种垂直居中的实现方法。

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

这种方法可以动态改变div.content的高度，但是ie兼容性不好。另外，嵌套标签也比较多。

** 方法二：使用绝对定位，结合top、margin-top等属性**
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

margin-top设置为负的div.content高度的一半。这种方法兼容性好，只是空间不够时，content会消失。

** 方法三：同样使用绝对定位，结合top、bottom、margin等属性**
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

使用绝对定位。这个div的top和bottom均为0，但是它有高度，实际上并不能上下偏移都为0，margin设为auto会使它居中（如果需要水平也居中，可以说将left、right也设置为0）。遗憾的是ie兼容性不好，空间不够时，content会被截断。

** 方法四：css3属性transform，3行搞定**
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

这里translateY表示将元素在Y方向上移动，-50%表示向上移动自身的50%高度。

** 方法五：css3 flex布局**
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

将父容器div.box的display属性设置为flex，align-items属性指定项目在交叉轴上居中。

** 最后，一个简单办法使得文本在div中垂直居中：**
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

将line-height属性设置为当前div的height值即可使文本垂直居中。

参考：
[1] http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
[2] http://zerosixthree.se/vertical-align-anything-with-just-3-lines-of-css/
[3] https://www.qianduan.net/css-to-achieve-the-vertical-center-of-the-five-kinds-of-methods/