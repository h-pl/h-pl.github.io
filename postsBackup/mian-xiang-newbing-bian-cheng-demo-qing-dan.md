---
title: '面向newbing编程Demo清单'
date: 2024-11-12 17:01:40
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### 1. 搜索框
    需求：
    1. 点击搜索按钮，搜索框从右向左弹出并进入输入模式。输入模式下：搜索按钮变为清除按钮。
    1.1. 点击清除按钮，退出输入模式。搜索框回弹。
    2. 输入模式下，输入1，过滤结果为 001, 012, 013，若没有过滤结果，则显示“没有相关摄像头”。
    输入模式下，使用 backspace 清除内容，不退出输入模式。再次点击空白退出输入模式，搜索框回缩。
    2.1. 点击清除按钮，清除内容并退出输入模式，搜索框回缩。

<iframe src="/html/搜索框.html" width="600" height="400"></iframe>
