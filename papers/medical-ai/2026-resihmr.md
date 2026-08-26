---
title: "ResiHMR: Residual-Limb Aware Single-Image 3D Human Mesh Recovery for Individuals with Limb Loss"
authors: "Jiaying Ying, Heming Du, Kaihao Zhang, Sean M. Tweedy, Xin Yu"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - medical-ai
  - 3d-human-pose
  - human-mesh-recovery
  - clinical-vision
---

# ResiHMR: Residual-Limb Aware Single-Image 3D Human Mesh Recovery for Individuals with Limb Loss

## 基本信息

- **作者：** Jiaying Ying, Heming Du, Kaihao Zhang, Sean M. Tweedy, Xin Yu
- **会议/期刊：** CVPR 2026 Highlight
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `3d-human-pose`, `human-mesh-recovery`, `clinical-vision`
- **论文：** [arXiv:2604.28025](https://arxiv.org/abs/2604.28025)
- **代码：** 暂无
- **数据集：** LDPose-LimbLoss Evaluation Dataset
- **项目主页：** 暂无

## 一句话总结

ResiHMR 针对标准 SMPL-X 假设完整肢体导致截肢者 HMR 产生错误补全的问题，引入 residual endpoint、有效运动学子图和残肢表面重建，使单图人体网格能适配 limb-loss topology。

## 研究问题与动机

通用 HMR 通常在完整人体上训练，遇到截肢/残肢人群时会强行补出不存在的 distal limb，甚至扭曲邻近关节。医疗视觉中，这类“正常人体先验”可能系统性掩盖真实病理结构。

## 核心方法

先用现有 HMR 初始化 SMPL-X，再使用 Residual Anchor-Factor Optimization 根据残肢 endpoint 修改有效运动学子图；Residual-Limb Reconstruction 删除不合理 distal geometry，并重建平滑、闭合的残肢终止表面。方法可作为现有 HMR backbone 的 plug-in refinement。

## 数据集与评价指标

LDPose-LimbLoss Evaluation Dataset 包含 255 张真实图像，每张提供 17 个标准 2D keypoints、8 个 residual endpoints 与人体 mask，仅用于 evaluation。指标包括 2D MPJPE 和 mIoU；论文没有 limb-loss 3D mesh / joint GT。

## 主要结果

以 SMPLify-X 为 backbone，Intact Kpts MPJPE 约 41.32→37.40 px，mIoU 约 0.662→0.703；以 HSMR 为 backbone，Residual-Limb MPJPE 约 73.61→23.19 px，Body Kpts 28.27→24.75 px，mIoU 0.705→0.741。

## 优点

- 直接指出通用人体模型拓扑对残障/病理人群的结构性偏差。
- 可作为现有 HMR 的后处理模块，不要求重训完整 backbone。
- 数据包含 residual endpoints，评价目标针对实际问题。

## 局限

- 没有 limb-loss 人群的真实 3D mesh / 3D joint GT，当前证据主要来自 2D reprojection 和 mask。
- 数据规模仅 255 张，并主动排除了部分严重遮挡/裁切案例。
- 优化式 plug-in 尚未学习大规模残肢形态分布。

## 个人评价

这篇对医疗 AI 的启发非常重要：人体模型的“正常结构先验”不一定对患者安全。病理姿态研究也应检查 backbone 是否把异常动作系统性拉回健康分布。

## 与我的研究关联

在脊柱畸形、异常步态或其他患者视频中，可以探索 pathology-aware pose prior、joint reliability 或结构自适应约束；评估时应区分模型误差与真实病理偏离，而不能只看通用 benchmark 的 MPJPE。

## 后续阅读

- SIMSPINE。
- SKEL-CF。
- 临床人群中通用 HMR prior 的 bias 与 uncertainty calibration。
