2027届本科毕业论文

基本信息

姓名：周涛

学号：202321020429

专业：人工智能

指导教师：曾维

毕业论文题目：面向嵌入式边缘设备的轻量化目标检测模型优化与部署研究

研究方向：轻量化目标检测 / 边缘智能

一、研究问题
研究对象是部署在嵌入式边缘设备上的目标检测模型，尤其是以 YOLOv8n 为代表的轻量级检测网络。

当前问题在于：主流检测模型参数量大、计算量高，难以在算力、内存和功耗受限的边缘设备上实时运行；直接部署会出现推理延迟高、帧率低、内存占用大等问题。

本论文准备解决什么问题：通过通道剪枝与 INT8/FP16 量化等轻量化方法，在可接受的精度损失范围内显著降低模型的参数量、模型大小与推理延迟，并实际部署到边缘设备上验证。

在无人机航拍场景（VisDrone 数据集）上建立标准 Baseline，统一比较 mAP、Precision、Recall、参数量、模型大小、推理延迟/FPS、内存与功耗。

通过消融与参数实验，量化每种轻量化方法的独立贡献，形成一套可复现的轻量化检测部署流程。

二、最低完成要求
□ Baseline / 基础系统：YOLOv8n 在 VisDrone2019 上跑通训练，得到第一组可复现指标
□ 核心方法或关键机制：通道剪枝（BN gamma 剪枝）+ INT8/FP16 量化
□ 对比实验：Baseline vs 剪枝 vs 量化 vs 完整方案
□ 消融/性能测试：剪枝率参数实验、量化前后精度-效率对比
□ 错误或异常情况分析：量化精度掉点、剪枝后通道不匹配、设备端部署失败案例
□ 完整毕业论文
三、拓展目标
□ 多节点协同推理
□ 进一步剪枝/量化（如混合精度量化、结构化剪枝）
□ 强化学习任务调度
四、技术路线
总体流程：VisDrone2019 数据集 → YOLOv8n Baseline 跑通 → 通道剪枝 → INT8/FP16 量化 → Jetson 边缘设备部署 → 消融与参数实验 → 完整对比表。

详见 docs/01-topic/technical_route.md。

五、当前进展
当前阶段：仓库初始化完成，文档模板建立，准备进入数据集与 Baseline 阶段。

最近完成：

建立毕业论文个人仓库并完成初始化

填写 README、topic_confirm、task_requirements、technical_route

确定检测场景为无人机航拍，数据集为 VisDrone2019

当前问题：

边缘设备型号待确认（Jetson Nano 不支持 INT8，需确认实验室可用设备）

VisDrone 数据集尚未下载与格式转换

下一步：

下载 VisDrone2019-DET 并转换为 YOLO 格式

跑通 YOLOv8n Baseline，记录第一组指标

开始文献核验与精读

六、主要实验结果
Experiment	Result	Status
YOLOv8n Baseline (VisDrone)	mAP50=<待填> mAP50-95=<待填> P=<待填> R=<待填>	未开始
+ 通道剪枝	参数量/模型大小/mAP变化	未开始
+ INT8/FP16 量化	延迟/内存/精度变化	未开始
剪枝 + 量化（完整方案）	全部指标	未开始
剪枝率参数实验 [0.2,0.4,0.6]	精度-效率曲线	未开始
七、仓库目录说明
text
5704-zt-graduation-project/
├── README.md
├── .gitignore
├── docs/
│   ├── 01-topic/
│   │   ├── topic_confirm.md
│   │   ├── task_requirements.md
│   │   └── technical_route.md
│   ├── 02-references/
│   │   └── verified_references.md
│   ├── 03-design/
│   │   └── experiment_design.md
│   └── 04-reports/
│       └── baseline_report.md
├── src/          # 训练、剪枝、量化、部署脚本
└── results/      # 训练日志、权重、实测数据
八、本人主要贡献
本人完成的代码、实验、数据和论文工作：

数据集准备与格式转换脚本

YOLOv8n Baseline 训练与评估

通道剪枝的稀疏训练、剪枝与微调流程

INT8/FP16 量化与 TensorRT engine 导出

Jetson 边缘设备部署与实测

消融与参数实验设计与执行

论文撰写

九、参考项目与第三方代码
项目：Ultralytics YOLOv8

URL：https://github.com/ultralytics/ultralytics

License：AGPL-3.0

本项目修改内容：使用其训练与导出接口，不修改核心网络结构；剪枝与量化在其基础上实现

项目：<剪枝工具，待确认>（如 JasonSloan/yolov8-prune）

URL：<待填>

License：<待填>

本项目修改内容：<待填>

十、环境与复现
Python：< 3.13>

PyTorch：<待填>

Ultralytics：<待填>

CUDA / cuDNN：<待填>

TensorRT：<待填>

边缘设备：Jetson <型号待确认>

OS：<待填，如 Ubuntu 20.04>

数据集：VisDrone2019-DET

随机种子、训练命令、导出命令：<跑通后补充>
