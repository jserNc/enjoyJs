---
title: 浏览器缓存
date: 2017-03-28 16:34:54
tags: problem
---

对于一些较少变化的静态资源文件，如 js、css、image 等，每次刷新页面都去服务器重新请求，这会加大带宽、增加服务器负载以及降低页面显示速度。比较好的做法是，在第一次请求这些文件后，浏览器会将这些文件存放在用户本地，下次需要这些文件的时候直接在用户浏览器缓存中去读取。

<!-- more -->

** 那么问题来了，一旦服务器上文件更新了，浏览器怎么知道不能再继续使用缓存的本地文件，而是要去重新请求服务器上的文件呢？**

为了解决这个问题，这里有一个**缓存协商**的概念：浏览器在请求某个文件的时候，服务器在返回文件时会在返回头加一些额外的信息，比如这个文件在多长时间内可以使用、文件的最后修改时间是什么等等。这样浏览器在下次再请求这个文件的时候就会根据之前服务器在头部加入的额外信息来进行一些判断。

## 下面看几个典型的头信息：

 ** Cache-Control **

取值范围：
public     可以被任何客户端缓存
private    可以被非共享缓存所缓存
no-cache   不可以被任何客户端缓存
max-age    缓存经过该值限定的时间后就会失效

** Last-modified **

Last-modified 为文件最后修改时间。

在浏览器第一次请求页面的时候如果返回头中有 last-modified 这个值，则下一次请求头中会加入 if-modified-since，服务器端取到这个值后去判断和该文件的最后修改时间是否一样，如果一样则返回 304 (not modified)，通知浏览器可以使用本地缓存。

** Expires **

过期时间。在这个过期时间之前，不会再去向浏览器发任何请求，直接使用本地缓存。

** E-tag **

E-tag为“被请求变量的实体标记”，由文件的索引节、 文件大小、最后修改时间等参数生成。

在浏览器第一次请求页面的时候如果返回头中有 E-tag，则会在下次请求头中加入 if-none-match 这个值，服务器取到这个值后去和这个文件生成的 E-tag 值进行对比，如果一样则返回 304 (not modified)，通知浏览器可以使用本地缓存。

** 以下几点需注意： **

① ** Cache-control 和 Expires **

Expires = 时间（绝对时间），HTTP 1.0 版本；
Max-age = 秒（相对时间），HTTP 1.1 版本；

Expires 的缺点：返回的到期时间是服务器时间，那么，如果客户端的时间与服务器时间相差很大，误差就很大。

如果同时存在 Cache-control: max-age 和 Expires，以前者为准。

② ** Last-modified 和 E-tag **

Last-Modified 的不足之处在于：Last-Modified 标注的最后修改只能精确到秒级，如果某些文件在 1 秒钟以内被修改多次的话，它将不能准确标注文件的修改时间。如果某些文件会被定期生成，有时文件内容并没有任何变化，但 Last-Modified 却改变了，可能导致文件没法使用缓存。

该两项参数将存储在客户端的浏览器 cache 中，Last-Modified 值存储为 If-Modified-Since，ETag 值存储为 If-None-Match。

如果同时存在 Last-modified 和 E-tag，以 E-tag 的值为准。

## 关于过期时间，在 Apache 服务器上，可以在 httpd.conf 文件中作如下配置：

```
# 设置缓存
<IfModule mod_expires.c> 
  ExpiresActive on 
  ExpiresDefault "access plus 1 days" 
  ExpiresByType text/html "access plus 1 days" 
  ExpiresByType text/css "access plus 2 days" 
  ExpiresByType image/jpg "access plus 6 days" 
  ExpiresByType image/png "access plus 7 days" 
  ExpiresByType video/x-flv "access plus 10 days"
</IfModule>
```

除了以上根据文件类型设置缓存时间外，还可以根据文件扩展名或特定文件名设置缓存时间。

## 浏览器不同的操作对应的不同请求：

① **回退**
   不论缓存过期与否，都使用本地缓存；

② **新打开浏览器窗口**
   未过期的使用本地缓存，不向服务器发送请求；

③ **地址栏单击回车**
   未过期的使用本地缓存，不向服务器发送请求（除了文档以外的资源）；

④ **F5**
   所有文件都会发送请求，有缓存的进行缓存协商，没有修改的返回 304 使用本地缓存；

⑤ **Ctrl+F5**
   所有文件都会发送请求，并且不进行缓存协商，每个文件都重新返回（200）。

## 各种操作对应的头信息有效性如下：

| 用户操作 | Expires/cache-control | Last-modified/E-tag |
| ------------- |:-------------:| -----:|
| 地址栏回车      | 有效 | 有效 |
| 页面链接跳转    | 有效 | 有效 |
| 新开窗口        | 有效 | 有效 |
| 前进、后退      | 有效 | 有效 |
| F5 刷新         | **无效** | 有效 |
| ctrl + F5 刷新  | **无效** | **无效** |

## 状态码：

**第一层：200(from cache)**
这一层由 expires/cache-control 控制。expires 是绝对时间，cache-control 是相对时间。两者并存时，后者覆盖前者。只要没过期，浏览器只访问自己的缓存。

**第二层：304**
这一层由 last-modified/E-tag 控制。当第一层失效或用户点击 [刷新]/F5 时，浏览器就发送请求给服务器，如果服务端没有变化，则返回 304 给浏览器。

**第三层：200**
当浏览器本地没有缓存或者上一层失效，或者用户点击 ctrl+F5 时，浏览器就会去服务器下载最新数据。

浏览器发起的资源请求，如果使用了缓存基本上是两种情况: **status code: 200 ok ( from cache )** 和 **status code 304 Not Modified**。

前者是不向浏览器发送请求，直接使用本地缓存文件。后者，浏览器虽然发现了本地有该资源的缓存，但是不确定是否是最新的，于是想服务器询问，若服务器认为浏览器的缓存版本还可用，那么便会返回 304。即客户端有缓存的文档并发出了一个条件性的请求（一般是提供 If-Modified-Since 头表示客户只想要比指定日期更新的文档）。服务器告诉客户，原来缓存的文档还可以继续使用。

## 总结：

① cache-control 的值覆盖其他的设置，如 expires；

② E-tag 和 Last-modified 共存的时候以 E-tag 为准；

③ F5 刷新的时候不论缓存是否过期，都会缓存协商（带上 E-tag、Last-modified 之类的信息）；

④ ctrl+F5 强制刷新的时候由浏览器单方面决定不使用缓存；

⑤ 在后退的时候除非显式声明 no-cache，否则不论是否过期，都使用缓存。
























