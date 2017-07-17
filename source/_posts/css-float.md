---
title: css 浮动及清理浮动
date: 2017-06-19 09:47:59
tags: css
---

默认情况下，元素处于普通流。给元素设置 float 属性后，可以使元素向左或向右浮动，直到它的外边缘碰到包含块或者另一个浮动框。当然了，如果元素 float 属性设为 none，那就表示不浮动，它还是出现在它原本应该出现的位置。

<!-- more -->

关于浮动，着重理解以下几点：

> **浮动框不在文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样。浮动元素漂浮在标准流之上，所以，如果有重叠，它会遮住普通流元素。**

<div style="width:700px;overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="background-color:#ff0;height:50px;width:100px;">普通流div1，宽100px</div><div style="background-color:#0f0;height:70px;width:200px;">普通流div2，宽200px</div><div style="background-color:#0ff;height:70px;width:200px;">普通流div3，宽200px</div></div><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:50px;width:100px;">左浮动div1，宽100px</div><div style="background-color:#0f0;height:70px;width:200px;">普通流div2，宽200px</div><div style="background-color:#0ff;height:70px;width:200px;">普通流div3，宽200px</div></div><div style="float:left;border:1px solid #aaa;"><div style="float:right;background-color:#ff0;height:50px;width:100px;">右浮动div1，宽100px</div><div style="background-color:#0f0;height:70px;width:200px;">普通流div2，宽200px</div><div style="background-color:#0ff;height:70px;width:200px;">普通流div3，宽200px</div></div></div>

上图，左边是 3 个普通流块级框，竖直方向依次排列；中间 div1 左浮动，于是它会脱离普通文档流向并左移动，直到它的边缘碰到父容器 content-box 边缘（如果父容器设置了 padding，那 div1 左边缘和父元素左边缘直接还隔着 padding-left）为止。其后的 div2 就当 div1 不存在一样，跑到父容器顶端了。浮动的 div1 漂浮在 div2 之上，遮住了 div2 部分区域。右边 div1 右浮动，和中间情况类似。

> **元素被浮动以后，需要指定一个明确的宽度；否则，它会尽可能地窄。浮动元素会生成一个块级框，而不论它本身是何种元素。**

<div style="width:600px;overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:50px;">左浮动div1，没设宽度</div><div style="background-color:#0f0;height:70px;width:200px;">普通流div2，宽200px</div><div style="background-color:#0ff;height:70px;width:200px;">普通流div3，宽200px</div></div><div style="float:left;border:1px solid #aaa;"><div style="float:left;background-color:#ff0;height:50px;">左浮动</div><div style="background-color:#0f0;height:70px;width:200px;">普通流div2，宽200px</div><div style="background-color:#0ff;height:70px;width:200px;">普通流div3，宽200px</div></div></div>

浮动 div1 没设置宽度，它的实际宽度会等于内容的宽度。也就是是它会尽可能得窄，只要能满足内容宽度即可。

> **浮动元素会依次横向排列。如果父容器宽度不够，剩下的浮动元素就会从移动到下一行开始继续横向排列，这里的“移动到下一行”是指：往下移动尽可能少的距离，只要剩余宽度足够放下这个浮动元素就行。**


<div style="width:620px;overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;height:150px;width:320px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div><div style="float:left;background-color:#0ff;height:50px;width:100px;">左浮动div3</div></div><div style="float:left;height:150px;width:220px;border:1px solid #aaa;"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div><div style="float:left;background-color:#0ff;height:50px;width:100px;">左浮动div3</div></div></div>

上图所示，如果容器宽度足够，3 个浮动 div 依次水平排列；如果容器宽度只能容纳 2 个浮动元素，那么第 3 个元素就会被“挤”到下一行，而这里的下一行并不是指 div1 下面，因为它不需要移动到 div 下面那么远（指垂直距离）就可以被容纳下了。简单地说，标准流元素竖向排列，浮动使得元素横向排列。

另外，这里的浮动元素依次排序，“依次”是指，左浮动时（float:left），依次从父容器左边缘向右排列；右浮动时（float:right），依次从父元素右边缘向左排列。

> **浮动框旁边的行框被缩短，从而给浮动框留出空间，行框围绕浮动框。因此，创建浮动框可以使文本围绕它。**

<div style="width:700px;overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="background-color:#ff0;height:50px;width:100px;">普通流div1，宽100px</div><div style="background-color:#0f0;height:70px;width:50px;">普通流div2，宽50px</div><div style="background-color:#0ff;height:120px;width:200px;">普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；</div></div><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="background-color:#ff0;height:50px;width:100px;">普通流div1，宽100px</div><div style="float:left;background-color:#0f0;height:70px;width:50px;">左浮动div2，宽50px</div><div style="background-color:#0ff;height:120px;width:200px;">普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；普通流div3宽200px；</div></div><div style="float:left;border:1px solid #aaa;margin-right:10px"><div style="background-color:#ff0;height:50px;width:100px;">普通流div1，宽100px</div><div style="float:left;background-color:#0f0;height:70px;width:50px;">左浮动div2，宽50px</div><div style="margin-top:70px;background-color:#0ff;height:120px;width:200px;">普通流div3 margin-top:70px；普通流div3 margin-top:70px；普通流div3 margin-top:70px；普通流div3 margin-top:70px；普通流div3 margin-top:70px；</div></div></div>

上图，左边 3 个 div 都是普通文档流，从上到下排序；中间 div2 左浮动，漂浮在 div3 上面，div 3 中的文本会围绕浮动的 div2，也就是，div3 中被 div2 覆盖的部分是不会有文本的；右边 div2 依然左浮动，只是 div3 的 margin-top 属性设置为 div2 的高，所以，看上去效果和左边一样。


> **假设某个元素 A 是浮动的，如果 A 元素上一个元素也是浮动的，那么 A 元素会跟随在上一个元素的后边（左浮动右边是后边，右浮动左边是后边。如果一行放不下这两个元素，那么 A 元素会被挤到下一行）。如果 A 元素上一个元素是普通流中的元素 B，根据这个元素 B 类型有以下规则：a) 元素 B 是块级元素，那么 A 相对于它浮动之前的位置不变，也就是说 A 的顶部总是和上一个元素的底部紧贴；b) 元素 B 是行内元素，行框被缩短，从而给浮动框留出空间，行框围绕浮动框。**

<div style="overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;width:235px;border:1px solid #aaa;margin-right:10px"><div style="background-color:#ff0;height:50px;width:100px;">普通流div1，宽度100px</div><div style="float:left;background-color:#0f0;height:70px;width:120px;">左浮动div2，宽120px</div><div style="float:left;background-color:#0ff;height:100px;width:80px;">左浮动div3，宽200px</div><div style="background-color:#f0f;height:200px;width:100px;">普通流div4，宽100px</div></div><div style="float:left;border:1px solid #aaa;margin-right:10px"><span style="background-color:#ff0;height:50px;">普通流span</span><div style="float:left;background-color:#0f0;height:70px;width:120px;">左浮动div2，宽120px</div><div style="float:left;background-color:#0ff;height:100px;width:80px;">左浮动div3，宽200px</div><div style="background-color:#f0f;height:200px;width:100px;">普通流div4，宽100px</div></div><div style="float:left;border:1px solid #aaa;margin-right:10px"><span style="background-color:#ff0;height:50px;">普通流span</span><div style="background-color:#0f0;height:70px;width:150px;">普通流div2，宽150px</div><div style="float:left;background-color:#0ff;height:100px;width:80px;">左浮动div3，宽200px</div><div style="background-color:#f0f;height:200px;width:120px;">普通流div4，宽120px</div></div></div>


左边 div2、div3 左浮动，div1、div4 普通流；中间和左边的区别是，div1 换成了 span 标签；右边和中间的区别是 div2 去掉了浮动，成为普通流元素。

> **清除浮动 clear : none | left | right | both，这个规则只能影响使用清除的元素本身，不能影响其他元素。**

<div style="overflow:hidden;zoom:1;margin-bottom:15px;"><div style="float:left;height:150px;width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div></div><div style="float:left;height:150px;width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;clear:right;background-color:#ff0;height:70px;width:100px">左浮动div1，clear : right</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div></div><div style="float:left;height:150px;width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1，clear : right</div><div style="float:left;clear:left;background-color:#0f0;height:50px;width:100px;">左浮动div2，clear : left</div></div></div>

以上三图中 div1、div2 都是都是左浮动，现在我们需要 div2 移到 div1 下面去，就好像 div1 是普通流元素一样。

中间图给 div1 加上属性 clear : right，结果和左图没什么区别，并没什么用；右图给 div2 加上属性 clear : left，成功把 div2 移到 div1 下面。

总结一下就是：如果要想 div2 左边没有浮动元素，那就该对 div2 设置 clear : left 属性，而不是对其他元素设置；如果要想 div2 右边没有浮动元素，那就该对 div2 设置 clear : right 属性，而不是对其他元素设置。两边都情浮动，则用 clear : both。

> **如果一个父容器中所有元素都浮动，则会父容器高度塌陷，表现出高度为 0。**

<div style="width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div></div><div style="background-color:#0ff;height:100px;margin-bottom:15px">紧跟上面父容器后的div3,宽度自适应，高度100px</div>

上图看到，div1、div2均浮动后，父容器高度变为 0了，上面的灰线是父容器的上下边框折叠一起的情形。浮动元素都漂浮在后面的普通流 div 上面了，这样显然是不行的。好，那就给 div3 清浮动吧！

<div style="width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div></div><div style="clear:left;background-color:#0ff;height:100px;margin-bottom:15px">紧跟上面父容器后的div3,宽度自适应，高度100px，clear:left</div>

div3 加上属性 clear:left 后，div3 移到浮动元素下面去了，这很好！可是，上面父容器还是高度塌陷，这还不是我们想要的！

给上面父容器加个属性 overflow:hidden 看看：

<div style="overflow:hidden;width:215px;border:1px solid #aaa;margin-right:10px"><div style="float:left;background-color:#ff0;height:70px;width:100px">左浮动div1</div><div style="float:left;background-color:#0f0;height:50px;width:100px;">左浮动div2</div></div><div style="background-color:#0ff;height:100px;margin-bottom:15px">紧跟上面父容器后的div3,宽度自适应，高度100px</div>

至此，父容器高度终于恢复正常了。至于 overflow:hidden 属性为什么有这样的作用，参见 [理解 css 块级格式化上下文（BFC）](http://nanchao.win/2017/06/15/bfc/) 。

使用 clear 清除了浮动，但是没有使父容器高度不再坍塌，而使用 overflow:hidden 使父容器高度恢复了正常，所以有人喜欢将前者叫作【**清除浮动**】，后者叫作【**闭合浮动**】。

> **闭合浮动的原理：只要触发了 hasLayout（针对 ie6-7 浏览器） 或 BFC（针对其他浏览器） 即可闭合浮动。**

** (1) hasLayout **

在 IE 浏览器中，一个元素要么自己对自身的内容进行组织和计算大小，要么依赖于包含块来计算尺寸和组织内容。为了协调这两种方式的矛盾，渲染引擎采用了 hasLayout 属性，当一个元素的 hasLayout 属性值为 true 时，我们说这个元素有一个布局（layout），或拥有布局。在 IE 中可以通过 hasLayout 属性来判断一个元素是否拥有 layout ，如 element.currentStyle.hasLayout 。

layout 是 IE 的专有概念，那些拥有 layout（hasLayout）的元素负责本身及其子元素的尺寸设置和定位。layout 可以被某些 CSS 属性不可逆的触发，而某些 HTML 元素本身就具有 layout 。

**hasLayout 触发的条件：**

① position: absolute（fixed 是 absolute 的子类）
② float: left | right 
③ display: inline-block 
④ width: 除 auto 外的任意值 
⑤ height: 除 auto 外的任意值 （例如很多人闭合浮动会用到 height: 1%  ） 
⑥ zoom: 除 normal 外的任意值 （例如：zoom:1）
⑦ writing-mode: tb-rl

另外，在 IE7 中，overflow 也可以触发 layout，overflow: hidden|scroll|auto。


** (2) BFC **

参见上文 [理解 css 块级格式化上下文（BFC）](http://nanchao.win/2017/06/15/bfc/) 。

> **闭合浮动的方法。**

** a) 使用 :after 伪元素。只要父容器中有普通文档流元素，并清除了浮动，它就不会塌陷；**

```
.clearfix:after {
    clear: both;
    content: ".";
    display: block;
    height: 0;
    visibility: hidden;
}
.clearfix {
    /** ie6-7 **/
    *zoom: 1;
}

<div class="wrap clearfix">
    <div class="fleft">左浮动</div>
    <div class="fleft">左浮动</div>
    <div style="clear:both;"></div>
</div>
```

其中，display:block 使生成的元素以块级元素显示，占满剩余空间；height:0 避免生成内容破坏原有布局的高度；visibility:hidden 使生成的内容不可见；通过 content:"."生成内容作为最后一个元素，至于 content 里面是 "." 还是其他都是可以的。

注意：css 属性名前加 \* 表示该属性对 ie6-7 有效，其他浏览器不识别，如这里的 \*zoom；而 \_ 开的属性只有 ie6 才能识别。

** b) 浮动末尾添加空标签，对该空标签添加 clear 属性；**

```
<div class="wrap">
    <div class="fleft">左浮动</div>
    <div class="fleft">左浮动</div>
    <div style="clear:both;"></div>
</div>
```

** c) 使用 br标签和其自带的 clear 属性，br 有 clear : all | left | right | none 属性； **

```
<div class="wrap">
    <div class="fleft">左浮动</div>
    <div class="fleft">左浮动</div>
    <br clear="all" />
</div>
```

** d) 父元素设置 overflow：hidden | auto（在 ie6 中设置 zoom:1）；**

```
.wrap {
    overflow：hidden;
    zoom:1
}

<div class="wrap">
    <div class="fleft">左浮动</div>
    <div class="fleft">左浮动</div>
</div>
```

** e) 父元素也浮动（或父元素设置 display:table）**

```
.wrap {
    float:left;
    /*display:table;*/
    zoom:1
}

<div class="wrap">
    <div class="fleft">左浮动</div>
    <div class="fleft">左浮动</div>
</div>
```

参考：
[1] http://www.w3school.com.cn/css/css_positioning_floating.asp
[2] http://www.iyunlu.com/view/css-xhtml/55.html
[3] http://www.cnblogs.com/iyangyuan/archive/2013/03/27/2983813.html
[4] https://segmentfault.com/a/1190000002616482