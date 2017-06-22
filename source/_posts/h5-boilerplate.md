---
title: h5 项目脚手架
date: 2017-06-21 10:17:33
tags: h5
---

开源项目 [HTML5 Boilerplate](https://github.com/h5bp/html5-boilerplate) 作为流行的前端项目模板，包含了百来位开发者的优秀实战经验。本文对这个项目做一个简单梳理，为以后的新项目开篇提供一个参考方案。

<!-- more -->

首先看一下项目结构：

```
├── css
│   ├── main.css
│   └── normalize.css
├── doc
├── img
├── js
│   ├── main.js
│   ├── plugins.js
│   └── vendor
│       ├── jquery.min.js
│       └── modernizr.min.js
├── .htaccess
├── 404.html
├── index.html
├── humans.txt
├── robots.txt
├── crossdomain.xml
├── favicon.ico
└── [apple-touch-icons]
```

**首先 index.html:**

**① ie 条件注释：**

```
<!--[if IE 8]> <html class="no-js lt-ie9"> <![endif]-->
```

文档开头一系列的 ie 条件注释，针对不同版本的 ie 浏览器给 html 标签标记不同的 class。这样做的好处是：可以将 class 作为全局条件来兼容不同的浏览器；避免了 ie6 条件注释引起的高版本 ie 文件阻塞问题；与 Modernizr 等特征检测类库使用相同的 class，具备通用性。

“no-js” class （表示禁用了 js）是需要与 Modernizr 等特征检测类库配合使用的，如果你不想在项目中引入Modernizr，需要在 head 部分加入脚本使 “no-js” class 变为 “js” class。

```
<script>
(function(H){
    H.className = H.className.replace(/\bno-js\b/,'js');
})(document.documentElement);
</script>
```

**② meta、title 标签顺序有讲究**

为了避免 ie 中潜在的编码相关的安全问题，meta charset 字符集声明应对尽量提前，保证其在最开始的 1024 个字节里；meta charset 还应该在 title 标签之前，以防止潜在的 XSS 攻击。兼容模式的 meta 标签需要在除 title 和 meta 之外的所有元素之前；

模板里的代码顺序是：

```
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible"content="IE=edge,chrome=1">
<title></title>
```

这里的 X-UA-Compatible 属性，确保有多个渲染引擎的 ie 浏览器使用最新的版本渲染页面，可以提升用户体验。

**③ Favicons 和 Touch Icons**

快捷方式图标需要放在项目根目录下。如果在某个子目录中，需要在 head 头部中用 link 标签引用。

**④ Modernizr**

Modernizr 是一个浏览器特性检测的 js 工具库，它可以根据特性检测结果给 html 标签加上 class 属性。这使得所有浏览器可以支持 h5 元素。

由于 js 脚本文本会阻塞页面渲染，一般会将 js 脚本请求放在页面底部。但是 Modernizr.js 是个例外，它会在页面开始渲染之前同步加载。因为这个脚本需要做浏览器特性检测，并某些不支持新的 h5 元素的浏览器做兼容处理。 

**④ ie6 等浏览器升级提示**

如果项目需要支持 ie6 浏览器，删除这个片段。

**⑤ jQuery 和 Google 统计**

默认去 Google cdn 取，如果 cdn 没取到，则用本地版本（离线开发时候也可以用）。文档最后是一段谷歌统计代码，虽然谷歌建议放在页面头部，如果放在头部，除了影响页面加载速度，还会多统计一些没有完全加载页面的用户，所以，还是放在底部比较好。

**humans.txt**

项目成员、项目技术等介绍。

**robots.txt**

可以编辑这个文件，来阻止搜索引擎蜘蛛对某些页面的抓取

**.htaccess**

服务器配置，这里默认服务器是 Apache。这里有很多最佳实践配置规则，使得页面更快更安全的展现给用户。

这里需要打开以下模块：

`mod_setenvif.c` (setenvif_module)
`mod_headers.c` (headers_module)
`mod_deflate.c` (deflate_module)
`mod_filter.c` (filter_module)
`mod_expires.c` (expires_module)
`mod_rewrite.c` (rewrite_module)

以 linux 平台为例，如果 Apache 是 apt-get 方式安装的：

1.打开终端，输入以下命令：

`sudo a2enmod setenvif headers deflate filter expires rewrite include`

2.重启 Apache，使修改生效：

`sudo /etc/init.d/apache2 restart`

**js 目录:**

**① main.js**

包含项目需要的 js 代码，对于较大的项目，还可以使用 Require.js 等模块加载器来加载其他脚本。

**② plugins.js**

这个文件用来包含所需的第三方插件。例如，我们可以这样用 jQuery 插件：

```
(function($){
    // code here
})(jQuery)
```

以上闭包写法可以命名空间冲突。

**③ vendor 目录**

这个目录用来存放第三方工具库，例如上面提到的 jQuery 和 Modernizr 库文件就默认放在这里了。

**css 目录:**

**① Normalize.css**

重置默认样式，针对特定元素（属性）使不同的浏览器表现统一，参考 [ Normalize.css](http://necolas.github.io/normalize.css/) 。

**② 常用 class**

.ir（图像背景）、.hidden（隐藏元素，屏幕阅读器不可见）、.visuallyhidden（隐藏文本，屏幕阅读器可见）、.invisible（元素隐藏，但保留 dom）、.clearfix（闭合浮动） 等。

**③ 媒体查询、打印样式，分页样式**





<!--
[官网](https://html5boilerplate.com/) 对这个项目总结如下：
 
① 一个干净、移动终端友好的 HTML 模版；优化过的 Google 统计代码；触摸屏设备上使用的图标（这个可以替换成你自己的）；还有丰富的文档、技巧、窍门。
② 包含了 Normalize.css v1 版本 — 一个先进的、支持HTML5的CSS reset — 和基础样式、辅助功能、media queries、打印样式。
③ 自带了两个优秀的Javascript工具库的最新版本： jQuery (默认链接到 Google的CDN, 如果CDN失效，本地文件作为后备) 和Modernizr 浏览器特性监测工具库。
④ 我们提供了一组 Apache 配置参数，帮你提高网站的性能。 还有几个单独维护的项目：server configs, node build script, 和ant build script.
-->

参考：
[1] http://www.bootcss.com/p/html5boilerplate/
[2] https://github.com/h5bp/html5-boilerplate
[3] http://www.cnblogs.com/koleyang/p/4602190.html