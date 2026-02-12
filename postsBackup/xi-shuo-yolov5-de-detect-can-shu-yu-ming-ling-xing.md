---
title: '细说yolov5的detect参数与命令行'
date: 2024-11-12 10:02:08
tags: [yolov5]
published: true
hideInList: false
feature: 
isTop: false
---
### detect常用命令行
```python
# 检测多张图片并保存检测结果；可作为自动检测的工具
python detect.py --weights model1.pt --source path/to/images --save-txt
#检测多张图片输出检测结果，其中检测结果中又置信度
python detect.py --weights model1.pt --source path/to/images --save-txt --save-conf
#检测视频流，每隔150帧检测一次，保存检测结果
python detect.py --weights vestAndHelmet.pt --source testVideo/video.mp4 --save-txt --vid-stride 150
 #检测视频流，每隔150帧检测一次，保存图片
python detect.py --weights vestAndHelmet.pt --source testVideo/video.mp4  --save-txt --save-crop --vid-stride 25
```

这是 YOLOv5 中 `detect.py` 的命令行参数定义。下面是每个参数的详细解释和翻译：

```python
def parse_opt():
    parser = argparse.ArgumentParser()
    
    # 权重文件路径，或者 Triton 推理服务的 URL（如果使用）。可以是多个权重文件。
    parser.add_argument('--weights', nargs='+', type=str, default=ROOT / 'yolov5s.pt', help='model path or triton URL')
    
    # 输入源，可以是文件、目录、URL、屏幕捕捉或摄像头。
    parser.add_argument('--source', type=str, default=ROOT / 'data/images', help='file/dir/URL/glob/screen/0(webcam)')
    
    # 数据集配置文件路径（可选）。默认是 COCO128 的 YAML 文件。
    parser.add_argument('--data', type=str, default=ROOT / 'data/coco128.yaml', help='(optional) dataset.yaml path')
    
    # 推理图像尺寸，指定高度和宽度。如果只提供一个值，会自动扩展为正方形尺寸。
    parser.add_argument('--imgsz', '--img', '--img-size', nargs='+', type=int, default=[640], help='inference size h,w')
    
    # 置信度阈值，决定检测框被认为是有效的置信度下限。默认是 0.25。
    parser.add_argument('--conf-thres', type=float, default=0.25, help='confidence threshold')
    
    # IoU（交并比）阈值，用于非极大值抑制（NMS）去除重叠框。默认是 0.45。
    parser.add_argument('--iou-thres', type=float, default=0.45, help='NMS IoU threshold')
    
    # 每张图片最多显示多少个检测结果。默认是 1000。
    parser.add_argument('--max-det', type=int, default=1000, help='maximum detections per image')
    
    # 设备设置，用于选择 GPU 或 CPU。例如 '0' 表示第一个 GPU，'cpu' 表示使用 CPU。
    parser.add_argument('--device', default='', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
    
    # 显示检测结果图像或视频。默认不显示。
    parser.add_argument('--view-img', action='store_true', help='show results')
    
    # 将检测结果保存为 .txt 文件。默认不保存。
    parser.add_argument('--save-txt', action='store_true', help='save results to *.txt')
    
    # 保存置信度到 .txt 文件中。与 `--save-txt` 配合使用。
    parser.add_argument('--save-conf', action='store_true', help='save confidences in --save-txt labels')
    
    # 保存裁剪的预测框图片。默认不保存。
    parser.add_argument('--save-crop', action='store_true', help='save cropped prediction boxes')
    
    # 不保存检测结果图片或视频。默认保存。
    parser.add_argument('--nosave', action='store_true', help='do not save images/videos')
    
    # 根据类别过滤检测结果。例如 `--classes 0` 只显示类别 0 的检测结果（比如人类），可以指定多个类别。
    parser.add_argument('--classes', nargs='+', type=int, help='filter by class: --classes 0, or --classes 0 2 3')
    
    # 类别无关的 NMS（非极大值抑制），即忽略类别进行框抑制。默认按照类别进行 NMS。
    parser.add_argument('--agnostic-nms', action='store_true', help='class-agnostic NMS')
    
    # 使用增强的推理（例如镜像翻转）。默认不启用。
    parser.add_argument('--augment', action='store_true', help='augmented inference')
    
    # 可视化模型特征图。默认不启用。
    parser.add_argument('--visualize', action='store_true', help='visualize features')
    
    # 更新所有模型。默认不启用。
    parser.add_argument('--update', action='store_true', help='update all models')
    
    # 设置项目保存的路径。默认保存到 `runs/detect/exp` 目录。
    parser.add_argument('--project', default=ROOT / 'runs/detect', help='save results to project/name')
    
    # 设置保存结果的名称。默认名称为 'exp'。
    parser.add_argument('--name', default='exp', help='save results to project/name')
    
    # 允许覆盖已存在的项目文件夹。默认会创建一个新文件夹。
    parser.add_argument('--exist-ok', action='store_true', help='existing project/name ok, do not increment')
    
    # 设置边界框的线条粗细，默认是 3 像素。
    parser.add_argument('--line-thickness', default=3, type=int, help='bounding box thickness (pixels)')
    
    # 隐藏类别标签。默认显示标签。
    parser.add_argument('--hide-labels', default=False, action='store_true', help='hide labels')
    
    # 隐藏置信度分数。默认显示置信度。
    parser.add_argument('--hide-conf', default=False, action='store_true', help='hide confidences')
    
    # 使用 FP16 半精度推理。默认使用全精度。
    parser.add_argument('--half', action='store_true', help='use FP16 half-precision inference')
    
    # 使用 OpenCV DNN 推理 ONNX 模型。默认不启用。
    parser.add_argument('--dnn', action='store_true', help='use OpenCV DNN for ONNX inference')
    
    # 设置视频帧跳跃间隔，默认为 1，表示不跳帧。该参数可以用来控制处理每隔几帧进行一次推理。
    parser.add_argument('--vid-stride', type=int, default=1, help='video frame-rate stride')
    
    opt = parser.parse_args()
    
    # 如果只提供了一个尺寸值，复制该值作为宽高值，确保尺寸为正方形。
    opt.imgsz *= 2 if len(opt.imgsz) == 1 else 1  # expand
    
    # 打印解析后的参数
    print_args(vars(opt))
    
    return opt
```

### 关键参数翻译总结：
1. **`--weights`**: 模型权重文件路径或 Triton URL
2. **`--source`**: 输入源（文件/目录/URL/屏幕/摄像头）
3. **`--data`**: 数据集配置文件路径
4. **`--imgsz`**: 推理图像大小（高度，宽度）
5. **`--conf-thres`**: 置信度阈值
6. **`--iou-thres`**: NMS 的 IoU 阈值
7. **`--max-det`**: 每张图像的最大检测数量
8. **`--device`**: 使用的设备（GPU 或 CPU）
9. **`--view-img`**: 显示检测结果
10. **`--save-txt`**: 保存检测结果为文本文件
11. **`--save-conf`**: 在保存的文本中保存置信度值
12. **`--save-crop`**: 保存裁剪的预测框图像
13. **`--nosave`**: 不保存结果图像或视频
14. **`--classes`**: 通过类别过滤检测结果
15. **`--agnostic-nms`**: 类别无关的 NMS
16. **`--augment`**: 增强推理
17. **`--visualize`**: 可视化特征图
18. **`--update`**: 更新所有模型
19. **`--project`**: 设置保存结果的项目路径
20. **`--name`**: 设置保存结果的文件夹名称
21. **`--exist-ok`**: 允许覆盖现有项目文件夹
22. **`--line-thickness`**: 边界框线条粗细
23. **`--hide-labels`**: 隐藏检测结果中的标签
24. **`--hide-conf`**: 隐藏置信度分数
25. **`--half`**: 使用 FP16 半精度推理
26. **`--dnn`**: 使用 OpenCV DNN 推理 ONNX 模型
27. **`--vid-stride`**: 视频帧跳跃间隔
    

置信度