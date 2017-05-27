---
title: 关于 js 脚本位置
date: 2017-03-23 09:24:41
tags: problem
---

html 文档中可以放入不限数量的脚本，脚本可以位于 html 的 &lt;head&gt; 或 &lt;body&gt; 部分中，或者同时存在于这两个部分中，当然，也可以把脚本保存在外部的扩展名为 .js 的文件中。这么说，好像我们可以随意选择脚本出现的位置，看完下面的例子，我们就知道，并不是这样。

<!-- more -->

看一段代码：

```
<html>
	<head>
		<script>
			p1.onclick = function(){
			    this.style.display = 'none';
			};
		</script>
	</head>
	<body>
		<p id="p1">点我就消失</p>
	</body>
</html>
```

以上代码保存为 .htm 文件，chrome 浏览器打开页面，点击“点我就消失”后，并没有任何反应，如果你有感到任何意外，那么，打开 chrome 控制台，重新刷新页面，我们会发现一个显著的报错：
<p style="color:red;margin-top:-40px;margin-bottom:-40px">
Uncaught TypeError: Cannot set property 'onclick' of null
<p>
这是在页面加载过程中的报错。

理论上，网页加载脚本的流程一般是这样的：
> ① 浏览器一边下载 html 代码，一边开始解析；
> ② 解析过程中，发现了 script 标签；
> ③ 暂停解析，如果是外部脚本，下载该脚本；
> ④ 下载完立即执行该脚本；
> ⑤ 继续往下解析 html 代码。

也就是说，如果在页面解析过程中，遇到了 script 标签（加了特殊属性的除外，参见 [关于 script 脚本加载方式（译）](http://nanchao.win/2016/10/28/script/) ）就会暂停页面的渲染，等脚本下载并执行完成后再继续渲染。所以，遇到加载较长时间的脚本，网页可能会暂时失去响应，出现“假死”状态。当然了，这么设计存在其合理性的，因为 JavaScript 代码是可以操作 dom 修改页面元素的，有必要等它执行完再接着渲染。

那么，上面报错的原因就是，** 在同步脚本执行过程中调用了并未渲染生成的 dom 元素。**所以，一般情况下，我们会将脚本放在页面底部，在页面 dom 元素生成完成后执行脚本，就可以避免上面的错误了。比如像这样：

```
<html>
	<head>
	</head>
	<body>
		<p id="p1">点我就消失</p>
        <script>
			p1.onclick = function(){
			    this.style.display = 'none';
			};
		</script>
	</body>
</html>
```

这么做确实达到了效果。但是，任性的我就是想要把这些操作 dom 元素的脚本放在页面顶部，行不行？

当然行！那就这样写：

```
<html>
	<head>
		<script>
			function hide(){
				console.log(this === window);
			    this.style.display = 'none';
			}
		</script>
	</head>
	<body>
		<p id="p1" onclick="hide()">点我就消失</p>
	</body>
</html>
```

刷新页面！没报错！可是，点击“点我就消失”，报错了：
<p style="color:red;margin-top:-40px;margin-bottom:-40px">
Uncaught TypeError: Cannot set property 'display' of undefined
<p>
上面第一段代码是页面加载解析过程中报错，这次是解析不报错，鼠标点击操作执行回调函数 hide 时报错。其实，这里涉及到了函数内部 this 指向的问题（关于 this ，参见 [理解 JavaScript 中 this](http://nanchao.win/2016/11/02/this/) ），通过打印 this 的指向我们可以看到点击 p 标签后 this 指向的是 window 对象，这里我们稍微修改一下回调函数 hide，把 this 改成 p 标签所在对象就可以了：

```
<html>
	<head>
		<script>
			function hide(){
				console.log(this === window);
			    p1.style.display = 'none';
			}
		</script>
	</head>
	<body>
		<p id="p1" onclick="hide()">点我就消失</p>
	</body>
</html>
```

这样是可以了，但是问题又来了：如果这个 hide 函数是外部脚本中定义好的方法，并不能随意修改呢？我们不修改 hide 函数，可以做到吗？当然也是可以的，只需修改 p 标签的回调函数绑定方式即可：

```
<html>
	<head>
		<script>
			function hide(){
			    this.style.display = 'none';
			}
		</script>
	</head>
	<body>
		<p id="p1">点我就消失</p>
		<script>
			p1.onclick = hide;
		</script>
	</body>
</html>
```

这样，hide 方法在执行的时候，内部 this 就会指向 p1 元素。点击，真的就消失了！

另外，为了方便，以上示例代码中，我们都是用元素 id 值来引用该元素的，但这种方式兼容性不好，实际生产过程中还是应该用 getElementById 方法。

参考：
[1] http://www.w3school.com.cn/js/js_howto.asp