---
title: 理解 css 块级格式化上下文（BFC）
date: 2017-06-15 14:16:59
tags: css
---

BFC（Block Formatting Contexts）即“块级格式化上下文”。BFC 是一个独立的布局环境，里面元素的位置是不受外界元素的影响的，其中的块盒和行盒（行盒由一行中所有的内联元素组成）都会沿着父容器边框垂直排列。例如，只要元素的 overflow 值不是 visible，就会触发 BFC。这也是我们将父元素设置 overflow: hidden 来清除浮动的原理。

<!-- more -->

css2.1 中除了 BFC（Block Formatting Contexts）还有 IFC（Inline formatting context）。其中 FC（Formatting Contexts）“格式化上下文”是指：页面中的一块渲染区域，该区域有自己的一套渲染规则，这个规则决定了这个区域元素如何摆放以及和其他元素的关系和作用。

**w3c 中 BFC 定义：**

> 浮动元素、绝对定位元素、非块级盒子的块级容器（inline-blocks, table-cells, 以及 table-captions 等）、以及 overflow 值不是 visible 的块级盒子，都会为它们的内容创建新的 BFC；

> 在 BFC 中，盒子从包含块顶部垂直地一个接一个排列，两个盒子之间的垂直间隙由它们的 margin 值决定。相邻盒子之间的垂直外边距会折叠。

> 在 BFC 中，每一个盒子的左外边缘（margin-left）触碰容器的左边缘（border-left）,即使存在浮动也这样，除非这个盒子又创建了一个新的 BFC。（如果是从右到左的格式，则每一个盒子的右外边缘触碰容器的右边缘。）

盒子常分为【块级盒子】和【行内盒子】。display 属性为 block，list-item，table 的元素会生成块级盒子（block-level box）；display 属性为 inline，inline-block，inline-table 的元素会生成行内盒子（inline-level box）。

**BFC 是一个独立容器，其里面元素不会影响到外面元素，也不会被外面元素影响。计算 BFC 高度时，浮动元素也参与计算。BFC 的区域不会与 float box 重叠。**举个例子：一个浮动元素 a 和另一个常规文档流元素 b 是重叠的，若给元素 b 设置一个 overflow: hidden 来创建一个 BFC，那么 a 和 b 便不再重合了，哪怕给 b 设置负的外边距也不会重合了。

**以下条件会出现 BFC：**

① float 值不是 none（float 的默认值，表示不浮动）的时候；
② position 值为 absolute | fixed；
③ overflow 值不是 visible 的时候（hidden | auto | scroll 等）；
④ display 值为 inline-block | table-cell | table-caption | flex | inline-flex；
⑤ 根元素、fieldset 元素。

需要注意的是，display:table 本身并不会创建 BFC，但是它会产生匿名框(anonymous boxes)，而匿名框中的 display:table-cell 可以创建新的 BFC，换句话说，触发块级格式化上下文的是匿名框，而不是 display:table。所以通过 display:table 和 display:table-cell 创建的 BFC 效果是不一样的。

**以下场景我们可以应用 BFC：**

**a. 阻止外边距合并**

我们知道，当两个垂直外边距相遇时，它们将形成一个外边距。合并规则如下：

> 两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。
> 两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。
> 两个外边距一正一负时，折叠结果是两者相加的和。

**注意，只有当元素在同一个 BFC 中时，垂直方向上的 margin 才会 clollpase。如果它们属于不同的 BFC，则不会有 margin collapse。**

如果 div1 和相邻的 div2 的外边距都是 50px，正常情况下他们的外边距会叠加，叠加后外边距高度还是 50px。要使得他们之间外边距高度变成 100px，可以在两个 div 外层分别加一个父级 div，给父级 div 加上属性 overflow:hidden，即可触发 BFC，外边距不会合并，高度变为 100px。

**b. 阻止行框围绕浮动框**

如果浮动的 div1 和相邻的 div2 处于同一个父元素下，正常情况下，div1 会漂浮并遮挡住 div2。如果 div2 中文本足够多，会出现文本围绕 div1 的情景。如果给 div2 加上属性 overflow:hidden，触发 BFC，div2 就不会和浮动的 div1 重叠，也就不会再文本围绕了。

**c. 闭合浮动**

如果一个容器里都是浮动元素，那么这个元素的高度就是 0。由于**计算 BFC 高度时，浮动元素也参与计算**，所以我们还可以给容器加上属性 overflow:hidden，触发 BFC，可以使得容器具有高度。


**既然以上提到多种条件可以触发 BFC，为什么上面几个例子中都选择用 overflow:hidden 来触发呢？**

我们可以为 container 容器附加属性，如 overflow:scroll，overflow:hidden，display: flex，float:left, 或 display:table 等等。尽管这些条件都能形成一个 BFC，但是它们各自却有着不一样的表现。如 overflow:scroll 可能会出现不想要的滚动条；float:left 会使得其他元素对其围绕；display:table 在响应式布局中会有问题。所以，比较简单的方式就是 overflow:hidden。

**再回过头来详细说说外边距合并：**

外边距合并的必要条件是 margin 必须邻接。**而 margin 邻接要求主体必须是处于常规文档流（非 float，非 position:absolute）的块级盒子，并且它们处于同一个 BFC 当中。**另外，如果有 clearance、padding 或 border 隔开了，也不算是邻接。

**所以，以下都是邻接，会发生外边距合并：**

① 元素的 margin-top 与其第一个常规文档流的子元素的 margin-top；
② 元素的 margin-bottom 与其下一个常规文档流的兄弟元素的 margin-top；
③ height 为 auto 的元素的 margin-bottom 与其最后一个常规文档流的子元素的 margin-bottom；
④ 高度为 0 并且最小高度也为 0，不包含常规文档流的子元素，并且自身没有建立新的 BFC 的元素的 margin-top 和 margin-bottom。前提是本身不能有 border 和 padding 高度。

**以下不会发生外边距合并：**

① 浮动元素、绝对定位元素、inline-block 元素不与任何元素外边距合并（包括其父元素、子元素）；
② 其他创建了新的 BFC 的元素（例如 overflow:hidden）和它的子元素外边距不合并；
③ 如果父元素包含 padding 或 border，那么其 margin-top 不会与第一个子元素的 margin-top 合并。margin-bottom 同理。

上面提到的 clearance（间隙）指的是：当浮动元素之后的相邻元素设置 clear 属性以清除相关方向的浮动时，这个清除浮动的元素其 margin-top 以上会产生一定的空隙（clearance）。该空隙会阻止元素 margin-top 的折叠，其作为间距存在于清除浮动元素的 margin-top 的上方。

<div style="margin-bottom:20px;"><div style="margin-bottom:20px;background-color:#0f0;height:100px;width:100px;box-shadow:0 20px 0 rgba(0,255,0,0.2);">常规文档流元素（下外边距 20px）</div><div style="float:left;margin:20px 0;background-color:#0ff;height:100px;width:200px">左浮动（上下外边距各 20px）</div><div style="clear:left;margin-top:50px;background-color:#f0f;height:100px;width:300px;box-shadow:0 -50px 0 rgba(255,0,255,0.2);">清除左浮动（上外边距 50px）</div></div>

（这里用阴影来标识外边距。）

代码结构如下：

```
// dom
<div>
	<div class="top">常规文档流元素</div>
	<div class="middle">左浮动</div>
	<div class="bottom">清除左浮动</div>
</div>

// css
.top {
    margin-bottom:20px;
    background-color:#0f0;
    height:100px;
    width:100px;
}
.middle {
    float:left;
    margin:20px 0;
    background-color:#0ff;
    height:100px;
    width:200px;
}
.bottom {
    clear:left;
    margin-top:50px;
    background-color:#f0f;
    height:100px;width:300px;
}
```

上面提到【浮动元素】不会与任何元素发生外边距合并。底部【清除浮动元素】的 margin-top 值设为 50px，可我们看到它与上面【浮动元素】的外边距是 20px。这里【浮动元素】和【清除浮动元素】貌似外边距合并了，实则不然。实际上，不管怎么修改这个 margin-top 值，这两个元素之间的间隙都等于【浮动元素】的下外边距。

w3c 规定：【清除浮动元素】在 margin-top 上所产生的间距（clearance）的值与该块盒的margin-top 之和应该足够大，即大于【浮动元素】的 margin-bottom，这样使得【清除浮动元素】的 border-top 恰好与【浮动元素】块盒的 margin-bottom 紧贴。

**也就是说，间隙多大我们不需要知道，只需记住【清除浮动元素】的 border-top 会紧贴着它上面的【浮动元素】的 margin-bottom。**

参考：
[1] http://www.html-js.com/article/1866
[2] https://segmentfault.com/a/1190000009078619
[3] http://www.jianshu.com/p/fc1d61dace7b
[4] http://www.iyunlu.com/view/css-xhtml/55.html
[5] https://www.w3cplus.com/css/understanding-bfc-and-margin-collapse.html
