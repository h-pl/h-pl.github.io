---
title: 'windows训练模型及适配瑞芯微3588'
date: 2024-11-13 13:25:06
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
### 1.思路
经过一段时间的摸索，终于可以自己训练模型，以及适配3588芯片了。整个流程其实比较繁琐。好在官方的SDK中有详细的文档。~~虽然部分地方有点过时。~~
    1. 安装WSL
    2. 安装Docker环境
    3. 使用瑞芯微SDK rknn-toolkit2-2.2.0 官方库
    4. 安装rknn-toolkit2-2.2.0-cp38的docker镜像
    5. 测试官方库中yolov5的demo算法
   ---

### yolov8
1. 下载瑞芯微yolov8的github库，官方已经直接改成v11了。
2. 安装依赖，`pip install -e .`，自动拿github库下的pyproject.toml安装依赖。
3. docker以gpu启动映射地址`docker run --gpus all -it -v /mnt/d/yolov8:/mnt/yolov8 --shm-size=14g rknn3588:gpu  /bin/bash`



### 命令行解释

#### docker命令行解释
```bash
docker run --gpus all -it -v /mnt/d/yolov8:/mnt/yolov8 --shm-size=14g rknn3588:gpu  /bin/bash
```

- `--gpus all`: 分配 **所有可用的 GPU** 给容器，允许容器使用主机上的 GPU 资源。
- `-it`: 启动一个 **交互式终端**，允许你与容器内部进行交互。
- `-v /mnt/d/3588:/mnt/3588`: **挂载宿主机目录** `/mnt/d/3588` 到容器内部的 `/mnt/3588`。这允许容器访问宿主机上的文件并共享数据。
- `--shm-size=16g`: 这个选项增加了容器 **共享内存 (shm) 的大小**，将其设置为 **16GB**。共享内存是容器内进程间通信和临时存储数据的区域。对于深度学习模型训练和大数据处理，通常需要较大的共享内存来提高性能。默认情况下，Docker 容器的共享内存大小为 64MB，而深度学习任务可能需要更多的内存。如果不增加共享内存的大小，某些程序可能会遇到 `out of memory` 错误。
- `rknn3588:dev`: 使用 `rknn3588` 镜像的 `dev` 标签来启动容器，表示这是一个开发环境的镜像版本。
- `/bin/bash`: 启动容器后运行一个 Bash Shell，这样你可以进入容器并执行命令进行交互。

##### 为什么需要 `--shm-size`？
在深度学习和其他计算密集型应用中，容器的共享内存（shm）可能不足以容纳模型、数据集或中间计算结果，特别是当处理大规模数据时。通过增加 `--shm-size`，你可以确保容器有足够的共享内存来运行这些任务，避免出现内存不足的错误。

##### 总结：
这条命令启动一个 GPU 加速的 Docker 容器，分配 16GB 的共享内存，并挂载指定的宿主机目录到容器中。它还确保你能够以交互模式访问容器，进行开发、调试等操作。这样可以支持大内存和 GPU 加速的工作负载，适合用于深度学习等任务。