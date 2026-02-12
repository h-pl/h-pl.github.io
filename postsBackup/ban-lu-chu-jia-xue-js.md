---
title: 'JS学习笔记'
date: 2021-11-23 14:56:47
tags: [jsNote]
published: true
hideInList: true
feature: 
isTop: false
---
学习js，你只需要一个vscode编辑器——它内置的js调试环境可以十分有效的帮助你。
![](https://h-pl.github.io/post-images/1642594378648.jpg)
1. 数组的方法或属性
   |方法名|函数|等效函数|结果|备注|
   |----|----|----|----|----|
   |数组连接|arr1.concat(arr2,...)|arr1,...arr2|
   |遍历数组|foreach()|
   |遍历并新建数组|map()|
   |过滤器|filter()|||类似rhino的dispatch|
   |迭代器|reduce()|||类似数学归纳法|
   |排序|sort()|a-b|1、大于0，交换位置 2、不大于0，不交换位置|会改变原来的数组|    
    注：**使用方法时，使用引用方式修改数组的元素，会改变原数组**
2. [函数](/post/han-shu-shi-te-ding-yu-ju-de-feng-zhuang)
3. 作用域和变量提升
   |定义名|特征/作用于|作用时长|备注|例子|
   |---|---|---|---|---|
   |全局作用域|作用于整个 script 标签内部或一个独立的 JS 文件|浏览器关闭时，较占内存|全局对象(变量、函数等)，如window|let a=100 等于 window.a|
   |局部作用域|作用于局部|代码块结束时|局部变量、函数，如函数作用域只作用于函数内|
   |预处理与变量提升|配合var声明变量||let声明的变量为块级变量|
   > 打开浏览器➡️浏览器创建全局对象window➡️创建的一切 X 都是window.X
   > 预处理：将当前 JS 中所有变量、函数的声明代码放到其他代码之前，也叫**声明提前**。但，并不执行赋值。
    ```js
    //var 声明提前
    console.log(a)
    var a  = 1
    // undefined。未报错➡️a已经声明

    //无var 声明不提前 
    console.log(a)
    a  = 1
    //报错

    //
    let a = 2
    console.log(a)
    if (true) {
        //块级变量只存在于这个函数作用域内
        let a = 3
        console.log(a)
    }
    console.log(a)
    //2
    //3
    //2
    ```
    > let 声明变量不会提前。
    > 没有 var 声明的变量都是全局变量，而且并不会提前声明。
    > 多个作用域叠加时（作用域链），变量查找遵循**就近原则**。
4. call 与 this 指向
1️⃣以函数的形式（包括普通函数、定时器函数、立即执行函数）调用时，this 的指向是 **window**。
2️⃣以对象的形式（包括对象的方法、构造函数的实例对象、事件绑定的对象）调用时，this 的指向是**对象**。
3️⃣改变函数内部的this指向，call()、apply()、blind()。

    ```js
    //call改变指向、实现继承
    function Father(myName, myAge) {
        this.name = myName;
        this.age = myAge;
    }

    function Son(myName, myAge) {
        Father.call(this, myName, myAge);
    }

    const son1 = new Son('heihei', 22)
    console.log(JSON.stringify(son1))
    //{"name":"heihei","age":22}

    //apply()与call()不同在于传入的参数，前者是数组
    //apply()的妙用
    
    ```





