---
title: 'WSL下Docke内开始训练第一个yolov5s模型'
date: 2024-11-11 14:37:35
tags: [Docker,yolov5,WSL,攻略]
published: true
hideInList: false
feature: 
isTop: false
---
## 1.训练与环境
Windows下的WSL，内部安装wsl，在wsl安装ubuntu22.04。👍还是gpu快一些。
硬件：
- CPU：Intel i5-1135G7@2.4Ghz
- 内存：16G
- GPU：NVIDIA GeForce MX450

### 1.1tips
🤡镜像内部的Ubuntu系统和wsl的系统版本无关，是依据于镜像配置决定的。

## **2.使用GPU进行训练的流程**
```bash
#启用gpu，映射硬盘，并进入docker
#为防止BUS error，使用增加共享内存大小，进入docker，指定了内存大小，
docker run --gpus all -it -v /mnt/d/3588:/mnt/3588 --shm-size=16g rknn3588:dev /bin/bash
#使用train.py
#根据自己的硬件条件调整，batch-size、workers
python train.py --data mydata.yaml --weights runs/train/exp11/weights/best.pt --cfg yolov5s.yaml --batch-size 4 --epochs 100 --workers 2
#接着训练
python train.py --resume --weights runs/train/exp11/weights/last.pt
#或手动接续训练，best.pt改成最近训练的pt
python train.py --data mydata.yaml --weights runs/train/exp11/weights/best.pt --cfg yolov5s.yaml --batch-size 4 --epochs 100 --workers 2
```
### 2.1 训练结果

### 2.2 检测pt
多使用yolov5工具包自带的detect.py，可以接受图片和视频。详情可以看
```python
# 检测多张图片并保存检测结果；可作为自动检测的工具
python detect.py --weights model1.pt --source path/to/images --save-txt
#检测多张图片输出检测结果，其中检测结果中又置信度
python detect.py --weights model1.pt --source path/to/images --save-txt --save-conf
#检测视频流，每隔150帧检测一次，保存检测结果
python detect.py --weights vestAndHelmet.pt --source testVideo/video.mp4 --save-txt --vid-stride 150
 #检测视频流，每隔25帧检测一次，保存检测部位的裁剪图片
python detect.py --weights vestAndHelmet.pt --source testVideo/video.mp4  --save-txt --save-crop --vid-stride 25
```

[《细说detect的参数》](/post/xi-shuo-yolov5-de-detect-can-shu-yu-ming-ling-xing)

---

## 3.关于硬件的一些问题
### 3.1. 数据加载器的内存问题（Bus Error）
从错误信息来看，出现了 `Bus error`，通常是由于 `DataLoader` 的工作线程用尽了共享内存。你可以通过调整数据加载器的设置来增加共享内存限制，并确保 GPU 正常工作。

#### 解决方法：
你可以尝试增加 Docker 容器的共享内存（`shm`）大小，或调整 `DataLoader` 的工作线程数量。

**增加共享内存：**

在运行 Docker 容器时，可以使用 `--shm-size` 参数来增加共享内存大小。例如，增加到 `16GB`：

```bash
docker run --gpus all -it -v /mnt/d/3588:/mnt/3588 --shm-size=16g rknn3588:dev /bin/bash
```

**减少 DataLoader 的工作线程：**

你可以减少 `DataLoader` 使用的工作线程数量，避免内存不足。将 `workers` 数量减少为 1 或 2，修改训练命令中的 `--workers` 参数：

```bash
python train.py --data mydata.yaml --weights runs/train/exp11/weights/best.pt --cfg yolov5s.yaml --batch-size 4 --epochs 100 --workers 2
```
### 3.1. 硬件与batch-size、workers的设置

`batch-size` 和 `workers` 的设置不仅仅影响收 GPU 性能，还会受系统内存 (RAM) 和 GPU 内存 (VRAM) 的限制，尤其是在内存较小的系统中。**内存往往成为训练的瓶颈。**
如：我的[硬件配置](#1环境)下，使用命令`python train.py --data mydata.yaml --weights runs/train/exp11/weights/best.pt --cfg yolov5s.yaml --batch-size 4 --epochs 100 --workers 2`后，硬件占用情况如下：
|硬件|型号|占用|
|---|---|---|
|CPU|Intel i5-1135G7@2.4Ghz|40%左右|
|GPUNVIDIA GeForce MX450|8%左右|
|内存|16G@3200Mhz|40%左右|70%左右|

#### 3.1.硬件与 `batch-size`、`workers` 经验关系表

| **硬件配置**                          | **系统内存**              | **GPU 内存**               | **建议的 `batch-size`**              | **建议的 `workers`**            | **备注**                                                                                       |
|---------------------------------------|---------------------------|----------------------------|-------------------------------------|---------------------------------|------------------------------------------------------------------------------------------------|
| **低端 CPU (如 i5-1135G7)**            | 8GB - 16GB                | 2GB - 4GB                  | 1 - 4                               | 1 - 2                           | **内存瓶颈**，`batch-size` 和 `workers` 都需控制，避免内存溢出。GPU 内存限制较小，`workers` 数量要少。 |
| **中端 CPU (如 i7 系列)**              | 16GB - 32GB               | 4GB - 6GB                  | 4 - 8                               | 2 - 4                           | 可处理较大的 `batch-size` 和 `workers`，但 GPU 内存限制仍然存在。适当增加 `workers`。               |
| **高端 CPU (如 i9 系列, AMD Ryzen 9)** | 32GB - 64GB               | 8GB - 16GB                 | 8 - 16                              | 4 - 8                           | **内存和 GPU 都足够**，可以处理较大的 `batch-size` 和更多的 `workers`，利用多线程加速数据加载。  |
| **入门级 GPU (如 MX450)**              | 8GB - 16GB                | 2GB - 4GB                  | 2 - 8                               | 2 - 4                           | 内存较小，**限制 `batch-size`**，需要减少 `workers`。较低的 GPU 内存影响 `batch-size`。              |
| **中端 GPU (如 GTX 1660 Ti, RTX 3050)**| 16GB - 32GB               | 6GB - 8GB                  | 8 - 16                              | 4 - 6                           | GPU 内存可支撑较大的 `batch-size`，增加 `workers` 以加速数据加载。内存仍有一定限制。              |
| **高端 GPU (如 RTX 3080, RTX 4090)**   | 32GB - 64GB               | 10GB - 24GB                | 16 - 32                             | 8 - 16                          | 高内存和 GPU 内存，**最大化 `batch-size`**，并且可以使用更多的 `workers`。                         |
| **多 GPU 环境**                        | 32GB - 128GB              | 8GB - 24GB (每个 GPU)      | 16 - 64                             | 8 - 16                          | **多卡训练**时，每个 GPU 使用更大的 `batch-size`，`workers` 数量随之增加。                         |

#### 3.2.内存对 `batch-size` 和 `workers` 的影响：
1. **系统内存 (RAM)**：
   - **高内存系统**：当系统内存较大时，可以使用更多的 `workers` 和更大的 `batch-size`。在训练时，`workers` 用于加载数据并传递给模型，内存较大可以并行加载更多的数据。
   - **低内存系统**：当系统内存较小（例如 8GB 以下）时，增加 `workers` 会导致内存占用过高，可能引发系统性能下降或内存溢出。在这种情况下，应该限制 `workers` 数量，并使用较小的 `batch-size`。

2. **GPU 内存 (VRAM)**：
   - **低 GPU 内存**（如 2GB-4GB）时，`batch-size` 必须较小，否则 GPU 会因内存不足而导致 OOM (Out Of Memory) 错误。如果显存较小，应该尽量减少 `batch-size`，或者使用 `half precision` (FP16) 来减少显存占用。
   - **较大 GPU 内存**（如 8GB-16GB）可以处理更大的 `batch-size`。但是，如果显存不足，依然会导致 OOM 错误。在这种情况下，使用 `gradient checkpointing` 或减少 `batch-size` 仍然是有效的选择。

3. **`workers` 的选择**：
   - **增加 `workers`** 可以加速数据加载，特别是在使用 SSD 存储时。但是每增加一个 worker 就会占用额外的系统内存，因此在内存受限时，需要减少 `workers` 数量。
   - **减少 `workers`** 可以减轻内存压力，但可能导致数据加载成为训练的瓶颈，尤其是在硬盘读写速度较慢时。

#### 3.3. 实际操作建议：
- **内存受限**：将 `batch-size` 控制在较小值，`workers` 设置为 2 或更少，以防止内存占用过高。
- **显存受限**：减少 `batch-size` 或尝试使用 `half precision`（FP16），可以减少 GPU 显存的占用。
- **内存和显存都足够**：可以增加 `batch-size` 和 `workers`，尽可能提高训练效率。

#### 3.4.如何调整训练参数：
1. **如果内存占用过高**，可以尝试：
   - 降低 `batch-size`，例如从 8 调整为 4 或 2。
   - 降低 `workers`，例如将 `workers` 从 4 调整为 2 或 1。
   - 在训练时，使用较低分辨率的图像（例如将 640 降至 416 或 320）。
   
2. **如果训练速度过慢**，且内存和 GPU 显存足够，可以考虑：
   - 增加 `batch-size`，例如从 4 提升到 8 或 16。
   - 增加 `workers`，例如将 `workers` 从 2 增加到 4 或 6。

3. **如何继续训练**：
   - 如果需要停掉当前训练并调整参数，确保你保存了训练的检查点（`.pt` 文件）。你可以通过命令 `python train.py --resume --weights runs/train/exp11/weights/last.pt` 来继续训练，而不会丢失已训练的进度。

这样，你可以根据硬件资源来动态调整训练的参数，以确保训练高效且稳定。

---
## 4.输出的参数

```
   Epoch    GPU_mem   box_loss   obj_loss   cls_loss  Instances       Size
    43/99      1.03G    0.04366    0.02842    0.00566          7        640: 100%|██████████| 55/55 00:33
    Class     Images  Instances          P          R      mAP50   mAP50-95: 100%|██████████| 5/5 00:01
    all         40        131      0.618      0.486      0.496       0.26
```
在 YOLOv5 或类似模型的训练日志中，输出的各个参数代表了训练和验证阶段的不同指标。下面是对这些参数的详细解释，整理成表格形式：

### 4.1.训练过程中的输出参数解释

| **参数**        | **说明**                                                                                     |
|-----------------|----------------------------------------------------------------------------------------------|
| **Epoch**       | 当前训练的轮次（例如，第 43 轮 / 总 99 轮）。                                                    |
| **GPU_mem**     | 当前训练使用的 GPU 内存（例如 1.03GB）。表示在该轮次的训练中，GPU 的内存占用。                 |
| **box_loss**    | 边框损失（bounding box loss），表示模型预测的框与真实框之间的差异，通常使用 **MSE** 或 **CIoU** 等衡量。 |
| **obj_loss**    | 目标损失（objectness loss），表示模型对物体存在与否的预测误差。                            |
| **cls_loss**    | 分类损失（classification loss），表示模型在物体分类方面的错误，通常使用交叉熵损失（cross-entropy）。 |
| **Instances**   | 当前批次中使用的实例数（例如，7）。表示用于训练的图片数量。                                  |
| **Size**        | 输入图片的尺寸（例如，640）。通常是输入图片的长或宽，模型的默认输入尺寸（如 640x640）表示每个图片的大小。 |

### 4.2.验证阶段输出的参数解释

| **参数**        | **说明**                                                                                     |
|-----------------|----------------------------------------------------------------------------------------------|
| **Class**       | 当前评估的类别（例如，`all` 表示所有类别的平均值）。                                            |
| **Images**      | 当前验证集中的图像数量。                                                                     |
| **Instances**   | 当前验证集中实例的总数。                                                                     |
| **P (Precision)**| 精度（Precision），表示预测为正类的样本中，真正正类的比例。公式：`P = TP / (TP + FP)`。 |
| **R (Recall)**  | 召回率（Recall），表示真实正类中被正确预测为正类的比例。公式：`R = TP / (TP + FN)`。   |
| **mAP50**       | 在 IoU 阈值为 0.5 时的平均精度（mean Average Precision at IoU=0.5）。                   |
| **mAP50-95**    | 在 IoU 阈值从 0.5 到 0.95 范围内（包括每个 0.05 步长）的平均精度（mAP at IoU=0.5:0.95）。       |

### 4.3.参数详细解释

1. **Precision (P)**：精度，衡量模型预测为正类的样本中，实际为正类的比例。高精度意味着较少的假阳性（FP）。
   
     P ={TP}/{TP + FP}

2. **Recall (R)**：召回率，衡量所有真实正类中，有多少被正确地预测为正类。高召回率意味着较少的假阴性（FN）。

   R = {TP}/{TP + FN}

3. **mAP50**：在 IoU 阈值为 0.5 时计算的平均精度，衡量在该阈值下模型的整体检测性能。通常是衡量目标检测性能的基本指标。

4. **mAP50-95**：在多个不同的 IoU 阈值下（从 0.5 到 0.95，每 0.05 步长）计算的平均精度。它更严格地评估了模型在不同重叠区域的检测精度。

5. **TP (True Positives)**：真正例，表示模型正确预测为正类的样本数。
6. **FP (False Positives)**：假正例，表示模型错误预测为正类的样本数。
7. **FN (False Negatives)**：假负例，表示模型错误预测为负类的样本数。

### 4.4.训练中的一些特定指标解释：

- **box_loss, obj_loss, cls_loss**：这些是反映训练中模型在不同任务上（定位、物体存在性、分类）的表现的损失值。较低的损失值意味着模型在这些任务上表现更好。
  
- **GPU_mem**：GPU 内存的使用情况，较高的内存占用可能意味着 `batch-size` 或模型过大，可能需要优化。

#### 4.5.总结：
- **训练阶段**：`box_loss`、`obj_loss` 和 `cls_loss` 显示了模型在训练过程中不同任务的损失。
- **验证阶段**：`mAP50` 和 `mAP50-95` 显示了模型的检测性能，而 `P` 和 `R` 是检验分类性能的关键指标。

希望这个表格能帮助你更好地理解训练日志中的参数，并优化你的训练过程！

## 其他问题

### 1.为什么GPU没有使用？
首先查看GPU的CUDA的驱动是否正确安装？然后查看Docker环境内的cuda驱动是否兼容？
```
#查看GPU驱动在WSL内
nvidia-smi
#打印结果
Mon Nov 11 14:28:53 2024
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.183.04             Driver Version: 538.78       CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce MX450           On  | 00000000:05:00.0 Off |                  N/A |
| N/A   51C    P8              N/A / ERR! |      0MiB /  2048MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+
```
#### 2.1.没有使用gpu的原因
发现wsl的nvidia-container-toolkit没有安装。现在只要在docker的宿主机上，即wsl内安装nvidia-container-toolkit就可以了。
不需要像过去一样，安装支持nvidia的镜像了。~~sudo apt install nvidia-docker2~~

#### 2.2.在wsl内安装支持docker的gpu驱动
```bash
#安装nvidia-container-toolkit
apt-get update
apt-get install -y nvidia-container-toolkit
systemctl restart docker
#检查nvidia驱动
nvidia-smi
#使用python，检查cuda是否正常；在python交互命令行中
>>import torch
>>print(torch.cuda.is_available())
TRUE
# 看到true安装成功
```
#### 2.3.查看docker内的cuda是否正常
```
#进入docker，开启gpu
docker run --gpus all -it -v /mnt/d/3588:/mnt/3588 rknn3588:dev /bin/bash
#使用python，检查cuda是否正常；在python交互命令行中
>>import torch
>>print(torch.cuda.is_available())
TRUE
# 看到true安装成功
```

### 3. 修改报错语句
```
train.py:307: FutureWarning: `torch.cuda.amp.autocast(args...)` is deprecated. Please use `torch.amp.autocast('cuda', args...)` instead.
  with torch.cuda.amp.autocast(amp):
# 一条弃用的api
```

要排查 `train.py:307: FutureWarning: torch.cuda.amp.autocast(args...) is deprecated. Please use torch.amp.autocast('cuda', args...) instead` 这个告警，建议按照以下步骤操作：

#### 3.1. **更新代码以避免使用弃用的 API**
告警提示 `torch.cuda.amp.autocast` 已经被弃用，建议使用 `torch.amp.autocast('cuda', args...)` 替代。你可以修改代码中相关部分，确保使用新 API。

修改前：

```python
with torch.cuda.amp.autocast(amp):
    # 训练代码
```

修改后：

```python
with torch.amp.autocast('cuda', enabled=amp):
    # 训练代码
```

这样可以消除警告，并且确保你使用的是 PyTorch 推荐的 API。

### 4.查看 TensorBoard 页面
我的环境是windows下的wsl内的docker，所以需要将docker的对应端口映射出来。


### 5.停止后接着上一次的训练
```
#训练模型，--batch-size会影响内存占用的大小
python train.py --data mydata.yaml --weight yolov5s.pt --cfg yolov5s.yaml --batch-size 2 --epochs 100
#接着上一次的训练
python train.py --data mydata.yaml --weights runs/train/exp8/weights/last.pt --cfg yolov5s.yaml --batch-size 2 --epochs 100
```
### 6.训练结果
```
Class       Images     Instances          P          R      mAP50   mAP50-95: 100%|██████████| 12/12 00:03
    all         47        267       0.88      0.532      0.613      0.308
    Vest         47        113      0.844      0.867      0.902      0.487
  helmet         47        115      0.786      0.809      0.826      0.379
noHelmet         47         21          1          0      0.172     0.0899
  noVest         47         18      0.891      0.453      0.554      0.274
```
这是一个使用 YOLOv5 进行物体检测后的结果，显示了模型在不同类别上的检测性能。以下是对输出的解析和每个参数的说明：

1. **Class (类别)**: 模型检测到的对象类别，包括 `Vest`（安全背心）、`helmet`（头盔）、`noHelmet`（未戴头盔）、`noVest`（未穿背心）。
   
2. **Images (图像)**: 用于验证模型的图像总数。这里是 47 张图像。

3. **Instances (实例)**: 数据集中标注的对象实例总数。总共有 267 个实例分布在不同类别中。
   
4. **P (Precision，精确率)**: 精确率衡量的是模型预测为正的样本中有多少是真正的正样本。精确率越高，说明模型对正类的预测更可靠。总的精确率是 0.88，其中 `Vest` 为 0.844，`helmet` 为 0.786，`noHelmet` 为 1，`noVest` 为 0.891。

5. **R (Recall，召回率)**: 召回率衡量的是所有真实的正样本中，有多少被模型正确识别。召回率较低说明漏检率较高。整体的召回率是 0.532，其中 `Vest` 的召回率为 0.867，`helmet` 为 0.809，`noHelmet` 的召回率为 0，`noVest` 为 0.453。

6. **mAP50 (mean Average Precision at IoU 0.5，平均精度均值，IoU = 0.5)**: mAP50 是在 IoU (Intersection over Union, 交并比) 为 0.5 时，计算每个类别的平均精度值（AP），并取均值。这个值衡量了模型在检测到目标位置时的准确性。整体 mAP50 为 0.613，其中 `Vest` 为 0.902，`helmet` 为 0.826，`noHelmet` 为 0.172，`noVest` 为 0.554。

7. **mAP50-95 (mean Average Precision at IoU 0.5 to 0.95)**: mAP50-95 是更严格的评价标准，它计算在 IoU 范围从 0.5 到 0.95（步长为 0.05）下的平均精度。这个指标综合考虑了模型对目标检测的精度和位置的准确性。整体 mAP50-95 为 0.308，其中 `Vest` 为 0.487，`helmet` 为 0.379，`noHelmet` 为 0.0899，`noVest` 为 0.274。

### 解析与分析：
- **精确率（P）较高**：模型对大多数类别预测为正时是可靠的，特别是 `noHelmet`，显示为 1（精确率非常高）。
- **召回率（R）偏低**：整体召回率较低，说明模型漏检较多，尤其是 `noHelmet` 类别，召回率为 0，表示模型几乎未能识别未戴头盔的实例。
- **mAP50 高于 mAP50-95**：mAP50 的值通常会高于 mAP50-95，因为在 IoU = 0.5 时，检测框的位置要求不如更高 IoU 时严格。从结果看，`Vest` 和 `helmet` 类别的检测较为理想，但对于未戴头盔（`noHelmet`）的检测效果较差。

### 优化建议：
- **提升召回率**：可以考虑进一步优化训练数据，或者尝试调整模型参数来改善召回率，尤其是对难以检测的类别（如 `noHelmet` 和 `noVest`）。
- **调整阈值**：可能需要调整检测阈值，尤其是针对 `noHelmet` 类别，以减少漏检。


