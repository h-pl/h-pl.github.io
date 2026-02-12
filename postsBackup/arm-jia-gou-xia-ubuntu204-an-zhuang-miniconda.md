---
title: 'ARM架构下ubuntu20.4安装miniconda'
date: 2024-11-19 14:09:18
tags: []
published: false
hideInList: false
feature: 
isTop: false
---
### 查看系统cpu
```
(rkv11py38) jetson@ubuntu:~/yolov11/ultralytics_yolov8-main$ lscpu
Architecture:                    aarch64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       2
Vendor ID:                       ARM
Model:                           1
Model name:                      ARMv8 Processor rev 1 (v8l)
Stepping:                        r0p1
CPU max MHz:                     1984.0000
CPU min MHz:                     115.2000
BogoMIPS:                        62.50
L1d cache:                       512 KiB
L1i cache:                       512 KiB
L2 cache:                        2 MiB
L3 cache:                        4 MiB
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected

```
在 Ubuntu 20.04 上安装 Miniconda（适用于 ARM 架构）可以通过以下步骤完成。因为你使用的是 Jetson 平台，基于 ARM 架构，因此需要特别注意选择适合 ARM 的 Miniconda 安装包。

### 步骤 1: 下载 Miniconda 安装脚本

首先，下载适用于 ARM64 架构的 Miniconda 安装包。使用 `wget` 命令来下载：

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
```

### 步骤 2: 赋予安装脚本执行权限

下载完成后，使用 `chmod` 命令赋予脚本执行权限：

```bash
chmod +x Miniconda3-latest-Linux-aarch64.sh
```

### 步骤 3: 安装 Miniconda

接下来，运行安装脚本来开始安装 Miniconda：

```bash
./Miniconda3-latest-Linux-aarch64.sh
```

安装过程中，脚本会询问你是否同意许可协议，并提示你选择安装路径。一般情况下，默认路径即可（通常是 `~/miniconda3`）。

### 步骤 4: 更新 `~/.bashrc`

安装完成后，需要在你的 `~/.bashrc` 文件中添加 Miniconda 的路径，使得每次登录时都能自动启用 conda 命令。安装脚本通常会提示是否自动更新你的 `~/.bashrc`，如果没有，你可以手动添加以下内容：

```bash
export PATH="$HOME/miniconda3/bin:$PATH"
```

### 步骤 5: 激活 Conda

然后，执行以下命令来使改动生效：

```bash
source ~/.bashrc
```

### 步骤 6: 更新 Conda

你可以选择更新 `conda` 到最新版本：

```bash
conda update conda
```

### 步骤 7: 创建和管理环境

现在你已经成功安装了 Miniconda。你可以使用 `conda` 创建新的环境：

```bash
conda create --name myenv python=3.8
conda activate myenv
```

### 注意事项

- 如果你使用的是 Jetson 设备，确保你下载的 Miniconda 版本是支持 ARM64 架构的。上面链接中的 `aarch64` 版本适用于 ARM 64 位系统。
- 你也可以根据需要选择 Miniconda 或 Anaconda，Anaconda 附带更多的库和工具，但 Miniconda 更为精简，占用空间较小。

## 问题
常见过程中，失败。需要增加对linux-aarch64的支持。
```
conda config --add channels defaults
conda config --add channels conda-forge
conda config --set channel_priority strict
```
然后回到[步骤7](#步骤-7-创建和管理环境)。

### 验证cuda正确安装
1. jetson下
2. 查看jetson驱动
```
jetson@ubuntu:~$ jetson_release -v
Software part of jetson-stats 4.2.3 - (c) 2023, Raffaello Bonghi
Model: NVIDIA Orin NX Developer Kit - Jetpack 5.1.2 [L4T 35.4.1]
NV Power Mode[0]: MAXN
Serial Number: [XXX Show with: jetson_release -s XXX]
Hardware:
 - 699-level Part Number: 699-13767-0000-300 K.1
 - P-Number: p3767-0000
 - Module: NVIDIA Jetson Orin NX (16GB ram)
 - SoC: tegra23x
 - CUDA Arch BIN: 8.7
 - Codename: P3768
Platform:
 - Machine: aarch64
 - System: Linux
 - Distribution: Ubuntu 20.04 focal
 - Release: 5.10.120-tegra
 - Python: 3.8.10
jtop:
 - Version: 4.2.3
 - Service: Active
Libraries:
 - CUDA: 11.4.315
 - cuDNN: 8.6.0.166
 - TensorRT: 5.1.2
 - VPI: 2.3.9
 - Vulkan: 1.3.204
 - OpenCV: 4.5.4 - with CUDA: NO
```
3.验证[cuda测试程序](https://forums.developer.nvidia.com/t/cuda-is-not-installed-on-jetson-orin/220661/7)
```
jetson@ubuntu:~$ cd /usr/local/cuda/samples/1_Utilities/deviceQuery
jetson@ubuntu:/usr/local/cuda/samples/1_Utilities/deviceQuery$ ls -a
.   deviceQuery      deviceQuery.o  NsightEclipse.xml  .vscode
..  deviceQuery.cpp  Makefile       readme.txt
jetson@ubuntu:/usr/local/cuda/samples/1_Utilities/deviceQuery$ make .
make: Nothing to be done for '.'.
jetson@ubuntu:/usr/local/cuda/samples/1_Utilities/deviceQuery$ make
make: Nothing to be done for 'all'.
jetson@ubuntu:/usr/local/cuda/samples/1_Utilities/deviceQuery$ ./deviceQuery
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "Orin"
  CUDA Driver Version / Runtime Version          11.4 / 11.4
  CUDA Capability Major/Minor version number:    8.7
  Total amount of global memory:                 15523 MBytes (16277004288 bytes)
  (008) Multiprocessors, (128) CUDA Cores/MP:    1024 CUDA Cores
  GPU Max Clock rate:                            918 MHz (0.92 GHz)
  Memory Clock rate:                             918 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total shared memory per multiprocessor:        167936 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  1536
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     No
  Integrated GPU sharing Host Memory:            Yes
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Managed Memory:                Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 0 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 11.4, CUDA Runtime Version = 11.4, NumDevs = 1
Result = PASS
```
