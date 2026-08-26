---
title: "EgoPoseFormer v2: Accurate Egocentric Human Motion Estimation for AR/VR"
authors: "Zhenyu Li, Sai Kumar Dwivedi, Filip Maric, Carlos Chacón, Nadine Bertsch, Filippo Arcadu, Tomas Hodan, Michael Ramamonjisoa, Peter Wonka, Amy Zhao, Robin Kips, Cem Keskin, Anastasia Tkach, Chenhongyi Yang"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - multiview
  - egocentric
  - temporal-modeling
  - uncertainty
---

# EgoPoseFormer v2: Accurate Egocentric Human Motion Estimation for AR/VR

## 基本信息

- **作者：** Zhenyu Li, Sai Kumar Dwivedi, Filip Maric, Carlos Chacón, Nadine Bertsch, Filippo Arcadu, Tomas Hodan, Michael Ramamonjisoa, Peter Wonka, Amy Zhao, Robin Kips, Cem Keskin, Anastasia Tkach, Chenhongyi Yang
- **会议/期刊：** CVPR 2026 Highlight
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `multiview`, `egocentric`, `temporal-modeling`, `uncertainty`
- **论文：** [arXiv:2603.04090](https://arxiv.org/abs/2603.04090)
- **代码：** 暂无
- **数据集：** [EgoBody3M](https://github.com/facebookresearch/EgoBody3M)
- **项目主页：** [EgoPoseFormer v2](https://zhyever.github.io/EgoPoseFormerv2/)

## 一句话总结

EgoPoseFormer v2 将同步已标定的多视角 egocentric RGB、设备 6DoF、view identity、时序上下文与 per-keypoint uncertainty 统一用于 world-space 3D human pose estimation。

## 研究问题与动机

头戴/近身相机下人体经常离开 FoV，遮挡和截断严重，单帧单视角难以稳定恢复全身。论文希望通过 calibrated multi-view + temporal context 恢复不可见部位，同时保持推理效率。

## 核心方法

模型采用 holistic pose query，并串联 coarse-to-fine Transformer decoders。第二阶段把初步 3D pose 投影到各相机平面，将 projected keypoints 与 camera/view identity 作为 cross-attention 条件；causal temporal attention 利用历史帧补足不可见区域，同时预测 per-keypoint uncertainty。半监督阶段采用 teacher-student / ALS 扩大训练数据。

## 数据集与评价指标

EgoBody3M 包含约 3.4M 真实帧、4 台同步 global-shutter 相机，其中约 2.4M train、426k test。论文还使用私有 EGO-ITW-70M 做大规模半监督训练。指标包括 MPJPE、wrist MPJPE、velocity error 等，并报告运行效率。

## 主要结果

EgoBody3M 上整体 MPJPE 约 4.02 cm、wrist MPJPE 约 4.99 cm；相较复现的 EgoBody3M baseline 和 EgoPoseFormer，整体 MPJPE 分别改善约 22.4% 和 15.4%。完整 ALS + uncertainty 进一步改善结果。pose head 参数量较小，论文还报告了高度优化后的实时推理能力。

## 优点

- camera/view geometry 真正参与 feature sampling，而不是简单 concat。
- per-keypoint uncertainty 可直接用于下游融合权重。
- temporal attention 针对 egocentric 截断和不可见身体部位设计。

## 局限

- 输入假设同步、已标定多相机，并直接获得设备 6DoF，因此不解决未知 camera trajectory。
- 大规模 EGO-ITW-70M 为私有数据，完整半监督实验难以公开复现。
- 场景主要是 AR/VR egocentric setting，与户外 moving-selfie geometry 存在 domain gap。

## 个人评价

它最值得借鉴的是 conditioned multi-view cross-attention 和 uncertainty。对于 360 perspective crops，virtual camera intrinsics/rotation 天然已知，非常适合直接作为 attention condition。

## 与我的研究关联

可将 SAM 3D Body / 3D keypoint per-view outputs 与 virtual-camera geometry结合，设计类似 conditioned cross-view attention；uncertainty 则可作为 camera-human joint optimization 的 joint-wise 权重。其框架适合作为 camera-aware fusion 的网络设计参考。

## 后续阅读

- LAMP。
- Balanced Multi-Modal Learning in 3D HPE。
- 在同中心多 perspective views 上测试 view-conditioned attention 的有效性。
