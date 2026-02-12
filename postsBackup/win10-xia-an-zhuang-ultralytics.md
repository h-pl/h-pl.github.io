---
title: 'win10下安装ultralytics'
date: 2024-11-20 10:15:32
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
1. 使用miniconda建立python的虚拟环境，`conda create --name myenv python=3.8.10`
2. 进入python虚拟环境，`conda activate myenv` 
3. 虚拟环境内先安装ultralytics，在安装cuda环境，因为ultralytics有很多依赖，就算是你的cuda环境安装好了，也会被ultralytics破坏掉。所以，先安装ultralytics，`pip install ultralytics`，测试yolov，`yolo predict model=yolo11n.pt source=bus.jpg`
4. 查看cuda的版本，`navidia-smi`
5. 根据cuda的版本，在[pytorch官方](https://pytorch.org/get-started/previous-versions/)上查看对应的torchvision、torchaudio版本，或直接使用对应版本的安装命令行
6. 我的cuda版本在`navidia-smi`的输出结果中查到是cuda12.2，在pytorch官方列表中没有查到。好在torch一般向下兼容，所以使用cuda12.1。
7. 在[pytorch官方](https://pytorch.org/get-started/previous-versions/)列表页搜索cuda12.1，于是查到很多支持的配置项目。根据[ultralytics官方库](https://github.com/ultralytics/ultralytics)的**myproject.toml**以下几行，查到Windows下不支持torch2.4.0
   
>"torch>=1.8.0",
"torch>=1.8.0,!=2.4.0; sys_platform == 'win32'", # Windows CPU errors w/ 2.4.0 https://github.com/ultralytics/ultralytics/issues/15049
"torchvision>=0.9.0",


8. 本着安装新版本不装旧版本的原则，选取`conda install pytorch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 pytorch-cuda=12.1 -c pytorch -c nvidia`
9. 验证cuda安装，进入python，`import torch`，`print(torch.cuda.is_available())`
10. 报错，删除当前环境的 libiomp5md.dll ，[解决方法的参考链接](https://stackoverflow.com/questions/20554074/sklearn-omp-error-15-initializing-libiomp5md-dll-but-found-mk2iomp5md-dll-a/69084316#69084316)

> import torch
OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized.
OMP: Hint This means that multiple copies of the OpenMP runtime have been linked into the program. That is dangerous, since it can degrade performance or cause incorrect results. The best thing to do is to ensure that only a single OpenMP runtime is linked into the process, e.g. by avoiding static linking of the OpenMP runtime in any library. As an unsafe, unsupported, undocumented workaround you can set the environment variable KMP_DUPLICATE_LIB_OK=TRUE to allow the program to continue to execute, but that may cause crashes or silently produce incorrect results. For more information, please see http://www.intel.com/software/products/support/.

11. 之后再次运行ultralytics的yolo检测，`yolo predict model=yolo11n.pt source=bus.jpg`, 应该可以看到cuda在运行了。
12. 如果是其他平台或者其他的卡，请下载响应的pytroch。如jetson卡，请参照nvidia社区维护的jetson专用pytorch环境，[参照链接](https://docs.nvidia.com/deeplearning/frameworks/install-pytorch-jetson-platform/index.html#overview__section_z4r_vjd_v2c)。`https://developer.download.nvidia.com/compute/redist/jp/v$JP_VERSION/pytorch/$PYT_VERSION`