---
title: 前端构建工具 grunt
date: 2017-06-29 14:57:49
tags: tools
---

同样作为前端构建工具，grunt 和 gulp 的功能很类似，都可以帮助我们开发者进行代码检查、编译、单元测试、文件合并、压缩等任务。它们主要区别在于，grunt 在多个处理流程过程中会有多次 I/O 操作，并生成多个中间文件；而 gulp 只需要一次 I/O 操作，随后都是文件流式处理，效率较高。不过，grunt 生态系统非常庞大，有很多插件可供选择，jQuery、Bootstrap 等优秀项目都在使用 grunt，所以还是很有必要学习 grunt 的。

<!-- more -->

关于 grunt 任务，插件等有很多内容可以深入研究的，这里着重介绍一下 grunt 的使用流程：

### 1. 安装node环境；

详细安装步骤见[ node 官网](https://nodejs.org/en/)

### 2. 安装 cli

将 grunt 命令行（cli）安装到全局环境中，例如，windows 下执行命令：

```
npm install -g grunt-cli
```

这样，grunt 命令就加入到系统路径中了，随后我们就可以在任何项目目录中执行 grunt 命令来构建项目了。

### 3. 项目本地安装 grunt 及 grunt 插件；

```
# 进入项目目录
# 确定已经有 package.json，没有就通过 npm init 创建

# 安装 grunt 依赖
npm install grunt --save-dev
```

如果配置文件 Gruntfile.js 文件中还调用了其他 grunt 插件，那我们需要依次安装每个插件。例如，我们需要对代码进行检查，就安装 jshint 插件，执行命令：

```
npm install grunt-contrib-jshint --save-dev
```

### 4. 编写 Gruntfile.js 文件；

在项目根目录中新建一个普通的 js 文件，命名为 Gruntfile.js。该文件的作用是定义 grunt 任务，即定义我们用 grunt 来做什么。大体格式如下：

```
module.exports = function(grunt) {
    grunt.initConfig({
        // 定义任务
    });

    // 加载插件，如：
    grunt.loadNpmTasks('grunt-contrib-jshint');

    // 定义默认任务，如：
    grunt.registerTask('default', ['jshint']);
};
```

简单来说，如果我们在这个文件中只定义了“文件检查”任务，那么执行 grunt 命令时，grunt 就会帮我们检查指定的代码是否有错误；如果我们在这个文件中定义了“代码检查”，“单元测试”、“文件压缩”等一系列任务，那么执行 grunt 命令时，grunt 就帮我们依次执行以上任务。

典型的 Gruntfile.js 代码如下：

```
module.exports = function(grunt) {
  // 任务配置
  grunt.initConfig({
    // 读取 package.json 文件
    pkg: grunt.file.readJSON('package.json'),
    jshint: {
        // 待处理文件
        files: ['Gruntfile.js','src/**/*.js','test/**/*.js'],
        options: {
            globals: {
                jQuery: true,
                console: true,
                module: true,
                document: true
            }
        }
    }
  });

  // 加载包含 "jshint" 任务的插件。
  grunt.loadNpmTasks('grunt-contrib-jshint');

  // 默认被执行的任务列表。
  grunt.registerTask('default', ['jshint']);
};
```

最后，执行 grunt 命令即可：

```
grunt
```




参考：
[1] https://segmentfault.com/a/1190000008443074
[2] http://www.gruntjs.net/