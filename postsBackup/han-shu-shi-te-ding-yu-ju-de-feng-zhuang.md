---
title: '函数是特定语句的封装'
date: 2022-01-19 18:48:08
tags: [jsNote]
published: true
hideInList: true
feature: 
isTop: false
---
函数将一些功能或语句进行**封装**，为的是方便**再次**实现这个功能时，去调用这些语句。
## 函数的定义与调用
```js
//函数的声明
function f(a,b){
    return a+b
}
//函数的调用
f(1,2)

//函数定义完后，立即执行
(function ( ) {
    console.log('1111')
})();
//其中，()之前的为函数的声明；()为调用，结果为1111

//绑定事件。由事件调用
<div>
    <div id="btn" style="cursor: pointer;font-size: large;">▶️点我 有惊喜</div>
    <script>
        let btn = document.getElementById("btn")
        btn.onclick = function () {
            alert('嘿 你点了我一下')
        }
    </script>
</div>
```
<div>
<div id="btn" style="cursor: pointer;font-size: medium;padding-left: 16px;"> ▶️点我 有惊喜</div>
<script>
    let btn = document.getElementById("btn")
    btn.onclick = function () {
        alert('嘿 你点了我一下')
    }
</script>
</div>
<br>

## 函数的形参与实参 
实参和形参的个数不匹配时，**调用函数时，解析器也不会检查实参的数量。** 
    1️⃣ 如果实参的数量多余形参的数量，多余实参不会被赋值。
    2️⃣ 如果实参的数量少于形参的数量，多余的形参会被定义为 undefined。表达式的运行结果为 NaN。

## 函数的 continue、break、return
**continue**  跳出当前循环的本次循环
**break** 跳出当前循环
**return** 退出本函数
1️⃣ 如果return语句后不跟任何值，就相当于返回一个undefined
2️⃣ 如果函数中不写return，则也会返回undefined
3️⃣ return 只能返回一个值。如果用逗号隔开多个值，则以最后一个为准。

## 函数与方法
对象的属性是函数时，那么，这个函数被称为该对象的方法。

## 函数的内置对象-arguments
所有函数都内置了一个 arguments 对象（只有函数才有 arguments 对象），用于存储传入的**所有**实参。所以，当我们[不确定有多少个参数](#函数的形参与实参)传入时，可用 arguments 来索引。
```js
function add(a, b, c) {
    console.log(arguments[3])
    return a + b + c;

}

console.log(add(1, 2, 3, 4))
//4
//6
```
arguments的展示形式是一个伪数组。因此具备伪数组的特征：
1️⃣ 可以进行遍历，具有数组的 length 属性。
2️⃣ 按索引方式存储数据。
3️⃣ 不具有数组的 push()、pop() 等方法，不能修改数组。
|属性或方法|解释|备注|
|---|---|---|
|arguments.length|实参的长度|console.log(arguments.length)|
|arguments.callee|正在指向的函数|递归函数中使用|
|arguments[0]|函数第一个实参|只能修改实参 arguments[0] = 99|
