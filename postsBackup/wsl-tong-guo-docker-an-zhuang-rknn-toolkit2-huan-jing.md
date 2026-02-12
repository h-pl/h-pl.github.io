---
title: 'WSL通过Docker安装RKNN-Toolkit2环境'
date: 2024-10-30 13:36:26
tags: [RKNN,WSL,攻略]
published: true
hideInList: false
feature: 
isTop: false
---
### 环境
我的环境是Windows下安装的WSL，系统内Ubuntu22.04。

### 1. 安装WSL Ubuntu22.04

1.1 启用或关闭Windows的功能，特别是要挂在到D盘下。
![](https://h-pl.github.io/post-images/1730267263970.png)
1.2 安装WSL，
   Microsoft AppStore 搜索 Ubuntu 22.04， 点击安装；
   等待安装；
   启动WSL，若启动异常，查看Ubuntu 22.04 的评论区。应是需更新wsl，```wsl --update```。

### 2.安装Docker
2.1 更换 apt 源为清华源。
```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo nano /etc/apt/sources.list
```
注释或Ctrl+K 删除原来的官方源头。
Ubuntu22.04 代号 jammy，修改 **sources.list** 内容为：
```bash
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
```



### 3.Docker的简单使用
```bash
#查看docker版本
docker --vesion
#查看docker已安装的镜像images
docker images
##打印镜像列表
REPOSITORY      TAG          IMAGE ID       CREATED         SIZE
rknn-toolkit2   2.2.0-cp38   d371fba025c2   7 weeks ago     3.31GB
hello-world     latest       d2c94e258dcb   18 months ago   13.3kB
#进入docker的某一个镜像，启用交互终端进入容器的 shell。
docker run -it rknn-toolkit2:2.2.0-cp38 /bin/bash
#以映射wsl的地址到docker内的方式，启用交互终端进入容器的 shell。
docker run -it -v /mnt/d/3588:/mnt/3588 rknn-toolkit2:2.2.0-cp38 /bin/bash
#安装好的python环境容器持久化，将e95容器生成镜像rknn3588:dev
docker commit e95 rknn3588:dev
docker commit c46 rknn3588:used1107
#以映射wsl地址的方式进入rknn3588:dev镜像，启用交互终端进入容器的 shell。
docker run -it -v /mnt/d/3588:/mnt/3588 rknn3588:dev /bin/bash
#docker内开启代理
root@ce60fd27a002:/mnt/3588/yolov5_export/yolov5_export# export https_proxy=http://10.16.168.39:10809
#显示docker所有镜像
docker image list
#显示docker运行的容器
docker ps -a
#进入docker正在运行的容器
docker exec -it 容器id前三位数字 /bin/bash
#启动关闭的容器
docker start 容器id前三位数字
```


### 4.pip的简单使用
```bash
#查询pip的配置项
pip config list
#更换pip源头
 pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
 ```

 ### 5.rknn的转换模型与测试
 ```bash
 #python 转换脚本 docker内 roadlight到onnx的模型
 python export.py --rknpu --weight roadlight.pt
 #python 转换脚本3588板子 onnx到rknn的模型
 python -m rknn.api.rknn_convert -t rk3588 -i ./model_config.yml -o ./
 #python 测试图片
 python test.py --img roadlight.jpg
 #python 测试视频、图片文件夹
python detect.py --weights vestAndHelmet.pt --source path/to/images --save-txt --save-conf
  ```

  ### 6.ffmpeg 视频抽帧
  ```
  #每个180个关键帧取出一个关键帧
ffmpeg -i .\21水-安全帽.mp4 -vf "select='eq(pict_type,I)*not(mod(n\,180))'" -vsync vfr .\21W\output_%04d.jpg
   ```
 
 ### 7. python与python的虚拟环境
```python
#创建项目路径
cd projectDir
#创建虚拟环境
python -m venv yourEnvName
#打开虚拟环境
yourEnvName\Scripts\activate
#安装所需要依赖
python install -r requirements.txt
#推出虚拟环境
deactivate
```

### 8.自动标注数据或验证模型
```
#使用yolov5的detect生成标注数据txt，不包含置信度
python detect.py --weights model1.pt model2.pt --source path/to/images --save-txt
#包含置信度的命令，一般不用
 python detect.py --weights model1.pt model2.pt --source path/to/images --save-txt --save-conf
 #添加classes.txt后，用labelImg二次编辑
 #验证模型的准确性，150为帧数*5，为多少帧检测一次
 python detect.py --weights vestAndHelmet.pt --source testVideo/video.mp4 --save-txt --vid-stride 150

 ```
 ### 9.使用lablestudio修改数据
```
#安装新的python虚拟环境labelstudio38
python -m venv labelstudio38
#安装labelstudio
pip install label-studio
#启动labelstudio
label-studio start
```


