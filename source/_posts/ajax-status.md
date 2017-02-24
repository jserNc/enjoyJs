---
title: 关于 ajax 请求 status 值为 0
date: 2016-12-19 09:44:25
tags: problem
---

一个ajax请求，onreadystatechange事件会触发5次，分别对应readyState从0到4的变化。例如：值为4表示请求已经完成或者传输过程中出现错误。readyState总共有5个状态值，分别为0~4，每个值代表了不同的含义：

<!-- more -->

> 0 (UNSENT)：初始化，XMLHttpRequest对象还没有完成初始化
> 1 (OPENED)：载入，XMLHttpRequest对象开始发送请求
> 2 (HEADERS_RECEIVED)：载入完成，XMLHttpRequest对象的请求发送完成
> 3 (LOADING)：解析，XMLHttpRequest对象开始读取服务器的响应
> 4 (DONE)：数据传输已完成或传输过程中出现问题（例如无限重定向、跨域等）。

status为表示响应的HTTP状态码。在HTTP1.1协议下，HTTP状态码总共可分为5大类:

> 1xx：信息响应类，表示接收到请求并且继续处理
> 2xx：处理成功响应类，表示动作被成功接收、理解和接受
> 3xx：重定向响应类，为了完成指定的动作，必须接受进一步处理
> 4xx：客户端错误，客户请求包含语法错误或者是不能正确执行
> 5xx：服务端错误，服务器不能正确执行一个正确的请求

一般情况下，对于一个异步的ajax请求，如需响应成功后执行某操作，我们会这样写：

```
function getResource(url){
    var xhr;
    if (window.XMLHttpRequest){
        xhr = new XMLHttpRequest();
    } else {
        //code for IE5, IE6
        xhr = new ActiveXObject("Microsoft.XMLHTTP");
    }
    xhr.onreadystatechange = function (){
        if (xhr.readyState == 4 && xhr.status == 200){
            //doSomething();
        }
    };
    xhr.open('GET', url, true);
    xhr.send(null); 
}
```

即xhr.readyState == 4 && xhr.status == 200需要同时满足，才能表示请求成功！

我们把以上代码稍微改写一下,打印每次ajax请求返回的状态码：

```
function getStatus(url){
    var xhr;
    if (window.XMLHttpRequest){
        xhr = new XMLHttpRequest();
    } else {
        //code for IE5, IE6
        xhr = new ActiveXObject("Microsoft.XMLHTTP");
    }
    xhr.onreadystatechange = function (){
        if (xhr.readyState == 4) {
            console.log('status:',xhr.status);
        }
    };
    xhr.open('GET', url, true);
    xhr.send(null); 
}
```

实际应用中，会发现存在少量用户打印出的xhr.status值为0。这就奇怪了，http状态码也没有值为0的场景，这里的0又是怎么来的？

[官方文档https://www.w3.org/TR/2014/WD-XMLHttpRequest-20140130/](https://www.w3.org/TR/2014/WD-XMLHttpRequest-20140130/) 指出：

status值会由以下几个步骤确定：

> 1.If the state is UNSENT or OPENED, return 0.
> 2.If the error flag is set, return 0.
> 3.Return the HTTP status code.

也就是说，如果readyState是UNSENT（0）或OPENED（1）状态，status值返回0；如果出现某种网络错误或者请求终止，error flag就会由unset变为set，status值返回0；否则，返回http状态码。

例如，我们跨域调取百度的资源或本地文件都会报跨域错误，同时status返回0：

```
getStatus('https://www.baidu.com');
```
控制台会打印：status: 0，同时报出错误：XMLHttpRequest cannot load https://www.baidu.com/. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.nc.com' is therefore not allowed access.

调取本地文件，同样会返回0：

```
getStatus('file:///E:/www/www.nc.com/index.html');
```

控制台会打印：status: 0，同时报出错误：XMLHttpRequest cannot load file:///D:/www/www.nc.com/index.html. Cross origin requests are only supported for protocol schemes: http, data, chrome, chrome-extension, https, chrome-extension-resource.

参考：
[1] https://www.w3.org/TR/2014/WD-XMLHttpRequest-20140130/
[2] http://www.w3school.com.cn/ajax/
[3] http://www.cnblogs.com/liu-fei-fei/p/5618782.html
[4] http://blog.csdn.net/iaiti/article/details/42192659
