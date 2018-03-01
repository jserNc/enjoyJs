---
title: 前端构建工具 gulp
date: 2016-11-17 09:58:52
tags: tools
---

**前端开发过程中，源代码编写完成后，一般还要经过代码语法检查、文件合并、代码压缩、浏览器刷新查看效果等步骤。**gulp 就是可以替我们开发者完成这些重复工作的自动化构建工具之一。换句话说，gulp 是一个编译、打包、压缩 javascript/coffee/sass/less/html/image/css 等文件的工具。类似的工具还有较早的 ant、grunt 等。

<!-- more -->

讲 gulp 之前先说一下 node。node 是 JavaScript 语言的服务器运行环境，与之相对的是浏览器环境。也就是说，除了浏览器，JavaScript 也可以在 node 环境里运行，node 提供了大量的库供 JavaScript 调用。node 和 chrome 浏览器的 JavaScript 解释器一样，都是 google 公司的 V8 引擎。

gulp 是基于 node 的一个工具。**gulp 借鉴了 Unix 操作系统的管道（pipe）思想（前一级的输出即是后一级的输入，以下实例中的 pipe() 方法会体现这种思想），其处理过程比较清晰高效。**

gulp 的使用流程比较简单：首先需要具备 node 环境；其次我们需要安装 gulp（以及 gulp 所需的插件）；然后编写 gulp 配置文件 gulpfile.js，定义 gulp 任务；最后通过 OS X 系统的终端（Terminal）或者 windows 系统的命令提示符（Command Prompt）即可运行gulp任务。

下面简单概括一下 gulp 的使用流程：

### 1. 安装node环境；

详细安装步骤见[ node 官网](https://nodejs.org/en/)

### 2. 全局安装 gulp；

```
npm install -g gulp
```
其中 npm 是随 node 一起安装的 node 模块管理工具，用来安装、卸载、管理 node 插件（模块）。

### 3. 项目本地安装 gulp 及 gulp 插件；

```
# 进入项目目录
# 确定已经有 package.json，没有就通过 npm init 创建
# 安装 gulp 依赖
npm install --save-dev gulp
```

如果 gulpfile.js 文件还调用了其他 gulp 插件，那我们需要安装每个插件。例如，如果需要对代码进行压缩，就安装 gulp-uglify 插件，执行命令：

```
npm install --save-dev gulp-uglify
```

值得注意的是：虽然前面已经全局安装了 gulp，这里在项目本地也是需要局部安装 gulp 的。

### 4. 编写 gulpfile.js 文件；

在项目根目录中新建一个普通的 js 文件，命名为 gulpfile.js。该文件的作用是定义 gulp 任务，即定义我们用 gulp 来做什么。大体格式如下：

```
gulp.task('taskName', function() {
  // 将你的任务代码放在这
});
```

taskName 是我们定义的任务名，其后的匿名函数，定义该任务的执行代码。

### 5. 运行 gulp 任务。

例如，在 gulpfile.js 中我们定义了一个压缩文件的 minify 任务。我们在项目目录中执行命令：

```
gulp minify
```
即可调用该压缩任务。

以上即是完整的 gulp 使用流程。下面我们回过头来看个典型的 gulpfile.js 文件。

```
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('minify', function () {
    gulp.src('js/app.js')
        .pipe(uglify())
        .pipe(gulp.dest('build'));
});
```

上面代码中，首先加载 gulp 和 gulp-uglify 两个模块（上述流程 3 中安装）。gulp.task() 方法用于指定具体任务，第一个参数是任务名称，第二个参数是任务函数或任务数组。在这里自定义的任务名是 minify，任务函数是一个匿名函数。在这个匿名任务函数中，使用 gulp.src() 方法指定要处理的文件，然后通过管道函数 pipe 进行链式处理。这里使用了两次 pipe 方法，第一次是使用 uglify() 方法对源文件进行压缩，第二步是 gulp.dest() 方法将上一步的输出写入到 build 目录下的同名压缩文件。

然后我们按照流程 5，运行 minify 任务，即可以将 js 目录下的 app.js 文件压缩到 build.js 文件。

** gulp 模块主要的几个方法如下：**

> ### gulp.task(name[, deps], fn)

定义具体的任务。第一个参数为自定义的任务名，第二个参数为任务函数（或者一个任务数组）。如果一个任务的名字为 default，就表明它是“默认任务”，在命令行直接输入 gulp 命令（后面不需要跟任务名），就可以运行该任务。

```
gulp.task('default', function() {
  // 默认的任务代码
});
```
也可以以一个新的任务名指定多个任务。

```
gulp.task('build', ['task1', 'task2', 'task3']);
```

以上定义 build 任务由 task1、task2、task3 组成。执行 build 任务时，会并行触发这三个任务。

> ### gulp.src(globs[, options])

该方法参数为待处理的源文件名（可以模糊匹配），输出为可读的数据流（stream）。

```
gulp.src(['js/**/*.js', 'css/style.css'])
```
以上指定待处理的源文件为：js 目录及其所有子目录中所有后缀为 .js 的文件以及 css 目录下的 style.css 文件。

> ### gulp.dest(path[, options])

参数为待写入文件名，将上一级的输出数据流写入该文件。还可以接收第二个参数表示配置对象。

```
gulp.dest('build', {
  cwd: './app',
  mode: '0644'  
  // 用户具有读写权限，组用户和其它用户具有只读权限；
})
```
配置对象有两个字段。cwd 字段指定写入路径的基准目录，默认是当前目录；mode 字段指定写入文件的权限，默认是 0777（用户可读可写可执行）。

> ### gulp.watch(glob [, opts], tasks) 或 gulp.watch(glob [, opts, cb])

监视特定文件，当文件发生改动，就执行指定任务或方法。当 glob 内容发生改变时，执行 tasks：

```
gulp.task('watch', function () {
   gulp.watch('js/app.js', ['build']);
});
```

一旦 js/app.js 发生改动，就执行 build 任务。

再看个完整的实例 gulpfile.js 文件：

```
var gulp = require('gulp');
var jshint = require('gulp-jshint'); // 语法检查
var concat = require('gulp-concat'); // 合并文件
var uglify = require('gulp-uglify'); // 压缩代码
var rename = require('gulp-rename'); // 重命名

// 语法检查
gulp.task('jshint', function () {
    return gulp.src('js/*.js')
               .pipe(jshint())
               .pipe(jshint.reporter('default'));
});

// 合并，然后压缩，最后重命名
gulp.task('minify', function (){
    return gulp.src('js/*.js')
               .pipe(concat('all.js'))
               .pipe(gulp.dest('js/dist'))
               .pipe(uglify())
               .pipe(rename('all.min.js'))
               .pipe(gulp.dest('js/dist'));
});

// 监视文件的变化
gulp.task('watch', function () {
    gulp.watch('js/*.js', ['jshint', 'minify']);
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
