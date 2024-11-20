---
title: 'win10下安装ultralytics'
date: 2024-11-20 10:15:32
tags: []
published: false
hideInList: false
feature: 
isTop: false
---
1. 使用miniconda建立python的虚拟环境，`conda create --name myenv python=3.8.10`
2. 进入python虚拟环境，`conda activate myenv` 
3. 虚拟环境内先安装ultralytics，在安装cuda环境，因为ultralytics有很多依赖，就算是你的cuda环境安装好了，也会被ultralytics破坏掉。所以，先安装ultralytics，`pip install ultralytics`，测试yolov，`yolo predict model=yolo11n.pt source=bus.jpg`
4. 查看cuda的版本，`navidia-smi`
5. 根据cuda的版本，在[pytorch官方](https://pytorch.org/get-started/previous-versions/)上查看对应的torchvision、torchaudio版本，或直接使用对应版本的安装命令行