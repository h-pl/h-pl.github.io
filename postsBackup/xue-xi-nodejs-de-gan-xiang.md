---
title: '学习nodejs的感想'
date: 2024-11-26 12:58:19
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
## node篇章
### 1.vscode创建http-server
vscode安装好[node插件](https://marketplace.visualstudio.com/items?itemName=chris-noring.node-snippets)后，使用node-http-server就可以快速创建出一个http-server。
```js
var http = require('http');
http.createServer(function (request, response) {
  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World');
}).listen(8081);

console.log('Server running at http://127.0.0.1:8081/');
```
### 2.Node.js是事件驱动的
要理解是由事件驱动的，从字面意思不好理解。那么，我类比一下，“汽车是发动机驱动的，蒸汽火车是由蒸汽机驱动的”。那么，就明白了事件对于nodejs的意义了。
怎么理解这句话：
>我们不知道这件事情什么时候会发生，但是我们现在有了一个处理请求的地方：它就是我们传递过去的那个函数。至于它是被预先定义的函数还是匿名函数，就无关紧要了。
这个就是传说中的 回调 。**我们给某个方法传递了一个函数，这个方法在有相应事件发生时调用这个函数**来进行 回调 。


---
## node.js篇
### 1.函数或匿名函数作为参数传入
要理解时间
