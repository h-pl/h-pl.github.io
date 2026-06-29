---
title: '如何设置UWP的开机自启动'
date: 2024-11-13 09:56:56
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### 1.常规方法
对于开发者已经支持的UWP应用来说，打开设置-启动-启动应用设置。
![](https://h-pl.github.io/post-images/1731463170704.jpg)
### 2.非常规方法
对于开发者没有支持UWP的应用。如，[专注清单](https://www.microsoft.com/store/productId/9N8GPB2TK8GB?ocid=pdpshare)
方法的步骤如下:
    1. 打开文件资源管理器
    2. 在地址栏中复制粘贴shell:AppsFolder
    3. 右键单击该应用程序，然后单击Create Shorcut。
    4. 消息框要求在桌面上创建快捷方式。单击Yes。
    5. 在文件资源管理器地址栏中，复制并粘贴shell:startup
    6. 转到桌面并将快捷方式复制并粘贴到文件资源管理器。
    7. 如果您想测试，请重新启动计算机。
这个方法来自于[StackOverflow](https://stackoverflow.com/questions/35940683/uwp-app-start-automatically-at-startup)，果然还是好好学英语吧。哦，我赞了一票。