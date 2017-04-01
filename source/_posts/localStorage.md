---
title: 浏览器本地存储（一）
date: 2017-03-31 14:18:14
tags: method
---

有时候需要将一些数据存储在用户浏览器本地，下次用户访问我们的页面的时候，我们就能根据存储在用户浏览器的本地数据，做一些针对当前用户的个性化操作。例如，用户访问我们页面的时候，也许会填写用户名，可以将用户名（userName）存储在用户的浏览器中，下次用户再访问我们页面的时候，弹出一个“欢迎你，{userName}” 的欢迎词。

<!-- more -->

这里主要讨论一下 cookie 和 localStorage 两种本地存储方式。

## cookie：

cookie 原意小甜点。这里指保存在用户浏览器中的一段文本信息。每个 cookie 大小一般不能超过 4KB。每个域名下最多只能存储几十个 cookie，具体个数因浏览器而异。

打开 chrome 开发者工具 Application 标签下的 Cookies 列表，可以看到所有 cookie 按域名分类。每个 cookie 包含 cookie 名（**Name**）、cookie 值（**Value**）、有效时间（**Expires/Max-Age**）、所属域名（**Domain**）、有效路径（**Path**）、安全（**Secure**）、是否能被 JavaScript 读取（**HttpOnly**）等属性。

**(1) 读取 cookie：（一次可以读出全部）**

document.cookie 属性会返回当前页面的所有 cookie，并以分号（;）隔开。

```
document.cookie
// "lc=58362; uUiD=88888; game_played=1; mail_index=2;"
```

**如需读取特定名称的 cookie，比如取出名为 game_played 的 cookie 的值，这里给出两种参考方法（其中的解码函数需要和设置 cookie 是用的编码函数对应）：**

**方法一：**
split 以上字符串，得到 cookie 数组，然后 split 该数组各个元素，循环匹配出对应的 value。

```
function getCookie(_name){
    var h = document.cookie.split('; '),
      g = h.length,
      e = [];

    for (var j = 0; j < g; j++) {
      e = h[j].split('=');
      if (_name === e[0] && e[1]) {
          try{
            return decodeURIComponent(e[1]);
          }catch(e){
            return unescape(e[1]);
          }
      }
    }
    return null;
}
```

**方法二：**
检查 document.cookie 对象中是否存有 cookie。如果存在，那么继续检查我们指定的 cookie 是否已储存。如果找到了我们要的 cookie，就返回值，否则返回空字符串。

```
function getCookie(c_name){
    var cookies = document.cookie;
    if (cookies.length>0){
        idx = cookies.indexOf(c_name + "=")
        if (idx != -1){ 
            idx = idx + c_name.length+1; 
            end = cookies.indexOf(";",idx);
            if (end == -1){
                end = cookies.length;
            }
            return unescape(cookies.substring(idx,end));
        } 
    }
    return ""
}
```

**(2) 写入 cookie：（一次只能写入 1 个）**

document.cookie = "name=value[; expires=date][; domain=domain][; path=path][; secure]"

```
document.cookie = "name=nanc";
```

以上属性字段，除了 name 和 value 是必须的，其他的都可选（如不设置，则分别取其默认值）。例如，如果不设置过期时间，则该 cookie 只在当前会话（session）有效，一旦浏览器关闭，这个 cookie 就会失效。

这种设置 cookie 的写法看似会覆盖原来的所有 cookie，其实不然，只是添加一个“新”的 cookie 而已。“新”字加引号，是因为未必真的是新的，也可能只是覆盖原先的同名 cookie。**如果这里设置的 cookie 的 name、domain、path 和 secure 等四个属性都和先前已经存在的某个 cookie 一样，则会覆盖原先同名的 cookie，更新该 cookie 值。**

```
document.cookie = 'name=nanc; '
            + 'expires=' + someDate.toGMTString() + '; '
            + 'path=/subdirectory; '
            + 'domain=*.example.com';
```
封装成与上面读取 cookie 对应的设置 cookie 方法：

**方法一：**

```
function setCookie(_para){
    var _last = new Date();
    var _name = _para.name,
        _val = _para.val,
        _exps = _para.exps,
        _domain = _para.domain,
        _path = _para.path || "/",
        _secure= _para.secure;
    _last.setDate(_last.getDate()+_exps);
    var _cookieVal = _name+"="+escape(_val)
        +(_exps?";expires="
        +_last.toUTCString():"")
        +(";path="+_path)
        +(";domain="+_domain)
        +(";secure="+_secure);
    document.cookie = _cookieVal;
}
```

**方法二：**

```
function setCookie(c_name,value,expTime){
    var exp = new Date();
    exp.setDate(exp.getDate() + expTime);
    var t = exp.toGMTString();
    document.cookie = c_name+ "=" 
        +escape(value)
        +((expTime == null) ? "" : ";expires=" + t)
}
```

**(3) 清除 cookie**

设置 cookie 的过期时间 expires 为 0，或者等于一个过去的日期，使之失效。

**(4) cookie 个数限制**

由于浏览器对 cookie 的个数又限制，每个域最多可以设置几十个。为了规避个数的限制，我们可以用一个 cookie 来存储多个键值对，如：name=a=b&c=d&e=f&g=h 这样实际上只设置 1 个 cookie，cookie 值内部使用 & 符号，隔开对个键值对。读取这个 cookie 的时候，我们自行解析，得到各个键值对。

**(5) 同源政策**

同源政策规定，两个网址只要域名和端口相同就可以共享 cookie，也即是说，它不要求协议相同，同域下 http 和 https 共享 cookie。即 http://a.com 下设置的 cookie，可以被 https://a.com 读取。

**(5) 无法被 JavaScript  读取的 cookie**

服务器端设置 cookie的时候，如果加上了 HttpOnly 属性，则这个 cookie 无法被 JavaScript 读取。即 document.cookie 不会返回这个 cookie 的值，但是这个 cookie 会跟随 http 请求发送至服务端。

先到这里，下篇继续这个话题。


参考：
[1] http://www.w3school.com.cn/js/js_cookies.asp
[2] http://javascript.ruanyifeng.com/bom/cookie.html
[3] http://jerryzou.com/posts/cookie-and-web-storage/