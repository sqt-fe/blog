---
title: webpack初识
date: 2017-12-15 10:57:09
tags: webpack
categories: 前端开发
---

## webpack初识
### 1、换个角度看webpack
近年来，前端技术蓬勃发展，我们想在js更方便的实现html , 社区就出现了jsx,我们觉得原生的css不够好用，社区就提出了scss,less，针对前端项目越来越强的模块化开发需求，社区出现了AMD,CommonJS,ES2015 import等等方案。遗憾的是，这些方案大多并不直接被浏览器支持，往往伴随这些方案而生的还有另外一些，让这些新技术应用于浏览器的方案，我们用babel来转换下一代的js，转换jsx；我们用各种工具转换scss,less为css；我们发现项目越来越复杂，代码体积越来越大，又要开始寻找各种优化，压缩，分割方案。前端工程化这个过程，让我们大费精力。我们就在寻找前端模块化解决方案的过程中知晓了webpack。
<!--more-->
### 2、什么是webpack？（官网描述）
![webpack官网图片](https://doc.webpack-china.org/bf093af83ee5548ff10fef24927b7cd2.svg)

大话webpack：
* webpack 是一个现代 JavaScript 应用程序的模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成少量的 bundle - 通常只有一个，由浏览器加载。
* Webpack的工作方式就是把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个浏览器可识别的JavaScript文件。
* webpack是一个模块化打包工具，使用js作为载体将所有静态资源打包在一起，支持的loader和plugin等能对各种静态资源进行预处理，极大地方便了前端的工程化开发。
* webpack是一个打包模块化js的工具，可以通过loader转换文件，通过plugin扩展功能。

## webpack核心
它是高度可配置的，先理解四个核心概念：入口(entry)、输出(output)、loader、插件(plugins)。
### 入口（Entry）
入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。每个依赖项随即被处理，最后输出到称之为 bundles 的文件中；
可以指定一个或多个入口文件；
### 出口（Output）
output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件。你可以通过在配置中指定一个 output 字段，来配置这些处理过程；
### 加载器（Loader）
loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）。loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块，然后你就可以利用 webpack 的打包能力，对它们进行处理。

在 webpack 的配置中 loader 有两个目标:
* 识别出应该被对应的 loader 进行转换的那些文件。(使用 test 属性)
* 转换这些文件，从而使其能够被添加到依赖图中（并且最终添加到 bundle 中）(use 属性)

### 插件（Plugins）
loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。插件接口功能极其强大，可以用来处理各种各样的任务。

想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例。
## 简要分析webpack打包后代码
 1、一个立即执行函数
 
 ` (function(modules){})([function(){},function(){}]);`
 
 2、匿名函数里面干了什么
 
 定义了一个对象：
 `installedModules = {}`

定义了一个函数：`__webpack_require__(moduleId){}`

`return __webpack_require__(0)`

3、__webpack_require__干了什么

```
function __webpack_require__(moduleId) {
    // Check if module is in cache
    if(installedModules[moduleId]) {
        return installedModules[moduleId].exports;
    }
 
    // Create a new module (and put it into the cache)
    var module = installedModules[moduleId] = {
        i: moduleId,
        l: false,
        exports: {}
    };
  
    // Execute the module function
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
  
    // Flag the module as loaded
    module.l = true;
 
    // Return the exports of the module
    return module.exports;
}
```
call能确保当模块中使用this的时候，this总是指向module.exports的：

`modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);`
## 辅助开发技巧
1、简化路径，定义路径别名：

* resolve.alias：创建 import 或 require 的别名，来确保模块引入变得更简单；
* resolve.extensions：自动解析确定的扩展，能够使用户在引入模块的时候不带扩展；

2、webpack中常见的占位符:
* [name]：代表打包后文件的名称，在entry或代码中(之后会看到)确定；
* [id]：webpack给块分配的内部chunk id，如果你没有隐藏，你能在打包后的命令行中看到；
* [hash]：每次构建过程中，生成的唯一 hash 值；
* [chunkhash]： 依据于打包生成文件内容的 hash 值,内容不变，值不变；
* [ext]： 资源扩展名,如js,jsx,png等等；

3、loader还可以实现这些：

* 转换编译：script-loader/babel-loader/ts-loader/coffee-loader等。
* 处理样式：style-loader/css-loader/less-loader/sass-loader/postcss-loader等。
* 处理文件：raw-loader/url-loader/file-loader/等。
* 处理数据：csv-loader/xml-loader等。
* 处理模板语言：html-loader/pug-loader/jade-loader/markdown-loader等。
* 清理和测试：mocha-loader/eslint-loader等。

4、CommonsChunkPlugin 提取公共模块的插件
* `new webpack.optimize.CommonsChunkPlugin(options)`
* 通过将公共模块拆出来，最终合成的文件能够在最开始的时候加载一次，以便存起来到缓存中供后续使用。这个带来速度上的提升，因为浏览器会迅速将公共的代码从缓存中取出来，而不是每次访问一个新页面时，再去加载一个更大的文件。
* 

### 扩展文档：
如何写一个插件：[https://github.com/webpack/docs/wiki/how-to-write-a-plugin](https://github.com/webpack/docs/wiki/how-to-write-a-plugin)