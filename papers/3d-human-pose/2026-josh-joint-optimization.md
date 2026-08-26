---
title: "Joint Optimization for 4D Human-Scene Reconstruction in the Wild"
authors: "Zhizheng Liu, Joe Lin, Wayne Wu, Bolei Zhou"
venue: "ICLR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - human-scene-reconstruction
  - camera-pose
  - joint-optimization
  - world-coordinate
---

# Joint Optimization for 4D Human-Scene Reconstruction in the Wild

## 基本信息

- **作者：** Zhizheng Liu, Joe Lin, Wayne Wu, Bolei Zhou
- **会议/期刊：** ICLR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `human-scene-reconstruction`, `camera-pose`, `joint-optimization`, `world-coordinate`
- **论文：** [arXiv:2501.02158](https://arxiv.org/abs/2501.02158)
- **代码：** [genforce/JOSH](https://github.com/genforce/JOSH)
- **数据集：** SLOPER4D、EMDB、RICH
- **项目主页：** [JOSH](https://vail-ucla.github.io/JOSH/)

## 一句话总结

JOSH 将 camera、scene 与 SMPL human motion 放入同一优化目标，通过 scene correspondence、2D reprojection、temporal prior 与 human-scene contact 相互约束，改善野外单目视频中的 world-coordinate 4D reconstruction。

## 研究问题与动机

传统 human-scene pipeline 往往先估 camera/scene，再恢复人体；任何 camera scale、depth 或 HMR 错误都会逐级传播。JOSH 的核心主张是这些变量不应完全独立求解，而应利用接触和重投影关系做联合校正。

## 核心方法

以现成 dense scene reconstruction、HMR 和 contact prediction 为初始化，同时优化 camera intrinsics/extrinsics、scene scale/depth 与 SMPL human motion。损失包括 scene correspondence、2D keypoint reprojection、human temporal prior，以及两类 contact consistency，使人体和场景在 metric world frame 中对齐。

## 数据集与评价指标

实验使用 SLOPER4D 的 6 个公开序列、EMDB-2 的 25 个序列，以及 RICH 的 40 个 moving-camera 序列。指标覆盖 W-MPJPE、WA-MPJPE、RTE、ATE、AbsRel、Chamfer Distance、jitter 与 foot sliding。

## 主要结果

JOSH 在 EMDB 上报告 WA-MPJPE 约 68.9 mm、W-MPJPE 174.7 mm、RTE 1.3%；SLOPER4D 上约 120.0 mm / 438.3 mm / 1.8%。在相同初始化下，joint optimization 可把 jitter 从约 123.9 降至 7.6，foot-floating rate 从约 9.0% 降到 3.3%。

## 优点

- 真正把 camera、human、scene 同时视为待优化变量。
- 指标覆盖姿态、全局轨迹、场景和物理合理性。
- contact 被用作连接人体和场景 metric geometry 的桥梁。

## 局限

- 强依赖 off-the-shelf 初始化；若 camera/scene/HMR 初值很差，联合优化并不能保证恢复。
- contact correspondence 需要可靠接触和可见几何。
- 长视频目前仍采用 chunk-based 处理，global bundle adjustment 有待进一步完善。

## 个人评价

JOSH 是 camera-human joint optimization 方向最直接的参考之一。它也提醒后续方法必须同时证明 camera 和 human 都改善，而不能只展示最终 pose 数值。

## 与我的研究关联

可以把 scene branch 简化为 ViPE/AnyCam camera prior + 多 perspective 3D keypoints，形成更轻量的 camera-human mutual refinement。实验可逐步比较 camera-only、human-only、camera-human、camera-human-scene 四种优化层级。

## 后续阅读

- HAC。
- PhysDynPose。
- UniSH。
- 在 360 自拍和雪地场景中研究 contact 不可靠时的替代几何约束。
