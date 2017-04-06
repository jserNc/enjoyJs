---
title: CSRF 攻击原理
date: 2017-04-06 09:40:53
tags: problem
---

CSRF : 跨站请求伪造（Cross Site Request Forgery）。攻击者冒充用户身份，以用户的名义做出各种操作，而这些操作用户未必知道和愿意做。比如，以用户的名义登录注销、发送消息、购买商品……

<!-- more -->

网站一般通过 cookie 来标识用户。用户在网站上登录成功后浏览器会得到一个标识其身份的 cookie，在当前登录会话期间（不关闭浏览器且不退出登录），用户再打开这个网站的页面都会带上这个身份标识 cookie。如果，在这个有效会话期间，攻击者控制用户浏览器访问了这个网站的 url，就可能执行一些用户不想做的操作（比如退出登录，修改会员资料……），由于这些操作并不是用户自己真正想做的，这就是请求伪造。

**场景一：**

某个 bbs 可以贴图，攻击者在贴图的 url 中写入的是退出登录的链接。 当后续用户浏览到这个帖子时，就会退出登录了，因为用户是以自己的身份访问了这个退出登录的链接，虽然他自己可能并不知道怎么回事。在用户看来，帖子里有一张图片（其实是“有问题”的图片），并没有想要退出登录，但是网页程序会认为用户要求退出登录，从而销毁其登录会话。

**CSRF 攻击流程如下：**

> ① 用户登录漏洞网站 A，用户浏览器存储身份 cookie；
> ② 用户没有退出登录 A 网站的情况下，被攻击者利用，使其向 A 网站发出了请求；
> ③ A 网站并不知道请求是用户主动还是被动发出的，由于请求带有身份标识 cookie，所以 A 网站还是会根据用户的权限来处理这个请求。

那么问题的关键就是攻击者如何让用户发出请求。上面场景一说明了其中一种请求伪造方式。下面再看一个场景。

**场景二：**

银行网站 A，它以 get 请求来完成转账操作，地址为：

```
http://A.com/t.php?toId=id&money=num
```

攻击者网站 B，里面有一段代码：

```
<img src = "http://A.com/t.php?toId=11&money=1000">
```

如果你登录了 A 网站后，然后访问网站 B，然后你的账户就少了 1000 块……

因为你在 A 网站登录状态下访问网站 B，网站 B 中有指向 A 网站的图片资源，这使得你又向 A 网站发出了请求（带有你的登录身份标识），A 网站受到这个请求后，发现是一个转账请求，于是，便在你的账户上划走了 1000 块。

以上 get 请求来更新资源是极不安全的，为了防止以上问题，银行改用 post 请求来执行转账操作。银行网站 A 的表单代码：

```
<form action="t.php" method="POST">
　　<p>ToBankId: <input type="text" name="toId" /></p>
　　<p>Money: <input type="text" name="money" /></p>
　　<p><input type="submit" value="转账" /></p>
</form>
```

t.php 代码：

```
<?php
　　session_start();
　　if (isset($_POST['toId'] && isset($_POST['money'])){
　　　　buy_stocks($_POST['toId'],$_POST['money']);
　　}
?>
```

B 网站也更新了代码：

```
<html>
　　<head>
　　　　<script type="text/javascript">
　　　　　　function steal(){
          　　　iframe = document.frames["steal"];
　　     　　   iframe.document.Submit("t");
　　　　　　}
　　　　</script>
　　</head>

　<body onload="steal()">
　　<iframe name="steal" display="none">
　　 <form method="post" name="t" action="A.com/t.php">
　　　　<input type="hidden" name="toId" value="11">
　　　　<input type="hidden" name="money" value="1000">
　　 </form>
　　</iframe>
　</body>
</html>
```

同样操作，还是被转走 1000 块……

> **以上 CSRF 攻击的根本原因：WEB的身份验证机制虽然可以保证一个请求是来自于用户的浏览器（用户身份），但却无法保证该请求是用户主动发送的！**

**CSRF 防御：**

鉴于以上攻击手段的特点，从用户和网站开发人员角度都可以做一些事情来降低受到 CSRF 攻击的可能性。

**（1）从用户角度：**

· 在使用完某网站之后，应该立即退出登录；
· 尽量不要让浏览器记住用户名密码；
· 陌生的链接不要轻易去点，尤其是邮件里的链接。

**（2）从开发人员角度：**

· url 添加会话相关信息，而不是仅仅将 cookie 作为会话唯一标识；
· 尽量不要是用 get 等明文请求；
· 用户提交表单时加入验证码、伪随机值等。




参考：
[1] https://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html
[2] http://netsecurity.51cto.com/art/200811/97281.htm
[3] https://blog.tonyseek.com/post/introduce-to-xss-and-csrf/
