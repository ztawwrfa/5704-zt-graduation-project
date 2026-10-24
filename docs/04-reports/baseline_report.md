# Baseline 与问题分析报告

1. 课题问题定义
面向嵌入式边缘设备的轻量化目标检测模型优化与部署。

具体来说：YOLOv8n 在 VisDrone 无人机航拍数据集上虽然参数量仅约 3.0M，但直接部署到 Jetson Nano 这类算力、内存、功耗受限的边缘设备上时，推理延迟可能无法满足实时性要求。本课题的目标是：先通过真机 profile 找到瓶颈算子，再据此选择有针对性的轻量化方法（结构轻量化/通道裁剪/分辨率调整/部署框架优化），在精度损失可接受的前提下，实测证明边缘端延迟确实下降。

导师明确要求：不预设某种轻量化方式一定有效，改造动机必须有 profile 数据支撑。

2. 现有方案/系统
从已有文献归纳，当前轻量化目标检测在边缘部署方向的主要方案包括：

（1）轻量骨干替换。 MobileNet 系列（V2/V3）使用深度可分离卷积降低计算量，MobileNetV3 通过 NAS 在精度和效率之间取得平衡。ShuffleNet V2 通过 Channel Shuffle 和组卷积实现高效特征提取，但其论文核心观点是 FLOPs 不能直接代表实际速度，内存访问成本同样关键——这与导师强调的“参数量下降 ≠ 延迟下降”一致。GhostNet 通过廉价操作生成冗余特征，进一步降低卷积计算量。

（2）模型压缩。 通道剪枝通过去除冗余特征通道实现压缩，结构化剪枝则针对网络层和卷积核；量化将浮点参数转为 FP16 或 INT8，FP16 精度损失小，INT8 加速显著。但量化/蒸馏路线由杨敏负责，周涛的路线是结构轻量化 + 部署框架优化。

（3）VisDrone 上的已有工作。 有研究在 VisDrone2019 上对 YOLOv8n 做了系统评估：mAP@0.5 约 0.35，参数量 3.0M，GFLOPs 8.1。另一项工作 GS-YOLO 以 0.84M 参数量达到 33.6% mAP50 和 70.42 FPS，但其作者明确指出极端遮挡场景下存在明显检测失败。DSF-YOLO-P 基于 YOLOv8n 改进，在 VisDrone2019 上参数量降低 26%、计算量降低 45%，精度不变。这些工作为你的 Baseline 和后续改造提供了参照。

（4）边缘设备实测数据。 一项针对 YOLOv8n 在 Jetson Nano 上的实测研究显示：TensorRT FP16 格式下，Jetson Nano 平均吞吐约 12.0–14.5 FPS，中位推理延迟约 48.46 ms，GPU 利用率 99.4%。另一项研究指出，Jetson Nano 相比高性能集成 GPU 系统，能效（GFLOPs/W）高出约 38%，但延迟代价显著；要在嵌入式 GPU 上达到实时性能，必须结合精度量化和网络剪枝。这些数据是后续真机测速的对照基准。

3. 数据或硬件环境
数据集：VisDrone2019-DET。训练集 6471 张，验证集 548 张，38759 个实例，10 个类别。特点是小目标密集、类别不平衡、遮挡严重。验证集单图最多 317 个目标。

边缘设备：Jetson Nano（暂未到位，待借/待登记）。参考实测数据：TensorRT FP16 下约 12–14.5 FPS，延迟约 48ms。

当前替代环境：PC（RTX 4060 Laptop GPU），用于训练和 PC 单线程 CPU 模拟受限环境。

软件环境：Python 3.12.12，PyTorch 2.2.0，Ultralytics 8.4.165，CUDA 12.1（RTX 4060）。Jetson Nano 预计使用 JetPack 4.6.1（TensorRT 8.2.1）。

数据集路径：VisDrone.yaml（Ultralytics 内置自动下载与格式转换）。

4. Baseline 配置与初始结果
模型
YOLOv8n，参数量 3,007,598（约 3.0M），GFLOPs 8.1。

训练配置
epochs=30，batch=16，imgsz=640

optimizer=auto，lr0=0.01，lrf=0.01

mosaic=1.0，close_mosaic=10

seed=0，deterministic=True

设备：RTX 4060 Laptop GPU

结果目录：runs/detect/train-3，验证目录 runs/detect/val-2

验证协议
max_det=500（避免目标密集图被截断，VisDrone 单图最多 317 个目标）

正式 Baseline 指标（max_det=500）
指标	值
mAP50	0.286
mAP50-95	0.159
Precision	0.41
Recall	0.31
与文献对照：不同论文在 VisDrone 上报告的 YOLOv8n 结果存在差异（mAP50 从 0.286 到 0.35 不等），差异来源包括训练轮数、超参数、验证协议（max_det、conf 阈值）不同。你的 30 epoch Baseline 属于“短训练”配置，后续如需与论文严格对比，可考虑跑 100 epoch 完整版。

GPU 推理速度（RTX 4060，非边缘设备）
preprocess: 0.9 ms

inference: 6.4 ms

postprocess: 4.3 ms

PC 单线程 CPU 模拟（待跑）
尚未执行 benchmark_pc.py，延迟数字待填。

5. 目前发现的主要问题
（1）边缘设备未到位，真机延迟未知。 当前所有速度数字来自 RTX 4060 GPU，不能代表 Jetson Nano。文献实测 Jetson Nano 上 YOLOv8n TensorRT FP16 延迟约 48ms、FPS 约 12–14，但你的具体部署环境（JetPack 版本、TensorRT 配置、是否含 NMS）尚未确定。

（2）小目标与遮挡是 VisDrone 的核心难点。 YOLOv8n 在 bicycle（mAP50=0.062）、awning-tricycle（0.0985）等小目标类别上表现明显差于 car（0.713）。文献同样指出 VisDrone 存在“小目标像素占比低、特征表示不足、遮挡下性能显著下降”的问题，且极端遮挡场景是现有轻量化模型的共同失败模式。

（3）环境稳定性问题反复出现。 NumPy 2.x 与 OpenCV/torchvision 的 ABI 不兼容导致多次报错，虽然通过降级暂时解决，但后续导出 ONNX、跑 profile 时可能再次触发。建议在正式部署前用 Conda 锁定 NumPy 1.26.4。

（4）测速协议尚未固定。 导师要求“预热 N 次、取 M 次平均、注明是否含 NMS”。当前 PC 单线程模拟脚本已写好但未跑通，真机协议待设备到位后统一。

6. 下一步改什么、为什么
第一步：跑通 PC 单线程模拟，固定测速协议。 为什么：设备未到位，但测速协议（预热次数、测量次数、是否含 NMS）必须在 PC 阶段就定下来，设备到了直接沿用，保证数据可比。导师明确将此列为风险控制项。

第二步：确认 Jetson Nano 借用或 Orin Nano 替代方案。 为什么：没有真机，profile 无法进行，轻量化方法的选择就没有数据依据。文献显示 Jetson Nano 上 YOLOv8n 的延迟约 48ms，如果你的目标场景要求更高 FPS，需要提前判断 Nano 是否够用。

第三步：设备到位后，在真机上跑 Baseline 并做推理耗时 profile。 为什么：导师修订意见的核心是“先 profile 再选方法”。参数量下降不等于延迟下降，depthwise 卷积在某些后端上效率差。profile 结果决定你选主干替换、通道裁剪还是分辨率调整。

第四步：根据 profile 结果设计消融实验。 导师要求消融做“主干替换、通道裁剪比例、输入分辨率”三选二，每项至少 3 个取值。错误案例选小目标漏检和遮挡误检各一组。
