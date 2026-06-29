---
title: 'Dynamo 的神奇拓展包'
date: 2022-01-18 13:37:10
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
进入下面的正题之前，如果你安装的是revit2018，那么需要加装Dynamo[2.0.4]。其他版本的revit，请适配对应的最新版Dynamo。额外的，你还需要具备一些常用的 Dynamo 基础节点知识。
|类目|节点名|作用|备注|
|---|---|---|---|
|Revit.Selection|Select Model Element(Elements)|选择图元|交互式选择，鼠标点选或框选|
|Revit.Selection|Element Types|选择项目中任意图元类型|枚举类下拉菜单|
|List.Modify|List.FilterByBoolMask|按真假过滤列表|需和条件(codeblock)配合使用，如(x=="...")|
|Geometry.Geometry|Geometry.Translate|将任意几何图形朝给定方向按给定距离平移|已知一个起点，平移得到端点|


1. [MEPover](https://bimstallatie.sites.google.com/site/bimstallatie/revit-add-ins/offsetincrementer)

2. 