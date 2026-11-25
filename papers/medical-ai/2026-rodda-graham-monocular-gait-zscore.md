---
title: "Quantifying Rodda and Graham Gait Classification from 3D Makerless Kinematics derived from a Single-view Video in a Heterogeneous Pediatric Clinical Cohort"
authors: "Lauhitya Reddy, Seth Donahue, Jeremy Bauer, Susan Sienko, Anita Bagley, Joseph Krzak, Maura Eveld, Karen Kruger, Ross Chafetz, Vedant Kulkarni, Hyeokhyen Kwon"
venue: "arXiv"
year: 2026
reading_date: 2026-11-26
status: skimmed
tags:
  - clinical-gait
  - monocular-3d-pose
  - cerebral-palsy
  - z-score
  - clinical-regression
  - agcn
---

# Quantifying Rodda and Graham Gait Classification from 3D Makerless Kinematics derived from a Single-view Video in a Heterogeneous Pediatric Clinical Cohort

## 基本信息

- **作者：** Lauhitya Reddy, Seth Donahue, Jeremy Bauer, Susan Sienko, Anita Bagley, Joseph Krzak, Maura Eveld, Karen Kruger, Ross Chafetz, Vedant Kulkarni, Hyeokhyen Kwon
- **会议/期刊：** arXiv:2605.11314（预印本，正式 venue 暂无核验）
- **首次提交：** 2026-05-11
- **年份：** 2026
- **阅读日期：** 2026-11-26
- **阅读状态：** `skimmed`
- **标签：** `clinical-gait`、`monocular-3d-pose`、`cerebral-palsy`、`z-score`、`clinical-regression`、`agcn`
- **论文：** https://arxiv.org/abs/2605.11314
- **DOI：** https://doi.org/10.48550/arXiv.2605.11314
- **代码：** 暂无
- **数据集：** 暂无（原文为 Shriners Children’s Philadelphia 临床队列，未核验公开下载入口）
- **项目主页：** 暂无

## 一句话总结

从单视角临床视频学习膝踝运动学的连续正常参考偏差 z-score，以独立仪器步态分析作为监督和评价，而不是仅输出疾病类别。

## 研究问题与动机

Rodda and Graham 步态分型使用矢状面膝/踝运动学偏差，但仪器化 3D-IGA 成本较高且观察式分型存在评估者差异；作者检验单视角视频能否恢复连续偏差并支持分型。

## 核心方法

- 由 MeTRAbs 的 BML-MoVi 87 点输出恢复单目 3D 骨架，插值、6 Hz Butterworth 低通、hip-centering 与尺度归一化，再计算 24 通道关节角或使用坐标。
- 由独立 3D-IGA 选择代表步态周期，取 mid-stance 20–45% 的膝踝平均矢状面角度，相对典型发育参考人群计算 z=(θ−μ)/σ。
- 1.5 s / 90 帧窗口、1 s stride，窗口继承 trial 级标签。AGCN 提取运动表征，ViT 聚合窗口 embedding 到 trial 级连续回归输出；以 ±1 阈值及 Rodda–Graham 规则产生步态分型。

## 数据集与评价指标

152 名可步行儿童/年轻患者（年龄 12.1±4.0 岁，60 种主要诊断、CP 54 人），529 次步行 trial，矢状面等 8 个独立分析视角及同步 marker-based 3D-IGA；完整视角 1,058 个双侧 limb samples。五次重复的 grouped 5-fold CV，以患者 ID 分组，3:1:1 train/val/test，nested 超参数选择。对比 direct biomechanics、DCL、ST-GCN、AGCN、AGCN+ViT。指标 R²、CCC、MAE、RMSE、Bland–Altman、AUROC、AUPRC、F1。

## 主要结果

- Table 5：AGCN+ViT 膝 z-score R²=0.80±0.02、CCC=0.89±0.02、MAE=1.55±0.07；踝 R²=0.57±0.02、CCC=0.72±0.02、MAE=1.64±0.05。AGCN 平均池化膝 R²=0.77，踝 R²=0.57。直接由单目角度计算 z-score 的最佳矢状面 R² 仍为负。
- Table 7：膝过屈二分类 AUROC=0.88±0.02，recall=0.83±0.02，specificity=0.72±0.01；Table 6 七类准确率 43±1%，macro-F1=0.37±0.03，macro-AUROC=0.78±0.01。
- 接近分类阈值的困难样本 CCC 仅 0.27，R²=-1.60；严重异常有向均值回归偏差。

## 优点

- 使用真实异质临床人群与同步独立 3D-IGA 参考；分组 CV 避免同一患者多个 trial 泄漏。
- 分离连续运动学回归、二分类与七分类，报告一致性、阈值边界和严重度偏差。

## 局限

- 单中心便利样本，尚缺外部验证；踝足关键点的误差及罕见分型不平衡限制性能。
- 推断：临床纵向应用仍需独立纵向可靠性验证；本研究并未证明成年 ASD 的 radiographic severity 可直接由儿童膝踝 z-score 得出。
- 专用代码、队列下载和项目主页暂无核验。

## 个人评价

推断：阅读优先级高；价值类型 Baseline / Method Module / Related Work。比单纯 normative reconstruction 更值得借鉴的是独立临床指标监督、按患者分组、极端病理低估与阈值误差分析。

## 与我的研究关联

推断：ClinicalGated 可新增连续临床量回归辅助任务（仅限已有同步且定义可靠的 SVA、TPA、PT 等影像学指标或独立运动学标注），比较 RGB-only、pose-only、direct geometry、AGCN+ViT、multitask fusion。影像静态测量与动态步态指标含义不同，必须按患者配对、报告回归误差/CCC 与分类 AUROC；无健康参考时不能把 Non-ASD 当正常。结合 GenGait 时区分无监督 deviation 与真实临床量监督，检查严重病例的系统性低估。

## 后续阅读

- 优先 Methods 的 Video-Based Gait Analysis Pipeline、Table 4–8 及 Discussion 的 direct biomechanics、boundary effects、severity bias。
- 对照 [GenGait](2026-gengait-normative-twin-anomaly-detection.md) 与 [ClinicalGait 相关标记测量](2026-biomechanically-accurate-gait-analysis.md)。
- 事实来源：https://arxiv.org/html/2605.11314v1。
