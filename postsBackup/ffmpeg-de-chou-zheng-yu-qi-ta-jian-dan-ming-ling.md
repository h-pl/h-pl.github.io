---
title: 'ffmpeg的抽帧与其他简单命令'
date: 2024-11-12 10:14:36
tags: []
published: false
hideInList: false
feature: 
isTop: false
---
 ### 1.抽帧
 ```
#每隔50个关键帧抽一次图片
 ffmpeg -i .\*.mp4 -vf "select='eq(pict_type,I)*not(mod(n\,50))'" -vsync vfr .\21W\output_%04d.jpg
```
#
