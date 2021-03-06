---
title: css 三角形 扇形
date: 2017-02-14 11:05:36
tags: css
---

向下拖动当前页面，我们会看到页面右下角有个“返回顶部”按钮，其中有一个三角形。除了用三角形形状的图片，我们还可以用 css 来实现这种三角形效果，下面一起看看。

<!-- more -->

首先，我们给一个 div 设置一个边框：

```
//样式
.div { 
    border : 30px solid green;
    height : 100px;
    width : 100px;
    background : #f5f5f5;
}
```

<div style="border:30px solid green;height:100px;width:100px;background-color:#f5f5f5;"><div>

看到这里，好像并没有什么特殊的地方，下面，我们给上面的边框换成 4 种不同的颜色：

```
.div { 
    border : 30px solid;
    border-color : red yellow blue green;
    height : 100px;
    width : 100px;
    background : #f5f5f5;
}
```

<div style="border:30px solid;border-color:red yellow blue green;height:100px;width:100px;background:#f5f5f5;"><div>

这时候会看到，边框每一边都呈梯形。我们进步一把 div 内容设置为空，即 width 和 height 都为 0：

```
.div { 
    border : 30px solid;
    border-color : red yellow blue green;
    height : 0;
    width : 0;
    background : #f5f5f5;
}
```

<div style="border:30px solid;border-color:red yellow blue green;height:0;width:0;background:#f5f5f5;"><div>

于是，出现了 4 个三角形，如果我们需要得到一个三角形，比如蓝色的三角形，只需要把其他三边框的颜色设置成背景色就好了

```
.div { 
    border : 30px solid;
    border-color : #f5f5f5 #f5f5f5 blue #f5f5f5;
    height : 0;
    width : 0;
    background : #f5f5f5;
}
```

<div style="border:30px solid;border-color:#f5f5f5 #f5f5f5 blue #f5f5f5;height :0;width:0;background:#f5f5f5;"><div>

这样，一个三角形就出现了。我们通过改变边框颜色，宽度，就可以随意改变这个三角形了。

如果，我们想得到的是扇形，只需对边框附加一个 border-radius 属性即可。下面，我们分别给上边 3 个图形加上该属性：

```
.div { 
    border-radius : 100px;
    border : 30px solid;
    border-color : red yellow blue green;
    height : 100px;
    width : 100px;
    background : #f5f5f5;
}
```

<div style="border-radius:100px;border:30px solid;border-color:red yellow blue green;height:100px;width:100px;background:#f5f5f5;"><div>

矩形框，变成了圆环。接着把内容设为空：

```
.div { 
    border-radius : 30px;
    border : 30px solid;
    border-color : red yellow blue green;
    height : 0;
    width : 0;
    background : #f5f5f5;
}
```

<div style="border-radius:30px;border:30px solid;border-color:red yellow blue green;height:0;width:0;background:#f5f5f5;"><div>

把三个边框设为背景色：

```
.div { 
    border-radius : 30px;
    border : 30px solid;
    border-color : #f5f5f5 #f5f5f5 blue #f5f5f5;
    height : 0;
    width : 0;
    background : #f5f5f5;
}
```

<div style="border-radius:30px;border:30px solid;border-color:#f5f5f5 #f5f5f5 blue #f5f5f5;height:0;width:0;background:#f5f5f5;"><div>

我们看到，扇形左右两侧有蓝色的弧形，下面，我们把 3 个边框背景透明化：

```
.div { 
  border-radius : 30px;
  border : 30px solid;
  border-color:transparent transparent blue transparent;
  height : 0;
  width : 0;
  background : #f5f5f5;
}
```

<div style="border-radius:30px;border:30px solid;border-color:transparent transparent blue transparent;height :0;width:0;background:#f5f5f5;"><div>

于是，得到一个扇形，我们还可以将 div 背景色设为透明：

```
.div { 
  border-radius : 30px;
  border : 30px solid;
  border-color:transparent transparent blue transparent;
  height : 0;
  width : 0;
  background : transparent;
}
```

<div style="border-radius:30px;border:30px solid;border-color:transparent transparent blue transparent;height :0;width:0;background:transparent;"><div>

** 实时编辑 css**

为了方便调试 css 代码，实时可视化编辑 css 代码是个不错的选择。给 style 标签加上 contentEditable 属性，并将 display 属性设为 block，可以让 style 标签处于可视可编辑状态，修改 css 代码内容，样式会实时地在页面呈现出来。

```
<!DOCTYPE html>
<html>
    <body>
        <style style="display:block" contentEditable>
        	body { color: blue }
        </style>
    </body>
</html>
```

参考：
[1] http://www.cnblogs.com/Wayou/p/things_you_dont_know_about_frontend.html