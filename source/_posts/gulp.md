---
title: 前端构建工具gulp
date: 2016-11-17 09:58:52
tags:
---

在前端开发过程中，源代码编写完成后，一般还要经过代码语法检查、文件合并、代码压缩、浏览器刷新查看效果等步骤。gulp就是可以替我们开发者完成这些重复工作的自动化构建工具。换句话说，gulp是一个编译、打包、压缩javascript/coffee/sass/less/html/image/css等文件的工具。类似的工具还有较早的ant、grunt等。

<!-- more -->

讲gulp之前先说一下node。node是JavaScript语言的服务器运行环境，与之相对的是浏览器环境。也就是说，除了浏览器，JavaScript也可以在node环境里运行，node提供了大量的库供JavaScript调用。其实node和chrome浏览器的JavaScript解释器一样，都是google公司的V8引擎。

gulp是基于node的一个工具。gulp借鉴了Unix操作系统的管道（pipe）思想（前一级的输出即是后一级的输入，以下实例中的pipe()方法会看到这种用法），使得其处理过程比较清晰高效。

gulp的使用流程比较简单：首先当然是需要具备node环境；其次我们需要安装gulp（以及gulp所需的插件）；然后编写gulp配置文件gulpfile.js，定义gulp任务；最后通过OS X系统的终端（Terminal）或者windows系统的命令提示符（Command Prompt）运行gulp任务即可。

简单概括一下使用gulp的流程：

### 1. 安装node环境；

详细安装步骤见[node官网](https://nodejs.org/en/)

### 2. 全局安装gulp；

```
npm install -g gulp
```
其中npm是随node一起安装的node模块管理工具，用来安装、卸载、管理node插件（模块）。

### 3. 项目本地安装gulp及gulp插件；

```
npm install --save-dev gulp
```
如果gulpfile.js文件中定义的任务还调用了其他gulp插件，那我们对每个插件都需要安装。例如，我们需要对代码进行压缩，需要安装gulp-uglify插件，执行命令：
```
npm install --save-dev gulp-uglify
```
值得注意的是：虽然前面已经全局安装了gulp，这里在项目本地也是需要局部安装gulp的。

### 4. 编写gulpfile.js文件；

项目根目录中新建一个普通的js文件，命名为gulpfile.js。该文件的作用是定义gulp任务，即我们用gulp来帮我们做什么。

```
gulp.task('taskName', function() {
  // 将你的任务代码放在这
});
```

### 5. 运行gulp任务。

例如，在gulpfile.js中我们定义了一个压缩文件的minify任务。我们在项目目录中执行命令：

```
gulp minify
```
即可调用该压缩任务。

以上即是完整的gulp使用流程。下面我们回过头来看个典型的gulpfile.js文件。

```
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('minify', function () {
    gulp.src('js/app.js')
        .pipe(uglify())
        .pipe(gulp.dest('build'));
});
```

上面代码中，首先加载gulp和gulp-uglify两个模块，需要在流程3中安装。gulp.task()方法用于指定具体任务，第一个参数是任务名称，第二个参数是任务函数或任务数组。在这里任务名是minify，任务函数是一个匿名函数。在这个匿名任务函数中，使用gulp.src()方法指定要处理的文件，然后通过管道函数pipe进行链式处理。这里使用了两次pipe方法，第一次是使用uglify()方法对源文件进行压缩，第二步是gulp.dest()方法将上一步的输出写入文件build.js（代码中可以省略后缀.js）。

然后我们按照流程5，运行minify任务，即可以将js目录下的app.js文件压缩到build.js文件。

gulp模块主要的几个方法如下：

### gulp.task(name[, deps], fn)

定义具体的任务。第一个参数为指定的任务名，第二个参数为任务函数（或者一个任务数组）。如果一个任务的名字为default，就表明它是“默认任务”，在命令行直接输入gulp命令，就会运行该任务。

```
gulp.task('default', function() {
  // 默认的任务代码
});
```
也可以以一个新的任务名指定多个任务。

```
gulp.task('build', ['task1', 'task2', 'task3']);
```

以上定义build任务由task1、task2、task3组成。调用build任务时，会并行触发这三个任务。

### gulp.src(globs[, options])

该方法参数为待处理的源文件名（可以模糊匹配），输出为可读的数据流（stream）。

```
gulp.src(['js/**/*.js', 'css/style.css'])
```
以上指定待处理的文件为：js目录即其所有子目录中所有后缀为js的文件以及css目录下的style.css文件。

### gulp.dest(path[, options])

参数为待写入文件名，将上一级的输出数据流写入该文件。还可以接收第二个参数表示配置对象。

```
gulp.dest('build', {
  cwd: './app',
  mode: '0644'
})
```
配置对象有两个字段。cwd字段指定写入路径的基准目录，默认是当前目录；mode字段指定写入文件的权限，默认是0777。

### gulp.watch(glob [, opts], tasks) 或 gulp.watch(glob [, opts, cb])

监视特定文件，当文件发生改动，就执行指定任务或方法。当glob内容发生改变时，执行tasks

```
gulp.task('watch', function () {
   gulp.watch('js/app.js', ['build']);
});
```

一旦js/app.js发生改动，就执行build任务。

再看个完整的实例gulpfile.js文件：

```
var gulp = require('gulp');
var jshint = require('gulp-jshint');//语法检查
var concat = require('gulp-concat');//合并文件
var uglify = require('gulp-uglify');//压缩代码
var rename = require('gulp-rename');//重命名

// 语法检查
gulp.task('jshint', function () {
    return gulp.src('public/javascripts/*.js')
               .pipe(jshint())
               .pipe(jshint.reporter('default'));
});

// 合并文件之后压缩代码
gulp.task('minify', function (){
    return gulp.src('public/javascripts/*.js')
               .pipe(concat('all.js'))
               .pipe(gulp.dest('public/javascripts/dist'))
               .pipe(uglify())
               .pipe(rename('all.min.js'))
               .pipe(gulp.dest('public/javascripts/dist'));
});

// 监视文件的变化
gulp.task('watch', function () {
    gulp.watch('public/javascripts/*.js', ['jshint', 'minify']);
});

// 注册缺省任务
gulp.task('default', ['jshint', 'minify', 'watch']);
```


参考：
[1] http://javascript.ruanyifeng.com/tool/gulp.html
[2] http://javascript.ruanyifeng.com/tool/grunt.html
[3] http://www.gulpjs.com.cn/
[4] http://www.ydcss.com/archives/18
[5] http://javascript.ruanyifeng.com/nodejs/basic.html
[6] https://www.zhihu.com/question/30597893?sort=created
[7] http://blog.sina.com.cn/s/blog_6592d8070102vmuq.html
