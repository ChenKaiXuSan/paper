---
title: "Towards Balanced Multi-Modal Learning in 3D Human Pose Estimation"
authors: "Mengshi Qi, Jiaxuan Peng, Xianlin Zhang, Huadong Ma"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - multimodal-learning
  - explainable-ai
  - sensor-fusion
---

# Towards Balanced Multi-Modal Learning in 3D Human Pose Estimation

## 基本信息

- **作者：** Mengshi Qi, Jiaxuan Peng, Xianlin Zhang, Huadong Ma
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `multimodal-learning`, `explainable-ai`, `sensor-fusion`
- **论文：** [arXiv:2501.05264](https://arxiv.org/abs/2501.05264)
- **代码：** [MICLAB-BUPT/AWC](https://github.com/MICLAB-BUPT/AWC)
- **数据集：** [MM-Fi](https://github.com/ybhbingo/MMFi_dataset)
- **项目主页：** 暂无

## 一句话总结

该工作把多模态 3D HPE 中“强模态压制弱模态”的训练失衡问题显式量化，并通过 Shapley contribution 与 Fisher-information-driven Adaptive Weight Constraint 调节各分支学习速度，从而改善融合效果。

## 研究问题与动机

RGB、LiDAR、mmWave、WiFi 等多模态联合训练时，网络往往快速依赖最容易优化的强模态，导致其他模态贡献逐渐降低。简单 concatenation 或 attention 并不能保证各模态被有效利用，因此论文关注的核心不是增加模态，而是控制优化过程中的贡献失衡。

## 核心方法

首先利用基于 Pearson correlation 的 Shapley-style contribution 分析各模态在不同训练阶段的实际贡献。随后使用 Fisher Information Matrix 估计参数重要性，并构建 Adaptive Weight Constraint（AWC），对过快收敛的强模态施加更强约束，让弱模态有机会学习互补信息。

## 数据集与评价指标

实验使用 MM-Fi，包含约 1080 个 clips、约 32 万同步帧、40 名受试者，覆盖 14 个日常动作和 13 个康复动作。输入组合包含 RGB-derived 2D joints、LiDAR、mmWave 和 WiFi CSI，输出 3D joints。主要指标为 MPJPE、PA-MPJPE，并比较不同 protocol 下的融合效果。

## 主要结果

Concatenation baseline 在三个 protocol 上的 MPJPE 约为 53.87 / 52.08 / 48.17 mm，引入 AWC 后下降到 51.16 / 50.71 / 47.55 mm；对应 PA-MPJPE 为 34.46 / 30.85 / 31.79 mm。论文的主要贡献在于展示训练平衡机制能带来稳定改进，而非依赖更复杂的 backbone。

## 优点

- 直接分析各模态“到底贡献了多少”，可解释性比只展示 attention map 更强。
- AWC 与具体 backbone 解耦，适合作为多分支网络的训练策略。
- 既有贡献度分析，也有最终 HPE 定量结果支持。

## 局限

- 主要实验集中在 MM-Fi，缺少跨数据集外部验证。
- 论文中的 RGB branch 实际使用由 RGB 提取的 2D joints，而不是完整视觉 feature，因此对原始 RGB/optical-flow 融合的可迁移性仍需验证。
- 40 名受试者规模有限，多模态硬件环境也与普通临床视频不同。

## 个人评价

这篇最值得借鉴的是“把融合是否有效转化成可测的训练过程问题”。如果后续多分支模型出现 RGB、光流或 skeleton 某一支长期独占性能，Shapley contribution + FIM regularization 是很好的解释与控制工具。

## 与我的研究关联

可用于 RGB / optical flow / keypoint 三分支，或双视角/双 360 分支的贡献度分析。除了最终 accuracy/MPJPE，可记录各 branch contribution curve，并比较普通融合与 AWC 式训练约束是否提高稳定性和可解释性。

## 后续阅读

- 多模态 Shapley value 与 modality collapse 文献。
- Calibrated Uncertainty for Trustworthy Clinical Gait Analysis。
- 在 gait classification 与 camera-aware pose fusion 中复现 contribution analysis。
