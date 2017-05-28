---
title: chrome 扩展开发
date: 2016-11-15 17:32:59
tags: method
---

浏览器扩展可以大大地增强我们浏览器的功能。比如捕捉网页内容、捕捉 http 报文、过滤页面广告、修改网页内容…… IE 扩展开发涉及 C++ 和 COM 技术，火狐扩展开发涉及环境搭建以及 web 开发以外的知识，而 chrome 扩展相对来说就比较简单，具备 JavaScript 等前端知识就能快速上手。以下，简单地介绍一下 chrome 扩展开发。

<!-- more -->

一个 chrome 扩展就是一个配置文件 manifest.json 和一系列 html、css、js、图片文件的集合，即一个包含这些文件的目录。发布的时候，把这个目录的所有文件打包到一个扩展名为 .crx 的压缩文件中即可。然后，打开 chrome://extensions/ 页面，把 .crx 文件拖进来，即可安装该扩展。（该页面还提供“加载已解压的扩展程序”和“打包扩展程序”功能）

chrome 扩展本质上就是 web 应用，它可以使用浏览器提供的所有 api（如 XMLHttpRequest），还可以访问浏览器的内部功能（如书签）。实际过程中，借助这些 api，再加上一些前端基础知识（主要是 JavaScript），就可以完成一个简单的 chrome 扩展的开发。

一般情况下，扩展会是 browser action 或 page action 两者中的一种。browser action 扩展图标会显示在 chrome 工具栏上（地址栏之外），并且一直都显示；而 page action 图标会显示在工具栏地址栏内部右边，特定页面才有。所以，实际开发中，如果不是全部页面都使用的功能应使用 page action 方式。

下面，我们跟着 [谷歌官网教程](https://developer.chrome.com/extensions/getstarted) 做一个获取图片的小插件。

首先新建一个文件夹 getPic。

### 1.创建 manifest.json 配置文件

文件夹 getPic 内创建一个名为 manifest.json 的配置文件。这个文件以 json 格式声明扩展程序的基本信息。如扩展程序名、版本号、功能描述等。更高级点的扩展还会声明这个扩展要做什么以及需要什么样的权限等信息。下面 manifest.json 文件中，我们声明一个 browser action。其中包括 activeTab（获取当前 tab 的 url）和 host 权限(获取外部的谷歌图片搜索 api)。

```json
{
    "manifest_version": 2,
    "name": "Getting started example",
    "description": "Shows a Image for the current page",
    "version": "1.0",
    "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup.html"
    },
    "permissions": [
    "activeTab",
    "https://ajax.googleapis.com/"
    ]
}
```

### 2.创建资源文件

我们看到 manifest.json 中声明了该扩展的两个资源文件（icon.png 和 popup.html）,下面我们在文件夹 getPic 中创建这两个文件。

   - icon.png

     这个图片就是该扩展显示在 chrome 工具栏的小图标。我们可以从网上下载或者自己制作一张自定义的小图标（19px * 19px）。

   - popup.html

     这个页面在用户点击上面工具栏的 icon 图标时弹出来。这是一个标准的 html 文件，就跟我们普通的 web 页面一样。下面我们创建该文件，代码如下：

```html
<!doctype html>
 <html>
   <head>
     <title>Getting Started Extension's Popup</title>
     <style>
       body {
         font-family: "Segoe UI",Tahoma, sans-serif;
         font-size: 100%;
       }
       #status {
         /* avoid an excessively wide status text */
         white-space: pre;
         text-overflow: ellipsis;
         overflow: hidden;
         max-width: 400px;
       }
     </style>
     <script src="popup.js"></script>
   </head>
   <body>
     <div id="status"></div>
     <img id="image-result" hidden>
   </body>
</html>
```

我们看到页面里好像也没写什么实质性内容。实际上，这个页面的主体内容是由 popup.js 渲染完成的。下面，我们再来看看核心的 popup.js 代码。

```javascript
function getCurrentTabUrl(callback) {
   var queryInfo = {
        active: true,
        currentWindow: true
   };

   chrome.tabs.query(queryInfo, function(tabs) {
        var tab = tabs[0];
        var url = tab.url;
        console.assert(typeof url == 'string', 
        'tab.url should be a string');

        callback(url);
   });
 }

function getImageUrl(searchTerm, callback, errorCallback) {
   var sUrl = 'https://ajax.googleapis.com/ajax/services/search/images';
   sUrl = sUrl + '?v=1.0&q=' + encodeURIComponent(searchTerm);

   var x = new XMLHttpRequest();
   x.open('GET', sUrl);
   x.responseType = 'json';
   x.onload = function() {

     var response = x.response;
     if (!response || !response.responseData || !response.responseData.results ||response.responseData.results.length === 0) {
        errorCallback('No response from Google Image search!');
        return;
     }
     var firstResult = response.responseData.results[0];

     var imageUrl = firstResult.tbUrl;
     var width = parseInt(firstResult.tbWidth);
     var height = parseInt(firstResult.tbHeight);
     console.assert(typeof imageUrl == 'string' && !isNaN(width) && !isNaN(height),'Unexpected respose from the Google Image Search API!');
     callback(imageUrl, width, height);
   };
   x.onerror = function() {
     errorCallback('Network error.');
   };
   x.send();
 }

function renderStatus(statusText) {
   document.getElementById('status').textContent = statusText;
}

document.addEventListener('DOMContentLoaded', function() {
   getCurrentTabUrl(function(url) {

     renderStatus('Performing Google Image search for ' + url);

     getImageUrl(url, function(imageUrl, width, height) {
        renderStatus('term: ' + url + '\n' + 'result: ' + imageUrl);

        var imageResult = document.getElementById('image-result');
        imageResult.width = width;
        imageResult.height = height;
        imageResult.src = imageUrl;
        imageResult.hidden = false;
     }, function(errorMessage) {
        renderStatus('Cannot display image. ' + errorMessage);
     });
   });
});
```

getCurrentTabUrl 方法获取当前 tab 标签的 url，获取到 url 后触发 callback 回调方法；getImageUrl 方法获取图片，并填充 popup.html 主体内容。该方法第一个参数为查询关键字，第二个参数是成功取回图片后的回调方法，第三个参数是图片请求失败的错误处理方法。

### 3.安装

安装该插件。以上步骤操作完毕后，文件夹 getPic 内应该有 manifest.json、icon.png、popup.html、popup.js 等4个文件。下面我们需要把文件夹 getPic 打包成 .crx 文件。打开 chrome，更多工具->扩展程序(或者输入地址：chrome://extensions/)打开 chrome 扩展页面。勾选页面上方的“开发者模式”。点击“打包扩展程序…”按钮，在弹窗里选中文件夹 getPic，点击“打包扩展程序”。打包完毕后，我们可以在 getPic 的父级文件夹里找到打包好的 crx 文件。然后把这个 crx 文件拖拽到刚才打开的 chrome 扩展页面即可安装。如果不能安装该 crx 文件，可以点击“加载已解压的扩展程序...”直接选择文件夹 getPic。

注：以上谷歌接口可能已经废弃，所以我们看不到谷歌返回的图片。但是以上开发流程是值得参考的。以后，还会有更详细的实例介绍。



参考：
[1] https://developer.chrome.com/extensions/getstarted
[2] http://open.chrome.360.cn/extension_dev/overview.html
[3] http://www.cnblogs.com/guogangj/p/3235703.html
