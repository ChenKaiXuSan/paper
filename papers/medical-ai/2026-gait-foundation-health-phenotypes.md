---
title: "A Gait Foundation Model Predicts Multi-System Health Phenotypes from 3D Skeletal Motion"
authors: "Adam Gabet, Sarah Kohn, Guy Lutsker, Shira Gelman, Anastasia Godneva, Gil Sasson, Arad Zulti, David Krongauz, Rotem Shaulitch, Assaf Rotem, Ohad Doron, Yuval Brodsky, Adina Weinberger, Eran Segal"
venue: "arXiv"
year: 2026
reading_date: 2026-09-01
status: skimmed
tags:
  - clinical-gait
  - foundation-model
  - self-supervised
  - skeletal-motion
  - digital-biomarker
  - health-phenotyping
---

# A Gait Foundation Model Predicts Multi-System Health Phenotypes from 3D Skeletal Motion

## 基本信息

- **作者：** Adam Gabet, Sarah Kohn, Guy Lutsker, Shira Gelman, Anastasia Godneva, Gil Sasson, Arad Zulti, David Krongauz, Rotem Shaulitch, Assaf Rotem, Ohad Doron, Yuval Brodsky, Adina Weinberger, Eran Segal
- **会议/期刊：** arXiv
- **年份：** 2026
- **阅读日期：** 2026-09-01
- **阅读状态：** `skimmed`
- **标签：** `clinical-gait`, `foundation-model`, `self-supervised`, `skeletal-motion`, `digital-biomarker`, `health-phenotyping`
- **论文：** https://arxiv.org/abs/2603.25283
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作用 3,414 名成人、351 小时 3D skeletal motion 预训练 GaitMAE，将 gait 从单一疾病指标扩展为可预测多系统健康表型的 foundation representation，并显示 self-supervised motion embedding 对年龄、BMI、内脏脂肪以及大量跨系统 phenotype 的预测明显强于传统 engineered gait features。

## 研究问题与动机

传统临床 gait analysis 往往围绕单一疾病或少量人工定义步态参数，例如 cadence、step length、joint angle 等。这种做法容易丢失复杂的全身时空协同模式，也很难回答一个更基础的问题：人体运动是否包含超越传统 gait parameters 的、可反映多个生理系统状态的潜在数字生物标志物？

作者因此把 gait representation learning 视为 foundation-model 问题，通过大规模自监督 3D skeletal motion 预训练获得通用 embedding，再测试其能否预测年龄、身体组成、代谢、睡眠、心血管、生活方式等广泛 phenotype，而不是只针对某一个 diagnosis 做 supervised classification。

## 核心方法

### 1. 大规模 3D skeletal motion cohort

数据来自 Human Phenotype Project，共 **3,414 名成人**，年龄 20–79 岁，其中 1,652 名男性、1,762 名女性，平均年龄约 52.5 ± 10.1 岁，平均 BMI 约 25.7 ± 3.8。共获得约 **351 小时** 3D skeletal motion。

受试者使用单台 Azure Kinect depth camera 完成五种标准化 motor tasks，包括 treadmill walking（3 km/h）、sit-to-stand 等。输入 skeleton sequence 采用 `(X, Y, Z, confidence)` 四通道关节表示。

### 2. GaitMAE 自监督预训练

GaitMAE 基于 dual-stream spatiotemporal Transformer / DSTformer，并受到 MotionBERT 类架构启发。预训练采用 denoising masked autoencoder：

- spatial joint-level masking；
- temporal masking；
- 对输入加入 Gaussian noise；
- 重建完整 3D skeletal sequence。

优化目标同时包含 MPJPE、normalized MPJPE 与 velocity error，以避免模型只恢复静态关节位置而忽略时间动态。预训练重建 MPJPE 约为 **0.008 m（8 mm）**。

### 3. Subject-level representation 与多任务融合

每种 motor activity 的 frame/token representation 经过 hierarchical pooling 得到 subject-level embedding。不同任务产生的预测还可通过 Gait Fusion 进行组合，用于 phenotype prediction。

## 数据集与评价指标

主要 cohort：

- 3,414 人；
- 20–79 岁；
- 351 h 3D skeletal motion；
- 单 Azure Kinect；
- 五种标准化 motor tasks。

作者对 **3,210 个 phenotype** 进行预测，覆盖 18 个生理/行为系统，并按 sex-specific model 分析。主要评价包括 Pearson correlation、AUC，以及相对于传统 engineered gait features 的增益，并执行多重比较/FDR 控制。

## 主要结果

GaitMAE representation 对多个基础健康指标表现很强：

- **年龄：**combined prediction 相关系数约 `r=0.69`；男性 `0.691 ± 0.002`，女性 `0.682 ± 0.002`。传统 engineered gait + height 对应约为男性 `0.455`、女性 `0.512`。
- **BMI：**男性 `r=0.882`，女性 `r=0.898`。
- **Visceral Adipose Tissue (VAT)：**男性 `r=0.817`、女性 `r=0.795`，而 engineered baseline 约为男性 `0.518`、女性 `0.467`。
- **Sex classification：**AUC 约 `0.99 ± 0.002`。

在 phenotype-wide 分析中，**3,210 个 phenotype 中有 1,980 个在 FDR 校正后仍能被 gait representation 显著预测**。即使进一步调整年龄、BMI、VAT 和身高，男性全部 18 个系统、女性 17/18 个系统仍存在独立预测增益。

Anatomical ablation 还显示不同身体区域携带不同类型信息：legs 对 metabolic / frailty signals 更重要，而 torso 对 sleep / lifestyle 等信息也有明显贡献。

## 优点

- **规模明显大于一般临床 gait study：**3,414 人与 351 h motion 支撑了真正意义上的 representation learning。
- **从 disease-specific 走向 phenotype-wide：**不再只证明 gait 能预测一个 diagnosis，而是系统检验 gait 作为全身数字生物标志物的可能性。
- **self-supervised：**大部分 motion 表征不依赖疾病标签，可用于后续多任务迁移。
- **包含 velocity objective：**使 representation 更关注运动动态，而不是单纯静态骨架形态。
- **解剖区域消融：**提供一定程度的结构解释性，而不仅仅报告黑盒预测精度。

## 局限

- cohort 主要是以色列的 Ashkenazi Jewish 成人，跨国家、种族与临床人群的外部泛化尚未验证。
- 数据依赖 Azure Kinect；迁移到 monocular RGB、2D keypoints、其他 markerless pose estimator 或不同 skeleton definition 仍需要 domain adaptation。
- 当前结果主要是 cross-sectional association，不能说明 gait 与 phenotype 之间存在因果关系。
- 解释性目前主要停留在四个 anatomical groups，尚未精细到具体 joint、phase 或 gait event。
- 作为真正的临床 screening system，还缺乏 prospective clinical validation。
- 本次没有从允许的一手来源核验到公开代码、公开 dataset 或独立项目主页。

## 个人评价

这篇论文的重要性在于重新定义了 gait model 的目标：不是“训练一个疾病分类器”，而是先学习一个具有系统健康信息的通用 motion representation，再针对下游临床任务迁移。这和传统直接用 RGB / keypoints 做 diagnosis 的思路差别很大。

不过高相关性并不等于临床因果性。年龄、BMI、body shape 与 motion 本身高度耦合，因此 phenotype prediction 中有多少真正来自动态 gait pattern、多少来自静态 anthropometric signal，需要更细的控制实验与 longitudinal validation。

## 与我的研究关联

**推断：**对临床 gait / spinal disorder video analysis，可以将现有 pipeline 从：

`RGB / keypoints → supervised disease classification`

扩展为：

`large-scale pose sequence → masked self-supervised pretraining → gait foundation embedding → disease / severity / phenotype prediction`

一个最低成本的实验可以比较：

1. engineered gait features；
2. task-specific pose encoder；
3. masked reconstruction pretrained pose encoder；
4. masked reconstruction + velocity objective；
5. 上述 embedding 与 RGB / optical flow / biomechanical variables 的 multimodal fusion。

同时应加入 cross-camera / cross-site / cross-pose-estimator validation，以确认 foundation representation 是否真正比单数据集 supervised model 更稳健。

对于 monocular video，还可以结合 OpenCap Monocular 的 constrained kinematics/kinetics，把 self-supervised gait embedding 与 joint angles、GRF proxy、joint moment 等显式 biomechanics feature 融合，从而提升解释性。

## 后续阅读

- 检查 GaitMAE embedding 在真实疾病 cohort 中是否仍保持 phenotype transfer 能力。
- 研究 Azure Kinect 3D skeleton → monocular RGB/2D pose 的 representation distillation / domain adaptation。
- 比较 static anthropometric information 与 dynamic gait information 对 phenotype prediction 的独立贡献。
- 将 masked reconstruction + velocity objective 与周期/phase-aware gait representation 结合。
