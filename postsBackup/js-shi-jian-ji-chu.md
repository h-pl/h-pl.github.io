---
title: 'JS 事件基础'
date: 2022-04-07 00:32:18
tags: [jsNote]
published: true
hideInList: true
feature: 
isTop: false
---
### js修改样式
1. 设置样式与通过js修改样式
   DOM中，样式的设置有**两种**形式，clssName和style。js读取样式有**两种**方式。
   ```js  
   //方式一
    xxx.style.className
    //方式二
    xxx.style["className"]
    ```
⚠️ js样式采用就近原则，因此，行内样式有较高的优先级。当然，<code>!important</code>会有更高的优先级。

2. style样式的常见属性
   |样式类别|属性名|语法|
   |---|---|---|
   |大小|width、height|
   |颜色|color与backgroundColor、backgroundImage|  
   |边框|border|2px solid red|
   |透明度|opacity|
3. 实际例子
   输入框聚焦，但chrome浏览器自带聚焦
   ```js
    <script>
    //获取事件源
    var inpArr = document.getElementsByTagName("input");
    //绑定事件
    for (let i = 0; i < inpArr.length; i++) {
        inpArr[i].onfocus = function () {
            this.style.border = "2px solid red";
            this.style.backgroundColor = "#ccc";
        }
        inpArr[i].onblur = function () {
            this.style.border = "";
            this.style.backgroundColor = "";
        }

    }

</script>
    ```
4. 
   