---
title: "MoRe: Motion-aware Feed-forward 4D Reconstruction Transformer"
authors: "Juntong Fang, Zequn Chen, Weiqi Zhang, Donglin Di, Xuancheng Zhang, Chengmin Yang, Yu-Shen Liu"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - multiview
  - camera-pose
  - dynamic-scene
  - 4d-reconstruction
  - transformer
---

# MoRe: Motion-aware Feed-forward 4D Reconstruction Transformer

## 基本信息

- **作者：** Juntong Fang, Zequn Chen, Weiqi Zhang, Donglin Di, Xuancheng Zhang, Chengmin Yang, Yu-Shen Liu
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `camera-pose`, `dynamic-scene`, `4d-reconstruction`, `transformer`
- **论文：** [arXiv:2603.05078](https://arxiv.org/abs/2603.05078)
- **代码：** [HellexF/MoRe](https://github.com/HellexF/MoRe)
- **数据集：** Dynamic Replica、PointOdyssey、Spring、Virtual KITTI、TartanAir、Co3Dv2、ScanNet、BlendedMVS、Hypersim、ARKitScenes、Waymo、OmniWorld-Game
- **项目主页：** [MoRe](https://hellexf.github.io/MoRe/)

## 一句话总结

MoRe 通过 motion-aware attention、causal temporal modeling 和相机 token refinement，在动态视频中同时恢复 camera pose、depth、dynamic point map 与 motion mask，重点解决动态前景干扰 camera estimation 的问题。

## 研究问题与动机

传统 feed-forward 3D reconstruction 或 SLAM 容易把人体和其他动态物体的运动误认为相机运动；优化式 4D reconstruction 又往往速度慢。MoRe 希望用一个统一 Transformer 在单目动态视频上做 feed-forward 4D reconstruction，同时显式区分 scene motion 与 camera motion。

## 核心方法

Attention-Forcing 在训练时迫使 camera token 更多依赖静态区域；Grouped Causal Attention 用分组因果机制做流式时序建模；模型还预测 motion mask 与 dynamic point map。为了补偿严格 causal inference 的长时漂移，作者增加类似 bundle adjustment 的 post-hoc camera-token refinement。

## 数据集与评价指标

训练整合 12 个公开数据集，覆盖 synthetic、indoor、outdoor 与 autonomous-driving 场景。camera pose 在 Sintel、TUM-dynamics、Bonn、ScanNet 等测试，使用 ATE、RPE-trans、RPE-rot；depth 使用 AbsRel、δ1.25 等指标。部分动态数据集为 zero-shot 测试。

## 主要结果

Sintel 上 full-attention MoRe 的 ATE 约 0.0877，VGGT 约 0.1715；streaming MoRe 约 0.1474，Stream3R 约 0.2144。streaming MoRe 在 Sintel depth 上 AbsRel 约 0.254、δ1.25 约 0.637；KITTI 上约 0.072 / 0.966。消融显示去掉 Attention-Forcing 后 Sintel ATE 从约 0.147 恶化到 0.163，去掉 BA-like refinement 约为 0.155。

## 优点

- 明确针对动态区域干扰 camera estimation 的问题设计 attention 机制。
- 同时输出 camera、depth、motion mask 和 dynamic geometry，适合作为场景几何前端。
- 支持 streaming 和 full-attention 两种设置。

## 局限

- 不是人体专用模型，不直接输出 skeleton/SMPL，因此仍需额外 HMR branch。
- streaming 版本长时 camera pose 仍会退化，需要后处理聚合。
- 训练阶段 Attention-Forcing 依赖 motion-mask supervision。

## 个人评价

MoRe 对自拍视频尤其有启发：人物通常是最大动态前景，而 camera motion 应主要从静态背景估计。相比简单 mask 掉人体，它提供了更系统的 learned camera/scene representation。

## 与我的研究关联

可作为 ViPE / VGGT / AnyCam 之外的 camera prior。可以测试 `无人体 mask / hard mask / soft uncertainty / MoRe-style motion-aware attention`，再把得到的 camera trajectory 与多 perspective 3D keypoints 联合优化。

## 后续阅读

- AnyCam。
- OnlineHMR 的 soft-human-mask SLAM。
- VGGT / ViPE 在动态人体自拍视频中的 camera trajectory 鲁棒性比较。
