---
title: node+express实现图片上传功能
date: 2017-11-27 13:27:26
tags: node
---

## 概述

本篇文章适用于node刚刚入门的读者。

本篇文章使用node+express实现了一个简单的图片上传功能：用户点击图片上传，会跳转到上传成功页面并展示上传的图片。

<!--more-->

## 前言

一直想找资料入门node，试着一步步实现一个功能，都没有合适的资料。直到看到[https://www.nodebeginner.org/index-zh-cn.html#](https://www.nodebeginner.org/index-zh-cn.html#) ，这本书教你如何**一步一步**结合基本的API搭建一个简单的应用，实现了简单的图片上传功能。我看完之后终于感觉自己**基本入门**node了。文章中有附源码地址，[https://github.com/manuelkiessling/nodebeginner.org/tree/master/code/application](https://github.com/manuelkiessling/nodebeginner.org/tree/master/code/application) ，如果你感觉还没有入门node，可以试试这本书。

当然我的建议是跟着教程一步步修改代码，而不是直接将源码clone下来。

## 问题

文章到后面给出页面展示的html是以`response.write(body);`的方式写的

```
function start(response) {
console.log("Request handler 'start' was called.");

var body = '<html>'+
'<head>'+
'<meta http-equiv="Content-Type" content="text/html; '+
'charset=UTF-8" />'+
'</head>'+
'<body>'+
'<form action="/upload" enctype="multipart/form-data" '+
'method="post">'+
'<input type="file" name="upload" multiple="multiple">'+
'<input type="submit" value="Upload file" />'+
'</form>'+
'</body>'+
'</html>';

response.writeHead(200, {"Content-Type": "text/html"});
response.write(body);
response.end();
}
```
实际应用中肯定不能以这样的方式写html文件，所以接下来就教你用node+express实现同样的功能，使我们的代码看起来更优雅

## node+express实现图片上传功能

### 环境
mac+node(v9.2.0)+express

### 安装express
> express官网:http://www.expressjs.com.cn/

新建文件夹`node-app`，在文件夹下新建`package.json`文件

```
{
  "name": "node-app",
  "version": "0.0.1",
  "dependencies": {
    "express": "^4.16.2",
  }
}
```
运行`npm install`。新建`app.js`，代码如下

```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

var server = app.listen(3000, function () {
    var host = server.address().address;
    var port = server.address().port;

    console.log('Example app listening at http://%s:%s', host, port);
});
```

运行`node app.js`，打开[localhost:3000](localhost:3000)，应用已经跑起来了

### 利用 Express 托管静态文件

下面利用 Express 托管静态文件，在`node-app`下新建文件夹`public`，新建两个html文件

* start.html

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>请上传您的文件</title>
</head>
<body>
<form action="./upload.html" enctype="multipart/form-data" method="get">
    <input type="file" name="upload" multiple="multiple">
    <input type="submit" value="Upload file" />
</form>
</body>
</html>

```

* upload.html

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>上传成功</title>
</head>
<body>
    <h1>上传成功</h1>
    <img  src="/public/test.png"/>
</body>
</html>

```

修改`app.js`，增加`app.use('/public', express.static('public'));`。修改后`app.js`如下
```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.use('/public', express.static('public'));

var server = app.listen(3000, function () {
    var host = server.address().address;
    var port = server.address().port;

    console.log('Example app listening at http://%s:%s', host, port);
});
```
现在，public 目录下面的文件就可以访问了。

> 参考文档：[http://www.expressjs.com.cn/starter/static-files.html](http://www.expressjs.com.cn/starter/static-files.html)

重启node服务，打开
`http://localhost:3000/public/start.html`，选择文件上传之后，页面就会自动跳转到上传成功页面

![](https://user-gold-cdn.xitu.io/2017/11/25/15ff0e22bfb38f51?w=756&h=176&f=png&s=25194)

### 处理上传的图片

使用模块`formidable`处理请求数据。在`package.json`中增加

```
  "dependencies": {
    "express": "^4.16.2",
    "formidable": "^1.1.1"
  }
```
运行`npm install`。

文件上传自然要用到post请求,更改`start.html`，改为`method="post"`

```
<form action="/upload" enctype="multipart/form-data" method="post">
    <input type="file" name="upload" multiple="multiple">
    <input type="submit" value="Upload file" />
</form>

```

处理post请求用到的Express的路由
> 参考 http://www.expressjs.com.cn/starter/basic-routing.html

修改后的`app.js`如下：

```
var express = require('express');
var app = express();
var formidable = require("formidable");
fs = require("fs");

app.get('/', function (req, res) {
    res.send('Hello World!');
});

app.use('/public', express.static('public'));

app.post('/upload', function (req, res) {
    var form = new formidable.IncomingForm();
    console.log("about to parse");
    form.parse(req, function(error, fields, files) {
        console.log("parsing done");
        console.log(files.upload.path);
        fs.writeFileSync("public/test.png", fs.readFileSync(files.upload.path));
        res.redirect("/public/upload.html") ;
    });
});

var server = app.listen(3000, function () {
    var host = server.address().address;
    var port = server.address().port;

    console.log('Example app listening at http://%s:%s', host, port);
});
```

在`public`文件夹下新增`upload.html`,

```
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>上传成功</title>
</head>
<body>
    <h1>上传成功</h1>
    <img  src="/public/test.png"/>
</body>
</html>
```

嗯，大功告成啦。重新启动服务，打开 [http://localhost:3000/public/start.html](http://localhost:3000/public/start.html) 选择一个图片上传，就能看到自己上传的图片了！

源码附上，https://github.com/Lie8466/node-app/tree/node-express

感谢您的阅读！这是我的学习过程，希望对你有所帮助~

## 参考文档

https://www.nodebeginner.org/index-zh-cn.html#


