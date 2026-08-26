---
title: "EmbodMocap: In-the-Wild 4D Human-Scene Reconstruction for Embodied Agents"
authors: "Wenjia Wang, Liang Pan, Huaijin Pi, Yuke Lou, Xuqian Ren, Yifan Wu, Zhouyingcheng Liao, Lei Yang, Rishabh Dabral, Christian Theobalt, Taku Komura"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - multiview
  - moving-camera
  - human-scene-reconstruction
  - world-coordinate
  - 4d-reconstruction
---

# EmbodMocap: In-the-Wild 4D Human-Scene Reconstruction for Embodied Agents

## 基本信息

- **作者：** Wenjia Wang, Liang Pan, Huaijin Pi, Yuke Lou, Xuqian Ren, Yifan Wu, Zhouyingcheng Liao, Lei Yang, Rishabh Dabral, Christian Theobalt, Taku Komura
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `moving-camera`, `human-scene-reconstruction`, `world-coordinate`, `4d-reconstruction`
- **论文：** [arXiv:2602.23205](https://arxiv.org/abs/2602.23205)
- **代码：** [WenjiaWang0312/EmbodMocap](https://github.com/WenjiaWang0312/EmbodMocap)
- **数据集：** EmbodMocap_release
- **项目主页：** 暂无

## 一句话总结

EmbodMocap 用两台随人物移动的消费级 RGB-D 手机构建统一 metric world frame，联合恢复 camera、human motion 与 scene geometry，并展示双动态视角在真实 4D human-scene capture 中的价值。

## 研究问题与动机

高质量 human–scene motion 数据通常依赖固定多相机、mocap suit 或昂贵扫描设备，难以在真实环境采集。论文尝试使用两台可移动消费级设备，实现更轻量的野外 4D 人体—场景重建。

## 核心方法

双移动 RGB-D 序列先进行跨设备时空对齐和 metric calibration，再恢复场景几何、camera trajectory 与人体 motion。双视角提供额外几何约束，减少单目深度歧义。论文还构建 monocular downstream baseline，利用 π3 / VIMO 等模型分析数据对 world-coordinate reconstruction 的价值。

## 数据集与评价指标

公开 release 包含双移动视角、RGB-D、camera/human/scene 对齐结果等。论文在 EMDB 等数据上评价 WA-MPJPE、W-MPJPE、RTE，并在若干真实场景中研究运动跟踪和 4D reconstruction。场景运动跟踪实验包括 4 个场景、39 clips、约 28.86 分钟。

## 主要结果

EMDB subset 2 上，基础 downstream 组合的 WA-MPJPE/W-MPJPE/RTE 约为 83.56/229.04 mm/1.78%；同时微调 scene/camera 与 human modules 后约 82.21/220.65 mm/1.71%。结果说明双移动采集和统一 metric alignment 可改善 global reconstruction，但依然存在长序列误差累积。

## 优点

- 两台相机都在移动，更接近真实跟随拍摄和自拍视频，而不是固定 studio rig。
- camera、human、scene 都进入同一 metric frame。
- 数据公开，具有作为 moving-camera benchmark 的价值。

## 局限

- 依赖 RGB-D 手机和后处理对齐，不等价于任意 RGB-only 相机。
- monocular downstream 仍依赖多个预训练模型及 chunk-wise alignment。
- 长视频存在累积漂移；高速、低纹理户外环境更困难。

## 个人评价

这篇是“双移动相机为什么值得做”的直接依据。相比固定多相机 benchmark，它的采集几何更接近跟随式运动分析。

## 与我的研究关联

上下双 360 相机同样可以视为一个 rigid moving rig。可比较单 360、双 360，以及多 perspective crop 在 camera/human global reconstruction 上的增益；还可以借鉴 EmbodMocap 的 metric alignment protocol 来评价 ViPE/AnyCam + joint refinement。

## 后续阅读

- OnlineHMR。
- JOSH。
- LAMP。
- 双 360 rigid-rig 的 camera consistency 与 global human trajectory 建模。
