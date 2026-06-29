---
title: 'JavaScript 的事件'
date: 2022-03-31 22:15:35
tags: [jsNote]
published: true
hideInList: true
feature: 
isTop: false
---
1️⃣JavaScript 是以**事件**驱动为核心的一门语言。JavaScript 与 HTML 之间的交互是通过事件实现的。
|事件|具体内容|步骤|
|---|---|---|
|事件源|html标签|document.getElementById("box");|
|事件|js代码|事件源box.事件onclick = function(){ 事件驱动程序 };|
|事件驱动程序|对css和html的操作|关于DOM的操作|

事件名
![](https://h-pl.github.io/post-images/1648737037035.png)


2️⃣获取事件源的方式（DOM节点的获取）
- 获取事件源的常见方式如下：
```js
//方式一：通过id获取单个标签
var div1 = document.getElementById("box1"); 
//方式二：通过 标签名 获得 标签数组，所以有s   
var arr1 = document.getElementsByTagName("div");   
//方式三：通过 类名 获得 标签数组，所以有s  
var arr2 = document.getElementsByClassName("hehe");  
```
- 绑定事件的方式：
```js
//方式一：直接绑定匿名函数
<div id="box1" ></div>
<script type="text/javascript">
    var div1 = document.getElementById("box1");
    //绑定事件的第一种方式
    div1.onclick = function () {
        alert("我是弹出的内容");
    }
</script>
//方式二：先单独定义函数，再绑定。注意上方代码的注释。绑定的时候，
//是写fn，不是写fn()。fn代表的是整个函数，而fn()代表的是返回值。
<div id="box1" ></div>
<script type="text/javascript">
    var div1 = document.getElementById("box1");
    //绑定事件的第二种方式
    div1.onclick = fn;   //注意，这里是fn，不是fn()。fn()指的是返回值。
    //单独定义函数
    function fn() {
        alert("我是弹出的内容");
    }
</script>
//方式三：行内绑定
<div id="box1" onclick="fn()"></div>
<script type="text/javascript">
    function fn() {
        alert("我是弹出的内容");
    }
</script>
```
3️⃣