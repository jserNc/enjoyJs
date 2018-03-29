---
title: css 多列布局
date: 2017-06-14 11:58:55
tags: css
---

二列布局是指侧栏宽度固定，主栏宽度自适应；三列布局是指两侧栏宽度固定，中间主栏宽度自适应。二列布局和三列布局主要思想是类似的，将三列布局的其中一个侧栏去掉即变成二列布局。下面，我们就只讨论三列布局的几种实现方法。

<!-- more -->

** 1.左栏左浮动 右栏右浮动 主栏设置 margin **

<div style="margin-bottom:20px;"><div style="float:left;height:100px;width:120px;background-color:#ff0;">左侧栏固定宽度 120px，左浮动。</div><div style="float:right;height:100px;width:150px;background-color:#f0f;">右侧栏固定宽度 150px，右浮动。</div><div style="margin-left:120px;margin-right:150px;height:100px;background-color:#0ff;">主栏宽度自适应。margin-left 为左栏宽度，margin-right 为右栏宽度。</div></div>

dom 结构及样式：

```
// dom:
<div id="content">
    <div class="sub">左侧栏固定宽度</div>
    <div class="extra">右侧栏固定宽度</div>
    <div class="main">主栏宽度自适应</div>
</div>

// css:
.sub {
    float:left;
    width:120px;
    height:100px;
    background-color:#ff0;
}
.extra {
    float:right;
    width:150px;
    height:100px;
    background-color:#f0f;
}
.main {
    height:100px;
    background-color:#0ff;
    margin-left:120px;
    margin-right:150px;
}
```

注意 dom 结构顺序，先写两边侧栏，再写主栏。如果主栏写在前面，侧栏会出现在主栏下一行，这不是我们想要的。这样的缺点是渲染的时候先渲染两边侧栏，而不是主栏。

** 2.两个侧栏绝对定位 主栏设置 margin **

<div style="position:relative;margin-bottom:20px;"><div style="position:absolute;left:0;top:0;height:100px;width:120px;background-color:#ff0;">左侧栏固定宽度 120px，绝对定位，left、top 均为 0。</div><div style="margin-left:120px;margin-right:150px;height:100px;background-color:#0ff;">主栏宽度自适应。margin-left 为左栏宽度，margin-right 为右栏宽度。</div><div style="position:absolute;right:0;top:0;height:100px;width:150px;background-color:#f0f;">左侧栏固定宽度 150px，绝对定位，right、top 均为 0。</div></div>

dom 结构及样式：

```
// dom:
<div id="content">
    <div class="sub">左侧栏固定宽度</div>
    <div class="main">主栏宽度自适应</div>
    <div class="extra">右侧栏固定宽度</div>
</div>

// css:
.sub,.extra {
    position:absolute;
    top:0;
}
.main {
    height:100px;
    background-color:#0ff;
    margin-left:120px;
    margin-right:150px;
}
.sub {
    left:0;
    width:120px;
    height:100px;
    background-color:#ff0;
}
.extra {
    right:0;
    width:150px;
    height:100px;
    background-color:#f0f;
}
#content {
    position:relative;
}
```

** 3.圣杯布局
（三栏都左浮动，两侧栏相对定位，左栏左移，右栏右移。两侧栏都负左边距，外层设置左右内边距）**

<div class="clear" style="padding:0 150px 0 120px;margin-bottom:20px;"><div style="float:left;background-color:#0ff;height:100px;width:100%;min-width:200px;">主栏宽 100%，左浮动。</div><div style="float:left;position:relative;left:-120px;background-color:#ff0;height:100px;width:120px;margin-left:-100%;">宽度 120px，左浮动；相对定位，左移自身宽度；负左边距 100%。</div><div style="float:left;position:relative;right:-150px;background-color:#f0f;height:100px;width:150px;margin-left:-150px;">宽度 150px，左浮动；相对定位，右移自身宽度；负左边距自身宽度。</div></div>

dom 结构及样式：

```
// dom:
<div id="bd">
    <div class="main">主栏宽 100%</div>
    <div class="sub">左栏定宽</div>
    <div class="extra">右栏定宽</div>
</div>

// css:
.main,.sub,.extra {
    float:left;
}
.main {
    background-color:#0ff;
    height:100px;
    width:100%;
    min-width:200px;
}
.sub {
    position:relative;
    left:-120px;
    background-color:#ff0;
    height:100px;
    width:120px;
    margin-left:-100%;
}
.extra {
    position:relative;
    right:-150px;
    background-color:#f0f;
    height:100px;
    width:150px;
    margin-left:-150px;
}

#bd {
    padding:0 150px 0 120px;
}
```

注意，以上主侧栏的 dom 顺序不能改变。

**主要思路为：**三栏都左浮动，主栏在最前，并且宽度 100%，所以，两个侧栏都会被挤到下一行；设置两侧栏负左边距可以将两个侧栏变成和主栏同一行，要想两侧栏分别与主栏左右两侧对齐，故左栏负左边距为 100%，右栏负左边距为自身宽度；此时，主栏左右两侧分别被左栏和右栏覆盖，要想主栏不被遮挡，将两侧栏设置相对定位，左栏左移自身宽度，右栏右移自身宽度，两侧栏向两边“扩散”；这时候，整个父容器宽度会被撑开了，为了使总宽度恢复 100%，需要给父容器设置左右内边距，使得两侧栏向中间“靠拢”。

** 4.双飞翼布局
（三栏都左浮动，主栏由外层宽度为 100% 元素包裹起来；两侧栏都负左边距，主栏设置左右外边距将侧栏撑开）**

<div style="float:left;width:100%;margin-bottom:20px;"><div style="background-color:#0ff;height:100px;margin:0 150px 0 120px;">由外层宽度为 100% 左浮动元素包裹起来,主栏设置左右外边距将两侧栏撑</div></div><div style="float:left;background-color:#ff0;height:100px;width:120px;margin-left:-100%;">左浮动，负左边距 100%</div><div style="float:left;background-color:#f0f;height:100px;width:150px;margin-left:-150px;">左浮动，负左边距自身宽度</div>

dom 结构及样式：

```
// dom:
<div class="main-wrap">
    <div class="main">主栏</div>
</div>
<div class="sub">左栏</div>
<div class="extra">右栏</div>

// css:
.main-wrap{
	float:left;
	width:100%;
}
.main {
	background-color:#0ff;
	height:100px;
	margin:0 150px 0 120px;
}
.sub {
	float:left;
	background-color:#ff0;
	height:100px;
	width:120px;
	margin-left:-100%;
}
.extra {
	float:left;
	background-color:#f0f;
	height:100px;
	width:150px;
	margin-left:-150px;
}
```

圣杯布局和双飞翼布局解决的问题是一样的，就是两边定宽，中间自适应的三栏布局，中间栏要在放在文档流前面以优先渲染。双飞翼布局中主栏外套了一层 div，该 div 两边被两侧栏覆盖后，可以设置主栏左右外边距使得主栏不被覆盖就行了。所以不必再给侧栏设置相对定位偏移位置。

圣杯布局和双飞翼布局解决问题的方案在前一半是相同的，也就是三栏全部左 float，在左右两栏分别加上负 margin 让其跟中间栏 div 并排，以形成三栏布局。不同点在于解决”中间栏 div 内容不被遮挡“问题的思路不一样：圣杯布局，为了中间 div 内容不被遮挡，将外层父元素设置了 padding-left 和 padding-right ，并将左右两个 div 用相对布局 position: relative 并分别配合 right 和 left 属性，以便左右两栏 div 移动后不遮挡中间 div。双飞翼布局，为了中间主栏 div 内容不被遮挡，在中间 div 内部创建子 div 用于放置主栏内容，然后对该主栏 div 设置 margin-left 和 margin-right 为左右两栏 div 留出位置。

** 5.flex 布局**
		
<div style="display:flex;margin-bottom:20px;"><aside style="background-color:#ff0;height:100px;width:200px;
">左侧边栏宽度固定</aside><div style="flex:1;height:100px;background-color:#0ff;
">主内容栏宽度自适应</div><aside style="background-color:#ff0;height:100px;width:200px;
">右侧边栏宽度固定</aside></div>

<div style="display:flex;margin-bottom:20px;"><aside style="background-color:#ff0;height:100px;width:200px;
">第一个侧边栏宽度固定</aside><aside style="background-color:#ff0;height:100px;width:200px;
">第二个侧边栏宽度固定</aside><div style="flex:1;height:100px;background-color:#0ff;
">主内容栏宽度自适应</div></div>

<div style="display:flex;margin-bottom:20px;"><div style="flex:1;height:100px;background-color:#0ff;
">主内容栏宽度自适应</div><aside style="background-color:#ff0;height:100px;width:200px;
">第一个侧边栏宽度固定</aside><aside style="background-color:#ff0;height:100px;width:200px;
">第二个侧边栏宽度固定</aside></div>

dom 结构及样式：

```
// dom:
<div class="layout">
    <aside class="layout__aside">左侧边栏宽度固定</aside>
    <div class="layout__main">主内容栏宽度自适应</div>
    <aside class="layout__aside">右侧边栏宽度固定</aside>
</div>


<div class="layout">
    <aside class="layout__aside">第1个侧边栏宽度固定</aside>
    <aside class="layout__aside">第2个侧边栏宽度固定</aside>
    <div class="layout__main">主内容栏宽度自适应</div>
</div>


<div class="layout">
    <div class="layout__main">主内容栏宽度自适应</div>
    <aside class="layout__aside">第1个侧边栏宽度固定</aside>
    <aside class="layout__aside">第2个侧边栏宽度固定</aside>
</div>

// css:
.layout {
	display:flex;
}
.layout__aside {
	background-color:#ff0;
	height:100px;
	width:200px;
}
.layout__main {
	flex:1;
	height:100px;
	background-color:#0ff;
}
```

参考：
[1] http://web.jobbole.com/90844/
[2] http://zh.learnlayout.com/
[3] http://www.cnblogs.com/imwtr/p/4441741.html
[4] https://www.zhihu.com/question/21504052