---
title: 'jetson卡的边缘盒子上安装cuda环境及ultralytics'
date: 2024-11-20 20:15:03
tags: []
published: false
hideInList: false
feature: 
isTop: false
---
>Validating VAH/exp1120-09/weights/best.pt...
Ultralytics 8.3.33 🚀 Python-3.8.10 torch-2.1.0a0+41361538.nv23.06 CUDA:0 (Orin, 15523MiB)
YOLO11s summary (fused): 238 layers, 9,414,348 parameters, 0 gradients, 21.3 GFLOPs
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95): 100%|██████████| 2/2 [00:02<00:00,  1.02s/it]
                   all         40        131      0.803      0.661      0.718      0.405
                  Vest         17         50      0.813        0.9      0.887      0.515
                helmet         19         51      0.805      0.784      0.812      0.512
              noHelmet         13         17      0.979      0.471      0.535      0.315
                noVest          7         13      0.614       0.49      0.637      0.277
