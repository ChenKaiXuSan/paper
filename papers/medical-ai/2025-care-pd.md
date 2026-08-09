---
title: "CARE-PD: A Multi-Site Anonymized Clinical Dataset for Parkinson's Disease Gait Assessment"
authors: "Vida Adeli, Ivan Klabučar, Javad Rajabi, Benjamin Filtjens, Soroush Mehraban, Diwei Wang, Hyewon Seo, Trung-Hieu Hoang, Minh N. Do, Candice Muller, Claudia Neves de Oliveira, Daniel Boari Coelho, Pieter Ginis, Moran Gilat, Alice Nieuwboer, Joke Spildooren, J. Lucas McKay, Hyeokhyen Kwon, Gari Clifford, Christine D. Esper, Stewart A. Factor, Imari Genias, Amirhossein Dadashzadeh, Leia Shum, Alan Whone, Majid Mirmehdi, Andrea Iaboni, Babak Taati"
venue: "NeurIPS 2025 Datasets and Benchmarks Track"
year: 2025
reading_date: 2026-08-10
status: skimmed
tags:
  - medical-ai
  - gait-analysis
  - parkinsons-disease
  - 3d-human-motion
  - representation-learning
  - domain-generalization
---

# CARE-PD: A Multi-Site Anonymized Clinical Dataset for Parkinson's Disease Gait Assessment

## 基本信息

- **作者：** Vida Adeli, Ivan Klabučar, Javad Rajabi, Benjamin Filtjens, Soroush Mehraban, Diwei Wang, Hyewon Seo, Trung-Hieu Hoang, Minh N. Do, Candice Muller, Claudia Neves de Oliveira, Daniel Boari Coelho, Pieter Ginis, Moran Gilat, Alice Nieuwboer, Joke Spildooren, J. Lucas McKay, Hyeokhyen Kwon, Gari Clifford, Christine D. Esper, Stewart A. Factor, Imari Genias, Amirhossein Dadashzadeh, Leia Shum, Alan Whone, Majid Mirmehdi, Andrea Iaboni, Babak Taati
- **会议/期刊：** NeurIPS 2025 Datasets and Benchmarks Track
- **年份：** 2025
- **阅读日期：** 2026-08-10
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `gait-analysis`, `parkinsons-disease`, `3d-human-motion`, `representation-learning`, `domain-generalization`
- **论文：** [arXiv](https://arxiv.org/abs/2510.04312) · [NeurIPS 2025](https://neurips.cc/virtual/2025/poster/121554)
- **代码：** [TaatiTeam/CARE-PD](https://github.com/TaatiTeam/CARE-PD)
- **数据集：** [CARE-PD Project / Data Portal](https://neurips2025.care-pd.ca/)
- **项目主页：** [CARE-PD](https://neurips2025.care-pd.ca/)

## 一句话总结

CARE-PD 将来自多个临床中心、由 RGB 视频或 motion capture 采集的帕金森病步态统一转换为匿名化 SMPL 三维人体运动，并用跨数据集、留一数据集等严格协议验证运动表征学习在 UPDRS gait severity 预测和三维运动预训练中的价值，为临床步态 AI 提供了比单中心数据更接近真实泛化问题的公开基准。

## 研究问题与动机

临床步态 AI 的一个长期瓶颈是数据量小、采集设备不同、中心间协议不一致，而且临床评分分布通常不平衡。即使模型在某一个医院或数据集内部取得较高准确率，也不代表它可以迁移到新的中心、设备和患者群体。

CARE-PD 的目标不是再建立一个单独的小型帕金森病队列，而是将来自 8 个临床中心的 9 个 cohort 统一到同一种匿名化三维运动表示中，并专门设计跨数据集泛化实验，从而让“临床运动表征是否具有可迁移性”成为可以系统比较的问题。

## 核心方法

### 1. 多中心数据统一为 SMPL 运动

原始数据来自 RGB 视频或 motion capture。作者通过统一 preprocessing pipeline 将不同来源的数据转换为 30 Hz 的匿名化 SMPL mesh motion。对于视频数据，人体运动由视频人体重建流程恢复；对于 motion-capture 数据，则将运动学信息拟合到统一的人体模型。

这种设计的意义在于，下游模型不再直接面对不同相机格式、不同 marker skeleton 或不同视频分辨率，而是使用统一的三维人体运动表示进行训练和评估。

### 2. 临床评分预测

主要临床任务是根据步态序列预测 Unified Parkinson's Disease Rating Scale（UPDRS）的 gait score。作者不仅做普通的同数据集训练测试，还设计了四类协议：

- within-dataset；
- cross-dataset；
- leave-one-dataset-out（LODO）；
- multi-dataset in-domain adaptation（MIDA）。

这些设置分别测试同中心性能、直接跨中心迁移、完全未见中心泛化，以及目标中心有少量域内数据时的适应能力。

### 3. Learned motion representation 与临床手工特征比较

论文比较多个通用人体运动 encoder，包括 POTR、MixSTE、PoseFormerV2、MotionBERT、MotionAGFormer、MotionCLIP 和 MoMask，并与传统 gait feature baseline 比较。传统特征包括 cadence、step length、step width、step time、walking speed、margin of stability、foot lifting、arm swing 和 stoop 等，再使用 Random Forest 进行预测。

因此这篇论文不只是“数据集介绍”，还直接比较了深度时空表征与临床可解释 gait feature 在跨域场景中的作用。

### 4. 三维运动预训练任务

CARE-PD 还定义了 2D-to-3D keypoint lifting 和 full-body 3D reconstruction 等 motion pretext tasks，用于检验临床步态数据能否成为人体运动模型的预训练资源。论文报告 CARE-PD 预训练可以显著改善后续三维运动恢复和 PD severity prediction。

## 数据集与评价指标

### 数据规模

- **来源：** 9 个 cohort，来自 8 个国际临床中心。
- **步态片段：** arXiv v1 表格合计 8,477 个 walking segments。
- **参与者：** arXiv v1 数据表合计为 362 名；项目页面后续版本曾显示 363 名参与者，因此这里将其视为版本更新造成的计数差异，而不强行统一为一个数字。
- **采集方式：** RGB video 与 motion capture 混合来源。
- **统一表示：** anonymized SMPL motion，统一到 30 Hz。

数据涵盖 PD-GaM、BMCLab、T-SDU-PD、3DGait、KUL-DT-T、DNE、E-LC、T-SDU 和 T-LTC 等 cohort。不同中心在设备、任务设置、受试者组成和 UPDRS 分布上存在明显差异，这也是论文重点研究 domain shift 的原因。

### 评价任务与指标

- **临床任务：** UPDRS gait severity classification，主要报告 macro-F1 等分类指标。
- **2D-to-3D lifting / reconstruction：** 使用 MPJPE 等三维人体误差指标。
- **泛化协议：** within-dataset、cross-dataset、LODO、MIDA。

## 主要结果

NeurIPS 官方页面报告，使用 CARE-PD 进行 motion pretraining 后，三维任务的 MPJPE 从 **60.8 mm 降至 7.5 mm**；同时 PD severity prediction 的 macro-F1 提升 **17 个百分点**。作者还报告，在临床评分任务中，所比较的 learned motion encoders 整体上优于基于手工 gait features 的传统 baseline。

这些结果的价值不只在于单一数字更高，而是说明经过临床步态数据预训练的三维运动表示可以同时改善低层运动恢复与高层疾病严重度建模，并且论文用跨数据集协议进一步检验模型是否真正学到可迁移的运动模式。

## 优点

- 将多个真实临床中心的数据统一成同一种三维人体运动表示，比单中心视频分类 benchmark 更接近临床部署时的 domain shift。
- 同时包含临床评分任务和 2D-to-3D / 3D reconstruction pretext tasks，可以研究底层运动表征与疾病识别之间的联系。
- 使用 LODO 和 MIDA 等协议，而不是只报告随机 train/test split，更适合判断模型是否真正具有跨中心泛化能力。
- 显式比较通用 motion encoder 与手工 gait features，为“深度特征是否真的优于临床特征”提供了基准。
- 数据与 benchmark code 已公开用于非商业研究，便于复现。

## 局限

- **临床标签不平衡：** 严重 gait impairment 的高 UPDRS score 较少，某些高严重度等级只存在于少数 cohort 中，因此 macro-F1 仍可能受到中心和严重度分布的共同影响。
- **数据异质性既是优势也是风险：** 不同中心采用 RGB 或 motion capture，统一 SMPL 后虽然输入格式一致，但上游重建误差、原始采集精度和 protocol 差异仍可能留下 domain-specific bias。
- **视频数据依赖人体重建：** RGB 数据先被转换成 SMPL，因此下游模型看到的不是原始视频，而是已经经过人体恢复模型处理后的表示；上游 HMR 错误可能传播到疾病分析。
- **许可限制：** 数据按非商业研究许可发布，并要求遵守各原始 cohort 的引用和使用条件。
- **样本计数存在版本差异：** arXiv v1 表格与项目页面后续统计在参与者总数上有 1 人差异，实际实验复现时应以下载版本的 metadata 为准。

## 个人评价

这篇论文对临床步态研究的价值很高，因为它把研究重点从“某一个数据集上的分类准确率”推进到“跨医院、跨设备是否还能工作”。对于医疗 AI，后者通常比继续提高同中心 accuracy 更重要。

尤其值得注意的是，作者没有只把 gait feature 或 skeleton 当成最终分类输入，而是把临床 gait data 当成一种 motion representation learning 资源。这样可以先通过 2D-to-3D、3D reconstruction 等任务学习人体运动结构，再迁移到疾病严重度判断，这种训练逻辑比完全端到端的小样本分类更适合临床数据规模。

阅读优先级：**很高**。

## 与我的研究关联

这篇论文与视频临床步态分析和脊柱疾病辅助诊断具有直接的方法学关联。虽然疾病对象是 Parkinson's Disease，而不是成人脊柱畸形，但“多中心 gait representation → clinical score/classification”这一框架可以直接作为临床视频研究的外部参考。

**推断性建议：** 可以重点借鉴以下实验设计：

- 将已有 gait classifier 从普通 cross-validation 扩展成按医院、设备或 cohort 划分的 leave-one-domain-out；
- 比较 learned motion representation 与 cadence、step length、trunk inclination、pelvis motion 等 clinical handcrafted features；
- 用 2D-to-3D / motion reconstruction 作为预训练，再做疾病分类或 severity prediction；
- 将当前 clinical-knowledge attention 与通用 motion encoder 结合，检查显式临床先验在跨中心条件下是否仍带来增益。

建议优先阅读论文的 **Data Harmonization、Clinical Score Estimation、Evaluation Protocols** 和 learned encoder vs. gait-feature baseline 部分。

## 后续阅读

- 进一步检查 CARE-PD 各 cohort 的年龄、性别、疾病严重度和采集设备分布，判断 domain shift 的主要来源。
- 复现 LODO 与 MIDA 设置，重点观察模型在完全未见临床中心上的下降幅度。
- 比较 MotionBERT、MotionAGFormer 等通用动作模型与临床知识引导模型在相同 gait sequence 上的特征差异。
- 研究 CARE-PD 的 SMPL gait representation 是否可以迁移到脊柱疾病、膝骨关节炎等其他病理性步态任务。
