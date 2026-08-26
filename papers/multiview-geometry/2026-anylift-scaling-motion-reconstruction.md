---
title: "AnyLift: Scaling Motion Reconstruction from Internet Videos via 2D Diffusion"
authors: "Hongjie Li, Heng Yu, Jiaman Li, Hong-Xing Yu, Ehsan Adeli, C. Karen Liu, Jiajun Wu"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - multiview
  - motion-reconstruction
  - diffusion
  - camera-conditioned
  - 3d-human-pose
---

# AnyLift: Scaling Motion Reconstruction from Internet Videos via 2D Diffusion

## 基本信息

- **作者：** Hongjie Li, Heng Yu, Jiaman Li, Hong-Xing Yu, Ehsan Adeli, C. Karen Liu, Jiajun Wu
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `multiview`, `motion-reconstruction`, `diffusion`, `camera-conditioned`, `3d-human-pose`
- **论文：** [arXiv:2604.17818](https://arxiv.org/abs/2604.17818)
- **代码：** 暂无
- **数据集：** 互联网 gymnastics / martial-arts clips；AIST++ 用于带 3D GT 的受控评估
- **项目主页：** [AnyLift](https://awfuact.github.io/anylift/)

## 一句话总结

AnyLift 通过 camera-conditioned 2D diffusion 生成额外虚拟视角，再利用多视角一致性恢复世界坐标人体运动，从而在缺乏大规模 3D MoCap 的互联网动态视频上扩展 motion reconstruction。

## 研究问题与动机

长尾体育动作往往只有互联网视频，没有高质量 3D GT；同时真实视频存在动态相机和集中视角分布。直接训练 3D motion model 容易受数据规模限制，因此论文转向大量易获得的 2D pose，并利用相机条件生成更多观察方向。

## 核心方法

第一阶段训练以 camera trajectory 与 epipolar information 为条件的 single-view 2D diffusion，产生额外虚拟视角；第二阶段使用 multi-view 2D diffusion 学习跨视角一致 motion；最后通过多视角几何恢复 world-coordinate SMPL/HOI。hybrid training 保留真实视频的 global 2D root motion，同时结合已有 3D estimator 的 local pose，并随机改变 camera 后重投影。

## 数据集与评价指标

作者处理约 4000 个 gymnastics 和 5000 个 martial-arts 10 秒 clip，过滤后约 1600 / 3000 个，按 9:1 划分。ViTPose 提取 2D pose，MegaSaM 提供 camera pose。AIST++ 用于有 3D GT 的动态相机实验。指标包括 Troot、MPJPE、PA-MPJPE、foot sliding，以及真实网络视频上的 2D reprojection / generation metrics。

## 主要结果

AIST++ 合成动态相机实验中，AnyLift 达到 Troot 64.2、MPJPE 109.3、PA-MPJPE 83.0、FS 0.446；MVLift 对应 64.9、122.1、94.3、0.487。真实网络视频上 gymnastics J2D 从 33.1 降至 21.6，martial arts 从 24.6 降至 15.1。去掉 hybrid training 后性能明显退化。

## 优点

- 利用大规模 2D 互联网视频，降低对 3D GT 的依赖。
- camera-conditioned virtual-view generation 与动态相机问题直接相关。
- 对真实网络视频和有 GT 的 AIST++ 都进行了验证。

## 局限

- AIST++ 中动态相机主要是模拟的；真实动态相机互联网数据缺少完整 3D GT。
- 依赖可靠 2D keypoints 和 camera pose，上游错误会传入多视角恢复。
- 模型按动作域训练，跨动作/跨相机配置 zero-shot 泛化仍需加强。

## 个人评价

AnyLift 的核心价值是证明“2D motion + camera condition + virtual views”可以替代部分昂贵 3D 数据。对多透视 360 pipeline，这种表示比直接依赖固定 camera ID 更容易泛化。

## 与我的研究关联

360 ERP 切出的 perspective views 天然具有已知 virtual rotation/intrinsics，可把真实 physical camera trajectory 与 virtual-view geometry结合，构造类似 AnyLift 的 camera-conditioned representation。可比较 per-view 3D fusion 与 2D virtual-view lifting 两条路线。

## 后续阅读

- Mocap-2-to-3。
- LAMP。
- 在 360 multi-perspective 设置下研究 camera-conditioned 2D / 3D motion prior。
