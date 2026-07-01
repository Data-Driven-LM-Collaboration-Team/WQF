# 大纲
## Introduction
然而，许多 LLM-based AutoML 系统仍然把 LLM 放在流程生成的位置：LLM 读取任务描述，生成一套配置或计划，然后由系统执行。这样的流程虽然提高了自动化程度，但与真实机器学习研究仍存在差距。真实研究者不会一次性决定所有配置，而是会观察数据分布和训练动态，提出关于模型行为的假设，设计控制变量的实验，在失败后分析原因，并基于证据持续调整方向。换言之，**机器学习研究是一种迭代式、证据驱动且可追踪的实验过程**。 

从系统角度看，自主机器学习研究还面临一个基础问题：LLM 的研究意图必须能够被稳定执行，并且实验过程必须可复现、可观测和可审计。如果直接让 LLM 修改训练脚本，容易引入代码错误、配置不一致和结果不可复现等问题；如果只有实验平台而没有研究型 Agent，系统又会停留在工具自动化层面，无法主动解释失败、形成假设或进行研究转向。因此，下一代 AutoML 系统需要同时设计平台层和 Agent 层：平台层提供稳定的机器学习实验执行环境，Agent 层负责研究决策和证据推理。

# 数据集
## Table: Datasets and Tasks Overview

| Task Type              | Dataset              | Task Description        | Evaluation Metric        |
|------------------------|---------------------|------------------------|--------------------------|
| Image Classification  | CIFAR-100           | 100-class image classification | Top-1 Accuracy |
| Object Detection      | VisDrone            | Aerial object detection in drone imagery | mAP@[0.5:0.95] |
| Image Segmentation    | Oxford-IIIT Pet     | Pixel-level semantic segmentation of pet images | mIoU |
| Text Classification   | Ecomm Dataset       | E-commerce text category classification | Accuracy |
| Text Classification   | Entail Dataset      | Natural language inference / entailment classification | Accuracy |
| Node Classification   | Cora                | Citation network node classification | Accuracy |
| Link Prediction       | OGBL-Collab        | Temporal collaboration link prediction | Hits@K / AUC |
| Graph Classification  | MUTAG               | Molecular graph classification | Accuracy | 

# 实验 
<img width="913" height="390" alt="image" src="https://github.com/user-attachments/assets/3ecdd70f-1442-4cd9-86a4-20de2c38b254" />

## Table 1: Vision  Results

| Method            | CIFAR-100 Top-1 Acc. ↑ | VisDrone mAP@[0.5:0.95] ↑ | Oxford-IIIT Pet mIoU ↑ |
|------------------|------------------------|---------------------------|-------------------------|
| Human Models     | 0.783                  | 0.251                     | 0.795                   |
| AutoML-Agent     | 0.788                  | *0.363*                   | *0.803*                 |
| AutoGluon        | **0.926**              | 0.215                     | 0.710                   |
| Claude Code      |                        |                           |                         |
| DeepAuto (Ours)  | *0.825*                | **0.385**                 | **0.814**               |


## Table 2: Text and Graph Results

| Method               | Ecomm Text Cls. ↑ | Entail Text Cls. ↑ | Cora Node Cls. ↑ | OGBL-Collab Link Pred. ↑ | MUTAG Graph Cls. ↑ |
|---------------------|-------------------|--------------------|------------------|---------------------------|-------------------|
| Human Models        | —                 | —                  | —                | —                         | —                 |
| AutoML-Agent        | —                 | —                  | —                | —                         | —                 |
| AutoGluon           | —                 | —                  | N/A              | N/A                       | N/A               |
| Claude Code         | —                 | —                  | —                | —                         | —                 |
| DeepAuto (Ours)     | —                 | —                  | —                | —                         | —                 |

## Table 1. 数据集

| Modality | Task                     | Dataset         | Metric         |特点  |
| -------- | ------------------------ | --------------- | -------------- |-------------- |
| Image    | Classification           | CIFAR-100       | Top-1 Accuracy |比coco小，但是又足够训练 |
| Image    | Object Detection         | VisDrone        | mAP@50:95      |大量小目标、复杂背景、类别不平衡，存在研究问题 |
| Image    | Instance Segmentation    | Oxford-IIIT Pet | mIoU           |比coco小，训练成本低 |
| Graph    | Node Classification      | Cora            | Accuracy       |经典数据集 |
| Graph    | Link Prediction          | OGBL-Collab     | Hits@50        |经典数据集 |
| Graph    | Graph Classification     | MUTAG           | Accuracy       |经典数据集 |
| Text     | Text Classification      | AG News         | Accuracy       | |
| Text     | Named Entity Recognition | CoNLL-2003      | Entity F1      | |
| Text     | Retrieval / Ranking      | MS MARCO        | MRR@10         | |



# 大纲
## 研究背景

AutoML 的目标是自动完成机器学习建模流程。 

现有 AutoML / LLM AutoML 多数还是偏 pipeline 配置生成。 

问题是：它们缺少像研究员一样的实验分析、失败总结和迭代优化。 


## 平台：

数据预处理 

数据增强 

模型管理 

训练模块 

测试评估 

可视化、实验记录与追踪 

## Agent
**Agent 不是 pipeline configurator，而是 ML researcher**

理解数据特征； 

提出实验方向； 

设计实验； 

监控训练过程； 

分析结果； 

根据失败或提升调整下一轮策略。

贡献1:本文设计了 DeepAuto，一个端到端自主机器学习研究平台，将数据预处理、数据增强、模型管理、训练、测试、可视化和实验追踪统一到一个可执行环境中。 

贡献2：在平台基础上，本文设计了 LLM Research Agent，使其不再只是 pipeline 配置生成器，而是能够像机器学习研究员一样进行数据理解、假设提出、受控实验、结果分析和策略调整。 

贡献3：本文通过代表性机器学习任务验证 DeepAuto，说明可执行实验平台与研究员式 Agent 的结合能够提升 AutoML 过程的性能、可解释性和可追踪性。 


| 数据集       | 图像数  | 类别数 |  核心挑战         |
| :----------- | :------ | :----- |  :--------------- |
| VisDrone-DET | ~8,600  | 10     |  密集小目标，航拍 |
| Pascal VOC   | ~16,500 | 20     |  标准Benchmark    |
| ExDark       | ~7,300  | 12     | 极端低光照       |
| 布匹缺陷     | 917     | 12      |           |


| Baseline       | 类别  |
| :----------- | :------ | 
| YOLO Default | 人类默认配置  | 
| RT-DETR Default   | 人类默认配置 |
| Optuna Search       | 传统 AutoML  | 
| AutoML-Agent     | LLM Agent AutoML   |  

# 布匹
## Agent给出方案: 
本方案针对**纺织品瑕疵检测数据集（训练779张、验证138张，共917张、12个类别）**制定首轮训练策略，数据集痛点：极小样本体量、瑕疵小目标占比34%、极端类别失衡（120.5:1）、暗光成像、瑕疵细粒度区分难、框长宽比波动大，推理速度要求均衡。

1. **训练方式：全量微调（full_finetune）**
样本落在500~5000区间，纺织品瑕疵和COCO基础图像特征（边缘、纹理）相通，无需从零训练或冻结骨干微调，依靠COCO预训练权重初始化。

2. **模型选用RF-DETR-M（自动拉取预训练权重）**
依托DINOv2骨干适配小目标、细粒度分类与密集瑕疵场景；备选YOLO26系列，但DETR架构对瑕疵检测精度更优。

3. **输入分辨率1280**
小目标占比超30%，高分辨率保障细小瑕疵特征留存，适配暗光下微弱缺陷。

4. **训练轮次200轮**
千张以内小样本+Transformer模型收敛偏慢，选用全微调上限轮次保障充分收敛。

5. **优化器AdamW**
DETR专属配置：初始学习率0.0001，配合对应动量、权重衰减，防止预训练Transformer权重崩坏。

6. **数据增强适配场景**
- 暗光优化：提高亮度增强系数，启用直方图均衡、视网膜变换；
- 框尺寸波动：缩放系数下调至0.7；
- 细粒度分类：禁用Mixup/CutMix避免类别混淆；
- 保留默认Mosaic提升小目标训练效果。

7. **风险说明**
- 高风险：极度类别不均衡，小众瑕疵类别检测指标大概率偏低；
- 中风险：数据量偏少存在过拟合隐患，无LoRA轻量化微调手段只能全参训练；
- 后续迭代方向：出现过拟合则下调马赛克增强、微调学习率与训练轮次。

![Uploading image.png…]()


