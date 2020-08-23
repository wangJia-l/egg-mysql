<!--
 * @Descripttion: nunjucks
 * @version: 1.0.0
 * @Author: 王佳
 * @Date: 2020-08-23 20:05:35
 * @LastEditors: sueRimn
 * @LastEditTime: 2020-08-23 20:28:07
-->
> Nunjucks是Mozilla开发的一个纯JavaScript编写的模板引擎，既可以用在Node环境下，又可以运行在浏览器端

### 特点
- 功能丰富且强大，并支持块级继承（block inheritance）、自动转义、宏（macro）、异步控制等等。完美继承了 [jinja2](http://jinja.pocoo.org/ "悬停显示") 的衣钵。
- 快速 & 干练 并且高效。运行时代码经过压缩之后只有 8K 大小， 可在浏览器端执行预编译模板。
- 可扩展 性超强，用户可以自定义过滤器（filter）和扩展模块。
- 到处 可运行，无论是 node 还是任何浏览器都支持，并且还可以预编译模板。

#### 安装：
```
yarn add nunjucks
```

#### 使用&&案例：
##### 渲染字符串
```
let nunjucks = require("nunjucks");
nunjucks.configure({ autoescape: true }); // 自动转译
let result = nunjucks.renderString(`hello {{name}}`, { name: "nunjucks" });
console.log(result) // hello nunjucks
```
PS：在默认情况下，nunjuck 渲染时会按原样输出，如果开启了自动转义 (autoescaping)，nunjuck 会转义所有的输出，为了安全建议一直开启。
##### 渲染文件
- 渲染文件.js
```
let nunjucks = require("nunjucks");
let path = require("path");
nunjucks.configure(path.resolve("view"), { autoescape: true }); // 参数1：视图所在的路径
nunjucks.render("index.html", { name: "nunjucks" });
```
PS：view是一个文件夹，里边装载着模版文件
- view\index.html
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    hello {{name}}
  </body>
</html>
```
**>> 执行结果：**
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    hello nunjucks
  </body>
</html>
```

##### 集成到 express
```
let nunjucks = require("nunjucks");
let path = require("path");
let express = require("express");
let app = express();

nunjucks.configure("view", {
  autoescape: true,
  express: app, // 通过这个属性就实现了 nunjucks 和 express 的关联
});
app.get("/", (req, res) => {
  res.render("index.html", { name: "nunjucks" });
});
app.listen(8080);
```
**内部实现机制：（response.render方法是express内部实现的）**
1. 读取模版文件，把模版文件和数据对象作为参数传递给nunjucks模版引擎
2. 由nunjucks模版引擎渲染出来最终的字符串，再由response发送给客户端

##### 过滤器
> 过滤器是一些可以执行变量的函数，通过管道操作符 (|) 调用，并可接受参数。

Nunjucks 提供了一些[内置的过滤器](https://nunjucks.bootcss.com/templating.html#part-cda1d805a3577fa5)，你也可以[自定义过滤器](https://nunjucks.bootcss.com/api#custom-filters)。
```
let nunjucks = require("nunjucks");
nunjucks.configure({ autoescape: true });
let result = nunjucks.renderString(`hello {{names | join('-')}}`, { names: ["a", "b", "c", "d"] });
console.log(result); // hello a-b-c-d
```




##### if-else
> if 为分支语句，与 javascript 中的 if 类似
```
let nunjucks = require("nunjucks");
nunjucks.configure({ autoescape: true });
let result = nunjucks.renderString(
`
{% if score >= 85 %}
优秀
{% elseif score >= 60 %}
及格
{% else %}
不及格
{% endif %}
`,
  { score: 90 }
);
console.log(result); // 优秀
```
##### for
```
let nunjucks = require("nunjucks");
nunjucks.configure({ autoescape: true });
let result = nunjucks.renderString(
  `
  <ul>
    {% for user in users %}
        <li data-id="{{user.id}}">{{loop.index}}:{{user.name}}</li>
    {% endfor %}
  </ul>
`,
  {
    users: [
      { id: 1, name: "王二" },
      { id: 2, name: "李三" },
    ],
  }
);
console.log(result);
```
>> 结果
```
<ul>
  <li data-id="1">1:王二</li>
	<li data-id="2">2:李三</li>
</ul>
```

**在循环中可获取一些特殊的变量:**

- loop.index: 当前循环数 (1 indexed)

- loop.index0: 当前循环数 (0 indexed)

- oop.revindex: 当前循环数，从后往前 (1 indexed)

- loop.revindex0: 当前循环数，从后往前 (0 based)

- loop.first: 是否第一个

- loop.last: 是否最后一个

- loop.length: 总数

##### 继承
>  模板继承可以达到模板复用的效果，当写一个模板的时候可以定义 "blocks"，子模板可以覆盖他
> 同时支持多层继承
- extends.js
```
let nunjucks = require("nunjucks");
let path = require("path");
nunjucks.configure(path.resolve("view"), { autoescape: true });
let result = nunjucks.render("login.html", { name: "nunjucks" });
console.log(result);
```
- view/index.html
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    头部
    {% block content %}
      我是index模板的内容name= {{name}}
    {% endblock %}
    尾部
  </body>
</html>
```
- view/login.html
```
{% extends "index.html" %}
{% block content %}
我是login模板的内容name= {{name}}
{% endblock %}
```

>> 结果
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    头部
    
我是login模板的内容name= nunjucks

    尾部
  </body>
</html>
```
##### 包含
> nclude 可引入其他的模板，可以在多模板之间共享一些小模板，如果某个模板已使用了继承那么 include 将会非常有用。
- includes.js
```
let nunjucks=require('nunjucks');
const path=require('path');
nunjucks.configure(path.resolve(__dirname,'view'),{autoescape:true});
let ret2=nunjucks.render('index.html',{items: [{id:1,name:'王二'},{id:2,name:'李四'}]});
console.log(ret2);
```
- view/index.html
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <ul>
      {% for item in items %} {% include "item.html" %} {% endfor %}
    </ul>
  </body>
</html>
```
- view/item.html
```
<li>{{item.id}}:{{item.name}}</li>
```
>> 结果
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <ul>
       <li>1:王二</li>  <li>2:李四</li> 
    </ul>
  </body>
</html>
```