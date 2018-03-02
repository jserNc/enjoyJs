---
title: windows 批处理脚本修改文件
date: 2016-12-22 11:46:32
tags: method
---

有时候，我们可能需要在一个文件的基础上复制出很多新文件，并针对每个文件做出指定的修改。若不想手工重复地处理这些操作，可以用批处理脚本来帮我们完成这些操作。当然了，如果这批文件只生成一次，编辑批处理脚本所花的时间相对手工一个个编辑文件并没有多大优势，但是若需要多次生成这批文件，那脚本的优势就很明显了。

<!-- more -->

下面以批处理文件 tg.cmd 为例来说明怎么用批处理脚本来做这些操作：

```
:: 新建文件夹
md middleFile

:: 复制中间文件
copy tg.htm middleFile\1.htm

:: 文件末尾插入 cnzz 代码
echo ^<span style="display:none"^>^<script src="//s95.cnzz.com/z_stat.php?id=1261027332&web_id=1261027332" language="JavaScript"^>^</script^>^</span^> >> middleFile\1.htm

:: 新建文件夹
md tgFile

:: 用 sed s 命令替换文本，把字符串 www.baidu.com/?36505 替换为 www.baidu.com/?37095-0001
sed "s/www.baidu.com\/?36505/www.baidu.com\/?37095-0001/" middleFile\1.htm>tgFile/tg37095-0001.htm

:: 删除过渡文件
rd /s/q middleFile 
```

其中，:: 为 cmd 文件注释，echo 命令中 <、> 等符号均需用 ^ 转义；

以上除了 sed 命令，其余均为基本的 cmd 命令。** sed 全名叫 stream editor，流编辑器，用程序的方式来编辑文本。sed 基本上就是正则模式匹配，所以，使用 sed，对正则表达式要求比较高 **。

要在 cmd 中使用 sed 命令，我们需要先进行一个简单的配置操作：
> ① 下载 [sed for Windows](http://gnuwin32.sourceforge.net/packages/sed.htm) 。下载 Binaries zip 文件，解压后在 bin 目录找到 sed.exe 文件;
> ② 下载 [libintl3.dll](http://gnuwin32.sourceforge.net/packages/libintl.htm) 、 [libiconv2.dll](http://gnuwin32.sourceforge.net/packages/libiconv.htm) 、[regex2.dll](http://gnuwin32.sourceforge.net/packages/regex.htm) ，操作同上；
> ③ 将 sed.exe 复制到 C:\Windows\System32；
> ④ 将 libintl3.dll、libiconv2.dll、regex2.dll 等 3 个 dll 文件复制到 C:\Windows\SysWOW64。

sed 有很多有用的命令，以上我们只用到 s 命令进行字符串替换。

s 命令主要用法：

```
sed "s/str1/str2/g" a.txt
```

s 表示替换命令，将 a.txt 中的字符串 “str1” 替换为 “str2”，/g 表示一行上的替换所有的匹配（上面 tg.cmd 例子中，我们字符串中包含 / 字符，所以用到 \/ 进行转义）。


参考： 
[1] http://coolshell.cn/articles/9104.html
[2] https://dotblogs.com.tw/jses88001/2014/11/13/147283
[3] http://gnuwin32.sourceforge.net/packages/sed.htm


    



