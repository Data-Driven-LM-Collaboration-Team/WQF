# 框架 
选择模型 -->  数据增强  -->  训练

参考了 `roboflow-python` 的 `Workspace -> Project -> Version` 组织方式，所有步骤都在本地完成，不依赖在线 API。

按“模型家族”选择底层实现，并统一管理：

- 数据集准备
- 离线数据增强
- 训练
- 评估
- 推理

- 任务类型：目前聚焦 `CV` 目标检测。
- 数据集输入：通过显式传入 `train / val / test` 三组图片目录与标注文件（coco格式）
- 离线增强：先对训练集做离线增强，再把处理后的数据交给模型训练流程。
- 模型家族：当前支持 `rtdetr` 和 `rfdetr`。
- 模型适配：不同模型家族由各自适配器负责生成配置、训练命令、评估命令和推理命令。

#  Agent 

Data Card ：提供数据集各种特征
<img width="1298" height="436" alt="image" src="https://github.com/user-attachments/assets/56bbb66f-79de-4394-81d4-d6840ceab65e" />

让 Agent 先用规则决策
例如:数据集很小，优先选择微调预训练模型的方式，并且充分利用好数据增强和正则化等。 

Model Card 提供模型的特征，并且让Agent先选择模型家族，再选择具体模型
<img width="1274" height="412" alt="image" src="https://github.com/user-attachments/assets/48639c08-06c5-40dd-b641-c19e389948e1" />

参数优化通过AutoResearch来完成


训练 预训练 微调
# YOLO系列
参考Ultralytics YOLO使用文档 https://docs.ultralytics.com/zh/modes/train/ 

# RT-DETR V2
## 数据预处理\数据增强
*dataloader.yml* 
A. train_dataloader.dataset.transforms.ops （增强链） 
- RandomPhotometricDistort(p) ：颜色扰动概率。 
- RandomZoomOut(fill) ：随机缩小+填充。 
- RandomIoUCrop(p, ...默认超参) ：IoU 裁剪概率与策略（实现支持 p ： RandomIoUCrop ）。 
- RandomHorizontalFlip ：水平翻转。 
- Resize(size=[H,W]) ：训练分辨率（非常关键）。 
- ConvertPILImage(dtype, scale) ：输入数值尺度（0~1）与 dtype。 
- ConvertBoxes(fmt, normalize) ：框格式/是否归一化（必须与模型/损失约定一致）。 
  
B. train_dataloader.dataset.transforms.policy （增强策略，强影响） 
- policy.name: stop_epoch + policy.epoch + policy.ops ：指定到某个 epoch 后停用某些强增强（实现： Compose.stop_epoch_forward ）。 
  
C. train_dataloader.collate_fn （多尺度训练） 
- collate_fn.scales ：多尺度候选尺寸列表（训练时随机采样 resize）。 
- collate_fn.stop_epoch ：到某 epoch 停止多尺度（实现： BatchImageCollateFuncion ）。 

D. Batch 相关（影响优化动态） 
- train_dataloader.total_batch_size （或 batch_size ）：batch size（强影响，通常需要配套调 LR/WD）。 

## 优化器/调度器 
*optimizer.yml* 
- epoches ：训练总 epoch 数（强影响）。 
- clip_max_norm ：梯度裁剪阈值（影响稳定性与最终点，det_engine.py ）。 
- optimizer.type ：AdamW...。 
- optimizer.lr / betas / weight_decay ：核心优化超参。 
- optimizer.params ：分参数组正则匹配。匹配逻辑见： get_optim_params 
- lr_scheduler.(type, milestones, gamma) ：学习率衰减策略（这里每 epoch step，见： det_solver.py ）。 
- lr_warmup_scheduler.(type, warmup_duration) ：warmup 步数（按 iteration 生效，见： warmup.py ）。 

## 模型架构 
rtdetrv2_pytorch/configs/rtdetrv2/include/rtdetrv2_r50vd.yml 
### Backbone - PResNet 
- PResNet.depth ：18/34/50/101；主干容量上限（精度/速度权衡）。
- PResNet.variant ： d 等；影响 stem/下采样结构（会影响精度）。 
- PResNet.pretrained ：是否加载预训练；对收敛与最终精度影响很大（尤其小数据）。 
- PResNet.freeze_at ：冻结到第几个 stage（-1 不冻结，0 会冻结 stem+前若干层）；影响可学习能力与收敛速度。 
- PResNet.freeze_norm ：把 BN 变成 FrozenBN（统计不更新）；对小 batch 或迁移训练经常影响明显。 
- PResNet.return_idx ：输出哪些 stage 特征；会改变后续 encoder 输入特征层级（影响） 
### HybridEncoder 
- HybridEncoder.in_channels ：输入通道（要与 backbone 输出匹配）。 
- HybridEncoder.hidden_dim ：投影到的通道维度（对精度/速度/显存都强影响）。 
- HybridEncoder.use_encoder_idx ：哪些特征层走 Transformer encoder（影响建模能力）。 
- HybridEncoder.num_encoder_layers / nhead / dim_feedforward / dropout / enc_act ：Transformer encoder 结构超参（影响容量与正则）。 
- HybridEncoder.expansion / depth_mult / act ：FPN/PAN 的 CSPRepLayer 宽度/深度倍率与激活（影响检测头特征质量）。 
### Decoder  
- RTDETRTransformerv2.num_layers：decoder 层数（强影响）。  
- RTDETRTransformerv2.num_queries：查询数量（影响召回上限与速度；也影响匹配与训练动态）。 
- RTDETRTransformerv2.hidden_dim / nhead / dim_feedforward / dropout / activation：Transformer 主体维度与正则（强影响）。 
- RTDETRTransformerv2.num_levels / feat_channels / feat_strides：使用多少层特征与其 stride（影响多尺度能力）。 
- RTDETRTransformerv2.num_points：deformable attention 每层采样点数（影响精度/速度）。 
- RTDETRTransformerv2.cross_attn_method：default / discrete（注意实现里 discrete 会冻结 sampling_offsets 参数：MSDeformableAttention，会明显改变可学习性）。 
- RTDETRTransformerv2.query_select_method：default / one2many / agnostic（影响 query 选择/训练方式）。 
- 去噪训练：num_denoising、label_noise_ratio、box_noise_scale（影响收敛与最终精度，尤其早期稳定性）。

# AnomalyCLIP
## 超参
- epoch
- lr
- batch_size
## 数据
--image_size
## 模型
- features_list(选择从 ViT 的哪些层导出 patch tokens 来做局部 anomaly map)
- feature_map_layer(从第几个导出的特征层开始参与局部 anomaly map/loss)

# RF-DETR
微调
## 数据增强
预设的增强方案 
预设名称	最适用场景 

AUG_CONSERVATIVE	小规模数据集（500 张图像以下） 

AUG_AGGRESSIVE	大规模数据集（2000 张以上图像） 

AUG_AERIAL	卫星 / 航拍图像 

AUG_INDUSTRIAL	工业制造 / 质检数据 

自定义：几何变换、像素级变换等 

model.train(dataset_dir="...", aug_config={"HorizontalFlip": {"p": 0.5}}) 
## Model
Nano/Small/Medium/Large/XLarge/2XLarge
## 训练
epochs,batch_size,lr,lr_encoder...
# Grounding DINO
微调
