---
title: 浏览器本地存储（二）
date: 2017-04-05 09:36:38
tags: method
---

上篇讨论了 cookie 本地存储方式，这里继续讨论 localStorage。鉴于 cookie 会随着每个服务器请求来回传递，所以会影响请求速度，而且 cookie 的个数和容量都很有限。html5 提供了 localStorage 和 sessionStorage 等两种客户端存储数据的新方法。

<!-- more -->

**localStorage 相比于 cookie，最显著的特点就是：**

① 除非主动清除数据，否则永远不会过期；
② 存储空间较大，5MB 左右；
③ 数据仅存在于浏览器客户端，不会随着请求来回传递；
④ ie8 以下浏览器不支持，兼容性不太好；
⑤ 同域名下，http 和 https 不可以共享数据。

**localStorage 与 sessionStorage 区别是：**

sessionStorage 针对一个 session 进行数据本地存储。当用户关闭当前浏览器窗口后，本地数据会被删除。sessionStorage 仅在当前浏览器窗口有效（允许刷新），关闭当前浏览器窗口或者浏览器后失效。另外，sessionStorage 不在不同的浏览器窗口中共享，即使是同一个页面也不行，而 localStorage 和 cookie 在所有同源窗口中都是共享的。

**localStorage 使用方法：**

**(1) 读取数据**

直接打印 localStorage 会输出一个包含各个“键值对”的 Storage 对象。

```
localStorage
/*
Storage {
    stopXP1 : "1",
    stopXPTime1 : "1480559911552",
    unloginTheme : "145,
    ...
}
*/
```

可以用点运算符或者方括号运算符直接访问数据，例如：

```
// 方式一：
localStorage.stopXP1
// "1"

// 方式二：
localStorage["stopXP1"]
// "1"
```

也可以用 localStorage 对象的 getItem 方法：

```
// 方式三：
var stopXP1 = localStorage.getItem("stopXP1");
stopXP1
// "1"
```

**(2) 写入数据**

使用 localStorage 对象的 setItem 方法，或者直接用对象访问运算符:

```
// 方式一：
localStorage.setItem("stopXP1","2");

// 方式二：
localStorage.stopXP1 = "2";

// 方式三：
localStorage["stopXP1"] = "2";
```

**(3) 清除数据**

removeItem 方法用于清除某个键名对应的数据，clear 方法用于清除所有保存的数据。

```
// 清除 1 条数据
localStorage.removeItem('stopXP1');

localStorage.stopXP1
// undefined

// 清除所有数据
localStorage.clear();

localStorage
// Storage {length: 0}
```

前面说到 ie8 以下低版本浏览器不支持 localStorage。针对这些低版本 ie 浏览器，我们用 userData 方式来存储数据。

判断浏览器是否支持 localStorage，只需判断 window 对象的 localStorage 属性是否存在即可：

```
if (window.localStorage){
    console.log("支持 localStorage");
}
```
userData 存储通过将数据写入一个 userData 存储区来保存数据，userData 将数据以 XML 格式保存在客户端上，每个域名使用一个文件夹保存 userData 存储区数据。注意，userData 这种存储方式只适用于 ie 浏览器。

使用 userData 我们首先需要在文档中某元素加入 behavior 属性，内容为 “#default#userdata”：

```
var html = document.documentElement;
html.addBehavior("#default#userdata");
```

或者显示地写在文档标签里，效果一样：

```
<input style="BEHAVIOR:url(#default#userData)"/>
```

以上 html 元素（或 input 元素）加上了 behavior 属性后，该元素就具备了一下属性和方法：

```
// 属性：
expires : 设置或者获取过期时间
XMLDocument : 获取 XML 引用

// 方法：
load(存储区名) :      
// 从 userData 存储区载入存储的对象数据

getAttribute() :      
// 获取指定的属性值

removeAttribute() :   
// 删除指定属性

setAttribute() :      
// 设置指定的属性值

save(存储区名) :      
// 将对象存储到一个 userData 存储区
```

有了这些理论基础，下面我们可以封装一个兼容所有主流浏览器的本地存储对象了：

```
localData = {
	 hname: location.hostname ? location.hostname : 'uD',
	 isLocalStorage: window.localStorage ? true : false,
	 dom: null,
	 initDom: function(){
	   if(!this.dom){
		 try{
		    this.dom = document.createElement('input');
		    this.dom.type = 'hidden';
		    this.dom.style.display = "none";
		    this.dom.addBehavior('#default#userData');
		    document.body.appendChild(this.dom);
		    var exDate = new Date();
		    exDate = exDate.getDate() + 30;
		    this.dom.expires = exDate.toUTCString();
		  } catch(ex) {
		    return false;
		  }
	   }
	   return true;
	 },
	 set: function(key,value){
		 if(this.isLocalStorage){
			 window.localStorage.setItem(key,value);
		 }else{
			 if(this.initDom()){
				 this.dom.load(this.hname);
				 this.dom.setAttribute(key,value);
				 this.dom.save(this.hname)
			 }
		 }
	 },
	 get: function(key){
		 if(this.isLocalStorage){
			 return window.localStorage.getItem(key);
		 }else{
			 if(this.initDom()){
				 this.dom.load(this.hname);
				 return this.dom.getAttribute(key);
			 }
		 }
	 },
	 remove: function(key){
		 if(this.isLocalStorage){
			 localStorage.removeItem(key);
		 }else{
			 if(this.initDom()){
				 this.dom.load(this.hname);
				 this.dom.removeAttribute(key);
				 this.dom.save(this.hname)
			 }
		 }
	 }
}
```

执行 localData 的初始化函数（initDom）后，就可以读（get）、写（set）、移除（remove）浏览器本地数据了。



参考：
[1] http://javascript.ruanyifeng.com/bom/webstorage.html
[2] http://www.w3school.com.cn/html5/html_5_webstorage.asp
[3] http://www.qdfuns.com/notes/20156/3034409dd4106d3d0803014844bbefb9.html
[4] http://barretlee.com/blog/2013/03/28/cb-javascript-userdata-and-localstorage/